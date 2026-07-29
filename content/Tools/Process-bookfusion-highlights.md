+++
title = 'How to Process Bookfusion Highlights with Vim'
description = 'Removing extra stuff from Bookfusion exported highlights'
+++
Here are my highlights pulled up in Vim:

![](/images/Pasted%20image%2020240414034031.png)

As you can see, Bookfusion gives you a lot of extra information when you export highlights. First, let's get rid of the lines that begin with `##`

Enter command mode in Vim by pressing `esc`. Then type `:g/^##/d` and press enter.

Much better. 


![](/images/Pasted%20image%2020240414034407.png)

Now let's get rid of the color references:`
```bash
:g/^Color/d
```

![](/images/Pasted%20image%2020240414034652.png)

To get rid of the timestamps, we must find a different commonality between the lines. In this case, each line ends with "UTC". Let's match that: 
```bash
:g/UTC$/d
```

Where `$` matches the end of the line.  

![](/images/Pasted%20image%2020240414035013.png)

Now, I want to get rid of the `> ` on each line: 
`%s/> //g`

![](/images/Pasted%20image%2020240414035218.png)

Almost there, you'll notice there are 6 empty lines in between each highlight. Let's shrink those down into one: 
```bash
:%s/\(\n\)\{3,}/\r\r/g
```


![](/images/Pasted%20image%2020240414035716.png)

The command above matches newline character `n` 3 or more times and replaces them with two newline characters `/r/r`.

As we scroll down, I see a few weird artifacts from the book conversion to markdown.

![](/images/Pasted%20image%2020240414040052.png)

Now, I want to get rid of any carrot brackets in the file. Let's use the substitute command again here:
```bash
%s/<//g
```

![](/images/Pasted%20image%2020240414040338.png)

Depending on your book and formatting. You may have some other stuff to edit. 