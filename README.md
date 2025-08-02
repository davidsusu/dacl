# David's (Super-Simple) Configuration Language

:construction: DACL is still under design.

DACL is a really simple and predictable configuration language.
It has a minimalistic mental model without dirty magic,
nevertheless it is comfortable.

## Concepts

DACL shares some similarities with YAML (indentation) and INI (key-value pairs).
However, it has its own concepts:

- maximum obviousness and simplicity, no implicit magic
- describes a list of string-to-string key-value pairs, no data types
- indentation-based hierarchy
- hierarchy of prefix scopes, but the output is flat
- supports comments and no-op empty lines
- supports quotes and escape sequences
- long content can be described using continuation lines
  without any magic of autoadding/normalizing newlines or spaces
- UTF-8

## Questions

- should it be strict (can fail) or permissive?
- what should be the exact rules for whitespace, empty lines, and comments?
- what should be the best-practice recommendations?

## Examples

Simple:

```
key1 = value1
key2 = value2
prefix.
  subkey1 = value1
  subkey2 = value2
```

Little bit more complex:

```
# Simple key
key = value

# Prefix
prefix.
  subkey = subvalue

# Continuation
longdata =
  Lorem ipsum dolor sit amet, consectetur adipiscing elit.
  ' '
  Aliquam at risus eget risus facilisis consectetur.
  ' '
  Suspendisse eu nibh eros. Sed nec lacus in lacus tristique blandit sit amet id orci.
  
# Escapes and quotes
'complicated = key  ' = \ Line 1\nLine 2\n'xxx"yyy"' \'\'\' \nTHE \\ END
```

Complicated:

```
# Comment 1
key1 = value1
key2 =
  Some sentence.
  ' '
  Some other sentence.

       # Comment 2

prefix.
  subkey1 = hello
  subkey2 =
    \ xxx
    ' '
##################### Comment 3
    'xyz \' lorem '
    A B C\nD E F\n
    xxx
  subprefixA.
    subsubkey = # Comment 4
      Some Line.
      \n
      Some Other Line.
  subprefixB.
    subsubprefix.
      subsubsubprefix.subsubetcprefix.
        'complex = key' = hello
        x = 'with spaces:    '   # Comment 5
file:///
  etc/
    lorem/
      file1 = F
      file2 = F
    ipsum/
      dolor/
        file3 = F
      file4 = F
  home/
    user/
      .bashrc = F
  tmp/ = D
```

## Parsing rules

Any line is tokenized to these parts:

- key
- separator (`=`)
- value
- comment

It follows this pattern:

```
( | <key> [ <separator> [ <value> ] ] | <value> )  [ <comment> ]
```

Or in words:

- there are op and no-op lines
- no-op lines are empty or comment-only
- op lines can start with key or value
- lines starting with key can optionally continue with a separator then optionally with a value
- lines starting with key can optionally ending with a comment
- lines starting with value can optionally ending with a comment
- between parts spaces are ignored unless escaped or quoted

## Processing rules

- any non-comment non-space content is considered a key by default until it reaches separator or comment
- if there is no separator and value, it will be considered a prefix
- after a prefix a new identation level can start
- if there is a separator after the key, it is considered as an assignment line, if no value was given, the value is considered empty
- if a new identation level was started immediatly after an assignment line, we enter value continuation mode
- in value continuation mode we accept value tokens only (with optional comments), all of these will be appended to the growing value
- when the indentation resets, we leave the value continuation mode, and expect the next key
- lines without value doesn't count even in continuation mode, to add spaces, quote or escape them

## Prefix hierarchy

Key prefixes will be prepended to all their subkeys recursively. 

For example:

```
file:///
  etc/
    lorem/
      file1 = F
      file2 = F
    ipsum/
      dolor/
        file3 = F
      file4 = F
  home/
    user/
      .bashrc = F
  tmp/ = D
```

The above can be flatten as follows

```

file:///etc/lorem/file1 = F
file:///etc/lorem/file2 = F
file:///etc/ipsum/dolor/file3 = F
file:///etc/ipsum/file4 = F
file:///home/user/.bashrc = F
file:///tmp/ = D
```

## Quotes and escapes

Both single and double quoted strings are supported.
A single quoted string starts with `'` and spans until the next unescaped `'`.
A double quoted string starts with `""` and spans until the next unescaped `"`.

These characters has special meaning:

```
= \ ' " #
```

- `=` losts its special meaning in any quoted strings if we have already read the key
- `'` losts its special meaning in double-quoted strings.
- `"` losts its special meaning in single-quoted strings.
- `#` losts its special meaning in any quoted strings.
- `\` losts its special meaning in comments.

To put any of them literally, it should be escaped using `\` (e.g.: `\#`).

These are special escape sequences:

- `\|` nothing (can be used e.g. for preventing space collapse between parts)
- `\n` newline
- `\r` carriage return
- `\t` tab
- `\u{...}` any unicode character by its hexadecimal codepoint
- (etc.)

Any other escaped letter will mean itself (e.g. `\.`).

## Warnings

I tend to define DACL as non-failable.
This means that any random textual input could be a parseable DACL configuration.
However, parseable doesn't mean fully valid.
So, it would be the responsibility of the user how strict is the processing.

So there are some rules that can be violated,
but in this case a warning will be generated.

### :warning: Unexpected exit from single/double quoted sequence

If a quoted string is still open when the end of line reached (newline or end of input),
the string will be forcefully terminated.
The input before is used as it would be closed normally
(the entire unclosed content will be part of the output).

Example:

