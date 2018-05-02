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
In this post, I will be focusing almost exclusively on the drawing of edges, but I just want to briefly touch upon some of the other basic things we need in order to visualise them. Below you see a screenshot of the Network Graph Visualisation widget with all the menus unfolded. In the middle of this screenshot you see the space where the drawing actually happens. The drawing space itself is an object of the [`QGraphicsViews` class][2], although I should add that in Q-SoPrA I use a sub-classed version in which I re-implemented many of the member functions of this class (the same goes for most other classes I discuss in this post). The `QGraphicsView` object visualises objects that are included in another object of the [`QGraphicsScene` class][3] (a `QGraphicsScene` object needs to be assigned to the `QGraphicsView` object). These classes are quite well documented in the Qt documentation, so I will not go into details here (also see [this page][4] to read more about the Graphics View Framework of Qt). 

Also, I recently read [this blog post][11], which seems to suggest that Qt Quick is a potential replacement for the Graphics View Framework. 

<br><br>[![Network Graph Widget](/assets/posts/drawing-parallel-edges-in-qt/Network_Graph_Widget.png){: .center-image}](/assets/posts/drawing-parallel-edges-in-qt/Network_Graph_Widget.png)<br><br>

In the screenshot we also see that we have drawn objects in the drawing space. We can see nodes, we can see edges, and we can see labels. All these objects are *items* that are currently included (and visible) in the `QGraphicsScene` object. The nodes are objects of a sub-classed version of the [`QGraphicsItem` class][5], the edges are objects of a sub-classed version of the [`QGraphicsLineItem` class][10], and the labels are objects of a sub-classed version of the [`QGraphicsTextItem` class][6]. 

So, the `QGraphicsView`, the `QGraphicsScene` and the `QGraphicsItem` are the three basic types of objects that you need to make visualisations like the ones included in Q-SoPrA. The `QGraphicsItem`s are the things we want to visualise, the `QGraphicsScene` contains and manages these items, and the `QGraphicsViews` visualises the contents of the `QGraphicsScene`.

## Drawing edges: some basics
Before I get into some specific challenges related to drawing parallel edges, it is useful to briefly discuss what goes into drawing a basic edge. There are a few basic properties of edges that we need to take into account:

