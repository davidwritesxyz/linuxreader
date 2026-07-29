# Using Rsync to sync a folder between two hosts 

I was using github to sync my notebook with my website. But that introduces some extra steps that rsync can easily eliminate.

## Install rsync 

```
apt install rsync
```

## To upload files from your local computer to your server 

```
rsync -rtvzP /path/to/file root@example.org:/path/on/the/server
```

## Turn the above script into an alias

We will be adding an alias so you do not have to type the entire command above every time. Add the following to ~/.bashrc
```
alias rsyncp='rsync -rtvzP ~/Documents/PerfectDarkMode_html/ root@perfectdarkmode.com:/var/www/PerfectDarkMode_html'
```
Update the source for the new alias to take effect:
```
$ source ~/.bashrc
```
Now simply type "rsyncp" to run the command. 

## Options 

* -r - Recursive
* -t - tranfer modification times (lets you skip files that have not been modified)
* -v - visual
* -z - with file compression
* -p - pick up where you left off if a file fails during transfer

## Download files from another host 

```
rsync -rtvzP root@example.org:/path/to/file /path/to/file
```



