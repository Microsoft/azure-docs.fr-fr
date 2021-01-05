---
title: Déboguer du code C# pour les travaux U-SQL Azure Data Lake
description: Cet article décrit comment déboguer un échec du vertex U-SQL à l’aide d’Azure Data Lake Tools pour Visual Studio.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 11/30/2017
ms.openlocfilehash: db1d57e3904087bc5cb3711b23cfe6bcf18c3455
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92218015"
---
# <a name="debug-user-defined-c-code-for-failed-u-sql-jobs"></a>Débogage de code C# défini par l’utilisateur pour des travaux U-SQL ayant échoué

U-SQL fournit un modèle d’extensibilité à l’aide de C#. Dans les scripts U-SQL, il est facile d’appeler des fonctions C# pour accomplir des fonctions d’analyse qu’un langage déclaratif apparenté à SQL ne prend pas en charge. Pour en savoir plus sur l’extensibilité U-SQL, consultez le [Guide de programmabilité U-SQL](./data-lake-analytics-u-sql-programmability-guide.md#use-user-defined-functions-udf). 

Dans la pratique, un code peut nécessiter un débogage, mais il est difficile de déboguer une tâche distribuée par le biais d’un code personnalisé sur le cloud avec des fichiers journaux limités. [Azure Data Lake Tools pour Visual Studio](https://aka.ms/adltoolsvs) fournit une fonctionnalité appelée **Débogage d’échec du vertex** qui facilite le débogage des défaillances se produisant dans votre code personnalisé. En cas d’échec d’une tâche U-SQL, le service conserve l’état d’échec, et l’outil permet de télécharger l’environnement de défaillance du cloud sur l’ordinateur local en vue de l’opération de débogage. Le téléchargement local capture l’environnement cloud en entier, y compris les données d’entrée et le code utilisateur.

La vidéo suivante montre la fonctionnalité Débogage d'échec du vertex dans Azure Data Lake Tools pour Visual Studio.

> [!VIDEO https://www.youtube.com/embed/3enkNvprfm4]
>

> [!IMPORTANT]
> Visual Studio a besoin des deux mises à jour suivantes pour utiliser cette fonctionnalité : [Microsoft Visual C++ 2015 Redistributable Update 3](https://www.microsoft.com/en-us/download/details.aspx?id=53840) et [Universal C Runtime pour Windows](https://www.microsoft.com/download/details.aspx?id=50410).
>

## <a name="download-failed-vertex-to-local-machine"></a>Télécharger le vertex ayant échoué sur l’ordinateur local

Lorsque vous ouvrez un travail ayant échoué dans Azure Data Lake Tools pour Visual Studio, vous voyez une barre d’alerte jaune avec des messages d’erreur détaillés sous l’onglet d’erreur.

1. Cliquez sur **Télécharger** pour télécharger toutes les ressources et les flux d’entrée requis. Si le téléchargement ne s’achève pas, cliquez sur **Réessayer**.

2. Une fois le téléchargement terminé, cliquez sur **Ouvrir** pour générer un environnement de débogage local. Une nouvelle solution de débogage va s’ouvrir. Si vous avez une solution existante ouverte dans Visual Studio, veillez à l’enregistrer et à la fermer avant d’entamer le débogage.

![Télécharger le vertex Visual Studio pour le débogage U-SQL Azure Data Lake Analytics](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-download-vertex.png)

## <a name="configure-the-debugging-environment"></a>Configurer l’environnement de débogage

> [!NOTE]
> Avant de déboguer, vérifiez que vous avez activé **Exceptions Common Language Runtime** dans la fenêtre Paramètres d'exception (**Ctrl+Alt+E**).

![Configuration de Visual Studio pour le débogage U-SQL Azure Data Lake Analytics](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-clr-exception-setting.png)

Dans la nouvelle instance Visual Studio lancée, vous pouvez trouver, ou ne pas trouver, le code source C# défini par l’utilisateur :

1. [Je trouve mon code source dans la solution](#source-code-is-included-in-debugging-solution)

2. [Je ne trouve pas mon code source dans la solution](#source-code-is-not-included-in-debugging-solution)

### <a name="source-code-is-included-in-debugging-solution"></a>Le code source est inclus dans la solution de débogage

Il existe deux cas pour lesquels le code source C# est capturé :

1. Le code utilisateur est défini dans le fichier code-behind (généralement nommé `Script.usql.cs` dans un projet U-SQL).

2. Le code utilisateur est défini dans un projet de bibliothèque de classes C# pour l’application U-SQL, et inscrit en tant qu’assembly dans les **informations de débogage**.

Si le code source est importé dans la solution, vous pouvez vous servir des outils de débogage de Visual Studio (espion, variables, etc.) pour résoudre le problème :

1. Appuyez sur **F5** pour démarrer le débogage. Le code s’exécute jusqu’à ce qu’il soit arrêté par une exception.

2. Ouvrez le fichier de code source et définissez des points d’arrêt, puis appuyez sur **F5** pour déboguer le code pas à pas.

    ![Exception de débogage U-SQL Azure Data Lake Analytics](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-debug-exception.png)

### <a name="source-code-is-not-included-in-debugging-solution"></a>Le code source n’est pas inclus dans la solution de débogage

Si le code utilisateur n’est pas inclus dans le fichier code-behind, ou si vous n’avez pas inscrit l’assembly dans les **informations de débogage**, le code source n’est pas inclus automatiquement dans la solution de débogage. Dans ce cas, vous avez besoin d’étapes supplémentaires pour ajouter votre code source :

1. Cliquez avec le bouton droit sur **Solution « VertexDebug » > Ajouter > Projet existant...** pour rechercher le code source de l’assembly et ajouter le projet à la solution de débogage.

    ![Projet d’ajout de débogage U-SQL Azure Data Lake Analytics](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-add-project-to-debug-solution.png)

2. Récupérez le chemin du dossier du projet pour le projet **FailedVertexDebugHost**. 

3. Cliquez avec le bouton droit sur le **projet du code source de l’assembly ajouté > Propriétés**, sélectionnez l’onglet **Générer** à gauche, puis collez le chemin copié finissant par \bin\debug en tant que **Sortie > Chemin de sortie**. Le chemin de sortie final ressemble à `<DataLakeTemp path>\fd91dd21-776e-4729-a78b-81ad85a4fba6\loiu0t1y.mfo\FailedVertexDebug\FailedVertexDebugHost\bin\Debug\`.

    ![Chemin d’accès de pdb de définition de débogage U-SQL Azure Data Lake Analytics](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-set-pdb-path.png)

Après la définition de ces paramètres, démarrez le débogage avec **F5** et les points d’arrêt. Vous pouvez également vous servir des outils de débogage de Visual Studio (espion, variables, etc.) pour résoudre le problème.

> [!NOTE]
> Régénérez le projet de code source d’assembly chaque fois vous modifiez le code, afin de générer des fichiers .pdb à jour.

## <a name="resubmit-the-job"></a>Renvoyer le travail

Après le débogage, si le projet se termine correctement, la fenêtre de sortie affiche le message suivant :

`The Program 'LocalVertexHost.exe' has exited with code 0 (0x0).`

![Débogage U-SQL Azure Data Lake Analytics réussi](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-debug-succeed.png)

Pour soumettre à nouveau la tâche ayant échoué :

1. Pour les tâches avec les solutions code-behind, copiez le code C# dans le fichier source code-behind (généralement `Script.usql.cs`).

2. Pour les tâches avec les assemblys, cliquez avec le bouton droit sur le projet du code source assembly dans la solution de débogage et inscrivez les assemblys .dll mis à jour à votre catalogue Azure Data Lake.

3. Soumettez à nouveau la tâche U-SQL.

## <a name="next-steps"></a>Étapes suivantes

- [Guide de programmabilité U-SQL](data-lake-analytics-u-sql-programmability-guide.md)
- [Développer des opérateurs U-SQL définis par l’utilisateur pour des travaux Azure Data Lake Analytics](data-lake-analytics-u-sql-develop-user-defined-operators.md)
- [Tester et déboguer des travaux U-SQL à l’aide d’une exécution locale et du Kit de développement logiciel (SDK) Azure Data Lake U-SQL](data-lake-analytics-data-lake-tools-local-run.md)
- [Guide pratique pour résoudre les problèmes liés à une tâche périodique inhabituelle](data-lake-analytics-data-lake-tools-debug-recurring-job.md)