# Introduction to Using HiPerGator

This is a general introduction for using the University of Florida's HiPerGator Computing Cluster. 

# 0. Your Computer

You'll need a basic understanding of your own computer before being able to log into HiPerGator. I've split this up into Mac/Linux and Windows. You'll also want to download a text editor- you shouldn't write code in Microsoft Word! Here are some text editors that I've used before and work pretty well: 
1. [Sublime Text](https://www.sublimetext.com/) 
2. [VSCode](https://code.visualstudio.com/)
3. [Text Wrangler/BBEdit](https://www.barebones.com/products/bbedit/) 
4. [Notepad++](https://notepad-plus-plus.org/downloads/) 

It is okay to write your code in any of these text editors (or use built-in ones on HiPerGator) and then transfer that text to a Word document to keep track of what you've done. 

<b> Windows </b> 

In order to transfer files between your computer and the cluster, you'll need to download a Secure File Transfer Protocol (SFTP) client. Two commonly used SFTP clients are: 
1. [WinSCP](https://winscp.net/eng/docs/free_sftp_client_for_windows) 
2. [Filezilla](https://filezilla-project.org/) 

Connect to hpg.rc.ufl.edu using one of these clients following their inscructions. You want omake sure you are connecting through SFTP (you should be connecting through Port 22). Log in with your GatorLink username and password. 

Accessing the command line: Using the command line on Windows machines is a bit different than using a Mac or Linux machine. If you are using Windows 10, search (using the search bar on the bottom left of the taskbar for "CMD". Open the application called "Command Prompt". You should be able to continue to the next step (#1 Logging In and Moving Around). If you are not able to do so, you may have to install [GitBash](https://gitforwindows.org/).  

<b> Mac/Linux </b> 

In order to transfer files between your computer and the cluster, you'll need to download a Secure File Transfer Protocol (SFTP) client. A commonly used SFTP client on Mac/Linux machines is [Cyberduck](https://cyberduck.io/). Connect to hpg.rc.ufl.edu using one of these clients following their inscructions. You want omake sure you are connecting through SFTP (you should be connecting through Port 22). Log in with your GatorLink username and password. 

Accessing the command line: On a Mac/Linux machine, you can log into HiPerGator using the "Terminal" application. Once you've opened the Terminal, you can continue to the next step (#1. Logging In and Moving Around). 

# 1. Logging In and Moving Around 
```
ssh username@hpg.rc.ufl.edu
```

When prompted, type in your password. You'll be moved into your home directory, but do not do any work in this directory! You'll have a directory within your PI's cluster. The `cd` command stands for "change directory". For example: 
```
cd /blue/sessa/username
  
cd /blue/barbazuk/username
```
Once you're in your directory within your PI's cluster, you can make folders, files, etc. Let's start by making a new folder called "scratch".
``` 
mkdir scatch
```
Now to make sure you made that folder, you can use the `ls` command to list the contents of your directory. There's some extra "flags" or extra semantics to pass to the command. With `ls` I tend to use two flags that you can combine, the `-l` for the "long version" of the listing, and `-h` for a human-readable list view. Try them out! 
``` 
ls 
ls -l 
ls -h
ls -lh 
```
If you want to move into your new "scratch" folder, you can simply: 
```
cd scratch 
```
Now if you type `ls` you'll notice that nothing is in this folder yet! If you want to go back one step to your folder in your PI's cluster, you can use `cd` again, but instead of typing the whole path (e.g., `cd /blue/barbazuk/username`), use can use a relative path. 
```
cd ..
```
The double periods signify the directory right above the one you are currently in. A single period (`.`) signifies the directory you are currently in. 

To see where you are at any time, you can use the "print working directory command" `pwd`. 
```
pwd
```
  
# 2. Copying, Editing, and Moving Files 

Although you can make files using a file-transfer program such as WinSCP, you can also make new files right from the command line. I prefer to use the program `nano` which is already installed on HiPerGator from every log-in node. You can make a new file like this: 
```
nano myfile 
```
`nano` will then bring you to a new screen where you can edit the text file. You'll see something like this: 

![alt text](https://github.com/jessiepelosi/hipergator_intro/blob/main/nano.PNG "nano example")

You can add text just by typing. When you're done adding text, press Control + X at the same time and save the file. Unlike other programs, `nano` tells you what each command does! 

Now that you're out of the `nano` screen, you can copy your new file to your scratch folder like so: 
```
cp myfile scratch/
```
Now change your directory into the scratch folder and list the contents! Try it out first, but the commands are below too. 
```
cd scratch
ls -lh 
```
You can also rename and move your copy; these two processes are in a single command, `mv`. Let's first rename the file: 
```
mv myfile myfile_1
```
In this example, `mv` is the command and it takes two arguments: the first is the current name of the file, the second is the new name. We wanted to first rename the file so when we move it back to your main directory, we don't overwrite the previous version. We can do this: 
```
mv myfile_1 ../
```
Often the files we work with are too long to display on the screen, so we can view parts of them using the `head` and `tail` commands. `head` will display the first ten lines of a file, while `tail` will display the last ten lines of a file. 
```
head myfile_1 
tail myfile_1
```
If you want to know how many lines, words, or characters are in a file, you can use the `wc` command (with various flags):
```
wc myfile_1
wc -l myfile_1
wc -c myfile_1
```
If you're not sure how to use a command or what the flags mean, you can often type `--help` after the command:
```
wc --help
```
This will display general information about the command, flags, and arguments that the command takes. You can always use Google! You're soon realize that learning to efficiently Google will be a key skill in any type of coding. 

One of the most useful tips for working on the command line is using the Tab key to auto-complete file and directory names! Try typing this (but don't hit enter yet):
```
cd sc
```
Then press Tab on your keyboard: it will autocomplete the name of the directory (since it's the only thing in your main directory that starts with "sc"). Then hit enter! 

# 3. Big Edits 

While `nano` is nice to make edits on small text files, you'll regularly be working with large files thousands of lines long and you won't be able to make edits using just `nano`. In many of these cases, you'll want to find a certain string (a series of characters) in your file and replace it with a different string. For example, imagine you are working with a fasta file that looks like the following and you want to replace `QAIV_1KP_ID:12345679` with the taxon name (e.g., <i> Dryopteris ludoviciana</i>).  
```
>QAIV_1KP_ID:123456789 Contig 1
ACTAGGGGTCAGTGAGTCGATGAGTGTGTGCGTGATCGATCGCCTAGCTAGCTACGTAGC
>QAIV_1KP_ID:123456789 Contig 2
AGTCTGATCGATCGATCATAGAACGAATATAGAGCTAGCTAGCTAGCTAATATAAAAAAA
```
And so on. You don't want to have to edit every single header individually- that would take way too long! In these cases you can use find-and-replace tools such as `sed`, `awk`, and `grep` to help. I'll summarize these command below. You'll also want to know some basic Regular Expresions (RegEx) for using these tools. Some information about RegEx is at the bottom of this section. 

<b>grep</b> 

`grep` searches through a file looking for a certain pattern. 

<b>awk</b>

<b>sed</b> 

<b>RegEx</b>

Regular Expressions are incredibly helpful and it is worth your time leaning them. These are essentially tools that you can use to find (or capture) a diverse array of characters for find-and-replace commands. Dr. Matt Gitzendanner has put together a very nice series of videos illustrating the power of RegEx and some simple examples [here](https://comptoolsres.github.io/TLCL_3.html). Note that there are some expressions that aren't consistent across platforms. I also prefer to use brackets for some expressions (such as digits). Some of the most common expressions I've used are listed below: 

| RegEx         | Captured Characters       |
| ------------- |:-------------------------:|
| [0-9] or \d         | any digit                 | 
| [A-Z]         | any letter (captial, depending on if that command is case-sensitive)      |
| [a-z]         | any letter (lowercase, depending on if that command is case-senstitive)   |
| .             | any character             |
| \s            | white space               |
| \t            | tab                       |
| \n            | new line (line break)     |

To escape characters such as the period (which usually capture any character) and capture just a period, you can put a backslash (\) in front of that character. For example:

|RegEx | Captured Characters|
| ------------- |:-------------------------:|
|\\.          | this will capture a period |
|\\,          |this will capture a comma|

<b>What do brackets do?</b> Brackets can be used to combine multiple expressions. Say you want to capture a digit <u> and</u> a letter. You can include both of the approriate RegEx commands in a pair of brackets, like this `[0-9A-Z]`. 

<b>How do you capture more than one character at a time?</b> There are several quantifiers that you can use to capture several characters at a time. Here is a short list: 

| RegEx         | Number of Captured Characters       |
| ------------- |:-------------------------:|
|*              |zero or more |
|+              |one or more  |

<b>Capturing and keeping parts of a search.</b> Sometimes you don't always want to replace a whole expression, but only part of it. You can keep parts of your expression by closing them in with parentheses. They stored in a variable that you can retrieve either with "1" or "$1". Take our example from above. 

```
>QAIV_1KP_ID:123456789 Contig 1
ACTAGGGGTCAGTGAGTCGATGAGTGTGTGCGTGATCGATCGCCTAGCTAGCTACGTAGC
```

Say we want to capture the `QAIV_1KP_ID:12345679` but want to keep `QAIV`. We can capture `QAIV` and put it in parantheses and then capture the rest of the header but leave it out of the parentheses (in this case, I want to keep "Contig 1"): `([A-Z]+)\_[A-Z\_\:0-9]*`. If we were using sed:
```
sed -i 's/([A-Z]+)\_[A-Z\_\:0-9]*/1/g' filename.txt
```

There are nice tables (cheat sheets) for RegEx [here](https://www.rexegg.com/regex-quickstart.html). You can practice in Sublime Text by doing "Find and Replace" or "ctrl-H". 

# 4. SLURM Jobs 

Since HiPerGator is used by thousands of people across the University, we cannot (generally) run big jobs interactively. If your command can run in less than 10 minutes, uses less than 64 Gb of RAM, and less than 16 cores, you can run it interactively on the log-in nodes (where you end up when you `ssh` into the cluster). Otherwise, you should pass your commands to the scheduler system, called SLURM, that will find the appropriate time and computer(s) to run your job. For almost everything we do, we will be going through the SLURM scheduler.  

There are few general parts to every SLURM script. You must always begin the script with this block of text:
```
#!/bin/sh
#SBATCH --job-name=your_job_name       # Job name
#SBATCH --mail-type=ALL                # Mail events (NONE, BEGIN, END, FAIL, ALL)
#SBATCH --mail-user=username@ufl.edu   # Where to send mail
#SBATCH --cpus-per-task=4              # Number of cores: Can also use -c=4
#SBATCH --mem-per-cpu=2gb              # Per processor memory
#SBATCH -t 4-00:00:00                  # Walltime
#SBATCH -o your-job-name.%j.out        # Name output file
#SBATCH --account=barbazuk             # Cluster account manager 
#SBATCH --qos=barbazuk-b               # Specify to run on burst (-b) or not 
#
```
We have to tell the scheduler how many resources and how much time the job will take. It can be hard to estimate this if you've never run a job before (or if you're running a job for a new program), but you'll get an idea about these parameters the more you use a program. 

One thing to note is that you can run jobs on the normal queue by specifiying `qos=barbazuk` where you'll be able to run jobs for up to thirty days, or the burst queue (`qos=barbazuk-b`) where you have access to more resources but can only run jobs for four days. If you know your job will take 4 or fewer days, run your jobs on burst. 

On the next few lines you can add commands about where to work; I usually use absolute paths for this. 
```
cd /blue/barbazuk/username
```
Then you can tell HiPerGator you need to use certain programs (called "modules") that are already installed. A full list of installed programs can be found [here](https://help.rc.ufl.edu/doc/Applications). 
```
module load mafft 
```
It is usually best practice to specify which version of a program you want to load (there are often several on HiPerGator). To see more details about a program you can do the following interactively:
```
module spider mafft
```
This will bring up a short description of the program and the versions available. Now we know that we want to load mafft ver. 7.407 so we can edit our SLURM script. 
```
module load mafft/7.407
```
And finally you want to include your command! 
```
mafft --maxiterate 1000 --localpair unaligned.fasta > aligned.fasta 
```
Your full SLURM script would look like this and I like to save them with a ".slurm" file ending.  
```
#!/bin/sh
#SBATCH --job-name=your_job_name       # Job name
#SBATCH --mail-type=ALL                # Mail events (NONE, BEGIN, END, FAIL, ALL)
#SBATCH --mail-user=username@ufl.edu   # Where to send mail
#SBATCH --cpus-per-task=4              # Number of cores: Can also use -c=4
#SBATCH --mem-per-cpu=2gb              # Per processor memory
#SBATCH -t 4-00:00:00                  # Walltime
#SBATCH -o your-job-name.%j.out        # Name output file
#SBATCH --account=barbazuk             # Cluster account manager 
#SBATCH --qos=barbazuk-b               # Specify to run on burst (-b) or not 
#

cd /blue/barbazuk/username

module load mafft/7.407

mafft --maxiterate 1000 --localpair unaligned.fasta > aligned.fasta 
```

To submit your SLURM job: 
```
sbatch myslurm.slurm
```

To check on your job or other jobs: 
```
squeue -A barbazuk       # will print all the jobs on the Barbazuk account 
squeue --qos barbazuk-b  # will print all the jobs ont he Barbazuk burst queue 
squeue -U username       # will print all of your jobs 
``` 

If you need to cancel a job for any reason (you can get the jobID from the above commands): 
```
scancel jobID
```
