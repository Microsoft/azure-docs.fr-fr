---
title: Activer Automanage pour machines virtuelles via Azure Policy
description: Découvrez comment activer Azure Automanage pour machines virtuelles via une stratégie Azure Policy intégrée dans le portail Azure.
author: ju-shim
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: how-to
ms.date: 09/04/2020
ms.author: jushiman
ms.openlocfilehash: 8f679626b69bd855e86b94cdde51955edd068e8f
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91714894"
---
# <a name="enable-automanage-for-virtual-machines-through-azure-policy"></a>Activer Automanage pour machines virtuelles via Azure Policy

Si vous souhaitez activer Automanage pour un grand nombre de machines virtuelles, vous pouvez le faire à l’aide d’une stratégie [Azure Policy](..\governance\azure-management.md) intégrée. Cet article vous guide tout au long de la recherche de la stratégie appropriée et de l’attribution de celle-ci afin d’activer Automanage dans le portail Azure.


## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/) avant de commencer.

> [!NOTE]
> Les comptes associés à un essai gratuit n’ont pas accès aux machines virtuelles utilisées dans ce tutoriel. Veuillez passer à un abonnement avec paiement à l’utilisation.

> [!IMPORTANT]
> L’autorisation Azure RBAC suivante est nécessaire pour activer Automanage : Rôles **Propriétaire** ou **Contributeur**, ainsi que **Administrateur de l’accès utilisateur**.


## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com/).


## <a name="locate-and-assign-the-policy"></a>Localiser et attribuer la stratégie

1. Accédez à **Stratégie** dans le portail Azure.
1. Accédez au volet **Définitions**.
1. Cliquez sur la liste déroulante **Catégories** pour afficher les options disponibles.
1. Sélectionnez l’option **Activer Automanage – Meilleures pratiques pour les machines virtuelles Azure**.
1. La liste est mise à jour pour afficher une stratégie intégrée avec un nom commençant par *Activer Automanage...* .
1. Cliquez sur le nom de stratégie intégrée *Activer Automanage – Meilleures pratiques pour les machines virtuelles Azure*.
1. Après avoir cliqué sur la stratégie, vous pouvez voir l’onglet **Définition**.

    > [!NOTE]
    > La définition d’Azure Policy est utilisée pour définir des paramètres d’Automanage, tels que le profil de configuration ou le compte. Elle définit également des filtres qui veillent à ce que la stratégie s’applique uniquement aux machines virtuelles appropriées.

1. Cliquez sur le bouton **Attribuer** pour créer une affectation.
1. Sous l’onglet **De base**, renseignez l’**Étendue** en définissant l’*Abonnement* et le *Groupe de ressources*.

    > [!NOTE]
    > L’Étendue vous permet de définir les machines virtuelles auxquelles cette stratégie s’applique. Vous pouvez définir l’application au niveau d’un abonnement ou d’un groupe de ressources. Si vous définissez un groupe de ressources, Automanage est automatiquement activé pour toutes les machines virtuelles qui se trouvent dans celui-ci ou que nous y ajouterons à l’avenir. 

1. Cliquez sur l’onglet **Paramètres**, puis définissez le **Compte Automanage** et le **Profil de configuration** souhaité. 
1. Sous l’onglet **Vérifier + créer**, examinez les paramètres.
1. Appliquez l’affectation en cliquant sur **Créer**.
1. Consultez vos affectations sous l’onglet **Affectations** en regard de **Définition**.

> [!NOTE]
> Un certain temps s’écoule avant que cette stratégie prenne effet sur les machines virtuelles qui se trouvent actuellement dans le groupe de ressources ou l’abonnement.


## <a name="next-steps"></a>Étapes suivantes 

Découvrez une autre façon d’activer Azure Automanage pour les machines virtuelles via le portail Azure. 

> [!div class="nextstepaction"]
> [Activer le service Automanage pour machines virtuelles dans le portail Azure](quick-create-virtual-machines-portal.md)