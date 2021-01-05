---
title: Utiliser les données de référence SQL Database dans une tâche Azure Stream Analytics
description: Cet article décrit comment utiliser une base de données SQL comme entrée de données de référence pour une tâche Azure Stream Analytics dans le portail Azure et dans Visual Studio.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 01/29/2019
ms.openlocfilehash: 7e2826221bd9d15472467c4dd8676d3d0538e0d6
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96326571"
---
# <a name="use-reference-data-from-a-sql-database-for-an-azure-stream-analytics-job"></a>Utiliser les données de référence d’une base de données SQL pour une tâche Azure Stream Analytics

Azure Stream Analytics prend en charge l'utilisation d'Azure SQL Database comme source d'entrée de données de référence. Vous pouvez utiliser les données SQL Database comme données de référence pour votre travail Stream Analytics sur le portail Azure et dans Visual Studio avec les outils Stream Analytics. Cet article décrit les deux méthodes.

## <a name="azure-portal"></a>Portail Azure

Utilisez les étapes suivantes pour ajouter une base de données Azure SQL comme source d’entrée de référence à l’aide du portail Azure :

### <a name="portal-prerequisites"></a>Prérequis pour le portail

1. Créez une tâche Stream Analytics.

2. Créez un compte de stockage qui sera utilisé par la tâche Stream Analytics.

3. Créez votre base de données Azure SQL avec un jeu de données à utiliser comme données de référence par la tâche Stream Analytics.

### <a name="define-sql-database-reference-data-input"></a>Définir la base de données SQL comme entrée de données de référence

1. Dans votre tâche Stream Analytics, sélectionnez **Entrées** sous **Topologie de la tâche**. Cliquez sur **Ajouter une entrée de référence** et choisissez **Base de données SQL**.

   ![Les entrées sont sélectionnées dans le volet de navigation de gauche. Dans le volet Entrées, l’option Ajouter la référence est sélectionnée, révélant une liste déroulante qui affiche les valeurs Stockage Blob et SQL Database.](./media/sql-reference-data/stream-analytics-inputs.png)

