---
title: Inscription d’une application API web protégée | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment créer une API web protégée et les informations dont vous avez besoin pour inscrire l’application.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/15/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 7a38e2384c5f24bc3a72e1ef8e8f7119b2db0f2f
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94443940"
---
# <a name="protected-web-api-app-registration"></a>API web protégée : Inscription d'application

Cet article explique les spécificités de l’inscription d’application pour une API web protégée.

Pour connaître les étapes courantes visant à inscrire une application, consultez [Démarrage rapide : Inscrire une application avec la plateforme des identités Microsoft](quickstart-register-app.md).

## <a name="accepted-token-version"></a>Version de jeton acceptée

Le point de terminaison de la plateforme d’identités Microsoft peut émettre des jetons v1.0 et des jetons v2.0. Pour plus d’informations sur ces jetons, consultez [Jetons d’accès](access-tokens.md).

La version de jeton que votre API peut accepter dépend de la sélection de **Types de compte pris en charge** lorsque vous créez l’inscription de votre application API Web dans le Portail Azure.

- Si la valeur de **Types de comptes pris en charge** est **Comptes dans un annuaire organisationnel et comptes personnels Microsoft (par exemple, Skype, Xbox, Outlook.com)** , la version de jeton acceptée est v2.0.
- Sinon, la version de jeton acceptée est v1.0.

Après avoir créé l’application, vous pouvez déterminer ou changer la version de jeton acceptée en procédant comme suit :

1. Dans le Portail Azure, sélectionnez votre application puis **Manifeste**.
1. Recherchez la propriété **accessTokenAcceptedVersion** dans le manifeste.
1. La valeur spécifie à Azure Active Directory (Azure AD) la version de jeton acceptée par l’API web.
    - Si la valeur est 2, l’API web accepte les jetons v2.0.
    - Si la valeur est **Null**, l’API web accepte les jetons v1.0.
1. Si vous avez modifié la version de jeton, sélectionnez **Enregistrer**.

> [!NOTE]
> L’API web spécifie la version de jeton qu’elle accepte. Quand un client demande un jeton pour votre API web auprès du point de terminaison de la plateforme d’identités Microsoft (v2.0), le client obtient un jeton qui indique quelle version de jeton est acceptée par l’API web.

## <a name="no-redirect-uri"></a>Aucun URI de redirection

Les API web n’ont pas besoin d’inscrire d’URI de redirection, car aucun utilisateur n’est connecté de manière interactive.

## <a name="exposed-api"></a>API exposée

D’autres paramètres spécifiques aux API web sont l’API exposée et les étendues exposées ou rôles d’application.

### <a name="application-id-uri-and-scopes"></a>URI d’ID d’application et étendues

Les étendues se présentent généralement sous la forme `resourceURI/scopeName`. Pour Microsoft Graph, les étendues ont des raccourcis. Par exemple, `User.Read` est un raccourci pour `https://graph.microsoft.com/user.read`.

Lors de l’inscription de l’application, vous devez définir les paramètres suivants :

- L’URI de ressource
- Une ou plusieurs étendues
- Un ou plusieurs rôles d’application

Par défaut, le portail d’inscription des applications vous recommande d’utiliser l’URI de ressource `api://{clientId}`. Cet URI est unique, mais n’est pas lisible. Si vous modifiez l’URI, assurez-vous que la nouvelle valeur est unique. Le portail d’inscription des applications s’assure que vous utilisez un [domaine de serveur de publication configuré](howto-configure-publisher-domain.md).

Pour les applications clientes, les étendues s’affichent en tant que *permissions déléguées* et les rôles d’application en tant que *permissions d’application* pour votre API web.

Les étendues s’affichent également dans la fenêtre de consentement présentée aux utilisateurs de votre application. Par conséquent, vous devez fournir les chaînes correspondantes qui décrivent l’étendue :

- Comme vue par un utilisateur.
- Comme vue par l’administrateur client, qui peut accorder un consentement administrateur.

Les rôles d’application ne peuvent pas être consentis par un utilisateur (car ils sont utilisés par une application qui appelle l’API Web pour son propre compte). Un administrateur client devra donner son consentement aux applications clientes de votre API Web qui exposent des rôles d’application. Pour plus d’informations, consultez [Consentement administrateur](v2-admin-consent.md)

### <a name="exposing-delegated-permissions-scopes"></a>Exposition d’autorisations déléguées (étendues)

1. Sélectionnez **Exposer une API** dans l’inscription de l’application.
1. sélectionner **Ajouter une étendue**.
1. Si vous y êtes invité, acceptez l’URI d’ID d’application proposé (`api://{clientId}`) en sélectionnant **Enregistrer et continuer**.
1. Spécifiez les valeurs suivantes :
    - Sélectionnez **Nom de l’étendue** et entrez **access_as_user**.
    - Sélectionnez **Qui peut donner son consentement** et assurez-vous que l’option **Administrateurs et utilisateurs** est sélectionnée.
    - Sélectionnez **Nom d’affichage du consentement administrateur** et entrez **Accéder à TodoListService en tant qu’utilisateur**.
    - Sélectionnez **Description du consentement administrateur** et entrez **Accède à l’API web TodoListService en tant qu’utilisateur**.
    - Sélectionnez **Nom d’affichage du consentement de l’utilisateur** et entrez **Accéder à TodoListService en tant qu’utilisateur**.
    - Sélectionnez **Description du consentement de l’utilisateur** et entrez **Accède à l’API web TodoListService en tant qu’utilisateur**.
    - Conservez la valeur **État** définie sur **Activé**.
 1. Sélectionnez **Ajouter une étendue**.

