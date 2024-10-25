---
title: "Running SQL queries on CSV data"
date: 2024-03-16
---

I was recently working with some CSV files pulled from a Samsung Health dump.  I wanted to run a few queries on the data in order to familiarize myself with their format and contents.  SQL seemed like a natural choice, and it turns out there is a myriad of different ways to directly query CSV data with SQL.

A few notes:

* The first line of the file is extraneous information that Samsung Health includes for its own purposes.  Hence the `tail -n +2` command.
* It turns out that there is one more header value than in the data rows themselves.  Some of these tools are not particularly happy about this, and so the `sed` command removes the very last (empty) header value.

# q

```sh
tail -n +2 device_profiles.csv \
  | q -d, -H -b -O -T 'select * from -' \
  | less -S
```

# trdsql

```sh
tail -n +2 device_profiles.csv \
  | trdsql -ih -icsv -oat 'select * from -' \
  | less -S
```

# sqlite3

```sh
tail -n +2 device_profiles.csv \
  | sqlite3 -csv :memory: '.import /dev/stdin stdin' '.mode table' 'select * from stdin' \
  | less -S
```

# duckdb

```sh
tail -n +2 device_profiles.csv \
  | sed 's@,$@@' \
  | duckdb -table -cmd 'create table stdin as select * from read_csv_auto("/dev/stdin"); select * from stdin;' \
  | less -S
```

# csvlook

```sh
tail -n +2 device_profiles.csv \
  | sed 's@,$@@' \
  | csvsql --query 'select * from stdin' \
  | csvlook \
  | less -S
```
