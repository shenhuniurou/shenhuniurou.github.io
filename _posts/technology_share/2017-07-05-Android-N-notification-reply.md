---
layout: post
title: Android N在通知栏上实现直接回复消息
category: 技术分享
tags: Android通知
---





Android N 版本中的通知又做了进一步的改进。主要改进了如下几点：

– 新的 UI 效果

– 增强对自定义 View 的支持

– 支持通知内直接回复

– 新的 MessagingStyle 样式通知

– 聚合通知 同一类型通知可以聚合一起了，再也不用担心用户手机满屏都是通知了




刚好，我司的app是一款社交类型的app，为了适配Android N的这些特性，于是花了点时间给自己的app加上了通知栏直接回复的功能。


```java
public static void sendNotification(Context context, String tickerText, String title, String content, Intent intent, String user, int notifyId) {

    // 初始化NotificationManager
    NotificationManager messageNotificatioManager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
    
    // 创建通知
    NotificationCompat.Builder builder = new NotificationCompat.Builder(context);
    builder.setTicker(content);
    builder.setContentInfo(tickerText);
    builder.setContentText(content);
    builder.setContentTitle(title);
    builder.setSmallIcon(R.drawable.icon_push);
    builder.setAutoCancel(true);

    // 设置通知的优先级(悬浮通知)
    builder.setPriority(NotificationCompat.PRIORITY_MAX);
    
    // 根据用户的偏好设置通知是否有声音和震动
    boolean hasVoice = AppDataCache.getInstance().getIsVoice();
    boolean hasVibrate = AppDataCache.getInstance().getIsShake();
    if (hasVoice) {
        // 使用系统通知声音
        Uri alarmSound = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
        // 设置通知的提示音
        builder.setSound(alarmSound);
    }
    if (hasVibrate) {
        long [] pattern = { 70, 150, 70, 150 };
        // 设置通知的震动
        builder.setVibrate(pattern);
    }

    builder.setWhen(System.currentTimeMillis());
    pendIntent = PendingIntent.getActivity(context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
    builder.setContentIntent(pendIntent);
    
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
        adaptAndroidN(context, builder, user);
    }

    messageNotification = builder.build();
    messageNotification.flags = Notification.FLAG_AUTO_CANCEL;
    messageNotificatioManager.notify(notifyId, messageNotification);
}
```

另外像微信QQ，用户在设置通知是否有声音时，点击选中时会播放一遍系统通知声音，具体做法是这样的：

```java
// 播放系统通知铃声
public static void palySound(Context context) {
    Uri soundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
    MediaPlayer mMediaPlayer = MediaPlayer.create(context, soundUri);
    mMediaPlayer.setLooping(false);
    mMediaPlayer.start();
}
```

其中RingtoneManager.TYPE_NOTIFICATION的值有如下几种：

```java
public static final int TYPE_RINGTONE = 1;
public static final int TYPE_NOTIFICATION = 2;
public static final int TYPE_ALARM = 4;
public static final int TYPE_ALL = TYPE_RINGTONE | TYPE_NOTIFICATION | TYPE_ALARM;
```

也就是通知声音、铃声声音、闹钟声音和这三种的组合。

在Android N上通知要显示时间，必须要设置下面这句才可以

```java
builder.setShowWhen(true);
```

上面发送通知的方法中，直接在通知栏中回复的关键代码就是


```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
    adaptAndroidN(context, builder, user);
}
```

```java
private static void adaptAndroidN(Context context, NotificationCompat.Builder builder, String user) {
    builder.setShowWhen(true);
    String replyLabel = "回复";
    RemoteInput remoteInput = new RemoteInput.Builder(KEY_TEXT_REPLY)
        .setLabel(replyLabel)
        .build();
    Intent intent = new Intent(context, ReplyService.class);
    intent.putExtra("userId", user);
    PendingIntent pendingIntent = PendingIntent.getService(context, 0, intent, PendingIntent.FLAG_CANCEL_CURRENT);
    NotificationCompat.Action action = new NotificationCompat.Action.Builder(R.drawable.icon_push, replyLabel, pendingIntent)
            .addRemoteInput(remoteInput)
            .setAllowGeneratedReplies(true)
            .build();
    builder.addAction(action);
}
```

