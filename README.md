# r-packages


This repo provides code and instruction for creating an R package using `devtools`, `roxygen2`, `usethis` and `desc`.

As an illustrative example I show how to build API wrappers as packages. Allowing us to retrieve data from the web and make structured requests which specify which data we are interested in.

This has a tonne of applications within official statistics at Forest Research. For instance, filtering and aggregating trade data on forestry and wood data from HMRC's uktradeinfo platform.


Here are the packages you will need to follow along

```{r}

install.packages("pacman")

pacman::p_load(devtools, roxygen2, usethis, curl, httr, jsonlite, attempt, purrr, desc)


```

# Setting up the project

`devtools` provides tools to make package development easier by providing R functions and R-Studio add-ins that simplify and expedite common tasks and give structure to packages.

When creating a new project with R-Studio, click "Create a package with devtools". This will set up a new project with some pre-existing folders and files.

DESCRIPTION//NAMESPACE --> Set up metadata and organise package functions

R --> Write R code for your package

test --> Verify your code is correct

man/vignettes --> Document your code and write tutorials and how-tos

data --> Include datasets in your package


# devstuffs

Here is a script using `devtools`, `usethis` and `desc` to setup all project metadata.

I like to name this file devstuffs.R and will typically stick it in a folder called data-raw which I can create using `usethis::use_data_raw()`


```{r}

pacman::p_load(devtools, usethis, desc)

# Remove default DESC
unlink("DESCRIPTION")
# Create and clean desc
my_desc <- description$new("!new")

# Set your package name
my_desc$set("Package", "packagedev")

#Set your name
my_desc$set("Authors@R", "person('Dan', 'Braby', email = 'daniel.braby@forestresearch.gov.uk', role = c('cre', 'aut'))")

# Remove some author fields
my_desc$del("Maintainer")

# Set the version
my_desc$set_version("0.0.0.9000")

# The title of your package
my_desc$set(Title = "R Package Example")
# The description of your package
my_desc$set(Description = "An example of developing API wrappers as R packages.")
# The urls
my_desc$set("URL", "http://github.com/fr-dan/r-packages")
my_desc$set("BugReports", "http://github.com/fr-dan/r-packages/issues")
# Save everyting
my_desc$write(file = "DESCRIPTION")

# If you want to use the MIT licence, code of conduct, and lifecycle badge
use_mit_license(name = "Daniel BRABY")
use_code_of_conduct()
use_lifecycle_badge("Experimental")
use_news_md()

# Get the dependencies
use_package("httr")
use_package("jsonlite")
use_package("curl")
use_package("attempt")
use_package("purrr")

# Clean your description
use_tidy_description()

```







