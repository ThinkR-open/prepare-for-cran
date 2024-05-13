# Preparing your package for a CRAN submission 

An open and collaborative list of things you have to check before submitting your package to the CRAN. 

## Why this repo? 

This repo was born after exchanges on Twitter regarding the CRAN submission process, especially [this thread](https://twitter.com/yoniceedee/status/989281686142771203). 

The idea here is to collect ground rules that would help the CRAN doing there work more easily, by listing common (or uncommon) things that they ask maintainers to change in order to be CRAN-proof.    

CRAN submission is strict, the CRAN team is doing its work voluntarily, and there are more than 15K packages to maintain. 

We believe we can help them by giving some good practices about package development and CRAN submission, so that package authors can work on these issues before the CRAN team ask them to do so. Hence, we could save everyone time by preventing the CRAN team from sending you an email because there is "with R" in the title of your DESCRIPTION. Because, as said by [Peter Dalgaard](https://twitter.com/pdalgd/status/989944476398432257): 

> too many people do not realise that the CRAN maintainer group can be counted on one hand, even one finger at times. 

## TL;DR

There are still steps to add to list automated tests as detailed in the following sections, but you can add it to a "dev/dev_history.R" file to run them each time you send something to CRAN.

```r
# Prepare for CRAN ----

# Update dependencies in DESCRIPTION
# install.packages('attachment', repos = 'https://thinkr-open.r-universe.dev')
attachment::att_amend_desc()

# Check package coverage
covr::package_coverage()
covr::report()

# Run tests
devtools::test()
testthat::test_dir("tests/testthat/")

# Run examples 
devtools::run_examples()

# autotest::autotest_package(test = TRUE)

# Check package as CRAN using the correct CRAN repo
withr::with_options(list(repos = c(CRAN = "https://cloud.r-project.org/")),
                     {callr::default_repos()
                         rcmdcheck::rcmdcheck(args = c("--no-manual", "--as-cran")) })
# devtools::check(args = c("--no-manual", "--as-cran"))

# Check content
# install.packages('checkhelper', repos = 'https://thinkr-open.r-universe.dev')
# All functions must have either `@noRd` or an `@export`.
checkhelper::find_missing_tags()

# Check that you let the house clean after the check, examples and tests
# If you used parallel testing, you may need to avoid it for the next check with `Config/testthat/parallel: false` in DESCRIPTION
all_files_remaining <- checkhelper::check_clean_userspace()
all_files_remaining
# If needed, set back parallel testing with `Config/testthat/parallel: true` in DESCRIPTION

# Check spelling - No typo
# usethis::use_spell_check()
spelling::spell_check_package()

# Check URL are correct
# install.packages('urlchecker', repos = 'https://r-lib.r-universe.dev')
urlchecker::url_check()
urlchecker::url_update()

# check on other distributions
# _rhub v2
rhub::rhub_setup() # Commit, push, merge
rhub::rhub_doctor()
rhub::rhub_platforms()
rhub::rhub_check() # launch manually


# _win devel CRAN
devtools::check_win_devel()
# _win release CRAN
devtools::check_win_release()
# _macos CRAN
# Need to follow the URL proposed to see the results
devtools::check_mac_release()

# Check reverse dependencies
# remotes::install_github("r-lib/revdepcheck")
usethis::use_git_ignore("revdep/")
usethis::use_build_ignore("revdep/")

devtools::revdep()
library(revdepcheck)
# In another session because Rstudio interactive change your config:
id <- rstudioapi::terminalExecute("Rscript -e 'revdepcheck::revdep_check(num_workers = 4)'")
rstudioapi::terminalKill(id)
# if [Exit Code] is not 0, there is a problem !
# to see the problem: execute the command in a new terminal manually.

# See outputs now available in revdep/
revdep_details(revdep = "pkg")
revdep_summary()                 # table of results by package
revdep_report()
# Clean up when on CRAN
revdep_reset()

# Update NEWS
# Bump version manually and add list of changes

# Add comments for CRAN
usethis::use_cran_comments(open = rlang::is_interactive())

# Upgrade version number
usethis::use_version(which = c("patch", "minor", "major", "dev")[1])

# Verify you're ready for release, and release
devtools::release()
```

## Package Infrastructure Check

### Be sure that `devtools::check()` returns 0 0 0 

The first thing to do is to run `devtools::check()`, and to be sure that there are no error, no warning, no note.

If you're in RStudio, you can also click on Build > Check. 

> If ever you think these warnings or notes are not justified, leave a comment when submitting where you specify why you think this is not justified.

### Use a spellcheck

You can call `usethis::use_spell_check()` inside your package to add a test for spelling. 
Call `spelling::spell_check_package()` at any time if you need to run the spellcheck.

### Use `{rhub}`

The `{rhub}` package allows to check your package on several platforms with the CRAN default configuration, using GitHub Actions.  
Run `rhub::rhub_setup()` and follow instructions.  

More about `{rhub}`: <https://github.com/r-hub/rhub>

### Check for windows

Test that your package builds using the [win-builder](https://win-builder.r-project.org/) tool, or with `devtools::check_win_devel()`.

### Check for MacOS - CRAN M1 build machine

Build and manually submit your archive here: https://mac.r-project.org/macbuilder/submit.html

### Explore potential solutions to errors returned by CRAN or win-builder

See https://github.com/DavisVaughan/extrachecks

## Submission

These checks might not catch everything the CRAN team will catch, so here is a list of good practices:

### (Re)submission

Note that if this is your first submission, you will automatically have a NOTE, for `New submission`.

#### Temporality

CRAN submission of a new version should not be done too frequently. One time every 30 days seems to be the general rule (unless you're resubmitting after a CRAN team member feedback).

#### Explain what you have changed

When resubmitting after a CRAN feedback, be sure to include that this is a resubmission after a feedback, and describe what you have done.

#### Bump your version number

If you resubmit after a CRAN feedback, add 1 to the patch component of your version number (e.g, if your first submission is 0.3.1, your resubmission should be 0.3.2).

## General Comments

### Spelling

CRAN can reject packages that have grammar errors on the `DESCRIPTION`. 
Some common spelling errors: 
  - Other packages should be between `'` (example: `Lorem-Ipsum Helper Function for 'shiny' Prototyping`)
  - Acronyms should be capitalized (example: `API`, not `Api`)

## About the DESCRIPTION file

### You're the copyright holder

`DESCRIPTION` file should have a `cph` (copyright holder). 
It can either be the first author, or the company where the author(s) work(s). 

### Fill all the slots 

This might not be caught by the CRAN, but be sure you have filled everything in this file. 

### Do not put 'in R' or 'with R' in the package title

It's tempting to write something like ['A friendlier condition handler for R'](https://github.com/ColinFay/attempt) or ['Easy Dockerfile creation for R'](https://github.com/ColinFay/dockerfiler) in the title of your repository on GitHub (and seems appropriate). 
The CRAN team will ask you to remove this, as it is redundant (you're only dealing with R packages on the CRAN).

### Description should be a paragraph

Write an elaborated Description  field. 

### Package title should be in title case

The title should be in title case

> [Capital and Lowercase Letters in Titles (Title Case)](http://www.grammar-monster.com/lessons/capital_letters_title_case.htm)

Note: this should be caught by r-hub.

### Polish the description of your package

A CRAN-proof package should have a long description that explains what the package does,  what are the benefits, what is new and how does it differ from what is already on the CRAN.

### Do not put "This package", package name, title or similar in the Description

Neither the title nor the description should start with "A package..." or with the name of the package.

### Use quote and upper case if needed

If you quote another package, an article/book or a website/API, put its name between single quote `'`. Also, package names are case sensitive. e.g. 'shiny' --> 'Shiny'.

### Use parentheses after function names

If a function name is used in your DESCRIPTION, make sure to follow it with parentheses.  e.g., "provides a drop-in replacement for cat() from the 'base' package."

### Cite a reference for implementation of a method or linking to a service.

If you write an R interface to an API or implement an algorithm from a published article/book, add a reference to the publication as a DOI, ISBN, or similar canonical link, or URL for the API or article in the 'Description' field of your DESCRIPTION file.

### Use angle brackets for auto linking

When linking to an article or website in the DESCRIPTION, use angle brackets to auto-link.

```
API name <http:...> or <https:...>
authors (year) <DOI:...> (see <https://en.wikipedia.org/wiki/Digital_object_identifier> )
authors (year) <arXiv:...>
authors (year, ISBN:...)
```
with no space after `https:`, `doi:`, `arXiv:` and angle brackets for auto-linking. 

## Documentation 

### All exported functions should have a return value

All the exported functions in your package should have a `@return` value. 
If a function does not return a value, document that too.

If there is an internal function (not exported) with partial documentation (title, and.or `@param`), use the tag `#' @noRd` to avoid generating documentation.  
You can use `checkhelper::find_missing_tags()` to help you find the missing tags in your documentation. Install {checkhelper} from GitHub: https://github.com/ThinkR-open/checkhelper

### About `\dontrun{}` 

`\dontrun{}` elements in the examples might in fact be run by CRAN. 
If you don't want an example to be run, wrap it between `if (interactive()) {}`. Do not wrap example between `if (FALSE) {}`.

`\dontrun{}` should only be used if the example really cannot be executed
(e.g. because of missing additional software, missing API keys, ...) by
the user. That's why wrapping examples in `\dontrun{}` adds the comment
("# Not run:") as a warning for the user.

Unwrap the examples if they are executable in < 5 sec, or replace
`\dontrun{}` with `\donttest{}`.  
Note that `\donttest{}` will be run by `check()` and may be run by CRAN...

### Use canonical and https URL 

If there are some URLs in your documentation, be sure to: 

+ use https
+ use the canonical form for CRAN package (i.e.: https://CRAN.R-project.org/package=***)
+ use canonical form for Bioconductor package (i.e.: https://bioconductor.org/packages/***)

You can use {urlchecker} to help: https://github.com/r-lib/urlchecker

### Do not use relative paths for links to files (e.g. `README.md`, `NEWS.md`, `LICENSE.md`)

If you have relative URIs pointing to files like `NEWS.md` or `CODE_OF_CONDUCT.md` from within the `README.md` for instance:

```{markdown}
Code is distributed under the [GPL-3.0-License](LICENSE.md).
```

They will throw the following CRAN error:

```
Found the following (possibly) invalid file URIs:
     URI: LICENSE.md
       From: README.md
```

Changing the relative links to absolute links pointing to the {pkgdown} website does the trick (see the Code of conduct in `{dplyr}` README) 

```{markdown}
Code is distributed under the [GPL-3.0-License](https://USERNAME.github.io/MY_PACKAGE/LICENSE.html).
```

Pointing to an external resource, for a license, also works (see the `{golem}` README):

```{markdown}
Code is distributed under the [GPL-3.0-License](https://www.gnu.org/licenses/gpl-3.0.en.html).
```


### Long running examples

If you have examples that take more than a few seconds each to run, wrap them in `\donttest{}`, don't use `dontrun{}`.

```
#' @example
#' \donttest{x <- foo(y)}
```

### Fill all the values in documentation

There should be no empty tags in the documentation (for the one requiring a value).
`devtools::check()` detects empty `@param` and `@return` outputs.  
Again, you can use `checkhelper::find_missing_tags()` to help you find the missing tags in your documentation. Install {checkhelper} from GitHub: https://github.com/ThinkR-open/checkhelper

### About HTML5

If you got this problem on CRAN
```
Warning: <img> attribute "align" not allowed for HTML5 
```

You can follow these steps: 

https://github.com/DavisVaughan/extrachecks-html5

## Package structure

### Use temporary files and folder if you write on the disk 

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

## About revdep 

If packages depend on your package, you should run a reverse dependencies test on packages listed with `devtools::revdep()`.  
Use {revdepcheck}: https://github.com/r-lib/revdepcheck

```r
# Check reverse dependencies
# remotes::install_github("r-lib/revdepcheck")
usethis::use_git_ignore("revdep/")
usethis::use_build_ignore("revdep/")

devtools::revdep()
library(revdepcheck)
# In another session
id <- rstudioapi::terminalExecute("Rscript -e 'revdepcheck::revdep_check(num_workers = 4)'")
rstudioapi::terminalKill(id)
# See outputs
revdep_details(revdep = "pkg")
revdep_summary()                 # table of results by package
revdep_report() # in revdep/
# Clean up when on CRAN
revdep_reset()
```

## What to do once your package is ready? 

#### CRAN submission comments

Creates cran-comments.md, a template for your communications with CRAN when submitting a package. The goal is to clearly communicate the steps you have taken to check your package on a wide range of operating systems. If you are submitting an update to a package that is used by other packages, you also need to summarize the results of your reverse dependency checks.

`usethis::use_cran_comments(open = rlang::is_interactive())`

#### Using devtools::release()

You can run `devtools::release()` to automatically send to CRAN from R.

#### Confirm by following the link 

You'll receive a link in your mailbox. Click on this link to confirm the upload. 

### Wait for the release

Depending on the package, it might take between one hour and several weeks, if it needs manual inspection, it can take some time. 

You can watch the status of your package with `{cransays}`: https://lockedata.github.io/cransays/articles/dashboard.html

## Resources to learn about package development : 

### Books and blogposts

+ [Checklist for CRAN submissions](https://cran.r-project.org/web/packages/submission_checklist.html)

+ [CRAN Repository Policy](https://cran.r-project.org/web/packages/policies.html)

+ [R packages](http://r-pkgs.org) - by Hadley Wickham & Jennifer Bryan

+ [Writing R Extensions](https://cran.r-project.org/doc/manuals/r-release/R-exts.html) - Official Documentation

+ [Mastering Software Development in R](https://bookdown.org/rdpeng/RProgDA/) - by Roger D. Peng, Sean Kross, and Brooke Anderson.

+ [How to develop good R packages (for open science)](http://www.masalmon.eu/2017/12/11/goodrpackages/) - by Maëlle Salmon (including list of tutorials)

+ [Sinew: Simple R Package Documentation](https://metrumresearchgroup.github.io/sinew/) - by Jonathan Sidi

+ [rOpenSci Packages: Development, Maintenance, and Peer Review](https://ropensci.github.io/dev_guide/) - by rOpenSci onboarding editors
