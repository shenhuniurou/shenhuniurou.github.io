---
layout: post
title: Volley的使用和源码解析
category: 源码解析
tags: Volley
---




## Volley简介

Volley 是 Google I/O 2013上发布的网络通信库，使网络通信更快、更简单、更健壮。Volley特别适合数据量不大但是通信频繁的场景，而对于大数据量的网络操作，比如说下载文件等，Volley的表现就会非常糟糕。Volley提供的功能有：

- JSON，图像等的异步下载；
- 网络请求的排序（scheduling）
- 网络请求的优先级处理
- 缓存
- 多级别取消请求
- 和Activity和生命周期的联动（Activity结束时同时取消所有网络请求）

## Volley的优点

- 体积小，使用Volley可以使用Volley.jar或者通过gradle导入依赖，全部只有42个类，只有100多k大小。
- 非常适合进行数据量不大，但通信频繁的网络操作。
- 可直接在主线程调用服务端并处理返回结果
- 可以取消请求，容易扩展，面向接口编程。
- 网络请求线程NetworkDispatcher默认开启了4个，可以优化，通过手机CPU数量。
- 通过使用标准的HTTP缓存机制保持磁盘和内存响应的一致。

## Volley的缺点

- 使用的是HttpClient、HttpURLConnection
- Android6.0不支持HttpClient了，如果想支持得添加org.apache.http.legacy.jar或者在app module中的build.gradle中加入useLibrary 'org.apache.http.legacy'
- 对大文件下载Volley的表现非常糟糕
- 只支持Http请求
- 图片加载性能一般


## Volley的基本用法

