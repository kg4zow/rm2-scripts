# The `rm2-cal` script

This is a Perl script which generates PDF files containing yearly or monthly calendars, or daily pages with a mostly open space in the middle (or lines, or dots) that the user can write on using a reMarkable tablet.

This came about because I've been reading the [r/remarkable](https://reddit.com/r/remarkable) subreddit, and I saw a lot of people asking about "day planners". I had already created a simple monthly calendar PDF using the [`PDF::API2`](https://metacpan.org/pod/PDF::API2) Perl module, and it occurred to me that expanding on that to create a "day planner" PDF might be *tedious*, but it shouldn't be *difficult*.

Turns out I was right - it's tedious, but not overly difficult.

## Sample Files

The PDF files in this directory were generated using the date specifier matching their filename (i.e. `2023-10.pdf` was generated using "`2023-10`" as the date specifier).

# Using the script

## Pre-requisites

* Perl

* Perl modules

    * `Getopt::Std` - included with Perl itself
    * `PDF::API2` - install using CPAN
    * `POSIX` - install using CPAN
    * `Time::Local` - included with Perl itself

&#x1F6D1; **TODO** link to page(s) explaining how to install modules using CPAN, also see if these modules are available as OS packages (centos, debian, etc.)

## Configuration

Some aspects of the calendars, as well as the list of calendar pages, are configurable. This can be done using command line arguments, a config file, or both.

### Command line arguments

Command line arguments can be any of the following:

* **An output filename**. These end with "`.pdf`".

    * A config file may specify an output filename, but cannot specify more than one.
    * The command line may also specify an output filename, but not more than one.
    * If a config file and the command line *both* specify an output filename, the one in the command line will be used. This allows you to set a "default" filename in a config file, and override that filename when you want to make a one-off file.

* **Date specifiers**, which tell the script which years, months, and days you want pages for in the file. These all start with digits, and currently require four-digit years. These are explained below.

* **Configuration options**, which affect how the program works. These are used for things like the font used for all text, the font *sizes* used on the various pages, how dates are formatted, and other page-specific options. These are explained below.

If you're happy with the default formatting, or you only need to change a few options, you can specify all options on the command line. For example, to generate a PDF with pages for all of 2023-10, whose "day" pages are filled with dots every &#xBC; inch and have a small calendar in the upper right (or "northeast") corner of the page, you could run this:

```
$ ./rm2-cal da_type=dots da_mcal_pos=ne 2023-10-xx output.pdf
```

The order of the command line options, date specifiers, and output filename is not important, as long as:

* there is *at least* one date specifier; and

* there is *exactly one* output filename.

### Configuration files

The script has a lot of possible options to customize the generated PDFs. You *could* specify them all on the command line, but it's easier to create a config file with the options you like for your own PDFs, and store your preferences there.

The script will look for a config file in the following locations. The *first* file it finds is the one it will use.

* File specified with the `-c` option
* `$XDG_CONFIG_HOME/rm2-cal-rm2-cal.cfg`
* `$HOME/.config/rm2-cal/rm2-cal.cfg`

Config options specified on the command line will override the selected config file.

The script has an option to *print* a sample config file. You can save this to a file and edit the file to make your own configuration with the specific settings you like. Having a config file can save you some typing, and make it easier to create the same *kind* of calendar in the future.

```
$ mkdir -p ~/.config/rm2-cal/rm2-cal.cfg
$ ./rm2-cal -g > ~/.config/rm2-cal/rm2-cal.cfg
```

The file it generates is all human-readable text. Each *line* in the file is treated as either a config option or a date specifier (explained below). Comments (starting with "`#`") and blank lines are ignored. Extra whitespace at the beginning or end of each line are also ignored.

To *use* a config file, you can specify it on the command line with the `-c` option, like so:

```
$ ./rm2-cal -c custom.cfg 2023-09,2024-12 output.pdf
```

You can also store the config file in `$XDG_CONFIG_HOME/rm2-cal-rm2-cal.cfg` or `$HOME/.config/rm2-cal/rm2-cal.cfg`, and if the script is run *without* a `-c` option, it will use the config file automatically.

## Date specifiers

The script knows how to create three different kinds of pages:

* A "year" page covers an entire year, and contains twelve monthly calendars.

* A "month" page covers a single month, with a traditional "calendar" view taking up the bulk of the page.

* A "day" page covers a single day. This *can* have a small calendar for the current month in one of the four corners.

A "date specifier" tells the script which year, month, day, or *range* of years, months, or days, you want to generate pages for. They all start with a four-digit year, and can have any of the following formats:

* `YYYY` adds a single "year" page, covering that year.

