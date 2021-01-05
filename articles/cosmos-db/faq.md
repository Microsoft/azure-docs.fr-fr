---
title: Forum aux questions sur les différentes API dans Azure Cosmos DB
description: Obtenez des réponses aux questions fréquemment posées sur Azure Cosmos DB, un service de base de données multimodèle distribué globalement. Découvrez la capacité, les niveaux de performances et la mise à l’échelle.
author: SnehaGunda
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/01/2019
ms.author: sngun
ms.custom: seodec18
ms.openlocfilehash: 81c959467c4bb3700120917cf150773fb18e866d
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359827"
---
# <a name="frequently-asked-questions-about-different-apis-in-azure-cosmos-db"></a>Questions fréquentes sur les différentes API dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

### <a name="what-are-the-typical-use-cases-for-azure-cosmos-db"></a>Quels sont les cas d’utilisation courants d’Azure Cosmos DB ?

Azure Cosmos DB est un choix judicieux pour les nouvelles applications web, mobiles, de jeu et IoT où une mise à l’échelle automatique, des performances prévisibles, des temps de réponse rapides de l’ordre d’une milliseconde et la capacité à exécuter des requêtes sur des données sans schéma sont importants. Azure Cosmos DB permet un développement rapide et prend en charge l’itération continue des modèles de données d’application. Les applications qui gèrent du contenu et des données générés par l’utilisateur sont [des cas d’utilisation courants pour Azure Cosmos DB](use-cases.md).

### <a name="how-does-azure-cosmos-db-offer-predictable-performance"></a>Comment Azure Cosmos DB offre-t-il des performances prévisibles ?

Dans Azure Cosmos DB, la mesure du débit est exprimée en [unités de requête](request-units.md) (RU). Un débit de 1 RU correspond au débit de la requête GET d’un document de 1 Ko. Chaque opération dans Azure Cosmos DB, y compris les lectures, les écritures, les requêtes SQL et les exécutions de procédures stockées, comporte une valeur de RU déterministe basée sur le débit nécessaire à l’exécution de l’opération. Au lieu de penser à l’UC, à l’E/S et à la mémoire, ainsi qu’à la façon dont ces éléments affectent le débit de votre application, vous pouvez penser en termes de mesure de RU unique.

