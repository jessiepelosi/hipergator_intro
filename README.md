# Introduction to Using HiPerGator

This is a general introduction for using the University of Florida's HiPerGator Computing Cluster. 

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

# 3. SLURM Jobs 

Since HiPerGator is used by thousands of people across the University, we cannot (generally) run big jobs interactively. 
 
