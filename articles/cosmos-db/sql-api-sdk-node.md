---
title: 'Azure Cosmos DB : API SQL Node.js, SDK et ressources'
description: Tout savoir sur l’API et le Kit SDK Node.js SQL, y compris les dates de sortie, les dates de déclassement et les modifications effectuées entre chaque version du kit SDK Node.js Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: nodejs
ms.topic: reference
ms.date: 08/12/2020
ms.author: anfeldma
ms.custom: devx-track-js
ms.openlocfilehash: 7f101097945e5f345bae351ef5ca2a23a3e14be6
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93091155"
---
# <a name="azure-cosmos-db-nodejs-sdk-for-sql-api-release-notes-and-resources"></a>Kit de développement logiciel .Node.js Azure Cosmos DB pour l’API SQL : Notes de publication et ressources
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]
> [!div class="op_single_selector"]
> * [Kit de développement logiciel (SDK) .NET v3](sql-api-sdk-dotnet-standard.md)
> * [SDK .NET v2](sql-api-sdk-dotnet.md)
> * [SDK .NET Core v2](sql-api-sdk-dotnet-core.md)
> * [SDK .NET Change Feed v2](sql-api-sdk-dotnet-changefeed.md)
> * [Node.JS](sql-api-sdk-node.md)
> * [SDK Java v4](sql-api-sdk-java-v4.md)
> * [SDK Java Async v2](sql-api-sdk-async-java.md)
> * [SDK Java Sync v2](sql-api-sdk-java.md)
> * [Spring Data v2](sql-api-sdk-java-spring-v2.md)
> * [Spring Data v3](sql-api-sdk-java-spring-v3.md)
> * [Spark Connector](sql-api-sdk-java-spark.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](/rest/api/cosmos-db/)
> * [API REST Resource Provider](/rest/api/cosmos-db-resource-provider/)
> * [SQL](./sql-query-getting-started.md)
> * [Exécuteur en bloc – .NET v2](sql-api-sdk-bulk-executor-dot-net.md)
> * [Exécuteur en bloc – Java](sql-api-sdk-bulk-executor-java.md)

