# PRD Platform SaaS SMK — Dedicated VPS per Sekolah

**Product Requirements Document (PRD) · Versi 4.7 · Revisi keputusan operasional**
**Ruang lingkup:** Core System + 15 modul operasional SMK.  
**Arsitektur:** satu sekolah = satu VPS = satu instalasi Laravel + MariaDB.  
**Tujuan utama:** aplikasi praktis; perpindahan siswa, kelulusan, alumni, dan keuangan selalu konsisten.

| **Dokumen** | **Keterangan**                                                       |
|-------------|----------------------------------------------------------------------|
| Versi       | 4.7 — pengembangan aplikasi saja; konfigurasi infrastruktur tidak termasuk pekerjaan developer |
| Tanggal     | Revisi keputusan operasional                                                      |
| Status      | Draf implementasi MVP; persetujuan alur operasional sekolah diperlukan                   |
| Target      | SMK di Indonesia                                                     |
| Stack       | Laravel 13 · Livewire 4 · Tailwind CSS 4 · Nginx · PHP 8.5 · MariaDB |
| Hosting     | 1 sekolah = 1 VPS = 1 instalasi dan database                         |
| Batasan     | Tanpa server/domain pusat, multi-tenant, Queue, dan Scheduler        |

**Prinsip produk.** Praktis bagi pengguna sekolah: input seperlunya, validasi hanya saat dibutuhkan, data induk tidak berulang, laporan mudah dicetak, dan **setiap proses yang lama menampilkan progres nyata**.

**Keputusan final.** Penilaian PKL ditranskripsi guru dari kertas; absensi guru dihapus; izin guru/siswa digabung dalam modul absensi siswa; surat boleh tanpa file; naik kelas dan lulus harus melalui konfirmasi admin; lulusan yang sudah tercatat memakai identitas siswa lama, sedangkan alumni lama dapat daftar sendiri tanpa dibuatkan siswa fiktif; Sarpras–Lab memakai satu master aset; menu website dikelola secara dinamis.

**Definisi MVP.** Utamakan alur pokok, formulir singkat, hak akses jelas, modal konfirmasi yang konsisten, loading yang informatif, serta proses massal dengan progres dan pemulihan. **Spesifikasi teknis dan kriteria rilis ada pada Bab 12–18**. Tidak ada Queue, Laravel Scheduler, aplikasi pusat, panel multi-sekolah, atau integrasi pemerintah pada versi ini.

# 1. Ringkasan keputusan produk

**Alur bagian:** Tetapkan Keputusan → Susun Aturan → Terapkan pada Fitur → Validasi Hasil.

| **Keputusan**  | **Ketentuan final**                                                                            |
|----------------|------------------------------------------------------------------------------------------------|
| Model aplikasi | Instalasi independen per sekolah, kode sumber sama pada seluruh VPS.                           |
| Jumlah modul   | 15 modul berbayar. Perizinan lama digabung ke Absensi; tidak dibuat modul pengganti.           |
| Absensi        | Hanya siswa. Pengajuan izin guru ada, tanpa pencatatan kehadiran guru.                         |
| PKL industri   | Tidak ada akun pembimbing industri. Nilai kertas diinput oleh guru pembimbing.                 |
| Tahun ajaran   | Kenaikan kelas dan kelulusan hanya melalui aksi admin, bukan otomatis karena tanggal berganti. |
| Data historis  | ID siswa tetap untuk lulusan yang pernah terdaftar; alumni lama dapat punya profil tanpa `siswa_id`. Riwayat kelas, tagihan, PKL dan alumni tidak ditimpa. |
| Sarpras + Lab  | Satu daftar aset, dengan stok bahan habis pakai dikelola Lab.                                  |
| Surat          | Boleh pencatatan saja, editor surat, atau upload; berkas opsional.                             |
| Website        | CMS mengelola menu, halaman, submenu, urutan dan tautan dinamis.                               |
| Di luar scope  | PPDB, UKK, absensi guru, Queue, Scheduler, billing/pembayaran gateway otomatis.                   |
| Proses massal | Batch kecil paralel terbatas dari browser; progres disimpan ke database, aman dilanjutkan tanpa duplikasi. |
| UI responsif | Loading jelas, progress bar nyata, modal konfirmasi, hasil sukses/gagal per item, pesan galat mudah dipahami. |

# 2. Tujuan, pengguna, dan cakupan

**Alur bagian:** Kenali Pengguna → Pilih Kebutuhan → Tetapkan Cakupan → Uji Alur Sekolah.

- Membantu operasional SMK dalam satu sistem, tanpa harus membeli semua modul.

- Setiap sekolah hanya mengakses modul berlisensi; core sekolah tersedia selalu.

- Setiap proses punya penanggung jawab, langkah minimal, status jelas, dan keluaran laporan.

- Mudah digunakan melalui desktop dan ponsel, tanpa kewajiban akun eksternal yang tidak diperlukan.

## 2.1 Daftar modul final (15 modul)

| **No.** | **Modul**                  | **Perubahan**                                       |
|---------|----------------------------|-----------------------------------------------------|
| 01      | Manajemen PKL              | Industri memberi nilai pada kertas; input oleh guru |
| 02      | Jurnal Kegiatan Harian Guru       | Tetap                                               |
| 03      | Absensi Siswa & Perizinan  | Absensi guru dihapus; perizinan siswa/guru digabung |
| 04      | Administrasi Surat Digital | Lampiran/file opsional                              |
| 05      | Keuangan & SPP             | Riwayat mengikuti siswa sampai alumni               |
| 06      | BKK & Lowongan Kerja       | Pendaftaran alumni mandiri dan profil karier         |
| 07      | Inventaris & Sarpras       | Satu master aset bersama Lab                        |
| 08      | Bimbingan Konseling        | Tetap; akses sensitif dibatasi                      |
| 09      | Jadwal Pelajaran           | Tetap                                               |
| 10      | Tracer Study Alumni        | Profil alumni lama/baru, riwayat aktivitas sepanjang waktu |
| 11      | HUBIN / DUDI               | Data mitra bisa dipakai PKL                         |
| 12      | Supervisi & Kinerja Guru   | Tetap                                               |
| 13      | Laboratorium & Bengkel     | Gunakan aset bersama Sarpras                        |
| 14      | Poin Pelanggaran Siswa     | Sebelumnya modul 15                                 |
| 15      | Website Sekolah + CMS      | Sebelumnya modul 16; menu dinamis                   |

**Catatan nomor modul.** Nomor 14 yang dahulu dipakai Perizinan tidak dipertahankan; seluruh modul setelahnya dinomori ulang. Kode modul harus ditetapkan stabil sejak awal implementasi agar aktivasi lisensi tidak bergantung pada nomor tampilan.

# 3. Arsitektur dan prinsip antarmuka

**Alur bagian:** Pilih Instalasi → Terapkan Konfigurasi Aplikasi → Bangun Antarmuka → Uji Akses.

| **Area**           | **Ketentuan**                                                                                              |
|--------------------|------------------------------------------------------------------------------------------------------------|
| Stack              | Laravel 13, PHP 8.5 FPM, Livewire 4, Blade, Tailwind CSS 4, Nginx, MariaDB.                                |
| Instalasi          | 1 VPS milik 1 sekolah; 1 Laravel; 1 MariaDB; domain .sch.id sekolah.                                       |
| Panel pemilik SaaS | /owner pada instalasi sekolah, dengan autentikasi dan MFA wajib sebelum rilis produksi.                                 |
| Panel operasional  | /admin; tampilan role dan aktivasi modul membatasi menu maupun API/aksi.                                   |
| Lisensi            | Lokal pada database aplikasi. Tanggal kedaluwarsa diperiksa saat membuka/menjalankan modul.                |
| Proses             | Form biasa sinkron; massal via batch kecil dan HTTP request paralel terbatas dari browser, tanpa Queue/Scheduler. |
| File               | Disimpan privat bila memuat data sekolah; validasi tipe dan ukuran file.                                   |
| Backup             | Backup VPS/database dilakukan di level server secara manual/alat infrastruktur; di luar scheduler Laravel. |

**UI standar.** Tiap modul memakai pola: Dashboard ringkas → Daftar → Tambah/Ubah → Detail → Laporan. Status proses ditampilkan dengan bahasa yang mudah dipahami; hindari menu duplikat.

## 3.1 Standar loading, modal, dan pemberitahuan (wajib seluruh modul)

| Situasi | Perilaku antarmuka | Catatan implementasi |
|---|---|---|
| Halaman/daftar sedang mengambil data | Skeleton atau indikator loading pada area yang benar | Jangan menutup seluruh halaman jika hanya tabel yang diperbarui. |
| Tombol Simpan/Kirim/Setujui diproses | Tombol menampilkan spinner + teks **Menyimpan…/Mengirim…**, sementara dinonaktifkan | Mencegah klik berulang; form tetap terbaca. |
| Konfirmasi tindakan berisiko | **Modal konfirmasi** berisi aksi, objek, jumlah data, dampak, tombol **Batal** dan **Lanjutkan** | Dipakai untuk kenaikan/kelulusan massal, perubahan status, penghapusan, pembayaran, dll. |
| Ada kesalahan validasi | Pesan di bawah field; ringkasan di bagian atas form bila perlu | Input pengguna jangan hilang. |
| Error sistem/koneksi | **Modal informasi** dengan alasan mudah dipahami dan tombol **Coba Lagi/Tutup** | Hindari pesan SQL atau stack trace. Jangan menyebut gagal sebelum memeriksa status yang tersimpan. |
| Tindakan ringan sukses | Toast singkat **Berhasil disimpan** | Tidak perlu modal untuk setiap perubahan kecil. |
| Proses massal selesai/parsial | **Modal ringkasan hasil**: berhasil, sudah pernah diproses, gagal, dilewati | Tautan lihat detail, perbaiki dan ulangi yang gagal. |
| Proses massal sedang berlangsung | **Modal progres**: tahap, progress bar %, hitungan item, status, tombol **Jeda** | Tidak dapat ditutup tanpa peringatan; tersedia **Lihat Proses** jika pengguna berpindah halaman. |
| Sesi habis saat memproses | Modal **Sesi berakhir**, arahkan login ulang, lalu lanjutkan proses yang belum selesai | Hasil batch yang sudah commit tidak diulang. |

**Aturan UI:** tidak menggunakan `window.alert()` atau `window.confirm()` bawaan browser; gunakan komponen modal konsisten dengan akses keyboard, fokus yang benar, label tombol jelas, dan warna status tidak menjadi satu-satunya indikator. **Progress bar dilarang menggunakan animasi palsu**: persen harus bersumber dari jumlah item yang sudah ditandai selesai dalam database. Pada operasi biasa yang durasinya tidak bisa diukur, tampilkan loading tak tentu (*indeterminate*), bukan persentase rekaan.


## 3.2 Batas tugas developer aplikasi (FINAL)

**Dalam scope developer:** pembuatan source code Laravel, migration/seeder MariaDB, Blade/Livewire/Tailwind, routing aplikasi, autentikasi dan otorisasi, pengamanan data di tingkat aplikasi, interaksi pengguna, pemrosesan massal HTTP paralel, feature/integration tests, perbaikan bug, dan dokumentasi antarmuka konfigurasi yang dibutuhkan untuk menjalankan aplikasi. Developer menyerahkan instruksi build/migrate/rollback secara operasional **tanpa menjalankan konfigurasi host**.

**Di luar scope developer:** penyediaan VPS, domain/DNS, konfigurasi Nginx, SSL/TLS, firewall/SSH/jaringan, monitoring host, backup dan pemulihan infrastruktur. Tim infrastruktur menerima daftar prasyarat dari developer dan menanggung penerapannya. **Kesiapan aplikasi** tidak identik dengan **kesiapan go-live end-to-end**: rilis nyata memerlukan persetujuan kedua pihak.

**Aturan aplikasi:** jangan hardcode domain atau IP, jangan simpan rahasia pada repository, sediakan `.env.example` tanpa kredensial, route memakai nama, dan uji pada lingkungan pengembangan/staging yang disediakan. Implementasi fitur tidak boleh bergantung pada Laravel Queue maupun Scheduler.

**Acuan dokumen:** `TASK.md` (urutan eksekusi), bagian Panduan implementasi (pedoman asisten coding), bagian Checklist kesiapan rilis aplikasi (gerbang kualitas dan bukti UAT), `prototype/index.html` (referensi antarmuka, bukan implementasi backend).

## 3.3 Bahasa aplikasi dan susunan kode (wajib)

- Seluruh antarmuka menggunakan **Bahasa Indonesia**: menu, tombol, label, placeholder, validasi, autentikasi/reset sandi, modal, notifikasi, status, pesan galat, laporan, cetakan, dan teks aksesibilitas. Halaman HTML memakai `lang="id"`; locale Laravel dan format tanggal/angka memakai Indonesia, mata uang Rupiah. Pesan bawaan Laravel/Livewire/paket harus diterjemahkan sebelum fitur dinyatakan selesai.
- Fokus implementasi fitur pada `routes/`, `app/Http/Controllers/`, `app/Livewire/`, `app/Models/`, `app/Http/Middleware/`, dan `resources/`. Gunakan struktur Laravel standar; kelompokkan per fitur seperlunya tanpa membuat paket modul, lapisan repository/service generik, atau direktori arsitektur tambahan.
- Route hanya memetakan URL dan middleware; Controller menangani alur HTTP; Livewire menangani interaksi, validasi dan status layar; Model menangani relasi, query bisnis yang berulang, serta transaksi data; Middleware menangani autentikasi, peran dan lisensi lintas request; `resources/` berisi Blade, CSS/JS dan teks antarmuka. Controller/Livewire tetap ringkas; jangan salin aturan bisnis di dua tempat.
- Pemeriksaan hak akses objek wajib berlaku pada setiap aksi server. Gunakan fasilitas otorisasi Laravel dari Controller/Livewire dan penugasan pada Model; middleware saja tidak cukup untuk keamanan per objek. Jangan menambah lapisan Policy terpisah sebagai pola wajib di luar fokus folder tersebut.
- File pendukung di luar fokus diubah **sebatas kebutuhan nyata**: `database/migrations`/`seeders` untuk skema dan data awal, `tests/` untuk membuktikan perilaku, `lang/id/` untuk pesan bawaan framework, konfigurasi/bootstrap dan manifest untuk menjalankan Laravel. Ini pendukung, bukan perluasan fitur atau pekerjaan infrastruktur.
- Kode bersih dan rapi: nama kelas/metode/variabel bermakna, metode singkat dengan satu tujuan, tipe parameter/return bila sesuai, validasi server, hindari duplikasi dan query N+1, serta komentar hanya untuk alasan bisnis yang tidak langsung terlihat. Ikuti format Laravel/Pint setelah aplikasi tersedia. Jangan membuat abstraksi sebelum ada kebutuhan nyata.

# 4. Core system dan role

**Alur bagian:** Siapkan Master → Buat Akun → Tetapkan Penugasan → Tampilkan Menu Sesuai Hak.

## 4.1 Fitur sistem inti

| **Fitur**            | **Subfitur wajib**                                                  |
|----------------------|---------------------------------------------------------------------|
| Akun                 | Login, logout, reset kata sandi, aktivasi/nonaktif akun.            |
| Profil Sekolah       | Identitas sekolah, NPSN, logo, alamat, kontak, kepala sekolah.      |
| Master Akademik      | Tahun ajaran, semester, jurusan/program (3 atau 4 tahun), kelas/rombel, mata pelajaran. |
| Siswa                | Identitas tetap, NIS/NISN, wali, status, riwayat kelas/tahun ajaran, impor/ekspor Excel. |
| Guru & Staf          | Identitas, jabatan, mata pelajaran, kontak.                         |
| Role/permission      | Role bisa rangkap; izin lihat/tambah/ubah/hapus/setujui bila perlu. |
| Kenaikan & Kelulusan | Proses massal dengan pratinjau, pengecualian, modal konfirmasi, progress real-time, dan lanjutkan proses. |
| Riwayat              | Riwayat kelas/status siswa, perubahan penting, audit dasar.         |

## 4.2 Role dan tugasnya

Gunakan **13 role inti** berikut. Jabatan seperti Waka Kurikulum/Kesiswaan/HUBIN/Sarpras, wali kelas, guru piket, pembimbing/koordinator PKL, supervisor, serta kepala/petugas lab adalah **penugasan dan hak akses**, bukan role tambahan. Satu akun bisa mendapat beberapa penugasan; lisensi modul dan lingkup objek tetap diperiksa. Pengunjung publik tidak memerlukan role/akun.

| Role | Tugas utama | Batas akses dan penugasan |
|---|---|---|
| Owner SaaS | Kelola lisensi, masa berlaku, pembayaran langganan dan versi instalasi. | MFA wajib; tidak otomatis membaca data pribadi sekolah. |
| Admin Sekolah | Kelola profil, master akademik, akun/peran operasional, impor, kenaikan dan kelulusan. | Tidak dapat memberikan role owner atau membuka isi konseling otomatis. |
| Pimpinan Sekolah | Kepala Sekolah/Waka memantau laporan dan memberi persetujuan sesuai bidang. | Bidang kurikulum, kesiswaan, HUBIN atau sarpras merupakan penugasan; laporan privat tetap dibatasi. |
| Guru | Catat presensi siswa per jam mapel, wajib isi jurnal kegiatan dan hasil kerja harian, izin diri dan pembinaan sesuai tugas. | Hanya mapel/kelas/siswa ditugaskan; wali kelas mengonfirmasi presensi kelas binaan. Pembimbing PKL/supervisor adalah penugasan, bukan role baru. |
| Tata Usaha | Kelola surat, templat, penomoran dan arsip; bantu administrasi yang diberi izin. | Surat tertentu memerlukan pejabat penandatangan; tidak mendapat akses keuangan/BK otomatis. |
| Bendahara | Kelola tarif, tagihan, cicilan, kuitansi, tunggakan dan koreksi beralasan. | Koreksi/void memerlukan hak khusus dan audit; tidak menghapus pembayaran sah. |
| Pengelola Alumni | Verifikasi alumni, kelola lowongan/lamaran BKK, periode/form tracer dan laporan alumni. | Satu role untuk BKK dan Tracer; fitur tetap mengikuti lisensi masing-masing, registrasi bila salah satunya aktif. |
| Pengelola Sarana | Kelola aset, ruang, peminjaman dan stok lab sesuai unit penugasan. | Sarpras dan Lab satu role dengan izin fitur/lokasi terpisah; master aset tetap tunggal. |
| Guru BK | Bersama wali kelas menangani masalah presensi, rujukan dan pembinaan; kelola konseling privat sesuai izin. | Hanya siswa/kasus ditugaskan; isi konseling tidak ikut laporan presensi bersama. |
| Editor Website | Kelola konten, media, menu/submenu, pratinjau, terbit dan arsip. | Website aktif; publik hanya melihat konten terbit. |
| Siswa | Lihat data sendiri, isi jurnal PKL dan ajukan izin/layanan sesuai fitur aktif. | Tidak mengubah presensi guru, konfirmasi wali kelas, nilai atau persetujuan. |
| Alumni | Kelola profil/riwayat sendiri, melamar dan mengisi tracer setelah verifikasi. | Hanya data sendiri dan fitur berlisensi aktif. |
| Orang Tua / Wali Siswa | Sampaikan izin anak melalui jalur sekolah, termasuk di luar aplikasi; pantau data anak atau ajukan izin online bila akun diaktifkan. | Akun opsional; hubungan wali–anak diverifikasi; bukan akses isi konseling privat. |

**Pemetaan jabatan:** Kepala Sekolah/Waka memakai role Pimpinan Sekolah; wali kelas, guru mapel/praktik/piket, pembimbing PKL dan supervisor guru memakai role Guru dengan penugasan terkait. Koordinator PKL/Staf HUBIN memakai Guru atau Tata Usaha sesuai pegawainya dan hak modul yang ditugaskan. Kepala/Petugas Lab dan Petugas Sarpras memakai Pengelola Sarana. Sebutan jabatan di Bab 7/13 menjelaskan penugasan pada role inti, bukan daftar role baru.

### 4.2.1 Penetapan peran dan alur persetujuan

- Owner dibuat melalui provisioning aman. Admin Sekolah hanya boleh memberi peran operasional yang diizinkan, tidak boleh memberikan role owner atau memperluas izinnya sendiri.
- Penugasan menyebut lingkup kelas, siswa bimbingan, kasus, ruang/aset, periode atau bidang. Rangkap jabatan tetap mengikuti lingkup tersebut; perubahan hak akses berlaku pada request berikutnya dan diaudit.
- Izin siswa S/I dicatat wali kelas dan langsung berlaku tanpa persetujuan berlapis. Izin guru diputuskan pejabat berwenang; guru tidak menyetujui izin dirinya sendiri.
- Pelanggaran diperiksa Waka Kesiswaan/petugas validator sebelum poin final. Koreksi pembayaran hanya oleh petugas dengan hak khusus; surat tertentu oleh penandatangan yang ditunjuk. Tidak menambah persetujuan berlapis pada pekerjaan biasa.
- Verifikasi alumni dilakukan Pengelola Alumni; tidak membuka portal tanpa salah satu lisensi tersebut.
- Hak membuat, melihat, mengubah, menyetujui, mengekspor dan mengunduh harus diuji terpisah. Rincian matriks per tindakan/modul pada Bab 13 melengkapi daftar tugas di sini.

## 4.2.2 Model penugasan dan hak akses yang dapat diimplementasikan

**Alur:** Admin Pilih Pengguna → Tetapkan Peran dan Lingkup → Tentukan Masa Berlaku → Simpan serta Audit → Server Periksa Setiap Aksi → Cabut/Ganti bila Tugas Berubah.

| Data | Kolom/aturan minimum | Pemakaian |
|---|---|---|
| `penugasan_mengajar` | `id`, `guru_id`, `kelas_id`, `mata_pelajaran_id`, `berlaku_mulai`, `berlaku_sampai` nullable, `status`, `dibuat_oleh` | Dasar pilihan kelas/mapel presensi; tidak berisi kalender/hari wajib mengajar. Cegah duplikasi lingkup dan rentang aktif sama. |
| `penugasan_wali_kelas` | `id`, `guru_id`, `kelas_id`, rentang berlaku, `status`, aktor | Satu wali utama aktif per kelas pada tanggal yang sama; pengganti lewat delegasi eksplisit. Histori tidak ditimpa saat pergantian. |
| `penugasan_pendampingan` | `id`, `guru_bk_id`, `kelas_id` atau `siswa_id` (tepat satu terisi), rentang berlaku, `status` | Lingkup BK untuk temuan presensi/pembinaan; catatan privat tetap mengikuti izin kasus BK. |
| `penugasan_bidang` | `id`, `pengguna_id`, `bidang` (kurikulum/kesiswaan/hubin/sarpras), rentang berlaku, `status` | Lingkup pimpinan dan laporan bidang. Tidak memberi akses otomatis isi konseling/uang personal. |
| `penugasan_unit` | `id`, `pengguna_id`, `ruang_id`, rentang berlaku, `status` | Batas ruang/aset Lab dan Sarpras. Aset memakai lokasi/ruang master yang sama. |
| `delegasi_tugas` | `id`, `pengguna_asal_id`, `pengguna_pengganti_id`, `jenis_tugas`, referensi penugasan, hak tindakan terbatas, rentang berlaku, `dibuat_oleh`, `alasan`, `status` | Guru pengganti, wali pengganti, koreksi data lampau atau petugas izin pengganti; tidak menggandakan role. Tidak boleh memberikan hak di luar izin pemberi delegasi/admin berwenang. |

- Peran memberi kelompok hak tindakan; tabel penugasan memberi lingkup objek. Server menolak bila pengguna nonaktif, lisensi fitur tidak aktif, hak tindakan tidak ada, atau objek di luar lingkup. `role` saja tidak cukup. Model FK harus nyata dan diuji, bukan ID teks bebas yang menerima semua tabel.
- Untuk PKL, pembimbing/koordinator mengacu periode/penempatan PKL; untuk supervisi mengacu guru yang dinilai pada record supervisi; untuk alumni satu instalasi sekolah, tetap batasi data personal sesuai tindakan.
- Admin mengelola penugasan dengan tanggal efektif tanpa kalender akademik. Tanggal akhir bersifat eksklusif; hanya rentang aktif yang memberi hak baru. Histori penugasan tetap tersimpan dan pencabutan berlaku mulai request berikutnya.
- Konfirmasi/koreksi presensi lampau **tanpa tenggat**, tetapi tetap membutuhkan penugasan kelas saat ini atau delegasi eksplisit untuk data historis kelas tersebut. Guru yang sudah tidak bertugas tidak mendapat akses kekal; wali baru yang diberi kelas boleh menindaklanjuti rekap kelasnya.
- Bila guru sekaligus wali kelas, ia boleh mengonfirmasi kelas binaannya, termasuk sesi yang dia ajar; tidak perlu aktor kedua. Pembatasan persetujuan izin diri berlaku untuk **izin guru**, bukan pencatatan izin siswa.
- Penugasan tidak mengubah bukti pelaku data lama. Audit menyimpan siapa yang mengisi, mengonfirmasi, mengoreksi dan atas delegasi siapa. Uji guru A/B, wali lama/baru, guru pengganti, rentang kedaluwarsa dan manipulasi ID untuk setiap route/Livewire/export/download.

## 4.3 Daftar fitur dan detail cakupan

Daftar ini adalah indeks fitur. Rincian field, alur, status dan aturan bisnis mengikuti bagian acuan; bukan tambahan modul. Setiap layar daftar menyediakan pencarian/filter, detail, status kosong/galat dan pagination sesuai kebutuhan. Tambah/ubah/hapus hanya muncul bila diizinkan; data historis memakai koreksi/arsip, bukan hapus permanen.

