# Additional Command-line topics and exercises

There are a number of tools and tricks that will enable you to operate more efficiently in the command line environment.  If we have time, we can work through these topics.

## Topics

- [Tab Completion](#Tab_completion)
- [Wildcards](#Wildcards)
- [Aliases](#Aliases)
- [Pipes](#Pipes)
- [Grep](#grep)
- [Shell scripting](#Shell_scripting)
- [Finding files in linux](#Finding_files)
- [Using the screen utility to avoid dropped connections](#Screen)
- [Paths_and_the PATH](#Paths_and_the_PATH)
- [Connecting to remote servers](#Connecting_to_remote_servers)
- [The shell history - a record of commands you've run](#history)

## Tab_completion

[Tab completion](https://en.wikipedia.org/wiki/Command-line_completion) will allow you to type faster and make fewer typos by automatically filling in the names of files and commands.  It works in 2 ways:

1. If you are at the beginning of a command line, it will complete the names of the **commands**.

2. If you are in the middle of a command line, it will complete the names of **files and directories**.

If you hit `tab` one time, and there is a command or file that could be completed, it will complete it as far as it can until there is ambiguity (multiple matching commands or files).  If you hit `tab` twice, it will show you all the possibly matching files or commands.


### command completion 

For instance, say you want to run bowtie2.  You could type out the program name, or you could:

- enter `bow` on the command line and hit `tab` once.  It should complete to `bowtie2` because there are no other commands in your PATH that begin with `bow`
- try hitting `tab` twice more at this point.  You will see the other commands that begin with bowtie2.
- enter `b` on the command line and hit `tab` twice.  You will see all the commands that begin with `b`.

### filename completion 

`cd` to get to your home directory.  Then type:

- `ls D` then hit `tab` twice.  You should see Desktop, Downloads, and Documents, which are all the directories that begin w/ D
- now type `ls De` and hit `tab` once.  This should complete to Desktop, since De is enough to distinguish Desktop from Downloads and Documents.

Becoming comfortable with tab completion will make your life much easier when typing commmands!  It also helps you avoid typos.

<br> <hr> <br>


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




### Pipes

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


## Shell_scripting

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


### Finding_files

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

How could you combine the `grep` command with the `find` command to search _within_ files?

### Screen

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

## Paths_and_the_PATH

Let's briefly revisit absolute paths, relative paths, and the PATH.  

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

The `man` (manual) command is a great way to get more information about built-in linux commands.  You can page through a manual page using the up and down arrows or the space bar.  Press q to exit.  

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

<br><hr><br>

## Connecting_to_remote_servers 

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

### history

You might want to run a command again or just remember how you ran a command previously.  bash keeps track of the commands you ran and you can view a record of your previous commands using the `history` command.  Try running `history`.
