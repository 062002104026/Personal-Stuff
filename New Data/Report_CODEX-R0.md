# Report: UI/UX + Process Improvement (R0)

**Tanggal Dibuat:** 09 Januari 2026
**Status:** Draft
**Tujuan:** Daftar peningkatan yang bisa di-tweak dari kondisi saat ini untuk UI/UX, proses, dan perapihan kode.

---

## OVERVIEW

Scope laporan ini fokus pada perbaikan yang realistis dan langsung terasa:

- UI/UX: konsistensi tampilan, keterbacaan, dan alur kerja user.
- Proses: import, export, logging, dan feedback ke user.
- Perapihan: pengurangan duplikasi, struktur lebih rapi, dan performa stabil.

---

## KATEGORI TASKS

### 1. UI/UX - Konsistensi Visual (HIGH)

**Status:** TODO
**Priority:** HIGH
**Files:** `ui/main_window_ui.py`, `ui/R4-design-ui.ui`

**Problem:**
Font, warna, dan spacing belum konsisten di seluruh tab. Ada perbedaan gaya tombol dan label antar-tab.

**Action Required:**

1. Standarisasi font (misal: "JetBrains Mono" untuk label, "Segoe UI" untuk tombol).
2. Standarisasi warna tombol:
   - Primary: hijau (OK/Start/Export)
   - Secondary: biru (Browse/Info)
   - Danger: merah (Cancel/Stop)
3. Standarisasi padding/margin groupBox agar layout lebih rapi.
4. Pastikan semua LCD/display ada label unit yang jelas.

**Verification:**

- Tampilan antar-tab konsisten.
- Tidak ada tombol dengan style berbeda tanpa alasan.

---

### 2. UI/UX - Flow Data Import (HIGH)

**Status:** TODO
**Priority:** HIGH
**Files:** `ui/main_window.py`

**Problem:**
Flow import sudah jalan, tapi feedback ke user masih minimal dan tidak ada ringkasannya di UI selain log.

**Action Required:**

1. Tambahkan ringkasan import (rows total, rows valid, rows skipped) pada `label_status_inisiasidata`.
2. Tambahkan alasan data skipped (V=0/-1, PF=0).
3. Setelah import, auto-scroll Home tab ke area grafik/table.

**Verification:**

- User langsung tahu apa yang masuk dan apa yang dibuang.

---

### 3. HOME Tab - Data Fokus & Filter (MEDIUM)

**Status:** TODO
**Priority:** MEDIUM
**Files:** `ui/main_window.py`

**Problem:**
Filter tanggal Average/Daily sudah ada, tapi belum ada indikator "range aktif" dan belum ada tombol "Apply".

**Action Required:**

1. Tambahkan label "Active Range" (start-end).
2. Tambahkan tombol "Apply Range" untuk Average.
3. Tambahkan tombol "Reset Range" (kembali ke full data).

**Verification:**

- User paham filter sedang aktif atau tidak.

---

### 4. Resume/Export - UX & Kontrol (MEDIUM)

**Status:** TODO
**Priority:** MEDIUM
**Files:** `ui/main_window.py`, `ui/export_manager.py`

**Problem:**
Export sudah berjalan tapi belum ada pilihan bagian mana yang diekspor (semuanya otomatis).

**Action Required:**

1. Tambahkan checkbox "Include Section" di Resume/Export:
   - Global Data
   - Initial Parameters
   - FTS
   - ANN
   - ARIMA
   - Metrics Comparison
2. Progress bar menampilkan nama langkah di label kecil (misal: "Step 3/7: FTS").
3. Auto-simpan lokasi terakhir export.

**Verification:**

- User bisa pilih bagian yang perlu saja.

---

### 5. Resume/Graphic - Skala & Responsif (MEDIUM)

**Status:** TODO
**Priority:** MEDIUM
**Files:** `ui/main_window.py`

**Problem:**
Grafik Resume sudah interaktif, tapi skala Y masih "gabung" semua nilai, dan resizing window belum optimal.

**Action Required:**

1. Tambahkan opsi "Auto Scale" atau "Normalize".
2. Tambahkan label nilai pada hover (sudah ada), tambah display unit.
3. Pastikan chart resize mengikuti ukuran tab.

**Verification:**

- Grafik tetap jelas di layar kecil.

**Existing - Log Events (Current)**

- Startup: reset cache runtime, load data awal.
- Main/Data: pilih folder, pilih file JSON, mulai import, hasil import (rows found/inserted), status sync Home+Initial.
- Home: load Average/Daily, jumlah row, ringkasan nilai (V/A/W/E/F/PF).
- Analysis: START RUN, source range, parameter yang dipakai.
- Preprocess: resample info, contoh raw sample, split train/test.
- Baseline: Naive & MA metrics.
- FTS/ANN/ARIMA: detail proses + metrics + beberapa artifact ringkas.
- Resume: simpan experiment log/results ke DB.
- Export: PDF report/export log (success/fail).
- UI actions: cancel analysis, clear data, open SQL (export temp), download panel (info), errors/warnings.