| Fitur | Detail yang disediakan | Acuan |
|---|---|---|
| Core System | Profil sekolah; master tahun/semester, program, kelas, guru/staf/siswa/wali; akun dan penugasan; impor/ekspor; histori kelas, kenaikan/kelulusan; progres massal dan audit. | Bab 4–5, 12–13 |
| PKL | Periode/mitra/kuota, penempatan, pembimbing, jurnal siswa, kehadiran PKL, kunjungan, nilai kertas diinput guru dan laporan. | Bab 7 modul 01 |
| Jurnal Kegiatan Guru | Jurnal harian wajib per kegiatan harian guru, draf/kirim/koreksi, kelengkapan harian dan rekap pimpinan. | Bab 7 modul 02 |
| Presensi dan Izin | Presensi siswa per jam, konfirmasi wali kelas, tindak lanjut bersama BK, izin wali online/offline, izin guru dan rekap. | Bab 7 modul 03 |
| Surat | Surat masuk/keluar, pencatatan tanpa file, editor/unggah opsional, penomoran final, persetujuan dan buku agenda. | Bab 7 modul 04 |
| Keuangan | Jenis biaya/tarif, tagihan individu/massal, cicilan manual, kuitansi, koreksi/void ber-audit, tunggakan historis. | Bab 7 modul 05 |
| BKK | Verifikasi alumni, lowongan, lamaran/CV opsional, hasil seleksi manual dan laporan penempatan. | Bab 7 modul 06 |
| Sarpras | Master aset tunggal, lokasi/kondisi, pinjam/kembali, perbaikan dan laporan inventaris. | Bab 7 modul 07 |
| BK | Rujukan, kasus/sesi, catatan konseling privat, tindak lanjut dan laporan agregat. | Bab 7 modul 08 |
| Jadwal | Jam belajar, penugasan guru/mapel, jadwal kelas/ruang, cek bentrok, publikasi dan cetak. | Bab 7 modul 09 |
| Tracer | Periode/form survei, profil dan riwayat alumni multiaktivitas, jawaban historis dan laporan agregat. | Bab 7 modul 10 |
| HUBIN | Master mitra bersama PKL, kerja sama/MoU, masa berlaku, lampiran opsional dan kegiatan. | Bab 7 modul 11 |
| Supervisi | Penjadwalan, instrumen, observasi/nilai, rekomendasi, tindak lanjut dan rekap semester. | Bab 7 modul 12 |
| Laboratorium | Ruang/aset bersama, jadwal praktik, pinjam alat, stok bahan masuk/pakai dan laporan. | Bab 7 modul 13 |
| Pelanggaran | Kategori/bobot, pencatatan kejadian, validasi/koreksi, keberatan dan pembinaan tanpa hukuman otomatis. | Bab 7 modul 14 |
| Website | Profil/halaman, berita, pengumuman, galeri, tampilan, menu/submenu dinamis dan publikasi. | Bab 7 modul 15 |

## 4.4 Menu dan detail akses tiap role

Menu dibentuk dari **role + penugasan + hak tindakan + lisensi aktif**, bukan semua menu otomatis untuk satu jabatan. Rangkap role menggabungkan menu tanpa duplikasi. Core, Profil Saya dan Ganti Sandi tersedia sesuai pengguna; menu modul berbayar hanya bila aktif. Tindakan pada URL/Livewire/ekspor tetap diperiksa server. Role publik tidak dibuat.

| Role | Menu → submenu | Isi layar dan tindakan | Batas |
|---|---|---|---|
| Owner SaaS | Ringkasan Instalasi; Modul dan Lisensi; Langganan; Pembayaran Langganan; Riwayat Perubahan; Akun/MFA | Lihat versi/status; aktif/nonaktifkan modul, masa berlaku, catat pembayaran manual dan audit. | Panel `/owner`; tidak membuka transaksi siswa atau konseling. |
| Admin Sekolah | Dashboard; Profil Sekolah; Data Induk → Tahun/Semester, Program, Mapel, Kelas, Guru/Staf, Siswa/Wali | CRUD master, filter/impor/ekspor, penempatan kelas dan lihat histori siswa. | Tidak menghapus histori; impor mengikuti mesin massal. |
| Admin Sekolah | Akun dan Penugasan; Akademik → Pindah Kelas, Kenaikan, Kelulusan; Proses Massal; Audit Operasional | Berikan peran operasional/lingkup; preview dan konfirmasi proses; pantau progres, jeda/lanjut/retry. | Tidak memberi role owner atau membaca konseling otomatis. |
| Pimpinan Sekolah | Dashboard/Laporan; Persetujuan → Izin Guru, Surat, Koreksi sesuai izin | Pantau agregat sesuai bidang; buka detail pengajuan dan setujui/tolak dengan alasan. | Hanya bidang/objek ditugaskan; bukan semua persetujuan untuk setiap Waka. |
| Pimpinan Sekolah | Kurikulum → Jadwal, Monitoring Jurnal, Supervisi; Kesiswaan → Rekap Presensi/Pembinaan | Susun/terbit jadwal, cek bentrok; lihat kegiatan/hasil jurnal terkirim dan minta perbaikan; pantau presensi terkonfirmasi. | Menu kurikulum/kesiswaan mengikuti penugasan dan lisensi. |
| Pimpinan Sekolah | HUBIN → Mitra/Kerja Sama/Laporan PKL; Sarana → Rekap Aset/Lab | Pantau kerja sama, periode dan hasil; lihat inventaris, kondisi, peminjaman dan stok. | Sesuai bidang HUBIN atau sarpras; tidak mengedit jika hanya diberi izin lihat. |
| Guru | Dashboard → Presensi Hari Ini; Jurnal Kegiatan Saya → Isi Kegiatan/Hasil, Draf, Terkirim, Perbaikan, Riwayat | Tulis kegiatan dan hasil pekerjaan harian, termasuk nonmengajar; kirim dan koreksi catatan sendiri. | Tidak bergantung jadwal/presensi/kalender; fitur wajib tersedia bila modul aktif. |
| Guru | Presensi Siswa → Sesi Per Jam, Isi/Kirim, Riwayat; Izin Saya | Isi kehadiran pada setiap jam mapel; kirim; koreksi beralasan; ajukan izin guru tanpa presensi guru. | Hanya sesi pengajaran sendiri; tidak mengonfirmasi sebagai wali kelas lain. |
| Guru | Kelas Binaan → Konfirmasi Presensi, Izin Siswa, Tindak Lanjut BK, Data Siswa | Bagi wali kelas: cek semua jam, minta perbaikan/konfirmasi, catat izin dari wali di luar aplikasi, koordinasikan pembinaan. | Hanya kelas binaan; izin S/I oleh wali kelas langsung berlaku. |
| Guru | Penugasan → PKL, Supervisi, HUBIN; Jadwal Saya; Peminjaman Sarana | Pembimbing memeriksa jurnal PKL/input nilai kertas; koordinator menempatkan; supervisor isi instrumen; ajukan pinjam. | Submenu hanya sesuai penugasan dan modul aktif. |
| Tata Usaha | Surat → Masuk, Keluar, Templat, Persetujuan, Arsip, Buku Agenda | Catat metadata, editor/unggah opsional, ajukan surat, tetapkan nomor final sesuai proses dan cetak agenda. | Nomor final unik; persetujuan penandatangan tidak dilewati. |
| Tata Usaha | Administrasi Ditugaskan → Catat Izin Siswa, Mitra/HUBIN, Administrasi PKL | Catat laporan wali telepon/WhatsApp/surat/tatap muka, data mitra dan administrasi jika diberi tugas. | Pencatatan izin siswa hanya dengan delegasi wali kelas; izin guru sendiri tetap memerlukan pejabat lain. |
| Bendahara | Keuangan → Jenis Biaya/Tarif, Tagihan, Pembayaran, Kuitansi, Koreksi/Void, Tunggakan, Laporan | Terbit tagihan individu/massal, terima cicilan, cetak bukti, proses koreksi ber-audit, filter lintas tahun/alumni. | Saldo dan snapshot historis dilindungi; hak koreksi terpisah. |
| Pengelola Alumni | Alumni → Pengajuan Verifikasi, Profil, Riwayat; BKK → Lowongan, Pelamar, Hasil, Laporan | Cocokkan klaim manual, setujui/klarifikasi/tolak; terbit lowongan dan perbarui hasil seleksi. | BKK hanya bila aktif; profil/registrasi bersama tersedia bila BKK atau Tracer aktif. |
| Pengelola Alumni | Tracer → Periode, Form Survei, Jawaban, Kelengkapan, Laporan | Buka periode, susun form sederhana, periksa respons dan cetak agregat. | Tracer hanya bila aktif; riwayat jawaban tidak ditimpa profil terbaru. |
| Pengelola Sarana | Sarpras → Aset, Lokasi/Kondisi, Peminjaman, Pengembalian, Riwayat, Laporan | Kelola aset unit, serah-terima, pengembalian dan perubahan kondisi/lokasi. | Aset sama dengan Lab; hanya unit ditugaskan. |
| Pengelola Sarana | Laboratorium → Ruang, Alat, Pemesanan, Bahan, Mutasi Stok, Laporan | Atur ruang, cek bentrok, pinjam alat, catat stok masuk/pakai dan kerusakan. | Lab bisa aktif sendiri; tidak boleh stok minus atau aset dipinjam ganda. |
| Guru BK | Dashboard Pendampingan; Presensi → Rujukan/Klarifikasi/Tindak Lanjut; Pelanggaran → Pembinaan | Bersama wali kelas tinjau pola presensi, catat tindak lanjut dan hasil; bantu pembinaan kasus sah. | Hanya siswa ditugaskan dan fitur aktif; tidak mengganti konfirmasi wali kelas. |
| Guru BK | Konseling → Permintaan, Kasus, Sesi, Catatan Privat, Tindak Lanjut, Rekap | Terima rujukan, tangani sesi, simpan catatan terbatas dan hasil. | Menu Konseling butuh lisensi BK; isi privat tidak masuk laporan presensi bersama. |
| Editor Website | Website → Halaman/Profil, Berita/Pengumuman, Galeri/Media, Menu/Submenu, Tampilan, Pratinjau | Simpan draf, pratinjau, terbit/arsip; atur urutan dan visibilitas menu. | Hanya konten terbit ke publik; tidak membuka file siswa privat. |
| Siswa | Beranda; Profil; Jadwal; Presensi Saya; Izin Saya; Pelanggaran Saya; Tagihan Saya | Lihat data sendiri sesuai izin; ajukan izin/koreksi melalui petugas; lihat tagihan/kuitansi bila diberi akses. | Tidak mengubah presensi, status pembayaran, poin final atau persetujuan. |
| Siswa | PKL Saya → Penempatan, Jurnal, Kehadiran, Hasil; Layanan BK | Isi jurnal/kehadiran PKL sendiri; lihat hasil yang diizinkan; ajukan layanan BK. | Hanya penempatan sendiri; nilai industri diinput guru, bukan siswa. |
| Alumni | Profil/Riwayat Saya; Lowongan/Lamaran Saya; Survei Tracer; Tagihan Historis Saya | Perbarui aktivitas, melamar, isi survei dan lihat tagihan lama bila terkait siswa dan diberi akses. | Akun terverifikasi; fitur sesuai lisensi; alumni lama tanpa siswa tidak dibuatkan tagihan fiktif. |
| Orang Tua / Wali Siswa | Anak Saya → Ringkasan, Presensi, Izin, Tagihan, Pembinaan yang Diizinkan | Jika akun dipilih: lihat anak terverifikasi, ajukan izin opsional, lihat keputusan/rekap sesuai hak. | Tanpa akun tetap bisa menyampaikan izin di luar aplikasi; bukan akses catatan privat BK. |

**Kelengkapan izin akses:** halaman bersama seperti Jadwal, Peminjaman, Persetujuan dan Laporan mengikuti izin tindakan; fitur lintas role tidak memerlukan role baru. Label menu seluruhnya Bahasa Indonesia. Daftar detail setiap fitur mengacu §4.3 dan Bab 7; tampilan prototype lama adalah ilustrasi dan harus disesuaikan saat implementasi.

# 5. Desain data siswa dan proses akademik — persyaratan KRITIS

**Alur bagian:** Daftar/Impor Siswa → Tempatkan Kelas → Simpan Histori → Naik/Pindah/Lulus dengan Konfirmasi.

Bagian ini **wajib selesai dan diuji sebelum modul lain dibuat**. Kesalahan yang paling harus dihindari adalah hanya menyimpan `kelas_id` di tabel `siswa`, mengubahnya massal saat naik kelas, lalu kehilangan sejarah kelas dan membuat tagihan/alumni tidak sinkron.

## 5.1 Prinsip desain yang tidak boleh dilanggar

1. **Satu identitas siswa sepanjang masa.** `siswa.id` adalah primary key internal yang tidak berubah sejak siswa masuk hingga lulus, menjadi alumni, pindah, atau keluar. Gunakan `BIGINT UNSIGNED` auto increment sesuai Bab 12; jangan memakai NIS/NISN sebagai foreign key.
2. **NIS dan NISN adalah identitas administratif, bukan kunci relasi.** NIS unik pada instalasi sekolah; NISN boleh kosong saat belum tersedia, harus divalidasi format dan ditinjau apabila duplikat.
3. **Status siswa dan kelas adalah dua konsep berbeda.** Siswa dapat berstatus aktif tetapi belum ditempatkan di kelas; siswa lulus menjadi alumni tanpa kehilangan profil siswa/riwayatnya. Akun login juga terpisah dari identitas siswa dan tidak wajib dibuat untuk semua siswa.
4. **Riwayat tahun ajaran dan kelas tidak boleh ditimpa.** Penempatan baru membuat riwayat baru, penempatan lama ditutup, bukan di-overwrite.
5. **Satu siswa maksimal satu kelas aktif pada tanggal yang sama**, di dalam satu tahun ajaran sekolah. Jika pindah kelas di tengah semester, riwayat dipisahkan berdasarkan tanggal efektif.
6. **Kelulusan tidak dipicu jam/tanggal kalender.** Admin harus memilih siswa dan mengonfirmasi keputusan berdasarkan penetapan sekolah.
7. **Biaya yang sudah diterbitkan tidak berubah karena naik kelas, pindah kelas, atau lulus.** Tagihan memakai nilai dan konteks kelas saat diterbitkan; transaksi harus bisa diaudit.
8. **Lulusan yang pernah menjadi siswa di aplikasi tetap menggunakan `siswa.id` lama.** Alumni angkatan lama yang tidak ada di tabel siswa boleh mendaftar dengan profil alumni mandiri tanpa membuat siswa fiktif. BKK/Tracer menggunakan `profil_alumni.id` sebagai identitas alumni.
9. **Tidak ada hapus permanen untuk siswa yang punya riwayat operasional.** Gunakan status `pindah`, `keluar`, atau `lulus`; koreksi data memakai audit log.
10. **Semua aksi massal memakai preview dan konfirmasi** serta transaksi database. Untuk impor/kenaikan/kelulusan/tagihan massal, transaksi atomik berlaku per item: item berhasil tetap tersimpan, item gagal dicatat dan dapat diulang. Khusus kirim presensi satu kelas/sesi, seluruh daftar wajib satu transaksi utuh sesuai modul Absensi; bukan satu request per siswa.

## 5.2 Entitas data yang disarankan

Tabel di bawah adalah **model logis minimal** untuk desain database. Nama kolom boleh disesuaikan saat implementasi, tetapi hubungannya jangan diubah sembarangan.

| Tabel | Kolom kunci (contoh) | Fungsi / aturan |
|---|---|---|
| `siswa` | `id`, `nis`, `nisn` nullable, `nama`, `jenis_kelamin`, `tanggal_lahir`, `status_siklus` | **Sumber identitas siswa.** `id` tetap, data pribadi bukan salinan per tahun. Status: `aktif`, `lulus`, `mutasi_keluar`, `mengundurkan_diri`. |
| `siswa_wali` | `siswa_id`, `wali_siswa_id`, `hubungan`, `utama` | Relasi siswa dengan orang tua/wali; satu wali dapat terkait beberapa anak. Boleh diisi bertahap. |
| `wali_siswa` | `id`, `nama`, `telepon` nullable | Identitas wali terpisah dari akun login. |
| `tahun_ajaran` | `id`, `nama`, `tanggal_mulai`, `tanggal_selesai`, `aktif` | Contoh `2026/2027`; hanya satu tahun ajaran aktif untuk transaksi operasional normal. |
| `semester` | `id`, `tahun_ajaran_id`, `nomor`, `tanggal_mulai`, `tanggal_selesai` | Semester 1/2; tahun ajaran berbeda tidak berbagi data pertemuan. |
| `program_keahlian` | `id`, `nama`, `durasi_tahun` = 3 atau 4 | Membedakan program 3 tahun (kelas akhir XII) dan 4 tahun (kelas akhir XIII). |
| `kelas` | `id`, `tahun_ajaran_id`, `program_keahlian_id`, `tingkat`, `nama`, `wali_kelas_id` | Rombel **milik suatu tahun ajaran**. Contoh `X TKJ 1` tahun 2026/27 berbeda ID dengan nama serupa tahun 2027/28. |
| `kepesertaan_siswa` | `id`, `siswa_id`, `tahun_ajaran_id`, `program_keahlian_id`, `tingkat`, `status` | Satu baris **per siswa per tahun ajaran**; unique (`siswa_id`, `tahun_ajaran_id`). Menyimpan tingkat/program pada tahun itu; berlaku juga untuk siswa yang kembali pada tahun yang sama. |
| `penempatan_kelas_siswa` | `id`, `kepesertaan_siswa_id`, `kelas_id`, `berlaku_mulai`, `berlaku_sampai` nullable, `alasan` | Riwayat kelas dalam tahun ajaran. Rentang tanggal tidak boleh tumpang tindih; `berlaku_sampai` adalah batas akhir **eksklusif**. |
| `riwayat_status_siswa` | `siswa_id`, `jenis_peristiwa`, `tanggal_efektif`, `catatan`, `dibuat_oleh` | Jejak perubahan status penting, termasuk pindah/keluar dan koreksi. |
| `kelompok_kenaikan` + `peserta_kenaikan` | tahun asal/tujuan, keputusan per siswa, kelas tujuan, `diproses_pada` | Audit keputusan naik kelas/tinggal kelas/pending serta pencegah proses ganda. |
| `kelulusan` | `siswa_id` unique, `tanggal_lulus`, `tahun_ajaran_id`, `referensi_keputusan` nullable | **Hanya untuk siswa yang dinyatakan lulus.** Kelulusan final per siswa dicatat sekali, koreksi melalui prosedur khusus. |
| `profil_alumni` | `id` PK, `siswa_id` **nullable unique**, `pengguna_id` nullable unique, nama, angkatan/tahun lulus, jurusan, kontak, status verifikasi | **Identitas alumni.** Lulusan tercatat ditautkan ke `siswa.id`; alumni lama tidak dipaksa punya record siswa. |
| `pengajuan_registrasi_alumni` | `id`, `pengguna_id`, nama, tahun lulus, jurusan, NIS opsional, `status`, `diperiksa_oleh` | Klaim mandiri yang ditinjau petugas alumni (Pengelola Alumni) sebelum membuka akses alumni. |
| `riwayat_alumni` | `id`, `profil_alumni_id`, `jenis`, `organisasi`, `jabatan_atau_program`, `tanggal_mulai`, `tanggal_selesai` nullable, `catatan` nullable | Riwayat pekerjaan, wirausaha, pendidikan, dan aktivitas lain yang berulang; jangan menimpa catatan lama. |
| `peristiwa_pribadi_alumni` | `id`, `profil_alumni_id`, `jenis_peristiwa`, `tanggal_kejadian` nullable, `catatan` nullable | Catatan pribadi **opsional** seperti menikah; privat dan tidak ditampilkan dalam laporan umum. |
| `pengguna` | `id`, `email`/login, password hash, status | Akun login **terpisah** dari identitas siswa; siswa boleh belum memiliki akun. Hubungkan dengan `siswa.pengguna_id` nullable atau tabel relasi akun. |

**Aturan database wajib:** foreign key dan index pada seluruh referensi; unique `kepesertaan_siswa(siswa_id, tahun_ajaran_id)`; unique `profil_alumni(siswa_id)` untuk nilai yang tidak NULL, unique `profil_alumni(pengguna_id)` untuk nilai yang tidak NULL, dan unique `kelulusan(siswa_id)`; validasi penempatan aktif dan kesesuaian tahun/tingkat/program di backend. MariaDB tidak menyediakan partial unique index seperti PostgreSQL: mencegah dua penempatan aktif harus ditangani melalui transaksi/penguncian baris, dan boleh ditambah constraint yang sesuai bila diperlukan.

**Penting:** jangan menyimpan `kelas_terkini_id` sebagai sumber kebenaran kedua di tabel `siswa`. Kelas terkini dihitung dari penempatan aktif pada tahun ajaran aktif. Jika butuh percepatan query di kemudian hari, cache bukan sumber kebenaran.

### Relasi logis

```mermaid
flowchart TD
    S[siswa - identitas tetap] --> E[kepesertaan_siswa - per tahun]
    E --> P[penempatan_kelas_siswa - riwayat kelas]
    P --> C[kelas - rombel per tahun]
    S --> G[kelulusan - lulus manual]
    G --> A[profil_alumni - identitas alumni]
    R[Registrasi alumni lama + verifikasi] --> A
    A --> H[riwayat_alumni - karier dan pendidikan]
    A --> L[peristiwa_pribadi_alumni - pribadi opsional]
    S --> I[tagihan - tagihan tetap]
    I --> T[pembayaran - pembayaran]
    S --> K[PKL, BK, absensi, pelanggaran]
```

## 5.3 Contoh riwayat seorang siswa

| Tahun ajaran | Tingkat | Kelas | Keputusan | Identitas |
|---|---|---|---|---|
| 2025/2026 | X | X TKJ 1 | Naik | `siswa.id = 1001` |
| 2026/2027 | XI | XI TKJ 1 | Naik | `siswa.id = 1001` |
| 2027/2028 | XII | XII TKJ 1 | Lulus (program 3 tahun) | `siswa.id = 1001` |
| Setelah kelulusan | — | — | Alumni | `profil_alumni.id` baru, `siswa_id = 1001` |

Tagihan saat kelas X tetap tertaut ke `siswa_id = 1001` dengan kelas/tarif historisnya, meski profil alumni sudah ada. Tidak membuat entri siswa baru.

## 5.4 Proses kenaikan kelas massal (X → XI, XI → XII, XII → XIII bila program 4 tahun)

**Pelaku:** Admin Sekolah; Kepala Sekolah/Waka Kurikulum dapat memantau hasil sesuai wewenang. 

1. Admin membuat **tahun ajaran tujuan** dan rombel-rombel tujuan terlebih dahulu. Tahun ajaran baru belum otomatis memindahkan siswa.
2. Admin membuka menu **Akademik → Kenaikan Kelas**, memilih tahun asal, tahun tujuan, dan kelas asal.
3. Sistem menampilkan **daftar siswa aktif dan eligible** pada kelas asal, dengan checkbox per siswa dan **Pilih Semua** untuk semua hasil filter.
4. Admin menentukan hasil per siswa: **Naik**, **Tinggal Kelas**, atau **Ditunda**. Pilihan **Pindah/Keluar** ditangani lewat proses mutasi siswa tersendiri, bukan dianggap naik kelas.
5. Admin menetapkan kelas tujuan: bisa **set massal ke satu kelas** atau pemetaan kelas asal → kelas tujuan. Setiap siswa masih boleh diubah secara individual (misalnya X TKJ 1 dibagi menjadi XI TKJ 1 dan XI TKJ 2).
6. Sistem menampilkan **Preview** berisi nama/NIS, kelas lama, kelas baru, tingkat baru, jumlah naik/tinggal/ditunda, dan kesalahan validasi (mis. kelas tujuan tidak sesuai jurusan atau peserta sudah diproses). Jumlah siswa terpilih harus jelas, termasuk jika fitur Pilih Semua mencakup lebih dari satu halaman hasil filter.
7. Setelah admin menyetujui **modal konfirmasi**, sistem mencatat satu operasi massal beserta daftar siswa final (*snapshot*) dan mengunci pilihan selama operasi berjalan.
8. Browser mengirim beberapa **batch kecil secara paralel terbatas**. Untuk setiap siswa, pembuatan `kepesertaan_siswa` dan `penempatan_kelas_siswa` dilakukan **atomik dalam satu transaksi per siswa**, lalu hasil disimpan. Tidak ada satu transaksi panjang untuk ratusan siswa.
9. Modal menampilkan **progres nyata** (selesai/total, %, berhasil, sudah diproses, gagal, dilewati) yang terus diperbarui selama browser aktif.
10. Jika ada yang gagal atau browser ditutup, Admin dapat **Lanjutkan Proses** atau **Ulangi yang Gagal** tanpa mengulang siswa berhasil. Siswa **Ditunda** tetap belum dipindahkan.

**Peraturan teknis yang wajib:** 

- `Naik` menaikkan tingkat 1; `Tinggal Kelas` menjaga tingkat tetapi tetap membuat riwayat tahun ajaran **baru**. Jangan menyalin seluruh transaksi lama.
- **Siswa tidak boleh masuk dua kali** pada tahun tujuan. Proses ulang harus menolak duplikasi atau mengembalikan hasil yang sudah ada (idempotent).
- Kelas tujuan harus terdaftar pada tahun ajaran tujuan, sesuai tingkat dan program peserta.
- Menonaktifkan tahun ajaran lama **tidak menghapus** riwayat peserta; pemilihan tahun ajaran tidak sama dengan perintah kenaikan kelas.
- **Tidak boleh ada kenaikan otomatis pada tanggal tertentu.** Semua dijalankan lewat klik Admin.

## 5.4.1 Standar proses massal paralel dan progress nyata

**Berlaku untuk:** impor siswa, penempatan kelas, kenaikan kelas, kelulusan, pembuatan tagihan massal, dan impor alumni. Gunakan komponen proses massal yang sama; fitur tidak perlu membuat mekanisme progress masing-masing.

### Langkah pengguna

1. Admin memilih data menggunakan **checkbox per baris**, **Pilih Semua halaman** atau **Pilih Semua hasil filter** dengan jumlah terpilih yang jelas. Pilihan massal disimpan sebagai snapshot daftar ID di server saat pratinjau dikonfirmasi agar perubahan filter tidak mengubah peserta di tengah proses.
2. Admin memilih aksi dan parameter (misalnya tahun ajaran, kelas tujuan, atau nominal tagihan), lalu menekan **Pratinjau**. Sistem memeriksa duplikat, status peserta, penempatan yang sudah ada, dan kelengkapan data **tanpa mengubah database operasional**.
3. **Modal konfirmasi** menampilkan total dipilih, yang valid, yang dilewati, tujuan/perubahan, dan peringatan penting. Tombol **Proses Sekarang** aktif hanya jika aksi valid.
4. Sistem membuat catatan `operasi_massal` dan `item_operasi_massal`; setiap siswa/item mendapatkan status awal `menunggu`.
5. Browser menjalankan **2–3 request HTTP paralel maksimum**, masing-masing menangani **10–25 siswa/item** per batch (nilai awal, dapat dikonfigurasi menurut kapasitas VPS). Setiap request berdurasi singkat; hasil commit per siswa.
6. Sistem menyimpan status per item (`berhasil`, `sudah_diproses`, `gagal`, `dilewati`) dan menaikkan hitungan progres berdasarkan catatan yang benar-benar selesai. Frontend membaca respons dan/atau memanggil endpoint status berkala **selama halaman terbuka**.
7. **Jeda** menghentikan pengiriman batch baru, tetapi request yang sudah berjalan dibiarkan selesai. **Lanjutkan** mengambil hanya item yang belum selesai. Tidak ada proses yang dijanjikan berjalan terus setelah browser ditutup; request yang telanjur dikirim bisa saja selesai.
8. Ketika semua item memiliki hasil akhir, tampilkan **modal hasil** dan opsi **Unduh daftar gagal**, **Ulangi yang gagal**, atau **Kembali ke daftar**.

