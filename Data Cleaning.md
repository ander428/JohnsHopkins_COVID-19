
# Data Cleaning
***

## Data Aquisition

### Pandemic Data


```python
import pandas as pd
import numpy as np
from numba import njit, jit
from typing import TypeVar
import multiprocessing
from joblib import Parallel, delayed
import time

num_cores = multiprocessing.cpu_count()
PandasDataFrame = TypeVar('pandas.core.frame.DataFrame')
NaN = np.nan
highlighted_countries = ["US", "Australia", "Canada", "China", "Netherlands", "UK", "France", "Denmark"]
```

#### US Cases


```python
cases_US = pd.read_csv("../data/pandemic/time_series_covid19_confirmed_US.csv")

cases_US = cases_US[5:] # exclude US territories
cases_US = cases_US.drop(["FIPS","Combined_Key","code3","iso2", "iso3","UID"], axis=1)
cases_US = cases_US.rename(columns={"Admin2": "County", "Long_": "Long"})

cases_US.head(10)
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
      <th>County</th>
      <th>Province_State</th>
      <th>Country_Region</th>
      <th>Lat</th>
      <th>Long</th>
      <th>1/22/20</th>
      <th>1/23/20</th>
      <th>1/24/20</th>
      <th>1/25/20</th>
      <th>1/26/20</th>
      <th>...</th>
      <th>6/19/20</th>
      <th>6/20/20</th>
      <th>6/21/20</th>
      <th>6/22/20</th>
      <th>6/23/20</th>
      <th>6/24/20</th>
      <th>6/25/20</th>
      <th>6/26/20</th>
      <th>6/27/20</th>
      <th>6/28/20</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.539527</td>
      <td>-86.644082</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>405</td>
      <td>425</td>
      <td>428</td>
      <td>436</td>
      <td>447</td>
      <td>463</td>
      <td>473</td>
      <td>482</td>
      <td>492</td>
      <td>497</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Baldwin</td>
      <td>Alabama</td>
      <td>US</td>
      <td>30.727750</td>
      <td>-87.722071</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>398</td>
      <td>405</td>
      <td>415</td>
      <td>422</td>
      <td>435</td>
      <td>449</td>
      <td>462</td>
      <td>500</td>
      <td>539</td>
      <td>559</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Barbour</td>
      <td>Alabama</td>
      <td>US</td>
      <td>31.868263</td>
      <td>-85.387129</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>265</td>
      <td>271</td>
      <td>271</td>
      <td>276</td>
      <td>279</td>
      <td>287</td>
      <td>303</td>
      <td>309</td>
      <td>314</td>
      <td>314</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Bibb</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.996421</td>
      <td>-87.125115</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>123</td>
      <td>123</td>
      <td>124</td>
      <td>126</td>
      <td>132</td>
      <td>138</td>
      <td>146</td>
      <td>150</td>
      <td>158</td>
      <td>159</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Blount</td>
      <td>Alabama</td>
      <td>US</td>
      <td>33.982109</td>
      <td>-86.567906</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>136</td>
      <td>140</td>
      <td>146</td>
      <td>150</td>
      <td>156</td>
      <td>165</td>
      <td>173</td>
      <td>181</td>
      <td>185</td>
      <td>186</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Bullock</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.100305</td>
      <td>-85.712655</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>318</td>
      <td>324</td>
      <td>324</td>
      <td>325</td>
      <td>325</td>
      <td>332</td>
      <td>347</td>
      <td>347</td>
      <td>354</td>
      <td>353</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Butler</td>
      <td>Alabama</td>
      <td>US</td>
      <td>31.753001</td>
      <td>-86.680575</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>567</td>
      <td>570</td>
      <td>574</td>
      <td>576</td>
      <td>579</td>
      <td>582</td>
      <td>586</td>
      <td>592</td>
      <td>597</td>
      <td>599</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Calhoun</td>
      <td>Alabama</td>
      <td>US</td>
      <td>33.774837</td>
      <td>-85.826304</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>202</td>
      <td>203</td>
      <td>205</td>
      <td>207</td>
      <td>208</td>
      <td>212</td>
      <td>225</td>
      <td>228</td>
      <td>237</td>
      <td>237</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Chambers</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.913601</td>
      <td>-85.390727</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>493</td>
      <td>502</td>
      <td>507</td>
      <td>514</td>
      <td>520</td>
      <td>529</td>
      <td>535</td>
      <td>545</td>
      <td>547</td>
      <td>547</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Cherokee</td>
      <td>Alabama</td>
      <td>US</td>
      <td>34.178060</td>
      <td>-85.606390</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>56</td>
      <td>56</td>
      <td>56</td>
      <td>56</td>
      <td>56</td>
      <td>56</td>
      <td>62</td>
      <td>65</td>
      <td>66</td>
      <td>67</td>
    </tr>
  </tbody>
</table>
<p>10 rows × 164 columns</p>
</div>




```python

