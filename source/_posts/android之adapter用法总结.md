title: Android之Adapter用法总结
date: 2014-12-05 02:33:19

---
# 概念

Adapter是连接后端数据和前端显示的适配器接口，是数据和UI（View）之间一个重要的纽带。在常见的View(ListView,GridView)等地方都需要用到Adapter。如下图直观的表达了Data、Adapter、View三者的关系：

<!-- more -->

![image](/img/Android之Adapter用法总结/1.jpg)

Android中所有的Adapter一览：

![image](/img/Android之Adapter用法总结/2.png)

由图可以看到在Android中与Adapter有关的所有接口、类的完整层级图。在我们使用过程中可以根据自己的需求实现接口或者继承类进行一定的扩展。比较常用的有 BaseAdapter，SimpleAdapter，ArrayAdapter，SimpleCursorAdapter等。

1. BaseAdapter是一个抽象类，继承它需要实现较多的方法，所以也就具有较高的灵活性；
2. ArrayAdapter支持泛型操作，最为简单，只能展示一行字。
3. SimpleAdapter有最好的扩充性，可以自定义出各种效果。
4. SimpleCursorAdapter可以适用于简单的纯文字型ListView，它需要Cursor的字段和UI的id对应起来。如需要实现更复杂的UI也可以重写其他方法。可以认为是SimpleAdapter对数据库的简单结合，可以方便地把数据库的内容以列表的形式展示出来。

# 应用案例

## ArrayAdapter

列表的显示需要三个元素：

1. ListView：用来展示列表的View。
2. 适配器:用来把数据映射到ListView上的中介。
3. 数据：具体的将被映射的字符串，图片，或者基本组件。

案例一

	public class ArrayAdapterActivity extends ListActivity {
	     @Override
	     public void onCreate(Bundle savedInstanceState) {
	         super.onCreate(savedInstanceState);
	         //列表项的数据
	         String[] strs = {"1","2","3","4","5"};
	         ArrayAdapter<String> adapter = new ArrayAdapter<String>(this,android.R.layout.simple_expandable_list_item_1,strs);
	         setListAdapter(adapter);
	     }
	 }
	
案例二

	public class MyListView extends Activity {
        private ListView listView;
        //private List<String> data = new ArrayList<String>();
        @Override
        public void onCreate(Bundle savedInstanceState){
            super.onCreate(savedInstanceState);
             
            listView = new ListView(this);
            listView.setAdapter(new ArrayAdapter<String>(this, android.R.layout.simple_expandable_list_item_1,getData()));
            setContentView(listView);
        }
         
        private List<String> getData(){
             
            List<String> data = new ArrayList<String>();
            data.add("测试数据1");
            data.add("测试数据2");
            data.add("测试数据3");
            data.add("测试数据4");
             
            return data;
        }
    }
    
上面代码使用了ArrayAdapter(Context context, int textViewResourceId, List<T> objects)来装配数据，要装配这些数据就需要一个连接ListView视图对象和数组数据的适配器来两者的适配工作，ArrayAdapter的构造需要三个参数，依次为this,布局文件（注意这里的布局文件描述的是列表的每一行的布局，android.R.layout.simple_list_item_1是系统定义好的布局文件只显示一行文字，数据源(一个List集合)。同时用setAdapter（）完成适配的最后工作。效果图如下：

![image](/img/Android之Adapter用法总结/3.png)

## SimpleAdapter

SimpleAdapter的扩展性最好，可以定义各种各样的布局出来，可以放上ImageView（图片），还可以放上Button（按钮），CheckBox（复选框）等等。下面的代码都直接继承了ListActivity，ListActivity和普通的Activity没有太大的差别，不同就是对显示ListView做了许多优化，方面显示而已。

案例一

simple.xml

	<?xml version="1.0" encoding="utf-8"?>
		<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
			android:orientation="vertical"
			android:layout_width="fill_parent"
			android:layout_height="fill_parent">
			
			<ImageView
				android:id="@+id/img"
				android:layout_width="wrap_content"
				android:layout_height="wrap_content"
				android:layout_margin="5dp"/>
			<TextView
				android:id="@+id/title"
				android:layout_width="wrap_content" 
				android:layout_height="wrap_content" 
				android:textColor="#ffffff"
				android:textSize="20sp"/>
		</LinearLayout>
	</xml>

