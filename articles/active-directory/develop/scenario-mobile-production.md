---
title: Préparer des API web appelant des applications mobiles pour la production | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment générer une application mobile qui appelle des API web. (Préparer des applications pour la production.)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.reviewer: jmprieur
ms.custom: aaddev
ms.openlocfilehash: d1a7407c947c1c5bae1eed00acb0f216722f3d09
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88121032"
---
# <a name="prepare-mobile-apps-for-production"></a>Préparer des applications mobiles pour la production

Cet article fournit des détails sur la façon d’améliorer la qualité et la fiabilité de votre application mobile avant de la mettre en production.

## <a name="handle-errors"></a>des erreurs

Lorsque vous préparez une application mobile pour la production, plusieurs conditions d’erreur peuvent se produire. Les principaux cas que vous allez gérer sont des échecs silencieux et des solutions de secours pour l’interaction. Vous devez aussi tenir compte des situations sans réseau, des interruptions de service, de la configuration requise pour le consentement administrateur et d’autres cas spécifiques du scénario.

Pour chaque type de Bibliothèque d’authentification Microsoft (MSAL), vous pouvez trouver des exemples de code et du contenu wiki décrivant comment gérer les conditions d’erreur :

- [Wiki Android MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-android)
- [Wiki iOS MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-objc/wiki)
- [Wiki MSAL.NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki)

## <a name="mitigate-and-investigate-issues"></a>Atténuer et examiner les problèmes

Pour mieux diagnostiquer des problèmes dans votre application, collectez des données. Pour plus d’informations sur les types de données que vous pouvez collecter, voir [Journalisation dans les applications MSAL](./msal-logging.md).

Voici quelques suggestions pour la collecte de données :

- Les utilisateurs peuvent demander de l’aide lorsqu’ils rencontrent des problèmes. Une meilleure pratique consiste à capturer et à stocker temporairement les journaux. Indiquez un emplacement où les utilisateurs peuvent charger les journaux. MSAL fournit des extensions de journalisation permettant de capturer des informations détaillées sur l’authentification.

- Si une télémétrie est disponible, activez-la via MSAL pour recueillir des données sur la façon dont les utilisateurs se connectent à votre application.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [Common steps to move to production](../../../includes/active-directory-develop-scenarios-production.md)]

Pour essayer des exemples supplémentaires, voir [Applications clientes publiques mobiles et de bureau](sample-v2-code.md#desktop-and-mobile-public-client-apps).