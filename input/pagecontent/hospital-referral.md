## 业务背景

&emsp;&emsp;“双向转诊”，简而言之就是“小病进社区，大病进医院”，积极发挥大中型医院在人才、技术及设备等方面的优势，同时充分利用各社区医院的服务功能和网点资源，促使基本医疗逐步下沉社区，社区群众危重病、疑难病的救治到大中型医院。

## 场景简介

&emsp;&emsp;医生根据患者个人意愿并综合患者病情、转入医院学科特长和床位占用情况等信息发送转诊预约申请请求，经转入医院医生审核确认同意后反馈转出医院，告知患者，同步传送电子病历、检查检验结果信息和（或）电子健康档案信息，患者到诊后，回传患者到诊消息通知。主要业务过程包括床位信息查询、转诊预约申请提交、申请审核与确认、审核应答、履约情况确认、病历上传等。

## 场景流程

### 定义流程的资源

&emsp;&emsp;场景所涉及的流程及流程中的活动是通过以下两个FHIR的资源来定义的：

- 活动定义：[ActivityDefinition](http://www.hl7fhir.cn/R4/activitydefinition.html)：活动定义资源，定义在业务流程中每一个活动步骤，描述其在流程中的作用。
- 流程定义：[PlanDefinition](http://www.hl7fhir.cn/R4/plandefinition.html)：通过计划义定资源可以对活动定义资源进行组装，并描述活动之间的先后关系，触发条件等信息，该资源可描述一个完整的业务流程。


### 场景中的活动与流程的定义
  
 双向转诊流-住院流程分为“预约申请”、“预约应答”、“到诊应答”、“上传病历”四个步骤，
使用[ActivityDefinition](http://www.hl7fhir.cn/R4/activitydefinition.html)资源分别定义如下：

- [转诊预约申请-ActivityDefinition定义](ActivityDefinition-application-for-referral-appointment.html)
- [转诊预约应答-ActivityDefinition定义](ActivityDefinition-application-for-referral-appointment-response.html)
- [患者到诊应答-ActivityDefinition定义](ActivityDefinition-patient-arrive-response.html)
- [上传完整病历-ActivityDefinition定义](ActivityDefinition-medical-records-submitted.html)

> 使用[PlanDefinition](http://www.hl7fhir.cn/R4/plandefinition.html)资源定义双向转诊-住院的业务流程：
[住院双转流程-PlanDefinition定义](PlanDefinition-hospital-referral.html)，
它通过其Action元素将以上四个步骤组装起来。每个Action关联一个步骤。可以通过此流程定义资源实例实现流程自动化。
转诊预约申请活动为该流程的第一步，转诊申请发起后，根据转诊预约应答来确定后续步骤，如果为不同意，结束流程，如果为同意，等待患者到诊应答，患者到诊应答结束后，上传完整病历信息。

### 活动实例资源
 [ActivityDefinition](http://www.hl7fhir.cn/R4/activitydefinition.html)活动定义中有一个kind元素，此元素描述了该活动具体使用哪种资源类型,即活动实例资源类型。
比如[转诊预约申请-ActivityDefinition定义](ActivityDefinition-application-for-referral-appointment.html)的活动实例资源类型为继承自Appointment资源类型的[HospitalReferral](StructureDefinition-hospital-referral.html)资源。
此场景中涉及的活动实例资源如下：

- [HospitalReferral](StructureDefinition-hospital-referral.html):转诊预约申请资源，该资源描述医院转诊的的申请。包括上转、下转都使用该资源。
- [HospitalReferralResponse](StructureDefinition-hospital-referral-response.html)：转诊预约应答资源，该资源描述在提交转诊申请后，由接收方给出是否同意的转诊应答。
- [Task](http://www.hl7fhir.cn/R4/task.html):任务资源，再该场景下描述上传病历的步骤任务。

上述流程定义、活动定义、活动实例资源的关系图见下：
![流程定义](PlanDefinition-ActivityDefinition-Task-Relationship.png)


> 流程启动后，每个活动步骤将产生一个活动实例资源的新实例，其示例如下：

- [转诊预约申请示例](Appointment-HospitalReferral-example.html)
- [转诊预约应答示例](AppointmentResponse-HospitalReferralResponse-example.html)
- [患者到诊应答示例](AppointmentResponse-PatientArriveResponse-example.html)
- [上传完整病历示例](Task-Medical-records-submitted-example.html)



## 数据交换方式与结构

FHIR的数据交换方式支持RESTful、SOA、消息交换等。本场景目前只定义了消息交换的方式，其他方式在进一步定义中。

[FHIR消息交换框架](http://www.hl7fhir.cn/R4/messaging.html)要求的应用程序符合“ FHIR消息传递”，它支持大多数消息组件、ESB等消息引擎，支持异步消息和消息应答机制。

数据交换使用的消息体（报文）通过[Bundle](http://www.hl7fhir.cn/R4/bundle.html)（资源捆束）组装数据。
本场景中所有的数据交换使用的消息体结构遵循以下规则：

- 1、资源捆束中的[Bundle.type](http://www.hl7fhir.cn/R4/bundle-definitions.html#Bundle.type)元素固定为“message”。
- 2、资源捆束内的第一个条目(entry)中的资源必须是[MessageHeader](http://www.hl7fhir.cn/R4/messageheader.html)（消息头）。
- 3、资源捆束内的第二个条目中的资源必须是描述关于业务流程活动步骤的活动实例资源。
- 4、资源捆束内的其它条目中应该包含对应步骤的相关业务资源。

> 资源捆束里的各条目中的资源的相互关系是通过消息头引用活动实例资源，活动实例资源再关联业务资源来展现的。示意图如下：

![数据结构](structure-bundle.png)

### 消息体结构定义
资源捆束中条目中的资源类型在不同的业务活动（步骤）中是不同的，
本规范集使用[MessageDefinition](http://www.hl7fhir.cn/R4/messagedefinition.html)定义每个活动数据交换的消息体结构，
遵循[住院双转流程定义](PlanDefinition-hospital-referral.html)中的Action元素对应的步骤定义。每个活动步骤的数据交换消息体结构的定义如下：

- 一、[转诊预约申请-MessageDefinition定义](MessageDefinition-hospital-referral.html)：
在转出医疗机构向转入医疗机构发起转诊预约申请时使用的消息体结构的定义。该消息要求作出消息应答：
	- 1.转入机构收到转诊申请后，根据自身情况和患者病情，审核是否通过转诊，审批通过后，作出应答。
	- 2.患者到达转入机构，办理手续后，发起患者到诊应答。
- 二、[转诊预约应答-MessageDefinition定义](MessageDefinition-hospital-referral-response.html)：
在转入医疗机构收到转出医疗机构发起的转诊申请后，回复转出机构是否审批通过时使用的消息体结构的定义。
- 三、[患者到诊应答-MessageDefinition定义](MessageDefinition-patient-arrive-response.html)：
当转诊患者到达转入医疗机构后，转入医疗机构向转出医疗机构发送的消息体结构的定义。该消息要求作出消息应答：上传完整病历。
- 四、[上传完整病历-MessageDefinition定义](MessageDefinition-medical-records-submitted.html)：
在转出医疗机构收到患者到诊应答消息后，上传完整患者完整病历时使用的消息体结构的定义。

### 相关业务资源  

在本场景中数据交换所涉及的相关业务资源（即资源捆束中可能包含的业务资源）如下：

- [MedicalRecordDocumentation](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-medical-record-documentation.html)：
病历引用资源，引用第三方的病历文书，并且把病历文书作为附件形式上传。
- [HospitalBed](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-hospital-bed.html)：
病床信息资源，描述医院床位的基础信息以及当前状态。
- [Patient](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-Patient.html)：
患者资源，接受医疗健康服务的个人或动物，医疗过程是以患者为中心的。对交叉索进行中国本地化约定。
- [Practitioner](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-Practitioner.html)：
医护人员资源，直接或间接参与提供医疗健康服务的人员。
- [Department](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-Department.html)：
科室/部门资源，描述医院科室/部门的基础信息。
- [Hospital](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-Hospital.html)：
医院资源，描述医疗机构（医院）的基本信息。
- [PractitionerRole](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-PractitionerRole.html)：
医护人员岗位资源，医务人员提供医疗服务时的岗位相关信息，包括所属组织、科室、角色/岗位等。
- [List](http://www.hl7fhir.cn/R4/list.html)：集合资源，在该场景下用作对患者的完整病历进行打包的容器。

> 资源捆束中包含的活动实例资源和业务资源相互关联以准确表达业务流程状态和业务数据信息，其相互关系图如下：
  
![业务类图](Class.png)

## 数据交互流程

数据交互流程遵循[PlanDefinition](http://www.hl7fhir.cn/R4/plandefinition.html)定义的流程规则和步骤，
按照[MessageDefinition](http://www.hl7fhir.cn/R4/messagedefinition.html)定义每个消息体结构组装数据。

双向转诊-住院 数据交互主要包括两种情况：

- 1、转入医院和转出医院都具备自己的系统，并且都分别和双向转诊平台做对接，实现功能。
- 2、转入医院不需要和双向转诊信息平台做系统对接，所有业务在双向转诊信息平台上直接操作。
  
### 通过双转平台对接

![流程图](sequence-platform.png)
  
> 具体流程：

1. 转出医院根据获取到的转入医院的科室床位资源情况，发起转诊预约申请，附带基本病情介绍发送到双转平台，双转平台收到转诊预约申请后，将转诊预约申请转发到转入医院。
该步骤符合流程定义[住院双转流程定义](PlanDefinition-hospital-referral.html)中的第一个action节点定义的步骤-转诊预约申请，
按照[转诊预约申请消息定义](MessageDefinition-hospital-referral.html)的消息结构使用
[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送。
具体示例参见： [转诊预约申请消息交换示例](Bundle-Bundle-hospital-referral-example.html)。
2. 转入医院根据实际情况进行审批，发送转诊预约应答到双转平台，双转平台收到转诊预约应答后，将转诊预约应答消息转发到转出医院。
该流程符合流程定义[住院双转流程定义](PlanDefinition-hospital-referral.html)中的第二个action节点定义的步骤-转诊预约应答，
按照[转诊预约应答消息定义](MessageDefinition-hospital-referral-response.html)的消息结构使用
[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送。
具体示例参见：[转诊预约应答消息交换示例](Bundle-Bundle-hospital-referral-response-example.html)。
3. 患者到转入医院报到后，办理入院手续，转入医院发送患者到诊应答到双转平台，双转平台收到患者到诊应答后，将患者到诊应答转发到转出医院。
该流程符合流程定义[住院双转流程定义](PlanDefinition-hospital-referral.html)中的第三个action节点定义的步骤-患者到诊应答，
按照[患者到诊应答消息定义](MessageDefinition-patient-arrive-response.html)的消息结构使用
[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送。
具体示例参见：[患者到诊应答消息交换示例](Bundle-Bundle-patient-arrive-response-example.html)。
4. 转出医院收到患者到诊应答后，发送该患者完整病历资料到双转平台，双转平台收到该患者的完整病历资料后，将患者完整病历转发到转入医院。
该流程符合流程定义[住院双转流程定义](PlanDefinition-hospital-referral.html)中的第四个action节点定义的步骤-上传完整病历，
按照[上传完整病历消息定义](MessageDefinition-medical-records-submitted.html)的消息结构使用
[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送。
具体示例参见：[上传完整病历消息交换示例](Bundle-Bundle-medical-records-submitted-example.html)。

### 通过双转平台操作

![流程图](sequence.png)
  
> 具体流程：

1. 转出医院根据获取到的转入医院的科室床位资源情况，发起转诊预约申请，附带基本病情介绍发送到双转平台。
该步骤符合流程定义[住院双转流程定义](PlanDefinition-hospital-referral.html)中的第一个action节点定义的步骤-转诊预约申请，
按照[转诊预约申请消息定义](MessageDefinition-hospital-referral.html)的消息结构使用
[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送。
具体示例参见： [转诊预约申请消息交换示例](Bundle-Bundle-hospital-referral-example.html)。
2. 双转平台根据实际情况进行审批，发送转诊预约应答到转出医院。
该流程符合流程定义[住院双转流程定义](PlanDefinition-hospital-referral.html)中的第二个action节点定义的步骤-转诊预约应答，
按照[转诊预约应答消息定义](MessageDefinition-hospital-referral-response.html)的消息结构使用
[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送。
具体示例参见：[转诊预约应答消息交换示例](Bundle-Bundle-hospital-referral-response-example.html)。
3. 患者到转入医院报到后，办理入院手续，双转平台发送患者到诊应答到转出医院。
该流程符合流程定义[住院双转流程定义](PlanDefinition-hospital-referral.html)中的第三个action节点定义的步骤-患者到诊应答，
按照[患者到诊应答消息定义](MessageDefinition-patient-arrive-response.html)的消息结构使用
[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送。
具体示例参见：[患者到诊应答消息交换示例](Bundle-Bundle-patient-arrive-response-example.html)。
1. 转出医院收到患者到诊应答后，发送该患者完整病历资料到双转平台。
该流程符合流程定义[住院双转流程定义](PlanDefinition-hospital-referral.html)中的第四个action节点定义的步骤-上传完整病历，
按照[上传完整病历消息定义](MessageDefinition-medical-records-submitted.html)的消息结构定义使用
[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送。
具体示例参见：[上传完整病历消息交换示例](Bundle-Bundle-medical-records-submitted-example.html)。