```

#### Global Cases


```python
cases_global = pd.read_csv("../data/pandemic/time_series_covid19_confirmed_global.csv")
cases_global = cases_global.rename(columns={"Province/State": "Province_State", 
                                            "Country/Region": "Country_Region"})
cases_global["County"] = NaN
cases_global.head(10)
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
      <th>Province_State</th>
      <th>Country_Region</th>
      <th>Lat</th>
      <th>Long</th>
      <th>1/22/20</th>
      <th>1/23/20</th>
      <th>1/24/20</th>
      <th>1/25/20</th>
      <th>1/26/20</th>
      <th>1/27/20</th>
      <th>...</th>
      <th>6/20/20</th>
      <th>6/21/20</th>
      <th>6/22/20</th>
      <th>6/23/20</th>
      <th>6/24/20</th>
      <th>6/25/20</th>
      <th>6/26/20</th>
      <th>6/27/20</th>
      <th>6/28/20</th>
      <th>County</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Afghanistan</td>
      <td>33.0000</td>
      <td>65.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>28424</td>
      <td>28833</td>
      <td>29157</td>
      <td>29481</td>
      <td>29640</td>
      <td>30175</td>
      <td>30451</td>
      <td>30616</td>
      <td>30967</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>Albania</td>
      <td>41.1533</td>
      <td>20.1683</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>1891</td>
      <td>1962</td>
      <td>1995</td>
      <td>2047</td>
      <td>2114</td>
      <td>2192</td>
      <td>2269</td>
      <td>2330</td>
      <td>2402</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>Algeria</td>
      <td>28.0339</td>
      <td>1.6596</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>11631</td>
      <td>11771</td>
      <td>11920</td>
      <td>12076</td>
      <td>12248</td>
      <td>12445</td>
      <td>12685</td>
      <td>12968</td>
      <td>13273</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>Andorra</td>
      <td>42.5063</td>
      <td>1.5218</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>855</td>
      <td>855</td>
      <td>855</td>
      <td>855</td>
      <td>855</td>
      <td>855</td>
      <td>855</td>
      <td>855</td>
      <td>855</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>Angola</td>
      <td>-11.2027</td>
      <td>17.8739</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>176</td>
      <td>183</td>
      <td>186</td>
      <td>189</td>
      <td>197</td>
      <td>212</td>
      <td>212</td>
      <td>259</td>
      <td>267</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>NaN</td>
      <td>Antigua and Barbuda</td>
      <td>17.0608</td>
      <td>-61.7964</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>65</td>
      <td>65</td>
      <td>65</td>
      <td>69</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>NaN</td>
      <td>Argentina</td>
      <td>-38.4161</td>
      <td>-63.6167</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>41204</td>
      <td>42785</td>
      <td>44931</td>
      <td>47203</td>
      <td>49851</td>
      <td>52457</td>
      <td>55343</td>
      <td>57744</td>
      <td>59933</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>NaN</td>
      <td>Armenia</td>
      <td>40.0691</td>
      <td>45.0382</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>19708</td>
      <td>20268</td>
      <td>20588</td>
      <td>21006</td>
      <td>21717</td>
      <td>22488</td>
      <td>23247</td>
      <td>23909</td>
      <td>24645</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Australian Capital Territory</td>
      <td>Australia</td>
      <td>-35.4735</td>
      <td>149.0124</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>108</td>
      <td>108</td>
      <td>108</td>
      <td>108</td>
      <td>108</td>
      <td>108</td>
      <td>108</td>
      <td>108</td>
      <td>108</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>New South Wales</td>
      <td>Australia</td>
      <td>-33.8688</td>
      <td>151.2093</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>4</td>
      <td>...</td>
      <td>3149</td>
      <td>3151</td>
      <td>3150</td>
      <td>3159</td>
      <td>3162</td>
      <td>3168</td>
      <td>3174</td>
      <td>3177</td>
      <td>3184</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>10 rows × 164 columns</p>
