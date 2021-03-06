# 9 Json
## 9.1 Overview
JSON stands for JavaScript Object Notation. It is a plain text format commonly used for serialization of data. It is specified in RFC 4627. The JSON format is designed to support 100% fidelity with with the full Haystack type system.

## 9.2 Type Mapping
The following is the mapping between Haystack and JSON types:
```
Haystack      JSON
--------      ----
Grid          Object (specified below)
List          Array
Dict          Object
null          null
Bool          Boolean
Marker        "m:"
Remove        "-:"
NA            "z:"
Number        "n:<float> [unit]" "n:45.5" "n:73.2 °F" "n:-INF"
Ref           "r:<id> [dis]"  "r:abc-123" "r:abc-123 RTU #3"
Str           "hello" "s:hello"
Date          "d:2014-01-03"
Time          "h:23:59"
DateTime      "t:2015-06-08T15:47:41-04:00 New_York"
Uri           "u:http://project-haystack.org/"
Coord         "c:<lat>,<lng>" "c:37.545,-77.449"
XStr          "x:Type:value"
```

Notes:

+ Number encodings use a space between the floating point value and unit for easier parsing (in Zinc there is no space)
+ Number specials use same values as Zinc: "INF", "-INF", and "NaN"
+ Refs strings use first space to separate id from dis portions of the string
+ DateTime, Date, and Time use ISO 8601 formats exactly as specified by Zinc
+ DateTime has requrired timezone name
+ Strings which contain a colon must be encoded with "s:" prefix
+ Strings without a colon may optionally omit the "s:" prefix
+ Any JSON string without a colon as the second char is assumed to be a string value

Here is an example:
```
// Haystack
dis: "Site-A", site, area: 5000ft², built: 1992-01-23

// JSON
{"dis":"Site-A", "site":"m:", "area":"n:5000 ft²", "built":"d:1992-01-23"}
```

The Haystack and JSON models are very similiar since they both support the same core list and object/dict types. The difference is that Haystack has a richer set of scalar types such as Date, Time, Uri which are not supported directly by JSON; so we encode them as strings using a special type code prefix.

## 9.3 Grid Format
In addition to the flexible type mapping defined above, we have a standard mapping of Grid into JSON which is used by the REST API.

The Grid to JSON mapping is as follows:

+ Grid is mapped into a JSON object with three fields: meta, cols, rows
+ The meta field is a JSON object with a required "ver" field
+ The cols field is a JSON list of column objects
+ Each column object defines a "name" field and the column metadata
+ The rows field is a list of JSON objects
+ Meta and row dicts are mapped to JSON objects
+ Dict values are mapped using type mappings defined above

Example

```
// Zinc
ver:"3.0" projName:"test"
dis dis:"Equip Name",equip,siteRef,installed
"RTU-1",M,@153c-699a "HQ",2005-06-01
"RTU-2",M,@153c-699a "HQ",1999-07-12

// JSON
{
  "meta": {"ver":"3.0", "projName":"test"},
  "cols":[
    {"name":"dis", "dis":"Equip Name"},
    {"name":"equip"},
    {"name":"siteRef"},
    {"name":"installed"}
  ],
  "rows":[
    {"dis":"RTU-1", "equip":"m:", "siteRef":"r:153c-699a HQ", "installed":"d:2005-06-01"},
    {"dis":"RTU-2", "equip":"m:", "siteRef":"r:153c-699a HQ", "installed":"d:999-07-12"}
  ]
}
```

Here is another example with nested lists, dicts, and grids:
```
// Zinc
ver:"3.0"
type,val
"list",[1,2,3]
"dict",{dis:"Dict!" foo}
"grid",<<
  ver:"2.0"
  a,b
  1,2
  3,4
  >>
"scalar","simple string"


// JSON
{
  "meta": {"ver":"2.0"},
  "cols":[
    {"name":"type"},
    {"name":"val"}
  ],
  "rows":[
    {"type":"list", "val":["n:1", "n:2", "n:3"]},
    {"type":"dict", "val":{"dis":"Dict!", "foo":"m:"}},
    {"type":"grid", "val":{
      "meta": {"ver":"2.0"},
      "cols":[
        {"name":"b"},
        {"name":"a"}
      ],
      "rows":[
        {"b":"n:20", "a":"n:10"}
      ]
    }},
    {"type":"scalar", "val":"simple string"}
  ]
}
```