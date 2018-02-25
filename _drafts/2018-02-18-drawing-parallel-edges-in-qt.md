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
In this post, I will be focusing almost exclusively on the drawing of edges, but I just want to briefly touch upon some of the other basic things we need in order to visualise them. Below you see a screenshot of the Network Graph Visualisation widget with all the menus unfolded. In the middle of this screenshot you see the space where the drawing actually happens. The drawing space itself is an object of the [`QGraphicsViews` class][2], although I should add that in Q-SoPrA I use a sub-classed version in which I re-implemented many of the member functions of this class (the same goes for most other classes I discuss in this post). The `QGraphicsView` object visualises objects that are included in another object of the [`QGraphicsScene` class][3] (a `QGraphicsScene` object needs to be assigned to the *`QGraphicsView` object). These classes are quite well documented in the Qt documentation, so I will not go into details here (also see [this page][4] to read more about the Graphics View Architecture of Qt). 

<br><br>[![Network Graph Widget](/assets/posts/drawing-parallel-edges-in-qt/Network_Graph_Widget.png){: .center-image}](/assets/posts/drawing-parallel-edges-in-qt/Network_Graph_Widget.png)<br><br>

In the screenshot we also see that we have drawn objects in the drawing space. We can see nodes, we can see edges, and we can see labels. All these objects are *items* that are currently included (and visible) in the `QGraphicsScene` object. The nodes are objects of a sub-classed version of the [`QGraphicsItem` class][5], the edges are objects of a sub-classed version of the [`QGraphicsLineItem` class][10], and the labels are objects of a sub-classed version of the [`QGraphicsTextItem` class][6]. 

So, the `QGraphicsView`, the `QGraphicsScene` and the `QGraphicsItem` are the three basic types of objects that you need to make visualisations like the ones included in Q-SoPrA. The `QGraphicsItem`s are the things we want to visualise, the `QGraphicsScene` contains and manages these items, and the `QGraphicsViews` visualises the contents of the `QGraphicsScene`.

## Drawing edges: some basics
Before I get into some specific challenges related to drawing parallel edges, it is useful to briefly discuss what goes into drawing a basic edge. There are a few basic properties of edges that we need to take into account:

1. An edge is mostly a *line* with a *starting point* (the centre of a source node) and an *end point* (the centre of a target node). Where we draw an edge will depend on where its source and target nodes are located. We thus need to somehow explicitly relate edges to their source and target nodes so that they can check their locations before 'drawing themselves'.
2. To indicate the direction of edges we use *arrowheads*. We thus need a way to draw these, which is a bit more complicated that just drawing a line.
3. The *arrowhead* should point at the target node, which means that the *line* we attach it to should stop a small distance before it reaches its *end point*. Otherwise the arrowhead would simply overlap with the target node (see screenshot below; I set the z-level of the source node lower than that of the edge to make clear from where to where we draw the edge's line). 

<br><br>[![Wrong and Right](/assets/posts/drawing-parallel-edges-in-qt/Wrong_Right_Edge.png){: .center-image}](/assets/posts/drawing-parallel-edges-in-qt/Wrong_Right_Edge.png)<br><br>

In Qt's documentation you can find the [Diagram Scene example][7] that achieves almost exactly this (see especially the [header file][8] and the [cpp file][9] of the Arrow class used in this example). The objects that Q-SoPrA uses to draw edges are based on this example, although obviously I had to make a number of changes to get exactly what I need. I will focus on my own adaptation of this example, which is a class I called `DirectedEdge`, but I want to make clear that I obviously did not come up with all of the following myself.

Let us first take a look at the constructor of the `DirectedEdge` class. Here is a code snippet from which I have removed some details that are not important for the examples in this post. 

{% highlight c++ %}
DirectedEdge::DirectedEdge(NetworkNode *startItem, NetworkNode *endItem,
			   QGraphicsItem *parent) : QGraphicsLineItem(parent) {
  // We set the starting point and end point of the edge in the constructor
  start = startItem;
  end = endItem;
  color = Qt::black; // The default color of edges
  // We need a QPen to do the actual drawing
  setPen(QPen(color, 2, Qt::SolidLine, Qt::RoundCap, Qt::RoundJoin));
  }
{% endhighlight %}

As you can see, we are passing pointers to objects of the 'NetworkNode' class to the constructor of 'DirectedEdge'. The 'NetworkNode' class is a sub-classed version I created of the ['QGraphicsItem' class][5]. If we add 'QGraphicsItem's to a 'QGraphicsScene', then the items will be assigned a *scene position*, which can be accessed with the `scenePos()` method, which returns a 'QPointF' object that contains a `x` coordinate and a `y` coordinate. So, when we want to know from which *starting point* to which *end point* to draw our edge, we can simply find these points by using `start->scenePos()` and `end->scenePos()`. If either of these positions changes (for example, because we are dragging one of the network nodes around with our mouse cursor), then the edge will immediately change as well, because its *starting point* and *end point* are tied to the `NetworkNode`s that we passed to it. 




So this is already a very good start! However, things become a little bit more complicated when we want to draw parallel edges.

## Drawing parallel edges
What is so special about drawing parallel edges? Well, we need to make sure that, where parallel (multiple edges between the same pair of nodes) exist, that they are all easily visible. If we would simple draw multiple straight lines with arrowheads, the drawings would overlap, and we would still only be able to see one of the edges (the one that was drawn last). 

In order to prevent this from happening, we can give the lines of the edges a curve, and we can increase the strength of curve as much as is needed. For example, if we have two parallel edges, we give one of the edges a weak curve, and we give the other a strong one, so that they don't overlap. If we have a third parallel edge, we give it an even stronger curve.

We also have to make sure that that arrowhead that we attach to the curve follows the angle of the curve as it reaches its end point. Otherwise we will end up with an awkward looking arrow like the one shown in the screenshot below.

<br><br>[![Awkward arrowhead](/assets/posts/drawing-parallel-edges-in-qt/Wrong_Arrowhead.png){: .center-image}](/assets/posts/drawing-parallel-edges-in-qt/Wrong_Arrowhead.png)<br><br>



[1]: {{ site.baseurl }}{%post_url 2018-02-21-relationships-in-qsopra %}
[2]: http://doc.qt.io/qt-5/qgraphicsview.html
[3]: http://doc.qt.io/qt-5/qgraphicsscene.html
[4]: http://doc.qt.io/qt-5/graphicsview.html
[5]: http://doc.qt.io/qt-5/qgraphicsitem.html
[6]: http://doc.qt.io/qt-5/qgraphicstextitem.html
[7]: http://doc.qt.io/qt-5/qtwidgets-graphicsview-diagramscene-example.html
[8]: http://doc.qt.io/qt-5/qtwidgets-graphicsview-diagramscene-arrow-h.html
[9]: http://doc.qt.io/qt-5/qtwidgets-graphicsview-diagramscene-arrow-cpp.html
[10]: http://doc.qt.io/qt-5/qgraphicslineitem.html
