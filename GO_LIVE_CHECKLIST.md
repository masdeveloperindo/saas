# GO_LIVE_CHECKLIST.md — Kesiapan Aplikasi SaaS SMK

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
- [ ] A04 `students.id` permanen; riwayat kelas per tahun tersimpan dan tidak overlap; FK dan UNIQUE di MariaDB diuji.
- [ ] A05 Seed awal hanya role/permission dan 15 katalog modul, **tanpa** password owner default.
- [ ] A06 Semua route Laravel yang dijanjikan ada, memakai named routes, tidak ada domain hardcoded, dan mutasi data tidak dilakukan melalui GET.
- [ ] A07 Aplikasi dapat berjalan tanpa queue worker, Scheduler, Redis, atau server lain.

## 3. Gerbang B — Siklus siswa, alumni, dan keuangan (BLOCKER)

- [ ] B01 Impor 600 siswa sintetis: pratinjau, validasi, jumlah sukses/gagal, tidak ada duplikat NIS/NISN non-null.
- [ ] B02 Pindah kelas di tengah tahun mempertahankan histori kelas asal dan tanggal efektif.
- [ ] B03 Kenaikan kelas massal X→XI→XII berhasil; bisa pilih semua dari filter, pengecualian, tinggal kelas, dan preview.
- [ ] B04 Program empat tahun mendukung XII→XIII, sedangkan program tiga tahun tidak memaksakan XIII.
- [ ] B05 Tahun ajaran baru tidak otomatis menaikkan siswa; siswa belum diproses tetap terdaftar dan terhitung.
- [ ] B06 Kelulusan disetujui operator berwenang; tidak otomatis karena tahun berubah. Akun siswa yang lulus berlanjut sebagai alumni bila ada.
- [ ] B07 Alumni lama registrasi mandiri, dicegah klaim identitas orang lain, diverifikasi Admin BKK, tidak perlu siswa fiktif.
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
- [ ] C02 Absensi hanya siswa; izin siswa **dan** guru dalam satu modul; tidak ada absensi guru.
- [ ] C03 Surat bisa dicatat/disimpan tanpa file dan tetap bisa dibuat/upload dokumen bila dibutuhkan.
- [ ] C04 Sarpras dan Lab memakai `assets.id` sama; stok bahan pakai tidak negatif; peminjaman unit tidak ganda.
- [ ] C05 Hubin–PKL dan BKK–Tracer berbagi master data, tidak ada identitas perusahaan/alumni ganda.
- [ ] C06 CMS: menu/submenu dinamis, urutan, status tampil, URL internal/eksternal, perlindungan dari HTML/script berbahaya.
- [ ] C07 Modul nonaktif/expired tersembunyi **dan** ditolak di URL/action/export; data tidak terhapus.

## 5. Gerbang D — Keamanan aplikasi (BLOCKER)

- [ ] D01 Login, logout, reset password, rate limiting, regenerasi session, dan CSRF diuji.
- [ ] D02 MFA untuk owner `/owner` diuji; akun sekolah tidak dapat memperoleh hak owner.
- [ ] D03 Policy/permission berbasis objek diuji pada route, aksi Livewire, download, ekspor, dan batch; uji akses dengan ID objek yang bukan milik role.
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
- [ ] F05 File `.env.example` aman; README aplikasi; named route map; daftar extension PHP yang dibutuhkan; urutan command build/migrate tersedia, **tanpa konfigurasi VPS/domain/Nginx/SSL**.
- [ ] F06 Prosedur rollback **aplikasi** dijelaskan dan diuji pada database sintetis; tidak menggunakan operasi destruktif di produksi.
- [ ] F07 Developer menyerahkan informasi data apa yang perlu dibackup dan cara aplikasi memeriksa hasil restore; pekerjaan backup/restore host oleh operator terpisah.
- [ ] F08 Tidak ada Queue, Scheduler, job worker, webhook payment, atau integrasi pemerintah yang diam-diam menjadi syarat utama.
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
