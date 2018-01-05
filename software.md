--- 
layout: page
title: Software
permalink: /software/
order: 4
---

## The software on this page

I am an amateur coder, currently working mostly with C++, and making heavy use of the Qt library for GUI functions, interfacing with Sql databases, and drawing graphics (you might say that I code in Qt rather than C++). Occasionally, I also write scripts for R (usually for quick work on data tables or matrices), and I have done very few things in Java (primarily writing plugins for [Gephi][1]) and Python. 

In the past, I use to share downloads for some of the software that I wrote directly from my website, but I decided to switch from WordPress to Github Pages (the page you are currently looking at), and to share software directly from the Github repositories. 

On this page, I either share direct links to the downloads attached to my Github repositories, or I share links to project pages for larger projects, such as Q-SoPrA. For the plugins I created for [Gephi][1], you'll have to use the Plugins menu of Gephi. 

### Contents

 * [Note on Q-Sopra](#a-note-on-q-sopra)
 * [Event Graph Layout](#event-graph-layout)
 * [Bi-Dynamic Line Graphs](#bi-dynamic-line-graphs)
 * [Multidimensional Scaling in Gephi](#multi-dimensional-scaling-in-gephi)
 * [Lineage plugin for Gephi](#lineage-plugin-for-gephi)
 * [Scripts for R](#scripts-for-r)
 
## A note on Q-SoPrA
I am currently working on an integrated software package for Qualitative analysis of Social Processes, called Q-SoPrA (**Q**ualitative **So**cial **Pr**ocess **A**nalysis). This is an open source project, written in C++, making heavy use of the Qt5 library (I won't actually open the source until I have a more or less fully featured beta version, and when I have got a wiki up and running for Q-SoPrA). Work on Q-SoPrA is still in progress, but I am nearing a releasable version.

Q-SoPrA will hopefully make some of the plugins I wrote for Gephi redundant, in the sense that it has built-in features for visualisation of [event graphs](#event-graph-layout) (and much more), finding [lineages](#lineage-plugin), and so on. I also included options to abstract network data from event data, and visualise the resulting networks. I was also able to include some advantageous options that are currently not (yet) available in Gephi, such as the ability to visualise parallel edges, and to perform quite advanced multi-mode transformations on the fly (Gephi does have a plugin for multi-mode transformations, but the options built into Q-SoPrA are more advanced). 

The initial releases of Q-SoPrA won't include any possibilities to obtain network metrics, but I do plan to implement such possibilities on the long term. For now, Q-SoPrA simply allows you to export node and edge lists, which can then be imported in other software for quantitative analysis (as the name suggests, the emphasis of Q-SoPrA is on qualitative analysis). 


## Event Graph Layout
The Event Graph Layout is a layout plugin for [Gephi][1]. The layout can be used to lay out any network in which the nodes have some temporal ordering that can be expressed by their position on a horizontal axis. I created the layout plugin primarily to make it easy to create event graphs in [Gephi][1]. In event graphs nodes are events and the edges express relationships between events. For example, an event graph could be used to visualise a sequence of social activities, where the edges express which activities occurred in response to which other activities, thus visualising interactions. 

### Basic instructions
The Event Graph Layout plugin requires you to supply an *order* variable in the node list of your graph (in Gephi's data laboratory). The *order* variable should be of a numeric type (e.g., integer, float, double). From the layout's menu, the *order* variable should be selected from the corresponding drop-down menu. One can then activate the plugin, which will cause the nodes to be assigned a position on the horizontal axis of the draw screen that corresponds to the value of their *order* variable. The layout plugin keeps running until you explicitly deactivate it. This allows you to drag around nodes on the vertical axis of the layout without changing their position on the horizontal axis.

The *Scale of Order*, unsurprisingly, scales the *order* variable. By default, it the *order* variable is scaled by 10, which means that the value of the *order* variable of each node is multiplied by 10 before the variable is used to determine the position of nodes on the horizontal axis. Thus, the distance between the nodes in the event graph can be changed by changing the *Scale of Order*. The appropriate *Scale of order* depends on the values of your order variable. I suggest changing the value until you find the distance that you like.

The *Set Vertical Force* option activates a simplified force-based layout algorithm that only affects the position of nodes on the vertical axis of the layout (I based this algorithm on one of Gephi's Force Atlas algorithms). The effect is that nodes that are more tightly interconnected will remain relatively close to each other on the vertical axis, while nodes that are more loosely connected will drift apart. This makes it easier to have a more organised layout when you have a graph with many nodes.

The *Vertical Scale* variable determines how strong the forces are that keep interconnected nodes together and that push unconnected nodes away from each other. The higher the value, the stronger the forces. Finding appropriate values for your graphs usually requires you to play around a bit.

The *Strong Gravity Mode* is a convenience function that sets the attracting forces to a very high value. This typically means that all nodes end up on a straight line. This can be a good way to quickly reset your layout if nodes drifted too far apart (for example, after you set an insanely high *Vertical Scale*). 

The *Gravity* can also be manipulated with the *Gravity* variable. It can be useful to prevent your nodes from drifting apart too much. Just setting a lower *Vertical Scale* does not always work as well, because it affects both the attracting and the repelling forces at the same time. Instead, the *Gravity* variable only affects the tendency of variables to stay together. 

Occasionally, you'll create a graph where nodes are on the same position on the horizontal axis. When the *Vertical Force* is activated, these nodes may block each other when trying to change position. The *Jitter Tolerance* basically determines how much nodes are allowed to jump around when they bump into other nodes. By setting an appropriate *Jitter Tolerance*, you can make it easier for nodes in the some horizontal axis position to pass each other. If you set it too high, your graph will go nuts.

The *Threads* variable sets the number of threads that your processor can use to process the layout (if you have the possibility for multi-threaded processes). This can increase performance for large graphs, although I have never really needed this option myself.

The *Center* variable tries to keep your graph approximately in the center of the drawspace. In hindsight, I am not sure if this variable is really that useful. I never use it myself.


### Download instructions

You can download the Event Graph Layout plugin from Gephi's plugin menu. The source of the Event Graph Layout was merged into the [Gephi Plugins GitHub Repository][6].

## Bi-Dynamic Line Graphs
The idea to use Bi-Dynamic Line Graphs (BDLGs) to study network dynamics was introduced by Chiara Broccatelli, Martin Everett, and Johan Koskinen, in the article [*Temporal dynamics in covert networks*][2]. I think that this idea is very useful, and I wanted to have a more automated way to convert incidence matrices into BDLGs. In addition, I wanted to make it easy to import and visualise these graphs in [Gephi][1], since I already created the Event Graph Layout plugin (see above), which is very suitable for creating layouts for BDLGs. 

I thus created a simple tool that converts incidence matrices (imported as csv files) into BDLG matrices, and that also allows one to export the results as edge files and node files that can be imported into Gephi, and that provide everything you need to create a decent layout using the Event Graph Layout plugin. Again, all credits for the idea of BDLGs are for the authors mentioned above. I only wrote this tool. 

You can download the program using the following links:

   * [Download for Linux][3]
  
   * [Download for Windows][4]
   
The GitHub repository for the program can be found [here][5]

## Multi-dimensional scaling in Gephi
I wrote two plugins for [Gephi][1] that introduce the possibility to create layouts based on Multi-Dimensional Scaling (MDS) of nodes based on their path distances. This layout only works well with graphs that form one component, since distances between unconnected nodes are infinite, leading to somewhat awkward results.

### MDS metrics
The *MDS metrics* plugin calculates coordinates for the nodes in a graph based on their path distances, using the Stress Minimization algorithm included in the MDSJ library. The MDSJ library was created by Algorithmics Group of the University of Konstanz (see reference below). The MDSJ library was made available under the Creative Commons License "by-nc-sa" 3.0. The license is available at http://creativecommons.org/licenses/by-nc-sa/3.0/. I had no involvement in creating this library. I simply included it in the *MDS metrics* plugin to make use of its functionality. 

Both plugins discussed below can be downloaded from Gephi's plugin menu. The source code for the plugins can be found in [Gephi's Github plugin repository][6].

#### Basic Instructions
After installing the *MDS metrics* plugin from the Gephi plugins menu, it will appear in the list of statistics algorithms. Using the plugin will open a dialog where you can:

 1. Select the number of dimensions in which the nodes are allowed to have coordinates (of course, in Gephi you can only actually use two dimensions to create a layout). 
 2. Set how edge weights should be treated (whether or not they should be included in the determination of distances, and, if yes, whether they should be interpreted as a degree of closeness, or as a degree of distance). I have to honestly admit that this does not work very well for most graphs.
 3. How the distances should be treated in the calculation of stress values. You can downweight large distances an upweight small ones, or weigh all distances equally. See the reference below for details.
 
After running the algorithm with the selected settings, a dialog will appear that shows some basic results, the most important one of which is the stress of the MDS configuration (coordinate system) that the plugin converged on. Stress values can be understood to indicate the fit of the resulting configuration with the actual distances that we put in. We will typically be looking for stress values somewhere below 0.1. Configurations that have stress values above that will not actually reflect the distances between the nodes very well. Of course, this is all a bit arbitrary, and with larger graphs we will usually have to be satisfied with somewhat larger stress values. 

The coordinates of the resulting configuration are automatically written to the node list in Gephi's data laboratory. These coordinates can then be fed into the *MDS Layout* plugin.

#### Reference
lgorithmics Group. MDSJ: Java Library for Multidimensional Scaling (Version 0.2). Available at http://www.inf.uni-konstanz.de/algo/software/mdsj/. University of Konstanz, 2009.

### MDS Layout
The *MDS Layout* plugin is a very simple layout plugin that takes coordinates in two dimensions (e.g., a horizontal dimension and a vertical dimension), and then lays out nodes based on the values of their coordinates in these dimensions. I created the plugin to allow you to create a layout based on coordinates produces by the *MDS Metrics* plugin, but you can of course also use coordinates that were produces in another way.

#### Basic instructions
The *Dimension 1* and *Dimension 2* drop-down menus can be used to select the dimensions of the coordinate system that you want to use for your layout. This assumes that your nodes have coordinate variables (which should be included in the node list of Gephi's data laboratory). The *Network Scale* variable can be used to scale the graph, that is, to increase or decrease the overall size of the graph.

## Lineage plugin for Gephi
The lineage plugin for Gephi can be used to highlight ancestors and descendants of a single node in a directed network. After downloading the plugin from Gephi's plugin menu, it will appear the in the list of statistics algorithms. When using the plugin, a dialog will appear that requires you to identify a node by typing its id into an entry field (Use the *id*, not the *label*!). 

The plugin will identify all ancestors and descendants of the selected node (we refer to this as the origin), as well as the distance of any of these ancestors or descendants from the origin. Several variables are automatically added to the node list in the data laboratory:

  * **Lineage**: A string variable that indicates for each node whether it is the *Origin*, an *Ancestor*, a *Descendant* or *Unrelated*. This can be used for easy partition colouring. 
  * **IsOrigin**: A boolean variable that indicates whether a node is the *Origin*.
  * **IsAncestor**: A boolean variable that indicates whether a node is an *Ancestor* of the *Origin*.
  * **DistanceAncestor**: An integer variable that indicates the path distance of *Ancestor* nodes from the *Origin* node. These distances are always expressed with negative values.
  * **IsDescendant**: A boolean variable that indicates whether a node is a *Descendant* of the *Origin*.
  * **DistanceDescendant**: An integer variable that indicates the path distance of *Descendant* nodes from the *Origin* node. These distances are always expressed with positive values.
  
The plugin can be downloaded from Gephi's plugin menu. The source code of the plugin can be found at the [Github repository for plugins][6].

## Dynamic Networks
One of my first stand-alone programs. This will be made more or less obsolete with the release of Q-SoPrA, although Q-SoPrA approaches the same problems a bit differently. 

Dynamic Networks is intended as a tool to be used alongside [Gephi][1]. The tool works with incidence matrices that are saved as csv files. The tool assumes that the columns of the matrix represent events that are put in temporal order. An example is offered below.

|       |Event1|Event2|Event3|Event4|
|-------|------|------|------|------|
|Actor A|  1   |   0  |   1  |  0   |
|Actor B|  0   |   1  |   0  |  1   |
|Actor C|  1   |   1  |   0  |  0   |

The tool can be used to convert the incidence matrix into various types of edge lists that can be imported into Gephi. For example, it is possible to take different fragments of the incidence matrix (overlapping or not), and convert this to a dynamic network that can be visualised using Gephi's dynamic visualisation options.

### Downloads
You can find download links for the binaries here:

 * [Download for Linux][11]
 * [Download for Mac OS X][12]
 * [Download for Windows][13]
 
See [this Github repository][14] for the source code and additional details.


## CSV parser for Gephi
I wrote this a while ago, and it will be mostly obsolete as soon as Q-SoPrA is ready. I created this program for personal use, to convert csv-output from event sequence datasets into nodes and edges files that can be imported in Gephi, and that are more easily parsed by Neo4J csv-import functions.

An appropriate input-file has the following characteristics:

 * It has a header that indicates different types of entities listed in the file. There may be multiple values within a cell, as long as these are separated with an appropriate delimiter. Make sure that this delimiter is different from the one used to separate the columns of the file.
 * The program requires the user to select an input file (csv), as well as the symbol that is used as column delimiter in the file. The user also has the option to select a second delimiter, in case the user has put multiple values in certain columns, and wants to separate these. After importing the file, the program can be configured to export a nodes list and edge list (see basic user instructions).

The purpose of this program is highly specific, but there may be things in the code that are useful for others. 

### Downloads

The download links for the binaries of the csv-parser can be found here:

 * [Download Linux version][8]
 * [Download Windows version][9]
 
See [this Github Repository][10] for the source code, and for more detailed instructions on use.

## Scripts for R
I wrote several scripts for R, mostly focused on stuff related to directed a-cyclic networks or event graphs. The scripts can be found at this [GitHub Repository][7].



[1]: http://www.gephi.org
[2]: http://journals.sagepub.com/doi/abs/10.1177/2059799115622766
[3]: https://github.com/WouterSpekkink/BDLG_Matrix_Converter/releases/download/1.0.0/BDLG_Matrix_Converter_Linux.tar.gz
[4]: https://github.com/WouterSpekkink/BDLG_Matrix_Converter/releases/download/1.0.0/BDLG_Matrix_Converter_win.zip)
[5]: https://github.com/WouterSpekkink/BDLG_Matrix_Converter
[6]: https://github.com/gephi/gephi-plugins/tree/master-forge
[7]: https://github.com/WouterSpekkink/R-Scripts
[8]: https://github.com/WouterSpekkink/csv_parser_gephi/releases/download/1.0/Csv_Parser_Linux.tar.gz
[9]: https://github.com/WouterSpekkink/csv_parser_gephi/releases/download/1.0/Csv_parser_Win.zip
[10]: https://github.com/WouterSpekkink/csv_parser_gephi
[11]: https://github.com/WouterSpekkink/DynamicNetworks/releases/download/1.0/DynamicNetworksLinux.tar.gz
[12]: https://github.com/WouterSpekkink/DynamicNetworks/releases/download/1.0/DynamicNetworks-Mac.zip)
[13]: https://github.com/WouterSpekkink/DynamicNetworks/releases/download/1.0/DynamicNetworks-Windows.zip)
[14]: https://github.com/WouterSpekkink/DynamicNetworks
