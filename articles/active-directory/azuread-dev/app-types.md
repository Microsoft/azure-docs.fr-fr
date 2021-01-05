---
title: Types d’applications dans v1.0 | Azure
description: Découvrez les types d’applications et scénarios pris en charge par le point de terminaison Azure Active Directory v1.0.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.workload: identity
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ryanwi
ms.reviewer: saeeda, jmprieur, andret
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: 5ff2858dd8b91ba036c517cbff07be96a729ef8c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88116442"
---
# <a name="application-types-in-v10"></a>Types d’applications dans v1.0

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

Azure Active Directory (Azure AD) prend en charge l’authentification pour de multiples architectures d’applications modernes, toutes basées sur des protocoles industriels standard OAuth 2.0 ou OpenID Connect.

Le diagramme suivant illustre les scénarios et types d’application, ainsi que la manière dont vous pouvez ajouter les différents composants :

![Types d’application et scénarios](./media/authentication-scenarios/application-types-scenarios.png)

Voici les cinq principaux scénarios d’application pris en charge par Azure AD :

- **[Application à page unique (SPA)](single-page-application.md)**  : un utilisateur doit se connecter à une application à page unique sécurisée par Azure AD.
- **[Navigateur web vers application web](web-app.md)**  : un utilisateur doit se connecter à une application web sécurisée par Azure AD.
- **[Application native vers API web](native-app.md)**  : une application native qui s’exécute sur un téléphone, une tablette ou un PC doit authentifier un utilisateur pour obtenir des ressources d’une API web sécurisée par Azure AD.
- **[Application web vers API web](web-api.md)**  : une application web doit obtenir des ressources d’une API web sécurisée par Azure AD.
- **[Application démon ou serveur vers API web](service-to-service.md)**  : une application démon ou une application serveur sans interface utilisateur web doit obtenir des ressources d’une API web sécurisée par Azure AD.

Cliquez sur les liens pour en savoir plus sur chaque type d’application et comprendre les lignes directrices des scénarios avant de commencer à travailler avec le code. Vous pouvez également obtenir des informations sur les différences à prendre en compte lorsque vous écrivez une application spécifique qui fonctionne avec le point de terminaison v1.0 ou le point de terminaison v2.0.

> [!NOTE]
> Le point de terminaison v2.0 ne prend pas en charge tous les scénarios et fonctionnalités d’Azure AD. Pour déterminer si vous devez utiliser le point de terminaison v2.0, consultez les [limitations de v2.0](./azure-ad-endpoint-comparison.md?bc=%2fazure%2factive-directory%2fazuread-dev%2fbreadcrumb%2ftoc.json&toc=%2fazure%2factive-directory%2fazuread-dev%2ftoc.json).

