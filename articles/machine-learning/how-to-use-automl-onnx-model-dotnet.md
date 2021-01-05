---
title: Effectuer des prédictions avec un modèle AutoML ONNX dans .NET
description: Découvrez comment effectuer des prédictions à l’aide d’un modèle AutoML ONNX dans .NET avec ML.NET
titleSuffix: Azure Machine Learning
author: luisquintanilla
ms.author: luquinta
ms.date: 10/30/2020
ms.topic: conceptual
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.custom: how-to, automl
ms.openlocfilehash: 4fb147dc5c57c3a98607a025f566fa583bf87460
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93358811"
---
# <a name="make-predictions-with-an-automl-onnx-model-in-net"></a>Effectuer des prédictions avec un modèle AutoML ONNX dans .NET

Dans cet article, vous allez apprendre à utiliser un modèle Automated ML (AutoML) Open Neural Network Exchange (ONNX) pour effectuer des prédictions dans une application console C# .NET Core avec ML.NET.

[ML.NET](/dotnet/machine-learning/) est un framework de Machine Learning, open source, multiplateforme pour l’écosystème .NET, qui vous permet d’effectuer l’apprentissage et de consommer des modèles de Machine Learning personnalisés à l’aide d’une approche orientée code en C# ou F#, ainsi que via des outils avec peu de code comme [Model Builder](/dotnet/machine-learning/automate-training-with-model-builder) et la [CLI ML.NET](/dotnet/machine-learning/automate-training-with-cli). Le framework est également extensible et vous permet de tirer parti d’autres frameworks de Machine Learning courantes comme TensorFlow et ONNX.

