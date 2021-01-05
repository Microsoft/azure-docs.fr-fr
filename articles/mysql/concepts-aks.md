---
title: Se connecter à Azure Kubernetes Service - Azure Database pour MySQL
description: Découvrez comment connecter Azure Kubernetes Service à Azure Database pour MySQL
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 07/14/2020
ms.openlocfilehash: 5c4a5f5d792a60ed3fef07797fdbdfa0c9cfb8fe
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94534328"
---
# <a name="connecting-azure-kubernetes-service-and-azure-database-for-mysql"></a>Connexion entre Azure Kubernetes Service et Azure Database pour MySQL

Azure Kubernetes Service (AKS) fournit un cluster Kubernetes managé que vous pouvez utiliser dans Azure. Voici quelques options à prendre en compte lors de l’utilisation conjointe d’AKS et d’Azure Database pour MySQL dans la création d’une application.


## <a name="accelerated-networking"></a>Mise en réseau accélérée
Utilisez des machines virtuelles sous-jacentes dotées de la mise en réseau accélérée dans votre cluster AKS. Lorsque la mise en réseau accélérée est activée sur une machine virtuelle, celle-ci bénéficie d’une latence plus faible, d’une diminution de l’instabilité et d’une utilisation moins importante du processeur. Apprenez-en davantage sur le fonctionnement de la mise en réseau accélérée, les versions de système d’exploitation prises en charge et les instances de machine virtuelle compatibles pour [Linux](../virtual-network/create-vm-accelerated-networking-cli.md).

À partir de novembre 2018, AKS prend en charge la mise en réseau accélérée sur ces instances de machine virtuelle prises en charge. La mise en réseau accélérée est activée par défaut sur les nouveaux clusters AKS qui utilisent ces machines virtuelles.

Vous pouvez vérifier si votre cluster AKS dispose de la mise en réseau accélérée de la façon suivante :
1. Accédez au portail Azure et sélectionnez votre cluster AKS.
2. Sélectionnez l’onglet Propriétés.
3. Copiez le nom du **Groupe de ressources d’infrastructure**.
4. Utilisez la barre de recherche du portail pour localiser et ouvrir le groupe de ressources d’infrastructure.
5. Sélectionnez une machine virtuelle dans ce groupe de ressources.
6. Accédez à l’onglet **Mise en réseau** de la machine virtuelle.
7. Vérifiez que la **Mise en réseau accélérée** est activée.

Ou via Azure CLI, en utilisant les deux commandes suivantes :
```azurecli
az aks show --resource-group myResourceGroup --name myAKSCluster --query "nodeResourceGroup"
```
La sortie correspond au groupe de ressources que crée AKS et contenant l'interface réseau. Prenez le nom « nodeResourceGroup » et utilisez-le dans la commande suivante. **EnableAcceleratedNetworking** sera true ou false :
```azurecli
az network nic list --resource-group nodeResourceGroup -o table
```


## <a name="next-steps"></a>Étapes suivantes
- [Créer un cluster Azure Kubernetes Service](../aks/kubernetes-walkthrough.md)
- Découvrir comment [Installer WordPress à partir du graphique Helm à l’aide de OSBA et d’Azure Database pour MySQL](../aks/index.yml)