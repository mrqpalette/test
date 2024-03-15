## [라이브러리 import]
1. numpy
2. pandas
3. (필요시) aicentro.session.Session
4. (필요시) aicentro.framework.keras.Keras

import pandas as pd
import numpy as np

from aicentro.session import Session
from aicentro.framework.keras import Keras as AiduFrm
from aicentro.framework.tensorflow import Tensorflow as AiduFrm

sacp_session = Session(verify=False)
aidu_framework = AiduFrm(session=aidu_session)

## [데이터불러오기]
데이터 프레임 변수로 저장(여기서는 CSV 기준으로 진행)
* csv : pd.read_csv("파일이름. csv")
* txt : pd.read_csv("파일이름. csv", sep="구분자") delimiter = sep
* xlsx : pd.read_excel('파일이름.xlsx')
* pickle : pd.read_pickle("파일이름.pkl")
* json : pd.read_json('파일이름.json')

pip install openpyxl
df = pd.read_excel('Feature Website.xlsx', engine='openpyxl')
df = pd.read_csv('TrainData.csv',delimiter=',')
df = pd.read_csv("air_data_01.csv", header=0, encoding='cp949')
df = pd.read_csv(aidu_framework.config.data_dir + "/파일일므.csv", sep = "|")

## 데이터 저장
df.to_csv(aidu_framework.config.data_dir + '/ansan_data_pre.csv', encoding='utf-8',header=True, index=False)

## 데이터 확인
df.info()
df.describe()
df.head()
df.tail()
df.dtypes
df.shape
df.columns
df[7:10] 7,8,9행을 가지고 옴 (인덱스 기준)
df[0:5] ~5행을 가지고 옴 (인덱스 기준)
df.loc[[100, 200, 300], ['cust_class', 'sex_type', 'age', 'r3m_avg_bill_amt', 'r3m_A_avg_arpu_amt']]   # row, col
df.loc vs dfiloc 차이는 구굴링

df['Result_v1'].unique()


## [데이터 변경]
# python 함수 정의 & for문
def col_len(col_val) :
    col_val_len = len(col_val)
    return col_val_len

cols_len = []  #전역변수

for index, row in df.iterrows():
    #print(index, col_len(row[0]))
    cols_len.append({index, col_len(row[0])})
    
print(cols_len)

## [type 변경]
# NaN의 경우 int type을 지원하지 않아 float type으로 변경
df=df.astype({'age': float })
# Intger type으로 변경
df=df.astype({'age': int})

# "_" 항목을 NaN 값으로 치환
import numpy as np
df = df.replace("_", np.NaN)

## [중복 데이터 제거] : drop_duplicates()
- inplace : 원본을 변경할지의 여부입니다.

df=df.drop_duplicates()
df.drop_duplicates(inplace=True)

## 필요한 컬럼만 추출하기

# 1) 컬럼만 추출
cust=df[["cust_class","sex_type","age","efct_svc_count","dt_stop_yn","npay_yn","r3m_avg_bill_amt","r3m_A_avg_arpu_amt","r3m_B_avg_arpu_amt", "termination_yn"]]

# 2)  컬럼&별칭사용 추출
cust=cust.rename(columns = {"cust_class" : 'class',"sex_type":'sex', "efct_svc_count":'service', "dt_stop_yn":'stop',"npay_yn":'npay', "r3m_avg_bill_amt":'avg_bill', "r3m_A_avg_arpu_amt":"A_bill", "r3m_B_avg_arpu_amt":'B_bill', "termination_yn":'termination'})
cust.head()

## [column 추가]
df['새로운컬럼명'] = 값 or df[기존 행]에서 계산된 값

## [column 삭제하기]
# axis : dataframe은 차원이 존재 함으로 항상 0과 1이 존재 
# (0은 행레벨, 1을 열 레벨)
df.drop('컬럼명', axis=1)
df.drop('r3m_avg_bill_amt10', axis=1, inplace=True) #열삭제&원본반영

## [컬럼명 변경]
df.rename({변경전컬럼명,변경후컬럼명}, ... ,axis=1)


## group by
* count : 데이터 개수
* #size : 집단별 크기
* sum : 데이터의 합
* mean, std, var : 평균, 표준편차, 분산
* min, max : 최소, 최대값

df_group = df.groupby('컬럼명')

