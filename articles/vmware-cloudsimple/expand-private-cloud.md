---
title: Développer Azure VMware Solution by CloudSimple – Composants VMware de cloud privé
description: Décrit comment étendre un cloud privé CloudSimple existant pour accroître la capacité d’un cluster existant ou en ajouter un nouveau
author: sharaths-cs
ms.author: b-shsury
ms.date: 06/06/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: a82ba1b433e62ed1c4b72b8e942d4ade29f26c4a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77025297"
---
# <a name="expand-a-cloudsimple-private-cloud"></a>Développer un cloud privé CloudSimple

CloudSimple fournit la flexibilité nécessaire pour étendre de manière dynamique un cloud privé. Vous pouvez commencer avec une configuration plus petite, puis la faire grandir quand vous avez besoin d’une capacité supérieure. Ou vous pouvez créer un cloud privé en fonction de vos besoins actuels et le développer ensuite lorsque la consommation augmente.

Un cloud privé se compose d’un ou de plusieurs clusters vSphere. Chaque cluster peut contenir entre 3 et 16 nœuds.  Lors du développement d’un cloud privé, vous ajoutez des nœuds au cluster existant ou créez un nouveau cluster. Pour développer un cluster existant, les nœuds supplémentaires doivent être du même type (référence SKU) que les nœuds existants. Pour la création d’un nouveau cluster, les nœuds peuvent être d’un type différent. Pour plus d’informations sur les limites du cloud privé, consultez la section Limites de l’article [Vue d’ensemble des clouds privés CloudSimple](cloudsimple-private-cloud.md).

Un cloud privé est créé avec un **centre de données** par défaut sur vCenter.  Chaque centre de données constitue une entité de gestion de niveau supérieur.  Pour un nouveau cluster, CloudSimple permet de choisir parmi l’ajout au centre de données existant ou la création d’un nouveau centre de données.

Dans le cadre de la nouvelle configuration de cluster, CloudSimple configure l’infrastructure VMware.  Les paramètres incluent les paramètres de stockage pour les groupes de disques vSAN, la haute disponibilité VMware et Distributed Resource Scheduler (DRS).

Un cloud privé peut être développé plusieurs fois. L’expansion peut être effectuée uniquement dans les limites de nœud globales. Chaque fois que vous développez un cloud privé, vous ajoutez au cluster existant ou en créez un nouveau.

## <a name="before-you-begin"></a>Avant de commencer

Les nœuds doivent être configurés avant de pouvoir développer votre cloud privé.  Pour plus d’informations sur le provisionnement des nœuds, consultez l’article [Provisionner des nœuds pour VMware Solution by CloudSimple - Azure](create-nodes.md).  Pour créer un nouveau cluster, vous devez disposer d’au moins trois nœuds disponibles avec la même référence SKU.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="expand-a-private-cloud"></a>Développer un cloud privé

1. [Accédez au portail CloudSimple](access-cloudsimple-portal.md).

2. Ouvrez la page **Ressources** et sélectionnez le cloud privé que vous souhaitez développer.

3. Dans la section Résumé , cliquez sur **Développer**.

    ![Développer un cloud privé](media/resources-expand-private-cloud.png)

4. Choisissez si vous souhaitez développer votre cluster existant ou créer un nouveau cluster vSphere. Lorsque vous apportez des modifications, les informations de résumé sur la page sont mises à jour.

    * Pour développer votre cluster existant, cliquez sur **Développez le cluster existant**. Sélectionnez le cluster que vous souhaitez développer et entrez le nombre de nœuds à ajouter. Chaque cluster peut avoir un maximum de 16 nœuds.
    * Pour ajouter un nouveau cluster, cliquez sur **Créer un cluster**. Entrez un nom pour le cluster. Sélectionnez un centre de données existant, ou entrez un nom pour créer un nouveau centre de données. Choisissez le type de nœud. Vous pouvez choisir un type de nœud différent lors de la création d’un nouveau cluster vSphere, mais pas lors du développement d’un cluster vSphere existant. Sélectionnez le nombre de nœuds. Chaque nouveau cluster doit avoir au moins trois nœuds.

    ![Développer le cloud privé - Ajouter des nœuds](media/resources-expand-private-cloud-add-nodes.png)

5. Cliquez sur **Envoyer** pour développer le cloud privé.

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser des machines virtuelles VMware sur Azure](quickstart-create-vmware-virtual-machine.md)
* Découvrir plus en détail les [clouds privés](cloudsimple-private-cloud.md)