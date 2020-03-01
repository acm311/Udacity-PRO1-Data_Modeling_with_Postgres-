## Project: Sparkify ETL

This is the first project for the Data Engineering Nanodegree, in which we put in practice concepts like:
- Data modeling using Postgres
- Star schema
- ETL pipeline with Python

### Context

A startup called Sparkify wants to analyze the data they've been collecting on songs and user activity on their new music streaming app. The analytics team is particularly interested in understanding what songs users are listening to. Currently, they don't have an easy way to query their data, which resides in a directory of JSON logs on user activity on the app, as well as a directory with JSON metadata on the songs in their app.

The objective is create a database model and put in place a pipeline that allows the company do data analysis.

This is the current state of the data:

<table style="width:100%">
  <tr>
    <th>Dataset type</th>
    <th>Location example</th>
    <th>Content example</th>
  </tr>
  <tr>
    <td>Song</td>
    <td>song_data/A/B/C/TRABCEI128F424C983.json</br>song_data/A/A/B/TRAABJL12903CDCF1A.json</td>
    <td>{"num_songs": 1, "artist_id": "ARJIE2Y1187B994AB7", "artist_latitude": null, "artist_longitude": null, "artist_location": "", "artist_name": "Line Renaud", "song_id": "SOUPIRU12A6D4FA1E1", "title": "Der Kleine Dompfaff", "duration": 152.92036, "year": 0}</td>
  </tr>
  <tr>
    <td>Log</td>
    <td>log_data/2018/11/2018-11-12-events.json</br>log_data/2018/11/2018-11-13-events.json</td>
    <td>{"artist":"Radiohead","auth":"Logged In","firstName":"Tegan","gender":"F","itemInSession":16,"lastName":"Levine","length":223.00689,"level":"paid","location":"Portland-South Portland, ME","method":"PUT","page":"NextSong","registration":1540794356796.0,"sessionId":1065,"song":"Sulk","status":200,"ts":1543527265796,"userAgent":"\"Mozilla\/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit\/537.36 (KHTML, like Gecko) Chrome\/36.0.1985.143 Safari\/537.36\"","userId":"80"}</td>
  </tr>
</table>

### Database Schema
In order to response to the business needs we're using a star schema, that gives to us the advantage to use simply queries and joins and do fast aggregations... way to do analytics!

The schema is compose by 4 dimentional tables and one fact table.

#### Fact Table
**songplays** - records in log data associated with song plays i.e. records with page NextSong
- songplay_id, start_time, user_id, level, song_id, artist_id, session_id, location, user_agent

#### Dimension Tables

**users** - users in the app
- user_id, first_name, last_name, gender, level

**songs** - songs in music database
- song_id, title, artist_id, year, duration

**artists** - artists in music database
- artist_id, name, location, latitude, longitude

**time** - timestamps of records in **songplays** broken down into specific units
- start_time, hour, day, week, month, year, weekday

### ETL Pipeline

<ol>
  <li>Connect to sparkifydb database</li>
  <li>Grab all the *json files from the path data/song_data</li>
  <ol>
    <li>For each file...
      <ol>
        <li>Read the file (as a dataframe) using read_json pandas function and for each line...
          <ol>
            <li>Obtain the song_id, title, artist_id, year, duration from the dataframe</li>
            <li>Insert the info on the song table</li>
            <li>Obtain the artist_id, artist_name, artist_location, artist_latitude, artist_longitude</li>
            <li>Insert the info to the artist table</li>
          </ol>
        </li>          
      </ol>
    </li>  
  </ol>
  <li>Grab all the *json files from the path data/log_data</li>
  <ol>
    <li>For each file...
      <ol>
        <li>Read the file (as a dataframe) using read_json pandas function where page = 'NextSong'</li>
        <li>For ts series, convert to timestamp and from there obtain the fields start_time, hour, day, week, month, year, weekday</li>
        <li>Populate the time table</li>
        <li>Take userId, firstName, lastName, gender, level series and populate user table</li>
        <li>With the dataframe obtain in the step 3.i.a and for each line...
          <ol>
            <li>Get songid and artistid from song and artist tables</li>
            <li>Insert the info to the songplay table</li>
          </ol>
         </li>
      </ol>
     </li>  
  </ol> 
  
  <li>Close the database connection</li>
</ol>