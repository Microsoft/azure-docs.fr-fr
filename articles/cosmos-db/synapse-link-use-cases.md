---
title: Cas d’utilisation de l’analyse en temps réel avec Azure Synapse Link pour Azure Cosmos DB
description: Découvrez comment Azure Synapse Link pour Azure Cosmos DB est utilisé dans l’analyse de chaîne logistique, la prévision, la création de rapports, la personnalisation en temps réel et la maintenance prédictive par IoT.
author: tknandu
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/19/2020
ms.author: ramkris
ms.openlocfilehash: 9855a53ebdf7501907b3e1e63d59823d3a0b0209
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96463117"
---
# <a name="azure-synapse-link-for-azure-cosmos-db-near-real-time-analytics-use-cases"></a>Azure Synapse Link pour Azure Cosmos DB : Scénarios d‘utilisation courants d’analyse en quasi temps réel
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

[Azure Synapse Link](synapse-link.md) pour Azure Cosmos DB est une fonctionnalité de traitement transactionnel et analytique (HTAP) hybride cloud native qui vous permet d’exécuter des analyses en temps quasi-réel sur les données opérationnelles. Synapse Link crée une intégration transparente entre Azure Cosmos DB et Azure Synapse Analytics.

Vous êtes peut-être curieux de comprendre quels sont les cas d’utilisation qui peuvent tirer parti de cette fonctionnalité HTAP cloud pour l’analyse en quasi temps réel des données opérationnelles. Voici trois cas d’utilisation courants pour Azure Synapse Link pour Azure Cosmos DB :

* Analyse de chaîne logistique, prévisions et rapports
* Personnalisation en temps réel
* Maintenance prédictive, détection d’anomalies dans les scénarios IoT

> [!NOTE]
> Azure Synapse Link pour Azure Cosmos DB cible le scénario dans lequel les équipes en entreprise cherchent à exécuter des analyses quasiment en temps réel. Ces analyses sont exécutées sans ETL sur des données opérationnelles générées sur les applications transactionnelles basées sur Azure Cosmos DB. Cela ne remplace l’entrepôt de données séparé lorsqu’il existe des besoins pour un entrepôt de données traditionnel, par exemple pour la gestion de la charge de travail, la concurrence élevée ou les agrégats de persistance sur plusieurs sources de données.

## <a name="supply-chain-analytics-forecasting--reporting"></a>Analyse de chaîne logistique, prévisions et rapports

Les études montrent que l’incorporation d’analyse de Big Data dans les opérations logistiques débouchent sur une amélioration des délais de livraison et de l’efficacité de la chaîne logistique.

Les fabricants s’intègrent des technologies cloud natives pour sortir des contraintes des anciens systèmes ERP et de gestion de la chaîne logistique. Alors que les chaînes logistiques génèrent de plus en plus de volumes de données opérationnelles (commande, expédition, données de transaction), les fabricants ont besoin d’une base de données opérationnelle. Cette base de données opérationnelle doit être mise à l’échelle pour gérer les volumes de données, ainsi qu’une plateforme analytique pour atteindre un niveau d’intelligence contextuelle en temps réel afin de garder un temps d’avance.

L’architecture suivante illustre la puissance d’Azure Cosmos DB comme base de données opérationnelle cloud native et Synapse Link dans l’analytique de chaîne logistique :

:::image type="content" source="./media/synapse-link-use-cases/supply-chain-analytics.png" alt-text="Azure Synapse Link pour Azure Cosmos DB dans l’analytique de chaîne logistique " border="false":::

Sur la base de l’architecture précédente, vous pouvez obtenir les cas d’utilisation suivants avec Synapse Link pour Azure Cosmos DB :

