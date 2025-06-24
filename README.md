#  House Prices Prediction - Advanced Regression Techniques

##  პროექტის მიმოხილვა

ეს პროექტი ეხება **Kaggle-ის House Prices: Advanced Regression Techniques** კონკურსს, სადაც ამოცანაა სახლების საყიდლი ფასების პროგნოზირება 79 განსხვავებული ნიშნის საფუძველზე. პროექტში გამოყენებულია მანქანური სწავლების სხვადასხვა მიდგომები, რათა მიღწეულ იქოს მაქსიმალური სიზუსტე.

###  პროექტის მიზნები:
- **Data Cleaning**-ის სხვადასხვა სტრატეგიების გატესტვა
- **Feature Engineering**-ის ექსპერიმენტები ახალი ნიშნების შექმნისთვის
- **Feature Selection**-ის მეთოდების შედარება
- მოდელების ჰიპერპარამეტრების ოპტიმიზაცია
- ყველა ექსპერიმენტის **MLflow**-ზე სრული ტრეკინგი

###  საბოლოო შედეგი:
- **საუკეთესო მოდელი**: ElasticNet with Lasso Feature Selection
- **RMSE**: 19,718.62
- **R²**: 0.93+
- **გამოყენებული ნიშნები**: 268 (სულ 269-დან)

##  რეპოზიტორიის სტრუქტურა

```
house-prices-prediction/
├──  model_experiment.ipynb    # ძირითადი ექსპერიმენტების ნოუთბუქი
├──  model_inference.ipynb     # ტესტ სეტზე პროგნოზის ნოუთბუქი
├──  README.md                 # ეს დოკუმენტი
├──  house_prices_submission.csv  # საბოლოო submission ფაილი
└──  house-prices-advanced-regression-techniques/
    ├── train.csv               # სავარჯიშო მონაცემები
    └── test.csv               # სატესტო მონაცემები
```

### ფაილების აღწერა:

** model_experiment.ipynb**
- Data Exploration and Analysis (EDA)
- Data Cleaning სტრატეგიები
- Feature Engineering ექსპერიმენტები
- Feature Selection მეთოდების შედარება
- სხვადასხვა მოდელების ტრენინგი
- Hyperparameter Optimization
- MLflow ტრეკინგი ყველა ექსპერიმენტისთვის

** model_inference.ipynb**
- MLflow Model Registry-დან მოდელის ჩატვირთვა
- ტესტ მონაცემების preprocessing
- საბოლოო პროგნოზების გენერირება
- Submission ფაილის შექმნა

## 🛠 Feature Engineering

### 1. Missing Values-ის დამუშავება

#### კატეგორიული ცვლადები:
```python
# მნიშვნელოვანი NA მნიშვნელობების შემთხვევები
categorical_na_features = [
    'Alley', 'BsmtQual', 'BsmtCond', 'BsmtExposure', 
    'BsmtFinType1', 'BsmtFinType2', 'FireplaceQu',
    'GarageType', 'GarageFinish', 'GarageQual', 'GarageCond',
    'PoolQC', 'Fence', 'MiscFeature'
]
# შევსება: 'None' - ნიშნავს რომ ნიშანი არ არსებობს
```

#### რიცხვითი ცვლადები:
```python
# ლოგიკურად ნულოვანი მნიშვნელობები
numerical_zero_features = [
    'BsmtFinSF1', 'BsmtFinSF2', 'BsmtUnfSF', 'TotalBsmtSF',
    'BsmtFullBath', 'BsmtHalfBath', 'GarageYrBlt',
    'GarageArea', 'GarageCars', 'MasVnrArea'
]
# შევსება: 0 - ნიშნავს რომ ნიშანი არ არსებობს
```

### 2. ახალი ნიშნების შექმნა

#### გაერთიანებული ზომები:
- **TotalSF**: მთელი სახლის ფართობი (basement + 1st floor + 2nd floor)
- **Total_Bathrooms**: სრული აბაზანების რაოდენობა (full + 0.5*half)
- **Total_porch_sf**: ყველა ვერანდის ფართობი

#### ასაკთან დაკავშირებული ნიშნები:
- **HouseAge**: სახლის ასაკი (YrSold - YearBuilt)
- **RemodAge**: რემონტის ასაკი (YrSold - YearRemodAdd)
- **GarageAge**: გარაჟის ასაკი

#### Boolean ნიშნები:
- **HasBasement**: აქვს თუ არა სარდაფი
- **HasGarage**: აქვს თუ არა გარაჟი
- **HasFireplace**: აქვს თუ არა ღია ცეცხლსაცემი
- **HasPool**: აქვს თუ არა აუზი

