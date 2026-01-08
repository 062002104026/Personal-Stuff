# Rumus/Formula untuk Triggered Log Events (R1)

**Tujuan:** Mapping rumus yang dipakai pada event log saat ini (1:1 dengan template log di kode).  
**Sumber event:** `docs/Report_LOG_EVENTS-R1.md`  
**Catatan:** Definisi simbol ada di bagian A agar formula lebih jelas.

---

## A. Notasi dan Variabel

| Simbol | Arti |
|---|---|
| `ts_server` | Timestamp epoch ms dari data mentah |
| `t_i` | Timestamp ke-i (datetime) |
| `dt_i` | Selisih waktu: `(t_i - t_{i-1}) / 3600` jam |
| `W` | Daya (watt) |
| `V` | Tegangan (volt) |
| `I` | Arus (ampere), di UI ditulis `A` |
| `F` | Frekuensi (Hz) |
| `PF` | Power factor |
| `E_kwh` | Energi meter kWh (nilai kumulatif dari sensor) |
| `E_range` | Selisih energi meter: `E_last - E_first` |
| `E_adj` | Energi ter-adjust: `E_kwh - E_kwh_first` |
| `n` | Jumlah data/sampel |
| `y_t` | Nilai deret waktu hasil resample (target model, satuan W) |
| `yhat_t` | Hasil prediksi pada waktu t |
| `split_ratio` | Proporsi data train (0..1) |
| `look_back` | Panjang lag untuk ANN (fixed 1) |
| `A_i` | Interval fuzzy ke-i pada FTS |
| `L` | Lebar interval FTS |
| `D1`, `D2` | Padding UoD (D1=0, D2=0.001) |
| `p,d,q` | Orde ARIMA |
| `P,D,Q,s` | Orde musiman SARIMA |
| `k` | Jumlah parameter model (AIC/BIC) |
| `epsilon` | Konstanta kecil untuk MAPE (1e-10) |

---

## B. Definisi Rumus (ID)

**F1. Range Timestamp**  
`start_ts = min(ts_server)`  
`end_ts = max(ts_server)`

**F2. Resample Mean (Watt)**  
`rule = "{interval_minutes}T"`  
`y_t = mean(W) untuk semua W di bucket resample`  
`missing_filled = count(NaN) sebelum ffill/bfill`  
Missing handling: `ffill -> bfill -> 0`

**F3. Train/Test Split (sekuensial)**  
`n = len(series)`  
`train_size = floor(n * split_ratio)`  
`train = series[0:train_size]`  
`test = series[train_size:n]`

**F4. Energi dari Daya (integrasi)**  
`dt_i = (t_i - t_{i-1}) / 3600`  
`E_kwh = sum(W_i * dt_i) / 1000`  
Catatan: `dt_0 = 0` (baris pertama).

**F5. Energi Range (kWh meter)**  
`E_range = E_kwh_last - E_kwh_first`  
`E_range = max(0, E_range)`  
Fallback: gunakan F4 jika kolom `energy_kwh` tidak ada.

**F6. Energi Adjusted untuk tabel/grafik**  
`E_adj(t_i) = E_kwh(t_i) - E_kwh_first`  
Fallback: `E_adj(t_i) = cumsum(W_i * dt_i) / 1000` (F4).

**F7. Rata-rata (Mean)**  
`x_avg = (x1 + x2 + ... + xn) / n`

**F8. Min/Max + Timestamp**  
`x_min = min(x)`  
`x_max = max(x)`  
`t_min = t[argmin(x)]`  
`t_max = t[argmax(x)]`

**F9. Baseline Naive**  
`yhat_t = y_{t-1}`  
`yhat_0 = last(train)` untuk test pertama.

**F10. Baseline Moving Average**  
`yhat_t = mean(window_values)`  
`window = max(2, fts_interval)`

**F11. FTS UoD + Interval Equal Width**  
`uod_min = min(y) - D1`  
`uod_max = max(y) + D2`  
`L = (uod_max - uod_min) / n_intervals`  
`A_i = [uod_min + (i-1)L, uod_min + iL]`  
`midpoint_i = (low_i + high_i) / 2`