* **Préparer et effectuer l’apprentissage d’un pipeline prédictif :** Générez des insights sur les données opérationnelles dans la chaîne logistique à l’aide de traductions Machine Learning. De cette façon, vous pouvez réduire les frais d’inventaire et d’exploitation et réduire les délais de livraison de la commande au client.

  Synapse Link vous permet d’analyser des données opérationnelles fluctuantes dans Azure Cosmos DB sans aucun processus ETL manuel. Cela vous fait économiser des coûts, et gagner en latence et en complexité opérationnelle. Synapse Link permet aux ingénieurs des données et aux scientifiques des données de créer des pipelines prédictifs solides :

  * Interroger les données opérationnelles du magasin analytique Azure Cosmos DB en tirant parti de l’intégration native avec les pools Apache Spark dans Azure Synapse Analytics. Vous pouvez interroger les données d’un bloc-notes interactif ou des tâches à distance planifiées sans ingénierie de données complexe.

  * Créez des modèles Machine Learning (ML) avec les algorithmes Spark ML et l’intégration d’Azure ML dans Azure Synapse Analytics.

  * Réécrivez les résultats après l’inférence de modèle dans Azure Cosmos DB pour un scoring en quasi temps réel.

* **Rapports opérationnels :** Les équipes responsables des chaînes logistiques ont besoin de rapports flexibles et personnalisés sur des données opérationnelles précises et en temps réel. Ces rapports sont nécessaires pour bénéficier d’une vue instantanée de l’efficacité, de la rentabilité et de la productivité de la chaîne logistique. Elle permet aux analystes de données et aux autres parties prenantes de réévaluer constamment l’activité et d’identifier les points à ajuster pour réduire les frais d’exploitation. 

  Synapse Link pour Azure Cosmos DB permet de créer des scénarios riches d’analyse décisionnelle ou de rapports :

  * Interroger des données opérationnelles à partir du magasin analytique Azure Cosmos DB à l’aide de l’intégration native avec un pool SQL serverless et l’expressivité complète du langage T-SQL.

  * Modéliser et publier automatiquement les tableaux de bord BI sur Azure Cosmos DB par le biais de la prise en charge de pool SQL serverless pour des outils BI classiques. Par exemple, Azure Analysis Services, Power BI Premium, etc.

Voici quelques conseils pour l’intégration de données pour les données par lot et les données en flux dans Azure Cosmos DB :

* **Intégration de données par lot et orchestration :** Avec la complexité grandissante des chaînes logistiques, les plateformes de données de chaîne logistique doivent intégrer de nombreux formats et sources de données. Azure Synapse comprend le même moteur d’intégration de données et les mêmes expériences qu’Azure Data Factory. Cette intégration permet aux ingénieurs des données de créer des pipelines de données enrichis sans autre moteur d’orchestration :

  * Déplacez les données de plus de 85 sources de données prises en charge vers [Azure Cosmos DB avec Azure Data Factory](../data-factory/connector-azure-cosmos-db.md).

  * Écrivez des pipelines ETL sans code pour Azure Cosmos DB, y compris des [mappages relationnels de hiérarchies et des mappages entre hiérarchies avec des flux de données de mappage](../data-factory/how-to-sqldb-to-cosmosdb.md).

* **Intégration et traitement des données de flux :** Avec la croissance de l’IoT industriel (capteurs de suivi des ressources « de la production à la mise en vente », flottes logistiques connectées, etc.), on constate une explosion des données en temps réel générées en continu et qui doivent être intégrées aux données classiques plus lentes afin de générer des insights. Azure Stream Analytics est un service recommandé pour les flux ETL et le traitement sur Azure dans un [large éventail de scénarios](../stream-analytics/streaming-technologies.md). Azure Stream Analytics prend en charge [Azure Cosmos DB en tant que récepteur de données natif](../stream-analytics/stream-analytics-documentdb-output.md).

## <a name="real-time-personalization"></a>Personnalisation en temps réel

Les commerçants doivent désormais créer des solutions de commerce électronique sécurisées et évolutives qui répondent aux besoins des clients et des entreprises. Ces solutions de commerce électronique doivent attirer les clients avec des produits et des offres personnalisés, traiter les transactions de façon rapide et sécurisée, et se concentrer sur le traitement et le service clientèle. Azure Cosmos DB avec la dernière version de Synapse Link pour Azure Cosmos DB permet aux détaillants de générer des recommandations personnalisées pour les clients en temps réel. Ils utilisent des paramètres de cohérence ajustables à faible latence pour obtenir des insights immédiats, comme présenté dans l’architecture suivante :

:::image type="content" source="./media/synapse-link-use-cases/real-time-personalization.png" alt-text="Azure Synapse Link pour Azure Cosmos DB dans la personnalisation en temps réel" border="false":::

Cas d’utilisation de Synapse Link pour Azure Cosmos DB :

