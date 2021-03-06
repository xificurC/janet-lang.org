{%
(def title "Syntax and the Parser")
(def description "A Janet program begins life as a text file, just a sequence of bytes like
any other on your system. Janet source files should be UTF-8 or ASCII
encoded. Before Janet can compile or run your program, it must transform
your source code into a data structure. Janet is a lisp, which means it is
homoiconic - code is data, so all of the facilities in the language for
manipulating arrays, tuples, strings, and tables can be used for manipulating
your source code as well.")
(def prev-page {
 :link "/introduction.html"
 :text "Introduction"})
(def next-page {
 :link "/numbers.html"
 :text "Numbers"})
%}

A Janet program begins life as a text file, just a sequence of bytes like
any other on your system. Janet source files should be UTF-8 or ASCII
encoded. Before Janet can compile or run your program, it must transform
your source code into a data structure. Janet is a lisp, which means it is
homoiconic - code is data, so all of the facilities in the language for
manipulating arrays, tuples, strings, and tables can be used for manipulating
your source code as well.

But before janet code is represented as a data structure, it must be read, or parsed,
by the janet parser. Called the reader in many other lisps, the parser is a machine
that takes in plain text and outputs data structures which can be used by both
the compiler and macros. In janet, it is a parser rather than a reader because
there is no code execution at read time. This is safer and simpler, and also
lets janet syntax serve as a robust data interchange format. While a parser
is not extensible, in janet the philosophy is to extend the language via macros
rather than reader macros.

## Nil, True and False

Nil, true and false are all literals than can be entered as such
in the parser.

```janet
nil
true
false
```

## Symbols

Janet symbols are represented as a sequence of alphanumeric characters
not starting with a digit or a colon. They can also contain the characters
\!, @, $, \%, \^, \&, \*, -, \_, +, =, \|, \~, :, \<, \>, ., \?, \\, /, as
well as any Unicode codepoint not in the ASCII range.

By convention, most symbols should be all lower case and use dashes to connect words
(sometimes called kebab case).

Symbols that come from another module often contain a forward slash that separates
the name of the module from the name of the definition in the module

```janet
symbol
kebab-case-symbol
snake_case_symbol
my-module/my-fuction
*****
!%$^*__--__._+++===~-crazy-symbol
*global-var*
你好
```

## Keywords

Janet keywords are like symbols that begin with the character :. However, they
are used differently and treated by the compiler as a constant rather than a name for
something. Keywords are used mostly for keys in tables and structs, or pieces of syntax
in macros.

```janet
:keyword
:range
:0x0x0x0
:a-keyword
::
:
```

## Numbers

Janet numbers are represented by IEEE-754 floating point numbers.
The syntax is similar to that of many other languages
as well. Numbers can be written in base 10, with
underscores used to separate digits into groups. A decimal point can be used for floating
point numbers. Numbers can also be written in other bases by prefixing the number with the desired
base and the character 'r'. For example, 16 can be written as `16`, `1_6`, `16r10`, `4r100`, or `0x10`. The
`0x` prefix can be used for hexadecimal as it is so common. The radix must be themselves written in base 10, and
can be any integer from 2 to 36. For any radix above 10, use the letters as digits (not case sensitive).

```janet
0
12
-65912
4.98
1.3e18
1.3E18
18r123C
11raaa&a
1_000_000
0xbeef
```

## Strings

Strings in janet are surrounded by double quotes. Strings are 8bit clean,
meaning they can contain any arbitrary sequence of bytes, including embedded
0s. To insert a double quote into a string itself, escape
the double quote with a backslash. For unprintable characters, you can either use
one of a few common escapes, use the `\xHH` escape to escape a single byte in
hexidecimal. The supported escapes are:

- \\xHH Escape a single arbitrary byte in hexidecimal.
- \\n Newline (ASCII 10)
- \\t Tab character (ASCII 9)
- \\r Carriage Return (ASCII 13)
- \\0 Null (ASCII 0)
- \\z Null (ASCII 0)
- \\f Form Feed (ASCII 12)
- \\e Escape (ASCII 27)
- \\" Double Quote (ASCII 34)
- \\\\ Backslash (ASCII 92)

Strings can also contain literal newline characters that will be ignored.
This lets one define a multiline string that does not contain newline characters.

An alternative way of representing strings in janet is the long string, or the backquote
delimited string. A string can also be define to start with a certain number of
backquotes, and will end the same number of backquotes. Long strings
do not contain escape sequences; all bytes will be parsed literally until
ending delimiter is found. This is useful
for defining multi-line strings with literal newline characters, unprintable
characters, or strings that would otherwise require many escape sequences.

```janet
"This is a string."
"This\nis\na\nstring."
"This
is
a
string."
``
This
is
a
string
``
```

## Buffers

Buffers are similar to strings except they are mutable data structures. Strings in janet
cannot be mutated after being created, where a buffer can be changed after creation.
The syntax for a buffer is the same as that for a string or long string, but
the buffer must be prefixed with the '@' character.

```janet
@""
@"Buffer."
@``Another buffer``
```

## Tuples

Tuples are a sequence of white space separated values surrounded by either parentheses
or brackets. The parser considers any of the characters ASCII 32, \\0, \\f, \\n, \\r or \\t
to be white-space.

```janet
(do 1 2 3)
[do 1 2 3]
```

Square brackets indicate that a tuple will be used as a tuple literal rather than
a function call, macro call, or special form. The parser will set a flag on a tuple
if it has square brackets to let the compiler know to compile the tuple into a
constructor. The programmer can check if a tuple has brackets via the
`tuple/type` function.

## Arrays

Arrays are the same as tuples, but have a leading @ to indicate mutability.

```janet
@(:one :two :three)
@[:one :two :three]
```

## Structs

Structs are represented by a sequence of white-space delimited key value pairs
surrounded by curly braces. The sequence is defined as key1, value1, key2, value2, etc.
There must be an even number of items between curly braces or the parser will
signal a parse error. Any value can be a key or value. Using nil or NaN as a key, however,
will drop that pair from the parsed struct. Using nil as value will drop that
pair from the parsed struct

```janet
{}
{:key1 "value1" :key2 :value2 :key3 3}
{(1 2 3) (4 5 6)}
{@[] @[]}
{1 2 3 4 5 6}
```
## Tables

Table have the same syntax as structs, except they have the @ prefix to indicate
that they are mutable.

```janet
@{}
@{:key1 "value1" :key2 :value2 :key3 3}
@{(1 2 3) (4 5 6)}
@{@[] @[]}
@{1 2 3 4 5 6}
```

## Comments

Comments begin with a \# character and continue until the end of the line.
There are no multi-line comments.

## Shorthand

Often called reader macros in other lisps, Janet provides several shorthand
notations for some forms.

### `'x`

Shorthand for `(quote x)`

### `;x`

Shorthand for `(splice x)`

### `~x`

Shorthand for `(quasiquote x)`

### `,x`

Shorthand for `(unquote x)`

These shorthand notations can be combined in any order, allowing
forms like `''x` (`(quote (quote x))`), or `,;x` (`(unquote (splice x))`).

## Syntax Highlighting

For syntax highlighting, there is some preliminary vim syntax highlighting in [janet.vim](https://github.com/janet-lang/janet.vim).
Generic lisp syntax highlighting should, however, provide good results. One can also generate a janet.tmLanguage
file for other programs with `make grammar` from the Janet source code.

## Grammar

For anyone looking for a more succinct description of the grammar, a PEG grammar
for recognizing Janet source code is below. The PEG syntax is itself similar to EBNF.
More info on the PEG syntax can be found in [the documentation for the peg module](/peg.html).

```janet
(def grammar
  ~{:ws (set " \t\r\f\n\0")
    :readermac (set "';~,")
    :symchars (+ (range "09" "AZ" "az" "\x80\xFF") (set "!$%&*+-./:<?=>@^_|"))
    :token (some :symchars)
    :hex (range "09" "af" "AF")
    :escape (* "\\" (+ (set "ntrzf0\"\\e")
                       (* "x" :hex :hex)
                       (error (constant "bad hex escape"))))
    :comment (* "#" (any (if-not (+ "\n" -1) 1)))
    :symbol :token
    :keyword (* ":" (any :symchars))
    :constant (+ "true" "false" "nil")
    :bytes (* "\"" (any (+ :escape (if-not "\"" 1))) "\"")
    :string :bytes
    :buffer (* "@" :bytes)
    :long-bytes {:delim (some "`")
                 :open (capture :delim :n)
                 :close (cmt (* (not (> -1 "`")) (-> :n) ':delim) ,=)
                 :main (drop (* :open (any (if-not :close 1)) :close))}
    :long-string :long-bytes
    :long-buffer (* "@" :long-bytes)
    :number (cmt (<- :token) ,scan-number)
    :raw-value (+ :comment :constant :number :keyword
                  :string :buffer :long-string :long-buffer
                  :parray :barray :ptuple :btuple :struct :dict :symbol)
    :value (* (any (+ :ws :readermac)) :raw-value (any :ws))
    :root (any :value)
    :root2 (any (* :value :value))
    :ptuple (* "(" :root (+ ")" (error "")))
    :btuple (* "[" :root (+ "]" (error "")))
    :struct (* "{" :root2 (+ "}" (error "")))
    :parray (* "@" :ptuple)
    :barray (* "@" :btuple)
    :dict (* "@" :struct)
    :main :root})
```
