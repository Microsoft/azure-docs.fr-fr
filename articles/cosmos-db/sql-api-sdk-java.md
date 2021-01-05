---
title: 'Azure Cosmos DB : API, SDK et ressources SQL Java'
description: Découvrez l’API et le Kit de développement logiciel (SDK) Java SQL, notamment les dates de lancement, les dates de suppression et les modifications apportées entre chaque version du Kit de développement logiciel (SDK) Java SQL d’Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 08/12/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 0b48b8d954b132caab96e5978c90687899ea04c2
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96549223"
---
# <a name="azure-cosmos-db-java-sdk-for-sql-api-release-notes-and-resources"></a>Kit de développement logiciel (SDK) Java Azure Cosmos DB pour API SQL : Notes de publication et ressources
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

Il s’agit du kit de développement logiciel (SDK) Java synchrone v2 d’origine Azure Cosmos DB pour l’API SQL qui prend en charge des opérations synchrones.

> [!IMPORTANT]  
> Il ne s’agit *pas* du Kit de développement logiciel (SDK) Java pour Azure Cosmos DB le plus récent ! Envisagez d’utiliser le [Kit de développement logiciel (SDK) Java v4 Azure Cosmos DB](sql-api-sdk-java-v4.md) pour votre projet. Suivez les instructions fournies dans les guides [Migrer vers le Kit de développement logiciel (SDK) Java v4 Azure Cosmos DB](migrate-java-v4-sdk.md) et [Reactor contre RxJava](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/reactor-rxjava-guide.md) pour la mise à niveau. 
>

