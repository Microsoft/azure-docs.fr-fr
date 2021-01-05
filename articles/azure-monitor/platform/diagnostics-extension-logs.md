---
title: Utilisation d’un Stockage Blob pour IIS et d’un Stockage Table pour les événements dans Azure Monitor | Microsoft Docs
description: Azure Monitor peut lire les journaux d’activité pour des services Azure qui écrivent des diagnostics dans un Stockage Table ou des journaux d’activité IIS écrits dans un Stockage Blob.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 02/14/2020
ms.openlocfilehash: 095fd0b534c0dffaf80d2464fb9734f295335b84
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87317176"
---
# <a name="collect-data-from-azure-diagnostics-extension-to-azure-monitor-logs"></a>Collecter des données de l’extension de diagnostic Azure et les enregistrer dans des journaux d’activité Azure Monitor
L’extension de diagnostic Azure est un [agent présent dans Azure Monitor](agents-overview.md) qui collecte des données d’analyse dans le système d’exploitation invité de ressources de calcul Azure, notamment des machines virtuelles. Cet article explique comment collecter des données recueillies par l’extension de diagnostic dans Stockage Azure et les enregistrer des journaux d’activité Azure Monitor.

> [!NOTE]
> L’agent Log Analytics dans Azure Monitor est généralement la méthode recommandée pour collecter des données du système d’exploitation invité et les enregistrer dans des journaux d’activité Azure Monitor. Pour une comparaison détaillée des agents, consultez [Vue d’ensemble des agents Azure Monitor](agents-overview.md).

## <a name="supported-data-types"></a>Types de données pris en charge
L’extension de diagnostic Azure stocke des données dans un compte Stockage Azure. Pour que des journaux d’activité Azure Monitor collectent ces données, ceux-ci doivent se trouver dans les emplacements suivants :

| Type de journal | Type de ressource | Location |
| --- | --- | --- |
| Journaux d’activité IIS |Virtual Machines <br> Rôles web <br> Rôles de travail |wad-iis-logfiles (Stockage Blob) |
| syslog |Virtual Machines |LinuxSyslogVer2v0 (Stockage Table) |
| Événements opérationnels Service Fabric |Nœuds Service Fabric |WADServiceFabricSystemEventTable |
| Événements Reliable Actor Service Fabric |Nœuds Service Fabric |WADServiceFabricReliableActorEventTable |
| Événements de service fiable Service Fabric |Nœuds Service Fabric |WADServiceFabricReliableServiceEventTable |
| Journaux d’événements Windows |Nœuds Service Fabric <br> Virtual Machines <br> Rôles web <br> Rôles de travail |WADWindowsEventLogsTable (Stockage Table) |
| Journaux d’activité de suivi des événements ETW Windows |Nœuds Service Fabric <br> Virtual Machines <br> Rôles web <br> Rôles de travail |WADETWEventTable (Stockage Table) |

## <a name="data-types-not-supported"></a>Types de données non pris en charge

- Données de performances du système d’exploitation invité
- Journaux d’activité IIS provenant de sites Azure


## <a name="enable-azure-diagnostics-extension"></a>Activer l’extension de diagnostic Azure
Pour plus d’informations sur l’installation et la configuration de l’extension de diagnostic, consultez [Installer et configurer l’extension de diagnostic Windows Azure (WAD)](diagnostics-extension-windows-install.md) ou [Utiliser l’extension de diagnostic Linux pour surveiller les métriques et les journaux d’activité](../../virtual-machines/extensions/diagnostics-linux.md). Cela vous permettra de spécifier le compte de stockage et de configurer la collecte des données que vous souhaitez transmettre aux journaux d’activité Azure Monitor.


## <a name="collect-logs-from-azure-storage"></a>Collecter des journaux d’activité depuis Stockage Azure
Utilisez la procédure suivante pour activer la collecte des données d’extension de diagnostic à partir d’un compte Stockage Azure :

1. Sur le Portail Azure, accédez à **Espaces de travail Log Analytics** et sélectionnez votre espace de travail.
1. Cliquez sur **Journaux de comptes de stockage** dans la section **Sources de données de l’espace de travail** du menu.
2. Cliquez sur **Ajouter**.
3. Sélectionnez le **compte de stockage** qui contient les données à collecter.
4. Sélectionnez le **type de données** que vous souhaitez collecter.
5. La valeur de Source est automatiquement complétée selon le type de données.
6. Cliquez sur **OK** pour enregistrer la configuration.
7. Répétez cette opération pour les autres types de données.

Après environ 30 minutes, vous pourrez voir les données du compte de stockage dans l’espace de travail Log Analytics. Vous voyez les données écrites dans le stockage uniquement après application de la configuration. L’espace de travail ne lit pas les données préexistantes du compte de stockage.

> [!NOTE]
> Le portail ne valide pas l’existence de la source dans le compte de stockage ou le fait que de nouvelles données sont écrites.



## <a name="next-steps"></a>Étapes suivantes

* [Collecter les journaux d’activité et les indicateurs de performance des services Azure](./resource-logs.md#send-to-log-analytics-workspace) pour les services pris en charge par Azure.
* [Activer les solutions](../insights/solutions.md) pour fournir des informations sur les données.
* [Utiliser les requêtes de recherche](../log-query/log-query-overview.md) pour analyser les données.

