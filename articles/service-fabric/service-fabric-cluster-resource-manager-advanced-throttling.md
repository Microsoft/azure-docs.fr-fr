---
title: Limitations de Service Fabric Cluster Resource Manager
description: Apprenez à configurer les limiteurs fournis par Service Fabric Cluster Resource Manager.
author: masnider
ms.topic: conceptual
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: b4d78b339bab02b5c44a31939e0da769dc21c3ec
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75452173"
---
# <a name="throttling-the-service-fabric-cluster-resource-manager"></a>Limitations de Service Fabric Cluster Resource Manager
Même si vous avez configuré le Cluster Resource Manager , le cluster peut être interrompu. Par exemple, il peut y avoir des défaillances de nœud et de domaine d’erreur simultanées. Que se passe-t-il alors si cela se produit pendant une mise à niveau ? Le Gestionnaire de ressources de cluster essaie de résoudre tous les problèmes en utilisant des ressources du cluster pour tenter de réparer et réorganiser le cluster. Les limitations servent de filet de sécurité afin que le cluster puisse utiliser des ressources pour se stabiliser. Les nœuds reviennent, les partitions réseau sont réparées et les bits corrigés sont déployés.

Pour résoudre ce genre de problème, le Gestionnaire de ressources de cluster Service Fabric comprend plusieurs limitations. Ces limitations sont toutes relativement importantes. En règle générale, elles ne doivent pas être modifiées sans une planification et des tests minutieux.

Si vous modifiez les limitations du Gestionnaire de ressources de cluster, vous devez les adapter à votre charge réelle prévue. Vous pouvez décider que vous avez besoin de certaines limitations, même si cela signifie que la stabilisation du cluster prend parfois plus de temps. Des tests sont nécessaires pour déterminer les valeurs correctes des limitations. Les limitations doivent être suffisamment élevées pour permettre au cluster de répondre aux modifications dans un délai raisonnable, et suffisamment faibles pour réellement empêcher une consommation trop importante des ressources. 

La plupart du temps, les clients utilisent des limitations, car ils travaillent déjà dans un environnement aux ressources restreintes. Par exemple, une bande passante réseau limitée pour des nœuds individuels ou des disques qui ne sont pas en mesure de générer de nombreux réplicas avec état en parallèle en raison des limites de débit. Sans limitations, les opérations risqueraient de saturer ces ressources, ce qui provoquerait l’échec ou le ralentissement de ces opérations. Dans ces cas de figure, les clients utilisaient des limitations et savaient qu’ils devaient prolonger la durée nécessaire au cluster pour avoir un état stable. Les clients comprenaient également qu’ils pourraient fonctionner avec une fiabilité globale inférieure pendant l’application des limitations.


## <a name="configuring-the-throttles"></a>Configuration des limitations

Service Fabric dispose de deux mécanismes pour limiter le nombre de mouvements de réplicas. Le mécanisme par défaut qui existait avant Service Fabric 5.7 correspond à une limitation sous forme de nombre absolu de déplacements autorisés. Il ne fonctionne pas avec des clusters de toutes tailles. En particulier, pour les grands clusters, la valeur par défaut peut être trop petite, ce qui ralentirait considérablement l’équilibrage même lorsqu’il est nécessaire, tout en n’ayant aucun effet sur les clusters plus petits. Le mécanisme antérieur a été remplacé par une limitation basée sur des pourcentages, qui s’adapte mieux aux clusters dynamiques dans lesquels le nombre de services et de nœuds change régulièrement.

Les limitations sont basées sur des pourcentages du nombre de réplicas dans les clusters. Ces limitations permettent, par exemple, de définir la règle : « Ne pas déplacer plus de 10 % des réplicas dans un intervalle de 10 minutes ».

