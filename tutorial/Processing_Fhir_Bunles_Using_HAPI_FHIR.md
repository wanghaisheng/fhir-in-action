[原文链接:Processing FHIR Bundles using HAPI](http://fhirblog.com/2014/08/18/processing-fhir-bundles-using-hapi)

[另外一篇文章:FHIR transactions: the Search functionality](http://fhirblog.com/2014/06/13/fhir-transactions-the-search-functionality)

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
通用型的事务处理过程是很复杂的,一方面包括了ID的重写操作,另一方面也要照顾bundle中需要删除操作的资源.通过profle资源来约束限制事务的处理过程的行为,比方说,服务器识别出profile,将其作为某次transaction的基准,而不需要实现所有功能.

不论选取那种方式,都需要在响应时返回一个bundle,包含了各个资源,每个资源都会分配一个ID.
下面的代码是方式2的范例,仅供参考.
*	GlucoseBundleProcessor迭代所有资源,更新其cid:id的引用,核对是否存在Patient Device资源.
*	该例中忽略了所有服务器想要的资源 显然是不科学的

    public class GlucoseBundleProcessor {

    private MyMongo _myMongo;   //the database helper class

    public GlucoseBundleProcessor(MyMongo myMongo){
    _myMongo = myMongo;
    }

    //process with a Bundle...
    public List<IResource> processGlucoseBundle(Bundle bundle) {
    //generate a list of resources...
    List<IResource> theResources = new ArrayList<IResource>();//list of resources in bundle
    for (BundleEntry entry : bundle.getEntries()) {
    theResources.add(entry.getResource());
    }
    return this.process(theResources);
    }


    //process with a List of resources
    public List<IResource> processGlucoseUploads(List<IResource> theResources) {
    return this.process(theResources);
    }


    //process a bundle of glucose results, adding them to the repository...
    private List<IResource> process(List<IResource> theResources) {
    Patient patient = null; //this will be the patient resource. There should only be one....
    Device device = null;   //this will be the device resource. There should only be one....
    List<IResource> insertList = new ArrayList<IResource>();//list of resource to insert...

    //First pass: assign all the CID:'s to a new ID. For more complex scenarios, we'd keep track of
    //the changes, but for this profile, we don't need to...
    //Note that this processing is highly specific to this profile !!! (For example, we ignore resources we don't expect to see)
    for (IResource resource : theResources) {
    String currentID = resource.getId().getValue();
    if (currentID.substring(0,4).equals("cid:")) {
    //Obviouslym the base URL should not be hard coded...
    String newID = "http://myUrl/" + java.util.UUID.randomUUID().toString();
    resource.setId(new IdDt(newID));//and here's the new URL
    }

    //if this resource is a patient or device, then set the appropriate objects. We'll use these to set
    // the references in the Observations in the second pass. In real life we'd want to be sure there is only one of each...
    if (resource instanceof Patient) {
    patient = (Patient) resource;
    //we need to see if there's already a patient with this identifier. If there is - and there is one,
    //then we use that Patient rather than adding a new one.
    // This could be triggered by a 'rel=search' link on the bundle entry in a generic routine...
    IdentifierDt identifier = patient.getIdentifier().size() >0 ? patient.getIdentifier().get(0) : null;
    if (identifier != null) {
    List<IResource> lst = _myMongo.findResourcesByIdentifier("Patient",identifier);
    if (lst.size() == 1) {
    //there is a single patient with that identifier...
    patient = (Patient) lst.get(0);
    resource.setId(patient.getId());//set the identifier in the list. We need to return this...
    } else if (lst.size() > 1) {
    //here is where we ought to raise an error - we cannot know which one to use.
    } else {
    //if there isn't a single resource with this identifier, we need to add a new one
    insertList.add(patient);
    }
    } else {
    insertList.add(patient);
    }
    }

    //look up a Device in the same way as as for a Patient
    if (resource instanceof Device) {
    device = (Device) resource;
    IdentifierDt identifier = device.getIdentifier().size() >0 ? device.getIdentifier().get(0) : null;
    if (identifier != null) {
    List<IResource> lst = _myMongo.findResourcesByIdentifier("Device", identifier);
    if (lst.size() == 1) {
    device = (Device) lst.get(0);
    resource.setId(device.getId());//set the identifier in the list. We need to retuen this...
    } else {
    insertList.add(device);
    }
    } else {
    insertList.add(device);
    }
    }

    if (resource instanceof Observation) {
    //we always add observations...
    insertList.add(resource);
    }
    }

    //Second Pass: Now we re-set all the resource references. This is very crude, and rather manual.
    // We also really ought to make sure that the patient and the device have been set.....
    for (IResource resource : theResources) {
    if (resource instanceof Observation) {
    Observation obs = (Observation) resource;

    //this will be the correct ID - either a new one from the bundle, or a pre-existing one...
    obs.setSubject(new ResourceReferenceDt(patient.getId()));

    //set the performer - there can be more than one in the spec, hence a list...
    List<ResourceReferenceDt> lstReferences = new ArrayList<ResourceReferenceDt>();
    lstReferences.add(new ResourceReferenceDt(device.getId()));
    obs.setPerformer(lstReferences);
    }
    }

    //Last pass - write out the resources
    for (IResource resource : insertList) {
    _myMongo.saveResource(resource);
    }

    //we return the bundle with the updated resourceID's - as per the spec...
    return theResources;
    }
    }



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
    response.setStatus(415);//unsupported media type
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

    response.setStatus(201);//created
    out.println(parser.encodeBundleToString(newBundle));

    }

    }