* **Préparer et effectuer l’apprentissage d’un pipeline prédictif :** Vous pouvez générer des insights sur les données opérationnelles de vos unités commerciales ou segments de clientèle à l’aide de Synapse Spark et des modèles Machine Learning. Cela se traduit par une livraison personnalisée pour cibler des segments de clientèle, des expériences prédictives pour utilisateur final et un marketing ciblé pour répondre aux besoins de vos utilisateurs finaux.

## <a name="iot-predictive-maintenance"></a>Maintenance prédictive IoT

Les innovations de l’IoT industriel ont considérablement réduit les temps d’arrêt des machines et accru l’efficacité globale dans tous les domaines de l’industrie. L’une de ces innovations est l’analytique de maintenance prédictive pour les machines périphérie du cloud.

Vous trouverez ci-dessous une architecture tirant parti des fonctionnalités HTAP natives dans le cloud d’Azure Synapse Link pour Azure Cosmos DB dans la maintenance prédictive IoT :

:::image type="content" source="./media/synapse-link-use-cases/iot-predictive-maintenance.png" alt-text="Azure Synapse Link pour Azure Cosmos DB dans la maintenance prédictive IoT" border="false" :::

Cas d’utilisation de Synapse Link pour Azure Cosmos DB :

* **Préparer et effectuer l’apprentissage d’un pipeline prédictif :** Les données opérationnelles historiques des capteurs IoT peuvent être utilisées pour effectuer l’apprentissage de modèles prédictifs, comme des détecteurs d’anomalies. Ces détecteurs d’anomalies sont ensuite déployés en périphérie pour une surveillance en temps réel. Cette boucle vertueuse permet d’effectuer un réapprentissage continu des modèles prédictifs.

* **Rapports opérationnels :** Avec la croissance des initiatives numériques, les entreprises collectent de vastes quantités de données opérationnelles par l’intermédiaire de nombreux capteurs pour créer une copie numérique de chaque ordinateur. Ces données servent au décisionnel pour pouvoir comprendre les tendances des données historiques en plus des applications en temps réel par rapport aux données chaudes récentes.

## <a name="sample-scenario-htap-for-azure-cosmos-db"></a>Exemple de scénario : HTAP pour Azure Cosmos DB

Depuis près d’une décennie, Azure Cosmos DB est utilisé par des milliers de clients dans le cadre d’applications stratégiques qui nécessitent une mise à l’échelle élastique, une distribution mondiale clé en main, une réplication d’écritures dans plusieurs régions pour une faible latence et une haute disponibilité, à la fois en lecture et en écriture dans les charges de travail transactionnelles.
 
La liste suivante présente une vue d’ensemble des différents types de charge de travail pris en charge avec données opérationnelles à l’aide d’Azure Cosmos DB :

* Applications et services en temps réel
* Traitement de flux d’événements
* Tableaux de bord BI
* Analytique du Big Data
* Apprentissage automatique

Azure Synapse Link permet à Azure Cosmos DB de ne pas simplement permettre l’exécution de charges de travail transactionnelles, mais également de traiter des charges de travail analytiques quasiment en temps réel sur les données opérationnelles historiques. Cela se produit sans besoins ETL et avec une isolation garantie des performances des charges de travail transactionnelles.

L’illustration suivante présente des modèles de charge de travail utilisant Azure Cosmos DB : :::image type="content" source="./media/synapse-link-use-cases/synapse-link-workload-patterns.png" alt-text="Modèles de charges de travail Azure Synapse Link pour Azure Cosmos DB" border="false":::

Prenons l’exemple d’une entreprise de commerce électronique CompanyXYZ présente dans plus de 20 pays/régions pour illustrer les avantages d’Azure Cosmos DB comme base de données en temps réel visant à répondre aux exigences transactionnelles et analytiques d’une plateforme de gestion des stocks.

* L’activité principale de CompanyXYZ dépend du système de gestion des stocks. La disponibilité et la fiabilité de ce systèmes sont de ce fait des piliers fondamentaux. Avantages de l’utilisation d’Azure Cosmos DB :

  * Avec l’intégration en profondeur de l’infrastructure Azure et de la réplication d’écritures dans plusieurs régions, Azure Cosmos DB offre une [haute disponibilité de 99,999 %](high-availability.md) face aux pannes régionales pour les lectures et les écritures.

