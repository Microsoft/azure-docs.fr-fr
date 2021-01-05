---
title: Avantages et modalités d’utilisation d’Azure Synapse Link pour Azure Cosmos DB
description: En savoir plus sur Azure Synapse Link pour Azure Cosmos DB. Synapse Link vous permet d’exécuter l’analytique en temps quasi-réel (HTAP) à l’aide d’Azure Synapse Analytics sur les données opérationnelles d’Azure Cosmos DB.
author: Rodrigossz
ms.author: rosouz
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 11/30/2020
ms.reviewer: sngun
ms.openlocfilehash: d083bc0b7726a284dcfd03e49d47c2a342db023c
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461802"
---
# <a name="what-is-azure-synapse-link-for-azure-cosmos-db"></a>Qu’est-ce qu’Azure Synapse Link pour Azure Cosmos DB ?
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

> [!IMPORTANT]
> La prise en charge du pool SQL serverless Synapse pour Azure Synapse Link pour Azure Cosmos DB est actuellement en préversion. Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Synapse Link pour Azure Cosmos DB est une fonctionnalité de traitement transactionnel et analytique (HTAP) hybride cloud native qui vous permet d’exécuter des analyses en temps quasi-réel sur les données opérationnelles dans Azure Cosmos DB. Azure Synapse Link crée une intégration transparente entre Azure Cosmos DB et Azure Synapse Analytics.

À l’aide du [magasin analytique Azure Cosmos DB](analytical-store-introduction.md), un magasin de colonnes totalement isolé, Azure Synapse Link n’autorise aucune analyse ETL (Extract-Transform-Load) dans [Azure Synapse Analytics](../synapse-analytics/overview-what-is.md) sur vos données opérationnelles à grande échelle. Les analystes d’entreprise, les ingénieurs de données et les chercheurs de données peuvent désormais utiliser Synapse Spark ou Synapse SQL de façon interchangeable pour exécuter des pipelines décisionnels, analytiques et Machine Learning en temps quasi réel. Vous pouvez y parvenir sans affecter les performances de vos charges de travail transactionnelles sur Azure Cosmos DB. 

L’illustration suivante représente l’intégration d’Azure Synapse Link dans Azure Cosmos DB et Azure Synapse Analytics : 

:::image type="content" source="./media/synapse-link/synapse-analytics-cosmos-db-architecture.png" alt-text="Diagramme d’architecture pour l’intégration d’Azure Synapse Analytics dans Azure Cosmos DB" border="false":::

## <a name="benefits"></a><a id="synapse-link-benefits"></a> Avantages

Pour analyser les jeux de données opérationnels volumineux tout en minimisant l’impact sur les performances des charges de travail transactionnelles stratégiques, les données opérationnelles dans Azure Cosmos DB sont habituellement extraites et traitées par les pipelines ETL (Extract-Transform-Load). Les pipelines ETL nécessitent de nombreuses couches de déplacement de données, ce qui réduit considérablement la complexité opérationnelle et l’impact sur les performances de vos charges de travail transactionnelles. Cela augmente également la latence d’analyse des données opérationnelles à partir de l’origine.

Par rapport aux solutions ETL traditionnelles, Azure Synapse Link pour Azure Cosmos DB offre plusieurs avantages tels que :  

### <a name="reduced-complexity-with-no-etl-jobs-to-manage"></a>Complexité réduite sans tâches ETL à gérer

Azure Synapse Link vous permet d’accéder directement au magasin analytique Azure Cosmos DB à l’aide d’Azure Synapse Analytics sans déplacement de données complexe. Toutes les mises à jour apportées aux données opérationnelles sont visibles dans le magasin analytique quasiment en temps réel, sans ETL ni travaux de flux de modification. Vous pouvez exécuter des traitements analytiques à grande échelle sur le magasin analytique à partir d’Azure Synapse Analytics sans transformation de données supplémentaire.

### <a name="near-real-time-insights-into-your-operational-data"></a>Insights en temps quasi réel dans vos données opérationnelles

