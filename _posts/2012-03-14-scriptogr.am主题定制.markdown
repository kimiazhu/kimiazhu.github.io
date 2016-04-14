---
layout:     post
title:      "Custom the Readnik Theme of Scriptogr.am"
date:       2012-03-14 15:53
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - scriptogr.am
---

Just a backup for the modified things..

scriptogr.am is a very wonderful blog tool, i hope it will be annother Wordpress in the near future, but for now there are still a lot things can be made it better, Specially the theme System..

i choose the *Readnik* theme and spend a few minites to make it look better (to me :D) by adding some CSS code, just a little:

###Enlarge Page width
1. in the main css, find *body* element and change the *width* property from 640px to 840px.

2. find the only property *width* in *.blog* and change the value to 840px

###Make the Code Looks More Pretty

```css

div.body-post p code {	background-color: #f5ff66;}	pre.prettyprint {    border-style: ridge;    background-color: #F6FEF2;    font-size: 15px;    line-height: 22px;    max-width: 840px;    word-break:  keep-all;    overflow: auto;}
```
put the above code to bottom of the main css of the site and save, refresh then you can see the effect, just like you see it in this article.