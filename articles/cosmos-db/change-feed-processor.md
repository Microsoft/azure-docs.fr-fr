---
title: Processeur de flux de modification dans Azure Cosmos DB
description: Découvrez comment utiliser le processeur de flux de modification Azure Cosmos DB pour lire le flux de modification et les composants du processeur de flux de modification.
author: timsander1
ms.author: tisande
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 10/12/2020
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: 409b51682700a8b13b2840f171642bdcbee6f6d2
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93340224"
---
# <a name="change-feed-processor-in-azure-cosmos-db"></a>Processeur de flux de modification dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Le processeur de flux de modification fait partie du [kit de développement logiciel Azure Cosmos DB V3](https://github.com/Azure/azure-cosmos-dotnet-v3). Elle simplifie le processus de lecture du flux de modification et répartit efficacement le traitement des événements sur plusieurs consommateurs.

Le principal avantage de la bibliothèque du processeur de flux de modification est son comportement à tolérance de panne qui garantit une livraison « au moins une fois » de tous les événements dans le flux de modification.

## <a name="components-of-the-change-feed-processor"></a>Composants du processeur de flux de modification

Il existe quatre composants principaux dans l’implémentation du processeur de flux de modification :

1. **Conteneur supervisé :** le conteneur supervisé est constitué des données à partir desquelles le flux de modification est généré. Toutes les insertions et les mises à jour apportées au conteneur supervisé sont répercutées dans le flux de modification du conteneur.

1. **Conteneur de baux :** Le conteneur de baux fait office d’entrepôt d’état et coordonne le traitement du flux de modification entre plusieurs rôles de travail. Le conteneur de baux peut être stocké dans le même compte que le conteneur surveillé ou dans un compte distinct.

1. **L’hôte :** Un hôte est une instance d’application qui utilise le processeur de flux de modification pour écouter les modifications. Plusieurs instances avec la même configuration de bail peuvent s’exécuter en parallèle, mais chaque instance doit avoir un **nom d’instance** différent.

1. **Le délégué :** Le délégué est le code qui définit ce que vous, le développeur, souhaitez faire avec chaque lot de modifications que le processeur de flux de modification lit. 

Pour mieux comprendre comment ces quatre éléments du processeur de flux de modification interagissent, examinons un exemple à l’aide du schéma suivant. Le conteneur surveillé stocke les documents et utilise « City » comme clé de partition. Nous voyons que les valeurs de clé de partition sont distribuées dans des plages qui contiennent des éléments. Il existe deux instances d’hôte et le processeur de flux de modification affecte différentes plages de valeurs de clés de partition à chaque instance pour optimiser la distribution de calcul. Chaque plage est lue en parallèle et sa progression est gérée séparément des autres plages dans le conteneur de baux.

:::image type="content" source="./media/change-feed-processor/changefeedprocessor.png" alt-text="Exemple de processeur de flux de modification" border="false":::

## <a name="implementing-the-change-feed-processor"></a>Implémentation du processeur de flux de modification

Le point d’entrée est toujours le conteneur surveillé, à partir d’une instance `Container` que vous appelez `GetChangeFeedProcessorBuilder` :

[!code-csharp[Main](~/samples-cosmosdb-dotnet-change-feed-processor/src/Program.cs?name=DefineProcessor)]

Le premier paramètre est un nom distinct qui décrit l’objectif de ce processeur et le deuxième nom est l’implémentation de délégué qui gérera les modifications. 

Voici un exemple de délégué :


[!code-csharp[Main](~/samples-cosmosdb-dotnet-change-feed-processor/src/Program.cs?name=Delegate)]

Enfin, vous définissez un nom pour cette instance de processeur avec `WithInstanceName` et avec quel conteneur vous souhaitez maintenir l’état du bail `WithLeaseContainer`.

Appeler `Build` vous donnera l’instance de processeur que vous pouvez démarrer en appelant `StartAsync`.

## <a name="processing-life-cycle"></a>Cycle de vie du traitement

Le cycle de vie normal d’une instance d’hôte est le suivant :