Vous pouvez désormais bénéficier d’insights détaillés sur vos données opérationnelles quasiment en temps réel à l’aide d’Azure Synapse Link. Les systèmes basés sur ETL ont tendance à avoir une latence plus élevée pour l’analyse de vos données opérationnelles, en raison des nombreuses couches nécessaires pour l’extraction, la transformation et le chargement des données opérationnelles. Avec l’intégration native du magasin analytique Azure Cosmos DB à l’aide d’Azure Synapse Analytics, vous pouvez analyser les données opérationnelles en temps quasi-réel en activant de nouveaux scénarios d’entreprise. 

### <a name="no-impact-on-operational-workloads"></a>Aucun impact sur les charges de travail opérationnelles

Avec Azure Synapse Link, vous pouvez exécuter des requêtes analytiques sur un magasin analytique Azure Cosmos DB (un magasin de colonnes distinct) tandis que les opérations transactionnelles sont traitées à l’aide du débit approvisionné pour la charge de travail transactionnelle (magasin transactionnel basé sur des lignes).  La charge de travail analytique est traitée indépendamment du trafic de la charge de travail transactionnelle sans consommer l’un des débits approvisionnés pour vos données opérationnelles.

### <a name="optimized-for-large-scale-analytics-workloads"></a>Optimisé pour les charges de travail analytiques à grande échelle

Le magasin analytique Azure Cosmos DB est optimisé pour fournir une extensibilité, une élasticité et des performances pour les charges de travail analytiques sans aucune dépendance des runtimes de calcul. La technologie de stockage est auto-gérée pour optimiser vos charges de travail analytiques. Avec la prise en charge intégrée dans Azure Synapse Analytics, l’accès à cette couche de stockage offre une simplicité d’utilisation et des performances élevées.

### <a name="cost-effective"></a>Rentabilité

Avec Azure Synapse Link, vous pouvez obtenir une solution complètement managée et optimisée pour l’analytique opérationnelle. Cela élimine les couches de stockage et de calcul supplémentaires requises dans les pipelines ETL traditionnels pour l’analyse des données opérationnelles. 

Le magasin analytique Azure Cosmos DB s’appuie sur un modèle de tarification basé sur la consommation, qui est basé sur le stockage de données et les opérations et requêtes de lecture/écriture analytiques exécutées. Vous n’avez pas besoin d’approvisionner un débit comme vous le faites aujourd’hui pour les charges de travail transactionnelles. L’accès à vos données avec des moteurs de calcul hautement élastiques à partir d’Azure Synapse Analytics optimisent l’efficacité du stockage et du calcul.


### <a name="analytics-for-locally-available-globally-distributed-multi-region-writes"></a>Analytique pour les écritures multirégions disponibles localement et distribuées globalement

Vous pouvez effectuer des recherches analytiques efficaces sur la copie régionale la plus proche des données dans Azure Cosmos DB. Azure Cosmos DB offre l’avantage d’exécuter les charges de travail analytiques distribuées dans le monde entier, ainsi que des charges de travail transactionnelles en mode actif/actif.

## <a name="enable-htap-scenarios-for-your-operational-data"></a>Activer les scénarios HTAP pour vos données opérationnelles

Synapse Link regroupe le magasin analytique Azure Cosmos DB et la prise en charge du runtime Azure Synapse Analytics. Cette intégration vous permet de créer des solutions HTAP (Hybrid transactional/analytical processing) qui génèrent des insights basés sur des mises à jour en temps réel de vos données opérationnelles sur des jeux de données volumineux. Elle permet de mettre en place de nouveaux scénarios d’entreprise pour déclencher des alertes basées sur des tendances en direct, créer des tableaux de bord en temps quasi réel et des expériences métier basées sur le comportement des utilisateurs.

### <a name="azure-cosmos-db-analytical-store"></a>Magasin analytique Azure Cosmos DB

Le magasin analytique Azure Cosmos DB est une représentation orientée colonne de vos données opérationnelles dans Azure Cosmos DB. Ce magasin analytique convient aux requêtes rapides et rentables sur de grands jeux de données opérationnelles, sans copier les données ni impacter les performances de vos charges de travail transactionnelles.

