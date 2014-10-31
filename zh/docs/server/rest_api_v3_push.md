# Push API v3

    这是 Push API 最新的版本。
    相比 API v2 版本，v3 版本的改进为：
    * 完全基于 https，不再提供 http 访问；
    * 使用 HTTP  Basic Authentication 的方式做访问授权。这样整个 API 请求可以使用常见的 HTTP 工具来完成，比如：curl, 浏览器插件等；
    * 推送内容完全使用 JSON 的格式；
    * 支持的功能有所改进：支持多 tag 的与或操作；可单独发送通知或自定义消息，也可同时推送通知与自定义消息; windows phone 目前只有通知。 
    

### 推送概述

#### 功能说明

向某单个设备或者某设备列表推送一条通知、或者消息。

推送的内容只能是 JSON 表示的一个推送对象。

调用地址：   
POST [https://api.jpush.cn/v3/push][0]

#### 请求示例

    curl --insecure -X POST -v https://api.jpush.cn/v3/push -u "7d431e42dfa6a6d693ac2d04:5e987ac6d2e04d95a9d8f0d1" -d  '{"platform":"all","audience":"all","notification":{"alert":"Hi,JPush!"}}'
    
    > POST /v3/push HTTP/1.1
    > Authorization: Basic N2Q0MzFlNDJkZmE2YTZkNjkzYWMyZDA0OjVlOTg3YWM2ZDJlMDRkOTVhOWQ4ZjBkMQ==
    

#### 返回示例

    < HTTP/1.1 200 OK
    < Content-Type: application/json
    {"sendno":"18","msg_id":"1828256757"}
    

#### 调用验证

HTTP Header（头）里加一个字段（Key/Value对）：

    Authorization: Basic base64_auth_string
    

其中 base64_auth_string 的生成算法为：base64(appKey:masterSecret)

即，对 appKey 加上冒号，加上 masterSecret 拼装起来的字符串，再做 base64 转换。

### 推送对象

一个推送对象，以 JSON 格式表达，表示一条推送相关的所有信息。

##### 示例与说明

    {
       "platform": "all",
       "audience" : {
          "tag" : ["深圳", "北京"]
       },
       "notification" : {
          "alert" : "Hi, JPush!",
          "android" : {}, 
          "ios" : {
             "extras" : { "newsid" : 321}
          }
       },
       "message": {
           "msg_content": "Demo msg"
       }
       "options" : {
          "time_to_live" : 60
       }
    }
    

关键字||含义 -|-|- platform | 必填 | 推送平台设置 audience | 必填 | 推送设备指定 notification|可选|通知内容体。是被推送到客户端的内容。与 message 一起二者必须有其一，可以二者并存 message|可选|消息内容体。是被推送到客户端的内容。与 notification 一起二者必须有其一，可以二者并存 options | 可选 | 推送参数 

#### platform

JPush 当前支持 Android, iOS, Windows Phone 三个平台的推送。其关键字分别为："android", "ios", "winphone"。

推送到所有平台：

    { "platform" : "all" }
    

指定特定推送平台：

    { "platform" : ["android", "winphone"] }
    

#### audience

推送设备对象，表示一条推送可以被推送到哪些设备列表。确认推送设备对象，JPush 提供了多种方式，比如：别名、标签、注册ID、分群、广播等。

##### all

如果要发广播（全部设备），则直接填写 “all”。

##### 推送目标

广播外的设备选择方式，有如下几种：

关键字|含义|值类型|说明 -|-|-|-|- tag|JSON Array|标签|数组。多个标签之间是 OR 的关系，即取并集。 | 用标签来进行大规模的设备属性、用户属性分群。 一次推送最多 20 个。 tag_and|JSON Array|标签AND|数组。多个标签之间是 AND 关系，即取交集。 | 注册与 tag 区分。一次推送最多 20 个。 alias|JSON Array |别名|数组。多个别名之间是 OR 关系，即取并集。 | 用别名来标识一个用户。一个设备只能绑定一个别名，但多个设备可以绑定同一个别名。一次推送最多 1000 个。 registration_id|JSON Array|注册ID|数组。多个注册ID之间是 OR 关系，即取并集。设备标识。一次推送最多 1000 个。

每种类型的值都是数组（Array），数组里多个值之间隐含的关系是是 OR，即取并集。但 tag_and 不同，其数组里多个值之间是 AND 关系，即取交集。

4 种类型至少需要有其一。如果值数组长度为 0，表示该类型不存在。

这几种类型可以并存。并存时多项的隐含关系是 AND，即取交集。

##### 示例

* 推送给全部（广播）：

{ "platform": "all", "audience" : "all", "notification" : { "alert" : "Hi, JPush!", } } * 推送给多个标签（只要在任何一个标签范围内都满足）：在深圳、广州、或者北京

{ "audience" : { "tag" : [ "深圳", "广州", "北京" ] } } * 推送给多个标签（需要同时在多个标签范围内）：在深圳并且是“女”

{ "audience" : { "tag_and" : [ "深圳", "女" ] } } * 推送给多个别名：

{ "audience" : { "alias" : [ "4314", "892", "4531" ] } } * 推送给多个注册ID：

{ "audience" : { "registration_id" : [ "4312kjklfds2", "8914afd2", "45fdsa31" ] } } * 可同时推送指定多类推送目标：在深圳或者广州，并且是 ”女“ “会员”

{ "audience" : { "tag" : [ "深圳", "广州" ] "tag_and" : [ “女”, "会员“ ] } }

#### notification

“通知”对象，是一条推送的实体内容对象之一（另一个是“消息”），是会作为“通知”推送到客户端的。

其下属属性包含 4 种，3 个平台属性，以及一个 "alert" 属性。

###### alert

通知的内容在各个平台上，都可能只有这一个最基本的属性 "alert"。

这个位置的 "alert" 属性（直接在 notification 对象下），是一个快捷定义，各平台的 alert 信息如果都一样，则可不定义。如果各平台有定义，则覆盖这里的定义。

    {
        "notification" : {
            "alert" : "Hello, JPush!"
        }
    }
    

上面定义的 notification 对象，将被推送到 "platform" 指定的多个平台，并且其通知 alert 信息都一样。

###### android

Android 平台上的通知。

被 JPush SDK 按照一定的通知栏样式展示。

支持的字段有：

关键字|||含义|说明 -|-|-|-|- alert|string|必填|通知内容|这里指定了，则会覆盖上级统一指定的 alert 信息；内容可以为空字符串，则表示不展示到通知栏。 title|string|可选|通知标题|如果指定了，则通知里原来展示 App名称的地方，将展示成这个字段。 builder_id|int|可选|通知栏样式ID|Android SDK 可设置通知栏样式，这里根据样式 ID 来指定该使用哪套样式。 extras|JSON Array|可选|扩展字段。 |这里自定义 JSON 格式的 Key/Value 信息，以供业务使用。

    {
        "notification" : {
            "android" : {
                 "alert" : "hello, JPush!", 
                 "title" : "JPush test", 
                 "builder_id" : 3, 
                 "extras" : {
                      "news_id" : 134, 
                      "my_key" : "a value"
                 }
            }
        }
    }
    

##### ios

iOS 平台上 APNs 通知。

该通知内容会由 JPush 代理发往 Apple APNs 服务器，并在 iOS 设备上在系统通知的方式呈现。

该通知内容满足 APNs 的规范，支持的字段如下：

关键字|||含义|说明 -|-|-|-|- alert|string|必填|通知内容|这里指定了，将会覆盖上级统一指定的 alert 信息；内容为空则不展示到通知栏。支持 emoji 表情。 sound|string|可选|通知提示声音|如果无此字段，则此消息无声音提示；有此字段，如果找到了指定的声音就播放该声音，否则播放默认声音。 badge|int|可选|应用角标|如果不填，表示不改变角标数字；否则把角标数字改为指定的数字；为 0 表示清除。 content-available|boolean|可选|静默推送标志|如果为 1 表示要静默推送。 extras|JSON Array|可选|扩展字段|这里自定义 Key/value 信息，以供业务使用。

> iOS 通知 JPush 要转发给 APNs 服务器。APNs 协议定义通知长度为 255 字节。JPush 因为需要重新组包，并且考虑一点安全冗余，要求本部分内容总体长度限制为：220 个字节。

> 另外，JPush 在推送时使用 utf-8 编码，所以一个汉字占用 3 个字节长度。 

    {
        "notification" : {
            "ios" : {
                 "alert" : "hello, JPush!", 
                 "sound" : "happy", 
                 "badge" : 1, 
                 "extras" : {
                      "news_id" : 134, 
                      "my_key" : "a value"
                 }
            }
        }
    }                
    

##### winphone

Windows Phone 平台上的通知。

该通知由 JPush 服务器代理向微软的 MPNs 服务器发送，并在 Windows Phone 客户端的系统通知栏上展示。

该通知满足 MPNs 的相关规范。当前 JPush 仅支持 toast 类型：

关键字|||含义|说明 -|-|-|-|- alert|string|必填|通知内容|会填充到 toast 类型 text2 字段上。 alert| string |必填 |通知内容 |这里指定了，将会覆盖上级统一指定的 alert 信息；内容为空则不展示到通知栏。 title| string| 可选 |通知标题 |会填充到 toast 类型 text1 字段上。 _open_page |string| 可选 |点击打开的页面名称 |点击打开的页面。会填充到推送信息的 param 字段上，表示由哪个 App 页面打开该通知。可不填，则由默认的首页打开。 extras| JSON Array |可选 |扩展字段| 作为参数附加到上述打开页面的后边。

    {
        "notification" : {
            "winphone" : {
                 "alert" : "hello, JPush!", 
                 "title" : "Push Test", 
                 "_open_page" : "/friends.xaml", 
                 "extras" : {
                      "news_id" : 134, 
                      "my_key" : "a value"
                 }
            }
        }
    }
    

#### message

应用内消息。或者称作：自定义消息，透传消息。

此部分内容不会展示到通知栏上，JPush SDK 收到消息内容后透传给 App。App 需要自行处理。

iOS 平台上，有此部分内容，才会推送应用内消息通道。

Windows Phone 平台上，暂时不支持应用内消息。

消息包含如下字段：

|关键字|||含义|说明 |-|-|-|-|-| |msg_content| string| 必填| 消息内容本身|   
|title| string| 可选| 消息标题|   
|content_type| string| 可选| 消息内容类型   
|extras| JSON Array| 可选| JSON 格式的可选参数| 

> notification 与 message 并存（一次推送都发）时，Android 1.6.2及以下版本 只能收到通知部分，message 部分暂未时透传给 App。 Android 1.6.3及以上SDK 版本已做相应调整。 只有iOS 1.7.3及以上的版本才能正确解析v3的message。

#### options

推送可选项。

当前包含如下几个可选项：

关键字|||含义 |说明 -|-|-|-|- sendno |int| 可选 |推送序号| 纯粹用来作为 API 调用标识，API 返回时被原样返回，以方便 API 调用方匹配请求与返回。 time_to_live |int |可选| 离线消息保留时长| 推送当前用户不在线时，为该用户保留多长时间的离线消息，以便其上线时再次推送。默认 86400 （1 天），最长 10 天。设置为 0 表示不保留离线消息，只有推送当前在线的用户可以收到。 big_push_duration| int| 可选| 定速推送时长(分钟)| 又名缓慢推送，把原本尽可能快的推送速度，降低下来，给定的n分钟内，均匀地向这次推送的目标用户推送。最大值为1400.未设置则不是定速推送。 override_msg_id| long| 可选| 要覆盖的消息ID| 如果当前的推送要覆盖之前的一条推送，这里填写前一条推送的 msg_id 就会产生覆盖效果，即：1）该 msg_id 离线收到的消息是覆盖后的内容；2）即使该 msg_id Android 端用户已经收到，如果通知栏还未清除，则新的消息内容会覆盖之前这条通知；覆盖功能起作用的时限是：1 天。如果在覆盖指定时限内该 msg_id 不存在，则返回 1003 错误，提示不是一次有效的消息覆盖操作，当前的消息不会被推送。 apns_production| boolean| 可选| APNs是否生产环境| 默认推送生产环境。False 表示要推送测试环境。

### 调用返回

##### HTTP 状态码

参考文档：HTTP-Status-Code

##### 业务返回码

Code| 描述| 详细解释| 实际提示信息| HTTP Status Code -|-|-|-|- 1000|系统内部错误|服务器端内部逻辑错误，请稍后重试。||500 1001|只支持 HTTP Post 方法 |不支持 Get 方法。||405 1002| 缺少了必须的参数| 必须改正。|| 400 1003| 参数值不合法| 必须改正。|| 400 1004| 验证失败 |必须改正。|| 401 1005| 消息体太大|必须改正。

JPush 限制整个的消息体长度为 1K 字节。

iOS 通知长度不得超过220字节（包括自定义参数和符号）。||400 1008| app_key参数非法| 必须改正。|| 400 1011| 没有满足条件的推送目标 ||| 400 1020| 只支持 HTTPS 请求| 必须改正。|| 404 1030| 内部服务超时| 稍后重试。|| 503

### 参考

* 获取推送送达API：[Report-API][0]
* 老版本 Push API：[Push API v2][1]
* HTTP规范参考：[HTTP基本认证][2]
* Apple APNs 规范：[Apple Push Notification Service][3]
* Microsoft MPNs 规范：[Push notifications for Windows Phone 8][4]


[0]: ../report_api_v3
[1]: ../rest_api_v2_push
[2]: http://zh.wikipedia.org/zh/HTTP基本认证
[3]: https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html#//apple_ref/doc/uid/TP40008194-CH100-SW12
[4]: http://msdn.microsoft.com/en-us/library/windowsphone/develop/ff402558(v=vs.105).aspx