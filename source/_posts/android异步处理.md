title: Android异步处理
date: 2014-12-09 01:09:30

---
#Android异步处理一：使用Thread+Handler实现非UI线程更新UI界面

概述：每个Android应用程序都运行在一个dalvik虚拟机进程中，进程开始的时候会启动一个主线程(MainThread)，主线程负责处理和ui相关的事件，因此主线程通常又叫UI线程。而由于Android采用UI单线程模型，所以只能在主线程中对UI元素进行操作。如果在非UI线程直接对UI进行了操作，则会报错：

<!--more-->

CalledFromWrongThreadException:only the original thread that created a view hierarchy can touch its views
。
Android为我们提供了消息循环的机制，我们可以利用这个机制来实现线程间的通信。那么，我们就可以在非UI线程发送消息到UI线程，最终让Ui线程来进行ui的操作。

对于运算量较大的操作和IO操作，我们需要新开线程来处理这些繁重的工作，以免阻塞ui线程。

例子：下面我们以获取CSDN logo的例子，演示如何使用Thread+Handler的方式实现在非UI线程发送消息通知UI线程更新界面。



ThradHandlerActivity.java:

	public class ThreadHandlerActivity extends Activity {
	    /** Called when the activity is first created. */
		
		private static final int MSG_SUCCESS = 0;//获取图片成功的标识
		private static final int MSG_FAILURE = 1;//获取图片失败的标识
		
		private ImageView mImageView;
		private Button mButton;
		
		private Thread mThread;
		
		private Handler mHandler = new Handler() {
			public void handleMessage (Message msg) {//此方法在ui线程运行
				switch(msg.what) {
				case MSG_SUCCESS:
					mImageView.setImageBitmap((Bitmap) msg.obj);//imageview显示从网络获取到的logo
					Toast.makeText(getApplication(), getApplication().getString(R.string.get_pic_success), Toast.LENGTH_LONG).show();
					break;
	
				case MSG_FAILURE:
					Toast.makeText(getApplication(), getApplication().getString(R.string.get_pic_failure), Toast.LENGTH_LONG).show();
					break;
				}
			}
		};
		
	    @Override
	    public void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.main);
	        mImageView= (ImageView) findViewById(R.id.imageView);//显示图片的ImageView
	        mButton = (Button) findViewById(R.id.button);
	        mButton.setOnClickListener(new OnClickListener() {
				
				@Override
				public void onClick(View v) {
					if(mThread == null) {
						mThread = new Thread(runnable);
						mThread.start();//线程启动
					}
					else {
						Toast.makeText(getApplication(), getApplication().getString(R.string.thread_started), Toast.LENGTH_LONG).show();
					}
				}
			});
	    }
	    
	    Runnable runnable = new Runnable() {
			
			@Override
			public void run() {//run()在新的线程中运行
				HttpClient hc = new DefaultHttpClient();
				HttpGet hg = new HttpGet("http://csdnimg.cn/www/images/csdnindex_logo.gif");//获取csdn的logo
				final Bitmap bm;
				try {
					HttpResponse hr = hc.execute(hg);
					bm = BitmapFactory.decodeStream(hr.getEntity().getContent());
				} catch (Exception e) {
					mHandler.obtainMessage(MSG_FAILURE).sendToTarget();//获取图片失败
					return;
				}
				mHandler.obtainMessage(MSG_SUCCESS,bm).sendToTarget();//获取图片成功，向ui线程发送MSG_SUCCESS标识和bitmap对象
	
	//			mImageView.setImageBitmap(bm); //出错！不能在非ui线程操作ui元素
	
	//			mImageView.post(new Runnable() {//另外一种更简洁的发送消息给ui线程的方法。
	//				
	//				@Override
	//				public void run() {//run()方法会在ui线程执行
	//					mImageView.setImageBitmap(bm);
	//				}
	//			});
			}
		};
		
	}
	
main.xml布局文件：

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
		android:orientation="vertical" android:layout_width="fill_parent"
		android:layout_height="fill_parent">
	    <Button android:id="@+id/button" android:text="@string/button_name" android:layout_width="wrap_content" android:layout_height="wrap_content"></Button>
		<ImageView android:id="@+id/imageView" android:layout_height="wrap_content"
			android:layout_width="wrap_content" />
	</LinearLayout>

