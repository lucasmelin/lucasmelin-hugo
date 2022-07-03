---
date: "2018-02-20"
title: "AWK Built-in Variables"
category: "unix"
---

A common choice for manipulating and processing large text files on UNIX systems is awk. I was using awk in March of last year and thought it would be helpful to have a list of built-in variables in awk for future reference.

- **FS** is the field separator (or column separator). This defines the character that separates each column of data, such as \t for tab-separated value files. It can be assigned directly, or awk can be invoked with the file flag -F in order to parse the separator out of the file automatically.
- **OFS** is the field separator to use when outputting. By default, the output field separator is the space character.
- **RS** is the record separator (or row separator).
- **ORS** is the output record separator. The delimiter to be output between every record instance.
- **NR** number of records processed (line number) across _all files_ in awk.
- **FNR** number of records processed (line number) in just the _current file_.
- **NF** number of fields in the current record (number of columns in the current row).
