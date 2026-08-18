# MODUL PEMBELAJARAN PERSIAPAN LKS
## Cloud Computing / DevOps — GitHub Actions (CI/CD) & AWS CloudFormation (IaC)

**Level:** SMK / Persiapan LKS Provinsi–Nasional
**Metode:** Theory → Demonstration → Guided Lab → Independent Lab → Challenge → Evaluation
**Komposisi Materi:** 70% GitHub Actions/CI-CD, 30% AWS CloudFormation

---

## Peta Besar Modul

```text
BAGIAN A — GITHUB ACTIONS (BAB 1–10)
BAGIAN B — AWS CLOUDFORMATION (BAB 11–20)
BAGIAN C — INTEGRASI GITHUB ACTIONS + CLOUDFORMATION (BAB 21–24)
BAGIAN D — SIMULASI LKS (3 Level)
BAGIAN E — RUBRIK PENILAIAN
BAGIAN F — TROUBLESHOOTING GUIDE
BAGIAN G — EVALUASI TEORI + ANSWER KEY
```

### Prasyarat Peserta
Sudah paham: dasar komputer, dasar Linux (cd, ls, cat, nano), dasar jaringan (IP, port, firewall), dasar Git (init, add, commit, push), dasar cloud computing (VM, storage, network as a service), dasar AWS (console, region, IAM sekilas).

Belum wajib paham: GitHub Actions, CloudFormation — inilah fokus modul ini.

> ⚠️ **Peringatan biaya AWS**: Sebagian besar lab di Bagian B menggunakan resource AWS (EC2, VPC) yang **berpotensi menimbulkan biaya** jika dibiarkan menyala. Setiap lab AWS di modul ini WAJIB diakhiri dengan langkah **cleanup** (`aws cloudformation delete-stack` atau delete stack via console). Jangan pernah meninggalkan stack menyala setelah selesai berlatih.

---

# BAGIAN A — GITHUB ACTIONS (CI/CD)

# BAB 1 — DASAR GIT DAN GITHUB

## 1.1 Konsep Inti

| Istilah | Definisi Singkat | Analogi |
|---|---|---|
| **Git** | Version control system, mencatat "riwayat" perubahan file secara lokal | Buku catatan revisi skripsi — tiap simpanan punya tanggal & isi perubahan |
| **GitHub** | Platform hosting online untuk repository Git + kolaborasi | Google Drive-nya kode, tapi punya fitur kolaborasi & automation |
| **Repository (repo)** | Folder project yang dilacak oleh Git | Map/folder proyek |
| **Commit** | Snapshot perubahan beserta pesan penjelas | "Save point" di game — bisa kembali ke titik itu |
| **Branch** | Cabang pengembangan terpisah dari `main` | Jalur alternatif cerita di game — tidak mengganggu jalur utama |
| **Push** | Mengirim commit lokal ke GitHub | Upload |
| **Pull** | Mengambil perubahan terbaru dari GitHub ke lokal | Download/sync |
| **Pull Request (PR)** | Permintaan untuk menggabungkan branch ke branch lain, disertai review | "Ajukan izin gabung jalur cerita ke jalur utama" |
| **Merge** | Proses menggabungkan branch | Menyatukan dua jalur cerita jadi satu |

## 1.2 Alur Kerja Standar

```text
Developer
   │  (edit file)
   ▼
git add . ──► Staging Area
   │
   ▼
git commit -m "pesan" ──► Local Repository
   │
   ▼
git push ──► GitHub Repository
   │
   ▼
Pull Request ──► Review ──► Merge ke main
```

**Kenapa tidak langsung commit ke `main`?**
Karena `main` biasanya adalah kode yang "siap produksi". Perubahan baru dikerjakan di branch terpisah, baru digabung setelah lolos review/CI — supaya `main` selalu dalam kondisi stabil. Prinsip inilah yang nanti jadi alasan utama CI/CD dibutuhkan.

## 1.3 Perintah Dasar

```bash
git init                     # inisialisasi repo Git baru di folder lokal
git clone <url>               # menyalin repo dari GitHub ke lokal
git status                    # melihat file apa saja yang berubah
git add .                     # menambahkan semua perubahan ke staging area
git commit -m "pesan jelas"   # menyimpan snapshot perubahan
git branch                    # melihat daftar branch
git branch feature/x          # membuat branch baru
git checkout feature/x        # pindah ke branch tersebut
git switch feature/x          # cara modern untuk pindah branch
git pull                      # mengambil perubahan terbaru dari remote
git push origin feature/x     # mengirim branch ke GitHub
```

## 1.4 Struktur Branch Umum

```text
main (stabil, siap produksi)
 │
 ├── develop (integrasi fitur-fitur)
 │
 ├── feature/login
 │
 └── feature/github-actions
```

## Latihan Dasar (Warm-up)

1. Buat repository baru di GitHub bernama `lks-devops-prep`.
2. Clone ke lokal: `git clone https://github.com/<username>/lks-devops-prep.git`
3. Buat file `README.md`, isi dengan nama dan asal sekolah.
4. `git add . && git commit -m "init: readme"`
5. `git push origin main`
6. Buat branch baru `feature/profil`, tambahkan 1 baris di README, push, lalu buat Pull Request ke `main` dan merge lewat GitHub.

**Expected Result:** Riwayat commit di GitHub menunjukkan 2 commit, dan ada 1 PR berstatus *Merged*.

---

# BAB 2 — KONSEP CI/CD

## 2.1 Apa itu Continuous Integration (CI)?

CI adalah kebiasaan **menggabungkan dan memeriksa kode secara otomatis dan sesering mungkin** (idealnya setiap ada push), bukan menumpuk perubahan besar lalu digabung sekali dalam sebulan.

**Analogi:** Bayangkan kerja kelompok membuat laporan di Word. Kalau semua orang menyimpan versi masing-masing selama sebulan lalu digabung di hari terakhir — pasti bentrok dan kacau. CI itu seperti "setiap orang wajib menyetor progres tiap hari, dan sistem otomatis mengecek apakah dokumennya masih rapi (build) dan bisa dibaca (test)".

```text
Developer
    ↓ git push
CI Server (GitHub Actions)
    ↓
Build   → apakah kode berhasil "dirakit"?
    ↓
Test    → apakah kode berjalan sesuai harapan?
    ↓
Result  → ✓ lolos / ✗ gagal
```

Jika Test gagal:

```text
Build → Test → ✗ FAILED
```

Pipeline **berhenti** — kode TIDAK boleh lanjut ke tahap deploy. Ini prinsip penting: CI adalah "gerbang kualitas" (quality gate).

## 2.2 Continuous Delivery vs Continuous Deployment

Keduanya disingkat "CD" tapi maknanya beda:

| | Continuous **Delivery** | Continuous **Deployment** |
|---|---|---|
| Definisi | Kode yang lolos CI otomatis **siap** dirilis, tapi rilis ke production butuh **approval manual** | Kode yang lolos CI otomatis **langsung** dirilis ke production tanpa campur tangan manusia |
| Analogi | Makanan sudah matang & siap disajikan, tinggal tunggu pelayan bilang "oke, antar ke meja" | Makanan matang langsung diantar otomatis oleh robot ke meja pelanggan |
| Risiko | Lebih aman, ada checkpoint manusia | Lebih cepat, tapi butuh test yang sangat kuat |

## 2.3 Perbedaan CI dan CD Secara Ringkas

```text
CI  = memastikan KODE BENAR (build + test)
CD  = memastikan KODE SAMPAI KE TUJUAN (delivery/deployment)
```

## 2.4 Kenapa CI/CD Dibutuhkan?

* **Deteksi bug lebih awal** — semakin cepat bug ketemu, semakin murah biaya perbaikannya.
* **Konsisten** — proses build/test/deploy dilakukan mesin, bukan manual, jadi hasilnya selalu sama.
* **Cepat** — rilis fitur baru bisa dalam hitungan menit, bukan minggu.
* **Kolaborasi lebih aman** — banyak developer bisa kerja bareng tanpa takut saling menimpa.

## 2.5 Contoh CI/CD di Dunia Kerja

```text
Developer
     │
     ▼
   GitHub
     │
     ▼
GitHub Actions
     │
     ├── Build
     ├── Test
     ├── Security Scan
     └── Package (Docker Image)
             │
             ▼
        Container Registry
             │
             ▼
          Deployment (server/cloud)
```

## 2.6 Hubungan Git, GitHub, dan GitHub Actions

```text
Git              = alat version control (bisa dipakai tanpa internet)
GitHub           = tempat hosting repo Git + kolaborasi online
GitHub Actions   = "mesin otomatis" milik GitHub yang bereaksi terhadap
                    kejadian di repo (push, PR, dll) untuk menjalankan
                    build/test/deploy
```

**Analogi sederhana:** Git itu buku catatan pribadi, GitHub itu perpustakaan tempat buku itu disimpan dan dibagikan, sedangkan GitHub Actions itu petugas robot di perpustakaan yang otomatis mengecek setiap ada buku baru masuk — apakah rapi, apakah lengkap, lalu menaruhnya di rak yang benar.

## Diagram Utama Bab Ini

```text
Developer
    ↓
GitHub
    ↓
CI
    ↓
Build
    ↓
Test
    ↓
CD
    ↓
Deployment
```

## Kesalahan Umum Pemula

* Mengira CI/CD = "GitHub Actions saja". Padahal CI/CD adalah **konsep/prinsip**, GitHub Actions hanyalah salah satu **tool** untuk mengimplementasikannya (tool lain: Jenkins, GitLab CI, dsb — tidak dibahas mendalam di modul ini).
* Mengira semua project otomatis punya CI/CD begitu di-push ke GitHub. Padahal harus ada file workflow yang didefinisikan manual terlebih dahulu (dibahas di Bab 3–4).

---

# BAB 3 — PENGENALAN GITHUB ACTIONS

## 3.1 Konsep Utama: Event → Workflow → Job → Step → Action → Runner

```text
Event  ──►  Workflow  ──►  Job  ──►  Step  ──►  Action
                                              (dijalankan di atas)
                                                  Runner
```

### 1) Event
**Definisi:** Kejadian yang memicu workflow berjalan.
**Fungsi:** Menentukan "kapan" otomasi dijalankan.
**Contoh:** push ke branch, pembuatan pull request, klik tombol run manual.
**Analogi:** Bel sekolah — begitu bel bunyi (event), murid-murid (workflow) langsung bergerak.
**Kesalahan umum:** Push ke branch yang tidak didaftarkan di `on.push.branches`, sehingga workflow tidak pernah jalan tapi peserta bingung kenapa.

### 2) Workflow
**Definisi:** File YAML yang mendefinisikan keseluruhan proses otomasi.
**Fungsi:** "Resep" lengkap — dari event apa, job apa saja, sampai step-nya.
**Contoh lokasi:** `.github/workflows/ci.yml`
**Analogi:** Rencana acara sekolah lengkap dari pembukaan sampai penutupan.
**Kesalahan umum:** Menaruh file YAML di folder yang salah (harus persis `.github/workflows/`), sehingga GitHub tidak mendeteksinya.

### 3) Job
**Definisi:** Sekumpulan step yang dijalankan bersama di satu runner (mesin virtual).
**Fungsi:** Mengelompokkan pekerjaan sejenis (misal: job "build", job "test").
**Contoh:** Job `build` dan job `test` bisa berjalan paralel atau berurutan.
**Analogi:** Divisi kerja dalam sebuah acara — divisi dekorasi, divisi konsumsi. Tiap divisi punya tugas (step) sendiri-sendiri.
**Kesalahan umum:** Mengira semua job otomatis berurutan — padahal defaultnya **paralel**, kecuali diatur `needs`.

### 4) Step
**Definisi:** Satu tugas individual di dalam job (misal: "install dependency", "jalankan test").
**Fungsi:** Unit kerja terkecil dalam workflow, dieksekusi berurutan dari atas ke bawah dalam satu job.
**Analogi:** Checklist tugas seorang anggota divisi: "pasang meja → pasang taplak → susun kursi".
**Kesalahan umum:** Lupa bahwa step berjalan **berurutan** dalam job yang sama — jika step 1 gagal, step berikutnya (default) tidak dijalankan.

