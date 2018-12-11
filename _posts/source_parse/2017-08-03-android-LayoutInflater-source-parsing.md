---
layout: post
title: LayoutInflater源码解析
category: blog
tags: LayoutInflater
---



LayoutInflater我们经常会用到，在列表适配器中或者在加载自定义布局时，它的作用就是将一个xml文件渲染成一个View或者ViewGroup，虽然知道它的作用的用法，但是弄清楚它的工作原理也是很有必要的，今天就通过解析它的源码来分析下它的工作原理。


## 基本使用方法

首先还是来说说它的几种用法：

```
LayoutInflater layoutInflater = getLayoutInflater();//如果是在Activity中，可以直接使用getLayoutInflater()方法
```

其他地方可以使用下面这两种方式：

```
LayoutInflater layoutInflater = LayoutInflater.from(context);
```

```
LayoutInflater layoutInflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
```

其实from(context)这种方法实际上也就是第三种方式了。可以看LayoutInflater类内部的from方法：

```
public static LayoutInflater from(Context context) {
	LayoutInflater LayoutInflater =
			(LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
	if (LayoutInflater == null) {
		throw new AssertionError("LayoutInflater not found.");
	}
	return LayoutInflater;
}
```

我们再看看getLayoutInflater()方法的实现：

```
public LayoutInflater getLayoutInflater() {
	return getWindow().getLayoutInflater();
}
```

说明这里了是调用了Window类的getLayoutInflater()方法，而Window类只是一个抽象类，它的实现类其实是PhoneWindow，那就看看PhoneWindow类中getLayoutInflater()方法的实现：

```
public LayoutInflater getLayoutInflater() {
	return mLayoutInflater;
}
```

而在PhoneWindow的构造方法中可以看到：

```
public PhoneWindow(Context context) {
	super(context);
	mLayoutInflater = LayoutInflater.from(context);
}
```

所以说这三种方式从根本上说都是一样的，通过Content的getSystemService方法，传递Context.LAYOUT_INFLATER_SERVICE参数，就能获取到LayoutInflater对象了。实际上在getSystemService方法中获取到的LayoutInflater对象其实是PhoneLayoutInflater这个子类对象，因为Context的包装类ContextThemeWrapper中定义了getSystemService方法：

```
public Object getSystemService(String name) {
	if (LAYOUT_INFLATER_SERVICE.equals(name)) {
		if (mInflater == null) {
			mInflater = LayoutInflater.from(getBaseContext()).cloneInContext(this);
		}
		return mInflater;
	}
	return getBaseContext().getSystemService(name);
}
```

而LayoutInflater中的cloneInContext方法只是一个抽象方法，在PhoneLayoutInflater有关于这个方法的实现：

```
public LayoutInflater cloneInContext(Context newContext) {
	return new PhoneLayoutInflater(this, newContext);
}
```

以上是获取LayoutInflater对象，接着是用LayoutInflater对象加载xml布局：

```
View view = layoutInflater.inflate(R.layout.activity_main, null);
```

另外inflate还有三个重载方法：

```
inflate(int resource,  ViewGroup root, boolean attachToRoot);

inflate(XmlPullParser parser, ViewGroup root);

inflate(XmlPullParser parser, ViewGroup root, boolean attachToRoot);
```

仔细看这几个方法的关系，不难发现，使用xml资源id加载的方法最终会调用这个方法：

```
public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) {
	final Resources res = getContext().getResources();
	if (DEBUG) {
		Log.d(TAG, "INFLATING from resource: \"" + res.getResourceName(resource) + "\" ("
				+ Integer.toHexString(resource) + ")");
	}

	final XmlResourceParser parser = res.getLayout(resource);
	try {
		return inflate(parser, root, attachToRoot);
	} finally {
		parser.close();
	}
}
```

先是获取到系统的Resource对象，然后通过getLayout方法传递资源id获取到一个XML解析器对象，最终会调用下面这个方法

```
inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot)；
```

Android中的布局都是xml格式的，所以涉及到xml的解析，而解析xml有DOM、SAX和PULL这几种方式，其中DOM不适合xml文档较大，内存较小的场景，SAX和PULL都具有解析速度快，占用内存小的优点，但PULL操作更简单，所以Android选择了PULL方式来解析布局xml文件。

> DOM，通用性强，它会将XML文件的所有内容读取到内存中，然后允许您使用DOM API遍历XML树、检索所需的数据；简单直观，但需要将文档读取到内存，并不太适合移动设备；SAX，SAX是一个解析速度快并且占用内存少的xml解析器；采用事件驱动，它并不需要解析整个文档；实现：继承DefaultHandler，覆写startElement、endElement、characters等方法；PULL，Android自带的XML解析器，和SAX基本类似，也是事件驱动，不同的是PULL事件返回的是数值型；推荐使用。

## 源码解析

inflate方法的最终实现：

