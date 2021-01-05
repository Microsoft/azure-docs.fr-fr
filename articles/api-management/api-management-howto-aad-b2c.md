---
title: Autoriser des comptes de développeurs avec Azure Active Directory B2C
titleSuffix: Azure API Management
description: Découvrez comment autoriser des utilisateurs avec Azure Active Directory B2C dans Gestion des API.
services: api-management
documentationcenter: API Management
author: miaojiang
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/04/2019
ms.author: apimpm
ms.openlocfilehash: 7b586edd7adce8bcea61419005a3ce8cfc814fb3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013540"
---
# <a name="how-to-authorize-developer-accounts-by-using-azure-active-directory-b2c-in-azure-api-management"></a>Comment autoriser des comptes de développeurs avec Azure Active Directory B2C dans Gestion des API Azure

## <a name="overview"></a>Vue d’ensemble

Azure Active Directory B2C est une solution de gestion des identités de cloud pour applications web et mobiles grand public. Vous pouvez l’utiliser pour gérer l’accès au portail des développeurs. Ce guide explique comment configurer votre service Gestion des API requis pour l’intégration avec Azure Active Directory B2C. Pour plus d’informations sur l’activation de l’accès au portail des développeurs à l’aide d’Active Directory Azure classique, consultez [Comment autoriser des comptes de développeurs avec Azure Active Directory].

> [!NOTE]
> Pour effectuer les étapes de ce guide, vous devez disposer d’un client Azure Active Directory B2C dans lequel vous souhaitez créer une application. Vous devez également disposer de stratégies d’inscription et de connexion. Pour plus d’informations, consultez la [Vue d’ensemble d’Azure Active Directory B2C].

[!INCLUDE [premium-dev-standard.md](../../includes/api-management-availability-premium-dev-standard.md)]

## <a name="authorize-developer-accounts-by-using-azure-active-directory-b2c"></a>Autoriser des comptes de développeurs avec Azure Active Directory B2C