# count 함수 확인
df_group.count()
# mean 함수 확인
df_group.mean()
# max값 확인하기
df_group.max()

## 결측치
* __listwise 방식__ : record의 항목 중 1개의 값이라도 NA이면 해당 데이터 행 전체 제거
* __pairwise 방식__ : 모든 항목이 NA인 데이터 행만 제거

#listwise 방식으로 제거 하기
df=df.dropna()

#pairwise 방식으로 제거하기
df=df.dropna(how='all')

## 결측치 채우기
#fillna 함수를 사용해서 특정 숫자나 문자로 결측치를 처리하는 방법
df=df.fillna(15)
df=df.fillna(method='backfill') # 뒤에 있는 data를 사용해서 결측치를 처리
df=df.fillna(method='ffill') # 앞에 있는 data를 사용해서 결측치 처리
df['age']=df['age'].replace(np.nan, df['age'].median())
df=df.interpolate() # interpolate 함수의 선형 방법을 사용하여 결측값을 채우기

## 이상치
# 범주형 데이터 이상치 처리
#범주형 데아터의 값 분포 확인하기
print(cust['sex'].value_counts())
print(cust['class'].value_counts())
print(cust['npay'].value_counts())

#이상치 제거하기
cust[(cust['class']!='H')]

#이상치 변경하기 
cust_data['class'].replace('H','F')

# 수치형 데이터 이상치 처리
#이상치를 제거하는 함수 만들기
def removeOutliers(x, column):
    # Q1, Q3구하기
    q1 = x[column].quantile(0.25)
    q3 = x[column].quantile(0.75)
    
    # 1.5 * IQR(Q3 - Q1)
    iqr = 1.5 * (q3 - q1)
    
    # 이상치를 제거
    y=x[(x[column] < (q3 + iqr)) & (x[column] > (q1 - iqr))]
    
    return(y)

# 이상치를 변경하는 함수 만들기
def changeOutliers(data, column):
    x=data.copy()
    # Q1, Q3구하기
    q1 = x[column].quantile(0.25)
    q3 = x[column].quantile(0.75)
    
    # 1.5 * IQR(Q3 - Q1)
    iqr = 1.5 * (q3 - q1)
    
    #이상치 대체값 설정하기
    Min = 0
    if (q1 - iqr) > 0 : Min=(q1 - iqr)
        
    Max = q3 + iqr
    
    # 이상치를 변경
    # X의 값을 직졉 변경해도 되지만 Pyhon Warning이 나오기 떄문에 인덱스를 이용
    x.loc[(x[column] > Max), column]= Max
    x.loc[(x[column] < Min), column]= Min
    
    return(x)

 ##Feature Engineering
 #Binning 연속형 변수를 범주형 변수로 만드는 방법 
 - cut(), qcut : 구간을 지정해서 범주화

q1=cust_data['avg_bill'].quantile(0.25)
q3=cust_data['avg_bill'].quantile(0.75)
pd.cut(cust_data["avg_bill"], bins=[0,q1,q3,cust_data["avg_bill"].max()] , labels=['low', 'mid','high'])
pd.qcut(cust_data["avg_bill"], 3 , labels=['low', 'mid','high'])

#Scaling
Standardization
정규 분포를 평균이 0 이고 분산이 1 인  표준 정규 분포로 변환합니다.
#표준화
Standardization_df = (cust_data_num - cust_data_num.mean())/cust_data_num.std()

#Normalization
#사이킷런 패키지의 MinMaxScaler를 이용하여  Scaling 하기
from sklearn.preprocessing import MinMaxScaler
scaler=MinMaxScaler()
normalization_df=cust_data_num.copy()
normalization_df[:]=scaler.fit_transform(normalization_df[:])
normalization_df.head()

#One-Hot Encording
#pandas에서는 get_dummies함수를 사용하면 쉽게 One-Hot Encording이 가능
pd.get_dummies(cust_data['class'])
#columns를 사용해서 기존 테이블에 One-Hot Encording으로 치환된 변수 생성하기
cust_data_end=pd.get_dummies(cust_data, columns=['class'])







## 데이터 시각화 

1) import
%matplotlib inline
import matplotlib.pyplot as plt

2) 차트그리기
#Matplotlib를 사용하여 간단한 차트를 그리기
plt.figure()
plt.plot([1,2,3], [100,120,110])
plt.show()