<!-- -->

	public class SimpleAdapterActivity extends ListActivity {
	     @Override
	     public void onCreate(Bundle savedInstanceState) {
	         super.onCreate(savedInstanceState);
         
         SimpleAdapter adapter = new SimpleAdapter(this, getData(), R.layout.simple, new String[] { "title",  "img" }, new int[] { R.id.title, R.id.img });
         setListAdapter(adapter);
     }
     
    private List<Map<String, Object>> getData() {
         //map.put(参数名字,参数值)
         List<Map<String, Object>> list = new ArrayList<Map<String, Object>>();
         Map<String, Object> map = new HashMap<String, Object>();
         map.put("title", "摩托罗拉");
         map.put("img", R.drawable.icon);
         list.add(map);
         
         map = new HashMap<String, Object>();
         map.put("title", "诺基亚");
         map.put("img", R.drawable.icon);
         list.add(map);
         
         map = new HashMap<String, Object>();
         map.put("title", "三星");
         map.put("img", R.drawable.icon);
         list.add(map);
         return list;
         }  
     
	}
	
案例二

下面的程序是实现一个带有图片的类表。首先需要定义好一个用来显示每一个列内容的xml，

vlist.xml

	<?xml version="1.0" encoding="utf-8"?>
	    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android" 			android:orientation="horizontal" 
	    	android:layout_width="fill_parent"
	        android:layout_height="fill_parent">
         
	        <ImageView 
	        	android:id="@+id/img" 
	        	android:layout_width="wrap_content" 
	        	android:layout_height="wrap_content" 
	        	android:layout_margin="5px"/>
	        	
	        <LinearLayout 
	        	android:orientation="vertical"  
	        	android:layout_width="wrap_content"  
	        	android:layout_height="wrap_content">
	        	
	            <TextView 
	            	android:id="@+id/title" 
	            	android:layout_width="wrap_content" 
	            	android:layout_height="wrap_content"
	                android:textColor="#FFFFFFFF" 
	                android:textSize="22px" />
	                
	            <TextView 
	            	android:id="@+id/info"  
	            	android:layout_width="wrap_content" 
	            	android:layout_height="wrap_content"
	                android:textColor="#FFFFFFFF" 
	                android:textSize="13px" />
	                
	        </LinearLayout>
	     </LinearLayout>
	 </xml>
	 
<!-- -->

	public class MyListView3 extends ListActivity {
        // private List<String> data = new ArrayList<String>();
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
     
            SimpleAdapter adapter = new SimpleAdapter(this,getData(),R.layout.vlist,
                    new String[]{"title","info","img"},
                    new int[]{R.id.title,R.id.info,R.id.img});
            setListAdapter(adapter);
        }
     
        private List<Map<String, Object>> getData() {
            List<Map<String, Object>> list = new ArrayList<Map<String, Object>>();
     
            Map<String, Object> map = new HashMap<String, Object>();
            map.put("title", "G1");
            map.put("info", "google 1");
            map.put("img", R.drawable.i1);
            list.add(map);
     
            map = new HashMap<String, Object>();
            map.put("title", "G2");
            map.put("info", "google 2");
            map.put("img", R.drawable.i2);
            list.add(map);
     
            map = new HashMap<String, Object>();
            map.put("title", "G3");
            map.put("info", "google 3");
            map.put("img", R.drawable.i3);
            list.add(map);
             
            return list;
        }
    }
    
使用simpleAdapter的数据用一般都是HashMap构成的List，list的每一节对应ListView的每一行。HashMap的每个键值数据映射到布局文件中对应id的组件上。因为系统没有对应的布局文件可用，我们可以自己定义一个布局vlist.xml。下面做适配，new一个SimpleAdapter参数一次是：this，布局文件（vlist.xml），HashMap的 title 和 info，img。布局文件的组件id，title，info，img。布局文件的各组件分别映射到HashMap的各元素上，完成适配。

