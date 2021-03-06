# titanic
practice for titanic
import matplotlib.pyplot as plt

import numpy as np

import pandas as pd

import statsmodels.api as sm

from statsmodels.nonparametric.kde import KDEUnivariate

from statsmodels.nonparametric import smoothers_lowess

from pandas import DataFrame

from patsy import dmatrices

df = pd.read_csv('C:/Users/huangshanshan1/downloads/train.csv')

df = df.drop(['Cabin'],axis=1)#而非df.drop(['Cabin'],axis=1)
fig = plt.figure(figsize=(18,6),dpi=100)
alpha = alpha_scatterplot = 0.2
alpha_bar_chart = 0.55

##开始群体作图
ax1 = plt.subplot2grid((2,3),(0,0))
df.Survived.value_counts().plot(kind='bar',alpha=alpha_bar_chart)
ax1.set_xlim(-1,2)
plt.title("Distribution of Survival,(1 = Survived)")

#年龄和幸存
ax2 = plt.subplot2grid((2,3),(0,1))
plt.scatter(df.Survived,df.Age,alpha=alpha_scatterplot)
plt.ylabel("Age")
plt.grid(b=True,which='major',axis='y')
plt.title("Survial by Age,(1 = Survived)")
#阶级
ax3 = plt.subplot2grid((2,3),(0,2))
df.Pclass.value_counts().plot(kind="barh",alpha=alpha_bar_chart)
ax3.set_ylim(-1,len(df.Pclass.value_counts()))
plt.title("Class Distribution")
#年龄与阶级
plt.subplot2grid((2,3),(1,0),colspan=2)
df.Age[df.Pclass == 1].plot(kind='kde')
df.Age[df.Pclass == 2].plot(kind='kde')
df.Age[df.Pclass == 3].plot(kind='kde')
plt.xlabel("Age")
plt.title("Survial by Age,(1 = Survived)")
plt.legend(('1st Class','2nd Class','3rd Class'),loc='best')
#位置
ax5 = plt.subplot2grid((2,3),(1,2))
df.Embarked.value_counts().plot(kind='bar',alpha=alpha_bar_chart)
ax5.set_xlim(-1,len(df.Embarked.value_counts()))
plt.title("Passengers per boarding location")


##更多的可视化
##幸存与否（已经是两个变量）
plt.figure(figsize=(6,4))
fig,ax = plt.subplots()
df.Survived.value_counts().plot(kind='barh',color="blue",alpha = .65)
ax.set_ylim(-1,len(df.Survived.value_counts()))
plt.title("Survival (1 = Survived ,0 = Died ）")

#性别&幸存
ax1 = plt.subplot(1,2,1)
df.Survived[df.Sex == 'male'].value_counts(sort=False).plot(kind='bar',label='Male')
df.Survived[df.Sex == 'female'].value_counts(sort=False).plot(kind='bar',color='#FA2379',label='Female')
ax1.set_xlim(-1,2)
plt.title("Sex Survived Distribution");plt.legend(loc='best')
#性别幸存百分比
ax2 = plt.subplot(1,2,2)
(df.Survived[df.Sex == 'male'].value_counts(sort=False)/float(df.Sex[df.Sex == 'male'].size)).plot(kind='bar',label='Male')
(df.Survived[df.Sex == 'female'].value_counts(sort=False)/float(df.Sex[df.Sex == 'female'].size)).plot(kind='bar',color='#FA2379',label='Female')
ax2.set_xlim(-1,2)
plt.title("Sex Survived Distribution proportion");plt.legend(loc='best')


#三元素

#Survived\Sex\Pclass
#活的高阶女
fig = plt.figure(figsize=(20,7),dpi=100)
alpha_level = 0.65
ax1 = fig.add_subplot(141)
df.Survived[df.Sex=='female'][df.Pclass!=3].value_counts().plot(kind='bar',label='highclassfemale',color='#FA2479')
#df.Survived[df.Sex=='female'][df.Pclass!=3].value_counts().plot(kind='bar',label='female,high class',color='#FA2479')
ax1.set_xlim(-1,len(df.Survived[df.Sex=='female'][df.Pclass != 3].value_counts()))
plt.title("high class female survived");plt.legend(loc="best")
#活的高阶男
ax2 = fig.add_subplot(142)
df.Survived[df.Sex == 'male'][df.Pclass != 3].value_counts().plot(kind='bar',color='steelblue',alpha=alpha_level)
ax1.set_xlim(-1,len(df.Survived[df.Sex == 'male'][df.Pclass !=3].value_counts()))
plt.title("high class male survived");plt.legend(loc="best")
 #活的低阶女
fig = plt.figure(figsize=(20,7),dpi=100)
ax3 = fig.add_subplot(143)
df.Survived[df.Sex == 'female'][df.Pclass == 3].value_counts().plot(kind='bar',color='#FA2479',alpha=alpha_level)
ax1.set_xlim(-1,len(df.Survived[df.Sex == 'female'][df.Pclass == 3].value_counts()))
plt.title("low class female survived");plt.legend(loc="best")
#活的低阶男
ax4 = fig.add_subplot(144)
df.Survived[df.Sex == 'male'][df.Pclass == 3].value_counts().plot(kind='bar',color='steelblue',alpha=alpha_level)
ax1.set_xlim(-1,len(df.Survived[df.Sex == 'male'][df.Pclass == 3].value_counts()))
plt.title("low class male survived");plt.legend(loc="best")
#logistics回归模型
formula = 'Survived ~ C(Pclass) + C(Sex) + Age +SibSp +C(Embarked)'
y,x = dmatrices(formula,data=df,return_type='dataframe')
model = sm.Logit(y,x)
model = model.fit()
model.summary()
#图
plt.figure(figsize=(18,4));
plt.subplot(121,axisbg='#DBDBDB')
ypred = model.predict(x)
plt.plot(x.index,ypred,'bo',x.index,y,'mo',alpha=.25)
plt.grid(color='White',linestyle='dashed')
plt.title('Logit predictions,Blue:\nFitted/predicted values:Red');
#图2
ax2 = plt.subplot(122,axisbg='#DBDBDB')
plt.plot(model.resid_dev,'r-')
plt.grid(color='White',linestyle='dashed')
ax2.set_xlim(-1,len(model.resid_dev))
plt.title('Logit Residuals');
#更多图
fig = plt.figure(figsize=(18,9),dpi=1600)
a = .2
#更多图1
fig.add_subplot(221,axisbg='#DBDBDB')
kde_res = KDEUnivariate(model.predict())
kde_res.fit()
plt.plot(kde_res.support,kde_res.density,alpha=a)
plt.title("Distribution of our Predictions")
#更多图2
fig.add_subplot(222,axisbg='#DBDBDB')
plt.scatter(model.predict(),x['C(Sex)[T.male]'],alpha =a)
plt.grid(b=True,which='major',axis='x')
plt.xlabel('Predicted chance of survival')
plt.ylabel('Gender Bool')
plt.title('The Change of Survival Probability by Gender(1=Male)')
#更多图3
fig.add_subplot(223,axisbg='#DBDBDB')
plt.scatter(model.predict(),x['C(Pclass)[T.3]'],alpha =a)
plt.grid(b=True,which='major',axis='x')
plt.xlabel('Predicted chance of survival')
plt.ylabel('Class Bool')
plt.title('The Change of Survival Probability by Class(1= 3rd Class')
#更多图4
fig.add_subplot(224,axisbg='#DBDBDB')
plt.scatter(model.predict(),x.Age,alpha =a)
plt.grid(b=True,linewidth=0.15)
plt.xlabel('Predicted chance of survival')
plt.ylabel('Age')
plt.title('The Change of Survival Probability by Age')
