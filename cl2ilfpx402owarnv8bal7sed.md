---
title: "Setting up a new bookdown project"
datePublished: Tue Nov 06 2018 07:41:26 GMT+0000 (Coordinated Universal Time)
cuid: cl2ilfpx402owarnv8bal7sed
slug: setting-up-a-new-bookdown-project
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1651234804202/sIF2AJtlc.png

---


I've been excited about writing my research in
[Rmarkdown](https://rmarkdown.rstudio.com/) for something like five years now,
and have written basically all my academic work with it. The biggest project
was my dissertation thesis, a monograph written entirely in Rmd.
When writing something that large, or, actually, anything you
might want to split into different sections, you should definitely
check out a recent development in the Rmarkdown universe, namely, the [Bookdown](https://bookdown.org/yihui/bookdown/)
package. In this post, I'll go through my basic setup
of a new bookdown project.

# 1. Setting up the project folder

I start with a fresh new folder with the following structure:

    bookdown/
    ├── 01-intro.Rmd
    ├── aux
    │   ├── preamble.tex
    │   ├── refs.bib
    │   └── utaltl.csl
    ├── _bookdown.yml
    └── _output.yml

Bookdown will try to get all the rmd files in the project folder
and turn them into a single document. To make sure you have the right
order of files in the final output, it's safest to prefix the files
with numbers.

The configuration of bookdown is done through the two [yaml](https://en.wikipedia.org/wiki/YAML)
files, `_bookdown.yml` and'`_output.yml` AND with a yaml block
at the beginning of the first file of the project (in this case, `01-intro.Rmd`).

## Basic configuration

In a project I recently started, this is what the yaml block
in `01-intro.Rmd` looks like:

```yaml
latexBackend: linguex
exampleRefFormat: "{}"
title: "Last year but not yesterday? Explaining differences in the locations of Finnish and Russian time adverbials using comparable corpora"
author: Juho Härme
bibliography: aux/refs.bib
link-citations: true
```

The options _latexBackend_ and _exampleRefFormat_ are specific
to my projects: they have to do with using linguistic examples
in the text, which I do with my own fork of a [pandoc filter](http://www.pandoc.org/filters.html)
called [pangloss](https://github.com/hrmJ/pangloss_linguex). These
lines should be left out if such a filter is not used.

Bibliography management and different outputs, output modification using cls
or bib(la)tex and all those kinds of factors definitely deserve
a separate post. For simplicity's sake, here I only
use the default settings and simply define the location of the `.bib`
file containing my references in the format of a bibtex database.

The file `_bookdown.yaml` is pretty simple, in this
project I only have one line:

```
delete_merged_file: true
```

## Output configuration

When writing Rmarkdown files, you can specify multiple output
formats in `_output.yml` and if you compile your
book with the [render_book function](https://bookdown.org/yihui/bookdown/new-session.html),
without additional arguments, all those output formats will be produced.

Here, I have specified two output formats, _html_ and _pdf_ (produced with
latex). The html is something I use when sketching and building the article / book.
At that stage I usually have the pdf parts commented out. When the work is getting
closer to be finished, I tend to switch to the pdf output.

Here's what my `_output.yml` looks like

```
bookdown::gitbook:
bookdown::pdf_book:
  toc: true
  toc_depth: 6
  includes:
    in_header: aux/preamble.tex
  latex_engine: xelatex
  keep_tex: true
  pandoc_args:
    - --filter
    - pangloss
    - --filter
    - pandoc_avm
```

## Auxiliary files

As can be seen in the folder structure above, I use a separate folder called _aux_ to store
my auxiliary files in. These include the bibliography database, possibly some additional pandoc
filters, cls files and the like. One especially important file is the `preamble.tex`, which
loads all the needed latex packages and adjusts the final document's layout.

# Compiling the book

I will probably write something about how to do this in Rstudio later on.
Here's how the compilation of the book happens in R terminal.

First, make sure to (install and) load the bookdown library

```r

library(bookdown)

```

Then, just run the `r render_book` function by specifying at least one file to be included
in the book:

```r

render_book("01-intro.Rmd")

```

I actually tend to have a separate `.Rprofile` file
which includes all the libraries that need to be loaded
etc. Place the file in the project's directory and
you'll have you workspace set up the way you want it.

After the compilation, you'll see that
the final output will end up in a folder called `_book`
which also includes a whole bunch
of auxiliary files, especially for the gitbook type of html format.
Here's the structure of my bookdown folder
after the compilation process:

    bookdown/
    ├── 01-intro.Rmd
    ├── aux
    │   ├── preamble.tex
    │   ├── refs.bib
    │   └── utaltl.csl
    ├── _book
    │   ├── a-real-section.html
    │   ├── introduction.html
    │   ├── libs
    │   │   ├── gitbook-2.6.7
    │   │   │   ├── css
    │   │   │   │   ├── fontawesome
    │   │   │   │   │   └── fontawesome-webfont.ttf
    │   │   │   │   ├── plugin-bookdown.css
    │   │   │   │   ├── plugin-fontsettings.css
    │   │   │   │   ├── plugin-highlight.css
    │   │   │   │   ├── plugin-search.css
    │   │   │   │   └── style.css
    │   │   │   └── js
    │   │   │       ├── app.min.js
    │   │   │       ├── jquery.highlight.js
    │   │   │       ├── lunr.js
    │   │   │       ├── plugin-bookdown.js
    │   │   │       ├── plugin-fontsettings.js
    │   │   │       ├── plugin-search.js
    │   │   │       └── plugin-sharing.js
    │   │   └── jquery-2.2.3
    │   │       └── jquery.min.js
    │   ├── _main.pdf
    │   ├── _main.tex
    │   └── search_index.json
    ├── _bookdown.yml
    └── _output.yml

The actual pdf output is the file `_book/_main.pdf`. For the html
output, just pick the name of the first html file, in this case
`introduction.html`.
