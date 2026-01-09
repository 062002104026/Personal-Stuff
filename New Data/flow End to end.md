PPDL_FLOW_SPEC:
  HOME:
    General_Calculation:
      Input_Data:
        fields: [ts, V, A, W, E, PF, F]
      Period_Definition:
        period:
          start: "first_date(ts)"
          end: "last_date(ts)"
      Qty_Data:
        value: "count_rows(period)"

      Average:
        scope: "HOME.AVERAGE_TAB"
        binds_to_period: "global_period(first_date -> last_date)"
        outputs:
          Avg.Period.A: { formula: "sum(A_valid)/count(A_valid)" }
          Avg.Period.V: { formula: "sum(V_valid)/count(V_valid)" }
          Avg.Period.W: { formula: "sum(W_valid)/count(W_valid)" }
          Avg.Period.E: { formula: "sum(E_valid)/count(E_valid)" }
          Avg.Period.PF: { formula: "sum(PF_valid)/count(PF_valid)" }
          Avg.Period.F: { formula: "sum(F_valid)/count(F_valid)" }

      Daily:
        scope: "HOME.DAILY_TAB"
        binds_to_period: "selected_date_only"
        df_daily: "filter(df_clean, ts in selected_date_range)"
        outputs:
          Avg.Daily.A: { formula: "sum(A_valid_in_df_daily)/count(A_valid_in_df_daily)" }
          Avg.Daily.V: { formula: "sum(V_valid_in_df_daily)/count(V_valid_in_df_daily)" }
          Avg.Daily.W: { formula: "sum(W_valid_in_df_daily)/count(W_valid_in_df_daily)" }
          Avg.Daily.E: { formula: "sum(E_valid_in_df_daily)/count(E_valid_in_df_daily)" }
          Avg.Daily.PF: { formula: "sum(PF_valid_in_df_daily)/count(PF_valid_in_df_daily)" }
          Avg.Daily.F: { formula: "sum(F_valid_in_df_daily)/count(F_valid_in_df_daily)" }

      Energy_Total_Period:
        scope: "HOME.AVERAGE_TAB"
        policy:
          if_E_first_is_zero:
            if: "E_first == 0"
            energy_total_period: "E_last"
          else_use_delta:
            if: "E_first > 0"
            energy_total_period: "E_last - E_first"

  INITIAL:
    Advance_Parameter_Lock:
      General:
        Parameter:
          fields:
            target_variable: "target_var_id"
            train_test_split: "split_pct"
            forecasting: "horizon"
        Metrics:
          enabled: [MAE, RMSE, MAPE]
      Methods:
        FTS:
          parameters: [intervals, sensitivity, valid_lock, uod_padding_policy]
        ANN:
          parameters: [epochs, neurons, layers, lr]
        ARIMA:
          parameters: [mode, order_p_d_q, seasonal_period_s]

  ANALYSIS:
    Global_Precondition:
      input_dataset:
        source: "HOME selected dataset scope"
        allowed_sources:
          - "HOME.AVERAGE_TAB (global period)"
          - "HOME.DAILY_TAB (selected date window)"
      required_states:
        - "df_clean exists"
        - "advance_params_locked exists"
        - "target_variable selected"
        - "metrics selected"
      output_artifacts:
        - "analysis_run_guid"
        - "dataset_scope_signature"
        - "params_lock_signature"

    Modeling_Dataset_Build:
      Target_Series:
        y_raw: "extract(df_scope[target_variable])"
        valid_mask: "isfinite(y_raw)"
        y_valid: "y_raw[valid_mask]"
      Time_Normalization:
        sort: "by ts asc"
        dt_profile:
          detect: ["dt_mode", "dt_median", "gap_count", "duplicate_ts_count"]
      Resample_and_Missing_Policy:
        enabled: "system_policy_resample (on/off)"
        if_enabled:
          freq: "system_freq (e.g., 5T)"
          agg_rule: "system_agg_rule (mean/sum)"
          missing_policy: "system_missing_policy (ffill/interpolate/drop)"
        output: "y_final"
      Train_Test_Split:
        split_pct: "INITIAL.General.Parameter.train_test_split"
        n_total: "len(y_final)"
        n_train: "floor(n_total * split_pct/100)"
        y_train: "y_final[:n_train]"
        y_test: "y_final[n_train:]"
      Horizon_Alignment_Contract:
        horizon: "INITIAL.General.Parameter.forecasting"
        y_true_eval: "y_test[horizon:]"
        rule: "y_hat_eval must align 1:1 with y_true_eval (same length after finite-filter)"

    BASELINE:
      Naive_Method:
        definition: "baseline predictor used for reference (optional)"
        build:
          y_hat_naive_full: "shift(y_test, +0 or +1 per system definition)"
        metrics:
          MAE: "mean(|y_true_eval - y_hat_naive_eval|)"
          RMSE: "sqrt(mean((y_true_eval - y_hat_naive_eval)^2))"
          MAPE:
            policy: "system_policy (skip_zero or epsilon)"
            formula: "mean(|(y_true_eval - y_hat)/den|) * 100"

    FTS:
      Build_UoD:
        source: "y_train"
        min_train: "min(y_train)"
        max_train: "max(y_train)"
        padding_policy: "INITIAL.Methods.FTS.uod_padding_policy"
        output: ["uod_min", "uod_max"]
      Partition_Intervals:
        K: "INITIAL.Methods.FTS.intervals"
        width: "(uod_max - uod_min) / K"
        intervals: "A1..AK (low/high boundaries)"
        midpoints: "mid(Ai) = (low_i + high_i)/2"
      Fuzzification_Train:
        mapping_rule: "assign y_train[t] -> Ai where Ai contains value"
        output: "L_train (sequence of fuzzy labels)"
      FLR_Build:
        definition: "L_t -> L_{t+1}"
        output: "FLR list"
      FLRG_Build_and_Support:
        group_rule: "group RHS per LHS"
        support_rule: "support(LHS->Aj) = count(LHS->Aj)/total(LHS->*)"
        output: "FLRG + support matrix"
      Forecast_Test:
        horizon: "INITIAL.General.Parameter.forecasting"
        for_each_eval_step:
          current_state_rule: "fuzzify current value -> L_t"
          if_flrg_exists:
            y_hat: "sum_j( support(L_t->A_j) * midpoint(A_j) )"
          else_fallback:
            y_hat: "midpoint(L_t)"
        output: "y_hat_fts_eval (aligned to y_true_eval)"
      Metrics:
        Pair_Filter:
          finite_mask: "isfinite(y_true_eval) & isfinite(y_hat_fts_eval)"
          y_true: "y_true_eval[finite_mask]"
          y_hat: "y_hat_fts_eval[finite_mask]"
        MAE: "mean(|y_true - y_hat|)"
        RMSE: "sqrt(mean((y_true - y_hat)^2))"
        MAPE:
          policy: "system_policy (skip_zero or epsilon)"
          formula: "mean(|(y_true - y_hat)/den|)*100"

    ANN:
      Dataset_Build:
        source: "y_train, y_test"
        windowing_policy: "system_policy (enabled/disabled)"
        if_windowing_enabled:
          window_len: "system_window_len"
          X_train: "sliding_window(y_train, window_len)"
          Y_train: "next_step_targets(y_train, window_len)"
          X_test: "windows built with tail(train)+test context"
        else_direct:
          X_train: "reshape(y_train)"
          Y_train: "y_train"
          X_test: "reshape(y_test)"
      Normalize:
        policy: "system_policy (on/off)"
        if_enabled:
          scaler_fit: "fit on TRAIN only"
          transform: ["X_train", "X_test", "Y_train(optional)"]
      Train:
        config:
          epochs: "INITIAL.Methods.ANN.epochs"
          neurons: "INITIAL.Methods.ANN.neurons"
          layers: "INITIAL.Methods.ANN.layers"
          lr: "INITIAL.Methods.ANN.lr"
        output: ["model", "train_loss_trace"]
      Predict:
        y_hat_ann_full: "model.predict(X_test)"
        inverse_scale: "if normalization enabled"
        align:
          horizon: "INITIAL.General.Parameter.forecasting"
          y_hat_ann_eval: "aligned to y_true_eval"
      Metrics:
        Pair_Filter: "finite mask as in FTS"
        MAE: "mean(|y_true - y_hat|)"
        RMSE: "sqrt(mean((y_true - y_hat)^2))"
        MAPE:
          policy: "system_policy (skip_zero or epsilon)"
          formula: "mean(|(y_true - y_hat)/den|)*100"

    ARIMA:
      Fit:
        source: "y_train"
        mode: "INITIAL.Methods.ARIMA.mode"
        order: "INITIAL.Methods.ARIMA.order_p_d_q"
        seasonal_period_s: "INITIAL.Methods.ARIMA.seasonal_period_s"
        output: "arima_model"
      Forecast:
        steps: "len(y_test) - horizon"
        y_hat_arima_eval: "forecast(steps) aligned to y_true_eval"
      Metrics:
        Pair_Filter: "finite mask as in FTS"
        MAE: "mean(|y_true - y_hat|)"
        RMSE: "sqrt(mean((y_true - y_hat)^2))"
        MAPE:
          policy: "system_policy (skip_zero or epsilon)"
          formula: "mean(|(y_true - y_hat)/den|)*100"

    RESUME:
      Aggregate_Results:
      PURPOSE: "Menyajikan hasil akhir evaluasi prediksi secara terstruktur, terpisah dari proses kalkulasi"
      SCOPE:
        source_data: "Output ANALYSIS (FTS, ANN, ARIMA)"
        locked_params: "Mengikuti parameter valid dari TAB INITIAL"
        dataset_scope: "Mengikuti periode & seleksi dari TAB HOME (Average/Daily)"
    
      SUB_TABS:
        MAE:
          description: "Mean Absolute Error untuk seluruh metode"
          units: "Same unit as target variable (ex: W)"
          structure:
            FTS:
              datasets:
                - TRAIN:
                    source: "y_train vs yhat_train_fts"
                    calc: "MAE = mean(|y - yhat|)"
                - TEST:
                    source: "y_test vs yhat_test_fts"
                    calc: "MAE = mean(|y - yhat|)"
                - BASELINE:
                    source: "y_test vs yhat_baseline"
                    calc: "MAE = mean(|y - yhat|)"
                - MOVING_AVERAGE:
                    source: "y_test vs yhat_ma"
                    calc: "MAE = mean(|y - yhat|)"
            ANN:
              datasets:
                - TRAIN:
                    source: "y_train vs yhat_train_ann"
                    calc: "MAE = mean(|y - yhat|)"
                - TEST:
                    source: "y_test vs yhat_test_ann"
                    calc: "MAE = mean(|y - yhat|)"
            ARIMA:
              datasets:
                - TRAIN:
                    source: "y_train vs yhat_train_arima"
                    calc: "MAE = mean(|y - yhat|)"
                - TEST:
                    source: "y_test vs yhat_test_arima"
                    calc: "MAE = mean(|y - yhat|)"
    
        MAPE:
          description: "Mean Absolute Percentage Error"
          units: "Percent (%)"
          rule:
            zero_handling: "skip y==0 (isfinite filter)"
          structure:
            FTS:
              datasets:
                - TRAIN
                - TEST
                - BASELINE
                - MOVING_AVERAGE
            ANN:
              datasets:
                - TRAIN
                - TEST
            ARIMA:
              datasets:
                - TRAIN
                - TEST
    
        RMSE:
          description: "Root Mean Squared Error"
          units: "Same unit as target variable (ex: W)"
          structure:
            FTS:
              datasets:
                - TRAIN
                - TEST
                - BASELINE
                - MOVING_AVERAGE
            ANN:
              datasets:
                - TRAIN
                - TEST
            ARIMA:
              datasets:
                - TRAIN
                - TEST
    
        GRAPHIC:
          description: "Visual comparison Actual vs Prediction"
          plots:
            FTS:
              - TRAIN_SERIES:
                  x: "time_train"
                  y:
                    - actual_train
                    - pred_train_fts
              - TEST_SERIES:
                  x: "time_test"
                  y:
                    - actual_test
                    - pred_test_fts
            ANN:
              - TRAIN_SERIES
              - TEST_SERIES
            ARIMA:
              - TRAIN_SERIES
              - TEST_SERIES
          baseline_overlay:
            enabled: true
            source: "baseline & moving average (FTS only)"
    
        EXPORT:
          description: "Ekspor hasil analisis & resume"
          formats:
            - CSV
            - XLSX
            - PNG
            - ZIP
          export_content:
            CSV:
              includes:
                - actual_series
                - pred_fts
                - pred_ann
                - pred_arima
                - error_mae
                - error_mape
                - error_rmse
            XLSX:
              sheets:
                - SUMMARY_METRICS
                - FTS_DETAIL
                - ANN_DETAIL
                - ARIMA_DETAIL
            PNG:
              includes:
                - all_graphics_from_GRAPHIC_tab
            ZIP:
              structure:
                - summary_metrics.csv
                - fts_detail.csv
                - ann_detail.csv
                - arima_detail.csv
                - graphics/
          naming_rule:
            pattern: "PPDL_RESUME_[METHOD]_[GUID_SHORT]"