---
title: Demander une augmentation des limites de quota de processeurs virtuels régionaux Azure
description: Guide pratique pour demander une augmentation de la limite de quota de processeurs virtuels pour une région dans le portail Azure.
author: sowmyavenkat86
ms.author: svenkat
ms.date: 01/27/2020
ms.topic: how-to
ms.assetid: ce37c848-ddd9-46ab-978e-6a1445728a3b
ms.openlocfilehash: eadf740c6b5caccbf678a1238f993d4ec0b34095
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745415"
---
# <a name="standard-quota-increase-limits-by-region"></a>Quota standard : augmenter les limites par région

Azure Resource Manager prend en charge deux types de quotas de processeurs virtuels pour les machines virtuelles :

* Les *machines virtuelles avec paiement à l’utilisation* et les *instances de machine virtuelle réservées* sont soumises à un *quota de processeurs virtuels standard*.
* Les *machines virtuelles Spot* sont soumises à un *quota de processeurs virtuels Spot*.

Le quota de processeurs virtuels standard pour le paiement à l’utilisation et les instances de machines virtuelles réservées est appliqué à deux niveaux pour chaque abonnement de chaque région :

* Le premier niveau correspond à la *limite du nombre total de processeurs virtuels régionaux* dans toutes les séries de machines virtuelles.
* Le deuxième niveau correspond à la *limite du nombre de processeurs virtuels pour chaque série de machines virtuelles*, par exemple les processeurs virtuels de série D.

Chaque fois qu’une nouvelle machine virtuelle Spot est déployée, l’utilisation totale des processeurs virtuels nouveaux et existants appartenant à cette série de machines virtuelles ne doit pas dépasser le quota de processeurs virtuels approuvé pour cette série. De plus, le nombre total de processeurs virtuels nouveaux et existants déployés sur toutes les séries de machines virtuelles ne doit pas dépasser le quota total des processeurs virtuels de toutes les régions qui a été approuvé pour l’abonnement. Si l’un de ces quotas est dépassé, le déploiement de la machine virtuelle n’est pas autorisé.

Vous pouvez demander une augmentation de la limite de quota de processeurs virtuels pour une série de machines virtuelles à partir du portail Azure. L’augmentation du quota des séries de machines virtuelles augmente automatiquement la limite totale des processeurs virtuels régionaux de la même valeur.

Lorsque vous créez un nouvel abonnement, le nombre total par défaut de processeurs virtuels régionaux peut ne pas être égal au quota total par défaut de processeurs virtuels pour toutes les séries de machines virtuelles. Avec un tel écart, l’abonnement peut avoir un quota suffisant pour chacune des séries de machines virtuelles à déployer. Toutefois, ce quota peut aussi ne pas être suffisant pour le nombre total de processeurs virtuels régionaux de tous les déploiements. Dans ce cas, vous devez demander explicitement l’augmentation du nombre maximal de processeurs virtuels régionaux. La limite du nombre total de processeurs virtuels régionaux ne peut pas dépasser le quota total approuvé pour toutes les séries de machines virtuelles d’une région.

Pour plus d’informations sur les quotas de processeurs virtuels standard, consultez [Quotas de processeurs virtuels pour les machines virtuelles](../../virtual-machines/windows/quotas.md) et [Limites, quotas et contraintes des abonnements et services Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md).

Pour plus d’informations sur l’augmentation des limites de processeurs virtuels de machines virtuelles Spot, consultez [Quota Spot : augmenter les limites de toutes les séries de machines virtuelles](low-priority-quota.md).

Vous pouvez demander une augmentation de la limite de quota standard des processeurs virtuels par région de deux façons.

## <a name="request-a-quota-increase-by-region-from-help--support"></a>Demander une augmentation du quota par région à partir de « Aide + support »

Pour demander une augmentation des quotas de processeurs virtuels par région à partir de **Aide + support** :

1. Dans le menu [Portail Azure](https://portal.azure.com), sélectionnez **Aide + support**.

   ![Lien « Aide et support »](./media/resource-manager-core-quotas-request/help-plus-support.png)

1. Dans **Aide + support**, sélectionnez **Nouvelle demande de support**.

    ![Nouvelle demande de support](./media/resource-manager-core-quotas-request/new-support-request.png)

1. Pour **Type de problème**, sélectionnez **Limites du service et des abonnements (quotas)** .

   ![Sélectionner un type de problème](./media/resource-manager-core-quotas-request/select-quota-issue-type.png)

1. Pour l’option **Abonnement**, sélectionnez l’abonnement dont vous voulez augmenter le quota.

   ![Sélectionner un abonnement](./media/resource-manager-core-quotas-request/select-subscription-support-request.png)

1. Pour l’option **Type de quota**, sélectionnez **Autres demandes**.

   ![Sélectionner un type de quota](./media/resource-manager-core-quotas-request/regional-quotatype.png)

1. Sélectionnez **Suivant : Solutions** pour ouvrir **DÉTAILS DU PROBLÈME**. Dans **Description**, indiquez les informations suivantes :

    1. Pour **Modèle de déploiement**, spécifiez **Resource Manager**.  
    1. Pour **Région**, spécifiez la région nécessaire, par exemple **USA Est 2**.  
    1. Pour **Nouvelle limite**, spécifiez une nouvelle limite de processeurs virtuels pour la région. Cette valeur ne doit pas dépasser la somme des quotas approuvés pour chacune des séries de références SKU de cet abonnement.

    ![Entrer les détails de la demande de quota](./media/resource-manager-core-quotas-request/regional-details.png)

1. Sélectionnez **Vérifier + créer** pour poursuivre la création de la demande de support.

## <a name="request-a-quota-increase-by-region-from-subscriptions"></a>Demander une augmentation de quota par région à partir de « Abonnements »

Pour demander une augmentation des quotas de processeurs virtuels par région à partir d’**Abonnements** :

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **Abonnements**.

   ![Accéder à Abonnements dans le portail Azure](./media/resource-manager-core-quotas-request/search-for-subscriptions.png)

1. Sélectionnez l’abonnement pour lequel vous souhaitez augmenter le quota.

   ![Sélectionner un abonnement à modifier](./media/resource-manager-core-quotas-request/select-subscription-change-quota.png)

1. Dans le volet de gauche, sélectionnez **Utilisation + quotas**.

   ![Suivre le lien Utilisation et quotas](./media/resource-manager-core-quotas-request/select-usage-plus-quotas.png)

1. En haut à droite, sélectionnez **Demander une augmentation**.

   ![Sélectionner cette option pour augmenter le quota](./media/resource-manager-core-quotas-request/request-increase-from-subscription.png)

1. Dans **Type de quota**, sélectionnez **Autres demandes**.

   ![Sélectionner le type de quota](./media/resource-manager-core-quotas-request/regional-quotatype.png)

1. Sélectionnez **Suivant : Solutions** pour ouvrir **DÉTAILS DU PROBLÈME**. Dans la zone **Description**, indiquez les informations supplémentaires suivantes :

    1. Pour **Modèle de déploiement**, spécifiez **Resource Manager**.  
    1. Pour **Région**, spécifiez la région nécessaire, par exemple **USA Est 2**.  
    1. Pour **Nouvelle limite**, spécifiez une nouvelle limite de processeurs virtuels pour la région. Cette valeur ne doit pas dépasser la somme des quotas approuvés pour chacune des séries de références SKU de cet abonnement.

    ![Entrer des informations dans les détails](./media/resource-manager-core-quotas-request/regional-details.png)

1. Sélectionnez **Vérifier + créer** pour poursuivre la création de la demande de support.
