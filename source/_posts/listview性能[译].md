title: 'ListView性能[译]'
date: 2014-12-10 01:14:21

---
ListView是一种可以显示一系列项目并能进行滚动显示的View。在每行里，既可以是简单的文本，也可以是复杂的结构。一般情况下，你都需要保证ListView运行得很好（即：渲染更快，滚动流畅）。在接下来的内容里，我将就ListView的使用，向大家提供几种解决不同性能问题的解决方案。

<!--more-->

如果你想使用ListView，你就不得不使用ListAdapter来显示内容。SDK中，已经有了几种简单实现的Adapter：

1. ArrayAdapter<T> （显示数组对象，使用toString()来显示）
2. SimpleAdapter （显示Maps列表）
3. SimpleCursorAdapter（显示通过Cursor从DB中获取的信息）

这些实现对于显示简单的列表来说，非常棒！一旦你的列表比较复杂，你就不得不书写自己的ListAdapter实现。在多数情况下，直接从ArrayAdapter扩展就能很好地处理一组对象。此时，你需要处理的工作只是告诉系统如何处理列表中的对象。通过重写getView(int, View, ViewGroup)方法即可达到。

在这里，举一个你需要自定义ListAdapter的例子：显示一组图片，图片的旁边有文字挨着。

ListView例：以图片文字方式显示的Youtube搜索结果

图片需要实时从internet上下载下来。让我们先创建一个Class来代表列表中的项目：

	publicclassImageAndText {
	    privateString imageUrl;
	    privateString text;
	    publicImageAndText(String imageUrl, String text) {
	        this.imageUrl = imageUrl;
	        this.text = text;
	    }
	    publicString getImageUrl() {
	        returnimageUrl;
	    }
	    publicString getText() {
	        returntext;
	    }
	}
	
现在，我们要实现一个ListAdapter，来显示ImageAndText列表。

	publicclassImageAndTextListAdapter extendsArrayAdapter<ImageAndText> {
	publicImageAndTextListAdapter(Activity activity, List<ImageAndText> imageAndTexts) {
	        super(activity, 0, imageAndTexts);
	    }
	    @Override
	    publicView getView(intposition, View convertView, ViewGroup parent) {
	        Activity activity = (Activity) getContext();
	        LayoutInflater inflater = activity.getLayoutInflater();
	        // Inflate the views from XML
	        View rowView = inflater.inflate(R.layout.image_and_text_row, null);
	        ImageAndText imageAndText = getItem(position);
	        // Load the image and set it on the ImageView
	        ImageView imageView = (ImageView) rowView.findViewById(R.id.image);
	        imageView.setImageDrawable(loadImageFromUrl(imageAndText.getImageUrl()));
	        // Set the text on the TextView
	        TextView textView = (TextView) rowView.findViewById(R.id.text);
	        textView.setText(imageAndText.getText());
	        returnrowView;
	    }
	    publicstaticDrawable loadImageFromUrl(String url) {
	        InputStream inputStream;
	        try{
	            inputStream = newURL(url).openStream();
	        } catch(IOException e) {
	            thrownewRuntimeException(e);
	        }
	        returnDrawable.createFromStream(inputStream, "src");
	}
	}
	
这些View都是从“image_and_text_row.xml”XML文件中inflate的：

	<?xmlversion="1.0"encoding="utf-8"?>
	<LinearLayoutxmlns:android="http://schemas.android.com/apk/res/android"
	              android:orientation="horizontal"
	              android:layout_width="fill_parent"
	              android:layout_height="wrap_content">
	        <ImageViewandroid:id="@+id/image"
	                   android:layout_width="wrap_content"
	                   android:layout_height="wrap_content"
	                   android:src="@drawable/default_image"/>
	        <TextViewandroid:id="@+id/text"
	                  android:layout_width="wrap_content"
	                  android:layout_height="wrap_content"/>
	</LinearLayout>
	
这个ListAdapter实现正如你所期望的那样，能在ListView中加载ImageAndText。但是，它唯一可用的场合是那些拥有很少项目、无需滚动即可看到全部的列表。如果ImageAndText列表内容很多的时候，你会看到，滚动起来不是那么的平滑（事实上，远远不是）。

性能改善

上面例子最大的瓶颈是图片需要从internet上下载。因为我们的代码都在UI线程中执行，所以，每当一张图片从网络上下载时，UI就会变得停滞。如果你用3G网络代替WiFi的话，性能情况会变得更糟。

为了避免这种情况，我们想让图片的下载处于单独的线程里，这样就不会过多地占用UI线程。为了达到这一目的，我们可能需要使用为这种情况特意设计的AsyncTask。实际情况中，你将注意到AsyncTask被限制在10个以内。这个数量是在Android SDK中硬编码的，所以我们无法改变。这对我们来说是一个制限事项，因为常常有超过10个图片同时在下载。

AsyncImageLoader

