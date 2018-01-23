---
layout: post
title:  "Attributes in Q-SoPrA"
date:   2018-01-23 09:07:00 +0000
categories: Software Q-Sopra Non-Technical 
tags: Software Q-Sopra Non-Technical 
---

## Introduction
One of the tasks that you are likely to use Q-SoPrA for is the qualification of incident data (see [this earlier post][1] for an explanation of what incidents are), by assigning codes to them. The tools you will use for this are not very different from those that you would typically use to assign codes to interviews or the contents of documents (using [CAQDAS software][2]. In Q-SoPrA, the codes that are used to qualify incident data are referred to as *attributes*. 

You can use attributes to capture anything in the incidents that is of interest to you. For example, you could use attributes to distinguish between different types of activities, to identify actors or other entities involved in incidents, to record changes in variables associated with the incidents, and so forth. What exactly you will use attributes for will depend heavily on your research question(s), and the theoretical basis for your research. You might of course also use attributes in a 'grounded research' approach, where you develop attributes on the fly, and develop these further as you proceed with your project. In this post I try to be mostly agnostic to different ways in which attributes might be used, while discussing various ways in which attributes (can) play a role when using Q-SoPrA in research projects.

In this post I discuss three main topics. I first discuss some basic, somewhat technical details of what attributes in Q-SoPrA are, how they can be organised, and how they are stored, and so on (but this is really quite simple). I then discuss different ways in which attributes can be assigned to incidents (and more abstract events; more on this later). Finally, I discuss some ways in which attributes can be used in visualisation when using Q-SoPrA in research projects.

In this post I discuss three main topics. I first discuss some basic, somewhat technical details of what attributes in Q-SoPrA are, how they can be organised, and how they are stored, and so on (but this is really quite simple). I then discuss different ways in which attributes can be assigned to incidents (and more abstract events; more on this later). Finally, I discuss some ways in which attributes can be used in visualisation and analysis.

## Attributes in Q-SoPrA
Attributes have to be defined by the user. There are three different widgets in which this can be done, but the most obvious 'place' to define new attributes is the *Attributes Widget* (see screenshot below).
 
<br><br>[![Attributes Widget](/assets/posts/attributes-in-qsopra/Attributes_Widget.png){: .center-image}](/assets/posts/attributes-in-qsopra/Attributes_Widget.png)<br><br>

On the left side of this screen you see some fields that show the data associated with the incident that is currently being inspected (the timing, source, description, raw source text, and other comments/memos written by the analyst). On the right side of the screen you can see the attributes view. When you create a new database, this view will be empty. In the bottom-right of the screen you see all controls that are specifically associated with attributes. For example, you find controls to define new attributes, to edit existing ones, to assign the currently selected attribute to the present incident, and etcetera. 

### Creating new attributes
We will focus first on the creation of new attributes. Attributes in Q-SoPrA can be understood to have three main properties: 

 1. All attributes have a unique label (or name);
 2. All attributes have a description (or definition);
 3. Attributes *can* have a parent attribute.
 
To create a new attribute, click the **New Attribute** button. This will open the dialog shown below.

<br><br>[![Attribute Dialog](/assets/posts/attributes-in-qsopra/Attribute_Dialog.png){: .center-image}](/assets/posts/attributes-in-qsopra/Attribute_Dialog.png)<br><br>

You are always required to provide a label and a description for your attributes. If one of these is missing, then Q-SoPrA won't allow you to save the attribute. Also, the name of the attribute has to be unique. The idea behind forcing you to provide a description is to make you think immediately about the definition of your attribute, and about the phenomena that your attribute is supposed to capture. 

### Attribute hierarchies
I mentioned above that attributes *can* have parents. That is, attributes in Q-SoPrA can be hierarchically organised by assigning parents/children to them. A parent is automatically assigned to a new attribute if you exist an existing one before clicking the *New Attribute* button. In this case, the new attribute will be created as a child of the existing attribute you selected. You can re-parent attributes by dragging and dropping them onto other attributes. If you drag an attribute to an empty space in the attribute view, the attribute will be 'orphaned', and appear in the highest level of the attribute hierarchy. In this way, you can create as many hierarchical levels of attributes as you like.

<br><br>[![Attributes Tree](/assets/posts/attributes-in-qsopra/Attributes_Tree.png){: .center-image}](/assets/posts/attributes-in-qsopra/Attributes_Tree.png)<br><br>

The position of an attribute in the hierarchy has important consequences. An attribute that has a parent is always considered as a sub-type of that parent. For example, if you have an attribute called *Activities*, and you have another attribute called *Night-time activities* that is a child of *Activities*, then Q-SoPrA assumes that *Night-time activities* are a kind of *Activities*. In this way, any attribute can serve as a sort of category for other attributes. 

The specific organisation of your attributes hierarchy will be important in, among other things, the identification of 'modes' in your event graph (modes are a topic I will have to deal with in more detail in a future post): When you create a new mode based on the attribute *Activities*, then all incidents and events that were assigned an attribute that is a child (or grandchild, or great-grandchild, or great-great... Well, you get the idea) of *Activities* will also be considered to belong to that mode. 



[1]: {{ site.baseurl }}{%post_url 2017-12-26-getting-started-with-qsopra %}
[2]: https://en.wikipedia.org/wiki/Computer-assisted_qualitative_data_analysis_software

