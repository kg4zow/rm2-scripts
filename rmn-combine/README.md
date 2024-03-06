# rmn-combine

This is a Perl script which combines the pages from one or more input `.rmn` files into a single output `.rmn` file.

I wrote this because I use [drawj2d](https://drawj2d.sourceforge.io/) to generate notebook pages containing the correct numbers, in the correct places, to match up with the [calendar template](https://remarkable.jms1.info/templates/calendar.html) I use. The drawj2d program can only create single-page `.rmn` files, so after I create a set of individual pages for the months in a year, I use `rmn-combine` to combine those pages into a single `.rmn` file with a page for each month.

This is just an example, I'm sure there are other reasons why somebody might need to combine notebooks ilke this.

### Background: `.rmn` files

An `.rmn` file is a gzip-compressed tarball (or "`.tar.gz`" file) containing the actual files from a reMarkable tablet's filesystem, which make up that document. Each document, and therefore each `.rmn` file, will contain several files. If you're curious, these files are explained on [this page](https://remarkable.jms1.info/info/filesystem.html) (or at least, explained as well as I've been able to figure out). If you're not interested in the technical details, just keep in mind that an `.rmn` file *contains* other files, much like a `.zip` file.

## Installing the script

### Pre-requisites

**Perl**. The script is written in Perl, so you'll need a working Perl installation on your computer, version 5.005 or later. Most operating systems either have one pre-installed, or have one available in a software repository of some kind.

**JSON module**. You will also need the Perl JSON module. Depending on your operating system, you may be able to install it using something like "`yum install perl-JSON`" or "`apt install libjson-perl`". Otherwise, you can use [CPAN](https://www.cpan.org/) to install it, using a command line "`cpan install JSON`".

> If you run "`perl -MJSON -e1`" and get no output, the module is installed. (That "e1" is a digit "one", not a letter "L".)

### Script

Copy the script to a directory in your `PATH`.

Make sure it has executable permissions (i.e. "`chmod +x rmn-combine`").

## Running the script

Run the script with a "`-o`" option specifying the OUTPUT file you want to create, followed by the names of the INPUT files you want to read from.

```
$ rmn-combine -o output.rmn input1.rmn input2.rmn input3.rmn
```

When you run the script, it will show you the pages from each input file, and the files which make up that page. Each file will have one or more pages, and each page will have one or more files.

The output file's "display name", used as the document's title in the tablet's file browser, will be taken from the output filename. If you like, you can use the "`-n`" option to specify a name. If the title contains any spaces, you will need to quote it, as shown here:

```
$ rmn-combine -o output.rmn -n '2024 Calendar' 2024-*.rmn
```

### Other options

`rmn-combine -h` will print a list of other available options.

## Future

At some point in the future I want to re-write this in Golang, so I can compile it and distribute binaries, and people won't have to mess with installing Perl or Perl libraries.

If you have any ideas for how to make this script better, please let me know.

## License

This script is licensed under the MIT License.

**The MIT License (MIT)**

Copyright &copy; 2024 John Simpson

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
