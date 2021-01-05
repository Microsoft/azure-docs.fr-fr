---
title: 'Tutoriel : Connecter un pool SQL serverless à Power BI Desktop et créer un rapport'
description: Dans ce tutoriel, vous allez voir comment connecter un pool SQL serverless dans Azure Synapse Analytics à Power BI Desktop, et comment créer un rapport de démonstration basé sur une vue.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: sql
ms.date: 05/20/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 6b45bad7c439fb98737f0caee08b2996323fbd3b
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602697"
---
# <a name="tutorial-use-serverless-sql-pool-with-power-bi-desktop--create-a-report"></a>Tutoriel : Utiliser un pool SQL serverless avec Power BI Desktop et créer un rapport

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
>
> - Créer une base de données de démonstration
> - Créer une vue utilisée pour le rapport
> - Connecter Power BI Desktop à un pool SQL serverless
> - Créer un rapport basé sur une vue

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce didacticiel, vous avez besoin de ce qui suit :

- [Power BI Desktop](https://powerbi.microsoft.com/downloads/) : nécessaire pour visualiser les données et créer un rapport.
- [Espace de travail Azure Synapse](https://docs.microsoft.com/azure/synapse-analytics/quickstart-synapse-studio) : nécessaire pour créer une base de données, une source de données externe et une vue.

Facultatif :

- Un outil de requête SQL, tel qu’[Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio) ou [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms).

Valeurs pour les paramètres suivants :

| Paramètre                                 | Description                                                   |
| ----------------------------------------- | ------------------------------------------------------------- |
| Adresse du point de terminaison de service du pool SQL serverless    | Utilisée comme nom de serveur                                   |
| Région du point de terminaison de service du pool SQL serverless     | Utilisée pour déterminer le stockage utilisé dans les exemples. |
| Nom d’utilisateur et mot de passe pour l’accès au point de terminaison | Utilisés pour accéder au point de terminaison                               |
| La base de données que vous allez utiliser pour créer des vues     | Base de données utilisée comme point de départ dans les exemples       |

## <a name="1---create-database"></a>1 - Créer la base de données

Pour l’environnement de démonstration, créez votre propre base de données de démonstration. Vous utilisez cette base de données pour voir les métadonnées, et non pour stocker les données réelles.

Créez la base de données de démonstration (et supprimez une base de données existante, si nécessaire) en exécutant le script Transact-SQL (T-SQL) suivant :

```sql
-- Drop database if it exists
DROP DATABASE IF EXISTS Demo
GO

-- Create new database
CREATE DATABASE [Demo];
GO
```

## <a name="2---create-data-source"></a>2 - Créer une source de données

Une source de données est nécessaire pour permettre au service de pool SQL serverless d’accéder aux fichiers dans le stockage. Créez la source de données pour un compte de stockage qui se trouve dans la même région que votre point de terminaison. Même si le pool SQL serverless peut accéder aux comptes de stockage dans différentes régions, le fait d’avoir le stockage et le point de terminaison dans la même région offre de meilleures performances.

Créez la source de données en exécutant le script Transact-SQL (T-SQL) suivant :

```sql
-- There is no credential in data surce. We are using public storage account which doesn't need a secret.
CREATE EXTERNAL DATA SOURCE AzureOpenData
WITH ( LOCATION = 'https://azureopendatastorage.blob.core.windows.net/')
```

## <a name="3---prepare-view"></a>3 - Préparer la vue

Exécutez le script Transact-SQL (T-SQL) suivant pour créer la vue à partir des données de démonstration externes que doit consommer Power BI :

Créez la vue `usPopulationView` à l’intérieur de la base de données `Demo` avec la requête suivante :

```sql
DROP VIEW IF EXISTS usPopulationView;
GO

CREATE VIEW usPopulationView AS
SELECT
    *
FROM
    OPENROWSET(
        BULK 'censusdatacontainer/release/us_population_county/year=20*/*.parquet',
        DATA_SOURCE = 'AzureOpenData',
        FORMAT='PARQUET'
    ) AS uspv;
```

Les données de démonstration contiennent les jeux de données suivants :

La population des États-Unis par sexe et race pour chaque comté des États-Unis à partir des recensements décennaux de 2000 et 2010 au format Parquet.

| Chemin d’accès du dossier                                                  | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| /release/                                                    | Dossier parent des données du compte de stockage de démonstration               |
| /release/us_population_county/                               | Les fichiers de données de la population des États-Unis au format Parquet, partitionnés par année à l’aide du schéma de partitionnement Hive/Hadoop. |

## <a name="4---create-power-bi-report"></a>4 - Créer un rapport Power BI

Créez le rapport pour Power BI Desktop en effectuant les étapes suivantes :

1. Ouvrez l’application Power BI Desktop et sélectionnez **Obtenir les données**.

   ![Ouvrez l’application Power BI Desktop et sélectionnez « Obtenir les données ».](./media/tutorial-connect-power-bi-desktop/step-0-open-powerbi.png)

2. Sélectionnez **Azure** > **Azure SQL Database**. 

   ![Sélectionnez la source de données.](./media/tutorial-connect-power-bi-desktop/step-1-select-data-source.png)

3. Tapez le nom du serveur sur lequel se trouve la base de données dans le champ **Serveur**, puis tapez `Demo` comme nom de base de données. Sélectionnez l’option **Importer**, puis sélectionnez **OK**. 

   ![Sélectionnez la base de données sur le point de terminaison.](./media/tutorial-connect-power-bi-desktop/step-2-db.png)

4. Sélectionnez la méthode d’authentification par défaut :

    - Exemple pour AAD 
  
        ![Cliquez sur Se connecter.](./media/tutorial-connect-power-bi-desktop/step-2.1-select-aad-auth.png)

    - Exemple pour une connexion SQL : entrez votre nom d’utilisateur et votre mot de passe.

        ![Utilisez une connexion SQL.](./media/tutorial-connect-power-bi-desktop/step-2.2-select-sql-auth.png)


5. Sélectionnez la vue `usPopulationView`, puis sélectionnez **Charger**. 

   ![Sélectionnez une vue dans la base de données sélectionnée.](./media/tutorial-connect-power-bi-desktop/step-3-select-view.png)

6. Quand l’opération se termine, une fenêtre contextuelle s’affiche indiquant `There are pending changes in your queries that haven't been applied`. Sélectionnez **Appliquer les modifications**. 

   ![Cliquez sur Appliquer les modifications.](./media/tutorial-connect-power-bi-desktop/step-4-apply-changes.png)

7. Attendez la disparition de la boîte de dialogue **Appliquer les modifications de la requête**, ce qui peut prendre quelques minutes. 

   ![Attendez la fin de l’exécution d’une requête.](./media/tutorial-connect-power-bi-desktop/step-5-wait-for-query-to-finish.png)

8. Une fois le chargement terminé, sélectionnez les colonnes suivantes dans l’ordre indiqué pour créer le rapport :
   - countyName
   - remplissage
   - stateName

   ![Sélectionnez les colonnes dignes d’intérêt pour générer un rapport cartographique.](./media/tutorial-connect-power-bi-desktop/step-6-select-columns-of-interest.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Une fois que vous avez fini d’utiliser ce rapport, supprimez les ressources en effectuant les étapes suivantes :

1. Supprimer les informations d’identification du compte de stockage

   ```sql
   DROP EXTERNAL DATA SOURCE AzureOpenData
   ```

2. Supprimer la vue

   ```sql
   DROP VIEW usPopulationView;
   ```

3. Supprimer la base de données

   ```sql
   DROP DATABASE Demo;
   ```

## <a name="next-steps"></a>Étapes suivantes

Passez à la page [Interroger des fichiers de stockage](develop-storage-files-overview.md) pour savoir comment interroger des fichiers de stockage à l’aide de Synapse SQL.
