---
title: Passer en production une application monopage - Plateforme d’identités Microsoft | Azure
description: En savoir plus sur la création d’une application monopage (passer en production)
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: nacanuma
ms.custom: aaddev
ms.openlocfilehash: 0a51442870fb72e2b3cd93d9f03736d2c679ed06
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94442818"
---
# <a name="single-page-application-move-to-production"></a>Application monopage : Passer en production

Maintenant que vous savez comment acquérir un jeton pour appeler des API web, découvrez comment passer en production.

## <a name="improve-your-app"></a>Améliorer votre application

[Activez la journalisation](msal-logging.md) pour préparer votre application à la production.

## <a name="test-your-integration"></a>Tester votre intégration

Testez votre intégration en suivant la [check-list de l’intégration à la plateforme d’identités Microsoft](identity-platform-integration-checklist.md).

## <a name="deploy-your-app"></a>Déployer votre application

Consultez un [exemple de déploiement](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnet-webapi-multitenant/tree/master/Chapter3) pour apprendre à déployer vos projets de SPA et d’API web respectivement avec le service Stockage Azure et Azure App Services. 

## <a name="next-steps"></a>Étapes suivantes

- Présentation approfondie de l’exemple de démarrage rapide, avec explication du code permettant de connecter des utilisateurs et d’obtenir un jeton d’accès pour appeler l’**API Microsoft Graph** à l’aide de **MSAL.js** : [Tutoriel de SPA JavaScript](./tutorial-v2-javascript-spa.md).

- Exemple montrant comment obtenir des jetons pour votre propre API web de serveur principal (ASP.NET Core) à l’aide de **MSAL.js** : [SPA avec serveur principal ASP.NET](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnetcore-webapi).

- Exemple montrant comment valider des jetons d’accès pour votre API web de serveur principal (Node.js) à l’aide de **passport-azure-ad** : [API web Node.js (Azure AD)](https://github.com/Azure-Samples/active-directory-javascript-nodejs-webapi-v2).

- Exemple montrant comment utiliser **MSAL.js** pour connecter des utilisateurs dans une application inscrite auprès d’**Azure Active Directory B2C** (Azure AD B2C) : [SPA avec Azure AD B2C](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp).

- Exemple montrant comment utiliser **passport-azure-ad** pour valider des jetons d’accès pour des applications inscrites auprès d’**Azure Active Directory B2C** (Azure AD B2C) : [API web Node.js (Azure AD B2C)](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi).
