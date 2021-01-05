---
title: Mise à l’échelle d’un cluster autonome Azure Service Fabric
description: En savoir plus sur la mise à l’échelle de clusters autonomes Service Fabric, qu’il s’agisse d’une diminution ou d’une réduction de taille des instances, ou d’une montée ou descente en puissance.
ms.topic: conceptual
ms.date: 11/13/2018
ms.openlocfilehash: 8184b2455e938fa0500308b462176e78b8dadcab
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91843029"
---
# <a name="scaling-service-fabric-standalone-clusters"></a>Mise à l’échelle de clusters autonomes Service Fabric
Un cluster Service Fabric est un groupe de machines virtuelles ou physiques connectées au réseau, sur lequel vos microservices sont déployés et gérés. Une machine ou une machine virtuelle faisant partie d’un cluster est appelée un nœud. Les clusters peuvent potentiellement comporter des milliers de nœuds. Une fois que vous avez créé un cluster Service Fabric, vous pouvez le mettre à l’échelle horizontalement (modifier le nombre de nœuds) ou verticalement (modifier les ressources des nœuds).  Une mise à l’échelle peut s’effectuer à tout moment, même lorsque des charges de travail sont en cours d’exécution sur le cluster.  Lorsque vous mettez vos nœuds à l’échelle, vos applications sont automatiquement mises à l’échelle.

Pourquoi procéder à la mise à l’échelle du cluster ? La demande des applications change au fil du temps.  Il vous faut éventuellement augmenter les ressources de cluster pour prendre en charge l’augmentation de la charge applicative ou du trafic réseau ou diminuer les ressources du cluster en cas de baisse de la demande.

## <a name="scaling-in-and-out-or-horizontal-scaling"></a>Augmentation et diminution de la taille des instances, ou mise à l’échelle horizontale
Modifie le nombre de nœuds dans le cluster.  Une fois que les nouveaux nœuds ont rejoint le cluster, [Cluster Resource Manager](service-fabric-cluster-resource-manager-introduction.md) déplace les services vers ces instances afin de réduire la charge pesant sur les nœuds existants.  Si les ressources du cluster ne sont pas utilisées efficacement, vous pouvez également réduire le nombre de nœuds.  À mesure que les nœuds quittent le cluster, les services se retirent également et la charge augmentent sur les nœuds restants.  La réduction du nombre de nœuds dans un cluster exécuté dans Azure peut vous permettre de réaliser des économies, dans la mesure où vous payez le nombre de machines virtuelles utilisées, non la charge de travail à laquelle elles sont soumises.  

- Avantages : une mise à l’échelle infinie, en théorie.  Si votre application est conçue pour l’évolutivité, vous pouvez prendre en charge une croissance illimitée en ajoutant davantage de nœuds.  Les outils de vos environnements cloud facilitant l’ajout et le retrait des nœuds, il est simple d’ajuster la capacité selon les besoins, en payant uniquement les ressources utilisées.  
- Inconvénients : les applications doivent être [conçues pour la scalabilité](service-fabric-concepts-scalability.md).  Les bases de données et la persistance de l’application peuvent nécessiter des activités architecturales supplémentaires dédiées là aussi à la mise à l’échelle.  [Les collections fiables](service-fabric-reliable-services-reliable-collections.md) dans les services avec état Service Fabric, cependant, simplifient grandement la mise à l’échelle de vos données applicatives.

Les clusters autonomes vous permettent de déployer un cluster Service Fabric en local ou auprès du fournisseur de cloud de votre choix.  Les types de nœuds sont constitués de machines physiques ou de machines virtuelles, en fonction de votre déploiement. Par rapport aux clusters en cours d’exécution dans Azure, le processus de mise à l’échelle d’un cluster autonome est un petit peu plus complexe.  Vous devez modifier manuellement le nombre de nœuds dans le cluster, puis exécuter une mise à niveau de la configuration du cluster.

La suppression de nœuds peut entraîner plusieurs mises à niveau. Certains nœuds, identifiés avec la balise `IsSeedNode=”true”`, peuvent être identifiés en interrogeant le manifeste du cluster à l’aide de [Get-ServiceFabricClusterManifest](/powershell/module/servicefabric/get-servicefabricclustermanifest). La suppression de ces nœuds peut prendre plus de temps car, dans ce cas, les nœuds initiaux devront être déplacés. Le cluster doit conserver au moins 3 nœuds de type nœud principal.

> [!WARNING]
> Nous vous recommandons de ne pas réduire le nombre de nœuds sous la [taille de niveau de fiabilité](service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster) pour le cluster. Cela interférera avec la possibilité de réplication des services système Service Fabric sur le cluster, et déstabilisera, voire détruira, le cluster.
>

Lors de la mise à l’échelle d’un cluster autonome, gardez les instructions suivantes à l’esprit :
- Le remplacement des nœuds principaux doit être effectué un nœud à la fois, au lieu de supprimer, puis d’ajouter des nœuds par lots.
- Avant de supprimer un type de nœud, vérifiez s’il existe des nœuds qui référencent le type de nœud concerné. Supprimez ces nœuds avant de supprimer le type de nœud correspondant. Une fois que tous les nœuds correspondants sont supprimés, vous pouvez supprimer le NodeType de la configuration du cluster et commencer une mise à niveau de la configuration à l’aide de [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade).

Pour plus d’informations, consultez la section consacrée à la [mise à l’échelle d’un cluster autonome](service-fabric-cluster-windows-server-add-remove-nodes.md).

## <a name="scaling-up-and-down-or-vertical-scaling"></a>Montée et descente en puissance, ou mise à l’échelle verticale 
Modifie les ressources (processeur, mémoire ou stockage) des nœuds du cluster.
- Avantages : l’architecture de l’application et du logiciel demeure identique.
- Inconvénients : mise à l’échelle limitée, dans la mesure où l’augmentation des ressources sur les nœuds individuels est limitée. Des temps d’arrêt, car il vous faudra mettre hors ligne les machines physiques ou virtuelles afin d’ajouter ou de supprimer des ressources.

## <a name="next-steps"></a>Étapes suivantes
* Découvrez-en plus sur l’[extensibilité des applications](service-fabric-concepts-scalability.md).
* [Procédez à une montée et une descente en puissance d’un cluster Azure](service-fabric-tutorial-scale-cluster.md).
* [Mettez à l’échelle un cluster Azure par programmation](service-fabric-cluster-programmatic-scaling.md), à l’aide du kit de développement logiciel de calcul Azure.
* [Augmentez ou diminuez la taille des instances d’un cluster autonome](service-fabric-cluster-windows-server-add-remove-nodes.md).

