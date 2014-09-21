[原文链接:FHIR Connectathon 7 for Java Dummies](http://fhirblog.com/2014/07/31/fhir-connectathon-7-for-java-dummies/)

FHIR Connectathon 7中罗列了三个应用场景.
*	一是对Patient资源的处理  包括了新增,修改和查询等
*	二是对Profile资源的处理  包括了新增和校验等
*	三是基于FHIR的SMART APP的server端和client端的编程
*	四是对FHIR在DICOM中的应用演示		主要是DICOMweb协议与FHIR中ImageStudy资源的整合:
对于FHIR服务器而言,能够暴露DICOMWEB接口 提供ImageStudy资源 并能够将其转换为DICOM 模型的xml json格式;
对于DICOMWEB 服务器 构建出ImageStudy资源,暴露FHIR 的rest接口.

让我们看一下利用现有的FHIR开源代码如何来实现这些场景.
所选场景:第一个
开发语言:JAVA(安装配置略)
IDE:IDEA community版本
开源库:HAPI-FHIR
测试系统:UBUNTU14.04 32位

1.	新建maven工程,假设为FHIRConnectation7UseCase1,
2. 	在pom文件中按照[HAPI-FHIR官网[()上的说明进行配置即可	为了偷懒起见,我直接copy了[源码附带的例子中的pom](https://github.com/jamesagnew/hapi-fhir/blob/master/restful-server-example/pom.xml)
3.	将main类中的代码用如下代码替换

```
import ca.uhn.fhir.context.FhirContext;
import ca.uhn.fhir.model.api.Bundle;
import ca.uhn.fhir.model.dstu.resource.Conformance;
import ca.uhn.fhir.model.dstu.resource.Patient;
import ca.uhn.fhir.model.dstu.valueset.AdministrativeGenderCodesEnum;
import ca.uhn.fhir.rest.client.IGenericClient;
import ca.uhn.fhir.rest.server.exceptions.ResourceNotFoundException;

//Code copied very substantially from http://jamesagnew.github.io/hapi-fhir/doc_rest_client.html

public class Main {

    public static void main(String[] args) {

        //create the FHIR context
        FhirContext ctx = new FhirContext();
        String serverBase = "https://fhir.orionhealth.com/blaze/fhir";
        IGenericClient client = ctx.newRestfulGenericClient(serverBase);

        // Retrieve the server's conformance statement and print its description
        Conformance conf = client.conformance();
        System.out.println(conf.getDescription().getValue());


        //Read a single patient
        //String id = "1";        //this is not found on the server, and will throw an exception
        String id = "77662";
        try {
            Patient patient = client.read(Patient.class, id);
            //Do something with patient

            // Change the patient gender
            patient.setGender(AdministrativeGenderCodesEnum.M);
            //and save...
            client.update(id, patient);

        } catch (ResourceNotFoundException ex) {
            System.out.println("No patient with the ID of " + id);
        } catch (Exception ex){
            System.out.println("Unexpected error: " + ex.getMessage());
        }

        //Create a new Patient
        Patient newPatient = new Patient();
        newPatient.addIdentifier("urn:oid:2.16.840.1.113883.2.18.2", "PRP1660");
        newPatient.addName().addFamily("Power").addGiven("Cold");
        client.create()
                .resource(newPatient)
                .encodedJson()
                .execute();

        //do a simple search
        Bundle bundle = client.search()
                .forResource(Patient.class)
                .where(Patient.NAME.matches().value("eve"))
        .execute();
        //bundle will be a HAPI bundle of patients
        System.out.println(bundle.getTitle());
        System.out.println(bundle.getEntries().size() + " matches.");

    }
}
```