### 5) Action
**Definisi:** Komponen siap pakai (reusable) yang bisa dipanggil dalam step, misalnya `actions/checkout@v4`.
**Fungsi:** Menghindari menulis ulang logika umum (checkout kode, setup Node.js, dll) dari nol.
**Analogi:** Membeli kue jadi dari toko, dibanding membuat kue dari nol setiap kali.
**Kesalahan umum:** Lupa mencantumkan versi (`@v4`), atau salah tulis nama action.

### 6) Runner
**Definisi:** Mesin (virtual machine) tempat job benar-benar dieksekusi.
**Fungsi:** "Tempat kerja" nyata — GitHub menyediakan runner Ubuntu, Windows, macOS (hosted), atau bisa pakai mesin sendiri (self-hosted).
**Analogi:** Ruang kelas tempat divisi bekerja — bisa pinjam ruang sekolah (GitHub-hosted) atau pakai ruang sendiri di rumah (self-hosted).

## 3.2 Lokasi Workflow

Semua file workflow **wajib** berada di:

```text
.github/workflows/
```

GitHub secara otomatis memindai folder ini. File di luar folder ini **tidak akan terdeteksi** sama sekali.

## 3.3 Contoh Struktur Repository

```text
repository/
├── .github/
│   └── workflows/
│       └── ci.yml
├── src/
├── tests/
└── README.md
```

## 3.4 Ringkasan Alur Berpikir

Ketika membaca/menulis workflow, selalu tanyakan 3 hal berurutan:
1. **KAPAN** ini jalan? → lihat `on:` (Event)
2. **DI MANA** ini jalan? → lihat `runs-on:` (Runner)
3. **APA** yang dikerjakan, urut dari atas? → lihat `steps:` (Step & Action)

---

# BAB 4 — WORKFLOW GITHUB ACTIONS (YAML DARI DASAR)

## 4.1 Kerangka Paling Dasar

```yaml
name:
on:
jobs:
```

* `name` → nama workflow, muncul di tab **Actions** di GitHub (opsional, tapi sebaiknya selalu diisi).
* `on` → event pemicu (wajib).
* `jobs` → daftar pekerjaan yang dijalankan (wajib).

## 4.2 Penjelasan Key-Key Penting

| Key | Fungsi |
|---|---|
| `name` | Nama workflow/job/step agar mudah dikenali di UI GitHub |
| `on` | Event pemicu: `push`, `pull_request`, `workflow_dispatch`, dll |
| `jobs` | Kumpulan job |
| `runs-on` | Menentukan runner (mis. `ubuntu-latest`) |
| `steps` | Daftar langkah dalam satu job |
| `uses` | Memanggil sebuah Action siap pakai |
| `run` | Menjalankan perintah shell langsung |

## 4.3 Contoh 1 — Hello World

```yaml
name: Hello World                 # (1) nama workflow di tab Actions

on:
  push:                           # (2) trigger: setiap ada push
    branches:
      - main                      #     hanya untuk branch main

jobs:
  hello:                          # (3) nama job bebas, di sini "hello"
    runs-on: ubuntu-latest        # (4) jalankan di mesin Ubuntu terbaru

    steps:
      - name: Say hello           # (5) nama step, muncul di log
        run: echo "Hello from GitHub Actions"   # (6) perintah shell
```

**Penjelasan baris per baris:**
1. `name: Hello World` — label workflow.
2. `on.push.branches: [main]` — workflow HANYA jalan saat push ke `main`, push ke branch lain diabaikan.
3. `jobs.hello` — mendefinisikan satu job bernama `hello`.
4. `runs-on: ubuntu-latest` — job dijalankan di runner Ubuntu versi terbaru yang disediakan GitHub gratis.
5–6. `steps` berisi satu step yang menjalankan perintah `echo`.

## 4.4 Contoh 2 — Menampilkan Informasi Runner

```yaml
name: Runner Info

on:
  workflow_dispatch:              # trigger manual (tombol "Run workflow")

jobs:
  info:
    runs-on: ubuntu-latest
    steps:
      - name: Show OS info
        run: uname -a             # menampilkan detail sistem operasi runner

      - name: Show current user
        run: whoami                # menampilkan user yang menjalankan job

      - name: Show date
        run: date                  # menampilkan waktu saat job berjalan
```

`workflow_dispatch` dipakai supaya peserta bisa menjalankan workflow **kapan saja lewat tombol di GitHub**, tanpa perlu push — sangat berguna untuk testing dan demo.

## 4.5 Contoh 3 — Checkout Repository

```yaml
name: Checkout Demo

on:
  push:
    branches: [main]

jobs:
  checkout:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4  # mengambil salinan isi repo ke dalam runner

      - name: List files
        run: ls -la                # membuktikan file repo sudah ada di runner
```

**Kenapa perlu `actions/checkout`?** Runner GitHub adalah mesin virtual **kosong** yang baru dibuat setiap kali job berjalan — ia tidak otomatis punya salinan kode kamu. Step `checkout` inilah yang "mendownload" isi repository ke dalam runner tersebut. Tanpa step ini, perintah seperti `npm install` akan gagal karena tidak ada file `package.json`.

## 4.6 Contoh 4 — Menjalankan Linux Command Berantai

```yaml
steps:
  - name: System check
    run: |
      echo "=== OS ==="
      uname -a
      echo "=== Disk ==="
      df -h
      echo "=== Memory ==="
      free -h
```

Tanda `|` (pipe) memungkinkan satu step menjalankan **banyak baris perintah** sekaligus, dieksekusi berurutan dalam satu shell session.

## 4.7 Contoh 5 — Menjalankan Script

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Run custom script
    run: bash scripts/setup.sh
```

Berguna ketika logika terlalu panjang untuk ditulis langsung di YAML — dipisah ke file script tersendiri di repo agar lebih rapi dan mudah ditest secara lokal juga.

## 4.8 Contoh 6 — Build Sederhana (Node.js)

```yaml
name: Build App

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'       # menentukan versi Node.js yang dipakai

      - name: Install dependency
        run: npm install

      - name: Build
        run: npm run build
```

`with:` digunakan untuk memberi **parameter/konfigurasi** ke sebuah action — di sini kita minta action `setup-node` menyiapkan Node.js versi 20.

## 4.9 Contoh 7 — Test Sederhana

```yaml
name: Test App

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - run: npm install
      - run: npm test
```

Jika `npm test` mengembalikan exit code selain 0 (artinya ada test yang gagal), GitHub Actions otomatis menandai step tersebut **failed**, dan job berhenti — step berikutnya tidak dijalankan (default behavior).

---

# BAB 5 — EVENT DAN TRIGGER

## 5.1 Jenis Event Utama

| Event | Kapan Terjadi |
|---|---|
| `push` | Setiap ada commit yang di-push ke repo |
| `pull_request` | Setiap PR dibuat/diupdate |
| `workflow_dispatch` | Dijalankan manual lewat tombol "Run workflow" di GitHub |
| `schedule` | Berjalan otomatis sesuai jadwal cron |

## 5.2 Push Event

```yaml
on:
  push:
```

Tanpa filter tambahan, ini akan trigger di **setiap branch**. Biasanya kita batasi:

```yaml
on:
  push:
    branches:
      - main
```

## 5.3 Pull Request Event

```yaml
on:
  pull_request:
    branches:
      - main
```

Artinya: workflow berjalan setiap kali ada PR yang **ditujukan ke** branch `main` (baik PR baru dibuat maupun di-update dengan commit baru).

## 5.4 Workflow Dispatch (Manual Trigger)

```yaml
on:
  workflow_dispatch:
```

Berguna untuk workflow yang tidak ingin otomatis jalan tiap push (misalnya deployment ke production) — cukup diklik manual saat dibutuhkan.

Bisa juga menerima input dari user:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'staging'
```

## 5.5 Schedule (Cron)

```yaml
on:
  schedule:
    - cron: '0 0 * * *'   # setiap hari jam 00:00 UTC
```

Berguna untuk task rutin, misalnya backup otomatis atau scan keamanan harian.

## 5.6 Kombinasi Beberapa Event

```yaml
on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

  workflow_dispatch:
```

Workflow di atas akan jalan jika: ada push ke `main` **ATAU** ada PR ke `main` **ATAU** ditekan manual.

## LATIHAN

**Tugas:** Buat workflow yang **hanya** boleh berjalan ketika terjadi push ke branch `main`.

```yaml
name: Push Main Only

on:
  push:
    branches:
      - main

jobs:
  demo:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Workflow berjalan karena push ke main"
```

**Expected Result:** Push ke branch lain (misal `dev`) tidak memicu workflow ini sama sekali (cek di tab Actions — tidak ada run baru).

## CHALLENGE

**Tugas:** Workflow harus berjalan ketika Pull Request dibuat menuju branch `main`.

<details><summary>Petunjuk (buka jika stuck)</summary>

Gunakan `on.pull_request.branches: [main]`, lalu buat branch baru, ubah 1 file, dan buka PR ke `main` untuk membuktikan workflow terpicu otomatis.

</details>

---

# BAB 6 — JOB DAN STEP

## 6.1 Job vs Step

* **Job** = sekumpulan step, berjalan di **satu runner** yang sama, secara default **paralel** dengan job lain.
* **Step** = satu tugas individual di dalam job, dieksekusi **berurutan**.

## 6.2 Job Berjalan Paralel (Default)

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Build application"

  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Run test"
```

Tanpa `needs`, job `build` dan `test` akan jalan **bersamaan**, di dua runner terpisah. Cocok kalau keduanya tidak saling bergantung (misalnya `lint` dan `unit-test` yang independen) — mempercepat total waktu pipeline.

## 6.3 Job Berurutan Menggunakan `needs`

```text
Build
  ↓
Test
  ↓
Deploy
```

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."

  test:
    needs: build              # job "test" menunggu job "build" selesai SUKSES
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing..."

  deploy:
    needs: test                # job "deploy" menunggu job "test" selesai SUKSES
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```

## 6.4 Diagram Dependency

```text
Build ──────┐
            ▼
           Test
            ▼
          Deploy
```

## 6.5 Apa yang Terjadi Jika Build Gagal?

Jika job `build` gagal (exit code bukan 0 di salah satu step-nya):

```text
Build ✗ FAILED
  ↓
Test   → TIDAK DIJALANKAN (skipped)
  ↓
Deploy → TIDAK DIJALANKAN (skipped)
```

Job yang menggunakan `needs: build` **otomatis di-skip**, karena syaratnya (`build` sukses) tidak terpenuhi. Ini penting dipahami untuk kompetisi LKS: jika stage awal gagal, jangan heran stage selanjutnya "hilang" dari log — itu bukan bug, tapi memang perilaku normal `needs`.

**Kesalahan umum:** Peserta panik mengira workflow error karena job deploy "tidak muncul", padahal itu memang di-skip otomatis akibat job sebelumnya gagal. Solusinya: cek log job yang gagal duluan.

---

# BAB 7 — GITHUB RUNNER

## 7.1 Apa itu Runner?

Runner adalah **mesin virtual** (VM) tempat job benar-benar dieksekusi. Setiap job mendapat runner yang **fresh/bersih** — dibuat baru dari nol, dan dihapus total setelah job selesai. Ini alasan kenapa kita selalu perlu `actions/checkout` di awal setiap job: runner tidak "mengingat" apapun dari job sebelumnya.

## 7.2 Jenis Runner

| Jenis | Keterangan |
|---|---|
| **GitHub-hosted** | Disediakan gratis oleh GitHub (dengan kuota tertentu), auto-provisioned, auto-dihapus setelah job selesai |
| Ubuntu | `runs-on: ubuntu-latest` — paling umum dipakai |
| Windows | `runs-on: windows-latest` |
| macOS | `runs-on: macos-latest` |
| **Self-hosted** | Mesin milik sendiri (fisik/VM/cloud) yang didaftarkan sebagai runner, cocok untuk kebutuhan khusus (hardware spesifik, akses jaringan internal, biaya jangka panjang lebih murah) |

**Analogi:** GitHub-hosted runner itu seperti sewa ruang meeting hotel — datang, dipakai, selesai langsung ditinggal bersih. Self-hosted runner itu seperti ruang meeting kantor sendiri — perlu dirawat sendiri, tapi bisa dikustomisasi bebas.

