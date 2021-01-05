---
title: Quota de machines virtuelles Spot
description: Augmentez les limites de quota pour les machines virtuelles Spot, qui fournissent un modèle d’utilisation d’Azure vous permettant de réduire les coûts dans Exchange afin qu’Azure puisse supprimer des machines virtuelles si nécessaire.
author: sowmyavenkat86
ms.author: svenkat
ms.date: 01/27/2020
ms.topic: how-to
ms.assetid: ce37c848-ddd9-46ab-978e-6a1445728a3b
ms.openlocfilehash: c8b9a2251d7923fe1919b7b934f6c97877cd5b37
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745483"
---
# <a name="spot-quota-increase-limits-for-all-vm-series"></a>Quota Spot : augmenter les limites de toutes les séries de machines virtuelles

Les machines virtuelles Spot fournissent un modèle d’utilisation Azure différent. Elles vous permettent de réduire les coûts si vous autorisez Azure à supprimer, si nécessaire, des machines virtuelles dans le cadre d’un paiement à l’utilisation ou du déploiement d’une instance de machine virtuelle réservée. Pour plus d’informations sur les machines virtuelles Spot, consultez [Machines virtuelles Spot Azure pour les groupes de machines virtuelles identiques](../../virtual-machine-scale-sets/use-spot.md).

Azure Resource Manager prend en charge deux types de quotas de processeurs virtuels pour les machines virtuelles :

* Les *machines virtuelles avec paiement à l’utilisation* et les *instances de machine virtuelle réservées* sont soumises à un *quota de processeurs virtuels standard*.
* Les *machines virtuelles Spot* sont soumises à un *quota de processeurs virtuels Spot*.

Pour le type de quota de processeurs virtuels Spot, les quotas de processeurs virtuels Resource Manager sont appliqués à toutes les séries de machines virtuelles disponibles en tant que limite régionale unique.

Chaque fois qu’une nouvelle machine virtuelle Spot est déployée, l’utilisation totale des processeurs virtuels nouveaux et existants de toutes les instances de machine virtuelle Spot ne doit pas dépasser la limite du quota de processeurs virtuels Spot approuvée. Si le quota Spot est dépassé, le déploiement de machine virtuelle Spot n’est pas autorisé.

Cet article explique comment faire une demande d’augmentation concernant le quota de processeurs virtuels Spot dans le portail Azure.

Pour plus d’informations sur les quotas de processeurs virtuels standard, consultez [Quotas de processeurs virtuels pour les machines virtuelles](../../virtual-machines/windows/quotas.md) et [Limites, quotas et contraintes des abonnements et services Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md).

Pour savoir comment augmenter le nombre maximal de processeurs virtuels par région, consultez [Quota standard : augmenter les limites par région](regional-quota-requests.md).

## <a name="request-a-quota-limit-increase-from-help--support"></a>Demander une augmentation de la limite de quota à partir de « Aide + support »

Pour demander une augmentation de la limite de quota pour toutes les séries de machines virtuelles avec **Aide + support** :

> [!NOTE]
> Vous pouvez également demander l’augmentation de la limite de quota pour plusieurs régions à l’aide d’un même cas de support. Pour plus d’informations, reportez-vous à l’étape 8.

