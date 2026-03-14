---
title: "Quick References"
date: 2026-03-14
slug: quickref
type: posts
draft: false
categories:
  - Miscellanous
tags: []
weight: 2
---
Repertory of useful commands.

# Git and Github

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

