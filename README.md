# rm2-scripts

This repo is where I'm storing the scripts that I write for working with the [reMarkable tablet](https://remarkable.com/).

## Scripts

Each script will be in its own directory, and each directory will have its own `README.md` file which talks about that script. This is just a *quick and dirty* list of the scripts, and is not guaranteed to be up to date. If you see a directory above for a script which isn't mentioned here, or if this list says something different than what's in the script's directory, you should trust what's in that directory instead of what you see on the list below.

### Maintenance Scripts

These scripts require that the tablet be connected to your computer via USB cable. I *think* they should also work if the tablet is connected to wi-fi and on the same network with your computer, but I haven't tried this (my own tablets have never been connected to wi-fi).

* `rm2-list` - shows a list of the "files" in your tablet.

* `rm2-backup` - uses `rsync` to make a backup of the tablet's filesystem, in a way that successive backups only store files which are *new or updated* since the most recent backup.

* `rm2-clean` - lists files which are in a "delete-able" state, and can actually *delete* those files.

### Template Scripts

These scripts create "templates" which can be uploaded to a reMarkable tablet, and used as "backgrounds" for pages in notebooks. These templates are 1404x1872 `.png` files.

> Templates can also be stored as `.svg` files, and all of reMarkable's built-in templates have both `.png` and `.svg` versions. I'm not sure yet, but I *think* the `.svg` versions are somehow used to implement the [continuous pages](https://support.remarkable.com/s/article/How-to-use-continuous-pages) feature in the 3.0 firmware.

* [`rm2-template-basic`](rm2-template-basic/) is a shell script which uses [ImageMagick](https://imagemagick.org/) to create a few simple templates. I thought it was cool because it provides a way to build images from scratch from a script.

* [`rm2-template-cover`](rm2-template-cover/) is another shell script using[ImageMagick](https://imagemagick.org/), to generate a *very* simple "cover page" template. If you use this template for the first page of a notebook, and set the notebook to use the first page as its thumbnail, this makes a nice (but simple) cover page.

* [`rm2-template-calendar`](rm2-template-calendar/) is a Perl script which uses the [Perl GD module](https://metacpan.org/pod/GD), which uses the [GD library](https://libgd.github.io/), to generate a monthly calendar, in portrait or landscape mode, and with right-handed (with room for the menu bar on the left) or left-handed (with room for the menu bar on the right) options.

    By default it generates a blank calendar which can be re-used for any month (if you don't mind writing the dates in), but it also has an option to include the dates for any `YYYY-MM` (if you don't mind the template only being usable for that one month).

    I started this as an exercise to teach myself how to use the Perl GD module, but it ended up being useful so I kept it.

* [`rm2-template-bowling`](rm2-template-bowling/) is another Perl-GD script I wrote. Somebody had asked a [question on Reddit](https://www.reddit.com/r/RemarkableTablet/comments/15l3a9p/screen_protector_and_templates/) about sports templates, and I was bored and figured "how hard could it be?", so I wrote this.

### Other Scripts

* [`rm2-make-pdf`](rm2-make-pdf/) is a shell script which uses [Calibre](https://calibre-ebook.org/)'s `ebook-convert` command to convert an ebook (usually EPUB, but it'll work with any ebook format that Calibre can read) to a PDF with the settings (page size, font, font size, margins, etc.) that I think look good on a the reMarkable tablet. I do this because I don't think the PDFs that the reMarkable tablet produces when you upload an EPUB file, look all that great.

## Other sites

Sites that I maintain (this is *not* a complete list)

* [`https://remarkable.jms1.info/`](https://remarkable.jms1.info/) is where I started writing documentation about my adventures with the reMarkable tablets, and was the original "home page" for these scripts. I will be migrating the scripts from that site into this repo and updating the site accordingly.

* [`https://jms1.pub/`](https://jms1.pub/) is a web version of my [Keybase](https://keybase.io/) public directory. Keybase used to have a service on the `keybase.pub` domain where *every* Keybase public directory was visible, but they took that down after introducing [Keybase Sites](https://book.keybase.io/sites). At first I thought they took down the `keybase.pub` site because it was costing them too much, but then it occurred to me that maybe some people didn't *want* their public directories visible on the web like that, so they set up Keybase Sites as a way to give people more control over which files are and are not visible.

* [`https://jms1.net/`](https://jms1.net) has been my "home page" since 1999. The site has a random assortment of things that I felt like writing about over the years. No guarantees that any of it will be relevant or interesting, I keep meaning to go back and clean it up.

* [`https://qmail.jms1.net/`](https://qmail.jms1.net/) is where I used to write information about [qmail](https://cr.yp.to/qmail.html), an open-source (now public domain) MTA (mail transfer agent, or "mail server program"). I still use it, but I haven't had time to keep up with it since I moved from doing private consulting back into the world of full-time employment.

## TODO

Ideas which may or may not be added to the script in the future.

* `da_type = daily`
    * override some other settings?
        * `da_mcal_pos = sw`
        * `da_h1_format = "%Y-%m-%d"`
    * add boxes in top row for start/stop/total times

* Cover/title page

* Figure out issue with first page not showing up in TOC - [link](https://github.com/kg4zow/rm2-scripts/issues/1)

## License

Each script is covered by its own license, as explained in that script's `README.md` file. They all have licenses which make them free to use, both as in "free beer" (i.e. zero pricetag) and "free speech" (i.e. you get the source, and you're free to share them with others, usually with the only restriction being that my original copyright notice has to be shared as well).
