# Business Case Analysis — Promotion Effectiveness

## B1. Problem Formulation

### (a) Machine Learning Formulation

The objective is to predict the effectiveness of different promotions across stores.

* **Target Variable:**
  `items_sold` (number of items sold per store per month)

* **Input Features:**

  * Store attributes: `store_size`, `location_type`, `competition_density`
  * Temporal features: `month`, `year`, `is_weekend`, `is_festival`
  * Promotion details: `promotion_type`
  * Historical performance: past sales (if available)

* **Type of ML Problem:**
  This is a **supervised regression problem** because:

  * The target (`items_sold`) is a continuous numerical value
  * We aim to predict quantity (not category)

Additionally, this can be extended into a **prescriptive problem**, where predictions are used to choose the optimal promotion.



### (b) Why Items Sold > Revenue

Using **items_sold** is more reliable than total revenue because:

* Revenue is influenced by **price variations**, discounts, and product mix
* Promotions like discounts artificially reduce revenue even if demand increases
* Items sold directly reflects **customer demand and engagement**

**Broader Principle:**
The target variable must **align with the business objective**. In this case, the goal is to maximize volume and customer response, not just monetary value.



### (c) Alternative Modelling Strategy

Instead of a single global model:

* Use a **hierarchical (multi-level) model** or **store-cluster-based models**
* Example strategies:

  * Cluster stores (urban vs rural) and train separate models
  * Add **store-level interaction features** (store × promotion)
  * Use **mixed-effects models** or **per-store fine-tuned models**

**Justification:**
Different stores respond differently due to demographics, competition, and footfall. A single model may average out these effects and reduce accuracy.



## B2. Data and EDA Strategy

### (a) Data Joining and Grain

**Tables:**

* Transactions
* Store attributes
* Promotion details
* Calendar

**Joins:**

* Join transactions with store attributes on `store_id`
* Join promotions using `promotion_id` or `promotion_type`
* Join calendar on `transaction_date`

**Final Dataset Grain:**
One row = **store × month**

**Aggregations:**

* Total `items_sold` per store per month
* Average `competition_density`
* Count of promotion days
* Flags for weekends/festivals



### (b) EDA Strategy

1. **Target Distribution Plot**

   * Check skewness of `items_sold`
   * Helps decide transformations (e.g., log scaling)

2. **Promotion-wise Sales Comparison**

   * Boxplots of `items_sold` by `promotion_type`
   * Identifies which promotions perform best

3. **Time Series Trends**

   * Monthly sales trends
   * Detect seasonality (festivals, holidays)

4. **Correlation Heatmap**

   * Identify relationships between features
   * Helps in feature selection and avoiding multicollinearity

5. **Store-Level Variation Analysis**

   * Compare performance across stores
   * Detect heterogeneity → supports segmented modelling



### (c) Handling Promotion Imbalance

Problem: 80% of data has **no promotion**

**Impact:**

* Model may become biased toward "no promotion"
* Poor learning of promotion effects

**Solutions:**

* Oversample promotion data
* Use **sample weighting**
* Create a binary feature: `is_promotion`
* Stratified analysis by promotion vs non-promotion
* Consider uplift modelling (treatment vs control)



## B3. Model Evaluation and Deployment

### (a) Train-Test Strategy & Metrics

**Train-Test Split:**

* Use **time-based split**
* Train on first ~2.5 years, test on last ~6 months

**Why Not Random Split:**

* Causes **data leakage**
* Future data may influence past predictions
* Violates real-world deployment conditions

**Evaluation Metrics:**

* **RMSE (Root Mean Squared Error):**

  * Penalizes large errors heavily
  * Useful when big mistakes are costly

* **MAE (Mean Absolute Error):**

  * Easy to interpret (average error in units sold)

* **R² Score (optional):**

  * Measures explained variance

**Interpretation:**

* Lower RMSE/MAE → better prediction accuracy
* Compare models based on consistency across stores



### (b) Explaining Model Recommendations

To explain why different promotions are recommended:

* Use **feature importance (Random Forest / SHAP values)**
* Analyze:

  * Month/seasonality effects (e.g., December → festive demand)
  * Store characteristics
  * Interaction between promotion and time

**Example Explanation:**

* December → high demand → Loyalty Points works better (retention)
* March → lower demand → Flat Discount boosts purchases

**Communication Approach:**

* Use simple visuals (feature importance charts)
* Translate technical insights into business terms



### (c) Deployment Strategy

**1. Model Saving**

* Save trained pipeline using `joblib` or `pickle`

**2. Data Pipeline**

* Each month:

  * Collect new store + calendar + promotion data
  * Apply same preprocessing pipeline
  * Generate predictions

**3. Recommendation System**

* Simulate all 5 promotions per store
* Choose promotion with highest predicted `items_sold`

**4. Automation**

* Schedule monthly batch job (e.g., Airflow, cron)

**5. Monitoring**

* Track:

  * Prediction vs actual sales
  * RMSE/MAE over time
  * Data drift (feature distribution changes)

**6. Retraining**

* Trigger retraining when:

  * Performance degrades beyond threshold
  * Significant seasonal or market changes occur



## Conclusion

This solution combines **predictive modelling with business understanding** to optimize promotion strategies. By focusing on volume-based targets, accounting for store-level differences, and deploying a robust pipeline, the company can systematically improve promotion effectiveness across all stores.
