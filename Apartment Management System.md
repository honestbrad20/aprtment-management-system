# Apartment Management System

## Latar Belakang

Seorang manajer IT dari sebuah apartemen mevvah di pengkolan kota Jakarta, ingin mempermudah proses pengelolaan apartemen tempat ia bekerja saat ini. Proses pengelolaan apartemen saat ini masih mengandalkan dokumen sheet yang tidak bisa diakses secara online, ditambah lagi ukuran dokumen tersebut semakin lama semakin besar. Oleh karena itu diperlukan adanya sistem pengelolaan apartemen yang dapat diakses secara online dengan mudah.

## Fitur

Berdasarkan kebutuhan itu, akan dibuat suatu sistem manajemen apartemen dengan persyaratan sebagai berikut.

1. Harus ada manajemen master data:

   1. **Unit Apartemen**, dengan deskripsi sbb.

      ```tsx
      enum ApartmentStatus {
        AVAILABLE = "available",
        RENTED = "rented",
        SOLD = "sold",
        UNAVAILABLE = "unavailable",
      }

      enum ApartmentDirection {
        NORTH,
        NORTHEAST,
        EAST,
        SOUTHEAST,
        SOUTH,
        SOUTHWEST,
        WEST,
        NORTHWEST,
      }

      enum ApartmentRentSchema {
        DAILY = "daily",
        WEEKLY = "weekly",
        MONTHLY = "monthly",
      }

      interface ApartmentUnit {
        id: number;
        unitCode: string;
        floor: number;
        rooms: number;
        direction: ApartmentDirection;
        status: ApartmentStatus;
        balcony: boolean;
        furnished: boolean;
        rentPrice: number;
        rentSchema: ApartmentRentSchema;
        sellPrice: number;
      }
      ```

   2. **Penghuni**, dengan deskripsi sbb.

      ```tsx
      enum MaritalStatus {
        SINGLE,
        TAKEN,
        MARRIED,
        DIVORCED,
        JONES,
      }

      interface Resident {
        id: number;
        fullname: string;
        email: string;
        phone: string;
        maritalStatus: MaritalStatus;
        dependents: number;
        birthDate: Date;
      }
      ```

2. Harus ada manajemen transaksi penyewaan dan penjualan apartemen.

   Unit apartemen yang sudah dibeli tidak dapat disewakan kembali dan status akan berubah menjadi **SOLD**.

   Unit apartemen yang sedang disewa (**RENTED**) tidak dapat dijual sampai masa sewa berakhir.

   Unit apartemen yang **UNAVAILABLE** tidak dapat dibeli.

   Profit dapat bernilai positif atau negatif.

   ```tsx
   interface Transaction {
     id: number;
     unit: ApartmentUnit;
     resident: Resident;
     transactionDate: Date;
     rentStartDate?: Date;
     rentEndDate?: Date;
     billingDate?: Date;
     period?: number;
     price: number;
     profit: number;
   }
   ```