```
public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
	synchronized (mConstructorArgs) {
		Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");

		final Context inflaterContext = mContext;
		final AttributeSet attrs = Xml.asAttributeSet(parser);
		Context lastContext = (Context) mConstructorArgs[0];
		mConstructorArgs[0] = inflaterContext;
		
		// 这里默认就把root赋值给要返回的view了
		View result = root;

		try {
			// Look for the root node.
			
			//  首先会找到布局的根节点，如RelativeLayout、LinearLayout等
			int type;
			while ((type = parser.next()) != XmlPullParser.START_TAG &&
					type != XmlPullParser.END_DOCUMENT) {
				// Empty
			}

			if (type != XmlPullParser.START_TAG) {
				throw new InflateException(parser.getPositionDescription()
						+ ": No start tag found!");
			}

			// 拿到根节点，注意这里的节点只可能是开始根节点或者结束根节点
			final String name = parser.getName();
			
			if (TAG_MERGE.equals(name)) {
				// 如果根节点是merge，那么它必须依附于root上，如果root为空或者依附属性为false，就会抛出异常。
				if (root == null || !attachToRoot) {
					throw new InflateException("<merge /> can be used only with a valid "
							+ "ViewGroup root and attachToRoot=true");
				}
				// 解析根节点下面的子节点，并创建对应的子View添加到根View中来
				rInflate(parser, root, inflaterContext, attrs, false);
			} else {
				// Temp is the root view that was found in the xml
				// 根节点不是merge，就根据root以及当前节点信息来生成一个根View
				final View temp = createViewFromTag(root, name, inflaterContext, attrs);

				ViewGroup.LayoutParams params = null;

				if (root != null) {
					
					// 如果指定了root，那么根据root的布局属性生成一个LayoutParams
					params = root.generateLayoutParams(attrs);
					if (!attachToRoot) {
						// 如果没有指定加载的资源xml依附于root，那么把上面生成的LayoutParams设置给根View
						temp.setLayoutParams(params);
					}
				}

				// 开始解析加载子节点，并创建对应的子View添加到根View中来
				rInflateChildren(parser, temp, attrs, true);

				// 如果指定了root并且指定加载的xml依附于root，那么当整个资源xml解析完后就将根View添加到root中去
				if (root != null && attachToRoot) {
					// 而且整个方法到这里就得到了要返回的结果了，因为一开始就把root赋值给result，最后返回的result仍然是添加了temp的root
					root.addView(temp, params);
				}

				// 如果root为空且attachToRoot为false时，返回的就是根View
				if (root == null || !attachToRoot) {
					result = temp;
				}
			}

		} catch (XmlPullParserException e) {
			final InflateException ie = new InflateException(e.getMessage(), e);
			ie.setStackTrace(EMPTY_STACK_TRACE);
			throw ie;
		} catch (Exception e) {
			final InflateException ie = new InflateException(parser.getPositionDescription()
					+ ": " + e.getMessage(), e);
			ie.setStackTrace(EMPTY_STACK_TRACE);
			throw ie;
		} finally {
			// Don't retain static reference on context.
			mConstructorArgs[0] = lastContext;
			mConstructorArgs[1] = null;

			Trace.traceEnd(Trace.TRACE_TAG_VIEW);
		}
		// 最后返回结果View
		return result;
	}
}
```

根据上面inflate()方法的源码，总结一下几点：

- 当root为null时，attachToRoot的值已无关紧要，它会直接返回资源xml对应加载好的View。
- 当root不为null、attachToRoot为false时，仍然会返回资源xml对应加载好的View，但是root的LayoutParams会被设置给返回的View。
- 当root不为null、attachToRoot为true时，则会把资源xml对应的View添加到root中然后返回root。


接下来我们再看看是如何将xml生成对应的View的。

无论根节点是不是merge，最后都会调用rInflate()方法去解析加载子View，rInflateChildren()方法其实也是调用了rInflate()，所以看下rInflate()方法的源码：

```
void rInflate(XmlPullParser parser, View parent, Context context,
		AttributeSet attrs, boolean finishInflate) throws XmlPullParserException, IOException {

	final int depth = parser.getDepth();
	int type;

	while (((type = parser.next()) != XmlPullParser.END_TAG ||
			parser.getDepth() > depth) && type != XmlPullParser.END_DOCUMENT) {

		if (type != XmlPullParser.START_TAG) {
			continue;
		}

		final String name = parser.getName();
		
		if (TAG_REQUEST_FOCUS.equals(name)) {
			parseRequestFocus(parser, parent);
		} else if (TAG_TAG.equals(name)) {
			parseViewTag(parser, parent, attrs);
		} else if (TAG_INCLUDE.equals(name)) {
			if (parser.getDepth() == 0) {
				throw new InflateException("<include /> cannot be the root element");
			}
			parseInclude(parser, context, parent, attrs);
		} else if (TAG_MERGE.equals(name)) {
			throw new InflateException("<merge /> must be the root element");
		} else {
			final View view = createViewFromTag(parent, name, context, attrs);
			final ViewGroup viewGroup = (ViewGroup) parent;
			final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);
			rInflateChildren(parser, view, attrs, true);
			viewGroup.addView(view, params);
		}
	}

	if (finishInflate) {
		parent.onFinishInflate();
	}
}
```

