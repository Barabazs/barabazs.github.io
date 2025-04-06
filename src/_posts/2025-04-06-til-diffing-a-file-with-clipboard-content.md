---
layout: post
title: "TIL: Diffing a File with Clipboard Content"
author: "Barabazs"
date: 2025-04-06 10:40:00 +0200
categories: posts
tags: [TIL]
published: true
permalink: /posts/2025/04/06/til-diffing-a-file-with-clipboard-content
---

I often need to *diff* a file and some pasted text from another application or a remote source. Until now, I was using Meld to paste the text into a new file and then *diff* that.

Don't ask me why, I just never bothered to look for the appropriate syntax to do this in the CLI.

Here's how to do it:

```bash
diff <(pbpaste) path/to/file
```
