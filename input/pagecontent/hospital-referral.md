## 业务背景
&emsp;&emsp;“双向转诊”，简而言之就是“小病进社区，大病进医院”，积极发挥大中型医院在人才、技术及设备等方面的优势，同时充分利用各社区医院的服务功能和网点资源，促使基本医疗逐步下沉社区，社区群众危重病、疑难病的救治到大中型医院。

## 场景简介
&emsp;&emsp;医生根据患者个人意愿并综合患者病情、转入医院学科特长和床位占用情况等信息发送转诊预约申请请求，经转入医院医生审核确认同意后反馈转出医院，告知患者，同步传送电子病历、检查检验结果信息和（或）电子健康档案信息，患者到诊后，回传患者到诊消息通知。主要业务过程包括床位信息查询、转诊预约申请提交、申请审核与确认、审核应答、履约情况确认、病历上传等。


## 双线转诊流-住院 流程定义

 ###  流程定义结构

![流程定义](PlanDefinition-ActivityDefinition-Task-Relationship.png)
   
 ###  流程具体定义
  
1.	先定义流程中的步骤，该流程分为四个步骤，使用[ActivityDefinition](http://hl7.org/fhir/r4/activitydefinition.html)资源进行定义。
具体定义如下：
- [转诊预约申请-ActivityDefinition定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/ActivityDefinition-ActivityDefinition-application-for-referral-appointment.html)
- [转诊预约应答-ActivityDefinition定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/ActivityDefinition-ActivityDefinition-application-for-referral-appointment-response.html)
- [患者到诊应答-ActivityDefinition定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/ActivityDefinition-ActivityDefinition-patient-arrive-response.html)
- [上传完整病历-ActivityDefinition定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/ActivityDefinition-ActivityDefinition-medical-records-submitted.html)

2.	使用[PlanDefinition](http://hl7.org/fhir/r4/plandefinition.html)资源进行定义整个双向转诊-住院的业务流程，[PlanDefinition](http://hl7.org/fhir/r4/plandefinition.html)资源中包含Action节点，该节点为0..*定义，可以组装多个步骤。每个Action关联一个 [ActivityDefinition](http://hl7.org/fhir/r4/activitydefinition.html)定义的步骤。
- [住院双转流程-PlanDefinition定义](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/PlanDefinition-PlanDefinition-hospital-referral.html)
3.	流程开始后，每次步骤实例化具体的资源对应 [ActivityDefinition](http://hl7.org/fhir/r4/activitydefinition.html)定义的步骤，具体资源示例如下：

- [转诊预约申请示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Appointment-HospitalReferral-example.html)
- [转诊预约应答示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/AppointmentResponse-HospitalReferralResponse-example.html)
- [患者到诊应答示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/AppointmentResponse-PatientArriveResponse-example.html)
- [上传完整病历示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Task-Medical-records-submitted-example.html)


### 流程定义资源
- [ActivityDefinition](http://hl7.org/fhir/r4/activitydefinition.html)：活动定义资源，定义在医务流程中每一个活动步骤，描述其在流程中的作用。
- [PlanDefinition](http://hl7.org/fhir/r4/plandefinition.html)：流程定义资源，通过活动计划资源可以对活动定义资源进行组装，并且实现活动流程的定义，以及活动之前的先后关系，触发条件等信息，该资源可描述一个完整的业务流程。

### 流程实例资源
- [HospitalReferral](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/StructureDefinition-hospital-referral.html):转诊预约申请资源，该资源描述医院转诊的的申请。包括上转、下转都使用该资源。
- [HospitalReferralResponse](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/StructureDefinition-hospital-referral-response.html)：转诊预约应答资源，该资源描述在提交转诊申请后，由接收方给出是否同意的转诊应答。
- [Task](http://hl7.org/fhir/r4/task.html):任务资源，再改场景下描述上传病历的步骤任务。


## 数据组装方式

本场景中，使用[Bundle](http://hl7.org/fhir/r4/bundle.html) 资源作为 消息传输的载体，组装流程和业务数据，通过通信协议进行交互，数据传输结构如下：

### 数据结构图

![数据结构](structure-bundle.png) 

### 数据结构说明
[Bundle](http://hl7.org/fhir/r4/bundle.html)作为数据载体，[Bundle](http://hl7.org/fhir/r4/bundle.html)资源下的[Bundle.type](http://hl7.org/fhir/r4/bundle-definitions.html#Bundle.type)节点在该场景下可选择两种方式：
- message：使用消息发送的方式传输数据，第一个资源必须为第一个资源是[MessageHeader](http://hl7.org/fhir/r4/messageheader.html)。[Bundle.type](http://hl7.org/fhir/r4/bundle-definitions.html#Bundle.type)节点为 message，[MessageHeader](http://hl7.org/fhir/r4/messageheader.html)引用一个 工作流资源的实例。工作流资源回关联业务资源实例，他们的关系为0..*.业务资源之间也会相互关联。
- transaction/transaction-response：使用事物请求/应答方式传输数据，该方式是一个事务-所有资源由服务器作为原子资源提交进行处理。[Bundle.type](http://hl7.org/fhir/r4/bundle-definitions.html#Bundle.type)节点为 transaction/transaction-response。该方式第一个资源为工作流资源的实例，工作流资源回关联业务资源实例，他们的关系为0..*.业务资源之间也会相互关联。


### 流程资源和业务资源关  

>介绍流程资源和业务资源的相互关联关系，关系图如下：
![业务类图](Class.png)

- [MedicalRecordDocumentation](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-medical-record-documentation.html)：病历引用资源，引用第三方的病历文书，并且把病历文书作为附件形式上传。
- [HospitalBed](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-hospital-bed.html)：病床信息资源，描述医院床位的基础信息以及当前状态。
- [Patient](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-Patient.html)：患者资源，接受医疗健康服务的个人或动物，医疗过程是以患者为中心的。对交叉索进行中国本地化约定。
- [Practitioner](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-Practitioner.html)：医护人员资源，直接或间接参与提供医疗健康服务的人员。
- [Department](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-Department.html)：科室/部门资源，描述医院科室/部门的基础信息。
- [Hospital](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-Hospital.html)：医院资源，描述医疗机构（医院）的基本信息。
- [PractitionerRole](https://build.fhir.org/ig/HL7China/CN-CORE-R4/branches/develop/StructureDefinition-PractitionerRole.html)：医护人员工作信息资源，医务人员提供医疗服务时的岗位相关信息，包括所属组织、科室、角色/岗位等。
- [List](http://hl7.org/fhir/r4/list.html)：集合资源，在该场景下对患者的完整病历进行打包操作。



## 数据交互流程

数据交互流程遵循[PlanDefinition](http://hl7.org/fhir/r4/plandefinition.html)定义的流程规则和步骤，满足传输数据结构，使用[Bundle](http://hl7.org/fhir/r4/bundle.html)作为数据载体组装数据。

双向转诊-住院 数据交互主要包括两类：
- 转入医院和转出医院都具备自己的系统，并且和双向转诊平台做数据接入，实现功能。
- 转入医院不需要和双向转诊信息平台做系统对接，所有业务在双向转诊信息平台上直接操作。
  
### 通过双转平台对接

![流程图](sequence-platform.png)

>具体流程：

1.	转出医院根据获取到的转入医院的科室床位资源情况，发起转诊预约申请，附带基本病情介绍，转诊预约申请经过发送到 双转平台。 [转诊预约申请消息交换示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Bundle-Bundle-hospital-referral-example.html)
2.	双转平台根据实际情况进行审批,审批通过后，通过双转平台下发申请审核应答到转出医院。[转诊预约应答消息交换示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Bundle-Bundle-hospital-referral-response-example.html)
3.	患者到转入医院报到后，办理入院手续，双转平台回传患者到诊通知到转出医院。[患者到诊应答消息交换示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Bundle-Bundle-patient-arrive-response-example.html)
4.	根据到诊通知，转出医院上传该患者相关的病历资料到双转平台。[上传完整病历消息交换示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Bundle-Bundle-medical-records-submitted-example.html)

### 通过双转平台操作

![流程图](sequence.png)

> 具体流程：

1.	转出医院根据获取到的转入医院的科室床位资源情况，发起转诊预约申请，附带基本病情介绍，转诊预约申请经过双转平台发送到 转入医院。 [转诊预约申请消息交换示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Bundle-Bundle-hospital-referral-example.html)	
2.	转入医院医生根据实际情况进行审批,审批通过后，通过双转平台下发申请审核应答到转出医院。[转诊预约应答消息交换示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Bundle-Bundle-hospital-referral-response-example.html)
3.	患者到转入医院报到后，办理入院手续，回传患者到诊通知到转出医院。[患者到诊应答消息交换示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Bundle-Bundle-patient-arrive-response-example.html)
4.	根据到诊通知，转出医院上传该患者相关的病历资料到双转平台，并且转发到转入医院。[上传完整病历消息交换示例](https://build.fhir.org/ig/karldavids/CN-Public-Health-Core-R4/Bundle-Bundle-medical-records-submitted-example.html)




