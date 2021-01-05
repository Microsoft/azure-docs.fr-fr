---
title: Concepts – Stockage
description: Découvrez les principales capacités de stockage dans les clouds privés d’Azure VMware Solution.
ms.topic: conceptual
ms.date: 11/03/2020
ms.openlocfilehash: 394b755f5dc8fdbcaa6092419e9451ae1691aa67
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93289256"
---
#  <a name="azure-vmware-solution-storage-concepts"></a>Concepts de stockage pour Azure VMware Solution

Les clouds privés Azure VMware Solution fournissent un stockage natif à l’échelle du cluster avec VMware vSAN. Tout le stockage local de chaque hôte dans un cluster est utilisé dans un magasin de données vSAN, et un chiffrement des données au repos est disponible et activé par défaut. Vous pouvez utiliser les ressources de Stockage Azure pour étendre les capacités de stockage de vos clouds privés.

## <a name="vsan-clusters"></a>Clusters vSAN

Un stockage local dans chaque hôte de cluster est utilisé en lien avec un magasin de données vSAN. Tous les groupes de disques utilisent une couche cache NVMe de 1,6 To avec la capacité brute sur SDD par ordinateur hôte de 15,4 To. La taille de la couche de capacité brute d’un cluster est égale à la capacité par hôte multipliée par le nombre d’hôtes. Par exemple, un cluster de quatre hôtes fournira une capacité brute de 61,6 To dans la couche de capacité vSAN.

Le stockage local dans les hôtes du cluster est utilisé dans un magasin de stockage vSAN à l’échelle du cluster. Tous les magasins de données sont créés dans le cadre d’un déploiement de cloud privé et immédiatement utilisables. L’utilisateur cloudadmin et Tous les utilisateurs membres du groupe CloudAdmin peuvent gérer les magasins de données avec les privilèges vSAN suivants :
- Datastore.AllocateSpace
- Datastore.Browse
- Datastore.Config
- Datastore.DeleteFile
- Datastore.FileManagement
- Datastore.UpdateVirtualMachineMetadata

## <a name="data-at-rest-encryption"></a>Chiffrement des données au repos

Par défaut, les magasins de données vSAN utilisent le chiffrement des données au repos. La solution de chiffrement est basée sur un service de gestion de clés et prend en charge les opérations de vCenter pour la gestion des clés. Les clés sont stockées chiffrées, encapsulées par une clé principale Azure Key Vault. Lors de la suppression d’un hôte d’un cluster pour une raison quelconque, les données sur disques SSD sont immédiatement invalidées.

## <a name="scaling"></a>Mise à l'échelle

La capacité de stockage du cluster natif est mise à l’échelle en ajoutant des hôtes à un cluster. Pour les clusters qui utilisent des hôtes haut de gamme, la capacité brute à l’échelle du cluster est augmentée de 15,4 To par hôte supplémentaire. La capacité brute des clusters créés avec des hôtes à usage général est augmentée de 7,7 To par hôte supplémentaire. Dans les deux types de clusters, l’ajout d’hôtes à un cluster prend environ 10 minutes. Pour obtenir des instructions sur la mise à l’échelle des clusters, consultez le [didacticiel sur la mise à l’échelle de cloud privé][tutorial-scale-private-cloud].

## <a name="azure-storage-integration"></a>Intégration du Stockage Azure

Vous pouvez utiliser les services de Stockage Azure sur des charges de travail qui s’exécutent dans votre cloud privé. Les services de Stockage Azure incluent les Comptes de stockage, le Stockage Table et le Stockage Blob. La connexion des charges de travail aux services de Stockage Azure ne traverse pas Internet. Cette connectivité offre une sécurité supplémentaire et vous permet d’utiliser les services de Stockage Azure basés sur un Contrat de niveau de service (SLA) dans vos charges de travail de cloud privé.

## <a name="next-steps"></a>Étapes suivantes

L’étape suivante consiste à découvrir les [concepts de l’identité de cloud privé][concepts-identity].

<!-- LINKS - external-->

<!-- LINKS - internal -->
[tutorial-scale-private-cloud]: ./tutorial-scale-private-cloud.md
[concepts-identity]: ./concepts-identity.md
