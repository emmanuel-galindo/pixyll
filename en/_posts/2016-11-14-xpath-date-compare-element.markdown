---
published: true
title: Compare dates at elements with XPath 1.0
layout: post
---
Using xmllint limits you to XPath 1.0, but on the other hand you'll have it available at every box around. 

For the following example, we want to get the cars that need service. Service data is at the element context. 

``` shell
$ xmllint --format ./example.xml
<?xml version="1.0"?>
<Cars>
  <Car>
    <Model>Taurus</Model>
    <NextService>2018-04-27T16:02:46+01:00</NextService>
  </Car>
  <Car>
    <Model>Corolla</Model>
    <NextService>2016-11-01T16:02:46+01:00</NextService>
  </Car>
  <Car>
    <Model>Civic</Model>
    <NextService>2017-04-01T16:02:46+01:00</NextService>
  </Car>
</Cars>
```

The important bit below is the ./, that places the search in the context of the element of each item:

``` shell
$ today=`date +"%Y%m%d"`
$ xmllint --xpath "//Cars/Car[ number(translate(substring(./NextService,0,11), '-','')) < $today ]" example.xml
``` 
