title: 横竖屏切换时候Activity的生命周期
date: 2014-12-05 04:20:12

---
1、新建一个Activity，并把各个生命周期打印出来

<!-- more -->

2、运行Activity，得到如下信息

	onCreate-->
	onStart-->
	onResume-->	

3、按crtl+f12切换成横屏时

	onSaveInstanceState-->
	onPause-->
	onStop-->
	onDestroy-->
	onCreate-->
	onStart-->
	onRestoreInstanceState-->
	onResume-->

4、再按crtl+f12切换成竖屏时，发现打印了两次相同的log

	onSaveInstanceState-->
	onPause-->
	onStop-->
	onDestroy-->
	onCreate-->
	onStart-->
	onRestoreInstanceState-->
	onResume-->
	onSaveInstanceState-->
	onPause-->
	onStop-->
	onDestroy-->
	onCreate-->
	onStart-->
	onRestoreInstanceState-->
	onResume-->

5、修改AndroidManifest.xml，把该Activity添加 android:configChanges="orientation"，执行步骤3

	onSaveInstanceState-->
	onPause-->
	onStop-->
	onDestroy-->
	onCreate-->
	onStart-->
	onRestoreInstanceState-->
	onResume-->

6、再执行步骤4，发现不会再打印相同信息，但多打印了一行onConfigChanged

	onSaveInstanceState-->
	onPause-->
	onStop-->
	onDestroy-->
	onCreate-->
	onStart-->
	onRestoreInstanceState-->
	onResume-->
	onConfigurationChanged-->

7、把步骤5的android:configChanges="orientation" 改成 android:configChanges="orientation|keyboardHidden"，执行步骤3，就只打印onConfigChanged

	onConfigurationChanged-->

8、执行步骤4

	onConfigurationChanged-->
	onConfigurationChanged-->

总结：

1. 不设置Activity的android:configChanges时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次

2. 设置Activity的android:configChanges="orientation"时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次

3. 设置Activity的android:configChanges="orientation|keyboardHidden"时，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法

总结一下整个Activity的生命周期

补充一点，当前Activity产生事件弹出Toast和AlertDialog的时候Activity的生命周期不会有改变

Activity运行时按下HOME键(跟被完全覆盖是一样的)：

	onSaveInstanceState--> 
	onPause--> 
	onStop-->      
	onRestart-->
	onStart--->
	onResume-->

Activity未被完全覆盖只是失去焦点：

	onPause--->onResume