### Status, perhitungan progres, dan contoh tampilan

- `operasi_massal.status`: `draf`, `diproses`, `dijeda`, `selesai`, `selesai_dengan_galat`, `terputus`.
- `item_operasi_massal.status`: `menunggu`, `diproses`, `berhasil`, `sudah_diproses`, `gagal`, `dilewati`.
- **Progres = (`berhasil` + `sudah_diproses` + `gagal` + `dilewati`) / total item × 100%**. Angka 100% berarti seluruh item sudah memperoleh hasil, **bukan semua berhasil**; kegagalan tetap ditampilkan jelas.
- Contoh teks modal: **Memproses kenaikan kelas — 380/600 (63%)**; **Berhasil 371 · Sudah diproses 4 · Gagal 5 · Belum selesai 220**.
- Jika jaringan terputus, modal berubah menjadi **Koneksi terputus — 380 dari 600 tercatat. Periksa status dan lanjutkan**. Jangan mengklaim item berhasil hanya karena request sempat dikirim.
- Perubahan setelah siswa selesai diolah segera tersimpan. **Penanda tahun ajaran aktif tidak otomatis berubah** akibat selesainya sebagian batch; Admin mengaktifkannya secara terpisah.

### Aturan backend (tanpa Queue dan Scheduler)

| Kebutuhan | Implementasi minimal |
|---|---|
| Identitas operasi | `operasi_massal`: `id` UUID, `jenis`, `parameter` JSON, `status`, `total_item`, `dibuat_oleh`, timestamp. |
| Identitas item | `item_operasi_massal`: `operasi_id`, `entitas_id`, `status`, `pesan_galat` ringkas, `diproses_pada`; unique `(operasi_id, entitas_id)`. |
| Paralel terbatas | Javascript/Livewire memulai HTTP request berbeda dengan concurrency limit; PHP-FPM memproses request yang masuk, **bukan worker antrean**. |
| Pengambilan batch | Backend **mengklaim** item `menunggu` secara atomik; dua request tidak boleh memproses item sama. |
| Konsistensi | DB transaction **per siswa/item** untuk data yang saling bergantung; index/unique dan penguncian baris mencegah dua penempatan/tagihan/kelulusan. |
| Retry | Request diberi token idempotensi. Item berhasil tidak diulang; kegagalan yang sudah diperbaiki boleh di-reset ke `menunggu` setelah verifikasi. |
| Request berhenti tiba-tiba | Item `diproses` yang tertinggal dapat dideteksi saat admin menekan **Periksa Status/Lanjutkan**; sistem memeriksa hasil aktual dahulu sebelum mengembalikannya ke `menunggu`. |
| Hak akses | Semua endpoint `preview`, `start`, `process`, `status`, `pause`, `resume`, `retry` wajib otorisasi sesuai role dan tindakan. |
| Pengoperasian | Tidak ada background job, cron Laravel, Queue, Scheduler, progress fiktif, atau satu request PHP sangat panjang. |

**Prioritas MVP:** tidak perlu WebSocket, Redis, atau dashboard pekerjaan yang rumit. Polling status singkat atau pembaruan setelah setiap respons batch sudah cukup. Sesuaikan batch/concurrency dengan RAM dan PHP-FPM agar aktivitas guru lain tidak melambat.

## 5.5 Khusus tingkat akhir: kelas XII dan XIII

Jangan mengasumsikan semua siswa kelas XII langsung menjadi alumni. **SMK dapat mempunyai program 3 tahun atau 4 tahun.** Gunakan `program_keahlian.durasi_tahun` untuk menentukan kelas akhir.

| Program | Ketika siswa berada di XII | Tindakan yang diperbolehkan |
|---|---|---|
| 3 tahun | XII adalah kelas akhir | **Proses Kelulusan** secara eksplisit; jika belum lulus, dapat ditempatkan ulang di XII untuk tahun berikutnya. |
| 4 tahun | XII belum kelas akhir | Boleh diproses **Naik XII → XIII** pada tahun ajaran baru. Kelulusan dilaksanakan setelah XIII. |

**Proses Kelulusan:**

1. Admin membuka **Akademik → Kelulusan** dan memilih tahun ajaran serta siswa tingkat akhir yang memenuhi syarat program.
2. Admin menandai hasil: **Lulus**, **Belum Lulus**, atau **Belum Diputuskan**. Sistem menampilkan sumber keputusan sekolah (nomor/tanggal SK bila tersedia) dan tanggal efektif lulus.
3. Sistem menampilkan pratinjau siswa beserta **informasi tagihan belum lunas**. Tunggakan hanya informasi bagi bendahara, **tidak otomatis menghalangi kelulusan**.
4. Setelah konfirmasi, **hanya siswa Lulus** yang dibuatkan record `kelulusan` dan `profil_alumni` (relasi ke `siswa.id` yang sama), serta status siswa diperbarui menjadi `lulus`.
5. Siswa **Belum Lulus** tetap berstatus siswa; jika melanjutkan tahun ajaran berikutnya, ia dapat ditempatkan ulang pada tingkat akhir melalui alur penempatan/tinggal kelas.
6. Siswa **Belum Diputuskan** tidak diubah statusnya.

**Kelulusan keliru:** hanya admin yang diizinkan boleh melakukan **Koreksi Kelulusan** dengan alasan tercatat, pemeriksaan data BKK/Tracer yang sudah dibuat, serta jejak audit. Jangan langsung menghapus data alumni/transaksi terkait.

## 5.6 Pindah kelas, mutasi keluar, dan siswa masuk di tengah tahun

| Kejadian | Input Admin | Efek data |
|---|---|---|
| **Pindah kelas satu sekolah** | Siswa, kelas baru, tanggal efektif, alasan | Tutup `penempatan_kelas_siswa` lama (end eksklusif) dan buat penempatan baru; `siswa_id`/`kepesertaan_siswa_id` tetap. |
| **Pindah sekolah (mutasi keluar)** | Tanggal keluar, sekolah tujuan jika diketahui, keterangan | `status_siklus = mutasi_keluar`, tutup penempatan aktif; data keuangan/akademik tetap tersimpan. |
| **Berhenti/keluar** | Tanggal, alasan | Status `mengundurkan_diri`; riwayat tetap. |
| **Siswa pindahan masuk di tengah tahun** | Tambah identitas siswa, tahun, program, kelas, tanggal efektif | Buat satu `siswa` baru hanya jika siswa belum pernah ada; buat enrollment dan placement sesuai tanggal. |
| **Siswa lama kembali** | Cari profil lama melalui NIS/NISN/nama-tanggal lahir dan verifikasi | **Gunakan kembali `siswa.id` lama** jika orang yang sama, bukan buat akun/identitas kedua. |

**Validasi:** kelas tujuan harus ada di tahun ajaran sama dan tingkat/program sesuai; penempatan lama berakhir tepat saat penempatan baru dimulai; tidak boleh ada dua kelas pada tanggal efektif yang sama. Absensi sebelum perpindahan tetap berasosiasi dengan kelas saat kejadian, bukan berubah mengikuti kelas sekarang. Jika siswa keluar lalu kembali **pada tahun ajaran yang sama**, gunakan kembali `kepesertaan_siswa` tahun tersebut dengan riwayat penempatan baru; jangan melanggar unique enrollment per tahun.

## 5.7 Keuangan yang tetap konsisten dengan perubahan siswa

Modul Keuangan menggunakan `siswa.id` sebagai penghubung ke siswa aktif maupun alumni. **Pembayaran tidak boleh disimpan sebagai kolom tunggal pada siswa.**

| Tabel | Field minimum | Aturan |
|---|---|---|
| `jenis_biaya` | `id`, `nama` | SPP, praktik, kegiatan, dan lainnya. |
| `aturan_tarif` | `jenis_biaya_id`, `tahun_ajaran_id`, `tingkat` atau `kelas_id`, `nominal` | Tarif acuan untuk **tagihan baru**, bukan untuk menghitung ulang tagihan lama. |
| `tagihan` | `id`, `siswa_id`, `tahun_ajaran_id`, `kepesertaan_siswa_id` nullable, `nama_kelas_saat_terbit`, `nama_biaya_saat_terbit`, `nominal_saat_terbit`, `tanggal_jatuh_tempo`, `status` | Tagihan yang sudah diterbitkan mempertahankan nilai/konteks meski kelas/tarif berubah; periode + jenis tagihan dipakai untuk cegah penerbitan ganda. |
| `pembayaran` | `id`, `tagihan_id`, `nominal`, `dibayar_pada`, `metode`, `dibuat_oleh` | Pembayaran mengurangi saldo tagihan; koreksi tidak menghapus riwayat transaksi. |
| `koreksi_pembayaran` *(bila diperlukan)* | `pembayaran_id`, `alasan`, `nominal`, `disetujui_oleh` | Jejak pembatalan/pembetulan dengan alasan. |

Aturan proses yang harus diuji:

- Saat pindah kelas, **tagihan lama tetap** dan kelas pada kuitansi historis tidak berubah.
- Saat naik kelas, **tidak otomatis menerbitkan tagihan**; bendahara memilih tahun/kelas baru dan menekan **Buat Tagihan**. Validasi mencegah tagihan ganda pada siswa, jenis, dan periode yang sama. Pembayaran lebih besar dari sisa tagihan harus ditolak atau ditangani lewat prosedur koreksi terkontrol.
- Ketika tarif diubah, tagihan yang sudah diterbitkan **tidak ikut dihitung ulang**.
- Ketika siswa lulus atau pindah sekolah, **tunggakan tetap bisa dicari** melalui nama/NIS/riwayat siswa; pembayaran tetap dapat dicatat jika secara kebijakan diperbolehkan.
- Tagihan lunas tidak boleh menjadi belum lunas karena perubahan status akademik. Nilai tagihan, total pembayaran, dan sisa ditentukan oleh transaksi, bukan kelas siswa terkini.
- Tidak menggunakan `ON DELETE CASCADE` dari siswa ke tagihan, pembayaran, alumni, PKL, atau absensi historis.

## 5.8 Penghubung modul lain dan snapshot historis

| Modul | Identitas utama | Data historis yang harus dipertahankan |
|---|---|---|
| Absensi Siswa | `siswa_id` + tanggal + tahun ajaran | Kelas/penempatan saat absensi; jangan berubah sesudah mutasi kelas. |
| Perizinan Siswa | `siswa_id` + rentang tanggal | Status izin dan keputusan; satu sesi/jam tidak boleh punya dua presensi siswa yang bertentangan; izin boleh mencakup sebagian jam. |
| PKL | `siswa_id` + periode penempatan | Lokasi PKL, guru pembimbing, jurnal dan nilai di periode itu. |
| BK & Pelanggaran | `siswa_id` + tanggal kejadian | Catatan tetap pada identitas yang sama tanpa ikut pindah kelas; tampilan laporan historis memakai kelas saat peristiwa. |
| Keuangan | `siswa_id` + tagihan | Kelas/tarif saat tagihan dibuat. |
| BKK & Tracer | `profil_alumni.id`, `siswa_id` opsional | Lulusan terdokumentasi ditautkan ke siswa; alumni lama bisa punya profil tanpa siswa; seluruh riwayat karier menggunakan ID alumni. |

## 5.9 Kriteria penerimaan desain siswa (wajib lulus sebelum rilis)

| ID | Skenario uji | Ekspektasi |
|---|---|---|
| S01 | 35 siswa X TKJ 1 dipilih sekaligus ke XI TKJ 1 | Preview menampilkan 35; setelah konfirmasi, ada 35 enrollment tahun tujuan dan tidak ada duplikat. |
| S02 | Dua siswa dikecualikan dari batch | Hanya 33 diproses; dua siswa tetap pending dan bisa diputuskan terpisah. |
| S03 | Satu siswa tinggal kelas | Tahun baru mencatat tingkat X dan kelas X baru; riwayat X tahun lalu tetap. |
| S04 | Admin tanpa sengaja mengklik proses dua kali | Tidak membuat enrollment/placement kedua. |
| S05 | Siswa pindah kelas pada 1 Februari | Absensi Januari menampilkan kelas lama, absensi Februari kelas baru; satu placement aktif per tanggal. |
| S06 | Tahun ajaran berganti ketika siswa XII program 3 tahun belum diputuskan lulus | Tidak menjadi alumni otomatis. |
| S07 | Siswa XII program 4 tahun | Dapat naik ke XIII, **belum** menjadi alumni. |
| S08 | Siswa XII program 3 tahun dinyatakan lulus | Profil alumni tercipta dengan `siswa_id` yang sama; tidak ada record siswa baru. |
| S09 | Siswa dinyatakan lulus dan masih menunggak | Tetap alumni; tagihan dan saldo tertunggak bisa dicari bendahara. |
| S10 | Tagihan kelas X Rp100.000, siswa kini XI dengan tarif Rp125.000 | Tagihan lama tetap Rp100.000; tagihan baru menggunakan tarif baru saat diterbitkan. |
| S11 | Koreksi kelulusan setelah alumni mengisi tracer | Sistem meminta peninjauan data terkait, bukan menghapus otomatis. |
| S12 | Siswa mutasi keluar lalu kembali | Data lama ditemukan dan dipakai kembali, tanpa duplikasi identitas. |
| S13 | Proses kenaikan kelas gagal pada satu peserta | Siswa lain yang sudah sukses tetap tersimpan; peserta gagal ditandai, bisa diperbaiki dan dicoba ulang tanpa duplikasi. |
| S14 | Tahun ajaran lama dinonaktifkan | Rekap surat/absensi/tagihan historis masih dapat dilihat sesuai hak akses. |

**Keputusan implementasi:** selesaikan seluruh skenario S01–S14 dan uji proses massal M01–M09 pada Core System sebelum modul Keuangan, PKL, BKK, dan Tracer dipakai dengan data nyata. Tambahan uji alumni A01–A09 pada §5.10 wajib lulus sebelum pendaftaran alumni dibuka.

## 5.10 Pendaftaran mandiri alumni lama dan riwayat sepanjang hidup

**Masalah yang diselesaikan.** Alumni lama (misalnya lulusan sebelum aplikasi dipasang) mungkin belum pernah masuk `siswa`. Mereka **harus bisa mendaftar sendiri**. Jangan membuat baris `siswa` fiktif hanya untuk memungkinkan registrasi alumni. Alumni baru yang telah dicatat lulus tetap menggunakan identitas siswa lama.

### 5.10.1 Alur registrasi / klaim akun alumni

1. Jika **modul BKK atau Tracer Study aktif**, portal `/alumni/daftar` tersedia walaupun modul Website tidak aktif. Pendaftaran dibatasi agar tidak disalahgunakan.
2. Alumni mengisi **nama lengkap, tahun lulus, jurusan saat lulus, email aktif, password**, dan **NIS jika ingat (opsional)**. Tanggal lahir atau informasi tambahan dapat diminta **untuk proses verifikasi**, tidak wajib ditampilkan pada profil publik. Tidak meminta status menikah saat daftar.
3. Sistem membuat akun **berstatus menunggu verifikasi** dan `pengajuan_registrasi_alumni`, **belum** memberi hak membaca data siswa, tagihan, atau riwayat orang lain. Email boleh diverifikasi dengan tautan jika SMTP tersedia; pengiriman langsung, tanpa Queue.
4. **Petugas alumni (Pengelola Alumni)** meninjau klaim dan mencari kecocokan pada `siswa`, `kelulusan`, dan `profil_alumni` berdasarkan beberapa informasi. Sistem menampilkan kandidat **hanya kepada admin**, bukan kepada pendaftar. **Jangan otomatis menghubungkan identitas hanya berdasarkan nama/NIS.**
5. Bila cocok dengan alumni yang **sudah punya `profil_alumni`**, petugas alumni menghubungkan akun ke profil yang sama setelah verifikasi identitas; **tidak membuat profil kedua**.
6. Bila cocok dengan siswa lama yang sudah lulus tetapi belum mempunyai profil alumni, petugas alumni membuat satu `profil_alumni` yang menaut ke `siswa.id` tersebut. Keputusan lulus harus dapat dibuktikan dari catatan sekolah.
7. Bila **alumni lama benar-benar tidak ada pada master siswa**, petugas alumni menyetujui dan membuat `profil_alumni` dengan `siswa_id = NULL`, memakai data kelulusan yang telah diverifikasi.
8. Jika klaim tidak dapat dibuktikan atau duplikat mencurigakan, status **Perlu Klarifikasi / Ditolak**. Pendaftar dapat melengkapi informasi; admin mencatat alasan keputusan.
9. Alumni yang disetujui dapat login, melengkapi profil dan mengubah **riwayatnya sendiri**. Pendaftar yang belum diverifikasi hanya dapat melihat status registrasi dan mengoreksi pengajuan.

**Petugas verifikasi:** Pengelola Alumni.

**Status registrasi:** Menunggu Verifikasi → Disetujui / Perlu Klarifikasi / Ditolak. **Akun tidak otomatis disetujui karena NIS/NISN cocok.**

**Lulusan yang sudah punya akun siswa:** Setelah sekolah menetapkan kelulusan, hak akses siswa dapat dialihkan/ditambah menjadi alumni sesuai proses klaim akun. Jangan membuat akun baru bila orang tersebut sudah memiliki akun yang sah.

### 5.10.2 Fitur profil dan riwayat alumni

| Fitur | Subfitur sederhana | Aturan data |
|---|---|---|
| Profil Alumni | Nama, foto opsional, tahun lulus, jurusan, kontak, domisili opsional | Tahun/jurusan historis tetap; koreksi identitas penting perlu verifikasi admin. |
| Riwayat Kerja | Perusahaan, jabatan, tanggal mulai, tanggal selesai (opsional), deskripsi singkat | Bisa lebih dari satu pekerjaan, pekerjaan lama tidak ditimpa. |
| Riwayat Wirausaha | Nama/bidang usaha, mulai, berakhir (opsional), keterangan | Bisa bekerja **dan** berwirausaha bersamaan. |
| Riwayat Pendidikan | Nama lembaga/kampus, program studi, mulai, selesai (opsional) | Bisa kuliah sambil bekerja. |
| Kondisi Saat Ini | Sedang mencari pekerjaan, aktivitas lain, belum mengisi data | Jangan simpulkan `menganggur` hanya karena tidak ada riwayat kerja. |
| Perkembangan Pribadi | Status pernikahan/peristiwa menikah, perubahan lain yang ingin dibagikan | **Opsional, privat**, dan tidak menentukan akses layanan BKK. Tidak meminta nama pasangan/dokumen. |
| Timeline | Gabungan entri berdasarkan tanggal | Riwayat dapat ditambah, diedit, atau ditutup tanpa menimpa entri sebelumnya. |
| Tracer | Formulir kondisi alumni pada tanggal/periode pengisian | Laporan agregat karier/pendidikan; status pernikahan tidak perlu masuk statistik umum. |

**Catatan status.** Satu alumni boleh memiliki beberapa aktivitas aktif dalam waktu yang sama: bekerja + kuliah, bekerja + usaha, atau kuliah + usaha. Status menikah **bukan** status pekerjaan. Data pekerjaan/pendidikan menyimpan `tanggal_mulai` serta `tanggal_selesai` agar dapat menampilkan riwayat lama dan keadaan saat ini.

### 5.10.3 Koneksi BKK, Tracer, dan Keuangan

- Lamaran BKK serta jawaban tracer menggunakan **`profil_alumni.id`**, bukan mewajibkan `siswa.id`. Alumni lama dan alumni baru mendapat fitur sama.
- Alumni baru yang berasal dari data siswa: profilnya mempunyai `siswa_id` dan riwayat akademik tetap tersambung. **Tagihan memakai `siswa.id` lama** dan tidak terpengaruh perubahan riwayat alumni.
- Alumni lama tanpa `siswa_id` **tidak memiliki data tagihan otomatis**. Jangan membuat riwayat pembayaran, kelas, ataupun kelulusan palsu; jika arsip lama diimpor kemudian, admin dapat **menghubungkan** profil setelah verifikasi dan pemeriksaan duplikasi, dengan riwayat alumni tetap utuh.
- Status **Diterima** pada lamaran BKK tidak otomatis dianggap **sudah mulai bekerja**. Alumni mengonfirmasi pekerjaan dan tanggal mulai, atau admin mencatatnya setelah verifikasi.
- Laporan tracer mengambil snapshot per periode pengisian (misalnya status saat survei), tidak menghitung ulang jawaban tahun lampau memakai pekerjaan terbaru.
- Akses pendaftaran alumni tetap tergantung aktivasi **BKK atau Tracer**; tidak perlu membuat modul ketujuh belas atau layanan pusat.

### 5.10.4 Uji penerimaan tambahan (alumni)

| ID | Skenario | Harus berhasil |
|---|---|---|
| A01 | Alumni 2017 tidak ada dalam `siswa`, daftar sendiri | Menunggu verifikasi; setelah disetujui punya `profil_alumni.id`, `siswa_id = NULL`, dan akun aktif. |
| A02 | Lulusan 2026 sudah ada `siswa` dan `kelulusan` | Akun terhubung ke profil dengan `siswa_id` lama; tidak membuat siswa baru. |
| A03 | Orang mendaftar dua kali dengan data sama | Admin melihat kandidat duplikat; tidak tercipta dua profil terverifikasi untuk orang yang sama. |
| A04 | Pendaftar mengaku memakai NIS siswa lain | Tidak langsung melihat tagihan/data akademik; verifikasi manual diperlukan. |
| A05 | Alumni bekerja 2021–2023, kuliah 2022–2026, usaha sejak 2025 | Ketiga riwayat tersimpan dan timeline benar; pekerjaan pertama tetap historis. |
| A06 | Alumni mengisi status menikah | Boleh dikosongkan; data privat dan tidak masuk laporan umum atau syarat lowongan. |
| A07 | Arsip siswa lama diimpor setelah alumni mendaftar | Admin bisa menautkan `siswa_id` setelah verifikasi tanpa menghapus riwayat. |
| A08 | Alumni tanpa `siswa_id` menggunakan BKK dan Tracer | Dapat melamar dan mengisi tracer; tidak ada tagihan/kelas fiktif. |
| A09 | Alumni menerima tawaran kerja lewat BKK tetapi belum mulai bekerja | Tidak otomatis menambah pekerjaan aktif; riwayat bertambah setelah konfirmasi. |

## 5.11 Uji penerimaan proses massal dan UX (wajib)

| ID | Skenario | Hasil yang harus terlihat |
|---|---|---|
| M01 | Admin memilih 600 siswa lintas halaman menggunakan **Pilih Semua hasil filter** | Modal pratinjau tepat 600, tidak hanya 20 siswa yang sedang tampil. |
| M02 | Memproses 600 siswa dengan 3 request paralel | Progres bertambah sesuai hasil commit, request tidak melebihi batas paralel dan tidak ada siswa terproses dua kali. |
| M03 | Dua batch bersamaan memuat siswa yang sama karena retry | Claim/unique/idempotensi memastikan satu hasil akademik, bukan data rangkap. |
| M04 | Browser ditutup pada 380/600 | Yang sudah selesai tetap tersimpan; saat login kembali bisa menemukan operasi dan melanjutkan sisanya tanpa duplikasi. |
| M05 | Lima dari 600 siswa datanya tidak valid | Modal hasil menjelaskan 595 berhasil dan 5 gagal; gagal bisa diekspor dan diulang setelah diperbaiki. |
| M06 | Koneksi terputus saat request dikirim | UI menampilkan gangguan, membaca status dari server, tidak mengklaim sukses tanpa bukti. |
| M07 | Admin klik **Jeda** | Batch baru tidak dikirim; batch yang sudah berjalan dibiarkan berakhir; **Lanjutkan** aman. |
| M08 | Konfirmasi kelulusan 200 siswa | Modal menyebut dampak ke profil alumni, jumlah peserta dan siswa yang dikecualikan sebelum proses. |
| M09 | Simpan formulir biasa atau hapus data | Loading tombol, validasi inline, modal konfirmasi tindakan penting, dan modal error yang jelas. |

# 6. Panel pemilik SaaS lokal dan langganan

**Alur bagian:** Owner Masuk → Pilih Modul → Periksa Kesiapan → Aktifkan/Nonaktifkan → Audit.

| **Fitur**        | **Subfitur / alur**                                                 |
|------------------|---------------------------------------------------------------------|
| Profil pelanggan | Profil sekolah ini, domain dan kontak penanggung jawab.             |
| Aktivasi modul   | Daftar 15 modul → aktifkan/nonaktifkan → isi masa berlaku → simpan. |
| Lisensi          | Status aktif/kedaluwarsa; pemeriksaan saat request tanpa cron.      |
| Langganan        | Nama paket/modul, harga, periode, catatan.                          |
| Pembayaran       | Input pembayaran manual; bukti optional; riwayat pembayaran.        |
| Audit            | Riwayat perubahan lisensi dan pengaturan penting.                   |

**Perilaku lisensi.** Menu operasional modul nonaktif disembunyikan dan mutasinya ditolak pada route/Livewire; arsip baca/ekspor terkontrol mengikuti §13.5. Data tidak dihapus saat lisensi habis. Core System dan /admin tetap ada untuk pengelolaan dasar.

# 7. Persyaratan fitur dan proses bisnis 15 modul

**Alur bagian:** Pilih Modul Rilis → Jalankan Alur Peran → Simpan Data → Laporan Sesuai Hak.

## 01. Manajemen PKL

**Alur operasional:** Koordinator Atur Periode/Mitra → Tempatkan Siswa → Siswa Isi Jurnal/Kehadiran → Guru Pantau → Guru Input Nilai Kertas → Finalisasi → Laporan.