## 7.3 Fokus Praktik: Ubuntu Runner

```yaml
runs-on: ubuntu-latest
```

Untuk kompetisi LKS, `ubuntu-latest` adalah pilihan paling umum karena ringan, cepat provisioning-nya, dan tooling DevOps mayoritas berbasis Linux.

## 7.4 Mengenal Lingkungan Runner

```yaml
name: Explore Runner

on: workflow_dispatch

jobs:
  explore:
    runs-on: ubuntu-latest
    steps:
      - name: OS detail
        run: uname -a

      - name: Current user
        run: whoami

      - name: Current directory
        run: pwd

      - name: List files
        run: ls -la

      - name: Disk usage
        run: df -h

      - name: Memory usage
        run: free -h
```

**Kenapa penting dipraktikkan?** Saat troubleshooting workflow yang gagal (misal "no space left on device" atau "command not found"), peserta LKS yang terbiasa mengecek `df -h`, `free -h`, dan `uname -a` akan jauh lebih cepat menemukan akar masalah dibanding yang langsung panik re-run tanpa membaca log.

---

# BAB 8 — VARIABLES DAN SECRETS

## 8.1 Perbedaan Variable dan Secret

| | Variables | Secrets |
|---|---|---|
| Tujuan | Nilai konfigurasi biasa, tidak sensitif (mis. nama environment) | Nilai rahasia/sensitif (password, token, access key) |
| Terlihat di log? | Boleh terlihat | **Otomatis disamarkan** (`***`) oleh GitHub jika tidak sengaja di-print |
| Lokasi setting | Settings → Secrets and variables → Actions | sama |

## 8.2 Kenapa Secrets Penting?

**JANGAN PERNAH** menulis langsung di kode/YAML:

```text
❌ Password
❌ API Token
❌ AWS Access Key
❌ AWS Secret Key
```

Jika ter-commit ke repo publik, credential tersebut **bocor permanen** — bahkan setelah dihapus dari commit terbaru, riwayat Git tetap menyimpannya kecuali di-rewrite history secara khusus. Di dunia nyata, banyak insiden kebocoran cloud berawal dari access key yang ter-push ke GitHub publik, lalu dalam hitungan menit di-scan dan disalahgunakan bot.

## 8.3 Level Secrets

```text
Repository Secret   → berlaku untuk satu repo saja
Organization Secret → berlaku untuk semua repo dalam organisasi
Environment Secret  → hanya tersedia saat workflow menargetkan environment tertentu (mis. "production")
```

## 8.4 Cara Membuat Secret

```text
GitHub Repo
   ↓
Settings
   ↓
Secrets and variables
   ↓
Actions
   ↓
New repository secret
```

## 8.5 Menggunakan Secret di Workflow

```yaml
jobs:
  demo:
    runs-on: ubuntu-latest
    steps:
      - name: Use secret safely
        env:
          API_KEY: ${{ secrets.MY_SECRET }}
        run: echo "Secret sudah tersedia sebagai environment variable"
```

**Perhatikan:** kita TIDAK menampilkan `$API_KEY` dengan `echo $API_KEY` — itu praktik buruk karena akan tercetak di log. Cukup buktikan variabel-nya ada tanpa mem-print isinya, misalnya:

```yaml
      - name: Verify secret exists (aman)
        env:
          API_KEY: ${{ secrets.MY_SECRET }}
        run: |
          if [ -n "$API_KEY" ]; then
            echo "Secret berhasil terbaca (nilai disembunyikan)"
          else
            echo "Secret TIDAK ditemukan"
            exit 1
          fi
```

## LAB — SECRET

**Tugas:**
1. Buat repository secret bernama `MY_SECRET` dengan nilai bebas (misal `lks-devops-2025`).
2. Buat workflow yang membaca secret tersebut lewat `env`.
3. Workflow harus **membuktikan** secret terbaca (misalnya panjang string / ada-tidaknya), **tanpa mencetak nilai aslinya** ke log.

**Expected Result:** Log menunjukkan "Secret berhasil terbaca" — tidak ada nilai asli secret yang tampil di mana pun.

---

# BAB 9 — ARTIFACT

## 9.1 Apa itu Artifact?

Artifact adalah **file hasil proses workflow** (misalnya hasil build, laporan test, file log) yang disimpan sementara oleh GitHub Actions agar bisa didownload atau dipakai oleh job lain.

**Perbedaan Artifact vs Source Code:**

| | Source Code | Artifact |
|---|---|---|
| Asal | Sudah ada sejak awal, tersimpan di Git | **Dihasilkan** saat workflow berjalan |
| Contoh | file `.js`, `.py`, `.yaml` | file `.zip` hasil build, laporan `.html`, binary hasil compile |
| Disimpan di | Repository (permanen, versioned) | GitHub Actions storage (sementara, ada masa retensi, default 90 hari) |

## 9.2 Kapan Artifact Digunakan?

* Menyimpan hasil build (misal folder `dist/`) untuk didownload manual.
* Mengoper file dari satu job ke job lain (misal job `build` upload artifact, job `deploy` download artifact yang sama).
* Menyimpan laporan test/coverage untuk diperiksa nanti.

## 9.3 Alur Kerja

```text
Build
 ↓
Generate file (misal: dist/app.js)
 ↓
Upload Artifact
 ↓
(job lain / manusia) Download Artifact
```

## 9.4 Contoh Praktik

```yaml
name: Artifact Demo

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Generate file
        run: |
          mkdir -p output
          echo "Build result $(date)" > output/report.txt

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: build-report          # nama artifact
          path: output/report.txt     # file/folder yang diupload

  download-demo:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: build-report

      - name: Show content
        run: cat report.txt
```

**Penjelasan:** Job `build` menghasilkan file lalu meng-upload-nya sebagai artifact bernama `build-report`. Job `download-demo` (yang butuh `needs: build` supaya berurutan) kemudian mendownload artifact dengan nama yang sama — membuktikan file bisa "berpindah" antar job meski dijalankan di runner yang berbeda dan terpisah.

## LAB — ARTIFACT SEDERHANA

**Tugas:** Buat workflow yang menghasilkan file `hasil.txt` berisi nama peserta dan timestamp, lalu upload sebagai artifact bernama `hasil-lab`.

**Expected Result:** Setelah workflow selesai, di halaman run terdapat bagian **Artifacts** dengan file `hasil-lab` yang bisa didownload dan isinya sesuai.

---

# BAB 10 — MEMBUAT CI PIPELINE LENGKAP

## 10.1 Target Pipeline

```text
Push
 ↓
Checkout
 ↓
Install
 ↓
Build
 ↓
Test
 ↓
Artifact
```

## 10.2 Struktur Repository Contoh

```text
lks-ci-demo/
├── .github/
│   └── workflows/
│       └── ci.yml
├── src/
│   └── index.js
├── tests/
│   └── index.test.js
├── package.json
└── README.md
```

## 10.3 Contoh Source Code Sederhana (Node.js)

`src/index.js`
```javascript
function sum(a, b) {
  return a + b;
}
module.exports = { sum };
```

`tests/index.test.js`
```javascript
const { sum } = require('../src/index');

test('menjumlahkan 2 angka', () => {
  expect(sum(2, 3)).toBe(5);
});
```

`package.json` (bagian relevan)
```json
{
  "name": "lks-ci-demo",
  "scripts": {
    "build": "echo 'Build selesai'",
    "test": "jest"
  },
  "devDependencies": {
    "jest": "^29.0.0"
  }
}
```

## 10.4 Workflow Lengkap

```yaml
name: CI Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  ci:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependency
        run: npm install

      - name: Build
        run: npm run build

      - name: Test
        run: npm test

      - name: Prepare artifact
        run: |
          mkdir -p dist
          cp package.json dist/

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/
```

## 10.5 Expected Result

```text
✓ Checkout
✓ Setup Node.js
✓ Install dependency
✓ Build
✓ Test
✓ Upload artifact
```

Semua step berstatus **Success** (centang hijau), dan artifact `build-output` tersedia untuk didownload.

## 10.6 Troubleshooting Bab Ini

| Masalah | Kemungkinan Penyebab | Solusi |
|---|---|---|
| `npm: command not found` | Lupa step `setup-node` | Tambahkan `actions/setup-node@v4` sebelum `npm install` |
| `Cannot find module '../src/index'` | Struktur folder tidak sesuai / lupa checkout | Pastikan `actions/checkout@v4` ada di step pertama |
| Test gagal padahal lokal sukses | Versi Node berbeda / dependency belum lengkap | Samakan `node-version` dengan versi di lokal, cek `package-lock.json` ikut ter-commit |

---

# BAGIAN B — AWS CLOUDFORMATION (INFRASTRUCTURE AS CODE)

# BAB 11 — KONSEP INFRASTRUCTURE AS CODE (IaC)

## 11.1 Apa itu IaC?

Infrastructure as Code adalah pendekatan mendefinisikan infrastruktur (server, network, storage) menggunakan **file konfigurasi/kode**, bukan klik manual di dashboard.

## 11.2 Manual vs IaC

**Cara Manual (AWS Console):**
```text
User → Login AWS Console → Klik "Create VPC" → Klik "Create Subnet"
     → Klik "Create Security Group" → Klik "Launch EC2" → ...
```
Setiap langkah dilakukan tangan manusia, sulit direplikasi persis sama, rawan lupa satu langkah, dan tidak ada "riwayat perubahan" yang jelas.

**Cara IaC:**
```text
Template (kode)  →  CloudFormation (mesin eksekusi)  →  AWS Resources
```
Infrastruktur "ditulis" sekali sebagai kode, lalu dieksekusi otomatis oleh CloudFormation.

## 11.3 Kenapa IaC Diperlukan?

| Manfaat | Penjelasan |
|---|---|
| **Reproducibility** | Infrastruktur yang sama bisa dibuat ulang persis di environment lain (dev, staging, production) hanya dengan menjalankan template yang sama |
| **Version Control** | Template disimpan di Git — bisa dilihat riwayat perubahannya, siapa yang mengubah apa, dan kapan |
| **Automation** | Bisa diintegrasikan ke pipeline CI/CD (inilah nanti Bagian C) |
| **Dokumentasi hidup** | Template itu sendiri sudah menjadi dokumentasi infrastruktur — tidak perlu tulis ulang di dokumen terpisah yang gampang basi |

**Analogi:** Membangun rumah tanpa blueprint (manual) vs membangun rumah dengan blueprint arsitek (IaC). Blueprint bisa dipakai lagi untuk membangun rumah kembar di lokasi lain dengan hasil yang konsisten.

---

# BAB 12 — PENGENALAN AWS CLOUDFORMATION

## 12.1 Istilah Kunci & Analogi

| Istilah | Definisi | Analogi |
|---|---|---|
| **CloudFormation** | Layanan AWS untuk mengelola infrastruktur berbasis template | Kontraktor yang membaca blueprint dan membangun sesuai instruksi |
| **Template** | File YAML/JSON berisi definisi infrastruktur yang diinginkan | **Blueprint** rumah |
| **Stack** | Kumpulan resource AWS yang dibuat & dikelola bersama dari satu template | **Bangunan** hasil dari blueprint tersebut |
| **Resource** | Satu komponen infrastruktur (S3, EC2, VPC, dll) | **Komponen bangunan** (pintu, jendela, atap) |
| **Parameter** | Nilai input yang bisa diberikan saat stack dibuat | **Pilihan custom** saat pesan rumah (warna cat, jumlah kamar) |
| **Output** | Informasi yang dikembalikan setelah stack selesai dibuat | **Kunci rumah & alamat** yang diserahkan setelah bangunan jadi |
| **Change Set** | Preview perubahan sebelum benar-benar diterapkan ke stack yang sudah ada | **Simulasi renovasi** — lihat dulu apa yang akan berubah sebelum tukang mulai bongkar |

## 12.2 Alur Besar

```text
template.yaml
      ↓
CloudFormation
      ↓
CloudFormation Stack
      ↓
AWS Resources (VPC, EC2, S3, dst)
```

