## The setNames + (l)apply pattern


My research is, most of the time, about the differences and similarities
between two languages, Finnish and Russian. Most of the stuff I deal with
is quantitative by methodology and I tend to have large datasets in
the following format:

| lang&nbsp;&nbsp; | variable_A&nbsp;&nbsp; | variable_B&nbsp;&nbsp; | variable_C |
| ---------------- | ---------------------- | ---------------------- | ---------- |
| fi               | x                      | 12                     | category a |
| fi               | x                      | 10                     | category a |
| fi               | x                      | 32                     | category b |
| fi               | z                      | 51                     | category b |
| ru               | x                      | 12                     | category b |
| ru               | z                      | 2                      | category a |
| ru               | z                      | 88                     | category a |
| ru               | x                      | 12                     | category a |
| fi               | x                      | 32                     | category b |
| fi               | x                      | 32                     | category b |
| ru               | x                      | 12                     | category a |

&nbsp;

My usual way of getting some quick numbers out of the data for both of the
languages is to use lists rather than single variables. E.g.
instead of having:

```r

    number_of_xyz_FI <- nrow(subset(mydata,lang = 'fi' & variable_A == x & variable_B > 10))
    number_of_xyz_RU <- nrow(subset(mydata,lang = 'ru' & variable_A == x & variable_B > 10))

```

I prefer:

```r

    number_of_xyz <- list(
    "fi" <- nrow(subset(mydata,lang = 'fi' & variable_A == x & variable_B > 10))
    "ru" <- nrow(subset(mydata,lang = 'ru' & variable_A == x & variable_B > 10)))

```

Now I can reference the data in my inline text with `number_of_xyz$fi` and `number_of_xyz$ru`.
This makes the global namespace less messy and looks more logical, in my opinion, at least.
There is still one downside to all of this: from the perspective of reproducibility
of the code and in order not to break the [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)
principle, it would be good to have the members of the list set up inside a loop.
So I usually end up using `lapply`, acting on a vector of language codes:

```r

    number_of_xyz <- lapply(c("fi","ru"),function(thislang){
        nrow(subset(mydata,lang == thislang & variable_A == x & variable_B > 10))
    })

```

This still has the downside, that I have to manually set the names of the produced list, i.e.

```r

    names(number_of_xyz) <- c("fi","ru")

```

which is somewhat annoying. Of course -- in the spirit of DRY, once again -- it's better to have
the language codes set up as a separate variable beforehand, so that the whole thing becomes

```r

langs <- c("fi","ru")

number_of_xyz <- lapply(langs,function(thislang){
    nrow(subset(mydata,lang == thislang & variable_A == x & variable_B > 10))
})

names(number_of_xyz) <- langs

```

Luckily, some time ago I found at StackOverflow a neater way to do this. The trick is
to use the `setNames` function:

```r

langs <- c("fi","ru")

number_of_xyz <- setNames(lapply(langs,function(thislang){
    nrow(subset(mydata,lang == thislang & variable_A == x & variable_B > 10))
},langs)

```

...and this whole pattern has become a real habit for me, so that every time I do a comparison,
I almost immediately write `setNames(lang,function(thislang),langs)`. A bit strange, though, that
it seems like this actually is the most straight-forward of naming the list resulting from
lapply.
