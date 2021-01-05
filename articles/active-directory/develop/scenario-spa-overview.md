---
title: Scénario d’application monopage JavaScript - Plateforme d’identités Microsoft | Azure
description: Découvrez comment créer une application monopage (vue d’ensemble du scénario) à l’aide de la plateforme d’identités Microsoft.
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: nacanuma
ms.custom: aaddev, identityplatformtop40, devx-track-js
ms.openlocfilehash: 17acb2bc5e96a136f31371c0be912c2c758c0f76
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94443855"
---
# <a name="scenario-single-page-application"></a>Scénario : Application monopage

Découvrez tout ce que vous devez savoir pour créer une application monopage.

## <a name="getting-started"></a>Prise en main

Si vous ne l’avez pas déjà fait, créez votre première application en suivant le guide de démarrage rapide SPA JavaScript :

[Démarrage rapide : application monopage](./quickstart-v2-javascript.md)

## <a name="overview"></a>Vue d’ensemble

De nombreuses applications web modernes sont créées en tant qu’applications monopages côté client. Les développeurs les écrivent à l’aide de JavaScript ou d’un framework d’application monopage, comme Angular, Vue et React. Ces applications s’exécutent sur un navigateur web et présentent des caractéristiques d’authentification différentes de celles des applications web classiques côté serveur.

La plateforme d’identités Microsoft offre **deux** options pour permettre aux applications monopages d’effectuer la connexion des utilisateurs et d’obtenir des jetons pour accéder aux services back-end ou aux API web :

- [Flux de code d’autorisation OAuth 2.0 (avec PKCE)](./v2-oauth2-auth-code-flow.md). Le flux de code d’autorisation permet à l’application d’échanger un code d’autorisation pour obtenir des jetons d’**ID** afin de représenter l’utilisateur authentifié et des jetons d’**accès** nécessaires pour appeler des API protégées. De plus, il retourne des jetons d’**actualisation** qui fournissent à votre application un accès à long terme à des ressources au nom d’utilisateurs sans nécessiter l’intervention de ces utilisateurs. Il s’agit de l’approche **recommandée**.

![Authentification des applications monopages](./media/scenarios/spa-app-auth.svg)

- [Flux implicite OAuth 2.0](./v2-oauth2-implicit-grant-flow.md). Le flux d’octroi implicite permet à l’application d’obtenir des jetons d’**ID** et d’**accès**. Contrairement au flux de code d’autorisation, le flux d’octroi implicite ne retourne pas de **jeton d’actualisation**.

![Flux implicite des applications monopages](./media/scenarios/spa-app.svg)

Ce flux d’authentification n’inclut pas de scénarios d’application utilisant des frameworks JavaScript multiplateformes, tels qu’Electron et React-Native. Ils exigent davantage de fonctionnalités pour l’interaction avec les plateformes natives.

## <a name="specifics"></a>Spécificités

Afin d’activer ce scénario pour votre application, vous avez besoin des éléments suivants :

* Inscription de l’application auprès d’Azure Active Directory (Azure AD). Les étapes d’inscription diffèrent entre le flux d’octroi implicite et le flux de code d’autorisation.
* Configuration de l’application avec les propriétés de l’application inscrite, comme l’ID d’application.
* Utilisation de la bibliothèque d’authentification Microsoft pour JavaScript (MSAL.js) pour permettre au flux d’authentification de se connecter et d’acquérir des jetons.

## <a name="recommended-reading"></a>Lectures recommandées

[!INCLUDE [recommended-topics](../../../includes/active-directory-develop-scenarios-prerequisites.md)]

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Inscription d’application](scenario-spa-app-registration.md).