1. Lire le flux de modification.
1. Si aucune modification n’est apportée, veillez pendant un intervalle de temps prédéfini (personnalisable avec `WithPollInterval` dans le générateur) et accédez à #1.
1. En cas de modifications, envoyez-les au **délégué**.
1. Lorsque le délégué finit de traiter **correctement** les modifications, mettez à jour le magasin de baux avec le dernier point traité dans le temps et accédez à #1.

## <a name="error-handling"></a>Gestion des erreurs

Le processeur de flux de modification résiste aux erreurs de code utilisateur. Cela signifie que si votre implémentation de délégué a une exception non gérée (étape #4), le thread qui traite ce lot de modifications particulier sera arrêté et un nouveau thread sera créé. Le nouveau thread vérifie le dernier point dans le temps du magasin de baux pour cette plage de valeurs de clé de partition, et redémarre à partir de là, en envoyant efficacement le même lot de modifications au délégué. Ce comportement se poursuit jusqu’à ce que votre délégué traite correctement les modifications et c’est la raison pour laquelle le processeur de flux de modification a une garantie « au moins une fois », car si le code de délégué lève une exception, il relance ce lot.

Pour empêcher votre processeur de flux de modification de « se bloquer » en relançant continuellement le même lot de modifications, vous devez ajouter une logique dans votre code de délégué pour écrire des documents, en cas d’exception, dans une file d’attente de lettres mortes. Cette conception garantit que vous pouvez effectuer le suivi des modifications non traitées tout en continuant à traiter les futures modifications. La file d’attente de lettres mortes peut être un autre conteneur Cosmos. Le magasin de données exact n’a pas d’importance. Il suffit juste que les modifications non traitées soient conservées.

En outre, vous pouvez utiliser l’[estimateur de flux de modification](how-to-use-change-feed-estimator.md) pour surveiller la progression de vos instances de processeur de flux de modification au fur et à mesure qu’elles lisent le flux de modification. Vous pouvez utiliser cette estimation pour déterminer si votre processeur de flux de modification est « bloqué » ou en retard en raison des ressources disponibles, telles que l’UC, la mémoire et la bande passante réseau.

## <a name="deployment-unit"></a>Unité de déploiement

Une unité de déploiement du processeur de flux de modification unique se compose d’une ou de plusieurs instances avec la même configuration de conteneur de `processorName` et de bail. Vous pouvez avoir de nombreuses unités de déploiement pour lesquelles chacune d’elles a un flux d’entreprise différent pour les modifications et chaque unité de déploiement composée d’une ou de plusieurs instances. 

Par exemple, vous pouvez avoir une unité de déploiement qui déclenche une API externe à chaque fois qu’une modification est apportée à votre conteneur. Une autre unité de déploiement peut déplacer des données, en temps réel, chaque fois qu’une modification est apportée. Lorsqu’une modification est apportée dans votre conteneur analysé, toutes vos unités de déploiement sont notifiées.

## <a name="dynamic-scaling"></a>Mise à l’échelle dynamique

Comme mentionné précédemment, vous pouvez avoir une ou plusieurs instances au sein d’une unité de déploiement. Pour tirer parti de la distribution de calcul au sein de l’unité de déploiement, les seules exigences principales sont les suivantes :

1. Toutes les instances doivent avoir la même configuration de conteneur de baux.
1. Toutes les instances doivent avoir le même `processorName`.
1. Chaque instance doit avoir un nom d’instance différent (`WithInstanceName`).

Si ces trois conditions s’appliquent, le processeur de flux de modification utilise un algorithme de distribution égal, répartit tous les baux dans le conteneur de baux sur toutes les instances en cours de cette unité de déploiement et parallélise le calcul. Un bail ne peut appartenir qu’à une seule instance à la fois, ce qui signifie que le nombre maximal d’instances est égal au nombre de baux.

Le nombre d’instances peut croître et diminuer, et le processeur de flux de modification ajuste la charge de façon dynamique en la redistribuant en conséquence.

En outre, le processeur de flux de modification peut s’ajuster de manière dynamique à l’échelle des conteneurs en raison des augmentations de débit ou de stockage. Lorsque votre conteneur croît, le processeur de flux de modification gère en toute transparence ces scénarios en augmentant les baux de manière dynamique et en distribuant les nouveaux baux aux instances existantes.

## <a name="change-feed-and-provisioned-throughput"></a>Flux de modification et débit provisionné

Les opérations de lecture du flux de modification sur le conteneur analysé consomment des unités de requête. 

Les opérations sur le conteneur de baux consomment des unités de requête. Plus le nombre d’instances utilisant le même conteneur de baux est élevé, plus la consommation potentielle d’unités de requête sera importante. N’oubliez pas de surveiller votre consommation d’unités de requête sur le conteneur de baux si vous décidez de mettre à l’échelle et d’augmenter le nombre d’instances.

## <a name="starting-time"></a>Heure de début

Par défaut, lorsqu’un processeur de flux de modification démarre pour la première fois, il initialise le conteneur de baux et démarre son [cycle de vie de traitement](#processing-life-cycle). Toutes les modifications survenues dans le conteneur analysé avant la première initialisation du processeur de flux de modification ne seront pas détectées.

### <a name="reading-from-a-previous-date-and-time"></a>Lecture à partir d’une date et d’une heure précédentes

Il est possible d’initialiser le processeur de flux de modification pour lire les modifications à partir d’une **date et d’une heure spécifiques** , en passant une instance `DateTime` dans l’extension de générateur `WithStartTime` :

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=TimeInitialization)]

