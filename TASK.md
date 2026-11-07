# TASK — Checklist Implementasi SaaS SMK Dedicated VPS

> Repository berisi spesifikasi dan desain HTML; aplikasi Laravel belum diimplementasikan.
> Dua dokumen acuan: `PRD.md` untuk kebutuhan produk dan `TASK.md` untuk pekerjaan, panduan developer, checklist rilis, serta bukti progres.
> Mulai dari F0.01–F0.03 dan kontrak K0/K1 yang diperlukan. Jangan menandai tugas selesai hanya karena desain HTML tersedia.
> Gunakan checkout yang sudah ada; setiap tugas cloud sudah terisolasi dan tidak memerlukan Git worktree baru.


**Versi:** 1.6 (App-Only, selaras PRD 4.8) · **Status:** Belum dimulai (semua kotak tugas masih kosong)
**Acuan utama:** [`PRD.md`](PRD.md)  
**Instruksi coding:** bagian Panduan implementasi dalam dokumen ini  
**Referensi UI:** [`prototype/index.html`](prototype/index.html) dan prototipe HTML lainnya.

> File ini adalah daftar kerja untuk mengubah PRD dan prototype menjadi **aplikasi Laravel sungguhan**. Menyelesaikan checklist tidak otomatis berarti siap produksi; setiap gerbang pengujian dan uji coba sekolah tetap wajib dilewati.

## Cara menggunakan checklist

1. Kerjakan tahap sesuai urutan **F0 → skema/master F1 → F4 → penyelesaian impor F1 → F2 → F3 → F5/F6 → F7 → F8/F9**; jangan mengerjakan seluruh modul sekaligus.
2. Tandai `[x]` **hanya** jika implementasi sudah ada, diuji, dan kriteria selesai terpenuhi. Tambahkan referensi commit/PR atau bukti pengujian pada laporan kemajuan.
3. Setiap modul yang sudah dibangun harus lulus tes izin pengguna, validasi input, aktivasi lisensi, dan integritas data.
4. Ikuti PRD bila ada perbedaan antara PRD dan HTML (HTML hanya contoh). Jangan memperluas MVP tanpa kebutuhan pengguna.
5. Uji fitur massal dengan data sintetis, bukan data pribadi siswa pada lingkungan pengembangan.

### Batasan tetap (tidak boleh berubah)

- **Satu sekolah = satu VPS = satu instalasi Laravel = satu MariaDB**, dengan domain `.sch.id` sekolah.
- Stack: **Laravel 13, PHP 8.5, Livewire 4, Blade, Tailwind CSS 4, Nginx, MariaDB**.
- **Satu Super Admin SaaS lokal** di `/owner`, terpisah dari Admin Sekolah di `/admin`. Tidak ada VPS/domain pusat dan **bukan multi-tenant**.
- **15 modul tetap**, modular monolith, diaktifkan menurut lisensi lokal. Core System selalu tersedia. Dokumen Markdown proyek tetap hanya `PRD.md` dan `TASK.md`; panduan dan bukti progres ditulis di dua dokumen ini.
- **Tanpa Queue, Scheduler, worker, Redis, WebSocket, cron aplikasi, pembayaran gateway, PPDB, UKK, absensi guru, atau akun pembimbing industri.**
- Perizinan siswa **dan guru** adalah bagian dari modul `absensi_izin`; **bukan modul ke-16**.
- Proses massal menggunakan request HTTP paralel terbatas; persentase berasal dari transaksi yang benar-benar tersimpan di database.

---

**Dependensi lintas tahap:** F1.07 (impor massal) diselesaikan setelah F4, bukan syarat membangun F4. Kontrol lisensi minimum F3.03/F3.05 disiapkan sebelum membuka portal F2.08. Model tagihan/pembayaran dan tes historis disiapkan pada F2.14; UI, laporan, dan penyelesaian Keuangan tetap F5.09. K0–K2 menyertai tahap terkait, tidak harus seluruh route modul selesai pada F0.

## Bahasa, struktur kode, dan penamaan (wajib)

Ikuti PRD §3.3, §4.2 dan §12.1.1. Fokus fitur: `routes/`, `app/Http/Controllers/`, `app/Livewire/`, `app/Models/`, `app/Http/Middleware/`, `resources/`. Migration/seeder, tes, terjemahan bawaan `lang/id/` dan konfigurasi minimum hanya pendukung yang diperlukan. Hindari arsitektur tambahan; alur Controller/Livewire ringkas, relasi/transaksi konsisten pada Model, validasi server dan otorisasi objek pada setiap aksi.

- [ ] K0.06 Tinjau struktur folder, metode singkat, penamaan jelas, penghindaran duplikasi/N+1 dan format Pint; tidak membuat lapisan generik yang belum dibutuhkan.
- [ ] K1.07 Terapkan tabel/kolom bisnis Bahasa Indonesia sesuai PRD; Model menetapkan tabel/FK eksplisit dan autentikasi `Pengguna` tetap kompatibel Laravel. Uji migration, relasi, constraint dan transaksi MariaDB.
- [ ] K2.09 Terapkan 13 role inti PRD §4.2/13; gabungkan pengelolaan BKK/Tracer dalam Pengelola Alumni dan jabatan lain sebagai penugasan, bukan role baru; uji rangkap role, lingkup penugasan, larangan eskalasi owner, S/I siswa langsung dari wali kelas; persetujuan pejabat hanya untuk izin guru sendiri dan privasi laporan.
- [ ] F0.11 Siapkan locale `id`, `lang="id"` dan terjemahan Indonesia untuk UI, autentikasi, validasi, pesan framework/paket, laporan serta teks aksesibilitas. Format tanggal/angka Indonesia dan Rupiah.
- [ ] F7.13 Uji seluruh layar/alur rilis dalam Bahasa Indonesia termasuk error/validasi/reset sandi, modal, laporan/cetak dan tampilan mobile; kode status internal tidak tampil mentah.

**Acuan tabel bisnis:** `siswa`, `tahun_ajaran`, `kepesertaan_siswa`, `penempatan_kelas_siswa`, `profil_alumni`, `tagihan`, `pembayaran`, `operasi_massal`, dan `item_operasi_massal`; rincian tabel/kolom lain mengikuti PRD Bab 12. Jangan membuat tabel paralel dengan nama Inggris.

**Acuan fitur dan navigasi:** PRD §4.3 memuat daftar fitur dan §4.4 memuat menu/submenu, isi layar, tindakan dan batas akses 13 role. Tidak menambah modul atau role. Jurnal guru berisi kegiatan/hasil harian saat modul aktif, tidak bergantung jadwal/kalender; izin wali bisa dicatat dari luar aplikasi tanpa akun wali.

## Keputusan operasional dan kontrak rilis terbaru

PRD modul 02/03, §4.2.2, Bab 10, §13.5 dan §14.6 adalah acuan: jurnal kegiatan/hasil harian, presensi hari ini satu kelas atomik, S/I wali langsung, konfirmasi tanpa batas waktu, tanpa kalender akademik, 13 role dengan penugasan rinci, rilis R1–R4 serta upgrade aman. Kalender/libur bukan prasyarat input. Transaksi per item berlaku pada operasi massal; transaksi seluruh daftar berlaku pada kirim presensi kelas, jangan dicampur.

- [ ] F1.12 Implementasikan penugasan mengajar/wali/BK/bidang/unit dan delegasi dengan FK, rentang tanggal, lingkup, audit dan konflik; uji pergantian/pencabutan dan akses historis PRD §4.2.2.
- [ ] F5.17 Implementasikan penyimpanan kelas atomik beserta `pengiriman_presensi`, token idempotensi, versi sesi/daftar/izin, validasi himpunan siswa, bukti jumlah tersimpan dan endpoint periksa hasil; bukan request per siswa dan bukan bulk item engine.
- [ ] F5.18 Implementasikan 5 tema publik klasik/modern/vokasi/berita/kreatif, satu admin layout, CMS/menu bersama, pratinjau terotorisasi, allowlist/fallback, penyimpanan pilihan dan ganti tema tanpa kehilangan data.
- [ ] F7.17 Uji toggle/expiry modul saat form terbuka/transaksi/bulk berjalan, dua tab, arsip, aktif ulang dan seluruh pasangan mandiri PRD §13.5; Core/fitur lain tidak error, tidak ada partial commit atau bypass cache.
- [ ] F7.18 Uji kelima tema desktop/HP/keyboard, menu/submenu, berita/detail, halaman statis, kosong/error, fallback tema invalid dan admin tidak berubah.
- [ ] F8.11 Uji upgrade dari rilis sebelumnya, bukan hanya fresh install: semua data historis, lisensi aktif/nonaktif, penugasan, tema/media, token transaksi tetap konsisten; uji payload tab lama, cache baru, rollback kompatibel dan simulasi migration gagal.
- [ ] F9.07 Catat rilis target R1/R2/R3/R4, minimum versi upgrade, modul layak jual, bukti UAT dan risiko. R1 hanya Core + Absensi/Izin + Jurnal Kegiatan; modul lain tidak diaktifkan sebelum lulus.

## C0 — Kontrak lintas modul, arsip dan cakupan rilis

- [ ] C0.01 Implementasikan `katalog_modul`, `lisensi_modul` dan `riwayat_lisensi_modul` §12.9; kesiapan rilis berasal manifest source, bukan toggle owner. Uji batas waktu eksklusif, cache versi dan seeder tidak mengaktifkan/memperpanjang lisensi existing.
- [ ] C0.02 Implementasikan route arsip daftar/detail/ekspor/unduh §15.1 dengan hak `arsip.<kode_modul>.lihat/ekspor`, lingkup objek dan bukti pernah aktif; tidak memakai `module-aktif`, tidak membaca resource/path arbitrer dari parameter.
- [ ] C0.03 Uji arsip saat lisensi expired/nonaktif, petugas pengganti berizin, pegawai lama dicabut, owner tanpa hak data, modul belum pernah aktif, kode tidak dikenal dan kolom privat; seluruh mutasi tetap ditolak.
- [ ] C0.04 Status bulk dan bukti presensi melalui Core hanya membaca hasil, tidak claim/lazy recovery/memproses ulang saat lisensi nonaktif. Uji timeout sesudah commit dan pencabutan lisensi tanpa kehilangan bukti.
- [ ] C0.05 Gunakan satu matriks cakupan R1–R4 PRD Bab 10/§14.1; seluruh gate, route, UAT dan task membedakan wajib Core, wajib modul dalam rilis, serta belum masuk rilis. Tidak menganggap katalog 15 modul sebagai 15 implementasi selesai.
- [ ] C0.06 Uji upgrade pembaca arsip seluruh modul yang pernah aktif, aktif maupun nonaktif; preflight versi/jalur perantara dan kegagalan migrasi tidak membuka source/skema tak kompatibel. Jalankan regresi modul terpasang meski bukan modul baru yang dijual.

## K — Kontrak routing, database dan keamanan aplikasi (wajib sebelum coding modul)

