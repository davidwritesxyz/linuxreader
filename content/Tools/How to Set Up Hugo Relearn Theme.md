+++
title = 'How to Set Up Hugo Relearn Theme'
description = 'Hugo Relearn setup guide'
+++
# Hugo Setup


## Adding a module as a theme

Make sure Go is installed

```
go version
```

Create a new site

```
hugo new site sitename
cd sitename
```

Initialize your site as a module

```
hugo mod init sitename
```

Confirm 
```
cat go.mod
```

Add the module as a dependency using it's git link

```
hugo mod get github.com/McShelby/hugo-theme-relearn
```

Confirm

```
cat go.mod
```

add the theme to config.toml

```
# add this line to config.toml and save
theme = ["github.com/McShelby/hugo-theme-relearn"]
```

Confirm by viewing site

```
hugo serve
# visit browser at http://localhost:1313/ to view site
```

Adding a new "chapter" page

```
hugo new --kind chapter Chapter/_index.md
```

Add a home page

```shell
hugo new --kind home _index.md
```

Add a default page 

```shell
hugo new <chapter>/<name>/_index.md
```

or

```shell
hugo new <chapter>/<name>.md
```


You will need to change some options in _index.md
```
+++
# is this a "chaper"?
chapter=true
archetype = "chapter"
# page title name
title = "Linux"
# The "chapter" number
weight = 1
+++
```

Adding a "content page" under a category

```shell
hugo new basics/first-content.md
```

Create a sub directory:
```
hugo new basics/second-content/_index.md
```

* change draft = true to draft = false in the content page to make a page render.

## Global site parameters
Add these to your config.toml file and edit as you please

```bash
[params]
  # This controls whether submenus will be expanded (true), or collapsed (false) in the
  # menu; if no setting is given, the first menu level is set to false, all others to true;
  # this can be overridden in the pages frontmatter
  alwaysopen = true
  # Prefix URL to edit current page. Will display an "Edit" button on top right hand corner of every page.
  # Useful to give opportunity to people to create merge request for your doc.
  # See the config.toml file from this documentation site to have an example.
  editURL = ""
  # Author of the site, will be used in meta information
  author = ""
  # Description of the site, will be used in meta information
  description = ""
  # Shows a checkmark for visited pages on the menu
  showVisitedLinks = false
  # Disable search function. It will hide search bar
  disableSearch = false
  # Disable search in hidden pages, otherwise they will be shown in search box
  disableSearchHiddenPages = false
  # Disables hidden pages from showing up in the sitemap and on Google (et all), otherwise they may be indexed by search engines
  disableSeoHiddenPages = false
  # Disables hidden pages from showing up on the tags page although the tag term will be displayed even if all pages are hidden
  disableTagHiddenPages = false
  # Javascript and CSS cache are automatically busted when new version of site is generated.
  # Set this to true to disable this behavior (some proxies don't handle well this optimization)
  disableAssetsBusting = false
  # Set this to true if you want to disable generation for generator version meta tags of hugo and the theme;
  # don't forget to also set Hugo's disableHugoGeneratorInject=true, otherwise it will generate a meta tag into your home page
  disableGeneratorVersion = false
  # Set this to true to disable copy-to-clipboard button for inline code.
  disableInlineCopyToClipBoard = false
  # A title for shortcuts in menu is set by default. Set this to true to disable it.
  disableShortcutsTitle = false
  # If set to false, a Home button will appear below the search bar on the menu.
  # It is redirecting to the landing page of the current language if specified. (Default is "/")
  disableLandingPageButton = true
  # When using mulitlingual website, disable the switch language button.
  disableLanguageSwitchingButton = false
  # Hide breadcrumbs in the header and only show the current page title
  disableBreadcrumb = true
  # If set to true, hide table of contents menu in the header of all pages
  disableToc = false
  # If set to false, load the MathJax module on every page regardless if a MathJax shortcode is present
  disableMathJax = false
  # Specifies the remote location of the MathJax js
  customMathJaxURL = "https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"
  # Initialization parameter for MathJax, see MathJax documentation
  mathJaxInitialize = "{}"
  # If set to false, load the Mermaid module on every page regardless if a Mermaid shortcode or Mermaid codefence is present
  disableMermaid = false
  # Specifies the remote location of the Mermaid js
  customMermaidURL = "https://unpkg.com/mermaid/dist/mermaid.min.js"
  # Initialization parameter for Mermaid, see Mermaid documentation
  mermaidInitialize = "{ \"theme\": \"default\" }"
  # If set to false, load the Swagger module on every page regardless if a Swagger shortcode is present
  disableSwagger = false
  # Specifies the remote location of the RapiDoc js
  customSwaggerURL = "https://unpkg.com/rapidoc/dist/rapidoc-min.js"
  # Initialization parameter for Swagger, see RapiDoc documentation
  swaggerInitialize = "{ \"theme\": \"light\" }"
  # Hide Next and Previous page buttons normally displayed full height beside content
  disableNextPrev = true
  # Order sections in menu by "weight" or "title". Default to "weight";
  # this can be overridden in the pages frontmatter
  ordersectionsby = "weight"
  # Change default color scheme with a variant one. Eg. can be "auto", "red", "blue", "green" or an array like [ "blue", "green" ].
  themeVariant = "auto"
  # Change the title separator. Default to "::".
  titleSeparator = "-"
  # If set to true, the menu in the sidebar will be displayed in a collapsible tree view. Although the functionality works with old browsers (IE11), the display of the expander icons is limited to modern browsers
  collapsibleMenu = false
  # If a single page can contain content in multiple languages, add those here
  additionalContentLanguage = [ "en" ]
  # If set to true, no index.html will be appended to prettyURLs; this will cause pages not
  # to be servable from the file system
  disableExplicitIndexURLs = false
  # For external links you can define how they are opened in your browser; this setting will only be applied to the content area but not the shortcut menu
  externalLinkTarget = "_blank"
```

