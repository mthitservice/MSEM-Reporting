# MSEM-Reporting

## Custom Report Models

https://learn.microsoft.com/en-us/mem/configmgr/core/servers/manage/creating-custom-report-models-in-sql-server-reporting-services

## German Learning Path Report Builder

https://learn.microsoft.com/de-de/sql/reporting-services/report-builder-tutorials?view=sql-server-ver16

https://learn.microsoft.com/de-de/sql/reporting-services/report-data/report-data-ssrs?view=sql-server-ver16




## All SCCM client computers with less than 1GB free disk space on C:
```sql
select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,
SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,
SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_LOGICAL_DISK on SMS_G_System_LOGICAL_DISK.ResourceID = SMS_R_System.ResourceId
where SMS_G_System_LOGICAL_DISK.DeviceID = "C:" and SMS_G_System_LOGICAL_DISK.FreeSpace <= 1000
```
## Computers with AVG and Security Essentials Antivirus

```sql
select SMS_R_SYSTEM.ResourceID, SMS_R_SYSTEM.ResourceType, SMS_R_SYSTEM.Name,
SMS_R_SYSTEM.SMSUniqueIdentifier, SMS_R_SYSTEM.ResourceDomainORWorkgroup,
SMS_R_SYSTEM.Client from SMS_R_System
inner join SMS_G_System_ADD_REMOVE_PROGRAMS on SMS_G_System_ADD_REMOVE_PROGRAMS.ResourceID = SMS_R_System.ResourceId
inner join SMS_G_System_ADD_REMOVE_PROGRAMS_64 on SMS_G_System_ADD_REMOVE_PROGRAMS_64.ResourceID = SMS_R_System.ResourceId
where SMS_G_System_ADD_REMOVE_PROGRAMS.DisplayName like "%avg%"
or SMS_G_System_ADD_REMOVE_PROGRAMS.DisplayName like "%security essentials%"
or SMS_G_System_ADD_REMOVE_PROGRAMS_64.DisplayName like "%security essentials%"
or SMS_G_System_ADD_REMOVE_PROGRAMS_64.DisplayName like "%avg%"
```

## All Workstations Client Agent Detailed Report
```sql
Declare @CollectionID as Varchar(8)
Set @CollectionID = 'SMS00001' -- specify scope collection ID
select distinct(Name),Case when IsClient= 1 then 'Healthy' else 'UnHealthy' end as 'AgentStatus',
(select case when count (v_GS_WORKSTATION_STATUS.ResourceID)=1 then 'Healthy' else 'UnHealthy' end
from v_GS_WORKSTATION_STATUS where DATEDIFF (day,LastHWScan,GetDate())<30 and
ResourceID=v_FullCollectionMembership.ResourceID)
as 'HWScanStatus',
(select case when count (v_GS_LastSoftwareScan.ResourceID)=1 then 'Healthy' else 'UnHealthy' end
from v_GS_LastSoftwareScan where DATEDIFF (day,LastScanDate,GetDate())<30 and
ResourceID=v_FullCollectionMembership.ResourceID)
as 'SWScanStatus',
(select case when count (v_UpdateScanStatus.ResourceID)=1 then 'Healthy' else 'UnHealthy' end
from v_UpdateScanStatus where DATEDIFF (day,LastScanTime,GetDate())<30 and LastErrorCode = 0 and
ResourceID=v_FullCollectionMembership.ResourceID)
as 'WSUSScanStatus',
(select DATEDIFF (day,LastHWScan,GetDate()) from v_GS_WORKSTATION_STATUS
where ResourceID=v_FullCollectionMembership.ResourceID)
as 'LastHWScanDays',
(select DATEDIFF (day,LastScanDate,GetDate()) from v_GS_LastSoftwareScan
where ResourceID=v_FullCollectionMembership.ResourceID)
as 'LastSWScanDays',
(select DATEDIFF (day,LastScanTime,GetDate()) from v_UpdateScanStatus
where LastErrorCode = 0 and ResourceID=v_FullCollectionMembership.ResourceID)
as 'LastWSUSScanDays'
from v_FullCollectionMembership where CollectionID = @CollectionID
and ResourceID in ( select ResourceID from v_R_System where Operating_System_Name_and0 like '%Workstation%')
order by 2 desc
```
## SCCM SQL Query to get Last State computers in Software Update deployment
```sql
select  
sn.StateName as LastEnforcementState,  
vrs.name0 as ComputerName,  
a.AssignmentName as DeploymentName,  
assc.StateTime ,  
a.CollectionName  
from v_CIAssignment a  
join v_AssignmentState_Combined assc on a.AssignmentID=assc.AssignmentID  
join v_StateNames sn on assc.StateType = sn.TopicType and sn.StateID=isnull(assc.StateID,0)  
join v_R_System vrs on vrs.ResourceID=assc.ResourceID  
where a.AssignmentID='xxxxxxxx'  
order by LastEnforcementState 

```

## All Workstations with Failed Software Updates

```sql
select  SMS_R_SYSTEM.ResourceID, 
        SMS_R_SYSTEM.ResourceType, 
        SMS_R_SYSTEM.Name, 
        SMS_R_SYSTEM.SMSUniqueIdentifier, 
        SMS_R_SYSTEM.ResourceDomainORWorkgroup, 
        SMS_R_SYSTEM.Client 
from    sms_r_system 
inner join SMS_UpdateComplianceStatus on SMS_UpdateComplianceStatus.machineid=sms_r_system.resourceid 
where   SMS_UpdateComplianceStatus.LastEnforcementMessageID = 11
```

## SCCM SQL Querys
https://eddiejackson.net/SCCM/SCCM_SQL_Queries.pdf
