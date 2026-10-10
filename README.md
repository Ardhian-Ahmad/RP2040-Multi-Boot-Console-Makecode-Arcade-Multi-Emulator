# YD-RP2040 Multi-Boot Console: MakeCode Arcade & Multi Emulator

Repositori ini merupakan bagian dari proyek pembangunan ekosistem konsol *handheld* edukasi DIY berbasis mikrokontroler Raspberry Pi Pico / RP2040. 

Konsol ini dirancang dengan arsitektur **Multi-Boot**, di mana **Microsoft MakeCode Arcade** bertindak sebagai sistem operasi utama (untuk pembelajaran *coding* dan pembuatan game). Ekosistem ini dikonfigurasi untuk dapat memuat dan menjalankan berbagai sistem tambahan (seperti *Multi Emulator* untuk game retro) langsung dari memori SD Card tanpa perlu melakukan *flashing* ulang dari komputer.

Proyek dalam repositori ini berisi modifikasi (*custom build*) dari emulator **InfoNES** yang diadaptasi khusus agar dapat berjalan mulus berdampingan dalam ekosistem *Multi-Boot* tersebut. Perangkat keras dirancang menggunakan sistem **Dual-SPI** (memisahkan jalur komunikasi Layar dan SD Card) guna mengeliminasi *glitch* grafis dan menjamin stabilitas performa.

**Personalisasi Konsol:** Mirza Ardhian Ahmad - Retro Console

---

## 🛠️ Perangkat Keras (Hardware)
*   **Mikrokontroler:** RP2040 / YD-RP2040 (Minimal 2MB Flash, kompatibel hingga 16MB)
*   **Layar:** Modul LCD TFT 2.4" / 2.8" (ILI9341) via SPI
*   **Penyimpanan:** Modul MicroSD Card (Format FAT32) via SPI
*   **Input:** Tombol *Tactile* (D-Pad, A, B, Start, Select)
*   **Audio:** Speaker / Modul Amplifier mini (via PWM)

---

## 🔌 Skema Pinout (Wiring / Pengkabelan)

Perhatian: Konfigurasi ini menggunakan dua jalur SPI yang **terpisah** (`spi0` dan `spi1`) untuk mencegah tabrakan sinyal (*bus contention*) antara layar dan memori.

### 1. Modul Layar ILI9341 (Jalur SPI 0)
*MISO pada layar tidak digunakan (diatur -1 di CMake) agar layar beroperasi secara murni sebagai output (Write-Only).*

| Pin ILI9341 | Pin RP2040 | Keterangan |
| :--- | :--- | :--- |
| **CLK / SCK** | **GPIO 18** | SPI0 SCK |
| **MOSI** | **GPIO 19** | SPI0 TX |
| **MISO** | *(Tidak dipasang)* | Dinonaktifkan (-1) |
| **CS** | **GPIO 17** | Chip Select Layar |
| **DC / RS** | **GPIO 20** | Data/Command |
| **RST / RES** | **GPIO 21** | Reset |
| **BLK / LED** | **GPIO 22** | Backlight |
| **VCC** | **3.3V** | Daya |
| **GND** | **GND** | Ground |

### 2. Modul MicroSD Card (Jalur SPI 1)
| Pin SD Card | Pin RP2040 | Keterangan |
| :--- | :--- | :--- |
| **CLK / SCK** | **GPIO 26** | SPI1 SCK |
| **MOSI** | **GPIO 27** | SPI1 TX |
| **MISO** | **GPIO 28** | SPI1 RX |
| **CS** | **GPIO 5** | Chip Select SD |
| **VCC** | **5V (VBUS)** | Daya (tergantung modul) |
| **GND** | **GND** | Ground |

### 3. Tombol Kontrol (Gamepad)
Hubungkan salah satu kaki tombol ke **GND**, dan kaki lainnya ke pin GPIO berikut (Pull-up internal telah diaktifkan oleh *software*):

| Tombol | Pin RP2040 |
| :--- | :--- |
| **UP** (Atas) | **GPIO 12** |
| **DOWN** (Bawah) | **GPIO 13** |
| **LEFT** (Kiri) | **GPIO 10** |
| **RIGHT** (Kanan) | **GPIO 11** |
| **A** | **GPIO 14** |
| **B** | **GPIO 15** |
| **SELECT** | **GPIO 6** |
| **START** | **GPIO 8** |

### 4. Modul Audio (Suara)
Sinyal audio dikirimkan menggunakan modulasi PWM. Anda bisa menyambungkannya ke modul amplifier kecil (seperti PAM8403) atau langsung ke *buzzer* piezoaktif.

| Modul Audio | Pin RP2040 | Keterangan |
| :--- | :--- | :--- |
| **Audio In / +** | **GPIO 9** | Sinyal Output Audio (PWM) |
| **GND / -** | **GND** | Ground |

---

## 🚀 Cara Kompilasi (Build)

1. Lakukan *Clone* repositori berserta modul (*submodules*):
   ```bash
   git clone [https://github.com/pondahai/rp2040-ili9341-infones.git](https://github.com/pondahai/rp2040-ili9341-infones.git) --recursive
   Buka folder menggunakan Pico - Visual Studio Code.
2. Sesuaikan pinout dan arsitektur SPI1 & SPI0 pada file software/infones/CMakeLists.txt.
3. Sesuaikan definisi tombol pada file main.cpp dan menu.cpp.
4. Konfigurasi Memori (Penting): Ubah nilai NES_FILE_ADDR di main.cpp menjadi 0x10100000 (1 MB) untuk mencegah sistem menimpa dirinya sendiri (Flash Wrap-Around) saat memuat game pada papan berkapasitas 2MB.
5. Pastikan kecepatan SPI untuk layar diatur maksimal 63000000 (63 MHz) di main.cpp untuk mencapai 60 FPS secara penuh (Overclocking layar ILI9341).
6. Jalankan CMake: Delete Cache and Reconfigure (Ctrl+Shift+P).
7. Tekan tombol Build.
8. Pindahkan (Drag & Drop) file infoNES.uf2 ke RP2040 dalam mode BOOTSEL.

## 📂 Pengaturan SD Card & Multi-Boot
* **Format MicroSD Card Anda menggunakan sistem file FAT32.
* **Letakkan file ROM NES Anda (berakhiran .nes) langsung di root (direktori utama) SD Card atau di dalam folder (File game tidak disertakan dalam repositori ini).
* **Menu BIOS kustom berbahasa Indonesia akan otomatis mencari dan menampilkan game NES Anda.
* **Mode Multi-Boot: Gunakan ekosistem bootloader pihak ketiga (seperti MakeCode Arcade Bootloader) untuk memuat file .uf2 ini langsung dari menu SD Card saat Anda ingin beralih dari mode coding ke mode Retro.

🙏 Kredit & Sumber Asli
Proyek ini adalah hasil modifikasi dan perakitan ulang (re-design) antarmuka, lokalisasi, dan perangkat keras. Sistem inti (core) berbasis pada karya open-source dari:
* **@pondahai (PCB Design & InfoNES RP2040 implementation)
* **@shuichi_takano (Pico Port)
* **@frenskefrens (Menu System & SD Card Module)

Apresiasi Khusus Ekosistem Multi-Boot:
* **Aqee (AqeeAqee) atas dedikasi dan kontribusinya dalam mengembangkan Board MakeCode Arcade arsitektur RP2040.
* **Raspberry Pi Foundation untuk rilis utilitas pembersih Flash Nuke (flash_nuke.uf2) yang sangat krusial dalam pemulihan dan penataan ulang blok memori selama masa pengembangan dan debugging konsol.
