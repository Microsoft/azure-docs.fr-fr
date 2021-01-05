---
title: Ajouter Facebook en tant que fournisseur d’identité – Azure AD
description: Configurez une fédération avec Facebook pour permettre à des utilisateurs externes (invités) de se connecter à vos applications Azure AD avec leurs propres comptes Facebook.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 05/19/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 547b455dc776b7304e413b3b6f1330e7cedcf2a2
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92441995"
---
# <a name="add-facebook-as-an-identity-provider-for-external-identities"></a>Ajouter Facebook en tant que fournisseur d’identité pour les identités externes

Vous pouvez ajouter Facebook à vos flux utilisateur d’inscription en libre-service (préversion) afin que les utilisateurs puissent se connecter à vos applications en utilisant leurs propres comptes Facebook. Pour permettre aux utilisateurs de se connecter à l’aide de Facebook, vous commencer par [activer l’inscription en libre-service](self-service-sign-up-user-flow.md) pour votre locataire. Après avoir ajouté Facebook en tant que fournisseur d’identité, configurez un flux utilisateur pour l’application, puis sélectionnez Facebook comme l’une des options de connexion.

> [!NOTE]
> Les utilisateurs peuvent uniquement utiliser leurs comptes Facebook pour s’inscrire via des applications à l’aide de l’inscription libre-service et des flux d’utilisateurs. Les utilisateurs ne peuvent pas être invités à accepter leur invitation à l’aide d’un compte Facebook.

## <a name="create-an-app-in-the-facebook-developers-console"></a>Créer une application dans la console des développeurs Facebook

Pour utiliser un compte Facebook en tant que [fournisseur d’identité](identity-providers.md), vous devez créer une application dans la console des développeurs Facebook. Si vous n’avez pas encore de compte Facebook, vous pouvez en créer un sur [https://www.facebook.com/](https://www.facebook.com).

> [!NOTE]  
> Utilisez les URL suivantes aux étapes 9 et 16 ci-dessous.
> - Dans **URL du site** , entrez l’adresse de votre application, par exemple `https://contoso.com`.
> - Pour **URI de redirection OAuth valides** , entrez `https://login.microsoftonline.com/te/<tenant-id>/oauth2/authresp`. Vous trouverez votre ID `<tenant-ID>` dans le panneau Vue d’ensemble d’Azure Active Directory.


1. Connectez-vous à [Facebook pour les développeurs](https://developers.facebook.com/) avec les informations d’identification de votre compte Facebook.
2. Si ce n’est pas déjà fait, vous devez vous inscrire en tant que développeur Facebook. Pour cela, sélectionnez **Prise en main** dans le coin supérieur droit de la page, acceptez les politiques de Facebook et suivez les étapes d’inscription.
3. Sélectionnez **Mes applications** , puis **Créer une application** .
4. Entrez un **nom d’affichage** et une **adresse e-mail de contact** valide.
5. Sélectionnez **Créer un ID d'application** . Vous devrez peut-être accepter les politiques de la plateforme Facebook et effectuer une vérification de sécurité en ligne.
6. Sélectionnez **Paramètres** > **Base** .
7. Choisissez une **catégorie** , par exemple, Entreprise et pages. Cette valeur est requise par Facebook, mais n’est pas utilisée pour Azure AD.
8. Au bas de la page, sélectionnez **Ajouter une plateforme** , puis sélectionnez **Site web** .
9. Dans **URL du site** , entrez l’URL appropriée (indiquée ci-dessus).
10. Dans **URL de la politique de confidentialité** , entrez l’URL de la page dans laquelle vous gérez les informations de confidentialité de votre application, par exemple `http://www.contoso.com`.
11. Sélectionnez **Enregistrer les modifications** .
12. En haut de la page, copiez la valeur de l’ **ID de l’application** .
13. Sélectionnez **Afficher** , puis copiez la valeur **Clé secrète de l’application** . Vous avez besoin de ces deux valeurs pour configurer Facebook en tant que fournisseur d’identité dans votre client. **App Secret** est une information d’identification de sécurité importante.
14. Cliquez sur le signe plus en regard de la zone **PRODUITS** , puis sélectionnez **Configurer** sous **Connexion Facebook** .
15. Sous **Connexion Facebook** , sélectionnez **Paramètres** .
16. Dans **URI de redirection OAuth valides** , entrez l’URL appropriée (indiquée ci-dessus).
17. Sélectionnez **Enregistrer les modifications** en bas de la page.
18. Pour rendre votre application Facebook disponible sur Azure AD, sélectionnez le sélecteur État dans la partie supérieure droite de la page et **activez-le** pour rendre l’application publique, puis sélectionnez **Changer de mode** . À ce stade, l’état doit passer de **Développement** à **Production** .
    
## <a name="configure-a-facebook-account-as-an-identity-provider"></a>Configuration d’un compte Facebook en tant que fournisseur d’identité
Désormais, vous définirez l’ID et la clé secrète client Facebook, soit en les entrant dans le portail Azure AD, soit en utilisant PowerShell. Vous pouvez tester votre configuration Facebook en vous inscrivant via un flux d’utilisateurs sur une application prenant en charge l’inscription en libre-service.

### <a name="to-configure-facebook-federation-in-the-azure-ad-portal"></a>Pour configurer la Fédération Facebook dans le portail Azure AD
1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général de votre locataire Azure AD.
2. Sous **Services Azure** , sélectionnez **Azure Active Directory** .
3. Dans le menu de gauche, sélectionnez **Identités externes** .
4. Sélectionnez **Tous les fournisseurs d’identité** , puis **Facebook** .
5. Dans **ID client** , entrez l’ **ID** de l’application Facebook que vous avez créée précédemment.
6. Dans **Clé secrète client** , entrez la **Clé secrète d’application** que vous avez consignée.

   ![Capture d’écran montrant la page Sélectionner un fournisseur d’identité sociale](media/facebook-federation/add-social-identity-provider-page.png)

7. Sélectionnez **Enregistrer** .
### <a name="to-configure-facebook-federation-by-using-powershell"></a>Pour configurer la Fédération Google à l’aide de PowerShell
1. Installez la dernière version d’Azure AD PowerShell pour le module Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)).
2. Exécutez la commande suivante : `Connect-AzureAD`.
3. À l’invite de connexion, connectez-vous avec le compte d’administrateur général géré.  
4. Exécutez la commande suivante : 
   
   `New-AzureADMSIdentityProvider -Type Facebook -Name Facebook -ClientId [Client ID] -ClientSecret [Client secret]`
 
   > [!NOTE]
   > Utilisez l’ID et la clé secrète client à partir de l’application que vous avez créée précédemment dans la console de développeur Facebook. Pour plus d’informations, consultez l’article [New-AzureADMSIdentityProvider](/powershell/module/azuread/new-azureadmsidentityprovider?view=azureadps-2.0-preview). 

