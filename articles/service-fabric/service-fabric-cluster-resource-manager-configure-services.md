---
title: Spécifier des métriques et des paramètres de positionnement
description: Découvrez comment décrire un service Service Fabric en spécifiant des métriques, des contraintes de positionnement et d’autres stratégies de positionnement.
author: masnider
ms.topic: conceptual
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: d4dcd319000edb204ba188ed14b4c797dba5cd38
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75610095"
---
# <a name="configuring-cluster-resource-manager-settings-for-service-fabric-services"></a>Configuration des paramètres Cluster Resource Manager pour les services Service Fabric
Service Fabric Cluster Resource Manager assure un contrôle précis des règles qui régissent chaque service nommé individuel. Chaque service nommé peut spécifier des règles pour son allocation dans le cluster. Chaque service nommé peut également définir l’ensemble de mesures à inclure dans ses rapports, notamment leur importance pour ce service. La configuration des services se décompose en trois tâches :

1. Configuration des contraintes de positionnement
2. Configuration des mesures
3. Configuration des stratégies et des autres règles de positionnement avancées (moins fréquent)

## <a name="placement-constraints"></a>Contraintes de placement
Les contraintes de placement sont utilisées pour contrôler sur quels nœuds du cluster un service peut s’exécuter. En général, l’exécution d’une instance de service nommé spécifique ou de tous les services d’un type donné est limitée à un type de nœud particulier. Les contraintes de placement sont extensibles. Vous pouvez définir n’importe quel jeu de propriétés par type de nœud, puis les sélectionner avec des contraintes lors de la création de services. Vous pouvez également modifier les contraintes de placement d’un service en cours d’exécution. Cela vous permet de répondre aux modifications apportées dans le cluster ou aux exigences du service. Les propriétés d’un nœud donné peuvent également être mises à jour dynamiquement dans le cluster. Vous trouverez plus d’informations sur les contraintes de placement et leur configuration dans [cet article](service-fabric-cluster-resource-manager-cluster-description.md#node-properties-and-placement-constraints).

## <a name="metrics"></a>Mesures
Les mesures correspondent à l’ensemble de ressources dont un de service nommé a besoin. La configuration de mesures d’un service détermine notamment la quantité d’une ressource que chaque réplica avec état ou instance sans état de ce service consomme par défaut. Les mesures comprennent également un poids qui indique l’importance de l’équilibrage de la mesure pour le service, dans le cas où un compromis est nécessaire.

## <a name="advanced-placement-rules"></a>Règles de placement avancées
Il existe d’autres types de règles de placement qui sont utiles dans des scénarios moins courants. Quelques exemples :
- Contraintes utiles avec des clusters dispersés géographiquement
- Certaines architectures d’application

La configuration des autres règles de positionnement s’effectue via Corrélations ou Stratégies.

## <a name="next-steps"></a>Étapes suivantes
- Les métriques représentent la façon dont Service Fabric Cluster Resource Manager gère la consommation et la capacité du cluster. Pour en savoir plus sur ces mesures et la façon de les configurer, consultez [cet article](service-fabric-cluster-resource-manager-metrics.md)
- L’affinité est un mode que vous pouvez configurer pour vos services. Ce n’est pas courant, mais si nécessaire, vous trouverez plus d’informations [ici](service-fabric-cluster-resource-manager-advanced-placement-rules-affinity.md)
- Il existe de nombreuses règles de positionnement différentes qui peuvent être configurées sur votre service pour gérer des scénarios supplémentaires. Vous trouverez plus d’informations sur ces différentes stratégies de positionnement [ici](service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md)
- Commencez au début pour [obtenir une présentation de Service Fabric Cluster Resource Manager](service-fabric-cluster-resource-manager-introduction.md)
- Pour en savoir plus sur la façon dont Cluster Resource Manager gère et équilibre la charge du cluster, consultez l’article sur [l’équilibrage de la charge](service-fabric-cluster-resource-manager-balancing.md)
- Cluster Resource Manager comporte de nombreuses options permettant de décrire le cluster. Pour en savoir plus sur celles-ci, consultez cet article sur la [description d’un cluster Service Fabric](service-fabric-cluster-resource-manager-cluster-description.md).
