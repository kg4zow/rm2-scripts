# The `rm2-make-pdf` Script

This is a shell script which uses [Calibre](https://calibre-ebook.com/)'s `ebook-convert` utility to convert any file that Calibre knows how to read, into a PDF with the right settings to look good on a reMarkable tablet.

The [Calibre](../info/calibre.md) page explains what this script does, and includes instructions for how to use Calibre's GUI to convert the files "by hand".

## Details

Run the script with the name of the file you want to convert to PDF.

If the file you're converting *from* doesn't have metadata (HTML, Markdown, plain text, etc.) or if you want to change the title or author (the only two metadata fields used by reMarkable tablets, as far as I can tell) you can add command line options to set the values in the PDF file.

* `-t` sets the title. If the title contains any spaces, you will need to quote the value.

* `-a` sets the author. If the author contains any spaces, you will need to quote the value.

* `-s` sets the "base" font size used in the PDF. Changing this value will make the overall text in the PDF larger or smaller. If not specified, the script will default to 12.

    This value *relates to* the base font size specified in the input file (i.e. if the input file says that the base font size is 11 and you use "`-s 12`", all of the text in the file will be a bit bigger than what the input file called for.

    Because of this, I normally start by creating a PDF using the default size 12, and viewing the PDF on the computer. If the text needs to be bigger or smaller, I'll delete the PDF and run the script, specifying a larger or smaller size.

* `-p` For input document types which have a document structure whose "section headings" translate to `<H1>` elements, using the `-p` option will insert a page break just before this element. This ensures that "top level" sections, such as chapters in a book, always start on a new page.

    This is not normally an issue when converting EPUB files, however it does come up a lot when I convert Markdown files. (Most of the documentation I write is authored using Markdown.)

    Note that the `ebook-convert` command's *default* behaviour is to add the page breaks. This script normally adds options to *disable* the page breaks, UNLESS you specify the `-p` option.

### Example

```
$ rm2-make-pdf -a 'Cory Doctorow' -t 'Little Brother' little-brother.epub
```

This will create a file called "`little-brother.pdf`".

## License

This script is licensed under the MIT License.

**The MIT License (MIT)**

Copyright &copy; 2023 John Simpson

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
