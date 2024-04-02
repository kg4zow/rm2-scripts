# Conversion scripts for rmdoc files

These scripts convert reMarkable archive files between reMarkable's `.rmdoc` format, and RCU's `.rmn` format.

I normally use [RCU](http://www.davisr.me/projects/rcu/) to manage the documents on my tablets. In particular, I don't use reMarkable's apps or web site, so I didn't know that `.rmdoc` files *existed* until somebody mentioned them on [Reddit](https://www.reddit.com/r/RemarkableTablet/comments/1bsf8i0/archive_files_from_web_interface/).

## Archive Files

For a long time, reMarkable didn't offer a way to create real "backups" of your notebooks. The only supported option was to use their cloud service. Any option they provided to create a "backup" of a notebook involved converting it to PDF format, which is great if you need to look at the notebooks on a computer, but if you later need to upload the PDF back into a tablet, any pen strokes that you may have added to the notebook would now be "burned into" the notebook, and you wouldn't be able to set templates for the pages in the notebook.

RCU offers a way to create actual backup files, as "`.rmn`" files. These files can later be uploaded into a tablet, in its original *edit-able* format. For a long time, this was the only option to make real "backups" of reMarkable notebooks.

With the 3.9 software, reMarkable introduced their own archive file format. Their "`.rmdoc`" files are essentially the same as RCU's "`.rmn`" files.

### Comparison

The only real differences between the two file formats are:

* `.rmn` files use `tar` as the container format, while `.rmdoc` files use `zip`. The files *within* the containers are the same. Unless you're looking at the contents of the archives, this doesn't really make any difference.

* You need to have a copy of RCU in order to download or upload `.rmn` files. The tablet's built-in web interface, and reMarkable's apps, know how to work with `.rmdoc` files.

The similarities are:

* Your computer can't *directly* do much with either type of file, other than *hold* them in case you need to upload them again.

* Both file types can be uploaded back to a reMarkable tablet (using the appropriate software), and the uploaded notebook be fully edit-able.

* Both file types can be sent to other people who own reMarkable tablets, and *they* can upload them into their tablets (again, using the appropriate software).

## Scripts

When I found out that `.rmdoc` files existed and what their file format was, I figured it should be fairly easy to write some little shell scripts to convert between the two formats. Turns out I was right.

* `rmdoc2rmn` uses `unzip` to expand the contents of an `.rmdoc` file into a temporary directory, then uses `tar` to create an `.rmn` file.

* `rmn2rmdoc` uses `tar` to expand the contents of an `.rmn` file into a temporary directory, then uses `zip` to create an `.rmdoc` file.

Enjoy.
