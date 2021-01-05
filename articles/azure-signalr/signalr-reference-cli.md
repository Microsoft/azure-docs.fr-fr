---
title: Exemples Azure CLI - Azure SignalR Service
description: Suivez des exemples réels pour en savoir plus sur Azure CLI pour Azure SignalR Service. Vous allez apprendre à créer un SignalR Service avec d’autres services Azure.
author: sffamily
ms.service: signalr
ms.topic: reference
ms.date: 11/13/2019
ms.author: zhshang
ms.openlocfilehash: 99ebe9c078ef4775e8f0df0545fb0af4b3d1057b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86515238"
---
# <a name="azure-cli-reference"></a>Informations de référence sur Azure CLI

Le tableau suivant contient des liens vers des scripts Bash pour Azure SignalR Service utilisant Azure CLI.

| Script | Descriptions |
|-|-|
|**Créer**||
| [Créer une ressource SignalR Service et un groupe de ressources](scripts/signalr-cli-create-service.md) | Crée une ressource Azure SignalR Service dans un nouveau groupe de ressources avec un nom aléatoire.  |
|**Intégrer**||
| [Créer une ressource SignalR Service et une application web configurée pour utiliser SignalR](scripts/signalr-cli-create-with-app-service.md) | Crée une ressource Azure SignalR Service dans un nouveau groupe de ressources avec un nom aléatoire. Ajoute également une application web et un plan App Service pour héberger une application web ASP.NET Core qui utilise SignalR Service. L’application web est configurée avec un paramètre d’application pour se connecter à la nouvelle ressource SignalR Service. |
| [Créer une ressource SignalR Service et une application web configurée pour utiliser SignalR et GitHub OAuth](scripts/signalr-cli-create-with-app-service-github-oauth.md) | Crée une ressource Azure SignalR Service dans un nouveau groupe de ressources avec un nom aléatoire. Ajoute également ajoute une application web Azure et un plan d’hébergement pour héberger une application web ASP.NET Core qui utilise SignalR Service. L’application web est configurée avec des paramètres d’application pour la chaîne de connexion à la nouvelle ressource SignalR Service et des clés secrètes client pour prendre en charge [l’authentification GitHub](https://developer.github.com/v3/guides/basics-of-authentication/) comme illustré dans le [didacticiel sur l’authentification](signalr-concept-authenticate-oauth.md). L’application web est également configurée pour utiliser une source de déploiement du référentiel git local. |
| | |
