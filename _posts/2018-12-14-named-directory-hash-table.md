---
layout: post
title:  "The zsh named directory hash table"
author: Jeremy
---

Like all cool things, it started with [a reddit comment](https://www.reddit.com/r/zsh/comments/9eklqu/shortcut_links_in_zsh/).
Someone recommended to use the zsh directory hash table.
It enabled me to set custom directory replacements like this:
![01](/assets/named-directory-hash-table/01.png)

This worked nicely, but I was curious about how it worked.  
Since is a zsh built-in function, the docs can be accessed with `man zshbuiltins`.

> hash can be used to directly modify the contents of the command hash table, and the named directory hash table. Normally one would modify these tables by modifying one’s PATH (for the command hash table) or by creating appropriate shell parameters (for the named directory hash table). The choice of hash table to work on is determined by the -d option; without the option the command hash table is used, and with the option the named directory hash table is used.

To display the command hash table contents, run `hash`. For the directory hash table, run `hash -d`.

So the use of `hash -d github=~/Documents/GitHub` only added the value to a cache.  
I found out that you can also add a static named directory like this:
![02](/assets/named-directory-hash-table/02.png)

The documentation for this behaviour can be found [here](http://zsh.sourceforge.net/Doc/Release/Expansion.html#Static-named-directories).  
So every variable starting with a slash is considered. Also every users home is included automatically.