## Syntax highlighting

Supports a variety of [Code Syntaxes]
To select the syntax, wrap the code in backticks and place the syntax by the first set of backticks.
```
```bash
echo hello
\```
```

## Adding tags

Tags are displayed in order at the top of the page. They will also display using the menu shortcut made further down.

Add tags to a page:

```toml
+++
tags = ["tutorial", "theme"]
title = "Theme tutorial"
weight = 15
+++
```

## Choose a default color [theme](https://xyproto.github.io/splash/docs/all.html)
Add to config.toml with the chosen theme for the "style" option:
```toml
[markup]
  [markup.highlight]
    # if `guessSyntax = true`, there will be no unstyled code even if no language
    # was given BUT Mermaid and Math codefences will not work anymore! So this is a
    # mandatory setting for your site if you want to use Mermaid or Math codefences
    guessSyntax = false

    # choose a color theme or create your own
    style = "base16-snazzy"
```

## Add Print option and search output page.
add the following to config.toml
```
[outputs]
  home = ["HTML", "RSS", "PRINT", "SEARCH"]
  section = ["HTML", "RSS", "PRINT"]
  page = ["HTML", "RSS", "PRINT"]
```

## Customization

This theme has a bunch of editable customizations called partials. You can overwrite the default partials by putting new ones in /layouts/partials/. 

to customize "partials", create a  "partials" directory under site/layouts/
```
cd layouts
mkdir partials
cd partials
```

You can find all of the partials available for this theme [here](https://mcshelby.github.io/hugo-theme-relearn/basics/customization/index.html)

### Change the site logo using the logo.html partial

Create logo.html in /layouts/partials
```
vim logo.html
```

Add the content you want in html. This can be an img html tag referencing an image in the static folder. Or even basic text. Here is the basic syntax of an html page, adding "Perfect Dark Mode" as the text to display:
```
<!DOCTYPE html>
<html>
<body>

<h3>Perfect Dark Mode</h3>

</body>
</html>
```

### Add a favicon to your site

* This is pasted from the relearn site. Add Favicon and edit *
If your favicon is a SVG, PNG or ICO, just drop off your image in your local `static/images/` folder and name it `favicon.svg`, `favicon.png` or `favicon.ico` respectively.

If no favicon file is found, the theme will lookup the alternative filename `logo` in the same location and will repeat the search for the list of supported file types.

If you need to change this default behavior, create a new file in `layouts/partials/` named `favicon.html`. Then write something like this:

```html
<link rel="icon" href="/images/favicon.bmp" type="image/bmp">
```

## Changing theme colors

In your config.toml file edit the themeVariant option under [params]

```toml
  themeVariant = "relearn-dark"
```

There are some options to choose from or you can custom make your theme colors by using this [stylesheet generator](https://mcshelby.github.io/hugo-theme-relearn/basics/generator/index.html)


Menu Shortcuts
Add a [[menu.shortcuts]] entry for each link

```toml
[[menu.shortcuts]]
name = "<i class='fab fa-fw fa-github'></i> GitHub repo"
identifier = "ds"
url = "https://github.com/McShelby/hugo-theme-relearn"
weight = 10

[[menu.shortcuts]]
name = "<i class='fas fa-fw fa-camera'></i> Showcases"
url = "more/showcase/"
weight = 11

[[menu.shortcuts]]
name = "<i class='fas fa-fw fa-bookmark'></i> Hugo Documentation"
identifier = "hugodoc"
url = "https://gohugo.io/"
weight = 20

[[menu.shortcuts]]
name = "<i class='fas fa-fw fa-bullhorn'></i> Credits"
url = "more/credits/"
weight = 30

[[menu.shortcuts]]
name = "<i class='fas fa-fw fa-tags'></i> Tags"
url = "tags/"
weight = 40
```

Extras

Menu button arrows. (Add to page frontmatter)
```
menuPre = "<i class='fa-fw fas fa-caret-right'></i> "
```