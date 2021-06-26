--- 
toc: true
toc_sticky: true
excerpt: "Getting you up and running R on
the UC Davis FARM computing cluster."
---

## Intro

This post is essentially a writeup of a \~2 hour in-person lesson I did
for a few friends who were interested in using the UC Davis CAES FARM
computing cluster. They both had a little experience, but not much,
working from the command line, and we were all on Macs, so it was pretty
easy to sit next to each other and have them follow along. Both of them
had similar needs as well: getting some long-running R scripts onto the
FARM to free up their computers, and maybe get some performance boosts
along the way.

This is by no means a comprehensive lesson on high performance
computing, nor is it a comprehensive lesson on using the Unix shell, nor
is it a compr… You get the idea: I’m not claiming to know all that much
about anything. This lesson is intended to help a novice get their R
scripts running on the FARM, using whatever strategy or philosophy I’ve
figured out to get **my** R scripts running on the FARM.

#### What Will Be Covered

  - a few commands in Unix shell
  - accessing the FARM with SSH
  - moving files back and forth between FARM and your computer
  - basic SLURM commands on the FARM
  - pairing an R script and a SLURM submission script
  - general format for saving R results/outputs
  - how to install R packages on the FARM

#### What Won’t Be Covered

  - parallelization of code
      - this is pretty specific to your task
      - I’m not good enough to feel comfortable giving a general
        overview
  - anything on Windows
      - ¯\\*(ツ)*/¯
  - a comprehensive description of any single topic
      - I’m not building the base of a pyramid here, I’m building a
        rickety ladder that will hopefully get you where you need to go
      - you can make that ladder less rickety or use it to access more
        places by building up a better foundation in the topics I’m
        touching on
  - other clusters
      - I’ve only ever worked on one cluster
      - most of this basic stuff should carry over to most clusters

## Unix Shell Basics

Terminal, command line, and shell, oh my\! I’m going to do a quick
rundown of these terms, to start. A Terminal is a program that lets you
type text in, and get text back as a response. The command line is just
the line where you type in those text commands. Finally, a shell is an
application that interprets those text commands, almost like a
translator between you and the computer. It takes the text you type and
translates it for the computer, the computer does something with those
instructions, and then the shell translates what the computer gives as
its response. The default shell for Macs and many Linux distributions is
`bash`, which we’ll be using today.

Go ahead and open up the Terminal application on your computer. You
should be greeted by some type of **prompt**, which will probably
involve something with your username, and it’ll end with the dollar sign
$. Once you’re here, type `ls` and hit `Enter`. Congratulations, you’ve
just run your first shell command\! `ls` will list all the files and
folders in your current directory, which should be your user by default.
Now try typing `pwd` and hitting enter, which will *print working
directory*, or where your shell is currently working. On my computer, it
is `MJ`, my username. You can also use `~` as a shortcut for your Home
directory.

<script id="asciicast-aRiIkcjPAW49sE0gBtSEURCL1" src="https://asciinema.org/a/aRiIkcjPAW49sE0gBtSEURCL1.js" async></script>

