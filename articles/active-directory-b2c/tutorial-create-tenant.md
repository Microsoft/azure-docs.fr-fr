---
title: Tutoriel – Créer un locataire Azure Active Directory B2C
description: Ce tutoriel explique comment préparer l’inscription de vos applications en créant un locataire Azure Active Directory B2C à l’aide du portail Azure.
services: B2C
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: tutorial
ms.date: 12/03/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 5229175af618f897caeae7310338cbe51966a48d
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96859588"
---
# <a name="tutorial-create-an-azure-active-directory-b2c-tenant"></a>Tutoriel : Créer un locataire Azure Active Directory B2C

Pour que vos applications puissent interagir avec Azure Active Directory B2C (Azure AD B2C), elles doivent être inscrites dans un locataire que vous gérez. 

> [!NOTE]
> Vous pouvez créer jusqu’à 20 locataires par abonnement. Cette limite aide à se prémunir contre les menaces visant vos ressources, telles que les attaques par déni de service, et est appliquée à la fois dans le portail Azure et dans l’API de création de locataire sous-jacente. Si vous avez besoin de créer plus de 20 locataires, contactez le [Support Microsoft](support-options.md).

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Créer un client Azure AD B2C
> * Lier votre locataire à votre abonnement
> * Passer à l’annuaire contenant votre locataire Azure AD B2C
> * Ajouter la ressource Azure AD B2C en tant que **favori** dans le portail Azure

Vous découvrirez comment inscrire une application dans le didacticiel suivant.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="create-an-azure-ad-b2c-tenant"></a>Créer un client Azure AD B2C

1. Connectez-vous au [portail Azure](https://portal.azure.com/). Connectez-vous à l’aide d’un compte Azure auquel au moins le rôle de [contributeur](../role-based-access-control/built-in-roles.md) de l’abonnement ou d’un groupe de ressources de l’abonnement a été attribué.

1. Sélectionnez le répertoire qui contient votre abonnement.

    Dans la barre d’outils du portail Azure, sélectionnez l’icône **Annuaire et abonnement**, puis sélectionnez le répertoire qui contient votre abonnement. Cet annuaire est différent de celui qui contiendra votre locataire Azure AD B2C.

    ![Abonné, filtre Annuaire et abonnement avec locataire d’abonnement sélectionné](media/tutorial-create-tenant/portal-01-pick-directory.png)

1. Dans le menu du Portail Azure ou dans la page **Accueil**, sélectionnez **Créer une ressource**.

   ![Sélectionner le bouton Créer une ressource](media/tutorial-create-tenant/create-a-resource.png)

1. Recherchez **Azure Active Directory B2C**, puis sélectionnez **Créer**.
2. Cliquez sur **Créer un nouveau client Azure Active Directory B2C**.

    ![Créer un locataire Azure AD B2C sélectionné dans le portail Azure](media/tutorial-create-tenant/portal-02-create-tenant.png)

1. Dans la page **Créer un annuaire**, entrez les informations suivantes :

   - **Nom de l’organisation** : entrez un nom pour votre locataire Azure AD B2C.
   - **Nom de domaine initial** : entrez un nom de domaine pour votre locataire Azure AD B2C.
   - **Pays ou région** : sélectionnez votre pays ou votre région dans la liste. Il ne sera pas possible de changer cette sélection ultérieurement.
   - **Abonnement** : sélectionnez votre abonnement dans la liste.
   - **Groupe de ressources** : sélectionnez ou recherchez le groupe de ressources qui contiendra le locataire.

    ![Formulaire de création de locataire avec des exemples de valeurs dans le portail Azure](media/tutorial-create-tenant/review-and-create-tenant.png)

1. Sélectionnez **Revoir + créer**.
1. Passez en revue les paramètres de votre annuaire. Sélectionnez ensuite **Créer**.

Vous pouvez lier plusieurs locataires Azure AD B2C à un seul abonnement Azure pour la facturation. Pour lier un locataire, vous devez être administrateur du locataire Azure AD B2C et vous devez disposer d’au moins le rôle Contributeur dans l’abonnement Azure. Consultez [Lier un locataire Azure AD B2C à un abonnement](billing.md#link-an-azure-ad-b2c-tenant-to-a-subscription).

## <a name="select-your-b2c-tenant-directory"></a>Sélectionner l’annuaire de votre locataire B2C

Pour commencer à utiliser votre nouveau locataire Azure AD B2C, vous devez passer à l’annuaire qui contient le locataire.

Sélectionnez le filtre **Annuaire et abonnement** dans le menu supérieur du portail Azure, puis l’annuaire qui contient votre locataire Azure AD B2C.

Si vous ne voyez pas dans un premier temps votre nouveau locataire Azure B2C dans la liste, actualisez la fenêtre du navigateur, puis sélectionnez à nouveau le filtre **Annuaire + abonnement** dans le menu du haut.

![Annuaire contenant le locataire B2C sélectionné dans le portail Azure](media/tutorial-create-tenant/portal-07-select-tenant-directory.png)

## <a name="add-azure-ad-b2c-as-a-favorite-optional"></a>Ajouter Azure AD B2C en tant que favori (facultatif)

Cette étape facultative vous permet de sélectionner plus facilement votre locataire Azure AD B2C dans les tutoriels suivants et ultérieurs.

Au lieu de rechercher *Azure AD B2C* dans **Tous les services** chaque fois que vous souhaitez utiliser votre locataire, vous pouvez à la place marquer la ressource comme favori. Vous pouvez ensuite la sélectionner dans la section **Favoris** du menu du portail pour accéder rapidement à votre locataire Azure AD B2C.

Vous ne devez effectuer cette opération qu’une seule fois. Avant d’effectuer ces étapes, assurez-vous d’avoir basculé vers l’annuaire contenant votre locataire Azure AD B2C comme décrit dans la section précédente, [Sélectionner l’annuaire de votre locataire B2C](#select-your-b2c-tenant-directory).

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans le menu du portail Azure, sélectionnez **Tous les services**.
1. Dans la zone de recherche **Tous les services**, recherchez **Azure AD B2C**, pointez sur le résultat de la recherche, puis sélectionnez l’icône en étoile dans l’info-bulle. **Azure AD B2C** apparaît maintenant dans le portail Azure sous **Favoris**.
1. Si vous souhaitez modifier la position de votre nouveau favori, accédez au menu Portail Azure, sélectionnez **Azure AD B2C**, puis faites-le glisser vers le haut ou vers le haut jusqu’à la position souhaitée.

    ![Azure AD B2C, menu Favoris, portail Microsoft Azure](media/tutorial-create-tenant/portal-08-b2c-favorite.png)

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer un client Azure AD B2C
> * Lier votre locataire à votre abonnement
> * Passer à l’annuaire contenant votre locataire Azure AD B2C
> * Ajouter la ressource Azure AD B2C en tant que **favori** dans le portail Azure

Ensuite, découvrez comment inscrire une application web dans votre nouveau locataire.

> [!div class="nextstepaction"]
> [Inscrire vos applications >](tutorial-register-applications.md)