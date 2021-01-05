---
title: Autoriser des comptes de développeurs à l’aide d’OAuth 2.0 dans Gestion des API
titleSuffix: Azure API Management
description: Apprenez à autoriser les utilisateurs à l'aide d'OAuth 2.0 dans Gestion des API. OAuth 2.0 sécurise l’API afin que les utilisateurs puissent accéder uniquement aux ressources pour lesquelles ils disposent d’autorisations.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 08/14/2020
ms.author: apimpm
ms.openlocfilehash: fae4e349d46425c0c2b2b923d6a61e2e588708c1
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93077249"
---
# <a name="how-to-authorize-developer-accounts-using-oauth-20-in-azure-api-management"></a>Comment autoriser des comptes de développeurs à l'aide de OAuth 2.0 dans Gestion des API Azure

De nombreuses API prennent en charge [OAuth 2.0](https://oauth.net/2/) pour sécuriser les API et assurer que seuls les utilisateurs valides y ont accès et peuvent accéder uniquement aux ressources pour lesquelles ils y sont autorisés. Pour utiliser la console de développement interactive de Gestion des API Azure avec ces API, le service vous permet de configurer votre instance de service pour travailler avec votre API compatible OAuth 2.0.

## <a name="prerequisites"></a><a name="prerequisites"> </a>Prérequis

Ce guide explique comment configurer votre instance de service Gestion des API pour utiliser l'autorisation OAuth 2.0 dans des comptes de développeur, mais n'explique pas comment configurer un fournisseur OAuth 2.0. La configuration de chaque fournisseur OAuth 2.0 est différente, bien que la procédure soit similaire et que les informations requises pour configurer OAuth 2.0 dans votre instance de service Gestion des API soient identiques. Cette rubrique inclut des exemples d'utilisation de Azure Active Directory en tant que fournisseur OAuth 2.0.

> [!NOTE]
> Pour plus d’informations sur la configuration de OAuth 2.0 à l’aide d’Azure Active Directory, consultez l’exemple [WebApp-GraphAPI-DotNet][WebApp-GraphAPI-DotNet] .

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="configure-an-oauth-20-authorization-server-in-api-management"></a><a name="step1"> </a>Configurer un serveur d’autorisation OAuth 2.0 dans Gestion des API

> [!NOTE]
> Si vous n’avez pas encore créé d’instance de service Gestion des API, consultez la page [Création d’une instance de service Gestion des API][Create an API Management service instance].

1. Cliquez sur l’onglet OAuth 2.0 dans le menu de gauche, puis cliquez sur **+Ajouter**.

    ![Menu OAuth 2.0](./media/api-management-howto-oauth2/oauth-01.png)

2. Entrez un nom et une description facultative dans les champs **Nom** et **Description**.

    > [!NOTE]
    > Ces champs sont utilisés pour identifier le serveur d'autorisation OAuth 2.0 dans l'instance de service Gestion des API actuelle, et leurs valeurs ne proviennent pas du serveur OAuth 2.0.

3. Entrez l' **URL de la page d'enregistrement client**. Cette page est l'endroit où les utilisateurs créent et gèrent leurs comptes ; elle varie en fonction du fournisseur OAuth 2.0 utilisé. L’ **URL de la page d’enregistrement client** pointe vers la page que les utilisateurs peuvent utiliser pour créer et configurer leurs propres comptes pour les fournisseurs OAuth 2.0 qui prennent en charge la gestion des comptes utilisateur (par exemple, `https://contoso.com/login`). Certaines organisations ne configurent pas cette fonctionnalité ou ne l’utilisent pas, même si le fournisseur OAuth 2.0 la prend en charge. Si la gestion des utilisateurs de comptes de votre fournisseur OAuth 2.0 n’a pas été configurée, entrez un espace réservé d’URL ici, par exemple l’URL de votre société ou une URL comme `https://placeholder.contoso.com`.

    ![Nouveau serveur OAuth 2.0](./media/api-management-howto-oauth2/oauth-02.png)

4. La section suivante du formulaire inclut les paramètres **Types d’accès accordé aux autorisations** , **URL de point de terminaison d’autorisation** et **Méthode de demande d’autorisation**.

    Spécifiez le paramètre **Types d’accès accordé aux autorisations** en sélectionnant les types souhaités. **code d'autorisation** est spécifié par défaut.

    Entrez l' **URL de point de terminaison d'autorisation**. Pour Azure Active Directory, cette URL est similaire à l’URL suivante, où `<tenant_id>` est remplacé par l’ID de votre locataire Azure AD.

    `https://login.microsoftonline.com/<tenant_id>/oauth2/authorize`

    La **Méthode de demande d’autorisation** spécifie comment la demande d'autorisation est envoyée au serveur OAuth 2.0. Par défaut, la méthode **GET** est sélectionnée.

5. Vous devez ensuite spécifier les paramètres **URL de point de terminaison de jeton** , **Méthodes d’authentification du client** , **Méthode d’envoi des jetons d’accès** et **Étendue par défaut**.

    ![Capture d’écran montrant l’écran Ajouter un service OAuth2.](./media/api-management-howto-oauth2/oauth-03.png)

    Pour un serveur OAuth 2.0 Azure Active Directory, l’ **URL de point de terminaison de jeton** a le format suivant, où `<TenantID>` a le format `yourapp.onmicrosoft.com`.

    `https://login.microsoftonline.com/<TenantID>/oauth2/token`

    Le paramètre par défaut pour **Méthodes d’authentification du client** est **De base** , et le paramètre par défaut pour **Méthode d’envoi des jetons d’accès** est **En-tête d’autorisation**. Ces valeurs sont configurées dans cette section du formulaire, ainsi que le paramètre **Étendue par défaut**.

6. La section **Informations d’identification du client** inclut les paramètres **ID client** et **Clé secrète client** , qui sont obtenus lors du processus de création et de configuration de votre serveur OAuth 2.0. Une fois que les paramètres **ID client** et **Clé secrète client** ont été spécifiés, le **redirect_uri** pour le **code d’autorisation** est généré. Cette URI est utilisée pour configurer l'URL de réponse dans la configuration de votre serveur OAuth 2.0.

    Dans le nouveau portail des développeurs, le suffixe URI se présente sous la forme suivante :

    - `/signin-oauth/code/callback/{authServerName}` pour le flux d’octroi du code d’autorisation
    - `/signin-oauth/implicit/callback` pour le flux d’octroi implicite

    ![Capture d’écran montrant où ajouter les informations d’identification du client pour le nouveau service OAuth2.](./media/api-management-howto-oauth2/oauth-04.png)

    Si le paramètre **Types d’accès accordés aux autorisations** est défini sur **Mot de passe du propriétaire des ressources** , la section **Informations d’identification du mot de passe du propriétaire des ressources** permet de spécifier ces informations d’identification ; sinon, vous pouvez la laisser vide.

    Une fois le formulaire complété, cliquez sur **Créer** pour enregistrer la configuration du serveur d’autorisation OAuth 2.0 de Gestion des API. Après l'enregistrement de la configuration du serveur, vous pouvez configurer les API pour utiliser cette configuration, tel qu'expliqué dans la section suivante.

## <a name="configure-an-api-to-use-oauth-20-user-authorization"></a><a name="step2"> </a>Configurer une API pour utiliser l’autorisation utilisateur OAuth 2.0

1. Cliquez sur **API** dans le menu **Gestion des API** à gauche.

    ![API OAuth 2.0](./media/api-management-howto-oauth2/oauth-05.png)

2. Cliquez sur le nom de l’API souhaitée, puis sur **Paramètres**. Faites défiler l’écran jusqu’à la section **Sécurité** , puis cochez la case **OAuth 2.0**.

    ![Paramètres OAuth 2.0](./media/api-management-howto-oauth2/oauth-06.png)

3. Sélectionnez le **Serveur d’autorisation** souhaité dans la liste déroulante, puis cliquez sur **Enregistrer**.

    ![Capture d’écran mettant en évidence le serveur d’autorisation sélectionné et le bouton Enregistrer.](./media/api-management-howto-oauth2/oauth-07.png)

## <a name="legacy-developer-portal---test-the-oauth-20-user-authorization"></a><a name="step3"> </a>Portail de développement hérité - test de l’autorisation utilisateur OAuth 2.0

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

Une fois que vous avez configuré votre serveur d'autorisation OAuth 2.0 et votre API pour utiliser ce serveur, vous pouvez le tester en accédant au portail de développement et en appelant une API. Cliquez sur **Portail des développeurs (hérité)** dans le menu supérieur de la page **Vue d’ensemble** de votre instance Gestion des API Azure.

Cliquez sur **API** dans le menu supérieur et sélectionnez **API Echo**.

![API Echo][api-management-apis-echo-api]

> [!NOTE]
> Si vous n'avez qu'une API configurée ou visible dans votre compte, cliquez sur des API pour accéder directement aux opérations associées.

Sélectionnez l’opération **Ressource GET** , cliquez sur **Ouvrir la console** , puis sélectionnez **Code d’autorisation** dans la liste déroulante.

![Open console][api-management-open-console]

Lorsque **Code d'autorisation** est sélectionné, une fenêtre contextuelle s'affiche, avec le formulaire d'inscription du fournisseur OAuth 2.0. Dans cet exemple, le formulaire d'inscription est fourni par Azure Active Directory.

> [!NOTE]
> Si l'utilisation des fenêtres contextuelles est désactivée, vous êtes invité à l'activer par le navigateur. Après l'activation, sélectionnez à nouveau **Code d'autorisation** , et le formulaire d'inscription s'affiche.

![Se connecter][api-management-oauth2-signin]

Une fois que vous êtes connecté, les **En-têtes de demandes`Authorization : Bearer` sont renseignés avec un en-tête** qui autorise la demande.

![Jeton d'en-tête de demande][api-management-request-header-token]

À présent, vous pouvez configurer les valeurs souhaitées pour les paramètres restants, puis soumettre la demande.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’utilisation d’OAuth 2.0 et la gestion des API, regardez la vidéo suivante et l’article qui l’ [accompagne](api-management-howto-protect-backend-with-aad.md).

[api-management-oauth2-signin]: ./media/api-management-howto-oauth2/api-management-oauth2-signin.png
[api-management-request-header-token]: ./media/api-management-howto-oauth2/api-management-request-header-token.png
[api-management-open-console]: ./media/api-management-howto-oauth2/api-management-open-console.png
[api-management-apis-echo-api]: ./media/api-management-howto-oauth2/api-management-apis-echo-api.png

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

[Prerequisites]: #prerequisites
[Configure an OAuth 2.0 authorization server in API Management]: #step1
[Configure an API to use OAuth 2.0 user authorization]: #step2
[Test the OAuth 2.0 user authorization in the Developer Portal]: #step3
[Next steps]: #next-steps
