title: android自定义View
date: 2014-12-10 00:47:57

---
哈哈，兄弟我终于自己写了一个view，不是网上那种简单的哦，还是有一定技术含量的，
我是通过学习ApiDemo（android自带的sample）里面LabelView实现的，
先谈谈学习过程，觉得一开始不应当盲目的动手做，应对想把原理搞明白，哪怕一个很小的View，也应当将各个细节弄清楚，
等这些搞定了，接下来的工作就是水道渠成了！

<!--more-->

自定义一个View那必须继承View，
首先说说我的View是啥，恩，很简单，就是一个椭圆，其中可以设置4个参数，分别是top、left、right、bottom，应该很清楚吧，因为canvas.drawOval时用到了4个值，


首先得定义4个属性值，是我的View专有的，
建立attr.xml放到values下面

	<?xml version="1.0" encoding="UTF-8"?>
	<resources>
	     <declare-styleable name="RocketView">
	        <attr name="ovalLeft" format="dimension" />
	        <attr name="ovalTop" format="dimension" />
	        <attr name="ovalRight" format="dimension" />
	        <attr name="ovalBottom" format="dimension" />
	    </declare-styleable>
	</resources>
	
然后写个RocketView

	package com.myos;
	
	import android.content.Context;
	import android.content.res.TypedArray;
	import android.graphics.Canvas;
	import android.graphics.Color;
	import android.graphics.Paint;
	import android.graphics.RectF;
	import android.util.AttributeSet;
	import android.util.Log;
	import android.view.View;
	import android.view.View.MeasureSpec;
	
	public class RocketView extends View{
		private Paint mOvalPaint;
		private int mStrokeWidth = 2;
		private int padding = 3;
		
		//椭圆参数
		private int mOval_l;
		private int mOval_t;
		private int mOval_r;
		private int mOval_b;
		
		
		
		//构造
		public RocketView(Context context, AttributeSet attrs) {
			super(context, attrs);
			initRocketView();
			
			TypedArray a = context.obtainStyledAttributes(attrs,
	                R.styleable.RocketView);
	
	        mOval_l = a.getDimensionPixelOffset(R.styleable.RocketView_ovalLeft, padding);  
	        mOval_t = a.getDimensionPixelOffset(R.styleable.RocketView_ovalTop, padding);  
	        mOval_r = a.getDimensionPixelOffset(R.styleable.RocketView_ovalRight, 100);  
	        mOval_b = a.getDimensionPixelOffset(R.styleable.RocketView_ovalBottom, 100);  
	
	        a.recycle();
		}
	
	    private void initRocketView() {
	    	mOvalPaint = new Paint();
	    	mOvalPaint.setAntiAlias(true);
	    	mOvalPaint.setColor(Color.BLUE);
	    	mOvalPaint.setStyle(Paint.Style.STROKE);
	    	mOvalPaint.setStrokeWidth(mStrokeWidth);
	    	setPadding(padding,padding,padding,padding);
	    }
	    
	    public void setOvalRect(int l, int t, int r, int b){
	    	mOval_l = l + padding;
	    	mOval_t = t + padding;
	    	mOval_r = r;
	    	mOval_b = b;
	    	requestLayout();
	        invalidate();
	    }
		@Override
		protected void onDraw(Canvas canvas) {
			// TODO Auto-generated method stub
			super.onDraw(canvas);
			canvas.drawColor(Color.WHITE);
			// 绘制椭圆
			RectF re11 = new RectF(mOval_l, mOval_t, mOval_r, mOval_b);
			canvas.drawOval(re11, mOvalPaint);	
			
	//		// 绘制圆形
	//		canvas.drawCircle(mCircle_x, mCircle_y, mCircle_r, mPaint);
		}
		
		 /**
		  * @see android.view.View#measure(int, int)
		  */
		 @Override
		 protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
		     setMeasuredDimension(measureWidth(widthMeasureSpec),
		             measureHeight(heightMeasureSpec));
		 }
	
		 /**
		  * Determines the width of this view
		  * @param measureSpec A measureSpec packed into an int
		  * @return The width of the view, honoring constraints from measureSpec
		  */
		 private int measureWidth(int measureSpec) {
		     int result = 0;
		     int specMode = MeasureSpec.getMode(measureSpec);
		     int specSize = MeasureSpec.getSize(measureSpec);
	
		     if (specMode == MeasureSpec.EXACTLY) {
		         // We were told how big to be
		         result = specSize;
		     } else {
		         // Measure the text
		         result = mOval_r + getPaddingLeft()
		                 + getPaddingRight();
		         if (specMode == MeasureSpec.AT_MOST) {
		             // Respect AT_MOST value if that was what is called for by measureSpec
		             result = Math.min(result, specSize);
		         }
		     }
			 
		     return result;
		 }
	
		 /**
		  * Determines the height of this view
		  * @param measureSpec A measureSpec packed into an int
		  * @return The height of the view, honoring constraints from measureSpec
		  */
		 private int measureHeight(int measureSpec) {
		     int result = 0;
		     int specMode = MeasureSpec.getMode(measureSpec);
		     int specSize = MeasureSpec.getSize(measureSpec);
	
		     if (specMode == MeasureSpec.EXACTLY) {
		         // We were told how big to be
		         result = specSize;
		     } else {
		         // Measure the text (beware: ascent is a negative number)
		         result = mOval_b + getPaddingTop()
		                 + getPaddingBottom();
		         if (specMode == MeasureSpec.AT_MOST) {
		             // Respect AT_MOST value if that was what is called for by measureSpec
		             result = Math.min(result, specSize);
		         }
		     }
		     
		     return result;
		 }
	}

在构造中getDimensionPixelOffset检索出一个属性值，没有的话就使用第2个参数做默认值，
可以在布局xml中初始化这个属性值，下面是我main.xml

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:app="http://schemas.android.com/apk/res/com.myos"
	    android:orientation="vertical"
	    android:layout_width="fill_parent"
	    android:layout_height="fill_parent"
	    >
		
		<TextView  
		    android:layout_width="wrap_content" 
		    android:layout_height="wrap_content" 
		    android:text="@string/hello"
		    />
		
		<com.myos.RocketView
		    app:ovalLeft="0dp"
		    android:id="@+id/rv"
		    android:layout_width="wrap_content"
		    android:layout_height="wrap_content" />
	
	</LinearLayout>

我这里没啥用处，仅仅试一下，待扩展属性时再说，说明一下xmlns:app中app可以随便命名的，这个android的规矩还真多
另外构造函数中onMeasure很重要的，onDraw很明显就是画椭圆了，android在画的时候呢，会去先测量，需要我们来提供View的宽和高，

大家都知道wrap_content、fill_parent，测量时会有3种模式，分别是UNSPECIFIED、EXACTLY、AT_MOST，
当使用
wrap_content时就是AT_MOST模式，fill_parent就是EXACTLY模式，可以看看官方文档：
http://developer.android.com/guide/topics/ui/how-android-draws.html 
相信研读一下代码就明白了

RocketView定义好了，接下来就是使用了

	package com.myos;
	
	import android.app.Activity;
	import android.os.Bundle;
	
	public class MainActivity extends Activity {
		RocketView mRocketView;
	    /** Called when the activity is first created. */
	    @Override
	    public void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.main);
	        
	        mRocketView = (RocketView)findViewById(R.id.rv);
	        mRocketView.setOvalRect(0, 0, 500, 100);
	    }
	}

可以通过setOvalRect函数来动态调整椭圆的大小！

下面是运行效果，ko啦！

![image](/img/android自定义View/1.png)