*Quick note*: all of these recordings were made with [asciinema](https://asciinema.org/), and you can play/pause them, rewind them, and even **copy the text** from them!

You can add *options* to a command like `ls`, like `ls -a` to list
**all** the files in your directory, including *hidden files* (which
we’ll get to in a minute), `ls -l` to list files in a longer format,
or `ls -t` to sort by time last modified. Let’s try running `ls -at` to
sort all your files by the time they were last modified.

<script id="asciicast-QAH6GuniCpKCxnGNzcRo9MIoM" src="https://asciinema.org/a/QAH6GuniCpKCxnGNzcRo9MIoM.js" async></script>

You can check the *manual* page for any function by using the command
`man`. For example, `man ls` will bring up the manual page for the `ls`
function, which includes all of the possible options you can use. You
can scroll up and down the manual page and press `q` to exit the manual
page.

Another key function we’ll use is `cd`, which we use to change the
current working directory. For example, if I’m in my home directory (my
username, MJ), which contains a Documents folder, I can use the command
`cd Documents/` to change my working directory to `MJ/Documents`. You
can type `pwd` to verify where you’ve moved to, and `ls` to list all the
files in your new working directory. To go up one level from your
current working directory, like moving from `MJ/Documents` up to `MJ`,
you type `cd ..`. `..` just means “up one level”.

<script id="asciicast-ZBgAUybHJYkbt3N2cXs1UJGCo" src="https://asciinema.org/a/ZBgAUybHJYkbt3N2cXs1UJGCo.js" async></script>

We will be using `cd` and `ls` a ton, and we’ll introduce other commands
as we need them.

## Showing Hidden Files

You may have noticed some strange files when I ran `ls -a` on my
computer, a whole bunch of files that all begin with `.`. These are
called “hidden files”, and by default, Finder on a Mac will not show
them. They typically deal with “under the hood” stuff on your computer,
and we’re about to get a little bit “under the hood”.

If you can see all the hidden files in your Home directory in Finder,
then you can skip the next section. If you **don’t** see them in Finder,
we’ll need to change that. If you’re on Windows, Google around a little
bit to see if this is even a problem for you, I honestly have no idea.
Linux should show them by default.

We’re going to set up your Mac to permanently show hidden files any time
you’re looking around in Finder, as this is going to be important in the
future. Copy-paste the following code into your Terminal: `defaults
write com.apple.finder AppleShowAllFiles YES`. Now hit Enter.

Now go up to the apple icon in your menu bar, select Force Quit, select
Finder, and click the Relaunch button. Now all your hidden files should
show up in your Finder. We’ll be looking at some of these files later
on.

## Making a FARM Account

If you go to [the FARM official
website](https://wiki.cse.ucdavis.edu/support/systems/farm) and scroll
down to the Access Policy section, where you’ll find a link to the
[Account Request Form](http://wiki.cse.ucdavis.edu/cgi-bin/index2.pl)
and instructions on making an account. Please follow these instructions.
When you log in to the Account Request Form, it will ask you to upload
an *SSH public key*. We’ll go through this process next.

## Generating an SSH Key

SSH is a widely-used protocol for securely logging into a computer from
another computer. Since the FARM is basically another gigantic computer,
this is what we’ve gotta do.

The way SSH works is that you generate a **key pair**. You can think of
this as a pair of extremely weird and long passwords that recognize each
other. One is your **public** key and the other is the **private** key.
As the names suggest, your public key will get shared with the other
computer you want to log into, and the private key stays on your
computer and **should never ever ever be shared**. I don’t know enough
to say “well actually it’s ok in this circumstance”, and if you’re
reading this, neither do you, so just never ever share it, ok?

To generate a key pair, we’ll use the command `ssh-keygen`. Hit Enter.
You will then be prompted to `Enter file in which to save the key
(your_home_directory/.ssh/id_rsa):`. **Just hit enter** to put the keys
in the default location. Next, you’ll be prompted to enter a passphrase.
Choose a hard password, but remember it. This isn’t Gmail, if you forget
this password, there’s no way to get it back. **As you type, nothing
will show up**, and this is ok. Just type out your passphrase and hit
Enter when you’re done. You’ll have to retype it again, and then press
Enter again. You should now get a confirmation that the key pair was
created.

These keys now live in your `.ssh` folder, which resides in your Home
directory. You should check to make sure you can get to this location in
your Finder. Go look in your Home directory in Finder, and look for the
`.ssh` folder. Go into this folder, and you shoud see your private
`id_rsa` and public `id_rsa.pub` files.

![](/assets/rmd-images/farm-cluster-intro/ssh_finder.gif)

Now, on the [Account Request
Form](http://wiki.cse.ucdavis.edu/cgi-bin/index2.pl), where it says to
upload your public `id_rsa.pub` key, you should be able to click the
button and navigate to this file and upload it. **Make sure it is the
public key you are uploading**. If, at this point, you cannot see your
`.ssh` folder (it’s still hidden), try restarting your internet browser.
Once the file is uploaded, finish off the instructions on making your
FARM account. You should get an email when your account is set up and
you’re able to access the FARM. Be sure to write down your username and
any other info you’re given.

## Set Up Known Host

Once you’ve been notified that your FARM account has been created, you
should be able to log on to the FARM\! Before that, we’ll set up some
files so that your computer recognizes the FARM as a “Known Host”. That
way, all we’ll have to type to log on to the FARM is `ssh farm`.

To make the necessary file, I’ll introduce you to another little command
line program that will come in very handy: a text editor called `nano`.
`nano` is a very simple text editor that comes standard on most Mac and
Linux systems, and you access it from the command line. It’s pretty
barebones, and I wouldn’t do anything too intense with it, but for
simple tasks, it’s very convenient.

If you type `nano file_name`, it will open nano on that file. If you
type `nano new_file_name` it will create a blank file with that name and
open nano to edit it. We are going to make a file in our `.ssh`
directory called `config`, which will allow us to set up the FARM as a
known host. First thing we’ll need to do is `cd` into our `.ssh`
directory. Assuming you’re already in your home directory, you can get
there by typing `cd .ssh`. Now use `ls` to check and see what’s inside.
You should see your `id_rsa` and `id_rsa.pub` files.

Now type out `nano config`. What you see now is the `nano` text editor,
editing a blank file called `config`. `nano` might look a bit funky, but
it’s pretty simple. Type out the text below:

    VerifyHostKeyDNS yes
    Host farm
       HostName agri.cse.ucdavis.edu
       User your-user-name

<script id="asciicast-kk9nwlj5JRaA1PBHZkygqTlbt" src="https://asciinema.org/a/kk9nwlj5JRaA1PBHZkygqTlbt.js" async></script>

We’re telling our `ssh` protocol that there’s a host “farm” that you can
access with a certain location and username. Now, press `CTRL-X` to exit
`nano`. You’ll be prompted to write the file before quitting, then asked
where to write the file. Press enter to write the file to the name you
already gave. You should now be back in your regular Terminal, and if
you run `ls`, you’ll see there’s a new file called `config`.

## Logging On and Looking Around

Now that we’ve told our `ssh` protocol that we have a host called
“farm”, let’s log into it. Open up a new Terminal window and put it
next to your current one. I always have one window open to access the
FARM and one to access my own computer. In the new Terminal window, type
`ssh farm`. This very first time, you may be prompted to accept the FARM
as a known host, just go ahead and approve it. You may also be prompted
to put in your ssh password, go ahead and enter it. If everything goes
correctly, you should be met with a greeting message telling you that
you’re on the FARM\! You can type `exit` to log off the FARM and get
back to your own computer’s command line.

In your FARM Terminal window, go ahead and run `pwd` to see where you
are. This directory should be your username, and from now on we’ll refer
to this as your **FARM home directory**. Use `ls` to see what’s here.

<script id="asciicast-yDlpQexokap99NCGs3uVKzNHT" src="https://asciinema.org/a/yDlpQexokap99NCGs3uVKzNHT.js" async></script>

## Making Directories and Files

You shouldn’t have any folders yet, so let’s make a folder in you FARM
home directory called `testing` by using the command `mkdir testing`.
We’ll be putting all of the files and folders we make for this little
tutorial into this folder. This reflects a general principle I will
advocate for when working on the FARM: use a consistent and nested
directory structure. In other words, put everything into its own folder.
For example, I like to have a folder for each project, then folders for
data, scripts, and outputs. Each of these contains subfolders as well,
and so on. You can definitely overdo it, and it can be annoying to dig
10 levels down to find your files, but I think the worst thing you can
do is just throw files onto the FARM willy nilly.

<script id="asciicast-blIbjGZF95PonDwPafOJiRYbG" src="https://asciinema.org/a/blIbjGZF95PonDwPafOJiRYbG.js" async ></script>

Another good general practice is to make a `README` file to just give a
brief explanation of what you’re doing with a given folder. Let’s use
`cd testing` to move into our testing folder, and then use `nano
README.md` to create a README Markdown file. Markdown is a simple plain
text file format that’s useful for taking notes and stuff. Once you’re
in `nano`, just give a brief description of what the `testing` folder is
for. Once you’re done, use `CTRL-X` to exit. You’ll be asked if you want
to “save the modified buffer”, and you should press `y`. Then it will
say “file name to write to”, and you should hit `ENTER`, since we want
to accept the file name `README.md`.

<script id="asciicast-syBGh9ZEC5kizenN22YYkkefB" src="https://asciinema.org/a/syBGh9ZEC5kizenN22YYkkefB.js" async></script>

## `rsync` Basics

Alright, we’ve now made some files on the FARM, but what if we want to
work with files you can’t just make from scratch, like data? This will
require moving files back and forth between the FARM and your computer,
which we’ll do with a command called `rsync`. `rsync` is a powerful and
flexible way to move files back and forth. The basic syntax is `rsync
options source destination`. I just use the same set of options every
time I use `rsync`, and I’ll describe them briefly: `-a` will move files
recursively, meaning it will move a folder, plus everything it contains,
and it’ll keep timestamps and everything like that; `-v` means verbose,
and it just means you’ll get a little more info on what files are moving
around; `-z` will compress files so they move quicker; `-e` will allow
us to specify the proper port to access the FARM through (don’t worry
too much about this, there’s just a dedicated route for moving files to
and from the FARM, which we’ll use).

All this leads to a general pattern of using `rsync`, which looks like
this: `rsync -avze 'ssh -p 2022' source_path destination_path`. The `ssh
-p 2022` is just us telling the FARM which port we want to access it
through. This will make the FARM happy, and that’s all you really need
to know about it. One more thing to note: file paths for files on the
FARM look a little funky. Here’s what the path would look like for the
`README` in my `testing` folder:
`mjculsha@farm.cse.ucdavis.edu:/home/mjculsha/testing/README.md`. You
need to include some information on how to get to the FARM itself.

We will **always** run `rsync` on our own computers and not on the FARM.
It’s not that you’ll mess anything up if you try (don’t quote me on this
though), it just won’t work. This establishes another part of your
workflow: you’ll almost always have 2 Terminal windows open, one
accessing the FARM and one on your own computer. Make sure you’ve got
that setup going on your computer now. On your own computer’s Terminal
window, use `mkdir` to create a folder in your home directory called
`FARM_learning`. We’re going to copy our `README` file over to this
folder using `rsync`. To do this, we’ll start in our home directory on
our computer. Now you’ll run `rsync -avze 'ssh -p 2022'
farm_username@farm.cse.ucdavis.edu:/home/farm_username/testing/README.md
~/FARM_learning`. This will copy the file from the listed destination to
the new location. If you were to leave the `README.md` portion off of
the source, `rsync` would copy the entire `testing` folder, and its
contents, into the destination folder. This is super useful if you want
to transfer a whole bunch of files at once, or move one of your nice and
neat self-contained project folders.

<script id="asciicast-T0VbuxhVqIgHZ5GB3Ht5mZJo6" src="https://asciinema.org/a/T0VbuxhVqIgHZ5GB3Ht5mZJo6.js" async></script>

You can also move things from your computer to the FARM, using the same
`rsync -avze 'ssh -p 2022' source_path destination_path` pattern, and
you can move whole folders too. Let’s `cd` into our `FARM_learning`
directory on our own computer, then make a folder called `dummy_files`.
Now `cd` into this folder. We’re going to make some blank files by using
the `touch` command, which will create an empty file with a given name.
Let’s run `touch dummy1 dummy2 dummy3` to create 3 dummy files. Run `ls`
to verify that they’re in your folder. Now let’s use `cd ..` to go back
up to our `FARM_learning` directory. Now we’re going to move the whole
`dummy_files` folder over to the FARM. Let’s run `rsync -avze 'ssh
-p 2022' dummy_files
farm_username@farm.cse.ucdavis.edu:/home/farm_username/testing`. Now
let’s go over to our FARM Terminal window, and `cd` into our `testing`
folder, then run `ls` to make sure our `dummy_files` folder is there.
Then `cd` into that folder and check to make sure your dummy files are
in there. You can use this same basic pattern to move data or other
files onto the FARM with relatively little work.

## `.R` and `.sh` Paired Scripts

Now that we know how to move stuff around, we’re ready to start making
actual scripts on the FARM. I’ve found that the best way to keep my
scripts nice and neat is to make a pair of scripts for every analysis I
run: a script that actually runs the analysis, and a script that
controls how the job is submitted to the FARM. One good reason for this
is that you can test your actual analysis scripts on your own computer,
then when they’re working, send them up to the FARM, where they can be
submitted with your submission script. For this example, we’ll be
writing a basic R script and a submission script. They aren’t really
going to be doing anything special, but they’ll have the basic structure
of some scripts you might actually want to run. Let’s start with the
`.R` script. Rather than explain it bit by bit in text, I’ll just show
you the script including some comments.

In the Terminal window open to the FARM, go into the `testing` directory
and type `nano test.R` to start editing a new R script. Type a script
that matches the following.

``` r
# create object called data that contains the first 5 columns of mtcars
data <- mtcars[1:5]

# print out our data
data

# compute the average mpg of the cars in the dataset
avg_mpg <- mean(data$mpg)

# save the avg_mpg object as a .rds file, in the directory where this script is running
saveRDS(avg_mpg, "avg_mpg_mtcars.rds")
```

There we go\! This script does a couple super simple things, using R’s
built-in mtcars dataset. Two things to notice: 1) running `data` would
normally print out the whole data.frame to the R console, but it will go
elsewhere when running the script on the cluster (we’ll cover this
later), and 2) we’ll be saving a `.rds` file, which is what you’ll often
do when you end up with a model object, like running `model1 <- lm(data,
y ~ x)`.

This R script should run on your computer just fine, and that makes it
really convenient to write a big ole script on your computer, test it
out a bit, then send it up to the cluster. This will be more typical,
compared to what we did, which was create a `.R` script directly on the
cluster. The next script, however, is going to be specific to the
cluster. I’m going to take a minute to talk about SLURM.

### SLURM

Slurm is a piece of software that controls how jobs get allocated across
the cluster. The cluster is made up of a bunch of different computing
“nodes”, which are where all the actual computation gets done.
Shocking as it may sound, you’re not the only one using the cluster\!
Tons of different researchers are using the cluster all the time, for
jobs with highly varying levels of complexity and computational need.
Trying to decide where to put all the jobs may sound like a tall task,
but SLURM manages it easily\! Essentially, everybody tells SLURM how
long they think their job will take, how much computational power it
will need, and what “priority” it is (I’ll explain priority later). Some
researchers, who have bought into the cluster, have access to more
powerful computing nodes, but if you’re reading this, you’ll probably be
using the more basic computing nodes.

In order to tell SLURM what our job requires, we’re going to make a
submission script. This script will be a `.sh` file, which is a `bash`
script. As you might recall, `bash` is the language you use in the
Terminal, running commands like `ls` and `cd`. Well, we can write
scripts that run a bunch of `bash`, just like we can write scripts that
run a bunch of R. Our submission script is essentially a bunch of
instructions to the cluster on how to run our R script. Because the
cluster is organized in a more complex way than your computer, and
because it’s shared by a bunch of other researchers, you’ve gotta give
some pretty specific instructions. I will by no means exhaustively list
all the things you can do with a cluster, especially stuff to do with
parallelization. All our submission script is gonna do is cover the
basics you’ll need to submit a job. I’ll show you the whole script, and
then we’ll walk through it line by line. To create this script, make
sure you’re in the `testing` directory, then use `nano test.sh` to
create our submission script. I try to make sure my `.R` and `.sh`
scripts have the same name other than the file extension, so it’s clear
that they’re a pair.

``` bash
#!/bin/bash -l

# setting name of job
#SBATCH --job-name=testR

# setting home directory
#SBATCH -D /home/mjculsha/testing

# setting standard error output
#SBATCH -e /home/mjculsha/testing/slurm_log/sterror_%j.txt

# setting standard output
#SBATCH -o /home/mjculsha/testing/slurm_log/stdoutput_%j.txt

# setting medium priority
#SBATCH -p med

# setting the max time
#SBATCH -t 0:10:00

# mail alerts at beginning and end of job
#SBATCH --mail-type=BEGIN
#SBATCH --mail-type=END

# send mail here
#SBATCH --mail-user=mjculshawmaurer@ucdavis.edu

# now we'll print out the contents of the R script to the standard output file
cat test.R
echo "ok now for the actual standard output"

# now running the actual script!

# load R
module load R

srun Rscript test.R
```

We’re gonna start from the top here. The **very first line** needs to be
`#!/bin/bash`. This is a little thing called a shebang, and it
essentially tells the computer what program it should use to run the
rest of the script. In this case, we’re telling the computer we want to
use `bash`.

The next thing you might notice is a ton of lines start with `#SBATCH`.
All of these lines are directions for SLURM. Each of these lines also
has a comment above it, which just starts with `#`. The first couple of
lines are setting the name and home directory for the job. The name will
help you identify your job later when you look at a list of all the jobs
running on the cluster, and it’s especially handy if you have multiple
jobs running at once. The home directory is going to be the directory
that contains everything needed for this particular script, which in our
case is the `test` directory.

Next up, we’re deciding where to put the standard output and standard
error logs. These files will be simple text files that contain all of
the output or error messages from your scripts, which will be invaluable
when something goes wrong. You’ll notice that I created a directory
under `testing` called `slurm_log`, where these will be stored. You can
go ahead and create this directory using `mkdir`. Another thing to note
is that the `%j` in each of these text file names will get turned into
the unique job ID number that gets created when you submit a job. What’s
nice about this is that you can submit the same script multiple times,
maybe tweaking it between runs, and each run will get a unique pair of
output and error files. This makes it much easier to fix problems with
scripts.

The next two directions are telling SLURM some stuff about how to
allocate resources for your job. A job’s priority affects how your job
may be kicked off a set of resources. Low priority jobs can be killed in
favor of higher priority jobs, medium priority jobs can be suspended but
will resume afterwards, and high priority jobs will keep the allocated
resources until the run finishes. You may now be asking yourself why
anybody would ever choose a lower priority job\! Well, resources can be
allocated more quickly for lower priority jobs, whereas you may have to
wait a while for exclusive access for a high priority job. I pretty much
run everything on medium priority, which works fine for every R script
I’ve ever run. We also have to tell SLURM the maximum time it will
take for your job to run. Asking for more time will also delay the start
of your job, so you should only ask for as much as you need. Honestly,
for me, this is primarily a wild guess followed by refinement based on
previous runs.

Next up, we’re setting up what I think is a pretty slick feature: email
alerts. You can have an email sent to you when your job starts and
finishes, which is really handy when you have jobs that take a few hours
to a few days. I pair this with a little Gmail filter that sends all the
emails with “slurm” in them to a specific folder. The “job finished”
email will get sent whether your job finishes nicely or something goes
wrong, so it can be useful to see that your job finished in 2 minutes
when you expected it to take 2 days.

The next line isn’t totally necessary, but I think it can be really
helpful. We’re using the `cat` command to print out the entire contents
of our `test.R` script, which will show up in our standard output file.
This can be handy in matching up the standard output of a particular job
with the exact R script used to run it. Maybe you submit the same R
script a couple times, making small changes in between each run. This
way, when you go look at the standard output file for a particular job,
you’ll know exactly what R code was used for that job. If something went
wrong with that job, you’ll have that job’s exact R code to help you
pinpoint the problem.

Finally, we actually get to run the R script\! First, we load up R so
the cluster knows we want to use it. Then we use `srun`, which is a
SLURM command, followed by `Rscript` and the filepath of our script,
**relative to our home directory** that we set earlier.

That’s a lot of stuff to submit a job, but the nice thing about this
script is that you can pretty much copy it for other jobs, with the
necessary modifications to file paths, run times, etc.

## `squeue`

One last stop before we actually submit our test job\! There are a few
SLURM commands you can use to take a look at information about the
cluster and jobs running on it. I’m going to introduce you to the most
useful one, which you’ll use to check on the status of your job. The
command `squeue` will give a list of all the jobs currently on the
cluster. You’ll see columns for Job ID, partition (aka priority), name,
user, ST for status, time (since the job started), nodes, CPU, minimum
memory, and nodelist. Some of these bits of information are more useful
than others. The primary things you’ll care about are the job ID, name,
user, status, and time. Status could use a bit more explanation: the
three states you’ll most likely encounter are PD for pending, R for
running, and S for suspended. Pending means the job hasn’t started yet,
R means it’s currently running, and S means it’s been temporarily
suspended so a higher priority job can run.

Since running `squeue` alone gives you **all** of the jobs on the
cluster, which is more information than you’ll often need, you can
narrow the list down to just your jobs by using `squeue -u
your_username`. In my case, that would be `squeue -u mjculsha`. I’m not
sure about you, but that is quite a bit to type every time you want to
check on your jobs. I’ll show you a little trick that can actually be
handy in a lot of circumstances. `bash` allows you to create shortcuts
called “aliases”, which basically just run some code when you type some
shorter bit of code. We’ll store these aliases in a file called
`.bash_profile`, which will reside in your home directory, which should
be your username. Make sure you’re in your home directory, which means
your prompt looks something like `mjculsha@farm`. Now run `nano
.bash_profile`, which will open a blank file with that name.

Your `.bash_profile` can be used to set all sorts of things, but we’ll
stick to aliases for now. The syntax to set an alias looks like this:
`alias short_thing="much longer thing"`. In this case, let’s type the
following line into `.bash_profile`: `alias sq="squeue -u username"`,
but using your username. This will create a shortcut so that any time
you type `sq` into the Terminal, it will interpret that as `squeue -u
username`. Since we want to check on our jobs pretty often, this will
save us a ton of typing\! Press `CTRL-X` to exit `nano` and save the
file. You might have to close your Terminal window, open a new one, and
get back onto the FARM in order for the `.bash_profile` to kick in. Now
typing `sq` should show you all of **your** jobs\!

## Submitting Jobs with `sbatch`

Finally, the moment you’ve all been waiting for… it’s time to submit a
job. First thing to do is navigate to the working directory you set in
the SLURM submission script. In our case, that’s the `testing`
directory. Once we’re in here, submitting a job is quite simple.
Remember that we’ve got a paired set of `test` scripts here, one with a
`.R` extension and the other with `.sh`. We submit a job using the `.sh`
script, which then runs the `.R` script. We’ve done a lot of work ahead
of time, so all we have to do now is run the line `sbatch test.sh`. This
will submit the `.sh` script to the cluster and our job should start
soon\!

<script id="asciicast-9vVoWipwLl5PxxqVk64LjjHTk" src="https://asciinema.org/a/9vVoWipwLl5PxxqVk64LjjHTk.js" async></script>

You can run `sq` to check all the jobs under your username. You may see
your job with the PD status, but you also might not see any job listed
at all\! Don’t fret too much- the script we’re running is extremely
simple, so it will take a really tiny amount of time to actually run. If
your job happens to be pending for a little bit, you might see it listed
with PD status, but if it doesn’t have to wait to run, it’ll get done so
quick that you don’t get to see it with the R status\! You can go ahead
and check your email to see if you got the start and finish notification
emails, otherwise we’ll check for the standard output and standard error
files.

### `scancel`

Before I forget, there’s an easy way to cancel a job if you need to.
Maybe you notice an error in your R script, but you’ve already submitted
the job. All you’ve gotta do is use the command `scancel`. You can use
`scancel jobid` to cancel a particular job (remember, you can now use
`sq` to see all your current jobs), or `scancel -u username` to cancel
all of your jobs.

## Checking `stdout` and `sterror`

Remember how we created a `slurm_log` directory to store the standard
output and error files created with each job? Now we’re going to go
check on those files to see how our job went. From the `testing`
directory, you can `cd` into the `slurm_log` directory. Now try running
`ls -lt`, which will list all the files in the directory, sorted by time
they were last modified. This is nice, because the top files will be the
ones generated by your most recent job, which is probably the one you’re
most interested in checking on. Since you’ve only submitted one job so
far, you should only have one standard output and one standard error
file. Run `cat filename` to print out the contents of that file so you
can check them out. For the standard output file, you should see your R
code followed by the `mtcars` dataframe, and then some other information
about the job. You can also run `cat filename | less`, which will pipe
the output of `cat filename` to `less`, which displays long files in a
format similar to `man`, where you can scroll easily.

<script id="asciicast-erRUTXsgYOPmuXNCY5an5fhVi" src="https://asciinema.org/a/erRUTXsgYOPmuXNCY5an5fhVi.js" async></script>

Our job should have run without any problems, so there shouldn’t be much
in the standard error file, but be aware that this is where any errors
will show up, whether they were generated by R or SLURM. Learning to
parse out these error files may take some time and plenty of Google, but
they’re an important part of learning to debug your work.

## `rsync` Results Back

If you look back at the R script that we ran, you’ll notice that we
saved a single R object as a `.rds` file. In our case, the object is
simply the average MPG from the mtcars dataset, but this could also be a
model object created by R’s `lm` function or something similar. This is
how I tend to do my work with Bayesian models that take a long time to
fit- I fit them on the cluster, save the model object as a `.rds` file,
bring that file back to my personal computer, load it into R, and then I
can play around with it **exactly** like it was fit on my own computer.
I’ve found this gives me the best combination of flexibility and
performance.

You’ll notice a step in there that involves bringing the results file
from the cluster to your personal computer. It’s time to bring back our
old friend `rsync`\! Taking a look at our R script, you’ll notice that
the path for our results is “avg\_mpg\_mtcars.rds”. That will be in
relation to our job’s **home directory**, which should be set as your
FARM home directory. That means we’ll find the `.rds` file in the home
directory. Just for future notice, I would strongly recommend creating a
`results` or `fit_models` directory under your main project directory,
so you can keep all your results in one spot.

Remember, you always use `rsync` in a Terminal that’s accessing **your**
computer, not the cluster. Recall that the syntax puts source first,
then destination. From **your** computer’s home directory, you can run
the following: `rsync -avze 'ssh -p 2022'
farm_username@farm.cse.ucdavis.edu:/home/farm_username/testing/avg_mpg_mtcars.rds
~/FARM_learning`, which will copy the results file into the
`FARM_learning` directory on your computer. If you feel like it, you can
open up an R session on your computer, and use
`readRDS("FARM_learning/avg_mpg_mtcars.rds")` to load that object into
R. This object will be just the same as if you ran the script on your
own computer instead of on the FARM.

# You Did It\! One More Thing Though

At this point, you’ve accomplished the basic goal of this lesson: run an
R script on the cluster, save your results, and bring them back to your
computer so you can work with them. However, if you’re running more
complicated R scripts, it’s almost guaranteed you will need to install
some packages. Rather than mess with installing packages to the cluster,
we’re just going to make a folder where we’ll install all our own
packages so things stay nice and neat. I will note that if you do want
an R package installed globally just email a request for it to
[help@cse.ucdavis.edu](help@cse.ucdavis.edu).

## `srun` Interactive R Session

So far, we’ve used R in a way you might not be used to. We made a
script, then submitted it, but we never actually opened R or used an R
console. On your computer, perhaps you use RStudio to interact with the
R console and get your work done. Well, we can’t use RStudio on the
cluster, but we sure can open an R console. On your own computer, if you
type `r` into the Terminal and hit ENTER, you will see an R console
appear. You can run R commands here just like you would in the RStudio
console. You can do something similar on the cluster.

When you log on to the FARM, you can run commands like `ls` and `cd` to
look around your FARM home directory, you can create directories and
edit files, stuff like that. When you’re doing these basic commands upon
logging in, you’re actually using one of the cluster’s nodes, called the
**head node**. The head node is shared by everyone, and it’s crucial
that everyone is able to use it when they need to. If you were to do
something more intense on the head node, such as running R, you would
tie up those resources and others wouldn’t be able to use the head node,
which would be a huge bummer. That’s why, up until now, we’ve only
**submitted** our R scripts, so that SLURM can take care of them and
send them to computing nodes.

Well, what if we want to do an interactive R session? On our computer,
we just ran R from the Terminal, but we don’t want to do this on the
FARM’s head node. Instead, we want to request a computing node to run
this interactive session. To do this, we’ll run the following line:
`srun -t 10 --pty R`. The `srun --pty` part will request an interactive
job, the `R` part says we want to use R, and the `-t 10` says we want
the job to only run for 10 minutes. If we ask for more time, it will
likely take longer for our interactive job to start, but if we don’t
request enough time, the job might end in the middle of something
important, like installing a package. Let’s try running an interactive
job now, but don’t install any packages yet. You can try running some R
commands just to see how it works. Let’s also run `sessionInfo()`, which
will give us some information about our current R session. Check the
output for the **R version**, which in my case is 3.6.1 (this may vary
depending on how far in the future you’re reading this). Remember what
the version is, it’ll be important in our next step.

<script id="asciicast-7jFHcRDOnBOLW7PhyfC1ATJ5V" src="https://asciinema.org/a/7jFHcRDOnBOLW7PhyfC1ATJ5V.js" async></script>

*Quick note*: I used `-p high` to use `high` priority for this run
because a few of the `med` priority nodes were down, and the interactive
sessions will default to `med` priority.

## Set Up Directory for R Packages

The FARM is set up so that you can use all sorts of software, like R, as
well as lots of R packages. For the sake of simplicity, we’re going to
install our R packages to a folder in our home directory. I actually do
this on my own computer too, since I’ve found it’s nice to have a little
more control over managing my packages. I actually have [a blog post on
this topic](https://mcmaurer.github.io/package-management/) if that
sounds interesting to you.

For now, all we’re going to do is make a directory in our home directory
using `mkdir`, which we’ll call `R_Packages`. `cd` into this directory,
and then make another one called `R3.6.1` or whatever your R version is.
When R packages are installed, they’re built with your specific version
of R, and if you try to use them when R gets updated, they won’t work
correctly. It’s just another bit of organization to keep them in their
own folders, but I find it useful.

## Install R Packages to Directory

Alright, now we’re going to make sure we’re in our FARM home directory
and then run another interactive R session using `srun -t 15 --pty R`.
Once your job starts and you see an R prompt `>` appear, go ahead and
run the code `rownames(installed.packages())`. This will get you a list
of all the installed packages that R can currently access. These are all
of the packages installed elsewhere on the cluster, which any user can
access. One handy trick to see if a package you want is already
installed is to use the following code: `"package_of_interest" %in%
rownames(installed.packages())`. If the output is `TRUE`, then it’s
installed, if `FALSE` then it’s not.

Just as an example, let’s install the `wesanderson` package to the
directory we created earlier. First thing, run `getwd()` in your R
session, just to verify that the working directory is your FARM home
directory. Next, let’s run the following line:
`install.packages("wesanderson", lib = "R_Packages/R3.6.1")`. This will
use the familiar `install.packages()` function, but we’ve added an
argument for `lib`, which will install the package into the directory we
made earlier.

<script id="asciicast-m6seqJZGOpLh2dqNH7lzaVY5E" src="https://asciinema.org/a/m6seqJZGOpLh2dqNH7lzaVY5E.js" async></script>

## Load Packages from Here

Now, any time you want to use that package in a script, you can run
`library("wesanderson", lib.loc = "R_Packages/R3.6.1")`, which will load
the package from the specified location. For a little bit more on how to
manage packages, which may save you some time if you’re going to be
installing a lot of your own packages, check out [my other blog post on
this topic](https://mcmaurer.github.io/package-management/).

<script id="asciicast-HnM15xUbTscDxGyRoAiyhJ275" src="https://asciinema.org/a/HnM15xUbTscDxGyRoAiyhJ275.js" async></script>

# Ok, NOW You Did It\!

I know I just threw a ton of information at you, and you probably have a
million questions. I have by no means given you a comprehensive overview
of the FARM, using the command line, or high performance computing. I
haven’t even given you a comprehensive look at running R scripts on the
FARM. But hopefully you have a better understanding of the **basic
workflow** of getting an R script running on the FARM. When you start
translating this process to your own work, you’re going to run into
problems, it’s just part of the gig. I still constantly run into
problems while doing work on the cluster, but I feel more equipped to
fix them than when I first started. My hope is that this lesson made the
initial process a little less daunting, and at least gave you an idea of
how to proceed.

There are lots of great resources that helped me figure this stuff out,
and will hopefully help you too. The FARM itself has [a Home
Page](https://wiki.cse.ucdavis.edu/support/systems/farm), a [Getting
Started](https://wiki.cse.ucdavis.edu/support/faq/getting_started)
section, and [a Farm Guide](https://wiki.cse.ucdavis.edu/farm_guide)
that can be super useful. The [High Performance Computing
website](https://www.hpc.ucdavis.edu/) has broader information about
UCD’s resources, including the FARM. Finally, the [Ross-Ibarra
Lab](http://www.rilab.org/) at UCD has a [really great bit of FARM
documentation](https://github.com/RILAB/lab-docs/wiki/Using-Farm) as
well. You should also feel free to contact me using [any method at the
bottom of this page](https://mcmaurer.github.io/About_Me/). Happy
FARMing\!
