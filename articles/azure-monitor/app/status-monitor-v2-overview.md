---
title: Vue d’ensemble d’Azure Application Insights Agent | Microsoft Docs
description: Vue d’ensemble d’Application Insights Agent. Surveillez les performances de site web sans avoir à redéployer le site web. Fonctionne avec les applications web ASP.NET hébergées en local, dans des machines virtuelles ou sur Azure.
ms.topic: conceptual
author: TimothyMothra
ms.author: tilee
ms.date: 09/16/2019
ms.openlocfilehash: f0ce969a4ab016d6a036d33a0e9bf027bb580a20
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186216"
---
# <a name="deploy-azure-monitor-application-insights-agent-for-on-premises-servers"></a>Déployer Azure Monitor Application Insights Agent pour les serveurs locaux

> [!IMPORTANT]
> Ces informations d’aide sont recommandées pour les déploiements locaux et les déploiements sur les environnements cloud non Azure d’Application Insights Agent. Voici l’approche recommandée pour les [déploiements de machines virtuelles et de groupes de machines virtuelles identiques Azure](./azure-vm-vmss-apps.md).

Application Insights Agent (anciennement appelé Status Monitor V2) est un module PowerShell publié sur [PowerShell Gallery](https://www.powershellgallery.com/packages/Az.ApplicationMonitor).
Il remplace [Status Monitor](./monitor-performance-live-website-now.md).
Les données de télémétrie sont envoyées au Portail Azure, dans lequel vous pouvez [surveiller](./app-insights-overview.md) votre application.

> [!NOTE]
> Le module prend uniquement en charge l’instrumentation sans code des applications web .NET hébergées à l’aide d’IIS. Utilisez un kit SDK pour instrumenter les applications ASP.NET Core, Java et Node.js.

## <a name="powershell-gallery"></a>PowerShell Gallery

Application Insights Agent se trouve ici : https://www.powershellgallery.com/packages/Az.ApplicationMonitor.

![PowerShell Gallery](https://img.shields.io/powershellgallery/v/Az.ApplicationMonitor.svg?color=Blue&label=Current%20Version&logo=PowerShell&style=for-the-badge)


## <a name="instructions"></a>Instructions
- Consultez les [instructions de prise en main](status-monitor-v2-get-started.md) pour vous familiariser avec les exemples de code concis.
- Consultez les [instructions détaillées](status-monitor-v2-detailed-instructions.md) pour une présentation approfondie de la prise en main.

## <a name="powershell-api-reference"></a>Référence API PowerShell
- [Disable-ApplicationInsightsMonitoring](./status-monitor-v2-api-reference.md#disable-applicationinsightsmonitoring)
- [Disable-InstrumentationEngine](./status-monitor-v2-api-reference.md#disable-instrumentationengine)
- [Enable-ApplicationInsightsMonitoring](./status-monitor-v2-api-reference.md#enable-applicationinsightsmonitoring)
- [Enable-InstrumentationEngine](./status-monitor-v2-api-reference.md#enable-instrumentationengine)
- [Get-ApplicationInsightsMonitoringConfig](./status-monitor-v2-api-reference.md#get-applicationinsightsmonitoringconfig)
- [Get-ApplicationInsightsMonitoringStatus](./status-monitor-v2-api-reference.md#get-applicationinsightsmonitoringstatus)
- [Set-ApplicationInsightsMonitoringConfig](./status-monitor-v2-api-reference.md#set-applicationinsightsmonitoringconfig)
- [Start-ApplicationInsightsMonitoringTrace](./status-monitor-v2-api-reference.md#start-applicationinsightsmonitoringtrace)

## <a name="troubleshooting"></a>Dépannage
- [Dépannage](status-monitor-v2-troubleshoot.md)
- [Problèmes connus](status-monitor-v2-troubleshoot.md#known-issues)


## <a name="faq"></a>Questions fréquentes (FAQ)

- Application Insights Agent prend-il en charge les installations basées sur un serveur proxy ?

  *Oui*. Il existe plusieurs façons de télécharger Application Insights Agent. Si votre ordinateur a accès à Internet, vous pouvez intégrer PowerShell Gallery à l’aide des paramètres `-Proxy`.
Vous pouvez également télécharger le module manuellement et l’installer sur votre ordinateur ou l’utiliser directement.
Chacune de ces options est décrite dans les [instructions détaillées](status-monitor-v2-detailed-instructions.md).

- Status Monitor v2 prend-il en charge les applications ASP.NET Core ?

  *Non*. Pour obtenir des instructions sur l’activation de la supervision des applications ASP.NET Core, consultez [Application Insights pour applications ASP.NET Core](./asp-net-core.md). Il n’est pas nécessaire d’installer StatusMonitor pour une application ASP.NET Core. C’est vrai même si l’application ASP.NET Core est hébergée dans IIS.

- Comment vérifier que l’activation a réussi ?

  - Le cmdlet [Get-ApplicationInsightsMonitoringStatus](./status-monitor-v2-api-reference.md#get-applicationinsightsmonitoringstatus) peut être utilisé pour vérifier que l’activation a réussi.
  - Nous vous recommandons d’utiliser les [métriques en temps réel](./live-stream.md) pour déterminer rapidement si votre application envoie les données de télémétrie.

  - Vous pouvez également utiliser [Log Analytics](../log-query/log-analytics-tutorial.md) pour répertorier tous les rôles de cloud qui envoient des données de télémétrie :
      ```Kusto
      union * | summarize count() by cloud_RoleName, cloud_RoleInstance
      ```

## <a name="next-steps"></a>Étapes suivantes

Affichez vos données de télémétrie :

* [Explorez les métriques](../platform/metrics-charts.md) pour surveiller les performances et l’utilisation.
* [Effectuez des recherches dans les événements et les journaux](./diagnostic-search.md) pour diagnostiquer les problèmes.
* [Utilisez la fonctionnalité Analytics](../log-query/log-query-overview.md) pour des requêtes plus élaborées.
* [Créez des tableaux de bord](./overview-dashboard.md).

Ajoutez des données de télémétrie :

* [Créez des tests web](monitor-web-app-availability.md) pour vous assurer que votre site reste actif.
* [Ajoutez la télémétrie de client web](./javascript.md) pour afficher les exceptions à partir du code de la page web et activer le suivi des appels.
* [Ajoutez le kit de développement logiciel (SDK) Application Insights à votre code](./asp-net.md) afin de pouvoir insérer un suivi et journaliser les appels.