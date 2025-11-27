---
layout: post
title: "TIL: comparing two git stashes"
author: "Barabazs"
date: 2025-11-27 17:00:00 +0100
categories: posts
tags: [TIL, git, stash]
published: true
permalink: /posts/2025/11/27/til-git-diff-stashed-files-only
---

I had two stashes containing overlapping files and needed to know if the stashed files had different changes between them.

## The Scenario

- `stash@{0}` contains files `a`, `b`, `c`, `d`
- `stash@{1}` contains files `a`, `b`, `c`

I wanted to know: did files `a`, `b`, or `c` change differently between the two stashes?

## Wrong Approach

```bash
git diff stash@{1} stash@{0}
```

This compares the full repository state at each stash point, including all unrelated files.

## Right Approach

```bash
git diff stash@{1}^1 stash@{1} stash@{0}^1 stash@{0}
```

This isolates only the stashed changes:

- `stash@{1}^1..stash@{1}` = what changed in stash 1
- `stash@{0}^1..stash@{0}` = what changed in stash 0
- Diff those two sets to see if the overlapping files evolved differently