Vous pouvez développer tous les scénarios et applications décrits ici à l’aide de différents langages et plateformes. Ils sont tous appuyés par des exemples de code complets disponibles dans le guide des exemples de code : [Exemples de code par scénario (v1.0)](sample-v1-code.md) et [Exemples de code par scénario (v2.0)](../develop/sample-v2-code.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json). Vous pouvez également télécharger les exemples de code directement à partir des [référentiels d’exemples GitHub](https://github.com/Azure-Samples?q=active-directory) correspondants.

De plus, si votre application nécessite un élément ou segment spécifique d’un scénario de bout en bout, vous pouvez ajouter cette fonctionnalité séparément dans la plupart des cas. Par exemple, si vous avez une application native qui appelle une API web, vous pouvez facilement ajouter une application web qui appelle elle-aussi l’API web.

## <a name="app-registration"></a>Inscription d'application

### <a name="registering-an-app-that-uses-the-azure-ad-v10-endpoint"></a>Inscrire une application qui utilise le point de terminaison Azure AD v1.0

Toute application qui externalise l’authentification pour la confier à Azure AD doit être inscrite dans un annuaire. Cette étape consiste à donner des informations sur votre application à Azure AD, y compris son URL, l’URL à laquelle les réponses doivent être envoyées après l’authentification, l’URI permettant d’identifier votre application, etc. Ces informations sont obligatoires pour les raisons principales suivantes :

* Azure AD a besoin de communiquer avec l’application lors de l’authentification ou de l’échange de jetons. Les informations échangées entre Azure AD et l’application sont les suivantes :
  
  * **URI d’ID d’application** : identificateur d’une application. Cette valeur est envoyée à Azure AD lors de l’authentification afin d’indiquer pour quelle application l’appelant souhaite un jeton. Cette valeur est également incluse dans le jeton pour que l’application sache qu’il s’agit de la cible prévue.
  * **URL de réponse** et **URI de redirection** : pour une API web ou une application web, l’URL de réponse correspond à l’emplacement auquel Azure AD envoie la réponse d’authentification qui inclut un jeton si l’authentification a réussi. Pour une application native, l’URI de redirection est un identificateur unique vers lequel Azure AD redirige l’agent utilisateur dans une requête OAuth 2.0.
  * **ID d’application** : ID d’une application généré par Azure AD au moment de l’inscription de l’application. Lors d’une demande de code d’autorisation ou de jeton, l’ID et la clé d’application sont envoyés à Azure AD lors de l’authentification.
  * **Clé** : clé envoyée avec un ID d’application lors de l’authentification auprès d’Azure AD en vue d’appeler une API web.
* Azure AD doit assurer que l’application dispose des autorisations nécessaires pour accéder à vos données d’annuaire, à d’autres applications de votre organisation, etc.

Pour en savoir plus, découvrez comment [inscrire une application](../develop/quickstart-register-app.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

## <a name="single-tenant-and-multi-tenant-apps"></a>Application monolocataire et application multilocataire

L’approvisionnement devient plus clair lorsque vous comprenez qu’il existe deux catégories d’applications que vous pouvez développer et intégrer avec Azure AD :

* **Application à client unique** : une application à client unique est destinée à être utilisée dans une seule organisation. Il s’agit généralement d’applications métiers écrites par un développeur d’entreprise. Une application à client unique doit être accessible uniquement aux utilisateurs d’un annuaire et, en conséquence, ne doit être approvisionnée que dans un seul annuaire. Ces applications sont généralement inscrites par un développeur de l’organisation.
* **Application mutualisée** : une application mutualisée est destinée à être utilisée dans plusieurs organisations, non dans une seule. Il s’agit généralement d’applications SaaS (software-as-a-service) écrites par un éditeur de logiciels indépendant. Les applications mutualisées doivent être approvisionnées dans chaque annuaire dans lequel elles sont utilisées, ce qui suppose le consentement d’un utilisateur ou d’un administrateur pour les inscrire. Ce processus de consentement démarre quand une application a été enregistrée dans l’annuaire et accède à l’API Graph ou à une autre API web. Lorsqu’un utilisateur ou un administrateur d’une autre organisation s’inscrit pour utiliser l’application, une boîte de dialogue contenant les autorisations que l’application requiert s’affiche. L’utilisateur ou l’administrateur peut alors donner son consentement à l’application, ce qui permet à cette dernière d’accéder aux données indiquées et inscrit l’application dans l’annuaire de l’utilisateur ou de l’administrateur. Pour plus d’informations, consultez la page [Vue d’ensemble de l’infrastructure de consentement](../develop/consent-framework.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

### <a name="additional-considerations-when-developing-single-tenant-or-multi-tenant-apps"></a>Considérations supplémentaires lors du développement d’applications à client unique ou à plusieurs clients

Vous devez tenir compte d’autres éléments lorsque vous choisissez de développer une application mutualisée plutôt qu’une application à client unique. Par exemple, si vous mettez votre application à la disposition des utilisateurs dans plusieurs annuaires, vous devez disposer d’un mécanisme permettant de déterminer dans quel locataire ils se trouvent. Il suffit à une application à client unique de rechercher l’utilisateur dans son propre annuaire, alors qu’une application mutualisée doit tenir compte de tous les annuaires d’Azure AD pour identifier un utilisateur particulier. À cet effet, Azure AD fournit un point de terminaison d’authentification commun vers lequel une application mutualisée peut diriger les demandes de connexion, plutôt que vers un point de terminaison spécifique au client. Ce point de terminaison est `https://login.microsoftonline.com/common` pour tous les annuaires dans Azure AD, tandis qu’un point de terminaison spécifique d’un client peut être `https://login.microsoftonline.com/contoso.onmicrosoft.com`. Lorsque vous développez votre application, il est particulièrement important de tenir compte du point de terminaison commun, car vous aurez besoin de la logique nécessaire à la gestion de plusieurs locataires lors de la connexion, de la déconnexion et de la validation des jetons.

Si vous développez actuellement une application à client unique, mais que vous souhaitez la mettre à disposition de plusieurs organisations, vous pouvez facilement apporter des modifications à l’application et à sa configuration dans Azure AD pour la rendre compatible avec la mutualisation. De plus, Azure AD utilise la même clé de signature pour tous les jetons de tous les annuaires, que vous fournissiez l’authentification dans une application à client unique ou mutualisée.

Chaque scénario répertorié dans ce document inclut une sous-section décrivant les exigences d’approvisionnement associées. Pour obtenir des informations plus détaillées sur l’approvisionnement d’une application dans Azure AD et sur les différences entre les applications à client unique et les applications mutualisées, consultez [Intégration d’applications dans Azure Active Directory](../develop/single-and-multi-tenant-apps.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json). Poursuivez votre lecture pour comprendre les scénarios d’application courants dans Azure AD.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [principes de base de l’authentification Azure AD](v1-authentication-scenarios.md)