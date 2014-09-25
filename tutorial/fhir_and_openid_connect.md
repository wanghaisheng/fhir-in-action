# FHIR and OpenID Connect
[原文链接:FHIR and OpenID Connect](http://fhirblog.com/2014/06/19/fhir-and-openid-connect/)

[fhir-and-oauth2](fhir-and-oauth2.md)和[fhir-oauth2-and-the-mobile-client](fhir-oauth2-and-the-mobile-client.md)中探讨了[OAuth2](http://oauth.net/2/)的用法，这样，我们就了解到用户如何授权一个应用程序无需再暴露自己的登录凭据的情况下从其他服务器上获取数据。授权服务器是唯一需要拥有用户名和密码或者所采用的其他授权方式相关信息的组件。

但除了这些登录相关内容，应用程序如何保证用户是谁？当向FHIR服务器查询数据时，返回的数据是给谁了呢？标识用户的流程常常被称之为应用系统间的语境传递，也有诸如CCOW之类的方法可以实现。

要解决此类问题，我们需要更多有关用户身份相关的信息，也就是一到多个应用程序和FHIR服务器都能够识别出来的属性，能够确保这个人是同一个人。在FHIR中，我们认为这就是一个Identifier，很多资源、实体都有这个属性，Patient也不例外。

从另一种角度来看，应用程序的查询应该是这样的：

    GET [base]/fhir/Condition?subject.identifier=PRP1660

比如查询患者标识为PRP1660的所有condition。

这里的Identifier说的是业务上的标识，比如社保卡号，身份证号，而不是服务器上该资源实例的标识，或者说是主键之类。

一个Identifier由2部分组成：标识的字符串和保持标识字符串唯一性的系统，或者说标识字符串的类型。

如何获得此类标识，与授权服务器类似，我们需要的是一个身份标识服务器，可以从中获取患者的标识-这正是[OpenID Connect](http://openid.net/connect/)的作用.

OpenID Connect是一种基于OAuth2的协议，能够提供标识服务，也就是说它和OAuth2使用同样的组件和流程，但是增加了与标识相关的内容，比如除了登录到授权服务器的信息之外，包含更多与人相关的属性，诸如姓名、email、出生日期和身份标识等!

 OpenID Connect中定义了与user用户相关的一些标准属性，[可以在此处查看](http://openid.net/specs/openid-connect-core-1_0.html#Claims),尽管也允许你定义一些其他属性，但是identifier并不在其中。由于identifier并不是标准属性，在所有采用这种方式的应用程序之间我们需要一份协议来来表示identifier。常常将这称之为 交易双方协议或者实施指南的一部分。

从另一种角度来OpenID Connect，它利用了OAuth现有的组件，使用了现有的流程并增加了一些额外的功能。但是其中组件名称与OAuth2中叫法不一样，Client-Relying Party，Authorization server-OpenID connect provider


与 OAuth2标准流程极其相似，如下图，注意其中的不同点

[![openid](openid.png?w=630)](openid.png)

1.  浏览器与APP进行交互，重定向至授权服务器。入参包括了scope参数，其中罗列了app需要的identity claim，还有一个“openid”用来标识这是一个OpenID Connect请求
2.  重定向浏览器调用授权服务器，渲染出一个登录界面。页面上常常包含了app所请求的claim，这样用户能够对其进行授权
3.  用户输入了登录凭据之后，对信息发布进行授权.
4.  详细信息发送至授权服务器，对用户进行认证，并向APP返回一个授权码
5.  应用程序拿到重定向请求和授权码之后，利用授权码作为参数发送请求到授权服务器的访问令牌节点，授权服务器校验该授权码，返回三种令牌：
    1.  访问令牌
    2.  刷新令牌
    3.  ID令牌
6.其他流程如上所述


上面省略了诸多细节，详细信息请参考 [OpenID Connect标准](http://openid.net/specs/openid-connect-core-1_0.html)

当然在上面的第七布也可以有另外一个单独的 ‘Claims Provider’ – 其中只返回访问令牌/刷新令牌，可以单独调用这个claim provider，这种情况可以使用标准中定义的_UserInfo_节点。

ID令牌是OpenID Connect特有的，它是 [JSON Web Token](http://tools.ietf.org/html/draft-ietf-oauth-json-web-token-21)形式的，用 “_a compact URL-safe means of representing claims to be transferred between two parties_”的方式来描述  这里的claim其实指的是user的属性。其中就包含了identifier属性，格式为JSON string.

如下例所示 [摘自标准](http://openid.net/specs/openid-connect-core-1_0.html#TokenResponse)) .

    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store
    Pragma: no-cache

    {
    "access_token": "SlAV32hkKG",
    "token_type": "Bearer",
    "refresh_token": "8xLOxBtZp8",
    "expires_in": 3600,
    "id_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjFlOWdkazcifQ.ewogImlzcyI6ICJodHRwOi8vc2VydmVyLmV4YW1wbGUuY29tIiwKICJzdWIiOiAiMjQ4Mjg5NzYxMDAxIiwKICJhdWQiOiAiczZCaGRSa3F0MyIsCiAibm9uY2UiOiAibi0wUzZV3pBMk1qIiwKICJleHAiOiAxMzExMjgxOTcwLAogImlhdCI6IDEzMTEyODA5NzAKfQ.ggW8hZ1EuVLuxNuuIJKX_V8a_OMXzR0EHR9R6jgdqrOOF4daGU96Sr_P6qJp6IcmD3HP99Obi1PRs-3LOp146waJ8IhehcwL7F09JdijmBqkvPeB2T9CJNqeGpe-ccMg4vfKjkM8FcGvnzZUN4_KSP0aAp1tOJ1zZwgjxqGByKHiOtX7TpdQyHE5lcMiKPXfEIQILVq0pc_E2DzL7emopWoaoZTF_m0_N0YzFC6g6EJbOEoRoSK5hoDalrcvRYLSrQAZZKflyuVCyixEoV9GfNQC3_osjzw2PAithfubEEBLuVVk4XUVrWOLrLl0nx7RkKU8NXNHq-rvKMzqg"
    }



JSON object响应中包含了三类令牌，id令牌是JWT格式的.其中包含了三部分，用 &#8216;.&#8217;作为分隔符, 每一部分都是base64编码。中间部分包含了claim 也就是user的属性。

综上所述，OpenID Connect在OAuth2之上增加了服务器存储和传输身份标识数据的功能，APP可以在管理患者信息时可以使用。至于说服务器中最初怎么获得标识，并将其与登录的凭据相关联则取决于具体的实现。


从客户端的角度来讲，
1.  使用OAuth2
2.  在第一步调用授权服务器时，选择你所需要的claim/属性，外加一个“openid”字符串
3.  在获得访问令牌时就能够拿到所需数据


OpenID Connect的基本应用就是这样子，但是一些未解决的问题包括：

*   当用户授权APP可以访问自己的数据时，如何控制其中的粒度，比如 检验结果可以给你看，但其他数据就不行
*   鉴于OpenID Connect的&#8216;claim&#8217;对于identifier而言，我们如何表示我们需要这个字段/属性?

如何利用这些标准来保证数据源和数据服务的安全性。
