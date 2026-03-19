---
title: "Cheat Sheet"
date: 2026-03-14
slug: cheatsheets
type: posts
draft: false
categories:
  - Miscellanous
tags: []
weight: 2
toc: true
---
Repertory of useful commands which don't need their own page.

## Git and Github

Undo last commit
```bash
git reset HEAD~
```

Set username and email
```bash
git config --global user.name "m0nkeyc0de"
git config --global user.email "30374340+m0nkeyc0de@users.noreply.github.com"
```

SSH remote instead of HTTPS
```bash
git remote remove origin
git remove add origin git@github.com:m0nkeyc0de/repo_name.git
git push --set-upstream origin main
```
## Hugo
Run in local
```
hugo server --disableFastRender --logLevel=debug
```

Add a post with **hugo** command
```
hugo new posts/hugo_cheat_sheet.md
```

Add a post manually in **contents/posts/new_post.md**
```
---
title: "New Post"
date: 2024-01-21
slug: new_post
type: posts
draft: true
categories:
  - default
tags:
  - default
---
```
