---
title: "Annotating random samples in R"
datePublished: Wed May 30 2018 05:25:08 GMT+0000 (Coordinated Universal Time)
cuid: cl2il74g402obarnv0pau64ov
slug: annotating-random-samples-in-r
cover: https://cdn.hashnode.com/res/hashnode/image/unsplash/wCo9UwZEa18/upload/v1651235111574/KIl86gvYW.jpeg

---


Years ago, when I was still working with my master's thesis, I used to do
a lot of manual annotating of data in a spreadsheet software (libreoffice calc, mainly).
Now that R has firmly become my main tool for doing research, I've been thinking
about what's the best way to, for instance, manually annotate a small sample
of sentences from a data frame.

## Random sampling with dplyr

First of all, I must say that during the last couple of months I've grown
more and more accustomed to using [dplyr's](https://dplyr.tidyverse.org/) piping and data
manipulation techniques. They have absolutely revolutionized the way I write
R code nowadays. Here's one typical use case for me that has to do with
annotating samples.

Consider [this dataset]("/data/headverbs.csv") consisting of Finnish and Russian verbs:

```r
# A tibble: 543 x 2
# Groups:   lang [2]
   lang  headverb
   <chr> <chr>
 1 fi    saada
 2 fi    tehdä
 3 ru    провести
 4 ru    принять
 5 ru    получить
 6 fi    voittaa
 7 ru    опубликовать
 8 fi    aloittaa
 9 fi    antaa
10 fi    ottaa
# ... with 533 more rows

```

If my goal were to take a random sample of these verbs, _dplyr_ offers the
convenient function `sample_n`. So I can just do:

```r

> headverbs %>% sample_n(10)
# A tibble: 10 x 2
   lang  headverb
   <chr> <chr>
 1 ru    вынести
 2 ru    приговорить
 3 fi    ampua
 4 ru    пройти
 5 fi    nimittää
 6 ru    сдать
 7 ru    предложить
 8 ru    заказывать
 9 ru    запретить
10 fi    määrätä


```

Even better, using the `group_by` function, I can first group my data by language
and then get a sample having `n` number of instances from both Finnish and Russian:

```r


> headverbs %>% group_by(lang) %>% sample_n(10)
# A tibble: 20 x 2
# Groups:   lang [2]
   lang  headverb
   <chr> <chr>
 1 fi    käydä
 2 fi    avata
 3 fi    korottaa
 4 fi    pistää
 5 fi    osoittaa
 6 fi    uudistaa
 7 fi    korjata
 8 fi    kilpailuttaa
 9 fi    todistaa
10 fi    pelata
11 ru    обыграть
12 ru    проиграть
13 ru    представлять
14 ru    опубликовать
15 ru    решить
16 ru    комментировать
17 ru    свести
18 ru    испечь
19 ru    перенести
20 ru    взорвать

```

## Manual annotations

Now, in order to make manual annotations possible without leaving R I
wrote the following little function:

```r

CheckSample_df <- function(r, cols_to_show, backup_file="/tmp/backup.txt"){
    content  <- sapply(r[cols_to_show],function(x) paste(strwrap(x, 79),collapse="\n"))
    cat("\n\n", paste(cols_to_show,content,sep="\n=====\n",collapse="\n\n"),"\n\n")
    def <- readline("\nYour annotation:\n")
    write_lines(paste0(
                       paste(r[cols_to_show],collapse="|"),
                       "|",def)
                ,backup_file,append=T)
    return(def)
}

```

The function is designed to be called with `apply` (for a tutorial cf. e.g
[here](https://www.datacamp.com/community/tutorials/r-tutorial-apply-family)).
For the verb dataset above, if
I wanted to define an additional column describing, e.g., my interpretation of
the semantic class of each verb, I could do the following:

```r

headverbs$semantic_class <- apply(headverbs,1,CheckSample_df, cols_to_show=c("lang","headverb"))

```

The `cols_to_show` parameter defines, which columns are shown for the user to
help with the annotation. The `backup_file` specifies a file the function copies
the annotation results. This is a reasonable thing to do especially if you have a
lot to annotate -- in case of R crashing in the middle of the process,
it's nice to have something to use as a basis for data recovery.

If you're just interested in a simpler version that
you can use with `sapply` , the function could be written this way:

```r

CheckSample_simple <- function(show_this, backup_file="/tmp/backup.txt"){
    cat("\n\n", paste(strwrap(show_this, 80), collapse="\n"), "\n\n")
    def <- readline("Your annotation:")
    write_lines(paste0(show_this,"|",def),backup_file,append=T)
    return(def)
}

```

### Fine-tuning with pbapply

One improvement to the aforementioned technique is to get some feedback on how
you are progressing with the annotation process. A great tool for
this is the [pbapply](https://github.com/psolymos/pbapply) package. We can
just turn the previous command into:

```r

headverbs$semantic_class <- pbapply(headverbs,1,CheckSample_df, cols_to_show=c("lang","headverb"))

```

And we get a nice progress bar indicating the work that has already been done

- an estimate of the time remaining:

```r

   |+++++                                             | 9 % ~02m 22s

```
