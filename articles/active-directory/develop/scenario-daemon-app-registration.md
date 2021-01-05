---
title: Inscrire des applications démon appelant des API web - Plateforme d’identités Microsoft | Azure
description: Apprenez à générer une application démon qui appelle des API web - inscription d'application
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/15/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 3b77cc93385efb0bbb8a9b87d29de9bad5cd4ceb
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996001"
---
# <a name="daemon-app-that-calls-web-apis---app-registration"></a>Application démon qui appelle des API web - inscription d'application

Voici ce que vous devez savoir lors de l’inscription d’une application démon.

## <a name="supported-account-types"></a>Types de comptes pris en charge

Les applications démon n’ont de sens que dans les locataires Azure AD. Ainsi, quand vous créez l’application, vous devez choisir l’une des options suivantes :

- **Comptes dans cet annuaire organisationnel uniquement**. Ce choix est le plus fréquent car les applications démon sont généralement écrites par des développeurs métier.
- **Comptes dans un annuaire organisationnel**. ce choix s'applique aux éditeurs de logiciels indépendants qui fournissent un utilitaire à leurs clients. Les administrateurs de locataires de vos clients doivent l’approuver.

## <a name="authentication---no-reply-uri-needed"></a>Authentification : aucun URI de réponse nécessaire

Si votre application cliente confidentielle utilise *uniquement* le flux d’informations d’identification du client, l’URI de réponse n’a pas besoin d’être inscrit. Il n’est pas nécessaire pour la configuration ou la construction de l’application. Le flux d'informations d'identification du client ne l'utilise pas.

## <a name="api-permissions---app-permissions-and-admin-consent"></a>Autorisations d’API : autorisations d’application et consentement administrateur

Une application démon peut uniquement demander des permissions d’application (et non des autorisations déléguées) aux APIs. Dans la page **Autorisation d’API** de l’inscription d’application, après avoir sélectionné **Ajouter une autorisation** et choisi la famille d’API, choisissez **Autorisations d’application**, puis sélectionnez vos autorisations.

![Permissions d'application et consentement administrateur](media/scenario-daemon-app/app-permissions-and-admin-consent.png)

> [!NOTE]
> L’API web que vous souhaitez appeler doit définir des *autorisations d’application (rôles d’application)* , et non des autorisations déléguées. Pour plus d’informations sur la façon d’exposer une telle API, voir [API web protégée : Inscription d’application : quand votre API web est appelée par une application démon](scenario-protected-web-api-app-registration.md#if-your-web-api-is-called-by-a-daemon-app).

Avec les applications démon, un administrateur de locataire doit accorder un consentement préalable à l’application qui appelle l’API web. Les administrateurs de locataires fournissent ce consentement dans la même page **Autorisation d’API** en sélectionnant **Accorder le consentement administrateur à *notre organisation*** .

Si vous êtes un éditeur de logiciels indépendant qui génère une application multilocataire, nous vous recommandons de lire la section [Déploiement : cas d’applications démon multilocataires](scenario-daemon-production.md#deployment---multitenant-daemon-apps).

[!INCLUDE [Pre-requisites](../../../includes/active-directory-develop-scenarios-registration-client-secrets.md)]

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Configuration du code de l’application](./scenario-daemon-app-configuration.md).
