---
title: Qu’est-ce que le magasin analytique Azure Cosmos DB ?
description: Découvrez le magasin transactionnel (basé sur des lignes) et analytique (basé sur des colonnes) d’Azure Cosmos DB. Avantages du magasin analytique, impact sur les performances pour les charges de travail à grande échelle et synchronisation automatique des données d’un magasin transactionnel à un magasin analytique
author: Rodrigossz
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 11/30/2020
ms.author: rosouz
ms.custom: seo-nov-2020
ms.openlocfilehash: 5dc233348188791404f826870b235d2bdfa4c202
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452848"
---
# <a name="what-is-azure-cosmos-db-analytical-store"></a>Qu’est-ce que le magasin analytique Azure Cosmos DB ?
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

Le magasin analytique Azure Cosmos DB est un magasin de colonnes totalement isolé qui permet d’effectuer des traitements analytiques à grande échelle sur les données opérationnelles de votre base de données Azure Cosmos DB sans incidence sur vos charges de travail transactionnelles. 

Le magasin transactionnel Azure Cosmos DB est indépendant des schémas et il vous permet d’itérer sur vos applications transactionnelles sans avoir à vous soucier de la gestion des schémas ou des index. À l’inverse, le magasin analytique Azure Cosmos DB est schématisé pour optimiser les performances des requêtes analytiques. Cet article décrit en détail le stockage analytique.

## <a name="challenges-with-large-scale-analytics-on-operational-data"></a>Défis liés à l’analytique à grande échelle des données opérationnelles

Les données opérationnelles multi-modèles dans un conteneur de Azure Cosmos DB sont stockées en interne dans un « magasin transactionnel » basé sur des lignes indexées. Le format du magasin de lignes est conçu pour permettre des requêtes opérationnelles ainsi que des lectures et des écritures transactionnelles rapides avec des temps de réponse de l’ordre de la milliseconde. Si votre jeu de données devient volumineux, les requêtes analytiques complexes peuvent être coûteuses en termes de débit approvisionné sur les données stockées dans ce format. Une consommation élevée de débit approvisionné a également un impact sur les performances des charges de travail transactionnelles utilisées par vos applications et services en temps réel.

Traditionnellement, pour analyser de grandes quantités de données, les données opérationnelles sont extraites du magasin transactionnel de Azure Cosmos DB et stockées dans une couche de données distincte. Par exemple, les données sont stockées dans un entrepôt de données ou un lac de données dans un format approprié. Ces données sont ensuite utilisées pour l’analytique à grande échelle et analysées à l’aide du moteur de calcul, comme les clusters Apache Spark. Cette séparation des couches de calcul et de stockage analytique des données opérationnelles entraîne une latence supplémentaire, car les pipelines ETL (Extract, Transform, Load) sont exécutés moins fréquemment pour réduire l’impact potentiel sur vos charges de travail transactionnelles.

Les pipelines ETL deviennent également complexes lors du traitement des mises à jour des données opérationnelles par rapport au traitement des données opérationnelles nouvellement reçues. 

## <a name="column-oriented-analytical-store"></a>Magasin analytique orienté colonne

Le magasin analytique Azure Cosmos DB traite les défis de complexité et de latence qui se produisent avec les pipelines ETL traditionnels. Le magasin analytique Azure Cosmos DB peut automatiquement synchroniser vos données opérationnelles dans un magasin en colonnes séparé. Le format du magasin de colonnes est approprié pour les requêtes analytiques à grande échelle qui sont exécutées de manière optimisée, ce qui permet d’améliorer la latence de ces requêtes.

À l’aide d’Azure Synapse Link, vous pouvez désormais créer des solutions HTAP non ETL en établissant une liaison directe avec le magasin analytique Azure Cosmos DB à partir d’Azure Synapse Analytics. Elle vous permet d’exécuter des analyses à grande échelle en temps quasi réel sur vos données opérationnelles.

## <a name="features-of-analytical-store"></a>Fonctionnalités du magasin analytique 

