# brunerd JSONPath

Another take on the [JSONPath](https://goessner.net/articles/JsonPath/) query language by Stefan Goessner.
This engine is purposefully written in ES5 for the broadest compatibility and is an integral part of my other tool the [`jpt`](https://github.com/brunerd/jpt)  (JSON Power Tool)

Notable enhancements include:  
- Arrays can now be referenced with positive *and* negative integers
- Slice can now operate on arrays *and* strings and also allows a negative step integer
- Property names can be quoted with either single or double quotes and `\u` Unicode escape sequences resolved
- Queries with sloppy or invalid syntax are no longer allowed
- Property names containing `;` and `]` are no longer inaccessible, no more gotchas
- New path output options for: dot style paths, single or double quoted brackets, and even RFC6901 JSON Pointer output
- Unions may now contain any and all valid JSONPath expressions
- Octal indices are disallowed, however octals (both old and new syntax) can be used in script and filter expressions along with any other valid JS expressions EXCEPT function invocations (apart from test/match/exec)

## JSONPath Syntax

JSONPath Expression | Description
-|-
`$` | The root of the object, all queries must begin with this
`.key`| Child operator `.` references the property named `key` (property names observe [Javascript naming rules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Property_accessors))
`..key`| Recursive descent operator `..` references all properties name `key` within the object
`.*` or `[*]`| Wildcard operator `*` matches all object property name or array indices
`..*` or `..[*]`| Wildcard operator with recursive descent explodes the contents of your JSON quite well
`()`| A script expression uses the returned value of the expression to as the property name or index
`?()`| A filter expression interrogates all array/object members against the expression, descending into or returning the value of those that match
`@` | Use inside a filter or script expression. `@` is substituted with the value of the current object, `@name` will match the current property name, `@.length` will reference the length of an array or string, `@.key` will reference the property "key" of the current object
`[]`| Subscript/child operator; can contain quoted property names (`'key'`,`"key"`), numbers (negative or positive), filter and script expressions, `*` and `-` operators
`[start:end:step]`| Array/string slice operator like Python's, all field are optional, start and end default to bounds, step can be negative
`[,]`| Union operator `,` allows multiple quoted key names, array indices, slices, script/filter expressions, and `*` to be combined
`[-]`| One _after_ the last element in an array, borrowed from JSON Pointer, used for JSON creation only (not retrieval)

## Example queries:
Use with the sample store.json data found below...

JSONPath Query| Description
-|-
`$.*` | Contents of all top level elements, in this case "store" and "expensive" 
`$..*` | All members of the JSON structure, mercilessly exploded via recursive descent
`$.store.*` | All things in store, which are some books and a red bicycle
`$.store..price` | The price of everything in the store
`$..author` | All authors using recursive descent and dot notation
`$..['author']` | All authors using recursive descent and single quoted bracket notation
`$..["author"]` | All authors using recursive descent and double quoted bracket notation
`$.store.book[*].author` | All authors of all books in the store (ensures "bicycle" is not included)
`$.store.book[*]["author","title"]` | All authors and titles of all books within an array via union
`$.expensive` | What is considered expensive in this store?
`$..book[?(@.price <= $["expensive"])]` | All books less than or equal to the 'expensive' property
`$..book[?(@.price > $.expensive)]` | All books more than the 'expensive' property
`$["store"]..price` | The price of everything in the store
`$..book[?(@.comment)]` | Filter all books with a "comment" property containing data
`$..book[?(@.comment !== undefined)]` | Filter all books with a "comment" property
`$.store.book[?(@.comment_hidden)]` | Filter books with the "comment_hidden" property
`$..book[2]` | The third book (zero based array)
`$..book[-1]` | The last book via negative index
`$..book[(@.length-1)]`| The last book via script expression subscript
`$..book[(@.length/2)]`| The middle book via script expression subscript
`$..book[-2]` | The next to last book only via negative index
`$..book[-2:]` | The last two books via slice
`$..book[::-1]` | All books in the array in reverse order via slice
`$..book[:(@.length/2)]` | First half of books
`$..book[(@.length/2):]` | Last half of the books
`$..book[0,1]`| The first two books via subscript union
`$..book[:2]` | The first two books via subscript array slice
`$..book[?(@.isbn)]` | Filter all books with isbn number
`$..book[?(@.price<10)]`| Filter all books less than 10
`$..book[?(!(@.price<10))]`| Filter all books NOT less than 10
`$..book[?(@.price>=10)]`| Filter all books greater than or equal to 10 (same as above)
`$..book[?(@.price==8.95)]`| Filter all books that cost exactly 8.95
`$..book[?(@.category=="fiction")]` | Filter all books with a category of "fiction" exactly (will not match "Fiction")
`$..book[?(@.price < $.expensive && @.category=~/fiction/i)]` | Filter all books less than the root property of "expensive" with case-insensitive category of "fiction"
`$..book[?(@.price < $.expensive && /fiction/i.test(@.category))]` | Same as above but with Javascript style regex used in the filter expression, supported but ugly
`$..book[?(@.price < 10 \|\| @.category=~/humor/i)]` | Filter all books less than 10 OR with case-insensitive category containing "humor"
`$..book[?(@.title =~/\u9053\u5fb7/)]` | Finds all books beginning with 道德 using Unicode escape sequences in a regex
`$.store.emoji["\ud83e\udd13"]` | Look up a Unicode key name using Unicode escape sequences 

### Sample data:  
Adapted and expanded from Stefan Goessner's [original post](http://goessner.net/articles/JsonPath/)
<details><summary><b>store.json</b></summary>
<p>

```javascript
{
  "store": {
    "book": [ 
      {
        "category": "reference",
        "author": "Nigel Rees",
        "title": "Sayings of the Century",
        "price": 8.95,
        "comment":"",
        "comment_hidden": "\"A bird in hand is worth two in the bush\" is still an awkward phrase."
      },
      {
        "category": "reference/humor",
        "author": "Eric S. Raymond",
        "title": "New Hackers Dictionary, 3rd edition",
        "isbn":"0-262-68092-0",
        "price": 18.99,
        "comment":"If you are a \ud83e\udd13 you are sure to be amused"
      },
      {
        "category": "fiction",
        "author": "Evelyn Waugh",
        "title": "Sword of Honour",
        "price": 12.99,
        "comment":"Page after page of war stories. Fun.",
        "comment_hidden":"This book is likely cursed."
      },
      {
        "category": "Fiction",
        "author": "Herman Melville",
        "title": "Moby Dick",
        "isbn": "0-553-21311-3",
        "price": 8.99,
        "comment":"Before Jaws, there was Moby Dick",
        "comment_hidden":"Based on Mocha Dick, who received no royalties"
      },
      {
        "category": "fiction",
        "author": "J. R. R. Tolkien",
        "title": "The Lord of the Rings",
        "isbn": "0-395-19395-8",
        "price": 22.99,
        "comment":"Precious. My precious. Collectors edition.",
        "comment_hidden":"Cursed but totally worth it."
      },
      {
        "category":"philosophy",
        "author":"老子",
        "title":"道德经",
        "price":30,
        "comment":"The Tao Te Ching in Chinese"
      },
      {
        "category":"philosophy",
        "author":"老子",
        "title":"道德經",
        "price":80,
        "comment":"The Tao Te Ching in Chinese, original title"
      }
    ],
    "bicycle": {
      "author":"Bikes don't have authors, silly",
      "color": "red",
      "price": 19.95,
      "comment":"A great bike for a kid!",
      "comment_hidden":"Cursed but shiny!"
    },
    "emoji":{
      "🤓":{
        "description":"smiling face with glasses",
        "description_alternate":"nerd"
      }
    }
  },
  "expensive": 20
}
```
</details>

## Source Files

- [jsonpath.js](./jsonpath.js) The JSONPath engine.
- [jsonpath.no_comment.js](./jsonpath.no_comment.js) Same as above but without comments.
- [jsonpath.min.js](./jsonpath.min.js) The minified "one-liner" version (not obfuscated)

### Invoking the `jsonpath()` function:
`jsonPath(obj, expr [, arg])`

Parameters:  
`obj` (Object|Array|String|Number|Boolean|null):  
	Object representing the JSON structure  

`expr` (String|Array):   
	Either a JSONPath expression in string form or a pre-composed array representation of a JSONPath or JSON Pointer expression.  

`arg` (Object|undefined):  
	The `arg` object controls output, it can contain the following properties and values:  

`resultType` value | Description
-|-
`VALUE`|the result is the matching values (default)  
`PATH`|path(s) matched by the query in bracket notation with double quotes  
`PATH_DOTTED`|path(s) matched by the query in dot notation where possible with double quoted bracket notation otherwise  
`PATH_JSONPOINTER`|path(s) matched by the query in [JSON Pointer (RFC6901)](https://tools.ietf.org/html/rfc6901) format

Properties that apply to `PATH` and `PATH_DOTTED` `resultType` output:

`singleQuoteKeys` value | Description
-|-
`true`|Use single quotes for bracket notation
`false`|The default, uses double quotes for bracket notation

`escapeUnicode` value| Description
-|-
`true` | Uses Unicode `\u` escape sequences for all characters outside the Basic Latin Unicode block
`false` | The default, characters `\u0000-\u001f` are *always* Unicode escaped, with exceptions of `\b` `\f` `\n` `\r` and `\t`

Example `arg` object:  
`{resultType:"PATH_DOTTED",singleQuoteKeys:true,escapeUnicode:true}`  

Return value:  
Always an array. Empty or otherwise. The original implementation returns false for no matches.

### JSONPath `expr` internal representation:
Within `jsonpath()` the `expr` string is parsed into an array containing strings, numbers, and objects (with a single property of "expression")

JSONPath example, the expression:
`$[*][(@.length/2)]["key"][?(@.subKey == 'cool')][0][0:(@.length/3)]` 
Is converted internally to an array, expressed here as JSON: 
`[{"expression":"*"},{"expression":"(@.length/2)"},"key",{"expression":"?(@.subKey == \"cool\")"},0,{"expression":[0,{"expression":"(@.length/3)"},null]}]`

An array of this same format can be passed into `jsonpath()` directly.  
Since JSON Pointer is so easily parsed, this allows for jsonpath() to be given an `expr` array derived from JSON Pointer.

```javascript
//split on /
expr = expr.split('/')
//throw out first entry
expr.shift()
//replace special symbols ~1 and ~0 (in this order) with the actual characters
//convert string representations of numbers to Number types (for proper quoting in PATH output)
expr = expr.map(function (f){
	return f.replace(/~1/g,"/").replace(/~0/g,"~") })
```

JSON Pointer example:
`/0/key/sub/` is a JSON Pointer the JSONPath equivalent is: `$[0].key.sub[""]` (yes, property names with empty strings are allowed in JSON!)
Using the above code `expr` will convert internally to: `[0,"key","sub",""]`

## brunerd JSONPath grammar
Hat tip to [cburgmer](https://github.com/cburgmer) for providing the bones of the grammar declaration with [Proposal A](https://github.com/cburgmer/json-path-comparison/blob/master/proposals/Proposal_A/README.md) which I've adjusted for this implementation

    Start
      ::= "$" Operator*

    Operator
      ::= DotChild
        | BracketChildren
        | RecursiveDescentChildren

    DotChild
      ::= "." DotChildName
        | ".*"

    DotChildName
      ::= [^0-9 -#%-\/:-@\[-^`{-~][\w\d$]*

    BracketChildren
      ::= "[" ws BracketElements ws "]"

    BracketElements
      ::= BracketElement ws "," ws BracketElements
        | BracketElement

    BracketElement
      ::= Integer? ":" Integer? ":" NonZeroInteger?
        | Integer? ":" Integer?
        | BracketChild
        | "*"
        | "-"
        | "?(" FilterExpression ")"
        | "(" ScriptExpression ")"

    RecursiveDescentChildren
      ::= ".." DotChildName
        | "..*"
        | ".." BracketChildren

    BracketChild
      ::= "'" SingleQuotedString "'"
        | '"' DoubleQuotedString '"'
        | Integer

    FilterExpression
      ::= LogicalAnd
        | LogicalOr
        | HigherPrecedenceFilterExpression

    ScriptExpression
      ::= LogicalAnd
        | LogicalOr
        | HigherPrecedenceFilterExpression

    LogicalAnd
      ::= HigherPrecedenceFilterExpression ws "&&" ws LogicalAnd
        | HigherPrecedenceFilterExpression ws "&&" ws HigherPrecedenceFilterExpression

    LogicalOr
      ::= HigherPrecedenceFilterExpression ws "||" ws LogicalOr
        | HigherPrecedenceFilterExpression ws "||" ws HigherPrecedenceFilterExpression

    HigherPrecedenceFilterExpression
      ::= FilterValue ws ComparisonOperator ws FilterValue
        | UnaryFilterExpression

    UnaryFilterExpression
      ::= FilterValue
        | "!" ws UnaryFilterExpression
        | "(" ws FilterExpression ws ")"

    ComparisonOperator
      ::= "=="
        | "!="
        | "==="
        | "!=="
        | "=~"
        | "<"
        | ">"
        | "<="
        | ">="
        | "<<"
        | ">>"
        | ">>>"
        | "&"
        | "^"
        | "|"
        | "instanceof"
        
    FilterValue
      ::= "@" ScalarOperator*
        | "$" ScalarOperator*
        | SimpleValue

    ScalarOperator
      ::= "." DotChildName
        | "[" ws BracketChild ws "]"

    SimpleValue
      ::= "'" SingleQuotedString "'"
        | '"' DoubleQuotedString '"'
        | '/' RegexSearch '/'
        | Number
        | "false"
        | "true"
        | "null"
        | undefined

## References
[Stefan Gössner](https://goessner.net/articles/JsonPath/) and his initial work on the concept of JSONPath.

Christoph Burgmer's [Proposal A](https://github.com/cburgmer/json-path-comparison/blob/master/proposals/Proposal_A/README.md) and his [JSON Path comparison](https://cburgmer.github.io/json-path-comparison/) matrix  
A vast collection of open source JSONPath implementations with all their varying behaviors.