ONNX est un format open source pour les modèles d’IA. ONNX prend en charge l’interopérabilité entre les frameworks. Cela signifie que vous pouvez effectuer l’apprentissage d’un modèle dans l’un des nombreux frameworks de Machine Learning populaires comme PyTorch, le convertir au format ONNX et consommer le modèle ONNX dans un autre framework comme ML.NET. Pour en savoir plus, consultez le [site web ONNX](https://onnx.ai/).

## <a name="prerequisites"></a>Prérequis

- [SDK .NET Core 3.1 ou ultérieur](https://dotnet.microsoft.com/download)
- Éditeur de texte ou IDE (par exemple [Visual Studio](https://visualstudio.microsoft.com/vs/) ou [Visual Studio Code](https://code.visualstudio.com/Download))
- Modèle ONNX. Pour savoir comment effectuer l’apprentissage d’un modèle AutoML ONNX, consultez le [notebook de classification Bank Marketing](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/classification-bank-marketing-all-features/auto-ml-classification-bank-marketing-all-features.ipynb) suivants.
- [Netron](https://github.com/lutzroeder/netron) (facultatif)

## <a name="create-a-c-console-application"></a>Création d’une application console C#

Dans cet exemple, vous utilisez la CLI .NET Core pour générer votre application, mais vous pouvez effectuer les mêmes tâches à l’aide de Visual Studio. En savoir plus sur la [CLI .NET Core](/dotnet/core/tools/).

1. Ouvrez un terminal et créez une nouvelle application console C# .NET Core. Dans cet exemple, le nom de l’application est `AutoMLONNXConsoleApp`. Un répertoire est créé avec ce même nom et le contenu de votre application.

    ```dotnetcli
    dotnet new console -o AutoMLONNXConsoleApp
    ```

1. Dans le terminal, accédez au répertoire *AutoMLONNXConsoleApp*.

    ```bash
    cd AutoMLONNXConsoleApp
    ```

## <a name="add-software-packages"></a>Ajouter des packages logiciels

1. Installez les packages NuGet **Microsoft.ML** , **Microsoft.ML.OnnxRuntime** et **Microsoft.ML.OnnxTransformer** à l’aide de la CLI .NET Core.

    ```dotnetcli
    dotnet add package Microsoft.ML
    dotnet add package Microsoft.ML.OnnxRuntime
    dotnet add package Microsoft.ML.OnnxTransformer
    ```

    Ces packages contiennent les dépendances nécessaires à l’utilisation d’un modèle ONNX dans une application .NET. ML.NET fournit une API qui utilise le [runtime ONNX](https://github.com/Microsoft/onnxruntime) pour les prédictions.

1. Ouvrez le fichier *Program.cs* et ajoutez les instructions `using` suivantes en haut pour référencer les packages appropriés.

    ```csharp
    using System.Linq;
    using Microsoft.ML;
    using Microsoft.ML.Data;
    using Microsoft.ML.Transforms.Onnx;
    ```

## <a name="add-a-reference-to-the-onnx-model"></a>Ajouter une référence au modèle ONNX

Pour que l’application console accède au modèle ONNX, une méthode consiste à l’ajouter au répertoire de sortie de la génération.  Pour en savoir plus sur les éléments courants MSBuild, consultez le [guide MSBuild](/visualstudio/msbuild/common-msbuild-project-items).

Ajouter une référence à votre fichier de modèle ONNX dans votre application

1. Copiez votre modèle ONNX dans le répertoire racine *AutoMLONNXConsoleApp* de votre application.
1. Ouvrez le fichier *AutoMLONNXConsoleApp.csproj* et ajoutez le contenu suivant à l’intérieur du nœud `Project`.

    ```xml
    <ItemGroup>
        <None Include="automl-model.onnx">
            <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
        </None>
    </ItemGroup>
    ```

    Dans ce cas, le nom du fichier de modèle ONNX est *automl-model.onnx*.

1. Ouvrez le fichier *Program.cs* et ajoutez la ligne suivante dans la classe `Program`.

    ```csharp
    static string ONNX_MODEL_PATH = "automl-model.onnx";
    ```

## <a name="initialize-mlcontext"></a>Initialiser MLContext

Dans la méthode `Main` de votre classe `Program`, créez une nouvelle instance [`MLContext`](xref:Microsoft.ML.MLContext).

```csharp
MLContext mlContext = new MLContext();
```

La classe [`MLContext`](xref:Microsoft.ML.MLContext) est un point de départ pour toutes les opérations ML.NET et l’initialisation de `mlContext` crée un nouvel environnement ML.NET qui peut être partagé durant tout le cycle de vie du modèle. De manière conceptuelle, cela est similaire à DbContext dans Entity Framework.

## <a name="define-the-model-data-schema"></a>Définir le schéma de données du modèle

Votre modèle attend vos données d’entrée et de sortie dans un format spécifique. ML.NET vous permet de définir le format de vos données à l’aide de classes. Parfois, vous savez peut-être déjà à quoi ressemble ce format. Dans les cas où vous ne connaissez pas le format des données, vous pouvez utiliser des outils comme Netron pour inspecter votre modèle ONNX.

Le modèle utilisé dans cet exemple utilise les données du jeu de données « NYC TLC Taxi Trip ». Vous pouvez consulter un exemple des données ci-dessous :

|vendor_id|rate_code|passenger_count|trip_time_in_secs|trip_distance|payment_type|fare_amount|
|---|---|---|---|---|---|---|
|VTS|1|1|1140|3,75|CRD|15,5|
|VTS|1|1|480|2.72|CRD|10.0|
|VTS|1|1|1680|7.8|CSH|26,5|

### <a name="inspect-the-onnx-model-optional"></a>Inspecter le modèle ONNX (facultatif)

Utilisez un outil tel que Netron pour inspecter les entrées et les sorties de votre modèle.

1. Ouvrez Netron.
1. Dans la barre de menus supérieure, sélectionnez **Fichier > Ouvrir** et utilisez l’Explorateur de fichiers pour sélectionner votre modèle.
1. Votre modèle s’ouvre. Par exemple, la structure du modèle *automl-model.onnx* ressemble à ce qui suit :

    :::image type="content" source="media/how-to-use-automl-onnx-model-dotnet/netron-automl-onnx-model.png" alt-text="Modèle Netron AutoML ONNX":::

1. Sélectionnez le dernier nœud en bas du graphique (`variable_out1` dans ce cas) pour afficher les métadonnées du modèle. Les entrées et les sorties dans la barre latérale affichent les entrées, les sorties et les types de données attendus du modèle. Utilisez ces informations pour définir le schéma d’entrée et de sortie de votre modèle.

### <a name="define-model-input-schema"></a>Définir le schéma d’entrée du modèle

Créez une nouvelle classe appelée `OnnxInput` avec les propriétés suivantes dans le fichier *Program.cs*.

```csharp
public class OnnxInput
{
    [ColumnName("vendor_id")]
    public string VendorId { get; set; }

    [ColumnName("rate_code"),OnnxMapType(typeof(Int64),typeof(Single))]
    public Int64 RateCode { get; set; }

    [ColumnName("passenger_count"), OnnxMapType(typeof(Int64), typeof(Single))]
    public Int64 PassengerCount { get; set; }

    [ColumnName("trip_time_in_secs"), OnnxMapType(typeof(Int64), typeof(Single))]
    public Int64 TripTimeInSecs { get; set; }

    [ColumnName("trip_distance")]
    public float TripDistance { get; set; }

    [ColumnName("payment_type")]
    public string PaymentType { get; set; }
}
```

Chacune des propriétés est mappée à une colonne dans le jeu de données. Les propriétés sont aussi annotées avec des attributs.

L’attribut [`ColumnName`](xref:Microsoft.ML.Data.ColumnNameAttribute) vous permet de spécifier la manière dont ML.NET doit faire référence à la colonne lors de l’utilisation des données. Par exemple, bien que la propriété `TripDistance` suive les conventions d’affectation de noms .NET standard, le modèle ne connaît qu’une colonne ou qu’une fonctionnalité appelée `trip_distance`. Pour répondre à cette incohérence de nommage, l’attribut [`ColumnName`](xref:Microsoft.ML.Data.ColumnNameAttribute) mappe la propriété `TripDistance` à une colonne ou une fonctionnalité avec le nom `trip_distance`.
  
Pour les valeurs numériques, ML.NET fonctionne uniquement sur les types de valeurs [`Single`](xref:System.Single). Toutefois, le type de données d’origine de certaines colonnes est un entier. L’attribut [`OnnxMapType`](xref:Microsoft.ML.Transforms.Onnx.OnnxMapTypeAttribute) mappe les types entre ONNX et ML.NET.

Pour en savoir plus sur les attributs de données, consultez le [guide sur le chargement des données ML.NET](/dotnet/machine-learning/how-to-guides/load-data-ml-net).

### <a name="define-model-output-schema"></a>Définir le schéma de sortie du modèle

Une fois les données traitées, elles produisent une sortie d’un format spécifique. Définissez votre schéma de sortie de données. Créez une nouvelle classe appelée `OnnxOutput` avec les propriétés suivantes dans le fichier *Program.cs*.

```csharp
public class OnnxOutput
{
    [ColumnName("variable_out1")]
    public float[] PredictedFare { get; set; }
}
```

Comme pour `OnnxInput`, utilisez l’attribut [`ColumnName`](xref:Microsoft.ML.Data.ColumnNameAttribute) pour mapper la sortie `variable_out1` à un nom plus descriptif `PredictedFare`.

## <a name="define-a-prediction-pipeline"></a>Définir un pipeline de prédiction

Dans ML.NET, un pipeline est généralement une série de transformations chaînées qui opèrent sur les données d’entrée pour produire une sortie. Pour en savoir plus sur les transformations de données, consultez le [guide sur la transformation des données ML.NET](/dotnet/machine-learning/resources/transforms).

1. Créer une nouvelle méthode appelée `GetPredictionPipeline` dans la classe `Program`

    ```csharp
    static ITransformer GetPredictionPipeline(MLContext mlContext)
    {

    }
    ```

1. Définissez le nom des colonnes d’entrée et de sortie. Ajoutez le code suivant dans la méthode `GetPredictionPipeline`.

    ```csharp
    var inputColumns = new string []
    {
        "vendor_id", "rate_code", "passenger_count", "trip_time_in_secs", "trip_distance", "payment_type"
    };

    var outputColumns = new string [] { "variable_out1" };
    ```

1. Définissez votre pipeline. Un [`IEstimator`](xref:Microsoft.ML.IEstimator%601) fournit un plan des opérations, des schémas d’entrée et de sortie de votre pipeline.

    ```csharp
    var onnxPredictionPipeline =
        mlContext
            .Transforms
            .ApplyOnnxModel(
                outputColumnNames: outputColumns,
                inputColumnNames: inputColumns,
                ONNX_MODEL_PATH);
    ```

    Dans ce cas, [`ApplyOnnxModel`](xref:Microsoft.ML.OnnxCatalog.ApplyOnnxModel%2A) est la seule transformation dans le pipeline, qui prend les noms des colonnes d’entrée et de sortie, ainsi que le chemin d’accès au fichier de modèle ONNX.

1. Un [`IEstimator`](xref:Microsoft.ML.IEstimator%601) définit uniquement l’ensemble des opérations à appliquer à vos données. Ce qui se produit sur vos données est appelé [`ITransformer`](xref:Microsoft.ML.ITransformer). Utilisez la méthode `Fit` pour en créer un à partir de `onnxPredictionPipeline`.

    ```csharp
    var emptyDv = mlContext.Data.LoadFromEnumerable(new OnnxInput[] {});

    return onnxPredictionPipeline.Fit(emptyDv);
    ```

    La méthode [`Fit`](xref:Microsoft.ML.IEstimator%601.Fit%2A) attend un [`IDataView`](xref:Microsoft.ML.IDataView) comme entrée sur laquelle effectuer les opérations. Un [`IDataView`](xref:Microsoft.ML.IDataView) est un moyen de représenter des données dans ML.NET à l’aide d’un format tabulaire. Comme, dans ce cas, le pipeline est utilisé uniquement pour les prédictions, vous pouvez fournir un [`IDataView`](xref:Microsoft.ML.IDataView) vide pour donner à [`ITransformer`](xref:Microsoft.ML.ITransformer) les informations de schéma d’entrée et de sortie nécessaires. Le [`ITransformer`](xref:Microsoft.ML.ITransformer) ajusté est ensuite retourné pour une utilisation ultérieure dans votre application.

    > [!TIP]
    > Dans cet exemple, le pipeline est défini et utilisé dans la même application. Toutefois, il est recommandé d’utiliser des applications distinctes pour définir et utiliser votre pipeline afin d’effectuer des prédictions. Dans ML.NET, vos pipelines peuvent être sérialisés et enregistrés pour une utilisation ultérieure dans d’autres applications d’utilisateur final .NET. ML.NET prend en charge différentes cibles de déploiement, telles que les applications de bureau, les services web, les applications WebAssembly* et bien plus encore. Pour en savoir plus sur l’enregistrement des pipelines, consultez le [guide sur l’enregistrement et le chargement de modèles formés ML.NET](/dotnet/machine-learning/how-to-guides/save-load-machine-learning-models-ml-net).
    >
    > *WebAssembly est pris en charge uniquement dans .NET Core 5 ou version ultérieure

1. Dans la méthode `Main`, appelez la méthode `GetPredictionPipeline` avec les paramètres requis.

    ```csharp
    var onnxPredictionPipeline = GetPredictionPipeline(mlContext);
    ```

## <a name="use-the-model-to-make-predictions"></a>Utiliser le modèle pour élaborer des prédictions

Maintenant que vous disposez d’un pipeline, il est temps de l’utiliser pour effectuer des prédictions. ML.NET fournit une API pratique pour effectuer des prédictions sur une instance de données unique appelée [`PredictionEngine`](xref:Microsoft.ML.PredictionEngine%602).

1. Dans la méthode `Main`, créez un [`PredictionEngine`](xref:Microsoft.ML.PredictionEngine%602) à l’aide de la méthode [`CreatePredictionEngine`](xref:Microsoft.ML.ModelOperationsCatalog.CreatePredictionEngine%2A).

    ```csharp
    var onnxPredictionEngine = mlContext.Model.CreatePredictionEngine<OnnxInput, OnnxOutput>(onnxPredictionPipeline);
    ```

1. Créez une entrée de données de test.

    ```csharp
    var testInput = new OnnxInput
    {
        VendorId = "CMT",
        RateCode = 1,
        PassengerCount = 1,
        TripTimeInSecs = 1271,
        TripDistance = 3.8f,
        PaymentType = "CRD"
    };
    ```

1. Utilisez `predictionEngine` pour effectuer des prédictions basées sur les nouvelles données `testInput` à l’aide de la méthode [`Predict`](xref:Microsoft.ML.PredictionEngineBase%602.Predict%2A).

    ```csharp
    var prediction = onnxPredictionEngine.Predict(testInput);
    ```

1. Générez le résultat de votre prédiction sur la console.

    ```csharp
    Console.WriteLine($"Predicted Fare: {prediction.PredictedFare.First()}");
    ```

1. Utilisez la CLI .NET Core pour exécuter votre application.

    ```dotnetcli
    dotnet run
    ```

    Le résultat doit ressembler à la sortie suivante :

    ```text
    Predicted Fare: 15.621523
    ```

Pour en savoir plus sur l’élaboration de prédictions dans ML.NET, consultez le [guide sur l’utilisation d’un modèle pour effectuer des prédictions](/dotnet/machine-learning/how-to-guides/machine-learning-model-predictions-ml-net).

## <a name="next-steps"></a>Étapes suivantes

- [Déployer votre modèle en tant qu’API web ASP.NET Core](/dotnet/machine-learning/how-to-guides/serve-model-web-api-ml-net)
- [Déployer votre modèle en tant que fonction Azure Function .NET serverless](/dotnet/machine-learning/how-to-guides/serve-model-serverless-azure-functions-ml-net)