* Les partenaires de la chaîne logistique de CompanyXYZ peuvent se trouver dans des régions différentes, mais ils peuvent accéder à une vue unique des stocks de produits dans le monde entier pour pouvoir assurer les activités au niveau local. Cela comprend la nécessité de pouvoir lire en temps réel les mises à jour effectuées par d’autres partenaires de la chaîne logistique. Vous pouvez également effectuer des mises à jour sans vous soucier des conflits avec les autres partenaires, à débit élevé. Avantages de l’utilisation d’Azure Cosmos DB :

  * Avec son protocole unique de réplication d’écritures dans plusieurs régions sans verrou, et son magasin transactionnel optimisé pour l’écriture, Azure Cosmos DB garantit des latences inférieures à 10 millisecondes pour les lectures et les écritures indexées, au 99e centile, dans le monde entier.

  * Ingestion à débit des flux de données par lot et en flux avec [indexation en temps réel](index-policy.md) dans le magasin transactionnel.

  * Le magasin transactionnel Azure Cosmos DB offre trois options supplémentaires par rapport aux deux extrêmes de niveaux de cohérence forte et finale, pour arriver à un compromis [performances et disponibilité](./consistency-levels.md) le plus proche possible des besoins de l’entreprise.

* Les partenaires de la chaîne logistique de CompanyXYZ ont des modèles de trafic très fluctuants, qui vont de quelques centaines à plusieurs millions de demandes par seconde, et la plateforme de gestion des stocks doit donc pouvoir faire face à ces pics de trafic.  Avantages de l’utilisation d’Azure Cosmos DB :

  * Le magasin transactionnel d’Azure Cosmos DB prend en charge la scalabilité élastique du stockage et du débit au moyen d’un partitionnement horizontal. Les conteneurs et les bases de données configurés en mode Autopilot peuvent effectuer automatiquement et immédiatement la mise à l’échelle du débit approvisionné en fonction des besoins de l’application, sans incidence sur la disponibilité, la latence, le débit ou les performances de la charge de travail de manière globale.

* CompanyXYZ doit mettre en place une plateforme d’analyse sécurisée pour héberger des données d’inventaire historiques à l’échelle du système, afin de pouvoir appliquer l’analyse et les insights à tous les partenaires de la chaîne logistique, toutes les divisions et toutes les fonctions. Cette plateforme d’analyse doit permettre la collaboration dans l’ensemble du système, avec les cas d’utilisation BI/rapports classiques, les cas d’utilisation d’analytique avancée et les solutions intelligentes de prédiction sur les données d’inventaire. Avantages de Synapse Link pour Azure Cosmos DB :

  * Avec le [magasin analytique Azure Cosmos DB](analytical-store-introduction.md), un magasin de colonnes totalement isolé, Azure Synapse Link n’autorise aucune analyse ETL (extraction-transformation-chargement) dans [Azure Synapse Analytics](../synapse-analytics/overview-what-is.md) sur les données opérationnelles distribuées.  Les analystes d’entreprise, les ingénieurs de données et les experts en sciences des données peuvent désormais utiliser Synapse Spark ou Synapse SQL de manière interopérable pour exécuter des opérations décisionnelles en temps quasi-réel, des analyses et des pipelines Machine Learning, sans affecter les performances de leurs charges de travail transactionnelles sur Azure Cosmos DB. Pour plus d’informations, découvrez les [avantages de Synapse Link dans Azure Cosmos DB](synapse-link.md).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus, consultez les documents suivants :

* [Azure Synapse Link pour Azure Cosmos DB](synapse-link.md) 

* [Magasin analytique Azure Cosmos DB](analytical-store-introduction.md)

* [Utilisation d’Azure Synapse Link pour Azure Cosmos DB](configure-synapse-link.md)

* [Foire aux questions sur Azure Synapse Link pour Azure Cosmos DB](synapse-link-frequently-asked-questions.md)

* [Apache Spark dans Azure Synapse Analytics](../synapse-analytics/spark/apache-spark-concepts.md)

* [Prise en charge du runtime de pool SQL serverless dans Azure Synapse Analytics](../synapse-analytics/sql/on-demand-workspace-overview.md)