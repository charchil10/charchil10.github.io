---
title: "Sound recognition by Convolutional2D"
date: 2018-10-04
tags: [machine learning, keras, tensorflow, neural network]
header:
  image: "images/convolutional2d/machine_learning_by_tags.jpg"
excerpt: "General Purpose audio tagging, Nueral Net, Data Science"
---

<h1><center>Can you guess the intrument by the sound?</center></h1>

<h4> <center> Well, Not anymore! In the world of AI we have computers to do so.  Atleast, Fourty One of those covered here!</center><h4>

---
<h4><center> As usual lets start by importing some commonly used libraries of python.</center></h4>

Python code block:

```python
	import numpy as np
	import pandas as pd
	import matplotlib.pyplot as plt
	import seaborn as sns
	import scipy
	%matplotlib inline
```

### We'll need the Audio files to train our PC along with the labels

Thanks to Freesound, Creative Commons and Google's AudioSet Ontology.

Python code block:
```python
	# Using Pandas we'll load the csv files
	train = pd.read_csv('train.csv')
	test = pd.read_csv('test_post_competition.csv')

	test = test[test['label']!= 'None'].head(50)                                   # using only rows with available label
	test = test.reset_index(drop = True)                                           # reset index 
	test.drop(['usage', 'freesound_id', 'license'], axis = 1, inplace =True)       # unnecessary columns
```

#### Lets have a look of first 8 rows of our Dataset.


```python
	train.head(8)
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
      <th>fname</th>
      <th>label</th>
      <th>manually_verified</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>00044347.wav</td>
      <td>Hi-hat</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>001ca53d.wav</td>
      <td>Saxophone</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>002d256b.wav</td>
      <td>Trumpet</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0033e230.wav</td>
      <td>Glockenspiel</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>00353774.wav</td>
      <td>Cello</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>003b91e8.wav</td>
      <td>Cello</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>003da8e5.wav</td>
      <td>Knock</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0048fd00.wav</td>
      <td>Gunshot_or_gunfire</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
test.head(8)
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
      <th>fname</th>
      <th>label</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>00326aa9.wav</td>
      <td>Oboe</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0038a046.wav</td>
      <td>Bass_drum</td>
    </tr>
    <tr>
      <th>2</th>
      <td>007759c4.wav</td>
      <td>Saxophone</td>
    </tr>
    <tr>
      <th>3</th>
      <td>008afd93.wav</td>
      <td>Saxophone</td>
    </tr>
    <tr>
      <th>4</th>
      <td>00ae03f6.wav</td>
      <td>Chime</td>
    </tr>
    <tr>
      <th>5</th>
      <td>00eac343.wav</td>
      <td>Electric_piano</td>
    </tr>
    <tr>
      <th>6</th>
      <td>010a0b3a.wav</td>
      <td>Shatter</td>
    </tr>
    <tr>
      <th>7</th>
      <td>01a5a2a3.wav</td>
      <td>Bark</td>
    </tr>
  </tbody>
</table>
</div>

