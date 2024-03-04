# cutr

Rust version of `cut` that respects delimiters

## Synopsis

```
Usage: cutr [OPTIONS] 
       <--fields <FIELDS>|--bytes <BYTES>|--chars <CHARS>> [FILES]...

Arguments:
  [FILES]...  Input file(s) [default: -]

Options:
  -d, --delimiter <DELIMITER>                Field delimiter [default: "\t"]
  -o, --output-delimiter <OUTPUT_DELIMITER>  Field delimiter
  -f, --fields <FIELDS>                      Selected fields
  -b, --bytes <BYTES>                        Selected bytes
  -c, --chars <CHARS>                        Selected chars
  -h, --help                                 Print help
  -V, --version                              Print version
```

## Rationale

The standard BSD/GNU versions of `cut` do not respect escaped field delimiter.
For example, given a file of comma-separated values (CSV) like so:

```
$ cat tests/inputs/books.csv
Author,Year,Title
Émile Zola,1865,La Confession de Claude
Samuel Beckett,1952,Waiting for Godot
Jules Verne,1870,"20,000 Leagues Under the Sea"
```

The comma in _20,000 Leagues Under the Sea_ is seen as a field delimiter and so the column is truncated:

```
$ cut -d , -f 3 tests/inputs/books.csv
Title
La Confession de Claude
Waiting for Godot
"20
```

This Rust version respects the escaped delimiter:

```
$ cutr -d , -f 3 tests/inputs/books.csv
Title
La Confession de Claude
Waiting for Godot
"20,000 Leagues Under the Sea"
```

Further, `cut` allows random selection of field but does not respect the order specified by the user.
For example, in the following command, I would expect `cut` to return the columns ordered _Year_ and then _Author_, but the tool returns them in their file order:

```
$ cut -d , -f 2,1 tests/inputs/books.csv
Author,Year
Émile Zola,1865
Samuel Beckett,1952
Jules Verne,1870
```

This Rust version will return the selections in the order requested:

```
$ cutr -d , -f 2,1 tests/inputs/books.csv
Year,Author
1865,Émile Zola
1952,Samuel Beckett
1870,Jules Verne
```

When parsing delimited files, the output field delimiter defaults to the input delimiter, but you may specify an alternate value.
For example, the input file may be CSV but I want the output delimited with the tab character:

```
$cutr -d , -o $'\t' -f 2,1 tests/inputs/books.csv
Year	Author
1865	Émile Zola
1952	Samuel Beckett
1870	Jules Verne
```

## TODO

* The original `cut` tools allow for open selections such as `-3` to indicate the first through third fields or `5-` to indicate the fifth field to the end of the record. This version allows only closed ranges, so I would like to add this feature.

## Author

Ken Youens-Clark <kyclark@gmail.com>
