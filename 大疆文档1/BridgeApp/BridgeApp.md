github: https://github.com/dji-sdk/Android-Bridge-App
>在注册成功后，或者其他地方添加以下语句来连接BridgeApp
```java
// ADD CALL HERE, Bridge App
DJISDKManager.getInstance().enableBridgeModeWithBridgeAppIP("BrideApp IP");
```