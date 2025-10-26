## The Terminal and You
The terminal is one of the most fundamental tools to use any computer ever made at a low level. It gives you the most direct access possible to your files; in fact, it *only* gives you access to your files. The terminal is based on having a **working directory** - i.e., when you are in a terminal environment, it is always running *inside some directory.* The core of learning how to use a terminal is familiarizing yourself with the commands used to **change between directories**, **list** the contents of directories, and **make and delete** files and directories.

The most common terminal used today is `bash`, which stands for "Bourne Again SHell"; the name isn't important - what matters is that it is relatively easy to access a terminal from any computer, and that said terminal is most likely going to be `bash` (unless you *really* know what you're doing). This means that once we get a terminal running on *any* system, the commands will largely be the same. On Linux and MacOS, the terminal is integrated directly; on Windows, the default terminal is Powershell, so you'll need to install WSL to run Linux inside your Windows machine to access a `bash` terminal instead. 

## Intro and Termin(al)ology
When working in a terminal, there are a couple terms and concepts you'll need to be familiar with to get the full functionality out of your environment. The first is the concept of a **working directory** - the current directory that your terminal is in. When you open a terminal for the first time, unless explicitly configured otherwise, it will open in your **home directory**; this most commonly will be `/home/username`, where `username` is the name of your user on that system. A directory is literally just a file folder; Linux and Unix treat most resources in the computer as a files (in various complicated ways), so effectively, all a terminal is is a glorified file explorer with lots of commands built in that you can run inside the various directories you're working in. Though annoying, we'll continue to use the term "directory" and not "folder" for this documentation; there are a lot of commands named after abbreviations for the word "directory," so we'll keep using "directory" so you can start getting familiar with the shorthand.

An alternate name for the terminal is, fittingly, the "command-line" (or CLI, for "Command-Line Interface"), as you'll be executing **commands** to move around between directories and interact with the files that you're working on. Everything in a terminal environment uses **relative paths** to wherever your working directory is; in practice, this means that any time you run commands or try to work with files, you'll have to access them by providing their location *relative* to wherever your current directory is. Understanding filepath notation and the tricks to write it out for your commands is 90% of the battle when learning to use a terminal. Though you don't need a full understanding of how the Linux/Unix filesystem is structured, you *will* need to know how to navigate and reference the paths to the files you're working on.

## Basic Commands
When using a terminal, you will get used to using a couple commands *very* quickly for navigation and general usage of the environment. The first, and most fundamental, is `pwd`, which stands for **P**resent **W**orking **D**irectory. This will print the full path of your working directory; in other words, *`pwd` will tell you where you currently are.*
```bash
> pwd
/home/cayenne
>_
```

Once you know *where* you are, the next question is "well, what's in here with me?" The answer to this question can be found with `ls`, which is literally just short for **l**i**s**t. This does exactly what it says on the tin - running the command with no arguments will list the contents of the current directory, and running it with an additional path added to it will list the contents of *that* directory.
```bash
> pwd
/home/cayenne/Documents/rocketry/avionics/active_drag_system
> ls
build  CMakeLists.txt  hardware  include  LICENSE  notes.txt  README.md  src  tools
> ls src
active_drag_system.cpp  adxl375.cpp  CMakeLists.txt  heartbeat.cpp  iim42653.cpp  kalman_filter.cpp  log_format.cpp  mmc5983ma.cpp  ms5607.cpp  pa1616d.cpp  pwm.cpp  serial.cpp
>_
```
Note that in the first `ls` call, some of the listed files have `.[filetype]` file extensions, while some of them don't; the ones with file extensions are actual files, whereas the ones without them are typically directories. Most of the time, the terminal will represent these in different colors for convenience.

Now that you know how to find where you are and know what's in various directories, the next order of business is moving around. This is done with `cd`, short for **c**hange **d**irectory. If done with no argument, this will move you back to the *home* directory, and if done with a path argument, it will move to that directory. 
```bash
> pwd
/home/cayenne/Documents/rocketry/avionics/active_drag_system
> cd
> pwd
/home/cayenne
> cd Documents
> pwd
/home/cayenne/Documents
>_
```

