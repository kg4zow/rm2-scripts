# rmbackup

Backs up the raw files from a reMarkable tablet.

Backups are gathered using `rsync` and stored in a designated "backup directory", which will have a `raw/` directory contianing the files from the tablet. Successive backups using the same directory will copy only the files which were updated since the previous backup.

The program can optionally produce "archive" files from the raw files. These archives allow you to easily restore notebooks to the tablet (or to a different tablet). Archive files are "`.rmdoc`" files which can be uploaded using the built-in web browser (in tablet software 3.10 and later), and "`.rmn`" files which can be uploaded using RCU.

If the tablet is connected via USB and the tablet's built-in web interface is enabled, the program can also download PDFs of the documents. When you do this, any pen strokes you may have added to the original file will become part of the PDF, and if you later upload the PDF back to a tablet, those pen strokes will not be edit-able.

Tablets linked to a *paid* reMarkable cloud account now have the ability to make documents "cloud archives", where the document's metadata is still on the tablet but the content is only stored in the cloud. The program will skip these documents when creating `.rmdoc` or `.rmn` files, or when downloading PDF files.

The program *can* also run "`git commit`" and "`git push`" commands before finishing. Obviously this requires that the backup directory be set up as a git repository, and optionally a git remote. This will be explained in more detail below.

**Note:** Now that I have this working in Perl, I am also planning to write an equivalent program in Golang which does the same things.

## Installing the Program

The program itself was written in Perl on macOS and tested using Linux.

