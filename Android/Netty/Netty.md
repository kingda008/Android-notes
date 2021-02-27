[TOC]



netty是什么

![image-20210215154759197](image-20210215154759197.png)

![image-20210215155141979](image-20210215155141979.png)



![image-20210215161616696](image-20210215161616696.png)

## channel

对应socket





## NioEventLoop 对应 Thread

```
  @Override
    protected void run() {
        int selectCnt = 0;
        for (;;) {
```

相当于while(true)服务端一直等待连接（case SelectStrategy.SELECT:）或者客户端循环不断的读数据。

NioEventLoop 起了两个线程，监听客户端连接线程，数据读写线程。



ByteBuf 对应 io bytes

Pipeline 对应逻辑链

ChannelHandler 对应逻辑处理块





#### channel的创建

**问题：服务端socket在哪里初始化？**

**问题：在哪里accept连接**







创建服务端channel



初始化服务端channel



注册selector



端口绑定

![image-20210217162146720](image-20210217162146720.png)



从 ChannelFuture f = b.bind(8888).sync(); 进入ServerBootstrap bind方法

```
public ChannelFuture bind(SocketAddress localAddress) {
        this.validate();
        return this.doBind((SocketAddress)ObjectUtil.checkNotNull(localAddress, "localAddress"));
    }

    private ChannelFuture doBind(final SocketAddress localAddress) {
        final ChannelFuture regFuture = this.initAndRegister();
        final Channel channel = regFuture.channel();
        if (regFuture.cause() != null) {
            return regFuture;
        } else if (regFuture.isDone()) {
            ChannelPromise promise = channel.newPromise();
            doBind0(regFuture, channel, localAddress, promise);
            return promise;
        } else {
            final AbstractBootstrap.PendingRegistrationPromise promise = new AbstractBootstrap.PendingRegistrationPromise(channel);
            regFuture.addListener(new ChannelFutureListener() {
                public void operationComplete(ChannelFuture future) throws Exception {
                    Throwable cause = future.cause();
                    if (cause != null) {
                        promise.setFailure(cause);
                    } else {
                        promise.registered();
                        AbstractBootstrap.doBind0(regFuture, channel, localAddress, promise);
                    }

                }
            });
            return promise;
        }
    }

```

在 initAndRegister 方法中

```
final ChannelFuture initAndRegister() {
        Channel channel = null;

        try {
            channel = this.channelFactory.newChannel();
            this.init(channel);
        } catch (Throwable var3) {
            if (channel != null) {
                channel.unsafe().closeForcibly();
                return (new DefaultChannelPromise(channel, GlobalEventExecutor.INSTANCE)).setFailure(var3);
            }

            return (new DefaultChannelPromise(new FailedChannel(), GlobalEventExecutor.INSTANCE)).setFailure(var3);
        }

        ChannelFuture regFuture = this.config().group().register(channel);
        if (regFuture.cause() != null) {
            if (channel.isRegistered()) {
                channel.close();
            } else {
                channel.unsafe().closeForcibly();
            }
        }

        return regFuture;
    }
```

channelFactory.newChannel(); 创建

```
/**
 * @deprecated Use {@link io.netty.channel.ChannelFactory} instead.
 */
@Deprecated
public interface ChannelFactory<T extends Channel> {
    /**
     * Creates a new channel.
     */
    T newChannel();
}
这是个接口
```

回到上面的  channel = this.channelFactory.newChannel();  这个factory在赋值的？

我们在初始化serverChannel的时候

```
 ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    
                    ---
                    
public B channel(Class<? extends C> channelClass) {
        return this.channelFactory((io.netty.channel.ChannelFactory)(new ReflectiveChannelFactory((Class)ObjectUtil.checkNotNull(channelClass, "channelClass"))));
    }
    
    
```

```
public class ReflectiveChannelFactory<T extends Channel> implements ChannelFactory<T> {
    private final Constructor<? extends T> constructor;

    public ReflectiveChannelFactory(Class<? extends T> clazz) {
        ObjectUtil.checkNotNull(clazz, "clazz");

        try {
            this.constructor = clazz.getConstructor();
        } catch (NoSuchMethodException var3) {
            throw new IllegalArgumentException("Class " + StringUtil.simpleClassName(clazz) + " does not have a public non-arg constructor", var3);
        }
    }

    public T newChannel() {
        try {
            return (Channel)this.constructor.newInstance();
        } catch (Throwable var2) {
            throw new ChannelException("Unable to create Channel from class " + this.constructor.getDeclaringClass(), var2);
        }
    }

    public String toString() {
        return StringUtil.simpleClassName(ReflectiveChannelFactory.class) + '(' + StringUtil.simpleClassName(this.constructor.getDeclaringClass()) + ".class)";
    }
}
```



这里通过反射把传递过来的NioServerSocketChannel.class 调用它的构造函数。

为啥这么设计？

**构造函数做了什么呢？**

![image-20210222231930442](image-20210222231930442.png)



**1,newSocket:构造方法里面直接调用的就是newSocket**

```Java
public NioServerSocketChannel() {
    this(newSocket(DEFAULT_SELECTOR_PROVIDER));
}
```

