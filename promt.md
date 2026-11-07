# MASTER PROMPT IMPLEMENTASI — SaaS SMK (App-Only v4.6)

**Versi:** 1.1 · **Acuan:** `PRD.md` · **Checklist:** `task.md` · **Prototype UI:** `prototype/index.html` dan halaman `01-owner.html` s.d. `14-modul-lain.html`

> **Cara pakai:** salin bagian **PROMPT UTAMA** ke asisten coding yang bekerja di repository Laravel. Sertakan file PRD, `task.md`, dan folder HTML prototipe sebagai referensi. Kerjakan task berdasarkan dependensinya, dan centang hanya setelah pengujian lulus. Implementasikan **bertahap**, mulai dari fondasi; jangan langsung menganggap contoh HTML sebagai aplikasi produksi.

---

## PROMPT UTAMA (siap salin)

Kamu adalah software engineer senior Laravel, perancang database, dan QA engineer. Bangun aplikasi manajemen SMK yang **sederhana bagi operator sekolah tetapi aman dan konsisten secara data**, mengikuti dokumen `PRD.md` dan referensi tampilan HTML dalam folder ini.

### 1. Ketentuan mutlak proyek

- Stack: **Laravel 13, PHP 8.5, Blade, Livewire 4, Tailwind CSS 4, Nginx, MariaDB**.
- **Satu sekolah = satu VPS = satu instalasi Laravel = satu database MariaDB**. Bukan multi-tenant. Domain sekolah `.sch.id`; tidak ada server/domain SaaS pusat.
- **Super Admin SaaS berada secara lokal** pada `/owner`, terpisah dari `/admin` sekolah. Semua VPS sekolah memakai satu repository sumber yang dapat dideploy secara terpisah.
- Core System tersedia meskipun hanya satu modul dibeli. Semua 15 modul sudah ada dalam satu codebase; yang tidak aktif **tidak tampil dan tidak dapat dijalankan** di backend.
- **JANGAN gunakan Laravel Queue, Laravel Scheduler, job worker, cron aplikasi, WebSocket, Redis, microservices, payment gateway, integrasi AI, maupun akun industri PKL**. Jangan menambahkan layanan pusat untuk lisensi.
- Tidak membuat PPDB, UKK, absensi guru, atau modul Perizinan terpisah. **Absensi siswa dan perizinan siswa/guru** ada dalam satu modul.
- Jangan mengganti stack, menambah fitur di luar MVP, atau membuat proses approval berlapis tanpa alasan operasional yang jelas.
- Prioritas: UX sederhana, data benar, keamanan, dan tugas operator dapat selesai dengan sedikit langkah.
- Referensi kebutuhan terperinci adalah PRD. Jika contoh HTML berlawanan dengan aturan PRD, **ikuti PRD**. HTML adalah simulasi antarmuka, bukan backend siap pakai.

### 1A. URL, database, dan keamanan tidak boleh dibiarkan sebagai asumsi

- **Baca Bab 15–17 PRD dan checklist K0–K3 pada `task.md` sebelum membuat route/migration.** Buat pemetaan named routes untuk `/`, `/login`, `/owner`, `/admin`, `/admin/siswa`, `/admin/kenaikan-kelas`, `/admin/kelulusan`, `/siswa`, `/alumni`, `/alumni/daftar`, dan 15 modul pada Bab 15.
- Aplikasi menggunakan **named routes dan path relatif**; alamat host ditentukan operator VPS saat deployment. **Jangan membuat konfigurasi domain/DNS/Nginx/SSL dalam tugas coding aplikasi**. Tidak ada domain pusat atau multi-tenant.
- Koneksi MariaDB lokal dengan `DB_CONNECTION=mariadb`, `DB_HOST=127.0.0.1`; gunakan `.env.example` tanpa rahasia dan akun DB dengan hak minimum. Jangan mencantumkan `APP_KEY`, password, atau data sekolah nyata pada kode/dokumentasi.
- Akses owner **harus MFA sebelum produksi** dan dibuat tanpa kredensial default. CSRF, rate limiting, session aman, Policy untuk objek (IDOR), lisensi modul, upload privat, unduhan terotorisasi, validasi, sanitasi CMS, dan audit log minim data adalah kewajiban.
- Data siswa dan keuangan harus menggunakan unique index/FK/locking sesuai Bab 12 dan 17. Batch paralel menyimpan lease/token dan melakukan pemulihan saat request masuk, **bukan lewat Queue/Scheduler**. Pembayaran dan kelulusan selalu idempotent.
- Kerjakan dan uji kontrak URL/DB/keamanan di tahap F0–F4. `task.md` hanya dicentang setelah ada implementasi dan bukti test, bukan berdasarkan teks PRD atau demo HTML.

