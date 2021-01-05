---
title: Ajouter Google comme fournisseur d’identité pour B2B – Azure AD
description: Fédérer avec Google pour permettre à des utilisateurs invités de se connecter à vos applications Azure AD avec leur propre compte Gmail.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 05/11/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: ff8912794169cf61f394a097248a8476b2e0c0f3
ms.sourcegitcommit: dd45ae4fc54f8267cda2ddf4a92ccd123464d411
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92926233"
---
# <a name="add-google-as-an-identity-provider-for-b2b-guest-users"></a>Ajouter Google comme fournisseur d’identité pour les utilisateurs invités B2B

En configurant la fédération avec Google, vous pouvez autoriser les utilisateurs invités à se connecter à vos applications et ressources partagées avec leur propre compte Gmail, sans avoir besoin de créer un compte Microsoft. 

> [!NOTE]
> La fédération Google est conçue spécialement pour les utilisateurs Gmail. Pour fédérer avec les domaines G Suite, utilisez la [fédération directe](direct-federation.md).

## <a name="what-is-the-experience-for-the-google-user"></a>Quelle est l’expérience de l’utilisateur Google ?
Lorsque vous envoyez une invitation à des utilisateurs de Google Gmail, les utilisateurs invités doivent accéder à vos applications ou ressources partagées à l’aide d’un lien qui inclut le contexte locataire. Son expérience varie selon qu’il est ou non déjà connecté à Google :
  - Les utilisateurs invités qui ne sont pas connectés à Google seront invités à le faire.
  - Les utilisateurs invités déjà connectés à Google seront invités à choisir le compte qu’ils souhaitent utiliser. Il doit choisir le compte que vous avez utilisé pour l’inviter.

Les utilisateurs invités qui voient une erreur « header too long » (en-tête trop long) peuvent effacer leurs cookies ou ouvrir une fenêtre privée ou incognito, puis essayer de se reconnecter.

![Capture d’écran montrant la page de connexion Google.](media/google-federation/google-sign-in.png)

## <a name="limitations"></a>Limites

Teams prend entièrement en charge les utilisateurs invités Google sur tous les appareils. Les utilisateurs de Google peuvent se connecter à Teams à partir d’un point de terminaison commun, comme `https://teams.microsoft.com`.

Les points de terminaison communs d’autres applications ne prennent pas forcément en charge les utilisateurs Google. Les utilisateurs invités Google doivent se connecter à l’aide d’un lien comportant les informations de votre locataire. Voici quelques exemples :
  * `https://myapps.microsoft.com/?tenantid=<your tenant ID>`
  * `https://portal.azure.com/<your tenant ID>`
  * `https://myapps.microsoft.com/<your verified domain>.onmicrosoft.com`

   S’ils essaient d’utiliser un lien comme `https://myapps.microsoft.com` ou `https://portal.azure.com`, les utilisateurs invités Google obtiendront une erreur.

Vous pouvez également fournir aux utilisateurs invités Google un lien direct vers une application ou une ressource, à condition qu’il comprenne les informations de votre locataire. Par exemple, `https://myapps.microsoft.com/signin/Twitter/<application ID?tenantId=<your tenant ID>`. 

## <a name="step-1-configure-a-google-developer-project"></a>Étape 1 : Configurer un projet de développeur Google
Commencez par créer un projet dans la console des développeurs Google pour obtenir un ID client et une clé secrète client que vous pourrez ajouter plus tard à Azure Active Directory (Azure AD). 
1. Accédez aux API Google à l’adresse https://console.developers.google.com et connectez-vous avec votre compte Google. Nous vous recommandons d’utiliser un compte Google d’équipe partagé.
2. Acceptez les conditions d’utilisation du service si vous y êtes invité.
3. Créer un projet : Dans le tableau de bord, sélectionnez **Créer un projet** , donnez un nom au projet (par exemple **Azure AD B2B** ), puis sélectionnez **Créer**  : 
   
   ![Capture d’écran montrant une page Nouveau projet.](media/google-federation/google-new-project.png)

4. Dans la page **API et Services** , sélectionnez **Afficher** sous votre nouveau projet.

5. Sélectionnez **Accéder à l’aperçu des API** sur la carte des API. Sélectionnez **Écran d’autorisation OAuth**.

6. Sélectionnez **Externe** , puis sélectionnez **Créer**. 

7. Dans l’ **écran de consentement OAuth** , entrez un **nom d’application**  :

   ![Capture d’écran montrant l’écran de consentement OAuth de Google.](media/google-federation/google-oauth-consent-screen.png)

8. Accédez à la section **Domaines autorisés** et entrez **microsoftonline.com**  :

   ![Capture d’écran montrant la section Domaines autorisés.](media/google-federation/google-oauth-authorized-domains.PNG)

9. Sélectionnez **Enregistrer**.

