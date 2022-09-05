# 一、开发准备工作

## ①、应用免登

1. 获取免登授权码。
   - [x]  [微应用免登](https://open.dingtalk.com/document/orgapp-client/logon-free-process) 
   
2. 获取AccessToken。
   - [x] 调用接口获取access_token，详情请参考 [获取企业内部应用的access_token](https://open.dingtalk.com/document/orgapp-server/obtain-orgapp-token)。
   
     ```JSON
     {
     	"errcode": 0,
     	"access_token": "e24bb103f31930fa964f13439d80e78a",
     	"errmsg": "ok",
     	"expires_in": 7200 //11.26.00请求
     }
     ```
   
     
   
3. 获取userid。
   - [x] 调用接口获取用户的userid，详情请参考 [通过免登码获取用户信息](https://open.dingtalk.com/document/orgapp-server/obtain-the-userid-of-a-user-by-using-the-log-free)。
   
     （利用 accesstoken 以及 免登码 code）
   
     ```json
     {
     	"errcode": 0,
     	"errmsg": "ok",
     	"result": {
     		"device_id": "20f1206c9e77357eb6889ae4188666d8",
     		"name": "吴年乐",
     		"sys": false,
     		"sys_level": 0,
     		"unionid": "Eqyl1iPsafiiN8iPhiPWRYLufgiEiE",
     		"userid": "01542667325021484944"
     	},
     	"request_id": "16m2wlaqfr37e"
     }
     ```
   
     
   
4. 获取用户详情。
   - [x] 调用接口获取用户详细信息，详情请参考 [根据userId获取用户详情](https://open.dingtalk.com/document/orgapp-server/query-user-details)。
   
     （利用 accesstoken 以及 userid）
   
     * 无该接口权限
   
     

## ②、JSAPI鉴权 (获取密钥)

1. 获取access_token

   - [x]  企业内部应用可通过[获取企业内部应用的access_token](https://open.dingtalk.com/document/orgapp-server/obtain-orgapp-token#topic-1936350)接口获取。

2. 获取jsapi_ticket

   - [x] 通过[获取jsapi_ticket](https://open.dingtalk.com/document/isvapp-server/obtain-jsapi_ticket#topic-1936785)接口获取 jsapi_ticket 时，请注意：

     ```JSON
     {
     	"errcode": 0,
     	"errmsg": "ok",
     	"ticket": "8T7tKdlbRbD8aHPLb6631q6ZpwYT7fYjHCkEeVaniZc8kKbghrG6V5SmkJrHQRSWJ8Zlb8wDEQb8rSeVGrKwFV",
     	"expires_in": 7200 // 11.47.00请求
     }
     ```
   
     
   
   - 企业内部应用和第三方企业应用获取 jsapi_ticket 后，当 jsapi_ticket 未过期时，再次调用 get_jsapi_ticket 接口获取到的 jsapi_ticket 和老的 jsapi_ticket 值相同，只是**过期时间续期2小时**。
   
   - 企业内部应用获取 jsapi_ticket ，一个 appKey 对应一个 jsapi_ticket ，所以在使用的时候需要将 jsapi_ticket 以 appKey 为维度进行缓存下来（设置缓存过期时间2小时），**并不需要每次都通过接口拉取**。

## ③、获取签名参数

在前端进行鉴权之前，需要获取以下签名所需的参数：

| **参数**      | **说明**                                                     | 开发企业内部应用                                             | ISV开发第三方企业应用                              |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------------------- |
| **url**       | 当前网页的URL，不包含#及其后面部分。**说明** 如果是单页面应用，请参考本文的最上方[接入必读](https://open.dingtalk.com/document/isvapp-client/jsapi-authentication#section-e37-g6p-xmc)**。** | —                                                            | —                                                  |
| **nonceStr**  | 自定义固定字符串。                                           | —                                                            | —                                                  |
| **agentId**   | 应用的标识                                                   | 可以在[开发者后台](https://open-dev.dingtalk.com/#/isveapp)的应用详情页中获取。 | 可以从授权信息中获取到。                           |
| **timeStamp** | 时间戳                                                       | 当前时间，但是前端和服务端进行校验时候的值要一致。           | 当前时间，但是前端和服务端进行校验时候的值要一致。 |
| **corpId**    | 企业ID                                                       | 企业的corpid，可以在[开发者后台](https://open-dev.dingtalk.com/#/isveapp)首页获取。 | 通过在页面地址上追加?`corpId=$CORPID$`进行获取。   |

## ④、计算签名

在服务端通过  **sign(** **`ticket` ,** **`nonceStr`**, **`timeStamp`**, **`url `** **)**  计算前端校验需要使用的签名信息。

示例代码：

```javascript
import java.net.URL;
import java.net.URLDecoder;
import java.security.MessageDigest;
import java.util.Formatter;
import java.util.Random;

/**
 * 计算dd.config的签名参数
 **/
public class DdConfigSign {

    /**
     * 计算dd.config的签名参数
     *
     * @param jsticket  通过微应用appKey获取的jsticket
     * @param nonceStr  自定义固定字符串
     * @param timeStamp 当前时间戳
     * @param url       调用dd.config的当前页面URL
     * @return
     * @throws Exception
     */
    public static String sign(String jsticket, String nonceStr, long timeStamp, String url) throws Exception {
        String plain = "jsapi_ticket=" + jsticket + "&noncestr=" + nonceStr + "&timestamp=" + String.valueOf(timeStamp)
            + "&url=" + decodeUrl(url);
        try {
            MessageDigest sha1 = MessageDigest.getInstance("SHA-256");
            sha1.reset();
            sha1.update(plain.getBytes("UTF-8"));
            return byteToHex(sha1.digest());
        } catch (Exception e) {
            throw new Exception(e.getMessage());
        }
    }

    // 字节数组转化成十六进制字符串
    private static String byteToHex(final byte[] hash) {
        Formatter formatter = new Formatter();
        for (byte b : hash) {
            formatter.format("%02x", b);
        }
        String result = formatter.toString();
        formatter.close();
        return result;
    }

    /**
     * 因为ios端上传递的url是encode过的，android是原始的url。开发者使用的也是原始url,
     * 所以需要把参数进行一般urlDecode
     *
     * @param url
     * @return
     * @throws Exception
     */
    private static String decodeUrl(String url) throws Exception {
        URL urler = new URL(url);
        StringBuilder urlBuffer = new StringBuilder();
        urlBuffer.append(urler.getProtocol());
        urlBuffer.append(":");
        if (urler.getAuthority() != null && urler.getAuthority().length() > 0) {
            urlBuffer.append("//");
            urlBuffer.append(urler.getAuthority());
        }
        if (urler.getPath() != null) {
            urlBuffer.append(urler.getPath());
        }
        if (urler.getQuery() != null) {
            urlBuffer.append('?');
            urlBuffer.append(URLDecoder.decode(urler.getQuery(), "utf-8"));
        }
        return urlBuffer.toString();
    }

    public static String getRandomStr(int count) {
        String base = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
        Random random = new Random();
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < count; i++) {
            int number = random.nextInt(base.length());
            sb.append(base.charAt(number));
        }
        return sb.toString();
    }
}
```

## ⑤、引入使用的JS

- [x]  ```
   npm install dingtalk-jsapi
   ```

## ⑥、JSAPI鉴权

**示例代码**：

**注意**

- **dd.config**中所有的参数必须直接来自服务端，不能直接在前端定义。
- 一个页面只需要调用一次**dd.config**即可，重复调用会复用第一次调用的参数，单页应用（SPA）的**router**切换视为同一个页面。

```javascript
dd.config({
    agentId: '', // 必填，微应用ID
    corpId: '',//必填，企业ID
    timeStamp: '', // 必填，生成签名的时间戳
    nonceStr: '', // 必填，自定义固定字符串。
    signature: '', // 必填，签名
    type:0/1,   //选填。0表示微应用的jsapi,1表示服务窗的jsapi；不填默认为0。该参数从dingtalk.js的0.8.3版本开始支持
    jsApiList : [
        'runtime.info',
        'biz.contact.choose',
        'device.notification.confirm',
        'device.notification.alert',
        'device.notification.prompt',
        'biz.ding.post',
        'biz.util.openLink',
    ] // 必填，需要使用的jsapi列表，注意：不要带dd。
});

dd.error(function (err) {
    alert('dd error: ' + JSON.stringify(err));
})//该方法必须带上，用来捕获鉴权出现的异常信息，否则不方便排查出现的问题
```

**参数说明**：

| **参数**  | 类型   | 是否必填 | 说明                                                |
| --------- | ------ | -------- | --------------------------------------------------- |
| agentId   | String | 是       | 微应用ID。                                          |
| corpId    | String | 是       | 企业的corpid。                                      |
| timeStamp | String | 是       | 生成签名的时间戳。                                  |
| nonceStr  | String | 是       | 自定义固定字符串。                                  |
| signature | String | 是       | JSAPI签名，可以参考简易教程中的获取签名的示例代码。 |
| type      | number | 否       | 如果是要调用服务窗的jsapi，请填1。                  |
| jsApiList | Array  | 是       | 需要调用的jsapi列表。                               |

## ⑦、调用JSAPI

完成鉴权后，便可以在dd.ready里调用JSAPI了。

代码示例：

```js
dd.ready(function() {
  dd.biz.contact.choose({
    multiple: true, //是否多选：true多选 false单选； 默认true
    users: ['10001', '10002', ...], //默认选中的用户列表，员工userid；成功回调中应包含该信息
    corpId: 'dingb4ff1079f84f8d54', //企业id
    max: 10, //人数限制，当multiple为true才生效，可选范围1-1500
    onSuccess: function(data) {
    /* data结构
      [{
        "name": "张三", //姓名
        "avatar": "
http://g.alicdn.com/avatar/zhangsan.png
" //头像图片url，可能为空
        "emplId": '0573', //员工userid
       },
       ...
      ]
    */
    },
    onFail : function(err) {}
});
```



# 二、业务逻辑

## ①、往指定的群里发指定消息

### 1. 调用请求打开指定的群聊

#### 1.1[根据corpid选择会话](https://open.dingtalk.com/document/isvapp-client/select-session-based-on-corpid) (只能选择群聊/创建一个新的群聊)

* 携带：

  |        字段         |            描述            |
  | :-----------------: | :------------------------: |
  |       corpId        |        企业的corpid        |
  | isAllowCreateGroup  |    是否允许创建新的会话    |
  | filterNotOwnerGroup | 是否限制为自己创建的会话。 |

* 返回值：

  |  参数  |           说明            |
  | :----: | :-----------------------: |
  | chatId | 会话id.该会话id永久有效。 |
  | title  |        会话的标题         |



### 2. 像分享一下，将编辑好的数据发送到选中的群聊

#### 2.1 [获取会话信息](https://open.dingtalk.com/document/isvapp-client/obtain-session-information-1) (可选择联系人)

* 携带：

  |   字段    |       描述       |
  | :-------: | :--------------: |
  |  corpId   |   企业的corpid   |
  | isConfirm | 是否弹出确认窗口 |

* 返回值：

  | 参数  |                             说明                             |
  | :---: | :----------------------------------------------------------: |
  |  cid  | 会话id。该cid在服务端API-发送普通消息中使用，只能使用一次，使用后会失效。发送普通消息使用说明，请查看[发送普通消息](https://open.dingtalk.com/document/isvapp-server/send-normal-messages#topic-1936787)。 |
  | title |                          会话的标题                          |



#### 2.2 [发送信息](https://open.dingtalk.com/document/isvapp-server/send-normal-messages)

- 携带：

  - Query参数： access_token

  - Body参数：

    | **名称** | 是否必填 |                           **描述**                           |
    | :------: | :------: | :----------------------------------------------------------: |
    |  sender  |    是    |                     消息发送者的userid。                     |
    |   cid    |    是    | 群会话或者个人会话的id，通过JSAPI接口唤起联系人界面选择会话获取会话cid。（上面获得） |
    |   msg    |    是    | 消息内容，最长不超过2048个字节。[消息格式参考](https://open.dingtalk.com/document/isvapp-server/message-types-and-data-format) |

    

### 3. 以**企业的名义**向企业群会话里推送消息

* 但只支持内部群、全员群以及部门群，普通群和合作群无法实现该功能。

* 携带：

  * Query参数： access_token
  * Body参数：chatid （[根据corpid选择会话](https://open.dingtalk.com/document/orgapp-client/select-session-based-on-corpid?spm=ding_open_doc.document.0.0.58c6722fNrTZDF#topic-2025118)）、 msg （最长不超过2048个字节，企业内部应用消息类型和样例参考[消息类型与数据格式](https://open.dingtalk.com/document/orgapp-server/message-types-and-data-format#topic-1945727)）

* 请求示例（HTTP）：

  * ```TXT
    POST https://oapi.dingtalk.com/chat/send?access_token=ACCESS_TOKEN
    ```

* 参考文档：https://open.dingtalk.com/document/orgapp-server/send-group-messages

### 4. 以某个应用的名义推送到员工的工作通知消息

* （这个相当于是发送给个人，不是发送到群聊当中）

### 5. 接入酷应用 - 用户身份发送卡片消息到群

[相关链接🔗](https://open.dingtalk.com/document/isvapp-client/send-card-messages-to-the-group)

| 参数                           | 类型     | 是否必填 | 说明                                                         |
| ------------------------------ | -------- | -------- | ------------------------------------------------------------ |
| context                        | Object   | 是       | 应用相关身份标识。                                           |
| context.clientId               | String   | 是       | 应用标识。企业内部应用，传Appkey。**说明** 如何获取Appkey，请参见[基础概念-AppKey](https://open.dingtalk.com/document/org/basic-concepts#topic-2152278)。第三方企业应用，传SuiteKey。**说明** 如何获取SuiteKey，请参见[基础概念-SuiteKey](https://open.dingtalk.com/document/org/basic-concepts#topic-2152278)。 |
| context.corpId                 | String   | 是       | 企业CorpId，用于校验群所属企业，仅能安装到该企业的内部群。**说明**小程序可通过[dd.corpId](https://open.dingtalk.com/document/isvapp-client/dd-corpid#topic-2056777)获取。微应用可通过[获取企业CorpId](https://open.dingtalk.com/document/isvapp-client/obtain-enterprise-corpid#topic-2209945)获取。 |
| openConversationIdList         | String[] | 是       | 需要发送消息的群ID。                                         |
| sendCardRequest                | Object   | 是       | 动态卡片的相关数据。                                         |
| sendCardRequest.cardTemplateId | String   | 是       | 互动卡片的消息模板ID。可通过[创建消息模板](https://open.dingtalk.com/document/group/create-message-template)获取模板ID。![模版ID](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/5017877561/p447241.png) |
| sendCardRequest.outTrackId     | String   | 是       | 唯一标示卡片的外部编码。**说明** 发送不同的卡片内容，需要使用不同的outTrackId。 |
| sendCardRequest.cardData       | Object   | 是       | 卡片数据。详情参见[发送钉钉互动卡片](https://open.dingtalk.com/document/group/send-interactive-dynamic-cards-1#topic-2043166)cardData字段。 |

## ②.从指定的群里读取消息

### 1. 通过酷应用获取信息

https://open.dingtalk.com/document/isvapp-client/message-menu