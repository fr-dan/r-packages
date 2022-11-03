# r-packages


This repo provides code and instruction for creating an R package using `devtools`, `roxygen2`, `usethis` and `desc`.

As an illustrative example I show how to build API wrappers as packages. Allowing us to retrieve data from the web and make structured requests which specify which data we are interested in.

This has a tonne of applications within official statistics at Forest Research. For instance, filtering and aggregating trade data on forestry and wood data from HMRC's uktradeinfo platform.


Here are the packages you will need to follow along

```{r}

install.packages("pacman")

pacman::p_load(devtools, roxygen2, usethis, curl, httr, janitor, jsonlite, attempt, purrr, desc)


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
use_package("janitor")

# Clean your description
use_tidy_description()

```

# Utils

Before creating the main calling functrions, we'll create some ultilitary functions that will run some tests. Firstly, whether the internet connection is running and secondly whether the the page returns the right http code

For this we'll create a file called utils.R, saved into the R/ folder:

```{r}

#' @importFrom attempt stop_if_not
#' @importFrom curl has_internet
check_internet <- function(){
  stop_if_not(.x = has_internet(), msg = "Please check your internet connexion")
}

#' @importFrom httr status_code
check_status <- function(res){
  stop_if_not(.x = status_code(res), 
              .p = ~ .x == 200,
              msg = "The API returned an error")
}


```

@importFrom allows us to specify functions from pre-existing packages we are using and later allows us to ensure that the dependency is established.

In this utils.R file we can also create objects, which we can reference in other functions. 

For this API, I am targetting the Historic Hansard record, which provides different parliamentary records prior to their updating in 2005

```{r}

base_url <- "https://api.parliament.uk/historic-hansard/"

members <- "all-members/"

commons <- "commons/"

js <- ".js"

```

# Creating functions

Create a new R script and write the functions used to call the API

With the Historic Hansard API, which may be one of the worst things ever made by a human, the URL path convention goes

`base_url` + `what (members/commons)` + `Mon` + `.js`

```{r}
#' Download all members on date from HHR
#' 
#' @param day number of day in month
#' @param month first 3 char of month
#'
#' @importFrom attempt stop_if_any
#' @importFrom jsonlite fromJSON
#' @export
#' @rdname hansard_members
#'
#' @return the results from the search
#' @examples 
#' \dontrun{
#' hansard_members(day = 5, month = "apr")
#' }

search_ban <- function(q = NULL, limit = NULL, autocomplete = NULL, lat = NULL, lon = NULL, type = NULL, postcode = NULL, citycode = NULL){
  args <- list(q = q, limit = limit, autocomplete = autocomplete, lat = lat, 
               lon = lon, type = type, postcode = postcode, citycode = citycode)
  # Check that at least one argument is not null
  stop_if_all(args, is.null, "You need to specify at least one argument")
  # Chek for internet
  check_internet()
  # Create the 
  res <- GET(base_url, query = compact(args))
  # Check the result
  check_status(res)
  # Get the content and return it as a data.frame
  fromJSON(rawToChar(res$content))$features
}

hansard_members <- function(day = NULL, month - NULL) {
args <- list(day = day, month = month)
stop_if_any(args, is.null, "Both day and month must be specified")
check_internet()
res <- GET(paste0(base_url, members, month, "/", day, js))
res <- clean_names(res)
}

```

We can do the same for any of the other APIs on Historic Hansard, optionally we could create another parameter and give them as options.

# Roxygen

`roxygen2::roxygenise()` does a bunch of stuff under the hood that I've never cared to look into. What it does do however, is use the metadata we have written into our script when creating our R functions, the imports, parameters and descriptions- and uses these to automatically write documentation. Meaning we do not need to write .Rd files by hand

```{r}

roxygen2::roxygenise()

```


# Build Package

Using `devtools` we can `check` and `build` our package

Check--> updates the documentation, then builds and checks the documentation locally. Main goal-- are there functions in the R scripts whose packages are not linked in the description

Build --> Builds a package file from package sources. You can use it to build a binary version of your package (might be the tarball thing- not too sure)

```{r}
devtools::check()

devtools::build()


```

# All that remains

- Write the Readme

- Write a vignette

- Write further tests (for another week)