|Ressource  |Lien  |
|---------|---------|
|Télécharger le SDK  |   [NPM](https://www.npmjs.com/package/@azure/cosmos) 
|Documentation de l’API  |  [Documentation de référence sur le SDK JavaScript](/javascript/api/%40azure/cosmos/?preserve-view=true&view=azure-node-latest)
|Instructions d’installation du SDK  |  [Instructions d’installation](https://github.com/Azure/azure-sdk-for-js)
|Contribuer au SDK | [GitHub](https://github.com/Azure/azure-cosmos-js/tree/master)
| Exemples | [Exemples de code Node.js](sql-api-nodejs-samples.md)
| Tutoriel pour bien démarrer | [Bien démarrer avec le SDK JavaScript](sql-api-nodejs-get-started.md)
| Tutoriel basé sur une application web | [Générer une application web Node.js à l’aide d’Azure Cosmos DB](sql-api-nodejs-application.md)
| Plateforme actuellement prise en charge | [Node.js v12.x](https://nodejs.org/en/blog/release/v12.7.0/) - SDK version 3.x.x<br/>[Node.js v10.x](https://nodejs.org/en/blog/release/v10.6.0/) - SDK version 3.x.x<br/>[Node.js v8.x](https://nodejs.org/en/blog/release/v8.16.0/) - SDK version 3.x.x<br/>[Node.js v6.x](https://nodejs.org/en/blog/release/v6.10.3/) - SDK version 2.x.x<br/>[Node.js v4.2.0](https://nodejs.org/en/blog/release/v4.2.0/) - SDK version 1.x.x<br/> [Node.js v0.12](https://nodejs.org/en/blog/release/v0.12.0/) - SDK version 1.x.x<br/> [Node.js v0.10](https://nodejs.org/en/blog/release/v0.10.0/) - SDK version 1.x.x

## <a name="release-notes"></a>Notes de publication

### <a name="310"></a><a name="3.1.0"></a>3.1.0
* Définition de la valeur ResponseContinuationTokenLimitInKB par défaut sur 1 Ko. Par défaut, nous limitons ce paramètre à 1 Ko pour éviter les en-têtes longs (Node.js a une limite de taille d’en-tête globale). Un utilisateur peut définir ce champ pour autoriser les en-têtes plus longs, ce qui peut aider le back-end à optimiser l’exécution des requêtes.
* Suppression de disableSSLVerification. Cette option dispose de nouvelles alternatives décrites dans l’article [388](https://github.com/Azure/azure-cosmos-js/pull/388).

### <a name="304"></a><a name="3.0.4"></a>3.0.4
* Autorisation d’initialHeaders à définir explicitement un en-tête de clé de partition
* Utilisation de package.json#files pour empêcher la publication de fichiers superflus
* Correction de l’erreur de tri des cartes de routage sur une version de Node antérieure à v8
* Correction du bogue survenant quand l’utilisateur fournit des options de nouvelle tentative partielles

### <a name="303"></a><a name="3.0.3"></a>3.0.3
* Empêcher webpack de résoudre les modules appelés avec require

### <a name="302"></a><a name="3.0.2"></a>3.0.2
* Correction d’un bogue de longue date où les RU (unités de requête) étaient toujours signalé comme 0 pour les requêtes d’agrégation

### <a name="300"></a><a name="3.0.0"></a>3.0.0

🎉 Version v3 ! 🎉 De nombreuses nouvelles fonctionnalités, des corrections de bogues et quelques changements cassants. Objectifs principaux de cette version :

* Implémenter de nouvelles fonctionnalités majeures
  * Requêtes DISTINCT
  * Requêtes LIMIT/OFFSET
  * Requêtes annulables par l’utilisateur
* Effectuer une mise à jour vers la dernière version de l’API REST Cosmos où tous les conteneurs ont une échelle illimitée
* Faciliter l’utilisation de Cosmos à partir du navigateur
* Mieux adhérer aux les nouvelles instructions du SDK Azure JavaScript

#### <a name="migration-guide-for-breaking-changes"></a>Guide de migration pour les changements cassants
##### <a name="improved-client-constructor-options"></a>Amélioration des options du constructeur client

Les options du constructeur ont été simplifiées :

* masterKey a été renommé key et a été déplacée vers le niveau supérieur
* Les propriétés qui se trouvaient précédemment sous options.auth ont été déplacées vers le niveau supérieur

``` js
// v2
const client = new CosmosClient({
    endpoint: "https://your-database.cosmos.azure.com",
    auth: {
        masterKey: "your-primary-key"
    }
})

// v3
const client = new CosmosClient({
    endpoint: "https://your-database.cosmos.azure.com",
    key: "your-primary-key"
})
```

##### <a name="simplified-queryiterator-api"></a>Simplification de l’API QueryIterator
Dans la version v2, il existait de nombreuses façons d’itérer ou de récupérer les résultats d’une requête. Nous avons tenté de simplifier l’API v3 et de supprimer les API similaires ou en double :

* Suppression d’iterator.next() et d’iterator.current(). Utilisez fetchNext() pour obtenir des pages de résultats.
* Suppression d’iterator.forEach(). Utilisez des itérateurs asynchrones à la place.
* Renommage d’iterator.executeNext() en iterator.fetchNext()
* Renommage d’iterator.toArray() en iterator.fetchAll()
* Les pages sont désormais des objets de réponse appropriés plutôt que des objets JavaScript simples
* const container = client.database(dbId).container(containerId)

``` js
// v2
container.items.query('SELECT * from c').toArray()
container.items.query('SELECT * from c').executeNext()
container.items.query('SELECT * from c').forEach(({ body: item }) => { console.log(item.id) })

// v3
container.items.query('SELECT * from c').fetchAll()
container.items.query('SELECT * from c').fetchNext()
for await(const { result: item } in client.databases.readAll().getAsyncIterator()) {
    console.log(item.id)
}
```

##### <a name="fixed-containers-are-now-partitioned"></a>Les conteneurs fixes sont maintenant partitionnés
Le service Cosmos prend désormais en charge les clés de partition sur tous les conteneurs, notamment ceux précédemment créés en tant que conteneurs fixes. Le SDK v3 est mis à jour vers la dernière version de l’API qui implémente ce changement, qui n’est toutefois pas cassant. Si vous ne fournissez pas de clé de partition pour les opérations, nous utiliserons par défaut une clé système qui fonctionne avec tous vos conteneurs et documents existants.

##### <a name="upsert-removed-for-stored-procedures"></a>Suppression de l’upsert pour les procédures stockées
Auparavant, l’upsert était autorisé pour les collections non partitionnées, mais avec la mise à jour de la version de l’API, toutes les collections sont partitionnées. Nous l’avons donc entièrement supprimé.

##### <a name="item-reads-will-not-throw-on-404"></a>Les lectures d’élément ne seront pas levées en cas d’erreur 404
const container = client.database(dbId).container(containerId)

``` js
// v2
try {
    container.items.read(id, undefined)
} catch (e) {
    if (e.code === 404) { console.log('item not found') }
}

// v3
const { result: item }  = container.items.read(id, undefined)
if (item === undefined) { console.log('item not found') }
```

##### <a name="default-multi-region-write"></a>Écriture multirégion par défaut
Le SDK écrira désormais par défaut dans plusieurs régions si votre configuration Cosmos prend en charge cette capacité. Ce comportement était auparavant optionnel.

##### <a name="proper-error-objects"></a>Objets d’erreur appropriés
Les requêtes ayant échoué lèvent désormais l’erreur ou les sous-classes d’erreur appropriées. Avant, elles levaient des objets JavaScript simples.

#### <a name="new-features"></a>Nouvelles fonctionnalités
##### <a name="user-cancelable-requests"></a>Requêtes annulables par l’utilisateur
Le passage à la récupération en interne nous permet d’utiliser l’API AbortController du navigateur pour prendre en charge les opérations annulables par l’utilisateur. Dans le cas d’opérations où plusieurs requêtes sont potentiellement en cours (comme les requêtes entre partitions), toutes les requêtes pour l’opération sont annulées. Les utilisateurs de navigateurs récents disposeront déjà d’AbortController. Les utilisateurs Node.js devront utiliser une bibliothèque de polyfills

``` js
 const controller = new AbortController()
 const {result: item} = await items.query('SELECT * from c', { abortSignal: controller.signal});
 controller.abort()
```

##### <a name="set-throughput-as-part-of-dbcontainer-create-operation"></a>Définition du débit dans le cadre de l’opération de création d’une base de données/d’un conteneur
``` js
const { database }  = client.databases.create({ id: 'my-database', throughput: 10000 })
database.containers.create({ id: 'my-container', throughput: 10000 })
```

##### <a name="azurecosmos-sign"></a>@azure/cosmos-sign
La génération d’un jeton d’en-tête a été divisée pour être incluse dans une nouvelle bibliothèque, @azure/cosmos-sign. Toute personne appelant l’API REST Cosmos directement peut utiliser cela pour signer des en-têtes avec le même code que celui que nous appelons dans @azure/cosmos.

##### <a name="uuid-for-generated-ids"></a>UUID des ID générés
La version v2 avait du code personnalisé pour générer des ID d’élément. Nous sommes passés à l’UUID bien connu et gérée de la bibliothèque de communauté.

##### <a name="connection-strings"></a>Chaînes de connexion
Il est maintenant possible de passer une chaîne de connexion copiée à partir du portail Azure :

``` js
const client = new CosmosClient("AccountEndpoint=https://test-account.documents.azure.com:443/;AccountKey=c213asdasdefgdfgrtweaYPpgoeCsHbpRTHhxuMsTaw==;")
Add DISTINCT and LIMIT/OFFSET queries (#306)
 const { results } = await items.query('SELECT DISTINCT VALUE r.name FROM ROOT').fetchAll()
 const { results } = await items.query('SELECT * FROM root r OFFSET 1 LIMIT 2').fetchAll()
```

#### <a name="improved-browser-experience"></a>Amélioration de l’expérience de navigateur
Bien qu’il était possible d’utiliser le SDK v2 dans le navigateur, cette expérience n’était pas optimale. Vous deviez effectuer un polyfill sur plusieurs bibliothèques intégrées node.js et utiliser un programme d’installation d’offre groupée (bundler) comme webpack ou Parcel. Le SDK v3 améliore considérablement l’expérience prête pour les utilisateurs de navigateur.

* Remplacement des éléments internes des requêtes par la récupération (fetch) (n° 245)
* Suppression de l’utilisation d’un tampon (n° 330)
* Suppression de l’utilisation intégrée de nœuds en faveur de packages universels/d’API (n° 328)
* Passage à un contrôleur des abandons de nœuds (n° 294)

#### <a name="bug-fixes"></a>Résolution des bogues
* Correction de la lecture des offres et rétablissement des tests de l’offre (n° 224)
* Correction d’EnableEndpointDiscovery (n° 207)
* Correction du unités de requête manquantes sur les résultats paginés (n° 360)
* Développement du type de paramètre de requête SQL (n° 346)
* Ajout de TTL à ItemDefinition (n° 341)
* Correction des métriques de requête de processeur central (n° 311)
* Ajout d’activityId à FeedResponse (n° 293)
* Changement du type _ts de chaîne en nombre (n° 252) (n° 295)
* Correction de l’agrégation des frais de requêtes (n° 289)
* Autorisation des clés de partition de chaîne vides (n° 277)
* Ajout d’une chaîne au type de requête de conflit (n° 237)
* Ajout d’uniqueKeyPolicy à un conteneur (n° 234)

#### <a name="engineering-systems"></a>Systèmes d’ingénierie
Ce ne sont pas toujours les changements les plus visibles, mais ils permettent à notre équipe à livrer plus rapidement un meilleur code.

* Utilisation d’un correctif cumulatif pour les builds de production (n° 104)
* Mise à jour vers Typescript 3.5 (n° 327)
* Conversion en références de projet TS. Extraction du dossier de test (n° 270)
* Activation de noUnusedLocals et de noUnusedParameters (n° 275)
* YAML Azure Pipelines pour les builds d’intégration continue (CI) (n° 298)

### <a name="215"></a><a name="2.1.5"></a>2.1.5
* Aucun changement de code. Correction d’un problème lié au fait que certains fichiers supplémentaires ont été inclus dans le package 2.1.4.

### <a name="214"></a><a name="2.1.4"></a>2.1.4
* Correction du basculement régional dans la stratégie de nouvelle tentative
* Correction de la propriété ChangeFeed hasMoreResults
* Mises à jour des dépendances de développement
* Ajout de PolicheckExclusions.txt

### <a name="213"></a><a name="2.1.3"></a>2.1.3
* Changement du type _ts de chaîne en nombre
* Correction des tests d’indexation par défaut
* Rétroportage d’uniqueKeyPolicy vers la version v2
* Correctifs pour la démonstration et le débogage de la démonstration

### <a name="212"></a><a name="2.1.2"></a>2.1.2
* Correctifs pour le rétroportage des offres disponibles dans la branche v3
* Correction du bogue dans la signature de type executeNext()
* Correction de fautes de frappe

### <a name="211"></a><a name="2.1.1"></a>2.1.1
* Restructuration de la build. Permet d’extraire la version du SDK au moment de la génération.

### <a name="210"></a><a name="2.1.0"></a>2.1.0
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Ajout de la prise en charge de ChangeFeed (n° 196)
* Ajout du type de données MultiPolygon pour l’indexation (n° 191)
* Ajout de la propriété « key  au constructeur en tant qu’alias de masterKey (n° 202)

#### <a name="fixes"></a>Correctifs
* Correction de bogue où next() retournait une valeur incorrecte sur l’itérateur

#### <a name="engineering-improvements"></a>Améliorations en matière d’ingénierie
* Ajout d’un test d’intégration pour la consommation de typescript (n° 199)
* Activation de l’installation directement à partir de GitHub (n° 194)

### <a name="205"></a><a name="2.0.5"></a>2.0.5
* Ajoute l’interface pour le type d’agent de nœud. Les utilisateurs de Typescript ne sont plus tenus d'installer @types/node en tant que dépendance
* Emplacements préférés désormais correctement honorés
* Améliorations en termes de contribution à la documentation du développeur
* Correction de diverses fautes de frappe

### <a name="204"></a><a name="2.0.4"></a>2.0.4
* Correction du problème de définition du type introduit dans la version 2.0.3

### <a name="203"></a><a name="2.0.3"></a>2.0.3
* Suppression de la dépendance `big-integer`
* Basculement vers les directives de référence pour le type AsyncIterable Les utilisateurs de Typescript ne sont plus tenus de personnaliser leur paramètre « lib ».
* Correction de fautes de frappe

### <a name="202"></a><a name="2.0.2"></a>2.0.2
* Correction des liens Lisez-moi

### <a name="201"></a><a name="2.0.1"></a>2.0.1
* Correction de l'implémentation d'interface de nouvelle tentative

### <a name="200"></a><a name="2.0.0"></a>2.0.0
* Disponibilité générale de la version 2.0.0 du SDK JavaScript
* Ajout de la prise en charge des écritures dans plusieurs régions.

### <a name="200-3"></a><a name="2.0.0-3"></a>2.0.0-3
* RC1 de la version 2.0.0 du SDK JavaScript pour la préversion publique.
* Nouveau modèle d’objet, avec le CosmosClient de niveau supérieur et les méthodes réparties entre les classes Database, Container et Item appropriées. 
* Prise en charge des [promesses](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Using_promises). 
* SDK converti en TypeScript.

### <a name="1144"></a><a name="1.14.4"></a>1.14.4
* Documentation npm mise à jour.

### <a name="1143"></a><a name="1.14.3"></a>1.14.3
* Support supplémentaire pour les nouvelles tentatives par défaut concernant les problèmes de connexion.
* Support supplémentaire pour lire le flux de modification de la collection.
* Bogue de cohérence de session fixe qui cause par intermittence le message « read session not available ».
* Support supplémentaire pour les mesures de la requête.
* Modification du nombre maximal de connexions à l’agent http.

### <a name="1142"></a><a name="1.14.2"></a>1.14.2
* Documentation mise à jour pour référencer Azure Cosmos DB au lieu d’Azure DocumentDB.
* Ajout de prise en charge pour le paramètre proxyUrl dans ConnectionPolicy.

### <a name="1141"></a><a name="1.14.1"></a>1.14.1
* Correctif mineur pour les systèmes de fichiers respectant la casse.

### <a name="1140"></a><a name="1.14.0"></a>1.14.0
* Ajout de la prise en charge de la cohérence de session.
* Cette version du SDK nécessite la dernière version de l’[émulateur Azure Cosmos DB](https://aka.ms/cosmosdb-emulator).

### <a name="1130"></a><a name="1.13.0"></a>1.13.0
* Requêtes Splitproofed entre partitions.
* Ajout de la prise en charge des liens de ressources comprenant des barres obliques de début et de fin (et des tests correspondants).

### <a name="1122"></a><a name="1.12.2"></a>1.12.2
*    Documentation npm mise à jour.

### <a name="1121"></a><a name="1.12.1"></a>1.12.1
* Correction d’un bogue dans executeStoredProcedure où les documents impliqués disposaient de caractères Unicode spéciaux (LS, PS).
* Correction d’un bogue lors de la gestion de documents contenant des caractères Unicode dans la clé de partition.
* Prise en charge rétablie de la création de collections avec le support de nom. Problème GitHub n° 114.
* Prise en charge rétablie du jeton d’autorisation. Problème GitHub n° 178.

### <a name="1120"></a><a name="1.12.0"></a>1.12.0
* Ajout de la prise en charge d’un nouveau [niveau de cohérence](consistency-levels.md) nommé ConsistentPrefix.
* Ajout de la prise en charge de UriFactory.
* Correction d’un bogue de prise en charge des caractères Unicode. Problème GitHub n° 171.

### <a name="1110"></a><a name="1.11.0"></a>1.11.0
* Ajout de la prise en charge des requêtes d’agrégation (COUNT, MIN, MAX, SUM et AVG).
* Ajout de l’option permettant de contrôler le degré de parallélisme des requêtes entre les partitions.
* Ajout de l’option permettant de désactiver la vérification TLS en cas d’exécution sur l’émulateur Azure Cosmos DB.
* Débit minimal réduit sur les collections partitionnées de 10 100 unités de demande/s à 2 500 unités de demande/s.
* Correction du bogue de jeton de continuation pour la collection à partition unique. Problème GitHub n° 107.
* Correction du bogue executeStoredProcedure lors de la gestion du 0 comme paramètre unique. Problème GitHub n° 155.

### <a name="1102"></a><a name="1.10.2"></a>1.10.2
* En-tête agent utilisateur fixe pour inclure la version du Kit de développement logiciel (SDK).
* Nettoyage de code mineur.

### <a name="1101"></a><a name="1.10.1"></a>1.10.1
* Désactiver la vérification TLS lors de l’utilisation du Kit de développement logiciel (SDK) pour cibler l’émulateur (hostname=localhost).
* Ajout de la prise en charge de l’activation de la journalisation de script pendant l’exécution de la procédure stockée.

### <a name="1100"></a><a name="1.10.0"></a>1.10.0
* Ajout de la prise en charge des requêtes parallèles sur plusieurs partitions.
* Ajout de la prise en charge des requêtes TOP/ORDER BY pour les collections partitionnées.

### <a name="190"></a><a name="1.9.0"></a>1.9.0
* Ajout de la prise en charge d’une stratégie de nouvelle tentative pour les requêtes limitées. (Les requêtes limitées reçoivent une exception de taux de requête excessif, code d’erreur 429.) Par défaut, Azure Cosmos DB accepte neuf nouvelles tentatives pour chaque requête lorsque le code d’erreur 429 est renvoyé, conformément à l’heure de retryAfter spécifiée dans l’en-tête de réponse. Il est désormais possible de définir un intervalle fixe de nouvelle tentative dans la propriété RetryOptions sur l’objet ConnectionPolicy, si vous souhaitez ignorer le temps retryAfter retourné par le serveur entre chaque nouvelle tentative. Azure Cosmos DB attend maintenant au maximum 30 secondes pour chaque requête limitée (quel que soit le nombre de nouvelles tentatives) et renvoie la réponse avec un code d’erreur 429. Cette durée peut également être remplacée dans la propriété RetryOptions sur l’objet ConnectionPolicy.
* Cosmos DB renvoie maintenant x-ms-throttle-retry-count et x-ms-throttle-retry-wait-time-ms comme en-têtes de réponse dans chaque requête pour signaler le nombre limite de nouvelles tentatives et le cumul de temps d’attente observé par la requête entre les nouvelles tentatives.
* La classe RetryOptions a été ajoutée pour exposer la propriété RetryOptions sur la classe ConnectionPolicy, qui peut être utilisée pour substituer certaines des options de nouvelle tentative par défaut.

### <a name="180"></a><a name="1.8.0"></a>1.8.0
* Ajout de la prise en charge des comptes de base de données de plusieurs régions.

### <a name="170"></a><a name="1.7.0"></a>1.7.0
* Ajout de la fonctionnalité de durée de vie (TTL) pour les documents.

### <a name="160"></a><a name="1.6.0"></a>1.6.0
* Implémentation des [collections partitionnées](partitioning-overview.md) et des [niveaux de performances définis par l’utilisateur](performance-levels.md).

### <a name="156"></a><a name="1.5.6"></a>1.5.6
* Résolution du bogue RangePartitionResolver.resolveForRead là où les liens n’étaient pas renvoyés en raison d’une concaténation incorrecte des résultats.

### <a name="155"></a><a name="1.5.5"></a>1.5.5
* Résout hashParitionResolver resolveForRead() : Levait une exception si aucune clé de partition n’était fournie, au lieu de renvoyer une liste de tous les liens enregistrés.

### <a name="154"></a><a name="1.5.4"></a>1.5.4
* Résolution du problème [n° 100](https://github.com/Azure/azure-documentdb-node/issues/100) : Agent HTTPS dédié : Éviter de modifier l’agent global pour les besoins d’Azure Cosmos DB. Utiliser un agent dédié pour toutes les requêtes de la bibliothèque.

### <a name="153"></a><a name="1.5.3"></a>1.5.3
* Corrige le problème [no 81](https://github.com/Azure/azure-documentdb-node/issues/81) : traite correctement les tirets dans les ID de média.

### <a name="152"></a><a name="1.5.2"></a>1.5.2
* Résolution du problème [n° 95](https://github.com/Azure/azure-documentdb-node/issues/95) : avertissement de fuite de l’écouteur EventEmitter.

### <a name="151"></a><a name="1.5.1"></a>1.5.1
* Résolution du problème [n° 92](https://github.com/Azure/azure-documentdb-node/issues/90) : dossier Hash renommé en hash pour les systèmes respectant la casse.

### <a name="150"></a><a name="1.5.0"></a>1.5.0
* Implémentation de la prise en charge du partitionnement via l’ajout de programmes de résolution de partitions de hachage et de plage.

### <a name="140"></a><a name="1.4.0"></a>1.4.0
* Implémentation de l’opération Upsert. Nouvelles méthodes upsertXXX sur documentClient.

### <a name="130"></a><a name="1.3.0"></a>1.3.0
* Ignorée pour aligner le numéro de version avec les autres Kits de développement logiciel (SDK).

### <a name="122"></a><a name="1.2.2"></a>1.2.2
* Fractionnement du wrapper des promesses Q dans un nouveau dépôt.
* Mise à jour du fichier de package pour le Registre npm.

### <a name="121"></a><a name="1.2.1"></a>1.2.1
* Implémentation du routage basé sur l’ID.
* Résolution du problème [n° 49](https://github.com/Azure/azure-documentdb-node/issues/49) - propriété actuelle en conflit avec la méthode current().

### <a name="120"></a><a name="1.2.0"></a>1.2.0
* Ajout de la prise en charge de l’index géospatial.
* Validation de la propriété ID pour toutes les ressources. Les ID des ressources ne peuvent pas contenir les caractères ?, /, #, &#47;&#47; ou se terminer par un espace.
* Ajoute le nouvel en-tête « progression de la transformation de l'index » à ResourceResponse.

### <a name="110"></a><a name="1.1.0"></a>1.1.0
* Implémente la stratégie d’indexation V2.

### <a name="103"></a><a name="1.0.3"></a>1.0.3
* Problème [n° 40](https://github.com/Azure/azure-documentdb-node/issues/40) : implémentation des configurations eslint et grunt dans le Kit de développement logiciel (SDK) principal et de promesse.

### <a name="102"></a><a name="1.0.2"></a>1.0.2
* Problème [#45](https://github.com/Azure/azure-documentdb-node/issues/45) - Le wrapper de promesses n’inclut pas d’en-tête avec erreur.

### <a name="101"></a><a name="1.0.1"></a>1.0.1
* Implémentation de la possibilité de créer une requête pour les conflits en ajoutant readConflicts, readConflictAsync et queryConflicts.
* Mise à jour de la documentation de l’API.
* Problème [n° 41](https://github.com/Azure/azure-documentdb-node/issues/41) : Erreur client.createDocumentAsync.

### <a name="100"></a><a name="1.0.0"></a>1.0.0
* Kit de développement logiciel (SDK) GA

## <a name="release--retirement-dates"></a>Dates de lancement et de suppression

Microsoft envoie une notification au moins **12 mois** avant le retrait d’un Kit de développement logiciel (SDK) pour faciliter la transition vers une version plus récente/prise en charge. Les nouvelles fonctionnalités et fonctions, et les optimisations sont uniquement ajoutées au Kit SDK actuel. Par conséquent, il est recommandé de toujours passer à la dernière version du SDK dès que possible.

| Version | Date de sortie | Date de suppression |
| --- | --- | --- |
| [3.1.0](#3.1.0) |26 juillet 2019 |--- |
| [3.0.4](#3.0.4) |22 juillet 2019 |--- |
| [3.0.3](#3.0.3) |17 juillet 2019 |--- |
| [3.0.2](#3.0.2) |9 juillet 2019 |--- |
| [3.0.0](#3.0.0) |28 juin 2019 |--- |
| [2.1.5](#2.1.5) |20 mars 2019 |--- |
| [2.1.4](#2.1.4) |15 mars 2019 |--- |
| [2.1.3](#2.1.3) |8 mars 2019 |--- |
| [2.1.2](#2.1.2) |28 janvier 2019 |--- |
| [2.1.1](#2.1.1) |5 décembre 2018 |--- |
| [2.1.0](#2.1.0) |4 décembre 2018 |--- |
| [2.0.5](#2.0.5) |7 novembre 2018 |--- |
| [2.0.4](#2.0.4) |30 octobre 2018 |--- |
| [2.0.3](#2.0.3) |30 octobre 2018 |--- |
| [2.0.2](#2.0.2) |10 octobre 2018 |--- |
| [2.0.1](#2.0.1) |25 septembre 2018 |--- |
| [2.0.0](#2.0.0) |24 septembre 2018 |--- |
| [2.0.0-3 (RC)](#2.0.0-3) |2 août 2018 |--- |
| [1.14.4](#1.14.4) |3 mai 2018 |30 août 2020 |
| [1.14.3](#1.14.3) |3 mai 2018 |30 août 2020 |
| [1.14.2](#1.14.2) |21 décembre 2017 |30 août 2020 |
| [1.14.1](#1.14.1) |10 novembre 2017 |30 août 2020 |
| [1.14.0](#1.14.0) |9 novembre 2017 |30 août 2020 |
| [1.13.0](#1.13.0) |11 octobre 2017 |30 août 2020 |
| [1.12.2](#1.12.2) |10 août 2017 |30 août 2020 |
| [1.12.1](#1.12.1) |10 août 2017 |30 août 2020 |
| [1.12.0](#1.12.0) |10 mai 2017 |30 août 2020 |
| [1.11.0](#1.11.0) |16 mars 2017 |30 août 2020 |
| [1.10.2](#1.10.2) |27 janvier 2017 |30 août 2020 |
| [1.10.1](#1.10.1) |22 décembre 2016 |30 août 2020 |
| [1.10.0](#1.10.0) |3 octobre 2016 |30 août 2020 |
| [1.9.0](#1.9.0) |7 juillet 2016 |30 août 2020 |
| [1.8.0](#1.8.0) |14 juin 2016 |30 août 2020 |
| [1.7.0](#1.7.0) |26 avril 2016 |30 août 2020 |
| [1.6.0](#1.6.0) |29 mars 2016 |30 août 2020 |
| [1.5.6](#1.5.6) |8 mars 2016 |30 août 2020 |
| [1.5.5](#1.5.5) |2 février 2016 |30 août 2020 |
| [1.5.4](#1.5.4) |1er février 2016 |30 août 2020 |
| [1.5.2](#1.5.2) |26 janvier 2016 |30 août 2020 |
| [1.5.2](#1.5.2) |22 janvier 2016 |30 août 2020 |
| [1.5.1](#1.5.1) |4 janvier 2016 |30 août 2020 |
| [1.5.0](#1.5.0) |31 décembre 2015 |30 août 2020 |
| [1.4.0](#1.4.0) |6 octobre 2015 |30 août 2020 |
| [1.3.0](#1.3.0) |6 octobre 2015 |30 août 2020 |
| [1.2.2](#1.2.2) |10 septembre 2015 |30 août 2020 |
| [1.2.1](#1.2.1) |15 août 2015 |30 août 2020 |
| [1.2.0](#1.2.0) |5 août 2015 |30 août 2020 |
| [1.1.0](#1.1.0) |9 juillet 2015 |30 août 2020 |
| [1.0.3](#1.0.3) |4 juin 2015 |30 août 2020 |
| [1.0.2](#1.0.2) |23 mai 2015 |30 août 2020 |
| [1.0.1](#1.0.1) |15 mai 2015 |30 août 2020 |
| [1.0.0](#1.0.0) |8 avril 2015 |30 août 2020 |

## <a name="faq"></a>Questions fréquentes (FAQ)
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>Voir aussi
Pour en savoir plus sur Cosmos DB, consultez la page du service [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).