**Acuan:** PRD Bab 15–18. Checklist hanya untuk source code, database secara logis, keamanan aplikasi, QA, dan handover. Pengadaan/konfigurasi infrastruktur tidak menjadi tugas developer.

### K0. URL dan routing Laravel

- [ ] K0.01 Implementasikan route Core dan modul yang masuk rilis sesuai Bab 15, termasuk arsip/bukti/progres Core. Lima belas prefix katalog boleh dikenali tetapi modul belum dirilis ditolak terkontrol; tidak dinyatakan sudah memiliki fitur. Uji route yang siap dan penolakan route yang belum siap.
- [ ] K0.02 Buat named routes dan route test (GET/POST/PATCH/DELETE sesuai aksi); jangan mengubah data menggunakan GET.
- [ ] K0.03 Semua URL private wajib autentikasi, permission dan scope objek; lisensi diperiksa untuk fitur modul berbayar, bukan sebagai syarat akses Core. Owner **tidak** boleh membaca BK/keuangan personal tanpa izin.
- [ ] K0.04 Semua tautan internal memakai named routes; aplikasi bekerja pada host yang disediakan operator VPS, tanpa domain hardcoded.
- [ ] K0.05 Website tidak aktif tetap menyajikan halaman informasi minimal pada `/` dan akses `/admin` tetap normal; portal alumni mandiri hanya terbuka jika BKK/Tracer aktif.

### K1. Database dan integritas

- [ ] K1.01 Gunakan koneksi MariaDB lewat konfigurasi Laravel `.env` privat (`DB_CONNECTION=mariadb`), tanpa kredensial di source code. Keamanan port/database server ditangani operator VPS.
- [ ] K1.02 Unique index NIS, NISN non-null, siswa-per-tahun, graduation-per-siswa, alumni linkage, tagihan-per-biaya-per-periode, receipt number, idempotency key.
- [ ] K1.03 Foreign key historis, larangan cascade pada enrolmen, kelulusan, tagihan, pembayaran; migrasi diverifikasi di staging.
- [ ] K1.04 Uji lock/transaksi untuk pindah kelas, kenaikan, kelulusan, pembayaran bersamaan, peminjaman aset, dan stok Lab.
- [ ] K1.05 Tabel `operasi_massal`/items memiliki status, progress faktual, attempt dan lease; lease kedaluwarsa dipulihkan **ketika ada request**, tanpa worker/scheduler.
- [ ] K1.06 Gunakan UTC untuk penyimpanan waktu dan `Asia/Jakarta` untuk waktu yang ditampilkan dan aturan tanggal efektif; tes batas pergantian hari/tahun.

### K2. Keamanan dan privasi

- [ ] K2.01 Pembuatan owner pertama melalui proses provisioning aman; tanpa password bawaan; MFA **wajib** sebelum rilis produksi.
- [ ] K2.02 Rate limit login, reset, registrasi alumni; CSRF, hash password, regenerasi/invalidate session, cookie HttpOnly/Secure/SameSite.
- [ ] K2.03 Test IDOR dan akses salah role pada route, aksi Livewire, file download/export, endpoint massal dan data BK.
- [ ] K2.04 Upload validasi MIME/ukuran/ekstensi; nama acak; private storage; unduh file privat hanya melalui controller + otorisasi objek.
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

- [ ] F0.01 Gunakan repository yang sudah ada dan instal Laravel 13 pada PHP 8.5, Livewire 4, Tailwind CSS 4.
- [ ] F0.02 Siapkan `.env.example` tanpa rahasia, koneksi MariaDB, timestamp UTC, tampilan/tanggal efektif `Asia/Jakarta`, dan storage privat.
- [ ] F0.03 Gunakan modular monolith sederhana dalam struktur Laravel standar: routes, Controllers, Livewire, Models, Middleware dan resources; kelompokkan per fitur seperlunya tanpa paket/lapisan tambahan. Kode modul harus stabil.
- [ ] F0.04 Siapkan login, logout, reset password, session, CSRF, rate limit login, dan pembuatan Super Admin pertama yang aman.
- [ ] F0.05 Terapkan rincian menu per role/penugasan PRD §4.4 dan daftar fitur §4.3; pisahkan layout `/owner`, `/admin`, portal siswa/alumni sesuai hak akses; implementasikan sidebar hanya untuk modul aktif.
- [ ] F0.06 Buat komponen UI reusable: tabel, pencarian/filter, empty state, badge status, pagination, toast.
- [ ] F0.07 Buat modal reusable untuk konfirmasi, error penting, hasil proses; fokus keyboard, escape/close aman, responsif HP.
- [ ] F0.08 Implementasikan skeleton/indikator loading, spinner pada tombol, disable double-submit, validasi per field.
- [ ] F0.09 Siapkan audit log minimal untuk perubahan data penting tanpa mencatat kata sandi/token atau isi konseling.
- [ ] F0.10 Uji aplikasi tanpa Queue/Scheduler, tidak ada dependency worker yang harus aktif agar fitur berjalan.

**Selesai jika:** login/logout aman; UI responsif; tindakan hapus/ubah penting memakai modal; navigasi berdasar role; tes fondasi lulus.

## F1 — Core System dan data induk

**Dependensi:** F0. **Hasil:** data sekolah dan data siswa dapat dikelola tanpa duplikasi.

- [ ] F1.01 Migration profil sekolah, jurusan/program 3–4 tahun, mata pelajaran, tahun ajaran/semester, kelas/rombel.
- [ ] F1.02 Migration `siswa`, `guru`, `pegawai`, `pengguna` dan relasi akun, dengan NIS unik dan NISN unik jika tersedia.
- [ ] F1.03 Migration `kepesertaan_siswa` dan `penempatan_kelas_siswa` sebagai riwayat; **jangan hanya menyimpan `siswa.kelas_id`**.
- [ ] F1.04 CRUD profil sekolah, jurusan, mata pelajaran, tahun ajaran/semester, kelas, guru, staf, siswa; filter/pencarian sederhana.
- [ ] F1.05 Role dapat rangkap, permission berdasarkan tindakan **dan** objek yang ditugaskan; terapkan otorisasi objek pada Controller/Livewire dan Middleware.
- [ ] F1.06 CRUD akun yang diizinkan, nonaktifkan akun, reset akses; tidak membuat akun siswa/alumni ganda.
- [ ] F1.07 Setelah F4 tersedia, impor siswa/guru/kelas dari Excel/CSV dengan template, validasi, preview, penanganan duplikat; ekspor sesuai izin.
- [ ] F1.08 Tahun ajaran baru dimulai sebagai **draft**, diaktifkan secara eksplisit; pergantian tahun **tidak otomatis** menaikkan kelas.
- [ ] F1.09 Riwayat kelas terlihat per siswa; pindah kelas mencatat tanggal efektif tanpa menimpa penempatan lama.
- [ ] F1.10 Uji foreign key, indeks unik, validasi data, dan larangan menghapus histori penting melalui cascade.

- [ ] F1.11 Implementasikan `wali_siswa`/`siswa_wali`, relasi satu wali ke beberapa siswa, wali utama, dan verifikasi tautan akun wali sebelum akses data anak; akun wali opsional sesuai PRD §4.2/§5.2/§12.2.

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
- [ ] F2.08 Portal registrasi mandiri **alumni lama**, pencocokan identitas, review/verifikasi Pengelola Alumni, pencegahan klaim ganda dan penyamaran.
- [ ] F2.09 Buat `profil_alumni` dengan `siswa_id` nullable untuk alumni lama; proses tautkan arsip siswa bila cocok dan diverifikasi.
- [ ] F2.10 Profil alumni dan histori **pekerjaan, kuliah, wirausaha** yang dapat tumpang tindih; tanggal mulai/selesai dan status aktif.
- [ ] F2.11 Data status perkawinan/kejadian pribadi hanya **opsional dan privat**, tidak wajib untuk tracer umum.
- [ ] F2.12 Pengunduran diri, mutasi masuk/keluar, dan siswa yang tinggal kelas tidak membuat alumni fiktif.
- [ ] F2.13 Uji identitas dan riwayat akademik tetap utuh setelah naik, pindah, lulus, dan pendaftaran alumni lama.

- [ ] F2.14 Siapkan model/migration tagihan dan pembayaran historis sesuai PRD §12.4; uji snapshot tarif/kelas serta tunggakan tetap utuh setelah naik/pindah/lulus dengan fixture sintetis. Ini persiapan skema dan tes, bukan aktivasi modul Keuangan sebelum F5.09.

- [ ] F2.15 Uji koreksi kelulusan melalui peninjauan data terkait dan audit tanpa menghapus profil/tracer/tagihan; siswa mutasi keluar yang kembali memakai identitas lama (PRD S11–S12).

**Cakupan F2:** R1 menyelesaikan siklus siswa, kelulusan dan profil alumni Core. F2.08 (portal registrasi), F2.10–F2.11 (riwayat/pribadi alumni) serta bagian registrasi mandiri F2.13 diselesaikan pada R3. F2.09 menyiapkan identitas/relasi Core sejak R1; penggunaan portal alumni lama mengikuti R3. F2.14 hanya kontrak/model/fixture histori saat fondasi, penyelesaian Keuangan tetap R4.

**Selesai jika per rilis:** R1: kelas XII/XIII tidak otomatis alumni dan identitas/histori Core utuh. R3: registrasi/verifikasi/riwayat alumni juga lulus tanpa akun atau profil ganda. Item belum masuk rilis tetap tidak dicentang dan tidak menghalangi penerimaan R1.

## F3 — Owner, lisensi, dan hak akses modul

**Dependensi:** F0–F1. **Hasil:** pemilik SaaS dapat mengelola modul di tiap instalasi sekolah.

- [ ] F3.01 Buat `/owner` dengan akses Super Admin terproteksi (MFA wajib sebelum rilis produksi; pembatasan akses tambahan sesuai lingkungan operator).
- [ ] F3.02 Profil pelanggan **satu sekolah** pada VPS tersebut; tidak ada katalog banyak tenant di instalasi.
- [ ] F3.03 Seed 15 modul dengan kode stabil; modul yang tidak aktif tidak muncul di menu.
- [ ] F3.04 Aktivasi/nonaktifkan modul, masa berlaku, catatan perpanjangan, dan pencatatan pembayaran langganan manual.
- [ ] F3.05 Blokir mutasi/fitur operasional modul nonaktif; arsip baca/ekspor dan bukti pengiriman Core tetap sesuai hak. Terapkan lock/versi lisensi dan invalidasi cache sesuai PRD §13.5, tanpa 500 atau mengganggu modul lain.
- [ ] F3.06 Saat modul kedaluwarsa: operasi modul diblokir, **data tidak dihapus**; akses core dan prosedur ekspor/retensi tetap tersedia sesuai kebijakan.
- [ ] F3.07 Owner **tidak otomatis** dapat membaca data BK, informasi keluarga, pembayaran personal, dan data sensitif sekolah.
- [ ] F3.08 Audit setiap perubahan lisensi/modul/pembayaran owner; tes role bypass dan akses URL langsung.