```
private static final SelectorProvider DEFAULT_SELECTOR_PROVIDER = SelectorProvider.provider();

这里的SelectProvider是Java类。package java.nio.channels.spi
```

```
private static java.nio.channels.ServerSocketChannel newSocket(SelectorProvider provider) {
    try {
        return provider.openServerSocketChannel();
    } catch (IOException var2) {
        throw new ChannelException("Failed to open a server socket.", var2);
    }
}
```

创建出来的是Java类ServerSocketChannel  （package java.nio.channels）

**2，NioServerSocketChannelConfig**

构造方法

```
public NioServerSocketChannel(java.nio.channels.ServerSocketChannel channel) {
    super((Channel)null, channel, 16);
    this.config = new NioServerSocketChannel.NioServerSocketChannelConfig(this, this.javaChannel().socket());
}
```

```
protected java.nio.channels.ServerSocketChannel javaChannel() {
    return (java.nio.channels.ServerSocketChannel)super.javaChannel();
}
```

这里把Java的serverSocket当做构造方法参数传递进来，后面获取，设置服务端参数用的就是这个config。

**3，调用父类构造函数**

```
AbstractNioChannel
protected AbstractNioChannel(Channel parent, SelectableChannel ch, int readInterestOp) {
        super(parent);
        this.ch = ch;
        this.readInterestOp = readInterestOp;

        try {
            ch.configureBlocking(false);//设置服务端非阻塞模式
        } catch (IOException var7) {
            try {
                ch.close();
            } catch (IOException var6) {
                logger.warn("Failed to close a partially initialized socket.", var6);
            }

            throw new ChannelException("Failed to enter non-blocking mode.", var7);
        }
    }
    
```

**4，AbstractNioChannel 创建id,unsafe,pipeline**

继续父类的构造

```
protected AbstractChannel(Channel parent) {
    this.parent = parent;
    this.id = this.newId();//channel 的唯一标识
    this.unsafe = this.newUnsafe();//对应tcp读写的一个类
    this.pipeline = this.newChannelPipeline();//后面会涉及到的一个非常重要的组件
}
```

## 初始化channel

![image-20210224224619128](image-20210224224619128.png)

![image-20210224224726498](image-20210224224726498.png)

上面initAndRegister方法中 newChannel之后就是init();

```
abstract void init(Channel var1) throws Exception;
```

在ServerBootstrap中

```
void init(Channel channel) {
//拿到用户配置的option，通过config配置进去
    setChannelOptions(channel, this.newOptionsArray(), logger);
    //设置属性
    setAttributes(channel, this.newAttributesArray());
    ChannelPipeline p = channel.pipeline();
    final EventLoopGroup currentChildGroup = this.childGroup;
    final ChannelHandler currentChildHandler = this.childHandler;
    final Entry<ChannelOption<?>, Object>[] currentChildOptions = newOptionsArray(this.childOptions);
    final Entry<AttributeKey<?>, Object>[] currentChildAttrs = newAttributesArray(this.childAttrs);
    
    //配置pipeline
    p.addLast(new ChannelHandler[]{new ChannelInitializer<Channel>() {
        public void initChannel(final Channel ch) {
            final ChannelPipeline pipeline = ch.pipeline();
            
            //把用户配置的handler添加进来
            ChannelHandler handler = ServerBootstrap.this.config.handler();
            if (handler != null) {
                pipeline.addLast(new ChannelHandler[]{handler});
            }

            ch.eventLoop().execute(new Runnable() {
                public void run() {
                
                //添加一个默认的handler 
                    pipeline.addLast(new ChannelHandler[]{new ServerBootstrap.ServerBootstrapAcceptor(ch, currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs)});
                }
            });
        }
    }});
}
```

```
static void setChannelOptions(Channel channel, Entry<ChannelOption<?>, Object>[] options, InternalLogger logger) {
    Entry[] var3 = options;
    int var4 = options.length;

    for(int var5 = 0; var5 < var4; ++var5) {
        Entry<ChannelOption<?>, Object> e = var3[var5];
        setChannelOption(channel, (ChannelOption)e.getKey(), e.getValue(), logger);
    }

}

private static void setChannelOption(Channel channel, ChannelOption<?> option, Object value, InternalLogger logger) {
    try {
        if (!channel.config().setOption(option, value)) {
            logger.warn("Unknown channel option '{}' for channel '{}'", option, channel);
        }
    } catch (Throwable var5) {
        logger.warn("Failed to set channel option '{}' with value '{}' for channel '{}'", new Object[]{option, value, channel, var5});
    }

}
```

```
static void setAttributes(Channel channel, Entry<AttributeKey<?>, Object>[] attrs) {
    Entry[] var2 = attrs;
    int var3 = attrs.length;

    for(int var4 = 0; var4 < var3; ++var4) {
        Entry<AttributeKey<?>, Object> e = var2[var4];
        AttributeKey<Object> key = (AttributeKey)e.getKey();
        channel.attr(key).set(e.getValue());
    }

}
```

问题：这里配置的option和attribute都有啥，有什么不同的

保存用户自定义的属性，通过这些属性创建一个连接接入器，当新连接到来accept的时候都会使用这些属性对新的连接做一些配置

## 注册selector

创建完后就是要把这些channel注册到轮询器中