---
slug: "bookdown-learnings"
title: 10 bookdown Things we Learned by Working on the Blog Guidance
# Delete the package_version line below if your post is not about a package
package_version: 0.1.0
authors:
  - Stefanie Butland
  - Maëlle Salmon
# Set the date below to the publication date of your post
date: 2020-03-01
# Set categories to technotes if this is a tech note
categories: blog
# Leave topicid blank below; will be set by editor
topicid:
# Minimal tags for a post about a community-contributed package 
# that has passed software peer review are listed below
# Consult the Technical Guidelines for information on choosing tags
tags:
  - community
  - bookdown
# the summary below will be used by e.g. Twitter cards
description: "10 bookdown tips for newbies and experts"
# If you have no preferred image for Twitter cards,
# delete the twitterImg line below 
# Note there is no '/' symbol before 'img' here
# if needed replace blog with technotes
#twitterImg: blog/2019/06/04/post-template/name-of-image.png
# 'output' is necessary to obtain index.md
# Do not commit index.html
output: 
  html_document:
    keep_md: true
---




Blablabla blog guidance blablabla. Technically, we structured the content as a bookdown gitbook which [Karthik Ram](/authors/karthik-ram/) judiciously suggested us. It was Stef's first foray into the glorious process of publishing a book with bookdown, and Maëlle's second one^1.

### Stef (first-timer)


### Maëlle (more experienced bookdowner)

#### 1. How to start a bookdown project

I've started both my bookdown projects using [Sean Kross' excellent primer](https://github.com/seankross/bookdown-start),
but whilst looking for a reference to show Stef^2,
I saw a tweet of Alison Hill's about starting a bookdown project from RStudio which looks handy. 
[Slide of Alison's showing how to start a bookdown from RStudio](https://arm.rbind.io/slides/bookdown.html#11).

#### 2. How to get the copy-paste button for code chunks

In the blog guidance, if you hover around the top-right corner of [e.g. the Markdown post template](https://blogguide.ropensci.org/templatemd.html) you get a copy-paste button. 
For this to work, the chunk needs to have some language information i.e.

````
```
code
```
````

will not get a copy-paste button, but 

````
```yaml
code
```
````

will! 
I'm glad I know that now. 
Chunks with language info are prettier anyway since they get adapted code highlighting.

##### 3. How to define functions and chunk options for all chapters

I found how to write code that'll be loaded before each chapter rendering thanks to [Christophe Dervieux's answer on an old RStudio community thread](https://community.rstudio.com/t/bookdown-caching/43652/2).

One simply needs to create an [R script called `_common.R` at the root of the bookdown project](https://raw.githubusercontent.com/ropensci-org/blog-guidance/master/_common.R). Here's ours below.


```r
details::details(readLines("https://raw.githubusercontent.com/ropensci-org/blog-guidance/master/_common.R"), summary = "our _common.R")
```

```
Warning in readLines("https://raw.githubusercontent.com/ropensci-org/
blog-guidance/master/_common.R"): incomplete final line found on 'https://
raw.githubusercontent.com/ropensci-org/blog-guidance/master/_common.R'
```

<details closed>
<summary> <span title='Click to Expand'> our _common.R </span> </summary>

```r

knitr::opts_chunk$set(
  cache = TRUE,
  echo = FALSE
)

library("magrittr")

show_template <- function(filename, 
                          lang = "markdown",
                          details = FALSE,
                          yaml_only = FALSE,
                          ...) {
  lines <- suppressWarnings(
    readLines(
      file.path("templates", filename)
    )
  ) 
  
  if (yaml_only) {
    lines <- bookdown:::fetch_yaml(lines)
  }
  
  lines %>%
    glue::glue_collapse(sep = "\n") -> template

  if (details) {
    toshow <- details::details(template, summary = filename,
                     lang = lang,
                     ...)
  } else {
    toshow <- glue::glue("````{lang}\n{template}\n````")
  }

  return(toshow)

}

show_checklist <- function(filenames) {
  filenames <- file.path("checklists", filenames)
  purrr::map(filenames, 
             readLines) %>% 
    unlist() %>%
    gluedown::md_task() %>%
    glue::glue_collapse("\n") -> x
  
  glue::glue("````markdown\n{x}\n````") %>% 
    knitr::asis_output()
}

```

</details>
<br>

### Conclusion

In this post, we shared useful bookdown things we learnt whilst working on rOpenSci blog guidance. 
We're quite happy with both the end product and our learnings! 
If _you_ want to get started with bookdown, we'd recommend [the bookdown book](https://bookdown.org/yihui/bookdown/introduction.html), [this introductory slidedeck by Alison Hill](https://alison.rbind.io/talk/2019-rsc-bookdown/) and [the bookdown gallery](https://bookdown.org/home/archive/).


[^1]: The first bookdown book Maëlle worked on is [rOpenSci dev guide](https://devguide.ropensci.org/)!
[^2]: [A tweet about using the config file for ordering chapters rather than numbering their filenames](https://twitter.com/hadleywickham/status/1137317951428747270)
