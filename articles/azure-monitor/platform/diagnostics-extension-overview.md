---
title: Vue d’ensemble de l’extension Diagnostics Azure
description: Utilisation des diagnostics Azure pour le débogage, la mesure des performances, la surveillance, l’analyse du trafic dans Cloud Services, Virtual Machines et Service Fabric
ms.subservice: diagnostic-extension
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 02/14/2020
ms.openlocfilehash: 6201a4e0551f0f75dde65b2bc4b8b560a0f5ea20
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87007994"
---
# <a name="azure-diagnostics-extension-overview"></a>Vue d’ensemble de l’extension Diagnostics Azure
L’extension Diagnostics Azure est un [agent présent dans Azure Monitor](agents-overview.md) qui collecte des données de supervision dans le système d’exploitation invité des ressources de calcul Azure, notamment des machines virtuelles. Cet article fournit une vue d’ensemble de l’extension Diagnostics Azure, notamment des fonctionnalités spécifiques qu’elle prend en charge ainsi que des options d’installation et de configuration. 

> [!NOTE]
> L’extension Diagnostics Azure est l’un des agents disponibles pour collecter des données de supervision à partir du système d’exploitation invité des ressources de calcul. Pour obtenir une description des différents agents et des conseils sur la sélection des agents appropriés en fonction de vos besoins, consultez [Vue d’ensemble des agents Azure Monitor](agents-overview.md).

## <a name="primary-scenarios"></a>Principaux scénarios
Les principaux scénarios traités par l’extension Diagnostics sont les suivants :

- Collecter des métriques d’invités dans les métriques Azure Monitor.
- Envoyer les journaux et les métriques d’invités au stockage Azure à des fins d’archivage.
- Envoyer les journaux et les métriques d’invité à des hubs d’événements Azure en dehors d’Azure.


## <a name="comparison-to-log-analytics-agent"></a>Comparaison avec l’agent Log Analytics
L’agent Log Analytics dans Azure Monitor peut également être utilisé pour collecter des données de supervision à partir du système d’exploitation invité des machines virtuelles. Vous pouvez choisir d’utiliser l’un ou l’autre, ou les deux en fonction de vos besoins. Pour une comparaison détaillée des agents Azure Monitor, consultez [Vue d’ensemble des agents Azure Monitor](agents-overview.md). 

Les principales différences à prendre en compte sont les suivantes :

