---
layout: post
keywords: Android Developer
description: Android Developer读书笔记
title: "[Training] Managing Audio Playback"
categories: [开发文档]
tags: [Android读书笔记]
group: archive
icon: file-alt
---
{% include site/setup %}
<hr>
重读Android Developer笔记核心记录
<hr>

#**Controlling Your App’s Volume and Playback**

##**Identify Which Audio Stream to Use**

原文重点摘要：

understanding which audio stream your app will use.

Android maintains a separate audio stream for playing music, alarms, notifications, the incoming call ringer, system sounds, in-call volume, and DTMF tones.
This is done primarily to allow users to control the volume of each stream independently.

Most of these streams are restricted to system events, so unless your app is a replacement alarm clock,
you’ll almost certainly be playing your audio using the STREAM_MUSIC stream.

精髓指点：

理解那种类型的流是你需要的。Android将音频流分类为playing music, alarms, notifications, the incoming call ringer, system sounds, in-call volume, and DTMF tones。
这样做是主要是为了允许用户独立控制每个流的音量。

大多数这些流是限制系统事件，所以除非你的应用程序是一个替代的闹钟，你几乎肯定会使用STREAM_MUSIC玩你的音频流。

##**Use Hardware Volume Keys to Control Your App’s Audio Volume**

原文重点摘要：

By default, pressing the volume controls modify the volume of the active audio stream. If your app isn't currently playing anything,
hitting the volume keys adjusts the ringer volume.

If you've got a game or music app, then chances are good that when the user hits the volume keys they want to control the volume of the game or music,
even if they’re currently between songs or there’s no music in the current game location.

You may be tempted to try and listen for volume key presses and modify the volume of your audio stream that way.
Resist the urge. Android provides the handy setVolumeControlStream() method to direct volume key presses to the audio stream you specify.

Having identified the audio stream your application will be using, you should set it as the volume stream target.
You should make this call early in your app’s lifecycle—because you only need to call it once during the activity lifecycle,
you should typically call it within the onCreate() method (of the Activity or Fragment that controls your media).
This ensures that whenever your app is visible, the volume controls function as the user expects.

{% highlight ruby %}
setVolumeControlStream(AudioManager.STREAM_MUSIC);
{% endhighlight %}

From this point onwards, pressing the volume keys on the device affect the audio stream you specify (in this case “music”)
whenever the target activity or fragment is visible.

精髓指点：

默认情况按下音量键调节的是铃音音量。当你在玩一个游戏或音乐应用程序，那么很可能他们想要的，当用户点击音量键来控制游戏或音乐的音量，
即使他们现在歌曲或游戏的位置。你可能会尝试和监听按键和修改你的音量。Android提供了setVolumeControlStream()方法来指定音频流响应按键。

你应该通常在onCreate()方法(控制你的媒体)的活动或片段。这将确保你的应用程序运行时可作为用户的预期控制。

##**Use Hardware Playback Control Keys to Control Your App’s Audio Playback**

原文重点摘要：

Whenever a user presses one of these hardware keys, the system broadcasts an intent with the ACTION_MEDIA_BUTTON action.

{% highlight ruby %}
<receiver android:name=".RemoteControlReceiver">
    <intent-filter>
        <action android:name="android.intent.action.MEDIA_BUTTON" />
    </intent-filter>
</receiver>
{% endhighlight %}

{% highlight ruby %}
public class RemoteControlReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if (Intent.ACTION_MEDIA_BUTTON.equals(intent.getAction())) {
            KeyEvent event = (KeyEvent)intent.getParcelableExtra(Intent.EXTRA_KEY_EVENT);
            if (KeyEvent.KEYCODE_MEDIA_PLAY == event.getKeyCode()) {
                // Handle key press.
            }
        }
    }
}
{% endhighlight %}

Because multiple applications might want to listen for media button presses,
you must also programmatically control when your app should receive media button press events.

The following code can be used within your app to register and de-register your media button event receiver using the AudioManager.
When registered, your broadcast receiver is the exclusive receiver of all media button broadcasts.

{% highlight ruby %}
AudioManager am = mContext.getSystemService(Context.AUDIO_SERVICE);
...

// Start listening for button presses
am.registerMediaButtonEventReceiver(RemoteControlReceiver);
...

// Stop listening for button presses
am.unregisterMediaButtonEventReceiver(RemoteControlReceiver);
{% endhighlight %}

Typically, apps should unregister most of their receivers whenever they become inactive or invisible (such as during the onStop() callback).
However, it’s not that simple for media playback apps—in fact, responding to media playback buttons is most important when your application
isn’t visible and therefore can’t be controlled by the on-screen UI.

A better approach is to register and unregister the media button event receiver when your application gains and loses the audio focus.
This is covered in detail in the next lesson.

精髓指点：

当按下物理音量控制键时会发送ACTION_MEDIA_BUTTON广播，按照如上前两段代码监听。由于很多应用应用都想去监听物理按下，所以你必须以编程的方式控制你的App
何时监听取消物理按键。如上第三段代码实现注册取消，如果你注册了，你的接收者就是唯一接受这个广播的。

