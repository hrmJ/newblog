---
title: "Getting readable pdfs from scans"
datePublished: Fri Feb 02 2018 14:33:25 GMT+0000 (Coordinated Universal Time)
cuid: cl2ilfmm302ovarnvgn9y0zjm
slug: getting-readable-pdfs-from-scans
cover: https://cdn.hashnode.com/res/hashnode/image/unsplash/Oaqk7qqNh_c/upload/v1651235231752/JIM1r4dyU.jpeg

---


Buying the Onyx boox N96ml reader (with 9 inch display and android os)
last spring quite literally revolutionized my reading of
ebooks. It's not so often nowadays that I face a situation
where I actually need to _scan_ a book in order to have it
as a digital version. Sometimes this still happens,
however -- especially with certain not--so--recent dissertations that are not available
in our university library. This was the case with
Knud Lambrecht's seminal _Information structure and sentence form: Topic, focus, and the mental representations of discourse referents_
(from 1996), which I acquired through an interlibrary loan and
only had a limited time to read. I had actually
scanned the book long ago, but ended up with just a raw
unedited pdf with two pages per sheet -- certainly
not ideal for e-ink displays.

So I finally got tired of reading the book on
a desktop computer and decided I could try
to improve the file a bit. In the past I
had been working a lot with a utility called [ScanTailor](http://scantailor.org/).
This is a tool that takes multi-color tif files as input and
outputs (in the ideal case) nice and clear black--and--white
tifs with nothing but the actual text left. ScanTailor tries to strip
away all noise such as illuminations and shadows. It automatically splits
pages, adjusts their orientation and, finally, tries to
"dewarp" and "despeckle" the
pages.

Here's my work flow with ScanTailor, if I'm starting with a multi-page pdf.

## 1. Convert the pdf to a single multi-page tif

As I noted above, ScanTailor doesn't accept pdfs as input, so I had to convert
my pdf to tif. Here's a trick I've learned with ghostscript:

```bash
gs -sDEVICE=tiff24nc -r300x300 -sOutputFile=my_new_tif.tif -- my_original_pdf.pdf`
```

## 2. Split the multi-page tif to individual files

I used to use ScanTailor's GUI version, which let's you tweak with individual pages
and see the results immediately. This was rather laborious, and you actually had to
go through every single page (in my current case it would have meant 396 pages).
It wasn't until recently that I actually discovered that ScanTailor
also has a command line version called scantailor-cli. The command-line version
cannot handle multi-page input, so I had to add an extra step to my work flow,
namely, converting the multi-page tif I just got from ghostscript to
multiple single-page tifs. This turned out to be harder than I first thought,
mainly because the tif was so large. I finally found this solution from StackOverflow
and modified it a bit.
It requires you to specify the number of pages in the tif.

```bash
mkdir split
END=<NUMBER_OF_PAGES>;
for ((i=1;i<=END;i++))
do
    echo $i
    convert my_new_tif.tif[$i] -scene 1 split/my_new_tif_$i.tif
done
```

## 3. Running scantailor

So I ended up having a folder named `split` full of individual tifs, which
is exactly what scantailor-cli needed. The command line version has a lot of
options you can tweak, but for me the default settings worked like a charm.

```bash
mkdir output
scantailor-cli split output
```

## 4. Modifying the file names

As a result of the previous command, I got a bunch of nicely formatted black
and white tif files in a folder called `output`. Before I could try to merge
these into a single file again, I had to do some renaming. Oh, by the way,
forgot to mention: all this is done on a linux machine (ubuntu 17.10).
I had to install the packages `libtiff-tools`, `pdftk` and `rename` for the next two
steps to work.

This is what I did to rename my tifs so that they could be merged in exactly
the right order (found the solution [from askubuntu](https://askubuntu.com/questions/473236/renaming-hundreds-of-files-at-once-for-proper-sorting)):

```bash
rename 's/\d+/sprintf("%05d", $&)/e' *.tif
```

## 5. Combining and converting

After the renaming, I just combined the tifs with `tiffcp`:

```bash
tiffcp *tif output.tif
```

The last step was to convert the tiff back to pdf:

```bash
tiff2pdf output.tif -o output.pdf
```

All this resulted in a nice 16MB (396 pages) pdf file that
was perfectly fine for reading with an e-ink device.