- L’extension Diagnostics Azure peut être utilisée uniquement avec des machines virtuelles Azure. L’agent Log Analytics peut être utilisé avec des machines virtuelles dans Azure, dans d’autres clouds et en local.
- L’extension Diagnostics Azure envoie des données à Stockage Azure, à [Azure Monitor Metrics](data-platform-metrics.md) (Windows uniquement) et à Event Hubs. L’agent Log Analytics collecte des données dans des [journaux d’activité Azure Monitor](data-platform-logs.md).
- L’agent Log Analytics est requis pour les [solutions](../monitor-reference.md#insights-and-core-solutions), [Azure Monitor pour machines virtuelles](../insights/vminsights-overview.md) et d’autres services tels qu’[Azure Security Center](../../security-center/index.yml).

## <a name="costs"></a>Coûts
Il n’y a aucun coût lié à l’extension Diagnostics Azure, mais vous pouvez être facturé pour les données ingérées. Consultez les [tarifs Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/) appliqués pour la destination dans laquelle vous collectez les données.

## <a name="data-collected"></a>Données collectées
Les tableaux suivants listent les données qui peuvent être collectées par l’extension de diagnostic Windows et Linux.

### <a name="windows-diagnostics-extension-wad"></a>Extension de diagnostic Windows (WAD)

| source de données | Description |
| --- | --- |
| Journaux d’événements Windows   | Événements du journal des événements Windows. |
| Compteurs de performance | Valeurs numériques mesurant les performances de différents aspects du système d’exploitation et des charges de travail. |
| Journaux d’activité IIS             | Informations sur l’utilisation des sites web IIS s’exécutant sur le système d’exploitation invité. |
| Journaux d’activité d’application     | Messages de trace écrits par votre application. |
| Journaux d’activité .NET EventSource |Événements d’écriture de code à l'aide de la classe .NET [EventSource](/dotnet/api/system.diagnostics.tracing.eventsource?view=netcore-3.1) |
| [Journaux d’activité ETW basés sur les manifestes](/windows/desktop/etw/about-event-tracing) |Événements de suivi d’événements pour Windows générés par n’importe quel processus. |
| Vidages sur incident (journaux d’activité)   | Informations sur l’état du processus si une application plante. |
| Journaux basés sur des fichiers    | Journaux d’activité créés par votre application ou votre service. |
| Journaux de diagnostic de l’agent | Informations sur Diagnostics Azure lui-même. |


### <a name="linux-diagnostics-extension-lad"></a>Extension de diagnostic Linux (LAD)

| source de données | Description |
| --- | --- |
| syslog | Événements envoyés au système de journalisation des événements Linux.   |
| Compteurs de performance  | Valeurs numériques mesurant les performances de différents aspects du système d’exploitation et des charges de travail. |
| Fichiers journaux | Entrées envoyées à un journal basé sur un fichier.  |

## <a name="data-destinations"></a>Destination des données
L’extension Diagnostics Azure pour Windows et Linux collecte toujours les données dans un compte de stockage Azure. Pour obtenir la liste des tables et objets blob dans lesquels ces données sont collectées, consultez [Installer et configurer l’extension Diagnostics Azure pour Windows (WAD)](diagnostics-extension-windows-install.md) et [Utiliser l’extension de diagnostic Linux pour surveiller les métriques et les journaux](../../virtual-machines/extensions/diagnostics-linux.md).

Configurez un ou plusieurs *récepteurs de données* pour envoyer des données à d’autres destinations supplémentaires. Les sections suivantes listent les récepteurs disponibles pour les extensions de diagnostic Windows et Linux.

### <a name="windows-diagnostics-extension-wad"></a>Extension de diagnostic Windows (WAD)

| Destination | Description |
|:---|:---|
| Métriques Azure Monitor | Collecter les données de performances dans les métriques Azure Monitor. Consultez [Envoyer des métriques de système d’exploitation invité à la base de données de métriques Azure Monitor](collect-custom-metrics-guestos-resource-manager-vm.md).  |
| Hubs d'événements | Utiliser Azure Event Hubs pour envoyer des données en dehors d’Azure. Consultez [Streaming des données Diagnostics Azure vers Event Hubs](diagnostics-extension-stream-event-hubs.md) |
| Objets blob de stockage Azure | Écrire les données dans des objets blob au sein du Stockage Azure, en plus de les écrire dans des tables. |
| Application Insights | Collecter les données des applications qui s’exécutent dans votre machine virtuelle vers Application Insights pour les intégrer à d’autres supervisions d’applications. Consultez [Envoyez les données de diagnostic à Application Insights](diagnostics-extension-to-application-insights.md). |

Vous pouvez également collecter des données WAD à partir du stockage dans un espace de travail Log Analytics pour les analyser avec les journaux Azure Monitor, bien que l’agent Log Analytics soit généralement utilisé pour cette fonctionnalité. Il peut envoyer les données directement à un espace de travail Log Analytics et prend en charge des solutions et des insights qui fournissent des fonctionnalités supplémentaires.  Consultez [Collecter les journaux de diagnostic Azure depuis le Stockage Azure](diagnostics-extension-logs.md). 


### <a name="linux-diagnostics-extension-lad"></a>Extension de diagnostic Linux (LAD)
LAD écrit les données dans des tables au sein du Stockage Azure. Elle prend en charge les récepteurs répertoriés dans le tableau suivant.

| Destination | Description |
|:---|:---|
| Hubs d'événements | Utiliser Azure Event Hubs pour envoyer des données en dehors d’Azure. |
| Objets blob de stockage Azure | Écrire les données dans des objets blob au sein du Stockage Azure, en plus de les écrire dans des tables. |
| Métriques Azure Monitor | Installer l’agent Telegraf en plus de LAD. Consultez [Collecter des métriques personnalisées pour une machine virtuelle Linux avec l’agent InfluxData Telegraf](collect-custom-metrics-linux-telegraf.md).


## <a name="installation-and-configuration"></a>Installation et configuration
L’extension de diagnostic étant implémentée en tant qu’[extension de machine virtuelle](../../virtual-machines/extensions/overview.md) dans Azure, elle prend en charge les mêmes options d’installation à l’aide de modèles Gestionnaire des ressources, de PowerShell et de l’interface CLI. Pour plus d’informations sur l’installation et la maintenance des extensions de machine virtuelle, consultez [Extensions et fonctionnalités de machine virtuelle pour Windows](../../virtual-machines/extensions/features-windows.md) et [Extensions et fonctionnalités de machine virtuelle pour Linux](../../virtual-machines/extensions/features-linux.md).

Vous pouvez également installer et configurer les deux extensions de diagnostic Windows et Linux dans le portail Azure sous **Paramètres de diagnostic** dans la section **Supervision** du menu de la machine virtuelle.

Pour plus d’informations sur l’installation et la configuration des extensions de diagnostic pour Windows et Linux, consultez les articles suivants.

- [Installer et configurer l’extension Diagnostics Azure pour Windows (WAD)](diagnostics-extension-windows-install.md)
- [Utiliser l’extension de diagnostic Linux pour superviser les métriques et les journaux](../../virtual-machines/extensions/diagnostics-linux.md)

## <a name="other-documentation"></a>Autre documentation

###  <a name="azure-cloud-service-classic-web-and-worker-roles"></a>Rôles de travail et web d’Azure Cloud Service (classique)
- [Introduction à la surveillance des services cloud](../../cloud-services/cloud-services-how-to-monitor.md)
- [Activation des diagnostics Azure dans Azure Cloud Services](../../cloud-services/cloud-services-dotnet-diagnostics.md)
- [Application Insights pour les services cloud Azure](../app/cloudservices.md)<br>[Assurer le suivi du flux dans une application Cloud Services avec les diagnostics Azure](../../cloud-services/cloud-services-dotnet-diagnostics-trace-flow.md) 

### <a name="azure-service-fabric"></a>Azure Service Fabric
- [Surveillance et diagnostic des services dans une configuration de développement d’ordinateur local](../../service-fabric/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

## <a name="next-steps"></a>Étapes suivantes


* Apprenez à [utiliser les compteurs de performances dans les diagnostics Azure](../../cloud-services/diagnostics-performance-counters.md).
* Si vous rencontrez des difficultés avec le lancement des diagnostics ou la recherche des données dans les tables de stockage Azure, consultez [Résolution des problèmes des diagnostics Azure](diagnostics-extension-troubleshooting.md).
