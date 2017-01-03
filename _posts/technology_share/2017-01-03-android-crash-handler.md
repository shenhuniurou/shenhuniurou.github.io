---
layout: post
title: Android��������쳣
category: ��������
tags: crash�쳣
---



��������д�Ĵ�����������һЩbug���Լ����ڲ��Ի����������������쵼�³��ֵ�һЩ�쳣���ڲ��Թ�����û�з��֣�����app����֮���żȻ���ֵ�һЩbug��������app��ʹ�ù����г���ANR�����Ǹ����˵��۵�����app���������ֺ����ȣ���֮��app�����쳣ʱ���û����鲻�Ѻã����ǿ�������Ҫȥ������Щ�쳣���ռ���Щ�쳣��Ϣ�������ϴ��������������ڿ�����Աȥ�����Щ���⣬ͬʱ�����ǻ���Ҫ���û�һ���ѺõĽ������顣

��Ҳ��������ಶ������쳣����������������඼ǧƪһ�ɣ�ֻ˵����ô�����쳣�������쳣����û�жԲ����쳣��Ľ��潻�������ܺõĴ���ϵͳ����ANRʱ���ȿ�һ��ʱ�䣨���ʱ�仹�е㳤��Ȼ�󵯳�һ��ϵͳĬ�ϵĶԻ��򣬵�����������Ҫ���ǵ������쳣���ʱ���������Ի��򣬲��ҶԻ��������Զ�����档

����ʵ�ֵ�Ч����ͼ��

<img src="http://upload-images.jianshu.io/upload_images/1159224-cdf0b4169183ecaf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="40%"  alt="����Ч��ͼ" align="center" />