| SIMBOL EVENT | Category | Triggered effect of |
|---|---|---|
| `[pipeli]` | Analysis | Penanda lifecycle run (START/END RUN) |
| `[source]` | Data | Range data raw yang dipakai analisis |
| `[prepro]` | Preprocess | Resample, split, raw sample, formula energi |
| `[baseline]` | Analysis | Hitung Naive & Moving Avg metrics |
| `[fts]` | Analysis | FTS training/forecast detail + artifacts ringkas |
| `[ann]` | Analysis | ANN config, loss, forecast detail |
| `[arima]` | Analysis | ARIMA stats (AIC/BIC) atau error |
| `[eval]` | Metrics | MAE/RMSE/MAPE result + formula |
| `[resume]` | UI/Resume | Ringkasan nilai ke UI (live/prediksi) |
| `Export:` | Export | Step progres export PDF + hasil |
| `Startup reset` | System | Bersihkan cache runtime saat app start |
| `Import success` | Import | Ringkasan rows found/inserted setelah import |

---

### 6. Logging - Keterbacaan (LOW)

**Status:** TODO
**Priority:** LOW
**Files:** `utils/app_logger.py`, `ui/main_window.py`

**Problem:**
Log sudah kontras, tapi masih belum ada "filter log" atau "clear".

**Action Required:**

1. Tambahkan tombol "Clear Log".
2. Tambahkan filter dropdown: INFO, WARNING, ERROR, ALL.
3. Tambahkan "Copy to Clipboard" untuk satu baris log terpilih.

**Verification:**

- Log mudah dibaca dan cepat dicari.

---

### 7. Code Cleanup - Duplikasi (MEDIUM)

**Status:** TODO
**Priority:** MEDIUM
**Files:** `ui/main_window.py`

**Problem:**
Ada beberapa duplikasi logic (contoh: `_load_database_table` dipanggil lebih dari sekali saat init).

**Action Required:**

1. Satukan pemanggilan `_load_database_table()` hanya sekali saat startup.
2. Satukan logika pembentukan tabel Home ke helper (sudah ada `_build_home_table`, lanjutkan refactor).
3. Kurangi logika import yang tersebar di dua tempat berbeda.

**Verification:**

- Startup lebih bersih dan tidak ada double work.

---

### 8. Performance - Data Besar (MEDIUM)

**Status:** TODO
**Priority:** MEDIUM
**Files:** `ui/main_window.py`, `database/db_manager.py`

**Problem:**
Jika dataset besar, rendering tabel dan grafik bisa berat.

**Action Required:**

1. Implement downsampling untuk grafik (misal: max 2000 points).
2. Gunakan QSortFilterProxyModel untuk filter tabel tanpa reloading data.
3. Cache hasil query Home Average/Daily agar tidak query ulang saat checkbox berubah.

**Verification:**

- UI tetap responsif untuk data besar.

---

### 9. Resource Cleanup (LOW)

**Status:** TODO
**Priority:** LOW
**Files:** `utils/resource_manager.py`

**Problem:**
Ada potensi error cleanup saat interpreter shutdown (log sebelumnya menunjukkan RuntimeError).

**Action Required:**

1. Pastikan cleanup tidak memakai executor saat shutdown.
2. Tambahkan guard `if executor is None or shutting_down`.

**Verification:**

- Tidak ada error "cannot schedule new futures after interpreter shutdown".

---

### 10. Testing Checklist (HIGH)

**Status:** TODO
**Priority:** HIGH
**Files:** `docs/`

**Problem:**
Belum ada checklist verifikasi manual setelah perubahan.

**Action Required:**

1. Buat `docs/Testing_Checklist-R4.md`.
2. Checklist minimal:
   - Import JSON
   - Home Average/Daily render
   - Export Resume
   - Resume Graphic (checkbox toggle)
   - Log export
   - Open SQL

**Verification:**

- QA manual bisa diulang dengan konsisten.

---

## PROGRESS SUMMARY

| Priority | Total | Done | In Progress | Pending |
| -------- | ----: | ---: | ----------: | ------: |
| HIGH     |     3 |    0 |           0 |       3 |
| MEDIUM   |     5 |    0 |           0 |       5 |
| LOW      |     2 |    0 |           0 |       2 |
| TOTAL    |    10 |    0 |           0 |      10 |

---

## RECOMMENDED EXECUTION ORDER

1. UI/UX konsistensi visual
2. Flow import & feedback user
3. Export UX controls
4. Performance (downsampling + cache)
5. Code cleanup
6. Logging & cleanup
7. Testing checklist

---

**Last Updated:** 09 Januari 2026
