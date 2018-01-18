---
layout: post
title:  "QSqlTableModels, booleans and check boxes"
date:   2018-01-18 09:07:00 +0000
categories: Software Q-Sopra Technical C++ Qt
tags: Software Q-SoPrA Technical C++ Qt
---

## Interfacing with sql databases with Qt5
This post is on an issue that I struggled with very recently, while working on Q-SoPrA. What I wanted to achieve was relatively simple: I wanted to have tables that fetch data from sql databases, and in which one column shows check boxes to set/unset a boolean variable. The screenshot below shows an example. 

<br><br>![Table with check boxes](/assets/posts/qsqltablemodels-booleans-and-check-boxes/table.png){: .center-image}<br><br>

What you see in the screen shot is [QTableView][9] widget that shows data that it fetches from [QSqlTableModel][5] that interfaces with a table of a [sqlite][2] database. This post is about how to create the interactive check boxes shown in the right-most column. There are two main hurdles in getting the [QTableView][9] widget to work with check boxes:
 1. Sqlite databases don't actually support boolean variables. You can use an integer variable to 'simulate' a boolean variable by setting it to `0`(for `false`) or `1` (for `true`), but you will of course need to do a bit of extra work to make the [QSqlTableModel][5] properly treat it as a boolean in read & write operations. This can be done by sub-classing the [QSqlTableModel][5], and re-implementing its [data()][12] and [setData()][13] functions, as suggested [here][11].
 2. But if you do that, you'll have a quite ugly looking table, since the check boxes are all left-aligned in their column, with no obvious way to change that. In this case, the only solution seems to be to use a [QStyledItemDelegate][16] to handle the `paint()` function that determines how the column is visualised. A somewhat outdated example of how to do that is offered in the [Qt FAQ][17].
 
 For good results in my use-case, where I want to have check boxes that (1) are able to handle a 'pretend boolean' variable (a boolean that is actually an integer) from a sqlite database, and (2) are not all aligned to the extreme left of their column, you have to combine the two solutions mentioned above. I haven't really encountered a worked out example of this combination, which is why I decided to provide one in this post.

In the below, I briefly outline how you could achieve a result like the one shown in the screenshot above.

<details>
<summary>In case you are not familiar with Qt5...</summary>
<blockquote>
<a href="https://www.qt.io/">Qt5</a> is a popular library for C++ that can be used for the development of Graphical User Interfaces (GUIs). It also includes a module that allows for easy interfacing of your program with sql databases. I make heavy use of the Qt5 library for the development of Q-SoPrA, including the possibilities it offers for interfacing with <a href="https://sqlite.org/">sqlite</a> databases. Qt5 comes packed with a number of great <a href="https://doc.qt.io/qt-5.10/sql-programming.html">sql database classes</a>, such as <a href="https://doc.qt.io/qt-5.10/qsqldatabase.html">QSqlDatabase</a>, <a href="https://doc.qt.io/qt-5.10/qsqlquerymodel.html">QSqlQueryModel</a>, <a href="https://doc.qt.io/qt-5.10/qsqltablemodel.html">QSqlTableModel</a>, <a href="https://doc.qt.io/qt-5.10/qsqlrelationaltablemodel.html">QSqlRelationalTableModel</a>, and <a href="https://doc.qt.io/qt-5.10/qsqlquery.html">QSqlQuery</a>. <br><br>For the development of Q-SoPrA, I make use of most of all these classes, although often a sub-classed version of them in which I re-implemented some of their member functions. <br><br>I think a very common setup for interfacing with sql databases is to have a <a href="https://doc.qt.io/qt-5.10/qsqltablemodel.html">QSqlTableModel</a> (or <a href="https://doc.qt.io/qt-5.10/qsqlrelationaltablemodel.html">QSqlRelationalTableModel</a>) that fetches data from a table in your sql database, a <a href="https://doc.qt.io/qt-5.10/qsortfilterproxymodel.html">QSortFilterProxyModel</a> that reads from the QSqlTableModel and acts as a 'filtering layer', and a <a href="https://doc.qt.io/qt-5.10/qtableview.html">QTableView</a> that reads from the filter and displays the results on the user's screen. This is also the kind of setup that I use for several widgets that I included in Q-SoPrA, although I typically subclass the QSqlTableModel / QSqlRelationalTableModel to change some of the ways in which the data is presented to the user (such as tool tip behaviour). 
</blockquote>
</details><br>

## Sub-classing QSqlTableModel
First, we actually need to sub-class the [QSqlTableModel][5] so that we can re-implement its `data()` and `setData()` member functions, and make our 'pretend boolean' (an integer set to `0` or `1`) behave as an actual boolean (I believe it was [this discussion][11] that led me to this insight). The [data()][12] and [setData()][13] are used to read data from, and write data to the sql table with which the QSqlTableModel is interfacing. We will want most of their behaviour to remain the same, but we want special treatment for the simulated boolean variable. 

We will make a sub-classed version of [QSqlTableModel][5] that is specific for the sql table that we want to interact with. In my case, this is a table which has our 'pretend boolean' stored in the seventh column. Thus, one thing I can do is to re-implement the behaviour of the two functions specifically for the data that is included in the seventh column of the table that is being accessed. This is of course easily achieved with an if-statement. 

