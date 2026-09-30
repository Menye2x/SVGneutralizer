# SVG Pure Shape

SVG Pure Shape adalah utility berbasis browser untuk membersihkan dan menormalisasi file SVG secara lokal tanpa server-side processing. Fokus aplikasinya adalah mengurangi metadata/editor bloat, atribut proteksi, wrapper yang tidak perlu, serta merapikan struktur `<g>` tanpa mengubah geometri visual secara sengaja.

## Konsep aplikasi

Aplikasi bekerja dengan tiga tahap utama:

1. **Source** — pengguna dapat memilih atau drag-and-drop file SVG, atau paste markup SVG langsung.
2. **Clean / Neutralize** — markup diparse sebagai XML, metadata/editor artifacts dibersihkan sesuai cleaning profile, lalu struktur group dinormalisasi.
3. **Inspection & Export** — pengguna dapat membandingkan Original vs Clean, memeriksa struktur Unwrap/Ungroup, melihat markup hasil, melakukan ungroup manual per root group, kemudian mengunduh SVG hasil.

Seluruh transformasi dilakukan di browser melalui DOM/XML APIs. Tidak ada upload SVG ke server yang dibutuhkan oleh aplikasi.

## Fitur

### 1. SVG metadata & editor cleanup

Mode **Remove editor metadata** menangani namespace/prefix editor yang dikenali, termasuk:

- Inkscape
- Sodipodi
- Adobe / Illustrator
- Sketch
- Figma
- CorelDRAW (`corel`, `corel-draw`, `cdr`)
- atribut `data-*` ketika opsi proteksi diaktifkan
- XML comments / processing instructions
- `<metadata>` dan `<namedview>` pada mode yang relevan

Daftar ini bersifat **known editor cleanup**, bukan database universal semua aplikasi yang pernah menulis SVG.

### 2. Strip data-lock / protection

Opsi ini menghapus atribut `data-*`, termasuk pola seperti `data-lock`, `data-protection`, dan `data-editor`.

### 3. Remove app-specific attributes

Membersihkan atribut/editor namespace tertentu yang tidak diperlukan oleh geometry output. **Atribut SVG root `version` tidak dihapus.**

### 4. Pure Shape mode

Inline `style` yang berisi presentation attributes yang aman akan dikonversi kembali menjadi atribut presentation sebelum `style` dibuang. Metadata non-geometry seperti `<title>` dan `<desc>` juga dibersihkan ketika mode ini aktif.

### 5. Redundant wrapper cleanup

Wrapper `<g>` yang benar-benar redundant dapat di-collapse. Namun, sebuah grouping yang sudah memiliki **2 atau lebih visual objects** diperlakukan sebagai grouping bermakna dan dipertahankan.

### 6. Root-level Unwrap / Ungroup

Mode **Unwrap / ungroup to root** menormalkan hierarchy menjadi maksimal satu level group di bawah `<svg>`.

Aturan utamanya:

- 1 visual object dalam sebuah branch → object dilepas langsung ke `<svg>`.
- 2+ visual objects dalam sebuah grouping → grouping dipertahankan sebagai satu root-level `<g>`.
- nested `<g>` tidak dibiarkan bertingkat di dalam `<g>` root.
- object yang tetap grouped boleh memiliki beberapa shape di dalam satu `<g>`.
- panel **Unwrap / Ungroup** menampilkan kondisi **Before** dan **After**.

### 7. Collapsible group controls

Setiap root-level `<g>` pada hasil After dapat dibuka/tutup melalui panel collapsible. Tombol **Remove `<g>`** menghapus wrapper group tersebut tanpa menghapus object-object di dalamnya.

### 8. SVG version & UTF encoding preservation

Aplikasi mempertahankan informasi format sumber selama proses export:

- **SVG `version`** dipertahankan.
- **XML declaration version** dipertahankan ketika tersedia.
- **UTF-8** dipertahankan.
- **UTF-8 BOM** dipertahankan jika sumber memakai BOM.
- **UTF-16LE / UTF-16BE** dideteksi dari BOM maupun signature byte tanpa BOM.
- File download ditulis kembali menggunakan encoding yang terdeteksi, bukan selalu dipaksa menjadi UTF-8.

Dengan demikian, cleanup tidak otomatis menghilangkan `version` atau mengganti encoding sumber hanya karena markup telah dinormalisasi.

### 9. Preview & inspection

Tersedia beberapa view:

- **Original** — preview SVG sumber.
- **Clean** — preview SVG setelah purification.
- **Unwrap / Ungroup** — perbandingan hierarchy Before/After serta kontrol group.
- **Clean markup** — markup hasil yang dapat dicopy atau didownload.

### 10. Change log & statistics

Inspection menampilkan jumlah element, node yang dibersihkan, atribut yang di-strip, persentase pengurangan ukuran, serta change log berdasarkan rule yang dijalankan.

## Workflow penggunaan

1. Buka `svg-pure-shape.html` di browser modern.
2. Masukkan SVG melalui file picker, drag-and-drop, atau paste markup.
3. Atur cleaning profile sesuai kebutuhan.
4. Klik **Neutralize SVG**.
5. Periksa **Original**, **Clean**, dan **Unwrap / Ungroup**.
6. Gunakan kontrol **Remove `<g>`** pada group tertentu bila grouping tersebut memang ingin dilepas.
7. Copy atau download clean SVG.

## Keyboard shortcut

- `Ctrl + Enter` pada Windows/Linux
- `⌘ + Enter` pada macOS

Shortcut tersebut menjalankan proses **Neutralize SVG**.

## Struktur repository

```text
svg-pure-shape/
├── svg-pure-shape.html
├── README.md
└── .gitignore
```

Aplikasi tidak memerlukan build step, package manager, framework, atau backend.

## Menjalankan repository secara lokal

Cukup buka `svg-pure-shape.html` dengan browser modern.

Untuk workflow repository/static-hosting, file HTML yang sama dapat dipublikasikan melalui GitHub Pages atau static hosting lain tanpa proses build khusus.

## Privasi & keamanan

Pemrosesan SVG dilakukan di sisi client/browser. File tidak perlu dikirim ke server aplikasi. Aplikasi juga menggunakan DOM/XML parsing dan tidak mencoba menonaktifkan DRM, signature kriptografis, atau mekanisme kontrol akses eksternal di luar markup SVG.

## Catatan kompatibilitas

SVG adalah format yang dapat membawa fitur kompleks seperti `<defs>`, clipping, masking, filter, gradients, markers, embedded references, dan style inheritance. Karena itu, cleaning profile sebaiknya dipahami sebagai normalisasi/sanitasi struktur, bukan sebagai jaminan bahwa setiap SVG kompleks akan identik secara byte-for-byte dengan sumber.

Versi output mempertahankan format penting sumber, tetapi serialisasi XML dapat berubah karena browser `DOMParser` dan `XMLSerializer` melakukan normalisasi markup.

## Lisensi

Belum menetapkan lisensi distribusi khusus di repository ini. Tambahkan file `LICENSE` dengan lisensi pilihan Anda sebelum mendistribusikan source code secara formal.