关于Volley的使用，官方文档的地址是[https://developer.android.com/training/volley/index.html](https://developer.android.com/training/volley/index.html)

Volley的用法非常简单，发起一条HTTP请求，然后接收HTTP响应。首先需要获取到一个RequestQueue对象，可以调用如下方法获取到：

```
RequestQueue queue = Volley.newRequestQueue(context);
```

RequestQueue是一个所有请求的队列对象，它可以缓存所有HTTP请求，它的内部有两个队列：

```
/** The cache triage queue. */
private final PriorityBlockingQueue<Request<?>> mCacheQueue = new PriorityBlockingQueue<Request<?>>();

/** The queue of requests that are actually going out to the network. */
private final PriorityBlockingQueue<Request<?>> mNetworkQueue = new PriorityBlockingQueue<Request<?>>();
```

缓存请求的队列和处理请求的队列都是优先级阻塞队列，RequestQueue是按照一定的算法并发地发出这些请求。RequestQueue内部的设计就是非常合适高并发的，因此我们不必为每一次HTTP请求都创建一个RequestQueue对象，这是非常浪费资源的，基本上在每一个需要和网络交互的Activity中创建一个RequestQueue对象就足够了。

接下来为了要发出一条HTTP请求，我们还需要创建一个StringRequest对象，如下所示：

```
String url ="http://shenhuniurou.com";
// Request a string response from the provided URL.
StringRequest stringRequest = new StringRequest(Request.Method.GET, url,
	new Response.Listener<String>() {
		@Override
		public void onResponse(String response) {
			
		}
	},
	new Response.ErrorListener() {
		@Override
		public void onErrorResponse(VolleyError error) {
		
		}
	}
);
```

StringRequest封装了一个请求，包括请求地址，请求方式，请求结果监听等。它有两个构造方法

```
public StringRequest(int method, String url, Listener<String> listener, ErrorListener errorListener) {
	super(method, url, errorListener);
	mListener = listener;
}

    
public StringRequest(String url, Listener<String> listener, ErrorListener errorListener) {
	this(Method.GET, url, listener, errorListener);
}
```

如果没传请求方式，那么默认是GET方式。最后把这个请求添加到RequestQueue中去。

```
// Add the request to the RequestQueue.
queue.add(stringRequest);
```

另外还需要在AndroidManifest.xml中添加用户权限

```
<uses-permission android:name="android.permission.INTERNET"/>
```

如果我们要使用POST方式并向服务器传递参数，那就必须在StringRequest的匿名类中重写getParams方法

```
StringRequest stringRequest = new StringRequest(Request.Method.POST, url, 
	new Response.Listener<String>() {
		@Override
		public void onResponse(String response) {
			
		}
	},
	new Response.ErrorListener() {
		@Override
		public void onErrorResponse(VolleyError error) {
			
		}
	}) {
	@Override protected Map<String, String> getParams() throws AuthFailureError {
		HashMap<String, String> params = new HashMap<>();
		params.put("key1", "value1");
		params.put("keyn", "keyn");
		return params;
	}
};
```



以上是使用StringRequest类来提交参数发起请求，StringRequest是继承自Request<String>的，Request<T>是一个泛型抽象类，除了StringRequest，Request还有一个直接子类JsonRequest，不过JsonRequest也是一个泛型抽象类，它有两个子类，JsonArrayRequest和JsonObjectRequest，用于请求JSON类型的数据，用法如下：

```
JsonObjectRequest jsonObjectRequest = new JsonObjectRequest(
	Request.Method.GET,
	"http://www.weather.com.cn/data/sk/101110101.html",
	null,
	new Response.Listener<JSONObject>() {
		@Override public void onResponse(JSONObject response) {
			mTextView.setText("Response is: "+ response.toString());
		}
	},
	new Response.ErrorListener() {
		@Override public void onErrorResponse(VolleyError error) {

		}
	});

queue.add(jsonObjectRequest);

JsonArrayRequest jsonArrayRequest = new JsonArrayRequest(
	Request.Method.POST, 
	url, 
	null,
	new Response.Listener<JSONArray>() {
		@Override public void onResponse(JSONArray response) {

		}
	}, 
	new Response.ErrorListener() {
	@Override public void onErrorResponse(VolleyError error) {
		
	}
});

queue.add(jsonArrayRequest);
```

> 注意，并不是所有格式的数据都可以使用这两个类来发起请求的，只有JSON格式的才可以，它提交的参数类型也是json格式的。


## 使用Volley加载网络图片

使用Volley加载网络图片使用的是Volley中的ImageRequest类来发起请求，ImageRequest同样是Request类，不过它的泛型是Bitmap，和StringRequest、JsonRequest用法类似，先创建一个RequestQueue对象，创建一个Request对象，然后将Request对象添加到RequestQueue中即可。

```
final ImageView imageView = (ImageView) findViewById(R.id.imageView);

ImageRequest imageRequest = new ImageRequest(
	"http://tuku.chengdu.cn/CHN_CommendationPic/20120319105921966875741.jpg",
	new Response.Listener<Bitmap>() {
		@Override public void onResponse(Bitmap response) {
			imageView.setImageBitmap(response);
		}
	},
	0,
	0,
	ImageView.ScaleType.FIT_XY,
	Bitmap.Config.RGB_565,
	new Response.ErrorListener() {
		@Override public void onErrorResponse(VolleyError error) {

		}
	}
);

queue.add(imageRequest);
```

ImageRequest的构造方法中有两个，其中有一个已经弃用了，我们看到一共有七个参数，第一个url是图片的地址，第二个参数是加载图片成功后的回调，里面返回的Bitmap位图，第三个maxWidth和第四个maxHeight是用于指定允许图片最大的宽度和高度，如果指定的网络图片的宽度或高度大于这里的最大值，则会对图片进行压缩，指定成0的话就表示不管图片有多大，都不会进行压缩；第五个参数scaleType表示图片显示的缩放类型，被弃用的那个构造方法是没有这个参数的，它默认使用ScaleType.CENTER_INSIDE，第六个参数Config是常量，用于指定颜色的属性，其中ARGB_8888可以展示最好的颜色属性，每个图片像素占据4个字节的大小，而RGB_565则表示每个图片像素占据2个字节大小。第七个参数是加载图片失败后的回调。

小结：Request类的子类发送请求的步骤基本是一样的，可分为三步：

- 创建RequestQueue对象
- 创建Request类的子类对象(StringRequest/JsonRequest/ImageRequest)
- 将Request的子类对象添加到RequestQueue对象中


除了使用ImageRequest可以来加载图片之外，Volley中还可以使用`ImageLoader`来加载图片，ImageLoader内部其实也是用ImageRequest来实现的，不过ImageLoader明显要比ImageRequest更加高效，因为它不仅可以帮我们对图片进行缓存，还可以过滤掉重复的链接，避免重复发送请求，使用方法：

```
RequestQueue queue = Volley.newRequestQueue(this);
        
ImageLoader imageLoader = new ImageLoader(queue, new ImageLoader.ImageCache() {
	@Override public Bitmap getBitmap(String url) {
		return null;
	}

	@Override public void putBitmap(String url, Bitmap bitmap) {

	}
});

ImageLoader.ImageListener listener = ImageLoader.getImageListener(imageView, R.mipmap.ic_launcher, R.mipmap.ic_launcher_round);
imageLoader.get("http://tuku.chengdu.cn/CHN_CommendationPic/20120319105921966875741.jpg", listener);
```

因为ImageLoader不是继承自Request类，所以它加载图片的用法和之前那些不一样了，大致步骤为：

- 创建RequestQueue对象
- 创建ImageLoader对象
- 创建ImageListener对象
- 调用ImageLoader的get方法

在创建ImageLoader对象时，我们传了两个参数，RequestQueue和ImageCache，ImageCache是实现图片缓存的的接口，当图片加载成功后，会将图片缓存起来，下次再加载同样一张图片时，就不用去网络上加载而是直接在缓存中加载即可，这样既节省了资源也节省了请求时间。

我们看到ImageCache是一个接口：

```
public interface ImageCache {
	public Bitmap getBitmap(String url);
	public void putBitmap(String url, Bitmap bitmap);
}
```

我们要自己实现图片的缓存，只要实现这个接口就行了：

```
public class BitmapCache implements ImageLoader.ImageCache {

	private LruCache<String, Bitmap> mCache;

	public BitmapCache() {
		int maxSize = 2 * 1024 * 1024;
		mCache = new LruCache<String, Bitmap>(maxSize) {
			@Override
			protected int sizeOf(String key, Bitmap bitmap) {
				return bitmap.getRowBytes() * bitmap.getHeight();
			}
		};
	}

	@Override
	public Bitmap getBitmap(String url) {
		return mCache.get(url);
	}

	@Override
	public void putBitmap(String url, Bitmap bitmap) {
		mCache.put(url, bitmap);
	}

}
```

所以当我们创建ImageLoader对象时，new一个BitmapCache对象传进去即可：

```
ImageLoader imageLoader = new ImageLoader(queue, new BitmapCache());
```


除了以上两种加载图片的方式之外，Volley还提供了另外一种，使用NetworkImageView，不同于以上两种方式，NetworkImageView是一个自定义View，它是继承自ImageView，具备ImageView控件的所有功能，并且在原生的基础之上加入了加载网络图片的功能。NetworkImageView控件的用法要比前两种方式更加简单，大致可以分为以下几步：

- 创建一个RequestQueue对象。
- 创建一个ImageLoader对象。
- 在布局文件中添加一个NetworkImageView控件。
- 在代码中获取该控件的实例。
- 设置要加载的图片地址。


```
RequestQueue mQueue = Volley.newRequestQueue(this);
ImageLoader imageLoader = new ImageLoader(mQueue, new BitmapCache());
NetworkImageView networkImageView = (NetworkImageView) findViewById(R.id.networkImageView);
networkImageView.setDefaultImageResId(R.mipmap.ic_launcher);
networkImageView.setErrorImageResId(R.mipmap.ic_launcher);
networkImageView.setImageUrl("http://tuku.chengdu.cn/CHN_CommendationPic/20120319105921966875741.jpg", imageLoader);
```


使用ImageRequest和ImageLoader这两种方式来加载网络图片，都可以传入一个最大宽度和高度的参数来对图片进行压缩，但是由于NetworkImageView是一个控件，在加载图片的时候它会自动获取自身的宽高，然后对比网络图片的宽度，再决定是否需要对图片进行压缩。也就是说，压缩过程是在内部完全自动化的，并不需要我们关心，所以我们加载的时候不需要手动传入最大宽高，NetworkImageView会始终呈现给我们一张大小刚刚好的网络图片，不会多占用任何一点内存，如果你不想对图片进行压缩，只需要在布局文件中把NetworkImageView的layout_width和layout_height都设置成wrap_content就可以了，这样NetworkImageView就会将该图片的原始大小展示出来，不会进行任何压缩。





## 实现自定义Request


一般如果请求的数据是字符串、图片、JSON类型的数据时，我们都不需要自定义Request，但是如果这些还不满足我们的需求时，就要自定义Request了，需要做的事有下面两步：

- 继承Request<T>泛型类，其中的泛型表示我们请求期望解析响应的类型。
- 实现抽象方法`parseNetworkResponse()`和`deliverResponse()`。


Gson是使用反射将Java对象转换为JSON或从JSON转换的库。你可以定义与其对应的JSON键具有相同名称的Java实体对象，将Gson传递给类对象，Gson就可以将这个Java对象的各个字段自动填充值。以下是使用Gson进行解析的Volley请求的完整实现：


```
public class GsonRequest<T> extends Request<T> {

    private Gson mGson;
    private Class<T> clazz;
    private Map<String, String> headers;
    private Response.Listener<T> listener;
    
    public GsonRequest(int method, String url, Class<T> clazz, Map<String, String> headers, Response.Listener<T> listener, Response.ErrorListener errorListener) {
        super(method, url, errorListener);  
        mGson = new Gson();  
        mClass = clazz;  
        mListener = listener;  
    }
	
	public GsonRequest(String url, Class<T> clazz, Listener<T> listener,  
            ErrorListener errorListener) {  
        this(Method.GET, url, clazz, listener, errorListener);  
    }  

    @Override
    public Map<String, String> getHeaders() throws AuthFailureError {
        return headers != null ? headers : super.getHeaders();
    }

    @Override
    protected void deliverResponse(T response) {
        listener.onResponse(response);
    }

    @Override
    protected Response<T> parseNetworkResponse(NetworkResponse response) {
        try {
            String json = new String(response.data, HttpHeaderParser.parseCharset(response.headers));
            return Response.success(mGson.fromJson(json, clazz), HttpHeaderParser.parseCacheHeaders(response));
        } catch (UnsupportedEncodingException e) {
            return Response.error(new ParseError(e));
        } catch (JsonSyntaxException e) {
            return Response.error(new ParseError(e));
        }
    }
}
```

使用的时候我们县创建一个对象实体类，定义其属性和getter/setter方法，然后使用这个GsonRequest类解析时应该这样：

```
GsonRequest<JavaBean> gsonRequest = new GsonRequest<JavaBean>(
	url, JavaBean.class,
	new Response.Listener<JavaBean>() {
		@Override
		public void onResponse(JavaBean javaBean) {
			
		}
	}, new Response.ErrorListener() {
	@Override
	public void onErrorResponse(VolleyError error) {
		Log.e("TAG", error.getMessage(), error);
	}
});
mQueue.add(gsonRequest);
```



## Volley源码解析



先来看下面这张图，是Volley官方文档主页上抠下来的，它解释了Volley的工作流程和整体结构

![volley-request](http://offfjcibp.bkt.clouddn.com/volley-request.png)


一般我们要分析一个框架的源码，首先我们要找到它的入口，从Volley的使用方法来看，我们第一步是先通过Volley类的`newRequestQueue(this)`创建一个RequestQueue对象，那就从这里开始吧。Volley类只有两个方法，都是`newRequestQueue`，一个是一个参数，另一个是两个参数，我们直接看最终调用的那个：

```
public static RequestQueue newRequestQueue(Context context, HttpStack stack) {
	File cacheDir = new File(context.getCacheDir(), DEFAULT_CACHE_DIR);

	String userAgent = "volley/0";
	try {
		String packageName = context.getPackageName();
		PackageInfo info = context.getPackageManager().getPackageInfo(packageName, 0);
		userAgent = packageName + "/" + info.versionCode;
	} catch (NameNotFoundException e) {
	}

	if (stack == null) {
		if (Build.VERSION.SDK_INT >= 9) {
			stack = new HurlStack();
		} else {
			// Prior to Gingerbread, HttpUrlConnection was unreliable.
            // See: http://android-developers.blogspot.com/2011/09/androids-http-clients.html
			stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));
		}
	}

	Network network = new BasicNetwork(stack);

	RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);
	queue.start();

	return queue;
}
```


我们传进来的HttpStack是null，所以这里会先判断sdk版本，如果是9以上就创建`HurlStack`类对象，否则创建`HttpClientStack`类型对象，至于HurlStack和HttpClientStack这两个类是什么？我们点进去可以看到这个类的说明：

```
/**
 * An {@link HttpStack} based on {@link HttpURLConnection}.
 */
public class HurlStack implements HttpStack {
```


```
/**
 * An HttpStack that performs request over an {@link HttpClient}.
 */
public class HttpClientStack implements HttpStack {
```

也就是说HurlStack内部是基于HttpURLConnection实现的，而HttpClientStack是基于HttpClient来实现的，这里为什么要根据sdk版本来分别选择这两种不同的类呢？

我们根据上面源码中提示的文章地址[http://android-developers.blogspot.com/2011/09/androids-http-clients.html](http://android-developers.blogspot.com/2011/09/androids-http-clients.html)可以知道，大多数的Android应用程序都会使用HTTP协议来发送和接收网络数据，而Android中主要提供了两种方式来进行HTTP操作，HttpURLConnection和HttpClient。这两种方式都支持HTTPS协议、以流的形式进行上传和下载、配置超时时间、IPv6、以及连接池等功能。但是在Android2.3之前，HttpURLConnection是不可靠，bug太多，而HttpClient比较稳定，bug较少，API很多，基本上已经满足了开发者们的需求，因此很难在不破坏其兼容性的情况下进行拓展。

在Android2.2版本之前，HttpURLConnection一直存在着一些令人厌烦的bug。比如说对一个可读的InputStream调用close()方法时，就有可能会导致连接池失效了。那么我们通常的解决办法就是直接禁用掉连接池的功能。

在Android2.3版本的时候，加入了更加透明化的响应压缩。HttpURLConnection会自动在每个发出的请求中加入如下消息头：Accept-Encoding: gzip，并处理相应的返回结果。但是如果启动了响应压缩的功能，HTTP响应头里的Content-Length就会代表着压缩后的长度，这时再使用getContentLength()方法来取出解压后的数据就是错误的了。正确的做法应该是一直调用InputStream.read()方法来读取响应数据，一直到出现-1为止。

在Android 2.3版本中还增加了一些HTTPS方面的改进，现在HttpsURLConnection会使用SNI(Server Name Indication)的方式进行连接，使得多个HTTPS主机可以共享同一个IP地址。除此之外，还增加了一些压缩和会话的机制。如果连接失败，它会自动去尝试重新进行连接。这使得HttpsURLConnection可以在不破坏老版本兼容性的前提下，更加高效地连接最新的服务器。

在Android4.0版本中，我们又添加了一些响应的缓存机制。当缓存被安装后(调用HttpResponseCache的install()方法)，所有的HTTP请求都会满足以下三种情况：

- 所有的缓存响应都由本地存储来提供。因为没有必要去发起任务的网络连接请求，所有的响应都可以立刻获取到。

- 有条件的缓存响应必须要有服务器来进行更新检查。比如说客户端发起了一条类似于“如果a.png这张图片发生了改变，就将它发送给我” 这样的请求，服务器需要将更新后的数据进行返回，或者返回一个304(Not Modified)状态。如果请求的内容没有发生，客户端就不会下载任何数据。

- 没有缓存的响应都是由服务器直接提供的。这部分响应会在稍后存储到响应缓存中。


在Android2.2版本之前，HttpClient拥有较少的bug，因此使用它是最好的选择。而在Android2.3版本及以后，HttpURLConnection则是最佳的选择。它的API简单，体积较小，因而非常适用于Android项目。压缩和缓存机制可以有效地减少网络访问的流量，在提升速度和省电方面也起到了较大的作用。对于新的应用程序应该更加偏向于使用HttpURLConnection，因为在以后的工作当中我们也会将更多的时间放在优化HttpURLConnection上面。



以上解释了在sdk为9以上使用HttpURLConnection的原因。下面还是回到Volley的源码解析中来，使用HttpStack对象创建好Network之后，再根据Network和Cache来构建RequestQueue对象，调用其start方法启动，然后将RequestQueue对象返回，newRequestQueue方法结束。

接着来看看这个start方法方法内部都做了什么。

```
public void start() {
	stop();  // Make sure any currently running dispatchers are stopped.
	// Create the cache dispatcher and start it.
	mCacheDispatcher = new CacheDispatcher(mCacheQueue, mNetworkQueue, mCache, mDelivery);
	mCacheDispatcher.start();

	// Create network dispatchers (and corresponding threads) up to the pool size.
	for (int i = 0; i < mDispatchers.length; i++) {
		NetworkDispatcher networkDispatcher = new NetworkDispatcher(mNetworkQueue, mNetwork,
				mCache, mDelivery);
		mDispatchers[i] = networkDispatcher;
		networkDispatcher.start();
	}
}
```

首先要知道这个CacheDispatcher和这个NetworkDispatcher都是继承自Thread，一开始停止所有线程，然后创建一个新的CacheDispatcher线程，启动，然后创建默认4个NetworkDispatcher，启动，当start方法执行完后，Volley已经启动了5个线程等待网络请求任务来处理了。CacheDispatcher是缓存线程，NetworkDispatcher是网络请求线程。

在拿到RequestQueue对象后，我们会把我们的请求add到RequestQueue对象中去，其中add的方法如下：

```
public <T> Request<T> add(Request<T> request) {
	// Tag the request as belonging to this queue and add it to the set of current requests.
	request.setRequestQueue(this);
	synchronized (mCurrentRequests) {
		mCurrentRequests.add(request);
	}

	// Process requests in the order they are added.
	request.setSequence(getSequenceNumber());
	request.addMarker("add-to-queue");

	// If the request is uncacheable, skip the cache queue and go straight to the network.
	if (!request.shouldCache()) {
		mNetworkQueue.add(request);
		return request;
	}

	// Insert request into stage if there's already a request with the same cache key in flight.
	synchronized (mWaitingRequests) {
		String cacheKey = request.getCacheKey();
		if (mWaitingRequests.containsKey(cacheKey)) {
			// There is already a request in flight. Queue up.
			Queue<Request<?>> stagedRequests = mWaitingRequests.get(cacheKey);
			if (stagedRequests == null) {
				stagedRequests = new LinkedList<Request<?>>();
			}
			stagedRequests.add(request);
			mWaitingRequests.put(cacheKey, stagedRequests);
			if (VolleyLog.DEBUG) {
				VolleyLog.v("Request for cacheKey=%s is in flight, putting on hold.", cacheKey);
			}
		} else {
			// Insert 'null' queue for this cacheKey, indicating there is now a request in
			// flight.
			mWaitingRequests.put(cacheKey, null);
			mCacheQueue.add(request);
		}
		return request;
	}
}
```

先是给这个请求设置了一些属性，然后根据该请求是否可以缓存，将其加入到不同的请求队列中，在默认情况下，每条请求都是可以缓存的，当然我们也可以调用Request的setShouldCache(false)方法来改变这一默认行为。


这里补充一点，请求默认是缓存的，然后通过拿到请求的cacheKey，以键值对形式将请求缓存到缓存队列中去，那么这里请求request的cacheKey是怎么来的？

看Request中这两个方法：

```
public String getUrl() {
	return mUrl;
}

public String getCacheKey() {
	return getUrl();
}
```

其实cacheKey就是请求的url，因为这个请求肯定是唯一的。


既然默认每条请求都是可以缓存的，自然就被添加到了缓存队列中，于是一直在后台等待的缓存线程就开始处理请求了，我们再看看缓存线程CacheDispatcher的run方法：

```
@Override
public void run() {
	if (DEBUG) VolleyLog.v("start new dispatcher");
	Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);

	// Make a blocking call to initialize the cache.
	mCache.initialize();

	while (true) {
		try {
			// Get a request from the cache triage queue, blocking until
			// at least one is available.
			final Request<?> request = mCacheQueue.take();
			request.addMarker("cache-queue-take");

			// If the request has been canceled, don't bother dispatching it.
			if (request.isCanceled()) {
				request.finish("cache-discard-canceled");
				continue;
			}

			// Attempt to retrieve this item from cache.
			Cache.Entry entry = mCache.get(request.getCacheKey());
			if (entry == null) {
				request.addMarker("cache-miss");
				// Cache miss; send off to the network dispatcher.
				mNetworkQueue.put(request);
				continue;
			}

			// If it is completely expired, just send it to the network.
			if (entry.isExpired()) {
				request.addMarker("cache-hit-expired");
				request.setCacheEntry(entry);
				mNetworkQueue.put(request);
				continue;
			}

			// We have a cache hit; parse its data for delivery back to the request.
			request.addMarker("cache-hit");
			Response<?> response = request.parseNetworkResponse(
					new NetworkResponse(entry.data, entry.responseHeaders));
			request.addMarker("cache-hit-parsed");

			if (!entry.refreshNeeded()) {
				// Completely unexpired cache hit. Just deliver the response.
				mDelivery.postResponse(request, response);
			} else {
				// Soft-expired cache hit. We can deliver the cached response,
				// but we need to also send the request to the network for
				// refreshing.
				request.addMarker("cache-hit-refresh-needed");
				request.setCacheEntry(entry);

				// Mark the response as intermediate.
				response.intermediate = true;

				// Post the intermediate response back to the user and have
				// the delivery then forward the request along to the network.
				mDelivery.postResponse(request, response, new Runnable() {
					@Override
					public void run() {
						try {
							mNetworkQueue.put(request);
						} catch (InterruptedException e) {
							// Not much we can do about this.
						}
					}
				});
			}

		} catch (InterruptedException e) {
			// We may have been interrupted because it was time to quit.
			if (mQuit) {
				return;
			}
			continue;
		}
	}
}
```

可以看到有一个while(true)循环，说明缓存线程始终是在运行的，接着会尝试从缓存当中取出响应结果，如果为空的话，则把这条请求加入到网络请求队列中，如果不为空的话，再判断该缓存是否已过期，如果已经过期了则同样把这条请求加入到网络请求队列中，否则就认为不需要重发网络请求，直接使用缓存中的数据即可。之后会调用Request的parseNetworkResponse()方法来对数据进行解析，再往后就是将解析出来的数据进行回调了，这部分代码我们先跳过，因为它的逻辑和NetworkDispatcher后半部分的逻辑是基本相同的，等会再看，先来看一下NetworkDispatcher中是怎么处理网络请求队列的，同样是NetworkDispatcher的run方法：

```
@Override
public void run() {
	Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
	while (true) {
		long startTimeMs = SystemClock.elapsedRealtime();
		Request<?> request;
		try {
			// Take a request from the queue.
			request = mQueue.take();
		} catch (InterruptedException e) {
			// We may have been interrupted because it was time to quit.
			if (mQuit) {
				return;
			}
			continue;
		}

		try {
			request.addMarker("network-queue-take");

			// If the request was cancelled already, do not perform the
			// network request.
			if (request.isCanceled()) {
				request.finish("network-discard-cancelled");
				continue;
			}

			addTrafficStatsTag(request);

			// Perform the network request.
			NetworkResponse networkResponse = mNetwork.performRequest(request);
			request.addMarker("network-http-complete");

			// If the server returned 304 AND we delivered a response already,
			// we're done -- don't deliver a second identical response.
			if (networkResponse.notModified && request.hasHadResponseDelivered()) {
				request.finish("not-modified");
				continue;
			}

			// Parse the response here on the worker thread.
			Response<?> response = request.parseNetworkResponse(networkResponse);
			request.addMarker("network-parse-complete");

			// Write to cache if applicable.
			// TODO: Only update cache metadata instead of entire record for 304s.
			if (request.shouldCache() && response.cacheEntry != null) {
				mCache.put(request.getCacheKey(), response.cacheEntry);
				request.addMarker("network-cache-written");
			}

			// Post the response back.
			request.markDelivered();
			mDelivery.postResponse(request, response);
		} catch (VolleyError volleyError) {
			volleyError.setNetworkTimeMs(SystemClock.elapsedRealtime() - startTimeMs);
			parseAndDeliverNetworkError(request, volleyError);
		} catch (Exception e) {
			VolleyLog.e(e, "Unhandled exception %s", e.toString());
			VolleyError volleyError = new VolleyError(e);
			volleyError.setNetworkTimeMs(SystemClock.elapsedRealtime() - startTimeMs);
			mDelivery.postError(request, volleyError);
		}
	}
}
```

真正执行网络请求是调用Network的performRequest方法，而Network是一个接口，它的实现类是BasicNetwork，performRequest方法的代码如下：

```
@Override
public NetworkResponse performRequest(Request<?> request) throws VolleyError {
	long requestStart = SystemClock.elapsedRealtime();
	while (true) {
		HttpResponse httpResponse = null;
		byte[] responseContents = null;
		Map<String, String> responseHeaders = Collections.emptyMap();
		try {
			// Gather headers.
			Map<String, String> headers = new HashMap<String, String>();
			addCacheHeaders(headers, request.getCacheEntry());
			httpResponse = mHttpStack.performRequest(request, headers);
			StatusLine statusLine = httpResponse.getStatusLine();
			int statusCode = statusLine.getStatusCode();
			.
			.
			.
		} catch (SocketTimeoutException e) {
			attemptRetryOnException("socket", request, new TimeoutError());
		} 
		.
		.
		.
	}
}
```

我们看到实际上是调用了HttpStack的performRequest方法，而HttpStack前面已经说过，就是HttpURLConnecttion或者HttpClient，请求到数据后封装成NetworkResponse对象返回。在NetworkDispatcher的run方法中发送请求后返回NetworkResponse对象，接着调用了Request的parseNetworkResponse方法来解析响应数据和把响应数据存入缓存中，不同的Request的子类其parseNetworkResponse方法都不一样，当我们自定义Request时也必需要重写parseNetworkResponse方法的。

解析完之后，接着会调用mDelivery.postResponse方法来回调解析出来的响应数据，这个mDelivery是一个接口ResponseDelivery，结果分发器，它的实现类是ExecutorDelivery，负责分发响应和错误数据，其中的postResponse方法如下：

```
@Override
public void postResponse(Request<?> request, Response<?> response) {
	postResponse(request, response, null);
}

@Override
public void postResponse(Request<?> request, Response<?> response, Runnable runnable) {
	request.markDelivered();
	request.addMarker("post-response");
	mResponsePoster.execute(new ResponseDeliveryRunnable(request, response, runnable));
}
```

可以看到在mResponsePoster的execute()方法中传入了一个ResponseDeliveryRunnable对象，它的作用是保证该对象中的run()方法就是在主线程当中运行，也就是把解析后的数据从子线程中切换到主线程了。

```
private class ResponseDeliveryRunnable implements Runnable {
	private final Request mRequest;
	private final Response mResponse;
	private final Runnable mRunnable;

	public ResponseDeliveryRunnable(Request request, Response response, Runnable runnable) {
		mRequest = request;
		mResponse = response;
		mRunnable = runnable;
	}

	@SuppressWarnings("unchecked")
	@Override
	public void run() {
		// If this request has canceled, finish it and don't deliver.
		if (mRequest.isCanceled()) {
			mRequest.finish("canceled-at-delivery");
			return;
		}

		// Deliver a normal response or error, depending.
		if (mResponse.isSuccess()) {
			mRequest.deliverResponse(mResponse.result);
		} else {
			mRequest.deliverError(mResponse.error);
		}

		// If this is an intermediate response, add a marker, otherwise we're done
		// and the request can be finished.
		if (mResponse.intermediate) {
			mRequest.addMarker("intermediate-response");
		} else {
			mRequest.finish("done");
		}

		// If we have been provided a post-delivery runnable, run it.
		if (mRunnable != null) {
			mRunnable.run();
		}
   }
}
```

可以看到请求成功后调用了Request的deliverResponse()方法，这个方法就是我们在自定义Request时需要重写的另外一个方法，每一条网络请求的响应都是回调到这个方法中，最后我们再在这个方法中将响应的数据回调到Response.Listener的onResponse()方法中就可以了。

至于这个从主线程切换到子线程再到主线程的过程到底是怎样的呢？我再来梳理一遍：

当我们创建RequestQueue时，在start方法中，会创建五个子线程，其中一个CacheDispatcher，默认四个NetworkDispatcher，他们的构造方法中都有传递一个对象，就是这个mDelivery，它是一个ResponseDelivery接口，实现类是ExecutorDelivery，我们看看RequestQueue的这两个构造方法就清除了：

```
public RequestQueue(Cache cache, Network network, int threadPoolSize, ResponseDelivery delivery) {
	mCache = cache;
	mNetwork = network;
	mDispatchers = new NetworkDispatcher[threadPoolSize];
	mDelivery = delivery;
}

public RequestQueue(Cache cache, Network network, int threadPoolSize) {
	this(cache, network, threadPoolSize, new ExecutorDelivery(new Handler(Looper.getMainLooper())));
}
```

所以RequestQueue是自己构建了这个ExecutorDelivery对象，然后再传递到后面的子线程后的，ExecutorDelivery的构造方法参数就是一个主线程的handler对象，所以这个mDelivery内部是持有一个主线程的消息系统的。然后在请求完成，解析完响应数据后，调用了mDelivery的postResponse方法，在postResponse方法最后调用了一个Executor对象的execute方法方法，参数就是一个Runnable，并且将解析后的响应数据传给Runnable。我们再看看ExecutorDelivery的构造方法：

```
public ExecutorDelivery(final Handler handler) {
	// Make an Executor that just wraps the handler.
	mResponsePoster = new Executor() {
		@Override
		public void execute(Runnable command) {
			handler.post(command);
		}
	};
}
```

由于这个消息系统是在主线程构造，也就是说Runnable会被发送到主线程去执行，这样就把解析后的数据传递到主线程了。现在再回过头看源码解析开头的那张流程图就比较清晰了，我们在主线程中调用RequestQueue的add()方法来添加一条网络请求，这条请求会先被加入到缓存队列当中，如果发现可以找到相应的缓存结果就直接读取缓存并解析，然后回调给主线程。如果在缓存中没有找到结果，则将这条请求加入到网络请求队列中，然后处理发送HTTP请求，解析响应结果，写入缓存，并回调主线程。



Volley的源码解析就结束了，其实源码解析没有必要每个类每个方法都去看一遍，我们只需要将这个框架的工作流程相关的源码即可，主要是掌握框架的优秀的设计思路以及一些良好的编码实现。好了，就到这里。






## 参考资料

[手撕 Volley（一）](http://www.jianshu.com/p/33be82da8f25)
[手撕 Volley（二）](http://www.jianshu.com/p/358b766c8d27)
[手撕 Volley（三）](http://www.jianshu.com/p/63c0cd0fd99c)
[Android Volley完全解析(一)，初识Volley的基本用法](http://blog.csdn.net/guolin_blog/article/details/17482095)
[Android Volley完全解析(二)，使用Volley加载网络图片](http://blog.csdn.net/guolin_blog/article/details/17482165)
[Android Volley完全解析(三)，定制自己的Request](http://blog.csdn.net/guolin_blog/article/details/17612763)
[Android Volley完全解析(四)，带你从源码的角度理解Volley](http://blog.csdn.net/guolin_blog/article/details/17656437)
[HTTP协议详解](http://www.cnblogs.com/li0803/archive/2008/11/03/1324746.html)
[HTTP协议详解](http://www.cnblogs.com/EricaMIN1987_IT/p/3837436.html)
[HttpClient和HttpURLConnection的区别](http://blog.csdn.net/hguang_zjh/article/details/33743249)