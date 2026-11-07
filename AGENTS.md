# Instruksi Codex — Proyek SaaS SMK

Baca file ini **sebelum mengubah kode**. Ini repository proyek aplikasi sekolah, **belum** aplikasi Laravel saat paket awal diunggah. Kerjakan pekerjaan secara bertahap dan beri bukti hasil uji; jangan menyatakan fitur selesai karena sudah ada mockup.

## Sumber kebenaran (urut prioritas)

1. `PRD.md` — fungsi, alur, arsitektur data, batasan scope, keamanan, dan kriteria penerimaan.
2. `task.md` — urutan pekerjaan, ID tugas, dan Definition of Done.
3. `GO_LIVE_CHECKLIST.md` — bukti kelulusan rilis aplikasi, bukan klaim server sudah siap.
4. `promt.md` — instruksi implementasi tambahan.
5. `prototype/` — acuan tampilan dan interaksi **simulasi saja**, bukan kode backend.

Jika ada kontradiksi, utamakan PRD; beri catatan konflik pada laporan progres. Jangan mengikuti nama berkas PRD lama yang tercetak pada sebagian mockup; dokumen otoritatif adalah `PRD.md` di root.

## Arsitektur dan scope tetap

- Laravel **13**, PHP **8.5**, Livewire **4**, Blade, Tailwind CSS **4**, MariaDB; Nginx hanya target runtime milik tim infrastruktur.
- **Satu instalasi Laravel + satu database per sekolah**, seluruh 15 modul dalam satu codebase. **Tidak** multi-tenant, tidak ada domain/API/panel SaaS pusat.
- Owner SaaS lokal di `/owner`; admin operasional sekolah di `/admin`. Lisensi setiap modul dikendalikan lokal.
- **Dilarang menambahkan Laravel Queue, Scheduler, Redis worker, cron aplikasi, WebSocket, microservices, PPDB, UKK, absensi guru, payment gateway, atau akun login pembimbing industri PKL.**
- Pengaturan VPS, domain, DNS, Nginx, TLS, firewall, monitoring server, dan backup host **di luar tanggung jawab developer aplikasi**. Developer hanya menyiapkan kode dan kontrak handover aplikasi.
- Jangan menurunkan versi stack secara diam-diam. Jika versi yang dipersyaratkan tidak tersedia di environment tugas, jelaskan hambatannya dan lanjutkan bagian yang bisa diverifikasi secara jujur.

## Persyaratan integritas data yang harus dijaga

- `students.id` permanen. Riwayat kelas dan tahun ajaran dalam tabel histori; kenaikan kelas dan kelulusan terpisah, termasuk program 4 tahun dengan kelas XIII.
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

1. Mulai dengan membaca `CODEX_START.md` dan tugas F0/K0 pada `task.md`. Jangan membuat seluruh modul dalam satu perubahan besar.
2. Periksa keadaan repository: jika proyek Laravel belum ada, scaffold di root **tanpa menghapus** dokumentasi/prototipe. Jika sudah ada, jangan scaffold ulang.
3. Sebelum coding, rangkum target task ID, file yang akan disentuh, desain singkat, dan tes yang perlu lulus.
4. Kerjakan satu irisan fungsional yang selesai dan dapat diuji; tambahkan migrations, Policies, komponen Livewire, serta tes yang relevan.
5. Jalankan test yang memungkinkan, termasuk `php artisan test` dan `npm run build` setelah aplikasinya tersedia; laporkan jika tool/database lokal tidak ada, jangan mengarang hasil lulus.
6. Perbarui kotak di `task.md` **hanya bila** kode dan pengujiannya benar-benar selesai, beserta bukti/commit atau ringkasan di `CODEX_PROGRESS.md`.
7. Berikan ringkasan hasil, perintah uji, risiko, dan langkah konkret berikutnya. Jangan klaim production-ready tanpa melewati `GO_LIVE_CHECKLIST.md`.

## Panduan prioritas

- F0: fondasi aplikasi dan autentikasi.
- K0–K2: routing, integritas database, dan keamanan sejalan dengan fondasi.
- F1: master data dan riwayat penempatan siswa.
- F4 lalu F2: mesin operasi massal sebelum kenaikan kelas/kelulusan massal.
- F3: lisensi lokal dan panel owner.
- F5–F6: modul, bertahap menurut ketergantungan.
- F7–F9: QA, kriteria kesiapan aplikasi, dan handover.

**Catatan:** tugas di repositori ini mencakup software development saja. Tim lain menangani penayangan dan operasi infrastrukturnya.