Le processeur de flux de modification sera initialisé pour cette date et cette heure spécifiques et commencera à lire les modifications survenues en amont.

### <a name="reading-from-the-beginning"></a>Lecture à partir du début

Dans d’autres scénarios tels que la migration de données ou l’analyse de l’intégralité de l’historique d’un conteneur, nous devons lire le flux de modification à partir du **début de la durée de vie de ce conteneur**. Pour ce faire, nous pouvons utiliser `WithStartTime` sur l’extension du générateur, mais en passant `DateTime.MinValue.ToUniversalTime()`, ce qui génère la représentation UTC de la valeur `DateTime` minimale, comme ci-après :

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=StartFromBeginningInitialization)]

Le processeur de flux de modification est initialisé et commence à lire les modifications à partir du début de la durée de vie du conteneur.

> [!NOTE]
> Ces options de personnalisation ne fonctionnent que pour configurer le point de départ dans le temps du processeur de flux de modification. Une fois que le conteneur de baux a été initialisé pour la première fois, les modifier n’a plus aucun effet.

## <a name="where-to-host-the-change-feed-processor"></a>Où héberger le processeur de flux de modification

Le processeur de flux de modification peut être hébergé sur n’importe quelle plateforme prenant en charge des processus ou des tâches de longue durée :

* Un [Azure WebJob](/learn/modules/run-web-app-background-task-with-webjobs/) exécuté en continu.
* Un processus sur une [machine virtuelle Azure](/azure/architecture/best-practices/background-jobs#azure-virtual-machines).
* Un travail en arrière-plan dans [Azure Kubernetes Service](/azure/architecture/best-practices/background-jobs#azure-kubernetes-service).
* Un [service hébergé ASP.NET](/aspnet/core/fundamentals/host/hosted-services).

Si le processeur de flux de modification peut s’exécuter dans des environnements à courte durée de vie, étant donné que le conteneur de baux conserve l’état, le cycle de démarrage de ces environnements ajoutera du retard à la réception des notifications (en raison de la surcharge liée au démarrage du processeur chaque fois que l’environnement est démarré).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Azure Cosmos DB SDK](sql-api-sdk-dotnet.md)
* [Exemple d’application complet sur GitHub](https://github.com/Azure-Samples/cosmos-dotnet-change-feed-processor)
* [Exemples d’utilisation supplémentaires sur GitHub](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed)
* [Labos Ateliers Cosmos DB pour le processeur de flux de modification](https://azurecosmosdb.github.io/labs/dotnet/labs/08-change_feed_with_azure_functions.html#consume-cosmos-db-change-feed-via-the-change-feed-processor)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le processeur de flux de modification, consultez les articles suivants :

* [Présentation du flux de modification](change-feed.md)
* [Modèle de tirage (pull) du flux de modification](change-feed-pull-model.md)
* [Guide pratique pour effectuer une migration à partir de la bibliothèque du processeur de flux de modification](how-to-migrate-from-change-feed-library.md)
* [Utilisation de l’estimateur de flux de modification](how-to-use-change-feed-estimator.md)
* [Heure de début du processeur de flux de modification](#starting-time)