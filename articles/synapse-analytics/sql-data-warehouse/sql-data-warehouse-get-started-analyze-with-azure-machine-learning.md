---
title: Analyse des données avec Azure Machine Learning
description: Utilisez Azure Machine Learning pour générer un modèle Machine Learning prédictif basé sur les données stockées dans Azure Synapse.
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: machine-learning
ms.date: 07/15/2020
ms.author: martinle
ms.reviewer: igorstan
ms.custom: seo-lt-2019
tag: azure-Synapse
ms.openlocfilehash: ea4038e88d41a089958d4199e4c5a00f0d2acabd
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92015564"
---
# <a name="analyze-data-with-azure-machine-learning"></a>Analyse des données avec Azure Machine Learning

Ce didacticiel utilise le [concepteur Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/concept-designer) pour générer un modèle Machine Learning prédictif. Le modèle est basé sur les données stockées dans Azure Synapse. Le scénario de ce didacticiel crée une campagne marketing ciblée pour Adventure Works, le magasin de vélos, en prévoyant si un client est susceptible d’acheter ou non un vélo.

## <a name="prerequisites"></a>Prérequis

Pour exécuter pas à pas ce didacticiel, vous avez besoin des éléments suivants :

* un pool SQL préchargé avec les exemples de données AdventureWorksDW. Pour l’approvisionner, consultez [Créer un pool SQL](create-data-warehouse-portal.md) et chargez les données d’exemple. Si vous disposez déjà d’un entrepôt de données, mais sans disposer d’exemples de données, vous pouvez [charger manuellement des exemples de données](load-data-from-azure-blob-storage-using-polybase.md).
* Un espace de travail Azure Machine Learning. Suivez [ce didacticiel](https://docs.microsoft.com/azure/machine-learning/how-to-manage-workspace) pour en créer un nouveau.

## <a name="get-the-data"></a>Obtenir les données

Les données utilisées sont indiquées dans la vue dbo.vTargetMail dans AdventureWorksDW. Pour utiliser la Banque de données dans ce didacticiel, les données sont d’abord exportées pour le compte Azure Data Lake Storage, car Azure Synapse ne prend pas actuellement en charge les jeux de données. Azure Data Factory peut être utilisé pour exporter des données de l’entrepôt de données vers Azure Data Lake Storage à l’aide de [l’activité de copie](https://docs.microsoft.com/azure/data-factory/copy-activity-overview). Utilisez la requête suivante pour importer :

```sql
SELECT [CustomerKey]
  ,[GeographyKey]
  ,[CustomerAlternateKey]
  ,[MaritalStatus]
  ,[Gender]
  ,cast ([YearlyIncome] as int) as SalaryYear
  ,[TotalChildren]
  ,[NumberChildrenAtHome]
  ,[EnglishEducation]
  ,[EnglishOccupation]
  ,[HouseOwnerFlag]
  ,[NumberCarsOwned]
  ,[CommuteDistance]
  ,[Region]
  ,[Age]
  ,[BikeBuyer]
FROM [dbo].[vTargetMail]
```

Une fois les données disponibles dans Azure Data Lake Storage, les banques de données dans Azure Machine Learning sont utilisées pour [se connecter aux services de stockage Azure](https://docs.microsoft.com/azure/machine-learning/how-to-access-data). Suivez les étapes ci-dessous pour créer un magasin de données et un jeu de données correspondant :

1. Lancez Azure Machine Learning studio à partir du portail Azure ou connectez-vous à [Azure Machine Learning studio](https://ml.azure.com/).

1. Cliquez sur **Magasins de données** dans le volet gauche de la section **Gérer**, puis cliquez sur **Nouveau magasin de données**.

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/datastores-tab.png" alt-text="Capture d’écran du volet gauche de l’interface Azure Machine Learning":::

1. Indiquez un nom pour le magasin de données, sélectionnez le type « stockage Blob Azure », indiquez l’emplacement et les informations d’identification. Cliquez sur **Créer**.

1. Ensuite, cliquez sur **Jeux de données** dans le volet gauche de la section **Actifs**. Sélectionnez **Créer un jeu de données** avec l’option **À partir du magasin de données**.

1. Spécifiez le nom du jeu de données et sélectionnez le type **Tabulaire**. Ensuite, cliquez sur **Suivant** pour passer à l’étape suivante.

1. Dans la **section Sélectionner ou créer un magasin de données**, sélectionnez l’option **Banque de données précédemment créée**. Sélectionnez la base de données que vous avez créée auparavant. Cliquez sur Suivant et spécifiez les paramètres de chemin d’accès et de fichier. Veillez à spécifier l’en-tête de colonne si les fichiers en contiennent un.

1. Cliquez sur **Créer** pour créer le jeu de données.

## <a name="configure-designer-experiment"></a>Configurer l’expérience du concepteur

Ensuite, suivez les étapes ci-dessous pour configurer le concepteur :

1. Cliquez sur l’onglet **Concepteur** dans le volet gauche de la section **Auteur**.

1. Sélectionnez **Easy-to-use-prebuild modules** (modules prêts à l’emploi) pour créer un pipeline.

1. Dans le volet Paramètres à droite, spécifiez le nom du pipeline.

1. Sélectionnez également un cluster de calcul cible pour l’intégralité de l’expérience dans le bouton paramètres vers un cluster précédemment approvisionné. Fermez la fenêtre Paramètres.

## <a name="import-the-data"></a>Importer les données

1. Sélectionnez le sous-onglet **Jeux de données** dans le volet gauche sous la zone de recherche.

1. Faites glisser le jeu données que vous avez créé précédemment dans le canevas.

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/import-dataset.png" alt-text="Capture d’écran du volet gauche de l’interface Azure Machine Learning":::

## <a name="clean-the-data"></a>Nettoyer les données

Pour nettoyer les données, supprimez certaines colonnes qui sont inutiles pour le modèle. Pour ce faire, procédez comme suit :

1. Sélectionnez le sous-onglet **Modules**  dans le volet gauche.

1. Faites glisser le module **Sélectionner des colonnes dans le jeu de données** sous **Transformation des données < Manipulation** dans le canevas. Connectez ce module au module **Jeu de données**.

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/select-columns-zoomed-in.png" alt-text="Capture d’écran du volet gauche de l’interface Azure Machine Learning" lightbox="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/select-columns-zoomed-out.png":::

1. Double-cliquez sur le module pour ouvrir le volet Propriétés. Cliquez sur Modifier la colonne pour spécifier les colonnes que vous souhaitez supprimer.

1. Excluez deux colonnes : CustomerAlternateKey et GeographyKey. Cliquez sur **Enregistrer**.

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/drop-columns.png" alt-text="Capture d’écran du volet gauche de l’interface Azure Machine Learning":::

## <a name="build-the-model"></a>Générer le modèle

Les données sont fractionnées en 80-20 : 80 % pour l’apprentissage d’un modèle Machine Learning et 20 % pour tester le modèle. Des algorithmes « À deux classes » pour ce problème de classification binaire sont utilisés.

1. Faites glisser le module **Fractionner les données** dans la zone de canevas.

1. Sur le volet Propriétés, entrez 0,8 comme **Fraction de lignes dans le premier jeu de données**.

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/split-data.png" alt-text="Capture d’écran du volet gauche de l’interface Azure Machine Learning":::

1. Faites glisser le module **Arbre de décision optimisé à deux classes** dans la zone de dessin.

1. Faites glisser le module **Effectuer l'apprentissage du modèle** dans la zone de canevas. Spécifiez des entrées en le connectant aux modules **Arbre de décision optimisé à deux classes** (algorithme ML) et **Fractionnement des données** (données sur lesquelles entraîner l’algorithme).

1. Pour le module Effectuer l'apprentissage du module, dans l’option de **Colonne d’étiquette** dans le volet Propriétés, sélectionnez Modifier la colonne. Sélectionnez la colonne **BikeBuyer** comme colonne à prédire et sélectionnez **Enregistrer**.

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/label-column.png" alt-text="Capture d’écran du volet gauche de l’interface Azure Machine Learning":::

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/train-model.png" alt-text="Capture d’écran du volet gauche de l’interface Azure Machine Learning":::

## <a name="score-the-model"></a>Noter le modèle

À présent, testez la manière dont le modèle s’exécute sur les données de test. Deux algorithmes différents seront comparés pour déterminer celui qui est le plus performant. Pour ce faire, procédez comme suit :

1. Faites glisser le module **Noter un modèle** dans le canevas et connectez-le aux modules **Entraîner le modèle** et **Fractionner les données**.

1. Faites glisser **Machines de points Bayes à deux classes** dans la canevas de l’expérience. Vous allez comparer comment cet algorithme fonctionne par rapport à l’arbre de décision optimisé à deux classes.

1. Copiez et collez les modules de **Former le modèle** et le modèle **Noter le modèle** dans le canevas.

1. Faites glisser le module **Évaluer le modèle** module dans la zone de dessin pour comparer les deux algorithmes.

1. Cliquez sur **Envoyer** pour configurer l’exécution du pipeline.

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/algo-comparison-zoomed-in.png" alt-text="Capture d’écran du volet gauche de l’interface Azure Machine Learning" lightbox="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/algo-comparison-zoomed-out.png":::

1. Une fois l’exécution terminée, cliquez avec le bouton droit sur le module **Évaluer le modèle**, puis cliquez sur **Visualiser les résultats de l’évaluation**.

    :::image type="content" source="./media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/result-visualize-zoomed-out.png" alt-text="Capture d’écran du volet gauche de l’interface Azure Machine Learning":::

Les mesures fournies sont la courbe ROC, le diagramme de rappel de précision et la courbe d’élévation. En examinant ces mesures, nous pouvons voir que le premier modèle fonctionne mieux que le second. Pour examiner ce que le premier modèle a prédit, cliquez avec le bouton droit sur le module Évaluer le modèle, puis cliquez sur Visualiser le jeu de données évalué pour voir les résultats prédits.

Vous verrez deux colonnes supplémentaires ajoutées à votre jeu de données de test.

* Probabilités évaluées : probabilité qu’un client soit un acheteur potentiel de vélo.
* Étiquette de marquage : classification effectuée par le modèle – acheteur de vélo (1) ou non (0). Ce seuil de probabilité pour l’étiquetage est défini à 50 % et peut être ajusté.

En comparant la colonne BikeBuyer (réelle) avec les étiquettes de marquage (prévision), vous pouvez voir comment le modèle a fonctionné. Ensuite, vous pouvez utiliser ce modèle pour faire des prédictions pour les nouveaux clients. Vous pouvez [Publier ce modèle en tant que service Web](https://docs.microsoft.com/azure/machine-learning/tutorial-designer-automobile-price-deploy) ou écrire les résultats dans Azure synapse.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur Azure Machine Learning, reportez-vous à [Introduction à Machine Learning sur Azure](https://docs.microsoft.com/azure/machine-learning/overview-what-is-azure-ml).

En savoir plus sur le scoring intégré dans l’entrepôt de données, [ici](/sql/t-sql/queries/predict-transact-sql?view=azure-sqldw-latest).