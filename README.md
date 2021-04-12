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



# 3. SLURM Jobs 
 