Lorsque vous activez le magasin analytique sur un conteneur Azure Cosmos DB, un nouveau magasin de colonnes est créé en interne en fonction des données opérationnelles de votre conteneur. Ce magasin de colonnes est maintenu séparément du magasin transactionnel en lignes pour ce conteneur. Les insertions, les mises à jour et les suppressions apportées à vos données opérationnelles sont automatiquement synchronisées avec le magasin analytique. Vous n’avez pas besoin du flux de modification ou de l’ETL pour synchroniser les données.

### <a name="column-store-for-analytical-workloads-on-operational-data"></a>Magasin de colonnes pour les charges de travail analytiques sur les données opérationnelles

Les charges de travail analytiques impliquent généralement des agrégations et des analyses séquentielles de champs sélectionnés. En stockant les données dans un ordre colonne-principal, le magasin analytique permet de sérialiser un groupe de valeurs pour chaque champ. Ce format réduit le nombre d’IOPS nécessaires pour analyser ou calculer des statistiques sur des champs spécifiques. Il améliore considérablement les temps de réponse des requêtes pour les analyses sur de grands jeux de données. 

Par exemple, si vos tables opérationnelles sont au format suivant :

:::image type="content" source="./media/analytical-store-introduction/sample-operational-data-table.png" alt-text="Exemple de table opérationnelle" border="false":::

Le magasin de lignes conserve les données ci-dessus dans un format sérialisé par ligne sur le disque. Ce format permet des lectures, des écritures et des requêtes opérationnelles plus rapides, faisant suite une demande de retour d’informations sur un produit spécifique, par exemple. Toutefois, au fur et à mesure que la taille du jeu de données augmente, l’exécution de requêtes analytiques complexes sur les données peut s’avérer plus coûteuse. Par exemple, si vous souhaitez obtenir « les tendances des ventes d’un produit de la catégorie "Équipement" dans différentes unités commerciales et sur différents mois », vous devrez lancer une requête complexe. Les analyses de grande envergure sur ce jeu de données peuvent être coûteuses en termes de débit approvisionné et avoir également un impact sur les performances des charges de travail transactionnelles qui alimentent vos applications et services en temps réel.

Le magasin analytique, qui est un magasin de colonnes, est mieux adapté à ce type de requêtes car il sérialise ensemble des champs de données similaires et réduit le nombre d’IOPS sur le disque.

L’image suivante représente le magasin de lignes transactionnelles et le magasin de colonnes analytiques dans Azure Cosmos DB :

:::image type="content" source="./media/analytical-store-introduction/transactional-analytical-data-stores.png" alt-text="Magasin de lignes transactionnelles et magasin de lignes analytiques dans Azure Cosmos DB" border="false":::

### <a name="decoupled-performance-for-analytical-workloads"></a>Performances découplées pour les charges de travail analytiques

Les requêtes analytiques n’ont aucune incidence sur les performances de vos charges de travail transactionnelles, car le magasin analytique est distinct du magasin transactionnel.  Le magasin analytique n’a pas besoin d’allouer des unités de requête distinctes.

### <a name="auto-sync"></a>Synchronisation automatique

La synchronisation automatique fait référence à la fonctionnalité complètement managée d’Azure Cosmos DB où les insertions, les mises à jour, les suppressions de données opérationnelles sont automatiquement synchronisées à partir du magasin transactionnel vers le magasin analytique en temps quasi-réel en temps quasi-réel. La latence de synchronisation automatique est généralement en moins de 2 minutes. Dans le cas d’une base de données de débit partagé avec un grand nombre de conteneurs, la latence de la synchronisation automatique des conteneurs individuels peut être supérieure et prendre jusqu’à 5 minutes. Nous aimerions en savoir plus, pour savoir si cette latence est adaptée à vos scénarios. Pour cela, contactez l’équipe [Azure Cosmos DB](mailto:cosmosdbsynapselink@microsoft.com).

La fonctionnalité de synchronisation automatique et le magasin analytique offrent les principaux avantages suivants :