1. An edge is mostly a *line* with a *starting point* (the centre of a source node) and an *end point* (the centre of a target node). Where we draw an edge will depend on where its source and target nodes are located. We thus need to somehow explicitly relate source and target nodes to edges, so that the edges can check their nodes' locations before 'drawing themselves'.
2. To indicate the direction of edges we use *arrowheads*. We thus need a way to draw these, which is a bit more complicated than just drawing a line.
3. The *arrowhead* should point at the target node, which means that the *line* we attach it to should stop a small distance before it reaches its *end point*. Otherwise the arrowhead would simply overlap with the target node (see screenshot below; I set the z-level of the source node lower than that of the edge to make clear from where to where we draw the edge's line). 

<br><br>[![Wrong and Right](/assets/posts/drawing-parallel-edges-in-qt/Wrong_Right_Edge.png){: .center-image}](/assets/posts/drawing-parallel-edges-in-qt/Wrong_Right_Edge.png)<br><br>

In Qt's documentation you can find the [Diagram Scene example][7] that achieves almost exactly this (see especially the [header file][8] and the [cpp file][9] of the Arrow class used in this example). The objects that Q-SoPrA uses to draw edges are based on this example, although obviously I had to make a number of changes to get exactly what I need. I will focus on my own adaptation of this example, which is a class I called `DirectedEdge`, but I want to make clear that I obviously did not come up with all of the following myself.

Let us first take a look at the constructor of the `DirectedEdge` class. Here is a code snippet from which I have removed some details that are not important for the examples in this post. 

{% highlight c++ %}
DirectedEdge::DirectedEdge(NetworkNode *startItem, NetworkNode *endItem, QString submittedType,
			   QString submittedName, QGraphicsItem *parent)
  : QGraphicsLineItem(parent) {
  start = startItem; // setting the start item of this edge.
  end = endItem; // setting the end item of this edge.
  color = Qt::black;
  setPen(QPen(color, 2, Qt::SolidLine, Qt::RoundCap, Qt::RoundJoin));
  height = 20;
  relType = submittedType;
  name = submittedName;
  filtered = true;
  massHidden = false;
  setFlag(QGraphicsItem::ItemSendsGeometryChanges);
  comment = "";
}
{% endhighlight %}

As you can see, we are passing pointers to objects of the `NetworkNode` class to the constructor of `DirectedEdge`. The `NetworkNode` class is a sub-classed version I created of the [`QGraphicsItem` class][5], and I use this class to draw the nodes of my network diagrams. When we add `NetworkNode`s (or any other `QGraphicsItem` to a `QGraphicsScene` object, then these will be assigned a *scene position*, that is, a point in the scene that is defined by an x-coordinate and a y-coordinate. We can access the *scene position* of a `NetworkNode` (or other types of `QGraphicsItems`) by using the ['scenePos()' member function][13]. This will return a ['QPointF' object][14] that contains the item's coordinates. So, if we want to know from where to where to draw a certain edge, the obvious thing to do would be to first find the *scene positions* of its start node - start->scenePos() - and end node - end->scenePos() - and then draw the line between those two positions.

As I mentioned before, we actually want our line to stop shortly before it reaches its end point. What we could do is to create a [`QLineF` object][15], passing our start and end points as parameters, and then use the `setLength()` function to make the line slightly shorter: `myLine.setLength(myLine.length() - 20)`. 

Then we still need to add our arrowhead. For this, I simply followed the [Diagram Scene Example][7] provide in the online documentation for Qt. Basically, this involves creating a [`QPolygonF`][16] object with the shape of our arrowhead, and have this object drawn near the end point of our line. 

We should do most of the above in the ['paint()' function][12] of our edge, because that is the function where we determine where and how the edge is drawn. I have included a code snippet below to illustrate what our `paint()` function might look like in this scenario.

{% highlight c++ %}
const qreal Pi = 3.14;

void DirectedEdge::paint(QPainter *painter, const QStyleOptionGraphicsItem *, QWidget *) {
  QPen myPen = pen(); // We need to create a pen for our painter...
  myPen.setColor(color); // ...and indicate which colour it will use.
  painter->setPen(myPen); // Now we set the pen to your painter.
  QLineF myLine = QLineF(start->scenePos(), end->scenePos()); // Here we create our line.
  myLine.setLength(myLine.length() - 20); // We shorten our line.
  
  // And then we create our arrowHead.
  qreal arrowSize = 10.0;
  QPointF arrowP1 = myLine.p2() - QPointF(sin(angle + Pi /3) * arrowSize,
    cos(angle + Pi / 3) * arrowSize);
  QPointF arrowP2 = myLine.p2() - QPointF(sin(angle + Pi - Pi / 3) * arrowSize,
  cos(angle + Pi - Pi / 3) * arrowSize);
  
  arrowHead.clear();
  arrowHead << myLine.p2() << arrowP1 << arrowP2;

  painter->drawLine(myLine);
  painter->drawPolygon(arrowHead);
}
{% endhighlight %}

And that should do the trick. At least, if we were only interested in drawing edges as straight lines. However, what if we want to draw parallel edges, that is, multiple edges between the same pair of nodes? In this case, if we would just use straight lines, then the edges would overlap, and we would actually not be able to see that multiple edges exist between our nodes. In this case, it is better to draw edges as curved lines, and to change the strength of the curve for each additional edge that we add to a given pair of nodes. The remained of this post will be about how we can do this, and what challenges we will face.

## Drawing parallel edges

Drawing a curved line with the Qt library is not difficult to do. One of the easiest ways to `paint()` a curved line is by creating a [`QPainterPath` object][17], and use its [`quadTo()` function][18]. This function takes two arguments: One of the arguments is the end point that we want to draw the curved line to, and the other argument is a so-called control point that we will use to determine how the line will be curved (how strong the curve will be and which direction it will curve in). We do not give the function a starting point. Instead, we should move the `QPainterPath` object to our starting point using its ['moveTo()' function][19] before calling the `quadTo()` function.

So, what about this control point? Consider the image I have linked to below (found through this [Stack OverFlow][20] discussion). The image shows nicely how the control point works. It is a point somewhere 'above' the place where we want our line to curve towards the control point. In the image you see that if we place the control point somewhere above the middle of the line, then the line will also curve around its middle point. This is exactly what I wanted for my parallel edges. In the image you also see that the curve will change if we move the control point closer to the starting point or the end point. This is something that I want to avoid.

<br><br>[![Bezier curves](https://www.andrew.cmu.edu/course/98-222/lecture/lec7/img/quadraticcurve.gif){: .center-image}](https://www.andrew.cmu.edu/course/98-222/lecture/lec7/img/quadraticcurve.gif)<br><br>

So far so good. Assume that our starting point is at coordinates `(0, 5)` in the scene, and our end point is at coordinate `(10, 5)` in the scene, we can first simply calculate the point that lies exactly in between them: `x = (10 + 0) / 2 = 5` and `y = (5 + 5) / 2 = 5`, giving us the point at coordinates `(5, 5)`. Then we still need to set the 'height' of the curve, which, in this case, we can do simply by adding some constant to the y-coordinate of our middle point, giving us, for example, the point at coordinates `(5, 25)`. If we then pass this point to the `quadTo()` function, we will get a nice curved line.

This example was relatively simple, because the slope of the straight line between our starting point and our end point is 0. This makes finding the control point relatively straightforward. However, consider now that we have a line that starts at coordinates `(0, 5)` and ends at coordinates `(10, 10)`. Finding the point that lies exactly in the middle is still quite easy: `x = (0 + 10) / 2 = 5` and `y = 5 + 10 / 2 = 7.5`. However, how do we now find the control point, somewhere 'above' this middle point? We cannot simply at a constant value to the y-coordinate of our middle point, because that would place the control point somewhere right from the middle of the line, and create a curve that skews to the left. It is very likely that someone with more training in geometry will have an easy solution to his problem, and could tell me very quickly how to find our control point in this case. I came up with a different solution that makes use of the flexibility of the [`QPainter` class][21]. The steps that I take are as follows:

### Finding the distance between the two points
I first calculate the distance between my starting point and my end point, which is basically what the length of a straight line between these points will be: 

{% highlight c++ %}

  qreal xDiff = end->pos().x() - start->pos().x();
  qreal yDiff = end->pos().y() - start->pos().y();

  qreal distance = sqrt(pow(xDiff, 2) + pow(yDiff, 2));
  
{% endhighlight %}

### Imagining a 0-slope line of the same length
The next step is to draw a line with a slope of `0`, and with a length that equals the distance we have just created. Actually, we do not actually draw this line, but we define the starting point and end point that this line would have. It does not really matter all that much what the starting point is. What is important, is that the y-coordinates of the starting point and the end point are the same. We could define our two points simply as follows:

{% highlight c++ %}
  QPointF tempStart = QPointF(0, 0);
  QPointF tempEnd = QPointF(distance, 0); // The distance was calculated in the previous snippet.
{% endhighlight %}


### Calculate the control point for the imaginary line
Now can calculate the point that lies in the middle of the two temporary points we just defined. Then, since the line has a slope of `0`, we can simply add a constant value to the y-coordinate of the middle point to create a control point that lies exactly above the middle point. 

{% highlight c++ %}
  qreal mX = (tempStart.x() + tempEnd.x()) / 2;
  qreal mY = (tempStart.y() + tempEnd.y()) / 2;
  midPoint = QPointF(mX, mY);
  midPoint.setY(midPoint.y() + height); // We use a separate function to set the height.
{% endhighlight %} 

With the information we have now, we could draw a curved line. However, this line would be drawn at the wrong location. To correct this, we take some additional steps.

### Moving the curved line to the correct location
First, we can easily set the correct starting point for our line by using the `moveTo()` function of the `QPainterPath` class. Indeed, our end point will still be incorrect. But here is a trick we can use: We can rotate our `QPainter` object, using its [`rotate()` function][22]. The neat thing about this is that this will rotate the `QPainter` within the coordinate system of the `QScene` object that we are drawing in. The `QPainter` will still keep its local coordinates, which means that we can still treat our line as if it is a line with a slope of `0`, even though it is actually rotated in the `QScene`. 

To find the rotation value, we first calculate the angle between our two points (relative to the x-axis of the `QScene`). This will give us a value in radians that we should then convert to degrees.

{% highlight c++ %}
  // theta is initially the angle in radians, so we convert it to degrees.
  theta = atan2(yDiff, xDiff);
  theta = qRadiansToDegrees(theta);
{% endhighlight %} 



My implementation of the paint function is as follows:

{% highlight c++ %}
const qreal Pi = 3.14;

void DirectedEdge::paint(QPainter *painter, const QStyleOptionGraphicsItem *, QWidget *) {
  QPen myPen = pen();
  myPen.setColor(color);
  painter->setPen(myPen);
  painter->setBrush(color);
  calc(); // see below

  arrowHead.clear();
  arrowHead << oLine.p2() << arrowP1 << arrowP2;

  QPainterPath myPath;
  myPath.moveTo(tempStart);
  myPath.quadTo(midPoint, oLine.p2());
  painter->translate(start->pos() - tempStart);
  painter->rotate(theta);
  painter->drawPolygon(arrowHead);
  painter->strokePath(myPath, QPen(color));
}

void DirectedEdge::calc() {
  // Let us first calculate the distance between our two points.
  qreal xDiff = end->pos().x() - start->pos().x();
  qreal yDiff = end->pos().y() - start->pos().y();

  qreal distance = sqrt(pow(xDiff, 2) + pow(yDiff, 2));

  // Then we draw two new points that are on a horizontal line.
  // This makes it easier to draw a control point perpendicular to the line.
  tempStart = QPointF(0, 0);
  QPointF tempEnd = QPointF(distance, 0);

  // Calculate midpoint
  qreal mX = (tempStart.x() + tempEnd.x()) / 2;
  qreal mY = (tempStart.y() + tempEnd.y()) / 2;
  midPoint = QPointF(mX, mY);
  midPoint.setY(midPoint.y() + height);

  QLineF newLine = QLineF(tempStart, tempEnd);
  setLine(newLine);

  // But then we still need to rotate and translate the painter such that it draws
  // the line between the correct points. 
  // theta is initially the angle in radians, so we convert it to degrees.
  theta = atan2(yDiff, xDiff);
  theta = qRadiansToDegrees(theta);

  oLine = QLineF(midPoint, tempEnd);
  oLine.setLength(oLine.length() - 18);
  
  double angle = ::acos(oLine.dx() / oLine.length());
  if (oLine.dy() >= 0)
    angle = (Pi * 2) - angle;

  qreal arrowSize = 10;
  
  arrowP1 = oLine.p2() - QPointF(sin(angle + Pi /3) * arrowSize,
    cos(angle + Pi / 3) * arrowSize);
  arrowP2 = oLine.p2() - QPointF(sin(angle + Pi - Pi / 3) * arrowSize,
  cos(angle + Pi - Pi / 3) * arrowSize);
  prepareGeometryChange();
}

{% endhighlight %}

As you can see, the paint function calls another function that I called `calc()`, which I have also included in the code snippet. The `calc()` function is actually where the main work on deciding where and how to draw the edge is done. 





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
[11]: http://blog.qt.io/blog/2017/01/19/should-you-be-using-qgraphicsview/
[12]: http://doc.qt.io/qt-5/qgraphicsitem.html#paint
[13]: http://doc.qt.io/qt-5/qgraphicsitem.html#scenePos
[14]: http://doc.qt.io/qt-5/qpointf.html
[15]: http://doc.qt.io/qt-5/qlinef.html
[16]: http://doc.qt.io/qt-5/qpolygonf.html
[17]: http://doc.qt.io/qt-5/qpainterpath.html
[18]: http://doc.qt.io/qt-5/qpainterpath.html#quadTo
[19]: http://doc.qt.io/qt-5/qpainterpath.html#moveTo
[20]: https://stackoverflow.com/questions/50129580/program-to-find-line-segment-and-bezier-curve-intersection
[21]: http://doc.qt.io/qt-5/qpainter.html
[22]: http://doc.qt.io/qt-5/qpainter.html#rotate
