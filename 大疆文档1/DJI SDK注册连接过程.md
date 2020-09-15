# DJI SDK注册连接过程
1. 检查和请求权限
```java
private void checkAndRequestPermissions()
    ...
    ActivityCompat.requestPermissions(this,
                        missingPermission.toArray(new String[missingPermission.size()]),
                        REQUEST_PERMISSION_CODE);
    ...
}

@Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        ...
        // If there is enough permission, we will start the registration
        if (missingPermission.isEmpty()) {
            //权限验证都通过之后，开始SDK的注册
            startSDKRegistration();
        } else {
            showToast("Missing permissions!!!");
        }
    
    }
```
2. 注册SDK
```java
private void startSDKRegistration() {
    AsyncTask.execute(() -> {
        //注册App
        DJISDKManager.getInstance().registerApp(getApplicationContext(), new DJISDKManager.SDKManagerCallback() {
            /**
            * Callback method after the application attempts to register.
            * @param djiError REGISTRATION_SUCCESS if registration is successful.
            */
           @Override
            public void onRegister(DJIError djiError) {
                //如果注册成功
                if (djiError == DJISDKError.REGISTRATION_SUCCESS) {
                    //开启SDK和Product之间的连接，如果连接成功，则返回true，同时onProductConnect()方法会被调用
                    DJISDKManager.getInstance().startConnectionToProduct();
                } else {
                    ...
                }
                ...
            } 
            @Override
            public void onProductDisconnect() {...}
            @Override
            public void onProductConnect(BaseProduct baseProduct) {...}
            /**
            * Called when a component object changes
            * @param componentKey An enum value of the ComponentKey
            * @param oldComponent An instance of BaseComponent
            * @param newComponent An instance of BaseComponent
            */
            @Override
            public void onComponentChange(BaseProduct.ComponentKey componentKey, BaseComponent oldComponent,
                                            BaseComponent newComponent) {...}
            /**
            * Called when load sdk resource.
            * @param djisdkInitEvent An event that show what resource is loading.
            * @param i The totalProcess of load resource, from 0 to 100.
            */
            @Override
            public void onInitProcess(DJISDKInitEvent djisdkInitEvent, int i) {}
            /**
            * Called when Fly Safe database download progress is updated.
            * If you excluded "fly-safe-database" in build.gradle, mobile SDK will download the database
            * when registerApp is invoked, otherwise it will not be invoked.
            * Please find onRegister for updated error.
            * @param l The current download data size.
            * @param l1 The total size of the file being downloaded.
            */
            @Override
            public void onDatabaseDownloadProgress(long l, long l1) {}
        });
    } 
} 
                              
                
