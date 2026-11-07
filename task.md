# TASK — Checklist Implementasi SaaS SMK Dedicated VPS

**Versi:** 1.3 (App-Only) · **Status:** Belum dimulai (semua kotak tugas masih kosong)  
**Acuan utama:** [`PRD.md`](PRD.md)  
**Instruksi coding:** [`promt.md`](promt.md)  
**Referensi UI:** [`prototype/index.html`](prototype/index.html) dan prototipe HTML lainnya.

> File ini adalah daftar kerja untuk mengubah PRD dan prototype menjadi **aplikasi Laravel sungguhan**. Menyelesaikan checklist tidak otomatis berarti siap produksi; setiap gerbang pengujian dan uji coba sekolah tetap wajib dilewati.

## Cara menggunakan checklist

1. Kerjakan tahap sesuai urutan **F0 → F1 → F2 → F3 → modul → QA → rilis aplikasi**; jangan mengerjakan seluruh modul sekaligus.
2. Tandai `[x]` **hanya** jika implementasi sudah ada, diuji, dan kriteria selesai terpenuhi. Tambahkan referensi commit/PR atau bukti pengujian pada laporan kemajuan.
3. Setiap modul yang sudah dibangun harus lulus tes izin pengguna, validasi input, aktivasi lisensi, dan integritas data.
4. Ikuti PRD bila ada perbedaan antara PRD dan HTML (HTML hanya contoh). Jangan memperluas MVP tanpa kebutuhan pengguna.
5. Uji fitur massal dengan data sintetis, bukan data pribadi siswa pada lingkungan pengembangan.

### Batasan tetap (tidak boleh berubah)

- **Satu sekolah = satu VPS = satu instalasi Laravel = satu MariaDB**, dengan domain `.sch.id` sekolah.
- Stack: **Laravel 13, PHP 8.5, Livewire 4, Blade, Tailwind CSS 4, Nginx, MariaDB**.
- **Satu Super Admin SaaS lokal** di `/owner`, terpisah dari Admin Sekolah di `/admin`. Tidak ada VPS/domain pusat dan **bukan multi-tenant**.
- **15 modul tetap**, modular monolith, diaktifkan menurut lisensi lokal. Core System selalu tersedia.
- **Tanpa Queue, Scheduler, worker, Redis, WebSocket, cron aplikasi, payment gateway, PPDB, UKK, absensi guru, atau akun pembimbing industri.**
- Perizinan siswa **dan guru** adalah bagian dari modul `absensi_izin`; **bukan modul ke-16**.
- Proses massal menggunakan request HTTP paralel terbatas; persentase berasal dari transaksi yang benar-benar tersimpan di database.

---

## K — Kontrak routing, database dan keamanan aplikasi (wajib sebelum coding modul)

**Acuan:** PRD Bab 15–18. Checklist hanya untuk source code, database secara logis, keamanan aplikasi, QA, dan handover. Pengadaan/konfigurasi infrastruktur tidak menjadi tugas developer.

### K0. URL dan routing Laravel

- [ ] K0.01 Implementasikan peta route PRD Bab 15: halaman publik `/`, login/reset, `/owner`, `/admin`, siswa, alumni, dan seluruh 15 prefix modul.
- [ ] K0.02 Buat named routes dan route test (GET/POST/PATCH/DELETE sesuai aksi); jangan mengubah data menggunakan GET.
- [ ] K0.03 Semua URL private wajib autentikasi, pemeriksaan permission dan lisensi (serta scope objek). Owner **tidak** boleh membaca BK/keuangan personal tanpa izin.
- [ ] K0.04 Semua tautan internal memakai named routes; aplikasi bekerja pada host yang disediakan operator VPS, tanpa domain hardcoded.
- [ ] K0.05 Website tidak aktif tetap menyajikan halaman informasi minimal pada `/` dan akses `/admin` tetap normal; portal alumni mandiri hanya terbuka jika BKK/Tracer aktif.

### K1. Database dan integritas