1. Pour commencer, connectez-vous au [portail Azure](https://portal.azure.com) et recherchez votre instance Gestion des API.

   > [!NOTE]
   > Si vous n’avez pas encore créé d’instance de service Gestion des API, consultez la page [Création d’une instance de service Gestion des API][Create an API Management service instance] dans le didacticiel [Prise en main de Gestion des API Azure][Get started with Azure API Management].

1. Sous **Identités**. Cliquez sur **+Ajouter** dans la partie supérieure.

   Le volet **Ajouter le fournisseur d’identité** s’affiche à droite. Choisissez **Azure Active Directory B2C**.
    
   ![Ajouter AAD B2C en tant que fournisseur d’identité][api-management-howto-add-b2c-identity-provider]

1. Copiez l’ **URL de redirection**.

   ![URL de redirection du fournisseur d’identité AAD B2C][api-management-howto-copy-b2c-identity-provider-redirect-url]

1. Dans un nouvel onglet, accédez à votre locataire Azure Active Directory B2C dans le portail Azure et ouvrez le panneau **Applications**.

   ![Inscrire une nouvelle application 1][api-management-howto-aad-b2c-portal-menu]

1. Cliquez sur le bouton **Ajouter** pour créer une nouvelle application Azure Active Directory B2C.

   ![Inscrire une nouvelle application 2][api-management-howto-aad-b2c-add-button]

1. Dans le panneau **Nouvelle application**, entrez un nom pour l’application. Choisissez **Oui** sous **Application web/API web**, puis choisissez **Oui** sous **Autoriser un flux implicite**. Puis collez **l’URL de redirection** copiée à l’étape 3 dans la zone de texte **URL de réponse**.

   ![Inscrire une nouvelle application 3][api-management-howto-aad-b2c-app-details]

1. Si vous utilisez le nouveau portail des développeurs (et non le portail des développeurs hérité), incluez le **prénom**, le **nom** et l’**ID d’objet de l’utilisateur** dans les revendications d’application.

    ![Revendications d’application](./media/api-management-howto-aad-b2c/api-management-application-claims.png)

1. Cliquez sur le bouton **Créer**. Lorsque l’application est créée, elle apparaît dans le panneau **Applications**. Cliquez sur le nom de l’application pour afficher les informations la concernant.

   ![Inscrire une nouvelle application 4][api-management-howto-aad-b2c-app-created]

1. À partir du panneau **Propriétés**, copiez **l’ID de l’application** dans le Presse-papiers.

   ![ID d’application 1][api-management-howto-aad-b2c-app-id]

1. Revenez au volet **Ajouter un fournisseur d’identité** dans Gestion des API et collez l’ID dans la zone de texte **ID client**.
    
1.  Revenez dans l’inscription d’application B2C, cliquez sur le bouton **Clés**, puis sur **Générer une clé**. Cliquez sur **Enregistrer** pour enregistrer la configuration et afficher la **clé d’application**. Copiez la clé dans le Presse-papiers.

    ![Clé d’application 1][api-management-howto-aad-b2c-app-key]

1.  Revenez au volet **Ajouter un fournisseur d’identité** dans Gestion des API et collez la clé dans la zone de texte **Secret client**.
    
1.  Spécifiez le nom de domaine du client Azure Active Directory B2C dans **Locataire de connexion**.

1.  Le champ **Autorité** vous permet de contrôler l’URL de connexion à Azure AD B2C à utiliser. Définissez la valeur sur **< votre nom_de_locataire_b2c>.b2clogin.com**.

1. Spécifiez la **stratégie d’inscription** et la **stratégie de connexion** à partir des stratégies du locataire B2C. Si vous le souhaitez, vous pouvez également fournir la **Stratégie de modification du profil** et la **Stratégie de réinitialisation du mot de passe**.

1. Après avoir spécifié la configuration souhaitée, cliquez sur **Enregistrer**.

    Après avoir enregistré les modifications, les développeurs pourront créer de nouveaux comptes et se connecter au portail des développeurs à l’aide d’Azure Active Directory B2C.

## <a name="developer-portal---add-azure-ad-b2c-account-authentication"></a>Portail des développeurs : ajouter l’authentification de compte Azure AD B2C

Dans le portail des développeurs, il est possible de se connecter avec AAD B2C à l’aide du widget **Bouton de connexion : OAuth**. Le widget est déjà inclus dans la page de connexion du contenu du portail des développeurs par défaut.

Bien qu’un compte soit automatiquement créé chaque fois qu’un utilisateur se connecte avec AAD B2C, vous pouvez envisager d’ajouter le même widget à la page d’inscription.

Le widget **Formulaire d’inscription : OAuth** représente un formulaire utilisé pour s’inscrire à l’aide d’OAuth.

> [!IMPORTANT]
> Vous devez [republier le portail](api-management-howto-developer-portal-customize.md#publish) pour que les modifications apportées à AAD prennent effet.

## <a name="legacy-developer-portal---how-to-sign-up-with-azure-ad-b2c"></a>Portail des développeurs hérité : comment s’inscrire avec Azure AD B2C

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

1. Pour inscrire un compte de développeur avec Azure Active Directory B2C, ouvrez une nouvelle fenêtre de navigateur et accédez au portail des développeurs. Cliquez sur le bouton **S’inscrire**.

   ![Portail des développeurs 1][api-management-howto-aad-b2c-dev-portal]

2. Choisissez de vous inscrire avec **Azure Active Directory B2C**.

   ![Portail des développeurs 2][api-management-howto-aad-b2c-dev-portal-b2c-button]

3. Vous êtes redirigé vers la stratégie d’inscription que vous avez configurée dans la section précédente. Choisissez de vous inscrire avec votre adresse de messagerie ou l’un de vos comptes sociaux existants.

   > [!NOTE]
   > Si Azure Active Directory B2C est la seule option activée dans l’onglet **Identités** sur le portail des éditeurs, vous êtes redirigé directement vers la stratégie d’inscription.

   ![Portail des développeurs][api-management-howto-aad-b2c-dev-portal-b2c-options]

   Une fois l’inscription terminée, vous êtes redirigé vers le portail des développeurs. Vous êtes maintenant inscrit sur le portail des développeurs pour votre instance de service Gestion des API.

    ![Référencement terminé][api-management-registration-complete]

## <a name="next-steps"></a>Étapes suivantes

*  [Vue d’ensemble d’Azure Active Directory B2C]
*  [Azure Active Directory B2C : Structure de stratégie extensible]
*  [Utiliser un compte Microsoft comme fournisseur d’identité dans Azure Active Directory B2C]
*  [Utiliser un compte Google comme fournisseur d’identité dans Azure Active Directory B2C]
*  [Utiliser un compte LinkedIn comme fournisseur d’identité dans Azure Active Directory B2C]
*  [Utiliser un compte Facebook comme fournisseur d’identité dans Azure Active Directory B2C]



[api-management-howto-add-b2c-identity-provider]: ./media/api-management-howto-aad-b2c/api-management-add-b2c-identity-provider.PNG
[api-management-howto-copy-b2c-identity-provider-redirect-url]: ./media/api-management-howto-aad-b2c/api-management-b2c-identity-provider-redirect-url.PNG
[api-management-howto-aad-b2c-portal-menu]: ./media/api-management-howto-aad-b2c/api-management-b2c-portal-menu.PNG
[api-management-howto-aad-b2c-add-button]: ./media/api-management-howto-aad-b2c/api-management-b2c-add-button.PNG
[api-management-howto-aad-b2c-app-details]: ./media/api-management-howto-aad-b2c/api-management-b2c-app-details.PNG
[api-management-howto-aad-b2c-app-created]: ./media/api-management-howto-aad-b2c/api-management-b2c-app-created.PNG
[api-management-howto-aad-b2c-app-id]: ./media/api-management-howto-aad-b2c/api-management-b2c-app-id.PNG
[api-management-howto-aad-b2c-client-id]: ./media/api-management-howto-aad-b2c/api-management-b2c-client-id.PNG
[api-management-howto-aad-b2c-app-key]: ./media/api-management-howto-aad-b2c/api-management-b2c-app-key.PNG
[api-management-howto-aad-b2c-app-key-saved]: ./media/api-management-howto-aad-b2c/api-management-b2c-app-key-saved.PNG
[api-management-howto-aad-b2c-client-secret]: ./media/api-management-howto-aad-b2c/api-management-b2c-client-secret.PNG
[api-management-howto-aad-b2c-allowed-tenant]: ./media/api-management-howto-aad-b2c/api-management-b2c-allowed-tenant.PNG
[api-management-howto-aad-b2c-policies]: ./media/api-management-howto-aad-b2c/api-management-b2c-policies.PNG
[api-management-howto-aad-b2c-dev-portal]: ./media/api-management-howto-aad-b2c/api-management-b2c-dev-portal.PNG
[api-management-howto-aad-b2c-dev-portal-b2c-button]: ./media/api-management-howto-aad-b2c/api-management-b2c-dev-portal-b2c-button.PNG
[api-management-howto-aad-b2c-dev-portal-b2c-options]: ./media/api-management-howto-aad-b2c/api-management-b2c-dev-portal-b2c-options.PNG
[api-management-complete-registration]: ./media/api-management-howto-aad/api-management-complete-registration.PNG
[api-management-registration-complete]: ./media/api-management-howto-aad/api-management-registration-complete.png

[api-management-security-external-identities]: ./media/api-management-howto-aad/api-management-b2c-security-tab.png
[api-management-security-aad-new]: ./media/api-management-howto-aad/api-management-security-aad-new.png
[api-management-new-aad-application-menu]: ./media/api-management-howto-aad/api-management-new-aad-application-menu.png
[api-management-new-aad-application-1]: ./media/api-management-howto-aad/api-management-new-aad-application-1.png
[api-management-new-aad-application-2]: ./media/api-management-howto-aad/api-management-new-aad-application-2.png
[api-management-new-aad-app-created]: ./media/api-management-howto-aad/api-management-new-aad-app-created.png
[api-management-aad-app-permissions]: ./media/api-management-howto-aad/api-management-aad-app-permissions.png
[api-management-aad-app-client-id]: ./media/api-management-howto-aad/api-management-aad-app-client-id.png
[api-management-client-id]: ./media/api-management-howto-aad/api-management-client-id.png
[api-management-aad-key-before-save]: ./media/api-management-howto-aad/api-management-aad-key-before-save.png
[api-management-aad-key-after-save]: ./media/api-management-howto-aad/api-management-aad-key-after-save.png
[api-management-client-secret]: ./media/api-management-howto-aad/api-management-client-secret.png
[api-management-client-allowed-tenants]: ./media/api-management-howto-aad/api-management-client-allowed-tenants.png
[api-management-client-allowed-tenants-save]: ./media/api-management-howto-aad/api-management-client-allowed-tenants-save.png
[api-management-aad-delegated-permissions]: ./media/api-management-howto-aad/api-management-aad-delegated-permissions.png
[api-management-dev-portal-signin]: ./media/api-management-howto-aad/api-management-dev-portal-signin.png
[api-management-aad-signin]: ./media/api-management-howto-aad/api-management-aad-signin.png
[api-management-aad-app-multi-tenant]: ./media/api-management-howto-aad/api-management-aad-app-multi-tenant.png
[api-management-aad-reply-url]: ./media/api-management-howto-aad/api-management-aad-reply-url.png
[api-management-permissions-form]: ./media/api-management-howto-aad/api-management-permissions-form.png
[api-management-configure-product]: ./media/api-management-howto-aad/api-management-configure-product.png
[api-management-add-groups]: ./media/api-management-howto-aad/api-management-add-groups.png
[api-management-select-group]: ./media/api-management-howto-aad/api-management-select-group.png
[api-management-aad-groups-list]: ./media/api-management-howto-aad/api-management-aad-groups-list.png
[api-management-aad-group-added]: ./media/api-management-howto-aad/api-management-aad-group-added.png
[api-management-groups]: ./media/api-management-howto-aad/api-management-groups.png
[api-management-edit-group]: ./media/api-management-howto-aad/api-management-edit-group.png

[How to add operations to an API]: ./mock-api-responses.md
[How to add and publish a product]: api-management-howto-add-products.md
[Monitoring and analytics]: api-management-monitoring.md
[Add APIs to a product]: api-management-howto-add-products.md#add-apis
[Publish a product]: api-management-howto-add-products.md#publish-product
[Get started with Azure API Management]: get-started-create-service-instance.md
[API Management policy reference]: ./api-management-policies.md
[Caching policies]: ./api-management-policies.md#caching-policies
[Create an API Management service instance]: get-started-create-service-instance.md

[https://oauth.net/2/]: https://oauth.net/2/
[WebApp-GraphAPI-DotNet]: https://github.com/AzureADSamples/WebApp-GraphAPI-DotNet
[Vue d’ensemble d’Azure Active Directory B2C]: ../active-directory-b2c/overview.md
[Comment autoriser des comptes de développeurs avec Azure Active Directory]: ./api-management-howto-aad.md
[Azure Active Directory B2C : Structure de stratégie extensible]: ../active-directory-b2c/user-flow-overview.md
[Utiliser un compte Microsoft comme fournisseur d’identité dans Azure Active Directory B2C]: ../active-directory-b2c/identity-provider-microsoft-account.md
[Utiliser un compte Google comme fournisseur d’identité dans Azure Active Directory B2C]: ../active-directory-b2c/identity-provider-google.md
[Utiliser un compte Facebook comme fournisseur d’identité dans Azure Active Directory B2C]: ../active-directory-b2c/identity-provider-facebook.md
[Utiliser un compte LinkedIn comme fournisseur d’identité dans Azure Active Directory B2C]: ../active-directory-b2c/identity-provider-linkedin.md

[Prerequisites]: #prerequisites
[Configure an OAuth 2.0 authorization server in API Management]: #step1
[Configure an API to use OAuth 2.0 user authorization]: #step2
[Test the OAuth 2.0 user authorization in the Developer Portal]: #step3
[Next steps]: #next-steps

[Log in to the Developer portal using an Azure Active Directory account]: #Log-in-to-the-Developer-portal-using-an-Azure-Active-Directory-account
