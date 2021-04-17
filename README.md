# Data Storage and Retrieval – Exploring Weather Data
## Overview
### *Purpose*
A friend is interested in opening a Surf n’ Shake shop in Oahu, Hawaii, and reached out to an investor, W. Avy. While the first meeting went well, W. Avy wanted some analyses done on a weather dataset he had to see how the weather may impact the success of the shop.

## Resources
-	Data Source: Hawaii.sqlite
-	Software: Python 3.6.1, Jupyter Notebook 6.2.0, Visual Studio Code 1.38.1

## Analysis
### *Reflect Tables into SQLAlchemy ORM*
To start my analysis, I imported style from the Matplotlib dependency and specified the fivethirtyeight style to use. Then, I added the pyplot module:
```
%matplotlib inline
from matplotlib import style
style.use('fivethirtyeight')
import matplotlib.pyplot as plt
```
In a new cell, I imported the NumPy and Pandas dependencies with the following code:
```
import numpy as np
import pandas as pd
```
Then, I imported the datetime dependency:
```
import datetime as dt
```
Finally, I imported the SQLAlchemy dependency, in addition to the dependencies for automap, session, create_engine, and func:
```
# Python SQL toolkit and Object Relational Mapper
import sqlalchemy
from sqlalchemy.ext.automap import automap_base
from sqlalchemy.orm import Session
from sqlalchemy import create_engine, func
```
Once SQLAlchemy was imported, I created an engine:
```
# Create engine
engine = create_engine("sqlite:///hawaii.sqlite")
```
Next, I created a base class for an automap schema in SQLAlchemy and reflected tables with the prepare() function:
```
# Reflect an existing database into a new model
Base = automap_base()

# Reflect the tables
Base.prepare(engine, reflect=True)
```
To confirm that the Automap was able to find all of the data in the SQLite database, I used the following code:
```
# View all of the classes that automap found
Base.classes.keys()
```
After running the code, I received the following output for classes: ['measurement', 'station'].

After viewing the classes, I created references to each table:
```
# Save references to each table
Measurement = Base.classes.measurement
Station = Base.classes.station
```
With the references saved to some variables, I created a session link to the database:
```
# Create our session (link) from Python to the DB
session = Session(engine)
```

### *Precipitation Analysis*
W. Avy asked for an exploratory analysis of the precipitation data he had collected, specifically looking at a full year of data starting from August 23, 2017. I created a variable called prev_year and used the dt.date() function and the dt.timedelta() function from the datetime dependency to trace back one year from August 23, 2017:
```
# Design a query to retrieve the last 12 months of precipitation data and plot the results. 
# Calculate the date one year from the last date in data set.
prev_year = dt.date(2017, 8, 23) - dt.timedelta(days=365) 
```
To retrieve the precipitation scores, I created a variable called results to store the results of the subsequent query. Then, I added my session using the session.query() function so that I could query the database. Within the session.query(), I included two parameters to reference the Measurement table: Measurement.data and Measurement.prcp. Next, I used the filter() function to filter out the data that was older than a year from the last record date. Lastly, I added .all() to the end of my query to extract all the results to a list:
```
# Perform a query to retrieve the data and precipitation scores
results = []
results = session.query(Measurement.date, Measurement.prcp).filter(Measurement.date >= prev_year).all()
print(results)
```
In order to access the weather results saved in the future, I saved the results into a Pandas DataFrame:
```
# Save the query results as a Pandas DataFrame and set the index to the date column
df = pd.DataFrame(results, columns=['date','precipitation'])
```
Then, I set the index column of the DataFrame to the date column using the set_index() function:
```
df.set_index(df['date'], inplace=True)
```
As the date was used as the index, there were two date columns within the DataFrame. So, I printed the DataFrame without the index to show only the date and precipitation by converting the DataFrame to strings and setting the index to “False”:
```
print(df.to_string(index=False))
```
Next, I sorted the values by date using the sort_index() function:
```
# Sort the dataframe by date
df = df.sort_index()
print(df.to_string(index=False))
```
Using Matplotlib, I plotted the results of the precipitation analysis:
```
# Use Pandas Plotting with Matplotlib to plot the data
df.plot()
```