- [ ] K1.01 Gunakan koneksi MariaDB lewat konfigurasi Laravel `.env` privat (`DB_CONNECTION=mariadb`), tanpa kredensial di source code. Keamanan port/database server ditangani operator VPS.
- [ ] K1.02 Unique index NIS, NISN non-null, siswa-per-tahun, graduation-per-siswa, alumni linkage, invoice-per-biaya-per-periode, receipt number, idempotency key.
- [ ] K1.03 Foreign key historis, larangan cascade pada enrolmen, kelulusan, invoice, payments; migrasi diverifikasi di staging.
- [ ] K1.04 Uji lock/transaksi untuk pindah kelas, kenaikan, kelulusan, pembayaran bersamaan, peminjaman aset, dan stok Lab.
- [ ] K1.05 Tabel `bulk_operations`/items memiliki status, progress faktual, attempt dan lease; lease kedaluwarsa dipulihkan **ketika ada request**, tanpa worker/scheduler.
- [ ] K1.06 Gunakan UTC untuk penyimpanan waktu dan `Asia/Jakarta` untuk waktu yang ditampilkan dan aturan tanggal efektif; tes batas pergantian hari/tahun.

### K2. Keamanan dan privasi

- [ ] K2.01 Pembuatan owner pertama melalui proses provisioning aman; tanpa password bawaan; MFA **wajib** sebelum rilis produksi.
- [ ] K2.02 Rate limit login, reset, registrasi alumni; CSRF, hash password, regenerasi/invalidate session, cookie HttpOnly/Secure/SameSite.
- [ ] K2.03 Test IDOR dan akses salah role pada route, aksi Livewire, file download/export, endpoint massal dan data BK.
- [ ] K2.04 Upload validasi MIME/ukuran/ekstensi; nama acak; private storage; unduh file privat hanya melalui controller + Policy.
- [ ] K2.05 Validasi input dan sanitasi konten CMS (XSS); tidak menerima SQL/raw HTML tidak aman; error tidak membocorkan stack trace.
- [ ] K2.06 Audit perubahan sensitif (pembayaran, kelulusan, lisensi, BK) **tanpa** mencatat token, password dan isi konseling.
- [ ] K2.07 Data pernikahan alumni opsional/privat; dokumen siswa dan transaksi tidak dapat dipublikasikan secara tidak sengaja.
- [ ] K2.08 Audit dependency PHP/JS, pastikan rahasia tidak bocor ke repo/log, serta uji Host header tidak dipakai mentah untuk membuat tautan sensitif.

### K3. Operasional dan syarat rilis

- [ ] K3.01 Dokumentasikan `.env.example` tanpa rahasia dan kebutuhan variabel Laravel/MariaDB; konfigurasi DNS, Nginx, TLS, VPS di luar scope task aplikasi.
- [ ] K3.02 Berikan **kontrak data** yang perlu dilindungi (database + private storage), serta prosedur pemeriksaan konsistensi aplikasi setelah restore yang dilakukan tim infrastruktur. Developer tidak membuat/menjalankan backup host.
- [ ] K3.03 Uji resumption bulk dari dua browser dan setelah koneksi hilang, progress berdasarkan DB, tidak ada siswa/tagihan ganda.
- [ ] K3.04 Uji deployment staging, rollback aman, `APP_ENV=production`, `APP_DEBUG=false`, logging tanpa data pribadi.
- [ ] K3.05 Simpan bukti hasil uji otomatis, keamanan, UAT sekolah, dan tetapkan modul yang benar-benar layak ditawarkan.

**Urutan eksekusi:** K0/K1/K2 dikerjakan bersama F0–F4 sesuai dependensi; K3 bersama F7–F8. Kategori K bukan modul tambahan dan bukan layanan server baru.

---

## F0 — Fondasi proyek dan antarmuka

**Dependensi:** tidak ada. **Hasil:** aplikasi Laravel bisa diakses dan login pada lingkungan development.

