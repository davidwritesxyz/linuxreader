+++
title = 'How to Build a website With Hugo'
description = 'Guide for setting up a static website with Hugo'
+++

Word Press is great, but it is probably a lot more bloated then you need for a personal website. Enter Hugo, it has less server capacity and storage needs than Word Press. Hugo is a static site generator than takes markdown files and converts them to html. 

Hosting your own website is also a lot cheaper than having a provider like Bluehost do it for you. Instead of \$15 per month, I am currently paying \$10 per year. 

This guide will walk through building a website step-by-step. 

1. Setting up a Virtual Private Server (VPS)
2. Registering a domain name
3. Pointing the domain to your server
4. Setting up hugo on your local PC
5. Syncing your Hugo generate site with your server
6. Using nginx to serve your site
7. Enable http over SSL

## Setting up a Virtual Private Server (VPS)

I use Vultr as my VPS. When I signed up they had a $250 credit towards a new account. If you select the cheapest server (you shouldn't need anything else for a basic site) that comes out to about $6 a month. Of course the $250 credit goes towards that which equates to around 41 months free. 

Head to vultr.com. Create and account and  Select the Cloud Compute option. 

Under CPU & Storage Technology, select "Regular Performance". Then under "Server Location, select the server closest to you. Or closest to where you think your main audience will be. 

Under Server image, select the OS you are most comfortable with. This guide uses Debian. 

Under Server Size, slect the 10GB SSD. Do not select the "IPv6 ONLY" option. Leave the other options as default and enter your server hostname.

On the products page, click your new server. You can find your server credentials and IPv4 address here. You will need these to log in to your server.

Log into your sever via ssh to test. From a Linux terminal run:
```
ssh username@serveripaddress
```

Then, enter your password when prompted. 

## Registering a Domain Name

I got my domain perfectdarkmode.com from Cloudflare.com for about $10 per year. You can check to see available domains there. You can also check https://www.namecheckr.com/ to see iof that name is available on various social media sites. 

In CloudFlare, just click "add a site" and pick a domain that works for you. Next, you will need your server address from earlier. 

Under domain Registration, click "Manage Domains", click "manage" on your domain. One the sidebar to the right, there is a qucik actions menu. Click "update DNS configuration".

Click "Add record". Type is an "A" record. Enter the name and the ip address that you used earlier for your server. Uncheck "Proxy Status" and save.

You can check to see if your DNS has updated on various DNS severs at https://dnschecker.org/. Once those are up to date (after a couple minutes) you should be able to ping your new domain. 
```
$ ping perfectdarkmode.com
PING perfectdarkmode.com (104.238.140.131) 56(84) bytes of data.
64 bytes from 104.238.140.131.vultrusercontent.com (104.238.140.131): icmp_seq=1 ttl=53 time=33.2 ms
64 bytes from 104.238.140.131.vultrusercontent.com (104.238.140.131): icmp_seq=2 ttl=53 time=28.2 ms
64 bytes from 104.238.140.131.vultrusercontent.com (104.238.140.131): icmp_seq=3 ttl=53 time=31.0 ms
```
Now, you can use the same ssh command to ssh into your vultr serverusing your domain name.
```
ssh username@domain.com
```
## Setting up hugo on your local PC

Hugo is a popular open-source static site generator. It allows you to take markdown files, and builds them into and html website. To start go to https://gohugo.io/installation/ and download Hugo on your local computer. (I will show you how to upload the site to your server later.)


Pick a theme
The theme I use is here https://themes.gohugo.io/themes/hugo-theme-hello-friend-ng/

You can browse your own themes as well. Just make sure to follow the installation instructions. Let's create a new Hugo site. Change into the directory where you want your site to be located in. Mine rests in ~/Documents/.

```
cd ~/Documents/
```

Create your new Hugo site.

```
hugo new site site-name
```

This will make a new folder with your site name in the ~/Documents directory. This folder will have a few directories and a config file in it. 

```
archetypes  config.toml  content  data  layouts  public  resources  static  themes
```

For this tutorial, we will be working with the config.toml file and the  content, public, static, and themes. Next, load the theme into your site directory. For the Hello Friend NG theme:

```
git clone https://github.com/rhazdon/hugo-theme-hello-friend-ng.git themes/hello-friend-ng
```

Now we will load the example site into our working site. Say yes to overwrite. 

```
cp -a themes/hello-friend-ng/exampleSite/* .
```

The top of your new config.toml site now contains:

```
baseURL = "https://example.com"
title   = "Hello Friend NG"
languageCode = "en-us"
theme = "hello-friend-ng"
```

Replace your baseURL with your site name and give your site a title. Set the enableGlobalLanguageMenu option to false if you want to remove the language swithcer option at the top. I also set enableThemeToggle to true so users could set the theme to dark or light.

You can also fill in the links to your social handles. Comment out any lines you don't want with a "#" like so:

```
[params.social](params.social)
    name = "twitter"
    url  = "https://twitter.com/"

  [params.social](params.social)
    name = "email"
    url  = "mailto:nobody@example.com"

  [params.social](params.social)
    name = "github"
    url  = "https://github.com/"

  [params.social](params.social)
    name = "linkedin"
    url  = "https://www.linkedin.com/"

 # [params.social](params.social)
   # name = "stackoverflow"
   # url  = "https://www.stackoverflow.com/"
```

You may also want to edit the footer text to your liking. I commented out the second line that comes with the example site:

```
[params.footer]
    trademark = true
    rss = true
    copyright = true
    author = true

    topText = []
    bottomText = [
     # "Powered by <a href=\"http://gohugo.io\">Hugo</a>",
     #  "Made with &#10084; by <a href=\"https://github.com/rhazdon\">Djordje Atlialp</a>"
    ]
```

Now, move the contents of the example contents folder over to your site's contents folder (giggidy):

```
cp -r ~/Documents/hugo/themes/hello-friend-ng/exampleSite/content/* ~/Documents/hugo/content/
```

Let's clean up a little bit. Cd into ~/Documents/hugo/content/posts. Rename the file to the name of your first post. Also, delete all of the other files here:

```
cd ~/Documents/hugo/contents/posts
mv goisforlovers.md newpostnamehere.md
find . ! -name 'newpostnamehere.md' -type f -exec rm -f {} +
```

Open the new post file and delete everything after this:

```
+++
title = "Building a Minimalist Website with Hugo"
description = ""
type = ["posts","post"]
tags = [
    "hugo",
    "nginx",
    "ssl",
    "http",
    "vultr",
]
date = "2023-03-26"
categories = [
    "tools",
    "linux",
]
series = ["tools"]
[ author ]
  name = "David Thomas"
+++
```

You will need to fill out this header information for each new post you make. This will allow you to give your site a title, tags, date, categories, etc. This is what is called a TOML header. TOML stands for Tom's Obvious Minimal Language. Which is a minimal language used for parsing data. Hugo uses TOML to fill out your site.

Save your doc and exit. Next, there should be an about.md page now in your ~/Documents/hugo/Contents folder. Edit this to edit your about page for your site. You can use this Markdown Guide if you need help learning markdown language. https://www.markdownguide.org/

### Serve your website locally

Let's test the website by serving it locally and accessing it at localhost:1313 in your web browser. Enter the command:

```
hugo serve
```

Hugo will now be generating your website. You can view it by entering localhost:1313 in your webbrowser.


You can use this to test new changes before uploading them to your server. When you svae a post or page file such as your about page, hugo will automatically update the changes to this local page if the local server is running. 

Press "Ctrl + c" to stop this local server. This is only for testing and does not need to be running to make your site work. 

### Build out your public directory

Okay, your website is working locally, how do we get it to your server to host it online? We are almost there. First, we will use the hugo command to build your website in the public folder. Then, we will make a copy of our public folder on our server using rsync. I will also show you how to create an alias so you do not have to remember the rsync command every time. 

From your hugo site folder run:

```
hugo
```

Next, we will put your public hugo folder into /var/www/ on your server. Here is how to do that with an alias. Open ~/.bashrc.

```
vim ~/.bashrc
```

Add the following line to the end of the file, making sure to replace the username and server name:

```
# My custom aliases
alias rsyncp='rsync -rtvzP ~/Documents/hugo/public/ username@myserver.com:/var/www/public'
```

Save and exit the file. Then tell bash to update it's source config file.

```
source ~/.bashrc
```

Now your can run the command by just using the new alias any time. Your will need to do this every time you update your site locally. 

```
rsyncp
```

## Set up nginx on your server
    
Install nginx

```
apt update
apt upgrade
apt install nginx
```

create an nginx config file in /etc/nginx/sites-available/

```
vim /etc/nginx/sites-available/public
```

You will need to add the following to the file, update the options, then save and exit:

```
server {
        listen 80 ;
        listen [::]:80 ;
        server_name example.org ;
        root /var/www/mysite ;
        index index.html index.htm index.nginx-debian.html ;
        location / {
                try_files $uri $uri/ =404 ;
        }
}
```

Enter your domain in "server_name" line in place of "example.org". Also, point "root" to your new site file from earlier. (/var/www/public). Then save and exit.

Link this site-available config file to sites-enabled to enable it. Then restart nginx:

```
ln -s /etc/nginx/sites-available/public /etc/nginx/sites-enabled
systemctl reload nginx
```
### Access Permissions

We will need to make sure nginx has permissions to your site folder so that it can access them to serve your site. Run:

```
chmod 777 /var/www/public
```

### Firewall Permissions

You will need to make sure your firewall allows port 80 and 443. Vultr installs the ufw program by default. But your can install it if you used a different provider. Beware, enabling a firewalll could block you from accessing your vm, so do your research before tinkering outside of these instructions. 

```
ufw allow 80
ufw allow 443
```

### Nginx Security

We will want to hide your nginx version number on error pages. This will make your site a bit harder for hackers to find exploits. Open your Nginx config file at /etc/nginx/nginx.conf and remove the "#" before "server_tokens off;"

Enter your domain into your browser. Congrats! You now have a running website!

## Use Certbot to enable HTTPS

Right now, our site uses the unencrypted http. We want it to use the encrypted version HTTPS (HTTP over SSL). This will increase user privacy, hide usernames and passwords used on your site, and you get the lock symbol by your URL name instead of "!not secure".

### Install Certbot and It's Nginx Module

```
apt install python3-certbot-nginx
```

### Run certbot

```
certbot --nginx
```

Fill out the information, certbot asks for your emaill so it can send you a reminder when the certs need to be renewed every 3 months. You do not need to consent to giving your email to the EFF. Press 1 to select your domain. And 2 too redirect all connections to HTTPS.

Certbot will build out some information in your site's config file. Refresh your site. You should see your new fancy lock icon. 

### Set Up a Cronjob to automatically Renew certbot certs

```
crontab -e
```

Select a text editor and add this line to the end of the file. Then save and exit the file:

```
0 0 1 * * certbot --nginx renew
```

You now have a running website. Just make new posts locally, the run "hugo" to rebuild the site. And use the rsync alias to update the folder on your server. I will soon be making tutorials on making an email address for your domain, such as david@perfectdarkmode.com on my site. I will also be adding a comments section, RSS feed, email subscription, sidebar, and more.

Feel free to reach out with any questions if you get stuck. This is meant to be an all encompassing guide. So I want it to work. 

## Extras

### Optimizing images

Create assets folder in main directory.

Create images folder in /assets

Access image using hugo pipes
```
{{ $image := resources.Get "images/test-image.jpg" }}
<img src="{{ ( $image.Resize "500x" ).RelPermalink }}" />
```

https://gohugo.io/content-management/image-processing/




