
# RAthena

The goal of the RAthena package is to provide a DBI-compliant interface
to Athena using Boto3 SDK. This allows for an efficient, easy setup
connection to Athena using the Boto3 SDK as a driver.

## Installation:

Before installing RAthena ensure that Python 3+ is installed onto your
machine: <https://www.python.org/downloads/>. To install Boto3 either it
can installed the pip command or using RAthena installation funtion:

    pip install boto3

``` r
RAthena::install_boto()
```

To install RAthena (currently not on cran):

``` r
# The development version from Github
remotes::install_github("dyfanjones/rathena")
```

## Usage

### Basic Usage

Connect to athena, and send a query and return results back to R.

``` r
library(DBI)

con <- dbConnect(RAthena::athena(),
                aws_access_key_id='YOUR_ACCESS_KEY_ID',
                aws_secret_access_key='YOUR_SECRET_ACCESS_KEY',
                s3_staging_dir='s3://YOUR_S3_BUCKET/path/to/',
                region_name='us-west-2')

res <- dbExecute(con, "SELECT * FROM one_row")
dbFetch(res)
dbClearResult(res)
```

To retrieve query in 1 step.

``` r
dbGetQuery(con, "SELECT * FROM one_row")
```

### Intermediate Usage

To create a tables in athena, `dbExecute` will send the query to athena
and wait until query has been executed. This makes it and idea method to
create tables within athena.

``` r
query <- 
  "CREATE EXTERNAL TABLE impressions (
      requestBeginTime string,
      adId string,
      impressionId string,
      referrer string,
      userAgent string,
      userCookie string,
      ip string,
      number string,
      processId string,
      browserCookie string,
      requestEndTime string,
      timers struct<modelLookup:string, requestTime:string>,
      threadId string,
      hostname string,
      sessionId string)
  PARTITIONED BY (dt string)
  ROW FORMAT  serde 'org.apache.hive.hcatalog.data.JsonSerDe'
      with serdeproperties ( 'paths'='requestBeginTime, adId, impressionId, referrer, userAgent, userCookie, ip' )
  LOCATION 's3://elasticmapreduce/samples/hive-ads/tables/impressions/' ;"
  
dbExecute(con, query)
```

To find out how the table has been partitioned simply call the
`dbGetParitiions` on the table you wish to find out. The result will be
returned in a dataframe.

``` r
RAthena::dbGetPartition(con, "impressions")
```

### Advanced Usage

#### Setting up AWS CLI

RAthena is compatible with AWS CLI. This allows your aws creditentials
to be stored and not be hard coded in your connection.

To install AWS CLI please refer to:
<https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html>,
to configure AWS CLI please refer to:
<https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html>

Once AWS CLI has been set up you will be able to connect to Athena by
only putting the `s3_staging_dir`.

``` r
library(RAthena)
con <- dbConnect(RAthena::athena(),
                 s3_staging_dir = 's3://YOUR_S3_BUCKET/path/to/')
```

#### Sending data to Athena

RAthena has created a method to send data.frame from R to Athena.

``` r
# Check existing tables
dbListTables(con)
# Upload mtcars to Athena
dbWriteTable(con, "mtcars", mtcars, 
             partition=c("TIMESTAMP" = format(Sys.Date(), "%Y%m%d")),
             s3.location = "s3://mybucket/data/")

# Read in mtcars from Athena
dbReadTable(con, "mtcars")

# Check new existing tables in Athena
dbListTables(con)

# Check if mtcars exists in Athena
dbExistsTable(con, "mtcars")
```

### Tidyverse Usage

``` r
library(DBI)
library(dplyr)

con <- dbConnect(RAthena::athena(),
                aws_access_key_id='YOUR_ACCESS_KEY_ID',
                aws_secret_access_key='YOUR_SECRET_ACCESS_KEY',
                s3_staging_dir='s3://YOUR_S3_BUCKET/path/to/',
                region_name='us-west-2')
tbl(con, sql("SELECT * FROM one_row"))
```

# To Do list:

  - Upload package to cran
  - Add a logo (as everyone loves a logo)
  - Improve dbFetch method
