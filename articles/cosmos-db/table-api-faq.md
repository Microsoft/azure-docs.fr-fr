---
title: Questions fréquentes (FAQ) sur l’API Table dans Azure Cosmos DB
description: Obtenir des réponses aux questions fréquentes sur l’API Table dans Azure Cosmos DB
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: conceptual
ms.date: 08/12/2020
ms.author: sngun
ms.openlocfilehash: c5404c026385f9ebf06c4f0289a4db0bf944d172
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93341231"
---
# <a name="frequently-asked-questions-about-the-table-api-in-azure-cosmos-db"></a>Questions fréquentes (FAQ) sur l’API Table dans Azure Cosmos DB
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

L’API Table dans Azure Cosmos DB est disponible dans le [portail Azure](https://portal.azure.com). Vous devez d’abord vous inscrire pour obtenir un abonnement Azure. Une fois inscrit, vous pouvez ajouter un compte d’API Table d’Azure Cosmos DB à votre abonnement Azure, puis ajouter des tables à votre compte. Vous trouverez les langages pris en charge et les démarrages rapides associés dans [Présentation de l’API Table d’Azure Cosmos DB](table-introduction.md).

## <a name="table-api-in-azure-cosmos-db-vs-azure-table-storage"></a><a id="table-api-vs-table-storage"></a>API Table dans Azure Cosmos DB et Stockage Table Azure

### <a name="where-is-table-api-not-identical-with-azure-table-storage-behavior"></a>Quelles sont les différences de comportement entre l’API Table et le stockage Table Azure ?

Si vous avez déjà créé des tables dans le stockage Table Azure et que vous souhaitez passer à l’API Table d’Azure Cosmos DB, tenez compte des différences de comportement suivantes :

* L’API Table d’Azure Cosmos DB utilise un modèle de capacité réservée pour garantir les performances. Que vous l’utilisez ou non, la capacité vous est donc facturée dès que la table est créée. Dans le cadre du stockage Table Azure, vous payez uniquement la capacité utilisée. Cela explique pourquoi l’API Table peut offrir un contrat de niveau de service (SLA) de 10 ms en lecture et de 15 ms en écriture au 99e centile, alors que le stockage Table Azure offre un SLA de 10 s. Même les tables vides sans requêtes de l’API Table engendrent donc des coûts pour assurer la disponibilité de la capacité et le traitement de toutes les requêtes selon les termes du SLA offert par Azure Cosmos DB.

* Les résultats des requêtes retournés par l’API Table ne sont pas triés dans l’ordre des clés de ligne/partition, alors qu’ils le sont dans le stockage Table Azure.

* Les clés de ligne ne peuvent pas dépasser 255 octets.

* Les lots ne peuvent contenir que 2 Mo au maximum.

* CORS n’est pas pris en charge.

* Les noms des tables ne respectent pas la casse dans le Stockage Table Azure, alors qu’ils la respectent dans l’API Table d’Azure Cosmos DB.

* Certains des formats internes Azure Cosmos DB pour les informations de codage, comme les champs binaires, ne sont actuellement pas aussi efficaces que souhaité. Par conséquent, des limitations inattendues sur la taille des données peuvent être observées. Par exemple, il n’est actuellement pas possible d’utiliser le méga-octet complet d’une entité de table pour stocker des données binaires, car l’encodage augmente la taille des données.

* L’« ID » du nom de propriété de l’entité n’est pas pris en charge.

* TableQuery TakeCount n’est pas limité à 1 000.

* En ce qui concerne l’API REST, il existe un certain nombre d’options liées aux points de terminaison/requêtes qui ne sont pas prises en charge par l’API Table d’Azure Cosmos DB :

  | Méthode(s) REST | Option de point de terminaison/requête REST | URL vers la documentation | Explication |
  | ------------| ------------- | ---------- | ----------- |
  | GET, PUT | `/?restype=service@comp=properties`| [Définir les propriétés du service Table](/rest/api/storageservices/set-table-service-properties) et [Obtenir les propriétés du service Table](/rest/api/storageservices/get-table-service-properties) | Ce point de terminaison est utilisé pour définir les règles CORS, la configuration de Storage Analytics et les paramètres de journalisation. CORS n’est pas actuellement pris en charge, et l’analytique et la journalisation sont gérées différemment dans Azure Cosmos DB et dans les tables Stockage Azure. |
  | OPTIONS | `/<table-resource-name>` | [Requête de table CORS préliminaire](/rest/api/storageservices/preflight-table-request) | Cette option fait partie de CORS, qui n’est pas pris en charge par Azure Cosmos DB pour l’instant. |
  | GET | `/?restype=service@comp=stats` | [Obtenir les statistiques du service Table](/rest/api/storageservices/get-table-service-stats) | Fournit des informations sur la rapidité de la réplication des données entre un emplacement principal et un emplacement secondaire. Cette option est inutile dans Cosmos DB puisque la réplication fait partie des écritures. |
  | GET, PUT | `/mytable?comp=acl` | [Obtenir la liste de contrôle d’accès de la table](/rest/api/storageservices/get-table-acl) et [Définir la liste de contrôle d’accès de la table](/rest/api/storageservices/set-table-acl) | Obtient et définit les stratégies d’accès stocké utilisées pour gérer les signatures d’accès partagé. Bien que ces signatures soient prises en charge, elles sont définies et gérées différemment. |

* L’API Table d’Azure Cosmos DB prend uniquement en charge le format JSON, pas le format ATOM.

* Si les signatures d‘accès partagé sont prises en charge par Azure Cosmos DB, certaines stratégie ne le sont pas, notamment celles liées aux opérations de gestion comme le droit de créer des tables.

* Pour le kit SDK .NET en particulier, certaines classes et méthodes ne sont pas prises en charge par Azure Cosmos DB pour l’instant.

  | Classe | Méthode non prise en charge |
  |-------|-------- |
  | CloudTableClient | \*ServiceProperties* |
  |                  | \*ServiceStats* |
  | CloudTable | SetPermissions* |
  |            | GetPermissions* |
  | TableServiceContext | * (cette classe est dépréciée) |
  | TableServiceContext | " " |
  | TableServiceExtensions | " " |
  | TableServiceContext | " " |

## <a name="other-frequently-asked-questions"></a>Autres questions fréquentes

### <a name="do-i-need-a-new-sdk-to-use-the-table-api"></a>Ai-je besoin d’un nouveau kit SDK pour utiliser l’API Table ?

Non, les kits SDK de stockage existants doivent continuer de fonctionner. Toutefois, nous vous recommandons de vous procurer les derniers kits SDK pour optimiser la prise en charge et, dans de nombreux cas, améliorer les performances. Consultez la liste des langages disponibles dans [Présentation de l’API Table d’Azure Cosmos DB](table-introduction.md).

### <a name="what-is-the-connection-string-that-i-need-to-use-to-connect-to-the-table-api"></a>Quelle chaîne de connexion faut-il utiliser pour se connecter à l’API Table ?

La chaîne de connexion est :

```
DefaultEndpointsProtocol=https;AccountName=<AccountNamefromCosmosDB;AccountKey=<FromKeysPaneofCosmosDB>;TableEndpoint=https://<AccountName>.table.cosmosdb.azure.com
```

Vous pouvez obtenir la chaîne de connexion dans la page Chaîne de connexion du portail Azure.

### <a name="how-do-i-override-the-config-settings-for-the-request-options-in-the-net-sdk-for-the-table-api"></a>Comment faire pour remplacer les paramètres de configuration des options de requête dans le kit SDK .NET pour l’API Table ?

Certains paramètres sont gérés sur la méthode de CreateCloudTableClient et d’autres par le biais du fichier app.config dans la section appSettings de l’application cliente. Pour plus d’informations sur les paramètres de configuration, voir [Fonctionnalités d’Azure Cosmos DB](tutorial-develop-table-dotnet.md).

### <a name="are-there-any-changes-for-customers-who-are-using-the-existing-azure-table-storage-sdks"></a>Y a-t-il des changements pour les clients qui utilisent les kits SDK de stockage Table Azure existants ?

Aucun. Il n’y a aucune modification pour les clients existants ou nouveaux utilisant le Kit de développement logiciel (SDK) de stockage Table Azure existant.

### <a name="how-do-i-view-table-data-thats-stored-in-azure-cosmos-db-for-use-with-the-table-api"></a>Comment faire pour afficher les données de table qui sont stockées dans Azure Cosmos DB afin de les utiliser avec l’API Table ?

Vous pouvez utiliser le portail Azure pour parcourir les données. Vous pouvez également utiliser le code ou les outils de l’API Table mentionnés dans la réponse suivante.

### <a name="which-tools-work-with-the-table-api"></a>Quels sont les outils qui fonctionnent avec l’API Table ?

Vous pouvez utiliser [l’Explorateur Stockage Azure](../vs-azure-tools-storage-manage-with-storage-explorer.md).

Les outils offrant la flexibilité nécessaire pour prendre une chaîne de connexion au format spécifié précédemment peuvent prendre en charge la nouvelle API Table. Vous trouverez une liste des outils de table dans la page [Outils clients d’Azure Storage](../storage/common/storage-explorers.md).

### <a name="is-the-concurrency-on-operations-controlled"></a>L’accès concurrentiel sur les opérations est-il contrôlé ?

Oui, l’accès concurrentiel optimiste est fourni via l’utilisation du mécanisme ETag.

### <a name="is-the-odata-query-model-supported-for-entities"></a>Le modèle de requête OData est-il pris en charge pour les entités ?

Oui, l’API Table prend en charge les requêtes OData et LINQ.

### <a name="can-i-connect-to-azure-table-storage-and-azure-cosmos-db-table-api-side-by-side-in-the-same-application"></a>Puis-je me connecter côte à côte au stockage Table Azure et à l’API Table d’Azure Cosmos DB dans la même application ?

Oui, vous pouvez vous connecter en créant deux instances distinctes de CloudTableClient, chacune pointant vers son propre URI via la chaîne de connexion.

### <a name="how-do-i-migrate-an-existing-azure-table-storage-application-to-this-offering"></a>Comment faire pour migrer une application de stockage Table Azure existante vers cette offre ?

[AzCopy](../storage/common/storage-use-azcopy-v10.md) et [l‘outil de migration de données Azure Cosmos DB](import-data.md) sont tous les deux pris en charge.

### <a name="how-is-expansion-of-the-storage-size-done-for-this-service-if-for-example-i-start-with-n-gb-of-data-and-my-data-will-grow-to-1-tb-over-time"></a>Comment l’extension de la taille de stockage est-elle gérée pour ce service, par exemple, si je commence par *n*  Go de données et que le volume de celles-ci passe à 1 To au fil du temps ?

Azure Cosmos DB est conçu pour offrir une capacité de stockage illimitée via l’utilisation de la mise à l’échelle horizontale. Le service peut surveiller votre stockage et en augmenter efficacement le volume.

### <a name="how-do-i-monitor-the-table-api-offering"></a>Comment faire pour surveiller l’offre de l’API Table ?

Le volet **Métriques** de l’API Table permet de surveiller les requêtes et l’utilisation du stockage.

### <a name="how-do-i-calculate-the-throughput-i-require"></a>Comment calculer le débit dont j’ai besoin ?

Vous pouvez utiliser l’estimateur de capacité pour calculer le débit de table requis pour les opérations. Pour plus d’informations, voir [Estimer les unités de requête et le stockage de données](https://www.documentdb.com/capacityplanner). En général, vous pouvez afficher votre entité au format JSON, et fournir les valeurs pour vos opérations.

### <a name="can-i-use-the-table-api-sdk-locally-with-the-emulator"></a>Puis-je utiliser le kit SDK de l’API Table localement avec l’émulateur ?

Pas pour l'instant.

### <a name="can-my-existing-application-work-with-the-table-api"></a>Mon application existante peut-elle fonctionner avec l’API Table ?

Oui, la même API est prise en charge.

### <a name="do-i-need-to-migrate-my-existing-azure-table-storage-applications-to-the-sdk-if-i-dont-want-to-use-the-table-api-features"></a>Dois-je migrer mes applications de stockage Table Azure existantes vers le nouveau kit SDK si je ne souhaite pas utiliser les fonctionnalités de l’API Table ?

Non, vous pouvez créer et utiliser des ressources de stockage Table Azure existantes sans la moindre interruption. Toutefois, si vous n’utilisez pas l’API Table, vous ne pouvez pas bénéficier de l’indexation automatique, de l’option de cohérence supplémentaire ou de la distribution mondiale.

### <a name="how-do-i-add-replication-of-the-data-in-the-table-api-across-more-than-one-region-of-azure"></a>Comment ajouter la réplication des données de l’API Table dans plusieurs régions Azure ?

Vous pouvez utiliser les [paramètres de réplication globale](tutorial-global-distribution-sql-api.md#portal) du portail Azure Cosmos DB pour ajouter des régions appropriées pour votre application. Pour développer une application distribuée globalement, vous devez également ajouter votre application avec les informations PreferredLocation définies sur la région locale afin de bénéficier d’une faible latence de lecture.

### <a name="how-do-i-change-the-primary-write-region-for-the-account-in-the-table-api"></a>Comment faire pour changer la région d’écriture principale du compte dans l’API Table ?

Vous pouvez utiliser le volet du portail de réplication globale d’Azure Cosmos DB pour ajouter une région, puis basculer vers la région requise. Pour des instructions, voir [Développement avec des comptes Azure Cosmos DB multirégions](high-availability.md).

### <a name="how-do-i-configure-my-preferred-read-regions-for-low-latency-when-i-distribute-my-data"></a>Comment configurer mes régions de lecture préférées pour une faible latence lors de la distribution de mes données ?

Pour faciliter la lecture à partir de l’emplacement local, utilisez la clé PreferredLocation dans le fichier app.config. Pour les applications existantes, l’API Table génère une erreur si LocationMode est défini. Supprimez ce code, car l’API Table extrait ces informations du fichier app.config. 

### <a name="how-should-i-think-about-consistency-levels-in-the-table-api"></a>En quoi consistent les niveaux de cohérence dans l’API Table ?

Azure Cosmos DB adopte des compromis bien pensés entre cohérence, disponibilité et latence. Azure Cosmos DB offre cinq niveaux de cohérence aux développeurs de l’API Table. Vous pouvez donc choisir le modèle de cohérence approprié au niveau de la table, et effectuer des demandes individuelles lors de l’interrogation des données. Quand un client se connecte, il peut spécifier un niveau de cohérence. Vous pouvez changer le niveau à l’aide de l’argument consistencyLevel de CreateCloudTableClient.

L’API Table permet d’effectuer des lectures à faible latence avec l’option « Lire vos écritures » et le niveau de cohérence Obsolescence limitée par défaut. Pour plus d’informations, consultez [Niveaux de cohérence](consistency-levels.md).

Par défaut, Stockage de table Azure offre une Cohérence forte au sein d’une région et une Cohérence éventuelle dans les sites secondaires.

### <a name="does-azure-cosmos-db-table-api-offer-more-consistency-levels-than-azure-table-storage"></a>L’API Table d’Azure Cosmos DB offre-t-elle davantage de niveaux de cohérence que le stockage Table Azure ?

Oui, pour plus d’informations sur la façon de tirer parti de la nature distribuée d’Azure Cosmos DB, voir [Niveaux de cohérence](consistency-levels.md). Des garanties étant fournies pour les niveaux de cohérence, vous pouvez utiliser ceux-ci en toute confiance.

### <a name="when-global-distribution-is-enabled-how-long-does-it-take-to-replicate-the-data"></a>Une fois la distribution globale activée, combien de temps faut-il pour répliquer les données ?

Azure Cosmos DB valide les données durablement dans la région locale, et envoie les données vers les autres régions immédiatement, en quelques millisecondes. Cette réplication dépend uniquement de la durée des boucles (RTT) du centre de données. Pour en savoir plus sur la fonctionnalité de distribution globale d’Azure Cosmos DB, voir [Azure Cosmos DB : Un service de base de données mondialement distribué sur Azure](distribute-data-globally.md).

### <a name="can-the-read-request-consistency-level-be-changed"></a>Est-il possible de modifier le niveau cohérence des demandes de lecture ?

Azure Cosmos DB permet de définir le niveau de cohérence au niveau du conteneur (sur la table). Le kit SDK .NET vous permet de changer le niveau en fournissant la valeur de la clé TableConsistencyLevel dans le fichier app.config. Les valeurs possibles sont les suivantes : Forte, Obsolescence limitée, Session, Préfixe cohérent et Éventuelle. Pour plus d’informations, voir [Niveaux de cohérence ajustables dans Azure Cosmos DB](consistency-levels.md). L’idée clé est que vous ne pouvez pas définir un niveau de cohérence de requête HTTP supérieur à celui défini pour la table. Par exemple, vous ne pouvez pas définir le niveau de cohérence pour la table sur Éventuelle et le niveau de cohérence de requête sur Forte.

### <a name="how-does-the-table-api-handle-failover-if-a-region-goes-down"></a>Comment l’API Table gère-t-elle le basculement en cas de défaillance d’une région ?

L’API Table tire parti de la plateforme mondialement distribuée d’Azure Cosmos DB. Pour vous assurer que votre application peut tolérer des temps d’arrêt de centre de données, activez au moins une région supplémentaire pour le compte dans le portail Azure Cosmos DB (voir [Développement avec des comptes Azure Cosmos DB multirégions](high-availability.md)). Vous pouvez définir la priorité de la région à l’aide du portail (voir [Developing with multi-region Azure Cosmos DB accounts](high-availability.md) (Développement avec des comptes Azure Cosmos DB multirégions)).

Vous pouvez ajouter autant de régions que vous le souhaitez pour le compte, et contrôler l’emplacement de basculement en indiquant une priorité. Pour utiliser la base de données, vous devez également fournir une application à cet emplacement. Si vous procédez de la sorte, vos clients ne subissent aucun temps d’arrêt. À la différence des autres kits SDK, la [dernière version du kit SDK client .NET](table-sdk-dotnet.md) prend en charge l’hébergement automatique. Autrement dit, il peut détecter la région défaillante et basculer automatiquement vers la nouvelle région.

### <a name="is-the-table-api-enabled-for-backups"></a>L’API Table prend-elle en charge les sauvegardes ?

Oui, l’API Table tire parti de la plateforme d’Azure Cosmos DB pour les sauvegardes. Les sauvegardes sont effectuées automatiquement. Pour plus d’informations, voir [Sauvegarde et restauration en ligne automatiques avec Azure Cosmos DB](online-backup-and-restore.md).

### <a name="does-the-table-api-index-all-attributes-of-an-entity-by-default"></a>L’API Table indexe-t-elle par défaut tous les attributs d’une entité ?

Oui, tous les attributs d’une entité sont indexés par défaut. Pour plus d’informations, voir [Azure Cosmos DB : Stratégies d’indexation](index-policy.md).

### <a name="does-this-mean-i-dont-have-to-create-more-than-one-index-to-satisfy-the-queries"></a>Cela signifie-t-il que je n’ai pas besoin de créer plusieurs index pour satisfaire les requêtes ?

Oui, l’API Table d’Azure Cosmos DB assure l’indexation automatique de tous les attributs sans aucune définition de schéma. Cette automatisation permet aux développeurs de se concentrer sur l’application plutôt que sur la création et la gestion d’index. Pour plus d’informations, voir [Azure Cosmos DB : Stratégies d’indexation](index-policy.md).

### <a name="can-i-change-the-indexing-policy"></a>Puis-je modifier la stratégie d’indexation ?

Oui, vous pouvez modifier la stratégie d’indexation en fournissant la définition d’index. Vous devez correctement encoder et placer dans une séquence d’échappement les paramètres.

Pour les kits SDK non-.NET, la stratégie d’indexation peut uniquement être définie dans le portail au niveau de l’ **Explorateur de données**. Naviguez jusqu’à la table à changer, accédez à **Mise à l‘échelle et paramètres**  -> Stratégie d‘indexation, effectuez le changement souhaité, puis cliquez sur **Enregistrer**.

À partir du kit SDK .NET, vous pouvez la soumettre dans le fichier app.config :

```JSON
{
  "indexingMode": "consistent",
  "automatic": true,
  "includedPaths": [
    {
      "path": "/somepath",
      "indexes": [
        {
          "kind": "Range",
          "dataType": "Number",
          "precision": -1
        },
        {
          "kind": "Range",
          "dataType": "String",
          "precision": -1
        }
      ]
    }
  ],
  "excludedPaths":
[
 {
      "path": "/anotherpath"
 }
]
}
```

### <a name="azure-cosmos-db-as-a-platform-seems-to-have-lot-of-capabilities-such-as-sorting-aggregates-hierarchy-and-other-functionality-will-you-be-adding-these-capabilities-to-the-table-api"></a>Azure Cosmos DB en tant que plateforme semble avoir de nombreuses fonctionnalités, telles que le tri, l’agrégation, la hiérarchisation et autres. Allez-vous ajouter ces fonctionnalités à l’API Table ?

L’API Table offre les mêmes fonctionnalités de requête que le stockage Table Azure. Azure Cosmos DB prend également en charge le tri, les agrégats, les requêtes géospatiales, les hiérarchies et un large éventail de fonctions intégrées. Pour plus d’informations, consultez la section [Requêtes SQL](./sql-query-getting-started.md).

### <a name="when-should-i-change-tablethroughput-for-the-table-api"></a>Quand dois-je changer le débit de table (TableThroughput) pour l’API Table ?

Vous devez modifier débit de table (TableThroughput) dans les situations suivantes :

* Vous effectuez une opération d’extraction, transformation et chargement (ETL) de données, ou souhaitez charger une grande quantité de données dans un laps de temps court.
* Vous avez besoin d’un débit supérieur du conteneur ou d’un ensemble de conteneurs sur le backend. Par exemple, vous voyez que le débit utilisé est supérieur au débit provisionné, et vous êtes limité. Pour plus d’informations, consultez [Définir le débit des conteneurs Azure Cosmos](set-throughput.md).

### <a name="can-i-scale-up-or-scale-down-the-throughput-of-my-table-api-table"></a>Puis-je augmenter ou réduire le débit ma table d’API Table ?

Oui, vous pouvez utiliser le volet de mise à l’échelle du portail Azure Cosmos DB pour régler le débit. Pour plus d’informations, voir [Définir le débit](set-throughput.md).

### <a name="is-a-default-tablethroughput-set-for-newly-provisioned-tables"></a>Un débit de table (TableThroughput) par défaut est-il défini pour les nouvelles tables approvisionnées ?

Oui, si vous ne remplacez pas le débit de table (TableThroughput) dans le fichier app.config et n’utilisez pas de conteneur créé préalablement dans Azure Cosmos DB, le service crée une table dont le débit est défini sur 400.

### <a name="is-there-any-change-of-pricing-for-existing-customers-of-the-azure-table-storage-service"></a>Les prix varient-ils pour les clients existants du service de stockage Table Azure ?

Aucun. Le tarif reste inchangé pour les clients existants du stockage Table Azure.

### <a name="how-is-the-price-calculated-for-the-table-api"></a>Comment le prix est-il calculé pour l’API Table ?

Cela dépend du débit de table (TableThroughput) alloué.

### <a name="how-do-i-handle-any-rate-limiting-on-the-tables-in-table-api-offering"></a>Comment faire pour gérer les limitations de taux qui affectent les tables dans l’offre API Table ?

Si le taux de requêtes dépasse la capacité du débit provisionné pour le conteneur sous-jacent ou un ensemble de conteneurs, vous recevez une erreur et le SDK essaie de renouveler l’appel en appliquant la stratégie de nouvelle tentative.

### <a name="why-do-i-need-to-choose-a-throughput-apart-from-partitionkey-and-rowkey-to-take-advantage-of-the-table-api-offering-of-azure-cosmos-db"></a>Pourquoi dois-je choisir un débit en dehors de PartitionKey et de RowKey pour tirer parti de l’offre API Table d’Azure Cosmos DB ?

Si vous ne spécifiez pas de débit par défaut pour votre conteneur dans le fichier app.config ou par le biais du portail, Azure Cosmos DB en définit un.

Azure Cosmos DB fournit des garanties de performances et de latence avec des limites supérieures pour les opérations. Cette garantie est possible lorsque le moteur peut assurer la gouvernance des opérations du client. La définition du débit de table (TableThroughput) est pour vous l’assurance de bénéficier du débit et de la latence garantis, car la plateforme réserve cette capacité et garantit le succès des opérations.

La spécification du débit vous permet de modifier celui-ci en souplesse pour tirer parti du caractère saisonnier de votre application, répondre aux besoins de débit et réduire les coûts.

### <a name="azure-table-storage-has-been-inexpensive-for-me-because-i-pay-only-to-store-the-data-and-i-rarely-query-the-azure-cosmos-db-table-api-offering-seems-to-be-charging-me-even-though-i-havent-performed-a-single-transaction-or-stored-anything-can-you-explain"></a>Le stockage Table Azure était peu onéreux pour moi, car je payais uniquement le stockage des données et n’effectuais que rarement des requêtes. L’offre API Table d’Azure Cosmos DB semble me facturer des frais alors que je n’ai pas effectué la moindre transaction ou stocké quoi que ce soit. Pouvez-vous m’expliquer pourquoi ?

Azure Cosmos DB est conçu pour être un système distribué globalement basé sur un contrat de niveau de service (SLA) offrant des garanties en matière de disponibilité, de latence et de débit. Le débit que vous réservez dans Azure Cosmos DB est garanti, à la différence du débit d’autres systèmes. Azure Cosmos DB offre des fonctionnalités supplémentaires qui ont été demandées par les clients, telles que les index secondaires et la distribution globale.

### <a name="i-never-get-a-quota-full-notification-indicating-that-a-partition-is-full-when-i-ingest-data-into-azure-table-storage-with-the-table-api-i-do-get-this-message-is-this-offering-limiting-me-and-forcing-me-to-change-my-existing-application"></a>Je ne reçois jamais de notification de « quota complet » (indiquant qu’une partition est pleine) lorsque j’ingère des données dans le stockage Table Azure. Avec l’API Table, je reçois ce message. Cette offre me limite-t-elle et m’oblige-t-elle à modifier mon application existante ?

Azure Cosmos DB est un système basé sur un contrat de niveau de service (SLA) permettant une mise à l’échelle illimitée, avec des garanties en matière de latence, de débit, de disponibilité et de cohérence. Pour être certain de bénéficier des performances premium garanties, assurez-vous que la taille de vos données et votre index sont gérables et extensibles. La limite de 20 Go appliquée au nombre d’entités ou d’éléments par clé de partition nous permet de garantir des performances de recherche et de requête exceptionnelles. Pour vous assurer que la mise à l’échelle de votre application fonctionne correctement, même pour le stockage Azure, nous vous recommandons de ne *pas* créer de partition sensible en stockant toutes les informations dans une seule partition et en interrogeant celle-ci.

### <a name="so-partitionkey-and-rowkey-are-still-required-with-the-table-api"></a>Si je comprends bien, une clé de partition et une clé de ligne sont toujours exigées avec l’API Table ?

Oui. Comme la surface d’exposition de l’API Table est semblable à celle du kit SDK Stockage Table Azure, la clé de partition constitue un moyen efficace de distribuer les données. La clé de ligne est unique au sein de cette partition. La clé de ligne doit être présente et ne peut pas avoir la valeur null comme dans le Kit de développement logiciel (SDK) standard. La longueur de la clé de ligne est de 255 octets, et celle de la clé de partition de 1 kilo-octet.

### <a name="what-are-the-error-messages-for-the-table-api"></a>Quels sont les messages d’erreur relatifs à l’API Table ?

Étant donné que le stockage Table Azure et l’API Table d’Azure Cosmos DB utilisent les mêmes SDK, la plupart des erreurs sont identiques.

### <a name="why-do-i-get-throttled-when-i-try-to-create-lot-of-tables-one-after-another-in-the-table-api"></a>Pourquoi suis-je limité quand je tente de créer successivement un grand nombre de tables dans l’API Table ?

Azure Cosmos DB est un système basé sur un contrat de niveau de service (SLA) qui offre des garanties en matière de latence, de débit, de disponibilité et de cohérence. Comme il s’agit d’un système provisionné, il réserve des ressources afin de garantir le respect de ces exigences. La vitesse de création des tables est détectée et limitée. Nous vous recommandons de consulter le taux de création de tables et de le réduire à moins de 5 par minute. N’oubliez pas que l’API Table est un système provisionné. Dès que vous le provisionnez, vous commencez à payer.

### <a name="how-do-i-provide-feedback-about-the-sdk-or-bugs"></a>Comment formuler des commentaires sur le Kit de développement logiciel (SDK) ou des bogues ?

Vous pouvez partager vos commentaires par les biais suivants :

* [User voice](https://feedback.azure.com/forums/263030-azure-cosmos-db)
* [Page de questions Microsoft Q&R](/answers/topics/azure-cosmos-db.html)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-cosmosdb). Consultez Stack Overflow pour les questions sur la programmation. Par souci de clarté et afin d’obtenir une réponse, vérifiez que votre question est [appropriée](https://stackoverflow.com/help/on-topic) et [ fournit un maximum de détails](https://stackoverflow.com/help/how-to-ask).

## <a name="next-steps"></a>Étapes suivantes

* [Générer une application API Table avec le kit SDK .NET et Azure Cosmos DB](create-table-dotnet.md)
* [Générer une application Java pour gérer les données de l’API Table d’Azure Cosmos DB](create-table-java.md)