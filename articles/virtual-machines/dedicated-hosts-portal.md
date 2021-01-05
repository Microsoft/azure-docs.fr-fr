---
title: Déployer des hôtes dédiés Azure à l’aide du portail Azure
description: Déployez des machines virtuelles et des groupes identiques sur des hôtes dédiés à l'aide du portail Azure.
author: cynthn
ms.service: virtual-machines
ms.topic: how-to
ms.workload: infrastructure
ms.date: 09/04/2020
ms.author: cynthn
ms.openlocfilehash: a6bef4944207e26f2de93daa89fa1418c5c44c4f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91373022"
---
# <a name="deploy-vms-and-scale-sets-to-dedicated-hosts-using-the-portal"></a>Déployer des machines virtuelles et des groupes identiques sur des hôtes dédiés à l'aide du portail 

Cet article vous guide dans la création d’un [hôte dédié](dedicated-hosts.md) Azure pour héberger vos machines virtuelles. 


> [!IMPORTANT]
> Cet article traite également de la sélection élective automatique des machines virtuelles et des groupes identiques. La fonctionnalité Sélection élective automatique est actuellement en préversion publique.
> Pour participer à la préversion, répondez à l'enquête d'intégration à l'adresse suivante : [https://aka.ms/vmss-adh-preview](https://aka.ms/vmss-adh-preview).
> Pour accéder à la fonctionnalité d'évaluation sur le portail Azure, vous devez utiliser l'URL suivante : [https://aka.ms/vmssadh](https://aka.ms/vmssadh).
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="limitations"></a>Limites

- Les tailles et les types de matériel disponibles pour les hôtes dédiés varient selon la région. Pour en savoir plus, consultez la [page de tarification de l’hôte](https://aka.ms/ADHPricing).

## <a name="create-a-host-group"></a>Créer un groupe hôte

Un **groupe hôte** est une ressource qui représente une collection d’hôtes dédiés. Vous créez un groupe hôte dans une région et une zone de disponibilité, et lui ajoutez des hôtes. Lors de la planification de la haute disponibilité, vous pouvez accéder à des options supplémentaires. Vous pouvez utiliser l’une des options suivantes, ou les deux, avec vos hôtes dédiés : 
- Application sur plusieurs zones de disponibilité. Dans ce cas, vous devez disposer d’un groupe hôte dans chacune des zones que vous souhaitez utiliser.
- Application sur plusieurs domaines d’erreur mappés à des racks physiques. 
 
Dans les deux cas, vous devez fournir le nombre de domaines d’erreur pour votre groupe hôte. Si vous ne souhaitez pas appliquer plusieurs domaines d’erreur dans votre groupe, utilisez un seul domaine d’erreur. 

Vous pouvez également choisir d’utiliser des zones de disponibilité et des domaines d’erreur. 

Dans cet exemple, nous allons créer un groupe hôte en utilisant 1 zone de disponibilité et 2 domaines d’erreur. 


1. Ouvrez le [Portail Microsoft Azure](https://portal.azure.com). Si vous souhaitez essayer la préversion de la fonctionnalité **Sélection élective automatique**, utilisez l'URL suivante : [https://aka.ms/vmssadh](https://aka.ms/vmssadh).
1. Sélectionnez l’option **Créer une ressource**, en haut à gauche du portail.
1. Recherchez des **groupes hôtes** et sélectionnez **Host Groups** (Groupes hôtes) dans les résultats.
1. Dans la page **Host Groups** (Groupes hôtes), sélectionnez **Créer**.
1. Sélectionnez l’abonnement que vous souhaitez utiliser, puis sélectionnez **Création** pour créer un groupe de ressources.
1. Indiquez *myDedicatedHostsRG* dans la zone **Nom**, puis sélectionnez **OK**.
1. Dans **Host Group Name** (Nom du groupe hôte), saisissez *myHostGroup*.
1. Pour **Emplacement**, sélectionnez **USA Est**.
1. Pour **Zone de disponibilité**, sélectionnez **1**.
1. Pour **Fault Domain Count** (Nombre de domaines d’erreur), sélectionnez **2**.
1. Si vous avez utilisé l'URL de la fonctionnalité **Sélection élective automatique**, sélectionnez cette option pour attribuer automatiquement des machines virtuelles et mettre à l'échelle des machines virtuelles et des groupes identiques sur un hôte disponible dans ce groupe.
1. Sélectionnez **Vérifier + Créer**, puis attendez la validation.
1. Lorsque le message **Validation passed** (Validation réussie) apparaît, sélectionnez **Créer** pour créer le groupe hôte.

Cette opération ne prend que quelques instants.


## <a name="create-a-dedicated-host"></a>Créer un hôte dédié

Créez maintenant un hôte dédié dans le groupe hôte. En plus d’un nom pour l’hôte, vous devez fournir la référence SKU pour l’hôte. La référence SKU de l’hôte capture la série de machines virtuelles prises en charge, ainsi que la génération du matériel pour l’hôte dédié.

Pour en savoir plus sur les références SKU et la tarification des hôtes, consultez la section relative à la [tarification des hôtes dédiés Azure](https://aka.ms/ADHPricing).

Si vous définissez un nombre de domaines d’erreur pour votre groupe hôte, vous êtes invité à spécifier le domaine d’erreur de votre hôte.  

1. Sélectionnez l’option **Créer une ressource**, en haut à gauche du portail.
1. Recherchez un **hôte dédié** et sélectionnez **Dedicated Hosts** (Hôtes dédiés) dans les résultats.
1. Dans la page **Dedicated Hosts** (Hôtes dédiés), sélectionnez **Créer**.
1. Sélectionnez l’abonnement à utiliser.
1. Sélectionnez *myDedicatedHostsRG* parmi les **groupes de ressources**.
1. Dans **Détails de l’instance**, saisissez *myHost* dans la zone **Nom** et sélectionnez l’emplacement *USA Est*.
1. Dans **Hardware Profile** (Profil matériel), sélectionnez *Standard Es3 family – Type 1* (Famille ES3 standard – Type 1) pour **Size Family** (Famille de tailles), *myHostGroup* pour **Host Group** (Groupe hôte) et *1* pour **Fault Domain** (Domaine d’erreur). Conservez les valeurs par défaut pour les autres champs.
1. Cela fait, sélectionnez **Vérifier + Créer** et attendez la validation.
1. Lorsque le message **Validation passed** (Validation réussie) apparaît, sélectionnez **Créer** pour créer l’hôte.

## <a name="create-a-vm"></a>Créer une machine virtuelle

1. Choisissez **Créer une ressource** en haut à gauche du Portail Azure.
1. Dans la zone de recherche située au-dessus de la liste des ressources de la Place de marché Azure, recherchez et sélectionnez l'image que vous souhaitez utiliser, puis choisissez **Créer**.
1. Sous l’onglet **De base**, sous **Détails du projet**, vérifiez que l’abonnement approprié est sélectionné, puis sélectionnez *myDedicatedHostsRG* comme **groupe de ressources**. 
1. Sous **Détails de l’instance**, tapez *myVM* comme **Nom de la machine virtuelle**, puis choisissez *USA Est* comme **Emplacement**.
1. Dans **Options de disponibilité**, sélectionnez **Zone de disponibilité**, puis sélectionnez *1* dans la liste déroulante.
1. Pour la taille, sélectionnez **Modifier la taille**. Dans la liste des tailles disponibles, choisissez-en une dans la série Esv3, par exemple **Standard E2s v3**. Vous devrez peut-être effacer le filtre afin d’afficher toutes les tailles disponibles.
1. Renseignez les autres champs de l'onglet **De base** selon vos besoins.
1. En haut de la page, sélectionnez l’onglet **Avancé**, puis dans la section **Hôte**, sélectionnez *myHostGroup* pour **Groupe hôte** et *myHost* pour l'**hôte**. 
    ![Sélectionner le groupe hôte et l’hôte](./media/dedicated-hosts-portal/advanced.png)
1. Conservez les valeurs par défaut restantes, puis sélectionnez le bouton **Vérifier + créer** en bas de la page.
1. Lorsque vous voyez le message indiquant que la validation a réussi, sélectionnez **Créer**.

Quelques minutes sont nécessaires pour le déploiement de votre machine virtuelle.

## <a name="create-a-scale-set-preview"></a>Créez un groupe identique (préversion)

> [!IMPORTANT]
> La fonctionnalité Groupes de machines virtuelles identiques sur Dedicated Host est actuellement en préversion publique.
>
> Pour participer à la préversion, répondez à l'enquête d'intégration à l'adresse suivante : [https://aka.ms/vmss-adh-preview](https://aka.ms/vmss-adh-preview).
>
> Pour accéder à la fonctionnalité d'évaluation sur le portail Azure, vous devez utiliser l'URL suivante : [https://aka.ms/vmssadh](https://aka.ms/vmssadh).
>
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. 
>
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Lorsque vous déployez un groupe identique, vous spécifiez le groupe hôte.

1. Recherchez *Groupe identique* et sélectionnez **Groupes de machines virtuelles identiques** dans la liste.
1. Sélectionnez **Ajouter** pour créer un groupe identique.
1. Renseignez les champs de l'onglet **De base** comme vous le feriez habituellement, mais veillez à sélectionner une taille de machine virtuelle appartenant à la série que vous avez choisie pour votre hôte dédié, par exemple **Standard E2s v3**.
1. Sous l'onglet **Avancé**, pour **Algorithme de diffusion** sélectionnez **Diffusion maximale**.
1. Sous **Groupe hôte**, sélectionnez le groupe hôte dans la liste déroulante. Si vous venez de créer le groupe, l'ajout à la liste peut prendre une minute.

## <a name="add-an-existing-vm"></a>Ajouter une machine virtuelle existante 

Vous pouvez ajouter une machine virtuelle existante à un hôte dédié, mais la machine virtuelle doit d’abord être arrêtée\libérée. Avant de déplacer une machine virtuelle vers un hôte dédié, vérifiez que la configuration de la machine virtuelle est prise en charge :

- La taille de la machine virtuelle doit appartenir à la même famille de tailles que l’hôte dédié. Par exemple, si votre hôte dédié est DSv3, la taille de la machine virtuelle peut être Standard_D4s_v3, mais pas Standard_A4_v2. 
- La machine virtuelle doit être située dans la même région que l’hôte dédié.
- La machine virtuelle ne peut pas faire partie d’un groupe de placements de proximité. Supprimez la machine virtuelle du groupe de placements de proximité avant de la déplacer vers un hôte dédié. Pour plus d’informations, consultez [Déplacer une machine virtuelle hors d’un groupe de placements de proximité](./windows/proximity-placement-groups.md#move-an-existing-vm-out-of-a-proximity-placement-group).
- La machine virtuelle ne peut pas se trouver dans un groupe à haute disponibilité.
- Si la machine virtuelle se trouve dans une zone de disponibilité, il doit s’agir de la même zone de disponibilité que celle du groupe hôte. Les paramètres de zone de disponibilité de la machine virtuelle et du groupe hôte doivent correspondre.

Déplacez la machine virtuelle vers un hôte dédié à l’aide du [portail](https://portal.azure.com).

1. Ouvrez la page de la machine virtuelle.
1. Sélectionnez **Arrêter** pour arrêter\libérer la machine virtuelle.
1. Sélectionnez **Configuration** dans le menu de gauche.
1. Sélectionnez un groupe hôte et un hôte dans les menus déroulants.
1. Lorsque vous avez terminé, sélectionnez **Enregistrer** en haut de la page.
1. Une fois la machine virtuelle ajoutée à l’hôte, sélectionnez **Vue d’ensemble** dans le menu de gauche.
1. En haut de la page, sélectionnez **Démarrer** pour redémarrer la machine virtuelle.

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus, voir la page de présentation [Hôtes dédiés](dedicated-hosts.md).

- Un exemple de modèle, disponible [ici](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-dedicated-hosts/README.md), utilise les zones et les domaines d’erreur pour offrir une résilience maximale dans une région.

- Vous pouvez également déployer un hôte dédié à l'aide d'[Azure CLI](./linux/dedicated-hosts-cli.md) ou de [PowerShell](./windows/dedicated-hosts-powershell.md).
