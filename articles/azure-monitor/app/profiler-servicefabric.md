---
title: Profiler des applications Azure Service Fabric en direct avec Application Insights
description: Activer le profileur pour une application Service Fabric
ms.topic: conceptual
ms.custom: devx-track-dotnet
author: cweining
ms.author: cweining
ms.date: 08/06/2018
ms.reviewer: mbullwin
ms.openlocfilehash: 317c02fe03a56ea3cd3a98b00f8a0080b3917108
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88930279"
---
# <a name="profile-live-azure-service-fabric-applications-with-application-insights"></a>Profiler des applications Azure Service Fabric en production avec Application Insights

Vous pouvez également déployer Application Insights Profiler sur les services suivants :
* [Azure App Service](profiler.md?toc=/azure/azure-monitor/toc.json)
* [Azure Cloud Services](profiler-cloudservice.md?toc=/azure/azure-monitor/toc.json)
* [Machines virtuelles Azure](profiler-vm.md?toc=/azure/azure-monitor/toc.json)

## <a name="set-up-the-environment-deployment-definition"></a>Configurer la définition du déploiement d’environnement

Application Insights Profiler est fourni avec Diagnostics Azure. Vous pouvez installer l'extension Diagnostics Azure à l'aide d'un modèle Azure Resource Manager pour votre cluster Service Fabric. Procurez-vous un [modèle qui installe Diagnostics Azure sur un cluster Service Fabric](https://github.com/Azure/azure-docs-json-samples/blob/master/application-insights/ServiceFabricCluster.json).

Pour configurer votre environnement, effectuez les actions suivantes :

1. Le profileur prend en charge .NET Framework et .NET Core. Si vous utilisez .NET Framework, vérifiez que vous utilisez [.NET Framework 4.6.1](/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed) ou une version ultérieure. Il vous suffit de vérifier que le système d’exploitation déployé est `Windows Server 2012 R2` ou une version ultérieure. Le profileur prend en charge .NET Core 2.1 et applications plus récentes.

1. Recherchez l'extension [Diagnostics Azure](../platform/diagnostics-extension-overview.md) dans le fichier du modèle de déploiement.

1. Ajoutez la section `SinksConfig` suivante comme élément enfant de `WadCfg`. Remplacez la valeur de propriété `ApplicationInsightsProfiler` par votre propre clé d’instrumentation Application Insights :  

      ```json
      "SinksConfig": {
        "Sink": [
          {
            "name": "MyApplicationInsightsProfilerSink",
            "ApplicationInsightsProfiler": "00000000-0000-0000-0000-000000000000"
          }
        ]
      }
      ```

      Pour plus d’informations sur l’ajout de l’extension Diagnostics à votre modèle de déploiement, consultez [Utiliser la surveillance et les diagnostics avec une machine virtuelle Windows et des modèles Azure Resource Manager](../../virtual-machines/extensions/diagnostics-template.md?toc=/azure/virtual-machines/windows/toc.json).

1. Déployez votre cluster Service Fabric à l'aide de votre modèle Azure Resource Manager.  
  Si vos paramètres sont corrects, Application Insights Profiler sera installé et activé une fois l'extension Diagnostics Azure installée. 

1. Ajoutez Application Insights à votre application Service Fabric.  
  Pour que le profileur collecte des profils pour vos requêtes, votre application doit suivre les opérations avec Application Insights. Pour les API sans état, vous pouvez consulter les instructions de [suivi des requêtes pour le profilage](profiler-trackrequests.md?toc=/azure/azure-monitor/toc.json). Pour plus d’informations sur le suivi des opérations personnalisées dans d’autres types d’applications, consultez [Suivi des opérations personnalisées avec le kit SDK .NET d’Application Insights](custom-operations-tracking.md?toc=/azure/azure-monitor/toc.json).

1. Redéployez votre application.


## <a name="next-steps"></a>Étapes suivantes

* Générez le trafic vers votre application (par exemple, lancez un [test de disponibilité](monitor-web-app-availability.md)). Attendez ensuite 10 à 15 minutes le temps que les traces soient envoyées à l’instance Application Insights.
* Consultez [Traces du profileur](profiler-overview.md?toc=/azure/azure-monitor/toc.json) dans le portail Azure.
* Pour obtenir de l'aide sur la résolution des problèmes liés au profileur, consultez [Résolution des problèmes liés au profileur](profiler-troubleshooting.md?toc=/azure/azure-monitor/toc.json).