**Selesai jika:** sekolah dengan hanya `surat` + `website` tidak dapat membuka modul lain walau menebak URL.

## F4 — Mesin proses massal simultan (tanpa Queue/Scheduler)

**Dependensi:** F0, skema data F1. **Digunakan oleh:** impor siswa/alumni, kenaikan, kelulusan, tagihan massal.

- [ ] F4.01 Migration `operasi_massal` (PK UUID) dan `item_operasi_massal` (FK `operasi_id` bertipe kompatibel, unique `operasi_id` + `kunci_entitas`), status, lease/token, dan error ringkas.
- [ ] F4.02 Endpoint authenticated/authorized: **preview → start → claim/process batch → status → pause/resume → retry failed**.
- [ ] F4.03 Snapshot target saat start agar hasil filter yang berubah tidak mengubah daftar sasaran proses berjalan.
- [ ] F4.04 Browser mengeksekusi maksimal **2–3 request HTTP paralel**, 10–25 item per request (konfigurasi; uji kapasitas VPS).
- [ ] F4.05 Claim item secara atomik; tiap item memakai transaksi dan validasi kunci bisnis agar retry **tidak membuat duplikat**.
- [ ] F4.06 Progress: `(berhasil + sudah_diproses + dilewati + gagal) / total_item × 100%` berdasarkan status **commit MariaDB**, bukan animasi.
- [ ] F4.07 Tampilkan progress bar, total, sukses, gagal, dilewati, tersisa, serta hasil ringkasan di modal.
- [ ] F4.08 **Pause** berhenti mengirim batch baru (request yang terlanjur berjalan boleh selesai); **resume** dari data server.
- [ ] F4.09 Saat koneksi putus/browser ditutup, baca status DB terlebih dahulu ketika kembali; selesaikan/reclaim item yang terputus secara aman.
- [ ] F4.10 Retry **hanya item gagal** yang boleh diproses ulang; cegah duplikasi dua tab browser dan dua petugas menjalankan operasi sama.
- [ ] F4.11 Simpan daftar item gagal, alasan yang aman ditampilkan, dan opsi unduh hasil.
- [ ] F4.12 Uji 100/600/1.000 siswa sintetis, refresh, timeout, dua browser, retry, dan tanpa data hilang/duplikat.

**Selesai jika:** 100% berarti semua item memperoleh hasil (bukan semua sukses), dan resume/retry tidak mengubah item yang telah berhasil.

---

## F5 — Modul aplikasi (15 produk, prioritas rilis R1–R4)

**Acuan cakupan:** setiap baris F5 wajib memenuhi seluruh fitur inti, proses bisnis, status, keluaran, dan aturan penting modul terkait pada PRD Bab 7; ringkasan tabel bukan pengganti rincian PRD. QR aset adalah opsional, bukan syarat MVP. Nomor F5 adalah ID tugas tetap; urutan pengerjaan mengikuti rilis R1–R4 pada PRD Bab 10, bukan urutan tabel; kode modul adalah identitas tetap.

**Syarat untuk setiap modul:** migrasi + validasi + role/permission + pemeriksaan lisensi + UI loading/modal + laporan bila dibutuhkan + pengujian proses utama. Modul harus berfungsi sendiri; integrasi PRD Bab 8 **wajib saat kedua modul terkait aktif**; saat pasangan tidak aktif, modul tetap berjalan dengan data minimum Core tanpa menduplikasi master bersama.

| Urutan | Kode modul | Minimum yang dikerjakan | Role dan alur selesai | Referensi HTML |
|---|---|---|---|---|
| 01 | `website` | CMS halaman, berita, pengumuman, galeri, profil jurusan, **menu/submenu dinamis** (urut/sembunyi/URL) | Editor buat → preview → terbit → pengunjung lihat | `12-website.html` |
| 02 | `surat` | Surat masuk/keluar, nomor unik, template, arsip; **file opsional** (catat metadata/editor/upload) | TU buat/catat → persetujuan seperlunya → arsip | `10-surat.html` |
| 03 | `absensi_izin` | **Presensi siswa per jam pelajaran**, konfirmasi wali kelas, penanganan bersama BK, izin siswa/guru dan penyelarasan per jam | Guru mapel simpan seluruh kelas hari ini → wali kelas konfirmasi tanpa tenggat → tindak lanjut bersama BK; S/I langsung dari wali kelas | `08-absensi.html` |
| 04 | `pelanggaran` | Tata tertib, bobot, catat, validasi, pembinaan, keberatan | Guru catat → Kesiswaan validasi → BK tindak lanjut | `13-pelanggaran.html` |
| 05 | `pkl` | Mitra, penempatan, jurnal, absensi, monitoring, nilai industri **diinput guru dari kertas** | Koordinator tempatkan → siswa jurnal → guru pantau/input nilai → laporan | `07-pkl.html` |
| 06 | `jurnal_guru` | Kegiatan dan hasil pekerjaan harian guru, draf/kirim/koreksi, riwayat dan rekap | Guru tulis kegiatan dan hasil → Kirim → Pimpinan lihat | `14-modul-lain.html` |
| 07 | `jadwal` | Hari/jam, pembagian guru, jadwal kelas, deteksi bentrok, cetak | Waka susun → cek bentrok → terbit | `14-modul-lain.html` |
| 08 | `bk` | Konseling, jadwal, tindak lanjut, statistik tanpa mengumbar rahasia | Guru BK tangani → tindak lanjut → rekap terbatas | `14-modul-lain.html` |
| 09 | `keuangan` | Tarif, tagihan, cicilan, kuitansi, tunggakan, koreksi/void ber-audit | Bendahara terbitkan → terima/catat → laporan | `09-keuangan.html` |
| 10 | `sarpras` | Master aset unit (QR opsional), kondisi, lokasi, peminjaman | Sarpras catat → pinjam → kembali | `11-sarpras.html` |
| 11 | `laboratorium` | Pakai **master aset yang sama**, stok bahan habis pakai, jadwal, peminjaman | Kepala Lab kelola → guru pinjam/pakai → kembali | `11-sarpras.html` |
| 12 | `hubin` | Master mitra bersama PKL, MoU, kegiatan, masa berlaku | HUBIN catat mitra → kerja sama → laporan | `14-modul-lain.html` |
| 13 | `bkk` | Lowongan, daftar alumni, lamaran, status hasil rekrutmen | BKK terbitkan → alumni melamar → BKK catat hasil | `06-alumni.html`, `14-modul-lain.html` |
| 14 | `tracer` | Form sederhana dan snapshot jawaban; analitik agregat | Alumni isi → Admin tracer periksa → laporan | `06-alumni.html`, `14-modul-lain.html` |
| 15 | `supervisi` | Jadwal, instrumen, nilai, tindak lanjut, laporan | Supervisor nilai → guru lihat/tindak lanjut → rekap | `14-modul-lain.html` |

### Pemetaan cakupan PRD ke tugas

| Bagian PRD | Kebutuhan | Task pelaksana | Bukti penerimaan |
|---|---|---|---|
| §3–4, §13, §15–16 | Arsitektur, Core, role, route dan keamanan | F0, F1, F3, K0–K2 | Tes akses role/objek, MFA, route publik/privat dan lisensi |
| §5.1–5.9, §12.2–12.4 | Identitas dan histori akademik/keuangan | F1, F2, F5.09, F6.01 | S01–S14, U04–U07, tes DB pada §14.3 |
| §5.10, §12.3 | Registrasi dan riwayat alumni | F2.07–F2.11, F5.13–F5.14 | A01–A09, U13–U14, tes BKK saja/Tracer saja |
| §5.4.1, §5.11, §12.7, §17.2 | Mesin massal dan pemulihan | F4, F7.09 | M01–M09, U15, BL01 dan tes bulk lain §14.3 |
| Bab 7 modul 01 | `pkl` | F5.05 | Semua fitur, alur, status, laporan dan aturan penting modul; tes role/lisensi dan UAT |
| Bab 7 modul 02 | `jurnal_guru` | F5.06 | Semua fitur, alur, status, laporan dan aturan penting modul; tes role/lisensi dan UAT |
| Bab 7 modul 03 | `absensi_izin` | F5.03 | Semua fitur, alur, status, laporan dan aturan penting modul; tes role/lisensi dan UAT |
| Bab 7 modul 04 | `surat` | F5.02 | Semua fitur, alur, status, laporan dan aturan penting modul; tes role/lisensi dan UAT |
| Bab 7 modul 05 | `keuangan` | F5.09 | Semua fitur, alur, status, laporan dan aturan penting modul; tes role/lisensi dan UAT |
| Bab 7 modul 06 | `bkk` | F5.13 | Semua fitur, alur, status, laporan dan aturan penting modul; tes role/lisensi dan UAT |
| Bab 7 modul 07 | `sarpras` | F5.10 | Semua fitur, alur, status, laporan dan aturan penting modul; tes role/lisensi dan UAT |
| Bab 7 modul 08 | `bk` | F5.08 | Semua fitur, alur, status, laporan dan aturan penting modul; tes role/lisensi dan UAT |
| Bab 7 modul 09 | `jadwal` | F5.07 | Semua fitur, alur, status, laporan dan aturan penting modul; tes role/lisensi dan UAT |
| Bab 7 modul 10 | `tracer` | F5.14 | Semua fitur, alur, status, laporan dan aturan penting modul; tes role/lisensi dan UAT |
| Bab 7 modul 11 | `hubin` | F5.12 | Semua fitur, alur, status, laporan dan aturan penting modul; tes role/lisensi dan UAT |
| Bab 7 modul 12 | `supervisi` | F5.15 | Semua fitur, alur, status, laporan dan aturan penting modul; tes role/lisensi dan UAT |
| Bab 7 modul 13 | `laboratorium` | F5.11 | Semua fitur, alur, status, laporan dan aturan penting modul; tes role/lisensi dan UAT |
| Bab 7 modul 14 | `pelanggaran` | F5.04 | Semua fitur, alur, status, laporan dan aturan penting modul; tes role/lisensi dan UAT |
| Bab 7 modul 15 | `website` | F5.01 | Semua fitur, alur, status, laporan dan aturan penting modul; tes role/lisensi dan UAT |
| Bab 8, §12.5–12.6 | Master bersama dan integrasi | F6, F7 | U08–U10; tes modul pasangan nonaktif, tanpa master ganda |
| Bab 14, §17.3, Bab 18 | QA dan bukti handover | F7–F9, K3 | Checklist kesiapan rilis pada dokumen ini, bukti test dan UAT sesuai cakupan |

### Checklist penyelesaian modul (duplikasi untuk setiap modul)

