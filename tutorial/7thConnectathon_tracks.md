[原文链接:FHIR Connectathon 7](http://wiki.hl7.org/index.php?title=FHIR_Connectathon_7)
## Connectathon tracks
**编者注:为了直观的了解目前FHIR标准的进展以及其现阶段能用来做什么，特地从wiki上摘录下来**
这部分介绍第7届connectathon中将要进行演示的场景.大致可分为两类，Track 1 是针对不熟悉FHIR的新手.  Track 2适合已经对FHIR有所了解的开发人员.

###  Track 1 - Patient

前置条件: 无

####  1. 注册一个新的patient

*   操作: (Patient Demographics consumer) 新建一个patient，调用Patient Service进行保存. 客户端可以为患者分配Id.

*   前置条件: 操作之前该Patient并不存在

*   成功条件: 服务器上成功创建该Patient(use browser to inspect Patient)

*   加分项: Patient资源包含扩展

&gt;&gt;备注: 客户端不一定非要给资源的ID赋值. 但如果是服务器分配的ID,要求客户端能够在服务器的响应获取该id .

####  2. 更新patient信息

*   操作: (Patient Demographics consumer) 更新了场景 #1中所创建的患者信息，调用Patient Service更新服务器端的信息. 通过id来调用patient资源.

*   前置条件: 服务器上已经创建了Patient资源

*   成功条件: 服务器上的Patient资源成功更新 (use browser to inspect Patient)

*   加分项 #1: 更新的患者资源中包含了扩展,但并不对扩展项进行修改 .

*   加分项 #2: 更新的患者资源中包含了扩展,同时对扩展项进行修改.

####  3. 检索Patient 历史记录

*   操作: (Patient Demographics consumer) 调用patient Service 查询Patient的历史记录

*   前置条件:  已经存在一个patient资源，并对其进行了至少一次更新

*   成功条件: 能够成功获得patient历史记录. (use browser query Patient Service)

*   加分项: UI上可以展示患者的历史记录

####  4. 根据姓名查询patient

*   操作: (Patient Demographics consumer) 调用patient Service，根据患者的given name来查询患者

*   前置条件: 服务器上存在Patients，且name字段有值

*   成功条件: 界面上展示出patients(use browser query to confirm)

参考资料:

*   [Java客户端实例](http://fhirblog.com/2014/07/31/fhir-connectathon-7-for-java-dummies/).

*   [.net 客户端实例](http://fhirblog.com/2014/06/29/c-fhir-client/).

###  Track 2 - Profile

分三个步骤:

*   生成 profiles 和 valuesets

*   创建服务端，用以测试一致性

*   实现测试一致性的过程(服务器端和客户端都可以)

这里也尝试使用了profile tags标签.

####  1. 创建 Profiles 和 Value Sets

只要能够生成Profiles and/or value sets就好了

*   创建一个profile and/or valuesets

    *   构建方法不限-可以手动编码，也可以从其他格式转换得到，以可以使用某些编辑工具

*   profile应包含的属性:

    *   Profile on Observation

        *   固定observation.name为某些LOINC code
        *   固定observation.value为某个类型
        *   固定observation.value.units
        *   固定一到多个属性的基数
        *   固定reliability 和 status的值

    *   将构建好的profile提交个profile库

        *   取保profile库中的profile/value set 引用准确

    *   创建一个可以通过和失败的资源实例，利用FHIR中提供的校验工具根据profile来校验它们 (可以从FHIR DSTU中下载)


####  2. 创建服务器，供测试一致性之用

*   已经有了一些observation resources实例 (zip file to be posted here)

*   已经创建好了一个profile (see below)

*   调用服务器上的校验操作，对提交的资源进行校验

    *   并不规定如何提交资源
    *   优先使用该服务器 [http://fhir.healthintersections.com.au/open](http://fhir.healthintersections.com.au/open) 也可使用如下#3中的一些服务器
    *   校验操作要求profile中的tag为profile所在的的服务器完整URL
    *   客户端必须能够正确处理响应 (成功 | 失败)

*   the profile master is here (link to be provided). Consult the server administrator for the correct profile tag for the test profile

####  3. 服务器校验 Validation

*   host a Profile and value set registry

*   实现校验操作 (只需在资源类型层面，无需到资源实例层面)

*   在校验过程中正确处理profile tags
*   根据可标识的profile正确的校验提交的资源
*   成功|失败的结果必须与FHIR校验工具的结果保存一致

###  Track 3 - SMART on FHIR

实验性质，关注面向用户的可在EHR/PHR系统界面中启动/调用APP. SMART on FHIR使用开放标准(FHIR, OAuth2, OpenID Connect)来构建了一个医疗APP的平台，这个平台能够与现有的医疗信息系统进行集成.

*   大概情况请参考 [http://docs.smartplatforms.org/](http://docs.smartplatforms.org/)

*   有问题请访问[SMART on FHIR Google Group](https://groups.google.com/forum/#!forum/smart-on-fhir).

####  1. 构建SMART服务器

一个服务器需要实现如下功能:

1. 支持 <tt>/Patient</tt> and <tt>/Observation</tt> end points,
APP可以获取人口统计学和生命体征信息.

2. 支持SMART on FHIR的启动和授权，APP可以获得许可，利用OAuth2得知患者的语境信息.

**[SMART servers快速入门](http://docs.smartplatforms.org/tutorials/server-quick-start/)**
，其中提供了URL、参数、LOINC编码和数据负载的实例.

*   其他参考资料
<dl>
<dd>[http://fhirblog.com/2014/08/02/smart-on-fhir-part-1/](http://fhirblog.com/2014/08/02/smart-on-fhir-part-1/)
</dd>
<dd>[http://fhirblog.com/2014/08/12/smart-on-fhir-adding-oauth2/](http://fhirblog.com/2014/08/12/smart-on-fhir-adding-oauth2/)
</dd>
</dl>

####  2. 构建 SMART App

客户端可以是web app或者是mobile app，可以在SMART on FHIR testing server上运行. 应满足如下功能:

*   能够与SMART's [sandbox server](https://fhir-api.smartplatforms.org) 通信

*   通过授权获取患者数据

*   获取诸如人口统计学、体征和实验室检查检验等数据

*   提供一些患者数据的展示，最简单的表格也可以

**[SMART apps快速入门](http://docs.smartplatforms.org/tutorials/testing/)**包含了一些代码和范例.

如需更多信息，可查看源代码
[sample apps on GitHub](https://github.com/smart-on-fhir/apps/tree/gh-pages/static/apps).

最开始可以在本地进行开发 `[http://localhost](http://localhost)`
无需进行注册 --如果你想在网络上运行你的APP， 只需[按照app 注册指南操作即可](http://docs.smartplatforms.org/sandbox/howto/).

###  Track 4 - DICOMweb (Joint with DICOM)

[DICOMweb](http://dicomweb.hcintegrations.ca/#/home)是医学影像的web标准. 主要是一些RESTFUL服务接口，能够让web应用开发人员充分利用既有的工具来实现医学影像的功能.

尽管DICOMweb's REST接口和序列化格式与 FHIR不同, FHIR中的 [ImagingStudy](http://www.hl7.org/implement/standards/fhir/imagingstudy.html)是联合DICOM一起开发的，为的是能够让DICOM服务器利用FHIR ImagingStudy通过[FHIR REST api](http://www.hl7.org/implement/standards/fhir/http.html)暴露服务器上的数据,反过来，对于FHIR服务器而言，能够利用ImagingStudy Resources 通过DICOMweb's REST api暴露服务器上的数据.

利用[WADO-RS](http://dicomweb.hcintegrations.ca/#/wado):

*   对于FHIR servers: 通过DICOMweb's REST interface来暴露FHIR ImagingStudy资源,将ImagingStudy转换为[DICOM model](http://dicomweb.hcintegrations.ca/#/xml)的xml和或json 格式

*   对于DICOMweb servers: 将一系列study实例整合成一个FHIR ImagingStudy资源, 通过 FHIR's REST interface (basically, a [read](http://www.hl7.org/implement/standards/fhir/http.html#read) operation)暴露资源，将FHIR的ImagingStudy转换为[DICOM serialization model](http://www.hl7.org/implement/standards/fhir/xml.html)的 xml和或json格式 .
