---
title: Développer un projet de base de données U-SQL - Azure Data Lake
description: Découvrez comment développer une base de données U-SQL avec Azure Data Lake Tools pour Visual Studio.
author: liudan66
ms.author: liud
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 07/03/2018
ms.openlocfilehash: 706457a602e20dd37e64e5f389948b351ac8ebcf
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015245"
---
# <a name="use-a-u-sql-database-project-to-develop-a-u-sql-database-for-azure-data-lake"></a>Utiliser un projet de base de données U-SQL afin de développer une base de données U-SQL pour Azure Data Lake

Une base de données U-SQL fournit des vues structurées de données non structurées, et des données structurées gérées sous forme de tables. Elle fournit également un système de catalogue de métadonnées générales pour organiser vos données structurées et votre code personnalisé. La base de données est le concept qui regroupe ces objets associés.

En savoir plus sur la [base de données U-SQL et sur le langage de définition de données (DDL)](/u-sql/data-definition-language-ddl-statements).

Un projet de base de données U-SQL est un type de projet dans Visual Studio qui permet aux développeurs de développer, de gérer et de déployer rapidement et facilement leurs bases de données U-SQL.

## <a name="create-a-u-sql-database-project"></a>Créer un projet de base de données U-SQL

Après la version 2.3.3000.0, Azure Data Lake Tools pour Visual Studio a ajouté un nouveau modèle de projet appelé projet de base de données U-SQL. Pour créer un projet U-SQL, sélectionnez **Fichier > Nouveau > Projet**. Le projet de base de données U-SQL est accessible sous **Azure Data Lake > Nœud U-SQL**.

![Data Lake Tools pour Visual Studio - Créer un projet de base de données U-SQL](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-create-usql-database-project-creation.png)

## <a name="develop-u-sql-database-objects-by-using-a-database-project"></a>Développer des objets de base de données U-SQL à l’aide d’un projet de base de données

Cliquez avec le bouton droit sur le projet de base de données U-SQL. Sélectionnez **Ajouter > Nouvel élément**. Vous trouverez tous les nouveaux types d’objets pris en charge dans l’Assistant **Ajouter un nouvel élément**.

Pour un objet non assembly (par exemple, une fonction table), un nouveau script U-SQL est créé après l’ajout d’un nouvel élément. Vous pouvez commencer à développer l’instruction DDL pour cet objet dans l’éditeur.

Pour un objet assembly, l’outil fournit un éditeur d’interface utilisateur convivial qui vous permet d’inscrire l’assembly et de déployer des fichiers .dll, ainsi que d’autres fichiers supplémentaires. Les étapes ci-dessous vous montrent comment ajouter une définition d’objet assembly au projet de base de données U-SQL :

1. Ajoutez des références au projet C# qui comprend l’UDO, l’UDAG ou l’UDF pour le projet de base de données U-SQL.

   ![Data Lake Tools pour Visual Studio - Ajouter une référence](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-add-project-reference.png)

   ![Data Lake Tools pour Visual Studio - Ajouter une référence de projet de base de données U-SQL](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-add-project-reference-wizard.png)

2. En mode création d’assembly, choisissez l’assembly référencé dans le menu déroulant **Créer un assembly à partir de la référence**.

   ![Data Lake Tools pour Visual Studio - Créer un assembly à partir de la référence](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-create-assembly-from-reference.png)

3. Ajoutez des **dépendances gérées** et des **fichiers supplémentaires**, s’il en existe. Lorsque vous ajoutez des fichiers supplémentaires, l’outil utilise le chemin d’accès relatif pour garantir que les assemblys seront trouvés aussi bien sur votre ordinateur local qu’ultérieurement sur l’ordinateur de build.

@_DeployTempDirectory est une variable prédéfinie qui pointe l’outil vers le dossier de sortie de build. Dans le dossier de sortie de build, chaque assembly a un sous-dossier portant son nom. Toutes les DLL et tous les fichiers supplémentaires se trouvent dans ce sous-dossier.

## <a name="build-a-u-sql-database-project"></a>Générer un projet de base de données U-SQL

La sortie de build du projet de base de données U-SQL est un package de déploiement de base de données U-SQL dont le nom est suivi du suffixe `.usqldbpack`. Le package `.usqldbpack` est un fichier .zip où toutes les instructions DDL se trouvent dans le script U-SQL du dossier **DDL**, et tous les fichiers .dll et fichiers supplémentaires se trouvent dans le dossier **Temp**.

En savoir plus sur la [création d’un projet de base de données U-SQL avec une ligne de commande MSBuild et une tâche de génération Azure DevOps Services](data-lake-analytics-cicd-overview.md).

## <a name="deploy-a-u-sql-database"></a>Déployer une base de données U-SQL

Le package .usqldbpack peut être déployé sur un compte local ou sur un compte Azure Data Lake Analytics à l’aide de Visual Studio ou du SDK de déploiement.

### <a name="deploy-a-u-sql-database-in-visual-studio"></a>Déployer une base de données U-SQL dans Visual Studio