#### <a name="scalability--elasticity"></a>Extensibilité et élasticité

En utilisant le partitionnement horizontal, le magasin transactionnel Azure Cosmos DB peut mettre à l’échelle de manière élastique le stockage et le débit sans temps d’arrêt. Le partitionnement horizontal dans le magasin transactionnel est évolutif et élastique lors de la synchronisation automatique afin de garantir la synchronisation des données avec le magasin analytique quasiment en temps réel. La synchronisation des données se produit quel que soit le débit de trafic transactionnel, qu’il s’agisse de 1 000 opérations/s ou 1 million d’opérations/s, et elle n’a pas d’impact sur le débit approvisionné dans le magasin transactionnel. 

#### <a name="automatically-handle-schema-updates"></a><a id="analytical-schema"></a>Gérer automatiquement les mises à jour de schéma

Le magasin transactionnel Azure Cosmos DB est indépendant des schémas et il vous permet d’itérer sur vos applications transactionnelles sans avoir à vous soucier de la gestion des schémas ou des index. À l’inverse, le magasin analytique Azure Cosmos DB est schématisé pour optimiser les performances des requêtes analytiques. Grâce à la capacité de synchronisation automatique, Azure Cosmos DB gère l’inférence du schéma sur les dernières mises à jour du magasin transactionnel.  Il gère aussi la représentation du schéma dans le magasin analytique, qui comprend la gestion des types de données imbriqués.

À mesure que votre schéma évolue et que de nouvelles propriétés sont ajoutées au fil du temps, le magasin analytique présente automatiquement un schéma uni dans tous les schémas historiques du magasin transactionnel.

##### <a name="schema-constraints"></a>Contraintes de schéma

Les contraintes suivantes s’appliquent aux données opérationnelles dans Azure Cosmos DB lorsque vous activez le magasin analytique pour inférer et représenter automatiquement le schéma correctement :

* Vous disposez d’un maximum de 200 propriétés à n’importe quel niveau d’imbrication dans le schéma et une profondeur maximale d’imbrication de 5.
  
  * Un élément avec 201 propriétés au niveau supérieur n’est pas conforme à cette contrainte et n’est donc pas représenté dans le magasin analytique.
  * Un élément avec plus de cinq niveaux imbriqués dans le schéma ne satisfait pas non plus cette contrainte et ne sera donc pas représenté dans le magasin analytique. Par exemple, l’élément suivant ne remplit pas la condition requise :

     `{"level1": {"level2":{"level3":{"level4":{"level5":{"too many":12}}}}}}`

* Les noms de propriété doivent être uniques en cas de comparaison ne tenant pas compte de la casse. Par exemple, les éléments suivants ne satisfont pas cette contrainte et ne seront donc pas représentés dans le magasin analytique :

  `{"Name": "fred"} {"name": "john"}` – « Name » et « name » sont les identique en cas de comparaison ne tenant pas compte de la casse.

##### <a name="schema-representation"></a>Représentation du schéma

Il existe deux modes de représentation de schéma dans le magasin analytique. Ces modes présentent des compromis entre la simplicité d’une représentation en colonnes, la gestion des schémas polymorphes et la simplicité de l’expérience de requête :

* Représentation de schéma bien définie
* Représentation de schéma avec une fidélité optimale

> [!NOTE]
> Pour les comptes d’API SQL (Core), lorsque le magasin analytique est activé, la représentation de schéma par défaut dans le magasin analytique est bien définie. Pour l’API Azure Cosmos DB pour les comptes MongoDB, la représentation de schéma par défaut dans le magasin analytique est une représentation de schéma de fidélité optimale. Si vous avez des scénarios nécessitant une représentation de schéma différente de l’offre par défaut pour chacune de ces API, contactez l’[équipe Azure Cosmos DB](mailto:cosmosdbsynapselink@microsoft.com) pour l’activer.

**Représentation de schéma bien définie**

La représentation de schéma bien définie crée une représentation tabulaire simple des données indépendantes du schéma dans le magasin transactionnel. La représentation de schéma bien définie prend en compte les considérations suivantes :

