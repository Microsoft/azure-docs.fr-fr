---
title: Vue d’ensemble d’Azure et des clusters Service Fabric autonomes
description: Vous pouvez créer des cluster Service Fabric sur toute machine virtuelle ou tout ordinateur exécutant Windows Server ou Linux. Cela signifie que vous pouvez déployer et exécuter des applications Service Fabric dans tout environnement dans lequel vous avez des ordinateurs Windows Server ou Linux interconnectés, que ce soit en local sur Microsoft Azure ou à l’aide d’un fournisseur de cloud.
ms.topic: conceptual
ms.date: 01/07/2020
ms.openlocfilehash: d83ec139c367d9a539db173a42ae909f57d5d71a
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576126"
---
# <a name="comparing-azure-and-standalone-service-fabric-clusters-on-windows-server-and-linux"></a>Comparaison entre Azure et des clusters Service Fabric autonomes sur Windows Server et Linux

Un cluster Service Fabric est un groupe de machines virtuelles ou physiques connectées au réseau, sur lequel vos microservices sont déployés et gérés. Une machine ou machine virtuelle faisant partie d’un cluster est appelée un nœud de cluster. Les clusters peuvent être mis à l’échelle pour des milliers de nœuds. Si vous ajoutez des nœuds au cluster, Service Fabric rééquilibre les réplicas de partition du service et les instances sur le nombre de nœuds augmenté. Les performances globales de l’application s’améliorent tandis que le conflit d’accès à la mémoire diminue. Si les nœuds du cluster ne sont pas utilisés efficacement, vous pouvez diminuer le nombre de nœuds dans le cluster. Service Fabric rééquilibre à nouveau les réplicas de partition et les instances sur le nombre réduit de nœuds afin de mieux utiliser le matériel sur chaque nœud.

Service Fabric permet la création de clusters Service Fabric sur toute machine virtuelle ou tout ordinateur exécutant Windows Server ou Linux. Cela signifie que vous pouvez déployer et exécuter des applications Service Fabric dans n’importe quel environnement dans lequel des ordinateurs Windows Server ou Linux sont interconnectés, que ce soit en local, sur Microsoft Azure ou avec n’importe quel fournisseur de cloud.

## <a name="benefits-of-clusters-on-azure"></a>Avantages des clusters sur Azure

Dans Azure, nous intégrons d’autres fonctionnalités et services Azure qui rendent l’exploitation et la gestion du cluster plus simple et plus fiable.

* **Portail Azure :** facilite la création et la gestion de clusters.
* **Azure Resource Manager :** l’utilisation d’Azure Resource Manager permet de faciliter la gestion de toutes les ressources utilisées par le cluster en tant qu’unité et de simplifier le suivi des coûts et la facturation.
* **Cluster Service Fabric en tant que ressource Azure :** un cluster Service Fabric est une ressource Azure, que vous pouvez modeler comme d’autres ressources dans Azure.
* **Intégration à l’infrastructure Azure :** Service Fabric se coordonne avec l’infrastructure Azure sous-jacente pour que le système d’exploitation, le réseau et d’autres mises à niveau améliorent la disponibilité et la fiabilité de vos applications.  
* **Diagnostics :** dans Azure, nous proposons l’intégration de Diagnostics Azure et des journaux Azure Monitor.
* **Mise à l’échelle automatique :** pour les clusters sur Azure, nous fournissons une fonctionnalité de mise à l’échelle automatique intégrée par le biais des groupes de machines virtuelles identiques. Dans des environnements locaux ou d’autres environnements cloud, vous devez créer votre propre fonctionnalité de mise à l’échelle automatique ou mettre à l’échelle manuellement à l’aide des API que Service Fabric expose pour la mise à l’échelle des clusters.

## <a name="benefits-of-standalone-clusters"></a>Avantages des clusters autonomes

* Vous pouvez choisir n’importe quel fournisseur de cloud pour héberger votre cluster.
* Une fois écrites, les applications Service Fabric peuvent être exécutées dans plusieurs environnements d’hébergement sans modification ou avec des modifications minimales.
* Les connaissances en matière de génération d’applications Service Fabric s’appliquent d’un environnement d’hébergement à un autre.
* L’expérience opérationnelle d’exécution et de gestion de clusters Service Fabric s’applique d’un environnement à un autre.
* Ciblage étendu en termes de clientèle sans contrainte d’environnement d’hébergement.
* Une couche supplémentaire de fiabilité et de protection contre les pannes répandues existe, car vous pouvez déplacer les services vers un autre environnement de déploiement si un centre de données ou le fournisseur de services cloud est confronté à une panne d’électricité.

## <a name="next-steps"></a>Étapes suivantes

* Lire la présentation des [clusters Service Fabric sur Azure](service-fabric-azure-clusters-overview.md)
* Lire la présentation des [clusters Service Fabric autonomes](service-fabric-standalone-clusters-overview.md)
* En savoir plus sur les [options de prise en charge de Service Fabric](service-fabric-support.md)