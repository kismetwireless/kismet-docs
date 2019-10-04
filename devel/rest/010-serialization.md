---
title: "Serialization types"
permalink: /docs/devel/webui_rest/serialization/
docgroup: "devel-rest"
excerpt: "Data can be serialized in several different ways, as traditional JSON, streaming pseudo-JSON for large queries, and as 'pretty' output for learning the API."
---
## Serialization Types

Kismet can export data as several different formats; generally these formats are indicated by the type of endpoint being requested (such as foo.json)

### JSON

Kismet will export objects in traditional JSON format suitable for consumption in javascript or any other language with a JSON interpreter.

### EKJSON

"EK" JSON is modeled after the Elastic Search JSON format, where a complete JSON object is found on each line of the output.

Kismet supports ekjson on any REST endpoint which returns a vector/list/array of results.  

*Added 2019-10*
To be compatible with the ELK interpretation of field names, Kismet now permutes all field names in `ekjson` output, replacing all instances of `.` with `_`.

### ITJSON

*Added 2019-10*

"IT" or "Iterative" JSON is a variant of JSON optimized for large vector/array data sets.  Instead of containing the entire output in a JSON array, each element of the array is sent on its own newline.

*All non-ELK use of previous ekjson endpoints should now use itjson endpoints*.  The ekjson serialization now modifies field names.

Kismet supports itjson on any REST endpoint which returns a vector/list/array of results.

The itjson results must be parsed *as a stream* instead of *as a traditional JSON object*.  Attempting to parse an itjson response as traditional JSON will result in syntax errors.

### PRETTYJSON

"Pretty" JSON is optimized for human readability and includes metadata fields describing what Kismet knows about each field in the JSON response.  For more information, see the previous section, `Exploring the REST system`.

"Pretty" JSON should only be used for learning about Kismet and developing; for actual use of the REST API standard "JSON" or "EKJSON" endpoints should be used as they are significantly faster and optimized.
