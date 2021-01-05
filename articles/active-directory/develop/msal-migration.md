---
title: Migrer vers la bibliothèque d’authentification Microsoft (MSAL)
titleSuffix: Microsoft identity platform
description: Découvrez les différences entre la bibliothèque d’authentification Microsoft (MSAL) et la bibliothèque d’authentification Azure AD (ADAL), et apprenez à effectuer une migration vers MSAL.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/07/2020
ms.author: jmprieur
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: ac9264ea8d6cc71d19d2c9bbd23b2123bdf1f924
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88224354"
---
# <a name="migrate-applications-to-microsoft-authentication-library-msal"></a>Migrer des applications vers la bibliothèque d’authentification Microsoft (MSAL)

De nombreux développeurs ont créé et déployé des applications à l’aide de la Bibliothèque d'authentification Azure Active Directory (ADAL). Nous vous recommandons maintenant d’utiliser la Bibliothèque d’authentification Microsoft (MSAL) pour l’authentification et l’autorisation des entités Azure AD.

En utilisant MSAL au lieu d’ADAL :

- vous pouvez authentifier un ensemble d’identités plus large :
  - des identités Azure AD ;
  - Comptes Microsoft
  - des comptes sociaux et locaux à l’aide Azure AD B2C ;
- vos utilisateurs bénéficieront de la meilleure expérience d’authentification unique ;
- Votre application peut activer le consentement incrémentiel.
- La prise en charge de l’accès conditionnel est simplifiée.
- Vous bénéficiez de l’innovation. Étant donné que tous les efforts de développement Microsoft se concentrent désormais sur MSAL, aucune nouvelle fonctionnalité ne sera implémentée dans ADAL.

**MSAL est maintenant la bibliothèque d’authentification recommandée à utiliser avec la plateforme d’identité Microsoft**.

## <a name="migration-guidance"></a>Recommandations en matière de migration

Les articles suivants peuvent vous aider à migrer vers MSAL :

- [Migrer vers MSAL.Android](migrate-android-adal-msal.md)
- [Migrer vers MSAL.iOS / macOS](migrate-objc-adal-msal.md)
- [Migrer vers MSAL Java](migrate-adal-msal-java.md)
- [Migrer vers MSAL.js](msal-compare-msal-js-and-adal-js.md)
- [Migrer vers MSAL.NET](msal-net-migration.md)
- [Migrer vers MSAL Python](migrate-python-adal-msal.md)
- [Migrer des applications Xamarin avec des répartiteurs vers MSAL.NET](msal-net-migration-ios-broker.md)

## <a name="frequently-asked-questions-faq"></a>Forum Aux Questions (FAQ)

__Q : ADAL est-il déconseillé ?__  
A : Oui. À partir du 30 juin 2020, nous n’ajouterons plus de nouvelles fonctionnalités à ADAL. Nous continuerons à ajouter des correctifs de sécurité critiques à ADAL jusqu’au 30 juin 2022. Après cette date, vos applications qui utilisent ADAL continueront de fonctionner, mais nous vous recommandons de procéder à une mise à niveau vers MSAL pour tirer parti des dernières fonctionnalités tout en veillant à la sécurité.

__Q : Mes applications ADAL existantes cesseront-elles de fonctionner ?__  
A : Non. Vos applications existantes continueront de fonctionner sans modification. Si vous envisagez de les conserver au-delà du 30 juin 2022, vous devez envisager de mettre à jour vos applications vers MSAL pour préserver leur sécurité, mais la migration vers MSAL n’est pas nécessaire pour assurer la maintenance des fonctionnalités existantes.

__Q : Comment savoir laquelle de mes applications utilise ADAL ?__  
A : Si vous disposez du code source de l’application, vous pouvez vous référer aux guides de migration ci-dessus pour déterminer la bibliothèque utilisée par l’application et savoir comment la migrer vers MSAL. Si vous avez conclu un partenariat avec un éditeur de logiciels indépendant, nous vous suggérons de le contacter directement pour comprendre son parcours de migration vers MSAL.

__Q : Pourquoi dois-je investir pour passer à MSAL ?__  
A : MSAL contient de nouvelles fonctionnalités manquantes dans ADAL, notamment le consentement incrémentiel, l’authentification unique et la gestion du cache de jetons. En outre, contrairement à ADAL, MSAL continuera de recevoir des correctifs de sécurité au-delà du 30 juin 2022. [Plus d’informations](msal-overview.md)

__Q : Microsoft mettra-t-il à jour ses propres applications sur MSAL ?__  
Oui. Microsoft est en train de migrer ses applications vers MSAL à l’échéance de fin de support, garantissant ainsi qu’elles pourront bénéficier des améliorations en cours apportées à la sécurité et aux fonctionnalités de MSAL.

__Q : Allez-vous publier un outil qui m’aidera à migrer mes applications d’ADAL vers MSAL ?__  
A : Non. Les différences entre les bibliothèques nécessitent que des ressources soient dédiées au développement et à la maintenance de l’outil qui, autrement, serait consacré à l’amélioration de MSAL. Toutefois, nous fournissons l’ensemble précédent de guides de migration pour vous aider à effectuer les modifications nécessaires dans votre application.

__Q : Comment MSAL fonctionne-t-il avec AD FS ?__  
A : MSAL.NET prend en charge certains scénarios pour s’authentifier auprès d’AD FS 2019. Si votre application doit acquérir des jetons directement à partir d’une version antérieure d’AD FS, vous devez rester sur ADAL. [Plus d’informations](msal-net-adfs-support.md)

__Q : Comment obtenir de l’aide sur la migration de mon application ?__  
A : Consultez la section [Conseils de migration](#migration-guidance) de cet article. Si, après avoir lu le guide de la plateforme de votre application, vous avez des questions supplémentaires, vous pouvez publier sur Stack Overflow avec l’étiquette `[adal-deprecation]` ou ouvrir un problème dans le référentiel GitHub de la bibliothèque. Consultez la section [Langages et infrastructures](msal-overview.md#languages-and-frameworks) de l’article de présentation de MSAL pour obtenir des liens vers les référentiel de chaque bibliothèque.

## <a name="next-steps"></a>Étapes suivantes

- [Mettre à jour vos applications pour utiliser la bibliothèque d’authentification Microsoft et l’API Microsoft Graph](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/update-your-applications-to-use-microsoft-authentication-library/ba-p/1257363)
- [Vue d’ensemble de la Plateforme d’identités Microsoft](v2-overview.md)
- [Consultez nos exemples de code MSAL](sample-v2-code.md)
