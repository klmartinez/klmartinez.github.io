---
toc: true 
toc_sticky: true
excerpt: "A little guide to managing your packages between versions of R." 
---

Do you want to make it easier to deal with packages in R? Have you ever
upgraded R only to find all your packages have gone missing? Have you
ever needed to use an old version of R to run a specific package, only
to find that it’s a gigantic pain to deal with package versions? **There
has to be a better way\!**

Well, **there is\!** You can just put your packages in a specific place
and use some simple commands to re-install packages when you upgrade R.
You can even keep separate folders for different versions of R, in case
you need to use a package that doesn’t work with the most current
version.

## Track Your Packages

If you want to change where your packages are stored, you should
probably start by figuring out where they are right now. You can use the
`.libPaths()` function to find out where R is set to look for
    packages.

``` r
.libPaths()
```

    ## [1] "/Users/MJ/R_Packages_3.5"                                      
    ## [2] "/Library/Frameworks/R.framework/Versions/3.5/Resources/library"

As you can see, I’ve got two paths listed here. The first one is my
personally managed set of packages, which is what I hope to get you by
the end of this post\! The second one is R’s default package location,
which is probably where your packages currently live.

## Find a New Home

To change where your packages get stored, you have to modify your
`.Renviron` file, which is a file R uses to find environmental variables
when it starts up. You can have project-specific `.Renviron` files, as
well as versions in your `HOME` and `R_HOME` directories. R will only
use one when it starts up, and it’ll choose the most specific one. The
file in your project directory will override the one in your `HOME`
directory, and so on. For package management on my personal computer,
where I am the only user, I use a `.Renviron` file in my `HOME`
directory.

First, let’s check to see if such a file even exists\!

``` r
user_renviron = path.expand(file.path("~", ".Renviron"))
file.exists(user_renviron)
```

If it does, you’re in great shape. If not, just create an empty file in
your `HOME` directory called `.Renviron`. Either way, the next thing
you’ll have to do is create a folder where you want to keep your
packages. I put mine in my `HOME` directory and called it
`R_Packages_3.5`, for my current version of R. Since I’ve done this for
a few different R versions, I have a couple folders like this with `3.4`
and so on.

Now we’ll return to the `.Renviron` file and add the following line:
`R_LIBS_SITE=~/R_Packages_3.5`. Now, when R starts up, it’ll read your
`.Renviron` file and set the environmental variable `R_LIBS_SITE`. Once
this is all done, you can restart your R session and run
`Sys.getenv("R_LIBS_SITE")` to verify. While you’re at it, you can run
`.libPaths()` again. You should now see two paths: your newly created
folder, followed by R’s default location. The order here matters, as R
will look for packages in each of them sequentially. Packages will also
install to the first location listed.

## Time for the Move

Now that you’ve got an empty folder where R is ready to install and look
for packages, it’s time to install all of your packages here. **Do not**
try to manually move files from one folder to another. Instead, you’ll
just get the names of all the packages you have installed in the default
location and install them in the new one.

Remember how `.libPaths()` now gives you two locations, the second of
which is R’s default? All the files in there are package names, so by
listing the files, you can easily get the names of all the packages into
a single vector.

``` r
packages <- list.files(.libPaths()[2])
head(packages)
```

    ## [1] "base"      "boot"      "class"     "cluster"   "codetools" "compiler"

The next thing to do is install all of those packages into your new
folder. This might take a while, so grab a snack or something.

``` r
install.packages(packages, .libPaths()[1])
```

Like many of you, I’ve got lots of packages I installed from Github
rather than CRAN, so `install.packages()` won’t find them. I wrote a
little function to find which packages didn’t make it to the new folder.

``` r
packages_compare <- function(old_path, new_path){
  old_packages <- list.files(old_path)
  new_packages <- list.files(new_path)
  
  in_both <- old_packages %in% new_packages
  not_installed_now <- old_packages[!in_both]
  return(not_installed_now)
}
```

Now you can find names of packages that appear in the old location but
not the new
one.

``` r
still_need <- packages_compare("/Users/MJ/R_Packages_3.4", "/Users/MJ/R_Packages_3.5")
```

Check the names of those packages, and see if they look like things you
installed from Github. If you’re like me, you may remember installing
them from Github, but you can’t remember the username. There’s a handy
little package that’ll search for package names on Github and install
them even if you don’t know whose they are:

``` r
library(githubinstall)
```

Now just run this function on the list of packages you still need:

``` r
githubinstall(still_need)
```

If there are multiple packages on Github that go by that name, it’ll
give you options of whose you want to install. This might jog your
memory, but if you still can’t remember, you can just go google it.

## What To Do When You Upgrade R Again

Now that you’ve got all your folders in a custom location, re-installing
packages when you’ve updated R should be really easy. You’ll follow the
same basic protocol described above: create a new folder to store
packages for the new R version, change `R_LIBS_SITE` in your `.Renviron`
file to match this new folder, and instead of the following code…

``` r
packages <- list.files(.libPaths()[2])
```

… you run something like this instead:

``` r
packages <- list.files("/Users/MJ/R_Packages_3.4")
```

Now you’ll be grabbing names of packages from your custom location for
the previous R version. Follow the rest of the directions above to
install your packages in your new R version package folder. As a bonus,
you’ll also have all your old packages in a nice neat location in case
you want to run that old version of R at some point. Speaking of that…

## Using Multiple R Versions

Sometimes you find a package that only works with an old version of R,
and you need to run that old version. It can be helpful to then keep
corresponding versions of packages in a specific folder, like
`R_Packages_R_3.1` or something like that. You can pair these specific
folders with [RSwitch](http://r.research.att.com/), an application from
the R folks that lets you easily and immediately switch between R
versions. One click in RSwitch, changing one name in your `.Renviron`,
and bam, you’ve got a quick and easy pairing of R and appropriate
packages. You could even establish separate project `.Renviron` folders
with the appropriate packages built under the correct R version.

## My Script For Moving Packages

``` r
# first, save current library path
old_folder <- .libPaths()[1]

# update R_LIBS_SITE with new folder 
user_renviron = path.expand(file.path("~", ".Renviron"))
file.edit(user_renviron)

# Restart R and then use libPaths to check new path
.libPaths()

# now, get the names of all the packages from the OLD folder
packages <- list.files(old_folder)

# reinstall all those packages in your new location (this will take a while)
install.packages(packages, .libPaths()[1])

# check for packages you don't have now, probably those from Github

packages_compare <- function(old_path, new_path){
  old_packages <- list.files(old_path)
  new_packages <- list.files(new_path)
  
  in_both <- old_packages %in% new_packages
  not_installed_now <- old_packages[!in_both]
  return(not_installed_now)
}

still_need <- packages_compare(old_folder, .libPaths()[1])
still_need

# find packages from Github and install them
githubinstall::githubinstall(still_need)

# one last check to see if there are any you didn't get
packages_compare(old_folder, .libPaths()[1])
```
