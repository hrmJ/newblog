---
title: "Summer's coming! Deno told me so..."
datePublished: Mon Jan 23 2023 16:50:30 GMT+0000 (Coordinated Universal Time)
cuid: cld91oeyh00020amkafq457m6
slug: summers-coming-deno-told-me-so
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/w9WVNJYb25k/upload/6e49f85562b35af061497ebade33f4db.jpeg
tags: r, deno

---

I just figured out a great use case for Deno: writing small cli scripts. So far I haven't dived into Deno and perhaps the reason has been waiting for a complicated project like a real web app. It happened, however, that the need arose for me to write a couple of small helper scripts and whereas in the past I might have reached for python or just plain old bash, I decided to try out Deno. After all, it is mainly typescript I spend most of my days writing and Deno promises to deliver built-in typescript support. Another plus I hadn't realised was, however, the ease with which it allowed me to pull in any extra libraries needed. Check this out!

## Project 1: decoding a jwt token

There must be dozens of tools for this. But I just wanted something quick that worked on the command line. How do I quickly decode a jwt token and read the information it actually contains? Using these lines:

```typescript
import { decode } from "https://deno.land/x/djwt@v2.8/mod.ts";

const jwt = Deno.args[0];
const payload = await decode(jwt);

console.log(payload);
```

The beauty here is the fact that the module needed for decoding is imported by just writing the URL on line 1 without the need to have something installed before the script is run (just think about running global pip installs or creating virtual envs for python scripts!).

The script can be run by just `deno run inspect_token.ts TOKEN`

## Project 2: querying a rest API

The older I get, the harder it seems to become for me to deal with the fact that our Nordic winter it's so cold and dark. I wanted to have something concrete that would give me hope about the fact that each new day is a step towards summer and an increase in the amount of daylight! My kids, too, tend to get a bit depressed about the fact that it's dark in the morning when we go to daycare and it's dark in the afternoon when we come home. Luckily for us, https://sunrise-sunset.org/api offers a rest API that lets you plug in coordinates + day and outputs times for sunrise and sunset. Here's what I wrote to dig out JSON for sunrise and sunset times for the next 200 days. Note at least the following goodies:

1. utilizing some nice standard lib functions for formatting
    
2. importing date-fns for some extra date functionality
    
3. top-level await
    
4. writing to disk with Deno.writeTextFile
    

```typescript
import {
  format,
  parse,
  difference,
} from "https://deno.land/std@0.67.0/datetime/mod.ts";
import * as dateFns from "https://cdn.skypack.dev/date-fns@^2.29.2";

async function fetchData(date: string) {
  const res = await fetch(
    `https://api.sunrise-sunset.org/json?lat=61.31667&lng=23.23.75&date=${date}&formatted=0`
  );
  const json = await res.json();
  const { sunrise, sunset } = json.results;
  const parsedSunrise = new Date(sunrise);
  const parsedSunset = new Date(sunset);
  const diff = difference(parsedSunset, parsedSunrise);
  return {
    sunrise: dateFns.addHours(parsedSunrise, 2),
    sunset: dateFns.addHours(parsedSunset, 2),
    diff: diff.seconds,
    date,
  };
}

async function incDate(date: Date, incremented = 0, allData: any[] = []) {
  if (incremented > 200) return allData;
  const formatted = format(date, "yyyy-MM-dd");
  const data = await fetchData(formatted);
  const newDate = dateFns.addDays(date, 1);
  return incDate(newDate, incremented + 1, [...allData, data]);
}

const data = await incDate(new Date());
await Deno.writeTextFile("sun.json", JSON.stringify(data));
```

Since the script sends a fetch request, deno asks for network permission. To grant it without an interactive prompt this script was run with `deno run --allow-net sunrise.ts`

I may have transitioned away from the academy, but I still can't resist a chance to write some R for plotting data. So off I went:

```r

library(dplyr)
library(parsedate)
library(ggplot2)
library(jsonlite)
# read in our data
d <- fromJSON("sun.json")

p2 <- d %>%
  head(230) %>%
  mutate(diff=diff/60/60) %>%
  mutate(sr=parse_date(gsub('^.*T','',sunset))) %>%
  mutate(ss=parse_date(gsub('^.*T','',sunrise))) %>%
  mutate(date=parse_date(date)) %>%
  ggplot( ) +
  geom_line(aes(x=date, y=sr), color='blue') +
  geom_line(aes(x=date, y=ss), color='red')
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1674492278613/1fe26f02-e07c-42fb-a484-1e0b49030f9d.png align="center")

The blue line shows how the sun is setting later and later and the red line demonstrates how the sun is rising earlier and earlier. Turns out it won't be long until it's not dark when I take the kids home from daycare!