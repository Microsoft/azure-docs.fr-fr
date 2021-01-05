---
title: Copier des données de Stockage Blob vers SQL Database – Azure
description: Ce didacticiel vous montre comment utiliser l’activité de copie dans un pipeline Azure Data Factory pour copier des données depuis Blob Storage vers une base de données SQL Azure.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: ''
editor: ''
ms.assetid: e4035060-93bf-4e8d-bf35-35e2d15c51e0
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 15bce219b96268124729de2f475e33fc386348a8
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021212"
---
# <a name="tutorial-copy-data-from-blob-storage-to-sql-database-using-data-factory"></a>Tutoriel : Copie de données Blob Storage vers une base de données SQL à l’aide de Data Factory
> [!div class="op_single_selector"]
> * [Vue d’ensemble et composants requis](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Assistant de copie](data-factory-copy-data-wizard-tutorial.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Modèle Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [API .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez le [tutoriel sur l’activité de copie](../quickstart-create-data-factory-dot-net.md).

Dans ce didacticiel, vous allez créer une fabrique de données avec un pipeline afin de copier des données entre Blob Storage et SQL Database.

L’activité de copie effectue le déplacement des données dans Azure Data Factory. Elle est mise en œuvre par un service disponible dans le monde entier, capable de copier des données entre différents magasins de données de façon sécurisée, fiable et évolutive. Pour plus d’informations sur l’activité de copie, consultez l’article [Activités de déplacement des données](data-factory-data-movement-activities.md) .  

> [!NOTE]
> Pour obtenir une présentation détaillée du service Data Factory, consultez l’article [Présentation d’Azure Data Factory](data-factory-introduction.md) .
>
>

## <a name="prerequisites-for-the-tutorial"></a>Configuration requise pour le didacticiel
Avant de commencer ce didacticiel, vous devez disposer des éléments suivants :

* **Abonnement Azure**.  Si vous n'êtes pas abonné, vous pouvez créer un compte d'essai gratuit en quelques minutes. Consultez l'article [Essai gratuit](https://azure.microsoft.com/pricing/free-trial/) pour plus d'informations.
* **Compte Azure Storage**. Dans le cadre de ce didacticiel, le stockage d’objets blob est utilisé comme magasin de données **source** . Si vous n’avez pas de compte de stockage Azure, consultez l’article [Créer un compte de stockage](../../storage/common/storage-account-create.md) pour découvrir comment en créer un.
* **Azure SQL Database**. Vous allez utiliser Azure SQL Database comme magasin de données **cible** dans ce tutoriel. Si vous n’avez pas de base de données dans Azure SQL Database pouvant être utilisée pour le tutoriel, consultez [Comment créer et configurer une base de données dans Azure SQL Database](../../azure-sql/database/single-database-create-quickstart.md) pour en créer une.
* **SQL Server 2012/2014 ou Visual Studio 2013**. Vous allez utiliser SQL Server Management Studio ou Visual Studio pour créer un exemple de base de données et afficher les données de résultat dans la base de données.  

## <a name="collect-blob-storage-account-name-and-key"></a>Récupération du nom de compte Blob Storage et de la clé d'accès
Pour réaliser ce didacticiel, vous avez besoin du nom et de la clé de votre compte de stockage Azure. Notez le **nom** et la **clé** de votre compte de stockage Azure.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Cliquez sur **Tous les services** dans le menu de gauche, puis sélectionnez **Comptes de stockage**.

    ![Parcourir - Comptes de stockage](media/data-factory-copy-data-from-azure-blob-storage-to-sql-database/browse-storage-accounts.png)
3. Dans le panneau **Comptes de stockage**, sélectionnez le **compte de stockage Azure** que vous souhaitez utiliser dans ce didacticiel.
4. Sélectionnez le lien **Clés d’accès** sous **PARAMÈTRES**.
5. Cliquez sur le bouton **copier** (image) situé en regard de la zone de texte **Nom du compte de stockage** et enregistrez/collez-la quelque part (dans un fichier texte, par exemple).
6. Répétez l'étape précédente pour copier ou noter la **clé1**.

    ![Clé d’accès de stockage](media/data-factory-copy-data-from-azure-blob-storage-to-sql-database/storage-access-key.png)
7. Fermez tous les panneaux en cliquant sur **X**.

## <a name="collect-sql-server-database-user-names"></a>Récupérer les noms de serveur SQL, de base de données et d’utilisateur
Pour suivre ce tutoriel, vous avez besoin des noms du serveur SQL logique, de la base de données et de l’utilisateur. Notez les noms du **serveur**, de la **base de données** et de **l’utilisateur** pour Azure SQL Database.

1. Dans le **portail Azure**, cliquez sur **Tous les services** dans le volet gauche, puis sélectionnez **Bases de données SQL**.
2. Dans le panneau **Bases de données SQL**, sélectionnez la **base de données** que vous souhaitez utiliser dans le cadre de ce didacticiel. Notez le **nom de la base de données**.  
3. Dans le panneau **Base de données SQL**, cliquez sur la vignette **Propriétés** sous **PARAMÈTRES**.
4. Notez les valeurs de **NOM DU SERVEUR** et de **CONNEXION D'ADMINISTRATEUR DU SERVEUR**.
5. Fermez tous les panneaux en cliquant sur **X**.

## <a name="allow-azure-services-to-access-sql-server"></a>Autoriser les services Azure à accéder au serveur
Vérifiez que le paramètre **Autoriser l’accès aux services Azure** est **ACTIVÉ** pour votre serveur afin que le service Data Factory puisse y accéder. Pour vérifier et activer ce paramètre, procédez comme suit :

1. Cliquez sur le hub **Tous les services** sur la gauche, puis sur **Serveurs SQL**.
2. Sélectionnez votre serveur, puis cliquez sur **Pare-feu** sous **PARAMÈTRES**.
3. Dans le panneau **Paramètres de pare-feu**, cliquez sur **ACTIVER** pour **Autoriser l’accès aux services Azure**.
4. Fermez tous les panneaux en cliquant sur **X**.

## <a name="prepare-blob-storage-and-sql-database"></a>Préparer Blob Storage et la Base de données SQL
À présent, préparez votre stockage Blob Azure et Azure SQL Database pour ce tutoriel, en procédant comme suit :  

1. Lancez le Bloc-notes. Copiez le texte suivant puis enregistrez-le sous le nom **emp.txt** dans le dossier **C:\ADFGetStarted** sur votre disque dur.

    ```
    John, Doe
    Jane, Doe
    ```
2. Utilisez des outils tels que [l’Explorateur de stockage Azure](https://storageexplorer.com/) pour créer le conteneur **adftutorial** et télécharger le fichier **emp.txt** vers ce dernier.

3. Utilisez le script SQL suivant pour créer la table **emp** dans votre base de données Azure SQL.  

    ```SQL
    CREATE TABLE dbo.emp
    (
        ID int IDENTITY(1,1) NOT NULL,
        FirstName varchar(50),
        LastName varchar(50),
    )
    GO

    CREATE CLUSTERED INDEX IX_emp_ID ON dbo.emp (ID);
    ```

    **Si vous avez installé SQL Server 2012/2014 sur votre ordinateur :** suivez les instructions de l’article [Gestion des bases de données Azure SQL à l’aide de SQL Server Management Studio](../../azure-sql/database/single-database-manage.md) pour vous connecter à votre serveur et exécuter le script SQL.

    Si votre client n’est pas autorisé à accéder au serveur SQL logique, vous devez configurer le pare-feu pour votre serveur afin d’autoriser l’accès à partir de votre ordinateur (adresse IP). Consultez [cet article](../../azure-sql/database/firewall-configure.md) pour savoir comment configurer le pare-feu de votre serveur.

## <a name="create-a-data-factory"></a>Créer une fabrique de données
Vous avez terminé les étapes préalables requises. Créez une fabrique de données à l’aide de l’une des manières suivantes. Cliquez sur l’une des options de la liste déroulante en haut ou sur les liens suivants pour suivre le didacticiel.     

* [Assistant de copie](data-factory-copy-data-wizard-tutorial.md)
* [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
* [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
* [Modèle Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
* [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
* [API .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)

> [!NOTE]
> Dans ce didacticiel, le pipeline de données copie les données d’un magasin de données source vers un magasin de données de destination. Il ne transforme pas les données d’entrée pour produire des données de sortie. Pour suivre un tutoriel sur la transformation des données à l’aide d’Azure Data Factory, consultez [Tutoriel : Générer votre premier pipeline pour transformer les données à l’aide du cluster Hadoop](data-factory-build-your-first-pipeline.md).
>
> Vous pouvez chaîner deux activités (une après l’autre) en configurant le jeu de données de sortie d’une activité en tant que jeu de données d’entrée de l’autre activité. Pour plus d’informations, voir [Planification et exécution dans Data Factory](data-factory-scheduling-and-execution.md).