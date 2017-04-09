## Welcome to Asens blog

my note application is [note.asens.cn](http://note.asens.cn) 
public Channel bind(final SocketAddress localAddress) {
    ChannelFuture future = bindAsync(localAddress);
    future.awaitUninterruptibly();
    return future.getChannel();
}
从bindAsync开始说起吧,异步的绑定端口
awaitUninterruptibly如果没有done会wait()直到被人叫醒,这个操作是属于future的
在bindAsync中

Binder binder = new Binder(localAddress);
ChannelPipeline bossPipeline = pipeline();
bossPipeline.addLast("binder", binder);
Channel channel = getFactory().newChannel(bossPipeline);
final ChannelFuture bfuture = new DefaultChannelFuture(channel, false);
System.out.println(Thread.currentThread().getName()+" : bindFuture addListener");
binder.bindFuture.addListener(new ChannelFutureListener() {
    public void operationComplete(ChannelFuture future) throws Exception {
        if (future.isSuccess()) {
            System.out.println(Thread.currentThread().getName()+" : bfuture 操作完成 回调setSuccess");
            bfuture.setSuccess();
        } else {
            // Call close on bind failure
            bfuture.getChannel().close();
            bfuture.setFailure(future.getCause());
        }
    }
});
return bfuture;

初始化一个binder ,继承SimpleChannelUpstreamHandler,负责绑定端口
初始化一个boss的pipeline,将这个handler放入pipeline里面
初始化一个channel
this是factory
我要初始化一个channel,我要设置对应的属性,打开ServerSocketChannel,设置为非阻塞,感兴趣的事件设置为read
然后创建一个config,然后通知channel open的事件
return new NioServerSocketChannel(this, pipeline, sink, bossPool.nextBoss(), workerPool);

static final ConcurrentMap<Integer, Channel> allChannels = new ConcurrentHashMap<Integer, Channel>();

int interestOps = OP_READ;

NioServerSocketChannel(
        ChannelFactory factory,
        ChannelPipeline pipeline,
        ChannelSink sink, Boss boss, WorkerPool<NioWorker> workerPool) {

    //set各种属性
    //random一个id

    socket = ServerSocketChannel.open();
    socket.configureBlocking(false);
    config = new DefaultServerSocketChannelConfig(socket.socket());
    fireChannelOpen(this);
}
在fireChannelOpen中sendUpstream,里面Event的channel就是刚才的newChannel,也就是NioServerSocketChannel

channel.getPipeline().sendUpstream(
        new UpstreamChannelStateEvent(
                channel, ChannelState.OPEN, Boolean.TRUE));

//找到一个可以获取能够处理Upstream的handler,也就是刚才的binder
public void sendUpstream(ChannelEvent e) {
    DefaultChannelHandlerContext head = getActualUpstreamContext(this.head);
    sendUpstream(head, e);
}

((ChannelUpstreamHandler) ctx.getHandler()).handleUpstream(ctx, e);

binder并没有重载handleUpstream,因此执行他的父类的SimpleChannelUpstreamHandler的handleUpstream

if (e instanceof ChannelStateEvent) {
    switch (evt.getState()) {
        case OPEN:
            if (Boolean.TRUE.equals(evt.getValue())) {
                channelOpen(ctx, evt);
            }
    }
}

binder重载了channelOpen的方法

evt.getChannel().bind(localAddress).addListener(new ChannelFutureListener() {
    public void operationComplete(ChannelFuture future) throws Exception {
        if (future.isSuccess()) {
            bindFuture.setSuccess();
        } else {
            bindFuture.setFailure(future.getCause());
        }
    }
});
调用的是刚才的NioServerSocketChannel的bind方法,在父类AbstractChannel里面
new一个ChannelFuture,这个future是在binder中调用的那个bind产生的future,会在这个sedDownstream结束后返回到binder里面并addListener,这个要看一下究竟是谁调用的这个future的operationComplete
public ChannelFuture bind(SocketAddress localAddress) {
    return Channels.bind(this, localAddress);
}

public static ChannelFuture bind(Channel channel, SocketAddress localAddress) {
    ChannelFuture future = future(channel);
    channel.getPipeline().sendDownstream(new DownstreamChannelStateEvent(
            channel, future, ChannelState.BOUND, localAddress));
    return future;
}
sendDownstream,暂时没有能处理Downstream的handler,这个要交给Sink来处理
public void sendDownstream(ChannelEvent e) {
    DefaultChannelHandlerContext tail = getActualDownstreamContext(this.tail);
    if (tail == null) {
        getSink().eventSunk(this, e);
        return;
    }
    sendDownstream(tail, e);
}
public void eventSunk(
        ChannelPipeline pipeline, ChannelEvent e) throws Exception {
    Channel channel = e.getChannel();
    if (channel instanceof NioServerSocketChannel) {
        handleServerSocket(e);
    } else if (channel instanceof NioSocketChannel) {
        handleAcceptedSocket(e);
    }
}
显然
private static void handleServerSocket(ChannelEvent e) {
    ChannelStateEvent event = (ChannelStateEvent) e;
    NioServerSocketChannel channel =
        (NioServerSocketChannel) event.getChannel();
    ChannelFuture future = event.getFuture();
    ChannelState state = event.getState();
    Object value = event.getValue();

    switch (state) {
    
    case BOUND:
        if (value != null) {
            ((NioServerBoss) channel.boss).bind(channel, future, (SocketAddress) value);
        }
        break;
    
}
channel是设置过boss的,在boss里面注册任务,这个任务是用来给boss线程的
void bind(final NioServerSocketChannel channel, final ChannelFuture future,
          final SocketAddress localAddress) {
    registerTask(new RegisterTask(channel, future, localAddress));
}
首先要new一下registerTask,然后再注册
在task队列里加入一个task,wakenUp设置为true,然后叫醒boss线程
protected final void registerTask(Runnable task) {
    taskQueue.add(task);

    Selector selector = this.selector;
    if (selector != null) {
        if (wakenUp.compareAndSet(false, true)) {
            selector.wakeup();
        }
    }
}
然后就要分2段来讨论了,一边boss线程睡醒要开始处理task,task.run(),来做真正的绑定
一边主线程要继续添加addListener,然后继续处理newChannel之后的新建ChannelFuture来接收bindFuture的success事件,这是一个连续或是链式的异步调用,把bindFuture直接返回去究竟行不行,为什么要这么设计,而且写了这么多东西真正有用的就4句话,这究竟实在干什么,真的很优雅吗?不过这倒是一个典型的生产者消费者的模型


boss线程

这个future也就是之前bind(localAddress)产生的future,这个futur被扔进了DownstreamChannelStateEvent,放到了downstream里,sink接住,sink调用event里面的channel的boss的bind方法把这个future又传了出去,扔进了registerTask里面,然后这个registerTask被扔进了taskQueue里面,然后被boss线程拿出来,并给这个future设置了success

processTaskQueue();

final Runnable task = taskQueue.poll();
task.run();

//NioServerBoss$RegisterTask->run()
public void run() {
    boolean bound = false;
    boolean registered = false;
    
        channel.socket.socket().bind(localAddress, channel.getConfig().getBacklog());
        bound = true;
        future.setSuccess();
        fireChannelBound(channel, channel.getLocalAddress());
        channel.socket.register(selector, SelectionKey.OP_ACCEPT, channel);

        registered = true;
    
}

setSuccess很多操作和awaitUninterruptibly有共同之处
设置done为true,并通知所用的listener
这个存在一个时序的问题,究竟是boss线程先完成并设置success还是主线程先添加完addListener,使得DefaultChannelFuture里面里面的firstListener不再为null影响着这个程序到底是怎么运行的,不过这个无论怎么运行都是对的.运行大概5次有1次是主线程先跑完的
如果是主线程先跑完的这个notifyListeners就能直接运行,否则就什么都没发生
public boolean setSuccess() {
    synchronized (this) {
        if (done) return false;
        done = true;
        if (waiters > 0) notifyAll();
    }

    notifyListeners();
    return true;
}

private void notifyListeners() {
    if (firstListener != null) {
        notifyListener(firstListener);
        firstListener = null;

        if (otherListeners != null) {
            for (ChannelFutureListener l: otherListeners) {
                notifyListener(l);
            }
            otherListeners = null;
        }
    }
}

private void notifyListener(ChannelFutureListener l) {
    //回调
    l.operationComplete(this);
}
不过boss被我debug了这么长时间肯定是主线程先跑完了,但是主线程现在不知道到哪了,估计在future.awaitUninterruptibly()等着呢
事件完成回调之前addListener里面ChannelFutureListener的operationComplete
这个bind(localAddress)产生的future是一个隐式的future,这个future在上述的过程中执行完成,被boss线程setSuccess
然后在这个隐式的future的listener的operationComplete -->bindFuture.setSuccess()
然后触发bindFuture的operationComplete,调用bfuture的setSuccess()
然后future.awaitUninterruptibly()结束等待进入下一步
当我调试到bfuture的setSuccess时,会有waiters在等待,在notifyAll的时候,main线程结束等待并执行完成
修改源代码,返回binder.bindFuture,不再创建bfuture,运行完全没问题,这应该算是为了不返回一个内部类的变量而作的优雅的选择吧
evt.getChannel().bind(localAddress).addListener(new ChannelFutureListener() {
    public void operationComplete(ChannelFuture future) throws Exception {
        if (future.isSuccess()) {
            bindFuture.setSuccess();
        }
    }
});

public synchronized boolean isSuccess() {
    return done && cause == null;
}

final ChannelFuture bfuture = new DefaultChannelFuture(channel, false);
System.out.println(Thread.currentThread().getName()+" : bindFuture addListener");
binder.bindFuture.addListener(new ChannelFutureListener() {
    public void operationComplete(ChannelFuture future) throws Exception {
        if (future.isSuccess()) {
            System.out.println(Thread.currentThread().getName()+" : bfuture 操作完成 回调setSuccess");
            bfuture.setSuccess();
        } else {
            // Call close on bind failure
            bfuture.getChannel().close();
            bfuture.setFailure(future.getCause());
        }
    }
});
return bfuture;

回到boss线程,boss线程把隐式的future.setSuccess之后调用了上述的一大串操作,然后boss在
fireChannelBound(channel, channel.getLocalAddress());
channel.socket.register(selector, SelectionKey.OP_ACCEPT, channel);
之后就运行完成了准备回去睡觉了
但binder并没有准备处理channelBound(ctx, evt)
然后注册下就准备睡了

这就是这4句话的执行过程和对应的时序分支


main线程

main线程在bind完了之后的动作
也就是这个

evt.getChannel().bind(localAddress).addListener(new ChannelFutureListener()
addListener,就是判断这个future的done是否被设置成true,也就是说是否被setSuccess了(或是以其他的方式结束了)
如果done为true,那么直接notifyListener()
如果没有的话,就进入这个future的listener list,至于为什么有firstListener和otherListeners可能是出于性能的考虑吧


public void addListener(ChannelFutureListener listener) {
    if (listener == null) {
        throw new NullPointerException("listener");
    }

    boolean notifyNow = false;
    synchronized (this) {
        if (done) {
            notifyNow = true;
        } else {
            if (firstListener == null) {
                firstListener = listener;
            } else {
                if (otherListeners == null) {
                    otherListeners = new ArrayList<ChannelFutureListener>(1);
                }
                otherListeners.add(listener);
            }

            if (listener instanceof ChannelFutureProgressListener) {
                if (progressListeners == null) {
                    progressListeners = new ArrayList<ChannelFutureProgressListener>(1);
                }
                progressListeners.add((ChannelFutureProgressListener) listener);
            }
        }
    }

    if (notifyNow) {
        notifyListener(listener);
    }
}

主线程在添加了2个addListener之后,bind就结束了,进入bind外边的await等待
在大部分情况,boss线程的wakeUp还是挺快的,addListener操作会直接进入notifyListener()

