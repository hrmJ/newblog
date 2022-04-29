## Annotating random samples part 2


Just a quick follow-up on the previous post about manually annotating random
samples. Yesterday I was facing a situation where I had to go and find some
broader contexts for concordances fetched from a corpus of Finnish newspaper
texts. These corpora don't allow access for more than a sentence, maximum
a paragraph at a time, and I needed to look at whole texts. Luckily, I figured
out that the city library has access to the electronic archives of many of the
newspapers included in the corpus.

So, I had a sample of 60 concordances I needed to get the full text for. The
access to the full texts was provided only through the computers physically
located in the library -- this meant that I, sadly, didn't have the
possibility of working with just R all the time. The easiest solution I came up with
was to quickly upload my data set to a Google sheet and then open that sheet
in the browser of the PC at the library. Fortunately, there is the nice
[googlesheets](https://github.com/jennybc/googlesheets) library that makes this easy.
All I had to do, was:

```r

devtools::install_github("jennybc/googlesheets")
library(googlesheets)

```

Then I converted an existing data frame (well, a tibble, actually), to a Google sheet by just:

```r

samp_gsheet <- gs_new("samp_press_fi",input=samp_press_fi)

```

...where `samp_press_fi` was the name of my tibble including the samples
I wanted to get the contexts for. The command first takes you to Google
authorization page, after which you're good to go. The nice thing about
googlesheets is, that after I got the contexts I needed, the data was automatically
updated in R by just, for instance:

```r

> samp_gsheet %>% gs_read
Accessing worksheet titled 'Sheet1'.
Downloading: 8.2 kB     Parsed with column specification:
cols(
  sent = col_character(),
  sourcetext = col_character(),
  location3 = col_character(),
  group = col_character(),
  context = col_character()
)

```
