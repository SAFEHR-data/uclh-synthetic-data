# Technical step-by-step for running datafaker from Windows at UCLH

(file: 11-technical-step-by-step.md)

Commands to run from the Windows CMD prompt or the Terminal within this RStudio project (recommended).

This assumes that you already have python installed, otherwise see these [python installation instructions](https://uclh.slab.com/posts/install-python-on-windows-e-g-on-an-nhs-laptop-017hz5cd). 

## installing datafaker/sqlsynthgen
Note that datafaker originated from a fork of sqlsythgen and access commands will be renamed soon.

```
pipx install -f git+https://github.com/tim-band/sqlsynthgen
#install from a branch
pipx install -f git+https://github.com/tim-band/sqlsynthgen@user-feedback
```

## set access to source database

setx should set environment variables that persist between sessions, you may need to restart for them to become active (use set for just this session).

Replace values in square brackets (so that there are no square brackets remaining).  

```
setx SRC_DSN postgresql://[user]:[pwd]@[server-address.xuclh.nhs.uk]:[port]/[database-name]
setx SRC_SCHEMA [source-read-schema-name]
```

### to check environment variables
```
echo %SRC_DSN%
```


## actions on the source data 
```
sqlsynthgen make-tables
sqlsynthgen configure-tables
# if need to overwrite config files use --force
sqlsynthgen configure-tables --force

sqlsynthgen configure-generators
```

Edit `orm.yaml` to e.g. reduce number of tables you need to go through in `configure-tables`


### if have vocab tables
```
sqlsynthgen make-vocab
```
### making stats from source data (not needed for all generators)
```
sqlsynthgen make-stats
```

## summary of commands & what they do

command  | interactive | file created | file contents
------------- | ---- | --------- | ---------
make-tables | no | orm.yaml | column names & brief attributes e.g. numeric or dates 
configure-tables | yes | config.yaml | what to do with each table e.g. ignore, copy, generate, num_rows_per_pass: 10
configure-generators | yes | config.yaml | choose generator for each column e.g. generic.datetime.datetime
make-vocab (optional) | no | tablename.yaml.gz | make vocab tables if there are any
make-stats (optional) | no | src-stats.yaml | make stats from the data that some generators use

## destination for created data 

Needs to be a database in which you have write access to a schema. Can be the same database as the source or different.
Note that it will add rows onto any existing tables that have the same name as the OMOP tables being writtem (e.g. person, death etc.).

```
setx DST_DSN postgresql://[user]:[pwd]@[server-address.xuclh.nhs.uk]:[port]/[database-name]
setx DST_SCHEMA [destination-write-schema-name]
```


## creating data in the destination
```
sqlsynthgen create-tables
sqlsynthgen create-generators
sqlsynthgen create-data

# force overwriting of ssg.py
sqlsynthgen create-generators --force

# delete data from tables
sqlsynthgen remove-data

# set num passes to produce more rows of data
sqlsynthgen create-data --num-passes=5
```

## check created data from R
```

# populate these database credentials
user    <- 
host    <- 
port    <- 
dbname  <- 
pwd     <- 
cdmSchema   <- 
writeSchema <- 

if("" %in% c(user, host, port, dbname, pwd, writeSchema))
  stop("seems you don't have (all?) db credentials")

con <- DBI::dbConnect(RPostgres::Postgres(),user = user, host = host, port = port, dbname = dbname, password=pwd)

#list tables
DBI::dbListObjects(con, DBI::Id(schema = cdmSchema))
DBI::dbListObjects(con, DBI::Id(schema = writeSchema))

synperson <- DBI::dbReadTable(con, DBI::Id(schema = writeSchema, table="person"))
syndeath <- DBI::dbReadTable(con, DBI::Id(schema = writeSchema, table="death"))
person <- DBI::dbReadTable(con, DBI::Id(schema = cdmSchema, table="person"))
death <- DBI::dbReadTable(con, DBI::Id(schema = cdmSchema, table="death"))
```

## solving specific needs

### set number of rows in tables

1. open config.yaml
1. add `num_rows_per_pass`
```
tables:
  person:
    num_rows_per_pass: 10
  death:
    num_rows_per_pass: 3
```

### ensure that `person_id` in the `death` table has values from `person`

1. open `orm.yaml`
1. Search for `death:`
1. make sure `person_id:` has
```
      person_id:
        foreign_keys:
          - person.person_id
```

### ensure that people can only die once
Need to set that the `person_id` values in the `death` table must be unique.

1. open `orm.yaml`
1. Search for `death:`
1. add this to end : 
```
unique:
      - name: unique_person
        columns: 
          - person_id
```

