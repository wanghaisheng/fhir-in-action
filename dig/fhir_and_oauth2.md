# FHIR and OAuth2
[原文链接：FHIR and OAuth2](http://fhirblog.com/2014/06/17/fhir-and-oauth2/)

**文中提到的request type到底有哪些
所说的code type和‘implicit’ flow到底该如何理解
**
一些基本概念的解释.

*   _Identity_实体的属性(诸如姓名、性别、出生日期等)之一，一个人可以有多个标识.
*   _Authentication_认证就是向服务器证明自己是自己所声称的那个实体。最常见的用户名密码就是其中一种方式.
*   _Authorization_授权就是确定和控制访问特定类型的资源.比方说 访问患者临床数据用户的角色必须是医师.
*   _Audit_系统中完成了哪些操作，是谁执行的的跟踪记录。FHIR中利用[SecurityEvent](http://www.hl7.org/implement/standards/fhir/securityevent.html)来表示, 是基于 [IHE ATNA](http://wiki.ihe.net/index.php?title=Audit_Trail_and_Node_Authentication "Audit_Trail_and_Node_Authentication") 规范的.

这些概念之间是互相独立的。比方说在授权之前你需要认证用户的身份， 在进行一些操作时新建一份审计记录。

FHIR [quite explicitly](http://www.hl7.org/implement/standards/fhir/security.html)并不是安全协议, 其他的一些组织机构负责构建此类的标准。但FHIR中对安全相关的问题也给出了一些参考意见(e.g. [tags](http://www.hl7.org/implement/standards/fhir/extras.html#tag) for privacy) ，也有一些很有用的资源 (such as [SecurityEvent](http://www.hl7.org/implement/standards/fhir/securityevent.html) 和 [Provenance](http://www.hl7.org/implement/standards/fhir/provenance.html)),当使用HTTP/S就可以使用诸如[TLS](http://en.wikipedia.org/wiki/Transport_Layer_Security) (Transport Layer Security), [OAuth2](http://oauth.net/2/) and [OpenID Connect](http://openid.net/connect/)等标准 后续的文章中我们将重点讨论后面2个.

**_OAuth2 是一个能够让应用程序访问位于另外的服务器上用户数据的框架，应用程序无需在本地保存用户凭证。_**

例如,你的web应用程序能够让患者从FHIR服务器上访问他们的数据，系统架构可能如下:

[![oauth-1](oauth-1.png?w=630)](oauth-1.png)

用户通过界面与网络上的应用程序进行交互，应用程序服务器端的程序从FHIR服务器中获取患者数据，比如说体重。.

假如用户通过用户名密码登录了应用程序的系统，但FHIR服务器又如何知道该传输患者的数据呢？
一种方式是用户在FHIR服务器上也有一个帐号，应用程序的系统之中保存了这些凭据，这样应用系统在请求数据时可以将其传输给FHIR服务器,这样做的问题在于：

*   应用系统本身会保存用户凭据，提高了信息泄漏的风险([Weakest links in a chain](http://www.phrases.org.uk/meanings/the-weakest-link.html) and all that stuff)
*   如果需要访问其他FHIR服务器上的数据，要么用户在所有服务器上的凭据都是一样的，要么就需要保存一个用户的多份凭据.
*   如果用户不希望应用程序继续访问服务器上的数据，用户必须更改所有服务器上的账户信息
*   每个服务器和应用程序都需要保存每个用户的凭据。极容易在变更、启动过程中产生混乱，由此增加泄漏的风险

OAuth2的方式通过份认证和授权角色与提供信息的橘色分离解决了此类问题.

[![oauth-2 (1)](oauth-2-1.png?w=630)](oauth-2-1.png)

授权服务器是一个单独的组件，用户凭据保存在这之中。其他组件如应用程序、FHIR服务器通过授权服务器来标识确定用户，表明调用程序可以访问那些数据.

一般而言，授权服务器是一些可靠的系统，用户在这些系统上已经有了帐号-诸如google、亚马逊，但也可以是一个单独的构件或者是FHIR服务器的一个功能.

工作机理如下.

[![OAuth2sequencediagram](oauth2sequencediagram.png?w=630&#038;h=359)](oauth2sequencediagram.png)

1.  用户向应用程序(客户端)发送请求(通过浏览器)，应用程序需要从FHIR服务器中获取数据，应用程序发送了一个“浏览器重定向“给用户的浏览器，用户的浏览器会向可信的授权服务器发送登录请求.
2.  授权服务器发送登录界面给用户.
3.  用户填写好登录信息，同意应用程序访问授权服务器上的数据，包括了数据及其相应的访问权限.
4.  将登陆界面提交给授权服务器 .
5.  授权服务器验证登录信息
6.  授权服务器发送浏览器重定向给用户浏览器，跳转到应用程序的某个节点。重定向链接中包含了授权码，允许访问用户数据
7.  用户浏览器重定向到应用程序的回调函数，这样应用程序就能:

    1.  利用授权码调用授权服务器，授权服务器返回Access token
    2.  应用程序向FHIR服务器发送查询请求，请求参数中包含access token.
    3.  FHIR服务器确认access token的合法性，并完成查询请求，将数据返回给应用程序.
    4.  应用程序利用数据 展示在web浏览器中

**备注:**

*   这只是对整体流程的大概描述，省略了大量的细节，大多数情况会使用某个库来实现OAuth2.
*   需要OAuth2授权的FHIR服务器通常称之为 ‘OAuth2 protected’.
*   资源服务器校验访问令牌的机制取决于开发人员，有诸多方案可供选择:

    *   假如授权服务器是FHIR服务器的一部分，可以直接进行校验.
    *   FHIR服务器可以单独调用授权服务器来确认这些信息
    *   授权服务器可以给访问令牌签名，以此向FHIR服务器证明令牌是合法的.

*   在授权服务器返回访问令牌的同时也要返回一个单独的刷新令牌，刷新令牌保存在应用程序本地。访问令牌有一定的生命周期，通常是一个小时，当其过期后，应用程序可以使用刷新令牌来获取一个新的访问令牌。这样设计的原因在于授权服务器可以废除刷新令牌，这样应用程序就无法获取新的访问令牌，无法在访问令牌过期之后继续访问FHIR服务器。没有这种机制，FHIR服务器就得每次都先与授权服务器核对一下是否访问令牌已经失效.
*   上述的流程称之为 ‘Code’ request type, 仅适用于应用程序可以保障刷新令牌的安全的情况，因为刷新令牌是保存在服务器上。对于桌面端或移动端的应用程序而言，不能采用这样的流程，但可以选用‘implicit’ flow .
*   在这种流程下，应用程序需要与授权服务器单独进行注册。在这个过程中，应用程序收到一个编码，用这个编码可以向授权服务器证明自己的身份，提供应用程序回调地址，这样子在用户登录后就可以重定向至浏览器.
*   OAuth2的安全性完全取决于安全的传输机制-因此向SSL之类的TLS是绝对需要的.
*   上述只是个大概的描述，详细请参阅标准英文，最好使用如下的一些库来实现 [标准库列表](http://oauth.net/2/).




参考资料:

[The OAuth2 Specification](http://oauth.net/2/)

A [post on Grahames Blog](http://www.healthintersections.com.au/?p=2108). This points out some issues with delegating authorization..

[A tutorial by Jakob Jenkov](http://tutorials.jenkov.com/oauth2/index.html)

[Blue Button](http://blue-button.github.io/blue-button-plus-pull/)

[SMART on FHIR](http://smartplatforms.org/smart-on-fhir/)