这里我只添加了一个回复按钮，是可以添加多个的，看源码发现addAction其实是把这个action添加到一个ArrayList中的，所有肯定是可以显示多个按钮的。

回复输入内容后，点击右边的发送按钮，这个后续动作需要我们通过PendingIntent来实现，比如我这里是要将这条消息发送给对方，所以我使用了IntentService来完成。

```java
public class ReplyService extends IntentService implements SendMessageCallback {

    private static final String KEY_TEXT_REPLY = "key_text_reply";
    private DatabaseHelper helper;
    private SocketMessage tmpMsg;
    private String userId;

    public ReplyService() {
        super("ReplyService");
    }


    @Override protected void onHandleIntent(@Nullable Intent intent) {
        Bundle remoteInput = RemoteInput.getResultsFromIntent(intent);
        String message = null;
        if (remoteInput != null) {
            message = remoteInput.getCharSequence(KEY_TEXT_REPLY).toString();
        }

        if (StringUtil.isBlank(message)) {
            return;
        }

        if (intent.hasExtra("userId")) {
            userId = intent.getStringExtra("userId");
        }

        helper = new DatabaseHelper(this);

        SocketMessage socketMessage = new SocketMessage();
        socketMessage.setMsgType(1);// 文本
        socketMessage.setSender(0); // 自己发送的
        socketMessage.setUser(userId);// 对方用户id
        socketMessage.setMsgContent(message);
        socketMessage.setMsgStamp(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
        socketMessage.setMsgStatus(0);
        socketMessage.setReadStatus(1);

        // 将本条发送的消息保存到数据库
        helper.save(socketMessage);
        tmpMsg = socketMessage;

        // 发送消息
        SocketManager.sendMessage(this, socketMessage, this.getApplicationContext());
    }


    /**
     * 回调发送结果
     * @param code
     */
    @Override public void getCode(int code) {
        if (code == 1) {
            // 发送成功
            tmpMsg.setMsgStatus(1);
        } else {
            // 发送失败
            tmpMsg.setMsgStatus(2);
        }
        // 更新数据库中该记录的发送状态值
        helper.updateMsgStatus(tmpMsg);

        // 查询与该用户的未读消息数
        int unReadCount = helper.findUnreadMessageCount(userId);

        // 设置该用户所有的消息为已读
        helper.setMessagesRead(userId);
        
        // 更新会话记录表
        List<ChatPerson> chatPersonList = helper.findByCondition("userId", userId);
        ChatPerson cp = chatPersonList.get(0);
        cp.setMessage(tmpMsg.getMsgContent());
        cp.setLastseen(tmpMsg.getMsgStamp());
        helper.update(cp);
        
        // 发送广播更新会话列表的未读消息数
        sendBroadcast(new Intent(BroadcastAction.ACTION_REFRESH_CHAT_LIST));
        
        // 发送广播更新底部tab上的未读消息数（能还有其他用户的未读消息）
        int num = AppDataCache.getInstance().getUnReadMsgNum();
        AppDataCache.getInstance().setUnReadMsgNum(AppDataCache.getInstance().getUnReadMsgNum() - unReadCount);
        sendBroadcast(new Intent(BroadcastAction.ACTION_UPDATE_MESSAGE_NUM));
        
        // 操作完之后清除通知
        NotificationUtil.clearChatNotifications(this);
    }

}
```

关键点就是在onHandleIntent方法中通过RemoteInput获取到刚输入的信息：

```java
Bundle remoteInput = RemoteInput.getResultsFromIntent(intent);
String message = null;
if (remoteInput != null) {
    message = remoteInput.getCharSequence(KEY_TEXT_REPLY).toString();
}
```

执行完一系列操作后，IntentService自动结束，完成。

结尾处关于通知我有一点疑惑，就是在Android5.0之后的系统，像QQ微信是刚下载安装后它就有悬浮锁屏显示通知的权限，而普通的app是没有的，即使我将优先级设置为PRIORITY_MAX还是不行。
不知道是不是国内的手机厂商将QQ微信加入白名单了啊？如果让用户手动去设置这招怕是不行，因为并不是每个用户都像Android开发者一样爱去折腾手机。我目前是用Android7.1的原生系统测试的，是默认可以显示悬浮通知的。




**参考文章**

[Android Nougat 的通知改进](http://blog.chengyunfeng.com/?p=1006)