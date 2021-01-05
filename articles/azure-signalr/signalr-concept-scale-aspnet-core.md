---
title: Mettre à l’échelle ASP.NET Core SignalR avec Azure SignalR
description: Vue d’ensemble de l’utilisation du service Azure SignalR pour mettre à l’échelle des applications ASP.NET Core SignalR.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.custom: devx-track-dotnet
ms.date: 03/01/2019
ms.author: zhshang
ms.openlocfilehash: 90853b5ff769b710c6c95e4f6e62b3a4aa19fadf
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92151086"
---
# <a name="scale-aspnet-core-signalr-applications-with-azure-signalr-service"></a>Mettre à l’échelle des applications ASP.NET Core SignalR avec le service Azure SignalR

## <a name="developing-signalr-apps"></a>Développer des applications SignalR

Il existe actuellement [deux versions](/aspnet/core/signalr/version-differences) de SignalR que vous pouvez utiliser avec vos applications web : SignalR pour ASP.NET et ASP.NET Core SignalR (la plus récente). Le service Azure SignalR est un service géré par Azure qui repose sur ASP.NET Core SignalR.

ASP.NET Core SignalR est une nouvelle version entièrement réécrite, qui par conséquent n’offre pas de compatibilité descendante avec la version antérieure de SignalR. Les API et les comportements sont différents. Le Kit de développement logiciel (SDK) ASP.NET Core SignalR cible .NET Standard ; il est donc utilisable avec .NET Framework. Il oblige toutefois à utiliser les nouvelles API, plutôt que les anciennes. Si vous utilisez SignalR et que vous souhaitez passer à ASP.NET Core SignalR ou au service Azure SignalR, vous devrez modifier votre code de façon à gérer les différences d’API.

Avec le service Azure SignalR, le composant côté serveur d’ASP.NET Core SignalR est hébergé dans Azure. Il est toutefois possible, dans la mesure où la technologie repose sur ASP.NET Core, d’exécuter l’application web proprement dite sur différentes plateformes (Windows, Linux et macOS) tout en conservant un hébergement [Azure App Service](../app-service/overview.md), [IIS](/aspnet/core/host-and-deploy/iis/index), [Nginx](/aspnet/core/host-and-deploy/linux-nginx), [Apache](/aspnet/core/host-and-deploy/linux-apache) ou [Docker](/aspnet/core/host-and-deploy/docker/index). Vous pouvez également utiliser l’auto-hébergement dans votre propre processus.

Si votre application répond aux objectifs suivants : prendre en charge les dernières fonctionnalités pour fournir des mises à jour de contenu en temps réel aux clients, gérer plusieurs plateformes (Azure, Windows, Linux et macOS) et maintenir un hébergement dans différents environnements, la meilleure solution est peut-être d’utiliser le service Azure SignalR.

## <a name="why-not-deploy-signalr-myself"></a>Pourquoi ne pas déployer SignalR soi-même ?

Vous avez toujours la possibilité de déployer votre application web Azure, compatible avec ASP.NET Core SignalR, sous la forme d’un composant principal de votre application web globale.

L’une des principales raisons d’utiliser le service Azure SignalR est la simplicité. Avec le service Azure SignalR, vous n’avez pas besoin de gérer les problèmes de performances, d’évolutivité ou de disponibilité. Le contrat de niveau de service de 99,9 % s’en occupe pour vous.

Par ailleurs, WebSocket est généralement la technique privilégiée pour prendre en charge des mises à jour de contenu en temps réel. Toutefois, plus le nombre de connexions WebSocket persistantes augmente, plus l’équilibrage de charge devient problématique. Il existe différentes solutions : l’équilibrage de charge DNS, les équilibreurs de charge matériels et l’équilibrage de charge logiciel. Le service Azure SignalR gère ce problème à votre place.

Il se peut également que vous n’ayez pas du tout besoin d’héberger réellement une application web. La logique de votre application web peut tirer parti de [l’informatique sans serveur](https://azure.microsoft.com/overview/serverless-computing/). Par exemple, votre code n’est hébergé et ne s’exécute qu’à la demande avec des déclencheurs [Azure Functions](../azure-functions/index.yml). Ce scénario peut être complexe, car votre code ne conserve pas longtemps les connexions avec les clients. Le service Azure SignalR est en mesure de gérer cette situation, car il gère déjà les connexions à votre place. Pour plus d’informations, consultez la [vue d’ensemble de l’utilisation du service SignalR avec Azure Functions](signalr-concept-azure-functions.md).

## <a name="how-does-it-scale"></a>Comment fonctionne la mise à l’échelle ?

Il est courant de mettre à l’échelle SignalR avec SQL Server, Azure Service Bus ou le Cache Azure pour Redis. Le service Azure SignalR gère l’approche de mise à l’échelle à votre place. Si les performances et le coût sont comparables à ces méthodes, il vous épargne la complexité associée à la gestion de ces services. Il suffit de mettre à jour le nombre d’unités du service. Chaque unité prend en charge jusqu’à 1 000 connexions clientes.

## <a name="next-steps"></a>Étapes suivantes

* [Guide de démarrage rapide : Créer une salle de conversation avec Azure SignalR](signalr-quickstart-dotnet-core.md)