- [ ] F0.01 Buat repository dan instal Laravel 13 pada PHP 8.5, Livewire 4, Tailwind CSS 4.
- [ ] F0.02 Siapkan `.env.example` tanpa rahasia, koneksi MariaDB, timezone `Asia/Jakarta`, dan storage privat.
- [ ] F0.03 Buat struktur modular monolith yang konsisten untuk Core System dan 15 modul; kode modul harus stabil.
- [ ] F0.04 Siapkan login, logout, reset password, session, CSRF, rate limit login, dan pembuatan Super Admin pertama yang aman.
- [ ] F0.05 Pisahkan layout `/owner`, `/admin`, portal siswa/alumni sesuai hak akses; implementasikan sidebar hanya untuk modul aktif.
- [ ] F0.06 Buat komponen UI reusable: tabel, pencarian/filter, empty state, badge status, pagination, toast.
- [ ] F0.07 Buat modal reusable untuk konfirmasi, error penting, hasil proses; fokus keyboard, escape/close aman, responsif HP.
- [ ] F0.08 Implementasikan skeleton/indikator loading, spinner pada tombol, disable double-submit, validasi per field.
- [ ] F0.09 Siapkan audit log minimal untuk perubahan data penting tanpa mencatat kata sandi/token atau isi konseling.
- [ ] F0.10 Uji aplikasi tanpa Queue/Scheduler, tidak ada dependency worker yang harus aktif agar fitur berjalan.

**Selesai jika:** login/logout aman; UI responsif; tindakan hapus/ubah penting memakai modal; navigasi berdasar role; tes fondasi lulus.

## F1 — Core System dan data induk

**Dependensi:** F0. **Hasil:** data sekolah dan data siswa dapat dikelola tanpa duplikasi.

- [ ] F1.01 Migration profil sekolah, jurusan/program 3–4 tahun, mata pelajaran, tahun ajaran/semester, kelas/rombel.
- [ ] F1.02 Migration `students`, `teachers`, `staff`, `users` dan relasi akun, dengan NIS unik dan NISN unik jika tersedia.
- [ ] F1.03 Migration `student_enrollments` dan `student_class_placements` sebagai riwayat; **jangan hanya menyimpan `students.class_id`**.
- [ ] F1.04 CRUD profil sekolah, jurusan, kelas, guru, staf, siswa; filter/pencarian sederhana.
- [ ] F1.05 Role dapat rangkap, permission berdasarkan tindakan **dan** objek yang ditugaskan; buat Policies/Gates/Middleware.
- [ ] F1.06 CRUD akun yang diizinkan, nonaktifkan akun, reset akses; tidak membuat akun siswa/alumni ganda.
- [ ] F1.07 Impor siswa/guru/kelas dari Excel/CSV dengan template, validasi, preview, penanganan duplikat; ekspor sesuai izin.
- [ ] F1.08 Tahun ajaran baru dimulai sebagai **draft**, diaktifkan secara eksplisit; pergantian tahun **tidak otomatis** menaikkan kelas.
- [ ] F1.09 Riwayat kelas terlihat per siswa; pindah kelas mencatat tanggal efektif tanpa menimpa penempatan lama.
- [ ] F1.10 Uji foreign key, indeks unik, validasi data, dan larangan menghapus histori penting melalui cascade.

**Selesai jika:** seorang siswa mempunyai satu identitas permanen; perubahan kelas tidak menghapus riwayat; siswa belum diproses tetap terlihat.

## F2 — Siklus siswa, kelulusan, dan alumni

**Dependensi:** F1 + fondasi operasi massal F4 (untuk aksi massal). **Hasil:** data siswa terjaga dari masuk kelas X sampai alumni.

