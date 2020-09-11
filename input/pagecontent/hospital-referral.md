## 业务背景
&emsp;&emsp;“双向转诊”，简而言之就是“小病进社区，大病进医院”，积极发挥大中型医院在人才、技术及设备等方面的优势，同时充分利用各社区医院的服务功能和网点资源，促使基本医疗逐步下沉社区，社区群众危重病、疑难病的救治到大中型医院。

## 场景简介
&emsp;&emsp;医生根据患者个人意愿并综合患者病情、转入医院学科特长和床位占用情况等信息发送转诊预约申请请求，经转入医院医生审核确认同意后反馈转出医院，告知患者，同步传送电子病历、检查检验结果信息和（或）电子健康档案信息，患者到诊后，回传患者到诊消息通知。主要业务过程包括床位信息查询、转诊预约申请提交、申请审核与确认、审核应答、履约情况确认、病历上传等。


## 双线转诊流-住院 流程定义

###  流程定义资源

&emsp;&emsp;这里描述整个双向转诊-住院的物业流程是如何通过FHIR的流程定义资源实现。
该流程中主要涉及到两个核心资源：

- [ActivityDefinition](http://www.hl7fhir.cn/R4/activitydefinition.html)：活动定义资源，定义在医务流程中每一个活动步骤，描述其在流程中的作用。
- [PlanDefinition](http://www.hl7fhir.cn/R4/plandefinition.html)：流程定义资源，通过活动计划资源可以对活动定义资源进行组装，并且实现活动流程的定义，以及活动之前的先后关系，触发条件等信息，该资源可描述一个完整的业务流程。

![流程定义](..\images/PlanDefinition-ActivityDefinition-Task-Relationship.png)


###  流程具体定义
  
1.	先定义流程中的步骤，该流程分为四个步骤，使用[ActivityDefinition](http://www.hl7fhir.cn/R4/activitydefinition.html)资源进行定义。
具体定义如下：
- [转诊预约申请-ActivityDefinition定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/ActivityDefinition-ActivityDefinition-application-for-referral-appointment.html)
- [转诊预约应答-ActivityDefinition定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/ActivityDefinition-ActivityDefinition-application-for-referral-appointment-response.html)
- [患者到诊应答-ActivityDefinition定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/ActivityDefinition-ActivityDefinition-patient-arrive-response.html)
- [上传完整病历-ActivityDefinition定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/ActivityDefinition-ActivityDefinition-medical-records-submitted.html)

2.	使用[PlanDefinition](http://www.hl7fhir.cn/R4/plandefinition.html)资源进行定义整个双向转诊-住院的业务流程，[PlanDefinition](http://www.hl7fhir.cn/R4/plandefinition.html)资源中包含Action节点，该节点为0..*定义，可以组装多个步骤。每个Action关联一个 [ActivityDefinition](http://www.hl7fhir.cn/R4/activitydefinition.html)定义的步骤。转诊预约申请 为该流程的第一步，转诊申请发起后，根据转诊预约应答来确定下面步骤，如果为不同意，结束流程，如果为同意，等待患者到诊应答，患者到诊应答结束后，上传完整病历信息。
- [住院双转流程-PlanDefinition定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/PlanDefinition-PlanDefinition-hospital-referral.html)
3.	流程开始后，每次步骤实例化具体的资源对应 [ActivityDefinition](http://www.hl7fhir.cn/R4/activitydefinition.html)定义的步骤，具体资源示例如下：

- [转诊预约申请示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Appointment-HospitalReferral-example.html)
- [转诊预约应答示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/AppointmentResponse-HospitalReferralResponse-example.html)
- [患者到诊应答示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/AppointmentResponse-PatientArriveResponse-example.html)
- [上传完整病历示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Task-Medical-records-submitted-example.html)




### 流程实例资源
- [HospitalReferral](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/StructureDefinition-hospital-referral.html):转诊预约申请资源，该资源描述医院转诊的的申请。包括上转、下转都使用该资源。
- [HospitalReferralResponse](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/StructureDefinition-hospital-referral-response.html)：转诊预约应答资源，该资源描述在提交转诊申请后，由接收方给出是否同意的转诊应答。
- [Task](http://www.hl7fhir.cn/R4/task.html):任务资源，再改场景下描述上传病历的步骤任务。


## 数据组装方式

本场景中，使用[MessageDefinition](http://www.hl7fhir.cn/R4/messagedefinition.html)定义每个消息体结构，遵循[住院双转流程定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/PlanDefinition-PlanDefinition-hospital-referral.html)中的Action节点下对应的步骤定义，每个步骤对应相应的消息定义，明确消息体的具体结构，以FHIR 语法方式描述整个数据结构，具体结构如下示例：
- [转诊预约申请-MessageDefinition定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/MessageDefinition-MessageDefinition-hospital-referral-example.html)：该消息定义为 双向转诊-住院场景中定义的消息，在转出医疗机构向转入医疗结构发起转诊预约申请时使用，该消息定义描述了在整个消息传输过程中必须具备的资源结构，包括 第一个资源 必须为MessageHeader资源（1..1）,第二个资源必须为Appointment资源(1..1),可以包含其他Resource资源（0..*），并且该消息具备2个应答消息：1.转入医院收到转诊申请后，根据自己医院情况和患者病情，审核是否通过转诊，审批通过后，作出应答。2.患者到达转入医院，办理手续后，发起患者到诊应答。
- [转诊预约应答-MessageDefinition定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/MessageDefinition-MessageDefinition-hospital-referral-response-example.html)：该消息定义为 双向转诊-住院场景中定义的消息，在转入医疗机构收到转入医疗机构发起的转诊申请，并且审批是否通过后使用，该消息定义描述了在整个消息传输过程中必须具备的资源结构，包括 第一个资源 必须为MessageHeader资源（1..1）,第二个资源必须为AppointmentResponse资源(1..1),可以包含其他Resource资源（0..*），并且该消息不需要应答。
- [患者到诊应答-MessageDefinition定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/MessageDefinition-MessageDefinition-patient-arrive-response-example.html)：该消息定义为 双向转诊-住院场景中定义的消息，在转出医院接收到转诊预约应答，并且通过审批后，患者到达转入医疗机构，发送患者到诊应答消息，使用该消息发送患者到诊应答信息，该消息定义描述了在整个消息传输过程中必须具备的资源结构，包括 第一个资源 必须为MessageHeader资源（1..1）,第二个资源必须为AppointmentResponse资源(1..1),可以包含其他Resource资源（0..*），并且该消息具备1个应答消息：患者到达转入医院，办理手续后，收到该患者到诊应答的消息，通过该消息上传完整病历作为应答。
- [上传完整病历-MessageDefinition定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/MessageDefinition-MessageDefinition-medical-records-submitted-example.html)：该消息定义为 双向转诊-住院场景中定义的消息，在转出医疗机构收到患者到诊应答消息后，使用该消息上传完整病历信息，该消息定义描述了在整个消息传输过程中必须具备的资源结构，包括 第一个资源 必须为MessageHeader资源（1..1）,第二个资源必须为Task资源(1..1),可以包含其他Resource资源（0..*），并且该消息不需要应答。


### 消息结构说明
本场景暂时只支持Message方式，其他方式在进一步定义中，消息结构使用
[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据，[Bundle](http://www.hl7fhir.cn/R4/bundle.html)资源下的[Bundle.type](http://www.hl7fhir.cn/R4/bundle-definitions.html#Bundle.type)节点在该场景下选择：message。

使用消息发送的方式传输数据，第一个资源必须为第一个资源是[MessageHeader](http://www.hl7fhir.cn/R4/messageheader.html)。[Bundle.type](http://www.hl7fhir.cn/R4/bundle-definitions.html#Bundle.type)节点为 message，[MessageHeader](http://www.hl7fhir.cn/R4/messageheader.html)引用一个 工作流资源的实例。工作流资源回关联业务资源实例，他们的关系为0..*.业务资源之间也会相互关联。

### 消息体定义结构图

![数据结构](..\images/structure-bundle.png) 

  
### 流程资源和业务资源关  

>介绍流程资源和业务资源的相互关联关系，关系图如下：
![业务类图](..\images/Class.png)

- [MedicalRecordDocumentation](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-medical-record-documentation.html)：病历引用资源，引用第三方的病历文书，并且把病历文书作为附件形式上传。
- [HospitalBed](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-hospital-bed.html)：病床信息资源，描述医院床位的基础信息以及当前状态。
- [Patient](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-Patient.html)：患者资源，接受医疗健康服务的个人或动物，医疗过程是以患者为中心的。对交叉索进行中国本地化约定。
- [Practitioner](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-Practitioner.html)：医护人员资源，直接或间接参与提供医疗健康服务的人员。
- [Department](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-Department.html)：科室/部门资源，描述医院科室/部门的基础信息。
- [Hospital](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-Hospital.html)：医院资源，描述医疗机构（医院）的基本信息。
- [PractitionerRole](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-PractitionerRole.html)：医护人员工作信息资源，医务人员提供医疗服务时的岗位相关信息，包括所属组织、科室、角色/岗位等。
- [List](http://www.hl7fhir.cn/R4/list.html)：集合资源，在该场景下对患者的完整病历进行打包操作。

## 数据交互流程

数据交互流程遵循[PlanDefinition](http://www.hl7fhir.cn/R4/plandefinition.html)定义的流程规则和步骤，按照[MessageDefinition](http://www.hl7fhir.cn/R4/messagedefinition.html)定义每个消息体结构组装数据

双向转诊-住院 数据交互主要包括两类：
- 转入医院和转出医院都具备自己的系统，并且和双向转诊平台做数据接入，实现功能。
- 转入医院不需要和双向转诊信息平台做系统对接，所有业务在双向转诊信息平台上直接操作。
  
### 通过双转平台对接

![流程图](..\images/sequence-platform.png)

>具体流程：

1.	转出医院根据获取到的转入医院的科室床位资源情况，发起转诊预约申请，附带基本病情介绍，发送转诊预约申请经过发送到 双转平台。该流程符合流程定义[住院双转流程定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/PlanDefinition-PlanDefinition-hospital-referral.html)中的第一个action节点定义的步骤-转诊预约申请，按照[转诊预约申请消息定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/MessageDefinition-MessageDefinition-hospital-referral-example.html)的消息结构定义使用[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送，具体示例参见： [转诊预约申请消息交换示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Bundle-Bundle-hospital-referral-example.html)
2.	双转平台根据实际情况进行审批，通过双转平台下发转诊预约应答到转出医院。该流程符合流程定义[住院双转流程定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/PlanDefinition-PlanDefinition-hospital-referral.html)中的第二个action节点定义的步骤-转诊预约应答，按照[转诊预约应答消息定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/MessageDefinition-MessageDefinition-hospital-referral-response-example.html)的消息结构定义使用[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送，具体示例参见：[转诊预约应答消息交换示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Bundle-Bundle-hospital-referral-response-example.html)
3.	患者到转入医院报到后，办理入院手续，双转平台回传患者到诊应答到转出医院。该流程符合流程定义[住院双转流程定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/PlanDefinition-PlanDefinition-hospital-referral.html)中的第三个action节点定义的步骤-患者到诊应答，按照[患者到诊应答消息定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/MessageDefinition-MessageDefinition-patient-arrive-response-example.html)的消息结构定义使用[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送，具体示例参见：[患者到诊应答消息交换示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Bundle-Bundle-patient-arrive-response-example.html)
4.	根据到诊通知，转出医院上传该患者相关的病历资料到双转平台。该流程符合流程定义[住院双转流程定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/PlanDefinition-PlanDefinition-hospital-referral.html)中的第四个action节点定义的步骤-上传完整病历，按照[上传完整病历消息定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/MessageDefinition-MessageDefinition-medical-records-submitted-example.html)的消息结构定义使用[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送，具体示例参见：[上传完整病历消息交换示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Bundle-Bundle-medical-records-submitted-example.html)

### 通过双转平台操作

![流程图](..\images/sequence.png)

> 具体流程：

1.	转出医院根据获取到的双转平台的科室床位资源情况，发起转诊预约申请，附带基本病情介绍发送到双转平台，该流程符合流程定义[住院双转流程定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/PlanDefinition-PlanDefinition-hospital-referral.html)中的第一个action节点定义的步骤-转诊预约申请，按照[转诊预约申请消息定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/MessageDefinition-MessageDefinition-hospital-referral-example.html)的消息结构定义使用[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送，具体示例参见： [转诊预约申请消息交换示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Bundle-Bundle-hospital-referral-example.html)。	
2.	双转平台根据实际情况进行审批，下发转诊预约应答到转出医院。该流程符合流程定义[住院双转流程定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/PlanDefinition-PlanDefinition-hospital-referral.html)中的第二个action节点定义的步骤-转诊预约应答，按照[转诊预约应答消息定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/MessageDefinition-MessageDefinition-hospital-referral-response-example.html)的消息结构定义使用[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送，具体示例参见：[转诊预约应答消息交换示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Bundle-Bundle-hospital-referral-response-example.html)
3.	患者到转入医院报到后，办理入院手续，双转平台回传患者到诊通知到转出医院。该流程符合流程定义[住院双转流程定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/PlanDefinition-PlanDefinition-hospital-referral.html)中的第三个action节点定义的步骤-患者到诊应答，按照[患者到诊应答消息定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/MessageDefinition-MessageDefinition-patient-arrive-response-example.html)的消息结构定义使用[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送，具体示例参见：[患者到诊应答消息交换示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Bundle-Bundle-patient-arrive-response-example.html)
4.	根据到诊通知，转出医院上传该患者相关的病历资料到双转平台，该流程符合流程定义[住院双转流程定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/PlanDefinition-PlanDefinition-hospital-referral.html)中的第四个action节点定义的步骤-上传完整病历，按照[上传完整病历消息定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/MessageDefinition-MessageDefinition-medical-records-submitted-example.html)的消息结构定义使用[Bundle](http://www.hl7fhir.cn/R4/bundle.html)组装数据并发送，具体示例参见：[上传完整病历消息交换示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Bundle-Bundle-medical-records-submitted-example.html)


## 数据交换方式

FHIR的数据交换方式支持多种类型包括：FHIR + REST、服务方式、消息方式。本场景中支持消息方式进行交互，其他方式正在制定中。

### FHIR + REST 方式
FHIR + REST（“ RESTful FHIR”），是当今实施FHIR的主要方法，FHIR被描述为基于REST的常见行业级别用法的“ RESTful”规范。实际上，FHIR仅支持REST成熟度模型的级别2 作为核心规范的一部分，尽管通过使用扩展也可以实现完全级别3的一致性。因为FHIR是标准，所以它依赖于资源结构和接口的标准化。这可能被视为违反REST原则，但是对于确保跨不同系统的一致互操作性至关重要。[FHIR + REST 方式 详细说明参见此处](http://www.hl7fhir.cn/R4/http.html)

### 服务方式
特定服务的定义是特定于域或上下文的任务。可以预期，这将作为单独的规范来完成，从而利用此规范中定义的基础设施。HL7已经定义了一个服务组合，这些服务可以利用基于REST和SOAP的接口，但是传入和传出这些服务的参数目前还不是基于资源的。这些是潜在的扩展点，可能非常适合FHIR实施。像这样定义的服务能够基于本规范中定义的公共基础平台功能（例如REST或消息传递），并在适当的地方添加特定的交互。或者，服务接口可以构建完全独立的实现。
FHIR + WS *使用Web服务服务作为通信协议而不是REST的FHIR实现。例如，这将包括使用FHIR资源作为SOAP调用中的有效负载参数。
FHIR + SOA模式说明了在SOA做法指导下，在实现技术（SOAP，REST或其他）之上应用交互模式。[服务方式 详细说明参见此处](http://www.hl7fhir.cn/R4/services.html)

### 消息方式
FHIR资源可以在传统的消息传递上下文中使用，就像HL7 v2一样。符合此框架要求的应用程序符合“ FHIR消息传递”。支持大多数消息组建、ESB等消息引擎，支持异步消息和消息应答机制。
[消息方式 详细说明参见此处](http://www.hl7fhir.cn/R4/messaging.html)

