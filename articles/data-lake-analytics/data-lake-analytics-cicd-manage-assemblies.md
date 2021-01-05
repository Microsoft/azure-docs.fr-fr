---
title: Gérer les assemblys U-SQL dans un pipeline CI/CD - Azure Data Lake
description: Découvrez les bonnes pratiques pour gérer les assemblys C# U-SQL dans un pipeline CI/CD avec Azure DevOps.
author: liudan66
ms.author: liud
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 10/30/2018
ms.openlocfilehash: e88616f45c69d33234aa35333e0d82ad8cc59bb6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015279"
---
# <a name="best-practices-for-managing-u-sql-assemblies-in-a-cicd-pipeline"></a>Bonnes pratiques pour gérer les assemblys U-SQL dans un pipeline CI/CD

Dans cet article, vous allez apprendre à gérer le code source des assemblys U-SQL avec le tout nouveau projet de base de données U-SQL. Vous allez également apprendre à implémenter un pipeline d'intégration et de déploiement continus (CI/CD) pour l'inscription des assemblys à l'aide d'Azure DevOps.

## <a name="use-the-u-sql-database-project-to-manage-assembly-source-code"></a>Utiliser le projet de base de données U-SQL pour gérer le code source des assemblys

Le [projet de base de données U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md) est un type de projet dans Visual Studio qui permet aux développeurs de développer, de gérer et de déployer rapidement et facilement leurs bases de données U-SQL. Vous pouvez gérer tous les objets de base de données U-SQL (à l'exception des informations d'identification) avec le projet de base de données U-SQL.

Pour gérer le code source C# des assemblys et les scripts U-SQL DDL d'inscription des assemblys, utilisez :

- le projet de base de données U-SQL pour gérer les scripts U-SQL d'inscription des assemblys ;
- la bibliothèque de classes (pour application U-SQL) afin de gérer le code source C# et les dépendances pour les opérateurs, fonctions et agrégateurs définis par l'utilisateur (UDO, UDF et UDAG) ;
- le projet de base de données U-SQL pour référencer le projet de bibliothèque de classes.

Un projet de base de données U-SQL peut référencer un projet de bibliothèque de classes (pour application U-SQL). Vous pouvez créer les assemblys inscrits dans la base de données U-SQL à l'aide de code source C# référencé à partir de ce projet de bibliothèque de classes (pour application U-SQL).

Suivez ces étapes pour créer des projets et ajouter des références.