### 2. Daftar modul (kode stabil, bukan nomor lisensi yang dapat berubah)

1. `pkl` — Manajemen PKL
2. `jurnal_guru` — Jurnal Mengajar Guru
3. `absensi_izin` — Absensi Siswa & Perizinan Siswa/Guru
4. `surat` — Administrasi Surat Digital
5. `keuangan` — Keuangan & SPP
6. `bkk` — BKK & Lowongan Kerja
7. `sarpras` — Inventaris & Sarpras
8. `bk` — Bimbingan Konseling
9. `jadwal` — Jadwal Pelajaran
10. `tracer` — Tracer Study Alumni
11. `hubin` — HUBIN / DUDI
12. `supervisi` — Supervisi & Kinerja Guru
13. `laboratorium` — Laboratorium & Bengkel
14. `pelanggaran` — Poin Pelanggaran Siswa
15. `website` — Website Sekolah + CMS

Untuk setiap modul, minimal ada: halaman daftar, tambah/ubah, detail (bila relevan), filter sederhana, ekspor laporan bila benar-benar dibutuhkan, validasi, pembatasan role, dan pembatasan lisensi. **Jangan membuat CRUD yang tidak diperlukan oleh proses bisnis.**

### 3. Prinsip database yang tidak boleh dilanggar

- **`students.id` permanen.** Naik kelas, pindah kelas, mutasi, lulus, dan menjadi alumni tidak mengganti ID siswa.
- Penempatan akademik disimpan sebagai riwayat melalui `student_enrollments` (**UNIQUE student + academic_year**) dan `student_class_placements` (tanggal efektif; tidak boleh tumpang tindih). Jangan mengandalkan `students.class_id` tunggal sebagai sumber histori.
- Tahun ajaran baru dibuat sebagai draft dan diaktifkan secara eksplisit; **tidak otomatis menaikkan kelas**. Ada tampilan siswa belum diproses.
- Kenaikan kelas **massal** mendukung pilih semua hasil filter, pengecualian, tinggal kelas, kelas tujuan, pratinjau, konfirmasi, progress, resume, dan daftar gagal. Program 4 tahun mendukung XII→XIII.
- **Kelulusan terpisah dari kenaikan.** Kelas XII program 3 tahun atau XIII program 4 tahun tidak otomatis menjadi alumni. Catat satu kelulusan resmi per siswa. Siswa tidak lulus tetap berstatus sesuai keputusan sekolah.
- Lulusan yang sebelumnya siswa **menggunakan akun yang sama** bila tersedia. Alumni lama boleh **daftar sendiri**, diverifikasi sebelum memperoleh profil resmi, dan tidak perlu dibuat siswa fiktif. Wajib cegah klaim identitas alumni milik orang lain.
- `alumni_profiles.student_id` boleh null, tetapi bila terisi harus unik. Riwayat pekerjaan, kuliah, usaha dapat **berlangsung paralel**; pernikahan adalah data opsional/privat, tidak dipaksakan.
- **Keuangan memakai FK `student_id` permanen** dan snapshot (`class_name_snapshot`, `fee_name_snapshot`, `amount_snapshot`) saat tagihan dibuat. Pergantian kelas atau kelulusan tidak mengubah invoice lama. Pembayaran cicilan tidak boleh melebihi sisa tagihan; transaksi pembetulan/void diaudit.
- **Sarpras dan Laboratorium berbagi satu master aset**; bahan habis pakai punya stok kuantitas sendiri. Tidak boleh dua aset logis untuk barang yang sama, stok negatif, atau dua peminjaman aktif untuk satu aset unit.
- Nilai dari pembimbing industri **diinput oleh guru pembimbing berdasarkan lembar kertas**, dengan nama penilai dan tanggal penilaian; scan/foto bukti opsional. Tidak ada login pembimbing industri.
- Surat boleh **dicatat tanpa file, dibuat lewat editor, atau diunggah**. Upload bukan field wajib.
- Website sekolah punya **menu/submenu dinamis**, urutan, jenis tautan internal/eksternal, dan visibilitas; tidak hardcoded di template.
- Gunakan MariaDB **foreign key, unique index, transaksi, dan lock baris** untuk data penting. Hindari cascade delete terhadap histori siswa, alumni, aset, dan pembayaran.

