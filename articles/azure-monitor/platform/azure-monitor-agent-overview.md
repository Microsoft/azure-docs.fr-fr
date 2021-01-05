---
title: Vue d’ensemble de l’agent Azure Monitor
description: Vue d’ensemble de l’agent Azure Monitor (AMA), qui collecte des données de supervision à partir du système d’exploitation invité des machines virtuelles.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/10/2020
ms.openlocfilehash: cf64deb17bea508637debb5612231d355d523fbb
ms.sourcegitcommit: 5ae2f32951474ae9e46c0d46f104eda95f7c5a06
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95315581"
---
# <a name="azure-monitor-agent-overview-preview"></a>Vue d’ensemble de l’agent Azure Monitor (préversion)
L’agent Azure Monitor (AMA) collecte des données de supervision à partir du système d’exploitation invité des machines virtuelles et les remet à Azure Monitor. Cet article fournit une vue d’ensemble de l’agent Azure Monitor, notamment son installation et la configuration de la collecte de données.

## <a name="relationship-to-other-agents"></a>Relation avec les autres agents
L’agent Azure Monitor remplace les agents suivants qui sont actuellement utilisés par Azure Monitor pour collecter des données d’invités à partir de machines virtuelles :

- [Agent Log Analytics](log-analytics-agent.md) : envoie des données à l’espace de travail Log Analytics et prend en charge les solutions d’Azure Monitor pour machines virtuelles et de surveillance.
- [Extension Diagnostic](diagnostics-extension-overview.md) : envoie des données aux Métriques Azure Monitor (Windows uniquement), à Azure Event Hubs et au Stockage Azure.
- [Agent Telegraf](collect-custom-metrics-linux-telegraf.md) : envoie des données aux métriques Azure Monitor (Linux uniquement).

En plus de consolider cette fonctionnalité en un seul agent, l’agent Azure Monitor offre les avantages suivants par rapport aux agents existants :

- Étendue de l’analyse. Configurez de manière centralisée un regroupement de différents jeux de données à partir de différents ensembles de machines virtuelles.  
- Multihébergement Linux : Envoyer des données à partir de machines virtuelles Linux vers plusieurs espaces de travail.
- Filtrage d’événements Windows : Utilisez des requêtes XPATH pour filtrer les événements Windows qui sont collectés.
- Gestion améliorée des extensions : L’agent Azure Monitor utilise une nouvelle méthode de gestion de l’extensibilité qui est plus transparente et contrôlable que les packs d’administration et les plug-ins Linux dans les agents Log Analytics actuels.

### <a name="changes-in-data-collection"></a>Modifications de la collecte de données
Les méthodes permettant de définir la collecte de données pour les agents existants diffèrent les unes des autres, et chacune d’entre elles présente des défis qui sont résolus avec l’agent Azure Monitor.

- L’agent Log Analytics obtient sa configuration à partir d’un espace de travail Log Analytics. La configuration de manière centralisée est facile, mais il est difficile de définir des définitions indépendantes pour des machines virtuelles différentes. Il peut uniquement envoyer des données à un espace de travail Log Analytics.

- L’extension de diagnostic a une configuration pour chaque machine virtuelle. Définir des définitions indépendantes pour différentes machines virtuelles est facile, mais il est difficile de les gérer de manière centralisée. Il peut uniquement envoyer des données aux métriques Azure Monitor, à Azure Event Hubs ou au Stockage Azure. Pour les agents Linux, l’agent Telegraf open source est nécessaire pour envoyer des données aux métriques Azure Monitor.

L’agent Azure Monitor utilise des [Règles de collecte de données (DCR)](data-collection-rule-overview.md) pour configurer les données à collecter à partir de chaque agent. Les règles de collecte de données permettent de gérer les paramètres de regroupement à grande échelle tout en autorisant des configurations uniques et étendues pour les sous-ensembles d’ordinateurs. Elles ne dépendent pas de l’espace de travail et de la machine virtuelle, ce qui leur permet d’être définies une fois réutilisées par les ordinateurs et environnements. Consultez [Configurer la collecte de données pour l’agent Azure Monitor (version préliminaire)](data-collection-rule-azure-monitor-agent.md).

