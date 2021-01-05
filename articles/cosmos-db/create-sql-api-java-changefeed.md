---
title: Créer un exemple d’application SDK Java v4 Azure Cosmos DB de bout en bout à l’aide du flux de modification
description: Ce guide présente une application d’API SQL Java simple qui insère des documents dans un conteneur Azure Cosmos DB, tout en conservant une vue matérialisée du conteneur avec le flux de modification.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: how-to
ms.date: 06/11/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 765fd3afc7fe688d3e6b0e3394e7dc8c39af69b3
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93096850"
---
# <a name="how-to-create-a-java-application-that-uses-azure-cosmos-db-sql-api-and-change-feed-processor"></a>Guide pratique pour créer une application Java qui utilise l’API SQL Azure Cosmos DB et le processeur de flux de modification
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Ce guide de procédure explique comment créer une application Java simple qui utilise l’API SQL Azure Cosmos DB pour insérer des documents dans un conteneur Azure Cosmos DB, tout en conservant une vue matérialisée du conteneur avec le flux de modification et le processeur de flux de modification. L’application Java communique avec l’API SQL Azure Cosmos DB à l’aide du SDK Java v4 Azure Cosmos DB.

> [!IMPORTANT]  
> Ce tutoriel s’applique uniquement au SDK Java v4 Azure Cosmos DB. Pour plus d’informations, consultez les [Notes de publication](sql-api-sdk-java-v4.md) du SDK Java v4 Azure Cosmos DB, le [Référentiel Maven](https://mvnrepository.com/artifact/com.azure/azure-cosmos), les [conseils en matière de performances](performance-tips-java-sdk-v4-sql.md) du SDK Java v4 Azure Cosmos DB et le [guide de dépannage](troubleshoot-java-sdk-v4-sql.md) du SDK Java v4 Azure Cosmos DB. Si vous utilisez actuellement une version antérieure à v4, consultez le guide [Migrer votre application pour utiliser le SDK Java v4 Azure Cosmos DB](migrate-java-v4-sdk.md) pour obtenir de l’aide sur la mise à niveau vers v4.
>

## <a name="prerequisites"></a>Prérequis

* URI et clé de votre compte Azure Cosmos DB

* Maven

* Java 8

## <a name="background"></a>Arrière-plan

Le flux de modification Azure Cosmos DB fournit une interface basée sur des événements pour déclencher des actions en réponse à l’insertion de documents. Il a de nombreux usages. Par exemple, dans les applications gourmandes en opérations de lecture et d’écriture, le flux de modification est important, car il permet de créer une **vue matérialisée** en temps réel d’un conteneur durant le processus d’ingestion des documents. Le conteneur de la vue matérialisée héberge les mêmes données, mais celles-ci sont partitionnées pour optimiser les lectures, ce qui rend l’application performante aussi bien en lecture qu’en écriture.

La gestion des événements du flux de modification est en grande partie prise en charge par la bibliothèque du processeur de flux de modification qui est intégrée au SDK. Cette bibliothèque est suffisamment puissante pour distribuer les événements du flux de modification entre plusieurs workers quand cela est nécessaire. Il vous suffit de fournir un rappel à la bibliothèque du processeur de flux de modification.

Cet exemple simple illustre une bibliothèque du processeur de flux de modification avec un seul worker qui crée et supprime des documents dans une vue matérialisée.

## <a name="setup"></a>Programme d’installation

Si vous ne l’avez pas déjà fait, clonez cet exemple de dépôt d’application :

```bash
git clone https://github.com/Azure-Samples/azure-cosmos-java-sql-app-example.git
```

Ouvrez un terminal à partir du répertoire du dépôt. Générez l’application en exécutant

```bash
mvn clean package
```

## <a name="walkthrough"></a>Procédure pas à pas

1. En premier lieu, vous devez avoir un compte Azure Cosmos DB. Ouvrez le **portail Azure** dans votre navigateur, accédez à votre compte Azure Cosmos DB et, dans le volet de gauche, accédez à l’ **Explorateur de données**.

   :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_account_empty.JPG" alt-text="Compte Azure Cosmos DB":::

1. Exécutez l’application dans le terminal à l’aide de la commande suivante :

    ```bash
    mvn exec:java -Dexec.mainClass="com.azure.cosmos.workedappexample.SampleGroceryStore" -DACCOUNT_HOST="your-account-uri" -DACCOUNT_KEY="your-account-key" -Dexec.cleanupDaemonThreads=false
    ```

1. Appuyez sur Entrée quand vous voyez

    ```bash
    Press enter to create the grocery store inventory system...
    ```

    Retournez ensuite dans l’Explorateur de données du portail Azure dans votre navigateur. Vous voyez qu’une base de données **GroceryStoreDatabase** avec trois conteneurs vides a été ajoutée : 

    * **InventoryContainer**  : enregistrement d’inventaire pour notre exemple de magasin, partitionné sur l’élément ```id``` qui est un UUID.
    * **InventoryContainer-pktype**  : vue matérialisée de l’enregistrement d’inventaire, optimisée pour les requêtes sur l’élément ```type```.
    * **InventoryContainer-leases**  : un conteneur de baux est obligatoire pour le flux de modification ; les baux suivent la progression de l’application dans la lecture du flux de modification.

    :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_account_resources_lease_empty.JPG" alt-text="Conteneurs vides":::

1. Dans le terminal, vous devez maintenant voir une invite

    ```bash
    Press enter to start creating the materialized view...
    ```

    Appuyez sur Entrée. À présent, le bloc de code suivant va exécuter et initialiser le processeur de flux de modification sur un autre thread : 

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-connection-policy-async"></a>API async du SDK Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-app-example/src/main/java/com/azure/cosmos/workedappexample/SampleGroceryStore.java?name=InitializeCFP)]

    ```"SampleHost_1"``` est le nom du worker du processeur de flux de modification. ```changeFeedProcessorInstance.start()``` est ce qui démarre le processeur de flux de modification.

    Retournez ensuite dans l’Explorateur de données du portail Azure dans votre navigateur. Sous le conteneur **InventoryContainer-leases** , cliquez sur **Éléments** pour voir le contenu. Vous voyez que le processeur de flux de modification a rempli le conteneur de baux, c’est-à-dire que le processeur a affecté au worker ```SampleHost_1``` un bail sur certaines partitions du conteneur **InventoryContainer**.

    :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_leases.JPG" alt-text="Baux":::

1. Appuyez une nouvelle fois sur Entrée dans le terminal. Cette action déclenche l’insertion de dix documents dans **InventoryContainer**. Chaque insertion de document apparaît dans le flux de modification au format JSON. Le code de rappel suivant gère ces événements en mettant en miroir les documents JSON dans une vue matérialisée :

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-connection-policy-async"></a>API async du SDK Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-app-example/src/main/java/com/azure/cosmos/workedappexample/SampleGroceryStore.java?name=CFPCallback)]

1. Laissez le code s’exécuter pendant 5 à 10 secondes. Ensuite, retournez dans l’Explorateur de données du portail Azure et accédez à **InventoryContainer > Éléments**. Vous devez voir que les éléments sont insérés dans le conteneur d’inventaire. Notez la clé de partition (```id```).

    :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_items.JPG" alt-text="Conteneur de flux":::

1. À présent, dans l’Explorateur de données, accédez à **InventoryContainer-pktype > Éléments**. Il s’agit de la vue matérialisée : les éléments de ce conteneur sont en miroir d’ **InventoryContainer** car ils ont été insérés programmatiquement par le flux de modification. Notez la clé de partition (```type```). Cette vue matérialisée est optimisée pour un filtrage des requêtes sur ```type```, mais c’est inefficace sur **InventoryContainer** qui est lui partitionné sur ```id```.

    :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_materializedview2.JPG" alt-text="Capture d’écran représentant la page Azure Data Explorer d’un compte Azure Cosmos DB avec des éléments sélectionnés.":::

1. Nous allons supprimer un document dans **InventoryContainer** et **InventoryContainer-pktype** en utilisant un seul appel ```upsertItem()```. Avant cela, jetons un coup d’œil à l’Explorateur de données du portail Azure. Nous supprimerons le document dont l’élément ```/type == "plums"``` est entouré en rouge ci-dessous

    :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_materializedview-emph-todelete.JPG" alt-text="Capture d’écran représentant la page Azure Data Explorer d’un compte Azure Cosmos DB avec un ID d’article particulier sélectionné.":::

    Appuyez de nouveau sur Entrée pour appeler la fonction ```deleteDocument()``` dans l’exemple de code. Cette fonction, illustrée ci-dessous, fait un upsert d’une nouvelle version du document avec ```/ttl == 5```, qui définit la durée de vie (TTL) du document à cinq secondes. 
    
    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-connection-policy-async"></a>API async du SDK Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-app-example/src/main/java/com/azure/cosmos/workedappexample/SampleGroceryStore.java?name=DeleteWithTTL)]

    Le flux de modification ```feedPollDelay``` est défini à 100 ms. Il répond donc presque instantanément à cette mise à jour et appelle ```updateInventoryTypeMaterializedView()``` comme illustré ci-dessus. Ce dernier appel de fonction fait un upsert du nouveau document avec une durée de vie de cinq secondes dans **InventoryContainer-pktype**.

    L’effet est qu’après environ cinq secondes, le document expire et est supprimé des deux conteneurs.

    Cette procédure est nécessaire car le flux de modification émet des événements quand des éléments sont insérés ou mis à jour, mais pas quand ils sont supprimés.

1. Appuyez une dernière fois sur Entrée pour fermer le programme et nettoyer ses ressources.