**Tujuan.** Mengelola penempatan, jurnal, absensi PKL, monitoring, dan hasil penilaian di satu tempat.

**Role.** Koordinator PKL, Guru Pembimbing, Siswa. Pembimbing industri tidak login.

| **Fitur inti**     | **Subfitur / cakupan**                                                                                                                  |
|--------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| Tempat PKL         | Perusahaan, PIC, kuota, alamat, kontak.                                                                                                 |
| Penempatan         | Siswa, lokasi, guru pembimbing, periode.                                                                                                |
| Jurnal & kehadiran | Input harian oleh siswa; pemeriksaan guru.                                                                                              |
| Monitoring         | Catatan kunjungan dan pembinaan guru.                                                                                                   |
| Nilai PKL          | Transkripsi nilai dari formulir kertas industri oleh guru pembimbing; sumber, penilai, tanggal; komponen nilai sekolah jika diperlukan. |
| Laporan            | Rekap siswa, jurnal, hadir, nilai; PDF/Excel.                                                                                           |

**Proses bisnis:**

> **1.** Koordinator menetapkan periode, mitra, penempatan dan guru pembimbing.
>
> **2.** Siswa mengisi jurnal dan kehadiran selama PKL; guru memeriksa.
>
> **3.** Pembimbing industri menyerahkan lembar nilai kertas kepada guru pembimbing.
>
> **4.** Guru pembimbing memasukkan nilai dari kertas ke aplikasi (scan/foto bukti opsional), menandai nama penilai industri dan tanggal.
>
> **5.** Koordinator memeriksa nilai final dan mengunduh rekap.

**Status.** Jurnal: Draft → Dikirim → Diperiksa / Perlu Perbaikan. Nilai: Belum Diinput → Draft → Final.

**Hasil.** Laporan PKL dan nilai final yang terlacak asalnya.

**Aturan penting.** Tidak ada akun/approval digital pembimbing industri; tanda tangan fisik tetap mengikuti prosedur sekolah. Nilai final yang dikoreksi harus memiliki alasan dan riwayat.

## 02. Jurnal Kegiatan Harian Guru

**Tujuan.** Guru wajib menuliskan **hari ini mengerjakan apa dan hasilnya apa**, bukan laporan per jadwal mengajar. Kegiatan boleh mengajar, menyiapkan materi, mengoreksi tugas, membimbing siswa, rapat, administrasi atau tugas lain. Kegiatan tidak mengajar tetap sah. Tidak ada ketergantungan jadwal, kalender akademik atau pengisian presensi.

**Alur operasional:** Buka Jurnal Saya → Pilih Hari Catatan → Tambah Kegiatan dan Hasil → Simpan Draf → Kirim → Lihat Riwayat → Koreksi Beralasan bila diperlukan.

| Bagian | Detail keputusan |
|---|---|
| Isian wajib | Tanggal kegiatan, uraian pekerjaan dan hasil pekerjaan. Satu jurnal per guru per tanggal, boleh berisi beberapa rincian kegiatan. Hasil boleh menjelaskan pekerjaan masih berlangsung dan capaian aktual; jangan memaksa klaim selesai. |
| Isian opsional | Kategori kegiatan, rentang jam, kelas/mapel bila relevan, kendala, rencana tindak lanjut dan lampiran. Tidak wajib memilih kelas/mapel atau mengisi materi mengajar. |
| Pengisian | Guru hanya mengelola jurnal sendiri. Default tanggal hari ini; tanggal lampau boleh untuk melengkapi kegiatan sebenarnya, tidak boleh tanggal masa depan. Larangan memilih tanggal hanya berlaku pada presensi, bukan jurnal. |
| Status | Draf → Terkirim → Perlu Perbaikan / Diperiksa. Draf bukan laporan terkirim. Koreksi jurnal terkirim menyimpan alasan, waktu dan aktor dalam audit. |
| Monitoring | Pimpinan kurikulum melihat jurnal yang ada, tanggal, isi/hasil dan status; dapat meminta perbaikan. Tanpa kalender, sistem **tidak menebak** hari kerja wajib dan tidak menyatakan guru melanggar karena tidak ada jurnal pada hari tertentu. Filter tanggal menghasilkan daftar catatan, bukan skor kepatuhan jadwal. |
| Lisensi | Tetap kode `jurnal_guru`; kewajiban melalui aplikasi berlaku pada sekolah yang mengaktifkan modul. Tidak memblokir presensi, tidak membuat presensi guru dan tidak memerlukan modul Jadwal. |
| Rekap | Per guru/periode/kategori; tampilkan kegiatan dan hasil, bukan persentase kehadiran guru. Hak lihat mengikuti penugasan pimpinan. |

**Contoh penerimaan:** guru tidak mengajar hari ini tetapi menyiapkan bahan ajar dan menghadiri rapat → jurnal dapat dikirim tanpa kelas/jadwal. Guru mengajar lima jam → satu jurnal harian dengan rincian kegiatan, sedangkan presensi tetap catatan siswa per jam yang terpisah.

## 03. Presensi Siswa & Perizinan

**Tujuan.** Saat modul `absensi_izin` aktif, guru mencatat **hari ini mengajar kelas/mapel apa dan siapa yang Sakit (S), Izin (I), Alpa (A)**. Siswa lain Hadir (H). Presensi per jam pelajaran, terpisah dari jurnal kegiatan guru. Tidak ada presensi guru, tidak memerlukan kalender akademik dan tidak wajib modul Jadwal.

**Alur guru:** Buka Presensi Hari Ini → Pilih Penugasan Kelas/Mapel dan Jam → Muat Seluruh Siswa serta Izin Wali → Periksa H/S/I/A → Simpan Satu Kelas → Terima Bukti Simpan Lengkap.

### 03.1 Penggunaan sehari-hari

- Tanggal presensi otomatis **hari ini menurut waktu server `Asia/Jakarta`**. Guru tidak disediakan pemilih tanggal dan tidak boleh mengirim tanggal lain melalui HTTP/Livewire. Pemeriksaan server wajib; menyembunyikan input saja tidak cukup.
- Guru memilih kelas/mapel dari penugasan aktif; jam pelajaran mengikuti jam yang benar-benar diajar. Jadwal aktif boleh menjadi saran, bukan syarat. Tidak membuat sesi otomatis untuk semua tanggal; tidak ada beban admin memelihara kalender/libur.
- Tampilan sederhana: nama/NIS siswa, radio H/S/I/A yang mudah disentuh di HP, ringkasan jumlah, dan satu tombol **Simpan Presensi**. H menjadi pilihan awal untuk siswa tanpa izin; tampilkan jelas bahwa ini **belum tersimpan**. Jam belum pernah dikirim tetap Belum Diisi, bukan otomatis H atau A.
- **Semua Hadir:** satu tindakan memilih H untuk siswa tanpa catatan wali; izin S/I yang sudah ada tidak boleh terhapus oleh tombol ini. Guru memeriksa lalu menyimpan. Tampilkan sumber izin dan alasan singkat seperlunya, bukan kontak/berkas pribadi wali.
- Izin yang dicatat wali kelas langsung membuat radio siswa **S atau I sesuai catatan** pada jam yang dicakup. A adalah ketidakhadiran tanpa izin, bukan kategori izin orang tua; bila ada catatan A dari wali kelas pada sesi terkait, tampilkan A dengan sumbernya. Konflik hadir dengan izin perlu klarifikasi wali kelas, bukan menimpa izin diam-diam.
- Penggantian guru/kelas hanya lewat penugasan pengganti yang sah. Satu guru tidak dapat mengambil kelas lain dengan mengganti ID dalam request.

### 03.2 Penyimpanan lengkap dan tahan pengiriman ulang (wajib)

**Batas transaksi:** satu pengiriman berisi seluruh siswa dalam satu sesi kelas/jam. Jika UI mengisi beberapa jam sekaligus, seluruh kombinasi siswa × sesi yang dipilih merupakan satu pengiriman utuh. **Dilarang satu request simpan per siswa.**

1. Saat layar dibuka, server membuat token pengiriman, versi daftar siswa dan versi catatan izin/presensi. Daftar bersumber dari penempatan kelas yang sah pada hari ini; catat jumlah dan himpunan ID yang diharapkan.
2. Payload berisi ID sesi, versi, kunci idempotensi, dan **seluruh** siswa serta H/S/I/A. Server memeriksa tanggal hari ini, guru berizin, lisensi, kesesuaian sesi, ID tidak ganda, tidak ada siswa hilang/tambahan, serta nilai status sah. Jangan memfilter baris invalid lalu menyimpan sisanya.
3. Di dalam satu transaksi MariaDB, kunci sesi, cek ulang versi daftar/izin/presensi, simpan semua baris dan status pengiriman. Bila satu baris gagal atau daftar/izin berubah, **rollback seluruh pengiriman** dan minta muat ulang; tidak ada sukses parsial.
4. Respons sukses hanya dikirim setelah commit dan verifikasi jumlah/himpunan baris tersimpan sama dengan yang diharapkan. Tampilkan **35 dari 35 siswa tersimpan**, H/S/I/A dan waktu simpan. Angka 35 hanya contoh; jumlah mengikuti daftar sebenarnya.
5. Tombol dinonaktifkan saat mengirim. Double-click, dua tab atau retry memakai kunci idempotensi yang sama mengembalikan hasil tersimpan, tidak menggandakan data. Sesi sudah berubah dari tab lain menghasilkan konflik versi, bukan last-write-wins.
6. Jika timeout/koneksi terputus, tampilkan **Status simpan belum diketahui**; periksa hasil token di server sebelum retry. Pertahankan input belum terkirim selama halaman tetap terbuka, dan jangan menampilkan toast sukses tanpa bukti commit. Tidak ada janji mode offline atau menyimpan data siswa sensitif ke localStorage.
7. Jika request dibuat sebelum tengah malam dan diterima setelah berganti hari, tolak tanggal lama dengan pesan muat ulang Presensi Hari Ini. Waktu server saat validasi transaksi menjadi acuan; jangan menggeser hasil ke tanggal baru diam-diam.

### 03.3 Izin sederhana dan konfirmasi wali kelas

**Alur izin:** Orang Tua Menghubungi Wali Kelas → Wali Kelas Catat S/I dan Rentang Waktu → Tersimpan serta Langsung Berlaku → Pilihan Presensi Guru Otomatis Sesuai → Wali Kelas Tinjau Rekap.

- Wali boleh menghubungi lewat telepon, WhatsApp, surat atau datang langsung; **tidak wajib akun, aplikasi atau lampiran**. Wali kelas memverifikasi pelapor secukupnya lalu mencatat siswa, S/I, alasan, rentang waktu, nama/hubungan pelapor dan kanal. Tidak perlu pejabat kedua atau tombol persetujuan terpisah untuk izin siswa ini.
- Pengajuan online siswa/wali, jika dipakai, menjadi pesan masuk untuk diperiksa wali kelas; setelah dicatat/diverifikasi wali kelas menjadi berlaku. Izin guru tetap mengikuti pejabat berwenang dan pengaju tidak menyetujui izin dirinya sendiri.
- Sumber izin berlaku juga untuk sesi yang dibuat kemudian. Jika sesi telah disimpan, terapkan perubahan secara transaksional, audit status lama/baru dan tandai Perlu Konfirmasi Ulang; jangan membiarkan laporan dan radio memiliki status berbeda.
- **Konfirmasi wali kelas tanpa batasan waktu.** Wali kelas dapat membuka rekap tanggal lampau, memeriksa satu kelas/hari, memilih semua catatan yang sesuai dan mengonfirmasi sekaligus. Tidak ada tenggat, kedaluwarsa, konfirmasi otomatis atau hukuman karena terlambat konfirmasi.
- Guru hanya mengisi/mengoreksi presensi hari ini. Kesalahan data lampau dilaporkan kepada wali kelas; wali kelas berwenang melakukan koreksi beralasan dengan audit, tanpa mengubah tanggal sesi. Admin hanya dapat membantu bila diberi delegasi koreksi eksplisit; tidak membuat sesi lampau atas nama guru tanpa jejak.
- Status kehadiran H/S/I/A berbeda dari status pemeriksaan Dikirim/Perlu Klarifikasi/Dikonfirmasi. Belum Diisi berarti sesi/baris belum dikirim, bukan A. Riwayat koreksi dan konfirmasi terdahulu tetap dapat ditelusuri.

### 03.4 Penanganan bersama BK

**Alur penanganan:** Temuan → Klarifikasi Wali Kelas → Rujukan BK bila diperlukan → Tindak Lanjut → Selesai.

Wali kelas mencatat temuan dari rekap; melakukan klarifikasi dan mencatat hasil. Jika selesai di wali kelas, catat tindakan/hasil lalu tutup tanpa rujukan. Bila perlu BK, tetapkan Guru BK yang bertanggung jawab, bagikan ringkasan relevan dan catat tindak lanjut/hasil. Penyelesaian wajib memiliki ringkasan hasil dan aktor/waktu; tidak ada hukuman atau poin otomatis. Kasus dapat dibuka kembali dengan alasan bila temuan berulang. Tidak ada ambang angka otomatis yang menjadi syarat wajib rujukan.

Kolaborasi presensi tersedia pada modul Absensi untuk wali kelas/Guru BK yang ditugaskan; catatan konseling rinci tetap privat pada modul BK bila aktif. Guru mapel melihat klarifikasi yang relevan, bukan isi konseling. Tanpa lisensi BK, tindak lanjut presensi dasar tetap dapat diselesaikan.

## 04. Administrasi Surat Digital

**Alur operasional:** TU Catat Surat → Simpan Metadata/File Opsional → Persetujuan bila Diperlukan → Nomor Final → Arsip → Rekap.

**Tujuan.** Mencatat surat masuk dan mengelola surat keluar baik dengan maupun tanpa file.

**Role.** Tata Usaha, Pejabat Penyetuju, Kepala Sekolah.

| **Fitur inti**    | **Subfitur / cakupan**                                                |
|-------------------|-----------------------------------------------------------------------|
| Surat masuk       | Nomor, tanggal, pengirim, perihal; file opsional.                     |
| Surat keluar      | Nomor, tanggal, tujuan, perihal; pilih pencatatan saja/editor/upload. |
| Template          | Kop dan isi surat sederhana bila memakai editor.                      |
| Penomoran         | Nomor unik untuk surat keluar final.                                  |
| Arsip & pencarian | Filter tanggal, jenis, perihal; lihat/unduh jika ada file.            |
| Laporan           | Buku agenda dan rekap surat.                                          |

**Proses bisnis:**

> **1.** TU memilih Tambah Surat Masuk atau Surat Keluar.
>
> **2.** TU mengisi metadata minimum; boleh tanpa lampiran, boleh mengunggah, atau menulis surat langsung di editor.
>
> **3.** Surat keluar yang perlu persetujuan diajukan ke pejabat; jika surat hanya diarsipkan, ikuti izin petugas.
>
> **4.** Nomor final ditetapkan sekali ketika surat keluar disahkan, lalu tercatat di buku agenda.
>
> **5.** TU melakukan pencarian dan cetak/unduh laporan.

**Status.** Surat keluar: Draft → Diajukan → Disetujui / Revisi → Arsip. Surat masuk: Dicatat → Arsip.

**Hasil.** Buku agenda dan surat digital bila ada.

**Aturan penting.** Tanpa file tetap valid; metadata wajib: nomor jika tersedia, tanggal, pihak terkait dan perihal. Hindari nomor surat ganda.

## 05. Keuangan & SPP

**Alur operasional:** Bendahara Atur Tarif → Preview Tagihan → Terbitkan → Catat Cicilan → Kuitansi → Tunggakan/Lunas → Koreksi Ber-audit bila Perlu.

**Tujuan.** Mencatat tagihan dan pembayaran sekolah tanpa memutus histori saat siswa naik kelas/lulus.

**Role.** Bendahara, Kepala Sekolah (ringkasan), Siswa/Orang Tua (tagihan sendiri bila diberikan akun).

| **Fitur inti**  | **Subfitur / cakupan**                              |
|-----------------|-----------------------------------------------------|
| Tarif & tagihan | Jenis, tarif, jatuh tempo, tagihan individu/massal. |
| Pembayaran      | Catat pembayaran dan cicilan manual.                |
| Kuitansi        | Nomor bukti dan cetak.                              |
| Tunggakan       | Sisa belum lunas, termasuk alumni.                  |
| Koreksi         | Pembetulan transaksi dengan alasan dan riwayat.     |
| Laporan         | Per siswa, kelas, tahun/periode, lunas/menunggak.   |

**Proses bisnis:**

> **1.** Bendahara mengatur tarif sesuai tahun ajaran/kelas dan menerbitkan tagihan.
>
> **2.** Tagihan menyimpan siswa_id serta snapshot kelas/tarif pada waktu penerbitan.
>
> **3.** Bendahara mencatat pembayaran, sistem menghitung sisa tagihan.
>
> **4.** Perpindahan kelas atau kelulusan tidak mengubah dan tidak menghapus tagihan lama.
>
> **5.** Bendahara melihat tunggakan siswa aktif maupun alumni, serta mencetak kuitansi/laporan.

**Status.** Tagihan: Belum Bayar → Sebagian → Lunas. Pembetulan dicatat sebagai koreksi.

**Hasil.** Buku pembayaran, kuitansi, tunggakan lintas tahun.

**Aturan penting.** Tidak ada pembayaran gateway pada MVP. Pembatalan/koreksi tidak boleh menghilangkan rekam audit.

## 06. BKK & Lowongan Kerja

**Alur operasional:** Alumni Daftar → Pengelola Verifikasi → Lowongan Terbit → Alumni Melamar → Hasil Seleksi Dicatat → Laporan.

**Tujuan.** Menyalurkan lowongan, menerima pendaftaran alumni mandiri, dan mencatat proses lamaran alumni/siswa akhir.

**Role.** Pengelola Alumni, Alumni terverifikasi, Siswa akhir (opsional).

| **Fitur inti** | **Subfitur / cakupan**                     |
|----------------|--------------------------------------------|
| Pendaftaran alumni | Form registrasi mandiri, verifikasi petugas alumni, hindari duplikat. |
| Profil alumni | Kontak, angkatan, jurusan, riwayat kegiatan (terhubung ke Tracer). |
| Lowongan | Perusahaan, posisi, syarat, batas tanggal. |
| Pelamar | Form lamaran dan CV opsional. |
| Status seleksi | Diproses, diterima, ditolak; hasil dari perusahaan dicatat admin. |
| Laporan | Rekap lowongan, pelamar, penempatan. |

**Proses bisnis:**

> **1.** Alumni lama mendaftar sendiri dan diverifikasi petugas alumni; alumni baru dapat memakai akun yang terhubung dari data kelulusan.
>
> **2.** Pengelola Alumni memasukkan dan menerbitkan lowongan.
>
> **3.** Alumni terverifikasi melamar pekerjaan dan mengunggah CV bila perlu.
>
> **4.** Pengelola Alumni menerima hasil seleksi dari perusahaan di luar sistem dan memperbarui status.
>
> **5.** Alumni mengisi riwayat kerja setelah benar-benar mulai bekerja; admin mengunduh laporan.

**Status.** Lowongan: Aktif/Arsip. Lamaran: Diajukan → Diproses → Diterima/Ditolak.

**Hasil.** Database lowongan dan rekap penempatan.

**Aturan penting.** Perusahaan tidak diwajibkan memiliki akun untuk MVP. Pendaftaran alumni tidak mensyaratkan adanya siswa di Core System; mekanisme §5.10 wajib diterapkan.

## 07. Inventaris & Sarpras

**Alur operasional:** Catat Aset → Tetapkan Lokasi/Kondisi → Pinjam → Serah Terima → Kembali/Perbaikan → Rekap.

**Tujuan.** Mencatat seluruh aset sekolah, penempatan, peminjaman, dan kondisinya.

**Role.** Petugas Sarpras, Guru/Staf peminjam, Waka Sarpras.

| **Fitur inti**      | **Subfitur / cakupan**                                   |
|---------------------|----------------------------------------------------------|
| Master aset bersama | Kode unik, nama, kategori, jumlah/unit, lokasi, kondisi. |
| Peminjaman          | Peminjam, tanggal, barang, pengembalian.                 |
| Perubahan kondisi   | Rusak/perbaikan; catatan riwayat.                        |
| Laporan             | Per lokasi, kondisi, dan peminjaman.                     |

**Proses bisnis:**

> **1.** Petugas Sarpras membuat/memperbaiki data aset pada master bersama.
>
> **2.** Pengguna meminjam, petugas mencatat penyerahan dan pengembalian.
>
> **3.** Petugas mengubah kondisi lokasi/barang disertai riwayat.
>
> **4.** Waka Sarpras melihat rekap aset seluruh ruangan, termasuk lab.

**Status.** Aset: Baik/Rusak/Perbaikan. Peminjaman: Dipinjam → Dikembalikan.

**Hasil.** Data aset tunggal dan laporan inventaris.

**Aturan penting.** Aset lab bukan salinan: modul Sarpras dan Lab memakai ID aset yang sama. Peralatan tetap memakai master aset; bahan habis pakai memakai catatan stok terpisah. Jika Lab aktif tetapi Sarpras tidak, petugas Lab tetap dapat mengelola aset Lab melalui antarmuka Lab terbatas.

## 08. Bimbingan Konseling

**Alur operasional:** Temuan → Klarifikasi Wali Kelas → Rujukan BK bila diperlukan → Tindak Lanjut → Selesai.

**Tujuan.** Mencatat layanan konseling dengan perlindungan catatan pribadi siswa.

**Role.** Guru BK, Siswa (permintaan jika diaktifkan), Wali Kelas (rujukan), Pimpinan (rekap).

| **Fitur inti**    | **Subfitur / cakupan**                                        |
|-------------------|---------------------------------------------------------------|
| Layanan BK        | Siswa, tanggal, jenis, guru BK.                               |
| Catatan konseling | Catatan privat, ringkasan tindak lanjut.                      |
| Rujukan           | Masukan dari wali kelas atau poin pelanggaran.                |
| Laporan           | Jumlah layanan/periode dan tindak lanjut, tanpa isi sensitif. |

**Proses bisnis:**

> **1.** Guru BK menerima permintaan atau rujukan.
>
> **2.** Guru BK melakukan layanan dan menyimpan catatan privat.
>
> **3.** Jika perlu, tindakan pembinaan/tindak lanjut ditentukan.
>
> **4.** Pimpinan hanya melihat statistik atau ringkasan yang diizinkan.

**Status.** Diajukan → Ditangani → Selesai.

**Hasil.** Riwayat BK dan laporan agregat.

**Aturan penting.** Catatan konseling tidak bisa dibaca Admin Sekolah/Owner secara otomatis.

## 09. Jadwal Pelajaran

**Alur operasional:** Tetapkan Jam/Pengajar → Susun Jadwal → Periksa Bentrok → Terbit → Guru/Siswa Lihat.

**Tujuan.** Menyusun jadwal yang tidak berbenturan dengan guru, kelas atau ruang.

**Role.** Waka Kurikulum, Guru, Siswa.

| **Fitur inti** | **Subfitur / cakupan**                     |
|----------------|--------------------------------------------|
| Waktu belajar  | Hari, jam, jeda.                           |
| Jadwal         | Kelas, guru, mapel, ruang, jam.            |
| Cek bentrok    | Validasi guru/kelas/ruang pada waktu sama. |
| Publikasi      | Jadwal aktif dan cetak per kelas/guru.     |

**Proses bisnis:**

> **1.** Waka mengatur jam dan pembagian guru.
>
> **2.** Waka membuat jadwal manual.
>
> **3.** Sistem menolak/memberi peringatan saat bentrok sebelum publikasi.
>
> **4.** Guru dan siswa membaca jadwal yang sudah terbit.

**Status.** Draft → Terbit.

**Hasil.** Jadwal kelas dan guru tanpa bentrok.

**Aturan penting.** Tidak perlu generator jadwal otomatis untuk MVP.

## 10. Tracer Study Alumni

**Alur operasional:** Alumni Terverifikasi → Isi Riwayat → Periode Survei Dibuka → Jawab → Simpan Snapshot → Rekap.

**Tujuan.** Alumni **lama dan baru** bisa mengisi serta memperbarui riwayat kerja, wirausaha, kuliah, dan perkembangan lain. Sekolah mendapatkan tracer study tanpa kehilangan perjalanan alumni dari waktu ke waktu.

**Role.** Alumni terverifikasi, Pengelola Alumni, Kepala Sekolah (rekap agregat).

| **Fitur inti** | **Subfitur / cakupan** |
|---|---|
| Registrasi alumni | Alumni lama mendaftar sendiri; petugas alumni memverifikasi sesuai §5.10. |
| Profil alumni | Kontak, angkatan, jurusan, data personal miliknya sendiri. |
| Riwayat pekerjaan | Perusahaan, jabatan, periode mulai/selesai, masih bekerja. |
| Riwayat wirausaha | Usaha/bidang usaha, periode mulai/selesai, masih berjalan. |
| Riwayat pendidikan | Perguruan tinggi/lembaga, program, periode, status studi. |
| Riwayat lain | Kegiatan lain dan peristiwa pribadi yang alumni pilih untuk dicatat; **status menikah opsional dan privat**. |
| Kondisi terkini | Bisa multi-aktivitas; mencari kerja dinyatakan oleh alumni sendiri. |
| Survei tracer | Isi kondisi pada tanggal survei; simpan snapshot respon per periode. |
| Rekap dan laporan | Angkatan, jurusan, pekerjaan, kuliah, usaha; Excel/PDF. |

**Proses bisnis:**

> **1.** Alumni yang belum tercatat mendaftar mandiri dan menunggu verifikasi petugas alumni; lulusan yang sudah terdata menghubungkan akun ke profil yang ada.
>
> **2.** Alumni login, melengkapi profil, menambahkan riwayat kerja/kuliah/usaha (boleh lebih dari satu dan saling tumpang tindih).
>
> **3.** Alumni dapat menutup riwayat yang selesai lalu menambah riwayat baru; peristiwa pribadi sepenuhnya opsional.
>
> **4.** Saat periode tracer dibuka, alumni mengisi ringkasan kondisi saat itu. Data survei historis tidak ditimpa perubahan profil terbaru.
>
> **5.** Pengelola Alumni melihat jumlah responden dan laporan agregat. Data pribadi sensitif tidak tampil dalam laporan umum.

**Status registrasi.** Menunggu Verifikasi → Disetujui / Perlu Klarifikasi / Ditolak. **Status survei.** Belum Mengisi → Sudah Mengisi.

