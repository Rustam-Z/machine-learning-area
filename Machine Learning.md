# Machine Learning

    - Handling, cleaning, and preparing data.
    - Selecting and engineering features.
    - Learning by fitting a model to data.
    - Optimizing a cost function.
    - Selecting a model and tuning hyperparameters using cross-validation.
    - Underfitting and overfitting (the bias/variance tradeoff).
    - Unsupervised learning techniques: clustering, density estimation and anomaly detection.
    - Algorithms: Linear and Polynomial Regression, Logistic Regression, k-Nearest Neighbors, Support Vector Machines, Decision Trees, Random Forests, and Ensemble methods.
<!-- 
it should include:
- important tips, notes
- code snippets which are usefull when starting new projects
- pipelines, ml models codes

So, here I will include:
End-to-End Machine Learning Project step by step code snippets and notes, section by section. When I will create new project, so that I can refer to any section I stuck.
-->

## End-to-End Machine Learning Project
- Frame the problem and look at the big picture
    - Goal and Performance measure
- Get the data
    - [Create test set](#Create-test-set)
- Explore the data to gain insights (EDA)
    - [Looking for correlations](#Looking-for-Correlations)
    - Experimenting with attribute combinations
- Prepare data for ML algorithms
    - [Data cleaning](#Data-cleaning)
    - [Handling text and categorical attributes](#Handling-text-and-categorical-attributes)
    - [Feature Scaling](#Feature-Scaling)
    - [Transformation Pipelines](#Transformation-Pipelines)
- [Explore many different models and short-list the best ones](#Select-and-Train-a-Model)
    - [Cross-Validation](#Cross-Validation)
- Fine-tune models and combine them into a great solution
    - [Grid Search](#Grid-Search)
    - [Randomized Search](#Randomized-Search)
    - Ensemble models
    - Evaluate on test set
- Launch and monitor

## Get the data
### Create test set
```py
'''Create test set'''
from sklearn.model_selection import train_test_split
train_set, test_set = train_test_split(data, test_size=0.2, random_state=42)
```
```py
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.33, random_state=42)
```
```py
from sklearn.model_selection import StratifiedShuffleSplit
import numpy as np
X = np.array([[1, 2], [3, 4], [1, 2], [3, 4], [1, 2], [3, 4]])
y = np.array([0, 0, 0, 1, 1, 1])
sss = StratifiedShuffleSplit(n_splits=1, test_size=0.5, random_state=0)
sss.get_n_splits(X, y) 

for train_index, test_index in sss.split(X, y):
    print("TRAIN:", train_index, "TEST:", test_index)
    X_train, X_test = X[train_index], X[test_index]
    y_train, y_test = y[train_index], y[test_index]
```

## Explore the data to gain insights
```py
'''Visualizing Geographical Data'''
data.plot(kind="scatter", x="longitude", y="latitude", alpha=0.1)
```
```py
housing.plot(kind="scatter", x="longitude", y="latitude", alpha=0.4,
    s=housing["population"]/100, label="population", figsize=(10,7),
    c="median_house_value", cmap=plt.get_cmap("jet"), colorbar=True,
)
plt.legend()
```
### Looking for Correlations
```py
'''Looking for Correlations'''
corr_matrix = data.corr()
corr_matrix["any_column"].sort_values(ascending=False)

from pandas.plotting import scatter_matrix
attributes = ["median_house_value", "median_income", "total_rooms", "housing_median_age"]
scatter_matrix(housing[attributes], figsize=(12, 8))
```
```py
# Correlations between features
all_data_corr = all_data.corr().abs().unstack().sort_values(kind="quicksort", ascending=False).reset_index()
all_data_corr.rename(columns={"level_0": "Feature 1", "level_1": "Feature 2", 0: 'Correlation Coefficient'}, inplace=True)
all_data_corr.drop(all_data_corr.iloc[1::2].index, inplace=True)
all_data_corr_nd = all_data_corr.drop(all_data_corr[all_data_corr['Correlation Coefficient'] == 1.0].index)

corr = all_data_corr_nd['Correlation Coefficient'] > 0.1
all_data_corr_nd[corr]
```
```py
# pivot_table() vs groupby(), the below lines are the same
pd.pivot_table(df, index=["a"], columns=["b"], values=["c"], aggfunc=np.sum)
df.groupby(['a','b'])['c'].sum()
```
```py
# Aggregate using one or more operations over the specified axis
# agg()-can be applied to multiple groups together
df.agg(['sum', 'min'])
df_all.groupby(['Sex', 'Pclass']).agg(lambda x:x.value_counts().index[0])['Embarked'] 

# Apply a function along an axis of the DataFrame
# apply()-cannot be applied to multiple groups together 
df.apply(np.sqrt)
df_all['Deck'] = df_all['Cabin'].apply(lambda s: s[0] if pd.notnull(s) else 'M')
```

## Prepare data for ML algorithms
- https://stackoverflow.com/questions/48673402/how-can-i-standardize-only-numeric-variables-in-an-sklearn-pipeline
- https://scikit-learn.org/stable/modules/preprocessing.html

### Data Cleaning
```py
housing.dropna(subset=["total_bedrooms"])    # Get rid of the corresponding districts
housing.drop("total_bedrooms", axis=1)       # Get rid of the whole attribute
median = housing["total_bedrooms"].median()  # Set the values to some value (zero, mean, median)
housing["total_bedrooms"].fillna(median, inplace=True)
```
```py
'''SimpleImputer, filling with the missing numerical attributes with the "median"'''
from sklearn.impute import SimpleImputer
imputer = SimpleImputer(strategy="median")
housing_num = housing.select_dtypes(include=[np.number]) # just numerical attributes
imputer.fit(housing_num) # "trained" inputer, now it is ready to transform the training set by replacing missing values with the learned medians
imputer.statistics_ # same as "housing_num.median().values"
X = imputer.transform(housing_num)
housing_tr = pd.DataFrame(X, columns=housing_num.columns,
                          index=housing.index) # new dataframe
```

### Handling Text and Categorical Attributes
- [select_dtypes](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.select_dtypes.html)
```py
'''Transforming continuous numerical attributes to categorical'''
housing["income_cat"] = pd.cut(housing["median_income"], 
                                bins=[0., 1.5, 3.0, 4.5, 6., np.inf], 
                                labels=[1, 2, 3, 4, 5])
```
```py
'''Categorical Attributes'''
from sklearn.preprocessing import OrdinalEncoder
from sklearn.preprocessing import OneHotEncoder

housing_cat = housing[["ocean_proximity"]]

ordinal_encoder = OrdinalEncoder()
housing_cat_encoded = ordinal_encoder.fit_transform(housing_cat)

housing_cat_encoded[:10] 
# array([[0.],
#    [0.],
#    [4.],
#    [1.],
#    [0.],
#    [1.],
#    [0.],
#    [1.],
#    [0.],
#    [0.]])

ordinal_encoder.categories_ # [array(['<1H OCEAN', 'INLAND', 'ISLAND', 'NEAR BAY', 'NEAR OCEAN'], dtype=object)]

cat_encoder = OneHotEncoder(sparse=False)
housing_cat_1hot = cat_encoder.fit_transform(housing_cat)
housing_cat_1hot
# array([[1., 0., 0., 0., 0.],
#        [1., 0., 0., 0., 0.],
#        [0., 0., 0., 0., 1.],
#        ...,
#        [0., 1., 0., 0., 0.],
#        [1., 0., 0., 0., 0.],
#        [0., 0., 0., 1., 0.]])
```
### Feature Scaling
```py
'''StandardScaler'''
from sklearn.preprocessing import StandardScaler
import numpy as np

X_train = np.array([[ 1., -1.,  2.],
                    [ 2.,  0.,  0.],
                    [ 0.,  1., -1.]])
scaler = StandardScaler().fit(X_train)

scaler.mean_
scaler.scale_

X_scaled = scaler.transform(X_train)
X_scaled
```
```py
from sklearn.preprocessing import MinMaxScaler

X_train = np.array([[ 1., -1.,  2.],
                    [ 2.,  0.,  0.],
                    [ 0.,  1., -1.]])

min_max_scaler = MinMaxScaler()
X_train_minmax = min_max_scaler.fit_transform(X_train)
X_train_minmax
# array([[0.5       , 0.        , 1.        ],
#        [1.        , 0.5       , 0.33333333],
#        [0.        , 1.        , 0.        ]])

# For the test data, we just need to use .transform()
X_test = np.array([[-3., -1.,  4.]])
X_test_minmax = min_max_scaler.transform(X_test)
X_test_minmax
# array([[-1.5       ,  0.        ,  1.66666667]])
```

### Custom Transformer
```py
from sklearn.base import BaseEstimator, TransformerMixin

# column index
rooms_ix, bedrooms_ix, population_ix, households_ix = 3, 4, 5, 6

class CombinedAttributesAdder(BaseEstimator, TransformerMixin):
    def __init__(self, add_bedrooms_per_room=True): # no *args or **kargs
        self.add_bedrooms_per_room = add_bedrooms_per_room
    def fit(self, X, y=None):
        return self  # nothing else to do
    def transform(self, X):
        rooms_per_household = X[:, rooms_ix] / X[:, households_ix]
        population_per_household = X[:, population_ix] / X[:, households_ix]
        if self.add_bedrooms_per_room:
            bedrooms_per_room = X[:, bedrooms_ix] / X[:, rooms_ix]
            return np.c_[X, rooms_per_household, population_per_household,
                         bedrooms_per_room]
        else:
            return np.c_[X, rooms_per_household, population_per_household]

attr_adder = CombinedAttributesAdder(add_bedrooms_per_room=False)
housing_extra_attribs = attr_adder.transform(housing.values)
```

### Transformation Pipelines
```py
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.compose import ColumnTransformer

num_pipeline = Pipeline([
        ('imputer', SimpleImputer(strategy="median")),
        ('attribs_adder', CombinedAttributesAdder()),
        ('std_scaler', StandardScaler()),
    ])

# housing_num_tr = num_pipeline.fit_transform(housing_num)

num_attribs = list(housing_num)
cat_attribs = ["ocean_proximity"]

full_pipeline = ColumnTransformer([
        ("num", num_pipeline, num_attribs),
        ("cat", OneHotEncoder(), cat_attribs),
    ])

housing_prepared = full_pipeline.fit_transform(housing)
housing_prepared # to get access to the new dataset
```
## Select and Train a Model
- Before using `.predict()` you have to use `full_pipeline.transform(some_data)`

### Cross-Validation
```py
from sklearn.model_selection import cross_val_score

scores = cross_val_score(model, data, labels, scoring="neg_mean_squared_eroor", cv=10)
rmse_scores = np.sqrt(-scores)

def display_scores(scores):
    print("Scores:", scores)
    print("Mean:", scores.mean())
    print("Standart deviation:", scores.std())

display_scores(rmse_scores)
```
```py
'''Save the model'''
import joblib
joblib.dump(my_model, "my_model.pkl") # to save model
my_model_loaded = joblib.load("my_model.pkl") # to load model
```


## Fine-tune Models
### Grid Search
```py
from sklearn.model_selection import GridSearchCV

param_grid = [
    # try 12 (3×4) combinations of hyperparameters
    {'n_estimators': [3, 10, 30], 'max_features': [2, 4, 6, 8]},
    # then try 6 (2×3) combinations with bootstrap set as False
    {'bootstrap': [False], 'n_estimators': [3, 10], 'max_features': [2, 3, 4]},
]

forest_reg = RandomForestRegressor(random_state=42)
# train across 5 folds, that's a total of (12+6)*5=90 rounds of training 
grid_search = GridSearchCV(forest_reg, param_grid, cv=5,
                           scoring='neg_mean_squared_error',
                           return_train_score=True)
grid_search.fit(housing_prepared, housing_labels)

grid_search.best_params_ # the best hyperparameters
grid_search.best_estimator_

# look at the score of each hyperparameter combination tested during the grid search:
cvres = grid_search.cv_results_
for mean_score, params in zip(cvres["mean_test_score"], cvres["params"]):
    print(np.sqrt(-mean_score), params)
```

### Randomized Search
```py
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import randint

param_distribs = {
        'n_estimators': randint(low=1, high=200),
        'max_features': randint(low=1, high=8),
    }

forest_reg = RandomForestRegressor(random_state=42)
rnd_search = RandomizedSearchCV(forest_reg, param_distributions=param_distribs,
                                n_iter=10, cv=5, scoring='neg_mean_squared_error', random_state=42)
rnd_search.fit(housing_prepared, housing_labels)

# looking at the scores during training
cvres = rnd_search.cv_results_
for mean_score, params in zip(cvres["mean_test_score"], cvres["params"]):
    print(np.sqrt(-mean_score), params)

feature_importances = grid_search.best_estimator_.feature_importances_
```