</div>



#### Combine Data and Change Time Series Dimension

**Here I wanted to treat the time series data as one feature. I explored several ways to approach this, but with a lack of user friendly solutions, I iteratively expanded each row. I was able to minimize the runtime through multiprocessing.**


```python
cases_total_temp = pd.concat([cases_US, cases_global], sort=False)
cases_total_temp = cases_total_temp[cases_total_temp['Country_Region'].isin(highlighted_countries)]
```


```python
def get_rows(row):
    temp = pd.DataFrame(columns=pd.DataFrame(columns=['County','Province_State','Country_Region', 
                                                      'Lat','Long','Date','Total_Cases']))
    for date in row[5:].iteritems():
            new_row = row[:5]
            new_row["Date"] = date[0]
            new_row["Total_Cases"] = date[1]
            temp = pd.concat([temp, new_row.to_frame().transpose()])
    return temp

def convert_time_series():
    cols = cases_total_temp.columns[:5].append(pd.Index(["Date","Total_Cases"]))
    temp = pd.DataFrame(columns=cols)

    row_n = 0
    result = Parallel(n_jobs=num_cores-1)(delayed(get_rows)(j) for i, j in cases_total_temp.iterrows())
    return pd.concat(result)

start_time = time.time()
cases_total = convert_time_series()
end_time = time.time() - start_time
# print("--- %s seconds ---" % (end_time))
```


```python
cases_total
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
      <th>County</th>
      <th>Province_State</th>
      <th>Country_Region</th>
      <th>Lat</th>
      <th>Long</th>
      <th>Date</th>
      <th>Total_Cases</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>1/22/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>1/23/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>1/24/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>1/25/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>1/26/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>1/27/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>1/28/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>1/29/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>1/30/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>1/31/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/1/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/2/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/3/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/4/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/5/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/6/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/7/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/8/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/9/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/10/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/11/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/12/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/13/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/14/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/15/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/16/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/17/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/18/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/19/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Autauga</td>
      <td>Alabama</td>
      <td>US</td>
      <td>32.5395</td>
      <td>-86.6441</td>
      <td>2/20/20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>5/30/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>5/31/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/1/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/2/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/3/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/4/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/5/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/6/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/7/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/8/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/9/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/10/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/11/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/12/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/13/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/14/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/15/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/16/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/17/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/18/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/19/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/20/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/21/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/22/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/23/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/24/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/25/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/26/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/27/20</td>
      <td>1</td>
    </tr>
    <tr>
      <th>258</th>
      <td>NaN</td>
      <td>Saint Pierre and Miquelon</td>
      <td>France</td>
      <td>46.8852</td>
      <td>-56.3159</td>
      <td>6/28/20</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>529629 rows × 7 columns</p>
</div>




```python
cases_total.to_csv("../data/pandemic/covid_19_time_series_all.csv")
```