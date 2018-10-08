---
title: "Extract-Transform-Load (ETL) with Python"
date: 2018-10-07
layout: posts
header:
  overlay_image: "images/ETL-python/ETL-Process1.jpg"
excerpt: " Read, Clean and Load bunch of csv files to MySQL with Python "
comments: true
toc: true
---

<h2>Create schema and load data in MySQL using python</h2>

I wanted to create a staging area in mysql to build Datawarehouse from bunch of csv files.
<br>
Before uploading the data in mysql I would want to perform some data quality check. 

1. Read csv or read data from the source API.
2. Load only files we need from the directory.
3. perform data quality check
 - find duplicate rows
 - find if dataset has a primary key 
 - check for null values
4. set up mysql engine
5. fuction to generate SQL to create schema in mysql
6. load csv using panda 
 - perform data cleaning tasks on each file
 - load dataframe in mysql
 - clean memory after each load
    

---
First Import following libraries


```python
import numpy as np
import pandas as pd
## set the connection to the db
import sqlalchemy
import pymysql
from IPython.display import Image
```

## About data:

### Background

Since CMS released the claims payment information for 2012, CMS has received a growing number of data requests for provider enrollment data, and there is a growing interest from the health care industry to identify Medicare-enrolled providers and suppliers and their associations to groups/organizations.<br>
<br>
CMS continues to move toward data transparency for all non-sensitive Medicare provider and supplier enrollment data. This aligns with the agency’s effort to promote and practice data transparency for Medicare information. Publishing this data allows users, including other health plans, to easily access and validate provider information against Medicare data.
<br>
### Scope 

The Public Provider Enrollment files will include enrollment information for providers and suppliers who were approved to bill Medicare at the time the file was created. The data elements on the files are disclosable to the public. This data will focus on data relationships as they relate to Medicare provider enrollment.<br>
<br>
The provider enrollment data will be published on https://data.cms.gov/public-provider-enrollment and will be updated on a quarterly basis. The initial data will consist of individual and organization provider and supplier enrollment information similar to what is on Physician Compare; however, it will be directly from PECOS and will only be updated through updates to enrollment information. Elements will include:<br>

-   Enrollment ID and PECOS Unique IDs
- 	Provider or Supplier Enrollment Type and State
- 	Provider’s or Supplier’s First and Last Name/ Legal Business Name
- 	Gender
- 	NPI
- 	Provider or Supplier Specialty 
- 	Limited address information (City, State, ZIP code)

## Data exploratory
lets import a example csv file


