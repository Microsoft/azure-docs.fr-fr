---
title: Tâches d’administrateur de compte dans le portail Azure
description: Décrit comment effectuer des opérations de paiement dans le portail Azure
author: bandersmsft
ms.reviewer: judupont
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 10/26/2020
ms.author: banders
ms.custom: contperf-fy21q2
ms.openlocfilehash: bd46e7b2f0713da67842def47dfeadc837027d8f
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97027966"
---
# <a name="account-administrator-tasks-in-the-azure-portal"></a>Tâches d’administrateur de compte dans le portail Azure

Cet article explique comment effectuer les tâches suivantes dans le portail Azure :
- Gérer les modes de paiement de votre abonnement
- Supprimer la limite de dépense de votre abonnement
- Ajouter des crédits à votre abonnement Azure dans Open

Vous devez être l’administrateur de compte pour effectuer l’une de ces tâches.

## <a name="navigate-to-your-subscriptions-payment-methods"></a>Accéder aux modes de paiement de votre abonnement

1. Connectez-vous au portail Azure en tant qu’administrateur de compte.

1. Recherchez **Gestion des coûts + facturation**.

    ![Capture d’écran montrant une recherche relative à la gestion des coûts et à la facturation ](./media/account-admin-tasks/search-bar.png)

1. Dans la liste **Mes abonnements**, sélectionnez l’abonnement auquel vous voulez ajouter la carte de crédit.

   ![Capture d’écran montrant la page Cost Management + Billing dans laquelle vous pouvez sélectionner un abonnement.](./media/account-admin-tasks/cost-management-billing-overview-x.png)

   > [!NOTE]
   > Si certains de vos abonnements ne s’affichent pas ici, c’est peut-être parce que vous avez modifié le répertoire de l’abonnement à un moment donné. Pour ces abonnements, vous devez faire basculer le répertoire vers le répertoire d’origine (le répertoire dans lequel vous vous êtes inscrit initialement). Ensuite, répétez l’étape 2.

1. Sélectionnez **Modes de paiement**.

    ![Capture d’écran montrant la page des modes de paiement dans laquelle vous pouvez ajouter un mode de paiement.](./media/account-admin-tasks/subscription-payment-methods-blade.png)

Vous pouvez ici ajouter une nouvelle carte de crédit, modifier le mode de paiement actif, modifier les détails de la carte de crédit et supprimer des cartes de crédit.

### <a name="change-active-payment-method"></a>Modifier le mode de paiement actif

Vous pouvez modifier le mode de paiement actif en ajoutant une nouvelle carte de crédit ou en choisissant celle qui est déjà enregistrée. Pour modifier le mode de paiement actif par une nouvelle carte de crédit :

1. Dans le coin supérieur gauche, sélectionnez « + » pour ajouter une carte de crédit.

    ![Capture d’écran montrant le signe plus](./media/account-admin-tasks/subscription-payment-methods-plus.png)

1. Entrez les détails de la carte de crédit dans le formulaire à droite.

    ![Capture d’écran montrant le formulaire d’ajout de carte de crédit.](./media/account-admin-tasks/subscription-add-payment-method-x.png)

1. Pour faire de cette carte le mode de paiement actif, cochez la case en regard de **Définir comme mon mode de paiement actif** au-dessus du formulaire. Cette carte devient l’instrument de paiement actif pour tous les abonnements utilisant la même carte que l’abonnement sélectionné.

    ![Capture d’écran montrant la case à cocher de définition du mode de paiement actif par carte.](./media/account-admin-tasks/subscription-make-active-payment-method-x.png)

1. Sélectionnez **Suivant**.

Pour modifier le mode de paiement actif par une carte de crédit qui est déjà enregistrée :

1. Cochez la case en regard de la carte que vous souhaitez définir comme mode de paiement actif.

    ![Capture d’écran montrant la case cochée en regard de la carte de crédit](./media/account-admin-tasks/subscription-checked-payment-method-x.png)

