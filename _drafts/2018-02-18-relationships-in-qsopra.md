---
layout: post
title:  "Relationships in Q-SoPrA"
date:   2018-02-18 09:07:00 +0000
categories: Software Q-Sopra Non-Technical
tags: Software Q-Sopra Non-Technical 
---

## Introduction: Process *and* structure
As the name suggests, Q-SoPrA (**Q**ualitative **So**ftware for **Pr**ocess **A**nalysis) is focused first and foremost on the qualitative analysis of social *processes*. However, I invested a lot of time and energy in also creating features that can be used to study how these processes relate to *structures*. In my opinion, this is a somewhat obvious things to do, since so many social theories posit some kind of relationship between process and structure (often couched in terms of *agency and structure*. I can use my own application of Q-SoPrA as an example. I am currently using Q-SoPrA in a study that, conceptually, builds on [Schatzki's][1] concepts of *social practices* (in short, organised doings and sayings) and *social arrangements* of people, artefacts, places, and other types of entities. I reconstruct practices as sequences of activities ([Schatzki][1] also writes about *chains of action* in this context), and I reconstruct arrangements as networks of relationships between entities. One aspect of my investigation is to study how unfolding practices relate to (changes in) arrangements. 

I believe that many other theoretical perspectives are compatible with Q-SoPrA, although Q-SoPrA probably fits best with perspectives that assume *primacy of process over structure* (see [Rescher][2]). Very simply said, this means that Q-SoPrA connects best with the perspective that social structures are (re)produced by social processes, as opposed to the idea that social processes are brought about by social structures. 

So how does Q-SoPrA assume primacy of social process? Well, by using [incidents][3] as indicators for relationships (although it is of course equally possible to think of incidents as enactments of relationships without changing the general approach). More specifically, Q-SoPrA allows you to define relationships (I discuss the details below), and then assign these to incidents in the same way that you would assign [attributes][4]. This also creates the benefit that it becomes possible to study relationships that were indicated by incidents in a particular part of the process, thereby providing a rudimentary way to look at changes of relationships over time, as I discuss further below. 

In the past, I used to do something similar by assigning actors to events (as attributes), and then looking at the co-participation of actors in events over time (also see my post about [bi-dynamic line graphs][5]). Two important limitations of this approach are that (1) *co-participation* is basically an abstract summary of many different ways in which actors can be related to each other through events, and (2) not all relationships that are *indicated* by events are necessarily captured by their co-participation. To offer an example of the first limitation, imagine that we have two events in which the same pair of actors interacted with each other, but that in one event the interaction concerned the joint organisation of an activity, and in the other event the interaction concerned one actor providing financial support to the other actor. If we simply model both occasions as co-participation, then we lose the ability to distinguish between the two situations. To offer an example of the second limitation, imagine that we have an activity in which an individual is acting as a representative of a certain group. If we want to capture this fact, it would be awkward to capture it as co-participation of the individual and the group in the activity. Instead, it would make more sense if we could simply define a *membership* relationship, and say that the activity is an indication of the individual being a member of the group. 

The way I implemented relationship coding in Q-SoPrA is basically an attempt to take away these limitations. In addition, I made sure that Q-SoPrA is able to visualise networks of relationships that have multiple modes (for an introduction into two-mode networks, see this [blog post][6]; Q-SoPrA allows you to define more than two modes), and multiple types of relationships. Parallel edges (multiple edges between the same pair of entities) are drawn with curved lines to make sure that multiple types of relationships can be visualised in one graph. In addition, it is possible to perform multi-mode transformations to infer 'latent' relationships from observed ones. I will explain all of this in detail in the remainder of this post.

# The relationships widget

<br><br>[![Relationship Widget](/assets/posts/relationships-in-qsopra/Relationships_Widget.png){: .center-image}](/assets/posts/relationships-in-qsopra/Relationships_Widget.png)<br><br>



[1]: https://books.google.co.uk/books?id=753WUmtDIiwC&printsec=frontcover&dq=The+Site+of+the+Social&hl=nl&sa=X&ved=0ahUKEwjPvMuKta_ZAhWNLlAKHZz0DIMQ6AEIJzAA#v=onepage&q=The%20Site%20of%20the%20Social&f=false
[2]: https://books.google.co.uk/books?id=8MjR29eJLYIC&printsec=frontcover&dq=Process+metaphysics&hl=nl&sa=X&ved=0ahUKEwj-8drota_ZAhVCZ1AKHVUwBKUQ6AEIJzAA#v=onepage&q=Process%20metaphysics&f=false
[3]: {{ site.baseurl }}{%post_url 2017-12-26-getting-started-with-qsopra %}
[4]: {{ site.baseurl }}{%post_url 2018-01-25-attributes-in-qsopra %}
[5]: {{ site.baseurl }}{%post_url 2017-03-10-bi-dynamic-line-graphs %}
[6]: https://toreopsahl.com/tnet/two-mode-networks/defining-two-mode-networks/
