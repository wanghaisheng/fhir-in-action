距离上次把玩hapi-fhir已经有半年了，版本也从0.7演进到HAPI FHIR 1.1。

这里使用IDEA。

注：由于FHIR spec本身并没有成为最终版，还是个试行版过程，这里HAPI-FHIR支持DSTU和DSTU2两个版本

特点
1、纯java
2、fluent 接口

```java
Patient patient = new Patient();
patient.addIdentifier().setUse(OFFICIAL).setSystem("urn:fake:mrns").setValue("7000135");
patient.addIdentifier().setUse(SECONDARY).setSystem("urn:fake:otherids").setValue("3287486");
 
patient.addName().addFamily("Smith").addGiven("John").addGiven("Q").addSuffix("Junior");
 
patient.setGender(AdministrativeGenderEnum.MALE);
```
3、支持JSON、XML

```java
FhirContext ctx = FhirContext.forDstu2();
String xmlEncoded = ctx.newXmlParser().encodeResourceToString(patient);
String jsonEncoded = ctx.newJsonParser().encodeResourceToString(patient);

```
4、易于构建rest客户端服务端

```java
public interface MyClientInterface extends IRestfulClient
{
  /** A FHIR search */
  @Search
  public List<Patient> findPatientsByIdentifier(@RequiredParam(name="identifier") IdentifierDt theIdentifier);
     
  /** A FHIR create */
  @Create
  public MethodOutcome createPatient(@ResourceParam Patient thePatient);
}
```

调用

```java
MyClientInterface client = ctx.newRestfulClient(MyClientInterface.class, "http://foo/fhir");
IdentifierDt searchParam = new IdentifierDt("urn:someidentifiers", "7000135");
List<Patient> clients = client.findPatientsByIdentifier(searchParam);
```



## 新手上路