```
key1 = What's the situation?
key2 = Some other value
```
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; :arrow_down_small::arrow_down_small::arrow_down_small:

```
key1 = Whats the situation?
key2 = Some other value
```

### :warning: Unexpected exit from unicode codepoint sequence

If end of line or a character that's not a hexadecimal digit nor an ending curly bracket,
the sequence will be forcefully terminated.
The hexadecimal digits before will be used as it would be closed normally
(the corresponding unicode character will be part of the output).

Example:

```
key = Some \u{12 34} value
```

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; :arrow_down_small::arrow_down_small::arrow_down_small:

```
key = Some \u{12} 34\} value
```

### :warning: Missing value for unicode codepoint sequence

An escaped `u` character (`\u`) not followed by starting curly bracket (`{}`) will output nothing.

Example:

```
key = Some \u value
```

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; :arrow_down_small::arrow_down_small::arrow_down_small:

```
key = Some  value
```

### :warning: Empty unicode codepoint sequence

An empty unicode codepoint sequence (`\u{}`) will output nothing.

Example:

```
key = Some \u{} value
```

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; :arrow_down_small::arrow_down_small::arrow_down_small:

```
key = Some  value
```

### :warning: Unexpected exit from escape sequence

If a dangling escape character (`\`) is at the end of a line/input,
it will be ignored entirely.

Example:

```
key1 = Some value\
key2 = Some other value\
```

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; :arrow_down_small::arrow_down_small::arrow_down_small:

```
key1 = Some value
key2 = Some other value
```

### :warning: Suspicious assign operator in value continuation

When a value continuation line contains an unescaped and unquoted assign operator char (`=`),
it will be part of the content.
The goal of the warning is to prevent accidentally indented assignments.

Example:

```
key1 = Some value
 key2 = Other value
```

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; :arrow_down_small::arrow_down_small::arrow_down_small:

```
key1 = Some valuekey2 = Other value
```

### :warning: Irregular decrease of indentation

When indentation decreases, it will be interpreted at the less level of in the hierarchy stack,
thats indentation is not greater.
The indentation of this level will not be changed.

```
  key1 = value1
prefix1.
  prefix2.
      key2 = value2
    key3 = value3
      key4 = value4
    key5 = value5
        key6 = value6
  key7 = value7
  prefix3.
    key8 = value8
 key9 = value9
    key10 = value10
  prefix4.
 prefix5.
   key11 = value11
key12 = value12
```

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; :arrow_down_small::arrow_down_small::arrow_down_small:

    
```
key1 = value1
prefix1.
  prefix2.
    key2 = value2
    key3 = value3
    key4 = value4
    key5 = value5key6 = value6
  key7 = value7
  prefix3.
    key8 = value8
  key9 = value9
    key10 = value10
  prefix4.
  prefix5.
    key11 = value11
key12 = value12
```

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; :arrow_down_small::arrow_down_small::arrow_down_small:

```
key1 = value1
prefix1.prefix2.key2 = value2
prefix1.prefix2.key3 = value3
prefix1.prefix2.key4 = value4
prefix1.prefix2.key5 = value5key6 = value6
prefix1.key7 = value7
prefix1.prefix3.key8 = value8
prefix1.key9 = value9key10 = value10
prefix1.prefix5.key11 = value11
key12 = value12
```

### :warning: Unusual input

Any white-space and non-printable character except space and line separation characters (newline, carriage return)
will be interpreted like to non-special printable characters (e.g. `a`).
The major goal of the warning is to alert about malicious content.
Another goal is to warn if tab characters are used, e.g. for indentation.
Tabs can not be used for indentation, just like in YAML you can use only space for this purpose.

## Recommendations for users

- Use two spaces for indentation.
- Use one single space before the assignment operator, and another after it.
- Do not provide value content in the assignment line if there will be continuation lines.
- Use separate continuation lines for spacing between sentences in a value continuation sequence (obvious and VCS-friendly).
  For example:

  ```
  paragraph =
    This is the first sentence.
    ' '
    This is the second sentence.
    ' '
    This is yet another one.
    ' '
    This is the last sentence in this paragraph.
  ```

- For contents with uneven left use the `\|` no-op sequence at the beginning of each line.
  It will 
  For example:

  ```
  codeblock =
    \|int[] a = [1, 5, 2, 7, 3, 4, 6];\n
    \|\n
    \|for (v in a) {\n
    \|    if (v > 3) {\n
    \|        print(v);\n
    \|    }\n
    \|}\n
  ```

## Recommendations for syntax highlighters

- make keys and key prefixes heavier
- add some different background color behind key/value content
  to distinghush from leading/trailing spaces and comments
- mark starting/ending quotes with another background color,
  but not their content
- make escape sequences easily distinghushable from normal content
- make comments more restrained
- mark unusual input characters (at least whitespaces), e.g. with red background, make all of them visible if possible
- when no live linter is available, it would be nice to mark other warning locations too, e.g. with yellow background

## Recommendations for prorcessors

- use dot (`.`) and colon (`:`) both as fieldname separator
- ignore trailing colon but not dot
- use square bracket pair (`[]`) as list item appender

An example using these recommendations:

```
lorem.ipsum:
  [] = Hello
  [] = World
dolor:
  sit:
    AAA = aaa
  amet:
    BBB: = bbb
    CCC. = ccc
```

Equivalent YAML:

```yaml
lorem:
  ipsum:
    - Hello
    - World
dolor:
  sit:
    AAA: aaa
  amet:
    BBB: bbb
    CCC:
      '': ccc
```