**F12. FTS Fuzzify + FLR/FLRG**  
Fuzzify: pilih `A_i` jika `low_i <= y_t < high_i` (clamp jika out of range).  
`FLR: A(t-1) -> A(t)`  
`FLRG: group by LHS`

**F13. FTS Defuzzify (Chen)**  
Jika rule ada: `yhat_t = mean(midpoints of RHS)`  
Jika rule tidak ada: `yhat_t = midpoint(current state)`

**F14. ANN Scaling + Dataset**  
MinMax scaling: `x' = (x - min) / (max - min)`  
Inverse: `x = x' * (max - min) + min`  
Look-back dataset:  
`X_t = [x_{t-look_back} ... x_{t-1}]`  
`y_t = x_t`

**F15. ANN Loss (MSE)**  
`Loss = mean((y - yhat)^2)`

**F16. ARIMA/SARIMA + AIC/BIC**  
Model umum:  
`y_t = c + sum(phi_i * y_{t-i}) + sum(theta_i * e_{t-i}) + seasonal_terms + e_t`  
Forecast index: `start = len(train)`, `end = len(train) + len(test) - 1`  
`AIC = 2k - 2 ln(L)`  
`BIC = k ln(n) - 2 ln(L)`

**F17. Metrics (MAE/RMSE/MAPE)**  
`MAE = mean(|y - yhat|)`  
`RMSE = sqrt(mean((y - yhat)^2))`  
`y_safe = where(y == 0, epsilon, y)`  
`MAPE = mean(|(y - yhat) / y_safe|) * 100`

**F18. Resume Live (nilai terakhir)**  
`Live = value at last row` (V, I, W, PF, F)

**F19. Prediksi t+1**  
`Prediksi t+1 = forecast[0]` dari masing-masing metode (FTS/ANN/ARIMA)

**F20. Daily Range Filter (Tab Home Daily)**  
`start_ts = date at 00:00:00`  
`end_ts = date at 23:59:59.999999`  
Filter: `start_ts <= ts_server <= end_ts`

**F21. Global Data Export (Terendah/Tertinggi/Rata-rata)**  
`min_val = min(series)`  
`max_val = max(series)`  
`avg = mean(series)`  
`min_ts = t[argmin(series)]`  
`max_ts = t[argmax(series)]`  
Energi: gunakan `E_adj` (F6).

**F22. Home Summary (Average/Daily)**  
V/I/W/F/PF: gunakan F7  
Energi: gunakan F5 (fallback F4)

---

## C. Mapping Event -> Rumus (1:1 Template Log)

### ui/main_window.py

| Log Event Template | Formula | Keterangan |
|---|---|---|
| `[source] Rows={len(df_src)} \| Range={start_ts} -> {end_ts}` | F1 | Range = min/max timestamp pada data analisis |
| `[param] Split=... \| Horizon=...` | N/A | Snapshot parameter |
| `[param] FTS interval=... \| sensitivity=... \| valid_lock=...` | N/A | Snapshot parameter |
| `[param] ANN epochs=... \| neurons=... \| layers=... \| lr=...` | N/A | Snapshot parameter |
| `[param] ARIMA mode=... \| order=(p,d,q) \| s=...` | N/A | Snapshot parameter |
| `HOME-AVG: rows=... \| V=... \| A=... \| W=... \| E=... \| F=... \| PF=...` | F22 | Rata-rata + energi range |
| `HOME-DAILY: rows=... \| V=... \| A=... \| W=... \| E=... \| F=... \| PF=...` | F22 | Rata-rata + energi range per hari |
| `[eval] {method.upper()} MAE=... \| RMSE=... \| MAPE=...` | F17 | Metrik evaluasi |
| `[resume] Live: V=... \| I=... \| P=... \| PF=... \| F=...` | F18 | Nilai terakhir data |
| `[resume] Prediksi t+1: ...` | F19 | Forecast item pertama |
| `Dashboard Average loaded: {len(df)} rows` | N/A | Hitung jumlah baris |
| `Dashboard Daily loaded: {len(df)} rows for {target_date}` | F20 | Filter data harian dari rentang tanggal |
| `HOME synced: Average rows=..., Daily rows=...` | N/A | Status sinkronisasi |
| `Import success: {total_found} rows found, {inserted_total} inserted` | N/A | Hitung baris dari import |
| `Import completed. {status_text}` | N/A | Status |
| `Startup reset: cleared raw=..., exp=...` | N/A | Status |
| `Open SQL exported to temp file: ...` | N/A | Status |
| `Generating PDF report...` | N/A | Status |
| `Full log exported to ...` | N/A | Status |
| `Analysis pipeline finished. Rendering results...` | N/A | Status |
| `[eval] ===== Evaluasi Prediksi =====` | N/A | Header log |
| `[pipeli] ===== START/END RUN =====` | N/A | Header log |
| `Analysis failed/cancelled/...` | N/A | Status error/cancel |

