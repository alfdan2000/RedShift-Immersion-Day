# RedShift-Immersion-Day

Lab 2.4 

Working with a 170GB Public Dataset (Global DB of Events, Language & Tone)

In the previous labs, you worked with an extremly small dataset (less than < 10MB) and with a single data source. In this lab, let’s use a public dataset with bigger size and more tables and observe various services. 

![GDelt Project image](https://csaimmersiondaymaterial.s3-us-west-2.amazonaws.com/gdelt+project.png)


The [Global Database of Events, Language and Tone (GDELT) Project](http://www.gdeltproject.org/) monitors the world's broadcast, print, and web news from nearly every corner of every country in over 100 languages and identifies the people, locations, organisations, counts, themes, sources, emotions, counts, quotes, images and events driving our global society every second of every day. The data set v1.0 is publicly available in S3 in the [Registry of Open Data on AWS](https://aws.amazon.com/public-datasets/gdelt/).

In this lab, you will explore, catalogue, visualize, interact with this data using AWS services.
The data set we will use contains (at the time to writing) thousands of uncompressed CSV files: hundreds of millions of lines, and is about 170GB. The Data format is defined [here](http://data.gdeltproject.org/documentation/GDELT-Data_Format_Codebook.pdf). The queries below are from Julien Simon’s blog.



You will use Athena to define columns you needed with the right type and point to the S3 bucket holding all files (in another AWS account). Athena will also be used in later labs to query data hosted in S3. 

<ol type="1">
<li>Navigate to Athena in the console 

<li>Enter the following HIVE DDL statement to create a database in glue metadata store</li>
	
```
CREATE DATABASE gdelt;
```

<li>Create a Table referring to the S3 bucket holding all files in the AWS account.</li>
<ol type="A">
<li>Before we proceed, a few remarks:</li>
<ol type="i">
<li>We are creating a schema definition in our Glue service, in our Data Catalogue</li>
<li>The actual data is in another AWS account</li>
<li>You can Access this data, because it is a public dataset located in 's3://gdelt-open-data/events/folder, and is open to everyone.</li>
<li>Although we are creating TABLEs, there is no database. The events table is a representation of thousands of TSV (Tab Seperated Files) files stored in S3. Technologies like Apache HIVE and Presto enables accessing them using SQL like expressions.</li>
</ol type="i">
</ol type="A">


```
CREATE EXTERNAL TABLE IF NOT EXISTS gdelt.events (
        `globaleventid` INT,
        `day` INT,
        `monthyear` INT,
        `year` INT,
        `fractiondate` FLOAT,
        `actor1code` string,
        `actor1name` string,
        `actor1countrycode` string,
        `actor1knowngroupcode` string,
        `actor1ethniccode` string,
        `actor1religion1code` string,
        `actor1religion2code` string,
        `actor1type1code` string,
        `actor1type2code` string,
        `actor1type3code` string,
        `actor2code` string,
        `actor2name` string,
        `actor2countrycode` string,
        `actor2knowngroupcode` string,
        `actor2ethniccode` string,
        `actor2religion1code` string,
        `actor2religion2code` string,
        `actor2type1code` string,
        `actor2type2code` string,
        `actor2type3code` string,
        `isrootevent` BOOLEAN,
        `eventcode` string,
        `eventbasecode` string,
        `eventrootcode` string,
        `quadclass` INT,
        `goldsteinscale` FLOAT,
        `nummentions` INT,
        `numsources` INT,
        `numarticles` INT,
        `avgtone` FLOAT,
        `actor1geo_type` INT,
        `actor1geo_fullname` string,
        `actor1geo_countrycode` string,
        `actor1geo_adm1code` string,
        `actor1geo_lat` FLOAT,
        `actor1geo_long` FLOAT,
        `actor1geo_featureid` INT,
        `actor2geo_type` INT,
        `actor2geo_fullname` string,
        `actor2geo_countrycode` string,
        `actor2geo_adm1code` string,
        `actor2geo_lat` FLOAT,
        `actor2geo_long` FLOAT,
        `actor2geo_featureid` INT,
        `actiongeo_type` INT,
        `actiongeo_fullname` string,
        `actiongeo_countrycode` string,
        `actiongeo_adm1code` string,
        `actiongeo_lat` FLOAT,
        `actiongeo_long` FLOAT,
        `actiongeo_featureid` INT,
        `dateadded` INT,
        `sourceurl` string
) 
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
        'serialization.format' = '\t','field.delim' = '\t') LOCATION 's3://gdelt-open-data/events/';
```

<li>Create lookup tables.</li>

<ol type="A">
<li>There are a few tables in the GDELT dataset, and they provide human-friendly descriptions to event codes and country codes in the events table defined in the previous step. They are also TSV files stored in S3.</li>

<ol type="i">
<li>The Countries file that will be used as a lookup table looks like below: [](https://www.gdeltproject.org/data/lookups/CAMEO.country.txt)</li>

![country files image](https://csaimmersiondaymaterial.s3-us-west-2.amazonaws.com/country+files.png) 

<li>The EventCodes file that will be used as a lookup table looks like below:  [https://www.gdeltproject.org/data/lookups/CAMEO.eventcodes.txt](https://www.gdeltproject.org/data/lookups/CAMEO.eventcodes.txt)</li>

![Event code image](https://csaimmersiondaymaterial.s3-us-west-2.amazonaws.com/event+codes.png) 

<li>The Groups file that will be used as a lookup table looks like below:</li>
 
![Group file image](https://csaimmersiondaymaterial.s3-us-west-2.amazonaws.com/group+files.png)

<li>The Types file that will be used as a lookup table looks like below:</li>

![Type file Image](https://csaimmersiondaymaterial.s3-us-west-2.amazonaws.com/type+files.png) 
</ol type="i">

<li>To maket his exercise interesting, you will store these files in your S3 bucket. The Events table and the lookup tables will be used in a few queries using Athena. </li>
</ol type="A">

<li>First, download the files below to your computer:</li>
<ol type="A">
<li>Eventcodes: [](https://www.gdeltproject.org/data/lookups/CAMEO.eventcodes.txt)</li>
<li>Countries: [](https://www.gdeltproject.org/data/lookups/CAMEO.country.txt)</li>
<li>Types: [](https://www.gdeltproject.org/data/lookups/CAMEO.type.txt)</li>
<li>Groups: [](https://www.gdeltproject.org/data/lookups/CAMEO.knowngroup.txt)</li>
</ol type="A">


<li>Navigate to S3 in your console. We will reuse the S3 bucket that was created in the previous lab.</li>

<li>Open your bucket and create the following 4 folders (all lowercase)</li>
<ol type="A">
<li>Folder 1: 	countries</li>
<li>Folder 2: 	eventcodes</li>
<li>Folder 3: 	groups </li>
<li>Folder 4: 	types</li>
</ol type="A">

![Folders images](https://csaimmersiondaymaterial.s3-us-west-2.amazonaws.com/folders.png)

<li>Put the corresponding file under each bucket (e.g. </li>
<ol type="A">
<li>Upload CAMEO.eventcodes.txt file from your computer under eventcodes</li>
<li>Upload CAMEO. countries.txt file from your computer under countries</li>
<li>Upload CAMEO. groups.txt file from your computer under groups</li>
<li>Upload CAMEO. types.txt file from your computer under types</li>
</ol type="A">

<li>Navigate to your Athena Console. You will now add the files to your data catalogue</li>

<li>Run the following DDL statements from the Athena Console for the lookup tables. Important: Replace YOURINITIALS in bucket name before running the DDL.</li>

<li>Add eventcodes to the data catalogue by pasting the DDL below to Athena Console, replacing your initials and selecting “Run Query”</li>

```
CREATE EXTERNAL TABLE IF NOT EXISTS gdelt.eventcodes (
         `code` string,
         `description` string 
) 
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
         'serialization.format' = '\t','field.delim' = '\t') 
LOCATION 's3://<your initials + two digit number>-tame-bda-immersion/eventcodes' 
TBLPROPERTIES ( "skip.header.line.count"="1")
```

<li>Add types to the data catalogue by pasting the DDL below to Athena Console, replacing your initials and selecting “Run Query”</li>

```
CREATE EXTERNAL TABLE IF NOT EXISTS gdelt.types (
        `type` string,
        `description` string
) 
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
        'serialization.format' = '\t','field.delim' = '\t') 
LOCATION 's3://<your initials + two digit number>-tame-bda-immersion/types/'
TBLPROPERTIES ( "skip.header.line.count"="1");
```

<li>Add groups to the data catalogue by pasting the DDL below to Athena Console, replacing your initials and selecting “Run Query”</li>

```
CREATE EXTERNAL TABLE IF NOT EXISTS gdelt.groups (
        `group` string,
        `description` string
) 
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
        'serialization.format' = '\t','field.delim' = '\t') 
LOCATION 's3://<your initials + two digit number>-tame-bda-immersion/groups/'
TBLPROPERTIES ( "skip.header.line.count"="1");
```

<li>Add countries to the data catalogue by pasting the DDL below to Athena Console, replacing your initials and selecting “Run Query”</li>

```
CREATE EXTERNAL TABLE IF NOT EXISTS gdelt.countries (
        `code` string,
        `country` string
) 
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
        'serialization.format' = '\t','field.delim' = '\t') 
LOCATION 's3://<your initials + two digit number>-tame-bda-immersion/countries/'
TBLPROPERTIES ( "skip.header.line.count"="1");
```

<li>Now lets explore the data with the queries below. </li>
<ol type="A">
<li>First find the number of events per year from the Events table</li>
</ol type="A">

```
	-- Find the number of events per year
SELECT year,
	       COUNT(globaleventid) AS nb_events
	FROM gdelt.events
	GROUP BY year
	ORDER BY year ASC;
```

Output:
![output 1 image](https://csaimmersiondaymaterial.s3-us-west-2.amazonaws.com/output.png)

<li>Notice the data amount scanned? The results are returned in less than 30 seconds by scanning 175 GB of data from thousands of uncompressed CSV files on S3. That’s the power of HIVE, Presto and other Hadoop Technologies simplified by Athena Service.</li>

<li>Now let’s show the sorted top 10 event categories by joining Events table and the Eventcode lookup table:</li>

```
	-- Show top 10 event categories
SELECT eventcode,
	       gdelt.eventcodes.description,
	       nb_events
	FROM (SELECT gdelt.events.eventcode,
	             COUNT(gdelt.events.globaleventid) AS nb_events
	      FROM gdelt.events
	      GROUP BY gdelt.events.eventcode
	      ORDER BY nb_events DESC LIMIT 10)
	  JOIN gdelt.eventcodes ON eventcode = gdelt.eventcodes.code
	ORDER BY nb_events DESC;
```

Output:
![output 2 image](https://csaimmersiondaymaterial.s3-us-west-2.amazonaws.com/output+2.png)
 

<li>Count US President Obama events per year:</li>

```
	-- Count Obama events per year
SELECT year,
	       COUNT(globaleventid) AS nb_events
	FROM gdelt.events
	WHERE actor1name='BARACK OBAMA'
	GROUP BY year
	ORDER BY year ASC;
```

Output:
![output 3 image](https://csaimmersiondaymaterial.s3-us-west-2.amazonaws.com/output+3.png)

<li>Count Obama/Putin events per category</li>

```
	-- Count Obama/Putin events per category
SELECT eventcode,
	       gdelt.eventcodes.description,
	       nb_events
	FROM (SELECT gdelt.events.eventcode,
	             COUNT(gdelt.events.globaleventid) AS nb_events
	      FROM gdelt.events
	      WHERE actor1name='BARACK OBAMA'and actor2name='VLADIMIR PUTIN'
	      GROUP BY gdelt.events.eventcode
	      ORDER BY nb_events DESC)
	  JOIN gdelt.eventcodes ON eventcode = gdelt.eventcodes.code
	  WHERE nb_events >= 50
	ORDER BY nb_events DESC;
```

Output:
![output 4 image](https://csaimmersiondaymaterial.s3-us-west-2.amazonaws.com/output+4.png)

<li>None of these took more than 30 seconds and that's with uncompressed CSV, the least performing data format possible. Converting the data set columnar formats such as Parquet would yield a massive improvement</li>
</ol type="1">



Lab 2.6 Working with Redshift

Now you will use Redshift to to query a different dataset. We will import a flights dataset to query.

1.	Navigate to the IAM console

2.	In the navigation pane, choose Roles

3.	Choose Create role

4.	In the AWS Service group, choose Redshift

5.	Under Select your use case, choose Redshift - Customizable then choose Next: Permissions

6.	On the Attach permissions policies page, choose AmazonS3ReadOnlyAccess. You can leave the default setting for Set permissions boundary. Then choose Next: Tags

7.	The Add tags page appears. You can optionally add tags. Choose Next: Review

8.	For Role name, type a name for your role. For this tutorial, type myRedshiftRole

9.	Review the information, and then choose Create Role

10.	Choose the role name of the role you just created.

11.	Copy the Role ARN to your clipboard—this value is the Amazon Resource Name (ARN) for the role that you just created. You use that value when you use the COPY command to load data in the next steps.

12.	Navigate to Redshift in the console 

13.	Click Quick Launch Cluster

14.	Set the Master user password and and select the IAM role you have created in the previous step. Then click Launch Cluster. Launching the cluster will take a few minutes

15.	Once the cluster has launched, navigate to the Query Editor and enter the credentials that you have set just earlier.
![Credentials Image](https://csaimmersiondaymaterial.s3-us-west-2.amazonaws.com/credentials.png) 

16.	Enter the following command to create a table to hold the flights data

```
CREATE TABLE flights (
  year smallint,
  month smallint,
  day smallint,
  carrier varchar(80) DISTKEY,
  origin  char(3),
  dest char(3),
  aircraft_code  char(3),
  miles int,
  departures int,
  minutes  int,
  seats int,
  passengers  int,
  freight_pounds int
);
```

17.	Then enter the following command to import data into your Redshift cluster. Important! You will need to replace the IAM_ROLE with the Role ARN you have copied earlier.

```
COPY flights
FROM 's3://redshift-sample-datasets-us-east-1/flights000.gz'
IAM_ROLE 'arn:aws:iam::XXXXXXXXXX:role/myRedshiftRole'
GZIP
DELIMITER '|'
REMOVEQUOTES
REGION 'us-east-1';
```

18.	 Now you are ready to query the data. Try using the following commands to query the data. Feel free to try your own commands.

	a.	Get the number of rows in the flights table

SELECT COUNT(*) FROM flights;

	b.	Query 25 random rows of data

SELECT * FROM flights ORDER BY random() LIMIT 25;

	c.	Top 10 airlines by the number of departures

SELECT carrier, SUM (departures) FROM flights GROUP BY carrier ORDER BY 2 DESC LIMIT 10;

19.	Enter the following command to create a table to hold the aircraft data

```
CREATE TABLE aircraft (
  aircraft_code CHAR(3) SORTKEY,
  aircraft      VARCHAR(100)
);
```

20.	Then enter the following command to import aircraft into your Redshift cluster. Important! You will need to replace the IAM_ROLE with the Role ARN you have copied earlier.

```
COPY aircraft
FROM 's3://redshift-sample-datasets-us-east-1/aircraft'
IAM_ROLE 'arn:aws:iam::XXXXXXXXXX:role/myRedshiftRole'
GZIP
DELIMITER '|'
REMOVEQUOTES
REGION 'us-east-1';
```

21.	Enter the following command verify that the data import was completed successfully and to view 10 rows of aircraft data

```
SELECT * FROM aircraft ORDER BY random() LIMIT 10;
```

22.	You can now join the two tables you have created. Try the following command to query fort he most-flown types of aircraft

```
SELECT
  aircraft,
  SUM(departures) AS trips
FROM flights
JOIN aircraft using (aircraft_code)
GROUP BY aircraft
ORDER BY trips DESC
LIMIT 10;
```

23.	Try the following query to see the compression rate of each column in the flights table

```
ANALYZE COMPRESSION flights;
```

24.	Enter the following command to create a table to hold the airport data

```
CREATE TABLE airports (
  airport_code CHAR(3) SORTKEY,
  airport      varchar(100)
);
```

25.	Then enter the following command to import airport data into your Redshift cluster. Important! You will need to replace the IAM_ROLE with the Role ARN you have copied earlier.

```
COPY airports
FROM 's3://redshift-sample-datasets-us-east-1/airports'
IAM_ROLE 'arn:aws:iam::XXXXXXXXX:role/myRedshiftRole'
GZIP
DELIMITER '|'
REMOVEQUOTES
REGION 'us-east-1';
```

26.	Now try running the following query to create a new table about Las Vegas flights

```
CREATE TABLE vegas_flights
  DISTKEY (origin)
  SORTKEY (origin)
AS
SELECT
  flights.*,
  airport
FROM flights
JOIN airports ON origin = airport_code
WHERE dest = 'LAS';
```

27.	Run the following query discover where the most popular flights to Las Vegas originate

```
SELECT
  airport,
  to_char(SUM(passengers), '999,999,999') as passengers
FROM vegas_flights
GROUP BY airport
ORDER BY SUM(passengers) desc
LIMIT 10;
```

















