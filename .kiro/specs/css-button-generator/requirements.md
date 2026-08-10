# Requirements Document

## Introduction

CSS Button Generator adalah sebuah web tool berbasis browser yang memungkinkan developer dan designer untuk membuat dan mengkustomisasi button secara visual tanpa menulis CSS secara manual. User dapat mengatur berbagai properti button melalui control panel interaktif, melihat hasil perubahan secara real-time di preview panel, lalu menyalin generated CSS dan HTML ke clipboard. Aplikasi ini berjalan sepenuhnya di sisi klien (client-side) tanpa backend atau database.

## Glossary

- **Generator**: Aplikasi web CSS Button Generator secara keseluruhan
- **Control_Panel**: Panel di sisi kiri/atas yang berisi semua input pengaturan button
- **Preview_Panel**: Panel di sisi kanan/bawah yang menampilkan button hasil kustomisasi
- **Live_Button**: Elemen button yang ditampilkan di Preview_Panel secara real-time
- **Code_Output**: Area yang menampilkan generated CSS dan HTML di bawah Preview_Panel
- **Preset**: Konfigurasi button yang sudah ditentukan sebelumnya dan dapat diterapkan sekaligus
- **Randomizer**: Fitur yang mengacak semua nilai properti button
- **Clipboard**: Papan klip sistem operasi tempat teks disalin

## Requirements

---

### Requirement 1: Layout dan Struktur Halaman

**User Story:** Sebagai seorang developer, saya ingin melihat layout halaman yang jelas dengan panel kontrol dan panel preview terpisah, sehingga saya dapat dengan mudah mengatur button dan langsung melihat hasilnya.

#### Acceptance Criteria

1. THE Generator SHALL menampilkan layout dua-panel dengan Control_Panel di sisi kiri (lebar 280px–400px) dan Preview_Panel mengisi sisa ruang di sisi kanan pada viewport lebar ≥ 1024px.
2. THE Generator SHALL menampilkan layout single-column dengan Control_Panel dan Preview_Panel masing-masing mengambil 100% lebar viewport pada viewport lebar < 1024px.
3. WHEN lebar viewport melewati breakpoint 1024px (baik mengecil maupun membesar), THE Generator SHALL beralih tata letak antara dua-panel dan single-column tanpa memerlukan reload halaman.
4. THE Generator SHALL menggunakan Tailwind CSS sebagai satu-satunya CSS framework.
5. THE Generator SHALL beroperasi sepenuhnya di browser tanpa memerlukan koneksi backend atau database.
6. THE Generator SHALL diimplementasikan hanya menggunakan HTML, Tailwind CSS, dan Vanilla JavaScript.

---

### Requirement 2: Control Panel — Input Properti Button

**User Story:** Sebagai seorang developer, saya ingin mengatur semua properti visual button melalui input yang intuitif, sehingga saya tidak perlu menulis CSS secara manual.

#### Acceptance Criteria

1. THE Control_Panel SHALL menyediakan input teks untuk mengatur teks yang ditampilkan pada Live_Button, dengan panjang maksimal 100 karakter.
2. THE Control_Panel SHALL menyediakan slider untuk mengatur font size dengan rentang nilai 10px hingga 48px.
3. THE Control_Panel SHALL menyediakan dropdown untuk mengatur font weight dengan pilihan: 300, 400, 500, 600, 700, 800.
4. THE Control_Panel SHALL menyediakan color picker untuk mengatur warna teks (text color) button.
5. THE Control_Panel SHALL menyediakan color picker untuk mengatur warna latar belakang (background color) button.
6. THE Control_Panel SHALL menyediakan color picker untuk mengatur warna border button.
7. THE Control_Panel SHALL menyediakan slider untuk mengatur border width dengan rentang nilai 0px hingga 10px.
8. THE Control_Panel SHALL menyediakan slider untuk mengatur border radius dengan rentang nilai 0px hingga 50px.
9. THE Control_Panel SHALL menyediakan slider untuk mengatur padding horizontal dengan rentang nilai 0px hingga 80px.
10. THE Control_Panel SHALL menyediakan slider untuk mengatur padding vertikal dengan rentang nilai 0px hingga 60px.
11. THE Control_Panel SHALL menyediakan dropdown untuk mengatur box shadow dengan pilihan: None, Small, Medium, Large, Custom; IF pilihan Custom dipilih, THEN THE Control_Panel SHALL menampilkan input tambahan untuk mengatur offset-x, offset-y, blur radius, spread radius, dan warna shadow secara manual.
12. THE Control_Panel SHALL menyediakan dropdown untuk mengatur hover effect dengan pilihan: None, Darken (15%), Lighten (15%), Scale Up (skala 1.05), Scale Down (skala 0.95), Lift (shadow meningkat satu tingkat dari shadow aktif).
13. WHEN nilai slider diubah, THE Control_Panel SHALL menampilkan nilai numerik saat ini di sebelah slider.
14. WHEN nilai properti apa pun pada Control_Panel diubah, THE Control_Panel SHALL memperbarui tampilan Live_Button secara langsung tanpa memerlukan tindakan konfirmasi tambahan dari pengguna.