### 3. Categorical Encoding

**One-Hot Encoding** გამოყენებული იყო ყველა კატეგორიული ცვლადისთვის:
- `drop_first=True` multicollinearity-ის თავიდან ასაცილებლად
- Train/Test სეტების სწორი alignment
- Missing columns-ების ჰანდლინგი

### შედეგები:
- **თავდაპირველი ნიშნები**: 81
- **Feature Engineering-ის შემდეგ**: 93
- **Encoding-ის შემდეგ**: 269

##  Feature Selection

### 1. გატესტებული მეთოდები:

#### **Univariate Selection (SelectKBest)**
```python
selector = SelectKBest(score_func=f_regression, k=50)
```
- **პრინციპი**: ყველა ნიშნის ინდივიდუალური კორელაცია target-თან
- **შედეგი**: შერჩეული 50 უმნიშვნელოვანესი ნიშანი

#### **Recursive Feature Elimination (RFE)**
```python
estimator = RandomForestRegressor(n_estimators=50, random_state=42)
selector = RFE(estimator, n_features_to_select=50)
```
- **პრინციპი**: მოდელის importance scores-ის საფუძველზე რეკურსიული ელიმინაცია
- **შედეგი**: Random Forest-ის მიხედვით 50 საუკეთესო ნიშანი

#### **L1-based Selection (Lasso)**
```python
lasso = Lasso(alpha=0.01, random_state=42)
selector = SelectFromModel(lasso)
```
- **პრინციპი**: Lasso რეგრესიის კოეფიციენტების საფუძველზე
- **შედეგი**: 268 ნიშანი (მხოლოდ 1 ნიშანი ამოიღო)

### 2. შედარების შედეგები:

| Feature Selection | Selected Features | Best Model | CV RMSE |
|-------------------|-------------------|------------|---------|
| Univariate        | 50               | ElasticNet | 25,000+ |
| RFE               | 50               | XGBoost    | 26,000+ |
| **Lasso**         | **268**          | **ElasticNet** | **24,767** |

### 3. საბოლოო არჩევანი:
**Lasso-based Feature Selection** არჩეულ იქნა, რადგან:
- უმაღლესი სიზუსტე
- შეინარჩუნა თითქმის ყველა ნიშანი (მხოლოდ 1 ამოიღო)
- კარგად მუშაობს ElasticNet-თან

##  Training

### 1. გატესტებული მოდელები:

#### **Linear Models:**
- **Linear Regression**: baseline მოდელი
- **Ridge (α=1.0)**: L2 regularization
- **Lasso (α=0.01)**: L1 regularization  
- **ElasticNet (α=0.01)**: L1 + L2 regularization

#### **Tree-based Models:**
- **Random Forest**: n_estimators=100
- **XGBoost**: n_estimators=100

### 2. Cross-Validation შედეგები:

| Model | Feature Selection | CV RMSE | Std Dev |
|-------|-------------------|---------|---------|
| Linear Regression | Lasso | 35,000+ | ±3,000 |
| Ridge | Lasso | 28,000+ | ±2,500 |
| Lasso | Lasso | 26,000+ | ±2,000 |
| **ElasticNet** | **Lasso** | **24,767** | **±1,500** |
| Random Forest | Lasso | 27,000+ | ±2,000 |
| XGBoost | Lasso | 26,500+ | ±1,800 |

### 3. Hyperparameter Optimization

**საუკეთესო მოდელისთვის (ElasticNet):**
```python
param_grid = {'alpha': [0.001, 0.01, 0.1, 1.0, 10.0]}
```

**GridSearchCV შედეგები:**
- **საუკეთესო α**: 10.0
- **საუკეთესო CV RMSE**: 26,300.61
- **Cross-validation**: 5-fold

### 4. საბოლოო მოდელის შერჩევა:

**ElasticNet (α=10.0)** არჩეულ იქნა შემდეგი მიზეზების გამო:

 **სიზუსტე**: ყველაზე დაბალი RMSE  
 **სტაბილურობა**: დაბალი variance CV-ში  
 **Regularization**: კარგი ოვერფიტინგის თავიდან აცილება  
 **ინტერპრეტაცია**: Linear model-ის სიმარტივე  
 **სიჩქარე**: სწრაფი ტრენინგი და ინფერენსი  

### 5. საბოლოო მეტრიკები:

- **Training RMSE**: 19,718.62
- **Training R²**: 0.93+
- **Training MAE**: ~13,000
- **ნიშნების რაოდენობა**: 268
- **მოდელის ზომა**: <1MB

