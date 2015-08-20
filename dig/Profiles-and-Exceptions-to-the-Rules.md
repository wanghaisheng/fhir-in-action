[原文链接:Profiles and Exceptions to the Rules](http://www.healthintersections.com.au/?p=2348)

## Profiles and Exceptions to the Rules

profile 是 FHIR 中核心组件之一。一个 profile 是指在某个解决方案中如何使用或应该如何使用 FHIR 资源的说明。
FHIR 资源都是些通用型的组件，可以完成诸多通用操作，如在PHR中存储数据，临床病历的展现等。

但如果你想完成一些更加具体的操作，就需要限定内容。比如，如果你想利用输入的血糖和HBA1c测量值来实现一个决策支持的模块，告知患者糖尿病的控制情况如何。
如果要让患者、医疗机构能够很好的使用该模块，开发人员必须明确哪些输入的测量值是可用的，实际上，并非所有的数据是可用的。相反地，
如果临床信息系统想要使用此类决策支持的模块，就必须明确输入的到底是哪类血糖测量值。

如果该决策支持模块和临床信息系统同时都能产生 profile，系统管理员可能能通过自动化的比较来知道二者是否兼容，至少，我们期望是这样。

目前而言，我们只考虑规则本身，临床信息系统可能遇到如下的情况：
* 能够提供血糖测量值的数据流给决策支持模块
* 这些数据来源于多个地方-检验科室、住院病人的监控系统、可穿戴设备、科室里的检测设备
* 血糖测量值与临床信息系统之间隔着一到多个中间系统，比如诊断系统(医生工作站)、床旁护理系统(护士工作站)、居家护理系统等
* 每种测量值可能会使用一些LOINC编码
39480-9: Glucose [Moles/volume] in Venous blood, 41652-9: Glucose [Mass/volume] in Venous blood,
14743-9: Glucose [Moles/volume] in Capillary blood by Glucometer)
* 测量值的单位可能是mg/dL or mmol/L
* 具体的数字值可能会有一个大于或小于的比较符，比如>45mmol/L

如果你有一个profile 约定了这些方面的内容，决策支持模块就能够知道它拿到的数据是怎么样的，该如何处理。

看起来这样子是可行的，但事实上，负责集成数据的工程师会发现，拒绝接受的消息里大约优百万分之一的血糖测量值只包含了一个文本值，而非数值。
比如，“Glucose value too high to determine”。从临床安全的角度来讲，工程师是不能够简单地将“ too high to determine”替换成“>N”
,N 是某个随机选择的数字，怎么选这个数字都是错的。如果不能够使数据来源的系统更改自己的接口的话，又不能简单的丢弃此类数据，必须要有一个方案。

如果系统中的数据并不符合它所声称该符合的profile中的约束，应该如何来做？
* 1、系统隐藏此类数据，决策支持模块无法接收到此类数据
* 2、系统修改profile，指出除了数值也可能只发送纯文本值
* 3、系统将此类数据发送给决策支持模块，但将其标记成非法值。

从临床上看，第一种方案是不可能接受的。
第二种是良好的系统实践。但当系统拿到这百万分之一的文本值后如何理解其中的数据，如何处理？
第三种，可能最好的方式莫过于让profile声明那种是理想情况，那种是期望得到的，允许系统对不符合的数据进行标记，或者使用扩展来表示不符合的情况，这样子消费端系统才能够适时的忽略此类数据。
但前提是消费端系统得知道这样的一些标记。

That is, if they know about the flag, and remember. Which means we’d need to define it globally, and the standard itself would have to tell people to check for data that isn’t consistent with it’s claims… and then we’d have to add overrides to say that some rules actually mean what they say, as opposed to not actually meaning that…. it all sounds really messy to me.

Perhaps, the right way to handle this is to have ideal and actual profiles? That would mean an extension to the Conformance resource so you could specify both – but already the interplay between system and use case profiles is not well understood.

Lloyd McKenzie

I like the idea of the “ideal” and “actual” profiles. However, I’d tighten the definitions. “actual” = If the data doesn’t adhere to this, the system will either be incapable of sending it or will withhold sending it. “ideal” = As designed and based in prior experience, the system expects at least 99% of shared instances to adhere to the profile.




