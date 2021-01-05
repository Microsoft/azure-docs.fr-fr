---
title: Mettre à jour une délégation
description: Découvrez comment mettre à jour une délégation pour un client précédemment intégré à Azure Lighthouse.
ms.date: 12/03/2020
ms.topic: how-to
ms.openlocfilehash: e204d1f3546e6e978f91c7e808065a388a4af4b3
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97093415"
---
# <a name="update-a-delegation"></a>Mettre à jour une délégation

Une fois que vous avez intégré un abonnement (ou un groupe de ressources) à Azure Lighthouse, vous devrez peut-être apporter des modifications. Par exemple, votre client peut vouloir que vous assumiez des tâches de gestion supplémentaires qui nécessitent un rôle intégré Azure différent, ou vous pouvez avoir besoin de modifier le locataire auquel un abonnement client est délégué.

> [!TIP]
> Même si nous faisons ici référence aux fournisseurs de services et aux clients, les [entreprises gérant plusieurs locataires](../concepts/enterprise.md) peuvent suivre le même processus pour configurer Azure Lighthouse et consolider leur expérience de gestion.

Si vous [avez intégré votre client via des modèles Azure Resource Manager (modèles ARM)](onboard-customer.md), un nouveau déploiement doit être effectué pour ce client. En fonction de ce que vous modifiez, vous souhaiterez peut-être mettre à jour l’offre d’origine ou la supprimer pour en créer une nouvelle.

- **Si vous modifiez uniquement les autorisations** : Vous pouvez mettre à jour votre délégation en modifiant uniquement la section **Autorisations** du modèle ARM.
- **Si vous modifiez le locataire gestionnaire** : Vous devez créer un nouveau modèle ARM en utilisant un **mspOfferName** différent de votre offre précédente.

## <a name="update-your-arm-template"></a>Mettre à jour votre modèle ARM

Pour mettre à jour votre délégation, vous devez déployer un modèle ARM qui comprend les modifications que vous souhaitez apporter.