---

### Requirement 3: Live Preview Real-Time

**User Story:** Sebagai seorang developer, saya ingin melihat perubahan pada button secara instan ketika saya mengubah setiap pengaturan, sehingga saya dapat bereksperimen dengan cepat tanpa harus menekan tombol "Apply".

#### Acceptance Criteria

1. WHEN nilai input manapun di Control_Panel berubah, THE Preview_Panel SHALL memperbarui tampilan Live_Button dalam waktu kurang dari 100ms setelah perubahan terakhir terdeteksi.
2. THE Preview_Panel SHALL menampilkan Live_Button di tengah area preview secara vertikal dan horizontal dengan toleransi penyimpangan ≤ 2px dari titik tengah.
3. THE Preview_Panel SHALL menampilkan Live_Button dengan latar belakang area preview berwarna abu-abu netral (tidak pure hitam atau putih) sehingga Live_Button memiliki kontras yang dapat dibedakan secara visual terhadap latar belakang tersebut.
4. WHEN hover effect dipilih selain "None", THE Live_Button SHALL menerapkan CSS transition dengan durasi antara 100ms hingga 500ms pada properti yang berubah saat pointer masuk dan keluar dari area button.
5. IF kombinasi properti yang dimasukkan menghasilkan nilai CSS yang tidak valid, THEN THE Preview_Panel SHALL mempertahankan tampilan Live_Button pada state terakhir yang valid tanpa menampilkan error visual pada Live_Button.
6. THE Preview_Panel SHALL menampilkan Live_Button dengan nilai default pada saat halaman pertama kali dimuat, sebelum pengguna melakukan interaksi apapun.

---

### Requirement 4: Generated Code Output

**User Story:** Sebagai seorang developer, saya ingin melihat kode CSS dan HTML yang dihasilkan secara otomatis berdasarkan pengaturan yang saya buat, sehingga saya dapat langsung menggunakannya di proyek saya.

#### Acceptance Criteria

1. THE Code_Output SHALL menampilkan blok HTML yang berisi elemen `<button>` dengan class `custom-button` dan teks button yang sesuai dengan nilai teks yang dikonfigurasi di Control_Panel.
2. THE Code_Output SHALL menampilkan blok CSS yang berisi semua properti yang dikonfigurasi untuk selector `.custom-button`, mencakup minimal properti: background-color, color, font-size, padding, border-radius, dan border.
3. WHEN nilai input manapun di Control_Panel berubah, THE Code_Output SHALL memperbarui seluruh konten kode HTML dan CSS yang ditampilkan dalam waktu tidak lebih dari 100ms setelah perubahan nilai input, selaras dengan pembaruan Live_Button.
4. WHEN hover effect yang dipilih bukan "None", THE Code_Output SHALL menampilkan blok CSS tambahan dengan selector `.custom-button:hover` yang memuat properti-properti yang berubah saat hover.
5. IF hover effect yang dipilih adalah "None", THEN THE Code_Output SHALL tidak menampilkan blok CSS dengan selector `.custom-button:hover`.
6. THE Code_Output SHALL memformat kode dengan indentasi 2 spasi per level dan setiap deklarasi properti CSS pada baris terpisah.
7. THE Code_Output SHALL menampilkan kode menggunakan elemen dengan font monospace sehingga karakter memiliki lebar seragam dan struktur kode dapat dibaca dengan jelas.

