---
layout: post
title:  "QSqlTableModels, booleans and check boxes"
date:   2018-01-18 09:07:00 +0000
categories: Software Q-Sopra Technical C++ Qt
tags: Software Q-SoPrA Technical C++ Qt
---

## Interfacing with sql databases with Qt5
I recently stumbled upon a somewhat annoying, not very well documented issue with the [QSql module][1] of the Qt5 library: It turns out to be relatively difficult to use (nice looking) check boxes to interact with boolean data included in sql databases. I found a solution that works great for me by tying together different other solutions that I found on the Web, and I thought I would outline it here.

In case you are not familiar with Qt5: It is a popular library for C++ that can be used for the development of Graphical User Interfaces (GUIs). It also includes a module that allows for easy interfacing of your program with sql databases. I make heavy use of the Qt5 library for the development of Q-SoPrA. The GUI of Q-SoPrA was developed entirely with Qt5, and I also use the Qt5 library to interface with the [sqlite][2] databases that Q-SoPrA depends on. 

Qt5 comes packed with a number of great database classes, such as [QSqlDatabase][3], [QSqlQueryModel][4], [QSqlTableModel][5], [QSqlRelationalTableModel][6], and [QSqlQuery][7]. For the development of Q-SoPrA, I make use of most of all these classes, although often a sub-classed version of them in which I re-implemented some of their member functions.

I think a very common setup for interfacing with sql databases is to have a [QSqlTableModel][5] (or [QSqlRelationalTableModel][6]) that fetches data from a table in your sql database, a [QSortFilterProxyModel][8] that reads from the QSqlTableModel and acts as a 'filtering layer', and a [QTableView][9] that reads from the filter and displays the results on the user's screen. This is also the kind of setup that I use for several widgets that I included in Q-SoPrA, although I typically subclass the QSqlTableModel / QSqlRelationalTableModel to change some of the ways in which the data is presented to the user (such as tip behaviour). 

## Boolean variables in the sql database
Some of the tables in the databases that Q-SoPrA relies on have boolean variables. Well, actually, sqlite databases don't have a boolean data type, but it is easy to 'simulate' one by using integers. Whenever I want to set a boolean variable to `false`, I use an integer that is set to `0`, and when I want to set a boolean variable to `true`, I use an integer that is set to `1`. In Q-SoPrA these variables are usually only important for what happens in the background when Q-SoPrA writes to, or reads from one of the sql tables, but in one case the boolean is something that I want the user to interact with. I allow the user of the program to mark certain entries in his/her data set (for example, to mark entry that should be revisited at some point). Whenever the user marks or unmarks a certain entry, in the background a boolean variable is set to `true` / `false`. 

One of the interfaces in which I want to allow the user to do this, is the table of Q-SoPrA's *Data Widget* ([see this earlier post for more background information on this widget][10]). This is the widget where the user will enter his/her data, and I found it would be useful to quickly mark entries to help the user remember that something must be done about them. The most obvious way to do this, in my opinion, is to include a check box for each entry which can be set to checked (marked) or unchecked (unmarked).

However, the [QTableView][9] that the user interacts with won't automatically recognise a boolean variable and show it as a check box. Also, as I explained, the variable is not an actual boolean in the first place, but an integer. Thus, by default the QTableView will just show the number `0` or `1`, depending on the current state of the variable. To make the variable appear as a check box, there are a few things we need to do.

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
