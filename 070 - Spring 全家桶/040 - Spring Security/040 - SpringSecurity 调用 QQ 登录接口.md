#还没有复习 

## qq互联上的数据



域名`http://cxstudy.top/qq/callback`

回调地址`http://cxstudy.top/qq/callback`

APP ID：`101873563`
APP Key：`d93c4b5f46edbc8c273f06be1bfdea75`



## 登录流程

获取`Access Token`有两种方式。这里使用的时授权码模式

获取`Authorization Code`—》 获取`Access Token`—》 获取`OpenID`—》 调用接口获取数据

1. 获取`access_token`

   1. 获取`Authorization Code`

**发送请求**

请求地址`https://graph.qq.com/oauth2.0/authorize`。`Get`请求

必须要传的参数列表

| 参数          | 值                                                          |
| ------------- | ----------------------------------------------------------- |
| response_type | code（固定值）                                              |
| client_id     | 申请接口后获取到的appid                                     |
| redirect_uri  | 申请接口时设置的回调地址。此url需要进进行URLEncode          |
| state         | pro（这个好像不是固定值，而是一串无序字符，用以保证安全性） |

**响应结果**

发送请求成功后，自动向原先设置的回调地址发送请求，并携带`code`作为请求参数（这个`code`即为所需要的`Authorization Code`）

此`code`有效期为10分钟

2. 使用`Authorization Code`获取`Access Token`

 **发送请求**

请求地址`https://graph.qq.com/oauth2.0/token`。`Get`请求

必须传递的请求参数

| 参数          | 值                                                           |
| ------------- | ------------------------------------------------------------ |
| grant_type    | 授权类型，在本步骤中，此值为"authorization_code"</br>值为此字符串，而非上一步获取到的`code` |
| client_id     | 申请QQ登录成功后，分配给网站的appid。                        |
| client_secret | 申请QQ登录成功后，分配给网站的appkey。                       |
| code          | 上一步返回的authorization code。                             |
| redirect_uri  | 与上面一步中传入的redirect_uri保持一致。                     |

**响应结果**

  | **参数说明**  |                                        **描述**                                         |
  |:-------------:|:---------------------------------------------------------------------------------------:|
  | access_token  |                                授权令牌，Access_Token。                                 |
  |  expires_in   |                           该access token的有效期，单位为秒。                            |
  | refresh_token | 在授权自动续期步骤中，获取新的Access_Token时需要提供的参数。注：refresh_token仅一次有效 |

3. （可选）获取新的`access_token`/刷新令牌

**发送请求**

请求地址`https://graph.qq.com/oauth2.0/token`。`Get`请求

必须传递的请求参数

  |     参数      |                                       含义                                       |
  |:-------------:|:--------------------------------------------------------------------------------:|
  |  grant_type   |                  授权类型，在本步骤中，此值为“refresh_token”。                   |
  |   client_id   |                      申请QQ登录成功后，分配给网站的appid。                       |
  | client_secret |                      申请QQ登录成功后，分配给网站的appkey。                      |
  | refresh_token | 使用上一步中获取到的最新的refresh_token。后续：使用刷新后返回的最新refresh_token |

**响应结果**

| **参数说明**  |                           **描述**                           |
| :-----------: | :----------------------------------------------------------: |
| access_token  |                   授权令牌，Access_Token。                   |
|  expires_in   |              该access token的有效期，单位为秒。              |
| refresh_token | 在授权自动续期步骤中，获取新的Access_Token时需要提供的参数。注：refresh_token仅一次有效 |

2. 使用`access_token`获取`OpenID`

**发送请求**

请求地址`https://graph.qq.com/oauth2.0/me`

必须传递的参数列表

   |     参数     |              含义               |
   | :----------: | :-----------------------------: |
   | access_token | 在Step1中获取到的access token。 |

   **响应结果**

```json
callback( {"client_id":"YOUR_APPID","openid":"YOUR_OPENID"} );
```


3. 调用API获取数据

> 这里调用的是`https://graph.qq.com/user/get_user_info`。此接口为获取用户的基本信息，例如昵称，各种尺寸的头像图片地址，性别等

**发送请求**

请求地址`https://graph.qq.com/user/get_user_info`。`Get`请求

必须传递的参数列表

|        参数        |                             含义                             |
| :----------------: | :----------------------------------------------------------: |
|    access_token    | 可通过[使用Authorization_Code获取Access_Token](http://wiki.connect.qq.com/使用authorization_code获取access_token) 或来获取。  access_token有3个月有效期。 |
| oauth_consumer_key |             申请QQ登录成功后，分配给应用的appid              |
|       openid       | 用户的ID，与QQ号码一一对应。  可通过调用https://graph.qq.com/oauth2.0/me?access_token=YOUR_ACCESS_TOKEN 来获取。 |

**响应结果**

响应结果为json串

|  **参数说明**  |                           **描述**                           |
| :------------: | :----------------------------------------------------------: |
|      ret       |                            返回码                            |
|      msg       | 如果ret<0，会有相应的错误信息提示，返回数据全部用UTF-8编码。 |
|    nickname    |                     用户在QQ空间的昵称。                     |
|   figureurl    |               大小为30×30像素的QQ空间头像URL。               |
|  figureurl_1   |               大小为50×50像素的QQ空间头像URL。               |
|  figureurl_2   |              大小为100×100像素的QQ空间头像URL。              |
| figureurl_qq_1 |                 大小为40×40像素的QQ头像URL。                 |
| figureurl_qq_2 | 大小为100×100像素的QQ头像URL。需要注意，不是所有的用户都拥有QQ的100x100的头像，但40x40像素则是一定会有。 |
|     gender     |              性别。 如果获取不到则默认返回"男"               |

`ret`值为0：正确返回。值为其他：失败。错误码说明详见：[公共返回码说明](http://wiki.connect.qq.com/公共返回码说明)

下图是实操时，成功调用`https://graph.qq.com/user/get_user_info`接口的返回结果

![[../../020 - 附件文件夹/Pasted image 20230402230507.png|750]]