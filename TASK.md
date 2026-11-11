# TASK — Checklist Implementasi SaaS SMK Dedicated VPS

> Repository berisi spesifikasi dan desain HTML; aplikasi Laravel belum diimplementasikan.
> Dua dokumen acuan: `PRD.md` untuk kebutuhan produk dan `TASK.md` untuk pekerjaan, panduan developer, checklist rilis, serta bukti progres.
> Mulai dari F0.01–F0.03 dan K0.07, kemudian ikuti bagian Urutan pelaksanaan per lapisan. Jangan menandai tugas selesai hanya karena desain HTML tersedia.
> Gunakan checkout yang sudah ada; setiap tugas cloud sudah terisolasi dan tidak memerlukan Git worktree baru.


**Versi:** 2.0 (App-Only, selaras PRD 4.11; inventaris teknis lengkap) · **Status:** Belum dimulai (semua kotak tugas aplikasi masih kosong)
**Acuan utama:** [`PRD.md`](PRD.md)  
**Instruksi coding:** bagian Panduan implementasi dalam dokumen ini  
**Referensi UI:** [`prototype/index.html`](prototype/index.html) dan prototipe HTML lainnya.

**Navigasi teknis:** [Migration/Model](#migration-model) · [Halaman dan view](#halaman-aplikasi) · [Aksi/hak/model/test](#aksi-server) · [Route HTTP](#route-http) · [File pendukung](#file-pendukung) · [124 skenario penerimaan](#penerimaan-prd) · [Kamus database PRD](PRD.md#kamus-database).

> File ini adalah daftar kerja untuk mengubah PRD dan prototype menjadi **aplikasi Laravel sungguhan**. Menyelesaikan checklist tidak otomatis berarti siap produksi; setiap gerbang pengujian dan uji coba sekolah tetap wajib dilewati.

## Cara menggunakan checklist

1. Ikuti **Urutan pelaksanaan per lapisan** di bawah: peta route/file → kontrak skema → migration → model → keamanan/lisensi → UI bersama → Core → gerbang fondasi → fitur rilis → QA/handover. ID F/K adalah kelompok tugas, bukan urutan eksekusi seluruh kelompok.
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

**Dependensi lintas tahap:** autentikasi menunggu skema/model identitas; navigasi modul menunggu katalog, manifest dan pemeriksaan lisensi; F4 mendahului F1.07 dan aksi akademik massal. F2.14 menetapkan kontrak histori Keuangan tetapi tabel/fitur R4 tidak memblokir R1. Seluruh prasyarat dan bukti per lapisan dijelaskan pada tabel berikut. Tes K/C0/F7 berlaku untuk cakupan rilis dan semua struktur/modul yang sudah terpasang, bukan memaksa implementasi modul masa depan.

## Urutan pelaksanaan per lapisan

**Alur utama:** Peta Route/File → Kontrak Database → Seluruh Migration Fondasi Rilis → Seluruh Model Fondasi Rilis → Keamanan/Lisensi → UI Bersama → Alur Core → Gerbang Fondasi → Fitur Rilis sampai Tuntas → QA/UAT/Upgrade → Handover.

Peta mencakup Core dan seluruh 15 modul sejak awal. Yang diimplementasikan terlebih dahulu adalah fondasi **R1**, bukan seluruh fitur R2–R4. Setiap rilis berikutnya mengulang lapisan untuk tambahannya dan menguji regresi source yang sudah terpasang, termasuk modul nonaktif. F/K/C0/B0/P0/M0/A0 adalah ID pekerjaan tetap; satu kelompok dapat dikerjakan pada beberapa lapisan. **Tabel ini menggantikan urutan fase lama di seluruh panduan dalam dokumen ini.**

| Langkah | Pekerjaan dan ID acuan | Prasyarat | Hasil yang harus dapat diperiksa |
|---|---|---|---|
| 1. Scaffold dan peta route/file | F0.01–F0.03, K0.07, C0.05 | PRD dan cakupan R1–R4 | Laravel bisa boot; peta seluruh URL/named route/metode/handler/middleware/menu/rilis/file/test tercatat. Route bisnis belum dibuka jika handler belum tersedia. |
| 2. Kontrak skema | K1.08, bagian desain K1.02/K1.03/K1.06/K1.07 | Langkah 1 | Daftar tabel/kolom/FK/UNIQUE/indeks/status/retensi, pemilik master bersama dan urutan migration rilis; tidak ada FK ke tabel yang belum dikirim. |
| 3. Migration dan seed fondasi | Bagian skema F1.01–F1.03/F1.11/F1.12, F2.09, F4.01, C0.01, F0.09; skema R1 F5.03/F5.06/F5.16/F5.17 | Langkah 2 | Semua tabel fondasi Core + Presensi/Izin + Jurnal R1 terpasang berurutan; seed peran/hak/katalog aman, tanpa owner default atau aktivasi lisensi diam-diam. Ini baru penyelesaian skema, belum mencentang tugas fitur F5. |
| 4. Model dan relasi fondasi | K1.09, bagian relasi K1.07, F1.05/F1.12/F2.09 | Langkah 3 | Seluruh model fondasi rilis punya tabel, casts, relasi dan query lingkup yang diuji; tidak ada model kosong atau query ke tabel rilis mendatang. Aturan transaksi fitur dilengkapi ketika aksi fiturnya dibangun. |
| 5. Autentikasi, hak akses dan lisensi | F0.04/F0.09, F1.05/F1.06, K0.03, K2.01–K2.09, F3.03/F3.05–F3.08, C0.02–C0.04 sesuai R1 | Langkah 4 | Login/reset/logout/MFA, akun aktif, peran/tindakan/lingkup, lisensi dan arsip terotorisasi; bootstrap tidak memerlukan owner yang sudah login atau data sekolah yang belum dibuat. |
| 6. Antarmuka bersama dan routing nyata | F0.05–F0.08/F0.11, K0.01/K0.02/K0.04/K0.05/K0.06 | Langkah 5 | Layout, komponen, Bahasa Indonesia dan menu berbasis hak; daftarkan route hanya bersama handler/view nyata. `route:list`/`route:cache` dan tes tamu/role salah lulus. |
| 7. Alur Core dan panel owner | F1 CRUD/akun/penugasan; F3.01/F3.02/F3.04; F4.02–F4.12 → F1.07 → bagian R1 F2; F6 terkait Core | Langkah 6; impor dan akademik massal menunggu F4 | Data induk, tahun/kelas, impor, perpindahan/kenaikan/kelulusan, profil alumni Core, audit, pembayaran langganan owner serta proses massal bekerja sampai DB; regresi dan bukti hasil nyata. |
| 8. Gerbang fondasi | K2.10 | Langkah 1–7 dan tesnya lulus | Fondasi R1 terverifikasi; belum ada klaim modul operasional atau produksi. Blocker fondasi diperbaiki sebelum implementasi operasional modul. |
| 9. Fitur rilis sampai tuntas | R1: F5.03/F5.16/F5.17 lalu F5.06; F6, A0/M0 yang relevan. R2–R4: matriks cakupan di bawah | Gerbang fondasi rilis target; kontrak spesifik fitur ditetapkan | Aksi server, transaksi bisnis, Livewire/Blade, bukti/arsip, laporan yang diperlukan dan tes setiap fitur selesai. QA dilakukan pada setiap perubahan. |
| 10. Kelayakan produksi dan handover | F7 → F8/F9, K3, C0.06 dan checklist rilis yang berlaku | Fitur rilis lulus; lingkungan dari operator tersedia | Uji integrasi/concurrency/keamanan/upgrade/pemulihan, build/cache dan UAT nyata; keputusan rilis berdasarkan bukti dan cakupan yang dinyatakan. |

**Dependensi yang tidak boleh dibalik:** migration identitas sebelum model/auth; kontrol lisensi sebelum menu/route operasional; F4 sebelum impor/kenaikan massal; kelulusan Core sebelum portal alumni R3; tabel induk sebelum FK anak; rilis source/skema sebelum aktivasi owner. Model/fixture Keuangan F2.14 boleh diuji lebih awal bila sudah dipasang, tetapi tidak memblokir R1 dan tidak membuka UI Keuangan.

### Gerbang fondasi sebelum fitur modul

K2.10 hanya selesai bila bukti berikut ada untuk rilis target. Ini gerbang aplikasi dasar, bukan izin go-live.

| Area | Bukti minimum |
|---|---|
| Boot dan konfigurasi | Install dari lockfile, provider autentikasi `Pengguna`, koneksi MariaDB, locale Indonesia, kunci lingkungan privat, `APP_DEBUG=false` pada uji staging; tidak bergantung worker/cron. |
| Route dan menu | Peta cocok dengan `route:list`; nama/metode unik, handler/view tersedia, URL privat tertutup, fallback modul belum siap terkontrol; Core dan halaman `/` minimal berjalan tanpa modul aktif. |
| Migration/model | Instalasi kosong dan upgrade salinan sintetis lulus, FK/UNIQUE/casts/relasi/lingkup terbukti; tidak ada target FK atau pembaca model yang belum tersedia. |
| Akun dan penugasan | Provisioning owner aman, MFA wajib, kombinasi peran dan lingkup/tanggal/delegasi diuji; owner tidak memperoleh akses data sekolah otomatis. |
| Lisensi dan arsip | Kesiapan source/skema dibedakan dari aktif lisensi; nonaktif/expiry/aktif ulang tidak mengubah data atau merusak Core; arsip/bukti/progres hanya baca dan tetap berizin. |
| Core dan massal | CRUD, impor, ID siswa permanen, histori kelas dan XII/XIII, kelulusan/profil alumni Core; 600 siswa sintetis, dua browser, retry/putus koneksi dan progres dari commit DB lulus. |
| UI dan privasi | Layout dan pesan Indonesia, mobile/keyboard/modal/loading, validasi server, upload opsional privat dan unduh berizin; uji CSRF/IDOR serta audit tanpa isi sensitif. |

**Batas uji fondasi:** mekanisme lisensi/hak/arsip diuji memakai fixture sintetis dan handler uji yang hanya tersedia pada lingkungan test. Ini tidak mendaftarkan halaman bisnis dummy atau menandai modul siap pada manifest produksi. Setelah handler/transaksi fitur selesai pada langkah 9, uji kembali akses dan toggle pada endpoint fitur yang sebenarnya sebelum gerbang rilis. Dengan demikian K2.10 tidak bergantung pada fitur yang justru menunggu K2.10.

<a id="inventaris-teknis"></a>

## Peta file implementasi dan kepemilikan data

**Baseline PRD 4.11 / TASK 2.0.** Inventaris ini menetapkan 144 tabel bisnis, 150 migration bisnis (termasuk 6 penambahan FK), 141 Model, 131 halaman privat, 59 route HTTP/transport, 240 aksi layar, 18 keluaran laporan dan 33 suite integrasi. Semua adalah **rancangan, belum implementasi dan belum hasil test lulus**. Tidak ada lagi tugas memilih arsitektur, pemilik tabel, handler atau nama test untuk cakupan yang dipetakan ini.

### T1. Keputusan implementasi yang dikunci

- Autentikasi memakai Laravel Fortify, guard session `web`, provider Model `Pengguna`, fitur reset sandi dan TOTP owner; registrasi umum/profile update bawaan/passkey tidak diaktifkan. `Fortify::ignoreRoutes()` lalu hanya route terpilih T5 didaftarkan sekali. Tidak menulis kriptografi atau reset-token sendiri.
- `app/Providers/AppServiceProvider.php` adalah adapter framework minimum: binding kontrak Fortify, view autentikasi, throttle, persistent middleware Livewire dan satu `Livewire::setUpdateRoute(fn($handle,$path) => ...)` pada `/livewire/update`. Pemilihan endpoint ini disengaja; jangan mengasumsikan endpoint hash default Livewire 4 tetap. Provider/route/config tidak melakukan query DB saat boot/route-cache.
- Semua komponen Livewire adalah class-based dan privat. Endpoint update memakai web/auth/akun-aktif/header guard; setiap action yang dipetakan di T4 memeriksa lagi hak, objek, lisensi dan versi pada server. Halaman publik/registrasi tamu memakai Controller/Blade. Tidak membuat endpoint CRUD HTTP kedua untuk aksi Livewire.
- Kode izin kanonik: halaman memakai `<named-route>.lihat`, aksi memakai `<named-route>.<namaMethod>`, route HTTP privat mutatif memakai `<named-route>.jalankan`; arsip tetap `arsip.<kode_modul>.lihat/ekspor`. Grant normal hanya kepada aktor yang ditulis pada row; aksi berlabel hak khusus memerlukan grant aktif pada `hak_khusus_pengguna` dan role minimum, tidak otomatis dari peran Bendahara. Lingkup tetap diperiksa; `W`, `K` dan supervisor adalah penugasan, bukan role. Hak tambahan CLI `pkl.lokasi.retensi` adalah khusus koordinator/petugas lokasi berizin, bukan owner otomatis. Alias `alumni_bersama` dan `mitra_bersama` adalah aturan OR lisensi, tidak masuk 15 katalog modul.
- Kode peran di `peran.kode`: `owner`, `admin_sekolah`, `pimpinan`, `guru`, `tata_usaha`, `bendahara`, `pengelola_alumni`, `pengelola_sarana`, `guru_bk`, `editor_website`, `siswa`, `alumni`, `wali_siswa`. Kode aktor: `O` Owner, `AS` Admin Sekolah, `P` Pimpinan bidang/program, `G` Guru, `T` Tata Usaha, `B` Bendahara, `PA` Pengelola Alumni, `PS` Pengelola Sarana, `BK` Guru BK, `E` Editor, `S` Siswa, `AL` Alumni, `WA` Orang Tua/Wali terverifikasi. `W`=Guru dengan tugas wali, `K`=Guru/TU koordinator PKL. Kata pemilik/pegawai selalu tunduk batas role dan objek pada layar. Hak khusus koreksi/uang/publikasi tidak otomatis mengikuti hak lihat.
- Tanggal/nama file migration baseline di T2 ditetapkan, bukan placeholder timestamp. Sesudah pernah dijalankan, file itu tidak diedit/dipecah/diurut ulang; perubahan baru memakai timestamp baru sesungguhnya. File framework sesi/reset disebut di T6 dan dijalankan pada R1.
- PHPUnit mengikuti scaffold Laravel untuk feature/integration; MariaDB untuk FK/lock/concurrency, bukan SQLite. Playwright untuk browser dengan data sintetis. Tidak mewajibkan test yang sekadar mencerminkan implementasi atau satu factory untuk setiap tabel.
- XLSX memakai `phpoffice/phpspreadsheet`, PDF memakai `barryvdh/laravel-dompdf` dengan remote fetch/PHP di HTML nonaktif. Composer memilih versi stabil kompatibel Laravel13/PHP8.5, lalu mengunci lockfile; tidak menurunkan stack. CSV streaming untuk ekspor besar. Ini keputusan library, bukan instruksi memasang dependency pada revisi dokumen ini.

<a id="migration-model"></a>

### T2. File migration, Model dan relasi

Setiap row menunjuk satu file nyata yang akan dibuat. Kolom/FK/indeks persis mengikuti PRD §12.10; dependensi induk tabel di kolom terakhir harus sudah ada. Relasi setiap Model mengikuti seluruh FK dan aturan query lingkup kamus; parameter bebas tidak pernah menjadi nama class/tabel. Relasi inverse/lookup scope diuji pada `tests/Feature/Fondasi/MigrasiDanRelasiTest.php::test_instalasi_dan_upgrade_fk_urut`.

| ID / rilis | File di database/migrations/ | Tabel / Model di app/Models/ | Dependensi |
|---|---|---|---|
| M001 / R1 | `2026_11_11_000001_buat_pengguna.php` | `pengguna` / `Pengguna.php` | pengguna/provisioning untuk aktor |
| M002 / R1 | `2026_11_11_000002_buat_peran.php` | `peran` / `Peran.php` | pengguna/provisioning untuk aktor |
| M003 / R1 | `2026_11_11_000003_buat_hak_akses.php` | `hak_akses` / `HakAkses.php` | pengguna/provisioning untuk aktor |
| M004 / R1 | `2026_11_11_000004_buat_profil_sekolah.php` | `profil_sekolah` / `ProfilSekolah.php` | pengguna/provisioning untuk aktor |
| M005 / R1 | `2026_11_11_000005_buat_program_keahlian.php` | `program_keahlian` / `ProgramKeahlian.php` | pengguna/provisioning untuk aktor |
| M006 / R1 | `2026_11_11_000006_buat_tahun_ajaran.php` | `tahun_ajaran` / `TahunAjaran.php` | pengguna/provisioning untuk aktor |
| M007 / R1 | `2026_11_11_000007_buat_ruang.php` | `ruang` / `Ruang.php` | pengguna/provisioning untuk aktor |
| M008 / R1 | `2026_11_11_000008_buat_katalog_modul.php` | `katalog_modul` / `KatalogModul.php` | pengguna/provisioning untuk aktor |
| M009 / R1 | `2026_11_11_000009_buat_langganan_saas.php` | `langganan_saas` / `LanggananSaas.php` | pengguna/provisioning untuk aktor |
| M010 / R1 | `2026_11_11_000010_buat_peran_pengguna.php` | `peran_pengguna` / pivot tanpa Model | pengguna, peran |
| M011 / R1 | `2026_11_11_000011_buat_hak_akses_peran.php` | `hak_akses_peran` / pivot tanpa Model | peran, hak_akses |
| M012 / R1 | `2026_11_11_000012_buat_log_aktivitas.php` | `log_aktivitas` / `LogAktivitas.php` | pengguna |
| M013 / R1 | `2026_11_11_000013_buat_konsentrasi_keahlian.php` | `konsentrasi_keahlian` / `KonsentrasiKeahlian.php` | program_keahlian |
| M014 / R1 | `2026_11_11_000014_buat_semester.php` | `semester` / `Semester.php` | tahun_ajaran |
| M015 / R1 | `2026_11_11_000015_buat_mata_pelajaran.php` | `mata_pelajaran` / `MataPelajaran.php` | program_keahlian |
| M016 / R1 | `2026_11_11_000016_buat_guru.php` | `guru` / `Guru.php` | pengguna |
| M017 / R1 | `2026_11_11_000017_buat_pegawai.php` | `pegawai` / `Pegawai.php` | pengguna |
| M018 / R1 | `2026_11_11_000018_buat_siswa.php` | `siswa` / `Siswa.php` | pengguna, program_keahlian |
| M019 / R1 | `2026_11_11_000019_buat_wali_siswa.php` | `wali_siswa` / `WaliSiswa.php` | pengguna |
| M020 / R1 | `2026_11_11_000020_buat_penugasan_bidang.php` | `penugasan_bidang` / `PenugasanBidang.php` | pengguna |
| M021 / R1 | `2026_11_11_000021_buat_penugasan_unit.php` | `penugasan_unit` / `PenugasanUnit.php` | pengguna, ruang |
| M022 / R1 | `2026_11_11_000022_buat_lisensi_modul.php` | `lisensi_modul` / `LisensiModul.php` | katalog_modul, pengguna |
| M023 / R1 | `2026_11_11_000023_buat_operasi_massal.php` | `operasi_massal` / `OperasiMassal.php` | pengguna |
| M024 / R1 | `2026_11_11_000024_buat_berkas_privat.php` | `berkas_privat` / `BerkasPrivat.php` | pengguna |
| M025 / R1 | `2026_11_11_000025_buat_hak_khusus_pengguna.php` | `hak_khusus_pengguna` / `HakKhususPengguna.php` | pengguna, hak_akses |
| M026 / R1 | `2026_11_11_000026_buat_siswa_wali.php` | `siswa_wali` / `SiswaWali.php` | siswa, wali_siswa, pengguna |
| M027 / R1 | `2026_11_11_000027_buat_kelas.php` | `kelas` / `Kelas.php` | tahun_ajaran, program_keahlian, konsentrasi_keahlian |
| M028 / R1 | `2026_11_11_000028_buat_kepesertaan_siswa.php` | `kepesertaan_siswa` / `KepesertaanSiswa.php` | siswa, tahun_ajaran, program_keahlian, konsentrasi_keahlian |
| M029 / R1 | `2026_11_11_000029_buat_riwayat_status_siswa.php` | `riwayat_status_siswa` / `RiwayatStatusSiswa.php` | siswa |
| M030 / R1 | `2026_11_11_000030_buat_kelulusan.php` | `kelulusan` / `Kelulusan.php` | siswa, tahun_ajaran |
| M031 / R1 | `2026_11_11_000031_buat_profil_alumni.php` | `profil_alumni` / `ProfilAlumni.php` | siswa, pengguna |
| M032 / R1 | `2026_11_11_000032_buat_penggabungan_identitas_siswa.php` | `penggabungan_identitas_siswa` / `PenggabunganIdentitasSiswa.php` | siswa, pengguna |
| M033 / R1 | `2026_11_11_000033_buat_penugasan_program.php` | `penugasan_program` / `PenugasanProgram.php` | pengguna, program_keahlian, konsentrasi_keahlian |
| M034 / R1 | `2026_11_11_000034_buat_riwayat_lisensi_modul.php` | `riwayat_lisensi_modul` / `RiwayatLisensiModul.php` | lisensi_modul |
| M035 / R1 | `2026_11_11_000035_buat_pembayaran_langganan_saas.php` | `pembayaran_langganan_saas` / `PembayaranLanggananSaas.php` | langganan_saas, berkas_privat |
| M036 / R1 | `2026_11_11_000036_buat_item_operasi_massal.php` | `item_operasi_massal` / `ItemOperasiMassal.php` | operasi_massal |
| M037 / R1 | `2026_11_11_000037_buat_kelompok_kenaikan.php` | `kelompok_kenaikan` / `KelompokKenaikan.php` | operasi_massal, tahun_ajaran |
| M038 / R1 | `2026_11_11_000038_buat_konteks_pkl_siswa.php` | `konteks_pkl_siswa` / `KonteksPklSiswa.php` | siswa, pengguna |
| M039 / R1 | `2026_11_11_000039_buat_pengajuan_izin_siswa.php` | `pengajuan_izin_siswa` / `PengajuanIzinSiswa.php` | siswa, pengguna, berkas_privat |
| M040 / R1 | `2026_11_11_000040_buat_pengajuan_izin_pegawai.php` | `pengajuan_izin_pegawai` / `PengajuanIzinPegawai.php` | pengguna, berkas_privat |
| M041 / R1 | `2026_11_11_000041_buat_pengiriman_presensi.php` | `pengiriman_presensi` / `PengirimanPresensi.php` | guru, pengguna |
| M042 / R1 | `2026_11_11_000042_buat_tindak_lanjut_presensi.php` | `tindak_lanjut_presensi` / `TindakLanjutPresensi.php` | siswa, guru, pengguna |
| M043 / R1 | `2026_11_11_000043_buat_jurnal_kegiatan_guru.php` | `jurnal_kegiatan_guru` / `JurnalKegiatanGuru.php` | guru |
| M044 / R1 | `2026_11_11_000044_buat_penempatan_kelas_siswa.php` | `penempatan_kelas_siswa` / `PenempatanKelasSiswa.php` | kepesertaan_siswa, kelas |
| M045 / R1 | `2026_11_11_000045_buat_penugasan_mengajar.php` | `penugasan_mengajar` / `PenugasanMengajar.php` | guru, kelas, mata_pelajaran |
| M046 / R1 | `2026_11_11_000046_buat_penugasan_wali_kelas.php` | `penugasan_wali_kelas` / `PenugasanWaliKelas.php` | guru, kelas |
| M047 / R1 | `2026_11_11_000047_buat_penugasan_pendampingan.php` | `penugasan_pendampingan` / `PenugasanPendampingan.php` | guru, kelas, siswa |
| M048 / R1 | `2026_11_11_000048_buat_peserta_kenaikan.php` | `peserta_kenaikan` / `PesertaKenaikan.php` | kelompok_kenaikan, siswa, kepesertaan_siswa, kelas |
| M049 / R1 | `2026_11_11_000049_buat_sesi_pelajaran.php` | `sesi_pelajaran` / `SesiPelajaran.php` | kelas, mata_pelajaran, guru, pengguna |
| M050 / R1 | `2026_11_11_000050_buat_versi_jurnal_guru.php` | `versi_jurnal_guru` / `VersiJurnalGuru.php` | jurnal_kegiatan_guru, pengguna |
| M051 / R1 | `2026_11_11_000051_buat_delegasi_tugas.php` | `delegasi_tugas` / `DelegasiTugas.php` | pengguna, penugasan_mengajar, penugasan_wali_kelas, penugasan_pendampingan, penugasan_bidang, penugasan_unit, penugasan_program |
| M052 / R1 | `2026_11_11_000052_buat_pengecualian_sesi_pkl.php` | `pengecualian_sesi_pkl` / `PengecualianSesiPkl.php` | sesi_pelajaran, siswa, konteks_pkl_siswa, pengguna |
| M053 / R1 | `2026_11_11_000053_buat_kehadiran_siswa.php` | `kehadiran_siswa` / `KehadiranSiswa.php` | sesi_pelajaran, siswa, penempatan_kelas_siswa, pengajuan_izin_siswa, pengguna |
| M054 / R1 | `2026_11_11_000054_buat_sesi_pengiriman_presensi.php` | `sesi_pengiriman_presensi` / pivot tanpa Model | pengiriman_presensi, sesi_pelajaran |
| M055 / R1 | `2026_11_11_000055_buat_klarifikasi_presensi.php` | `klarifikasi_presensi` / `KlarifikasiPresensi.php` | sesi_pelajaran, siswa, pengguna |
| M056 / R1 | `2026_11_11_000056_buat_rincian_jurnal_guru.php` | `rincian_jurnal_guru` / `RincianJurnalGuru.php` | jurnal_kegiatan_guru, versi_jurnal_guru, kelas, mata_pelajaran, berkas_privat |
| M057 / R1 | `2026_11_11_000057_buat_rujukan_presensi.php` | `rujukan_presensi` / `RujukanPresensi.php` | tindak_lanjut_presensi, kehadiran_siswa |
| M058 / R2 | `2026_11_11_000058_buat_templat_surat.php` | `templat_surat` / `TemplatSurat.php` | pengguna/provisioning untuk aktor |
| M059 / R2 | `2026_11_11_000059_buat_urutan_nomor_surat.php` | `urutan_nomor_surat` / `UrutanNomorSurat.php` | pengguna/provisioning untuk aktor |
| M060 / R2 | `2026_11_11_000060_buat_kategori_artikel_website.php` | `kategori_artikel_website` / `KategoriArtikelWebsite.php` | pengguna/provisioning untuk aktor |
| M061 / R2 | `2026_11_11_000061_buat_halaman_website.php` | `halaman_website` / `HalamanWebsite.php` | pengguna/provisioning untuk aktor |
| M062 / R2 | `2026_11_11_000062_buat_pengumuman_website.php` | `pengumuman_website` / `PengumumanWebsite.php` | pengguna/provisioning untuk aktor |
| M063 / R2 | `2026_11_11_000063_buat_media_website.php` | `media_website` / `MediaWebsite.php` | pengguna |
| M064 / R2 | `2026_11_11_000064_buat_album_website.php` | `album_website` / `AlbumWebsite.php` | pengguna/provisioning untuk aktor |
| M065 / R2 | `2026_11_11_000065_buat_aturan_pelanggaran.php` | `aturan_pelanggaran` / `AturanPelanggaran.php` | pengguna/provisioning untuk aktor |
| M066 / R2 | `2026_11_11_000066_buat_versi_jadwal.php` | `versi_jadwal` / `VersiJadwal.php` | tahun_ajaran, pengguna |
| M067 / R2 | `2026_11_11_000067_buat_surat.php` | `surat` / `Surat.php` | urutan_nomor_surat, templat_surat, pengguna |
| M068 / R2 | `2026_11_11_000068_buat_artikel_website.php` | `artikel_website` / `ArtikelWebsite.php` | kategori_artikel_website |
| M069 / R2 | `2026_11_11_000069_buat_menu_website.php` | `menu_website` / `MenuWebsite.php` | halaman_website, kategori_artikel_website |
| M070 / R2 | `2026_11_11_000070_buat_item_galeri_website.php` | `item_galeri_website` / `ItemGaleriWebsite.php` | album_website, media_website |
| M071 / R2 | `2026_11_11_000071_buat_catatan_pelanggaran.php` | `catatan_pelanggaran` / `CatatanPelanggaran.php` | siswa, tahun_ajaran, semester, kelas, aturan_pelanggaran, berkas_privat |
| M072 / R2 | `2026_11_11_000072_buat_jadwal_pelajaran.php` | `jadwal_pelajaran` / `JadwalPelajaran.php` | versi_jadwal, kelas, guru, mata_pelajaran, ruang |
| M073 / R2 | `2026_11_11_000073_buat_revisi_surat.php` | `revisi_surat` / `RevisiSurat.php` | surat, berkas_privat |
| M074 / R2 | `2026_11_11_000074_buat_revisi_konten_website.php` | `revisi_konten_website` / `RevisiKontenWebsite.php` | halaman_website, artikel_website, pengumuman_website |
| M075 / R2 | `2026_11_11_000075_buat_versi_catatan_pelanggaran.php` | `versi_catatan_pelanggaran` / `VersiCatatanPelanggaran.php` | catatan_pelanggaran, pengguna |
| M076 / R2 | `2026_11_11_000076_buat_keberatan_pelanggaran.php` | `keberatan_pelanggaran` / `KeberatanPelanggaran.php` | catatan_pelanggaran, pengguna |
| M077 / R2 | `2026_11_11_000077_buat_tindak_lanjut_pelanggaran.php` | `tindak_lanjut_pelanggaran` / `TindakLanjutPelanggaran.php` | catatan_pelanggaran, pengguna |
| M078 / R2 | `2026_11_11_000078_buat_pengecualian_jadwal.php` | `pengecualian_jadwal` / `PengecualianJadwal.php` | jadwal_pelajaran, delegasi_tugas |
| M079 / R2 | `2026_11_11_000079_buat_penggunaan_ruang.php` | `penggunaan_ruang` / `PenggunaanRuang.php` | ruang, jadwal_pelajaran |
| M080 / R2 | `2026_11_11_000080_buat_pemakaian_media_website.php` | `pemakaian_media_website` / `PemakaianMediaWebsite.php` | media_website, revisi_konten_website, item_galeri_website, profil_sekolah |
| M081 / R2 | `2026_11_11_000081_buat_permintaan_konseling.php` | `permintaan_konseling` / `PermintaanKonseling.php` | siswa, pengguna, tindak_lanjut_presensi, tindak_lanjut_pelanggaran |
| M082 / R2 | `2026_11_11_000082_buat_kasus_konseling.php` | `kasus_konseling` / `KasusKonseling.php` | permintaan_konseling, siswa, pengguna |
| M083 / R2 | `2026_11_11_000083_buat_riwayat_penanggung_jawab_bk.php` | `riwayat_penanggung_jawab_bk` / `RiwayatPenanggungJawabBk.php` | kasus_konseling, pengguna |
| M084 / R2 | `2026_11_11_000084_buat_sesi_konseling.php` | `sesi_konseling` / `SesiKonseling.php` | kasus_konseling, pengguna |
| M085 / R2 | `2026_11_11_000085_buat_revisi_sesi_konseling.php` | `revisi_sesi_konseling` / `RevisiSesiKonseling.php` | sesi_konseling |
| M086 / R2 | `2026_11_11_000086_tambah_referensi_jadwal_pada_sesi_pelajaran.php` | `sesi_pelajaran` (additive) | sesi_pelajaran, jadwal_pelajaran |
| M087 / R2 | `2026_11_11_000087_tambah_logo_publik_pada_profil_sekolah.php` | `profil_sekolah` (additive) | profil_sekolah, media_website |
| M088 / R3 | `2026_11_11_000088_buat_mitra_industri.php` | `mitra_industri` / `MitraIndustri.php` | pengguna/provisioning untuk aktor |
| M089 / R3 | `2026_11_11_000089_buat_periode_pkl.php` | `periode_pkl` / `PeriodePkl.php` | tahun_ajaran, pengguna |
| M090 / R3 | `2026_11_11_000090_buat_pengajuan_registrasi_alumni.php` | `pengajuan_registrasi_alumni` / `PengajuanRegistrasiAlumni.php` | pengguna, profil_alumni |
| M091 / R3 | `2026_11_11_000091_buat_riwayat_alumni.php` | `riwayat_alumni` / `RiwayatAlumni.php` | profil_alumni |
| M092 / R3 | `2026_11_11_000092_buat_peristiwa_pribadi_alumni.php` | `peristiwa_pribadi_alumni` / `PeristiwaPribadiAlumni.php` | profil_alumni |
| M093 / R3 | `2026_11_11_000093_buat_penggabungan_identitas_alumni.php` | `penggabungan_identitas_alumni` / `PenggabunganIdentitasAlumni.php` | profil_alumni, pengguna |
| M094 / R3 | `2026_11_11_000094_buat_periode_tracer.php` | `periode_tracer` / `PeriodeTracer.php` | pengguna/provisioning untuk aktor |
| M095 / R3 | `2026_11_11_000095_buat_lokasi_pkl.php` | `lokasi_pkl` / `LokasiPkl.php` | mitra_industri |
| M096 / R3 | `2026_11_11_000096_buat_kerja_sama_industri.php` | `kerja_sama_industri` / `KerjaSamaIndustri.php` | mitra_industri, berkas_privat |
| M097 / R3 | `2026_11_11_000097_buat_lowongan_kerja.php` | `lowongan_kerja` / `LowonganKerja.php` | mitra_industri |
| M098 / R3 | `2026_11_11_000098_buat_versi_formulir_tracer.php` | `versi_formulir_tracer` / `VersiFormulirTracer.php` | periode_tracer |
| M099 / R3 | `2026_11_11_000099_buat_sasaran_tracer.php` | `sasaran_tracer` / `SasaranTracer.php` | periode_tracer, profil_alumni |
| M100 / R3 | `2026_11_11_000100_buat_revisi_sasaran_tracer.php` | `revisi_sasaran_tracer` / `RevisiSasaranTracer.php` | periode_tracer |
| M101 / R3 | `2026_11_11_000101_buat_jawaban_tracer.php` | `jawaban_tracer` / `JawabanTracer.php` | periode_tracer, profil_alumni |
| M102 / R3 | `2026_11_11_000102_buat_versi_lokasi_pkl.php` | `versi_lokasi_pkl` / `VersiLokasiPkl.php` | lokasi_pkl |
| M103 / R3 | `2026_11_11_000103_buat_kuota_pkl.php` | `kuota_pkl` / `KuotaPkl.php` | periode_pkl, lokasi_pkl |
| M104 / R3 | `2026_11_11_000104_buat_penempatan_pkl.php` | `penempatan_pkl` / `PenempatanPkl.php` | siswa, periode_pkl, lokasi_pkl, guru |
| M105 / R3 | `2026_11_11_000105_buat_adendum_kerja_sama.php` | `adendum_kerja_sama` / `AdendumKerjaSama.php` | kerja_sama_industri, berkas_privat |
| M106 / R3 | `2026_11_11_000106_buat_kegiatan_hubin.php` | `kegiatan_hubin` / `KegiatanHubin.php` | mitra_industri, kerja_sama_industri, pengguna, berkas_privat |
| M107 / R3 | `2026_11_11_000107_buat_revisi_lowongan_kerja.php` | `revisi_lowongan_kerja` / `RevisiLowonganKerja.php` | lowongan_kerja |
| M108 / R3 | `2026_11_11_000108_buat_penyerahan_kandidat.php` | `penyerahan_kandidat` / `PenyerahanKandidat.php` | lowongan_kerja |
| M109 / R3 | `2026_11_11_000109_buat_versi_jawaban_tracer.php` | `versi_jawaban_tracer` / `VersiJawabanTracer.php` | jawaban_tracer, versi_formulir_tracer, pengguna |
| M110 / R3 | `2026_11_11_000110_buat_monitoring_pkl.php` | `monitoring_pkl` / `MonitoringPkl.php` | penempatan_pkl, guru, berkas_privat |
| M111 / R3 | `2026_11_11_000111_buat_token_lokasi_pkl.php` | `token_lokasi_pkl` / `TokenLokasiPkl.php` | pengguna, penempatan_pkl, versi_lokasi_pkl |
| M112 / R3 | `2026_11_11_000112_buat_kehadiran_pkl.php` | `kehadiran_pkl` / `KehadiranPkl.php` | siswa, penempatan_pkl |
| M113 / R3 | `2026_11_11_000113_buat_jurnal_pkl.php` | `jurnal_pkl` / `JurnalPkl.php` | siswa, penempatan_pkl |
| M114 / R3 | `2026_11_11_000114_buat_penilaian_pkl.php` | `penilaian_pkl` / `PenilaianPkl.php` | penempatan_pkl |
| M115 / R3 | `2026_11_11_000115_buat_lamaran_kerja.php` | `lamaran_kerja` / `LamaranKerja.php` | lowongan_kerja, profil_alumni, siswa, revisi_lowongan_kerja, berkas_privat |
| M116 / R3 | `2026_11_11_000116_buat_peristiwa_kehadiran_pkl.php` | `peristiwa_kehadiran_pkl` / `PeristiwaKehadiranPkl.php` | kehadiran_pkl, versi_lokasi_pkl, pengguna, token_lokasi_pkl |
| M117 / R3 | `2026_11_11_000117_buat_versi_jurnal_pkl.php` | `versi_jurnal_pkl` / `VersiJurnalPkl.php` | jurnal_pkl, berkas_privat, pengguna |
| M118 / R3 | `2026_11_11_000118_buat_versi_penilaian_pkl.php` | `versi_penilaian_pkl` / `VersiPenilaianPkl.php` | penilaian_pkl, berkas_privat, pengguna |
| M119 / R3 | `2026_11_11_000119_buat_riwayat_lamaran_kerja.php` | `riwayat_lamaran_kerja` / `RiwayatLamaranKerja.php` | lamaran_kerja |
| M120 / R3 | `2026_11_11_000120_buat_rincian_penyerahan_kandidat.php` | `rincian_penyerahan_kandidat` / `RincianPenyerahanKandidat.php` | penyerahan_kandidat, lamaran_kerja, berkas_privat |
| M121 / R3 | `2026_11_11_000121_buat_permintaan_bantuan_pkl.php` | `permintaan_bantuan_pkl` / `PermintaanBantuanPkl.php` | siswa, penempatan_pkl, pengguna, peristiwa_kehadiran_pkl |
| M122 / R3 | `2026_11_11_000122_tautkan_konteks_pkl_dan_delegasi_segmen.php` | `konteks_pkl_siswa` (additive) | konteks_pkl_siswa, penempatan_pkl |
| M123 / R3 | `2026_11_11_000123_tambah_delegasi_pembimbing_pkl.php` | `delegasi_tugas` (additive) | delegasi_tugas, penempatan_pkl |
| M124 / R4 | `2026_11_11_000124_buat_jenis_biaya.php` | `jenis_biaya` / `JenisBiaya.php` | pengguna/provisioning untuk aktor |
| M125 / R4 | `2026_11_11_000125_buat_aset.php` | `aset` / `Aset.php` | ruang, pengguna |
| M126 / R4 | `2026_11_11_000126_buat_peminjaman_aset.php` | `peminjaman_aset` / `PeminjamanAset.php` | pengguna |
| M127 / R4 | `2026_11_11_000127_buat_bahan_habis_pakai.php` | `bahan_habis_pakai` / `BahanHabisPakai.php` | pengguna/provisioning untuk aktor |
| M128 / R4 | `2026_11_11_000128_buat_pemesanan_ruang_lab.php` | `pemesanan_ruang_lab` / `PemesananRuangLab.php` | ruang, kelas, guru, pengguna |
| M129 / R4 | `2026_11_11_000129_buat_instrumen_supervisi.php` | `instrumen_supervisi` / `InstrumenSupervisi.php` | pengguna/provisioning untuk aktor |
| M130 / R4 | `2026_11_11_000130_buat_tarif_biaya.php` | `tarif_biaya` / `TarifBiaya.php` | jenis_biaya, tahun_ajaran, kelas |
| M131 / R4 | `2026_11_11_000131_buat_tagihan.php` | `tagihan` / `Tagihan.php` | siswa, jenis_biaya, tahun_ajaran |
| M132 / R4 | `2026_11_11_000132_buat_rincian_peminjaman_aset.php` | `rincian_peminjaman_aset` / `RincianPeminjamanAset.php` | peminjaman_aset, aset, pengguna |
| M133 / R4 | `2026_11_11_000133_buat_perpindahan_aset.php` | `perpindahan_aset` / `PerpindahanAset.php` | aset, ruang |
| M134 / R4 | `2026_11_11_000134_buat_saldo_bahan_lokasi.php` | `saldo_bahan_lokasi` / `SaldoBahanLokasi.php` | bahan_habis_pakai, ruang |
| M135 / R4 | `2026_11_11_000135_buat_mutasi_stok_lab.php` | `mutasi_stok_lab` / `MutasiStokLab.php` | bahan_habis_pakai, ruang |
| M136 / R4 | `2026_11_11_000136_buat_versi_instrumen_supervisi.php` | `versi_instrumen_supervisi` / `VersiInstrumenSupervisi.php` | instrumen_supervisi |
| M137 / R4 | `2026_11_11_000137_buat_pembayaran.php` | `pembayaran` / `Pembayaran.php` | tagihan |
| M138 / R4 | `2026_11_11_000138_buat_penyesuaian_tagihan.php` | `penyesuaian_tagihan` / `PenyesuaianTagihan.php` | tagihan, pengguna |
| M139 / R4 | `2026_11_11_000139_buat_kejadian_aset.php` | `kejadian_aset` / `KejadianAset.php` | aset, rincian_peminjaman_aset, pengguna |
| M140 / R4 | `2026_11_11_000140_buat_penyesuaian_stok_lab.php` | `penyesuaian_stok_lab` / `PenyesuaianStokLab.php` | bahan_habis_pakai, ruang, pengguna, mutasi_stok_lab |
| M141 / R4 | `2026_11_11_000141_buat_perpindahan_bahan_lab.php` | `perpindahan_bahan_lab` / `PerpindahanBahanLab.php` | bahan_habis_pakai, ruang, mutasi_stok_lab |
| M142 / R4 | `2026_11_11_000142_buat_supervisi.php` | `supervisi` / `Supervisi.php` | guru, pengguna, versi_instrumen_supervisi |
| M143 / R4 | `2026_11_11_000143_buat_koreksi_pembayaran.php` | `koreksi_pembayaran` / `KoreksiPembayaran.php` | pembayaran, pengguna |
| M144 / R4 | `2026_11_11_000144_buat_pengembalian_pembayaran.php` | `pengembalian_pembayaran` / `PengembalianPembayaran.php` | pembayaran, pengguna |
| M145 / R4 | `2026_11_11_000145_buat_riwayat_penanggung_jawab_supervisi.php` | `riwayat_penanggung_jawab_supervisi` / `RiwayatPenanggungJawabSupervisi.php` | supervisi, pengguna |
| M146 / R4 | `2026_11_11_000146_buat_versi_hasil_supervisi.php` | `versi_hasil_supervisi` / `VersiHasilSupervisi.php` | supervisi, versi_instrumen_supervisi, pengguna |
| M147 / R4 | `2026_11_11_000147_buat_tindak_lanjut_supervisi.php` | `tindak_lanjut_supervisi` / `TindakLanjutSupervisi.php` | supervisi, pengguna |
| M148 / R4 | `2026_11_11_000148_buat_nilai_supervisi.php` | `nilai_supervisi` / `NilaiSupervisi.php` | versi_hasil_supervisi |
| M149 / R4 | `2026_11_11_000149_tambah_pemesanan_lab_pada_penggunaan_ruang.php` | `penggunaan_ruang` (additive) | penggunaan_ruang, pemesanan_ruang_lab |
| M150 / R4 | `2026_11_11_000150_tambah_delegasi_supervisi.php` | `delegasi_tugas` (additive) | delegasi_tugas, supervisi |

Model domain mewarisi Eloquent; `SiswaWali` mewarisi Pivot dengan `$incrementing=true`, metadata/verifikasi dan `using(SiswaWali::class)`; `Pengguna` mewarisi Authenticatable dan trait resmi Notifiable/Fortify. UUIDv7 dari trait resmi HasUuids, primary-key UUID hanya `OperasiMassal` (`$incrementing=false`, `$keyType=string`). `$fillable` tidak pernah menerima aktor/status final/saldo/hak dari payload umum; field server ditetapkan metode domain. Query eager-load dipakai untuk daftar/rekap agar tidak N+1.

#### T2.1 Relasi langsung Model dan aggregate

Relasi berikut adalah `belongsTo` dengan FK eksplisit sesuai kamus. Nama dari kolom tanpa `_id` dalam camelCase; FK additive hanya ada mulai rilisnya. Aktor teknis bersama `dibuatOleh -> Pengguna` dipakai hanya bila diperlukan audit, tanpa membuat seluruh inverse audit yang tidak berguna. Alias import `Model<Nama>` dipakai jika nama komponen sama dengan Model.

| Model | Relasi langsung nama -> target |
|---|---|
| `Pengguna` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `Peran` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `HakAkses` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `LogAktivitas` | `pengguna() -> Pengguna` |
| `ProfilSekolah` | `logoMedia() -> MediaWebsite` |
| `ProgramKeahlian` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `KonsentrasiKeahlian` | `programKeahlian() -> ProgramKeahlian` |
| `TahunAjaran` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `Semester` | `tahunAjaran() -> TahunAjaran` |
| `MataPelajaran` | `programKeahlian() -> ProgramKeahlian` |
| `Ruang` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `Guru` | `pengguna() -> Pengguna` |
| `Pegawai` | `pengguna() -> Pengguna` |
| `Siswa` | `pengguna() -> Pengguna`, `programKeahlian() -> ProgramKeahlian` |
| `WaliSiswa` | `pengguna() -> Pengguna` |
| `SiswaWali` | `siswa() -> Siswa`, `waliSiswa() -> WaliSiswa`, `diverifikasiOleh() -> Pengguna` |
| `Kelas` | `tahunAjaran() -> TahunAjaran`, `programKeahlian() -> ProgramKeahlian`, `konsentrasiKeahlian() -> KonsentrasiKeahlian` |
| `KepesertaanSiswa` | `siswa() -> Siswa`, `tahunAjaran() -> TahunAjaran`, `programKeahlian() -> ProgramKeahlian`, `konsentrasiKeahlian() -> KonsentrasiKeahlian` |
| `PenempatanKelasSiswa` | `kepesertaanSiswa() -> KepesertaanSiswa`, `kelas() -> Kelas` |
| `RiwayatStatusSiswa` | `siswa() -> Siswa` |
| `Kelulusan` | `siswa() -> Siswa`, `tahunAjaran() -> TahunAjaran` |
| `ProfilAlumni` | `siswa() -> Siswa`, `pengguna() -> Pengguna` |
| `PenggabunganIdentitasSiswa` | `siswaAsal() -> Siswa`, `siswaUtama() -> Siswa`, `disahkanOleh() -> Pengguna` |
| `PenugasanMengajar` | `guru() -> Guru`, `kelas() -> Kelas`, `mataPelajaran() -> MataPelajaran` |
| `PenugasanWaliKelas` | `guru() -> Guru`, `kelas() -> Kelas` |
| `PenugasanPendampingan` | `guruBk() -> Guru`, `kelas() -> Kelas`, `siswa() -> Siswa` |
| `PenugasanBidang` | `pengguna() -> Pengguna` |
| `PenugasanUnit` | `pengguna() -> Pengguna`, `ruang() -> Ruang` |
| `PenugasanProgram` | `pengguna() -> Pengguna`, `programKeahlian() -> ProgramKeahlian`, `konsentrasiKeahlian() -> KonsentrasiKeahlian` |
| `DelegasiTugas` | `penggunaAsal() -> Pengguna`, `penggunaPengganti() -> Pengguna`, `penugasanMengajar() -> PenugasanMengajar`, `penugasanWaliKelas() -> PenugasanWaliKelas`, `penugasanPendampingan() -> PenugasanPendampingan`, `penugasanBidang() -> PenugasanBidang`, `penugasanUnit() -> PenugasanUnit`, `penugasanProgram() -> PenugasanProgram`, `penempatanPkl() -> PenempatanPkl`, `supervisi() -> Supervisi` |
| `KatalogModul` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `LisensiModul` | `modul() -> KatalogModul`, `diubahOleh() -> Pengguna` |
| `RiwayatLisensiModul` | `lisensiModul() -> LisensiModul` |
| `LanggananSaas` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `PembayaranLanggananSaas` | `langgananSaas() -> LanggananSaas`, `berkasPrivat() -> BerkasPrivat` |
| `OperasiMassal` | `dibuatOleh() -> Pengguna` |
| `ItemOperasiMassal` | `operasi() -> OperasiMassal` |
| `KelompokKenaikan` | `operasi() -> OperasiMassal`, `tahunAsal() -> TahunAjaran`, `tahunTujuan() -> TahunAjaran` |
| `PesertaKenaikan` | `kelompokKenaikan() -> KelompokKenaikan`, `siswa() -> Siswa`, `kepesertaanAsal() -> KepesertaanSiswa`, `kelasTujuan() -> Kelas` |
| `BerkasPrivat` | `diunggahOleh() -> Pengguna` |
| `KonteksPklSiswa` | `siswa() -> Siswa`, `diverifikasiOleh() -> Pengguna`, `penempatanPkl() -> PenempatanPkl` |
| `PengajuanIzinSiswa` | `siswa() -> Siswa`, `dicatatOleh() -> Pengguna`, `diverifikasiOleh() -> Pengguna`, `berkasPrivat() -> BerkasPrivat` |
| `PengajuanIzinPegawai` | `pengguna() -> Pengguna`, `berkasPrivat() -> BerkasPrivat`, `diputuskanOleh() -> Pengguna` |
| `SesiPelajaran` | `kelas() -> Kelas`, `mataPelajaran() -> MataPelajaran`, `guru() -> Guru`, `dicatatOleh() -> Pengguna`, `jadwalPelajaran() -> JadwalPelajaran` |
| `PengecualianSesiPkl` | `sesiPelajaran() -> SesiPelajaran`, `siswa() -> Siswa`, `konteksPklSiswa() -> KonteksPklSiswa`, `disahkanOleh() -> Pengguna` |
| `KehadiranSiswa` | `sesiPelajaran() -> SesiPelajaran`, `siswa() -> Siswa`, `penempatanKelasSiswa() -> PenempatanKelasSiswa`, `izinSiswa() -> PengajuanIzinSiswa`, `dikonfirmasiOleh() -> Pengguna` |
| `PengirimanPresensi` | `guru() -> Guru`, `dicatatOleh() -> Pengguna` |
| `KlarifikasiPresensi` | `sesiPelajaran() -> SesiPelajaran`, `siswa() -> Siswa`, `diputuskanOleh() -> Pengguna` |
| `TindakLanjutPresensi` | `siswa() -> Siswa`, `waliKelas() -> Guru`, `guruBk() -> Guru`, `penanggungJawab() -> Pengguna`, `diselesaikanOleh() -> Pengguna` |
| `RujukanPresensi` | `tindakLanjutPresensi() -> TindakLanjutPresensi`, `kehadiranSiswa() -> KehadiranSiswa` |
| `JurnalKegiatanGuru` | `guru() -> Guru` |
| `VersiJurnalGuru` | `jurnalKegiatanGuru() -> JurnalKegiatanGuru`, `diperiksaOleh() -> Pengguna` |
| `RincianJurnalGuru` | `jurnalKegiatanGuru() -> JurnalKegiatanGuru`, `versiJurnalGuru() -> VersiJurnalGuru`, `kelas() -> Kelas`, `mataPelajaran() -> MataPelajaran`, `berkasPrivat() -> BerkasPrivat` |
| `TemplatSurat` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `UrutanNomorSurat` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `Surat` | `urutanNomorSurat() -> UrutanNomorSurat`, `templatSurat() -> TemplatSurat`, `penggantiSurat() -> Surat`, `diputuskanOleh() -> Pengguna` |
| `RevisiSurat` | `surat() -> Surat`, `berkasPrivat() -> BerkasPrivat` |
| `KategoriArtikelWebsite` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `HalamanWebsite` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `ArtikelWebsite` | `kategoriArtikelWebsite() -> KategoriArtikelWebsite` |
| `PengumumanWebsite` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `MediaWebsite` | `diunggahOleh() -> Pengguna` |
| `RevisiKontenWebsite` | `halamanWebsite() -> HalamanWebsite`, `artikelWebsite() -> ArtikelWebsite`, `pengumumanWebsite() -> PengumumanWebsite` |
| `MenuWebsite` | `induk() -> MenuWebsite`, `halamanWebsite() -> HalamanWebsite`, `kategoriArtikelWebsite() -> KategoriArtikelWebsite` |
| `AlbumWebsite` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `ItemGaleriWebsite` | `albumWebsite() -> AlbumWebsite`, `mediaWebsite() -> MediaWebsite` |
| `PemakaianMediaWebsite` | `mediaWebsite() -> MediaWebsite`, `revisiKontenWebsite() -> RevisiKontenWebsite`, `itemGaleriWebsite() -> ItemGaleriWebsite`, `profilSekolah() -> ProfilSekolah` |
| `AturanPelanggaran` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `CatatanPelanggaran` | `siswa() -> Siswa`, `tahunAjaran() -> TahunAjaran`, `semester() -> Semester`, `kelas() -> Kelas`, `aturanPelanggaran() -> AturanPelanggaran`, `berkasPrivat() -> BerkasPrivat` |
| `VersiCatatanPelanggaran` | `catatanPelanggaran() -> CatatanPelanggaran`, `diputuskanOleh() -> Pengguna` |
| `KeberatanPelanggaran` | `catatanPelanggaran() -> CatatanPelanggaran`, `diputuskanOleh() -> Pengguna` |
| `TindakLanjutPelanggaran` | `catatanPelanggaran() -> CatatanPelanggaran`, `penanggungJawab() -> Pengguna` |
| `PermintaanKonseling` | `siswa() -> Siswa`, `diajukanOleh() -> Pengguna`, `tindakLanjutPresensi() -> TindakLanjutPresensi`, `tindakLanjutPelanggaran() -> TindakLanjutPelanggaran`, `ditanganiOleh() -> Pengguna` |
| `KasusKonseling` | `permintaanKonseling() -> PermintaanKonseling`, `siswa() -> Siswa`, `penanggungJawab() -> Pengguna` |
| `RiwayatPenanggungJawabBk` | `kasusKonseling() -> KasusKonseling`, `penggunaLama() -> Pengguna`, `penggunaBaru() -> Pengguna` |
| `SesiKonseling` | `kasusKonseling() -> KasusKonseling`, `petugas() -> Pengguna` |
| `RevisiSesiKonseling` | `sesiKonseling() -> SesiKonseling` |
| `VersiJadwal` | `tahunAjaran() -> TahunAjaran`, `disahkanOleh() -> Pengguna` |
| `JadwalPelajaran` | `versiJadwal() -> VersiJadwal`, `kelas() -> Kelas`, `guru() -> Guru`, `mataPelajaran() -> MataPelajaran`, `ruang() -> Ruang` |
| `PengecualianJadwal` | `jadwalPelajaran() -> JadwalPelajaran`, `delegasiTugas() -> DelegasiTugas` |
| `PenggunaanRuang` | `ruang() -> Ruang`, `jadwalPelajaran() -> JadwalPelajaran`, `pemesananRuangLab() -> PemesananRuangLab` |
| `MitraIndustri` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `LokasiPkl` | `mitraIndustri() -> MitraIndustri` |
| `VersiLokasiPkl` | `lokasiPkl() -> LokasiPkl` |
| `PeriodePkl` | `tahunAjaran() -> TahunAjaran`, `koordinator() -> Pengguna` |
| `KuotaPkl` | `periodePkl() -> PeriodePkl`, `lokasiPkl() -> LokasiPkl` |
| `PenempatanPkl` | `siswa() -> Siswa`, `periodePkl() -> PeriodePkl`, `lokasiPkl() -> LokasiPkl`, `guruPembimbing() -> Guru`, `segmenSebelumnya() -> PenempatanPkl` |
| `MonitoringPkl` | `penempatanPkl() -> PenempatanPkl`, `guru() -> Guru`, `berkasPrivat() -> BerkasPrivat` |
| `TokenLokasiPkl` | `pengguna() -> Pengguna`, `penempatanPkl() -> PenempatanPkl`, `versiLokasiPkl() -> VersiLokasiPkl` |
| `KehadiranPkl` | `siswa() -> Siswa`, `penempatanPkl() -> PenempatanPkl` |
| `PeristiwaKehadiranPkl` | `kehadiranPkl() -> KehadiranPkl`, `versiLokasiPkl() -> VersiLokasiPkl`, `menggantikan() -> PeristiwaKehadiranPkl`, `retensiDicatatOleh() -> Pengguna`, `tokenLokasiPkl() -> TokenLokasiPkl` |
| `PermintaanBantuanPkl` | `siswa() -> Siswa`, `penempatanPkl() -> PenempatanPkl`, `diperiksaOleh() -> Pengguna`, `eventKoreksi() -> PeristiwaKehadiranPkl` |
| `JurnalPkl` | `siswa() -> Siswa`, `penempatanPkl() -> PenempatanPkl` |
| `VersiJurnalPkl` | `jurnalPkl() -> JurnalPkl`, `berkasPrivat() -> BerkasPrivat`, `diperiksaOleh() -> Pengguna` |
| `PenilaianPkl` | `penempatanPkl() -> PenempatanPkl` |
| `VersiPenilaianPkl` | `penilaianPkl() -> PenilaianPkl`, `berkasPrivat() -> BerkasPrivat`, `difinalkanOleh() -> Pengguna` |
| `KerjaSamaIndustri` | `mitraIndustri() -> MitraIndustri`, `berkasPrivat() -> BerkasPrivat`, `perpanjanganDari() -> KerjaSamaIndustri` |
| `AdendumKerjaSama` | `kerjaSamaIndustri() -> KerjaSamaIndustri`, `berkasPrivat() -> BerkasPrivat` |
| `KegiatanHubin` | `mitraIndustri() -> MitraIndustri`, `kerjaSamaIndustri() -> KerjaSamaIndustri`, `pelaksana() -> Pengguna`, `berkasPrivat() -> BerkasPrivat` |
| `PengajuanRegistrasiAlumni` | `pengguna() -> Pengguna`, `profilAlumni() -> ProfilAlumni`, `diperiksaOleh() -> Pengguna` |
| `RiwayatAlumni` | `profilAlumni() -> ProfilAlumni` |
| `PeristiwaPribadiAlumni` | `profilAlumni() -> ProfilAlumni` |
| `PenggabunganIdentitasAlumni` | `alumniAsal() -> ProfilAlumni`, `alumniUtama() -> ProfilAlumni`, `disahkanOleh() -> Pengguna` |
| `LowonganKerja` | `mitraIndustri() -> MitraIndustri` |
| `RevisiLowonganKerja` | `lowonganKerja() -> LowonganKerja` |
| `LamaranKerja` | `lowonganKerja() -> LowonganKerja`, `profilAlumni() -> ProfilAlumni`, `siswa() -> Siswa`, `revisiLowonganKerja() -> RevisiLowonganKerja`, `cvBerkas() -> BerkasPrivat` |
| `RiwayatLamaranKerja` | `lamaranKerja() -> LamaranKerja` |
| `PenyerahanKandidat` | `lowonganKerja() -> LowonganKerja` |
| `RincianPenyerahanKandidat` | `penyerahanKandidat() -> PenyerahanKandidat`, `lamaranKerja() -> LamaranKerja`, `cvBerkas() -> BerkasPrivat` |
| `PeriodeTracer` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `VersiFormulirTracer` | `periodeTracer() -> PeriodeTracer` |
| `SasaranTracer` | `periodeTracer() -> PeriodeTracer`, `profilAlumni() -> ProfilAlumni` |
| `RevisiSasaranTracer` | `periodeTracer() -> PeriodeTracer` |
| `JawabanTracer` | `periodeTracer() -> PeriodeTracer`, `profilAlumni() -> ProfilAlumni` |
| `VersiJawabanTracer` | `jawabanTracer() -> JawabanTracer`, `versiFormulirTracer() -> VersiFormulirTracer`, `disahkanOleh() -> Pengguna` |
| `JenisBiaya` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `TarifBiaya` | `jenisBiaya() -> JenisBiaya`, `tahunAjaran() -> TahunAjaran`, `kelas() -> Kelas` |
| `Tagihan` | `siswa() -> Siswa`, `jenisBiaya() -> JenisBiaya`, `tahunAjaran() -> TahunAjaran` |
| `Pembayaran` | `tagihan() -> Tagihan` |
| `KoreksiPembayaran` | `pembayaran() -> Pembayaran`, `diajukanOleh() -> Pengguna`, `disetujuiOleh() -> Pengguna` |
| `PengembalianPembayaran` | `pembayaran() -> Pembayaran`, `membalikPengembalian() -> PengembalianPembayaran`, `diajukanOleh() -> Pengguna`, `disetujuiOleh() -> Pengguna` |
| `PenyesuaianTagihan` | `tagihan() -> Tagihan`, `diajukanOleh() -> Pengguna`, `disetujuiOleh() -> Pengguna`, `membalikPenyesuaian() -> PenyesuaianTagihan` |
| `Aset` | `ruang() -> Ruang`, `penanggungJawab() -> Pengguna` |
| `PeminjamanAset` | `peminjam() -> Pengguna`, `disetujuiOleh() -> Pengguna` |
| `RincianPeminjamanAset` | `peminjamanAset() -> PeminjamanAset`, `aset() -> Aset`, `diterimaOleh() -> Pengguna` |
| `KejadianAset` | `aset() -> Aset`, `rincianPeminjamanAset() -> RincianPeminjamanAset`, `diselesaikanOleh() -> Pengguna` |
| `PerpindahanAset` | `aset() -> Aset`, `ruangAsal() -> Ruang`, `ruangTujuan() -> Ruang` |
| `BahanHabisPakai` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `SaldoBahanLokasi` | `bahanHabisPakai() -> BahanHabisPakai`, `ruang() -> Ruang` |
| `MutasiStokLab` | `bahanHabisPakai() -> BahanHabisPakai`, `ruang() -> Ruang`, `membalikMutasi() -> MutasiStokLab` |
| `PenyesuaianStokLab` | `bahanHabisPakai() -> BahanHabisPakai`, `ruang() -> Ruang`, `disetujuiOleh() -> Pengguna`, `mutasiStokLab() -> MutasiStokLab` |
| `PerpindahanBahanLab` | `bahanHabisPakai() -> BahanHabisPakai`, `ruangAsal() -> Ruang`, `ruangTujuan() -> Ruang`, `mutasiKeluar() -> MutasiStokLab`, `mutasiMasuk() -> MutasiStokLab`, `membalikTransfer() -> PerpindahanBahanLab` |
| `PemesananRuangLab` | `ruang() -> Ruang`, `kelas() -> Kelas`, `guru() -> Guru`, `diputuskanOleh() -> Pengguna` |
| `InstrumenSupervisi` | Lookup/record tanpa FK domain; casts dan aturan kamus tetap berlaku. |
| `VersiInstrumenSupervisi` | `instrumenSupervisi() -> InstrumenSupervisi` |
| `Supervisi` | `guru() -> Guru`, `supervisor() -> Pengguna`, `versiInstrumenSupervisi() -> VersiInstrumenSupervisi` |
| `RiwayatPenanggungJawabSupervisi` | `supervisi() -> Supervisi`, `penggunaLama() -> Pengguna`, `penggunaBaru() -> Pengguna` |
| `VersiHasilSupervisi` | `supervisi() -> Supervisi`, `versiInstrumenSupervisi() -> VersiInstrumenSupervisi`, `disahkanOleh() -> Pengguna` |
| `NilaiSupervisi` | `versiHasilSupervisi() -> VersiHasilSupervisi` |
| `TindakLanjutSupervisi` | `supervisi() -> Supervisi`, `diverifikasiOleh() -> Pengguna` |
| `HakKhususPengguna` | `pengguna() -> Pengguna`, `hakAkses() -> HakAkses`, `diberikanOleh() -> Pengguna` |

Inverse koleksi yang wajib karena dipakai aggregate:

| Model induk | Nama, tipe dan FK relasi aggregate |
|---|---|
| `Pengguna` | peran() belongsToMany Peran via peran_pengguna; hakKhusus() hasMany HakKhususPengguna/pengguna_id; guru()/pegawai()/siswa()/profilAlumni()/waliSiswa() hasOne masing-masing lewat pengguna_id |
| `Peran` | pengguna() belongsToMany Pengguna via peran_pengguna; hakAkses() belongsToMany HakAkses via hak_akses_peran |
| `HakAkses` | peran() belongsToMany Peran via hak_akses_peran |
| `Siswa` | kepesertaan() hasMany KepesertaanSiswa/siswa_id; kelulusan()/profilAlumni() hasOne masing-masing lewat siswa_id; wali() belongsToMany WaliSiswa via SiswaWali (pivot berperilaku/verifikasi) |
| `WaliSiswa` | siswa() belongsToMany Siswa via SiswaWali dengan Pivot terverifikasi |
| `KepesertaanSiswa` | penempatan() hasMany PenempatanKelasSiswa/kepesertaan_siswa_id |
| `Kelas` | penempatan() hasMany PenempatanKelasSiswa/kelas_id; penugasanWali() hasMany PenugasanWaliKelas/kelas_id; sesi() hasMany SesiPelajaran/kelas_id |
| `Guru` | penugasanMengajar()/penugasanWaliKelas()/jurnalHarian() hasMany masing-masing lewat guru_id |
| `OperasiMassal` | item() hasMany ItemOperasiMassal/operasi_id; kelompokKenaikan() hasOne KelompokKenaikan/operasi_id |
| `KelompokKenaikan` | peserta() hasMany PesertaKenaikan/kelompok_kenaikan_id |
| `PengirimanPresensi` | sesi() belongsToMany SesiPelajaran via sesi_pengiriman_presensi |
| `SesiPelajaran` | kehadiran() hasMany KehadiranSiswa/sesi_pelajaran_id; pengecualianPkl() hasMany PengecualianSesiPkl/sesi_pelajaran_id; pengiriman() belongsToMany PengirimanPresensi via sesi_pengiriman_presensi |
| `JurnalKegiatanGuru` | versi() hasMany VersiJurnalGuru/jurnal_kegiatan_guru_id |
| `VersiJurnalGuru` | rincian() hasMany RincianJurnalGuru/versi_jurnal_guru_id |
| `Surat` | revisi() hasMany RevisiSurat/surat_id |
| `HalamanWebsite` | versi() hasMany RevisiKontenWebsite/halaman_website_id |
| `ArtikelWebsite` | versi() hasMany RevisiKontenWebsite/artikel_website_id |
| `PengumumanWebsite` | versi() hasMany RevisiKontenWebsite/pengumuman_website_id |
| `MediaWebsite` | pemakaian() hasMany PemakaianMediaWebsite/media_website_id |
| `AlbumWebsite` | item() hasMany ItemGaleriWebsite/album_website_id |
| `CatatanPelanggaran` | versi()/keberatan()/tindakLanjut() hasMany masing-masing lewat catatan_pelanggaran_id |
| `KasusKonseling` | sesi()/riwayatPenanggungJawab() hasMany masing-masing lewat kasus_konseling_id |
| `VersiJadwal` | rincian() hasMany JadwalPelajaran/versi_jadwal_id |
| `MitraIndustri` | lokasiPkl()/kerjaSama()/lowongan() hasMany masing-masing lewat mitra_industri_id |
| `PenempatanPkl` | kehadiran()/jurnal()/monitoring() hasMany masing-masing lewat penempatan_pkl_id; penilaian() hasOne PenilaianPkl/penempatan_pkl_id |
| `KehadiranPkl` | peristiwa() hasMany PeristiwaKehadiranPkl/kehadiran_pkl_id |
| `JurnalPkl` | versi() hasMany VersiJurnalPkl/jurnal_pkl_id |
| `PenilaianPkl` | versi() hasMany VersiPenilaianPkl/penilaian_pkl_id |
| `ProfilAlumni` | riwayat()/peristiwaPribadi()/lamaran()/jawabanTracer() hasMany masing-masing lewat profil_alumni_id |
| `LowonganKerja` | revisi()/lamaran()/penyerahan() hasMany masing-masing lewat lowongan_kerja_id |
| `PenyerahanKandidat` | rincian() hasMany RincianPenyerahanKandidat/penyerahan_kandidat_id |
| `PeriodeTracer` | form()/sasaran()/jawaban() hasMany masing-masing lewat periode_tracer_id |
| `JawabanTracer` | versi() hasMany VersiJawabanTracer/jawaban_tracer_id |
| `Tagihan` | pembayaran()/penyesuaian() hasMany masing-masing lewat tagihan_id |
| `Pembayaran` | koreksi()/pengembalian() hasMany masing-masing lewat pembayaran_id |
| `Aset` | rincianPinjaman()/kejadian()/perpindahan() hasMany masing-masing lewat aset_id |
| `PeminjamanAset` | rincian() hasMany RincianPeminjamanAset/peminjaman_aset_id |
| `BahanHabisPakai` | saldo()/mutasi() hasMany masing-masing lewat bahan_habis_pakai_id |
| `InstrumenSupervisi` | versi() hasMany VersiInstrumenSupervisi/instrumen_supervisi_id |
| `Supervisi` | versiHasil()/tindakLanjut()/riwayatPenanggungJawab() hasMany masing-masing lewat supervisi_id |
| `VersiHasilSupervisi` | nilai() hasMany NilaiSupervisi/versi_hasil_supervisi_id |

<a id="halaman-aplikasi"></a>

### T3. Halaman, named route, komponen dan view

Semua row adalah `GET` dengan nama route pada kolom pertama; handler adalah `app/Livewire/<komponen>.php`, view tepat pada kolom file. `render()` hanya query/proyeksi dan state baca (filter, pagination, tab, detail, riwayat), bukan mutasi. Form tambah/ubah/detail/konfirmasi berada dalam view/komponen row itu; tidak membutuhkan file form/detail kosong lain. Layout berasal `resources/views/layouts/<layout>.blade.php`. Hak halaman `<nama>.lihat` ditambah scope row. Aksi tulis lengkap ada di T4; transport HTTP di T5.

| Nama / rilis / modul | GET URL / file route | Komponen / view / layout | Aktor dan lingkup / tabel / task |
|---|---|---|---|
| `inti.beranda` / R1 / `core` | `/admin`; `routes/inti.php` | `app/Livewire/Inti/BerandaSekolah.php`<br>`resources/views/livewire/inti/beranda-sekolah.blade.php`; admin | AS/P/G/T/B/PA/PS/BK/E menurut bidang; tanpa detail sensitif; `profil_sekolah`; F0.05 |
| `inti.profil` / R1 / `core` | `/admin/profil-sekolah`; `routes/inti.php` | `app/Livewire/Inti/ProfilSekolah.php`<br>`resources/views/livewire/inti/profil-sekolah.blade.php`; admin | AS; `profil_sekolah`; F1.04 |
| `inti.program` / R1 / `core` | `/admin/program-keahlian`; `routes/inti.php` | `app/Livewire/Inti/ProgramKeahlian.php`<br>`resources/views/livewire/inti/program-keahlian.blade.php`; admin | AS; `program_keahlian`; F1.04 |
| `inti.konsentrasi` / R1 / `core` | `/admin/konsentrasi-keahlian`; `routes/inti.php` | `app/Livewire/Inti/KonsentrasiKeahlian.php`<br>`resources/views/livewire/inti/konsentrasi-keahlian.blade.php`; admin | AS; `konsentrasi_keahlian`; F1.04 |
| `inti.mapel` / R1 / `core` | `/admin/mata-pelajaran`; `routes/inti.php` | `app/Livewire/Inti/MataPelajaran.php`<br>`resources/views/livewire/inti/mata-pelajaran.blade.php`; admin | AS; `mata_pelajaran`; F1.04 |
| `inti.ruang` / R1 / `core` | `/admin/ruang`; `routes/inti.php` | `app/Livewire/Inti/Ruang.php`<br>`resources/views/livewire/inti/ruang.blade.php`; admin | AS; `ruang`; F1.04 |
| `inti.guru` / R1 / `core` | `/admin/guru`; `routes/inti.php` | `app/Livewire/Inti/Guru.php`<br>`resources/views/livewire/inti/guru.blade.php`; admin | AS; `guru`; F1.04 |
| `inti.pegawai` / R1 / `core` | `/admin/pegawai`; `routes/inti.php` | `app/Livewire/Inti/Pegawai.php`<br>`resources/views/livewire/inti/pegawai.blade.php`; admin | AS; `pegawai`; F1.04 |
| `inti.tahun` / R1 / `core` | `/admin/tahun-ajaran`; `routes/inti.php` | `app/Livewire/Inti/TahunAjaran.php`<br>`resources/views/livewire/inti/tahun-ajaran.blade.php`; admin | AS; `tahun_ajaran`; F1.08 |
| `inti.semester` / R1 / `core` | `/admin/semester`; `routes/inti.php` | `app/Livewire/Inti/Semester.php`<br>`resources/views/livewire/inti/semester.blade.php`; admin | AS; `semester`; F1.01 |
| `inti.kelas` / R1 / `core` | `/admin/kelas`; `routes/inti.php` | `app/Livewire/Inti/Kelas.php`<br>`resources/views/livewire/inti/kelas.blade.php`; admin | AS; `kelas`; F1.04 |
| `inti.siswa` / R1 / `core` | `/admin/siswa`; `routes/inti.php` | `app/Livewire/Inti/Siswa.php`<br>`resources/views/livewire/inti/siswa.blade.php`; admin | AS; guru/wali/P hanya proyeksi siswa lingkup, tanpa hak mutasi; `siswa`; F1.04 |
| `inti.wali` / R1 / `core` | `/admin/wali-siswa`; `routes/inti.php` | `app/Livewire/Inti/WaliSiswa.php`<br>`resources/views/livewire/inti/wali-siswa.blade.php`; admin | AS; `wali_siswa`; F1.11 |
| `inti.akun` / R1 / `core` | `/admin/akun`; `routes/inti.php` | `app/Livewire/Inti/AkunPengguna.php`<br>`resources/views/livewire/inti/akun-pengguna.blade.php`; admin | AS; `pengguna`; F1.06 |
| `inti.penugasan` / R1 / `core` | `/admin/penugasan`; `routes/inti.php` | `app/Livewire/Inti/Penugasan.php`<br>`resources/views/livewire/inti/penugasan.blade.php`; admin | AS dengan hak lingkup yang boleh diberikan; `penugasan_mengajar`; F1.12 |
| `inti.audit` / R1 / `core` | `/admin/audit`; `routes/inti.php` | `app/Livewire/Inti/AuditOperasional.php`<br>`resources/views/livewire/inti/audit-operasional.blade.php`; admin | AS/Owner hanya audit lingkup administrasinya; tanpa data BK/keuangan privat; `log_aktivitas`; K2.06 |
| `inti.konteks_pkl` / R1 / `core` | `/admin/konteks-pkl`; `routes/inti.php` | `app/Livewire/Inti/KonteksPklSiswa.php`<br>`resources/views/livewire/inti/konteks-pkl-siswa.blade.php`; admin | AS/W sesuai kelas; `konteks_pkl_siswa`; M0.02 |
| `inti.proses` / R1 / `core` | `/admin/proses`; `routes/inti.php` | `app/Livewire/Inti/ProsesMassal.php`<br>`resources/views/livewire/inti/proses-massal.blade.php`; admin | AS/pembuat/inspektur berizin jenis operasi; baca status tidak recovery; `operasi_massal`; F4.07 |
| `akademik.pindah` / R1 / `core` | `/admin/pindah-kelas`; `routes/inti.php` | `app/Livewire/Akademik/PindahKelas.php`<br>`resources/views/livewire/akademik/pindah-kelas.blade.php`; admin | AS akademik; data bidang lain memerlukan hak khusus; `penempatan_kelas_siswa`; F2.13 |
| `akademik.kenaikan` / R1 / `core` | `/admin/kenaikan-kelas`; `routes/inti.php` | `app/Livewire/Akademik/KenaikanKelas.php`<br>`resources/views/livewire/akademik/kenaikan-kelas.blade.php`; admin | AS akademik; data bidang lain memerlukan hak khusus; `kelompok_kenaikan`; F2.13 |
| `akademik.kelulusan` / R1 / `core` | `/admin/kelulusan`; `routes/inti.php` | `app/Livewire/Akademik/Kelulusan.php`<br>`resources/views/livewire/akademik/kelulusan.blade.php`; admin | AS akademik; data bidang lain memerlukan hak khusus; `kelulusan`; F2.13 |
| `akademik.rekonsiliasi` / R1 / `core` | `/admin/rekonsiliasi-identitas`; `routes/inti.php` | `app/Livewire/Akademik/RekonsiliasiIdentitas.php`<br>`resources/views/livewire/akademik/rekonsiliasi-identitas.blade.php`; admin | AS akademik; data bidang lain memerlukan hak khusus; `penggabungan_identitas_siswa`; F2.13 |
| `owner.ringkasan` / R1 / `core` | `/owner`; `routes/owner.php` | `app/Livewire/Owner/Ringkasan.php`<br>`resources/views/livewire/owner/ringkasan.blade.php`; owner | O + MFA; bukan akses data sekolah; `profil_sekolah`; F3.04 |
| `owner.modul` / R1 / `core` | `/owner/modul`; `routes/owner.php` | `app/Livewire/Owner/ModulDanLisensi.php`<br>`resources/views/livewire/owner/modul-dan-lisensi.blade.php`; owner | O + MFA; bukan akses data sekolah; `lisensi_modul`; F3.04 |
| `owner.langganan` / R1 / `core` | `/owner/langganan`; `routes/owner.php` | `app/Livewire/Owner/Langganan.php`<br>`resources/views/livewire/owner/langganan.blade.php`; owner | O + MFA; bukan akses data sekolah; `langganan_saas`; F3.04 |
| `owner.pembayaran` / R1 / `core` | `/owner/pembayaran`; `routes/owner.php` | `app/Livewire/Owner/PembayaranLangganan.php`<br>`resources/views/livewire/owner/pembayaran-langganan.blade.php`; owner | O + MFA; bukan akses data sekolah; `pembayaran_langganan_saas`; F3.04 |
| `akun.profil` / R1 / `core` | `/akun/profil`; `routes/akun.php` | `app/Livewire/Akun/Profil.php`<br>`resources/views/livewire/akun/profil.blade.php`; portal | Diri sendiri semua role; `pengguna`; F1.06 |
| `akun.keamanan` / R1 / `core` | `/akun/keamanan`; `routes/akun.php` | `app/Livewire/Akun/Keamanan.php`<br>`resources/views/livewire/akun/keamanan.blade.php`; portal | Diri sendiri; O boleh setup sebelum MFA, halaman owner lain belum boleh; `pengguna`; K2.01 |
| `siswa.beranda` / R1 / `core` | `/siswa`; `routes/siswa.php` | `app/Livewire/Siswa/Beranda.php`<br>`resources/views/livewire/siswa/beranda.blade.php`; portal | S pemilik; `siswa`; F0.05 |
| `alumni.beranda` / R1 / `core` | `/alumni`; `routes/alumni.php` | `app/Livewire/Alumni/Beranda.php`<br>`resources/views/livewire/alumni/beranda.blade.php`; portal | AL terverifikasi pemilik, Core tanpa BKK/Tracer; `profil_alumni`; F0.05 |
| `wali.beranda` / R1 / `core` | `/wali`; `routes/wali.php` | `app/Livewire/Wali/Beranda.php`<br>`resources/views/livewire/wali/beranda.blade.php`; portal | WA tautan anak terverifikasi; akun opsional; `siswa_wali`; F0.05 |
| `presensi.hari_ini` / R1 / `absensi_izin` | `/admin/absensi-izin`; `routes/modul/absensi_izin.php` | `app/Livewire/Presensi/HariIni.php`<br>`resources/views/livewire/presensi/hari-ini.blade.php`; admin | G sesi sendiri/penugasan pengganti, hari server Jakarta; `sesi_pelajaran`; F5.03 |
| `presensi.izin_siswa` / R1 / `absensi_izin` | `/admin/absensi-izin/izin-siswa`; `routes/modul/absensi_izin.php` | `app/Livewire/Presensi/IzinSiswa.php`<br>`resources/views/livewire/presensi/izin-siswa.blade.php`; admin | W/T dengan delegasi wali di kelas; bukan pejabat kedua; `pengajuan_izin_siswa`; F5.16 |
| `presensi.izin_pegawai` / R1 / `absensi_izin` | `/admin/absensi-izin/izin-saya`; `routes/modul/absensi_izin.php` | `app/Livewire/Presensi/IzinPegawai.php`<br>`resources/views/livewire/presensi/izin-pegawai.blade.php`; admin | G/T pegawai pemilik; pejabat bidang memutuskan orang lain; `pengajuan_izin_pegawai`; F5.16 |
| `presensi.konfirmasi` / R1 / `absensi_izin` | `/admin/absensi-izin/kelas-binaan`; `routes/modul/absensi_izin.php` | `app/Livewire/Presensi/KonfirmasiWali.php`<br>`resources/views/livewire/presensi/konfirmasi-wali.blade.php`; admin | W kelas saat ini/delegasi historis; `kehadiran_siswa`; F7.14 |
| `presensi.susulan` / R1 / `absensi_izin` | `/admin/absensi-izin/susulan`; `routes/modul/absensi_izin.php` | `app/Livewire/Presensi/Susulan.php`<br>`resources/views/livewire/presensi/susulan.blade.php`; admin | W/delegasi khusus; guru biasa tidak boleh; `sesi_pelajaran`; A0.03 |
| `presensi.tindak_lanjut` / R1 / `absensi_izin` | `/admin/absensi-izin/tindak-lanjut`; `routes/modul/absensi_izin.php` | `app/Livewire/Presensi/TindakLanjut.php`<br>`resources/views/livewire/presensi/tindak-lanjut.blade.php`; admin | W/BK pada kelas/siswa ditugaskan; P hanya agregat; `tindak_lanjut_presensi`; F6.04 |
| `presensi.riwayat` / R1 / `absensi_izin` | `/admin/absensi-izin/riwayat`; `routes/modul/absensi_izin.php` | `app/Livewire/Presensi/Riwayat.php`<br>`resources/views/livewire/presensi/riwayat.blade.php`; admin | G sesi sendiri/W kelas/BK lingkup/P agregat; `kehadiran_siswa`; F5.03 |
| `siswa.presensi` / R1 / `absensi_izin` | `/siswa/presensi`; `routes/modul/absensi_izin.php` | `app/Livewire/Siswa/Presensi.php`<br>`resources/views/livewire/siswa/presensi.blade.php`; portal | S pemilik; `kehadiran_siswa`; F5.03 |
| `siswa.izin` / R1 / `absensi_izin` | `/siswa/izin`; `routes/modul/absensi_izin.php` | `app/Livewire/Siswa/Izin.php`<br>`resources/views/livewire/siswa/izin.blade.php`; portal | S pemilik, WA hanya pada portal anak terverifikasi; `pengajuan_izin_siswa`; F5.16 |
| `wali.presensi` / R1 / `absensi_izin` | `/wali/presensi`; `routes/modul/absensi_izin.php` | `app/Livewire/Wali/Presensi.php`<br>`resources/views/livewire/wali/presensi.blade.php`; portal | WA anak terverifikasi, tanpa isi BK privat; `kehadiran_siswa`; F1.11 |
| `wali.izin` / R1 / `absensi_izin` | `/wali/izin`; `routes/modul/absensi_izin.php` | `app/Livewire/Wali/Izin.php`<br>`resources/views/livewire/wali/izin.blade.php`; portal | WA anak terverifikasi; pengajuan opsional; `pengajuan_izin_siswa`; F5.16 |
| `jurnal.saya` / R1 / `jurnal_guru` | `/admin/jurnal-guru`; `routes/modul/jurnal_guru.php` | `app/Livewire/JurnalGuru/JurnalSaya.php`<br>`resources/views/livewire/jurnal-guru/jurnal-saya.blade.php`; admin | G pemilik; `jurnal_kegiatan_guru`; F5.06 |
| `jurnal.monitoring` / R1 / `jurnal_guru` | `/admin/jurnal-guru/monitoring`; `routes/modul/jurnal_guru.php` | `app/Livewire/JurnalGuru/Monitoring.php`<br>`resources/views/livewire/jurnal-guru/monitoring.blade.php`; admin | P kurikulum/program ditugaskan; hanya jurnal terkirim; `jurnal_kegiatan_guru`; F7.15 |
| `jurnal.riwayat` / R1 / `jurnal_guru` | `/admin/jurnal-guru/riwayat`; `routes/modul/jurnal_guru.php` | `app/Livewire/JurnalGuru/Riwayat.php`<br>`resources/views/livewire/jurnal-guru/riwayat.blade.php`; admin | G pemilik/P lingkup; draf hanya pemilik; `versi_jurnal_guru`; A0.06 |
| `surat.daftar` / R2 / `surat` | `/admin/surat`; `routes/modul/surat.php` | `app/Livewire/Surat/DaftarSurat.php`<br>`resources/views/livewire/surat/daftar-surat.blade.php`; admin | T petugas surat; P penandatangan hanya bidang ditugaskan; `surat`; F5.02 |
| `surat.persetujuan` / R2 / `surat` | `/admin/surat/persetujuan`; `routes/modul/surat.php` | `app/Livewire/Surat/Persetujuan.php`<br>`resources/views/livewire/surat/persetujuan.blade.php`; admin | P penandatangan ditugaskan; `surat`; B0.01 |
| `surat.templat` / R2 / `surat` | `/admin/surat/templat`; `routes/modul/surat.php` | `app/Livewire/Surat/Templat.php`<br>`resources/views/livewire/surat/templat.blade.php`; admin | T hak templat; `templat_surat`; F5.02 |
| `surat.arsip` / R2 / `surat` | `/admin/surat/arsip`; `routes/modul/surat.php` | `app/Livewire/Surat/ArsipSurat.php`<br>`resources/views/livewire/surat/arsip-surat.blade.php`; admin | T/P lingkup; saat nonaktif gunakan arsip Core; `surat`; F5.02 |
| `website.halaman` / R2 / `website` | `/admin/website`; `routes/modul/website.php` | `app/Livewire/Website/Halaman.php`<br>`resources/views/livewire/website/halaman.blade.php`; admin | E hak konten/publikasi; `halaman_website`; F5.01 |
| `website.artikel` / R2 / `website` | `/admin/website/artikel`; `routes/modul/website.php` | `app/Livewire/Website/Artikel.php`<br>`resources/views/livewire/website/artikel.blade.php`; admin | E hak konten/publikasi; `artikel_website`; F5.01 |
| `website.pengumuman` / R2 / `website` | `/admin/website/pengumuman`; `routes/modul/website.php` | `app/Livewire/Website/Pengumuman.php`<br>`resources/views/livewire/website/pengumuman.blade.php`; admin | E hak konten/publikasi; `pengumuman_website`; F5.01 |
| `website.kategori` / R2 / `website` | `/admin/website/kategori`; `routes/modul/website.php` | `app/Livewire/Website/Kategori.php`<br>`resources/views/livewire/website/kategori.blade.php`; admin | E; `kategori_artikel_website`; F5.01 |
| `website.menu` / R2 / `website` | `/admin/website/menu`; `routes/modul/website.php` | `app/Livewire/Website/Menu.php`<br>`resources/views/livewire/website/menu.blade.php`; admin | E hak menu; `menu_website`; F5.01 |
| `website.media` / R2 / `website` | `/admin/website/media`; `routes/modul/website.php` | `app/Livewire/Website/Media.php`<br>`resources/views/livewire/website/media.blade.php`; admin | E hak media; `media_website`; B0.08 |
| `website.galeri` / R2 / `website` | `/admin/website/galeri`; `routes/modul/website.php` | `app/Livewire/Website/Galeri.php`<br>`resources/views/livewire/website/galeri.blade.php`; admin | E; `album_website`; F5.01 |
| `website.tampilan` / R2 / `website` | `/admin/website/tampilan`; `routes/modul/website.php` | `app/Livewire/Website/Tampilan.php`<br>`resources/views/livewire/website/tampilan.blade.php`; admin | E hak tampilan; `profil_sekolah`; F5.18 |
| `pelanggaran.catatan` / R2 / `pelanggaran` | `/admin/pelanggaran`; `routes/modul/pelanggaran.php` | `app/Livewire/Pelanggaran/Pencatatan.php`<br>`resources/views/livewire/pelanggaran/pencatatan.blade.php`; admin | G/piket pada siswa lingkup, belum boleh finalisasi sendiri tanpa hak; `catatan_pelanggaran`; F5.04 |
| `pelanggaran.aturan` / R2 / `pelanggaran` | `/admin/pelanggaran/aturan`; `routes/modul/pelanggaran.php` | `app/Livewire/Pelanggaran/Aturan.php`<br>`resources/views/livewire/pelanggaran/aturan.blade.php`; admin | P kesiswaan/petugas hak aturan; `aturan_pelanggaran`; F5.04 |
| `pelanggaran.validasi` / R2 / `pelanggaran` | `/admin/pelanggaran/validasi`; `routes/modul/pelanggaran.php` | `app/Livewire/Pelanggaran/Validasi.php`<br>`resources/views/livewire/pelanggaran/validasi.blade.php`; admin | P kesiswaan/petugas validasi siswa lingkup; `versi_catatan_pelanggaran`; A0.11 |
| `pelanggaran.pembinaan` / R2 / `pelanggaran` | `/admin/pelanggaran/pembinaan`; `routes/modul/pelanggaran.php` | `app/Livewire/Pelanggaran/Pembinaan.php`<br>`resources/views/livewire/pelanggaran/pembinaan.blade.php`; admin | W/BK/P kesiswaan lingkup; `tindak_lanjut_pelanggaran`; F6.07 |
| `pelanggaran.rekap` / R2 / `pelanggaran` | `/admin/pelanggaran/rekap`; `routes/modul/pelanggaran.php` | `app/Livewire/Pelanggaran/Rekap.php`<br>`resources/views/livewire/pelanggaran/rekap.blade.php`; admin | W/BK/P lingkup/agregat; `catatan_pelanggaran`; A0.11 |
| `siswa.pelanggaran` / R2 / `pelanggaran` | `/siswa/pelanggaran`; `routes/modul/pelanggaran.php` | `app/Livewire/Siswa/Pelanggaran.php`<br>`resources/views/livewire/siswa/pelanggaran.blade.php`; portal | S pemilik hanya catatan sah; `catatan_pelanggaran`; F5.04 |
| `wali.pembinaan` / R2 / `pelanggaran` | `/wali/pembinaan`; `routes/modul/pelanggaran.php` | `app/Livewire/Wali/Pembinaan.php`<br>`resources/views/livewire/wali/pembinaan.blade.php`; portal | WA anak terverifikasi, hanya catatan sah/ringkasan diizinkan; `catatan_pelanggaran`; F1.11 |
| `bk.permintaan` / R2 / `bk` | `/admin/bk`; `routes/modul/bk.php` | `app/Livewire/Bk/Permintaan.php`<br>`resources/views/livewire/bk/permintaan.blade.php`; admin | BK ditugaskan, tanpa akses owner/AS default; `permintaan_konseling`; F5.08 |
| `bk.kasus` / R2 / `bk` | `/admin/bk/kasus`; `routes/modul/bk.php` | `app/Livewire/Bk/Kasus.php`<br>`resources/views/livewire/bk/kasus.blade.php`; admin | BK penanggung jawab/delegasi sah; `kasus_konseling`; A0.08 |
| `bk.sesi` / R2 / `bk` | `/admin/bk/sesi`; `routes/modul/bk.php` | `app/Livewire/Bk/Sesi.php`<br>`resources/views/livewire/bk/sesi.blade.php`; admin | BK penanggung jawab/delegasi; kolom privat terenkripsi; `sesi_konseling`; F5.08 |
| `bk.rekap` / R2 / `bk` | `/admin/bk/rekap`; `routes/modul/bk.php` | `app/Livewire/Bk/Rekap.php`<br>`resources/views/livewire/bk/rekap.blade.php`; admin | BK/P agregat saja; private export hak khusus; `kasus_konseling`; F5.08 |
| `siswa.bk` / R2 / `bk` | `/siswa/bk`; `routes/modul/bk.php` | `app/Livewire/Siswa/LayananBk.php`<br>`resources/views/livewire/siswa/layanan-bk.blade.php`; portal | S pemilik, tidak membutuhkan persetujuan wali; `permintaan_konseling`; A0.08 |
| `jadwal.penyusunan` / R2 / `jadwal` | `/admin/jadwal`; `routes/modul/jadwal.php` | `app/Livewire/Jadwal/Penyusunan.php`<br>`resources/views/livewire/jadwal/penyusunan.blade.php`; admin | P kurikulum/program lingkup; `versi_jadwal`; F5.07 |
| `jadwal.saya` / R2 / `jadwal` | `/admin/jadwal/saya`; `routes/modul/jadwal.php` | `app/Livewire/Jadwal/JadwalSaya.php`<br>`resources/views/livewire/jadwal/jadwal-saya.blade.php`; admin | G sendiri/W kelas/P lingkup; `jadwal_pelajaran`; F5.07 |
| `jadwal.riwayat` / R2 / `jadwal` | `/admin/jadwal/riwayat`; `routes/modul/jadwal.php` | `app/Livewire/Jadwal/Riwayat.php`<br>`resources/views/livewire/jadwal/riwayat.blade.php`; admin | P/G sesuai lingkup; `versi_jadwal`; A0.09 |
| `siswa.jadwal` / R2 / `jadwal` | `/siswa/jadwal`; `routes/modul/jadwal.php` | `app/Livewire/Siswa/Jadwal.php`<br>`resources/views/livewire/siswa/jadwal.blade.php`; portal | S kelas efektif sendiri; `jadwal_pelajaran`; F5.07 |
| `pkl.penempatan` / R3 / `pkl` | `/admin/pkl`; `routes/modul/pkl.php` | `app/Livewire/Pkl/Penempatan.php`<br>`resources/views/livewire/pkl/penempatan.blade.php`; admin | K koordinator/G pembimbing segmen; hanya K boleh menempatkan; `penempatan_pkl`; F5.05 |
| `pkl.periode` / R3 / `pkl` | `/admin/pkl/periode`; `routes/modul/pkl.php` | `app/Livewire/Pkl/Periode.php`<br>`resources/views/livewire/pkl/periode.blade.php`; admin | K; `periode_pkl`; F5.05 |
| `pkl.lokasi` / R3 / `pkl` | `/admin/pkl/lokasi`; `routes/modul/pkl.php` | `app/Livewire/Pkl/Lokasi.php`<br>`resources/views/livewire/pkl/lokasi.blade.php`; admin | K; `lokasi_pkl`; P0.01 |
| `pkl.monitoring` / R3 / `pkl` | `/admin/pkl/monitoring`; `routes/modul/pkl.php` | `app/Livewire/Pkl/Monitoring.php`<br>`resources/views/livewire/pkl/monitoring.blade.php`; admin | G pembimbing/K koordinator segmen saat ini/delegasi historis; `jurnal_pkl`; P0.06 |
| `pkl.penilaian` / R3 / `pkl` | `/admin/pkl/penilaian`; `routes/modul/pkl.php` | `app/Livewire/Pkl/Penilaian.php`<br>`resources/views/livewire/pkl/penilaian.blade.php`; admin | G pembimbing isi, K final/revisi; `penilaian_pkl`; A0.07 |
| `pkl.bantuan` / R3 / `pkl` | `/admin/pkl/bantuan`; `routes/modul/pkl.php` | `app/Livewire/Pkl/Bantuan.php`<br>`resources/views/livewire/pkl/bantuan.blade.php`; admin | G pembimbing/K dalam segmen; `permintaan_bantuan_pkl`; P0.05 |
| `pkl.laporan` / R3 / `pkl` | `/admin/pkl/laporan`; `routes/modul/pkl.php` | `app/Livewire/Pkl/Laporan.php`<br>`resources/views/livewire/pkl/laporan.blade.php`; admin | K/G lingkup; P agregat, lokasi rinci hak tersendiri; `penempatan_pkl`; P0.07 |
| `siswa.pkl` / R3 / `pkl` | `/siswa/pkl`; `routes/modul/pkl.php` | `app/Livewire/Siswa/Pkl/Beranda.php`<br>`resources/views/livewire/siswa/pkl/beranda.blade.php`; portal | S pemilik penempatan; koordinat hanya pemilik/pembimbing berhak; `penempatan_pkl`; P0.02 |
| `siswa.pkl_jurnal` / R3 / `pkl` | `/siswa/pkl/jurnal`; `routes/modul/pkl.php` | `app/Livewire/Siswa/Pkl/Jurnal.php`<br>`resources/views/livewire/siswa/pkl/jurnal.blade.php`; portal | S pemilik penempatan; koordinat hanya pemilik/pembimbing berhak; `jurnal_pkl`; P0.02 |
| `siswa.pkl_riwayat` / R3 / `pkl` | `/siswa/pkl/riwayat`; `routes/modul/pkl.php` | `app/Livewire/Siswa/Pkl/Riwayat.php`<br>`resources/views/livewire/siswa/pkl/riwayat.blade.php`; portal | S pemilik penempatan; koordinat hanya pemilik/pembimbing berhak; `kehadiran_pkl`; P0.02 |
| `hubin.mitra` / R3 / `mitra_bersama` | `/admin/mitra-industri`; `routes/inti.php` | `app/Livewire/Inti/MitraIndustri.php`<br>`resources/views/livewire/inti/mitra-industri.blade.php`; admin | K/G/T/P HUBIN/PA BKK ditugaskan; salah satu PKL/HUBIN/BKK aktif; `mitra_industri`; F6.05 |
| `hubin.kerja_sama` / R3 / `hubin` | `/admin/hubin/kerja-sama`; `routes/modul/hubin.php` | `app/Livewire/Hubin/KerjaSama.php`<br>`resources/views/livewire/hubin/kerja-sama.blade.php`; admin | G/T/P HUBIN ditugaskan; `kerja_sama_industri`; B0.06 |
| `hubin.kegiatan` / R3 / `hubin` | `/admin/hubin/kegiatan`; `routes/modul/hubin.php` | `app/Livewire/Hubin/Kegiatan.php`<br>`resources/views/livewire/hubin/kegiatan.blade.php`; admin | G/T/P HUBIN ditugaskan; `kegiatan_hubin`; F5.12 |
| `hubin.laporan` / R3 / `hubin` | `/admin/hubin/laporan`; `routes/modul/hubin.php` | `app/Livewire/Hubin/Laporan.php`<br>`resources/views/livewire/hubin/laporan.blade.php`; admin | P HUBIN/G/T sesuai hak; `kerja_sama_industri`; F5.12 |
| `alumni.verifikasi` / R3 / `alumni_bersama` | `/admin/alumni/verifikasi`; `routes/alumni.php` | `app/Livewire/Alumni/Verifikasi.php`<br>`resources/views/livewire/alumni/verifikasi.blade.php`; admin | PA, BKK atau Tracer aktif; `pengajuan_registrasi_alumni`; F2.08 |
| `alumni.rekonsiliasi` / R3 / `alumni_bersama` | `/admin/alumni/rekonsiliasi`; `routes/alumni.php` | `app/Livewire/Alumni/Rekonsiliasi.php`<br>`resources/views/livewire/alumni/rekonsiliasi.blade.php`; admin | PA berhak identitas, konflik domain oleh petugas bidang; `penggabungan_identitas_alumni`; A0.05 |
| `alumni.profil` / R3 / `alumni_bersama` | `/alumni/profil`; `routes/alumni.php` | `app/Livewire/Alumni/Profil.php`<br>`resources/views/livewire/alumni/profil.blade.php`; portal | AL terverifikasi pemilik, BKK atau Tracer; `profil_alumni`; F2.10 |
| `alumni.riwayat` / R3 / `alumni_bersama` | `/alumni/riwayat`; `routes/alumni.php` | `app/Livewire/Alumni/Riwayat.php`<br>`resources/views/livewire/alumni/riwayat.blade.php`; portal | AL pemilik, BKK atau Tracer; `riwayat_alumni`; F2.10 |
| `bkk.lowongan` / R3 / `bkk` | `/admin/bkk`; `routes/modul/bkk.php` | `app/Livewire/Bkk/Lowongan.php`<br>`resources/views/livewire/bkk/lowongan.blade.php`; admin | PA BKK; `lowongan_kerja`; F5.13 |
| `bkk.pelamar` / R3 / `bkk` | `/admin/bkk/pelamar`; `routes/modul/bkk.php` | `app/Livewire/Bkk/Pelamar.php`<br>`resources/views/livewire/bkk/pelamar.blade.php`; admin | PA BKK; `lamaran_kerja`; B0.03 |
| `bkk.penyerahan` / R3 / `bkk` | `/admin/bkk/penyerahan`; `routes/modul/bkk.php` | `app/Livewire/Bkk/Penyerahan.php`<br>`resources/views/livewire/bkk/penyerahan.blade.php`; admin | PA hak penyerahan/CV; `penyerahan_kandidat`; M0.05 |
| `bkk.laporan` / R3 / `bkk` | `/admin/bkk/laporan`; `routes/modul/bkk.php` | `app/Livewire/Bkk/Laporan.php`<br>`resources/views/livewire/bkk/laporan.blade.php`; admin | PA/P bidang HUBIN agregat; `lamaran_kerja`; F5.13 |
| `alumni.lowongan` / R3 / `bkk` | `/alumni/lowongan`; `routes/modul/bkk.php` | `app/Livewire/Alumni/Lowongan.php`<br>`resources/views/livewire/alumni/lowongan.blade.php`; portal | AL terverifikasi pemilik; siswa akhir melalui portal siswa; `lamaran_kerja`; F5.13 |
| `alumni.lamaran` / R3 / `bkk` | `/alumni/lamaran`; `routes/modul/bkk.php` | `app/Livewire/Alumni/Lamaran.php`<br>`resources/views/livewire/alumni/lamaran.blade.php`; portal | AL terverifikasi pemilik; siswa akhir melalui portal siswa; `lamaran_kerja`; F5.13 |
| `siswa.lowongan` / R3 / `bkk` | `/siswa/lowongan`; `routes/modul/bkk.php` | `app/Livewire/Siswa/Lowongan.php`<br>`resources/views/livewire/siswa/lowongan.blade.php`; portal | S tingkat akhir sah, lowongan mengizinkan; bukan alumni fiktif; `lowongan_kerja`; B0.03 |
| `tracer.periode` / R3 / `tracer` | `/admin/tracer`; `routes/modul/tracer.php` | `app/Livewire/Tracer/Periode.php`<br>`resources/views/livewire/tracer/periode.blade.php`; admin | PA Tracer; `periode_tracer`; F5.14 |
| `tracer.formulir` / R3 / `tracer` | `/admin/tracer/formulir`; `routes/modul/tracer.php` | `app/Livewire/Tracer/Formulir.php`<br>`resources/views/livewire/tracer/formulir.blade.php`; admin | PA Tracer; `versi_formulir_tracer`; B0.05 |
| `tracer.sasaran` / R3 / `tracer` | `/admin/tracer/sasaran`; `routes/modul/tracer.php` | `app/Livewire/Tracer/Sasaran.php`<br>`resources/views/livewire/tracer/sasaran.blade.php`; admin | PA Tracer; `sasaran_tracer`; M0.06 |
| `tracer.jawaban` / R3 / `tracer` | `/admin/tracer/jawaban`; `routes/modul/tracer.php` | `app/Livewire/Tracer/Jawaban.php`<br>`resources/views/livewire/tracer/jawaban.blade.php`; admin | PA Tracer sesuai hak input/koreksi; `jawaban_tracer`; B0.05 |
| `tracer.laporan` / R3 / `tracer` | `/admin/tracer/laporan`; `routes/modul/tracer.php` | `app/Livewire/Tracer/Laporan.php`<br>`resources/views/livewire/tracer/laporan.blade.php`; admin | PA/P bidang agregat; data pribadi tersembunyi; `sasaran_tracer`; M0.06 |
| `alumni.survei` / R3 / `tracer` | `/alumni/survei`; `routes/modul/tracer.php` | `app/Livewire/Alumni/Survei.php`<br>`resources/views/livewire/alumni/survei.blade.php`; portal | AL terverifikasi pemilik/sasaran; `jawaban_tracer`; F5.14 |
| `keuangan.tagihan` / R4 / `keuangan` | `/admin/keuangan`; `routes/modul/keuangan.php` | `app/Livewire/Keuangan/Tagihan.php`<br>`resources/views/livewire/keuangan/tagihan.blade.php`; admin | B berizin keuangan; snapshot siswa/alumni historis; `tagihan`; F5.09 |
| `keuangan.tarif` / R4 / `keuangan` | `/admin/keuangan/tarif`; `routes/modul/keuangan.php` | `app/Livewire/Keuangan/Tarif.php`<br>`resources/views/livewire/keuangan/tarif.blade.php`; admin | B hak tarif; `tarif_biaya`; F5.09 |
| `keuangan.pembayaran` / R4 / `keuangan` | `/admin/keuangan/pembayaran`; `routes/modul/keuangan.php` | `app/Livewire/Keuangan/Pembayaran.php`<br>`resources/views/livewire/keuangan/pembayaran.blade.php`; admin | B pencatat penerimaan; `pembayaran`; F5.09 |
| `keuangan.koreksi` / R4 / `keuangan` | `/admin/keuangan/koreksi`; `routes/modul/keuangan.php` | `app/Livewire/Keuangan/Koreksi.php`<br>`resources/views/livewire/keuangan/koreksi.blade.php`; admin | B hak ajukan/koreksi dipisah; `koreksi_pembayaran`; B0.02 |
| `keuangan.pengembalian` / R4 / `keuangan` | `/admin/keuangan/pengembalian`; `routes/modul/keuangan.php` | `app/Livewire/Keuangan/Pengembalian.php`<br>`resources/views/livewire/keuangan/pengembalian.blade.php`; admin | B hak pengajuan/pengesahan pengembalian; `pengembalian_pembayaran`; B0.02 |
| `keuangan.laporan` / R4 / `keuangan` | `/admin/keuangan/laporan`; `routes/modul/keuangan.php` | `app/Livewire/Keuangan/Laporan.php`<br>`resources/views/livewire/keuangan/laporan.blade.php`; admin | B/P dengan hak agregat atau detail sesuai bidang; `tagihan`; F5.09 |
| `siswa.tagihan` / R4 / `keuangan` | `/siswa/tagihan`; `routes/modul/keuangan.php` | `app/Livewire/Siswa/Tagihan.php`<br>`resources/views/livewire/siswa/tagihan.blade.php`; portal | S pemilik; `tagihan`; F6.01 |
| `alumni.tagihan` / R4 / `keuangan` | `/alumni/tagihan`; `routes/modul/keuangan.php` | `app/Livewire/Alumni/Tagihan.php`<br>`resources/views/livewire/alumni/tagihan.blade.php`; portal | AL pemilik terkait siswa, tanpa tagihan fiktif alumni lama; `tagihan`; F6.01 |
| `wali.tagihan` / R4 / `keuangan` | `/wali/tagihan`; `routes/modul/keuangan.php` | `app/Livewire/Wali/Tagihan.php`<br>`resources/views/livewire/wali/tagihan.blade.php`; portal | WA anak terverifikasi, hak keuangan diizinkan; `tagihan`; F6.01 |
| `sarpras.aset` / R4 / `sarpras` | `/admin/sarpras`; `routes/modul/sarpras.php` | `app/Livewire/Sarpras/Aset.php`<br>`resources/views/livewire/sarpras/aset.blade.php`; admin | PS unit ditugaskan; `aset`; F5.10 |
| `sarpras.peminjaman` / R4 / `sarpras` | `/admin/sarpras/peminjaman`; `routes/modul/sarpras.php` | `app/Livewire/Sarpras/Peminjaman.php`<br>`resources/views/livewire/sarpras/peminjaman.blade.php`; admin | G/pegawai pemohon; PS serah-terima unit lingkup; `peminjaman_aset`; B0.04 |
| `sarpras.pengembalian` / R4 / `sarpras` | `/admin/sarpras/pengembalian`; `routes/modul/sarpras.php` | `app/Livewire/Sarpras/Pengembalian.php`<br>`resources/views/livewire/sarpras/pengembalian.blade.php`; admin | PS unit; `rincian_peminjaman_aset`; B0.04 |
| `sarpras.kondisi` / R4 / `sarpras` | `/admin/sarpras/kondisi`; `routes/modul/sarpras.php` | `app/Livewire/Sarpras/Kondisi.php`<br>`resources/views/livewire/sarpras/kondisi.blade.php`; admin | PS unit; hak penghapusan tersendiri; `kejadian_aset`; B0.04 |
| `sarpras.laporan` / R4 / `sarpras` | `/admin/sarpras/laporan`; `routes/modul/sarpras.php` | `app/Livewire/Sarpras/Laporan.php`<br>`resources/views/livewire/sarpras/laporan.blade.php`; admin | PS/P sarana lingkup; `aset`; F5.10 |
| `laboratorium.pemesanan` / R4 / `laboratorium` | `/admin/laboratorium`; `routes/modul/laboratorium.php` | `app/Livewire/Laboratorium/Pemesanan.php`<br>`resources/views/livewire/laboratorium/pemesanan.blade.php`; admin | G pemohon; PS unit menyetujui; `pemesanan_ruang_lab`; F5.11 |
| `laboratorium.peralatan` / R4 / `laboratorium` | `/admin/laboratorium/peralatan`; `routes/modul/laboratorium.php` | `app/Livewire/Laboratorium/Peralatan.php`<br>`resources/views/livewire/laboratorium/peralatan.blade.php`; admin | PS unit/G peminjam; Lab aktif cukup tanpa Sarpras; `aset`; F6.03 |
| `laboratorium.bahan` / R4 / `laboratorium` | `/admin/laboratorium/bahan`; `routes/modul/laboratorium.php` | `app/Livewire/Laboratorium/Bahan.php`<br>`resources/views/livewire/laboratorium/bahan.blade.php`; admin | PS unit; `bahan_habis_pakai`; F5.11 |
| `laboratorium.mutasi` / R4 / `laboratorium` | `/admin/laboratorium/mutasi`; `routes/modul/laboratorium.php` | `app/Livewire/Laboratorium/MutasiStok.php`<br>`resources/views/livewire/laboratorium/mutasi-stok.blade.php`; admin | PS unit, pengesahan penyesuaian hak khusus; `mutasi_stok_lab`; B0.07 |
| `laboratorium.transfer` / R4 / `laboratorium` | `/admin/laboratorium/transfer`; `routes/modul/laboratorium.php` | `app/Livewire/Laboratorium/Transfer.php`<br>`resources/views/livewire/laboratorium/transfer.blade.php`; admin | PS berhak kedua lokasi; `perpindahan_bahan_lab`; M0.04 |
| `laboratorium.laporan` / R4 / `laboratorium` | `/admin/laboratorium/laporan`; `routes/modul/laboratorium.php` | `app/Livewire/Laboratorium/Laporan.php`<br>`resources/views/livewire/laboratorium/laporan.blade.php`; admin | PS/P sarana lingkup; `saldo_bahan_lokasi`; F5.11 |
| `supervisi.penugasan` / R4 / `supervisi` | `/admin/supervisi`; `routes/modul/supervisi.php` | `app/Livewire/Supervisi/Penugasan.php`<br>`resources/views/livewire/supervisi/penugasan.blade.php`; admin | P kurikulum berizin, G supervisor ditugaskan; `supervisi`; F5.15 |
| `supervisi.instrumen` / R4 / `supervisi` | `/admin/supervisi/instrumen`; `routes/modul/supervisi.php` | `app/Livewire/Supervisi/Instrumen.php`<br>`resources/views/livewire/supervisi/instrumen.blade.php`; admin | P kurikulum hak instrumen; `instrumen_supervisi`; A0.10 |
| `supervisi.observasi` / R4 / `supervisi` | `/admin/supervisi/observasi`; `routes/modul/supervisi.php` | `app/Livewire/Supervisi/Observasi.php`<br>`resources/views/livewire/supervisi/observasi.blade.php`; admin | G supervisor/P berizin pada guru ditugaskan; `versi_hasil_supervisi`; A0.10 |
| `supervisi.tindak_lanjut` / R4 / `supervisi` | `/admin/supervisi/tindak-lanjut`; `routes/modul/supervisi.php` | `app/Livewire/Supervisi/TindakLanjut.php`<br>`resources/views/livewire/supervisi/tindak-lanjut.blade.php`; admin | G guru pemilik isi hasil/G supervisor verifikasi; `tindak_lanjut_supervisi`; A0.10 |
| `supervisi.hasil` / R4 / `supervisi` | `/admin/supervisi/hasil`; `routes/modul/supervisi.php` | `app/Livewire/Supervisi/Hasil.php`<br>`resources/views/livewire/supervisi/hasil.blade.php`; admin | G guru pemilik hanya hasil sah/G supervisor/P lingkup; `versi_hasil_supervisi`; F5.15 |
| `inti.impor` / R1 / `core` | `/admin/impor`; `routes/inti.php` | `app/Livewire/Inti/ImporData.php`<br>`resources/views/livewire/inti/impor-data.blade.php`; admin | AS; impor alumni R3 PA dengan BKK atau Tracer; `operasi_massal`; F1.07 |
| `inti.laporan` / R1 / `core` | `/admin/laporan`; `routes/inti.php` | `app/Livewire/Inti/Laporan.php`<br>`resources/views/livewire/inti/laporan.blade.php`; admin | AS/P bidang agregat; tidak menerima kolom uang/BK tanpa hak; `siswa`; F0.05 |

<a id="aksi-server"></a>

### T4. Aksi server setiap layar dan pengujiannya

Transport default: POST `livewire.update`, method publik komponen T3 pada kolom aksi. Form hanya mengirim field yang ditulis; `?` opsional. Nama input yang lebih ringkas (`mapel_id`, `mulai`, `berkas_id`) dinormalisasi ke kolom kamus oleh method domain, tidak pernah diteruskan sebagai mass assignment. `id`, `versi_data` dan objek anak selalu divalidasi ulang. Aksi berlabel HTTP memanggil route T5 dari UI dan tidak membuat method Livewire mutasi duplikat. Owner domain adalah satu implementasi kanonik, boleh dipakai komponen yang berbeda (contoh pinjam Lab/Sarpras) dengan kode modul penulis ditentukan server.

| Layar / aksi / izin | Model::metode / field yang diterima | Aktor, hasil dan kegagalan penting | File test dan metode wajib |
|---|---|---|---|
| `inti.profil::simpan`<br>`inti.profil.simpan` | `ProfilSekolah::simpanProfil`<br>`nama,npsn,domain,alamat,kontak` | AS. Update record id=1, audit; domain hanya metadata | `tests/Feature/Antarmuka/Inti/ProfilSekolahTest.php`<br>`test_simpan_profil_tunggal_tidak_mengubah_host` |
| `inti.program::simpan`<br>`inti.program.simpan` | `ProgramKeahlian::simpanMaster`<br>`kode,nama,durasi_tahun,id?,versi_data?` | AS. Create/update tervalidasi; kode/FK unik, audit | `tests/Feature/Antarmuka/Inti/ProgramKeahlianTest.php`<br>`test_simpan_durasi_hanya_3_atau_4` |
| `inti.program::nonaktifkan`<br>`inti.program.nonaktifkan` | `ProgramKeahlian::nonaktifkan`<br>`id,versi_data,alasan` | AS. Arsipkan master, tidak cascade histori | `tests/Feature/Antarmuka/Inti/ProgramKeahlianTest.php`<br>`test_nonaktifkan_master_terpakai_tidak_dihapus` |
| `inti.konsentrasi::simpan`<br>`inti.konsentrasi.simpan` | `KonsentrasiKeahlian::simpanMaster`<br>`program_keahlian_id,kode,nama,id?,versi_data?` | AS. Create/update tervalidasi; kode/FK unik, audit | `tests/Feature/Antarmuka/Inti/KonsentrasiKeahlianTest.php`<br>`test_simpan_induk_program_cocok` |
| `inti.konsentrasi::nonaktifkan`<br>`inti.konsentrasi.nonaktifkan` | `KonsentrasiKeahlian::nonaktifkan`<br>`id,versi_data,alasan` | AS. Arsipkan master, tidak cascade histori | `tests/Feature/Antarmuka/Inti/KonsentrasiKeahlianTest.php`<br>`test_nonaktifkan_master_terpakai_tidak_dihapus` |
| `inti.mapel::simpan`<br>`inti.mapel.simpan` | `MataPelajaran::simpanMaster`<br>`kode,nama,program_keahlian_id?,id?,versi_data?` | AS. Create/update tervalidasi; kode/FK unik, audit | `tests/Feature/Antarmuka/Inti/MataPelajaranTest.php`<br>`test_simpan_kode_unik` |
| `inti.mapel::nonaktifkan`<br>`inti.mapel.nonaktifkan` | `MataPelajaran::nonaktifkan`<br>`id,versi_data,alasan` | AS. Arsipkan master, tidak cascade histori | `tests/Feature/Antarmuka/Inti/MataPelajaranTest.php`<br>`test_nonaktifkan_master_terpakai_tidak_dihapus` |
| `inti.ruang::simpan`<br>`inti.ruang.simpan` | `Ruang::simpanMaster`<br>`kode,nama,jenis,id?,versi_data?` | AS. Create/update tervalidasi; kode/FK unik, audit | `tests/Feature/Antarmuka/Inti/RuangTest.php`<br>`test_simpan_ruang_terpakai_tetap_ada` |
| `inti.ruang::nonaktifkan`<br>`inti.ruang.nonaktifkan` | `Ruang::nonaktifkan`<br>`id,versi_data,alasan` | AS. Arsipkan master, tidak cascade histori | `tests/Feature/Antarmuka/Inti/RuangTest.php`<br>`test_nonaktifkan_master_terpakai_tidak_dihapus` |
| `inti.guru::simpan`<br>`inti.guru.simpan` | `Guru::simpanMaster`<br>`nomor_induk?,nama,kontak?,pengguna_id?,id?,versi_data?` | AS. Create/update tervalidasi; kode/FK unik, audit | `tests/Feature/Antarmuka/Inti/GuruTest.php`<br>`test_simpan_akun_guru_tidak_ganda` |
| `inti.guru::nonaktifkan`<br>`inti.guru.nonaktifkan` | `Guru::nonaktifkan`<br>`id,versi_data,alasan` | AS. Arsipkan master, tidak cascade histori | `tests/Feature/Antarmuka/Inti/GuruTest.php`<br>`test_nonaktifkan_master_terpakai_tidak_dihapus` |
| `inti.pegawai::simpan`<br>`inti.pegawai.simpan` | `Pegawai::simpanMaster`<br>`nomor_induk?,nama,jabatan?,kontak?,pengguna_id?,id?,versi_data?` | AS. Create/update tervalidasi; kode/FK unik, audit | `tests/Feature/Antarmuka/Inti/PegawaiTest.php`<br>`test_simpan_akun_pegawai_tidak_ganda` |
| `inti.pegawai::nonaktifkan`<br>`inti.pegawai.nonaktifkan` | `Pegawai::nonaktifkan`<br>`id,versi_data,alasan` | AS. Arsipkan master, tidak cascade histori | `tests/Feature/Antarmuka/Inti/PegawaiTest.php`<br>`test_nonaktifkan_master_terpakai_tidak_dihapus` |
| `inti.tahun::simpan`<br>`inti.tahun.simpan` | `TahunAjaran::simpanDraf`<br>`id?,nama,tanggal_mulai,tanggal_selesai,versi_data?` | AS. Draf tahun, tidak otomatis menaikkan siswa | `tests/Feature/Antarmuka/Inti/TahunAjaranTest.php`<br>`test_simpan_tahun_baru_tidak_menaikkan_siswa` |
| `inti.tahun::aktifkan`<br>`inti.tahun.aktifkan` | `TahunAjaran::aktifkan`<br>`id,versi_data,konfirmasi` | AS. Lock profil, satu aktif, siswa belum ditempatkan tetap terlihat | `tests/Feature/Antarmuka/Inti/TahunAjaranTest.php`<br>`test_aktifkan_dua_aktivasi_tidak_dua_tahun_aktif` |
| `inti.semester::simpan`<br>`inti.semester.simpan` | `Semester::simpanDraf`<br>`id?,tahun_ajaran_id,nomor,tanggal_mulai,tanggal_selesai` | AS. Dalam rentang tahun, nomor 1/2 | `tests/Feature/Antarmuka/Inti/SemesterTest.php`<br>`test_simpan_semester_di_luar_tahun_ditolak` |
| `inti.semester::aktifkan`<br>`inti.semester.aktifkan` | `Semester::aktifkan`<br>`id,versi_data` | AS. Lock tahun, satu semester aktif | `tests/Feature/Antarmuka/Inti/SemesterTest.php`<br>`test_aktifkan_semester_aktif_tunggal` |
| `inti.kelas::simpan`<br>`inti.kelas.simpan` | `Kelas::simpanMaster`<br>`id?,tahun_ajaran_id,program_keahlian_id,konsentrasi_keahlian_id?,tingkat,nama,versi_data?` | AS. Validasi konsentrasi/tingkat; tahun kelas historis tidak diganti sesudah dipakai | `tests/Feature/Antarmuka/Inti/KelasTest.php`<br>`test_simpan_xiii_hanya_program_empat_tahun` |
| `inti.kelas::nonaktifkan`<br>`inti.kelas.nonaktifkan` | `Kelas::nonaktifkan`<br>`id,versi_data,alasan` | AS. Tidak menghapus penempatan | `tests/Feature/Antarmuka/Inti/KelasTest.php`<br>`test_nonaktifkan_kelas_historis_tetap_terbaca` |
| `inti.siswa::simpan`<br>`inti.siswa.simpan` | `Siswa::simpanIdentitas`<br>`id?,nis,nisn?,nama,tanggal_lahir?,kontak?,program_keahlian_id?,versi_data?` | AS. ID tetap, validasi akun/NIS/NISN; penempatan melalui aksi akademik | `tests/Feature/Antarmuka/Inti/SiswaTest.php`<br>`test_simpan_nisn_null_boleh_berulang_nis_tidak` |
| `inti.siswa::catatMutasi`<br>`inti.siswa.catatMutasi` | `Siswa::catatPerubahanStatus`<br>`siswa_id,jenis_peristiwa,tanggal_efektif,alasan,versi_data` | AS. Lock siswa/enrolmen; riwayat event; kembali memakai identitas sama | `tests/Feature/Antarmuka/Inti/SiswaTest.php`<br>`test_catat_mutasi_mutasi_kembali_tidak_siswa_baru` |
| `inti.wali::simpan`<br>`inti.wali.simpan` | `WaliSiswa::simpanIdentitas`<br>`id?,nama,kontak?,pengguna_id?,versi_data?` | AS. Identitas wali tanpa wajib akun | `tests/Feature/Antarmuka/Inti/WaliSiswaTest.php`<br>`test_simpan_kontak_sama_bukan_akses_anak` |
| `inti.wali::tautkanAnak`<br>`inti.wali.tautkanAnak` | `SiswaWali::verifikasiHubungan`<br>`wali_siswa_id,siswa_id,hubungan,utama,bukti_verifikasi_ringkas` | AS. Lock siswa, wali utama tunggal, tautkan terverifikasi | `tests/Feature/Antarmuka/Inti/WaliSiswaTest.php`<br>`test_tautkan_anak_wali_tidak_melihat_anak_tak_terverifikasi` |
| `inti.wali::cabutHubungan`<br>`inti.wali.cabutHubungan` | `SiswaWali::cabutHubungan`<br>`id,alasan,versi_data` | AS. Cabut akses mulai request berikut; audit hubungan lama | `tests/Feature/Antarmuka/Inti/WaliSiswaTest.php`<br>`test_cabut_hubungan_pencabutan_tautan_menutup_akses` |
| `inti.akun::simpan`<br>`inti.akun.simpan` | `Pengguna::kelolaAkun`<br>`id?,nama,email,aktif,versi_data?` | AS. Admin tidak membuat owner atau memberi owner; akun alumni/siswa tetap satu | `tests/Feature/Antarmuka/Inti/AkunPenggunaTest.php`<br>`test_simpan_admin_tidak_dapat_menaikkan_ke_owner` |
| `inti.akun::tetapkanPeran`<br>`inti.akun.tetapkanPeran` | `Pengguna::tetapkanPeranOperasional`<br>`pengguna_id,peran_id[],alasan` | AS. Gabung peran yang diizinkan; tidak mengubah katalog hak fixed | `tests/Feature/Antarmuka/Inti/AkunPenggunaTest.php`<br>`test_tetapkan_peran_rangkap_peran_tetap_lingkup` |
| `inti.akun::nonaktifkan`<br>`inti.akun.nonaktifkan` | `Pengguna::nonaktifkan`<br>`pengguna_id,alasan,versi_data` | AS. Batalkan sesi, role/histori tidak dihapus | `tests/Feature/Antarmuka/Inti/AkunPenggunaTest.php`<br>`test_nonaktifkan_akun_nonaktif_ditolak_livewire_langsung` |
| `inti.akun::kirimReset`<br>`inti.akun.kirimReset` | `Pengguna::kirimResetTerotorisasi`<br>`pengguna_id` | AS. Broker Laravel, notifikasi sinkron; tidak menampilkan sandi | `tests/Feature/Antarmuka/Inti/AkunPenggunaTest.php`<br>`test_kirim_reset_reset_tidak_mengungkap_token` |
| `inti.penugasan::tetapkan`<br>`inti.penugasan.tetapkan` | `Pengguna::tetapkanPenugasan`<br>`jenis,FK_lingkup_typed,pengguna_atau_guru_id,berlaku_mulai,berlaku_sampai?,hak_tindakan[]` | AS dengan hak lingkup yang boleh diberikan. Jenis allowlist memetakan tabel source; XOR/FK/rentang dan subset hak | `tests/Feature/Antarmuka/Inti/PenugasanTest.php`<br>`test_tetapkan_wali_utama_overlap_ditolak` |
| `inti.penugasan::delegasikan`<br>`inti.penugasan.delegasikan` | `DelegasiTugas::tetapkan`<br>`jenis_tugas,FK_penugasan_typed,pengguna_pengganti_id,hak_tindakan[],mulai,sampai,alasan` | AS dengan hak lingkup yang boleh diberikan. Tidak melebihi pemberi, tidak ID model bebas | `tests/Feature/Antarmuka/Inti/PenugasanTest.php`<br>`test_delegasikan_delegasi_tidak_memperluas_hak` |
| `inti.penugasan::cabut`<br>`inti.penugasan.cabut` | `Pengguna::cabutPenugasan`<br>`jenis,id,alasan,tanggal_efektif,versi_data` | AS dengan hak lingkup yang boleh diberikan. Tutup rentang dengan audit, tidak mengubah pelaku historis | `tests/Feature/Antarmuka/Inti/PenugasanTest.php`<br>`test_cabut_wali_baru_berhak_wali_lama_tidak` |
| `inti.konteks_pkl::simpan`<br>`inti.konteks_pkl.simpan` | `KonteksPklSiswa::tetapkan`<br>`siswa_id,mulai,sampai,sumber,sumber_referensi?,alasan` | AS/W sesuai kelas. Konteks dasar, bukan absen lokasi; invalidasi versi daftar sesi | `tests/Feature/Antarmuka/Inti/KonteksPklSiswaTest.php`<br>`test_simpan_pkl_tidak_masuk_semua_hadir` |
| `inti.konteks_pkl::akhiri`<br>`inti.konteks_pkl.akhiri` | `KonteksPklSiswa::akhiri`<br>`id,tanggal_efektif,alasan,versi_data` | AS/W sesuai kelas. Akhiri konteks beralasan; histori/snapshot lama tetap | `tests/Feature/Antarmuka/Inti/KonteksPklSiswaTest.php`<br>`test_akhiri_nonaktif_pkl_tidak_menghapus_konteks` |
| `akademik.pindah::simpan`<br>`akademik.pindah.simpan` | `PenempatanKelasSiswa::pindahkan`<br>`siswa_id,kelas_tujuan_id,tanggal_efektif,alasan,versi_data` | AS akademik; data bidang lain memerlukan hak khusus. Baca snapshot/validasi; aksi pindah satu siswa transaksional | `tests/Feature/Antarmuka/Akademik/PindahKelasTest.php`<br>`test_simpan_pindah_lock_menolak_overlap` |
| `akademik.kenaikan::pratinjau`<br>`akademik.kenaikan.pratinjau` | `KelompokKenaikan::pratinjau`<br>`tahun_asal_id,tahun_tujuan_id,kelas_asal_id,kelas_tujuan_id,pilihan,pengecualian` | AS akademik; data bidang lain memerlukan hak khusus. Baca snapshot/validasi; aksi pindah satu siswa transaksional | `tests/Feature/Antarmuka/Akademik/KenaikanKelasTest.php`<br>`test_pratinjau_pratinjau_tidak_mutasi_siswa` |
| `akademik.kelulusan::pratinjau`<br>`akademik.kelulusan.pratinjau` | `Kelulusan::pratinjau`<br>`tahun_ajaran_id,pilihan,keputusan,tanggal_lulus,referensi_keputusan?` | AS akademik; data bidang lain memerlukan hak khusus. Baca snapshot/validasi; aksi pindah satu siswa transaksional | `tests/Feature/Antarmuka/Akademik/KelulusanTest.php`<br>`test_pratinjau_xii_empat_tahun_tidak_lulus` |
| `akademik.rekonsiliasi::pratinjau`<br>`akademik.rekonsiliasi.pratinjau` | `PenggabunganIdentitasSiswa::pratinjau`<br>`siswa_asal_id,siswa_utama_id` | AS akademik; data bidang lain memerlukan hak khusus. Baca snapshot/validasi; aksi pindah satu siswa transaksional | `tests/Feature/Antarmuka/Akademik/RekonsiliasiIdentitasTest.php`<br>`test_pratinjau_pratinjau_tidak_bocor_nominal_bk` |
| `akademik.rekonsiliasi::sahkan`<br>`akademik.rekonsiliasi.sahkan` | `PenggabunganIdentitasSiswa::sahkan`<br>`token_pratinjau,versi_data,alasan,hasil_konflik_bidang[]` | AS akademik; data bidang lain memerlukan hak khusus. Lock identitas/relasi, larang cycle, mapping alias tanpa hapus histori | `tests/Feature/Antarmuka/Akademik/RekonsiliasiIdentitasTest.php`<br>`test_sahkan_konflik_uang_tidak_dijumlah_otomatis` |
| `owner.modul::simpan`<br>`owner.modul.simpan` | `LisensiModul::ubahAktivasi`<br>`modul_id,aktif,mulai_pada?,berakhir_pada?,versi_status,alasan` | O + MFA; bukan akses data sekolah. Kesiapan source/skema, lock lisensi, audit/version/cache setelah commit | `tests/Feature/Antarmuka/Owner/ModulDanLisensiTest.php`<br>`test_simpan_aktivasi_modul_belum_siap_ditolak` |
| `owner.langganan::simpan`<br>`owner.langganan.simpan` | `LanggananSaas::simpanKontrak`<br>`id?,mulai_pada,berakhir_pada,paket_saat_kontrak,modul_saat_kontrak[],nilai_kontrak,catatan?,versi_data?` | O + MFA; bukan akses data sekolah. Satu sekolah, tidak otomatis mengubah lisensi | `tests/Feature/Antarmuka/Owner/LanggananTest.php`<br>`test_simpan_kontrak_bukan_aktivasi_lisensi` |
| `owner.pembayaran::catat`<br>`owner.pembayaran.catat` | `PembayaranLanggananSaas::catat`<br>`langganan_saas_id,nominal,tanggal,metode,referensi_bukti?,berkas_privat_id?` | O + MFA; bukan akses data sekolah. Pencatatan manual, bukan pembayaran siswa | `tests/Feature/Antarmuka/Owner/PembayaranLanggananTest.php`<br>`test_catat_bayar_saas_tidak_menyentuh_tagihan_siswa` |
| `owner.pembayaran::koreksi`<br>`owner.pembayaran.koreksi` | `PembayaranLanggananSaas::koreksi`<br>`id,alasan,versi_data` | O + MFA; bukan akses data sekolah. Audit sebelum/sesudah; record tidak dihapus | `tests/Feature/Antarmuka/Owner/PembayaranLanggananTest.php`<br>`test_koreksi_koreksi_saas_terlacak` |
| `akun.profil::simpan`<br>`akun.profil.simpan` | `Pengguna::ubahProfilSendiri`<br>`nama` | Diri sendiri semua role. Tidak menerima role/id pemilik/status akun dari form | `tests/Feature/Antarmuka/Akun/ProfilTest.php`<br>`test_simpan_profil_tidak_dapat_mengubah_peran` |
| `presensi.hari_ini::simpan`<br>`presensi.hari_ini.simpan` | `PengirimanPresensi::simpanKelas`<br>`token,kunci_idempotensi,sesi[{id?,kelas_id,mata_pelajaran_id,jam_ke,waktu_mulai,waktu_selesai,versi_data}],versi_daftar,kehadiran_lengkap[],pengecualian_snapshot` | G sesi sendiri/penugasan pengganti, hari server Jakarta. Seluruh siswa × sesi satu commit; jumlah/himpunan lengkap, H awal belum tersimpan | `tests/Feature/Antarmuka/Presensi/HariIniTest.php`<br>`test_simpan_satu_baris_gagal_rollback_seluruh_kelas` |
| `presensi.hari_ini::catatPerbedaan`<br>`presensi.hari_ini.catatPerbedaan` | `KlarifikasiPresensi::ajukan`<br>`sesi_id,siswa_id,status_usulan,versi_sumber,alasan` | G sesi sendiri/penugasan pengganti, hari server Jakarta. Observasi guru terpisah izin, tidak menimpa S/I diam-diam | `tests/Feature/Antarmuka/Presensi/HariIniTest.php`<br>`test_catat_perbedaan_prefill_izin_bukan_observasi_fiktif` |
| `presensi.izin_siswa::simpan`<br>`presensi.izin_siswa.simpan` | `PengajuanIzinSiswa::catatLangsung`<br>`siswa_id,waktu_mulai,waktu_selesai,status_kehadiran,alasan,kanal_penerimaan,nama_pelapor,hubungan_pelapor,berkas_privat_id?` | W/T dengan delegasi wali di kelas; bukan pejabat kedua. S/I langsung berlaku, versi sumber berubah dan konfirmasi lama dibatalkan berjejak | `tests/Feature/Antarmuka/Presensi/IzinSiswaTest.php`<br>`test_simpan_izin_telepon_tanpa_akun_lampiran` |
| `presensi.izin_siswa::periksaPengajuan`<br>`presensi.izin_siswa.periksaPengajuan` | `PengajuanIzinSiswa::periksaOnline`<br>`id,keputusan,alasan,versi_data` | W/T dengan delegasi wali di kelas; bukan pejabat kedua. Dari portal menunggu, wali verifikasi langsung, tidak approval pejabat | `tests/Feature/Antarmuka/Presensi/IzinSiswaTest.php`<br>`test_periksa_pengajuan_izin_online_cukup_verifikasi_wali` |
| `presensi.izin_siswa::batalkan`<br>`presensi.izin_siswa.batalkan` | `PengajuanIzinSiswa::batalkan`<br>`id,alasan,versi_data` | W/T dengan delegasi wali di kelas; bukan pejabat kedua. Status dasar independen dipulihkan atau Belum Diisi, bukan auto H/A | `tests/Feature/Antarmuka/Presensi/IzinSiswaTest.php`<br>`test_batalkan_batal_izin_tanpa_observasi_jadi_null` |
| `presensi.izin_pegawai::simpanDraf`<br>`presensi.izin_pegawai.simpanDraf` | `PengajuanIzinPegawai::simpanDraf`<br>`waktu_mulai,waktu_selesai,jenis,alasan,berkas_privat_id?,id?,versi_data?` | G/T/P pemilik. Pemilik dari sesi server, bukan parameter pengguna | `tests/Feature/Antarmuka/Presensi/IzinPegawaiTest.php`<br>`test_simpan_draf_izin_pegawai_bukan_presensi_guru` |
| `presensi.izin_pegawai::ajukan`<br>`presensi.izin_pegawai.ajukan` | `PengajuanIzinPegawai::ajukan`<br>`id,versi_data` | Pemilik. Validasi isi dan draf -> diajukan | `tests/Feature/Antarmuka/Presensi/IzinPegawaiTest.php`<br>`test_ajukan_draf_izin_belum_disetujui` |
| `presensi.izin_pegawai::putuskan`<br>`presensi.izin_pegawai.putuskan` | `PengajuanIzinPegawai::putuskan`<br>`id,keputusan,alasan,versi_data` | P bidang/petugas persetujuan yang ditugaskan. Pejabat lain berizin; tidak menyetujui diri sendiri | `tests/Feature/Antarmuka/Presensi/IzinPegawaiTest.php`<br>`test_putuskan_izin_diri_tidak_disetujui_sendiri` |
| `presensi.izin_pegawai::batalkan`<br>`presensi.izin_pegawai.batalkan` | `PengajuanIzinPegawai::batalkan`<br>`id,alasan,versi_data` | G/T pegawai pemilik; pejabat bidang memutuskan orang lain. Pemilik tarik sebelum diputuskan; koreksi keputusan hanya pejabat berizin/audit | `tests/Feature/Antarmuka/Presensi/IzinPegawaiTest.php`<br>`test_batalkan_izin_diputuskan_tidak_dihapus` |
| `presensi.konfirmasi::konfirmasi`<br>`presensi.konfirmasi.konfirmasi` | `KehadiranSiswa::konfirmasiKelas`<br>`kelas_id,tanggal,sesi_ids[],versi_data[]` | W kelas saat ini/delegasi historis. Tanpa tenggat; cek kelengkapan dan versi, guru merangkap wali boleh | `tests/Feature/Antarmuka/Presensi/KonfirmasiWaliTest.php`<br>`test_konfirmasi_konfirmasi_tanggal_lampau_tanpa_tenggat` |
| `presensi.konfirmasi::klarifikasi`<br>`presensi.konfirmasi.klarifikasi` | `KlarifikasiPresensi::putuskan`<br>`id,status_hasil,alasan,versi_sumber` | W kelas saat ini/delegasi historis. Selesaikan konflik sumber secara berjejak, koreksi minta konfirmasi ulang | `tests/Feature/Antarmuka/Presensi/KonfirmasiWaliTest.php`<br>`test_klarifikasi_konflik_s_i_tidak_last_write_wins` |
| `presensi.susulan::simpan`<br>`presensi.susulan.simpan` | `PengirimanPresensi::simpanSusulan`<br>`kelas_id,tanggal,jam_ke,guru_sebenarnya_id,mapel_id,alasan,daftar_historis_lengkap[],kunci_idempotensi,versi_daftar` | W/delegasi khusus; guru biasa tidak boleh. Aktor pencatat berbeda pengajar; satu transaksi seluruh daftar historis | `tests/Feature/Antarmuka/Presensi/SusulanTest.php`<br>`test_simpan_guru_tidak_bisa_memalsukan_tanggal_susulan` |
| `presensi.tindak_lanjut::catatTemuan`<br>`presensi.tindak_lanjut.catatTemuan` | `TindakLanjutPresensi::catatTemuan`<br>`siswa_id,kehadiran_ids[],ringkasan_masalah` | W/BK pada kelas/siswa ditugaskan; P hanya agregat. Temuan tanpa hukuman/poin otomatis | `tests/Feature/Antarmuka/Presensi/TindakLanjutTest.php`<br>`test_catat_temuan_temuan_hanya_siswa_lingkup` |
| `presensi.tindak_lanjut::klarifikasi`<br>`presensi.tindak_lanjut.klarifikasi` | `TindakLanjutPresensi::klarifikasi`<br>`id,hasil_klarifikasi,versi_data` | W/BK pada kelas/siswa ditugaskan; P hanya agregat. Wali menjelaskan sumber temuan | `tests/Feature/Antarmuka/Presensi/TindakLanjutTest.php`<br>`test_klarifikasi_wali_lama_tidak_memiliki_akses_kekal` |
| `presensi.tindak_lanjut::rujukBk`<br>`presensi.tindak_lanjut.rujukBk` | `TindakLanjutPresensi::rujukBk`<br>`id,guru_bk_id,ringkasan_aman,versi_data` | W/BK pada kelas/siswa ditugaskan; P hanya agregat. BK bila diperlukan; kolaborasi dasar tetap tanpa lisensi BK penuh | `tests/Feature/Antarmuka/Presensi/TindakLanjutTest.php`<br>`test_rujuk_bk_tanpa_modul_bk_pendampingan_tetap_jalan` |
| `presensi.tindak_lanjut::catatTindakan`<br>`presensi.tindak_lanjut.catatTindakan` | `TindakLanjutPresensi::catatTindakan`<br>`id,tindakan,hasil?,versi_data` | W/BK pada kelas/siswa ditugaskan; P hanya agregat. Tindak lanjut oleh penanggung jawab | `tests/Feature/Antarmuka/Presensi/TindakLanjutTest.php`<br>`test_catat_tindakan_isi_konseling_tidak_masuk_rekap` |
| `presensi.tindak_lanjut::selesaikan`<br>`presensi.tindak_lanjut.selesaikan` | `TindakLanjutPresensi::selesaikan`<br>`id,hasil,versi_data` | W/BK pada kelas/siswa ditugaskan; P hanya agregat. Selesai dengan hasil nyata | `tests/Feature/Antarmuka/Presensi/TindakLanjutTest.php`<br>`test_selesaikan_kasus_tidak_selesai_tanpa_hasil` |
| `presensi.tindak_lanjut::bukaKembali`<br>`presensi.tindak_lanjut.bukaKembali` | `TindakLanjutPresensi::bukaKembali`<br>`id,alasan,versi_data` | W/BK pada kelas/siswa ditugaskan; P hanya agregat. Kejadian berulang terbuka lagi dengan audit | `tests/Feature/Antarmuka/Presensi/TindakLanjutTest.php`<br>`test_buka_kembali_buka_kembali_tidak_menghapus_hasil_lama` |
| `siswa.izin::ajukan`<br>`siswa.izin.ajukan` | `PengajuanIzinSiswa::ajukanOnline`<br>`waktu_mulai,waktu_selesai,status_kehadiran,alasan,berkas_privat_id?` | S pemilik, WA hanya pada portal anak terverifikasi. Pemilik dari server; menunggu_pemeriksaan wali, bukan langsung S/I | `tests/Feature/Antarmuka/Siswa/IzinTest.php`<br>`test_ajukan_siswa_tidak_mengaktifkan_izin_sendiri` |
| `wali.izin::ajukan`<br>`wali.izin.ajukan` | `PengajuanIzinSiswa::ajukanDariWali`<br>`siswa_id,waktu_mulai,waktu_selesai,status_kehadiran,alasan,berkas_privat_id?` | WA anak terverifikasi; pengajuan opsional. Verifikasi tautan anak; menunggu wali kelas | `tests/Feature/Antarmuka/Wali/IzinTest.php`<br>`test_ajukan_wali_tidak_bisa_izin_anak_lain` |
| `jurnal.saya::simpanDraf`<br>`jurnal.saya.simpanDraf` | `JurnalKegiatanGuru::simpanDraf`<br>`id?,tanggal,rincian[{kegiatan,hasil,kelas_id?,mata_pelajaran_id?,jam_mulai?,jam_selesai?,kategori?,kendala?,tindak_lanjut?,berkas_privat_id?}],versi_data?` | G pemilik. Satu guru/hari, masa depan ditolak; kegiatan/hasil boleh belum lengkap pada draf | `tests/Feature/Antarmuka/JurnalGuru/JurnalSayaTest.php`<br>`test_simpan_draf_draf_bukan_terkirim` |
| `jurnal.saya::kirim`<br>`jurnal.saya.kirim` | `JurnalKegiatanGuru::kirim`<br>`id,versi_data,kunci_idempotensi` | G pemilik. Minimal satu kegiatan+hasil; tanpa jadwal/mapel/presensi wajib | `tests/Feature/Antarmuka/JurnalGuru/JurnalSayaTest.php`<br>`test_kirim_jurnal_nonmengajar_sah_tanpa_kelas` |
| `jurnal.saya::revisi`<br>`jurnal.saya.revisi` | `JurnalKegiatanGuru::buatRevisi`<br>`id,versi_data,alasan` | G pemilik. Versi baru, hasil/pemeriksaan lama tetap pada versi asal | `tests/Feature/Antarmuka/JurnalGuru/JurnalSayaTest.php`<br>`test_revisi_revisi_diperiksa_kembali_terkirim` |
| `jurnal.monitoring::mintaPerbaikan`<br>`jurnal.monitoring.mintaPerbaikan` | `VersiJurnalGuru::mintaPerbaikan`<br>`versi_jurnal_id,catatan,versi_data` | P kurikulum/program ditugaskan; hanya jurnal terkirim. Pimpinan tidak mengubah kegiatan guru | `tests/Feature/Antarmuka/JurnalGuru/MonitoringTest.php`<br>`test_minta_perbaikan_pimpinan_tidak_bisa_edit_isi` |
| `jurnal.monitoring::tandaiDiperiksa`<br>`jurnal.monitoring.tandaiDiperiksa` | `VersiJurnalGuru::periksa`<br>`versi_jurnal_id,catatan?,versi_data` | P kurikulum/program ditugaskan; hanya jurnal terkirim. Review terikat versi; tanpa skor kepatuhan kalender | `tests/Feature/Antarmuka/JurnalGuru/MonitoringTest.php`<br>`test_tandai_diperiksa_tidak_ada_jurnal_bukan_pelanggaran_otomatis` |
| `surat.daftar::simpanDraf`<br>`surat.daftar.simpanDraf` | `Surat::simpanDraf`<br>`id?,asal_surat,arah_surat,jenis,tanggal,perihal,pengirim_tujuan,nomor_asli?,metode_input,isi?,berkas_privat_id?,templat_surat_id?,perlu_persetujuan,versi_data?` | T petugas surat. Eksternal tanpa counter; file opsional semua metode; draf tidak bernomor final | `tests/Feature/Antarmuka/Surat/DaftarSuratTest.php`<br>`test_simpan_draf_surat_tanpa_berkas_dapat_disimpan` |
| `surat.daftar::ajukan`<br>`surat.daftar.ajukan` | `Surat::ajukanPersetujuan`<br>`id,versi_data` | T petugas surat. Ajukan hanya surat sekolah yang memerlukan penandatangan | `tests/Feature/Antarmuka/Surat/DaftarSuratTest.php`<br>`test_ajukan_pengajuan_tidak_mengambil_nomor_final` |
| `surat.daftar::finalkan`<br>`surat.daftar.finalkan` | `Surat::finalkan`<br>`id,versi_data,kunci_idempotensi` | T hak finalisasi. Jika perlu approval wajib sudah disetujui; lock urutan nomor, arsip final unik | `tests/Feature/Antarmuka/Surat/DaftarSuratTest.php`<br>`test_finalkan_dua_finalisasi_tidak_nomor_ganda` |
| `surat.daftar::buatRevisi`<br>`surat.daftar.buatRevisi` | `Surat::buatRevisi`<br>`id,alasan,versi_data` | T. Revisi menyimpan isi/metadata versi lama; nomor arsip tetap | `tests/Feature/Antarmuka/Surat/DaftarSuratTest.php`<br>`test_buat_revisi_revisi_tidak_menimpa_arsip_final` |
| `surat.daftar::buatPengganti`<br>`surat.daftar.buatPengganti` | `Surat::buatPengganti`<br>`id,alasan` | T. Record baru menunjuk surat lama; nomor baru saat final | `tests/Feature/Antarmuka/Surat/DaftarSuratTest.php`<br>`test_buat_pengganti_surat_pengganti_bukan_nomor_lama` |
| `surat.daftar::batalkan`<br>`surat.daftar.batalkan` | `Surat::batalkanFinal`<br>`id,alasan,versi_data` | T/P hak pembatalan. Record final tetap, nomor tidak dipakai ulang | `tests/Feature/Antarmuka/Surat/DaftarSuratTest.php`<br>`test_batalkan_nomor_batal_tidak_dipakai_ulang` |
| `surat.persetujuan::putuskan`<br>`surat.persetujuan.putuskan` | `Surat::putuskanPersetujuan`<br>`id,setuju,alasan,versi_data` | P penandatangan ditugaskan. Satu keputusan per versi; isi berubah memerlukan peninjauan baru | `tests/Feature/Antarmuka/Surat/PersetujuanTest.php`<br>`test_putuskan_persetujuan_versi_lama_tidak_mengesahkan_isi_baru` |
| `surat.templat::simpan`<br>`surat.templat.simpan` | `TemplatSurat::simpan`<br>`id?,kode,nama,isi_template,aktif,versi_data?` | T hak templat. Placeholder allowlist, tidak menjalankan PHP dari isi | `tests/Feature/Antarmuka/Surat/TemplatTest.php`<br>`test_simpan_templat_tidak_mengeksekusi_kode` |
| `website.halaman::simpanDraf`<br>`website.halaman.simpanDraf` | `RevisiKontenWebsite::simpanDraf`<br>`judul,slug,isi,ringkasan?,metadata,id?,versi_data?` | E hak konten/publikasi. Konten disanitasi, pointer draf terpisah, publik tetap versi terbit | `tests/Feature/Antarmuka/Website/HalamanTest.php`<br>`test_simpan_draf_draf_tidak_mengganti_versi_publik` |
| `website.halaman::terbitkan`<br>`website.halaman.terbitkan` | `RevisiKontenWebsite::terbitkan`<br>`id,nomor_versi,versi_data` | E hak konten/publikasi. Lock konten/media, publish/cache setelah commit, URL tetap | `tests/Feature/Antarmuka/Website/HalamanTest.php`<br>`test_terbitkan_dua_editor_konflik_tidak_timpa` |
| `website.halaman::arsipkan`<br>`website.halaman.arsipkan` | `RevisiKontenWebsite::arsipkan`<br>`id,alasan,versi_data` | E hak konten/publikasi. Lepas publikasi, simpan semua versi dan pemakaian media | `tests/Feature/Antarmuka/Website/HalamanTest.php`<br>`test_arsipkan_arsip_tidak_menghapus_versi` |
| `website.halaman::pulihkanVersi`<br>`website.halaman.pulihkanVersi` | `RevisiKontenWebsite::buatDrafDariVersi`<br>`id,nomor_versi_lama,alasan,versi_data` | E hak konten/publikasi. Buat draf baru, tidak mengubah sejarah | `tests/Feature/Antarmuka/Website/HalamanTest.php`<br>`test_pulihkan_versi_restore_versi_membuat_draf_baru` |
| `website.artikel::simpanDraf`<br>`website.artikel.simpanDraf` | `RevisiKontenWebsite::simpanDraf`<br>`judul,slug,kategori_artikel_website_id?,isi,ringkasan?,metadata,id?,versi_data?` | E hak konten/publikasi. Konten disanitasi, pointer draf terpisah, publik tetap versi terbit | `tests/Feature/Antarmuka/Website/ArtikelTest.php`<br>`test_simpan_draf_draf_tidak_mengganti_versi_publik` |
| `website.artikel::terbitkan`<br>`website.artikel.terbitkan` | `RevisiKontenWebsite::terbitkan`<br>`id,nomor_versi,versi_data` | E hak konten/publikasi. Lock konten/media, publish/cache setelah commit, URL tetap | `tests/Feature/Antarmuka/Website/ArtikelTest.php`<br>`test_terbitkan_dua_editor_konflik_tidak_timpa` |
| `website.artikel::arsipkan`<br>`website.artikel.arsipkan` | `RevisiKontenWebsite::arsipkan`<br>`id,alasan,versi_data` | E hak konten/publikasi. Lepas publikasi, simpan semua versi dan pemakaian media | `tests/Feature/Antarmuka/Website/ArtikelTest.php`<br>`test_arsipkan_arsip_tidak_menghapus_versi` |
| `website.artikel::pulihkanVersi`<br>`website.artikel.pulihkanVersi` | `RevisiKontenWebsite::buatDrafDariVersi`<br>`id,nomor_versi_lama,alasan,versi_data` | E hak konten/publikasi. Buat draf baru, tidak mengubah sejarah | `tests/Feature/Antarmuka/Website/ArtikelTest.php`<br>`test_pulihkan_versi_restore_versi_membuat_draf_baru` |
| `website.pengumuman::simpanDraf`<br>`website.pengumuman.simpanDraf` | `RevisiKontenWebsite::simpanDraf`<br>`judul,slug,tampil_mulai?,tampil_sampai?,isi,metadata,id?,versi_data?` | E hak konten/publikasi. Konten disanitasi, pointer draf terpisah, publik tetap versi terbit | `tests/Feature/Antarmuka/Website/PengumumanTest.php`<br>`test_simpan_draf_draf_tidak_mengganti_versi_publik` |
| `website.pengumuman::terbitkan`<br>`website.pengumuman.terbitkan` | `RevisiKontenWebsite::terbitkan`<br>`id,nomor_versi,versi_data` | E hak konten/publikasi. Lock konten/media, publish/cache setelah commit, URL tetap | `tests/Feature/Antarmuka/Website/PengumumanTest.php`<br>`test_terbitkan_dua_editor_konflik_tidak_timpa` |
| `website.pengumuman::arsipkan`<br>`website.pengumuman.arsipkan` | `RevisiKontenWebsite::arsipkan`<br>`id,alasan,versi_data` | E hak konten/publikasi. Lepas publikasi, simpan semua versi dan pemakaian media | `tests/Feature/Antarmuka/Website/PengumumanTest.php`<br>`test_arsipkan_arsip_tidak_menghapus_versi` |
| `website.pengumuman::pulihkanVersi`<br>`website.pengumuman.pulihkanVersi` | `RevisiKontenWebsite::buatDrafDariVersi`<br>`id,nomor_versi_lama,alasan,versi_data` | E hak konten/publikasi. Buat draf baru, tidak mengubah sejarah | `tests/Feature/Antarmuka/Website/PengumumanTest.php`<br>`test_pulihkan_versi_restore_versi_membuat_draf_baru` |
| `website.kategori::simpan`<br>`website.kategori.simpan` | `KategoriArtikelWebsite::simpan`<br>`id?,nama,slug,aktif,versi_data?` | E. Slug unik, nonaktif tidak menghapus berita | `tests/Feature/Antarmuka/Website/KategoriTest.php`<br>`test_simpan_kategori_nonaktif_tidak_hapus_berita` |
| `website.menu::simpan`<br>`website.menu.simpan` | `MenuWebsite::simpan`<br>`id?,induk_id?,judul,jenis,halaman_website_id?,kategori_artikel_website_id?,url_eksternal?,urutan,aktif,versi_data?` | E hak menu. XOR tujuan, kedalaman 2, tanpa cycle/script URL; induk mati menyembunyikan anak | `tests/Feature/Antarmuka/Website/MenuTest.php`<br>`test_simpan_menu_berbahaya_dan_cycle_ditolak` |
| `website.menu::urutkan`<br>`website.menu.urutkan` | `MenuWebsite::urutkan`<br>`induk_id?,urutan_id_lengkap[],versi_data[]` | E hak menu. Pindah satu grup dengan anak; tidak orphan | `tests/Feature/Antarmuka/Website/MenuTest.php`<br>`test_urutkan_submenu_ikut_induk_saat_urut` |
| `website.media::unggah` (HTTP `website.media.unggah`)<br>`website.media.unggah` | `MediaWebsite::simpanUnggahan`<br>`file,keterangan?` | E hak media. JPEG/PNG/WebP, ukuran tervalidasi, random path publik, bukan berkas privat siswa | `tests/Feature/Antarmuka/Website/MediaTest.php`<br>`test_unggah_berkas_privat_tidak_bisa_dijadikan_media` |
| `website.media::hapus`<br>`website.media.hapus` | `MediaWebsite::hapusJikaTidakDipakai`<br>`id,versi_data` | E hak media. Lock media, tolak bila direferensi draf/revisi/terbit/galeri/logo | `tests/Feature/Antarmuka/Website/MediaTest.php`<br>`test_hapus_media_draf_dan_terbit_tidak_bisa_dihapus` |
| `website.galeri::simpanAlbum`<br>`website.galeri.simpanAlbum` | `AlbumWebsite::simpan`<br>`id?,judul,slug,deskripsi?,versi_data?` | E. Album draf; media tercatat pemakaiannya | `tests/Feature/Antarmuka/Website/GaleriTest.php`<br>`test_simpan_album_album_draf_tidak_publik` |
| `website.galeri::simpanItem`<br>`website.galeri.simpanItem` | `ItemGaleriWebsite::simpan`<br>`album_id,id?,media_id?,url_video?,keterangan?,urutan` | E. XOR foto/video, provider video allowlist | `tests/Feature/Antarmuka/Website/GaleriTest.php`<br>`test_simpan_item_video_tidak_menerima_html_bebas` |
| `website.galeri::terbitkan`<br>`website.galeri.terbitkan` | `AlbumWebsite::terbitkan`<br>`id,versi_data` | E. Konten valid; referensi media terkunci | `tests/Feature/Antarmuka/Website/GaleriTest.php`<br>`test_terbitkan_galeri_hanya_album_terbit` |
| `website.galeri::arsipkan`<br>`website.galeri.arsipkan` | `AlbumWebsite::arsipkan`<br>`id,alasan,versi_data` | E. Album tidak publik, riwayat/referensi tetap | `tests/Feature/Antarmuka/Website/GaleriTest.php`<br>`test_arsipkan_arsip_album_tidak_merusak_media` |
| `website.tampilan::simpan`<br>`website.tampilan.simpan` | `ProfilSekolah::ubahTema`<br>`tema_website,versi_data` | E hak tampilan. Allowlist klasik/modern/vokasi/berita/kreatif; admin tetap satu desain | `tests/Feature/Antarmuka/Website/TampilanTest.php`<br>`test_simpan_tema_invalid_fallback_klasik` |
| `pelanggaran.catatan::catat`<br>`pelanggaran.catatan.catat` | `CatatanPelanggaran::catat`<br>`siswa_id,aturan_id,terjadi_pada,kejadian,berkas_privat_id?` | G/piket pada siswa lingkup, belum boleh finalisasi sendiri tanpa hak. Snapshot kelas/tahun/semester/aturan/bobot; menunggu validasi | `tests/Feature/Antarmuka/Pelanggaran/PencatatanTest.php`<br>`test_catat_bobot_master_baru_tidak_ubah_catatan_lama` |
| `pelanggaran.catatan::koreksi`<br>`pelanggaran.catatan.koreksi` | `CatatanPelanggaran::buatKoreksi`<br>`id,alasan,kejadian,versi_data` | G/piket pada siswa lingkup, belum boleh finalisasi sendiri tanpa hak. Versi baru, poin belum final sampai divalidasi | `tests/Feature/Antarmuka/Pelanggaran/PencatatanTest.php`<br>`test_koreksi_koreksi_tidak_menghitung_versi_ganda` |
| `pelanggaran.aturan::simpan`<br>`pelanggaran.aturan.simpan` | `AturanPelanggaran::simpan`<br>`id?,kode,nama,bobot,aktif,versi_data?` | P kesiswaan/petugas hak aturan. Bobot nonnegatif; versi kejadian lama tetap | `tests/Feature/Antarmuka/Pelanggaran/AturanTest.php`<br>`test_simpan_aturan_baru_tidak_retroaktif` |
| `pelanggaran.validasi::putuskan`<br>`pelanggaran.validasi.putuskan` | `CatatanPelanggaran::putuskan`<br>`id,nomor_versi,keputusan,alasan,versi_data` | P kesiswaan/petugas validasi siswa lingkup. Sah/perlu_koreksi/ditolak/dibatalkan; agregat versi efektif saja | `tests/Feature/Antarmuka/Pelanggaran/ValidasiTest.php`<br>`test_putuskan_catatan_batal_tidak_masuk_total` |
| `pelanggaran.validasi::putuskanKeberatan`<br>`pelanggaran.validasi.putuskanKeberatan` | `KeberatanPelanggaran::putuskan`<br>`id,keputusan,hasil,versi_data` | P kesiswaan/petugas validasi siswa lingkup. Jika diterima bentuk koreksi tercatat, bukan hapus catatan | `tests/Feature/Antarmuka/Pelanggaran/ValidasiTest.php`<br>`test_putuskan_keberatan_keberatan_tercatat_dengan_versi_asal` |
| `pelanggaran.pembinaan::simpan`<br>`pelanggaran.pembinaan.simpan` | `TindakLanjutPelanggaran::catat`<br>`catatan_pelanggaran_id,penanggung_jawab_id,tindakan` | W/BK/P kesiswaan lingkup. Tidak otomatis membuat sanksi atau membuka isi konseling | `tests/Feature/Antarmuka/Pelanggaran/PembinaanTest.php`<br>`test_simpan_pembinaan_bukan_hukuman_otomatis` |
| `pelanggaran.pembinaan::selesaikan`<br>`pelanggaran.pembinaan.selesaikan` | `TindakLanjutPelanggaran::selesaikan`<br>`id,hasil,versi_data` | W/BK/P kesiswaan lingkup. Tutup dengan hasil nyata, audit | `tests/Feature/Antarmuka/Pelanggaran/PembinaanTest.php`<br>`test_selesaikan_pembinaan_selesai_berhasil_tercatat` |
| `siswa.pelanggaran::ajukanKeberatan`<br>`siswa.pelanggaran.ajukanKeberatan` | `KeberatanPelanggaran::ajukan`<br>`catatan_pelanggaran_id,versi_yang_disanggah,alasan` | S pemilik hanya catatan sah. Siswa sendiri, tidak mengubah poin final | `tests/Feature/Antarmuka/Siswa/PelanggaranTest.php`<br>`test_ajukan_keberatan_siswa_tidak_bisa_koreksi_poin_sendiri` |
| `wali.pembinaan::ajukanKeberatan`<br>`wali.pembinaan.ajukanKeberatan` | `KeberatanPelanggaran::ajukanDariWali`<br>`catatan_pelanggaran_id,versi_yang_disanggah,alasan` | WA anak terverifikasi, hanya catatan sah/ringkasan diizinkan. Verifikasi hubungan anak setiap request | `tests/Feature/Antarmuka/Wali/PembinaanTest.php`<br>`test_ajukan_keberatan_wali_tidak_dapat_menggugat_siswa_lain` |
| `bk.permintaan::terima`<br>`bk.permintaan.terima` | `KasusKonseling::terimaPermintaan`<br>`permintaan_id,penanggung_jawab_id,versi_data` | BK ditugaskan, tanpa akses owner/AS default. Satu kasus per permintaan; langsung tidak butuh rujukan wali | `tests/Feature/Antarmuka/Bk/PermintaanTest.php`<br>`test_terima_permintaan_langsung_tanpa_wali` |
| `bk.permintaan::tolak`<br>`bk.permintaan.tolak` | `PermintaanKonseling::tolak`<br>`id,alasan,versi_data` | BK ditugaskan, tanpa akses owner/AS default. Alasan aman pada pemohon, bukan catatan klinis | `tests/Feature/Antarmuka/Bk/PermintaanTest.php`<br>`test_tolak_penolakan_tidak_bocorkan_catatan` |
| `bk.kasus::gantiPenanggungJawab`<br>`bk.kasus.gantiPenanggungJawab` | `KasusKonseling::gantiPenanggungJawab`<br>`id,pengguna_baru_id,alasan,versi_data` | BK penanggung jawab/delegasi sah. Tutup rentang lama, akses petugas lama dicabut | `tests/Feature/Antarmuka/Bk/KasusTest.php`<br>`test_ganti_penanggung_jawab_petugas_lama_tidak_bisa_baca_setelah_diganti` |
| `bk.kasus::selesaikan`<br>`bk.kasus.selesaikan` | `KasusKonseling::selesaikan`<br>`id,hasil_aman,versi_data` | BK penanggung jawab/delegasi sah. Hasil aman terpisah catatan privat | `tests/Feature/Antarmuka/Bk/KasusTest.php`<br>`test_selesaikan_rekap_wali_tidak_memuat_catatan_sesi` |
| `bk.kasus::bukaKembali`<br>`bk.kasus.bukaKembali` | `KasusKonseling::bukaKembali`<br>`id,alasan,versi_data` | BK penanggung jawab/delegasi sah. Riwayat penyelesaian lama tetap | `tests/Feature/Antarmuka/Bk/KasusTest.php`<br>`test_buka_kembali_buka_kembali_beralasan` |
| `bk.sesi::simpanDraf`<br>`bk.sesi.simpanDraf` | `SesiKonseling::simpanDraf`<br>`id?,kasus_id,waktu_mulai,waktu_selesai?,catatan_privat,tindak_lanjut_privat?,versi_data?` | BK penanggung jawab/delegasi; kolom privat terenkripsi. Encrypt; tidak masuk log/progres/ekspor umum | `tests/Feature/Antarmuka/Bk/SesiTest.php`<br>`test_simpan_draf_isi_bk_tidak_ada_di_log` |
| `bk.sesi::sahkan`<br>`bk.sesi.sahkan` | `SesiKonseling::sahkan`<br>`id,versi_data` | BK penanggung jawab/delegasi; kolom privat terenkripsi. Versi sah dipertahankan | `tests/Feature/Antarmuka/Bk/SesiTest.php`<br>`test_sahkan_sesi_sah_tidak_ditumpuk_draf` |
| `bk.sesi::revisi`<br>`bk.sesi.revisi` | `SesiKonseling::revisi`<br>`id,alasan,catatan_privat,tindak_lanjut_privat?,versi_data` | BK penanggung jawab/delegasi; kolom privat terenkripsi. Revisi terenkripsi baru, tidak hapus catatan asal | `tests/Feature/Antarmuka/Bk/SesiTest.php`<br>`test_revisi_revisi_bk_terenkripsi_dan_terlacak` |
| `bk.sesi::batalkan`<br>`bk.sesi.batalkan` | `SesiKonseling::batalkan`<br>`id,alasan,versi_data` | BK penanggung jawab/delegasi; kolom privat terenkripsi. Sesi batal tetap ada di arsip privat | `tests/Feature/Antarmuka/Bk/SesiTest.php`<br>`test_batalkan_sesi_batal_tidak_dihapus` |
| `siswa.bk::ajukan`<br>`siswa.bk.ajukan` | `PermintaanKonseling::ajukanLangsung`<br>`ringkasan,waktu_diharapkan?` | S pemilik, tidak membutuhkan persetujuan wali. Pemilik server, sumber langsung tanpa rujukan | `tests/Feature/Antarmuka/Siswa/LayananBkTest.php`<br>`test_ajukan_siswa_langsung_bisa_ajukan_bk` |
| `siswa.bk::batalkan`<br>`siswa.bk.batalkan` | `PermintaanKonseling::batalkan`<br>`id,alasan,versi_data` | S pemilik, tidak membutuhkan persetujuan wali. Pemilik sebelum ditangani; kasus terbuka melalui BK | `tests/Feature/Antarmuka/Siswa/LayananBkTest.php`<br>`test_batalkan_permintaan_orang_lain_tidak_bisa_dibatalkan` |
| `jadwal.penyusunan::simpanDraf`<br>`jadwal.penyusunan.simpanDraf` | `VersiJadwal::simpanDraf`<br>`id?,tahun_ajaran_id,berlaku_mulai,berlaku_sampai?,rincian[{kelas_id,guru_id,mapel_id,ruang_id?,hari_ke,jam_ke,waktu_mulai,waktu_selesai}],versi_data?` | P kurikulum/program lingkup. Draf versi, satu guru satu kelas tanpa split | `tests/Feature/Antarmuka/Jadwal/PenyusunanTest.php`<br>`test_simpan_draf_jadwal_tidak_memerlukan_kalender_libur` |
| `jadwal.penyusunan::periksaBentrok`<br>`jadwal.penyusunan.periksaBentrok` | `VersiJadwal::periksaBentrok`<br>`id,versi_data` | P kurikulum/program lingkup. Baca konflik guru/kelas/ruang lintas modul tanpa publikasi | `tests/Feature/Antarmuka/Jadwal/PenyusunanTest.php`<br>`test_periksa_bentrok_bentrok_lab_tetap_saat_modul_nonaktif` |
| `jadwal.penyusunan::terbitkan`<br>`jadwal.penyusunan.terbitkan` | `VersiJadwal::terbitkan`<br>`id,versi_data` | P kurikulum/program lingkup. Lock guru/kelas/ruang dan versi; publikasi snapshot penggunaan ruang | `tests/Feature/Antarmuka/Jadwal/PenyusunanTest.php`<br>`test_terbitkan_dua_publikasi_bentrok_ditolak` |
| `jadwal.penyusunan::buatRevisi`<br>`jadwal.penyusunan.buatRevisi` | `VersiJadwal::buatRevisi`<br>`id,alasan` | P kurikulum/program lingkup. Versi baru tanpa menimpa sesi lama | `tests/Feature/Antarmuka/Jadwal/PenyusunanTest.php`<br>`test_buat_revisi_presensi_historis_tidak_ikut_jadwal_baru` |
| `jadwal.penyusunan::catatPengecualian`<br>`jadwal.penyusunan.catatPengecualian` | `PengecualianJadwal::catat`<br>`jadwal_id,tanggal,jenis,delegasi_tugas_id?,alasan` | P kurikulum/program lingkup. Pengganti sah atau batal sesi nyata | `tests/Feature/Antarmuka/Jadwal/PenyusunanTest.php`<br>`test_catat_pengecualian_guru_pengganti_perlu_delegasi` |
| `jadwal.penyusunan::batalkan`<br>`jadwal.penyusunan.batalkan` | `VersiJadwal::batalkan`<br>`id,alasan,versi_data` | P kurikulum/program lingkup. Batalkan masa depan, simpan histori/pemakaian lama | `tests/Feature/Antarmuka/Jadwal/PenyusunanTest.php`<br>`test_batalkan_batal_jadwal_tidak_menghapus_presensi` |
| `pkl.penempatan::simpanDraf`<br>`pkl.penempatan.simpanDraf` | `PenempatanPkl::simpanDraf`<br>`id?,siswa_id,periode_pkl_id,lokasi_pkl_id,guru_pembimbing_id,berlaku_mulai,berlaku_sampai,versi_data?` | K. Draf segmen, belum mengurangi kuota aktif | `tests/Feature/Antarmuka/Pkl/PenempatanTest.php`<br>`test_simpan_draf_draf_penempatan_belum_aktif` |
| `pkl.penempatan::aktifkan`<br>`pkl.penempatan.aktifkan` | `PenempatanPkl::aktifkan`<br>`id,versi_data` | K. Lock siswa/kuota/lokasi; konteks Core ikut satu commit, jangan overlap | `tests/Feature/Antarmuka/Pkl/PenempatanTest.php`<br>`test_aktifkan_dua_penempatan_tidak_melampaui_kuota` |
| `pkl.penempatan::pindahkan`<br>`pkl.penempatan.pindahkan` | `PenempatanPkl::pindahkanSegmen`<br>`id,lokasi_baru_id,pembimbing_baru_id,tanggal_efektif,alasan,versi_data` | K. Tutup segmen lama, buat baru, pertahankan referensi jurnal/nilai/lokasi lama | `tests/Feature/Antarmuka/Pkl/PenempatanTest.php`<br>`test_pindahkan_pindah_mitra_tidak_memindahkan_jurnal_lama` |
| `pkl.penempatan::hentikan`<br>`pkl.penempatan.hentikan` | `PenempatanPkl::hentikan`<br>`id,tanggal_akhir,alasan,versi_data` | K. Sebelum mulai dibatalkan; sudah berjalan dihentikan; konteks Core berakhir sah | `tests/Feature/Antarmuka/Pkl/PenempatanTest.php`<br>`test_hentikan_pkl_berjalan_tidak_dihapus` |
| `pkl.penempatan::selesaikan`<br>`pkl.penempatan.selesaikan` | `PenempatanPkl::selesaikan`<br>`id,tanggal_selesai,alasan?,versi_data` | K. Selesai eksplisit, tidak dipicu MoU berakhir | `tests/Feature/Antarmuka/Pkl/PenempatanTest.php`<br>`test_selesaikan_mou_habis_tidak_menyelesaikan_pkl` |
| `pkl.periode::simpan`<br>`pkl.periode.simpan` | `PeriodePkl::simpan`<br>`id?,nama,tahun_ajaran_id,mulai_pada,selesai_pada,koordinator_id,versi_data?` | K. Periode tanpa kalender kerja | `tests/Feature/Antarmuka/Pkl/PeriodeTest.php`<br>`test_simpan_periode_tanpa_kalender_dapat_dipakai` |
| `pkl.periode::tetapkanKuota`<br>`pkl.periode.tetapkanKuota` | `KuotaPkl::tetapkan`<br>`periode_id,lokasi_id,kuota,versi_data?` | K. Tidak menurunkan di bawah peserta sah pada rentang overlap | `tests/Feature/Antarmuka/Pkl/PeriodeTest.php`<br>`test_tetapkan_kuota_turunkan_kuota_tidak_merusak_peserta` |
| `pkl.lokasi::simpan`<br>`pkl.lokasi.simpan` | `LokasiPkl::simpan`<br>`id?,mitra_id,kode,nama,aktif,versi_data?` | K. Master lokasi satu mitra, siswa tidak boleh edit | `tests/Feature/Antarmuka/Pkl/LokasiTest.php`<br>`test_simpan_siswa_ditolak_mengubah_lokasi` |
| `pkl.lokasi::buatVersi`<br>`pkl.lokasi.buatVersi` | `VersiLokasiPkl::buatVersi`<br>`lokasi_id,lintang,bujur,radius_meter,batas_akurasi_meter,berlaku_mulai,alasan,versi_data` | K. Versi koordinat/radius baru, akurasi min(50,radius/2), bukti lama utuh | `tests/Feature/Antarmuka/Pkl/LokasiTest.php`<br>`test_buat_versi_radius_baru_tidak_mengubah_bukti_lama` |
| `pkl.monitoring::catatKunjungan`<br>`pkl.monitoring.catatKunjungan` | `MonitoringPkl::catat`<br>`penempatan_id,tanggal,kegiatan,hasil,tindak_lanjut?,berkas_privat_id?` | G pembimbing/K koordinator segmen saat ini/delegasi historis. Catatan guru pada segmen yang benar | `tests/Feature/Antarmuka/Pkl/MonitoringTest.php`<br>`test_catat_kunjungan_guru_lain_tidak_bisa_monitor_segmen` |
| `pkl.monitoring::periksaJurnal`<br>`pkl.monitoring.periksaJurnal` | `VersiJurnalPkl::periksa`<br>`jurnal_id,nomor_versi,keputusan,catatan?,versi_data` | G pembimbing/K koordinator segmen saat ini/delegasi historis. Diperiksa/perlu_perbaikan terikat versi, isi siswa tidak ditimpa | `tests/Feature/Antarmuka/Pkl/MonitoringTest.php`<br>`test_periksa_jurnal_revisi_jurnal_pkl_bukan_menimpa` |
| `pkl.penilaian::simpanDraf`<br>`pkl.penilaian.simpanDraf` | `VersiPenilaianPkl::simpanDraf`<br>`penempatan_id,nilai_industri,komponen_sekolah?,nama_penilai_industri,tanggal_penilaian,berkas_privat_id?,versi_data` | G pembimbing. 0..100, penilai/tanggal wajib; tidak ada login industri | `tests/Feature/Antarmuka/Pkl/PenilaianTest.php`<br>`test_simpan_draf_nilai_kertas_tanpa_scan_sah` |
| `pkl.penilaian::finalkan`<br>`pkl.penilaian.finalkan` | `PenilaianPkl::finalkan`<br>`id,nomor_versi,versi_data` | K. Kunci versi final, asal kertas disimpan | `tests/Feature/Antarmuka/Pkl/PenilaianTest.php`<br>`test_finalkan_nilai_final_tidak_diedit_langsung` |
| `pkl.penilaian::bukaRevisi`<br>`pkl.penilaian.bukaRevisi` | `PenilaianPkl::bukaRevisi`<br>`id,alasan,versi_data` | K. Koordinator membuka versi baru; versi final lama tetap | `tests/Feature/Antarmuka/Pkl/PenilaianTest.php`<br>`test_buka_revisi_revisi_final_perlu_alasan_koordinator` |
| `pkl.bantuan::putuskan`<br>`pkl.bantuan.putuskan` | `PermintaanBantuanPkl::putuskan`<br>`id,keputusan,hasil,versi_data` | G pembimbing/K dalam segmen. Permintaan disetujui tidak otomatis GPS berhasil; koreksi terpisah berizin | `tests/Feature/Antarmuka/Pkl/BantuanTest.php`<br>`test_putuskan_bantuan_bukan_hadir_otomatis` |
| `pkl.bantuan::catatVerifikasi`<br>`pkl.bantuan.catatVerifikasi` | `PeristiwaKehadiranPkl::koreksiPetugas`<br>`penempatan_id,tanggal_kerja,jenis,waktu_fakta,status_harian,alasan,referensi_bukti,kunci_idempotensi` | G pembimbing/K dalam segmen. Sumber verifikasi_petugas; koordinat tidak direkayasa; versi efektif/giliran masuk-pulang sah | `tests/Feature/Antarmuka/Pkl/BantuanTest.php`<br>`test_catat_verifikasi_verifikasi_manual_tidak_dilabeli_lolos_gps` |
| `hubin.mitra::simpan`<br>`hubin.mitra.simpan` | `MitraIndustri::simpan`<br>`id?,kode,nama,alamat?,kontak?,nama_pic?,kontak_pic?,aktif,versi_data?` | K atau G/T/P HUBIN atau PA BKK sesuai hak master, salah satu modul aktif. Hak master dari salah satu HUBIN/PKL/BKK aktif, bukan ketiganya wajib | `tests/Feature/Antarmuka/Inti/MitraIndustriTest.php`<br>`test_simpan_pkl_tanpa_hubin_dapat_kelola_mitra` |
| `hubin.kerja_sama::simpan`<br>`hubin.kerja_sama.simpan` | `KerjaSamaIndustri::simpanDraf`<br>`id?,mitra_id,nomor,jenis,mulai_pada,berakhir_pada,berkas_privat_id?,versi_data?` | G/T/P HUBIN ditugaskan. Draf, nomor/dokumen/periode tersimpan | `tests/Feature/Antarmuka/Hubin/KerjaSamaTest.php`<br>`test_simpan_dokumen_hubin_opsional` |
| `hubin.kerja_sama::sahkan`<br>`hubin.kerja_sama.sahkan` | `KerjaSamaIndustri::sahkan`<br>`id,versi_data` | G/T/P HUBIN ditugaskan. Kerja sama efektif menurut rentang saat request | `tests/Feature/Antarmuka/Hubin/KerjaSamaTest.php`<br>`test_sahkan_mou_kedaluwarsa_tanpa_cron` |
| `hubin.kerja_sama::buatAdendum`<br>`hubin.kerja_sama.buatAdendum` | `AdendumKerjaSama::catat`<br>`kerja_sama_id,nomor_adendum,berlaku_mulai,berlaku_sampai,isi_perubahan,berkas_privat_id?` | G/T/P HUBIN ditugaskan. Append versi/adendum, bukan timpa dokumen | `tests/Feature/Antarmuka/Hubin/KerjaSamaTest.php`<br>`test_buat_adendum_adendum_tidak_menghapus_versi_lama` |
| `hubin.kerja_sama::perpanjang`<br>`hubin.kerja_sama.perpanjang` | `KerjaSamaIndustri::perpanjang`<br>`id,nomor,mulai_pada,berakhir_pada,alasan` | G/T/P HUBIN ditugaskan. Record baru menunjuk lama | `tests/Feature/Antarmuka/Hubin/KerjaSamaTest.php`<br>`test_perpanjang_perpanjangan_memiliki_riwayat` |
| `hubin.kerja_sama::batalkan`<br>`hubin.kerja_sama.batalkan` | `KerjaSamaIndustri::batalkan`<br>`id,alasan,versi_data` | G/T/P HUBIN ditugaskan. Tidak membatalkan penempatan/nilai/jurnal PKL | `tests/Feature/Antarmuka/Hubin/KerjaSamaTest.php`<br>`test_batalkan_batal_mou_tidak_mematikan_pkl` |
| `hubin.kegiatan::simpan`<br>`hubin.kegiatan.simpan` | `KegiatanHubin::catat`<br>`mitra_id,kerja_sama_id?,tanggal,nama,pelaksana_id,hasil,berkas_privat_id?` | G/T/P HUBIN ditugaskan. Kegiatan dan hasil pada mitra yang sama | `tests/Feature/Antarmuka/Hubin/KegiatanTest.php`<br>`test_simpan_kegiatan_tidak_boleh_mitra_berbeda` |
| `alumni.verifikasi::putuskan`<br>`alumni.verifikasi.putuskan` | `PengajuanRegistrasiAlumni::putuskan`<br>`id,keputusan,profil_atau_siswa_cocok_id?,alasan,versi_data` | PA, BKK atau Tracer aktif. Cocokkan arsip manual, link akun/profil tanpa siswa fiktif; klarifikasi/ditolak tak membuka akses alumni | `tests/Feature/Antarmuka/Alumni/VerifikasiTest.php`<br>`test_putuskan_klaim_alumni_tidak_otomatis_dari_nis` |
| `alumni.rekonsiliasi::pratinjau`<br>`alumni.rekonsiliasi.pratinjau` | `PenggabunganIdentitasAlumni::pratinjau`<br>`alumni_asal_id,alumni_utama_id` | PA berhak identitas, konflik domain oleh petugas bidang. Tanpa data privat tidak berizin; tidak mutasi | `tests/Feature/Antarmuka/Alumni/RekonsiliasiTest.php`<br>`test_pratinjau_pratinjau_alumni_tidak_bocor_uang` |
| `alumni.rekonsiliasi::sahkan`<br>`alumni.rekonsiliasi.sahkan` | `PenggabunganIdentitasAlumni::sahkan`<br>`token_pratinjau,alasan,konflik_terselesaikan[]` | PA berhak identitas, konflik domain oleh petugas bidang. Alias/akun utama, lock konflik lamaran/jawaban, tidak menjumlah histori | `tests/Feature/Antarmuka/Alumni/RekonsiliasiTest.php`<br>`test_sahkan_rekonsiliasi_alumni_mencegah_cycle` |
| `alumni.profil::simpan`<br>`alumni.profil.simpan` | `ProfilAlumni::ubahProfilSendiri`<br>`kontak,alamat?` | AL terverifikasi pemilik, BKK atau Tracer. Identitas kelulusan/tautan siswa tidak bisa diubah sendiri | `tests/Feature/Antarmuka/Alumni/ProfilTest.php`<br>`test_simpan_alumni_tidak_mengubah_tahun_lulus_sendiri` |
| `alumni.riwayat::simpan`<br>`alumni.riwayat.simpan` | `RiwayatAlumni::simpan`<br>`id?,jenis,organisasi,jabatan_atau_program?,tanggal_mulai,tanggal_selesai?,catatan?,versi_data?` | AL pemilik, BKK atau Tracer. Aktivitas dapat paralel; tidak menimpa riwayat lama | `tests/Feature/Antarmuka/Alumni/RiwayatTest.php`<br>`test_simpan_kerja_kuliah_usaha_boleh_bersamaan` |
| `alumni.riwayat::akhiri`<br>`alumni.riwayat.akhiri` | `RiwayatAlumni::akhiri`<br>`id,tanggal_selesai,alasan,versi_data` | AL pemilik, BKK atau Tracer. Menutup tanggal bukan menghapus riwayat | `tests/Feature/Antarmuka/Alumni/RiwayatTest.php`<br>`test_akhiri_riwayat_berakhir_masih_terbaca` |
| `alumni.riwayat::catatPribadi`<br>`alumni.riwayat.catatPribadi` | `PeristiwaPribadiAlumni::catat`<br>`jenis_peristiwa,tanggal_kejadian?,catatan?` | AL pemilik, BKK atau Tracer. Opsional privat terenkripsi, bukan data wajib survei | `tests/Feature/Antarmuka/Alumni/RiwayatTest.php`<br>`test_catat_pribadi_pernikahan_tidak_muncul_di_agregat` |
| `bkk.lowongan::simpanDraf`<br>`bkk.lowongan.simpanDraf` | `LowonganKerja::simpanDraf`<br>`id?,mitra_id,judul,batas_lamar,untuk_siswa_akhir,syarat_snapshot,deskripsi,versi_data?` | PA BKK. Versi syarat baru; snapshot lamaran lama tetap | `tests/Feature/Antarmuka/Bkk/LowonganTest.php`<br>`test_simpan_draf_syarat_baru_tidak_menimpa_lamaran` |
| `bkk.lowongan::terbitkan`<br>`bkk.lowongan.terbitkan` | `LowonganKerja::terbitkan`<br>`id,versi_data` | PA BKK. Tenggat belum lewat, mitra sah | `tests/Feature/Antarmuka/Bkk/LowonganTest.php`<br>`test_terbitkan_lowongan_kedaluwarsa_tidak_diterbitkan` |
| `bkk.lowongan::tutup`<br>`bkk.lowongan.tutup` | `LowonganKerja::tutup`<br>`id,alasan,versi_data` | PA BKK. Tutup manual atau efektif tenggat server; lamaran lama tetap | `tests/Feature/Antarmuka/Bkk/LowonganTest.php`<br>`test_tutup_tenggat_diperiksa_server_tanpa_cron` |
| `bkk.lowongan::batalkan`<br>`bkk.lowongan.batalkan` | `LowonganKerja::batalkan`<br>`id,alasan,versi_data` | PA BKK. Riwayat alasan dan lamaran terdampak | `tests/Feature/Antarmuka/Bkk/LowonganTest.php`<br>`test_batalkan_lowongan_batal_tidak_hapus_pelamar` |
| `bkk.pelamar::catatStatus`<br>`bkk.pelamar.catatStatus` | `LamaranKerja::catatStatus`<br>`id,status_baru,alasan,versi_data` | PA BKK. Riwayat status; diterima tidak otomatis riwayat pekerjaan | `tests/Feature/Antarmuka/Bkk/PelamarTest.php`<br>`test_catat_status_diterima_bukan_riwayat_kerja_otomatis` |
| `bkk.pelamar::koreksiHasil`<br>`bkk.pelamar.koreksiHasil` | `LamaranKerja::koreksiHasil`<br>`id,status_baru,alasan,versi_data` | PA BKK. Versi baru mempertahankan hasil lama | `tests/Feature/Antarmuka/Bkk/PelamarTest.php`<br>`test_koreksi_hasil_koreksi_seleksi_terlacak` |
| `bkk.penyerahan::catat`<br>`bkk.penyerahan.catat` | `PenyerahanKandidat::catat`<br>`lowongan_id,lamaran_ids[],nama_pic,kanal,waktu_penyerahan,referensi_pengiriman?,persetujuan_per_berkas[]` | PA hak penyerahan/CV. Hanya CV yang dipilih dan diizinkan, komunikasi manual | `tests/Feature/Antarmuka/Bkk/PenyerahanTest.php`<br>`test_catat_terkirim_tidak_sama_diterima_kerja` |
| `bkk.penyerahan::kirimUlang`<br>`bkk.penyerahan.kirimUlang` | `PenyerahanKandidat::catatKirimUlang`<br>`id,alasan,berkas_terpilih_dan_diizinkan[]` | PA hak penyerahan/CV. Versi penyerahan baru, bukan kirim otomatis | `tests/Feature/Antarmuka/Bkk/PenyerahanTest.php`<br>`test_kirim_ulang_kirim_ulang_mempertahankan_versi_berkas` |
| `alumni.lamaran::ajukan`<br>`alumni.lamaran.ajukan` | `LamaranKerja::ajukan`<br>`lowongan_id,cv_berkas_id?,persetujuan_tujuan` | AL terverifikasi pemilik; siswa akhir melalui portal siswa. Pemilik dari server; tenggat/UNIQUE, CV opsional | `tests/Feature/Antarmuka/Alumni/LamaranTest.php`<br>`test_ajukan_dua_lamaran_lowongan_sama_satu_record` |
| `alumni.lamaran::tarik`<br>`alumni.lamaran.tarik` | `LamaranKerja::tarik`<br>`id,alasan,versi_data` | AL terverifikasi pemilik; siswa akhir melalui portal siswa. Tarik sebelum status terminal sesuai lowongan, riwayat tetap | `tests/Feature/Antarmuka/Alumni/LamaranTest.php`<br>`test_tarik_tarik_lamaran_tidak_menghapus` |
| `alumni.lamaran::ajukanUlang`<br>`alumni.lamaran.ajukanUlang` | `LamaranKerja::ajukanUlang`<br>`id,cv_berkas_id?,persetujuan_tujuan,versi_data` | AL terverifikasi pemilik; siswa akhir melalui portal siswa. Record yang sama bila lowongan masih terbuka | `tests/Feature/Antarmuka/Alumni/LamaranTest.php`<br>`test_ajukan_ulang_ajukan_ulang_tidak_duplikasi` |
| `siswa.lowongan::ajukan`<br>`siswa.lowongan.ajukan` | `LamaranKerja::ajukanSiswaAkhir`<br>`lowongan_id,cv_berkas_id?,persetujuan_tujuan` | S tingkat akhir sah, lowongan mengizinkan; bukan alumni fiktif. XOR siswa/alumni kanonik, tidak membuat profil alumni sebelum lulus | `tests/Feature/Antarmuka/Siswa/LowonganTest.php`<br>`test_ajukan_siswa_belum_lulus_tidak_membuat_alumni` |
| `siswa.lowongan::tarik`<br>`siswa.lowongan.tarik` | `LamaranKerja::tarik`<br>`id,alasan,versi_data` | S tingkat akhir sah, lowongan mengizinkan; bukan alumni fiktif. Hanya pemilik siswa akhir, riwayat tetap | `tests/Feature/Antarmuka/Siswa/LowonganTest.php`<br>`test_tarik_siswa_tidak_bisa_tarik_lamaran_orang_lain` |
| `siswa.lowongan::ajukanUlang`<br>`siswa.lowongan.ajukanUlang` | `LamaranKerja::ajukanUlang`<br>`id,cv_berkas_id?,persetujuan_tujuan,versi_data` | S tingkat akhir sah, lowongan mengizinkan; bukan alumni fiktif. Record sama, syarat/tenggat diperiksa lagi | `tests/Feature/Antarmuka/Siswa/LowonganTest.php`<br>`test_ajukan_ulang_siswa_ajukan_ulang_record_sama` |
| `tracer.periode::simpan`<br>`tracer.periode.simpan` | `PeriodeTracer::simpanDraf`<br>`id?,nama,angkatan_mulai,angkatan_selesai,buka_pada,tutup_pada,versi_data?` | PA Tracer. Periode draf, form dan sasaran sebelum buka | `tests/Feature/Antarmuka/Tracer/PeriodeTest.php`<br>`test_simpan_periode_tanpa_form_tidak_dibuka` |
| `tracer.periode::buka`<br>`tracer.periode.buka` | `PeriodeTracer::buka`<br>`id,versi_data` | PA Tracer. Kunci versi form/sasaran, validasi rentang | `tests/Feature/Antarmuka/Tracer/PeriodeTest.php`<br>`test_buka_survei_terbuka_memakai_versi_form_tetap` |
| `tracer.periode::tutup`<br>`tracer.periode.tutup` | `PeriodeTracer::tutup`<br>`id,alasan,versi_data` | PA Tracer. Batas request/penutupan eksplisit, respons lama tetap | `tests/Feature/Antarmuka/Tracer/PeriodeTest.php`<br>`test_tutup_periode_tutup_tidak_menghapus_jawaban` |
| `tracer.formulir::simpanDraf`<br>`tracer.formulir.simpanDraf` | `VersiFormulirTracer::simpanDraf`<br>`periode_id,pertanyaan[{id,tipe,label,wajib,opsi?,batas?}],versi_data?` | PA Tracer. Tipe teks/pilihan/tanggal/angka; ID opsi stabil, bukan HTML/SQL | `tests/Feature/Antarmuka/Tracer/FormulirTest.php`<br>`test_simpan_draf_tipe_pertanyaan_di_luar_allowlist_ditolak` |
| `tracer.sasaran::tetapkan`<br>`tracer.sasaran.tetapkan` | `SasaranTracer::tetapkanSnapshot`<br>`periode_id,filter_angkatan_program,profil_ids[],alasan` | PA Tracer. Alumni tanpa akun ikut penyebut, versi snapshot daftar | `tests/Feature/Antarmuka/Tracer/SasaranTest.php`<br>`test_tetapkan_sasaran_tidak_hanya_pemilik_akun` |
| `tracer.sasaran::catatJangkauan`<br>`tracer.sasaran.catatJangkauan` | `SasaranTracer::catatJangkauan`<br>`id,status_jangkauan,alasan?,versi_data` | PA Tracer. Belum_dihubungi/dihubungi/tidak_terjangkau terpisah jawaban/pekerjaan | `tests/Feature/Antarmuka/Tracer/SasaranTest.php`<br>`test_catat_jangkauan_tidak_terjangkau_bukan_menganggur` |
| `tracer.sasaran::revisi`<br>`tracer.sasaran.revisi` | `SasaranTracer::revisiSnapshot`<br>`periode_id,perubahan_ids[],alasan,versi_daftar` | PA Tracer. Daftar versi baru, respons lama tetap satu kanonik | `tests/Feature/Antarmuka/Tracer/SasaranTest.php`<br>`test_revisi_revisi_sasaran_tidak_menggandakan_penyebut` |
| `tracer.jawaban::catatWawancara`<br>`tracer.jawaban.catatWawancara` | `JawabanTracer::catatWawancara`<br>`periode_id,alumni_id,jawaban,referensi_verifikasi,alasan` | PA Tracer sesuai hak input/koreksi. Sumber petugas, alumni tanpa akun terverifikasi, tidak memalsukan login alumni | `tests/Feature/Antarmuka/Tracer/JawabanTest.php`<br>`test_catat_wawancara_wawancara_alumni_tanpa_akun_terlacak` |
| `tracer.jawaban::koreksiSetelahTutup` **hak khusus**<br>`tracer.jawaban.koreksiSetelahTutup` | `JawabanTracer::koreksiTerbatas`<br>`jawaban_id,jawaban,alasan,versi_data` | PA Tracer sesuai hak input/koreksi. Hak khusus, versi form asli dan jawaban lama dipertahankan | `tests/Feature/Antarmuka/Tracer/JawabanTest.php`<br>`test_koreksi_setelah_tutup_koreksi_setelah_tutup_tidak_menimpa` |
| `alumni.survei::simpanDraf`<br>`alumni.survei.simpanDraf` | `JawabanTracer::simpanDraf`<br>`periode_id,jawaban,versi_data?` | AL terverifikasi pemilik/sasaran. Respons satu alumni/periode, belum dihitung menjawab | `tests/Feature/Antarmuka/Alumni/SurveiTest.php`<br>`test_simpan_draf_draf_survei_bukan_respons_final` |
| `alumni.survei::kirim`<br>`alumni.survei.kirim` | `JawabanTracer::kirim`<br>`id,jawaban,versi_data` | AL terverifikasi pemilik/sasaran. Pertanyaan/opsi snapshot form yang dibekukan, validasi wajib | `tests/Feature/Antarmuka/Alumni/SurveiTest.php`<br>`test_kirim_profil_baru_tidak_menimpa_jawaban_lama` |
| `alumni.survei::revisi`<br>`alumni.survei.revisi` | `JawabanTracer::revisiSebelumTutup`<br>`id,jawaban,alasan,versi_data` | AL terverifikasi pemilik/sasaran. Versi baru selama periode terbuka; setelah tutup petugas berizin | `tests/Feature/Antarmuka/Alumni/SurveiTest.php`<br>`test_revisi_alumni_tidak_koreksi_setelah_tutup` |
| `keuangan.tagihan::terbitkan`<br>`keuangan.tagihan.terbitkan` | `Tagihan::terbitkan`<br>`siswa_id,jenis_biaya_id,tahun_ajaran_id,periode,jatuh_tempo?,kunci_idempotensi` | B berizin keuangan; snapshot siswa/alumni historis. Tarif/kelas snapshot server, unik siswa/biaya/periode | `tests/Feature/Antarmuka/Keuangan/TagihanTest.php`<br>`test_terbitkan_naik_lulus_tidak_mengubah_snapshot_tagihan` |
| `keuangan.tagihan::usulkanPembatalan`<br>`keuangan.tagihan.usulkanPembatalan` | `Tagihan::usulkanPembatalan`<br>`id,alasan,versi_data` | B berizin keuangan; snapshot siswa/alumni historis. Menunggu penyelesaian, pembayaran baru dibekukan | `tests/Feature/Antarmuka/Keuangan/TagihanTest.php`<br>`test_usulkan_pembatalan_tagihan_menunggu_tidak_terima_bayar` |
| `keuangan.tagihan::sahkanPembatalan` **hak khusus**<br>`keuangan.tagihan.sahkanPembatalan` | `Tagihan::sahkanPembatalan`<br>`id,alasan,versi_data,kunci_idempotensi` | B hak koreksi/pembatalan. Penerimaan neto nol, penyesuaian pembatalan sampai tagihan efektif nol | `tests/Feature/Antarmuka/Keuangan/TagihanTest.php`<br>`test_sahkan_pembatalan_tagihan_dengan_uang_tidak_langsung_batal` |
| `keuangan.tarif::simpanJenis`<br>`keuangan.tarif.simpanJenis` | `JenisBiaya::simpan`<br>`id?,kode,nama,periode,aktif,versi_data?` | B hak tarif. Master tanpa edit invoice lama | `tests/Feature/Antarmuka/Keuangan/TarifTest.php`<br>`test_simpan_jenis_jenis_biaya_nonaktif_tidak_hapus_tagihan` |
| `keuangan.tarif::tetapkanTarif`<br>`keuangan.tarif.tetapkanTarif` | `TarifBiaya::tetapkan`<br>`jenis_biaya_id,tahun_ajaran_id,kelas_id?,nominal,berlaku_mulai,berlaku_sampai?` | B hak tarif. Versi tarif efektif, jangan update snapshot invoice | `tests/Feature/Antarmuka/Keuangan/TarifTest.php`<br>`test_tetapkan_tarif_tarif_baru_tidak_ubah_tagihan_lama` |
| `keuangan.pembayaran::catat`<br>`keuangan.pembayaran.catat` | `Pembayaran::catat`<br>`tagihan_id,nominal,dibayar_pada,metode,referensi_bukti?,kunci_idempotensi` | B pencatat penerimaan. Nomor kuitansi server, lock tagihan, tidak melebihi sisa | `tests/Feature/Antarmuka/Keuangan/PembayaranTest.php`<br>`test_catat_dua_bayar_paralel_tidak_lebih_bayar` |
| `keuangan.koreksi::ajukanVoid`<br>`keuangan.koreksi.ajukanVoid` | `KoreksiPembayaran::ajukanVoid`<br>`pembayaran_id,uang_tidak_pernah_diterima,alasan,kunci_idempotensi` | B hak ajukan. Hanya catatan salah tanpa uang nyata | `tests/Feature/Antarmuka/Keuangan/KoreksiTest.php`<br>`test_ajukan_void_void_bukan_pengembalian_uang` |
| `keuangan.koreksi::putuskanVoid` **hak khusus**<br>`keuangan.koreksi.putuskanVoid` | `KoreksiPembayaran::putuskan`<br>`id,keputusan,alasan,versi_data` | B hak koreksi. Hak koreksi, lock pembayaran/tagihan, audit saldo | `tests/Feature/Antarmuka/Keuangan/KoreksiTest.php`<br>`test_putuskan_void_void_tidak_menghapus_kuitansi_asal` |
| `keuangan.koreksi::ajukanPenyesuaian`<br>`keuangan.koreksi.ajukanPenyesuaian` | `PenyesuaianTagihan::ajukan`<br>`tagihan_id,delta_nominal,jenis,alasan,kunci_idempotensi` | B hak ajukan. Delta bertanda, bukan edit invoice | `tests/Feature/Antarmuka/Keuangan/KoreksiTest.php`<br>`test_ajukan_penyesuaian_penyesuaian_tidak_edit_nominal_saat_terbit` |
| `keuangan.koreksi::putuskanPenyesuaian` **hak khusus**<br>`keuangan.koreksi.putuskanPenyesuaian` | `PenyesuaianTagihan::putuskan`<br>`id,keputusan,alasan,versi_data` | B hak koreksi. Hitung ulang efektif/neto/sisa, tidak negatif | `tests/Feature/Antarmuka/Keuangan/KoreksiTest.php`<br>`test_putuskan_penyesuaian_penyesuaian_negatif_melebihi_saldo_ditolak` |
| `keuangan.pengembalian::ajukan`<br>`keuangan.pengembalian.ajukan` | `PengembalianPembayaran::ajukan`<br>`pembayaran_id,nominal,tanggal,penerima,metode,referensi_bukti,alasan,kunci_idempotensi` | B hak ajukan. Uang telah dikembalikan manual, belum efektif sampai disahkan | `tests/Feature/Antarmuka/Keuangan/PengembalianTest.php`<br>`test_ajukan_refund_tanpa_bukti_ditolak` |
| `keuangan.pengembalian::putuskan` **hak khusus**<br>`keuangan.pengembalian.putuskan` | `PengembalianPembayaran::putuskan`<br>`id,keputusan,alasan,versi_data` | B hak pengembalian. Nominal <= uang sah belum dikembalikan; status/ledger satu commit | `tests/Feature/Antarmuka/Keuangan/PengembalianTest.php`<br>`test_putuskan_dua_refund_paralel_tidak_melebihi_penerimaan` |
| `keuangan.pengembalian::balik` **hak khusus**<br>`keuangan.pengembalian.balik` | `PengembalianPembayaran::balik`<br>`id,nominal,alasan,referensi_bukti,pulihkan_tagihan_batal,kunci_idempotensi,dasar_pembalik` | B hak pengembalian+koreksi. Pembalik arah bertanda; pada tagihan batal sekaligus pulihkan record/penyesuaian atau tolak semua | `tests/Feature/Antarmuka/Keuangan/PengembalianTest.php`<br>`test_balik_pembalik_tagihan_batal_wajib_pulihkan_atomik` |
| `sarpras.aset::simpan`<br>`sarpras.aset.simpan` | `Aset::simpanUnit`<br>`id?,kode_aset,nama,kategori,ruang_id,penanggung_jawab_id?,tanggal_perolehan?,nilai_perolehan?,versi_data?` | PS unit ditugaskan. Satu ID unit, tidak membuat salinan di Lab | `tests/Feature/Antarmuka/Sarpras/AsetTest.php`<br>`test_simpan_kode_aset_unik_lintas_sarpras_lab` |
| `sarpras.aset::ubahKondisi`<br>`sarpras.aset.ubahKondisi` | `Aset::catatKondisi`<br>`id,kondisi,alasan,versi_data` | PS unit ditugaskan. Kondisi dan ketersediaan terpisah, kejadian tercatat | `tests/Feature/Antarmuka/Sarpras/AsetTest.php`<br>`test_ubah_kondisi_aset_rusak_kembali_tidak_tersedia` |
| `sarpras.aset::pindahkan`<br>`sarpras.aset.pindahkan` | `Aset::pindahkan`<br>`id,ruang_tujuan_id,alasan,versi_data` | PS unit ditugaskan. Lock unit dan hak kedua lokasi; perpindahan immutable | `tests/Feature/Antarmuka/Sarpras/AsetTest.php`<br>`test_pindahkan_pindah_aset_tidak_hilang_histori` |
| `sarpras.peminjaman::ajukan`<br>`sarpras.peminjaman.ajukan` | `PeminjamanAset::ajukan`<br>`aset_ids[],rencana_kembali?,alasan` | G/T/P/PS sesuai hak peminjam. Permohonan tidak otomatis menyerahkan unit | `tests/Feature/Antarmuka/Sarpras/PeminjamanTest.php`<br>`test_ajukan_setuju_pinjam_belum_berarti_diserahkan` |
| `sarpras.peminjaman::putuskan`<br>`sarpras.peminjaman.putuskan` | `PeminjamanAset::putuskan`<br>`id,keputusan,alasan,versi_data` | PS unit. Unit/ruang dalam lingkup petugas | `tests/Feature/Antarmuka/Sarpras/PeminjamanTest.php`<br>`test_putuskan_petugas_unit_lain_tidak_bisa_setujui` |
| `sarpras.peminjaman::serahkan`<br>`sarpras.peminjaman.serahkan` | `PeminjamanAset::serahkanUnit`<br>`id,rincian_ids[],waktu,versi_data` | PS unit. Lock aset urut ID, tidak ada pinjaman aktif lain | `tests/Feature/Antarmuka/Sarpras/PeminjamanTest.php`<br>`test_serahkan_dua_pinjam_unit_sama_tidak_disetujui` |
| `sarpras.peminjaman::batalkan`<br>`sarpras.peminjaman.batalkan` | `PeminjamanAset::batalkanSebelumSerah`<br>`id,alasan,versi_data` | G/pegawai pemohon; PS serah-terima unit lingkup. Hanya unit belum diserahkan, tidak membuat pengembalian palsu | `tests/Feature/Antarmuka/Sarpras/PeminjamanTest.php`<br>`test_batalkan_batal_sebelum_serah_tidak_mencatat_kembali` |
| `sarpras.pengembalian::terima`<br>`sarpras.pengembalian.terima` | `PeminjamanAset::terimaUnit`<br>`id,rincian_ids_dan_kondisi[],waktu,catatan,versi_data` | PS unit. Hanya unit kembali; rusak tidak jadi tersedia, header parsial | `tests/Feature/Antarmuka/Sarpras/PengembalianTest.php`<br>`test_terima_dua_dari_tiga_kembali_satu_tetap_dipinjam` |
| `sarpras.pengembalian::catatKehilangan`<br>`sarpras.pengembalian.catatKehilangan` | `KejadianAset::catatKehilangan`<br>`rincian_id,tanggal,alasan,versi_data` | PS unit. Unit hilang bukan dikembalikan, header belum diselesaikan palsu | `tests/Feature/Antarmuka/Sarpras/PengembalianTest.php`<br>`test_catat_kehilangan_hilang_tidak_menjadi_tersedia` |
| `sarpras.kondisi::selesaikanPerbaikan`<br>`sarpras.kondisi.selesaikanPerbaikan` | `KejadianAset::selesaikanPerbaikan`<br>`id,hasil_penyelesaian,kondisi_akhir,versi_data` | PS unit; hak penghapusan tersendiri. Ketersediaan baru sesuai kondisi dan pinjaman nyata | `tests/Feature/Antarmuka/Sarpras/KondisiTest.php`<br>`test_selesaikan_perbaikan_selesai_perbaikan_bukan_otomatis_tutup_pinjam` |
| `sarpras.kondisi::hapusAdministratif` **hak khusus**<br>`sarpras.kondisi.hapusAdministratif` | `KejadianAset::hapusAdministratif`<br>`aset_id,tanggal,alasan,bukti_keputusan,versi_data` | PS unit; hak penghapusan tersendiri. Status dihapus administratif, ID dan histori tetap | `tests/Feature/Antarmuka/Sarpras/KondisiTest.php`<br>`test_hapus_administratif_hapus_administratif_tidak_delete_aset` |
| `laboratorium.pemesanan::ajukan`<br>`laboratorium.pemesanan.ajukan` | `PemesananRuangLab::ajukan`<br>`ruang_id,kelas_id?,mulai_pada,selesai_pada,alasan,kunci_idempotensi` | G pemohon. Guru pemilik dari sesi; permohonan tidak mengurangi bahan/menyerahkan alat | `tests/Feature/Antarmuka/Laboratorium/PemesananTest.php`<br>`test_ajukan_booking_tidak_otomatis_meminjam_aset` |
| `laboratorium.pemesanan::putuskan`<br>`laboratorium.pemesanan.putuskan` | `PemesananRuangLab::putuskan`<br>`id,keputusan,alasan,versi_data` | PS unit. Lock ruang, periksa recurring jadwal/booking, satu pemakaian bersama | `tests/Feature/Antarmuka/Laboratorium/PemesananTest.php`<br>`test_putuskan_bentrok_jadwal_lab_ditolak_termasuk_nonaktif` |
| `laboratorium.pemesanan::batalkan`<br>`laboratorium.pemesanan.batalkan` | `PemesananRuangLab::batalkan`<br>`id,alasan,versi_data` | G pemohon; PS unit menyetujui. Sebelum dipakai lepas slot, pemakaian bahan yang nyata tetap | `tests/Feature/Antarmuka/Laboratorium/PemesananTest.php`<br>`test_batalkan_batal_booking_tidak_mengembalikan_stok_otomatis` |
| `laboratorium.peralatan::simpanUnit`<br>`laboratorium.peralatan.simpanUnit` | `Aset::simpanUnit`<br>`id?,kode_aset,nama,kategori,ruang_id,versi_data?` | PS unit. Gunakan master aset sama dengan Sarpras | `tests/Feature/Antarmuka/Laboratorium/PeralatanTest.php`<br>`test_simpan_unit_lab_tanpa_sarpras_bisa_kelola_alat` |
| `laboratorium.peralatan::ubahKondisi`<br>`laboratorium.peralatan.ubahKondisi` | `Aset::catatKondisi`<br>`id,kondisi,alasan,versi_data` | PS unit. Kondisi sama langsung terbaca Sarpras | `tests/Feature/Antarmuka/Laboratorium/PeralatanTest.php`<br>`test_ubah_kondisi_perubahan_lab_terlihat_pada_aset_sama` |
| `laboratorium.peralatan::ajukanPinjam`<br>`laboratorium.peralatan.ajukanPinjam` | `PeminjamanAset::ajukan`<br>`aset_ids[],rencana_kembali?,alasan` | G pemohon. Header/rincian sama, bukan pinjaman Lab paralel | `tests/Feature/Antarmuka/Laboratorium/PeralatanTest.php`<br>`test_ajukan_pinjam_lab_tidak_membuat_salinan_pinjaman` |
| `laboratorium.peralatan::putuskanPinjam`<br>`laboratorium.peralatan.putuskanPinjam` | `PeminjamanAset::putuskan`<br>`id,keputusan,alasan,versi_data` | PS unit. Lisensi penulis Lab, hak unit tetap dicek | `tests/Feature/Antarmuka/Laboratorium/PeralatanTest.php`<br>`test_putuskan_pinjam_lab_mandiri_menyetujui_pinjam` |
| `laboratorium.peralatan::serahkanUnit`<br>`laboratorium.peralatan.serahkanUnit` | `PeminjamanAset::serahkanUnit`<br>`id,rincian_ids[],waktu,versi_data` | PS unit. Lock unit yang sama | `tests/Feature/Antarmuka/Laboratorium/PeralatanTest.php`<br>`test_serahkan_unit_pinjam_lab_sarpras_tidak_ganda` |
| `laboratorium.peralatan::terimaUnit`<br>`laboratorium.peralatan.terimaUnit` | `PeminjamanAset::terimaUnit`<br>`id,rincian_ids_dan_kondisi[],waktu,catatan,versi_data` | PS unit. Parsial/rusak/hilang memakai invariant bersama | `tests/Feature/Antarmuka/Laboratorium/PeralatanTest.php`<br>`test_terima_unit_lab_mandiri_bisa_terima_kembali` |
| `laboratorium.bahan::simpanMaster`<br>`laboratorium.bahan.simpanMaster` | `BahanHabisPakai::simpan`<br>`id?,kode,nama,satuan,aktif,versi_data?` | PS unit. Master tanpa field saldo yang dapat diedit | `tests/Feature/Antarmuka/Laboratorium/BahanTest.php`<br>`test_simpan_master_edit_bahan_tidak_mengubah_stok` |
| `laboratorium.mutasi::catat`<br>`laboratorium.mutasi.catat` | `MutasiStokLab::catat`<br>`bahan_id,ruang_id,jenis,jumlah,alasan,kunci_idempotensi` | PS unit, pengesahan penyesuaian hak khusus. Delta server untuk masuk/pakai, lock saldo, tidak minus | `tests/Feature/Antarmuka/Laboratorium/MutasiStokTest.php`<br>`test_catat_dua_pemakaian_tidak_stok_minus` |
| `laboratorium.mutasi::ajukanPenyesuaian`<br>`laboratorium.mutasi.ajukanPenyesuaian` | `PenyesuaianStokLab::ajukan`<br>`bahan_id,ruang_id,jumlah_fisik,alasan` | PS unit, pengesahan penyesuaian hak khusus. Saldo tercatat server, simpan hasil fisik | `tests/Feature/Antarmuka/Laboratorium/MutasiStokTest.php`<br>`test_ajukan_penyesuaian_penyesuaian_bukan_edit_master` |
| `laboratorium.mutasi::sahkanPenyesuaian` **hak khusus**<br>`laboratorium.mutasi.sahkanPenyesuaian` | `PenyesuaianStokLab::sahkan`<br>`id,versi_data` | PS unit, pengesahan penyesuaian hak khusus. Hitung ulang selisih, satu mutasi, audit | `tests/Feature/Antarmuka/Laboratorium/MutasiStokTest.php`<br>`test_sahkan_penyesuaian_dua_pengesahan_tidak_ganda` |
| `laboratorium.mutasi::balikMutasi`<br>`laboratorium.mutasi.balikMutasi` | `MutasiStokLab::balik`<br>`id,jumlah,alasan,kunci_idempotensi` | PS unit, pengesahan penyesuaian hak khusus. Record pembalik baru <= asal belum dibalik, saldo tetap sah | `tests/Feature/Antarmuka/Laboratorium/MutasiStokTest.php`<br>`test_balik_mutasi_pembalik_stok_tidak_edit_record_asal` |
| `laboratorium.transfer::pindahkan`<br>`laboratorium.transfer.pindahkan` | `PerpindahanBahanLab::pindahkan`<br>`bahan_id,ruang_asal_id,ruang_tujuan_id,jumlah,alasan,kunci_idempotensi` | PS berhak kedua lokasi. Keluar/masuk dan saldo kedua sisi satu commit, urut lock | `tests/Feature/Antarmuka/Laboratorium/TransferTest.php`<br>`test_pindahkan_gagal_satu_sisi_rollback_transfer` |
| `laboratorium.transfer::balikTransfer`<br>`laboratorium.transfer.balikTransfer` | `PerpindahanBahanLab::balik`<br>`id,jumlah,alasan,kunci_idempotensi` | PS berhak kedua lokasi. Pasangan pembalik, tidak melebihi transfer tersisa | `tests/Feature/Antarmuka/Laboratorium/TransferTest.php`<br>`test_balik_transfer_transfer_retry_dan_pembalik_tidak_ganda` |
| `supervisi.penugasan::tetapkan`<br>`supervisi.penugasan.tetapkan` | `Supervisi::tetapkan`<br>`guru_id,supervisor_id,versi_instrumen_id,dijadwalkan_pada` | P kurikulum. Supervisor tidak menilai dirinya sendiri, instrumen sah | `tests/Feature/Antarmuka/Supervisi/PenugasanTest.php`<br>`test_tetapkan_penugasan_supervisor_terbatas_guru` |
| `supervisi.penugasan::gantiSupervisor`<br>`supervisi.penugasan.gantiSupervisor` | `Supervisi::gantiSupervisor`<br>`id,supervisor_baru_id,alasan,versi_data` | P kurikulum. Tutup penugasan lama dan simpan riwayat | `tests/Feature/Antarmuka/Supervisi/PenugasanTest.php`<br>`test_ganti_supervisor_supervisor_lama_kehilangan_akses` |
| `supervisi.penugasan::batalkan`<br>`supervisi.penugasan.batalkan` | `Supervisi::batalkan`<br>`id,alasan,versi_data` | P kurikulum. Hasil historis tetap, status batal tercatat | `tests/Feature/Antarmuka/Supervisi/PenugasanTest.php`<br>`test_batalkan_batal_supervisi_tidak_hapus_nilai` |
| `supervisi.instrumen::simpanDraf`<br>`supervisi.instrumen.simpanDraf` | `VersiInstrumenSupervisi::simpanDraf`<br>`instrumen_id?,kode,nama,indikator_rubrik[{id,label,min,max,bobot}],versi_data?` | P kurikulum hak instrumen. ID indikator stabil; rentang nilai/bobot valid | `tests/Feature/Antarmuka/Supervisi/InstrumenTest.php`<br>`test_simpan_draf_rubrik_invalid_ditolak` |
| `supervisi.instrumen::sahkan`<br>`supervisi.instrumen.sahkan` | `VersiInstrumenSupervisi::sahkan`<br>`id,versi_data` | P kurikulum hak instrumen. Terpakai menjadi immutable; perubahan berikut versi baru | `tests/Feature/Antarmuka/Supervisi/InstrumenTest.php`<br>`test_sahkan_rubrik_baru_tidak_ubah_nilai_lama` |
| `supervisi.observasi::simpanDraf`<br>`supervisi.observasi.simpanDraf` | `VersiHasilSupervisi::simpanDraf`<br>`supervisi_id,catatan_observasi,nilai_per_indikator[],umpan_balik?,versi_data?` | G supervisor/P berizin pada guru ditugaskan. Rubrik snapshot, nilai/total dihitung server | `tests/Feature/Antarmuka/Supervisi/ObservasiTest.php`<br>`test_simpan_draf_guru_tidak_bisa_mengubah_nilai_sendiri` |
| `supervisi.observasi::sahkan`<br>`supervisi.observasi.sahkan` | `VersiHasilSupervisi::sahkan`<br>`id,versi_data` | G supervisor/P berizin pada guru ditugaskan. Simpan versi sah dengan reviewer | `tests/Feature/Antarmuka/Supervisi/ObservasiTest.php`<br>`test_sahkan_hasil_sah_terikat_instrumen_asli` |
| `supervisi.observasi::buatRevisi`<br>`supervisi.observasi.buatRevisi` | `VersiHasilSupervisi::buatRevisi`<br>`id,alasan,versi_data` | G supervisor/P berizin pada guru ditugaskan. Versi baru tanpa timpa hasil lama | `tests/Feature/Antarmuka/Supervisi/ObservasiTest.php`<br>`test_buat_revisi_revisi_hasil_menjaga_nilai_lama` |
| `supervisi.tindak_lanjut::tetapkan`<br>`supervisi.tindak_lanjut.tetapkan` | `TindakLanjutSupervisi::tetapkan`<br>`supervisi_id,tindakan,target_selesai?` | G supervisor. Rencana tindak lanjut, bukan selesai otomatis | `tests/Feature/Antarmuka/Supervisi/TindakLanjutTest.php`<br>`test_tetapkan_supervisi_belum_selesai_tanpa_verifikasi` |
| `supervisi.tindak_lanjut::isiHasil`<br>`supervisi.tindak_lanjut.isiHasil` | `TindakLanjutSupervisi::isiHasilGuru`<br>`id,hasil_guru,versi_data` | G guru pemilik. Guru hanya miliknya, tidak mengubah skor | `tests/Feature/Antarmuka/Supervisi/TindakLanjutTest.php`<br>`test_isi_hasil_hasil_tindak_lanjut_bukan_edit_nilai` |
| `supervisi.tindak_lanjut::verifikasi`<br>`supervisi.tindak_lanjut.verifikasi` | `TindakLanjutSupervisi::verifikasi`<br>`id,keputusan,catatan,versi_data` | G supervisor. Supervisor memastikan hasil sebelum selesai | `tests/Feature/Antarmuka/Supervisi/TindakLanjutTest.php`<br>`test_verifikasi_tindak_lanjut_harus_diverifikasi` |
| `supervisi.tindak_lanjut::selesaikan`<br>`supervisi.tindak_lanjut.selesaikan` | `Supervisi::selesaikan`<br>`supervisi_id,versi_data` | G supervisor/P berizin. Semua tindak lanjut wajib tuntas/sah, hasil final ada | `tests/Feature/Antarmuka/Supervisi/TindakLanjutTest.php`<br>`test_selesaikan_supervisi_tidak_selesai_dengan_tindak_lanjut_terbuka` |
| `laboratorium.peralatan::batalkanPinjam`<br>`laboratorium.peralatan.batalkanPinjam` | `PeminjamanAset::batalkanSebelumSerah`<br>`id,alasan,versi_data` | PS unit/G peminjam; Lab aktif cukup tanpa Sarpras. Lab aktif cukup, tidak membuat pengembalian palsu | `tests/Feature/Antarmuka/Laboratorium/PeralatanTest.php`<br>`test_batalkan_pinjam_lab_mandiri_bisa_batalkan_sebelum_serah` |
| `laboratorium.peralatan::catatKehilangan`<br>`laboratorium.peralatan.catatKehilangan` | `KejadianAset::catatKehilangan`<br>`rincian_id,tanggal,alasan,versi_data` | PS unit. Unit hilang bukan tersedia, histori pinjaman tetap | `tests/Feature/Antarmuka/Laboratorium/PeralatanTest.php`<br>`test_catat_kehilangan_lab_hilang_tidak_menjadi_kembali` |
| `laboratorium.peralatan::selesaikanPerbaikan`<br>`laboratorium.peralatan.selesaikanPerbaikan` | `KejadianAset::selesaikanPerbaikan`<br>`id,hasil_penyelesaian,kondisi_akhir,versi_data` | PS unit. Perbaikan pada unit yang sama, tidak otomatis menutup pinjaman | `tests/Feature/Antarmuka/Laboratorium/PeralatanTest.php`<br>`test_selesaikan_perbaikan_lab_mandiri_menyelesaikan_perbaikan` |
| `sarpras.kondisi::selesaikanKehilangan`<br>`sarpras.kondisi.selesaikanKehilangan` | `KejadianAset::selesaikanKehilangan`<br>`id,jenis_penyelesaian,hasil_penyelesaian,versi_data` | PS unit; hak penghapusan tersendiri. Ditemukan/ganti rugi/penghapusan berizin; tidak transaksi uang otomatis | `tests/Feature/Antarmuka/Sarpras/KondisiTest.php`<br>`test_selesaikan_kehilangan_ganti_unit_memakai_id_baru_histori_lama_tetap` |
| `pelanggaran.pembinaan::rujukKonseling`<br>`pelanggaran.pembinaan.rujukKonseling` | `PermintaanKonseling::ajukanDariPelanggaran`<br>`tindak_lanjut_pelanggaran_id,ringkasan_aman` | W/BK/P kesiswaan lingkup. Hanya bila BK aktif; pembinaan dasar tetap dapat selesai tanpa BK | `tests/Feature/Antarmuka/Pelanggaran/PembinaanTest.php`<br>`test_rujuk_konseling_rujukan_pelanggaran_tidak_membuka_isi_bk` |
| `akademik.kelulusan::koreksi`<br>`akademik.kelulusan.koreksi` | `Kelulusan::koreksiKeputusan`<br>`kelulusan_id,jenis_koreksi,tanggal_lulus?,referensi_keputusan?,alasan,hasil_pemeriksaan_relasi[],versi_data` | AS akademik + petugas bidang untuk konflik data privat. Hak koreksi khusus; histori profil/alumni/survei/tagihan dipertahankan, tidak menghapus jejak lulus | `tests/Feature/Antarmuka/Akademik/KelulusanTest.php`<br>`test_koreksi_koreksi_kelulusan_tidak_menghapus_data_terkait` |
| `inti.penugasan::berikanHakKhusus`<br>`inti.penugasan.berikanHakKhusus` | `HakKhususPengguna::berikan`<br>`pengguna_id,hak_akses_id,berlaku_mulai,berlaku_sampai?,alasan` | AS dengan hak lingkup yang boleh diberikan. Hak yang ditandai khusus saja; role minimum/lingkup tetap, tidak owner dan tidak peningkatan diri sendiri | `tests/Feature/Antarmuka/Inti/PenugasanTest.php`<br>`test_berikan_hak_khusus_bendahara_tanpa_hak_khusus_tidak_bisa_refund` |
| `inti.penugasan::cabutHakKhusus`<br>`inti.penugasan.cabutHakKhusus` | `HakKhususPengguna::cabut`<br>`id,alasan,versi_data` | AS dengan hak lingkup yang boleh diberikan. Kunci pengguna penerima, berlaku mulai request berikutnya | `tests/Feature/Antarmuka/Inti/PenugasanTest.php`<br>`test_cabut_hak_khusus_hak_refund_dicabut_tidak_bisa_dipakai_tab_lama` |

Setiap metode test row T4 menegaskan happy path, field tidak sah, role/objek salah, status awal salah, konflik versi serta tidak ada partial commit bila relevan. Table historis/versi/pivot tidak memperoleh CRUD mandiri di luar aksi di atas. Konten yang sudah sah/terkirim dibaca melalui parent/riwayat, bukan endpoint edit generik.

<a id="route-http"></a>

### T5. Route HTTP, transport dan keluaran

Nama, metode, URI dan handler dikunci. Handler vendor tetap milik dependency, tidak disalin ke source lokal. Handler lokal berada `app/Http/Controllers/`; method pada nama handler adalah method tepat yang akan dibuat. Semua mutasi web memakai CSRF; GET tidak mengubah data bisnis/lease. File test adalah rencana, bukan bukti hasil. Setiap route memakai `test_<nama_case>_<metode>_<nomor-row>` untuk memastikan nama test unik. Row fallback modul harus didaftarkan terakhir, sedangkan arsip berkas/ekspor sebelum detail generik.

| Metode / URL / nama / rilis | File registrasi / handler | Akses / input | Respons / file test / kasus |
|---|---|---|---|
| `GET /login`<br>`login` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\AuthenticatedSessionController@create` | Publik guest<br>`-` | views/auth/login.blade.php<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_form_login_indonesia_get_001` |
| `POST /login`<br>`login.store` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\AuthenticatedSessionController@store` | Guest + CSRF + throttle login<br>`email,password,remember?` | Session regenerate; challenge TOTP owner<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_login_rate_limit_dan_akun_nonaktif_post_002` |
| `POST /logout`<br>`logout` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\AuthenticatedSessionController@destroy` | Auth + CSRF<br>`-` | Invalidate session/token, redirect login<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_logout_bukan_get_post_003` |
| `GET /forgot-password`<br>`password.request` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\PasswordResetLinkController@create` | Guest<br>`-` | views/auth/lupa-sandi.blade.php<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_form_reset_indonesia_get_004` |
| `POST /forgot-password`<br>`password.email` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\PasswordResetLinkController@store` | Guest + CSRF + throttle reset<br>`email` | Pesan generik; broker/mail sinkron<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_reset_tidak_enumerasi_akun_post_005` |
| `GET /reset-password/{token}`<br>`password.reset` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\NewPasswordController@create` | Guest<br>`email,token` | views/auth/reset-sandi.blade.php<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_reset_token_dibatasi_get_006` |
| `POST /reset-password`<br>`password.update` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\NewPasswordController@store` | Guest + CSRF<br>`email,token,password,password_confirmation` | Broker konsumsi token, hash sandi<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_token_reset_tidak_bisa_dipakai_ulang_post_007` |
| `GET /two-factor-challenge`<br>`two-factor.login` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\TwoFactorAuthenticatedSessionController@create` | Guest dengan challenge session<br>`-` | views/auth/tantangan-mfa.blade.php<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_challenge_tanpa_session_ditolak_get_008` |
| `POST /two-factor-challenge`<br>`two-factor.login.store` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\TwoFactorAuthenticatedSessionController@store` | Challenge + CSRF + throttle MFA<br>`code XOR recovery_code` | Recovery single-use, session sah<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_recovery_code_tidak_dapat_dipakai_ulang_post_009` |
| `GET /user/confirm-password`<br>`password.confirm` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\ConfirmablePasswordController@show` | Auth akun aktif<br>`-` | views/auth/konfirmasi-sandi.blade.php<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_konfirmasi_sandi_milik_sendiri_get_010` |
| `POST /user/confirm-password`<br>`password.confirm.store` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\ConfirmablePasswordController@store` | Auth + CSRF + throttle MFA<br>`password` | Timestamp konfirmasi session<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_sandi_salah_tidak_membuka_mfa_post_011` |
| `PUT /user/password`<br>`user-password.update` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\PasswordController@update` | Auth aktif + CSRF<br>`current_password,password,password_confirmation` | Hash baru, sesi lain dicabut<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_ganti_sandi_memerlukan_sandi_lama_put_012` |
| `POST /user/two-factor-authentication`<br>`two-factor.enable` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\TwoFactorAuthenticationController@store` | Auth O + password.confirm + CSRF<br>`-` | Fortify generate secret terenkripsi; belum memberi akses owner<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_mfa_belum_confirm_owner_ditolak_post_013` |
| `POST /user/confirmed-two-factor-authentication`<br>`two-factor.confirm` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\ConfirmedTwoFactorAuthenticationController@store` | Auth O + password.confirm + CSRF + throttle MFA<br>`code` | Konfirmasi TOTP lalu panel owner terbuka<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_mfa_confirm_baru_boleh_panel_owner_post_014` |
| `GET /user/two-factor-qr-code`<br>`two-factor.qr-code` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\TwoFactorQrCodeController@show` | Auth O + password.confirm<br>`-` | QR standar Fortify, no-store, bukan log<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_qr_hanya_owner_milik_sendiri_get_015` |
| `GET /user/two-factor-recovery-codes`<br>`two-factor.recovery-codes` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\RecoveryCodeController@index` | Auth O + password.confirm<br>`-` | Recovery no-store, dilarang log<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_recovery_tidak_boleh_dibaca_admin_get_016` |
| `POST /user/two-factor-recovery-codes`<br>`two-factor.regenerate-recovery-codes` / R1 | `routes/autentikasi.php`<br>`Laravel\Fortify\Http\Controllers\RecoveryCodeController@store` | Auth O + password.confirm + CSRF<br>`-` | Ganti daftar recovery dengan primitive resmi<br>`tests/Feature/Autentikasi/RuteFortifyTest.php`<br>`test_recovery_lama_invalid_setelah_regenerasi_post_017` |
| `POST /livewire/update`<br>`livewire.update` / R1 | `app/Providers/AppServiceProvider.php`<br>`Livewire\Mechanisms\HandleRequests\HandleRequests@handleUpdate` | web + auth + pengguna aktif + RequireLivewireHeaders; tiap action periksa ulang hak/lingkup/lisensi<br>`Snapshot signed + action + whitelist props` | 422 validasi / 403 hak / 409 versi / bukti commit<br>`tests/Feature/Otorisasi/TransportLivewireTest.php`<br>`test_seluruh_action_ditolak_bila_tanpa_hak_post_018` |
| `GET /`<br>`publik.beranda` / R1 | `routes/inti.php`<br>`HalamanPublikController@index` | Publik; baca CMS hanya bila siap+aktif<br>`-` | views/publik/informasi-sekolah.blade.php atau tema publik<br>`tests/Feature/Http/HalamanPublikControllerTest.php`<br>`test_website_nonaktif_admin_tetap_jalan_get_019` |
| `GET /modul-belum-tersedia/{kode_modul}`<br>`inti.modul_belum_tersedia` / R1 | `routes/inti.php`<br>`ModulBelumTersediaController@show` | Auth; kode katalog allowlist, tanpa query model modul<br>`kode_modul` | 404/403 terkontrol, bukan fitur kosong<br>`tests/Feature/Http/ModulBelumTersediaControllerTest.php`<br>`test_kode_modul_arbitrer_ditolak_get_020` |
| `POST /berkas/unggah`<br>`berkas.unggah` / R1 | `routes/inti.php`<br>`BerkasController@store` | Auth + aktif + hak buat tujuan/lingkup/lisensi asal<br>`file,tujuan_allowlist,objek_id?` | ID metadata privat sementara milik pengunggah, tidak path publik<br>`tests/Feature/Http/BerkasControllerTest.php`<br>`test_unggah_jenis_dan_ukuran_invalid_ditolak_post_021` |
| `GET /berkas/{berkas}/unduh`<br>`berkas.unduh` / R1 | `routes/inti.php`<br>`BerkasController@download` | Auth + objek pemilik + hak unduh + lisensi asal; arsip melalui jalur Core<br>`berkas` | Stream no-store, filename aman<br>`tests/Feature/Http/BerkasControllerTest.php`<br>`test_id_file_bukan_otorisasi_get_022` |
| `GET /admin/bukti-presensi/{token}`<br>`inti.presensi.bukti` / R1 | `routes/inti.php`<br>`BuktiPresensiController@show` | Auth pembuat/petugas inspeksi dan lingkup kelas, tanpa module-aktif<br>`token` | Bukti commit lengkap atau belum ditemukan, tidak retry<br>`tests/Feature/Http/BuktiPresensiControllerTest.php`<br>`test_bukti_tetap_terbaca_saat_modul_nonaktif_get_023` |
| `POST /admin/proses/pratinjau`<br>`inti.proses.pratinjau` / R1 | `routes/inti.php`<br>`OperasiMassalController@preview` | Auth + pembuat/petugas berizin jenis; lisensi berdasarkan jenis operasi<br>`jenis,parameter,pilihan,berkas_id?` | Snapshot valid/tidak valid tanpa mutasi bisnis<br>`tests/Feature/Http/OperasiMassalControllerTest.php`<br>`test_pratinjau_tidak_menyimpan_data_bisnis_post_024` |
| `POST /admin/proses`<br>`inti.proses.mulai` / R1 | `routes/inti.php`<br>`OperasiMassalController@start` | Auth + pembuat/petugas berizin jenis; lisensi berdasarkan jenis operasi<br>`token_pratinjau,versi,kunci_idempotensi` | Operasi UUID dan item snapshot<br>`tests/Feature/Http/OperasiMassalControllerTest.php`<br>`test_snapshot_target_tidak_berubah_karena_filter_post_025` |
| `POST /admin/proses/{operasi}/batch`<br>`inti.proses.batch` / R1 | `routes/inti.php`<br>`OperasiMassalController@process` | Auth + pembuat/petugas berizin jenis; lisensi berdasarkan jenis operasi<br>`batas_item<=25,token_permintaan` | Terminal item dari commit, bukan timer<br>`tests/Feature/Http/OperasiMassalControllerTest.php`<br>`test_klaim_dua_browser_tidak_memproses_item_ganda_post_026` |
| `POST /admin/proses/{operasi}/jeda`<br>`inti.proses.jeda` / R1 | `routes/inti.php`<br>`OperasiMassalController@pause` | Auth + pembuat/petugas berizin jenis; lisensi berdasarkan jenis operasi<br>`versi_data` | Stop klaim baru; in-flight boleh selesai<br>`tests/Feature/Http/OperasiMassalControllerTest.php`<br>`test_jeda_tidak_membatalkan_commit_post_027` |
| `POST /admin/proses/{operasi}/lanjutkan`<br>`inti.proses.lanjutkan` / R1 | `routes/inti.php`<br>`OperasiMassalController@resume` | Auth + pembuat/petugas berizin jenis; lisensi berdasarkan jenis operasi<br>`versi_data` | Periksa/recover lease aman, lanjut snapshot lama<br>`tests/Feature/Http/OperasiMassalControllerTest.php`<br>`test_lanjut_setelah_timeout_tidak_duplikasi_post_028` |
| `POST /admin/proses/{operasi}/ulangi-gagal`<br>`inti.proses.ulangi_gagal` / R1 | `routes/inti.php`<br>`OperasiMassalController@retry` | Auth + pembuat/petugas berizin jenis; lisensi berdasarkan jenis operasi<br>`item_ids_gagal[],alasan,versi_data` | Reset hanya item eligible, putaran baru<br>`tests/Feature/Http/OperasiMassalControllerTest.php`<br>`test_retry_tidak_mengulang_item_berhasil_post_029` |
| `GET /admin/proses/{operasi}/status`<br>`inti.proses.status` / R1 | `routes/inti.php`<br>`StatusOperasiMassalController@show` | Auth + pembuat/inspektur berizin, tanpa mutasi atau module-aktif<br>`operasi` | Jumlah terminal DB, lease hanya dibaca<br>`tests/Feature/Http/StatusOperasiMassalControllerTest.php`<br>`test_get_status_tidak_recovery_lease_get_030` |
| `GET /admin/impor/{jenis}/template`<br>`inti.impor.template` / R1 | `routes/inti.php`<br>`ImporController@template` | Auth + hak impor jenis allowlist; alumni hanya R3 dan BKK/Tracer<br>`jenis` | XLSX/CSV template tanpa data nyata<br>`tests/Feature/Http/ImporControllerTest.php`<br>`test_template_jenis_arbitrer_ditolak_get_031` |
| `GET /admin/arsip/{kode_modul}`<br>`arsip.index` / R1 | `routes/inti.php`<br>`ArsipController@index` | Auth + arsip.<kode>.lihat/ekspor + objek/kolom + pernah aktif/reader kompatibel; tanpa module-aktif<br>`filter,tanggal_mulai?,tanggal_selesai?,page` | Daftar proyeksi diizinkan<br>`tests/Feature/Http/ArsipControllerTest.php`<br>`test_arsip_modul_belum_pernah_aktif_ditolak_get_032` |
| `POST /admin/arsip/{kode_modul}/ekspor`<br>`arsip.ekspor` / R1 | `routes/inti.php`<br>`ArsipController@export` | Auth + arsip.<kode>.lihat/ekspor + objek/kolom + pernah aktif/reader kompatibel; tanpa module-aktif<br>`laporan,format,filter,kolom_allowlist` | CSV/XLSX/PDF sinkron berizin; audit tanpa data isi<br>`tests/Feature/Http/ArsipControllerTest.php`<br>`test_ekspor_tidak_bocor_kolom_privat_post_033` |
| `GET /admin/arsip/{kode_modul}/berkas/{berkas}/unduh`<br>`arsip.berkas.unduh` / R1 | `routes/inti.php`<br>`ArsipController@download` | Auth + arsip.<kode>.lihat/ekspor + objek/kolom + pernah aktif/reader kompatibel; tanpa module-aktif<br>`berkas` | Stream file privat, kelayakan dan objek tetap wajib<br>`tests/Feature/Http/ArsipControllerTest.php`<br>`test_pegawai_dicabut_tidak_bisa_unduh_arsip_get_034` |
| `GET /admin/arsip/{kode_modul}/{jenis}/{id}`<br>`arsip.show` / R1 | `routes/inti.php`<br>`ArsipController@show` | Auth + arsip.<kode>.lihat/ekspor + objek/kolom + pernah aktif/reader kompatibel; tanpa module-aktif<br>`jenis,id,format?` | Detail/versi/kuitansi sesuai reader typed<br>`tests/Feature/Http/ArsipControllerTest.php`<br>`test_jenis_resource_tidak_bisa_nama_model_bebas_get_035` |
| `GET /halaman/{slug}`<br>`publik.halaman` / R2 | `routes/modul/website.php`<br>`HalamanPublikController@halaman` | Publik, website siap+aktif; hanya versi/album terbit<br>`slug` | tema/<kode>/halaman.blade.php<br>`tests/Feature/Http/HalamanPublikControllerTest.php`<br>`test_publik_tidak_bisa_membaca_draf_get_036` |
| `GET /berita`<br>`publik.berita` / R2 | `routes/modul/website.php`<br>`HalamanPublikController@berita` | Publik, website siap+aktif; hanya versi/album terbit<br>`kategori?,page` | tema/<kode>/berita.blade.php<br>`tests/Feature/Http/HalamanPublikControllerTest.php`<br>`test_publik_tidak_bisa_membaca_draf_get_037` |
| `GET /berita/{slug}`<br>`publik.berita.detail` / R2 | `routes/modul/website.php`<br>`HalamanPublikController@artikel` | Publik, website siap+aktif; hanya versi/album terbit<br>`slug` | tema/<kode>/artikel.blade.php<br>`tests/Feature/Http/HalamanPublikControllerTest.php`<br>`test_publik_tidak_bisa_membaca_draf_get_038` |
| `GET /pengumuman`<br>`publik.pengumuman` / R2 | `routes/modul/website.php`<br>`HalamanPublikController@pengumuman` | Publik, website siap+aktif; hanya versi/album terbit<br>`page` | tema/<kode>/pengumuman.blade.php<br>`tests/Feature/Http/HalamanPublikControllerTest.php`<br>`test_publik_tidak_bisa_membaca_draf_get_039` |
| `GET /galeri`<br>`publik.galeri` / R2 | `routes/modul/website.php`<br>`HalamanPublikController@galeri` | Publik, website siap+aktif; hanya versi/album terbit<br>`page` | tema/<kode>/galeri.blade.php<br>`tests/Feature/Http/HalamanPublikControllerTest.php`<br>`test_publik_tidak_bisa_membaca_draf_get_040` |
| `GET /galeri/{slug}`<br>`publik.galeri.detail` / R2 | `routes/modul/website.php`<br>`HalamanPublikController@album` | Publik, website siap+aktif; hanya versi/album terbit<br>`slug` | tema/<kode>/album.blade.php<br>`tests/Feature/Http/HalamanPublikControllerTest.php`<br>`test_publik_tidak_bisa_membaca_draf_get_041` |
| `GET /admin/website/pratinjau/{jenis}/{id}`<br>`website.pratinjau` / R2 | `routes/modul/website.php`<br>`PratinjauWebsiteController@show` | E hak konten/objek; website aktif<br>`jenis allowlist,id,versi` | Tema preview noindex/no-store, bukan URL publik<br>`tests/Feature/Http/PratinjauWebsiteControllerTest.php`<br>`test_preview_draf_memerlukan_login_get_042` |
| `POST /admin/website/media/unggah`<br>`website.media.unggah` / R2 | `routes/modul/website.php`<br>`MediaWebsiteController@store` | E hak media + website aktif + CSRF<br>`file` | ID media publik tervalidasi<br>`tests/Feature/Http/MediaWebsiteControllerTest.php`<br>`test_unggahan_siswa_privat_bukan_media_publik_post_043` |
| `GET /alumni/daftar`<br>`alumni.daftar` / R3 | `routes/alumni.php`<br>`RegistrasiAlumniController@create` | Guest; BKK atau Tracer aktif<br>`-` | views/auth/daftar-alumni.blade.php<br>`tests/Feature/Http/RegistrasiAlumniControllerTest.php`<br>`test_portal_terbuka_dengan_satu_modul_alumni_get_044` |
| `POST /alumni/daftar`<br>`alumni.daftar.store` / R3 | `routes/alumni.php`<br>`RegistrasiAlumniController@store` | Guest + CSRF + throttle registrasi; BKK atau Tracer<br>`nama,email,password,password_confirmation,tahun_lulus,program_klaim,nis_klaim?,kontak` | User+klaim pending atomik, tanpa akses profil resmi sebelum verifikasi<br>`tests/Feature/Http/RegistrasiAlumniControllerTest.php`<br>`test_registrasi_duplikat_tidak_akun_ganda_post_045` |
| `POST /siswa/pkl/lokasi-token`<br>`siswa.pkl.lokasi_token` / R3 | `routes/modul/pkl.php`<br>`Pkl/TokenLokasiController@store` | Auth S pemilik penempatan + PKL aktif + CSRF + throttle + tanggal server<br>`penempatan_id,jenis` | Token 60 detik, versi lokasi/radius/akurasi; no koordinat tersimpan di browser<br>`tests/Feature/Http/Pkl/TokenLokasiControllerTest.php`<br>`test_token_terikat_pemilik_tindakan_dan_versi_post_046` |
| `POST /siswa/pkl/masuk`<br>`siswa.pkl.masuk` / R3 | `routes/modul/pkl.php`<br>`Pkl/PresensiController@masuk` | Auth S pemilik penempatan + PKL aktif + CSRF + throttle + tanggal server<br>`token,kunci_idempotensi,lintang,bujur,akurasi,captured_at` | Validasi server, satu event masuk + bukti commit<br>`tests/Feature/Http/Pkl/PresensiControllerTest.php`<br>`test_jarak_ditambah_akurasi_di_dalam_radius_post_047` |
| `POST /siswa/pkl/pulang`<br>`siswa.pkl.pulang` / R3 | `routes/modul/pkl.php`<br>`Pkl/PresensiController@pulang` | Auth S pemilik penempatan + PKL aktif + CSRF + throttle + tanggal server<br>`token,kunci_idempotensi,lintang,bujur,akurasi,captured_at` | Masuk dahulu, satu terbuka, tanggal kerja masuk termasuk tengah malam<br>`tests/Feature/Http/Pkl/PresensiControllerTest.php`<br>`test_pulang_tengah_malam_tetap_hari_masuk_post_048` |
| `POST /siswa/pkl/jurnal`<br>`siswa.pkl.jurnal.store` / R3 | `routes/modul/pkl.php`<br>`Pkl/JurnalController@store` | Auth S pemilik/segmen + PKL aktif + CSRF; tanggal jurnal/kejadian dalam rentang dan tidak masa depan<br>`id?,tanggal_kerja,rincian[{kegiatan,hasil,kendala?,tindak_lanjut?}],berkas_id?,aksi=draf/kirim/revisi,alasan_revisi?,versi_data?` | Jurnal satu hari/segmen, versi terkirim utuh, tanpa GPS wajib<br>`tests/Feature/Http/Pkl/JurnalControllerTest.php`<br>`test_jurnal_dapat_dikirim_saat_gps_gagal_post_049` |
| `POST /siswa/pkl/bantuan`<br>`siswa.pkl.bantuan.store` / R3 | `routes/modul/pkl.php`<br>`Pkl/BantuanController@store` | Auth S pemilik/segmen + PKL aktif + CSRF; tanggal jurnal/kejadian dalam rentang dan tidak masa depan<br>`penempatan_id,tanggal_kejadian,jenis_kendala,keterangan` | Permintaan saja, bukan hadir otomatis<br>`tests/Feature/Http/Pkl/BantuanControllerTest.php`<br>`test_bantuan_tidak_bypass_radius_post_050` |
| `GET /siswa/bukti-pkl/{token}`<br>`inti.pkl.bukti` / R3 | `routes/inti.php`<br>`Pkl/BuktiPklController@show` | Auth pemilik/G pembimbing/K lingkup; tanpa PKL aktif<br>`token` | Bukti event tersimpan, tanpa mengambil lokasi/menulis ulang<br>`tests/Feature/Http/Pkl/BuktiPklControllerTest.php`<br>`test_bukti_pkl_tetap_terbaca_setelah_nonaktif_get_051` |
| `GET /pengumuman/{slug}`<br>`publik.pengumuman.detail` / R2 | `routes/modul/website.php`<br>`HalamanPublikController@pengumumanDetail` | Publik + website aktif, konten terbit dan rentang tampil<br>`slug` | tema/<kode>/pengumuman-detail.blade.php<br>`tests/Feature/Http/HalamanPublikControllerTest.php`<br>`test_pengumuman_draf_tidak_bocor_get_052` |
| `POST /admin/laporan/ekspor`<br>`inti.laporan.ekspor` / R1 | `routes/inti.php`<br>`LaporanIntiController@export` | AS/P hak laporan dan lingkup/kolom; Core tanpa lisensi<br>`laporan,format,filter,kolom_allowlist` | CSV/XLSX/PDF sinkron; tidak membaca model modul belum terpasang<br>`tests/Feature/Http/LaporanIntiControllerTest.php`<br>`test_ekspor_core_tidak_tergantung_keuangan_post_053` |
| `GET /admin/{prefix_modul}/{sisa?}`<br>`inti.modul_blokir.get` / R1 | `routes/inti.php`<br>`ModulBelumTersediaController@handle` | Auth aktif; regex prefix hanya 15 URL modul, diletakkan paling akhir; mutasi CSRF<br>`prefix_modul,sisa?` | 404 bila belum dirilis/path tak dikenal; 403 bila hak/lisensi ditolak; tidak memanggil class yang tidak tersedia<br>`tests/Feature/Http/ModulBelumTersediaControllerTest.php`<br>`test_fallback_tidak_menelan_route_core_get_054` |
| `POST /admin/{prefix_modul}/{sisa?}`<br>`inti.modul_blokir.post` / R1 | `routes/inti.php`<br>`ModulBelumTersediaController@handle` | Auth aktif; regex prefix hanya 15 URL modul, diletakkan paling akhir; mutasi CSRF<br>`prefix_modul,sisa?` | 404 bila belum dirilis/path tak dikenal; 403 bila hak/lisensi ditolak; tidak memanggil class yang tidak tersedia<br>`tests/Feature/Http/ModulBelumTersediaControllerTest.php`<br>`test_fallback_tidak_menelan_route_core_post_055` |
| `PATCH /admin/{prefix_modul}/{sisa?}`<br>`inti.modul_blokir.patch` / R1 | `routes/inti.php`<br>`ModulBelumTersediaController@handle` | Auth aktif; regex prefix hanya 15 URL modul, diletakkan paling akhir; mutasi CSRF<br>`prefix_modul,sisa?` | 404 bila belum dirilis/path tak dikenal; 403 bila hak/lisensi ditolak; tidak memanggil class yang tidak tersedia<br>`tests/Feature/Http/ModulBelumTersediaControllerTest.php`<br>`test_fallback_tidak_menelan_route_core_patch_056` |
| `DELETE /admin/{prefix_modul}/{sisa?}`<br>`inti.modul_blokir.delete` / R1 | `routes/inti.php`<br>`ModulBelumTersediaController@handle` | Auth aktif; regex prefix hanya 15 URL modul, diletakkan paling akhir; mutasi CSRF<br>`prefix_modul,sisa?` | 404 bila belum dirilis/path tak dikenal; 403 bila hak/lisensi ditolak; tidak memanggil class yang tidak tersedia<br>`tests/Feature/Http/ModulBelumTersediaControllerTest.php`<br>`test_fallback_tidak_menelan_route_core_delete_057` |
| `GET /media/{media}`<br>`publik.media` / R2 | `routes/modul/website.php`<br>`MediaWebsiteController@showPublic` | Publik hanya referensi versi/album terbit saat website aktif; logo profil Core boleh tanpa CMS aktif<br>`media` | Stream no-store dari namespace CMS; berkas_privat tidak bisa dipilih<br>`tests/Feature/Http/MediaWebsiteControllerTest.php`<br>`test_media_draf_tidak_dapat_diakses_publik_get_058` |
| `GET /admin/website/media/{media}/pratinjau`<br>`website.media.pratinjau` / R2 | `routes/modul/website.php`<br>`MediaWebsiteController@preview` | E hak media+website aktif<br>`media` | Stream private no-store, bukan URL storage<br>`tests/Feature/Http/MediaWebsiteControllerTest.php`<br>`test_pratinjau_media_membutuhkan_editor_get_059` |

<a id="file-pendukung"></a>

### T6. Seluruh file pendukung yang diperlukan

Tidak ada lapisan Service/Repository/Policy baru atau file infrastruktur. Daftar ini melengkapi Model/migration/komponen/view/Controller/test pada T2–T5. File bawaan vendor/generated build tidak menjadi source kustom yang harus ditulis.

| File | Isi dan tanggung jawab | Rilis |
|---|---|---|
| `app/Providers/AppServiceProvider.php` | Adapter Fortify/Livewire/throttle/binding response, tanpa query DB saat boot | R1 |
| `app/Http/Controllers/Autentikasi/ResponsMasuk.php` | Implement LoginResponse + TwoFactorLoginResponse; redirect berdasarkan role/akses, owner belum MFA ke keamanan akun | R1 |
| `app/Http/Controllers/Autentikasi/AturUlangSandi.php` | Adapter ResetsUserPasswords resmi, validasi/hash lewat Laravel, model Pengguna; tidak kriptografi custom | R1 |
| `app/Http/Controllers/Autentikasi/UbahSandi.php` | Adapter UpdatesUserPasswords resmi; current_password dan logout sesi lain | R1 |
| `bootstrap/app.php` | Registrasi alias middleware dan pemuat route; error aman, tanpa pekerjaan host | R1 |
| `bootstrap/providers.php` | Registrasi AppServiceProvider scaffold, tidak query DB saat boot | R1 |
| `.env.example` | Nama variabel runtime tanpa rahasia, locale/waktu/MariaDB/private disk/mail sinkron | R1 |
| `composer.json` | Laravel13/PHP8.5/Livewire4/Fortify; dependency kompatibel dan command test/Pint | R1 |
| `composer.lock` | Versi PHP dependency hasil resolve nyata, wajib di-commit | R1 |
| `package.json` | Tailwind4/build/Vite dan Playwright dev; tanpa worker runtime | R1 |
| `package-lock.json` | Lock NPM hasil install, satu lockfile package manager | R1 |
| `vite.config.js` | Build app.css/app.js dan aset tema, tanpa domain hardcoded | R1 |
| `phpunit.xml` | Suite test MariaDB terisolasi dan environment test | R1 |
| `playwright.config.cjs` | Browser baseURL dari environment test, desktop/mobile, hanya data sintetis | R1 |
| `routes/web.php` | Muat file route rilis secara eksplisit sekali; fallback setelah semua route nyata | R1 |
| `routes/console.php` | Perintah manual aplikasi: owner-awal, admin-awal, pulihkan-mfa-owner, periksa-rilis, bersihkan-unggahan; tidak Scheduler | R1 |
| `config/app.php` | Locale id, timezone UTC, URL trusted dari runtime | R1 |
| `config/auth.php` | Guard web/provider Pengguna/password broker reset | R1 |
| `config/hashing.php` | Argon2id melalui Hash resmi; preflight PHP harus mendukung, tanpa fallback diam-diam | R1 |
| `config/database.php` | Koneksi mariadb, strict mode, tanpa credential source | R1 |
| `config/filesystems.php` | Disk private dan public terpisah; URL privat tidak publik | R1 |
| `config/session.php` | Database sessions, lifetime120, secure/HttpOnly/SameSite=lax saat produksi | R1 |
| `config/cache.php` | File cache; tidak menjadi sumber izin mutasi | R1 |
| `config/mail.php` | SMTP sinkron, no queue; log mail hanya test, produksi harus pengiriman teruji | R1 |
| `config/fortify.php` | Reset/TOTP, ignore public register/passkey; password timeout900, rate limiter terdaftar | R1 |
| `config/modul.php` | 15 kode + prefix + rilis/minimum schema/reader; readiness vs lisensi terpisah | R1 |
| `config/hak_akses.php` | Seluruh kode izin dari T3–T5/T7 dan grant aktor/lingkup; owner bukan superuser data | R1 |
| `config/arsip.php` | Resource/reader/query proyeksi fixed T7; tanpa resolusi nama class/path dari request | R1 |
| `config/operasi_massal.php` | Jenis inti/alumni/tagihan, pemroses Model fixed, batch20, paralel3, lease120s, budget12s | R1 |
| `config/privasi.php` | Whitelist kolom/log/upload, batas ukuran dan retensi manual; tidak ada penghapusan otomatis | R1 |
| `config/services.php` | Konfigurasi integrasi framework standar dari environment, tanpa credential repository | R1 |
| `database/migrations/2026_11_11_000000_buat_sesi_dan_reset_sandi.php` | Tabel teknis Laravel sessions/password_reset_tokens sesuai PRD; user_id terindeks, tidak mensyaratkan akun seed | R1 |
| `database/seeders/DatabaseSeeder.php` | Panggil dua seeder katalog/role, tanpa data sekolah/owner bawaan | R1 |
| `database/seeders/PeranHakAksesSeeder.php` | Upsert13 peran/hak dan grant normal normatif; hak khusus tidak diberikan otomatis, peran/penugasan akun existing tidak direset | R1 |
| `database/seeders/KatalogModulSeeder.php` | Upsert15 modul, lisensi awal nonaktif hanya bila row belum ada; jangan reset expiry/tema/data | R1 |
| `tests/TestCase.php` | Bootstrap Laravel pengujian dengan koneksi uji terisolasi | R1 |
| `tests/Support/DataSekolahSintetis.php` | Builder guru/siswa/kelas/penugasan dan fixture modul sesuai rilis, bukan data nyata | R1 |
| `tests/Support/DuaKoneksiMariaDb.php` | Dua koneksi DB independen untuk race/lock/deadlock tanpa membungkus semua dalam satu transaction test | R1 |
| `tests/Browser/AlurSekolah.spec.cjs` | Guru/wali/pimpinan/siswa/admin, data dari fixture reset test, mobile/keyboard/error/timeout | R1 |
| `resources/css/app.css` | Token admin, responsif/touch44, lima tema CSS terisolasi saat R2 | R1 |
| `resources/js/app.js` | Modal/fokus, form HTTP PKL, uploader, batch3, proof polling saat perlu; tidak GPS tracking/localStorage lokasi | R1 |
| `resources/views/publik/informasi-sekolah.blade.php` | Fallback publik tanpa CMS, tetap ada link login | R1 |
| `resources/views/errors/403.blade.php` | Akses ditolak dengan langkah berikutnya, tanpa data privat | R1 |
| `resources/views/errors/404.blade.php` | Data/path tak tersedia; tidak membocorkan kepemilikan | R1 |
| `resources/views/errors/419.blade.php` | Sesi/CSRF berakhir, login ulang dan baca bukti sebelum retry | R1 |
| `resources/views/errors/429.blade.php` | Batas percobaan, tampil waktu coba lagi | R1 |
| `resources/views/errors/500.blade.php` | Pesan aman dan ID referensi, tanpa SQL/stacktrace | R1 |
| `resources/views/errors/503.blade.php` | Perawatan/ketidakcocokan rilis; tidak sukses palsu | R1 |
| `resources/views/surat/cetak.blade.php` | Cetak surat versi final/nomor asli, file opsional | R2 |
| `resources/views/website/pratinjau.blade.php` | Wrapper preview noindex/no-store pada tema dipilih | R2 |
| `config/dompdf.php` | Remote/PHP nonaktif, font lokal, path/chroot controlled | R2 |
| `routes/console-pkl.php` | Perintah manual retensi koordinat berizin/preview; dimuat routes/console.php hanya saat R3 dikirim | R3 |
| `app/Http/Middleware/PastikanPenggunaAktif.php` | Pemeriksaan lintas request sesuai T3–T5; action tetap memeriksa objek/kolom/version | R1 |
| `app/Http/Middleware/PastikanOwner.php` | Pemeriksaan lintas request sesuai T3–T5; action tetap memeriksa objek/kolom/version | R1 |
| `app/Http/Middleware/PastikanMfaOwner.php` | Pemeriksaan lintas request sesuai T3–T5; action tetap memeriksa objek/kolom/version | R1 |
| `app/Http/Middleware/PeriksaHakAkses.php` | Pemeriksaan lintas request sesuai T3–T5; action tetap memeriksa objek/kolom/version | R1 |
| `app/Http/Middleware/PeriksaModulAktif.php` | Pemeriksaan lintas request sesuai T3–T5; action tetap memeriksa objek/kolom/version | R1 |
| `app/Http/Middleware/PeriksaAksesArsip.php` | Pemeriksaan lintas request sesuai T3–T5; action tetap memeriksa objek/kolom/version | R1 |
| `resources/views/layouts/admin.blade.php` | Layout satu bahasa desain, slot halaman dan navigasi diizinkan | R1 |
| `resources/views/layouts/owner.blade.php` | Layout satu bahasa desain, slot halaman dan navigasi diizinkan | R1 |
| `resources/views/layouts/portal.blade.php` | Layout satu bahasa desain, slot halaman dan navigasi diizinkan | R1 |
| `resources/views/layouts/publik.blade.php` | Layout satu bahasa desain, slot halaman dan navigasi diizinkan | R1 |
| `resources/views/components/navigasi.blade.php` | Komponen UI bersama, label Indonesia/keyboard/responsif, tanpa query bisnis | R1 |
| `resources/views/components/modal.blade.php` | Komponen UI bersama, label Indonesia/keyboard/responsif, tanpa query bisnis | R1 |
| `resources/views/components/tombol.blade.php` | Komponen UI bersama, label Indonesia/keyboard/responsif, tanpa query bisnis | R1 |
| `resources/views/components/tabel.blade.php` | Komponen UI bersama, label Indonesia/keyboard/responsif, tanpa query bisnis | R1 |
| `resources/views/components/pesan-validasi.blade.php` | Komponen UI bersama, label Indonesia/keyboard/responsif, tanpa query bisnis | R1 |
| `resources/views/components/notifikasi.blade.php` | Komponen UI bersama, label Indonesia/keyboard/responsif, tanpa query bisnis | R1 |
| `resources/views/components/progres.blade.php` | Komponen UI bersama, label Indonesia/keyboard/responsif, tanpa query bisnis | R1 |
| `resources/views/components/filter.blade.php` | Komponen UI bersama, label Indonesia/keyboard/responsif, tanpa query bisnis | R1 |
| `resources/views/components/state-kosong.blade.php` | Komponen UI bersama, label Indonesia/keyboard/responsif, tanpa query bisnis | R1 |
| `resources/views/components/input.blade.php` | Komponen UI bersama, label Indonesia/keyboard/responsif, tanpa query bisnis | R1 |
| `resources/views/components/badge-status.blade.php` | Komponen UI bersama, label Indonesia/keyboard/responsif, tanpa query bisnis | R1 |
| `resources/views/components/paginasi.blade.php` | Komponen UI bersama, label Indonesia/keyboard/responsif, tanpa query bisnis | R1 |
| `resources/views/auth/login.blade.php` | View Fortify, named routes T5, CSRF, validasi/error Indonesia | R1 |
| `resources/views/auth/lupa-sandi.blade.php` | View Fortify, named routes T5, CSRF, validasi/error Indonesia | R1 |
| `resources/views/auth/reset-sandi.blade.php` | View Fortify, named routes T5, CSRF, validasi/error Indonesia | R1 |
| `resources/views/auth/tantangan-mfa.blade.php` | View Fortify, named routes T5, CSRF, validasi/error Indonesia | R1 |
| `resources/views/auth/konfirmasi-sandi.blade.php` | View Fortify, named routes T5, CSRF, validasi/error Indonesia | R1 |
| `resources/views/auth/daftar-alumni.blade.php` | Registrasi klaim alumni tamu, bukan pendaftaran siswa/PPDB | R3 |
| `lang/id/auth.php` | Terjemahan pesan resmi framework/paket | R1 |
| `lang/id/passwords.php` | Terjemahan pesan resmi framework/paket | R1 |
| `lang/id/validation.php` | Terjemahan pesan resmi framework/paket | R1 |
| `lang/id/pagination.php` | Terjemahan pesan resmi framework/paket | R1 |
| `lang/id.json` | Terjemahan JSON string bawaan Fortify/Livewire dan paket yang tidak memakai kelompok pesan PHP | R1 |
| `database/factories/PenggunaFactory.php` | Factory sintetis R1; master/relasi dari helper test | R1 |
| `database/factories/SiswaFactory.php` | Factory sintetis R1; master/relasi dari helper test | R1 |
| `database/factories/GuruFactory.php` | Factory sintetis R1; master/relasi dari helper test | R1 |
| `database/factories/KelasFactory.php` | Factory sintetis R1; master/relasi dari helper test | R1 |
| `database/factories/PenempatanKelasSiswaFactory.php` | Factory sintetis R1; master/relasi dari helper test | R1 |
| `database/factories/ProfilAlumniFactory.php` | Factory sintetis sesuai struktur rilis, tanpa data nyata | R3 |
| `database/factories/MitraIndustriFactory.php` | Factory sintetis sesuai struktur rilis, tanpa data nyata | R3 |
| `database/factories/TagihanFactory.php` | Factory sintetis sesuai struktur rilis, tanpa data nyata | R4 |
| `database/factories/AsetFactory.php` | Factory sintetis sesuai struktur rilis, tanpa data nyata | R4 |
| `resources/views/tema/klasik/beranda.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/klasik/halaman.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/klasik/berita.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/klasik/artikel.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/klasik/pengumuman.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/klasik/pengumuman-detail.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/klasik/galeri.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/klasik/album.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/klasik/navigasi.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/klasik/footer.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/modern/beranda.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/modern/halaman.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/modern/berita.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/modern/artikel.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/modern/pengumuman.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/modern/pengumuman-detail.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/modern/galeri.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/modern/album.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/modern/navigasi.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/modern/footer.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/vokasi/beranda.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/vokasi/halaman.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/vokasi/berita.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/vokasi/artikel.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/vokasi/pengumuman.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/vokasi/pengumuman-detail.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/vokasi/galeri.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/vokasi/album.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/vokasi/navigasi.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/vokasi/footer.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/berita/beranda.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/berita/halaman.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/berita/berita.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/berita/artikel.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/berita/pengumuman.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/berita/pengumuman-detail.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/berita/galeri.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/berita/album.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/berita/navigasi.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/berita/footer.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/kreatif/beranda.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/kreatif/halaman.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/kreatif/berita.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/kreatif/artikel.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/kreatif/pengumuman.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/kreatif/pengumuman-detail.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/kreatif/galeri.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/kreatif/album.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/kreatif/navigasi.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |
| `resources/views/tema/kreatif/footer.blade.php` | Presentasi konten publik sama; tidak memuat query/aturan bisnis atau file privat | R2 |

Adapter autentikasi di Controllers adalah binding kontrak framework, bukan Controller HTTP duplikat. Tidak membuat `app/Actions`, Service/Repository atau Policy baru. Source Fortify/Livewire/Composer/Vite standar tetap dikelola dependency/scaffold, bukan disalin ke folder aplikasi.

### T7. Reader arsip, laporan dan format

Controller Core memakai allowlist resource, proyeksi kolom dan filter di bawah. Core laporan memakai `inti.laporan.ekspor`; modul memakai `arsip.ekspor`. Format default CSV, XLSX/PDF bila <=10000 row dan sesuai jenis; lebih besar minta filter/CSV streaming. BK hanya agregat tanpa catatan sesi. Kuitansi hanya PDF/HTML pemilik/berizin. Website inventaris konten hanya HTML, tidak menambah kewajiban ekspor bisnis yang tidak diperlukan.

| Kode / laporan | Model / kolom / filter | Hak dan keluaran | File template / pengujian |
|---|---|---|---|
| `core.siswa` | `Siswa`; NIS,nama,kelas_historis,status<br>Filter: tahun,kelas,program,status | AS/P sesuai bidang | `resources/views/laporan/core/siswa.blade.php`<br>`tests/Feature/Laporan/CoreSiswaTest.php::test_laporan_siswa_tidak_berisi_uang_bk` |
| `core.akademik` | `Kelulusan`; NIS,nama,tahun,keputusan,referensi<br>Filter: tahun,kelas,keputusan | AS/P akademik | `resources/views/laporan/core/akademik.blade.php`<br>`tests/Feature/Laporan/CoreAkademikTest.php::test_laporan_kelulusan_tidak_perlu_keuangan` |
| `absensi_izin.presensi` | `KehadiranSiswa`; tanggal,jam,kelas,nama,H/S/I/A,sumber,konfirmasi<br>Filter: tanggal,kelas,mapel,status | G/W/BK/P lingkup | `resources/views/laporan/absensi_izin/presensi.blade.php`<br>`tests/Feature/Laporan/AbsensiIzinPresensiTest.php::test_rekap_tidak_menjumlah_pkl_sebagai_hari_sekolah` |
| `jurnal_guru.jurnal` | `JurnalKegiatanGuru`; guru,tanggal,kegiatan,hasil,status,nomor_versi<br>Filter: tanggal,guru,kategori | G pemilik/P lingkup | `resources/views/laporan/jurnal_guru/jurnal.blade.php`<br>`tests/Feature/Laporan/JurnalGuruJurnalTest.php::test_rekap_bukan_skor_kepatuhan_kalender` |
| `surat.agenda` | `Surat`; tanggal,arah,nomor_final_atau_asli,perihal,tujuan,status<br>Filter: tanggal,arah,jenis,status | T/P bidang | `resources/views/laporan/surat/agenda.blade.php`<br>`tests/Feature/Laporan/SuratAgendaTest.php::test_agenda_membedakan_nomor_sistem_dan_asli` |
| `website.konten` | `HalamanWebsite`; jenis,judul,slug,versi_draf,versi_terbit,status<br>Filter: jenis,status | E | `resources/views/laporan/website/konten.blade.php`<br>`tests/Feature/Laporan/WebsiteKontenTest.php::test_ekspor_konten_tidak_berkas_privat` |
| `pelanggaran.poin` | `CatatanPelanggaran`; siswa,kelas,tahun,aturan_snapshot,bobot_sah,tindak_lanjut<br>Filter: tahun,kelas,siswa,status | W/BK/P lingkup | `resources/views/laporan/pelanggaran/poin.blade.php`<br>`tests/Feature/Laporan/PelanggaranPoinTest.php::test_rekap_hanya_versi_efektif_sah` |
| `bk.layanan` | `KasusKonseling`; bulan,jumlah_permintaan,jumlah_kasus,status_agregat<br>Filter: tanggal,program,kelas | BK/P agregat | `resources/views/laporan/bk/layanan.blade.php`<br>`tests/Feature/Laporan/BkLayananTest.php::test_laporan_umum_tidak_catatan_privat` |
| `jadwal.jadwal` | `JadwalPelajaran`; versi,hari,jam,guru,kelas,mapel,ruang<br>Filter: versi,kelas,guru,ruang | G/P/W lingkup | `resources/views/laporan/jadwal/jadwal.blade.php`<br>`tests/Feature/Laporan/JadwalJadwalTest.php::test_cetak_jadwal_sesuai_versi` |
| `pkl.pkl` | `PenempatanPkl`; siswa,segmen,mitra,pembimbing,hadir,jurnal,nilai_final<br>Filter: periode,kelas,mitra,pembimbing | K/G lingkup, tanpa koordinat mentah | `resources/views/laporan/pkl/pkl.blade.php`<br>`tests/Feature/Laporan/PklPklTest.php::test_laporan_pkl_tidak_bocor_koordinat` |
| `hubin.kerja_sama` | `KerjaSamaIndustri`; mitra,nomor,versi,periode,status_efektif,kegiatan<br>Filter: mitra,tanggal,status | G/T/P HUBIN | `resources/views/laporan/hubin/kerja_sama.blade.php`<br>`tests/Feature/Laporan/HubinKerjaSamaTest.php::test_mou_habis_tetap_histori` |
| `bkk.seleksi` | `LamaranKerja`; lowongan,kandidat,status_seleksi,waktu_penyerahan<br>Filter: lowongan,angkatan,status | PA/P HUBIN, tanpa CV/kontak bila tidak perlu | `resources/views/laporan/bkk/seleksi.blade.php`<br>`tests/Feature/Laporan/BkkSeleksiTest.php::test_terkirim_tidak_dihitung_diterima` |
| `tracer.cakupan` | `SasaranTracer`; angkatan,sasaran_aktif,jangkauan,respons_efektif,persentase,status_aktivitas<br>Filter: periode,versi_daftar,program | PA/P agregat | `resources/views/laporan/tracer/cakupan.blade.php`<br>`tests/Feature/Laporan/TracerCakupanTest.php::test_sasaran_nol_dan_tanpa_akun_dihitung_benar` |
| `keuangan.saldo` | `Tagihan`; siswa,tahun,kelas_snapshot,tagihan_efektif,penerimaan_neto,sisa,status_admin<br>Filter: tahun,periode,kelas_historis,status | B/P dengan hak uang eksplisit | `resources/views/laporan/keuangan/saldo.blade.php`<br>`tests/Feature/Laporan/KeuanganSaldoTest.php::test_saldo_neto_dan_pembalik_refund_benar` |
| `sarpras.aset` | `Aset`; kode,nama,lokasi,kondisi,ketersediaan,pinjaman_aktif<br>Filter: ruang,kondisi,status | PS/P sarana unit | `resources/views/laporan/sarpras/aset.blade.php`<br>`tests/Feature/Laporan/SarprasAsetTest.php::test_pengembalian_parsial_unit_lain_tetap_dipinjam` |
| `laboratorium.stok` | `SaldoBahanLokasi`; bahan,lokasi,satuan,saldo,mutasi,transfer<br>Filter: tanggal,ruang,bahan | PS/P sarana unit | `resources/views/laporan/laboratorium/stok.blade.php`<br>`tests/Feature/Laporan/LaboratoriumStokTest.php::test_saldo_sesuai_mutasi_pasangan_transfer` |
| `supervisi.hasil` | `Supervisi`; guru,supervisor,instrumen_versi,hasil_efektif,tindak_lanjut,status<br>Filter: tahun,guru,supervisor,status | G pemilik/G supervisor/P lingkup | `resources/views/laporan/supervisi/hasil.blade.php`<br>`tests/Feature/Laporan/SupervisiHasilTest.php::test_instrumen_baru_tidak_ubah_hasil_lama` |
| `keuangan.kuitansi` | `Pembayaran`; nomor_kuitansi,siswa,kelas_snapshot,nominal,tanggal,metode,status_sah_void<br>Filter: pembayaran_id | B/ S/AL pemilik/WA anak terverifikasi hak lihat kuitansi | `resources/views/laporan/keuangan/kuitansi.blade.php`<br>`tests/Feature/Laporan/KeuanganKuitansiTest.php::test_kuitansi_void_tetap_berlabel_batal` |

Resource detail arsip yang diizinkan: Presensi=sesi/pengiriman/izin/tindak_lanjut; Jurnal=jurnal/versi; Surat=surat/revisi; Keuangan=tagihan/pembayaran/kuitansi/penyesuaian/pengembalian; BKK=lowongan/lamaran/penyerahan; Sarpras=aset/pinjaman/kejadian; BK=kasus/sesi (hanya BK lingkup, isi privat tetap encrypted); Jadwal=versi/jadwal; Tracer=periode/jawaban/versi/sasaran; HUBIN=kerja_sama/adendum/kegiatan; Supervisi=supervisi/hasil/instrumen; Lab=booking/mutasi/transfer; Pelanggaran=catatan/versi/pembinaan; Website=konten/versi/menu/media; PKL=penempatan/kehadiran/jurnal/nilai. Setiap alias di `config/arsip.php` menunjuk Model/kondisi/kolom tetap, bukan refleksi dari request. Resource tidak dikenal ditolak sebelum query. Core identitas/akademik tidak memakai syarat pernah aktif.

### T8. Metode domain Model dan sumber tunggal aturan

Daftar ini melengkapi relasi/casts tiap Model T2. Method UI/Controller hanya melakukan validasi transport, memeriksa hak, memanggil method domain, lalu merender hasil. Isi transaksi, pemetaan versi/sumber, locking dan invariant tidak disalin ke Controller lain.

| Model | Metode domain kanonik yang wajib dibuat |
|---|---|
| `app/Models/AdendumKerjaSama.php` | `catat()` |
| `app/Models/AlbumWebsite.php` | `arsipkan()`, `simpan()`, `terbitkan()` |
| `app/Models/Aset.php` | `catatKondisi()`, `pindahkan()`, `simpanUnit()` |
| `app/Models/AturanPelanggaran.php` | `simpan()` |
| `app/Models/BahanHabisPakai.php` | `simpan()` |
| `app/Models/BerkasPrivat.php` | `tautkan()`, `unduhTerotorisasi()`, `unggahSementara()` |
| `app/Models/CatatanPelanggaran.php` | `buatKoreksi()`, `catat()`, `putuskan()` |
| `app/Models/DelegasiTugas.php` | `tetapkan()` |
| `app/Models/Guru.php` | `nonaktifkan()`, `simpanMaster()` |
| `app/Models/HakKhususPengguna.php` | `berikan()`, `cabut()` |
| `app/Models/ItemGaleriWebsite.php` | `simpan()` |
| `app/Models/JawabanTracer.php` | `catatWawancara()`, `kirim()`, `koreksiTerbatas()`, `revisiSebelumTutup()`, `simpanDraf()` |
| `app/Models/JenisBiaya.php` | `simpan()` |
| `app/Models/JurnalKegiatanGuru.php` | `buatRevisi()`, `kirim()`, `simpanDraf()` |
| `app/Models/JurnalPkl.php` | `buatRevisi()`, `kirim()`, `simpanDraf()` |
| `app/Models/KasusKonseling.php` | `bukaKembali()`, `gantiPenanggungJawab()`, `selesaikan()`, `terimaPermintaan()` |
| `app/Models/KategoriArtikelWebsite.php` | `simpan()` |
| `app/Models/KeberatanPelanggaran.php` | `ajukan()`, `ajukanDariWali()`, `putuskan()` |
| `app/Models/KegiatanHubin.php` | `catat()` |
| `app/Models/KehadiranSiswa.php` | `hitungStatusEfektif()`, `konfirmasiKelas()`, `queryRekapBerizin()` |
| `app/Models/KejadianAset.php` | `catatKehilangan()`, `hapusAdministratif()`, `selesaikanKehilangan()`, `selesaikanPerbaikan()` |
| `app/Models/Kelas.php` | `nonaktifkan()`, `simpanMaster()` |
| `app/Models/KelompokKenaikan.php` | `pratinjau()` |
| `app/Models/Kelulusan.php` | `koreksiKeputusan()`, `pratinjau()` |
| `app/Models/KerjaSamaIndustri.php` | `batalkan()`, `perpanjang()`, `sahkan()`, `simpanDraf()` |
| `app/Models/KlarifikasiPresensi.php` | `ajukan()`, `putuskan()` |
| `app/Models/KonsentrasiKeahlian.php` | `nonaktifkan()`, `simpanMaster()` |
| `app/Models/KonteksPklSiswa.php` | `akhiri()`, `tetapkan()` |
| `app/Models/KoreksiPembayaran.php` | `ajukanVoid()`, `putuskan()` |
| `app/Models/KuotaPkl.php` | `tetapkan()` |
| `app/Models/LamaranKerja.php` | `ajukan()`, `ajukanSiswaAkhir()`, `ajukanUlang()`, `catatStatus()`, `koreksiHasil()`, `tarik()` |
| `app/Models/LanggananSaas.php` | `simpanKontrak()` |
| `app/Models/LisensiModul.php` | `kelayakanArsip()`, `pastikanOperasional()`, `pastikanSebelumCommit()`, `ubahAktivasi()` |
| `app/Models/LogAktivitas.php` | `catatAman()` |
| `app/Models/LokasiPkl.php` | `simpan()` |
| `app/Models/LowonganKerja.php` | `batalkan()`, `simpanDraf()`, `terbitkan()`, `tutup()` |
| `app/Models/MataPelajaran.php` | `nonaktifkan()`, `simpanMaster()` |
| `app/Models/MediaWebsite.php` | `hapusJikaTidakDipakai()`, `simpanUnggahan()` |
| `app/Models/MenuWebsite.php` | `simpan()`, `urutkan()` |
| `app/Models/MitraIndustri.php` | `simpan()` |
| `app/Models/MonitoringPkl.php` | `catat()` |
| `app/Models/MutasiStokLab.php` | `balik()`, `catat()` |
| `app/Models/OperasiMassal.php` | `jeda()`, `lanjutkan()`, `mulai()`, `pratinjau()`, `prosesBatch()`, `statusBacaSaja()`, `ulangiGagal()` |
| `app/Models/Pegawai.php` | `nonaktifkan()`, `simpanMaster()` |
| `app/Models/Pembayaran.php` | `catat()` |
| `app/Models/PembayaranLanggananSaas.php` | `catat()`, `koreksi()` |
| `app/Models/PemesananRuangLab.php` | `ajukan()`, `batalkan()`, `putuskan()` |
| `app/Models/PeminjamanAset.php` | `ajukan()`, `batalkanSebelumSerah()`, `putuskan()`, `serahkanUnit()`, `terimaUnit()` |
| `app/Models/PenempatanKelasSiswa.php` | `pindahkan()` |
| `app/Models/PenempatanPkl.php` | `aktifkan()`, `hentikan()`, `pindahkanSegmen()`, `selesaikan()`, `simpanDraf()` |
| `app/Models/PengajuanIzinPegawai.php` | `ajukan()`, `batalkan()`, `putuskan()`, `simpanDraf()` |
| `app/Models/PengajuanIzinSiswa.php` | `ajukanDariWali()`, `ajukanOnline()`, `batalkan()`, `catatLangsung()`, `periksaOnline()` |
| `app/Models/PengajuanRegistrasiAlumni.php` | `daftarkan()`, `putuskan()` |
| `app/Models/PengecualianJadwal.php` | `catat()` |
| `app/Models/PengembalianPembayaran.php` | `ajukan()`, `balik()`, `putuskan()` |
| `app/Models/PenggabunganIdentitasAlumni.php` | `pratinjau()`, `sahkan()` |
| `app/Models/PenggabunganIdentitasSiswa.php` | `pratinjau()`, `sahkan()` |
| `app/Models/Pengguna.php` | `aturUlangSandi()`, `cabutPenugasan()`, `kelolaAkun()`, `kirimResetTerotorisasi()`, `lingkupPenugasan()`, `nonaktifkan()`, `pastikanBoleh()`, `sesiAktif()`, `tetapkanPenugasan()`, `tetapkanPeranOperasional()`, `ubahProfilSendiri()`, `ubahSandi()` |
| `app/Models/PengirimanPresensi.php` | `buktiBacaSaja()`, `simpanKelas()`, `simpanSusulan()` |
| `app/Models/PenilaianPkl.php` | `bukaRevisi()`, `finalkan()` |
| `app/Models/PenyerahanKandidat.php` | `catat()`, `catatKirimUlang()` |
| `app/Models/PenyesuaianStokLab.php` | `ajukan()`, `sahkan()` |
| `app/Models/PenyesuaianTagihan.php` | `ajukan()`, `putuskan()` |
| `app/Models/PeriodePkl.php` | `simpan()` |
| `app/Models/PeriodeTracer.php` | `buka()`, `simpanDraf()`, `tutup()` |
| `app/Models/PeristiwaKehadiranPkl.php` | `buktiBacaSaja()`, `catatMasuk()`, `catatPulang()`, `koreksiPetugas()` |
| `app/Models/PeristiwaPribadiAlumni.php` | `catat()` |
| `app/Models/PermintaanBantuanPkl.php` | `ajukan()`, `putuskan()` |
| `app/Models/PermintaanKonseling.php` | `ajukanDariPelanggaran()`, `ajukanLangsung()`, `batalkan()`, `tolak()` |
| `app/Models/PerpindahanBahanLab.php` | `balik()`, `pindahkan()` |
| `app/Models/ProfilAlumni.php` | `identitasKanonik()`, `scopeUntukPengguna()`, `ubahProfilSendiri()` |
| `app/Models/ProfilSekolah.php` | `halamanMinimal()`, `simpanProfil()`, `ubahTema()` |
| `app/Models/ProgramKeahlian.php` | `nonaktifkan()`, `simpanMaster()` |
| `app/Models/RevisiKontenWebsite.php` | `arsipkan()`, `buatDrafDariVersi()`, `simpanDraf()`, `terbitkan()` |
| `app/Models/RiwayatAlumni.php` | `akhiri()`, `simpan()` |
| `app/Models/Ruang.php` | `kunciPenggunaan()`, `nonaktifkan()`, `periksaBentrok()`, `simpanMaster()` |
| `app/Models/SasaranTracer.php` | `catatJangkauan()`, `revisiSnapshot()`, `tetapkanSnapshot()` |
| `app/Models/Semester.php` | `aktifkan()`, `simpanDraf()` |
| `app/Models/SesiKonseling.php` | `batalkan()`, `revisi()`, `sahkan()`, `simpanDraf()` |
| `app/Models/Siswa.php` | `catatPerubahanStatus()`, `identitasKanonik()`, `penempatanPadaTanggal()`, `scopeUntukPengguna()`, `simpanIdentitas()` |
| `app/Models/SiswaWali.php` | `cabutHubungan()`, `verifikasiHubungan()` |
| `app/Models/Supervisi.php` | `batalkan()`, `gantiSupervisor()`, `selesaikan()`, `tetapkan()` |
| `app/Models/Surat.php` | `ajukanPersetujuan()`, `batalkanFinal()`, `buatPengganti()`, `buatRevisi()`, `finalkan()`, `putuskanPersetujuan()`, `simpanDraf()` |
| `app/Models/Tagihan.php` | `sahkanPembatalan()`, `terbitkan()`, `usulkanPembatalan()` |
| `app/Models/TahunAjaran.php` | `aktifkan()`, `simpanDraf()` |
| `app/Models/TarifBiaya.php` | `tetapkan()` |
| `app/Models/TemplatSurat.php` | `simpan()` |
| `app/Models/TindakLanjutPelanggaran.php` | `catat()`, `selesaikan()` |
| `app/Models/TindakLanjutPresensi.php` | `bukaKembali()`, `catatTemuan()`, `catatTindakan()`, `klarifikasi()`, `rujukBk()`, `selesaikan()` |
| `app/Models/TindakLanjutSupervisi.php` | `isiHasilGuru()`, `tetapkan()`, `verifikasi()` |
| `app/Models/TokenLokasiPkl.php` | `terbitkan()` |
| `app/Models/VersiFormulirTracer.php` | `simpanDraf()` |
| `app/Models/VersiHasilSupervisi.php` | `buatRevisi()`, `sahkan()`, `simpanDraf()` |
| `app/Models/VersiInstrumenSupervisi.php` | `sahkan()`, `simpanDraf()` |
| `app/Models/VersiJadwal.php` | `batalkan()`, `buatRevisi()`, `periksaBentrok()`, `simpanDraf()`, `terbitkan()` |
| `app/Models/VersiJurnalGuru.php` | `mintaPerbaikan()`, `periksa()` |
| `app/Models/VersiJurnalPkl.php` | `periksa()` |
| `app/Models/VersiLokasiPkl.php` | `buatVersi()` |
| `app/Models/VersiPenilaianPkl.php` | `simpanDraf()` |
| `app/Models/WaliSiswa.php` | `simpanIdentitas()` |

### T9. Suite integrasi, UAT dan bukti rilis

Selain setiap file/method test di T3–T5 dan T7, suite berikut membuktikan invariant lintas layar. Hasil masih BELUM DIUJI. Uji penolakan harus menjaga prasyarat lain tetap sah (modul aktif, versi benar, objek ada) agar tidak lulus karena ditolak guard yang keliru; uji concurrency memakai dua koneksi tanpa transaction wrapper tunggal. Tiap halaman T3 wajib file `tests/Feature/Antarmuka/<komponen>Test.php` dengan `test_halaman_menolak_akses_salah`, `test_filter_paginasi_dan_state_kosong`, `test_detail_dalam_lingkup` serta metode action T4. Jangan melewati pengujian karena class/file rencana sudah ada.

| File / metode | Task / rilis | Skenario penerimaan |
|---|---|---|
| `tests/Feature/Fondasi/RuteDanManifestTest.php`<br>`test_named_route_unik_handler_view_tersedia` | K0.01 / R1–R4 | route:list/cache + seluruh role + kode modul invalid; tidak mengaktifkan modul belum siap |
| `tests/Feature/Fondasi/MigrasiDanRelasiTest.php`<br>`test_instalasi_dan_upgrade_fk_urut` | K1.08 / R1–R4 | Seluruh migration registry, FK/UNIQUE/nullable, model casts/relasi, fresh DB uji dan upgrade salinan |
| `tests/Feature/Fondasi/PembacaArsipTest.php`<br>`test_reader_lintas_versi_saat_modul_nonaktif` | C0.06 / R1–R4 | Reader/kunci identitas/tema/unggahan dari semua rilis terpasang, preflight versi tak didukung menolak sebelum mutasi |
| `tests/Feature/Autentikasi/KeamananAkunTest.php`<br>`test_login_reset_session_throttle` | K2.02 / R1 | Login/logout/reset, csrf, session fixation, inactive account, token single-use, tidak enumerate akun |
| `tests/Feature/Autentikasi/MfaOwnerTest.php`<br>`test_owner_wajib_mfa_dan_recovery_single_use` | K2.01 / R1 | Setup, konfirmasi, challenge/recovery, kode regenerasi, owner-awal/admin-awal terpisah, SMTP gagal/resend tanpa akun ganda dan pemulihan manual |
| `tests/Feature/Otorisasi/MatriksHakAksesTest.php`<br>`test_setiap_halaman_dan_aksi_menolak_role_salah` | K2.03 / R1–R4 | Data provider semua halaman/aksi HTTP+Livewire; IDOR, column projection, owner bukan bypass |
| `tests/Feature/Otorisasi/PenugasanEfektifTest.php`<br>`test_tanggal_delegasi_dan_pencabutan_lingkup` | F1.12 / R1–R4 | Guru A/B, wali lama/baru, kepala program, unit, delegasi subset, kelas/siswa tidak sesuai |
| `tests/Feature/Akademik/HistoriSiswaTest.php`<br>`test_id_tetap_penempatan_tidak_overlap` | F2.13 / R1 | Mutasi tengah tahun/kembali, belum ditempatkan, satu enrolmen/tahun, jangan cascade histori |
| `tests/Feature/Akademik/KelulusanDanAliasTest.php`<br>`test_xii_xiii_koreksi_dan_rekonsiliasi` | F2.15 / R1/R3/R4 | Kelulusan eksplisit sekali, akun sama, koreksi data terkait, alias tanpa cycle/duplikasi/bocor uang |
| `tests/Feature/ProsesMassal/ParalelDanPemulihanTest.php`<br>`test_seribu_item_dua_browser_dan_lease` | F4.12 / R1–R4 | 100/600/1000 sintetis; dua koneksi MariaDB, crash setelah claim/commit, pause/resume/retry, status GET tidak recovery |
| `tests/Feature/Presensi/PengirimanAtomikTest.php`<br>`test_seluruh_himpunan_tersimpan_atau_rollback` | F5.17 / R1 | 35 sasaran + pengecualian PKL, payload kurang/tambahan/duplikat, gagal baris akhir, dua tab, timeout, hari Jakarta berubah |
| `tests/Feature/Presensi/SumberIzinTest.php`<br>`test_izin_prefill_bukan_observasi_dasar` | A0.02 / R1 | S/I overlap, cancel -> NULL tanpa observasi, klarifikasi, konfirmasi ulang tanpa deadline, susulan wali saja |
| `tests/Feature/JurnalGuru/VersiKegiatanTest.php`<br>`test_nonmengajar_revisi_dan_review_versi` | F7.15 / R1 | Tanpa kelas/mapel/GPS/jadwal, tanggal lampau boleh, masa depan ditolak, banyak rincian, guru lain ditolak |
| `tests/Feature/Owner/ToggleDanKedaluwarsaTest.php`<br>`test_form_terbuka_lock_expiry_dan_aktif_ulang` | F7.17 / R1–R4 | Dua tab/request saat toggle, cache versi; expiry sebelum commit rollback; Core/modul lain/arsip tetap jalan |
| `tests/Feature/Surat/NomorDanRevisiTest.php`<br>`test_nomor_final_unik_tidak_dipakai_ulang` | B0.01 / R2 | Paralel, nomor eksternal tidak ambil counter, no-file, revisi/pengganti/batal, approval per versi |
| `tests/Feature/Website/PublikasiDanMediaTest.php`<br>`test_draf_tetap_privat_media_terpakai_aman` | B0.08 / R2 | Dua editor, sanitize XSS, restore, URL/menu stabil, parent-child move, media referenced dilarang hapus |
| `tests/Feature/Website/LimaTemaTest.php`<br>`test_seluruh_jenis_halaman_di_lima_tema` | F7.18 / R2 | 5 tema × beranda/halaman/berita/detail/pengumuman/galeri, mobile/keyboard/kosong/error, fallback/admin tetap |
| `tests/Feature/Bk/KerahasiaanDanRujukanTest.php`<br>`test_langsung_tanpa_wali_catatan_privat_tidak_bocor` | A0.08 / R2 | Permintaan langsung/rujukan, petugas diganti, sesi encrypted, buka kembali, log/laporan/owner terlarang |
| `tests/Feature/Jadwal/BentrokDanVersiTest.php`<br>`test_recurring_jadwal_vs_booking_tanpa_kalender` | M0.03 / R2/R4 | Bentrok guru/kelas/ruang, lock ruang dua koneksi, pengecualian pengganti, nonaktif modul tidak melepas reservasi |
| `tests/Feature/Pkl/GeolokasiDanUrutanTest.php`<br>`test_token_jarak_akurasi_urutan_idempoten` | P0.04 / R3 | Radius100/accuracy<=50, d+a<=r, invalid/old/outside/border, client distance palsu, token60detik, satu open, lewat tengah malam18jam |
| `tests/Feature/Pkl/SegmenJurnalNilaiTest.php`<br>`test_pindah_segmen_kuota_dan_revisi_nilai` | A0.07 / R3 | Kuota paralel, pindah pembimbing/mitra, tanggal lama, jurnal tanpa GPS, koreksi petugas bukan lolos GPS, nilai kertas final berversi |
| `tests/Feature/Alumni/VerifikasiDanRiwayatTest.php`<br>`test_alumni_lama_tidak_siswa_fiktif` | F2.13 / R3 | Klaim cocok/duplikat/penyamaran, link profil, kontak aman, rangkap riwayat, data pribadi opsional |
| `tests/Feature/Bkk/PenyerahanDanSeleksiTest.php`<br>`test_cv_terpilih_pic_manual_dan_versi_hasil` | M0.05 / R3 | Batas lamar, tarik/ajukan ulang record sama, siswa akhir XOR alumni, CV opsional, tujuan consent, dikirim bukan diterima |
| `tests/Feature/Tracer/SnapshotDanCakupanTest.php`<br>`test_penyebut_mencakup_alumni_tanpa_akun` | M0.06 / R3 | Sasaran nol, form/jawaban berversi, wawancara sumber petugas, jangkauan terpisah, koreksi setelah tutup, alias kanonik |
| `tests/Feature/Keuangan/SaldoDanPengembalianTest.php`<br>`test_ledger_neto_dan_pembalik_pada_tagihan_batal` | B0.02 / R4 | Paralel bayar/void/refund/adjust, invoice immutable, pengembalian parsial, batal neto0, pembalik atomik buka invoice atau tolak |
| `tests/Feature/Sarana/UnitDanPengembalianTest.php`<br>`test_satu_unit_pinjam_paralel_kembali_parsial` | B0.04 / R4 | 2/3 kembali, rusak/hilang, perbaikan/penghapusan, master tunggal Lab/Sarpras, unit scope dan histori lokasi |
| `tests/Feature/Laboratorium/StokDanTransferTest.php`<br>`test_saldo_tidak_minus_pasangan_transfer_atomik` | M0.04 / R4 | Dua pemakaian, salah satu sisi gagal, retry transfer, pembalik, satuan snapshot, penyesuaian fisik |
| `tests/Feature/Supervisi/InstrumenDanHasilTest.php`<br>`test_hasil_terikat_rubrik_dan_penanggung_jawab` | A0.10 / R4 | Versi instrumen/hasil, ganti supervisor, batal/revisi, guru tidak nilai sendiri, selesai hanya setelah verifikasi |
| `tests/Feature/Keamanan/UnggahanDanEksporTest.php`<br>`test_mime_path_xss_csv_formula_dan_kolom` | K2.04 / R1–R4 | File invalid/oversize/ZIP bomb/path traversal/IDOR; konten sanitize; ekspor sinkron dan formula spreadsheet aman |
| `tests/Feature/Keamanan/ModulMandiriTest.php`<br>`test_semua_pasangan_mandiri_tanpa_query_tabel_tidak_ada` | F6.08 / R1–R4 | Absensi tanpa Jadwal/BK/Jurnal; Jurnal sendiri; PKL tanpa HUBIN; BKK/Tracer masing-masing; Lab tanpa Sarpras; web nonaktif |
| `tests/Feature/Rilis/UpgradeDanPemulihanTest.php`<br>`test_upgrade_source_skema_data_dan_reader_tetap_utuh` | F8.11 / R1–R4 | Rilis lama->baru, migration gagal, tab lama, mixed schema/source menolak, rollback source kompatibel, restore operator dismoke |
| `tests/Feature/Rilis/UatNonItTest.php`<br>`test_alur_utama_selesai_dengan_label_indonesia` | M0.07 / R1–R4 | Guru/wali/siswa/operator awam, 320/390/1440px, touch44, tanpa istilah teknis di alur produk, status simpan nyata |
| `tests/Feature/Pelanggaran/VersiDanKeberatanTest.php`<br>`test_bobot_snapshot_validasi_koreksi_dan_keberatan` | A0.11 / R2 | Perubahan aturan, validasi per versi, sanggahan siswa/wali, record batal dan poin efektif tanpa sanksi otomatis |

<a id="penerimaan-prd"></a>

#### T9.1 Pemetaan seluruh ID penerimaan PRD

Seluruh 124 ID pada tabel penerimaan PRD memiliki file/method berikut. Setiap ID menjadi label data-provider pada test terkait, dengan fixture dan assertion sesuai skenario PRD, bukan otomatis lulus karena sudah tercatat. Cakupan rilis mengikuti fitur yang disentuh dan regresi modul terpasang. Bukti hasil tetap BELUM DIUJI.

| ID / kebutuhan PRD | File dan metode test / label dataset |
|---|---|
| S01 / 35 siswa X TKJ 1 dipilih sekaligus ke XI TKJ 1 | `tests/Feature/Akademik/KelulusanDanAliasTest.php::test_xii_xiii_koreksi_dan_rekonsiliasi` / `S01` |
| S02 / Dua siswa dikecualikan dari batch | `tests/Feature/Akademik/KelulusanDanAliasTest.php::test_xii_xiii_koreksi_dan_rekonsiliasi` / `S02` |
| S03 / Satu siswa tinggal kelas | `tests/Feature/Akademik/KelulusanDanAliasTest.php::test_xii_xiii_koreksi_dan_rekonsiliasi` / `S03` |
| S04 / Admin tanpa sengaja mengklik proses dua kali | `tests/Feature/Akademik/KelulusanDanAliasTest.php::test_xii_xiii_koreksi_dan_rekonsiliasi` / `S04` |
| S05 / Siswa pindah kelas pada 1 Februari | `tests/Feature/Akademik/KelulusanDanAliasTest.php::test_xii_xiii_koreksi_dan_rekonsiliasi` / `S05` |
| S06 / Tahun ajaran berganti ketika siswa XII program 3 tahun belum diputuskan lulus | `tests/Feature/Akademik/KelulusanDanAliasTest.php::test_xii_xiii_koreksi_dan_rekonsiliasi` / `S06` |
| S07 / Siswa XII program 4 tahun | `tests/Feature/Akademik/KelulusanDanAliasTest.php::test_xii_xiii_koreksi_dan_rekonsiliasi` / `S07` |
| S08 / Siswa XII program 3 tahun dinyatakan lulus | `tests/Feature/Akademik/KelulusanDanAliasTest.php::test_xii_xiii_koreksi_dan_rekonsiliasi` / `S08` |
| S09 / Siswa dinyatakan lulus dan masih menunggak | `tests/Feature/Keuangan/SaldoDanPengembalianTest.php::test_ledger_neto_dan_pembalik_pada_tagihan_batal` / `S09` |
| S10 / Tagihan kelas X Rp100.000, siswa kini XI dengan tarif Rp125.000 | `tests/Feature/Keuangan/SaldoDanPengembalianTest.php::test_ledger_neto_dan_pembalik_pada_tagihan_batal` / `S10` |
| S11 / Koreksi kelulusan setelah alumni mengisi tracer | `tests/Feature/Akademik/KelulusanDanAliasTest.php::test_xii_xiii_koreksi_dan_rekonsiliasi` / `S11` |
| S12 / Siswa mutasi keluar lalu kembali | `tests/Feature/Akademik/KelulusanDanAliasTest.php::test_xii_xiii_koreksi_dan_rekonsiliasi` / `S12` |
| S13 / Proses kenaikan kelas gagal pada satu peserta | `tests/Feature/ProsesMassal/ParalelDanPemulihanTest.php::test_seribu_item_dua_browser_dan_lease` / `S13` |
| S14 / Tahun ajaran lama dinonaktifkan | `tests/Feature/Fondasi/PembacaArsipTest.php::test_reader_lintas_versi_saat_modul_nonaktif` / `S14` |
| A01 / Alumni 2017 tidak ada dalam `siswa`, daftar sendiri | `tests/Feature/Alumni/VerifikasiDanRiwayatTest.php::test_alumni_lama_tidak_siswa_fiktif` / `A01` |
| A02 / Lulusan 2026 sudah ada `siswa` dan `kelulusan` | `tests/Feature/Alumni/VerifikasiDanRiwayatTest.php::test_alumni_lama_tidak_siswa_fiktif` / `A02` |
| A03 / Orang mendaftar dua kali dengan data sama | `tests/Feature/Alumni/VerifikasiDanRiwayatTest.php::test_alumni_lama_tidak_siswa_fiktif` / `A03` |
| A04 / Pendaftar mengaku memakai NIS siswa lain | `tests/Feature/Alumni/VerifikasiDanRiwayatTest.php::test_alumni_lama_tidak_siswa_fiktif` / `A04` |
| A05 / Alumni bekerja 2021–2023, kuliah 2022–2026, usaha sejak 2025 | `tests/Feature/Alumni/VerifikasiDanRiwayatTest.php::test_alumni_lama_tidak_siswa_fiktif` / `A05` |
| A06 / Alumni mengisi status menikah | `tests/Feature/Alumni/VerifikasiDanRiwayatTest.php::test_alumni_lama_tidak_siswa_fiktif` / `A06` |
| A07 / Arsip siswa lama diimpor setelah alumni mendaftar | `tests/Feature/Alumni/VerifikasiDanRiwayatTest.php::test_alumni_lama_tidak_siswa_fiktif` / `A07` |
| A08 / Alumni tanpa `siswa_id` menggunakan BKK dan Tracer | `tests/Feature/Alumni/VerifikasiDanRiwayatTest.php::test_alumni_lama_tidak_siswa_fiktif` / `A08` |
| A09 / Alumni menerima tawaran kerja lewat BKK tetapi belum mulai bekerja | `tests/Feature/Bkk/PenyerahanDanSeleksiTest.php::test_cv_terpilih_pic_manual_dan_versi_hasil` / `A09` |
| M01 / Admin memilih 600 siswa lintas halaman menggunakan **Pilih Semua hasil filter** | `tests/Feature/ProsesMassal/ParalelDanPemulihanTest.php::test_seribu_item_dua_browser_dan_lease` / `M01` |
| M02 / Memproses 600 siswa dengan 3 request paralel | `tests/Feature/ProsesMassal/ParalelDanPemulihanTest.php::test_seribu_item_dua_browser_dan_lease` / `M02` |
| M03 / Dua batch bersamaan memuat siswa yang sama karena retry | `tests/Feature/ProsesMassal/ParalelDanPemulihanTest.php::test_seribu_item_dua_browser_dan_lease` / `M03` |
| M04 / Browser ditutup pada 380/600 | `tests/Feature/ProsesMassal/ParalelDanPemulihanTest.php::test_seribu_item_dua_browser_dan_lease` / `M04` |
| M05 / Lima dari 600 siswa datanya tidak valid | `tests/Feature/ProsesMassal/ParalelDanPemulihanTest.php::test_seribu_item_dua_browser_dan_lease` / `M05` |
| M06 / Koneksi terputus saat request dikirim | `tests/Feature/ProsesMassal/ParalelDanPemulihanTest.php::test_seribu_item_dua_browser_dan_lease` / `M06` |
| M07 / Admin klik **Jeda** | `tests/Feature/ProsesMassal/ParalelDanPemulihanTest.php::test_seribu_item_dua_browser_dan_lease` / `M07` |
| M08 / Konfirmasi kelulusan 200 siswa | `tests/Feature/ProsesMassal/ParalelDanPemulihanTest.php::test_seribu_item_dua_browser_dan_lease` / `M08` |
| M09 / Simpan formulir biasa atau hapus data | `tests/Feature/Rilis/UatNonItTest.php::test_alur_utama_selesai_dengan_label_indonesia` / `M09` |
| U01 / SMK membeli hanya Surat + Website | `tests/Feature/Owner/ToggleDanKedaluwarsaTest.php::test_form_terbuka_lock_expiry_dan_aktif_ulang` / `U01` |
| U02 / PKL memberi nilai industri melalui kertas | `tests/Feature/Pkl/SegmenJurnalNilaiTest.php::test_pindah_segmen_kuota_dan_revisi_nilai` / `U02` |
| U03 / Surat masuk hanya metadata, tanpa berkas | `tests/Feature/Surat/NomorDanRevisiTest.php::test_nomor_final_unik_tidak_dipakai_ulang` / `U03` |
| U04 / Pilih seluruh siswa kelas X untuk naik ke XI | `tests/Feature/Akademik/KelulusanDanAliasTest.php::test_xii_xiii_koreksi_dan_rekonsiliasi` / `U04` |
| U05 / Kelas XII/XIII pada pergantian tahun | `tests/Feature/Akademik/KelulusanDanAliasTest.php::test_xii_xiii_koreksi_dan_rekonsiliasi` / `U05` |
| U06 / Lulus dengan tagihan belum lunas | `tests/Feature/Keuangan/SaldoDanPengembalianTest.php::test_ledger_neto_dan_pembalik_pada_tagihan_batal` / `U06` |
| U07 / Tagihan semester lalu setelah naik kelas | `tests/Feature/Keuangan/SaldoDanPengembalianTest.php::test_ledger_neto_dan_pembalik_pada_tagihan_batal` / `U07` |
| U08 / Alat Lab diperbaiki melalui menu Lab | `tests/Feature/Sarana/UnitDanPengembalianTest.php::test_satu_unit_pinjam_paralel_kembali_parsial` / `U08` |
| U09 / Wali kelas mencatat S/I siswa | `tests/Feature/Presensi/SumberIzinTest.php::test_izin_prefill_bukan_observasi_dasar` / `U09` |
| U10 / Guru mengajukan izin | `tests/Feature/Presensi/SumberIzinTest.php::test_izin_prefill_bukan_observasi_dasar` / `U10` |
| U11 / Editor menambah submenu website | `tests/Feature/Website/PublikasiDanMediaTest.php::test_draf_tetap_privat_media_terpakai_aman` / `U11` |
| U12 / Masa berlaku modul habis | `tests/Feature/Owner/ToggleDanKedaluwarsaTest.php::test_form_terbuka_lock_expiry_dan_aktif_ulang` / `U12` |
| U13 / Alumni lama mendaftar mandiri | `tests/Feature/Alumni/VerifikasiDanRiwayatTest.php::test_alumni_lama_tidak_siswa_fiktif` / `U13` |
| U14 / Alumni bekerja sambil kuliah lalu wirausaha | `tests/Feature/Alumni/VerifikasiDanRiwayatTest.php::test_alumni_lama_tidak_siswa_fiktif` / `U14` |
| U15 / Proses massal 600 siswa memakai progres dan modal | `tests/Feature/ProsesMassal/ParalelDanPemulihanTest.php::test_seribu_item_dua_browser_dan_lease` / `U15` |
| DB01 / Identitas siswa | `tests/Feature/Fondasi/MigrasiDanRelasiTest.php::test_instalasi_dan_upgrade_fk_urut` / `DB01` |
| DB02 / Tahun ajaran | `tests/Feature/Akademik/HistoriSiswaTest.php::test_id_tetap_penempatan_tidak_overlap` / `DB02` |
| DB03 / Penempatan | `tests/Feature/Akademik/HistoriSiswaTest.php::test_id_tetap_penempatan_tidak_overlap` / `DB03` |
| DB04 / Promosi | `tests/Feature/Akademik/KelulusanDanAliasTest.php::test_xii_xiii_koreksi_dan_rekonsiliasi` / `DB04` |
| DB05 / Kelulusan | `tests/Feature/Akademik/KelulusanDanAliasTest.php::test_xii_xiii_koreksi_dan_rekonsiliasi` / `DB05` |
| DB06 / Kelulusan | `tests/Feature/Akademik/KelulusanDanAliasTest.php::test_xii_xiii_koreksi_dan_rekonsiliasi` / `DB06` |
| DB07 / Alumni | `tests/Feature/Alumni/VerifikasiDanRiwayatTest.php::test_alumni_lama_tidak_siswa_fiktif` / `DB07` |
| DB08 / Riwayat alumni | `tests/Feature/Alumni/VerifikasiDanRiwayatTest.php::test_alumni_lama_tidak_siswa_fiktif` / `DB08` |
| DB09 / Keuangan | `tests/Feature/Keuangan/SaldoDanPengembalianTest.php::test_ledger_neto_dan_pembalik_pada_tagihan_batal` / `DB09` |
| DB10 / Keuangan | `tests/Feature/Keuangan/SaldoDanPengembalianTest.php::test_ledger_neto_dan_pembalik_pada_tagihan_batal` / `DB10` |
| DB11 / Keuangan | `tests/Feature/Keuangan/SaldoDanPengembalianTest.php::test_ledger_neto_dan_pembalik_pada_tagihan_batal` / `DB11` |
| DB12 / Aset | `tests/Feature/Sarana/UnitDanPengembalianTest.php::test_satu_unit_pinjam_paralel_kembali_parsial` / `DB12` |
| DB13 / Stok | `tests/Feature/Laboratorium/StokDanTransferTest.php::test_saldo_tidak_minus_pasangan_transfer_atomik` / `DB13` |
| DB14 / Absensi | `tests/Feature/Presensi/SumberIzinTest.php::test_izin_prefill_bukan_observasi_dasar` / `DB14` |
| DB15 / Surat | `tests/Feature/Surat/NomorDanRevisiTest.php::test_nomor_final_unik_tidak_dipakai_ulang` / `DB15` |
| DB16 / PKL | `tests/Feature/Pkl/SegmenJurnalNilaiTest.php::test_pindah_segmen_kuota_dan_revisi_nilai` / `DB16` |
| DB17 / Website | `tests/Feature/Website/PublikasiDanMediaTest.php::test_draf_tetap_privat_media_terpakai_aman` / `DB17` |
| PM01 / Izin | `tests/Feature/Otorisasi/MatriksHakAksesTest.php::test_setiap_halaman_dan_aksi_menolak_role_salah` / `PM01` |
| PM02 / Data privat | `tests/Feature/Otorisasi/MatriksHakAksesTest.php::test_setiap_halaman_dan_aksi_menolak_role_salah` / `PM02` |
| PM03 / Penugasan | `tests/Feature/Otorisasi/MatriksHakAksesTest.php::test_setiap_halaman_dan_aksi_menolak_role_salah` / `PM03` |
| PM04 / Lisensi | `tests/Feature/Owner/ToggleDanKedaluwarsaTest.php::test_form_terbuka_lock_expiry_dan_aktif_ulang` / `PM04` |
| PM05 / Pendaftaran alumni | `tests/Feature/Alumni/VerifikasiDanRiwayatTest.php::test_alumni_lama_tidak_siswa_fiktif` / `PM05` |
| BL01 / Proses massal | `tests/Feature/ProsesMassal/ParalelDanPemulihanTest.php::test_seribu_item_dua_browser_dan_lease` / `BL01` |
| BL02 / Proses massal | `tests/Feature/ProsesMassal/ParalelDanPemulihanTest.php::test_seribu_item_dua_browser_dan_lease` / `BL02` |
| BL03 / Proses massal | `tests/Feature/ProsesMassal/ParalelDanPemulihanTest.php::test_seribu_item_dua_browser_dan_lease` / `BL03` |
| BL04 / Proses massal | `tests/Feature/ProsesMassal/ParalelDanPemulihanTest.php::test_seribu_item_dua_browser_dan_lease` / `BL04` |
| UX01 / UI | `tests/Feature/Rilis/UatNonItTest.php::test_alur_utama_selesai_dengan_label_indonesia` / `UX01` |
| UX02 / UI | `tests/Feature/Rilis/UatNonItTest.php::test_alur_utama_selesai_dengan_label_indonesia` / `UX02` |
| UX03 / UI | `tests/Feature/Rilis/UatNonItTest.php::test_alur_utama_selesai_dengan_label_indonesia` / `UX03` |
| SEC01 / Keamanan | `tests/Feature/Keamanan/UnggahanDanEksporTest.php::test_mime_path_xss_csv_formula_dan_kolom` / `SEC01` |
| OPS01 / Backup | `tests/Feature/Rilis/UpgradeDanPemulihanTest.php::test_upgrade_source_skema_data_dan_reader_tetap_utuh` / `OPS01` |
| OPS02 / Rilis | `tests/Feature/Rilis/UpgradeDanPemulihanTest.php::test_upgrade_source_skema_data_dan_reader_tetap_utuh` / `OPS02` |
| OP01 / Guru membuat jurnal rapat/persiapan tanpa kelas, jadwal dan presensi | `tests/Feature/JurnalGuru/VersiKegiatanTest.php::test_nonmengajar_revisi_dan_review_versi` / `OP01` |
| OP02 / 35 siswa tampil, dua S dan satu I dari wali; lainnya H; guru menyimpan | `tests/Feature/Presensi/PengirimanAtomikTest.php::test_seluruh_himpunan_tersimpan_atau_rollback` / `OP02` |
| OP03 / Payload kehilangan satu siswa, ID ganda/tambahan, status invalid atau satu baris gagal DB | `tests/Feature/Presensi/PengirimanAtomikTest.php::test_seluruh_himpunan_tersimpan_atau_rollback` / `OP03` |
| OP04 / Commit berhasil tetapi respons timeout; klik ulang/two-tab | `tests/Feature/Presensi/PengirimanAtomikTest.php::test_seluruh_himpunan_tersimpan_atau_rollback` / `OP04` |
| OP05 / Guru mengirim tanggal lampau/future atau halaman melewati tengah malam | `tests/Feature/Presensi/PengirimanAtomikTest.php::test_seluruh_himpunan_tersimpan_atau_rollback` / `OP05` |
| OP06 / Wali kelas mencatat S/I saat halaman guru sudah terbuka | `tests/Feature/Presensi/SumberIzinTest.php::test_izin_prefill_bukan_observasi_dasar` / `OP06` |
| OP07 / Wali kelas mengonfirmasi rekap berbulan lalu | `tests/Feature/Presensi/SumberIzinTest.php::test_izin_prefill_bukan_observasi_dasar` / `OP07` |
| OP08 / Temuan selesai di wali atau diteruskan ke BK | `tests/Feature/Presensi/SumberIzinTest.php::test_izin_prefill_bukan_observasi_dasar` / `OP08` |
| OP09 / Guru pengganti, wali baru, delegasi berakhir atau ID kelas dimanipulasi | `tests/Feature/Otorisasi/PenugasanEfektifTest.php::test_tanggal_delegasi_dan_pencabutan_lingkup` / `OP09` |
| OP10 / Owner nonaktifkan modul saat request simpan/bulk sedang berjalan | `tests/Feature/Owner/ToggleDanKedaluwarsaTest.php::test_form_terbuka_lock_expiry_dan_aktif_ulang` / `OP10` |
| OP11 / Aktif ulang setelah expiry dan setelah upgrade | `tests/Feature/Owner/ToggleDanKedaluwarsaTest.php::test_form_terbuka_lock_expiry_dan_aktif_ulang` / `OP11` |
| OP12 / Editor mengganti lima tema berulang dan mencoba kode tema invalid | `tests/Feature/Website/LimaTemaTest.php::test_seluruh_jenis_halaman_di_lima_tema` / `OP12` |
| OP13 / Upgrade fitur atau framework dari versi lama dengan data dan modul nonaktif | `tests/Feature/Rilis/UpgradeDanPemulihanTest.php::test_upgrade_source_skema_data_dan_reader_tetap_utuh` / `OP13` |
| AU01 / Rincian tunggakan pada kelulusan berpotensi bocor | `tests/Feature/Otorisasi/MatriksHakAksesTest.php::test_setiap_halaman_dan_aksi_menolak_role_salah` / `AU01` |
| AU02 / Izin bertumpuk, dibatalkan atau siswa ternyata hadir | `tests/Feature/Presensi/SumberIzinTest.php::test_izin_prefill_bukan_observasi_dasar` / `AU02` |
| AU03 / Arsip saat lisensi nonaktif belum punya jalur akses | `tests/Feature/Fondasi/PembacaArsipTest.php::test_reader_lintas_versi_saat_modul_nonaktif` / `AU03` |
| AU04 / Rilis R1 tercampur kewajiban R2–R4 | `tests/Feature/Fondasi/RuteDanManifestTest.php::test_named_route_unik_handler_view_tersedia` / `AU04` |
| AU05 / PKL pindah tempat/pembimbing, kuota dan koreksi nilai | `tests/Feature/Pkl/SegmenJurnalNilaiTest.php::test_pindah_segmen_kuota_dan_revisi_nilai` / `AU05` |
| AU06 / Pemeriksaan ulang jurnal setelah koreksi | `tests/Feature/JurnalGuru/VersiKegiatanTest.php::test_nonmengajar_revisi_dan_review_versi` / `AU06` |
| AU07 / Seluruh sesi terlewat sampai berganti hari | `tests/Feature/Presensi/SumberIzinTest.php::test_izin_prefill_bukan_observasi_dasar` / `AU07` |
| AU08 / Nomor surat eksternal, pembatalan dan nomor terpakai | `tests/Feature/Surat/NomorDanRevisiTest.php::test_nomor_final_unik_tidak_dipakai_ulang` / `AU08` |
| AU09 / Salah tagihan, koreksi pembayaran dan pengembalian uang | `tests/Feature/Keuangan/SaldoDanPengembalianTest.php::test_ledger_neto_dan_pembalik_pada_tagihan_batal` / `AU09` |
| AU10 / Batas lamaran, tenggat, penarikan dan hasil seleksi | `tests/Feature/Bkk/PenyerahanDanSeleksiTest.php::test_cv_terpilih_pic_manual_dan_versi_hasil` / `AU10` |
| AU11 / Aset unit, hilang/rusak dan pengembalian sebagian | `tests/Feature/Sarana/UnitDanPengembalianTest.php::test_satu_unit_pinjam_paralel_kembali_parsial` / `AU11` |
| AU12 / Konseling langsung berbeda dari rujukan presensi | `tests/Feature/Bk/KerahasiaanDanRujukanTest.php::test_langsung_tanpa_wali_catatan_privat_tidak_bocor` / `AU12` |
| AU13 / Revisi jadwal, pengganti, batal dan bentrok | `tests/Feature/Jadwal/BentrokDanVersiTest.php::test_recurring_jadwal_vs_booking_tanpa_kalender` / `AU13` |
| AU14 / Form tracer berubah dan koreksi jawaban historis | `tests/Feature/Tracer/SnapshotDanCakupanTest.php::test_penyebut_mencakup_alumni_tanpa_akun` / `AU14` |
| AU15 / MoU berakhir/revisi membatalkan PKL secara keliru | `tests/Feature/Pkl/SegmenJurnalNilaiTest.php::test_pindah_segmen_kuota_dan_revisi_nilai` / `AU15` |
| AU16 / Instrumen supervisi, pengganti, nilai dan selesai | `tests/Feature/Supervisi/InstrumenDanHasilTest.php::test_hasil_terikat_rubrik_dan_penanggung_jawab` / `AU16` |
| AU17 / Pembatalan ruang, stok fisik dan pengembalian Lab | `tests/Feature/Laboratorium/StokDanTransferTest.php::test_saldo_tidak_minus_pasangan_transfer_atomik` / `AU17` |
| AU18 / Bobot pelanggaran berubah dan kejadian dibatalkan | `tests/Feature/Pelanggaran/VersiDanKeberatanTest.php::test_bobot_snapshot_validasi_koreksi_dan_keberatan` / `AU18` |
| AU19 / Revisi konten terbit dan media sedang digunakan | `tests/Feature/Website/PublikasiDanMediaTest.php::test_draf_tetap_privat_media_terpakai_aman` / `AU19` |
| AU20 / Tahun baru aktif tetapi siswa belum ditempatkan | `tests/Feature/Akademik/HistoriSiswaTest.php::test_id_tetap_penempatan_tidak_overlap` / `AU20` |
| AU21 / Dua identitas siswa/alumni telanjur terbentuk | `tests/Feature/Akademik/KelulusanDanAliasTest.php::test_xii_xiii_koreksi_dan_rekonsiliasi` / `AU21` |
| SM01 / Guru membuka satu kelas, termasuk siswa sedang PKL | `tests/Feature/Presensi/PengirimanAtomikTest.php::test_seluruh_himpunan_tersimpan_atau_rollback` / `SM01` |
| SM02 / Siswa PKL login dan berada pada lokasi dengan akurasi memadai | `tests/Feature/Pkl/GeolokasiDanUrutanTest.php::test_token_jarak_akurasi_urutan_idempoten` / `SM02` |
| SM03 / Luar radius, lokasi ditolak/tak akurat/token kedaluwarsa atau jarak klien palsu | `tests/Feature/Pkl/GeolokasiDanUrutanTest.php::test_token_jarak_akurasi_urutan_idempoten` / `SM03` |
| SM04 / Timeout setelah commit, klik dua kali, pulang lewat tengah malam, ganti segmen atau lisensi nonaktif | `tests/Feature/Pkl/GeolokasiDanUrutanTest.php::test_token_jarak_akurasi_urutan_idempoten` / `SM04` |
| SM05 / Siswa menulis jurnal saat GPS/presensi bermasalah | `tests/Feature/Pkl/SegmenJurnalNilaiTest.php::test_pindah_segmen_kuota_dan_revisi_nilai` / `SM05` |
| SM06 / Kepala program/konsentrasi membuka laporan | `tests/Feature/Otorisasi/PenugasanEfektifTest.php::test_tanggal_delegasi_dan_pencabutan_lingkup` / `SM06` |
| SM07 / Jadwal pelajaran dan reservasi Lab pada ruang/waktu sama | `tests/Feature/Jadwal/BentrokDanVersiTest.php::test_recurring_jadwal_vs_booking_tanpa_kalender` / `SM07` |
| SM08 / Bahan sama di dua bengkel dan transfer gagal pada satu sisi | `tests/Feature/Laboratorium/StokDanTransferTest.php::test_saldo_tidak_minus_pasangan_transfer_atomik` / `SM08` |
| SM09 / Pengelola BKK menyerahkan kandidat terpilih ke PIC | `tests/Feature/Bkk/PenyerahanDanSeleksiTest.php::test_cv_terpilih_pic_manual_dan_versi_hasil` / `SM09` |
| SM10 / Tracer 100 sasaran dengan 80 jawaban dan beberapa alumni belum berakun | `tests/Feature/Tracer/SnapshotDanCakupanTest.php::test_penyebut_mencakup_alumni_tanpa_akun` / `SM10` |
| SM11 / Guru/siswa/petugas non-IT menyelesaikan tugas dari HP | `tests/Feature/Rilis/UatNonItTest.php::test_alur_utama_selesai_dengan_label_indonesia` / `SM11` |

#### T9.2 UAT sekolah yang tidak dapat digantikan test otomatis

UatNonItTest/Playwright menguji perilaku UI terotomasi. Kelulusan UAT tetap memerlukan orang sekolah menjalankan alur, mengonfirmasi hasil nyata di DB/layar, mencatat kebingungan/galat dan menyetujui bukti; tidak otomatis dicentang karena suite tersedia.

| Rilis | Pelaku sekolah | Tugas yang harus dibuktikan dan dicatat |
|---|---|---|
| R1 | Admin, guru mapel, wali kelas, BK pendampingan, pimpinan; siswa/wali bila portal dipilih | Setup/impor/penugasan, pindah/kenaikan/kelulusan, presensi utuh+timeout, izin lewat telepon tanpa akun wali, konfirmasi lampau, jurnal kegiatan nonmengajar, hak data. |
| R2 | TU, Editor, Guru BK, pimpinan kurikulum/kesiswaan dan guru/siswa terkait | Surat tanpa file/final nomor, lima tema dan draf/media, validasi/keberatan, konseling langsung privat, bentrok/versi jadwal. |
| R3 | Siswa PKL, pembimbing/koordinator, HUBIN, Pengelola Alumni dan alumni | Masuk/pulang/lokasi+kendala, jurnal tanpa GPS, nilai kertas/segmen, mitra mandiri, klaim alumni, CV terpilih/manual PIC, sasaran tracer termasuk tanpa akun. |
| R4 | Bendahara, Pengelola Sarana, guru peminjam/supervisor, pimpinan dan pengguna tagihan sendiri | Cicilan/void/refund/pembalik aman, pinjam parsial/rusak/hilang, Lab tanpa Sarpras, stok/transfer/ruang, versi supervisi/tindak lanjut. |

Format bukti per sesi: tanggal/penguji/role/rilis/commit/data sintetis/langkah/hasil yang diharapkan/hasil nyata/isu/perbaikan/keputusan. Nilai host/rahasia/performa nyata tidak diisi rekaan. Lihat bagian Checklist kesiapan rilis untuk tanda persetujuan dan NO-GO yang berlaku.

### T10. Daftar perubahan dan batas kelengkapan

**Kelengkapan spesifikasi** diukur dengan seluruh 15 kode modul dan Core memiliki tabel/kolom/constraint/status, file migration/model, halaman/aksi/route, hak objek, pemilik transaksi, laporan yang diperlukan dan test penerimaan yang terpetakan pada baseline ini. **Kelengkapan implementasi** tetap nol sampai source dibuat dan bukti test/UAT tersedia. Tidak ada persentase kesiapan produksi yang diturunkan dari jumlah baris dokumen.

Perubahan kebutuhan baru boleh menambah kontrak secara eksplisit; tidak ada dokumen software yang menjamin tidak pernah membutuhkan revisi. Versi dependency patch, nilai rahasia, alamat host, data sekolah nyata, hasil performa dan persetujuan UAT berasal pelaksanaan/deployment yang sah, bukan keputusan teknis yang sengaja dibiarkan kosong. Jika source framework berbeda dari adapter yang diverifikasi, perubahan harus tercatat dan dites pada commit implementasi, tanpa menurunkan stack.

## Bahasa, struktur kode, dan penamaan (wajib)

Ikuti PRD §3.3, §4.2 dan §12.1.1. Fokus fitur: `routes/`, `app/Http/Controllers/`, `app/Livewire/`, `app/Models/`, `app/Http/Middleware/`, `resources/`. Migration/seeder, tes, terjemahan bawaan `lang/id/` dan konfigurasi minimum hanya pendukung yang diperlukan. Hindari arsitektur tambahan; alur Controller/Livewire ringkas, relasi/transaksi konsisten pada Model, validasi server dan otorisasi objek pada setiap aksi.

- [ ] K0.06 Tinjau struktur folder, metode singkat, penamaan jelas, penghindaran duplikasi/N+1 dan format Pint; tidak membuat lapisan generik yang belum dibutuhkan.
- [ ] K0.07 Implementasikan inventaris exact T3–T6 untuk rilis target, lalu cocokkan metode/URL/named route/handler/hak/lingkup/view/test terhadap route:list dan route:cache. Pemetaan spesifikasi lengkap; checkbox menunggu source dan pengujian nyata, bukan memilih handler nanti.
- [ ] K1.07 Terapkan tabel/kolom bisnis Bahasa Indonesia sesuai PRD; Model menetapkan tabel/FK eksplisit dan autentikasi `Pengguna` tetap kompatibel Laravel. Uji migration, relasi, constraint dan transaksi MariaDB.
- [ ] K1.08 Buat migration exact T2/PRD §12.10 untuk rilis target, termasuk kolom/FK/UNIQUE/indeks/status dan dependensi yang sudah ditetapkan. Uji instalasi kosong serta upgrade salinan sintetis tanpa FK ke source masa depan; checkbox hanya setelah bukti implementasi lulus, bukan menunda finalisasi skema.
- [ ] K1.09 Selesaikan seluruh Model/relasi/casts/query lingkup fondasi rilis setelah migration terkait; uji pada MariaDB, termasuk tanggal efektif, ID permanen, alias identitas, nullable/UNIQUE dan tidak meng-query struktur modul belum terpasang. Tidak mencentang selesai hanya karena file Model tersedia.
- [ ] K2.09 Terapkan 13 role inti PRD §4.2/13; gabungkan pengelolaan BKK/Tracer dalam Pengelola Alumni dan jabatan lain sebagai penugasan, bukan role baru; uji rangkap role, lingkup penugasan, larangan eskalasi owner, S/I siswa langsung dari wali kelas; persetujuan pejabat hanya untuk izin guru sendiri dan privasi laporan.
- [ ] K2.10 Luluskan Gerbang fondasi pada dokumen ini sebelum implementasi operasional modul: boot, route/model/schema, akun/MFA/penugasan, lisensi/arsip, UI bersama serta Core/massal dengan bukti pengujian. Modul dalam manifest belum dinyatakan siap sebelum alur bisnis dan gerbang produksi rilisnya lulus.
- [ ] F0.11 Siapkan locale `id`, `lang="id"` dan terjemahan Indonesia untuk UI, autentikasi, validasi, pesan framework/paket, laporan serta teks aksesibilitas. Format tanggal/angka Indonesia dan Rupiah.
- [ ] F7.13 Uji seluruh layar/alur rilis dalam Bahasa Indonesia termasuk error/validasi/reset sandi, modal, laporan/cetak dan tampilan mobile; kode status internal tidak tampil mentah.

**Acuan tabel bisnis:** `siswa`, `tahun_ajaran`, `kepesertaan_siswa`, `penempatan_kelas_siswa`, `profil_alumni`, `tagihan`, `pembayaran`, `operasi_massal`, dan `item_operasi_massal`; rincian tabel/kolom lain mengikuti PRD Bab 12. Jangan membuat tabel paralel dengan nama Inggris.

**Acuan fitur dan navigasi:** PRD §4.3 memuat daftar fitur dan §4.4 memuat menu/submenu, isi layar, tindakan dan batas akses 13 role. Tidak menambah modul atau role. Jurnal guru berisi kegiatan/hasil harian saat modul aktif, tidak bergantung jadwal/kalender; izin wali bisa dicatat dari luar aplikasi tanpa akun wali.

## Keputusan operasional dan kontrak rilis terbaru

PRD modul 02/03, §4.2.2, Bab 10, §13.5 dan §14.6 adalah acuan: jurnal kegiatan/hasil harian, presensi hari ini satu kelas atomik, S/I wali langsung, konfirmasi tanpa batas waktu, tanpa kalender akademik, 1 guru–1 kelas tanpa kelompok, 13 role dengan penugasan rinci, rilis R1–R4 serta upgrade aman. Kalender/libur bukan prasyarat input. Transaksi per item berlaku pada operasi massal; transaksi seluruh daftar berlaku pada kirim presensi kelas, jangan dicampur.

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

**Urutan eksekusi:** kontrak K0.07/K1.08 lebih dahulu; migration dan K1.09 sebelum auth/UI; K0/K2 diterapkan saat handler/aksi tersedia dan dibuktikan pada K2.10. K3 bersama F7–F9 setiap rilis. Gunakan tabel Urutan pelaksanaan per lapisan; K bukan modul tambahan atau layanan server baru.

---

## F0 — Fondasi proyek dan antarmuka

**Dependensi per item:** F0.01–F0.03 tidak menunggu fitur bisnis; F0.04 menunggu migration/model identitas dan izin; F0.05 menunggu kontrol akses/lisensi; F0.06–F0.08 menunggu pola UI yang dipilih. **Hasil akhir kelompok:** aplikasi dapat diakses dan login; item F0 diselesaikan lintas lapisan, bukan syarat menyelesaikan seluruh F0 sebelum F1.

- [ ] F0.01 Gunakan repository yang sudah ada dan instal Laravel 13 pada PHP 8.5, Livewire 4, Tailwind CSS 4.
- [ ] F0.02 Siapkan `.env.example` tanpa rahasia, koneksi MariaDB, timestamp UTC, tampilan/tanggal efektif `Asia/Jakarta`, dan storage privat.
- [ ] F0.03 Gunakan modular monolith sederhana dalam struktur Laravel standar: routes, Controllers, Livewire, Models, Middleware dan resources; kelompokkan per fitur seperlunya tanpa paket/lapisan tambahan. Kode modul harus stabil.
- [ ] F0.04 Siapkan login, logout, reset password, session, CSRF, rate limit login, dan pembuatan Super Admin pertama yang aman.
- [ ] F0.05 Terapkan rincian menu per role/penugasan PRD §4.4 dan daftar fitur §4.3; pisahkan layout `/owner`, `/admin`, portal siswa/alumni sesuai hak akses; implementasikan sidebar hanya untuk fitur yang sudah tersedia dalam rilis dan modul berlisensi aktif; jangan tampilkan handler/view kosong.
- [ ] F0.06 Buat komponen UI reusable: tabel, pencarian/filter, empty state, badge status, pagination, toast.
- [ ] F0.07 Buat modal reusable untuk konfirmasi, error penting, hasil proses; fokus keyboard, escape/close aman, responsif HP.
- [ ] F0.08 Implementasikan skeleton/indikator loading, spinner pada tombol, disable double-submit, validasi per field.
- [ ] F0.09 Siapkan audit log minimal untuk perubahan data penting tanpa mencatat kata sandi/token atau isi konseling.
- [ ] F0.10 Uji aplikasi tanpa Queue/Scheduler, tidak ada dependency worker yang harus aktif agar fitur berjalan.

**Selesai jika:** login/logout aman; UI responsif; tindakan hapus/ubah penting memakai modal; navigasi berdasar role; tes fondasi lulus.

## F1 — Core System dan data induk

**Dependensi:** F0.01–F0.03 dan kontrak K1.08 untuk skema; K1.09 dan lapisan keamanan/UI untuk CRUD. Tidak menunggu seluruh F0 selesai agar migration identitas tidak bergantung kembali pada login. **Hasil:** data sekolah dan data siswa dapat dikelola tanpa duplikasi.

- [ ] F1.01 Migration profil sekolah, jurusan/program 3–4 tahun, mata pelajaran, tahun ajaran/semester, kelas/rombel.
- [ ] F1.02 Migration `siswa`, `guru`, `pegawai`, `pengguna` dan relasi akun, dengan NIS unik dan NISN unik jika tersedia.
- [ ] F1.03 Migration `kepesertaan_siswa` dan `penempatan_kelas_siswa` sebagai riwayat; **jangan hanya menyimpan `siswa.kelas_id`**.
- [ ] F1.04 CRUD profil sekolah, jurusan/konsentrasi, mata pelajaran, ruang master, tahun ajaran/semester, kelas, guru, staf, siswa; filter/pencarian sederhana.
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

- [ ] F2.14 Tetapkan kontrak ID siswa dan snapshot tagihan/pembayaran sejak desain fondasi. Saat skema Keuangan dipasang (R4 atau lebih awal secara eksplisit), buat migration/model dan fixture sintetis lalu uji saldo/histori tetap utuh setelah naik/pindah/lulus. Penyelesaian task dan tes uang tidak menjadi syarat R1 bila Keuangan belum terpasang; UI/aktivasi tetap F5.09 pada R4.

- [ ] F2.15 Uji koreksi kelulusan melalui peninjauan data terkait dan audit tanpa menghapus profil/tracer/tagihan; siswa mutasi keluar yang kembali memakai identitas lama (PRD S11–S12).

**Cakupan F2:** R1 menyelesaikan siklus siswa, kelulusan dan profil alumni Core. F2.08 (portal registrasi), F2.10–F2.11 (riwayat/pribadi alumni) serta bagian registrasi mandiri F2.13 diselesaikan pada R3. F2.09 menyiapkan identitas/relasi Core sejak R1; penggunaan portal alumni lama mengikuti R3. Bagian desain F2.14 berlaku pada fondasi; model/migration/fixture Keuangan mengikuti saat skema dipasang dan penyelesaian transaksi uang tetap R4, bukan blocker R1.

**Selesai jika per rilis:** R1: kelas XII/XIII tidak otomatis alumni dan identitas/histori Core utuh. R3: registrasi/verifikasi/riwayat alumni juga lulus tanpa akun atau profil ganda. Item belum masuk rilis tetap tidak dicentang dan tidak menghalangi penerimaan R1.

## F3 — Owner, lisensi, dan hak akses modul

**Dependensi per item:** katalog/skema C0.01 dan model identitas lebih dahulu; pemeriksaan lisensi disiapkan pada lapisan keamanan; panel F3.01/F3.02/F3.04 setelah login/MFA serta UI bersama tersedia. **Hasil:** pemilik SaaS mengelola produk/lisensi pada instalasi sekolah, sebelum modul operasional dibuka.

- [ ] F3.01 Buat `/owner` dengan akses Super Admin terproteksi (MFA wajib sebelum rilis produksi; pembatasan akses tambahan sesuai lingkungan operator).
- [ ] F3.02 Profil pelanggan **satu sekolah** pada VPS tersebut; tidak ada katalog banyak tenant di instalasi.
- [ ] F3.03 Seed 15 modul dengan kode stabil; modul yang tidak aktif tidak muncul di menu.
- [ ] F3.04 Aktivasi/nonaktifkan modul, masa berlaku, catatan perpanjangan, dan pencatatan pembayaran langganan manual.
- [ ] F3.05 Blokir mutasi/fitur operasional modul nonaktif; arsip baca/ekspor dan bukti pengiriman Core tetap sesuai hak. Terapkan lock/versi lisensi dan invalidasi cache sesuai PRD §13.5, tanpa 500 atau mengganggu modul lain.
- [ ] F3.06 Saat modul kedaluwarsa: operasi modul diblokir, **data tidak dihapus**; akses core dan prosedur ekspor/retensi tetap tersedia sesuai kebijakan.
- [ ] F3.07 Owner **tidak otomatis** dapat membaca data BK, informasi keluarga, pembayaran personal, dan data sensitif sekolah.
- [ ] F3.08 Audit setiap perubahan lisensi/modul/pembayaran owner; tes role bypass dan akses URL langsung.

**Selesai sesuai rilis:** pada R1, `absensi_izin`/`jurnal_guru` mengikuti status lisensi dan readiness; modul belum dirilis tidak bisa diaktifkan, Core/arsip berizin tetap berjalan. Pada R2 tambahkan uji sekolah dengan hanya `surat` + `website` dan akses langsung ke modul lain ditolak.

## F4 — Mesin proses massal simultan (tanpa Queue/Scheduler)

**Dependensi:** skema/model `OperasiMassal` dan identitas, autentikasi/izin serta UI bersama. Migration F4.01 dibuat pada lapisan skema; engine F4.02–F4.12 sebelum impor F1.07 dan akademik massal. **Digunakan oleh:** impor siswa/alumni, kenaikan, kelulusan, tagihan massal sesuai rilis, tanpa bergantung fitur impor yang belum selesai.

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

**Syarat untuk setiap modul:** K2.10 fondasi rilis target lulus; migration/model rilis sudah terpasang sebelum UI bisnis dibuka. Kemudian selesaikan transaksi bisnis + validasi + role/permission + pemeriksaan lisensi + UI loading/modal + laporan bila dibutuhkan + pengujian proses utama. Modul harus berfungsi sendiri; integrasi PRD Bab 8 **wajib saat kedua modul terkait aktif**; saat pasangan tidak aktif, modul tetap berjalan dengan data minimum Core tanpa menduplikasi master bersama.

| ID F5 | Kode modul | Minimum yang dikerjakan | Role dan alur selesai | Referensi HTML |
|---|---|---|---|---|
| 01 | `website` | CMS halaman, berita, pengumuman, galeri, profil jurusan, **menu/submenu dinamis** (urut/sembunyi/URL) | Editor buat → preview → terbit → pengunjung lihat | `12-website.html` |
| 02 | `surat` | Surat masuk/keluar, nomor unik, template, arsip; **file opsional** (catat metadata/editor/upload) | TU buat/catat → persetujuan seperlunya → arsip | `10-surat.html` |
| 03 | `absensi_izin` | **Presensi siswa per jam pelajaran**, konfirmasi wali kelas, penanganan bersama BK, izin siswa/guru dan penyelarasan per jam | Guru mapel simpan seluruh kelas hari ini → wali kelas konfirmasi tanpa tenggat → tindak lanjut bersama BK; S/I langsung dari wali kelas | `08-absensi.html` |
| 04 | `pelanggaran` | Tata tertib, bobot, catat, validasi, pembinaan, keberatan | Guru catat → Kesiswaan validasi → BK tindak lanjut | `13-pelanggaran.html` |
| 05 | `pkl` | Mitra, penempatan, jurnal, absensi, monitoring, nilai industri **diinput guru dari kertas** | Koordinator tempatkan → siswa jurnal → guru pantau/input nilai → laporan | `07-pkl.html` |
| 06 | `jurnal_guru` | Kegiatan dan hasil pekerjaan harian guru, draf/kirim/koreksi, riwayat dan rekap | Guru tulis kegiatan dan hasil → Kirim → Pimpinan lihat | `16-jurnal-guru.html` |
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
- [ ] F5.05 `pkl` — presensi lokasi siswa masuk/pulang dengan radius, jurnal harian kegiatan/hasil, bantuan guru, serta P0.01–P0.07; nilai industri tetap dari kertas oleh guru tanpa akun industri.
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

## P0 — Presensi lokasi dan jurnal PKL siswa

Checklist ini mengikuti rilis PKL (R3), bukan modul baru; belum membuktikan aplikasi berjalan. Gunakan alur ringkas untuk pengguna non-IT dan bedakan presensi siswa PKL dari presensi sekolah 1 guru–1 kelas.

- [ ] P0.01 Koordinator mengatur lokasi/versi koordinat dan radius awal 100 meter, akurasi dan penempatan sah; siswa tidak dapat mengganti pusat/radius/penempatan atau tanggal tindakan. Uji revisi lokasi tidak mengubah bukti lama.
- [ ] P0.02 Halaman PKL Saya mengutamakan Absen Masuk, Absen Pulang dan Isi Jurnal; ambil lokasi setelah tindakan/izin pengguna, tanpa pemantauan terus-menerus, foto wajib, API peta berbayar atau akun industri.
- [ ] P0.03 Implementasikan token 60 detik, lokasi baru/akurasi valid, jarak Haversine di server dan syarat `jarak + akurasi <= radius`; HTTPS dan Geolocation API. Uji luar radius, akurasi buruk, koordinat invalid, token lama, client memalsukan jarak, serta pesan Indonesia yang menjelaskan langkah berikutnya.
- [ ] P0.04 Simpan masuk/pulang secara transaksional dan idempotent, satu header per siswa/penempatan/hari, satu event efektif per jenis; uji dua tab, double-tap, timeout setelah commit, tanggal server Jakarta dan pulang lewat tengah malam pada hari kerja masuk yang sama.
- [ ] P0.05 Bantuan lokasi/terlewat diperiksa guru berizin tanpa bypass siswa; sumber koreksi verifikasi_petugas, alasan dan versi event tersimpan. Uji sakit/izin tidak membuat koordinat/hadir palsu, penempatan lain ditolak dan modul nonaktif tidak merusak record terbuka.
- [ ] P0.06 Jurnal PKL harian berisi kegiatan dan hasil, draf/kirim/periksa/perbaiki; tidak memerlukan GPS/presensi sukses. Uji tanggal/segmen, privasi siswa lain, koreksi jurnal diperiksa membentuk versi baru dan histori penempatan tetap.
- [ ] P0.07 Uji hak lihat koordinat/ringkasan/ekspor dan kebijakan retensi manual; owner/publik/role salah tidak mendapat lokasi rinci. Tidak menyimpan lokasi kontinu, localStorage atau log umum; bukti Core read-only tetap milik pengguna berhak.

## M0 — Penyesuaian kebutuhan SMK dan kemudahan penggunaan

Ini penajaman modul yang ada, bukan tambahan role/modul; presensi sekolah tetap 1 guru–1 kelas tanpa pembagian kelompok.

- [ ] M0.01 Master konsentrasi opsional serta penugasan kepala program/konsentrasi memakai role Pimpinan yang ada; uji lingkup, program induk, snapshot nama dan histori perpindahan.
- [ ] M0.02 Daftar kelas membedakan sasaran sekolah dan konteks Sedang PKL tanpa mengubah rombel. Uji Semua Hadir tidak mengisi peserta PKL, snapshot/pengecualian sah dan satu transaksi kelas tetap lengkap; konteks dasar dapat dicatat di Core tanpa membuka fitur PKL berbayar.
- [ ] M0.03 Jadwal dan Lab memakai master/pemakaian ruang bersama; uji bentrok lintas modul/paralel, taut kegiatan sama dan pemakaian sah tidak hilang saat modul nonaktif.
- [ ] M0.04 Stok bahan per lokasi, hak unit dan transfer keluar/masuk atomik; uji stok kurang, gagal satu sisi, transfer ulang dan pasangan pembalik tanpa saldo minus.
- [ ] M0.05 BKK mencatat penyerahan kandidat ke PIC berikut berkas/kanal/waktu dan sumber persetujuan tujuan lamaran; uji ekspor data minimum, versi kirim ulang dan Terkirim tidak berarti Diterima kerja. Tanpa kirim otomatis/akun industri.
- [ ] M0.06 Tracer memakai snapshot sasaran periode, termasuk alumni belum berakun; uji penyebut cakupan, status jangkauan, sasaran nol, revisi daftar dan jawaban efektif tanpa duplikasi.
- [ ] M0.07 UAT pengguna non-IT sesuai PRD §3.4: setup bertahap, template impor, isian minimum, mobile, tombol jelas, pesan solusi dan status simpan nyata; jurnal guru dan siswa cukup kegiatan/hasil, lokasi hanya presensi PKL.

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

**Skenario penerimaan terbaru:** OP01–OP13 pada PRD §14.7, AU01–AU21 pada §14.8 dan SM01–SM11 pada §14.9 wajib diuji sesuai rilis. Prototipe tema bukan bukti CMS produksi selesai.

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

1. Baca PRD §3.5/§14.1 serta bagian Urutan pelaksanaan per lapisan dan peta file pada TASK. Kerjakan prasyarat ID yang tersedia, bukan seluruh F0 atau seluruh modul dalam satu perubahan besar.
2. Periksa keadaan repository: jika proyek Laravel belum ada, scaffold di root **tanpa menghapus** dokumentasi/prototipe. Jika sudah ada, jangan scaffold ulang.
3. Sebelum coding, rangkum target task ID, file yang akan disentuh, desain singkat, dan tes yang perlu lulus.
4. Pada fondasi, selesaikan satu kelompok lapisan yang koheren beserta tesnya: route/kontrak, migration, Model, keamanan atau UI. Setelah gerbang fondasi lulus, kerjakan fitur sampai transaksi/UI/test tuntas. Jangan membuat stub seluruh fitur hanya untuk memenuhi daftar file.
5. Jalankan test yang memungkinkan, termasuk `php artisan test` dan `npm run build` setelah aplikasinya tersedia; laporkan jika tool/database lokal tidak ada, jangan mengarang hasil lulus.
6. Perbarui kotak di `TASK.md` **hanya bila** kode dan pengujiannya benar-benar selesai, beserta bukti/commit atau ringkasan di bagian Catatan progres implementasi.
7. Berikan ringkasan hasil, perintah uji, risiko, dan langkah konkret berikutnya. Jangan klaim production-ready tanpa melewati bagian Checklist kesiapan rilis aplikasi.

## Panduan prioritas

Gunakan satu urutan kanonik pada bagian **Urutan pelaksanaan per lapisan**. Nomor F/K adalah ID stabil; peta file menunjukkan tanggung jawab, bukan instruksi membuat semua file sekaligus. Fondasi rilis harus lulus sebelum fitur operasional, lalu QA/handover dilakukan pada setiap rilis R1–R4. Jangan mengulang daftar fase F0→F1 yang mengabaikan prasyarat akun/model/lisensi.

**Catatan:** tugas di repositori ini mencakup software development saja. Tim lain menangani penayangan dan operasi infrastrukturnya.


---

## Panduan implementasi


**Versi:** 1.3 · **Acuan:** `PRD.md` · **Checklist:** `TASK.md` · **Prototype UI:** `prototype/index.html`, halaman `01-owner.html` s.d. `16-jurnal-guru.html`, dan lima tema publik

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
- Kerjakan dan uji kontrak URL/DB/keamanan menurut tabel Urutan pelaksanaan per lapisan dan peta file pada TASK; migration/model mendahului auth/UI dan gerbang K2.10 mendahului modul operasional. `TASK.md` hanya dicentang setelah ada implementasi dan bukti test, bukan berdasarkan teks PRD atau demo HTML.

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
- Role dapat rangkap, misalnya Guru dan Pimpinan. Wali kelas/pembimbing PKL adalah penugasan pada role yang ada, bukan role tambahan. Batas akses berdasarkan **fungsi + objek yang ditugaskan**.
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

### 7. Urutan implementasi (gunakan ID checklist dan prasyarat lapisan)

Gunakan bagian **Urutan pelaksanaan per lapisan** serta **Peta file implementasi dan kepemilikan data** pada TASK sebagai urutan kanonik. Mulai F0.01–F0.03/K0.07, lalu kontrak K1.08 → migration → Model K1.09 → keamanan/lisensi → UI → Core/massal → K2.10 → fitur rilis → QA/UAT/upgrade/handover. Bagian F/K dipakai lintas lapisan; jangan menunggu semua F0 selesai untuk membuat tabel pengguna F1. F4 mendahului F1.07 dan akademik massal. R1 hanya Core + Presensi/Izin + Jurnal; R2–R4 mengikuti PRD Bab 10.

**Untuk setiap lapisan/fitur:** tampilkan ID/cakupan dan daftar file aktual; perbarui peta jika nama/pecahan file berubah; jalankan tes yang membuktikan tanggung jawab lapisan; catat hasil/risiko. Jangan buka route/menu bisnis sebelum handler, skema, otorisasi dan perilakunya tersedia; jangan centang F5 karena hanya model/UI tersedia. Bukti gerbang fondasi dan produksi dipisahkan.

### 8. Minimal test dan release gate

Harus diuji sesuai cakupan rilis dan regresi seluruh fitur yang telah terpasang; modul masa depan yang belum dipasang dicatat Belum masuk rilis. Sekurang-kurangnya:

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

**Mulai dari F0.01–F0.03/K0.07, lalu prasyarat per lapisan pada tabel kanonik.** Jangan membangun 15 modul sekaligus dan jangan mengarang hasil testing atau membuat data produksi palsu.

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

- [ ] C01 PKL: siswa login untuk absen lokasi berbatas jarak dan jurnal kegiatan/hasil; nilai industri dari kertas tetap diinput guru, tanpa akun industri dan tanpa foto wajib.
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
| Peta route/file dan kontrak skema — K0.07/K1.08 | Inventaris lengkap T1–T10 ditetapkan; belum diimplementasikan/diverifikasi pada Laravel | Kamus PRD §12.10–12.11 dan inventaris TASK; belum bukti implementasi | — |
| Migration/model dan gerbang fondasi — K1.09/K2.10 | Belum dimulai | — | — |
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

## Bukti revisi kebutuhan SMK PRD 4.9

- Presensi sekolah tetap 1 guru–1 kelas tanpa pembagian kelompok. PKL siswa ditambah Absen Masuk/Pulang berbasis lokasi saat tindakan, radius dan bantuan guru, serta jurnal kegiatan/hasil yang tidak bergantung GPS.
- Program/konsentrasi dan kepala program memakai penugasan yang ada; ruang Jadwal–Lab, stok per lokasi, penyerahan kandidat BKK dan sasaran tracer dirinci. Kriteria penggunaan non-IT berada di PRD §3.4 dan SM01–SM11.
- Desain mandiri `prototype/15-pkl-siswa.html` dapat dibuka melalui indeks dan halaman PKL. Hanya data contoh dalam memori halaman; tidak ada login/backend atau kiriman pesan/lokasi ke sekolah pada prototipe.
- Pengujian aktual prototipe dengan Playwright/Chromium dan lokasi sintetis: 14 skenario lulus, mencakup masuk/pulang sah, penolakan luar/dekat batas area, akurasi buruk, lokasi ditolak/tidak tersedia/timeout/kedaluwarsa/invalid, jurnal tanpa lokasi, serta tampilan dan bantuan pada lebar 320/390/1440px. Tidak meminta lokasi nyata saat uji.
- Validasi dokumen: 186 ID task bertitik unik, 13 role, 15 modul, rilis R1–R4, OP01–OP13/AU01–AU21/SM01–SM11 serta tautan lokal konsisten. Seluruh 21 HTML hanya memakai file HTML pada folder prototype; 34 blok JavaScript valid secara sintaks.
- Ini bukti desain/dokumen dan simulasi browser; transaksi MariaDB, login, radius server, pergantian lisensi dan penyimpanan lokasi nyata belum diimplementasikan. Task aplikasi tetap tidak dicentang; UAT pengguna non-IT nyata tetap wajib setelah aplikasi tersedia.

## Bukti pembaruan prototipe sesuai PRD 4.9 — 10 November 2026

- Presensi sekolah menjadi satu guru–satu kelas per jam dengan tanggal hari ini, satu penyimpanan seluruh kelas, bukti lengkap, contoh gagal/koneksi putus/konflik data, izin S/I wali langsung, pembatalan tanpa otomatis H, pengecualian siswa PKL, serta konfirmasi wali tanpa tenggat. Tampilan ponsel memakai kartu siswa dengan pilihan H/S/I/A satu baris.
- `16-jurnal-guru.html` menampilkan kegiatan dan hasil harian, beberapa kegiatan, kelas/mapel opsional, draf, kirim, tinjauan dan revisi yang mempertahankan isi versi sebelumnya. Tidak bergantung jadwal/presensi/kalender.
- Kenaikan mengikuti pengecualian yang dipilih; kelulusan mencontohkan tingkat akhir XII/XIII; filter/tambah/pindah siswa menampilkan perubahan dan riwayat contoh. Verifikasi alumni mempertahankan status keputusan. Nilai industri PKL menyimpan versi dan alasan revisi; radius menampilkan batas akurasi sesuai PRD.
- Keuangan memperlihatkan penyesuaian, void administratif, pengembalian manual, pembatalan setelah penerimaan nol, dan pembalik yang memulihkan tagihan bersama penyesuaiannya. Surat membedakan nomor internal/eksternal, final/revisi/batal. Sarpras memperlihatkan pengembalian sebagian/rusak, stok per lokasi dan bentrok ruang. Pelanggaran memakai bobot saat kejadian dan versi koreksi, tanpa pilihan prestasi.
- CMS menyediakan lima pilihan tema, draf yang tidak menimpa versi publik, serta perpindahan/visibilitas submenu mengikuti induk. Seluruh HTML dirapikan; tema diperbaiki pada lebar 320px. Katalog modul lain memberi detail alur sesuai PRD dan batas rilis R1–R4 tetap dijelaskan.
- Kode modul demo memakai `jurnal_guru`, `absensi_izin` dan `sarpras` dengan migrasi pilihan lama. Mode tinjau desain dibedakan dari simulasi akses sekolah; modul nonaktif/lisensi berakhir tidak menghapus data contoh atau merusak layanan inti. Lab dapat aktif sendiri, layanan alumni bersama mengikuti BKK atau Tracer. Proses tertunda tidak menampilkan sukses setelah akses dicabut.
- Validasi aktual: 22 HTML, 40 blok JavaScript lulus pemeriksaan sintaks, tautan lokal tersedia, dan hanya HTML berada di folder `prototype/`. Dua dokumen Markdown tetap `PRD.md` dan `TASK.md`; 186 ID tugas unik dan tidak ada task aplikasi dicentang selesai.
- Chromium/Playwright: 18 kelompok pengujian utama lulus, termasuk seluruh 22 halaman pada 320/390/1440px, navigasi ponsel, ID elemen unik, tanpa overflow halaman atau galat JavaScript, dan alur transaksi simulasi di atas. Lima pemeriksaan tambahan lulus untuk penghentian/kelanjutan batch, struktur submenu, fokus modal dan pembatalan absen PKL tertunda. Empat belas skenario lokasi/jurnal/bantuan PKL sebelumnya diuji ulang dan lulus memakai lokasi sintetis. Nilai PKL berversi, radius/akurasi, kelulusan XII/XIII dan tombol presensi ponsel juga diperiksa; tangkapan layar presensi, jurnal guru dan CMS ditinjau.
- **Batas bukti:** status proses dan arsip pada desain adalah simulasi browser, sebagian hanya selama halaman terbuka. Mode akses bukan autentikasi atau kontrol lisensi server. Belum membuktikan transaksi atomik MariaDB, keamanan role, persistensi, deployment atau kesiapan produksi. Peta HTML representatif ada pada PRD; layar yang belum dibuat tetap mengikuti task aplikasi sesuai rilis.

## Bukti revisi fondasi per lapisan — 11 November 2026

- PRD 4.10 menegaskan tanggung jawab PRD/TASK, kontrak arsitektur §3.5 dan migration lintas rilis §12.1.2. Kebutuhan bisnis, 13 role, 15 kode modul dan cakupan R1–R4 tetap. Metadata berkas/master ruang dan administrasi langganan SaaS diperinci untuk mendukung fitur yang sudah disepakati.
- TASK 1.9 memiliki satu urutan kanonik per lapisan, peta route/file Core + katalog modul, kelompok migration/model fondasi R1, kepemilikan master dan file utama R2–R4. Empat tugas K0.07/K1.08/K1.09/K2.10 menuntut inventaris dan pembuktian fondasi, bukan file kosong.
- Dependensi auth–model, lisensi–navigasi, F4–impor/akademik dan FK lintas rilis diperjelas. F2.14 tidak mewajibkan tabel Keuangan R4 untuk memulai R1. Bagian panduan/prioritas/prompt mengacu urutan yang sama; tugas fitur tetap selesai setelah alur pengguna sampai database dan tes lulus.
- Pemeriksaan dokumentasi: 190 ID tugas bertitik unik dengan referensi valid, cakupan modul/rilis konsisten, dependensi berarah tanpa siklus, urutan induk–anak migration serta peta komponen/route diperiksa. Tidak ada task aplikasi dicentang atau klaim produksi dinaikkan.
- Perubahan ini hanya spesifikasi dan rencana implementasi. Laravel/MariaDB belum dibangun; belum ada bukti boot, migration, feature/security/concurrency test, build, upgrade atau UAT aplikasi nyata. Gerbang fondasi dan produksi tetap BELUM DIUJI / NO-GO sampai pekerjaan dan buktinya tersedia.

## Bukti penutupan rincian spesifikasi — 11 November 2026

- PRD4.11/TASK2.0 mengganti peta minimum menjadi kamus fisik dan inventaris teknis:144 tabel bisnis,150 migration bisnis termasuk6 FK additive,141 Model, 131 halaman,59 route HTTP/transport, 240 aksi,18 keluaran laporan, 33 suite integrasi, dan seluruh 124 ID penerimaan dipetakan ke file/method test. Nama/type/FK/UNIQUE/status/relasi, field input, pemilik transaksi, hak/lingkup, file view/handler dan kegagalan kritis dinyatakan.
- Keputusan Fortify/TOTP, endpoint Livewire4, class-based component, library impor/PDF, baseline file migration, standar lock/idempotensi, session/upload/export/batch dan retensi ditetapkan. Hak khusus uang/pengesahan tidak otomatis melekat pada role. Mitra/alat tetap mandiri terhadap modul pasangan; token/GET/progres tidak menyembunyikan mutasi bisnis.
- Referensi API resmi yang dibaca: [route Fortify](https://github.com/laravel/fortify/blob/1.x/routes/routes.php) dan [HandleRequests Livewire4](https://github.com/livewire/livewire/blob/4.x/src/Mechanisms/HandleRequests/HandleRequests.php). Ini verifikasi rancangan adapter, bukan bukti dependency sudah terpasang atau test Laravel telah lulus.
- Pemeriksaan spesifikasi: coverage seluruh kode modul/ID penerimaan, kecocokan inventaris ke kamus, DAG migration dan FK lintas rilis, owner method Model, nama/URL unik, status/default dan konsistensi tabel Markdown. Hanya PRD/TASK diperbarui; tidak membuat source Laravel, tidak mengubah prototipe, tidak mencentang task aplikasi.
- **Status produksi tetap BELUM SIAP / NO-GO.** Kelengkapan spesifikasi berarti kebutuhan disepakati terpetakan, bukan semua kode/test/UAT/performa/upgrade/keamanan sudah terbukti. Nilai rahasia/host/data sekolah/hasil uji nyata tetap diisi melalui proses implementasi/deployment, bukan direka pada dokumen.
