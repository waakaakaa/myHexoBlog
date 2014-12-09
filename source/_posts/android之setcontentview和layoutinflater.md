title: Android之setContentView和LayoutInflater
date: 2014-12-05 04:11:46

---
# setContentView：

1.常用的构造函数：

	  1) setContentView(int layoutResID)
	
	  2) setContentView(View view)
	
	  3) setContentView(View view, ViewGroup.LayoutParams params)
	  
<!-- more -->

2.用法

	1）setContentView(R.layout.main);
	
	2）LayoutInflater inflater = (LayoutInflater)getSystemService(Context.LAYOUT_INFLATER_SERVICE);
	
	   View view = (View) inflater.inflate(R.layout.apploader, null, true);
	
	   setContentView(view);	
	   
3.两种用法的适用场景：


setContentView()一旦调用, layout就会立刻显示UI；而inflate只会把Layout形成一个以view类实现成的对象，有需要时再用setContentView(view)显示出来。

一般在activity中通过setContentView()将界面显示出来，但是如果要在非activity中如何对控件布局进行设置操作，就需LayoutInflater动态加载。

# LayoutInflater：

获得 LayoutInflater 实例的三种方式

	1. LayoutInflater inflater = getLayoutInflater();  //调用Activity的getLayoutInflater()
	
	2. LayoutInflater localinflater = (LayoutInflater)context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
	
	3. LayoutInflater inflater = LayoutInflater.from(context);   

其实，这三种方式本质是相同的，从源码中可以得出结论：这三种方式最终本质是都是调用的Context.getSystemService()。