**Poin penting:** Stack bukan sekadar "menjalankan sekali lalu selesai" — stack adalah entitas yang **hidup**. Selama stack ada, CloudFormation terus "mengingat" resource apa saja yang menjadi tanggung jawabnya. Kalau kamu update template dan apply ulang, CloudFormation akan menghitung **perbedaannya** dan hanya mengubah yang perlu. Kalau kamu hapus stack, CloudFormation otomatis menghapus semua resource yang dibuatnya (kecuali diberi proteksi khusus).

---

# BAB 13 — STRUKTUR CLOUDFORMATION TEMPLATE

## 13.1 Kerangka Lengkap

```yaml
AWSTemplateFormatVersion: '2010-09-09'    # versi format template (nyaris selalu nilai ini)

Description: >                             # deskripsi singkat template

Parameters:                                # (opsional) input dari user

Mappings:                                  # (opsional) tabel lookup statis

Conditions:                                # (opsional) logika kondisional

Resources:                                 # (WAJIB) resource yang akan dibuat

Outputs:                                   # (opsional) info hasil deployment
```

## 13.2 Wajib vs Opsional

| Bagian | Status | Kegunaan |
|---|---|---|
| `AWSTemplateFormatVersion` | Opsional (tapi lazim disertakan) | Menandai versi skema template |
| `Description` | Opsional | Dokumentasi singkat |
| `Parameters` | Opsional | Membuat template reusable/fleksibel |
| `Mappings` | Opsional | Data lookup tetap, misal mapping region → AMI ID |
| `Conditions` | Opsional | Membuat resource dibuat/tidak berdasarkan kondisi |
| **`Resources`** | **WAJIB** | Tanpa ini, template tidak melakukan apa-apa |
| `Outputs` | Opsional | Menampilkan info penting hasil stack (URL, bucket name, IP, dll) |

Untuk kompetisi LKS, fokus utama ada di tiga bagian: **Parameters, Resources, Outputs** — itulah kombinasi yang paling sering diuji dan paling sering dipakai di dunia kerja nyata.

---

# BAB 14 — RESOURCES

## 14.1 Resource Paling Sederhana: S3 Bucket

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
```

* `MyBucket` → **Logical ID**, nama yang kita pilih sendiri untuk merujuk resource ini di dalam template (bukan nama asli di AWS).
* `Type: AWS::S3::Bucket` → jenis resource AWS yang ingin dibuat, formatnya selalu `AWS::<Service>::<ResourceType>`.

## 14.2 Progresi Bertahap: Dari S3 sampai EC2

Jangan langsung buat template kompleks. Ikuti urutan berikut agar konsepnya melekat:

**1. S3 (paling sederhana, tanpa network)**
```yaml
Resources:
  AppBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: lks-app-bucket-2025   # harus GLOBALLY unique di seluruh AWS
```

**2. Security Group (aturan firewall)**
```yaml
Resources:
  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP and SSH
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0        # ⚠️ demo saja; di dunia nyata batasi IP tertentu
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
```

**3. VPC (jaringan virtual)**
```yaml
Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      Tags:
        - Key: Name
          Value: lks-vpc
```

**4. Subnet (di dalam VPC)**
```yaml
Resources:
  MySubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC              # merujuk resource VPC di atas
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
```

**5. EC2 (instance virtual machine)**
```yaml
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro
      ImageId: ami-0c101f26f147fa7fd   # contoh AMI ID (region-specific, cek AMI terbaru)
      SubnetId: !Ref MySubnet
      SecurityGroupIds:
        - !Ref WebSecurityGroup
      Tags:
        - Key: Name
          Value: lks-web-server
```

**Kenapa harus bertahap seperti ini?** Karena EC2 **bergantung** pada Subnet, Subnet **bergantung** pada VPC, dan koneksinya harus valid ke Security Group. Jika langsung menulis template lengkap tanpa paham urutan dependency ini, saat terjadi error peserta akan bingung harus mulai debug dari mana. Dengan membangun bertahap (S3 dulu → lalu network → lalu compute), peserta membangun mental model dependency resource yang benar.

---

# BAB 15 — PARAMETERS

## 15.1 Definisi

Parameter membuat template **fleksibel** — nilai tertentu tidak "dikeraskan" (hardcode) di dalam template, tapi diberikan saat stack dibuat/diupdate.

```yaml
Parameters:
  EnvironmentName:
    Type: String
    Default: dev
    Description: Nama environment (dev/staging/production)

  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues:
      - t3.micro
      - t3.small
      - t3.medium
    Description: Tipe instance EC2

  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Nama key pair untuk akses SSH ke EC2
```

## 15.2 Penjelasan Atribut

| Atribut | Fungsi |
|---|---|
| `Type` | Tipe data parameter (`String`, `Number`, `AWS::EC2::KeyPair::KeyName`, dll — AWS punya tipe khusus yang otomatis divalidasi) |
| `Default` | Nilai default jika user tidak mengisi |
| `AllowedValues` | Membatasi pilihan hanya ke daftar tertentu — mencegah salah ketik/nilai tidak valid |
| `Description` | Keterangan yang muncul di console saat user mengisi parameter |

## 15.3 Menggunakan Parameter di Resources

```yaml
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType   # ambil nilai dari Parameters
      KeyName: !Ref KeyName
      Tags:
        - Key: Environment
          Value: !Ref EnvironmentName
```

## LAB — SATU TEMPLATE UNTUK 3 ENVIRONMENT

**Tugas:** Buat satu template `ec2.yaml` dengan parameter `EnvironmentName` (`AllowedValues: [dev, staging, production]`). Deploy stack yang sama tiga kali dengan nama stack berbeda:

```bash
aws cloudformation deploy \
  --template-file ec2.yaml \
  --stack-name lks-dev \
  --parameter-overrides EnvironmentName=dev

aws cloudformation deploy \
  --template-file ec2.yaml \
  --stack-name lks-staging \
  --parameter-overrides EnvironmentName=staging
```

**Expected Result:** Dua stack terpisah muncul di CloudFormation console, masing-masing dengan tag `Environment` sesuai parameter yang diberikan — membuktikan **satu template bisa dipakai berulang** untuk konteks berbeda.

> ⚠️ Jangan lupa cleanup kedua stack setelah lab selesai (lihat Bab 18).

---

# BAB 16 — OUTPUTS

## 16.1 Definisi

Output digunakan untuk **mengeluarkan informasi** dari stack setelah deployment selesai — misalnya nama bucket yang dibuat, IP publik EC2, atau URL endpoint.

```yaml
Outputs:
  BucketName:
    Description: Nama S3 Bucket yang dibuat
    Value: !Ref MyBucket

  InstancePublicIP:
    Description: Public IP dari EC2 instance
    Value: !GetAtt MyEC2Instance.PublicIp
```

## 16.2 `Ref` vs `Fn::GetAtt`

| Fungsi | Kegunaan |
|---|---|
| `!Ref` | Mengambil nilai "utama" sebuah resource (biasanya ID atau nama) — misal `!Ref MyBucket` menghasilkan nama bucket |
| `!GetAtt` | Mengambil **atribut spesifik** dari resource, misal `!GetAtt MyEC2Instance.PublicIp` untuk IP publik, `.PrivateIp` untuk IP privat |

**Analogi:** `!Ref` itu seperti menyebut "nama orangnya", sedangkan `!GetAtt` itu seperti bertanya "detail spesifik tentang orang itu" (nomor HP-nya, alamatnya).

## 16.3 Cara Mengambil Output Stack

Lewat AWS CLI:

```bash
aws cloudformation describe-stacks \
  --stack-name lks-s3 \
  --query "Stacks[0].Outputs"
```

Lewat AWS Console: buka stack → tab **Outputs**.

## LAB — OUTPUT DARI RESOURCE YANG DIBUAT

**Tugas:** Pada template S3 di Bab 14, tambahkan section `Outputs` yang menampilkan nama bucket. Deploy, lalu ambil output-nya via CLI dan buktikan nilainya sesuai dengan bucket yang benar-benar terbentuk di console.

---

# BAB 17 — INTRINSIC FUNCTIONS

Intrinsic function adalah "fungsi bawaan" CloudFormation yang dipakai langsung di dalam template untuk logika dinamis.

## 17.1 `Ref`

Mengambil nilai utama dari Parameter atau Resource.

```yaml
InstanceType: !Ref InstanceType
VpcId: !Ref MyVPC
```

## 17.2 `Fn::Sub`

Menggabungkan string dengan variabel di dalamnya (mirip f-string/template string di bahasa pemrograman).

```yaml
BucketName: !Sub "${EnvironmentName}-lks-bucket"
```

Jika `EnvironmentName = dev`, hasilnya: `dev-lks-bucket`.

Bisa juga dengan atribut resource:

```yaml
Outputs:
  WebsiteURL:
    Value: !Sub "http://${MyEC2Instance.PublicIp}"
```

## 17.3 `Fn::GetAtt`

Mengambil atribut spesifik dari sebuah resource (sudah dibahas di Bab 16).

```yaml
!GetAtt MyEC2Instance.PublicIp
```

## 17.4 `Fn::Join`

Menggabungkan list string dengan separator tertentu.

```yaml
Value: !Join
  - "-"
  - - !Ref EnvironmentName
    - "lks"
    - "bucket"
```

Hasilnya sama seperti `Fn::Sub` di atas: `dev-lks-bucket`. Untuk kasus sederhana, `!Sub` biasanya lebih mudah dibaca; `!Join` lebih berguna saat menggabungkan list dinamis (hasil dari fungsi lain).

## 17.5 Contoh Gabungan Nyata

```yaml
Resources:
  AppBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "${EnvironmentName}-${AWS::AccountId}-app-bucket"
      Tags:
        - Key: Environment
          Value: !Ref EnvironmentName

Outputs:
  BucketArn:
    Value: !GetAtt AppBucket.Arn
  BucketURL:
    Value: !Sub "https://${AppBucket}.s3.amazonaws.com"
```

`${AWS::AccountId}` adalah **pseudo parameter** bawaan AWS (tersedia otomatis tanpa perlu dideklarasikan) — berguna untuk memastikan nama bucket unik secara global karena mengandung Account ID.

---

# BAB 18 — CLOUDFORMATION STACK LIFECYCLE

## 18.1 Create Stack

```text
Template
 ↓
aws cloudformation create-stack
 ↓
CREATE_IN_PROGRESS
 ↓
CREATE_COMPLETE   (atau CREATE_FAILED)
```

```bash
aws cloudformation create-stack \
  --stack-name lks-s3 \
  --template-body file://s3.yaml
```

Atau (lebih direkomendasikan karena otomatis handle create/update):

```bash
aws cloudformation deploy \
  --stack-name lks-s3 \
  --template-file s3.yaml
```

## 18.2 Update Stack

```text
Template (sudah diubah)
 ↓
aws cloudformation update-stack / deploy (lagi)
 ↓
UPDATE_IN_PROGRESS
 ↓
UPDATE_COMPLETE   (atau UPDATE_ROLLBACK_COMPLETE jika gagal)
```

**Penting:** Jika update gagal di tengah jalan, CloudFormation otomatis **rollback** ke kondisi sebelumnya — ini fitur keamanan bawaan yang mencegah infrastruktur "setengah jadi".

## 18.3 Delete Stack (Cleanup!)

```text
DELETE
 ↓
DELETE_IN_PROGRESS
 ↓
DELETE_COMPLETE
```

```bash
aws cloudformation delete-stack --stack-name lks-s3
```

> ⚠️ **WAJIB dilakukan setiap selesai lab** — jika stack berisi EC2/NAT Gateway/dsb dibiarkan menyala, biaya akan terus berjalan meski tidak dipakai.

## 18.4 Membaca CloudFormation Events

```text
CloudFormation
 ↓
Pilih Stack
 ↓
Tab "Events"
 ↓
Cari baris dengan status *_FAILED
 ↓
Baca kolom "Status reason" — biasanya berisi pesan error spesifik
```

Lewat CLI:

```bash
aws cloudformation describe-stack-events \
  --stack-name lks-s3 \
  --query "StackEvents[?ResourceStatus=='CREATE_FAILED']"
