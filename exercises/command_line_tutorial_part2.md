# Command-line clinic, continued 

## Paths: absolute, relative, and the PATH

This morning, Bob introduced you to the concept of a path in unix, and explained how paths relate to the heirarchical filesystem.  Let's briefly revisit that topic by discussing absolute paths, relative paths, and the PATH.  

### absolute paths

Absolute paths start at the root of the filesystem.  In other words, absolute paths start with `/`.  These are absolute paths:

- `/Users/gdw/Desktop` 
- `/usr/local/bin`
- `/Applications`

**Absolute paths always refer to the same place, no matter your present working directory**

### relative paths

Relative paths don't start with an `/`.   These are relative paths:

- `./Desktop`
- `Desktop` 
- `../Desktop`  

Their meaning is dependent on where you are in the filesystem (i.e., what is your pwd).

**A relative path's meaning depends on your present working directory**

## The PATH

The (upper case) PATH has a special meaning in unix environments.  The PATH is a list of directories where the shell will look for commands.  If you didn't have the PATH, you'd always have to type the full path (absolute or relative) of a command to run it, which would be annoying.

PATH is what's called an [environmental variable](https://en.wikipedia.org/wiki/Environment_variable).  Here are two ways to find your PATH:

- Run the `env` command in the terminal to show all the environmental variables.  Do you see your PATH?
- Run `echo $PATH` in your terminal.  

Can you recognize what other environmental variables mean?

### The PATH and the `which` command

The which command will you tell you where in the filesystem a command is, or more specically whether a command exists in your PATH.  To see a description of this command, run:

```
man which
```

