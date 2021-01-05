---
title: Demander une augmentation des limites de quota de processeurs virtuels par série de machines virtuelles Azure
description: Découvrez comment demander une augmentation de la limite de quota de processeurs virtuels pour une série de machines virtuelles dans le portail Azure, ce qui augmente la limite du nombre total de processeurs virtuels régionaux de la même valeur.
author: sowmyavenkat86
ms.author: svenkat
ms.date: 01/27/2020
ms.topic: how-to
ms.assetid: ce37c848-ddd9-46ab-978e-6a1445728a3b
ms.openlocfilehash: 5db3e538a64e275313e1e0ab01f6cc6350eabb77
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745432"
---
# <a name="standard-quota-increase-limits-by-vm-series"></a>Quota standard : augmenter les limites par série de machines virtuelles

Azure Resource Manager prend en charge deux types de quotas de processeurs virtuels pour les machines virtuelles :

* Les *machines virtuelles avec paiement à l’utilisation* et les *instances de machine virtuelle réservées* sont soumises à un *quota de processeurs virtuels standard*.
* Les *machines virtuelles Spot* sont soumises à un *quota de processeurs virtuels Spot*.

Le quota de processeurs virtuels standard pour le paiement à l’utilisation et les instances de machines virtuelles réservées est appliqué à deux niveaux pour chaque abonnement de chaque région :

* Le premier niveau correspond à la *limite du nombre total de processeurs virtuels régionaux* dans toutes les séries de machines virtuelles.
* Le deuxième niveau correspond à la *limite du nombre de processeurs virtuels pour chaque série de machines virtuelles*, par exemple les processeurs virtuels de série Dv3.

Chaque fois qu’une nouvelle machine virtuelle Spot est déployée, l’utilisation totale des processeurs virtuels nouveaux et existants de toutes les instances de machine virtuelle Spot ne doit pas dépasser la limite du quota de processeurs virtuels Spot approuvée. Si le quota Spot est dépassé, le déploiement de machine virtuelle Spot n’est pas autorisé.

Vous pouvez demander une augmentation de la limite de quota de processeurs virtuels pour une série de machines virtuelles à partir du portail Azure. L’augmentation du quota des séries de machines virtuelles augmente automatiquement la limite totale des processeurs virtuels régionaux de la même valeur.

Pour plus d’informations sur les quotas de processeurs virtuels standard, consultez [Quotas de processeurs virtuels pour les machines virtuelles](../../virtual-machines/windows/quotas.md) et [Limites des abonnements et services Azure](./classic-deployment-model-quota-increase-requests.md).

Pour savoir comment augmenter le nombre maximal de processeurs virtuels par région pour le quota Standard, consultez [Quota standard : augmenter les limites par région](regional-quota-requests.md).

Pour plus d’informations sur l’augmentation des limites de processeurs virtuels de machines virtuelles Spot, consultez [Quota Spot : augmenter les limites de toutes les séries de machines virtuelles](low-priority-quota.md).

Il existe deux façons de demander une augmentation des limites de quota de processeurs virtuels standard pour chaque série de machines virtuelles, comme indiqué dans les sections suivantes.

## <a name="request-a-standard-quota-increase-from-help--support"></a>Demander une augmentation du quota standard à partir de « Aide + support »

