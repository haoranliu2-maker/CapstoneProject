Project: Predicting Patient Access Disruption Risk via Machine Learning

**1. Problem Statement**

The expiration of enhanced Affordable Care Act (ACA) Premium Tax Credits (ePTCs) creates a significant "policy cliff," threatening the insurance stability of millions of Americans. This Capstone project aims to develop a machine learning solution to identify and map U.S. communities at the highest risk of patient access disruption due to this financial shock. By identifying these high-risk counties at a granular level, biotech companies like Genentech can proactively deploy patient assistance, health equity resources, and medical education to ensure continuity of care for vulnerable populations.

**2. Model Outcomes and Prediction**

This project utilizes a hybrid machine learning approach combining both unsupervised and supervised learning:

**Unsupervised Learning (Clustering):**  K-means clustering groups ~2,100 counties into distinct "Community Profiles" based on structural and behavioral features.

**Supervised Learning (Classification):**  An XGBoost model classifies counties into "High Risk" vs. "Stable" categories based on the Access Disruption Risk Index (ADRI).

**Output:**  The model provides a binary classification (High Risk = 1) and Feature Importance Scores, which explain the primary drivers (economic, behavioral, or social) behind a community's risk profile.

**3. Data Acquisition**

For this project, publicly available population health data was integrated from three primary sources to create a multi-dimensional view of risk:

**CMS Open Enrollment Period PUF Data:**  County-level enrollment, metal tier selections, and average Affordable Care Act premiums.

**KFF (Kaiser Family Foundation) Research:**  Used to project the specific dollar-value increase in premiums for the benchmark silver plan within each county.

**CDC Social Vulnerability Index (SVI):**  Incorporated to capture non-financial barriers, including disability status, language barriers, housing burdens, and poverty levels.

**4. Data Preprocessing/Preparation**

**A. Data Cleaning and Integrity**

**Handling Missing Values:**  CMS OEP PUF data often contains "null" or "suppressed" values for small-population counties to protect privacy. Median imputation was used for skewed financial variables (like premiums) and zero-filling for specific demographic counts where a null strongly implied a lack of that specific population segment.

**Outlier Mitigation:**  Preliminary analysis revealed extreme premium values in a small number of rural counties. A log-transform to the "Subsidy Cliff" dollar-value increases was used to compress the range and prevent these outliers from exerting undue influence on the XGBoost gradients.

**Consistency Checks:**  FIPS codes (Federal Information Processing Series) was used to cross-reference and stitch data across the CMS, KFF, and CDC SVI datasets to ensure accurate spatial join at the county level.

**B. Feature Engineering & Target Construction**

**Target Variable (ADRI) Calculation:**  The Access Disruption Risk Index (ADRI) was developed as a synthetic label. This was calculated as the RMS of two sub-indices: the "Subsidy Cliff Risk" and the "Low-Income Risk". Using RMS was a deliberate analytical choice to ensure that a high score in either risk vector would elevate the county's total risk profile.

**Composite Feature Creation:**  A new feature, plan_value_density, was developed and calculated as the ratio of Gold enrollees to the sum of Gold and Bronze enrollees. This provides the model with a single "risk appetite" metric for each county.

**C. Encoding and Analysis Steps**

**Categorical Encoding:**  While most variables were numeric, the K-Means Cluster Assignments (0, 1, 2) were treated as categorical identifiers. To prevent the XGBoost model from assuming a mathematical order between clusters (i.e., that Cluster 2 is "greater than" Cluster 1), one-hot encoding was applied to the cluster labels. This allowed the model to treat each community archetype as a distinct binary feature.

**Feature Scaling:**  For the K-means clustering phase, all features were processed using StandardScaler to reach a mean of 0 and a standard deviation of 1. This was critical because variables like “avg_premium_before_APTC” (ranging in the hundreds) would otherwise mathematically overwhelm percentage-based features (ranging from 0 to 1).

**Collinearity Analysis:**  A Correlation Matrix analysis was used to identify redundant features. For example, feat_pct_white was excluded from the initial clustering to avoid the "dummy variable trap" since the racial percentages sum to 100%, opting to use it only in the post-modeling evaluation phase.