```python
example_df = pd.read_csv('Data/Clinic_Group_Practice_Reassignment_A-D.csv')
example_df.head(6)
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
      <th>Group PAC ID</th>
      <th>Group Enrollment ID</th>
      <th>Group Legal_Business Name</th>
      <th>Group State Code</th>
      <th>Group Due Date</th>
      <th>Group Reassignments and Physician Assistants</th>
      <th>Record Type</th>
      <th>Individual Enrollment ID</th>
      <th>Individual NPI</th>
      <th>Individual First Name</th>
      <th>Individual Last Name</th>
      <th>Individual State Code</th>
      <th>Individual Specialty Description</th>
      <th>Individual Due Date</th>
      <th>Individual Total Employer Associations</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1153364203</td>
      <td>O20050609000145</td>
      <td>A &amp; A Audiology, Pc</td>
      <td>TX</td>
      <td>NaN</td>
      <td>1</td>
      <td>Reassignment</td>
      <td>I20050609000172</td>
      <td>1.306861e+09</td>
      <td>Tonia</td>
      <td>Fleming</td>
      <td>TX</td>
      <td>Audiologist</td>
      <td>NaN</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1759440886</td>
      <td>O20081105000006</td>
      <td>A &amp; A Chiropractic. Llc</td>
      <td>NJ</td>
      <td>NaN</td>
      <td>1</td>
      <td>Reassignment</td>
      <td>I20080819000132</td>
      <td>1.134400e+09</td>
      <td>Sharon</td>
      <td>Barnum</td>
      <td>NJ</td>
      <td>Chiropractic</td>
      <td>NaN</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1153442256</td>
      <td>O20101222000765</td>
      <td>A &amp; A Eye Associates, Pc</td>
      <td>PA</td>
      <td>09/30/2017</td>
      <td>2</td>
      <td>Reassignment</td>
      <td>I20101222000889</td>
      <td>1.538155e+09</td>
      <td>Daniel</td>
      <td>Anderson</td>
      <td>PA</td>
      <td>Optometry</td>
      <td>09/30/2017</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1153442256</td>
      <td>O20101222000765</td>
      <td>A &amp; A Eye Associates, Pc</td>
      <td>PA</td>
      <td>09/30/2017</td>
      <td>2</td>
      <td>Reassignment</td>
      <td>I20091118000149</td>
      <td>1.407852e+09</td>
      <td>Amanda</td>
      <td>Temnykh</td>
      <td>PA</td>
      <td>Optometry</td>
      <td>07/31/2016</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3274425004</td>
      <td>O20040326000876</td>
      <td>A &amp; A Health Systems, Inc</td>
      <td>MS</td>
      <td>NaN</td>
      <td>1</td>
      <td>Reassignment</td>
      <td>I20040329000500</td>
      <td>1.063480e+09</td>
      <td>Sheryll</td>
      <td>Vincent</td>
      <td>MS</td>
      <td>Pediatric Medicine</td>
      <td>NaN</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>4082882519</td>
      <td>O20110727000379</td>
      <td>A &amp; A Hearing Group, Ps</td>
      <td>WA</td>
      <td>06/30/2016</td>
      <td>2</td>
      <td>Reassignment</td>
      <td>I20110727000462</td>
      <td>1.396781e+09</td>
      <td>Ashley</td>
      <td>Al Izzi</td>
      <td>WA</td>
      <td>Audiologist</td>
      <td>08/31/2016</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



Lets look at the data types of dataframe


```python
dtype_pd = pd.DataFrame(example_df.dtypes, columns = ['data_type']).reset_index()
unique_records = pd.DataFrame(example_df.nunique(), columns = ['unique_records']).reset_index()
info_df = pd.merge(dtype_pd, unique_records, on = 'index')
info_df
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
      <th>index</th>
      <th>data_type</th>
      <th>unique_records</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Group PAC ID</td>
      <td>int64</td>
      <td>59730</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Group Enrollment ID</td>
      <td>object</td>
      <td>63038</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Group Legal_Business Name</td>
      <td>object</td>
      <td>59176</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Group State Code</td>
      <td>object</td>
      <td>55</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Group Due Date</td>
      <td>object</td>
      <td>35</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Group Reassignments and Physician Assistants</td>
      <td>object</td>
      <td>542</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Record Type</td>
      <td>object</td>
      <td>2</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Individual Enrollment ID</td>
      <td>object</td>
      <td>450876</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Individual NPI</td>
      <td>float64</td>
      <td>429093</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Individual First Name</td>
      <td>object</td>
      <td>39181</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Individual Last Name</td>
      <td>object</td>
      <td>134023</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Individual State Code</td>
      <td>object</td>
      <td>56</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Individual Specialty Description</td>
      <td>object</td>
      <td>169</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Individual Due Date</td>
      <td>object</td>
      <td>35</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Individual Total Employer Associations</td>
      <td>int64</td>
      <td>51</td>
    </tr>
  </tbody>
</table>
</div>



---
It seams that there's no primary key in our DataFrame
lets check that:


```python
print('Is there a column with unique values in entire dataFrame? : ', len(example_df) in info_df['unique_records'])
```

Is there a column with unique values in entire dataFrame? :  False
    


```python
example_df.describe().T
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
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Group PAC ID</th>
      <td>552790.0</td>
      <td>4.962019e+09</td>
      <td>2.888691e+09</td>
      <td>4.210108e+07</td>
      <td>2.567352e+09</td>
      <td>4.880689e+09</td>
      <td>7.517196e+09</td>
      <td>9.931498e+09</td>
    </tr>
    <tr>
      <th>Individual NPI</th>
      <td>552788.0</td>
      <td>1.499834e+09</td>
      <td>2.881970e+08</td>
      <td>1.003000e+09</td>
      <td>1.245784e+09</td>
      <td>1.497990e+09</td>
      <td>1.750322e+09</td>
      <td>1.993000e+09</td>
    </tr>
    <tr>
      <th>Individual Total Employer Associations</th>
      <td>552790.0</td>
      <td>2.927397e+00</td>
      <td>5.034805e+00</td>
      <td>1.000000e+00</td>
      <td>1.000000e+00</td>
      <td>2.000000e+00</td>
      <td>3.000000e+00</td>
      <td>7.800000e+01</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Are there duplicate Rows?
print('Total rows repeating is/are', sum(example_df.duplicated()))
```

Total rows repeating is/are 0
    

## Conclusion from data Quality Check

- DataFrame columns have space in them, we'll have to remove them before writing sql 
- Data has to be converted in mysql data type
- Dataframe has no primary key column
 - We will have to create a PK in mySql with our schema query
- We will need to have assign BIGINT for INT columns, since there max exceeds INT criteria
- No Duplicate records

