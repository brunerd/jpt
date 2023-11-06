### jpt (JSON Power Tool)

### Change Log

### v1.0.9
* Fixed issue with the [JSONPath engine](https://github.com/brunerd/jsonpath) that caused members named `values` with an array as the value, to return malformed paths ending in a `[NaN]`

### v1.0.8
* `OPTIND` and `OPTARG` now declared as **local** variables within `jpt` due to issues when embedded in `bash` scripts which uses global variables for them that are not sanitized within functions as it is for `zsh`

* Minified version (jpt.min) was not ES5 due to [changes in Closure](https://github.com/google/closure-compiler/wiki/Releases#november-7-2021-v20211107) (my bad) this caused issues in macOS 10.15 and under, this has been addressed.

### v1.0.7 
* Reverted back to standard `/usr/bin` sh-bang paths, using `env` can cause issues with upgraded shells

### v1.0.6b 
* Textual output (-T) now escapes solidus/backslashes \\ with \\\\, when encoding (-E) for escaped formats (hex, octal, Unicode code points) or if whitespace escaping (-e) is specified but without -E encoding.

### v1.0.5 
* fixed handling of keypaths with multiple escaped periods

### v1.0.4
* Support and refinements for JSONPath "existence test" in filter expressions for nodes that contain data that evaluates as "falsey" (e.g. zero, negative numbers, empty strings, null, and false). 
* Negation operator (!) now works on both existence tests and standard comparison operations in filter expressions

* For example:
```
% jpt -i0 '$[?(@.falsey)]' <<< '[{"falsey":0}]'
{"falsey":0}

% jpt -i0 $'$[?(!@.a && !@.b)]' <<< '[{"a":5,"b":9},{"c":3,"d":4}]'
{"c":3,"d":4}
```
* `-W` flag to also encode whitespace when outputting encoded text with `-T` and `-E <value>` where the modes typically only encodes non-ASCII text, for example octal output suitable for fstab:
```
% jpt -S -W -T -EO <<< 'Macintosh HD'
Macintosh\040HD
```
* Fixed bug in [closure](https://mvnrepository.com/artifact/com.google.javascript/closure-compiler) compiled minified version that affected filters. Now using whitespace only compilation level.
* More forgiving of `jq`-isms in queries: a single `.` acts as root, `.[` at the beginning is treated as `$[`
##### v1.0.3 (this is not the release you are looking for)

### v1.0.2
* Disables xtrace while inside the function for quieter debugging of your scripts
* plutil-style keypaths are now accepted
* jq-style paths that begin with `[` or `.` are now accepted (jq pipes `|` are _not_ support) 
* `jpt` function now runs in a subshell, so shell variables no longer need use `local` to avoid name collisions with global variables
* Cleanup of shell syntax for `test` from chained `-a` and `-o` style boolean operators to discrete `&&` and `||` expressions

### v1.0.1
* Help text from `jpt -h` now pipes through `less` for pagination (and has one less typo!)
* If no inputs are detected (via file path, pipe, or file redirection) `jpt` displays info and exits instead of waiting for input
```
% jpt  
jpt (v1.0.1) - JSON Power Tool (https://github.com/brunerd/jpt)
Usage: jpt [options] [query] [filepath] (see jpt -h for more help...)
```
* Fixes pkg installer bug if `/usr/local/bin` did not exist

### v1.0.0 bug fixes and enhancements
Notable new features include:
* The ability to parse and generate JSON Text Sequences, NDJSON/JSON Lines, and "concatenated JSON"
* The ability to parse "JSON5" and rehabilitate back to JSON with conversion options for NaN and Infinity number types
* Two new merge modes: `merge0` to merge object properties that DO NOT intersect with source JSON  and `merge1` to merge only ones that **do**
* Revamped hex, binary and octal numeric encoding modes using ECMAScript notation
* Continued support for JSONPath object literals (-L mode) now with record separator support for multiple JSON texts
* Truncated JSON can be detected and capped for recovery.

Read more about jpt at: [https://www.brunerd.com/blog/category/projects/jpt/](https://www.brunerd.com/blog/category/projects/jpt/)

**Installation Notes**: The package is _not_ notarized (because it breaks on older macOS). Simply right-click in Finder and choose Open to allow Gatekeeper bypass. You can also remove the quarantine by opening Terminal and running this command: `xattr -rc <drag in file from Finder>`

The installer pkg includes both the full script (jpt) and the minified version (jpt.min) installed to /usr/local/jpt/ with a soft link to jpt created in /usr/local/bin. The minified version `jpt.min` is for easy inclusion in _your_ shell scripts while `jpt` is for standalone use.
