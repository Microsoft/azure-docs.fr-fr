---
title: Prise en charge pour Azure Active Directory B2C | Microsoft Docs
description: Dépôt de demandes d’assistance pour Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 12/06/2016
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 5195241003b1ce4ea505002e2cc3c10410e6cde1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "78183718"
---
# <a name="azure-active-directory-b2c-file-support-requests"></a>Azure Active Directory B2C : dépôt de demandes de support
Pour déposer des demandes de support pour Azure Active Directory B2C (Azure AD B2C) dans le portail Azure, procédez comme suit :

1. Basculez de votre client B2C vers un autre client auquel un abonnement Azure est associé. En règle générale, ce dernier est le client de vos employés ou le client par défaut créé pour vous lorsque vous avez souscrit un abonnement Azure. Pour en savoir plus, consultez [Association d’un abonnement Azure à Azure AD](../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).

    ![Portail Azure avec sélection du locataire en surbrillance](./media/support-options/support-switch-dir.png)

1. Après avoir basculé entre les clients, cliquez sur **Aide + Support**.

    ![Vignette Aide + support mise en surbrillance dans le portail Azure](./media/support-options/support-support.png)

1. Cliquez sur **Nouvelle demande de support**.

    ![Nouvelle vignette de demande de support mise en surbrillance dans le portail Azure](./media/support-options/support-new.png)

1. Dans le panneau **Fonctions de base**, renseignez ces informations, puis cliquez sur **Suivant**.

    * Le **Type de problème** est **Technique**.
    * Sélectionnez **l’Abonnement**approprié.
    * Le **Service** est **Active Directory**.
    * Sélectionnez le **Plan de support**approprié. Si vous n’en avez pas, vous pouvez en adopter un [ici](https://azure.microsoft.com/support/plans/).

     ![Page Basics avec le bouton Suivant mis en surbrillance dans le portail Azure](./media/support-options/support-basics.png)

1. Dans le panneau **Problème**, renseignez ces détails, puis cliquez sur **Suivant**.

    * Sélectionnez le niveau de **Gravité** approprié.
    * Le **Type de problème** est **B2C**.
    * Sélectionnez la **Catégorie**appropriée.
    * Décrivez votre problème dans le champ **Détails** . Fournissez des détails comme le nom du client B2C, la description du problème, les messages d’erreur, les ID de corrélation (si disponibles), etc.
    * Dans le champ **Délai d’exécution** , renseignez la date et l’heure (avec le fuseau horaire) de survenue du problème.
    * Sous **Téléchargement de fichiers**, chargez l’ensemble des captures et des fichiers nécessaires à la résolution du problème.

     ![Page Problème avec le bouton Suivant mis en surbrillance dans le portail Azure](./media/support-options/support-problem.png)

1. Dans le panneau **Informations de contact** , renseignez vos coordonnées. Cliquez sur **Créer**.

    ![Page Informations de contact avec le bouton Créer mis en surbrillance dans le portail](./media/support-options/support-contact.png)

1. Après avoir soumis votre demande d’assistance, vous pouvez la surveiller en cliquant sur **Aide + support** dans le tableau d’accueil, puis sur **Gérer les demandes de support**.

## <a name="known-issue-filing-a-support-request-in-the-context-of-a-b2c-tenant"></a>Problème connu : dépôt d’une demande de support dans le contexte d’un client B2C

Si vous avez manqué l'étape 2 décrite ci-dessus et essayez de créer une demande de support dans le contexte de votre client B2C, l'erreur suivante s'affiche.

> [!IMPORTANT]
> N'essayez pas de souscrire un nouvel abonnement Azure dans votre client B2C.

![Aucune erreur d’abonnement ne s’affiche dans le Portail Azure](./media/support-options/support-no-sub.png)
