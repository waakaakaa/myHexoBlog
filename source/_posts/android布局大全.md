title: Android布局大全
date: 2014-12-05 18:11:50

---
Android的界面是有布局和组件协同完成的，布局好比是建筑里的框架，而组件则相当于建筑里的砖瓦。组件按照布局的要求依次排列，就组成了用户所看见的界面。

<!--more-->

![image](/img/Android布局大全/1.jpg)

所有的布局方式都可以归类为ViewGroup的5个类别，即ViewGroup的5个直接子类。其它的一些布局都扩展自这5个类。

### LinearLayout，线性布局方式

这种布局比较常用，也比较简单，就是每个元素占一行，当然也可能声明为横向排放，也就是每个元素占一列。

LinearLayout按照垂直或者水平的顺序依次排列子元素，每一个子元素都位于前一个元素之后。如果是垂直排列，那么将是一个N行单列的结构，每一行只会有一个元素，而不论这个元素的宽度为多少；如果是水平排列，那么将是一个单行N列的结构。如果搭建两行两列的结构，通常的方式是先垂直排列两个元素，每一个元素里再包含一个LinearLayout进行水平排列。

LinearLayout中的子元素属性android:layout_weight生效，它用于描述该子元素在剩余空间中占有的大小比例。加入一行只有一个文本框，那么它的默认值就为0，如果一行中有两个等长的文本框，那么他们的android:layout_weight值可以是同为1。如果一行中有两个不等长的文本框，那么他们的android:layout_weight值分别为1和2，那么第一个文本框将占据剩余空间的三分之二，第二个文本框将占据剩余空间中的三分之一。android:layout_weight遵循数值越小，重要度越高的原则。

	<?xml version="1.0" encoding="utf-8"?>
	 <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android" android:orientation="vertical" android:layout_width="fill_parent" android:layout_height="fill_parent">
	     <TextView  android:layout_width="fill_parent" android:layout_height="wrap_content" android:background="#ff000000" android:text="@string/hello"/>
	     <LinearLayout android:orientation="horizontal" android:layout_width="fill_parent" android:layout_height="fill_parent">
	         <TextView  android:layout_width="fill_parent" android:layout_height="wrap_content" android:background="#ff654321" android:layout_weight="1" android:text="1"/>
	         <TextView  android:layout_width="fill_parent" android:layout_height="wrap_content" android:background="#fffedcba" android:layout_weight="2"  android:text="2"/>
	     </LinearLayout>
	 </LinearLayout>
	</xml>
	
###Relative Layout，相对布局

RelativeLayout按照各子元素之间的位置关系完成布局。在此布局中的子元素里与位置相关的属性将生效。例如android：layout_below,  android:layout_above, android:layout_centerVertical等。注意在指定位置关系时，引用的ID必须在引用之前，先被定义，否则将出现异常。

RelativeLayout是Android五大布局结构中最灵活的一种布局结构，比较适合一些复杂界面的布局。

	<?xml version="1.0" encoding="utf-8"?>
	 <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android" android:orientation="vertical" android:layout_width="fill_parent" android:layout_height="fill_parent">
	     <TextView android:id="@+id/text_01" android:layout_width="50dp" android:layout_height="50dp" android:background="#ffffffff" android:gravity="center" android:layout_alignParentBottom="true" android:text="1"/>
	     <TextView android:id="@+id/text_02" android:layout_width="50dp" android:layout_height="50dp" android:background="#ff654321" android:gravity="center" android:layout_above="@id/text_01" android:layout_centerHorizontal="true" android:text="2"/>
	     <TextView android:id="@+id/text_03" android:layout_width="50dp" android:layout_height="50dp" android:background="#fffedcba" android:gravity="center" android:layout_toLeftOf="@id/text_02" android:layout_above="@id/text_01" android:text="3"/>
	 </RelativeLayout>
	</xml>
	
###AbsoluteLayout，绝对位置布局

在此布局中的子元素的android:layout_x和android:layout_y属性将生效，用于描述该子元素的坐标位置。屏幕左上角为坐标原点（0,0），第一个0代表横坐标，向右移动此值增大，第二个0代表纵坐标，向下移动，此值增大。在此布局中的子元素可以相互重叠。在实际开发中，通常不采用此布局格式，因为它的界面代码过于刚性，以至于有可能不能很好的适配各种终端。

	<?xml version="1.0" encoding="utf-8"?>
	 <AbsoluteLayout xmlns:android="http://schemas.android.com/apk/res/android" android:orientation="vertical" android:layout_width="fill_parent" android:layout_height="fill_parent">
	     <TextView android:layout_width="50dp" android:layout_height="50dp" android:background="#ffffffff" android:gravity="center" android:layout_x="50dp" android:layout_y="50dp" android:text="1"/>
	     <TextView android:layout_width="50dp" android:layout_height="50dp" android:background="#ff654321" android:gravity="center" android:layout_x="25dp" android:layout_y="25dp" android:text="2"/>
	     <TextView  android:layout_width="50dp" android:layout_height="50dp" android:background="#fffedcba" android:gravity="center" android:layout_x="125dp" android:layout_y="125dp" android:text="3"/>
	 </AbsoluteLayout>
	</xml>
	
###FrameLayout，帧布局

