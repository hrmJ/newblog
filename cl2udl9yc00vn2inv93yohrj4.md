---
title: "Running puppeteer in windows from inside wsl"
datePublished: Fri May 06 2022 11:50:23 GMT+0000 (Coordinated Universal Time)
cuid: cl2udl9yc00vn2inv93yohrj4
slug: running-puppeteer-in-windows-from-inside-wsl
cover: https://cdn.hashnode.com/res/hashnode/image/unsplash/Xns9glZ9QOU/upload/v1651837696820/ws_Fzw1VT.jpeg
tags: testing, jest, wsl

---



Call me weird, but I prefer to do all my writing and development from inside
neovim using a Linux terminal. I also want my development environment to reside
in a Linux distro, which -- thanks to wsl -- has become a rather trivial thing
to achieve even on a Windows box. Every once in a while you do, however, notice the 
difference between working inside WSL and a native Linux distro.

How about this one: running headless puppeteer and saving image snaphots?

## The basic problem

Basically, this shouldn't be that difficult: just go `npm run test` inside wsl, let the 
test script run jest with the puppeteer snapshot steps and enjoy the resulting PNGs.
The problem is that when someone in your team runs the same snaphsots inside Windows, the snapshots
are slightly different from the ones produced under WSL/linux (or macos, for that matter):
https://github.com/americanexpress/jest-image-snapshot/issues/201#issuecomment-663325205. 
Using a dockerized setup would be a nice solution, but hey: if I am running windows anyway, shouldn't
it be easy for me to just run the tests outside wsl?

## Let's run it on Windows!

So here's the scenario

1. My code resides inside wsl
2. I install node + npm on Windows
3. I run `npm run test` in the project folder using npm on Windows

Not working. The basic way to access wsl folders in windows is as a network
folder: `\\wsl$`. However, when running npm inside a network folder you get

```
UNC paths are not supported.  Defaulting to Windows directory.
```

...and cannot run the project-specific node code.

### Solution 1: move the project to /mnt/c/whatever

An easy fix, I suppose, would be to move the project under development from
inside WSL to outside: after all, even though Windows accesses wsl stuff using a clumsy 
network setup, wsl can access windows folders as a mounted drive in /mnt/c.

There's a catch: git is practically unusable inside the mounted windows drive. Just cd'ing into
/mnt/c/project takes forever, let alone actually doing real file operations (cf. https://github.com/microsoft/WSL/issues/4197)



### Solution 2: use the network share

Turns out, you can use the network share. The trick is to first run: `pushd \\wsl$\Ubuntu\home\xxx\parent` after which
you end up having a new windows drive z with your project folder inside it. Now
you can go `npx jest target_folder` and your tests do run. However, puppeteer complains:

```
Could not find expected browser (chrome) locally. Run `npm install` to download the correct Chromium revision (970485).
```

Turns out, puppeteer's chrome executable resides inside the network folder and
is blocked from running by the system firewall / antivirus. Time for one last
trick.

### Final touch

To overcome the firewall / antivirus block I did the following:

1. Install puppeteer using npm on windows
2. Explicitly specify chrome's path to point to the windows installation

In my case, puppeteer was used behind
[storyshots](https://github.com/storybookjs/storybook/tree/master/addons/storyshots)
and was configurable via the `chromeExecutablePath` option, which I ended up setting to
`C:\\Users\\xxx\\yyyy\\node_modules\\puppeteer\\.local-chromium\\win64-982053\\chrome-win\\chrome.exe`


