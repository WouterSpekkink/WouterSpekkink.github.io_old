---
layout: post
title:  "QSqlTableModels, booleans and check boxes"
date:   2018-01-19 09:07:00 +0000
categories: Software Q-Sopra Technical C++ Qt
tags: Software Q-SoPrA Technical C++ Qt
---

## Interfacing with sql databases with Qt5
This post is on an issue that I struggled with very recently, while working on Q-SoPrA. What I wanted to achieve was relatively simple: I wanted to have tables that fetch data from sql databases, and in which one column shows check boxes to set/unset a boolean variable. The screenshot below shows an example. 

<br><br>[![Table with check boxes](/assets/posts/qsqltablemodels-booleans-and-check-boxes/table.png){: .center-image}](/assets/posts/qsqltablemodels-booleans-and-check-boxes/table.png)<br><br>

What you see in the screen shot is [QTableView][9] widget that shows data that it fetches from [QSqlTableModel][5] that interfaces with a table of a [sqlite][2] database. This post is about how to create the interactive check boxes shown in the right-most column. There are two main hurdles in getting the [QTableView][9] widget to work with check boxes:
 1. Sqlite databases don't actually support boolean variables. You can use an integer variable to 'simulate' a boolean variable by setting it to `0`(for `false`) or `1` (for `true`), but you will of course need to do a bit of extra work to make the [QSqlTableModel][5] properly treat it as a boolean (or something that can be switched on or off) in read & write operations. This can be done by sub-classing the [QSqlTableModel][5], and re-implementing its [flags()][18], [data()][12] and [setData()][13] functions, as suggested [here][11].
 2. But if you do that, you'll have a quite ugly looking table, since the check boxes are all left-aligned in their column, with no obvious way to change that. In this case, the only solution seems to be to use a [QStyledItemDelegate][16] to handle the `paint()` function that determines how the column is visualised. A somewhat outdated example of how to do that is offered in the [Qt FAQ][17].
 
For good results in my use-case, where I want to have check boxes that (1) are able to handle a 'pretend boolean' variable (a boolean that is actually an integer) from a sqlite database, and (2) are not all aligned to the extreme left of their column, you have to combine the two solutions mentioned above. I haven't really encountered a worked out example of this combination, which is why I decided to provide one in this post.

In the below, I briefly outline how you could achieve a result like the one shown in the screenshot above.

<details>
<summary>In case you are not familiar with Qt5...</summary>
<blockquote>
<a href="https://www.qt.io/">Qt5</a> is a popular library for C++ that can be used for the development of Graphical User Interfaces (GUIs). It also includes a module that allows for easy interfacing of your program with sql databases. I make heavy use of the Qt5 library for the development of Q-SoPrA, including the possibilities it offers for interfacing with <a href="https://sqlite.org/">sqlite</a> databases. Qt5 comes packed with a number of great <a href="https://doc.qt.io/qt-5.10/sql-programming.html">sql database classes</a>, such as <a href="https://doc.qt.io/qt-5.10/qsqldatabase.html">QSqlDatabase</a>, <a href="https://doc.qt.io/qt-5.10/qsqlquerymodel.html">QSqlQueryModel</a>, <a href="https://doc.qt.io/qt-5.10/qsqltablemodel.html">QSqlTableModel</a>, <a href="https://doc.qt.io/qt-5.10/qsqlrelationaltablemodel.html">QSqlRelationalTableModel</a>, and <a href="https://doc.qt.io/qt-5.10/qsqlquery.html">QSqlQuery</a>. <br><br>For the development of Q-SoPrA, I make use of most of all these classes, although often a sub-classed version of them in which I re-implemented some of their member functions. <br><br>I think a very common setup for interfacing with sql databases is to have a <a href="https://doc.qt.io/qt-5.10/qsqltablemodel.html">QSqlTableModel</a> (or <a href="https://doc.qt.io/qt-5.10/qsqlrelationaltablemodel.html">QSqlRelationalTableModel</a>) that fetches data from a table in your sql database, a <a href="https://doc.qt.io/qt-5.10/qsortfilterproxymodel.html">QSortFilterProxyModel</a> that reads from the QSqlTableModel and acts as a 'filtering layer', and a <a href="https://doc.qt.io/qt-5.10/qtableview.html">QTableView</a> that reads from the filter and displays the results on the user's screen. This is also the kind of setup that I use for several widgets that I included in Q-SoPrA, although I typically subclass the QSqlTableModel / QSqlRelationalTableModel to change some of the ways in which the data is presented to the user (such as tool tip behaviour). 
</blockquote>
</details><br>

