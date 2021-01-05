---
title: Gestionnaire des ressources clusters Service Fabric - Affinité
description: Vue d’ensemble de l’affinité de service pour les services Azure Service Fabric et conseils sur la configuration de l’affinité de service.
services: service-fabric
documentationcenter: .net
author: masnider
ms.topic: conceptual
ms.date: 08/18/2017
ms.author: masnider
ms.custom: devx-track-csharp
ms.openlocfilehash: 9c141cd96877fd140b858d0aaed9197f2de80eca
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89005732"
---
# <a name="configuring-and-using-service-affinity-in-service-fabric"></a>Configuration et utilisation de l’affinité de service dans Service Fabric
L’affinité est un contrôle qui permet principalement de faciliter la transition d’applications volumineuses monolithiques vers le cloud et les microservices. Elle peut également servir pour améliorer les performances des services, même si cela peut avoir des conséquences.

Prenons un exemple : vous importez une application volumineuse ou qui n’a pas été conçue pour les microservices dans Service Fabric (ou tout environnement distribué). Ce type de transition est courant. Vous commencez par intégrer l’application complète dans l’environnement, vous la déployez et vous vous assurez qu’elle fonctionne sans problème Puis vous la décomposez en différents services plus petits qui communiquent entre eux.

Par la suite, vous pouvez constater que l’application rencontre des problèmes. Ces problèmes se situent généralement dans l’une des catégories suivantes :

1. Certains composants X de l’application monolithique avaient une dépendance non documentée sur le composant Y et vous les avez transformés en services séparés. Dans la mesure où ces services sont maintenant exécutés sur différents nœuds du cluster, ils sont interrompus.
2. Ces composants communiquent par vecteur (canaux locaux nommés | mémoire partagée | fichiers sur disque), et ils doivent maintenant pouvoir écrire sur une ressource locale partagée pour des raisons de performances. Cette dépendance ferme est éventuellement supprimée ultérieurement.
3. Tout fonctionne correctement, sauf que ces deux composants sont en fait très occupés et sensibles aux performances. Lorsqu’ils sont transférés dans des services distincts, la performance globale des applications diminue ou la latence augmente. Par conséquent, l’application globale ne répond pas aux attentes.

Dans ce cas, nous ne voulons pas perdre notre travail de refactorisation et nous ne souhaitons pas revenir au modèle monolithique. La dernière condition peut même être souhaitable comme optimisation brute. Cependant, jusqu’à ce que nous puissions modifier les composants de manière à ce qu’ils fonctionnent naturellement en tant que services (ou jusqu’à ce que nous puissions répondre autrement aux attentes en matière de performances), nous devrons avoir une certaine notion de la localisation.

Que faire, alors ? Il est possible d’activer l’affinité.

## <a name="how-to-configure-affinity"></a>Configuration de l’affinité
Pour configurer l’affinité, vous définissez une relation d’affinité entre deux services distincts. Cela revient à faire pointer un service sur un autre, ce qui fait qu’un service peut uniquement fonctionner quand cet autre service est en cours d’exécution. Il s’agit parfois de relations parent-enfant (où l’enfant pointe sur le parent). L’affinité permet de s’assurer que les instances ou les réplicas d’un service sont placés sur les mêmes nœuds que les réplicas ou instances d’un autre service.