������Ҫ�Զ���`Application`��������`AndroidManifest.xml`�н�������
![AndroidManifest.xml.png](http://upload-images.jianshu.io/upload_images/1159224-ae3de39787e5a75d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

����Ҫ�Զ���һ��Ӧ���쳣������`AppUncaughtExceptionHandler`���������ʵ��`Thread.UncaughtExceptionHandler`�ӿڣ����⻹��Ҫ��д`uncaughtException`������ȥ�������Լ��ķ�ʽ�������쳣

��`Application`������ֻ��Ҫ��ʼ���Զ�����쳣�����༴�ɣ�

```java
@Override public void onCreate() {
	super.onCreate();
	mInstance = this;
	// ��ʼ���ļ�Ŀ¼
	SdcardConfig.getInstance().initSdcard();
	// ��׽�쳣
	AppUncaughtExceptionHandler crashHandler = AppUncaughtExceptionHandler.getInstance();
	crashHandler.init(getApplicationContext());
}
```

> �����ļ�Ŀ¼���쳣��Ϣ������sd���е�Ŀ¼����������������app��ȫ�ֲ����쳣�����������Զ���Ĳ������ǵ�����

```java
/**
 * ��ʼ��������
 *
 * @param context
 */
public void init(Context context) {
	applicationContext = context.getApplicationContext();
	crashing = false;
	//��ȡϵͳĬ�ϵ�UncaughtException������
	mDefaultHandler = Thread.getDefaultUncaughtExceptionHandler();
	//���ø�CrashHandlerΪ�����Ĭ�ϴ�����
	Thread.setDefaultUncaughtExceptionHandler(this);
}
```

������Ϲ��̺󣬽�����Ҫ��д`uncaughtException`������

```java
@Override
public void uncaughtException(Thread thread, Throwable ex) {
	if (crashing) {
		return;
	}
	crashing = true;

	// ��ӡ�쳣��Ϣ
	ex.printStackTrace();
	// ����û�д����쳣 ����Ĭ���쳣����Ϊ�� �򽻸�ϵͳ����
	if (!handlelException(ex) && mDefaultHandler != null) {
		// ϵͳ����
		mDefaultHandler.uncaughtException(thread, ex);
	}
	byebye();
}

private void byebye() {
	android.os.Process.killProcess(android.os.Process.myPid());
	System.exit(0);
}
```

��Ȼ�������Լ������쳣�����Ի���ִ��`handlelException(ex)`������

```java
private boolean handlelException(Throwable ex) {
	if (ex == null) {
		return false;
	}
	try {
		// �쳣��Ϣ
		String crashReport = getCrashReport(ex);
		// TODO: �ϴ���־��������
		// ���浽sd��
		saveExceptionToSdcard(crashReport);
		// ��ʾ�Ի���
		showPatchDialog();
	} catch (Exception e) {
		return false;
	}
	return true;
}
```

��ȡ���쳣��Ϣ����ϵͳ��Ϣ��app�汾��Ϣ���Լ��ֻ���������Ϣ�ȣ�

```java
/**
 * ��ȡ�쳣��Ϣ
 * @param ex
 * @return
 */
private String getCrashReport(Throwable ex) {
	StringBuffer exceptionStr = new StringBuffer();
	PackageInfo pinfo = CrashApplication.getInstance().getLocalPackageInfo();
	if (pinfo != null) {
		if (ex != null) {
			//app�汾��Ϣ
			exceptionStr.append("App Version��" + pinfo.versionName);
			exceptionStr.append("_" + pinfo.versionCode + "\n");

			//�ֻ�ϵͳ��Ϣ
			exceptionStr.append("OS Version��" + Build.VERSION.RELEASE);
			exceptionStr.append("_");
			exceptionStr.append(Build.VERSION.SDK_INT + "\n");

			//�ֻ�������
			exceptionStr.append("Vendor: " + Build.MANUFACTURER+ "\n");

			//�ֻ��ͺ�
			exceptionStr.append("Model: " + Build.MODEL+ "\n");

			String errorStr = ex.getLocalizedMessage();
			if (TextUtils.isEmpty(errorStr)) {
				errorStr = ex.getMessage();
			}
			if (TextUtils.isEmpty(errorStr)) {
				errorStr = ex.toString();
			}
			exceptionStr.append("Exception: " + errorStr + "\n");
			StackTraceElement[] elements = ex.getStackTrace();
			if (elements != null) {
				for (int i = 0; i < elements.length; i++) {
					exceptionStr.append(elements[i].toString() + "\n");
				}
			}
		} else {
			exceptionStr.append("no exception. Throwable is null\n");
		}
		return exceptionStr.toString();
	} else {
		return "";
	}
}
```

���쳣��Ϣ���浽sd������Ҿ��ÿ�ѡ�ɣ������ϴ�������˻��Ǻ��б�Ҫ�ģ�

```java
/**
 * ������󱨸浽sd��
 * @param errorReason
 */
private void saveExceptionToSdcard(String errorReason) {
	try {
		Log.e("CrashDemo", "AppUncaughtExceptionHandlerִ����һ��");
		String time = mFormatter.format(new Date());
		String fileName = "Crash-" + time + ".log";
		if (SdcardConfig.getInstance().hasSDCard()) {
			String path = SdcardConfig.LOG_FOLDER;
			File dir = new File(path);
			if (!dir.exists()) {
				dir.mkdirs();
			}
			FileOutputStream fos = new FileOutputStream(path + fileName);
			fos.write(errorReason.getBytes());
			fos.close();
		}
	} catch (Exception e) {
		Log.e("CrashDemo", "an error occured while writing file..." + e.getMessage());
	}
}
```

������sd���е��쳣�ļ���ʽ��

![�쳣��Ϣ](http://upload-images.jianshu.io/upload_images/1159224-1b1efcc2ad2642cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> �����ϴ������������ͱȽ�����ˣ����Խ������ļ��ϴ��������ϴ��쳣��Ϣ���ַ��������Ժͺ�˿�����Ա��ϡ�


��Ϊ�����쳣����Ҫ���Ϲرյ�app�������`byebye`�������ǽ�app��������ɱ�����������Ҫ��ʾ��ʾ�Ի�������Ҫ���µ�����ջ�д�`activity`��

```java
public static Intent newIntent(Context context, String title, String ultimateMessage) {

	Intent intent = new Intent();
	intent.setClass(context, PatchDialogActivity.class);
	intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);

	intent.putExtra(EXTRA_TITLE, title);
	intent.putExtra(EXTRA_ULTIMATE_MESSAGE, ultimateMessage);
	return intent;
}
```

�Ի����и���������������ѡ��������̵�ʵ�֣�

```java
private void restart() {
	Intent intent = getBaseContext().getPackageManager().getLaunchIntentForPackage(getBaseContext().getPackageName());
	intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
	startActivity(intent);
	super.onDestroy();
}
```

Դ�����ַ:[https://github.com/shenhuniurou/BlogDemos/tree/master/CrashDemo](https://github.com/shenhuniurou/BlogDemos/tree/master/CrashDemo)��ӭstar��



