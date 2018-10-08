---
title: "Machine Learning in Finance - python"
date: 2018-10-04
layout: posts
header:
  overlay_image: "images/finance/finance_cover.jpg"
excerpt: "Machine Learning in Finance with Python, Data Science"
comments: true
toc: true
---
+ [Download Data](https://github.com/charchil10/ML_in_Finance.git)
+ [Jupter Notebook](http://nbviewer.jupyter.org/github/charchil10/ML_in_Finance/blob/7d6791791ef79e11a16771c40f26429bd38bdaf4/Inital_setup.ipynb)

## Lets Start by Importing few Important Libraries
please make sure to save the notebook on the same directory where you have saved the data. 
### Enjoy! 


```python
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
import seaborn as sns
import os
import datetime
from functools import reduce
import warnings
warnings.filterwarnings('ignore')
```

## Set up jupyter notebook
Extract the repositories from git <br>
Run following kernel to get file import hassel free working <br>
#### Enjoy!


```python
from pathlib import Path
mypath = Path().absolute()
mypath = str(mypath) + '/Data'

from os import listdir
from os.path import isfile, join
onlyfiles = [f for f in listdir(mypath) if isfile(join(mypath, f))]
onlyfiles
```




['DGS1.xls',
 'DGS10.xls',
 'DGS2.xls',
 'DGS3.xls',
 'DGS30.xls',
 'DGS3MO.xls',
 'DGS5.xls']



## Example data from the exel, uncleaned
First few lines contains the description about the data, which we don't need for our analysis. 


```python
file_1 = pd.read_excel(str(mypath)+'/DGS10.xls')
file_1.head(15)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>FRED Graph Observations</th>
      <th>Unnamed: 1</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Federal Reserve Economic Data</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Link: https://fred.stlouisfed.org</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Help: https://fred.stlouisfed.org/help-faq</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Economic Research Division</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Federal Reserve Bank of St. Louis</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>DGS10</td>
      <td>10-Year Treasury Constant Maturity Rate, Perce...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Frequency: Daily</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>observation_date</td>
      <td>DGS10</td>
    </tr>
    <tr>
      <th>10</th>
      <td>1962-01-02 00:00:00</td>
      <td>4.06</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1962-01-03 00:00:00</td>
      <td>4.03</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1962-01-04 00:00:00</td>
      <td>3.99</td>
    </tr>
    <tr>
      <th>13</th>
      <td>1962-01-05 00:00:00</td>
      <td>4.02</td>
    </tr>
    <tr>
      <th>14</th>
      <td>1962-01-08 00:00:00</td>
      <td>4.03</td>
    </tr>
  </tbody>
</table>
</div>



## Data Import and Cleaning
    
Using a For loop to find all the excel files <br>
Each loop does following:<br>
1. Import the file <br>
2. Remove the unwanted description.<br>
3. Remove NAs <br>
4. Reset Index <br>
5. Rename the column <br>
6. Remove records with interest rate = 0 (mostly Sunday and public holidays) <br>
7. Convert date from str to datetime <br>



```python

files_xls = [i for i in onlyfiles if i[-3:]== 'xls']                 # import only xls

all_file={}

for j in files_xls:
    
    file = pd.read_excel('Data/'+ str(j)).dropna()                   # import excel file, removed rows with Nan
    
    file.reset_index(inplace= True, drop =True)                      # reset index

    for k in file.index:                                             # To rename the columns, for loop will search for datetime 
        if type(file.loc[k][0]) == datetime.datetime:                # untill finds a datetime and rename the column 
            break
    file.columns = file.iloc[k-1]

    for k,i in  enumerate(file[str(file.columns[0])]):               # Data cleaning for bad datetime
        if type(i) != datetime.datetime:
            file.drop(k, inplace= True)
            
    file.drop(file[file[file.columns[-1]]==0].index, inplace= True)  # Drop all rows with interest rate 0, mainly weekends
    
    file.reset_index(drop= True, inplace = True)                     # Reset Index
    
    file['observation_date'] = pd.to_datetime(file['observation_date'])
    
    all_file[str(j)] = file                                          # Save clean DataFrame in the empty dictionary
    
```

---
## Merge columns from all the DataFrames

Using Pandas.merge(), inner join on date field 
Using reduce fuction, inner join all seven Dataframes in single line of code, 


```python
result = reduce(lambda left, right: pd.merge(left, right , sort=False,
                                              on = ['observation_date'], 
                                              how = 'outer'), all_file.values())
result.tail(15)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>1</th>
      <th>observation_date</th>
      <th>DGS1</th>
      <th>DGS10</th>
      <th>DGS2</th>
      <th>DGS3</th>
      <th>DGS30</th>
      <th>DGS3MO</th>
      <th>DGS5</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>14146</th>
      <td>2018-08-22</td>
      <td>2.43</td>
      <td>2.82</td>
      <td>2.6</td>
      <td>2.65</td>
      <td>2.99</td>
      <td>2.09</td>
      <td>2.7</td>
    </tr>
    <tr>
      <th>14147</th>
      <td>2018-08-23</td>
      <td>2.43</td>
      <td>2.82</td>
      <td>2.61</td>
      <td>2.66</td>
      <td>2.97</td>
      <td>2.08</td>
      <td>2.72</td>
    </tr>
    <tr>
      <th>14148</th>
      <td>2018-08-24</td>
      <td>2.44</td>
      <td>2.82</td>
      <td>2.63</td>
      <td>2.68</td>
      <td>2.97</td>
      <td>2.09</td>
      <td>2.72</td>
    </tr>
    <tr>
      <th>14149</th>
      <td>2018-08-27</td>
      <td>2.47</td>
      <td>2.85</td>
      <td>2.67</td>
      <td>2.7</td>
      <td>3</td>
      <td>2.12</td>
      <td>2.74</td>
    </tr>
    <tr>
      <th>14150</th>
      <td>2018-08-28</td>
      <td>2.47</td>
      <td>2.88</td>
      <td>2.67</td>
      <td>2.73</td>
      <td>3.03</td>
      <td>2.13</td>
      <td>2.77</td>
    </tr>
    <tr>
      <th>14151</th>
      <td>2018-08-29</td>
      <td>2.48</td>
      <td>2.89</td>
      <td>2.67</td>
      <td>2.75</td>
      <td>3.02</td>
      <td>2.13</td>
      <td>2.78</td>
    </tr>
    <tr>
      <th>14152</th>
      <td>2018-08-30</td>
      <td>2.47</td>
      <td>2.86</td>
      <td>2.64</td>
      <td>2.72</td>
      <td>3</td>
      <td>2.11</td>
      <td>2.75</td>
    </tr>
    <tr>
      <th>14153</th>
      <td>2018-08-31</td>
      <td>2.46</td>
      <td>2.86</td>
      <td>2.62</td>
      <td>2.7</td>
      <td>3.02</td>
      <td>2.11</td>
      <td>2.74</td>
    </tr>
    <tr>
      <th>14154</th>
      <td>2018-09-04</td>
      <td>2.49</td>
      <td>2.9</td>
      <td>2.66</td>
      <td>2.73</td>
      <td>3.07</td>
      <td>2.13</td>
      <td>2.78</td>
    </tr>
    <tr>
      <th>14155</th>
      <td>2018-09-05</td>
      <td>2.49</td>
      <td>2.9</td>
      <td>2.66</td>
      <td>2.72</td>
      <td>3.08</td>
      <td>2.14</td>
      <td>2.77</td>
    </tr>
    <tr>
      <th>14156</th>
      <td>2018-09-06</td>
      <td>2.5</td>
      <td>2.88</td>
      <td>2.64</td>
      <td>2.71</td>
      <td>3.06</td>
      <td>2.13</td>
      <td>2.76</td>
    </tr>
    <tr>
      <th>14157</th>
      <td>2018-09-07</td>
      <td>2.53</td>
      <td>2.94</td>
      <td>2.71</td>
      <td>2.78</td>
      <td>3.11</td>
      <td>2.14</td>
      <td>2.82</td>
    </tr>
    <tr>
      <th>14158</th>
      <td>2018-09-10</td>
      <td>2.54</td>
      <td>2.94</td>
      <td>2.73</td>
      <td>2.78</td>
      <td>3.09</td>
      <td>2.14</td>
      <td>2.83</td>
    </tr>
    <tr>
      <th>14159</th>
      <td>2018-09-11</td>
      <td>2.55</td>
      <td>2.98</td>
      <td>2.76</td>
      <td>2.83</td>
      <td>3.13</td>
      <td>2.15</td>
      <td>2.87</td>
    </tr>
    <tr>
      <th>14160</th>
      <td>2018-09-12</td>
      <td>2.56</td>
      <td>2.97</td>
      <td>2.74</td>
      <td>2.82</td>
      <td>3.11</td>
      <td>2.16</td>
      <td>2.87</td>
    </tr>
  </tbody>
</table>
</div>



---
## Plot Interactive Visualizations

- for installation <br>
$ pip install plotly == 2.7.0 


Using plotly for python,
use mouse pointer to see the exact tooltip values
Enjoy the zoom by drawing a square to part of the graph to look closely. 

#### Enjoy!

Future version will have a drop down filters



```python
import plotly
from plotly.offline import download_plotlyjs, init_notebook_mode, plot, iplot
import cufflinks as cf
init_notebook_mode(connected=True)
cf.go_offline()

#plotly.tools.set_credentials_file(username='charchil1010', api_key='h6iK0B8bluU0nNJ1oZti')

y_col =  [i for i in result.columns if i != 'observation_date']

result.iplot(kind='scatter',x='observation_date',y= y_col, size=25)
```

### If plotly doesn't work
Using matplotlib plot()


```python
result.plot(x ='observation_date', y = y_col, figsize= (30,20))
```




<matplotlib.axes._subplots.AxesSubplot at 0x2188f0d9630>




![png](/images/2018-10-05-Machine_learning_Finance_13_1.png)

## This post is still In process...