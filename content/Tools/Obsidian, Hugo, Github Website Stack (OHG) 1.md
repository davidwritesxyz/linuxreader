First, you'll need to create a github account. If you won't be using a custom domain (You can add one later), make sure you name the account what you want your website to be. The domain will be {{ account-name.github.io }}

Once you have an account, the first thing you will do is generate a new ssh key and add it to github. This will give you passwordless access to your github account from your desktop. From my Fedora Linux machine, I will run:  
`ssh-keygen -t ed25519 -C "exampleemail@email.com"`

When it asks for the name of the key, give it a unique name to use for this account. I just picked the github account name to make it easy to identify:  
`id_ed25519_example`

Then, grab your public key from ~/.ssh/id_ed25519_example.pub and add it in github under settings > SSH and GPG Keys > New SSH Key

![](../../images/Pasted%20image%2020251019034622%201.png)


Next, go to your github homepage and select "Create repository". The repository name needs to match your github account name from earlier. It also must have public visibility:

![](../../images/Pasted%20image%2020260226034623.png)

From your desktop, create a directory with {{ reponame.github.io }}. You must use this format for this to work:
`mkdir linuxreader.github.io && cd linuxreader.github.io`

Follow the instructions on github for "…or create a new repository on the command line". 

```bash
echo "# linuxreader" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:davidwritesxyz/linuxreader.git
git push -u origin main
git remote set-url origin git@github.com:davidwritesxyz/linuxreader.git

```

Then add credentials:  
```bash
git config user.name "username"
git config user.email "username@email.com"
```

If you have multiple github accounts, you'll have to create aliases for each one in the ~/.ssh/config file:
```bash
Host linuxreader.github.com
    HostName github.com
    IdentityFile ~/.ssh/id_ed25519

Host secondaccount.github.com
    HostName github.com
    IdentityFile ~/.ssh/id_ed25519_secondaccount

```

Then add your private keys to the ssh agent:
```bash
ssh-add ~/.ssh/id_ed25519
ssh-add ~/.ssh/id_ed25519_repo
```

Then add the username and email settings while in the new project directory:
```bash
everythingelsexyz.github.io on  main 
❯ git config user.name "username"

everythingelsexyz.github.io on  main 
❯ git config user.email "example@email.com"
```

Then you will need cd to each repo and add an alias:
```bash
linuxreader.github.io on  main [⇡] via 🐹 via  
❯ git remote set-url origin git@linuxreader.github.com:linuxreader/linuxreader.git

everythingelsexyz.github.io on  main 
git remote set-url origin git@everythingelsexyz.github.com:everythingelsexyz/everythingelsexyz.git
```

Load the keys into the agent:
```bash
ssh-add ~/.ssh/id_ed25519
ssh-add ~/.ssh/id_ed25519_repo
```

## Adding a Hugo Theme to your Github Repository

Next, initialize the directory with Hugo
`cd ~/Documents && hugo new site linuxreader.github.io --force`

You should now have everything needed in your repository directory:
```bash
~/Documents 
❯ cd linuxreader.github.io/

linuxreader.github.io on  main [?] 
❯ ls
archetypes  content  hugo.toml  layouts    static
assets      data     i18n       README.md  themes

```

### Download the theme

There are many ways of adding a theme to Hugo. I prefer manually downloading the theme. Then just adding it to the themes directory. Then you just need to reference the theme version in your hugo.toml file. 

I chose the Relearn theme because it's the best theme I found that works great with Obsidian. Just go to the repo, on the right you'll see "Releases" 

![](../../images/Pasted%20image%2020260226042031.png)

I'm on Linux so I'll download the tar.gx file. 
![](../../images/Pasted%20image%2020260226042139.png)

Move the file to your themes directory:
`mv hugo*tar* ~/Documents/linuxreader.github.io/themes`

