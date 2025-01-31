---
title: Supervisión de la carga de trabajo - Azure Portal | Microsoft Docs
description: Supervisión de Azure SQL Data Warehouse mediante Azure Portal
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: manage
ms.date: 03/22/2019
ms.author: kevin
ms.reviewer: jrasnick
ms.openlocfilehash: 176762e0d1c007f924d779a1b77dd52c7ed56e01
ms.sourcegitcommit: 6d2a147a7e729f05d65ea4735b880c005f62530f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2019
ms.locfileid: "69981482"
---
# <a name="monitor-workload---azure-portal"></a>Supervisión de la carga de trabajo - Azure Portal

En este artículo se describe cómo usar Azure Portal para supervisar la carga de trabajo. Esto incluye la configuración de registros de Azure Monitor para investigar las tendencias de carga de trabajo y ejecución de consultas mediante análisis de registros para [Azure SQL Data Warehouse](https://azure.microsoft.com/blog/workload-insights-with-sql-data-warehouse-delivered-through-azure-monitor-diagnostic-logs-pass/).

## <a name="prerequisites"></a>Requisitos previos

- Suscripción de Azure: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Azure SQL Data Warehouse: Se recopilarán registros para una instancia de SQL Data Warehouse. Si no tiene una instancia de SQL Data Warehouse aprovisionada, consulte las instrucciones de [Creación de una instancia de SQL Data Warehouse](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-get-started-tutorial).

## <a name="create-a-log-analytics-workspace"></a>Creación de un área de trabajo de Log Analytics

Navegar a la hoja Examinar de las áreas de trabajo de Log Analytics y crear un área de trabajo 

![Áreas de trabajo de Log Analytics](media/sql-data-warehouse-monitor/log_analytics_workspaces.png)

![Adición de área de trabajo de Analytics](media/sql-data-warehouse-monitor/add_analytics_workspace.png)

![Adición de área de trabajo de Analytics](media/sql-data-warehouse-monitor/add_analytics_workspace_2.png)

Para obtener más detalles sobre las áreas de trabajo, visite la siguiente [documentación](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace#create-a-workspace).

## <a name="turn-on-diagnostic-logs"></a>Activación de los registros de diagnósticos 

Configure las opciones de diagnóstico para emitir registros desde SQL Data Warehouse. Los registros constan de vistas de telemetría del almacenamiento de datos equivalentes a las DMV de solución de problemas de rendimiento más utilizadas para SQL Data Warehouse. Actualmente se admiten las siguientes vistas:

- [sys.dm_pdw_exec_requests](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?view=aps-pdw-2016-au7)
- [sys.dm_pdw_request_steps](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql?view=aps-pdw-2016-au7)
- [sys.dm_pdw_dms_workers](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-dms-workers-transact-sql?view=aps-pdw-2016-au7)
- [sys.dm_pdw_waits](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-waits-transact-sql?view=aps-pdw-2016-au7)
- [sys.dm_pdw_sql_requests](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-sql-requests-transact-sql?view=aps-pdw-2016-au7)


![Habilitación de registros de diagnóstico](media/sql-data-warehouse-monitor/enable_diagnostic_logs.png)

Los registros se pueden emitir a Azure Storage, Stream Analytics o Log Analytics. Para este tutorial, seleccione Log Analytics.

![Especificar registros](media/sql-data-warehouse-monitor/specify_logs.png)

## <a name="run-queries-against-log-analytics"></a>Ejecución de consultas en Log Analytics

Desplácese hasta el área de trabajo de Log Analytics, donde puede hacer lo siguiente:

- Analizar registros mediante consultas de registros y guardar las consultas para reutilizarlas
- Guardar las consultas para reutilizarlas
- Crear alertas del registro
- Anclar los resultados de la consulta a un panel

Para obtener más información sobre las capacidades de las consultas de registro, visite la siguiente [documentación](https://docs.microsoft.com/azure/azure-monitor/log-query/query-language).

![Editor del área de trabajo de Log Analytics](media/sql-data-warehouse-monitor/log_analytics_workspace_editor.png)



![Consultas del área de trabajo de Log Analytics](media/sql-data-warehouse-monitor/log_analytics_workspace_queries.png)

## <a name="sample-log-queries"></a>Consultas de registros de ejemplo



```Kusto
//List all queries 
AzureDiagnostics
| where Category contains "ExecRequests"
| project TimeGenerated, StartTime_t, EndTime_t, Status_s, Command_s, ResourceClass_s, duration=datetime_diff('millisecond',EndTime_t, StartTime_t)
```
```Kusto
//Chart the most active resource classes
AzureDiagnostics
| where Category contains "ExecRequests"
| where Status_s == "Completed"
| summarize totalQueries = dcount(RequestId_s) by ResourceClass_s
| render barchart 
```
```Kusto
//Count of all queued queries
AzureDiagnostics
| where Category contains "waits" 
| where Type_s == "UserConcurrencyResourceType"
| summarize totalQueuedQueries = dcount(RequestId_s)
```
## <a name="next-steps"></a>Pasos siguientes

Ahora que ha instalado y configurado los registros de Azure Monitor, [personalice los paneles de Azure](https://docs.microsoft.com/azure/azure-portal/azure-portal-dashboards) para compartirlos con su equipo.
