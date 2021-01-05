---
title: Exécuter des travaux U-SQL en Python, R et C# - Azure Data Lake Analytics
description: Découvrez comment utiliser du code-behind avec Python, R et C# pour soumettre des travaux dans Azure Data Lake.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 11/22/2017
ms.custom: devx-track-python
ms.openlocfilehash: d6066bd6ec2a4c986ae17ad0cce3e7f6f73b21e7
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92219970"
---
# <a name="develop-u-sql-with-python-r-and-c-for-azure-data-lake-analytics-in-visual-studio-code"></a>Développer U-SQL avec Python, R et C# pour Azure Data Lake Analytics dans Visual Studio Code
Découvrez comment utiliser Visual Studio Code (VSCode) pour écrire du code-behind Python, R et C# avec U-SQL et soumettre des travaux au service Azure Data Lake. Pour plus d’informations sur Azure Data Lake Tools pour VSCode, consultez [Utilisation d’Azure Data Lake Tools pour Visual Studio Code](data-lake-analytics-data-lake-tools-for-vscode.md).

Pour pouvoir écrire du code personnalisé code-behind, vous devez ouvrir un dossier ou un espace de travail dans VSCode.


## <a name="prerequisites-for-python-and-r"></a>Prérequis pour Python et R
Inscrivez les assemblys d’extensions R et Python pour votre compte ADL. 
1. Ouvrez votre compte sur le portail.
   - Sélectionnez **Vue d’ensemble**. 
   - Cliquez sur **Exemple de script**.
2. Cliquez sur **Plus**.
3. Sélectionnez **Installer des extensions U-SQL**. 
4. Un message de confirmation s’affiche une fois que les extensions U-SQL sont installées. 

   ![Configurer l’environnement pour Python et R](./media/data-lake-analytics-data-lake-tools-for-vscode/setup-the-enrionment-for-python-and-r.png)

   > [!Note]
   > Pour optimiser les expériences sur le service de langage R et Python, veuillez installer l’extension VSCode Python et R. 

## <a name="develop-python-file"></a>Développer un fichier Python
1. Cliquez sur le **Nouveau fichier** dans votre espace de travail.
2. Écrivez votre code en U-SQL. Voici un exemple de code.
    ```U-SQL
    REFERENCE ASSEMBLY [ExtPython];
    @t  = 
        SELECT * FROM 
        (VALUES
            ("D1","T1","A1","@foo Hello World @bar"),
            ("D2","T2","A2","@baz Hello World @beer")
        ) AS 
            D( date, time, author, tweet );

    @m  =
        REDUCE @t ON date
        PRODUCE date string, mentions string
        USING new Extension.Python.Reducer("pythonSample.usql.py", pyVersion : "3.5.1");

    OUTPUT @m
        TO "/tweetmentions.csv"
        USING Outputters.Csv();
    ```
    
3. Cliquez avec le bouton droit sur un fichier de script, puis sélectionner **ADL: Generate Python Code Behind File** (ADL : Générer un fichier code-behind Python). 
4. Le fichier **xxx.usql.py** est généré dans votre dossier de travail. Écrivez votre code dans un fichier Python. Voici un exemple de code.

    ```Python
    def get_mentions(tweet):
        return ';'.join( ( w[1:] for w in tweet.split() if w[0]=='@' ) )

    def usqlml_main(df):
        del df['time']
        del df['author']
        df['mentions'] = df.tweet.apply(get_mentions)
        del df['tweet']
        return df
    ```
5. Cliquez avec le bouton droit sur le fichier **USQL**. Vous pouvez cliquer sur **Compile Script** (Compiler le script) ou sur **Submit Job** (Soumettre le travail) pour un travail en cours d’exécution.

## <a name="develop-r-file"></a>Développer un fichier R
1. Cliquez sur le **Nouveau fichier** dans votre espace de travail.
2. Écrivez votre code dans le fichier U-SQL. Voici un exemple de code.
    ```U-SQL
    DEPLOY RESOURCE @"/usqlext/samples/R/my_model_LM_Iris.rda";
    DECLARE @IrisData string = @"/usqlext/samples/R/iris.csv";
    DECLARE @OutputFilePredictions string = @"/my/R/Output/LMPredictionsIris.txt";
    DECLARE @PartitionCount int = 10;

    @InputData =
        EXTRACT SepalLength double,
                SepalWidth double,
                PetalLength double,
                PetalWidth double,
                Species string
        FROM @IrisData
        USING Extractors.Csv();

    @ExtendedData =
        SELECT Extension.R.RandomNumberGenerator.GetRandomNumber(@PartitionCount) AS Par,
            SepalLength,
            SepalWidth,
            PetalLength,
            PetalWidth
        FROM @InputData;

    // Predict Species

    @RScriptOutput =
        REDUCE @ExtendedData
        ON Par
        PRODUCE Par,
                fit double,
                lwr double,
                upr double
        READONLY Par
        USING new Extension.R.Reducer(scriptFile : "RClusterRun.usql.R", rReturnType : "dataframe", stringsAsFactors : false);
    OUTPUT @RScriptOutput
    TO @OutputFilePredictions
    USING Outputters.Tsv();
    ```
