市面上已经存在多款具有预约挂号功能的APP，有第三方独立APP比如微医、挂号网，也有用微信公众号来做的，比如重庆市大坪医院。
有关门诊挂号、预约挂号的内容请参阅“北京协和医院：门诊多途径挂号方式及规则的探析”一文。
挂号一般分：
    - 初诊
        - 预约挂号
            - 号源平台预约挂号
            - 电话预约挂号
        - 当日挂号
            - 号源平台剩余号
    - 复诊
        - 预约挂号
            - 诊间预约
        - 当日挂号
            - 诊间加号
            
如果我们把范围仅仅限制在网上预约挂号的话，让我们一起来捋一捋其中相关的功能。

## 功能需求
### 大坪医院公众号

注：由于只支持复诊患者的网络预约，初诊患者需到医院内部办理就诊卡才行

* 1、根据一级科室来查找二级科室
* 2、根据二级科室来查找医生列表
* 3、根据医生信息获取排班信息
* 4、点击选择想要预约的医生及其时段，确认挂号时段、初诊/复诊类型，就诊对象信息，并提交预约申请
* 4a、选择支付方式，并完成支付(微信支付)
* 5、获取手机验证码，并提交验证码，确认预约申请
* 6、接收预约挂号成功提醒短信
后续的就诊直接到医院拿号，看诊顺序与拿到的号码为准

### 微医app

* 1、根据医院名称关键词、医院所在地市、综合排序(患者评价、预约量)、医院等级、候诊时间(快、非常快、较快、一般、不限)来搜索过滤医院
* 2、根据一级科室来查找二级科室
* 3、根据二级科室来查找医生列表
* 4、根据医生信息获取排班信息
* 5、点击选择想要预约的医生及其时段，确认挂号时段、初诊/复诊类型，就诊对象信息，并提交预约申请
* 5a、选择支付方式，并完成支付(到医院支付)
* 6、获取手机验证码，并提交验证码，确认预约申请
* 7、接收预约挂号成功提醒短信

### 掌上浙一

* 1、选择手机挂号
* 2、选择院区(庆春院区、海创园门诊部)，选择挂号类型(实时挂号、普通号预约、专家号预约)
* 3、选择科室，根据推荐标签(肝炎、妇科、消化科)选择
* 3a、选择科室，根据智能导诊，按照症状提示选择推荐科室，
* 3b、选择科室，根据一级科室查找二级科室
* 4、选择二级科室，确认挂号日期、时间段(上午、下午)，就诊人姓名，性别，手机号码，就诊卡号码(必填项 可以是浙一就诊卡、门诊部就诊卡、医保卡)，身份证号
提交挂号申请
* 5、确认挂号科室、就诊人信息、挂号费用，选择支付方式(先诊疗后结算)
* 5a、完成支付
* 6、完成挂号


大致上我们需要这几类信息
1、患者/就诊人信息
2、医院基本信息
3、医生信息
4、科室信息
5、医生排班信息
6、预约挂号申请信息
7、预约挂号短信确认
8、预约平台账号信息

## 准备工作

### 数据准备

1、推荐使用Binux大神的pyspider工具爬取挂号网上上海地区所有三甲医院的信息和医生以及排班信息
2、搭建FHIRBASE数据库作为后台存储
3、搭建军卫一号数据库作为备用测试库


### 开发环境

* JDK 1.7
* OSX 10.10.4
* IDEA
* HAPI-FHIR 1.2 SNAPSHOT
* POSTGRESQL
* postman
* curlX

## 数据需求分析

### 医院基本信息

| 字段名称 | 定义 | 示例值 |
| ---- | ----- | ----- |
| 医院名称 | ----- | 复旦大学附属华山医院  |
| 地址 | ----- | 上海市静安区乌鲁木齐中路12号  |
| 电话 | ----- |  021-52889999 |
| 官网 | ----- | http://huashan.org.cn/ |
| 简介 | ----- | 复旦大学附属华山医院是卫生部直属复旦大学（原上海医科大学）附属的一所综合性教学医院。建院于1907年，前身是中国红十字会总院，是上海地区中国人最早创办的医院，1991年重新恢复为中国红十字会直属医院。1992年首批通过国家三级甲等医院评审，目前已成为一所国家高层次的医疗机构，并为全国医疗、预防、教学、科研相结合的技术中心，在国内外 享有较高的声誉。 华山医院医疗技术力量雄厚，全院近1800名职工之中，医疗专业技术人员占80％，其中副高职以上专家教授290人，博士点10个，博士生导师39名，中国科学院院士、中国工程院院士各1名，开通博士后流动站2个；硕士点19个，硕士生导师79名。许多专家教授在国内外享有较高知名度。还有一整套行之有效的管理体制和管理人才。 |
| 医院等级  | ----- | 三级甲等 |
| 所属省份 | ----- | 上海 |
| 所属市县 | ----- | 黄浦 |
| 患者评价 | ----- | 12.5万 |
| 预约量 | ----- | 151.4万 |
| 导医服务 | ----- | 84%满意 |
| 候诊时间 | ----- | 74%满意 |
| 医院图片 | ----- | http://img.guahao.cn/images/hospital/hlarge_125336754304601.gif |