通常，应用程序应该注销大部分接收器时变得不活跃或无形的(比如在onStop()回调)。无论如何，它不是简单的媒体播放应用，应对媒体播放按钮当您的应用程序是最重要的
不可见，因此不能控制屏幕上的UI。更好的方法是注册和注销媒体按钮事件接收者当您的应用程序获得和失去了音频的焦点。这些详细介绍在接下来的课中。这将确保你的应用程序时可见，
音量控制功能作为用户的预期。

<hr>

#**Managing Audio Focus**

原文重点摘要：

With multiple apps potentially playing audio it's important to think about how they should interact.
To avoid every music app playing at the same time,
Android uses audio focus to moderate audio playback—only apps that hold the audio focus should play audio.

Before your app starts playing audio it should request—and receive—the audio focus.
Likewise, it should know how to listen for a loss of audio focus and respond appropriately when that happens.

精髓指点：

多个应用程序同时播放音频必须要考虑他们应该如何交互。为了避免每一个音乐应用同时播放，Android使用设置可以让音频焦点在应该播放的音频上。
你的应用程序开始播放音频前应该请求和接收音频的焦点。同样，当这种情况发生时它应该知道如何处理音频焦点和做出适当反应。

##**Request the Audio Focus**

原文重点摘要：

The following snippet requests permanent audio focus on the music audio stream.
You should request the audio focus immediately before you begin playback,
such as when the user presses play or the background music for the next game level begins.

{% highlight ruby %}
AudioManager am = mContext.getSystemService(Context.AUDIO_SERVICE);
...

// Request audio focus for playback
int result = am.requestAudioFocus(afChangeListener,
                                 // Use the music stream.
                                 AudioManager.STREAM_MUSIC,
                                 // Request permanent focus.
                                 AudioManager.AUDIOFOCUS_GAIN);
   
if (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
    am.registerMediaButtonEventReceiver(RemoteControlReceiver);
    // Start playback.
}
{% endhighlight %}

Once you've finished playback be sure to call abandonAudioFocus().
This notifies the system that you no longer require focus and unregisters the associated AudioManager.OnAudioFocusChangeListener.
In the case of abandoning transient focus, this allows any interupted app to continue playback.

{% highlight ruby %}
// Abandon audio focus when playback complete    
am.abandonAudioFocus(afChangeListener);
{% endhighlight %}

When requesting transient audio focus you have an additional option: whether or not you want to enable "ducking." Normally,
when a well-behaved audio app loses audio focus it immediately silences its playback.
By requesting a transient audio focus that allows ducking you tell other audio apps that it’s acceptable for them to keep playing,
provided they lower their volume until the focus returns to them.

{% highlight ruby %}
// Request audio focus for playback
int result = am.requestAudioFocus(afChangeListener,
                             // Use the music stream.
                             AudioManager.STREAM_MUSIC,
                             // Request permanent focus.
                             AudioManager.AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK);
   
if (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
    // Start playback.
}
{% endhighlight %}

Ducking is particularly suitable for apps that use the audio stream intermittently, such as for audible driving directions.

Whenever another app requests audio focus as described above, its choice between permanent and transient (with or without support for ducking)
audio focus is received by the listener you registered when requesting focus.

精髓指点：

在您的应用程序开始播放任何音频时，它需要得到当前流焦点。调用requestAudioFocus（），如果你的要求是成功的则返回AUDIOFOCUS_REQUEST_GRANTED。
你必须指定你是否希望要求暂时或永久的音频流。上面第一部分的代码片段请求永久的音频流。

一旦你完成播放一定要调用abandonAudioFocus（）。通知系统你不再需要焦点和注销相关AudioManager.OnAudioFocusChangeListener。
在请求瞬间焦点的情况下，结束焦点以后允许任何所中断的应用程序继续播放。如上第二段代码就是如何失去焦点。

当请求瞬间音频焦点，你有一个额外的选项：您是否要启用“闪避（ducking）”。通常情况下，当一个很好的音频应用程序失去音频焦点时立刻沉默了播放。通过请求，
允许闪避（ducking）你告诉其他音频应用程序，这是可以接受他们继续玩一个短暂的音频焦点，只要降低音量，直到焦点返回到他们。如上第三段代码就说明如何设置。

闪避（ducking）是特别适合使用的音频流间歇应用，如用于发声的行车路线。每当另一个应用程序请求如上所述，音频焦点可以有的选择是永久的和瞬态的
（有或没有闪避（ducking）的支持）。通过监听可以得到焦点的状态。

##**Handle the Loss of Audio Focus**

原文重点摘要：

{% highlight ruby %}
OnAudioFocusChangeListener afChangeListener = new OnAudioFocusChangeListener() {
    public void onAudioFocusChange(int focusChange) {
        if (focusChange == AUDIOFOCUS_LOSS_TRANSIENT
            // Pause playback
        } else if (focusChange == AudioManager.AUDIOFOCUS_GAIN) {
            // Resume playback 
        } else if (focusChange == AudioManager.AUDIOFOCUS_LOSS) {
            am.unregisterMediaButtonEventReceiver(RemoteControlReceiver);
            am.abandonAudioFocus(afChangeListener);
            // Stop playback
        }
    }
};
{% endhighlight %}