---

### Requirement 5: Salin ke Clipboard

**User Story:** Sebagai seorang developer, saya ingin menyalin generated CSS dan HTML dengan satu klik, sehingga saya dapat langsung menggunakannya tanpa harus memilih teks secara manual.

#### Acceptance Criteria

1. THE Code_Output SHALL menyediakan tombol "Copy CSS" yang menyalin seluruh konten CSS ke Clipboard.
2. THE Code_Output SHALL menyediakan tombol "Copy HTML" yang menyalin seluruh konten HTML ke Clipboard.
3. WHEN tombol "Copy CSS" diklik, THE Generator SHALL menyalin teks CSS ke Clipboard dan mengubah label tombol menjadi "Copied!" selama 2000ms sebelum kembali ke "Copy CSS".
4. WHEN tombol "Copy HTML" diklik, THE Generator SHALL menyalin teks HTML ke Clipboard dan mengubah label tombol menjadi "Copied!" selama 2000ms sebelum kembali ke "Copy HTML".
5. IF Clipboard API tidak tersedia di browser, THEN THE Generator SHALL menampilkan pesan error yang menjelaskan bahwa fungsi salin tidak didukung di browser ini dan menyarankan user untuk menyalin teks secara manual.
6. IF operasi salin ke Clipboard gagal meskipun Clipboard API tersedia, THEN THE Generator SHALL menampilkan pesan error yang menjelaskan bahwa operasi salin gagal, dan label tombol TIDAK berubah menjadi "Copied!".
7. WHILE tidak ada konten CSS atau HTML yang ter-generate, THE Code_Output SHALL menonaktifkan tombol "Copy CSS" dan tombol "Copy HTML" sehingga tidak dapat diklik.

---

### Requirement 6: Preset Button

**User Story:** Sebagai seorang developer, saya ingin memilih dari beberapa preset button yang sudah tersedia, sehingga saya dapat dengan cepat memulai dari desain yang sudah jadi dan memodifikasinya sesuai kebutuhan.

#### Acceptance Criteria

1. THE Generator SHALL menyediakan tepat 5 preset button dengan nama deskriptif: Primary, Outline, Ghost, Pill, dan Danger; setiap preset SHALL berbeda dari preset lainnya pada setidaknya satu properti.
2. WHEN sebuah preset dipilih, THE Control_Panel SHALL memperbarui semua nilai input (text, font-size, font-weight, text color, background color, border color, border width, border radius, padding horizontal, padding vertical, box shadow, hover effect) sesuai konfigurasi preset tersebut dalam waktu tidak lebih dari 100ms.
3. WHEN sebuah preset dipilih, THE Preview_Panel SHALL memperbarui tampilan Live_Button secara real-time sesuai preset dalam waktu tidak lebih dari 100ms tanpa reload halaman.
4. THE Generator SHALL menampilkan 5 kartu atau tombol preset yang seluruhnya terlihat (visible) di viewport tanpa perlu melakukan scroll, pada perangkat dengan lebar layar ≥ 768px saat halaman pertama kali dimuat.

---

### Requirement 7: Reset Pengaturan

**User Story:** Sebagai seorang developer, saya ingin mereset semua pengaturan ke nilai default dengan satu klik, sehingga saya dapat memulai dari awal dengan cepat.

#### Acceptance Criteria

1. THE Generator SHALL menyediakan tombol "Reset" yang terlihat di Control_Panel atau area aksi tanpa tersembunyi di balik menu atau dropdown.
2. WHEN tombol "Reset" diklik, THE Control_Panel SHALL mengembalikan semua nilai input ke nilai default awal dalam waktu tidak lebih dari 500ms tanpa reload halaman.
3. WHEN tombol "Reset" diklik dan Control_Panel telah selesai mereset, THE Preview_Panel SHALL memperbarui tampilan Live_Button sesuai nilai default.
4. WHEN tombol "Reset" diklik dan Control_Panel telah selesai mereset, THE Code_Output SHALL memperbarui kode yang ditampilkan sesuai nilai default.
5. THE Generator SHALL mendefinisikan "nilai default awal" sebagai konfigurasi button yang identik dengan state yang ditampilkan saat halaman pertama kali dimuat, bukan state dari session atau preset terakhir yang dipilih.