- [ ] F5.01 `website` — UI + backend + role + test selesai.
- [ ] F5.02 `surat` — UI/backend/role/test termasuk sumber nomor, arsip eksternal, revisi/pengganti dan pembatalan B0.01; upload tidak wajib.
- [ ] F5.03 `absensi_izin` — tabel `sesi_pelajaran`, `kehadiran_siswa` dan `tindak_lanjut_presensi` sesuai PRD §12.6, sesi per jam, input guru mapel, konfirmasi wali kelas, penanganan bersama BK, rekap dan izin per rentang waktu; UI/backend/otorisasi dan tes selesai, tanpa presensi guru.
- [ ] F5.04 `pelanggaran` — UI + backend + role + test selesai (validasi sebelum poin final).
- [ ] F5.05 `pkl` — UI + backend + role + test selesai (**tanpa akun industri**).
- [ ] F5.06 Jurnal kegiatan guru: `jurnal_kegiatan_guru` + `rincian_jurnal_guru`, kegiatan/hasil wajib, satu jurnal per guru/hari, kelas/jam/mapel opsional; tanpa ketergantungan jadwal/presensi/kalender. Draf/kirim/koreksi dan rekap sesuai PRD modul 02.
- [ ] F5.07 `jadwal` — UI + backend + role + test selesai (tanpa generator rumit).
- [ ] F5.08 `bk` — UI + backend + role + test selesai (data rahasia dibatasi).
- [ ] F5.09 `keuangan` — UI/backend/role/test serta B0.02: snapshot historis, penyesuaian, void administratif, refund nyata, saldo neto dan arsip aman.
- [ ] F5.10 `sarpras` — UI + backend + role + test selesai.
- [ ] F5.11 `laboratorium` — UI + backend + role + test selesai (aset sinkron).
- [ ] F5.12 `hubin` — UI + backend + role + test selesai (mitra bersama PKL).
- [ ] F5.13 `bkk` — lowongan, lamaran/CV opsional, status seleksi dan laporan; gunakan registrasi/profil alumni F2, uji BKK aktif tanpa Tracer/Website.
- [ ] F5.14 `tracer` — periode/form survei, snapshot jawaban historis dan laporan agregat; aktivitas alumni paralel; uji Tracer aktif tanpa BKK/Website dan perubahan profil tidak menimpa jawaban lama.
- [ ] F5.15 `supervisi` — UI + backend + role + test selesai.
- [ ] F5.16 Izin siswa S/I dari wali kelas langsung berlaku tanpa pejabat kedua; sumber telepon/WhatsApp/surat/tatap muka, tanpa wajib akun/lampiran; radio presensi otomatis, konflik versi ditolak utuh. Izin guru tetap persetujuan pejabat.

## B0 — Penyelesaian alur administrasi, alumni dan sarana

ID B0 melengkapi F5/F6/F7 dan mengikuti rilis R2–R4; bukan modul/role tambahan. Seluruhnya membutuhkan bukti implementasi dan tes sebelum dicentang.

- [ ] B0.01 Surat: bedakan keluaran sistem/arsip eksternal, nomor asli dan nomor final; uji finalisasi paralel/idempotent, revisi ber-snapshot, surat pengganti dan pembatalan final yang tidak memakai ulang nomor (PRD modul 04).
- [ ] B0.02 Keuangan: penyesuaian tagihan tanpa edit snapshot; void administratif tanpa uang nyata; pengembalian uang nyata terpisah; tagihan Menunggu Penyelesaian/batal dan buka kembali record yang sama. Uji saldo efektif, uang neto, pembatalan setelah refund, pembalik refund pada tagihan batal yang wajib buka kembali atomik atau ditolak, idempotensi serta dua koreksi/pengembalian paralel (PRD modul 05/§12.4).
- [ ] B0.03 BKK: UNIQUE pelamar/lowongan, tenggat server tanpa cron, tarik/ajukan ulang pada record sama, pembatalan lowongan, snapshot syarat dan koreksi hasil; Diterima tidak otomatis menambah pekerjaan aktif (PRD modul 06).
- [ ] B0.04 Sarpras: satu ID per unit, permohonan/rincian serah-terima, kembali sebagian, kondisi rusak, kehilangan dan penghapusan administratif; uji pinjam paralel dan histori lintas lokasi, tanpa aset salinan Lab (PRD modul 07/§12.5).
- [ ] B0.05 Tracer: versi form terkunci per periode, satu respons logis alumni/periode, revisi jawaban tersimpan, koreksi setelah tutup lewat akses terbatas; uji laporan satu responden, pertanyaan asli tetap terbaca dan profil baru tidak menimpa snapshot (PRD modul 10).
- [ ] B0.06 HUBIN: adendum/perpanjangan sebagai versi/record baru, batas waktu saat request, pembatalan beralasan; uji MoU habis tidak membatalkan PKL/jurnal/nilai dan tidak menduplikasi mitra (PRD modul 11).
- [ ] B0.07 Lab: batal reservasi sebelum digunakan, bentrok paralel, serah-terima unit bersama, pemakaian dan penyesuaian stok melalui mutasi/pembalik immutable; uji saldo tidak minus dan pembatalan reservasi tidak mengembalikan bahan yang telah dipakai (PRD modul 13).
- [ ] B0.08 Website: draf revisi terpisah dari versi terbit, publikasi atomik/cache setelah commit, konflik dua editor, restore versi, URL/menu stabil; uji media yang dirujuk draf/revisi/terbit tidak dapat dihapus dan kelima tema/admin tetap konsisten (PRD modul 15).

## F6 — Integrasi data lintas modul

**Dependensi:** modul terkait ada. Jika salah satunya tidak aktif, modul yang lain tetap berfungsi.

- [ ] F6.01 **Kenaikan/kelulusan ↔ Keuangan:** tagihan lama tetap menunjuk `siswa_id`, tarif dan kelas saat diterbitkan tidak berubah; tunggakan tidak hilang.
- [ ] F6.02 **Kelulusan ↔ Alumni/BKK/Tracer:** alumni baru ditautkan ke siswa yang lulus, alumni lama tidak membuat siswa fiktif.
- [ ] F6.03 **Sarpras ↔ Laboratorium:** satu `aset.id` untuk aset unit yang sama, perubahan kondisi/lokasi muncul di dua modul.
- [ ] F6.04 **Absensi ↔ Izin:** izin siswa diselaraskan hanya ke sesi/jam dalam rentang izin; perubahan sesudah konfirmasi diaudit dan meminta konfirmasi ulang wali kelas.
- [ ] F6.05 **PKL ↔ HUBIN:** satu tabel mitra industri; data perusahaan yang sama tidak diduplikasi.
- [ ] F6.06 **Jurnal Kegiatan Guru ↔ Presensi/Jadwal:** tidak saling menjadi syarat; jurnal kegiatan/hasil harian bukan jurnal pertemuan. Presensi memakai penugasan mengajar, jadwal hanya saran.
- [ ] F6.07 **Pelanggaran ↔ BK:** hanya rujukan tindak lanjut minimal; isi konseling tidak bocor ke pelapor.
- [ ] F6.08 **Website ↔ lisensi:** jika website tidak aktif, admin sekolah tetap bisa login ke aplikasi melalui domain sekolah.

**Selesai jika:** menonaktifkan modul pasangan tidak menimbulkan error pada modul yang masih aktif.

**Skenario penerimaan terbaru:** OP01–OP13 pada PRD §14.7 dan AU01–AU21 pada §14.8 wajib diuji sesuai rilis. Prototipe tema bukan bukti CMS produksi selesai.

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
- [ ] F7.09 Uji penanganan gangguan koneksi ketika bulk berjalan, status pemulihan, pause/resume/retry, snapshot 600 siswa lintas halaman, lease kedaluwarsa, serta pencabutan role/lisensi di tengah operasi (request berikutnya ditolak tanpa membatalkan hasil yang sudah commit).
- [ ] F7.10 Uji performa pada **spesifikasi VPS target yang sebenarnya**, termasuk pembuatan laporan dan 2–3 batch HTTP paralel.
- [ ] F7.11 Lakukan review privasi: BK, catatan pelanggaran, informasi keluarga, dokumen, foto siswa, dan audit log.
- [ ] F7.12 Jalankan tes otomatis serta UAT operator sekolah (TU, guru, wali kelas, BK, Bendahara, HUBIN, Admin).

**Selesai jika:** tidak ada bug kritis/tinggi terbuka; tes kritis lulus; pengguna bisa menyelesaikan alur utama tanpa instruksi teknis.

- [ ] F7.14 Uji presensi hari ini dari server Jakarta tanpa pemilih tanggal, semua H awal belum tersimpan, otomatis S/I wali kelas, kirim seluruh kelas satu transaksi; satu baris gagal rollback semua. Uji payload kurang/duplikat/tambahan, timeout setelah commit, retry token sama, dua tab, izin berubah, pergantian hari dan tidak ada sukses palsu. Konfirmasi wali kelas tanpa tenggat; koreksi lampau khusus wali/delegasi; alur BK lengkap tanpa bocor konseling.

- [ ] F7.15 Uji jurnal kegiatan nonmengajar tanpa kelas/mapel/jadwal; kegiatan dan hasil wajib, beberapa rincian dalam satu hari, draf bukan terkirim, guru lain ditolak, tanggal lampau diizinkan untuk jurnal saja; tidak ada hitungan pelanggaran kalender.
- [ ] F7.16 Uji menu tiap role/penugasan sesuai PRD §4.4, lisensi aktif/nonaktif, gabungan role tanpa duplikasi, akses URL langsung dan ekspor; wali tanpa akun tetap dapat menyampaikan izin melalui petugas dan menjalani keputusan sah.

## F8 — Validasi rilis aplikasi (infrastruktur terpisah)

**Dependensi:** F7 lulus untuk cakupan modul yang akan dirilis.

- [ ] F8.01 Serahkan daftar kebutuhan runtime yang harus disediakan operator, tanpa melakukan pengaturan VPS, DNS, Nginx, SSL, firewall, atau SSH.
- [ ] F8.02 Verifikasi konfigurasi runtime produksi yang dipasok operator: `APP_DEBUG=false`, `APP_ENV=production`, cookie aman, `.env` tidak terbuka melalui aplikasi.
- [ ] F8.03 Deployment reproducible: build aset, migration aman, cek kompatibilitas skema, cache config/view/route sesuai aplikasi.
- [ ] F8.04 Dokumentasikan aset data aplikasi yang harus dicakup backup (MariaDB dan file privat), serta pemeriksaan konsistensi aplikasi usai restore oleh operator; **tanpa menulis skrip backup host**.
- [ ] F8.05 Setelah tim infrastruktur memulihkan salinan data uji, jalankan smoke test integritas siswa, keuangan, file privat, dan hak akses dari sisi aplikasi.
- [ ] F8.06 Siapkan log error aplikasi yang aman dan panduan pemeriksaan error Laravel. Monitoring infrastruktur ditangani terpisah.
- [ ] F8.07 Terapkan kontrak upgrade PRD §14.6: tag/versi aplikasi-skema-runtime, migration additive, uji rilis lama→baru, data/lisensi/tema utuh, rollback source kompatibel atau perbaikan maju; tidak reset database sekolah.
- [ ] F8.08 Jalankan uji coba **satu sekolah** dan verifikasi akses aplikasi melalui URL yang tersedia, akun, lisensi, data, hak akses, serta laporan.
- [ ] F8.09 Tetapkan rilis hanya untuk **modul yang telah lulus QA**; yang belum selesai tidak boleh ditawarkan/diaktifkan.
- [ ] F8.10 Dokumentasikan hasil akhir, migrasi, panduan admin, panduan owner, dan catatan rilis.

