
# UCB Extension Data Science Homework 4
Academy Exercise 
written by: A. Lam

# Analysis
- Observation 1: Top Performing Schools are Charters despite having the lowest per student budgets
- Observation 2: Despite the prompt to categorize school into 4 spending bins on a per student basis, the spread is surprisingly small. (~$80) It would be interesting to know what time span this budget is applicable to.
- Observation 3: Smaller school appear to have distinctly higher passing rates overall in both subjects.


```python
# Modules
import os
import pandas as pd
import numpy as np
```


```python
# Create path to File
# file 1
school_path = os.path.join('..','Instructions','PyCitySchools','raw_data','schools_complete.csv')
# file 2
student_path = os.path.join('..','Instructions','PyCitySchools','raw_data','students_complete.csv')
```


```python
# Read file
school_data = pd.read_csv(school_path)
student_data = pd.read_csv(student_path)
```

# District Summary

Create overall district metrics incl. % passing in math and reading, total students, etc.
Will need to bin students into passing or failing on scores, and subsequently group over school.


```python
# Categorize students as passing or failing
bins = [0,69,100]
score = ['failing','passing']
student_data['reading_result'] = pd.cut(student_data['reading_score'],bins,labels = score)
student_data['math_result'] = pd.cut(student_data['math_score'],bins,labels = score)
# student_data[['math_result','math_score','reading_result','reading_score']].head()
```


```python
# calculate values
n_schools = school_data['name'].nunique()
n_students = student_data['name'].count()
tot_budget = school_data['budget'].sum()
mean_math = student_data['math_score'].mean()
mean_reading = student_data['reading_score'].mean()
pcnt_pass_math = 100*student_data['math_result'].value_counts()['passing'] / n_students
pcnt_pass_reading = 100*student_data['reading_result'].value_counts()['passing'] / n_students
oall_pass = 100*student_data[(student_data['math_result'] == 'passing') & 
              (student_data['reading_result'] == 'passing')].count()['Student ID'] / n_students
```


```python
# create dataframe
district_summary = pd.DataFrame({
    'Total Schools' : n_schools,
    'Total Students' : n_students,
    'Total Budget' : tot_budget,
    'Average Math Score' : mean_math,
    'Average Reading Score' : mean_reading,
    '% Passing Math' : pcnt_pass_math,
    '% Passing Reading' : pcnt_pass_reading,
    'Overall Passing Rate' : oall_pass},index =['District'])
district_summary.head()
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
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>Overall Passing Rate</th>
      <th>Total Budget</th>
      <th>Total Schools</th>
      <th>Total Students</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>District</th>
      <td>74.980853</td>
      <td>85.805463</td>
      <td>78.985371</td>
      <td>81.87784</td>
      <td>65.172326</td>
      <td>24649428</td>
      <td>15</td>
      <td>39170</td>
    </tr>
  </tbody>
</table>
</div>



# School Summary


```python
# school_data.head()
```


```python
# student_data.head()
```


```python
# replace 'passing' with True and 'failing' with False
def res2bool(arg):
    if arg == 'passing':
        return True
    elif arg == 'failing':
        return False
    else:
        return float('nan')
student_data_bool = student_data[['reading_result','math_result']].applymap(res2bool)
student_data_bool.columns = ['r_bool','m_bool']
# use logical & to see students that passed both math and reading
student_data_bool['oall_bool'] = student_data_bool['r_bool']&student_data_bool['m_bool']
# student_data_bool.head()
```


```python
# rejoin booleans to student_data to compute stats after groupby
student_data = pd.merge(student_data,student_data_bool,left_index=True,right_index=True)
students_by_school = student_data.groupby('school')
student_performance = students_by_school[['math_score','reading_score']].mean()
student_performance.columns = ['Average Math Score','Average Reading Score']
```


```python
passing_rates_by_school = 100*students_by_school[['r_bool','m_bool','oall_bool']].sum()/students_by_school[['r_bool','m_bool','oall_bool']].count()
passing_rates_by_school.columns=['% Passing Reading', '% Passing Math', 'Overall Passing Rate']
# passing_rates_by_school.head()
```


```python
# combine dataframes
school_summary = pd.merge(school_data,student_performance,left_on='name',right_index=True)
school_summary = pd.merge(school_summary,passing_rates_by_school,left_on='name',right_index=True)
school_summary.rename(columns = {
    'size':'Total Students',
    'budget':'Total Budget',
    'name':'School Name',
    'type':'School Type'
    },inplace=True)
