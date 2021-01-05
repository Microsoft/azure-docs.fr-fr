---
title: Lire le flux de modification Azure Cosmos DB
description: Cet article décrit les différentes options disponibles pour accéder au flux de modification et le lire dans Azure Cosmos DB.
author: timsander1
ms.author: tisande
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 10/27/2020
ms.reviewer: sngun
ms.openlocfilehash: 7021367e1230573343ddf57ccd399d998ad5280e
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93339272"
---
# <a name="reading-azure-cosmos-db-change-feed"></a>Lire le flux de modification Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Vous pouvez utiliser le flux de modifications Azure Cosmos DB à l’aide d’un modèle d’envoi (push) ou de tirage (pull). Avec un modèle d’envoi (push), le processeur de flux de modification envoie (push) le travail à un client qui dispose d’une logique métier pour traiter ce travail. Toutefois, la complexité liée à la vérification du travail et au stockage de l’état du dernier travail traité est gérée sur le processeur de flux de modification.

Avec un modèle d’extraction (pull), le client doit extraire le travail du serveur. Dans ce cas, le client dispose d’une logique métier pour non seulement traiter le travail, mais également stocker l’état du dernier travail traité, gérer l’équilibrage de charge entre plusieurs clients traitant un travail en parallèle et gérer les erreurs.

Lors de la lecture du flux de modification Azure Cosmos DB, nous vous recommandons généralement d’utiliser un modèle push. De cette façon, vous n’avez pas à vous soucier de ce qui suit :