1. Cliquez sur **Définir comme actif** dans la barre de commandes.

    ![Capture d’écran montrant le bouton Définir comme actif](./media/account-admin-tasks/subscription-checked-payment-method-set-active.png)

### <a name="edit-credit-card-details"></a>Entrer les détails de la carte de crédit

Pour modifier les détails de la carte de crédit, tels que la date d’expiration ou l’adresse, cliquez sur la carte de crédit que vous souhaitez modifier. Un formulaire de carte de crédit s’affiche à droite.

![Capture d’écran montrant la carte de crédit sélectionnée](./media/account-admin-tasks/subscription-edit-payment-method-x.png)

Mettez à jour les détails de la carte de crédit et cliquez sur **Enregistrer**.

### <a name="remove-a-credit-card-from-the-account"></a>Suppression d’une carte de crédit à partir du compte

1. Cochez la case en regard de la carte que vous souhaitez supprimer.

    ![Capture d’écran montrant la case cochée en regard de la carte de crédit](./media/account-admin-tasks/subscription-checked-payment-method-x.png)

1. Cliquez sur **Supprimer** dans la barre de commandes.

    ![Capture d’écran montrant le bouton Supprimer](./media/account-admin-tasks/subscription-checked-payment-method-delete.png)

Si votre carte de crédit est le mode de paiement actif pour un de vos abonnements Microsoft, vous ne pouvez pas la supprimer de votre compte Azure. Modifiez le mode de paiement actif pour tous les abonnements liés à cette carte de crédit, puis réessayez.

### <a name="switch-to-invoice-payment"></a>Passer au paiement de la facture

Si vous êtes éligible au paiement par facture (chèque/virement bancaire), vous pouvez faire passer votre abonnement au paiement de facture (chèque/virement bancaire) dans le portail Azure.

1. Sélectionnez **Payer par facture** dans la barre de commandes.

    ![Capture d’écran montrant la page des modes de paiement avec le mode de paiement par facture sélectionné.](./media/account-admin-tasks/subscription-payment-methods-pay-by-invoice.png)

1. Entrez l’adresse du mode de paiement de la facture.
1. Cliquez sur **Suivant**.

Si vous souhaitez être approuvé pour payer par facture, [découvrez comment payer par facture](pay-by-invoice.md).

### <a name="edit-invoice-payment-address"></a>Modifier l’adresse de paiement de la facture

Pour modifier l’adresse de votre mode de paiement de facture, cliquez sur **Facture** dans la liste des modes de paiement de votre abonnement. Le formulaire d’adresse s’ouvre à droite.

## <a name="remove-spending-limit"></a>Supprimer la limite de dépense

La limite de dépense dans Azure vous empêche de dépasser la quantité de vos crédits. Vous pouvez supprimer la limite de dépense à tout moment, pour autant qu’un mode de paiement valide soit associé à votre abonnement Azure. Pour les types d’abonnements qui ont un crédit sur plusieurs mois, comme Visual Studio Enterprise ou Visual Studio Professional, vous pouvez également choisir de réactiver la limite de dépense au début de votre prochaine période de facturation.

La limite de dépense n’est pas disponible pour les abonnements impliquant un plan d’engagement ou un paiement à l’utilisation.

1. Connectez-vous au portail Azure en tant qu’administrateur de compte.
1. Recherchez **Gestion des coûts + facturation**.

    ![Capture d’écran montrant une recherche relative à la gestion des coûts et à la facturation ](./media/account-admin-tasks/search-bar.png)

1. Dans la liste **Mes abonnements**, sélectionnez votre abonnement Visual Studio Enterprise.

   ![Capture d’écran montrant la zone Mes abonnements, où vous pouvez sélectionner votre abonnement Visual Studio Enterprise.](./media/account-admin-tasks/cost-management-overview-msdn-x.png)

    > [!NOTE]
    > Si certains de vos abonnements Visual Studio ne s’affichent pas ici, c’est peut-être parce que vous avez modifié un répertoire d’abonnement à un moment donné. Pour ces abonnements, vous devez faire basculer le répertoire vers le répertoire d’origine (le répertoire dans lequel vous vous êtes inscrit initialement). Ensuite, répétez l’étape 2.

