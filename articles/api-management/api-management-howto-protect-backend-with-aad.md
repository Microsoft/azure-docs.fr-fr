---
title: Protéger un back-end d'API dans Gestion des API à l'aide d'OAuth 2.0 et d'Azure AD
titleSuffix: Azure API Management
description: Apprenez à sécuriser l'accès à un back-end d'API web dans Gestion des API Azure avec l'autorisation utilisateur OAuth 2.0 et Azure Active Directory
services: api-management
author: miaojiang
ms.service: api-management
ms.topic: article
ms.date: 09/23/2020
ms.author: apimpm
ms.custom: contperf-fy21q1
ms.openlocfilehash: face4beab450e92be76b2bb90e45625e025de6ee
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97027915"
---
# <a name="protect-a-web-api-backend-in-azure-api-management-by-using-oauth-20-authorization-with-azure-ad"></a>Protéger un back-end d'API web dans Gestion des API Azure en utilisant l'autorisation OAuth 2.0 avec Azure AD 

Ce guide explique comment configurer une instance de [Gestion des API Azure](api-management-key-concepts.md) pour protéger une API en utilisant le [protocole OAuth 2.0 avec Azure Active Directory (Azure AD)](../active-directory/develop/active-directory-v2-protocols.md). 

> [!NOTE]
> Cette fonctionnalité est disponible dans les niveaux **Développeur**, **De base** et **Standard** et **Premium** de gestion des API.

## <a name="prerequisites"></a>Prérequis

Pour suivre les étapes décrites dans cet article, vous devez avoir :

- Une instance Gestion des API
- Une API publiée qui utilise l’instance Gestion des API
- un locataire Azure AD ;

## <a name="overview"></a>Vue d’ensemble

Voici un petit aperçu des étapes :

1. Inscrire une application (backend-app) dans Azure AD pour représenter l’API.
1. Inscrire une autre application (client-app) dans Azure AD pour représenter une application cliente qui doit appeler l’API.
1. Dans Azure AD, accorder des autorisations pour permettre à client-app d’appeler backend-app.
1. Configurez la console de développeur pour appeler l’API à l’aide de l’autorisation utilisateur OAuth 2.0.
1. Ajouter une stratégie **validate-jwt** afin de valider le jeton OAuth pour chaque requête entrante.

## <a name="register-an-application-in-azure-ad-to-represent-the-api"></a>Inscrire une application dans Azure AD pour représenter l’API

Pour protéger une API avec Azure AD, enregistrez d’abord dans Azure AD une application qui représente l’API. Dans les étapes suivantes, le portail Azure est utilisé pour inscrire l'application. Pour plus d'informations sur l'inscription d'une application, consultez [Démarrage rapide : Configurer une application pour exposer une API web](../active-directory/develop/quickstart-configure-app-expose-web-apis.md).