strings.xml

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
		android:orientation="vertical" android:layout_width="fill_parent"
		android:layout_height="fill_parent">
	    <Button android:id="@+id/button" android:text="@string/button_name" android:layout_width="wrap_content" android:layout_height="wrap_content"></Button>
		<ImageView android:id="@+id/imageView" android:layout_height="wrap_content"
			android:layout_width="wrap_content" />
	</LinearLayout>

Manifest.xml:

	<?xml version="1.0" encoding="utf-8"?>
	<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	      package="com.zhuozhuo"
	      android:versionCode="1"
	      android:versionName="1.0">
	    <uses-sdk android:minSdkVersion="9" />
	    <uses-permission android:name="android.permission.INTERNET"></uses-permission><!--不要忘记设置网络访问权限-->
	
	    <application android:icon="@drawable/icon" android:label="@string/app_name">
	        <activity android:name=".ThreadHandlerActivity"
	                  android:label="@string/app_name">
	            <intent-filter>
	                <action android:name="android.intent.action.MAIN" />
	                <category android:name="android.intent.category.LAUNCHER" />
	            </intent-filter>
	        </activity>
	
	    </application>
	</manifest>

运行结果：

![image](/img/Android异步处理/1.jpg)

为了不阻塞ui线程，我们使用mThread从网络获取了CSDN的LOGO ，并用bitmap对象存储了这个Logo的像素信息。
此时，如果在这个线程的run()方法中调用

	mImageView.setImageBitmap(bm)  
	
会出现：CalledFromWrongThreadException:only the original thread that created a view hierarchy can touch its views。原因是run()方法是在新开的线程中执行的，我们上面提到不能直接在非ui线程中操作ui元素。

非UI线程发送消息到UI线程分为两个步骤

一、发送消息到UI线程的消息队列

通过使用Handler的

	Message obtainMessage(int what,Object object)  
	
构造一个Message对象，这个对象存储了是否成功获取图片的标识what和bitmap对象，然后通过message.sendToTarget()方法把这条message放到消息队列中去。

二、处理发送到UI线程的消息

在ui线程中，我们覆盖了handler的 

	public void handleMessage (Message msg)   

这个方法是处理分发给ui线程的消息，判断msg.what的值可以知道mThread是否成功获取图片，如果图片成功获取，那么可以通过msg.obj获取到这个对象。

最后，我们通过

	mImageView.setImageBitmap((Bitmap) msg.obj);  
	
设置ImageView的bitmap对象，完成UI的更新。

补充：

事实上，我们还可以调用View的post方法来更新ui

	mImageView.post(new Runnable() {//另外一种更简洁的发送消息给ui线程的方法。
					
					@Override
					public void run() {//run()方法会在ui线程执行
						mImageView.setImageBitmap(bm);
					}
				});

这种方法会把Runnable对象发送到消息队列，ui线程接收到消息后会执行这个runnable对象。

从例子中我们可以看到handler既有发送消息和处理消息的作用，会误以为handler实现了消息循环和消息分发，其实Android为了让我们的代码看起来更加简洁，与UI线程的交互只需要使用在UI线程创建的handler对象就可以了。如需深入学习，了解消息循环机制的具体实现，请关注《Android异步处理三：Handler+Looper+MessageQueue深入详解》

===

#Android异步处理二：使用AsyncTask异步更新UI界面

概述： AsyncTask是在Android SDK 1.5之后推出的一个方便编写后台线程与UI线程交互的辅助类。AsyncTask的内部实现是一个线程池，每个后台任务会提交到线程池中的线程执行，然后使用Thread+Handler的方式调用回调函数（如需深入了解原理请看《Android异步处理四：AsyncTask的实现原理》）。

AsyncTask抽象出后台线程运行的五个状态，分别是：1、准备运行，2、正在后台运行，3、进度更新，4、完成后台任务，5、取消任务，对于这五个阶段，AsyncTask提供了五个回调函数：

1. 准备运行：onPreExecute(),该回调函数在任务被执行之后立即由UI线程调用。这个步骤通常用来建立任务，在用户接口（UI）上显示进度条。

2. 正在后台运行：doInBackground(Params...),该回调函数由后台线程在onPreExecute()方法执行结束后立即调用。通常在这里执行耗时的后台计算。计算的结果必须由该函数返回，并被传递到onPostExecute()中。在该函数内也可以使用publishProgress(Progress...)来发布一个或多个进度单位(unitsof progress)。这些值将会在onProgressUpdate(Progress...)中被发布到UI线程。

