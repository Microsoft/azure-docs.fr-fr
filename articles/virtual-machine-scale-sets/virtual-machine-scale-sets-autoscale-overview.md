---
title: Vue d’ensemble de la mise à l’échelle automatique avec des groupes de machines virtuelles identiques Azure
description: En savoir plus sur les différentes méthodes de mise à l’échelle automatique d’un groupe de machines virtuelles identiques Azure définies en fonction des performances ou selon une planification fixe
author: avirishuv
ms.author: avverma
ms.topic: overview
ms.service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 06/30/2020
ms.reviewer: jushiman
ms.custom: avverma
ms.openlocfilehash: 092b95845ed033ac0705e325fc6535739088848f
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94888791"
---
# <a name="overview-of-autoscale-with-azure-virtual-machine-scale-sets"></a>Vue d’ensemble de la mise à l’échelle automatique avec des groupes de machines virtuelles identiques Azure
Un groupe de machines virtuelles identiques Azure peut augmenter ou diminuer automatiquement le nombre d’instances de machine virtuelle qui exécutent votre application. Ce comportement élastique et automatisé réduit la charge de gestion pour analyser et optimiser les performances de votre application. Vous créez des règles qui définissent les performances acceptables pour une expérience utilisateur positive. Lorsque les seuils définis sont respectés, les règles de mise à l’échelle automatique prennent des mesures pour ajuster la capacité de votre groupe identique. Vous pouvez également planifier des événements qui augmentent ou diminuent automatiquement la capacité de votre groupe identique à des moments donnés. Cet article fournit une vue d’ensemble des mesures de performance disponibles et des actions réalisables par la mise à l’échelle automatique.


## <a name="benefits-of-autoscale"></a>Avantages de la mise à l’échelle automatique
Si la demande de votre application augmente, la charge sur les instances de machine virtuelle dans votre groupe identique augmente. Si cette augmentation de la charge est cohérente, au lieu d’une brève demande, vous pouvez configurer des règles de mise à l’échelle automatique pour augmenter le nombre d’instances de machine virtuelle dans le groupe identique.

> [!NOTE]
> Lorsque vous utilisez des réparations automatiques d’instances pour votre groupe identique, le nombre maximal d’instances dans le groupe identique est 200. Apprenez-en davantage sur les [réparations automatiques d’instances](./virtual-machine-scale-sets-automatic-instance-repairs.md).

Lorsque ces instances de machine virtuelle sont créées et que vos applications sont déployées, le groupe identique commence à distribuer le trafic vers les instances via l’équilibreur de charge. Vous contrôlez les métriques à surveiller, telles que l’usage du processeur ou de la mémoire, la durée pendant laquelle la charge de l’application doit respecter un seuil donné, et le nombre d’instances de machine virtuelle à ajouter au groupe identique.

Au cours d’une soirée ou d’un week-end, la demande de votre application peut diminuer. Si cette charge réduite est constante pendant un certain temps, vous pouvez configurer des règles de mise à l’échelle automatique pour réduire le nombre d’instances de machine virtuelle dans le groupe identique. Cette action de diminution du nombre d’instances a pour effet de réduire le coût d’exécution de votre groupe identique, car vous seul exécutez le nombre d’instances requis pour répondre à la demande en cours.


## <a name="use-host-based-metrics"></a>Utiliser des mesures basées sur les hôtes
Vous pouvez créer des règles de mise à l’échelle automatique qui intègrent des mesures d’hôtes disponibles à partir de vos instances de machine virtuelle. Les mesures d’hôtes vous offrent une visibilité sur les performances des instances de machine virtuelle dans un groupe identique sans devoir installer ou configurer des agents supplémentaires et des collections de données. Les règles de mise à l’échelle automatique qui utilisent ces mesures peuvent augmenter ou diminuer le nombre d’instances de machine virtuelle en réponse à l’utilisation du processeur, la demande de mémoire ou l’accès au disque.

Les règles de mise à l’échelle qui utilisent des indicateurs basés sur les hôtes peuvent être créées avec l’un des outils suivants :

- [Azure portal](virtual-machine-scale-sets-autoscale-portal.md)
- [Azure PowerShell](tutorial-autoscale-powershell.md)
- [Azure CLI](tutorial-autoscale-cli.md)
- [Modèle Azure](tutorial-autoscale-template.md)