---
## Connect to Mysql
We'll have to do follow following procedure
- Rename column name
- Set up a connection to MySql
- Define Data Mapping as per MySql
- Write a function to return a sql schema according to input dataframe. 
- Close connection


```python
#engine = sqlchemy.create_engine('mysql+pymsql://<username>:<password>@<server-name>:<port_number>/<database_name>')
engine = sqlalchemy.create_engine('mysql+pymysql://root:info7370@localhost:3306/try_python')
```


```python
Image(filename='empty.JPG')
```




![jpeg](/images/2018-10-07-ETL_with_python_16_0.jpeg)




```python

sql_table_name= 'provider'
initial_sql = "CREATE TABLE IF NOT EXISTS " +str(sql_table_name)+ "(key_pk INT AUTO_INCREMENT PRIMARY KEY"

def rename_df_cols(df):
    '''Input a dataframe, outputs same dataframe with No Space in column names'''
    col_no_space =  dict((i, i.replace(' ','')) for i in list(df.columns))
    df.rename(columns= col_no_space, index= str, inplace= True)
    return df

def dtype_mapping():
    '''Returns a dict to refer correct data type for mysql'''
    return {'object' : 'TEXT',
        'int64' : 'BIGINT',
        'float64' : 'FLOAT',
        'datetime64' : 'DATETIME',
        'bool' : 'TINYINT',
        'category' : 'TEXT',
        'timedelta[ns]' : 'TEXT'}

#engine = sqlchemy.create_engine('mysql+pymsql://<username>:<password>@<server-name>:<port_number>/<database_name>')

def create_sql(engine, df, sql = initial_sql):
    '''input engine: engine (connection for mysql), df: dataframe that you would like to create a schema for,
        outputs Mysql schema creation'''

    df = rename_df_cols(df)

    col_list_dtype = [(i, str(df[i].dtype)) for i in list(df.columns)]

    map_data= dtype_mapping()

    for i in col_list_dtype:
        key = str(df[i[0]].dtypes)
        sql += ", " + str(i[0])+ ' '+ map_data[key]
    sql= sql + str(')')
    
    print('\n', sql, '\n')
    
    try:
        conn = engine.raw_connection()
    except ValueError:
        print('You have connection problem with Mysql, check engine parameters')
    
    cur = conn.cursor()
    
    try:
        cur.execute(sql)
    except ValueError: 
        print("Ohh Damn it couldn't create schema, check Sql again")
    
    cur.close()         
    
```


```python
create_sql(engine, df= example_df)
```

    
CREATE TABLE IF NOT EXISTS provider(key_pk INT AUTO_INCREMENT PRIMARY KEY, GroupPACID BIGINT, GroupEnrollmentID TEXT, GroupLegal_BusinessName TEXT, GroupStateCode TEXT, GroupDueDate TEXT, GroupReassignmentsandPhysicianAssistants TEXT, RecordType TEXT, IndividualEnrollmentID TEXT, IndividualNPI FLOAT, IndividualFirstName TEXT, IndividualLastName TEXT, IndividualStateCode TEXT, IndividualSpecialtyDescription TEXT, IndividualDueDate TEXT, IndividualTotalEmployerAssociations BIGINT) 
    
    


```python
Image(filename='Schema.JPG')
```




![jpeg](/images/2018-10-07-ETL_with_python_19_0.jpeg)



---
## Load bulk csv files in sequence:

We'll now load the data in the schema that we just created
- Define relative path 
 - Add Data in your path
- Search for files that need to be uploaded
- For loop to read files and load in mysql
 - data cleaning in every loop
 - clearing memory after every loop
 


```python
import os

from pathlib import Path
mypath = Path().absolute()


data = str(mypath) + str('\\Data\\')
list_of_files = os.listdir(data)
dir_data = [str(data)+str(i) for i in list_of_files if "Clinic_Group_Practice_Reassignment" in i]

def load_data_mysql(dir_data):
    for i in dir_data:
        i = pd.read_csv(i, low_memory= False)
        rename_df_cols(i)
        i.to_sql(name= sql_table_name,con = engine, index =False, if_exists = 'append',)
        lst = list(i)
        del lst
```

---
## Time to load

Lets calculate the time to run and later we'll compare it with SSIS


```python
Image(filename='Loaded.JPG')
```




![jpeg](/images/2018-10-07-ETL_with_python_23_0.jpeg)




```python
import time
start_time = time.time()
load_data_mysql(dir_data=dir_data)
print("--- %s seconds ---" % (time.time() - start_time))
```

--- 133.69091987609863 seconds ---
    

#### So my laptop with 12 GB ram took 133 seconds to read clean and upload all the data to MySQL
#### Data is 450 MBs
#### Total rows loaded is 2,288,756

## This post is still in progress...