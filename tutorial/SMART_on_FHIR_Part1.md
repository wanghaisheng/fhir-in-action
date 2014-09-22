[原文链接:SMART on FHIR: Part 1](http://fhirblog.com/2014/08/02/smart-on-fhir-part-1)

[另外一篇文章:SMART on FHIR – adding OAuth2英文版](http://fhirblog.com/2014/08/12/smart-on-fhir-adding-oauth2)

[另外一篇文章:SMART on FHIR – adding OAuth2中文版](SMART_on_FHIR_adding_OAuth2.md)

[第七届connectathon](http://wiki.hl7.org/index.php?title=FHIR_Connectathon_7 "FHIR_Connectathon_7")眼看着就要开始了, 之前的[一篇里](FHIR_Connectathon7_for_Java_Dummies.md)我们讨论了其中第一个场景(Patient的访问)，其中利用一些开源库进行了简单的实现(btw 你完全可以不使用这些开源框架，使用自己熟悉的平台和编程语言即可).

这篇主要探讨一下第三个场景如何实现[SMART on FHIR](http://wiki.hl7.org/index.php?title=FHIR_Connectathon_7#Track_3_-_SMART_on_FHIR "FHIR_Connectathon_7#Track_3_-_SMART_on_FHIR"). 对于这个场景具体的描述请参考[7thConnectathon_tracks场景说明](7thConnectathon_tracks.md) – 其核心是构建一个标准，不同的独立开发APP/应用程序能够安全的访问存储在任何支持这种标准的服务器上的数据-电子病历、患者门户、区域平台相当于这里的应用程序，而类如苹果的HealthKit就是服务器，而苹果定义的HealthKit的接口就是我们所说的标准，这个比方打的不太恰当，但大致上是这么个意思。



第一步，我们先实现一个简单的服务器，主要是参考[Smart Server快速入门](http://docs.smartplatforms.org/tutorials/server-quick-start/), 如需深入了解，请细读该文档. 这篇文章中的服务器没有考虑安全性，后续的[SMART on FHIR adding OAuth2](SMART_on_FHIR_adding_OAuth2.md)中会详细介绍如何添加安全性.

这里假设我们的服务器是一个电子病历系统, 用户可以通过SMART团队所开发的[儿童成长曲线](https://fhir.smartplatforms.org/apps/growth-chart/launch.html?fhirServiceUrl=https://fhir-open-api.smartplatforms.org&amp;patientId=1482713)的APP来访问其中的数据.

最简单的做法，比如在我们的电子病历系统的界面上添加一个按钮“查看儿童成长曲线”.当用户点击该按钮:

1.  第一步，弹出一个界面，到这个页面上，指向儿童成长曲线的APP启动界面(类似你在Quaro上登录时选择使用google账号登录那样). 我们会把参数如患者ID还有APP所需要用到的数据以URL的形式暴露出去(FHIR中定义的Patient 和 Observation接口URL/endpoint)，
2.  启动界面在我们的界面上加载之后，通过ajax调用我们服务器上的FHIR endpoint来获取服务器上的数据 (安全性主要是在这一步中需要考虑).
3.  拿到数据之后，就可以进行展示，或者向服务器回写数据.

在这篇里面，我们要做如下内容:

*   一个HTML页面，用来模仿电子病历系统，当然也要有一个启动按钮
*   FHIR [Patient](http://www.hl7.org/implement/standards/fhir/patient.html) endpoint，能够返回患者的基本/人口统计学信息
*   FHIR [Observation](http://www.hl7.org/implement/standards/fhir/observation.html) endpoint，能够返回患者的生命体征等观察项的信息(如身高、体重、BMI等).

HTML页面很简单，加入如下的Jquery代码即可。 其他的内容需要大家自己动手。



	$(document).ready(function(){
	      //the url to launch the app (hardcoded patientID, and local server)
	      var url = "https://fhir.smartplatforms.org/apps/growth-chart/launch.html?";
	      url += "fhirServiceUrl=http://localhost:8080/fhir";
	      url += "&patientId=100";
	      //var iframe = &amp;amp;quot;&amp;amp;quot;; // this line should contain the html for an iframe, if wordpress would allow it...
	      //the jQuery handler to create an iFrame and launch the app in it
	      $("#launchApp").on('click',function(){
	        //$('#launchFrameDiv').append(iframe);
	        $("#launchIframe").attr("src",url);
	      })
	});



服务器端的话稍微要复杂一些。这里我们使用[HAPI开源库](http://jamesagnew.github.io/hapi-fhir/) 和Tomcat服务器.IDE的话选IntelliJ IDEA IDE.

利用HAPI构建服务器端,首先要为你所支持的每种资源定义一个[resource provider](http://jamesagnew.github.io/hapi-fhir/doc_rest_server.html#Defining_Resource_Providers) , 然后利用 [server class](http://jamesagnew.github.io/hapi-fhir/doc_rest_server.html#Create_a_Server)来发布服务 ；.

Patient Provider如下:

	public class PatientResourceProvider implements IResourceProvider {
	    //return a single Patient by ID
	    @Read()
	    public Patient getResourceById(@IdParam IdDt theId) {
	        Patient patient = new Patient();
	        Calendar cal = Calendar.getInstance();
	        cal.add(Calendar.YEAR, -18);         //18 years old
	        patient.setBirthDate(new DateTimeDt(cal.getTime()));
	        patient.addName().addFamily(&amp;amp;quot;Power&amp;amp;quot;);
	        patient.getName().get(0).addGiven(&amp;amp;quot;Cold&amp;amp;quot;);
	        patient.setGender(AdministrativeGenderCodesEnum.M);
	        return patient;
	    }
	}

为了简单起见，这里的赋值都是硬编码的形式，当然你可以使用数据库来set get.

Observation Provider如下:

	public class ObservationResourceProvider implements IResourceProvider {
	    @Search()
	    public List&amp;amp;lt;Observation&amp;amp;gt; getObservationBySubject(@RequiredParam(name = Observation.SP_SUBJECT) StringDt theSubject,
	                                                     @RequiredParam(name = Observation.SP_NAME) TokenOrListParam theObsNames) {
	        List&amp;amp;lt;Observation&amp;amp;gt; lstObservations = new ArrayList&amp;amp;lt;Observation&amp;amp;gt;();

	        //emulates calling an existing non-FHIR REST service and getting a JSON object back...
	        //we'd probably pass across the theObsNames to only return the list that we want
	        InputStream is = null;
	        try {
	            URL url = new URL(&amp;amp;quot;http://localhost:4001/dataplatform/obs/&amp;amp;quot;+theSubject);
	            URLConnection conn = url.openConnection();
	            conn.connect();
	            is = conn.getInputStream();
	            JsonReader rdr = Json.createReader(is);
	            JsonObject obj = rdr.readObject();
	            //assume that the json structure is {data[{id:,unit:,code:,value:,date: }]}

	            JsonArray results = obj.getJsonArray(&amp;amp;quot;data&amp;amp;quot;);
	            SimpleDateFormat format = new SimpleDateFormat(&amp;amp;quot;yyyy-MM-dd'T'HH:mm:ss&amp;amp;quot;);
	            for (JsonObject result : results.getValuesAs(JsonObject.class)) {
	                //get the basic data for the Observation
	                String id = result.getJsonString(&amp;amp;quot;id&amp;amp;quot;).getString();
	                String unit = result.getJsonString(&amp;amp;quot;unit&amp;amp;quot;).getString();
	                String code = result.getJsonString(&amp;amp;quot;code&amp;amp;quot;).getString();
	                String display = result.getJsonString(&amp;amp;quot;display&amp;amp;quot;).getString();
	                double value =  result.getJsonNumber(&amp;amp;quot;value&amp;amp;quot;).doubleValue();
	                Date date = format.parse(result.getJsonString(&amp;amp;quot;date&amp;amp;quot;).getString());

	                //create an Observation resource
	                Observation obs = new Observation();
	                obs.setId(id);
	                obs.setApplies(new DateTimeDt(date));
	                obs.setName(new CodeableConceptDt(&amp;amp;quot;http://loinc.org&amp;amp;quot;, code));
	                QuantityDt quantityDt = new QuantityDt(value).setUnits(unit).setSystem(&amp;amp;quot;http://unitsofmeasure.org&amp;amp;quot;).setCode(unit);
	                obs.setValue(quantityDt);
	                obs.setStatus(ObservationStatusEnum.FINAL);
	                obs.setReliability(ObservationReliabilityEnum.OK);

	                //the text...
	                obs.getText().setDiv(result.getJsonString(&amp;amp;quot;date&amp;amp;quot;).getString() + &amp;amp;quot; &amp;amp;quot; + display + &amp;amp;quot; &amp;amp;quot; + value);
	                obs.getText().setStatus(NarrativeStatusEnum.GENERATED);

	                //and add to the list...
	                lstObservations.add(obs);
	            }
	        } catch (Exception ex) {
	            System.out.println(&amp;amp;quot;Error during REST call &amp;amp;quot; + ex.toString());
	        }
	        finally {
	            try {
	                if (is != null) {
	                    is.close();
	                }
	            } catch (Exception ex){               //was going to throw an exception - but that seems to need to be in a try/catch loop, which seems to defeat the purpose...
	                System.out.println(&amp;amp;quot;Exception closing stream &amp;amp;quot;+ ex.getMessage());
	            }
	        }
	        return lstObservations;

	    }
	}
这个provider类似于一个代理，现有的FHIR服务器可能支持类似的功能，但接口不是FHIR形式的.对于系统的改造 大多数情况应该是这样的。

server servlet如下:


	@WebServlet(urlPatterns= {&amp;amp;quot;/fhir/*&amp;amp;quot;}, displayName=&amp;amp;quot;FHIR Server&amp;amp;quot;)
	public class OrionRestfulServlet extends RestfulServer {
	    public OrionRestfulServlet() {
	        List&amp;amp;lt;IResourceProvider&amp;amp;gt; resourceProviders = new ArrayList&amp;amp;lt;IResourceProvider&amp;amp;gt;();
	        resourceProviders.add(new ConditionResourceProvider(_myMongo));
	        resourceProviders.add(new ObservationResourceProvider(_myMongo));
	        resourceProviders.add(new PatientResourceProvider(_myMongo));
	        setResourceProviders(resourceProviders);
	    }
	}
剩下的就只需要打成war包，发布到tomcat中去即可. 输入URL，点击按钮，即可查看到生长曲线图!

后续的例子我们会使用OAuth2来给服务器添加安全保护。
