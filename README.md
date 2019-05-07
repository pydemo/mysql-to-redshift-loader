# MySQL-to-Redshift-Data-Loader
    Ground to cloud data integration tool
    Used for ad-hoc query data results load from MySQL to Amazon-Redshift.
    

Features:
 - Loads MySQL table (or query) data to Amazon-Redshift.
 - Data stream is compressed while loaded to S3 (and then to Redshift).
 - AWS Access Keys are not passed as arguments. 
 - Requires MySQL client (mysql.exe)
 - You can modify default Python [extractor.py](https://github.com/alexbuz/MySQL_To_Redshift_Loader/blob/master/sources/include/extractor.py) and [loader.py](https://github.com/alexbuz/MySQL_To_Redshift_Loader/blob/master/sources/include/loader.py)
 - Written using Python/boto/psycopg2
 - Compiled using PyInstaller.


##Other scripts
  - [Oracle -> S3](https://github.com/alexbuz/Oracle_To_S3_Data_Uploader/blob/master/README.md) data loader
  - [Oracle -> Redshift](https://github.com/alexbuz/Oracle-To-Redshift-Data-Loader/blob/master/README.md) data loader
  - [PostgreSQL -> Redshift](https://github.com/alexbuz/PostgreSQL_To_Redshift_Loader/blob/master/README.md) data loader
  - [CSV -> Redshift](https://github.com/alexbuz/CSV_Loader_For_Redshift/blob/master/README.md) data loader

[<img src="https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png">](https://www.buymeacoffee.com/0nJ32Xg)

##Purpose

- Stream/pipe/load MySQL table data to Amazon-Redshift.

## How it works
- Script connects to source MySQL using mysql.exe and extracts data to temp file.
- File is compressed and uploaded to S3 using multipart upload.
- Optional upload to Reduced Redundancy storage (not RR by default).
- Optional "make it public" after upload (private by default).
- If S3 bucket doesn't exists it will be created.
- You can control the region where new bucket is created.
- Streamed data can be tee'd (dumped on disk) during load.
- If not set, S3 Key defaulted to input query file name.
- Data is loaded to Redshift from S3 using COPY command
- Target Redshift table has to exist
- It's a Python/boto/psycopg2 script
	* Boto S3 docs: http://boto.cloudhackers.com/en/latest/ref/s3.html
	* psycopg2 docs: http://initd.org/psycopg/docs/
- Executable is created using [pyInstaller] (http://www.pyinstaller.org/)

##Audience

Database/ETL developers, Data Integrators, Data Engineers, Business Analysts, AWS Developers, SysOps

##Designated Environment
Pre-Prod (UAT/QA/DEV)

##Usage

```
c:\Python35-32\PROJECTS\MySQL2redshift>dist\MySQL_to_Redshift_loader.exe
#############################################################################
#MySQL-to-Redshift Data Loader (v1.2, beta, 04/05/2016 15:11:53) [64bit]
#Copyright (c): 2016 Alex Buzunov, All rights reserved.
#Agreement: Use this tool at your own risk. Author is not liable for any damages
#           or losses related to the use of this software.
################################################################################
Usage:
  set AWS_ACCESS_KEY_ID=test_key
  set AWS_SECRET_ACCESS_KEY=test_secret_key
  set PGPASSWORD=test123
  set MYSQL_CLIENT_HOME="C:\Program Files\MySQL\9.5"

  set REDSHIFT_CONNECT_STRING="dbname='***' port='5439' user='***' password='***' host='mycluster.***.redshift.amazonaws.com'"  
  
  
  mysql_to_redshift_loader.exe [<mysql_query_file>] [<mysql_col_delim>] [<mysql_add_header>] 
			    [<s3_bucket_name>] [<s3_key_name>] [<s3_use_rr>] [<s3_public>]
	
	--mysql_query_file -- SQL query to execure in source MySQL db.
	--mysql_col_delim  -- CSV column delimiter for downstream(,).
	--mysql_quote	-- Enclose values in quotes (")
	--mysql_add_header -- Add header line to CSV file (False).
	--mysql_lame_duck  -- Limit rows for trial upload (1000).
	--create_data_dump -- Use it if you want to persist streamed data on your filesystem.
	
	--s3_bucket_name -- S3 bucket name (always set it).
	--s3_location	 -- New bucket location name (us-west-2)
				Set it if you are creating new bucket
	--s3_key_name 	 -- CSV file name (to store query results on S3).
		if <s3_key_name> is not specified, the MySQL query filename (ora_query_file) will be used.
	--s3_use_rr -- Use reduced redundancy storage (False).
	--s3_write_chunk_size -- Chunk size for multipart upoad to S3 (10<<21, ~20MB).
	--s3_public -- Make uploaded file public (False).
	
	--red_to_table  -- Target Amazon-Redshit table name.
	--red_col_delim  -- CSV column delimiter for upstream(,).
	--red_quote 	-- Set it if input values are quoted.
	--red_timeformat -- Timestamp format for Redshift ('MM/DD/YYYY HH12:MI:SS').
	--red_ignoreheader -- skip header in input stream
	
	MySQL data uploaded to S3 is always compressed (gzip).

	Boto S3 docs: http://boto.cloudhackers.com/en/latest/ref/s3.html
	psycopg2 docs: http://initd.org/psycopg/docs/

```
#Example


###Environment variables
Set the following environment variables (for all tests):

```
  set AWS_ACCESS_KEY_ID=test_key
  set AWS_SECRET_ACCESS_KEY=test_secret_key
  set PGPASSWORD=test123
  set MYSQL_CLIENT_HOME="C:\Program Files\MySQL\9.5"

  set REDSHIFT_CONNECT_STRING="dbname='***' port='5439' user='***' password='***' host='mycluster.***.redshift.amazonaws.com'"  
  

```

### Test load with data dump.
MySQL table `crime_test` contains data from data.gov [Crime](https://catalog.data.gov/dataset/crime) dataset.
In this example complete table `crime_test` get's uploaded to Aamzon-S3 as compressed CSV file.

Contents of the file *table_query.sql*:

```
SELECT * FROM crime_test;

```
Also temporary dump file is created for analysis (by default there are no files created)
Use `-s, --create_data_dump` to dump streamed data.

If target bucket does not exists it will be created in user controlled region.
Use argument `-t, --s3_location` to set target region name

Contents of the file *test.bat*:
```
dist-64bit\MySQL_to_redshift_loader.exe ^
-q table_query.sql ^
-d "," ^
-b test_bucket ^
-k mysql_table_export ^
-r ^
-o crime_test ^
-m "DD/MM/YYYY HH12:MI:SS" ^
-s
```
Executing `test.bat`:

```
c:\Python35-32\PROJECTS\MySQL2redshift>dist-64bit\MySQL_to_redshift_loader.exe -q table_query.sql -d "," -b test_bucket -k mysql_table_export -r -o crime_test -m "DD/MM/YYYY HH12:MI:SS" -s
Uploading results of "table_query.sql" to existing bucket "test_bucket"
Started reading from MySQL (1.25 sec).
Dumping data to: c:\Python35-32\PROJECTS\Ora2redshift\data_dump\table_query\test_bucket\mysql_table_export.20160408_203221.gz
1 chunk 10.0 MB [11.36 sec]
2 chunk 10.0 MB [11.08 sec]
3 chunk 10.0 MB [11.14 sec]
4 chunk 10.0 MB [11.12 sec]
5 chunk 877.66 MB [0.96 sec]
Size: Uncompressed: 40.86 MB
Size: Compressed  : 8.95 MB
Elapsed: MySQL+S3    :69.12 sec.
Elapsed: S3->Redshift :3.68 sec.
--------------------------------
Total elapsed: 72.81 sec.


```

### Modifying default Redshift COPY command.
You can modify default Redshift COPY command this script is using.

Open file `include\loader.py` and modify `sql` variable on line 24.

```
	sql="""
COPY %s FROM '%s' 
	CREDENTIALS 'aws_access_key_id=%s;aws_secret_access_key=%s' 
	DELIMITER '%s' 
	FORMAT CSV %s 
	GZIP 
	%s 
	%s; 
	COMMIT;
	...
```


###Download
* `git clone https://github.com/alexbuz/MySQL-to-Redshift-Data-Loader`
* [Master Release](https://github.com/alexbuz/MySQL-to-Redshift-Data-Loader/archive/master.zip) -- `MySQL_to_redshift_loader 1.2`




#
#
#
#
#   
#FAQ
#  
#### Can it load MySQL data to Amazon Redshift Database?
Yes, it is the main purpose of this tool.

#### Can developers integrate `MySQL-to-Redshift-Data-Loader` into their ETL pipelines?
Yes. Assuming they use Python.

#### I'm trying to test your script. How do I preload data Crime.csv into source MySQL db?
You can use [CSV loader for MySQL] (https://github.com/data-buddy/Databuddy/releases/tag/0.3.7)

####How to increase load speed?
Input data stream is getting compressed before upload to S3. So not much could be done here.
You may want to run it closer to source or target endpoints for better performance.

#### What are the other ways to move large amounts of data from MySQL to Redshift?
You can write a sqoop script that can be scheduled with Data Pipeline.

#### Does it create temporary data file?
No

#### Can I log transfered data for analysis?
Yes, Use `-s, --create_data_dump` to dump streamed data.

#### Explain first step of data transfer?
The query file you provided is used to select data form target MySQL server.
Stream is compressed before load to S3.

#### Explain second step of data transfer?
Compressed data is getting uploaded to S3 using multipart upload protocol.

#### Explain third step of data load. How data is loaded to Amazon Redshift?
You Redshift cluster has to be open to the world (accessible via port 5439 from internet).
It uses MySQL COPY command to load file located on S3 into Redshift table.

#### What technology was used to create this tool
I used psql.exe, Python, Boto to write it.
Boto is used to upload file to S3. 
`psql.exe` is used to spool data to compressor pipe.
psycopg2 is used to establish ODBC connection with Redshift clusted and execute `COPY` command.

#### Why don't you use ODBC driver for Redshift to insert data?
From my experience it's much slower that COPY command.
It's 10x faster to upload CSV file to Amazon-S3 first and then run COPY command.
You can still use ODBC for last step.
If you are a Java shop, take a look at Progress [JDBC Driver](https://www.progress.com/blogs/booyah-amazon-redshift-challenge-loaded-in-8-min-oow14).
They claim it can load 1 mil records in 6 min.


#### What would be my MySQL-to-Redshift migration strategy?
 - Size the database
 - Network
 - Version of MySQL
 - MySQL clinet (psql.exe) availability
 - Are you doing it in one step or multiple iterations?
 

#### Do you use psql to execute COPY command against Redshift?
No. I use `psycopg2` python module (ODBC).

#### Why are you uploading extracted data to S3? whould it be easier to just execute COPY command for local spool file?
As of now you cannot load from local file. You can use COPY command with Amazon Redshift, but only with files located on S3.
If you are loading CSV file from Windows command line - take a look at [CSV_Loader_For_Redshift](https://github.com/alexbuz/CSV_Loader_For_Redshift)

#### Can I modify default psql COPY command?
Yes. Edit include/loader.py and add/remove COPY command options

Other options you may use:

    COMPUPDATE OFF
    EMPTYASNULL
    ACCEPTANYDATE
    ACCEPTINVCHARS AS '^'
    GZIP
    TRUNCATECOLUMNS
    FILLRECORD
    DELIMITER '$DELIM'
    REMOVEQUOTES
    STATUPDATE ON
    MAXERROR AS $MaxERROR

#### Does it delete file from S3 after upload?
No

#### Does it create target Redshift table?
By default no, but using `include\loader.py` you can extend default functionality and code in target table creation.


#### Where are the sources?
Sources are [here](https://github.com/alexbuz/MySQL_To_Redshift_Loader/tree/master/sources).

#### Can you modify functionality and add features?
Yes, please, ask me for new features.


## Teardown
https://github.com/pydemo/teardown



[<img src="https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png">](https://www.buymeacoffee.com/0nJ32Xg)













