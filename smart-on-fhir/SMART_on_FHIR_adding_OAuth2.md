[原文链接:SMART on FHIR – adding OAuth2](http://fhirblog.com/2014/08/12/smart-on-fhir-adding-oauth2/)

[相关文章链接:SMART on FHIR: Part1英文版](http://fhirblog.com/2014/08/02/smart-on-fhir-part-1)

[相关文章链接:SMART on FHIR: Part 1中文版](SMART_on_FHIR_Part1.md)



在[SMART on FHIR: Part 1中文版](SMART_on_FHIR_Part1.md) 基础上 ,利用SMART版本 的OAuth2 标准为其提供安全保障.这里还是沿用 [HAPI开源库](http://jamesagnew.github.io/hapi-fhir/) 、Tomcat 和IntelliJ IDEA  IDE.

这里的应用场景是假设我是已经登录了电子病历/区域平台的用户，我想利用一些外部的应用程序来实现一些其他功能，比如调用SMART团队开发的儿童生存曲线的APP。

整体的架构如下所示：
[![oauth2](oauth2.png?w=630&#038;h=470)](oauth2.png)

这是一个web应用程序，发布在Tomcat上，其中暴露了FHIR data endpoints 和  OAuth endpoints .  SMART app部署在其他服务器上，在我们的页面上以iFrame的形式来调用. Tomcat server 上的数据可以存储在本地，但是这里调用的其他外部的FHIR服务器，其实就相当于外部FHIR服务器的一个代理。

使用 [Maven](http://maven.apache.org/what-is-maven.html) 来管理项目代码.安装配置略。

在IDE中创建一个 “maven-archetype-webapp” 的maven项目.

由于SMART客户端是一个运行在其他domain的JavaScript程序，对于我们的服务器而言，需要允许它访问我们的数据，为此需要添加对 [CORS](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) 的支持，按照[HAPI中的说明 ](http://jamesagnew.github.io/hapi-fhir/doc_cors.html)，只需在pom.xml 添加dependency 来下载filter即可,修改web.xml file，修改其中的权限，可以直接从HAPI官网上的例子中复制.

接下来考虑如何SMART进行集成.  [SMART服务器端快速入门](http://docs.smartplatforms.org/tutorials/server-quick-start/)提供了很详细的资料 – 总结一下:

1.  在EMR web 界面上, 新建一个 iframe, 将其指向 SMART应用程序, 进行一些基本的配置即可.
比如
    <html>
    <head>
      <title>OpenMRS-FHIR</title>
    </head>
    <body>
      <h1>OpenMRS-FHIR</h1>
      <ul>
        <li><a href="https://fhir.smartplatforms.org/apps/growth-chart/launch.html?fhirServiceUrl=http://104.131.34.222/openmrs-standalone/ws/fhir&patientId=dd74ed8e-1691-11df-97a5-7038c432aabf">Happiness</a></li>
        <li><a href="https://fhir.smartplatforms.org/apps/growth-chart/launch.html?fhirServiceUrl=http://104.131.34.222/openmrs-standalone/ws/fhir&patientId=dda12af7-1691-11df-97a5-7038c432aabf">Valentine</a></li>
      </ul>
    </body>
    </html>

2.  SMART 应用程序从服务器上加载一致性声明/conformance                statement，这样就知道如何获取授权以及token的endpoint
3.  SMART 应用程序调用authorization end point
4.  假设通过授权,  endpoint会重新返回给 SMART 应用程序一个授权码
5.  SMART 应用程序使用获取的授权码再次调用 token endpoint,获取 Authorization Token.
6.  SMART 应用程序利用Authorization Token调用 EMR FHIR endpoints 来获取所需的数据，并进行展示.


下一步构建 Authorization and Token end points, 告知SMART 应用程序如何找到它们。
利用[对Conformance资源的扩展可以实现](http://docs.smartplatforms.org/authorization/conformance-statement/).

首先:

在我们演示用的电子病历系统中新增一个登录功能，用户发送登录信息到login endpoint,对登录信息进行校验，创建一个user token并保存在一个context对象当中，同时将其发送给本地的APP ，这样在启动时即可使用这个user token。由于系统用户同时也需要访问一些受限的资源，用户也会得到一个access token。

Login servlet示例:

    @WebServlet(urlPatterns= {&quot;/auth/login&quot;}, displayName=&quot;Login&quot;).
    public class Login extends HttpServlet {

        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            System.out.println(&quot;login&quot;);
            PrintWriter out = response.getWriter();
            //Here is where we check and validate username &amp; password. We're going to cheat right now...
            //create a user object. This would ultimately be a FHIR obect I suspect...
            Person person = new Person();
            person.userName = request.getParameter(&quot;username&quot;);
            person.userToken = java.util.UUID.randomUUID().toString(); //generate a user token

            //save the user details in the context - we previously created this map...
            ServletContext context =  getServletContext();
            Map&lt;String,Person&gt; usertokens = (Map&lt;String,Person&gt;) context.getAttribute(&quot;usertokens&quot;);
            //save the access token for later use - in production persistent store...
            usertokens.put(person.userToken,person);

            //create an access token for this person ...
            Map&lt;String,JsonObject&gt; oauthtokens = (Map&lt;String,JsonObject&gt;) context.getAttribute(&quot;oauthtokens&quot;);
            JsonObject json = Json.createObjectBuilder()
                    .add(&quot;access_token&quot;, person.userToken)
                    .add(&quot;token_type&quot;, &quot;bearer&quot;)
                    .add(&quot;expires_in&quot;, 3600)
                    .add(&quot;scope&quot;, &quot;patient/*.read&quot;)
                    .build();
            oauthtokens.put(person.userToken,json);

            response.addHeader(&quot;Content-Type&quot;,&quot;application/json+fhir&quot;);
            out.println(person.getJson().toString());
        }
    }


新建一个end-point来启动SMART app  如下:

    @WebServlet(urlPatterns= {&quot;/auth/launch&quot;}, displayName=&quot;Launch SMART application&quot;)
    public class Launch extends HttpServlet {
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            System.out.println(&quot;launch endpoint accessed&quot;);
            String userToken = request.getParameter(&quot;usertoken&quot;);
            String patientId = request.getParameter(&quot;patientid&quot;);

            //make sure this is a logged in person (the have a valid token)
            ServletContext context =  getServletContext();
            Map&lt;String,Person&gt; usertokens = (Map&lt;String,Person&gt;) context.getAttribute(&quot;usertokens&quot;);

            if (usertokens.containsKey(userToken)) {
                //yep, this is a valid user...

                //retrieve the user object and update with the patient they have in context. We'll need this for the access token...
                Person person = (Person) usertokens.get(userToken);
                person.currentPatientId = patientId;

                //the re-direct URL. In reality the url and 'iss' would come from config...
                String url = &quot;https://fhir.smartplatforms.org/apps/growth-chart/launch.html?&quot;;
                url += &quot;iss=http://localhost:8081/fhir&quot;;

                //we'll use the user token as the launch token as we can use that to validate the Auth call..
                url += &quot;&amp;launch=&quot; + userToken;

                response.sendRedirect(url);
            } else {
                response.setStatus(403);    //forbidden.
                PrintWriter out = response.getWriter();
                out.println(&quot;&lt;html&gt;&lt;head&gt;&lt;/head&gt;&lt;body&gt;&lt;h1&gt;User not logged in&lt;/h1&gt;&lt;/body&gt;&lt;/html&gt;&quot;);
            }
        }
    }


接下来是**authorization** end-point. 这里用用户token做启动token来确保请求方的可信，重定向到 _redirect_url_.

代码如下

    @WebServlet(urlPatterns= {"/auth/authorize"}, displayName="Authorize endpoint for FHIR Server")
    public class Authorize extends HttpServlet {
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            System.out.println("auth check...");

            String response_type = request.getParameter("response_type");
            String client_id = request.getParameter("client_id");   //the id of the client
            String redirect_uri = request.getParameter("redirect_uri");
            String scope = request.getParameter("scope");   //what the app wants to do
            String state = request.getParameter("state");

            //the scope parameter includes the launch token - eg patient/*.read launch:7bceb3c6-66e9-46c9-8efd-9f87e76a5f9a
            //so we would pull out both scope and token, check that the token matches the one we set (actually the patient token)
            //and that the scope is acceptable to us. Should move this to a function somewhere...
            String[] arScopes =  scope.split(" ");
            String launchToken = "";
            for (int i = 0; i < arScopes.length; i++){
                System.out.println(arScopes[i]);
                if (arScopes[i].substring(0,7).equals("launch:")) {
                    launchToken = arScopes[i].substring(7);
                }
            }

            ServletContext context =  getServletContext();
            Map<String,Person> usertokens = (Map<String,Person>) context.getAttribute("usertokens");

            if (usertokens.containsKey(launchToken)) {
                //we'll assume that the user is OK with this scope, but this is where we can check...
                //so, now we create an auth_code and re-direct to the redirect_url...
                String auth_code = java.util.UUID.randomUUID().toString();
                //we'll save the auth code in a previously defined context variable. In real life you'd use a
                //persistent store of some type, and likely save more details...
                Map<String,Person> oauthcodes = (Map<String,Person>) context.getAttribute("oauthcodes");

                Person person = (Person) usertokens.get(launchToken);

                oauthcodes.put(auth_code,person);
                //and re-direct to the 'authenticated' endpoint of the application
                response.sendRedirect(redirect_uri + "?code="+auth_code+ "&state="+state);
            } else {
                response.setStatus(403);    //forbidden.
            }

        }
    }

我们可以使用一个注册过程来存储应用程序的ID和回调URL来进一步确保安全性

这样我们就可以交换授权token的授权码。
如下是**Token endpoint**的范例:

    @WebServlet(urlPatterns= {"/auth/token"}, displayName="Token endpoint for FHIR Server")
    public class Token extends HttpServlet {
        protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            PrintWriter out = response.getWriter();

            String code = request.getParameter("code");

            //the map containing auth_code that was set during the authorization phase...
            ServletContext context =  getServletContext();
            Map<String,Person> oauthcodes = (Map<String, Person>) context.getAttribute("oauthcodes");

            //is this a valid access code?
            if (oauthcodes.containsKey(code)) {
                Person person = (Person) oauthcodes.get(code);
                String access_token  = java.util.UUID.randomUUID().toString();
                JsonObject json = Json.createObjectBuilder()
                        .add("access_token", access_token)
                        .add("patient", person.currentPatientId)
                        .add("token_type", "bearer")
                        .add("expires_in", 3600)
                        .add("scope", "patient/*.read")
                        .build();
                response.addHeader("Content-Type","application/json+fhir");

                //save the access token for later use - like the codes, you'd use a persistent store...
                Map<String,JsonObject> oauthtokens = (Map<String,JsonObject>) context.getAttribute("oauthtokens");
                oauthtokens.put(access_token,json);
                //and return the token to the applciation
                out.println(json.toString());

            } else {
                //the auth codes don't match.
                response.setStatus(403);    //forbidden.
                out.println("{}");
            }
        }
    }

有了auth token之后就可以调用实际的FHIR endpoint.

    @Override
    public void handleRequest(SearchMethodBinding.RequestType theRequestType,
                              javax.servlet.http.HttpServletRequest theRequest,
                              javax.servlet.http.HttpServletResponse theResponse)
            throws javax.servlet.ServletException,
            IOException {

        String uri = theRequest.getRequestURI();

        //anyone can access metadata...
        if (uri.equals("/fhir/metadata")) {
            super.handleRequest(theRequestType,theRequest,theResponse);
        } else {
            //but you need to be authorized to access clincial data...
            String auth = theRequest.getHeader("Authorization");
            if (auth != null){
                auth = auth.substring(7);//get rid of the 'Bearer ' at the front
                ServletContext context =  getServletContext();// request.    .setAttribute("oauthtokens", oauthtokens);
                Map<String,JsonObject> oauthtokens = (Map<String,JsonObject>) context.getAttribute("oauthtokens");
                if (oauthtokens.containsKey(auth)) {
                    //we could pull out the actual access token, and apply security logic there...
                    super.handleRequest(theRequestType,theRequest,theResponse);
                } else {
                    theResponse.setStatus(403);    //forbidden.
                }
            } else {
                theResponse.setStatus(403);    //forbidden.
            }
        }
    }

成功之后就会出现下图的效果 有点丑

[![Screen Shot 2014-08-12 at 10.58.22 am](screen-shot-2014-08-12-at-10-58-22-am.png?w=630&#038;h=213)](screen-shot-2014-08-12-at-10-58-22-am.png)

后续也应该考虑如何判断 Auth tokens是否过期。