### context
HAPI 为FHIR 中的每一种资源类型、数据类型定义了model class 模型类。比如[Patient](http://jamesagnew.github.io/hapi-fhir/apidocs/ca/uhn/fhir/model/dstu/resource/Patient.html)中你可以看到所有属性的get set方法。
FhirContext 是使用HAPI的入口点，就像所有其他API的工厂类一样。如果熟悉JAXB API的，FhirContext和JAXBContext 作用是十分类似的。

新建一个 FhirContext 只需要实例化一个即可，不同FHIR 标准的版本对应特殊的 FhirContext ，如下所示，你可以根据自己的系统想要支持
的FHIR版本来具体选择

```java
// Create a context for DSTU1
FhirContext ctx = FhirContext.forDstu1();
 
// Alternately, create a context for DSTU2
FhirContext ctxDstu2 = FhirContext.forDstu2();
```

### 如何将String反序列化成资源对象

一个context下可以创建多个[Parser](http://jamesagnew.github.io/hapi-fhir/apidocs/ca/uhn/fhir/parser/IParser.html)来解析消息。
处于性能上的考虑，推荐新增一个FhirContext之后不要销毁，保存起来 直至你的应用程序的整个生命周期。而Parser则很轻量级，可以重复使用。

```java
// The following is an example Patient resource
String msgString = "<Patient xmlns=\"http://hl7.org/fhir\">"
  + "<text><status value=\"generated\" /><div xmlns=\"http://www.w3.org/1999/xhtml\">John Cardinal</div></text>"
  + "<identifier><system value=\"http://orionhealth.com/mrn\" /><value value=\"PRP1660\" /></identifier>"
  + "<name><use value=\"official\" /><family value=\"Cardinal\" /><given value=\"John\" /></name>"
  + "<gender><coding><system value=\"http://hl7.org/fhir/v3/AdministrativeGender\" /><code value=\"M\" /></coding></gender>"
  + "<address><use value=\"home\" /><line value=\"2222 Home Street\" /></address><active value=\"true\" />"
  + "</Patient>";
 
// The hapi context object is used to create a new XML parser
// instance. The parser can then be used to parse (or unmarshall) the
// string message into a Patient object
IParser parser = ctx.newXmlParser();
Patient patient = parser.parseResource(Patient.class, msgString);
 
// The patient object has accessor methods to retrieve all of the
// data which has been parsed into the instance.
String patientId = patient.getIdentifier().get(0).getValue();
String familyName = patient.getName().get(0).getFamily().get(0).getValue();
String gender = patient.getGender();
 
System.out.println(patientId); // PRP1660
System.out.println(familyName); // Cardinal
System.out.println(gender); // M
```

### 如何将资源对象序列化成String
同样可以使用parser来完成序列化
```java
/**
 * FHIR model types in HAPI are simple POJOs. To create a new
 * one, invoke the default constructor and then
 * start populating values.
 */
Patient patient = new Patient();
 
// Add an MRN (a patient identifier)
IdentifierDt id = patient.addIdentifier();
id.setSystem("http://example.com/fictitious-mrns");
id.setValue("MRN001");
 
// Add a name
HumanNameDt name = patient.addName();
name.setUse(NameUseEnum.OFFICIAL);
name.addFamily("Tester");
name.addGiven("John");
name.addGiven("Q");
 
// We can now use a parser to encode this resource into a string.
String encoded = ctx.newXmlParser().encodeResourceToString(patient);
System.out.println(encoded);
```

### 采用fluent 接口 方式来序列化成json字符串
有关fluent 接口的讨论请在新浪微博中以该关键词搜索即可看到更多有趣的探讨。

```java
Patient patient = new Patient();
patient.addIdentifier().setSystem("http://example.com/fictitious-mrns").setValue("MRN001");
patient.addName().setUse(NameUseEnum.OFFICIAL).addFamily("Tester").addGiven("John").addGiven("Q");
 
encoded = ctx.newJsonParser().setPrettyPrint(true).encodeResourceToString(patient);
System.out.println(encoded);
```

## 如何在自己的项目中使用HAPI-FHIR
由于 FHIR 自身在不断调整，HAPI也是在不断调整，SNAPSHOT指的不是最终的稳定版。

### MAVEN用户
对于使用maven进行包管理的用户而言，只需要将hapi-fhir-base、hapi-fhir-structures-dstu两个jar包添加到pom文件中，比如，
目前官方发布的唯一正式的试行版是DSTU1，截至目前，DSTU2的的投票过程还没有结束。如果你的系统想要支持这个版本，在pom中添加如下行
这里的version值请根据情况调整，日前最新的为“1.2-SNAPSHOT”。

```java
<dependency>
   <groupId>ca.uhn.hapi.fhir</groupId>
   <artifactId>hapi-fhir-base</artifactId>
   <version>1.1</version>
</dependency>
<dependency>
   <groupId>ca.uhn.hapi.fhir</groupId>
   <artifactId>hapi-fhir-structures-dstu</artifactId>
   <version>1.1</version>
</dependency>
```

这里需要注意的是，由于hapi-fhir-structures-dstu指的是hapi对于fhir中资源和数据类型的实现，除了两个DSTU1、DSTU2版本，hapi还支持FHIR标准自带的实现，也就是hapi-fhir-structures-hl7org-dstu2，请按需选择。

### Gradle用户

请添加如下配置
DSTU1
```Gradle
compile 'ca.uhn.hapi.fhir:hapi-fhir-base:1.1'
compile 'ca.uhn.hapi.fhir:hapi-fhir-structures-dstu:1.1'
```
DSTU2
```Gradle
compile 'ca.uhn.hapi.fhir:hapi-fhir-base:1.1'
compile 'ca.uhn.hapi.fhir:hapi-fhir-structures-dstu2:1.1'
```

### ANDROID 用户

```
<dependency>
   <groupId>ca.uhn.hapi.fhir</groupId>
   <artifactId>hapi-fhir-android</artifactId>
   <version>1.2-SNAPSHOT</version>
</dependency>
```

```
dependencies {
    compile group: 'ca.uhn.hapi.fhir', name: 'hapi-fhir-android', version: '1.2-SNAPSHOT'
    }
```


### SNAPSHOT的使用
如果想使用不稳定版本，需要在pom中添加
```
<repositories>
   <repository>
      <id>oss-snapshots</id>
      <snapshots>
         <enabled>true</enabled>
      </snapshots>
      <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
   </repository>
</repositories>
```

或者在Gradle

```
repositories {
   mavenCentral()
   maven {
      url "https://oss.sonatype.org/content/repositories/snapshots"
   }
}
```

### 其他依赖的包 

#### 日志
HAPI 中使用SLF4J来记录日志，建议加入Loback包，更多请参考[](http://jamesagnew.github.io/hapi-fhir/doc_logging.html)

#### XML 解析和处理
XML解析和构建使用了JAVA StAX API，而Woodstox是StAX的一个实现库。
在启动之初，HAPI会打印一条日志显示使用的是哪个版本的StAX的实现，比如
```
08:01:32.044 [main] INFO  ca.uhn.fhir.util.XmlUtil - FHIR XML procesing will use StAX implementation 'Woodstox XML-processor' version '4.4.0'
```

尽管HAPI 中采用Woodstox，但理论上应该可用使用任何实现StAX接口的库。如果存在多个StAX的实现，可用如下设置来强制使用Woodstox。

```
System.setProperty("javax.xml.stream.XMLInputFactory", "com.ctc.wstx.stax.WstxInputFactory");
System.setProperty("javax.xml.stream.XMLOutputFactory", "com.ctc.wstx.stax.WstxOutputFactory");
System.setProperty("javax.xml.stream.XMLEventFactory", "com.ctc.wstx.stax.WstxEventFactory");
```

#### XML的schematron校验

如果你要使用schematron validation的模块，则需要将Phloc加到classpath中来；


### 可运行的demo

为了简便起见，这里直接拿源代码中提供jpa-server来说明，实际项目中，根据我们自己的数据库、已有的信息系统，我们可以使用RestfulServer来实现一个FHIR 服务器。
这个小demo中使用单张表来保存资源内容(以CLOB行驶，可以使用GZIPPED来节省空间)，其他一些表来保存查询索引、tag、历史记录等。(后续我们可以考虑使用真实的比如军卫一号的数据结构或者FHIRBASE的NOSQL存储方案)
源代码[hapi-fhir-jpaserver-example](https://github.com/jamesagnew/hapi-fhir/tree/master/hapi-fhir-jpaserver-example),这个例子包含了一个完整的FHIR服务器，支持所有的标准操作read、create、delete等，使用内嵌式的apache derby java数据库。

```
$ git clone https://github.com/jamesagnew/hapi-fhir.git
$ cd hapi-fhir-jpaserver-example
$ mvn install
```
你可以将war包发布到 Tomcat/JBoss/Websphere/等服务器中，也可以直接使用内置的jetty
```
$ mvn jetty:run
```
然后访问http://localhost:8080/hapi-fhir-jpaserver-example

这时候如果你使用postman或curl等调试工具，例如这里选择curl，查询一下刚才部署成功的本地服务器上有没有patient
在命令行中键入
```
curl  http://localhost:8080/hapi-fhir-jpaserver-example/base/Patient
```
返回结果
```
edwin@edwindeMacBook-Pro ~/workspace/fhir-in-action (master)$ curl  http://localhost:8080/hapi-fhir-jpaserver-example/base/Patient

{
    "resourceType":"Bundle",
    "id":"091dc9de-00fb-4a93-b201-6c91c97e4b05",
    "meta":{
        "lastUpdated":"2015-08-20T11:07:10.156+08:00"
    },
    "type":"searchset",
    "total":1,
    "link":[
        {
            "relation":"self",
            "url":"http://localhost:8080/hapi-fhir-jpaserver-example/base/Patient"
        }
    ],
    "entry":[
        {
            "resource":{
                "resourceType":"Patient",
                "id":"1",
                "meta":{
                    "versionId":"1",
                    "lastUpdated":"2015-08-20T10:55:50.183+08:00"
                },
                "text":{
                    "status":"generated",
                    "div":"<div><div class=\"hapiHeaderText\"><b>TESTCREATERESOURCECONDITIONAL </b></div><table class=\"hapiPropertyTable\"><tbody></tbody></table></div>"
                },
                "name":[
                    {
                        "family":[
                            "testCreateResourceConditional"
                        ]
                    }
                ]
            },
            "search":{
                "mode":"match"
            }
        }
    ]
}%  
```

继续查询这唯一一条记录的具体内容话
```
curl  http://localhost:8080/hapi-fhir-jpaserver-example/base/Patient/1

```

得到结果
```
edwin@edwindeMacBook-Pro ~/workspace/fhir-in-action (master●)$ curl  http://localhost:8080/hapi-fhir-jpaserver-example/base/Patient/1

{
    "resourceType":"Patient",
    "id":"1",
    "meta":{
        "versionId":"1",
        "lastUpdated":"2015-08-20T10:55:50.183+08:00"
    },
    "text":{
        "status":"generated",
        "div":"<div><div class=\"hapiHeaderText\"><b>TESTCREATERESOURCECONDITIONAL </b></div><table class=\"hapiPropertyTable\"><tbody></tbody></table></div>"
    },
    "name":[
        {
            "family":[
                "testCreateResourceConditional"
            ]
        }
    ]
}%           
```
#### 整体架构

* Resource provider 对于某个版本的FHIR， 每个资源类型都会有一个[Resource Provider](http://jamesagnew.github.io/hapi-fhir/doc_rest_server.html#resource_providers)
其中@Search方法实现了FHIR 标准中定义了该资源类型的所有查询参数。该provider继承了IResourceProvider超级类，其中实现了所有的FHIR 方法，如read、create、delete、update等
注意，这些provider是在HAPI build编译过程中生成的，所以源代码中是看不到的。可以通过[JXR report](http://jamesagnew.github.io/hapi-fhir/xref-jpaserver/)来查看源代码
provider 中并不会实现任何查询、更新逻辑，只是接收HTTP 调用请求，并将请求传递给对应的DAO

* HAPI DAO：DAO负责具体的实现所有数据库相关的业务逻辑，包括 FHIR 资源的存储、索引和检索，使用的底层的JPA API

* Hibernate：HAPI JPA Server使用Hibernate中实现的JPA 库，没有用到任何只针对Hibernate的功能，应该也能支持其他库比如Eclipselink。

* database：该demo使用的内嵌式的Derby数据库，但可以使用任何hibernate支持的数据库。
具体可以通过修改hapi-fhir-server-database-config.xml文件中的配置项来实现


