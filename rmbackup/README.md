# rmbackup

Backs up the raw files from a reMarkable tablet.

Backups are gathered using `rsync` and stored in a designated "backup directory", which will have a `raw/` directory contianing the files from the tablet. Successive backups using the same directory will copy only the files which were updated since the previous backup.

The program can optionally produce "archive" files from the raw files. These archives allow you to easily restore notebooks to the tablet (or to a different tablet). Archive files are "`.rmdoc`" files which can be uploaded using the built-in web browser (in tablet software 3.10 and later), and "`.rmn`" files which can be uploaded using RCU.

If the tablet is connected via USB and the tablet's built-in web interface is enabled, the program can also download PDFs of the documents. When you do this, any pen strokes you may have added to the original file will become part of the PDF, and if you later upload the PDF back to a tablet, those pen strokes will not be edit-able.

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

    The program *downloads* these files using the tablet's built-in web interface. Normally this is only available when connecting to the tablet via USB, however there are ways to "hack" the tablet so the web interface is available over wifi.

* `pdf_wifi` = whether or not to allow PDFs to be downloaded when connecting to an IP address other than `10.11.99.1`.

    This should only be set to `true` if your tablet has been "hacked" so the web interface is available over wifi, *and* you plan to download PDF files.

* `git_commit` = whether or not to run "`git commit`" after sync'ing files and creating or updating any archives.

    The program will check for a "`.git/`" directory first.

* `git_push` = whether or not to run "`git push`" after running "`git commit`".

    This should only be set to `true` if `git_commit` is also `true`.

### Maybe: Set up a git repository

You *can* use the directory as a git repository. If you do this, you will be able to view the history of the files, and access past versions of the files. Also, you *can* also link the repo to a remote server, such as Github or Keybase.

Note that you are not *required* to set up a git repo, or to link it to a remote server. The program will check the backup directory for a `.git` directory when it finishes. If it exists, it will run a `git commit` command to save the files which changed since the previous commit.

In the backup directory (i.e. the directory containing the `rmbackup.cfg` file) ...

* Initialize the git repo structure. This will create a "`.git`" directory.

    ```
    git init -b main
    ```

* Possibly update the `.gitignore` file. This tells the `git` command to ignore certain files when adding files or figuring out what changes were made.

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

If you've created a git repo, you may want to synchronize it with a remote git service like Github or Keybase.

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