3. 进度更新：onProgressUpdate(Progress...),该函数由UI线程在publishProgress(Progress...)方法调用完后被调用。一般用于动态地显示一个进度条。

4. 完成后台任务：onPostExecute(Result),当后台计算结束后调用。后台计算的结果会被作为参数传递给这一函数。

5. 取消任务：onCancelled ()，在调用AsyncTask的cancel()方法时调用

AsyncTask的构造函数有三个模板参数：

1. Params，传递给后台任务的参数类型。

2. Progress，后台计算执行过程中，进步单位（progress　units）的类型。（就是后台程序已经执行了百分之几了。）

3. Result， 后台执行返回的结果的类型。

AsyncTask并不总是需要使用上面的全部3种类型。标识不使用的类型很简单，只需要使用Void类型即可。

例子：与《Android异步处理一：使用Thread+Handler实现非UI线程更新UI界面》实现的例子相同，我们在后台下载CSDN的LOGO，下载完成后在UI界面上显示出来，并会模拟下载进度更新。

AsyncTaskActivity.java

	package com.zhuozhuo;
	
	
	import org.apache.http.HttpResponse;
	import org.apache.http.client.HttpClient;
	import org.apache.http.client.methods.HttpGet;
	import org.apache.http.impl.client.DefaultHttpClient;
	
	import android.app.Activity;
	import android.graphics.Bitmap;
	import android.graphics.BitmapFactory;
	import android.os.AsyncTask;
	import android.os.Bundle;
	import android.view.View;
	import android.view.View.OnClickListener;
	import android.widget.Button;
	import android.widget.ImageView;
	import android.widget.ProgressBar;
	import android.widget.Toast;
	
	public class AsyncTaskActivity extends Activity {
	    
		private ImageView mImageView;
		private Button mButton;
		private ProgressBar mProgressBar;
		
	    @Override
	    public void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.main);
	        
	        mImageView= (ImageView) findViewById(R.id.imageView);
	        mButton = (Button) findViewById(R.id.button);
	        mProgressBar = (ProgressBar) findViewById(R.id.progressBar);
	        mButton.setOnClickListener(new OnClickListener() {
				
				@Override
				public void onClick(View v) {
					GetCSDNLogoTask task = new GetCSDNLogoTask();
					task.execute("http://csdnimg.cn/www/images/csdnindex_logo.gif");
				}
			});
	    }
	    
	    class GetCSDNLogoTask extends AsyncTask<String,Integer,Bitmap> {//继承AsyncTask
	
			@Override
			protected Bitmap doInBackground(String... params) {//处理后台执行的任务，在后台线程执行
				publishProgress(0);//将会调用onProgressUpdate(Integer... progress)方法
				HttpClient hc = new DefaultHttpClient();
				publishProgress(30);
				HttpGet hg = new HttpGet(params[0]);//获取csdn的logo
				final Bitmap bm;
				try {
					HttpResponse hr = hc.execute(hg);
					bm = BitmapFactory.decodeStream(hr.getEntity().getContent());
				} catch (Exception e) {
					
					return null;
				}
				publishProgress(100);
				//mImageView.setImageBitmap(result); 不能在后台线程操作ui
				return bm;
			}
			
			protected void onProgressUpdate(Integer... progress) {//在调用publishProgress之后被调用，在ui线程执行
				mProgressBar.setProgress(progress[0]);//更新进度条的进度
		     }
	
		     protected void onPostExecute(Bitmap result) {//后台任务执行完之后被调用，在ui线程执行
		    	 if(result != null) {
		    		 Toast.makeText(AsyncTaskActivity.this, "成功获取图片", Toast.LENGTH_LONG).show();
		    		 mImageView.setImageBitmap(result);
		    	 }else {
		    		 Toast.makeText(AsyncTaskActivity.this, "获取图片失败", Toast.LENGTH_LONG).show();
		    	 }
		     }
		     
		     protected void onPreExecute () {//在 doInBackground(Params...)之前被调用，在ui线程执行
		    	 mImageView.setImageBitmap(null);
		    	 mProgressBar.setProgress(0);//进度条复位
		     }
		     
		     protected void onCancelled () {//在ui线程执行
		    	 mProgressBar.setProgress(0);//进度条复位
		     }
	    	
	    }
	    
	
	}
	
