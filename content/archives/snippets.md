+++
title       = "Snippets"
date        = 2024-11-25
weight      = 9900
hidden      = true
+++

SVG to PNG

```
(user)$ rsvg-convert -w 800 input.svg > output.png
```

Write protecting file

```
(root)# chattr +i data.txt
(root)# chattr -i data.txt
```

Android WiFi "Limited Connection"

```
(user)$ adb shell
(user)$ settings put global captive_portal_https_url https://httpstat.us/204
(user)$ settings put global captive_portal_http_url http://httpstat.us/204
```

Extend Library PATH

```
# add dir to library PATH
(user)$ export LD_LIBRARY_PATH=$PWD:$LD_LIBRARY_PATH

# list files in RPM
(user)$ rpm -qlp example.rpm

# extract files from RPM
(user)$ rpm2cpio example.rpm | cpio -idmv
```

Image Converting (need ImageMagick package installed)

```
# convert and resize image
(user)$ convert -scale 800x input.jpg output.webp

# convert image to grayscale
(user)$ convert -colorspace gray input.jpg output.jpg

# convert transparent image to white background
(user)$ convert -background white -flatten input.png output.jpg
```

File Encoding

```
# unzip with specific encoding
(user)$ unzip -O GB18030 <filename> -d <target_dir>

# convert txt encoding
(user)$ iconv -f GB18030 -t UTF-8 < in.txt > out.txt

# urlencode
(user)$ echo "example text" | perl -MURI::Escape -lne 'print uri_escape($_)'
```

Extract .z01

```
(user)$ 7z x example.z01
```

Recursively Change Files and Directories

```
# change permissin
(user)$ find . -type d -exec chmod 755 {} \;
(user)$ find . -type f -exec chmod 644 {} \;

# rename files (rename.ul)
(user)$ find . -type f -name "foo*" -exec rename foo bar {} \;

# rename files (prename)
(user)$ find . -type f -name "foo*" -exec rename 's/foo/bar' '{}' \;
```

Git 

```
# remove untracked files
(user)$ git clean -f -d

# force to use LF (Windows)
(user)$ git config --global core.autocrlf input
(user)$ git config --global core.eol lf
```

