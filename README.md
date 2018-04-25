

```python
# dependencies

import numpy as np
import pandas as pd
import datetime as dt
import matplotlib.pyplot as plt

import sqlalchemy
from sqlalchemy import create_engine, inspect
from sqlalchemy.orm import Session
from sqlalchemy.ext.automap import automap_base
from sqlalchemy import Column, Integer, String, Float, and_, Date, func, desc
```


```python
# collect the start and end dates of the trip
date_entry1 = "2017-07-01"
year, month, day = map(int, date_entry1.split('-'))
start_date = dt.date(year, month, day)

date_entry2 = "2017-07-07"
year, month, day = map(int, date_entry2.split('-'))
end_date = dt.date(year, month, day)
```


```python
# create engine using the `hawaii.sqlite` database file
engine = create_engine("sqlite:///hawaii.sqlite")
conn = engine.connect()
```


```python
# Use `engine.execute` to select and display the first 10 rows from the emoji table
### BEGIN SOLUTION
engine.execute('SELECT * FROM stations LIMIT 10').fetchall()
### END SOLUTION
```




    [(1, 'USC00519397', 'WAIKIKI 717.2, HI US', 21.2716, -157.8168, 3.0),
     (2, 'USC00513117', 'KANEOHE 838.1, HI US', 21.4234, -157.8015, 14.6),
     (3, 'USC00514830', 'KUALOA RANCH HEADQUARTERS 886.9, HI US', 21.5213, -157.8374, 7.0),
     (4, 'USC00517948', 'PEARL CITY, HI US', 21.3934, -157.9751, 11.9),
     (5, 'USC00518838', 'UPPER WAHIAWA 874.3, HI US', 21.4992, -158.0111, 306.6),
     (6, 'USC00519523', 'WAIMANALO EXPERIMENTAL FARM, HI US', 21.33556, -157.71139, 19.5),
     (7, 'USC00519281', 'WAIHEE 837.5, HI US', 21.45167, -157.84888999999995, 32.9),
     (8, 'USC00511918', 'HONOLULU OBSERVATORY 702.2, HI US', 21.3152, -157.9992, 0.9),
     (9, 'USC00516128', 'MANOA LYON ARBO 785.2, HI US', 21.3331, -157.8025, 152.4)]




```python
# Use `engine.execute` to select and display the first 10 rows from the emoji table
### BEGIN SOLUTION
engine.execute('SELECT * FROM measurements LIMIT 10').fetchall()
### END SOLUTION
```




    [(1, 'USC00519397', '2010-01-01', 0.08, 65.0),
     (2, 'USC00519397', '2010-01-02', 0.0, 63.0),
     (3, 'USC00519397', '2010-01-03', 0.0, 74.0),
     (4, 'USC00519397', '2010-01-04', 0.0, 76.0),
     (5, 'USC00519397', '2010-01-07', 0.06, 70.0),
     (6, 'USC00519397', '2010-01-08', 0.0, 64.0),
     (7, 'USC00519397', '2010-01-09', 0.0, 68.0),
     (8, 'USC00519397', '2010-01-10', 0.0, 73.0),
     (9, 'USC00519397', '2010-01-11', 0.01, 64.0),
     (10, 'USC00519397', '2010-01-12', 0.0, 61.0)]




```python
# Declare a Base using `automap_base()`
Base = automap_base()
```


```python
# Use the Base class to reflect the database tables
Base.prepare(engine, reflect=True)
```


```python
# Create a session
session = Session(bind=engine)
```


```python
# Print all of the classes mapped to the Base
Base.classes.keys()
```




    ['measurements', 'stations']




```python
Station = Base.classes.stations
Measurement = Base.classes.measurements
session = Session(bind=engine)
```


```python
# Precipitation Analysis

# get the date 1 year ago
year_before = end_date - dt.timedelta(365)

pcp_year = session.query(Measurement.date, Measurement.prcp).filter(and_(Measurement.date <= end_date, Measurement.date >= year_before)).all()

# use the date as index
prcp_df = pd.DataFrame(pcp_year, columns=["date", "precipitation"])
prcp_df.set_index('date', inplace=True)
prcp_df.iloc[::-1].plot(title="Precipitation from %s to %s" % (year_before, end_date))
plt.xlabel("Date")
plt.ylabel("Precipitation")
plt.title("Precipitation from %s to %s" %(year_before,end_date))
plt.xticks(rotation='45')
plt.show()

```


![png](output_10_0.png)



```python
# give summary statistics of the dataframe
prcp_df.describe()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>precipitation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2058.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>0.207585</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.568348</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>0.020000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>0.150000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>9.640000</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Station Analysis
# calculate the number of stations
num_stations = session.query(Station.station).count()
print("Station count: is %s" % num_stations)
```

    Station count: is 9
    


```python
# find the most active station
act_stations = session.query(Measurement.station, func.count(Measurement.date)).group_by(Measurement.station).order_by(desc(func.count(Measurement.date))).all()

most_active = act_stations[0][0]
num_observations = act_stations[0][1]

print("The most active station: %s" % most_active)
print("The number of observations for station %s: %s" % (most_active, num_observations))
```

    The most active station: USC00519281
    The number of observations for station USC00519281: 2772
    


```python
# get the last 12 months of temperature observation data

temp_year = session.query(Measurement.date, Measurement.tobs).filter(and_(Measurement.date <= end_date,
                                                                        Measurement.date >= year_before,
                                                                         Measurement.station == most_active)).all()

# create the dataframe and set date as the index
temp_df = pd.DataFrame(temp_year, columns = ["date", "temperature"])
temp_df.set_index('date', inplace=True)
```


```python
# create a histogram of the temperature data with bins=12
temp_df.iloc[::-1].plot.hist(title="Temperature from %s to %s" % (year_before, end_date), bins = 12)
plt.tight_layout()
plt.show()

#Temperature Analysis
```


![png](output_15_0.png)



```python
#Temperature Analysis

def calc_temps(start, end):
    # calculate the date 1 year ago
    prior_year = end - dt.timedelta(365)
    
    # get the maximum temperature
max_temp = session.query(func.max(Measurement.tobs)).filter(and_(Measurement.date <= end_date, Measurement.date >= year_before)).all()
maximum = max_temp[0][0]
    
    # get the minimum temperature
min_temp = session.query(func.min(Measurement.tobs)).filter(and_(Measurement.date <= end_date, Measurement.date >= year_before)).all()
minimum = min_temp[0][0]

    # get the average temperature
avg_temp = session.query(func.avg(Measurement.tobs)).filter(and_(Measurement.date <= end_date, Measurement.date >= year_before)).all()
average = avg_temp[0][0]
    

```


```python
 # create the plot
objects = [str(end_date)]
x_axis = np.arange(len(objects))
fig, ax = plt.subplots()
temp_plot = ax.bar(x_axis, average, yerr=(maximum-minimum), color = "orange", alpha = .5, width = .5)
tick_locations = [value for value in x_axis]
plt.xticks(tick_locations, [])
plt.xlim(-1, len(x_axis))
plt.title("Trip Avg Temp")
plt.ylabel("Temp (F)")

plt.show()

# call the function
calc_temps(start_date, end_date)
```


![png](output_17_0.png)

