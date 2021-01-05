---
title: Guide pratique pour tester votre code Azure Data Lake Analytics
description: Découvrez comment ajouter des cas de test pour U-SQL et un code C# étendu pour Azure Data Lake Analytics.
author: liudan66
ms.author: liud
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 08/30/2019
ms.openlocfilehash: 7310c67ef20a4134d4f613ea969c96802958bf62
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015228"
---
# <a name="test-your-azure-data-lake-analytics-code"></a>Tester votre code Azure Data Lake Analytics

Azure Data Lake fournit le langage [U- SQL](data-lake-analytics-u-sql-get-started.md). U-SQL combine le langage SQL déclaratif avec le langage C# impératif pour traiter des données quelle que soit l’échelle. Dans ce document, vous allez apprendre à créer des cas de test pour U-SQL et du code C# UDO (opérateur défini par l’utilisateur) étendu.

## <a name="test-u-sql-scripts"></a>Tester les scripts U-SQL

Le script U-SQL est compilé et optimisé pour le code exécutable, en vue d’une exécution dans Azure ou sur votre ordinateur local. Le processus de compilation et d’optimisation traite l’intégralité du script U-SQL. Vous ne pouvez pas effectuer un test unitaire traditionnel pour chaque instruction. Cependant, en utilisant le SDK de test U-SQL et le SDK d’exécution locale, vous pouvez effectuer des tests au niveau du script.

### <a name="create-test-cases-for-u-sql-script"></a>Créer des cas de test pour le script U-SQL

Azure Data Lake Tools pour Visual Studio vous permet de créer des cas de test pour le script U-SQL.

1. Cliquez avec le bouton droit sur un script U-SQL dans l’Explorateur de solutions, puis sélectionnez **Créer un test unitaire**.

1. Créez un projet de test ou insérez le cas de test dans un projet de test existant.

   ![Data Lake Tools pour Visual Studio - Créer une configuration de projet de test U-SQL](./media/data-lake-analytics-cicd-test/data-lake-tools-create-usql-test-project-configure.png)

### <a name="manage-the-test-data-source"></a>Gérer la source de données de test

Lorsque vous testez des scripts U-SQL, vous avez besoin de fichiers d’entrée de test. Pour gérer les données de test, dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet U-SQL et sélectionnez **Propriétés**. Vous pouvez entrer une source dans **Source de données de test**.

![Data Lake Tools pour Visual Studio - Configurer la source des données de test du projet](./media/data-lake-analytics-cicd-test/data-lake-tools-configure-project-test-data-source.png)

Quand vous appelez l'interface `Initialize()` dans le kit de développement logiciel (SDK) de test d’U-SQL, un dossier racine de données locales temporaire est créé sous le répertoire de travail du projet de test. Tous les fichiers et dossiers du dossier de source de données de test sont copiés dans le dossier racine de données locales temporaire avant l’exécution des cas de test de script U-SQL. Vous pouvez ajouter des dossiers de source de données de test en divisant le chemin du dossier des données de test à l’aide d’un point-virgule.

### <a name="manage-the-database-environment-for-testing"></a>Gérer l’environnement de base de données pour les tests

Si vos scripts U-SQL utilisent des objets de base de données U-SQL ou interrogent à l’aide de ceux-ci, vous devez initialiser l’environnement de base de données avant d’exécuter les cas de test U-SQL. Cette approche peut être nécessaire lors de l’appel de procédures stockées. L’interface `Initialize()` du SDK de test U-SQL vous permet de déployer toutes les bases de données qui sont référencées par le projet U-SQL dans le dossier racine des données local temporaire, lui-même situé dans le répertoire de travail du projet de test.

