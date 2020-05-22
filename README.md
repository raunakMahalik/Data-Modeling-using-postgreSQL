# Data Modeling with PostgreSQL


## Introduction

The music streaming app start-up, Sparkify, has collected copious amount of song and user activity data. The analytics team wants to use this data and deliver insights which can drive business decisions and improve user experience. Currently, they do not have an efficient way to achieve this as the data resides in a directory of JSON logs on user activity and a  JSON metadata on the songs on the app. The aim of this project is to give meaningful structure to the data and create a relational database of songs and user activity which supports efficient querying and analytical functions. The name of this database is **`sparkifydb`**


## Database Schema

The analytics team at Sparkify is interested in understanding what songs the users listen to frequently. Given this use case, a star schema implementation which is attuned to this requirement is implemented. 

A relational database model is ideal for this use case as:

- The data is well structured and can be represented in a tabular format.
- The songs and log data presented are not so large that they mandate the use of big data solutions like NoSQL.
- The insights required by the analytics team are simple and can be achieved by using SQL queries.
- The ability to perform JOINs is something that is relevant to this scenario.

Below is a description of the fact and dimension tables :

### Fact Table

A single fact table called **`songplays`** is implemented whose records hold log data associated with song plays. It has the following attributes:

| Attribute   | Data Type | Notes                              |
|-------------|-----------|------------------------------------|
| songplay_id | serial    | auto-incremented (*Primary Key* )  |
| start_time  | time      | timestamp of App usage by user     |
| user_id     | int       | unique integer ID of user          |
| level       | varchar   | user subscription (paid/free)      |
| song_id     | varchar   | song ID of the requested song      |
| artist_id   | varchar   | artist ID of the requested song.   |
| session_id  | int       | Browser session ID.                |
| location    | varchar   | user location                      |
| user_agent  | varchar   | Browser and OS details of the user |

### Dimension Tables

Four dimension tables are created to capture the attributes corresponding to the different data elements. Below is a brief description of the same :

#### 1. **`users`**

This table captures user information like age, gender, name, etc. It has the following attributes:

| Attribute  | Data Type | Notes        |
|------------|-----------|--------------|
| user_id    | int       | *Primary Key * |
| first_name | varchar   | first name of user   |
| last_name  | varchar   | last name of user  |
| gender     | varchar   |  M/F   |
| level      | varchar   | paid/free |

#### 2. **`songs`**

The songs table holds all the information pertaining to songs available in the music streaming app like artist name, duration, year of release etc. It has the following attributes :

| Attribute | Data Type | Notes                |
|-----------|-----------|----------------------|
| song_id   | varchar   |  unique alpha-numeric ID  of song (*Primary Key*) |
| title     | varchar   |  name of the song   |
| artist_id | varchar   |  |
| year      | int       | release year         |
| duration  | numeric   | length of song in ms |

#### 3. **`artists`**

Just like the songs table, the artists table holds all the information pertaining to the artists like name, location, etc. This table has the following attributes :

| Attribute | Data Type | Notes                                              |
|-----------|-----------|----------------------------------------------------|
| artist_id | varchar   | unique alpha-numeric ID  of artist (*Primary Key*) |
| name      | varchar   | name of artist                                     |
| location  | varchar   | Location of artist                                 |
| latitude  | numeric   | latitude coordinate of the location                |
| longitude | numeric   | longitude coordinate of the location               |

#### 4. **`time`**

The timestamps of records in the songplays table are broken down into specific units and are contained in the time table. The following are the attributes of this table:

| Attribute  | Data Type | Notes                                  |
|------------|-----------|----------------------------------------|
| start_time | time      | time stamp of App usage (*Primary Key*)|
| hour       | int       | Hour of the day (00 - 23)              |
| day        | int       | Day of the week (1-7)                  |
| week       | int       | Week number (1-52)                     |
| month      | int       | Month number (1-12)                    |
| year       | int       | Year of activity                       |

## ETL Pipeline

### STEP 1:

To populate the  **`songs`** and  **`artists`** tables, the songs meta data (JSON files) present in path *data/song_data* are used. Each JSON file is  read and stored as a dataframe whose columns are used to populate the tables as follows:

-  The **artist_id, artist_name, artist_location, artist_latitude **and** artist_longitude** columns of the dataframe are used to populate the **`artists`** table.

- The **song_id, title, artist_id, year **and** duration** columns of the dataframe are used to populate the **`songs`** table.


### STEP 2:

In order to fill the **`time`**, **`users`** and **`songplays`** tables, the user activity logs (JSON files) residing at *data/song_data* are used. Each JSON log file is read and stored as a dataframe. Only the records having page action value NextSong A few transformations are performed on the data present in these intermediate dataframes before injecting the data into the database. Below is a brief description of the transformations performed :

- The **ts**  column of the dataframe contains timestamps in milliseconds, hence the [to_datetime](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.to_datetime.html) pandas API is used to break it into time, hour, day, week, month and year components. Each of these are then used to populate the attributes of the **`time`** table.

- The **userId, firstName, lastName, gender **and** level** columns of the dataframe are used to populate the **`users`** table. The logs contain multiple instances for different users, hence the duplicates are first removed from the dataframe and only then the data is filled into the respective tables.

- The transformations performed to fill the **`songplays`** table are a little bit more involved than the former transformations. As the log files do not contain the **artist_id** and **song_id**, these values were fetched by performing a matrix multiplication (a JOIN can be used as well) between the **`songs`** and  **`artists`** tables. The values obtained along with the columns values of **ts** (after transforming it using the [Timestamp](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Timestamp.html) pandas API)**, userId, level, sessionId, location** and** userAgent** are then used to fill the **`songplays`** table.


## File Structure  and usage

Along with the logs and song data contained in the ***data*** folder, the following files are used in this project:

| File name        | Description                                        |
|------------------|----------------------------------------------------|
| create_tables.py | Creates and drops all the databases.               |
| etl.ipynb        | Performs ETL operations on a single JSON file.     |
| etl.py           | Performs ETL operations on all the JSON files.     |
| README.md        | Contains the project description.                  |
| sql_queries.py   | Contains SQL queries like CREATE, DROP and INSERT. |
| test.ipynb       | Prints outputs of sample queries.                  |


To use the project the following steps must be performed:
1. Run the **create_tables.py** file by using the command : `python3 create_tables.py`.
2. Run the **etl.py** file by using the command : `python3 etl.py`
3. Test the sample queries by using the **test.ipynb** file.

It should be noted that one must restart the kernel after using the **test.ipynb** or **etl.ipynb** files before using any other file. This will close the connection to `sparkifydb` database.