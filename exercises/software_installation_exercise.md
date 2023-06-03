# Software Installation Exercise

Installing and managing software in a command-line environment can be a challenge.  Let's practice installing a bioinformatics tool named jellyfish from the command line.  We will do this two different ways:

1. We will download a jellyfish binary: that is, a program ready to run. 

2. We will install jellyfish using [conda](https://docs.conda.io/en/latest/), which is a tool that is useful for installing bioinformatics software.


### Download the jellyfish binary (program file) from github

[Jellyfish](https://github.com/gmarcais/Jellyfish) is a tool for counting [k-mers](https://en.wikipedia.org/wiki/K-mer).  The jellyfish github repository contains a [releases page](https://github.com/gmarcais/Jellyfish/releases) that includes [a jellyfish binary](https://github.com/gmarcais/Jellyfish/releases/tag/v2.2.10) ready to use on MacOS.  

Let's download the jellyfish binary.  Download the file named jellyfish-macosx from [this releases page](https://github.com/gmarcais/Jellyfish/releases/tag/v2.2.10).  We will use the [curl utility](https://en.wikipedia.org/wiki/CURL) to do this.

``` 
# go to your home directory
cd

# download jellyfish using curl
curl -OL https://github.com/gmarcais/Jellyfish/releases/download/v2.2.10/jellyfish-macosx
```

Type `ls -lh` to confirm that you have downloaded a file to the pwd named jellyfish-macosx that has a size of 623 kb.  The `-h` option to `ls` outputs file sizes in a **h**uman readable format and the `-l` option outputs a more detailed "**l**ong" listing.

Now that we have the file, let's try to run it.  That was the point of downloading it, after all.  Try typing:

```
jellyfish-macosx
```

What is going on???  How can you fix this?  

<br><br><br><br> <br><br><br><br> <br><br><br><br> <br><br><br><br> 

One issue is that the file jellyfish-macosx is not in your [PATH](https://www.digitalocean.com/community/tutorials/how-to-view-and-update-the-linux-path-environment-variable).  The jellyfish-macosx file is in your home directory, which is not in your PATH.  Let's confirm this by using the which command:

```
# is jellyfish-macosx in your PATH?
which jellyfish-macosx

# what is your PATH?
echo $PATH
```

To fix this situation, we could either:

1. move the jellyfish-macosx file into a directory in your PATH, or 
2. add your home directory to your PATH [generally not recommended], or  
3. run the jellyfish-macosx command by specifying its location (e.g. `./jellyfish-macosx`)

Let's do option 1: move jellyfish-macosx to a directory already in our PATH.

```
# move jellyfish to a directory already in your PATH
sudo mv jellyfish-macosx /usr/local/bin
```

`/usr/local/bin` is a directory where user installed programs often reside.  

Note that you had to use the sudo (**s**uper **u**ser **do**) command to move jellyfish to /usr/local/bin.  This is because you lacked the necessary permissions to move a file into that directory.  Running commands prepended by sudo is the same as doing something with Administrator priveleges (so be careful!).

After moving jellyfish-macosx to /usr/local/bin, it should now be in your PATH.  let's see what happens when we try to run it:

```
jellyfish-macosx
```

Arrgh!  This is why people get frustrated with the command line.  What's happening now and how can we fix?

<br><br><br><br><br><br><br><br><br><br>

#### Permissions

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

[chmod](https://www.ibm.com/docs/en/aix/7.2?topic=c-chmod-command) is the linux command that changes file permissions.  

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

### installing jellyfish using conda

Many commonly used bioinformatics software is available as conda packages.  For instance, here is a [conda package for jellyfish](https://anaconda.org/bioconda/jellyfish).

"[Conda](https://docs.conda.io/en/latest/) quickly installs, runs and updates packages and their dependencies".  Conda is a great way to instal software.  Advantages of using conda include:

1. It's possible to specify exact versions of software to use, which is an important component of [reproducible computational research](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1003285#s4)
2. Each user on a multi-user computer (like a server) can have their own software installed and can use different versions of the same software.
3. You can create multiple conda environments, allowing you to create different software environments for different tasks.
4. Conda works well on multiple operating systems.


In order to use conda, you must first [install the conda software itself](https://conda.io/projects/conda/en/stable/user-guide/install/index.html).  Conda has already been installed for you on the workshop laptops.  

You can tell conda is installed because your command line prompt should begin with `(base)`.  This indicates that you are in your base conda environment.  You can also check that conda is installed by running `which conda`. 

If you want to see what software is installed in your base conda environment, run:

```
# list all packges installed in current active conda environment
conda list
```

Let's create a new conda environment, named jellyfish, that just contains the jellyfish software.  Instructions for creating and managing conda environments [are here](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#managing-environments).  You can search for different conda packages [here](https://anaconda.org/).  Jellyfish is available in the [bioconda channel](https://bioconda.github.io/). 

```
# create a conda environment named jellyfish that includes jellyfish
conda create -n jellyfish bioconda::jellyfish
```

The installation process will run for a minute and prompt you if you want to continue (answer yes [y]).  

Once installation is complete, you should see a message that tells you how to activate the jellyfish channel, by running:

```
conda activate jellyfish
```

You should see your CLI prompt switch to `(jellyfish)` to indicate that this is your active conda environment.

Let's try to understand how conda works.  With the jellyfish conda environment active, run this command:

```
which jellyfish
```

What was the output?  

What about this command:

```
echo $PATH
```

What happens if you run these commands:

```
# deactivate the jellyfish conda environment
conda deactivate
which jellyfish
echo $PATH
```

Now what output do `which jellyfish` and `echo $PATH` produce?  What does this tell you about how conda works?

Conda environments can include many different software tools.  [Here's an example](https://github.com/stenglein-lab/MIP_280A4_Fall_2022/tree/main/conda_environment) of a conda environment that includes many commonly-used bioinformatic tools.

