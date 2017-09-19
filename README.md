# 4d-component-csv
Import and export CSV

### Version

<img src="https://cloud.githubusercontent.com/assets/1725068/18940649/21945000-8645-11e6-86ed-4a0f800e5a73.png" width="32" height="32" /> 

### New

Option ``delimiter`` can be specified to import ``;`` instead of ``,``.

Option ``skip_errors`` can be specified to continue parsing past incorrectly escaped double-quotes. 

## Examples

### Import

Create a text matrix (2D text array) from CSV.

```
C_OBJECT($params)
OB SET($params;\
"in";Get 4D folder(Database folder)+"CSV"+Folder separator+"in.csv";\
"encoding";"utf-8")

ARRAY TEXT($values;0;0)
CSV_TO_TEXT_MATRIX ($params;->$values)
```

Import the text matrix.

```
ARRAY POINTER($fieldPtrs;5)
$fieldPtrs{1}:=->[Example]Field_2
$fieldPtrs{2}:=->[Example]Field_3
$fieldPtrs{3}:=->[Example]Field_4
$fieldPtrs{4}:=->[Example]Field_5
$fieldPtrs{5}:=->[Example]Field_6

TRUNCATE TABLE([Example])
SET DATABASE PARAMETER([Example];Table sequence number;0)

TEXT_MATRIX_TO_SELECTION (->$values;->$fieldPtrs)
```

### Export

Create an object array to export.

```
ALL RECORDS([Example])
		
C_OBJECT($template)
OB SET($template;\
	"Field_2";->[Example]Field_2;\
	"Field_3";->[Example]Field_3;\
	"Field_4";->[Example]Field_4;\
	"Field_5";->[Example]Field_5;\
	"Field_6";->[Example]Field_6)

C_TEXT($json)
ARRAY OBJECT($records;0)

$json:=Selection to JSON([Example];$template)
JSON PARSE ARRAY($json;$records)
```

Export as CSV.

```
C_OBJECT($params)
OB SET($params;\
"out";System folder(Desktop)+"out0.csv";\
"encoding";"utf-8";\
"with_bom";True;\
"with_column_title";False;\
"no_last_line_delimiter";True)

JSON_TO_CSV ($params;->$records)
```

## About

Simple CSV files are easy to process. You just set the system variable ``RecDelimit`` to ``0x2C`` (or ``44`` decimal).

```
FldDelimit:=Character code(",")
ALL RECORDS([Product])
EXPORT TEXT([Product];System folder(Desktop)+"test.csv")
```

But this won't work for more complex cases.

CSV isn't just text separated by commas and carriage returns.

According to Read [RFC 4180](https://www.ietf.org/rfc/rfc4180.txt),

* The correct record delimiter is ``CRLF``.
* The last record may or may not end with ``CRLF``.
* The first record may or may not be a header line.
* Each line contains the same number of fields.
* Each field may or may not be enclosed by double-quotes.
* Quote field may contain ``CRLF``, commas or double-quotes.
* Double quotes are escaped by a preceding double quote.

Technically, CSV is a proprietary format, but these rules are pretty much universal.

Moreover, in business applications, a CSV does not necessarily follow these rules. It could also mean a derivative format. For example, some industries use CSV to represent structured relational data, where the number of records per line is variable. In such cases, a "header" field, usually the first field of each line, is used to identify the type of record and to infer the number of fields in that particular line.

The component is designed to deal with such issues.

On Export, you can

* Specify the encoding.
* Omit last line delimiter.
* Add Unicode Byte Order Mark.
* Add header line.

On import,

* Time may be formatted in the classic (i.e. HH:MM:SS)  or ISO format.
* Date may be formatted in the classic (e.g. YYYY/MM/DD) or ISO format.
* The last line may or may not end with a ``CRLF``.

On both import an export,

* The number of fields per record need not be the same for all lines.
* The field may contain commas, quotes, carriage returns or line feeds.