Le magasin analytique récupère automatiquement les insertions, les mises à jour et les suppressions à haute fréquence dans vos charges de travail transactionnelles en temps quasi-réel, en tant que fonctionnalité complètement managée (la fameuse « synchronisation automatique ») d’Azure Cosmos DB. Aucun flux de modification ou ETL n’est requis. 

Si vous avez un compte Azure Cosmos DB distribué globalement, une fois que vous avez activé le magasin analytique pour un conteneur, il est disponible dans toutes les régions de ce compte. Pour plus d’informations sur le magasin analytique, consultez l’article [Vue d’ensemble du magasin analytique Azure Cosmos DB](analytical-store-introduction.md).

### <a name="integration-with-azure-synapse-analytics"></a><a id="synapse-link-integration"></a>Intégration dans Azure Synapse Analytics

Synapse Link vous permet désormais de vous connecter directement à vos conteneurs Azure Cosmos DB à partir d’Azure Synapse Analytics et d’accéder au magasin analytique sans connecteurs distincts. Azure Synapse Analytics prend actuellement en charge Synapse Link avec [Synapse Apache Spark](../synapse-analytics/spark/apache-spark-concepts.md) et un [pool SQL serverless](../synapse-analytics/sql/on-demand-workspace-overview.md).

Vous pouvez interroger les données à partir du magasin analytique Azure Cosmos DB simultanément en bénéficiant de l’interopérabilité entre les différentes durées d’exécution des traitements analytiques prises en charge par Azure Synapse Analytics. Aucune transformation de données supplémentaire n’est requise pour analyser les données opérationnelles. Vous pouvez interroger et analyser les données du magasin analytique à l’aide des éléments suivants :

* Synapse Apache Spark avec prise en charge complète de Scala, Python, SparkSQL et C#. Synapse Spark est essentiel à l’engineering données et aux scénarios de science des données

* Pool SQL serverless avec langage T-SQL et la prise en charge des outils décisionnels courants (par exemple, Power BI Premium, etc.)

> [!NOTE]
> À partir d’Azure Synapse Analytics, vous pouvez accéder aux magasins analytiques et transactionnels dans votre conteneur Azure Cosmos DB. Toutefois, si vous souhaitez exécuter des traitements analytiques ou des analyses à grande échelle sur vos données opérationnelles, nous vous recommandons d’utiliser le magasin analytique pour éviter de subir une dégradation des performances des charges de travail transactionnelles.

> [!NOTE]
> Vous pouvez exécuter des traitements analytiques avec une faible latence dans une région Azure en connectant votre conteneur Azure Cosmos DB au runtime Synapse dans cette région.

Cette intégration active les scénarios HTAP suivants pour des utilisateurs différents :

* Ingénieur décisionnel qui souhaite modéliser et publier un rapport Power BI pour accéder aux données opérationnelles d’Azure Cosmos DB directement par le biais de Synapse SQL.

* Analyste de données qui souhaite obtenir des insights à partir des données opérationnelles dans un conteneur Azure Cosmos DB en l’interrogeant avec Synapse SQL, lire des données à grande échelle et combiner ces informations avec d’autres sources de données.

* Scientifique des données qui souhaite utiliser Synapse Spark pour trouver une fonctionnalité permettant d’améliorer son modèle et d’effectuer l’apprentissage de ce modèle sans faire appel à l’engineering données complexe. Il peut également écrire les résultats de l’inférence de la publication du modèle dans Azure Cosmos DB pour un scoring en temps réel sur les données par le biais de Spark Synapse.

* Ingénieur de données qui souhaite rendre les données accessibles aux consommateurs en créant des tables SQL ou Spark sur des conteneurs Azure Cosmos DB sans processus ETL manuels.

Pour plus d’informations sur la prise en charge du runtime Azure Synapse Analytics pour Azure Cosmos DB, consultez [Prise en charge d’Azure Synapse Analytics pour Cosmos DB](../synapse-analytics/synapse-link/concept-synapse-link-cosmos-db-support.md).

## <a name="when-to-use-azure-synapse-link-for-azure-cosmos-db"></a>Quand utiliser Azure Synapse Link pour Azure Cosmos DB ?