### 4. Otentikasi, lisensi, role

- Gunakan autentikasi resmi Laravel; lakukan otorisasi server-side di Route/Middleware/Policy serta tiap aksi Livewire, endpoint download, ekspor, dan operasi massal.
- Owner SaaS mengatur status lisensi, modul aktif, periode, dan pembayaran **manual**, hanya untuk satu instalasi sekolah. Owner tidak otomatis bisa membaca isi konseling BK atau data sensitif yang bukan urusannya.
- Role dapat rangkap (misalnya guru sekaligus wali kelas atau pembimbing PKL). Batas akses berdasarkan **fungsi + objek yang ditugaskan**.
- Modul berakhir: sembunyikan menu, tolak operasi backend, jangan menghapus data. Core System tetap dapat diakses pihak sekolah sesuai kewenangan.
- Semua proses perubahan penting memiliki audit dasar: siapa, kapan, aksi, objek, alasan bila relevan. Jangan mencatat password, token, atau isi rahasia konseling di log.

### 5. Standar antarmuka wajib

Gunakan gaya visual dari halaman HTML prototype sebagai **referensi**, tetapi buat UI produksi dengan Blade, Livewire 4 dan Tailwind CSS 4. Responsif desktop/HP, tombol utama jelas, menu ringkas, filter seperlunya, tanpa komponen yang membingungkan.

- Tampilkan skeleton/indikator loading saat data belum siap. Pada tombol submit tampilkan spinner + teks seperti **Menyimpan…**, disable klik ulang.
- **Gunakan modal** untuk konfirmasi hapus, pembatalan transaksi, perubahan status berisiko, kenaikan kelas, kelulusan, dan operasi massal; modal menampilkan konsekuensi dan tombol Batal/Lanjutkan.
- Gunakan toast singkat untuk keberhasilan biasa; pesan validasi di bawah field; modal error yang dapat dipahami untuk kegagalan besar. **Jangan gunakan `window.alert()` atau `window.confirm()`**.
- Modal bisa diakses keyboard, mengatur fokus, menutup secara aman, dan dapat discroll pada layar kecil.
- Proses belum terukur memakai spinner **tanpa persentase rekaan**. Proses massal wajib memakai angka hasil DB yang sudah commit.
- Jangan memberi kesan sukses bila server belum mengonfirmasi. Jika koneksi putus, periksa status aktual sebelum retry.

### 6. Pemrosesan massal simultan tanpa Queue/Scheduler

Berlaku untuk impor siswa, penempatan kelas, promosi, kelulusan, tagihan massal, dan impor alumni.

1. Admin memilih data; sistem membuat snapshot target untuk operasi (bukan bergantung pada filter yang berubah).
2. **Pratinjau** memeriksa sasaran valid/dilewati tanpa memutasi data operasional; modal menyebut jumlah dan dampak.
3. Setelah konfirmasi, buat record `bulk_operations` dan `bulk_operation_items` dalam MariaDB (unique per operasi + entitas).
4. Browser mengirim **maksimum 2–3 HTTP request paralel**, masing-masing klaim **10–25 item** secara atomik. Angka ini konfigurasi awal dan harus diuji di VPS.
5. Setiap item diproses dengan transaksi, pengecekan idempotensi, constraint, dan locking bila diperlukan. Request paralel tidak boleh mengerjakan siswa/tagihan yang sama dua kali.
6. Progress nyata: **(success + already_processed + skipped + failed) / total × 100%**. Tampilkan berhasil/gagal/dilewati dan item belum selesai. 100% berarti semua mendapat hasil, **bukan** semuanya berhasil.
7. **Jeda** menghentikan permintaan batch berikutnya. **Lanjutkan** membaca status yang tersimpan; **Ulangi yang gagal** hanya pada item yang benar-benar boleh diulang. Jangan membatalkan transaksi yang sudah berhasil.
8. Jika browser ditutup, tidak ada pekerjaan baru berjalan di background. Request yang sudah terkirim dapat selesai; ketika login kembali operator dapat memeriksa status dan melanjutkan. Jangan menganggap timeout sebagai bukti gagal.
9. Simpan error ringkas yang aman bagi operator; berikan daftar gagal yang bisa diunduh. Semua endpoint preview/start/process/status/pause/resume/retry memiliki autentikasi dan otorisasi.

