## 后台播放

### 功能描述

点播和回放支持后台及锁屏播放。

### 实现思路

实际上我们的 SDK 已经支持后台播放和锁屏播放了，之前控制后台播放的方法是是否在界面 onPause 的时候暂停播放，这次功能实际上是在播放视频时在通知栏创建一个通知并实时更新视频状态。

### 具体实现

#### 创建通知渠道

从 Android 8.0 开始，必须为所有的通知分配渠道，否则通知将不会显示。

![检查并创建通知](https://github.com/Yuyi-Chen/MarkDown/raw/main/picture/checkOrCreateNotificationChannel.png)

channerlId 是通知渠道的唯一标识，先根据通知渠道 ID 查询是否已存在该通知渠道，若不存在，则创建该渠道。

创建通知渠道后检测用户是否打开通知开关，若未打开，则跳转到通知设置界面，上图中方法为 `showNotificationDialog()`。

![showNotificationDialog() 方法](https://github.com/Yuyi-Chen/MarkDown/raw/main/picture/showNotificationDialog.png)

#### 视频资源准备好之后创建并发送通知

第一步创建完通知渠道后，就可以随时发布通知了，我们在视频信息初始化完成之后发布第一条通知，因为此时可以拿到视频的所有信息，回调为 `videoView.getObservableVideoStatus()`。创建通知调用 

![创建通知方法](https://github.com/Yuyi-Chen/MarkDown/raw/main/picture/createNotification.png)

可以看到这里创建通知是为类里面的属性进行赋值，其最终调用的方法是：

```kotlin
private void createNotification() {
    PlaybackStateCompat.Builder stateBuilder = new PlaybackStateCompat.Builder()
            .setActions(PlaybackStateCompat.ACTION_PLAY |
                    PlaybackStateCompat.ACTION_PLAY_PAUSE |
                    PlaybackStateCompat.ACTION_SEEK_TO)
            .setState(PlaybackStateCompat.STATE_PLAYING, currentDuration, 1f, 0);
    mediaSession.setPlaybackState(stateBuilder.build());
    mediaSession.setMetadata(new MediaMetadataCompat.Builder()
            .putBitmap(MediaMetadataCompat.METADATA_KEY_ALBUM_ART, BitmapFactory.decodeResource(getResources(), largeIcon))
            .putString(
                    MediaMetadataCompat.METADATA_KEY_TITLE,
                    mContentTitle
            )
            .putLong(
                    MediaMetadataCompat.METADATA_KEY_DURATION,
                    maxDuration
            )
            .build()
    );
    Intent contentIntent = new Intent(this, toActivity);
    contentIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_SINGLE_TOP);
    PendingIntent contentPendingIntent = PendingIntent.getActivity(this, 0, contentIntent, 0);
    Intent controlIntent = new Intent(UIEventKey.ACTION_NOTIFICATION_CONTROL_PLAY);
    controlIntent.setPackage(getPackageName());
    PendingIntent controlPendingIntent = PendingIntent.getBroadcast(this, 0, controlIntent, 0);
    notification = new NotificationCompat.Builder(this, channelId)
            .setContentTitle(mContentTitle)
            .setContentText(mContentText)
            .setLargeIcon(BitmapFactory.decodeResource(getResources(), largeIcon))
            .setSmallIcon(smallIcon)
            .setContentIntent(contentPendingIntent)
            .addAction(new NotificationCompat.Action(playIcon, "pause_or_play", controlPendingIntent))
            .setStyle(
                    new androidx.media.app.NotificationCompat.MediaStyle()
                            .setMediaSession(mediaSession.getSessionToken())
                            .setShowActionsInCompactView(0)
            )
            .setAutoCancel(false)
            .setShowWhen(false)
            .build();
    notification.flags = NotificationCompat.FLAG_NO_CLEAR;
    NotificationManagerCompat notificationManagerCompat = NotificationManagerCompat.from(this);
    notificationManagerCompat.notify(notificationId, notification);
}
```

#### 播放时更新通知

在播放器进行播放的时候，我们要实时更新播放进度，其本质就是更改属性值然后创建一条 cnannelId 一样的通知，这样就完成了通知的更新。

![更新方法](https://github.com/Yuyi-Chen/MarkDown/raw/main/picture/updateNotification.png)

#### 播放完成后销毁通知。

在最后退出的时候需要销毁通知。

![销毁通知](https://github.com/Yuyi-Chen/MarkDown/raw/main/picture/destroyNotification.png)

## 聊天快捷回复

### 功能描述

在发送消息的输入框上方增加一行快捷回复词并可以滑动，点击快捷回复词直接输入到输入框，快捷回复词根据点击次数排序，并存在本地。

### 实现思路

1. 创建View是请求数据。
2. 收到返回数据并与本地数据合并。
3. 点击后更新点击次数并保存。

### 具体实现

#### 读取本地快捷回复

![读取本地快捷回复](https://github.com/Yuyi-Chen/MarkDown/raw/main/picture/getLocalQuickReply.png)

#### 获取远端快捷回复并与本地快捷回复合并

![获取远端快捷回复](https://github.com/Yuyi-Chen/MarkDown/raw/main/picture/getQuickReplyFromServer.png)

![combineQuickReply.png](https://github.com/Yuyi-Chen/MarkDown/raw/main/picture/combineQuickReply.png))

#### 点击时更新次数

![quickReplyClickListener.png](https://github.com/Yuyi-Chen/MarkDown/raw/main/picture/quickReplyClickListener.png)

#### 销毁时保存数据

![destroyNotification.png](https://github.com/Yuyi-Chen/MarkDown/raw/main/picture/destroyNotification.png)

