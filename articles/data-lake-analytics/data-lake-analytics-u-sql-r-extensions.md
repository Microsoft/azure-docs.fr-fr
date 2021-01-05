---
title: Étendre des scripts U-SQL à l’aide de code R dans Azure Data Lake Analytics
description: Découvrez comment exécuter du code R dans des scripts U-SQL à l’aide d’Azure Data Lake Analytics. Incorporez du code R en ligne ou une référence à partir de fichiers.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/20/2017
ms.openlocfilehash: ab28ba1bb6185f5f0ab90df99f0abea7895b1bf4
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92217998"
---
# <a name="extend-u-sql-scripts-with-r-code-in-azure-data-lake-analytics"></a>Étendre des scripts U-SQL à l’aide de code R dans Azure Data Lake Analytics

L’exemple suivant illustre les étapes de base pour déployer un code R :

* Utilisation de l’instruction `REFERENCE ASSEMBLY` pour activer les extensions R pour le script U-SQL.
* Utilisation de l’opération `REDUCE` pour partitionner les données d’entrée sur une clé.
* Les extensions R pour U-SQL comprennent un réducteur intégré (`Extension.R.Reducer`) qui exécute le code R sur chaque vertex affecté au réducteur.
* Utilisation de tableaux de données nommés dédiés, respectivement intitulés `inputFromUSQL` et `outputToUSQL`, pour transférer des données entre U-SQL et R. Les noms des identificateurs des tableaux de données d’entrée et de sortie sont fixes. Autrement dit, les utilisateurs ne peuvent pas modifier les noms prédéfinis des identificateurs des tableaux de données d’entrée et de sortie.

## <a name="embedding-r-code-in-the-u-sql-script"></a>Incorporation de code R dans le script U-SQL

Vous pouvez incorporer du code R à votre script U-SQL à l’aide du paramètre de commande du réducteur `Extension.R.Reducer`. Par exemple, vous pouvez déclarer le script R comme une variable de chaîne et le passer en tant que paramètre au raccord de réduction.

```usql
REFERENCE ASSEMBLY [ExtR];

DECLARE @myRScript = @"
inputFromUSQL$Species = as.factor(inputFromUSQL$Species)
lm.fit=lm(unclass(Species)~.-Par, data=inputFromUSQL)
#do not return readonly columns and make sure that the column names are the same in usql and r cripts,
outputToUSQL=data.frame(summary(lm.fit)$coefficients)
colnames(outputToUSQL) <- c(""Estimate"", ""StdError"", ""tValue"", ""Pr"")
outputToUSQL
";

@RScriptOutput = REDUCE … USING new Extension.R.Reducer(command:@myRScript, ReturnType:"dataframe");
```

## <a name="keep-the-r-code-in-a-separate-file-and-reference-it--the-u-sql-script"></a>Conserver le code R dans un fichier distinct et le référencer dans le script U-SQL

L’exemple suivant illustre un usage plus complexe. Dans ce cas, le code R est déployé comme une ressource correspondant au script U-SQL.

Enregistrez ce code R dans un fichier distinct.

```usql
load("my_model_LM_Iris.rda")
outputToUSQL=data.frame(predict(lm.fit, inputFromUSQL, interval="confidence"))
```

Utilisez un script U-SQL pour déployer ce script R avec l’instruction DEPLOY RESOURCE.

```usql
REFERENCE ASSEMBLY [ExtR];
DEPLOY RESOURCE @"/usqlext/samples/R/RinUSQL_PredictUsingLinearModelasDF.R";
DEPLOY RESOURCE @"/usqlext/samples/R/my_model_LM_Iris.rda";
DECLARE @IrisData string = @"/usqlext/samples/R/iris.csv";
DECLARE @OutputFilePredictions string = @"/my/R/Output/LMPredictionsIris.txt";
DECLARE @PartitionCount int = 10;
@InputData =
    EXTRACT
        SepalLength double,
        SepalWidth double,
        PetalLength double,
        PetalWidth double,
        Species string
    FROM @IrisData
    USING Extractors.Csv();
@ExtendedData =
    SELECT
        Extension.R.RandomNumberGenerator.GetRandomNumber(@PartitionCount) AS Par,
        SepalLength,
        SepalWidth,
        PetalLength,
        PetalWidth
    FROM @InputData;
// Predict Species
@RScriptOutput = REDUCE @ExtendedData ON Par
    PRODUCE Par, fit double, lwr double, upr double
    READONLY Par
    USING new Extension.R.Reducer(scriptFile:"RinUSQL_PredictUsingLinearModelasDF.R", rReturnType:"dataframe", stringsAsFactors:false);
    OUTPUT @RScriptOutput TO @OutputFilePredictions USING Outputters.Tsv();
```

## <a name="how-r-integrates-with-u-sql"></a>Intégration de R à U-SQL

### <a name="datatypes"></a>Types de données

* Les colonnes de chaîne et numériques de U-SQL sont converties en l’état entre un tableau de données R et U-SQL [types pris en charge : `double`, `string`, `bool`, `integer`, `byte`].
* Le type de données `Factor` n’est pas pris en charge dans U-SQL.
* `byte[]` doit être sérialisé sous forme de `string` codé en base64.
* Les chaînes U-SQL peuvent être converties en facteurs dans le code R quand U-SQL crée un tableau de données d’entrée R ou en définissant le paramètre du réducteur sur `stringsAsFactors: true`.