1. Créez un projet de bibliothèque de classes (pour application U-SQL) en sélectionnant **Fichier** > **Nouveau** > **Projet**. Le projet se trouve sous le nœud **Azure Data Lake > U-SQL**.

   ![Data Lake Tools pour Visual Studio : créer un projet de bibliothèque de classes C#](./media/data-lake-analytics-cicd-manage-assemblies/create-c-sharp-class-library-project.png)

1. Ajoutez votre code C# défini par l’utilisateur au projet de bibliothèque de classes (pour application U-SQL).

1. Créez un projet U-SQL en sélectionnant **Fichier** > **Nouveau** > **Projet**. Le projet se trouve sous le nœud **Azure Data Lake** > **U-SQL**.

   ![Data Lake Tools pour Visual Studio - Créer un projet de base de données U-SQL](media/data-lake-analytics-cicd-manage-assemblies/create-u-sql-database-project.png)

1. Ajoutez une référence au projet de bibliothèque de classes C# pour le projet de base de données U-SQL.

   ![Data Lake Tools pour Visual Studio - Ajouter une référence](./media/data-lake-analytics-cicd-manage-assemblies/data-lake-tools-add-project-reference.png)

   ![Data Lake Tools pour Visual Studio - Ajouter une référence de projet de base de données U-SQL](./media/data-lake-analytics-cicd-manage-assemblies/data-lake-tools-add-project-reference-wizard.png)

1. Créez un script d'assembly dans le projet de base de données U-SQL en cliquant avec le bouton droit sur le projet et en sélectionnant **Ajouter un nouvel élément**.

   ![Data Lake Tools pour Visual Studio : ajouter un script d'assembly](media/data-lake-analytics-cicd-manage-assemblies/add-assembly-script.png)

1. Ouvrez le script d'assembly en mode création d'assembly. Sélectionnez l'assembly référencé dans le menu déroulant **Créer un assembly à partir de la référence**.

   ![Data Lake Tools pour Visual Studio - Créer un assembly à partir de la référence](./media/data-lake-analytics-cicd-manage-assemblies/data-lake-tools-create-assembly-from-reference.png)

1. Ajoutez des **dépendances gérées** et des **fichiers supplémentaires**, le cas échéant. Lorsque vous ajoutez des fichiers supplémentaires, l'outil utilise le chemin d'accès relatif pour garantir que les assemblys seront trouvés aussi bien sur votre ordinateur local qu'ultérieurement sur l'ordinateur de build.

**\@_DeployTempDirectory** en bas de la fenêtre de l’éditeur est une variable prédéfinie qui pointe l’outil vers le dossier de sortie de build. Dans le dossier de sortie de build, chaque assembly a un sous-dossier portant son nom. Toutes les DLL et tous les fichiers supplémentaires se trouvent dans ce sous-dossier.

## <a name="build-a-u-sql-database-project"></a>Générer un projet de base de données U-SQL

La sortie de build du projet de base de données U-SQL est un package de déploiement de base de données U-SQL. Son nom est suivi du suffixe `.usqldbpack`. Le package `.usqldbpack` est un fichier .zip qui rassemble toutes les instructions DDL dans un même script U-SQL, situé dans le dossier DDL. Tous les fichiers .dll générés ainsi que les fichiers supplémentaires des assemblys se trouvent dans le dossier Temp.

## <a name="deploy-a-u-sql-database"></a>Déployer une base de données U-SQL

Le package `.usqldbpack` peut être déployé sur un compte local ou sur un compte Azure Data Lake Analytics. Utilisez Visual Studio ou le kit de développement logiciel (SDK) de déploiement.

### <a name="deploy-a-u-sql-database-in-visual-studio"></a>Déployer une base de données U-SQL dans Visual Studio

Vous pouvez déployer une base de données U-SQL à l'aide d'un projet de base de données U-SQL ou d'un package `.usqldbpack` dans Visual Studio.

#### <a name="deploy-by-using-a-u-sql-database-project"></a>Déployer à l'aide d'un projet de base de données U-SQL

1. Cliquez avec le bouton droit sur le projet de base de données U-SQL, puis sélectionnez **Déployer**.

1. Dans l'Assistant **Déployer la base de données U-SQL**, sélectionnez le **compte ADLA** sur lequel vous souhaitez déployer la base de données. Les comptes locaux et les comptes ADLA sont pris en charge.

1. La **source de base de données** est renseignée automatiquement. Elle pointe vers le package .usqldbpack situé dans le dossier de sortie de génération du projet.

1. Pour créer une base de données, entrez un nom dans **Nom de la base de données**. S'il existe une base de données du même nom sur le compte Azure Data Lake Analytics cible, tous les objets définis dans le projet de base de données sont créés sans que vous ayez à recréer la base de données.

1. Pour déployer la base de données U-SQL, cliquez sur **Envoyer**. Toutes les ressources (comme les assemblys et les fichiers supplémentaires) sont chargées. Un travail U-SQL incluant toutes les instructions DDL est soumis.

   ![Data Lake Tools pour Visual Studio - Déployer un projet de base de données U-SQL](./media/data-lake-analytics-cicd-manage-assemblies/data-lake-tools-deploy-usql-database-project.png)

   ![Data Lake Tools pour Visual Studio - Assistant Déployer un projet de base de données U-SQL](./media/data-lake-analytics-cicd-manage-assemblies/data-lake-tools-deploy-usql-database-project-wizard.png)

### <a name="deploy-a-u-sql-database-in-azure-devops"></a>Déployer une base de données U-SQL dans Azure DevOps

`PackageDeploymentTool.exe` fournit les interfaces de programmation et de ligne de commande qui vous aident à déployer des bases de données U-SQL. Le SDK est inclus dans le [package Nuget du SDK U-SQL](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/), qui se trouve à l’emplacement `build/runtime/PackageDeploymentTool.exe`.

Dans Azure DevOps, vous pouvez utiliser une tâche de ligne de commande et ce kit de développement logiciel (SDK) afin de configurer un pipeline d'automatisation pour l'actualisation de la base de données U-SQL. [En savoir plus sur le kit de développement logiciel (SDK) et sur la configuration d'un pipeline CI/CD pour le déploiement d'une base de données U-SQL](data-lake-analytics-cicd-overview.md#deploy-u-sql-database-through-azure-pipelines).

## <a name="next-steps"></a>Étapes suivantes

- [Configurer un pipeline CI/CD pour Azure Data Lake Analytics](data-lake-analytics-cicd-overview.md)
- [Tester votre code Azure Data Lake Analytics](data-lake-analytics-cicd-test.md)
- [Exécuter un script U-SQL sur votre ordinateur local](data-lake-analytics-data-lake-tools-local-run.md)
