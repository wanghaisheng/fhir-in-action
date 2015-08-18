开放数据专题
1、立法推动usa 政府放开更多数据
2、epic和CommonWell之争
3、API能否拯救HIT
4、HIT OpenAPI之面面观

##  1、立法进一步推动USA政府开放更多数据—[SGR repeal opens doors to big data](http://www.medicalpracticeinsider.com/best-practices/how-sgr-repeal-law-promises-supercharge-data-analysis)

在2015年4月16号，一个因废止了Medicare中过时的根据可持续增长率来计算医生收入的公式而出名的Medicare Access and CHIP Reauthorization
Act (MACRA)东东正式成为了法律，有人称此举给Affordable Care Act法案的Medicare Data Sharing Program注入了一剂强心剂，更有人称其中的105章节要求政府将更多的
claim(医保索赔)数据开放给准入机构，更是给中小型医疗机构带来了春天。
```
1、由于平价医疗法的存在，HHS也就是美国卫生部公开了一部分Medicare claim数据给准入机构，这些机构利用医保数据和商业化数据进行数据分析，来获取
如何更好的达到要求的医疗质量和效率等指标来指导实践。由于此举风险甚高，议会持保守态度，但很快大家都意识到这样做远远不够，故在MACRA的105章节中放开更多数据

2、其中要求HHS Secretary不仅开放Medicare数据，也开放Medicaid和CHIP数据。这样子系统中的医保数据将会翻倍，能够支持更加复杂的数据分析。

3、尽管开放多少数据，开放不开放的决定权握在了HHS Secretary手里，但还是有很大希望的，毕竟Medicare的医保数据已经开放了2年，如果够快的话，Medicare每年应该能够开放

```
### 背景资料

ACA平价法案
```
美国总统奥巴马上任后积极推动医保改革，去年立法的《平价医疗法》当地时间12日却被上诉法院裁定违宪，令医保改革前路茫茫。分析指，表面上，共和党明显打赢一仗，但由于法院仅裁定强制参保条款违宪，其余部分仍然有效，最大输家其实是医保公司，民主党不但未必介怀，还可能对裁决乐观其成。

据报道，目前美国约有5000万人没有基本医疗保险，医院及纳税人被迫每年代为支付高达430亿美元(约合人民币2749亿元)。民主党控制的国会去年3月通过立法，规定由2014年起，美国所有18岁或以上的民众必须终生购买保险，否则会遭到税务惩罚，政府则会给予无力付款的民众津贴。

据介绍，强制参保条款是奥巴马医改法案的重点，能使3200万未投保的民众被纳入医保“保护伞”下，令美国全国医保覆盖率提升至95%，并要求保险公司接受已患病的民众投保。

分析认为，美国民主党其实并非很支持强制投保，最支持的是可赚大钱的保险公司。保险公司要求全民强制参保，才愿意接受早已患病的投保人。民主党当初为平息保险公司的反对声音，支持全民投保只是权宜之计。若该条款最后被删除，其余部分仍有效，保险公司将被迫继续接受患病者的投保，民主党也可达成全民医保的目标，民主党何乐而不为？
```
需求驱动的数据开放计划
```
https://github.com/demand-driven-open-data
Tools and methods that provide a systematic, ongoing and transparent mechanism to tell data owners what's most valuable to you
```

美国政府的开放数据集
```
自从Data.gov开放以来，美国已经公开了超过八万五千个政府数据集并提供免费下载。现在杀出了一个公司叫做enigma，把多个国家的开放数据整合索引，
提供开放数据的搜索服务。
enigma: https://app.enigma.io 纽约时报:http://www.nytimes.com/2014/03/16/technology/a-harvest-of-company-details-all-in-one-basket.html?_r=0
```
开放数据计划
```
https://project-open-data.cio.gov/
https://github.com/project-open-data
```
## 2、epic和CommonWell之争

