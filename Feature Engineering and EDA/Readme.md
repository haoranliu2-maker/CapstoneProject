Note: In order to efficiently manage different phases of the project, I modularized my code and created separate ipynb's based on the function that I am executing. For Feature Engineering and EDA, there are 4 relevant notebooks, which are described below.

How to run the notebooks and what they do:
- [1] Capstone_HL_TargetY_v2.ipynb: generates the synthetic "Access Disruption Risk Index" (ADRI), which will serve as the target variable (y) in future analysis; exports "master_df_v2.xlsx".
- [2] Capstone_HL_featureengineering.ipynb: notebook which generates features (X) using the CMS Open Enrollment Period Public Use Files (OEP PUF) and appends; also generates a profile report for features that will be deployed for modeling later in the project; requires "master_df_v2.xlsx" as input file; exports "master_df_with_features.xlsx". 
- [3] Capstone_HL_corranalysis.ipynb: a separate notebook which loads the output from [2] and runs a Pearson's Correlation Analysis; requires "master_df_with_features.xlsx" as input file.
- [X] Baseline_Model.ipynb: runs a Logistic Regression model using the output from [2] to provide a baseline model for comparison later on; requires "master_df_with_features.xlsx" as input file.

Notebooks [1] and [2] should be run first and in order, prior to running [3] and [X].
____________________________________________________________________________________________________________


Visualizations generated from [3]:
- CorrelationHeatmap_TopADRIDrivers.png 
- Top3StatisticalDrivers.png

Visualizations generated using custom statistical/geo-mapping tools:
- Distribution of Risk Scores (RMS ADRI, log-trans risk cliff).png -- plot of ADRI distribution, including the "subsidy cliff" and "low income" risk scores that were used to generate ADRI.
- Histogram of Risk Scores (RMS ADRI, log-trans risk cliff).png -- histogram distributions where subsidy cliff risk, low income risk, and ADRI are bucketed into 4 categories (low/stable, moderate/"monitor", high/watchlist, and critical/priority)
- Norm_Log Cliff Risk Score.png -- plot of subsidy risk score (normalized, log-based) at the US county-level; states that were excluded are shaded in gray (these reflect state-managed ACA markets, not federally managed).
- Norm_LowIncome_Risk score.png -- plot of low income risk score (normalized) at the US county-level; states that were excluded are shaded in gray (these reflect state-managed ACA markets, not federally managed).

____________________________________________________________________________________________________________


Results from Baseline Model:
I selected Logistic Regression as the baseline model to establish a performance benchmark using linear relationships and the initial CMS dataset. This provides a comparative reference point for the future XGBoost model (note: I will also be running a K-Means Clustering), which will incorporate non-linear clustering interactions and external SVI data to improve predictive accuracy.

--- Baseline Results ---
              precision    recall  f1-score   support

           0       0.85      0.92      0.89       404
           1       0.69      0.52      0.59       135

    accuracy                           0.82       539
   macro avg       0.77      0.72      0.74       539
weighted avg       0.81      0.82      0.81       539

Baseline ROC-AUC: 0.8656

--- Feature Coefficients (Top Drivers) ---
feat_subsidy_coverage_ratio   -7.393034
feat_pct_aian                 -0.500975
feat_pct_black                -0.095079
feat_net_attrition            -0.013285
feat_pct_hispanic             -0.009649
feat_passive_renewal          -0.007159
avg_premium_before_APTC        0.011143
feat_pct_gold                  0.021579
feat_pct_FPL_250_400           0.023823
feat_pct_bronze                0.030790
feat_pct_asian                 0.031432
feat_market_newness            0.067956
feat_pct_age_55_over           0.087767
feat_pct_FPL_150_250           0.159933
feat_pct_FPL_100_150           0.281675
feat_pct_FPL_400_plus          0.356508

The baseline Logistic Regression model achieved a strong ROC-AUC of 0.86, confirming that the engineered CMS features (specifically income distribution and premium levels) are highly predictive of market risk. However, the model suffers from low sensitivity (Recall: 0.52), failing to identify nearly half of the high-risk counties. This suggests that the relationship between market features and risk is likely non-linear, justifying the future use of K-Means clustering and XGBoost to capture complex market behaviors and improve detection rates.
