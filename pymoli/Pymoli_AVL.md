
# UCB Extension Data Science Homework 4
Pymoli Exercise
written by: A. Lam

# Analysis:
- Observation 1: Player Demographics are primarily Male, between ages 20 and 30.
- Observation 2: Despite having upwards of 570 players, the most frequently purchased item has only been bought 11 times, meaning in-game purchase items are quite low.
- Observation 3: Without knowing what the mechanics of the game are or what these items are for, out of the most profitable items available for sale, only one of them is in the top ten most purchased items. 


```python
# Modules
import os
import pandas as pd
import numpy as np
```


```python
# Create path to File
# file 1
file_path = os.path.join('..','Instructions','HeroesOfPymoli','purchase_data.json')
# file 2
# file_path = os.path.join('..','Instructions','HeroesOfPymoli','purchase_data2.json')
```


```python
# Read json file
json_data = pd.read_json(file_path)
json_data.head()
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
      <th>Age</th>
      <th>Gender</th>
      <th>Item ID</th>
      <th>Item Name</th>
      <th>Price</th>
      <th>SN</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>38</td>
      <td>Male</td>
      <td>165</td>
      <td>Bone Crushing Silver Skewer</td>
      <td>3.37</td>
      <td>Aelalis34</td>
    </tr>
    <tr>
      <th>1</th>
      <td>21</td>
      <td>Male</td>
      <td>119</td>
      <td>Stormbringer, Dark Blade of Ending Misery</td>
      <td>2.32</td>
      <td>Eolo46</td>
    </tr>
    <tr>
      <th>2</th>
      <td>34</td>
      <td>Male</td>
      <td>174</td>
      <td>Primitive Blade</td>
      <td>2.46</td>
      <td>Assastnya25</td>
    </tr>
    <tr>
      <th>3</th>
      <td>21</td>
      <td>Male</td>
      <td>92</td>
      <td>Final Critic</td>
      <td>1.36</td>
      <td>Pheusrical25</td>
    </tr>
    <tr>
      <th>4</th>
      <td>23</td>
      <td>Male</td>
      <td>63</td>
      <td>Stormfury Mace</td>
      <td>1.27</td>
      <td>Aela59</td>
    </tr>
  </tbody>
</table>
</div>



# Player Count


```python
player_count = pd.DataFrame({'Total Players' : [json_data['SN'].nunique()]})
player_count
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
      <th>Total Players</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>573</td>
    </tr>
  </tbody>
</table>
</div>



Purchasing Analysis (total)


```python
tot_purchase = pd.DataFrame({
    'Number of Unique Items' : [json_data['Item Name'].nunique()],
    'Average Purchase Price' : ['$' + str(round(json_data['Price'].mean(),2))],
    'Total Number of Purchases' : [len(json_data.index)],
    'Total Revenue' : ['$' + str(json_data['Price'].sum())]
    })
tot_purchase
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
      <th>Average Purchase Price</th>
      <th>Number of Unique Items</th>
      <th>Total Number of Purchases</th>
      <th>Total Revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>$2.93</td>
      <td>179</td>
      <td>780</td>
      <td>$2286.33</td>
    </tr>
  </tbody>
</table>
</div>



# Gender Demographics


```python
# Create gender group
gender_data = json_data.groupby(['Gender'])
# Percentages calculated using player count value
demographics = pd.DataFrame(gender_data['SN'].nunique())
demographics = demographics.rename(columns = {'SN': 'Total Count'})
demographics['Percentage of Players'] = round(100*(gender_data['SN'].nunique()/player_count['Total Players'][0]),2)
demographics
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
      <th>Total Count</th>
      <th>Percentage of Players</th>
    </tr>
    <tr>
      <th>Gender</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Female</th>
      <td>100</td>
      <td>17.45</td>
    </tr>
    <tr>
      <th>Male</th>
      <td>465</td>
      <td>81.15</td>
    </tr>
    <tr>
      <th>Other / Non-Disclosed</th>
      <td>8</td>
      <td>1.40</td>
    </tr>
  </tbody>
</table>
</div>



# Purchasing Analysis (Gender)


```python
purchase_by_gender = pd.concat([gender_data['SN'].count(),
    round(gender_data['Price'].mean(),2), 
    round(gender_data['Price'].sum(),2),
    round(gender_data['Price'].sum()/gender_data['SN'].nunique(),2)], axis = 1)
purchase_by_gender.columns = ['Purchase Count','Average Price','Total Value','Normalized Total']
# purchase_by_gender
```

