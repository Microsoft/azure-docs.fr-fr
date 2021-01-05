---
title: Collecter les compteurs de performances dans Azure Cloud Services | Microsoft Docs
description: Découvrez comment découvrir, utiliser et créer des compteurs de performances dans Azure Cloud Services avec Diagnostics Azure et Application Insights.
services: cloud-services
documentationcenter: .net
author: tgore03
ms.service: cloud-services
ms.topic: article
ms.date: 02/02/2018
ms.author: tagore
ms.openlocfilehash: 39843ad83830a72b5d6b01cc00ecd65269c02e12
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92078593"
---
# <a name="collect-performance-counters-for-your-azure-cloud-service"></a>Collecter les compteurs de performances pour votre Azure Cloud Service

Les compteurs de performances vous permettent de suivre les performances de votre application et de l’hôte. Windows Server fournit de nombreux compteurs de performances différents liés au matériel, aux applications, au système d’exploitation, et bien plus encore. En collectant et en envoyant des compteurs de performances à Azure, vous pouvez analyser ces informations pour contribuer à la prise de meilleures décisions. 

## <a name="discover-available-counters"></a>Découvrir les compteurs disponibles

Un compteur de performances est constitué de deux parties : un nom d’ensemble de compteurs (également appelé catégorie) et un ou plusieurs compteurs. Vous pouvez utiliser PowerShell pour obtenir la liste des compteurs de performances disponibles :

```powershell
Get-Counter -ListSet * | Select-Object CounterSetName, Paths | Sort-Object CounterSetName

CounterSetName                                  Paths
--------------                                  -----
.NET CLR Data                                   {\.NET CLR Data(*)\SqlClient...
.NET CLR Exceptions                             {\.NET CLR Exceptions(*)\# o...
.NET CLR Interop                                {\.NET CLR Interop(*)\# of C...
.NET CLR Jit                                    {\.NET CLR Jit(*)\# of Metho...
.NET Data Provider for Oracle                   {\.NET Data Provider for Ora...
.NET Data Provider for SqlServer                {\.NET Data Provider for Sql...
.NET Memory Cache 4.0                           {\.NET Memory Cache 4.0(*)\C...
AppV Client Streamed Data Percentage            {\AppV Client Streamed Data ...
ASP.NET                                         {\ASP.NET\Application Restar...
ASP.NET Apps v4.0.30319                         {\ASP.NET Apps v4.0.30319(*)...
ASP.NET State Service                           {\ASP.NET State Service\Stat...
ASP.NET v2.0.50727                              {\ASP.NET v2.0.50727\Applica...
ASP.NET v4.0.30319                              {\ASP.NET v4.0.30319\Applica...
Authorization Manager Applications              {\Authorization Manager Appl...

#... results cut to save space ...
```

La propriété `CounterSetName` représente un ensemble de compteurs (ou catégorie), et est un bon indicateur de ce à quoi on trait les compteurs de performances. La propriété `Paths` représente une collection de compteurs pour un ensemble. Vous pouvez également obtenir la propriété `Description` pour plus d’informations sur l’ensemble de compteurs.

Pour obtenir tous les compteurs d’un ensemble, utilisez la valeur `CounterSetName` et développez la collection `Paths`. Chaque élément du chemin d’accès est un compteur que vous pouvez interroger. Par exemple, pour obtenir les compteurs disponibles relatifs à l’ensemble de compteurs `Processor`, développez la collection `Paths` :

```powershell
Get-Counter -ListSet * | Where-Object CounterSetName -eq "Processor" | Select -ExpandProperty Paths

\Processor(*)\% Processor Time
\Processor(*)\% User Time
\Processor(*)\% Privileged Time
\Processor(*)\Interrupts/sec
\Processor(*)\% DPC Time
\Processor(*)\% Interrupt Time
\Processor(*)\DPCs Queued/sec
\Processor(*)\DPC Rate
\Processor(*)\% Idle Time
\Processor(*)\% C1 Time
\Processor(*)\% C2 Time
\Processor(*)\% C3 Time
\Processor(*)\C1 Transitions/sec
\Processor(*)\C2 Transitions/sec
\Processor(*)\C3 Transitions/sec
```