**Hasil.** Profil alumni berkelanjutan, timeline aktivitas, dan statistik alumni per angkatan.

**Aturan penting.** Pendaftaran dapat digunakan jika BKK **atau** Tracer aktif, tidak perlu modul Website; tidak menggantikan sistem pelaporan tracer resmi pemerintah. Tidak memaksa alumni memilih hanya satu status (bekerja/kuliah/usaha), dan tidak memaksa data pernikahan.

## 11. HUBIN / DUDI

**Alur operasional:** Catat Mitra → Catat Kerja Sama → Periksa Dokumen/Masa Berlaku → Catat Kegiatan → Laporan.

**Tujuan.** Menyimpan daftar mitra industri dan arsip kerja sama.

**Role.** Waka HUBIN, Staf HUBIN, Kepala Sekolah.

| **Fitur inti** | **Subfitur / cakupan**                                    |
|----------------|-----------------------------------------------------------|
| Mitra          | Nama, PIC, bidang, alamat, kontak.                        |
| Kerja sama     | Jenis, nomor dokumen, tanggal mulai/akhir, file opsional. |
| Kegiatan       | Kunjungan, guru tamu, kolaborasi.                         |
| Laporan        | Daftar mitra dan kerja sama aktif.                        |

**Proses bisnis:**

> **1.** Staf memasukkan mitra dan rincian kerja sama.
>
> **2.** Waka memeriksa periode dan dokumen bila ada.
>
> **3.** Staf mencatat pelaksanaan kegiatan dengan mitra.
>
> **4.** Pimpinan melihat rekap.

**Status.** Draft → Aktif → Berakhir.

**Hasil.** Database mitra dan arsip kerja sama.

**Aturan penting.** Mitra yang digunakan PKL mengambil sumber data yang sama jika kedua modul aktif.

## 12. Supervisi & Kinerja Guru

**Alur operasional:** Jadwalkan Supervisi → Supervisor Observasi/Nilai → Guru Lihat Hasil → Tindak Lanjut → Selesai.

**Tujuan.** Mencatat observasi dan tindak lanjut supervisi internal sekolah.

**Role.** Kepala Sekolah, Waka Kurikulum, Supervisor, Guru.

| **Fitur inti** | **Subfitur / cakupan**            |
|----------------|-----------------------------------|
| Jadwal         | Guru, supervisor, tanggal, kelas. |
| Instrumen      | Indikator, nilai, catatan.        |
| Hasil          | Ringkasan dan saran perbaikan.    |
| Laporan        | Rekap supervisi per semester.     |

**Proses bisnis:**

> **1.** Waka menjadwalkan supervisi.
>
> **2.** Supervisor mengisi observasi dan penilaian.
>
> **3.** Guru membaca hasil serta menindaklanjuti rekomendasi.
>
> **4.** Kepala Sekolah melihat rekap.

**Status.** Dijadwalkan → Dinilai → Selesai.

**Hasil.** Dokumen hasil supervisi dan rekap.

## 13. Laboratorium & Bengkel

**Alur operasional:** Siapkan Ruang/Aset → Catat Bahan Masuk → Pesan Ruang/Pinjam Alat → Pakai Bahan → Kembali → Rekap.

**Tujuan.** Mengatur penggunaan ruang praktik, aset alat, dan bahan habis pakai.

**Role.** Kepala Lab, Petugas Lab, Guru Praktik.

| **Fitur inti**    | **Subfitur / cakupan**                       |
|-------------------|----------------------------------------------|
| Data ruang        | Nama lab/bengkel, jurusan, penanggung jawab. |
| Alat praktik      | Referensi aset dari master aset bersama.     |
| Bahan habis pakai | Stok masuk, dipakai, sisa.                   |
| Jadwal praktik    | Guru, kelas, tanggal, ruang.                 |
| Peminjaman alat   | Pinjam, kembali, kerusakan.                  |
| Laporan           | Pemakaian ruang, alat dan stok bahan.        |

**Proses bisnis:**

> **1.** Kepala Lab mengatur ruang dan mengelola alat melalui daftar aset bersama.
>
> **2.** Petugas mencatat stok masuk bahan habis pakai.
>
> **3.** Guru menggunakan ruang dan meminjam alat; petugas mencatat pemakaian bahan.
>
> **4.** Petugas memperbarui stok dan kondisi alat; laporan tersedia bagi Kepala Lab/Sarpras.

**Status.** Bahan: Masuk/Dipakai. Alat: Tersedia/Dipinjam/Rusak.

**Hasil.** Riwayat praktik dan stok aktual.

**Aturan penting.** Bila hanya modul Lab yang aktif, pengelolaan aset terbatas ruang lab tetap tersedia melalui core aset yang sama. Tidak membuat tabel/identitas aset duplikat.

## 14. Poin Pelanggaran Siswa Digital

**Alur operasional:** Guru Catat Temuan → Kesiswaan Klarifikasi/Validasi → Poin Sah → Pembinaan → Koreksi bila Perlu → Selesai.

**Tujuan.** Mencatat kejadian dan pembinaan dengan mekanisme validasi.

**Role.** Guru Piket/Guru, Waka Kesiswaan, Wali Kelas, Guru BK, Siswa/Orang Tua (ringkasan).

| **Fitur inti** | **Subfitur / cakupan**                          |
|----------------|-------------------------------------------------|
| Tata tertib    | Kategori pelanggaran dan bobot.                 |
| Catatan        | Siswa, tanggal, kejadian, poin, bukti opsional. |
| Validasi       | Disetujui/ditolak/dikoreksi.                    |
| Pembinaan      | Tindakan, catatan guru BK/wali kelas.           |
| Laporan        | Riwayat dan rekap per siswa/kelas.              |

**Proses bisnis:**

> **1.** Guru mencatat kejadian dan memilih kategori poin.
>
> **2.** Petugas kesiswaan memeriksa sebelum poin berlaku.
>
> **3.** Siswa/orang tua dapat meminta koreksi melalui petugas bila terjadi kesalahan.
>
> **4.** Guru BK/wali kelas mencatat pembinaan jika diperlukan.

**Status.** Menunggu Verifikasi → Sah/Dikoreksi/Ditolak.

**Hasil.** Riwayat kedisiplinan yang dapat ditelusuri.

**Aturan penting.** Tidak ada hukuman otomatis berdasarkan angka poin; keputusan pembinaan dibuat manusia.

## 15. Website Sekolah + CMS

**Alur operasional:** Editor Isi Konten → Atur Menu → Pilih/Pratinjau Tema → Terbitkan → Publik Melihat → Arsip bila Perlu.

**Tujuan.** Membuat dan memperbarui website publik sekolah pada alamat yang telah disediakan oleh pengelola infrastruktur.

**Role.** Editor Website, Admin Sekolah, Pengunjung publik.

| **Fitur inti**      | **Subfitur / cakupan**                                                                  |
|---------------------|-----------------------------------------------------------------------------------------|
| Halaman & profil    | Sejarah, visi-misi, sambutan, jurusan, kontak.                                          |
| Berita & pengumuman | Tambah, edit, draft, terbit, arsip.                                                     |
| Galeri              | Foto dan tautan video.                                                                  |
| Menu dinamis        | Tambah/ubah/hapus menu, urutan, submenu 1 tingkat, tujuan halaman/URL, tampil/sembunyi. |
| Tampilan            | Logo, warna, banner, footer.                                                            |
| Pratinjau           | Lihat hasil sebelum publikasi.                                                          |

**Proses bisnis:**

> **1.** Admin melengkapi profil dan identitas sekolah.
>
> **2.** Editor membuat halaman/berita dan menekan Simpan Draft atau Terbitkan.
>
> **3.** Editor mengatur menu dinamis, posisi, parent submenu dan tautannya.
>
> **4.** Pengunjung melihat menu serta konten yang sudah diterbitkan; editor dapat mengarsipkan konten.

**Status.** Konten: Draft → Terbit → Arsip. Menu: Tampil/Tersembunyi.

**Hasil.** Website responsif dengan navigasi yang dikelola lewat CMS.

**Aturan penting.** Menu tidak dikode secara statis; CMS menyediakan pengelolaan judul, URL tujuan, urutan, status tampil, dan submenu. Saat halaman ditarik dari publikasi, menu terkait disembunyikan atau diarahkan ulang agar tidak menjadi tautan rusak. Tidak ada PPDB atau page builder kompleks.

### 15.1 Lima tema website publik

**Alur:** Editor Buka Tampilan Website → Pilih Tema → Pratinjau Desktop/HP → Konfirmasi Terapkan → Publik Melihat Tema Baru → Kembali ke Tema Lama bila Diperlukan.

| Kode tema | Nama | Karakter dan susunan utama | Prototipe |
|---|---|---|---|
| `klasik` | Sekolah Klasik | Biru-putih formal; header identitas, sambutan, berita dan panel pengumuman. | `prototype/tema-01-klasik.html` |
| `modern` | Modern Minimalis | Ruang lega, hijau, tipografi besar, kartu layanan dan berita sederhana. | `prototype/tema-02-modern.html` |
| `vokasi` | Vokasi Industri | Navy-oranye, program keahlian unggulan, fasilitas praktik dan mitra industri. | `prototype/tema-03-vokasi.html` |
| `berita` | Portal Informasi | Merah-putih, berita utama dan kolom informasi, pengumuman serta agenda informasi. | `prototype/tema-04-berita.html` |
| `kreatif` | Kreatif Dinamis | Ungu-kuning, bentuk lembut, karya/prestasi dan galeri sebagai fokus. | `prototype/tema-05-kreatif.html` |

- Lima tema berlaku **hanya website publik**. Semua halaman admin, owner, guru dan portal operasional memakai satu desain/admin layout yang sama; tema publik tidak mengubah sidebar, form, otorisasi atau alur kerja admin.
- Tema adalah pilihan layout/presentasi dalam `resources/views/website/tema/` dan aset resources, bukan lima aplikasi atau database. Semua memakai konten CMS, menu dinamis, logo, warna identitas dan media sekolah yang sama; perbedaan susunan bukan hardcoded konten sekolah.
- Simpan `tema_website` pada profil/pengaturan website, hanya menerima lima kode allowlist, bukan path template dari input. Tema awal `klasik`. Simpan pilihan secara atomik; kode lama/tidak dikenal menggunakan fallback klasik dan log aman, bukan halaman 500.
- Pratinjau draft hanya untuk editor terautentikasi; publik tetap melihat tema aktif sampai Terapkan. Pergantian tidak menghapus berita, URL, menu, media atau SEO dasar; template menampilkan empty state jika konten belum tersedia.
- Seluruh tema responsif, navigasi keyboard, kontras memadai, Bahasa Indonesia, judul/deskripsi yang dapat diatur, serta menu/submenu CMS. Tidak ada PPDB, payment, formulir kontak atau fitur baru hanya karena ada elemen visual tema.
- Setiap tema wajib lulus uji halaman publik, berita/detail, halaman statis, menu mobile, submenu dan kondisi kosong/error. Pilihan tema dipertahankan saat upgrade maupun modul nonaktif/aktif ulang. Prototipe HTML adalah contoh desain, bukan bukti backend CMS sudah tersedia.

# 8. Integrasi wajib lintas fitur

**Alur bagian:** Gunakan Master Bersama → Periksa Lisensi Pasangan → Sinkronkan Relasi → Uji Modul Mandiri.

| **Sumber bersama**             | **Dipakai oleh**                         | **Ketentuan**                                                      |
|--------------------------------|------------------------------------------|--------------------------------------------------------------------|
| siswa + kepesertaan_siswa + penempatan_kelas_siswa | Absensi, PKL, Keuangan, BK, Poin | Satu ID identitas; tahun ajaran/kelas/masa aktif disimpan historis.                             |
| profil_alumni (`siswa_id` nullable) | BKK, Tracer, dan Keuangan lewat `siswa.id` bila ada | Lulusan terdata tetap tertaut tagihan; alumni lama tanpa siswa tidak mendapat tagihan fiktif. |
| aset                         | Sarpras, Laboratorium                    | Satu kode/ID aset; stok bahan habis pakai terpisah.                |
| mitra_industri              | HUBIN, PKL, BKK jika relevan             | Hindari pencatatan perusahaan ganda.                               |
| kelas + guru             | Jadwal dan Absensi                  | Jadwal hanya saran sesi presensi; jurnal kegiatan tidak bergantung jadwal.     |
| pengajuan_izin_siswa            | Absensi                                  | Catatan S/I wali kelas langsung menyelaraskan presensi hanya pada sesi/jam dalam rentang izin. |

**Independensi modul.** Jika satu modul pasangan tidak aktif, modul lain tetap dapat beroperasi dengan data minimum yang tersedia pada Core System.

# 9. Kriteria penerimaan minimum (UAT)

**Alur bagian:** Pilih Skenario → Siapkan Data Sintetis → Jalankan UAT → Catat Hasil → Perbaiki.

| **ID** | **Skenario uji**                             | **Harus berhasil**                                                                     |
|--------|----------------------------------------------|----------------------------------------------------------------------------------------|
| U01    | SMK membeli hanya Surat + Website            | Hanya dua modul tersebut tampil dan bisa dipakai; modul lain ditolak saat URL diakses. |
| U02    | PKL memberi nilai industri melalui kertas    | Guru mengisi nilai atas nama penilai industri; tanpa membuat akun industri.            |
| U03    | Surat masuk hanya metadata, tanpa berkas     | Dapat disimpan, dicari, dan masuk agenda.                                              |
| U04    | Pilih seluruh siswa kelas X untuk naik ke XI | Daftar calon perpindahan, kelas tujuan, dan pengecualian tampil sebelum konfirmasi.    |
| U05    | Kelas XII/XIII pada pergantian tahun        | Program 3 tahun: kelas XII hanya lulus lewat keputusan; program 4 tahun: XII boleh naik XIII. |
| U06    | Lulus dengan tagihan belum lunas             | Profil alumni muncul; tagihan tetap terlihat pada bendahara.                           |
| U07    | Tagihan semester lalu setelah naik kelas     | Kelas/nominal tagihan historis tidak berubah.                                          |
| U08    | Alat Lab diperbaiki melalui menu Lab         | Sarpras membaca kondisi aset terbaru tanpa duplikasi.                                  |
| U09    | Wali kelas mencatat S/I siswa                   | Hanya jam dalam rentang izin yang diselaraskan; catatan jam lain tetap, dan perubahan meminta konfirmasi ulang wali kelas.   |
| U10    | Guru mengajukan izin                         | Alur izin berjalan, tanpa membuat rekaman absensi guru.                                |
| U11    | Editor menambah submenu website              | Menu tampil di publik; urutan dan visibilitas sesuai pengaturan CMS.                   |
| U12    | Masa berlaku modul habis                     | Mutasi modul ditolak terkontrol; arsip tetap baca/ekspor sesuai hak, Core dan modul lain berjalan.                                |
| U13    | Alumni lama mendaftar mandiri                | Verifikasi petugas alumni diperlukan, tidak membuat siswa fiktif, riwayat alumni tersimpan. |
| U14    | Alumni bekerja sambil kuliah lalu wirausaha  | Semua entri berperiode tersimpan tanpa menimpa riwayat lama. |
| U15 | Proses massal 600 siswa memakai progres dan modal | Loading/progres nyata, konfirmasi sebelum eksekusi, ringkasan hasil, dapat jeda/lanjutkan/retry tanpa duplikat. |

# 10. Paket produk dan cakupan rilis

**Alur:** Tetapkan Rilis → Bangun Core serta Modul Target → Uji Alur Nyata → Pilot Sekolah → Catat Bukti → Aktifkan Penjualan Modul Lulus → Lanjutkan Rilis Berikutnya.

Katalog akhir tetap **Core + 15 modul berlisensi**. Istilah MVP berarti **R1**, bukan kewajiban membangun 15 modul sekaligus. Paket di bawah adalah urutan pengembangan dan penawaran awal, bukan role baru, bukan perubahan kode modul dan bukan harga langganan. Harga/periode kontrak diatur manajer bisnis di luar spesifikasi kode.

| Rilis | Modul dan cakupan | Kriteria penerimaan |
|---|---|---|
| R1 — Operasional harian | Core lengkap untuk data sekolah/siswa/guru, penugasan, owner/lisensi, akun, histori kelas/kenaikan/kelulusan dasar; `absensi_izin` dan `jurnal_guru`. Pendampingan presensi wali/BK dasar termasuk Absensi, bukan modul konseling penuh. | Presensi hari ini seluruh kelas tersimpan utuh; S/I langsung dari wali kelas; konfirmasi tanpa tenggat; jurnal kegiatan/hasil tanpa jadwal; penugasan dan privasi lulus; pilot operator/guru/wali/BK/pimpinan. |
| R2 — Administrasi dan pembinaan | `surat`, `website`, `pelanggaran`, `bk`, `jadwal` | Surat tanpa file, 5 tema web publik, poin tervalidasi, konseling privat dan jadwal independen lulus UAT modul terkait. |
| R3 — Kejuruan dan alumni | `pkl`, `hubin`, `bkk`, `tracer` | Mitra bersama, PKL nilai kertas oleh guru, alumni terverifikasi, satu Pengelola Alumni, riwayat dan snapshot tracer lulus. |
| R4 — Keuangan dan sarana | `keuangan`, `sarpras`, `laboratorium`, `supervisi` | Snapshot tagihan/cicilan, koreksi, aset bersama, stok, ruang dan supervisi lulus uji integritas/concurrency. |

- Core selalu tersedia; R1 menjual hanya dua modul yang benar-benar selesai. Modul rilis berikutnya tetap nonaktif dan tidak boleh ditampilkan seolah siap. Pemilik sekolah kelak dapat memilih modul satuan yang sudah lulus; paket tidak memaksa ketergantungan fitur berbayar.
- Penyiapan skema/model keuangan untuk menguji histori pada fondasi tidak berarti Keuangan sudah siap dijual. Bukti R4 tetap wajib. Alumni Core dari kelulusan tetap terhubung, tetapi portal registrasi mandiri hanya dibuka saat BKK/Tracer aktif.
- F5 pada TASK adalah daftar ID stabil, bukan urutan prioritas baru. Implementasi R1 memprioritaskan F5.03/F5.06, kemudian kelompok R2–R4 di tabel ini; K0–K3/F7–F9 diterapkan pada rilis masing-masing.
- Pilot R1 harus memperlihatkan: guru menyimpan satu kelas dan memperoleh jumlah tepat, wali menginput S/I dan melihat perubahan, konfirmasi rekap tanggal lampau, rujukan BK selesai, jurnal nonmengajar terkirim. Tidak menilai kepatuhan hari kerja dari kalender yang memang tidak disediakan.
- Lisensi aktif/nonaktif/berakhir mengikuti §13.5. Akses data lama dan ekspor terkontrol tetap tersedia untuk petugas berwenang; data tidak dihapus. Tidak ada masa tenggang otomatis pada MVP: batas akhir lisensi eksplisit menurut `Asia/Jakarta`, mutasi baru ditolak setelahnya.

# 11. Ketentuan keamanan dan operasional

**Alur bagian:** Validasi → Otorisasi → Transaksi → Audit → Tampilkan Hasil Aman.

- Akses berdasarkan role dan izin per tindakan; modul nonaktif tidak bisa diakses lewat URL atau pemanggilan Livewire secara langsung.

- File sensitif (nilai, pembayaran, dokumen siswa, konseling) berada di penyimpanan privat dengan pemeriksaan izin saat unduh.

- Gunakan transaksi database **per siswa/item** untuk operasi massal kenaikan/kelulusan/tagihan, dan transaksi yang sesuai untuk pembayaran serta perubahan stok; kegagalan item tidak membatalkan ratusan item lain yang sukses. Cegah konflik lintas-request dengan constraint dan kunci baris.

- Perubahan historis sensitif (kelulusan, nilai final, transaksi pembayaran, aset) memiliki audit berisi pengguna, waktu, alasan dan nilai sebelum/sesudah.

- Proses upload menggunakan validasi tipe/ukuran; tidak semua formulir harus mempunyai lampiran.

- Backup, pemeliharaan, dan pemulihan host dilakukan tim infrastruktur, bukan Queue/Scheduler Laravel. Developer menguji integritas aplikasi pada salinan data yang dipulihkan operator.

- Gunakan HTTPS, autentikasi aman, pembatasan login/registrasi, MFA wajib untuk owner sebelum rilis produksi dan dianjurkan untuk pengelola data sensitif. Alumni menilai sendiri data pribadi apa yang ingin dibagikan; catatan pernikahan tidak ditampilkan di dashboard publik atau laporan tracer umum.

- Tidak ada penghapusan otomatis ketika lisensi kedaluwarsa. Kebijakan retensi dan ekspor diatur dalam kontrak layanan.

- Setiap operasi massal memiliki catatan status yang dapat dilihat kembali, pembatasan jumlah request paralel, serta audit siapa yang memulai/menjeda/melanjutkan. Saat sekolah mengubah hak akses atau lisensi di tengah operasi, endpoint selanjutnya harus memeriksa hak akses ulang.

- Semua konfirmasi penting menggunakan modal yang konsisten, tombol tidak bisa diklik dua kali, loading dan pesan gangguan koneksi harus jelas; hindari menampilkan data sensitif di notifikasi yang bisa dilihat role lain.

# 12. Desain Database MariaDB — keputusan implementasi

**Alur bagian:** Tetapkan Entitas → Buat Kunci/Relasi → Migrasi → Uji Integritas → Pelihara Histori.

**Tujuan:** desain database yang cukup sederhana untuk satu sekolah, tetapi tidak kehilangan riwayat ketika siswa naik kelas, pindah, lulus, menjadi alumni, atau memiliki tunggakan. Bab ini memperinci model logis Bab 5; jika terdapat perbedaan penamaan, **aturan integritas dan relasi pada Bab 12 menjadi acuan implementasi**.

## 12.1 Konvensi dasar

| Area | Aturan yang wajib dipakai |
|---|---|
| Primary key | `id` BIGINT UNSIGNED auto increment untuk tabel utama; pakai pola sama di seluruh modul. Pengecualian: `operasi_massal.id` memakai UUID sesuai §12.7; `item_operasi_massal.operasi_id` wajib bertipe UUID yang kompatibel. |
| Foreign key | Gunakan FK pada data yang berkaitan; hindari penghapusan berantai untuk data historis penting. |
| NIS / NISN | `siswa.id` adalah identitas relasional permanen; NIS unik dalam sekolah, NISN nullable dan unik jika terisi setelah verifikasi. |
| Waktu | `created_at`, `updated_at` untuk tabel utama; `dibuat_oleh` / `diubah_oleh` pada aksi sensitif. Timestamp disimpan dalam UTC; tampilan dan tanggal efektif akademik memakai `Asia/Jakarta`. Kolom tanggal tanpa waktu tetap menyimpan tanggal lokal; uji batas hari/tahun. |
| Status | Gunakan enum aplikasi / nilai status terdokumentasi, bukan sekadar teks bebas yang berbeda-beda. |
| Data historis | Koreksi status menggunakan event/audit; catatan kelas, kelulusan, transaksi dan nilai tidak dihapus diam-diam. |
| Lampiran | Simpan file privat dan referensi path di DB; file boleh nullable untuk surat, PKL, izin, dan catatan tertentu. |
| Database | Satu MariaDB pada satu VPS/sekolah; **tidak perlu `sekolah_id` di seluruh tabel operasional**. |
| Modul tidak aktif | Tabel modul boleh tetap ada. Penonaktifan tidak menghapus baris data atau membuat akun hilang. |

**Prinsip penting:** data operasional tidak boleh mengambil kelas terkini dari nilai tunggal `siswa.kelas_id`. Kelas yang benar didapat dari riwayat penempatan siswa pada tahun ajaran dan tanggal yang relevan.

### 12.1.1 Penamaan database Bahasa Indonesia

- Tabel dan kolom bisnis memakai Bahasa Indonesia, `snake_case`, tanpa aksen dan tanpa pemaksaan bentuk jamak: contoh `siswa`, `penempatan_kelas_siswa`, `tagihan`, `pembayaran`, `tahun_ajaran_id`. Seluruh referensi Bab 5/8/12/17 dan `TASK.md` harus memakai nama yang sama.
- `id`, NIS/NISN/NPSN, `email`, `status`, tipe SQL, serta istilah teknis baku dipertahankan. Kolom teknis Laravel `created_at`, `updated_at`, `deleted_at` (jika dipakai), `password`, `remember_token`, dan `email_verified_at` boleh tetap mengikuti framework agar autentikasi/Eloquent kompatibel; label pengguna tetap Bahasa Indonesia. Tabel sesi/reset sandi/cache bawaan framework boleh tetap baku dan bukan tabel bisnis.
- Model memakai nama domain jelas seperti `Siswa`, `Tagihan`, `Pembayaran`, `Pengguna`; tetapkan `$table` dan foreign key relasi secara eksplisit karena pluralisasi Eloquent berbahasa Inggris. Model `Pengguna` tetap mengikuti kontrak autentikasi Laravel dan dikaitkan melalui konfigurasi provider. Jangan menerjemahkan nama API/method framework secara sembarang.
- Nilai status bisnis memakai kode Indonesia yang konsisten (contoh `menunggu`, `diproses`, `berhasil`, `gagal`); tampilkan label manusiawi seperti **Sedang diproses**. Kode modul lisensi dan path route yang sudah dikunci tidak diubah hanya untuk penerjemahan.
- Ini perubahan spesifikasi sebelum aplikasi dibuat, bukan migrasi database operasional. Bila kelak ditemukan database berisi data, buat migration perubahan nama beserta pengujian relasi; jangan hapus tabel atau reset data untuk menerapkan penamaan.

## 12.2 Tabel inti yang harus dibuat lebih dahulu