运行效果如下图：

![image](/img/Android之Adapter用法总结/4.png)

## SimpleCursorAdapter

	public class SimpleCursorAdapterActivity extends ListActivity {
	     @Override
	     public void onCreate(Bundle savedInstanceState) {
	         super.onCreate(savedInstanceState);
	         //获得一个指向系统通讯录数据库的Cursor对象获得数据来源
	         Cursor cur = getContentResolver().query(People.CONTENT_URI, null, null, null, null);
	         startManagingCursor(cur);
	         //实例化列表适配器
	         
	         ListAdapter adapter = new SimpleCursorAdapter(this, android.R.layout.simple_list_item_1, cur, new String[] {People.NAME}, new int[] {android.R.id.text1});
	         setListAdapter(adapter);
	     }
	}
	
一定要以数据库作为数据源的时候,才能使用SimpleCursorAdapter，这里特别需要注意的一点是：不要忘了在AndroidManifest.xml文件中加入权限

	<uses-permission android:name="android.permission.READ_CONTACTS"></uses-permission>

效果如下：

![image](/img/Android之Adapter用法总结/5.jpg)

## BaseAdapter

有时候，列表不光会用来做显示用，我们同样可以在在上面添加按钮。添加按钮首先要写一个有按钮的xml文件，然后自然会想到用上面的方法定义一个适配器，然后将数据映射到布局文件上。但是事实并非这样，因为按钮是无法映射的，即使你成功的用布局文件显示出了按钮也无法添加按钮的响应，这时就要研究一下ListView是如何现实的了，而且必须要重写一个类继承BaseAdapter。下面的示例将显示一个按钮和一个图片，两行字如果单击按钮将删除此按钮的所在行。并告诉你ListView究竟是如何工作的。

vlist2.xml

	<?xml version="1.0" encoding="utf-8"?>
	    <LinearLayout 
	    	xmlns:android="http://schemas.android.com/apk/res/android" 			android:orientation="horizontal" 
	    	android:layout_width="fill_parent"
	        android:layout_height="fill_parent">
	        
	        <ImageView 
	        	android:id="@+id/img" 
	        	android:layout_width="wrap_content" 
	        	android:layout_height="wrap_content" 
	        	android:layout_margin="5px"/>
	        	
	        <LinearLayout 
	        	android:orientation="vertical" 
	        	android:layout_width="wrap_content" 
	        	android:layout_height="wrap_content">
	        	
	           <TextView 
	               android:id="@+id/title" 
	               android:layout_width="wrap_content" 
	               android:layout_height="wrap_content"
	               android:textColor="#FFFFFFFF" 
	               android:textSize="22px" />
	               
	           <TextView 
	               android:id="@+id/info" 
	               android:layout_width="wrap_content" 
	               android:layout_height="wrap_content"
	               android:textColor="#FFFFFFFF" 
	               android:textSize="13px" />
	               
	       </LinearLayout>
	
	       <Button 
	           android:id="@+id/view_btn" 
	           android:layout_width="wrap_content"  
	           android:layout_height="wrap_content"
	           android:text="@string/s_view_btn" 
	           android:layout_gravity="bottom|right" />
	           
	    </LinearLayout>
	</xml>
	
