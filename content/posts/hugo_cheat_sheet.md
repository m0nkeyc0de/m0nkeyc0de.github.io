---
title: "Hugo Cheat Sheet"
date: 2024-01-16
slug: hugo
draft: false
tags: 
- hugo
categories:
- How-to
---
Quick command reference for using Hugo day to day.

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
