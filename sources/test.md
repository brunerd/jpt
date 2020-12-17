## Automatic recovery of malformed JSON!

Comment types recognized and filtered out:

 * \# - line comment from: shell, Perl, PHP, Python, R, Ruby, MySQL

 * // - line comment from: Javascript, Java, Swift, C(99)

 * /* ... */ - single and multi-line comment block from: C, Java, Javascript, Swift

 * ; - line comment from: assembly language

 * -- line comment from: Ada, Applescript, Haskell, Lua, SQL

 * <!-- ... --> - single and multi-line comment block from: XML

 * \*\*\* - line comment from macOS's ionodecache.json

If a trailing comma is found after the last element in an object or array it is removed.

JSON Lines files are converted into an array (use -c to warn) 

Hard wrapped lines corrected by removing line endings.

Use the new *-c* flag to "complain" about each level of recovery efforts via stderr and to exit with status 1  
Otherwise malformed JSON recovery is silent unless failure.

The installer pkg includes both the full script (jpt) and the minified version (jpt.min) installed to /usr/local/jpt/ with a soft link to jpt created in /usr/local/bin. The minified version (jpt.min) is for easy inclusion in your own shell scripts while jpt is for standalone use.