With that out of the way, let's make some files and directories. There are a couple of ways to do this, and it varies if you're working with directories or files - we'll list them out preemptively. For files, `touch` followed by the name of the file will create a file in the current directory if it doesn't already exist, or at the configured directory if the name of the file is passed with a relative path as well (see below). The equivalent of this for directories is `mkdir`, short for **m**a**k**e **dir**ectory. 
```bash
> ls
testfile.txt
> touch testfile2.txt
> ls
testfile.txt testfile2.txt
> mkdir exampledir
> ls
exampledir testfile.txt testfile2.txt
> touch exampledir/examplefile.txt
> ls exampledir
examplefile.txt
>_
```

Now to *move* files and directories, we simply use the `mv` command - take a guess what that's short for. This works for directories regardless of whether they have any files currently in them; the whole directory will be moved and preserve its internal structure. Simply add an argument for the file/directory that you'd like to move, followed by the path to where you want to move it to.
```bash
> mkdir movedir
> ls 
exampledir movedir testfile.txt testfile2.txt
> mv testfile.txt movedir
> mv exampledir movedir
> ls
movedir testfile2.txt
> ls movedir
exampledir testfile.txt
> ls movedir/exampledir
examplefile.txt
>_
```

To *remove* files on the other hand, we use the `rm` command, which again is pretty easy to intuit the shorthand. This will simply delete files, no questions asked; to remove directories, the `-r` flag needs to be passed, which is short for `--recursive` to recursively `rm` all of the directory's subdirectories and the files within them. **Be careful when doing this.** The terminal does not discriminate, and if you accidentally provide the wrong path, it *will* just delete it without confirmation.
```bash
> ls 
movedir testfile.txt
> rm testfile.txt
> ls 
movedir
> rm -r movedir
> ls 
# [nothing is printed here, as this directory is now empty]
>_
```

Now as for modifying files, this is up to personal preference, but we'll add a couple useful commands here. The `echo` command can be used to print anything to the terminal, including variables; it can *also* be used to `echo` strings into files, by using the `>` operator. To read the contents of a file out to the terminal, use the `cat` command, which is short for con**cat**enate; `cat` can be used for a couple other string handling things, but we'll just use it as a file readout for now.
```bash
> touch testfile.txt
> cat testfile.txt
# [the (touch) command creates empty files, so this is doesn't print anything]
> echo "Hello, World!" > testfile.txt
> cat testfile.txt
Hello, World!
>_
```

There *are* terminal based text-editors that let one edit files over the command-line and write in them natively. Documenting them is outside the scope of this doc; but for reference, the "default" one installed on most systems is `nano`, a very barebones editor which includes popups in its window for the various hotkeys (such as saving and exiting, copy/paste, etc). To edit a file with `nano`, simply use `nano` followed by the path to the file you want to edit; the rest of the hotkeys will be listed at the bottom of the screen. A more advanced alternative is `vim`, but only open a file with `vim` if you've read about how to use it first, as the hotkeys are not listed in the interface (and are *not* initially intuitive). Note that whichever terminal editor you use (if you choose to use one), if you open a file that does not yet exist and write to it, it will create that file on the drive.
```bash
> ls
testfile.txt
> nano nanofile.txt
# [inside the nano editor, "I wrote this in nano!" is written and saved]
> ls
nanofile.txt testfile.txt
> cat nanofile.txt
I wrote this in nano!
>_
```

Lastly, if you have files/directories that you'd like to copy, the requisite command is `cp`; all you need is to add an argument for the file/directory, and then a second one for the path of where you want to copy it to. The copy path argument is explicit, i.e. if you want to change the name of a copy of a file just change its name by adding the changed name to the copied path. For directories, the same flag as `rm` is necessary - add a `-r` to tell the `cp` command to recursively copy the contents of a directory when you copy it. 
```bash
> ls
nanofile.txt testfile.txt
> mkdir copydir
> cp testfile.txt copydir/copytestfile.txt
> ls
copydir nanofile.txt copytestfile.txt 
> cat copydir/copytestfile.txt
Hello, World!
> cp -r copydir copyofcopydir
> ls 
copydir copyofcopydir testfile.txt
> cat copyofcopydir/copytestfile.txt
Hello, World!
>_ 
```

