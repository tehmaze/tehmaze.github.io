---
layout: post
title: go-opensmtpd v52
categories: [blog,hacking]
share: true
comments: false
excerpt: go-opensmtpd updated to OpenSMTPD API version 52
tags: [hacking]
author: maze
image:
  feature: /posts/hacking.jpg
date: 2018-10-16T08:56:00+02:00
---

[go-opensmtpd](https://gopkg.in/opensmtpd.v52) has been updates to reflect
the changes in OpenSMTPD API version 52. The API is still not stable, and
the filter API has been discontinued. 

The only working plugin type are tables at the time of writing, on OpenSMTPD
version 6.0.3p1.

Project sources at https://github.com/go-opensmtpd/opensmtpd
