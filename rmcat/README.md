# rmcat

Combines the pages from one or more reMarkable notebooks, into a single notebook.

The script can read, write, and convert between ...

* `.rmn` files, used by [RCU](http://www.davisr.me/projects/rcu/)

* `.rmdoc` files, used by the reMarkable tablet's built-in software, version 3.10 and later. (Note that 3.9's web interface *has* a "Download to Archive" option, but there's a bug in the web interface code - this actually downloads a PDF file.)

The script can also be used to convert `.rmn` files to `.rmdoc` and vice-versa, by "combining" one input file into an output file of the other type. This can be useful if you're using [drawj2d](https://drawj2d.sourceforge.io/) to generate pages containing shapes (or in my case, text that looks better than my lousy handwriting) - if you don't have RCU to upload the resulting `.rmn` files to your tablet, you can convert the `.rmn` file to an `.rmdoc` file and upload it through the tablet's built-in web interface.

### Custom Templates

If an input document has any pages which use custom templates ...

* If you use RCU to download the document, the `.rmn` file will *contain* an `.rmt` file for each custom template used by one or more pages in the document.

* If you use the tablet's web interface to download the document, the custom template will not be present in the `.rmdoc` file.

In both cases, embedded `.rmt` files will not be included in the output files. This means that pages in the input notebooks which *use* those templates, will have "blank" templates if uploaded to a different tablet, or if the original template has been deleted.

### PDF-backed Notebooks

The script cannot PDF-backed notebooks (i.e. which were created by uploading a PDF or EPUB file), with any other notebooks, whether PDF-backed or otherwise.

## Installation

The script itself is just a single file. It can be "installed" anywhere you like, but you'll probably want it in a directory in your `PATH`.

Wherever you install it, make sure it has executable permissions.

```
chmod +x rmcat
```

### Perl Modules

The script is written in Perl, and requires the following modules:

* [`File::Temp`](https://metacpan.org/pod/File::Temp)
* [`Getopt::Long`](https://metacpan.org/pod/Getopt::Long)
* [`JSON`](https://metacpan.org/pod/JSON)

Modules you don't already have on your system *might* be installable from your system's package repository.

* **CentOS/RHEL 7**, and probably also RHEL 8/9, Fedora, [Alma Linux](https://almalinux.org/), [Rocky Linux](https://rockylinux.org/), and other RedHat-flavoured distros

    ```
    yum install perl perl-File-Temp perl-Getopt-Long perl-JSON
    ```

* **Debian 12** - Debian includes the `File::Temp` and `Getopt::Long` modules in the `perl` package itself, so only `JSON` needs to be added.

    ```
    apt install perl libjson-perl
    ```

* **macOS** - comes with Perl itself, newer versions can be installed using [Homebrew](https://brew.sh/) if desired. Modules need to be installed using CPAN.

If any modules are not available using your system's package manager, you can install them using CPAN. [This page](https://www.cpan.org/modules/INSTALL.html) explains how to install modules from CPAN.

Note that if a module is available from your system's package manager, you should use that instead of CPAN.

## Using the script

The script is designed to be run from a command line. The basic pattern is this:

```
rmcat -o OUTFILE INFILE INFILE INFILE
```

The script requires exactly one "`-o OUTFILE`" option, which tells it the output file to create. The output filename should end with "`.rmdoc`" or "`.rmn`", so it knows  which output format to write. If you need to write an output file with some other name, add the `-d` or `-r` option to explicitly tell it which output format to use.

The script also requires at least one input file. The input files' names must end with "`.rmdoc`" or "`.rmn`", so the script knows how to uncompress each one.

Other options exist, you can run "`rmcat -h`" to see them.

## Examples

You can use the tablet's built-in web interface to download the notebooks as `.rmdoc` files, or use [RCU](http://www.davisr.me/projects/rcu/) to download them as `.rmn` files.

For these examples, I created dummy notebooks on a tablet called `aaa`, `bbb`, and `ccc`. I used the tablet's built-in web interface to download `.rmdoc` "Archive" files, and used RCU to download `.rmn` files.

### Combine notebooks

You can specify the same input file multiple times to build a notebook containing multiple copies of it.

```
$ rmcat -o new.rmdoc aaa.rmn bbb.rmdoc ccc.rmdoc
READ    aaa.rmn
READ    bbb.rmdoc
READ    ccc.rmdoc
CREATE  new.rmdoc
```

The pages in `out.rmdoc` will be the same order they were in the input notebooks, in the order they were listed on the command line. In this example, if `aaa` has two pages, `bbb` has three, and `ccc` has four, the new notebook would contain pages (A1, A2, B1, B2, B3, C1, C2, C3, C4) in that order.

### Duplicate pages

This builds a notebook containing multiple copies of the same input notebook.

```
$ rmcat -o a3.rmdoc aaa.rmdoc aaa.rmdoc aaa.rmdoc
READ    aaa.rmdoc
READ    aaa.rmdoc
READ    aaa.rmdoc
CREATE  a3.rmdoc
```

If `aaa.rmdoc` contains two pages, then `a3.rmdoc` will be created with pages (A1, A2, A1, A2, A1, A2) in that order.

**Tip**: if you need to create a notebook with *many* copies of the same page, you can do it in stages. For example, to make a notebook with 100 copies of an input file "`x1.rmdoc`", you could do something like this:

```
rmcat -o x5.rmdoc x1.rmdoc x1.rmdoc x1.rmdoc x1.rmdoc x1.rmdoc
rmcat -o x25.rmdoc x5.rmdoc x5.rmdoc x5.rmdoc x5.rmdoc x5.rmdoc
rmcat -o x100.rmdoc x25.rmdoc x25.rmdoc x25.rmdoc x25.rmdoc
```

### Convert file formats

The script can also convert from one file format to another, by "combining" a single input file and writing the output in the *other* format.

```
$ rmcat -o aaa.rmdoc aaa.rmn
READ    aaa.rmn
CREATE  aaa.rmdoc
```

Note that [rmdoc2rmn and rmn2rmdoc](../rmdoc-convert/README.md) are shell scripts which do this. They are simpler because they don't need to do anything with the JSON files *within* the notebooks, they just uncompresss it from one format and compress it in the other format. If all you need to do is convert files from one format to another, and you don't want to deal with installing Perl modules, they may be a better choice.

# TODO

* Figure out how to handle `.rmt` files.
    * multiple input docs with identically named `.rmt` files - safe to assume they will contain the same template?
    * how does RCU react when uploading an `.rmn` contianing an `.rmt` which doesn't already exist in the tablet?

* Figure out if it's possible to combine a single PDF with other notebooks.
    * how are PDF-backed pages tracked in `UUID.content`? how is this different from normal pages?
    * how to handle things when the PDF is not the first input file, i.e. adjust page numbers?

# Changelog

### 2024-09-28 v0.1.1

- Fixes for https://github.com/kg4zow/rm2-scripts/issues/6
- Explicitly remove the work directory at the end, rather than relying on `END{}` (which causes an error message if you run the script without actually building an output file, like running it with `-h`).
- Run the 'mv' command to save an `.rmdoc` output file, rather than using `rename()`. This way if the work directory is on a different filesystem than where the output file needs to be written, it actually *works*.

### 2024-07-12 (no version)

- Initial public release.

# License

The MIT License (MIT)

Copyright &copy; 2024 John Simpson

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