10. Sélectionnez **Credentials (Informations d’identification)**. Dans le menu **Créer les informations d’identification** , sélectionnez **ID client OAuth**  :

    ![Capture d’écran montrant le menu Créer les informations d’identification des API Google.](media/google-federation/google-api-credentials.png)

11. Sous **Type d’application** , sélectionnez **Application web**. Donnez à l’application un nom approprié, par exemple **Azure AD B2B**. Sous **URI de redirection autorisés** , entrez les URI suivants :
    - `https://login.microsoftonline.com`
    - `https://login.microsoftonline.com/te/<tenant ID>/oauth2/authresp` <br>(où `<tenant ID>` est votre ID de locataire)
   
    > [!NOTE]
    > Pour trouver votre ID de locataire, accédez au [portail Azure](https://portal.azure.com). Sous **Azure Active Directory** , sélectionnez **Propriétés** et copiez l’ **ID de locataire**.

    ![Capture d’écran montrant la section URI de redirection autorisée.](media/google-federation/google-create-oauth-client-id.png)

12. Sélectionnez **Créer**. Copiez l’ID client et la clé secrète client. Vous les utiliserez lorsque vous ajouterez le fournisseur d’identité dans le portail Azure.

    ![Capture d’écran montrant l’ID client et la clé secrète client OAuth.](media/google-federation/google-auth-client-id-secret.png)

## <a name="step-2-configure-google-federation-in-azure-ad"></a>Étape 2 : Configurer la fédération de Google dans Azure AD 
Maintenant vous définirez l’ID client Google et la clé secrète client. Pour cela, vous pouvez utiliser le portail Azure ou PowerShell. Pensez à tester votre configuration de fédération de Google en invitant vous-même. Utilisez une adresse Gmail et essayez de donner suite à l’invitation avec votre compte Google invité. 

**Pour configurer la fédération de Google dans le portail Azure** 
1. Accédez au [portail Azure](https://portal.azure.com). Sélectionnez **Azure Active Directory** dans le volet de gauche. 
2. Sélectionnez **Identités externes**.
3. Sélectionnez **Tous les fournisseurs d’identité** , puis cliquez sur le bouton **Google**.
4. Entrez l’ID client et la clé secrète client obtenus précédemment. Sélectionnez **Enregistrer**  : 

   ![Capture d’écran montrant la page d’ajout de fournisseur d’identité Google.](media/google-federation/google-identity-provider.png)

**Pour configurer la fédération de Google à l’aide de PowerShell**
1. Installez la dernière version d’Azure AD PowerShell pour le module Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)).
2. Exécutez cette commande : `Connect-AzureAD`.
3. À l’invite de connexion, connectez-vous avec le compte d’administrateur général géré.  
4. Exécutez la commande suivante : 
   
   `New-AzureADMSIdentityProvider -Type Google -Name Google -ClientId <client ID> -ClientSecret <client secret>`
 
   > [!NOTE]
   > Utilisez l’ID client et la clé secrète client à partir de l’application créée à l’« étape 1 : Configurer un projet de développeur Google ». Pour plus d’informations, consultez [New-AzureADMSIdentityProvider](/powershell/module/azuread/new-azureadmsidentityprovider?view=azureadps-2.0-preview). 
 
## <a name="how-do-i-remove-google-federation"></a>Comment supprimer la fédération de Google ?
Vous pouvez supprimer votre configuration de fédération de Google. Si vous le faites, les utilisateurs invités Google qui ont déjà utilisé leur invitation ne pourront plus se connecter. Mais vous pouvez leur donner accès à vos ressources à nouveau en les supprimant du répertoire et en les réinvitant. 
 
**Pour supprimer la fédération de Google dans le portail Azure AD**
1. Accédez au [portail Azure](https://portal.azure.com). Sélectionnez **Azure Active Directory** dans le volet de gauche. 
2. Sélectionnez **Identités externes**.
3. Sélectionnez **Tous les fournisseurs d’identité**.
4. Dans la ligne **Google** , sélectionnez le bouton représentant des points de suspension ( **...** ), puis choisissez **Supprimer**. 
   
   ![Capture d’écran montrant le bouton de suppression du fournisseur d’identité sociale.](media/google-federation/google-social-identity-providers.png)

1. Sélectionnez **Oui** pour confirmer la suppression. 

**Pour supprimer la fédération de Google à l’aide de PowerShell** 
1. Installez la dernière version d’Azure AD PowerShell pour le module Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)).
2. Exécutez `Connect-AzureAD`.  
4. Dans l’invite de connexion, connectez-vous avec le compte d’administrateur général géré.  
5. Entrez la commande suivante :

    `Remove-AzureADMSIdentityProvider -Id Google-OAUTH`

   > [!NOTE]
   > Pour plus d’informations, consultez [Remove-AzureADMSIdentityProvider](/powershell/module/azuread/Remove-AzureADMSIdentityProvider?view=azureadps-2.0-preview).