### <a name="schemas"></a>Schémas

* Les jeux de données U-SQL ne peuvent pas avoir de noms de colonnes en double.
* Les noms de colonnes des jeux de données U-SQL doivent être des chaînes.
* Les noms de colonnes doivent être identiques dans U-SQL et dans les scripts R.
* Les colonnes en lecture seule ne peuvent pas faire partie du tableau de données de sortie. Car les colonnes en lecture seule sont automatiquement réinjectées dans la table U-SQL si elles font partie du schéma de sortie de l’opérateur défini par l’utilisateur (UDO).

### <a name="functional-limitations"></a>Limitations fonctionnelles

* Le moteur R ne peut pas être instancié deux fois dans le même processus.
* Pour la prédiction, l’utilisation d’opérateurs de combinateur définis par l’utilisateur avec des modèles partitionnés générés à l’aide d’opérateurs de réducteur définis par l’utilisateur n’est pas prise en charge par U-SQL. Les utilisateurs peuvent déclarer les modèles partitionnés comme des ressources et les utiliser dans leur script R (voir l’exemple de code `ExtR_PredictUsingLMRawStringReducer.usql`)

### <a name="r-versions"></a>Versions de R

Seul R 3.2.2 est pris en charge.

### <a name="standard-r-modules"></a>Modules R standard

```usql
base
boot
Class
Cluster
codetools
compiler
datasets
doParallel
doRSR
foreach
foreign
Graphics
grDevices
grid
iterators
KernSmooth
lattice
MASS
Matrix
Methods
mgcv
nlme
Nnet
Parallel
pkgXMLBuilder
RevoIOQ
revoIpe
RevoMods
RevoPemaR
RevoRpeConnector
RevoRsrConnector
RevoScaleR
RevoTreeView
RevoUtils
RevoUtilsMath
Rpart
RUnit
spatial
splines
Stats
stats4
survival
Tcltk
Tools
translations
utils
XML
```

### <a name="input-and-output-size-limitations"></a>Limitations de taille d’entrée et de sortie

Chaque vertex possède une quantité limitée de mémoire qui lui est assignée. Étant donné que les tableaux de données d’entrée et de sortie doivent exister dans la mémoire dans le code R, la taille totale de l’entrée et de la sortie ne peut pas dépasser 500 Mo.

### <a name="sample-code"></a>Exemple de code

Des exemples de code supplémentaires sont mis à disposition dans votre compte Data Lake Store après l’installation des extensions analytiques avancées U-SQL. Pour obtenir des exemples de code supplémentaires, suivez ce chemin : `<your_account_address>/usqlext/samples/R`.

## <a name="deploying-custom-r-modules-with-u-sql"></a>Déploiement des modules R personnalisés avec U-SQL

Tout d’abord, créez un module R personnalisé, incluez-le dans un fichier zip, puis chargez le fichier de module R personnalisé zippé dans votre magasin ADL. Dans l’exemple, nous allons charger magittr_1.5.zip vers la racine du compte ADLS par défaut pour le compte ADLA que nous utilisons. Une fois le module chargé dans le magasin ADL, déclarez-le à l’aide de DEPLOY RESOURCE pour le rendre disponible dans votre script U-SQL, puis appelez `install.packages` pour l’installer.

```usql
REFERENCE ASSEMBLY [ExtR];
DEPLOY RESOURCE @"/magrittr_1.5.zip";
DECLARE @IrisData string =  @"/usqlext/samples/R/iris.csv";
DECLARE @OutputFileModelSummary string = @"/R/Output/CustomPackages.txt";
// R script to run
DECLARE @myRScript = @"
# install the magrittr package,
install.packages('magrittr_1.5.zip', repos = NULL),
# load the magrittr package,
require(magrittr),
# demonstrate use of the magrittr package,
2 %>% sqrt
";
@InputData =
EXTRACT SepalLength double,
SepalWidth double,
PetalLength double,
PetalWidth double,
Species string
FROM @IrisData
USING Extractors.Csv();
@ExtendedData =
SELECT 0 AS Par,
*
FROM @InputData;
@RScriptOutput = REDUCE @ExtendedData ON Par
PRODUCE Par, RowId int, ROutput string
READONLY Par
USING new Extension.R.Reducer(command:@myRScript, rReturnType:"charactermatrix");
OUTPUT @RScriptOutput TO @OutputFileModelSummary USING Outputters.Tsv();
```

## <a name="next-steps"></a>Étapes suivantes

* [Vue d'ensemble de Microsoft Azure Data Lake Analytics](data-lake-analytics-overview.md)
* [Développer des scripts de U-SQL à l’aide d’outils Data Lake Tools pour Visual Studio](data-lake-analytics-data-lake-tools-get-started.md)
* [Utilisation des fonctions U-SQL dans les travaux Analytique Data Lake Azure](./data-lake-analytics-u-sql-get-started.md)