It *might* also work on windows if you're able to set up the pre-requisites, but I don't have any windows machines to try it on. Nor do I have any interest in figuring it out. ([Here's a nickel, kid.](https://jms1.pub/heres-a-nickel-kid.jpg)  Get yourself a better computer.)

### Pre-requisites

The following items need to be installed on the system where you plan to run the program.

* Perl, with the following modules that may or may not be included as part of your system's Perl distribution.
    * `Cwd`
    * `File::Temp`
    * `Getopt::Long`
    * `JSON`
    * `LWP`

* `ssh`

* `rsync`

**CentOS 7**

```
yum -y install \
    openssh-clients \
    perl \
    perl-File-Temp \
    perl-Getopt-Long \
    perl-JSON \
    perl-libwww-perl \
    perl-PathTools \
    rsync
```

**Debian 12**

```
apt -y install \
    libjson-perl \
    libwww-perl \
    openssh-client \
    perl-base \
    rsync
```

**CPAN**

The [Comprehensive Perl Archive Network](https://cpan.org), or CPAN, is the definitive repository of Perl modules which are not distributed as part of Perl itself. Most Perl distributions include a `cpan` tool to install additional modules. If not, the CPAN web site should have directions on how to install it.

To use the tool ...

* First make sure the module isn't already installed. Using the "`JSON`" module as an example:

    ```
    perl -e1 -MJSON
    ```

    **Note:** do not add a space between "`-M`" and the name of the module.

    If this command *doesn't* print an error message, the module is already installed.

* If the module needs to be installed ...

    ```
    cpan install JSON
    ```

    Note that the first time you run `cpan`, you will be asked to configure it. Newer versions have a "wizard" that will configure most of the settings automatically. I've always had good results with this, so if you're asked about it, use it - especially if you're not already familiar with Perl.

### Install the Program

Copy the program somewhere in your `PATH`.

Make sure it has executable permissions.

```
cd $HOME/bin/
chmod +x rmbackup
```

## Starting a new Backup Directory

Each backup directory will contain the files from a specific tablet. If you have multiple tablets, you will want to create separate backup directories for each one.

### Create the Backup Directory

The directory must not already exist. The program will create it.

```
rmbackup --create /path/to/backup
```

### Configure the Program

When program created the directory, it also created an `rmbackup.cfg` file *in* the directory. This file tells the program about the tablet and which kinds of files to save.

Edit the `rmbackup.cfg` file. It will have comments explaining each option. These options are:

* `tablet` = the hostname or IP address of the tablet. The program creates the file with `10.11.99.1` here, which is the correct IP if you're connecting to the tablet via USB cable.

    Note that the program *can* connect to a tablet via wifi, however if you do this, the program won't be able to save PDF files. This is because the program uses the tablet's built-in web interface to download the PDF files, and reMarkable only allows web connections over the USB cable.

* `rmdoc` = whether or not to create `.rmdoc` archive files. These files are used by the reMarkable web interface in firmware version 3.10 and later. (Note that 3.9 *claimed* to support it, but it doesn't work.)

    The program builds these archives from the raw files downloaded from the tablet.

* `rmn` = whether or not to create `.rmn` archive files. These files are used by [RCU](http://www.davisr.me/projects/rcu/).

    The program builds these archives from the raw files downloaded from the tablet.

* `pdf` = whether or not to download `.pdf` files for each notebook.

    The program *downloads* these files using the tablet's built-in web interface. Normally this is only available when connecting to the tablet via USB, however there are ways to "hack" the tablet, such as ...

    * [webinterface-onboot](https://github.com/rM-self-serve/webinterface-onboot/) makes the tablet enable the web interface when `xochitl` (the "app" running on the tablet) starts. This is needed for the rM1, however it *might* not be needed on the rM2.

    * [webinterface-wifi](https://github.com/rM-self-serve/webinterface-wifi) makes the web interface available over wifi. Normally it can only be accessed via the USB cable.

        **This one can be dangerous.** The tablet's web interface is not encrypted and has no authentication, so if you do this, anybody on your wifi network will be able to browse, download, and upload documents on your tablet, and you won't know anything about it while it's happening.)

    * Other "hacks" are listed on the [remarkable.guide](https://remarkable.guide/tech/usb-web-interface.html) site (which is a good place to get started if you're interested in "hacking" on reMarkable tablets).

    I'm not a *huge* fan of hacks like this which modify the `xochitl` binary itself, but if you *really* want to be able to download PDF files over wifi, these seem to be the only choice.

* `pdf_wifi` = whether to download PDFs when connecting to the tablet using an IP address other than `10.11.99.1`. If this is not `true`, the script won't even *try* to download PDFs if connected to the tablet via wifi.

    This should only be set to `true` if your tablet has been "hacked" so the web interface is available over wifi, and you plan to download PDF files.

* `git_commit` = whether or not to run "`git commit`" after sync'ing files and creating or updating any archives.

    The program will check for a "`.git/`" directory first, and will throw an error if the directory does not exist. Only set this to `true` if the git repo is ready to go. (See below for details.)

* `git_push` = whether or not to run "`git push`" after running "`git commit`".

    This should only be set to `true` if `git_commit` is also `true`.

### Maybe: Set up a git repository

You *can* use the directory as a git repository. If you do this, you will be able to view the history of the files, and access past versions of the files.

In addition, you *can* also link the git repo to a remote server, such as Github or Keybase.

Note that you are not *required* to set up a git repo, or to link it to a remote server.

In the backup directory (i.e. the directory containing the `rmbackup.cfg` file) ...

* Initialize the git repo structure. This will create a "`.git`" directory.

    ```
    git init -b main
    ```

* Possibly create or update the `.gitignore` file. This tells the `git` command to ignore certain files when adding files or figuring out what changes were made.

    My primary OS is macOS, so all of my `.gitignore` files include the following:

    ```
    .DS_Store
    ._*
    ```

* Create the first commit, with the current contents of the directory (i.e. the `rmbackup.cfg` and `.gitignore` files).

    ```
    git add .
    git commit -m 'initial commit'
    ```

* Optionally, "tag" the first commit. (I've had a few cases where having a tag on the first commit in the repo turned out to be useful, so I've started doing it in every repo I create.)

    ```
    git tag -m 'initial commit' initial
    ```

### Maybe: Link to a remote Git server

If you've created a git repo, you may also want to synchronize it with a remote git service like Github or Keybase.

Note that the exact process will depend on the remote server, but the process will normally look something like this.

* Create an empty repo on the remote server. If the remote server offers to automatically create things like a `README`, `LICENSE`, or `.gitignore` file, be sure NOT to do this. You need to start with a totally empty repo.

    Get the URL of the new remote repo.

* In the backup directory, add a "remote" pointing to the new remote repo.

    ```
    git remote add origin git@github.com:USERNAME/REPONAME
    ```

* Push the commit (and tag, if you created one) to the remote.

    ```
    git push --all
    git push --tags
    ```

## Back up the Tablet

Now that the backup directory is set up, all that remains is to start taking backups. This is simple - run the program, and specify the location of the backup directory.

```
rmbackup /path/to/backup/dir
```

The *first* time you do this, it will ...

* Get the connected tablet's serial number and write it to a `serial.txt` file in the backup directory.
* Copy most of the files, including all of your notebooks, to the backup directory's `raw/` directory.
* If any `.rmdoc`, `.rmn`, or `.pdf` files are being created ...
    * Figure out the correct output filename for each notebook.
    * Maybe create `.rmdoc` files for each notebook, in the `rmdoc/` directory.
    * Maybe create `.rmn` files for each notebook, in the `rmn/` directory.
    * Maybe download `.pdf` files for each notebook, to the `pdf/` directory.
* Maybe run "`git commit`" to commit the changes.
* Maybe run "`git push`" to push the commit to a remote server.

After this, each additional time you run the same command, it will ...

* Get the connected tablet's serial number and *compare it* to the contents of the `serial.txt` file. If they don't match, the program will stop.
* Copy any files which have been *updated* since the last backup.
* Identify which documents changed since the previous backup.
* If any `.rmdoc`, `.rmn`, or `.pdf` files are being created ...
    * Figure out the correct output filename for each notebook.
    * Maybe create or update `.rmdoc` files for each updated notebook, in the `rmdoc/` directory.
    * Maybe create or update `.rmn` files for each updated notebook, in the `rmn/` directory.
    * Maybe download `.pdf` files for each updated notebook, to the `pdf/` directory.
* Maybe run "`git commit`" to commit the changes.
* Maybe run "`git push`" to push the commit to a remote server.


# License

The MIT License (MIT)

Copyright &copy; 2024 John Simpson

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

# Other Information

## SSH Authentication

If you haven't already done so, I strongly recommend you set up an SSH key pair and configure your tablet(s) to use it for authentication. If you don't, you'll have to enter the tablet's password several times while the program is running.

I wrote [this page](https://remarkable.jms1.info/info/ssh.html) a few months ago, to explain how to set this up.

# Changelog

### 2024-09-30 - v0.6

* Added support for reMarkable Paper Pro
    * the only real change was figuring out how to read its serial number

### 2024-09-16 - v0.0.5

* Highlight "skipping cloud archive" messages
* Better support for UTF-8 characters in display names

### 2024-09-13 - v0.0.4

* Fixed bugs related to new functionality I haven't finished writing yet, let alone tested it.
    * When I tested v0.0.3 this morning, I was using my laptop instead of my regular machine, and accidentally ran the 2024-07-20 version (which was earlier in my PATH) instead of running the version I had just modified. This is what happens when you try to write code with a single cup of coffee.

### 2024-09-13 - v0.0.3 - DO NOT USE

* Checking for 'cloud archive' documents, not creating or downloading local files when these are found.
    * Thanks to Julien Ma for [pointing this out](https://github.com/kg4zow/rm2-scripts/issues/5), I don't have a paid reMarkable cloud account so I never would have found this on my own.
* Other minor updates to the `README.md` file.

### 2024-07-20

* Added changelog at the bottom of `README.md`.
* Added information about "hacks" to make the web interface (1) always active, and (2) accessible via wifi.
* Didn't commit/push at the time, this sat on my laptop's disk for a while. (Bad John, no coffee.) &#x1F928;

### 2024-07-13 - v0.0.2

* Updated initial generated config file
* Added version number, updated `usage()` message

### 2024-06-09 - (no version number)

* Initial public release.
* Added MIT license statement.
