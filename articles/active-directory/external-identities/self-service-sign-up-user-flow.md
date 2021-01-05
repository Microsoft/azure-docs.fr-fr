---
title: Ajouter un flux utilisateur d’inscription en libre-service – Azure AD
description: Créez des flux d’utilisateurs pour les applications générées par votre organisation. Ensuite, les utilisateurs qui visitent cette application peuvent obtenir un compte invité à l’aide des options configurées dans le flux utilisateur.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 06/16/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: f76f4a3e5fc87420c242c693e3c48a91244641e0
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97560031"
---
# <a name="add-a-self-service-sign-up-user-flow-to-an-app-preview"></a>Ajouter un flux utilisateur d’inscription en libre-service à une application (préversion)
> [!NOTE]
> L’inscription en libre-service est une fonctionnalité en préversion publique d’Azure Active Directory. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Vous pouvez créer des flux d’utilisateurs pour les applications générées par votre organisation. L’association de votre flux utilisateur avec une application vous permet d’activer l’inscription sur cette application. Vous pouvez choisir plusieurs applications à associer au flux utilisateur. Une fois que vous avez associé le flux d’utilisateurs à une ou plusieurs applications, les utilisateurs qui visitent cette application pourront s'inscrire et obtenir un compte invité à l’aide des options configurées dans le flux d’utilisateurs.

> [!NOTE]
> Vous pouvez associer des flux d’utilisateurs à des applications générées par votre organisation. Les flux d’utilisateurs ne peuvent pas être utilisés pour les applications Microsoft, telles que SharePoint ou Teams.

## <a name="before-you-begin"></a>Avant de commencer

### <a name="add-social-identity-providers-optional"></a>Ajouter des fournisseurs d’identité sociale (facultatif)

Azure AD est le fournisseur d’identité par défaut pour l’inscription en libre-service. Cela signifie que les utilisateurs peuvent s’inscrire par défaut avec un compte Azure AD. Des fournisseurs d’identité sociale peuvent également être inclus dans ces flux d’inscription pour prendre en charge les comptes Google et Facebook.

- [Ajouter Facebook à votre liste de fournisseurs d’identité sociale](facebook-federation.md)
- [Ajouter Google à votre liste de fournisseurs d’identité sociale](google-federation.md)

> [!NOTE]
> Dans la préversion actuelle, si un flux d’utilisateurs d’inscription en libre-service est associé à une application et que vous envoyez à l’utilisateur une invitation à cette application, l’utilisateur ne pourra pas utiliser un compte Gmail pour accepter l’invitation. En guise de solution de contournement, l’utilisateur peut passer par le processus d’inscription en libre-service. Ou, il peut accepter l’invitation en accédant à une autre application ou à l’aide de leur portail Mes applications sur https://myapps.microsoft.com.

### <a name="define-custom-attributes-optional"></a>Définir des attributs personnalisés (facultatif)

Les attributs utilisateur sont des valeurs collectées auprès de l’utilisateur lors de son inscription en libre-service. Azure AD est fourni avec un ensemble intégré d’attributs, mais vous pouvez créer des attributs personnalisés à utiliser dans votre flux utilisateur. Vous pouvez également lire et écrire ces attributs à l’aide de l’API Microsoft Graph. Consultez [Définir des attributs personnalisés pour des flux utilisateur](user-flow-add-custom-attributes.md)

## <a name="enable-self-service-sign-up-for-your-tenant"></a>Activer l’inscription en libre-service pour votre locataire

Avant de pouvoir ajouter un flux utilisateur d’inscription en libre-service à vos applications, vous devez activer la fonctionnalité pour votre locataire. Une fois celle-ci activée, des contrôles deviennent disponibles dans le flux utilisateur, qui vous permettent d’associer le flux utilisateur à une application.

