---
title: Fichier include
description: Fichier include
services: active-directory
author: kalyankrishna1
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: include
ms.date: 11/05/2020
ms.author: kkrishna
ms.openlocfilehash: 174946667885debc348370ef2c6f93206890e9c1
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94628003"
---
Vous pouvez utiliser l’implémentation du cache de jeton MSAL pour permettre aux applications, aux API et aux services en arrière-plan d’utiliser le cache de jeton d’accès afin de continuer à agir au nom des utilisateurs en leur absence. Cela s’avère particulièrement utile si les applications et les services en arrière-plan doivent continuer à travailler au nom de l’utilisateur une fois que celui-ci a quitté l’application web front-end.

Aujourd’hui, la plupart des processus en arrière-plan utilisent des [autorisations d’application](/graph/auth/auth-concepts#microsoft-graph-permissions) lorsqu’ils doivent travailler avec les données d’un utilisateur sans que l’utilisateur soit présent pour l’authentification ou la réauthentification. Étant donné que les autorisations d’application requièrent souvent un consentement de l’administrateur, ce qui nécessite une élévation des privilèges, un conflit inutile est décelé, car le développeur n’avait pas l’intention d’obtenir une autorisation au-delà de celle consentie à l’origine par l’utilisateur pour son application.

Cet exemple de code sur GitHub montre comment éviter ce conflit inutile en accédant au cache de jeton MSAL à partir d’applications en arrière-plan :

 [Accès au cache de jeton de l’utilisateur connecté à partir d’applications, d’API et de services en arrière-plan](https://github.com/Azure-Samples/ms-identity-dotnet-advanced-token-cache)