main.xml

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
		android:orientation="vertical" android:layout_width="fill_parent"
		android:layout_height="fill_parent">
	    <ProgressBar android:layout_width="fill_parent" android:layout_height="wrap_content" android:id="@+id/progressBar" style="?android:attr/progressBarStyleHorizontal"></ProgressBar>
	    <Button android:id="@+id/button" android:text="下载图片" android:layout_width="wrap_content" android:layout_height="wrap_content"></Button>
		<ImageView android:id="@+id/imageView" android:layout_height="wrap_content"
			android:layout_width="wrap_content" />
	</LinearLayout>

manifest.xml

	<?xml version="1.0" encoding="utf-8"?>
	<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	      package="com.zhuozhuo"
	      android:versionCode="1"
	      android:versionName="1.0">
	    <uses-sdk android:minSdkVersion="10" />
	<uses-permission android:name="android.permission.INTERNET"></uses-permission>
	    <application android:icon="@drawable/icon" android:label="@string/app_name">
	        <activity android:name=".AsyncTaskActivity"
	                  android:label="@string/app_name">
	            <intent-filter>
	                <action android:name="android.intent.action.MAIN" />
	                <category android:name="android.intent.category.LAUNCHER" />
	            </intent-filter>
	        </activity>
	
	    </application>
	</manifest>

运行结果：

![image](/img/Android异步处理/2.jpg)

流程说明：

1. 当点击按钮时，创建一个task，并且传入CSDN的LOGO地址（String类型参数，因此AsyncTask的第一个模板参数为String类型）

2. UI线程执行onPreExecute()，把ImageView的图片清空，progrssbar的进度清零。

3. 后台线程执行doInBackground()，不可以在doInBackground()操作ui，调用publishProgress(0)更新进度，此时会调用onProgressUpdate(Integer...progress)更新进度条（进度用整形表示，因此AsyncTask的第二个模板参数是Integer）。函数最后返回result（例子中是返回Bitmap类型，因此AsyncTask的第三个模板参数是Bitmap）。

4. 当后台任务执行完成后，调用onPostExecute(Result)，传入的参数是doInBackground()中返回的对象。

总结：

AsyncTask为我们抽象出一个后台任务的五种状态，对应了五个回调接口，我们只需要根据不同的需求实现这五个接口（doInBackground是必须要实现的），就能完成一些简单的后台任务。使用AsyncTask的方式使编写后台进程和UI进程交互的代码变得更为简洁，使用起来更加方便，但是，AsyncTask也有一些缺憾，我们留到以后再讲。

===

#Android异步处理三：Handler+Looper+MessageQueue深入详解

概述：Android使用消息机制实现线程间的通信，线程通过Looper建立自己的消息循环，MessageQueue是FIFO的消息队列，Looper负责从MessageQueue中取出消息，并且分发到消息指定目标Handler对象。Handler对象绑定到线程的局部变量Looper，封装了发送消息和处理消息的接口。

例子：在介绍原理之前，我们先介绍Android线程通讯的一个例子，这个例子实现点击按钮之后从主线程发送消息"hello"到另外一个名为” CustomThread”的线程。

LooperThreadActivity.java

	package com.zhuozhuo;
	
	import android.app.Activity;
	import android.os.Bundle;
	import android.os.Handler;
	import android.os.Looper;
	import android.os.Message;
	import android.util.Log;
	import android.view.View;
	import android.view.View.OnClickListener;
	
	public class LooperThreadActivity extends Activity{
	    /** Called when the activity is first created. */
		
		private final int MSG_HELLO = 0;
	    private Handler mHandler;
		
	    @Override
	    public void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.main);
	        new CustomThread().start();//新建并启动CustomThread实例
	        
	        findViewById(R.id.send_btn).setOnClickListener(new OnClickListener() {
				
				@Override
				public void onClick(View v) {//点击界面时发送消息
					String str = "hello";
			        Log.d("Test", "MainThread is ready to send msg:" + str);
					mHandler.obtainMessage(MSG_HELLO, str).sendToTarget();//发送消息到CustomThread实例
					
				}
			});
	        
	    }
	    
	    
	    
	    
	    
	    class CustomThread extends Thread {
	    	@Override
	    	public void run() {
	    		//建立消息循环的步骤
	    		Looper.prepare();//1、初始化Looper
	    		mHandler = new Handler(){//2、绑定handler到CustomThread实例的Looper对象
	    			public void handleMessage (Message msg) {//3、定义处理消息的方法
	    				switch(msg.what) {
	    				case MSG_HELLO:
	    					Log.d("Test", "CustomThread receive msg:" + (String) msg.obj);
	    				}
	    			}
	    		};
	    		Looper.loop();//4、启动消息循环
	    	}
	    }
	}
	
