# Log Events (Current Runtime)

Daftar ini diekstrak langsung dari string log di kode (1:1 template).

## ui/main_window.py

| Level | Message Template | Source |
|---|---|---|
| `INFO` | `[pipeli] ===== START RUN =====` | `ui/main_window.py:1289` |
| `INIT` | `Starting new analysis run with current configuration.` | `ui/main_window.py:1290` |
| `SUCCESS` | `Analysis pipeline finished. Rendering results...` | `ui/main_window.py:1372` |
| `INFO` | `[pipeli] ===== END RUN (OK) =====` | `ui/main_window.py:1436` |
| `ERROR` | `Analysis failed: {msg}` | `ui/main_window.py:1447` |
| `INFO` | `[pipeli] ===== END RUN (FAILED) =====` | `ui/main_window.py:1448` |
| `INFO` | `Analysis cancelled by user.` | `ui/main_window.py:1459` |
| `INFO` | `[pipeli] ===== END RUN (CANCELLED) =====` | `ui/main_window.py:1460` |
| `INFO` | `Cleared raw telemetry table. Deleted rows: {deleted}.` | `ui/main_window.py:1712` |
| `INFO` | `Perbarui Data (BigQuery) skipped. User is instructed to use Data panel for local JSON import.` | `ui/main_window.py:1803` |
| `INFO` | `User clicked cancel on download panel (no active background job).` | `ui/main_window.py:1814` |
| `INFO` | `Selected folder for data initiation: {folder}` | `ui/main_window.py:1853` |
| `INFO` | `Selected JSON path: {file_path}` | `ui/main_window.py:1888` |
| `INIT` | `Starting import from selected file: {selected_file}` | `ui/main_window.py:1889` |
| `SUCCESS` | `Import completed. {status_text}` | `ui/main_window.py:1925` |
| `INFO` | `Synchronize process completed: Tab Home + Tab Initial refreshed` | `ui/main_window.py:1942` |
| `SUCCESS` | `Experiment log saved with run_id={run_id}` | `ui/main_window.py:2595` |
| `SUCCESS` | `Experiment results stored to database for all methods.` | `ui/main_window.py:2607` |
| `INFO` | `Startup reset: cleared raw={deleted_map.get('raw_deleted', 0)} rows, exp={deleted_map.get('exp_deleted', 0)} rows.` | `ui/main_window.py:496` |
| `INFO` | `No data found in DB on startup.` | `ui/main_window.py:733` |
| `SUCCESS` | `Open SQL exported to temp file: {temp_path}` | `ui/main_window.py:853` |
| `INFO` | `All parameters LOCKED - Ready to start analysis` | `ui/main_window.py:1000` |
| `INFO` | `[param] Split={glob.get('split_ratio', 0) * 100:.0f}% | Horizon={glob.get('forecast_horizon', 1)}` | `ui/main_window.py:1307` |
| `INFO` | `[param] FTS interval={fts.get('interval')} | sensitivity={fts.get('sensitivity')} | valid_lock={fts.get('sensitivity_locked')}` | `ui/main_window.py:1315` |
| `INFO` | `[param] ANN epochs={ann.get('epoch')} | neurons={ann.get('neuron')} | layers={ann.get('layers')} | lr={ann.get('lr')}` | `ui/main_window.py:1324` |
| `INFO` | `[param] ARIMA mode={arima_mode} | order=({arima.get('p')},{arima.get('d')},{arima.get('q')}) | s={arima.get('s')}` | `ui/main_window.py:1335` |
| `INFO` | `[eval] ===== Evaluasi Prediksi =====` | `ui/main_window.py:1382` |
| `INFO` | `Log display limit changed to: {new_limit} lines` | `ui/main_window.py:1670` |
| `INFO` | `User requested analysis cancellation.` | `ui/main_window.py:1679` |
| `INFO` | `Cancel clicked but no active worker.` | `ui/main_window.py:1683` |
| `PROCESS` | `Generating PDF report...` | `ui/main_window.py:1734` |
| `SUCCESS` | `Full log exported to {os.path.basename(target_path)}` | `ui/main_window.py:1782` |
| `INFO` | `User cancelled folder selection for data initiation.` | `ui/main_window.py:1825` |
| `SUCCESS` | `Import success: {total_found} rows found, {inserted_total} inserted` | `ui/main_window.py:1909` |
| `INFO` | `HOME-AVG: rows={len(avg_tbl)} | V={avg_volt:.2f} | A={avg_curr:.2f} | W={avg_watt:.2f} | E={avg_energy:.4f} kWh | F={avg_freq:.2f} | PF={avg_pf:.2f}` | `ui/main_window.py:2407` |
| `INFO` | `HOME-DAILY: rows={len(daily_tbl)} | V={daily_volt:.2f} | A={daily_curr:.2f} | W={daily_watt:.2f} | E={daily_energy:.4f} kWh | F={daily_freq:.2f} | PF={daily_pf:.2f}` | `ui/main_window.py:2497` |
| `INFO` | `HOME synced: Average rows={len(avg_tbl)}, Daily rows={len(daily_tbl)}` | `ui/main_window.py:2534` |
| `INFO` | `Main window closing. Cleared runtime cache: raw={deleted_map.get('raw_deleted', 0)} rows, exp={deleted_map.get('exp_deleted', 0)} rows.` | `ui/main_window.py:2565` |
| `INFO` | `Chart plotted with {len(self.lines)} active lines` | `ui/main_window.py:192` |
| `ERROR` | `Failed startup cache reset: {e}` | `ui/main_window.py:501` |
| `ERROR` | `Failed to load DB for export: {e}` | `ui/main_window.py:839` |
| `ERROR` | `Open SQL failed: {e}` | `ui/main_window.py:855` |
| `ERROR` | `Init Home canvases failed: {e}` | `ui/main_window.py:1219` |
| `INFO` | `Average interactive chart initialized` | `ui/main_window.py:1241` |
| `INFO` | `Daily interactive chart initialized` | `ui/main_window.py:1262` |
| `ERROR` | `Failed to initialize interactive charts: {e}` | `ui/main_window.py:1272` |
| `INFO` | `[source] Rows={len(df_src)} | Range={start_ts} -> {end_ts}` | `ui/main_window.py:1296` |
| `ERROR` | `Failed to render results: {e}` | `ui/main_window.py:1379` |
| `INFO` | `[resume] Live: V={float(last['voltage']):.2f} | I={float(last['current']):.2f} | P={float(last['watt']):.2f} W | PF={float(last['pf']):.2f} | F={float(last['frequency']):.2f} Hz` | `ui/main_window.py:1402` |
| `INFO` | `[resume] Prediksi t+1: {' | '.join(forecast_line)}` | `ui/main_window.py:1423` |
| `ERROR` | `Failed to persist experiment to DB: {e}` | `ui/main_window.py:1434` |
| `ERROR` | `Failed to export results: {e}` | `ui/main_window.py:1759` |
| `ERROR` | `Export log failed: {e}` | `ui/main_window.py:1785` |
| `INFO` | `Found {len(json_files)} JSON files in folder` | `ui/main_window.py:1843` |
| `WARNING` | `No JSON files found in selected folder` | `ui/main_window.py:1849` |
| `ERROR` | `Import failed: {res.get('message')}` | `ui/main_window.py:1912` |
| `ERROR` | `Failed to draw home graph: {e}` | `ui/main_window.py:2093` |
| `INFO` | `No data available for dashboard Average` | `ui/main_window.py:2127` |
| `SUCCESS` | `Dashboard Average loaded: {len(df)} rows` | `ui/main_window.py:2176` |
| `WARNING` | `DataFrame empty for table view` | `ui/main_window.py:2178` |
| `ERROR` | `Failed to load dashboard Average: {e}` | `ui/main_window.py:2185` |
| `WARNING` | `No data for date: {target_date}` | `ui/main_window.py:2235` |
| `SUCCESS` | `Dashboard Daily loaded: {len(df)} rows for {target_date}` | `ui/main_window.py:2277` |
| `WARNING` | `No data for date: {target_date}` | `ui/main_window.py:2279` |
| `ERROR` | `Failed to load dashboard Daily: {e}` | `ui/main_window.py:2286` |
| `ERROR` | `Failed to clear runtime cache on exit: {e}` | `ui/main_window.py:2567` |
| `ERROR` | `Failed to plot chart: {e}` | `ui/main_window.py:196` |
| `ERROR` | `Failed to plot result chart: {e}` | `ui/main_window.py:391` |
| `INFO` | `Please LOCK all parameter tabs before starting` | `ui/main_window.py:1004` |
| `RESULT` | `[eval] {method.upper()} MAE={m['mae']:.6f} | RMSE={m['rmse']:.6f} | MAPE={m['mape']:.4f}%` | `ui/main_window.py:1386` |
| `WARNING` | `No data available for daily dashboard` | `ui/main_window.py:2225` |
| `WARNING` | `Please initiate data first (Tab Main)` | `ui/main_window.py:1006` |

