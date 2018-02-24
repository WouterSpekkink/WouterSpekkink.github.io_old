---
layout: post
title:  "Drawing parallel edges in Qt"
date:   2018-02-23 09:07:00 +0000
categories: Software Q-SoPrA Technical Qt
tags: Software Q-SoPrA Technical Qt
---

## Introduction
I wrote a non-technical post on [relationships in Q-SoPrA][1]. One of the things I discuss there is plotting parallel edges, that is, multiple edges between the same pair of nodes. This is something I definitely wanted to be able to do, since I am interested in looking at social arrangements (of people, places, things, etc.) that can be related to each other in various ways. If I want to look at multiple relationships at the same time, being able to visualise parallel edges is a necessity. In this post I discuss some of the details of visualising parallel edges using Qt's tools for visualisation.

## Some things we need for drawing
Let me first briefly touch upon some basic things we need in order to make visualisation widgets like the ones I included in Q-SoPrA. 


[1]: {{ site.baseurl }}{%post_url 2018-02-21-relationships-in-qsopra %}
