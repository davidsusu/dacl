# David's (Super-Simple) Configuration Language

:construction: Under design...

## Concepts

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
      subsubsubprefix.
        'complex = key' = hello
        x = 'with spaces:    '   # Comment 5
```