精髓指点：

如果您的应用程序可以请求音频焦点，当另一个应用程序发出请求时它又会失去焦点，如何使你的应用程序响应的音频焦点失去取决于失去的方式。您请求音频焦点时注册的音频焦点
变化监听器onAudioFocusChange（）回调方法接收描述焦点变化事件的参数。具体来说，可能的重点失去焦点事件反映了上一节说的永久、短暂的损失，闪避（duck）允许短暂的焦点
请求类型。

一般来说，暂时性的失去音频焦点会导致您的应用程序音频流静音，但在其他方面保持相同的状态。您应该继续监测变化的音频焦点，一旦你恢复焦点就要准备从暂停处继续播放。

如果音频焦点损失是永久性的，假定另一个应用程序正在监听音频，而且你的应用程序应该有效地结束自己。在实际应用中，这意味着停止播放，删除媒体按钮监听，
让新的音频播放器专门处理这些事件，并放弃音频焦点。在这一点上，你会期望你恢复播放音频之前需要用户操作（在你的应用程序按着播放）。

在上面的代码中，当我们已经恢复了焦点，我们暂停播放或媒体播放器对象音频失去焦点是暂时的。如果损失是永久性的，注销我们的媒体按钮事件接收器，并停止监听音频焦点的变化。

在音频焦点短暂失去时闪避是允许的，而不是暂停播放，您可以“闪避（duck）”代替。

##**Duck!**

原文重点摘要：

{% highlight ruby %}
OnAudioFocusChangeListener afChangeListener = new OnAudioFocusChangeListener() {
    public void onAudioFocusChange(int focusChange) {
        if (focusChange == AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK) {
            // Lower the volume
        } else if (focusChange == AudioManager.AUDIOFOCUS_GAIN) {
            // Raise it back to normal
        }
    }
};
{% endhighlight %}

精髓指点：

闪避（duck）是降低你的音频流输出音量，从另一个应用程序使瞬间音频容易听到没有完全中断从自己的应用程序的音频的处理。

音频焦点失去最重要的是广播作出反应，但不是唯一的一个。系统会广播一些intent，下面教你如何监听它们，以改善用户的整体体验。

<hr>

#**Dealing with Audio Output Hardware**

原文重点摘要：

Users have a number of alternatives when it comes to enjoying the audio from their Android devices.
Most devices have a built-in speaker, headphone jacks for wired headsets, and many also feature Bluetooth connectivity and support for A2DP audio.

精髓指点：

用户有很多替代品，当涉及到如何听取音乐。大多数设备有内置扬声器，有线耳机，许多还设有蓝牙连接，并支持A2DP的音频。

##**Check What Hardware is Being Used**

原文重点摘要：

How your app behaves might be affected by which hardware its output is being routed to.

You can query the AudioManager to determine if the audio is currently being routed to the device speaker,
wired headset, or attached Bluetooth device as shown in the following snippet:

{% highlight ruby %}
if (isBluetoothA2dpOn()) {
    // Adjust output for Bluetooth.
} else if (isSpeakerphoneOn()) {
    // Adjust output for Speakerphone.
} else if (isWiredHeadsetOn()) {
    // Adjust output for headsets
} else { 
    // If audio plays and noone can hear it, is it still playing?
}
{% endhighlight %}

精髓指点：

应用程序的行为可能会受到硬件输出不同的影响。

你可以通过如上代码段查询AudioManager确定音频当前输出口是扬声器，还是有线耳机，还是蓝牙设备。

##**Handle Changes in the Audio Output Hardware**

原文重点摘要：

When a headset is unplugged, or a Bluetooth device disconnected, the audio stream automatically reroutes to the built in speaker.
If you listen to your music at as high a volume as I do, that can be a noisy surprise.

Luckily the system broadcasts an ACTION_AUDIO_BECOMING_NOISY intent when this happens.
It’s good practice to register a BroadcastReceiver that listens for this intent whenever you’re playing audio.
In the case of music players, users typically expect the playback to be paused—while for games you may choose to significantly lower the volume.

{% highlight ruby %}
private class NoisyAudioStreamReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if (AudioManager.ACTION_AUDIO_BECOMING_NOISY.equals(intent.getAction())) {
            // Pause the playback
        }
    }
}

private IntentFilter intentFilter = new IntentFilter(AudioManager.ACTION_AUDIO_BECOMING_NOISY);

private void startPlayback() {
    registerReceiver(myNoisyAudioStreamReceiver(), intentFilter);
}

private void stopPlayback() {
    unregisterReceiver(myNoisyAudioStreamReceiver);
}
{% endhighlight %}

精髓指点：

当耳机被拔掉，或者蓝牙设备断开连接，音频流会自动重新路由到内置扬声器。

幸运的是，发生这种情况时系统会广播ACTION_AUDIO_BECOMING_NOISY。只要你播放音频，在音乐播放器的情况下，你只需要注册一个BroadcastReceiver监听这个广播。

