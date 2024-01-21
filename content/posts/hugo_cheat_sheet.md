---
title: "Hugo Cheat Sheet"
date: 2024-01-16T20:23:50+01:00
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
hugo server -D --disableFastRender
```

Add a post with **hugo** command
```
hugo new posts/hugo_cheat_sheet.md
```

Add a post manually in **contents/posts/new_post.md**
```
---
title: "New Post"
date: 2024-01-21T10:08:53+01:00
slug: 2024-01-21-dirty_bytes
type: posts
draft: true
categories:
  - default
tags:
  - default
---
```