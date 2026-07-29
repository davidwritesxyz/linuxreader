# 

# Pandoc
Used for converting file formats

https://ins3cure.com/pandoc-on-rhel-8/

Go to https://github.com/jgm/pandoc/releases/latest, download the latest tarball (currently pandoc-2.17.0.1-linux-64.tar.gz) and run:
```
sudo tar xvzf pandoc-2.17.0.1-linux-64.tar.gz --strip-components 1 -C /usr/local/
```

install TeX to be able to generate PDF output:
```
sudo yum -y install texlive
```
## Basic useage
```
pandoc inputfile -f inputfiletype -t outputfiletype -o outputfile
```

## Convert full folder
```
for f in *; do pandoc "$f" -s -o "../bloghtml/${f%}.html"; done
```

## Convert epub to markdown and migrate to Obsidian

Run `pandoc bookname.epub -o bookname.md --extract-media=images`

This will create the file and an image folder. Place the image folder in your Obsidian attachments folder. Also, place the markdown file in your Obsidian vault and Obsidian will see all of the image references. 

Then, you can run Obsidian's [Text Extractor](../../Personal/obsidian/image_to_text.md) to pull the text from the image to your clipboard. 



