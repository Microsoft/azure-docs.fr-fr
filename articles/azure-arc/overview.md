---
title: Vue d’ensemble d’Azure Arc
description: Découvrez ce qu’est le service Azure Arc et comment il permet aux clients d’activer la gestion et la gouvernance de leurs ressources hybrides avec d’autres services et fonctionnalités Azure.
ms.date: 09/23/2020
ms.topic: overview
ms.openlocfilehash: 8e48378e5032768b3f56a5d99c1189e282ff37f8
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91714143"
---
# <a name="azure-arc-overview"></a>Vue d’ensemble d’Azure Arc

Aujourd’hui, les entreprises éprouvent des difficultés à contrôler et à gérer un environnement qui devient de plus en plus complexe. Ces environnements s’étendent aux centres de données, à plusieurs clouds et à la périphérie. Chaque environnement et chaque cloud a son propre ensemble d’outils de gestion différents que vous devez découvrir et utiliser.

En parallèle, il est difficile d’implémenter de nouveaux modèles opérationnels DevOps et ITOps, car les outils existants ne prennent pas en charge les nouveaux modèles cloud natifs.

Azure Arc simplifie la gouvernance et la gestion en proposant une plateforme de gestion multicloud et locale cohérente. Azure Arc vous permet de gérer l’ensemble de votre environnement, avec un volet unique, en projetant vos ressources existantes dans Azure Resource Manager. Vous pouvez maintenant gérer les machines virtuelles, les clusters Kubernetes et les bases de données comme s’ils s’exécutaient dans Azure. Quel que soit l’endroit où ils résident, vous pouvez utiliser des services et des fonctionnalités de gestion connus d’Azure. Azure Arc vous permet de continuer à utiliser des opérateurs informatiques (ITOps) traditionnels, tout en introduisant des pratiques DevOps pour prendre en charge de nouveaux modèles cloud natifs dans votre environnement.

:::image type="content" source="./media/overview/azure-arc-control-plane.png" alt-text="Diagramme du plan de contrôle de gestion d’Azure Arc" border="false":::

Aujourd’hui, Azure Arc vous permet de gérer les types de ressources suivants hébergés en dehors d’Azure :

* Serveurs : machines virtuelles et machines physiques exécutant Windows ou Linux.
* Clusters Kubernetes : prise en charge de plusieurs distributions Kubernetes.
* Services de données Azure : services Azure SQL Database et PostgreSQL Hyperscale.

## <a name="what-does-azure-arc-deliver"></a>Que propose Azure Arc ?

Les fonctionnalités clés d’Azure Arc sont les suivantes :

* Implémenter une cohérence au niveau inventaire, gestion, gouvernance et sécurité pour les serveurs dans votre environnement.

* Configurer des [extensions de machine virtuelle Azure](./servers/manage-vm-extensions.md) pour utiliser les services de gestion Azure afin de superviser, de sécuriser et de mettre à jour vos serveurs.

* Gérer et régir les clusters Kubernetes à grande échelle.

* Utiliser la configuration basée sur GitOps comme gestion du code pour déployer les applications et la configuration sur un ou plusieurs clusters directement à partir du contrôle de code source, comme GitHub.

* Conformité et configuration sans intervention pour vos clusters Kubernetes à l’aide d’Azure Policy.

* Exécuter les services de données Azure sur n’importe quel environnement Kubernetes, en particulier Azure SQL Managed Instance et Azure Database pour PostgreSQL Hyperscale, avec des avantages tels que les mises à niveau/mises à jour, la sécurité et la supervision, comme s’ils s’exécutaient dans Azure. Tirer parti de la mise à l’échelle élastique, appliquer des mises à jour sans temps d’arrêt minimal de l’application, même si elle ne dispose pas d’une connexion continue à Azure.

* Une expérience unifiée de l’affichage de vos ressources Azure Arc activées, que vous utilisiez le portail Azure, l’interface Azure CLI, Azure PowerShell ou l’API REST Azure.

## <a name="how-much-does-azure-arc-cost"></a>Combien coûte Azure Arc ?

Voici les détails du tarif des fonctionnalités disponibles aujourd’hui avec Azure Arc.

### <a name="arc-enabled-servers"></a>Serveurs avec Arc

La fonctionnalité de plan de contrôle Azure Arc est proposée sans coût supplémentaire.notamment :

* Organisation des ressources avec les groupes d'administration et les étiquettes Azure

* Recherche et indexation avec Azure Resource Graph

* Accès et sécurité avec Azure RBAC et des abonnements

* Environnements et automatisation avec des modèles et des extensions

* Gestion des mises à jour

Tout service Azure utilisé sur des serveurs avec Arc, par exemple Azure Security Center ou Azure Monitor, sera facturé sur la base des tarifs de ce service. Pour plus d’informations, consultez la page [Tarification Azure](https://azure.microsoft.com/pricing/).

### <a name="azure-arc-enabled-kubernetes"></a>Kubernetes avec Azure Arc

Dans la phase de préversion actuelle, Kubernetes avec Azure Arc est proposé sans frais supplémentaires.

### <a name="azure-arc-enabled-data-services"></a>Services de données avec Azure Arc

Dans la phase de préversion actuelle, les services de données avec Azure Arc sont proposés sans frais supplémentaires.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les serveurs avec Arc, consultez cette [vue d’ensemble](./servers/overview.md)

* Pour plus d’informations sur Kubernetes avec Arc, consultez cette [vue d’ensemble](./kubernetes/overview.md)

* Pour plus d’informations sur les services de données avec Arc, consultez cette [vue d’ensemble](https://azure.microsoft.com/services/azure-arc/hybrid-data-services/)