* Une propriété a toujours le même type sur plusieurs éléments.

  * Par exemple, `{"a":123} {"a": "str"}` n’a pas de schéma bien défini, car `"a"` est parfois une chaîne et parfois un nombre. Dans ce cas, le magasin analytique inscrit le type de données `“a”` en tant que type de données `“a”` dans l’élément au début de la durée de vie du conteneur. Les éléments pour lesquels le type de données de `“a”` diffère ne sont pas inclus dans le magasin analytique.
  
    Cette condition ne s’applique pas aux propriétés NULL. Par exemple, `{"a":123} {"a":null}` est encore bien défini.

* Les types de tableau doivent contenir un type répété unique.

  * Par exemple, `{"a": ["str",12]}` n’est pas un schéma bien défini car le tableau contient un mélange de types d’entiers et de chaînes.

> [!NOTE]
> Si le magasin analytique Azure Cosmos DB suit la représentation de schéma bien définie et que la spécification ci-dessus n’est pas respectée par certains éléments, ceux-ci ne sont pas inclus dans le magasin analytique.

**Représentation du schéma de fidélité optimale**

La représentation du schéma de fidélité optimale est conçue pour gérer l’intégralité des schémas polymorphes dans les données opérationnelles indépendantes du schéma. Dans cette représentation de schéma, aucun élément n’est supprimé du magasin analytique, même si les contraintes de schéma bien définies (qui ne sont pas des champs de type de données mixtes ou des tableaux de types de données mixtes) ne sont pas respectées.

Cela est possible en traduisant les propriétés de nœud terminal des données opérationnelles dans le magasin analytique avec des colonnes distinctes en fonction du type de données des valeurs de la propriété. Les noms de propriété de nœud terminal sont étendus avec des types de données, tels qu’un suffixe dans le schéma de magasin analytique, de sorte qu’ils peuvent être des requêtes sans ambiguïté.

Prenons l’exemple de document suivant dans le magasin transactionnel :

```json
{
name: "John Doe",
age: 32,
profession: "Doctor",
address: {
  streetNo: 15850,
  streetName: "NE 40th St.",
  zip: 98052
},
salary: 1000000
}
```

La propriété de nœud terminal `streetNo` dans l’objet imbriqué `address` sera représentée dans le schéma de magasin analytique en tant que colonne `address.object.streetNo.int32`. Le type de données est ajouté en tant que suffixe à la colonne. De cette façon, si un autre document est ajouté au magasin transactionnel où la valeur de la propriété de nœud terminal `streetNo` est « 123 » (notez qu’il s’agit d’une chaîne), le schéma du magasin analytique évolue automatiquement sans modification du type d’une colonne écrite précédemment. Nouvelle colonne ajoutée au magasin analytique en tant que `address.object.streetNo.string` où cete valeur « 123 » est stockée.

**Mappage de type de données au suffixe**

Voici un mappage de tous les types de données de propriété et de leurs représentations de suffixe dans le magasin analytique :

|Type de données d’origine  |Suffixe  |Exemple  |
|---------|---------|---------|
| Double |  ".float64" |    24.99|
| Array | ".array" |    ["a", "b"]|
|Binary | ".binary" |0|
|Booléen    | ".bool"   |Vrai|
|Int32  | ".int32"  |123|
|Int64  | ".int64"  |255486129307|
|Null   | ".null"   | null|
|String|    ".string" | "ABC"|
|Timestamp |    ".timestamp" |  Timestamp(0, 0)|
|DateTime   |".date"    | ISODate("2020-08-21T07:43:07.375Z")|
|ObjectId   |".objectId"    | ObjectId("5f3f7b59330ec25c132623a2")|
|Document   |".object" |    {"a": "a"}|

### <a name="cost-effective-archival-of-historical-data"></a>Archivage rentable des données historiques

