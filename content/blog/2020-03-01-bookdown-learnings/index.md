---
slug: "bookdown-learnings"
title: 10 bookdown Things we Learned by Working on the Blog Guidance
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




Blablabla blog guidance blablabla. Technically, we structured the content as a bookdown gitbook which [Karthik Ram](/authors/karthik-ram/) judiciously suggested us. It was Stef's first foray into the glorious process of publishing a book with bookdown, and Maëlle's second one[^1].

### Stef (first-timer)


### Maëlle (more experienced bookdowner)

#### 1. How to start a bookdown project

I've started both my bookdown projects using [Sean Kross' excellent primer](https://github.com/seankross/bookdown-start),
but whilst looking for a reference to show Stef[^2],
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

#### 3. How to define functions and chunk options for all chapters

I found how to write code that'll be loaded before each chapter rendering thanks to [Christophe Dervieux's answer on an old RStudio community thread](https://community.rstudio.com/t/bookdown-caching/43652/2).

One simply needs to create an [R script called `_common.R` at the root of the bookdown project](https://raw.githubusercontent.com/ropensci-org/blog-guidance/master/_common.R). 
Here's ours below. 
It contains chunk options, `magrittr` loading, and two helper functions for rendering templates.

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

Our bookdown project uses `DESCRIPTION` to track dependencies, I suppose I could use the package infrastructure more and define the helper functions as functions, but the approach above is pleasant too.

#### 4. How to have alternative text but no captions for figures

In R Markdown, the same chunk options `fig.cap` controls the caption and alternative text of images.
We wanted alternative text but no caption.
The header option `figure_caption` didn't work.
I opened [an issue in bookdown repo](https://github.com/rstudio/bookdown/issues/856) after not getting solutions [on an RStudio community post](https://community.rstudio.com/t/have-an-alternative-text-but-no-caption-or-a-different-caption-in-bookdown-git-book/50370).

So if bookdown doesn't really support having alternative text but no captions for figures, what did we do? Thanks to a good tip by [Romain Lesur](https://github.com/RLesur), I wrote CSS to remove all captions.

```css
.caption {
  display: none;
}
```

The lines above live with other styling stuff in a file called `style.css`, that we refer to in [the `_output.yml` config file](https://github.com/ropensci-org/blog-guidance/blob/cae5887f392b4137eec50467779e9bba973b0b69/_output.yml#L3).

#### 5. How to deploy a preview of the book for PRs

I felt quite strongly about having some sort of CI/CD for the book. We achieved that using GitHub Actions, a new CI service by GitHub. If you're curious about it, I'd recommend watching [Jim Hester's talk from the RStudio conference earlier this year](https://resources.rstudio.com/rstudio-conf-2020/azure-pipelines-and-github-actions-jim-hester). Our GitHub Actions workflows make good use of [Jim Hester's actions and examples](https://github.com/r-lib/actions).

Here's what we now have

* after each commit to the master branch, the book is built and deploy to the gh-pages branch that points to the blog guide official URL. Code below, [log example](https://github.com/ropensci-org/blog-guidance/runs/471978792?check_suite_focus=true);

<details closed>
<summary> <span title='Click to Expand'> master workflow </span> </summary>

```Markdown

on:
  push:
    branches:
      master
  
name: Render-Book-from-master

jobs:
  bookdown:
    name: Render-Book
    runs-on: macOS-latest
    steps:
      - uses: actions/checkout@v1
      - uses: r-lib/actions/setup-r@v1
      - uses: r-lib/actions/setup-pandoc@v1
      - name: Query dependencies
        run:
          Rscript -e "install.packages('remotes')" -e "saveRDS(remotes::dev_package_deps(dependencies = TRUE), 'depends.Rds', version = 2)"

      - name: Cache R packages
        uses: actions/cache@v1
        with:
          path: ${{ env.R_LIBS_USER }}
          key: ${{ runner.os }}-r-${{ hashFiles('depends.Rds') }}
          restore-keys: ${{ runner.os }}-r-

      - name: Install dependencies
        run:
          Rscript -e "library(remotes)" -e "deps <- readRDS('depends.Rds')" -e "deps[['installed']] <- vapply(deps[['package']], remotes:::local_sha, character(1))" -e "update(deps)"

      - name: Render Book
        run: Rscript -e 'bookdown::render_book("index.Rmd")'
      - name: Commit results
        if: github.repository == 'ropensci-org/blog-guidance'
        run: |
          cp ghpagescname docs/CNAME
          cp -r favicon/ docs/
          cd docs
          git init
          git add .
          git commit -m 'update book'
          git push https://${{github.actor}}:${{secrets.GITHUB_TOKEN}}@github.com/${{github.repository}}.git HEAD:gh-pages --force

```

</details>
<br>

* after each commit in a PR from a fork, the book is built so we'd notice something breaking the Rmd. [log example](https://github.com/ropensci-org/blog-guidance/runs/466995315?check_suite_focus=true);

* after each commit in a PR from the repo, the book is built and deployed to a Netlify preview whose URL is posted in the PR checks. [Log example](https://github.com/ropensci-org/blog-guidance/runs/471968808?check_suite_focus=true), [Direct link to the check with the preview link](https://github.com/ropensci-org/blog-guidance/runs/471974005?check_suite_focus=true).

<details closed>
<summary> <span title='Click to Expand'> PR workflow </span> </summary>

```Markdown

on: pull_request
  
name: PR-workflow

jobs:
  bookdown:
    name: Render Book
    runs-on: macOS-latest
    steps:
      - name: Is this a fork
        run: |
          fork=$(jq --raw-output .pull_request.head.repo.fork "${GITHUB_EVENT_PATH}");echo "::set-env name=fork::$fork"
            
      
      - uses: actions/checkout@v1
        
      - uses: r-lib/actions/setup-r@v1
      
      - uses: r-lib/actions/setup-pandoc@v1
      
      - name: Query dependencies
        run:
          Rscript -e "install.packages('remotes')" -e "saveRDS(remotes::dev_package_deps(dependencies = TRUE), 'depends.Rds', version = 2)"

      - name: Cache R packages
        uses: actions/cache@v1
        with:
          path: ${{ env.R_LIBS_USER }}
          key: ${{ runner.os }}-r-${{ hashFiles('depends.Rds') }}
          restore-keys: ${{ runner.os }}-r-

      - name: Install dependencies
        run:
          Rscript -e "library(remotes)" -e "deps <- readRDS('depends.Rds')" -e "deps[['installed']] <- vapply(deps[['package']], remotes:::local_sha, character(1))" -e "update(deps)"

      - name: Render Book
        run: Rscript -e 'bookdown::render_book("index.Rmd")'
        
      - uses: actions/setup-node@v1
        if: env.fork == 'false'
        with:
          node-version: "12.x"
          
      - name: Install Netlify CLI
        if: env.fork == 'false'
        run: npm install netlify-cli -g
        
      - name: Deploy to Netlify (test)
        if: env.fork == 'false'
        run: DEPLOY_URL=$(netlify deploy --site ${{ secrets.NETLIFY_SITE_ID }} --auth ${{ secrets.NETLIFY_AUTH_TOKEN }} --dir=docs --json | jq '.deploy_url' --raw-output);echo "::set-env name=DEPLOY_URL::$DEPLOY_URL"
                
      - name: Create check
        if: env.fork == 'false'
        run: |
          curl --request POST \
          --url https://api.github.com/repos/${{ github.repository }}/check-runs \
          --header 'authorization: Bearer ${{ secrets.GITHUB_TOKEN }}' \
          --header 'Accept: application/vnd.github.antiope-preview+json' \
          --header 'content-type: application/json' \
          --data '{
            "name": "Preview Book",
            "head_sha": "${{ github.event.pull_request.head.sha }}",
            "conclusion": "success",
            "output": {
                "title": "Preview link",
                "summary": "[Preview link](${{ env.DEPLOY_URL }}) :rocket:"
            }
            }'

```

</details>
<br>

Highlights from the workflow above

* To deploy to Netlify _and_ get the preview URL, the workflow doesn't use Netlify's GitHub Actions but instead installs Netlify CI, extracts the URL using `jq`[^3] and set it as an environment variable that can be used by next steps. I got the idea from [a thread on Netlify forum](https://community.netlify.com/t/deploying-preview-web-hook-via-api/3320/2).

```yaml
run: DEPLOY_URL=$(netlify deploy --site ${{ secrets.NETLIFY_SITE_ID }} --auth ${{ secrets.NETLIFY_AUTH_TOKEN }} --dir=docs --json | jq '.deploy_url' --raw-output);echo "::set-env name=DEPLOY_URL::$DEPLOY_URL"
```

* To post the preview URL in the PR, the workflow uses [a GitHub check run](https://developer.github.com/v3/checks/runs/). You can only create and patch them as a GitHub app. Luckily when in a GitHub Actions runner the default token has these permissions! My only regret is that trying to post the preview URL as a "details" URl for that check didn't work (GitHub reset it to the check page on GitHub Actions) so one needs to [click once more](https://github.com/ropensci-org/blog-guidance/runs/471974005?check_suite_focus=true). Using say Travis CI instead of GitHub Actions, I think I'd have needed to implement my own GitHub app.

After the successful deployment of a preview, in the PR checks one check called "Preview book" appears.

<!--html_preserve--> {{< figure src = "PR-checks.png" width = "600" alt = "Screenshots of PR checks in a PR to the blog guide" >}}<!--/html_preserve-->

If clicking on "Details" one get to [a check page where the preview link is prominent](https://github.com/ropensci-org/blog-guidance/runs/471974005?check_suite_focus=true). 

<!--html_preserve--> {{< figure src = "preview-check.png" width = "600" alt = "Screenshots of a GitHub Action check page featuring a Netlify preview link" >}}<!--/html_preserve-->


* To find out whether the PR is from a fork (and to skip all deploy steps for that, since forks don't have access to Netlify secrets), I use `jq` on the raw info about the build, idea I got [from Vanessa Sochat](https://github.com/r-universe/vsoch/blob/2bf10b1e30f59f5fab64ec2b7332526c47f1f4d3/.github/workflows/pull-request-update-packages.yml#L26).

If you're feeling motivated to add GitHub Actions deployment to your bookdown project, on top of Jim's video mentioned earlier and our workflows, be sure to note that Emil Hvitfeld wrote an excellent guide to deploying your book on Netlify with GitHub Actions](https://www.hvitfeldt.me/blog/bookdown-netlify-github-actions/), with screenshots!

### Conclusion

In this post, we shared useful bookdown things we learnt whilst working on rOpenSci blog guidance. 
We're quite happy with both the end product and our learnings! 
If _you_ want to get started with bookdown, we'd recommend [the bookdown book](https://bookdown.org/yihui/bookdown/introduction.html), [this introductory slidedeck by Alison Hill](https://alison.rbind.io/talk/2019-rsc-bookdown/) and [the bookdown gallery](https://bookdown.org/home/archive/).


[^1]: The first bookdown book Maëlle worked on is [rOpenSci dev guide](https://devguide.ropensci.org/)!
[^2]: [A tweet about using the config file for ordering chapters rather than numbering their filenames](https://twitter.com/hadleywickham/status/1137317951428747270)
[^3]: Maëlle discovered `jq` in [a blog post by Carl Boettiger](https://www.carlboettiger.info/2017/12/11/data-rectangling-with-jq/) and reported on her use of `jq` with rOpenSci `jqr` R package [in a blog post about getting data about Software Peer Review](/blog/2018/04/26/rectangling-onboarding/).
