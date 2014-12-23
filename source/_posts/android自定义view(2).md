title: android自定义View(2)
date: 2014-12-10 00:53:12

---
转载请标明出处：http://blog.csdn.net/lmj623565791/article/details/24252901

很多的Android入门程序猿来说对于Android自定义View，可能都是比较恐惧的，但是这又是高手进阶的必经之路，所有准备在自定义View上面花一些功夫，多写一些文章。先总结下自定义View的步骤：

<!--more-->

1. 自定义View的属性

2. 在View的构造方法中获得我们自定义的属性

3. 重写onMesure

4. 重写onDraw

3不一定是必须的，当然了大部分情况下还是需要重写的。

1、自定义View的属性，首先在res/values/  下建立一个attrs.xml ， 在里面定义我们的属性和声明我们的整个样式。

	<?xml version="1.0" encoding="utf-8"?>  
	<resources>  
	  
	    <attr name="titleText" format="string" />  
	    <attr name="titleTextColor" format="color" />  
	    <attr name="titleTextSize" format="dimension" />  
	  
	    <declare-styleable name="CustomTitleView">  
	        <attr name="titleText" />  
	        <attr name="titleTextColor" />  
	        <attr name="titleTextSize" />  
	    </declare-styleable>  
	  
	</resources>  
	
我们定义了字体，字体颜色，字体大小3个属性，format是值该属性的取值类型:
一共有：string,color,demension,integer,enum,reference,float,boolean,fraction,flag;不清楚的可以google一把。

然后在布局中声明我们的自定义View

	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"  
	    xmlns:tools="http://schemas.android.com/tools"  
	    xmlns:custom="http://schemas.android.com/apk/res/com.example.customview01"  
	    android:layout_width="match_parent"  
	    android:layout_height="match_parent" >  
	  
	    <com.example.customview01.view.CustomTitleView  
	        android:layout_width="200dp"  
	        android:layout_height="100dp"  
	        custom:titleText="3712"  
	        custom:titleTextColor="#ff0000"  
	        custom:titleTextSize="40sp" />  
	  
	</RelativeLayout>  
	
一定要引入 xmlns:custom="http://schemas.android.com/apk/res/com.example.customview01"我们的命名空间，后面的包路径指的是项目的package

2、在View的构造方法中，获得我们的自定义的样式	

	/** 
     * 文本 
     */  
    private String mTitleText;  
    /** 
     * 文本的颜色 
     */  
    private int mTitleTextColor;  
    /** 
     * 文本的大小 
     */  
    private int mTitleTextSize;  
  
    /** 
     * 绘制时控制文本绘制的范围 
     */  
    private Rect mBound;  
    private Paint mPaint;  
  
    public CustomTitleView(Context context, AttributeSet attrs)  
    {  
        this(context, attrs, 0);  
    }  
  
    public CustomTitleView(Context context)  
    {  
        this(context, null);  
    }  
  
    /** 
     * 获得我自定义的样式属性 
     *  
     * @param context 
     * @param attrs 
     * @param defStyle 
     */  
    public CustomTitleView(Context context, AttributeSet attrs, int defStyle)  
    {  
        super(context, attrs, defStyle);  
        /** 
         * 获得我们所定义的自定义样式属性 
         */  
        TypedArray a = context.getTheme().obtainStyledAttributes(attrs, R.styleable.CustomTitleView, defStyle, 0);  
        int n = a.getIndexCount();  
        for (int i = 0; i < n; i++)  
        {  
            int attr = a.getIndex(i);  
            switch (attr)  
            {  
            case R.styleable.CustomTitleView_titleText:  
                mTitleText = a.getString(attr);  
                break;  
            case R.styleable.CustomTitleView_titleTextColor:  
                // 默认颜色设置为黑色  
                mTitleTextColor = a.getColor(attr, Color.BLACK);  
                break;  
            case R.styleable.CustomTitleView_titleTextSize:  
                // 默认设置为16sp，TypeValue也可以把sp转化为px  
                mTitleTextSize = a.getDimensionPixelSize(attr, (int) TypedValue.applyDimension(  
                        TypedValue.COMPLEX_UNIT_SP, 16, getResources().getDisplayMetrics()));  
                break;  
  
            }  
  
        }  
        a.recycle();  
  
        /** 
         * 获得绘制文本的宽和高 
         */  
        mPaint = new Paint();  
        mPaint.setTextSize(mTitleTextSize);  
        // mPaint.setColor(mTitleTextColor);  
        mBound = new Rect();  
        mPaint.getTextBounds(mTitleText, 0, mTitleText.length(), mBound);  
  
    }  
    
我们重写了3个构造方法，默认的布局文件调用的是两个参数的构造方法，所以记得让所有的构造调用我们的三个参数的构造，我们在三个参数的构造中获得自定义属性。