```

**Kebiasaan wajib untuk LKS:** Saat stack gagal, JANGAN langsung hapus dan buat ulang tanpa membaca Events terlebih dahulu — kamu akan kehilangan informasi penyebab error dan berpotensi mengulang kesalahan yang sama.

---

# BAB 19 — CLOUDFORMATION VALIDATION

## 19.1 Kebiasaan: Validate → Deploy → Verify

```text
Validate  → cek template sebelum dijalankan
   ↓
Deploy    → jalankan template
   ↓
Verify    → cek hasil sesuai ekspektasi
```

Jangan pernah langsung `deploy` tanpa `validate` — banyak kesalahan syntax bisa ditangkap lebih cepat dan lebih murah di tahap validasi.

## 19.2 Perintah Validasi

```bash
aws cloudformation validate-template --template-body file://s3.yaml
```

Perintah ini hanya mengecek **syntax dan struktur**, BUKAN mengecek apakah resource benar-benar bisa dibuat (misal quota habis) — itu baru ketahuan saat deploy.

## 19.3 Jenis Error yang Sering Muncul

| Jenis Error | Contoh | Cara Deteksi |
|---|---|---|
| YAML syntax error | Indentasi salah, kurang tanda `:` | `validate-template` langsung gagal dengan pesan parsing |
| Invalid resource type | Salah tulis `AWS::S3::Buckett` | Error "Unrecognized resource type" |
| Missing required property | Lupa `GroupDescription` di Security Group | Error "Required property missing" |
| Invalid parameter | Memberi `InstanceType=xxx` yang tidak ada di `AllowedValues` | Error saat deploy "Parameter validation failed" |
| Dependency error | EC2 merujuk Subnet yang belum/tidak ada | Error CREATE_FAILED pada resource yang mereferensikan |

---

# BAB 20 — STUDI KASUS CLOUDFORMATION BERTAHAP

> ⚠️ Semua Case di bawah membuat resource AWS nyata. Selalu **cleanup** (`delete-stack`) setelah setiap Case selesai diverifikasi, sebelum lanjut ke Case berikutnya.

## Case 1 — S3 Bucket Sederhana

**Requirement:** Membuat 1 S3 bucket dengan nama unik.

**Template awal:**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Case 1 - Simple S3 Bucket

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
```

**Task:** Deploy dengan nama stack `lks-case1`.

**Expected Result:** Bucket muncul di S3 console dengan nama random (auto-generated karena `BucketName` tidak ditentukan).

**Troubleshooting:** Jika `CREATE_FAILED` dengan pesan "already exists" — kemungkinan kamu sudah menetapkan `BucketName` manual yang bentrok dengan bucket orang lain (nama S3 bucket unik **secara global**, bukan hanya per-akun).

**Challenge:** Tambahkan `BucketName` custom menggunakan `!Sub` yang menyertakan `${AWS::AccountId}` agar pasti unik.

---

## Case 2 — S3 dengan Parameter

**Requirement:** Bucket dengan nama environment sebagai parameter, plus tag.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Case 2 - S3 with Parameter

Parameters:
  EnvironmentName:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, production]

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "${EnvironmentName}-${AWS::AccountId}-lks-bucket"
      Tags:
        - Key: Environment
          Value: !Ref EnvironmentName

Outputs:
  BucketName:
    Value: !Ref MyBucket
```

**Task:** Deploy dengan `EnvironmentName=staging`, verifikasi tag di console sesuai.

**Troubleshooting:** Error "Parameter value staging2 for parameter name EnvironmentName does not exist" → nilai tidak ada di `AllowedValues`, cek kembali ejaan parameter override.

---

## Case 3 — VPC

**Requirement:** VPC dengan CIDR `10.0.0.0/16`.

```yaml
Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: lks-vpc

Outputs:
  VPCId:
    Value: !Ref MyVPC
```

**Task:** Deploy dan verifikasi VPC ID muncul di Outputs, cocokkan dengan VPC console.

---

## Case 4 — VPC + Subnet + Security Group

```yaml
Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16

  MySubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']

  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP and SSH
      VpcId: !Ref MyVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
```

**Task:** Deploy, verifikasi Subnet muncul di dalam VPC yang benar, dan Security Group memiliki 2 rule inbound.

**Troubleshooting:** Error "Subnet's Availability Zone does not belong to region" → biasanya karena AZ hardcode tidak sesuai region yang dipakai; gunakan `!Select [0, !GetAZs '']` agar otomatis menyesuaikan region.

---

## Case 5 — VPC + Subnet + Security Group + EC2 (Full Stack)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Case 5 - Full Web Server Stack

Parameters:
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
  InstanceType:
    Type: String
    Default: t3.micro

Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16

  InternetGateway:
    Type: AWS::EC2::InternetGateway

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref MyVPC
      InternetGatewayId: !Ref InternetGateway

  MySubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true

  RouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref MyVPC

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref RouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  SubnetRouteAssoc:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref MySubnet
      RouteTableId: !Ref RouteTable

  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP and SSH
      VpcId: !Ref MyVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: ami-0c101f26f147fa7fd
      KeyName: !Ref KeyName
      SubnetId: !Ref MySubnet
      SecurityGroupIds:
        - !Ref WebSecurityGroup
      Tags:
        - Key: Name
          Value: lks-web-server

Outputs:
  PublicIP:
    Description: Public IP EC2
    Value: !GetAtt MyEC2Instance.PublicIp
```

**Penjelasan resource dependency (kenapa urutannya begini):**
```text
VPC
 ↓
Internet Gateway ── attach ke VPC
 ↓
Subnet (di dalam VPC)
 ↓
Route Table ── route 0.0.0.0/0 lewat Internet Gateway
 ↓
Associate Route Table ke Subnet
 ↓
Security Group (di dalam VPC)
 ↓
EC2 (pakai Subnet + Security Group)
```

**Troubleshooting:**
- `CREATE_FAILED` pada `PublicRoute` dengan pesan gateway belum attached → pastikan `DependsOn: AttachGateway` ada, karena CloudFormation tidak selalu otomatis tahu urutan dependency implisit ini.
- EC2 ter-create tapi tidak dapat Public IP → cek `MapPublicIpOnLaunch: true` di Subnet.
- Tidak bisa SSH → cek Security Group inbound port 22, dan pastikan `KeyName` yang dipakai memang ada di region tersebut.

**Challenge:** Tambahkan Output `SSHCommand` menggunakan `!Sub` yang menghasilkan string siap pakai, misalnya `ssh -i key.pem ec2-user@<IP>`.

**Cleanup:**
```bash
aws cloudformation delete-stack --stack-name lks-case5
```

---

# BAGIAN C — INTEGRASI GITHUB ACTIONS + CLOUDFORMATION

# BAB 21 — CI/CD + INFRASTRUCTURE AS CODE

## 21.1 Kenapa Digabung?

Sejauh ini GitHub Actions dipakai untuk kode aplikasi, dan CloudFormation dijalankan manual lewat CLI. Di dunia nyata, keduanya digabung: **perubahan infrastruktur juga melewati proses CI/CD**, sama seperti perubahan kode aplikasi — supaya konsisten, aman, dan tercatat.

## 21.2 Arsitektur Gabungan

```text
Developer
    ↓
GitHub
    ↓
GitHub Actions
    ↓
Validate  (validate-template)
    ↓
Test      (test aplikasi seperti biasa)
    ↓
CloudFormation  (deploy template)
    ↓
AWS
    ↓
Infrastructure (VPC, EC2, S3, dst)
```

**Analogi:** Sebelumnya kita membangun rumah (infrastruktur) dengan memanggil tukang secara manual setiap kali ada revisi blueprint. Sekarang, setiap revisi blueprint yang di-push ke GitHub otomatis dicek dulu (validate), lalu tukang robot (GitHub Actions) langsung mengeksekusinya ke lokasi pembangunan (AWS) — tanpa kita perlu telepon manual setiap kali.

---

# BAB 22 — GITHUB ACTIONS MENJALANKAN CLOUDFORMATION

## 22.1 Alur Bertahap

```text
Git Push
 ↓
GitHub Actions terpicu
 ↓
Validate CloudFormation
 ↓
Deploy CloudFormation
 ↓
AWS Stack terbentuk/terupdate
```

## 22.2 Tahap 1 — Hanya Validasi (Paling Aman untuk Belajar)

```yaml
name: Validate CloudFormation

on:
  push:
    branches: [main]
    paths:
      - 'infra/**'          # hanya trigger jika ada perubahan di folder infra/

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-southeast-1

      - name: Validate template
        run: |
          aws cloudformation validate-template \
            --template-body file://infra/s3.yaml
```

**Penjelasan:** `paths: ['infra/**']` membuat workflow ini hanya berjalan jika ada perubahan file di folder `infra/` — pipeline aplikasi (kode) dan pipeline infrastruktur bisa dipisah agar lebih efisien dan jelas tanggung jawabnya.

## 22.3 Tahap 2 — Validasi + Deploy

```yaml
name: Deploy CloudFormation

on:
  push:
    branches: [main]
    paths:
      - 'infra/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-southeast-1

      - name: Validate template
        run: |
          aws cloudformation validate-template \
            --template-body file://infra/s3.yaml

      - name: Deploy stack
        run: |
          aws cloudformation deploy \
            --stack-name lks-s3-ci \
            --template-file infra/s3.yaml \
            --parameter-overrides EnvironmentName=staging \
            --no-fail-on-empty-changeset

      - name: Show stack outputs
        run: |
          aws cloudformation describe-stacks \
            --stack-name lks-s3-ci \
            --query "Stacks[0].Outputs"
```

**Penjelasan `--no-fail-on-empty-changeset`:** jika tidak ada perubahan sama sekali dari deployment sebelumnya (misal template persis sama), tanpa flag ini command akan dianggap "gagal" walau sebenarnya wajar. Flag ini membuat pipeline tidak berhenti hanya karena "tidak ada yang perlu diubah".

**Kesalahan umum:** Peserta lupa `aws-region`, sehingga deploy berhasil tapi mengarah ke region yang salah/berbeda dari yang diharapkan, dan bingung kenapa resource "tidak ada" saat dicek manual di console (padahal cuma beda region).

---

# BAB 23 — AWS CREDENTIAL DAN SECURITY

## 23.1 Kenapa Credential Tidak Boleh Ditulis di YAML?

```yaml
# ❌ JANGAN PERNAH LAKUKAN INI
- run: aws configure set aws_access_key_id AKIAxxxxxxxxxxx
```

Jika ini ter-push ke GitHub — bahkan di repo privat sekalipun — credential tersebut tersimpan permanen di riwayat Git dan berisiko bocor (misal repo tiba-tiba jadi publik, atau ada kolaborator nakal). Bot pemindai kredensial di internet bisa menemukan access key yang bocor di repo publik **dalam hitungan menit** dan langsung menyalahgunakannya (contoh nyata: banyak kasus tagihan AWS membengkak ratusan juta rupiah akibat access key bocor dan dipakai untuk mining crypto oleh pihak tidak bertanggung jawab).

## 23.2 GitHub Secrets (Cara Aman #1)

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ap-southeast-1
```

Nilai `secrets.AWS_ACCESS_KEY_ID` **tidak pernah tampil** di kode maupun log — hanya "disuntikkan" saat runtime.

## 23.3 IAM & Least Privilege

**IAM (Identity and Access Management)** adalah sistem AWS untuk mengatur **siapa boleh melakukan apa**.

**Prinsip Least Privilege:** berikan **hanya izin yang benar-benar dibutuhkan**, tidak lebih.

```text
❌ AdministratorAccess (semua izin, semua resource — sangat berbahaya jika bocor)
✓  Custom Policy (hanya izin CloudFormation + EC2 + S3 + IAM PassRole yang diperlukan)
```

**Analogi:** Memberi kunci gudang penuh ke office boy yang tugasnya cuma buang sampah, dibanding memberi kunci khusus tempat sampah saja. Kalau kunci itu hilang/dicuri, kerugiannya jauh lebih kecil pada kasus kedua.

## 23.4 Role-based Authentication

Alih-alih membagikan access key/secret key statis (yang bisa bocor & tidak pernah "expired" otomatis), pendekatan lebih aman adalah menggunakan **Role** — sebuah identitas sementara dengan izin terbatas yang bisa "dipinjam" (assume) oleh entitas tertentu, dan otomatis punya kredensial yang berumur pendek (temporary credentials).

## 23.5 OIDC (Konsep)

**OpenID Connect (OIDC)** adalah mekanisme yang memungkinkan GitHub Actions **login ke AWS tanpa perlu menyimpan Access Key/Secret Key sama sekali**.

```text
Cara Lama (Static Key):
GitHub Secrets (AWS_ACCESS_KEY_ID + SECRET) ── disimpan permanen ──► AWS