- Interrogation du flux de modification au sujet de modifications ultérieures.
- Stockage de l’état pour la dernière modification traitée. Lors de la lecture du flux de modification, il est automatiquement stocké dans un [conteneur de bail](change-feed-processor.md#components-of-the-change-feed-processor).
- Équilibrage de charge sur plusieurs clients consommant des modifications. Exemple : un client ne peut pas suivre le rythme imposé par le traitement des modifications, mais un autre a de la capacité disponible.
- [Gestion des erreurs](change-feed-processor.md#error-handling). Exemple : nouvelle tentative automatique des modifications qui n’ont pas été correctement traitées après une exception non prise en charge dans le code ou un problème réseau temporaire.

La majorité des scénarios utilisant le flux de modification Azure Cosmos DB a recours à l’une des options du modèle push. Toutefois, dans certains scénarios, vous souhaiterez peut-être le contrôle de bas niveau supplémentaire du modèle pull, notamment :

- Lecture des modifications à partir d’une clé de partition particulière
- Contrôle de la vitesse à laquelle votre client reçoit les modifications à traiter
- Lecture unique des données existantes dans le flux de modification (par exemple, pour effectuer une migration de données)

## <a name="reading-change-feed-with-a-push-model"></a>Lecture du flux de modification avec un modèle push

Un modèle push est le moyen le plus simple de lire le flux de modification. Vous pouvez lire le flux de modification avec un modèle push de deux manières : avec les [déclencheurs Azure Functions pour Cosmos DB](change-feed-functions.md) et avec la [bibliothèque du processeur de flux de modification](change-feed-processor.md). Azure Functions utilise le processeur de flux de modification en arrière-plan. Ces deux méthodes sont donc très similaires pour lire le flux de modifications. Imaginez Azure Functions comme une simple plateforme d’hébergement pour le processeur de flux de modification, et non comme une méthode entièrement séparée de lecture du flux de modification.

### <a name="azure-functions"></a>Azure Functions

Azure Functions est l’option la plus simple si vous commencez tout juste à utiliser le flux de modification. Sa simplicité en font également l’option recommandée pour la plupart des cas d’usage liés au flux de modification. Quand vous créez un déclencheur Azure Functions pour Azure Cosmos DB, vous pouvez sélectionner le conteneur auquel vous souhaitez vous connecter, et la fonction Azure se déclenche chaque fois qu’une modification est apportée au conteneur. Dans la mesure où Azure Functions utilise le processeur de flux de modification en arrière-plan, il parallélise automatiquement le traitement des modifications entre les [partitions](partitioning-overview.md) de votre conteneur.

Le développement avec Azure Functions est une expérience simple qui peut être plus rapide que de déployer le processeur de flux de modification par vous-même. Les déclencheurs peuvent être créés à l’aide du portail Azure Functions ou par programmation à l’aide de kits SDK. Visual Studio et VS Code fournissent un accompagnement pour l'écriture des fonctions Azure. Vous pouvez même utiliser l'interface CLI d'Azure Functions pour le développement multiplateforme. Vous pouvez écrire et déboguer le code sur votre bureau, puis déployer la fonction avec un seul clic. Pour en savoir plus, consultez les articles [Traitement de base de données serverless à l’aide d’Azure Functions](serverless-computing-database.md) et [Utilisation du flux de modification avec Azure Functions](change-feed-functions.md).

### <a name="change-feed-processor-library"></a>Bibliothèque du processeur de flux de modification

Le processeur de flux de modification vous donne davantage de contrôle sur le flux de modification et masque toujours une grande partie de la complexité. La bibliothèque du processeur de flux de modification suit le modèle Observateur, dans lequel la bibliothèque appelle votre fonction de traitement. La bibliothèque du processeur de flux de modification recherche automatiquement la présence de modifications et, le cas échéant, les envoie (push) au client. Si vous disposez d’un flux de modification à débit élevé, vous pouvez instancier plusieurs clients pour lire le flux de modification. La bibliothèque du processeur de flux de modification répartit automatiquement la charge entre les différents clients. Vous n’aurez pas à implémenter de logique d’équilibrage de charge sur plusieurs clients ou de logique pour maintenir l’état du bail.

La bibliothèque du processeur de flux de modification garantit une remise « au moins une fois » de toutes les modifications. En d’autres termes, si vous utilisez la bibliothèque du processeur de flux de modification, votre fonction de traitement est appelée avec pour chaque élément dans le flux de modification. En cas d’exception non prise en charge dans la logique métier de votre fonction de traitement, les modifications ayant échoué font l’objet de nouvelles tentatives jusqu’à ce qu’elles soient traitées. Pour empêcher votre processeur de flux de modification de « se bloquer » en relançant continuellement les mêmes modifications, ajoutez une logique dans votre fonction de traitement pour écrire des documents, en cas d’exception, dans une file d’attente de lettres mortes. Découvrez-en plus sur la [gestion des erreurs](change-feed-processor.md#error-handling).

Dans Azure Functions, la recommandation pour gérer les erreurs est la même. Vous devez toujours ajouter une logique dans votre code de délégué pour écrire des documents, en cas d’exception, dans une file d’attente de lettres mortes. Toutefois, en cas d’exception non prise en charge dans votre fonction Azure, la modification à l’origine de l’exception ne fait pas l’objet d’une nouvelle tentative. En cas d’exception non prise en charge dans la logique métier, la fonction Azure passe au traitement de la modification suivante. La fonction Azure ne retente pas d’apporter la modification ayant échoué.

Comme Azure Functions, le développement avec la bibliothèque du processeur de flux de modification est simple. Toutefois, vous êtes responsable du déploiement d’un ou plusieurs hôtes pour le processeur de flux de modification. Un hôte est une instance d’application qui utilise le processeur de flux de modification pour écouter les modifications. Même si Azure Functions offre des fonctionnalités de mise à l’échelle automatique, vous êtes responsable de la mise à l’échelle de vos hôtes. Pour en savoir plus, consultez [Utilisation du processeur de flux de modification](change-feed-processor.md#dynamic-scaling). La bibliothèque du processeur de flux de modification fait partie du [SDK Azure Cosmos DB V3](https://github.com/Azure/azure-cosmos-dotnet-v3).

## <a name="reading-change-feed-with-a-pull-model"></a>Lecture du flux de modification avec un modèle pull

Le [modèle pull du flux de modification](change-feed-pull-model.md) vous permet de consommer le flux de modification à votre rythme. Les modifications doivent être demandées par le client et il n’y a pas d’interrogation automatique pour les modifications. Si vous souhaitez « marquer » de manière définitive la dernière modification traitée (semblable au conteneur de bail du modèle push), vous devez [enregistrer un jeton de continuation](change-feed-pull-model.md#saving-continuation-tokens).

Le modèle pull du flux de modification vous offre un contrôle plus précis du flux de modification. Quand vous lisez le flux de modification avec le modèle pull, trois options s’offrent à vous :

- Lire les modifications pour un conteneur entier
- Lire les modifications pour un [FeedRange](change-feed-pull-model.md#using-feedrange-for-parallelization) spécifique
- Lire les modifications pour une valeur de clé de partition spécifique

Vous pouvez paralléliser le traitement des modifications sur plusieurs clients, tout comme avec le processeur de flux de modification. Toutefois, le modèle pull ne gère pas automatiquement l’équilibrage de charge entre les clients. Quand vous utilisez le modèle pull pour paralléliser le traitement du flux de modification, vous obtenez d’abord une liste de FeedRanges. Un FeedRange s’étend sur une plage de valeurs de clé de partition. Vous devez disposer d’un processus Orchestrator qui obtient les FeedRanges et les répartit entre vos machines. Vous pouvez ensuite utiliser ces FeedRanges pour que plusieurs machines lisent le flux de modification en parallèle.

Il n’existe aucune garantie de remise « au moins une fois » avec le modèle pull. Le modèle pull vous permet de contrôler avec précision comment gérer les erreurs.

> [!NOTE]
> Le modèle pull du flux de modification n’est actuellement disponible qu’en [préversion dans le SDK .NET Azure Cosmos DB](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.15.0-preview). La préversion n’est pas encore disponible pour d’autres versions du SDK.

## <a name="change-feed-in-apis-for-cassandra-and-mongodb"></a>Flux de modification dans les API pour Cassandra et MongoDB

La fonctionnalité de flux de modification apparaît comme des flux de modification dans l’API MongoDB et sous la forme Interroger avec un prédicat dans l’API Cassandra. Pour en savoir plus sur les détails d’implémentation de l’API MongoDB, consultez les [flux de modification dans l’API Azure Cosmos DB pour MongoDB](mongodb-change-streams.md).

Native Apache Cassandra propose la capture des changements de données (CDC), un mécanisme de marquage de tables spécifiques à des fins d’archivage, mais aussi de rejet des écritures dans ces tables une fois que la taille sur disque configurable du journal CDC est atteinte. La fonctionnalité de flux de modification dans l’API Azure Cosmos DB pour Cassandra améliore la capacité à interroger les modifications avec un prédicat par le biais du langage CQL. Pour en savoir plus sur les détails d’implémentation, consultez [Flux de modification dans l’API Azure Cosmos DB pour Cassandra](cassandra-change-feed.md).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le flux de modification, consultez les articles suivants :

* [Présentation du flux de modification](change-feed.md)
* [Utilisation du flux de modification avec Azure Functions](change-feed-functions.md)
* [Utilisation de la bibliothèque du processeur de flux de modification](change-feed-processor.md)