3、我们重写onDraw，onMesure调用系统提供的：

	@Override  
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)  
    {  
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);  
    }  
  
    @Override  
    protected void onDraw(Canvas canvas)  
    {  
        mPaint.setColor(Color.YELLOW);  
        canvas.drawRect(0, 0, getMeasuredWidth(), getMeasuredHeight(), mPaint);  
  
        mPaint.setColor(mTitleTextColor);  
        canvas.drawText(mTitleText, getWidth() / 2 - mBound.width() / 2, getHeight() / 2 + mBound.height() / 2, mPaint);  
    }  
    
此时的效果是：

![image](/img/android自定义View(2)/1.jpg)

是不是觉得还不错，基本已经实现了自定义View。但是此时如果我们把布局文件的宽和高写成wrap_content，会发现效果并不是我们的预期：

![image](/img/android自定义View(2)/2.jpg)

系统帮我们测量的高度和宽度都是MATCH_PARNET，当我们设置明确的宽度和高度时，系统帮我们测量的结果就是我们设置的结果，当我们设置为WRAP_CONTENT,或者MATCH_PARENT系统帮我们测量的结果就是MATCH_PARENT的长度。

所以，当设置了WRAP_CONTENT时，我们需要自己进行测量，即重写onMesure方法”：

重写之前先了解MeasureSpec的specMode,一共三种类型：

EXACTLY：一般是设置了明确的值或者是MATCH_PARENT

AT_MOST：表示子布局限制在一个最大值内，一般为WARP_CONTENT

UNSPECIFIED：表示子布局想要多大就多大，很少使用

下面是我们重写onMeasure代码：

	@Override  
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)  
	{  
	    int widthMode = MeasureSpec.getMode(widthMeasureSpec);  
	    int widthSize = MeasureSpec.getSize(widthMeasureSpec);  
	    int heightMode = MeasureSpec.getMode(heightMeasureSpec);  
	    int heightSize = MeasureSpec.getSize(heightMeasureSpec);  
	    int width;  
	    int height ;  
	    if (widthMode == MeasureSpec.EXACTLY)  
	    {  
	        width = widthSize;  
	    } else  
	    {  
	        mPaint.setTextSize(mTitleTextSize);  
	        mPaint.getTextBounds(mTitle, 0, mTitle.length(), mBounds);  
	        float textWidth = mBounds.width();  
	        int desired = (int) (getPaddingLeft() + textWidth + getPaddingRight());  
	        width = desired;  
	    }  
	  
	    if (heightMode == MeasureSpec.EXACTLY)  
	    {  
	        height = heightSize;  
	    } else  
	    {  
	        mPaint.setTextSize(mTitleTextSize);  
	        mPaint.getTextBounds(mTitle, 0, mTitle.length(), mBounds);  
	        float textHeight = mBounds.height();  
	        int desired = (int) (getPaddingTop() + textHeight + getPaddingBottom());  
	        height = desired;  
	    }  
	      
	      
	  
	    setMeasuredDimension(width, height);  
	}  
	
现在我们修改下布局文件：

	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"  
	    xmlns:tools="http://schemas.android.com/tools"  
	    xmlns:custom="http://schemas.android.com/apk/res/com.example.customview01"  
	    android:layout_width="match_parent"  
	    android:layout_height="match_parent" >  
	  
	    <com.example.customview01.view.CustomTitleView  
	        android:layout_width="wrap_content"  
	        android:layout_height="wrap_content"  
	        custom:titleText="3712"  
	        android:padding="10dp"  
	        custom:titleTextColor="#ff0000"  
	        android:layout_centerInParent="true"  
	        custom:titleTextSize="40sp" />  
	  
	</RelativeLayout>  
	
现在的效果是：

![image](/img/android自定义View(2)/3.jpg)

完全复合我们的预期，现在我们可以对高度、宽度进行随便的设置了，基本可以满足我们的需求。

当然了，这样下来我们这个自定义View与TextView相比岂不是没什么优势，所有我们觉得给自定义View添加一个事件：

在构造中添加：

	this.setOnClickListener(new OnClickListener()  
        {  
  
            @Override  
            public void onClick(View v)  
            {  
                mTitleText = randomText();  
                postInvalidate();  
            }  
  
        });  
        
	private String randomText()  
	    {  
        Random random = new Random();  
        Set<Integer> set = new HashSet<Integer>();  
        while (set.size() < 4)  
        {  
            int randomInt = random.nextInt(10);  
            set.add(randomInt);  
        }  
        StringBuffer sb = new StringBuffer();  
        for (Integer i : set)  
        {  
            sb.append("" + i);  
        }  
  
        return sb.toString();  
    }  
    
下面再来运行：

![image](/img/android自定义View(2)/4.gif)

我们添加了一个点击事件，每次让它随机生成一个4位的随机数，有兴趣的可以在onDraw中添加一点噪点，然后改写为验证码，是不是感觉很不错。


好了，各位学习的，打酱油的留个言，顶个呗~