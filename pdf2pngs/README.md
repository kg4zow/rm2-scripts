# `pdf2pngs`

In the "world" of [reMarkable](https://remarkable.com/) tablets, the word "template" refers to a "background image" which can be assigned to pages in notebooks. However, a lot of people use the word "template" when referring to a PDF file. This is because the tablets "treat" the pages in a PDF *as if* they were templates.

After reading [this comment](https://www.reddit.com/r/RemarkableTablet/comments/18vx8a7/pdf_template_help/) on Reddit, it occurred to me that if you take the pages in a PDF and convert them to PNG files, those files could be used as real templates.

This script reads a PDF, and writes a PNG file for each page in the PDF, using the correct size and "colour depth" for a reMarkable tablet.

## Ghostscript

The script uses [Ghostscript](https://ghostscript.com/) to do its work. The computer where you run the script will need Ghostscript's `gs` command installed.

### macOS

The easiest way to install Ghostscript on macOS is to use [Homebrew](https://brew.sh/).

* Install Homebrew. &#x2192; [`https://brew.sh/`](https://brew.sh/]

* `brew install gs`

### Linux

Ghostscript is usually available from your Linux distribution's package repositories, with the name "`ghostscript`".

* CentOS, Fedora, etc. - `yum install ghostscript`

* Debian, Ubuntu, etc. - `apt install ghostscript`

### windows

No idea. See [`https://ghostscript.readthedocs.io/`](https://ghostscript.readthedocs.io/) for information on how to install it.

## Install the script

* Copy the script to a directory in your `PATH`.

    Usually this involves cloning the repo to your system and copying the file, but you *could* also download the file dierctly.

* Make sure the script's permissions allow it to be executed.

```
$ cd ~/git/
$ git clone https://github.com/kg4zow/rm2-scripts
$ cd rm2-scripts/pdf2pngs/
$ cp pdf2pngs ~/bin/
$ chmod 0755 ~/bin/pdf2pngs
```

## Run the script

* `cd` into the directory where you want the PNG files to be written.

* Run the script, with the PDF filename on the command line.

This will print the two `gs` commands that it runs - first to count the pages (so it knows how many digits to use in the page numbers in the output filenames), then to actually *do* the conversion.

```
# mkdir ~/work
$ cd ~/work/
$ pdf2pngs ~/input.pdf
++ gs -q -dNODISPLAY --permit-file-read=input.pdf -c '(input.pdf) (r) file runpdfbegin pdfpagecount = quit'
+ PAGES=5
+ gs -dBATCH -dNOPAUSE -dSAFER -dPDFFitPage -sDEVICE=pnggray -g1404x1872 -sOutputFile=input.%01d.png input.pdf
GPL Ghostscript 10.02.1 (2023-11-01)
Copyright (C) 2023 Artifex Software, Inc.  All rights reserved.
This software is supplied under the GNU AGPLv3 and comes with NO WARRANTY:
see the file COPYING for details.
Processing pages 1 through 5.
Page 1
Page 2
Page 3
Page 4
Page 5
$ ls -1
input.1.png
input.2.png
input.3.png
input.4.png
input.5.png
```

This will write a series of `.png` files, one for each page in the PDF. The files' names will have the same "base" as the PDF, with "`.p01.png`", "`.p02.png`", etc. as the extensions.

These will be 1404x1872 8-bit greyscale PNG files, which are the correct size and colour depth for the reMarkable 1 and 2 tablets.

```
$ magick identify input.1.png
input.1.png PNG 1404x1872 1404x1872+0+0 8-bit Grayscale Gray 256c 22326B 0.000u 0:00.002
```

# License

The `pdf2pngs` script is licensed under the MIT License.

**The MIT License (MIT)**

Copyright &copy; 2024 John Simpson

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
