---
title: Résoudre les problèmes lors de l’utilisation du déclencheur Azure Functions pour Cosmos DB
description: Problèmes courants, solutions de contournement et procédures de diagnostic relatifs à l’utilisation du déclencheur Azure Functions pour Cosmos DB
author: ealsur
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 03/13/2020
ms.author: maquaran
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: 9fc5da214a50cb000d2154d08bb9b6f6f98ac5ec
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93340527"
---
# <a name="diagnose-and-troubleshoot-issues-when-using-azure-functions-trigger-for-cosmos-db"></a>Diagnostiquer et résoudre les problèmes lors de l’utilisation du déclencheur Azure Functions pour Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article aborde les problèmes courants, solutions de contournement et procédures de diagnostic relatifs à l’utilisation du [déclencheur Azure Functions pour Cosmos DB](change-feed-functions.md).

## <a name="dependencies"></a>Les dépendances

Les liaisons et le déclencheur Azure Functions pour Cosmos DB dépendent des packages d’extension plutôt que du runtime Azure Functions de base. Mettez systématiquement ces packages à jour, car ils peuvent comprendre des correctifs et de nouvelles fonctionnalités capables de résoudre les problèmes éventuels que vous pouvez rencontrer :

* Pour Azure Functions V2, consultez [Microsoft.Azure.WebJobs.Extensions.CosmosDB](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.CosmosDB).
* Pour Azure Functions V1, consultez [Microsoft.Azure.WebJobs.Extensions.DocumentDB](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.DocumentDB).

Sauf indication explicite, le runtime mentionné dans cet article fait toujours référence à Azure Functions V2.

## <a name="consume-the-azure-cosmos-db-sdk-independently"></a>Utiliser le Kit de développement logiciel (SDK) Azure Cosmos DB de manière indépendante

La prise en charge des liaisons et du déclencheur Azure Functions pour Cosmos DB représente la principale fonctionnalité de ce package d’extension. Celui-ci comprend également le [Kit de développement logiciel (SDK) Azure Cosmos DB .NET](sql-api-sdk-dotnet-core.md), ce qui est utile si vous souhaitez interagir par programmation avec Azure Cosmos DB sans utiliser de déclencheurs et de liaisons.

Si souhaitez utiliser le Kit de développement logiciel (SDK) Azure Cosmos DB, assurez-vous de n’ajouter aucune autre référence de package NuGet à votre projet. Au lieu de cela, **laissez la référence du Kit de développement logiciel (SDK) résoudre l’ensemble du package d’extension d’Azure Functions**. Utiliser le Kit de développement logiciel (SDK) Azure Cosmos DB indépendamment des déclencheurs et des liaisons