---

### Requirement 8: Randomize Button

**User Story:** Sebagai seorang developer, saya ingin mengacak semua properti button secara otomatis, sehingga saya dapat mendapatkan inspirasi desain baru dengan cepat.

#### Acceptance Criteria

1. THE Generator SHALL menyediakan tombol "Randomize" yang terlihat jelas di Control_Panel atau area aksi.
2. WHEN tombol "Randomize" diklik, THE Generator SHALL menghasilkan nilai acak yang valid untuk setiap properti button dalam rentang berikut: font-size antara 12px hingga 32px, border-radius antara 0px hingga 50px, border-width antara 0px hingga 8px, padding-horizontal antara 8px hingga 64px, padding-vertical antara 4px hingga 32px, dan font-weight dari nilai standar CSS (400, 500, 600, 700).
3. WHEN tombol "Randomize" diklik, THE Control_Panel SHALL memperbarui semua nilai input yang ditampilkan kepada pengguna dengan nilai acak yang dihasilkan dalam waktu tidak lebih dari 300ms setelah klik.
4. WHEN tombol "Randomize" diklik, THE Preview_Panel SHALL memperbarui tampilan Live_Button dalam waktu tidak lebih dari 300ms setelah klik sehingga Live_Button mencerminkan seluruh nilai acak yang baru dihasilkan.
5. WHEN tombol "Randomize" diklik, THE Generator SHALL menghasilkan kombinasi text color dan background color yang memiliki rasio kontras ≥ 4.5:1 sesuai standar WCAG AA.
6. IF kombinasi warna acak yang dihasilkan memiliki rasio kontras < 4.5:1, THEN THE Generator SHALL menghasilkan ulang kombinasi warna baru hingga rasio kontras ≥ 4.5:1 tercapai, dengan batas maksimal 10 kali percobaan ulang sebelum menggunakan pasangan warna fallback yang telah memenuhi syarat kontras.

---

### Requirement 9: Visual Design dan Aksesibilitas

**User Story:** Sebagai seorang developer, saya ingin menggunakan tool yang memiliki tampilan minimal, clean, dan modern, sehingga pengalaman menggunakannya menyenangkan dan tidak mengalihkan perhatian dari button yang sedang dibuat.

#### Acceptance Criteria

1. THE Generator SHALL menerapkan desain visual yang minimal dengan dominasi warna netral (putih, abu-abu, hitam) dan whitespace antar elemen UI minimal 8px.
2. THE Generator SHALL menggunakan tipografi yang konsisten dengan hierarki visual yang dapat dibedakan secara ukuran: heading minimal 1.25x ukuran label, dan label minimal 1x ukuran nilai.
3. THE Generator SHALL memastikan semua elemen interaktif di Control_Panel dapat diakses melalui keyboard dengan urutan fokus tab mengikuti urutan visual dari atas ke bawah dan kiri ke kanan.
4. THE Generator SHALL memberikan label teks yang deskriptif untuk setiap elemen input di Control_Panel sehingga setiap input dapat diidentifikasi secara unik oleh screen reader tanpa bergantung pada posisi visual.
5. WHILE ukuran viewport berubah antara desktop (lebar ≥ 1024px) dan mobile (lebar < 768px), THE Generator SHALL mempertahankan semua elemen UI dan fungsionalitas Control_Panel tanpa ada elemen yang terpotong atau tersembunyi.
6. WHEN pengguna berinteraksi dengan elemen interaktif di Control_Panel menggunakan keyboard, THE Generator SHALL menampilkan indikator fokus yang terlihat dengan kontras minimum 3:1 terhadap warna latar belakang sekitarnya.
7. IF ukuran viewport berada di antara 768px dan 1023px, THEN THE Generator SHALL menyesuaikan tata letak Control_Panel sehingga semua elemen input tetap dapat dijangkau tanpa horizontal scrolling.