### Gerbang rilis per cakupan (semua yang relevan wajib terbukti)

ID R1–R8 pada checklist berikut adalah ID gerbang lama, bukan nama paket rilis. Catat paket target R1/R2/R3/R4 pada informasi rilis. Gerbang Core/keamanan selalu wajib; modul yang belum masuk paket dicatat **Belum masuk rilis**, tidak dicentang lulus. Jangan menunda R1 karena fitur Keuangan/Sarpras R4 belum dibangun.

- [ ] R1 Semua fungsi yang dijual benar-benar mempunyai backend Laravel/MariaDB (bukan simulasi HTML).
- [ ] R2 Isolasi akses role + lisensi teruji dari UI hingga endpoint backend.
- [ ] R3 Histori siswa/kelas/kelulusan dan profil alumni Core konsisten sejak R1; portal alumni R3 serta tagihan/aset R4 diuji ketika masuk rilis atau sudah terpasang dan terdampak upgrade.
- [ ] R4 Operasi massal paralel terbatas menunjukkan progress nyata, aman retry, tanpa duplikasi.
- [ ] R5 Modal/loading/error/validasi benar dan responsif pada desktop/HP.
- [ ] R6 Tes baca ulang database dan file pasca-restore yang disediakan tim infrastruktur lulus; dokumentasikan hasil dan dependensi yang masih dipegang operator.
- [ ] R7 Keamanan aplikasi, penyimpanan berkas privat dan proteksi data diuji; keamanan server/HTTPS menjadi prasyarat terpisah dari operator VPS.
- [ ] R8 Pengguna sekolah lulus UAT pada alur penting, tanpa bug kritis terbuka.

---

## F9 — Bukti kualitas dan handover aplikasi

**Dependensi:** F7/F8. **Hasil:** bagian Checklist kesiapan rilis aplikasi dan artefak aplikasi yang bisa dievaluasi secara independen.

- [ ] F9.01 Isi checklist go-live aplikasi disertai bukti test (nama test, commit, screenshot/error log teredaksi, penguji, tanggal).
- [ ] F9.02 Dokumentasikan daftar 15 modul beserta status: siap diuji / lulus / belum diimplementasikan. Jangan mengaktifkan penjualan modul yang belum selesai.
- [ ] F9.03 Serahkan release tag, source code, build instructions, migration/seeder, `.env.example`, route map, panduan role, dan panduan penggunaan inti.
- [ ] F9.04 Pastikan dokumentasi tidak memuat domain/IP rahasia, password sekolah, NISN nyata, atau data BK/keuangan asli.
- [ ] F9.05 Handover kontrak kebutuhan aplikasi ke tim infrastruktur, **tanpa** konfigurasi server; penerimaan operasi host dilakukan terpisah.
- [ ] F9.06 Catat keputusan **READY FOR APP HANDOVER / NO-GO / LIMITED PILOT** serta pemberi persetujuan sekolah dan developer. Status 'READY FOR APP HANDOVER' tetap bukan klaim infrastruktur telah siap.

**Selesai jika:** bagian Checklist kesiapan rilis aplikasi berisi bukti pemeriksaan seluruh gerbang wajib dan tidak ada blocker aplikasi.

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

**Catatan:** kontrak baru PRD Bab 15–17 dan checklist K0–K3 wajib dipenuhi; `TASK.md` adalah checklist pelaksanaan, bagian Panduan implementasi memandu cara coding, PRD menentukan kebutuhan, dan HTML menunjukkan contoh pengalaman pengguna. Semua file harus dibaca bersama.


---

## Panduan kerja developer


Baca file ini **sebelum mengubah kode**. Ini repository proyek aplikasi sekolah, **belum** aplikasi Laravel saat paket awal diunggah. Kerjakan pekerjaan secara bertahap dan beri bukti hasil uji; jangan menyatakan fitur selesai karena sudah ada mockup.

## Sumber kebenaran (urut prioritas)

1. `PRD.md` — fungsi, alur, arsitektur data, batasan scope, keamanan, dan kriteria penerimaan.
2. `TASK.md` — urutan pekerjaan, ID tugas, dan Definition of Done.
3. bagian Checklist kesiapan rilis aplikasi — bukti kelulusan rilis aplikasi, bukan klaim server sudah siap.
4. bagian Panduan implementasi — instruksi implementasi tambahan.
5. `prototype/` — acuan tampilan dan interaksi **simulasi saja**, bukan kode backend.

Jika ada kontradiksi, utamakan PRD; beri catatan konflik pada laporan progres. Jangan mengikuti nama berkas PRD lama yang tercetak pada sebagian mockup; dokumen otoritatif adalah `PRD.md` di root.

## Arsitektur dan scope tetap

- Laravel **13**, PHP **8.5**, Livewire **4**, Blade, Tailwind CSS **4**, MariaDB; Nginx hanya target runtime milik tim infrastruktur.
- **Satu instalasi Laravel + satu database per sekolah**, seluruh 15 modul dalam satu codebase. **Tidak** multi-tenant, tidak ada domain/API/panel SaaS pusat.
- Owner SaaS lokal di `/owner`; admin operasional sekolah di `/admin`. Lisensi setiap modul dikendalikan lokal.
- **Dilarang menambahkan Laravel Queue, Scheduler, Redis worker, cron aplikasi, WebSocket, microservices, PPDB, UKK, absensi guru, pembayaran gateway, atau akun login pembimbing industri PKL.**
- Pengaturan VPS, domain, DNS, Nginx, TLS, firewall, monitoring server, dan backup host **di luar tanggung jawab developer aplikasi**. Developer hanya menyiapkan kode dan kontrak handover aplikasi.
- Jangan menurunkan versi stack secara diam-diam. Jika versi yang dipersyaratkan tidak tersedia di environment tugas, jelaskan hambatannya dan lanjutkan bagian yang bisa diverifikasi secara jujur.

## Persyaratan integritas data yang harus dijaga

- `siswa.id` permanen. Riwayat kelas dan tahun ajaran dalam tabel histori; kenaikan kelas dan kelulusan terpisah, termasuk program 4 tahun dengan kelas XIII.
- Alumni lama dapat daftar sendiri dan harus diverifikasi; alumni dari siswa aktif yang lulus mempertahankan kaitan identitas dan akun bila ada. Alumni bisa bekerja, kuliah, dan berwirausaha secara bersamaan.
- Invoice menyimpan snapshot kelas/tarif saat dibuat. Kenaikan/pindah kelas/kelulusan tidak mengubah riwayat tagihan maupun pembayaran.
- Aset Sarpras dan Laboratorium memakai master yang sama. Stok bahan habis pakai menggunakan pencatatan tersendiri.
- Penilaian pembimbing industri PKL dicatat oleh guru dari lembar kertas, bukti lampiran opsional; surat bisa disimpan tanpa file.
- Absensi **hanya siswa**, sementara izin siswa dan izin guru merupakan satu modul yang sama.
- Website CMS memiliki menu/submenu dinamis, bukan menu publik hardcoded.

## Pola coding dan keamanan

- Gunakan fitur resmi framework bila cocok; kode mudah dibaca lebih penting daripada abstraksi berlebihan.
- Seluruh form tervalidasi. Otentikasi, role dan permission berbasis objek harus berlaku di route, Livewire action, ekspor, unduhan, dan endpoint batch.
- `GET` tidak boleh mengubah data. Perubahan kritis diaudit tanpa menyimpan sandi, token atau isi catatan konseling.
- File privat menggunakan validasi MIME/ukuran, nama acak, private storage, dan download terotorisasi. CMS harus aman terhadap XSS.
- Tidak ada secret atau data siswa nyata dalam repo, fixture, log, maupun screenshot.
- Owner diberi perlindungan MFA sebelum dianggap siap rilis. Akses owner tidak otomatis membuka data BK/keuangan personal.
- Proses massal menggunakan **HTTP batch paralel terbatas, 2–3 request aktif, 10–25 item/batch** sebagai konfigurasi awal yang diuji; simpan progress aktual DB, idempotensi, pause/resume/retry. Tanpa proses latar.
- UI wajib: modal konfirmasi untuk aksi berisiko, spinner/loading nyata, validasi per field, toast, empty/error state, dan pengalaman mobile yang baik. Jangan memakai `window.alert()`/`window.confirm()`.

## Cara mengerjakan setiap permintaan

1. Mulai dengan membaca bagian Panduan kerja developer dan tugas F0/K0 pada `TASK.md`. Jangan membuat seluruh modul dalam satu perubahan besar.
2. Periksa keadaan repository: jika proyek Laravel belum ada, scaffold di root **tanpa menghapus** dokumentasi/prototipe. Jika sudah ada, jangan scaffold ulang.
3. Sebelum coding, rangkum target task ID, file yang akan disentuh, desain singkat, dan tes yang perlu lulus.
4. Kerjakan satu irisan fungsional yang selesai dan dapat diuji; tambahkan migration pendukung, otorisasi objek, komponen Livewire, serta tes yang relevan.
5. Jalankan test yang memungkinkan, termasuk `php artisan test` dan `npm run build` setelah aplikasinya tersedia; laporkan jika tool/database lokal tidak ada, jangan mengarang hasil lulus.
6. Perbarui kotak di `TASK.md` **hanya bila** kode dan pengujiannya benar-benar selesai, beserta bukti/commit atau ringkasan di bagian Catatan progres implementasi.
7. Berikan ringkasan hasil, perintah uji, risiko, dan langkah konkret berikutnya. Jangan klaim production-ready tanpa melewati bagian Checklist kesiapan rilis aplikasi.

## Panduan prioritas

- F0: fondasi aplikasi dan autentikasi.
- K0–K2: routing, integritas database, dan keamanan sejalan dengan fondasi.
- F1: master data dan riwayat penempatan siswa.
- F4 lalu F2: mesin operasi massal sebelum kenaikan kelas/kelulusan massal.
- F3: lisensi lokal dan panel owner.
- F5–F6: modul, bertahap menurut ketergantungan.
- F7–F9: QA, kriteria kesiapan aplikasi, dan handover.

**Catatan:** tugas di repositori ini mencakup software development saja. Tim lain menangani penayangan dan operasi infrastrukturnya.


---

## Panduan implementasi