### <a name="if-your-web-api-is-called-by-a-daemon-app"></a>Si votre API web est appelée par une application démon

Dans cette section, vous apprenez à inscrire votre API web protégée afin que les applications démon puissent l’appeler.

- Vous déclarez et exposez uniquement les *permissions d’application*, car les applications démon n’interagissent pas avec les utilisateurs. Des permissions déléguées n’auraient aucun sens.
- Les administrateurs client peuvent exiger d’Azure AD qu’il émette des jetons d’API web uniquement aux applications qui se sont inscrites pour accéder aux permissions d’application de l’une des API.

#### <a name="exposing-application-permissions-app-roles"></a>Exposition des autorisations d’application (rôles d’application)

Pour exposer les permissions d’application, vous devez modifier le manifeste.

1. Dans l’inscription d’application pour votre application, sélectionnez **Manifeste**.
1. Pour modifier le manifeste, recherchez le paramètre `appRoles` et ajoutez des rôles d’application. La définition de rôle est fournie dans l’exemple de bloc JSON suivant.
1. Laissez `allowedMemberTypes` défini sur `"Application"` uniquement.
1. Assurez-vous que `id` est un GUID unique.
1. Assurez-vous que `displayName` et `value` ne contiennent pas d’espaces.
1. Enregistrez le manifeste.

L’exemple suivant montre le contenu de `appRoles`, où la valeur de `id` peut être n’importe quel GUID unique.

```json
"appRoles": [
    {
    "allowedMemberTypes": [ "Application" ],
    "description": "Accesses the TodoListService-Cert as an application.",
    "displayName": "access_as_application",
    "id": "ccf784a6-fd0c-45f2-9c08-2f9d162a0628",
    "isEnabled": true,
    "lang": null,
    "origin": "Application",
    "value": "access_as_application"
    }
],
```

#### <a name="ensuring-that-azure-ad-issues-tokens-for-your-web-api-to-only-allowed-clients"></a>S’assurer qu’Azure AD émet des jetons pour votre API web uniquement aux clients autorisés

L’API web recherche le rôle d’application. Ce rôle est un moyen pour les développeurs de logiciels d’exposer les permissions d’application. Vous pouvez également configurer Azure AD de sorte qu’il émette des jetons d’API uniquement aux applications approuvées par l’administrateur client pour accéder aux API.

Pour ajouter cette sécurité renforcée :

1. Accédez à la page **Vue d’ensemble** de l’application pour l’inscription de votre application.
1. Sous **Application managée dans l’annuaire local**, sélectionnez le lien portant le nom de votre application. L’étiquette de cette sélection peut être tronquée. Par exemple, vous pouvez voir **Application managée dans …**

   > [!NOTE]
   >
   > Lorsque vous sélectionnez ce lien, vous accédez à la page **Vue d’ensemble de l’application d’entreprise**. Cette page est associée au principal de service de votre application dans le locataire où vous l’avez créée. Vous pouvez accéder à la page d’inscription d’application à l’aide du bouton Précédent de votre navigateur.

1. Sélectionnez la page **Propriétés** dans la section **Gérer** des pages d’application d’entreprise.
1. Si vous souhaitez qu’Azure AD autorise l’accès à votre API web à partir de certains clients seulement, définissez **Assignation requise de utilisateur ?** sur **Oui**.

   > [!IMPORTANT]
   >
   > Si vous définissez **Attribution de l’utilisateur requise ?** sur **Oui**, Azure AD vérifie les attributions de rôles d’application d’un client lorsqu’il demande un jeton d’accès pour l’API web. Si le client n’est attribué à aucun rôle d’application, Azure AD renvoie le message d’erreur « invalid_client: AADSTS501051: L’application \<application name\> n’est pas affectée à un rôle pour \<web API\> ».
   >
   > Si vous maintenez l’option **Attribution de l’utilisateur requise ?** définie sur **Non**, Azure AD ne vérifiera pas les attributions de rôles d’application lorsqu’un client demandera un jeton d’accès pour votre API web. N’importe quel client démon, c’est-à-dire tous les clients utilisant le flux d’informations d’identification du client, peut obtenir un jeton d’accès pour l’API en spécifiant simplement son public. N’importe quelle application peut accéder à l’API sans avoir à demander de permissions pour cela.
   >
   > Toutefois, comme cela est expliqué dans la section précédente, votre API web peut toujours vérifier que l’application possède le rôle approprié, ce qui est autorisé par l’administrateur client. L’API effectue cette vérification en validant que le jeton d’accès a une revendication de rôles et que la valeur de cette revendication est correcte. Dans l’exemple JSON précédent, la valeur est `access_as_application`.

1. Sélectionnez **Enregistrer**.

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Configuration du code de l’application](scenario-protected-web-api-app-configuration.md).