1. Dans le menu [Portail Azure](https://portal.azure.com), sélectionnez **Aide + support**.

   ![Lien « Aide + support »](./media/resource-manager-core-quotas-request/help-plus-support.png)

1. Dans **Aide + support**, sélectionnez **Nouvelle demande de support**.

    ![Créer une demande de support](./media/resource-manager-core-quotas-request/new-support-request.png)

1. Pour **Type de problème**, sélectionnez **Limites du service et des abonnements (quotas)** .

   ![Sélectionner un type de problème](./media/resource-manager-core-quotas-request/select-quota-issue-type.png)

1. Pour l’option **Abonnement**, sélectionnez l’abonnement dont vous voulez augmenter le quota.

   ![Sélectionner un abonnement pour une augmentation du quota](./media/resource-manager-core-quotas-request/select-subscription-support-request.png)

1. Pour **Type de quota**, sélectionnez **La limite d’abonnement de calcul/machine virtuelle (cœurs/processeurs virtuels) augmente**.

   ![Sélectionner un type de quota](./media/resource-manager-core-quotas-request/select-quota-type.png)

1. Sélectionnez **Suivant : Solutions** pour ouvrir **DÉTAILS DU PROBLÈME**. Sélectionnez **Fournir des détails** pour entrer des informations supplémentaires.

   ![Lien « Fournir des détails »](./media/resource-manager-core-quotas-request/provide-details-link.png)

1. Dans **Détails du quota**, effectuez les étapes suivantes :

   1. Pour **Modèle de déploiement**, sélectionnez le modèle approprié, et pour **Emplacements**, sélectionnez un emplacement.

      ![Fournir des détails de quota supplémentaires](./media/resource-manager-core-quotas-request/quota-details-deployment-locations.png)

   1. Pour l’emplacement sélectionné, sous **Types**, dans **Sélectionner un type**, choisissez **Spot**.

      ![Sélectionner le type Spot](./media/resource-manager-core-quotas-request/select-spot-type.png)

       Sous **Types**, vous pouvez demander des types de quota Standard et Spot à partir d’un même cas de support grâce à la sélection multiple.

       Pour plus d’informations, consultez [Quota standard : augmenter les limites par série de machines virtuelles](per-vm-quota-requests.md).

   1. Entrez la nouvelle limite de quota que vous souhaitez pour cet abonnement.

      ![Sélectionner un nouveau quota pour la machine virtuelle Spot](./media/resource-manager-core-quotas-request/spot-set-new-quota.png)

1. Pour demander une augmentation de quota pour plusieurs emplacements, sélectionnez un emplacement supplémentaire dans **Emplacements**, puis sélectionnez le type de machine virtuelle approprié. Vous pouvez ensuite entrer une limite qui s’applique à l’emplacement supplémentaire.

   ![Spécifier des emplacements supplémentaires dans les détails du quota](./media/resource-manager-core-quotas-request/quota-details-multiple-locations.png)

1. Sélectionnez **Enregistrer et continuer** pour poursuivre la création de la demande de support.

## <a name="request-a-quota-limit-increase-from-the-subscriptions-pane"></a>Demander une augmentation de la limite de quota à partir du volet « Abonnements »

Pour demander une augmentation de la limite de quota Spot pour toutes les séries de machines virtuelles à partir du volet **Abonnements** :

> [!NOTE]
> Vous pouvez également demander l’augmentation de la limite de quota pour plusieurs régions à l’aide d’un même cas de support. Pour plus d’informations, reportez-vous à l’étape 7.

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **Abonnements**.

   ![Abonnements dans la recherche du portail Azure](./media/resource-manager-core-quotas-request/search-for-subscriptions.png)

1. Sélectionnez l’abonnement pour lequel vous souhaitez augmenter le quota.

   ![Abonnements à sélectionner en vue de changements](./media/resource-manager-core-quotas-request/select-subscription-change-quota.png)

1. Dans le volet de gauche, sélectionnez **Utilisation + quotas**.

   ![Lien « Utilisation + quotas »](./media/resource-manager-core-quotas-request/select-usage-plus-quotas.png)

1. En haut à droite, sélectionnez **Demander une augmentation**.

   ![Sélectionner cette option pour augmenter le quota](./media/resource-manager-core-quotas-request/request-increase-from-subscription.png)

1. Pour **Type de quota**, sélectionnez **La limite d’abonnement de calcul/machine virtuelle (cœurs/processeurs virtuels) augmente**.

   ![Sélectionner un type de quota](./media/resource-manager-core-quotas-request/select-quota-type.png)

1. Sélectionnez **Suivant : Solutions** pour ouvrir **DÉTAILS DU PROBLÈME**. Sélectionnez **Fournir des détails** pour entrer des informations supplémentaires. Dans **Détails du quota**, entrez les informations suivantes :

   1. Pour **Modèle de déploiement**, sélectionnez le modèle approprié, et pour **Emplacements**, sélectionnez un emplacement.

      ![Fournir les détails du quota](./media/resource-manager-core-quotas-request/quota-details-deployment-locations.png)

   1. Pour l’emplacement sélectionné, sous **Types**, dans **Sélectionner un type**, choisissez **Spot**.

      ![Sélectionner le type Spot](./media/resource-manager-core-quotas-request/select-spot-type.png)

      Pour plus d’informations, consultez [Quota standard : augmenter les limites par série de machines virtuelles](per-vm-quota-requests.md).

   1. Entrez la nouvelle limite de quota que vous souhaitez pour cet abonnement.

      ![Entrer une nouvelle valeur pour la limite de processeurs virtuels](./media/resource-manager-core-quotas-request/spot-set-new-quota.png)

1. Pour demander une augmentation de quota pour plusieurs emplacements, sélectionnez un emplacement supplémentaire dans **Emplacements**, puis sélectionnez le type de machine virtuelle approprié. Vous pouvez ensuite entrer une limite qui s’applique à l’emplacement supplémentaire.

   ![Sélectionnez des emplacements supplémentaires dans les détails du quota](./media/resource-manager-core-quotas-request/quota-details-multiple-locations.png)

1. Sélectionnez **Enregistrer et continuer** pour poursuivre la création de la demande de support.