Ces chemins d’accès de compteurs individuels peuvent être ajoutés à l’infrastructure de diagnostics que votre service cloud utilise. Pour plus d’informations sur la construction d’un chemin d’accès au compteur de performances, voir [Spécification d’un chemin d’accès au compteur](/windows/win32/perfctrs/specifying-a-counter-path).

## <a name="collect-a-performance-counter"></a>Collecter un compteur de performances

Un compteur de performances peut être ajouté à votre service cloud pour Diagnostics Azure ou Application Insights.

### <a name="application-insights"></a>Application Insights

Azure Application Insights pour Cloud Services vous permet de spécifier les compteurs de performances que vous souhaitez collecter. Après l’[ajout d’Application Insights à votre projet](../azure-monitor/app/cloudservices.md#sdk), un fichier de configuration nommé **ApplicationInsights.config** est ajouté à votre projet Visual Studio. Ce fichier de configuration définit le type d’informations qu’Application Insights collecte et envoie à Azure.

Ouvrez le fichier **ApplicationInsights.config** et recherchez l’élément **ApplicationInsights** > **TelemetryModules**. Chaque élément enfant `<Add>` définit un type de télémétrie à collecter, ainsi que sa configuration. Le type de module de télémétrie de compteur de performances est `Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule, Microsoft.AI.PerfCounterCollector`. Si cet élément est déjà défini, ne l’ajoutez pas une deuxième fois. Chaque compteur de performances à collecter est défini sous un nœud nommé `<Counters>`. Voici un exemple qui collecte des compteurs de performances de lecteur :

```xml
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings">

  <TelemetryModules>

    <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule, Microsoft.AI.PerfCounterCollector">
      <Counters>
        <Add PerformanceCounter="\LogicalDisk(C:)\Disk Write Bytes/sec" ReportAs="Disk write (C:)" />
        <Add PerformanceCounter="\LogicalDisk(C:)\Disk Read Bytes/sec" ReportAs="Disk read (C:)" />
      </Counters>
    </Add>

  </TelemetryModules>

<!-- ... cut to save space ... -->
```

Chaque compteur de performances est représenté en tant qu’élément `<Add>` sous `<Counters>`. L’attribut `PerformanceCounter` définit le compteur de performances à collecter. L’attribut `ReportAs` est le titre à afficher dans le portail Azure pour le compteur de performances. Tout compteur de performances que vous collectez est placé dans une catégorie nommée **Personnalisé** dans le portail. Contrairement à Diagnostics Azure, vous ne pouvez pas définir l’intervalle auquel ces compteurs de performances sont collectés et envoyés à Azure. Avec Application Insights, les compteurs de performances sont collectés et envoyés toutes les minutes. 

Application Insights collecte automatiquement des compteurs de performances suivants :

* \Processus(??APP_WIN32_PROC??)\% Temps processeur
* \Memory\Octets disponibles
* \.Exceptions .NET CLR(??APP_CLR_PROC??)\# Nombre d'exceptions levées/s
* \Processus(??APP_WIN32_PROC??)\Octets privés
* \Processus(??APP_WIN32_PROC??)\Nombre d’octets de données E/S par s
* \Processor(_Total)\% temps processeur

Pour plus d’informations, voir [Compteurs de performances système dans Application Insights](../azure-monitor/app/performance-counters.md) et [Application Insights pour Azure Cloud Services](../azure-monitor/app/cloudservices.md#performance-counters).

### <a name="azure-diagnostics"></a>Diagnostics Azure

> [!IMPORTANT]
> Alors que toutes ces données sont regroupées dans le compte de stockage, le portail ne fournit **pas** un moyen natif pour les représenter visuellement. Il est fortement recommandé d’intégrer un autre service tel qu’Application Insights dans votre application.

L’extension Diagnostics Azure pour Cloud Services vous permet de que spécifier les compteurs de performances que vous souhaitez collecter. Pour configurer Diagnostics Azure, voir [Vue d’ensemble de la surveillance de service cloud](cloud-services-how-to-monitor.md#setup-diagnostics-extension).

Les compteurs de performances à collecter sont définis dans le fichier **diagnostics.wadcfgx**. Ouvrez ce fichier (il est défini par le rôle) dans Visual Studio, puis recherchez l’élément **DiagnosticsConfiguration** > **PublicConfig** > **WadCfg** > **DiagnosticMonitorConfiguration** > **PerformanceCounters**. Ajoutez un nouvel élément **PerformanceCounterConfiguration** en tant qu’enfant. Cet élément a deux attributs : `counterSpecifier` et `sampleRate`. L’attribut `counterSpecifier` définit l’ensemble de compteurs de performances système (indiqué dans la section précédente) à collecter. La valeur `sampleRate` indique la fréquence à laquelle cette valeur est interrogée. Dans l’ensemble, tous les compteurs de performances sont transférés vers Azure en fonction de la valeur d’attribut `PerformanceCounters` de l’élément `scheduledTransferPeriod` parent.

Pour plus d’informations sur l’élément de schéma `PerformanceCounters`, voir le [schéma Diagnostics Azure](../azure-monitor/platform/diagnostics-extension-schema-windows.md#performancecounters-element).

La période définie par l’attribut `sampleRate` utilise le type de données de durée XML pour indiquer la fréquence à laquelle le compteur de performances est interrogé. Dans l’exemple ci-dessous, la fréquence est définie sur `PT3M`, ce qui signifie `[P]eriod[T]ime[3][M]inutes` : toutes les trois minutes.

Pour plus d’informations sur la façon dont les valeurs `sampleRate` et `scheduledTransferPeriod` sont définies, voir la section **Duration Data Type** (Type de données de date) du didacticiel [W3 XML Date and Time Date Types](https://www.w3schools.com/XML/schema_dtypes_date.asp) (Types de données de date et d’heure XML W3).

```xml
<?xml version="1.0" encoding="utf-8"?>
<DiagnosticsConfiguration  xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
  <PublicConfig>
    <WadCfg>
      <DiagnosticMonitorConfiguration overallQuotaInMB="4096">

        <!-- ... cut to save space ... -->

        <PerformanceCounters scheduledTransferPeriod="PT1M">
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\ISAPI Extension Requests/sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\Bytes Total/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Requests/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Errors Total/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Queued" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Rejected" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT3M" />

          <!-- This is a new perf counter which will track the C: disk read activity in bytes per second, every minute -->
          <PerformanceCounterConfiguration counterSpecifier="\LogicalDisk(C:)\Disk Read Bytes/sec" sampleRate="PT1M" />

        </PerformanceCounters>
      </DiagnosticMonitorConfiguration>
    </WadCfg>
    
    <!-- ... cut to save space ... -->

  </PublicConfig>
</DiagnosticsConfiguration>
```

## <a name="create-a-new-perf-counter"></a>Créer un compteur de performances

Votre code peut créer et utiliser un nouveau compteur de performances. Le code créant un compteur de performances doit s’exécuter avec élévation de privilèges, sans quoi son exécution échoue. Le code de démarrage `OnStart` de votre service cloud peut créer le compteur de performances, ce qui exige que vous exécutiez le rôle dans un contexte avec élévation de privilèges. Vous pouvez également créer une tâche de démarrage qui s’exécute avec élévation de privilèges et crée le compteur de performances. Pour plus d’informations, voir [Comment configurer et exécuter des tâches de démarrage pour un service cloud](cloud-services-startup-tasks.md).

Pour configurer votre rôle afin qu’il s’exécute avec élévation de privilèges, ajoutez un élément `<Runtime>` au fichier [.csdef](cloud-services-model-and-package.md#servicedefinitioncsdef).

```xml
<ServiceDefinition name="CloudServiceLoadTesting" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2015-04.2.6">
  <WorkerRole name="WorkerRoleWithSBQueue1" vmsize="Large">

    <!-- ... cut to save space ... -->

    <Runtime executionContext="elevated">
      
    </Runtime>

    <!-- ... cut to save space ... -->

  </WorkerRole>
</ServiceDefinition>
```

Vous pouvez créer et inscrire un compteur de performances avec quelques lignes de code. Utilisez la surcharge de méthode `System.Diagnostics.PerformanceCounterCategory.Create` qui crée la catégorie et le compteur. Le code suivant vérifie d’abord si la catégorie existe et, dans la négative, crée la catégorie et le compteur.

```csharp
using System.Diagnostics;
using Microsoft.WindowsAzure;
using Microsoft.WindowsAzure.ServiceRuntime;

namespace WorkerRoleWithSBQueue1
{
    public class WorkerRole : RoleEntryPoint
    {
        // Perf counter variable representing times service was used.
        private PerformanceCounter counterServiceUsed;

        public override bool OnStart()
        {
            // ... Other startup code here ...

            // Define the category and counter names.
            string perfCounterCatName = "MyService";
            string perfCounterName = "Times Used";

            // Create the counter if needed. Our counter category only has a single counter.
            // Both the category and counter are created in the same method call.
            if (!PerformanceCounterCategory.Exists(perfCounterCatName))
            {
                PerformanceCounterCategory.Create(perfCounterCatName, "Collects information about the cloud service.", 
                                                  PerformanceCounterCategoryType.SingleInstance, 
                                                  perfCounterName, "How many times the cloud service was used.");
            }

            // Get reference to our counter
            counterServiceUsed = new PerformanceCounter(perfCounterCatName, perfCounterName);
            counterServiceUsed.ReadOnly = false;
            
            return base.OnStart();
        }

        // ... cut class code to save space
    }
}
```

Lorsque vous souhaitez utiliser le compteur, appelez la méthode `Increment` ou `IncrementBy`.

```csharp
// Increase the counter by 1
counterServiceUsed.Increment();
```

À présent que votre application utilise votre compteur personnalisé, vous devez configurer Diagnostics Azure ou Application Insights pour suivre le compteur.


### <a name="application-insights"></a>Application Insights

Comme indiqué précédemment, les compteurs de performances pour Application Insights sont définis dans le fichier **ApplicationInsights.config**. Ouvrez **ApplicationInsights.config** et recherchez l’élément **ApplicationInsights** > **TelemetryModules** > **Add** > **Counters**. Créez un élément enfant `<Add>` et définissez l’attribut `PerformanceCounter` sur la catégorie et le nom du compteur de performances que vous avez créé dans votre code. Définissez l’attribut `ReportAs` sur un nom convivial que vous souhaitez afficher dans le portail.

```xml
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings">

  <TelemetryModules>

    <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule, Microsoft.AI.PerfCounterCollector">
      <Counters>
        <!-- ... cut other perf counters to save space ... -->

        <!-- This new perf counter matches the [category name]\[counter name] defined in your code -->
        <Add PerformanceCounter="\MyService\Times Used" ReportAs="Service used counter" />
      </Counters>
    </Add>

  </TelemetryModules>

<!-- ... cut to save space ... -->
```

### <a name="azure-diagnostics"></a>Diagnostics Azure

Comme indiqué précédemment, les compteurs de performances à collecter sont définis dans le fichier **diagnostics.wadcfgx**. Ouvrez ce fichier (il est défini par le rôle) dans Visual Studio, puis recherchez l’élément **DiagnosticsConfiguration** > **PublicConfig** > **WadCfg** > **DiagnosticMonitorConfiguration** > **PerformanceCounters**. Ajoutez un nouvel élément **PerformanceCounterConfiguration** en tant qu’enfant. Définissez l’attribut `counterSpecifier` sur la catégorie et le nom du compteur de performances que vous avez créé dans votre code. 

```xml
<?xml version="1.0" encoding="utf-8"?>
<DiagnosticsConfiguration  xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
  <PublicConfig>
    <WadCfg>
      <DiagnosticMonitorConfiguration overallQuotaInMB="4096">

        <!-- ... cut to save space ... -->

        <PerformanceCounters scheduledTransferPeriod="PT1M">
          <!-- ... cut other perf counters to save space ... -->
          
          <!-- This new perf counter matches the [category name]\[counter name] defined in your code -->
          <PerformanceCounterConfiguration counterSpecifier="\MyService\Times Used" sampleRate="PT1M" />

        </PerformanceCounters>
      </DiagnosticMonitorConfiguration>
    </WadCfg>
    
    <!-- ... cut to save space ... -->

  </PublicConfig>
</DiagnosticsConfiguration>
```

## <a name="more-information"></a>Informations complémentaires

- [Application Insights pour Azure Cloud Services](../azure-monitor/app/cloudservices.md#performance-counters)
- [Compteurs de performances système dans Application Insights](../azure-monitor/app/performance-counters.md)
- [Spécification d’un chemin d’accès au compteur](/windows/win32/perfctrs/specifying-a-counter-path)
- [Schéma Diagnostics Azure - Compteurs de performances](../azure-monitor/platform/diagnostics-extension-schema-windows.md#performancecounters-element)