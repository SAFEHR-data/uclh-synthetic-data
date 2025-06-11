# Planned process for generating synthetic data in UCLH

## Outline
This is a draft outline of a process for generating and releasing synthetic hospital data at UCLH. It is work in progress and will change. 

## Software to generate synthetic data in UCLH - datafaker/sqlsynthgen
UCLH have developed/extended a tool to create synthetic data from our data extracts. [datafaker is the new tool to generate synthetic data](https://github.com/SAFEHR-data/datafaker) based upon an earlier tool [sqlsynthgen developed at the Alan Turing Institute](https://github.com/alan-turing-institute/sqlsynthgen). 

## How datafaker works (the short non-technical version)
1. you point datafaker at an existing (OMOP) extract in a database
1. datafaker creates initial configuration files that contain 
    + the names of the tables and columns in the data extract
    + some broad summary statistics for the data in each column
1. datafaker can be used to refine the configuration files according to
    + which tables we want to include in the synthetic data
    + for each column how we want to generate the synthetic data
1. from these configuration files datafaker can be used to create & populate synthetic data tables in an existing database
1.  we will export synthetic data from the database into e.g. csv files that can be published elsewhere 

## How we avoid releasing sensitive information
1. The OMOP data extracts used by datafaker contain no patient identifiers
1. Each data column is created independently (currently) so there is no chance that a patient could be identified by a combination of rare events
1. We will review the configuration files to ensure they contain no information that could be sensitive
1. We will review the generated synthetic data to ensure they contain no information that could be sensitive

## Adding disclaimer to the synthetic data
"These data are artificially generated, any resemblance to real patients is coincidental."

## How we will store configuration files for review
We plan to store [configuration files in these folders](resources/configuration-files/) so that they can be reviewed and changes tracked.

[Using datafaker - the step-by-step technical version](11-technical-step-by-step.md)