3. Harus ada laporan pengelolaan apartemen dan transaksi

   1. Laporan **pengelolaan apartemen**

      | #   | Floor | Unit | Status | Price           | Rental Price  | Rental Schema | Details    | Actions |
      | --- | ----- | ---- | ------ | --------------- | ------------- | ------------- | ---------- | ------- |
      | 1   | 10    | 10AA | sold   | IDR 500.000.000 | IDR 5.000.000 | monthly       | - Rooms: 3 |

      - Furnished: yes
      - Balcony: no
      - Direction: west
      - Resident: Paijo Sukirman | - Manage |
        | 2 | 10 | 10AB | rented | IDR 450.000.000 | IDR 4.500.000 | monthly | - Rooms: 3
      - Furnished: no
      - Balcony: yes
      - Direction: southwest
      - Resident: Tukiyem Markonah | - Manage |
        | 3 | 10 | 10AC | available | IDR 550.000.000 | IDR 5.500.000 | weekly | - Rooms: 3
      - Furnished: yes
      - Balcony: yes
      - Direction: south
      - Resident: (none) | - Manage |

      **Persyaratan**

      1. Action **Manage** akan membuka page baru berisikan informasi lengkap terkait unit, termasuk informasi lengkap terkait penyewa/pembeli jika ada.
      2. Pada halaman **Manage** untuk melakukan pembaruan data terkait unit apartemen, dapat diarahkan menuju ke halaman update unit apartemen.
      3. Pada halaman **Manage** juga dapat melakukan pembaruan data terkait resident jika statusnya **SOLD/RENTED**, langsung ke halaman update resident.
      4. Pada halaman laporan ini dapat dilakukan filtering data apartemen berdasarkan floor, status, atau rental schema.
      5. Pada halaman laporan ini dapat dilakukan sorting data berdasarkan sell price maupun rental price, secara ascending atau descending

   2. Laporan **Transaksi**

      | #   | Floor | Unit | Resident         | Status | Price           | Profit         | Transaction Date | Rental Schema | Start / End Date          | Period   | Billing Date |
      | --- | ----- | ---- | ---------------- | ------ | --------------- | -------------- | ---------------- | ------------- | ------------------------- | -------- | ------------ |
      | 1   | 10    | 10AA | Paijo Sukirman   | sold   | IDR 550.000.000 | IDR 50.000.000 | 19-Sep-2022      | -             | -                         | -        | -            |
      | 2   | 10    | 10AB | Tukiyem Markonah | rented | IDR 4.000.000   | (IDR 500.000)  | 20-Sep-2022      | monthly       | 20-Sep-2022 / 19-Mar-2022 | 6 months | 20-Oct-2022  |

      **Persyaratan**

      1. Pada halaman laporan ini dapat dilakukan pencarian data resident berdasarkan nama.
      2. Pada halaman laporan ini dapat dilakukan filtering data transaksi berdasarkan floor atau status.
      3. Pada halaman laporan ini dapat dilakukan sorting data berdasarkan profit atau transaction date, secara ascending atau descending.

4. Harus ada **form transaksi penjualan / penyewaan**.
   1. Memiliki field dropdown Unit Apartemen yang available.
   2. Memiliki field dropdown nama lengkap calon Resident yang akan membeli/menyewa.
   3. Memiliki field dropdown jenis transaksi **sewa** atau **jual**.
   4. Memiliki field price (readonly) otomatis terisi sesuai jenis transaksi, jika **sewa** menampilkan harga **sewa**, jika **jual** menampilkan harga **jual**.
   5. Memiliki field transaction price yang dapat diisi, pada mulanya field ini langsung diisikan price pada resident sesuai jenis transaksi, namun dapat diubah.
   6. Memiliki field transaction date yang dapat diisi, field bertipe date.
   7. Jika memilih jenis transaksi **sewa**, akan menambahkan field baru sbb (jika transaksi jual tidak ditampilkan).
      1. Field start date, tanggal mulai sewa.
      2. Field end date, tanggal berakhir sewa.
      3. Field periode, dihitung otomatis berdasarkan tanggal mulai dan selesai, dikonversi sesuai skema sewa sebuah unit apartemen.
      4. Field billing date, tanggal tagihan sewa.
   8. Seluruh field wajib diisi, field tambahan untuk transaksi sewa hanya dapat wajib diisi ketika jenis transaksi adalah sewa.

## Technical Requirements

1. Wajib menerapkan HTTP Request dan Routing.
2. Wajib menambahkan fitur login user, seluruh halaman manajemen tidak dapat diakses tanpa login.
3. Wajib menerapkan form validation pada form transaction, master data, dan pada form login.
4. Bebas menggunakan state redux atau redux toolkit.
5. Bebas menggunakan function component atau class component.
6. Bebas menggunakan css framework manapun, selama dapat menghasilkan output tampilan yang rapih.
7. Bebas menggunakan template react JavaScript atau TypeScript.

## Kriteria Penilaian

1. Keseluruhan fitur terimplementasi dengan lengkap.
2. Technical requirements yang wajib sudah terpenuhi.
3. Tampilan rapih, dan jika bisa responsive.

### Timeline Pengerjaan

Sesuai kesepakatan bersama.