Pour demander une augmentation du quota de processeurs virtuels standard pour chaque série de machines virtuelles à partir d’**Aide + support** :

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

   ![Fournir des détails de quota supplémentaires](./media/resource-manager-core-quotas-request/quota-details-deployment-rm-locations.png)

   1. Pour **Modèle de déploiement**, sélectionnez le modèle approprié.

   1. Pour **Emplacements**, sélectionnez un emplacement. Pour l’emplacement sélectionné, sous **Types**, dans **Sélectionner un type**, choisissez **Standard**.

      ![Détails du quota - Types de quotas](./media/resource-manager-core-quotas-request/quota-details-select-standard-type.png)

      Sous **Types**, vous pouvez demander des types de quota Standard et Spot à partir d’un même cas de support grâce à la sélection multiple.

      Pour plus d’informations sur l’augmentation des limites de quota Spot, consultez [Machines virtuelles Spot Azure pour les groupes de machines virtuelles identiques](../../virtual-machine-scale-sets/use-spot.md).

   1. Dans **Standard**, sélectionnez la série de références SKU pour augmenter les quotas.

      ![Détails du quota - Série de références SKU](./media/resource-manager-core-quotas-request/quota-details-standard-select-series.png)

   1. Entrez la nouvelle limite de quota que vous souhaitez pour cet abonnement. Pour supprimer une référence SKU de votre liste, décochez la case en regard de la référence SKU ou sélectionnez l’icône d’abandon « X ».

      ![Sélectionner une nouvelle limite de processeurs virtuels](./media/resource-manager-core-quotas-request/quota-details-standard-set-vcpu-limit.png)

1. Pour demander une augmentation de quota pour plusieurs emplacements, sélectionnez un emplacement supplémentaire dans **Emplacements**, puis sélectionnez le type de machine virtuelle approprié. Vous pouvez ensuite entrer une limite qui s’applique à l’emplacement supplémentaire.

   ![Spécifier des emplacements supplémentaires dans les détails du quota](./media/resource-manager-core-quotas-request/quota-details-multiple-locations.png)

1. Sélectionnez **Enregistrer et continuer** pour poursuivre la création de la demande de support.

## <a name="request-a-standard-quota-increase-from-subscriptions"></a>Demander une augmentation du quota standard à partir de « Abonnements »

Pour demander une augmentation du quota de processeurs virtuels standard pour chaque série de machines virtuelles à partir d’**Abonnements** :

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

1. Dans **Détails du quota**, effectuez les étapes suivantes :

   1. Pour **Modèle de déploiement**, sélectionnez le modèle approprié, et pour **Emplacements**, sélectionnez un emplacement.

      ![Fournir les détails du quota](./media/resource-manager-core-quotas-request/quota-details-deployment-rm-locations.png)

   1. Pour l’emplacement sélectionné, sous **Types**, sélectionnez **Sélectionner un type**, puis **Standard**.

      ![Sélectionner le type Standard](./media/resource-manager-core-quotas-request/quota-details-select-standard-type.png)

      Sous **Types**, vous pouvez demander des types de quota Standard et Spot à partir d’un même cas de support grâce à la sélection multiple.

      Pour plus d’informations sur l’augmentation des limites de quota Spot, consultez [Machines virtuelles Spot Azure pour les groupes de machines virtuelles identiques](../../virtual-machine-scale-sets/use-spot.md).

   1. Pour **Standard**, sélectionnez la série de références SKU pour laquelle vous souhaitez augmenter les quotas.

      ![Détails du quota - Série de références SKU](./media/resource-manager-core-quotas-request/quota-details-standard-select-series.png)

   1. Entrez la nouvelle limite de quota que vous souhaitez pour cet abonnement. Pour supprimer une référence SKU de la liste, décochez la case en regard de la référence SKU ou sélectionnez l’icône « X ».

      ![Sélectionner une nouvelle limite de processeurs virtuels](./media/resource-manager-core-quotas-request/quota-details-standard-set-vcpu-limit.png)

1. Pour demander une augmentation de quota pour plusieurs emplacements, sélectionnez un emplacement supplémentaire dans **Emplacements**, puis sélectionnez le type de machine virtuelle approprié.

   Cette étape précharge la série de références SKU que vous avez sélectionnée pour les précédents emplacements. Entrez les limites de quota que vous souhaitez appliquer à cette série.

   ![Sélectionnez des emplacements supplémentaires dans les détails du quota](./media/resource-manager-core-quotas-request/quota-details-multiple-locations.png)

1. Sélectionnez **Enregistrer et continuer** pour poursuivre la création de la demande de support.