![precipitation.png](https://github.com/kcharb7/Surfs_up/blob/main/Resources/precipitation.png)

This plot showcased that some months had higher amounts of precipitation each day than other months.

Additionally, I calculated summary statistics using the describe() function:
```
# Use Pandas to calculate the summary statistics for the precipitation data
df.describe()
```

![summary_statistics.png](https://github.com/kcharb7/Surfs_up/blob/main/Resources/summary_statistics.png)

Precipitation was observed 2,021 times. The mean precipitation was 0.18mm (SD = 0.46), with the minimum at 0.00mm and the maximum at 6.7mm. 

### *Weather Station Analysis*
W.Avy additionally wished to know how many stations were used to collect the weather information to ensure enough stations were used to make the information valid. To get the number of stations in the dataset, I used my session to query the database and the func.count with a reference for Station.station to get the count of the stations. Then, I added the .all() function to the end of the query to return the results as a list:
```
# How many stations are available in this dataset?
session.query(func.count(Station.station)).all()
```
There were 9 stations from which the precipitation data was collected. To determine how much data was collected from each station, I additionally determined how active the stations were. I created another query with the parameters to list the stations and the counts. I grouped the data by station name using the group_by() function and ordered the count for each station in descending order. The .all() function was added to return the results to a list:
```
# What are the most active stations?
# List the stations and the counts in descending order.
session.query(Measurement.station, func.count(Measurement.station)).\
group_by(Measurement.station).order_by(func.count(Measurement.station).desc()).all()
```

![Station_counts.png](https://github.com/kcharb7/Surfs_up/blob/main/Resources/Station_counts.png)

After providing W. Avy with the station counts, he stated interest in the most active station and believed it would provide the most data to help determine the best location for the surf shop. To investigate the location further, I gathered some basic statistics from the most activate station on the minimum, maximum, and average temperatures. To find these values, I used the func.min, func.max, and func.avg functions. I added a filter for the most active station, USC00519281:
```
# Using the station id from the previous query, calculate the lowest temperature recorded, 
# highest temperature recorded, and average temperature most active station
session.query(func.min(Measurement.tobs), func.max(Measurement.tobs), func.avg(Measurement.tobs)).\
filter(Measurement.station == 'USC00519281').all()
```
The average temperature was about 71.7 degrees, the minimum temperature was 54 degrees, while the maximum temperature was 85 degrees. 

Next, I created a plot showcasing the temperatures in a given year for the station with the highest number of temperature observations. To get the total observations count, I pulled the Measurement.tobs and filtered the stations to only the most active station. I added an additional filter to consider only the most recent year:
```
# Choose the station with the highest number of temperature observations.
# Query the last 12 months of temperature observation data for this station
results = session.query(Measurement.tobs).\
filter(Measurement.station == 'USC00519281').\
filter(Measurement.date >= prev_year).all()

print(results)
```
To make the results easier to read and interpret, I placed them in a DataFrame:
```
df = pd.DataFrame(results, columns=['tobs'])
print(df)
```
I created a histogram from the temperature observations using the plot() and hist() functions and added the number of bins as a parameter. I chose to divide the temperature observations into 12 bins. I additionally added plt.tight_layout() to compress the x-axis labels to fit into the box holding the plot:
```
#Plot the temperature observations as a histogram
df.plot.hist(bins=12)
plt.tight_layout()
```

![temp_histogram.png](https://github.com/kcharb7/Surfs_up/blob/main/Resources/temp_histogram.png)

Looking at the histogram it was apparent that a vast majority of temperature observations were above 67 degrees. 

### *Create a Climate App Using Flask*
To show the results to the board of directors in a webpage, I used Flask to create five routes, one for each segment of my analysis (i.e., Precipitation, Stations, Monthly Temperature, and Statistics), as well as a welcome route orienting W. Avy and his associates to the webpage. To start, I created a new Python file called app.py in VS Code and imported my dependencies:
```
# Import dependencies
import datetime as dt
import numpy as np
import pandas as pd
import sqlalchemy
from sqlalchemy.ext.automap import automap_base
from sqlalchemy.orm import Session
from sqlalchemy import create_engine, func
from flask import Flask, jsonify
```
Then, I set up my database engine for the Flask application, as well as reflected the database and tables:
```
# Create engine
engine = create_engine("sqlite:///hawaii.sqlite")
# Reflect database into a new model
Base = automap_base()
# Reflect the tables
Base.prepare(engine, reflect=True)
```
After the database was reflected, I saved the references to each table as was done previously:
```
# Save the references to each table
Measurement = Base.classes.measurement
Station = Base.classes.station
```
Next, I created a session link from Python to the database:
```
# Create our session (link) from Python to the DB
session = Session(engine)
```
To define my Flask app, I used the following code:
```
# Define Flask app
app = Flask(__name__)
```
The Welcome route was the first route created and was set as the root so that it could be the entryway to the rest of the analysis. I created a function welcome() with a return statement that would provide f-strings as a reference to all of the other routes:
```
 # Create the Welcome route
@app.route('/')
def welcome():
    return(
    '''
    Welcome to the Climate Analysis API!
    Available Routes:
    /api/v1.0/precipitation
    /api/v1.0/stations
    /api/v1.0/tobs
    /api/v1.0/temp/start/end
    ''')
```
After creating the Welcome route, I moved on to creating the Precipitation route. To create the route, I added the following code, along with a new precipitation() function:
```
# Create Precipitation route
@app.route("/api/v1.0/precipitation")
def precipitation():
    return
```
Within the precipitation() function, I added the line of code that calculated the date one year ago from the most recent date in the database:
```
def precipitation():
    prev_year = dt.date(2017, 8, 23) - dt.timedelta(days=365)
    return
```
Then, I added the query to get the date and precipitation for the previous year:
```
def precipitation():
    prev_year = dt.date(2017, 8, 23) - dt.timedelta(days=365)
    precipitation = session.query(Measurement.date, Measurement.prcp).\
      filter(Measurement.date >= prev_year).all()
    return
```
I created a dictionary with the data as the key and precipitation as the value and used the jsonify() function to format the results into a JSON structured file:
```
def precipitation():
    prev_year = dt.date(2017, 8, 23) - dt.timedelta(days=365)
    precipitation = session.query(Measurement.date, Measurement.prcp).\
      filter(Measurement.date >= prev_year).all()
    precip = {date: prcp for date, prcp in precipitation}
    return jsonify(precip)
```
Once the Precipitation route was complete, I created a third route for the stations analysis:
```
# Create Stations route
@app.route("/api/v1.0/stations")
```
After defining the route, I created a new function called stations():
```
def stations():
    return
```
Then, I created a query to retrieve all of the stations in the database:
```
def stations():
    results = session.query(Station.station).all()
    return
```
I used the np.ravel() function to unravel the results into a one-dimensional array. The unraveled results were then converted into a list. I additionally used the jsonify() function to return the list as JSON:
```
def stations():
    results = session.query(Station.station).all()
    stations = list(np.ravel(results))
    return jsonify(stations=stations)
```
I created a fourth route for the temperature observations and a new function called temp_monthly():
```
# Create Monthly Temperature route
@app.route("/api/v1.0/tobs")
def temp_monthly():
    return
```
Within the temp_monthly() function, I added the code to calculate the date one year ago from the last date in the database:
```
def temp_monthly():
    prev_year = dt.date(2017, 8, 23) - dt.timedelta(days=365)
    return
```
Next, I added the code to query the primary station for all the temperature observations from the last year:
```
def temp_monthly():
    prev_year = dt.date(2017, 8, 23) - dt.timedelta(days=365)
    results = session.query(Measurement.tobs).\
        filter(Measurement.station == 'USC00519281').\
        filter(Measurement.date >= prev_year).all()
    return
```
Finally, I unravelled the results into a one-dimensional array, converted that array into a list, and then jsonified the temps list:
```
def temp_monthly():
    prev_year = dt.date(2017, 8, 23) - dt.timedelta(days=365)
    results = session.query(Measurement.tobs).\
      filter(Measurement.station == 'USC00519281').\
      filter(Measurement.date >= prev_year).all()
    temps = list(np.ravel(results))
    return jsonify(temps=temps)
```
A fifth and final route was created to report the minimum, maximum, and average temperature. For this route, I added both a starting and ending date:
```
# Create a Statistics route
@app.route("/api/v1.0/temp/<start>")
@app.route("/api/v1.0/temp/<start>/<end>")
```
Then, I created a new function called stats() with a start parameter and an end parameter both set to “None”:
```
def stats(start=None, end=None):
     return
```
Within the stats() function, I created a list to select the minimum, maximum, and average temperatures from the SQLite database:
```
def stats(start=None, end=None):
    sel = [func.min(Measurement.tobs), func.avg(Measurement.tobs), func.max(Measurement.tobs)]
```
This list was included within a query after an if-not statement that was added to determine a start and end date. Code was additionally added to unravel the results into a one-dimensional array, convert them to a list, and subsequently jsonify the results:
```
def stats(start=None, end=None):
    sel = [func.min(Measurement.tobs), func.avg(Measurement.tobs), func.max(Measurement.tobs)]

    if not end:
        results = session.query(*sel).\
            filter(Measurement.date >= start).\
            filter(Measurement.date <= end).all()
        temps = list(np.ravel(results))
        return jsonify(temps=temps)
```
Then, I created a second query to calculate the temperature minimum, maximum, and average with the start and end dates using the sel list:
```
def stats(start=None, end=None):
    sel = [func.min(Measurement.tobs), func.avg(Measurement.tobs), func.max(Measurement.tobs)]

    if not end:
        results = session.query(*sel).\
            filter(Measurement.date >= start).\
            filter(Measurement.date <= end).all()
        temps = list(np.ravel(results))
        return jsonify(temps)

    results = session.query(*sel).\
        filter(Measurement.date >= start).\
        filter(Measurement.date <= end).all()
    temps = list(np.ravel(results))
    return jsonify(temps=temps)
```

# Challenge
## Overview
### *Purpose*
As a potential investor in a Surf n’ Shake shop in Oahu, Hawaii, W. Avy has been very concerned over the impact the weather in this location may have on the business. After providing my initial weather analysis on the precipitation, most active weather stations, temperature observations, and summary statistics for the previous year, W. Avy has asked for more information on temperature data in Oahu for the specific months of June and December to determine if the surf and ice cream shop business would be sustainable year-round.

## Analysis
### *Summary Statistics for June*
To calculate the summary statistics of temperatures in June in Oahu, I created a query that filtered the date column from the Measurement table using the extract() function from sqlalchemy and converted the results to a list:
```
# Import the sqlalchemy extract function.
from sqlalchemy import extract

# Write a query that filters the Measurement table to retrieve the temperatures for the month of June. 
# Convert the June temperatures to a list.
June_temps = session.query(Measurement.tobs).\
filter(extract('month', Measurement.date) == 6).all()
```
Then, I created a DataFrame from the list of June temperatures:
```
# Create a DataFrame from the list of temperatures for the month of June. 
June_df = pd.DataFrame(June_temps, columns=['June Temps'])
print(June_df)
```
To generate the summary statistics for the June temperatures DataFrame, I used the describe() method:
```
# Calculate and print out the summary statistics for the June temperature DataFrame.
June_df.describe()
```

### *Summary Statistics for December*
I used the same code as above to generate the summary statistics for December, altering the extract function to only include the month December:
```
# Write a query that filters the Measurement table to retrieve the temperatures for the month of December.
# Convert the December temperatures to a list.
Dec_temps = session.query(Measurement.tobs).\
filter(extract('month', Measurement.date) == 12).all()

# Create a DataFrame from the list of temperatures for the month of December. 
December_df = pd.DataFrame(Dec_temps, columns=['December Temps'])
print(December_df)

# Calculate and print out the summary statistics for the December temperature DataFrame.
December_df.describe()
```

## Results 

June Summary Statistics

![June_Temps.png](https://github.com/kcharb7/Surfs_up/blob/main/Resources/June_Temps.png)

December Summary Statistics 

![December_Temps.png](https://github.com/kcharb7/Surfs_up/blob/main/Resources/December_Temps.png)
-	The average temperature for June and December did not greatly differ, as June had an average temperature of 74.9 degrees and December had an average temperature of 71.0 degrees. 
-	Maximum temperatures in June and December similarly did not differ greatly. June had a maximum temperature of 85.0 degrees, while December had a maximum temperature of 83.0 degrees.
-	The months of June and December varied in their minimum temperatures. June had a minimum temperature of 64.0 degrees, while December had a much lower minimum temperature of 56.0 degrees.
-	Temperature was collected more often in June than December, with June having 1,700 temperature counts and December having 1,517 temperature counts.

## Summary
The average and maximum temperatures did not greatly differ between June and December; however, June had a higher minimum temperature by 8 degrees compared to December. Additionally, more temperatures were collected in June versus December. 

While average temperatures did not greatly differ between June and December, it is important to determine how much precipitation occurred in each month, as this would impact business as well. To determine the average precipitation for each month, I used the same code as above, selecting the prcp column instead of the tobs column:
```
# Determine the precipitation in June
June_prcp = session.query(Measurement.prcp).\
filter(extract('month', Measurement.date) == 6).all()
June_prcp_df = pd.DataFrame(June_prcp, columns=['June Precipitation'])
June_prcp_df.describe()
```

![June_prcp.png](https://github.com/kcharb7/Surfs_up/blob/main/Resources/June_prcp.png)

```
# Determine the precipitation in December
Dec_prcp = session.query(Measurement.prcp).\
filter(extract('month', Measurement.date) == 12).all()
Dec_prcp_df = pd.DataFrame(Dec_prcp, columns=['December Precipitation'])
Dec_prcp_df.describe()
```

![Dec_prcp.png](https://github.com/kcharb7/Surfs_up/blob/main/Resources/Dec_prcp.png)


The precipitation levels for the months of June and December only differed by 0.08mm. Though, December’s maximum precipitation was 2mm higher than June’s. 