### 背景介绍
主角：
Epic
```
1、参考dr2的[新文章](http://ww2.sinaimg.cn/bmiddle/be7d1ecagw1erwsfgwnlqj20c84mob29.jpg)
2、[美国的The top 100 EHR companies (Part 1 of 4)前一百](http://medicaleconomics.modernmedicine.com/medical-economics/content/tags/top100ehrs/top-100-ehr-companies-part-1-4)
```

主角：
CommonWell联盟(Cerner, McKesson, athenahealth and Greenway四大公司，亦可参考上面的top100了解其大概的规模)

```
1、参考http://www.commonwellalliance.org/news/commonwell-health-alliance-announces-member-expansion-plans-for-2015/
2、参考http://www.commonwellalliance.org/
CommonWell Health Alliance today announced the commitment of five of its health IT vendor members—athenahealth, Cerner, CPSI, Greenway Health and McKesson—to actively deploy CommonWell services to health care provider sites nationwide throughout 2015.

Already, more than 60 provider sites are live on CommonWell services across 15 states including: Alabama, Delaware, Florida, Illinois, Indiana, Kansas, Massachusetts, Mississippi, Nebraska, North Carolina, Ohio, South Carolina, South Dakota, Texas and Washington. CommonWell expects 2015 deployments to enable at least 5,000 provider sites to be live on the services nationwide. CommonWell’s built-in services include patient identification, record location, patient privacy and consent, and trusted data access.
```

### 缘起
2014年7月17号，在[众议院能源和商务委员会的小组委员会通讯技术和健康 House Energy and Commerce Committee's subcommittee on Communications and Technology and Health](http://energycommerce.house.gov/press-release/subcommittees-team-learn-how-21st-century-technology-can-improve-21st-century-cures)的[听证会上](http://www.fierceemr.com/story/phil-gingrey-ehr-interoperability-fraud-against-american-taxpayers/2014-07-22
) ,从医生成为议员的Rep. Phil Gingrey 指责epic道：
```
一些电子病历的供应商的系统本身就在阻碍数据的共享和交换，就不应该拿到Meaningful Use incentive计划的钱。但根据RAND的一份报告，刺激计划的240亿美金中的一半以上都被
Epic公司拿走了，可Epic玩的是自己的封闭平台，这是HITECH的初衷么，纳税人的钱是这么花的么。

无独有偶，在RAND的报告中建议国会“取消那些需要额外组件、费用和定制才能够实现数据共享的医疗信息化产品的认证资格”。同期的一份研究中也指出Meaningful Use program 计划
第二阶段的数据共享的架构是不够健壮的，不足以支撑数据的共享。
```
原文如下
```
Electronic health record vendors--particularly Epic--may not deserve Meaningful Use incentive money because their systems hinder data sharing, according to physician-turned-lawmaker Rep. Phil Gingrey (R-Ga.).  

In a July 17 hearing of the House Energy and Commerce Committee's subcommittee on Communications and Technology and Health, Gingrey (pictured) questioned whether the nation is currently on a path of interoperability or whether changes to the law need to be made. He expressed concern that according to a recent RAND report, more than half of the $24 billion spent by the Meaningful Use program has gone to Epic, a vendor operating a "closed platform."

Pointing out that the committee has jurisdiction over the Office of the National Coordinator for Health IT and the HITECH Act--which created the Meaningful Use program--Gingrey said that if the RAND report is true, "we have been subsidizing systems that block information instead of allowing for information transfers, which was never the intent of the [HITECH] statute.

"It may be time for this committee to take a closer look at the practices of vendor companies in this space given the possibility that fraud may be perpetrated against the American taxpayer," he added.

The hearing was focused on how healthcare and technology can accelerate the pace of cures in the U.S., which is an initiative of the committee.

Gingrey is not alone in his concern about EHRs' lack of interoperability. The coalition Health IT Now, buoyed by the RAND report that decried the lack of interoperability among EHRs, recommended that Congress "decertify systems that require additional modules, expenses, and customization to share data," and to investigate business practices that prohibit or restrict data sharing in federal incentive programs.

Another recent study revealed that the current architecture for data exchange required by Stage 2 of the Meaningful Use program doesn't allow for robust patient sharing.  
```