Vous pouvez déployer une base de données U-SQL via un projet de base de données U-SQL ou un package .usqldbpack dans Visual Studio.

#### <a name="deploy-through-a-u-sql-database-project"></a>Déployer via un projet de base de données U-SQL

1. Cliquez avec le bouton droit sur le projet de base de données U-SQL, puis sélectionnez **Déployer**.

1. Dans **l’Assistant Déployer la base de données U-SQL**, sélectionnez le **compte ADLA** dans lequel vous souhaitez déployer la base de données. Les comptes locaux et les comptes ADLA sont pris en charge.

1. La **source de base de données** est renseignée automatiquement, et pointe vers le package .usqldbpack situé dans le dossier de sortie de génération du projet.

1. Pour créer une base de données, entrez un nom dans **Nom de la base de données**. S’il existe une base de données du même nom dans le compte Azure Data Lake Analytics cible, tous les objets définis dans le projet de base de données sont créés sans que vous ayez à recréer la base de données.

1. Pour déployer la base de données U-SQL, cliquez sur **Envoyer**. Toutes les ressources (assemblys et fichiers supplémentaires) sont chargées, et une tâche U-SQL incluant toutes les instructions DDL est envoyée.

   ![Data Lake Tools pour Visual Studio - Déployer un projet de base de données U-SQL](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-deploy-usql-database-project.png)

   ![Data Lake Tools pour Visual Studio - Assistant Déployer un projet de base de données U-SQL](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-deploy-usql-database-project-wizard.png)

#### <a name="deploy-through-a-u-sql-database-deployment-package"></a>Déployer via un package de déploiement de base de données U-SQL

1. Ouvrez l’**Explorateur de serveurs**. Ensuite, développez le **compte Azure Data Lake Analytics** dans lequel vous souhaitez déployer la base de données.

1. Cliquez avec le bouton droit sur **Bases de données U-SQL**, puis choisissez **Déployer la base de données**.

1. Définissez la **source de base de données** pour le chemin de package (fichier .usqldbpack) de déploiement de base de données U-SQL.

1. Pour créer une base de données, entrez un nom dans **Nom de la base de données**. S’il existe une base de données du même nom dans le compte Azure Data Lake Analytics cible, tous les objets définis dans le projet de base de données sont créés sans que vous ayez à recréer la base de données.

   ![Data Lake Tools pour Visual Studio - Déployer un package de base de données U-SQL](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-deploy-usql-database-package.png)

   ![Data Lake Tools pour Visual Studio - Assistant Déployer un package de base de données U-SQL](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-deploy-usql-database-package-wizard.png)
  
### <a name="deploy-u-sql-database-by-using-the-sdk"></a>Déployer une base de données U-SQL à l’aide du SDK

`PackageDeploymentTool.exe` fournit les interfaces de programmation et de ligne de commande qui vous aident à déployer des bases de données U-SQL. Le SDK est inclus dans le [package Nuget du SDK U-SQL](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/), qui se trouve à l’emplacement `build/runtime/PackageDeploymentTool.exe`.

[En savoir plus sur le kit de développement logiciel (SDK) et sur la manière de configurer le pipeline CI/CD pour le déploiement de base de données U-SQL](data-lake-analytics-cicd-overview.md).

## <a name="reference-a-u-sql-database-project"></a>Référencer un projet de base de données U-SQL

Un projet U-SQL peut référencer un projet de base de données U-SQL. La référence affecte deux charges de travail :

- *Build de projet* : configurez les environnements de base de données référencés avant de générer les scripts U-SQL.
- *Exécution locale vs compte (projet local)* : les environnements de base de données référencés sont déployés dans le compte (de projet local) avant l’exécution du script U-SQL. [En savoir plus sur l’exécution locale et sur la différence entre le compte (ordinateur local) et le compte (projet local)](data-lake-analytics-data-lake-tools-local-run.md).

### <a name="how-to-add-a-u-sql-database-reference"></a>Comment ajouter une référence de base de données U-SQL

1. Cliquez avec le bouton droit sur le projet U-SQL dans **l’Explorateur de solutions**, puis choisissez **Ajouter une référence de base de données U-SQL...** .

    ![Data Lake Tools pour Visual Studio - Ajouter une référence de projet de base de données](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-add-database-project-reference.png)

2. Configurez une référence de base de données à partir d’un projet de base de données U-SQL dans la solution actuelle ou d’un fichier de package de base de données U-SQL.

3. Indiquez le nom de la base de données.

    ![Data Lake Tools pour Visual Studio ajoute un Assistant de référence de projet de base de données.](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-add-database-project-reference-wizard.png)

## <a name="next-steps"></a>Étapes suivantes

- [Comment configurer un pipeline CI/CD pour Azure Data Lake Analytics](data-lake-analytics-cicd-overview.md)
- [Comment tester votre code Azure Data Lake Analytics](data-lake-analytics-cicd-test.md)
- [Exécuter un script U-SQL sur votre ordinateur local](data-lake-analytics-data-lake-tools-local-run.md)