We should also keep in mind that in the QSqlTableModel, data are stored under different 'roles' (see an overview of these roles [here][14]. For example, data stored under the `Qt::DisplayRole` are the data that are actually shown to the user, data stored under the `Qt::ToolTipRole` are the data that are shown to the user when (s)he hovers his/her mouse over an entry in the table, and data stored under the `Qt::EditRole` are the data that the user can manipulate in an editor. In the [list of roles][14] you will also find the `Qt::CheckStateRole`, and this is the role that we want to (re-)implement for our 'pretend boolean'. 

Let's start with our `data()` function. Basically, we want it to return the state `Qt::Checked` whenever:
	1. we are reading the seventh column of our sql table (which contains the 'pretend boolean'),
	2. we specifically want to return the `Qt::CheckState` role for the index currently being read,
	3. the value of the integer at this index is set to `1`.
	
You'll probably have guessed that we also want the function to return the state Qt::Unchecked whenever:
	1. we are reading the seventh column of our sql table (which contains the 'pretend boolean'),
	2. we specifically want to return the `Qt::CheckState` role for the index currently being read,
	3. the value of the integer at this index is set to `0`.

Moreover, in this case, we *only* want to return data under the `Qt::CheckState` role, because this is the only way in which the user needs to interact with the 'pretend boolean' variable. 

Below I have included a snippet of code that achieves this. The re-implemented `data()` function in this snippet actually does a bit more than that, but I think it is good to show the re-implemented behaviour 'in context'. The version of the `data()` function shown below also handles cases where it is supposed to return data under the `Qt::ToolTipRole`. I re-implemented this function earlier in order to simply have it show a tool tip with the data that is included in the cell that the user is hovering his/her mouse over. 

<details>
<summary><b>Click to here to see the re-implemented data() function</b></summary>
{% highlight c++ %}
QVariant EventTableModel::data(const QModelIndex &index, int role) const {
  if (index.column() == 7) { // This is always the column with the boolean variable
    if (role == Qt::CheckStateRole) { // Only do the below when we are setting the checkbox.
      // We want to fetch the state of the boolean from the sql table.
      QSqlQuery *query = new QSqlQuery;
      int order = index.row() + 1;
      query->prepare("SELECT mark FROM incidents WHERE ch_order = :order");
      query->bindValue(":order", order);
      query->exec();
      query->first();
      int mark = query->value(0).toInt();
      // Return the appropriate check state based on the state of mar.
      if (mark == 1) {
	return Qt::Checked;
      } else if (mark == 0) {
	return Qt::Unchecked;
      }
    } else {
      /*
	We return an empty variant in all other cases. This is to prevent, for example,
	that we also see a '0' or '1' in the same column.
      */
      return QVariant();
    }
    // Only do the below if we want to fetch a tool tip.
  } else if (role == Qt::ToolTipRole) {
    // I just want the tool tip to show the data in the column.
    const QString original = QSqlTableModel::data(index, Qt::DisplayRole).toString();
    QString toolTip = breakString(original); // breakString() breaks the text in smaller lines.
    return toolTip;
  } else {
    /* 
       In all other cases, we want the default behaviour of this function. 
       This can be done easily by returning the default version of the function,
       rather than the re-implemented version we have here.
    */
    return QSqlTableModel::data(index, role);
  }
  return QVariant();
}
{% endhighlight %}
</details> <br>


So, the basic structure of this function is quite simple. There are three different situations that it considers in turn:

 1. Are we reading the seventh column of our table? If no, then...
 2. are we trying to return data under the 'Qt::ToolTipRole'? If no, then...
 3. in all other situations, just revert to default behaviour.
 
The first two situations are 'special cases', the first of which deals with the boolean variable that is the topic of this post, and the second of which deals with custom tool tip behaviour. The third situation can be read as 'in all other cases.' We want our function to behave like it normally would 'in all other cases', which is why we return `QSqlTableModel::data(index, role)`, basically calling the original version of the `data()` function, instead of our re-implemented one. 

<details>
<summary><b>Click to here to see the re-implemented setData() function</b></summary>
{% highlight c++ %}

{% endhighlight %}
</details> <br>



[1]: https://doc.qt.io/qt-5.10/sql-programming.html
[2]: https://sqlite.org/
[3]: https://doc.qt.io/qt-5.10/qsqldatabase.html
[4]: https://doc.qt.io/qt-5.10/qsqlquerymodel.html
[5]: https://doc.qt.io/qt-5.10/qsqltablemodel.html
[6]: https://doc.qt.io/qt-5.10/qsqlrelationaltablemodel.html
[7]: https://doc.qt.io/qt-5.10/qsqlquery.html
[8]: http://doc.qt.io/archives/qt-4.8/qsortfilterproxymodel.html
[9]: http://doc.qt.io/archives/qt-4.8/qtableview.html
[10]: {{ site.baseurl }}{%post_url 2017-12-26-getting-started-with-qsopra %}
[11]: http://www.qtcentre.org/threads/12008-QSqlTableModel-and-checkboxes-with-SQLite-database
[12]: http://doc.qt.io/qt-5/qsqltablemodel.html#data
[13]: http://doc.qt.io/qt-5/qsqltablemodel.html#setData
[14]: http://doc.qt.io/qt-5/qt.html#ItemDataRole-enum
[15]: https://www.qt.io/
[16]: http://doc.qt.io/qt-5/qstyleditemdelegate.html
[17]: https://wiki.qt.io/Technical_FAQ#How_can_I_align_the_checkboxes_in_a_view.3F