FrameLayout是五大布局中最简单的一个布局，可以说成是层布局方式。在这个布局中，整个界面被当成一块空白备用区域，所有的子元素都不能被指定放置的位置，它们统统放于这块区域的左上角，并且后面的子元素直接覆盖在前面的子元素之上，将前面的子元素部分和全部遮挡。如下，第一个TextView被第二个TextView完全遮挡，第三个TextView遮挡了第二个TextView的部分位置。

	<?xml version="1.0" encoding="utf-8"?>
	 <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android" android:orientation="vertical" android:layout_width="fill_parent" android:layout_height="fill_parent">
	     <TextView android:layout_width="fill_parent" android:layout_height="fill_parent" android:background="#ff000000" android:gravity="center" android:text="1"/>
	     <TextView android:layout_width="fill_parent" android:layout_height="fill_parent" android:background="#ff654321" android:gravity="center" android:text="2"/>
	     <TextView android:layout_width="50dp" android:layout_height="50dp" android:background="#fffedcba" android:gravity="center" android:text="3"/>
	 </FrameLayout>
	</xml>
	
###TableLayout，表格布局

适用于N行N列的布局格式。一个TableLayout由许多TableRow组成，一个TableRow就代表TableLayout中的一行。

TableRow是LinearLayout的子类，ablelLayout并不需要明确地声明包含多少行、多少列，而是通过TableRow，以及其他组件来控制表格的行数和列数， TableRow也是容器，因此可以向TableRow里面添加其他组件，没添加一个组件该表格就增加一列。如果想TableLayout里面添加组件，那么该组件就直接占用一行。在表格布局中，列的宽度由该列中最宽的单元格决定，整个表格布局的宽度取决于父容器的宽度（默认是占满父容器本身）。

TableLayout继承了LinearLayout，因此他完全可以支持LinearLayout所支持的全部XML属性，除此之外TableLayout还支持以下属性：

XML属性                  |         相关用法                    |              说明      
-------------------     | ---------------------------------  | ------------------------
andriod：collapseColumns | setColumnsCollapsed（int ，boolean）| 设置需要隐藏的列的序列号，多个用逗号隔开
android：shrinkColumns   | setShrinkAllColumns（boolean）      | 设置被收缩的列的序列号，多个用逗号隔开
android：stretchColimns  | setSretchAllColumnds（boolean）     | 设置允许被拉伸的列的序列号，多个用逗号隔开

	<?xml version="1.0" encoding="utf-8"?>
	 <TableLayout xmlns:android="http://schemas.android.com/apk/res/android" android:orientation="vertical" android:layout_width="fill_parent" android:layout_height="fill_parent">
	     <TableRow android:layout_width="fill_parent" android:layout_height="wrap_content">
	         <TextView  android:background="#ffffffff" android:gravity="center" android:padding="10dp" android:text="1"/>
	         <TextView android:background="#ff654321" android:gravity="center" android:padding="10dp" android:text="2"/>
	         <TextView  android:background="#fffedcba" android:gravity="center" android:padding="10dp" android:text="3"/>
	     </TableRow>
	     <TableRow android:layout_width="fill_parent" android:layout_height="wrap_content">
	         <TextView  android:background="#ff654321" android:gravity="center" android:padding="10dp" android:text="2"/>
	         <TextView android:background="#fffedcba" android:gravity="center" android:padding="10dp" android:text="3"/>        
	     </TableRow>
	     <TableRow android:layout_width="fill_parent" android:layout_height="wrap_content">
	         <TextView android:background="#fffedcba" android:gravity="center" android:padding="10dp" android:text="3"/>
	         <TextView  android:background="#ff654321" android:gravity="center" android:padding="10dp" android:text="2"/>
	         <TextView  android:background="#ffffffff" android:gravity="center" android:padding="10dp" android:text="1"/>        
	     </TableRow>
	 </TableLayout>
	</xml>
	
###其他布局(隶属关系请看上图)

1. 列表视图（List View）
List View是可滚动的列表。以列表的形式展示具体内容，并且能够根据数据的长度自适应显示。
具体应用请看：用法一  [http://www.cnblogs.com/allin/archive/2010/05/11/1732200.html](http://www.cnblogs.com/allin/archive/2010/05/11/1732200.html)，用法二  [http://blog.csdn.net/koupoo/article/details/7018727](http://blog.csdn.net/koupoo/article/details/7018727)　　　　　　

2. 网格视图（Grid View）
Grid View一个ViewGroup以网格显示它的子视图（view）元素，即二维的、滚动的网格。具体应用查看：[http://www.cnblogs.com/linzheng/archive/2011/01/19/1938760.html](http://www.cnblogs.com/linzheng/archive/2011/01/19/1938760.html)

3. 标签布局（Tab Layout）
以标签的方式显示它的子视图元素，就像在Firefox中的一个窗口中显示多个网页一样。为了狂创建一个标签UI（tabbed UI），需要使用到TabHost和TabWidget。TabHost必须是布局的根节点，它包含为了显示标签的TabWidget和显示标签内容的FrameLayout。具体应用查看：[http://www.cnblogs.com/devinzhang/archive/2012/01/18/2325887.html](http://www.cnblogs.com/devinzhang/archive/2012/01/18/2325887.html)