En outre, si vous créez manuellement votre propre instance du [client de Kit de développement logiciel (SDK) Azure Cosmos DB](./sql-api-sdk-dotnet-core.md), vous devez suivre le modèle en utilisant une seule instance du client [à l’aide d’une approche par modèle Singleton](../azure-functions/manage-connections.md#documentclient-code-example-c) . Ce processus vous permettra d’éviter les problèmes de socket éventuels lors de vos opérations.

## <a name="common-scenarios-and-workarounds"></a>Scénarios courants et solutions de contournement

### <a name="azure-function-fails-with-error-message-collection-doesnt-exist"></a>La fonction Azure échoue et le message d’erreur « La collection n’existe pas » s’affiche

La fonction Azure échoue et le message d’erreur suivant s’affiche : « La collection de source "nom-de-la-collection" ("nom-de-la-base-de-données" pour une base de données) ou la collection de baux "nom-de-la-collection2" ("nom-de-la-base-de-données2" pour une base de données) n’existe pas. Les deux collections doivent exister avant le démarrage de l’écouteur. Pour créer automatiquement la collection de baux, définissez "CreateLeaseCollectionIfNotExists" sur "true" ».

Cela signifie qu’au moins l’un des deux conteneurs Azure Cosmos nécessaires au fonctionnement du déclencheur n’existent pas ou que la fonction Azure ne peut pas y accéder. **L’erreur indiquera elle-même quel conteneur et quelle base de données Azure Cosmos le déclencheur recherche** en fonction de votre configuration.

1. Vérifiez l’attribut `ConnectionStringSetting` et assurez-vous qu’il **fait référence à un paramètre qui existe dans votre application de fonction Azure**. La valeur de cet attribut ne doit pas être la chaîne de connexion, mais le nom du paramètre de configuration.
2. Vérifiez que `databaseName` et `collectionName` existent dans votre compte Azure Cosmos. Si vous utilisez le remplacement de valeur automatique (à l’aide des modèles `%settingName%`), assurez-vous que le nom du paramètre existe dans votre application de fonction Azure.
3. Si vous ne spécifiez pas une collection `LeaseCollectionName/leaseCollectionName`, la valeur par défaut est « baux ». Vérifiez l’existence de ce type de conteneur. Bien que ce soit facultatif, vous pouvez définir l’attribut `CreateLeaseCollectionIfNotExists` dans votre déclencheur sur `true` pour le créer automatiquement.
4. Assurez-vous que la [configuration du pare-feu de votre compte Azure Cosmos](how-to-configure-firewall.md) ne bloque pas la fonction Azure.

### <a name="azure-function-fails-to-start-with-shared-throughput-collection-should-have-a-partition-key"></a>Le démarrage de la fonction Azure échoue et le message d’erreur « La collection de débit partagé doit avoir une clé de partition » s’affiche

Dans les versions précédentes, l’extension Azure Cosmos DB ne prenait pas en charge l’utilisation de conteneurs de baux créés au sein d’une [base de données de débit partagé](./set-throughput.md#set-throughput-on-a-database). Pour résoudre ce problème, mettez à jour l’extension [Microsoft.Azure.WebJobs.Extensions.CosmosDB](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.CosmosDB) pour obtenir la dernière version.

### <a name="azure-function-fails-to-start-with-partitionkey-must-be-supplied-for-this-operation"></a>Le démarrage de la fonction Azure échoue et le message d’erreur « PartitionKey doit être fourni pour cette opération » s’affiche

Cette erreur signifie que vous utilisez actuellement une collection de baux partitionnée avec une [dépendance à une ancienne extension](#dependencies). Passez à la dernière version disponible. Si vous utilisez actuellement Azure Functions v1, vous devrez effectuer une mise à niveau vers Azure Functions v2.

### <a name="azure-function-fails-to-start-with-the-lease-collection-if-partitioned-must-have-partition-key-equal-to-id"></a>Le démarrage de la fonction Azure échoue et le message d’erreur « Si elle est partitionnée, la collection de baux doit avoir une clé de partition correspondant à id » s’affiche.

Cette erreur signifie que votre conteneur de baux actuel est partitionné et que le chemin d’accès de la clé de partition n’est pas `/id`. Pour résoudre ce problème, vous devez recréer le conteneur de baux en utilisant `/id` comme clé de partition.

### <a name="you-see-a-value-cannot-be-null-parameter-name-o-in-your-azure-functions-logs-when-you-try-to-run-the-trigger"></a>Lorsque vous essayez d’exécuter le déclencheur, le message « La valeur ne peut pas être Null. Nom du paramètre : o » s’affiche dans vos journaux Azure Functions.

Ce problème se produit si vous utilisez le portail Azure et que vous essayez de sélectionner le bouton **Exécuter** sur l’écran lors de l’inspection d’une fonction Azure qui utilise le déclencheur. Il n’est pas nécessaire de sélectionner Exécuter pour démarrer le déclencheur. Il démarrera automatiquement lorsque la fonction Azure sera déployée. Si vous souhaitez vérifier le flux de journaux de la fonction Azure sur le portail Azure, il vous suffit d’accéder à votre conteneur surveillé et d’insérer de nouveaux éléments, puis vous verrez votre déclencheur s’exécuter automatiquement.

### <a name="my-changes-take-too-long-to-be-received"></a>La réception de mes modifications prend trop de temps

Il existe plusieurs causes possibles pour ce scénario et vous devez vérifier chacune d’entre elles :

1. Votre fonction Azure est-elle déployée dans la même région que votre compte Azure Cosmos ? Pour une latence réseau optimale, la fonction Azure et votre compte Azure Cosmos doivent être colocalisés dans la même région Azure.
2. Les modifications se produisant dans votre conteneur Azure Cosmos sont-elles continues ou sporadiques ?
Dans le deuxième cas, il peut y avoir un délai entre le stockage de vos modifications et leur récupération par la fonction Azure. En effet, en interne, lorsque le déclencheur recherche les modifications dans votre conteneur Azure Cosmos et n’en trouve aucune en attente de lecture, il se met en veille pendant une durée configurable (par défaut, cinq secondes) avant de recommencer à chercher de nouvelles modifications (ce qui permet d’éviter une consommation trop élevée d’unités de requête). Vous pouvez configurer la durée de veille à l’aide du paramètre `FeedPollDelay/feedPollDelay` dans la [configuration](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md#configuration) de votre déclencheur (la valeur indiquée est en millisecondes).
3. Votre conteneur Azure Cosmos peut avoir une [limitation de débit](./request-units.md).
4. Vous pouvez utiliser l’attribut `PreferredLocations` dans votre déclencheur pour spécifier une liste de régions Azure séparée par des virgules permettant de définir un ordre de connexion privilégié et personnalisé.

### <a name="some-changes-are-repeated-in-my-trigger"></a>Certaines modifications se répètent dans mon déclencheur

Le concept de « modification » correspond à une opération sur un document. Voici les scénarios les plus courants dans lesquels des événements sont reçus pour le même document :
* Le compte utilise la cohérence à terme. Si le flux de modification est consommé au niveau de cohérence à terme, il peut se produire des événements en double entre les opérations de lecture de flux de modification ultérieures (le dernier événement d’une opération de lecture apparaît comme le premier événement de la suivante).
* Le document est en cours de mise à jour. Le flux de modification peut contenir plusieurs opérations pour le même document ; si celui-ci reçoit des mises à jour, il peut sélectionner plusieurs événements (un par mise à jour). Un moyen simple de faire la distinction entre les différentes opérations d’un même document consiste à effectuer le suivi de la propriété `_lsn` [pour chaque modification](change-feed.md#change-feed-and-_etag-_lsn-or-_ts). Si elle n’est pas identique, il s’agit de modifications différentes du même document.
* Si vous identifiez des documents par `id` uniquement, n’oubliez pas que l’identificateur unique d’un document est son `id` et sa clé de partition (il peut exister deux documents disposant du même `id` mais possédant une clé de partition différente).

### <a name="some-changes-are-missing-in-my-trigger"></a>Il manque certaines modifications dans mon déclencheur

Si vous découvrez que certaines modifications dans votre conteneur Azure Cosmos ne sont pas récupérées par la fonction Azure, vous devez suivre une étape pour inspecter en amont.

Lorsque votre fonction Azure reçoit les modifications, elle les traite souvent et peut éventuellement envoyer les résultats à une autre destination. Lorsque vous recherchez des modifications manquantes, assurez-vous de **mesurer les modifications qui sont reçues au niveau du point d’ingestion** (au démarrage de la fonction Azure) et non à la destination.

S’il manque des modifications à la destination, cela peut signifier que des erreurs se produisent lors de l’exécution de la fonction Azure après la réception des modifications.

Dans ce scénario, la meilleure méthode consiste à ajouter des blocs `try/catch` dans votre code et à l’intérieur de boucles susceptibles de traiter les modifications afin de détecter tout échec au niveau d’un sous-ensemble d’éléments particulier, puis de s’en occuper de manière appropriée (envoyez-les vers un autre stockage pour les analyser davantage ou réessayez). 

> [!NOTE]
> Par défaut, le déclencheur Azure Functions pour Cosmos DB n’essaiera pas de traiter à nouveau les modifications si une exception non prise en charge est survenue lors de l’exécution du code. Cela signifie que les modifications ne sont pas arrivées à destination en raison d’une erreur lors de leur traitement.

Si vous découvrez que votre déclencheur n’a pas reçu du tout certaines modifications, le scénario le plus courant est qu’il existe une **autre fonction Azure en cours d’exécution** . Il peut s’agir d’une autre fonction Azure déployée dans Azure ou exécutée en local sur la machine d’un développeur qui a **exactement la même configuration** (conteneurs surveillés et de baux identiques). Dans ce dernier cas, cette fonction Azure vole un sous-ensemble des modifications que votre fonction Azure est censée traiter.

En outre, il est possible de confirmer ce scénario si vous connaissez le nombre d’instances d’application de fonction Azure en cours d’exécution dont vous disposez. Si vous examinez votre conteneur de baux et comptez le nombre d’éléments de bail qu’il contient, les valeurs distinctes de la propriété `Owner` devraient correspondre au nombre d’instances de votre application de fonction. S’il existe plus de propriétaires que d’instances Azure Function App, cela signifie que ce sont ces propriétaires supplémentaires qui « volent » les modifications.

Un moyen simple de contourner cette situation consiste à appliquer un préfixe `LeaseCollectionPrefix/leaseCollectionPrefix` à votre fonction avec une nouvelle ou une autre valeur. Vous pouvez également la tester en utilisant un nouveau conteneur de baux.

### <a name="need-to-restart-and-reprocess-all-the-items-in-my-container-from-the-beginning"></a>Je dois redémarrer et traiter à nouveau tous les éléments de mon conteneur à partir du début 
Pour traiter à nouveau tous les éléments d’un conteneur à partir du début :
1. Arrêtez votre fonction Azure si elle est en cours d’exécution. 
1. Supprimez les documents de la collection de baux (ou supprimez et recréez la collection de baux pour qu’elle soit vide)
1. Affectez la valeur True à l’attribut [StartFromBeginning](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md#configuration) CosmosDBTrigger dans votre fonction. 
1. Redémarrez la fonction Azure. Elle va maintenant lire et traiter toutes les modifications depuis le début. 

La définition de [StartFromBeginning](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md#configuration) sur True indique à la fonction Azure de commencer à lire les modifications à partir du début de l’historique de la collection au lieu de l’heure actuelle. Cela ne fonctionne que si aucun bail n’a déjà été créé (c’est-à-dire s’il n’y a pas de documents dans la collection de baux). L’affectation de la valeur True à cette propriété lorsque des baux ont déjà été créés n’a aucun effet. Dans ce scénario, lorsqu’une fonction est arrêtée et redémarrée, elle commence la lecture à partir du dernier point de contrôle, tel que défini dans la collection de baux. Pour effectuer un nouveau traitement à partir du début, suivez les étapes 1 à 4 ci-dessus.  

### <a name="binding-can-only-be-done-with-ireadonlylistdocument-or-jarray"></a>Une liaison peut uniquement être établie avec IReadOnlyList\<Document> ou JArray

Cette erreur se produit si votre projet Azure Functions (ou tout autre projet référencé) contient une référence NuGet manuelle au Kit de développement logiciel (SDK) Azure Cosmos DB et qu’elle présente une version différente de celle fournie par [l’extension Azure Functions Cosmos DB](./troubleshoot-changefeed-functions.md#dependencies).

Pour contourner cette situation, supprimez la référence NuGet manuelle qui a été ajoutée et laissez la référence au kit SDK Azure Cosmos DB se résoudre à travers le package de l’extension Azure Functions Cosmos DB.

### <a name="changing-azure-functions-polling-interval-for-the-detecting-changes"></a>Modification de l’intervalle d’interrogation de Fonction Azure pour la détection des modifications

Comme expliqué précédemment pour [La réception de mes modifications prend trop de temps](./troubleshoot-changefeed-functions.md#my-changes-take-too-long-to-be-received), Azure Function reste en veille pendant un laps de temps configurable (5 secondes, par défaut) avant de vérifier la présence de nouvelles modifications (pour éviter une consommation élevée de RU). Vous pouvez configurer la durée de veille à l’aide du paramètre `FeedPollDelay/feedPollDelay` dans la [configuration](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md#configuration) de votre déclencheur (la valeur indiquée est en millisecondes).

## <a name="next-steps"></a>Étapes suivantes

* [Activer la supervision dans vos applications Azure Functions](../azure-functions/functions-monitoring.md)
* [Résolution des problèmes relatifs au Kit de développement logiciel (SDK) .NET Azure Cosmos DB](./troubleshoot-dot-net-sdk.md)