前面几个条件判断是解析几个特殊的节点`requestFocus`、`tag`、`include`和`merge`，真正生成View并添加到父View中的是在最后一个else中。首先根据当前节点信息生成一个View，再递归这个View下所有子节点，最后将这个View添加到父View中，如果子节点都解析完了，就调用父View的onFinishInflate方法来结束解析。

通过上面的源码可以看出，无论是temp还是每一个子View，都是通过createViewFromTag()方法生成的，下面我们就来一探究竟。

```
View createViewFromTag(View parent, String name, Context context, AttributeSet attrs, boolean ignoreThemeAttr) {

	...

	try {
		View view;
		if (mFactory2 != null) {
			view = mFactory2.onCreateView(parent, name, context, attrs);
		} else if (mFactory != null) {
			view = mFactory.onCreateView(name, context, attrs);
		} else {
			view = null;
		}

		if (view == null && mPrivateFactory != null) {
			view = mPrivateFactory.onCreateView(parent, name, context, attrs);
		}

		if (view == null) {
			final Object lastContext = mConstructorArgs[0];
			mConstructorArgs[0] = context;
			try {
				if (-1 == name.indexOf('.')) {
					view = onCreateView(parent, name, attrs);
				} else {
					view = createView(name, null, attrs);
				}
			} finally {
				mConstructorArgs[0] = lastContext;
			}
		}

		return view;
	} catch (InflateException e) {
		throw e;

	} catch (ClassNotFoundException e) {
		final InflateException ie = new InflateException(attrs.getPositionDescription() + ": Error inflating class " + name, e);
		ie.setStackTrace(EMPTY_STACK_TRACE);
		throw ie;
	} catch (Exception e) {
		final InflateException ie = new InflateException(attrs.getPositionDescription() + ": Error inflating class " + name, e);
		ie.setStackTrace(EMPTY_STACK_TRACE);
		throw ie;
	}
}
```

如果mFactory2、mFactory、mPrivateFactory这几个自定义工厂不为null时就调用他们的onCreateView方法来生成一个View，但从LayoutInflater的构造方法来看，这三个变量初始时都是空的，所以最终会调用LayoutInflater本身的onCreateView方法或者createView方法，其中`if (-1 == name.indexOf('.'))`这个判断是xml中当前节点名中是否含有`.`，等于-1表示没有，即代表系统的widget下的View，默认前缀是`android.view.`，最终他们都会调用createView(name, null, attrs)这个方法。

```
public final View createView(String name, String prefix, AttributeSet attrs) throws ClassNotFoundException, InflateException {
	Constructor<? extends View> constructor = sConstructorMap.get(name);
	// 先从缓存中找是否存在该节点对应View的构造方法
	if (constructor != null && !verifyClassLoader(constructor)) {
		constructor = null;
		sConstructorMap.remove(name);
	}
	Class<? extends View> clazz = null;

	try {
		if (constructor == null) {
			// 没有就先根据类加载器通过全类名得到该类的反射类，再获取其构造方法
			clazz = mContext.getClassLoader().loadClass(
					prefix != null ? (prefix + name) : name).asSubclass(View.class);
			
			if (mFilter != null && clazz != null) {
				boolean allowed = mFilter.onLoadClass(clazz);
				if (!allowed) {
					failNotAllowed(name, prefix, attrs);
				}
			}
			constructor = clazz.getConstructor(mConstructorSignature);
			constructor.setAccessible(true);
			sConstructorMap.put(name, constructor);
		} else {
			if (mFilter != null) {
				Boolean allowedState = mFilterMap.get(name);
				if (allowedState == null) {
					clazz = mContext.getClassLoader().loadClass(
							prefix != null ? (prefix + name) : name).asSubclass(View.class);
					
					boolean allowed = clazz != null && mFilter.onLoadClass(clazz);
					mFilterMap.put(name, allowed);
					if (!allowed) {
						failNotAllowed(name, prefix, attrs);
					}
				} else if (allowedState.equals(Boolean.FALSE)) {
					failNotAllowed(name, prefix, attrs);
				}
			}
		}

		Object[] args = mConstructorArgs;
		args[1] = attrs;
		
		// 通过构造方法来生成对应的View
		final View view = constructor.newInstance(args);
		
		// 如果是ViewStub的情况
		if (view instanceof ViewStub) {
			final ViewStub viewStub = (ViewStub) view;
			viewStub.setLayoutInflater(cloneInContext((Context) args[0]));
		}
		return view;

	} 
	
	...
	
}
```


**LayoutInflater加载xml的原理**

归根到底就是通过XML解析器从根节点开始，递归解析xml的每个节点，通过当前节点名称（全类名），使用ClassLoader获取到对应类的构造方法，然后创建对应类的实例（View），最后将这个View添加到它的上层节点（父View）。并同时会解析对应xml节点的属性作为View的属性。每个层级的节点都会被生成一个个的View，并根据View的层级关系添加到对应的上层节点中去（直接父View），最终返回一个包含了所有解析好的子View的布局根View。