3. Cliquez avec le bouton droit sur le fichier **USQL**, puis sélectionnez **ADL: Generate R Code Behind File** (ADL : Générer un fichier code-behind R). 
4. Le fichier **xxx.usql.r** est généré dans votre dossier de travail. Écrivez votre code dans le fichier R. Voici un exemple de code.

    ```R
    load("my_model_LM_Iris.rda")
    outputToUSQL=data.frame(predict(lm.fit, inputFromUSQL, interval="confidence"))
    ```
5. Cliquez avec le bouton droit sur le fichier **USQL**. Vous pouvez cliquer sur **Compile Script** (Compiler le script) ou sur **Submit Job** (Soumettre le travail) pour un travail en cours d’exécution.

## <a name="develop-c-file"></a>Développer un fichier C#
Un fichier code-behind est un fichier C# associé à un script U-SQL. Vous pouvez définir un script dédié à UDO, UDA, UDT et UDF dans le fichier code-behind. UDO, UDA, UDT et UDF peuvent être utilisés directement dans le script sans inscription préalable de l’assembly. Le fichier code-behind est placé dans le même dossier que le fichier de script U-SQL correspondant. Si le script est nommé xxx.usql, le fichier code-behind est nommé xxx.usql.cs. Si vous supprimez manuellement le fichier code-behind, la fonctionnalité code-behind est désactivée pour le script U-SQL associé. Pour plus d’informations sur l’écriture de code client pour le script U-SQL, consultez [Écriture et utilisation de code personnalisé dans U-SQL - Fonctions définies par l’utilisateur]( https://blogs.msdn.microsoft.com/visualstudio/2015/10/28/writing-and-using-custom-code-in-u-sql-user-defined-functions/).

1. Cliquez sur le **Nouveau fichier** dans votre espace de travail.
2. Écrivez votre code dans le fichier U-SQL. Voici un exemple de code.
    ```U-SQL
    @a = 
        EXTRACT 
            Iid int,
        Starts DateTime,
        Region string,
        Query string,
        DwellTime int,
        Results string,
        ClickedUrls string 
        FROM @"/Samples/Data/SearchLog.tsv" 
        USING Extractors.Tsv();

    @d =
        SELECT DISTINCT Region 
        FROM @a;

    @d1 = 
        PROCESS @d
        PRODUCE 
            Region string,
        Mkt string
        USING new USQLApplication_codebehind.MyProcessor();

    OUTPUT @d1 
        TO @"/output/SearchLogtest.txt" 
        USING Outputters.Tsv();
    ```
3. Cliquez avec le bouton droit sur le fichier **USQL**, puis sélectionner **ADL: Generate CS Code Behind File** (ADL : Générer un fichier code-behind CS). 
4. Le fichier **xxx.usql.cs** est généré dans votre dossier de travail. Écrivez votre code dans le fichier CS. Voici un exemple de code.

    ```CS
    namespace USQLApplication_codebehind
    {
        [SqlUserDefinedProcessor]

        public class MyProcessor : IProcessor
        {
            public override IRow Process(IRow input, IUpdatableRow output)
            {
                output.Set(0, input.Get<string>(0));
                output.Set(1, input.Get<string>(0));
                return output.AsReadOnly();
            } 
        }
    }
    ```
5. Cliquez avec le bouton droit sur le fichier **USQL**. Vous pouvez cliquer sur **Compile Script** (Compiler le script) ou sur **Submit Job** (Soumettre le travail) pour un travail en cours d’exécution.

## <a name="next-steps"></a>Étapes suivantes
* [Utilisation d’Azure Data Lake Tools pour Visual Studio Code](data-lake-analytics-data-lake-tools-for-vscode.md)
* [Exécution locale et débogage local U-SQL avec Visual Studio Code](data-lake-tools-for-vscode-local-run-and-debug.md)
* [Prise en main de Data Lake Analytics à l’aide de PowerShell](data-lake-analytics-get-started-powershell.md)
* [Prise en main de Data Lake Analytics à l’aide du portail Azure](data-lake-analytics-get-started-portal.md)
* [Utiliser les outils Data Lake pour Visual Studio pour le développement d’applications U-SQL](data-lake-analytics-data-lake-tools-get-started.md)
* [Utilisation du catalogue Data Lake Analytics (U-SQL)](./data-lake-analytics-u-sql-get-started.md)