## An assumption about your 'pretend boolean' variable
In the below I make one main assumption about your 'pretend boolean' variable, which is that this variable is stored in one of the tables of your sql database as an integer, and that you programmatically set this integer to `0` or `1` whenever necessary. 

There are of course multiple ways to achieve this. I tend to use the [QSqlQuery][7] class for reading and writing interactions with sql databases. Basically, this class allows you to write and execute an sql query like you normally would. In the snippet below, I have included a fictional example where I set our 'pretend boolean' variable to `0` or `1` depending on some conditional.

{% highlight c++ %}
/*
  Assume that we have a column in our sql table in which we store a variable
  that corresponds to the sorting order of the table's rows. 
  We call this variable "order", and we set the lowest value of this variable
  to be 0. 
  If we consistently sort the sql table based on this variable, its value
  will always be the row number of the corresponding entry + 1.
*/
int order = tableView->currentIndex().row() + 1; 
int mark = 0;
/* 
  We have a 'pretend boolean' variable that is called "mark". This is an integer
  that is set either to "0" or to "1".
  Now assume that we have some conditional that determines what state should be
  recorded for "mark". We initialise "mark" as "0", and change its value to "1" 
  if our condition holds true.
*/
if (myCondition == true) {
   mark = 1;
}
// Then we create an QSqlQuery object to record the result in our sql table.
QSqlQuery *query = new QSqlQuery; 
/* 
  In our sql table, the order variable is recorded in a column called "ch_order".
  There is a good reason for this: order is also a key word in sql query language,
  so it cannot be used as a name for table columns.
*/
query->prepare("UPDATE myTable SET mark = :mark, WHERE ch_order = :order");
// We need to bind our order variable to the query.
query->bindValue(":order", order); 
// And then we can execute it. This will write the data to the table.
query->exec(); 
delete query; // Memory management

{% endhighlight %}

Reading the state of our 'pretend boolean' can also be done with an [QSqlQuery][7] object. Say that we want to read the state of this variable for all rows of our sql table. The following snippet would achieve something like that.

{% highlight c++ %}
// We create our QSqlQuery object.
QSqlQuery *query = new QSqlQuery;
// No need to bind values to our query in this case, so we can immediately execute.
query->exec("SELECT mark FROM myTable");
/*
  The results are now store in the query object, 
  and we can use the next() method to iterate through the results.
*/
while (query->next()) {
  // The query->value() method returns a QVariant, so we need to convert it to an integer.
  int currentMark = query->value(0).toInt();
  // Do something with the variable. In this case we just write it to the console output.
  qDebug() << currentMark;
}
delete query; // Memory management
{% endhighlight %}

## Sub-classing QSqlTableModel
Now, if we want user interaction with our 'pretend boolean' variable to be handled by a check box, we need to sub-class the [QSqlTableModel][5] that interfaces between the sql table, and the [QTableView][9] that visualises the data for the user. More specifically, we need to re-implement the `flags()`, `data()` and `setData()` member functions, and make our 'pretend boolean' (an integer set to `0` or `1`) behave as an item that can be checked and unchecked (I believe it was [this discussion][11] that led me to this insight). The [flags()][18] function basically determines how items recorded in the [QSqlTableModel][5] can be manipulated (see [this list][19] of possible flags). We want to re-implement this function to make sure that items that correspond to our 'pretend boolean' can be checked or unchecked by the user. This can be achieved quite easily:

{% highlight c++ %}
/* 
   To make sure that the column with the pretend boolean is recognised
   as a 'checkable' column, we need to re-implement the flags() function
   and create a special case for the target column.
*/
Qt::ItemFlags EventTableModel::flags(const QModelIndex & index) const {
  // Column 7 always records the mark variable (our boolean).
  if (index.column() == 7) {
    // Make sure that this item is checkable.
    return QSqlTableModel::flags(index) | Qt::ItemIsUserCheckable;
  }
  // Default behaviour in all other cases.
  return QSqlTableModel::flags(index);
}
{% endhighlight %}

In the code snippet above, we make the assumption that our 'pretend boolean' is always recorded in the seventh column of our sql table. This may of course be different in your case. All we do is to check whether the current index being accessed exists in the seventh column. If yes, then we communicate to the program that the item at this index should be checkable by the user. If no, then we revert to the default behaviour of the `QSqlTableModel::flags()` member function. 

Our sub-classed version of [QSqlTableModel][5] still won't understand how to handle our 'pretend boolean' properly. We also need to re-implement the `data()` and `setData()` functions first. The [data()][12] and [setData()][13] are used to read data from, and write data to the sql table with which the QSqlTableModel is interfacing. 

We should keep in mind here that in the QSqlTableModel, data are stored under different 'roles' (see an overview of these roles [here][14]). For example, data stored under the `Qt::DisplayRole` are the data that are actually shown to the user in the [QTableView][9], data stored under the `Qt::ToolTipRole` are the data that are shown when the user hovers his/her mouse cursor over an entry in the table, and data stored under the `Qt::EditRole` are the data that the user can manipulate in an editor. In the [list of roles][14] you will also find the `Qt::CheckStateRole`, and this is the role that we want to (re-)implement for our 'pretend boolean'. 

Another thing we should keep in mind is that we don't want *all* data in our sql table to be treated as 'pretend booleans.' We will want to keep the default behaviour of the `data()` and `setData()` functions in most cases. We can do that in the same way by as we did with the `flags()` function: We create a special case for the column that holds our `pretend boolean` variable, and we revert to the default implementation of `QSqlTableModel::data()` and `QSqlTableModel::SetData()` in all other cases. 

<blockquote>Actually, in the example below we have one other exception, which is the special case in which the user is hovering the mouse cursor over a cell, which will cause the QSqlTableModel to return data under the Qt::ToolTipRole.</blockquote>

Let's start with our `data()` function. See a snippet with its re-implemented version below.

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
  return QVariant(); // This prevents a compiler warning.
}
{% endhighlight %}

So, the basic structure of this function is quite simple. We first check if the column of the sql table that is being accessed is the column with our `pretend boolean`. If yes, then we check whether the data is being access under the 'Qt::CheckStateRole'. If the answer is yes again, we run a block of code that reads the current value of `mark` in the sql table (which can be `0` or `1`, assuming that we have been consistent in our implementation of the interaction with this column of the sql table), and then returns the value `Qt::Checked` if the value read is `1`, or returns the value `Qt::Unchecked` if the value read is `0`. 

There are a few other situations that the re-implemented function handles. If we are accessing data in the column with our `pretend boolean`, but we are not accessing the data under the `Qt::CheckState` role, then the function simply returns an empty QVariant(), effectively returning nothing. I did this to make sure that the corresponding column in the [QTableView][9] only shows a check box that visualises the current check state, and nothing else. If we are access data in any other column, the function first checks whether we are accessing data under the `Qt::ToolTipRole`. If yes, then we treat it as another special case, in which the user gets shown a tool tip that simply contains the visible contents of the cell currently being hovered over with the mouse cursor. If we are not accessing the data under the `Qt::ToolTipRole` (in all other cases), we just revert to the default implementation of the `QSqlTableModel::data()` function. 

If you have re-implemented the `data()` function in this way, then your [QTableView][9] should already show check boxes in the corresponding column. So far, so good. However, there are still a few problems. One problem is that the check boxes are aligned to the extreme left of the column, which makes the table look ugly. We will deal with this later. A more urgent problem is that checking / unchecking the check boxes won't actually do anything meaningful with the underlying data unless we also re-implement the `setData()` function. Let's do that next.

{% highlight c++ %}

