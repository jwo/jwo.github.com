---
layout: post
title: "Using pg_dumpall to move all postgres databases to a new laptop"
date: 2014-10-16 10:35
comments: true
categories: 
---

I recently upgraded laptops because it’s been 2 years and my business lease on
it was up. Which: cool to have the new MacBook Air with the day-long-battery TM, but moving computers is a pain.  

Except: I have [dot files](https://github.com/jwo/dotfiles) and that makes it easier. After copying my home directory over, re-dotfiling, things
seemed good.  

Except: my postgres databases. How to get them all from Air A to Air B without copying every.single.one.  

Enter: pg_dumpall. It’ll dump every database into a file, with which you import on new computer.

When moving to a new computer

```bash
$ pg_dumpall > db.out
```

(Move file to to new computer)

```bash
$ psql -f dbout postgres
```

> if you have trouble with “database $username not found”, type in “createdb”


To confirm, 

```bash
psql
```

And then `\l` to list the databases


