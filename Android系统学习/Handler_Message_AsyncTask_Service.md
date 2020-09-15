
# 子线程中更新UI
>直接使用子线程来更新UI，会导致程序崩溃。
## 异步消息处理机制
### 1. Message
>在线程之间传递的消息。携带的消息除了what字段，还有整形arg1和arg2，Object类型的obj。
### 2. Handler
>sendMessage()和handleMessage方法。
### 3. MessageQueue
>Message存放在消息队列中，每个线程只会有一个MessageQueue对象（1：1）。
### 4. Looper
>MessageQueue对象的管家（1：1），Looper.loop()后，每当队列中有消息，就取出传递到handleMessage方法中。

**也可以借助getActivity().runOnUiThread() 方法来实现从子线程切换到主线程，其实就是异步消息处理机制的接口封装**

```java
//主线程中定义一个Handler
private Handler handler = new Handler() {
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case UPDATE_TEXT:
                // 在这里可以进行UI操作
                text.setText("Nice to meet you");
                break;
            default:
            break;
        }
    }
};  

//新开一个线程使用Handler来更新UI
new Thread(new Runnable() {
    @Override
    public void run() {
        Message message = new Message();
        message.what = UPDATE_TEXT;
        // 将Message对象发送出去，使其在主线程中更新UI
        handler.sendMessage(message); 
    }
}).start();
```  
# AsyncTask
```java
class DownloadTask extends AsyncTask<Void, Integer, Boolean> {
    @Override
    protected void onPreExecute() {
        progressDialog.show(); // 显示进度对话框
    }
    @Override
    protected Boolean doInBackground(Void... params) {
        try {
            while (true) {
                int downloadPercent = doDownload(); // 这是一个虚构的方法
                publishProgress(downloadPercent);
                if (downloadPercent >= 100) {
                    break;
                }
            }
        } catch (Exception e) {
            return false;
        }
        return true;
    }
    @Override
    protected void onProgressUpdate(Integer... values) {
        // 在这里更新下载进度
        progressDialog.setMessage("Downloaded " + values[0] + "%");
    }
    @Override
    protected void onPostExecute(Boolean result) {
        progressDialog.dismiss(); // 关闭进度对话框
        // 在这里提示下载结果
        if (result) {
            Toast.makeText(context, "Download succeeded", 
            Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(context, " Download failed", 
            Toast.LENGTH_SHORT).show();
        }
    }
}
```
## 3个泛型参数
### 第一个参数Param1对应doInBackground的传入参数类型
>作为执行AsyncTask时需要传入的参数类型。
### 第二个参数Param2对应onProgressUpdate的传入参数类型
>后台任务执行时，作为界面上显示当前进度的单位。
### 第三个参数Param3对应doInBackground的返回参数类型，以及对应onPostExecute的传入参数类型
>后台任务执行完毕后，作为结果的返回类型。
## 4个重写的方法
### onPreExecute()
>在后台任务开始之前调用，用于进行界面上的初始化操作（比如进度条对话框）。
### Param3 doInBackGroud(Param1 ...)
>该方法中的代码都在**子线程**中运行，该方法中不能进行UI更新操作（不能再子线程中更新UI），若需要更新UI元素，如反馈当前任务进度，可以调用`publishProgress(Param2)`，接着，`onProgressUpdate(Param2)`会很快被调用。
### onProgressUpdate(Param2)
>可以对UI进行操作，利用参数的数值来更新界面元素。
### onPostExecute(Param3) 
>后台任务`doInBackground`执行完毕后,这个方法很快被调用。可以使用返回的数据进行一些UI操作，比如提醒任务执行的结果、关闭进度条对话框等。
# 服务
## 服务的创建
>继承抽象类`android.app.Service`，重写`onBind()`，`onCreate()`（服务第一次创建时调用），`onStartCommand()`（每次服务启动的时候调用），`onDestory()`等方法。必须在Manifest中注册:
```xml
<application
...>
    ...
    <service
        android:name=".Myservice"
        android:enabled="true"
        android:exported="true">
    </service>
    ...
</application>
```
## 启动和停止服务
>由活动来启动和停止服务：
```java
Intent startIntent = new Intent(this, Myservice.class);
startService(startIntent);
Intent stopIntent = new Intent(this, MyService.class);
stopService(stopIntent);
```
>在服务内部停止服务：在MyService中调用`stopSelf()`方法。

>服务启动后，可以在`Settings→Developer options→Running services`中找到它。
## 活动与服务通信
>利用`public IBinder onBind(Intent intent)`方法，注意该方法返回的类型IBinder。

>在实际的应用中，希望MyService类提供下载功能，在活动中可以决定何时开始下载，以及随时查看下载进度等。实现该功能的方法是创建一个Binder对象对下载功能进行管理。在MyService类中添加成员变量`private DownloadBinder mBinder = new DownloadBinder();`，并且在`onBind`方法中返回`mBinder`。