Synapse Link est recommandé dans les cas suivants :

* Si vous êtes un client Azure Cosmos DB et que vous souhaitez effectuer de l’analytique, du décisionnel et du Machine Learning sur vos données opérationnelles. Dans ce cas, Synapse Link offre une expérience analytique plus intégrée sans affecter le débit approvisionné de votre magasin transactionnel. Par exemple :

  * Si vous exécutez des traitements analytiques ou décisionnels sur vos données opérationnelles Azure Cosmos DB directement à l’aide de connecteurs séparés, ou

  * Si vous exécutez des processus ETL pour extraire des données opérationnelles dans un système analytique distinct.
 
Dans ce cas, Synapse Link offre une expérience analytique plus intégrée sans affecter le débit approvisionné de votre magasin transactionnel.

L’utilisation de Synapse Link est déconseillée si vos exigences en matière d’entrepôt de données sont, par exemple, la concurrence élevée, la gestion des charges de travail et la persistance des agrégats sur plusieurs sources de données. Pour plus d’informations, consultez [Scénarios courants pouvant être alimentés Azure Synapse Link pour Azure Cosmos DB](synapse-link-use-cases.md).

## <a name="limitations"></a>Limites

* Aujourd’hui, Azure Synapse Link pour Azure Cosmos DB est pris en charge pour l’API SQL et l’API Azure Cosmos DB pour MongoDB. Il n’est pas pris en charge pour l’API Table et l’API Gremlin. La prise en charge de l’API Cassandra est en préversion privée. Pour plus d’informations, contactez [l’équipe Azure Synapse Link](mailto:cosmosdbsynapselink@microsoft.com).  

* Actuellement, le magasin analytique ne peut être activé que pour les nouveaux conteneurs. Pour utiliser le magasin analytique pour les conteneurs existants, migrez les conteneurs existants vers de nouveaux conteneurs à l’aide des [outils de migration Azure Cosmos DB](cosmosdb-migrationchoices.md). Vous pouvez activer Synapse Link sur les comptes Azure Cosmos DB, nouveaux comme existants.

* Pour les conteneurs avec le magasin analytique activé, la sauvegarde et la restauration automatiques de vos données dans le magasin analytique ne sont pas prises en charge pour l’instant. Lorsque Synapse Link est activé sur un compte de base de données, Azure Cosmos DB continue automatiquement [d’effectuer des sauvegardes](./online-backup-and-restore.md) de vos données dans le magasin transactionnel (uniquement) des conteneurs selon l’intervalle de sauvegarde planifié, comme toujours. Il est important de noter que lorsqu’un conteneur avec le magasin analytique activé est restauré vers un nouveau compte, le conteneur est restauré avec uniquement le magasin transactionnel et aucun magasin analytique activé. 

* L’accès au magasin Azure Cosmos DB Analytics avec Synapse SQL approvisionné n’est pas disponible actuellement.

## <a name="pricing"></a>Tarifs

Le modèle de facturation d’Azure Synapse Link comprend les coûts engendrés par l’utilisation du magasin analytique Azure Cosmos DB et du runtime Synapse. Pour plus d’informations, consultez la [tarification du magasin analytique Azure Cosmos DB](analytical-store-introduction.md#analytical-store-pricing) et les articles sur la [tarification d’Azure Synapse Analytics](https://azure.microsoft.com/pricing/details/synapse-analytics/).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus, consultez les documents suivants :

* [Vue d’ensemble du magasin analytique Azure Cosmos DB](analytical-store-introduction.md)

* [Prise en main d’Azure Synapse Link pour Azure Cosmos DB](configure-synapse-link.md)
 
* [Éléments pris en charge durant le temps d’exécution d’Azure Synapse Analytics](../synapse-analytics/synapse-link/concept-synapse-link-cosmos-db-support.md)

* [Forum aux questions sur Azure Synapse Link pour Azure Cosmos DB](synapse-link-frequently-asked-questions.md)

* [Cas d’utilisation d’Azure Synapse Link pour Azure Cosmos DB](synapse-link-use-cases.md)