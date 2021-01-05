---
title: Créer un tableau de bord en temps réel à l’aide d’Azure Cosmos DB, Azure Analysis Services et Power BI
description: Découvrez comment créer un tableau de bord de météo en temps réel dans Power BI à l’aide d’Azure Cosmos DB et Azure Analysis Services.
author: SnehaGunda
ms.author: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 09/04/2019
ms.reviewer: sngun
ms.openlocfilehash: b3ec3e96aa1ba4bce3893c1af2446bb509a867b6
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93333594"
---
# <a name="create-a-real-time-dashboard-using-azure-cosmos-db-and-power-bi"></a>Créer un tableau de bord en temps réel à l’aide d’Azure Cosmos DB et Power BI
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article décrit les étapes à suivre pour créer un tableau de bord de météo en temps réel dans Power BI à l’aide d’Azure Cosmos DB et Azure Analysis Services. Le tableau de bord Power BI affiche des graphiques pour présenter des informations sur la température et les précipitations dans une région en temps réel.

## <a name="reporting-scenarios"></a>Scénarios de génération de rapports

Il existe plusieurs façons de configurer des tableaux de bord de rapport basés sur des données stockées dans Azure Cosmos DB. Le tableau suivant décrit la configuration de la génération de rapports pour chaque scénario selon les exigences d’obsolescence et la taille des données :


