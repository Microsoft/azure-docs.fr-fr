---
title: 'Tutoriel : Créer une application web sécurisée sur Azure App Service | Azure'
description: Dans ce tutoriel, vous allez apprendre à créer une application web avec Azure App Service, activer l’authentification, appeler le stockage Azure et appeler Microsoft Graph.
services: active-directory, app-service-web, storage, microsoft-graph
author: rwike77
manager: CelesteDG
ms.service: app-service-web
ms.topic: tutorial
ms.workload: identity
ms.date: 11/09/2020
ms.author: ryanwi
ms.reviewer: stsoneff
ms.custom: azureday1
ms.openlocfilehash: 17931ee92072059d94cd950e9ce8ef6588f7e891
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905500"
---
# <a name="tutorial-enable-authentication-in-app-service-and-access-storage-and-microsoft-graph"></a>Tutoriel : Activer l’authentification dans App Service et accéder au stockage et à Microsoft Graph

Ce tutoriel décrit un scénario d’application courant dans lequel vous allez apprendre à :

- [Configurer l’authentification pour une application web](scenario-secure-app-authentication-app-service.md) et limiter l’accès aux utilisateurs de votre organisation. Consultez A dans le diagramme.
- [Accéder de manière sécurisée au stockage Azure](scenario-secure-app-access-storage.md) pour le compte de l’application web à l’aide d’identités managées. Consultez B dans le diagramme.
- Accéder à des données dans Microsoft Graph [pour le compte de l’utilisateur connecté](scenario-secure-app-access-microsoft-graph-as-user.md) ou [pour le compte de l’application web](scenario-secure-app-access-microsoft-graph-as-app.md) à l’aide d’identités managées. Consultez C dans le diagramme.
- [Nettoyer les ressources](scenario-secure-app-clean-up-resources.md) que vous avez créées pour ce tutoriel.

:::image type="content" source="./media/scenario-secure-app-overview/web-app.svg" alt-text="Diagramme illustrant les scénarios d’application dans la plateforme d’identités Microsoft." border="false":::

Pour commencer, découvrez comment activer l’authentification pour une application web.

> [!div class="nextstepaction"]
> [Configurer l’authentification pour une application web](scenario-secure-app-authentication-app-service.md)
