Tentu, ini adalah draf `README.md` yang menjelaskan standar model JSON yang telah kita buat. Anda bisa menyimpan file ini dengan nama `README.md` di *root* direktori proyek Anda.

-----

# Standar Model Definisi Sistem dalam Format JSON

Dokumen ini menjelaskan standar format JSON yang digunakan untuk mendefinisikan model sebuah sistem. Tujuannya adalah untuk menciptakan struktur yang konsisten, jelas, dan dapat dibaca oleh mesin (*machine-readable*) sebagai *single source of truth* untuk meng-generate *source code*, dokumentasi, atau artefak lainnya secara otomatis. 🧑‍💻

## 1\. Filosofi Desain

  * **Deklaratif**: Model mendeskripsikan "apa" yang ada di dalam sistem (entitas, atribut, relasi), bukan "bagaimana" cara kerjanya.
  * **Struktur Jelas**: Memisahkan definisi **entitas** (`entities`), **relasi** (`relationships`), dan **konfigurasi global** (`configuration`) untuk kemudahan pemahaman dan parsing.
  * **Siap Kompilasi**: Struktur dirancang agar mudah diproses oleh skrip atau *compiler* untuk diubah menjadi kelas, properti, dan metode dalam berbagai bahasa pemrograman.

-----

## 2\. Struktur File Utama

File JSON harus memiliki struktur akar (*root structure*) sebagai berikut:

```json
{
  "system_name": "nama_sistem",
  "version": "1.0.0",
  "configuration": { ... },
  "entities": [ ... ],
  "relationships": [ ... ]
}
```

  * `system_name`: Nama subsistem atau sistem utama (contoh: "daring").
  * `version`: Versi dari model definisi ini.
  * `configuration`: Objek untuk menyimpan konfigurasi global, seperti daftar *event* yang valid di seluruh sistem.
  * `entities`: Sebuah *array* yang berisi semua objek entitas (kelas, superkelas, dll.).
  * `relationships`: Sebuah *array* yang mendefinisikan hubungan antar entitas.

-----

## 3\. Definisi Entitas (`entities`) 🏗️

Setiap objek dalam *array* `entities` merepresentasikan sebuah "benda" dalam sistem.

### Properti Umum Entitas:

  * `entity_type`: Jenis entitas. Nilai yang valid: `"class"`, `"superclass"`, `"association_class"`.
  * `id`: Pengenal unik untuk entitas ini di dalam model.
  * `name`: Nama entitas yang mudah dibaca (contoh: "mahasiswa").
  * `key_letter`: Singkatan atau kunci unik untuk entitas (contoh: "mhs").
  * `attributes`: *Array* yang berisi semua atribut milik entitas.
  * `inherits_from`: (Opsional) `id` dari `superclass` jika entitas ini adalah turunan.

### 3.1. Atribut (`attributes`)

Setiap atribut dalam *array* `attributes` memiliki struktur:

```json
{
  "name": "nama_atribut",
  "data_type": "tipe_data",
  "attribute_type": "jenis_atribut",
  "default_value": "nilai_default"
}
```

  * **`data_type`**: Tipe data dari atribut. Nilai yang valid:
      * `id`: Kunci unik.
      * `string`: Teks.
      * `integer`: Bilangan bulat.
      * `real`: Bilangan desimal.
      * `state`: Mereferensikan sebuah keadaan dari *state machine*.
      * `timestamp`: Waktu dan tanggal.
      * `boolean`: Nilai `true`/`false`.
  * **`attribute_type`**: Peran atribut. Nilai yang valid:
      * `naming`: Atribut yang menjadi pengenal unik (Primary Key).
      * `descriptive`: Atribut yang mendeskripsikan entitas.
      * `referential`: Atribut yang merujuk ke entitas lain (Foreign Key).

### 3.2. State Machine (`state_machine`)

Jika sebuah entitas memiliki siklus hidup atau perilaku dinamis, definisikan di dalam objek `state_machine`.

```json
"state_machine": {
  "initial_state": "nama_state_awal",
  "states": [
    { "name": "aktif", "id": "1" },
    { "name": "cuti", "id": "2" }
  ],
  "transitions": [
    {
      "from_state": "aktif",
      "to_state": "cuti",
      "event": "setCuti",
      "parameters": ["nim:id"]
    }
  ]
}
```

  * `initial_state`: Nama keadaan awal saat objek pertama kali dibuat.
  * `states`: Daftar semua kemungkinan keadaan (*state*).
  * `transitions`: Aturan perubahan keadaan. Setiap transisi mendefinisikan perubahan dari `from_state` ke `to_state` yang dipicu oleh sebuah `event`.

-----

## 4\. Definisi Relasi (`relationships`) 🔗

*Array* ini mendefinisikan bagaimana entitas saling terhubung.

```json
{
  "relationship_id": "R1",
  "type": "association",
  "participants": [
    { "entity_id": "1", "role_name": "takes", "multiplicity": "0..*" },
    { "entity_id": "3", "role_name": "is taken by", "multiplicity": "0..*" }
  ],
  "association_class": { ... }
}
```

  * `relationship_id`: ID unik untuk relasi (contoh: "R1", "R2").
  * `type`: Jenis relasi (saat ini hanya `"association"`).
  * `participants`: *Array* yang berisi entitas yang terlibat dalam relasi.
      * `entity_id`: `id` dari entitas yang berpartisipasi.
      * `role_name`: Peran entitas dalam relasi tersebut.
      * `multiplicity`: Kardinalitas hubungan (`1..1`, `0..*`, `1..*`).
  * `association_class`: (Opsional) Jika relasi itu sendiri memiliki atribut, definisikan sebagai sebuah entitas di sini.

-----

## 5\. Contoh Penggunaan

Berikut adalah contoh sederhana definisi untuk kelas `mahasiswa`:

```json
{
  "entity_type": "class",
  "id": "1",
  "name": "mahasiswa",
  "key_letter": "mhs",
  "attributes": [
    { "name": "nim", "data_type": "id", "attribute_type": "naming" },
    { "name": "nama", "data_type": "string", "attribute_type": "descriptive" },
    { "name": "status", "data_type": "state", "attribute_type": "descriptive", "default_value": "aktif" }
  ],
  "state_machine": {
    "initial_state": "aktif",
    "states": [
      { "name": "aktif", "id": "1" },
      { "name": "lulus", "id": "3" }
    ],
    "transitions": [
      {
        "from_state": "aktif",
        "to_state": "lulus",
        "event": "setLulus",
        "parameters": ["nim:id", "tanggal_lulus:timestamp"]
      }
    ]
  }
}
```

## 6\. Alur Kerja yang Direkomendasikan

1.  **Pemodelan**: Definisikan semua entitas, atribut, dan relasi sistem Anda dalam satu atau beberapa file JSON sesuai standar ini.
2.  **Validasi**: (Opsional) Gunakan skema validator JSON untuk memastikan model Anda sesuai dengan standar yang ditetapkan.
3.  **Kompilasi/Generasi Kode**: Jalankan skrip *compiler* kustom yang akan membaca file JSON ini dan menghasilkan:
      * File-file kelas (`Mahasiswa.java`, `Dosen.py`, dll.).
      * Skrip migrasi database (SQL `CREATE TABLE`).
      * File dokumentasi API.
