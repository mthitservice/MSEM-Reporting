# MSEM-Reporting

## Custom Report Models

https://learn.microsoft.com/en-us/mem/configmgr/core/servers/manage/creating-custom-report-models-in-sql-server-reporting-services

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