main.xml

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:orientation="vertical"
	    android:layout_width="fill_parent"
	    android:layout_height="fill_parent"
	    >
	<TextView  
	    android:layout_width="fill_parent" 
	    android:layout_height="wrap_content" 
	    android:text="@string/hello"
	    />
	<Button android:text="发送消息" android:id="@+id/send_btn" android:layout_width="wrap_content" android:layout_height="wrap_content"></Button>
	</LinearLayout>

Log打印结果：

![image](/img/Android异步处理/3.png)

原理：


我们看到，为一个线程建立消息循环有四个步骤：

1. 初始化Looper

2. 绑定handler到CustomThread实例的Looper对象

3. 定义处理消息的方法

4. 启动消息循环

下面我们以这个例子为线索，深入Android源代码，说明Android Framework是如何建立消息循环，并对消息进行分发的。

1、  初始化Looper : Looper.prepare()

Looper.java

	private static final ThreadLocal sThreadLocal = new ThreadLocal();
	public static final void prepare() {
	        if (sThreadLocal.get() != null) {
	            throw new RuntimeException("Only one Looper may be created per thread");
	        }
	        sThreadLocal.set(new Looper());
	}

一个线程在调用Looper的静态方法prepare()时，这个线程会新建一个Looper对象，并放入到线程的局部变量中，而这个变量是不和其他线程共享的（关于ThreadLocal的介绍）。下面我们看看Looper()这个构造函数：

Looper.java

	final MessageQueue mQueue;
	private Looper() {
	        mQueue = new MessageQueue();
	        mRun = true;
	        mThread = Thread.currentThread();
	    }

可以看到在Looper的构造函数中，创建了一个消息队列对象mQueue，此时，调用Looper. prepare()的线程就建立起一个消息循环的对象（此时还没开始进行消息循环）。

2、  绑定handler到CustomThread实例的Looper对象 : mHandler= new Handler()

Handler.java

	final MessageQueue mQueue;
	 final Looper mLooper;
	public Handler() {
	        if (FIND_POTENTIAL_LEAKS) {
	            final Class<? extends Handler> klass = getClass();
	            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
	                    (klass.getModifiers() & Modifier.STATIC) == 0) {
	                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
	                    klass.getCanonicalName());
	            }
	        }
	
	        mLooper = Looper.myLooper();
	        if (mLooper == null) {
	            throw new RuntimeException(
	                "Can't create handler inside thread that has not called Looper.prepare()");
	        }
	        mQueue = mLooper.mQueue;
	        mCallback = null;
	}

Handler通过mLooper = Looper.myLooper();绑定到线程的局部变量Looper上去，同时Handler通过mQueue =mLooper.mQueue;获得线程的消息队列。此时，Handler就绑定到创建此Handler对象的线程的消息队列上了。

3、定义处理消息的方法：Override public void handleMessage (Message msg){}
   子类需要覆盖这个方法，实现接受到消息后的处理方法。

4、启动消息循环 ： Looper.loop()

所有准备工作都准备好了，是时候启动消息循环了！Looper的静态方法loop()实现了消息循环。

Looper.java

	 public static final void loop() {
	        Looper me = myLooper();
	        MessageQueue queue = me.mQueue;
	        
	        // Make sure the identity of this thread is that of the local process,
	        // and keep track of what that identity token actually is.
	        Binder.clearCallingIdentity();
	        final long ident = Binder.clearCallingIdentity();
	        
	        while (true) {
	            Message msg = queue.next(); // might block
	            //if (!me.mRun) {
	            //    break;
	            //}
	            if (msg != null) {
	                if (msg.target == null) {
	                    // No target is a magic identifier for the quit message.
	                    return;
	                }
	                if (me.mLogging!= null) me.mLogging.println(
	                        ">>>>> Dispatching to " + msg.target + " "
	                        + msg.callback + ": " + msg.what
	                        );
	                msg.target.dispatchMessage(msg);
	                if (me.mLogging!= null) me.mLogging.println(
	                        "<<<<< Finished to    " + msg.target + " "
	                        + msg.callback);
	                
	                // Make sure that during the course of dispatching the
	                // identity of the thread wasn't corrupted.
	                final long newIdent = Binder.clearCallingIdentity();
	                if (ident != newIdent) {
	                    Log.wtf("Looper", "Thread identity changed from 0x"
	                            + Long.toHexString(ident) + " to 0x"
	                            + Long.toHexString(newIdent) + " while dispatching to "
	                            + msg.target.getClass().getName() + " "
	                            + msg.callback + " what=" + msg.what);
	                }
	                
	                msg.recycle();
	            }
	        }
	    }

