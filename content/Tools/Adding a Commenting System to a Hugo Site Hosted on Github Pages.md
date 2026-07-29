+++
title = " "
draft = "true"
menuPre = "<i class='fa-fw fas fa-caret-right'></i> "
+++

Requirements:
Privacy friendly.
Free to start out with.
Notify people when their comment is replied to. 
Not require sign in to comment


Should you add comments to your blog? https://blog.codinghorror.com/a-blog-without-comments-is-not-a-blog/

Potentials: 
- https://cusdis.com/

Staticman
- open source
- needs a server
- self-hosted

Found a really cool post breaking down the pros and cons of Giscus as well as some requirements to look out for https://cdwilson.dev/articles/using-giscus-for-comments-in-hugo/

And another post breaking down many different options: https://darekkay.com/blog/static-site-comments/

Giscus Setup:

Fill out the form here: https://giscus.app/

Add the generated \<script\> tag to the sites template: layouts/posts/giscus.html

```html
{{- if isset .Site.Params "giscus" -}}
  {{- if and (isset .Site.Params.giscus "repo") (not (eq .Site.Params.giscus.repo "" )) (eq (.Params.disable_comments | default false) false) -}}
  <script src="https://giscus.app/client.js"
    data-repo="{{ .Site.Params.giscus.repo }}"
    data-repo-id="{{ .Site.Params.giscus.repoID }}"
    data-category="{{ .Site.Params.giscus.category }}"
    data-category-id="{{ .Site.Params.giscus.categoryID }}"
    data-mapping="{{ default "url" .Site.Params.giscus.mapping }}"
    data-reactions-enabled="{{ default "1" .Site.Params.giscus.reactionsEnabled }}"
    data-emit-metadata="{{ default "0" .Site.Params.giscus.emitMetadata }}"
    data-input-position="{{ default "top" .Site.Params.giscus.inputPosition }}"
    data-theme="{{ default "dark" .Site.Params.giscus.theme }}"
    data-lang="{{ default "en" .Site.Params.giscus.lang }}"
    data-loading="{{ default "lazy" .Site.Params.giscus.loading }}"
    crossorigin="anonymous"
    async>
  </script>
  {{- end -}}
{{- end -}}
```


```bash
<script src="https://giscus.app/client.js"
        data-repo="DavidVargasxyz/davidvargasxyz.github.io"
        data-repo-id="R_kgDOM0-g3g"
        data-category="General"
        data-category-id="DIC_kwDOM0-g3s4Cj5Rk"
        data-mapping="url"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="top"
        data-theme="dark"
        data-lang=en""
        data-loading="lazy"
        crossorigin="anonymous"
        async>
</script>
```

include this partial in the footer of the post template (e.g. `layouts/posts/single.html`):
```html
<footer>
  {{ partial "posts/giscus.html" . }}
</footer>
```

configure the options in the site’s `config.toml` file:
```toml
[params.giscus]
repo = "DavidVargasxyz/davidvargasxyz.github.io"
repoID = "R_kgDOM0-g3g"
category = "General"
categoryID = "DIC_kwDOM0-g3s4Cj5Rk"
mapping = "url"
reactionsEnabled = "1"
emitMetadata = "0"
inputPosition = "top"
theme = "dark"
lang = "en"
loading = "lazy"
```

relearn has a custom-comments partial and I added:
```
{{- /* Comments area start */ -}}
 
{{ partial "giscus" . }}

{{- /* Comments area end */ -}}
```                                         


Some guide I'm trying:
https://morosanu.io/posts/adding-comments-to-my-blog/
https://blog.mrhaydendp.com/posts/adding-giscus-to-hugo-site/
https://www.justinjbird.me/blog/2023/adding-comments-to-a-hugo-site-using-giscus/#configure-giscus-codeblock

Problems with Giscus:
- Requires people to sign in with Github. 
- Doesn't email user when you reply to comment.
- Themes don't match my website

