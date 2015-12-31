title: Android-frameworks-SoundPool
comments: true
date: 2015-12-31 10:17:46
tags:
	- Android
	- frameworks
	- AOSP

categories: AOSP
updated:
---

###给Android物理按键添加操作声音

目标：通过修改系统源码给物理旋钮添加操作提示音
实现：参照View的touch声音，在最底层的InputFilter接收输入事件，播放声音

>Android声音播放有俩种方式`MediaPlayer`跟`SoundPool`。

>SoundPool —— 适合短促且对反应速度比较高的情况（游戏音效或按键声等）。

>MediaPlayer —— 适合比较长且对时间要求不高的情况。   
	
方案：针对SoundPool与MediaPlayer进行比较，由于旋钮操作较为短促，本次开发采用SoundPool进行。

车载盒子AOSP中封装了输入事件，代码位置:

`/framework/base/services/java/com/android/server/CarInputHub.java`

注意：需在CarInputHub初始化时便对SoundPool进行load，不然play声音时会报 no ready错误并且没有声音。

```java
public CarInputHub(Context context) {
	super(context.getMainLooper());
    mContext = context;

	mSoundPool = new SoundPool(NUM_SOUNDPOOL_CHANNELS, AudioManager.STREAM_MUSIC, 0);
	String filePath = "/system/media/audio/ui/suibian.ogg";
	sampleId = mSoundPool.load(filePath, 1);
	Log.i(TAG, "SoundPool init");
}
```

然后在点击处理点击事件的函数中进行play即可   

```java
@Override
public void onInputEvent(InputEvent inputEvent, int policyFlags) {
	if(inputEvent instanceof KeyEvent){
		KeyEvent event = (KeyEvent) inputEvent;
		if(event.getAction()==KeyEvent.ACTION_UP){
			    playClickSound();
		}
		.
		.
		.
	}
}

private void playClickSound() {
	mSoundPool.play(sampleId, 1, 1, 1, 0, 1.0f);
}
```
