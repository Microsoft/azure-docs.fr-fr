---
title: Exécuter des travaux U-SQL localement - SDK Azure Data Lake U-SQL
description: Découvrez comment exécuter et tester localement des travaux U-SQL avec les interfaces de ligne de commande et de programmation sur votre station de travail locale.
ms.service: data-lake-analytics
author: yanacai
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 03/01/2017
ms.openlocfilehash: 8fb60e62a63bfc4562f19d483dc84c99c37676b0
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92215533"
---
# <a name="run-and-test-u-sql-with-azure-data-lake-u-sql-sdk"></a>Exécuter et tester U-SQL à l’aide du Kit de développement logiciel (SDK) Azure Data Lake U-SQL

Lorsque vous développez un script U-SQL, il est courant de l’exécuter et de le tester localement avant de l’envoyer dans le cloud. Azure Data Lake fournit un package NuGet appelé Kit de développement logiciel (SDK) Azure Data Lake U-SQL pour ce scénario, par le biais duquel vous pouvez facilement mettre à l’échelle l’exécution et les tests U-SQL. Il est également possible d’intégrer ce test U-SQL avec le système CI (intégration continue) pour automatiser la compilation et les tests.

Si vous vous demandez comment exécuter et déboguer un script U-SQL localement et manuellement avec les outils de GUI, vous pouvez pour cela utiliser Azure Data Lake Tools pour Visual Studio. Pour plus d’informations, cliquez [ici](data-lake-analytics-data-lake-tools-local-run.md).

## <a name="install-azure-data-lake-u-sql-sdk"></a>Installer le Kit SDK Azure Data Lake U-SQL

Vous pouvez obtenir le Kit SDK Azure Data Lake U-SQL [ici](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/) sur Nuget.org. Avant de l’utiliser, vous devez vérifier que vous disposez des dépendances suivantes.

### <a name="dependencies"></a>Les dépendances

Le kit SDK U-SQL Data Lake requiert les dépendances suivantes :

