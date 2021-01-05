---
title: Se connecter à Synapse SQL avec Power BI Professional
description: Dans ce tutoriel, nous allons suivre les étapes permettant de connecter Power BI Desktop à SQL à un pool SQL serverless.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: sql
ms.date: 04/15/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: b33edb99109b7516e1792497a936031cf954bc15
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96451616"
---
# <a name="connect-to-serverless-sql-pool-with-power-bi-professional"></a>Se connecter à un pool SQL serverless avec Power BI Professional

> [!div class="op_single_selector"]
>
> - [Azure Data Studio](get-started-azure-data-studio.md)
> - [Power BI](get-started-power-bi-professional.md)
> - [Visual Studio](../sql-data-warehouse/sql-data-warehouse-query-visual-studio.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
> - [sqlcmd](../sql/get-started-connect-sqlcmd.md)
> - [SSMS](get-started-ssms.md)

Dans ce tutoriel, nous allons suivre les étapes permettant de connecter Power BI Desktop à un pool SQL serverless.

## <a name="prerequisites"></a>Prérequis

Vous avez besoin des outils suivants pour émettre des requêtes :

- Client SQL de votre choix :

  - Azure Data Studio
  - SQL Server Management Studio

- Power BI Desktop est installé

Paramètres :

| Paramètre                                 | Description                                                   |
| ----------------------------------------- | ------------------------------------------------------------- |
| Adresse du point de terminaison de service du pool SQL serverless    | Utilisée comme nom de serveur                                   |
| Région du point de terminaison de service du pool SQL serverless     | Utilisée pour déterminer le stockage que nous allons utiliser dans les exemples |
| Nom d’utilisateur et mot de passe pour l’accès au point de terminaison | Utilisés pour accéder au point de terminaison                               |
| La base de données que vous allez utiliser pour créer des vues       | Cette base de données est utilisée comme point de départ dans les exemples       |

## <a name="first-time-setup"></a>Première configuration

Deux étapes précèdent l’utilisation des exemples :

1. Créer une base de données pour vos vues
2. Créer les informations d’identification que devra utiliser le pool SQL serverless pour accéder aux fichiers dans le stockage

### <a name="create-database"></a>Créer une base de données

Pour cet article de démarrage, vous devez créer votre propre base de données pour l’utiliser en démonstration. Une base de données est nécessaire pour créer des vues. Vous allez utiliser cette base de données dans certains exemples de requêtes proposés dans cette documentation.

> [!NOTE]
> Les bases de données sont utilisées uniquement pour l’affichage de métadonnées, et non de données réelles.
>
> Notez le nom de la base de données que vous utilisez, vous en aurez besoin par la suite.

```sql
DROP DATABASE IF EXISTS demo;
```

### <a name="create-credentials"></a>Créer des informations d’identification

Nous devons créer des informations d’identification afin que vous puissiez exécuter des requêtes. Ces informations d’identification seront utilisées par le service de pool SQL serverless pour accéder aux fichiers dans le stockage.

> [!NOTE]
> Vous devez créer des informations d’identification pour l’accès au compte de stockage. Même si le pool SQL serverless peut accéder au stockage à partir de différentes régions, le fait d’avoir le stockage et l’espace de travail Azure Synapse dans la même région offre une meilleure expérience sur le plan des performances.

**Extrait de code sur la façon de créer des informations d’identification pour les conteneurs de données de recensement**. Exécutez ce qui suit :

```sql
IF EXISTS (SELECT * FROM sys.credentials WHERE name = 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer')
DROP CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer];
GO

-- Create credentials for Census Data container which resides in a azure open data storage account
-- There is no secret. We are using public storage account which doesn't need secret
CREATE CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]  
WITH IDENTITY='SHARED ACCESS SIGNATURE',  
SECRET = '';
GO
```

## <a name="create-a-power-bi-desktop-report"></a>Créer un rapport Power BI Desktop

Ouvrez l’application Power BI Desktop et sélectionnez l’option **Obtenir les données**.

![Ouvrez l’application Power BI Desktop et sélectionnez « Obtenir les données ».](./media/get-started-power-bi-professional/step-0-open-powerbi.png)

### <a name="step-1---select-data-source"></a>Étape 1 : Sélectionner la source de données

Sélectionnez **Azure** dans le menu, puis **Azure SQL Database**.
![Sélectionnez la source de données.](./media/get-started-power-bi-professional/step-1-select-data-source.png)

### <a name="step-2---select-database"></a>Étape 2 : Sélectionner la base de données

Entrez l’URL de la base de données et le nom de la base de données où réside la vue.
![Sélectionnez la base de données sur le point de terminaison.](./media/get-started-power-bi-professional/step-2-db.png)

## <a name="next-steps"></a>Étapes suivantes

Passez à [Interroger des fichiers de stockage](get-started-azure-data-studio.md) pour savoir comment vous connecter au pool SQL serverless à l’aide d’Azure Data Studio.