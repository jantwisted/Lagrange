---
layout: post
title: "Tor transparent proxy with WOWTor"
author: "jantwisted"
categories: tech
tags: [tor]
image:
---

Have you ever thought of plugging your machine to the Internet and making it anonymous? If you have used Tor browser, I don't have to explain how powerful the Tor network is. There's a nice tutorial in the official Tor documentation, which explains transparently routing traffic through the Tor network.

https://trac.torproject.org/projects/tor/wiki/doc/TransparentProxy

I wrote `WOWTor` for the same purpose and it is fully based on this official documentation. `WOWTor` is a python script, which runs only in Debian with Python3+.

* Project : [source](https://github.com/jantwisted/wowtor)
