---
title: Android 进程间通信
date: 2017-01-18 13:35:23
categories:
  	- Android
tags:
	- Android
	- IPC
---
要讲 Android 进程通信的话，就不得不先讲讲 Service. Service 是 Android 的四大组件之一，它主要的作用是后台执行操作，Activity 属于带有 UI 界面跟用户进行交互，而 Service 则没有 UI 界面，所有的操作都是基于后台运行完成。并且 Service 跟 Activity 一样也是可以由其它的应用程序调用启动的，而且就算用户切换了应用程序，Service 依旧保持运行。一个组件如果与 Service 进行了绑定( bind ), 就可以跟 Service 进行数据的交互，并且也可以跟不同的进程之间进行交互 (IPC)。通常会使用到 Service 的情况有进行网络请求，音乐的操控，文件的 I/O 操作等。

## Service
### 声明
在 Manifest 里声明 Service, 类似于 Activity，所有的 Service 都要在 Manifest 里面进行声明，如下:
```xml
<manifest ... >
  ...
  <application ... >
      <service android:name=".ExampleService" />
      ...
  </application>
</manifest>
```
查看 [service 标签](https://developer.android.com/guide/topics/manifest/service-element.html) 的官方文档来获取更多信息

<!-- more -->

### 启动
Service 通常是通过以下两种方式进行启动

- startService
- bindService

#### Start Service
当组件(例如 activity)通过调用 `startService()` 来启动 Service 的时候。一旦启动后，Service 就会独立的在后台运行，即使调用的组件已经销毁了，Service 还是可以继续在后台运行。一般情况下，只需要进行一次单独的操作，不需要将操作后的结果返回给调用者的时候，会使用该方式启动 Service。例如，**进行上传或者下载操作**的时候，当操作完成后，Service 应该自行调用 stopService() 或 stopSelf() 来结束运行。

#### Bind Service
当组件(例如 activity)通过调用 `bindService()` 来启动 Service 的时候。这种方式提供了 client - service 的接口，可以让调用组件跟 Service 进行发送请求及返回结果的操作，设置可以进行进程间的通信 (IPC)。只要有一个组件对该 Service 进行了绑定，那该 Service 就不会销毁。并且多个组件可以同时对一个 Service 进行绑定，**只有在所有进行了绑定的组件都解绑的时候，Service 才会销毁**。

尽管两种方式是分开讨论的，但是并不是互斥的关系，使用 `startService` 启动了 Service 后，也是可以通过 `bindService`绑定的。
> **注意**： 虽然 Service 是在后台运行，但是其实还是在**主线程**里进行所有的操作的。Service 在启动时除非单独进行了定义否则并没有在单独的线程或者进程了而都是在**主线程**里。所以这表示任何能堵塞主线程的操作（例如音乐的播放或者网络请求）都应该单独开辟新的线程来进行操作，否则很容易出现 ANR 。

如果某个组件是通过调用 `startService()` 的方式来启动了 Service，**那这个 Service 就会一直在后台运行直到 Service 内部调用 `stopSelf()` 或某个组件调用 `stopService()` 来结束该 Service**。

如果某个组件是通过调用 `bindService()` 的方式来启动了 Service，**那这个 Service 就会一直在后台运行直到该组件与其解绑**。Service 在没有任何组件绑定的时候，系统会将其销毁

关于 Service 更多详细的介绍可以查看[这里](http://www.jianshu.com/p/63792fc84d94)

### Service 生命周期
![service 生命周期图](http://upload-images.jianshu.io/upload_images/854027-d02db439c9b748e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## AIDL (Android Interface Definition Language )
Android IPC 是通过 Binder 实现的,但是 Binder 相关的概念非常复杂,为了方便开发者 Google 就推出了 AIDL (安卓接口定义语言)。通过编写 AIDL 文件，Android Studio 就可以帮我们生成 Binder 通信的相关代码。开发者即使不了解 Binder 机制也可以实现 IPC 了。

### 关键字
#### oneway
正常情况下 Client 调用 AIDL 接口方法时会阻塞，直到 Server 进程中该方法被执行完。oneway 可以修饰 AIDL 文件里的方法，oneway 修饰的方法在用户请求相应功能时不需要等待响应可直接调用返回，非阻塞效果，该关键字可以用来声明接口或者声明方法，如果接口声明中用到了 oneway 关键字，则该接口声明的所有方法都采用 oneway 方式。（注意,如果 Client 和 Server 在同一进程中, oneway 修饰的方法还是会阻塞）
#### in
非基本数据类型和 String 的参数类型必须加参数修饰符, in 的意思是只输入,既最终 Server 端执行完后不会影响到参数对象
#### out
与 in 相反, out 修饰的参数只能由 Server 写入并传递到 Client,而 Client 传入的值并不会传递到 Server
#### inout
被 inout 修饰的参数,既可以从 Client 传递到 Server,也可以 Server 传递到 Client

### AIDL 自动生成文件讲解
Talk is cheap, show you the code. 
```Java
interface ISocketService {

    int getState();

    oneway void registerCallback(in ISocketServiceCallback callback);
    oneway void unregisterCallback(in ISocketServiceCallback callback);

    oneway void runShadowSocks(in Config config);
}
```

IDE 自动生成的代码如下

```Java
public interface ISocketService extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.shadark.app.aidl.ISocketService {
        private static final java.lang.String DESCRIPTOR = "com.shadark.app.aidl.ISocketService";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an com.shadark.app.aidl.ISocketService interface,
         * generating a proxy if needed.
         */
        public static com.shadark.app.aidl.ISocketService asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.shadark.app.aidl.ISocketService))) {
                return ((com.shadark.app.aidl.ISocketService) iin);
            }
            return new com.shadark.app.aidl.ISocketService.Stub.Proxy(obj);
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(DESCRIPTOR);
                    return true;
                }
                case TRANSACTION_getState: {
                    data.enforceInterface(DESCRIPTOR);
                    int _result = this.getState();
                    reply.writeNoException();
                    reply.writeInt(_result);
                    return true;
                }
                case TRANSACTION_registerCallback: {
                    data.enforceInterface(DESCRIPTOR);
                    com.shadark.app.aidl.ISocketServiceCallback _arg0;
                    _arg0 = com.shadark.app.aidl.ISocketServiceCallback.Stub.asInterface(data.readStrongBinder());
                    this.registerCallback(_arg0);
                    return true;
                }
                case TRANSACTION_unregisterCallback: {
                    data.enforceInterface(DESCRIPTOR);
                    com.shadark.app.aidl.ISocketServiceCallback _arg0;
                    _arg0 = com.shadark.app.aidl.ISocketServiceCallback.Stub.asInterface(data.readStrongBinder());
                    this.unregisterCallback(_arg0);
                    return true;
                }
                case TRANSACTION_runShadowSocks: {
                    data.enforceInterface(DESCRIPTOR);
                    com.shadark.app.aidl.Config _arg0;
                    if ((0 != data.readInt())) {
                        _arg0 = com.shadark.app.aidl.Config.CREATOR.createFromParcel(data);
                    } else {
                        _arg0 = null;
                    }
                    this.runShadowSocks(_arg0);
                    return true;
                }
            }
            return super.onTransact(code, data, reply, flags);
        }

        private static class Proxy implements com.shadark.app.aidl.ISocketService {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            @Override
            public int getState() throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                int _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_getState, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.readInt();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }

            @Override
            public void registerCallback(com.shadark.app.aidl.ISocketServiceCallback callback) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeStrongBinder((((callback != null)) ? (callback.asBinder()) : (null)));
                    mRemote.transact(Stub.TRANSACTION_registerCallback, _data, null, android.os.IBinder.FLAG_ONEWAY);
                } finally {
                    _data.recycle();
                }
            }

            @Override
            public void unregisterCallback(com.shadark.app.aidl.ISocketServiceCallback callback) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeStrongBinder((((callback != null)) ? (callback.asBinder()) : (null)));
                    mRemote.transact(Stub.TRANSACTION_unregisterCallback, _data, null, android.os.IBinder.FLAG_ONEWAY);
                } finally {
                    _data.recycle();
                }
            }

            @Override
            public void runShadowSocks(com.shadark.app.aidl.Config config) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    if ((config != null)) {
                        _data.writeInt(1);
                        config.writeToParcel(_data, 0);
                    } else {
                        _data.writeInt(0);
                    }
                    mRemote.transact(Stub.TRANSACTION_runShadowSocks, _data, null, android.os.IBinder.FLAG_ONEWAY);
                } finally {
                    _data.recycle();
                }
            }
        }

        static final int TRANSACTION_getState = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_registerCallback = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
        static final int TRANSACTION_unregisterCallback = (android.os.IBinder.FIRST_CALL_TRANSACTION + 2);
        static final int TRANSACTION_runShadowSocks = (android.os.IBinder.FIRST_CALL_TRANSACTION + 3);
    }

    public int getState() throws android.os.RemoteException;

    public void registerCallback(com.shadark.app.aidl.ISocketServiceCallback callback) throws android.os.RemoteException;

    public void unregisterCallback(com.shadark.app.aidl.ISocketServiceCallback callback) throws android.os.RemoteException;

    public void runShadowSocks(com.shadark.app.aidl.Config config) throws android.os.RemoteException;
}
```
> IDE 用我们编写的 AIDL 文件，帮我们做了如下这些事情:  
1.创建了 ISocketService 的实现类 Stub 和 Stub 的子类 Proxy  
2.Stub 类中实现了有 IBinder 对象转换为 ISocketService 类型的 asInterface, asInterface 中通过 queryLocalInterface(DESCRIPTOR) 方法查看本进程是否有 ISocketService 在 Server 端的实现类(既判断 Server 与 Client 是否在同一进程),如果是同一进程就直接返回 Server 端的 ISocketService 实现者,如果不在同一进程就返回代理对象  
3.Proxy 类中实现了 AIDL 中定义的方法,根据 oneway、in、out、inout 修饰符来生成不同的代码，决定是否向 binder 驱动写入数据或者执行完后向方法参数回写数据。注意：oneway 修饰一个方法后，该方法不阻塞 client 调用线程，但是方法没有返回值，方法参数在执行方法执行完后也不会回写。  
4.Proxy 类中实现的方法最终通过 transact() 方法向 Binder 驱动写入数据(运行在 Client 进程),最终 Stub 类中的 onTransact() 方法会被调用到(运行在 Server 进程),就这样完成一次跨进程方法调用。

### Binder 死亡处理
在进程间通信过程中，很可能出现一个进程死亡的情况。如果这时活着的一方不知道另一方已经死了就会出现问题。那我们如何在 A 进程中获取 B 进程的存活状态呢？
Android 肯定给我们提供了解决方式，那就是 `Binder` 的 `linkToDeath` 和 `unlinkToDeath` 方法, `linkToDeath` 方法需要传入一个 `DeathRecipient` 对象, `DeathRecipient` 类里面有个 `binderDied` 方法,当 `binder` 对象的所在进程死亡, `binderDied` 方法就会被执行,我们就可以在 `binderDied` 方法里面做一些**异常处理,释放资源**等操作了。

示例如下:
```Java
...

    @Override
    public void onServiceConnected(ComponentName name, IBinder binder) {

        try {
            mBinder = binder;
            binder.linkToDeath(SocketServiceManager.this, 0);
            mSocketService = ISocketService.Stub.asInterface(binder);
            registerCallback();
            mCallback.onServiceConnected();
        } catch (RemoteException e) {
            LogUtils.e(TAG, "onServiceConnected: " + e.getMessage());
        }
    }
    
    @Override
    public void onServiceDisconnected(ComponentName name) {
        unregisterCallback();
        mCallback.onServiceDisconnected();

        if (null != mBinder) {
            mBinder.unlinkToDeath(this, 0);
            mBinder = null;
        }
    }
...
```

```Java
private class SocketServiceManager implements IBinder.DeathRecipient {

    ...

        @Override
        public void binderDied() {
            mCallbackList.unregister(mClientCallBack);
            mClientCallBack = null;
            Logger.d(TAG,"client  is died");
        }
    
    ...
}
```
上面是在 Server 端对 Client 的回调接口的 Binder 对象设置的 DeathRecipient。在 Client 死亡时,解注册 Client 的回调，并且置空。

### Client 注册回调接口
之前一直说的都是 Client 向 Server 的通信，那如果 Server 要调用 Client 呢？
一个比较容易想到的办法就是通过 AIDL 在 Server 端设置一个 Client 的回调。这样的话就相当于 Client 端是 Server 端的 Server 了。
有注册回调就肯定有解注册，但是 Client 端与 Server 不在一个进程，Server 是无法得知 Client 解注册时传入的回调接口是哪一个（ Client 调用解注册时，是通过 Binder 传输到 Server 端，所以解注册时的回调接口是新创建的，而不是注册时的回调接口）。为了解决这个问题，Android 提供了 RemoteCallbackList 这个类来专门管理 remote 回调的注册与解注册。

AIDL 类
```Java
interface ITaskCallback {   
    void actionPerformed(int actionId);  
} 
```

```Java
interface ITaskBinder {   
    boolean isTaskRunning();   
    void stopRunningTask();   
    void registerCallback(ITaskCallback cb);   
    void unregisterCallback(ITaskCallback cb);   
}
```

Service 类
```Java
public class MyService extends Service {   
    private static final String TAG = "aidltest";  
    final RemoteCallbackList <ITaskCallback>mCallbacks = new RemoteCallbackList <ITaskCallback>(); 
    ...

    @Override  
    public IBinder onBind(Intent t) {  
        printf("service on bind");  
        return mBinder;   
    }  

    @Override  
    public boolean onUnbind(Intent intent) {   
        printf("service on unbind");  
        return super.onUnbind(intent);   
    }  

    void callback(int val) {   
        final int N = mCallbacks.beginBroadcast();  
        for (int i=0; i<N; i++) {   
            try {  
                mCallbacks.getBroadcastItem(i).actionPerformed(val);   
            }  
            catch (RemoteException e) {   
                // The RemoteCallbackList will take care of removing   
                // the dead object for us.     
            }  
        }  
        mCallbacks.finishBroadcast();  
    }  

    private final ITaskBinder.Stub mBinder = new ITaskBinder.Stub() {  

        public void stopRunningTask() {  

        }  

        public boolean isTaskRunning() {   
            return false;   
        }   

        public void registerCallback(ITaskCallback cb) {   
            if (cb != null) {   
                mCallbacks.register(cb);  
            }  
        }  

        public void unregisterCallback(ITaskCallback cb) {  
            if(cb != null) {  
                mCallbacks.unregister(cb);  
            }  
        }  
    };   

}
```
Client 类
```Java
public class MyActivity extends Activity {   

    private static final String TAG = "aidltest";  
    private Button btnOk;   
    private Button btnCancel;  

...

    ITaskBinder mService;   

    private ServiceConnection mConnection = new ServiceConnection() {   

        public void onServiceConnected(ComponentName className, IBinder service) {  
            mService = ITaskBinder.Stub.asInterface(service);   
            try {   
                mService.registerCallback(mCallback);  
            } catch (RemoteException e) {  

            }  
        }  

        public void onServiceDisconnected(ComponentName className) {   
            mService = null;  
        }   
    };   

    private ITaskCallback mCallback = new ITaskCallback.Stub() {  

        public void actionPerformed(int id) {   
            printf("callback id=" + id);  
        }   
    };   

}
```
**RemoteCallbackList 可以实现正常注册于解注册的原因在于注册与解注册时虽然对应的回调接口不是同一个,但是其对应的 Binder 对象却是同一个。**

## Messenger 通信
以下概括了Messenger的使用方法:
1. 服务实现一个 `Handler` ，用于客户端每次调用时接收回调
2. 此 `Handler` 用于创建一个 `Messenger` 对象（它是一个对 `Handler` 的引用)
3. 此 `Messenger` 对象创建一个 `IBinder` ，服务在 onBind() 中把它返回给客户端
4. 客户端用 `IBinder` 将 `Messenger`（引用服务的 `Handler`）实例化，客户端用它向服务发送消息对象 `Message`
5. 服务接收 `Handler` 中的每个消息 `Message` ——确切的说，是在 handleMessage() 方法中接收

Service 类
```Java
public class MessengerService extends Service {

/**
     * Target we publish for clients to send messages to IncomingHandler.
     */
    final Messenger mMessenger = new Messenger(new IncomingHandler());

    /**
     * When binding to the service, we return an interface to our messenger
     * for sending messages to the service.
     */
    @Override
    public IBinder onBind(Intent intent) {
        return mMessenger.getBinder();
    }
    
    class IncomingHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_REGISTER_CLIENT:
                    mClients.add(msg.replyTo);
                    break;
                case MSG_UNREGISTER_CLIENT:
                    mClients.remove(msg.replyTo);
                    break;
                case MSG_SET_VALUE:
                    mValue = msg.arg1;
                    for (int i=mClients.size()-1; i>=0; i--) {
                        try {
                            mClients.get(i).send(Message.obtain(null,
                                    MSG_SET_VALUE, mValue, 0));
                        } catch (RemoteException e) {
                            // The client is dead.  Remove it from the list;
                            // we are going through the list from back to front
                            // so this is safe to do inside the loop.
                            mClients.remove(i);
                        }
                    }
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    }
    
    ...
}
```
Client 类
```Java
public static class Binding extends Activity {

    /**
     * Messenger for communicating with service.
     */
    Messenger mService = null;

    /**
     * Handler of incoming messages from service.
     */
    class IncomingHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MessengerService.MSG_SET_VALUE:
                    mCallbackText.setText("Received from service: " + msg.arg1);
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    }

    /**
     * Target we publish for clients to send messages to IncomingHandler.
     */
    final Messenger mMessenger = new Messenger(new IncomingHandler());

    /**
     * Class for interacting with the main interface of the service.
     */
    private ServiceConnection mConnection = new ServiceConnection() {
        public void onServiceConnected(ComponentName className,
                                       IBinder service) {
            // This is called when the connection with the service has been
            // established, giving us the service object we can use to
            // interact with the service.  We are communicating with our
            // service through an IDL interface, so get a client-side
            // representation of that from the raw service object.
            mService = new Messenger(service);
            mCallbackText.setText("Attached.");
            // We want to monitor the service for as long as we are
            // connected to it.
            try {
                Message msg = Message.obtain(null,
                        MessengerService.MSG_REGISTER_CLIENT);
                msg.replyTo = mMessenger;
                mService.send(msg);

                // Give it some value as an example.
                msg = Message.obtain(null,
                        MessengerService.MSG_SET_VALUE, this.hashCode(), 0);
                mService.send(msg);
            } catch (RemoteException e) {
                // In this case the service has crashed before we could even
                // do anything with it; we can count on soon being
                // disconnected (and then reconnected if it can be restarted)
                // so there is no need to do anything here.
            }

            // As part of the sample, tell the user what happened.
            Toast.makeText(Binding.this, R.string.remote_service_connected,
                    Toast.LENGTH_SHORT).show();
        }

        public void onServiceDisconnected(ComponentName className) {
            // This is called when the connection with the service has been
            // unexpectedly disconnected -- that is, its process crashed.
            mService = null;
            mCallbackText.setText("Disconnected.");
            // As part of the sample, tell the user what happened.
            Toast.makeText(Binding.this, R.string.remote_service_disconnected,
                    Toast.LENGTH_SHORT).show();
        }
    };
    
    ...
}
```
在 [MessengerService.java](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/app/MessengerService.java)（服务）和 [MessengerServiceActivities.java](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/app/MessengerServiceActivities.java)（客户端）例程中，可以看到如何关于 Messenger 的实用例子。

## Messenger 和 AIDL 的异同
其实 Messenger 的底层也是用 AIDL 实现的，但用起来还是有些不同的，这里总结了几点区别：
1. **Messenger 本质也是 AIDL，只是进行了封装，开发的时候不用再写 .aidl 文件**

    结合自身的使用，因为不用去写 .aidl 文件，相比起来，Messenger 使用起来十分简单。但前面也说了，Messenger 本质上也是 AIDL，故在底层进程间通信这一块，两者的效率应该是一样的。
2. **在 Service 端，Messenger 处理 Client 端的请求是单线程的，而 AIDL 是多线程的**
    
    使用 AIDL 的时候，service 端每收到一个 client 端的请求时，就在 Binder 线程池中取一个线程去执行相应的操作。而 Messenger ，service 收到的请求是放在 `Handler` 的 `MessageQueue` 里面，Handler 大家都用过，它需要绑定一个 Thread，然后不断 poll message 执行相关操作，这个过程是同步执行的。
3. **Client 的方法，使用 AIDL 获取返回值是同步的，而 Messenger 是异步的**

    Messenger 只提供了一个方法进行进程间通信，就是 send(Message msg) 方法，发送的是一个 Message，没有返回值，要拿到返回值，需要把 Client 的 Messenger 作为 `msg.replyTo` 参数传递过去，Service 端处理完之后，在调用客户端的 Messenger 的 send(Message msg) 方法把返回值传递回 Client，这个过程是异步的，而 AIDL 你可以自己指定方法，指定返回值，它获取返回值是同步的（如果没有用 oneway 修饰方法的话）。

P.S. 该文章还配有对应的 PPT， 请[点击这里](https://speakerdeck.com/zhgqthomas/android-interprocess-commuincation)

<script async class="speakerdeck-embed" data-id="43c803edf21b48a9a885ba21295b1b11" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>