- [Microsoft .NET Framework 4.6 ou une version ultérieure](https://www.microsoft.com/download/details.aspx?id=17851).
- Microsoft Visual C++ 14 et le Kit SDK Windows 10.0.10240.0 ou une version plus récente (appelé CppSDK dans cet article). Il existe deux façons d’obtenir CppSDK :

  - Installez [Visual Studio Community Edition](https://developer.microsoft.com/downloads/vs-thankyou). Vous devez disposer d’un sous-dossier \Windows Kits\10 dans le dossier Program Files, par exemple C:\Program Files (x86)\Windows Kits\10\. Vous trouverez également la version du SDK Windows 10 sous \Windows Kits\10\Lib. Si vous ne voyez pas ces dossiers, réinstallez Visual Studio et veillez à sélectionner le kit SDK Windows 10 lors de l’installation. S’il est installé avec Visual Studio, le compilateur local U-SQL le trouvera automatiquement.

    ![Data Lake Tools pour Visual Studio exécution locale Windows 10 SDK](./media/data-lake-analytics-data-lake-tools-local-run/data-lake-tools-for-visual-studio-local-run-windows-10-sdk.png)

  - Installez [Data Lake Tools for Visual Studio](https://aka.ms/adltoolsvs). Vous pouvez trouver les fichiers préconfigurés du SDK Windows et Visual C++ à l’adresse `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\Extensions\Microsoft\ADL Tools\X.X.XXXX.X\CppSDK.`

    Dans ce cas, le compilateur U-SQL local ne trouve pas ces dépendances automatiquement. Vous devez spécifier le chemin d’accès CppSDK pour lui. Vous pouvez copier les fichiers vers un autre emplacement ou les utiliser tels quels.

## <a name="understand-basic-concepts"></a>Comprendre les concepts de base

### <a name="data-root"></a>Racine de données

Ce dossier est un « magasin local » pour le compte de calcul local. Il est équivalent au compte Azure Data Lake Store d’un compte Data Lake Analytics dans Azure. Basculer vers un dossier racine de données différent revient à basculer vers un autre compte de magasin. Si vous souhaitez accéder aux données généralement partagées avec des dossiers racines de données différents, vous devez utiliser des chemins d’accès absolus dans vos scripts. Vous pouvez aussi créer des liens symboliques de système de fichiers (par exemple, **mklink** sur NTFS) sous le dossier racine de données, afin de pointer vers les données partagées.

Le dossier racine de données est utilisé pour :

- Stocker les métadonnées locales, notamment des bases de données, des tables, des fonctions table (TVF) et des assemblys.
- Rechercher les chemins d’entrée et de sortie qui sont définis comme des chemins relatifs dans U-SQL. Les chemins relatifs facilitent le déploiement de vos projets U-SQL dans Azure.

### <a name="file-path-in-u-sql"></a>Chemin d’accès des fichiers dans U-SQL

Vous pouvez utiliser un chemin d’accès relatif et un chemin d’accès absolu local dans des scripts SQL-U. Le chemin d’accès relatif dépend du chemin d’accès au dossier racine de données spécifié. Il est recommandé d’utiliser « / » en tant que séparateur de chemin afin de rendre vos scripts compatibles avec le côté serveur. Voici quelques exemples de chemins relatifs, avec leurs chemins absolus équivalents. Dans ces exemples, le chemin C:\LocalRunDataRoot correspond à la racine de données.

|Chemin relatif|Chemin absolu|
|-------------|-------------|
|/abc/def/input.csv |C:\LocalRunDataRoot\abc\def\input.csv|
|abc/def/input.csv  |C:\LocalRunDataRoot\abc\def\input.csv|
|D:/abc/def/input.csv |D:\abc\def\input.csv|

### <a name="working-directory"></a>Répertoire de travail

Lors d’une exécution locale du script U-SQL, un répertoire de travail est créé pendant la compilation sous le répertoire d’exécution actif. Outre les sorties de compilation, les fichiers exécutables nécessaires à l’exécution locale seront copiés sous forme de clichés instantanés dans ce répertoire de travail. Le dossier racine du répertoire de travail se nomme « ScopeWorkDir » et les fichiers sous le répertoire de travail sont les suivants :

|Répertoire/fichier|Répertoire/fichier|Répertoire/fichier|Définition|Description|
|--------------|--------------|--------------|----------|-----------|
|C6A101DDCB470506| | |Chaîne de hachage de la version exécutable|Cliché instantané des fichiers exécutables nécessaires à l'exécution locale|
| |Script_66AE4909AA0ED06C| |Nom de script + chaîne de hachage du chemin du script|Résultats de la compilation et journalisation des étapes d'exécution|
| | |\_script\_.abr|Sortie du compilateur|Fichier algèbre|
| | |\_ScopeCodeGen\_.*|Sortie du compilateur|Code géré généré|
| | |\_ScopeCodeGenEngine\_.*|Sortie du compilateur|Code natif généré|
| | |referenced assemblies|Référence d’assembly|Fichiers d’assemblys de référence|
| | |deployed_resources|Déploiement de ressources|Fichiers de déploiement de ressources|
| | |xxxxxxxx.xxx[1..n]\_\*.\*|Journal d’exécution|Journal des étapes d'exécution|

## <a name="use-the-sdk-from-the-command-line"></a>Utiliser le Kit de développement logiciel (SDK) depuis la ligne de commande

### <a name="command-line-interface-of-the-helper-application"></a>Interface de ligne de commande de l’application d’assistance

Sous SDK directory\build\runtime, le fichier LocalRunHelper.exe correspond à l’application d’assistance de ligne de commande qui fournit des interfaces pour la plupart des fonctions d’exécution locale couramment utilisées. Notez que la commande et les commutateurs d’arguments respectent la casse. Pour l’appeler :

```console
LocalRunHelper.exe <command> <Required-Command-Arguments> [Optional-Command-Arguments]
```

Exécutez le fichier LocalRunHelper.exe sans argument ou avec le commutateur **help** pour afficher les informations d’aide :

```console
> LocalRunHelper.exe help
    Command 'help' :  Show usage information
    Command 'compile' :  Compile the script
    Required Arguments :
        -Script param
                Script File Path
    Optional Arguments :
        -Shallow [default value 'False']
                Shallow compile
```

Dans les informations d’aide :

- Le paramètre **Commande** fournit le nom de la commande.  
- Le paramètre **Argument obligatoire** répertorie les arguments qui doivent être fournis.  
- Le paramètre **Argument facultatif** répertorie les arguments facultatifs, avec des valeurs par défaut.  Les arguments booléens facultatifs n’incluent aucun paramètre, et leur valeur est négative par défaut.

### <a name="return-value-and-logging"></a>Valeur de retour et journalisation

L’application d’assistance renvoie la valeur **0** (réussite) et **-1** (échec). Par défaut, l’assistant envoie tous les messages vers la console active. Toutefois, la plupart des commandes prennent en charge l’argument facultatif **-MessageOut path_to_log_file** qui redirige les sorties vers un fichier journal.

### <a name="environment-variable-configuring"></a>Configuration de la variable d’environnement

L’exécution U-SQL locale requiert une racine de données spécifiée comme compte de stockage local, ainsi qu’un chemin d’accès CppSDK spécifié pour les dépendances. Vous pouvez définir l’argument en ligne de commande ou définir leur variable d’environnement.

- Définissez la variable d’environnement **SCOPE_CPP_SDK**.

  Si vous obtenez Microsoft Visual C++ et le kit SDK Windows en installant Data Lake Tools pour Visual Studio, vérifiez que vous disposez du dossier suivant :

    `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\Extensions\Microsoft\Microsoft Azure Data Lake Tools for Visual Studio 2015\X.X.XXXX.X\CppSDK`

  Définissez une nouvelle variable d’environnement appelée **SCOPE_CPP_SDK**, qui pointe vers ce répertoire. Vous pouvez aussi copier le dossier vers un autre emplacement et spécifiez **SCOPE_CPP_SDK**.

  Outre la définition de la variable d’environnement, vous pouvez également spécifier l’argument **-CppSDK** lorsque vous utilisez la ligne de commande. Cet argument remplace votre variable d’environnement CppSDK par défaut.

- Définissez la variable d’environnement **LOCALRUN_DATAROOT**.

  Définissez une variable d’environnement appelée **LOCALRUN_DATAROOT**, qui pointe vers la racine de données.

  Outre la définition de la variable d’environnement, vous pouvez spécifier l’argument **-DataRoot** avec le chemin vers la racine de données lorsque vous utilisez une ligne de commande. Cet argument remplace votre variable d’environnement de racine de données par défaut. Vous devez ajouter cet argument à chaque ligne de commande que vous exécutez, afin de pouvoir remplacer la variable d’environnement de racine de données par défaut, pour toutes les opérations.

### <a name="sdk-command-line-usage-samples"></a>Exemples d’utilisation en ligne de commande du Kit SDK

#### <a name="compile-and-run"></a>Compilation et exécution

La commande **run** sert à compiler le script, puis à exécuter les résultats compilés. Ses arguments de ligne de commande sont une combinaison des commandes **compile** et **execute**.

```console
LocalRunHelper run -Script path_to_usql_script.usql [optional_arguments]
```

Voici les arguments facultatifs de **run** :

|Argument|Valeur par défaut|Description|
|--------|-------------|-----------|
|-CodeBehind|False|Le script a du code-behind .cs|
|-CppSDK| |Répertoire CppSDK|
|-DataRoot| Variable d’environnement DataRoot|DataRoot pour l’exécution locale, par défaut la variable d’environnement « LOCALRUN_DATAROOT »|
|-MessageOut| |Vidage des messages de la console dans un fichier|
|-Parallel|1|Exécuter le plan avec le parallélisme spécifié|
|-References| |Liste des chemins d’accès vers des assemblys de référence supplémentaires ou des fichiers de données de code-behind, séparés par ';'|
|-UdoRedirect|False|Générer la configuration de redirection d’assembly Udo|
|-UseDatabase|master|Base de données à utiliser pour l’inscription temporaire des assemblys et du code-behind|
|-Verbose|False|Afficher les sorties détaillées du runtime|
|-WorkDir|Répertoire actif|Répertoire des sorties et de l’utilisation du compilateur|
|-RunScopeCEP|0|Mode ScopeCEP à utiliser|
|-ScopeCEPTempPath|temp|Chemin d’accès temporaire à utiliser pour la diffusion en continu de données|
|-OptFlags| |Liste séparée par des virgules d’indicateurs d’optimiseur|

Voici un exemple :

`LocalRunHelper run -Script d:\test\test1.usql -WorkDir d:\test\bin -CodeBehind -References "d:\asm\ref1.dll;d:\asm\ref2.dll" -UseDatabase testDB –Parallel 5 -Verbose`

En plus de combiner les commandes **compile** et **execute**, vous pouvez compiler et exécuter séparément les fichiers exécutables compilés.

#### <a name="compile-a-u-sql-script"></a>Compiler un script U-SQL

La commande **compile** permet de compiler un script U-SQL vers des exécutables.

```console
LocalRunHelper compile -Script path_to_usql_script.usql [optional_arguments]
```

Voici les arguments facultatifs de **compile** :

|Argument|Description|
|--------|-----------|
| -CodeBehind [valeur par défaut 'False']|Le script a du code-behind .cs|
| -CppSDK [valeur par défaut '']|Répertoire CppSDK|
| -DataRoot [valeur par défaut ’DataRoot environment variable’]|DataRoot pour l’exécution locale, par défaut la variable d’environnement « LOCALRUN_DATAROOT »|
| -MessageOut [valeur par défaut '']|Vidage des messages de la console dans un fichier|
| -References [valeur par défaut '']|Liste des chemins d’accès vers des assemblys de référence supplémentaires ou des fichiers de données de code-behind, séparés par ';'|
| -Shallow [valeur par défaut 'False']|Compilation superficielle|
| -UdoRedirect [valeur par défaut 'False']|Générer la configuration de redirection d’assembly Udo|
| -UseDatabase [valeur par défaut 'master']|Base de données à utiliser pour l’inscription temporaire des assemblys et du code-behind|
| -WorkDir [valeur par défaut ’Current Directory’]|Répertoire des sorties et de l’utilisation du compilateur|
| -RunScopeCEP [valeur par défaut ’0’]|Mode ScopeCEP à utiliser|
| -ScopeCEPTempPath [valeur par défaut ’temp’]|Chemin d’accès temporaire à utiliser pour la diffusion en continu de données|
| -OptFlags [valeur par défaut ’’]|Liste séparée par des virgules d’indicateurs d’optimiseur|

Voici quelques exemples d’utilisation.

Compilez un script U-SQL :

```console
LocalRunHelper compile -Script d:\test\test1.usql
```

Compilez un script SQL-U et définissez le dossier racine de données. Notez que cette opération remplace la variable d’environnement définie.

```console
LocalRunHelper compile -Script d:\test\test1.usql –DataRoot c:\DataRoot
```

Compilez un script U-SQL et définissez un répertoire de travail, un assembly de référence et une base de données :

```console
LocalRunHelper compile -Script d:\test\test1.usql -WorkDir d:\test\bin -References "d:\asm\ref1.dll;d:\asm\ref2.dll" -UseDatabase testDB
```

#### <a name="execute-compiled-results"></a>Exécuter les résultats compilés

La commande **execute** est utilisée pour exécuter les résultats compilés.

```console
LocalRunHelper execute -Algebra path_to_compiled_algebra_file [optional_arguments]
```

Voici les arguments facultatifs **d’execute** :

|Argument|Valeur par défaut|Description|
|--------|-------------|-----------|
|-DataRoot | « |Racine de données pour l’exécution des métadonnées. La valeur par défaut correspond à la variable d’environnement **LOCALRUN_DATAROOT**.|
|-MessageOut | « |Videz les messages de la console dans un fichier.|
|-Parallel | « 1 » |Indicateur de lancement des étapes d’exécution locale générées avec le niveau de parallélisme spécifié.|
|-Verbose | « False » |Indicateur affichant les sorties détaillées du runtime.|

Voici un exemple d’utilisation :

```console
LocalRunHelper execute -Algebra d:\test\workdir\C6A101DDCB470506\Script_66AE4909AA0ED06C\__script__.abr –DataRoot c:\DataRoot –Parallel 5
```

## <a name="use-the-sdk-with-programming-interfaces"></a>Utiliser le kit SDK avec des interfaces de programmation

Les interfaces de programmation se trouvent toutes dans LocalRunHelper.exe. Vous pouvez les utiliser pour intégrer les fonctionnalités du kit SDK U-SQL, et l’infrastructure de test C# pour mettre à l’échelle votre test local du script SQL-U. Dans cet article, je vais utiliser le projet de test unitaire C# standard pour montrer comment utiliser ces interfaces afin de tester un script U-SQL.

### <a name="step-1-create-c-unit-test-project-and-configuration"></a>Étape 1 : Créer une configuration et un projet de test unitaire C#

- Créez un projet de test unitaire C# dans Fichier > Nouveau > Projet > Visual C# > Test > Projet de test unitaire.
- Ajoutez LocalRunHelper.exe comme référence du projet. LocalRunHelper.exe se trouve dans \build\runtime\LocalRunHelper.exe, dans le package NuGet.

   ![Kit SDK Azure Data Lake U-SQL - Ajouter une référence](./media/data-lake-analytics-u-sql-sdk/data-lake-analytics-u-sql-sdk-add-reference.png)

- Le Kit SDK U-SQL prend en charge **uniquement** l’environnement x64 ; veillez donc à définir la plateforme cible de build sur x64. Vous pouvez le faire dans Propriété du projet > Build > Plateforme cible.

   ![Kit SDK Azure Data Lake U-SQL - Configurer un projet x64](./media/data-lake-analytics-u-sql-sdk/data-lake-analytics-u-sql-sdk-configure-x64.png)

- Veillez à définir votre environnement de test sur x64. Dans Visual Studio, vous pouvez le faire dans Test > Paramètres de test > Architecture de processeur par défaut > x64.

   ![Kit SDK Azure Data Lake U-SQL - Configurer un environnement de test x64](./media/data-lake-analytics-u-sql-sdk/data-lake-analytics-u-sql-sdk-configure-test-x64.png)

- Veillez à copier tous les fichiers de dépendance sous NugetPackage\build\runtime\ dans le répertoire de travail qui se trouve généralement sous ProjectFolder\bin\x64\Debug.

### <a name="step-2-create-u-sql-script-test-case"></a>Étape 2 : Créer des cas de test du script U-SQL

Voici l’exemple de code de test du script U-SQL. Pour le test, vous devez préparer les scripts, les fichiers d’entrée et les fichiers sortie attendus.

```usql
using System;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System.IO;
using System.Text;
using System.Security.Cryptography;
using Microsoft.Analytics.LocalRun;
namespace UnitTestProject1
{
    [TestClass]
    public class USQLUnitTest
    {
        [TestMethod]
        public void TestUSQLScript()
        {
            //Specify the local run message output path
            StreamWriter MessageOutput = new StreamWriter("../../../log.txt");
            LocalRunHelper localrun = new LocalRunHelper(MessageOutput);
            //Configure the DateRoot path, Script Path and CPPSDK path
            localrun.DataRoot = "../../../";
            localrun.ScriptPath = "../../../Script/Script.usql";
            localrun.CppSdkDir = "../../../CppSDK";
            //Run U-SQL script
            localrun.DoRun();
            //Script output
            string Result = Path.Combine(localrun.DataRoot, "Output/result.csv");
            //Expected script output
            string ExpectedResult = "../../../ExpectedOutput/result.csv";
            Test.Helpers.FileAssert.AreEqual(Result, ExpectedResult);
            //Don't forget to close MessageOutput to get logs into file
            MessageOutput.Close();
        }
    }
}
namespace Test.Helpers
{
    public static class FileAssert
    {
        static string GetFileHash(string filename)
        {
            Assert.IsTrue(File.Exists(filename));
            using (var hash = new SHA1Managed())
            {
                var clearBytes = File.ReadAllBytes(filename);
                var hashedBytes = hash.ComputeHash(clearBytes);
                return ConvertBytesToHex(hashedBytes);
            }
        }
        static string ConvertBytesToHex(byte[] bytes)
        {
            var sb = new StringBuilder();
            for (var i = 0; i < bytes.Length; i++)
            {
                sb.Append(bytes[i].ToString("x"));
            }
            return sb.ToString();
        }
        public static void AreEqual(string filename1, string filename2)
        {
            string hash1 = GetFileHash(filename1);
            string hash2 = GetFileHash(filename2);
            Assert.AreEqual(hash1, hash2);
        }
    }
}
```

### <a name="programming-interfaces-in-localrunhelperexe"></a>Interfaces de programmation dans LocalRunHelper.exe

LocalRunHelper.exe fournit les interfaces de programmation pour la compilation, l’exécution, etc. U-SQL locales. Les interfaces sont les suivantes :

### <a name="constructor"></a>Constructeur

public LocalRunHelper([System.IO.TextWriter messageOutput = null])

|Paramètre|Type|Description|
|---------|----|-----------|
|messageOutput|System.IO.TextWriter|pour les messages de sortie, donner la valeur null pour utiliser la Console|

### <a name="properties"></a>Propriétés

|Propriété|Type|Description|
|--------|----|-----------|
|AlgebraPath|string|Chemin d’accès du fichier d’algèbre (le fichier d’algèbre est un des résultats de la compilation)|
|CodeBehindReferences|string|Si le script a des références code-behind supplémentaires, spécifiez les chemins d’accès séparés par « ; »|
|CppSdkDir|string|Répertoire CppSDK|
|CurrentDir|string|Répertoire actif|
|DataRoot|string|Chemin d’accès de la racine de données|
|DebuggerMailPath|string|Chemin d’accès du port d’insertion/éjection du débogueur|
|GenerateUdoRedirect|bool|Si vous souhaitez générer une configuration de remplacement de redirection de chargement d’assembly|
|HasCodeBehind|bool|Si le script a du code-behind|
|InputDir|string|Répertoire des données d’entrée|
|MessagePath|string|Chemin d’accès du fichier de vidage des messages|
|OutputDir|string|Répertoire des données de sortie|
|Parallélisme|int|Parallélisme pour exécuter l’algèbre|
|ParentPid|int|PID du parent que le service surveille pour quitter, donner la valeur 0 ou une valeur négative pour ignorer|
|ResultPath|string|Chemin d’accès du fichier de vidage des résultats|
|RuntimeDir|string|Répertoire du runtime|
|ScriptPath|string|Emplacement du script|
|Shallow|bool|Compilation superficielle ou non|
|TempDir|string|Répertoire Temp|
|UseDataBase|string|Spécifiez la base de données à utiliser pour l’inscription des assemblys temporaires du code-behind, MASTER par défaut|
|WorkDir|string|Répertoire de travail favori|

### <a name="method"></a>Méthode

|Méthode|Description|Renvoie|Paramètre|
|------|-----------|------|---------|
|public bool DoCompile()|Compilation du script U-SQL|True en cas de réussite| |
|public bool DoExec()|Exécution du résultat compilé|True en cas de réussite| |
|public bool DoRun()|Lancement du script U-SQL (compile + execute)|True en cas de réussite| |
|public bool IsValidRuntimeDir(string chemin-d’accès)|Vérification de la validité du chemin d’accès du runtime fourni|True en cas de validité|Chemin d’accès du répertoire du runtime|

## <a name="faq-about-common-issue"></a>FAQ sur les problèmes courants

### <a name="error-1"></a>Erreur 1

E_CSC_SYSTEM_INTERNAL: Erreur interne ! Impossible de charger le fichier ou l’assembly « ScopeEngineManaged.dll » ou une de ses dépendances. Le module spécifié est introuvable.

Vérifiez les éléments suivants :

- Assurez-vous que vous avez un environnement x64. La plateforme cible de build et de l’environnement de test doit être x64. Consultez la section **Étape 1 : Créer une configuration et un projet de test unitaire C#** ci-dessus.
- Vérifiez que vous avez copié tous les fichiers de dépendance sous NugetPackage\build\runtime\ dans le répertoire de travail du projet.

## <a name="next-steps"></a>Étapes suivantes

- Pour connaître U-SQL, voir [Prise en main du langage U-SQL d’Analytique Data Lake Azure](data-lake-analytics-u-sql-get-started.md).
- Pour consigner les informations de diagnostic, voir [Accès aux journaux de diagnostic d’Azure Data Lake Analytics](data-lake-analytics-diagnostic-logs.md).
- Pour consulter une requête plus complexe, voir [Analyse de journaux d’activité des sites web à l’aide d’Azure Data Lake Analytics](data-lake-analytics-analyze-weblogs.md).
- Pour afficher les détails d’un travail, voir [Utilisation de l’Explorateur de travaux et de la Vue des travaux pour les travaux Azure Data Lake Analytics](data-lake-analytics-data-lake-tools-view-jobs.md).
- Pour utiliser la vue d’exécution du vertex, voir [Utilisation de la vue d’exécution du vertex dans Data Lake Tools pour Visual Studio](data-lake-analytics-data-lake-tools-use-vertex-execution-view.md).