2. Renseignez les configurations d’entrée de Stream Analytics. Choisissez le nom de la base de données, le nom du serveur, le nom d’utilisateur et le mot de passe. Si vous souhaitez actualiser périodiquement votre entrée de données de référence, choisissez « Activé » pour spécifier la fréquence d’actualisation au format JJ:HH:MM. Si vous avez des jeux de données volumineux avec une fréquence de d’actualisation courte, vous pouvez utiliser une [requête delta](sql-reference-data.md#delta-query).

   ![Lorsque SQL Database est sélectionné, la page Nouvelle entrée SQL Database s’affiche. Un formulaire de configuration apparaît dans le volet gauche, et une requête d’instantané s’affiche dans le volet droit.](./media/sql-reference-data/sql-input-config.png)

3. Testez la requête d’instantané dans l’éditeur de requête SQL. Pour plus d’informations, consultez [Utiliser l’éditeur de requête SQL du portail Azure pour se connecter aux données et les interroger](../azure-sql/database/connect-query-portal.md).

### <a name="specify-storage-account-in-job-config"></a>Spécifier le compte de stockage dans la configuration de la tâche

Accédez à **Paramètres du compte de stockage** sous **Configurer**, puis sélectionnez **Ajouter un compte de stockage**.

   ![Les paramètres du compte de stockage sont sélectionnés dans le volet gauche. Le volet droit contient un bouton Ajouter un compte de stockage.](./media/sql-reference-data/storage-account-settings.png)

### <a name="start-the-job"></a>Démarrage du travail

Après avoir configuré les autres entrées, les sorties et la requête, vous pouvez démarrer la tâche Stream Analytics.

## <a name="tools-for-visual-studio"></a>Outils pour Visual Studio

Suivez ces étapes pour ajouter Azure SQL Database comme source d’entrée de référence à l’aide de Visual Studio :

### <a name="visual-studio-prerequisites"></a>Prérequis pour Visual Studio

1. [Installez les outils Stream Analytics pour Visual Studio](stream-analytics-tools-for-visual-studio-install.md). Les versions suivantes de Visual Studio sont prises en charge :

   * Visual Studio 2015
   * Visual Studio 2019

2. Prenez connaissance du guide de démarrage rapide traitant des [outils Stream Analytics pour Visual Studio](stream-analytics-quick-create-vs.md).

3. Créez un compte de stockage.

### <a name="create-a-sql-database-table"></a>Création d’une table de base de données SQL

Utilisez SQL Server Management Studio pour créer une table dans laquelle stocker vos données de référence. Pour plus d’informations, consultez [Concevoir votre première base de données Azure SQL à l’aide de SSMS](../azure-sql/database/design-first-database-tutorial.md).

L’exemple de table utilisé dans l’exemple suivant a été créé à partir de l’instruction suivante :

```SQL
create table chemicals(Id Bigint,Name Nvarchar(max),FullName Nvarchar(max));
```

### <a name="choose-your-subscription"></a>Choisir votre abonnement

1. Dans le menu **Affichage** de Visual Studio, sélectionnez **Explorateur de serveurs**.

2. Cliquez avec le bouton droit sur **Azure**, Sélectionnez **Se connecter à un abonnement Microsoft Azure**, puis connectez-vous avec votre compte Azure.

### <a name="create-a-stream-analytics-project"></a>Créer un projet Stream Analytics

1. Sélectionnez **Fichier > Nouveau Projet**. 

2. Dans la liste des modèles sur la gauche, sélectionnez **Stream Analytics**, puis **Application Azure Stream Analytics**. 

3. Entrez les éléments **Nom**, **Emplacement** et **Nom de la solution** du projet, puis sélectionnez **OK**.

   ![Le modèle Stream Analytics est sélectionné, l’option Application Azure Stream Analytics est sélectionnée, et les zones Nom, Emplacement et Solution sont mises en surbrillance.](./media/sql-reference-data/stream-analytics-vs-new-project.png)

### <a name="define-sql-database-reference-data-input"></a>Définir la base de données SQL comme entrée de données de référence

1. Créez une entrée.

   ![Dans Ajouter un nouvel élément, Entrée est sélectionnée.](./media/sql-reference-data/stream-analytics-vs-input.png)

2. Double-cliquez sur **Input.json** dans l’**Explorateur de solutions**.

3. Renseignez la **configuration d’entrée Stream Analytics**. Choisissez le nom de la base de données, le nom du serveur, le type d’actualisation et la fréquence d’actualisation. Spécifiez la fréquence d’actualisation au format `DD:HH:MM`.

   ![Dans Configuration de l’entrée Stream Analytics, les valeurs sont entrées ou sélectionnées dans des listes déroulantes.](./media/sql-reference-data/stream-analytics-vs-input-config.png)

   Si vous choisissez « Exécuter une seule fois » ou « Exécuter périodiquement », un fichier code-behind SQL nommé **[Alias d’entrée].snapshot.sql** est généré dans le projet sous le nœud de fichier **Input.json**.

   ![Le fichier SQL CodeBehind Chemicals.snapshot.sql est mis en surbrillance.](./media/sql-reference-data/once-or-periodically-codebehind.png)

   Si vous choisissez « Actualiser régulièrement avec une requête delta », deux fichiers code-behind SQL sont générés : **[Alias d’entrée].snapshot.sql** et **[Alias d’entrée].delta.sql**.

   ![Les fichiers SQL CodeBehind Chemicals.delta.sql et Chemicals.snapshot.sql sont mis en surbrillance.](./media/sql-reference-data/periodically-delta-codebehind.png)

4. Ouvrez le fichier SQL dans l’éditeur et écrivez la requête SQL.

5. Si vous utilisez Visual Studio 2019 et que vous avez installé SQL Server Data Tools, vous pouvez tester la requête en cliquant sur **Exécuter**. Une fenêtre d’Assistant s’affiche pour vous aider à vous connecter à la base de données SQL. Le résultat de la requête s’affiche en bas de la fenêtre.

### <a name="specify-storage-account"></a>Spécifier le compte de stockage

Ouvrez **JobConfig.json** pour spécifier le compte de stockage utilisé pour stocker les instantanés de référence SQL.

   ![La fenêtre Configuration de la tâche Stream Analytics s’affiche avec les valeurs par défaut. Les paramètres de stockage globaux sont mis en surbrillance.](./media/sql-reference-data/stream-analytics-job-config.png)

### <a name="test-locally-and-deploy-to-azure"></a>Tester localement et déployer sur Azure

Avant de déployer la tâche sur Azure, vous pouvez tester la logique de la requête localement par rapport à des données d’entrée actives. Pour plus d’informations sur cette fonctionnalité, consultez [Tester des données actives localement à l’aide des outils Azure Stream Analytics pour Visual Studio (préversion)](stream-analytics-live-data-local-testing.md). Une fois les tests terminés, cliquez sur **Envoyer sur Azure**. Pour savoir comment démarrer la tâche, reportez-vous au guide de démarrage rapide [Créer une tâche Stream Analytics à l’aide des outils Azure Stream Analytics pour Visual Studio](stream-analytics-quick-create-vs.md).

## <a name="delta-query"></a>Requête delta

Quand vous utilisez la requête delta, des [tables temporelles dans Azure SQL Database](../azure-sql/temporal-tables.md) sont recommandées.

1. Créer une table temporelle dans la base de données Azure SQL Database.
   
   ```SQL 
      CREATE TABLE DeviceTemporal 
      (  
         [DeviceId] int NOT NULL PRIMARY KEY CLUSTERED 
         , [GroupDeviceId] nvarchar(100) NOT NULL
         , [Description] nvarchar(100) NOT NULL 
         , [ValidFrom] datetime2 (0) GENERATED ALWAYS AS ROW START
         , [ValidTo] datetime2 (0) GENERATED ALWAYS AS ROW END
         , PERIOD FOR SYSTEM_TIME (ValidFrom, ValidTo)
      )  
      WITH (SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.DeviceHistory));  -- DeviceHistory table will be used in Delta query
   ```
2. Créez la requête de capture instantanée. 

   Utilisez le paramètre **\@snapshotTime** pour indiquer au runtime Stream Analytics d’obtenir le jeu de données de référence à partir de la table temporelle de la base de données SQL valide à l’heure système. Si vous ne fournissez pas ce paramètre, vous risquez d’obtenir un jeu de données de référence de base incorrect en raison des décalages d’horloge. Un exemple complet de requête d’instantané est présenté ci-dessous :
   ```SQL
      SELECT DeviceId, GroupDeviceId, [Description]
      FROM dbo.DeviceTemporal
      FOR SYSTEM_TIME AS OF @snapshotTime
   ```
 
2. Créez la requête delta. 
   
   Cette requête récupère toutes les lignes dans la base de données SQL qui ont été insérées ou supprimées entre l’heure de début **\@deltaStartTime** et l’heure de fin **\@deltaEndTime**. La requête delta doit retourner les mêmes colonnes que la requête d’instantané, ainsi que l’opération (**_operation_**) de la colonne. Cette colonne définit si la ligne est insérée ou supprimée entre **\@deltaStartTime** et **\@deltaEndTime**. Les lignes obtenues sont marquées avec le chiffre **1** si les enregistrements ont été insérés ou avec le chiffre **2** s’ils ont été supprimés. La requête doit également ajouter un **filigrane** du côté SQL Server pour s’assurer que toutes les mises à jour de la période delta sont correctement capturées. L’utilisation de la requête delta sans **filigrane** peut produire un jeu de données de référence incorrect.  

   Pour les enregistrements qui ont été mis à jour, la table temporelle se charge de la comptabilité en capturant une opération d’insertion et de suppression. Le runtime Stream Analytics applique ensuite les résultats de la requête delta à l’instantané précédent pour conserver les données de référence à jour. Un exemple de requête delta est présenté ci-dessous :

   ```SQL
      SELECT DeviceId, GroupDeviceId, Description, ValidFrom as _watermark_, 1 as _operation_
      FROM dbo.DeviceTemporal
      WHERE ValidFrom BETWEEN @deltaStartTime AND @deltaEndTime   -- records inserted
      UNION
      SELECT DeviceId, GroupDeviceId, Description, ValidTo as _watermark_, 2 as _operation_
      FROM dbo.DeviceHistory   -- table we created in step 1
      WHERE ValidTo BETWEEN @deltaStartTime AND @deltaEndTime     -- record deleted
   ```
 
   Notez que le runtime Stream Analytics peut exécuter périodiquement la requête d’instantané en plus de la requête delta pour stocker des points de contrôle.

## <a name="test-your-query"></a>Tester votre requête
   Il est important de vérifier que votre requête renvoie le jeu de données attendu, que le travail Stream Analytics utilisera comme information de référence. Pour tester votre requête, accédez à la zone Entrée, dans la section Topologie de la tâche sur le portail. Vous pouvez ensuite sélectionner des échantillons de données sur votre entrée de référence Microsoft Azure SQL Database. Lorsque l’échantillon est disponible, vous pouvez télécharger le fichier et vérifier que les données renvoyées correspondent aux attentes. Si vous souhaitez optimiser vos itérations de test et de développement, il est recommandé d’utiliser les [outils Azure Steam Analytics pour Visual Studio](./stream-analytics-tools-for-visual-studio-install.md). Vous pouvez également utiliser l’outil de votre choix pour vérifier au préalable que la requête renvoie les bons résultats de Microsoft Azure SQL Database, puis l’utiliser dans votre travail Steam Analytics. 

### <a name="test-your-query-with-visual-studio-code"></a>Tester votre requête avec Visual Studio Code

   Installez [Azure Stream Analytics Tools](https://marketplace.visualstudio.com/items?itemName=ms-bigdatatools.vscode-asa) et [SQL Server (mssql)](https://marketplace.visualstudio.com/items?itemName=ms-mssql.mssql) sur Visual Studio Code et configurez votre projet ASA. Pour plus d’informations, consultez [Démarrage rapide : Créer une tâche Azure Stream Analytics dans Visual Studio Code](./quick-create-visual-studio-code.md) et le [tutoriel sur l’extension SQL Server (mssql)](/sql/tools/visual-studio-code/sql-server-develop-use-vscode).

1. Configurez votre entrée de données de référence SQL.
   
   ![Un éditeur Visual Studio Code (onglet) affiche ReferenceSQLDatabase.json.](./media/sql-reference-data/configure-sql-reference-data-input.png)

2. Sélectionnez l’icône SQL Server, puis cliquez sur **Ajouter une connexion**.
   
   ![L’option Ajouter une connexion apparaît dans le volet gauche et est mise en surbrillance.](./media/sql-reference-data/add-sql-connection.png)

3. Renseignez les informations de connexion.
   
   ![Les deux zones pour les informations de base de données et de serveur sont mises en surbrillance.](./media/sql-reference-data/fill-connection-information.png)

4. Cliquez avec le bouton droit dans les données de référence SQL, puis sélectionnez **Exécuter la requête**.
   
   ![L’option Exécuter la requête est mise en surbrillance dans le menu contextuel.](./media/sql-reference-data/execute-query.png)

5. Choisissez votre connexion.
   
   ![La boîte de dialogue indique « Create a connection profile from the list below » (créer un profil de connexion dans la liste ci-dessous), et la liste comporte une entrée mise en surbrillance.](./media/sql-reference-data/choose-connection.png)

6. Examinez et vérifiez le résultat de votre requête.
   
   ![Les résultats de la recherche de requête s’affichent dans un onglet de l’éditeur VS Code.](./media/sql-reference-data/verify-result.png)


## <a name="faqs"></a>FAQ

**L’utilisation d’une entrée de données de référence SQL dans Azure Stream Analytics entraîne-t-elle des frais supplémentaires ?**

Le [coût par unité de streaming](https://azure.microsoft.com/pricing/details/stream-analytics/) dans la tâche Stream Analytics ne change pas. Toutefois, la tâche Stream Analytics doit avoir un compte de stockage Azure associé. La tâche Stream Analytics interroge la base de données SQL (au démarrage de la tâche et à chaque intervalle d’actualisation) pour récupérer le jeu de données de référence, puis stocke cet instantané dans le compte de stockage. Le stockage de ces instantanés engendre des frais supplémentaires décrits dans la [page des tarifs](https://azure.microsoft.com/pricing/details/storage/) du compte de stockage Azure.

**Comment faire pour savoir si un instantané de données de référence est interrogé à partir de la base de données SQL et utilisé dans la tâche Azure Stream Analytics ?**

Vous pouvez recourir à deux mesures filtrées par nom logique (sous Mesures, dans le portail Microsoft Azure) pour superviser l’intégrité de la base de données SQL utilisée comme entrée de données de référence.

   * InputEvents : Cette métrique mesure le nombre d’enregistrements chargés à partir du jeu de données de référence de la base de données SQL.
   * InputEventBytes : Cette métrique mesure la taille de l’instantané de données de référence chargé en mémoire de la tâche Stream Analytics. 

La combinaison de ces deux métriques permet de déduire si la tâche interroge la base de données SQL pour récupérer le jeu de données de référence et le charge en mémoire.

**La base de données Azure SQL doit-elle être d’un type spécial ?**

Azure Stream Analytics fonctionne avec n’importe quel type de base de données Azure SQL. Toutefois, il est important de comprendre que la fréquence d’actualisation définie pour votre entrée de données de référence peut impacter la charge de votre requête. Pour utiliser l’option de requête delta, il est recommandé d’utiliser des tables temporelles dans Azure SQL Database.

**Pourquoi Azure Stream Analytics stocke-t-il des instantanés dans un compte de stockage Azure ?**

Stream Analytics garantit un traitement des événements « Exactement une fois » et une remise des événements « Une fois au minimum ». Dans les cas où des problèmes temporaires impactent votre travail, une petite quantité de relecture est nécessaire pour restaurer l’état. Pour permettre la relecture, les instantanés doivent être stockés dans un compte de stockage Azure. Pour plus d’informations sur la relecture des points de contrôle, consultez [Concepts de point de contrôle et de relecture dans les tâches Azure Stream Analytics](stream-analytics-concepts-checkpoint-replay.md).

## <a name="next-steps"></a>Étapes suivantes

* [Utilisation de données de référence pour effectuer des recherches dans Stream Analytics](stream-analytics-use-reference-data.md)
* [Démarrage rapide : créer une tâche Stream Analytics à l’aide des outils Azure Stream Analytics pour Visual Studio](stream-analytics-quick-create-vs.md)
* [Tester des données actives localement à l’aide des outils Azure Stream Analytics pour Visual Studio (préversion)](stream-analytics-live-data-local-testing.md)