1. Dans la vue d’ensemble de l’abonnement, cliquez sur la bannière orange pour supprimer la limite de dépense.

    ![Capture d’écran montrant la bannière de suppression de la limite de dépense](./media/account-admin-tasks/msdn-remove-spending-limit-banner-x.png)

1. Indiquez si vous souhaitez supprimer la limite de dépense définitivement ou uniquement pour la période de facturation actuelle.

   ![Capture d’écran montrant le panneau de suppression de la limite de dépense](./media/account-admin-tasks/remove-spending-limit-blade-x.png)

1. Cliquez sur **Sélectionner le mode de paiement** pour choisir un mode de paiement pour votre abonnement. Celui-ci devient le mode de paiement actif pour votre abonnement.

1. Cliquez sur **Terminer**.

## <a name="add-credits-to-azure-in-open-subscription"></a>Ajouter des crédits à votre abonnement Azure dans Open

Si vous disposez d’un abonnement Azure en licence Open, vous pouvez ajouter des crédits à votre abonnement dans le portail Azure en échangeant une clé de produit ou en achetant des crédits avec une carte de crédit.

1. Connectez-vous au portail Azure en tant qu’administrateur de compte.
1. Recherchez **Gestion des coûts + facturation**.

    ![Capture d’écran montrant une recherche relative à la gestion des coûts et à la facturation ](./media/account-admin-tasks/search-bar.png)

1. Dans la liste **Mes abonnements**, sélectionnez l’abonnement Azure dans Open.

    ![Capture d’écran montrant la zone Mes abonnements, où vous pouvez sélectionner votre abonnement Azure dans Open.](./media/account-admin-tasks/cost-management-overview-aio-x.png)

   > [!NOTE]
   > Si votre abonnement ne s’affiche pas ici, c’est peut-être parce que vous en avez modifié le répertoire à un moment donné. Vous devez faire basculer le répertoire de l’abonnement vers le répertoire d’origine (le répertoire dans lequel vous vous êtes inscrit initialement). Ensuite, répétez l’étape 2.

1. Sélectionnez **Historique de crédit**.

    ![Capture d’écran montrant l’historique de crédit](./media/account-admin-tasks/aio-credit-history-blade.png)

1. Dans le coin supérieur gauche, sélectionnez « + » pour ajouter d’autres crédits.

    ![Capture d’écran montrant le bouton d’ajout de crédits](./media/account-admin-tasks/aio-credit-history-plus.png)

1. Sélectionnez un type de mode de paiement dans la liste déroulante. Vous pouvez ajouter une clé de produit ou acheter des crédits à l’aide d’une carte de crédit.

    ![Capture d’écran de la liste déroulante de mode de paiement dans le panneau d’ajout de crédits](./media/account-admin-tasks/add-credits-select-payment-method.png)

1. Si vous avez choisi une clé de produit :
    - Entrez la clé de produit
    - Cliquez sur **Valider**

1. Si vous avez choisi une carte de crédit :
    - Cliquez sur **Sélectionner le mode de paiement** pour ajouter une carte de crédit ou en sélectionner une existante.
    - Spécifiez le nombre de crédits que vous souhaitez ajouter.

1. Cliquez sur **Appliquer**

## <a name="troubleshooting"></a>Dépannage
Les cartes virtuelles ou prépayées ne sont pas prises en charge. Si vous obtenez des erreurs lors de l’ajout ou de la mise à jour d’une carte de crédit valide, essayez d’ouvrir votre navigateur en mode privé.

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur l’[analyse des frais inattendus](../understand/analyze-unexpected-charges.md)
