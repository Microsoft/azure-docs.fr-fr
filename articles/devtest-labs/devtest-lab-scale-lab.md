---
title: Limites et quotas de mise à l’échelle dans votre laboratoire Azure DevTest Labs | Microsoft Docs
description: Cet article explique comment mettre à l’échelle votre labo dans Azure DevTest Labs. Affichez vos quotas et limites d’utilisation, et demandez une augmentation.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 2166eaab073e2abc24ca24494ae13eb876db1fcf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87533952"
---
# <a name="scale-quotas-and-limits-in-devtest-labs"></a>Limites et quotas de mise à l’échelle dans votre laboratoire Azure DevTest Labs
Lorsque vous travaillez dans DevTest Labs, vous pouvez remarquer un certain nombre de limites par défaut qui s’appliquent à certaines ressources Azure, ce qui peut affecter le service DevTest Labs. Ces limites sont appelées **quotas**.

> [!NOTE]
> Le service DevTest Labs n’impose aucun quota. Les quotas que vous pouvez rencontrer sont les contraintes par défaut de l’abonnement Azure global.

Vous pouvez utiliser chaque ressource Azure jusqu’à ce que vous ayez atteint son quota. Chaque abonnement a des quotas distincts et l’utilisation est suivie par abonnement.

Par exemple, chaque abonnement possède un quota par défaut de 20 cœurs. Par conséquent, si vous créez des machines virtuelles dans votre laboratoire avec quatre cœurs, vous pouvez uniquement créer cinq machines virtuelles.

La page [Abonnement Azure et limites de service](../azure-resource-manager/management/azure-subscription-service-limits.md) dresse la liste des quotas les plus courants pour les ressources Azure. Les ressources couramment utilisées dans un laboratoire, et pour lesquelles vous pouvez rencontrer des quotas, incluent les noyaux de machine virtuelle, les adresses IP publiques, l’interface réseau, les disques managés, l’attribution de rôle Azure et les circuits ExpressRoute.

## <a name="view-your-usage-and-quotas"></a>Voir votre utilisation et les quotas
Ces étapes vous montrent comment afficher les quotas actuels de votre abonnement pour des ressources Azure spécifiques, ainsi que le pourcentage utilisé pour chaque quota.

1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Sélectionnez **Autres services**, puis **Facturation** dans la liste.
1. Dans le panneau Facturation, sélectionnez un abonnement.
4. Sélectionnez **Utilisation + quotas**.

   ![Bouton Utilisation et quotas](./media/devtest-lab-scale-lab/devtestlab-usage-and-quotas-new.png)

   Le panneau Utilisation + quotas s’affiche. Il répertorie les différentes ressources disponibles dans cet abonnement et le pourcentage de quota utilisé par la ressource.

   ![Quotas et utilisation](./media/devtest-lab-scale-lab/devtestlab-view-quotas-new.png)

## <a name="requesting-more-resources-in-your-subscription"></a>Demander plus de ressources dans votre abonnement
Si vous atteignez le plafond d’un quota, la limite par défaut d’une ressource dans un abonnement peut être augmentée jusqu’à une limite maximale, comme décrit dans [Abonnement Azure et limites de service](../azure-resource-manager/management/azure-subscription-service-limits.md).

Ces étapes vous montrent comment demander une augmentation du quota au moyen du [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Sélectionnez **Autres services**, **Facturation**, puis **Utilisation + quotas**.
1. Dans le panneau Utilisation + quotas, sélectionnez le bouton **Demander une augmentation**.

   ![Bouton Demander une augmentation](./media/devtest-lab-scale-lab/devtestlab-request-increase-new.png)

1. Pour finaliser et envoyer la demande, remplissez les informations requises sous les trois onglets du formulaire **Nouvelle demande de support**.

   ![Formulaire Demander une augmentation](./media/devtest-lab-scale-lab/devtestlab-support-form-new.png)

La page [Understanding Azure Limits and Increases (Présentation des limites et des augmentations Azure)](https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests/) vous explique plus en détail comment contacter le support Azure pour demander une augmentation du quota.



[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

### <a name="next-steps"></a>Étapes suivantes
* Explorez la [Galerie de modèles de démarrage rapide d’Azure Resource Manager DevTest Labs](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates).
