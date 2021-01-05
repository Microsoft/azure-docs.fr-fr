---
title: API, Kit de développement logiciel (SDK) et ressources Python SQL Azure Cosmos DB
description: Découvrez l’API et le Kit SDK Python SQL, y compris les dates de publication, les dates de suppression et les modifications apportées entre chaque version du Kit SDK Python Azure Cosmos DB.
author: Rodrigossz
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: python
ms.topic: reference
ms.date: 08/12/2020
ms.author: anfeldma
ms.custom: devx-track-python
ms.openlocfilehash: b69b3a3888b2d6e19b8eaa92ba9b2fedf0e15545
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93091116"
---
# <a name="azure-cosmos-db-python-sdk-for-sql-api-release-notes-and-resources"></a>Kit de développement logiciel Python Azure Cosmos DB pour l’API SQL : Notes de publication et ressources
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

| |  |
|---|---|
|**Téléchargement du Kit de développement logiciel (SDK)**|[PyPI](https://pypi.org/project/azure-cosmos)|
|**Documentation de l’API**|[Documentation de référence sur l’API Python](/python/api/azure-cosmos/?preserve-view=true&view=azure-python)|
|**Instructions d’installation du Kit de développement logiciel (SDK)**|[Instructions d’installation du Kit de développement logiciel (SDK) Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cosmos/azure-cosmos)|
|**Prise en main**|[Bien démarrer avec le Kit de développement logiciel (SDK) Python](create-sql-api-python.md)|
|**Plateforme actuellement prise en charge**|[Python 2.7](https://www.python.org/downloads/) et [Python 3.5.3+](https://www.python.org/downloads/)|

## <a name="release-history"></a>Historique des mises en production

### <a name="410-2020-08-10"></a>4.1.0 (2020-08-10)

- Ajout d’un avertissement de désapprobation pour le mode d’indexation « différé ». Le serveur principal n’autorise plus la création de conteneurs avec ce mode et les définit comme cohérents à la place.

**Nouvelles fonctionnalités**
- Ajout de la possibilité de définir la durée de vie de stockage analytique lors de la création d’un conteneur.

**Résolution des bogues**
- Correction de la prise en charge des dictionnaires en tant qu’entrées pour les API get_client.
- Correction de la compatibilité avec Python 2/3 dans les itérateurs de requête.
- Correction de l’erreur d’indicateur de type (problème #12570).
- Correction du bogue qui avait pour effet que les en-têtes d’options n’étaient pas été ajoutés à la fonction upsert_item. Problème #11791 - Merci @aalapatirvbd.
- Correction de l’erreur qui était déclenchée en cas d’utilisation d’un ID de type autre que chaîne dans un élément. L’erreur TypeError est désormais déclenchée à la place de l’erreur AttributeError (problème #11793).

### <a name="400"></a>4.0.0

* Version stable.
* Ajout de HttpLoggingPolicy au pipeline pour permettre le passage dans un enregistreur d’événements personnalisé pour les requêtes et les en-têtes de réponse.

### <a name="400b6"></a>4.0.0b6

* Correction du bogue dans synchronized_request pour les API multimédias.
* Suppression de MediaReadMode et MediaRequestTimeout de ConnectionPolicy, car les requêtes de média ne sont pas prises en charge.

### <a name="400b5"></a>4.0.0b5

* Module azure.cosmos.errors déconseillé et remplacé par azure.cosmos.exceptions.
* Les paramètres de condition d’accès (`access_condition`, `if_match`, `if_none_match`) sont déconseillés en faveur de paramètres `match_condition` et `etag` distincts.
* Correction du bogue dans le fournisseur de mappage de routage.
* Ajout de la prise en charge des requêtes Distinct, Offset et Limit.
* Contexte d’exécution par défaut de la requête de document actuellement utilisé pour :

  * Requêtes ChangeFeed
  * Requêtes à partition unique (partitionkey ; partitionKeyRangeId est présent parmi les options)
  * Requêtes sans document.

* Erreurs pour les agrégats sur plusieurs partitions, avec la valeur « Activer la requête entre les partitions » définie sur true, mais sans la présence de mot clé « value ».
* Accès au point de terminaison du plan de requête pour d’autres scénarios afin de récupérer le plan de requête.
* Ajout de la prise en charge de `__repr__` pour les objets entité Cosmos.
* Mise à jour de la documentation.

### <a name="400b4"></a>4.0.0b4

* Ajout de la prise en charge d’un argument de mot clé `timeout` à toutes les opérations pour spécifier un délai d’expiration absolu en secondes pendant lequel l’opération doit être effectuée. Si la valeur du délai d’expiration est dépassée, un `azure.cosmos.errors.CosmosClientTimeoutError` est déclenché.

* Ajout d’un nouveau `ConnectionRetryPolicy` pour gérer le comportement des nouvelles tentatives pendant les erreurs de connexion HTTP.

* Ajout de nouveaux constructeurs et d’arguments de mot clé de configuration par opération :

  * `retry_total` : nombre maximal de nouvelles tentatives.
  * `retry_backoff_max` : délai d’attente maximal pour une nouvelle tentative en secondes.
  * `retry_fixed_interval` : intervalle fixe entre les tentatives en millisecondes.
  * `retry_read` : nombre maximal de nouvelles tentatives de lecture de sockets.
  * `retry_connect` : nombre maximal de nouvelles tentatives pour une erreur de connexion.
  * `retry_status` : nombre maximal de nouvelles tentatives sur les codes d’état d’erreur.
  * `retry_on_status_codes` : liste de codes d’état spécifiques sur lesquels effectuer une nouvelle tentative.
  * `retry_backoff_factor` : facteur de calcul du délai d’attente entre chaque tentative.

### <a name="400b3"></a>4.0.0b3

* Ajout des fonctionnalités `create_database_if_not_exists()` et `create_container_if_not_exists` à CosmosClient et Database respectivement.

### <a name="400b2"></a>4.0.0b2

* La version 4.0.0b2 est la deuxième itération de nos efforts visant à créer une bibliothèque de client adaptée aux meilleures pratiques en matière de langage Python.

**Dernières modifications**

* La connexion client a été adaptée pour consommer le pipeline HTTP défini dans `azure.core.pipeline`.

* Les objets interactifs ont été renommés en tant que proxys. notamment :

  * `Database` -> `DatabaseProxy`
  * `User` -> `UserProxy`
  * `Container` -> `ContainerProxy`
  * `Scripts` -> `ScriptsProxy`

* Le constructeur de `CosmosClient` a été mis à jour :

  * Le paramètre `auth` a été renommé `credential` et prendra maintenant un type d’authentification directement. Cela signifie que vous pouvez passer la valeur de clé primaire, un dictionnaire de jetons de ressources ou une liste d’autorisations. Toutefois, l’ancien format de dictionnaire est toujours pris en charge.

  * Le paramètre `connection_policy` a été transformé en paramètre de mot clé uniquement et, bien qu’il soit toujours pris en charge, chacun des attributs individuels de la stratégie peut désormais être transmis en tant qu’argument de mot clé explicite :

    * `request_timeout`
    * `media_request_timeout`
    * `connection_mode`
    * `media_read_mode`
    * `proxy_config`
    * `enable_endpoint_discovery`
    * `preferred_locations`
    * `multiple_write_locations`

* Un nouveau constructeur a été ajouté à `CosmosClient` pour permettre la création via une chaîne de connexion récupérée à partir du portail Azure.

* Certaines opérations `read_all` ont été renommées opérations `list` :

  * `CosmosClient.read_all_databases` -> `CosmosClient.list_databases`
  * `Container.read_all_conflicts` -> `ContainerProxy.list_conflicts`
  * `Database.read_all_containers` -> `DatabaseProxy.list_containers`
  * `Database.read_all_users` -> `DatabaseProxy.list_users`
  * `User.read_all_permissions` -> `UserProxy.list_permissions`

* Toutes les opérations qui acceptent des paramètres `request_options` ou `feed_options` ont été déplacées vers des paramètres de mot clé uniquement. En outre, bien que ces dictionnaires d’options soient toujours pris en charge, chacune des options individuelles dans le dictionnaire est maintenant prise en charge en tant qu’argument de mot clé explicite.

* La hiérarchie d’erreurs est désormais héritée de `azure.core.AzureError` :

  * `HTTPFailure` a été renommé en `CosmosHttpResponseError`
  * `JSONParseFailure` a été supprimé et remplacé par `azure.core.DecodeError`
  * Ajout d’erreurs supplémentaires pour des codes de réponse spécifiques :
    * `CosmosResourceNotFoundError` pour l’état 404
    * `CosmosResourceExistsError` pour l’état 409
    * `CosmosAccessConditionFailedError` pour l’état 412

* `CosmosClient` peut maintenant être exécuté dans un gestionnaire de contexte pour traiter la fermeture de la connexion cliente.

* Les réponses pouvant être itérées (par exemple, les réponses aux requêtes et les réponses de liste) sont désormais du type `azure.core.paging.ItemPaged`. La méthode `fetch_next_block` a été remplacée par un itérateur secondaire, accessible par la méthode `by_page`.

### <a name="400b1"></a>4.0.0b1

La version 4.0.0b1 est la première préversion de nos efforts visant à créer une bibliothèque de client conviviale adaptée aux meilleures pratiques en matière de langage Python. Pour plus d’informations à ce sujet et pour obtenir des versions préliminaires d’autres bibliothèques du Kit de développement logiciel (SDK) Azure, rendez-vous sur le site https://aka.ms/azure-sdk-preview1-python.

**Changements cassants : nouvelle conception de l’API**

* Les opérations sont désormais étendues à un client particulier :

  * `CosmosClient`: ce client traite les opérations au niveau du compte. Cela comprend la gestion des propriétés de service et le référencement des bases de données dans un compte.
  * `Database`: ce client traite les opérations au niveau de la base de données. Cela comprend la création et la suppression de conteneurs, d’utilisateurs et de procédures stockées. Il est accessible à partir d’une instance `CosmosClient` par nom.
  * `Container`: ce client traite les opérations pour un conteneur particulier. Cela comprend l’interrogation et l’insertion d’éléments et la gestion des propriétés.
  * `User`: ce client traite les opérations pour un utilisateur particulier. Cela comprend l’ajout et la suppression d’autorisations et la gestion des propriétés d’utilisateur.

    Vous pouvez accéder à ces clients en parcourant la hiérarchie du client à l’aide de la méthode `get_<child>_client`. Pour tout savoir sur la nouvelle API, consultez la [documentation de référence](https://aka.ms/azsdk-python-cosmos-ref).

* Les clients sont accessibles par leur nom plutôt que par leur ID. Il n’est pas nécessaire de concaténer des chaînes pour créer des liens.

* Il n’est plus nécessaire d’importer des types et des méthodes à partir de modules individuels. La surface d’exposition de l’API publique est disponible directement dans le package `azure.cosmos`.

* Les propriétés de requête individuelles peuvent être fournies en tant qu’arguments de mot clé au lieu de construire une instance `RequestOptions` distincte.

### <a name="302"></a>3.0.2

* Ajout de la prise en charge du type de données multipolygone
* Correctif de bogue dans la stratégie de nouvelles tentatives de lecture de session
* Correctif de bogue pour les problèmes de remplissage incorrect lors du décodage de chaînes base 64

### <a name="301"></a>3.0.1

* Correction de bogue dans LocationCache
* Correction de bogue au niveau de la logique de nouvelle tentative de point de terminaison
* Documentation mise à jour

### <a name="300"></a>3.0.0

* Ajout de la prise en charge des écritures multirégions
* Modifications des dénominations
  * DocumentClient devient CosmosClient
  * Collection devient Container
  * Document devient Item
  * Nom du package mis à jour en « azure-cosmos »
  * Espace de noms mis à jour en « azure.cosmos »

### <a name="233"></a>2.3.3

* Ajout de la prise en charge de proxy
* Ajout de la prise en charge de la lecture du flux de modification
* Ajout de la prise en charge des en-têtes de quota de collection
* Correction du problème de jetons de session volumineuse
* Correction de bogue au niveau de l’API ReadMedia
* Correction de bogue dans le cache de plage de clés de partition

### <a name="232"></a>2.3.2

* Support supplémentaire pour les nouvelles tentatives par défaut concernant les problèmes de connexion.

### <a name="231"></a>2.3.1

* Documentation mise à jour pour référencer Azure Cosmos DB au lieu d’Azure DocumentDB.

### <a name="230"></a>2.3.0

* Cette version du kit de développement logiciel (SDK) nécessite la dernière version de l’émulateur Azure Cosmos DB, que vous pouvez télécharger à l’adresse https://aka.ms/cosmosdb-emulator.

### <a name="221"></a>2.2.1

* Correctif de bogue pour le dictionnaire d’agrégation
* Correctif de bogue pour la suppression des barres obliques dans le lien de ressource
* Tests pour l’encodage Unicode

### <a name="220"></a>2.2.0

* Prise en charge ajoutée pour la fonctionnalité Unité de requête par minute (RU/m).
* Prise en charge ajoutée pour un nouveau niveau de cohérence nommé ConsistentPrefix.

### <a name="210"></a>2.1.0

* Ajout de la prise en charge des requêtes d’agrégation (COUNT, MIN, MAX, SUM et AVG).
* Ajout d’une option permettant de désactiver la vérification SSL pendant son exécution sur l’émulateur DocumentDB.
* Suppression de la restriction du module de demandes dépendantes qui devait correspondre exactement à la version 2.10.0.
* Débit minimal réduit sur les collections partitionnées de 10 100 unités de demande/s à 2 500 unités de demande/s.
* Ajout de la prise en charge de l’activation de la journalisation de script pendant l’exécution de la procédure stockée.
* Version de l’API REST passée à « 2017-01-19 » à l’occasion de cette publication.

### <a name="201"></a>2.0.1

* Modifications éditoriales apportées aux commentaires de documentation.

### <a name="200"></a>2.0.0

* Ajout de la prise en charge de Python 3.5.
* Ajout de la prise en charge du regroupement de connexions à l’aide du module de demandes.
* Ajout de la prise en charge de la cohérence de session.
* Ajout de la prise en charge des requêtes TOP/ORDERBY pour les collections partitionnées.

### <a name="190"></a>1.9.0

* Ajout de la prise en charge d’une stratégie de nouvelle tentative pour les requêtes limitées. (Les requêtes limitées reçoivent une exception de taux de requête excessif, code d’erreur 429.) Par défaut, DocumentDB accepte neuf nouvelles tentatives pour chaque requête lorsque le code d’erreur 429 est retourné, conformément au temps retryAfter spécifié dans l’en-tête de réponse.
  Il est désormais possible de définir un intervalle fixe de nouvelle tentative dans la propriété RetryOptions sur l’objet ConnectionPolicy, si vous souhaitez ignorer le temps retryAfter retourné par le serveur entre chaque nouvelle tentative.
  DocumentDB attend maintenant au maximum 30 secondes pour chaque requête limitée (quel que soit le nombre de nouvelles tentatives) et renvoie la réponse avec un code d’erreur 429.
  Cette durée peut également être remplacée dans la propriété RetryOptions sur l’objet ConnectionPolicy.

* DocumentDB retourne maintenant x-ms-throttle-retry-count and x-ms-throttle-retry-wait-time-ms comme en-têtes de réponse dans chaque requête afin d’indiquer le nombre limite de nouvelles tentatives et le cumul de temps d’attente observé par la requête entre les nouvelles tentatives.

* Suppression de la classe RetryPolicy et de la propriété correspondante (retry_policy) exposées sur la classe document_client, et introduction d’une classe RetryOptions qui expose la propriété RetryOptions sur la classe ConnectionPolicy pouvant être utilisée pour substituer certaines des options de nouvelle tentative par défaut.

### <a name="180"></a>1.8.0

* Ajout de la prise en charge des comptes de base de données géorépliqués.
* Correctif des tests pour déplacer l’hôte global et la masterKey dans les classes de test individuelles.

### <a name="170"></a>1.7.0

* Ajout de la fonctionnalité de durée de vie (TTL) pour les documents.

### <a name="161"></a>1.6.1

* Résolution des bogues liés au partitionnement côté serveur pour autoriser des caractères spéciaux dans le chemin d’accès à la clé de partition.

### <a name="160"></a>1.6.0

* Ajout de la prise en charge de la fonctionnalité de collections partitionnées côté serveur.

### <a name="150"></a>1.5.0

* Ajout de l’infrastructure de partitionnement côté client au Kit de développement logiciel (SDK). Implémentation des classes HashPartionResolver et RangePartitionResolver.

### <a name="142"></a>1.4.2

* Implémentation de l’opération Upsert. Nouvelles méthodes UpsertXXX ajoutées pour prendre en charge la fonctionnalité Upsert.
* Implémentation du routage basé sur l’ID. Aucune modification d'API publique, toutes les modifications en interne.

### <a name="130"></a>1.3.0

* Version ignorée pour aligner le numéro de version avec les autres Kits de développement logiciel (SDK)

### <a name="120"></a>1.2.0

* Prise en charge de l'index géospatial.
* Validation de la propriété ID pour toutes les ressources. Les ID des ressources ne peuvent pas contenir les caractères `?, /, #, \\` ni se terminer par une espace.
* Ajoute le nouvel en-tête « progression de la transformation de l'index » à ResourceResponse.

### <a name="110"></a>1.1.0

* Implémente la stratégie d'indexation V2

### <a name="101"></a>1.0.1

* Prise en charge la connexion proxy

## <a name="release--retirement-dates"></a>Dates de lancement et de suppression

Microsoft envoie une notification au moins **12 mois** avant le retrait d’un Kit de développement logiciel (SDK) pour faciliter la transition vers une version plus récente/prise en charge. Les nouvelles fonctionnalités et fonctions, et les optimisations sont uniquement ajoutées au Kit SDK actuel. Par conséquent, il est recommandé de toujours passer à la dernière version du SDK dès que possible.

> [!WARNING]
> Après le 31 août 2022, Azure Cosmos DB n’apportera plus de correctifs de bogues, n'ajoutera plus de nouvelles fonctionnalités et ne fournira plus de support aux versions 1.x ou 2.x du Kit de développement logiciel (SDK) Python Azure Cosmos DB pour l’API SQL. Si vous préférez ne pas effectuer la mise à niveau, les requêtes envoyées depuis la version 1.x et 2.x du Kit de développement logiciel (SDK) continueront à être traitées par le service Azure Cosmos DB.

| Version | Date de sortie | Date de suppression |
| --- | --- | --- |
| [4.0.0](#400) |20 mai 2020 |--- |
| [3.0.2](#302) |15 novembre 2018 |--- |
| [3.0.1](#301) |04 octobre 2018 |--- |
| [2.3.3](#233) |8 septembre 2018 |30 août 2020 |
| [2.3.2](#232) |8 mai 2018 |30 août 2020 |
| [2.3.1](#231) |21 décembre 2017 |30 août 2020 |
| [2.3.0](#230) |10 novembre 2017 |30 août 2020 |
| [2.2.1](#221) |29 septembre 2017 |30 août 2020 |
| [2.2.0](#220) |10 mai 2017 |30 août 2020 |
| [2.1.0](#210) |1er mai 2017 |30 août 2020 |
| [2.0.1](#201) |30 octobre 2016 |30 août 2020 |
| [2.0.0](#200) |29 septembre 2016 |30 août 2020 |
| [1.9.0](#190) |7 juillet 2016 |30 août 2020 |
| [1.8.0](#180) |14 juin 2016 |30 août 2020 |
| [1.7.0](#170) |26 avril 2016 |30 août 2020 |
| [1.6.1](#161) |8 avril 2016 |30 août 2020 |
| [1.6.0](#160) |29 mars 2016 |30 août 2020 |
| [1.5.0](#150) |3 janvier 2016 |30 août 2020 |
| [1.4.2](#142) |6 octobre 2015 |30 août 2020 |
| 1.4.1 |6 octobre 2015 |30 août 2020 |
| [1.2.0](#120) |6 août 2015 |30 août 2020 |
| [1.1.0](#110) |9 juillet 2015 |30 août 2020 |
| [1.0.1](#101) |25 mai 2015 |30 août 2020 |
| 1.0.0 |7 avril 2015 |30 août 2020 |
| 0.9.4-prelease |14 janvier 2015 |29 février 2016 |
| 0.9.3-prelease |9 décembre 2014 |29 février 2016 |
| 0.9.2-prelease |25 novembre 2014 |29 février 2016 |
| 0.9.1-prelease |23 septembre 2014 |29 février 2016 |
| 0.9.0-prelease |21.08.14 |29 février 2016 |

## <a name="faq"></a>Questions fréquentes (FAQ)

[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur Cosmos DB, consultez la page du service [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).