1. Accédez au [Portail Azure](https://portal.azure.com) pour inscrire votre application. Recherchez et sélectionnez **Inscriptions d'applications**.

1. Sélectionnez **Nouvelle inscription**. 

1. Lorsque la page **Inscrire une application** s’affiche, saisissez les informations d’inscription de votre application :

   - Dans la section **Nom**, saisissez un nom d’application cohérent qui s’affichera pour les utilisateurs de l’application, par exemple *backend-app*. 
   - Dans la section **Types de comptes pris en charge**, sélectionnez une option adaptée à votre scénario. 

1. Laissez la section **URI de redirection** vide pour l’instant.

1. Sélectionnez **Inscrire** pour créer l’application. 

1. Dans la page **Vue d’ensemble** de l’application, recherchez la valeur **ID d’application (client)** et notez-la.

1. Sélectionnez **Exposer une API** et définissez **l’URI d’ID d’application** sur la valeur par défaut. Enregistrez cette valeur pour une utilisation ultérieure.

1. Sélectionnez le bouton **Ajouter une étendue** pour afficher la page **Ajouter une étendue**. Créez ensuite une nouvelle étendue qui est prise en charge par l’API (par exemple, `Files.Read`).

1. Sélectionnez le bouton **Ajouter une étendue** pour créer l’étendue. Répétez cette étape pour ajouter toutes les étendues prises en charge par votre API.

1. Lorsque les étendues sont créées, prenez-en note afin de les utiliser à une étape ultérieure. 

## <a name="register-another-application-in-azure-ad-to-represent-a-client-application"></a>Inscrire une autre application dans Azure AD pour représenter une application cliente

Chaque application cliente qui appelle l’API doit être inscrite en tant qu’application dans Azure AD. Dans cet exemple, l’application cliente est la **console de développeur** dans le portail des développeurs de gestion des API. 

Pour inscrire une autre application dans Azure AD afin de représenter la console de développeur :

1. Accédez au [Portail Azure](https://portal.azure.com) pour inscrire votre application.

1. Recherchez et sélectionnez **Inscriptions d'applications**.

1. Sélectionnez **Nouvelle inscription**.

1. Lorsque la page **Inscrire une application** s’affiche, saisissez les informations d’inscription de votre application :

   - Dans la section **Nom**, saisissez un nom d’application cohérent qui s’affichera pour les utilisateurs de l’application, par exemple *client-app*. 
   - Dans la section **Types de comptes pris en charge**, sélectionnez **Comptes dans un répertoire d’organisation (tout répertoire Azure AD – Multilocataire)** . 

1. Dans la section **URI de redirection**, sélectionnez `Web` et laissez le champ URL vide pour le moment.

1. Sélectionnez **Inscrire** pour créer l’application. 

1. Dans la page **Vue d’ensemble** de l’application, recherchez la valeur **ID d’application (client)** et notez-la.

1. Créez un secret client pour cette application afin de l’utiliser à une étape ultérieure.

   1. Dans la liste des pages de votre application cliente, sélectionnez **Certificats et secrets**, puis sélectionnez **Nouveau secret client**.

   1. Sous **Ajouter un secret client**, fournissez une **Description**. Déterminez quand la clé doit expirer et sélectionnez **Ajouter**.

Lorsque le secret est créé, notez la valeur de clé afin de l’utiliser à une étape ultérieure. 

## <a name="grant-permissions-in-azure-ad"></a>Accorder des autorisations dans Azure AD

Maintenant que vous avez inscrit deux applications pour représenter l’API et la console de développeur, accordez des autorisations pour permettre à client-app d’appeler backend-app.  

1. Accédez au [Portail Azure](https://portal.azure.com) pour accorder des autorisations à votre application cliente. Recherchez et sélectionnez **Inscriptions d'applications**.

1. Choisissez votre application cliente. Ensuite, dans la liste des pages de l’application, sélectionnez **Autorisations de l’API**.

1. Sélectionnez **Ajouter une autorisation**.

1. Sous **Sélectionnez une API**, sélectionnez **Mes API**, puis recherchez et sélectionnez votre backend-app.

1. Sous **Permissions déléguées**, sélectionnez les permissions appropriées pour votre backend-app, puis sélectionnez **Ajouter des permissions**.

1. Si vous le souhaitez, dans la page **Autorisations de l’API**, sélectionnez **Accorder le consentement administrateur pour \<your-tenant-name>** pour octroyer le consentement au nom de tous les utilisateurs de ce répertoire. 

## <a name="enable-oauth-20-user-authorization-in-the-developer-console"></a>Activer l’autorisation utilisateur OAuth 2.0 dans la console de développeur

À ce stade, vous avez créé vos applications dans Azure AD et vous avez accordé les autorisations appropriées pour que client-app puisse appeler backend-app. 

Dans cet exemple, la console de développeur est l’application cliente. Les étapes suivantes décrivent comment activer l’autorisation utilisateur OAuth 2.0 dans la console de développeur. 

1. Sur le Portail Azure, accédez à votre instance Gestion des API.

1. Sélectionnez **OAuth 2.0** > **Ajouter**.

1. Spécifiez un **Nom d’affichage** et une **Description**.

1. Pour **l’URL de la page d’inscription client**, entrez une valeur d’espace réservé telle que `http://localhost`. L’**URL de la page d’enregistrement client** pointe vers la page que les utilisateurs peuvent utiliser pour créer et configurer leurs propres comptes pour les fournisseurs OAuth 2.0 qui prennent en charge cette fonctionnalité. Dans cet exemple, les utilisateurs ne peuvent pas créer et configurer leurs propres comptes. Un espace réservé est donc utilisé à la place.

1. Sélectionnez **Code d’autorisation** comme **Types d’octroi d’autorisation**.

1. Spécifiez l’**URL de point de terminaison d’autorisation** et l’**URL de point de terminaison de jeton**. Récupérez ces valeurs à partir de la page **Points de terminaison** dans votre locataire Azure AD. Accédez à nouveau à la page **Inscriptions des applications** puis sélectionnez **Points de terminaison**.


1. Copiez la valeur de **Point de terminaison d’autorisation OAuth 2.0** et collez-la dans la zone de texte **URL de point de terminaison d’autorisation**. Sélectionnez **PUBLICATION** sous Méthode de demande d’autorisation.

1. Copiez la valeur de **Point de terminaison de jeton OAuth 2.0** et collez-la dans la zone de texte **URL de point de terminaison de jeton**. 

   >[!IMPORTANT]
   > Utilisez des points de terminaison **v1** ou **v2**. Toutefois, selon la version que vous choisissez, l’étape ci-dessous sera différente. Nous vous recommandons d’utiliser des points de terminaison v2. 

1. Si vous utilisez des points de terminaison **v1**, ajoutez un paramètre de corps nommé **Ressource**. Comme valeur pour ce paramètre, utilisez l’**ID d’application** de l’application back-end. 

1. Si vous utilisez des points de terminaison **v2**, utilisez l’étendue que vous avez créée pour l’application back-end dans le champ **Étendue par défaut**. En outre, veillez à définir la valeur de la propriété [`accessTokenAcceptedVersion`](../active-directory/develop/reference-app-manifest.md#accesstokenacceptedversion-attribute) sur `2` dans le [manifeste de votre application](../active-directory/develop/reference-app-manifest.md).

1. Ensuite, spécifiez les informations d’identification du client. Il s’agit des informations d’identification de client-app.

1. Pour **ID client**, utilisez l’**ID d’application** de client-app.

1. Pour **Secret client**, utilisez la clé que vous avez créée pour client-app. 

1. Juste après le secret client figure la valeur **redirect_url** pour le type d’octroi du code d’autorisation. Notez cette URL.

1. Sélectionnez **Create** (Créer).

1. Revenez à l’inscription de votre application cliente dans Azure Active Directory et sélectionnez **Authentification**.

1. Sous **Configurations de plateforme** cliquez sur **Ajouter une plateforme**, sélectionnez le type **Web**, collez le **redirect_url** sous **URI de redirection**, puis cliquez sur le bouton **Configurer** pour enregistrer.

Maintenant que vous avez configuré un serveur d’autorisation OAuth 2.0, la console de développeur peut obtenir des jetons d’accès à partir d’Azure AD. 

L’étape suivante consiste à activer l’autorisation utilisateur OAuth 2.0 pour votre API. Cela permet à la console de développeur de savoir qu’elle doit obtenir un jeton d’accès pour l’utilisateur avant d’effectuer des appels vers votre API.

1. Accédez à votre instance Gestion des API,puis accédez à **API**.

1. Sélectionnez l’API que vous souhaitez protéger. Par exemple : `Echo API`.

1. Accédez à **Settings**.

1. Sous **Sécurité**, choisissez **OAuth 2.0** et sélectionnez le serveur OAuth 2.0 que vous avez configuré précédemment. 

1. Sélectionnez **Enregistrer**.

## <a name="successfully-call-the-api-from-the-developer-portal"></a>Appel de l’API à partir du portail des développeurs

> [!NOTE]
> Cette section ne s’applique pas au niveau **consommation**, qui ne prend pas en charge le portail des développeurs.

Maintenant que l’autorisation utilisateur OAuth 2.0 est activée sur votre API, la console de développeur obtient un jeton d’accès pour le compte de l’utilisateur avant d’appeler l’API.

1. Accédez à l’opération de votre choix sous l’API dans le portail des développeurs puis sélectionnez **Essayer**. Cela vous amène à la console de développeur.

1. Observez le nouvel élément dans la section **Autorisation** correspondant au serveur d’autorisation que vous venez d’ajouter.

1. Sélectionnez **Code d’autorisation** dans la liste déroulante d’autorisations. Vous êtes alors invité à vous connecter au locataire Azure AD. Si vous êtes déjà connecté avec ce compte, il est possible qu’aucune invite ne s’affiche.

1. Une fois la connexion établie, un en-tête `Authorization` est ajouté à la requête avec un jeton d’accès d’Azure AD. Voici un exemple de jeton (encodage Base64) :

   ```
   Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IlNTUWRoSTFjS3ZoUUVEU0p4RTJnR1lzNDBRMCIsImtpZCI6IlNTUWRoSTFjS3ZoUUVEU0p4RTJnR1lzNDBRMCJ9.eyJhdWQiOiIxYzg2ZWVmNC1jMjZkLTRiNGUtODEzNy0wYjBiZTEyM2NhMGMiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC80NDc4ODkyMC05Yjk3LTRmOGItODIwYS0yMTFiMTMzZDk1MzgvIiwiaWF0IjoxNTIxMTUyNjMzLCJuYmYiOjE1MjExNTI2MzMsImV4cCI6MTUyMTE1NjUzMywiYWNyIjoiMSIsImFpbyI6IkFWUUFxLzhHQUFBQUptVzkzTFd6dVArcGF4ZzJPeGE1cGp2V1NXV1ZSVnd1ZXZ5QU5yMlNkc0tkQmFWNnNjcHZsbUpmT1dDOThscUJJMDhXdlB6cDdlenpJdzJLai9MdWdXWWdydHhkM1lmaDlYSGpXeFVaWk9JPSIsImFtciI6WyJyc2EiXSwiYXBwaWQiOiJhYTY5ODM1OC0yMWEzLTRhYTQtYjI3OC1mMzI2NTMzMDUzZTkiLCJhcHBpZGFjciI6IjEiLCJlbWFpbCI6Im1pamlhbmdAbWljcm9zb2Z0LmNvbSIsImZhbWlseV9uYW1lIjoiSmlhbmciLCJnaXZlbl9uYW1lIjoiTWlhbyIsImlkcCI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0Ny8iLCJpcGFkZHIiOiIxMzEuMTA3LjE3NC4xNDAiLCJuYW1lIjoiTWlhbyBKaWFuZyIsIm9pZCI6IjhiMTU4ZDEwLWVmZGItNDUxMS1iOTQzLTczOWZkYjMxNzAyZSIsInNjcCI6InVzZXJfaW1wZXJzb25hdGlvbiIsInN1YiI6IkFGaWtvWFk1TEV1LTNkbk1pa3Z3MUJzQUx4SGIybV9IaVJjaHVfSEM1aGciLCJ0aWQiOiI0NDc4ODkyMC05Yjk3LTRmOGItODIwYS0yMTFiMTMzZDk1MzgiLCJ1bmlxdWVfbmFtZSI6Im1pamlhbmdAbWljcm9zb2Z0LmNvbSIsInV0aSI6ImFQaTJxOVZ6ODBXdHNsYjRBMzBCQUEiLCJ2ZXIiOiIxLjAifQ.agGfaegYRnGj6DM_-N_eYulnQdXHhrsus45QDuApirETDR2P2aMRxRioOCR2YVwn8pmpQ1LoAhddcYMWisrw_qhaQr0AYsDPWRtJ6x0hDk5teUgbix3gazb7F-TVcC1gXpc9y7j77Ujxcq9z0r5lF65Y9bpNSefn9Te6GZYG7BgKEixqC4W6LqjtcjuOuW-ouy6LSSox71Fj4Ni3zkGfxX1T_jiOvQTd6BBltSrShDm0bTMefoyX8oqfMEA2ziKjwvBFrOjO0uK4rJLgLYH4qvkR0bdF9etdstqKMo5gecarWHNzWi_tghQu9aE3Z3EZdYNI_ZGM-Bbe3pkCfvEOyA
   ```

1. Sélectionnez **Send** pour appeler l’API avec succès.

## <a name="configure-a-jwt-validation-policy-to-pre-authorize-requests"></a>Configuration d’une stratégie de validation JWT pour autoriser des demandes

À ce stade, lorsqu’un utilisateur tente d’effectuer un appel à partir de la console de développeur, l’utilisateur est invité à se connecter. La console de développeur obtient un jeton d’accès pour le compte de l’utilisateur et comprend le jeton dans la requête adressée à l’API.

Toutefois, que se passe-t-il si quelqu’un appelle votre API sans jeton ou avec un jeton non valide ? Par exemple, essayez d’appeler l’API sans l'en-tête `Authorization`. L’appel marchera quand même. Cela est dû au fait que Gestion des API ne valide pas le jeton d’accès à ce stade. Elle transfert simplement l’en-tête `Authorization` à l’API principale.

Utilisez la stratégie [Validate JWT](./api-management-access-restriction-policies.md#ValidateJWT) pour pré-autoriser les requêtes dans Gestion des API en validant les jetons d’accès de chaque requête entrante. Si une requête n’a pas de jeton valide, Gestion des API la bloque. Par exemple, vous pouvez ajouter la stratégie ci-dessous à la section de stratégie `<inbound>` de `Echo API`. Il vérifie la revendication d’audience dans un jeton d’accès et retourne un message d’erreur si le jeton n’est pas valide. Pour plus d’informations sur la façon de configurer des stratégies, consultez [Définir ou modifier des stratégies](./set-edit-policies.md).


```xml
<validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
    <openid-config url="https://login.microsoftonline.com/{aad-tenant}/.well-known/openid-configuration" />
    <required-claims>
        <claim name="aud">
            <value>{Application ID of backend-app}</value>
        </claim>
    </required-claims>
</validate-jwt>
```

> [!NOTE]
> Cette URL `openid-config` correspond au point de terminaison v1. Pour le `openid-config`point de terminaison v2, utilisez l’URL suivante :
>
> `https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration`.

## <a name="build-an-application-to-call-the-api"></a>Générer une application pour appeler l’API

Dans ce guide, vous avez utilisé la console de développeur dans Gestion des API comme exemple d’application cliente pour appeler le `Echo API` protégé par OAuth 2.0. Pour en savoir plus sur la façon de générer une application et d’implémenter OAuth 2.0, consultez [exemples de code Azure Active Directory](../active-directory/develop/sample-v2-code.md).

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [Azure Active Directory et OAuth2.0](../active-directory/develop/authentication-vs-authorization.md).
- Découvrez plus de [vidéos](https://azure.microsoft.com/documentation/videos/index/?services=api-management) sur Gestion des API.
- Pour les autres méthodes permettant de sécuriser votre service principal, consultez [Authentification mutuelle des certificats](./api-management-howto-mutual-certificates.md).
- [Création d’une instance du service Gestion des API](./get-started-create-service-instance.md).
- [Gérer votre première API](./import-and-publish.md).
