---
title: Vue d’ensemble des groupes de machines virtuelles identiques Azure
description: En savoir plus sur les groupes identiques de machines virtuelles Azure et comment mettre automatiquement à l’échelle vos applications
author: mimckitt
ms.author: mimckitt
ms.topic: overview
ms.service: virtual-machine-scale-sets
ms.subservice: ''
ms.date: 06/30/2020
ms.reviewer: jushiman
ms.custom: mimckitt
ms.openlocfilehash: dff83159511c6e8a08e8f212c01244e923ad1f5b
ms.sourcegitcommit: 051908e18ce42b3b5d09822f8cfcac094e1f93c2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2020
ms.locfileid: "94376942"
---
# <a name="what-are-virtual-machine-scale-sets"></a>Que sont les groupes de machines virtuelles identiques ?
Les groupes identiques de machines virtuelles Azure vous permettent de créer et de gérer un groupe de machines virtuelles et disposant d’une charge équilibrée. Le nombre d’instances de machine virtuelle peut augmenter ou diminuer automatiquement en fonction d’une demande ou d’un calendrier défini. Les groupes identiques offrent une haute disponibilité à vos applications, et vous permettent de gérer, configurer et mettre à jour de manière centralisée un grand nombre de machines virtuelles. Avec les groupes identiques de machines virtuelles, vous pouvez créer des services à grande échelle pour des zones telles que le calcul, Big Data et des charges de travail de conteneur.


## <a name="why-use-virtual-machine-scale-sets"></a>Pourquoi utiliser les groupes identiques de machines virtuelles ?
Pour offrir une redondance et de meilleures performances, les applications sont généralement réparties sur plusieurs instances. Les clients peuvent accéder à votre application via un équilibreur de charge qui distribue les requêtes à l’une des instances de l’application. Si vous avez besoin effectuer une maintenance ou de mettre à jour une instance d’application, vos clients doivent être répartis sur une autre instance disponible de l’application. Pour suivre la demande de clients supplémentaires, vous devrez peut-être augmenter le nombre d’instances d’application qui exécutent votre application.

Les groupes identiques de machines virtuelles Azure fournissent les fonctionnalités de gestion pour des applications exécutées sur nombreuses machines virtuelles, la [mise à l’échelle automatique des ressources](virtual-machine-scale-sets-autoscale-overview.md)et un équilibrage de charge du trafic. Les groupes identiques vous offrent les avantages suivants :

- **Création et gestion faciles de plusieurs machines virtuelles**
    - Lorsque vous disposez de nombreuses machines virtuelles qui exécutent votre application, il est important de conserver une configuration cohérente dans votre environnement. Pour des performances fiables de votre application, la taille de la machine virtuelle, la configuration du disque et les installations de l’application doivent correspondre entre toutes les machines virtuelles.
    - Avec les groupes identiques, toutes les instances de machine virtuelle sont créées à partir de la même image de système d’exploitation de base et de la même configuration. Cette approche vous permet de gérer facilement des centaines de machines virtuelles sans tâches de configuration supplémentaires ou gestion de réseau.
    - Les groupes identiques prennent en charge l’utilisation de l’[équilibreur de charge Azure](../load-balancer/load-balancer-overview.md) pour la distribution du trafic de type Couche 4 de base, et [Azure Application Gateway](../application-gateway/overview.md) pour une distribution du trafic de type Couche 7 plus avancée et un arrêt TLS.

- **Offre une haute disponibilité et une résilience des applications**
    - Les groupes identiques sont utilisés pour exécuter plusieurs instances de votre application. Si l’une de ces instances de machine virtuelle a un problème, les clients continuent d’accéder à votre application via l’une des autres instances de machine virtuelle avec une interruption minimale.
    - Pour une disponibilité supplémentaire, vous pouvez utiliser des [zones de disponibilité](../availability-zones/az-overview.md) pour distribuer automatiquement des instances de machine virtuelle dans un groupe identique au sein d’un centre de données unique ou de plusieurs centres de données.

- **Permet une mise à l’échelle automatique de votre application en fonction des variations du besoin en ressources**
    - La demande du client pour votre application peut changer pendant la journée ou la semaine. Pour suivre la demande du client, les groupes identiques peuvent augmenter automatiquement le nombre d’instances de machine virtuelle lorsque la demande de l’application augmente, et le réduire lorsque la demande diminue.
    - La mise à l’échelle automatique réduit également le nombre d’instances de machine virtuelle inutiles exécutant votre application lorsque la demande est faible, tandis que les clients continuent de recevoir un niveau de performance acceptable lorsque la demande se développe et des instances de machines virtuelles supplémentaires sont ajoutées automatiquement. Cette capacité permet de réduire les coûts et de créer efficacement des ressources Azure en fonction des besoins.