<!-- -->

    public class MyListView4 extends ListActivity {   
    
       private List<Map<String, Object>> mData;
        
       @Override
       public void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           mData = getData();
           MyAdapter adapter = new MyAdapter(this);
           setListAdapter(adapter);
       }
    
       private List<Map<String, Object>> getData() {
           List<Map<String, Object>> list = new ArrayList<Map<String, Object>>();
    
           Map<String, Object> map = new HashMap<String, Object>();
           map.put("title", "G1");
           map.put("info", "google 1");
           map.put("img", R.drawable.i1);
           list.add(map);
    
           map = new HashMap<String, Object>();
           map.put("title", "G2");
           map.put("info", "google 2");
           map.put("img", R.drawable.i2);
           list.add(map);
    
           map = new HashMap<String, Object>();
           map.put("title", "G3");
           map.put("info", "google 3");
           map.put("img", R.drawable.i3);
           list.add(map);
            
           return list;
       }
        
       // ListView 中某项被选中后的逻辑
       @Override
       protected void onListItemClick(ListView l, View v, int position, long id) {
            
           Log.v("MyListView4-click", (String)mData.get(position).get("title"));
       }
        
       /**
        * listview中点击按键弹出对话框
        */
       public void showInfo(){
           new AlertDialog.Builder(this)
           .setTitle("我的listview")
           .setMessage("介绍...")
           .setPositiveButton("确定", new DialogInterface.OnClickListener() {
               @Override
               public void onClick(DialogInterface dialog, int which) {
               }
           })
           .show();
            
       }
        
        
        
       public final class ViewHolder{
           public ImageView img;
           public TextView title;
           public TextView info;
           public Button viewBtn;
       }
        
        
       public class MyAdapter extends BaseAdapter{
    
           private LayoutInflater mInflater;
            
            
           public MyAdapter(Context context){
               this.mInflater = LayoutInflater.from(context);
           }
           @Override
           public int getCount() {
               // TODO Auto-generated method stub
               return mData.size();
           }
    
           @Override
           public Object getItem(int arg0) {
               // TODO Auto-generated method stub
               return null;
           }
    
           @Override
           public long getItemId(int arg0) {
               // TODO Auto-generated method stub
               return 0;
           }
    
           @Override
           public View getView(int position, View convertView, ViewGroup parent) {
                
               ViewHolder holder = null;
               if (convertView == null) {
                    
                   holder=new ViewHolder(); 
                    
                   convertView = mInflater.inflate(R.layout.vlist2, null);
                   holder.img = (ImageView)convertView.findViewById(R.id.img);
                   holder.title = (TextView)convertView.findViewById(R.id.title);
                   holder.info = (TextView)convertView.findViewById(R.id.info);
                   holder.viewBtn = (Button)convertView.findViewById(R.id.view_btn);
                   convertView.setTag(holder);
                    
               }else {
                    
                   holder = (ViewHolder)convertView.getTag();
               }
                
                
               holder.img.setBackgroundResource((Integer)mData.get(position).get("img"));
               holder.title.setText((String)mData.get(position).get("title"));
               holder.info.setText((String)mData.get(position).get("info"));
                
               holder.viewBtn.setOnClickListener(new View.OnClickListener() {
                    
                   @Override
                   public void onClick(View v) {
                       showInfo();                
                   }
               });
                
                
               return convertView;
           }
            
       }     
    }
    
下面将对上述代码，做详细的解释，listView在开始绘制的时候，系统首先调用getCount（）函数，根据他的返回值得到listView的长度（这也是为什么在开始的第一张图特别的标出列表长度），然后根据这个长度，调用getView（）逐一绘制每一行。如果你的getCount（）返回值是0的话，列表将不显示同样return 1，就只显示一行。

系统显示列表时，首先实例化一个适配器（这里将实例化自定义的适配器）。当手动完成适配时，必须手动映射数据，这需要重写getView（）方法。系统在绘制列表的每一行的时候将调用此方法。getView()有三个参数，position表示将显示的是第几行，covertView是从布局文件中inflate来的布局。我们用LayoutInflater的方法将定义好的vlist2.xml文件提取成View实例用来显示。然后将xml文件中的各个组件实例化（简单的findViewById()方法）。这样便可以将数据对应到各个组件上了。但是按钮为了响应点击事件，需要为它添加点击监听器，这样就能捕获点击事件。至此一个自定义的listView就完成了，现在让我们回过头从新审视这个过程。系统要绘制ListView了，他首先获得要绘制的这个列表的长度，然后开始绘制第一行，怎么绘制呢？调用getView()函数。在这个函数里面首先获得一个View（实际上是一个ViewGroup），然后再实例并设置各个组件，显示之。好了，绘制完这一行了。那再绘制下一行，直到绘完为止。在实际的运行过程中会发现listView的每一行没有焦点了，这是因为Button抢夺了listView的焦点，只要布局文件中将Button设置为没有焦点就OK了。

效果如下：

![image](/img/Android之Adapter用法总结/6.png)