Pour créer des règles de mise à l’échelle automatique qui utilisent des mesures de performance plus détaillées, vous pouvez [installer et configurer l’extension des diagnostics Azure](#in-guest-vm-metrics-with-the-azure-diagnostics-extension) sur les instances de machine virtuelle, ou [configurer votre application à l’aide d’Application Insights](#application-level-metrics-with-app-insights).

Les règles de mise à l’échelle automatique qui utilisent des mesures basées sur les hôtes, des mesures de machine virtuelle intégrées à l’extension des diagnostics Azure et Application Insights peuvent utiliser les paramètres de configuration suivants.

### <a name="metric-sources"></a>Sources des mesures
Les règles de mise à l’échelle automatique peuvent utiliser des mesures provenant d’une des sources suivantes :

| Source de la mesure        | Cas d’utilisation                                                                                                                     |
|----------------------|------------------------------------------------------------------------------------------------------------------------------|
| Groupe identique actuel    | Pour les mesures basées sur les hôtes qui ne nécessitent pas l’installation ou la configuration d’agents supplémentaires.                                  |
| Compte de stockage      | L’extension des diagnostics Azure écrit des mesures de performances dans le stockage Azure, qui est ensuite utilisé pour déclencher des règles de mise à l’échelle automatique. |
| File d’attente Service Bus    | Votre application ou d’autres composants peuvent transmettre des messages dans une file d’attente Service Bus Azure pour déclencher des règles.                   |
| Application Insights | Un package d’instrumentation installé dans votre application qui diffuse en continu les mesures directement à partir de l’application.                         |


### <a name="autoscale-rule-criteria"></a>Critères de règle de mise à l’échelle automatique
Les métriques basées sur les hôtes suivantes sont disponibles lorsque vous créez des règles de mise à l’échelle automatique. Si vous utilisez l’extension des diagnostics Azure ou Application Insights, vous définissez quelles mesures surveiller et utiliser avec les règles de mise à l’échelle automatique.

| Nom de métrique               |
|---------------------------|
| Pourcentage UC            |
| Network In                |
| Network Out               |
| Disk Read Bytes           |
| Disk Write Bytes          |
| Disk Read Operations/Sec  |
| Disk Write Operations/Sec |
| Crédits de processeurs restants     |
| Crédits de processeur consommés      |

Lorsque vous créez des règles de mise à l’échelle automatique pour surveiller une mesure donnée, les règles s’intéressent à une des actions d’agrégation des mesures suivantes :

| Type d’agrégation |
|------------------|
| Moyenne          |
| Minimum          |
| Maximale          |
| Total            |
| Dernier             |
| Count            |

Les règles de mise à l’échelle automatique sont alors déclenchées lorsque les mesures sont comparées à votre seuil défini avec l’un des opérateurs suivants :

| Opérateur                 |
|--------------------------|
| Supérieur à             |
| Supérieur ou égal à |
| Inférieur à                |
| Inférieur ou égal à    |
| Égal à                 |
| Différent de             |


### <a name="actions-when-rules-trigger"></a>Actions lors du déclenchent des règles
Lorsqu’une règle de mise à l’échelle automatique est déclenchée, votre groupe identique peut automatiquement diminuer la taille des instances de l’une des manières suivantes :

| Opération de mise à l'échelle     | Cas d’utilisation                                                                                                                               |
|---------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| Augmenter le nombre de   | Un nombre fixe d’instances de machines virtuelles à créer. Utile dans des groupes identiques avec un petit nombre de machines virtuelles.                                           |
| Augmenter le pourcentage de | Une augmentation basée sur des pourcentages des instances de machine virtuelle. Utile dans des groupes identiques plus grands, lorsqu’une augmentation fixe peut ne pas améliorer considérablement les performances. |
| Augmenter le nombre à   | Créer le nombre d’instances de machines virtuelles requis pour atteindre le nombre maximal souhaité.                                                            |
| Diminuer le nombre de   | Un nombre fixe d’instances de machines virtuelles à supprimer. Utile dans des groupes identiques avec un petit nombre de machines virtuelles.                                           |
| Diminuer le pourcentage de | Une diminution basée sur des pourcentages des instances de machines virtuelles. Utile dans des groupes identiques plus grands, lorsqu’une augmentation fixe peut ne pas réduire considérablement la consommation de ressources et les coûts. |
| Réduire le nombre à   | Supprimer le nombre d’instances de machines virtuelles requis pour atteindre le nombre minimal souhaité.                                                            |


## <a name="in-guest-vm-metrics-with-the-azure-diagnostics-extension"></a>Mesures de machines virtuelles intégrées avec l’extension des diagnostics Azure
L’extension des diagnostics Azure est un agent qui s’exécute à l’intérieur d’une instance de machine virtuelle. L’agent surveille et enregistre les mesures de performances dans le stockage Azure. Ces mesures de performances contiennent des informations plus détaillées sur l’état de la machine virtuelle, tel que *AverageReadTime* pour les disques ou *PercentIdleTime* pour l’UC. Vous pouvez créer des règles de mise à l’échelle automatique basées sur une plus grande précision des performances de la machine virtuelle, pas seulement sur le pourcentage de consommation de mémoire ou d’utilisation du processeur.

Pour utiliser l’extension des diagnostics Azure, vous devez créer des comptes de stockage Azure pour vos instances de machine virtuelle, installer l’agent de diagnostics Azure puis configurer les machines virtuelles pour transmettre des compteurs de performance spécifiques au compte de stockage.

Pour en savoir plus, consultez les articles sur l’activation de l’extension de diagnostic Azure sur une [machine virtuelle Linux](../virtual-machines/extensions/diagnostics-linux.md) ou [Windows](../virtual-machines/extensions/diagnostics-windows.md).


## <a name="application-level-metrics-with-app-insights"></a>Mesures au niveau de l’application avec Application Insights
Pour gagner en visibilité dans les performances de vos applications, vous pouvez utiliser Application Insights. Vous installez un petit package d’instrumentation dans votre application qui surveille l’application et envoie des données de télémétrie à Azure. Vous pouvez surveiller des mesures, telles que les temps de réponse de votre application, les performances de chargement des pages et le nombres de sessions. Ces mesures de l’application peuvent être utilisées pour créer des règles de mise à l’échelle automatique à un niveau granulaire et incorporé lors du déclenchement de règles basées sur des informations exploitables qui peuvent avoir un impact sur l’expérience utilisateur.

Pour plus d’informations sur App Insights, consultez [Présentation d’Application Insights](../azure-monitor/app/app-insights-overview.md).


## <a name="scheduled-autoscale"></a>Mise à l’échelle automatique planifiée
Vous pouvez également créer des règles de mise à l’échelle automatique basées sur des planifications. Ces règles de planification vous permettent de mettre à l'échelle automatiquement le nombre d’instances de machines virtuelles à des moments donnés. Avec des règles basées sur la performance, il peut y avoir un impact sur les performances de l’application avant le déclenchement des règles de mise à l’échelle automatique et l’approvisionnement des nouvelles instances de machines virtuelles. Si vous pouvez anticiper ce type de demande, les instances de machines virtuelles supplémentaires sont approvisionnées et prêtes à répondre aux demandes de l’application et aux utilisations supplémentaires par le client.

Les exemples suivants sont des scénarios qui peuvent bénéficier de l’utilisation de règles de mise à l’échelle automatique basées sur la planification :

- Redimensionner automatiquement le nombre d’instances de machines virtuelles au début de la journée de travail lorsque la demande du client augmente. À la fin de la journée de travail, redimensionner automatiquement le nombre d’instances de machines virtuelles afin de réduire les coûts des ressources pendant la nuit, lorsque l’utilisation de l’application est faible.
- Si un service utilise fortement une application durant certaines parties du mois ou du cycle fiscal, redimensionner automatiquement le nombre d’instances de machines virtuelles pour prendre en compte des exigences supplémentaires.
- Lors d’un événement marketing, d’une saison de promotions ou de fêtes, vous pouvez automatiquement mettre à l’échelle le nombre d’instances de machines virtuelles afin d’anticiper la demande du client. 


## <a name="next-steps"></a>Étapes suivantes
Vous pouvez créer des règles de mise à l’échelle automatique qui utilisent des indicateurs basés sur les hôtes avec l’un des outils suivants :

- [Azure PowerShell](tutorial-autoscale-powershell.md)
- [Azure CLI](tutorial-autoscale-cli.md)
- [Modèle Azure](tutorial-autoscale-template.md)

Cette présentation vous a appris à utiliser des règles de mise à l’échelle automatique pour mettre à l’échelle horizontalement, et augmenter ou diminuer le *nombre* d’instances de machines virtuelles dans votre groupe identique. Vous pouvez également mettre à l’échelle verticalement pour augmenter ou diminuer la *taille* d’instance de machine virtuelle. Pour plus d’informations, voir [Mise à l’échelle verticale avec des groupes de machines virtuelles identiques](virtual-machine-scale-sets-vertical-scale-reprovision.md).

Pour plus d’informations sur la gestion de vos instances de machine virtuelle, voir [Gérer les groupes de machines virtuelles identiques avec Azure PowerShell](./virtual-machine-scale-sets-manage-powershell.md).

Pour découvrir comment générer des alertes lorsque vos règles de mise à l’échelle automatique déclenchent, voir [Utilisation d’actions de mise à l’échelle automatique pour envoyer des notifications d’alerte webhook et par courrier électronique dans Azure Monitor](../azure-monitor/platform/autoscale-webhook-email.md). Vous pouvez également [Utiliser des journaux d’audit pour envoyer des notifications d’alerte webhook et par courrier électronique dans Azure Monitor](../azure-monitor/platform/alerts-log-webhook.md).
