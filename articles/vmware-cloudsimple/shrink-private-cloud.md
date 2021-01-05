---
title: Réduire le cloud privé Azure VMware Solution by CloudSimple
description: Découvrez comment réduire dynamiquement un Cloud privé dans CloudSimple en supprimant un nœud d’un cluster vSphere existant ou en supprimant un cluster entier.
author: sharaths-cs
ms.author: b-shsury
ms.date: 07/01/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 2759b35c9ba521a63eaf54c4c696598d545120b4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88141735"
---
# <a name="shrink-a-cloudsimple-private-cloud"></a>Réduire un cloud privé CloudSimple

CloudSimple fournit la flexibilité nécessaire pour réduire de manière dynamique un cloud privé.  Un cloud privé se compose d’un ou de plusieurs clusters vSphere. Chaque cluster peut contenir entre 3 et 16 nœuds. Lors de la réduction d’un cloud privé, vous supprimez un nœud du cluster existant ou supprimez un cluster entier. 

## <a name="before-you-begin"></a>Avant de commencer

Les conditions suivantes doivent être remplies pour réduire un cloud privé.  Le cluster de gestion (premier cluster) créé lors de la création d’un cloud privé ne peut pas être supprimé.

* Un cluster vSphere doit contenir trois nœuds.  Un cluster ne contenant que trois nœuds ne peut pas être réduit.
* Le stockage total consommé ne doit pas dépasser la capacité totale après réduction du cluster.
* Vérifiez si des règles DRS (Distributed Resource Scheduler) empêchent vMotion d’une machine virtuelle.  Si des règles sont présentes, désactivez ou supprimez-les.  Les règles DRS incluent la machine virtuelle pour héberger des règles d’affinité.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="shrink-a-private-cloud"></a>Réduire un cloud privé

1. [Accédez au portail CloudSimple](access-cloudsimple-portal.md).

2. Ouvrez la page **Ressources**.

3. Cliquez sur le cloud privé que vous souhaitez réduire.

4. Sur la page de résumé, cliquez sur **Réduire**.

    ![Réduire un cloud privé](media/shrink-private-cloud.png)

5. Sélectionnez le cluster que vous souhaitez réduire ou supprimer. 

    ![Réduire un cloud privé - sélectionner un cluster](media/shrink-private-cloud-select-cluster.png)

6. Sélectionnez **Remove one node** (Supprimer un nœud) ou **Delete the whole cluster** (Supprimer tout le cluster). 

7. Vérifier la capacité du cluster

8. Cliquez sur **Soumettre** pour réduire le cloud privé.

La réduction du cloud privé démarre.  Vous pouvez suivre la progression dans des tâches.  Le processus de réduction peut prendre quelques heures en fonction des données, qui doivent être resynchronisées sur le vSAN.

> [!NOTE]
> 1. Si vous réduisez un cloud privé en supprimant le dernier ou le seul cluster dans le centre de données, le centre de données n’est pas supprimé.
> 2. Si une violation de règle DRS se produit, le nœud n’est pas supprimé du cluster et la description de la tâche indique que la suppression d’un nœud viole les règles DRS sur le cluster.    


## <a name="next-steps"></a>Étapes suivantes

* [Utiliser des machines virtuelles VMware sur Azure](quickstart-create-vmware-virtual-machine.md)
* Découvrir plus en détail les [clouds privés](cloudsimple-private-cloud.md)