Cara Modern (OIDC):
GitHub Actions ── "token identitas sementara" ──► AWS IAM Role ──► Akses sementara (auto expired)
```

**Kelebihan OIDC:**
- Tidak ada credential statis yang bisa bocor.
- Akses otomatis kedaluwarsa setelah job selesai.
- Bisa dibatasi hanya untuk repository/branch tertentu (mis. hanya branch `main` di repo tertentu yang boleh assume role).

Untuk kompetisi LKS tingkat pemula-menengah, memahami **konsepnya** sudah cukup; implementasi detail OIDC biasanya menjadi materi lanjutan.

## 23.6 Ringkasan Checklist Keamanan

```text
✓ Simpan credential HANYA di GitHub Secrets (atau lebih baik lagi, pakai OIDC)
✓ Gunakan IAM policy sesempit mungkin (least privilege)
✓ Jangan pernah echo/print nilai secret ke log
✓ Batasi Security Group hanya ke IP/port yang benar-benar perlu
✓ Selalu cleanup resource setelah selesai lab (hindari biaya nyasar)
```

---

# BAB 24 — FINAL PROJECT

## Studi Kasus

> "Anda bekerja sebagai **Junior DevOps Engineer**. Sebuah perusahaan memiliki repository aplikasi web sederhana di GitHub. Perusahaan meminta Anda membuat pipeline CI/CD sederhana menggunakan GitHub Actions serta menyediakan infrastructure AWS menggunakan CloudFormation."

## Daftar Tugas

```text
1.  Membuat repository GitHub
2.  Membuat branch feature
3.  Membuat workflow GitHub Actions
4.  Melakukan checkout
5.  Melakukan build
6.  Melakukan testing
7.  Melakukan artifact upload
8.  Membuat CloudFormation template
9.  Melakukan validation
10. Membuat CloudFormation stack
11. Melakukan verification
12. Melakukan documentation
```

## Final Flow

```text
GitHub
   ↓
GitHub Actions
   ↓
Build
   ↓
Test
   ↓
Validate CloudFormation
   ↓
Deploy CloudFormation
   ↓
AWS Infrastructure
   ↓
