## 业务背景
&emsp;&emsp;“双向转诊”，简而言之就是“小病进社区，大病进医院”，积极发挥大中型医院在人才、技术及设备等方面的优势，同时充分利用各社区医院的服务功能和网点资源，促使基本医疗逐步下沉社区，社区群众危重病、疑难病的救治到大中型医院。

## 场景简介
&emsp;&emsp;医生根据患者个人意愿并综合患者病情、转入医院学科特长和床位占用情况等信息发送转诊预约申请请求，经转入医院医生审核确认同意后反馈转出医院，告知患者，同步传送电子病历、检查检验结果信息和（或）电子健康档案信息，患者到诊后，回传患者到诊消息通知。主要业务过程包括床位信息查询、转诊预约申请提交、申请审核与确认、审核应答、履约情况确认、病历上传等。


## FHIR资源应用

> 在本场景中主要使用到以下资源：

### 业务资源  

>关系图如下：
![业务类图](Class.png)

- [HospitalReferral](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/StructureDefinition-hospital-referral.html):双向转诊预约申请资源，该资源描述医院转诊的的申请。包括上转、下转都使用该资源。
- [HospitalReferralResponse](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/StructureDefinition-hospital-referral-response.html)：双向转诊应答资源，该资源描述在提交转诊申请后，由接收方 给出是否同意的转诊应答。
- [MedicalRecordDocumentation](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-medical-record-documentation.html)：病历引用资源，引用第三方的病历文书，并且把病历文书作为附件形式上传。
- [HospitalBed](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-hospital-bed.html)：病床信息资源，描述医院床位的基础信息以及当前状态。
- [Patient](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-Patient.html)：患者资源，接受医疗健康服务的个人或动物，医疗过程是以患者为中心的。对交叉索进行中国本地化约定。
- [Practitioner](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-Practitioner.html)：医护人员资源，直接或间接参与提供医疗健康服务的人员。
- [Department](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-Department.html)：科室/部门资源，描述医院科室/部门的基础信息。
- [Hospital](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-Hospital.html)：医院资源，描述医疗机构（医院）的基本信息。
- [PractitionerRole](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-PractitionerRole.html)：医护人员工作信息资源，医务人员提供医疗服务时的岗位相关信息，包括所属组织、科室、角色/岗位等。


### 工作流资源

>关系图如下：
![工作流关系图](PlanDefinition-ActivityDefinition-Task-Relationship.png)
  
- [ActivityDefinition](http://hl7.org/fhir/r4/activitydefinition.html)：活动定义资源，定义在医务流程中每一个活动，描述期在流程中的作用。
- [PlanDefinition](http://hl7.org/fhir/r4/plandefinition.html)：活动计划资源，通过活动计划资源可以对活动定义资源进行组装，并且实现活动流程的定义，以及活动之前的先后关系，触发条件等信息，该资源可描述一个完整的业务流程。
- [Task](http://hl7.org/fhir/r4/task.html)：任务资源，作为[ActivityDefinition](http://hl7.org/fhir/r4/activitydefinition.html)的实例，每次开启流程后，每一个步骤都对应一个[Task](http://hl7.org/fhir/r4/task.html)资源，作为流程步骤的附加产物，并且关联该任务执行中 产出的业务资源。



## 对接方式

双向转诊-住院 业务流程主要包括两类：
- 转入医院和转出医院都具备自己的系统，并且和双向转诊平台做数据接入，实现功能。
- 转入医院不需要和双向转诊信息平台做系统对接，所有业务在双向转诊信息平台上直接操作。
  
### 通过双转平台对接

![流程图](sequence-platform.png)

>转诊路程：

1.	转出医院根据获取到的转入医院的科室床位资源情况，发起转诊预约申请，附带基本病情介绍，转诊预约申请经过发送到 双转平台。
2.	双转平台根据实际情况进行审批。
3.	审批通过后，通过双转平台下发申请审核应答到转出医院。
4.	患者到转入医院报到后，办理入院手续，双转平台回传患者到诊通知到转出医院。
5.	根据到诊通知，转出医院上传该患者相关的病历资料到双转平台。

### 通过双转平台操作

![流程图](sequence.png)

> 转诊流程：

1.	转出医院根据获取到的转入医院的科室床位资源情况，发起转诊预约申请，附带基本病情介绍，转诊预约申请经过双转平台发送到 转入医院。
2.	转入医院医生根据实际情况进行审批。
3.	审批通过后，通过双转平台下发申请审核应答到转出医院。
4.	患者到转入医院报到后，办理入院手续，回传患者到诊通知到转出医院。
5.	根据到诊通知，转出医院上传该患者相关的病历资料到双转平台，并且转发到转入医院。



## 数据传输方式

本场景中，使用[Bundle](http://hl7.org/fhir/r4/bundle.html) 资源作为 消息传输的载体，组装流程和业务数据，通过通信协议进行交互，数据传输结构如下：

### 数据传输结构

>数据传输流程图如下：

![数据结构](structure-bundle.png) 

[Bundle](http://hl7.org/fhir/r4/bundle.html)作为数据载体，[Bundle](http://hl7.org/fhir/r4/bundle.html)资源下的[Bundle.type](http://hl7.org/fhir/r4/bundle-definitions.html#Bundle.type)节点在该场景下可选择两种方式：
- message：使用消息发送的方式传输数据，第一个资源必须为第一个资源是[MessageHeader](http://hl7.org/fhir/r4/messageheader.html)。[Bundle.type](http://hl7.org/fhir/r4/bundle-definitions.html#Bundle.type)节点为 message。
- transaction/transaction-response：使用事物请求/应答方式传输数据，该方式是一个事务-所有资源由服务器作为原子资源提交进行处理。[Bundle.type](http://hl7.org/fhir/r4/bundle-definitions.html#Bundle.type)节点为 transaction/transaction-response。