| Tabel | Kolom pokok | Relasi / batasan penting |
|---|---|---|
| `profil_sekolah` | `id`, `nama`, `npsn`, `domain`, `alamat`, `lokasi_logo`, `tema_website` | Satu baris profil sekolah dalam instalasi ini; `domain` hanya metadata profil, bukan pengelolaan DNS/server. |
| `pengguna` | `id`, `nama`, `email`, `password`, `aktif` | `email` unik untuk akun dengan login email; hindari akun duplikat. |
| `peran`, `hak_akses`, `peran_pengguna`, `hak_akses_peran` | `pengguna_id`, `peran_id`, `hak_akses_id` | Akun bisa punya beberapa role; pastikan kombinasi pada tabel pivot unik. |
| `siswa` | `id`, `nis`, `nisn` nullable, `nama`, `program_keahlian_id`, `status_siklus`, `pengguna_id` nullable | `nis` unik; `pengguna_id` unik bila terisi; `id` tidak berubah saat lulus. |
| `wali_siswa`, `siswa_wali` | Nama/kontak wali, `siswa_id`, `wali_siswa_id`, `hubungan` | Wali dapat terkait dengan beberapa siswa; akses akun wali diverifikasi, bukan hanya nomor telepon cocok. |
| `guru`, `pegawai` | Identitas, kontak, jabatan, `pengguna_id` nullable | Tidak wajib punya akun login; `pengguna_id` unik bila terisi. |
| `program_keahlian` | `id`, `nama`, `durasi_tahun` | Hanya durasi program yang valid (3 atau 4 tahun). |
| `tahun_ajaran`, `semester` | Tahun, tanggal, status aktif, nomor semester | Satu tahun ajaran operasional aktif pada satu waktu; semester terkait tahun yang benar. |
| `mata_pelajaran` | Kode, nama, program opsional | Dipakai Jadwal dan Jurnal. |
| `kelas` | `tahun_ajaran_id`, `program_keahlian_id`, `tingkat`, `nama`, `wali_kelas_id` | Rombel harus terkait satu tahun ajaran; nama kelas unik **di dalam tahun ajaran**, bukan global. |
| `kepesertaan_siswa` | `siswa_id`, `tahun_ajaran_id`, `program_keahlian_id`, `tingkat`, `status` | **UNIQUE (`siswa_id`, `tahun_ajaran_id`)**: satu enrolmen per siswa per tahun ajaran. |
| `penempatan_kelas_siswa` | `kepesertaan_siswa_id`, `kelas_id`, `berlaku_mulai`, `berlaku_sampai`, `alasan` | Riwayat pindah kelas; tidak boleh ada rentang tanggal bertabrakan untuk enrolmen siswa yang sama. |
| `riwayat_status_siswa` | `siswa_id`, `jenis_peristiwa`, `tanggal_efektif`, `catatan`, `dibuat_oleh` | Catat mutasi, keluar, kelulusan, pembetulan status. |
| `kelulusan` | `siswa_id`, `tahun_ajaran_id`, `tanggal_lulus`, `referensi_keputusan` nullable, `dibuat_oleh` | **UNIQUE (`siswa_id`)**; lulus hanya melalui aksi terkonfirmasi. |
| `log_aktivitas` | Aktor, aksi, objek, waktu, ringkasan perubahan | Untuk perubahan penting; jangan menyimpan password, token, atau catatan konseling lengkap dalam log. |

**Aturan penempatan kelas:** saat membuat penempatan, server mengunci enrolmen siswa (`SELECT ... FOR UPDATE`) lalu memastikan tidak ada penempatan dengan rentang tanggal tumpang tindih. `berlaku_sampai` adalah batas akhir **eksklusif**. Jika kelas X pindah ke kelas X lain pada 1 September, penempatan lama berakhir pada 1 September dan yang baru mulai 1 September. Jangan menerima penempatan lintas tahun ajaran atau tingkat yang tidak cocok.

**Aturan pergantian tahun:** tahun ajaran baru dibuat sebagai draft; Admin melakukan penempatan/kenaikan dan memeriksa daftar siswa yang belum diproses, lalu mengaktifkan tahun yang baru. Penggantian tahun tidak otomatis mengubah status siswa atau menciptakan kelulusan. Siswa yang tertunda tetap terlihat di daftar **Belum Diproses**.

## 12.3 Kenaikan kelas, kelulusan, dan akun alumni

| Tabel | Kolom pokok | Batasan |
|---|---|---|
| `kelompok_kenaikan` | `id`, tahun asal/tujuan, `status`, `dibuat_oleh`, waktu | Satu operasi massal yang tercatat; pratinjau sebelum konfirmasi. |
| `peserta_kenaikan` | `kelompok_kenaikan_id`, `siswa_id`, `kepesertaan_asal_id`, `kelas_tujuan_id` nullable, `keputusan`, `status` | UNIQUE (`kelompok_kenaikan_id`, `siswa_id`); tidak boleh menempatkan siswa dua kali di tahun tujuan. |
| `profil_alumni` | `id`, `siswa_id` nullable, `pengguna_id` nullable, tahun lulus, jurusan historis, `status_verifikasi` | `siswa_id` dan `pengguna_id` masing-masing unik bila terisi. Alumni lama boleh tanpa `siswa_id`. |
| `pengajuan_registrasi_alumni` | `id`, `pengguna_id`, nama klaim, angkatan, jurusan, `status`, `diperiksa_oleh` | Registrasi mandiri **wajib diverifikasi** sebelum ditautkan ke alumni/siswa existing. |
| `riwayat_alumni` | `profil_alumni_id`, `jenis`, organisasi, peran/program, `tanggal_mulai`, `tanggal_selesai` nullable | Banyak riwayat per alumni; pekerjaan, usaha dan kuliah dapat berlangsung bersamaan. |
| `peristiwa_pribadi_alumni` | `profil_alumni_id`, `jenis_peristiwa`, `tanggal_kejadian` nullable | Data opsional/privat; **bukan** statistik pekerjaan. |

**Algoritme promosi per siswa:** periksa enrolmen asal; validasi keputusan (naik/tinggal/ditunda) dan kelas tujuan; kunci siswa dan enrolmen tujuan; buat enrolmen tahun tujuan bila belum ada; buat penempatan bila belum ada; simpan item berhasil **dalam satu transaksi**. Jika enrolmen sudah dibuat oleh request paralel lain, baca hasilnya dan jangan buat duplikat. Gunakan unique constraint sebagai pagar akhir.

**Aturan kelas akhir:** program 3 tahun kelas XII dan program 4 tahun kelas XIII **tidak** masuk proses promosi standar. Admin melakukan kelulusan terpisah. Program 4 tahun boleh promosi XII → XIII. Kelulusan harus mencatat `kelulusan`, mengubah `siswa.status_siklus`, membuat/menautkan satu `profil_alumni`, dan menyesuaikan hak akun yang sudah ada **dalam transaksi yang konsisten**. Alumni yang belum tercatat lulus tidak boleh dibuat otomatis karena pergantian tahun.

**Akun siswa berlanjut:** apabila `siswa.pengguna_id` terisi, akun pengguna **sama** dapat memperoleh role alumni setelah kelulusan, tanpa membuat akun baru. Akses kelas aktif siswa berhenti sesuai status, tetapi catatan historis tetap ada. Alumni lama yang mendaftar mandiri mendapatkan `profil_alumni.id` tanpa membuat `siswa` fiktif. Jika nanti arsip siswa ditemukan, penautan dilakukan melalui verifikasi dan pengecekan duplikasi.

## 12.4 Tagihan dan pembayaran: jaga histori meski siswa pindah/lulus

Untuk MVP, gunakan **satu tagihan untuk satu jenis biaya dalam satu periode**, dan satu pembayaran menunjuk satu tagihan. Pembayaran bertahap/cicilan tetap bisa dibuat tanpa tabel alokasi yang rumit.

| Tabel | Kolom pokok | Ketentuan |
|---|---|---|
| `jenis_biaya` | `id`, `nama`, `aktif` | Jenis biaya seperti SPP/praktik; nama baru tidak mengubah tagihan lama. |
| `aturan_tarif` | `jenis_biaya_id`, `tahun_ajaran_id`, tingkat/kelas, `nominal` | Tarif acuan untuk **tagihan baru**. |
| `tagihan` | `id`, `siswa_id`, `jenis_biaya_id`, `kunci_periode`, `tahun_ajaran_id`, `kepesertaan_siswa_id` nullable, `nama_kelas_saat_terbit`, `nama_biaya_saat_terbit`, `nominal_saat_terbit`, `tanggal_jatuh_tempo`, `status` | **UNIQUE (`siswa_id`, `jenis_biaya_id`, `kunci_periode`)** dalam lingkup biaya yang ditagih sekali per periode; sesuaikan `kunci_periode` agar mencakup tahun ajaran yang relevan. |
| `pembayaran` | `id`, `tagihan_id`, `nomor_kuitansi`, `nominal`, `dibayar_pada`, `metode`, `status`, `dibuat_oleh` | Pembayaran parsial diperbolehkan; `nomor_kuitansi` unik. Koreksi/void dicatat, bukan menghapus pembayaran. |
| `koreksi_pembayaran` *(opsional)* | `pembayaran_id`, alasan, nilai perubahan, `disetujui_oleh` | Dipakai bila sekolah butuh pembetulan formal; jika belum, perubahan hanya lewat void yang diaudit. |

**Invarian keuangan:** `total_tagihan = tagihan.nominal_saat_terbit`; `total_bayar_efektif = SUM(pembayaran.nominal yang status-nya sah)`; `sisa = total_tagihan - total_bayar_efektif`. `sisa` tidak boleh negatif. Status **Belum Bayar / Sebagian / Lunas** dihitung dari transaksi sah, tidak diubah manual sembarangan. Saat beberapa request menerima pembayaran untuk tagihan yang sama, kunci baris tagihan dan hitung kembali saldo di dalam transaksi agar tidak kelebihan bayar.

**Larangan:** kenaikan kelas, pemindahan kelas, kelulusan, perubahan tarif dan perubahan role tidak boleh mengubah `nominal_saat_terbit`, `nama_biaya_saat_terbit`, maupun `nama_kelas_saat_terbit` pada tagihan yang sudah diterbitkan. Alumni tanpa `siswa_id` tidak mendapat tagihan fiktif. Pembuatan tagihan untuk alumni lama tanpa riwayat siswa **di luar MVP**.

**Tagihan massal:** pasangan (`siswa_id`, `jenis_biaya_id`, `kunci_periode`) menjadi kunci bisnis pencegah duplikasi. Jika sekolah memerlukan beberapa komponen berbeda pada periode sama, buat tagihan terpisah per jenis dan tampilkan total gabungannya di laporan. Retry batch dengan kunci sama harus melewati item yang sudah dibuat, bukan membuat tagihan ganda.

## 12.5 Satu master aset untuk Sarpras dan Laboratorium

| Tabel | Kolom pokok | Aturan |
|---|---|---|
| `aset` | `id`, `kode_aset`, nama, kategori, lokasi, kondisi, penanggung jawab | **UNIQUE (`kode_aset`)**; milik bersama Sarpras–Lab. |
| `peminjaman_aset` | `aset_id`, peminjam, tanggal pinjam/kembali, status | Satu aset individu tidak boleh dipinjam ke dua pihak secara aktif bersamaan. |
| `perpindahan_aset` | `aset_id`, lokasi asal/tujuan, waktu, alasan | Riwayat perpindahan dan perubahan penting. |
| `bahan_habis_pakai` | `id`, nama bahan, satuan, stok | Barang habis pakai berbeda dari aset berseri/QR. |
| `mutasi_stok_lab` | `bahan_habis_pakai_id`, jenis masuk/keluar, jumlah, aktor | Saldo stok berasal dari transaksi valid; tidak boleh minus. |
| `pemesanan_ruang_lab` | `ruang_id`, kelas/guru, waktu mulai/selesai | Validasi bentrok penggunaan ruang. |

**Pembedaan wajib:** laptop dengan nomor inventaris dikelola per unit di `aset`, sedangkan bahan seperti kabel, cairan, atau baut yang habis pakai dikelola kuantitas di stok Lab. Jika alat Praktik diperbaiki melalui modul Lab, perubahan kondisi terjadi pada **baris `aset` yang sama** dan langsung terlihat di Sarpras. Untuk transaksi stok paralel gunakan penguncian baris stok.

## 12.6 Data sederhana untuk modul lain

| Modul | Tabel minimum yang relevan | Aturan lintas modul |
|---|---|---|
| PKL | `mitra_industri`, `periode_pkl`, `penempatan_pkl`, `jurnal_pkl`, `kehadiran_pkl`, `penilaian_pkl` | Nilai industri diinput guru dari kertas, penilai dan tanggal dicatat; berkas bukti opsional. |
| Jurnal Kegiatan Guru | `jurnal_kegiatan_guru`, `rincian_jurnal_guru` | Catatan pekerjaan dan hasil harian, tanpa ketergantungan jadwal/presensi/kalender. |
| Absensi + Izin | `sesi_pelajaran`, `kehadiran_siswa`, `tindak_lanjut_presensi`, `pengajuan_izin_siswa`, `pengajuan_izin_pegawai` | `sesi_pelajaran` unik (`kelas_id`, `tanggal`, `jam_ke`); pengiriman satu kelas/sesi wajib atomik lengkap; `kehadiran_siswa` unik (`sesi_pelajaran_id`, `siswa_id`) untuk setiap jam; **tanpa** tabel absensi guru. |
| Surat | `surat`, `templat_surat` | Mode `metadata`, `editor`, atau `upload`; path file nullable. Penomoran final unik sesuai tahun/jenis. |
| BKK | `lowongan_kerja`, `lamaran_kerja` | Lamaran menunjuk `profil_alumni.id` atau siswa tingkat akhir yang sah. |
| BK | `kasus_konseling`, `sesi_konseling` | Catatan rahasia terpisah dan izin sangat terbatas. |
| Jadwal | `jadwal_pelajaran` | Periksa bentrok guru/kelas/ruang untuk rentang waktu yang sama. |
| Tracer | `jawaban_tracer` | Snapshot jawaban pada tanggal/periode pengisian, tidak ditimpa perubahan riwayat terbaru. |
| HUBIN | `mitra_industri`, `kerja_sama_industri` | `mitra_industri` dipakai bersama PKL; satu mitra jangan diduplikasi. |
| Supervisi | `supervisi`, `nilai_supervisi` | Guru hanya melihat hasil miliknya; supervisor menginput. |
| Poin pelanggaran | `aturan_pelanggaran`, `catatan_pelanggaran`, `tindak_lanjut_pelanggaran` | Poin berlaku sesudah validasi; koreksi mempunyai audit. |
| Website | `halaman_website`, `artikel_website`, `menu_website`, `media_website` | Menu dinamis dengan `induk_id` nullable, urutan, tujuan halaman/URL, status tampil. |

**Struktur jurnal dan izin dari luar aplikasi:**

| Tabel | Kolom pokok | Aturan |
|---|---|---|
| `jurnal_kegiatan_guru` | `id`, `guru_id`, `tanggal`, `status`, `dikirim_pada` nullable, `diperiksa_oleh` nullable, `catatan_pemeriksa` nullable | UNIQUE (`guru_id`, `tanggal`); satu jurnal harian, tidak bergantung jadwal/mapel/sesi. Rincian kegiatan pada `rincian_jurnal_guru`. |
| `pengajuan_izin_siswa` | `id`, `siswa_id`, `waktu_mulai`, `waktu_selesai`, `status_kehadiran` (S/I), `alasan`, `kanal_penerimaan`, `nama_pelapor`, `hubungan_pelapor`, `diterima_pada`, `dicatat_oleh`, `status` (menunggu_pemeriksaan/berlaku/dibatalkan), `diverifikasi_oleh` nullable, `lokasi_lampiran` nullable | Catatan wali kelas langsung berlaku; pengajuan online menunggu pemeriksaan wali kelas. Tanpa persetujuan pejabat lain, tanpa akun wali wajib. Koreksi/pembatalan ber-audit. |
| `rincian_jurnal_guru` | `id`, `jurnal_kegiatan_guru_id`, `urutan`, `kegiatan`, `hasil`, `kategori` nullable, `jam_mulai` nullable, `jam_selesai` nullable, `kelas_id` nullable, `mata_pelajaran_id` nullable, `kendala` nullable, `tindak_lanjut` nullable | FK jurnal; kegiatan dan hasil wajib tiap rincian; minimal satu rincian valid sebelum kirim. Jam/kelas/mapel opsional. |

**Struktur presensi per jam:**

| Tabel | Kolom pokok | Aturan |
|---|---|---|
| `sesi_pelajaran` | `id`, `kelas_id`, `tanggal`, `jam_ke`, `mata_pelajaran_id`, `guru_id`, `waktu_mulai`, `waktu_selesai`, `jadwal_pelajaran_id` nullable | UNIQUE (`kelas_id`, `tanggal`, `jam_ke`); validasi guru/mapel, waktu dan penugasan. Data sesi historis tidak ditimpa perubahan jadwal. |
| `kehadiran_siswa` | `id`, `sesi_pelajaran_id`, `siswa_id`, `penempatan_kelas_siswa_id`, `status_kehadiran`, `status_pemeriksaan`, `catatan`, `dikirim_pada`, `dikonfirmasi_oleh` nullable, `dikonfirmasi_pada` nullable | UNIQUE (`sesi_pelajaran_id`, `siswa_id`); siswa sesuai penempatan saat sesi. Koreksi membatalkan konfirmasi lama dengan jejak audit, lalu minta konfirmasi ulang. |
| `tindak_lanjut_presensi` | `id`, `siswa_id`, `tanggal`, `wali_kelas_id`, `guru_bk_id` nullable, `ringkasan_masalah`, `hasil_klarifikasi` nullable, `dirujuk_pada` nullable, `penanggung_jawab_id`, `tindakan`, `status` (temuan/klarifikasi/dirujuk/tindak_lanjut/selesai), `hasil` nullable, `diselesaikan_oleh` nullable, `diselesaikan_pada` nullable | Kolaborasi wali kelas/BK sesuai penugasan; bukan tempat catatan rahasia konseling. Referensi presensi terkait harus dapat ditelusuri. |

**Aturan izin–presensi:** catatan S/I oleh wali kelas langsung berlaku pada jam dalam rentang izin, termasuk sesi yang baru dibuat kemudian. Jika ada konflik versi ketika guru menyimpan, tolak seluruh kiriman dan muat ulang status terbaru. Catatan lampau dikoreksi wali kelas dengan audit, tidak membuka pemilih tanggal guru. Izin guru terpisah pada `pengajuan_izin_pegawai` sesuai persetujuan pejabat, tanpa presensi guru.

**Jejak pengiriman presensi:** tabel `pengiriman_presensi` menyimpan `id`, `kunci_idempotensi` unik, `guru_id`, `tanggal`, `versi_daftar`, `versi_data`, `jumlah_diharapkan`, `jumlah_tersimpan`, `ringkasan_status`, `disimpan_pada`. Relasi `sesi_pengiriman_presensi` mengaitkan pengiriman ke sesi yang ditulis. Token yang belum commit tidak boleh dilaporkan berhasil; token tersimpan menjadi bukti hasil retry. Tambahkan versi integer pada sesi untuk konflik dua tab dan versi sumber izin; kunci sesi dalam urutan ID konsisten bila menyimpan beberapa jam agar mengurangi deadlock. `jumlah_tersimpan = jumlah_diharapkan` dan kesamaan himpunan siswa wajib diverifikasi sebelum commit. Tidak memakai mesin operasi massal per siswa untuk pengiriman kelas ini.

## 12.7 Proses massal: progres nyata dan keamanan idempotensi

| Tabel | Kolom inti | Fungsi |
|---|---|---|
| `operasi_massal` | `id` UUID, `jenis`, `dibuat_oleh`, `status`, `total_item`, `parameter` JSON, `created_at`, `selesai_pada` | Mencatat impor, kenaikan, kelulusan, tagihan massal, atau impor alumni. |
| `item_operasi_massal` | `operasi_id`, `kunci_entitas`, `status`, `jumlah_percobaan`, `galat_terakhir`, `diproses_pada` | UNIQUE (`operasi_id`, `kunci_entitas`); tiap siswa/baris impor dihitung sekali. |

- Status operasi konsisten dengan Bab 5: `draf`, `diproses`, `dijeda`, `selesai`, `selesai_dengan_galat`, `terputus`.
- Status item: `menunggu`, `diproses`, `berhasil`, `sudah_diproses`, `dilewati`, `gagal`; item `gagal` bisa diulang setelah diperbaiki.
- **Progress = (`berhasil` + `sudah_diproses` + `dilewati` + `gagal`) / `total_item` × 100%** pada satu putaran proses. Artinya 100% adalah **seluruh item sudah mendapat hasil**, bukan berarti semua berhasil; tampilkan rincian berhasil/gagal/dilewati. `diproses` belum dihitung selesai.
- Jika Admin menekan **Ulangi yang Gagal**, item yang dipilih dikembalikan ke status `menunggu` sehingga persentase proses putaran baru bisa turun. Tunjukkan informasi **Mengulang item gagal** agar tidak membingungkan.
- `kunci_entitas` adalah identitas stabil seperti `siswa_id` atau baris file impor + checksum; request berulang untuk operasi/item yang sama tidak membuat duplikat.
- Tiap item diproses dengan transaksi untuk claim, validasi, perubahan bisnis, dan status akhir. Jika request mati, lanjutkan hanya setelah memeriksa hasil bisnis yang telah commit; jangan mengandalkan timeout sebagai tanda gagal.
- Browser mengirim **2–3 HTTP request paralel**, masing-masing mengerjakan **10–25 item**. Ini batas awal yang harus diuji pada VPS, bukan jaminan performa.
- Klik **Jeda** mencegah pengiriman batch berikutnya; request yang telanjur berjalan tetap boleh selesai. Setelah browser ditutup, proses tidak berjalan terus otomatis; saat login lagi Admin dapat **Lanjutkan** setelah pemeriksaan status.
- **Tidak ada Queue, Scheduler, cron aplikasi, atau worker latar belakang.** Persentase berasal dari status item yang benar-benar tersimpan di MariaDB, bukan animasi.
- Semua endpoint proses, status, retry dan resume memeriksa role serta lisensi setiap request. Setelah jaringan putus, frontend mengambil kembali status DB sebelum meminta retry.

## 12.8 Daftar constraint yang wajib ada sejak migrasi pertama

1. Unique NIS sekolah; NISN unik hanya jika terisi; `siswa.id` tidak boleh diperbarui.
2. Unique enrolmen siswa/tahun; kesesuaian kelas dengan tahun, program dan tingkat diperiksa dalam transaksi.
3. Tidak ada rentang penempatan kelas tumpang tindih; keputusan promosi dan kelulusan tidak dapat diproses dua kali.
4. Unique graduasi per siswa; alumni terkait siswa dan akun masing-masing maksimal satu profil.
5. Foreign key/tagihan selalu menunjuk `siswa.id` lama; snapshot nominal dan kelas tagihan tak berubah karena kenaikan/kelulusan.
6. Pembayaran tidak boleh melebihi sisa tagihan; transaksi dibatalkan/dikoreksi melalui audit, bukan dihapus.
7. Unique kode aset; stok lab tidak negatif; satu aset tidak dipinjam aktif oleh dua pihak.
8. UNIQUE sesi kelas/tanggal/jam dan UNIQUE presensi siswa/sesi; satu siswa tidak boleh tercatat di dua sesi yang waktunya bertabrakan. Status izin hanya berlaku pada jam yang dicakup; konfirmasi wali kelas terpisah dari status hadir.
9. Unique item per operasi massal dan kunci bisnis untuk tagihan massal, agar retry tidak menduplikasi.
10. Foreign key dengan `RESTRICT`/perlakuan aman untuk histori siswa, alumni, transaksi, dan aset; data sensitif tidak dihapus berantai.

---

# 13. Permission Matrix — role dan kewenangan

**Alur bagian:** Periksa Akun → Periksa Lisensi → Periksa Hak Tindakan → Periksa Lingkup → Izinkan/Tolak.

**Prinsip:** role hanya menjelaskan jabatan. Izin aktual harus mengecek **(1) pengguna aktif, (2) modul berlisensi aktif, (3) permission tindakan, dan (4) lingkup data miliknya/kelas/unitnya**. Menyembunyikan menu saja tidak aman. Pemeriksaan yang sama berlaku pada URL, aksi Livewire, unduhan, dan request massal.

## 13.1 Akses sistem inti dan panel owner

| Tindakan | Owner SaaS | Admin Sekolah | Kepala Sekolah/Waka | Guru/Wali Kelas | Siswa/Alumni |
|---|---|---|---|---|---|
| Kelola lisensi, aktivasi modul, masa berlaku | **Ya** | Tidak | Tidak | Tidak | Tidak |
| Catat pembayaran langganan SaaS | **Ya** | Tidak | Tidak | Tidak | Tidak |
| Kelola profil sekolah & tahun ajaran | Teknis terbatas | **Ya** | Lihat | Tidak | Tidak |
| Tambah/impor siswa, guru, kelas | Tidak secara default | **Ya** | Lihat sesuai tugas | Lihat lingkup tugas | Diri sendiri (profil) |
| Kenaikan kelas massal & kelulusan | Tidak | **Ya** | Lihat/konfirmasi sesuai kebijakan | Tidak | Tidak |
| Kelola akun dan role operasional | Owner akun sendiri | **Ya** | Tidak secara default | Tidak | Tidak |
| Lihat data BK/keuangan siswa | **Tidak default** | Sesuai izin khusus, bukan otomatis | Rekap seperlunya | Hanya sesuai penugasan | Hanya data sendiri yang diizinkan |
| Lihat informasi versi dan kesehatan instalasi | **Ya** | Terbatas | Tidak | Tidak | Tidak |

**Aturan owner:** pemilik SaaS boleh memperbaiki instalasi dan lisensi, tetapi tidak boleh otomatis membaca catatan konseling atau rincian keuangan sekolah. Bantuan yang memerlukan akses data sekolah harus diotorisasi secara eksplisit dan diaudit. **Jangan sediakan tombol bypass bebas**.

## 13.2 Matriks kewenangan tiap modul

