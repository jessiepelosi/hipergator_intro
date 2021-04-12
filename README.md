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