La hiérarchisation des données fait référence à la séparation des données entre les infrastructures de stockage optimisées pour différents scénarios. Cela améliore les performances globales et la rentabilité de la pile de données de bout en bout. Avec le magasin analytique, Azure Cosmos DB prend désormais en charge la hiérarchisation automatique des données du magasin transactionnel vers le magasin analytique avec différentes dispositions de données. Le magasin analytique optimisé en termes de coût de stockage par rapport au magasin transactionnel vous permet de conserver des échéances plus longues pour l’analyse historique des données opérationnelles.

Une fois le magasin analytique activé, selon les besoins de conservation des données des charges de travail transactionnelles, vous pouvez configurer la propriété « Transactional Store Time to Live (Transactional TTL) » de façon à ce que les enregistrements soient automatiquement supprimés du magasin transactionnel au bout d’un certain laps de temps. De même, la propriété « Analytical Store Time To Live (Analytical TTL) » vous permet de gérer le cycle de vie des données conservées dans le magasin analytique indépendamment du magasin transactionnel. En activant le magasin analytique et en configurant les propriétés TTL, vous pouvez hiérarchiser et définir de manière transparente la période de conservation des données pour les deux magasins.

### <a name="global-distribution"></a>Diffusion mondiale

Si vous avez un compte Azure Cosmos DB distribué globalement, une fois que vous avez activé le magasin analytique pour un conteneur, il est disponible dans toutes les régions de ce compte.  Toutes les modifications apportées aux données opérationnelles sont répliquées globalement dans toutes les régions. Vous pouvez effectuer des recherches analytiques efficaces sur la copie régionale la plus proche de vos données dans Azure Cosmos DB.

### <a name="security"></a>Sécurité

L’authentification auprès du magasin analytique est identique à celle du magasin transactionnel pour une base de données particulière. Vous pouvez utiliser des clés primaires ou en lecture seule pour l’authentification. Vous pouvez tirer parti du service lié dans Synapse Studio pour empêcher le collage des clés Azure Cosmos DB dans les notebooks Spark. L’accès à ce service lié est accessible à toute personne ayant accès à l’espace de travail.

### <a name="support-for-multiple-azure-synapse-analytics-runtimes"></a>Support de plusieurs runtimes Azure Synapse Analytics

Le magasin analytique est optimisé pour fournir une extensibilité, une élasticité et des performances pour les charges de travail analytiques sans aucune dépendance des runtimes de calcul. La technologie de stockage est auto-gérée pour optimiser vos charges de travail analytiques sans effort manuel.

En découplant le système de stockage analytique du système de calcul analytique, les données du magasin analytique Azure Cosmos DB peuvent être interrogées simultanément à partir des différents runtimes analytiques pris en charge par Azure Synapse Analytics. À l’heure actuelle, Azure Synapse Analytics prend en charge Apache Spark et le pool SQL serverless avec le magasin analytique Azure Cosmos DB.

> [!NOTE]
> Vous pouvez uniquement lire à partir du magasin analytique à l’aide du runtime Azure Synapse Analytics. Vous pouvez réécrire les données dans votre magasin transactionnel en tant que couche de service.

## <a name="pricing"></a><a id="analytical-store-pricing"></a> Tarification

Le magasin analytique suit un modèle de tarification basé sur la consommation dans lequel vous êtes facturé :

* Stockage : volume des données conservées dans le magasin analytique chaque mois, y compris les données historiques telles que définies par le TTL analytique.

* Opérations d’écriture analytique : synchronisation complètement managée des mises à jour des données opérationnelles vers le magasin analytique à partir du magasin transactionnel (synchronisation automatique)

* Opérations de lecture analytique : opérations de lecture effectuées sur le magasin analytique à partir des runtimes du pool Azure Synapse Analytics Spark et du pool SQL serverless.

La tarification du magasin analytique est distincte du modèle de tarification du magasin de transactions. Il n’existe aucun concept d’unités de demande approvisionnées dans le magasin analytique. Pour plus d’informations sur le modèle de tarification du magasin analytique, consultez la [page de tarification Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/).

