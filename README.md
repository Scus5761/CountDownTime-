# CountDownTime-
andriod倒计时
        相信大家在项目里面不少会用到倒计时操作吧，倒计时功能在我们业务开发中使用概率非常高，例如用户操作姿势错误，我们给一个提示，提示是带有倒计时的对话框，当然你会问为什么不直接用Toast呢？

        确实，我们可以直接用土司，但是往往这不是产品想要的，他们觉得没有交互，体验很差，再例如我们用户完成某个任务也可以通过这种倒计时框给用户提醒，倒计时操作再android开发需求很广泛，这里就不多说。

       在andriod中倒计时的实现也有很多种，你可以通过最常用的Handler+Thread方式实现，也可以通过Timer方式实现，当然也可以通过本章要介绍的Google官方推荐的CountDownTimer来实现，当然解决问题的方式又很多，不仅仅就这几种方法，这几种只是个众多方法中的代表，像Handler实现倒计时还有很多变种，例如很Message搭配方式，跟Runnable结合使用方式等等，总之，归根结底都是在子线程进行耗时操作，在UI线程进行更新。

那么现在我们分别介绍这几种不同的方式：
1）通过Handler+Thread/Runnable方式

先上码再说：

handler+Thread

正如大家所见我们在主线程中创建一个Handler,通过handler机制来更新我们的UI，这里更新UI是指我们展示给大家看的倒计时，这里我只介绍倒计时的逻辑和实现，具体应用在什么场景大家自己发挥吧，你可以展示在一个TextView上，也可以弹出一个对话框当作提示，这里我们对倒计时的载体忽略，大家关心倒计时的逻辑并根据情况移植到自己的案例中。

我们在主线程中（即ui线程）创建一个handler,这里我们用到handler消息机制，不明白的可以去看这篇文章www.jianshu.com/p/138363a97da8

在handler中对控件更新内容，这里指秒数，再自减向下循环，然后通过handler将消息发送出去，是通过handler.sendEmptyMessageDelayed(0,1)，第一个参数是延迟时间，第二个参数是时间间隔，当second小于0的时候，这时候倒计时完毕，我们就必须取消发送，通过removeCallbacksAndMessages（）方法，不然handler会内存泄漏导致程序崩溃，就这样完了？？？  似乎我们还确定什么，对，一开始我们就在handler中处理MessageQueue中的消息，但是第一条消息来自哪儿？ 好像没找到，没错，这里我省略掉了我们第一条消息这个引子，再次上图：

创建线程开启循环

这里的show方法大家可以不用关心，因为我这里倒计时放在对话弹框里面，属于对话框的逻辑，大家可以调用new Thread(new MyThread()).start（）直接开启我们的倒计时，这就是handler的实现倒计时，熟悉Handler机制的同学理解起来应该没问题。


2）直接通过Handler方式

这种方式跟上一种区别在于handler是在oncreate()中创建的（initView()在onCreate()方法中），activity创建的时候会调用生命周期函数完成其整个生命过程，在onCreate中会创建hanlder，然后通过obtainMessage()创建Message,最后通过sendEmptyMessage()将消息发送出去，这里message我们只是创建但是空的，因为我们不需要携带消息到UI线程，所以我们向MessageQueue发送一条新消息，然后handler进入循环状态，线程内部Looper开始轮询不断从MwssageQueue中取出消息分发给handler处理，知道所有消息处理完，handler不再发送消息为止，这个过程业务层面的实现也就是handleMessage()中的逻辑，我们在handler初始化的时候可以设定一个倒计时时长——mLimitTime，在oncreate()中就发送一条空消息让handler循环起来，每一次处理消息时候对时长mLimitTime进行判断，在对应的控件上更新当前时长，不要忘了mLimitTime--，不断循环直到我们时长等于0也就是else流程，这里我回调对话框dismiss()方法，在这个方法里面我们需要removeCallbacksAndMessages()取消我们的handler机制，防止出现内存泄漏，跟方式1逻辑上没有太大的差别，主要熟悉handler机制。

不过这种方式我用的是Kotlin实现的，如果第一次接触Kotlin的可能看起来不是很舒服，但是对于会Java的人来说应该不是太大问题，你也可以根据这个逻辑用java实现这个倒计时。

3）Timer倒计时方式

       例外使用Timer和TimerTask也是很简单，用法很固定，所以大家直接根据模板调用就行，首先我们在类初始化的时候创建好Timer和TimerTask，这个和Handler用法很相似，task的内部我们是通过runOnUiThread()方式在ui线程更状态，循环逻辑也是差不多，当我们倒数计时长recLen等于0的时候我们就cancel()取消Timer操作，这和handler的removecallbackandMessage()差不多，后面的Intent大家直接可以忽略，这个是针对业务的逻辑，然后准备工作都完成后，我们在onFinishCreateView()中通过schedule(task,0,1000)开启这个task，这个和使用handler机制中的sendEmptyMessage()作用是一样的，这里的onFinishCreateView()方法也是业务需求方法，大家可以把task.schedule()放到onCreate()或者onResume()启动方法中，开启任务并进行循环，直到条件不合理跳出循环，期间每次循环都更新控件内容。

是不是很简单！！！！

创建Timer

创建任务

4）CountDownTimer Google墙裂推荐方式：

那我们来看一看google到底是如何来封装这一款倒计时的

构造方法：

CountDownTimer构造

millisInFuture：倒计时时长，

countDownInterval：倒计时时间隔



启动程序段

首先会对millisInFuture合理判断，倒计时不合理就直接finish掉，mStopTimeInFuture=SystemClock.elapsedRealtime()+mMillisInFuture获取倒计时终止完成时间，是什么意思呢？ 先拿到们系统当前时长，然后再加上我们倒计时时长，相当于再代码中对终止时间做了一个标记mStopTimeInFuture，接着看，是不是出现很熟悉的代码——sendMessage()，原来CountDownTime内部已经为我们封装好了handler机制，怪不得Google非常推荐得方式，避免开发者开发过程中姿势使用不对导致内存泄漏引发程序崩溃，接着继续看源码



handler消息处理

这里就是处理消息的逻辑，首先google为了程序的健壮性和一致性为当前倒计时任务进行枷锁，大家看这段代码：final longmillisLeft=mStopTimeInFuture-SystemClock.elapsedRealtime();  每次从消息队列中取出消息都会计算剩下时长，同样对剩下时长进行合理判断，有一点需要注意，onTick(millisLeft)这是个啥东西，好像是个回调方法，确实google为我们抽象了两个比较常用的回调方法，当我们没执行一个时间间隔后，就会调用这个回调方法更新我们控件状态等操作，接着看：



向消息队列中发送消息


没错，内部不断循环发送消息，handler的用法主要就是这些，无非是google替我们封装好了逻辑，同理直到millisLeft等于0回调onFinish()方法

回调方法


上面我们将源码简单过了一下，下面我们继续贴代码，看看该怎么用：

定义一个TimerCount继承CountDownTimer



实例化倒计时类并开启任务

onFinish（）和onTick（）方法你可以自由发挥，根据需求来执行逻辑，

其实有个更简单做法，直接new出一个CountDownTimer（）并start这个倒计时就ok了 ，然后在回调里面进行UI更新操作，不用在定义一个TimeCount,之所以这样写因为扩展性好。

到此，我们介绍的几种倒计时基本结束了，说来说去无非就是handler的用法以及对其进行的封装，还不是很了解handler的宝宝去看一下handler的文章，暂时就先到这了，祝大家周末愉快哟！！！