## workers/calc_thread.py

| Level | Message Template | Source |
|---|---|---|
| `INFO` | `Stop requested for analysis worker.` | `workers/calc_thread.py:357` |
| `INFO` | `[prepro] Split sizes: train={len(train_series)} test={len(test_series)}` | `workers/calc_thread.py:92` |
| `DEBUG` | `[prepro] Train sample={train_series.head(10).round(4).tolist()}` | `workers/calc_thread.py:96` |
| `DEBUG` | `[prepro] Test sample={test_series.head(10).round(4).tolist()}` | `workers/calc_thread.py:100` |
| `INFO` | `[fts] UoD={uod} | intervals={len(intervals)} | flrg_groups={len(flrg)}` | `workers/calc_thread.py:165` |
| `DEBUG` | `[fts] Forecast sample={fts_out.get('forecast', [])[:5]}` | `workers/calc_thread.py:172` |
| `RESULT` | `[eval] FTS MAE={fts_metrics['mae']:.6f} | RMSE={fts_metrics['rmse']:.6f} | MAPE={fts_metrics['mape']:.4f}%` | `workers/calc_thread.py:176` |
| `FORMU` | `[eval] MAE = mean(|y - yhat|) = {fts_metrics['mae']:.6f}` | `workers/calc_thread.py:184` |
| `FORMU` | `[eval] RMSE = sqrt(mean((y - yhat)^2)) = {fts_metrics['rmse']:.6f}` | `workers/calc_thread.py:188` |
| `FORMU` | `[eval] MAPE = mean(|(y - yhat)/y|)*100 = {fts_metrics['mape']:.4f}%` | `workers/calc_thread.py:192` |
| `INFO` | `[ann] Config: epochs={ann_params.get('epoch')} | neurons={ann_params.get('neuron')} | layers={ann_params.get('layers')} | lr={ann_params.get('lr')}` | `workers/calc_thread.py:217` |
| `RESULT` | `[eval] ANN MAE={ann_metrics['mae']:.6f} | RMSE={ann_metrics['rmse']:.6f} | MAPE={ann_metrics['mape']:.4f}%` | `workers/calc_thread.py:254` |
| `FORMU` | `[eval] MAE = mean(|y - yhat|) = {ann_metrics['mae']:.6f}` | `workers/calc_thread.py:262` |
| `FORMU` | `[eval] RMSE = sqrt(mean((y - yhat)^2)) = {ann_metrics['rmse']:.6f}` | `workers/calc_thread.py:266` |
| `FORMU` | `[eval] MAPE = mean(|(y - yhat)/y|)*100 = {ann_metrics['mape']:.4f}%` | `workers/calc_thread.py:270` |
| `RESULT` | `[eval] ARIMA MAE={arima_metrics['mae']:.6f} | RMSE={arima_metrics['rmse']:.6f} | MAPE={arima_metrics['mape']:.4f}%` | `workers/calc_thread.py:313` |
| `FORMU` | `[eval] MAE = mean(|y - yhat|) = {arima_metrics['mae']:.6f}` | `workers/calc_thread.py:321` |
| `FORMU` | `[eval] RMSE = sqrt(mean((y - yhat)^2)) = {arima_metrics['rmse']:.6f}` | `workers/calc_thread.py:325` |
| `FORMU` | `[eval] MAPE = mean(|(y - yhat)/y|)*100 = {arima_metrics['mape']:.4f}%` | `workers/calc_thread.py:329` |
| `DEBUG` | `[prepro] Raw sample P[W]={raw_sample}` | `workers/calc_thread.py:63` |
| `INFO` | `[prepro] Resample={artifacts.get('interval_used')} | original={artifacts.get('original_count')} | resampled={artifacts.get('resampled_count')} | missing_filled={artifacts.get('missing_filled')}` | `workers/calc_thread.py:69` |
| `FORMU` | `[prepro] E_kWh = sum(P[W] * dt_hours) / 1000` | `workers/calc_thread.py:79` |
| `INFO` | `Analysis cancelled by user during preprocessing.` | `workers/calc_thread.py:113` |
| `RESULT` | `[baseline] Naive MAE={naive_metrics['mae']:.6f} | RMSE={naive_metrics['rmse']:.6f} | MAPE={naive_metrics['mape']:.4f}%` | `workers/calc_thread.py:126` |
| `RESULT` | `[baseline] MA({ma_window}) MAE={ma_metrics['mae']:.6f} | RMSE={ma_metrics['rmse']:.6f} | MAPE={ma_metrics['mape']:.4f}%` | `workers/calc_thread.py:134` |
| `INFO` | `Analysis cancelled by user before FTS stage.` | `workers/calc_thread.py:147` |
| `DEBUG` | `[fts] FLRG sample={flrg_sample}` | `workers/calc_thread.py:171` |
| `INFO` | `Analysis cancelled by user after FTS stage.` | `workers/calc_thread.py:204` |
| `INFO` | `Analysis cancelled by user before ANN stage.` | `workers/calc_thread.py:210` |
| `INFO` | `[ann] Final loss={ann_art.get('final_loss')}` | `workers/calc_thread.py:244` |
| `DEBUG` | `[ann] Loss head/tail={loss_hist[:3]} ... {loss_hist[-3:]}` | `workers/calc_thread.py:250` |
| `INFO` | `Analysis cancelled by user after ANN stage.` | `workers/calc_thread.py:282` |
| `INFO` | `Analysis cancelled by user before ARIMA stage.` | `workers/calc_thread.py:288` |
| `ERROR` | `[arima] {arima_art.get('error')}` | `workers/calc_thread.py:307` |
| `INFO` | `[arima] AIC={arima_art.get('aic')} | BIC={arima_art.get('bic')}` | `workers/calc_thread.py:309` |
| `INFO` | `Analysis cancelled by user after ARIMA stage.` | `workers/calc_thread.py:341` |
| `ERROR` | `Worker crashed: {e}` | `workers/calc_thread.py:350` |
| `INFO` | `[source] Rows={raw_rows} | Range={start_dt} -> {end_dt}` | `workers/calc_thread.py:52` |
| `ERROR` | `[baseline] Failed: {e}` | `workers/calc_thread.py:143` |
| `INFO` | `[source] Rows={raw_rows} | Range={min_ts} -> {max_ts}` | `workers/calc_thread.py:57` |

## ui/export_manager.py

| Level | Message Template | Source |
|---|---|---|
| `SUCCESS` | `PDF report exported: {os.path.basename(file_path)}` | `ui/export_manager.py:378` |
| `SUCCESS` | `Resume export saved: {os.path.basename(file_path)}` | `ui/export_manager.py:859` |
| `ERROR` | `PDF export failed: {e}` | `ui/export_manager.py:384` |
| `INFO` | `Export: {msg} ({val}%)` | `ui/export_manager.py:405` |
| `ERROR` | `Resume export failed: {e}` | `ui/export_manager.py:863` |
