Tentu, ini adalah dokumentasi yang lebih lengkap mengenai nilai-nilai (*values*) dalam model JSON. Panduan ini dirancang untuk membantu desainer dan arsitek sistem dalam membuat keputusan yang tepat saat merancang model.

---

## Panduan Desain dan Referensi Value untuk Model JSON 🧠

Dokumen ini adalah referensi mendalam untuk setiap *value* yang dapat dikonfigurasi dalam file model JSON. Tujuannya adalah untuk menjelaskan **apa arti** setiap pilihan dan **kapan harus menggunakannya**, serta **implikasi teknis** dari setiap keputusan desain.

---

### 1. Referensi `entity_type`

Properti ini mendefinisikan peran sebuah entitas dalam arsitektur sistem. Pilihan yang tepat akan menentukan bagaimana entitas tersebut diimplementasikan dalam kode dan database.

#### a. `"class"`
* **Deskripsi**: Entitas dasar yang akan menjadi objek konkret dalam sistem Anda. Ini adalah unit bangunan utama dari model Anda.
* **Kapan Digunakan?**: Gunakan ini untuk merepresentasikan hampir semua konsep utama dalam sistem Anda, seperti **Mahasiswa**, **Dosen**, **MataKuliah**, atau **Produk**.
* **Implikasi Teknis**:
    * **OOP**: Akan di-generate menjadi sebuah **kelas** (misal: `public class Mahasiswa`).
    * **Database**: Akan di-generate menjadi sebuah **tabel** utama (misal: `CREATE TABLE mahasiswa`).

#### b. `"superclass"`
* **Deskripsi**: Entitas abstrak yang berfungsi sebagai "induk" atau templat untuk `class` lainnya. Entitas ini tidak dapat dibuat menjadi objek secara langsung.
* **Kapan Digunakan?**: Gunakan saat Anda memiliki beberapa `class` yang berbagi atribut dan perilaku yang sama. Contoh: **`baseEntity`** bisa menjadi superclass untuk `mahasiswa` dan `dosen` yang sama-sama memiliki atribut `nama` dan `status`.
* **Implikasi Teknis**:
    * **OOP**: Akan di-generate menjadi sebuah **kelas abstrak** atau *interface* (`public abstract class baseEntity`). Kelas turunan akan mewarisi (`extends`) propertinya.
    * **Database**: Biasanya **tidak menjadi tabel sendiri**. Atributnya akan "turun" dan menjadi kolom di dalam tabel anak-anaknya (strategi *table-per-concrete-class*).

#### c. `"association_class"`
* **Deskripsi**: Sebuah `class` yang muncul dari sebuah relasi *Many-to-Many*. Entitas ini menyimpan atribut yang spesifik untuk hubungan antara dua entitas lain.
* **Kapan Digunakan?**: Gunakan ketika sebuah relasi `*..*` perlu memiliki data sendiri. Contoh: Relasi antara `Mahasiswa` dan `MataKuliah` adalah *Many-to-Many*. Kelas asosiasi **`mahasiswa_matakuliah`** bisa muncul untuk menyimpan nilai, semester pengambilan, atau status kelulusan mata kuliah tersebut untuk mahasiswa spesifik.
* **Implikasi Teknis**:
    * **OOP**: Di-generate menjadi sebuah kelas normal.
    * **Database**: Menjadi **tabel penghubung** (*junction/linking table*) yang berisi *foreign key* ke dua tabel yang dihubungkannya, ditambah kolom untuk atributnya sendiri.

---

### 2. Referensi `attribute_type` 🔑

Properti ini mendefinisikan peran sebuah atribut di dalam entitasnya, yang secara langsung memengaruhi struktur database.

#### a. `"naming"`
* **Deskripsi**: Atribut yang berfungsi sebagai pengenal unik untuk setiap instance/rekaman dari entitas.
* **Kapan Digunakan?**: Untuk ID unik seperti **NIM** (Nomor Induk Mahasiswa), **NIP** (Nomor Induk Pegawai), `product_code`, atau `id` numerik. Setiap entitas harus memiliki tepat satu atribut `naming`.
* **Implikasi Teknis**:
    * **Database**: Akan menjadi **Primary Key** dari tabel.
    * **OOP**: Menjadi properti final atau *read-only* setelah diinisialisasi.

#### b. `"descriptive"`
* **Deskripsi**: Atribut yang memberikan informasi atau deskripsi tentang entitas.
* **Kapan Digunakan?**: Untuk hampir semua atribut non-kunci, seperti `nama`, `alamat`, `tanggal_lahir`, `deskripsi`, `harga`, dll.
* **Implikasi Teknis**:
    * **Database**: Menjadi **kolom standar** di dalam tabel.
    * **OOP**: Menjadi properti biasa dengan *getter* dan *setter*.

#### c. `"referential"`
* **Deskripsi**: Atribut yang nilainya "menunjuk" atau merujuk ke atribut `naming` dari entitas lain. Fungsinya adalah untuk menciptakan hubungan.
* **Kapan Digunakan?**: Untuk menghubungkan satu entitas ke entitas lain. Contoh: Di dalam entitas `TugasAkhir`, atribut `mahasiswa_id` akan menjadi atribut referensial yang menunjuk ke `nim` di entitas `Mahasiswa`.
* **Implikasi Teknis**:
    * **Database**: Menjadi kolom **Foreign Key** yang memiliki *constraint* ke tabel lain.
    * **OOP**: Menjadi properti yang menyimpan ID, atau bisa juga diimplementasikan sebagai objek referensi ke entitas lain (misal: `private Mahasiswa mahasiswa;`).

