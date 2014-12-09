title: Android界面刷新
date: 2014-12-05 04:37:46

---
Android的invalidate与postInvalidate都是用来刷新界面的，用法区别在于：

<!-- more -->

invalidate():实例化一个Handler对象，并重写handleMessage方法调用invalidate()实现界面刷新;而在线程中通过sendMessage发送界面更新消息。 

	// 在onCreate()中开启线程
	new Thread(new GameThread()).start();
	
	// 实例化一个handler
	Handler myHandler = new Handler() {
	　　// 接收到消息后处理
	　　public void handleMessage(Message msg) {
	　　　　switch (msg.what) {
	　　　　　　case Activity01.REFRESH:
	　　　　　　　　mGameView.invalidate(); // 刷新界面
	　　　　　　　　break;
	　　　　}
	
	　　　　super.handleMessage(msg);
	　　}
	};
	
	class GameThread implements Runnable {
	　　public void run() {
	　　　　while (!Thread.currentThread().isInterrupted()) {
	　　　　　　Message message = new Message();
	　　　　　　message.what = Activity01.REFRESH;
	　　　　　　// 发送消息
	　　　　　　Activity01.this.myHandler.sendMessage(message);
	　　　　　　try {
	　　　　　　　　Thread.sleep(100);
	　　　　　　} catch (InterruptedException e) {
	　　　　　　　　Thread.currentThread().interrupt();
	　　　　　　}
	　　　　}
	　　}
	}
	
使用postInvalidate则比较简单，不需要handler，直接在线程中调用postInvalidate即可

	class GameThread implements Runnable {
	　　public void run() {
	　　　　while (!Thread.currentThread().isInterrupted()) {
	　　　　　　try {
	　　　　　　　　Thread.sleep(100);
	　　　　　　} catch (InterruptedException e) {
	　　　　　　　　Thread.currentThread().interrupt();
	　　　　　　}
	
	　　　　　　// 使用postInvalidate可以直接在线程中更新界面
	　　　　　　mGameView.postInvalidate();
	　　　　}
	　　}
	}