And finally, one last quick note - there are a *lot* of commands for navigating the terminal, and you'll get more used to them over time. Luckily, there is documentation for *every single command* just directly built into the terminal - type `man`, short for **man**ual, followed by the command that you want to reference the usage of. To exit these `man` windows, hit `q`, or use `ctrl + c` to force quit; the latter applies to any process running in a terminal if you want to force quit it prematurely.

## Basic Command Quick Reference
- **Directory Movement/Content**
	- `pwd` - print the **p**resent **w**orking **d**irectory
	- `ls` - **l**i**s**t the contents of a directory
	- `cd` - **c**hange **d**irectories to a provided target
- **File Creation/Destruction**
	- `touch` - create an empty file if it does not exist already
	- `mkdir` - **m**a**k**e **dir**ectory with configured name
	- `cp` - **c**o**p**y a given file to a second location via path
		- `cp -r` - do the above but for directories, **r**ecursively
	- `rm` - **r**e**m**ove a given file (delete it)
		- `rm -r` - do the above but for directories, deleting all contents
- **File Modification and Content**
	- `cat` - con**cat**enate one or multiple files, and *print their contents*
	- `echo "some string" > file.extension` - append `some string` to the end of `file.extension`
	- `nano` - in-terminal file editor with keybinds listed
- **File Path References**
	- `.` - current directory, i.e. equivalent to `pwd`
	- `..` - *parent* directory of current directory
	- `~` - home directory, usually `/home/username`

## Filepaths, Filepaths Galore
Referencing Linux/Unix filepaths seems very scary at first, but is quite nice once gotten used to. The most important part of this is knowing the necessary shorthand, of which there is not much. There are three symbols which will be used in conjunction with these paths frequently:

- `.` - A single dot denotes the *current* directory, i.e. `.` is equivalent to running `pwd`
- `..` - Two dots denotes the *parent* directory of your current one, i.e. if `pwd` returns `/parent/child` then `..` would return `parent`
- `~` - The tilde denotes the *home* directory, i.e. `~` would by default return `/home/username`

Most of the difficulty of learning to work with terminals is learning the shorthand and tricks for moving between directories and referencing them with the above. Notably, you can *chain* these operators together like you would parent and child directories with a `/`. For example, say you stored all of your cloned Github repositories inside the same parent directory. If you were in the CMake `build/` folder of a separate repository, and wanted to `cd` into the `build/` folder of the `active_drag_system` to compare something, then you could simply run the following:
```bash
cd ../../active_drag_system/build
```
The double `../..` references the parent *of* the parent, the shared directory where both of your repos are; then, the `active_drag_system/build` folder can just be directly accessed from there. You can provide either the *direct* or the *relative* path to any given directory or file, as long as that actually is where it is - and this can be done with any combination of the shorthand above. If you're currently in `/home/username/` for example, and you want to reference `Documents/file.txt` for something, the following would all be equivalent:

- `Documents/file.txt`
- `./Documents/file.txt`
- `home/username/Documents/file.txt`
- `~/Documents/file.txt`
- `../username/Documents/file.txt`
- `Documents/../Documents/../Documents/../Documents/file.txt`

Generally, though the last one would work just fine, try to be as concise with your paths as possible.

## Packages and Where to Find Them
When you're running a terminal, you're a bit *closer* than you would otherwise be to the underlying software that runs your operating system. This is good and bad - it means that you have access to the actual libraries that the OS will use to do things like compile and run programs, but it also means that you'll have to *manually* go in and install most of them, as they don't come by default in most cases. Luckily, there is usually something called a **package manager** installed by default in most terminals. Which package manager you have depends entirely on what OS you're running - Debian Linux (including Ubuntu) uses a common package manager called aptitude (or `apt` in a terminal), MacOS commonly uses `brew`, Arch Linux uses `pacman`, and so on and so forth; the syntax for using each one is entirely based on which one you have, so it's outside the scope of this document to try and reference each one. For this case, we'll use `apt` as an example, as it's the most common; installing packages with `apt` is simple, just needing a command to `update` the list of packages, and then a second one to `install` the packages by name once they're found.
```bash
sudo apt update
sudo apt install package anotherpackage thirdpackage 
```
