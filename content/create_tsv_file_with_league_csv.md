+++
date = "2016-09-06T14:42:34+09:00"
draft = false
title = "Create tsv file with league csv"
tags = ["php"]

+++

<!--more-->

tsv = tab-separated values.  
<https://en.wikipedia.org/wiki/Tab-separated_values>

It's super easy to create tsv file with league/csv.  
<http://csv.thephpleague.com/>  

## Install

```
$ composer require league/csv
```

php
```
use League\Csv\Writer;

$tsv_list =[
  [1, 2, 3],
  ['foo', 'bar', 'baz'],
];

$writer = Writer::createFromFileObject(new SplTempFileObject());
$writer->setDelimiter("\t");
$writer->insertAll($tsv_list);

file_put_contents("/path/to/data.tsv", $writer->__toString());
```

The point is `setDelimiter` method.  