**Versi:** 1.1 · **Acuan:** `PRD.md` · **Checklist:** `TASK.md` · **Prototype UI:** `prototype/index.html` dan halaman `01-owner.html` s.d. `14-modul-lain.html`

> **Cara pakai:** salin bagian **PROMPT UTAMA** ke asisten coding yang bekerja di repository Laravel. Sertakan file PRD, `TASK.md`, dan folder HTML prototipe sebagai referensi. Kerjakan task berdasarkan dependensinya, dan centang hanya setelah pengujian lulus. Implementasikan **bertahap**, mulai dari fondasi; jangan langsung menganggap contoh HTML sebagai aplikasi produksi.

---

## PROMPT UTAMA (siap salin)

Kamu adalah software engineer senior Laravel, perancang database, dan QA engineer. Bangun aplikasi manajemen SMK yang **sederhana bagi operator sekolah tetapi aman dan konsisten secara data**, mengikuti dokumen `PRD.md` dan referensi tampilan HTML dalam folder ini.

### 1. Ketentuan mutlak proyek

- Stack: **Laravel 13, PHP 8.5, Blade, Livewire 4, Tailwind CSS 4, Nginx, MariaDB**.
- **Satu sekolah = satu VPS = satu instalasi Laravel = satu database MariaDB**. Bukan multi-tenant. Domain sekolah `.sch.id`; tidak ada server/domain SaaS pusat.
- **Super Admin SaaS berada secara lokal** pada `/owner`, terpisah dari `/admin` sekolah. Semua VPS sekolah memakai satu repository sumber yang dapat dideploy secara terpisah.
- Core tersedia meskipun hanya satu modul dibeli. Katalog menetapkan 15 kode modul dalam satu codebase; fitur dikembangkan sesuai R1–R4. Modul belum dirilis/nonaktif tidak dapat dijalankan operasionalnya; arsip baca/ekspor tetap melalui Core sesuai kelayakan, hak data dan lingkup §13.6.
- **JANGAN gunakan Laravel Queue, Laravel Scheduler, job worker, cron aplikasi, WebSocket, Redis, microservices, pembayaran gateway, integrasi AI, maupun akun industri PKL**. Jangan menambahkan layanan pusat untuk lisensi.
- Tidak membuat PPDB, UKK, absensi guru, atau modul Perizinan terpisah. **Absensi siswa dan perizinan siswa/guru** ada dalam satu modul.
- Jangan mengganti stack, menambah fitur di luar MVP, atau membuat proses approval berlapis tanpa alasan operasional yang jelas.
- Prioritas: UX sederhana, data benar, keamanan, dan tugas operator dapat selesai dengan sedikit langkah.
- Referensi kebutuhan terperinci adalah PRD. Jika contoh HTML berlawanan dengan aturan PRD, **ikuti PRD**. HTML adalah simulasi antarmuka, bukan backend siap pakai.

### 1A. URL, database, dan keamanan tidak boleh dibiarkan sebagai asumsi

- **Baca Bab 15–17 PRD dan checklist K0–K3 pada `TASK.md` sebelum membuat route/migration.** Buat pemetaan named routes untuk `/`, `/login`, `/owner`, `/admin`, `/admin/siswa`, `/admin/kenaikan-kelas`, `/admin/kelulusan`, `/siswa`, `/alumni`, `/alumni/daftar`, dan 15 modul pada Bab 15.
- Aplikasi menggunakan **named routes dan path relatif**; alamat host ditentukan operator VPS saat deployment. **Jangan membuat konfigurasi domain/DNS/Nginx/SSL dalam tugas coding aplikasi**. Tidak ada domain pusat atau multi-tenant.
- Koneksi MariaDB lokal dengan `DB_CONNECTION=mariadb`, `DB_HOST=127.0.0.1`; gunakan `.env.example` tanpa rahasia dan akun DB dengan hak minimum. Jangan mencantumkan `APP_KEY`, password, atau data sekolah nyata pada kode/dokumentasi.
- Akses owner **harus MFA sebelum produksi** dan dibuat tanpa kredensial default. CSRF, rate limiting, session aman, otorisasi objek (pencegahan IDOR), lisensi modul, upload privat, unduhan terotorisasi, validasi, sanitasi CMS, dan audit log minim data adalah kewajiban.
- Data siswa dan keuangan harus menggunakan unique index/FK/locking sesuai Bab 12 dan 17. Batch paralel menyimpan lease/token dan melakukan pemulihan saat request masuk, **bukan lewat Queue/Scheduler**. Pembayaran dan kelulusan selalu idempotent.
- Kerjakan dan uji kontrak URL/DB/keamanan di tahap F0–F4. `TASK.md` hanya dicentang setelah ada implementasi dan bukti test, bukan berdasarkan teks PRD atau demo HTML.

### 2. Daftar modul (kode stabil, bukan nomor lisensi yang dapat berubah)

1. `pkl` — Manajemen PKL
2. `jurnal_guru` — Jurnal Kegiatan Harian Guru
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

- **`siswa.id` permanen.** Naik kelas, pindah kelas, mutasi, lulus, dan menjadi alumni tidak mengganti ID siswa.
- Penempatan akademik disimpan sebagai riwayat melalui `kepesertaan_siswa` (**UNIQUE student + academic_year**) dan `penempatan_kelas_siswa` (tanggal efektif; tidak boleh tumpang tindih). Jangan mengandalkan `siswa.kelas_id` tunggal sebagai sumber histori.
- Tahun ajaran baru dibuat sebagai draft dan diaktifkan secara eksplisit; **tidak otomatis menaikkan kelas**. Ada tampilan siswa belum diproses.
- Kenaikan kelas **massal** mendukung pilih semua hasil filter, pengecualian, tinggal kelas, kelas tujuan, pratinjau, konfirmasi, progress, resume, dan daftar gagal. Program 4 tahun mendukung XII→XIII.
- **Kelulusan terpisah dari kenaikan.** Kelas XII program 3 tahun atau XIII program 4 tahun tidak otomatis menjadi alumni. Catat satu kelulusan resmi per siswa. Siswa tidak lulus tetap berstatus sesuai keputusan sekolah.
- Lulusan yang sebelumnya siswa **menggunakan akun yang sama** bila tersedia. Alumni lama boleh **daftar sendiri**, diverifikasi sebelum memperoleh profil resmi, dan tidak perlu dibuat siswa fiktif. Wajib cegah klaim identitas alumni milik orang lain.
- `profil_alumni.siswa_id` boleh null, tetapi bila terisi harus unik. Riwayat pekerjaan, kuliah, usaha dapat **berlangsung paralel**; pernikahan adalah data opsional/privat, tidak dipaksakan.
- **Keuangan memakai FK `siswa_id` permanen** dan snapshot (`nama_kelas_saat_terbit`, `nama_biaya_saat_terbit`, `nominal_saat_terbit`) saat tagihan dibuat. Pergantian kelas atau kelulusan tidak mengubah tagihan lama. Pembayaran cicilan tidak boleh melebihi sisa tagihan; transaksi pembetulan/void diaudit.
- **Sarpras dan Laboratorium berbagi satu master aset**; bahan habis pakai punya stok kuantitas sendiri. Tidak boleh dua aset logis untuk barang yang sama, stok negatif, atau dua peminjaman aktif untuk satu aset unit.
- Nilai dari pembimbing industri **diinput oleh guru pembimbing berdasarkan lembar kertas**, dengan nama penilai dan tanggal penilaian; scan/foto bukti opsional. Tidak ada login pembimbing industri.
- Surat boleh **dicatat tanpa file, dibuat lewat editor, atau diunggah**. Upload bukan field wajib.
- Website sekolah punya **menu/submenu dinamis**, urutan, jenis tautan internal/eksternal, dan visibilitas; tidak hardcoded di template.
- Gunakan MariaDB **foreign key, unique index, transaksi, dan lock baris** untuk data penting. Hindari cascade delete terhadap histori siswa, alumni, aset, dan pembayaran.

### 4. Otentikasi, lisensi, role

- Gunakan autentikasi resmi Laravel; lakukan otorisasi server-side di Route/Middleware dan otorisasi objek serta tiap aksi Livewire, endpoint download, ekspor, dan operasi massal.
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
3. Setelah konfirmasi, buat record `operasi_massal` dan `item_operasi_massal` dalam MariaDB (unique per operasi + entitas).
4. Browser mengirim **maksimum 2–3 HTTP request paralel**, masing-masing klaim **10–25 item** secara atomik. Angka ini konfigurasi awal dan harus diuji di VPS.
5. Setiap item diproses dengan transaksi, pengecekan idempotensi, constraint, dan locking bila diperlukan. Request paralel tidak boleh mengerjakan siswa/tagihan yang sama dua kali.
6. Progress nyata: **(berhasil + sudah_diproses + dilewati + gagal) / total × 100%**. Tampilkan berhasil/gagal/dilewati dan item belum selesai. 100% berarti semua mendapat hasil, **bukan** semuanya berhasil.
7. **Jeda** menghentikan permintaan batch berikutnya. **Lanjutkan** membaca status yang tersimpan; **Ulangi yang gagal** hanya pada item yang benar-benar boleh diulang. Jangan membatalkan transaksi yang sudah berhasil.
8. Jika browser ditutup, tidak ada pekerjaan baru berjalan di background. Request yang sudah terkirim dapat selesai; ketika login kembali operator dapat memeriksa status dan melanjutkan. Jangan menganggap timeout sebagai bukti gagal.
9. Simpan error ringkas yang aman bagi operator; berikan daftar gagal yang bisa diunduh. Semua endpoint preview/start/process/status/pause/resume/retry memiliki autentikasi dan otorisasi.

### 7. Urutan implementasi (gunakan ID checklist, bukan penomoran fase lain)

1. **F0 + K0–K2 terkait:** fondasi, autentikasi, permission, konfigurasi MariaDB, layout dan tes dasar.
2. **Skema/master F1:** identitas siswa/guru/wali, kelas dan histori akademik; siapkan kontrol lisensi minimum F3.03/F3.05 sebelum portal alumni dibuka.
3. **F4 lalu F1.07:** bangun dan uji mesin massal, kemudian integrasikan impor. Jangan menunggu impor selesai untuk membangun F4.
4. **F2:** siklus siswa, kenaikan/kelulusan, alumni dan registrasi/verifikasi; F2.14 menyiapkan model tagihan/pembayaran dan tes historis.
5. **Selesaikan F3:** owner, MFA, lisensi lokal dan pembayaran langganan manual.
6. **F5/F6:** ikuti R1–R4 PRD Bab 10; dahulukan F5.03/F5.06, modul lain bertahap serta integrasi pasangan yang sudah ada. Keuangan lengkap berada di F5.09.
7. **F7 → F8/F9 + K3:** QA, keamanan, UAT, bukti rilis dan handover aplikasi; infrastruktur dikelola terpisah.

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
- S/I wali kelas langsung konsisten dengan presensi; tidak ada absensi guru; surat bisa disimpan tanpa file.
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

