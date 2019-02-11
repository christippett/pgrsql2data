Intertec TimePro Utils
=============================================================

[![PyPI version](https://img.shields.io/pypi/v/pgrsql2data.svg)](https://pypi.python.org/pypi/pgrsql2data)
[![Build status](https://img.shields.io/travis/christippett/pgrsql2data.svg)](https://travis-ci.org/christippett/pgrsql2data)
[![Coverage](https://img.shields.io/coveralls/github/christippett/pgrsql2data.svg)](https://coveralls.io/github/christippett/pgrsql2data?branch=master)
[![Python versions](https://img.shields.io/pypi/pyversions/pgrsql2data.svg)](https://pypi.python.org/pypi/pgrsql2data)
[![Github license](https://img.shields.io/github/license/christippett/pgrsql2data.svg)](https://github.com/christippett/pgrsql2data)

Description
===========

This tool aims to convert the SQL files generated by [osm2po](https://osm2po.de) into generic datasets that can be imported anywhere.

Supported data formats include `csv`, `json` and `avro`.

### Long(er) description


osm2po  generates a PgRouting-compatible SQL file that contains DDL statements to create and insert data into PostgreSQL. These statements are tightly coupled to PostgreSQL/PostGIS and make it difficult to import this data elsewhere (e.g. Google BigQuery).

This tool converts SQL statements into a structured data format (`csv`, `json` or `avro`).

- Data schema is automatically inferred from the script's `CREATE TABLE` statement
- Geometry data is converted from hexidecimal WKB to WKT

This tool can theoretically be used on any SQL script that contains a (single) `CREATE TABLE` and `INSERT INTO` statements.


Installation
============

Install with `pip`:

``` bash
pip install pgrsql2data
```

Usage
=====

## Command line

``` bash
$ pgrsql2data --input osm_2po_pgr.sql --format csv
```

### Example

`sample.sql`

``` sql
-- Created by  : osm2po-core
-- Version     : 5.2.43
-- Author (c)  : Carsten Moeller - info@osm2po.de
-- Date        : Sat Jan 12 08:51:32 UTC 2019

SET client_encoding = 'UTF8';

DROP TABLE IF EXISTS nz_2po_4pgr;
-- SELECT DropGeometryTable('nz_2po_4pgr');

CREATE TABLE nz_2po_4pgr(id integer, osm_id bigint, osm_name character varying, osm_meta character varying, osm_source_id bigint, osm_target_id bigint, clazz integer, flags integer, source integer, target integer, km double precision, kmh integer, cost double precision, reverse_cost double precision, x1 double precision, y1 double precision, x2 double precision, y2 double precision);
SELECT AddGeometryColumn('nz_2po_4pgr', 'geom_way', 4326, 'LINESTRING', 2);

INSERT INTO nz_2po_4pgr VALUES 
(1, 2850, 'Ketetahi Road', null, 691734, 1680775132, 43, 3, 1, 2, 0.9172049, 50, 0.0183441, 0.0183441, 175.6671636, -39.065529, 175.6641327, -39.0733589, '0102000020E6100...'),
(2, 2857, 'Ngauruhoe Place', null, 441520023, 2938455973, 43, 3, 3, 239248, 0.0059348, 50, 1.187E-4, 1.187E-4, 175.5417358, -39.1998235, 175.5417999, -39.199804, '0102000020E6100...'),
(3, 2857, 'Ngauruhoe Place', null, 2938455973, 4173847805, 43, 3, 239248, 295015, 0.0501935, 50, 0.0010039, 0.0010039, 175.5417999, -39.199804, 175.542348, -39.1996541, '0102000020E6100...');
```

Converts to...

**CSV**
``` bash
$ pgrsql2data --input sample.sql --format csv
$ cat sample.csv

"id","osm_id","osm_name","osm_meta","osm_source_id","osm_target_id","clazz","flags","source","target","km","kmh","cost","reverse_cost","x1","y1","x2","y2","geom_way"
1,2850,"Ketetahi Road","",691734,1680775132,43,3,1,2,0.9172049,50,0.0183441,0.0183441,175.6671636,-39.065529,175.6641327,-39.0733589,"LINESTRING (175.6671636000000092 -39.0655289999999979, 175.6669300000000078 -39.0660262999999972, 175.6665609000000075 -39.0667807000000025, 175.6664728000000082 -39.0669753999999969, 175.6664277000000141 -39.0671368000000001, 175.6663901000000010 -39.0673267999999965, 175.6663532000000032 -39.0676137000000026, 175.6663232999999877 -39.0679250000000025, 175.6662765000000093 -39.0681926999999973, 175.6662121999999897 -39.0685058999999981, 175.6661235999999917 -39.0688525999999996, 175.6660063999999863 -39.0693205000000034, 175.6659176000000002 -39.0697118000000003, 175.6658382999999901 -39.0700554000000011, 175.6657261999999946 -39.0705236000000014, 175.6656762000000072 -39.0707115000000016, 175.6656149000000084 -39.0709337999999988, 175.6655820000000006 -39.0710349000000008, 175.6655240000000049 -39.0711586999999980, 175.6654651999999999 -39.0712526000000011, 175.6653054999999881 -39.0715093999999965, 175.6650927999999965 -39.0717934999999983, 175.6649300999999923 -39.0720382000000015, 175.6646489999999972 -39.0725479999999976, 175.6643620000000112 -39.0730144999999993, 175.6641327000000103 -39.0733589000000023)"
2,2857,"Ngauruhoe Place","",441520023,2938455973,43,3,3,239248,0.0059348,50,0.0001187,0.0001187,175.5417358,-39.1998235,175.5417999,-39.199804,"LINESTRING (175.5417357999999979 -39.1998235000000008, 175.5417999000000009 -39.1998040000000003)"
3,2857,"Ngauruhoe Place","",2938455973,4173847805,43,3,239248,295015,0.0501935,50,0.0010039,0.0010039,175.5417999,-39.199804,175.542348,-39.1996541,"LINESTRING (175.5417999000000009 -39.1998040000000003, 175.5422246999999913 -39.1996748999999980, 175.5423480000000040 -39.1996540999999965)"
```

**JSON**
``` bash
$ pgrsql2data --input sample.sql --format json
$ cat sample.json

{"id": 1, "osm_id": 2850, "osm_name": "Ketetahi Road", "osm_meta": null, "osm_source_id": 691734, "osm_target_id": 1680775132, "clazz": 43, "flags": 3, "source": 1, "target": 2, "km": 0.9172049, "kmh": 50, "cost": 0.0183441, "reverse_cost": 0.0183441, "x1": 175.6671636, "y1": -39.065529, "x2": 175.6641327, "y2": -39.0733589, "geom_way": "LINESTRING (175.6671636000000092 -39.0655289999999979, 175.6669300000000078 -39.0660262999999972, 175.6665609000000075 -39.0667807000000025, 175.6664728000000082 -39.0669753999999969, 175.6664277000000141 -39.0671368000000001, 175.6663901000000010 -39.0673267999999965, 175.6663532000000032 -39.0676137000000026, 175.6663232999999877 -39.0679250000000025, 175.6662765000000093 -39.0681926999999973, 175.6662121999999897 -39.0685058999999981, 175.6661235999999917 -39.0688525999999996, 175.6660063999999863 -39.0693205000000034, 175.6659176000000002 -39.0697118000000003, 175.6658382999999901 -39.0700554000000011, 175.6657261999999946 -39.0705236000000014, 175.6656762000000072 -39.0707115000000016, 175.6656149000000084 -39.0709337999999988, 175.6655820000000006 -39.0710349000000008, 175.6655240000000049 -39.0711586999999980, 175.6654651999999999 -39.0712526000000011, 175.6653054999999881 -39.0715093999999965, 175.6650927999999965 -39.0717934999999983, 175.6649300999999923 -39.0720382000000015, 175.6646489999999972 -39.0725479999999976, 175.6643620000000112 -39.0730144999999993, 175.6641327000000103 -39.0733589000000023)"}
{"id": 2, "osm_id": 2857, "osm_name": "Ngauruhoe Place", "osm_meta": null, "osm_source_id": 441520023, "osm_target_id": 2938455973, "clazz": 43, "flags": 3, "source": 3, "target": 239248, "km": 0.0059348, "kmh": 50, "cost": 0.0001187, "reverse_cost": 0.0001187, "x1": 175.5417358, "y1": -39.1998235, "x2": 175.5417999, "y2": -39.199804, "geom_way": "LINESTRING (175.5417357999999979 -39.1998235000000008, 175.5417999000000009 -39.1998040000000003)"}
{"id": 3, "osm_id": 2857, "osm_name": "Ngauruhoe Place", "osm_meta": null, "osm_source_id": 2938455973, "osm_target_id": 4173847805, "clazz": 43, "flags": 3, "source": 239248, "target": 295015, "km": 0.0501935, "kmh": 50, "cost": 0.0010039, "reverse_cost": 0.0010039, "x1": 175.5417999, "y1": -39.199804, "x2": 175.542348, "y2": -39.1996541, "geom_way": "LINESTRING (175.5417999000000009 -39.1998040000000003, 175.5422246999999913 -39.1996748999999980, 175.5423480000000040 -39.1996540999999965)"}
```

**Avro**
``` bash
$ pgrsql2data --input sample.sql --format avro
$ avrocat sample.avro

{"y2": -39.0733589, "x1": 175.6671636, "target": 2, "osm_name": "Ketetahi Road", "km": 0.9172049, "clazz": 43, "x2": 175.6641327, "source": 1, "osm_target_id": 1680775132, "cost": 0.0183441, "flags": 3, "osm_id": 2850, "y1": -39.065529, "osm_meta": null, "reverse_cost": 0.0183441, "kmh": 50, "geom_way": "LINESTRING (175.6671636000000092 -39.0655289999999979, 175.6669300000000078 -39.0660262999999972, 175.6665609000000075 -39.0667807000000025, 175.6664728000000082 -39.0669753999999969, 175.6664277000000141 -39.0671368000000001, 175.6663901000000010 -39.0673267999999965, 175.6663532000000032 -39.0676137000000026, 175.6663232999999877 -39.0679250000000025, 175.6662765000000093 -39.0681926999999973, 175.6662121999999897 -39.0685058999999981, 175.6661235999999917 -39.0688525999999996, 175.6660063999999863 -39.0693205000000034, 175.6659176000000002 -39.0697118000000003, 175.6658382999999901 -39.0700554000000011, 175.6657261999999946 -39.0705236000000014, 175.6656762000000072 -39.0707115000000016, 175.6656149000000084 -39.0709337999999988, 175.6655820000000006 -39.0710349000000008, 175.6655240000000049 -39.0711586999999980, 175.6654651999999999 -39.0712526000000011, 175.6653054999999881 -39.0715093999999965, 175.6650927999999965 -39.0717934999999983, 175.6649300999999923 -39.0720382000000015, 175.6646489999999972 -39.0725479999999976, 175.6643620000000112 -39.0730144999999993, 175.6641327000000103 -39.0733589000000023)", "id": 1, "osm_source_id": 691734}
{"y2": -39.199804, "x1": 175.5417358, "target": 239248, "osm_name": "Ngauruhoe Place", "km": 0.0059348, "clazz": 43, "x2": 175.5417999, "source": 3, "osm_target_id": 2938455973, "cost": 0.0001187, "flags": 3, "osm_id": 2857, "y1": -39.1998235, "osm_meta": null, "reverse_cost": 0.0001187, "kmh": 50, "geom_way": "LINESTRING (175.5417357999999979 -39.1998235000000008, 175.5417999000000009 -39.1998040000000003)", "id": 2, "osm_source_id": 441520023}
{"y2": -39.1996541, "x1": 175.5417999, "target": 295015, "osm_name": "Ngauruhoe Place", "km": 0.0501935, "clazz": 43, "x2": 175.542348, "source": 239248, "osm_target_id": 4173847805, "cost": 0.0010039, "flags": 3, "osm_id": 2857, "y1": -39.199804, "osm_meta": null, "reverse_cost": 0.0010039, "kmh": 50, "geom_way": "LINESTRING (175.5417999000000009 -39.1998040000000003, 175.5422246999999913 -39.1996748999999980, 175.5423480000000040 -39.1996540999999965)", "id": 3, "osm_source_id": 2938455973}
```