## <a name="should-i-switch-to-azure-monitor-agent"></a>Dois-je basculer sur l’agent Azure Monitor ?
L’agent Azure Monitor coexiste avec les [agents généralement disponibles pour Azure Monitor](agents-overview.md), mais vous pouvez envisager de retirer vos machines virtuelles des agents actuels pendant la période de préversion publique de l’agent Azure Monitor. Tenez compte des facteurs suivants lorsque vous prenez cette décision.

- **Spécifications de l’environnement.** L’agent Azure Monitor dispose d’un ensemble de systèmes d’exploitation, d’environnements et de besoins de mise en réseau pris en charge plus limité que les agents actuels. Toute future prise en charge d’environnement telle que les nouvelles versions de système d’exploitation et les types de configuration de mise en réseau sera très probablement fournie uniquement dans l’agent Azure Monitor. Vous devez déterminer si votre environnement est pris en charge ou non par l’agent Azure Monitor. Si ce n’est pas le cas, vous devez conserver l’agent actuel. Si l’agent Azure Monitor prend en charge votre environnement actuel, vous devriez envisager une transition vers celui-ci.
- **Tolérance aux risques de la préversion publique.** Bien que l’agent Azure Monitor ait été testé minutieusement pour les scénarios actuellement pris en charge, l’agent est toujours disponible en préversion publique. Les mises à jour de version et les améliorations des fonctionnalités seront fréquentes et peuvent introduire des bogues. Vous devez évaluer le risque d’un bogue dans l’agent sur vos machines virtuelles susceptible d’arrêter la collecte de données. Si une lacune dans la collecte de données ne risque pas d’avoir un impact significatif sur vos services, poursuivez avec l’agent Azure Monitor. Si vous avez une faible tolérance pour toute instabilité, vous devez conserver les agents mis à la disposition générale jusqu’à ce que l’agent Azure Monitor atteigne cet état.
- **Spécifications des fonctionnalités actuelles et nouvelles.** L’agent Azure Monitor introduit plusieurs nouvelles capacités telles que le filtrage, l’étendue et l’hébergement multiple, mais il n’atteint pas encore un pied d’égalité avec les agents actuels pour d’autres fonctionnalités telles que la collecte de journaux personnalisés et l’intégration à des solutions. La plupart des nouvelles capacités d’Azure Monitor ne seront disponibles qu’avec l’agent Azure Monitor. Par conséquent, au fil du temps, d’autres fonctionnalités seront uniquement disponibles dans le nouvel agent. Vous devez déterminer si l’agent Azure Monitor dispose des fonctionnalités dont vous avez besoin et s’il existe certaines fonctionnalités dont vous pouvez temporairement vous passer pour obtenir d’autres fonctionnalités importantes dans le nouvel agent. Si l’agent Azure Monitor dispose de toutes les capacités de base dont vous avez besoin, envisagez de migrer vers celui-ci. Si vous avez besoin de fonctionnalités critiques, continuez avec l’agent actuel jusqu’à ce que l’agent Azure Monitor atteigne un pied d’égalité.
- **Tolérance au remaniement.** Si vous configurez un nouvel environnement avec des ressources telles que des scripts de déploiement et des modèles d’intégration, vous devez déterminer si vous serez en mesure de les remanier lorsque l’agent Azure Monitor sera mis à la disposition générale. Si l’effort pour ce remaniement est minime, restez sur les agents actuels pour le moment. Si cela représente une quantité importante de travail, envisagez de configurer votre nouvel environnement avec le nouvel agent. L’agent Azure Monitor devrait être mis à la disposition générale et une date de dépréciation être publiée pour les agents Log Analytics en 2021. Les agents actuels seront pris en charge pendant plusieurs années après le début de la dépréciation.



## <a name="current-limitations"></a>Limites actuelles
Les limitations suivantes s’appliquent pendant la préversion publique de l’Agent Azure Monitor :

