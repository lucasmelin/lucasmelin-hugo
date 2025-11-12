---
date: "2018-02-20"
title: "AWK Built-in Variables"
summary: "A common choice for manipulating and processing large text files on UNIX systems is `awk`. I was using `awk` in March of 2017 and thought it would be helpful to have a list of `awk` built-in variables for future reference."
tags: ["unix"]
slug: awk-built-in-variables
---

A common choice for manipulating and processing large text files on UNIX systems is `awk`. I was using `awk` in March of 2017 and thought it would be helpful to have a list of `awk` built-in variables for future reference.

| Name  | Description                                                                                                                                                                                                                                                                                     |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `FS`  | The field separator (or column separator). This defines the character that separates each column of data, such as `\t` for tab-separated value files. It can be assigned directly, or awk can be invoked with the file flag `-F` in order to parse the separator out of the file automatically. |
| `OFS` | The field separator to use when outputting. By default, the output field separator is the space character.                                                                                                                                                                                      |
| `RS`  | The record separator (or row separator). This defines the character that separates each row of data, such as `\n` for newline-delimited files.                                                                                                                                                  |
| `ORS` | The output record separator. The delimiter is output between every record instance.                                                                                                                                                                                                             |
| `NR`  | The number of records processed (line number) across _all files_                                                                                                                                                                                                                                |
| `FNR` | The number of records processed (line number) in just the _current file_.                                                                                                                                                                                                                       |
| `NF`  | The number of fields in the current record (number of columns in the current row).                                                                                                                                                                                                              |