## <a name="how-do-i-remove-facebook-federation"></a>Comment supprimer la Fédération Facebook ?
Vous pouvez supprimer votre configuration de Fédération de Google. Dans ce cas, tous les utilisateurs qui se sont inscrits par le biais de flux d’utilisateurs avec leurs comptes Facebook ne pourront plus se connecter. 

### <a name="to-delete-facebook-federation-in-the-azure-ad-portal"></a>Pour supprimer la Fédération Facebook dans le portail Azure AD : 
1. Accédez au [portail Azure](https://portal.azure.com). Sélectionnez **Azure Active Directory** dans le volet de gauche. 
2. Sélectionnez **Identités externes** .
3. Sélectionnez **Tous les fournisseurs d’identité** .
4. Dans la ligne **Google** , sélectionnez le menu contextuel ( **...** ), puis **Supprimer** . 
5. Sélectionnez **Oui** pour confirmer la suppression.

### <a name="to-delete-facebook-federation-by-using-powershell"></a>Pour supprimer la Fédération Facebook à l’aide de PowerShell : 
1. Installez la dernière version d’Azure AD PowerShell pour le module Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)).
2. Exécutez `Connect-AzureAD`.  
4. Dans l’invite de connexion, connectez-vous avec le compte d’administrateur général géré.  
5. Entrez la commande suivante :

    `Remove-AzureADMSIdentityProvider -Id Facebook-OAUTH`

   > [!NOTE]
   > Pour plus d’informations, consultez [Remove-AzureADMSIdentityProvider](/powershell/module/azuread/Remove-AzureADMSIdentityProvider?view=azureadps-2.0-preview). 

## <a name="next-steps"></a>Étapes suivantes

- [Ajouter l’inscription en libre-service à une application](self-service-sign-up-user-flow.md)