**Mulai hanya dari F0.01–F0.03 dan persiapan K0/K1 yang diperlukan; tahap selanjutnya mengikuti checklist.** Jangan membangun 15 modul sekaligus dan jangan mengarang hasil testing atau membuat data produksi palsu.

---

## Catatan terakhir

Paket ini **siap untuk review desain dan mulai development**, **belum siap produksi**. Produksi membutuhkan aplikasi Laravel/MariaDB yang selesai, bukti testing, keamanan, verifikasi aplikasi setelah restore oleh operator, serta pilot sekolah sesuai Bab 14 PRD.


## 10. Definition of Done dan batas tanggung jawab (FINAL)

Sebelum mencentang satu item pada `TASK.md`, tulis: file yang berubah, perubahan migration, pengujian yang dijalankan, bukti hasilnya, dan risiko yang tersisa. Jika belum ada code dan test, status adalah **belum selesai**.

**Developer hanya menulis aplikasi.** Jangan menghasilkan skrip untuk provisioning VPS, konfigurasi DNS/domain, Nginx virtual host, sertifikat SSL, firewall/SSH, monitoring host, atau backup host. Boleh mendokumentasikan variabel lingkungan yang dibutuhkan kode (`.env.example`, tanpa nilai rahasia), bukan menyediakan nilai production.

Gunakan bagian Checklist kesiapan rilis aplikasi sebagai gerbang final. Sebut aplikasi *ready for handover* hanya setelah fitur rilis lulus test dan UAT. Jangan mengaku *go-live* sampai penyedia infrastruktur dan sekolah juga menyatakan siap.

Setiap permintaan membuat fitur harus mempertahankan seluruh keputusan PRD: 15 modul, PKL nilai kertas diinput guru, absensi hanya siswa, izin siswa+guru satu modul, surat boleh tanpa file, alumni lama self-register diverifikasi, kelas XII→XIII bila 4 tahun, kelulusan tidak otomatis, tagihan dan aset tetap konsisten, menu CMS dinamis, bulk HTTP paralel progres DB, modal/indikator loading, dan **tanpa Queue/Scheduler**.


---

## Checklist kesiapan rilis aplikasi

Gerbang modul/integrasi diuji sesuai paket rilis Bab 10 dan regresi modul terdahulu yang sudah terpasang. Core, keamanan dan mesin massal yang digunakan wajib sejak R1. B07–B08 dan portal alumni diterapkan pada R3; B09–B10/keuangan pada R4; C01 PKL pada R3; C03/C06 Surat/Website pada R2; C04 Sarpras/Lab pada R4; C02/C07 serta keamanan lisensi/arsip berlaku sejak R1. Item belum masuk paket diberi alasan **Belum masuk rilis**, bukan centang lulus. Data historis modul yang sudah terpasang tidak boleh dikecualikan dari regresi upgrade hanya karena lisensinya kini nonaktif.


**Versi:** 1.0 — 4 November 2026  
**Scope:** aplikasi Laravel + MariaDB **saja**. Bukan checklist VPS/domain/DNS/Nginx/SSL/firewall/jaringan.  
**Status awal:** BELUM DIUJI / NO-GO. Jangan menandai siap produksi hanya berdasarkan PRD, prompt, atau HTML.

## 1. Informasi rilis dan bukti

| Item | Diisi saat verifikasi |
|---|---|
| Release tag / commit | Belum ada |
| Tanggal pengujian | Belum ada |
| Lingkungan pengujian | Belum disediakan |
| Versi Laravel / PHP / Livewire / MariaDB | Verifikasi dari aplikasi nyata |
| Modul yang akan dirilis (kode stabil) | Belum dipilih |
| Penguji (developer) | Belum ditetapkan |
| Penguji (sekolah/UAT) | Belum ditetapkan |
| Status keputusan | **NO-GO — belum ada aplikasi siap diuji** |

**Aturan bukti:** setiap centang harus dilengkapi minimal nama test dan hasil, bukti screenshot/rekaman yang sudah menutupi data pribadi, atau nomor commit/issue. Isi log uji di tabel Bagian 8.

## 2. Gerbang A — Fondasi aplikasi dan database

- [ ] A01 Laravel 13, Livewire 4, Tailwind CSS 4, dan PHP 8.5 berjalan di lingkungan pengujian; MariaDB terhubung lewat konfigurasi lingkungan tanpa secret di repo.
- [ ] A02 Migration dapat dibuat ulang pada **database uji kosong**; migrasi upgrade diuji pada salinan data sintetis dan tidak merusak histori.
- [ ] A03 Master data siswa, guru, kelas, jurusan, tahun ajaran, dan mata pelajaran tidak menggunakan data ganda antar-modul.
- [ ] A04 `siswa.id` permanen; riwayat kelas per tahun tersimpan dan tidak overlap; FK dan UNIQUE di MariaDB diuji.
- [ ] A05 Seed awal hanya role/permission dan 15 katalog modul, **tanpa** password owner default.
- [ ] A06 Semua route Core/modul **dalam rilis** tersedia dengan named routes, tanpa domain hardcoded dan tanpa mutasi GET; modul belum siap ditolak terkontrol. Arsip/bukti/progres Core tidak mewarisi middleware lisensi aktif tetapi tetap memeriksa hak objek.
- [ ] A07 Aplikasi dapat berjalan tanpa queue worker, Scheduler, Redis, atau server lain.

## 3. Gerbang B — Siklus siswa, alumni, dan keuangan (BLOCKER)

- [ ] B01 Impor 600 siswa sintetis: pratinjau, validasi, jumlah sukses/gagal, tidak ada duplikat NIS/NISN non-null.
- [ ] B02 Pindah kelas di tengah tahun mempertahankan histori kelas asal dan tanggal efektif.
- [ ] B03 Kenaikan kelas massal X→XI→XII berhasil; bisa pilih semua dari filter, pengecualian, tinggal kelas, dan preview.
- [ ] B04 Program empat tahun mendukung XII→XIII, sedangkan program tiga tahun tidak memaksakan XIII.
- [ ] B05 Tahun ajaran baru tidak otomatis menaikkan siswa; siswa belum diproses tetap terdaftar dan terhitung.
- [ ] B06 Kelulusan disetujui operator berwenang; tidak otomatis karena tahun berubah. Akun siswa yang lulus berlanjut sebagai alumni bila ada.
- [ ] B07 Alumni lama registrasi mandiri, dicegah klaim identitas orang lain, diverifikasi Pengelola Alumni, tidak perlu siswa fiktif.
- [ ] B08 Alumni mengelola riwayat kerja/kuliah/usaha yang boleh bertumpang tindih; informasi pernikahan opsional, privat.
- [ ] B09 Tagihan menyimpan snapshot periode/tarif/kelas, tidak berubah saat siswa naik/pindah/lulus.
- [ ] B10 Pembayaran cicilan, kuitansi unik, koreksi/void diaudit, transaksi bersamaan tidak menyebabkan overpayment.

## 4. Gerbang C — Modul dan integrasi (per modul yang dirilis)

| Kode modul | Fitur inti selesai | Test role & lisensi | UAT sekolah | Status |
|---|---|---|---|---|
| `pkl` | ☐ | ☐ | ☐ | Belum diuji |
| `jurnal_guru` | ☐ | ☐ | ☐ | Belum diuji |
| `absensi_izin` | ☐ | ☐ | ☐ | Belum diuji |
| `surat` | ☐ | ☐ | ☐ | Belum diuji |
| `keuangan` | ☐ | ☐ | ☐ | Belum diuji |
| `bkk` | ☐ | ☐ | ☐ | Belum diuji |
| `sarpras` | ☐ | ☐ | ☐ | Belum diuji |
| `bk` | ☐ | ☐ | ☐ | Belum diuji |
| `jadwal` | ☐ | ☐ | ☐ | Belum diuji |
| `tracer` | ☐ | ☐ | ☐ | Belum diuji |
| `hubin` | ☐ | ☐ | ☐ | Belum diuji |
| `supervisi` | ☐ | ☐ | ☐ | Belum diuji |
| `laboratorium` | ☐ | ☐ | ☐ | Belum diuji |
| `pelanggaran` | ☐ | ☐ | ☐ | Belum diuji |
| `website` | ☐ | ☐ | ☐ | Belum diuji |

**Pemeriksaan integrasi kritis:**

- [ ] C01 PKL: nilai industri dari lembar kertas diinput guru; tidak perlu akun industri; bukti file opsional.
- [ ] C02 Guru menyimpan semua siswa per sesi hari ini secara atomik; S/I langsung dari wali kelas, konfirmasi tanpa tenggat dan alur penanganan BK; izin siswa/guru dalam satu modul, tanpa presensi guru.
- [ ] C03 Surat bisa dicatat/disimpan tanpa file dan tetap bisa dibuat/upload dokumen bila dibutuhkan.
- [ ] C04 Sarpras dan Lab memakai `aset.id` sama; stok bahan pakai tidak negatif; peminjaman unit tidak ganda.
- [ ] C05 Hubin–PKL dan BKK–Tracer berbagi master data, tidak ada identitas perusahaan/alumni ganda.
- [ ] C06 CMS: menu/submenu dinamis, urutan, status tampil, URL internal/eksternal, perlindungan dari HTML/script berbahaya.
- [ ] C07 Modul nonaktif/expired menolak mutasi operasional tanpa 500; Core/modul lain tetap berjalan, arsip baca/ekspor sesuai hak, data tidak terhapus, aktif ulang aman (PRD §13.5).

## 5. Gerbang D — Keamanan aplikasi (BLOCKER)

- [ ] D01 Login, logout, reset password, rate limiting, regenerasi session, dan CSRF diuji.
- [ ] D02 MFA untuk owner `/owner` diuji; akun sekolah tidak dapat memperoleh hak owner.
- [ ] D03 otorisasi/hak akses berbasis objek diuji pada route, aksi Livewire, download, ekspor, dan batch; uji akses dengan ID objek yang bukan milik role.
- [ ] D04 Catatan BK, riwayat pelanggaran, data pernikahan alumni, dan pembayaran dilindungi sesuai hak akses.
- [ ] D05 Upload validasi isi MIME, ekstensi, ukuran; file privat tidak bisa dibaca melalui URL langsung; unduh lewat otorisasi.
- [ ] D06 Konten CMS disanitasi; XSS/SQLi/IDOR/CSRF dasar diuji dan diblokir.
- [ ] D07 Pesan error tidak menampilkan SQL, stack trace, token, kunci, atau data pribadi; audit log tidak menyimpan isi rahasia.
- [ ] D08 `composer audit` dan `npm audit` ditinjau; celah keamanan kritis/tinggi tidak dibiarkan terbuka tanpa mitigasi yang disetujui.
- [ ] D09 Mode produksi aplikasi telah dicek dalam staging: debug nonaktif; kredensial tidak disertakan dalam source dan dokumen.