while(true)体现了消息循环中的“循环“，Looper会在循环体中调用queue.next()获取消息队列中需要处理的下一条消息。当msg != null且msg.target != null时，调用msg.target.dispatchMessage(msg);分发消息，当分发完成后，调用msg.recycle();回收消息。

msg.target是一个handler对象，表示需要处理这个消息的handler对象。Handler的void dispatchMessage(Message msg)方法如下：

Handler.java

	public void dispatchMessage(Message msg) {
	        if (msg.callback != null) {
	            handleCallback(msg);
	        } else {
	            if (mCallback != null) {
	                if (mCallback.handleMessage(msg)) {
	                    return;
	                }
	            }
	            handleMessage(msg);
	        }
	}

可见，当msg.callback== null 并且mCallback == null时，这个例子是由handleMessage(msg);处理消息，上面我们说到子类覆盖这个方法可以实现消息的具体处理过程。


总结：从上面的分析过程可知，消息循环的核心是Looper，Looper持有消息队列MessageQueue对象，一个线程可以把Looper设为该线程的局部变量，这就相当于这个线程建立了一个对应的消息队列。Handler的作用就是封装发送消息和处理消息的过程，让其他线程只需要操作Handler就可以发消息给创建Handler的线程。由此可以知道，在上一篇《Android异步处理一：使用Thread+Handler实现非UI线程更新UI界面》中，UI线程在创建的时候就建立了消息循环(在ActivityThread的public static final void main(String[] args)方法中实现)，因此我们可以在其他线程给UI线程的handler发送消息，达到更新UI的目的。

===

#Android异步处理四：AsyncTask的实现原理

概述：AsyncTask的本质是一个线程池，所有提交的异步任务都会在这个线程池中的工作线程内执行，当工作线程需要跟UI线程交互时，工作线程会通过向在UI线程创建的Handler（原理见：《Android异步处理三：Handler+Looper+MessageQueue深入详解》）传递消息的方式，调用相关的回调函数，从而实现UI界面的更新。


例子：

本章还是以《Android异步处理二：使用AsyncTask异步更新UI界面》中的例子说明AsyncTask的实现原理。

这个例子是在后台下载CSDN的LOGO，下载完成后在UI界面上显示出来。

AsyncTask.java

	package com.zhuozhuo;

	import org.apache.http.HttpResponse;
	import org.apache.http.client.HttpClient;
	import org.apache.http.client.methods.HttpGet;
	import org.apache.http.impl.client.DefaultHttpClient;
	
	import android.app.Activity;
	import android.graphics.Bitmap;
	import android.graphics.BitmapFactory;
	import android.os.AsyncTask;
	import android.os.Bundle;
	import android.view.View;
	import android.view.View.OnClickListener;
	import android.widget.Button;
	import android.widget.ImageView;
	import android.widget.ProgressBar;
	import android.widget.Toast;
	
	public class AsyncTaskActivity extends Activity {
	    
		private ImageView mImageView;
		private Button mButton;
		private ProgressBar mProgressBar;
		
	    @Override
	    public void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.main);
	        
	        mImageView= (ImageView) findViewById(R.id.imageView);
	        mButton = (Button) findViewById(R.id.button);
	        mProgressBar = (ProgressBar) findViewById(R.id.progressBar);
	        mButton.setOnClickListener(new OnClickListener() {
				
				@Override
				public void onClick(View v) {
					GetCSDNLogoTask task = new GetCSDNLogoTask();
					task.execute("http://csdnimg.cn/www/images/csdnindex_logo.gif");
				}
			});
	    }
	    
	    class GetCSDNLogoTask extends AsyncTask<String,Integer,Bitmap> {//继承AsyncTask
	
			@Override
			protected Bitmap doInBackground(String... params) {//处理后台执行的任务，在后台线程执行
				publishProgress(0);//将会调用onProgressUpdate(Integer... progress)方法
				HttpClient hc = new DefaultHttpClient();
				publishProgress(30);
				HttpGet hg = new HttpGet(params[0]);//获取csdn的logo
				final Bitmap bm;
				try {
					HttpResponse hr = hc.execute(hg);
					bm = BitmapFactory.decodeStream(hr.getEntity().getContent());
				} catch (Exception e) {
					
					return null;
				}
				publishProgress(100);
				//mImageView.setImageBitmap(result); 不能在后台线程操作ui
				return bm;
			}
			
			protected void onProgressUpdate(Integer... progress) {//在调用publishProgress之后被调用，在ui线程执行
				mProgressBar.setProgress(progress[0]);//更新进度条的进度
		     }
	
		     protected void onPostExecute(Bitmap result) {//后台任务执行完之后被调用，在ui线程执行
		    	 if(result != null) {
		    		 Toast.makeText(AsyncTaskActivity.this, "成功获取图片", Toast.LENGTH_LONG).show();
		    		 mImageView.setImageBitmap(result);
		    	 }else {
		    		 Toast.makeText(AsyncTaskActivity.this, "获取图片失败", Toast.LENGTH_LONG).show();
		    	 }
		     }
		     
		     protected void onPreExecute () {//在 doInBackground(Params...)之前被调用，在ui线程执行
		    	 mImageView.setImageBitmap(null);
		    	 mProgressBar.setProgress(0);//进度条复位
		     }
		     
		     protected void onCancelled () {//在ui线程执行
		    	 mProgressBar.setProgress(0);//进度条复位
		     }
	    	
	    }
	    
	
	}

