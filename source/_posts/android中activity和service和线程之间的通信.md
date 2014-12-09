title: Android中Activity和Service和线程之间的通信
date: 2014-12-05 06:33:54

---
Activity、Service和线程应该是Android编程中最常见的几种类了，几乎大多数应用程序都会涉及到这几个类的编程，自然而然的，也就会涉及到三者之间的相互通信，本文就试图简单地介绍一下这三者通信的方式。

想写这篇文章的起因是，笔者跟几个同学在做一个Android上的应用，起初代码写得很凌乱，因为我在Activity中直接创建了线程，去执行某些任务。但是我们知道线程可能需要运行的时间比较长，而Android在内存不足的时候，会将一些Activity销毁，这样线程就会失去了管理的对象，从而使程序发生意想不到的结果。此外，在Activity中创建线程，线程跟Activity的通信也比较麻烦，一般借助Handler类来进行通信（[http://blog.sina.com.cn/s/blog_3fe961ae0100mvc5.html](http://blog.sina.com.cn/s/blog_3fe961ae0100mvc5.html)）。

与Activity相比，Service一般“默默”地运行在后台，生命周期比较长，所以它更合适去为主程序提供某些服务，创建线程并管理线程。因此，笔者将原程序改成三层架构，从高到低分别为：Activity层--Service层--Thread层。Activity将需要的服务“告诉”Service层，Service层创建Thread去完成服务。Thread将任务的进度、状态、错误信息反馈给Service，Service将这些消息反馈给相关的Activity，并且还可以利用Notification更新通知栏消息。大体的结构就是这样。

1. Activity和Service之间的通信
    1. 利用Handler通信：[http://blog.sina.com.cn/s/blog_3fe961ae0100mvc5.html](http://blog.sina.com.cn/s/blog_3fe961ae0100mvc5.html)

    2. Activity调用startService (Intent service)方法，将消息添加到Intent对象中，这样Service对象可以在调用onStartCommand (Intent intent, int flags, int startId)的时候可以得到这些消息。这种方法很简单，但如果有大量的信息要传递的话，就很麻烦了。因为Service端还要判断一下消息是什么，才能作进一步的动作。

    3. Activity调用bindService (Intent service, ServiceConnection conn, int flags)方法，得到Service对象的一个引用，这样Activity可以直接调用到Service中的方法。具体代码：[http://blog.csdn.net/liuhe688/article/details/6623924](http://blog.csdn.net/liuhe688/article/details/6623924)。

    4. Service向Activity发送消息，除了可以利用Handler外，还可以使用广播，当然Activity要注册相应的接收器。比如Service要向多个Activity发送同样的消息的话，用这种方法就更好。具体方法可以看一下这篇文章：[http://blog.csdn.net/liuhe688/article/details/6641806](http://blog.csdn.net/liuhe688/article/details/6641806)。

2. Service跟Thread之间的通信

    1. Service创建Thread后，如果要对线程进行控制（启动，暂停，停止等），那么Service中应该保留线程的引用，这不用多说。那么有了这个引用，Service就可以直接调用Thread的其它方法了。运行的线程要向Service发送消息的话，通常使用Handler就可以了：[http://blog.sina.com.cn/s/blog_3fe961ae0100mvc5.html](http://blog.sina.com.cn/s/blog_3fe961ae0100mvc5.html)。

3. Activity和Thread之间的通信
	1. 不用多想了，直接使用Handler吧。不推荐Activity直接去创建线程，因为不好管理线程。