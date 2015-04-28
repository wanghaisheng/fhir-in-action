# FHIR: Securing an ecosystem
[原文链接:FHIR: Securing an ecosystem](http://fhirblog.com/2014/06/23/fhir-securing-an-ecosystem/)

**编者注：请关注以下原文下面的评论，Trond Elde提到当患者作为自身数据的使用者，OAuth2和OpenID的方式是使用的，但医疗中很多情况下，在患者授权的前提下，医疗机构扮演了患者数据使用者/用户的角色，可能需要XACML和IHE Secure Retrieve (SeR)解决此类问题**

前面一篇文章[fhir-and-openid-connect](fhir-and-openid-connect.md) 中探讨了 [OAuth2](http://oauth.net/2/) 和 [OpenID Connect](http://openid.net/connect/), 其中介绍了授权服务器如何认证用户，为资源服务器提供访问令牌，资源服务器根据访问令牌，确定它所能提供的信息和服务。[FHIR标准](http://hl7.org/implement/standards/fhir/security.html) 中也提到了如何解决此类问题，同时指出可以使用[IHE IUA 规范](http://wiki.ihe.net/index.php?title=Internet_User_Authorization "Internet_User_Authorization") ([可在此下载](http://stage.ihe.net/Technical_Framework/upload/IHE_ITI_Suppl_IUA_Rev1-0_PC_2013-06-03.pdf)).

每个服务器都拥有自己的认证机制是最直接了当的，但当要在不同服务器之间形成一个生态圈的话就会变得异常复杂，可能会存在如下问题：

*   资源服务器如何保证某个访问令牌的有效性
*   授权服务器如何告诉资源服务器的访问范围，也就是患者允许你访问的数据范围

简单说明一下生态圈，也就是有哪些单独的服务器，扮演哪些角色

处于简化问题的考虑，假设我们的生态圈内只有一个授权服务器。与[IHE XDS](http://wiki.ihe.net/index.php?title=Cross-Enterprise_Document_Sharing "Cross-Enterprise_Document_Sharing") Affinity domain concept的概念类似:

_ “一些医疗机构，以公共的策略和公用的架构来协同工作”_

区别之处在于affinity domain都有自己单独的注册库，我们这里只有一个授权服务器，

生态圈示意图:

[![sources](sources.png?w=630&#038;h=384)](sources.png)


*   授权服务器依托一些数据库来标识患者和医务人员/医疗机构.
*   FHIR Profile和ValueSet 注册库/registry. [profiles](http://www.hl7.org/implement/standards/fhir/profile.html), [ValueSets](http://www.hl7.org/implement/standards/fhir/valueset.html)和扩展.当然也可以使用其他的注册库，主要是HL7所定义的一些扩展。
*   记录定位服务/Record Locator Service.  它扮演的是XDS Registry 的角色，上面保存了DocumentReference资源[getting-documents-from-a-fhir-xds-infrastructure](getting-documents-from-a-fhir-xds-infrastructure.md).
*   一些资源服务器，一个保存资源本身，其他都作为现有医疗信息系统的代理.

**可信的访问令牌**

授权服务器在拿到一个访问令牌后如何确保其有效性，以及用户所同意的数据发布的范围。


当授权服务器和资源服务器是同一个应用时就很简单, 在处理访问令牌的过程中，授权服务器能够在本地缓存中保存详细信息.当多个不同服务器请求该访问令牌时就变得尤为复杂.

有一些方法可供选择：

*   授权服务器通过安全的后台通道向资源服务器提供缓存有效的访问令牌 (比如[Redis](http://en.wikipedia.org/wiki/Redis)服务器内存中保存了有效的访问令牌). 当资源服务器接收到新的请求，先向内存中以确认其有效性和获取范围)
*   实际上访问令牌的结构比[Bearer Token](http://tools.ietf.org/html/rfc6750)要复杂的多 – 比如可以是[JWT](http://tools.ietf.org/html/draft-ietf-oauth-json-web-token-23) (JSON Web Token)，JSON web令牌中包含了失效时间和授权服务器的签名。这就要求资源服务器要有授权服务器的公钥.

选择上面的2种的哪一个，取决于具体的实现(“Affinity Domain”的协议). 无论哪种情况，复杂度要么在服务端 要么在客户端。

**范围定义**


起初用户与授权服务器进行认证时，其中包含了他们允许应用程序能够以用户的身份从资源服务器上访问哪些信息.也就是请求的范围，可以在 [ OAuth2 标准3.3章有详细描述](http://tools.ietf.org/html/rfc6749#section-3.3):

_ scope 参数的值是以空格为分隔符，大小写敏感的字符串列表。这些字符串是由授权服务器所定义的。   如果值中包含了多个空格分隔的字符串，它们的次序并不重要，每个字符串都限定了请求的访问范围._

访问粒度的控制如何做到，并没有共同的标准，参考Grahame提供的一些方案:

*   读取任何数据
*   读取任何未标记为保密的数据
*   读/写操作

    *   用药
    *   过敏
    *   症状
    *   其他临床信息



很明显，原则上，你可以使用一到多种策略，但在实现时具体要看情况选择.