1. Connectez-vous au [Portail Azure](https://portal.azure.com) en tant qu’administrateur Azure AD.
2. Sous **Services Azure**, sélectionnez **Azure Active Directory**.
3. Sélectionnez **Paramètres utilisateur**, puis, sous **Utilisateurs externes**, sélectionnez **Gérer les paramètres de collaboration externes**.
4. Positionnez le bouton bascule **Activer l’inscription en libre-service d’invité via des flux utilisateur (préversion)** sur **Oui**.

   ![Activer l’inscription en libre-service d’invité](media/self-service-sign-up-user-flow/enable-self-service-sign-up.png)
5. Sélectionnez **Enregistrer**.
## <a name="create-the-user-flow-for-self-service-sign-up"></a>Créer le flux utilisateur pour l’inscription en libre-service

Ensuite, vous allez créer le flux utilisateur pour l’inscription en libre-service et l’ajouter à une application.

1. Connectez-vous au [Portail Azure](https://portal.azure.com) en tant qu’administrateur Azure AD.
2. Sous **Services Azure**, sélectionnez **Azure Active Directory**.
3. Dans le menu de gauche, sélectionnez **Identités externes**.
4. Sélectionnez **Flux utilisateur (préversion)** , puis **Nouveau flux utilisateur**.

   ![Bouton Ajouter un nouveau flux utilisateur](media/self-service-sign-up-user-flow/new-user-flow.png)

5. Dans la page **Créer**, entrez un **Nom** pour le flux utilisateur. Notez que le nom est automatiquement préfixé avec **B2X_1_** .
6. Dans la liste **Fournisseurs d’identité**, sélectionnez un ou plusieurs fournisseurs d’identité que vos utilisateurs externes peuvent utiliser pour se connecter à votre application. L’option **Inscription à Azure Active Directory** est activée par défaut (pour savoir comment ajouter des fournisseurs d’identité, consultez [Avant de commencer](#before-you-begin) plus haut dans cet article).
7. Sous **Attributs utilisateur**, choisissez les attributs à collecter auprès de l’utilisateur. Pour Attributs supplémentaires, sélectionnez **Afficher plus**. Par exemple, sélectionnez **Afficher plus**, puis choisissez des attributs et des revendications pour **Pays/région**, **Nom d’affichage** et **Code postal**. Sélectionnez **OK**.

   ![Page Créer un nouveau flux utilisateur](media/self-service-sign-up-user-flow/create-user-flow.png)

8. Sélectionnez **Create** (Créer).
9. Le nouveau flux utilisateur s’affiche dans la liste **Flux utilisateur (préversion)** . Si nécessaire, actualisez la page.

## <a name="select-the-layout-of-the-attribute-collection-form"></a>Sélectionner la mise en page du formulaire de collection d’attributs

Vous pouvez choisir l’ordre dans lequel les attributs s’affichent sur la page d’inscription. 

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory**.
2. Sélectionnez **Identités externes**, **Flux utilisateur (préversion)** .
3. Sélectionnez le flux utilisateur d’inscription en libre-service dans la liste.
4. Sous **Personnaliser**, sélectionnez **Mises en page**.
5. Les attributs à collecter que vous avec choisis sont répertoriés. Pour modifier l’ordre d’affichage, sélectionnez un attribut, puis choisissez **Monter**, **Descendre**, **Déplacer vers le haut** ou **Déplacer vers le bas**.
6. Sélectionnez **Enregistrer**.

## <a name="add-applications-to-the-self-service-sign-up-user-flow"></a>Ajouter des applications au flux utilisateur d’inscription en libre-service

Vous pouvez maintenant associer des applications au flux utilisateur.

1. Connectez-vous au [Portail Azure](https://portal.azure.com) en tant qu’administrateur Azure AD.
2. Sous **Services Azure**, sélectionnez **Azure Active Directory**.
3. Dans le menu de gauche, sélectionnez **Identités externes**.
4. Sous **Inscription en libre-service**, sélectionnez **Flux utilisateurs (préversion)** .
5. Sélectionnez le flux utilisateur d’inscription en libre-service dans la liste.
6. Dans le menu de gauche, sous **Utiliser**, sélectionnez **Applications**.
7. Sélectionnez **Ajouter une application**.

   ![Affecter une application au flux utilisateur](media/self-service-sign-up-user-flow/assign-app-to-user-flow.png)

8. Sélectionnez l’application dans la liste. Vous pouvez aussi utiliser la zone de recherche pour trouver l’application, puis la sélectionner.
9. Cliquez sur **Sélectionner**.

## <a name="next-steps"></a>Étapes suivantes

- [Ajouter Google à votre liste de fournisseurs d’identité sociale](google-federation.md)
- [Ajouter Facebook à votre liste de fournisseurs d’identité sociale](facebook-federation.md)
- [Utiliser des connecteurs d’API pour personnaliser et étendre vos flux utilisateur via des API web](api-connectors-overview.md)
- [Ajouter un flux de travail d’approbation personnalisé à votre flux utilisateur](self-service-sign-up-add-approvals.md)