- [ ] F2.01 Siapkan kelas tujuan per tahun ajaran, aturan naik/tinggal kelas, dan pengecualian siswa.
- [ ] F2.02 Buat **pratinjau kenaikan kelas massal**: sumber, kelas tujuan, pilihan semua/checkbox, jumlah valid/tidak valid/belum diproses.
- [ ] F2.03 Konfirmasi modal sebelum memulai; proses menaikkan siswa hanya setelah Admin menyetujui.
- [ ] F2.04 Catat kenaikan tiap siswa dengan transaksi dan idempotensi; X→XI, XI→XII, serta XII→XIII untuk program 4 tahun.
- [ ] F2.05 Buat dashboard **sudah diproses / belum diproses / gagal**, agar sebagian siswa tidak hilang ketika tahun ajaran aktif berubah.
- [ ] F2.06 Buat **proses kelulusan terpisah** (tidak otomatis saat naik tingkat/tahun ajaran), mendukung lulus, belum lulus, dan pengecualian.
- [ ] F2.07 Pastikan kelulusan hanya sekali per siswa, alumni baru terhubung ke siswa dan memakai **akun yang sama** jika sudah ada.
- [ ] F2.08 Portal registrasi mandiri **alumni lama**, pencocokan identitas, review/verifikasi BKK, pencegahan klaim ganda dan penyamaran.
- [ ] F2.09 Buat `alumni_profiles` dengan `student_id` nullable untuk alumni lama; proses tautkan arsip siswa bila cocok dan diverifikasi.
- [ ] F2.10 Profil alumni dan histori **pekerjaan, kuliah, wirausaha** yang dapat tumpang tindih; tanggal mulai/selesai dan status aktif.
- [ ] F2.11 Data status perkawinan/kejadian pribadi hanya **opsional dan privat**, tidak wajib untuk tracer umum.
- [ ] F2.12 Pengunduran diri, mutasi masuk/keluar, dan siswa yang tinggal kelas tidak membuat alumni fiktif.
- [ ] F2.13 Uji identitas dan riwayat akademik tetap utuh setelah naik, pindah, lulus, dan pendaftaran alumni lama.

**Selesai jika:** siswa kelas XII/XIII tidak otomatis alumni; alumni lama bisa mendaftar sendiri; akun dan histori terkait tidak terduplikasi.

## F3 — Owner, lisensi, dan hak akses modul

**Dependensi:** F0–F1. **Hasil:** pemilik SaaS dapat mengelola modul di tiap instalasi sekolah.

- [ ] F3.01 Buat `/owner` dengan akses Super Admin terproteksi (MFA jika tersedia, pembatasan akses jika memungkinkan).
- [ ] F3.02 Profil pelanggan **satu sekolah** pada VPS tersebut; tidak ada katalog banyak tenant di instalasi.
- [ ] F3.03 Seed 15 modul dengan kode stabil; modul yang tidak aktif tidak muncul di menu.
- [ ] F3.04 Aktivasi/nonaktifkan modul, masa berlaku, catatan perpanjangan, dan pencatatan pembayaran langganan manual.
- [ ] F3.05 Middleware/Policy memblokir akses URL, aksi Livewire, ekspor, unggah, dan endpoint massal modul tidak aktif.
- [ ] F3.06 Saat modul kedaluwarsa: operasi modul diblokir, **data tidak dihapus**; akses core dan prosedur ekspor/retensi tetap tersedia sesuai kebijakan.
- [ ] F3.07 Owner **tidak otomatis** dapat membaca data BK, informasi keluarga, pembayaran personal, dan data sensitif sekolah.
- [ ] F3.08 Audit setiap perubahan lisensi/modul/pembayaran owner; tes role bypass dan akses URL langsung.

**Selesai jika:** sekolah dengan hanya `surat` + `website` tidak dapat membuka modul lain walau menebak URL.

## F4 — Mesin proses massal simultan (tanpa Queue/Scheduler)

**Dependensi:** F0, skema data F1. **Digunakan oleh:** impor siswa/alumni, kenaikan, kelulusan, tagihan massal.

