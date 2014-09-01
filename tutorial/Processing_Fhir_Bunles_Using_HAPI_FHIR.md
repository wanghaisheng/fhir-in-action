[原文链接:Processing FHIR Bundles using HAPI](http://fhirblog.com/2014/08/18/processing-fhir-bundles-using-hapi/)		
[另外一篇文章:FHIR transactions: the Search functionality](http://fhirblog.com/2014/06/13/fhir-transactions-the-search-functionality/)	

## FHIR 事务:search功能	
FHIR中是通过[transaction](http://www.hl7.org/implement/standards/fhir/http.html#transaction)来实现资源批量处理的.服务器对每个资源都单独处理,看起来就像是每个资源都是单独发送/提交的(除了批量更新的情况之外 要么都成功 要么都失败).
对于服务器而言,要处理bundle中每个资源的引用是很困难的,不论是新添加的资源或者是已经存在的.
其中有一点,服务器首先要判断这个资源服务器上存在不存在.		
用例描述:	
通过FHIR标准将某个可穿戴式设备(家用血糖仪)采集的血糖数据导入到数据仓库当中去.
血糖仪能够采集血糖数据,通过USB接口,利用桌面应用程序把数据取出来,发送给数据仓库.数据仓库期望
所存储的是结构化数据,这样子便于展示和临床决策支持之用.			
思路:	
大体上可以有多次操作和单次操作.	
1.	首先向数据仓库发送查询请求,确保Patient和Device存在,不存在就新增.然后利用Observation来传输(POST方法)
每个血糖结果,在observation中加上对Patient和device的引用即可.
2.	用一个bundle来表达整个与血糖相关的信息,其中包含一个Patient资源,一个Device资源,一个Observation资源
(用于记录每一项结果).	 Observation.subject 是patient  ,  Observation.performer是Device.最终URL类如  POST [base] {?_format=[mime-type]}
客户端生成这个bundle的时候,Patient的ID和Device的ID应该如何赋值,鉴于血糖结果总是新添加的,Observation的ID使用cid:id即可.	
对于Patient资源和Device资源而言,我们是已经给他们了一个id,但这个id又不是资源中的患者或设备的标识,假设我们给ID赋值为cid:ID的话 ,服务器会不断的新增一条患者记录,设备记录,
我们无法与可能之前就存在的记录进行关联.在标准中有如下[一段话](www.hl7.org/implement/standards/fhir/http.html#transaction),提出了解决这个问题的方法:		
```
The application constructing a bundle may not be sure whether a particular resource will already exist at the time that the transaction is executed; this is typically the case with reference resources such as patient and provider. In this case, the bundle should contain a candidate resource with a cid: identifier, and an additional search parameter using an Atom link:
```
也就是说.遇到这种情况,资源的id赋值为cid:id,另外在atom的link属性中增加一条如下		
```
<link href="http://localhost/Patient?[parameters]" rel="search"/>
```
服务器接收到这样的一条记录,首先依据href中那个的url和参数在服务器上进行检索有无匹配记录,如果有,
就直接使用服务器上的匹配记录,没有的话,就按原来的思路处理.如果发现多条记录,不进行处理,直接报异常.	
如下是patient的实例片段:	
```
<entry>
    <title>Patient details</title>
    <id>cid:patient@bundle</id>
    <updated>2014-05-28T22:12:21Z</updated>
    <link href="http://localhost/Patient?identifier=PRP1660" rel="search"/>
    <content type="text/xml">
        <Patient xmlns="http://hl7.org/fhir">
            <text>
                <status value="generated"/>
                <div xmlns="http://www.w3.org/1999/xhtml">Joe Bloggs</div>
            </text>
            <identifier>
                <value value="PRP1660"/>
            </identifier>
            <name>
                <text value="Joe Bloggs"/>
            </name>
        </Patient>
    </content>
</entry>
```
3.	利用mailbox节点,我们给方法2中的bundle添加一个MessagHeader资源,使用MessageHeader.event 来
表示bundle是用来干嘛的.mailbox就能够正常处理了.	
4.	利用transaction节点,也就是服务器根节点,通过profile资源来定义bundle的内容.在事务transaction过程中,
可以识别出profile,使用预定义的处理方法,可能包含了校验和资源的更新操作.	


方法2是一种自定义方法,需要发送方接受方的协调,FHIR中暂时没有定义和描述此类服务的方式.	
方法3 你要定义消息事件类型,添加消息头资源,发送方和接受方需要协调	
方法4POSTing to the root (#4) is attractive –provided that the server knows how to perform the ‘search processing’ (i.e. recognize the search link for a resource in the bundle and process accordingly. We could possibly use a profile so that the server can validate the bundle first.

<pre class="brush: java; title: ; notranslate" title="">
@WebServlet(urlPatterns= {&quot;/fhir/service/glucoseprocessor&quot;}, displayName=&quot;Process Glucose bundle&quot;)
public class GlucoseResultServlet extends HttpServlet {

    private MyMongo _myMongo;
    private FhirContext _fhirContext;

    @Override
    public void init(ServletConfig config) throws ServletException {
        //get the 'global' resources from the servlet context
        ServletContext ctx = config.getServletContext();
        _myMongo =  (MyMongo) ctx.getAttribute(&quot;mymongo&quot;);
        _fhirContext = (FhirContext) ctx.getAttribute(&quot;fhircontext&quot;);
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        PrintWriter out = response.getWriter();
        //first parse into a fhir bundle (need to check the mime type here. Should also check for _format paramters as well)
        //if we have to to this a lot, then a separate utility class would be good...

        Bundle bundle = null;
        IParser parser = null;
        String ct =  request.getHeader(&quot;content-type&quot;);
        if (ct.equals(&quot;application/xml+fhir&quot;)){
            parser = _fhirContext.newXmlParser();
            bundle = parser.parseBundle(request.getReader());
        } else if (ct.equals(&quot;application/json+fhir&quot;)){
            parser = _fhirContext.newJsonParser();
            bundle = parser.parseBundle(request.getReader());
        } else {
            OperationOutcome operationOutcome = new OperationOutcome();
            operationOutcome.getText().setDiv(&quot;Invalid content-type header: &quot; + ct);
            parser = _fhirContext.newXmlParser();
            response.setStatus(415);    //unsupported media type
            out.println(parser.encodeResourceToString(operationOutcome));
            return;
        }

        //now we have a bundle we can process it...
        GlucoseBundleProcessor glucoseBundleProcessor = new GlucoseBundleProcessor(_myMongo);
        //process the bundle, and get back the list of resources with updated ID's...
        List&lt;IResource&gt; resources = glucoseBundleProcessor.processGlucoseBundle(bundle);
        Bundle newBundle = new Bundle();
        newBundle.getTitle().setValue(&quot;Processed Glucose results&quot;);
        newBundle.setId(new IdDt(java.util.UUID.randomUUID().toString()));
        newBundle.setPublished(new InstantDt());
        for (IResource resource : resources) {
            newBundle.addResource(resource,_fhirContext,&quot;http://localServer/fhir&quot;);
        }

        response.setStatus(201);    //created
        out.println(parser.encodeBundleToString(newBundle));

    }

}

</pre>

The code is pretty well documented, and is certainly not intended to be production level code &#8211; just to show concepts. There&#8217;s stuff we had to do manually (like choosing the correct HAPI parser based on HTTP headers) that HAPI does automatically elsewhere &#8211; makes you appreciate the heavy lifting that it does!

&nbsp;

One thing we’ve not looked at is security. But – given that we’ve implemented OAuth2 as part of our SMART development, all we need to do is to ‘protect’ the root endpoint by requiring a valid Access Token to use the endpoint. Then, the client simply logs in before submitting a bundle.

So that’s a simple app to process a specific type of bundle. Bundles are a very useful mechanism in FHIR to manage complexity (especially transactional complexity) and reduce the number of REST calls required when there are multiple resources to create or update.

However, generic processing of bundles is quite complex – and specific use case processing will often be much simpler.

As with many things in FHIR, there’s more than one way to achieve a specific objective, so choose the one that best meets the requirements.