>`DownloadBinder`类继承`android.os.Binder`类型，提供开始下载`startDownload`和查看下载进度`getProgress`的方法。
```java
public class MyService extends Service {
    private DownloadBinder mBinder = new DownloadBinder();
    class DownloadBinder extends Binder {
        public void startDownload() {
            Log.d("MyService", "startDownload executed");
        }
        public int getProgress() {
            Log.d("MyService", "getProgress executed");
            return 0;
        }
    }
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
    ...
}
```
>绑定和解绑服务
```java
//活动类MainActivity中
private MyService.DownloadBinder downloadBinder;
private ServiceConnection connection = new ServiceConnection() {
    //活动与服务的连接断开时调用
    @Override
    public void onServiceDisconnected(ComponentName name) {}
    //活动与服务成功绑定时调用
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {   
        //获取DownloadBinder实例,由onBind()方法返回downloadBinder实例
        downloadBinder = (MyService.DownloadBinder) service;
        downloadBinder.startDownload();
        downloadBinder.getProgress();
    }
};
    ...
    //将MainActivity和MyService进行绑定
    Intent bindIntent = new Intent(this, MyService.class);
    //传入BIND_AUTO_CREATE 表示在活动和服务进行绑定后自动创建服务。这会使得MyService中的onCreate() 方法得到执行，但onStartCommand() 方法不会执行。
    //会回调服务中的onBind()方法来返回downloadBinder实例
    bindService(bindIntent, connection, BIND_AUTO_CREATE);// 绑定服务
    ...
    unbindService(connection); // 解绑服务
    ...
```
>服务在整个应用程序范围内都可用，可以和任何一个活动绑定，绑定后都可以获**取相同的```DownloadBinder```实例**（服务是单例的）。
## 服务的生命周期
>调用Context的`startService()`方法，相应的服务就会启动启动起来，并回调`onStartCommand()`。如果服务之前还没创建过，`onCreate()`会先于`onStartCommand()`执行。服务启动之后会一直保持运行状态，直到**Context**的`stopService()`或**Service**内部的`stopSelf()`调用。注意，服务时单例的，故不论调用多少次startService()，每个服务都只存在一个实例。

>调用Context的`bindService()`，会回调Servicez中的`onBind()`方法，返回Binder对象，若该服务也没有创建国，`onCreate()`会先于`onBind()`执行。只要调用方和服务的连接没有断开，服务就会一直保持运行状态。

>当调用了`startService()`方法后，又去调用`stopService()`方法，这时服务中的
`onDestroy()`方法就会执行，表示服务已经销毁了。当调用了`bindService()`方
法后，又去调用`unbindService()`方法，`onDestroy()`方法也会执行。当一个服务既调用了`startService()`方法，又调用了`bindService()`方法，要同时调用`stopService()`和`unbindService()`方法，`onDestroy()`方法才会执行。
## 服务的更多技巧
###前台服务
>普通服务的系统优先级比较低，在内存不足时，可能被系统回收。**前台服务**可以让服务一直保持运行状态，不被系统以系统不足的原因回收，一直有一个正在运行的图标在系统的状态栏显示，下拉状态栏后可以看到更加详细的信息，非常类似于通知的效果。
```java
public class MyService extends Service {
    ...
    @Override
    public void onCreate() {
        super.onCreate();
        Log.d(TAG, "onCreate: 创建服务");

        //创建通知的方法
        Intent intent = new Intent(this, MainActivity.class);
        PendingIntent pi = PendingIntent.getActivity(this, 0, intent, 0);
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this, "channellId")
                .setContentTitle("这是标题")
                .setContentText("这是内容")
                .setWhen(System.currentTimeMillis())
                .setSmallIcon(R.mipmap.ic_launcher)
                .setLargeIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher))
                .setContentIntent(pi);
        // 大于Android 8.0的版本适配
        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.O) {
            NotificationChannel notificationChannel = null;
            NotificationManager notificationManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
            // 注意此处channellId要和前面一样
            notificationChannel = new NotificationChannel("channellId", "name", NotificationManager.IMPORTANCE_HIGH);
            notificationManager.createNotificationChannel(notificationChannel);
        }
        //build()创建了Notification 对象后并没有使用NotificationManager来将通知显示出来，而是调用了startForeground() 方法。
        startForeground(1, builder.build());
    }
    ...
}
```
>注意，需要在Manifest.xml中添加权限：
```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
```
### IntentService
>服务的代码都是默认运行在主线程中的。容易出现ANR(Application Not Responding)的情况。

>一般都在服务中创建子线程去处理耗时的逻辑，服务一旦启动，就一直处于运行状态，必须调用Context的stopService()或者Service本身的stopSelf()来停止服务。
```java
public class MyService extends Service {
    ...
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                // 处理具体的逻辑
                ...
                // 让服务在执行完毕后自动停止
                stopSelf();
            }
        }).start();
        return super.onStartCommand(intent, flags, startId);
    }
    ...
}
```
>为了防止忘记开线程或者忘记调用stopSelf方法，可以简单地创建一个异步的、会自动停止的服务--IntentService类。
```java
public class MyIntentService extends IntentService {
    private static final String TAG = "MyIntentService";

    public MyIntentService() {
        super("MyIntentService");
    }

    @Override
    protected void onHandleIntent(@Nullable Intent intent) {
        Log.d(TAG, "onHandleIntent: Thread id is" + Thread.currentThread().getId());
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy: ");
    }
}

//在MainActivity中开启服务
// 打印主线程的id
    Log.d("MainActivity", "Thread id is " + Thread.currentThread().getId());
    Intent intentService = new Intent(this, MyIntentService.class);
    startService(intentService);
```
>注意在Manifest中注册服务
```xml
<application
    ...>
    ...
    <service android:name=".MyIntentService" />
    ...
</application>
```