- [ ] F4.01 Migration `bulk_operations` dan `bulk_operation_items` (unique `operation_id` + `entity_key`), status dan error ringkas.
- [ ] F4.02 Endpoint authenticated/authorized: **preview → start → claim/process batch → status → pause/resume → retry failed**.
- [ ] F4.03 Snapshot target saat start agar hasil filter yang berubah tidak mengubah daftar sasaran proses berjalan.
- [ ] F4.04 Browser mengeksekusi maksimal **2–3 request HTTP paralel**, 10–25 item per request (konfigurasi; uji kapasitas VPS).
- [ ] F4.05 Claim item secara atomik; tiap item memakai transaksi dan validasi kunci bisnis agar retry **tidak membuat duplikat**.
- [ ] F4.06 Progress: `(success + already_processed + skipped + failed) / total_items × 100%` berdasarkan status **commit MariaDB**, bukan animasi.
- [ ] F4.07 Tampilkan progress bar, total, sukses, gagal, dilewati, tersisa, serta hasil ringkasan di modal.
- [ ] F4.08 **Pause** berhenti mengirim batch baru (request yang terlanjur berjalan boleh selesai); **resume** dari data server.
- [ ] F4.09 Saat koneksi putus/browser ditutup, baca status DB terlebih dahulu ketika kembali; selesaikan/reclaim item yang terputus secara aman.
- [ ] F4.10 Retry **hanya item gagal** yang boleh diproses ulang; cegah duplikasi dua tab browser dan dua petugas menjalankan operasi sama.
- [ ] F4.11 Simpan daftar item gagal, alasan yang aman ditampilkan, dan opsi unduh hasil.
- [ ] F4.12 Uji 100/500/1.000 siswa sintetis, refresh, timeout, dua browser, retry, dan tanpa data hilang/duplikat.

**Selesai jika:** 100% berarti semua item memperoleh hasil (bukan semua sukses), dan resume/retry tidak mengubah item yang telah berhasil.

---

## F5 — Modul aplikasi (15 produk, kerjakan berurutan)

**Syarat untuk setiap modul:** migrasi + validasi + role/permission + pemeriksaan lisensi + UI loading/modal + laporan bila dibutuhkan + pengujian proses utama. Modul harus berfungsi sendiri; integrasi dengan modul lain **opsional** berdasarkan lisensi aktif.

| Urutan | Kode modul | Minimum yang dikerjakan | Role dan alur selesai | Referensi HTML |
|---|---|---|---|---|
| 01 | `website` | CMS halaman, berita, pengumuman, galeri, profil jurusan, **menu/submenu dinamis** (urut/sembunyi/URL) | Editor buat → preview → terbit → pengunjung lihat | `12-website.html` |
| 02 | `surat` | Surat masuk/keluar, nomor unik, template, arsip; **file opsional** (catat metadata/editor/upload) | TU buat/catat → persetujuan seperlunya → arsip | `10-surat.html` |
| 03 | `absensi_izin` | **Absensi siswa saja**, izin siswa/guru, persetujuan, rekonsiliasi izin–absensi | Guru piket absen; siswa/guru ajukan izin → pejabat setujui → rekap | `08-absensi.html` |
| 04 | `pelanggaran` | Tata tertib, bobot, catat, validasi, poin prestasi, pembinaan, keberatan | Guru catat → Kesiswaan validasi → BK tindak lanjut | `13-pelanggaran.html` |
| 05 | `pkl` | Mitra, penempatan, jurnal, absensi, monitoring, nilai industri **diinput guru dari kertas** | Koordinator tempatkan → siswa jurnal → guru pantau/input nilai → laporan | `07-pkl.html` |
| 06 | `jurnal_guru` | Isi jurnal, riwayat, pemeriksaan, rekap | Guru isi → Waka monitor | `14-modul-lain.html` |
| 07 | `jadwal` | Hari/jam, pembagian guru, jadwal kelas, deteksi bentrok, cetak | Waka susun → cek bentrok → terbit | `14-modul-lain.html` |
| 08 | `bk` | Konseling, jadwal, tindak lanjut, statistik tanpa mengumbar rahasia | Guru BK tangani → tindak lanjut → rekap terbatas | `14-modul-lain.html` |
| 09 | `keuangan` | Tarif, tagihan, cicilan, kuitansi, tunggakan, koreksi/void ber-audit | Bendahara terbitkan → terima/catat → laporan | `09-keuangan.html` |
| 10 | `sarpras` | Master aset unit/QR, kondisi, lokasi, peminjaman | Sarpras catat → pinjam → kembali | `11-sarpras.html` |
| 11 | `laboratorium` | Pakai **master aset yang sama**, stok bahan habis pakai, jadwal, peminjaman | Kepala Lab kelola → guru pinjam/pakai → kembali | `11-sarpras.html` |
| 12 | `hubin` | Master mitra bersama PKL, MoU, kegiatan, masa berlaku | HUBIN catat mitra → kerja sama → laporan | `14-modul-lain.html` |
| 13 | `bkk` | Lowongan, daftar alumni, lamaran, status hasil rekrutmen | BKK terbitkan → alumni melamar → BKK catat hasil | `06-alumni.html`, `14-modul-lain.html` |
| 14 | `tracer` | Form sederhana dan snapshot jawaban; analitik agregat | Alumni isi → Admin tracer periksa → laporan | `06-alumni.html`, `14-modul-lain.html` |
| 15 | `supervisi` | Jadwal, instrumen, nilai, tindak lanjut, laporan | Supervisor nilai → guru lihat/tindak lanjut → rekap | `14-modul-lain.html` |