|Scénario |Programme d’installation |
|---------|---------|
|1. Génération de rapports ad hoc (sans actualisation)    |  [Connecteur Power BI Azure Cosmos DB avec mode d’importation](powerbi-visualize.md)       |
|2. Génération de rapports ad hoc avec actualisation périodique   |  [Connecteur Power BI Azure Cosmos DB avec mode d’importation (actualisation périodique planifiée)](powerbi-visualize.md)       |
|3. Génération de rapports sur des jeux de données volumineux (< 10 Go)     |  Connecteur Power BI Azure Cosmos DB avec actualisation incrémentielle       |
|4. Génération de rapports en temps réel sur des jeux de données volumineux    |  Connecteur Power BI Azure Analysis Services avec requête directe + Azure Analysis Services (connecteur Azure Cosmos DB)       |
|5. Génération de rapports sur des données actives avec des agrégats     |  [Connecteur Power BI Spark avec requête directe + Azure Databricks + connecteur Cosmos DB Spark.](https://github.com/Azure/azure-cosmosdb-spark/wiki/Connecting-Cosmos-DB-with-PowerBI-using-spark-and-databricks-premium)       |
|6. Génération de rapports sur des données actives avec des agrégats sur des jeux de données volumineux   |  Connecteur Power BI Azure Analysis Services avec requête directe + Azure Analysis Services + Azure Databricks + connecteur Cosmos DB Spark       |

Les scénarios 1 et 2 peuvent être facilement configurés à l’aide du connecteur Azure Cosmos DB Power BI. Cet article décrit les configurations pour les scénarios 3 et 4.

### <a name="power-bi-with-incremental-refresh"></a>Power BI avec actualisation incrémentielle

Power BI offre un mode permettant de configurer une actualisation incrémentielle. Ce mode élimine la nécessité de créer et gérer des partitions Azure Analysis Services. L’actualisation incrémentielle peut être configurée pour filtrer uniquement les dernières mises à jour dans des jeux de données volumineux. Toutefois, ce mode fonctionne uniquement avec le service Power BI Premium dont la limite de jeu de données est de 10 Go.

### <a name="power-bi-azure-analysis-connector--azure-analysis-services"></a>Connecteur Power BI Azure Analysis + Azure Analysis Services

Azure Analysis Services offre une plateforme en tant que service complètement managée qui héberge des modèles de données de qualité professionnelle dans le cloud. Les jeux de données volumineux peuvent être chargés à partir d’Azure Cosmos DB dans Azure Analysis Services. Pour éviter l’interrogation systématique de l’ensemble des jeux de données, ceux-ci peuvent être subdivisés en partitions Azure Analysis Services, qui peuvent être actualisées de manière indépendante, à des fréquences différentes.

## <a name="power-bi-incremental-refresh"></a>Actualisation incrémentielle Power BI

### <a name="ingest-weather-data-into-azure-cosmos-db"></a>Ingérer des données météo dans Azure Cosmos DB

Configurez un pipeline d’ingestion pour charger les [données météo](https://catalog.data.gov/dataset/local-weather-archive/resource/c28974a2-fc83-4722-8977-9a701323f729) dans Azure Cosmos DB. Vous pouvez configurer un travail [Azure Data Factory (ADF)](../data-factory/connector-azure-cosmos-db.md) pour charger périodiquement les dernières données météo dans Azure Cosmos DB à l’aide de la source HTTP et du récepteur Cosmos DB.


### <a name="connect-power-bi-to-azure-cosmos-db"></a>Connecter Power BI à Azure Cosmos DB

1. **Connectez le compte Azure Cosmos à Power BI**  : ouvrez Power BI Desktop et utilisez le connecteur Azure Cosmos DB pour sélectionner la base de données et le conteneur appropriés.

   :::image type="content" source="./media/create-real-time-weather-dashboard-powerbi/cosmosdb-powerbi-connector.png" alt-text="Connecteur Power BI Azure Cosmos DB":::

1. **Configurez l’actualisation incrémentielle**  : suivez les étapes de l’article [Actualisation incrémentielle dans Power BI](/power-bi/service-premium-incremental-refresh) pour configurer l’actualisation incrémentielle du jeu de données. Ajoutez les paramètres **RangeStart** et **RangeEnd** comme indiqué dans la capture d’écran suivante :

   :::image type="content" source="./media/create-real-time-weather-dashboard-powerbi/configure-range-parameters.png" alt-text="Configurer les paramètres de plage":::

   Étant donné que le jeu de données a une colonne Date au format texte, les paramètres **RangeStart** et **RangeEnd** doivent être transformés de façon à utiliser le filtre suivant. Dans le volet **Éditeur avancé** , modifiez votre requête en ajoutant le texte suivant pour filtrer les lignes en fonction des paramètres RangeStart et RangeEnd :

   ```
   #"Filtered Rows" = Table.SelectRows(#"Expanded Document", each [Document.date] > DateTime.ToText(RangeStart,"yyyy-MM-dd") and [Document.date] < DateTime.ToText(RangeEnd,"yyyy-MM-dd"))
   ```
   
   Vous pouvez modifier les champs RangeStart et RangeEnd selon la colonne et le type de données présents dans le jeu de données source.

   
   |Propriété  |Type de données  |Filtrer  |
   |---------|---------|---------|
   |_ts     |   Numérique      |  [_ts] > Duration.TotalSeconds(RangeStart - #datetime(1970, 1, 1, 0, 0, 0)) and [_ts] < Duration.TotalSeconds(RangeEnd - #datetime(1970, 1, 1, 0, 0, 0)))       |
   |Date (par exemple : - 2019-08-19)     |   String      | [Document.date]> DateTime.ToText(RangeStart,"yyyy-MM-dd") and [Document.date] < DateTime.ToText(RangeEnd,"yyyy-MM-dd")        |
   |Date (par exemple : - 2019-08-11 12:00:00)   |  String       |  [Document.date]> DateTime.ToText(RangeStart," yyyy-mm-dd HH:mm:ss") and [Document.date] < DateTime.ToText(RangeEnd,"yyyy-mm-dd HH:mm:ss")       |


1. **Définissez la stratégie d’actualisation**  : définissez la stratégie d’actualisation en accédant à l’onglet **Actualisation incrémentielle** par le biais du **menu contextuel** de la table. Définissez la stratégie d’actualisation pour actualiser les données **chaque jour** et stocker les données du mois dernier.

   :::image type="content" source="./media/create-real-time-weather-dashboard-powerbi/define-refresh-policy.png" alt-text="Définir la stratégie d’actualisation":::

   Ignorez l’avertissement indiquant qu’il est *impossible de vérifier si la requête M peut être pliée*. Le connecteur Azure Cosmos DB plie les requêtes de filtre.

1. **Chargez les données et générez les rapports**  : en utilisant les données que vous avez chargées précédemment, créez les graphiques indiquant les températures et les précipitations.

   :::image type="content" source="./media/create-real-time-weather-dashboard-powerbi/load-data-generate-report.png" alt-text="Charger les données et générer le rapport":::

1. **Publiez le rapport sur Power BI Premium**  : dans la mesure où l’actualisation incrémentielle est une fonctionnalité disponible uniquement avec le service Premium, la boîte de dialogue de publication permet uniquement de sélectionner un espace de travail de capacité Premium. L’importation des données d’historique peut être plus longue lors de la première actualisation. Les actualisations de données suivantes sont nettement plus rapides, car il s’agit d’actualisations incrémentielles.


## <a name="power-bi-azure-analysis-connector--azure-analysis-services"></a>Connecteur Power BI Azure Analysis + Azure Analysis Services 

### <a name="ingest-weather-data-into-azure-cosmos-db"></a>Ingérer des données météo dans Azure Cosmos DB 

Configurez un pipeline d’ingestion pour charger les [données météo](https://catalog.data.gov/dataset/local-weather-archive/resource/c28974a2-fc83-4722-8977-9a701323f729) dans Azure Cosmos DB. Vous pouvez configurer un travail Azure Data Factory (ADF) pour charger périodiquement les dernières données météo dans Azure Cosmos DB à l’aide de la source HTTP et du récepteur Cosmos DB.

### <a name="connect-azure-analysis-services-to-azure-cosmos-account"></a>Connecter Azure Analysis Services à un compte Azure Cosmos

1. **Créez un cluster Azure Analysis Services** - [Créez une instance Azure Analysis Services](../analysis-services/analysis-services-create-server.md) dans la même région que le compte Azure Cosmos et le cluster Databricks.

1. **Créez un projet tabulaire Analysis Services dans Visual Studio** -  [Installez SQL Server Data Tools (SSDT)](/sql/ssdt/download-sql-server-data-tools-ssdt?view=sql-server-2017&preserve-view=true) et créez un projet tabulaire Analysis Services dans Visual Studio.

   :::image type="content" source="./media/create-real-time-weather-dashboard-powerbi/create-analysis-services-project.png" alt-text="Créer un projet Azure Analysis Services":::

   Choisissez l’instance **Espace de travail intégré** et définissez le niveau de compatibilité sur **SQL Server 2017 / Azure Analysis Services (1400)** .

   :::image type="content" source="./media/create-real-time-weather-dashboard-powerbi/tabular-model-designer.png" alt-text="Concepteur de modèle tabulaire Azure Analysis Services":::

1. **Ajoutez la source de données Azure Cosmos DB**  : accédez à **Modèles**> **Sources de données** > **Nouvelle source de données** et ajoutez la source de données Azure Cosmos DB comme indiqué dans la capture d’écran suivante :

   :::image type="content" source="./media/create-real-time-weather-dashboard-powerbi/add-data-source.png" alt-text="Ajouter une source de données Cosmos DB":::

   Connectez-vous à Azure Cosmos DB en indiquant l’ **URI du compte** , le **nom de la base de données** et le **nom du conteneur**. Vous voyez à présent que les données du conteneur Azure Cosmos ont été importées dans Power BI.

   :::image type="content" source="./media/create-real-time-weather-dashboard-powerbi/preview-cosmosdb-data.png" alt-text="Prévisualiser les données Azure Cosmos DB":::

1. **Construisez le modèle Analysis Services**  : ouvrez l’éditeur de requête et effectuez les opérations nécessaires pour optimiser le jeu de données chargé :

   * Procédez à l’extraction des colonnes liées à la météo uniquement (température et précipitations).

   * Procédez à l’extraction des informations du mois à partir de la table. Ces données sont utiles pour la création de partitions comme décrit dans la section suivante.

   * Convertir les colonnes de température en nombres

   L’expression M obtenue se présente comme suit :

   ```
    let
        Source=#"DocumentDB/https://[ACCOUNTNAME].documents.azure.com:443/",
        #"Expanded Document" = Table.ExpandRecordColumn(Source, "Document", {"id", "_rid", "_self", "_etag", "fogground", "snowfall", "dust", "snowdepth", "mist", "drizzle", "hail", "fastest2minwindspeed", "thunder", "glaze", "snow", "ice", "fog", "temperaturemin", "fastest5secwindspeed", "freezingfog", "temperaturemax", "blowingsnow", "freezingrain", "rain", "highwind", "date", "precipitation", "fogheavy", "smokehaze", "avgwindspeed", "fastest2minwinddir", "fastest5secwinddir", "_attachments", "_ts"}, {"Document.id", "Document._rid", "Document._self", "Document._etag", "Document.fogground", "Document.snowfall", "Document.dust", "Document.snowdepth", "Document.mist", "Document.drizzle", "Document.hail", "Document.fastest2minwindspeed", "Document.thunder", "Document.glaze", "Document.snow", "Document.ice", "Document.fog", "Document.temperaturemin", "Document.fastest5secwindspeed", "Document.freezingfog", "Document.temperaturemax", "Document.blowingsnow", "Document.freezingrain", "Document.rain", "Document.highwind", "Document.date", "Document.precipitation", "Document.fogheavy", "Document.smokehaze", "Document.avgwindspeed", "Document.fastest2minwinddir", "Document.fastest5secwinddir", "Document._attachments", "Document._ts"}),
        #"Select Columns" = Table.SelectColumns(#"Expanded Document",{"Document.temperaturemin", "Document.temperaturemax", "Document.rain", "Document.date"}),
        #"Duplicated Column" = Table.DuplicateColumn(#"Select Columns", "Document.date", "Document.month"),
        #"Extracted First Characters" = Table.TransformColumns(#"Duplicated Column", {{"Document.month", each Text.Start(_, 7), type text}}),
        #"Sorted Rows" = Table.Sort(#"Extracted First Characters",{{"Document.date", Order.Ascending}}),
        #"Changed Type" = Table.TransformColumnTypes(#"Sorted Rows",{{"Document.temperaturemin", type number}, {"Document.temperaturemax", type number}}),
        #"Filtered Rows" = Table.SelectRows(#"Changed Type", each [Document.month] = "2019-07")
    in
        #"Filtered Rows"
   ```

   À présent, modifiez le type de données des colonnes de températures en choisissant Décimal. Vous vous assurez ainsi que ces valeurs peuvent être représentées dans Power BI.

1. **Créez des partitions Azure Analysis**  : créez des partitions dans Azure Analysis Services pour diviser le jeu de données en partitions logiques qui peuvent être actualisées de manière indépendante, à des fréquences différentes. Dans cet exemple, vous créez deux partitions qui divisent le jeu de données : l’une pour les données du mois le plus récent et l’autre pour tout le reste.

   :::image type="content" source="./media/create-real-time-weather-dashboard-powerbi/create-analysis-services-partitions.png" alt-text="Créer des partitions Analysis Services":::

   Créez les deux partitions suivantes dans Azure Analysis Services :

   * **Dernier mois** - `#"Filtered Rows" = Table.SelectRows(#"Sorted Rows", each [Document.month] = "2019-07")`
   * **Historique** -  `#"Filtered Rows" = Table.SelectRows(#"Sorted Rows", each [Document.month] <> "2019-07")`

1. **Déployez le modèle sur le serveur Azure Analysis**  : cliquez avec le bouton droit sur le projet Azure Analysis Services et choisissez **Déployer**. Ajoutez le nom du serveur dans le volet **Propriétés du serveur de déploiement**.

   :::image type="content" source="./media/create-real-time-weather-dashboard-powerbi/analysis-services-deploy-model.png" alt-text="Déployer le modèle Azure Analysis Services":::

1. **Configurez des actualisations et fusions de partitions**  : Azure Analysis Services permet de traiter les partitions de façon indépendante. Nous voulons que la partition **Dernier mois** soit constamment mise à jour avec les données les plus récentes. Vous allez donc définir l’intervalle d’actualisation sur 5 minutes. Vous pouvez actualiser les données à l’aide de [l’API REST](../analysis-services/analysis-services-async-refresh.md), [d’Azure Automation](../analysis-services/analysis-services-refresh-azure-automation.md) ou d’une [application logique](../analysis-services/analysis-services-refresh-logic-app.md). Il n’est pas nécessaire d’actualiser les données de la partition Historique. En outre, vous devez écrire du code pour consolider la partition Dernier mois sur la partition Historique et créer une nouvelle partition Dernier mois.

## <a name="connect-power-bi-to-analysis-services"></a>Connecter Power BI à Analysis Services

1. **Connectez-vous au serveur Azure Analysis à l’aide du connecteur de base de données Azure Analysis Services**  : choisissez le **mode réel** et connectez-vous à l’instance Azure Analysis Services comme indiqué dans la capture d’écran suivante :

   :::image type="content" source="./media/create-real-time-weather-dashboard-powerbi/analysis-services-get-data.png" alt-text="Obtenir les données d’Azure Analysis Services":::

1. **Chargez les données et générez les rapports**  : en utilisant les données que vous avez chargées précédemment, créez les graphiques indiquant les températures et les précipitations. Étant donné que vous créez une connexion active, les requêtes doivent être exécutées sur les données du modèle Azure Analysis Services que vous avez déployé à l’étape précédente. Les graphiques de température seront mis à jour dans les cinq minutes suivant le chargement des nouvelles données dans Azure Cosmos DB.

   :::image type="content" source="./media/create-real-time-weather-dashboard-powerbi/load-data-generate-report.png" alt-text="Charger les données et générer les rapports":::

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Power BI, consultez la section [Get started with Power BI (Prise en main de Power BI)](https://powerbi.microsoft.com/documentation/powerbi-service-get-started/).

* [Connecter Qlik Sense à Azure Cosmos DB et visualiser vos données](visualize-qlik-sense.md)