bool EventTableModel::setData(const QModelIndex & index,
			      const QVariant & value, int role) {
  /* 
     Let's check whether the selected column is the column with our boolean variable
     (always column 7), and whether we are trying to set data under the 
     Qt::CheckStateRole.
  */
  if (index.column() == 7 && role == Qt::CheckStateRole) {
    // Writing the data when the check box is set to checked.
    if (value == Qt::Checked) {
      QSqlQuery *query = new QSqlQuery;
      int order = index.row() + 1;
      query->prepare("UPDATE incidents SET mark = 1 WHERE ch_order = :order");
      query->bindValue(":order", order);
      query->exec();
      delete query;
      return true;
      // Writing the data when the check box is set to unchecked
    } else if (value == Qt::Unchecked) {
      QSqlQuery *query = new QSqlQuery;
      int order = index.row() + 1;
      query->prepare("UPDATE incidents SET mark = 0 WHERE ch_order = :order");
      query->bindValue(":order", order);
      query->exec();
      delete query;
      return true;
    }
  }
  // In all other situations revert to default behaviour.
  return QSqlTableModel::setData(index, value, role);
}

{% endhighlight %}

We have a similar kind of check to the one we had with the `setData()` function: We check whether we column we are writing to is the column that contains our `pretend boolean` variable. In this case, we check at the same time whether we are trying to write data under the `Qt::CheckStateRole`. If yes, we check whether the corresponding check box was set to checked or to unchecked, and write a `1` to the sql table in the first case, or a `0` in the latter. In all other situations we simply refer to the default implementation of the `QSqlTableModel::setData()` function.

After re-implementing the `data()` function this way, checking / unchecking the check boxes in our [QTableView][9] will actually do something meaningful with the data in the underlying sql table. 

If you don't care about the alignment of the check boxes in their corresponding column of the [QTableView][9], then you're done. However, I think that the table looks much nicer with the check boxes aligned to the centre of their column. How to achieve this is what I will discuss next.

## Styling the column with the check boxes
It turns out that the only way to align the check boxes to the centre of their column is to use a [QStyledItemDelegate][16]. Well, there is [another approach that uses layouts][20], but that only works if you (1) manually append another column to your [QSqlTableModel][5], (2) explicitly create `QCheckBox` objects, (3) assign these to a parent `QWidget` object, (4) to which you then apply a centralised layout. This is quite an expensive procedure that can significantly reduce the performance of your program, and it only works well if you don't update your [QSqlTableModel][5] frequently. 

<blockquote>
This is because every time that you update your QSqlTableModel the manually appended column will disappear. You can of course solve this by writing a function that creates the extra column, and fills it with your manually created check boxes, and run this function whenever the QSqlTableModel's select function gets called. However, this will slow down your program quite a bit, and I also found some other drawbacks to this approach that are a bit outside the scope of this discussion.
</blockquote>

