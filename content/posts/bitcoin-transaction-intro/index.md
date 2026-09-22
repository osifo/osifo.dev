---
title: "What is a Bitcoin Transaction"
date: 2025-01-27T23:27:45+01:00
draft: fale # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: ''
categories:
  - Bitcoin
  - Tech
tags:
  - Bitcoin
---

A bitcoin is made up of fields, each contaning bytes of data. Every bitcoin has the same basic structure. 
In order to decode the fields correctly, I need to know their size and the format in which the data is in.

The fields in any given transaction are:

| Field | Example | Size | Format | Description|
|---|---|---|---|---|
|version | 0f4172c5 | 4 bytes | Little-Endian | This indicates teh version number of the transaction. It is used to determine whether certain features can be enabled.|
| Marker | 00 | 1 byte |  | This is used to indicate a Segregated witness (SegWit) transaction. It must always have the value 00. |
| Flag | 01 | 1 byte |  | This is also used to indicate a SegWit transaction. Value must be 01 or greater. |
| Input Count| 04 | variable size | [Compact Size](https://learnmeabitcoin.com/technical/general/compact-size) | This indicates the number of inputs in the transaction. |
| Output Count | 01 | variable size | [Compact Size](https://learnmeabitcoin.com/technical/general/compact-size) | This indicates the number of ouptuts from the transaction. |
| Inputs | [See example below](/#input-data) | a variable list | | This lists all the inputs for this transaction.|
| Outputs | [See example below](/#ouput-data) | a variable list |  |  This lists all the ouputs from this transaction. |
| Witness | [See example below](/#witness-data) | a variable list |  |  This field is used in SegWit transactions. | 




