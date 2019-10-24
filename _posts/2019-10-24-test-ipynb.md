
## ref


```python
from google.cloud import bigquery

import pandas as pd

import datetime
import sys

from pandasql import *

pysqldf = lambda q: sqldf(q, globals())
```


```python
import util
from util.gs import df2gs, readFromGs, get_v, updateGSbyDF, dfUpdate

from util.gbqUtil import edxkeyNorm, edxkey2gbq, is_exist

from util.redash_util import get_fresh_query_result

from util.weeklyGoalFit import get_fit
```


```python
open_users = get_fresh_query_result(488)
```


```python
open_users.head()
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
      <th>comments</th>
      <th>email</th>
      <th>engagement</th>
      <th>follows_on_comment</th>
      <th>follows_on_post</th>
      <th>name</th>
      <th>posts</th>
      <th>total_cnt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>268</td>
      <td>agrau@mit.edu</td>
      <td>366.0</td>
      <td>0</td>
      <td>0</td>
      <td>Arthur Grau</td>
      <td>49</td>
      <td>317</td>
    </tr>
    <tr>
      <th>1</th>
      <td>24</td>
      <td>lshanaha@mit.edu</td>
      <td>418.0</td>
      <td>0</td>
      <td>0</td>
      <td>Lindsay Shanahan</td>
      <td>197</td>
      <td>221</td>
    </tr>
    <tr>
      <th>2</th>
      <td>205</td>
      <td>paulg@geetwo.net</td>
      <td>209.0</td>
      <td>0</td>
      <td>0</td>
      <td>Paul F. Groepler</td>
      <td>2</td>
      <td>207</td>
    </tr>
    <tr>
      <th>3</th>
      <td>153</td>
      <td>okiyasteven@yahoo.com</td>
      <td>199.0</td>
      <td>0</td>
      <td>0</td>
      <td>Stephen Shisoka Okiya</td>
      <td>23</td>
      <td>176</td>
    </tr>
    <tr>
      <th>4</th>
      <td>112</td>
      <td>sophielarrouy@googlemail.com</td>
      <td>135.0</td>
      <td>0</td>
      <td>5</td>
      <td>sophie Larrouy</td>
      <td>9</td>
      <td>126</td>
    </tr>
  </tbody>
</table>
</div>




```python
project_id = 'mitx-research'
destination_table = 'MIT_ODL_marketing.open_user_list'
```


```python
open_users.to_gbq(destination_table, 
                  project_id, 
                  chunksize=1000, 
                  reauth=True, 
                  if_exists='replace', 
                  private_key=None)
```

    /home/maxliu/anaconda2/envs/py3/lib/python3.6/site-packages/google/auth/_default.py:66: UserWarning: Your application has authenticated using end user credentials from Google Cloud SDK. We recommend that most server applications use service accounts instead. If your application continues to use end user credentials from Cloud SDK, you might receive a "quota exceeded" or "API not enabled" error. For more information about service accounts, see https://cloud.google.com/docs/authentication/
      warnings.warn(_CLOUD_SDK_CREDENTIALS_WARNING)
    57it [05:25,  5.90s/it]



```python
sql_str = """

    #standardSQL
    
    select 
        a.*,
        case when b.email is Null then 0 else 1 end as is_inMITx,
        b.ncourses
    from `mitx-research.MIT_ODL_marketing.open_user_list` as a
    left join `mitx-data.course_report_latest.email_addresses` as b
    on a.email= b.email


"""
```


```python
df = pd.read_gbq(sql_str, project_id='mitx-research', dialect='standard')
df.head()
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
      <th>comments</th>
      <th>email</th>
      <th>engagement</th>
      <th>follows_on_comment</th>
      <th>follows_on_post</th>
      <th>name</th>
      <th>posts</th>
      <th>total_cnt</th>
      <th>is_inMITx</th>
      <th>ncourses</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>lipkina.anna@gmail.com</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>Anna Lipkina</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>5.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>robertyip88@gmail.com</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>Robert</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>6.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>jplimaribeiro@gmail.com</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>João Paulo Lima Ribeiro</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>5.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>edalover@163.com</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>chai1206</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>solenn.hedouin@gmail.com</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>Solennh</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>5.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(df)
```




    56687




```python
sum(df['is_inMITx'])
```




    47174




```python
sum(df['is_inMITx']) / len(df)
```




    0.8321837458323778




```python
len(df.query("ncourses>=2"))
```




    37721




```python
len(df.query("ncourses>=2")) / len(df)
```




    0.6654259353996507




```python
len(df.query("ncourses>=1")) / len(df)
```




    0.8321837458323778




```python
len(df.query("ncourses>=3")) / len(df)
```




    0.5477975549949724




```python
len(df.query("ncourses>=1").query("total_cnt>0"))
```




    1169




```python
len(df.query("ncourses>=1").query("total_cnt>0")) / len(df)
```




    0.020622012101540035




```python
len(df.query("is_inMITx==0").query("total_cnt>0"))
```




    111




```python

```
