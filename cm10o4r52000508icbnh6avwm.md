---
title: "Giving Deno another try - this time with a Friend"
datePublished: Fri Sep 13 2024 12:02:05 GMT+0000 (Coordinated Universal Time)
cuid: cm10o4r52000508icbnh6avwm
slug: giving-deno-another-try-this-time-with-a-friend
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/MbG7kwWptII/upload/43b055bc6a359fbed9b9cdbd637e00c2.jpeg
tags: deno, temporal-api, chatgpt

---

Kind of ironic that I had a huge hiatus between my latest blog posts and without any planning it just turned out that I continue where I left of: with Deno.

Deno 2.0 is around the corner, which is a logical reason to revisit the subject. Also, I just happened to have an idea for a small ad-hoc script / utility I wanted to make and I remembered how handy it was to pull up deno as a tool for this. Why?

1. Typescript out of the box with no extra hassle
2. A broad standard lib + url based easy dependency imports that don’t require setting up a package.json, running separate installation commands and essentially starting a whole project when all you need is something quick to run
3. Nice, friendly documentation

### So, what is it, this time?

I stay on the command line all day. During the day I tend to juggle a lot between different tasks and tickets and assistance requests and what have you. At the end of the day I always have a hard time remembering how much I allocated time for each task. I wanted a simple way to

1. Get a reminder about writing down what I've been working with for the last hour or so
2. Get a quick prompt on the command line that will take my input and append it to a markdown file for me to look at at the end of each day.

Here's the repo of the end product: https://github.com/hrmJ/whatdidido

### Learned along the way: Temporal api, asking AI for help

I have not been on board with the AI hype and have to admit that do I feel a bit uneasy about the whole thing.
But you do what you have to do and test things out in order to stay relevant. So I opened a ChatGPT prompt in a separate window as I started trying to remember how deno works and how to get up and running. I usually did some double checking: after asking ChatGPT I also googled the same thing and tried to compare the process.

I started with simple questions like "in Deno, what is the simplest way to append a string to a text file?" and got good answers (worth noting that I wasn't using anything like copilot here since I don't yet have code assistants set up due to work/client related restrictions). Usually, same result was achieved with some pretty trivial googling, but I have to admit I liked ChatGPT's output format and the way information was presented to me. I also liked how the whole thing was an ongoing conversation about deno and my specific problem and I did kind of feel like ChatGPT was there to help me out on the whole journey, not just answering separate questions.

Then I wanted to mess around with the new temporal api (https://tc39.es/proposal-temporal/docs/) that should make working with dates and times nicer not just in Deno but in any js runtime once the proposal is finalized.
It's a pretty nice API that does most of what you used to require date-fns (or moment, back in the day) for: for instance, adding and subtracting from a time like on these lines:

```typescript
const previousEntry = Temporal.Now.plainTimeISO();

const endTime = previousEntry
  .add({ minutes: conf.intervalInMinutes })
  .toLocaleString(locale);
```

Something I (and [many others](https://github.com/js-temporal/proposal-temporal-v2/issues/5) ) found frustrating was the lack of supporting custom string formatting: if you want to format a string using
a completely custom format, you have to rely on string manipulation. Every formatting operation is done via locales and
that doesn't always work the way you would like. I, for instance, wanted to log my intervals of how much time was spent on
a task using the format `12:00 - 13:00`. Turned out not to be that easy:

1. Apparently, neither with Temporal api specifically nor with the already existing Intl api can you format a time without seconds.
2. It is impossible to adapt a system like in Finnish where the literary standard is to use dots as hour/minute separators but where colons are the de facto
   way of presenting times as data / in tables, lists etc

It was interesting to go through the saga with ChatGpt:

Me:

> using the Temporal api and a plaintime object, how do i print out a formatted string with just hours and minutes and with colon as the separator?

AI: 

> to format a Temporal.PlainTime object in the Temporal API with just hours and minutes using a colon (:) as the separator, you can use the toLocaleString() method with appropriate options.

```typescript 

const time = Temporal.PlainTime.from("14:30:00");
const formattedTime = time.toLocaleString(undefined, {
  hour: "2-digit",
  minute: "2-digit",
});

console.log(formattedTime);  // Output: "14:30"


```

> This will produce a string like "14:30" (in 24-hour format) or "02:30 PM" (in 12-hour format if hour12: true).


... but it didn't work out in my locale, so I wrote

> Hey, about the code you provided for formatting the time string in Finnish. the code actually outputs dots instead of colons as a separator

AI: 

> You're right! In Finland, the default time format typically uses dots (.) as the separator between hours and minutes, rather than colons. This is the standard for the Finnish locale ("fi-FI"), and it’s why toLocaleString formats times with dots by default

That was actually useful, and I remembered that when writing dates in text the Institute for the languages of Finland actually recommends using dots. But I also remembered that
I encounter colons all the time, so I continued: 


> How interesting that dots would be default for the Finnish locale. I'm Finnish and I very rarely see dots as a separator anywhere between hours and minutes


AI: 

> It is indeed interesting that the Finnish locale (fi-FI) defaults to using dots (.) as the separator between hours and minutes in most programming environments, including toLocaleString(). However, in everyday usage, colons (:) are much more common in Finland, especially in digital displays, printed material, schedules, and other contexts where time is shown.

That sounded right, but I was a bit hesitant and googled about the issue. Turned out, the answer was pretty valid. I also typed, this time in Finnish, in the prompt: 

> Onko kielitoimistolla ohjeita pisteen käytöstä tuntien ja minuuttien erottimena?

(Asking about if the Institute has any recomendations about what to use as a separator).

That caused ChatGPT to run a bing search to bring me the same sources I googled earliear.


All in all, the process was interesting and I liked the way how I could easily turn my coding-related conversation into a conversation about Finnish literary practices and tie it all together... And then return to the coding conversation and ask ChatGPT for advise on how to remove the seconds from my formatted string. It told me:

```typescript 
// Use the Finnish locale and exclude seconds
const formattedTimeFI = time.toLocaleString("fi-FI", {
  hour: "2-digit",
  minute: "2-digit",
  second: undefined  // This removes the seconds from the output
});
```

... But that was just wrong, that didn't do the job, and I got another answer: 

> You're correct, setting second: undefined won't prevent the seconds from being included, as toLocaleString() will default to including them for Temporal.PlainTime objects. Unfortunately, there is no direct way to suppress the seconds using the toLocaleString() options alone.

### Final words

I feel like deno and the temporal API provided some nice practical examples about taking an AI assistant with you when learning about something new. The final product also turned out usable: I just got a reminder to write down that for the last hour I have been writing about deno and AI. One nice thing to note is that deno also provides a `compile` command that creates a single-file executable from your project or script. So I was  able to just run `./whatdidido` as a command and start logging my activities.
