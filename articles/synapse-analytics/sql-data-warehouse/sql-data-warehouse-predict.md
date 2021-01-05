---
title: Noter des modèles Machine Learning avec PREDICT
description: Découvrez comment noter des modèles Machine Learning à l’aide de la fonction T-SQL PREDICT dans un pool SQL dédié.
services: synapse-analytics
author: anumjs
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: machine-learning
ms.date: 07/21/2020
ms.author: anjangsh
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: ce77a169e28e21aa37be2a49997a58ee42c93807
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96510826"
---
# <a name="score-machine-learning-models-with-predict"></a>Noter des modèles Machine Learning avec PREDICT

Un pool SQL dédié vous offre la possibilité de noter des modèles Machine Learning en utilisant le langage familier T-SQL. La fonction T-SQL [PREDICT](https://docs.microsoft.com/sql/t-sql/queries/predict-transact-sql?view=azure-sqldw-latest&preserve-view=true) vous noter vos modèles Machine Learning entraînés avec des données historiques dans les limites sécurisées de votre entrepôt de données. La fonction PREDICT prend un modèle [ONNX (Open neuronal Network Exchange)](https://onnx.ai/) et des données en entrée. Cette fonctionnalité élimine l’étape de déplacement de données précieuses hors de l’entrepôt de données pour effectuer le scoring. Elle vise à permettre aux professionnels de l’informatique de déployer facilement des modèles Machine Learning avec l’interface T-SQL familière, et de collaborer de façon transparente avec des scientifiques de données travaillant avec l’infrastructure appropriée pour accomplir leur tâche.

> [!NOTE]
> Cette fonctionnalité n’est actuellement pas prise en charge dans un pool SQL serverless.

Elle requiert que le modèle soit entraîné hors de Synapse SQL. Après avoir compilé le modèle, chargez-le dans l’entrepôt de données et notez-le avec la syntaxe T-SQL PREDICT pour extraire des informations à partir des données.

![predictoverview](./media/sql-data-warehouse-predict/datawarehouse-overview.png)

## <a name="training-the-model"></a>Apprentissage du modèle

Un pool SQL dédié attend un modèle préentraîné. Lors de l’entraînement d’un modèle Machine Learning utilisé pour effectuer des prédictions dans un pool SQL dédié, gardez à l’esprit les facteurs suivants.

- Un pool SQL dédié ne prend en charge que les modèles au format ONNX. ONNX est un format de modèle open source qui vous permet d’échanger des modèles entre différentes infrastructures pour activer l’interopérabilité. Vous pouvez convertir vos modèles existants au format ONNX à l’aide d’infrastructures qui le prennent en charge en mode natif ou qui disposent de packages de conversion. Par exemple, le package [sklearn-onnx](https://github.com/onnx/sklearn-onnx) convertit des modèles scikit-learn au format ONNX. Le [dépôt GitHub ONNX](https://github.com/onnx/tutorials#converting-to-onnx-format) fournit une liste d’infrastructures et d’exemples pris en charge.

   Si vous utilisez un [Machine Learning automatisé](https://docs.microsoft.com/azure/machine-learning/concept-automated-ml) pour l’apprentissage, veillez à définir le paramètre *enable_onnx_compatible_models* sur TRUE pour produire un modèle au format ONNX. Le [Bloc-notes Machine Learning automatisé](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/classification-bank-marketing-all-features/auto-ml-classification-bank-marketing-all-features.ipynb) présente un exemple d’utilisation d’AutoML pour créer un modèle Machine Learning au format ONNX.

- Les types de données pris en charge en entrée sont les suivants :
    - int, bigint, real, float
    - char, varchar, nvarchar

- Les données de scoring doivent être au même format que les données d’apprentissage. PREDICT ne prend pas en charge les types de données complexes tels que des tableaux multidimensionnels. Ainsi, pour l’apprentissage, assurez-vous que chaque entrée du modèle correspond à une colonne unique de la table de scoring au lieu de passer un tableau unique contenant toutes les entrées.

- Assurez-vous que les noms et les types de données des entrées du modèle correspondent aux noms de colonne et aux types de données des nouvelles données de prédiction. La visualisation d’un modèle ONNX à l’aide de différents outils open source disponibles en ligne peut faciliter le débogage.

## <a name="loading-the-model"></a>Chargement du modèle

Le modèle est stocké dans une table utilisateur de pool SQL dédié sous la forme d’une chaîne hexadécimale. Des colonnes supplémentaires telles que ID et Description peuvent être ajoutées dans la table du modèle pour identifier celui-ci. Utilisez varbinary (max) comme type de données de la colonne du modèle. Voici un exemple de code pour une table utilisable pour stocker des modèles :

```sql
-- Sample table schema for storing a model and related data
CREATE TABLE [dbo].[Models]
(
    [Id] [int] IDENTITY(1,1) NOT NULL,
    [Model] [varbinary](max) NULL,
    [Description] [varchar](200) NULL
)
WITH
(
    DISTRIBUTION = ROUND_ROBIN,
    HEAP
)
GO

```

Une fois le modèle converti en chaîne hexadécimale et la définition de table spécifiée, utilisez la [commande COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest&preserve-view=true) ou PolyBase pour charger le modèle dans la table du pool SQL dédié. L’exemple de code suivant utilise la commande Copy pour charger le modèle.

```sql
-- Copy command to load hexadecimal string of the model from Azure Data Lake storage location
COPY INTO [Models] (Model)
FROM '<enter your storage location>'
WITH (
    FILE_TYPE = 'CSV',
    CREDENTIAL=(IDENTITY= 'Shared Access Signature', SECRET='<enter your storage key here>')
)
```

## <a name="scoring-the-model"></a>Scoring du modèle

Une fois le modèle et les données chargés dans l’entrepôt de données, utilisez la fonction **T-SQL PREDICT** pour noter le modèle. Assurez-vous que les nouvelles données d’entrée ont le même format que les données d’apprentissage utilisées pour générer le modèle. T-SQL PREDICT prend deux entrées, le modèle et les nouvelles données d’entrée de scoring, puis génère de nouvelles colonnes pour la sortie. Le modèle peut être spécifié en tant que variable, literal ou scalar sub_query. Utilisez [WITH common_table_expression](https://docs.microsoft.com/sql/t-sql/queries/with-common-table-expression-transact-sql?view=azure-sqldw-latest&preserve-view=true) pour spécifier un jeu de résultats nommé pour le paramètre de données.

L’exemple ci-dessous montre une requête utilisant une fonction de prédiction. Une colonne supplémentaire nommée *Score* avec le type de données *float* est créée, qui contient les résultats de la prédiction. Toutes les colonnes de données en entrée ainsi que les colonnes de prédiction en sortie peuvent être affichées avec l’instruction select. Pour plus d’informations, consultez [PREDICT (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/queries/predict-transact-sql?view=azure-sqldw-latest&preserve-view=true).

```sql
-- Query for ML predictions
SELECT d.*, p.Score
FROM PREDICT(MODEL = (SELECT Model FROM Models WHERE Id = 1),
DATA = dbo.mytable AS d, RUNTIME = ONNX) WITH (Score float) AS p;
```

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la fonction PREDICT, consultez [PREDICT (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/queries/predict-transact-sql?view=azure-sqldw-latest&preserve-view=true).
