# David's (Super-Simple) Configuration Language

:construction: This is under design.

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
- long content can be described using continuation lines without any magic of autoadding newlines or spaces
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
- `\t` tab
- `\u{...}` any unicode character by its hexadecimal codepoint
- (etc.)

Any other escaped letter will mean itself (e.g. `\.`).