### [ONC发布互操作性路线图interoperability roadmap](http://www.fiercehealthit.com/story/onc-interoperability-road-map-draft-outlines-governance-certification-stand/2014-10-14?utm_medium=nl&utm_source=internal)
关于这个10年的互操作性路线图,更多中文信息可参考

1、[ 【OMAHA】美国联邦政府医疗信息化战略规划2015 - 2020](http://mp.weixin.qq.com/s?__biz=MzA4NDAyNTU2NQ==&mid=204074676&idx=1&sn=d4d7b74c32d1486b79a95a0172005e5b&3rd=MzA3MDU4NTYzMw==&scene=6#rd)

2、[一张图看美国互操作性路线图](http://mp.weixin.qq.com/s?__biz=MzA5Mzg2MTMwMA==&mid=203956582&idx=4&sn=9f5ba195373b0acdec836a77af48118b&3rd=MzA3MDU4NTYzMw==&scene=6#rd)

3、[【国外】ONC发布HIT互操作路线图 助推精准医学计划](http://mp.weixin.qq.com/s?__biz=MzA5OTA2NDg4Mg==&mid=204722876&idx=4&sn=10475865952acb6137bdb1d48c815024&3rd=MzA3MDU4NTYzMw==&scene=6#rd)


某个组的老大如是说“如果我们能够让不同的系统能够互相'说话'，那已经是相当牛逼了”
原文如下
```
"If we just get the systems themselves to talk to each other, that would be a huge accomplishment," Health IT Policy Committee member Paul Egerman, former CEO of eScription, said at the meeting. "Perhaps we're making this harder than we need to make it, and it's already pretty hard."

尽管局部范围已经实现了一定程度的互操作性，但大规模的应用还是很大的问题。
while interoperability has increased, widespread interoperability remains a challenge. The report identifies several barriers to interoperability, including unchanged provider practice patterns, the lack of standardization among EHRs and the lower priority placed on EHRs by providers who are ineligible for the Meaningful Use program.
```

### [Omnibus bill keeps ONC funding at same level as 2014](http://www.fiercehealthit.com/story/omnibus-bill-keeps-onc-funding-unchanged/2014-12-15)
Omnibus法案成为导火索
```
The Omnibus Appropriations bill that passed Congress over the weekend will fund the Office of the National Coordinator for Health IT at the same level as last fiscal year with a budget of just under $60.4 million. After the bill passed the House last week, the Senate approved it Saturday night.

Additionally, the bill grants $14.9 million to the Federal Office of Rural Health Policy within the Health Resources and Services Administration to administer the Small Rural Hospital Improvement Grant program for quality improvement and adoption of health information technology. It also awards the VA $3.9 billion in IT funds, $200 million more than FY2014, including $344 million for the modernization of VistA and the development of an interoperable system with the Department of the Defense.

Of the total IT budget, $548.34 million is earmarked for IT development, modernization and enhancement.
```

[国会敦促ONC ：要专注在互操作性上](http://www.fierceemr.com/story/congress-onc-focus-interoperability/2014-12-16):

在国会通过2015 omnibus appropriation bill法案的同时提交的[报告](http://docs.house.gov/billsthisweek/20141208/113-HR83sa-ES-G.pdf)中，国会很生气，敦促ONC：
“”要合理使用你们手中的认证的权利来保障合格的产品能够医疗机构和人民群众带来价值。只给那些满足我们的认证标准的且没有阻碍区域医疗(医疗信息交换)的产品给予授权，对于阻碍信息共享的产品要收回执照。

```
"[U]se its certification program judiciously in order to ensure certified electronic health record technology provides value to eligible hospitals, eligible providers and taxpayers. ONC should use its authority to certify only those products that clearly meet current meaningful use program standards and that do not block health information exchange. ONC should take steps to decertify products that proactively block the sharing of information because those practices frustrate congressional intent, devalue taxpayer investments in CEHRT, and make CEHRT less valuable and more burdensome for eligible hospitals and eligible providers to use."
```

同时在报告里要求ONC90天内提交一份报告，分析一下目前信息阻塞问题的严重程度，要有个大概的厂商、医疗机构的数目以及解决问题的方案。

### [信息化厂商和医院都是信息阻塞的罪魁祸首](http://www.fiercehealthit.com/story/providers-vendors-both-blame-information-blocking/2015-04-10)

这里就是上面ONC给国会交的[作业](http://healthit.gov/sites/default/files/reports/info_blocking_040915.pdf)
作业中指出：
1、医院或厂商通过收费来控制转诊和加强自己的市场占有率
A common charge is that some hospitals or health systems engage in information blocking to control referrals and enhance their market dominance
2、厂商反馈 原因很多啦 比如说HIPAA 但作业中指出
都是借口 和HIPAA压根没多大关联
It has been reported to ONC that privacy and security laws are cited in circumstances in which they do not in fact impose restrictions

作业中给出的解决方案：

1、Strengthen in-the-field surveillance of ONC certified health IT tools, which was proposed in the certification requirements accompanying the Stage 3 Meaningful Use rule
2、Constrain standards and implementation specifications
3、Promote better transparency in certified health IT products and services that would "make developers more responsive to customer demands and help ameliorate market distortions" that lead to vendor information blocking
4、Establish governance rules deterring information blocking, which the report notes is addressed in ONC's interoperability roadmap
5、Work with the U.S. Department of Health and Human Services Office for Civil Rights to ensure healthcare stakeholders understand HIPAA privacy and security standards, particularly those related to information sharing
6、Coordinate with the HHS Office of Inspector General and the Centers for Medicare & Medicaid Services on information blocking as it relates to physician self-referral and the federal anti-kickback statute
7、Refer illegal business practices to law enforcement agencies
8、Work with CMS to provide incentives for interoperability while simultaneously discouraging information blocking
9、Promote competition and innovation in health IT, which the Federal Trade Commission recommended in its comments on the interoperability roadmap

但看了总结就知道 美帝也是各种部门互相扯皮呀，这种东西往后10年看有起色不
```
"While important, these actions alone will not provide a complete solution to the information blocking problem," the report's authors say. "A comprehensive approach will require overcoming significant gaps in current knowledge, programs and authorities that limit the ability of ONC and other federal agencies to effectively target, deter and remedy this conduct, even though it violates public policy and frustrates congressional intent.
```

### [口水仗正式开始](http://www.healthcareitnews.com/news/epic-vs-commonwell-showdown)

Senate HELP Committee Tuesday 2015年3月17日的组委会例行会议上，
在听证会的问答环节，参议员Senator Tammy Baldwin, D-Wisconsin问 为什么Epic 没有参加CommonWell Health Alliance
Epic  的主管互操作性的头Peter DeVault说了如下一段话，引发了CommonWell Health Alliance的反击：
```
"When we were approached by them and asked to join, we were told that it would be multiple millions of dollars for us to join and that we would have to sign an NDA.To us, the only reasons to have an NDA are if they're going to tell you something that otherwise they wouldn't want people to know" - which he said could include the possibility they may sell data downstream or wanted to ensure there were no intellectual property conflicts.That lack of transparency didn't sit well with us
```
同时还不饶人的嘲讽 你丫们只是嘴上说说 雷声大雨点小
CommonWell至今成立2年，目前只涵盖了1000个医生 和Epic的care Everywhere的10万医生简直高下立判
```
DeVault also underscored CommonWell's low participant numbers in its interoperability project so far – which he called an "aspiring network," noting a stark difference when compared to Epic's Care Everywhere network.
```
CommonWell联盟随后在Healthcare IT News发表了联合声明：
```
We are committed to openness and transparency," the statement read. "Accordingly we publish our services and use case specifications, along with our nominal membership and service fees on our website for everyone to see.
```
那么Epic要入会 大概要每年缴纳多少钱呢  根据目前CommonWell会员费和服务费用标准，结合Epic2014年的年收入18亿美元，要掏
125万美元annual subscription fee和 $50,000 to $90,000 in annual membership dues 。


周三晚上 就是3.18日  athenahealth CEO Jonathan Bush在twitter上对Epic CEO Judy Faulkner 进行了高调嘲讽
![](bb-epic-ceo.png)

Cerner. 行业老二也发话了
```
His "rhetoric is a slap in the face to many parties working to advance interoperability," according to a statement released by Cerner officials shortly after the committee hearing. "It was discouraging to hear more potshots and false statements when it's clear there is real work to be done. We're committed to CommonWell as a practical, market-led way to achieve meaningful interoperability
```

梗在这里 2013年HIMSS上Epic and CommonWell就有口角
```
At that HIMSS13 announcement, athenahealth CEO Jonathan Bush 说 大家都可以加入哟
emphasized that anyone was invited to CommonWell – even a vendor of "epic proportions."

In a subsequent interview, Healthcare IT News asked McKesson CEO John Hammergren whether that invitation was dangled only because the founding companies suspected Epic wouldn't bite.

"We'd like them to bite! We want them to bite," said Hammergren. "I'm hopeful that they will see it the same way we see it."
```
但 Epic CEO说 完全不知情,这帮sb居然想背后地里阴我们，手够黑的啊
```
But Epic CEO Judy Faulkner noted that her company wasn't asked to join before the announcement. "We did not know about it. We were not invited," Faulkner told Bloomberg back in 2013. "It appears on the surface to be used as a competitive weapon, and that's just wrong. It's wrong for the country."
```

对于行业内愈演愈烈的竞争所引发这场闹剧，围观群众也有话说：
1、各位看官和Epic签合同要慎重啊 且要是Epic-to-non-Epic能达到Epic-to-Epic 的90%的功能，"哥就表演吃翔"
2、路人乙嘲讽到Epic到底是如何腆着脸皮说all of our systems can talk to each other
3、入会费是不是有点多
4、Epic does not need to join CommonWell. CommonWell needs to join Epic.
5、CommonWell sounds like a group of union thugs who want to strong-arm their way into other vendors' businesses
6、CommonWell or is it CommonHell? Later seems to be more appropriate.此人称Epic很好合作 方案也性价比高Stop creating mindless standards organizations with a scheme to collect additional revenues from licensing. Healthcare organizations dole out enough on their EMRs to these vendors and they don't need these vendors charging nickel and dime on every petty thing that gives patients control of their health information
"我深深的表示赞同"
7、It just doesn't seem right that a private company pulling down $1.8B in revenue should be able to dictate interoperability. The government needs to step in and make sure all of these companies play well in the sandbox. Greed stands in the way of a national clinical data repository that researchers could use to find treatments and cures for diseases that have plagued us for decades.
问题不存在一个厂商上 此人观点有理
8、Joining CommonWell doesn't seem like a great business opportunity if after two years they only have 1000 doctors live on it. Epic seems to understand the fundamentals of sharing patient data if they have 100,000 doctors on their version. My guess is that CommonWell is floundering and they want Epic in there to help them out. If they can also take some digs at Epic for not joining in the meantime - all the better in their mind.

The healthcare market will drive the need for interoperability. I don't think getting the government involved will help anything.

9、What would be interesting to know is how many of Epic's 100,000 physicians are using a product other than something in the Epic family.
Epic excels at interoperability between their own products, but what is their track record when connecting to someone outside the family, so to speak? Their numbers may be impressive, I don't know. But I do know that the huge challenge CommonWell has is interoperability across disparate vendors and products.
10、It seems to me that CommonWell is primarily a marketing "gimmick". Aids and standards for inter-operability e.g. LOINC, CLSI standards, and other accepted nomenclatures have existed in the marketplace for some time. We probably have enough standards but many vendors simply pay "lip service" to them. Such a "collaboration" among competitors may be a "surface phenomenon" with relatively little substance.

11、对第10条的回复 CommonWell is a response to Epic's market share and market power by vendors that heretofore did not feel compelled to cooperate with each other on data exchange. Epic dramatically dominates the large HCDO/ Academic Medical Center market and is viewed as almost unstoppable. As Epic conquers the large market, it will target the mid-size market - probably taking advantage of a cloud service at some point in time. This probably best explains why Epic doesn't want to join CommonWell - they understand the strategic alignment and competitive response that CommonWell represents.
12、It does seem a bit childish to see these types of exchanges between behemoths of the industry. However, I do wonder why Epic would have to sign an NDA to participate in a joint effort to share and propagate openess.
### 后续报道 [Epic 投降了](http://www.healthcareitnews.com/news/epic-latest-drop-fees-data-exchanges)

Epic CEO Judy Faulkner 在每年度最盛大HIT行业会展，也就是2015年的HIMSS15大会上宣布，直到2020年之前 咱大EPIC是不会再收大伙钱了。其实athenahealth和Cerner这俩货也是不久之前宣布他们也不收费


群众观点：
1、up-front predictable cost and a pay-per-use model两种付费模式的区别而已，钱总是会出的
2、市场的不成熟导致没有有效的激励方式来促使信息的无缝免费流动。而能因为厂商收取服务费用就认为是某个厂商作恶。要解决的是到底谁来掏钱的问题
3、不额外收钱的话，，软件的成本、维护费咨询费必然要上涨，羊毛出在羊身上
http://www.healthcareitnews.com/news/epic-latest-drop-fees-data-exchange



### 思考
国内最早喊互操作性的应该是李包罗老爷子了，从最初引入这样的概念至今大概也有10来个年头了，不管是学术界还是工业界，这些年来沿着的一条路是说想要给复杂的医疗领域定义出一种通用的one-fit-all的能够描述整个医疗领域的模型，进而演化出一种可以用于不同系统间交互数据的通用格式和统一的医疗术语，也就是所谓的互操作性标准。这些年也有人在提学习型的医疗信息化系统的建设，提精准医疗，大喊互联网医疗的口号，但这些目标的实现都离不开数据，离不开鲜活的在不同系统间流动的数据，就像阿里、京东、腾讯、amazon一样，没有各部门间数据的无缝整合，它们是不能提供大家每天都在享受的这些服务的。近几年不管是google还是豌豆荚，都在说一个“应用内搜索”的东西，说白了，就是想把接入各自平台的系统的数据格式标准化，这样子就可以点开具体的应用来搜索查询信息，但似乎也没有什么进展，对于如日中天的互联网应用来讲，对于宇宙第一的google来讲尚且很难突破的东西，反过来看医疗行业，业务只会更复杂，业务系统的成熟度只会更低，似乎互操作性的口号，我们喊得太早了，能从美帝、互联网厂商身上学到的是，未来势必在国内的HIT圈要成长出一两家如Epic这样的公司，在自己封闭的平台中，实现各级医疗机构的信息无缝流通，只有这样的巨无霸，通过自己所收集的患者全生命周期的数据链，利用医疗大数据的应用打造出优质的医疗服务来服务大众，但有没有其他实现互操作性的路径呢?

### [Epic和IBM的合作不加入是另有所谋](http://www.healthcareitnews.com/news/epic-watson-work-interoperability)
Epic与IBM watson的合作 是否预示着实现互操作性的另一条路  智能的解析各种异构的数据源中的数据。


### 3、API能否拯救HIT
Jason and the Argonauts
http://geekdoctor.blogspot.nl/2014/12/kindling-fhir.html
http://xmlmodeling.com/2014/12/jason-argonauts/
http://xmlmodeling.com/2014/10/jason-task-force-final-report/
http://argonautwiki.hl7.org/index.php?title=Main_Page


### [Apps and APIs: A Positive Step for Patients](http://www.healthwise.org/insights/healthwiseblog/lkhall/may-2015/2496.aspx)



### OMAHA联盟 后感
http://getmyhealthdata.org/ 请愿书
