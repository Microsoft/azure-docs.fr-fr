---
title: Autorisations dans Azure Active Directory | Microsoft Docs
description: Découvrez les autorisations dans Azure Active Directory et leur utilisation.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.workload: identity
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ryanwi
ms.reviewer: jesakowi
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: 2b85115d905cb6a7eb7c6aed64a4834425d2f1d7
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92366392"
---
# <a name="permissions-and-consent-in-the-azure-active-directory-v10-endpoint"></a>Autorisations et consentement dans le point de terminaison Azure Active Directory v1.0

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

Azure Active Directory (Azure AD) utilise très souvent des autorisations pour les flux OAuth et OpenID Connect (OIDC). Quand votre application reçoit un jeton d’accès d’Azure AD, ce jeton inclut des revendications qui décrivent les autorisations dont dispose votre application par rapport à une ressource particulière.

Les *autorisations* , également appelées *étendues* , facilitent l’autorisation pour la ressource, car cette dernière a uniquement besoin de vérifier que le jeton contient l’autorisation appropriée pour l’API appelée par l’application.

## <a name="types-of-permissions"></a>Types d’autorisations

Azure AD définit deux types d’autorisations :

* **Autorisations déléguées**  : utilisées par les applications qui ont un utilisateur connecté présent. Pour ces applications, l’utilisateur ou un administrateur accorde les autorisations que l’application demande. Ensuite, l’application se voit déléguer une autorisation d’agir en tant qu’utilisateur connecté lors des appels à une API. En fonction de l’API, l’utilisateur peut ne pas être en mesure de donner son consentement à l’API directement et [avoir besoin d’un « consentement de l’administrateur »](../develop/howto-convert-app-to-be-multi-tenant.md).
* **Autorisations d’application**  : utilisées par les applications qui s’exécutent sans utilisateur connecté présent ; par exemple, les applications qui s’exécutent en tant que services ou démons en arrière-plan. Les permissions d’application ne peuvent être [accordées que par un administrateur](../develop/v2-permissions-and-consent.md#requesting-consent-for-an-entire-tenant), car elles sont généralement puissantes et permettent d’accéder à des données situées au-delà des limites de l’utilisateur ou à des données normalement réservées aux seuls administrateurs. Les utilisateurs définis en tant que propriétaires de l’application de ressource (c’est-à-dire l’API qui publie les autorisations) sont également autorisés à accorder des permissions d’application pour leurs API.

Les autorisations effectives sont celles qu’a votre application lors des requêtes faites à une API. 

* Pour les autorisations déléguées, les autorisations effectives de votre application correspondent au niveau de privilège le moins élevé entre les autorisations déléguées que l’application a reçues (par le biais d’un consentement) et les privilèges de l’utilisateur actuellement connecté. Votre application ne peut jamais avoir plus de privilèges que l’utilisateur connecté. Au sein des organisations, les privilèges de l’utilisateur connecté peuvent être déterminés par la stratégie ou l’appartenance à un ou plusieurs rôles d’administrateur. Pour connaître les rôles administrateur habilités à donner leur consentement pour les autorisations déléguées, consultez [Autorisations du rôle administrateur dans Azure AD](../roles/permissions-reference.md).
    Par exemple, supposons que votre application ait reçu l’autorisation déléguée `User.ReadWrite.All` dans Microsoft Graph. Cette autorisation permet nominalement à votre application de lire et mettre à jour le profil de chaque utilisateur dans une organisation. Si l’utilisateur connecté est un administrateur général, votre application est en mesure de mettre à jour le profil de chaque utilisateur de l’organisation. Toutefois, si l’utilisateur connecté n’a pas de rôle d’administrateur, votre application peut uniquement mettre à jour le profil de l’utilisateur connecté. Elle ne peut pas mettre à jour les profils des autres utilisateurs de l’organisation, car l’utilisateur pour lequel elle est autorisée à agir n’a pas ces privilèges.
* Pour les autorisations d’application, les autorisations effectives de votre application correspondent au niveau complet des privilèges impliqués par l’autorisation. Par exemple, une application qui a l’autorisation `User.ReadWrite.All` peut mettre à jour le profil de chaque utilisateur de l’organisation.

## <a name="permission-attributes"></a>Attributs d’autorisation
Les autorisations dans Azure AD ont plusieurs propriétés qui aident les utilisateurs, administrateurs ou développeurs d’applications à prendre des décisions avisées sur les éléments auxquels l’autorisation accorde l’accès.

> [!NOTE]
> Vous pouvez voir les autorisations exposées par une application Azure AD ou un principal du service à l’aide du portail Azure ou de PowerShell. Essayez ce script pour voir les autorisations exposées par Microsoft Graph.
> ```powershell
> Connect-AzureAD
> 
> # Get OAuth2 Permissions/delegated permissions
> (Get-AzureADServicePrincipal -filter "DisplayName eq 'Microsoft Graph'").OAuth2Permissions
> 
> # Get App roles/application permissions
> (Get-AzureADServicePrincipal -filter "DisplayName eq 'Microsoft Graph'").AppRoles
> ```

| Nom de la propriété | Description | Exemple |
| --- | --- | --- |
| `ID` | Valeur GUID qui identifie cette autorisation de façon unique. | 570282fd-fa5c-430d-a7fd-fc8dc98a9dca |
| `IsEnabled` | Indique si cette autorisation est disponible à l’utilisation. | true |
| `Type` | Indique si cette autorisation nécessite le consentement de l’utilisateur ou de l’administrateur. | Utilisateur |
| `AdminConsentDescription` | Description présentée aux administrateurs pendant leur expérience de consentement. | Autorise l’application à lire des e-mails dans des boîtes aux lettres utilisateur. |
| `AdminConsentDisplayName` | Nom convivial présenté aux administrateurs pendant leur expérience de consentement. | Accéder en lecture aux e-mails utilisateur |
| `UserConsentDescription` | Description présentée aux utilisateurs pendant une expérience de consentement. |  Autorise l’application à lire des e-mails dans votre boîte aux lettres. |
| `UserConsentDisplayName` | Nom convivial présenté aux utilisateurs pendant une expérience de consentement. | Lire vos e-mails |
| `Value` | Chaîne utilisée pour identifier l’autorisation pendant des flux d’autorisation OAuth 2.0. `Value` peut également être combinée avec la chaîne d’URI d’ID d’application pour former un nom d’autorisation complet. | `Mail.Read` |

## <a name="types-of-consent"></a>Types de consentement

Les applications dans Azure AD reposent sur un consentement pour obtenir l’accès aux ressources ou API nécessaires. Il existe plusieurs types de consentement dont votre application peut avoir besoin pour fonctionner correctement. Si vous définissez des autorisations, vous devez également comprendre la façon dont vos utilisateurs vont accéder à votre application ou API.

* **Consentement de l’utilisateur statique**  : se produit automatiquement pendant le [flux d’autorisation OAuth 2.0](v1-protocols-oauth-code.md#request-an-authorization-code) quand vous spécifiez la ressource avec laquelle votre application veut interagir. Dans le cadre d’un consentement de l’utilisateur statique, votre application doit déjà avoir spécifié toutes les autorisations dont elle a besoin dans sa configuration dans le portail Azure. Si l’utilisateur (ou administrateur, selon le cas) n’a pas donné son consentement à cette application, Azure AD invite l’utilisateur à le donner à ce stade. 

    Découvrez-en plus sur l’inscription d’une application Azure AD qui demande l’accès à un ensemble statique d’API.
* **Consentement de l’utilisateur dynamique**  : fonctionnalité du modèle d’application Azure AD v2. Dans ce scénario, votre application demande un ensemble d’autorisations dont elle a besoin dans le [flux d’autorisation OAuth 2.0 pour des applications v2](../develop/v2-permissions-and-consent.md#requesting-individual-user-consent). Si l’utilisateur n’a pas déjà donné son consentement, il est invité à le faire à ce stade. [Découvrez-en plus sur le consentement dynamique](./azure-ad-endpoint-comparison.md#incremental-and-dynamic-consent).

    > [!IMPORTANT]
    > Le consentement dynamique peut s’avérer pratique, mais il représente un véritable défi pour les autorisations qui nécessitent un consentement de l’administrateur, dans la mesure où l’expérience de consentement de l’administrateur n’a pas connaissance de ces autorisations au moment du consentement. Si vous avez besoin d’autorisations à privilège administratif ou si votre application utilise le consentement dynamique, vous devez inscrire toutes les autorisations dans le portail Azure (pas seulement le sous-ensemble d’autorisations qui nécessite le consentement de l’administrateur). Ceci permet aux administrateurs de locataires de donner leur consentement au nom de tous leurs utilisateurs.
  
* **Consentement de l’administrateur**  : nécessaire quand votre application a besoin d’accéder à certaines autorisations à privilèges élevés. Le consentement de l’administrateur garantit que les administrateurs disposent de contrôles supplémentaires avant d’autoriser des applications ou des utilisateurs à accéder aux données à privilèges élevés de l’organisation. [Découvrez-en plus sur la manière d’accorder un consentement de l’administrateur](../develop/v2-permissions-and-consent.md#using-the-admin-consent-endpoint).

## <a name="best-practices"></a>Meilleures pratiques

### <a name="client-best-practices"></a>Meilleures pratiques clientes

- Demandez uniquement les autorisations dont votre application a besoin. Les applications qui ont trop d’autorisations courent le risque d’exposer des données utilisateur si elles sont compromises.
- Choisissez entre les autorisations déléguées et les autorisations d’application en fonction du scénario que votre application prend en charge.
    - Utilisez toujours des autorisations déléguées si l’appel est effectué pour le compte d’un utilisateur.
    - Utilisez uniquement des autorisations d’application si l’application est non interactive et qu’elle n’effectue pas d’appels pour le compte d’un utilisateur spécifique. Les autorisations d’application sont à privilèges élevés et ne doivent être utilisées qu’en cas de nécessité absolue.
- Quand vous utilisez une application basée sur le point de terminaison v2.0, définissez toujours les autorisations statiques (celles spécifiées dans votre inscription d’application) pour qu’elles soient le sur-ensemble des autorisations dynamiques que vous demandez au moment de l’exécution (celles spécifiées dans le code et envoyées en tant que paramètres de requête dans votre requête d’autorisation) afin que des scénarios comme le consentement de l’administrateur fonctionnent correctement.

### <a name="resourceapi-best-practices"></a>Bonnes pratiques pour les ressources/API

- Les ressources qui exposent des API doivent définir des autorisations spécifiques aux données ou actions qu’elles protègent. Suivre cette bonne pratique permet de garantir que les clients n’obtiennent pas une autorisation d’accéder à des données dont ils n’ont pas besoin et que les utilisateurs sont bien informés sur les données pour lesquelles ils donnent leur consentement.
- Les ressources doivent explicitement définir des autorisations `Read` et `ReadWrite` séparément.
- Les ressources doivent marquer toutes les autorisations qui permettent d’accéder à des données situées au-delà des limites de l’utilisateur comme des autorisations `Admin`.
- Les ressources doivent suivre le modèle de nommage `Subject.Permission[.Modifier]`, où :
  - `Subject` correspond au type de données disponibles
  - `Permission` correspond à l’action qu’un utilisateur peut effectuer sur ces données
  - `Modifier` est éventuellement utilisé pour décrire les spécialisations d’une autre autorisation
    
    Par exemple :
  - Mail.Read : autorise les utilisateurs à lire des messages.
  - Mail.ReadWrite : autorise les utilisateurs à lire ou écrire des messages.
  - Mail.ReadWrite.All : autorise un administrateur ou utilisateur à accéder à tous les messages de l’organisation.