### Checklist penyelesaian modul (duplikasi untuk setiap modul)

- [ ] F5.01 `website` — UI + backend + role + test selesai.
- [ ] F5.02 `surat` — UI + backend + role + test selesai (upload **tidak wajib**).
- [ ] F5.03 `absensi_izin` — UI + backend + role + test selesai (**tanpa absensi guru**).
- [ ] F5.04 `pelanggaran` — UI + backend + role + test selesai (validasi sebelum poin final).
- [ ] F5.05 `pkl` — UI + backend + role + test selesai (**tanpa akun industri**).
- [ ] F5.06 `jurnal_guru` — UI + backend + role + test selesai.
- [ ] F5.07 `jadwal` — UI + backend + role + test selesai (tanpa generator rumit).
- [ ] F5.08 `bk` — UI + backend + role + test selesai (data rahasia dibatasi).
- [ ] F5.09 `keuangan` — UI + backend + role + test selesai (histori tagihan aman).
- [ ] F5.10 `sarpras` — UI + backend + role + test selesai.
- [ ] F5.11 `laboratorium` — UI + backend + role + test selesai (aset sinkron).
- [ ] F5.12 `hubin` — UI + backend + role + test selesai (mitra bersama PKL).
- [ ] F5.13 `bkk` — UI + backend + role + test selesai.
- [ ] F5.14 `tracer` — UI + backend + role + test selesai (aktivitas alumni paralel).
- [ ] F5.15 `supervisi` — UI + backend + role + test selesai.

## F6 — Integrasi data lintas modul

**Dependensi:** modul terkait ada. Jika salah satunya tidak aktif, modul yang lain tetap berfungsi.

- [ ] F6.01 **Kenaikan/kelulusan ↔ Keuangan:** invoice lama tetap menunjuk `student_id`, tarif dan kelas saat diterbitkan tidak berubah; tunggakan tidak hilang.
- [ ] F6.02 **Kelulusan ↔ Alumni/BKK/Tracer:** alumni baru ditautkan ke siswa yang lulus, alumni lama tidak membuat siswa fiktif.
- [ ] F6.03 **Sarpras ↔ Laboratorium:** satu `assets.id` untuk aset unit yang sama, perubahan kondisi/lokasi muncul di dua modul.
- [ ] F6.04 **Absensi ↔ Izin:** persetujuan izin siswa selaras dengan status absensi dan meninggalkan jejak perubahan.
- [ ] F6.05 **PKL ↔ HUBIN:** satu tabel mitra industri; data perusahaan yang sama tidak diduplikasi.
- [ ] F6.06 **Jurnal Mengajar ↔ Jadwal:** jadwal otomatis menjadi pilihan bila modul jadwal aktif, bukan syarat wajib jurnal.
- [ ] F6.07 **Pelanggaran ↔ BK:** hanya rujukan tindak lanjut minimal; isi konseling tidak bocor ke pelapor.
- [ ] F6.08 **Website ↔ lisensi:** jika website tidak aktif, admin sekolah tetap bisa login ke aplikasi melalui domain sekolah.