| |  |
|---|---|
|**Téléchargement du Kit de développement logiciel (SDK)**|[Maven](https://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22com.microsoft.azure%22%20AND%20a%3A%22azure-documentdb%22)|
|**Documentation de l’API**|[Documentation de référence sur l’API Java](/java/api/com.microsoft.azure.documentdb)|
|**Contribution au Kit de développement logiciel (SDK)**|[GitHub](https://github.com/Azure/azure-documentdb-java/)|
|**Prise en main**|[Bien démarrer avec le Kit de développement logiciel (SDK) Java](./create-sql-api-java.md)|
|**Didacticiel d’application web**|[Développement d’applications web avec Azure Cosmos DB](sql-api-java-application.md)|
|**Runtime minimal pris en charge**|[Kit de développement Java (JDK) 7+](/java/azure/jdk/?view=azure-java-stable&preserve-view=true)|

## <a name="release-notes"></a>Notes de publication

### <a name="251"></a><a name="2.5.1"></a>2.5.1
* Corrige le problème de cache de partition principale sur la requête documentCollection.

### <a name="250"></a><a name="2.5.0"></a>2.5.0
* Ajout de la prise en charge de la configuration personnalisée de nouvelle tentative 449.

### <a name="247"></a><a name="2.4.7"></a>2.4.7
* Corrige le problème de délai d’attente du pool de connexions.
* Corrige l’actualisation des jetons d’authentification en cas de nouvelles tentatives internes.

### <a name="246"></a><a name="2.4.6"></a>2.4.6
* Mise à jour de la bonne étiquette de stratégie de réplica côté client sur databaseAccount et mise à jour des lectures de configuration databaseAccount à partir du cache.

### <a name="245"></a><a name="2.4.5"></a>2.4.5
* Nouvelle tentative évitée en cas d’erreur de plage de clés de partition non valide, si l’utilisateur fournit pkRangeId.

### <a name="244"></a><a name="2.4.4"></a>2.4.4
* Actualisations du cache des plages de clés de partition optimisées.
* Résout le scénario dans lequel le kit SDK n’a pas d’indicateur de fractionnement de partition du serveur et entraîne incorrectement l’actualisation des caches de routage côté client.

### <a name="242"></a><a name="2.4.2"></a>2.4.2
* Actualisations optimisées du cache de collection.

### <a name="241"></a><a name="2.4.1"></a>2.4.1
* Ajout de la prise en charge de la récupération du message d’exception interne de la chaîne de diagnostic de la requête.

### <a name="240"></a><a name="2.4.0"></a>2.4.0
* Introduction de l’API Version sur PartitionKeyDefinition.

### <a name="230"></a><a name="2.3.0"></a>2.3.0
* Ajout de la prise en charge des délais d’attente distincts pour le mode direct.

### <a name="223"></a><a name="2.2.3"></a>2.2.3
* Utilisation d’un message d’erreur null du service et production d’une exception client de document.

### <a name="222"></a><a name="2.2.2"></a>2.2.2
* Amélioration de la connexion au socket, ajout de SoKeepAlive par défaut sur true.

### <a name="220"></a><a name="2.2.0"></a>2.2.0
* Ajout de la prise en charge des chaînes de diagnostic des requêtes.

### <a name="213"></a><a name="2.1.3"></a>2.1.3
* Correction d’un bogue dans PartitionKey pour Hash V2.

### <a name="212"></a><a name="2.1.2"></a>2.1.2
* Ajout de la prise en charge des index composites.
* Correction d’un bogue dans le gestionnaire global des points de terminaison pour forcer le rafraîchissement.
* Correction d’un bogue pour les upserts avec des conditions préalables en mode direct.

### <a name="211"></a><a name="2.1.1"></a>2.1.1
* Correction d’un bogue dans le cache d’adresse de la passerelle.

### <a name="210"></a><a name="2.1.0"></a>2.1.0
* Ajout de la prise en charge de l’écriture multirégion pour le mode direct.
* Ajout de la prise en charge du traitement des IOExceptions levées en tant qu’exceptions ServiceUnavailable à partir d’un proxy.
* Correction d’un bogue dans la stratégie de nouvelle tentative de découverte de point de terminaison.
* Correction d’un bogue pour éviter que des exceptions soient levées dans BaseDatabaseAccountConfigurationProvider de pointeur null.
* Correction d’un bogue pour éviter que l’itérateur de requête ne retourne des valeurs NULL.
* Correction d’un bogue pour autoriser une clé de partition volumineuse

### <a name="200"></a><a name="2.0.0"></a>2.0.0
* Ajout de la prise en charge de l’écriture multirégion pour le mode passerelle.

### <a name="1164"></a><a name="1.16.4"></a>1.16.4
* Correction d’un bogue dans les plages de clés de partition en lecture pour une requête.

### <a name="1163"></a><a name="1.16.3"></a>1.16.3
* Correction d’un bogue au niveau de la définition de la taille de l’en-tête du jeton de continuation en mode DirectHttps.

### <a name="1162"></a><a name="1.16.2"></a>1.16.2
* Ajout de la prise en charge du basculement de la diffusion en continu.
* Ajout de la prise en charge des métadonnées personnalisées.
* Amélioration de la logique de gestion de session.
* Correction d’un bogue dans le cache de plage de clés de partition.
* Correction d’un bogue NPE en mode direct.

### <a name="1161"></a><a name="1.16.1"></a>1.16.1
* Ajout de la prise en charge de l’index unique.
* Possibilité de limiter la taille du jeton de continuation dans les options de flux.
* Correction d’un bogue dans la sérialisation Json (timestamp).
* Correction d’un bogue dans la sérialisation Json (enum).
* Dépendance à com.fasterxml.jackson.core:jackson-databind mise au niveau de version 2.9.5.

### <a name="1160"></a><a name="1.16.0"></a>1.16.0
* Amélioration du regroupement de connexions pour le mode Direct.
* Amélioration de la prérécupération pour la requête multipartitionnelle sans clause OrderBy.
* Amélioration de la génération d’UUID.
* Amélioration de la logique de cohérence de session.
* Ajout de la prise en charge de multipolygones.
* Ajout de la prise en charge des statistiques de plage de clés de partition pour la collecte.
* Correction d’un bogue dans la prise en charge de plusieurs régions.

### <a name="1150"></a><a name="1.15.0"></a>1.15.0
* Amélioration des performances de la sérialisation Json.
* Cette version du SDK nécessite la dernière version de l’[émulateur Azure Cosmos DB](https://aka.ms/cosmosdb-emulator).

### <a name="1140"></a><a name="1.14.0"></a>1.14.0
* Changements internes concernant les bibliothèques Microsoft.

### <a name="1130"></a><a name="1.13.0"></a>1.13.0
* Correction d’un problème de lecture des plages de clés de partition uniques.
* Correction d’un problème d’analyse d’ID de ressource affectant les bases de données avec des noms courts.
* Correction d’un problème dû à l’encodage des clés de partition.

### <a name="1120"></a><a name="1.12.0"></a>1.12.0
* Correctifs de bogues critiques pour demander le traitement lors de fractionnements de partition.
* Correction d’un problème avec les niveaux de cohérence Strong et BoundedStaleness.

### <a name="1110"></a><a name="1.11.0"></a>1.11.0
* Prise en charge ajoutée pour un nouveau niveau de cohérence nommé ConsistentPrefix.
* Correction d’un bogue de lecture de collection en mode de session.

### <a name="1100"></a><a name="1.10.0"></a>1.10.0
* Activation de la prise en charge de la collection partitionnée avec au minimum 2 500 RU/seconde et des incréments d’évolution de 100 RU/seconde.
* Correction d’un bogue dans l’assembly natif, qui pouvait entraîner des exceptions NullRef pour certaines requêtes.

### <a name="196"></a><a name="1.9.6"></a>1.9.6
* Correction d’un bogue dans la configuration du moteur de requête qui peut provoquer des exceptions pour les requêtes en mode de passerelle.
* Correction de quelques bogues dans le conteneur de session qui peut provoquer une exception « Ressource propriétaire introuvable » pour les demandes dès la création de la collection.

### <a name="195"></a><a name="1.9.5"></a>1.9.5
* Ajout de la prise en charge des requêtes d’agrégation (COUNT, MIN, MAX, SUM et AVG). Consultez l’article [Aggregation support (Prise en charge de l’agrégation)](sql-query-aggregate-functions.md).
* Ajout de la prise en charge de la modification de flux.
* Ajout de la prise en charge des informations relatives aux quotas de collections via RequestOptions.setPopulateQuotaInfo.
* Ajout de la prise en charge de l’enregistrement de script de procédure stockée via RequestOptions.setScriptLoggingEnabled.
* Correction d’un bogue dans lequel la requête en mode DirectHttps peut arrêter de répondre lorsqu’elle rencontre des échecs de limitation.
* Correction d’un bogue dans le mode de cohérence de session.
* Correction d’un bogue susceptible d’entraîner l’exception NullReferenceException dans HttpContext lorsque le taux de demandes est élevé.
* Amélioration des performances du mode DirectHttps.

### <a name="194"></a><a name="1.9.4"></a>1.9.4
* Ajout de la prise en charge simple d’un proxy basé sur les instances d’un client avec l’API ConnectionPolicy.setProxy().
* Ajout de l’API DocumentClient.close() permettant de fermer correctement l’instance DocumentClient.
* Amélioration des performances de requête en mode de connectivité directe en dérivant le plan de requête à partir de l’assembly natif au lieu de la passerelle.
* Définissez FAIL_ON_UNKNOWN_PROPERTIES = false, de sorte que les utilisateurs n’aient pas besoin de définir JsonIgnoreProperties dans leur POJO.
* Journalisation refactorisée pour utiliser SLF4J.
* Correction de quelques autres bogues dans un lecteur de cohérence.

### <a name="193"></a><a name="1.9.3"></a>1.9.3
* Correction d’un bogue dans la gestion des connexions pour éviter les fuites de connexion en mode de connectivité directe.
* Correction dans la requête TOP d'un bogue qui pouvait générer une exception NullReference.
* Amélioration des performances avec la réduction du nombre d’appels réseau pour les caches internes.
* Ajout d’un code d’état, d’un ID d’activité et d’un URI de requête dans DocumentClientException pour une meilleure résolution des problèmes.

### <a name="192"></a><a name="1.9.2"></a>1.9.2
* Résolution d’un problème de gestion des connexions pour plus de stabilité.

### <a name="191"></a><a name="1.9.1"></a>1.9.1
* Ajout de la prise en charge du niveau de cohérence BoundedStaleness.
* Ajout de la prise en charge de la connectivité directe pour les opérations CRUD pour les collections partitionnées.
* Correction d’un bogue dans l’interrogation d’une base de données avec SQL.
* Correction d’un bogue dans le cache de session où le jeton de session peut être défini de manière incorrecte.

### <a name="190"></a><a name="1.9.0"></a>1.9.0
* Ajout de la prise en charge des requêtes parallèles sur plusieurs partitions.
* Ajout de la prise en charge des requêtes TOP/ORDER BY pour les collections partitionnées.
* Ajout de la prise en charge de la cohérence forte.
* Ajout de la prise en charge des requêtes en fonction du nom lors de l’utilisation de la connectivité directe.
* Correction visant à rendre ActivityId cohérent entre toutes les nouvelles tentatives de requête.
* Correction d’un bogue lié au cache de session lors de la nouvelle création d’une collection avec le même nom.
* Ajout des types de données Polygone et LineString lors de la spécification de la stratégie d’indexation de collection pour les requêtes spatiales à délimitation géographique.
* Résolution des problèmes liés à Java Doc pour Java 1.8.

### <a name="181"></a><a name="1.8.1"></a>1.8.1
* Correction d’un bogue dans PartitionKeyDefinitionMap pour la mise en cache de collections de partitions uniques, sans aucune extraction supplémentaire des demandes de clés de partition.
* Correction d’un bogue pour l’absence de nouvelle tentative en cas de valeur de clé de partition incorrecte.

### <a name="180"></a><a name="1.8.0"></a>1.8.0
* Ajout de la prise en charge des comptes de base de données de plusieurs régions.
* Ajout de la prise en charge d’une nouvelle tentative automatique sur les requêtes limitées avec options de personnalisation du nombre maximum de tentatives et du délai d’attente maximum entre deux tentatives.  Voir RetryOptions et ConnectionPolicy.getRetryOptions().
* Propriété IPartitionResolver déconseillée basée sur un code de partitionnement personnalisé. Utilisez des collections partitionnées pour bénéficier d’un niveau de stockage et de débit supérieur.

### <a name="171"></a><a name="1.7.1"></a>1.7.1
* Ajout de la prise en charge d’une stratégie de nouvelle tentative pour la limitation du débit.  

### <a name="170"></a><a name="1.7.0"></a>1.7.0
* Ajout de la prise en charge de la durée de vie (TTL) pour les documents.

### <a name="160"></a><a name="1.6.0"></a>1.6.0
* Implémentation des [collections partitionnées](partitioning-overview.md) et des [niveaux de performances définis par l’utilisateur](performance-levels.md).

### <a name="151"></a><a name="1.5.1"></a>1.5.1
* Correction d’un bogue dans HashPartitionResolver pour générer des valeurs de hachage en little-endian dans un souci de cohérence avec les autres kits de développement logiciel.

### <a name="150"></a><a name="1.5.0"></a>1.5.0
* Ajoutez des programmes de résolution de partitions par hachage et par spécification de plages de valeurs pour vous aider lors du partitionnement des applications sur plusieurs partitions.

### <a name="140"></a><a name="1.4.0"></a>1.4.0
* Implémentation de l’opération Upsert. Nouvelles méthodes upsertXXX ajoutées pour prendre en charge la fonctionnalité Upsert.
* Implémenter l'ID en fonction du routage. Aucune modification d'API publique, toutes les modifications en interne.

### <a name="130"></a><a name="1.3.0"></a>1.3.0
* Version ignorée pour aligner le numéro de version avec les autres Kits de développement logiciel (SDK)

### <a name="120"></a><a name="1.2.0"></a>1.2.0
* Prise en charge de l'index géospatial
* Validation de la propriété ID pour toutes les ressources. Les ID des ressources ne peuvent pas contenir les caractères ?, /, #, \, ou se terminer par un espace.
* Ajoute le nouvel en-tête « progression de la transformation de l'index » à ResourceResponse.

### <a name="110"></a><a name="1.1.0"></a>1.1.0
* Implémente la stratégie d'indexation V2

### <a name="100"></a><a name="1.0.0"></a>1.0.0
* Kit SDK GA

## <a name="release-and-retirement-dates"></a>Dates de lancement et de suppression

Microsoft fournira une notification au moins **12 mois** avant le retrait d’un Kit de développement logiciel (SDK) pour faciliter la transition vers une version plus récente/prise en charge. Les nouvelles fonctionnalités et fonctions, et les optimisations sont uniquement ajoutées au Kit de développement logiciel (SDK) actuel. Par conséquent, il est recommandé de toujours passer à la dernière version du Kit de développement logiciel (SDK) dès que possible.

> [!WARNING]
> Après le 30 mai 2020, Azure Cosmos DB n’apportera plus de correctifs de bogues, n’ajoutera plus de nouvelles fonctionnalités et ne fournira plus de support aux versions 1.x du Kit de développement logiciel (SDK) Java Azure Cosmos DB pour l’API SQL. Si vous préférez ne pas effectuer la mise à niveau, les requêtes envoyées depuis la version 1.x du Kit de développement logiciel (SDK) continueront à être traitées par le service Azure Cosmos DB.
>
> Après le 29 février 2016, Azure Cosmos DB n’apportera plus de correctifs de bogues, n’ajoutera plus de nouvelles fonctionnalités et ne fournira plus de support aux versions 0.x du Kit de développement logiciel (SDK) Java Azure Cosmos DB pour l’API SQL. Si vous préférez ne pas effectuer la mise à niveau, les requêtes envoyées depuis la version 0.x du Kit de développement logiciel (SDK) continueront à être traitées par le service Azure Cosmos DB.


| Version | Date de sortie | Date de suppression |
| --- | --- | --- |
| [2.5.1](#2.5.1) |03 juin 2020 |--- |
| [2.5.0](#2.5.0) |12 mai 2020 |--- |
| [2.4.7](#2.4.7) |20 février 2020 |--- |
| [2.4.6](#2.4.6) |24 janvier 2020 |--- |
| [2.4.5](#2.4.5) |10 novembre 2019 |--- |
| [2.4.4](#2.4.4) |24 octobre 2019 |--- |
| [2.4.2](#2.4.2) |26 septembre 2019 |--- |
| [2.4.1](#2.4.1) |18 juillet 2019 |--- |
| [2.4.0](#2.4.0) |4 mai 2019 |--- |
| [2.3.0](#2.3.0) |24 avril 2019 |--- |
| [2.2.3](#2.2.3) |16 avril 2019 |--- |
| [2.2.2](#2.2.2) |5 avril 2019 |--- |
| [2.2.0](#2.2.0) |27 mars 2019 |--- |
| [2.1.3](#2.1.3) |13 mars 2019 |--- |
| [2.1.2](#2.1.2) |9 mars 2019 |--- |
| [2.1.1](#2.1.1) |13 décembre 2018 |--- |
| [2.1.0](#2.1.0) |20 novembre 2018 |--- |
| [2.0.0](#2.0.0) |21 septembre 2018 |--- |
| [1.16.4](#1.16.4) |10 septembre 2018 |30 mai 2020 |
| [1.16.3](#1.16.3) |09 septembre 2018 |30 mai 2020 |
| [1.16.2](#1.16.2) |29 juin 2018 |30 mai 2020 |
| [1.16.1](#1.16.1) |16 mai 2018 |30 mai 2020 |
| [1.16.0](#1.16.0) |15 mars 2018 |30 mai 2020 |
| [1.15.0](#1.15.0) |14 novembre 2017 |30 mai 2020 |
| [1.14.0](#1.14.0) |28 octobre 2017 |30 mai 2020 |
| [1.13.0](#1.13.0) |25 août 2017 |30 mai 2020 |
| [1.12.0](#1.12.0) |11 juillet 2017 |30 mai 2020 |
| [1.11.0](#1.11.0) |10 mai 2017 |30 mai 2020 |
| [1.10.0](#1.10.0) |11 mars 2017 |30 mai 2020 |
| [1.9.6](#1.9.6) |21 février 2017 |30 mai 2020 |
| [1.9.5](#1.9.5) |31 janvier 2017 |30 mai 2020 |
| [1.9.4](#1.9.4) |24 novembre 2016 |30 mai 2020 |
| [1.9.3](#1.9.3) |30 octobre 2016 |30 mai 2020 |
| [1.9.2](#1.9.2) |28 octobre 2016 |30 mai 2020 |
| [1.9.1](#1.9.1) |26 octobre 2016 |30 mai 2020 |
| [1.9.0](#1.9.0) |3 octobre 2016 |30 mai 2020 |
| [1.8.1](#1.8.1) |30 juin 2016 |30 mai 2020 |
| [1.8.0](#1.8.0) |14 juin 2016 |30 mai 2020 |
| [1.7.1](#1.7.1) |30 avril 2016 |30 mai 2020 |
| [1.7.0](#1.7.0) |27 avril 2016 |30 mai 2020 |
| [1.6.0](#1.6.0) |29 mars 2016 |30 mai 2020 |
| [1.5.1](#1.5.1) |31 décembre 2015 |30 mai 2020 |
| [1.5.0](#1.5.0) |4 décembre 2015 |30 mai 2020 |
| [1.4.0](#1.4.0) |5 octobre 2015 |30 mai 2020 |
| [1.3.0](#1.3.0) |5 octobre 2015 |30 mai 2020 |
| [1.2.0](#1.2.0) |5 août 2015 |30 mai 2020 |
| [1.1.0](#1.1.0) |9 juillet 2015 |30 mai 2020 |
| 1.0.1 |12 mai 2015 |30 mai 2020 |
| [1.0.0](#1.0.0) |7 avril 2015 |30 mai 2020 |
| 0.9.5-prelease |9 mars 2015 |29 février 2016 |
| 0.9.4-prelease |17 février 2015 |29 février 2016 |
| 0.9.3-prelease |13 janvier 2015 |29 février 2016 |
| 0.9.2-prelease |19 décembre 2014 |29 février 2016 |
| 0.9.1-prelease |19 décembre 2014 |29 février 2016 |
| 0.9.0-prelease |10 décembre 2014 |29 février 2016 |

## <a name="faq"></a>Questions fréquentes (FAQ)
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>Voir aussi
Pour en savoir plus sur Cosmos DB, consultez la page du service [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).