Les paramètres de configuration de la limitation basée sur des pourcentages sont :

  - GlobalMovementThrottleThresholdPercentage : nombre maximal de mouvements autorisés dans le cluster à tout moment, exprimé en pourcentage du nombre total de réplicas dans le cluster. 0 indique aucune limite. La valeur par défaut est 0. Si ce paramètre et le paramètre GlobalMovementThrottleThreshold sont tous deux spécifiés, la limite la plus conservatrice est utilisée.
  - GlobalMovementThrottleThresholdPercentageForPlacement : nombre maximal de mouvements autorisés durant la phase de placement, exprimé en pourcentage du nombre total de réplicas dans le cluster. 0 indique aucune limite. La valeur par défaut est 0. Si ce paramètre et le paramètre GlobalMovementThrottleThresholdForPlacement sont tous deux spécifiés, la limite la plus conservatrice est utilisée.
  - GlobalMovementThrottleThresholdPercentageForBalancing : nombre maximal de mouvements autorisés durant la phase d’équilibrage, exprimé en pourcentage du nombre total de réplicas dans le cluster. 0 indique aucune limite. La valeur par défaut est 0. Si ce paramètre et le paramètre GlobalMovementThrottleThresholdForBalancing sont tous deux spécifiés, la limite la plus conservatrice est utilisée.

Lorsque vous spécifiez le pourcentage de la limitation, vous devez par exemple indiquer 5 % sous la forme 0.05. L’intervalle régissant ces limitations correspond au paramètre GlobalMovementThrottleCountingInterval, qui est exprimé en secondes.


``` xml
<Section Name="PlacementAndLoadBalancing">
     <Parameter Name="GlobalMovementThrottleThresholdPercentage" Value="0" />
     <Parameter Name="GlobalMovementThrottleThresholdPercentageForPlacement" Value="0" />
     <Parameter Name="GlobalMovementThrottleThresholdPercentageForBalancing" Value="0" />
     <Parameter Name="GlobalMovementThrottleCountingInterval" Value="600" />
</Section>
```

via ClusterConfig.json pour les déploiements autonomes ou Template.json pour les clusters hébergés sur Azure :

```json
"fabricSettings": [
  {
    "name": "PlacementAndLoadBalancing",
    "parameters": [
      {
          "name": "GlobalMovementThrottleThresholdPercentage",
          "value": "0.0"
      },
      {
          "name": "GlobalMovementThrottleThresholdPercentageForPlacement",
          "value": "0.0"
      },
      {
          "name": "GlobalMovementThrottleThresholdPercentageForBalancing",
          "value": "0.0"
      },
      {
          "name": "GlobalMovementThrottleCountingInterval",
          "value": "600"
      }
    ]
  }
]
```

### <a name="default-count-based-throttles"></a>Limitations basées sur le nombre par défaut
Ces informations sont fournies dans le cas où vous avez d’anciens clusters ou conservez ces configurations en clusters qui ont par la suite été mis à niveau. En règle générale, il est recommandé de remplacer ces dernières par les limitations basées sur des pourcentages mentionnées précédemment. Étant donné que la limitation basée sur des pourcentages est désactivée par défaut, ces limitations demeurent les limitations par défaut d’un cluster jusqu'à ce qu’elles soient désactivées et remplacées par les limitations basées sur des pourcentages. 

  - GlobalMovementThrottleThreshold : ce paramètre contrôle le nombre total de mouvements dans le cluster au cours d’une certaine période. Cette dernière est exprimée en secondes à l’aide du paramètre GlobalMovementThrottleCountingInterval. La valeur par défaut du paramètre GlobalMovementThrottleThreshold est 1000 et la valeur par défaut du paramètre GlobalMovementThrottleCountingInterval est 600.
  - MovementPerPartitionThrottleThreshold : ce paramètre contrôle le nombre total de mouvements pour n’importe quelle partition de service au cours d’une certaine période. Cette dernière est exprimée en secondes à l’aide du paramètre MovementPerPartitionThrottleCountingInterval. La valeur par défaut du paramètre MovementPerPartitionThrottleThreshold est 50 et la valeur par défaut du paramètre MovementPerPartitionThrottleCountingInterval est 600.

La configuration de ces limitations est similaire à celle de la limitation basée sur des pourcentages.

## <a name="next-steps"></a>Étapes suivantes
- Pour en savoir plus sur la façon dont Cluster Resource Manager gère et équilibre la charge du cluster, consultez l’article sur [l’équilibrage de la charge](service-fabric-cluster-resource-manager-balancing.md)
- Cluster Resource Manager comporte de nombreuses options permettant de décrire le cluster. Pour en savoir plus sur celles-ci, consultez cet article sur la [description d’un cluster Service Fabric](service-fabric-cluster-resource-manager-cluster-description.md).
