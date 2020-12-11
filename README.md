# Rmarkdown files for Orchestrating Single Cell Analyses

Goal: To use docker image bioconductor/orchestratingsinglecellanalysis as a starting point for my own scRNA-seq pipeline.
To this end I need to install/update my usual packages.

Updating/installing from RStudio (as default docker's user rstudio) does not work:
`BiocManager::install(remotes::local_package_deps(dependencies=TRUE))`
nlme: Installation path not writeable, unable to update packages: nlme
gert: failed to find libgit2.pc

"Bashing" as root into docker container (named osca) kid of works:
sudo docker exec -it osca bash
cd /home/rstudio/Documents/OrchestratingSingleCellAnalysis
R -e 'BiocManager::install(remotes::local_package_deps(dependencies=TRUE), Ncpus=8)'
R -e 'BiocManager::install(c(
"brms",
"cowplot",
"emmeans",
"flextable",
"ggridges",
"ggstance",
"modelr",
"officer",
"openxlsx",
"rstan",
"rvg",
"tidybayes",
"tidyverse",
"tidyxl",
"unpivotr"
), update=TRUE, ask=FALSE, Ncpus=8)'

Then in Rstudio (user rstudio) in terminal
git config --global user.name "DaniilSarkisyan"
git config --global user.email "MyMail"

## Overview

This repository contains the basic ingredients for the [Orchestrating Single Cell Analysis](https://osca.bioconductor.org) book.
Developers wanting to contribute scientific content to the book should make pull requests to this repository, 
the others with `-release` or `-devel` suffixes are just used to host the book content on GitHub Pages.
Our set-up provides a light code-only repository (this one) for day-to-day developer use,
which avoids the Git blob bloat from storing PNGs and HTMLs in the other repository.

## Structure

The book is split into four parts:

1. Introductory chapters focusing on how to install and use R and Bioconductor.
2. Topic chapters, the meat of the book where each chapter describes a different step of a scRNA-seq analysis.
3. Workflows containing end-to-end analysis Rmarkdown reports with minimal explanatory text.
4. Appendices containing some bits and pieces about the contributors.

Compilation of the workflows will cache the objects generated after each chunk.
This allows objects to be quickly re-used in the chapters without having to repeat or rewrite the prior steps.
The `extractCached()` calls littered in the chapters will extract objects of interest from each cache,
also reporting the steps used to generate those objects in a folded code chunk.
This enables readers of each chapter to inspect the code without interrupting the pedagogical flow.

As a consequence, compilation of many of the chapters depends on compilation of the workflows.
Those writing new chapters should move all set-up code into a similar workflow 
and exploit the `extractCached()` to obtain a starting point for their chapter.
Also note the `chapterPreamble()` code chunk that is required at the top of each chapter to set up the collapsible elements.

## Build instructions

Install all relevant packages in the `DESCRIPTION`:

```r
BiocManager::install(remotes::local_package_deps(dependencies=TRUE))
```

Then, run the usual **bookdown** invocation, for example:

```r
bookdown::render_book("index.Rmd", "bookdown::gitbook")
```

Advanced users can call `make` to perform a "pre-compilation" prior to the above command.
This generates cached content to be used by the serial **bookdown** invocation,
and is most useful when the `make` itself is parallelized.

```sh
make
rm -rf _bookdown_files
R -e 'bookdown::render_book("index.Rmd", "bookdown::gitbook")'
```

The docker image contains the book in `/home/book` where these commands can be executed.

## Developer instructions

To contribute reports, follow standard procedure: fork and PR.

- All `topic` chapters must start from a `SingleCellExperiment` object and should be independent of other `topic` chapters.
- All `workflow` chapters should use a `SingleCellExperiment` object throughout the various chunks.
This allows chapters to pick up the SCE at any point.

Any changes should be accompanied by house-keeping updates, performed automatically by the GitHub Action in this repository.

- Updating the listed dependencies in the `DESCRIPTION`.
Some `extra` help is required for those dependencies that are implicitly required via `Suggests`.

  ```r
  rebook::updateDependencies(extra=c("Rtsne", "RMTstat", "statmod", "GO.db"))
  ```

- Updating the `Makefile` for parallel builds.

  ```r
  createMakefile()
  ```

## Deployment instructions

1. Commit changes to this repository, usually on the `master` branch.
2. The [**OSCA**](https://github.com/LTLA/OSCA) package is a "trojan" that we will use to sneak the book onto Bioconductor's build system (BBS) owing to the latter's package-centric nature.
It contains a GitHub Action to poll for and incorporate any changes in this repository, after which it will bump the version number and push the changes to the Bioconductor Git servers.
3. The BBS does its stuff and compiles the book every Tuesday and Friday, deploying the HTMLs to a relevant Bioconductor-hosted location for viewing.


