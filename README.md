# Preparing your package for a CRAN submission 

An open and collaborative list of things you have to check before submitting your package to the CRAN. 

## Why this repo? 

This repo was born after recent exchanges on Twitter regarding the CRAN submission process, especially [this thread](https://twitter.com/yoniceedee/status/989281686142771203). 

The idea here is to collect ground rules that would help the CRAN doing there work more easily, by listing common (or uncommon) things that they ask maintainers to change in order to be CRAN-proof.    

CRAN submission is strict, the CRAN team is doing its work voluntarily, and there are more than 12K packages to maintain. 

We believe we can help them by giving some good practices about package development and CRAN submission, so that package authors can work on these issues before the CRAN team ask them to do so. Hence, we could save everyone time by preventing the CRAN team from sending you an email because there is "with R" in the title of your DESCRIPTION. Because, as said by [Peter Dalgaard](https://twitter.com/pdalgd/status/989944476398432257): 

> too many people do not realise that the CRAN maintainer group can be counted on one hand, even one finger at times. 


## Check your package

### Be sure that `devtools::check()` returns 0 0 0 

The first thing to do is to run `devtools::check()`, and to be sure that there are no error, no warning, no note.

If you're in RStudio, you can also click on Build > Check. 

> If ever you think these warnings or notes are not justified, leave a comment when submitting where you specify why you think this is not justified.

### Use `{rhub}`

The `{rhub}` package and API allow you to check your package on several platforms, and for CRAN with `rhub::check_for_cran()`.

More about `{rhub}`: <https://github.com/r-hub/rhub>

## What next? 

These two checks might not catch everything the CRAN team will catch, so here is a list of good practices, based on "after-submission exchanges" we had with the CRAN team:

### (Re)submission

#### Temporality

CRAN submission of a new version should not be done too frequently. One time every 30 days seems to be the general rule (unless you're resubmitting after a CRAN team member feedback).

#### Explain what you have changed

When resubmitting after a CRAN feedback, be sure to include that this is a resubmission after a feedback, and describe what you have done.

### About the DESCRIPTION file

#### Fill all the slots 

This might not be caught by the CRAN, but be sure you have filled everything in this file. 

#### Do not put 'in R' or 'with R' in the package title

It's tempting to write something like ['A friendlier condition handler for R'](https://github.com/ColinFay/attempt) or ['Easy Dockerfile creation for R'](https://github.com/ColinFay/dockerfiler) in the title of your repository on GitHub (and seems appropriate). The CRAN team will ask you to remove this, as it is redundant (you're only dealing with R packages on the CRAN).

#### Package title should be in title case

The title should be in title case

> [Capital and Lowercase Letters in Titles (Title Case)](http://www.grammar-monster.com/lessons/capital_letters_title_case.htm)

Note: this should be caught by r-hub.

#### Polish the description of your package

A CRAN-proof package should have a long description that explains what the package does and why it is "original" regarding what is already on the CRAN.

#### Do not put 'package' in the DESCRIPTION

Neither the title nor the description should start with "A package..." or with the name of the package.

### Documentation 

#### Use canonical and https url 

If there are some urls in your documentation, be sure to: 

+ use https
+ use the canonical form for CRAN package (i.e.: https://CRAN.R-project.org/package=***)

### Package structure

#### Use temporary files and folder if you write on the disk 

CRAN Repository Policy state : 

> Packages should not write in the user’s home filespace (including clipboards), nor anywhere else on the file system apart from the R session’s temporary directory (or during installation in the location pointed to by TMPDIR: and such usage should be cleaned up). Installing into the system’s R installation (e.g., scripts to its bin directory) is not allowed.

You might not know what temporary directories / files are or how to use them. These temporary files are created for the current R session, and they are deleted when the session is closed. 

You can create them with: 

```
file <- tempfile()
```

Add an extension with 

```
tmp <- tempfile(fileext = ".csv")
tmp
[1] "/var/folders/lz/thnnmbpd1rz0h1tmyzgg0mh00000gn/T//Rtmpnh8kAc/fileae1e28878432.csv"
```

So you can: 

```
write.csv(iris, file = tmp)
```

See: [Create Names for Temporary Files](https://stat.ethz.ch/R-manual/R-devel/library/base/html/tempfile.html)

### About revdep 

If packages depend on your package, you should run a reverse dependencies test with `devtools::revdep_check()`.

### Free resources to learn about package development : 

#### Books and blogposts

+ [R packages](http://r-pkgs.had.co.nz/) - by Hadley Wickham

+ [Writing R Extensions](https://cran.r-project.org/doc/manuals/r-release/R-exts.html) - Official Documentation

+ [Mastering Software Development in R](https://bookdown.org/rdpeng/RProgDA/) - by Roger D. Peng, Sean Kross, and Brooke Anderson.

+ [How to develop good R packages (for open science)](http://www.masalmon.eu/2017/12/11/goodrpackages/) - by Maëlle Salmon 
