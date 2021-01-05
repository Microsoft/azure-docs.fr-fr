---
title: Fichier Include
description: Fichier Include
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: include file
ms.openlocfilehash: 3d4e45d1bf53bab4d1f9c45367f9d051f1668e2b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76309038"
---
### <a name="authenticationresult-properties-in-msalnet"></a>Propriétés AuthenticationResult dans MSAL.NET

Les méthodes d'acquisition de jetons renvoient `AuthenticationResult`. Pour les méthodes asynchrones, `Task<AuthenticationResult>` retourne.

Dans MSAL.NET, `AuthenticationResult` expose :

- `AccessToken` pour que l’API web accède aux ressources. Ce paramètre correspond à une chaîne, généralement un JWT encodé en base 64. Le client ne doit jamais regarder dans le jeton d’accès. La stabilité du format n’est pas garantie et ce dernier peut être chiffré pour la ressource. L’écriture de code qui dépend du contenu des jetons d’accès sur le client constitue l’une des sources d’erreurs et des ruptures de logique client les plus importantes. Pour plus d’informations, consultez [Jetons d'accès](../articles/active-directory/develop/access-tokens.md).
- `IdToken` pour l'utilisateur. Ce paramètre est un jeton JWT encodé. Pour plus d'informations, consultez [Jetons d’ID](../articles/active-directory/develop/id-tokens.md).
- `ExpiresOn` exprime la date et l'heure d’expiration du jeton.
- `TenantId` contient le locataire dans lequel l’utilisateur a été trouvé. Pour les utilisateurs invités d'Azure Active Directory (Azure AD) B2B, l’ID de locataire correspond au locataire invité, pas au locataire unique.
Lorsque le jeton est remis pour un utilisateur, `AuthenticationResult` contient également les informations sur cet utilisateur. Pour les flux de clients confidentiels où les jetons sont demandés sans aucun utilisateur pour l’application, ces informations sur l’utilisateur sont manquantes ou inconnues.
- Les `Scopes` pour lesquelles le jeton a été émis.
- L’ID unique de l’utilisateur.

### <a name="iaccount"></a>IAccount

MSAL.NET définit la notion de compte via l’interface `IAccount`. Ce changement cassant fournit la sémantique qui convient. Un même utilisateur peut disposer de plusieurs comptes, dans des répertoires Azure AD différents. De plus, MSAL.NET fournit de meilleures informations dans les scénarios d’invité, car des informations de compte d’accueil sont fournies.
Le schéma suivant présente la structure de l’interface `IAccount`.

![Structure de l’interface IAccount](https://user-images.githubusercontent.com/13203188/44657759-4f2df780-a9fe-11e8-97d1-1abbffade340.png)

La classe `AccountId` identifie un compte dans un locataire spécifique avec les propriétés indiquées dans le tableau suivant.

| Propriété | Description |
|----------|-------------|
| `TenantId` | Une représentation de chaîne pour un identificateur global unique, qui correspond à l’ID du locataire où réside le compte. |
| `ObjectId` | Une représentation de chaîne pour un identificateur global unique, qui correspond à l’ID de l’utilisateur qui possède le compte dans le locataire. |
| `Identifier` | Identificateur unique pour ce compte. `Identifier` est la concaténation de `ObjectId` et de `TenantId` séparés par une virgule. Ils ne sont pas encodés base64. |

L’interface `IAccount` représente les informations d’un seul compte. Le même utilisateur peut être présent dans différents locataires, ce qui signifie qu'un utilisateur peut disposer de plusieurs comptes. Ses membres sont disponibles dans le tableau suivant.

| Propriété | Description |
|----------|-------------|
| `Username` | Une chaîne contenant la valeur affichée au format UserPrincipalName (UPN), par exemple, john.doe@contoso.com. Cette chaîne peut être Null, alors que HomeAccountId et HomeAccountId.Identifier ne le sont pas. Cette propriété remplace la propriété `DisplayableId` de `IUser` dans les versions précédentes de MSAL.NET. |
| `Environment` | Une chaine contenant le fournisseur d’identité de ce compte, par exemple, `login.microsoftonline.com`. Cette propriété remplace la propriété `IdentityProvider` de `IUser`, sauf que `IdentityProvider` disposait aussi des informations sur le locataire (en plus de l’environnement cloud). Ici, la valeur n’est que l’hôte. |
| `HomeAccountId` | L'ID du compte d’accueil de l’utilisateur. Cette propriété identifie de façon unique l’utilisateur entre les locataires Azure AD. |

### <a name="use-the-token-to-call-a-protected-api"></a>Utiliser le jeton pour appeler une API protégée

Après renvoi de `AuthenticationResult` par MSAL dans `result`, ajoutez-le à l’en-tête d’autorisation HTTP avant de passer l’appel pour accéder à l’API web protégée.

```csharp
httpClient = new HttpClient();
httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);

// Call the web API.
HttpResponseMessage response = await _httpClient.GetAsync(apiUri);
...
}
```