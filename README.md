# SaaS SMK — Repository untuk GitHub + Codex

**Status:** Paket dokumentasi implementasi dan prototipe frontend; **belum ada aplikasi Laravel**.  
**Acuan fitur:** `PRD.md` v4.6 — **Core System + 15 modul**.  
**Scope:** hanya software development. Infrastruktur server, domain, DNS, Nginx, SSL, firewall, serta operasi backup server bukan pekerjaan developer aplikasi.

## Langkah awal (paling penting)

1. Baca [`CODEX_START.md`](CODEX_START.md) untuk cara mengunggah ke GitHub dan contoh prompt pertama untuk Codex.
2. Codex akan membaca [`AGENTS.md`](AGENTS.md) sebagai instruksi proyek.
3. Prioritaskan [`PRD.md`](PRD.md), lalu [`task.md`](task.md), [`promt.md`](promt.md), dan [`GO_LIVE_CHECKLIST.md`](GO_LIVE_CHECKLIST.md).
4. Buka [`prototype/index.html`](prototype/index.html) untuk melihat 15 halaman mockup/interaksi (14 layar + indeks); **semua memakai data simulasi**.
5. Update [`CODEX_PROGRESS.md`](CODEX_PROGRESS.md) setelah setiap fase, gunakan bukti pengujian, bukan perkiraan.

## Struktur repository sebelum coding

```text
saas-smk/
├── AGENTS.md                   # Dibaca Codex otomatis
├── README.md                   # Petunjuk repository
├── CODEX_START.md              # Cara memulai tugas Codex
├── CODEX_PROGRESS.md           # Catatan progres berbukti
├── PRD.md                      # Fitur/role/proses bisnis/database/URL
├── task.md                     # 136 checklist implementasi
├── promt.md                    # Master prompt implementasi
├── GO_LIVE_CHECKLIST.md        # Gate QA dan release aplikasi
├── .gitignore
├── .gitattributes
└── prototype/                 # HTML/CSS/JS simulasi + preview
    ├── index.html
    ├── 01-owner.html ... 14-modul-lain.html
    ├── assets/
    └── preview/
```

**Belum ada folder `app/`, file `artisan`, atau `composer.json` sebelum Codex melakukan F0.** Jangan menganggap README ini sebagai bukti bahwa Laravel sudah terinstal.

## Unggah ke GitHub

Buat repository kosong, sebaiknya private, bernama `saas-smk`. Ekstrak ZIP dan unggah semua isi folder ke root repository lewat UI GitHub atau perintah Git berikut:

```bash
git init
git add .
git commit -m "docs: initial SaaS SMK PRD and Codex setup"
git branch -M main
git remote add origin https://github.com/USERNAME/saas-smk.git
git push -u origin main
```

Ganti `USERNAME` sesuai nama akun/organisasi GitHub. Jika repository sudah ada, sesuaikan perintah dan jangan menimpa pekerjaan yang telah disimpan di sana.

## Jalankan dengan Codex

Pilih repository GitHub di Codex Cloud, siapkan/publikasikan environment yang sesuai, lalu tempel prompt pertama dari `CODEX_START.md`. Lakukan per fase, review perubahan dan hasil tes, lalu commit/PR. Untuk Codex lokal, buka directory repository di Codex CLI/aplikasi desktop.

## Aturan arsitektur yang dikunci

- Laravel 13 + PHP 8.5 + Livewire 4 + Blade + Tailwind CSS 4 + MariaDB.
- Satu instalasi dan database untuk satu sekolah (dedicated VPS; infrastruktur di luar scope coding).
- Panel owner lokal `/owner`, panel sekolah `/admin`.
- Core System + 15 modul, lisensi dan pembayaran manual lokal.
- Tidak menggunakan Queue, Scheduler, multi-tenant, domain/API pusat, atau akun industri PKL.
- Siswa dan alumni punya riwayat berkesinambungan; keuangan tidak berubah saat naik/pindah kelas/lulus.
- Operasi massal punya progress database yang nyata, idempotensi, dan retry aman; UI memakai modal/loading.

## Kapan siap produksi?

Belum. Developer harus membangun backend, migration MariaDB, UI Livewire, keamanan, tes otomatis dan UAT sesuai `GO_LIVE_CHECKLIST.md`. Go-live operasional di server tetap diverifikasi tim infrastruktur secara terpisah.