main.xml

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
		android:orientation="vertical" android:layout_width="fill_parent"
		android:layout_height="fill_parent">
	    <ProgressBar android:layout_width="fill_parent" android:layout_height="wrap_content" android:id="@+id/progressBar" style="?android:attr/progressBarStyleHorizontal"></ProgressBar>
	    <Button android:id="@+id/button" android:text="下载图片" android:layout_width="wrap_content" android:layout_height="wrap_content"></Button>
		<ImageView android:id="@+id/imageView" android:layout_height="wrap_content"
			android:layout_width="wrap_content" />
	</LinearLayout>

mainifest.xml

	<?xml version="1.0" encoding="utf-8"?>
	<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	      package="com.zhuozhuo"
	      android:versionCode="1"
	      android:versionName="1.0">
	    <uses-sdk android:minSdkVersion="10" />
	<uses-permission android:name="android.permission.INTERNET"></uses-permission>
	    <application android:icon="@drawable/icon" android:label="@string/app_name">
	        <activity android:name=".AsyncTaskActivity"
	                  android:label="@string/app_name">
	            <intent-filter>
	                <action android:name="android.intent.action.MAIN" />
	                <category android:name="android.intent.category.LAUNCHER" />
	            </intent-filter>
	        </activity>
	
	    </application>
	</manifest>
	
运行结果：

![image](/img/Android异步处理/4.jpg)

分析：

在分析实现流程之前，我们先了解一下AsyncTask有哪些成员变量。

	   private static final int CORE_POOL_SIZE =5;//5个核心工作线程
	   private static final int MAXIMUM_POOL_SIZE = 128;//最多128个工作线程
	   private static final int KEEP_ALIVE = 1;//空闲线程的超时时间为1秒
	 
	   private static final BlockingQueue<Runnable> sWorkQueue =
	           new LinkedBlockingQueue<Runnable>(10);//等待队列
	 
	   private static final ThreadPoolExecutorsExecutor = new ThreadPoolExecutor(CORE_POOL_SIZE,
	           MAXIMUM_POOL_SIZE, KEEP_ALIVE, TimeUnit.SECONDS, sWorkQueue,sThreadFactory);//线程池是静态变量，所有的异步任务都会放到这个线程池的工作线程内执行。
	           
回到例子中，点击按钮之后会新建一个GetCSDNLogoTask对象：

	GetCSDNLogoTask task = new GetCSDNLogoTask();
	
此时会调用父类AsyncTask的构造函数：