Afin d’obtenir une estimation précise des coûts d’activation du magasin analytique sur un conteneur Azure Cosmos DB, vous pouvez utiliser l’[outil de planification Azure Cosmos DB Capacity](https://cosmos.azure.com/capacitycalculator/) et obtenir une estimation des coûts de votre magasin analytique et des opérations d’écriture. Les coûts des opérations de lecture analytique dépendent des caractéristiques de la charge de travail analytique mais, selon une estimation précise, l’analyse de 1 To de données dans le magasin analytique entraîne généralement 130 000 opérations de lecture analytique et se traduit par un coût de 0,065 USD.

## <a name="analytical-time-to-live-ttl"></a><a id="analytical-ttl"></a> Durée de vie (TTL) analytique

La durée de vie analytique indique la durée pendant laquelle les données doivent être conservées dans votre magasin analytique, pour un conteneur. 

Si le magasin analytique est activé, les insertions, les mises à jour et les suppressions apportées aux données opérationnelles sont automatiquement synchronisées entre le magasin transactionnel et le magasin analytique, quelle que soit la configuration de la durée de vie transactionnelle. La conservation de ces données opérationnelles dans le magasin analytique peut être contrôlée par la valeur de la durée de vie analytique au niveau du conteneur, comme indiqué ci-dessous :

La durée de vie analytique sur un conteneur est définie à l’aide de la propriété `AnalyticalStoreTimeToLiveInSeconds` :

* Si la valeur est définie sur « 0 », manquante (ou définie sur null) : le magasin analytique est désactivé et aucune donnée n’est répliquée depuis le magasin transactionnel vers le magasin analytique

* S’il existe et que la valeur est définie sur « -1 » : le magasin analytique conserve toutes les données d’historique, indépendamment de la conservation des données dans le magasin transactionnel. Ce paramètre indique que le magasin analytique a une conservation infinie de vos données opérationnelles

* S’il existe et que la valeur est définie sur un nombre positif « n » : les éléments expirent du magasin analytique « n » secondes après leur dernière heure de modification dans le magasin transactionnel. Ce paramètre peut être utilisé si vous souhaitez conserver vos données opérationnelles pendant une période limitée dans le magasin analytique, indépendamment de la conservation des données dans le magasin transactionnel.

Éléments à prendre en considération :

*   Une fois que le magasin analytique est activé avec une valeur TTL analytique, il peut être mis à jour avec une valeur valide différente ultérieurement. 
*   Si la durée de vie transactionnelle peut être définie au niveau du conteneur ou de l’élément, la durée de vie analytique ne peut être définie que sur le niveau du conteneur actuellement.
*   Vous pouvez effectuer une conservation plus longue de vos données opérationnelles dans le magasin analytique en définissant la durée de vie des analyses > = transaction TTL au niveau du conteneur.
*   Le magasin analytique peut être créé pour mettre en miroir le magasin transactionnel en définissant la durée de vie transactionnelle sur la valeur de durée de vie de l’analytique TTL.

Quand vous activez le magasin analytique sur un conteneur :

* À partir du portail Azure, l’option de durée de vie analytique est définie sur la valeur par défaut -1. Vous pouvez modifier cette valeur en « n » secondes en accédant aux paramètres du conteneur sous Explorateur de données. 
 
* À partir du kit de développement logiciel (SDK) Azure, de PowerShell ou de l’interface de ligne de commande, vous pouvez activer l’option de durée de vie analytique en lui affectant la valeur -1 ou « n ». 

Pour plus d’informations, consultez [Guide pratique pour configurer la durée de vie analytique d’un conteneur](configure-synapse-link.md#create-analytical-ttl).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus, consultez les documents suivants :

* [Synapse Link pour Azure Cosmos DB](synapse-link.md)

* [Prise en main d’Azure Synapse Link pour Azure Cosmos DB](configure-synapse-link.md)

* [Forum aux questions sur Azure Synapse Link pour Azure Cosmos DB](synapse-link-frequently-asked-questions.md)

* [Cas d’utilisation d’Azure Synapse Link pour Azure Cosmos DB](synapse-link-use-cases.md)