- L’agent Azure Monitor ne prend pas en charge les solutions et les insights telles qu’Azure Monitor pour machines virtuelles et Azure Security Center. Le seul scénario actuellement pris en charge consiste à collecter des données à l’aide des règles de collecte de données que vous configurez. 
- Les règles de collecte de données doivent être créées dans la même région que n’importe quel espace de travail Log Analytics utilisé comme destination.
- Les machines virtuelles Azure et les serveurs Azure Arc sont actuellement pris en charge. Les groupes de machines virtuelles identiques, Azure Kubernetes Service et d’autres types de ressources de calcul ne sont pas pris en charge actuellement.
- La machine virtuelle doit avoir accès aux points de terminaison HTTPS suivants :
  - *.ods.opinsights.azure.com
  - *.ingest.monitor.azure.com
  - *.control.monitor.azure.com


## <a name="coexistence-with-other-agents"></a>Coexistence avec d’autres agents
L’agent Azure Monitor peut coexister avec les agents existants afin que vous puissiez continuer à utiliser leurs fonctionnalités existantes lors de l’évaluation ou de la migration. Ceci est particulièrement important en raison des limitations de la préversion publique dans la prise en charge des solutions existantes. Vous devez être prudent lors de la collecte de données en double, car cela peut fausser les résultats de la requête et entraîner des frais supplémentaires pour l’ingestion et la rétention de données.

Par exemple, Azure Monitor pour machines virtuelles utilise l’agent Log Analytics pour envoyer des données de performances à un espace de travail Log Analytics. Vous avez peut-être également configuré l’espace de travail pour collecter les événements Windows et les événements Syslog à partir d’agents. Si vous installez l’agent Azure Monitor et que vous créez une règle de collecte de données pour ces mêmes événements et données de performances, des données dupliquées sont générées.


## <a name="costs"></a>Coûts
Il n’y a aucun coût lié à l’agent Azure Monitor, mais vous pouvez être facturé pour les données ingérées. Pour plus d’informations sur la collecte et la rétention de données Log Analytics et les mesures client, consultez [Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

## <a name="data-sources-and-destinations"></a>Sources et destinations de données
Le tableau suivant répertorie les types de données que vous pouvez actuellement collecter avec l’agent Azure Monitor à l’aide de règles de collecte de données et de l’emplacement où vous pouvez envoyer ces données. Consultez [Quels sont les éléments supervisés par Azure Monitor ?](../monitor-reference.md) pour obtenir la liste des analyses, des solutions et d’autres solutions qui utilisent l’agent Azure Monitor pour collecter d’autres types de données.


L’agent Azure Monitor envoie des données aux métriques Azure Monitor ou à un espace de travail Log Analytics qui prend en charge les journaux d’Azure Monitor.

| source de données | Destinations | Description |
|:---|:---|:---|
| Performances        | Métriques Azure Monitor<br>Espace de travail Log Analytics | Valeurs numériques mesurant les performances de différents aspects du système d’exploitation et des charges de travail. |
| Journaux d'événements Windows | Espace de travail Log Analytics | Informations envoyées au système de journalisation des événements Windows. |
| syslog             | Espace de travail Log Analytics | Informations envoyées au système de journalisation des événements Linux. |


## <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge
Pour obtenir la liste des versions de système d’exploitation Windows et Linux que l’agent Azure Monitor prend actuellement en charge, consultez [Systèmes d’exploitation pris en charge](agents-overview.md#supported-operating-systems).



## <a name="security"></a>Sécurité
L’agent Azure Monitor n’a besoin d’aucune clé, mais nécessite une [identité managée affectée par le système](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#system-assigned-managed-identity). Vous devez disposer d’une identité managée affectée par le système activée sur chaque machine virtuelle avant de déployer l’agent.

## <a name="networking"></a>Mise en réseau
L’agent Azure Monitor prend en charge les étiquettes de service Azure (les étiquettes AzureMonitor et AzureResourceManager sont exigées), mais il ne fonctionne pas encore avec les étendues de liaison privée Azure Monitor ou les proxys directs.


## <a name="next-steps"></a>Étapes suivantes

- [Installez l’agent Azure Monitor](azure-monitor-agent-install.md) sur des machines virtuelles Windows et Linux.
- [Créez une règle de collecte de données](data-collection-rule-azure-monitor-agent.md) pour collecter des données à partir de l’agent et les envoyer à Azure Monitor.