| Modul | Boleh membuat/mengubah | Boleh memeriksa/menyetujui | Boleh melihat laporan | Batas data yang wajib diterapkan |
|---|---|---|---|---|
| 01 PKL | Koordinator (penempatan), siswa (jurnal sendiri), guru pembimbing (nilai kertas) | Guru pembimbing/koordinator | Koordinator, Waka HUBIN | Siswa: dirinya; guru: siswa binaan. Tidak ada akun industri. |
| 02 Jurnal | Guru pemilik jurnal harian | Pimpinan kurikulum (pemeriksaan opsional) | Waka Kurikulum, Kepala Sekolah | Guru hanya kegiatan/hasil sendiri, tidak wajib kelas/mapel; pimpinan sesuai bidang. |
| 03 Absensi + Izin | Guru mapel: presensi tiap jam; siswa/wali: izin; wali kelas dan BK: tindak lanjut | Wali kelas: konfirmasi presensi tanpa tenggat; wali kelas: S/I langsung; pejabat: izin guru | Wali kelas/BK sesuai penugasan, pimpinan agregat | Guru hanya sesi pengajarannya; wali kelas hanya kelas binaan; konseling privat terpisah. |
| 04 Surat | Tata Usaha | Kepala Sekolah/pejabat penandatangan untuk surat tertentu | Tata Usaha, Kepala Sekolah | Arsip/dokumen sesuai tugas; file boleh kosong. |
| 05 Keuangan | Bendahara | Bendahara berizin untuk koreksi/void | Bendahara, Kepala Sekolah (ringkasan) | Siswa/orang tua hanya tagihannya sendiri; Owner bukan bendahara. |
| 06 BKK | Pengelola Alumni (lowongan/status), alumni (lamaran sendiri) | Pengelola Alumni (validasi lowongan/akun alumni) | Pengelola Alumni | Alumni hanya lamaran/profil sendiri; perusahaan belum perlu akun. |
| 07 Sarpras | Petugas Sarpras; guru/staf (ajukan pinjam) | Petugas Sarpras | Petugas, Waka Sarpras | Unit/lokasi aset yang dikelola. |
| 08 BK | Guru BK (catatan), siswa (permohonan) | Guru BK untuk penanganan | Guru BK, pimpinan hanya angka agregat | Catatan rahasia per kasus; tidak otomatis terlihat oleh Admin/Owner. |
| 09 Jadwal | Waka Kurikulum/petugas penjadwalan | Waka Kurikulum (publikasi) | Guru, siswa, pimpinan | Guru/siswa hanya jadwal relevan; versi draft dibatasi. |
| 10 Tracer | Alumni (riwayat/jawaban sendiri), Pengelola Alumni (form/periode) | Pengelola Alumni (verifikasi akun) | Pengelola Alumni, pimpinan (agregat) | Data pernikahan privat dan tidak masuk laporan umum. |
| 11 HUBIN | Staf/Waka HUBIN | Waka HUBIN/pejabat berwenang | HUBIN, Kepala Sekolah | Dokumen kerja sama sesuai penugasan. |
| 12 Supervisi | Supervisor | Kepala Sekolah (hasil akhir bila diperlukan) | Guru (hasil sendiri), Kepala Sekolah | Hasil supervisi guru lain tidak terbuka untuk semua guru. |
| 13 Lab/Bengkel | Kepala/Petugas Lab (stok), guru praktik (pinjam) | Petugas/Kepala Lab | Kepala Lab, Waka Sarpras | Lokasi lab yang ditugaskan; aset dari master Sarpras bersama. |
| 14 Pelanggaran | Guru/Guru piket (catat), BK (pembinaan) | Waka Kesiswaan/petugas terpilih (validasi/koreksi) | Wali kelas/BK sesuai siswa, pimpinan agregat | Siswa/orang tua hanya catatan dirinya/anak terkait yang telah valid. |
| 15 Website | Editor/pegawai ditugaskan (konten/menu) | Editor/pengelola yang ditunjuk (publikasi) | Admin Sekolah; publik hanya konten terbit | Draft dan media internal tidak boleh terekspos publik. |

### Aturan approval minimal

- **Wajib persetujuan:** poin sebelum berlaku, izin guru, koreksi transaksi keuangan sah dan surat tertentu. Izin siswa S/I dari wali kelas langsung berlaku; pengajuan online siswa/wali cukup diperiksa wali kelas.
- **Tidak perlu approval berlapis:** jurnal guru standar, konten yang diterbitkan Editor berwenang, pembaruan profil sendiri, dan data operasional biasa (kecuali kebijakan sekolah mengharuskan).
- **Alumni lama:** mendaftar mandiri → status pending → Pengelola Alumni memverifikasi → baru mendapat akses alumni. Tidak ada klaim otomatis berdasarkan NIS/NISN.

## 13.3 Bentuk permission yang diterapkan di Laravel

Penamaan permission tetap konsisten: `resource.action`, misalnya `siswa.view`, `siswa.create`, `siswa.update`, `siswa.import`, `siswa.promote`, `siswa.graduate`, `finance.pembayaran.record`, `finance.pembayaran.void`, `discipline.records.verify`, `website.posts.publish`, `modules.activate`.

- Pengguna dengan dua role mendapat gabungan izin **yang sah**, tetapi tetap dibatasi penugasan data, bukan akses penuh otomatis.
- Setiap fitur minimal mendukung `view`, `create`, `update`, `delete` **hanya bila memang relevan**. Fitur historis/keuangan memakai `correct` atau `void`, bukan hard delete.
- Pengecekan otorisasi berbasis objek: Guru A tidak boleh mengubah jurnal Guru B, wali kelas tidak boleh membaca kelas lain, siswa tidak dapat memanipulasi `siswa_id` dalam request.
- Endpoint massal dan endpoint status operasi memeriksa pembuat/petugas berizin; angka progres operasi sensitif tidak bisa dibaca akun lain tanpa izin.
- Akses orang tua mensyaratkan relasi wali–anak sudah diverifikasi; tidak berdasarkan pencarian NIS saja.
- Ketika lisensi berakhir di tengah operasi massal, request berikutnya wajib berhenti dan menjelaskan status; data yang berhasil tetap tersimpan.

## 13.4 Akses ketika modul berakhir

**Alur:** Lisensi Berakhir/Nonaktif → Mutasi Baru Ditolak → Arsip Tetap Bisa Dibaca Sesuai Hak → Aktif Ulang → Lanjutkan Fitur Tanpa Kehilangan Data.

| Kondisi | Perilaku |
|---|---|
| Aktif + pengguna berizin | Fitur sesuai hak dan penugasan. |
| Belum dibeli/belum dirilis | Tidak ada menu operasional; route ditolak terkontrol. Modul belum dirilis tidak bisa diaktifkan owner. |
| Nonaktif/kedaluwarsa | Mutasi baru ditolak; arsip milik sekolah tetap baca/ekspor bagi petugas berwenang, tanpa membuka fungsi berbayar lain. |
| Core | Tetap berjalan; dashboard/proses/arsip hanya menampilkan data sesuai izin. |

## 13.5 Aktivasi dan nonaktivasi modul tanpa mengganggu aplikasi

**Alur:** Owner Pilih Modul → Lihat Dampak dan Proses Berjalan → Konfirmasi → Simpan Lisensi Atomik → Request Baru Mengikuti Status → Modul Lain Tetap Berjalan.

1. Katalog 15 modul tetap tersedia dalam codebase. Aktivasi/nonaktivasi **hanya mengubah konfigurasi lisensi**, bukan menginstal/menghapus source, tabel, menu template atau dependency. Core, login, owner dan dashboard tidak boleh ikut mati.
2. Sebelum aktivasi, periksa versi aplikasi/skema dan kesiapan modul. Modul belum dirilis atau migrasi belum lengkap ditolak dengan pesan jelas; jangan mengaktifkan lalu membiarkan route error. Fitur modul lain tidak boleh memanggil komponen/relasi/modul nonaktif tanpa pemeriksaan.
3. Simpan perubahan lisensi dalam satu transaksi dan invalidasi cache status/menu **setelah commit**. Otorisasi server memakai versi lisensi terkini setiap request; UI lama pada tab pengguna bukan bukti akses masih sah. Hindari cache per proses yang terus memperbolehkan izin lama.
4. Mutasi modul dan perubahan lisensi memakai lock/versi lisensi secara konsisten. Transaksi yang sudah memegang izin/lock boleh selesai utuh; nonaktivasi menunggu transaksi pendek tersebut. Jika owner menonaktifkan dahulu, pengiriman berikutnya ditolak **tanpa menulis sebagian data**. Tidak membatalkan commit yang telah berhasil.
5. Halaman modul yang sedang terbuka mendapat pesan Indonesia **Modul sedang tidak aktif** beserta jalan kembali ke dashboard saat request berikutnya. Tanggapi terkontrol, bukan 500, komponen hilang, stack trace, atau redirect berulang. Request presensi yang statusnya tidak diketahui tetap dapat memeriksa bukti pengiriman miliknya melalui endpoint audit Core berizin tanpa membuka mutasi modul.
6. Proses massal dihentikan mengirim batch baru; item sudah commit tetap tersimpan. Progres milik pengguna dapat dilihat melalui Core; proses dilanjutkan hanya setelah modul aktif dan izin valid. Sistem tidak menyatakan transaksi yang selesai sebagai gagal hanya karena lisensi berubah.
7. Data historis tetap ada. Sediakan mode **baca/ekspor arsip** untuk pengguna yang sebelumnya berwenang dan masih memiliki hak data, termasuk setelah lisensi habis; tidak membuka mutasi atau semua data untuk owner. Daftar arsip bisa diakses dari Core, tanpa route mutasi modul aktif.
8. Pasangan mandiri diuji: Lab tanpa Sarpras, PKL tanpa HUBIN, Tracer tanpa BKK/Website, Absensi tanpa Jadwal/BK/Jurnal, Jurnal tanpa semua modul lain. Master bersama tetap dapat dipakai antarmuka modul aktif tanpa membuka fitur pasangan.
9. Website nonaktif menampilkan halaman informasi minimal pada `/`; admin tetap satu desain dan login berfungsi. Tema tersimpan tidak dihapus; aktif ulang mengembalikan tema/data sebelumnya.
10. Tombol aktif ulang memakai pemeriksaan kesiapan yang sama, tanpa seed ulang yang menimpa data. Audit siapa, waktu, status lama/baru dan alasan; uji toggle berulang, expiry saat request, dua tab dan proses bersamaan.

# 14. Implementation & Testing Plan — dari kode hingga produksi

**Alur bagian:** Pilih Rilis → Implementasi → Uji → Pilot → Handover → Upgrade Terkendali.

**Prinsip rilis:** jangan mengklaim aplikasi siap produksi hanya karena PRD lengkap. Tahap di bawah adalah gerbang kerja dan pengujian nyata. Tidak menambah modul dan tidak mengharuskan Queue/Scheduler.

## 14.1 Urutan implementasi yang tidak merusak data

Nomor fase pada tabel ini adalah ringkasan produk, sedangkan ID pekerjaan tetap mengikuti `TASK.md`. Urutan eksekusi teknis: **F0 → skema/master F1 → F4 → penyelesaian impor F1 → F2 → F3 → F5/F6 → F7 → F8/F9**. K0–K2 dikerjakan bersama fondasi; K3 bersama QA/rilis. Kontrol lisensi minimum dari F3 disiapkan sebelum portal alumni dibuka. F4 harus tersedia sebelum impor, kenaikan, kelulusan, atau tagihan massal. Desain dan tes model tagihan/pembayaran disiapkan dalam F2; UI Keuangan tetap F5.09.

| Fase | Implementasi | Syarat selesai |
|---|---|---|
| 0. Fondasi proyek | Laravel 13 + PHP 8.5 + Livewire 4 + Tailwind 4 + MariaDB; konfigurasi aplikasi dan `.env.example` aman | Aplikasi stabil, login/logout, migrasi dan rollback teruji di staging. |
| 1. Data induk (F1 + F4 untuk impor) | Users/peran, profil sekolah, tahun ajaran, jurusan, kelas, siswa, guru, wali; impor validasi | ID siswa permanen, FK/unique berjalan, hasil impor jelas. |
| 2. Akademik historis (F2, sesudah F4) | Enrolmen, penempatan, pindah kelas, kenaikan kelas, program 3/4 tahun, kelulusan | Riwayat tidak terhapus; 600 siswa bisa diproses/diulang dengan progres nyata. |
| 3. Alumni & tagihan | Alumni lama/baru, riwayat kerja/kuliah/usaha, desain tagihan/pembayaran historis | Lulusan tetap terhubung ke tagihan; alumni lama tidak dibuat siswa fiktif. |
| 4. Super Admin lokal | Aktivasi 15 modul, masa berlaku, catatan pembayaran manual, audit, pengaturan owner | Modul tidak aktif ditolak backend; tidak ada akses bebas owner ke data sensitif. |
| 5. Modul operasional awal | Surat (tanpa file juga boleh), Website (menu dinamis), Absensi+Izin, Poin Pelanggaran | Alur lengkap per role, modal/konfirmasi/loading lulus uji. |
| 6. Modul pembelajaran & PKL | PKL, Jurnal, Jadwal, BK | Nilai industri diinput guru; data siswa mengikuti enrolmen tepat. |
| 7. Modul lain | Keuangan, BKK, Tracer, HUBIN, Sarpras, Lab, Supervisi | Integrasi data bersama tidak ganda; bisa aktif sendiri-sendiri. |
| 8. Pilot & rilis | UAT sekolah, tes beban, pengetesan keamanan aplikasi, verifikasi pemulihan data bersama operator, SOP support aplikasi | Semua **release gate** Bab 14.4 terpenuhi. |

**Catatan implementasi:** fase 3 menyiapkan model data keuangan dan alumni walaupun antarmuka modul Keuangan/BKK/Tracer dibangun belakangan. Ini mencegah perubahan besar pada relasi `siswa`, `profil_alumni`, dan tagihan saat modul baru diaktifkan.

## 14.2 Standar semua halaman agar pengalaman pengguna konsisten

| Situasi | Perilaku wajib |
|---|---|
| Masuk halaman / pindah tab | Skeleton atau loading jelas ketika konten belum siap; bukan halaman putih. |
| Simpan formulir | Tombol menunjukkan **Menyimpan...**, tidak dapat ditekan dua kali, validasi di bawah field. |
| Hapus/batalkan/ubah status penting | Modal konfirmasi menyebut objek dan dampak, tombol **Batal** dan tindakan utama jelas. |
| Perubahan berhasil | Toast sukses singkat dan data layar langsung sesuai database. |
| Gagal validasi | Pesan mudah dipahami dan data input tidak hilang. |
| Request lama tidak terukur | Spinner indeterminate; **tidak membuat persentase palsu**. |
| Operasi massal | Modal preview jumlah sasaran, bar persen dari DB, jumlah berhasil/gagal/lewat, tombol Jeda/Lanjutkan/Retry, ringkasan akhir. |
| Koneksi putus | Tampilkan status koneksi dan **Periksa Status** sebelum melakukan retry. |
| Hak akses/lisensi ditolak | Pesan jelas, bukan halaman error mentah; tidak mengungkap data modul. |
| Tampilan HP | Tabel responsif dengan filter sederhana, aksi utama mudah diakses; modal bisa digulir dengan benar. |

## 14.3 Test cases wajib untuk QA dan sekolah

| ID | Area | Skenario uji | Kriteria lulus |
|---|---|---|---|
| DB01 | Identitas siswa | NIS/NISN duplikat pada impor | Data duplikat ditolak per baris dengan alasan; siswa existing tidak tertimpa diam-diam. |
| DB02 | Tahun ajaran | Buat tahun baru, siswa belum diproses | Siswa tertunda terlihat; tidak otomatis naik/terhapus. |
| DB03 | Penempatan | Pindah kelas tengah tahun | Riwayat lama tertutup, yang baru aktif, tidak tumpang tindih. |
| DB04 | Promosi | Dua request paralel menaikkan siswa sama | Satu enrolmen dan satu penempatan yang sah; tidak ada duplikat. |
| DB05 | Kelulusan | Kelas XII program 4 tahun | Bisa naik XIII, tidak menjadi alumni otomatis. |
| DB06 | Kelulusan | Siswa program akhir dinyatakan lulus | `kelulusan` unik, profil alumni dibuat/ditautkan, akun lama dipakai. |
| DB07 | Alumni | Alumni lama tidak ada di master siswa | Dapat mendaftar/terverifikasi tanpa siswa fiktif. |
| DB08 | Riwayat alumni | Bekerja + kuliah bersamaan, lalu usaha | Riwayat per periode tetap utuh. |
| DB09 | Keuangan | Pindah kelas setelah tagihan diterbitkan | Nominal dan kelas snapshot tagihan tetap; saldo tak berubah. |
| DB10 | Keuangan | Lulus dengan tunggakan | Tagihan masih terhubung `siswa.id` lama dan bisa ditelusuri bendahara. |
| DB11 | Keuangan | Dua pembayaran paralel untuk tagihan yang sama | Tidak terjadi overpayment / kuitansi ganda. |
| DB12 | Aset | Ubah kondisi alat di Lab | Sarpras melihat aset yang sama dan perubahan terbaru. |
| DB13 | Stok | Dua pemakaian bahan bersamaan | Stok tidak negatif dan setiap pemakaian tercatat tepat sekali. |
| DB14 | Absensi | Izin siswa disetujui setelah terlanjur Alpa | Satu status kehadiran sah dan riwayat koreksi dapat dilihat. |
| DB15 | Surat | Buat surat masuk tanpa lampiran | Berhasil tersimpan dan muncul di agenda. |
| DB16 | PKL | Pembimbing industri memberi formulir nilai kertas | Guru input skor, identitas penilai tercatat, akun industri tidak diperlukan. |
| DB17 | Website | Susun menu/submenu dinamis | Urutan, visibilitas, arah tautan, dan perubahan publik konsisten. |
| PM01 | Izin | Guru biasa membuka pembayaran bendahara | Ditolak di route, Livewire action, dan unduhan. |
| PM02 | Data privat | Owner mencoba membuka catatan konseling | Ditolak tanpa otorisasi bantuan yang sah. |
| PM03 | Penugasan | Wali kelas A mencoba lihat siswa kelas B | Ditolak sesuai batas penugasan. |
| PM04 | Lisensi | Modul tidak dibeli / habis | Mutasi ditolak tanpa 500; arsip berizin tetap tersedia dan modul lain berjalan. |
| PM05 | Pendaftaran alumni | Pendaftar memakai NIS orang lain | Tidak memperoleh akses data siswa sebelum verifikasi. |
| BL01 | Proses massal | 600 siswa, 3 batch request paralel | Progres sesuai commit dan hasil benar, tidak ada proses ganda. |
| BL02 | Proses massal | Browser ditutup setelah sebagian selesai | Setelah login lagi, sisa proses bisa dilanjutkan. |
| BL03 | Proses massal | Lima item gagal | Sukses lain tetap tercatat; daftar gagal, alasan dan retry tersedia. |
| BL04 | Proses massal | Jaringan putus setelah server commit | UI cek status server sebelum retry; tidak membuat duplikat. |
| UX01 | UI | Double click tombol simpan | Hanya satu perubahan data; indikator loading tampil. |
| UX02 | UI | Hapus data penting/kelulusan | Modal konfirmasi eksplisit dan preview dampak tersedia. |
| UX03 | UI | Data input tidak valid | Field terkait menunjukkan pesan; isian lain tidak hilang. |
| SEC01 | Keamanan | Akses file sensitif dengan URL langsung | Ditolak tanpa login dan permission. |
| OPS01 | Backup | Simulasikan kegagalan DB pada staging | Backup berhasil dipulihkan, relasi/riwayat tidak rusak. |
| OPS02 | Rilis | Migrasi versi baru pada salinan data staging | Tidak menghilangkan atau menimpa data siswa/keuangan/alumni. |

Tes pada Bab 5 (`Sxx`, `Mxx`, `Axx`) dan Bab 9 (`Uxx`) **tetap berlaku**; tabel di sini menambahkan pemeriksaan teknis yang eksplisit dan tidak menggantikannya.

## 14.4 Release gate sebelum dipakai sekolah sungguhan

| Gerbang | Bukti yang harus ada |
|---|---|
| Alur inti benar | Tes Core/siklus siswa lulus; tes tagihan historis wajib saat Keuangan masuk rilis. Uji fondasi skema tidak dianggap penyelesaian modul Keuangan. |
| Hak akses aman | Setiap role diuji pada halaman, Livewire action, ekspor, dan API; tidak ada kebocoran data lintas peran. |
| Proses massal stabil | Tes paralel, pengulangan request, koneksi putus, browser ditutup, progres akurat, dan resume lulus. |
| Integritas DB | FK/unique terpasang, transaksi konsisten, audit perubahan sensitif tersedia. |
| Data sensitif terlindungi | HTTPS aktif, file privat, otorisasi unduh, keamanan akun dan registrasi alumni telah diuji. |
| Prasyarat infrastruktur (di luar PRD aplikasi) | Lingkungan hosting siap dan diamankan oleh penanggung jawab VPS; developer hanya memverifikasi aplikasi dapat dijalankan. |
| Backup nyata | Operator infrastruktur bertanggung jawab membuat dan menguji restore data. Developer memverifikasi integritas aplikasi pada salinan yang dipulihkan. |
| UAT sekolah | Admin, Tata Usaha, Guru, Bendahara, Koordinator PKL, serta petugas lain yang relevan berhasil menyelesaikan tugas utamanya. |
| Dokumentasi minimum | Panduan Admin Sekolah, langkah ekspor data, SOP koreksi transaksi, dan prosedur pemulihan tersedia. |
| Pilot | Satu SMK menggunakan alur utama, masalah kritis telah ditutup sebelum penjualan lebih luas. |

**Keputusan produksi:** tahap development boleh dimulai memakai PRD ini. Status **siap produksi** hanya diberikan setelah implementasi dan bukti seluruh gerbang relevan di atas telah lolos; tidak bisa disimpulkan dari PRD saja.

## 14.5 Yang sengaja tidak dibangun pada MVP

Tidak ada PPDB, UKK, absensi guru, modul perizinan tersendiri, akun pembimbing industri, chat internal, pembayaran gateway, AI, generator jadwal canggih, SSO lintas sekolah, multi-tenancy, domain/server pusat, **Queue**, **Scheduler**, maupun layanan background otomatis. Notifikasi cukup melalui status halaman/toast saat pengguna membuka aplikasi. Pengembangan baru ditambah hanya jika ada kebutuhan sekolah yang terkonfirmasi.

## 14.6 Upgrade aplikasi dan penambahan fitur yang aman

**Alur:** Siapkan Rilis → Periksa Kompatibilitas → Uji Upgrade dari Versi Lama → Persiapkan Pemulihan oleh Operator → Terapkan → Migrasi Bertahap → Smoke Test → Buka Layanan → Catat Hasil / Pulihkan bila Gagal.

- **Versi jelas:** setiap rilis memiliki tag source, versi aplikasi, versi skema, minimum PHP/MariaDB/Node serta versi Laravel/Livewire yang didukung. Lockfile disertakan dan instalasi memakai versi terkunci. Penambahan fitur dan upgrade mayor framework adalah rilis berbeda; jangan sekaligus mengubah arsitektur, skema dan framework tanpa kebutuhan teruji.
- **Upgrade sumber:** satu codebase baku untuk semua sekolah, konfigurasi sekolah di environment/database, tema pada resources dan data CMS; jangan mengedit source unik per pelanggan. Operator tidak menyalin seluruh folder release menimpa `.env`, storage privat, unggahan atau kunci aplikasi. Tidak ada auto-update via tombol web yang mengeksekusi shell/Composer.
- **Migrasi additive:** tambahkan tabel/kolom nullable/default aman dahulu, jalankan versi kompatibel, isi data lama secara bertahap, validasi, baru hapus struktur lama pada rilis berikutnya jika diperlukan. Tidak memakai `migrate:fresh`, truncate, seed reset atau down migration destruktif untuk upgrade data sekolah.
- **Backfill tanpa worker:** perubahan data besar memakai command manual yang dapat dilanjutkan atau batch HTTP admin berizin, dengan checkpoint/idempotensi dan hasil audit; tidak menambah Queue/Scheduler. Pekerjaan database tidak selesai hanya karena command mengembalikan exit 0; verifikasi jumlah/relasi.
- **Uji jalur nyata:** fresh install bukan bukti upgrade. Wajib uji dari rilis sebelumnya yang didukung pada salinan data sintetis berisi siswa/histori, lisensi campuran aktif/nonaktif, presensi, jurnal, pembayaran, aset, media dan tema. Tetapkan versi minimum jalur upgrade; versi lebih lama membutuhkan langkah antara yang tertulis.
- **Integritas:** ID/relasi, akun/peran/penugasan, status lisensi, snapshot tagihan, konfirmasi, arsip, nomor dokumen, tema dan unggahan harus tetap utuh. Modul nonaktif juga dimigrasikan skemanya dengan aman agar aktif ulang tidak error. Seeder katalog hanya upsert kode tetap tanpa mengubah pilihan lisensi atau isi sekolah.
- **Penerapan:** operator menyediakan backup/salinan pemulihan DB + storage sebelum perubahan. Developer menyediakan urutan perintah rilis (install dependency terkunci, build, maintenance singkat bila perlu, migrate noninteraktif, cache config/route/view, smoke test) dengan prasyarat dan estimasi berdasarkan uji. Tunggu transaksi aktif selesai; jangan mematikan request simpan kelas di tengah commit.
- **Cache/browser:** aset memiliki versi/hash; setelah rilis, tab lama yang mengirim versi payload tidak cocok mendapat pesan muat ulang dan tidak menyimpan data parsial. Invalidasi cache lisensi/menu setelah deploy; uji komponen Livewire dan semua tema terhadap aset baru.
- **Rollback realistis:** rollback source hanya boleh bila skema masih kompatibel dengan source lama. Jangan menjanjikan `migrate:rollback` selalu aman. Jika perubahan tidak kompatibel, gunakan perbaikan maju atau pemulihan DB/storage oleh operator dengan pencatatan transaksi setelah backup dan persetujuan dampak kehilangan data; aplikasi tetap ditutup sampai konsisten.
- **Bukti rilis:** catat versi awal/akhir, migrasi, perintah sebenarnya, hasil test, waktu henti, tindakan pemulihan dan isu tersisa pada TASK. Upgrade mayor mengikuti panduan resmi framework, audit dependency dan regresi fitur, tanpa menurunkan versi stack yang disepakati secara diam-diam.

## 14.7 Skenario penerimaan keputusan operasional terbaru

**Alur:** Siapkan Kondisi → Jalankan Aksi → Periksa Database dan UI → Cocokkan Hasil → Catat Bukti.

