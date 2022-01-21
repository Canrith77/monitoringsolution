**Requirement** 
An end-to-end comprehensive monitoring solution will support the MC environment, providing extensive alerting and reporting capabilities.

[[_TOC_]]

###Objectives
- Centralized solution to enable proactive alerts after the collection of event and diagnostics data from multiple resources
- Different ingestion methods depending on the export/integration capabilities of source and destination

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