**D. Data Splitting**

**Train/Test Strategy:**  The final dataset of ~2,100 counties was split using an 80/20 stratified split. Stratification was performed based on the ADRI risk tiers to ensure that the test set contained a representative proportion of the "Critical Risk" (Minority Class) counties.

**Imbalance Handling:**  Due to the relatively small number of "High Risk" counties (Class 1), the scale_pos_weight parameter in XGBoost was used during the training phase to penalize misclassifications of the minority risk class more heavily.

**5. Modeling**

**K-Means Clustering:**  Used to segment the U.S. into actionable community archetypes rather than treating all "at-risk" counties as identical.

K-Means Analysis ipynb: _See "[4] Capstone_HL_kmeans_clustering.ipynb"_

**XGBoost Classifier:**  Chosen for its superior performance on tabular data and its ability to provide interpretability through feature importance, which is essential for business storytelling.

Link to XGBoost Model ipynb: _See "[6] Capstone_HL_XGBoost (threshold).ipynb"_

**6. Model Evaluation and Important Findings**

**A. Classification Model Performance**

The final tuned XGBoost model achieved a test ROC-AUC of 0.8675, indicating excellent ability to distinguish between stable and high-risk counties.

**Recall (0.65):**  The model captures roughly two-thirds of all truly high-risk counties. In the context of a health policy study focusing on the massive financial shock of expiring ePTCs, catching 65% of the risk is a solid baseline for a proactive mitigation strategy. 

**Precision (0.64):**  Flagged counties have a 64% probability of being true disruption sites, ensuring efficient resource allocation.

**Feature Importance Visualization:**  A detailed breakdown of the variables driving these predictions is available (_see "XGBoost_FeatureImportance (Gain).png"_). This plot highlights the top features that hold the most weight in the model's decision-making process.

**B. Findings: The Three Community Profiles**

The unsupervised K-Means clustering analysis identified three distinct risk archetypes, which can be visualized in high-dimensional space (_see "KMeansClusterPCA_Graph.png"_). 

For a granular breakdown of the specific metrics for each group, see _"KmeansClusterProfiles.txt"_.

**Cluster 0:**  The "Passive Core" (high volume, high vulnerability): This was the largest group by county count and was characterized by high passive renewal (56.7%). These patients are at risk of "silent churn"—dropping coverage because they didn't realize their costs changed until receiving the first bill.

**Cluster 1:**  The "Cliff-Edge Seniors" (high margin, high cost): This cluster is represented by a high concentration of age 55+ (31%) and 400%+ federal poverty line (FPL) (13%) enrollees. These are the primary victims of the "Subsidy Cliff" and face the highest gross premium increases.

**Cluster 2:**  The "High-Value Strivers" (urban): The final cluster only contained 271 counties but consisted of 40% of all enrollees, high Hispanic population (19%) and high Gold-plan adoption. This cluster was also the #1 predictor of high risk in the XGBoost model.

**7. Strategic Suggestions & Next Steps**

**Recommendations for Genentech**

**Targeted Metro Strategy:**  Focus Key Account Managers on the large health systems within Cluster 2 counties, as these hubs hold the highest volume of at-risk patients.
**Proactive Enrollment Nudges:**  Launch digital engagement campaigns in Cluster 0 counties to combat passive renewal churn before the plan year ends.

**Health Equity Prioritization:**  Use SVI drivers like Disability (EP_DISABL) and AIAN Population % to justify the deployment of bilingual patient educators and specialized access support in vulnerable zones.

**Field Sales Preparedness:**  Alert field teams in Cluster 1 to anticipate "plan buy-downs" (Seniors moving from Gold to Bronze plans), which can trigger high deductibles and sudden prescription abandonment.

**Technical Next Steps**

**Longitudinal Tracking:**  A great next step would be to incorporate actual 2025/2026 enrollment churn data as it becomes available to validate the model's predictions.
**Internal Data Integration:**  Additionally, there’s an opportunity to cross-reference high-risk counties with internal Genentech sales data to identify specific products and therapeutic areas (e.g. oncology vs. ophthalmology) that most exposed to these enrollment risks/shifts.

