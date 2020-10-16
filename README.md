# signate 債務不履行リスク前処理

#Libraries
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import sklearn
import sklearn.linear_model
import sklearn.ensemble
import sklearn.metrics
from dnn_utils_v2 import sigmoid, sigmoid_backward, relu, relu_backward
from testCases_v4a import *
from dnn_app_utils_v3 import *
from sklearn.metrics import f1_score
from sklearn.ensemble import RandomForestRegressor
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import mean_absolute_error
from RandomForestModelImprovement import RandomForestModelImprovement


df = pd.read_csv("signate 債務不履行リスク.csv")
print('dataframeの行数・列数の確認==>\n', df.shape)
print('indexの確認==>\n', df.index)
print('columnの確認==>\n', df.columns)
print("dataframeの各列のデータ型を確認==>\n", df.dtypes)
#df['loan_status'].unique() #array(['FullyPaid', 'ChargedOff'], dtype=object) onehot
#df.isnull().sum()
#df['term'].unique() array(['3 years', '5 years'], dtype=object) onehot
#df['grade'].unique() array(['A5', 'B1', 'C2', 'C1', 'E5', 'D5', 'B4', 'A3', 'D3', 'C3', 'A2',
                               #'D1', 'A1', 'C5', 'A4', 'C4', 'E4', 'B2', 'B5', 'D2', 'D4', 'B3',
                               #'E2', 'E1', 'E3', 'F3', 'F5'], dtype=object)⇒27parameters
#df['employment_length'].unique() array(['0 years', '10 years', '1 year', '2 years', '3 years', '6 years',
                                        #'9 years', '7 years', '4 years', '5 years', '8 years'],
                                         # dtype=object)
#df['purpose'].unique() array(['debt_consolidation', 'credit_card', 'medical', 'other',
                               #'home_improvement', 'car', 'major_purchase', 'small_business',
                               #'house'], dtype=object) onehot
#df['application_type'].unique() array(['Individual', 'Joint App'], dtype=object) onehot


df = pd.get_dummies(df, columns=['application_type'])
df = pd.get_dummies(df, columns=['purpose'])
df = pd.get_dummies(df, columns=['grade'])


loan_status_seri = df["loan_status"]
loan_status_list = loan_status_seri.values.tolist()


loan_status_list2 = []
for loan_status in loan_status_list:
    if loan_status == "FullyPaid":
        loan_status = 0
    else:
        loan_status = 1
    loan_status_list2.append(loan_status)
    
    
loan_status_seri = pd.Series(loan_status_list2)
df["loan_status"] = loan_status_seri


term_seri = df["term"]
term_list = term_seri.values.tolist()


term_list2 =[]
for term in term_list:
    term = term.replace("years", "")
    term = float(term)
    term_list2.append(term)
    
    
term_seri = pd.Series(term_list2)


df["term"] = term_seri


employment_length_seri = df["employment_length"]
employment_length_list = employment_length_seri.values.tolist()


employment_length_list2=[]
for employment_length in employment_length_list:
    if "years" in employment_length:
        employment_length = employment_length.replace("years", "")
        employment_length = float(employment_length)
    else:
        employment_length = employment_length.replace("year", "")
        employment_length = float(employment_length)
    employment_length_list2.append(employment_length)
    
    
employment_length_seri = pd.Series(employment_length_list2)


df["employment_length"] = employment_length_seri
df.head()


def standarize(column):
    df_ = df.copy()
    s = df_[column]
    df_['standardization'] = (s - s.mean()) / s.std()
    df[column] = df_['standardization']
    
    return df


df = standarize("loan_amnt")
df = standarize("interest_rate")
df = standarize("credit_score")
df.head()


#grade_mapping = {'A5':27, 'B1':18, 'C2':14, 'C1':13, 'E5':7, 'D5':12, 'B4':21, 'A3':25, 'D3':10, 'C3':15, 'A2':24,'D1':8, 'A1':23,
                 #'C5':17, 'A4':26, 'C4':16, 'E4':6, 'B2':19, 'B5':22, 'D2':9, 'D4':11, 'B3':20,'E2':4, 'E1':3, 'E3':5, 'F3':1, 'F5':2}
#df["grade"] = df["grade"].map(grade_mapping)
#df.head()


def X_y(df, column):
    y_train = np.array(df[column])
    del df[column]
    del df["id"]
    X_train = df
    return X_train, y_train


X, y = X_y(df, "loan_status")


X_train = X[:230000]
y_train = y[:230000].reshape(-1, 1)

X_val = X[230000:]
y_val = y[230000:].reshape(-1, 1)
