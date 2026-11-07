# Mulai Proyek dengan Codex

Repository ini disiapkan agar Codex membangun aplikasi Laravel **dari awal**. Saat pertama diunggah, belum ada `artisan`, `composer.json`, migration implementasi, atau database aktif; yang ada adalah spesifikasi dan demo HTML.

## Langkah penggunaan GitHub + Codex Cloud

1. Ekstrak ZIP, unggah **seluruh isi folder `saas-smk`** ke root repository GitHub (sebaiknya repository **Private**).
2. Pastikan `AGENTS.md`, `PRD.md`, `task.md`, `promt.md`, `GO_LIVE_CHECKLIST.md`, dan `prototype/` tampak di GitHub. Jangan unggah `.env`, password atau data asli sekolah.
3. Buka Codex, pilih repository tersebut di environment GitHub yang terhubung. Siapkan runtime pengembangan yang memenuhi versi PHP dan MariaDB untuk mengetes aplikasi.
4. Mulai dari prompt **TUGAS PERTAMA** di bawah. Setelah Codex membuat perubahan, tinjau diff, hasil tes, dan commit/PR sebelum tugas berikutnya.
5. Ulangi per fase. Jangan meminta 15 modul selesai dalam satu prompt.

## TUGAS PERTAMA — salin ke Codex

```text
Baca AGENTS.md, PRD.md, task.md, promt.md, GO_LIVE_CHECKLIST.md, dan prototype/index.html.
Repository saat ini baru berisi dokumen dan prototipe HTML, bukan proyek Laravel.
Kerjakan hanya F0.01–F0.03 dan persiapan K0/K1 yang benar-benar diperlukan:
(1) audit kesiapan repo dan runtime; (2) scaffold Laravel 13 di root tanpa menghapus file dokumentasi/prototype;
(3) pasang Laravel Livewire Starter Kit yang kompatibel dengan Livewire 4 + Tailwind CSS 4;
(4) atur konfigurasi MariaDB melalui .env.example yang bebas secret;
(5) buat struktur modular monolith untuk Core dan katalog 15 modul tanpa CRUD semu;
(6) jalankan tes default dan build aset bila environment mendukung;
(7) tulis hasil, blocker, versi aktual, dan tindakan berikutnya ke CODEX_PROGRESS.md.
JANGAN buat fitur lain, JANGAN konfigurasikan VPS/domain/DNS/Nginx/SSL,
JANGAN gunakan Queue/Scheduler, dan JANGAN menyatakan aplikasi siap produksi.
Tandai task.md hanya untuk pekerjaan yang sudah ada dan diuji. Jika toolchain PHP 8.5/MariaDB
belum tersedia, jelaskan hambatan dan jangan pura-pura tes berhasil.
```

## TUGAS BERIKUTNYA — setelah F0.01–F0.03 selesai

### Sprint 2: Auth dan UI dasar

```text
Baca AGENTS.md dan CODEX_PROGRESS.md. Kerjakan F0.04–F0.10 serta K0/K2 terkait.
Implementasikan autentikasi aman, owner lokal terpisah dari admin sekolah, policy awal,
layout responsif, komponen modal/toast/loading reusable. Jangan gunakan data nyata.
Tambahkan tes akses/role dasar, jalankan tes dan build, lalu perbarui status beserta bukti.
```

### Sprint 3: Master data siswa

```text
Baca PRD.md desain siswa dan task F1. Implementasikan migrasi, relasi, FK/UNIQUE,
CRUD sederhana dan histori kelas/tahun ajaran. Tulis tes untuk identitas permanen,
impor duplikat, pergantian tahun ajaran tidak otomatis menaikkan kelas,
dan pindah kelas historis. Jangan membuat kenaikan massal sebelum mekanisme F4 siap.
```

### Sprint 4: Mesin operasi massal

```text
Kerjakan F4 sesuai PRD: snapshot target, preview, modal konfirmasi, batch HTTP paralel
terbatas, progress dari status commit database, pause/resume/retry dan idempotensi.
Uji 600 siswa sintetis, dua tab, koneksi putus, timeout, dan pencegahan duplikasi.
Tanpa Queue/Scheduler, tanpa worker background.
```

### Sprint 5: Kenaikan, kelulusan, alumni, dan lisensi

```text
Kerjakan F2/F3 secara berurutan setelah dependensi F1/F4 selesai.
Uji kelas X-XI-XII/XIII, siswa tertunda/tinggal kelas, alumni lama daftar sendiri,
riwayat alumni dapat tumpang tindih, invoice histori tidak berubah, dan modul
expired diblokir di UI dan backend. Sertakan bukti test per kriteria PRD.
```

### Sprint modul dan rilis

Setelah Core, `task.md` menentukan tahapan implementasi setiap modul. Jalankan pengembangan modul secara bertahap. Setelah setiap fase, tinjau diffs, commit/PR, dan tes. `GO_LIVE_CHECKLIST.md` **bukan** diisi otomatis hanya karena kode sudah ditulis.

## Perintah yang relevan setelah Laravel benar-benar tersedia

```bash
composer install
npm install
php artisan migrate --env=testing  # hanya setelah DB testing dikonfigurasi dengan aman
php artisan test
npm run build
```

Jangan menjalankan migration ke database nyata tanpa pemeriksaan konfigurasi dan backup oleh pihak berwenang. `composer install` dan perintah PHP memerlukan toolchain runtime di environment tempat Codex berjalan.

## Batasan

- Prototipe HTML adalah acuan UI; bukan aplikasi operasional.
- Satu sekolah = satu VPS/instalasi/database; tidak ada panel pusat atau multi-tenant.
- Pekerjaan developer hanya aplikasi, database, QA, dan handover; infrastruktur di luar scope.
- Jika task sudah selesai di commit sebelumnya, verifikasi hasilnya alih-alih membuat ulang.
