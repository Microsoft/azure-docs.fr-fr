---
title: Authentification et clouds nationaux Azure AD | Azure
titleSuffix: Microsoft identity platform
description: Découvrez l’inscription d’application et les points de terminaison d’authentification pour les clouds nationaux.
services: active-directory
author: negoe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/28/2019
ms.author: negoe
ms.reviewer: marsma, negoe,celested
ms.custom: aaddev,references_regions
ms.openlocfilehash: da1ea6462d22242d23629bcec192d7760314dfed
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96444806"
---
# <a name="national-clouds"></a>Clouds nationaux

Les clouds nationaux sont des instances Azure physiquement isolées. Ces régions d’Azure sont conçues pour garantir que les conditions de résidence, de souveraineté et de conformité des données sont respectées au sein de limites géographiques.

Outre le cloud global, Azure Active Directory (Azure AD) est déployé dans les clouds nationaux suivants :  

- Azure Government
- Azure Germany
- Azure China 21Vianet

Les clouds nationaux sont des environnements uniques, différents d’Azure Global. Il est important d’être conscient des différences clés lors du développement de votre application pour ces environnements. Ces différences incluent l’inscription des applications, l’acquisition de jetons et la configuration des points de terminaison.

## <a name="app-registration-endpoints"></a>Points de terminaison d’inscription d’application

Il existe un Portail Azure distinct pour chacun des clouds nationaux. Pour intégrer des applications avec la plateforme d’identités Microsoft dans un cloud national, vous devez inscrire votre application séparément dans chaque Portail Azure propre à l’environnement.

Le tableau suivant répertorie les URL de base des points de terminaison Azure AD utilisés pour inscrire une application pour chaque cloud national.

| Cloud national | Point de terminaison du Portail Azure AD |
|----------------|--------------------------|
| Azure AD pour le gouvernement des États-Unis | `https://portal.azure.us` |
| Azure AD Allemagne | `https://portal.microsoftazure.de` |
| Azure AD Chine géré par 21Vianet | `https://portal.azure.cn` |
| Azure AD (service mondial) |`https://portal.azure.com` |

## <a name="azure-ad-authentication-endpoints"></a>Points de terminaison d’authentification Azure AD

Tous les clouds nationaux authentifient les utilisateurs séparément dans chaque environnement et ont des points de terminaison d’authentification distincts.

Le tableau suivant répertorie les URL de base des points de terminaison Azure AD utilisés pour obtenir des jetons pour chaque cloud national.

| Cloud national | Point de terminaison d’authentification Azure AD |
|----------------|-------------------------|
| Azure AD pour le gouvernement des États-Unis | `https://login.microsoftonline.us` |
| Azure AD Allemagne| `https://login.microsoftonline.de` |
| Azure AD Chine géré par 21Vianet | `https://login.chinacloudapi.cn` |
| Azure AD (service mondial)| `https://login.microsoftonline.com` |

Vous pouvez former des requêtes pour les points de terminaison de jeton ou d’autorisation Azure AD à l’aide de l’URL de base spécifique de la région appropriée. Par exemple, pour Azure Allemagne :

- Le point de terminaison d’autorisation commun est `https://login.microsoftonline.de/common/oauth2/v2.0/authorize`.
- Le point de terminaison de jeton commun est `https://login.microsoftonline.de/common/oauth2/v2.0/token`.

Pour les applications à client unique, remplacez « common » dans les URL ci-dessus par l’ID ou le nom de votre locataire. par exemple `https://login.microsoftonline.de/contoso.com`.

## <a name="microsoft-graph-api"></a>API Microsoft Graph

Pour savoir comment appeler les API Microsoft Graph dans un environnement de cloud national, accédez à [Microsoft Graph in national cloud deployments (Microsoft Graph dans les déploiements sur les clouds nationaux)](/graph/deployments).

> [!IMPORTANT]
> Certains services et fonctionnalités dans des régions spécifiques du service global peuvent ne pas être disponibles dans l’ensemble des clouds nationaux. Pour connaître les services disponibles, accédez à la [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/?products=all&regions=usgov-non-regional,us-dod-central,us-dod-east,usgov-arizona,usgov-iowa,usgov-texas,usgov-virginia,china-non-regional,china-east,china-east-2,china-north,china-north-2,germany-non-regional,germany-central,germany-northeast).

Pour savoir comment créer une application à l’aide de la plateforme d’identités Microsoft, suivez le [tutoriel de la bibliothèque d’authentification Microsoft (MSAL)](msal-national-cloud.md). Plus précisément, cette application doit connecter un utilisateur et obtenir un jeton d’accès pour appeler l’API Microsoft Graph.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment utiliser la [Bibliothèque d’authentification Microsoft (MSAL) dans un environnement cloud national](msal-national-cloud.md).

Documentation sur le cloud national :

- [Azure Government](../../azure-government/index.yml)
- [Azure China 21Vianet](/azure/china/)
- [Azure Allemagne](../../germany/index.yml)
