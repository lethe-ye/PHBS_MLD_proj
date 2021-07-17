# PHBS_MLD_proj

This is the courese project of Machine Learning

# PART1 Introduction

## 1.1 Motivation

As the global financial market is generating mass data of different types every day, it is becoming more crucial and more **difficult to effectively extract and use these data to predict the trend of stocks**. The short-term timing strategy has a few difficulties, a few of which are listed as follows:

1. Market sentiments strongly influence the short-term market trend;
2. How to extract effective factors;
3. How to build nonlinear factors;
4. How to solve collinearity among factors.

## 1.2 Our project goal

In this project, we recognize the **price up or down** as a **classification problem** and implement several **machine learning algorithms** to predict the future price up or down of **WindA Index(Y)**([881001.csv](https://github.com/lethe-ye/PHBS_MLD_proj/blob/main/00%20data/881001.csv)), an index indicating the trend of Chinese A Share stocks, to build a **short-term timing strategy**.

## 1.3 Brief Summary of Dataset

The X (dataset) consists of three parts: **macroeconomic data in china**([cleanedFactor.pkl](https://github.com/lethe-ye/PHBS_MLD_proj/blob/main/00%20data/cleanedFactor.pkl)), **American index indicators** and some alpha factors built using OHLC prices of WindA as in WorldQuant101.
The Y is 0/1 **boolean value indicating fall/rise of windA** in next trading day.
The total number of features is 60.
The time period: from 20080401 to 20200306.
The data can be acquired from Wind Database directly. All factors are based on daily frequency data.

## 1.4 Dataset sample
Here is a sample of the dataset.

See our [proposal notebook](https://github.com/lethe-ye/PHBS_MLD_proj/blob/main/01%20proposalMaterials/assignment3_proposal.ipynb).

## 1.5 Work Flow

We plan to implement a feature selection to choose certain features (factors) out of 52 daily factor data and some alpha factors from WorldQuant101 to establish classifiers using logistic regression, naive Bayes, KNN, perceptron, decision tree, SVM, XGBoost and a Sequential neural network model in Keras to predict the rise or fall of Wind All A Index the next day. We build our models and renew them, using them to predict the price up or down of the next trading days. Next we calculate the long-short net asset value (NAV) of WindA based on the position we hold according to the predictions. After that we do some tests and assessments on the strategy. The whole work flow is shown in the following figure.
![images](01%20proposalMaterials/workFlow.PNG)

## 1.6 Rolling Prediction

As the financial data are time series data, we implement an **expanding window** training and prediction procedure as follows: 

1. We get at least 1800 days' data as the training dataset and use k-fold cross validation method to tune the hyperparameters for the best model, so the first signal we can get is the 1801 day.
2. The signal is the predict results of the up or down of WindA Index in the next day. If the signal is predicted to be 1, then we buy WindA Index at the close of the day. If it is predicted as 0, then we short WindA or do nothing at the close of the day.
3. We use the best model in Step 2 for 20 consecutive trading days and then add the 20 days' data into the training set in Step 1 to enter Step 1 again.
![images](01%20proposalMaterials/expand.png)
As we can see from the figure above, every 20 consecutive trading days the training dataset will expand 20 more days' data.

# PART2 Data Preprocessing and Feature Selection

## 2.1 Data Collection

The goal of our timing model is to forecast Wind All A Index, using 60 factors including interest rate factors, commodity factors and other factors. And the time range of our data is from April 1, 2008 to March 6, 2020. All raw data are collected in the [rawMacroFactor](https://raw.githubusercontent.com/lethe-ye/PHBS_MLD_proj/main/00%20data/rawMacroFactor). 

Except for the factors chosen in our reference research report, we add two kinds of features. One of them is Shanghai Composite Index, which is a good indicator that reflects Chinese stock market expectations. The other are stock indexes of foreign stock market, including Nikkei Index and three American stock indexes, because we believe that the volatility of foreign stock prices can have a significant influence on Chinese market. All new factors are collected in the [newData](https://raw.githubusercontent.com/lethe-ye/PHBS_MLD_proj/main/00%20data/newData). We list a few of these factors in the following table.

|                  Name(Chinese)                  |                             Name                             |                          Frequency                           |
| :---------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  银行间同业拆借加权利率：1天：过去五天的变化率  | Interbank Offered Rate: 1 day: Change rate in   the past five days |                              D                               |
|                  7天期回购利率                  |                       7-day repo rate                        |                              D                               |
|         7天期回购利率：过去五天的变化率         |      7-day repo rate: Change rate in the past five days      |                              D                               |
|             银行间回购加权利率：7天             |        Interbank repo weighted interest rate: 7 days         |                              D                               |
|    银行间回购加权利率：7天：过去五天的变化率    | Interbank repo weighted interest rate: 7 days: Change rate in the past five days |                              D                               |
|                shibor利率（0N）                 |                         shibor（0N）                         |                              D                               |
|                shibor利率（1W）                 |                         shibor（1W）                         |                              D                               |
|                shibor利率（2W）                 |                         shibor（2W）                         |                              D                               |
|                shibor利率（1M）                 |                         shibor（1M）                         |                              D                               |
|                shibor利率（3M）                 |                         shibor（3M）                         |                              D                               |
|                shibor利率（6M）                 |                         shibor（6M）                         |                              D                               |
|       shibor利率（0N）：过去五天的变化率        |       shibor（0N）: Change rate in the past five days        |                              D                               |
|       shibor利率（1W）：过去五天的变化率        |       shibor（1W）: Change rate in the past five days        |                              D                               |
|       shibor利率（2W）：过去五天的变化率        |       shibor（2W）: Change rate in the past five days        |                              D                               |
|       shibor利率（1M）：过去五天的变化率        |       shibor（1M）: Change rate in the past five days        |                              D                               |
|       shibor利率（3M）：过去五天的变化率        |       shibor（3M）: Change rate in the past five days        |                              D                               |
|       shibor利率（6M）：过去五天的变化率        |       shibor（6M）: Change rate in the past five days        |                              D                               |
|            中债国债到期收益率（0年）            |                  Government Bond YTM（0Y）                   |                              D                               |
|   中债国债到期收益率（0年）：过去五天的变化率   | Government Bond YTM（0Y）: Change rate in the past five days |                              D                               |
|            中债国债到期收益率（3年）            |                  Government Bond YTM（3Y）                   |                              D                               |
|   中债国债到期收益率（3年）：过去五天的变化率   | Government Bond YTM（3Y）: Change rate in the past five days |                              D                               |
|                    南华指数                     |                             NHCI                             |                              D                               |
|           南华指数：过去五天的变化率            |          NHCI: Change rate   in the past five days           |                              D                               |
|                CRB现货指数：综合                |                             CRB                              |                              D                               |
|       CRB现货指数：综合：过去五天的变化率       |            CRB: Change rate in the past five days            |                              D                               |
|          期货收盘价（连续）：COMEX黄金          |        Futures closing price (continuous): COMEX Gold        |                              D                               |
| 期货收盘价（连续）：COMEX黄金：过去五天的变化率 | Futures closing price (continuous): COMEX Gold: Change   rate in the past five days |                              D                               |
|           期货结算价（连续）：WTI原油           |     Futures settlement price (continuous): WTI Crude Oil     |                              D                               |
|  期货结算价（连续）：WTI原油：过去五天的变化率  | Futures settlement price (continuous): WTI Crude Oil: Change rate in the past five days |                              D                               |
|                COMEX黄金/WTI原油                |                  COMEX Gold/ WTI Crude Oil                   |                              D                               |
|       COMEX黄金/WTI原油：过去五天的变化率       | COMEX Gold/ WTI Crude Oil: Change rate in the   past five days |                              D                               |
|                     标普500                     |                          S & P 500                           |                              D                               |
|            标普500：过去五天的变化率            |         S & P 500: Change rate in the past five days         |                              D                               |
|                 市场动量指标RSI                 |                  Market momentum indicator                   | RSI=Sum(Max(Close-LastClose,0),N,1)/Sum(ABS(Close-LastClose),N,1)*100 |
|         市场动量指标：过去五天的收益率          | Market momentum indicator: Change rate in the past five days |                              D                               |
|           市场交易活跃指标（成交量）            |                            Volume                            |                              D                               |
|    市场交易活跃指标：过去五天成交量的变化率     |          Volume: Change rate in the past five days           |                              D                               |
|                 Beta分离度指标                  |                    Beta resolution index                     | beta is calculated by CAPM, then calculate the difference between 90% percentile and 10% percentile of beta |
|        Beta分离度指标：过去五天的变化率         |   Beta resolution index: Change rate in the past five days   |                              D                               |
|              50ETF过去60日的波动率              |            50ETF volatility over the past 60 days            |                              D                               |
|     50ETF过去60日的波动率：过去五天的变化率     | 50ETF volatility over the past 60 days: Change   rate in the past five days |                              D                               |
|             50ETF过去120日的波动率              |            50ETF volatility over the past 60 days            |                              D                               |
|    50ETF过去120日的波动率：过去五天的变化率     | 50ETF volatility over the past 60 days: Change rate in the past five days |                              D                               |
|           银行间同业拆借加权利率：1天           |                Interbank Offered Rate: 1 day                 |                              D                               |
|                   日经225指数                   |                            Nikki                             |                              D                               |
|                 道琼斯工业指数                  |                             DJIA                             |                              D                               |
|                10年期美债收益率                 |                   YTM of 10- year Treasury                   |                              D                               |

## 2.2 Label Generation

We compute the daily return of WindA index and label each day based on the return. If the return of the next day is positive, which means the price goes up, we label this day 1; else we label this day 0.

## 2.3 Tackle with NaN

Then we compute the number of NaN in each factor. After we drop NaN including non-trading day data and other missing data, we get a DataFrame including 2,903 observations. Cleaned factors are in the [cleanedFactor](https://raw.githubusercontent.com/lethe-ye/PHBS_MLD_proj/main/02%20dataProcess/cleanedFactor.pkl).

## 2.4 Feature Selection

We can see that correlation among these factors are relatively high, which is easy to understand. In order to solve this problem, we adopt some particular feature selection functions to deal with this issue as can be seen in the following part.

Here we build five models to [select features](https://github.com/lethe-ye/PHBS_MLD_proj/tree/main/03%20featureSelection):

* [naiveSelection.py](https://github.com/lethe-ye/PHBS_MLD_proj/tree/main/03%20featureSelection/naiveSelection.py)
* [pcaSelection.py](https://github.com/lethe-ye/PHBS_MLD_proj/blob/main/03%20featureSelection/pcaSelection.py)
* [SVCL1Selection.py](https://github.com/lethe-ye/PHBS_MLD_proj/blob/main/03%20featureSelection/SVCL1Selection.py)
* [treeSelection.py](https://github.com/lethe-ye/PHBS_MLD_proj/blob/main/03%20featureSelection/treeSelection.py)
* [varianceThresholdSelection.py](https://github.com/lethe-ye/PHBS_MLD_proj/blob/main/03%20featureSelection/varianceThresholdSelection.py)

To avoid high correlation among features as much as possible, we can choose [LASSO in SVC model](https://github.com/lethe-ye/PHBS_MLD_proj/blob/main/03%20featureSelection/SVCL1Selection.py). To find the most import features, we can choose PCA methods. Also, XGBoost includes feature selection itself. Moreover, to make it easy to call feature selection model, we encapsulate them as standard functions.

Below is a sample feature selection function ([pcaSelection.py](https://github.com/lethe-ye/PHBS_MLD_proj/blob/main/03%20featureSelection/pcaSelection.py)). As we can see, 12 PCA components can explain 82% of total variance, so we consider that 12 is the proper number of features to work with. 

<p align="center">Table 2. Portion explained by different numbers of components


| Number of PCA components | Total explained variance |
| ------------------------ | ------------------------ |
| 6                        | 65%                      |
| 8                        | 74%                      |
| 10                       | 79%                      |
| 12                       | 82%                      |

```python
import pandas as pd
import os
from sklearn import preprocessing
import warnings
warnings.filterwarnings('ignore')
from sklearn.decomposition import PCA

def pcaSelection(X_train, y_train, X_test, y_test, verbal = None, returnCoef = False):
    '''
    choose the feature selection method = 'pca'
    fit any feature_selection model with the X_train, y_train
    transform the X_train, X_test with the model
    do not use the X_test to build feature selection model
    
    return the selected X_train, X_test
    print info of the selecter
    return the coef or the score of each feature if asked
    
    '''
    #transform to standardscaler
    features = X_train.columns.tolist()
    scaler = preprocessing.StandardScaler().fit(X_train)
    X_train = pd.DataFrame(scaler.transform(X_train))
    X_test = pd.DataFrame(scaler.transform(X_test))
    X_train.columns = features
    X_test.columns = features
    
    pca = PCA(n_components = 12)
    X_train = pca.fit_transform(X_train)
    print ('The explained variance ratio is:')
    print(pca.explained_variance_ratio_)
    print('The total explained variance ratio is ')
    print(sum(pca.explained_variance_ratio_))
    print ('The explained variance is:')
    print(pca.explained_variance_)
    X_test = pca.transform(X_test)
    
    coef = pd.Series()
    # featureName = None
    
    if verbal == True:
       print('The total feature number is '+ str(X_train.shape[1]))
       # print('The selected feature name is '+ str(featureName))
       
    if not returnCoef:
        return(X_train, X_test)
    else:
        return(X_train, X_test, coef)
```

# PART3 Building Classifiers

As we have already converted the problem into a classification prediction problem, we need to build [classifiers](https://github.com/lethe-ye/PHBS_MLD_proj/tree/main/04%20buildClassifierModel) based on machine learning algorithms to implement on the selected features.

## 3.1 Machine Learning Algorithms

We implement [logistic regression](https://github.com/lethe-ye/PHBS_MLD_proj/tree/main/04%20buildClassifierModel/MyClassifier.py), [naive Bayes](https://github.com/lethe-ye/PHBS_MLD_proj/tree/main/04%20buildClassifierModel/MyClassifier.py), [KNN](https://github.com/lethe-ye/PHBS_MLD_proj/tree/main/04%20buildClassifierModel/MyKNNClassifier.py), [perceptron](https://github.com/lethe-ye/PHBS_MLD_proj/tree/main/04%20buildClassifierModel/MyClassifier.py), [decision tree](https://github.com/lethe-ye/PHBS_MLD_proj/tree/main/04%20buildClassifierModel/MyDecisionTreeClassifier.py), [SVM](https://github.com/lethe-ye/PHBS_MLD_proj/tree/main/04%20buildClassifierModel/MySVMClassifier.py), [XGBoost](https://github.com/lethe-ye/PHBS_MLD_proj/tree/main/04%20buildClassifierModel/MyXGBoostClassifier.py) and [a Sequential neural network model in Keras](https://github.com/lethe-ye/PHBS_MLD_proj/tree/main/04%20buildClassifierModel/MyDeepLearningClassifier.py) to predict the rise or fall of Wind All A Index the next day. Below are some sample codes of implementing these algorithms as classifiers.

```python
class MyLogisticRegClassifier:
    def __init__(self):
        self.parameter = self.getPara()
        self.model = LogisticRegression()
        
    def getPara(self):
        # do some how cv or things to decide the hyperparameter
        return({})
        
    def fit(self, X, y):
        # do what ever plot or things you like 
        # just like your code
        # self.model.fit(X,y)
        return(self.model.fit(X, y))
```

```python
from xgboost import XGBClassifier
from parametersRepo import *
import matplotlib.pyplot as plt
from matplotlib import pyplot

class MyXGBoostClassifier:
    def __init__(self):
        self.parameter = self.getPara()
        self.model =  XGBClassifier(seed=self.parameter['model_seed'],
                                    n_estimators=self.parameter['n_estimators'],
                                    max_depth=self.parameter['max_depth'],
                                    learning_rate=self.parameter['learning_rate'],
                                    min_child_weight=self.parameter['min_child_weight'])
        
    def getPara(self):
        # do some how cv or things to decide the hyperparameter
        # n_neighbors = 15
        # weights = 'uniform'
        return(paraXGBoost)
        
    def fit(self, X, y):
        # do what ever plot or things you like 
        # just like your code
        self.model.fit(X,y)
        print('The feature importance is :')
        print(self.model.feature_importances_)
        
        plt.figure(figsize = (20, 6))
        pyplot.bar(range(len(self.model.feature_importances_)),     self.model.feature_importances_)
        pyplot.show()
        plt.title('The feature importance')
        plt.savefig('The feature importance')
        plt.show()
        # print('The total score of feature importance is:')
        # print(sum(self.model.feature_importances_))
        return(self.model.fit(X, y))
        
    def predict(self, X):
        return(self.model.predict(X))
```

```python
from parametersRepo import *
from keras.models import Sequential
from keras.layers import Dense, Dropout
from keras import models,layers
import pandas as pd

class MyDeepLearningClassifier:
    def __init__(self):
        self.parameter = None
        self.model = None
        
    def getPara(self):
        # do some how cv or things to decide the hyperparameter
        # return dict
        if self.parameter == None:
            print('Hi~ please first use fit function to get model :)')
        else:
            print('haha! We already trained deepLearning Model~')
            return self.parameter
        return self.parameter
        
    def fit(self, X, y):
        # do what ever plot or things you like 
        # just like your code
        self.parameter = len(X.columns)
        model = models.Sequential()
        model.add(Dense(30,activation = 'relu',input_shape=(len(X.columns),)))
        model.add(Dropout(0.1))
        model.add(Dense(1,activation = 'sigmoid' ))
        # model.summary()
        model.compile( loss='binary_crossentropy', optimizer='adam', metrics=['accuracy'] )
        self.model = model
        return(self.model.fit(X, y,
                              validation_split=0.2, 
                              epochs=1, batch_size=10, verbose=2))
        
    def predict(self, X):
       	 return(pd.Series(self.model.predict_classes(X).flatten()).astype(bool))
```

## 3.2 Importance Features

It would be nice to show the importance of features used. Conveniently, the XGBoost implementation in scikit-learn already collects the feature importance values for us so we can access them via the

```
 model.feature_importance_
```

attribute after fitting a XGBoost Classifier. By executing the code, we can see the importance of the features. Below is the feature importance from one XGBoost model in rolling prediction.

(Thanks to professor.choi's reminder.)

![images](09 readmeMaterial\TheFeatureImportance.png)

<p align="center">Figure 6. The feature importance from XGBoost model</p>

## 3.3 Rolling Prediction

As 1.6 has already explained, we implement an expanding window prediction procedure to predict future price trends of WindA. Based on the predictions, we make our decisions about when to buy/long and when to sell/short. Figure 7 shows the buy and sell points during the whole process ([naiveSelection+XGBoost](https://github.com/lethe-ye/PHBS_MLD_proj/tree/main/05%20rollingPrediction/outputResults/naiveSelection_MyXGBoostClassifier), the below figures all using this pair).

![images](09 readmeMaterial\buySell.png)

<p align="center">Figure 7. the buy and sell time points</p>

# PART4 Timing Investment Return and Assessments

## 4.1 Position Formation and Return Calculation

We form two position strategies: pure long and long-short. We use the (today's position, next day's prediction) pair to explain how these two strategies work.

<p align="center">Table 3. pure long strategy</p>

| (today's position, next day's prediction) pair | movement |
| ---------------------------------------------- | -------- |
| (0, 0)                                         | empty    |
| (0, 1)                                         | buy      |
| (1, 0)                                         | sell     |
| (1, 1)                                         | hold     |

<p align="center">Table 4. long-short strategy</p>

| (today's position, next day's prediction) pair | movement |
| ---------------------------------------------- | -------- |
| (0, 0)                                         | short    |
| (0, 1), (-1, 1)                                | buy      |
| (1, 0)                                         | sell     |
| (1, 1), (-1, 0)                                | hold     |

Implementing these two rules, we calculate the return of the strategies and Figure 8 shows their performances.

## 4.2 Assessments

![images](09 readmeMaterial\performance.png)

<p align="center">Figure 8. Strategies' performances and win time</p>

![images](09 readmeMaterial\SharpRatio.png)

<p align="center">Figure 9. Sharp ratio</p>

![images](09 readmeMaterial\maxDrawback.png)

<p align="center">Figure 10. maxDrawback</p>

# Part5 Model Valuation

![images](09 readmeMaterial\f1.jpg)

<p align="center">Figure 11. Precision and f1-score</p>

# Part6 Conclusion and Further Improvement

## 6.1 Conclusion 

In our project, we build a timing strategy based on the prediction of WindA's performance the next day. And we build 5 feature selection models and 6 classifiers to train models. After trying all combinations of feature selection models and classifiers, we find that combination of naive selection and XGBoost can show the best strategy performance. Table 5 below shows performances of four strategy combinations. We implement precision and F1-score as the accuracy indicators rather than recall is because we want to make sure that the signals we predict are real opportunities so that we do not lose money on wrong signals (so precision is important) but we do not need to predict all the correct signals (so recall is not that important). The selection methods do not set hypeparameters.

<p align="center">Table 5. Performances of each strategy combination</p>

| strategy combination | naive selection+XGBoost                                      | PCA+KNN                                                      |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Sharpe ratio         | 1.25                                                         | -0.14                                                        |
| max draw down        | 42.06%                                                       | 65.36%                                                       |
| precision            | 0.56902                                                      | 0.57159                                                      |
| F1-score             | 0.58705                                                      | 0.56694                                                      |
| parameters           | `paraXGBoost = {'model_seed':100,                'n_estimators':100,                'max_depth':3,                'learning_rate':0.1,                'min_child_weight':1}` | `paraKNN = {'n_neighbors':15,            'weights':'uniform'}` |
| return figure        | ![images](09%20readmeMaterial/windA_naiveSelection_MyXGBoostClassifier_performance.png) | ![images](09%20readmeMaterial/windA_pcaSelection_MyKNNClassifier_performance.png) |

| strategy combination | tree selection+logistic regression                           | SVM(L1) +naive Bayes                                         |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Sharpe ratio         | 0.47                                                         | -0.03                                                        |
| max draw down        | 56.49%                                                       | 69.06%                                                       |
| precision            | 0.55636                                                      | 0.34407                                                      |
| F1-score             | 0.65394                                                      | 0.24248                                                      |
| parameters           | no hyperparameters set                                       | no hyperparameters set                                       |
| return figure        | ![images](09%20readmeMaterial/windA_treeSelection_MyLogisticRegClassifier_performance.png) | ![images](09%20readmeMaterial/windA_SVCL1Selection_MyNaiveBayesClassifier_performance.png) |

Also, we compare two strategies, pure long strategy and long-short strategy, both of which are better than simple holding strategy. Moreover, long-short strategy has better performance, with 406.83% total compounded yield rate from February 25, 2005 to March 18, 2020 and 1.25 daily Sharpe ratio.

Basically, XGBoost (of original manuscripts) for classification can be divided into the following 2 parts: (A) Gradient boosting + regularization + XGBoost decision tree. (B). For large/complicated dataset only: (algorithm optimization) approximate greedy algorithm + parallel learning + weighted quantile sketch + (dealing with missing data)sparsity-aware split finding + (hardware acceleration)cache-aware access + blocks for out-of-core computation major **hyperparameters** that will be covered includes: gamma (gaining threshold, used for pruning), cover(control minimum size of each leaf), lambda (regularization parameter). Compared to AdaBoost, gradient boosting algorithm use same weight(aka. learning rate eta) for each tree in iteration. What's also worth emphasizing in the model part is that, when growing m-th tree, if not specified, XGBoost will grow a tree greedily, which literally means trying all possible partition and choosing the one with greatest gain(which is defined by right_child_node similarity + left_child_node_similarity - parent_node_similarity). After growing the tree, the pruning step is followed, one way to prune is simply computing whether the best gain excessed gamma threshold or not.

As XGBoost implements such an ensemble approach and an automatic feature selection function, this model shows better performance than other models. The 55% precision is actually not bad in the stock market because we can only train the historical data and implement no cross validation to reduce the model variance. The result matches what the research paper of Xingye Securities shows.

We also implement naive selection + XGBoost on three other main indexes in China A-Share market, namely HS300, ZZ500 and ZZ800. The performances are listed as follows in Table 6.

<p align="center">Table 6. Performances of four indexes (naive selection+XGBoost)</p>

| index         | windA                                                        | HS300                                                        |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Sharpe ratio  | 1.25                                                         | 0.36                                                         |
| max draw down | 42.06%                                                       | 41.30%                                                       |
| precision     | 0.56902                                                      | 0.55178                                                      |
| F1-score      | 0.58705                                                      | 0.55641                                                      |
| parameters    | `paraXGBoost = {'model_seed':100,                'n_estimators':100,                'max_depth':3,                'learning_rate':0.1,                'min_child_weight':1}` | `paraXGBoost = {'model_seed':100,                'n_estimators':100,                'max_depth':3,                'learning_rate':0.1,                'min_child_weight':1}` |
| return figure | ![images](09%20readmeMaterial/windA_naiveSelection_MyXGBoostClassifier_performance.png) | ![images](09%20readmeMaterial/hs300_naiveSelection_MyXGBoostClassifier_performance.png) |

| index         | ZZ500                                                        | ZZ800                                                        |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Sharpe ratio  | 1.46                                                         | 0.79                                                         |
| max draw down | 41.66%                                                       | 32.41%                                                       |
| precision     | 0.55212                                                      | 0.54750                                                      |
| F1-score      | 0.58988                                                      | 0.54074                                                      |
| parameters    | `paraXGBoost = {'model_seed':100,                'n_estimators':100,                'max_depth':3,                'learning_rate':0.1,                'min_child_weight':1}` | `paraXGBoost = {'model_seed':100,                'n_estimators':100,                'max_depth':3,                'learning_rate':0.1,                'min_child_weight':1}` |
| return figure | ![images](09%20readmeMaterial/zz500_naiveSelection_MyXGBoostClassifier_performance.png) | ![images](09%20readmeMaterial/zz800_naiveSelection_MyXGBoostClassifier_performance.png) |

## 6.2 Further Improvements

1. Correlation between our features is high, more low-correlation features can be added to improve our model.
2. Tune the hyperparameters of the model and find better hyperparameters for each model.
3. Implement CSCV framework to evaluate the probability of overfitting problem.
4. If this timing strategy framework is flexible enough, we will upload it to the python community to become a package which may benefit for someone.

# Reference

Mingming Yu, Min Guan. (2019). Systematic Asset Allocation Series III of Xingye Securities: A Short-Term Market Timing Strategy Based on AdaBoost Machine Learning Algorithms. Xingye Securities.

Xiaoming Lin, Ye Chen. (2019). Artificial Intelligence Series XXII of Huatai Securities: Probability of Backtest Overfitting Based on CSCV Framework. Huatai Securities.

Zura Kakushadze. (2016). 101 formulaic alphas. *Social Science Electronic Publishing*, *2016*(84), 72–81.





