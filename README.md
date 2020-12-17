# jpt
JSON Power Tool: Retrieve and manipulate JSON data using [JSONPath](https://github.com/brunerd/jsonpath), [JSON Pointer](https://tools.ietf.org/html/rfc6901), [JSON Patch](http://tools.ietf.org/html/rfc6902), and [JSON Merge Patch](https://tools.ietf.org/html/rfc7386)

Written mostly in Javascript (ES5) and wrapped in a bit of shell, it can be used standalone or embedded in your bash/zsh scripts, requiring only `jsc` which has been standard on Macs since 10.4 and is widely available for Linux and even Windows with Linux subsystem installed.  

It is a non-compiled script, so it can be easily studied, maintained, or modified. It avoids the need for code-signing and notarization on the Mac platform.  

See my blog for articles, examples, and musing on the jpt: https://www.brunerd.com/blog/category/projects/jpt/

## Examples
Please see the [brunerd JSONPath](https://github.com/brunerd/jsonpath) GitHub page for an *extensive* list of example JSONPath queries.
  
[JSONPath](https://github.com/brunerd/jsonpath) has a richly expressive query syntax that can do things JSON Pointer can't do:
```
jpt '$.please.be["patient","excited"]' <<< $'{"please":{"be":{"patient":"Coming Soon","excited":"I can\'t wait!"}}}' 
[
  "Coming Soon",
  "I can't wait!"
]
```
  
[JSON Pointer](https://tools.ietf.org/html/rfc6901) is extremely simple and can query a *single* property only
```
jpt /please/be/excited <<< $'{"please":{"be":{"patient":"Coming Soon","excited":"I can\'t wait!"}}}'
"I can't wait!" 
```


## Help File (-h)
```
jpt - JSON Power Tool (https://github.com/brunerd/jpt)

Usage:
jpt [options] [<query>] [<file>]

Arguments:

[<query>] - JSONPath or JSON Pointer expression, optional. Returns entire document otherwise.
		
[<file>] - path to JSON file

Notes:
	JSON is the default output mode, to change see Alternate Output Modes.
	Single item arrays will be reduced to a single primitive JSON value (-a to inhibit)
	File/stream input accepts both JSON and JSON Path Object Literal (-k to inhibit processing)
	If no file specified, jpt will await input by Unix pipe, file redirection, or interactively by /dev/stdin until receiving Control-D
	Comments, trailing commas, and hard-wrap newlines will be automatically and progressively stripped if initial JSON parsing fails
	JSON Lines will be converted to an array for processing, queries, and output
	Use -c to be made aware of either of these automatic recovery processes

Options:
	-h this help file

JSON Output Options:
	-a always output an array for empty or single element arrays, do NOT reduce to a single JSON primitive value
	-i "<number/string>"  indent number of spaces (0-10) or use a string for each level of indent
	-u Unicode escape (\u) characters above 0x7F
	-F Flatten array output, if possible, using flat()
	-N Nesting reduction, single element arrays will be flattened, stopping if array length > 1
	-O Order all property names alphabetically in all objects (array order is not changed)

Input Options:
	-c "complain" via stderr and an exit code of 1 for comments, trailing commas, JSON Lines, or hard-wrapped data
	-S String input, treat main input (file or stdin) as a JSON string

Advanced Input Options:
	-g "guess", use null values for JSON Path Object Literals with no assigned value (path-only)
	-k kill processing of JSONPath Object Literals (see -L for outputting JPOL)
	-G "Gripe" when importing contradictory JSONPath Object Literals and non-existant JSONPaths and JSON Patch ops
	-I Inhibit type inference for JSONPath Object Literals that lack array and object initializers

Alternate Output Modes:
	-l output the length of the array, number of properties in an object, or length of string

	-j JSONPath path(s) matched by the query results
	-J JSONPath path(s) of the object returned by the query

	-r JSON Pointer path(s) matched by the query results
	-R JSON Pointer path(s) of the object returned by the query
		-E% encodes both -r and -R JSON Pointer paths in URI fragment style

	  -J and -R options:
		-C append the "constructor" type (Array, Object, String, Boolean, Number, or null)
		-K key/property name only (no preceding path)
			-i "<value>" indent spaces (0-10) or character string for each level of indent

	-L output JSON Path Object Literal notation of the resulting object
		Format: <JSON Path>=<value>
			<JSON Path> is simply Javascript expression syntax with $ as the object name
				Example: $.key, $["key"], $['key'], or $[0] 
			<value> is any valid JSON valid plus single quoted strings
				Example: 'string', "string", [], {}, 42, true, false, null

	-L, -J, and -R options:
		-P Only print Primitive data types (String, Boolean, Number, null) omitting Arrays and Objects
		-Z "<int>" Depth
			Combined with -L it will coalesce lower depth levels into a compound JSON object/array statement
			Combined with -P, -J or -R will return only purely primitive nodes at or below the Z level

	JSON Path output options for -L -J and -j:
		-d Use dot notation rather than bracket notation, when possible
		-q Use single quotes for bracketed property names
		-Q Use single quotes for both bracketed property names and string values (-L only)


	-T output resulting data as Text aka "Trampling" by omitting all structures and property names
		-e Print escaped characters literally: \b \f \n \r \t \\
		-n Print null values as the string 'null'
		-i "<value>" indent spaces (0-10) or character string for each level of indent

		-E "<value>" encoding options for -T output:
		  Encoding for character values above 0x007F (by default):
			x hexadecimal value (UTF-8) with \x escapes 
			O octal value (UTF-8), with \nnn octal escapes (shell style)
			0 octal value (UTF-8) with \0nnn octal escapes (zsh style)
			u Unicode value (UTF-16) with \u escapes
			U Unicode Code Point with \U zsh escapes
			E Unicode Code Point with \u{} ES2016 style escapes
			% URI encoded with %, JSON Pointer paths are URI encoded

			-A encodes all characters (0x0000-0xFFFF)
		
		  Encoding for all characters by default:
			h hexadecimal value (UTF-8), raw, lowercase
			H hexadecimal value (UTF-8), raw, uppercase
			o octal value (UTF-8) with ES2015 octal prefix "0o"
			6 binary value (UTF-16), 16 bits (all)
			B binary value (UTF-16), 8 bit wide (all)
			b binary value (UTF-8), 8 bit wide (all)
				Binary note: Numbers are converted to "positional" binary
				This is NOT the internal representation

Data Alteration:
	Data can be manipulated by applying a JSON Patch file or an inline statement (-X and -x)
	A single operation can be made with -o and a combination of -c -f -v

	JSON Patch document processing:
	  Create a file for common operations
		-x <inline JSON Patch> JSON Patch array expressed as a string
		-X <filespec to JSON Patch> a file path spec to a JSON Patch array

	JSON Patch and JSON Merge Patch operations:
	  A la carte operations can be crafted using -o -c -v and -f

		Operation (-o):
			-o "add" -p <path> -v <value>
				add or replace value of an object property or add/insert array index (RFC7396)
			-o "replace" -p <path> -v <value>
				replace value of existing object key or array index (RFC6902)
			-o "remove" -p <path>
				remove specified path from the document (RFC6902)
			-o "move" -f <from> -p <path>
				move a path to a new or existing path/node(RFC6902)
			-o "copy" -f <from> -p <path>
				copy a path to a new or existing path/node (RFC6902)
			-o "test" -p <path> -v <value>
				test if a path matches a value exactly (RFC6902)
			-o "mergepatch" <-v/-V <value> | -f <path>> [-p <path>]
				RFC7396 JSON Merge Patch operation: null values delete target key
				Can pull data from -f <path> instead a -v/-V value, can merge to -p (if JSON Path can be multiple)
			-o "merge" <-v/-V <value> | -f <path>> [-p <path>]
				Non-RFC7396 merging operation: null values are retained
				Can pull data from -f <path> instead a -v/-V value, can merge to -p (if JSON Path can be multiple)
			-o "diff" -v <value>
				Given a value, compare with main document and produce a JSON Merge Patch document
				
		Path (-p):
			-p <path> the target path in JSON Pointer or JSON Path

		From (-f):
			-f <path> the "from" path (JSON Pointer or Path) for copy and move operations

		Value (-v/-V):
			-v <JSON value> inline JSON expression
			-V <filespec to JSON value> a file to use for the contents of value
				-s treat input from -v or -V as a string
				
JSON Path Primer
	$ - the root of the JSON document, ALL JSONPath QUERIES MUST BEGIN WITH $
	.key - is the "child" operator for a property named 'key' in a JSON object
	..key - is a recursive operator that will find all properties name 'key' in the object
	['key] or ["key"] - quoted keys can accomodate any name, \u Unicode escaping will be processed
		.. may precede a bracket, otherwise put them next to each other with no spaces, no dots
	.* ..* or [*] - the values of all the keys within an object or indices in an array
	[start:stop:step] - slice, behavior like Python, accepts integers (pos or neg),script and slice expressions, or empty
	[?(@.subKey == "This One!")] - filter expression substitute @ with the current object level
		  Comparison operators: can be ==, != and also regex =~ /string/ 
	[(@.length/2)] - script expression returns value, good for arrays. Works in slices.
	[-] - for data alteration ops, used to add to the end of an array, borrowed from JSON Pointer
	[1] - an array index, integers only, cannot reference an numeric property name
	["a","b"] - a comma separated union can collect the values of multiple properties at the same level in an object
		 Unions allow for: quoted property names (single or double), numbers, *, filter and script expressions, and slices

	Horrible example: $..["wow"].this['is'][1][?(@.ugly == "query")][:(@.length/2 - 1):-2][*]
	For a more examples: https://github.com/brunerd/jsonpath

JSON Pointer Primer
	""		an empty string represents the JSON document
	/		is the root or next child node with a property named "" an empty string (JSONPath $[""])
	/key		property named key in an object
	/1		property named 1 in an object or a numeric index in an array
	/-		references the end of an array, where the next value goes (always non-existant in queries)
	~ in a property name must be escaped as ~0
	/ in a property name must be escaped as ~1

	Example: /JSON pointer/does/this/1/thing/well

```

### Requirements:
bash or zsh  
macOS 10.4+  
Linux with jsc installed  
Windows with Linux subsystem and jsc  