---

### 3. Referensi `data_type`

Tipe data mendefinisikan jenis nilai yang dapat disimpan oleh sebuah atribut.

| Value | Deskripsi | Contoh Penggunaan | Implikasi Teknis (SQL/OOP) |
| :--- | :--- | :--- | :--- |
| `id` | Tipe data khusus untuk atribut pengenal. Bisa berupa string atau integer. | `nim`, `matkul_id` | `VARCHAR` atau `BIGINT`, `PRIMARY KEY` / `String` atau `Long` |
| `string` | Teks dengan panjang bervariasi. | `nama`, `judul_skripsi` | `VARCHAR(255)` / `String` |
| `text` | Teks dengan panjang tidak terbatas. | `deskripsi_matakuliah`, `abstrak` | `TEXT` / `String` |
| `integer` | Bilangan bulat. | `semester`, `jumlah_sks` | `INT` atau `BIGINT` / `int` atau `Long` |
| `real` | Bilangan desimal (pecahan). | `ipk`, `harga` | `DECIMAL` atau `DOUBLE` / `double` atau `BigDecimal` |
| `boolean`| Nilai benar atau salah (`true`/`false`). | `is_active`, `is_required` | `BOOLEAN` atau `TINYINT(1)` / `boolean` |
| `timestamp`| Menyimpan tanggal dan waktu. | `waktu_presensi`, `created_at` | `TIMESTAMP` atau `DATETIME` / `LocalDateTime` atau `Date` |
| `state` | Tipe khusus yang nilainya adalah salah satu dari keadaan yang didefinisikan di `state_machine`. | `status_mahasiswa` | `VARCHAR` (menyimpan nama state) atau `INT` / `Enum` atau `String` |

---

### 4. Panduan Desain `state_machine` 🔄

Gunakan `state_machine` untuk memodelkan entitas yang memiliki siklus hidup yang jelas.

#### Konsep Kunci
* **State (Keadaan)**: Kondisi diskrit di mana sebuah objek bisa berada. Contoh: `Draft`, `Submitted`, `Approved`, `Rejected`.
* **Event (Peristiwa)**: Aksi atau pemicu yang menyebabkan perubahan keadaan. Contoh: `submit()`, `approve()`, `reject()`.
* **Transition (Transisi)**: Aturan yang mendefinisikan perubahan dari satu *state* ke *state* lain saat sebuah *event* terjadi.

#### Langkah-langkah Desain
1.  **Identifikasi Siklus Hidup**: Tanyakan, "Objek ini bisa berada dalam kondisi-kondisi apa saja selama masa hidupnya?"
    * *Contoh Mahasiswa*: `Aktif`, `Cuti`, `Lulus`, `Dropout`.
2.  **Definisikan Pemicu (Events)**: Tanyakan, "Aksi apa saja yang dapat mengubah kondisi objek tersebut?"
    * *Contoh*: `mengajukanCuti`, `menyelesaikanStudi`, `diberhentikan`.
3.  **Petakan Transisi**: Hubungkan *state* dan *event*.
    * Jika mahasiswa dalam state `Aktif` dan terjadi event `menyelesaikanStudi`, maka state berikutnya adalah `Lulus`.
    * Jika dalam state `Aktif` dan terjadi event `mengajukanCuti`, maka state berikutnya adalah `Cuti`.
4.  **Tentukan Initial State**: Apa kondisi objek saat pertama kali dibuat? Biasanya `Aktif` atau `Baru`.

---

### 5. Panduan Desain Relasi dan Kardinalitas (`multiplicity`) 🔗

Kardinalitas atau `multiplicity` mendefinisikan aturan bisnis tentang berapa banyak instance satu entitas dapat terhubung ke instance entitas lain. Pilihan yang tepat sangat krusial untuk integritas data.

| Value | Arti | Contoh Kasus | Implikasi Database |
| :--- | :--- | :--- | :--- |
| `1..1` | **One-to-One** | Setiap **Mahasiswa** hanya boleh memiliki satu **TugasAkhir**, dan setiap **TugasAkhir** hanya milik satu **Mahasiswa**. | `tugas_akhir.mahasiswa_id` adalah **Foreign Key** dengan constraint **UNIQUE**. |
| `1..*` | **One-to-Many** | Satu **Dosen** dapat membimbing **banyak Mahasiswa**, tetapi setiap **Mahasiswa** hanya dibimbing oleh satu **Dosen** (Wali). | Tabel `mahasiswa` memiliki kolom `dosen_wali_id` sebagai **Foreign Key** ke tabel `dosen`. |
| `0..1` | **Zero-or-One** | Seorang **Pegawai** bisa memiliki satu **AkunSistem**, atau tidak sama sekali. | `pegawai.akun_sistem_id` adalah **Foreign Key** yang **NULL-able** (boleh kosong). |
| `0..*` atau `*` | **Zero-or-Many** | Satu **Produk** bisa memiliki **banyak Ulasan**, atau tidak sama sekali. | Tabel `ulasan` memiliki kolom `produk_id` sebagai **Foreign Key** ke tabel `produk`. |
| `*..*` | **Many-to-Many** | Satu **Mahasiswa** bisa mengambil **banyak MataKuliah**, dan satu **MataKuliah** bisa diambil oleh **banyak Mahasiswa**. | Membutuhkan **tabel penghubung** (didefinisikan dengan `association_class`) bernama `mahasiswa_matakuliah`. |
