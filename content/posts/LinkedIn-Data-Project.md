---
date: "2024-01-22T11:22:45-08:00"
title: "LinkedIn-Data-Project"
authors: ['michaelye']
categories:
  - AI
tags:
  - Python
draft: false
---
This is a work-in-progress project where we will build a model that predicts whether an individual on LinkedIn works at a top 15 tech company (by market cap) based on their LinkedIn profile. Our dataset[^1] contains 10,000 LinkedIn profiles from Australian users, and we will only use the profile data from the dataset for the purposes of this analysis. So far, we have only conducted preliminary data analysis on the dataset. 


```python
import numpy as np
import seaborn as sns
from sklearn import linear_model
from sklearn.preprocessing import PolynomialFeatures
from sklearn import metrics
import pandas as pd
import matplotlib.pyplot as plt
```


```python
data = pd.read_csv('dump.csv', index_col=0)
dropCol = ['companyHasLogo', 'companyName', 'companyUrl', 'companyUrn', 'country', 'endDate', 'genderEstimate', 'hasPicture', 'mbrLocation', 'mbrLocationCode', 'mbrTitle', 'memberUrn', 'posLocation', 'posLocationCode', 'posTitle', 'positionId', 'startDate', 'avgMemberPosDuration', 'avgCompanyPosDuration']
```

## Exploratory Data Analysis


```python
ax = plt.axes()
ax.set_title('LinkedIn Profile Job Heat Map')
plotData = data.drop(dropCol, inplace=False, axis=1)
sns.heatmap(plotData.corr(), annot = True, vmin=-1, vmax=1, center= 0, ax = ax)
```




    <Axes: title={'center': 'LinkedIn Profile Job Heat Map'}>




    
![png](output_4_1.png)
    



```python
def isBigTech(d):
    return 'bigTech' if d in ('Facebook', 'Apple', 'Amazon', 'Google', 'NVIDIA', 'Microsoft', 'Tesla', 'TSMC', 'Broadcom', 'Samsung', 'Tencent', 'Oracle', 'ASML', 'AMD', 'Adobe') else 'not top 10'

companies = data['companyName']
new_companies = companies.apply(isBigTech)
plotData.insert(0, 'bigTech', new_companies)
```


```python
sns.pairplot(plotData.sample(1000), hue='bigTech', diag_kind='kde')
```




    <seaborn.axisgrid.PairGrid at 0x7ff7020cdd60>




    
![png](output_6_1.png)
    



```python
print(len(plotData) * .8)
trainSet = plotData[:31630]
testSet = plotData[31630:]
```

    31629.600000000002


[^1]: Andrew Truman. LinkedIn Profiles and Jobs Data, Version 2. Retrieved 01/11/2024 from https://www.kaggle.com/datasets/killbot/linkedin-profiles-and-jobs-data.