* `YYYY,YYYY` adds "year" pages for every year in the range.

* `YYYY-MM` adds a single "month" page, covering that month.

* `YYYY-MM,YYYY-MM` adds "month" pages for every month in the range.

* `YYYY-MM-DD` adds a single "day" page, covering that date.

* `YYYY-MM-DD,YYYY-MM-DD` adds "day" pages for every date in the range.

* `YYYY-xx` adds the following:

    * A "month" page
    * A "day" page for every day in that month

* `YYYY-MM-xx` adds the following:

    * A "month" page for that month
    * A "day" page for every day in that month

* `YYYY-xx-xx` adds the following:

    * A "year" page
    * A "month" page for January
    * A "day" page for every day in January
    * A "month" page for February
    * A "day" page for every day in February
    * ... and so on through the end of December

All ranges are *inclusive*, i.e. "`2023,2025`" generates year pages for 2023, 2024, and 2025.

The script keeps track of which pages are included in the file, and will throw an error if you try to add the same page more than once. (This is a technical requirement for cross-page linking - each year, month, or day can only be on one page, because if there were two pages for the same date, a link to that date wouldn't know *which* page to jump to.)

## Configuration options

Config options look like `key=value` declarations. Spaces around the `=` are optional, however if you're entering them on the command line and want to include spaces, you'll need to quote the entire `"key = value"` string.

Values within a config option (i.e. the part after the `=`) may be quoted, and *must* be quoted if they contain any `#` charcters.

### Global

* **`font_name`** (string, default `Helvetica`) - can be the name of one of the standard PDF fonts, or the filename of a font file (tested with TTF and OTF font files, may work with others.

    The standard font names are:

    * `Courier`
    * `Courier-Bold`
    * `Courier-BoldOblique`
    * `Courier-Oblique`
    * `Helvetica` &#x2190; default
    * `Helvetica-Bold`
    * `Helvetica-BoldOblique`
    * `Helvetica-Oblique`
    * `Symbol`
    * `Times-Bold`
    * `Times-BoldItalic`
    * `Times-Italic`
    * `Times-Roman`
    * `ZapfDingbats`

    Examples:

    ```
    font_name = "Helvetica"
    ```

    ```
    font_name = "/Users/jms1/Library/Fonts/LiberationSans-Regular.ttf"
    ```

    Note that the *default* font sizes (below) were chosen to look good when using the `Helvetica` standard font. If you use a different font, you may need to adjust some of the font sizes as well.

* **`menu_width_px`** (pixels, default 104) - the width of the area on the left which is covered up by the reMarkable menu bar. This will be different depending on which reMarkable firmware you're using.

    * 3.0.5.56 &#x2192; 120
    * 3.5.2.1807 &#x2192; 104

* **`close_width_px`** (pixels, default 104) - the width of the "box" on the top right around the reMarkable "&#x2A02;" (close) button.

* **`margin_x_px`** and **`margin_y_px`** (pixels, default 20) - the width of a small "margin" around the edges of the calendars on the year and month pages, as well as the lines on a daily page with lines.

* **`header_ht_px`** (pixels, default 104) - the height of the "header bar" across the top of the page, where the page titles and navigation buttons are drawn.

* **`links`** (boolean, default `1`) - whether or not to generate cross-page links from calendars, and the navigation buttons. The value can be:

    * `yes`, `true`, or `1` to include links
    * `no`, `false`, or `0` to NOT include links

* **`nav_link_size`** (points, default 8) - the font size of the text below the arrows in the navigation buttons.

### Year pages

* **`yr_title_size`** (points, default 20) - the font size of the title at the top of the page.

* **`yr_mo_size`** (points, default 12) - the font size of the titles in the smaller monthly calendars.

* **`yr_da_size`** (points, default 12) - the font size of the date numbers in the smaller monthly calendars.

### Month pages

* **`mo_title_format`** (string, default `%B %Y`) - how the name of the month is formatted in the title. Uses `strftime()` to do the formatting, a web search for "`strftime`" will lead to dozens of web pages explaining the format. The default format string uses:

    * `%B` = the full name of the month, i.e. "October".
    * `%Y` = the full four-digit year, i.e. "2023".

* **`mo_title_size`** (points, default 20) - the font size of the title at the top of the page.

* **`mo_dw_size`** (points, default 10) - the font size of the "day of the week" headers at the top of the calendar, i.e. "Sunday", "Monday", etc.

* **`mo_da_size`** (points, default 12) - the font size of the date numbers in the main calendar.

* **`mo_sm_size`** (points, default 10) - the font size of the month names in the smaller "previous month" and "next month" calendars at the bottom of the page.

* **`mo_sd_size`** (points, default 10) - the font size of the date numbers in the smaller "previous month" and "next month" calendars at the bottom of the page.

### Day Pages

The titles of the day pages have two lines, with the "second" line *above* the first one. My original idea was for the "first" line (the date) to be above the second line (the day of the week), but I didn't really like how it looked. I ended up switching them around because I think it looks better, and so that the "main title" would be in the same place it is on the year and month pages.

* **`da_h1_format`** (string, default `%B %e, %Y`) - how the date is formatted in the lower line of the title. Uses `strftime()` to do the formatting, the default format string uses:

    * `%B` = the full name of the month, i.e. "October".
    * `%e` = the date, with a space in front of single-digit numbers (i.e. `" 4"` or `"14`").
    * `%Y` = the full four-digit year, i.e. "2023".

* **`da_h1_size`** (points, default 20) - the font size of the lower line of the title.

* **`da_h2_format`** (string, default `%A`) - how the date is formatted in the upper line of the title.

    * `%A` = the full day of the week, i.e. "Sunday", "Monday", etc.

* **`da_h2_size`** (points, default 10) - the font size of the upper line of the title.

* **`da_type`** (string, default empty) - how to "fill in" the main portion of the page. This can be one of the following:

    * `blank` (or empty string) - leave the area blank
    * `lines` - horizontal lines
    * `dots` - regularly spaced dots
    * `daily` - the daily log sheets I use for work. (I work in software development.)

* **`da_pitch`** (points, default 18) - if `da_type` specifies lines or dots, how far apart those lines or dots should be from each other. The default, 18, results in the lines or dots being &#xBC; inch apart. Larger values will make more space between the lines or dots.

* **`da_ch_size`** (points, default 10) - if `da_type` specifies the "daily" type, this is the text size of the column headers.

* **`da_mcal_pos`** (string, default empty) - if non-empty, a small calendar of the current month will be added to one of the four corners of the page. The value can be:

    * empty string - no small calendar
    * `ne` - northeast (upper right) corner
    * `nw` - northwest (upper left) corner
    * `se` - southeast (lower right) corner
    * `sw` - southwest (lower left) corner

    &#x26A0;&#xFE0F; If `da_type` is "daily", don't use `nw` here. If you do, it will mess up the column headers.

* **`da_mcal_h_pt`** (points, default 72) - the height of the small calendar.

* **`da_mcal_w_pt`** (points, default 108) - the width of the small calendar.

* **`da_mcal_tsize`** (points, default 10) - the font size of the month title in the small calendar.

* **`da_mcal_dsize`** (points, default 10) - the font size of the date numbers in the small calendar.

## Examples

Quick examples:

* `./rm2-pdf 2023 output.pdf`

    Generates a one-page PDF with a yearly calendar for 2023.

* `./rm2-pdf 2023-xx output.pdf`

    Generates a PDF with the following pages, in this order:

    * yearly, 2023
    * monthly, 2023-01 through 2023-12

* `./rm2-pdf 2023-xx-xx output.pdf`

    Generates a PDF with the following pages, in this order:

    * yearly, 2023
    * monthly, 2023-01
    * daily, 2023-01-01 through 2023-01-31
    * monthly, 2023-02
    * daily, 2023-02-01 through 2023-02-28
    * ... more monthly/daily sets through 2023-12-31

Date *ranges* can be specified by separating the first and last year/month/date with a comma, i.e. `2023,2027` would create five "year" pages. Ranges must use the same "kind" of date, i.e. you can't specify "YYYY,YYYY-MM".

You can also specify multiple dates or ranges on the command line, for example:

* `./rm2-pdf 2023-09 2023-10,2023-12 output.pdf`

    Generates a PDF with those four "month" pages.

## License

This script is licensed under the MIT License.

**The MIT License (MIT)**

Copyright &copy; 2023 John Simpson

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.


## Future

I have the following ideas for future versions of this script.

* Landscape view

* Left-handed files

* Re-write it in Golang, to make it easier for others to run (i.e. so people won't have to mess around with installing CPAN modules)

* For `da_type = daily`
    * override some other settings?
        * `da_mcal_pos = sw`
        * `da_h1_format = "%Y-%m-%d"`
    * add boxes in top row for start/stop/total times

* Cover/title page

* Figure out issue with first page not showing up in TOC - [link](https://github.com/kg4zow/rm2-scripts/issues/1)

* Option to specify only certain days of the week (i.e. Mon-Fri only), obviously default to "every day"

    * mini-calendars: if target date doesn't exist in the file, use "lighter" text?