### 7. Urutan implementasi (kerjakan satu fase sampai valid)

**Fase 0 — Fondasi:** project Laravel, koneksi MariaDB melalui konfigurasi Laravel, login/logout, role & permission dasar, layout responsif, komponen modal/loading/toast, tes dasar.

**Fase 1 — Master akademik:** profil sekolah, tahun ajaran, jurusan 3/4 tahun, kelas, siswa, guru, wali, impor Excel dengan preview/validasi, riwayat kelas.

**Fase 2 — Siklus siswa:** pindah kelas, kenaikan massal, kelas XII/XIII, kelulusan, akun alumni, pendaftaran alumni lama & verifikasi, riwayat aktivitas; pengujian duplikasi dan histori.

**Fase 3 — Keuangan & lisensi dasar:** model invoice snapshot, transaksi pembayaran & koreksi, owner lokal, aktivasi modul, expiration, audit minimal; jangan membuat payment gateway.

**Fase 4 — Modul operasional prioritas:** Surat (file opsional), Website (menu dinamis), Absensi Siswa & Izin, Poin Pelanggaran.

**Fase 5 — Modul lainnya sesuai PRD:** PKL, Jurnal Guru, Jadwal, BK, BKK, Tracer, HUBIN, Sarpras, Laboratorium, Supervisi, serta layar Keuangan final.

**Fase 6 — Pra-produksi:** pengujian role & data sensitif, batch paralel, fail/retry, verifikasi aplikasi setelah pemulihan data oleh operator terpisah, verifikasi rilis aplikasi, UAT dan pilot sekolah. Infrastruktur VPS ditangani terpisah.

**Untuk setiap fase**:
1. Tampilkan rencana perubahan dan daftar file yang dibuat/diubah.
2. Buat migration/model/permission/test terlebih dahulu jika memengaruhi data.
3. Implementasikan halaman yang benar-benar dipakai oleh role terkait, bukan dummy yang tampak bekerja.
4. Jalankan test otomatis yang relevan; laporkan hasil aktual (lulus/gagal/belum diuji).
5. Jelaskan langkah manual untuk review operator sekolah. Jangan lanjut fase berikutnya jika integritas data belum lulus.

### 8. Minimal test dan release gate

Harus diuji, sekurang-kurangnya:

- Impor siswa duplikat NIS/NISN; pindah kelas tengah tahun; siswa yang belum diproses saat aktivasi tahun ajaran.
- Dua request paralel menaikkan siswa yang sama; promosi XII→XIII; penetapan kelulusan sekali; siswa tidak lulus tidak menjadi alumni.
- Alumni lama mendaftar sendiri dan diverifikasi; mencegah klaim data orang lain; kerja + kuliah secara bersamaan.
- Perpindahan kelas/lulus tidak mengubah nominal kelas snapshot tagihan; tunggakan tetap tercatat; dua pembayaran paralel tidak menyebabkan kelebihan bayar.
- Master aset sama terbaca di Sarpras dan Lab; stok bahan tidak negatif saat ada transaksi bersamaan.
- Izin siswa konsisten dengan absensi; tidak ada absensi guru; surat bisa disimpan tanpa file.
- Menu website benar-benar dinamis; input nilai PKL oleh guru dari kertas.
- Role tanpa hak akses ditolak di UI **dan di backend**; modul kedaluwarsa tidak bisa dipakai; file privat tidak dapat diunduh tanpa izin.
- Batch 600 siswa dengan paralel terbatas, progress sesuai commit DB, modal hasil/failed/retry, browser tertutup/koneksi putus, resume tanpa duplikasi.
- Uji konfigurasi runtime produksi yang disuplai operator (`APP_DEBUG=false`), perlindungan `.env`, keamanan aplikasi, integritas data pada hasil restore yang disediakan operator, staging aman, dan UAT minimal operator sekolah. Pengaturan DNS/Nginx/SSL/firewall bukan bagian pekerjaan coding.