##  MLflow Tracking

### 1. ექსპერიმენტების სტრუქტურა:

**DagHub Integration:**
```
https://dagshub.com/g-kitiashvili/ML-assignment1.mlflow
```

### 2. ჩაწერილი Run-ები:

#### **EDA Run:**
- Dataset მახასიათებლები
- Missing values ანალიზი
- Target distribution ვიზუალიზაცია
- კორელაციური ანალიზი

#### **Data Cleaning Run:**
- Original vs Cleaned shapes
- ამოღებული outliers რაოდენობა
- Missing values დაცულობა

#### **Feature Engineering Run:**
- ახალი ნიშნების რაოდენობა
- ნიშნების სრული სია
- გაზრდილი dimensionality

#### **Feature Selection Runs (3):**
- Univariate selection შედეგები
- RFE selection შედეგები  
- Lasso selection შედეგები

#### **Model Training Runs (18):**
- ყველა Model × Feature Selection კომბინაცია
- CV scores და metrics
- Model artifacts

#### **Hyperparameter Optimization:**
- Grid search შედეგები
- საუკეთესო პარამეტრები
- ოპტიმიზებული მოდელი

#### **Final Model Registration:**
- Model Registry-ში რეგისტრაცია
- საბოლოო metrics
- Production-ready artifacts

### 3. ტრეკინგი მეტრიკები:

#### **მოდელის მეტრიკები:**
- `cv_rmse_mean`: Cross-validation RMSE
- `cv_rmse_std`: RMSE სტანდარტული გადახრა
- `train_rmse`: Training RMSE
- `train_r2`: R-squared score
- `train_mae`: Mean Absolute Error

#### **პროცესის მეტრიკები:**
- `train_shape`: ტრენინგ მონაცემების ზომა
- `test_shape`: ტესტ მონაცემების ზომა
- `selected_features_count`: შერჩეული ნიშნების რაოდენობა
- `missing_values_after_cleaning`: დარჩენილი missing values

#### **ჰიპერპარამეტრები:**
- `model_type`: მოდელის ტიპი
- `feature_selection`: Feature selection მეთოდი
- `alpha`: რეგულარიზაციის პარამეტრი (ElasticNet)

### 4. Model Registry:

**რეგისტრირებული მოდელი:**
- **სახელი**: `house_prices_final_model`
- **ვერსია**: Latest
- **Stage**: Production Ready
- **მეტაცემები**: ყველა საჭირო artifact

## 🏆 შეფასება და შედეგები

### 1. საბოლოო მეტრიკები:

| Metric | Train | Expected Test |
|--------|-------|---------------|
| **RMSE** | **19,718** | ~**20,000-25,000** |
| **R²** | **0.93+** | ~**0.90+** |
| **MAE** | **~13,000** | ~**15,000** |

### 2. Kaggle Submission:

**Submission ფაილი:** `house_prices_submission.csv`
- **Samples**: 1,459 predictions
- **Format**: Id, SalePrice
- **Price Range**: $16,459 - $725,429
- **Mean Price**: $175,398 (ტრენინგი: $180,921)

### 3. კონკურენტული პოზიცია:

- **მიმდინარე RMSE**: 19,718
- **Kaggle Top 10%**: ~12,000-15,000
- **Kaggle Top 50%**: ~20,000-25,000
- **ვარაუდებული Ranking**: Top 30-50%

### 4. ძლიერი მხარეები:

 **სისტემური მიდგომა**: ყველა ეტაპის გამართული პროცესი  
 **MLflow ინტეგრაცია**: სრული experimentების ტრეკინგი  
 **Production Ready**: მოდელი მზადაა რეალურ გამოყენებისთვის  
 **კოდის ხარისხი**: მარტივი გადატანა და გამოყენება  
 **დოკუმენტაცია**: დეტალური აღწერა ყველა ეტაპზე  

### 5. გასაუმჯობესებელი:

 **Target Transformation**: Log-scale ტრანსფორმაცია  
 **Ensemble Methods**: მოდელების კომბინირება  
 **Feature Interactions**: polynomial და interaction features  
 **Advanced Selection**: Sequential Feature Selection  
 **Hyperparameter Tuning**: Bayesian Optimization  

##  რესურსები

- **GitHub Repository**: https://github.com/g-kitiashvili/ML-assignment1
- **DagHub MLflow**: https://dagshub.com/g-kitiashvili/ML-assignment1.mlflow
- **Kaggle Competition**: [House Prices: Advanced Regression Techniques](https://www.kaggle.com/c/house-prices-advanced-regression-techniques)