Not to be confused with [manwich](https://manwich.com/) ![manwich](./manwich.jpg)

The `man` (manual) command is a great way to get more information about built-in linux commands.  You can page through a manual page using the up and down arrows or the space bar.  Press q to exit.  

Most linux commands have detailed manual pages that list all the different options for running them. 
``` 
man ls
```

OK, back to `which`.  All of the commands you run, even basic ones like `ls` and `cd`, are just a file somewhere in your path.  To find out where they are, run:

```
which ls
```

The PATH is a convience, so that instead of always typing:

```
/bin/ls
```

You can just type `ls`.    Though it is worth noting that `/bin/ls` and `ls` both do the same thing (run the `ls` command located in the `/bin` directory).


Other software that you've installed that's not a part of the core operating system, for instance the blastn and bowtie2 aligners, can also be found by `which`:

```
which blastn
which bowtie2
```

Let's try something.  Let's change your PATH:

```
cd
export PATH=/Users/gdw
```

Now look at the value of the PATH variable:
```
echo $PATH
```

What do you think the impact of this change will be?  Recall that the PATH is how the operating system finds commands.

What happens when you run these commands?

```
ls
/bin/ls
../../bin/ls
../bin/ls
```

What's going on?  Why do these commands work or not work?

In real life, you wouldn't sabotage yourself by overwriting your PATH.  There will be an example below of how to change your PATH to add additional directories to it.  For now, close your terminal window and open a new one to reset your enviroment. 

## Installing software from the command line

Installing software in a linux command-line environment can be a roadblock to beginners.  Let's practice installing a bioinformatics tool from the command line.

We'll install jellyfish, a tool for counting [k-mers](https://en.wikipedia.org/wiki/K-mer).  The main webpage for Jellyfish can be found [here](https://github.com/gmarcais/Jellyfish).  If you read that page, you will find installation instructions that direct you to the [releases page](https://github.com/gmarcais/Jellyfish/releases).

Here you find there are two options:

1. You can download a so-called pre-compiled binary.  Pre-compiled binaries are ready to run programs, but they must match to your operating system and computer.  Since we are working on mac laptops, you'll want to download the jellyfish-macosx binary from github

2. You can download the source code for the software and compile it to create a binary program file yourself. Sometimes this is the only option, for example for [minimap2](https://github.com/lh3/minimap2/releases), a program we'll use later this week.  


Let's first try downloading the pre-compiled binary.  Download the file named jellyfish-macosx
``` 
cd
curl -OL https://github.com/gmarcais/Jellyfish/releases/download/v1.1.12/jellyfish-macosx
```

`curl` is a program that will download files from the internet.

Type `ls -lh` to confirm that you have downloaded a file to the pwd named jellyfish-macosx that has a size of 1.0 Mb.  The `-h` option to `ls` outputs file sizes in a **h**uman readable format and the `-l` option outputs a more detailed "**l**ong" listing.

Now that we have the file, let's try to run it.  That was the point of downloading it, after all.  Try typing:

```
jellyfish-macosx
```

You should have gotten an error like this:

```
-bash: jellyfish-macosx: command not found
```

What do you think is going on?

<br><br><br><br> <br><br><br><br> <br><br><br><br> <br><br><br><br> 

One issue is that the file jellyfish-macosx is not in your PATH.  The jellyfish-macosx file is in your home directory, which is not in your PATH.  Let's confirm this by using the which command again:

```
which jellyfish-macosx
```

If which doesn't produce any output, it means that it coudn't fine a file with that name in your PATH.

To fix this situation, we could either move the jellyfish file into a directory in your PATH, or could add your home directory to your PATH.  The first option would manifest as:

```
# option 1: move jellyfish to a directory already in your PATH
sudo mv jellyfish-macosx /usr/local/bin
```

`/usr/local/bin` is a directory where user installed programs often get put.

Note that you had to use the sudo (**s**uper **u**ser **do**) command to move jellyfish to /usr/local/bin.  This is because you lacked the necessary permissions to move a file into that directory.  Running commands prepended by sudo is the same as doing something with Administrator priveleges (so be careful!).

The second option would be:
```
# option 2: add ~ to your PATH
export PATH=$PATH:/Users/gdw
```

Here, you are changing the PATH the way you are supposed to, by appending a new directory to the PATH instead of overwriting it as we did above.  So bash interprets `PATH=$PATH:/Users/gdw` as "assign to the variable named PATH the current value of the variable PATH plus ":/Users/gdw" 

After either (or both) of these, jellyfish-macosx should now be in your PATH.  let's see what happens when we try to run it:

```
jellyfish-macosx
```

You should see an error like this:

```
-bash: /usr/local/bin/jellyfish-macosx: Permission denied
```

Arrgh!  This is why people get frustrated with the command line.  Let's power through!

### Permissions

This gets us to another common pitfall: [file permissions](http://linuxcommand.org/lc3_lts0090.php).

Every file and every directory in linux has a set of permissions that tell whether the file or directory can be read, written, or executed.  

Your user lacked write permissions for `/usr/local/bin`, which is why you couldn't move a file into that directory, and had to use the `sudo` command, which gives you super privileges.  

Similarly, the jellyfish-macosx file that you downloaded *did not download with executable permissions*. And in order to be run as a command, a file needs executable permissions.  

Let's check the permissions on jellyfish:

```
ls -l /usr/local/bin/jellyfish-macosx
```

When you list a file using the `-l` option, the first part of the output for each file is the permissions, which look like this:

```
-rw-r--r--
```

These meaning of these permissions are described [here](https://en.wikipedia.org/wiki/File_system_permissions)

You can see that the jellyfish-macosx file indeed lacks e**x**ecutable permissions.  We can change that by running:

```
chmod +x /usr/local/bin/jellyfish-macosx
```

chmod is the linux command that changes file permissions.  

Now try running:

```
ls -l /usr/local/bin/jellyfish-macosx
```

Then:

```
jellyfish-macosx
```

Phrew.  

Jellyfish got a little mad because you didn't supply enough arguments, but at least it ran!  That's a good sign.  You may also note that it outputs usage information about how to actually run it, which most well-written software should do.  You can also see that it has a `--help` option, which is another common feature of good software (sometimes the option is `-h` or `-help`).  

### installing from source code

If you recall, we also could have downloaded the source code for jellyfish and compiled it to make our own executable program file.  Let's go through that quickly, just so you can see what that process looks like. 

First, download and unpack the source code

```
curl -OL https://github.com/gmarcais/Jellyfish/releases/download/v1.1.12/jellyfish-1.1.12.tar.gz
tar xvf jellyfish-1.1.12.tar.gz
```

This should have created a new directory named jellyfish-1.1.12.  We'll change into it and compile the software according to the instructions on the main github page: 

```
cd jellyfish-1.1.12
./configure --prefix=$HOME
make -j 4
```

Note that `$HOME` in the above code refers to the value of the environmental variable HOME, which is the path of your home directory.  When bash runs the above code, it will replace `$HOME` with the value `/Users/gdw`

You will see a bunch of computer sciencey things print to the terminal as you run this command.  There may be a warning, but there should be no errors, which would indicate that compilation failed.  This compilation process created a binary executable file in the bin directory.  Look at it by running:

```
ls -l bin
```

This process created a file named jellyfish that is the same size as the jellyfish-macosx file that we downloaded previously.  Let's try running it.  Note that this directory is not in your PATH, so we'll have to refer to it explicitly by running:

```
bin/jellyfish
```

You should see the same output that you saw previously. 

Note that different software will have different installation / compilation instructions.  Good software has good installation instructions and will hopefully be easy to install.  

### Conda environments

Many commonly used bioinformatics software is available as conda packages.  For instance, here is a [conda package for jellyfish](https://anaconda.org/conda-forge/jellyfish).

"[Conda](https://docs.conda.io/en/latest/) quickly installs, runs and updates packages and their dependencies".  Conda is a great way to handle installing software and avoids many of the pitfalls associated with manual installation mentioned above.  We will not work directly with conda today but it may well be something worth looking into on your own. 


## Operating more efficiently in the command line environment

There are a number of simple tricks that will enable you to operate more efficiently in the command line environment.


### Tab completion

[Tab completion](https://en.wikipedia.org/wiki/Command-line_completion) will allow you to type less by automatically filling in the names of files and commands when possible.  It works in 2 ways:

1. If you are at the beginning of a command line, it will complete using the already typed characters and the names of the commands available in your PATH.  

2. If you are in the middle of a command line, it will complete using the already typed characters and the names of files and directories in your pwd.  

If you hit `tab` one time, and there is an unambiguous matching command or file that could be completed, it will complete it as far as it can until there is ambiguity again.  If you hit `tab` twice, it will show you all the possibly matching files or commands.


##### command completion 

For instance, say you want to run bowtie2.  You could type out the program name, or you could:

- enter `bow` on the command line and hit `tab` once.  It should complete to `bowtie2` because there are no other commands in your PATH that begin with `bow`
- try hitting `tab` twice more at this point.  You will see the other commands that begin with bowtie2.
- enter `b` on the command line and hit `tab` twice.  You will see all the commands that begin with `b`.

##### filename completion 

`cd` to get to your home directory.  Then type:

- `ls D` then hit `tab` twice.  You should see Desktop, Downloads, and Documents, which are all the directories that begin w/ D
- now type `ls De` and hit `tab` once.  This should complete to Desktop, since De is enough to distinguish Desktop from Downloads and Documents.

Becoming comfortable with tab completion will make your life much easier when typing commmands!  It also helps you avoid typos that make commands fail.

### Wildcards

Wildcards are somewhat related to tab completion in that they allow specified ambiguity.  When bash is interpreting a wildcard character, it will replace the word containing a wildcard with all the possible matching files or directories.  Some useful wildcards are:

- `*`     match any number of any character
- `?`     match one of any character
- `[123]` match one of the characters `1`, `2`, or `3` 

There are a number of wildcards that we won't go into, but you can read more [here](http://tldp.org/LDP/GNU-Linux-Tools-Summary/html/x11655.htm)

For example:

```
ls /usr/local/bin/b*
ls /usr/local/bin/*seq*
ls /usr/local/bin/bowtie2-align-?
ls /usr/local/bin/bowtie2-align-[sl]
```

We will use wildcards more during the workshop.

### Aliases

Creating command aliases can be a convenient shortcut.  They allow you to define a new command or to redefine the meaning of a command, for instance:

```
alias 'ls=ls -G'
ls
alias 'll=ls -l'
ll
```

Want to know what the `-G` and `-l` options do for `ls`?  See `man ls`.

Now, with these aliases, when you type `ls`, the shell will replace `ls` with `ls -G`.  And when you type `ll`, it will replace `ll` with `ls -l`, which will in turn be expanded to `ls -G -l` if you've created both the above aliases.

Note that aliases like these will disappear when you close your terminal window and open a new one.  In order to make them persistent, you need to add them to a file in your home directory called `.bash_profile`.  [Here's](https://www.thegeekdiary.com/what-is-the-purpose-of-bash_profile-file-under-user-home-directory-in-linux/) some more reading on the subject.


## The shell history, pipes, and `grep`

### history

You might want to run a command again or just remember how you ran a command previously.  bash keeps track of the commands you ran and you can view a record of your previous commands using the `history` command.  Try running `history`.


### pipes

One really useful feature of bash and similar shells is the ability to use the output of one command as the input of another command.  This process is called piping.  The symbol `|` in bash is called a pipe.  Here's a simple example of how you could pipe two commands together:

```
# show me the files in /usr/local/bin that begin with bowtie
ls /usr/local/bin/bowtie*

# use the wc -l command to count those files
ls /usr/local/bin/bowtie* | wc -l
```

### grep

`grep` is a command that will search for a pattern in its input and report lines that match that pattern.  The input to `grep` could be a file or it could be input from a pipe, as in this example:

```
# show me the files in /usr/local/bin
ls /usr/local/bin

# in the output of that command, search for files that match the pattern "bowtie"
ls /usr/local/bin | grep bowtie

# you can pipe multiple commands together to create a "pipeline"
ls /usr/local/bin | grep bowtie | wc -l
```

Note that this last command did the same thing as `ls /usr/local/bin/bowtie* | wc -l`, which we ran earlier.  It's also the same as `ls /usr/local/bin/bowtie* | grep -c bowtie`.  In bash there are often many ways to do the same thing!


## bash scripting

Any file that contains bash commands is a bash script!  A bash script must have executable permissions to be run.

Storing commands in simple bash scripts is helpful to:

1. Document the commands you ran to do something: this can be a form of record-keeping, like notes you'd make in a lab notebook.
2. To increase efficiency by automating tasks that you will do repeatedly.

Let's make some simple scripts to demystify the process.  First, let's make a script that measures the disk usage of the files in the present working directory.  Open BBEdit, and add the following one line to a file: 

```
du -sch *
```

The [du command](https://www.tecmint.com/check-linux-disk-usage-of-files-and-directories/)  measures disk usage of files or directories.

Save the file to the Desktop, naming it check_disk_usage (delete the .txt part of the filename, though it would run fine with that left on).  

Now find the file:

```
cd ~/Desktop
ls -l check_disk_usage
```

What would happen if you tried to run this script?  What command do we need to run to fix it?

<br><br><br><br> <br><br><br><br> <br><br><br><br> <br><br><br><br> 

That's right, it doesn't have executable permissions.  Let's add them using `chmod`

```
chmod +x check_disk_usage
```

Now we can run this script and it will execute whatever commands are listed in it.  In this case, it will run `du -sch *`

Since ~/Desktop is not in our PATH, we have to specify the full path to the script using ./ (assumes pwd is ~/Desktop)

```
./check_disk_usage
```

Bash is not just a command line shell but is also a full-fledged programming language. Your script can include flow control (if/else statements, loops), and variables.  Let's make a script that includes a for loop and a variable.  Open BBEdit and enter this into a file:

```
# a for loop 
for bowtie_file in /usr/local/bin/bowtie*
do

   # $bowtie_file is a variable whose value changes each loop.
   echo "I found this bowtie file in /usr/local/bin: $bowtie_file"
done
```

Save this file to the Desktop, naming it bowtie_script or something like that, give it executable permissions, and run it.  Did it work?

Remember that you can put into a script any command or series of commands that you woul run on the normal command line.


## Finding things in Linux 

In this section, we will discuss how to find particular files in linux and also how to find text within files.

### Finding files in linux (time permitting)

The `find` command is a useful tool for finding files with specific attributes. 

You need to tell find where to look for files and what type of files to look for.  For instance, running this command:

```
find .
find /usr/local/
```

will find and report all the files in the present working directory.

That's not that interesting.  What's more useful is finding a particular type of file.  For instance, we could find all the fastq files in your home directory by running:

```
# change to your home dir
cd 

# find files beginning with jellyfish
find . -name "jellyfish*"
```

You can also combine finding files with performing actions on them.  For instance, say we wanted to list the # of lines in each of those fastq files, you could run a find command like this:

```
find . -name "*.fastq" -exec wc -l {} \;
```

The -exec option to find allows you run another command on each found file.  Note that the syntax is strange looking and frankly difficult to remember.  I personally have to google it every time I want to do this.


`locate` is another linux tool that enables you to find files.  It is like `which`, but doesn't just search the PATH.  It is more like Spotlight in MacOSX.  It works by having a database of all the filenames in the filesystem that it can quickly search for a pattern.  (Unlike Spotlight, it doesn't search the contents of files, though, just the filenames.)

For example:

```
locate bowtie2
```

How does the output of this command differ from that of `which bowtie2`?

How might you combine the `grep` command with the find command to search _within_ files?


### Time permitting, screen

A common scenario is to connect to a remote server to kick off a long running task like a genome assembly.  In this case, if your connection is not stable, for instance if you connected using ssh over wifi from your laptop, then you risk having the process prematurely stop when your connection closes.  The `screen` utility allows you to establish a more permanent terminal session that will not close when you lose your connection.

Here are some commonly used screen commands:
```
# Start a screen session
screen 

# Start a named screen session
screen -S my_assembly_session

# Detach from a session.  The detached session will persist even though you are detached from it.
# You will probably be automatically detached if you lose your connection.
ctrl-a d  

# exit from a session and close it.
exit  

# list your screen sessions
screen -ls --> list sessions

# reattach to a detached session
screen -r  

# reattach to a named session
screen -r session_id  
```

Let's try running screen. 

First, let's start a session and give it a name.  From anywhere:

```
screen -S session_1
```

You should see an intro screen.  Press `space` or `enter` to continue.

Let's kick off a long running process.  In this case, let's run the `sleep` command, which does nothing for a specified # of seconds:

```
# Note that we are running this within a screen session:
# sleep for 3 minutes
sleep 180
```

OK, `sleep` is running.  Let's detach from your screen session by typing `ctrl-A d` (press the `ctrl` and `a` keys simultaneously then press the `d` key by itself).  You should return to the session from which you ran screen.

Let's see if your session is persisting:

```
screen -ls
```

Do you see your session?  Let's reattach to it:

```
screen -r session_1
```

You should now be back in your session, with the sleep command still sleeping.  

When I run long commands remotely, I almost always do so using `screen`. 


<br><br><br>

### Connecting to remote servers 

This section is for reference only - we will not be connecting to remote servers today

#### ssh remote shell

We already used `ssh` to connect to a remote server when we were checking the resources available on the cctsi-104 server.  `ssh` provides a shell (like bash) on a remote server.   Let's try connecting again to the cctsi-104 server:

```
ssh gdw@cctsi-104.cvmbs.colostate.edu
```

Once logged in, use `ls` to see what files are in the gdw user's home directory on cctsi-104.  

#### sftp file transfer

`sftp` is a command line tool that allows you to copy files back and forth from remote servers.  (`sftp` is the secure (encrypted) version of `ftp`, **f**ile **t**ransfer **p**rotocol).

Let's get those dataset (.fastq files) from the cctsi-104 server.  

```
sftp gdw@cctsi-104.cvmbs.colostate.edu
```

`sftp` has some limited shell-like functionality.  For instance, you can `cd`, `ls`, and `pwd` from within your sftp session.  By default, you will be logged into your home directory.  Let's double check that those fastq files are there and then use the `get` command within sftp to transfer them from the remote server to your local computer:

```
# note, here we are within the sftp session
pwd
ls 
# wildcards work within sftp
get *.fastq 
```

Now exit from the sftp session by typing `exit`

You should see that you transferred 3 fastq format sequence files to whatever directory you were in when you ran `sftp`.

`put` is the opposite of `get` in sftp.  You can use it to transfer files _to_ a remote server from the command line.



