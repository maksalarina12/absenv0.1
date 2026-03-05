# SimKuliah Auto Attendance Bot

<div align="center">

![Python](https://img.shields.io/badge/Python-3.12-blue?style=flat-square&logo=python)
![Selenium](https://img.shields.io/badge/Selenium-4.35-43B02A?style=flat-square&logo=selenium)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-Automated-2088FF?style=flat-square&logo=github-actions)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)
![Platform](https://img.shields.io/badge/Platform-USK_SimKuliah-red?style=flat-square)

Automated attendance bot for Universitas Syiah Kuala's SimKuliah portal.  
Runs on a scheduled GitHub Actions pipeline — no server required.

[English](#english) · [Bahasa Indonesia](#bahasa-indonesia)

</div>

---

<a name="english"></a>
# English

## Overview

SimKuliah Auto Attendance Bot automates the daily attendance check-in process on [simkuliah.usk.ac.id](https://simkuliah.usk.ac.id). It uses a headless Chrome browser driven by Selenium, triggered automatically by GitHub Actions cron jobs based on your class schedule.

**Key capabilities:**
- Headless browser login with credential injection via environment secrets
- Automated detection and click of attendance and confirmation buttons
- Handles up to 2 sequential attendance sessions per run
- Push notification via [ntfy.sh](https://ntfy.sh) for success, no-schedule, and error states
- Screenshot capture at each critical step for audit and debugging
- Time-gating: skips execution outside weekday lecture hours (08:00–21:00 WIB)

## Architecture

```
+--------------------------------------------------+
|              GitHub Actions Runner               |
|                                                  |
|  +----------+    +----------+    +-----------+  |
|  |   Cron   |    | absen.py |    | SimKuliah |  |
|  | Schedule |--->| Selenium |--->|  Portal   |  |
|  +----------+    +-----+----+    +-----------+  |
|                        |                         |
|                  +-----v----+                    |
|                  | ntfy.sh  |---> Your Phone     |
|                  +----------+                    |
+--------------------------------------------------+
```

**Execution flow:**
1. Cron trigger fires at the scheduled time (WIB timezone, offset applied for UTC)
2. Runner checks out the repo, sets up Chrome and Python
3. `absen.py` reads credentials from GitHub Secrets via environment variables
4. Time-gate check: exits early if outside weekday lecture window
5. Logs in to SimKuliah, navigates to the `/absensi` page
6. Iterates up to 2 times, clicking available attendance buttons
7. Sends ntfy push notification based on outcome
8. Saves screenshots as artifacts for debugging

## Getting Started

### Prerequisites

- A GitHub account
- NPM (Nomor Pokok Mahasiswa) and SimKuliah password
- A smartphone with [ntfy](https://ntfy.sh) installed (optional, for notifications)

### Installation

**1. Fork this repository**

```bash
git clone https://github.com/NapoleonPro/absenv0.1.git
cd absenv0.1
```

**2. Configure GitHub Secrets**

Go to **Settings** > **Secrets and variables** > **Actions** > **New repository secret**.

| Secret Name | Description |
|-------------|-------------|
| `NPM` | Your student ID (Nomor Pokok Mahasiswa) |
| `PASSWORD` | Your SimKuliah account password |

**3. Enable GitHub Actions**

Navigate to the **Actions** tab in your repository and click **Enable Workflows**.

**4. Adjust the schedule**

Edit `.github/workflows/absen.yml` to match your class timetable. See [Schedule Configuration](#schedule-configuration).

## Configuration

### Schedule Configuration

Schedules are defined in `.github/workflows/absen.yml` using cron syntax. GitHub Actions runs in **UTC**, so WIB (UTC+7) times must be offset by -7 hours.

**Conversion example:**
```
WIB 10:45  ->  UTC 03:45  ->  cron: '45 3 * * 1'
```

**Default schedule (adjust to your own timetable):**

| Day | WIB Time | Cron (UTC) |
|-----|----------|------------|
| Monday | 10:45 | `45 3 * * 1` |
| Tuesday | 10:45 | `45 3 * * 2` |
| Wednesday | 16:45 | `45 9 * * 3` |
| Thursday | 08:00 | `0 1 * * 4` |
| Thursday | 14:00 | `0 7 * * 4` |
| Friday | 16:35 | `35 9 * * 5` |
| Saturday | 14:00 | `0 7 * * 6` |

### Bot Behavior

The bot includes a time-gate guard that prevents execution outside 08:00–21:00 WIB or on weekends:

```python
if now.weekday() >= 5 or not (8 <= now.hour < 21):
    print("Outside lecture hours. Skipping.")
    exit()
```

For back-to-back sessions, the bot will attempt up to 2 consecutive attendance clicks per run.

## Push Notifications

This bot uses [ntfy.sh](https://ntfy.sh), a free and open-source push notification service.

**Setup:**

1. Download the ntfy app on your phone ([Android](https://play.google.com/store/apps/details?id=io.heckel.ntfy) / [iOS](https://apps.apple.com/app/ntfy/id1625396347))
2. Choose a unique topic name, e.g. `yourname-attendance-bot`
3. Replace the three topic URLs in `absen.py`:

```python
os.system('curl -H "Title: Absen Berhasil" -d "..." ntfy.sh/YOUR-TOPIC-sukses')
os.system('curl -H "Title: Tidak Ada Jadwal" -d "..." ntfy.sh/YOUR-TOPIC-info')
os.system('curl -H "Title: Absen ERROR" -d "..." ntfy.sh/YOUR-TOPIC-error')
```

4. Subscribe to the same topic in the ntfy app.

**Notification states:**

| Status | Trigger |
|--------|---------|
| Absen Berhasil | Attendance button found and clicked successfully |
| Tidak Ada Jadwal | Bot ran but no attendance button was available |
| Absen ERROR | An unexpected exception occurred |

## Local Development

```bash
# Create and activate virtual environment
python -m venv mymyenv
source mymyenv/bin/activate        # Linux/macOS
# mymyenv\Scripts\activate         # Windows

# Install dependencies
pip install -r requirement.txt

# Set up local credentials
cp .env.example .env
# Edit .env:
# NPM=your_npm_here
# PASSWORD=your_password_here

# Run the bot
python absen.py
```

> Local runs require Google Chrome to be installed. The bot runs in headless mode by default.

## Tech Stack

| Component | Version | Purpose |
|-----------|---------|---------|
| Python | 3.12 | Runtime |
| Selenium | 4.35.0 | Browser automation |
| Chrome (headless) | latest | Browser engine |
| python-dotenv | 1.1.1 | Local credential management |
| GitHub Actions | — | Scheduled CI/CD runner |
| ntfy.sh | — | Push notifications |

## Roadmap

- [x] Automated login and attendance clicking
- [x] Multi-session support (up to 2 per run)
- [x] Push notifications via ntfy
- [x] Screenshot capture for audit trail
- [ ] Multi-account support
- [ ] Per-user configurable schedule without code editing
- [ ] Telegram / Discord notification support
- [ ] Retry logic on login failure

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/your-feature-name`
3. Commit your changes: `git commit -m 'feat: add some feature'`
4. Push to the branch: `git push origin feat/your-feature-name`
5. Open a Pull Request

Please follow [Conventional Commits](https://www.conventionalcommits.org/) for commit messages.

## Disclaimer

This project is intended for educational and personal productivity purposes. The developer is not responsible for any academic policy violations arising from misuse. SimKuliah may update its authentication or UI at any time, which could break this bot. Always verify your attendance was recorded correctly on SimKuliah.

---

<a name="bahasa-indonesia"></a>
# Bahasa Indonesia

## Ikhtisar

SimKuliah Auto Attendance Bot mengotomatiskan proses absensi harian di [simkuliah.usk.ac.id](https://simkuliah.usk.ac.id). Bot ini menggunakan browser Chrome headless yang dikendalikan oleh Selenium, dan dipicu secara otomatis oleh cron job GitHub Actions sesuai jadwal kuliah yang dikonfigurasi.

**Fitur utama:**
- Login otomatis dengan injeksi kredensial melalui environment secrets
- Deteksi dan klik otomatis tombol absen dan konfirmasi
- Mendukung hingga 2 sesi absensi berurutan per eksekusi
- Push notification via [ntfy.sh](https://ntfy.sh) untuk status berhasil, tidak ada jadwal, dan error
- Pengambilan screenshot di setiap langkah penting untuk audit dan debugging
- Time-gate: melewati eksekusi di luar jam kuliah hari kerja (08:00–21:00 WIB)

## Arsitektur

```
+--------------------------------------------------+
|           GitHub Actions Runner                  |
|                                                  |
|  +----------+    +----------+    +-----------+  |
|  |   Cron   |    | absen.py |    | SimKuliah |  |
|  | Schedule |--->| Selenium |--->|  Portal   |  |
|  +----------+    +-----+----+    +-----------+  |
|                        |                         |
|                  +-----v----+                    |
|                  | ntfy.sh  |---> HP Anda        |
|                  +----------+                    |
+--------------------------------------------------+
```

**Alur eksekusi:**
1. Cron trigger berjalan sesuai jadwal (timezone WIB, dikonversi ke UTC)
2. Runner melakukan checkout repo, setup Chrome dan Python
3. `absen.py` membaca kredensial dari GitHub Secrets melalui environment variable
4. Pemeriksaan time-gate: keluar lebih awal jika di luar jam kuliah
5. Login ke SimKuliah, navigasi ke halaman `/absensi`
6. Iterasi hingga 2 kali, mengklik tombol absen yang tersedia
7. Mengirim push notification via ntfy sesuai hasil eksekusi
8. Menyimpan screenshot sebagai artefak untuk debugging

## Memulai

### Prasyarat

- Akun GitHub
- NPM (Nomor Pokok Mahasiswa) dan password SimKuliah
- Smartphone dengan aplikasi [ntfy](https://ntfy.sh) (opsional, untuk notifikasi)

### Instalasi

**1. Fork repository ini**

```bash
git clone https://github.com/NapoleonPro/absenv0.1.git
cd absenv0.1
```

**2. Konfigurasi GitHub Secrets**

Buka **Settings** > **Secrets and variables** > **Actions** > **New repository secret**.

| Nama Secret | Deskripsi |
|-------------|-----------|
| `NPM` | Nomor Pokok Mahasiswa |
| `PASSWORD` | Password akun SimKuliah |

**3. Aktifkan GitHub Actions**

Buka tab **Actions** di repository dan klik **Enable Workflows**.

**4. Sesuaikan jadwal**

Edit `.github/workflows/absen.yml` sesuai jadwal kuliah Anda. Lihat bagian [Konfigurasi Jadwal](#konfigurasi-jadwal).

## Konfigurasi

<a name="konfigurasi-jadwal"></a>
### Konfigurasi Jadwal

Jadwal didefinisikan di `.github/workflows/absen.yml` menggunakan sintaks cron. GitHub Actions berjalan dalam **UTC**, sehingga waktu WIB (UTC+7) harus dikurangi 7 jam.

**Contoh konversi:**
```
WIB 10:45  ->  UTC 03:45  ->  cron: '45 3 * * 1'
```

**Jadwal default (sesuaikan dengan jadwal kuliah Anda):**

| Hari | Waktu WIB | Cron (UTC) |
|------|-----------|------------|
| Senin | 10:45 | `45 3 * * 1` |
| Selasa | 10:45 | `45 3 * * 2` |
| Rabu | 16:45 | `45 9 * * 3` |
| Kamis | 08:00 | `0 1 * * 4` |
| Kamis | 14:00 | `0 7 * * 4` |
| Jumat | 16:35 | `35 9 * * 5` |
| Sabtu | 14:00 | `0 7 * * 6` |

### Perilaku Bot

Bot memiliki time-gate guard yang mencegah eksekusi di luar jam 08:00–21:00 WIB atau pada hari akhir pekan:

```python
if now.weekday() >= 5 or not (8 <= now.hour < 21):
    print("Di luar jam kuliah. Tidak mencoba absen.")
    exit()
```

Untuk sesi berurutan, bot akan mencoba hingga 2 klik absensi per eksekusi.

## Push Notification

Bot ini menggunakan [ntfy.sh](https://ntfy.sh), layanan push notification gratis dan open-source.

**Setup:**

1. Unduh aplikasi ntfy di HP ([Android](https://play.google.com/store/apps/details?id=io.heckel.ntfy) / [iOS](https://apps.apple.com/app/ntfy/id1625396347))
2. Pilih nama topic yang unik, contoh: `namaanda-absen-bot`
3. Ganti ketiga URL topic di `absen.py`:

```python
os.system('curl -H "Title: Absen Berhasil" -d "..." ntfy.sh/TOPIC-ANDA-sukses')
os.system('curl -H "Title: Tidak Ada Jadwal" -d "..." ntfy.sh/TOPIC-ANDA-info')
os.system('curl -H "Title: Absen ERROR" -d "..." ntfy.sh/TOPIC-ANDA-error')
```

4. Subscribe ke topic yang sama di aplikasi ntfy.

**Status notifikasi:**

| Status | Pemicu |
|--------|--------|
| Absen Berhasil | Tombol absen ditemukan dan berhasil diklik |
| Tidak Ada Jadwal | Bot berjalan tetapi tidak ada tombol absen yang tersedia |
| Absen ERROR | Terjadi exception yang tidak terduga |

## Pengembangan Lokal

```bash
# Buat dan aktifkan virtual environment
python -m venv mymyenv
source mymyenv/bin/activate        # Linux/macOS
# mymyenv\Scripts\activate         # Windows

# Install dependensi
pip install -r requirement.txt

# Setup kredensial lokal
cp .env.example .env
# Edit .env:
# NPM=npm_anda
# PASSWORD=password_anda

# Jalankan bot
python absen.py
```

> Menjalankan secara lokal memerlukan Google Chrome yang sudah terinstal. Bot berjalan dalam mode headless secara default.

## Tech Stack

| Komponen | Versi | Kegunaan |
|----------|-------|----------|
| Python | 3.12 | Runtime |
| Selenium | 4.35.0 | Otomatisasi browser |
| Chrome (headless) | latest | Browser engine |
| python-dotenv | 1.1.1 | Manajemen kredensial lokal |
| GitHub Actions | — | Scheduled CI/CD runner |
| ntfy.sh | — | Push notification |

## Roadmap

- [x] Login otomatis dan klik absensi
- [x] Dukungan multi-sesi (hingga 2 per eksekusi)
- [x] Push notification via ntfy
- [x] Screenshot untuk audit trail
- [ ] Dukungan multi-akun
- [ ] Konfigurasi jadwal per pengguna tanpa edit kode
- [ ] Notifikasi via Telegram / Discord
- [ ] Retry logic saat login gagal

## Kontribusi

1. Fork repository ini
2. Buat branch fitur: `git checkout -b feat/nama-fitur-anda`
3. Commit perubahan: `git commit -m 'feat: tambah fitur tertentu'`
4. Push ke branch: `git push origin feat/nama-fitur-anda`
5. Buka Pull Request

Gunakan format [Conventional Commits](https://www.conventionalcommits.org/) untuk pesan commit.

## Disclaimer

Proyek ini dibuat untuk tujuan edukasi dan produktivitas pribadi. Developer tidak bertanggung jawab atas pelanggaran kebijakan akademik akibat penyalahgunaan. SimKuliah dapat memperbarui autentikasi atau antarmukanya sewaktu-waktu, yang berpotensi menyebabkan bot tidak berfungsi. Selalu verifikasi bahwa absensi Anda benar-benar tercatat di SimKuliah.

---

## License

```
MIT License

Copyright (c) 2025 NapoleonPro

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

<div align="center">
Made with coffee by <a href="https://github.com/NapoleonPro">NapoleonPro</a> — If this project helped you, consider leaving a star.
</div>