**Selesai jika:** menonaktifkan modul pasangan tidak menimbulkan error pada modul yang masih aktif.

## F7 — QA, keamanan, performa dan aksesibilitas

**Dependensi:** fondasi + modul yang hendak dirilis. Uji dengan data fiktif.

- [ ] F7.01 Feature test akses URL/API/Livewire/export/download tanpa login, salah role, dan lisensi kedaluwarsa.
- [ ] F7.02 Uji validasi field, file upload (tipe/ukuran/nama/path), CSRF, XSS, dan rate limiting titik rawan.
- [ ] F7.03 Uji modal (konfirmasi/galat/hasil), spinner, double-submit, akses keyboard, fokus modal, dan layar kecil.
- [ ] F7.04 Uji kenaikan massal dan kelulusan: kelas akhir XII/XIII, tinggal kelas, 20 siswa tertunda, rerun tanpa duplikasi.
- [ ] F7.05 Uji histori: data keuangan, surat, PKL, BK, absensi tetap valid sesudah pindah kelas/lulus.
- [ ] F7.06 Uji transaksi pembayaran paralel: tidak overpayment, tidak terjadi kuitansi duplikat.
- [ ] F7.07 Uji stok dan pinjaman aset paralel: stok tidak minus, aset sama tidak dipinjam ganda.
- [ ] F7.08 Uji registrasi alumni lama: duplikat, upaya klaim identitas, verifikasi ditolak, akun terhubung dengan benar.
- [ ] F7.09 Uji penanganan gangguan koneksi ketika bulk berjalan, status pemulihan, pause/resume/retry.
- [ ] F7.10 Uji performa pada **spesifikasi VPS target yang sebenarnya**, termasuk pembuatan laporan dan 2–3 batch HTTP paralel.
- [ ] F7.11 Lakukan review privasi: BK, catatan pelanggaran, informasi keluarga, dokumen, foto siswa, dan audit log.
- [ ] F7.12 Jalankan tes otomatis serta UAT operator sekolah (TU, guru, wali kelas, BK, Bendahara, HUBIN, Admin).

**Selesai jika:** tidak ada bug kritis/tinggi terbuka; tes kritis lulus; pengguna bisa menyelesaikan alur utama tanpa instruksi teknis.

## F8 — Validasi rilis aplikasi (infrastruktur terpisah)

**Dependensi:** F7 lulus untuk cakupan modul yang akan dirilis.

- [ ] F8.01 Serahkan daftar kebutuhan runtime yang harus disediakan operator, tanpa melakukan pengaturan VPS, DNS, Nginx, SSL, firewall, atau SSH.
- [ ] F8.02 Verifikasi konfigurasi runtime produksi yang dipasok operator: `APP_DEBUG=false`, `APP_ENV=production`, cookie aman, `.env` tidak terbuka melalui aplikasi.
- [ ] F8.03 Deployment reproducible: build aset, migration aman, cek kompatibilitas skema, cache config/view/route sesuai aplikasi.
- [ ] F8.04 Dokumentasikan aset data aplikasi yang harus dicakup backup (MariaDB dan file privat), serta pemeriksaan konsistensi aplikasi usai restore oleh operator; **tanpa menulis skrip backup host**.
- [ ] F8.05 Setelah tim infrastruktur memulihkan salinan data uji, jalankan smoke test integritas siswa, keuangan, file privat, dan hak akses dari sisi aplikasi.
- [ ] F8.06 Siapkan log error aplikasi yang aman dan panduan pemeriksaan error Laravel. Monitoring infrastruktur ditangani terpisah.
- [ ] F8.07 Siapkan prosedur update versi, rollback kode/database yang aman, dan penanganan insiden.
- [ ] F8.08 Jalankan uji coba **satu sekolah** dan verifikasi akses aplikasi melalui URL yang tersedia, akun, lisensi, data, hak akses, serta laporan.
- [ ] F8.09 Tetapkan rilis hanya untuk **modul yang telah lulus QA**; yang belum selesai tidak boleh ditawarkan/diaktifkan.
- [ ] F8.10 Dokumentasikan hasil akhir, migrasi, panduan admin, panduan owner, dan catatan rilis.

