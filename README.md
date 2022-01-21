**Requirement** 
An end-to-end comprehensive monitoring solution will support the MC environment, providing extensive alerting and reporting capabilities.

[[_TOC_]]

###Monitoring Control Objectives

| **Control Objective ID** | **Control Objective** | **Enabled via** | **Gap exists for compliance** |
|--|--|--|--|
| 08.11.03 | Define and implement procedures to monitor the IT infrastructure and related events. Ensure that sufficient chronological information is being stored in operations logs to enable the reconstruction, review and examination of the time sequences of operations and the other activities surrounding or supporting operations. | Azure Log Analytcis / Power BI activity log | No |
| 08.12.01 | Detection, prevention and recovery controls to protect against malicious code and appropriate user awareness procedures should be implemented. | Azure Sentinel / Defender | No |
| 08.17.01 | Audit logs recording user activities, exceptions and information security events should be produced and kept for an agreed period to assist in future investigations and access control monitoring. | Azure Log Analytcis / Power BI activity log | No |
| 08.17.04 | System administrator and system operator activities should be logged. | Azure Log Analytcis / Power BI activity log | No |
| 09.02.02 | The allocation and use of privileges should be restricted and controlled. Privileges should be  granted through the use of role based definitions that comply with segregation of duties requirements. | Azure PIM | No |
| 11.12.03 | Timely information about technical vulnerabilities of information systems being used should be obtained, the organization's exposure to such vulnerabilities evaluated, and appropriate measures taken to address the associated risk. | Azure Log Analytcis / Power BI activity log | Yes, alerts |


###Monitoring Solution considerations
- Centralized solution to enable proactive alerts after the collection of event and diagnostics data from multiple resources
- Different ingestion methods depending on the export/integration capabilities of source and destination
- Data refresh frequency is stated at 3-4 times a day
- Alerting should be near real time
- Should persist data for long term (180 days) in a cost effective way

###Ingestion paths
Depending on the workload, three different paths have been identified:

1. Log Analytics export for [supported Azure workloads](https://docs.microsoft.com/en-us/azure/azure-monitor/monitor-reference#azure-supported-services) to Azure Event Hub and [into Azure Data Explorer](https://docs.microsoft.com/en-us/azure/data-explorer/ingest-data-no-code?tabs=diagnostic-logs).

<IMG  src="https://docs.microsoft.com/en-us/azure/azure-monitor/logs/media/logs-data-export/data-export-overview.png"  alt="Data export overview"/>

2. Direct Streaming from supported services
(i.e. [Defender Streaming API](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/raw-data-export?view=o365-worldwide))

3. Azure Function API calls to other services

Some of these methos support a range of configuration alternatives.
For example, while using an Azure Function to query the PowerBI API, we could then define:
- A route from the source to Azure Function to ADX
- A route from the source to Azure Function to Storage to ADX via Event Grid trigger
- A route from the source to Azure Function to Event Hubs to ADX

Whenever we have these alternatives, we are leaning on using Event Hubs, both for simplicity and for future-proofing (i.e. a later evolution of the solution requires subscribing to the Event Hubs for near real-time streaming of the same data).

Log filtering will be handled by ingesting all logs into an ADX staging/raw table and using an update policy to filter data to be stored in a final destination table. Data retention policy for the staging/raw table is 0 days so its only used as a hop to filter data.

###Alerts

We will leverage ADX native Logic Apps integration via [Conditional Queries](https://docs.microsoft.com/en-us/azure/data-explorer/flow-usage#conditional-queries)

<IMG  src="https://docs.microsoft.com/en-us/azure/data-explorer/media/flow-usage/flow-conditionactions.png"  alt="Adding actions for when a condition is true or false, flow conditions based on Kusto query results, Azure Data Explorer"/>

###Conceptual Monitoring Arquitecture

![ADX Monitoring.jpg](/.attachments/ADX%20Monitoring-5a4e49ae-4476-43bb-b965-627133355f1b.jpg)

###MVP1 Minimal Implementation for R2.0

During the current release (Sprint 14) we will focus on deploying the ingestion pipelines foundation that will enable alerting in a future release.

- Validate existing [Log Analytics workspace](https://sede-it-itso.visualstudio.com/Digital%20Platform%20MC%20Enablement/_wiki/wikis/Digital-Platform-MC-Enablement.wiki/5353/Logging-to-Log-analytics-workspace) - 8 hrs
- Deploy Azure Function with API query to PowerBI Activity Log - 8 hrs
- Deploy Azure Data Explorer clusters - 8 hrs
- Create target tables for activity / diagnostic logs - 4 hrs
- Configure retention policies - 4 hrs
- Create table mappings - 4 hrs
- Define update policies - 4 hrs
- Create Azure Event Hubs namespaces - 8 hrs
- Connect activity / diagnostic logs to Event Hubs - 8 hrs
- Generate Kusto (KQL) query for reports - 24 hrs

![landscape.jpg](/.attachments/landscape-63134630-71d4-4b5d-934b-c04d1648cfd5.jpg)