school_summary['Budget Per Student'] = school_summary['Total Budget']/school_summary['Total Students']
school_summary.head(15)
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
      <th>School ID</th>
      <th>School Name</th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Reading</th>
      <th>% Passing Math</th>
      <th>Overall Passing Rate</th>
      <th>Budget Per Student</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>Huang High School</td>
      <td>District</td>
      <td>2917</td>
      <td>1910635</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>81.316421</td>
      <td>65.683922</td>
      <td>53.513884</td>
      <td>655.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>Figueroa High School</td>
      <td>District</td>
      <td>2949</td>
      <td>1884411</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>80.739234</td>
      <td>65.988471</td>
      <td>53.204476</td>
      <td>639.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>Shelton High School</td>
      <td>Charter</td>
      <td>1761</td>
      <td>1056600</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>95.854628</td>
      <td>93.867121</td>
      <td>89.892107</td>
      <td>600.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>Hernandez High School</td>
      <td>District</td>
      <td>4635</td>
      <td>3022020</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>80.862999</td>
      <td>66.752967</td>
      <td>53.527508</td>
      <td>652.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>Griffin High School</td>
      <td>Charter</td>
      <td>1468</td>
      <td>917500</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>97.138965</td>
      <td>93.392371</td>
      <td>90.599455</td>
      <td>625.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5</td>
      <td>Wilson High School</td>
      <td>Charter</td>
      <td>2283</td>
      <td>1319574</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>96.539641</td>
      <td>93.867718</td>
      <td>90.582567</td>
      <td>578.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6</td>
      <td>Cabrera High School</td>
      <td>Charter</td>
      <td>1858</td>
      <td>1081356</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>97.039828</td>
      <td>94.133477</td>
      <td>91.334769</td>
      <td>582.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>7</td>
      <td>Bailey High School</td>
      <td>District</td>
      <td>4976</td>
      <td>3124928</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>81.933280</td>
      <td>66.680064</td>
      <td>54.642283</td>
      <td>628.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8</td>
      <td>Holden High School</td>
      <td>Charter</td>
      <td>427</td>
      <td>248087</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>96.252927</td>
      <td>92.505855</td>
      <td>89.227166</td>
      <td>581.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>9</td>
      <td>Pena High School</td>
      <td>Charter</td>
      <td>962</td>
      <td>585858</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>95.945946</td>
      <td>94.594595</td>
      <td>90.540541</td>
      <td>609.0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>10</td>
      <td>Wright High School</td>
      <td>Charter</td>
      <td>1800</td>
      <td>1049400</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>96.611111</td>
      <td>93.333333</td>
      <td>90.333333</td>
      <td>583.0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>11</td>
      <td>Rodriguez High School</td>
      <td>District</td>
      <td>3999</td>
      <td>2547363</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>80.220055</td>
      <td>66.366592</td>
      <td>52.988247</td>
      <td>637.0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>12</td>
      <td>Johnson High School</td>
      <td>District</td>
      <td>4761</td>
      <td>3094650</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>81.222432</td>
      <td>66.057551</td>
      <td>53.539172</td>
      <td>650.0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>13</td>
      <td>Ford High School</td>
      <td>District</td>
      <td>2739</td>
      <td>1763916</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>79.299014</td>
      <td>68.309602</td>
      <td>54.289887</td>
      <td>644.0</td>
    </tr>
    <tr>
      <th>14</th>
      <td>14</td>
      <td>Thomas High School</td>
      <td>Charter</td>
      <td>1635</td>
      <td>1043130</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>97.308869</td>
      <td>93.272171</td>
      <td>90.948012</td>
      <td>638.0</td>
    </tr>
  </tbody>
</table>
</div>



# Top Performing School (By Passing Rate)


