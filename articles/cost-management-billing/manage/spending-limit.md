---
title: Limite de dépense d’Azure
description: Cet article décrit le fonctionnement de la limite de dépense d’Azure et comment la désactiver.
author: bandersmsft
ms.reviewer: judupont
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: df5c5ed74a5e0a63727962fa86d261cfb588a384
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690559"
---
# <a name="azure-spending-limit"></a>Limite de dépense d’Azure

La limite de dépense dans Azure vous empêche de dépasser la quantité de vos crédits. Pour tout nouveau client qui s’inscrit en vue d’obtenir un compte gratuit Azure ou des types d’abonnements incluant des crédits s’étalant sur plusieurs mois, la limite de dépense est activée par défaut. La limite de dépense est égale au montant de crédit et ne peut pas être modifiée. Par exemple, si vous vous êtes inscrit à un compte gratuit Azure, votre limite de dépense est fixée à 200 USD et vous ne pouvez pas la remplacer par 500 USD. Cependant, vous pouvez supprimer la limite de dépense. Par conséquent, soit vous n’avez aucune limite, soit vous avez une limite égale au montant de votre crédit. Cela vous évite la plupart des dépenses. La limite de dépense n’est pas disponible pour les abonnements impliquant un plan d’engagement ou un paiement à l’utilisation. Pour plus d’informations, consultez la [liste complète des types d’abonnements Azure et la disponibilité de la limite de dépense](https://azure.microsoft.com/support/legal/offer-details/).

## <a name="reaching-a-spending-limit"></a>Atteindre une limite de dépense

Si votre utilisation entraîne des frais dépassant votre limite de dépense, les services que vous avez déployés sont désactivés jusqu’à la fin de la période de facturation.

Par exemple, quand vous dépensez tous les crédits inclus dans votre abonnement gratuit Azure, les ressources Azure que vous déployez sont supprimées de la production et vos machines virtuelles Azure sont arrêtées et désallouées. Les données de vos comptes de stockage sont accessibles en lecture seule.

Si votre offre d’abonnement inclut des crédits sur plusieurs mois, votre abonnement est automatiquement réactivé au début de la période de facturation suivante. Vous pouvez redéployer vos ressources Azure et disposer d’un accès complet à vos comptes de stockage et bases de données.

Azure envoie des notifications par e-mail quand vous atteignez la limite de dépense. Connectez-vous au [portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) pour voir les notifications concernant les abonnements qui ont atteint leur limite de dépense.

Si vous vous êtes inscrit pour obtenir un compte gratuit Azure et avez atteint la limite de dépense, vous pouvez passer à un abonnement avec [paiement à l’utilisation](upgrade-azure-subscription.md) pour supprimer la limite de dépense et réactiver automatiquement l’abonnement.

## <a name="remove-the-spending-limit-in-azure-portal"></a>Supprimer la limite de dépense dans le portail Azure

Vous pouvez supprimer la limite de dépense à tout moment, pour autant qu’un mode de paiement valide soit associé à votre abonnement Azure. Pour les types d’abonnements qui ont un crédit sur plusieurs mois, comme Visual Studio Enterprise ou Visual Studio Professional, vous pouvez choisir de supprimer la limite de dépense indéfiniment ou seulement pour votre période de facturation en cours. Si vous choisissez la période de facturation en cours uniquement, la limite de dépense sera automatiquement activée au début de votre prochaine période de facturation.

Si vous disposez d’un compte gratuit Azure, consultez [Mettre à niveau votre abonnement Azure](upgrade-azure-subscription.md) pour supprimer votre limite de dépense. Sinon, effectuez ces étapes pour supprimer votre limite de dépense :

<a id="remove"></a>

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur de compte.
1. Recherchez **Gestion des coûts + facturation**.

    ![Capture d’écran montrant une recherche relative à la gestion des coûts et à la facturation ](./media/spending-limit/search-bar.png)

1. Dans la liste **Mes abonnements**, sélectionnez votre abonnement. Par exemple, *Visual Studio Enterprise*.

   ![Capture d’écran montrant la grille Mes abonnements dans Vue d’ensemble](./media/spending-limit/cost-management-overview-msdn-x.png)

    > [!NOTE]
    > Si certains de vos abonnements Visual Studio ne s’affichent pas ici, c’est peut-être parce que vous avez modifié un répertoire d’abonnement à un moment donné. Pour ces abonnements, vous devez faire basculer le répertoire vers le répertoire d’origine (le répertoire dans lequel vous vous êtes inscrit initialement). Ensuite, répétez l’étape 2.

1. Dans la vue d’ensemble de l’abonnement, cliquez sur la bannière orange pour supprimer la limite de dépense.

    ![Capture d’écran montrant la bannière de suppression de la limite de dépense](./media/spending-limit/msdn-remove-spending-limit-banner-x.png)

1. Indiquez si vous souhaitez supprimer la limite de dépense définitivement ou uniquement pour la période de facturation actuelle.

      ![Capture d’écran montrant le panneau de suppression de la limite de dépense](./media/spending-limit/remove-spending-limit-blade-x.png)

      | Option | Résultat |
      | --- | --- |
      | Supprimer la limite de dépense pour une durée indéterminée | La limite de dépense ne se réactive pas automatiquement au début de la période de facturation suivante. Vous pouvez cependant la réactiver vous-même à tout moment. |
      | Supprimer la limite de dépense pour la période de facturation en cours | La limite de dépense se réactive automatiquement au début de la période de facturation suivante. |


1. Cliquez sur **Sélectionner le mode de paiement** pour choisir un mode de paiement pour votre abonnement. Celui-ci devient le mode de paiement actif pour votre abonnement.

1. Cliquez sur **Terminer**.


## <a name="why-you-might-want-to-remove-the-spending-limit"></a>Quel est l’intérêt de supprimer la limite de dépense ?

La limite de dépense peut vous empêcher de déployer ou d’utiliser certains services Microsoft et tiers. Voici les situations dans lesquelles vous devez lever la limite de dépense liée à votre abonnement.

-  Vous prévoyez de déployer des images de tiers comme Oracle ou des services comme Azure DevOps Services. Cette situation entraîne presque immédiatement l’atteinte de votre limite de dépense, ce qui a pour effet de désactiver votre abonnement.
- Vous avez des services que vous ne souhaitez pas interrompre. Lorsque vous atteignez votre limite de dépense, les ressources Azure que vous avez déployées sont supprimées de l’environnement de production, et vos machines virtuelles Azure sont arrêtées et désallouées. Si vous avez des services que vous ne souhaitez pas interrompre, vous devez supprimer votre limite de dépense.
- Vous avez des services et ressources utilisant des paramètres tels que des adresses IP virtuelles que vous ne voulez pas perdre. Ces paramètres sont perdus lorsque vous atteignez votre limite de dépense, et les services et ressources sont désalloués.

## <a name="turn-on-the-spending-limit-after-removing"></a>Activer la limite de dépense après l’avoir supprimée

Cette fonctionnalité est disponible uniquement lorsque la limite de dépense a été supprimée de façon indéfinie pour les types d’abonnements qui incluent des crédits sur plusieurs mois. Vous pouvez utiliser cette fonctionnalité pour activer automatiquement votre limite de dépense au début de la période de facturation suivante.


1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur de compte.
1. Recherchez **Gestion des coûts + facturation**.

    ![Capture d’écran montrant une recherche relative à la gestion des coûts et à la facturation ](./media/spending-limit/search-bar.png)

1. Dans la liste **Mes abonnements**, sélectionnez votre abonnement. Par exemple, *Visual Studio Enterprise*.

   ![Capture d’écran montrant la grille Mes abonnements dans Vue d’ensemble](./media/spending-limit/cost-management-overview-msdn-x.png)

    > [!NOTE]
    > Si certains de vos abonnements Visual Studio ne s’affichent pas ici, c’est peut-être parce que vous avez modifié un répertoire d’abonnement à un moment donné. Pour ces abonnements, vous devez faire basculer le répertoire vers le répertoire d’origine (le répertoire dans lequel vous vous êtes inscrit initialement). Ensuite, répétez l’étape 2.

1. Dans la vue d’ensemble de l’abonnement, cliquez sur la bannière en haut de la page pour réactiver la limite de dépense.

## <a name="custom-spending-limit"></a>Limite de dépense personnalisée

Les limites de dépense personnalisées ne sont pas disponibles.

## <a name="a-spending-limit-doesnt-prevent-all-charges"></a>Une limite de dépense n’empêche pas la facturation de tous les frais

[Certains services externes publiés sur la Place de marché Azure](../understand/understand-azure-marketplace-charges.md) ne sont pas utilisables avec des crédits d’abonnement et risquent d’occasionner des frais distincts même si une limite de dépense est définie. C’est le cas, par exemple, des licences Visual Studio, d’Azure Active Directory Premium, des plans de support et de la plupart des services vendus sous marque de tiers. Lors de l’approvisionnement d’un nouveau service externe, un avertissement indique que les services sont facturés séparément :

![Avertissement d’achat Marketplace](./media/spending-limit/marketplace-warning01.png)

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes
- Effectuez une mise à niveau vers un plan avec [paiement à l’utilisation](upgrade-azure-subscription.md).