# Age Demographics


```python
# Setting up bins and group names
bins = [x * 5 for x in [0,2,3,4,6,8,10,12,16,20]]
group_names = []
for i in range(1,len(bins)):
    if i == 1:
        group_names.append('< ' + str(bins[i]))
    elif i == (len(bins)-1):
        group_names.append(str(bins[i-1]) + '+')    
    else:
        group_names.append(str(bins[i-1]) + ' - ' + str(bins[i]))
```


```python
# bin data and create group
json_data_binned = json_data
json_data_binned['Age Group'] = pd.cut(json_data['Age'],bins,labels = group_names)
age_data = json_data_binned.groupby(['Age Group'])
# json_data_binned.describe()
```


```python
player_count['Total Players'][0]
```




    573




```python
age_data['SN'].nunique()
```




    Age Group
    < 10        22
    10 - 15     54
    15 - 20    139
    20 - 30    286
    30 - 40     69
    40 - 50      3
    50 - 60      0
    60 - 80      0
    80+          0
    Name: SN, dtype: int64




```python
age_demographics = pd.concat([
    age_data['SN'].nunique(),
    round(100*(age_data['SN'].nunique()/player_count['Total Players'][0]),2)
    ], axis = 1)
age_demographics.columns=['Total Count','Percentage of Players']
age_demographics
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
      <th>Total Count</th>
      <th>Percentage of Players</th>
    </tr>
    <tr>
      <th>Age Group</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt; 10</th>
      <td>22</td>
      <td>3.84</td>
    </tr>
    <tr>
      <th>10 - 15</th>
      <td>54</td>
      <td>9.42</td>
    </tr>
    <tr>
      <th>15 - 20</th>
      <td>139</td>
      <td>24.26</td>
    </tr>
    <tr>
      <th>20 - 30</th>
      <td>286</td>
      <td>49.91</td>
    </tr>
    <tr>
      <th>30 - 40</th>
      <td>69</td>
      <td>12.04</td>
    </tr>
    <tr>
      <th>40 - 50</th>
      <td>3</td>
      <td>0.52</td>
    </tr>
    <tr>
      <th>50 - 60</th>
      <td>0</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>60 - 80</th>
      <td>0</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>80+</th>
      <td>0</td>
      <td>0.00</td>
    </tr>
  </tbody>
</table>
</div>



# Purchasing Analysis (Age)


```python
purchase_by_age = pd.concat([
    age_data['SN'].count(),
    round(age_data['Price'].mean(),2),
    round(age_data['Price'].sum(),2),
    round(age_data['Price'].sum()/age_data['SN'].nunique(),2)
    ], axis = 1)
purchase_by_age.columns = ['Purchase Count','Average Price','Total Value','Normalized Total']
# remove empty rows
purchase_by_age.dropna(how = 'any', inplace=True)
purchase_by_age
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
      <th>Purchase Count</th>
      <th>Average Price</th>
      <th>Total Value</th>
      <th>Normalized Total</th>
    </tr>
    <tr>
      <th>Age Group</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt; 10</th>
      <td>32</td>
      <td>3.02</td>
      <td>96.62</td>
      <td>4.39</td>
    </tr>
    <tr>
      <th>10 - 15</th>
      <td>78</td>
      <td>2.87</td>
      <td>224.15</td>
      <td>4.15</td>
    </tr>
    <tr>
      <th>15 - 20</th>
      <td>184</td>
      <td>2.87</td>
      <td>528.74</td>
      <td>3.80</td>
    </tr>
    <tr>
      <th>20 - 30</th>
      <td>381</td>
      <td>2.95</td>
      <td>1122.43</td>
      <td>3.92</td>
    </tr>
    <tr>
      <th>30 - 40</th>
      <td>102</td>
      <td>3.00</td>
      <td>305.75</td>
      <td>4.43</td>
    </tr>
    <tr>
      <th>40 - 50</th>
      <td>3</td>
      <td>2.88</td>
      <td>8.64</td>
      <td>2.88</td>
    </tr>
  </tbody>
</table>
</div>



# Top Spenders


