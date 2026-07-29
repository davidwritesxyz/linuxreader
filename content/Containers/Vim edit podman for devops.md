Chapter stuff
:%s/\[\]{[^}]*}//g

:%s/\[\]{\_[^}]*}//g

:%s/{#[^}]*}//g

Clean link format
:%s/\[\[\(https[^]]*\)\]{\.url}](https[^)]*){style="text-decoration: none;"}/\1/g

get rid of new lines
```
:%s/\([^-#]\)\n\([^-#\n]\)/\1 \2/g
```

:%s/\ {.programlisting .snippet-con}//g

:%s/{.inlineCode}//g

Get rid of double backticks:
```
:%s/```/TRIPLEBACKTICK/g
:%s/``//g
:%s/TRIPLEBACKTICK/```/g
```

Put bullets on their own line:
```
:%s/\s\+-\s\+/\r- /g
```

Collapse double spaces:
```
:%s/  \+/ /g
```

Delete Figure lines:
```
:g/^Figure [0-9]\+\.[0-9]\+/d
```

Delete important note lines:
```
:g/^ note.*Important/d
```

Get rid of double or more new lines:
```
:%s/\(\n\)\{3,}/\r\r/g
```

Clean up images:
```
:%s/<figure class="mediaobject">\_.\{-}<img src="\([^"]*\)"\_.\{-}<\/figure>/![](\1)/g
```

Put all in a file:
Then run it against your file from the command line:

bash

```bash
vim -s cleanup.vim yourfile.md
```

Or from inside Vim:

vim

```vim
:source cleanup.vim
```

This way you can reuse it on every chapter file and tweak individual lines as you find new edge cases.