Vous pouvez configurer chaque conteneur Azure Cosmos avec un débit provisionné en termes de RU de débit par seconde. Pour les applications de toute échelle, vous pouvez évaluer les requêtes individuelles pour mesurer leur valeur de RU, et approvisionner un conteneur pour gérer la totalité des unités de requête sur l’ensemble des requêtes. Vous pouvez également mettre à l’échelle le débit de votre conteneur à mesure de l’évolution des besoins de votre application. Pour plus d’informations sur les unités de requête et pour obtenir de l’aide afin de déterminer vos besoins en matière de conteneurs, essayez la [calculatrice de débit](https://www.documentdb.com/capacityplanner).

### <a name="how-does-azure-cosmos-db-support-various-data-models-such-as-keyvalue-columnar-document-and-graph"></a>Comment Azure Cosmos DB prend-il en charge différents modèles de données comme clé/valeur, en colonnes, document et graphe ?

Les modèles de données clé/valeur (table), en colonnes, document et graphe sont tous pris en charge de manière native en raison de la conception ARS (atomes, enregistrements et séquences) sur laquelle repose Azure Cosmos DB. Les atomes, enregistrements et séquences peuvent facilement être mappés et projetés vers différents modèles de données. Les API pour certains modèles sont déjà disponibles (SQL, MongoDB, Table et Gremlin), tandis que d’autres API propres à des modèles de données supplémentaires seront disponibles plus tard.

Azure Cosmos DB a un moteur d’indexation indépendant du point de vue du schéma capable d’indexer automatiquement toutes les données qu’il reçoit sans nécessiter de schéma ou d’index secondaire de la part du développeur. Le moteur s’appuie sur un ensemble de dispositions d’index logiques (inversée, en colonnes, arborescence) qui découplent la disposition de stockage des sous-systèmes de traitement de requêtes et d’index. Cosmos DB peut également prendre en charge un ensemble de protocoles de transmission et d’API de manière extensible, et les traduire efficacement vers le modèle de données principal (1) et les dispositions d’index logiques (2), ce qui en fait une solution unique capable de prendre en charge plus d’un modèle de données en mode natif.

### <a name="can-i-use-multiple-apis-to-access-my-data"></a>Puis-je utiliser plusieurs API pour accéder à mes données ?

Azure Cosmos DB est le service de base de données multi-modèle de Microsoft distribué à l’échelle mondiale. « Multimodèle » signifie qu’Azure Cosmos DB prend en charge plusieurs API et plusieurs modèles de données, et différentes API utilisent des formats de données différents pour le stockage et le protocole filaire. Par exemple, SQL utilise JSON, MongoDB utilise BSON, Table utilise EDM, Cassandra utilise CQL, Gremlin utilise le format JSON. Par conséquent, nous vous recommandons d’utiliser la même API pour tous les accès aux données d’un compte spécifique.

Chaque API fonctionne de manière indépendante, à l’exception de Gremlin et de l’API SQL, qui sont interopérables.

### <a name="is-azure-cosmos-db-hipaa-compliant"></a>Azure Cosmos DB est-il conforme à la loi HIPAA ?

Oui, Azure Cosmos DB est conforme à la loi HIPAA. HIPAA établit les conditions requises pour l’utilisation, la divulgation et la protection des informations de santé identifiables de façon individuelle. Pour plus d’informations, consultez le [Centre de gestion de la confidentialité de Microsoft](/compliance/regulatory/offering-hipaa-hitech).

### <a name="what-are-the-storage-limits-of-azure-cosmos-db"></a>Quelles sont les limites de stockage d’Azure Cosmos DB ?

Il n’existe aucune limite à la quantité totale de données qu’un conteneur peut stocker dans Azure Cosmos DB.

### <a name="what-are-the-throughput-limits-of-azure-cosmos-db"></a>Quelles sont les limites de débit d’Azure Cosmos DB ?

Il n’existe aucune limite au débit total qu’un conteneur peut prendre en charge dans Azure Cosmos DB. L’idée principale est de distribuer votre charge de travail à peu près uniformément entre un nombre suffisant de clés de partition.

### <a name="are-direct-and-gateway-connectivity-modes-encrypted"></a>Les modes de connectivité directe et de passerelle sont-ils chiffrés ?

Oui, les deux modes sont toujours entièrement chiffrés.

### <a name="how-much-does-azure-cosmos-db-cost"></a>Combien coûte Azure Cosmos DB ?

Pour plus d’informations, voir [Tarification d’Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/). Les frais d’utilisation d’Azure Cosmos DB sont déterminés par le nombre de conteneurs approvisionnés, le nombre d’heures durant lequel les conteneurs ont été en ligne et le débit approvisionné pour chaque conteneur.

### <a name="is-a-free-account-available"></a>Un compte gratuit est-il disponible ?

Oui, vous pouvez bénéficier d’un compte à durée limitée sans frais ni engagement. Pour y souscrire, accédez à la page [Essayez gratuitement Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) ou consultez les [questions fréquentes (FAQ) sur Azure Cosmos DB](#try-cosmos-db) pour en savoir plus.

Si vous débutez avec Azure, vous pouvez vous inscrire pour bénéficier d’un [compte Azure gratuit](https://azure.microsoft.com/free/), qui vous donne 30 jours et un crédit pour essayer tous les services Azure. Si vous avez un abonnement Visual Studio, vous pouvez aussi bénéficier de [crédits Azure gratuits](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) à utiliser sur n’importe quel service Azure.

Vous pouvez également utiliser l’[émulateur Azure Cosmos DB](local-emulator.md) pour développer et tester votre application localement, sans créer d’abonnement Azure et sans frais. Lorsque vous êtes satisfait du fonctionnement de votre application dans l’émulateur Azure Cosmos DB, vous pouvez commencer à utiliser un compte Azure Cosmos DB dans le cloud.

### <a name="how-can-i-get-additional-help-with-azure-cosmos-db"></a>Comment puis-je obtenir une aide supplémentaire avec Azure Cosmos DB ?

Pour poser une question technique, connectez-vous à l’un de ces forums de questions-réponses :

* [Page de questions Microsoft Q&A](/answers/topics/azure-cosmos-db.html)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-cosmosdb). Consultez Stack Overflow pour les questions sur la programmation. Par souci de clarté et afin d’obtenir une réponse, vérifiez que votre question est [appropriée](https://stackoverflow.com/help/on-topic) et [ fournit un maximum de détails](https://stackoverflow.com/help/how-to-ask).

Pour demander de nouvelles fonctionnalités, faites une nouvelle demande sur [User voice](https://feedback.azure.com/forums/263030-azure-cosmos-db).

Pour résoudre un problème relatif à votre compte, enregistrez une [demande de support](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) sur le portail Azure.

## <a name="try-azure-cosmos-db-subscriptions"></a><a id="try-cosmos-db"></a>Souscrire à l’offre Essayez gratuitement Azure Cosmos DB

Vous pouvez désormais profiter pendant une durée limitée de l’expérience Azure Cosmos DB, sans abonnement, libre de frais et d’engagement. Pour souscrire à l'offre Essayez gratuitement Azure Cosmos DB, accédez à la page [Essayez gratuitement Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) et utilisez n'importe quel compte Microsoft personnel (MSA). Cette souscription est indépendante de l’offre [Créez votre compte Azure gratuit](https://azure.microsoft.com/free/) et peut être couplée à celle-ci ou à un abonnement Azure payant.

Les souscriptions à l’offre Essayez gratuitement Azure Cosmos DB figurent dans le portail Azure aux côtés des autres abonnements associés à votre ID utilisateur.

Les souscriptions à l’offre Essayez gratuitement Azure Cosmos DB obéissent aux conditions suivantes :

* L'accès peut être octroyé aux comptes Microsoft personnels (MSA). Évitez d’utiliser des comptes Azure Active Directory (Azure AD) ou des comptes appartenant à des locataires Azure AD d’entreprise, car ceux-ci peuvent présenter des limitations susceptibles de bloquer l’octroi de l’accès.
* Un seul [conteneur avec débit provisionné](./set-throughput.md#set-throughput-on-a-container) par abonnement pour les comptes SQL, API Gremlin et Table.
* Jusqu’à trois [collections avec débit provisionné](./set-throughput.md#set-throughput-on-a-container) par abonnement pour les comptes MongoDB.
* Une [base de données avec débit provisionné](./set-throughput.md#set-throughput-on-a-database) par abonnement. Les bases de données avec débit provisionné peuvent inclure un nombre illimité de conteneurs.
* 10 Go de capacité de stockage.
* La réplication globale est disponible dans les [régions Azure](https://azure.microsoft.com/regions/) suivantes : USA Centre, Europe Nord et Asie Sud-Est
* Débit maximal de 5 000 RU/s lors de l’approvisionnement au niveau du conteneur.
* Débit maximal de 20 000 RU/s lors de l’approvisionnement au niveau de la base de données.
* Les abonnements expirent dans un délai de 30 jours et peuvent être prolongés pour une durée totale maximale de 31 jours. Une fois ce délai expiré, les informations contenues sont supprimées.
* Il n’est pas possible de créer des tickets de support Azure pour les comptes d’évaluation d’Azure Cosmos DB ; cependant, les abonnés titulaires de plans de support actifs peuvent bénéficier du support.

## <a name="set-up-azure-cosmos-db"></a>Configurer Azure Cosmos DB

### <a name="how-do-i-sign-up-for-azure-cosmos-db"></a>Comment s’inscrire pour Azure Cosmos DB ?

Azure Cosmos DB est disponible dans le portail Azure. Tout d’abord, souscrivez un abonnement Azure. Une fois que vous êtes inscrit, vous pouvez ajouter un compte Azure Cosmos DB à votre abonnement Azure.

### <a name="what-is-a-primary-key"></a>Qu’est-ce qu’une clé primaire ?

Une clé primaire est un jeton de sécurité permettant d’accéder à toutes les ressources d’un compte. Les personnes disposant de cette clé ont un accès en lecture et en écriture à toutes les ressources du compte de la base de données. Soyez prudent lorsque vous distribuez des clés primaires. La clé primaire et la clé secondaire sont disponibles dans le panneau **Clés** du [portail Azure][azure-portal]. Pour plus d’informations sur les clés, consultez la rubrique [Affichage, copie et régénération de clés d’accès](manage-with-cli.md#list-account-keys).

### <a name="what-are-the-regions-that-preferredlocations-can-be-set-to"></a>Quelles sont les régions configurables pour PreferredLocations ?

La valeur de PreferredLocations peut être définie sur toute région Azure dans laquelle Cosmos DB est disponible. Pour obtenir la liste des régions disponibles, voir [Régions Azure](https://azure.microsoft.com/regions/).

### <a name="is-there-anything-i-should-be-aware-of-when-distributing-data-across-the-world-via-the-azure-datacenters"></a>Y a-t-il quelque chose que je dois savoir concernant la distribution de données à travers le monde via les centres de données Azure ?

Azure Cosmos DB est présent dans toutes les régions Azure, comme l’indique la page [Régions Azure](https://azure.microsoft.com/regions/). Comme il s’agit du service principal, chaque nouveau centre de données a une présence Azure Cosmos DB.

Lorsque vous définissez une région, n’oubliez pas qu’Azure Cosmos DB respecte les clouds souverains et du secteur public. Autrement dit, si vous créez un compte dans une [région souveraine](https://azure.microsoft.com/global-infrastructure/), vous ne pouvez pas répliquer en dehors de cette [région souveraine](https://azure.microsoft.com/global-infrastructure/). De même, vous ne pouvez pas activer la réplication dans d’autres emplacements souverains à partir d’un compte externe.

### <a name="is-it-possible-to-switch-from-container-level-throughput-provisioning-to-database-level-throughput-provisioning-or-vice-versa"></a>Est-il possible de passer du provisionnement de débit au niveau du conteneur au provisionnement de débit au niveau de la base de données ? Et inversement ?

L’approvisionnement de débit au niveau du conteneur et de la base de données constitue des offres distinctes, et tout changement de l’un vers l’autre nécessite la migration de données de la source vers la destination. Cela signifie donc que vous devez créer une base de données ou un conteneur, puis migrer les données avec la [bibliothèque de l’exécuteur en bloc](bulk-executor-overview.md) ou [Azure Data Factory](../data-factory/connector-azure-cosmos-db.md).

### <a name="does-azure-cosmosdb-support-time-series-analysis"></a>Azure CosmosDB prend-il en charge l’analyse des séries chronologiques ?

Oui, Azure CosmosDB prend en charge l’analyse des séries chronologiques. Voici un exemple de [modèle de série chronologique](https://github.com/Azure/azure-cosmosdb-dotnet/tree/master/samples/Patterns). Cet exemple montre comment utiliser des flux de modification pour créer des affichages agrégés sur les données de série chronologique. Vous pouvez étendre cette approche à l’aide de Spark Streaming ou d’un autre processeur de données de flux.

## <a name="what-are-the-azure-cosmos-db-service-quotas-and-throughput-limits"></a>Quels sont les quotas de service Azure Cosmos DB et les limites de débit ?

Pour plus d’informations, consultez les articles sur les [quotas de service](concepts-limits.md) et les [limites de débit par conteneur et base de données](set-throughput.md#comparison-of-models) pour Azure Cosmos DB.

## <a name="frequently-asked-questions-about-sql-api"></a><a id="sql-api-faq"></a>Questions fréquentes (FAQ) sur l’API SQL

### <a name="how-do-i-start-developing-against-the-sql-api"></a>Comment commencer à développer par rapport l’API SQL ?

Vous devez d’abord souscrire un abonnement Microsoft Azure. Après avoir souscrit un abonnement Microsoft Azure, vous pouvez ajouter un conteneur d’API SQL à votre abonnement Azure. Pour savoir comment ajouter un compte Azure Cosmos DB, consultez [Créer un compte de base de données Azure Cosmos](create-sql-api-dotnet.md#create-account).

[Kits de développement logiciel (SDK)](sql-api-sdk-dotnet.md) sont disponibles pour .NET, Python, Node.js, JavaScript et Java. Les développeurs peuvent également utiliser les [API HTTP RESTful](/rest/api/cosmos-db/) pour interagir avec les ressources Azure Cosmos DB sur différentes plateformes et dans de nombreux langages.

### <a name="can-i-access-some-ready-made-samples-to-get-a-head-start"></a>Puis-je accéder à certains exemples prêts à l’emploi pour gagner du temps ?

Des exemples pour les SDK [.NET](sql-api-dotnet-samples.md), [Java](https://github.com/Azure/azure-documentdb-java), [Node.js](sql-api-nodejs-samples.md) et [Python](sql-api-python-samples.md) de l’API SQL sont disponibles sur GitHub.

### <a name="does-the-sql-api-database-support-schema-free-data"></a>La base de données de l’API SQL prend-elle en charge les données sans schéma ?

Oui, l’API SQL permet aux applications de stocker des documents JSON arbitraires sans définition ou conseil de schéma. Les données peuvent être interrogées immédiatement via l’interface de requête SQL d’Azure Cosmos DB.

### <a name="does-the-sql-api-support-acid-transactions"></a>L’API SQL prend-elle en charge les transactions ACID ?

Oui, l’API SQL prend en charge les transactions entre documents exprimées sous forme de procédures stockées et de déclencheurs JavaScript. Les transactions sont étendues à une seule partition au sein de chaque conteneur, et exécutées en mode « Tout ou rien » avec des sémantiques ACID, isolément d’autres codes et requêtes utilisateur s’exécutant simultanément. Si des exceptions surviennent lors de l’exécution du code d’application JavaScript côté serveur, la transaction entière est annulée. 

### <a name="what-is-a-container"></a>Qu’est-ce qu’un conteneur ?

Un conteneur est un groupe de documents accompagnés de leur logique d’application JavaScript. Un conteneur est une entité facturable, où le [coût](performance-levels.md) est déterminé par le débit et le stockage utilisé. Des conteneurs peuvent s’étendre à un ou plusieurs serveurs ou partitions, et peuvent être mis à l’échelle pour gérer des volumes de stockage ou de débit pratiquement illimités.

* Pour l’API SQL, la ressource est appelée conteneur.
* Pour les comptes d’API Azure Cosmos DB pour MongoDB, les conteneurs sont mappés vers des collections.
* Pour les comptes d’API Cassandra et Table, les conteneurs sont mappés vers des tables.
* Pour les comptes d’API Gremlin, les conteneurs sont mappés vers des graphes.

Les conteneurs constituent également les entités de facturation d’Azure Cosmos DB. Chaque conteneur est facturé à l’heure, sur la base du débit provisionné et de l’espace de stockage utilisé. Pour plus d’informations, consultez la [Tarification Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/).

### <a name="how-do-i-create-a-database"></a>Comment créer une base de données ?

Vous pouvez créer des bases de données à l’aide du [portail Azure](https://portal.azure.com) comme décrit dans [Ajouter un conteneur](create-sql-api-java.md#add-a-container), de l’un des [Kits de développement logiciel (SDK) Azure Cosmos DB](sql-api-sdk-dotnet.md) ou d’[API REST](/rest/api/cosmos-db/).

### <a name="how-do-i-set-up-users-and-permissions"></a>Comment configurer des utilisateurs et des autorisations ?

Vous pouvez créer des utilisateurs et des autorisations à l’aide de l’un des [SDK d’API Cosmos DB](sql-api-sdk-dotnet.md) ou des [API REST](/rest/api/cosmos-db/).

### <a name="does-the-sql-api-support-sql"></a>L’API SQL prend-elle en charge SQL ?

Le langage de requête SQL pris en charge par les comptes d’API SQL est un sous-ensemble amélioré des fonctionnalités de requête prises en charge par SQL Server. Le langage de requête SQL d’Azure Cosmos DB fournit des opérateurs hiérarchiques et relationnels enrichis ainsi qu’une extensibilité par le biais de fonctions JavaScript définies par l’utilisateur. La grammaire JSON permet de modéliser des documents JSON en tant qu’arborescences comprenant des nœuds étiquetés qui sont utilisés par les techniques d’indexation automatique et le langage de requête SQL d’Azure Cosmos DB. Pour plus d’informations sur l’utilisation de la grammaire SQL, consultez l’article [Requêtes SQL][query].

### <a name="does-the-sql-api-support-sql-aggregation-functions"></a>L’API SQL prend-elle en charge les fonctions d’agrégation SQL ?

L’API SQL prend en charge l’agrégation à faible latence à n’importe quelle échelle par l’intermédiaire des fonctions d’agrégation `COUNT`, `MIN`, `MAX`, `AVG` et `SUM` par le biais de la grammaire SQL. Pour plus d’informations, consultez [Fonctions d’agrégation](sql-query-aggregate-functions.md).

### <a name="how-does-the-sql-api-provide-concurrency"></a>Comment l’API SQL assure-t-elle l’accès concurrentiel ?

L’API SQL prend en charge le contrôle d’accès concurrentiel optimiste par le biais des balises d’entité HTTP (ou ETags). Chaque ressource de l’API SQL est dotée d’une ETag définie sur le serveur à chaque mise à jour d’un document. L’en-tête et la valeur actuelle ETag sont inclus dans tous les messages de réponse. Les ETag peuvent être utilisées avec l’en-tête If-Match pour permettre au serveur de déterminer si une ressource nécessite une mise à jour. La valeur If-Match est la valeur ETag utilisée pour la vérification. Si la valeur ETag correspond à la valeur ETag du serveur, la ressource est mise à jour. Si l’ETag n’est plus actuelle, le serveur rejette l’opération en retournant un code de réponse « HTTP 412 Échec de la condition préalable ». Dans ce cas, le client extrait à nouveau la ressource afin d’obtenir la valeur ETag actuelle pour la ressource. De plus, les ETag peuvent être utilisées avec l’en-tête If-None-Match pour déterminer si une nouvelle extraction d’une ressource est nécessaire.

Pour utiliser l’accès concurrentiel optimiste dans .NET, utilisez la classe [AccessCondition](/dotnet/api/microsoft.azure.documents.client.accesscondition) . Pour un exemple .NET, voir [Program.cs](https://github.com/Azure/azure-cosmos-dotnet-v2/blob/master/samples/code-samples/DocumentManagement/Program.cs) dans l’exemple DocumentManagement sur github.

### <a name="how-do-i-perform-transactions-in-the-sql-api"></a>Comment effectuer des transactions dans l’API SQL ?

L’API SQL prend en charge les transactions intégrées au langage par le biais de déclencheurs et de procédures stockées JavaScript. Toutes les opérations de base de données à l’intérieur des scripts sont exécutées en isolement de capture instantanée. S’il s’agit d’un conteneur à partition unique, l’exécution est étendue au conteneur. Si le conteneur est partitionné, l’exécution est étendue aux documents ayant la même valeur de clé de partition au sein du conteneur. Une capture instantanée des versions des documents (ETags) est prise au début de la transaction et validée uniquement si le script fonctionne. Si le JavaScript renvoie une erreur, la transaction est annulée. Pour plus d’informations, consultez [Programmation en JavaScript côté serveur pour Azure Cosmos DB](stored-procedures-triggers-udfs.md).

### <a name="how-can-i-bulk-insert-documents-into-cosmos-db"></a>Comment insérer des documents en bloc dans Cosmos DB ?

Vous pouvez insérer en bloc des documents dans Azure Cosmos DB de l’une des façons suivantes :

* L’outil de l’exécuteur en bloc, comme décrit dans [Utilisation de la bibliothèque .NET de l’exécuteur en bloc](bulk-executor-dot-net.md) et [Utilisation de la bibliothèque Java de l’exécuteur en bloc](bulk-executor-java.md)
* Avec l’outil de migration de données, comme décrit dans [Outil de migration de base de données pour Azure Cosmos DB](import-data.md).
* Avec des procédures stockées, comme décrit dans [Programmation en JavaScript côté serveur pour Azure Cosmos DB](stored-procedures-triggers-udfs.md).

### <a name="does-the-sql-api-support-resource-link-caching"></a>L’API SQL prend-elle en charge la mise en cache de lien de ressource ?

Oui. Azure Cosmos DB étant un service RESTful, les liens de ressource sont immuables et peuvent être mis en cache. Les clients de l’API SQL peuvent spécifier un en-tête « If-None-Match » pour des lectures effectuées en comparaison avec des documents ou conteneurs de type ressource, puis mettre à jour leurs copies locales après modification de la version du serveur.

### <a name="is-a-local-instance-of-sql-api-available"></a>Une instance locale de l’API SQL est-elle disponible ?

Oui. L’[émulateur Azure Cosmos DB](local-emulator.md) fournit une émulation haute fidélité du service Cosmos DB. Il prend en charge les mêmes fonctionnalités qu’Azure Cosmos DB, dont la création et l’interrogation de documents JSON, l’approvisionnement et la mise à l’échelle des collections, et l’exécution des procédures stockées et déclencheurs. Vous pouvez développer et tester des applications à l’aide de l’émulateur Azure Cosmos DB, puis les déployer vers Azure à une échelle globale en apportant une simple modification à la configuration du point de terminaison de connexion pour Azure Cosmos DB.

### <a name="why-are-long-floating-point-values-in-a-document-rounded-when-viewed-from-data-explorer-in-the-portal"></a>Pourquoi les longues valeurs à virgule flottante dans un document sont-elles arrondies quand elles sont affichées à partir de l’Explorateur de données dans le portail ?

Il s’agit d’une limitation propre à JavaScript. JavaScript utilise des nombres de format à virgule flottante double précision comme spécifié dans la norme IEEE 754. Les nombres qu’il peut contenir de la sorte sont compris entre -(2<sup>53</sup> - 1) et 2<sup>53</sup>-1 (c’est-à-dire, 9007199254740991) uniquement.

### <a name="where-are-permissions-allowed-in-the-object-hierarchy"></a>Où se trouvent les autorisations accordées dans la hiérarchie d’objets ?

La création d’autorisations à l’aide de ResourceTokens est autorisée au niveau du conteneur et de ses descendants (tels que les documents ou les pièces jointes). Cela implique qu’une tentative de création d’autorisation au niveau de la base de données ou du compte n’est pas autorisée actuellement.

[azure-portal]: https://portal.azure.com
[query]: ./sql-query-getting-started.md

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les questions fréquentes relatives à d’autres API, consultez :

* Questions fréquentes sur l’[API MongoDB d’Azure Cosmos DB](mongodb-api-faq.md)
* Questions fréquentes sur l’[API Gremlin dans Azure Cosmos DB](gremlin-api-faq.md)
* Questions fréquentes sur l’[API Cassandra dans Azure Cosmos DB](cassandra-faq.md)
* Questions fréquentes sur l’[API Table dans Azure Cosmos DB](table-api-faq.md)