### workers/calc_thread.py

| Log Event Template | Formula | Keterangan |
|---|---|---|
| `[source] Rows=... \| Range={start_dt} -> {end_dt}` | F1 | Range data raw |
| `[source] Rows=... \| Range={min_ts} -> {max_ts}` | F1 | Range data raw |
| `[prepro] Raw sample P[W]=[...]` | N/A | Sample data |
| `[prepro] Resample=... \| original=... \| resampled=... \| missing_filled=...` | F2 | Resample mean + missing fill |
| `[prepro] E_kWh = sum(P[W] * dt_hours) / 1000` | F4 | Energi dari daya |
| `[prepro] Split sizes: train=... test=...` | F3 | Split sekuensial |
| `[prepro] Train sample=...` | N/A | Sample data |
| `[prepro] Test sample=...` | N/A | Sample data |
| `[baseline] Naive MAE=... \| RMSE=... \| MAPE=...%` | F9 + F17 | Naive + metrics |
| `[baseline] MA(N) MAE=... \| RMSE=... \| MAPE=...%` | F10 + F17 | Moving Avg + metrics |
| `[fts] UoD=... \| intervals=... \| flrg_groups=...` | F11 | UoD & interval |
| `[fts] FLRG sample=...` | F12 | FLRG grouping |
| `[fts] Forecast sample=...` | F13 | Defuzzifikasi Chen |
| `[eval] FTS MAE=... \| RMSE=... \| MAPE=...%` | F17 | Metrics |
| `[ann] Config: epochs=... \| neurons=... \| layers=... \| lr=...` | N/A | Snapshot parameter |
| `[ann] Final loss=...` | F15 | Loss MSE |
| `[ann] Loss head/tail=...` | F15 | Loss history |
| `[eval] ANN MAE=... \| RMSE=... \| MAPE=...%` | F17 | Metrics |
| `[arima] AIC=... \| BIC=...` | F16 | AIC/BIC |
| `[eval] ARIMA MAE=... \| RMSE=... \| MAPE=...%` | F17 | Metrics |
| `Analysis cancelled ...` | N/A | Status |
| `Worker crashed: {e}` | N/A | Error |

### ui/export_manager.py (Resume Export)

| Log Event Template | Formula | Keterangan |
|---|---|---|
| `Export: {msg} ({val}%)` | F21 (saat msg = "Global Data") | Log progress; formula hanya berlaku pada step Global Data |
| `Resume export saved: {filename}` | N/A | Status |
| `Resume export failed: {e}` | N/A | Error |
| `PDF report exported: {filename}` | N/A | Status |
| `PDF export failed: {e}` | N/A | Error |

---

## D. Contoh Detail (Global Data + Home)

**GLOBAL -> Rata-rata tegangan (V):**  
`V_avg = (V1 + V2 + ... + Vn) / n`  
Keterangan: `V` = voltage, `n` = jumlah data V (F7).

**GLOBAL -> Terendah tegangan + waktu:**  
`V_min = min(V)`  
`t_min = t[argmin(V)]` (F8, F21).

**GLOBAL -> Energi (kWh) adjusted:**  
`E_adj = E_kwh - E_kwh_first`  
Fallback: `E_adj = cumsum(W_i * dt_i) / 1000` (F6, F4).

**HOME -> Energi range pada ringkasan:**  
`E_range = max(0, E_kwh_last - E_kwh_first)`  
Fallback: gunakan F4 jika kolom `energy_kwh` tidak ada (F5).
