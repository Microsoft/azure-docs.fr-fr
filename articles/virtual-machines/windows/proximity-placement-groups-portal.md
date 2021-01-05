---
title: Créer un groupe de placements de proximité à l’aide du portail
description: Découvrez comment créer un groupe de placements de proximité à l’aide du Portail Azure.
author: cynthn
ms.service: virtual-machines
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 04/24/2020
ms.author: cynthn
ms.openlocfilehash: 6a14e2bd7385430c3d0fbec06259a876af556e38
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010722"
---
# <a name="create-a-proximity-placement-group-using-the-portal"></a>Créer un groupe de placements de proximité à l’aide du portail

Pour que les machines virtuelles soient aussi proches que possible, avec la latence la plus faible possible, déployez-les dans un [groupe de placements de proximité](co-location.md#proximity-placement-groups).

Le groupe de placements de proximité est un regroupement logique utilisé pour s’assurer que les ressources de calcul Azure se trouvent proches les unes des autres. Les groupes de placements de proximité sont utiles pour les charges de travail où une latence faible est requise.

> [!NOTE]
> Les groupes de placement de proximité ne peuvent pas être utilisés avec des hôtes dédiés.
>
> Si vous souhaitez utiliser des zones de disponibilité avec des groupes de placement, vous devez vous assurer que les machines virtuelles du groupe de placement se trouvent également dans la même zone de disponibilité.
>

## <a name="create-the-proximity-placement-group"></a>Créer le groupe de placements de proximité

1. Tapez **groupe de placements de proximité** dans la recherche.
1. Sous **Services** dans les résultats de la recherche, sélectionnez **Groupes de placements de proximité**.
1. Dans la page **Groupes de placements de proximité**, sélectionnez **Ajouter**.
1. Sous l’onglet **De base**, sous **Détails du projet**, vérifiez que l’abonnement approprié est sélectionné.
1. Dans **Groupe de ressources**, sélectionnez **Créer** pour créer un groupe ou sélectionnez un groupe de ressources vide qui existe déjà dans la liste déroulante. 
1. Dans **Région**, sélectionnez l’emplacement où vous souhaitez créer le groupe de placements de proximité.
1. Dans **Groupe de placements de proximité**, tapez un nom, puis sélectionnez **Vérifier + créer**.
1. Après la validation, sélectionnez **Créer** pour créer le groupe de placements de proximité.

    ![Capture d’écran de la création d’un groupe de placements de proximité](./media/ppg/ppg.png)


## <a name="create-a-vm"></a>Créer une machine virtuelle

1. Lors de la création d’une machine virtuelle dans le portail, accédez à l’onglet **Avancé**. 
1. Dans la sélection **Groupe de placements de proximité**, choisissez le groupe de placements approprié. 

    ![Capture d’écran de la section de groupe de placements de proximité lors de la création d’une machine virtuelle dans le portail](./media/ppg/vm-ppg.png)

1. Lorsque vous avez terminé d’effectuer toutes les autres sélections requises, sélectionnez **Vérifier + créer**.
1. Après la validation, sélectionnez **Créer** pour déployer la machine virtuelle dans le groupe de placements.


## <a name="add-vms-in-an-availability-set-to-a-proximity-placement-group"></a>Ajouter des machines virtuelles dans un groupe à haute disponibilité à un groupe de placement de proximité

Si la machine virtuelle fait partie du groupe à haute disponibilité, vous devez ajouter ce groupe au groupe de placement, avant d’ajouter les machines virtuelles.

1. Dans le [portail](https://portal.azure.com), recherchez *Groupes à haute disponibilité* et sélectionnez votre groupe à haute disponibilité dans les résultats.
1. Arrêtez\libérez chaque machine virtuelle dans le groupe à haute disponibilité en sélectionnant la machine virtuelle, puis **Arrêter** dans la page de la machine virtuelle, puis **OK** pour arrêter la machine virtuelle.
1. Dans la page de votre groupe à haute disponibilité, vérifiez que toutes les machines virtuelles affichent la valeur **Arrêté (libéré)** pour **État**.
1. Dans le menu de gauche, sélectionnez **Configuration**.
1. Sous **Groupe de placement de proximité**, sélectionnez un groupe de placement dans la liste déroulante, puis **Enregistrer**.
1. Sélectionnez **Vue d’ensemble** dans le menu de gauche pour afficher à nouveau la liste des machines virtuelles. 
1. Sélectionnez chaque machine virtuelle dans le groupe à haute disponibilité, puis **Démarrer** dans la page de chaque machine virtuelle. 


## <a name="add-existing-vm-to-placement-group"></a>Ajouter une machine virtuelle existante au groupe de placement 


1. Sur la page de la machine virtuelle, sélectionnez **Arrêter**.
1. Une fois que l’état de la machine virtuelle est listé comme **Arrêté (libéré)** , sélectionnez **Configuration** dans le menu de gauche.
1. Sous **Groupe de placement de proximité**, sélectionnez un groupe de placement dans la liste déroulante, puis **Enregistrer**.
1. Sélectionnez **Vue d’ensemble** dans le menu de gauche, puis **Démarrer** pour redémarrer la machine virtuelle.

 

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez également utiliser [Azure PowerShell](proximity-placement-groups.md) pour créer des groupes de placements de proximité.

