# DASHBOARD FOR HUMAN RESOURCES

This will detail the various modules in Samarth under Human Resources such as:
- Career Advancement Scheme
- E-Housing
- Employee Management System
- Leave Management System

## CAREER ADVANCEMENT SCHEME

The dashboard for the CAS module should have information regarding the applications, in terms of their status. In the demo portal, the following dashboard is created for the CAS module:

![Demo CAS](/screenshots/Samarth%20Dashboards/Dashboard%20for%20Human%20Resources/Demo%20CAS.png)

The metrics being displayed in the CAS Dashboard in Kibana are as follows:

| S No. | METRIC | DESCRIPTION | FIELDS AND TABLE | QUERY |
| :---: | :---: | :---: | :---: | :---: |
| 1 | Drafted Application | Applications for career advancement that have been drafted | cas_pbas_application.status | select status from cas_pbas_applications |
| 2 | Forwarded to HOD | Applications forwarded to the HOD | cas_pbas_application.status | select status from cas_pbas_applications |
| 3 | Forwarded to VC | Applications forwarded to the VC | cas_pbas_application.status | select status from cas_pbas_applications |
| 4 | Forwarded to Reg | Applications forwarded to the reg | cas_pbas_application.status | select status from cas_pbas_applications |
| 5 | Forwarded to IQAC | Applications forwarded to the IQAC | cas_pbas_application.status | select status from cas_pbas_applications |
| 6 | In Screening | Applications currently in screening | cas_pbas_application.status | select status from cas_pbas_applications |
| 7 | Screening Done | Applications that have been screened | cas_pbas_application.status | select status from cas_pbas_applications |
| 8 | Reviewed | Applications that have been reviewed | cas_pbas_application.status | select status from cas_pbas_applications |

Using the field status from the cas_pbas_applications table we can get the information as shown in Kibana

![CAS](/screenshots/Samarth%20Dashboards/Dashboard%20for%20Human%20Resources/CAS%20Status.png)

The values need to be resolved during ingestion via FluentD through the record-transformer plugin. The values are as follows: 

```
DRAFT = 0;
FWD_TO_HOD = 1;
FWD_TO_VC = 2;
FWD_TO_REG = 3;
FWD_TO_IQAC = 4;
SCREENING = 5;
SCREENING_DONE = 6;
REVIEWED = 7;
```