```python
school_summary_sorted = school_summary.sort_values(by='Overall Passing Rate',ascending=False)
school_summary_sorted.head()
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
      <th>School ID</th>
      <th>School Name</th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Reading</th>
      <th>% Passing Math</th>
      <th>Overall Passing Rate</th>
      <th>Budget Per Student</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>6</th>
      <td>6</td>
      <td>Cabrera High School</td>
      <td>Charter</td>
      <td>1858</td>
      <td>1081356</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>97.039828</td>
      <td>94.133477</td>
      <td>91.334769</td>
      <td>582.0</td>
    </tr>
    <tr>
      <th>14</th>
      <td>14</td>
      <td>Thomas High School</td>
      <td>Charter</td>
      <td>1635</td>
      <td>1043130</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>97.308869</td>
      <td>93.272171</td>
      <td>90.948012</td>
      <td>638.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>Griffin High School</td>
      <td>Charter</td>
      <td>1468</td>
      <td>917500</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>97.138965</td>
      <td>93.392371</td>
      <td>90.599455</td>
      <td>625.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5</td>
      <td>Wilson High School</td>
      <td>Charter</td>
      <td>2283</td>
      <td>1319574</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>96.539641</td>
      <td>93.867718</td>
      <td>90.582567</td>
      <td>578.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>9</td>
      <td>Pena High School</td>
      <td>Charter</td>
      <td>962</td>
      <td>585858</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>95.945946</td>
      <td>94.594595</td>
      <td>90.540541</td>
      <td>609.0</td>
    </tr>
  </tbody>
</table>
</div>



# Bottom Performing Schools (By Passing Rate)


```python
school_summary_sorted.tail()
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
      <th>School ID</th>
      <th>School Name</th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Reading</th>
      <th>% Passing Math</th>
      <th>Overall Passing Rate</th>
      <th>Budget Per Student</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>12</th>
      <td>12</td>
      <td>Johnson High School</td>
      <td>District</td>
      <td>4761</td>
      <td>3094650</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>81.222432</td>
      <td>66.057551</td>
      <td>53.539172</td>
      <td>650.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>Hernandez High School</td>
      <td>District</td>
      <td>4635</td>
      <td>3022020</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>80.862999</td>
      <td>66.752967</td>
      <td>53.527508</td>
      <td>652.0</td>
    </tr>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>Huang High School</td>
      <td>District</td>
      <td>2917</td>
      <td>1910635</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>81.316421</td>
      <td>65.683922</td>
      <td>53.513884</td>
      <td>655.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>Figueroa High School</td>
      <td>District</td>
      <td>2949</td>
      <td>1884411</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>80.739234</td>
      <td>65.988471</td>
      <td>53.204476</td>
      <td>639.0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>11</td>
      <td>Rodriguez High School</td>
      <td>District</td>
      <td>3999</td>
      <td>2547363</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>80.220055</td>
      <td>66.366592</td>
      <td>52.988247</td>
      <td>637.0</td>
    </tr>
  </tbody>
</table>
</div>



# Math Scores by Grade


```python
math_by_school = pd.concat([student_data[['school','name']],
           student_data[['grade','math_score']].pivot(columns = 'grade',values = 'math_score')]
          , axis = 1).groupby(['school']).mean()[['9th','10th','11th','12th']]
math_by_school.head()
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
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>77.083676</td>
      <td>76.996772</td>
      <td>77.515588</td>
      <td>76.492218</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.094697</td>
      <td>83.154506</td>
      <td>82.765560</td>
      <td>83.277487</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.403037</td>
      <td>76.539974</td>
      <td>76.884344</td>
      <td>77.151369</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.361345</td>
      <td>77.672316</td>
      <td>76.918058</td>
      <td>76.179963</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>82.044010</td>
      <td>84.229064</td>
      <td>83.842105</td>
      <td>83.356164</td>
    </tr>
  </tbody>
</table>
</div>



# Reading Scores by Grade


```python
reading_by_school = pd.concat([student_data[['school','name']],
           student_data[['grade','reading_score']].pivot(columns = 'grade',values = 'reading_score')]
          , axis = 1).groupby(['school']).mean()[['9th','10th','11th','12th']]
reading_by_school.head()
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
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>81.303155</td>
      <td>80.907183</td>
      <td>80.945643</td>
      <td>80.912451</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.676136</td>
      <td>84.253219</td>
      <td>83.788382</td>
      <td>84.287958</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.198598</td>
      <td>81.408912</td>
      <td>80.640339</td>
      <td>81.384863</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>80.632653</td>
      <td>81.262712</td>
      <td>80.403642</td>
      <td>80.662338</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.369193</td>
      <td>83.706897</td>
      <td>84.288089</td>
      <td>84.013699</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Spending
