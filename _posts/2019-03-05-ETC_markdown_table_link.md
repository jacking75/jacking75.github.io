---
layout: post
title: 마크다운 문법 - Table of contents
published: true
categories: [ETC]
tags: etc md markdown 마크다운
---
<pre>
Table of contents
=================

<!--ts-->
   * [gh-md-toc](#gh-md-toc)
   * [Table of contents](#table-of-contents)
   * [Installation](#installation)
   * [Usage](#usage)
      * [STDIN](#stdin)
      * [Local files](#local-files)
      * [Remote files](#remote-files)
      * [Multiple files](#multiple-files)
      * [Combo](#combo)
      * [Auto insert and update TOC](#auto-insert-and-update-toc)
      * [Github token](#github-token)
   * [Tests](#tests)
   * [Dependency](#dependency)
<!--te-->
</pre> 
  
  
<pre>
Installation
============

Linux
```bash
$ wget https://raw.githubusercontent.com/ekalinin/github-markdown-toc/master/gh-md-toc
$ chmod a+x gh-md-toc
```

OSX
```bash
$ curl https://raw.githubusercontent.com/ekalinin/github-markdown-toc/master/gh-md-toc -o gh-md-toc
$ chmod a+x gh-md-toc
```

Usage
=====


STDIN
-----

Here's an example of TOC creating for markdown from STDIN:
  
</pre>