Pour plus d’informations sur la gestion des références de projet de base de données U-SQL pour un projet U-SQL, consultez [Référencer un projet de base de données U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md#reference-a-u-sql-database-project).

### <a name="verify-test-results"></a>Vérifier les résultats du test

L’interface `Run()` retourne les résultats de l’exécution d’une tâche. *0* signifie Réussite, *1* signifie Échec. Vous pouvez également utiliser les fonctions C# assert pour vérifier les sorties.

### <a name="run-test-cases-in-visual-studio"></a>Exécuter des cas de test dans Visual Studio

Un projet de test du script U-SQL s’appuie sur le framework de test unitaire C#. Après avoir généré le projet, sélectionnez **Test** > **Windows** > **Explorateur de tests**. Vous pouvez exécuter des cas de test à partir de **l’Explorateur de tests**. Vous pouvez également cliquer sur le fichier .cs dans votre test unitaire, puis sélectionner **Exécuter les tests**.

## <a name="test-c-udos"></a>Tester les opérateurs définis par l’utilisateur (UDO) C#

### <a name="create-test-cases-for-c-udos"></a>Créer des cas de test pour les UDO C#

Vous pouvez utiliser le framework de test unitaire C# pour tester vos opérateurs définis par l’utilisateur C#. Lorsque vous testez les opérateurs définis par l’utilisateur, vous devez préparer les objets **IRowset** correspondants comme des entrées.

Il existe deux façons de créer un objet **IRowset** :

- Charger des données à partir d’un fichier pour créer **IRowset** :

    ```csharp
    //Schema: "a:int, b:int"
    USqlColumn<int> col1 = new USqlColumn<int>("a");
    USqlColumn<int> col2 = new USqlColumn<int>("b");
    List<IColumn> columns = new List<IColumn> { col1, col2 };
    USqlSchema schema = new USqlSchema(columns);

    //Generate one row with default values
    IUpdatableRow output = new USqlRow(schema, null).AsUpdatable();

    //Get data from file
    IRowset rowset = UnitTestHelper.GetRowsetFromFile(@"processor.txt", schema, output.AsReadOnly(), discardAdditionalColumns: true, rowDelimiter: null, columnSeparator: '\t');
    ```

- Utiliser les données d’une collecte pour créer **IRowset** :

    ```csharp
    //Schema: "a:int, b:int"
    USqlSchema schema = new USqlSchema(
        new USqlColumn<int>("a"),
        new USqlColumn<int>("b")
    );

    IUpdatableRow output = new USqlRow(schema, null).AsUpdatable();

    //Generate Rowset with specified values
    List<object[]> values = new List<object[]>{
        new object[2] { 2, 3 },
        new object[2] { 10, 20 }
    };

    IEnumerable<IRow> rows = UnitTestHelper.CreateRowsFromValues(schema, values);
    IRowset rowset = UnitTestHelper.GetRowsetFromCollection(rows, output.AsReadOnly());
    ```

### <a name="verify-test-results"></a>Vérifier les résultats du test

Après l’appel des fonctions UDO, vous pouvez vérifier les résultats par l’intermédiaire du schéma et de la vérification de la valeur Rowset à l’aide des fonctions C# Assert. Vous pouvez ajouter un **projet de test unitaire C# U-SQL UDO** à votre solution. Pour ce faire, sélectionnez **Fichier > Nouveau > Projet** dans Visual Studio.

### <a name="run-test-cases-in-visual-studio"></a>Exécuter des cas de test dans Visual Studio

Après avoir généré le projet, sélectionnez **Test** > **Windows** > **Explorateur de tests**. Vous pouvez exécuter des cas de test à partir de **l’Explorateur de tests**. Vous pouvez également cliquer sur le fichier .cs dans votre test unitaire, puis sélectionner **Exécuter les tests**.

## <a name="run-test-cases-in-azure-pipelines"></a>Exécuter des cas de test dans Azure Pipelines<a name="run-test-cases-in-azure-devops"></a>

Les **projets de test du script U-SQL** et les **projets de test C# UDO** héritent des projet de test unitaire C#. La [tâche Visual Studio Test](/azure/devops/pipelines/test/getting-started-with-continuous-testing) dans Azure Pipelines peut exécuter ces cas de test.

### <a name="run-u-sql-test-cases-in-azure-pipelines"></a>Exécuter des cas de test U-SQL dans Azure Pipelines

Pour un test U-SQL, assurez-vous que vous chargez `CPPSDK` sur votre ordinateur de build , puis passez le chemin `CPPSDK` à `USqlScriptTestRunner(cppSdkFolderFullPath: @"")`.

#### <a name="what-is-cppsdk"></a>Qu’est-ce que CPPSDK ?

CPPSDK est un package qui comprend Microsoft Visual C++ 14 et Windows SDK 10.0.10240.0. Ce package contient l’environnement nécessaire pour le runtime U-SQL. Ce package est disponible sous le dossier d’installation d’Azure Data Lake Tools pour Visual Studio :

- Pour Visual Studio 2015, il se trouve sous `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\Extensions\Microsoft\Microsoft Azure Data Lake Tools for Visual Studio 2015\X.X.XXXX.X\CppSDK`
- Pour Visual Studio 2017, il se trouve sous `C:\Program Files (x86)\Microsoft Visual Studio\2017\<Visual Studio Edition>\SDK\ScopeCppSDK`
- Pour Visual Studio 2019, il se trouve sous `C:\Program Files (x86)\Microsoft Visual Studio\2019\<Visual Studio Edition>\SDK\ScopeCppSDK`

#### <a name="prepare-cppsdk-in-the-azure-pipelines-build-agent"></a>Préparer CPPSDK dans l’agent de build Azure Pipelines

La méthode la plus courante pour préparer la dépendance CPPSDK dans Azure Pipelines est la suivante :

1. Compressez le dossier qui contient les bibliothèques CPPSDK.

1. Archivez le fichier .zip dans votre système de contrôle de code source Le fichier .zip garantit que toutes les bibliothèques du dossier CPPSDK sont archivées, pour qu’aucun fichier ne soit ignoré à cause d’un fichier `.gitignore`.

1. Décompressez le fichier .zip dans le pipeline de build.

1. Pointez `USqlScriptTestRunner` sur ce dossier décompressé sur l’ordinateur de build.

### <a name="run-c-udo-test-cases-in-azure-pipelines"></a>Exécuter des cas de test C# UDO dans Azure Pipelines

Pour le test C# UDO, veillez à référencer les assemblys ci-dessous, qui sont nécessaires pour les opérateurs définis par l’utilisateur.

- Microsoft.Analytics.Interfaces
- Microsoft.Analytics.Types
- Microsoft.Analytics.UnitTest

Si vous y faites référence via [le package Nuget Microsoft.Azure.DataLake.USQL.Interfaces](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.Interfaces/), assurez-vous d’ajouter une tâche de restauration NuGet dans votre pipeline de build.

## <a name="next-steps"></a>Étapes suivantes

- [Comment configurer un pipeline CI/CD pour Azure Data Lake Analytics](data-lake-analytics-cicd-overview.md)
- [Exécuter un script U-SQL sur votre ordinateur local](data-lake-analytics-data-lake-tools-local-run.md)
- [Utiliser un projet de base de données U-SQL afin de développer une base de données U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md)