AsyncTask.java

	public AsyncTask() {
	        mWorker = new WorkerRunnable<Params, Result>() {
	            public Result call() throws Exception {
	                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
	                return doInBackground(mParams);
	            }
	        };
	
	        mFuture = new FutureTask<Result>(mWorker) {
	            @Override
	            protected void done() {
	                Message message;
	                Result result = null;
	
	                try {
	                    result = get();
	                } catch (InterruptedException e) {
	                    android.util.Log.w(LOG_TAG, e);
	                } catch (ExecutionException e) {
	                    throw new RuntimeException("An error occured while executing doInBackground()",
	                            e.getCause());
	                } catch (CancellationException e) {
	                    message = sHandler.obtainMessage(MESSAGE_POST_CANCEL,
	                            new AsyncTaskResult<Result>(AsyncTask.this, (Result[]) null));
	                    message.sendToTarget();//取消任务，发送MESSAGE_POST_CANCEL消息
	                    return;
	                } catch (Throwable t) {
	                    throw new RuntimeException("An error occured while executing "
	                            + "doInBackground()", t);
	                }
	
	                message = sHandler.obtainMessage(MESSAGE_POST_RESULT,
	                        new AsyncTaskResult<Result>(AsyncTask.this, result));//完成任务，发送MESSAGE_POST_RESULT消息并传递result对象
	                message.sendToTarget();
	            }
	        };
	    }

WorkerRunnable类实现了callable接口的call()方法，该函数会调用我们在AsyncTask子类中实现的doInBackground(mParams)方法，由此可见，WorkerRunnable封装了我们要执行的异步任务。FutureTask中的protected void done() {}方法实现了异步任务状态改变后的操作。当异步任务被取消，会向UI线程传递MESSAGE_POST_CANCEL消息，当任务成功执行，会向UI线程传递MESSAGE_POST_RESULT消息，并把执行结果传递到UI线程。

由此可知，AsyncTask在构造的时候已经定义好要异步执行的方法doInBackground(mParams)和任务状态变化后的操作（包括失败和成功）。

当创建完GetCSDNLogoTask对象后，执行

	task.execute("http://csdnimg.cn/www/images/csdnindex_logo.gif");  
	
此时会调用AsyncTask的execute(Params...params)方法

AsyncTask.java

	public final AsyncTask<Params,Progress, Result> execute(Params... params) {
	        if (mStatus != Status.PENDING) {
	            switch (mStatus) {
	                case RUNNING:
	                    throw newIllegalStateException("Cannot execute task:"
	                            + " the taskis already running.");
	                case FINISHED:
	                    throw newIllegalStateException("Cannot execute task:"
	                            + " the taskhas already been executed "
	                            + "(a task canbe executed only once)");
	            }
	        }
	 
	        mStatus = Status.RUNNING;
	 
	        onPreExecute();//运行在ui线程，在提交任务到线程池之前执行
	 
	        mWorker.mParams = params;
	        sExecutor.execute(mFuture);//提交任务到线程池
	 
	        return this;
	}
	
当任务正在执行或者已经完成，会抛出IllegalStateException，由此可知我们不能够重复调用execute(Params...params)方法。在提交任务到线程池之前，调用了onPreExecute()方法。然后才执行sExecutor.execute(mFuture)是任务提交到线程池。

前面我们说到，当任务的状态发生改变时（1、执行成功2、取消执行3、进度更新），工作线程会向UI线程的Handler传递消息。在《Android异步处理三：Handler+Looper+MessageQueue深入详解》一文中我们提到，Handler要处理其他线程传递过来的消息。在AsyncTask中，InternalHandler是在UI线程上创建的，它接收来自工作线程的消息，实现代码如下：

AsyncTask.java

	private static class InternalHandler extends Handler {
	       @SuppressWarnings({"unchecked","RawUseOfParameterizedType"})
	        @Override
	        public voidhandleMessage(Message msg) {
	            AsyncTaskResult result =(AsyncTaskResult) msg.obj;
	            switch (msg.what) {
	                caseMESSAGE_POST_RESULT:
	                    // There is onlyone result
	                    result.mTask.finish(result.mData[0]);//执行任务成功
	                    break;
	                caseMESSAGE_POST_PROGRESS:
	                   result.mTask.onProgressUpdate(result.mData);//进度更新
	                    break;
	                caseMESSAGE_POST_CANCEL:
	                    result.mTask.onCancelled();//取消任务
	                    break;
	            }
	        }
	    }
	    
当接收到消息之后，AsyncTask会调用自身相应的回调方法。

总结：

1. AsyncTask的本质是一个静态的线程池，AsyncTask派生出的子类可以实现不同的异步任务，这些任务都是提交到静态的线程池中执行。

2. 线程池中的工作线程执行doInBackground(mParams)方法执行异步任务

3. 当任务状态改变之后，工作线程会向UI线程发送消息，AsyncTask内部的InternalHandler响应这些消息，并调用相关的回调函数