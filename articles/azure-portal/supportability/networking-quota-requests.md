---
title: Augmentation de la limite réseau
description: Augmentation de la limite réseau
author: anavinahar
ms.author: anavin
ms.date: 01/23/2020
ms.topic: how-to
ms.assetid: ce37c848-ddd9-46ab-978e-6a1445728a3b
ms.openlocfilehash: e945dec2ce8514c3e4f1edecdecd13c5c43f7c75
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745449"
---
# <a name="networking-limit-increase"></a>Augmentation de la limite réseau

Utilisez le [portail Azure](https://portal.azure.com) pour augmenter votre quota réseau.

Pour voir l’utilisation et le quota actuels de votre réseau dans le portail Azure, ouvrez votre abonnement, puis sélectionnez **Utilisation + quotas**. Vous pouvez également utiliser les options suivantes pour voir l’utilisation et les limites de votre réseau.

* [CLI d’utilisation](/cli/azure/network#az-network-list-usages)
* [PowerShell](/powershell/module/azurerm.network/get-azurermnetworkusage)
* [L’API d’utilisation du réseau](/rest/api/virtualnetwork/virtualnetworks/listusage)

Vous pouvez demander une augmentation en utilisant **Aide et support** ou dans **Utilisations + quotas** dans le portail.

> [!Note]
> Pour changer la taille par défaut des **Préfixes d’adresses IP publiques**, sélectionnez **Longueur minimale du préfixe d’interconnexion des adresses IP publiques** dans la liste déroulante.

## <a name="request-networking-quota-increase-at-subscription-level-using-help--support"></a>Demander une augmentation du quota réseau au niveau de l’abonnement avec « Aide et support »

Suivez les instructions ci-dessous pour créer une demande de support en utilisant **Aide et support** dans le portail Azure.

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis sélectionnez **Aide et support** dans son menu, ou recherchez et sélectionnez **Aide et support**.

    ![Aide + support](./media/networking-quota-request/help-plus-support.png)

1. Sélectionnez **Nouvelle demande de support**.

    ![Nouvelle demande de support](./media/networking-quota-request/new-support-request.png)

1. Pour **Type de problème**, choisissez **Limites du service et de l’abonnement (quotas)** .

    ![Sélectionner les limites de l’abonnement dans la liste déroulante des types de problèmes](./media/networking-quota-request/select-quota-issue-type.png)

1. Sélectionnez l’abonnement nécessitant une augmentation du quota.

    ![Sélectionner l’abonnement newSR](./media/networking-quota-request/select-subscription-support-request.png)

1. Sous **Type de quota**, sélectionnez **Réseaux**. Sélectionnez **Suivant : Solutions**.

    ![Sélectionner un type de quota](./media/networking-quota-request/select-quota-type-network.png)

1. Dans **DÉTAILS DU PROBLÈME**, sélectionnez **Fournir des détails**, puis renseignez des informations supplémentaires pour faciliter le traitement de votre demande.

    ![Fournir des informations](./media/networking-quota-request/provide-details-link.png)

1. Dans le panneau **Détails du quota**, sélectionnez le modèle de déploiement, un emplacement et les ressources à inclure dans votre demande.

    ![Détails des quotas : modèle de déploiement](./media/networking-quota-request/quota-details-network.png)

1. Entrez les nouvelles limites que vous souhaitez appliquer à l’abonnement. Pour supprimer une ligne, désélectionnez la ressource dans le menu **Ressources** ou sélectionnez l’icône de fermeture « x ». Après avoir entré le quota pour chaque ressource, sélectionnez **Enregistrer et continuer** pour poursuivre la création de demande de support.

    ![Nouvelles limites](./media/networking-quota-request/network-new-limits.png)

## <a name="request-networking-quota-increase-at-subscription-level-using-usages--quotas"></a>Demander une augmentation du quota réseau au niveau de l’abonnement avec « Utilisations + quotas »

Suivez les instructions suivantes pour créer une demande de support en utilisant **Utilisation + quotas** dans le portail Azure.

1. Dans https://portal.azure.com, recherchez et sélectionnez **Abonnements**.

    ![Abonnements](./media/networking-quota-request/search-for-suscriptions.png)

1. Sélectionnez l’abonnement nécessitant une augmentation du quota.

    ![Sélectionner un abonnement](./media/networking-quota-request/select-subscription-change-quota.png)

1. Sélectionnez **Utilisation + quotas**.

    ![Sélectionnez Utilisation + quotas](./media/networking-quota-request/select-usage-plus-quotas.png)

1. Dans l’angle supérieur droit, cliquez sur **Demander une augmentation**.

    ![Demander une augmentation](./media/networking-quota-request/request-increase-from-subscription.png)

1. Suivez les étapes décrites dans [Demander une augmentation du quota réseau au niveau de l’abonnement](#request-networking-quota-increase-at-subscription-level-using-help--support), en commençant à l’étape 3.

## <a name="about-networking-limits"></a>À propos des limites de mise en réseau

Pour en savoir plus sur les limites réseau, consultez la section [Réseaux](../../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits) de la page Limites ou nos questions fréquentes (FAQ) sur les limites réseau.
