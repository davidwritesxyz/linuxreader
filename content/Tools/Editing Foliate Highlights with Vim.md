Get rid of lines with color reference:
`:g/^**rgb/d`

get rid of the `> ` on each line: 
`%s/> //g`

Get rid of dashed lines:
`:g/^---/d`

Get rid of consecutive empty lines:
`:%s/\(\n\)\{3,}/\r\r/g`