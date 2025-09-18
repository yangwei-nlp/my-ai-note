# WebSocket接口

用户向虚拟人发送任何消息时都只有一个长链接和一个会话id（如下的`session_id`），包括文本聊天、语音通话和视频通话，具体是通过信令（如下的`call_type`）来指定。

## 1、地址

WebSocket 连接地址：

> ws://121.36.17.169:7860/api/algo/platform/ws/friendo?session\_id="conv\_1001"

连接参数（Client → Server）：

```json
{
  "session_id": "conv_1001"
}
```

* **`session_id`**：会话id；不同用户、虚拟人下，会话id不同。（同一组用户和虚拟人但不同设备，会话id相同）



## 2、打断策略

不管是文本聊天、语音聊天或视频聊天，总共分为三种场景（下面以语音聊天举例）：

1、后台检测到用户连续说了两（N）句话，且算法没回复。后台应该分别发送两条消息到算法端，算法端同时处理两条消息后回复一句。

![]()

2、后台发送一条消息到算法端，算法端在回复的同时后台又发送一条消息到算法端，此时算法应该及时中止（end\_flag=True），并重新生成新的回复。

![]()

3、后台发送一条消息到算法端，算法端回复完毕后，后台发送另一条消息到算法端，按正常逻辑处理即可。

![]()







## 3、数据交互

server->client 使用这个基础结构包装核心数据结构

```json
{
  "code": "",
  "message": "",
  "data": {
    
  }
}
```



client->server直接使用核心数据结构输入

核心数据结构

```json
{
  "message_type": "",   
  "cmd_msg":{
    "cmd_code":"",
    "cmd_payload":{
      
    }
  
  },
  "chat_msg": {
      "msg_id":"",
      "scene":"",
      "type": "",
      "data": {
      
      }
  }
}
```

参数介绍如下：

* message\_type，不能为 null,取值如下

  * COMMAND ：表示控制信令消息

  * NORMAL：表示正常的消息交互

* cmd\_msg: 当 message\_type 为`COMMAND`时候有值；

  * cmd\_code 控制信令的唯一标识，取值如下：

    client端使用：

    * `chat->voice_call ` 表示从聊天切换为音频通话

    * `chat->video_call ` 表示从聊天切换为视频通话

    * `voice_call->chat` 表示从音频通话切换为聊天

    * `video_call->chat` 表示从视频通话切换为聊天

    server端使用：

    * `call_failed`表示无法接通

    * `chat_failed`表示算法回复失败

  * cmd\_payload:

    一、如果信令需要携带数据，则通过此字段进行传输(client->server)，为一个对象类型

    * 当cmd\_code 为`chat->voice_call ` 和`chat->video_call` 时候字段如下

      * room\_id  :房间 id

      * push\_url :推流地址

    * 当cmd\_code 为`voice_call->chat` 和`video_call->chat`

      * roomId  :房间 id

    二、server端内部无法正常响应，也会通过cmd\_payload传输数据(server->client)

    * 当cmd\_code为`call_failed`时，cmd\_payload如下：

      * room\_id  :房间 id

      * push\_url :推流地址（如果有的话）

    * 当cmd\_code为`chat_failed`时，cmd\_payload如下：

      也即当大模型回复文本失败时，需要**原封不动**将chat\_msg传到cmd\_payload

      * msg\_id

      * scene

      * type

      * data

        * content\_raw

        * content

        * room\_id(如果有)

* chat\_msg: 当 message\_type 为`NORMAL`时候有值：

  * msg\_id：消息id

  * scene: 场景，取值如下

    * chat: 聊天场景

    * voice\_call：音频电话

    * video\_call：视频电话

  * type: 类型，取值如下

    * text:文本

    * image:图片

  * data: 消息内容

    * 当 type 取值为 text 时候

      * content\_raw：算法端的回复（含情绪标签）

      * content：用户看到的文本（无情绪标签）

    * 当 type 取值为 image 时候

      * height：高

      * width：宽

      * data: 图片的base64内容

      * prompt: 生成图片的完整提示词，比如：\<img>xxx\</img>

    * 当scene=voice\_call 或 video\_call的时候，(client->server)还会传：

      * room\_id

