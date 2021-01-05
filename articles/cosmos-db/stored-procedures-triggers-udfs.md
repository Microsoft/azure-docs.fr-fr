---
title: Utiliser des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur dans Azure Cosmos DB
description: 'Cet article présente les concepts suivants : procédures stockées, des déclencheurs et fonctions définies par l’utilisateur dans Azure Cosmos DB.'
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 04/09/2020
ms.author: tisande
ms.reviewer: sngun
ms.openlocfilehash: 0bd572da9bba9048e2c8b9c4b426056620c4c265
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93340700"
---
# <a name="stored-procedures-triggers-and-user-defined-functions"></a>Procédures stockées, déclencheurs et fonctions définies par l’utilisateur
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB offre une exécution transactionnelle, intégrée au langage, de JavaScript. Lorsque vous utilisez l’API SQL dans Azure Cosmos DB, vous pouvez écrire les **procédures stockées** , les **déclencheurs** et les **fonctions définies par l’utilisateur** dans le langage JavaScript. Vous pouvez écrire votre logique dans JavaScript et l’exécuter dans le moteur de base de données. Vous pouvez créer et exécuter des déclencheurs, des procédures stockées et des fonctions définies par l’utilisateur à l’aide du [Portail Azure](https://portal.azure.com/), de [l’API de requête avec langage intégré JavaScript dans Azure Cosmos DB](javascript-query-api.md) ou des [Kits de développement logiciel (SDK) clients de l’API SQL Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md).

## <a name="benefits-of-using-server-side-programming"></a>Avantages de l’utilisation de la programmation côté serveur

L’écriture de procédures stockées, déclencheurs et fonctions définies par l’utilisateur (UDF) dans JavaScript vous permet de créer des applications riches avec les avantages suivants :

* **Logique procédurale :** JavaScript en tant que langage de programmation de haut niveau offre une interface riche et familière permettant d’exprimer la logique métier. Vous pouvez effectuer une séquence d’opérations complexes sur les données.

* **Transactions atomiques :** Les opérations de base de données Azure Cosmos DB effectuées dans un déclencheur ou une procédure stockée sont atomiques. Cette fonctionnalité atomique permet à une application de combiner des applications connexes en un seul lot de façon à ce que toutes les opérations réussissent ou qu’aucune ne réussisse.

* **Performances :** Les données JSON sont mappées de manière intrinsèque au système de type de langage JavaScript. Ce mappage permet plusieurs optimisations telles que la matérialisation différée de documents JSON dans le pool de tampons et leur disponibilité sur demande à l’exécution de code. Il existe d'autres avantages en matière de performances en lien avec l'expédition de la logique métier à la base de données, parmi lesquels :

   * *Traitement par lot :* Vous pouvez regrouper les opérations telles que les insertions et les envoyer en bloc. Les coûts liés à la latence du trafic réseau et la surcharge en matière de stockage pour créer des transactions séparées sont considérablement réduits.

   * *Précompilation :* Les procédures stockées, les déclencheurs et les fonctions définies par l’utilisateur sont précompilés de façon implicite en format de code d’octet afin d’éviter les frais de compilation à chaque appel de script. La précompilation permet de s’assurer que les appels de procédures stockées sont rapides et présentent un encombrement réduit.

   * *Séquencement :* Parfois, les opérations ont besoin d’un mécanisme de déclenchement qui peut exécuter une ou plusieurs mises à jour sur les données. En plus de l’atomicité, il existe également des avantages de performances lors de l’exécution côté serveur.

* **Encapsulation :** Les procédures stockées peuvent être utilisées pour regrouper la logique à un endroit. L’encapsulation ajoute une couche d'abstraction aux données, ce qui vous permet de faire évoluer vos applications indépendamment des données. Cette couche d’abstraction est utile lorsque les données sont sans schéma et que vous n’êtes pas obligé de gérer l’ajout d’une logique supplémentaire directement dans votre application. L’abstraction vous permet d'assurer la sécurité de vos données en simplifiant l'accès à partir des scripts.

> [!TIP]
> Les procédures stockées conviennent parfaitement pour des opérations intenses en écriture et nécessitant une transaction sur une valeur de clé de partition. Lorsque vous décidez d’utiliser des procédures stockées, optimisez en encapsulant la quantité maximale d’écritures possibles. En règle générale, les procédures stockées ne sont pas le moyen le plus efficace pour effectuer un grand nombre d’opérations de lecture ou de requête. L’utilisation de procédures stockées pour traiter par lot un grand nombre de lectures à retourner au client n’offre donc pas l’avantage souhaité. Pour des performances optimales, ces opérations intenses en lecture doivent être effectuées côté client à l’aide du Kit de développement logiciel (SDK) Cosmos. 

## <a name="transactions"></a>Transactions

Une transaction dans une base de données classique peut être définie comme étant une séquence d'opérations effectuées en tant qu'unité de travail logique unique. Chaque transaction offre des **garanties de propriété ACID**. ACID est un acronyme bien connu qui est l’abréviation de : **A** tomicity, **C** onsistency, **I** solation, **D** urability (Atomicité, cohérence, isolation et durabilité). 

* L'atomicité permet de s'assurer que toutes les opérations effectuées au sein d'une transaction sont traitées en tant que simple unité validée dans son intégralité ou aucunement. 

* La cohérence permet de s'assurer que les données sont toujours dans un état valide d'une transaction à l'autre. 

* L'isolation, quant à elle, permet de garantir qu'aucune transaction n'interfère avec les autres. De nombreux systèmes commerciaux fournissent plusieurs niveaux d'isolation pouvant être utilisés en fonction des besoins des applications. 

* La durabilité permet de s'assurer que toute modification validée dans une base de données sera toujours présente.

Dans Azure Cosmos DB, le runtime JavaScript est hébergé dans le moteur de base de données. Par conséquent, les demandes effectuées au sein de procédures stockées et de déclencheurs s'exécutent dans la même étendue qu'une session de base de données. Cette fonctionnalité permet à Azure Cosmos DB de garantir les propriétés ACID pour toutes les opérations qui font partie d’une procédure stockée ou d’un déclencheur. Pour obtenir des exemples, consultez l’article [Guide pratique pour implémenter des transactions](how-to-write-stored-procedures-triggers-udfs.md#transactions).

### <a name="scope-of-a-transaction"></a>Étendue d’une transaction

Les procédures stockées sont associées à un conteneur Azure Cosmos et l’exécution de la procédure stockée est limitée à une clé de partition logique. Les procédures stockées doivent inclure une valeur de clé de partition logique lors de l’exécution qui définit la partition logique pour l’étendue de la transaction. Pour plus d’informations, consultez l’article [Partitionnement dans Azure Cosmos DB](partitioning-overview.md).

### <a name="commit-and-rollback"></a>Validation et restauration

Les transactions sont intégrées de façon native dans le modèle de programmation JavaScript d’Azure Cosmos DB. Dans une fonction JavaScript, toutes les opérations sont automatiquement encapsulées dans une transaction unique. Si la logique JavaScript dans une procédure stockée se termine sans aucune exception, toutes les opérations au sein de la transaction sont validées dans la base de données. Les instructions comme `BEGIN TRANSACTION` et `COMMIT TRANSACTION` (familières aux bases de données relationnelles) sont implicites dans Azure Cosmos DB. En cas d’exceptions dans le script, le runtime JavaScript d’Azure Cosmos DB annule toute la transaction. Par conséquent, lever une exception équivaut en fait à un `ROLLBACK TRANSACTION` dans Azure Cosmos DB.

### <a name="data-consistency"></a>Cohérence des données

Les procédures stockées et les déclencheurs sont toujours exécutés dans le réplica principal d’un conteneur Azure Cosmos. Cette fonctionnalité permet de s'assurer que les lectures à partir des procédures stockées offrent une [cohérence forte](./consistency-levels.md). Les requêtes utilisant des fonctions définies par l’utilisateur peuvent être exécutées sur le réplica principal ou un réplica secondaire. Les procédures stockées et les déclencheurs sont destinés à soutenir les écritures transactionnelles. Pendant ce temps, la logique en lecture seule est mieux implémentée en tant que logique côté application et les requêtes utilisant les [kits SDK d’API SQL Azure Cosmos DB](sql-api-dotnet-samples.md) vous aideront à saturer le débit de base de données. 

> [!TIP]
> Les requêtes exécutées au sein d’une procédure stockée ou d’un déclencheur peuvent ne pas voir les modifications apportées aux éléments par la même transaction de script. Cette instruction s’applique à la fois aux requêtes SQL, telles que `getContent().getCollection.queryDocuments()`, et aux requêtes intégrées au langage, telles que `getContext().getCollection().filter()`.

## <a name="bounded-execution"></a>Exécution limitée

Toutes les opérations Azure Cosmos DB doivent s’effectuer avant le délai d’expiration spécifié. Cette contrainte s’applique aux fonctions JavaScript (procédures stockées, déclencheurs et fonctions définies par l’utilisateur). Si une opération n'est pas terminée dans ce délai imparti, la transaction est annulée.

Vous pouvez vous assurer que les fonctions JavaScript s’exécutent dans ce délai ou mettre en œuvre un modèle basé sur la continuation pour traiter par lots/reprendre l’exécution. Afin de simplifier le développement de procédures stockées et de déclencheurs pour gérer les limites de temps, toutes les fonctions sous le conteneur Azure Cosmos (par exemple, création, lecture, remplacement et suppression d’éléments) retournent une valeur booléenne qui indique si l'opération arrivera à son terme. Si cette valeur est false, cela indique que la procédure doit clôturer l’exécution, car le script consomme plus de temps ou de débit provisionné que la valeur configurée. Les opérations mises en file d'attente avant la première opération de stockage non acceptée sont assurées de s'exécuter si la procédure stockée s'exécute à temps et ne place pas d'autres demandes dans la file d'attente. Par conséquent, les opérations doivent être mises en file d’attente une à la fois à l’aide de la convention de rappel de JavaScript pour gérer le flux de contrôle du script. Étant donné que les scripts sont exécutés dans un environnement côté serveur, ils sont strictement régis. Les scripts enfreignant de façon répétitive les limites de l’exécution peuvent être marqués comme inactifs et ne peuvent pas être exécutés. Ils doivent être recréés pour respecter les limites de l’exécution.

Les fonctions JavaScript sont également soumises à la [capacité de débit provisionné](request-units.md). Les fonctions JavaScript peuvent potentiellement finir par utiliser un grand nombre d’unités de demande en peu de temps et leur débit peut être limité si la limite de capacité de débit provisionné est atteinte. Il est important de noter que les scripts consomment un débit supplémentaire en plus du débit consacré à l’exécution des opérations de base de données, bien que celles-ci soient légèrement moins coûteuses que l’exécution des mêmes opérations à partir du client.

## <a name="triggers"></a>Déclencheurs

Azure Cosmos DB prend en charge deux types de déclencheurs :

### <a name="pre-triggers"></a>Prédéclencheurs

Azure Cosmos DB fournit des déclencheurs qui peuvent être appelés par l’exécution d’une opération sur un élément Azure Cosmos. Par exemple, vous pouvez spécifier un prédéclencheur lorsque vous créez un élément. Dans ce cas, le prédéclencheur sera exécuté avant la création de l’élément. Les pré-déclencheurs ne peuvent pas avoir de paramètres en entrée. Si nécessaire, l’objet de la demande peut être utilisé pour mettre à jour le corps du document à partir de la demande d’origine. Lorsque les déclencheurs sont enregistrés, les utilisateurs peuvent spécifier les opérations avec lesquelles il peut s'exécuter. Si un déclencheur a été créé avec `TriggerOperation.Create`, cela signifie que l’utilisation du déclencheur dans une opération de remplacement n’est pas autorisée. Pour obtenir des exemples, consultez l’article [Guide pratique pour écrire des déclencheurs](how-to-write-stored-procedures-triggers-udfs.md#triggers).

### <a name="post-triggers"></a>Post-déclencheurs

Comme pour les prédéclencheurs, les post-déclencheurs sont également associés à une opération sur un élément Azure Cosmos et ils ne nécessitent aucun paramètre d’entrée. Ils sont exécutés *après* la fin de l’opération et ont accès au message de réponse qui est envoyé au client. Pour obtenir des exemples, consultez l’article [Guide pratique pour écrire des déclencheurs](how-to-write-stored-procedures-triggers-udfs.md#triggers).

> [!NOTE]
> Les déclencheurs inscrits ne s’exécutent pas automatiquement lorsque leurs opérations correspondantes (créer / supprimer / remplacer / mettre à jour) sont effectuées. Ils doivent être appelés explicitement lors de l’exécution de ces opérations. Pour plus d’informations, consultez l’article [Comment exécuter des déclencheurs](how-to-use-stored-procedures-triggers-udfs.md#pre-triggers).

## <a name="user-defined-functions"></a><a id="udfs"></a>Fonctions définies par l’utilisateur

[Les fonctions définies par l'utilisateur](sql-query-udfs.md) permettent d'étendre la syntaxe du langage de requête API SQL et de mettre en œuvre facilement une logique métier personnalisée. Elles ne peuvent être appelées que dans les requêtes. Les fonctions définies par l’utilisateur n'ont pas accès à l'objet de contexte et sont destinées à être utilisées en tant que JavaScript en calcul seul. Par conséquent, elles peuvent être exécutées sur des réplicas secondaires. Pour obtenir des exemples, consultez l’article [Guide pratique pour écrire des fonctions définies par l’utilisateur](how-to-write-stored-procedures-triggers-udfs.md#udfs).

## <a name="javascript-language-integrated-query-api"></a><a id="jsqueryapi"></a>API de requête intégrée au langage JavaScript

En plus de l’émission de requêtes à l’aide de la syntaxe de requête API SQL, le [kit SDK côté serveur](https://azure.github.io/azure-cosmosdb-js-server) vous permet d’effectuer des requêtes à l’aide d’une interface JavaScript sans aucune connaissance de SQL. L’API de requête JavaScript permet de créer programmatiquement des requêtes en passant des fonctions de prédicat dans une séquence d’appels de fonctions. Les requêtes sont analysées par le runtime JavaScript et exécutées efficacement dans Azure Cosmos DB. Pour en savoir plus sur la prise en charge de l’API de requête JavaScript, consultez l’article [Utilisation de l’API de requête intégrée au langage JavaScript](javascript-query-api.md). Pour obtenir des exemples, consultez l’article [Guide pratique pour écrire des procédures stockées et des déclencheurs dans Azure Cosmos DB à l’aide de l’API de requête JavaScript](how-to-write-javascript-query-api.md).

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour découvrir comment écrire et utiliser des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur dans Azure Cosmos DB :

* [Guide pratique pour écrire des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur](how-to-write-stored-procedures-triggers-udfs.md)

* [Guide pratique pour utiliser des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur](how-to-use-stored-procedures-triggers-udfs.md)

* [Utilisation de l’API de requête intégrée au langage JavaScript](javascript-query-api.md)