## 6. Gerbang E — UI/UX dan pemrosesan massal (BLOCKER)

- [ ] E01 Loading halaman skeleton/indikator benar; submit tombol spinner, disabled double-submit, validasi di field.
- [ ] E02 Tindakan berisiko (hapus, kelulusan, tagihan, koreksi pembayaran, kenaikan kelas) memakai **modal konfirmasi**, bukan `alert/confirm` browser.
- [ ] E03 Modal dapat digunakan keyboard, fokus kembali ke tombol awal, responsif di HP, ada tombol Batal.
- [ ] E04 Bulk operation pratinjau target dan akibat sebelum mutasi; progress menampilkan total/sukses/gagal/dilewati secara akurat dari DB.
- [ ] E05 Uji setidaknya 600 siswa sintetis dengan maksimum 2–3 request paralel, 10–25 item/batch (sesuaikan kapasitas pengujian).
- [ ] E06 Koneksi putus, browser ditutup, dua tab, timeout, pause/resume, dan retry gagal tidak menyebabkan data ganda.
- [ ] E07 100% progress berarti seluruh item terminal, bukan semua sukses; daftar gagal dapat ditinjau.
- [ ] E08 Responsif desktop/mobile dan keadaan kosong/tidak ada hasil dapat dipahami nonteknis.

## 7. Gerbang F — Test, UAT, dan handover aplikasi

- [ ] F01 Automated feature/integration tests untuk alur kritis dijalankan dan hasilnya disimpan.
- [ ] F02 UAT minimal Admin Sekolah, TU, Guru/Wali, Waka, Bendahara, Guru BK, dan BKK sesuai modul yang dilepas.
- [ ] F03 Semua blocker Severity 1/2 (kebocoran data, uang salah, duplikasi, otorisasi, data hilang) sudah ditutup dan diuji regresi.
- [ ] F04 Tag rilis + commit hash + catatan perubahan tersedia.
- [ ] F05 File `.env.example` aman; panduan setup/penggunaan aplikasi pada `TASK.md`; named route map; daftar extension PHP yang dibutuhkan; urutan command build/migrate tersedia, **tanpa konfigurasi VPS/domain/Nginx/SSL**.
- [ ] F06 Prosedur rollback **aplikasi** dijelaskan dan diuji pada database sintetis; tidak menggunakan operasi destruktif di produksi.
- [ ] F07 Developer menyerahkan informasi data apa yang perlu dibackup dan cara aplikasi memeriksa hasil restore; pekerjaan backup/restore host oleh operator terpisah.
- [ ] F08 Tidak ada Queue, Scheduler, job worker, webhook pembayaran, atau integrasi pemerintah yang diam-diam menjadi syarat utama.
- [ ] F09 Sekolah menyetujui hasil UAT dan cakupan modul yang siap digunakan.

## 8. Catatan bukti pengujian

| ID cek | Bukti (nama test/commit/screenshot) | Hasil | Diuji oleh | Tanggal |
|---|---|---|---|---|
| Contoh: B03 | `tests/Feature/PromoteStudentsTest.php` — `test_batch_promotion_idempotent` | Belum dijalankan | — | — |
| | | | | |
| | | | | |

## 9. Keputusan rilis aplikasi

- [ ] **NO-GO:** terdapat blocker, tidak ada bukti test, atau fitur rilis belum berfungsi.
- [ ] **LIMITED PILOT:** hanya modul yang sudah lulus, akses dan pengguna terbatas, risiko terdokumentasi.
- [ ] **READY FOR APP HANDOVER:** semua gate app-only yang relevan lulus; tim infrastruktur boleh melanjutkan validasi platform terpisah.

**Catatan:** tombol GO-LIVE operasional/penayangan sekolah bukan keputusan developer aplikasi sendiri. Konfirmasi infrastruktur tetap ditangani pihak lain dan **tidak dimasukkan sebagai checklist kerja developer**.

**Persetujuan:**

| Peran | Nama | Tanggal | Keputusan |
|---|---|---|---|
| Lead developer | — | — | Belum ada |
| Perwakilan sekolah/UAT | — | — | Belum ada |
| Pemilik aplikasi | — | — | Belum ada |


---

## Penutupan audit alur akademik dan pembinaan

Checklist berikut melengkapi tugas F/K terkait; belum dicentang karena merupakan persyaratan implementasi, bukan bukti aplikasi sudah selesai.

- [ ] A0.01 Kelulusan hanya menampilkan data keuangan kepada pengguna berizin; uji Admin akademik tanpa nominal/indikator tunggakan dan kelulusan tetap berjalan saat Keuangan nonaktif.
- [ ] A0.02 Terapkan sumber status presensi dan klarifikasi wali; uji izin S/I overlap, pembatalan, penggantian, konflik hadir serta pemulihan status dasar/Belum Diisi tanpa auto H/A. Penyimpanan tetap atomik dan konfirmasi tanpa tenggat.
- [ ] A0.03 Implementasikan sesi susulan historis oleh wali/delegasi saja dengan daftar historis, alasan, pelaku nyata, token dan transaksi seluruh kelas; endpoint guru tetap menolak tanggal selain hari ini.
- [ ] A0.04 Siswa tertunda tahun baru tetap pada daftar Core Belum Ditempatkan; tidak otomatis masuk kelas lama/presensi. Uji penempatan efektif yang benar, pembaruan versi daftar dan susulan setelah rekonsiliasi.
- [ ] A0.05 Implementasikan rekonsiliasi dua identitas siswa maupun dua profil alumni lama tanpa siswa fiktif: pemetaan utama, preview seluruh relasi sesuai hak (tanpa nominal/BK bila tidak berizin), penyelesaian konflik oleh petugas bidang, akun utama, larangan siklus, lock/commit atomik, audit dan regresi query/constraint/laporan/otorisasi yang mencakup seluruh alias. Tidak hapus ID/histori atau menjumlah pembayaran otomatis.
- [ ] A0.06 Koreksi jurnal harian yang sudah Diperiksa membuat versi Terkirim baru; pemeriksaan lama tetap terikat versi asal. Pimpinan hanya minta perbaikan, guru pemilik mengubah kegiatan/hasil.
- [ ] A0.07 Uji PKL pindah mitra/pembimbing, kuota paralel, pembatalan sebelum mulai, penghentian saat berjalan dan revisi nilai final; histori tiap segmen tetap utuh tanpa akun industri.
- [ ] A0.08 BK menerima permintaan langsung siswa tanpa persetujuan/rujukan wali maupun rujukan presensi; uji privasi, penanggung jawab, batal/selesai/buka kembali dan laporan minimum ke wali.
- [ ] A0.09 Jadwal memakai versi efektif; publikasi bentrok ditolak, pengganti sementara/delegasi dan pembatalan kejadian tanpa kalender akademik tidak mengubah presensi atau jurnal lama.
- [ ] A0.10 Supervisi memakai versi instrumen dan hasil, penanggung jawab yang sah, alur perubahan/pembatalan/koreksi dan verifikasi tindak lanjut sebelum selesai; uji guru tidak mengubah nilai sendiri.
- [ ] A0.11 Pelanggaran menyimpan snapshot bobot/aturan/periode/kelas dan koreksi/void; uji rekap per tahun/semester tidak menghitung record batal atau versi ganda, perubahan master tidak mengubah histori.

## Catatan progres implementasi


**Status repository:** Documentation + prototype only; Laravel app belum dibuat.  
**Kesiapan produksi:** BELUM SIAP.  
**Cara mengisi:** update setelah setiap tugas selesai, sertakan bukti test dan commit/PR. Jangan mengganti status berdasarkan asumsi.

| Fase | Status | Bukti implementasi / hasil pengujian | Commit / PR |
|---|---|---|---|
| F0 — Fondasi aplikasi | Belum dimulai | — | — |
| K0–K2 — Route/DB/security | Belum dimulai | — | — |
| F1 — Master data | Belum dimulai | — | — |
| F4 — Operasi massal | Belum dimulai | — | — |
| F2 — Kenaikan/kelulusan/alumni | Belum dimulai | — | — |
| F3 — Owner dan lisensi | Belum dimulai | — | — |
| F5–F6 — 15 modul | Belum dimulai | — | — |
| F7 — QA | Belum dimulai | — | — |
| F8–F9 — Handover aplikasi | Belum dimulai | — | — |

## Format pembaruan per tugas

- **Tanggal:**
- **Task ID:**
- **Implementasi yang benar-benar selesai:**
- **File utama yang diubah:**
- **Tes dijalankan dan hasil:**
- **Hambatan/risiko:**
- **Commit atau PR:**
- **Tugas berikutnya:**

## Bukti revisi spesifikasi PRD 4.7 dan desain tema

- Dokumen memuat jurnal kegiatan/hasil, presensi hari ini seluruh kelas atomik, S/I wali langsung, konfirmasi tanpa tenggat, model penugasan, rilis R1–R4, upgrade aman dan toggle modul terkontrol.
- Lima desain publik mandiri tersedia melalui `prototype/index.html`; seluruh admin tetap satu desain.
- Validasi aktual: 147 ID task unik, 13 role, 15 modul dalam R1–R4, OP01–OP13 tercantum, tautan lokal valid dan sintaks 33 blok JavaScript pada 20 HTML lulus.
- Chromium/Playwright: kelima tema berhasil dimuat pada desktop 1440px dan mobile 390px, tidak melebar horizontal pada ukuran mobile tersebut, tombol menu membuka/menutup tanpa galat JavaScript.
- Ini bukti pemeriksaan dokumen dan prototipe, **bukan** bukti transaksi MariaDB, toggle lisensi, CMS, penugasan atau upgrade Laravel telah diimplementasikan. Seluruh task aplikasi tetap belum dicentang; uji OP dan integrasi dijalankan setelah aplikasi tersedia.

## Bukti penutupan audit spesifikasi PRD 4.8

- Semua 21 temuan audit dipetakan ke aturan, skema/relasi, kewenangan dan skenario penerimaan AU01–AU21 pada PRD §14.8. Alur pembatalan, koreksi, penggantian dan penyelesaian ditambahkan pada modul terkait; keputusan sebelumnya tetap berlaku.
- Review silang menutup konflik arsip versus lisensi, cakupan R1–R4, sumber prefill izin versus observasi guru, privasi rekonsiliasi identitas, invariant pengembalian uang dan koreksi tracer setelah periode ditutup.
- Validasi dokumen: 18 bab, 15 modul, 13 role, 172 ID task bertitik unik; OP01–OP13 dan AU01–AU21 tersedia, pemetaan modul/rilis dan tautan lokal valid. Struktur tabel konsisten dan `git diff --check` bersih.
- Perubahan pada revisi ini hanya `PRD.md` dan `TASK.md`. Tidak ada perubahan pada prototipe HTML atau implementasi Laravel; tidak ada task aplikasi yang dicentang lulus. Skenario penerimaan masih harus dieksekusi pada aplikasi nyata sesuai cakupan rilis.