cusdis
```html
<div id="cusdis_thread"
  data-host="https://cusdis.com"
  data-app-id="6b9b747a-1ebb-4d6a-930a-90c6da53ed6e"
  data-page-id="{{ .Permalink }}"
  data-page-url="{{ .Permalink }}"
  data-page-title="{{ .Title }}"
  data-theme="dark"
></div>
<script async defer src="https://cusdis.com/js/cusdis.es.js"></script>

<script>
const observer = new MutationObserver((mutations) => {
    mutations.forEach((mutation) => {
        if (mutation.addedNodes.length == 0) return;
        const iframe = mutation.addedNodes[0];
        if (iframe.tagName !== 'IFRAME') return;

        const additionalStyles = `
        .dark {
            background: #100E17;
        }
        `;
        
        iframe.srcdoc = iframe.srcdoc.replace('</style>', additionalStyles + '</style>');
    });
});
observer.observe(document.getElementById("cusdis_thread"), { childList: true, subtree: true });
</script>
```

```html
  <script async defer src="https://cusdis.com/js/cusdis.es.js"></script>

    <script>
    const observer = new MutationObserver((mutations) => {
        mutations.forEach((mutation) => {
            if (mutation.addedNodes.length == 0) return;
            const iframe = mutation.addedNodes[0];
            if (iframe.tagName !== 'IFRAME') return;

            const additionalStyles = `
            .dark {
                background: #100E17;
            }
            `;
            
            iframe.srcdoc = iframe.srcdoc.replace('</style>', additionalStyles + '</style>');
        });
    });
    observer.observe(document.getElementById("cusdis_thread"), { childList: true, subtree: true });
    </script>
```

Works with menu in content-footer.html
```html
<div id="cusdis_thread"
  data-host="https://cusdis.com"
  data-app-id="6b9b747a-1ebb-4d6a-930a-90c6da53ed6e"
  data-page-id="{{ .Permalink }}"
  data-page-url="{{ .Permalink }}"
  data-page-title="{{ .Title }}"
  data-theme="dark"
></div>
<script async defer src="https://cusdis.com/js/cusdis.es.js"></script>
```

Problems with Cusdis:
- Don't receive user's inputted email address.
- Doesn't notify user when you have replied. 
- Little documentation on how to customize. Leading to very small window size. 

isso

https://isso-comments.de/

Potential self hosted solution to try on a server.

Commento
https://commento.io/

$10 per month. 

### Chirpy 

https://chirpy.dev/dashboard/davidvargasxyz/davidvargas.xyz/get-started
A privacy-friendly, customizable and open-source service. Public beta as of April 2022.

Get started

### Usage on any website

Chirpy is not just your average comment system - it's a feature-packed tool with powerful **built-in analytics**. With Chirpy, you can seamlessly integrate the analytics service and save time and effort.

Getting started is a breeze. Simply follow these steps:

1. Copy the script provided below and paste it into the body of your HTML, make sure it will be loaded on every page.
    
    1<script defer src="https://chirpy.dev/bootstrapper.js" data-chirpy-domain="davidvargas.xyz"></script>
    
2. Then, paste this html element to any page that should render the comment widget:
    
<!--
    
The widget follows "system" settings for light/dark mode
    
by default, but you can manually select "light" or "dark"
    
mode for a consistent display
    
-->
    
<div
    
data-chirpy-theme="system"
    
data-chirpy-comment="true"
    
id="chirpy-comment"
    
></div>

https://commentbox.io/docs
5745952027574272-proj


```html
<div class="commentbox"></div>
<script src="https://unpkg.com/commentbox.io/dist/commentBox.min.js"></script>
<script>commentBox('5745952027574272-proj')</script>
```

https://docs.convocomet.dev/guides/custom-widget/
- Easy setup
- Can comment with our without account. 
- Stock widget is bulky

Or just don't use comments https://coywolf.blog/the-case-against-comments/