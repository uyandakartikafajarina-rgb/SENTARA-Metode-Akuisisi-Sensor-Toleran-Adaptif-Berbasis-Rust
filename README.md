# SENTARA — ESP32-S3 + DHT22 + LCD I2C + 3 LED | Rust

**Sensor Tolerant Adaptive Rust-based Acquisition**  
Departemen Teknik Instrumentasi — Fakultas Vokasi — ITS 2026  
Penulis: **Vyanda Kartika Fajarina & Aulia Qotrunnada** | NRP: **2042241030 7 2042241031**

---

## Konversi Arduino → Rust (Pin Identik)

| Arduino (C++) | Rust | Fungsi |
|---|---|---|
| `#define DHT_PIN 18` | `gpio18` | DHT22 DATA |
| `#define LED_HIJAU 15` | `gpio15` | LED Normal (hijau) |
| `#define LED_KUNING 16` | `gpio16` | LED Warning (kuning) |
| `#define LED_MERAH 17` | `gpio17` | LED Critical (merah) |
| `Wire.begin(8, 9)` | `gpio8=SDA, gpio9=SCL` | LCD I2C addr 0x27 |

---

## Logika Status (Identik dengan Arduino)

| Kondisi | Status | LED | LCD |
|---|---|---|---|
| T<35°C dan RH<80% | NORMAL | 🟢 Hijau | Status: NORMAL |
| T≥35°C atau RH≥80% | WARNING | 🟡 Kuning | Status: WARNING |
| T≥40°C atau RH≥90% | KRITIS | 🔴 Merah (kedip) | Status: KRITIS! |
| Sensor gagal | ERROR | 🔴 Merah (kedip) | Sensor Error! |

---

## Cara Menjalankan

### 1. Install toolchain (sekali saja)
```powershell
cargo install espup
espup install
```

### 2. Build (setiap kali)
```powershell
# Di PowerShell
. "$env:USERPROFILE\export-esp.ps1"
cd SENTARA-FINAL
cargo build
```

### 3. Run Wokwi Simulator
- Buka VS Code → buka folder `SENTARA-FINAL`
- Buka file `wokwi/diagram.json`
- Tekan **F1** → `Wokwi: Start Simulator`
- LED otomatis nyala sesuai suhu DHT22

---

## GNUPlot — Buat Grafik

### Install GNUPlot (Windows)
Download dari: http://www.gnuplot.info/download.html

### Jalankan plot
```bash
cd gnuplot/
gnuplot plot_temperature.gp   # Grafik Suhu
gnuplot plot_humidity.gp      # Grafik Kelembaban
gnuplot plot_dashboard.gp     # Dashboard 4-panel lengkap
```

### Output grafik
- `sentara_temperature.png` — Grafik suhu raw, kalibrasi, moving avg, heat index
- `sentara_humidity.png`    — Grafik kelembaban raw, kalibrasi, moving avg
- `sentara_dashboard.png`   — Dashboard 4-panel (suhu + kelembaban + perbandingan + heat index)

---

## Format CSV Output (untuk GNUPlot)

```
timestamp_ms, temp_raw, hum_raw, temp_cal, hum_cal, heat_index, avg_temp, avg_hum, reliability
0, 28.5, 65.0, 28.0, 67.0, 28.8, 28.0, 67.0, 100.0
```

---

## Struktur Proyek

```
SENTARA-FINAL/
├── src/main.rs          ← Kode Rust utama (konversi dari Arduino)
├── wokwi/diagram.json   ← Rangkaian Wokwi (buka di VS Code)
├── gnuplot/
│   ├── plot_temperature.gp  ← Script grafik suhu
│   ├── plot_humidity.gp     ← Script grafik kelembaban
│   ├── plot_dashboard.gp    ← Script dashboard 4-panel
│   └── run_plots.sh         ← Jalankan semua sekaligus (Linux/Mac)
├── data/sentara_log.csv ← Data sampel untuk GNUPlot
├── Cargo.toml
├── wokwi.toml
├── build.rs
└── sdkconfig.defaults
```
