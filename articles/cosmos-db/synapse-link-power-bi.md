---
title: Power BI et un pool SQL serverless pour analyser les données Azure Cosmos DB avec Synapse Link
description: Apprenez à créer une base de données de pool SQL serverless et des affichages sur Synapse Link pour Azure Cosmos DB, à interroger les conteneurs Azure Cosmos DB et à créer un modèle avec Power BI sur ces affichages.
author: ArnoMicrosoft
ms.service: cosmos-db
ms.topic: how-to
ms.date: 11/30/2020
ms.author: acomet
ms.openlocfilehash: 959070ca431c3397779a2a22c16f03b3adebbb35
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96444501"
---
# <a name="use-power-bi-and-serverless-synapse-sql-pool-preview-to-analyze-azure-cosmos-db-data-with-synapse-link"></a>Utiliser Power BI et le pool Synapse SQL serverless (préversion) pour analyser les données Azure Cosmos DB avec Synapse Link 
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

Dans cet article, vous allez apprendre à créer une base de données de pools SQL serverless et des affichages sur Synapse Link pour Azure Cosmos DB. Vous interrogerez les conteneurs Azure Cosmos DB, puis créerez un modèle avec Power BI sur ces affichages pour refléter cette requête.

Dans ce scénario, vous allez utiliser des données factices sur les ventes de produits superficiels dans un magasin de vente au détail des partenaires. Vous allez analyser le revenu par magasin en fonction de la proximité des ménages importants et de l’impact de la publicité pour une semaine spécifique. Dans cet article, vous allez créer deux affichages, nommés **RetailSales** et **StoreDemographics**, et une requête entre eux. Vous pouvez récupérer les exemples de données de produit à partir de ce référentiel [GitHub](https://github.com/Azure-Samples/Synapse/tree/master/Notebooks/PySpark/Synapse%20Link%20for%20Cosmos%20DB%20samples/Retail/RetailData).

> [!IMPORTANT]
> La prise en charge du pool SQL serverless Synapse pour Azure Synapse Link pour Azure Cosmos DB est actuellement en préversion. Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Avant de commencer, veillez à créer les ressources suivantes :

* [Créez un compte Azure Cosmos DB de type SQL (Core) ou MongoDB.](create-cosmosdb-resources-portal.md)

* Activer Azure Synapse Link pour votre compte [Azure Cosmos](configure-synapse-link.md#enable-synapse-link)

* Créez une base de données dans le compte Azure Cosmos et deux conteneurs pour lesquels le [magasin analytique est activé.](configure-synapse-link.md#create-analytical-ttl)

* Chargez des données de produits dans les conteneurs Azure Cosmos, comme décrit dans ce notebook [ d’ingestion des données par lots](https://github.com/Azure-Samples/Synapse/blob/master/Notebooks/PySpark/Synapse%20Link%20for%20Cosmos%20DB%20samples/Retail/spark-notebooks/pyspark/1CosmoDBSynapseSparkBatchIngestion.ipynb).

* [Créez un espace de travail Synapse](../synapse-analytics/quickstart-create-workspace.md) nommé **SynapseLinkBI**.

* [Connectez la base de données Azure Cosmos DB à l’espace de travail Synapse](../synapse-analytics/synapse-link/how-to-connect-synapse-link-cosmos-db.md?toc=/azure/cosmos-db/toc.json&bc=/azure/cosmos-db/breadcrumb/toc.json).

## <a name="create-a-database-and-views"></a>Créer une base de données et des affichages

Dans l’espace de travail Synapse, accédez à l’onglet **Développer**, sélectionnez l’icône **+** et sélectionnez **Script SQL**.

:::image type="content" source="./media/synapse-link-power-bi/add-sql-script.png" alt-text="Ajouter un script SQL à l’espace de travail Synapse Analytics":::

Chaque espace de travail est fourni avec un point de terminaison SQL sans serveur. Après avoir créé un script SQL, à partir de la barre d’outils du haut, connectez-vous à **Intégré**.

:::image type="content" source="./media/synapse-link-power-bi/enable-sql-on-demand-endpoint.png" alt-text="Autoriser le script SQL à utiliser le point de terminaison SQL sans serveur dans l’espace de travail":::

Créez une nouvelle base de données nommée **RetailCosmosDB** et un affichage SQL sur les conteneurs pour lesquels Synapse Link est activée. La commande suivante montre comment créer une base de données :

```sql
-- Create database
Create database RetailCosmosDB
```

Ensuite, créez plusieurs affichages sur différents conteneurs Azure Cosmos compatibles avec Synapse Link. Les affichages vous permettent d’utiliser T-SQL pour joindre et interroger des données Azure Cosmos DB se trouvant dans des conteneurs différents.  Veillez à sélectionner la base de données **RetailCosmosDB** lors de la création des affichages.

Les scripts suivants montrent comment créer des affichages sur chaque conteneur. Pour des raisons de simplicité, nous allons utiliser la fonctionnalité d'[inférence automatique de schéma](analytical-store-introduction.md#analytical-schema) du pool SQL serverless sur des conteneurs compatibles avec Synapse Link :


### <a name="retailsales-view"></a>Affichage RetailSales :

```sql
-- Create view for RetailSales container
CREATE VIEW  RetailSales
AS  
SELECT  *
FROM OPENROWSET (
    'CosmosDB', N'account=<Your Azure Cosmos account name>;database=<Your Azure Cosmos database name>;region=<Your Azure Cosmos DB Region>;key=<Your Azure Cosmos DB key here>',RetailSales)
AS q1
```

Veillez à insérer votre région Azure Cosmos DB et la clé primaire dans le script SQL précédent. Tous les caractères du nom de la région doivent être des minuscules sans espaces. Contrairement aux autres paramètres de la commande `OPENROWSET`, le paramètre Container Name doit être spécifié sans guillemets.

### <a name="storedemographics-view"></a>Affichage StoreDemographics :

```sql
-- Create view for StoreDemographics container
CREATE VIEW StoreDemographics
AS  
SELECT  *
FROM OPENROWSET (
    'CosmosDB', N'account=<Your Azure Cosmos account name>;database=<Your Azure Cosmos database name>;region=<Your Azure Cosmos DB Region>;key=<Your Azure Cosmos DB key here>', StoreDemographics)
AS q1
```

À présent, exécutez le script SQL en sélectionnant la commande **Exécuter** .

## <a name="query-the-views"></a>Interroger les affichages

Maintenant que les deux affichages sont créés, nous allons définir la requête pour les joindre comme suit :

```sql
SELECT 
sum(p.[revenue]) as revenue
,p.[advertising]
,p.[storeId]
,p.[weekStarting]
,q.[largeHH]
 FROM [dbo].[RetailSales] as p
INNER JOIN [dbo].[StoreDemographics] as q ON q.[storeId] = p.[storeId]
GROUP BY p.[advertising], p.[storeId], p.[weekStarting], q.[largeHH]
```

Sélectionnez **Exécuter** qui donne le résultat suivant à la table :

:::image type="content" source="./media/synapse-link-power-bi/join-views-query-results.png" alt-text="Résultats de la requête après avoir joint les affichages StoreDemographics et RetailSales":::

## <a name="model-views-over-containers-with-power-bi"></a>Affichage de modèle sur des conteneurs avec Power BI

Ensuite, ouvrez le bureau Power BI et connectez-vous au point de terminaison SQL sans serveur en procédant comme suit :

1. Ouvrez l’application Power BI Desktop. Sélectionnez **Obtenir des données** et sélectionnez **Plus**.

1. Choisissez **Azure Synapse Analytics (SQL DW)** dans la liste des options de connexion.

1. Entrez le nom du point de terminaison SQL où se trouve la base de données. Entrez `SynapseLinkBI-ondemand.sql.azuresynapse.net` dans le champ **Serveur** Dans cet exemple, **SynapseLinkBI** est le nom de l’espace de travail. Remplacez-le si vous avez attribué un nom différent à votre espace de travail. Sélectionnez **Requête directe** pour le mode de connectivité des données, puis **OK**.

1. Sélectionnez la méthode d’authentification par défaut, par exemple Azure AD.

1. Sélectionnez la base de données **RetailCosmosDB** et les affichages **RetailSales**, **StoreDemographics**.

1. Sélectionnez **Charger** pour charger les deux affichages dans le mode de requête directe.

1. Sélectionnez **Modèle** pour créer une relation entre les deux affichages via la colonne **storeId**.

1. Faites glisser la colonne **StoreId** de l’affichage **RetailSales** vers la colonne **StoreId** de l’affichage **StoreDemographics**.

1. Sélectionnez la relation plusieurs-à-un (* : 1), car il existe plusieurs lignes avec le même ID de magasin dans la vue **RetailSales**. **StoreDemographics** n’a qu’une seule ligne d’ID de magasin (il s’agit d’une table de dimension).

À présent, accédez à la fenêtre du **rapport** et créez un rapport pour comparer l’importance relative de la taille du ménage au revenu moyen par magasin, en fonction de la représentation éparpillée du chiffre d’affaires et de l’index LargeHH :

1. Sélectionnez **Graphique à nuages de points**.

1. Glissez-déplacez **LargeHH** à partir de l’affichage **StoreDemographics** sur l’axe X.

1. Glissez-déplacez **Chiffre d’affaires** entre l’affichage **RetailSales** et l’axe Y. Sélectionnez **Moyenne** pour connaître le nombre moyen de ventes par produit et par semaine.

1. Glissez-déplacez **productCode** entre l’affichage **RetailSales** et la légende pour sélectionner une gamme de produits spécifique.
Une fois ces options choisies, un graphique semblable à la capture d’écran suivante doit s’afficher :

:::image type="content" source="./media/synapse-link-power-bi/household-size-average-revenue-report.png" alt-text="Rapport comparant l’importance relative de la taille du ménage au revenu moyen par magasin":::

## <a name="next-steps"></a>Étapes suivantes

[Utiliser T-SQL pour interroger des données d’Azure Cosmos DB à l’aide d’Azure Synapse Link](../synapse-analytics/sql/query-cosmos-db-analytical-store.md)

Utiliser un pool SQL serverless pour [analyser Azure Open Datasets et visualiser les résultats dans Azure Synapse Studio](../synapse-analytics/sql/tutorial-data-analyst.md)
