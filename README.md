jpt (v1.0.5) - JSON Power Tool (https://github.com/brunerd/jpt)

Usage:
jpt [options] [<query>] [<file>]

Arguments:
	[<query>] - JSONPath, JSON Pointer, or plutil-style keypath expression (optional), returns entire document otherwise.
	[<file>] - path to JSON file

Notes:
	Default output mode is JSON (RFC8259/STD90)
	Multiple JSON texts will be output as a JSON Text Sequence (RFC 7464) (use -M to change behavior and processing)
	jpt accepts input by file redirection, Unix pipe, here string, heredoc and via /dev/stdin (end with Control-D)
	jpt accepts JSON, JSON text Sequences, and non-JSON mutations such as JSON5, NDJSON and jpt's own JSONPath Object Literals
	JSON Text Sequences will be strictly parsed to RFC 7464 specs, non-JSON elements are only tolerated in single and concatenated JSON texts
	Non-JSON elements are corrected automatically, if possible, before parsing (use -c for comments via stderr with an exit status of 1)
	  JSON5 Rehabilitations (mostly):
	    Unquoted object keys and single quoted strings are converted to JSON strings
	    Additional Unicode whitespaces and paragraph separators are converted to normal spaces (0x20) and newlines (0x0A)
	    Trailing commas are removed from object and array elements
	    Explicit plus signs on numbers are removed, decimals without leading zeroes are restored
	    Multi-line strings with escaped line breaks are reverted to standard JSON strings
	      Literal tabs in strings are removed also
	    Escaped character conversions: 
		  \\0 to null
		  \\v to '\\u000c'
	    JSON5 number type conversions: 
		  NaN to null (use -8 to convert to "NaN" string)
		  Inifinity to null (use -I to convert to "Infinity" string, with sign if specified)
	    Single-line (//) and multi-line (/* */) style comments are removed
		  Additionally multi-line XML (<!-- -->) and single line Shell (#), ASM (;), and Lua (--) comments are removed
	
General Options:
	-h this help screen

JSON Output Options:
	-i "<number/string>" indent number of spaces (0-10) or use a <string> for each level of indent
	-O Order property names in objects alphabetically
	-u Unicode escape (\\u) all characters above 0x7E

Advanced JSON Output Options:
	-a always output result in an array
	-c comment on corrections to non-standard JSON via stderr output (with exit status of 1)
	-F Flatten array output
	-I Convert JSON5 +/-Infinity value to a string with signedness (otherwise converts to null)
	-N Nested single element arrays are reduced
	-8 Convert JSON5 NaN to a string (otherwise converts to null)
	-/ Escape solidus / with a reverse solidus like \\/, useful for HTML with embedded JSON encoded HTML

Input Options:
	-D Detect truncation within objects and arrays in concatenated JSON only
	-g path-only JSONPath Object Literals are assigned a value of null ("gratis nulls")
	-G Guard previously defined JSONPath Object Literals from being modified by new statements
	-H Help truncated JSON by closing up open strings, objects and arrays
	-S Treat input as a JSON string

Multiple JSON Text Options:
	-M "<value>" - options for working with multiple JSON texts
		S - Output JSON Text Sequences strictly conforming to RFC 7464 (default)
		N - Output newline delimited JSON texts
		C - Output concatenated JSON texts
		A - Gather JSON texts into an array, post-query and post-patching
		a - Gather JSON texts into an array, pre-query and pre-patching 

Alternate Output Modes (non-JSON):
	-l output the length of the resulting array, object property count, or length of string

	-J JSONPath path(s) of the object returned by the query
	-j JSONPath path(s) matched by the query results

	-R JSON Pointer path(s) of the object returned by the query
	-r JSON Pointer path(s) matched by the query results
		Specifiying -EW or -Ew encodes both -r and -R JSON Pointer paths in URI fragment style

	  -J and -R path output options:
		-C append the "constructor" type (Array, Object, String, Boolean, Number, or null)
		-K/-k print property (key) names and array indices only with no preceding path, default indent is 2
			-i "<value>" indent spaces (0-10) or character for each level of indent

	-L JSONPath Object Literal notation output of the resulting object
		JPOL Format: <JSONPath expression>=<JSON value>
			<JSONPath expression> is simply Javascript expression syntax with $ as the object name
				Example: $.key, $["key"], $['key'], or $[0]
			<JSON value> is any valid JSON value (plus single quoted strings)
				Example: 'string', "string", [], {}, 42, true, false, null

	  JSONPath output options for -L -J and -j:
		-d Use dot notation for object property names when possible, rather than bracket notation
		-q Use single quotes for bracketed property names, string values remain JSON double quoted
		-Q Use single quotes for BOTH bracketed property names AND string values (-L only)
		-u encode characters above 7E with \\u escape

	  Output limiting options for -L, -J, and -R:
		-P Only print Primitive data types (String, Boolean, Number, null) omitting Arrays and Objects
		-Z "<int>" Depth
			Combined with -L it will coalesce lower depth levels into a compound JSON object/array statement
			Combined with -P both -J and -R will return only purely primitive nodes at or below the Z level

	-T textual output of all data (omits property names and indices)
	  Options:
		-e Print escaped characters literally: \\b \\f \\n \\r \\t \\v and \\\\ (\\ escaped formats only)
		-i "<value>" indent spaces (0-10) or character string for each level of indent
		-n convert null value to string 'null' (pre-encoding)

		-E "<value>" encoding options for -T output:

		  Encodes string characters below 0x20 and above 0x7E with pass-through for all else:
			x 	"\\x" prefixed hexadecimal UTF-8 strings
			O 	"\\nnn" style octal for UTF-8 strings
			0 	"\\0nnn" style octal for UTF-8 strings
			u 	"\\u" prefixed Unicode for UTF-16 strings
			U 	"\\U "prefixed Unicode Code Point strings
			E 	"\\u{...}" prefixed ES2016 Unicode Code Point strings
			W 	"%nn" Web encoded UTF-8 string using encodeURI (respects scheme and domain of URL)
			w 	"%nn" Web encoded UTF-8 string using encodeURIComponent (encodes all components URL)

			  -W encodes whitespace characters also
			  -A encodes ALL characters
		
		  Encodes both strings and numbers with pass-through for all else:
			h 	"0x" prefixed lowercase hexadecimal, UTF-8 strings
			H 	"0x" prefixed uppercase hexadecimal, UTF-8 strings
			o 	"0o" prefixed octal, UTF-8 strings
			6 	"0b" prefixed binary, 16 bit _ spaced numbers and UTF-16 strings
			B 	"0b" prefixed binary, 8 bit _ spaced numbers and UTF-16 strings
			b 	"0b" prefixed binary, 8 bit _ spaced numbers and UTF-8 strings

			  -U whitespace is left untouched (not encoded)

Data Processing:
	jpt can apply JSON Patch (RFC 6902), JSON Merge Patch (RFC 7396), and other operations to JSON text(s)

	JSON Patch processing:
		-X "<filepath to JSON Patch>" a file path to a JSON Patch array (RFC 6902)
		-x "<inline JSON Patch>" inline JSON Patch text (RFC 6902)
		
	JSON Patch Example (from RFC 6902):
		 [
			 { "op": "test", "path": "/a/b/c", "value": "foo" },
			 { "op": "remove", "path": "/a/b/c" },
			 { "op": "add", "path": "/a/b/c", "value": [ "foo", "bar" ] },
			 { "op": "replace", "path": "/a/b/c", "value": 42 },
			 { "op": "move", "from": "/a/b/c", "path": "/a/b/d" },
			 { "op": "copy", "from": "/a/b/d", "path": "/a/b/e" }
		]

	Single operations for JSON Patch, JSON Merge Patch, and others can be crafted using -o plus -p (path) and possibly -f (from) or -v/-V (value)

		-o "<operation>":
		 [JSON Patch Operations]
		  add
			-o add -p "<path>" -v "<value>"
				add/replace value in object or insert value into array index (RFC 6902)
		  replace
			-o replace -p "<path>" -v "<value>"
				replace value of existing object property (key) or array index (RFC 6902)
		  remove
			-o remove -p "<path>"
				remove specified path from the document (RFC 6902)
		  move
			-o move -f "<from>" -p "<path>"
				move a path to a new or existing path/node(RFC 6902)
		  copy
			-o copy -f "<from>" -p "<path>"
				copy a path to a new or existing path/node (RFC 6902)
		  test
			-o test -p "<path>" -v "<value>"
				test if a path matches a value exactly (RFC 6902), return 0 if true, 1 if false

		 [Other Operations]
		  diff
			-o diff -v "<value>"
				Given a value, compare with JSON text and produce a JSON Merge Patch (RFC 7396) document
		  mergepatch
			-o mergepatch (-v/-V "<value>" || -f "<path>") [-p <path>]
				JSON Merge Patch (RFC 7396) operation: null values delete target object property
				Can pull data from -v/-V "<value>" OR -f "<path>", can merge to specific -p "<path>" (JSONPaths allows multiple)
		  merge
			-o merge (-v/-V "<value>" || -f "<path>") [-p "<path>"]
				Non-RFC 7396 merging operation, object properties with null values are NOT removed
				Can pull data from -v/-V "<value>" or -f "<path>", can merge to specific -p "<path>" (JSONPaths allows multiple)
		  merge0
			-o merge0 (-v/-V "<value>" || -f "<path>") [-p "<path>"]
				Merge object properties that DO NOT intersect with source JSON
				Can pull data from -v/-V "<value>" or -f "<path>", can merge to specific -p "<path>" (JSONPaths allows multiple)
		  merge1
			-o merge1 (-v/-V "<value>" || -f "<path>") [-p "<path>"]
				Merge object properties that intersect with source JSON
				Can pull data from -v/-V "<value>" or -f "<path>", can merge to specific -p "<path>" (JSONPaths allows multiple)
				
		-p "<path>":
			-p "<path>" the target path expressed in JSON Pointer (RFC 6901) or JSONPath

		-f "<from path>":
			copy and move require a JSON Pointer (RFC 6901) or JSONPath expression

		-v/-V "<value>":
			-v "<JSON value>" is a single JSON text
			-V "<filespec to JSON value>" is a file path to a single JSON text
		  Options for -v/-V:
			-s treat input as a string
				
JSONPath Primer
	$
		the root of the JSON document, ALL JSONPath QUERIES MUST BEGIN WITH $
	.key or ['key']
		a "child" operator for an object property named 'key' in a JSON object
	..key or ..['key']
		is a recursive operator that will find all properties named 'key' within the object
	['key name'] or ["key \ud83d\udd11"]
		property names with spaces or escaped values MUST use bracket notation
	.* or [*]
		* returns the values of all the keys within an object or all indices in an array
	[start:stop:step]
		slice operation for arrays, accepts pos/neg integers, script and filter expressions, or leave empty
	[?(@.id >= 1 && @.id <= 10 )] or [?(@.key == "string" && !@['key2'])] or the off-spec [?(@name =~ /key.*/)], etc...
		filter expressions can return one or more matching objects within an array
		@ is the current object, dot and bracket notation can be used to query child nodes
		Use logical operators like: == (equal), != (not equal), > (greater than), < (less than), >= (greater or equal), <= (less or equal)
		Multiple criteria can be evaluated with && (AND) and || (OR). Regular expressions can also be used: '=~ /regexp/'
		You can negate a value using !
		@.length is the length of an array or string
		@name is the current property name (an off-spec but useful quirk of the original Goessner code)
	[(@.length/2)]
		"script expression", returns a single value, use as an array index selector that allow division
		Note: These will NOT make it into the IETF JSONPath spec
	[1] or [-1]
		array index, integers only, positive starts at the beginning, negative references from the end 
	["-"]
		The nonexistent array member after the last array element
		Use with data alteration operations, will always be empty for queries of arrays
		Akin to /- in JSON Pointer, if used with an object simply looks for a key named "-"
	["a","b"]
		union, a comma separated list of expressions values of multiple properties at the same level in an object
		Unions allow for: quoted property names (single or double), numbers, *, filter and script expressions, and slices

	For more examples: https://github.com/brunerd/jsonpath
	
JSON Pointer (RFC 6901) Primer
	""		an empty string represents the JSON document
	/		is the root or next child node with a property named "" an empty string (JSONPath $[""])
	/key		property named key in an object
	/1		property named 1 in an object or a numeric index in an array
	/-		the nonexistent array member after the last array element, use with data alteration operations
	~		in a property name must be escaped as ~0
	/		in a property name must be escaped as ~1

	Example: /JSON pointer/does/this/1/thing/well

keypath primer:
	This arcane summoning hails from ye olde NextStep and still used by plutil
	
	.		a period is used to delimit key names, 
				literal periods can be backslash (\\) escaped
	key		an object name or array element, there is no root character
								
	Keypath Example: this.is.1.ugly.key\\.path

	In JSON Pointer: /this/is/1/ugly/key.path
	And in JSONPath: $.this.is[1].ugly.["key.path"]

	Note: If a keypath query begins with characters that collide with JSONPath ($), jq-style (. or [) or JSON Pointer (/) it will be evaluated as one of those.