### 科室基本信息

| 字段名称 | 定义 | 示例值 |
| ---- | ----- | ----- |
| 科室类型 | ----- | 一级科室  二级科室 |
| 科室名称 | ----- | 心内科 |
| 科室简介 | ----- | ----- |
| 所属医院 | ----- | 上海市复旦大学附属华山医院 |
| 所属一级科室 | ----- | 内科 |
| 是否特色科室 | ----- | 否 |
| 科室人数 | ----- | 13 |


### 医生基本信息

| 字段名称 | 定义 | 示例值 |
| ---- | ----- | ----- |
| 所属医院ID | ----- | ----- |
| 所属医院名称 | ----- | 上海市复旦大学附属华山医院 |
| 所属一级科室名称 | ----- | 内科 |
| 所属一级科室编码 | ----- | ----- |
| 所属二级科室名称 | ----- | 心内科 |
| 所属二级科室编码 | ----- | ----- |
| 医生工号 | ----- | ----- |
| 医生姓名 | ----- | 戚玮琳 |
| 医生职级 | ----- | 主任医师 副教授  |
| 擅长领域 | ----- |  多年致力于心血管药物的临床研究 尤其擅长于高血压、高脂血症、心力衰竭以及冠心病的诊断与优化药物治疗。  |
| 专治疾病 | ----- | ----- |
| 个人网站 | ----- | ----- |
| 医生图片 | ----- | http://img.guahao.cn/images/expert/125336754304601/elarge_125619251378284.jpg |
| 执业经历 | ----- | ----- |
| 医生简介 | ----- |  戚玮琳，女，副主任医师，副教授，多年致力于心血管药物的研究 尤其擅长于高血压、高脂血症、心力衰竭以及冠心病的诊断与药物治疗。  |
| 医生评分 | ----- | ----- |
| 已预约数量 | ----- | 4653 |
| 患者评价数量 | ----- | 705 |

### 排班信息

| 字段名称 | 定义 | 示例值 |
| ---- | ----- | ----- |
| 医生ID | ----- | ----- |
| 医生姓名 | ----- | 张哲 |
| 排班日期 | ----- | ----- |
| 总约数量 | ----- | ----- |
| 早中晚类型 | ----- | ----- |
| 预约状态 | ----- | 可约、约满、停诊 |
| 科室信息 | ----- | 口腔修复 |
| 类型 | ----- | 专家门诊 普通门诊 |
| ---- | ----- | ----- |

### 可选时间段信息

| 字段名称 | 定义 | 示例值 |
| ---- | ----- | ----- |
| 排班ID | ----- | ----- |
| 要挂号医生姓名 | ----- | ----- |
| 要挂号医生ID | ----- |   |
| 类型 | ----- | 专家、普通、专病 |
| 状态 | ----- | 可约、约满、停诊 |
| 预约的开始时间 | ----- | ----- |
| 预约的结束时间 | ----- | ----- |
| 开始时间 | ----- | ----- |
| 挂号费 | ----- | ----- |
| 挂号地点 | ----- | ----- |


### 预约请求信息


| 字段名称 | 定义 | 示例值 |
| ---- | ----- | ----- |
| 要挂号的医生ID | ----- | ----- |
| 选定的排班ID | ----- | ----- |
| 选定的可选时间段ID | ----- | ----- |
| 进行预约的日期 | ----- | ----- |
| 疾病描述 预约原因 | ----- | ----- |
| 病人姓名 | ----- | ----- |
| 病人就诊卡号 | ----- | ----- |
| 病人身份证号 | ----- | ----- |
| 病人手机号码 | ----- | ----- |



### 预约请求确认信息



| 字段名称 | 定义 | 示例值 |
| ---- | ----- | ----- |
| 预约请求信息的id | ----- | ----- |
| 手机验证码 | ----- | ----- |


###  账号信息

| 字段名称 | 定义 | 示例值 |
| ---- | ----- | ----- |
| 账号ID | ----- | ----- |
| 就医卡类型 | ----- | 社保卡、医保卡、医联卡、自费卡、无卡、身份证 |
| 就医卡号码 | ----- | ----- |
| 身份证件类型 | ----- | 身份证 军官证 护照 |
| 就医人姓名 | ----- | ----- |
| 手机号码 | ----- | ----- |
| 账号状态 | ----- | 正常、黑名单、暂停使用、永久停用 |