Then, extract the files and remove the tarball:
```bash
~/Downloads 
❯ cd ~/Documents/linuxreader.github.io/themes

linuxreader.github.io/themes on  main [?] 
❯ ls
hugo-theme-relearn-9.0.3.tar.gz

linuxreader.github.io/themes on  main [?] 
❯ tar -xf hugo-theme-relearn-9.0.3.tar.gz 

linuxreader.github.io/themes on  main [?] 
❯ ls
hugo-theme-relearn-9.0.3  hugo-theme-relearn-9.0.3.tar.gz

linuxreader.github.io/themes on  main [?] 
❯ rm hugo*tar*

```

### hugo.yaml

Now that we have the basic structure and theme laid out for the site, we need to add some stuff to the hugo.toml file. Which is basically just the settings for your site. 

I want to use yaml instead so I just change the name of the file to hugo.yaml
```bash
❯ mv hugo.toml hugo.yaml
```

I'll start with the basics. Update the baseURL, language, title, and add the theme name from earlier. 

```yml
baseURL: 'https://linuxreader.com/'
languageCode: 'en-us'
title: 'Linux Documentation Site'
theme: ["hugo-theme-relearn-9.0.3"]
```

For github pages specifically, we need our publish directory to be "docs".

```yml
baseURL: 'https://linuxreader.com/'
languageCode: 'en-us'
title: 'Linux Documentation Site'
theme: ["hugo-theme-relearn-9.0.3"]
publishDir: "docs"
```

Next, I added some customization options under [params] in hugo.yaml like this:
```yaml
[params]
  linktitle: 'Linux Reader'
  imagesDir: "content/images"
  lightbox: false
  alwaysopen: false
  editURL: ""
  author.name: "David Thomas"
  description: "Linux, Tech, Open Source Software, Privacy"
  showVisitedLinks: false
  disableSearch: false
  disableSearchHiddenPages: false
  disableSeoHiddenPages: false
  disableTagHiddenPages: false
  disableAssetsBusting: false
  disableGeneratorVersion: false
  disableInlineCopyToClipBoard: false
  disableShortcutsTitle: false
  disableLandingPageButton: true
  disableLanguageSwitchingButton: false
  disableMarkdownButton: false
  disableBreadcrumb: true
  disableToc: false
  disableNextPrev: false
  ordersectionsby: "weight"
  titleSeparator: "::"
  highlightWrap: true
  collapsibleMenu: false
  disableExplicitIndexURLs: true
  externalLinkTarget: "_blank"
```

Then, set up a dummy \_index.md file so we can test what we have so far.
`echo "Hello World" > content/_index.md`

Serve the site locally:
`hugo serve`

You should see something like this:
```bash

                  │ EN 
──────────────────┼────
 Pages            │  9 
 Paginator pages  │  0 
 Non-page files   │  0 
 Static files     │  0 
 Processed images │  0 
 Aliases          │  0 
 Cleaned          │  0 

Built in 33 ms
Environment: "development"
Serving pages from disk
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1) 
Press Ctrl+C to stop
```

Grab the url it gives (http://localhost:1313/ ), and open it in your browser:
![](../../images/Pasted%20image%2020260226050419.png)

At this point, I'm going to push the changes to github to make sure things are set up right:
`git add . && git commit -m "First Update" && git push`

Everything is up to date remotely! 
![](../../images/Pasted%20image%2020260226051225.png)

## Setting up Obsidian

Since I also keep private files in my Obsidian vault, I don't like to use the git repository as the vault. Since the repository must be set to public in order for it to work with Github pages.

Instead, I use a private vault, and set up a simple script to rsync the proper folders to my content directory.
## Setting up GitHub Pages

Go through quick setup here: https://pages.github.com/

Hugo Setup: https://gohugo.io/hosting-and-deployment/hosting-on-github/

Here's how to set up a free website using Github pages and Hugo. You can add a custom domain name using [Cloudflare](https://www.cloudflare.com/) for about $10 per year. 

The general steps include:
1. Create a Github repository for your site. 
2. Change your pages settings.
3. Build the repo into a Hugo site. 
4. Add DNS records for a custom domain. 
