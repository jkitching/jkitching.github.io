---
title: "Running SQL queries on CSV data"
date: 2024-03-16
---

A few different ways to run SQL queries on CSV data:

```sh
tail -n +2 device_profiles.csv | q -d, -H -b -O -T 'select * from -' | less -S

tail -n +2 device_profiles.csv | trdsql -ih -icsv -oat 'select * from -' | less -S

tail -n +2 device_profiles.csv | sqlite3 -csv :memory: '.import /dev/stdin stdin' '.mode table' 'select * from stdin' | less -S

tail -n +2 device_profiles.csv | sed 's@,$@@' | duckdb -table -cmd 'create table stdin as select * from read_csv_auto("/dev/stdin"); select * from stdin;' | less -S

tail -n +2 device_profiles.csv | sed 's@,$@@' | csvsql --query 'select * from stdin' | csvlook | less -S
```