The approach that uses a [QStyledItemDelegate][16] is actually what is recommended in [Qt's FAQ][11] section. An example code snippet is offered there as well, although it is a bit outdated (it uses some functions that were deprecated in Qt5). Essentially, what you are required to do is to create your own sub-class of [QStyledItemDelegate][16], and re-implement its `paint()` and `editorEvent()` functions. The `paint()` function determines how the check box is visualised to the user, and the `editorEvent()` function determines how the user can interact with the check box.

I included the snippet with my slightly adapted version of the re-implemented functions below. I am not going to pretend that I grasp every detail of what happens in these functions. I mostly different replaced some of the deprecated functions in the example offered in [Qt's FAQ][11].

{% highlight c++ %}
/* 
   This is a slightly adapted version of the example that is provided here:
   https://wiki.qt.io/Technical_FAQ#How_can_I_align_the_checkboxes_in_a_view.3F
*/

void CheckBoxDelegate::paint (QPainter * painter, const QStyleOptionViewItem & option,
	    const QModelIndex & index ) const {
  QStyleOptionViewItem viewItemOption(option);
  // Only do this if we are accessing the column with boolean variables.
  if (index.column() == 7) {
    // This basically changes the rectangle in which the check box is drawn.
    const int textMargin = QApplication::style()->pixelMetric(QStyle::PM_FocusFrameHMargin) + 1;
    QRect newRect = QStyle::alignedRect(option.direction, Qt::AlignCenter,
					QSize(option.decorationSize.width() +
					      5,option.decorationSize.height()),
					QRect(option.rect.x() + textMargin, option.rect.y(),
					      option.rect.width() -
					      (2 * textMargin), option.rect.height()));
    viewItemOption.rect = newRect;
  }
  // Draw the check box using the new rectangle.
  QStyledItemDelegate::paint(painter, viewItemOption, index);
}

bool CheckBoxDelegate::editorEvent(QEvent *event, QAbstractItemModel *model,
			 const QStyleOptionViewItem &option,
			 const QModelIndex &index) {
  Q_ASSERT(event);
  Q_ASSERT(model);

  // make sure that the item is checkable
  Qt::ItemFlags flags = model->flags(index);
  if (!(flags & Qt::ItemIsUserCheckable) || !(flags & Qt::ItemIsEnabled)) {
    return false;
  }

  // make sure that we have a check state
  QVariant value = index.data(Qt::CheckStateRole);
  if (!value.isValid()) {
    return false;
  }

  // make sure that we have the right event type
  if (event->type() == QEvent::MouseButtonRelease) {
    const int textMargin = QApplication::style()->pixelMetric(QStyle::PM_FocusFrameHMargin) + 1;
    QRect checkRect = QStyle::alignedRect(option.direction, Qt::AlignCenter,
					  option.decorationSize,
					  QRect(option.rect.x() + (2 * textMargin), option.rect.y(),
						option.rect.width() - (2 * textMargin),
						option.rect.height()));
    // We handle the mouse event...
    QMouseEvent *mEvent = (QMouseEvent*) event;
    if (!checkRect.contains(mEvent->pos())) {
      return false;
    }
    // as well as the key press event.
  } else if (event->type() == QEvent::KeyPress) {
    if (static_cast<QKeyEvent*>(event)->key() !=
	Qt::Key_Space&& static_cast<QKeyEvent*>(event)->key() != Qt::Key_Select) {
      return false;
    }
  } else {
    return false;
  }
  // Determine the new check state
  Qt::CheckState state = (static_cast<Qt::CheckState>(value.toInt()) == Qt::Checked
			  ? Qt::Unchecked : Qt::Checked);
  // And set the new check state by calling the model's setData() function.
  return model->setData(index, state, Qt::CheckStateRole);
}
{% endhighlight %}

The re-implemented version of the `paint()` function basically just changes the position at which the check box is drawn, by changing the rectangle within which the paint event takes place, and then calling the default version of the `paint()` function with an option parameter that includes the altered rectangle.

The re-implemented version of the `editorEvent()` function handles various events through which the user might manipulate the current state of the check box. If an event meets the required conditions (e.g., a `QEvent::MouseButtonRelease` took place within the rectangle where the check box is drawn), then we call the `setData()` function (the one we re-implemented earlier) with the appropriate parameters. 

There is actually one other thing we need to do if we want our [QStyledItemDelegate][16] to work. We need to tell our [QTableView][9] object to use the delegate in its seventh column. In my case, the [QTableView][9] object is name `tableView`, and the sub-classed version of the [QStyledItemDelegate][16] I created is named `QCheckBoxDelegate`. I set `tableView` to use the `QCheckBoxDelegate` by using the following function:

{% highlight c++ %}
tableView->setItemDelegateForColumn(7, new CheckBoxDelegate(tableView));
{% endhighlight %}

And that should do the trick! Now you should have a [QTableView][9] with a nice-looking column of check boxes through which the user can interact with 'pretend boolean' variables in your sql database. 


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
[18]: http://doc.qt.io/qt-5/qsqltablemodel.html#flags
[19]: http://doc.qt.io/qt-5/qt.html#ItemFlag-enum
[20]: https://falsinsoft.blogspot.co.uk/2013/11/qtablewidget-center-checkbox-inside-cell.html