Sort by spending values, include average scores, and percent passing rates


```python
bins = 4
#group_names = ['Small', 'Medium', 'Large']
school_summary['Spending'] = pd.cut(school_summary['Budget Per Student'],bins)
ss_by_spending = school_summary.groupby('Spending')
ss_by_spending.mean().sort_values(by = 'Budget Per Student', ascending = False).head().drop('School ID',axis = 1)
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
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Reading</th>
      <th>% Passing Math</th>
      <th>Overall Passing Rate</th>
      <th>Budget Per Student</th>
    </tr>
    <tr>
      <th>Spending</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>(635.75, 655.0]</th>
      <td>3376.428571</td>
      <td>2180875.00</td>
      <td>77.866721</td>
      <td>81.368774</td>
      <td>82.995575</td>
      <td>70.347325</td>
      <td>58.858741</td>
      <td>645.0</td>
    </tr>
    <tr>
      <th>(616.5, 635.75]</th>
      <td>3222.000000</td>
      <td>2021214.00</td>
      <td>80.199966</td>
      <td>82.425360</td>
      <td>89.536122</td>
      <td>80.036217</td>
      <td>72.620869</td>
      <td>626.5</td>
    </tr>
    <tr>
      <th>(597.25, 616.5]</th>
      <td>1361.500000</td>
      <td>821229.00</td>
      <td>83.599686</td>
      <td>83.885211</td>
      <td>95.900287</td>
      <td>94.230858</td>
      <td>90.216324</td>
      <td>604.5</td>
    </tr>
    <tr>
      <th>(577.923, 597.25]</th>
      <td>1592.000000</td>
      <td>924604.25</td>
      <td>83.455399</td>
      <td>83.933814</td>
      <td>96.610877</td>
      <td>93.460096</td>
      <td>90.369459</td>
      <td>581.0</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Size
Create bins of Small, Medium, Large


```python
bins = 3
group_names = ['Small', 'Medium', 'Large']
school_summary['Size'] = pd.cut(school_summary['Total Students'],bins,labels = group_names)
ss_by_size = school_summary.groupby('Size')
ss_by_size.mean().sort_values(by = 'Total Students', ascending = True).head().drop('School ID',axis = 1)
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
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Reading</th>
      <th>% Passing Math</th>
      <th>Overall Passing Rate</th>
      <th>Budget Per Student</th>
    </tr>
    <tr>
      <th>Size</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small</th>
      <td>1415.857143</td>
      <td>8.545616e+05</td>
      <td>83.502373</td>
      <td>83.883125</td>
      <td>96.593182</td>
      <td>93.585560</td>
      <td>90.410769</td>
      <td>602.571429</td>
    </tr>
    <tr>
      <th>Medium</th>
      <td>2722.000000</td>
      <td>1.719634e+06</td>
      <td>78.429493</td>
      <td>81.769122</td>
      <td>84.473577</td>
      <td>73.462428</td>
      <td>62.897703</td>
      <td>629.000000</td>
    </tr>
    <tr>
      <th>Large</th>
      <td>4592.750000</td>
      <td>2.947240e+06</td>
      <td>77.063340</td>
      <td>80.919864</td>
      <td>81.059691</td>
      <td>66.464293</td>
      <td>53.674303</td>
      <td>641.750000</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Type


```python
ss_by_type = school_summary.groupby(['School Type'])
ss_by_type.mean().drop('School ID', axis = 1)
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
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Reading</th>
      <th>% Passing Math</th>
      <th>Overall Passing Rate</th>
      <th>Budget Per Student</th>
    </tr>
    <tr>
      <th>School Type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Charter</th>
      <td>1524.250000</td>
      <td>9.126881e+05</td>
      <td>83.473852</td>
      <td>83.896421</td>
      <td>96.586489</td>
      <td>93.620830</td>
      <td>90.432244</td>
      <td>599.500000</td>
    </tr>
    <tr>
      <th>District</th>
      <td>3853.714286</td>
      <td>2.478275e+06</td>
      <td>76.956733</td>
      <td>80.966636</td>
      <td>80.799062</td>
      <td>66.548453</td>
      <td>53.672208</td>
      <td>643.571429</td>
    </tr>
  </tbody>
</table>
</div>


