# David's (Super-Simple) Configuration Language

:construction: This isnder design.

DACL is a really simple and predictable configuration language.
It has a minimalistic mental model without dirty magic,
nevertheless it is comfortable.

## Concepts

DACL shares some similarities with YAML (indentation) and INI (key-value pairs).
However, it has its own concepts:

- maximum obviousness and simplicity, no implicit magic
- describes a list of string to string key-value pairs, no data types
- indentation-based hierarchy
- hierarchy of prefix scopes, but the output is flat
- supports comments and empty lines
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
      file1
    ipsum/
      file2
  home/
    .bashrc
  tmp/
```
