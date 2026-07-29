---
title: 'You Need to Learn Man Pages'
summary: 'You need to be using man pages. Here is how. A short guide to using Man Pages'
date: 2026-02-02
---

https://www.youtube.com/watch?v=RzAkjX_9B7E&t=295s

Man (manual) pages are the built in help system for Linux. They contain documentation for most commands. 

Run the `man` command on a command to get to it's man page. 
`man man`

**Navigating a man page**
`h` 
- Get help

`q` 
- Quit out of the man page

Man uses less

^ mean ctrl

^f Forward one page

^b backward one page

can use # followed by command to repeat that many times

g first line in file

G last line in file

CR means press enter

## Searching

/searchword

press enter to jump first occurance of searched word

n to jump to next match

N to go to previous match

?searchword to do a backward search (n and N are reversed when going through results)

## Man page conventions

**bold text** type as shown

*italic text* replace with arguments

* Italic may not render in terminal and may be underlined or colored text instead.

[-abc] optional

-a | -b Options separated by a pipe symbol cannot be used together.

<u>argument</u> ... (followed by 3 dots) can be repeated. (Argument is repeatable)

<u>[expression]</u> ... entire expression within [ ] is repeatable.


## Parts of a man page

**Name** 
- name of command

**Synopsis** 
- How to use the command

When you see file in a man page, think file and or directory

**Description**
 short and long options do the same thing
 ![](../../../images/Pasted%20image%2020240709211540%201.png)

Current section number is printed at the top left of the man page. 

-k to search sections using apropos
```bash
[root@server30 ~]# man -k unlink
mq_unlink (2)        - remove a message queue
mq_unlink (3)        - remove a message queue
mq_unlink (3p)       - remove a message queue (REALT...
sem_unlink (3)       - remove a named semaphore
sem_unlink (3p)      - remove a named semaphore
shm_open (3)         - create/open or unlink POSIX s...
shm_unlink (3)       - create/open or unlink POSIX s...
shm_unlink (3p)      - remove a shared memory object...
unlink (1)           - call the unlink function to r...
unlink (1p)          - call theunlink() function
unlink (2)           - delete a name and possibly th...
unlink (3p)          - remove a directory entry
unlinkat (2)         - delete a name and possibly th...]
```

Shows page number in ()

The sections that end in p are POSIX documentation. Theese are not specific to Linux.

```bash
[root@server30 ~]# man -k "man pages"
lexgrog (1)          - parse header information in man pages
man (7)              - macros to format man pages
man-pages (7)        - conventions for writing Linux man pages
man.man-pages (7)    - macros to format man pages
[root@server30 ~]# man man-pages
```

Use man-pages to learn more about man pages
```bash
Sections within a manual page
       The list below shows conventional or suggested sections.  Most manual
       pages should include at least the highlighted  sections.   Arrange  a
       new manual page so that sections are placed in the order shown in the
       list.

              NAME
              LIBRARY          [Normally only in Sections 2, 3]
              SYNOPSIS
              CONFIGURATION    [Normally only in Section 4]
              DESCRIPTION
              OPTIONS          [Normally only in Sections 1, 8]
              EXIT STATUS      [Normally only in Sections 1, 8]
              RETURN VALUE     [Normally only in Sections 2, 3]
              ERRORS           [Typically only in Sections 2, 3]
              ENVIRONMENT
              FILES
              ATTRIBUTES       [Normally only in Sections 2, 3]
              VERSIONS         [Normally only in Sections 2, 3]
              STANDARDS
              HISTORY
              NOTES
              CAVEATS
              BUGS
              EXAMPLES
              AUTHORS          [Discouraged]
              REPORTING BUGS   [Not used in man-pages]
              COPYRIGHT        [Not used in man-pages]
              SEE ALSO
```

Shell builtins do not have man pages. Look at the shell man page for info on them. 
`man bash`

Search for the Shell Builtins section:
`/SHELL BUILTIN COMMANDS`

![](../../../images/Pasted%20image%2020240710050240%201.png)

You can find help on builtins with the help command:
![](../../../images/Pasted%20image%2020240710050536%201.png)

```bash
david@fedora:~$ help hash
hash: hash [-lr] [-p pathname] [-dt] [name ...]
    Remember or display program locations.
    
    Determine and remember the full pathname of each command NAME.  If
    no arguments are given, information about remembered commands is displayed.
    
    Options:
      -d	forget the remembered location of each NAME
      -l	display in a format that may be reused as input
      -p pathname	use PATHNAME as the full pathname of NAME
      -r	forget all remembered locations
      -t	print the remembered location of each NAME, preceding
    		each location with the corresponding NAME if multiple
    		NAMEs are given
    Arguments:
      NAME	Each NAME is searched for in $PATH and added to the list
    		of remembered commands.
    
    Exit Status:
    Returns success unless NAME is not found or an invalid option is given.
```

`help` without any arguments displays commands you can get help on. 
```bash
david@fedora:~/Documents/davidvargas/davidvargasxyz.github.io$ help help
help: help [-dms] [pattern ...]
    Display information about builtin commands.
    
    Displays brief summaries of builtin commands.  If PATTERN is
    specified, gives detailed help on all commands matching PATTERN,
    otherwise the list of help topics is printed.
    
    Options:
      -d	output short description for each topic
      -m	display usage in pseudo-manpage format
      -s	output only a short usage synopsis for each topic matching
    		PATTERN
    
    Arguments:
      PATTERN	Pattern specifying a help topic
    
    Exit Status:
    Returns success unless PATTERN is not found or an invalid option is given.
```

`type` command tells you what type of command something is. 

Using man on some shell builtins brings you to the bash man page Shell Builtin Section

Many commands support `-h` or `--help` options to get quick info on a command. 
