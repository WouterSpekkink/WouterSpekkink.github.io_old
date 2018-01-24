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

In this post I discuss three main topics. I first discuss some basic, somewhat technical details of what attributes in Q-SoPrA are, how they can be organised, and how they are stored, and so on (but this is really quite simple). I then discuss different ways in which attributes can be assigned t;o incidents (and more abstract events; more on this later). Finally, I discuss some ways in which attributes can be used in visualisation and analysis.

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

<blockquote>
Like all other data imported into, or created with Q-SoPrA, attributes are stored in sql tables. One of these tables simply lists the names, descriptions and parents of attributes. The parent of an attributes is either (1) one of the other attributes, or (2) a string called 'NONE', which tells Q-SoPrA that they exist at the highest level of the attributes hierarchy.
</blockquote>

## Assigning attributes
After you create an attribute, you can assign the attribute to [incidents][1] in your data set. The attributes widget lets you scroll through all incidents in your data set, and assign attributes to these incidents. Assigning an attribute to an incident can be done by first selecting it in your attributes tree, and then clicking the *Assign attribute* button. 

<br><br>[![Assigning Attributes](/assets/posts/attributes-in-qsopra/Assigning_Attribute.png){: .center-image}](/assets/posts/attributes-in-qsopra/Assigning_Attribute.png)<br><br>

If an attribute has been assigned to the incident that you are currently inspecting, it will show up in the attributes tree with a **bold** font. If an attribute has a child that has been assigned to the currently inspected incident, then it will show up in the attributes tree with an *italic* font. If an attribute is assigned to the incident, and also has a child that was assigned to the same incident, it will show up with a font that is both **_italic and bold_**.

After assigning an attribute, it is also possible to give the attribute a value. This value can be anything from a numeric value to a string of words. Assigning a value can only be done after assigning the attribute itself. The value can be typed into the appropriate field (one of the fields below the attributes tree), and the stored by clicking the *Store value* button (which is greyed out in the screenshot above). 

<blockquote>
Combinations of attributes and incidents are stored in a separate sql-table. Each row in this table records (1) the name of the attribute, (2) the ID of the incident that the attribute was assigned to, and (3) the value of the attribute (if one was assigned).
</blockquote>

You will notice in the screenshot above that a small fragment of the text in the *Raw* field was highlighted (it is underlined and it is bolded). As I explained in a [previous post][1], the *Raw* field is used to record any fragments of text from your sources (for example, interviews, documents, news items, and etcetera) that were the basis for creating the incident. Q-SoPrA allows you to associate fragments of text in the *Raw* field with attributes that you assign to incidents (however, you can still assign an attribute without highlighting text). Associating a fragment of text with an attribute can be achieved by (1) selecting the attribute, (2) selecting a piece of text with your mouse cursor, and (3) then clicking the *Assign attribute* button. Even when an attribute has already been assigned, you can assign additional fragments of text to the attribute by using this procedure.

<blockquote>
The fragments of text assigned to combinations of attributes and incidents are stored in yet another sql table. In this case the rows of the table record (1) the name of the attribute, (2) the name of the incident that the attribute was assigned to, and (3) the fragment of text that is associated with the attribute-incident combination. Multiple fragments of text may exist per attribute-incident combination. 
</blockquote>

If fragments of text are assigned to an attribute, these will be highlighted whenever the attribute is selected.

### Assigning attributes in the Event Graph Widget

There are two other 'places' in Q-SoPrA where attributes can be assigned to incidents (or events). One of these places is the *Event Graph Widget*. I won't go into the details of the *Event Graph Widget* here, but, briefly, the *Event Graph Widget* is what you will use to visualise (1) incidents and/or events, and (2) relationships between these incidents/events. An event graph gives an abstract overview of the process of interest, and how different developments in the process are related. 

You can also use the *Event Graph Widget* to build more abstract events from your incident data. I will dedicate a future post to an explanation of how exactly this works, but the basic idea underlying the creation of abstract events is that you take a group of inter-related incidents, and say that these together constitute some larger event. For example, we might have a few incidents that capture meetings between actors 'X' and 'Y' that took place over time, and we might group these together in an event that we describe as 'X and Y meet over an extended period of time.' These events are different from your incidents, because they are constituted from incidents, and thus an additional level above your data (or even more than one level, because you can also use abstract events as building blocks for yet more abstract events). 

In the screenshot below you see an example of an event graph that includes incidents, as well as abstract events that have been built from incidents. 

[[SHOW SCREENSHOT OF AN EVENT GRAPH WITH ABSTRACT EVENTS]]

In the left part of the screenshot, you see some details of the currently selected incident (the same details you would see in the *Attributes Widget*), and an attributes tree. In the *Event Graph Widget*, you can select any incident, and then assign attributes to them in the same way you would do this in the *Attributes Widget*. 

One important difference is that, in this case, you can also assign attributes to abstract events. This works largely in the same way (you select an abstract event in the event graph, after which you can assign attributes), with the exception that you cannot associate a particular fragment of text with the attribute-event combination. This is because the abstract events don't have 'raw' data directly associated with them (indeed, 'raw' data can still be indirectly associated, through the incidents that constitute abstract events). Abstract events therefore don't have a *Raw* field.

### Assigning attributes in Hierarchy Graphs

[1]: {{ site.baseurl }}{%post_url 2017-12-26-getting-started-with-qsopra %}
[2]: https://en.wikipedia.org/wiki/Computer-assisted_qualitative_data_analysis_software

