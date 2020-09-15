## java客户端
```xml
<!--paho.mqtt:由独立的非营利性公司 Eclipse Foundation 开发-->
<!-- https://github.com/eclipse/paho.mqtt.java -->
<dependency>
    <groupId>org.eclipse.paho</groupId>
    <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
    <version>1.2.0</version>
</dependency>
<!--mqtt-client:由 Red Hat 子公司 FuseSource 开发-->
<!-- https://github.com/fusesource/mqtt-client -->
<dependency>
    <groupId>org.fusesource.mqtt-client</groupId>
    <artifactId>mqtt-client</artifactId>
    <version>1.14</version>
</dependency>
```
## 服务端EMQ
### 安装和启动
https://www.emqx.io/
下载windows版本解压，进入目录，打开powershell,输入`.\bin\emqx.cmd start`,然后进入http://127.0.0.1:18083，用户名为admin，密码为public
控制台模式启动：`bin\emqx console`
### 对 EMQ X 进行简单的配置
>所有对 EMQ X 的配置都可以通过修改配置文件完成。配置文件的位置：
etc/emqx.conf : EMQ X 服务器的参数设置
etc/plugins/*.conf : EMQ X 插件配置文件，每个插件都有单独的配置文件。
一些常用功能的配置也在 Web Dashboard 上进行修改。

>配置文件etc/plugins/emqx_dashboard.conf可以修改默认的用户名和密码:
```java
dashboard.default_user.login = admin
dashboard.default_user.password = public
```
### 配置端口
> 在安装以后，EMQ X 默认会使用以下端口：
* 1883: MQTT 协议端口
* 8883: MQTT/SSL 端口
* 8083: MQTT/WebSocket 端口
* 8080: HTTP API 端口
* 18083: Dashboard 管理控制台端口
>修改配置:
* 配置文件'etc/emqx.conf'修改tcp/ssl/ws的端口
* 配置文件'etc/plugins/emqx_management.conf'修改http端口
* 配置文件'etc/plugins/emqx_dashboard.conf'修改dashboard端口
### 启动/停止插件
>命令行工具 emqx_ctl 来启动和停止各个插件:
```
bin/emqx_ctl plugins load plugin_name
bin/emqx_ctl plugins unload plugin_name
```
>也可以在 Dashboard 的 MANAGEMENT -> plugins 菜单下启动和停止插件，或对插件进行简单的配置。
* 注意:EMQ X 的 Dashboard 本身也是一个插件，如果您在 Web 界面下停止了 Dashboard 插件，您将无法再使用 dashboard，直至您使用命令行工具再次启动 Dashboard。
### 修改 Erlang 虚拟机启动参数
>EMQ X 运行在 Erlang 虚拟机上，在'etc/emqx.conf'中有两个限定了虚拟机允许的最大连接数。在运行 EMQ X 前可以修改这两个参数以适配连接需求：
* node.process_limit : Erlang 虚拟机允许的最大进程数，EMQ X 一个连接会消耗 2 个 Erlang 进程;
* node.max_ports : Erlang 虚拟机允许的最大 Port 数量，EMQ X 一个连接消耗 1 个 Port
* 在 Erlang 虚拟机中的 Port 概念并不是 TCP 端口，可以近似的理解为文件句柄。*
>这两个参数可以设置为：
* node.process_limit： 大于最大允许连接数 * 2
* node.max_ports： 大于最大允许连接数
-----------
## MQTT协议
> MQTT 协议使用发布 / 订阅消息范式来做到一对多的消息分发以及应用程序的解耦

> MQTT 协议提供了 3 种（QoS）服务质量用于消息传输，适应不同的物联网数据传输场景
1. QoS 0：最多一次传送 (只负责传送，发送过后就不管数据的传送情况)
2. QoS 1：至少一次传送 (确认数据交付)
3. QoS 2：正好一次传送 (保证数据交付成功)
> 通过很小的传输开销，以及最小化的协议交换来减少网络流量

> 发生异常断线时通知各方的机制
## 发布/订阅
>发布者：温度传感器，通过主题 "sensor/1/temperature" 发布一条温度为 37.5 度的消息

>3 个自上而下的订阅者分别通过订阅不同的主题得到发布者发出的消息
1. 移动设备：通过订阅主题 "sensor/1/#"
2. 电脑：通过订阅主题 "sensor/+/temperature"
3. 服务器：通过订阅主题 "sensor/1/temperature"

## MQTT服务器
>MQTT 服务器是发布者和订阅者之间通信的代理（因此中文也有将 MQTT 服务器翻译为 MQTT 代理），主要提供了以下的功能，
1. 基于主题的 Pub/Sub 模式，将发布者和订阅着解耦
2. 对于服务器来说，发布者和订阅者都是“客户端”
3. 客户端与服务器连接都通过 TCP、TLS 或者 WebSocket
4. 客户端（发布者）发送一条消息到服务器
5. 一个或者多个客户端（订阅者）从服务器接收消息
>按照 MQTT 协议标准，服务器提供三种连接方式，
1. TCP：默认端口为 1883
2. TLS：默认端口为 8883
3. WebSocket：默认端口为 8083
## MQTT 在线测试工具
http://tools.emqx.io/connection/create
>MQTT Web Toolkit 是 EMQ 最近开源的一款 MQTT (WebSocket) 测试工具，支持在线访问使用。该工具采用了聊天界面形式，简化了页面操作逻辑，方便用户快速测试验证 MQTT 应用场景：
1. 支持通过普通或者加密的 WebSocket 端口连接至 MQTT 消息服务器；
2. 链接的新建、编辑、删除以及缓存链接方便下次访问使用；
3. 不同链接的订阅列表管理；
4. 消息发布、接收、以及接收到新消息时提示，同时支持按照消息类型过滤消息列表。
## Java Demo
>使用paho.mqtt客户端
```java
import org.eclipse.paho.client.mqttv3.IMqttDeliveryToken;
import org.eclipse.paho.client.mqttv3.MqttCallback;
import org.eclipse.paho.client.mqttv3.MqttClient;
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.eclipse.paho.client.mqttv3.MqttException;
import org.eclipse.paho.client.mqttv3.MqttMessage;
import org.eclipse.paho.client.mqttv3.persist.MemoryPersistence;

public class Demo {
    public static void main(String[] args) {
        String broker = "tcp://localhost:1883";
        String clientId = "JavaSample";
        //Use the memory persistence
        MemoryPersistence persistence = new MemoryPersistence();

        try {
            MqttClient sampleClient = new MqttClient(broker, clientId, persistence);
            MqttConnectOptions connOpts = new MqttConnectOptions();
            connOpts.setCleanSession(true);
            System.out.println("Connecting to broker:" + broker);
            sampleClient.connect(connOpts);
            System.out.println("Connected");

            String topic = "demo/topics";
            System.out.println("Subscribe to topic:" + topic);
            sampleClient.subscribe(topic);
            sampleClient.setCallback(new MqttCallback() {
                @Override
                public void messageArrived(String topic, MqttMessage message) throws Exception {
                    String theMsg = MessageFormat.format("{0} is arrived for topic {1}.", new String(message.getPayload()), topic);
                    System.out.println(theMsg);
                }

                @Override
                public void deliveryComplete(IMqttDeliveryToken token) {
                }

                @Override
                public void connectionLost(Throwable throwable) {
                }
            });


            String content = "Message from MqttPublishSample";
            int qos = 2;
            System.out.println("Publishing message:" + content);
            MqttMessage message = new MqttMessage(content.getBytes());
            message.setQos(qos);
            sampleClient.publish(topic, message);
            System.out.println("Message published");

        } catch (MqttException me) {
            System.out.println("reason" + me.getReasonCode());
            System.out.println("msg" + me.getMessage());
            System.out.println("loc" + me.getLocalizedMessage());
            System.out.println("cause" + me.getCause());
            System.out.println("excep" + me);
            me.printStackTrace();
        }
    }
}
```



