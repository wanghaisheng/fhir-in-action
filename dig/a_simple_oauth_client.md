# A simple OAuth client
[A simple OAuth client](http://fhirblog.com/2014/06/29/a-simple-oauth-client/)


选用的技术：Node.js Javascript，构建一个node服务器，作为OAuth客户端来调用其他的OAuth服务器。整体架构如下图所示


[![OAuth arch](oauth-arch.png?w=630)](oauth-arch.png)


一个基本的web应用程序，包含一个HTML页面，通过HTTPS与 OAuth服务器通讯，由于代码是允许在服务器端而不是客户端，因此可以 选用"授权码"流程。

选用node模块[simple-outh2](https://github.com/andreareginato/simple-oauth2)’

实现拿Google来试验. Google使用OAuth2来授权访问自己的服务-实际上，Google在OAuth模型中包含了授权服务器和资源服务器。第一步，获取客户端用以进行授权服务器认证的凭据，可以在[Developer website](https://developers.google.com/accounts/docs/OAuth2)中找到.

app代码如下:

    var request = require('request');   //https://github.com/mikeal/request
    var path = require('path');
    var express = require('express'),
        app = express();

    app.use(express.cookieParser());
    app.use(express.session({secret: '1234567890QWERTY'}));
    app.use(express.static(path.join(__dirname, 'public')));

    var OAuth2;

    var credentialsGoogle = {
        clientID: "<myclientid>",
        clientSecret: "<mysecret>",
        'site': 'https://accounts.google.com/o/oauth2/',
        'authorizationPath' : 'auth',
        'tokenPath' : 'token'
    };

    // Initial call redirecting to the Auth Server
    app.get('/auth', function (req, res) {
        OAuth2 = require('simple-oauth2')(credentialsGoogle);
        authorization_uri = OAuth2.AuthCode.authorizeURL({
            redirect_uri: 'http://localhost:3001/callback',
            scope: 'openid email https://www.googleapis.com/auth/drive https://www.googleapis.com/auth/tasks',
            state: '3(#0/!~',
            access_type: "offline"      //causes google to return a refresh token
        });

        res.redirect(authorization_uri);
    });

    // Callback endpoint parsing the authorization token and asking for the access token
    app.get('/callback', function (req, res) {
        var code = req.query.code;

        OAuth2.AuthCode.getToken({
            code: code,
            redirect_uri: 'http://localhost:3001/callback'
        }, saveToken);

        function saveToken(error, result) {
            if (error) {
                console.log('Access Token Error', error.message, error);
                res.json({'Access Token Error': error.message});
            } else {
                //see what we've got...
                console.log(result);
                //this adds the expiry time to the token by adding the validity time to the token
                token = OAuth2.AccessToken.create(result);
                //save the response back from the token endpoint in a session
                req.session.token = result;

                //perform the res of the processing now that we have an Access Token
                gotToken(req,res);
            }
        }

        //Now the token has been received and saved in the session, return the next page for processing
        function gotToken(req, res) {
            res.sendfile(__dirname +"/public/main.html");
        };

    });

    //get the lists for the current user...
    app.get("/tasks", function (req, res) {
        //Need a token to access the google services
        if (req.session.token) {
            var AT = req.session.token["access_token"];
            var url = "https://www.googleapis.com/tasks/v1/users/@me/lists";

            var options = {
                method: "GET",
                headers: {
                    "content-type": "application/json+fhir",
                    "authorization": "Bearer " + AT
                },
                rejectUnauthorized: false,      //to allow self-signed cetificates
                uri: url
            };

            request(options, function (error, response, body) {
                res.json(body);
            });
        } else {
            res.json({err:"Not logged in"});
        }

    });

    app.listen(3001);

    console.log("OAuth Client started on port 3001");


1.  首先调用/auth endpoint.使用凭据初始化OAuth2对象，授权和令牌endpoint的位置;然后利用如范围、状态等参数重定向至授权服务器，详细信息可参考google官网
2.  Google提供一个登录界面，用户进行登录，对该请求进行认证。
3.  如果一切顺利，浏览器会重定向至本地的调用endpoint节点， ([http://localhost:3001/callback](http://localhost:3001/callback) 从响应中拿到授权码，向google再次请求访问令牌
4.  获得访问令牌之后，打开一个HTML页面。.
5.  这样，用户就可以访问Google的资源了，比如/task节点能够获得当前用户的所有任务列表.

第27行代码中access_type 的值设为 &#8216;offline&#8217;，要求Google在响应中包含一个刷新令牌，但似乎并没有成功，这里还需要再研究研究。
在获取授权令牌的响应中 你可以看到 id_token，因为我们在scope的属性中包含了 &#8216;openid&#8217;Google也实现了OpenId Connect，同时也能够返回标识信息
