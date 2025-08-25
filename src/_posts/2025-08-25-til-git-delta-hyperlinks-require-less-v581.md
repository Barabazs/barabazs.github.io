---
layout: post
title: "TIL: git-delta hyperlinks require less v581+"
author: "Barabazs"
date: 2025-08-25 13:00:00 +0200
categories: posts
tags: [TIL, git-delta, less, devcontainer]
published: true
permalink: /posts/2025/08/25/til-git-delta-hyperlinks-require-less-v581
---

I was puzzled why my git-delta was showing malformed hyperlink escape sequences like `8;;https://github.com/...8;;` instead of proper clickable links. The escape sequences were being truncated and
weren't following the proper hyperlink protocol. 

This issue only occured in devcontainers, not on my local machine. The git-delta configuation, git config, and terminal settings were identical. And even the versions were the same.

The issue was simply that less version was too old: I had less 551 on Debian 11 (Bullseye), but hyperlinks require version ≥ 581.

Here's what different Debian versions ship with:
- Debian 11 (Bullseye): less 551 ❌ (truncates escape sequences)
- Debian 12 (Bookworm): less 590 ✅
- Debian 13 (Trixie): less 668 ✅

The older less version wasn't just failing to make links clickable, it was mangling the escape sequences entirely, showing raw partial sequences instead of either clickable links or clean text.

The fix was simply compiling a newer less version:

# Download and compile latest less
```bash
curl -L -o less-679.tar.gz "https://www.greenwoodsoftware.com/less/less-679.tar.gz"
tar -xzf less-679.tar.gz && cd less-679
./configure --prefix=/usr/local && make && sudo make install
```

Now `git show` and `git log` display proper clickable hyperlinks instead of garbled escape sequences.

Thank you Claude Code for finding and fixing the issue in less than 5 minutes.