Verification
```

## Struktur Repository yang Diharapkan

```text
final-project/
├── .github/
│   └── workflows/
│       ├── ci.yml          # build + test + artifact aplikasi
│       └── infra.yml       # validate + deploy CloudFormation
├── src/
├── tests/
├── infra/
│   └── template.yaml
├── docs/
│   └── README.md           # dokumentasi arsitektur & cara deploy
└── package.json
```

## Kriteria Kelulusan Final Project

```text
✓ Kedua workflow (ci.yml dan infra.yml) berjalan sukses tanpa error
✓ CloudFormation stack ter-deploy dan resource sesuai requirement
✓ Output stack menampilkan informasi yang relevan (misal endpoint/IP)
✓ Secrets AWS tidak pernah bocor ke log
✓ Dokumentasi (README) menjelaskan arsitektur dan cara menjalankan ulang
✓ Cleanup dilakukan di akhir (stack dihapus setelah demo selesai dinilai)
```

---

# BAGIAN D — SIMULASI LKS

## SIMULASI 1 — BASIC (Durasi: 60 menit)

**Materi diuji:** GitHub, Workflow, Job, Step, Runner

**Soal Praktik:**
1. Buat repository baru.
2. Buat workflow yang trigger saat push ke `main`.
3. Buat 2 job (`build` dan `test`) yang berjalan **paralel**.
4. Setiap job menampilkan info runner (`uname -a`, `whoami`, `date`).
5. Tambahkan 1 job `report` yang **bergantung** (`needs`) pada kedua job di atas.

**Kriteria Selesai:** Ketiga job berstatus sukses, dapat dibuktikan urutan eksekusi `report` selalu setelah `build` dan `test` selesai.

---

## SIMULASI 2 — INTERMEDIATE (Durasi: 120 menit)

**Materi diuji:** CI Pipeline, Secrets, Artifact, CloudFormation, Parameters, Outputs

**Soal Praktik:**
1. Buat CI pipeline lengkap: checkout → install → build → test → upload artifact.
2. Buat 1 repository secret dan gunakan secara aman di workflow (tanpa tercetak di log).
3. Buat CloudFormation template S3 bucket dengan parameter `EnvironmentName`.
4. Deploy template tersebut untuk 2 environment berbeda (`dev` dan `staging`).
5. Tampilkan Output `BucketName` dari masing-masing stack.
6. Lakukan cleanup kedua stack di akhir sesi.

**Kriteria Selesai:** Pipeline CI sukses dengan artifact terupload, dua stack CloudFormation berhasil dibuat dengan nama bucket berbeda sesuai parameter, dan kedua stack sudah dihapus sebelum sesi berakhir.

---

## SIMULASI 3 — COMPETITION (Durasi: 180–240 menit)

**Materi diuji:** GitHub Actions, CI, Secrets, Artifact, CloudFormation, VPC, Subnet, Security Group, EC2, Deployment, Troubleshooting

**Skenario:**
> Perusahaan simulasi "PT LKS Cloud Nusantara" membutuhkan pipeline CI/CD lengkap untuk aplikasi web internal beserta infrastruktur pendukungnya di AWS.

**Soal Praktik:**
1. Setup repository dengan branch strategy (`main` + `feature/*`).
2. Buat CI pipeline: build, test, artifact.
3. Buat CloudFormation template full-stack: VPC + Subnet + Internet Gateway + Route Table + Security Group + EC2.
4. Buat parameter untuk `EnvironmentName`, `InstanceType`, `KeyName`.
5. Integrasikan GitHub Actions untuk validate + deploy CloudFormation menggunakan AWS credentials dari Secrets.
6. Tampilkan Output `PublicIP` EC2 setelah deploy.
7. **Troubleshooting terinjeksi:** Panitia akan sengaja merusak salah satu bagian (misal Security Group salah port, atau parameter salah tipe) — peserta harus menemukan dan memperbaiki dalam waktu terbatas.
8. Dokumentasikan seluruh arsitektur dan langkah reproduksi di README.
9. Cleanup seluruh stack di akhir kompetisi.

**Kriteria Penilaian:** Lihat Bagian E — Rubrik Penilaian.

---

# BAGIAN E — RUBRIK PENILAIAN

**Total: 100 poin**

| Komponen | Bobot |
|---|---:|
| Git & GitHub | 10 |
| GitHub Actions | 25 |
| CI Pipeline | 15 |
| Secrets & Security | 10 |
| CloudFormation | 25 |
| AWS Infrastructure | 10 |
| Documentation | 5 |
| **TOTAL** | **100** |

## Indikator Detail per Komponen

### Git & GitHub — 10 poin
- **9–10** = branching rapi, commit message jelas, PR & merge digunakan dengan benar.
- **6–8** = repo & branch ada, tapi commit message kurang deskriptif atau PR tidak digunakan.
- **3–5** = hanya push langsung ke main tanpa branch/PR sama sekali.
- **1–2** = repository dibuat tapi tidak ada aktivitas commit yang berarti.
- **0** = tidak mengerjakan.

### GitHub Actions — 25 poin
- **20–25** = workflow berjalan sempurna dan peserta memahami struktur YAML (event, job, step, action, runner) secara utuh.
- **15–19** = workflow berjalan tetapi terdapat kesalahan minor (misal urutan step kurang optimal, penamaan kurang jelas).
- **10–14** = workflow sebagian berjalan (ada job/step yang gagal atau ter-skip karena kesalahan konfigurasi).
- **1–9** = workflow tidak berjalan sama sekali, tetapi peserta memahami sebagian konsep saat wawancara/penjelasan lisan.
- **0** = tidak mengerjakan.

### CI Pipeline — 15 poin
- **12–15** = pipeline lengkap (checkout, install, build, test, artifact) berjalan sukses end-to-end.
- **8–11** = pipeline berjalan tapi ada tahap yang hilang (misal tidak ada artifact).
- **4–7** = hanya build/test dasar tanpa struktur pipeline yang jelas.
- **1–3** = pipeline gagal total tapi ada usaha konfigurasi.
- **0** = tidak mengerjakan.

### Secrets & Security — 10 poin
- **8–10** = credential dikelola sepenuhnya lewat Secrets, tidak ada kebocoran nilai di log, least privilege diterapkan.
- **5–7** = Secrets digunakan tapi ada praktik kurang aman (misal `echo` nilai secret walau tidak sengaja).
- **2–4** = credential sempat ditulis manual di YAML tapi disadari & diperbaiki.
- **1** = credential ditulis manual dan tidak diperbaiki.
- **0** = tidak mengerjakan / kredensial bocor fatal ke publik.

### CloudFormation — 25 poin
- **20–25** = template valid, stack berhasil dibuat, resource sesuai requirement (VPC/Subnet/SG/EC2 saling terhubung benar), memakai Parameters & Outputs dengan baik.
- **15–19** = stack berhasil dibuat tapi ada resource yang kurang optimal/kurang lengkap.
- **10–14** = template ada tapi deployment gagal sebagian (`CREATE_FAILED` pada beberapa resource).
- **1–9** = template tidak valid / deployment gagal total, tapi peserta memahami struktur dasar.
- **0** = tidak mengerjakan.

### AWS Infrastructure — 10 poin
- **8–10** = seluruh resource berfungsi sesuai tujuan (misal EC2 bisa diakses, Security Group tepat).
- **5–7** = resource terbentuk tapi ada konfigurasi yang kurang tepat (misal SG terlalu terbuka tanpa alasan).
- **2–4** = sebagian resource terbentuk, sebagian gagal.
- **1** = resource nyaris semua gagal terbentuk.
- **0** = tidak mengerjakan.

### Documentation — 5 poin
- **4–5** = README jelas, menjelaskan arsitektur, cara deploy, dan cleanup.
- **2–3** = README ada tapi minim detail.
- **1** = hanya judul/placeholder.
- **0** = tidak ada dokumentasi.

---

# BAGIAN F — TROUBLESHOOTING GUIDE

Format: **Problem → Possible Cause → How to Check → Solution → Prevention**

## F.1 — Troubleshooting GitHub Actions

### 1. Workflow tidak muncul di tab Actions
- **Possible Cause:** File YAML tidak berada di `.github/workflows/`, atau ekstensi file salah (bukan `.yml`/`.yaml`).
- **How to Check:** Buka repo → pastikan path persis `.github/workflows/nama.yml`.
- **Solution:** Pindahkan file ke lokasi yang benar, commit & push ulang.
- **Prevention:** Selalu buat folder lewat CLI (`mkdir -p .github/workflows`) untuk menghindari typo folder.

### 2. Workflow tidak ter-trigger
- **Possible Cause:** Event/branch filter di `on:` tidak cocok dengan aksi yang dilakukan (misal push ke branch selain yang didaftarkan).
- **How to Check:** Bandingkan branch tempat push dengan `on.push.branches`.
- **Solution:** Sesuaikan filter branch, atau push ke branch yang benar.
- **Prevention:** Saat development awal, gunakan `workflow_dispatch` supaya bisa ditest manual tanpa bergantung branch/event.

### 3. YAML syntax error
- **Possible Cause:** Indentasi tidak konsisten (YAML sangat sensitif terhadap spasi), tanda `:` hilang, atau tab tercampur spasi.
- **How to Check:** Lihat pesan error di tab Actions (biasanya sudah menunjuk baris bermasalah), atau gunakan validator YAML online/editor dengan linter.
- **Solution:** Perbaiki indentasi (gunakan 2 spasi konsisten, jangan tab).
- **Prevention:** Gunakan editor (VS Code) dengan ekstensi YAML yang menampilkan error real-time.

### 4. Workflow jalan di branch yang salah
- **Possible Cause:** Filter `branches` terlalu longgar atau memakai wildcard yang tidak sesuai maksud.
- **How to Check:** Cek riwayat run di tab Actions, lihat kolom branch.
- **Solution:** Perjelas filter, misal `branches: [main]` bukan `branches: ['*']`.
- **Prevention:** Selalu eksplisit menyebutkan nama branch, hindari wildcard kecuali benar-benar perlu.

### 5. Job failed
- **Possible Cause:** Salah satu step dalam job mengembalikan exit code bukan 0.
- **How to Check:** Buka run → klik job yang gagal → baca log step per step, cari tanda ✗ merah.
- **Solution:** Perbaiki perintah/kode pada step yang gagal berdasarkan pesan error di log.
- **Prevention:** Test perintah secara lokal terlebih dahulu sebelum dimasukkan ke workflow.

### 6. Step failed karena command not found
- **Possible Cause:** Tool yang dipakai belum di-install/di-setup di runner (misal `npm` dipakai tanpa `setup-node`).
- **How to Check:** Baca log, cari pesan `command not found`.
- **Solution:** Tambahkan step setup yang sesuai (`actions/setup-node`, `actions/setup-python`, dll) sebelum step yang membutuhkannya.
- **Prevention:** Selalu ingat urutan: checkout → setup tools → install dependency → jalankan perintah.

### 7. Permission denied
- **Possible Cause:** Token GitHub Actions default tidak punya izin yang dibutuhkan (misal push balik ke repo, akses ke resource tertentu).
- **How to Check:** Cek pesan error, biasanya menyebut `403` atau `permission denied`.
- **Solution:** Sesuaikan `permissions:` di workflow, atau gunakan token/secret dengan scope yang tepat.
- **Prevention:** Terapkan least privilege — beri permission sesuai kebutuhan minimum saja.

### 8. Secret tidak terbaca (kosong)
- **Possible Cause:** Nama secret di workflow tidak persis sama dengan nama yang didaftarkan di Settings, atau secret didaftarkan di level environment yang berbeda dari yang dipakai job.
- **How to Check:** Cocokkan penulisan `secrets.NAMA_SECRET` dengan nama persis di Settings → Secrets and variables.
- **Solution:** Perbaiki penamaan, atau daftarkan ulang secret di level yang benar (repository/environment).
- **Prevention:** Gunakan penamaan konsisten (huruf besar + underscore) dan dokumentasikan daftar secret yang dipakai di README.

### 9. Artifact gagal diupload/didownload
- **Possible Cause:** Path file di `path:` tidak sesuai lokasi sebenarnya, atau nama artifact antara upload dan download tidak sama persis.
- **How to Check:** Bandingkan `path` di step generate file dengan `path` di `upload-artifact`.
- **Solution:** Samakan path dan nama artifact, gunakan `ls -la` sebelum upload untuk memastikan file benar ada.
- **Prevention:** Selalu tambahkan step verifikasi (`ls`) sebelum upload artifact.

### 10. Runner error / job stuck
- **Possible Cause:** Runner kehabisan resource (disk penuh) atau ada proses yang menunggu input interaktif (hang).
- **How to Check:** `df -h` di awal job untuk cek disk, cek apakah ada perintah yang butuh konfirmasi manual (`-y` flag terlupa).
- **Solution:** Tambahkan flag non-interaktif (`apt-get install -y`), bersihkan file besar yang tidak perlu.
- **Prevention:** Selalu gunakan flag non-interaktif di setiap perintah instalasi dalam CI.

## F.2 — Troubleshooting CloudFormation

### 1. Template validation failed
- **Possible Cause:** Kesalahan syntax YAML, atau struktur field tidak sesuai skema CloudFormation.
- **How to Check:** `aws cloudformation validate-template --template-body file://template.yaml`, baca pesan error.
- **Solution:** Perbaiki sesuai pesan error (biasanya menunjuk baris/field bermasalah).
- **Prevention:** Validasi template setiap kali ada perubahan sebelum deploy.

### 2. CREATE_FAILED
- **Possible Cause:** Resource gagal dibuat karena konfigurasi salah (misal nama bucket sudah dipakai, quota limit tercapai, dependency belum siap).
- **How to Check:** Buka tab **Events** di stack, cari baris dengan status `CREATE_FAILED` dan baca kolom "Status reason".
- **Solution:** Perbaiki penyebab spesifik sesuai pesan (misal ganti nama bucket, minta quota naik, perbaiki `DependsOn`).
- **Prevention:** Gunakan nama resource dinamis (`!Sub` dengan AccountId) untuk menghindari bentrok nama.

### 3. UPDATE_FAILED
- **Possible Cause:** Perubahan yang diminta tidak valid untuk resource yang sudah ada (beberapa properti tidak bisa diubah tanpa replace resource).
- **How to Check:** Cek Events, cari pesan seperti "cannot be updated".
- **Solution:** Untuk properti yang immutable, kadang perlu hapus & buat ulang resource (`Replacement: True` pada Change Set).
- **Prevention:** Selalu cek Change Set sebelum apply update ke stack production.

### 4. DELETE_FAILED
- **Possible Cause:** Resource masih memiliki dependency aktif (misal S3 bucket masih berisi objek, tidak bisa dihapus jika tidak kosong).
- **How to Check:** Cek Events, biasanya pesan spesifik menyebut resource yang menolak dihapus.
- **Solution:** Kosongkan/lepas dependency manual (misal kosongkan bucket dulu), lalu retry delete-stack.
- **Prevention:** Untuk S3 bucket yang dipakai lab, aktifkan `DeletionPolicy` sesuai kebutuhan dan pastikan bucket kosong sebelum cleanup.

### 5. IAM AccessDenied
- **Possible Cause:** User/Role yang menjalankan CloudFormation tidak punya izin cukup untuk membuat resource tertentu.
- **How to Check:** Baca pesan error, biasanya eksplisit menyebut aksi IAM yang ditolak (misal `ec2:RunInstances`).
- **Solution:** Tambahkan policy yang sesuai ke IAM user/role (tetap dengan prinsip least privilege, jangan langsung `AdministratorAccess`).
- **Prevention:** Siapkan IAM policy khusus untuk kebutuhan CloudFormation sejak awal, uji coba di environment non-produksi dulu.

### 6. Resource tidak ditemukan (does not exist)
- **Possible Cause:** Merujuk resource dengan `!Ref`/`!GetAtt` yang salah Logical ID, atau merujuk resource di region/account yang berbeda.
- **How to Check:** Cocokkan Logical ID di `Resources` dengan yang direferensikan.
- **Solution:** Perbaiki penulisan Logical ID, pastikan region yang dipakai konsisten.
- **Prevention:** Konsisten menamai Logical ID dengan pola jelas (misal `WebSecurityGroup`, bukan `SG1`, `sg-web`, dll campur-campur).

### 7. Parameter error
- **Possible Cause:** Nilai yang diberikan tidak sesuai `Type` atau tidak ada di `AllowedValues`.
- **How to Check:** Baca pesan error saat deploy, biasanya jelas menyebut parameter mana yang bermasalah.
- **Solution:** Sesuaikan nilai parameter dengan `Type`/`AllowedValues` yang didefinisikan.
- **Prevention:** Selalu isi `AllowedValues` dan `Description` untuk parameter penting agar user lain tidak salah input.

### 8. Security Group error
- **Possible Cause:** Rule tidak valid (misal `FromPort` lebih besar dari `ToPort`), atau referensi `VpcId` tidak ada.
- **How to Check:** Baca pesan error spesifik di Events.
- **Solution:** Perbaiki range port, pastikan `VpcId: !Ref MyVPC` merujuk VPC yang benar-benar ada di template yang sama.
- **Prevention:** Selalu buat Security Group setelah VPC didefinisikan, uji dengan Case sederhana dulu (Bab 20).

### 9. VPC dependency error
- **Possible Cause:** Resource seperti Route mereferensikan Internet Gateway yang belum ter-attach ke VPC saat resource itu dibuat.
- **How to Check:** Baca Events, cari pesan terkait gateway/route.
- **Solution:** Tambahkan `DependsOn` eksplisit ke resource yang menjadi prasyarat (lihat Bab 20 Case 5).
- **Prevention:** Selalu urutkan mental model: VPC → Gateway → Attach → Subnet → Route Table → Associate → Security Group → EC2.

### 10. EC2 gagal dibuat
- **Possible Cause:** `ImageId` (AMI) tidak valid untuk region yang dipakai, `KeyName` tidak ada di region tersebut, atau `InstanceType` tidak tersedia di Availability Zone terpilih.
- **How to Check:** Baca pesan error di Events, biasanya sangat spesifik (misal "InvalidAMIID.NotFound").
- **Solution:** Cek AMI ID terbaru yang valid untuk region yang dipakai lewat AWS Console/CLI, pastikan KeyPair sudah dibuat di region yang sama.
- **Prevention:** Jangan hardcode AMI ID lama tanpa verifikasi — AMI bisa deprecated. Pertimbangkan menggunakan SSM Parameter untuk AMI terbaru otomatis (materi lanjutan).

---

# BAGIAN G — EVALUASI TEORI

## G.1 Pilihan Ganda — GitHub Actions (20 Soal)

1. File workflow GitHub Actions wajib disimpan di folder...
   a) `.github/workflow/`
   b) `.github/workflows/`
   c) `github/actions/`
   d) `.actions/workflows/`

2. Apa yang dimaksud dengan "Event" dalam GitHub Actions?
   a) Nama file YAML
   b) Kejadian yang memicu workflow berjalan
   c) Mesin virtual tempat job berjalan
   d) Komponen reusable dalam step

3. Key YAML yang digunakan untuk menentukan runner adalah...
   a) `uses:`
   b) `on:`
   c) `runs-on:`
   d) `jobs:`

4. Apa fungsi `actions/checkout@v4`?
   a) Menginstall Node.js
   b) Mengambil salinan isi repository ke dalam runner
   c) Mengupload artifact
   d) Membuat secret baru

5. Event yang digunakan untuk trigger workflow secara manual adalah...
   a) `push`
   b) `pull_request`
   c) `workflow_dispatch`
   d) `schedule`

6. Secara default, dua job tanpa `needs` akan berjalan...
   a) Berurutan
   b) Paralel
   c) Tidak berjalan sama sekali
   d) Bergantung pada urutan penulisan di file

7. Fungsi `needs:` dalam job adalah...
   a) Menentukan runner
   b) Membuat job bergantung pada job lain
   c) Mengatur secret
   d) Mengatur artifact

8. Jika job `build` gagal dan job `test` memiliki `needs: build`, maka job `test` akan...
   a) Tetap berjalan seperti biasa
   b) Otomatis di-skip
   c) Berjalan tapi dengan warning
   d) Error tanpa penjelasan

9. Di mana tempat menyimpan credential AWS agar aman di GitHub Actions?
   a) Langsung di file YAML
   b) Di file README
   c) GitHub Secrets
   d) Di comment YAML

10. Apa yang membedakan Variable dan Secret di GitHub Actions?
    a) Tidak ada bedanya
    b) Secret otomatis disamarkan di log, Variable tidak
    c) Variable hanya untuk angka
    d) Secret hanya bisa dipakai di job pertama

11. Apa itu Artifact dalam GitHub Actions?
    a) Source code asli di repository
    b) File hasil proses workflow yang bisa disimpan/didownload
    c) Nama lain dari Secret
    d) Runner khusus untuk build

12. Action yang digunakan untuk mengupload artifact adalah...
    a) `actions/checkout@v4`
    b) `actions/upload-artifact@v4`
    c) `actions/setup-node@v4`
    d) `actions/download@v4`

13. `runs-on: ubuntu-latest` menunjukkan bahwa job dijalankan di...
    a) Runner self-hosted milik user
    b) Runner GitHub-hosted berbasis Ubuntu
    c) Runner Windows
    d) Runner macOS

14. Step dalam satu job dieksekusi secara...
    a) Paralel
    b) Acak
    c) Berurutan dari atas ke bawah
    d) Berurutan dari bawah ke atas

15. Apa fungsi `with:` dalam sebuah step yang menggunakan action?
    a) Menentukan nama step
    b) Memberikan parameter/konfigurasi ke action tersebut
    c) Menentukan runner
    d) Mengatur secret

16. Filter berikut menyebabkan workflow hanya berjalan pada push ke branch tertentu:
    ```yaml
    on:
      push:
        branches:
          - main
    ```
    Jika push dilakukan ke branch `dev`, maka workflow akan...
    a) Tetap berjalan
    b) Tidak berjalan sama sekali
    c) Berjalan tapi gagal
    d) Berjalan dengan warning

17. Kesalahan umum penyebab workflow tidak terdeteksi GitHub adalah...
    a) Nama job terlalu panjang
    b) File YAML tidak berada di folder `.github/workflows/`
    c) Menggunakan `ubuntu-latest`
    d) Menggunakan lebih dari 1 step

18. Manfaat utama Continuous Integration (CI) adalah...
    a) Mempercantik tampilan repository
    b) Mendeteksi bug lebih awal melalui build & test otomatis
    c) Mengganti kebutuhan dokumentasi
    d) Menghapus riwayat commit lama

19. Perbedaan Continuous Delivery dan Continuous Deployment adalah...
    a) Tidak ada perbedaan
    b) Delivery butuh approval manual sebelum ke production, Deployment otomatis penuh
    c) Delivery hanya untuk mobile app
    d) Deployment tidak memakai CI sama sekali

20. Best practice ketika membaca secret di dalam step adalah...
    a) `echo $SECRET` untuk memastikan nilainya benar
    b) Menampilkan secret di nama step
    c) Memverifikasi keberadaan secret tanpa mencetak nilainya
    d) Menyimpan secret ke file publik

## G.2 Pilihan Ganda — AWS CloudFormation (20 Soal)

1. Apa itu CloudFormation Template?
   a) Mesin virtual AWS
   b) File YAML/JSON berisi definisi infrastruktur yang diinginkan
   c) Nama lain dari IAM Role
   d) Dashboard AWS Console

2. Bagian mana dari template CloudFormation yang WAJIB ada?
   a) `Parameters`
   b) `Outputs`
   c) `Resources`
   d) `Mappings`

3. Apa fungsi `Parameters` dalam template?
   a) Menampilkan hasil deployment
   b) Membuat template fleksibel dengan input saat deploy
   c) Mendefinisikan resource
   d) Menghapus stack

4. Fungsi intrinsic `!Ref` digunakan untuk...
   a) Menghapus resource
   b) Mengambil nilai utama dari Parameter atau Resource
   c) Menjalankan script
   d) Membuat Change Set

5. Fungsi `!GetAtt` digunakan untuk...
   a) Mengambil atribut spesifik dari suatu resource
   b) Menghapus stack
   c) Membuat parameter baru
   d) Validasi template

6. Apa itu CloudFormation Stack?
   a) File template mentah
   b) Kumpulan resource AWS yang dibuat & dikelola bersama dari satu template
   c) Nama lain dari Security Group
   d) Command CLI AWS

7. Status yang menandakan stack berhasil dibuat adalah...
   a) `CREATE_IN_PROGRESS`
   b) `CREATE_COMPLETE`
   c) `CREATE_FAILED`
   d) `DELETE_COMPLETE`

8. Jika update stack gagal di tengah proses, CloudFormation akan...
   a) Membiarkan infrastruktur setengah jadi
   b) Otomatis rollback ke kondisi sebelumnya
   c) Menghapus seluruh stack
   d) Mengulang proses tanpa henti

9. Command untuk memvalidasi template sebelum deploy adalah...
   a) `aws cloudformation deploy`
   b) `aws cloudformation validate-template`
   c) `aws cloudformation delete-stack`
   d) `aws s3 validate`

10. Apa fungsi `Outputs` dalam template?
    a) Menentukan region AWS
    b) Mengeluarkan informasi hasil deployment (misal bucket name, IP)
    c) Mengatur Parameter
    d) Membuat Change Set

11. Resource type untuk membuat S3 bucket adalah...
    a) `AWS::S3::Object`
    b) `AWS::S3::Bucket`
    c) `AWS::EC2::Bucket`
    d) `AWS::Storage::S3`

12. `AllowedValues` pada Parameter berfungsi untuk...
    a) Menentukan nilai default
    b) Membatasi input hanya ke daftar nilai tertentu
    c) Menghapus parameter yang tidak dipakai
    d) Mengatur region

13. Fungsi `Fn::Sub` digunakan untuk...
    a) Menghapus resource
    b) Menggabungkan string dengan variabel di dalamnya
    c) Membuat Security Group
    d) Validasi template

14. Command untuk menghapus CloudFormation stack adalah...
    a) `aws cloudformation remove-stack`
    b) `aws cloudformation delete-stack`
    c) `aws cloudformation destroy`
    d) `aws s3 rm stack`

15. Apa yang dimaksud dengan Change Set?
    a) Daftar user yang mengakses stack
    b) Preview perubahan sebelum benar-benar diterapkan ke stack
    c) File backup template
    d) Log error CloudFormation

16. Sebuah EC2 Instance membutuhkan referensi ke Subnet menggunakan properti...
    a) `NetworkId`
    b) `SubnetId`
    c) `VpcRef`
    d) `SubnetRef`

17. Prinsip keamanan IAM yang dianjurkan dalam pengelolaan credential adalah...
    a) AdministratorAccess untuk semua user
    b) Least privilege
    c) Membagikan access key ke semua tim
    d) Menonaktifkan IAM sepenuhnya

18. Jika Security Group memiliki `FromPort: 80` dan `ToPort: 80` dengan `CidrIp: 0.0.0.0/0`, artinya...
    a) Semua port terbuka untuk semua IP
    b) Hanya port 80 (HTTP) terbuka untuk semua IP
    c) Port 80 hanya terbuka untuk IP tertentu
    d) Tidak ada port yang terbuka

19. Untuk membaca penyebab kegagalan (`CREATE_FAILED`) pada stack, sebaiknya peserta memeriksa...
    a) Tab Outputs
    b) Tab Events
    c) Tab Parameters
    d) Tab Resources saja tanpa detail

20. Mengapa nama S3 Bucket harus unik secara global?
    a) Karena aturan AWS, bucket berbagi namespace global di seluruh pengguna AWS dunia
    b) Karena hanya berlaku di satu region saja
    c) Karena S3 tidak mendukung banyak bucket
    d) Tidak ada alasan khusus, hanya rekomendasi

## G.3 Benar / Salah (10 Soal)

1. GitHub Actions hanya bisa dijalankan di runner berbasis Linux. **(Salah — tersedia juga Windows dan macOS)**
2. File workflow GitHub Actions harus berformat YAML dan berada di `.github/workflows/`. **(Benar)**
3. Secara default, semua job dalam satu workflow berjalan secara berurutan tanpa perlu `needs`. **(Salah — default paralel)**
4. Secrets di GitHub Actions otomatis disamarkan di log jika tidak sengaja tercetak. **(Benar)**
5. CloudFormation dapat melakukan rollback otomatis jika update stack gagal. **(Benar)**
6. Bagian `Resources` dalam template CloudFormation bersifat opsional. **(Salah — wajib)**
7. `!Ref` dan `!GetAtt` selalu menghasilkan nilai yang sama persis. **(Salah — fungsinya berbeda)**
8. Least privilege berarti memberikan akses seluas mungkin agar tidak menghambat pekerjaan. **(Salah — sebaliknya, akses seminimal mungkin sesuai kebutuhan)**
9. Artifact di GitHub Actions memiliki masa retensi (tidak disimpan selamanya). **(Benar)**
10. Menghapus CloudFormation stack akan otomatis menghapus seluruh resource yang dibuat stack tersebut (kecuali ada proteksi khusus). **(Benar)**

## G.4 Soal Essay (10 Soal)

1. Jelaskan perbedaan antara Continuous Integration, Continuous Delivery, dan Continuous Deployment!
2. Jelaskan alur Event → Workflow → Job → Step → Action → Runner dengan bahasa sendiri, sertakan analogi!
3. Mengapa setiap job di GitHub Actions membutuhkan step `actions/checkout` di awal, padahal repository sudah ada di GitHub?
4. Jelaskan kapan sebaiknya menggunakan `needs:` dalam job, dan apa konsekuensinya terhadap waktu total eksekusi pipeline!
5. Jelaskan mengapa credential AWS tidak boleh ditulis langsung di file YAML, dan sebutkan minimal dua cara aman untuk mengelolanya!
6. Jelaskan konsep Infrastructure as Code dan bandingkan dengan cara manual melalui AWS Console!
7. Jelaskan hubungan antara Template, Stack, dan Resource dalam CloudFormation menggunakan analogi blueprint bangunan!
8. Jelaskan perbedaan fungsi `!Ref` dan `!GetAtt` beserta contoh penggunaannya masing-masing!
9. Jelaskan mengapa urutan pembuatan resource VPC → Internet Gateway → Subnet → Route Table → Security Group → EC2 penting diperhatikan!
10. Jelaskan alur integrasi GitHub Actions dengan CloudFormation dari push kode hingga infrastruktur AWS terbentuk!

## G.5 Soal Troubleshooting (5 Soal)

1. Sebuah workflow tidak muncul sama sekali di tab Actions meskipun sudah di-push. Sebutkan minimal 2 kemungkinan penyebab dan cara mengatasinya!
2. Job `test` menggunakan `needs: build`, tetapi `build` gagal. Jelaskan apa yang terjadi pada job `test` dan mengapa itu bukan bug!
3. Deploy CloudFormation gagal dengan status `CREATE_FAILED` pada resource `AWS::EC2::Instance` dengan pesan terkait AMI. Jelaskan langkah investigasi yang harus dilakukan!
4. Sebuah Security Group berhasil dibuat tetapi peserta tidak bisa mengakses EC2 via SSH maupun HTTP. Sebutkan 3 hal yang perlu diperiksa!
5. Delete stack CloudFormation gagal dengan status `DELETE_FAILED` karena S3 bucket tidak kosong. Jelaskan penyebabnya dan langkah penyelesaiannya!

---

# ANSWER KEY

## Pilihan Ganda — GitHub Actions
1-b, 2-b, 3-c, 4-b, 5-c, 6-b, 7-b, 8-b, 9-c, 10-b, 11-b, 12-b, 13-b, 14-c, 15-b, 16-b, 17-b, 18-b, 19-b, 20-c

## Pilihan Ganda — AWS CloudFormation
1-b, 2-c, 3-b, 4-b, 5-a, 6-b, 7-b, 8-b, 9-b, 10-b, 11-b, 12-b, 13-b, 14-b, 15-b, 16-b, 17-b, 18-b, 19-b, 20-a

## Benar/Salah
1-Salah, 2-Benar, 3-Salah, 4-Benar, 5-Benar, 6-Salah, 7-Salah, 8-Salah, 9-Benar, 10-Benar

## Essay & Troubleshooting
*(Dinilai kualitatif oleh instruktur/juri berdasarkan pemahaman konsep, kelengkapan penjelasan, dan ketepatan analogi/istilah teknis yang digunakan sesuai pembahasan pada Bab 1–24.)*

---

# PENUTUP

Setelah menyelesaikan seluruh modul ini, peserta diharapkan mampu:

```text
✓ Membangun CI pipeline dari nol menggunakan GitHub Actions
✓ Mengelola credential secara aman menggunakan Secrets
✓ Menulis CloudFormation template dari resource sederhana hingga full-stack
✓ Mengintegrasikan GitHub Actions dengan CloudFormation untuk deployment otomatis
✓ Melakukan troubleshooting terhadap pipeline maupun infrastruktur yang gagal
✓ Menerapkan praktik keamanan dasar (least privilege, tanpa credential bocor)
```

Materi lanjutan yang bisa dieksplorasi setelah modul ini (di luar cakupan modul ini): OIDC penuh untuk GitHub-AWS tanpa static key, multi-stack CloudFormation dengan Nested Stacks, serta orkestrasi container (Kubernetes/ECS) sebagai kelanjutan dari materi Docker + GHCR.

**Selamat berlatih dan semoga sukses di LKS!**
