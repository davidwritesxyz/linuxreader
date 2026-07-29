## Convert .MOV to MP4
All .MOV files in  a folder
```
for f in *.MOV; do ffmpeg -i $f $f.mp4; done
```

More info at Techwalla https://www.techwalla.com/articles/how-to-convert-mov-to-mp4-with-ffmpeg-on-linux

## Convert .HEIC to .jpg

Fedora Install
```
sudo dnf install -y https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install libheif
sudo dnf install libheif-tools
```
Convert all .HEIC files 
```
for f in *.HEIC; do heif-convert -q 100 $f $f.jpg; done
```

See https://linuxnightly.com/convert-heif-images-to-jpg-or-png-on-linux/ for more options.