**Jangan menyatakan 'production ready'** sebelum kode aplikasi dan seluruh release gate yang relevan benar-benar diimplementasikan serta diuji. Prototipe HTML yang tersedia hanya ilustrasi alur.

### 9. Bentuk jawaban dan deliverable yang saya minta dari asisten coding

Pada awal pekerjaan, berikan:

1. Ringkasan pemahaman arsitektur dan modul.
2. Rencana implementasi fase berjalan, tidak membuat semuanya sekaligus.
3. Rancangan tabel/relasi serta permission yang terpengaruh fase tersebut.
4. Daftar file yang akan dihasilkan.

Setelah mengerjakan fase, berikan:

- Daftar perubahan kode/migration yang **betul-betul dibuat**.
- Perintah install, migrate, test, dan build yang benar untuk fase tersebut.
- Hasil pengujian aktual, dengan kegagalan dan pekerjaan tersisa yang dinyatakan jelas.
- Screenshot atau preview UI **bila benar-benar dihasilkan**.
- Poin penerimaan yang sudah dan belum terpenuhi.

**Mulai dari Fase 0 dan Fase 1.** Jangan membangun 15 modul sekaligus dan jangan mengarang hasil testing atau membuat data produksi palsu.

---

## Ringkasan pemakaian file pada paket ini

| Berkas | Peran |
|---|---|
| `PRD.md` | Sumber persyaratan produk, aturan data, role, database, dan pengujian |
| `PETA_PRD_KE_HTML.md` | Peta referensi halaman HTML terhadap bagian PRD |
| `prototype/index.html` | Titik masuk pratinjau frontend offline |
| `01-owner.html` s.d. `14-modul-lain.html` | Contoh layar dan interaksi frontend, **bukan** implementasi backend |
| `promt.md` | Instruksi siap salin untuk mengimplementasikan PRD secara bertahap
| `task.md` | Checklist rinci termasuk kontrak URL, database, dan keamanan |

## Catatan terakhir

Paket ini **siap untuk review desain dan mulai development**, **belum siap produksi**. Produksi membutuhkan aplikasi Laravel/MariaDB yang selesai, bukti testing, backup/restore, keamanan, dan pilot sekolah sesuai Bab 14 PRD.


## 10. Definition of Done dan batas tanggung jawab (FINAL)

Sebelum mencentang satu item pada `task.md`, tulis: file yang berubah, perubahan migration, pengujian yang dijalankan, bukti hasilnya, dan risiko yang tersisa. Jika belum ada code dan test, status adalah **belum selesai**.

**Developer hanya menulis aplikasi.** Jangan menghasilkan skrip untuk provisioning VPS, konfigurasi DNS/domain, Nginx virtual host, sertifikat SSL, firewall/SSH, monitoring host, atau backup host. Boleh mendokumentasikan variabel lingkungan yang dibutuhkan kode (`.env.example`, tanpa nilai rahasia), bukan menyediakan nilai production.

Gunakan `GO_LIVE_CHECKLIST.md` sebagai gerbang final. Sebut aplikasi *ready for handover* hanya setelah fitur rilis lulus test dan UAT. Jangan mengaku *go-live* sampai penyedia infrastruktur dan sekolah juga menyatakan siap.

Setiap permintaan membuat fitur harus mempertahankan seluruh keputusan PRD: 15 modul, PKL nilai kertas diinput guru, absensi hanya siswa, izin siswa+guru satu modul, surat boleh tanpa file, alumni lama self-register diverifikasi, kelas XII→XIII bila 4 tahun, kelulusan tidak otomatis, tagihan dan aset tetap konsisten, menu CMS dinamis, bulk HTTP paralel progres DB, modal/indikator loading, dan **tanpa Queue/Scheduler**.
