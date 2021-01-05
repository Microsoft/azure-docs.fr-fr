---
title: Vue d’ensemble d’Azure Functions
description: Découvrez comment Azure Functions peut vous aider à générer des applications serverless robustes.
author: craigshoemaker
ms.assetid: 01d6ca9f-ca3f-44fa-b0b9-7ffee115acd4
ms.topic: overview
ms.date: 11/20/2020
ms.author: cshoe
ms.custom: contperf-fy21q2
ms.openlocfilehash: d0641129d641043f10cdb94d9e7f634fb35fbbef
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97032998"
---
# <a name="introduction-to-azure-functions"></a>Introduction à Azure Functions

Azure Functions est une solution serverless qui vous permet d’écrire moins de code, de maintenir une infrastructure plus légère et de réduire les coûts. Au lieu de vous préoccuper du déploiement et de la maintenance de serveurs, l’infrastructure cloud met à votre disposition tous les serveurs à jour pour assurer l’exécution de vos applications.

Vous vous concentrez sur les parties de code les plus importantes pour vous, et Azure Functions s’occupe du reste.<br /><br />

> [!VIDEO https://www.youtube.com/embed/8-jz5f_JyEQ]

Nous élaborons souvent des systèmes pour réagir à une série d’événements critiques. Que ce soit pour générer une API web, répondre à des modifications de base de données, traiter des flux de données IoT ou même gérer des files d’attente de messages, chaque application a besoin d’un moyen d’exécuter du code à mesure que ces événements se produisent.

Pour répondre à ce besoin, Azure Functions propose un « calcul à la demande », et ce de deux façons significatives.

Tout d’abord, Azure Functions vous permet d’implémenter la logique de votre système dans des blocs de code immédiatement disponibles. Ces blocs de code sont appelés « fonctions ». Différentes fonctions peuvent s’exécuter chaque fois que vous devez répondre à des événements critiques.

Deuxièmement, quand les demandes augmentent, Azure Functions répond à la demande avec autant de ressources et d’instances de fonction que nécessaire, mais uniquement quand il le faut. Quand les demandes diminuent, les ressources supplémentaires et les instances d’application baissent automatiquement.

D’où proviennent toutes les ressources de calcul ? Azure Functions [fournit le minimum requis de ressources de calcul](./functions-scale.md) pour répondre à la demande de votre application.

Fournir à la demande des ressources de calcul est l’essence même de l’[informatique serverless](https://azure.microsoft.com/solutions/serverless/) dans Azure Functions.

## <a name="scenarios"></a>Scénarios

Dans de nombreux cas, une fonction [s’intègre à un tableau de services cloud](./functions-triggers-bindings.md) pour fournir des implémentations riches en fonctionnalités.

Vous trouverez ci-dessous un ensemble de scénarios courants pour Azure Functions, _mais en aucun cas exhaustif_.

| Pour... | alors... |
| --- | --- |
| **Créer une API web** | Implémentez un point de terminaison pour vos applications web à l’aide du [déclencheur HTTP](./functions-bindings-http-webhook.md) |
| **Traiter des chargements de fichiers** | Exécutez du code quand un fichier est chargé ou modifié dans [Stockage Blob](./functions-bindings-storage-blob.md) |
| **Générer un workflow serverless** | Chaînez une série de fonctions ensemble à l’aide de [fonctions durables](./durable/durable-functions-overview.md) |
| **Répondre à des modifications de base de données** | Exécutez une logique personnalisée lorsqu’un document est créé ou mis à jour dans [Cosmos DB](./functions-bindings-cosmosdb-v2.md) |
| **Exécuter des tâches planifiées** | Exécutez le code aux [heures définies](./functions-bindings-timer.md) |
| **Créer des systèmes de file d’attente de messages fiables** | Traitez les files d’attente de messages en utilisant [Stockage File d’attente](./functions-bindings-storage-queue.md), [Service Bus](./functions-bindings-service-bus.md) ou [Event Hubs](./functions-bindings-event-hubs.md) |
| **Analyser les flux de données IoT** | Collectez et traitez les [données issues d’appareils IoT](./functions-bindings-event-iot.md) |
| **Traiter des données en temps réel** | Utilisez [Functions et SignalR](./functions-bindings-signalr-service.md) pour répondre immédiatement aux données |

Au fur et à mesure que vous créez vos fonctions, vous disposez des options et des ressources suivantes :

- **Utiliser votre langage préféré** : Écrivez des fonctions en [C#, Java, JavaScript, PowerShell ou Python](./supported-languages.md), ou utilisez un [gestionnaire personnalisé](./functions-custom-handlers.md) pour utiliser pratiquement n’importe quel autre langage.

- **Automatiser le déploiement** : En basant sur les outils votre approche de l’utilisation des pipelines externes, une [myriade d’options de déploiement](./functions-deployment-technologies.md) est disponible.

- **Dépanner une fonction** : Utilisez les [outils de surveillance](./functions-monitoring.md) et les [stratégies de test](./functions-test-a-function.md) pour obtenir des insights sur vos applications.

- **Options tarifaires flexibles** : Avec le plan [Consommation](./pricing.md), vous payez uniquement le temps d’exécution de vos fonctions, tandis que les plans [Premium](./pricing.md) et [App Service](./pricing.md) offrent des fonctionnalités pour des besoins spécifiques.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Démarrer avec des leçons, des exemples et des tutoriels interactifs](./functions-get-started.md)
