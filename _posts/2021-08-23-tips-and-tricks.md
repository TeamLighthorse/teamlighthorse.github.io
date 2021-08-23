---
layout: post
title: Tips and Tricks
tags: [tips and tricks]
excerpt_separator: <!--more-->
author: matthews58
---

Tips and Tricks

<!--more-->
### Visual Studio Code

Quick search to open file:
`Ctrl+P`

Zen mode:
`Ctrl+K Z`

Multi cursor selection:
`Ctrl+Alt+Up` or `Ctrl+Alt+Down`

Format whole document:
Install extension Prettier
`Shift+Alt+F`

### Smoke tests

If Smoke test fails, go into artifacts -> recordings -> go to failed test -> open .webm file to watch the video recording.
Download the .har file -> open a new browser -> open network tab -> import .har file to see all the network requests

When developing an e2e test locally, add `slowMo` to browser in order to slow the tests down by 10 seconds.
```ts   
    browser = await chromium.launch({
        args: ["--ignore-certificate-errors"],
        headless: process.env.TF_BUILD === 'True',
        slowMo: 10000
    });
```