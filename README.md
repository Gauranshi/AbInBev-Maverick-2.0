# AbInBev-Maverick-2.0 - Team Blitzkreig

## Overview:
An intelligent and efficient model to recommend customized discounts for customers based on their business significance and performance. The data captures the product sold at multiple retail outlets.
</br>
Discounts, promotions and pricing strategies drive a major portion of the revenue to the Companies. Hence, it becomes very important for companies to provide discounts by looking at the ROI of the complete discount profile of the customer. Running a promotion or giving a discount can, when done properly, deliver incremental sale or help gaining access to new customers. 
## Link to Web-App: 
https://share.streamlit.io/arusarkabose/discount-predicter/main/app.py

## Pipeline:
![FLOW_DIAG](https://user-images.githubusercontent.com/45457551/117551576-73be4680-b064-11eb-8885-41955a4a8f92.PNG)

The Dataset can be found [here](https://github.com/Gauranshi/AbInBev-Maverick-2.0/blob/main/data.xlsx)

## Methodology
### Pre-Processing:
* Replace missing data with mode of Attribute
* If numerical values are replaced with 0 if less than a threshold(0.0005), to prevent Numerical Overflow
* Drop non-significant attributes(Ship-to ID, Product Set, Discount_Total)
* One-hot encode Nominal categorical columns using Pandas
* Label encode Ordinal categorical columns using Sk-learn

### Outlier removal
* Remove data points with Z-score above a threshold. Formula for Z-score is given as:
<img src="https://user-images.githubusercontent.com/45457551/117552486-3b6d3700-b069-11eb-9741-a29cb2e22f6d.PNG" width="280" height="70" />

Threshold is chosen as 3 as it indicates a spread of 3 Standard Deviation which covers 99.7% of the gaussian distribution.

* Remove data points with Negative values of discount, volume or GTO
### Feature Engineering
1. **Volume Difference**: Discount multiplier based on the sales volume difference between the financial years 2019 and 2018. 

* For reduced volumes, a multiplier factor of 1.24 is provided to incentivise higher sales.
* For stagnant volumes, a multiplier factor of 1.12 is provided to incentivise higher sales.
* For increased volumes, the multiplier factor is 0.95 in order to increase the sales profits.

2. **Tiering Structure**: Modified tiering structure clubbing the lower tier premiums with the higher tier mainstreams, in accordance with similarity trends between the same:

                      Tier 2 Mainstream -> Tier 3  
                      Tier 1 Mainstream + Tier 2 Premium -> Tier 2 
                      Tier 0 Mainstream + Tier 1 Premium -> Tier 1 
                      Tier 0 Premium -> Tier 0

3. **Discount Multiplier**: Discount multipier factor based on the average sales volume. The multiplier has been defined through the creation of a slab system which would encourage the PoS to increase the order volumes to move higher up the discount slabs:

| Volume Slab | Discount Multiplier |
| ----------- | ----------- |
| > 60 percentile and < 70 percentile | 1.1 |
| > 70 percentile and < 80 percentile | 1.15 |
| > 80 percentile and < 90 percentile | 1.2 |
| > 90 percentile and < 95 percentile | 1.3 |
| > 95 percentile and < 99 percentile | 1.35 |
| > 99 percentile and < 99.9 percentile | 1.4 |
| > 99.9 percentile | 1.5 |
<br>

4. **Sales Share**: For each POC, it is the ratio of volume sold by that POC to total volume of liquor sold in that province.


5. **Product Cost**: It is the price of a particular product being sold at POC. Assuming tax rate to be r, total transaction value will be Tax/r, thus procuct price can be given as Tax/(r\*volume)

### Models 

**Training Classifier**

Two classifier are trained:
* Decision Tree Classifier: Creates a model that predicts the value of a target variable by learning simple decision rules inferred from the data features. 
* XGBoost Classifer: Boosting is based on week learners and it reduces the bias error thus is usefull to explain variance of the dataset.

Decision Tree has high bias, low variance whereas XGBoost has low bias, high variance, thus ensemble of these two models produces the best results.

**Regressor**

* The Regressor is trained on Non-Zero data points only, while the classifier models are trained on the complete dataset.
* The Regression results are stacked on the classification predictions for generating the final inference.

XGBoost and Random Forest Regressor were used for inferencing. 

### Results:
|   Discount |RMSE | RMSE| MAE | MAE | MAPE | MAPE|
|---------------|------|------|------|------|------|------|
|               | **Train** | **Test** |  **Train** | **Test** |**Train**| **Test** |
| Off-Invoice Discount | 0.77 | 0.76 | 0.69 | 0.70 | 0.97 | 0.96 |
| On-Invoice Discount    | 186.34 | 268.34 | 36.48 | 53.22 | 42.28 | 42.96 |

## Requirements

For installing requirements run - 
```

pip install -r requirements.txt
```