### Gerbang rilis (wajib semua tercentang sebelum disebut “siap produksi”)

- [ ] R1 Semua fungsi yang dijual benar-benar mempunyai backend Laravel/MariaDB (bukan simulasi HTML).
- [ ] R2 Isolasi akses role + lisensi teruji dari UI hingga endpoint backend.
- [ ] R3 Siswa, kelas, kelulusan, alumni, tagihan, dan aset **konsisten secara historis**.
- [ ] R4 Operasi massal paralel terbatas menunjukkan progress nyata, aman retry, tanpa duplikasi.
- [ ] R5 Modal/loading/error/validasi benar dan responsif pada desktop/HP.
- [ ] R6 Tes baca ulang database dan file pasca-restore yang disediakan tim infrastruktur lulus; dokumentasikan hasil dan dependensi yang masih dipegang operator.
- [ ] R7 Keamanan aplikasi, penyimpanan berkas privat dan proteksi data diuji; keamanan server/HTTPS menjadi prasyarat terpisah dari operator VPS.
- [ ] R8 Pengguna sekolah lulus UAT pada alur penting, tanpa bug kritis terbuka.

---

## F9 — Bukti kualitas dan handover aplikasi

**Dependensi:** F7/F8. **Hasil:** dokumen `GO_LIVE_CHECKLIST.md` dan artefak aplikasi yang bisa dievaluasi secara independen.

- [ ] F9.01 Isi checklist go-live aplikasi disertai bukti test (nama test, commit, screenshot/error log teredaksi, penguji, tanggal).
- [ ] F9.02 Dokumentasikan daftar 15 modul beserta status: siap diuji / lulus / belum diimplementasikan. Jangan mengaktifkan penjualan modul yang belum selesai.
- [ ] F9.03 Serahkan release tag, source code, build instructions, migration/seeder, `.env.example`, route map, panduan role, dan panduan penggunaan inti.
- [ ] F9.04 Pastikan dokumentasi tidak memuat domain/IP rahasia, password sekolah, NISN nyata, atau data BK/keuangan asli.
- [ ] F9.05 Handover kontrak kebutuhan aplikasi ke tim infrastruktur, **tanpa** konfigurasi server; penerimaan operasi host dilakukan terpisah.
- [ ] F9.06 Catat keputusan **GO / NO-GO / LIMITED PILOT** serta pemberi persetujuan sekolah dan developer. Status 'GO' aplikasi tetap bukan klaim infrastruktur telah siap.

**Selesai jika:** `GO_LIVE_CHECKLIST.md` berisi bukti pemeriksaan seluruh gerbang wajib dan tidak ada blocker aplikasi.

---

## Template catatan progres untuk developer

Salin blok ini ke issue atau catatan rilis tiap kali mengerjakan satu task:

```md
- Task ID: F?.??
- Status: Belum dimulai / Dalam pengerjaan / Diuji / Selesai / Terblokir
- Implementasi: [ringkasan singkat]
- Bukti: [commit/PR/screenshot/hasil pengujian]
- Pengujian: [apa yang lulus dan belum]
- Risiko atau dependensi: [jika ada]
```

**Catatan:** kontrak baru PRD Bab 15–17 dan checklist K0–K3 wajib dipenuhi; `task.md` adalah checklist pelaksanaan, `promt.md` memandu cara coding, PRD menentukan kebutuhan, dan HTML menunjukkan contoh pengalaman pengguna. Semua file harus dibaca bersama.
