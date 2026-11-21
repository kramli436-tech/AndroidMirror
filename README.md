# Android Mirror (`amirror`)

Aplikasi mirroring Android ke laptop Windows (video UHD + audio + kontrol keyboard & mouse + OBS Studio) yang dibangun **100% dari nol** — **tanpa root** dan **tanpa install APK** di HP.

## Fitur Utama
- **Tanpa Root & Tanpa APK**: `amirror-server.jar` (~22 KB DEX) dijalankan sementara lewat `adb shell app_process`.
- **Resolusi Full HD Optimal (`1920p`, kelipatan 16) & 30–120 FPS Stabil**:
  - 36% lebih ringan dari UHD sehingga tidak patah-patah saat main game berat, namun tetap sangat tajam (tanpa pecah-pecah) berkat **16–20 Mbps**, **H.264 High Profile** (CABAC + 8x8 DCT), **VBR**, **GOP 2 detik**, warna **BT.709**, dan *keyframe-gated flush*.
- **Audio Jernih Tanpa Pecah-Pecah (Android 11+)**:
  - Server mengirim PCM 48 kHz stereo **20 ms ter-align 4-byte** dengan buffer capture internal 8× (anti drop saat CPU HP sibuk main game).
  - Client otomatis menyesuaikan sample rate native speaker/headset Windows (**WASAPI 44.1 / 48 / 96 kHz**), pre-buffer 65 ms, **adaptive linear resampler** (±1,5% halus, tanpa potong sampel paksa / tanpa *click/pop*), dan *soft fade* 3 ms.
- **Ringan di CPU & Optimal FPS**:
  - **Zero-Copy Hardware DXVA/D3D11 GPU Decode** triple-buffered (`IMFDXGIDeviceManager` + `CopySubresourceRegion` + cached `VideoProcessorInputView`, 0 alokasi per frame di VRAM).
  - Antrian video maksimal **1 frame** (*latest-frame-wins*), `SetMaximumFrameLatency(1)`, `TCP_NODELAY`, `timeBeginPeriod(1)`.
- **Siap OBS Studio (`--obs` / `run-obs.bat`)**:
  - Judul jendela statis `amirror [OBS]` (OBS tidak pernah kehilangan target Window Capture).
  - Otomatis ubah rasio jendela saat HP berputar **Portrait ↔ Landscape** (tanpa bar hitam saat main game!).
  - Warna BT.709 akurat, VSync halus untuk streaming, borderless (`Alt+B`), ukuran 1:1 native (`Alt+1`), serta audio game terpisah via *Application Audio Capture*.
- **Audio Internal (Android 11+)**: 48 kHz stereo PCM 10 ms/frame + koreksi drift otomatis.
- **Rekam Layar Tanpa Re-Encode**: muxing langsung ke `.mkv` (`--record hasil.mkv`).

## Cara Menjalankan (Siap Pakai)

Pastikan **USB Debugging** aktif di HP Android:
- **Aplikasi Desktop GUI (Rekomendasi — tanpa jendela CMD hitam):** buka folder `android-mirror\dist\` lalu double-click **`AndroidMirror.exe`** (4.1 MB, Dark Mode modern, auto-detect HP, preset 1-klik Gaming FHD / OBS Studio / Wi-Fi, remote HP + screenshot PNG, rekam `.mkv`, dan live log).
- **Atau shortcut cepat di folder `android-mirror\`:**
  - **Main Game / Mirror USB (FHD 1920 optimal, 30–120 fps, 16 Mbps):** double-click **`run-usb.bat`**
  - **Streaming OBS Studio (FHD 1920, 60 fps stabil, 20 Mbps, judul statis):** double-click **`run-obs.bat`**
  - **Mirror via Wi-Fi (TCP/IP):** colok USB sebentar → double-click **`run-wifi.bat`**

### Cara Pakai Lewat PowerShell / CLI
```powershell
cd "android-mirror\dist"

# Full HD optimal, 120 fps, 16 Mbps (halus & tajam tanpa patah-patah)
.\amirror.exe --max-size 1920 --max-fps 120 --bitrate-mbps 16

# Mode OBS Studio (FHD 60 fps, judul tetap "amirror [OBS]", 20 Mbps, VSync halus)
.\amirror.exe --obs --max-size 1920 --max-fps 60 --bitrate-mbps 20

# Opsi UHD jika PC/HP sanggup (opsional)
.\amirror.exe --max-size 3840 --max-fps 60 --bitrate-mbps 24

# Rekam layar langsung ke MKV tanpa re-encode
.\amirror.exe --record gameplay.mkv

# Pairing Wireless Debugging (Android 11+)
.\amirror.exe --pair 192.168.1.50:37099 123456
```

## Setup OBS Studio (Kualitas Broadcast)
1. Jalankan **`run-obs.bat`** (jendela akan muncul dengan judul `amirror [OBS]`).
2. Di **OBS Studio → Sources → + → Window Capture**:
   - **Window**: `[amirror.exe]: amirror [OBS]`
   - **Capture Method**: `Windows 10 (1903 and up)`
   - **Client Area**: centang (aktif)
   - Klik kanan source → **Transform → Fit to Screen** (`Ctrl + F`).
3. Untuk **Audio Game Terpisah** (tanpa bocor suara Discord/Windows):
   - **Sources → + → Application Audio Capture (BETA)** → pilih `[amirror.exe]: amirror [OBS]`.
4. **Tips kualitas maksimal**:
   - Saat game sudah masuk mode Landscape, jendela `amirror` otomatis menyesuaikan rasio tanpa bar hitam.
   - Tekan **`Alt + B`** untuk mode Borderless, **`Alt + 1`** untuk resolusi piksel 1:1 native.
   - Di OBS → Settings → Video: gunakan **1920×1080** atau **3840×2160** (60 fps), Downscale Filter **Lanczos**, Color Space **Rec. 709**.

## Shortcut Keyboard & Mouse
| Input | Fungsi |
|---|---|
| Klik kiri / drag | Sentuh / geser layar |
| Klik kanan | **Back** (atau nyalakan layar HP jika mati) |
| Klik tengah | **Home** |
| Scroll wheel | Scroll layar |
| `Alt + F` | Toggle Fullscreen |
| `Alt + B` | Toggle **Borderless** (ideal untuk OBS) |
| `Alt + 1` | Ukuran jendela **1:1 piksel native** |
| `Alt + 2` | Ukuran jendela pas layar desktop |
| `Alt + I` | Tampilkan / sembunyikan statistik FPS & delay di title bar |
| `Alt + K` | Paksa kirim keyframe baru (refresh ketajaman instan) |
| `Alt + R` | Putar layar (Rotate) |
| `Alt + O` | Matikan / nyalakan layar fisik HP |
| `Alt + H` / `Alt + S` / `Alt + Backspace` | Home / Recent Apps / Back |
| `Alt + Up` / `Alt + Down` / `Alt + P` | Volume + / Volume - / Power |

## Build Ulang dari Source
```powershell
powershell -ExecutionPolicy Bypass -File scripts\package-dist.ps1
```
