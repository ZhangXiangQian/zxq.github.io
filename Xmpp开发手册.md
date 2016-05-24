### Xmpp开发手册



**Xmpp**是基于Xmpp协议而编写的移动端即时通信SDK。目前支持的功能有：
 
- **连接状况反馈** ：实时监控通讯连接状况，在连接失败的情况下支持自动重新连接，并将连接状况及时反馈出去；
- **收发消息** ：支持用户收发消息（包括离线消息）；


-------------------




## 工程配置

### android端
   android支持2.3以上系统，开发工具目前仅支持android studio，工程配置分为两种情况：
   1、arr文件存放在本地，将arr文件复制到libs文件夹下，并在对应的module下得build.gradle文件夹下加入下面代码：
   

    repository {
    flatDir
            {
                dirs 'libs'
            }
    }

  2、arr存放在服务器并使用内网连接，加入下面的配置信息：
    工程下的build.gradle文件下 jcenter()前面加入下面一行代码：
 
     maven { 
     url 'http://10.7.7.100:8081/nexus/content/groups/public/' }
  module下的build.gradle文件dependencies下加入下面代码
  

     compile 'com.bankeys.library:xmpp:0.1.0@aar'

###IOS端

##方法说明
###参数配置
####XMPPHelper类
 这个类负责XMPP的参数配置，在启动服务之前调用；
  
   

    XMPPHelper x_helper = XMPPHelper.getConfig()

####通讯连接状况的监听
     
     x_helper.setConnectChange(new ConnectChange() {
                @Override
                public void onConnectChange(XmppConnectStatus status) {
                    
                }
            });
####接收消息

     x_helper.setPacketListener(new PacketListener() {
            @Override
            public void processPacket(Packet packet) {
                
            }
        });
        
####消息过滤
  在包中封装了十种左右的适配器，不过目前我们仅使用PacketTypeFilter，如果此项为null，则所有消息在PacketListener里获取

    public class ChatMessageFilter extends PacketTypeFilter {
    private Context mContext;
    /**
     * Creates a new packet type filter that will filter for packets that are the
     * same type as <tt>packetType</tt>.
     *
     * @param packetType the Class type.
     */
    public ChatMessageFilter(Context mContext,Class<? extends Packet> packetType) {
        super(packetType);
        this.mContext = mContext;
    }

    @Override
    public boolean accept(Packet packet) {
        LogUtil.i("接收到消息...XML报文：" + packet.toXML());
        Message msg = (Message) packet;
        String content = msg.getBody();
        return super.accept(packet);
    }
}

     x_helper.setPacketTypeFilter(new ChatMessageFilter(context, Message.class))

###相关函数
相关函数是以静态方式存放在ConnecMethod类，这部分都是耗时操作，在调用时要放在异步线程里执行
####登录
方法名：login；
参数说明：

| 参数名     |     是否必须  |   说明    |
| :-------- | :--------:  | :------: |
| account   | 是         |  用户名|
|  password | 是         |     用户密码     |


     try {
           ConnecMethod.login(account, password);
         } catch (ChatException e) {
           e.printStackTrace();
     }
####发送消息
方法名：sendMessageEventRequest
参数说明：

| 参数     |     是否必须 |   说明    |
| :-------- | :--------:| :------: |
| userId    |   是      |  用户ID   |
| roomId    |   是      | 房间ID    |
| sendMsg   |   是      | 消息内容   |
示例代码：

        try {
               ConnecMethod.sendMessageEventRequest( userId, roomId, sendMsg);
          } catch (ChatException e) {
                e.printStackTrace();
          }
      
####其它
#####XmppConnectStatus 连接状态

| 名称             |     说明 | 
| :--------       | :--------:    |
| RemoteLoginErr  |   异地登录     |  
|ConnectTimeOutErr|   连接超时     |
|SystemShutdownErr|   服务器关闭   |
|Reconnecting     |  正在重新连接  |
|Failure          |   连接失败    |
|Error            |   其它错误    |
|Close            |    关闭       |
#####Message.Type 消息类型

| 类型        |     说明  |  
| :--------: | :--------:| 
| normal     |   普通消息 |  
|chat        | 聊天消息   |
| groupchat  | 群聊消息   |
|headline    |           |
|QR          | 二维码     |
|order       | 订单消息   |
|sign        | 签名信息   |
|error       | 错误信息   |



