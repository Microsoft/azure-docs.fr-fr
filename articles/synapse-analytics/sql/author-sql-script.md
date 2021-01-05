---
title: Scripts SQL dans Synapse Studio
description: Présentation des scripts SQL Synapse Studio dans Azure Synapse Analytics.
services: synapse-analytics
author: pimorano
ms.service: synapse-analytics
ms.subservice: sql
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: pimorano
ms.reviewer: omafnan
ms.openlocfilehash: 077815d117af0328a3466346727e135caaf8ff21
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96458002"
---
# <a name="synapse-studio-sql-scripts-in-azure-synapse-analytics"></a>Scripts SQL Synapse Studio dans Azure Synapse Analytics 

Synapse Studio fournit une interface web de script SQL qui vous permet de créer des requêtes SQL. 

## <a name="begin-authoring-in-sql-script"></a>Commencer à créer dans un script SQL 

Il existe plusieurs façons de démarrer l’expérience de création dans un script SQL. Vous pouvez créer un script SQL à l’aide de l’une des méthodes suivantes.

1. Dans le menu Développer, sélectionnez l’icône de **« + »** , puis choisissez **Script SQL**.

2. Dans le menu **Actions**, choisissez **Nouveau script SQL**.

3. Choisir **Importer** dans le menu **Actions** sous Développer des scripts SQL. Sélectionnez un script SQL existant dans votre stockage local.
![new sql script 3 actions](media/author-sql-script/new-sql-script-3-actions.png)

## <a name="create-your-sql-script"></a>Créer votre script SQL

1. Choisissez un nom pour votre script SQL en sélectionnant le bouton **Propriété**, puis en remplaçant le nom par défaut attribué au script. 
![new sql script rename](media/author-sql-script/new-sql-script-rename.png)

2. Choisissez le pool SQL dédié ou le pool SQL serverless spécifique dans le menu déroulant **Se connecter à** . Ou bien, si nécessaire, choisissez la base de données dans **Utiliser la base de données**. 
![new sql choose pool](media/author-sql-script/new-sql-choose-pool.png)

3. Commencez à créer votre script SQL à l’aide de la fonctionnalité intellisense.

## <a name="run-your-sql-script"></a>Exécuter votre script SQL

Sélectionnez le bouton **Exécuter** pour exécuter votre script SQL. Par défaut, les résultats sont affichés dans une table.

![table de résultats du nouveau script sql](media/author-sql-script/new-sql-script-results-table.png)

## <a name="export-your-results"></a>Exporter vos résultats

Vous pouvez exporter les résultats vers votre stockage local dans différents formats (CSV, Excel, JSON, XML) en sélectionnant « Exporter les résultats », puis en choisissant l’extension.

Vous pouvez également visualiser les résultats du script SQL dans un graphique en sélectionnant le bouton **Graphique**. Sélectionnez le « Type de graphique » et la **Colonne Catégorie**. Vous pouvez exporter le graphique dans une image en sélectionnant **Enregistrer en tant qu’image**. 

![graphique de résultats du nouveau script sql](media/author-sql-script/new-sql-script-results-chart.png)

## <a name="explore-data-from-a-parquet-file"></a>Explorer les données d’un fichier Parquet

Vous pouvez explorer des fichiers Parquet dans un compte de stockage à l’aide du script SQL pour obtenir un aperçu de leur contenu.

![parquet du nouveau script sql](media/author-sql-script/new-script-sqlod-parquet.png)

## <a name="sql-tables-external-tables-views"></a>Tables SQL, tables externes, affichages

En sélectionnant le menu **Actions** sous les données, vous pouvez choisir plusieurs actions telles que :

- Nouveau script SQL
- Sélectionner les 1000 lignes du haut
- CREATE
- DROP et CREATE 
 
Explorez les mouvements disponibles en cliquant avec le bouton droit sur les nœuds de base de données SQL.
 
![base de données du nouveau script sql](media/author-sql-script/new-script-database.png)

## <a name="create-folders-and-move-sql-scripts-into-a-folder"></a>Créer des dossiers et déplacer des scripts SQL dans un dossier

Dans le menu Actions, sous Développer des scripts SQL, choisissez « Nouveau dossier ». Saisissez ensuite le nom du nouveau dossier dans la fenêtre contextuelle. 

> [!div class="mx-imgBorder"] 
> ![newsqlscript](./media/author-sql-script/new-sql-script-create-folder.png)

Pour déplacer un script SQL dans un dossier, vous pouvez sélectionner le script SQL et choisir « Déplacer vers » dans le menu Actions. Recherchez ensuite le dossier de destination dans la nouvelle fenêtre et déplacez le script SQL dans le dossier sélectionné. Vous pouvez également faire glisser rapidement le script SQL et le déposer dans un dossier.  

> [!div class="mx-imgBorder"] 
> ![newsqlscript](./media/author-sql-script/new-sql-script-move-folder.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la création d’un script SQL, consultez [Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics).
