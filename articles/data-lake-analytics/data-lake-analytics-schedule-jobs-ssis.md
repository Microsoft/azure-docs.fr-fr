---
title: Planifier des tâches U-SQL Azure Data Lake Analytics avec SSIS
description: Découvrez comment utiliser SQL Server Integration Services (SSIS) pour planifier des tâches U-SQL avec un script inclus ou à partir de fichiers de requête U-SQL.
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 07/17/2018
ms.openlocfilehash: b080b433f5af49e970faba02003fb68e21a08365
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92221449"
---
# <a name="schedule-u-sql-jobs-using-sql-server-integration-services-ssis"></a>Planifier des tâches U-SQL à l’aide de SQL Server Integration Services (SSIS)

Dans ce document, vous apprenez à orchestrer et à créer des tâches U-SQL avec SQL Server Integration Services. 

## <a name="prerequisites"></a>Prérequis

Le [Feature Pack Azure pour Integration Services](/sql/integration-services/azure-feature-pack-for-integration-services-ssis#scenario-managing-data-in-the-cloud) fournit la [tâche Azure Data Lake Analytics](/sql/integration-services/control-flow/azure-data-lake-analytics-task), et le [Gestionnaire de connexions Azure Data Lake Analytics](/sql/integration-services/connection-manager/azure-data-lake-analytics-connection-manager) qui vous aide à vous connecter au service Azure Data Lake Analytics. Pour utiliser cette tâche, veillez à installer les éléments suivants :

- [Téléchargez et installez SQL Server Data Tools (SSDT) pour Visual Studio](/sql/ssdt/download-sql-server-data-tools-ssdt)
- [Installer le Feature Pack Azure pour Integration Services (SSIS)](/sql/integration-services/azure-feature-pack-for-integration-services-ssis)

## <a name="azure-data-lake-analytics-task"></a>Tâche Azure Data Lake Analytics

La tâche Azure Data Lake Analytics permet aux utilisateurs d’envoyer des tâches U-SQL au compte Azure Data Lake Analytics. 

[Découvrez comment configurer une tâche Azure Data Lake Analytics](/sql/integration-services/control-flow/azure-data-lake-analytics-task).

![Tâche Azure Data Lake Analytics dans SSIS](./media/data-lake-analytics-schedule-jobs-ssis/data-lake-analytics-azure-data-lake-analytics-task-in-ssis.png)

Vous pouvez obtenir le script U-SQL à partir de différents emplacements par le biais de tâches et de fonctions intégrées SSIS ; les scénarios ci-dessous montrent comment configurer les scripts U-SQL pour différents cas d’utilisateurs.

## <a name="scenario-1-use-inline-script-call-tvfs-and-stored-procs"></a>Scénario 1 : Utilisation d’un script inline pour l’appel de fonctions table et de procédures stockées

Dans l’éditeur de tâche d’Azure Data Lake Analytics, configurez **SourceType** en tant que **DirectInput** et placez les instructions U-SQL dans **USQLStatemnt**.

Pour faciliter la maintenance et la gestion du code, placez uniquement de petits scripts U-SQL comme scripts inline ; par exemple, vous pouvez appeler des fonctions table et des procédures stockées existantes dans vos bases de données U-SQL. 

![Modifier le script U-SQL inline dans une tâche SSIS](./media/data-lake-analytics-schedule-jobs-ssis/edit-inline-usql-script-in-ssis.png)

Article connexe : [Guide pratique pour transmettre un paramètre à des procédures stockées](#scenario-6-pass-parameters-to-u-sql-script)

## <a name="scenario-2-use-u-sql-files-in-azure-data-lake-store"></a>Scénario 2 : Utilisation des fichiers U-SQL dans Azure Data Lake Storage

Vous pouvez également utiliser des fichiers U-SQL dans Azure Data Lake Storage à l’aide d’une **tâche de système de fichiers Azure Data Lake Storage** dans le Feature Pack Azure. Cette approche vous permet d’utiliser les scripts stockés sur le cloud.

Suivez les étapes ci-dessous pour configurer la connexion entre une tâche de système de fichiers Azure Data Lake Storage et une tâche Azure Data Lake Analytics.

### <a name="set-task-control-flow"></a>Définir le flux de contrôle d’une tâche

En mode création du package SSIS, ajoutez une **tâche de système de fichiers Azure Data Lake Storage**, un **conteneur de boucle Foreach** et une **tâche Azure Data Lake Analytics (ADLS)** dans le conteneur de boucle Foreach. La tâche de système de fichiers Azure Data Lake Storage permet de télécharger les fichiers U-SQL dans un dossier temporaire de votre compte ADLS. Le conteneur de boucle Foreach et la tâche Azure Data Lake Analytics permettent de soumettre chaque fichier U-SQL du dossier temporaire au compte Azure Data Lake Analytics en tant que tâche U-SQL.

![Diagramme qui montre une tâche de système de fichiers Azure Data Lake Storage ajoutée à un conteneur de boucles Foreach.](./media/data-lake-analytics-schedule-jobs-ssis/use-u-sql-files-in-azure-data-lake-store.png)

### <a name="configure-azure-data-lake-store-file-system-task"></a>Configurer une tâche de système de fichiers Azure Data Lake Storage

1. Définissez **Opération** sur **CopyFromADLS**.
2. Configurez **AzureDataLakeConnection** ; pour en savoir plus, consultez [Gestionnaire de connexions Azure Data Lake Storage](/sql/integration-services/connection-manager/azure-data-lake-store-connection-manager).
3. Définissez **AzureDataLakeDirectory**. Pointez sur le dossier de stockage de vos scripts U-SQL. Utilisez le chemin relatif qui est relatif au dossier racine du compte Azure Data Lake Storage.
4. Définissez **Destination** sur un dossier qui met en cache les scripts U-SQL téléchargés. Ce chemin du dossier est utilisé dans le conteneur de boucle Foreach pour la soumission de tâches U-SQL. 

![Configurer une tâche de système de fichiers Azure Data Lake Storage](./media/data-lake-analytics-schedule-jobs-ssis/configure-azure-data-lake-store-file-system-task.png)

[En savoir plus sur une tâche de système de fichiers Azure Data Lake Storage](/sql/integration-services/control-flow/azure-data-lake-store-file-system-task).

### <a name="configure-foreach-loop-container"></a>Configurer un conteneur de boucle Foreach

1. Dans la page **Collection**, définissez **Énumérateur** sur **Énumérateur Foreach File**.

2. Définissez **Dossier** situé dans la zone **Configuration de l’énumérateur** sur le dossier temporaire qui inclut les scripts U-SQL téléchargés.

3. Définissez **Fichiers** dans la zone **Configuration de l’énumérateur** sur `*.usql`, afin que le conteneur de boucle intercepte uniquement les fichiers se terminant par `.usql`.

    ![Capture d’écran qui montre l’éditeur de boucle Foreach avec « Collection » sélectionné et les sections Énumérateur et Configuration de l’énumérateur mises en évidence.](./media/data-lake-analytics-schedule-jobs-ssis/configure-foreach-loop-container-collection.png)

4. Dans la page **Mappage de variables**, ajoutez une variable définie par l’utilisateur pour obtenir le nom de fichier pour chaque fichier U-SQL. Définissez l’**Index** sur 0 pour obtenir le nom de fichier. Dans cet exemple, définissez une variable appelée `User::FileName`. Cette variable est utilisée pour obtenir de façon dynamique la connexion des fichiers de script U-SQL et pour définir le nom de la tâche U-SQL dans la tâche Azure Data Lake Analytics.

    ![Configurer un conteneur de boucle Foreach pour obtenir le nom de fichier](./media/data-lake-analytics-schedule-jobs-ssis/configure-foreach-loop-container-variable-mapping.png)

### <a name="configure-azure-data-lake-analytics-task"></a>Configurer une tâche Azure Data Lake Analytics 

1. Définissez **SourceType** sur **FileConnection**.

2. Définissez **FileConnection** sur le fichier de connexion qui pointe vers les objets fichier retournés depuis le conteneur de boucle Foreach.
    
    Pour créer cette connexion de fichier :

   1. Choisissez **\<New Connection...>** dans le paramètre FileConnection.
   2. Définissez **Type d’utilisation** sur **Fichier existant** et définissez le **Fichier** sur un chemin de fichier du fichier existant.

       ![Capture d’écran qui montre l’éditeur du gestionnaire de connexions de fichiers avec « Fichier existant » sélectionné pour « Type d’utilisation ».](./media/data-lake-analytics-schedule-jobs-ssis/configure-file-connection-for-foreach-loop-container.png)

   3. Dans la vue **Gestionnaires de connexions**, cliquez avec le bouton droit sur la connexion de fichier créée à l’instant et choisissez **Propriétés**.

   4. Dans la fenêtre **Propriétés**, développez **Expressions**et définissez **ConnectionString** sur la variable définie dans le conteneur de boucle Foreach, par exemple, `@[User::FileName]`.

       ![Configurer un conteneur de boucle Foreach](./media/data-lake-analytics-schedule-jobs-ssis/configure-file-connection-property-for-foreach-loop-container.png)

3. Définissez **AzureDataLakeAnalyticsConnection** sur le compte Azure Data Lake Analytics auquel vous souhaitez envoyer des tâches. En savoir plus sur le [Gestionnaire de connexions Azure Data Lake Analytics](/sql/integration-services/connection-manager/azure-data-lake-analytics-connection-manager).

4. Définissez d’autres configurations de tâche. [En savoir plus](/sql/integration-services/control-flow/azure-data-lake-analytics-task)

5. Utilisez **Expressions** pour définir dynamiquement le nom de la tâche U-SQL :

    1. Dans la page **Expressions**, ajoutez une nouvelle paire clé-valeur d’expression pour **JobName**.
    2. Définissez la valeur de JobName sur la variable définie dans le conteneur de boucle Foreach, par exemple, `@[User::FileName]`.
    
        ![Configurer une expression SSIS pour le nom de la tâche U-SQL](./media/data-lake-analytics-schedule-jobs-ssis/configure-expression-for-u-sql-job-name.png)

## <a name="scenario-3-use-u-sql-files-in-azure-blob-storage"></a>Scénario 3 : Utilisation des fichiers U-SQL dans le Stockage Blob Azure

Vous pouvez utiliser les fichiers U-SQL dans le Stockage Blob Azure par le biais d’une **tâche de téléchargement d’objet blob Azure** dans le Feature Pack Azure. Cette approche vous permet d’utiliser les scripts sur le cloud.

Les étapes sont similaires à celle du [scénario 2 : Utiliser des fichiers U-SQL dans Azure Data Lake Storage](#scenario-2-use-u-sql-files-in-azure-data-lake-store). Changez la tâche de système de fichiers Azure Data Lake Storage en tâche de téléchargement d’objet blob Azure. [En savoir plus sur la tâche de téléchargement d’objet blob Azure](/sql/integration-services/control-flow/azure-blob-download-task).

Le flux de contrôle ressemble à ce qui suit.

![Utilisation des fichiers U-SQL dans Azure Data Lake Storage](./media/data-lake-analytics-schedule-jobs-ssis/use-u-sql-files-in-azure-blob-storage.png)

## <a name="scenario-4-use-u-sql-files-on-the-local-machine"></a>Scénario 4 : Utilisation des fichiers U-SQL sur la machine locale

Outre l’utilisation de fichiers U-SQL stockés dans le cloud, vous pouvez également utiliser des fichiers sur votre machine locale ou des fichiers déployés avec vos packages SSIS.

1. Avec le bouton droit, cliquez sur **Gestionnaires de connexions** dans un projet SSIS et choisissez **Nouveau gestionnaire de connexions**.

2. Sélectionnez le type **Fichier** et cliquez sur **Ajouter...** .

3. Définissez **Type d’utilisation** sur **Fichier existant** et **Fichier** sur le fichier de la machine locale.

    ![Ajouter une connexion de fichier au fichier local](./media/data-lake-analytics-schedule-jobs-ssis/configure-file-connection-for-foreach-loop-container.png)

4. Ajoutez une **tâche Azure Data Lake Analytics** et :
    1. Définissez **SourceType** sur **FileConnection**.
    2. Définissez **FileConnection** sur la connexion de fichier créée à l’instant.

5. Terminez les autres configurations pour la tâche Azure Data Lake Analytics.

## <a name="scenario-5-use-u-sql-statement-in-ssis-variable"></a>Scénario 5 : Utilisation d’une instruction U-SQL dans une variable SSIS

Dans certains cas, vous pouvez avoir besoin de générer dynamiquement les instructions U-SQL. Vous pouvez utiliser une **variable SSIS** avec une **expression SSIS** et d’autres tâches SSIS, comme une tâche de script, pour pouvoir générer l’instruction U-SQL de manière dynamique.

1. Ouvrez la fenêtre de l’outil Variables via le menu de niveau supérieur **SSIS > Variables**.

2. Ajoutez une variable SSIS et définissez la valeur directement ou utilisez **Expression** pour générer la valeur.

3. Ajoutez une **tâche Azure Data Lake Analytics** et :
    1. Définissez **SourceType** sur **Variable**.
    2. Définissez **SourceVariable** sur la variable SSIS créée à l’instant.

4. Terminez les autres configurations pour la tâche Azure Data Lake Analytics.

## <a name="scenario-6-pass-parameters-to-u-sql-script"></a>Scénario 6 : Transmission de paramètres au script U-SQL

Dans certains cas, vous pouvez souhaiter définir dynamiquement la valeur de la variable U-SQL dans le script U-SQL. La fonction **Mappage de paramètre** dans la tâche Azure Data Lake Analytics est utile dans ce scénario. Il existe généralement deux cas d’utilisation classiques :

- Définissez de façon dynamique les variables des chemins de fichier d’entrée et de sortie, en fonction de la date et de l’heure actuelles.
- Définissez le paramètre pour les procédures stockées.

[En savoir plus sur la définition des paramètres du script U-SQL](/sql/integration-services/control-flow/azure-data-lake-analytics-task#parameter-mapping-page-configuration).

## <a name="next-steps"></a>Étapes suivantes

- [Exécuter des packages SSIS dans Azure](../data-factory/how-to-invoke-ssis-package-ssis-activity.md)
- [Feature Pack Azure pour Integration Services (SSIS)](/sql/integration-services/azure-feature-pack-for-integration-services-ssis#scenario-managing-data-in-the-cloud)
- [Planifier des tâches U-SQL avec Azure Data Factory](../data-factory/transform-data-using-data-lake-analytics.md)