| ID | Skenario | Hasil wajib |
|---|---|---|
| OP01 | Guru membuat jurnal rapat/persiapan tanpa kelas, jadwal dan presensi | Kegiatan/hasil dapat dikirim sebagai jurnal hari itu; tidak diminta jadwal. |
| OP02 | 35 siswa tampil, dua S dan satu I dari wali; lainnya H; guru menyimpan | Seluruh 35 tersimpan sesuai status, ringkasan 32 H/2 S/1 I, bukan hanya siswa yang diubah radionya. |
| OP03 | Payload kehilangan satu siswa, ID ganda/tambahan, status invalid atau satu baris gagal DB | Seluruh pengiriman ditolak/rollback; tidak menyimpan 34 siswa dan mengaku sukses. |
| OP04 | Commit berhasil tetapi respons timeout; klik ulang/two-tab | Periksa token membuktikan hasil 35/35; retry tidak ganda, versi tab lama ditolak bila ada perubahan baru. |
| OP05 | Guru mengirim tanggal lampau/future atau halaman melewati tengah malam | Server menolak dengan pesan muat ulang; tidak membuat presensi bertanggal pilihan guru. |
| OP06 | Wali kelas mencatat S/I saat halaman guru sudah terbuka | Radio pada pemuatan berikutnya sesuai S/I; kiriman versi lama ditolak utuh dan diminta muat ulang. Tidak butuh pejabat kedua. |
| OP07 | Wali kelas mengonfirmasi rekap berbulan lalu | Tetap boleh dengan lingkup sah, tanpa tenggat; koreksi menyimpan audit dan konfirmasi ulang. |
| OP08 | Temuan selesai di wali atau diteruskan ke BK | Alur menyimpan penanggung jawab, klarifikasi/tindakan/hasil; tidak membuka catatan konseling privat ke pelapor. |
| OP09 | Guru pengganti, wali baru, delegasi berakhir atau ID kelas dimanipulasi | Hak objek dan masa berlaku diuji; tidak membuka semua kelas dan histori pelaku tetap utuh. |
| OP10 | Owner nonaktifkan modul saat request simpan/bulk sedang berjalan | Transaksi selesai utuh sebelum perubahan atau ditolak tanpa partial write; Core/modul lain tidak error, bukti simpan tetap bisa diperiksa. |
| OP11 | Aktif ulang setelah expiry dan setelah upgrade | Data, relasi, lisensi pilihan, media dan tema tetap; kesiapan skema dicek dan tidak seed reset. |
| OP12 | Editor mengganti lima tema berulang dan mencoba kode tema invalid | Publik mengikuti tema terpilih tanpa kehilangan konten/URL; invalid fallback aman; seluruh admin tetap desain sama. |
| OP13 | Upgrade fitur atau framework dari versi lama dengan data dan modul nonaktif | Migrasi aman, backfill dapat dilanjutkan, smoke/regresi lulus; kegagalan tidak menyebabkan source/skema tidak kompatibel dibuka ke pengguna. |

# 15. Kontrak route aplikasi dan koneksi database — wajib diikuti

**Alur bagian:** Buka Route → Validasi Identitas/Input → Otorisasi → Proses → Respons.

Bagian ini mengunci alamat URL **di setiap instalasi sekolah**. Contoh `smka.sch.id` hanya placeholder; tidak ada domain pusat, tidak ada instalasi sekolah lain pada VPS ini. **Path bersifat relatif terhadap domain sekolah**, bukan host layanan terpisah. Nama route Laravel harus stabil agar tautan dari menu, email reset password, cetak dan tes tidak berubah-ubah.

## 15.1 Peta URL dan akses minimum

| URL / pola route | Fungsi | Akses | Modul wajib aktif |
|---|---|---|---|
| `/` | Website publik jika aktif, jika tidak halaman informasi sekolah sederhana | Publik | `website` hanya untuk CMS lengkap |
| `/login`, `/forgot-password`, `/reset-password/{token}` | Autentikasi dan pemulihan akun | Publik dengan pembatasan percobaan | Core |
| `POST /logout` | Keluar dari akun; tidak menggunakan GET | Pengguna login + CSRF | Core |
| `/owner` | Dashboard dan lisensi lokal milik pemilik SaaS | Role `owner` + MFA | Core |
| `/owner/modul`, `/owner/langganan`, `/owner/pembayaran` | Aktivasi modul, masa berlaku, catatan pembayaran manual | Role `owner` + MFA | Core |
| `/admin` | Dashboard admin sekolah | Pengguna login dengan permission sekolah | Core |
| `/admin/siswa`, `/admin/kelas`, `/admin/tahun-ajaran` | Data induk akademik | Role/permission terkait | Core |
| `/admin/kenaikan-kelas`, `/admin/kelulusan` | Proses siswa massal dan kelulusan | Admin akademik berizin | Core |
| `/admin/alumni/verifikasi` | Verifikasi alumni lama | Petugas yang ditugaskan | Core (portal alumni aktif bila BKK atau Tracer aktif) |
| `/siswa` | Portal data dan layanan siswa | Siswa terautentikasi | Core + modul sesuai fitur |
| `/alumni` | Portal profil dan riwayat alumni | Alumni terverifikasi | Core + fitur portal terkait |
| `/alumni/daftar` | Registrasi mandiri alumni lama, tanpa PPDB | Publik dengan rate limit; hanya bila BKK/Tracer aktif | `bkk` **atau** `tracer` |
| `/admin/pkl` | Manajemen PKL | Permission PKL | `pkl` |
| `/admin/jurnal-guru` | Jurnal kegiatan dan hasil kerja harian | Permission Jurnal | `jurnal_guru` |
| `/admin/absensi-izin` | Absensi siswa dan izin siswa/guru | Permission Absensi/Izin | `absensi_izin` |
| `/admin/surat` | Surat masuk/keluar, template, arsip | Permission Surat | `surat` |
| `/admin/keuangan` | Tagihan, pembayaran, tunggakan | Permission Keuangan | `keuangan` |
| `/admin/bkk` | Lowongan dan lamaran alumni | Permission BKK | `bkk` |
| `/admin/sarpras` | Inventaris dan aset bersama | Permission Sarpras | `sarpras` |
| `/admin/bk` | Konseling, pembinaan terbatas | Permission BK khusus | `bk` |
| `/admin/jadwal` | Jadwal pelajaran | Permission Jadwal | `jadwal` |
| `/admin/tracer` | Tracer study dan statistik | Permission Tracer | `tracer` |
| `/admin/hubin` | Mitra industri dan kerja sama | Permission HUBIN | `hubin` |
| `/admin/supervisi` | Supervisi guru | Permission Supervisi | `supervisi` |
| `/admin/laboratorium` | Alat, stok bahan, peminjaman lab | Permission Lab | `laboratorium` |
| `/admin/pelanggaran` | Tata tertib, poin dan pembinaan | Permission Kesiswaan | `pelanggaran` |
| `/admin/website` | CMS, menu/submenu dinamis | Permission Editor Website | `website` |
| `/berkas/{berkas}/unduh` | Unduh dokumen privat melalui Controller | Role + policy per objek + lisensi bila terkait | Sesuai asal berkas |

**Kontrak routing:** route publik yang disebut di tabel (website, login/reset, registrasi alumni) mengikuti batasan publiknya; tidak diwajibkan login. Seluruh route privat dan aksi Livewire privat harus melewati `auth`, pemeriksaan role/permission **dan** lingkup objek (siswa/kelas yang ditugaskan), serta `module-aktif` jika masuk modul berbayar. Route owner wajib terpisah dari operator sekolah; `owner` **bukan** bypass untuk catatan BK, pembayaran siswa, data sensitif siswa. Gunakan HTTP method POST/PATCH/DELETE untuk mutasi dan validasi CSRF; **jangan melakukan mutasi melalui GET**. Alumni yang hanya memiliki riwayat tidak boleh melihat profil siswa lain. Nama path tidak menjadi mekanisme keamanan.

## 15.2 Konfigurasi Laravel dan MariaDB (lingkup aplikasi)

**Batas tanggung jawab:** PRD ini mengatur **kode aplikasi Laravel**, bukan pengadaan domain, DNS, Nginx, SSL, firewall, atau konfigurasi sistem operasi. Infrastruktur dikelola oleh penanggung jawab VPS dalam prosedur terpisah. Teknologi Nginx tetap menjadi bagian stack yang disepakati, tetapi tidak perlu dibuatkan konfigurasi server di PRD atau task pengembangan fitur.

- URL pada tabel di atas adalah **path relatif**, misalnya `/admin/siswa`, dan tidak mengunci nama domain. Pakai **named routes** Laravel (`route('...')`), bukan URL host yang ditulis langsung dalam komponen.
- `APP_URL` adalah **nilai lingkungan yang disediakan saat deployment**; developer aplikasi cukup mengonsumsinya secara benar. Jangan membuat pengaturan domain/DNS/Nginx dari dashboard sekolah.
- Koneksi MariaDB didefinisikan melalui environment/config Laravel. Jangan hardcode kredensial di source code atau commit `.env`.
- Setiap instalasi berisi **data satu sekolah saja**, satu database MariaDB, tanpa `tenant_id` atau komunikasi ke server pusat.
- Jika modul Website tidak aktif, route `/` menampilkan halaman informasi sekolah sederhana; halaman `/admin` tetap berfungsi untuk modul yang aktif.

**Variabel aplikasi minimum pada `.env.example` (hanya nama dan contoh yang tidak rahasia):**

```dotenv
APP_NAME="Aplikasi SMK"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=mariadb
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=smk_app
DB_USERNAME=smk_app
DB_PASSWORD=

SESSION_DRIVER=database
FILESYSTEM_DISK=local
```

- Nilai produksi (`APP_ENV=production`, `APP_DEBUG=false`, `APP_URL` nyata, password database, kunci aplikasi) **disuplai secara aman oleh proses deployment**, tidak ditentukan di PRD.
- `APP_KEY` harus unik per instalasi; key/token/kata sandi tidak boleh tersimpan dalam dokumentasi/repository.
- Tanggal efektif dan laporan akademik memakai zona waktu sekolah (`Asia/Jakarta`); timestamp disimpan dalam UTC dan dikonversi saat ditampilkan; kolom tanggal efektif memakai tanggal lokal dan diuji pada batas pergantian hari.
- Database wajib memiliki **foreign key, unique constraint, transaksi, dan indeks** sebagaimana Bab 12/17. Hak akses akun DB mengikuti prinsip minimum; pembukaan port dan pengamanan server termasuk lingkup operator VPS, bukan tugas pengembangan modul.

# 16. Ketentuan keamanan dan privasi — minimum untuk produksi

**Alur bagian:** Kenali Data Sensitif → Batasi Akses → Uji Serangan/Kesalahan → Audit → Perbaiki.

Ketentuan berlaku pada seluruh modul tanpa menambah proses yang mengganggu pengguna biasa. Keamanan diperiksa di server, bukan hanya disembunyikan pada menu HTML.

| Area | Aturan wajib | Cara menguji |
|---|---|---|
| Akun dan session | Hash password melalui Laravel; regenerasi session saat login; invalidasi session saat logout; CSRF aktif; cookie Secure, HttpOnly, SameSite sesuai kebutuhan | Login/logout, session fixation, CSRF ditolak |
| Owner | Hanya akun owner hasil provisioning aman (tidak ada password default); **MFA wajib untuk akses owner pada rilis produksi**; pertimbangkan VPN/pembatasan IP | Role lain gagal masuk `/owner`; MFA teruji |
| Role dan objek | Middleware + otorisasi objek untuk route, Livewire method, export/import, download; guru hanya data siswa yang ditugaskan; owner bukan admin data BK | Uji IDOR: ubah ID siswa/file di URL dan tetap ditolak |
| Lisensi | Cek modul aktif di server setiap mutasi/akses fitur terkait; kedaluwarsa tidak menghapus data dan tidak membocorkan data lama | Mutasi/fitur operasional nonaktif ditolak; arsip baca/ekspor tetap sesuai hak (§13.5) |
| Input dan XSS | Validasi server-side, query parameterized/Eloquent; output di-escape; HTML berita diproses dengan allowlist sanitasi sebelum ditampilkan | Payload XSS/SQLi tidak dieksekusi |
| Upload | Validasi MIME berdasarkan isi, ekstensi, ukuran maksimum, dan otorisasi; nama file acak; simpan file siswa/BK/keuangan di disk privat **di luar public** | Upload file salah tipe ditolak; direct URL tidak membuka file |
| Unduh | Controller unduhan dengan otorisasi berdasarkan dokumen dan pemilik/lingkup; file tidak dapat diakses hanya dengan menebak path | File milik role lain ditolak 403 |
| Data sensitif | Catatan BK, riwayat pelanggaran, data keluarga, pembayaran, dan informasi pribadi alumni dibatasi; status menikah opsional dan privat | Audit permission per field dan laporan agregat |
| Registrasi alumni | Verifikasi manual sebelum taut akun ke siswa lama; rate limit register/login/reset; cegah email enumeration dan klaim NIS orang lain | Akun palsu tidak dapat mengambil profil alumni lain |
| Transaksi penting | MariaDB transaction, lock, unique constraint, idempotency, audit koreksi; tidak `cascade delete` histori penting | Tes dua request bersamaan untuk tagihan/kelulusan |
| Monitoring & log | Log error seperlunya tanpa token/password/data BK; hak akses log dibatasi. Enkripsi dan lokasi backup di luar VPS merupakan tanggung jawab tim infrastruktur | Cari data sensitif dalam log; developer memverifikasi integritas aplikasi setelah restore oleh operator |
| Dependency aplikasi | Perbarui dependency Laravel/Livewire yang terdampak advisory, audit paket PHP/JS | `composer audit`, `npm audit`, regresi fitur utama |
| Browser | CSRF, XSS/sanitasi CMS, cookie session yang aman, tidak membocorkan data privat; kebijakan header harus kompatibel Livewire | Tes interaksi frontend dan keamanan aplikasi |

**Mekanisme error:** jangan pernah tampilkan stack trace, SQL query, token, kredensial, atau path internal kepada pengguna; simpan detail teknis pada log terbatas. Modal kesalahan menampilkan pesan sederhana dan opsi ulangi/periksa status. Upload HTML/JS sebagai dokumen harus ditolak atau diperlakukan sebagai unduhan non-eksekusi, bukan ditampilkan di origin sekolah.

**Privasi:** sekolah dan penyedia layanan harus menetapkan tujuan pengolahan data, pembagian hak akses, retensi, prosedur ekspor/penghapusan, serta tanggung jawab bila ada insiden. Sistem tidak boleh mempublikasikan NISN, nilai, daftar pelanggaran, atau detail pribadi tanpa dasar otorisasi yang sah. Jangan menjadikan fitur opsional seperti status pernikahan alumni sebagai kolom wajib. Kebijakan ini perlu ditinjau terhadap ketentuan perlindungan data Indonesia sebelum rilis.

**Lisensi lokal:** karena tidak ada server pusat, perubahan lisensi dikendalikan pada instalasi masing-masing. Pemeriksaan database lokal adalah pembatasan aplikasi; tidak boleh dijanjikan anti-tamper jika administrator mempunyai kendali root/server/source code.

# 17. Kontrak integritas database dan checklist kesiapan rilis

**Alur bagian:** Pasang Constraint → Uji Transaksi → Uji Retry → Verifikasi Histori → Blokir Rilis bila Gagal.

## 17.1 Kontrak tabel, kunci unik, dan referensi yang tidak boleh hilang

Bab 12 mendeskripsikan tabel utama; migration nyata harus mempertahankan aturan ini:

| Area | Constraint minimal (di MariaDB, bukan sekadar validasi UI) | Dampak bisnis |
|---|---|---|
| `siswa` | `PRIMARY KEY(id)`, `UNIQUE(nis)`, `UNIQUE(nisn)` dengan nilai kosong disimpan sebagai NULL | Siswa tidak ganda; NISN opsional |
| `kepesertaan_siswa` | `FK siswa_id`, `FK tahun_ajaran_id`, `UNIQUE(siswa_id, tahun_ajaran_id)` | Maksimum satu enrolmen per siswa per tahun |
| `penempatan_kelas_siswa` | FK enrolmen + FK kelas; validasi tanggal tidak overlap **dalam transaksi dengan lock enrolmen** | Riwayat mutasi kelas utuh; query current class berdasarkan tanggal |
| `kelulusan` | `FK siswa_id`, `UNIQUE(siswa_id)` | Kelulusan tidak dapat ganda |
| `profil_alumni` | `FK siswa_id NULL`, `FK pengguna_id NULL`, masing-masing UNIQUE bila tidak null | Alumni baru tersambung siswa; alumni lama tanpa siswa fiktif |
| `tagihan` | FK siswa, tarif snapshot, `UNIQUE(siswa_id, jenis_biaya_id, kunci_periode)` dengan kunci_periode yang membedakan tahun/periode | Tagihan tidak dibuat dua kali |
| `pembayaran` | FK tagihan, `UNIQUE(nomor_kuitansi)`; transaksi tagihan terkunci sebelum menambah pembayaran | Tidak overpayment atau kuitansi ganda |
| `aset` | Identitas aset unit unik; Lab merujuk `aset.id` yang sama; bukan tabel aset duplikat | Sarpras dan Lab konsisten |
| `sesi_pelajaran` | FK kelas/mapel/guru, UNIQUE (`kelas_id`, `tanggal`, `jam_ke`); waktu sesi tidak bertabrakan untuk kelas sama | Satu sesi sah per jam kelas, jadwal opsional |
| `kehadiran_siswa` | FK sesi/siswa/penempatan; UNIQUE (`sesi_pelajaran_id`, `siswa_id`); konfirmasi hanya wali kelas binaan | Presensi per jam tidak tertimpa; koreksi diaudit dan dikonfirmasi ulang |
| `item_operasi_massal` | `UNIQUE(operasi_id, kunci_entitas)`; item bersifat idempotent | Retry/dua tab tidak menggandakan mutasi |
| `peran_pengguna`, `hak_akses_peran` | UNIQUE pasangan masing-masing FK | Peran/hak tidak terpasang berkali-kali |

Gunakan `RESTRICT` atau strategi arsip/koreksi untuk relasi historis penting, bukan `ON DELETE CASCADE` pada tagihan, graduation, enrolmen atau data siswa. Pembayaran yang sudah sah tidak boleh dihapus; gunakan void/koreksi dengan audit. Satu sekolah hanya memiliki satu `profil_sekolah` secara logis (enforce via instalasi/seed, bukan `sekolah_id` berulang).

## 17.2 Kewajiban proses massal tanpa worker

`operasi_massal` mencatat operasi, filter yang sudah disnapshot, total, status, owner, dan waktu. `item_operasi_massal` menyimpan entity key, status, hasil, attempt, timestamp, `token_klaim`/`klaim_berakhir_pada` untuk klaim atomik. Status item hanya `menunggu`, `diproses`, `berhasil`, `sudah_diproses`, `dilewati`, `gagal` sesuai transition sah.

- Maksimum **2–3 request HTTP paralel** per operasi, setiap request memproses paling banyak **10–25 item**; konfigurasi dapat diturunkan bila VPS tidak kuat.
- Browser menjalankan permintaan batch; server **tidak menjalankan task di background**. Jeda berarti tidak mengirim batch baru; request terlanjur berjalan boleh commit.
- Saat browser tertutup, lease kedaluwarsa hanya ditinjau saat user membuka kembali proses atau ketika request baru masuk (lazy recovery), **bukan lewat scheduler**.
- Satu item diklaim dengan locking/unique key, diberi lease dan token; commit hasil hanya jika token sesuai. Timeout **tidak otomatis berarti gagal**, status server harus dibaca sebelum retry.
- Mutasi inti tiap item dilindungi constraint unik dan transaksi sehingga item yang lease-nya diklaim ulang tidak dapat menyebabkan mutasi kedua. Untuk pembayaran, sekolah harus memvalidasi transaksi manusia sebelum input; jangan memakai retry generik untuk membuat pembayaran tanpa kunci idempotensi.
- Persen = jumlah status terminal (`berhasil + sudah_diproses + dilewati + gagal`) / total snapshot, bukan countdown buatan. Tampilkan persentase, hitungan berhasil/gagal/lewat, dan sisa; 100% tidak otomatis berarti semua berhasil.

## 17.3 Gerbang keamanan aplikasi, routing, database, dan rilis (blokir rilis bila gagal)

| Pemeriksaan wajib | Bukti minimal |
|---|---|
| Semua URL pada Bab 15 dan akses role/lisensi | Feature tests untuk tamu, pengguna salah role, lisensi nonaktif, URL langsung dan Livewire action |
| Konfigurasi DB dan koneksi | koneksi MariaDB dikonfigurasi aman lewat environment; migration FK/UNIQUE dan transaksi concurrent lulus (pengamanan host database oleh tim infrastruktur) |
| Routing Laravel | Semua named routes, redirect, reset password, dan tautan bekerja pada host yang disediakan deployment; tidak mempercayai Host input mentah |
| Akses owner dan akun | MFA owner, rate limit, CSRF, session aman, tidak ada password default |
| Upload, unduh, dan privasi | MIME/ukuran benar, file privat tak bisa diakses langsung, data BK tidak bocor |
| Progress batch dan resume | 100–1.000 siswa sintetis diuji, dua browser dan koneksi putus tanpa duplikat |
| Backup di luar VPS dan restore | Hasil restore oleh operator telah diverifikasi dari aplikasi: data siswa, tagihan, aset, dan dokumen bisa dibaca tanpa ketidaksesuaian |
| Deployment | Aset dibuild, migrasi diuji pada salinan, `APP_DEBUG=false`, dependensi dan permission aman |
| UAT sekolah | Admin, TU, Waka, Guru, Bendahara, BK menuntaskan tugas yang relevan dengan modul rilis |

**Status file:** ini adalah spesifikasi dan checklist, belum bukti aplikasi berhasil dibuat. Status *production ready* tidak boleh diberikan hanya karena PRD/`TASK.md` sudah lengkap.


# 18. Gerbang penyelesaian pekerjaan developer (App-Only)

**Alur bagian:** Kumpulkan Bukti → Periksa Cakupan Rilis → UAT → Putuskan Handover/NO-GO.

PRD adalah spesifikasi, bukan bukti aplikasi berfungsi. Status **READY FOR APP HANDOVER** hanya boleh diberikan jika semua kriteria berikut mempunyai bukti tes, URL layar atau nama test, dan catatan penguji:

1. Seluruh modul yang dinyatakan masuk rilis sudah berjalan dari tampilan sampai MariaDB (bukan mock data HTML), dilengkapi tes role, lisensi, dan kegagalan input. Modul yang belum siap **tidak boleh dipasarkan sebagai aktif**.
2. Core System menangani tambah/impor siswa, siswa pindah kelas, kenaikan kelas massal, kelas XII→XIII untuk program empat tahun, kelulusan kelas akhir, alumnisasi tanpa mengganti identitas, dan alumni lama registrasi/verifikasi mandiri.
3. Tagihan/cicilan/riwayat pembayaran mempertahankan snapshot tarif dan kelas saat diterbitkan; identitas siswa dan alumni tetap terhubung; perubahan historis yang salah memakai koreksi terotorisasi.
4. Sarpras dan Laboratorium berbagi master aset; perizinan siswa+guru satu modul dengan absensi siswa (tanpa absensi guru); PKL menerima input nilai dari guru tanpa akun industri; surat boleh tanpa file; menu CMS dinamis.
5. Pemrosesan massal 600 siswa sintetis dan retry/dua browser/koneksi putus tidak menghasilkan duplikasi. Persen progress sesuai data commit; tombol Jeda hanya menghentikan pengiriman batch baru.
6. Semua layar penting menampilkan status loading, pesan validasi, konfirmasi modal sebelum tindakan berisiko, modal ringkasan hasil, empty state, dan error yang tidak mengekspos detail sensitif.
7. Uji autentikasi, MFA owner, permission object-level, session, CSRF, IDOR, sanitasi CMS, upload privat, dan unduh berizin lulus. Dependency audit tidak menunjukkan kerentanan kritis yang belum ditangani.
8. `migrate:fresh` hanya pada **database uji**, migration upgrade pada data sintetis, release rollback, dan seeder aman diuji. Jangan menjalankan operasi destruktif pada database produksi.
9. UAT Admin, Waka Kurikulum, Guru, TU, Bendahara, BK, dan BKK (sesuai modul rilis) lulus serta bug penghambat ditutup.
10. Handover mencakup source/tag rilis, `.env.example` aman, dokumentasi setup **aplikasi saja**, daftar konfigurasi yang diperlukan dari operator, perintah migration/build aplikasi, hasil test, dan isu tersisa. Tidak ada konfigurasi domain/Nginx/SSL/VPS di dalam task developer.

Detail bukti dan keputusan rilis ada di bagian Checklist kesiapan rilis aplikasi pada `TASK.md`. Kriteria modul berlaku untuk modul dalam cakupan rilis; modul di luar cakupan tetap nonaktif. Core, keamanan, dan proses massal yang digunakan tetap wajib lulus. Status **GO LIVE** untuk sekolah hanya dapat diputuskan setelah tim infrastruktur menyatakan platform pendukung siap melalui proses terpisah.


---

## Peta desain HTML

Semua file di `prototype/` hanya merupakan desain dan simulasi antarmuka. CSS dan JavaScript tertanam di setiap HTML; tidak memerlukan build atau backend. Buka `prototype/index.html` untuk mulai. Panduan pengerjaan, checklist rilis, dan progres ada di `TASK.md`.


HTML berikut adalah **simulasi frontend**, bukan aplikasi production. Data dan statusnya adalah contoh. Prioritas implementasi tetap `PRD.md`.

| File HTML | Alur yang didemokan |
|---|---|
| `index.html` | Halaman indeks prototipe |
| `01-owner.html` | Owner lokal, aktivasi modul, masa lisensi |
| `02-dashboard.html` | Dashboard sekolah berdasarkan modul aktif |
| `03-siswa.html` | Data induk siswa, pencarian |
| `04-kenaikan.html` | Pilih massal, konfirmasi, progress |
| `05-kelulusan.html` | Penetapan lulus, status alumni |
| `06-alumni.html` | Pendaftaran alumni lama, verifikasi, riwayat |
| `07-pkl.html` | Jurnal dan input nilai pembimbing industri melalui guru |
| `08-absensi.html` | Absensi siswa dan izin siswa/guru |
| `09-keuangan.html` | Tagihan, pembayaran, tunggakan |
| `10-surat.html` | Surat dengan atau tanpa lampiran |
| `11-sarpras.html` | Aset bersama Sarpras/Lab |
| `12-website.html` | CMS dan menu dinamis |
| `13-pelanggaran.html` | Poin kedisiplinan dan validasi |
| `14-modul-lain.html` | Ringkasan modul lain |
| `tema-01-klasik.html` | Tema publik Sekolah Klasik |
| `tema-02-modern.html` | Tema publik Modern Minimalis |
| `tema-03-vokasi.html` | Tema publik Vokasi Industri |
| `tema-04-berita.html` | Tema publik Portal Informasi |
| `tema-05-kreatif.html` | Tema publik Kreatif Dinamis |

**Catatan:** prototipe HTML bukan pengganti Blade/Livewire, database, autentikasi, permission, atau unit test.