```python
# group by SN, sort by sum of purchase
individuals = json_data.groupby(['SN'])
purchase_by_name = pd.concat([
    individuals['Price'].sum(),
    individuals['Price'].count(),
    round(individuals['Price'].mean(),2)], axis = 1)
purchase_by_name.columns=['Total Purchase','Count of Purchases','Average Purchase']
purchase_by_name.sort_values(by = 'Total Purchase' ,ascending = False).head()
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
      <th>Total Purchase</th>
      <th>Count of Purchases</th>
      <th>Average Purchase</th>
    </tr>
    <tr>
      <th>SN</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Undirrala66</th>
      <td>17.06</td>
      <td>5</td>
      <td>3.41</td>
    </tr>
    <tr>
      <th>Saedue76</th>
      <td>13.56</td>
      <td>4</td>
      <td>3.39</td>
    </tr>
    <tr>
      <th>Mindimnya67</th>
      <td>12.74</td>
      <td>4</td>
      <td>3.18</td>
    </tr>
    <tr>
      <th>Haellysu29</th>
      <td>12.73</td>
      <td>3</td>
      <td>4.24</td>
    </tr>
    <tr>
      <th>Eoda93</th>
      <td>11.58</td>
      <td>3</td>
      <td>3.86</td>
    </tr>
  </tbody>
</table>
</div>



# Most Popular Items


```python
# group by item name, sort by purchase count
items = json_data.groupby(['Item ID'])
purchase_by_item = pd.concat([
    items['Item Name'].min(),
    items['Price'].count(),
    items['Price'].min(),
    items['Price'].sum()], axis = 1)
purchase_by_item.columns=['Item Name','Count of Purchases','Item Price','Total Purchase Value']
purchase_by_item.sort_values(by = 'Count of Purchases' ,ascending = False).head(10)
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
      <th>Item Name</th>
      <th>Count of Purchases</th>
      <th>Item Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Item ID</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>39</th>
      <td>Betrayal, Whisper of Grieving Widows</td>
      <td>11</td>
      <td>2.35</td>
      <td>25.85</td>
    </tr>
    <tr>
      <th>84</th>
      <td>Arcane Gem</td>
      <td>11</td>
      <td>2.23</td>
      <td>24.53</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Trickster</td>
      <td>9</td>
      <td>2.07</td>
      <td>18.63</td>
    </tr>
    <tr>
      <th>175</th>
      <td>Woeful Adamantite Claymore</td>
      <td>9</td>
      <td>1.24</td>
      <td>11.16</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Serenity</td>
      <td>9</td>
      <td>1.49</td>
      <td>13.41</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Retribution Axe</td>
      <td>9</td>
      <td>4.14</td>
      <td>37.26</td>
    </tr>
    <tr>
      <th>65</th>
      <td>Conqueror Adamantite Mace</td>
      <td>8</td>
      <td>1.96</td>
      <td>15.68</td>
    </tr>
    <tr>
      <th>152</th>
      <td>Darkheart</td>
      <td>8</td>
      <td>3.15</td>
      <td>25.20</td>
    </tr>
    <tr>
      <th>44</th>
      <td>Bonecarvin Battle Axe</td>
      <td>8</td>
      <td>2.46</td>
      <td>19.68</td>
    </tr>
    <tr>
      <th>107</th>
      <td>Splitter, Foe Of Subtlety</td>
      <td>8</td>
      <td>3.61</td>
      <td>28.88</td>
    </tr>
  </tbody>
</table>
</div>



# Most Profitable Items


```python
purchase_by_item.sort_values(by = 'Total Purchase Value' ,ascending = False).head()
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
      <th>Item Name</th>
      <th>Count of Purchases</th>
      <th>Item Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Item ID</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>34</th>
      <td>Retribution Axe</td>
      <td>9</td>
      <td>4.14</td>
      <td>37.26</td>
    </tr>
    <tr>
      <th>115</th>
      <td>Spectral Diamond Doomblade</td>
      <td>7</td>
      <td>4.25</td>
      <td>29.75</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Orenmir</td>
      <td>6</td>
      <td>4.95</td>
      <td>29.70</td>
    </tr>
    <tr>
      <th>103</th>
      <td>Singed Scalpel</td>
      <td>6</td>
      <td>4.87</td>
      <td>29.22</td>
    </tr>
    <tr>
      <th>107</th>
      <td>Splitter, Foe Of Subtlety</td>
      <td>8</td>
      <td>3.61</td>
      <td>28.88</td>
    </tr>
  </tbody>
</table>
</div>