```csharp
ServiceCorrelationDescription affinityDescription = new ServiceCorrelationDescription();
affinityDescription.Scheme = ServiceCorrelationScheme.Affinity;
affinityDescription.ServiceName = new Uri("fabric:/otherApplication/parentService");
serviceDescription.Correlations.Add(affinityDescription);
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

> [!NOTE]
> Un service enfant ne peut participer qu’à une seule relation d’affinité. Si vous souhaitez créer une affinité entre un enfant et deux services parent à la fois, vous avez deux options :
> - Inverser les relations (faire pointer parentService1 et parentService2 vers le service enfant actuel), ou
> - Désigner l’un des parents comme concentrateur par convention et faire pointer tous les services vers ce service. 
>
> Le comportement de placement qui en résulte dans le cluster doit être le même.
>

## <a name="different-affinity-options"></a>Différentes options d’affinité
L’affinité est représentée par un des jeux de corrélations et comporte deux modes différents. Le mode d’affinité courant est ce que nous appelons « affinité non alignée ». Dans ce mode, les réplicas ou instances des différents services sont placés sur les mêmes nœuds. L’autre mode est appelé « affinité alignée ». L’affinité alignée est utilisée uniquement avec les services avec état. La configuration de deux services avec état pour obtenir une affinité alignée garantit que les serveurs primaires de ces services sont placés sur les mêmes nœuds que d’autres. Par conséquent, chaque paire de serveurs secondaires est placée sur les mêmes nœuds. Il est également possible (bien que moins fréquent) de configurer l’affinité non alignée pour les services avec état. Pour NonAlignedAffinity les différents réplicas des deux services avec état s’exécuteraient sur les mêmes nœuds, mais leurs serveurs primaires seraient installés sur des nœuds différents.

<center>

![Modes d’affinité et leurs effets][Image1]
</center>

### <a name="best-effort-desired-state"></a>Meilleur état souhaité
Une relation d’affinité est conseillée. Elle n’offre pas les mêmes garanties de colocation ou de fiabilité que l’exécution dans le même processus exécutable. Les services dans une relation d’affinité sont fondamentalement différentes entités qui peuvent échouer et être déplacées séparément. Une relation d’affinité peut également être interrompue, bien que ces interruptions soient temporaires. Par exemple, des limites de capacité peuvent indiquer que seuls certains objets de services peuvent être inclus sur un nœud donné. Dans ces cas, même s’il existe une relation d’affinité, elle n’est pas appliquée en raison des autres contraintes. Si la situation le permet, la violation sera corrigée automatiquement plus tard.

### <a name="chains-vs-stars"></a>Chaînes et étoiles
Actuellement, Cluster Resource Manager ne peut pas modéliser les chaînes de relations d’affinité. Cela signifie qu’un service qui est un service enfant dans une relation d’affinité ne peut pas être un parent dans une autre relation d’affinité. Si vous souhaitez modéliser ce type de relation, vous devez le modéliser en étoile plutôt qu’en chaîne. Pour passer d’une chaîne à une étoile, l’enfant le plus bas est apparenté au parent du premier enfant. En fonction de la disposition de vos services, il est possible que vous deviez effectuer cette opération plusieurs fois. S’il n’existe aucun service parent naturel, il est possible que vous deviez en créer un qui sert d’espace réservé. Selon vos besoins, vous pouvez également opter pour les [Groupes d’applications](service-fabric-cluster-resource-manager-application-groups.md).

<center>

![Chaînes et étoiles dans le contexte des relations d’affinité][Image2]
</center>

Notez également que les relations d’affinité sont aujourd’hui directionnelles par défaut. Cela signifie que la règle d’affinité implique que l’enfant se trouve au même endroit que le parent. Elle ne garantit pas que le parent se trouve au même endroit que l’enfant. Par conséquent, s’il existe une violation d’affinité et que, pour corriger la violation, il n’est pas possible pour une raison quelconque de déplacer l’enfant vers le nœud du parent, le parent n’est pas déplacé vers le nœud de l’enfant, même si le déplacement du parent vers le nœud de l’enfant aurait corrigé la violation. La définition du paramètre de configuration [MoveParentToFixAffinityViolation](service-fabric-cluster-fabric-settings.md) sur true supprime la directionalité. Il est également important de noter que la relation d’affinité ne peut pas être idéale ou instantanément appliquée car différents services ont différents cycles de vie et peuvent échouer et se déplacer indépendamment. Par exemple, supposons que le parent bascule soudainement vers un autre nœud suite à un plantage. Cluster Resource Manager et Failover Manager gèrent d’abord le basculement puisque le maintien, la cohérence et la disponible des services restent la priorité. Une fois le basculement terminé, la relation d’affinité est brisée, mais Cluster Resource Manager pense que tout fonctionne correctement, jusqu’à ce qu’il remarque que l’enfant ne se trouve pas avec le parent. Ces types de vérifications sont effectuées régulièrement. Vous trouverez plus d’informations sur la façon dont Cluster Resource Manager évalue les contraintes dans [cet article](service-fabric-cluster-resource-manager-management-integration.md#constraint-types), et [celui-ci](service-fabric-cluster-resource-manager-balancing.md) explique comment configurer la cadence à laquelle ces contraintes sont évaluées.   


### <a name="partitioning-support"></a>Prise en charge du partitionnement
Voici une dernière remarque concernant l’affinité : les relations d’affinité ne sont pas prises en charge lorsque le parent est partitionné. Des services parent partitionnés peuvent être pris en charge par la suite, mais cela n’est pas autorisé à l’heure actuelle.

## <a name="next-steps"></a>Étapes suivantes
- Pour plus d’informations sur la configuration des services, consultez la rubrique [En savoir plus sur la configuration des services](service-fabric-cluster-resource-manager-configure-services.md)
- Pour limiter les services à un petit nombre d’ordinateurs ou agréger la charge des services, utilisez [Groupes d’applications](service-fabric-cluster-resource-manager-application-groups.md)

[Image1]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-affinity/cluster-resrouce-manager-affinity-modes.png
[Image2]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-affinity/cluster-resource-manager-chains-vs-stars.png