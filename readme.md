dumpntds
========

# Background

The normal workflow for dumping password hashes from **ntds.dit** files is to use the **esedbexport** application from the **[libesedb](https://github.com/libyal/libesedb)** project. The files generated by esedbextract are then fed into the [ntdsxtract](https://github.com/csababarta/ntdsxtract) project. ntdsxtract uses the files to parse out various different items of information, in this case we would want password hashes that could be fed into john the ripper.

On large domains, the ntds.dit file can be extremely large (10 GB+), from which extracting all of the columns to a CSV file can take a long time, considering the **datatable** table contains over 1000 columns.

The aim of dumpntds is to extract the minimal amount of data required (45 columns) to perform the task in hand, thus speeding up the process.

dumpntds uses the [ManagedEsent](https://github.com/microsoft/ManagedEsent) library to access the data stored in the ntds.dit file. The ManagedEsent library wraps the underlying Windows API calls and therefore needs to be run using .Net, rather than Mono.

This fork updates the underlying framewor to .NET Framework 4.8.1 and uses NuGet packages rather than deploying dependencies as a part of repository.

There is also a single-file JSON export mode whcich can be used to integrate other tools while breaking `ntdsxtract` importability. The JSON export is opinionated and ignores null values to minimize the exported JSON file size.

# Usage

## Export JSON

Extract the ntds.dit file from the host and run using the following:
```
dumpntds -n path\to\ntds.dit\file -t Json
```

Once the process has been completed it will have generated two output files in the application directory:

- ntds.json

## Export CSV

Extract the ntds.dit file from the host and run using the following:

```
dumpntds -n path\to\ntds.dit\file
```

Once the process has been completed it will have generated two output files in the application directory:

- datatable.csv
- linktable.csv

### dsusers

The extracted files can then be used with the **dsusers.py** script from the ntdsxtract project:

```
python ./dsusers.py datatable.csv linktable.csv . --passwordhashes --syshive SYSTEM --pwdformat john --lmoutfile lm.txt --ntoutfile nt.txt
```

### dshashes.py

I have also included an updated version of the [dshashes](http://ptscripts.googlecode.com/svn/trunk/dshashes.py) python script, which was broken due to changes in the underlying ntds library. The dshashes script can be used as follows:

```
python ./dshashes.py datatable.csv linktable.csv . --passwordhashes SYSTEM
```
