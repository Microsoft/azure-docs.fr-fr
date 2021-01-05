---
title: Modèle de déploiement classique Azure
description: Le modèle de déploiement Classic, désormais remplacé par le modèle Resource Manager, applique une limite de quota de processeurs virtuels globale aux machines virtuelles et groupes de machines virtuelles identiques.
author: sowmyavenkat86
ms.author: svenkat
ms.date: 01/27/2020
ms.topic: how-to
ms.assetid: ce37c848-ddd9-46ab-978e-6a1445728a3b
ms.openlocfilehash: b5b5e8c11bfe164aaa3539742dac8c4d267c69e1
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745602"
---
# <a name="classic-deployment-model"></a>Modèle de déploiement classique

Le modèle de déploiement Classic est le modèle de déploiement Azure de génération antérieure. Il applique une limite de quota de processeur virtuel global pour les machines virtuelles et les groupes de machines virtuelles identiques. Le modèle de déploiement Classic n’est plus recommandé. Il est désormais remplacé par le modèle Resource Manager.

Pour en savoir plus sur ces deux modèles de déploiement et sur les avantages liés à l’utilisation de Resource Manager, consultez [Déploiements Resource Manager et Classic](../../azure-resource-manager/management/deployment-models.md).

Quand un abonnement est créé, un quota de processeurs virtuels par défaut lui est attribué. Chaque fois qu’une nouvelle machine virtuelle doit être déployée à l’aide du modèle de déploiement Classic, la somme de l’utilisation des processeurs virtuels nouveaux et existants dans toutes les régions ne doit pas dépasser le quota de processeur virtuel approuvé pour le modèle de déploiement Classic.

Pour en savoir plus sur les quotas, consultez [Abonnement Azure et limites, quotas et contraintes de service](../../azure-resource-manager/management/azure-subscription-service-limits.md).

Vous pouvez demander une augmentation de la limite de quota de processeurs virtuels pour le modèle de déploiement Classic. Utilisez **Aide + support** ou **Utilisation + quotas** dans le portail Azure.

## <a name="request-per-vm-series-vcpu-quota-increase-at-subscription-level-using-help--support"></a>Demander une augmentation du quota total de processeurs virtuels par série de machines virtuelles au niveau de l’abonnement en utilisant Aide + support

Suivez les instructions ci-dessous pour créer une demande de support en utilisant **Aide et support** dans le portail Azure.

1. Dans le menu [Portail Azure](https://portal.azure.com), sélectionnez **Aide + support**.

   ![Sélectionner Aide + support dans le portail Azure](./media/resource-manager-core-quotas-request/help-plus-support.png)

1. Sélectionnez **Nouvelle demande de support**.

   ![Créer une demande de support dans le portail Azure](./media/resource-manager-core-quotas-request/new-support-request.png)

1. Dans **Type de problème**, sélectionnez **Limites du service et de l’abonnement (quotas)** .

   ![Sélectionner des quotas comme le type de problème](./media/resource-manager-core-quotas-request/select-quota-issue-type.png)

1. Sélectionnez l’abonnement pour lequel vous souhaitez augmenter le quota.

   ![Sélectionner un abonnement pour lequel augmenter un quota](./media/resource-manager-core-quotas-request/select-subscription-support-request.png)

1. Pour **Type de quota**, sélectionnez **La limite d’abonnement de calcul/machine virtuelle (cœurs/processeurs virtuels) augmente**.

   ![Sélectionner le type de quota à augmenter](./media/resource-manager-core-quotas-request/select-quota-type.png)

1. Sélectionnez **Suivant : Solutions** pour ouvrir **DÉTAILS DU PROBLÈME**. Sélectionnez **Fournir des détails** pour fournir des informations supplémentaires.

   ![Fournir des détails pour faciliter la demande](./media/resource-manager-core-quotas-request/provide-details-link.png)

1. Dans **Détails du quota**, sélectionnez **Classique**, puis un **emplacement**.

   ![Ajouter des détails, notamment le modèle de déploiement et l’emplacement](./media/resource-manager-core-quotas-request/quota-details-classic.png)

1. Pour la **famille de références SKU**, sélectionnez une ou plusieurs familles de références SKU à augmenter.

   ![Spécifier la famille de références SKU à augmenter](./media/resource-manager-core-quotas-request/sku-family-classic.png)

1. Entrez les nouvelles limites que vous souhaitez appliquer à l’abonnement. Pour supprimer une ligne, désélectionnez la référence SKU dans **Famille de références SKU** ou sélectionnez l’icône de fermeture « X ». Après avoir entré un quota pour chaque famille de références SKU, sélectionnez **Enregistrer et continuer** dans **Détails du quota** afin de poursuivre la demande de support.

   ![Demander de nouvelles limites](./media/resource-manager-core-quotas-request/new-limits-classic.png)

## <a name="request-per-vm-series-vcpu-quota-increase-at-subscription-level-using-usage--quotas"></a>Demandez une augmentation du quota total de processeurs virtuels par série de machines virtuelles au niveau de l’abonnement avec Utilisation + quotas

Suivez les instructions ci-dessous pour créer une demande de support à l'aide de **Utilisation + quotas** dans le portail Azure.

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **Abonnements**.

   ![Accéder à Abonnements dans le portail Azure](./media/resource-manager-core-quotas-request/search-for-subscriptions.png)

1. Sélectionnez l’abonnement pour lequel vous souhaitez augmenter le quota.

   ![Sélectionner l'abonnement à modifier](./media/resource-manager-core-quotas-request/select-subscription-change-quota.png)

1. Sélectionnez **Utilisation + quotas**.

   ![Sélectionner Utilisation et quotas pour un abonnement](./media/resource-manager-core-quotas-request/select-usage-plus-quotas.png)

1. Dans l’angle supérieur droit, cliquez sur **Demander une augmentation**.

   ![Sélectionner cette option pour augmenter le quota](./media/resource-manager-core-quotas-request/request-increase-from-subscription.png)

1. Sélectionnez **La limite d’abonnement de calcul/machine virtuelle (cœurs/processeurs virtuels) augmente** comme **type de quota**.

   ![Sélectionner un type de quota](./media/resource-manager-core-quotas-request/select-quota-type.png)

1. Sélectionnez **Suivant : Solutions** pour ouvrir **DÉTAILS DU PROBLÈME**. Sélectionnez **Fournir des détails** pour fournir des informations supplémentaires.

   ![Fournir les détails de la requête](./media/resource-manager-core-quotas-request/provide-details-link.png)

1. Dans **Détails du quota**, sélectionnez **Classique**, puis un **emplacement**.

   ![Sélectionner les détails de quota, notamment le modèle de déploiement et l’emplacement](./media/resource-manager-core-quotas-request/quota-details-classic.png)

1. Sélectionnez une ou plusieurs familles de références SKU à augmenter.

   ![Sélectionner la famille de références SKU à augmenter](./media/resource-manager-core-quotas-request/sku-family-classic.png)

1. Entrez les nouvelles limites que vous souhaitez appliquer à l’abonnement. Pour supprimer une ligne, désélectionnez la référence SKU dans **Famille de références SKU** ou sélectionnez l’icône de fermeture « X ». Après avoir entré un quota pour chaque famille de références SKU, sélectionnez **Enregistrer et continuer** dans **Détails du quota** afin de poursuivre la demande de support.

   ![Entrer un nouveau quota](./media/resource-manager-core-quotas-request/new-limits-classic.png)

