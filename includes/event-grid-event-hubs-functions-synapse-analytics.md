---
title: Fichier include
description: Fichier include
services: event-grid
author: spelluru
ms.service: event-grid
ms.topic: include
ms.date: 12/07/2020
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: d6b8b35b36830568d6ff8c46a381fec0cfc57e84
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96854156"
---
:::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/overview.png" alt-text="Vue d’ensemble de l’application":::

Ce diagramme illustre le flux de travail de la solution que vous créez dans ce didacticiel : 

1. Les données envoyées à un hub d’événements Azure sont capturées dans un stockage d’objet blob Azure.
2. Lorsque la capture de données est terminée, un événement est généré et envoyé à Azure Event Grid. 
3. Event Grid transfert ces données d’événements à une application de fonction Azure.
4. L’application de fonction utilise l’URL d’objet blob dans les données d’événement pour récupérer l’objet blob depuis le stockage. 
5. L’application de fonction effectue la migration des données blob vers Azure Synapse Analytics. 

Dans cet article, vous procédez comme suit :

> [!div class="checklist"]
> - Déployer l’infrastructure requise pour le tutoriel
> - Publier du code vers une application de fonction
> - Créer un abonnement Event Grid 
> - Envoyer en streaming les exemples de données vers Event Hubs
> - Vérifier les données capturées dans Azure Synapse Analytics

## <a name="prerequisites"></a>Prérequis
Pour suivre ce didacticiel, vous avez besoin de :

- Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
- [Visual studio 2019](https://www.visualstudio.com/vs/) avec des charges de travail pour : le développement de bureau .NET, le développement Azure, le développement ASP.NET et web, le développement Node.js et le développement Python.
- Télécharger l’[exemple de projet EventHubsCaptureEventGridDemo](https://github.com/Azure/azure-event-hubs/tree/master/samples/e2e/EventHubsCaptureEventGridDemo) sur votre ordinateur.
    - WindTurbineDataGenerator – un éditeur simple qui envoie des exemples de données d’éoliennes vers un hub d’événements avec la capture activée
    - FunctionDWDumper – une Azure Function qui reçoit une notification de Event Grid lorsqu’un fichier Avro est capturé dans l’objet blob de stockage Azure. Elle reçoit le chemin de l’URI de l’objet blob, lit son contenu et envoie (push) ces données vers Azure Synapse Analytics (pool SQL dédié).

## <a name="deploy-the-infrastructure"></a>Déployer l’infrastructure
Dans cette étape, vous déployez l’infrastructure requise avec un [modèle Resource Manager](https://github.com/Azure/azure-docs-json-samples/blob/master/event-grid/EventHubsDataMigration.json). Lorsque vous déployez le modèle, les ressources suivantes sont créées :

* Un hub d’événements avec la fonctionnalité Capture activée.
* Un compte de stockage pour fichiers capturés. 
* Un plan App Service pour héberger l’application de fonction.
* Application de fonction pour traiter l’événement
* SQL Server pour héberger l’entrepôt de données
* Azure Synapse Analytics (pool SQL dédié) pour stocker les données migrées

### <a name="use-azure-cli-to-deploy-the-infrastructure"></a>Utiliser Azure CLI pour déployer l’infrastructure

1. Connectez-vous au [portail Azure](https://portal.azure.com). 
2. Sélectionnez le bouton **Cloud Shell** en haut.

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/azure-portal.png" alt-text="Azure portal":::
3. Cloud Shell s’ouvre dans la partie inférieure du navigateur.

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/launch-cloud-shell.png" alt-text="Cloud Shell":::
4. Dans Cloud Shell, si vous avez la possibilité de choisir entre **Bash** et **PowerShell**, sélectionnez **Bash**. 
5. Si vous utilisez Cloud Shell pour la première fois, créez un compte de stockage en sélectionnant **Créer le stockage**. Azure Cloud Shell requiert un compte de stockage Azure pour stocker des fichiers. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/create-storage-cloud-shell.png" alt-text="Créer un stockage pour Cloud Shell":::
6. Attendez que le Cloud Shell soit initialisé. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/cloud-shell-initialized.png" alt-text="Cloud Shell initialisé":::
1. Créez un groupe de ressources Azure en exécutant la commande CLI suivante : 
    1. Copiez et collez la commande suivante dans la fenêtre Cloud Shell. Si vous le souhaitez, modifiez le nom et l’emplacement du groupe de ressources.

        ```azurecli
        az group create -l eastus -n rgDataMigration
        ```
    2. Appuyez sur **Entrée**. 

        Voici un exemple :
    
        ```azurecli
        user@Azure:~$ az group create -l eastus -n rgDataMigration
        {
          "id": "/subscriptions/00000000-0000-0000-0000-0000000000000/resourceGroups/rgDataMigration",
          "location": "eastus",
          "managedBy": null,
          "name": "rgDataMigration",
          "properties": {
            "provisioningState": "Succeeded"
          },
          "tags": null
        }
        ```
2. Déployez toutes les ressources mentionnées dans la section précédente (hub d’événements, compte de stockage, application de fonction, Azure Synapse Analytics) en exécutant la commande CLI suivante : 
    1. Copiez et collez la commande dans la fenêtre Cloud Shell. Vous pouvez également copier/coller la commande dans l’éditeur de votre choix, définir des valeurs, puis la copiez dans Cloud Shell. 

        > [!IMPORTANT]
        > Spécifiez des valeurs pour les entités suivantes avant d’exécuter la commande : 
        > - Nom du groupe de ressources créé précédemment.
        > - Nom de l’espace de noms Event Hub. 
        > - Nom du hub d’événements. Vous pouvez laisser la valeur telle quelle (hubdatamigration).
        > - Nom du serveur SQL.
        > - Nom de l’utilisateur SQL et mot de passe. 
        > - Nom de la base de données.
        > - Nom du compte de stockage. 
        > - Nom de l’application de fonction. 


        ```azurecli
        az deployment group create \
            --resource-group rgDataMigration \
            --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/event-grid/EventHubsDataMigration.json \
            --parameters eventHubNamespaceName=<event-hub-namespace> eventHubName=hubdatamigration sqlServerName=<sql-server-name> sqlServerUserName=<user-name> sqlServerPassword=<password> sqlServerDatabaseName=<database-name> storageName=<unique-storage-name> functionAppName=<app-name>
        ```
    3.  Appuyez sur **Entrée** dans la fenêtre Cloud Shell pour exécuter la commande. Ce processus peut prendre un certain temps dans la mesure où vous créez beaucoup de ressources. Dans le résultat de la commande, assurez-vous qu’il n’y a eu aucun échec. 
1. Fermez Cloud Shell en sélectionnant le bouton **Cloud Shell** dans le portail (ou) le bouton **X** dans le coin supérieur droit de la fenêtre Cloud Shell. 

### <a name="verify-that-the-resources-are-created"></a>Vérifier que les ressources ont été créées

1. Dans le portail Azure, sélectionnez **Groupes de ressources** dans le menu de gauche. 
2. Filtrez la liste des groupes de ressources en entrant le nom de votre groupe de ressources dans la zone de recherche. 
3. Sélectionnez votre groupe de ressources dans la liste.

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/select-resource-group.png" alt-text="Sélectionnez votre groupe de ressources":::
4. Vérifiez que les ressources suivantes s’affichent dans le groupe de ressources :

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/resources-in-resource-group.png" alt-text="Ressources comprises dans le groupe de ressources" lightbox="media/event-grid-event-hubs-functions-synapse-analytics/resources-in-resource-group.png":::

### <a name="create-a-table-in-azure-synapse-analytics"></a>Créer une table dans Azure Synapse Analytics
Créez une table dans votre entrepôt de données en exécutant le script [CreateDataWarehouseTable.sql](https://github.com/Azure/azure-event-hubs/blob/master/samples/e2e/EventHubsCaptureEventGridDemo/scripts/CreateDataWarehouseTable.sql). Pour exécuter le script, vous pouvez utiliser Visual Studio ou l’éditeur de requête dans le portail. Les étapes suivantes montrent comment utiliser l’éditeur de requête : 

1. Dans la liste des ressources comprises dans le groupe de ressources, sélectionnez votre **pool SQL dédié**. 
2. Dans la page **Pool SQL dédié**, dans la section **Tâches courantes** du menu de gauche, sélectionnez **Éditeur de requêtes (préversion)** . 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/sql-data-warehouse-page.png" alt-text="Page Azure Synapse Analytics":::
2. Entrez le nom d’**utilisateur** et le **mot de passe** du serveur SQL, puis sélectionnez **OK**. Si un message s’affiche pour autoriser votre client à accéder au serveur SQL Server, effectuez ces étapes :
    1. Sélectionnez le lien : **Définir le pare-feu du serveur**. 
    2. Dans la page **Paramètres du pare-feu**, sélectionnez **Ajouter une adresse IP cliente** dans la barre d’outils, puis sélectionnez **Enregistrer** dans la barre d’outils. 
    3. Sélectionnez **OK** dans le message de réussite.
    4. Revenez dans la page **Pool SQL dédié**, puis sélectionnez **Éditeur de requêtes (préversion)** dans le menu de gauche. 
    5. Entrez l'**utilisateur** et le **mot de passe**, puis sélectionnez **OK**. 
1. Dans la fenêtre de requête, copiez et exécutez le script SQL suivant : 

    ```sql
    CREATE TABLE [dbo].[Fact_WindTurbineMetrics] (
        [DeviceId] nvarchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL, 
        [MeasureTime] datetime NULL, 
        [GeneratedPower] float NULL, 
        [WindSpeed] float NULL, 
        [TurbineSpeed] float NULL
    )
    WITH (CLUSTERED COLUMNSTORE INDEX, DISTRIBUTION = ROUND_ROBIN);
    ```

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/run-sql-query.png" alt-text="Exécuter une requête SQL":::
5. Ne fermez pas cet onglet ou cette fenêtre pour pouvoir vérifier que les données ont été créées à la fin du didacticiel. 

### <a name="update-the-function-runtime-version"></a>Mettre à jour la version d’exécution de la fonction

1. Ouvrez un autre onglet dans le navigateur web, puis accédez au [portail Azure](https://portal.azure.com).
1. Dans le portail Azure, sélectionnez **Groupes de ressources** dans le menu de gauche.
1. Sélectionnez le groupe de ressources dans lequel se trouve l’application de fonction. 
1. Sélectionnez l’**application de fonction** dans la liste des ressources du groupe de ressources.
1. Sélectionnez **Configuration** sous **Paramètres** dans le volet gauche. 
1. Basculez vers l’onglet **Paramètres d’exécution de la fonction** dans le volet droit. 
1. Mettez à jour la **version du runtime** sur **~3**. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/function-runtime-version.png" alt-text="Mettre à jour la version d’exécution de la fonction":::
6. Sélectionnez **Enregistrer** dans la barre d’outils. 
1. Dans la fenêtre contextuelle de confirmation **Enregistrer les modifications**, sélectionnez **Continuer**. 

## <a name="publish-the-azure-functions-app"></a>Publie l’application Azure Functions

1. Lancez Visual Studio.
2. Ouvrez la solution **EventHubsCaptureEventGridDemo.sln** que vous avez téléchargée depuis [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/e2e/EventHubsCaptureEventGridDemo) dans le cadre des conditions préalables. La solution se trouve dans le dossier `/samples/e2e/EventHubsCaptureEventGridDemo`. 
3. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **FunctionEGDWDumper**, puis sélectionnez **Publier**.
4. Si l’écran suivant apparaît, sélectionnez **Démarrer**. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/start-publish-button.png" alt-text="Bouton Démarrer dans la section Publier.":::
5. Dans la boîte de dialogue **Publier**, sélectionnez **Azure** dans **Cible**, puis sélectionnez **Suivant**. 
6. Sélectionnez **Application de fonction Azure (Windows)** , puis **Suivant**.
7. Sous l’onglet **Instance Functions**, sélectionnez votre abonnement Azure, développez le groupe de ressources, sélectionnez votre application de fonction, puis sélectionnez **Terminer**. Vous devez le cas échéant vous connecter à votre compte Microsoft Azure. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/publish-select-function-app.png" alt-text="Sélectionnez votre application de fonction":::
8. Dans la page **Publier**, dans la section **Dépendances de service**, sélectionnez **Configurer** pour **Stockage**. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/publish-storage-configure-link.png" alt-text="Sélectionner le lien Configurer pour la dépendance du service de stockage":::
1. Dans la page **Configurer la dépendance**, effectuez les étapes suivantes : 
    1. Sélectionnez le **compte de stockage** que vous avez préalablement créé, puis sélectionnez **Suivant**. 

        :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/select-dependency-storage.png" alt-text="Sélectionner un compte de stockage":::
    10. Spécifiez un **nom de chaîne de connexion**, sélectionnez **Aucun** pour l’option **Enregistrer la chaîne de connexion**, puis sélectionnez **Suivant**. 
    
        :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/dependency-storage-connection-string.png" alt-text="Spécifier un nom de chaîne de connexion":::      
    1. Désactivez l’option **Fichier de code C#** et l’option **Magasin de secrets**, puis sélectionnez **Terminer**.  
    
        :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/dependency-storage-changes-summary.png" alt-text="Revoir le récapitulatif des modifications":::
1. Lorsque Visual Studio a configuré le profil, sélectionnez **Publier**.

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/select-publish.png" alt-text="Select publish":::
2. Sous l’onglet où la page **Azure Function** est ouverte, sélectionnez **Fonctions** dans le menu de gauche. Vérifiez que la fonction **EventGridTriggerMigrateData** figure dans la liste. Si vous ne la voyez pas, réessayez de la publier à partir de Visual Studio, puis actualisez la page dans le portail. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/confirm-function-creation.png" alt-text="Confirmer la création de la fonction":::    

Après la publication de la fonction, vous êtes prêt à vous abonner à l’événement.

## <a name="subscribe-to-the-event"></a>S’abonner à l’événement

1. Accédez au [portail Azure](https://portal.azure.com) depuis une nouvelle fenêtre ou un nouvel onglet dans un navigateur web.
2. Dans le portail Azure, sélectionnez **Groupes de ressources** dans le menu de gauche. 
3. Filtrez la liste des groupes de ressources en entrant le nom de votre groupe de ressources dans la zone de recherche. 
4. Sélectionnez votre groupe de ressources dans la liste.
1. Sélectionnez l’**espace de noms Event Hubs** dans la liste des ressources.
1. Dans la page **Espace de noms Event Hubs** , sélectionnez **Événements** dans le menu de gauche, puis sélectionnez **+ Abonnement aux événements** dans la barre d’outils. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/event-hub-add-subscription-link.png" alt-text="Lien Ajouter un abonnement à un événement dans la page Événements pour un espace de noms Event Hubs":::
1. Dans la page **Créer un abonnement aux événements**, effectuez les étapes suivantes :
    1. Entrez un nom pour l’**abonnement aux événements**. 
    1. Entrez un nom pour la **rubrique système**. Une rubrique système fournit un point de terminaison permettant l’envoi d’événements par l’expéditeur. Pour plus d’informations, consultez [Rubriques système](../articles/event-grid/system-topics.md)
    1. Comme **Type de point de terminaison**, sélectionnez **Fonction Azure**.
    1. Pour **Point de terminaison**, sélectionnez le lien.
    1. Dans la page **Sélectionner une fonction Azure**, effectuez les étapes suivantes pour les options qui ne sont pas automatiquement renseignées.
        1. Sélectionnez l’abonnement Azure qui contient la fonction Azure. 
        1. Sélectionnez le groupe de ressources pour la fonction. 
        1. Sélectionnez l’application de fonction.
        1. Sélectionnez l’emplacement de déploiement. 
        1. Sélectionnez la fonction **EventGridTriggerMigrateData**. 
    1. Dans la page **Sélectionner une fonction Azure**, sélectionnez **Confirmer la sélection**.
    1. Ensuite, dans la page **Créer un abonnement aux événements**, sélectionnez **Créer**. 
    
        :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/event-subscription-select-function.png" alt-text="Créer un abonnement aux événements à l’aide de la fonction" lightbox="media/event-grid-event-hubs-functions-synapse-analytics/event-subscription-select-function.png":::
1. Vérifiez que l’abonnement aux événements a été créé. Accédez à l’onglet **Abonnements aux événements** dans la page **Événements** de l’espace de noms Event Hubs. 
    
    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/confirm-event-subscription.png" alt-text="Confirmer l’abonnement aux événements" lightbox="media/event-grid-event-hubs-functions-synapse-analytics/confirm-event-subscription.png":::
1. Sélectionnez le plan App Service (et non App Service) dans la liste des ressources du groupe de ressources. 

## <a name="run-the-app-to-generate-data"></a>Exécuter l’application pour générer des données
Vous avez terminé de configurer le hub d’événements, le pool SQL dédié (anciennement SQL Data Warehouse), l’application de fonction Azure et l’abonnement aux événements. Avant d’exécuter une application qui génère des données pour le hub d’événements, vous devez configurer certaines valeurs.

1. Dans le portail Azure, accédez à votre groupe de ressources comme précédemment. 
2. Sélectionner l’espace de noms Event Hubs.
3. Sur la page **Espace de noms Event Hubs**, cliquez sur **Stratégies d’accès partagé** dans le menu de gauche.
4. Sélectionnez **RootManageSharedAccessKey** dans la liste des stratégies. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/event-hub-namespace-shared-access-policies.png" alt-text="Page Stratégies d’accès partagé pour un espace de noms Event Hubs":::    
1. Cliquez sur le bouton copier à côté de la zone de texte **Clé primaire de la chaîne de connexion**. 
1. Retournez à votre solution Visual Studio. 
1. Cliquez avec le bouton droit sur le projet **WindTurbineDataGenerator**, puis sélectionnez **Définir comme projet de démarrage**. 
1. Dans le projet WindTurbineDataGenerator, ouvrez **program.cs**.
1. Remplacez `<EVENT HUBS NAMESPACE CONNECTION STRING>` par la chaîne de connexion que vous avez copiée du portail. 
1. Remplacez `<EVENT HUB NAME>` par le nom du hub d’événements. 

   ```cs
   private const string EventHubConnectionString = "Endpoint=sb://demomigrationnamespace.servicebus.windows.net/...";
   private const string EventHubName = "hubdatamigration";
   ```
6. Générez la solution. Exécutez l’application **WindTurbineGenerator.exe**. 
7. Après quelques minutes, dans l’autre onglet de navigateur où la fenêtre de requête est ouverte, interrogez la table dans votre entrepôt de données pour obtenir les données migrées.

    ```sql
    select * from [dbo].[Fact_WindTurbineMetrics]    
    ```

    ![Résultats de la requête](media/event-grid-event-hubs-functions-synapse-analytics/query-results.png)

## <a name="monitor-the-solution"></a>Surveiller la solution
Cette section vous montre comment superviser la solution ou résoudre les problèmes que vous rencontrez avec elle. 

### <a name="view-captured-data-in-the-storage-account"></a>Afficher les données capturées dans le compte de stockage
1. Accédez au groupe de ressources et sélectionnez le compte de stockage utilisé pour capturer les données d’événement. 
1. Dans la page **Compte de stockage**, sélectionnez **Explorateur Stockage (préversion)** dans le menu de gauche.
1. Développez **CONTENEURS BLOB**, puis sélectionnez **windturbinecapture**. 
1. Ouvrez le dossier du même nom que votre **espace de noms Event Hubs** dans le volet droit. 
1. Ouvrez le dossier du même nom que votre hub d’événements (**hubdatamigration**). 
1. Explorez les dossiers pour trouver les fichiers AVRO. Voici un exemple :

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/storage-captured-file.png" alt-text="Fichier capturé dans le stockage" lightbox="media/event-grid-event-hubs-functions-synapse-analytics/storage-captured-file.png":::
    

### <a name="verify-that-the-event-grid-trigger-invoked-the-function"></a>Vérifier que le déclencheur Event Grid a appelé la fonction
1. Accédez au groupe de ressources et sélectionnez l’application de fonction. 
1. Sélectionnez **Fonctions** dans le menu de gauche.
1. Sélectionnez la fonction **EventGridTriggerMigrateData** dans la liste. 
1. Dans la page **Fonction**, sélectionnez **Superviser** dans le menu de gauche. 
1. Sélectionnez **Configurer** pour configurer Application Insights afin de capturer les journaux d’appels. 
1. Créez une ressource **Application Insights** ou utilisez une ressource existante. 
1. Revenez à la page **Superviser** pour la fonction. 
1. Vérifiez que l’application cliente (**WindTurbineDataGenerator**) qui envoie les événements est toujours en cours d’exécution. Si ce n’est pas le cas, réexécutez-la. 
1. Patientez quelques minutes (au moins 5) et sélectionnez le bouton **Actualiser** pour voir les appels de fonction.    

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/function-invocations.png" alt-text="Appels de fonction":::
1. Sélectionnez un appel pour en voir les détails.

    Event Grid distribue des données d’événement pour les abonnés. L’exemple suivant montre les données d’événement générées lorsque des données en diffusion via un hub d’événements sont capturées dans un objet blob. Notez en particulier que la propriété `fileUrl` dans l’objet `data` pointe vers l’objet blob dans le stockage. L’application de fonction utilise cette URL pour récupérer le fichier blob avec des données capturées.

    ```json
    {
        "topic": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourcegroups/rgDataMigration/providers/Microsoft.EventHub/namespaces/spehubns1207",
        "subject": "hubdatamigration",
        "eventType": "Microsoft.EventHub.CaptureFileCreated",
        "id": "4538f1a5-02d8-4b40-9f20-36301ac976ba",
        "data": {
            "fileUrl": "https://spehubstorage1207.blob.core.windows.net/windturbinecapture/spehubns1207/hubdatamigration/0/2020/12/07/21/49/12.avro",
            "fileType": "AzureBlockBlob",
            "partitionId": "0",
            "sizeInBytes": 473444,
            "eventCount": 2800,
            "firstSequenceNumber": 55500,
            "lastSequenceNumber": 58299,
            "firstEnqueueTime": "2020-12-07T21:49:12.556Z",
            "lastEnqueueTime": "2020-12-07T21:50:11.534Z"
        },
        "dataVersion": "1",
        "metadataVersion": "1",
        "eventTime": "2020-12-07T21:50:12.7065524Z"
    }
    ```

### <a name="verify-that-the-data-is-stored-in-the-dedicated-sql-pool"></a>Vérifier que les données sont stockées dans le pool SQL dédié
Sous l’onglet du navigateur où la fenêtre de requête est ouverte, interrogez la table dans votre pool SQL dédié pour obtenir les données migrées.

![Résultats de la requête](media/event-grid-event-hubs-functions-synapse-analytics/query-results.png)