Si vous mettez uniquement à jour les autorisations, comme l’ajout d’un groupe d’utilisateurs avec un rôle que vous n’aviez pas inclus précédemment ou la modification du rôle d’un utilisateur existant, vous pouvez utiliser le même **mspOfferName** que dans le [modèle ARM](onboard-customer.md#create-an-azure-resource-manager-template) que vous avez utilisé pour la délégation précédente. Vous pouvez utiliser votre modèle précédent comme point de départ. Ensuite, apportez les modifications nécessaires, par exemple en remplaçant un rôle intégré Azure par un autre, ou en ajoutant une autorisation entièrement nouvelle au modèle.

Si vous modifiez le **mspOfferName**, cela sera considéré comme une nouvelle offre distincte. Cela est nécessaire si vous modifiez le locataire gestionnaire.

Il n’est pas nécessaire de modifier le **mspOfferName** si le locataire gestionnaire reste le même. Dans la plupart des cas, nous vous recommandons de n’avoir qu’un seul **mspOfferName** utilisé par le même client et le même locataire gestionnaire. Si vous choisissez de le changer quand même, assurez-vous que la délégation précédente du client est supprimée avant de déployer la nouvelle.

## <a name="remove-the-previous-delegation"></a>Supprimer la délégation précédente

Avant d’effectuer un nouveau déploiement, vous pouvez [supprimer l’accès à la délégation précédente](remove-delegation.md). Cela permet de s’assurer que toutes les autorisations précédentes sont supprimées, ce qui vous permet de commencer proprement avec les utilisateurs/groupes et les rôles exacts qui doivent s’appliquer à l’avenir.

> [!IMPORTANT]
> Si vous utilisez un nouveau **mspOfferName** et que vous conservez les mêmes valeurs de **principalId**, vous devez supprimer l’accès à la délégation précédente avant de déployer la nouvelle offre. Si vous ne supprimez pas d’abord l’offre, les utilisateurs qui avaient précédemment reçu une autorisation peuvent perdre leur accès en raison d’attributions conflictuelles.

Si vous modifiez le locataire gestionnaire, vous pouvez laisser l’offre précédente en place si vous souhaitez que les deux locataires continuent à y avoir accès. Si vous souhaitez que seul le nouveau locataire gestionnaire ait accès, l’offre précédente doit être supprimée. Vous pouvez effectuer cette opération avant ou après l’intégration de la nouvelle offre.

Si vous mettez à jour l’offre pour ajuster les autorisations uniquement et que vous conservez le même **mspOfferName**, vous n’êtes pas obligé de supprimer la délégation précédente. Le nouveau déploiement remplacera la délégation précédente, et seules les autorisations du modèle le plus récent s’appliqueront.

:::image type="content" source="../media/update-delegation.jpg" alt-text="Diagramme illustrant quand modifier mspOfferName et supprimer une délégation précédente.":::

La suppression de l’accès à la délégation peut être effectuée par n’importe quel utilisateur du locataire gestionnaire qui a reçu le rôle [Suppression de l’attribution d’inscription de services gérés](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role) dans la délégation d’origine. Si aucun utilisateur de votre locataire gestionnaire n’a ce rôle, vous pouvez demander au client de [supprimer l’accès à l’offre dans le portail Azure](view-manage-service-providers.md#add-or-remove-service-provider-offers).

> [!TIP]
> Si vous avez supprimé la délégation précédente en suivant les étapes ci-dessus et que vous ne parvenez toujours pas à déployer le nouveau modèle ARM, vous devrez peut-être [supprimer complètement la définition d’inscription](/powershell/module/az.managedservices/remove-azmanagedservicesdefinition). Cela peut être effectué par tout utilisateur ayant le rôle de propriétaire dans le locataire client.  

## <a name="deploy-the-arm-template"></a>Déployer le modèle ARM

Votre client peut [déployer le modèle mis à jour](onboard-customer.md#deploy-the-azure-resource-manager-templates) de la même manière qu’auparavant : dans le portail Azure, à l’aide de PowerShell ou à l’aide d’Azure CLI.

Une fois le déploiement terminé, [confirmez sa réussite](onboard-customer.md#confirm-successful-onboarding). Les autorisations mises à jour seront alors appliquées pour l’abonnement ou les groupes de ressources que le client a délégués.

## <a name="updating-managed-service-offers"></a>Mise à jour des offres de service géré

Si vous avez intégré votre client via une offre de service géré publiée sur Place de marché Azure et que vous souhaitez mettre à jour les autorisations, vous pouvez mettre à jour la délégation en [publiant une nouvelle version de votre offre](../../marketplace/partner-center-portal/update-existing-offer.md) avec les [autorisations](../../marketplace/partner-center-portal/create-new-managed-service-offer.md#authorization) que vous souhaitez utiliser mises à jour dans le plan de ce client. Le client sera ensuite en mesure de mettre à jour vers la version la plus récente dans le portail Azure.

Si vous souhaitez modifier le locataire gestionnaire, vous devez [créer et publier une nouvelle offre de service géré](../../marketplace/partner-center-portal/create-new-managed-service-offer.md) que le client doit accepter.

> [!TIP]
> Comme mentionné précédemment, nous vous recommandons de ne pas utiliser plusieurs offres différentes entre le même client et le même locataire gestionnaire. Si vous publiez une nouvelle offre pour le même client qui utilise le même locataire gestionnaire, assurez-vous que l’offre précédente est supprimée avant que le client accepte la nouvelle offre.

## <a name="next-steps"></a>Étapes suivantes

- [Affichez et gérez les clients](view-manage-customers.md) en accédant à **Mes clients** sur le portail Azure.
- Découvrez comment [supprimer l’accès à une délégation](remove-delegation.md) qui a été intégrée.