- **Fonctionne à grande échelle**
    - Les groupes identiques peuvent prendre en charge jusqu’à 1 000 instances de machines virtuelles. Si vous créez et chargez vos propres images de machine virtuelle personnalisées, la limite est de 600 instances de machine virtuelle.
    - Pour des performances optimales avec des charges de travail de production, utilisez [Azure Disques managés](../virtual-machines/managed-disks-overview.md).


## <a name="differences-between-virtual-machines-and-scale-sets"></a>Différences entre les machines virtuelles et les groupes identiques
Les groupes identiques sont conçus à partir de machines virtuelles. Avec les groupes identiques, les couches d’automatisation et de gestion sont fournies pour exécuter et faire évoluer vos applications. À la place, vous pourriez créer et gérer manuellement des machines virtuelles individuelles, ou intégrer des outils existants pour créer un niveau similaire d’automatisation. Le tableau suivant présente les avantages des groupes identiques comparés à la gestion manuelle de plusieurs instances de machine virtuelle.

| Scénario                           | Groupe manuelle de machines virtuelles                                                                    | Jeu de mise à l’échelle de machine virtuelle |
|------------------------------------|----------------------------------------------------------------------------------------|---------------------------|
| Ajouter des instances de machine virtuelle supplémentaires        | Processus manuel permettant de créer, configurer et d’assurer la conformité                             | Créer automatiquement à partir d’une configuration centrale |
| Équilibrage et distribution du trafic | Processus manuel pour créer et configurer un équilibreur de charge Azure ou Application Gateway      | Peut créer et s’intégrer automatiquement à l’équilibreur de charge Azure ou Application Gateway |
| Haute disponibilité et redondance   | Créer manuellement un groupe à haute disponibilité ou distribuer et suivre des machines virtuelles entre des zones de disponibilité | Distribution automatique des instances de machine virtuelle entre des zones de disponibilité ou des groupes à haute disponibilité |
| Mise à l’échelle de machines virtuelles                     | Surveillance manuelle et Azure Automation                                                 | Mise à l’échelle basée sur des mesures d’hôte, des mesures d’invité, Application Insights ou une planification |

Les groupes identiques n’entraînent aucun coût supplémentaire. Vous payez uniquement les ressources de calcul sous-jacentes telles que les instances de machine virtuelle, un équilibreur de charge ou un stockage par disque managé. Les fonctionnalités de gestion et d’automatisation, telles que la mise à l’échelle automatique et la redondance, n’entraînent aucun frais supplémentaires pour l’utilisation de machines virtuelles.

## <a name="how-to-monitor-your-scale-sets"></a>Comment superviser vos groupes identiques

Utilisez [Azure Monitor pour machines virtuelles](../azure-monitor/insights/vminsights-overview.md), dont le processus d’intégration est simple et automatise la collecte des données à partir des compteurs importants du processeur, de la mémoire, des disques et des performances réseau sur les machines virtuelles de votre groupe identique. Il comprend également des fonctionnalités de supervision supplémentaires et des visualisations prédéfinies qui vous permettent de vous concentrer sur la disponibilité et les performances de vos groupes identiques.

Activez la supervision de votre [application de groupe de machines virtuelles identiques](../azure-monitor/app/azure-vm-vmss-apps.md) avec Application Insights afin de collecter des informations détaillées sur votre application, notamment les vues de pages, les requêtes d’application et les exceptions. Vérifiez la disponibilité de votre application en configurant un [test de disponibilité](../azure-monitor/app/monitor-web-app-availability.md) pour simuler le trafic utilisateur.

## <a name="data-residency"></a>Résidence des données

Dans Azure, la fonctionnalité permettant le stockage de données client dans une seule région n’est actuellement disponible que dans la région Asie Sud-Est (Singapour) de la zone géographique Asie-Pacifique et la région Brésil Sud (État de Sao Paulo) de la zone géographique Brésil. Pour toutes les autres régions, les données client sont stockées dans Zone géographique. Pour plus d’informations, consultez le [Centre de gestion de la confidentialité](https://azuredatacentermap.azurewebsites.net/).

## <a name="next-steps"></a>Étapes suivantes
Pour commencer, créez votre premier groupe identique de machines virtuelles dans le portail Azure.

> [!div class="nextstepaction"]
> [Créer un groupe identique depuis le portail Azure](quick-create-portal.md)
