#  Sripilan yang harus di install


**Modul lengkap:** [GitAction.md](./GitAction.md)


##  Alat sing di butuhkn

### 1. Akun & Platform

| Kebutuhan | Keterangan |
|---|---|
| **Akun GitHub** | Daftar gratis di [github.com](https://github.com) |
| **Akun AWS** | Daftar di [aws.amazon.com](https://aws.amazon.com) |
| **Git terinstal** | Download di [git-scm.com](https://git-scm.com) |

### 2. Software di Komputer Lokal

| Software | Fungsi | Link |
|---|---|---|
| **Git** | Version control (wajib) | [git-scm.com](https://git-scm.com) |
| **VS Code** (atau editor lain) | Menulis file YAML workflow | [code.visualstudio.com](https://code.visualstudio.com) |
| **Terminal / PowerShell** | Menjalankan perintah git & CLI | Sudah ada di Windows/macOS/Linux |
| **AWS CLI** | Berinteraksi dengan AWS dari terminal (untuk Bagian B) | [docs.aws.amazon.com/cli](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) |

> **Opsional tapi disarankan:** Ekstensi VS Code `YAML` (Red Hat) agar penulisan file `.yml` lebih nyaman (ada autocomplete & validasi syntax).

### 3. Konfigurasi Git di Lokal

Setelah Git terinstal, jalankan perintah ini **sekali saja** di terminal untuk mengatur identitas kamu:

```bash
git config --global user.name "Nama Kamu"
git config --global user.email "email@kamu.com"
```

Cek hasilnya:

```bash
git config --list
```

### 4. Setup Repositori Latihan

```bash
# Clone repo ini ke komputermu
git clone https://github.com/<username>/Git_Action.git

# Masuk ke folder project
cd Git_Action

# Buat branch latihan baru (jangan langsung edit di main)
git checkout -b latihan/nama-kamu
```

### 5. Struktur Folder Workflow GitHub Actions

Semua file workflow GitHub Actions **wajib** diletakkan di:

```text
.github/
└── workflows/
    └── nama-workflow.yml
```

Buat folder ini terlebih dahulu sebelum mulai menulis workflow:

```bash
mkdir -p .github/workflows
```

---

## Peringatan Biaya AWS

Bagian B dan C modul ini menggunakan resource AWS (EC2, VPC, CloudFormation stack) yang **berpotensi menimbulkan biaya** jika dibiarkan menyala.

**Selalu jalankan cleanup setelah selesai latihan:**

```bash
aws cloudformation delete-stack --stack-name nama-stack-kamu
```

Atau hapus stack langsung dari **AWS Console → CloudFormation → Stacks → Delete**.

---