一个变通的做法是手动的为每个图片创建一个线程。另外，我们还应该使用Handler来将下载的图片invoke到UI线程。我们这样做的原因是我们只能在UI线程中修改UI。我创建了一个AsyncImageLoader类，利用线程和Handler来负责图片的下载。此外，它还缓存了图片，防止单个图片被下载多次。

	publicclassAsyncImageLoader {
	    privateHashMap<String, SoftReference<Drawable>> imageCache;
	    publicAsyncImageLoader() {
	        imageCache = newHashMap<String, SoftReference<Drawable>>();
	    }
	    publicDrawable loadDrawable(finalString imageUrl, finalImageCallback imageCallback) {
	        if(imageCache.containsKey(imageUrl)) {
	            SoftReference<Drawable> softReference = imageCache.get(imageUrl);
	            Drawable drawable = softReference.get();
	            if(drawable != null) {
	                returndrawable;
	            }
	        }
	        finalHandler handler = newHandler() {
	            @Override
	            publicvoidhandleMessage(Message message) {
	                imageCallback.imageLoaded((Drawable) message.obj, imageUrl);
	            }
	        };
	        newThread() {
	            @Override
	            publicvoidrun() {
	                Drawable drawable = loadImageFromUrl(imageUrl);
	                imageCache.put(imageUrl, newSoftReference<Drawable>(drawable));
	                Message message = handler.obtainMessage(0, drawable);
	                handler.sendMessage(message);
	            }
	        }.start();
	        returnnull;
	    }
	    publicstaticDrawable loadImageFromUrl(String url) {
	        // ...
	    }
	    publicinterfaceImageCallback {
	        publicvoidimageLoaded(Drawable imageDrawable, String imageUrl);
	}
	}
	
注意：我使用了SoftReference来缓存图片，允许GC在需要的时候可以对缓存中的图片进行清理。它这样工作：

1. 调用loadDrawable(ImageUrl, imageCallback)，传入一个匿名实现的ImageCallback接口
2. 如果图片在缓存中不存在的话，图片将从单一的线程中下载并在下载结束时通过ImageCallback回调
3. 如果图片确实存在于缓存中，就会马上返回，不会回调ImageCallback

在你的程序中，只能存在一个AsyncImageLoader实例，否则，缓存不能正常工作。在ImageAndTextListAdapter类中，我们可以这样替换：

	ImageView imageView = (ImageView) rowView.findViewById(R.id.image);
	imageView.setImageDrawable(loadImageFromUrl(imageAndText.getImageUrl()));
	
换成

	finalImageView imageView = (ImageView) rowView.findViewById(R.id.image);
	Drawable cachedImage = asyncImageLoader.loadDrawable(imageAndText.getImageUrl(), newImageCallback() {
	    publicvoidimageLoaded(Drawable imageDrawable, String imageUrl) {
	        imageView.setImageDrawable(imageDrawable);
	    }
	});
	
	imageView.setImageDrawable(cachedImage);
	
使用这个方法，ListView执行得很好了，并且感觉滑动更平滑了，因为UI线程再也不会被图片加载所阻塞。

更好的性能改善

如果你尝试了上面的解决方案，你将注意到ListView也不是100%的平滑，仍然会有些东西阻滞着它的平滑性。这里，还有两个地方可以进行改善：

1. findViewById()的昂贵调用
2. 每次都inflate XML

因此，修改代码如下：

	publicclassImageAndTextListAdapter extendsArrayAdapter<ImageAndText> {
	privateListView listView;
	    privateAsyncImageLoader asyncImageLoader;
	    publicImageAndTextListAdapter(Activity activity, List<ImageAndText> imageAndTexts, ListView listView) {
	        super(activity, 0, imageAndTexts);
	        this.listView = listView;
	asyncImageLoader = newAsyncImageLoader();
	    }
	    @Override
	    publicView getView(intposition, View convertView, ViewGroup parent) {
	        Activity activity = (Activity) getContext();
	        // Inflate the views from XML
	        View rowView = convertView;
	        ViewCache viewCache;
	        if(rowView == null) {
	            LayoutInflater inflater = activity.getLayoutInflater();
	            rowView = inflater.inflate(R.layout.image_and_text_row, null);
	            viewCache = newViewCache(rowView);
	            rowView.setTag(viewCache);
	        } else{
	            viewCache = (ViewCache) rowView.getTag();
	        }
	        ImageAndText imageAndText = getItem(position);
	        // Load the image and set it on the ImageView
	        String imageUrl = imageAndText.getImageUrl();
	        ImageView imageView = viewCache.getImageView();
	        imageView.setTag(imageUrl);
	        Drawable cachedImage = asyncImageLoader.loadDrawable(imageUrl, newImageCallback() {
	            publicvoidimageLoaded(Drawable imageDrawable, String imageUrl) {
	                ImageView imageViewByTag = (ImageView) listView.findViewWithTag(imageUrl);
	                if(imageViewByTag != null) {
	                    imageViewByTag.setImageDrawable(imageDrawable);
	                }
	            }
	        });
	        imageView.setImageDrawable(cachedImage);
	        // Set the text on the TextView
	        TextView textView = viewCache.getTextView();
	        textView.setText(imageAndText.getText());
	returnrowView;
	    }
	}
	
这里有两点需要注意：第一点是drawable不再是加载完毕后直接设定到ImageView上。正确的ImageView是通过tag查找的，这是因为我们现在重用了View，并且图片有可能出现在错误的行上。我们需要拥有一个ListView的引用来通过tag查找ImageView。


另外一点是，实现中我们使用了一个叫ViewCache的对象。它这样定义：

	publicclassViewCache {
	    privateView baseView;
	    privateTextView textView;
	    privateImageView imageView;
		publicViewCache(View baseView) {
			this.baseView = baseView;
	    }
	    publicTextView getTextView() {
	        if(textView == null) {
	            textView = (TextView) baseView.findViewById(R.id.text);
	        }
	        returntitleView;
	    }
	    publicImageView getImageView() {
	        if(imageView == null) {
	            imageView = (ImageView) baseView.findViewById(R.id.image);
	        }
	        returnimageView;
	    }
	}
	
有了ViewCache对象，我们就不需要使用findViewById()来多次查询View对象了。

总结

我已经向大家演示了3种改进ListView性能的方法：

1. 在单一线程里加载图片
2. 重用列表中行
3. 缓存行中的View

xirihanlin 2011.04.06