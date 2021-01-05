---
title: Azure Event Hubs - Visualiser les anomalies des données dans les événements en temps réel
description: 'Tutoriel : Visualiser les anomalies des données dans les événements en temps réel envoyés à Microsoft Azure Event Hubs'
ms.topic: tutorial
ms.date: 06/23/2020
ms.openlocfilehash: b72b82f3959565e6bd0598fef8e21bb64fedb053
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97655677"
---
# <a name="tutorial-visualize-data-anomalies-in-real-time-events-sent-to-azure-event-hubs"></a>Tutoriel : Visualiser les anomalies des données dans les événements en temps réel envoyés à Azure Event Hubs

Avec Azure Event Hubs, vous pouvez utiliser Azure Stream Analytics pour vérifier les données entrantes et rejeter les anomalies, que vous pouvez ensuite visualiser dans Power BI. Supposons que vous avez des milliers d’appareils qui envoient en permanence des données en temps réel à un hub d’événements, ajoutant des millions d’événements par seconde. Comment trouver les anomalies ou les erreurs dans de telles quantités de données ? Par exemple, que se passe-t-il si les appareils envoient des transactions par carte de crédit, et que vous devez les capturer partout où vous devez gérer plusieurs transactions dans plusieurs pays/régions, dans un intervalle de 5 secondes ? Ce scénario peut se produire si quelqu’un vole des cartes de crédit et les utilise pour acheter des articles à travers le monde au même moment. 

Dans ce tutoriel, vous simulez cet exemple. Vous exécutez une application qui crée et envoie des transactions par carte de crédit à un hub d’événements. Vous lisez ensuite le flux de données en temps réel avec Azure Stream Analytics, qui sépare les transactions valides des transactions non valides, puis vous utilisez Power BI pour identifier visuellement les transactions qui sont marquées comme non valides.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Créer un espace de noms Event Hubs
> * Créer un hub d’événements
> * Exécuter l’application qui envoie des transactions par carte de crédit
> * Configurer une tâche Stream Analytics pour traiter ces transactions
> * Configurer une visualisation Power BI pour afficher les résultats

Pour suivre ce tutoriel, vous devez disposer d’un abonnement Azure. Si vous n’en avez pas, [créez un compte gratuit][] avant de commencer.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Installer [Visual Studio](https://www.visualstudio.com/). 
- Vous avez besoin d’un compte Power BI pour analyser la sortie d’une tâche Stream Analytics. Vous pouvez [essayer Power BI gratuitement](https://app.powerbi.com/signupredirect?pbi_source=web).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="set-up-resources"></a>Configurer des ressources

Pour ce tutoriel, vous avez besoin d’un espace de noms Event Hubs et d’un hub d’événements. Vous pouvez créer ces ressources avec Azure CLI ou Azure PowerShell. Utilisez le même groupe de ressources et le même emplacement pour toutes les ressources. Puis à la fin, vous pouvez supprimer tous les éléments en une seule fois en supprimant le groupe de ressources.

Les sections suivantes expliquent comment effectuer ces étapes nécessaires. Suivez les instructions destinées à l’interface CLI *ou* à PowerShell pour effectuer les étapes suivantes :

1. Créez un [groupe de ressources](../azure-resource-manager/management/overview.md). 

2. Créez un espace de noms Event Hubs. 

3. Créez un hub d’événements.

> [!NOTE]
> Des variables sont définies dans chaque script dont vous avez besoin plus loin dans le tutoriel. Celles-ci incluent le nom du groupe de ressources ($resourceGroup), l’espace de noms du hub d’événements ( **$eventHubNamespace**) et le nom du hub d’événements ( **$eventHubName**). Ils sont désignés avec leur préfixe (le signe dollar ($)) plus loin dans cet article, ce qui vous permet de savoir qu’ils ont été définis dans le script.

<!-- some day they will approve the tab control; 
  When that happens, put CLI and PSH in tabs. -->

### <a name="set-up-your-resources-using-azure-cli"></a>Configurer vos ressources avec Azure CLI

Copiez et collez ce script dans Cloud Shell. En supposant que vous êtes déjà connecté, il exécute le script, ligne par ligne.

Les variables qui doivent être globalement uniques sont concaténées avec la chaîne `$RANDOM`. Quand le script est exécuté et que les variables sont définies, une chaîne numérique aléatoire est générée et ajoutée à la fin de la chaîne fixe, la rendant ainsi unique.

```azurecli-interactive
# Set the values for location and resource group name.
location=westus
resourceGroup=ContosoResourcesEH

# Create the resource group to be used
#   for all the resources for this tutorial.
az group create --name $resourceGroup \
    --location $location

# The Event Hubs namespace name must be globally unique, so add a random number to the end.
eventHubNamespace=ContosoEHNamespace$RANDOM
echo "Event Hub Namespace = " $eventHubNamespace

# Create the Event Hubs namespace.
az eventhubs namespace create --resource-group $resourceGroup \
   --name $eventHubNamespace \
   --location $location \
   --sku Standard

# The event hub name must be globally unique, so add a random number to the end.
eventHubName=ContosoEHhub$RANDOM
echo "event hub name = " $eventHubName

# Create the event hub.
az eventhubs eventhub create --resource-group $resourceGroup \
    --namespace-name $eventHubNamespace \
    --name $eventHubName \
    --message-retention 3 \
    --partition-count 2

# Get the connection string that authenticates the app with the Event Hubs service.
connectionString=$(az eventhubs namespace authorization-rule keys list \
   --resource-group $resourceGroup \
   --namespace-name $eventHubNamespace \
   --name RootManageSharedAccessKey \
   --query primaryConnectionString \
   --output tsv)
echo "Connection string = " $connectionString 
```

### <a name="set-up-your-resources-using-azure-powershell"></a>Configurer vos ressources avec Azure PowerShell

Copiez et collez ce script dans Cloud Shell. En supposant que vous êtes déjà connecté, il exécute le script, ligne par ligne.

Les variables qui doivent être globalement uniques sont concaténées avec la chaîne `$(Get-Random)`. Quand le script est exécuté et que les variables sont définies, une chaîne numérique aléatoire est générée et ajoutée à la fin de la chaîne fixe, la rendant ainsi unique.

```azurepowershell-interactive
# Log in to Azure account.
Login-AzAccount

# Set the values for the location and resource group.
$location = "West US"
$resourceGroup = "ContosoResourcesEH"

# Create the resource group to be used  
#   for all resources for this tutorial.
New-AzResourceGroup -Name $resourceGroup -Location $location

# The Event Hubs namespace name must be globally unique, so add a random number to the end.
$eventHubNamespace = "contosoEHNamespace$(Get-Random)"
Write-Host "Event Hub Namespace is " $eventHubNamespace

# The event hub name must be globally unique, so add a random number to the end.
$eventHubName = "contosoEHhub$(Get-Random)"
Write-Host "Event hub Name is " $eventHubName

# Create the Event Hubs namespace.
New-AzEventHubNamespace -ResourceGroupName $resourceGroup `
     -NamespaceName $eventHubNamespace `
     -Location $location

# Create the event hub.
$yourEventHub = New-AzEventHub -ResourceGroupName $resourceGroup `
    -NamespaceName $eventHubNamespace `
    -Name $eventHubName `
    -MessageRetentionInDays 3 `
    -PartitionCount 2

# Get the event hub key, and retrieve the connection string from that object.
# You need this to run the app that sends test messages to the event hub.
$eventHubKey = Get-AzEventHubKey -ResourceGroupName $resourceGroup `
    -Namespace $eventHubNamespace `
    -AuthorizationRuleName RootManageSharedAccessKey

# Save this value somewhere local for later use.
Write-Host "Connection string is " $eventHubKey.PrimaryConnectionString
```

## <a name="run-app-to-produce-test-event-data"></a>Exécuter une application pour produire des données d’événements de test

Les [exemples Event Hubs sur GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet) incluent une application de détection d’anomalies qui produit des données de test pour vous. Elle simule l’utilisation de cartes de crédit en écrivant des transactions par carte de crédit dans le hub d’événements, en y incluant parfois l’écriture de plusieurs transactions pour la même carte de crédit à plusieurs emplacements, de sorte qu’elles soient marquées comme anomalies. Pour exécuter cette application, suivez ces étapes : 

1. Téléchargez les [exemples Azure Event Hubs](https://github.com/Azure/azure-event-hubs/archive/master.zip) depuis GitHub et décompressez-les localement.
2. Accédez au dossier **\azure-event-hubs-master\samples\DotNet\\** . 
3. Accédez au dossier **Azure.Messaging.EventHubs\AnomalyDetector\\** et double-cliquez sur **AnomalyDetector.sln** pour ouvrir la solution dans Visual Studio. 

    Pour utiliser l’ancienne version de l’exemple qui utilise l’ancien package Microsoft.Azure.EventHubs, ouvrez la solution à partir du dossier **Microsoft.Azure.EventHubs\AnomalyDetector**. 
3. Ouvrez le fichier Program.cs et remplacez **Event Hubs connection string** par la chaîne de connexion que vous avez enregistrée lors de l’exécution du script. 
4. Remplacez **Event Hub name** par le nom de votre hub d’événements. Appuyez sur F5 pour exécuter l’application. Elle commence à envoyer des événements à votre hub d’événements et continue jusqu’à atteindre 1 000 événements. Il existe quelques cas où l’application doit être en cours d’exécution pour que vous puissiez récupérer des données. Ces cas sont précisés dans les instructions suivantes, quand c’est nécessaire.

## <a name="set-up-azure-stream-analytics"></a>Configurer Azure Stream Analytics

Vous pouvez maintenant envoyer des données dans votre hub d’événements. Pour utiliser ces données dans une visualisation Power BI, commencez par configurer une tâche Stream Analytics pour récupérer les données qui sont ensuite transmises dans la visualisation Power BI.

### <a name="create-the-stream-analytics-job"></a>Créer la tâche Stream Analytics

1. Dans le portail Azure, cliquez sur **Créer une ressource**. Tapez **stream analytics** dans la zone de recherche et appuyez sur **Entrée**. Sélectionnez **Tâche Stream Analytics**. Cliquez sur **Créer** dans le volet de la tâche Stream Analytics. 

2. Entrez les informations suivantes pour la tâche :

   **Nom du travail** : utilisez **contosoEHjob**. Ce champ est le nom de la tâche et il doit être globalement unique.

   **Abonnement**: Sélectionnez votre abonnement.

   **Groupe de ressources** : utilisez le même groupe de ressources que celui utilisé par votre hub d’événements (**ContosoResourcesEH**).

   **Emplacement** : utilisez le même emplacement que celui utilisé dans le script de configuration (**USA Ouest**).

   ![Capture d’écran montrant comment créer une tâche Azure Stream Analytics.](./media/event-hubs-tutorial-visualize-anomalies/stream-analytics-add-job.png)

    Acceptez les valeurs par défaut pour les autres champs. Cliquez sur **Créer**. 

### <a name="add-an-input-to-the-stream-analytics-job"></a>Ajouter une entrée à la tâche Stream Analytics

Si vous n’êtes pas dans le volet **Tâche Stream Analytics** du portail, vous pouvez revenir à votre tâche Stream Analytics en cliquant sur **Groupes de ressources** dans le portail, puis en sélectionnant votre groupe de ressources ( **ContosoResourcesEH**). Cette action montre toutes les ressources du groupe, et vous pouvez ensuite sélectionner votre tâche Stream Analytics. 

Les entrées pour la tâche Stream Analytics sont les transactions par carte de crédit provenant du hub d’événements.

> [!NOTE]
> Les valeurs des variables commençant par le signe dollar ($) sont définies dans les scripts de démarrage des sections précédentes. Vous devez utiliser ici les mêmes valeurs quand vous spécifiez les champs, qui sont l’espace de noms Event Hubs et le nom du hub d’événements.

1. Sous **Topologie de la tâche**, cliquez sur **Entrées**.

2. Dans le volet **Entrées**, cliquez sur **Ajouter une entrée de flux** et sélectionnez Event Hubs. Dans l’écran qui apparaît, renseignez les champs suivants :

   **Alias d’entrée** : utilisez **contosoinputs**. Ce champ est le nom du flux d’entrée, utilisé lors de la définition de la requête pour les données.

   **Abonnement**: Sélectionnez votre abonnement.

   **Espace de noms Event Hubs** : sélectionnez l’espace de noms de votre hub d’événements ($**eventHubNamespace**). 

   **Nom du hub d’événements** : cliquez sur **Utiliser l’existant** et sélectionnez votre hub d’événements ($**eventHubName**).

   **Nom de la stratégie du hub d’événements** : Sélectionnez **RootManageSharedAccessKey**.

   **Groupe de consommateurs du hub d’événements**  : laissez ce champ vide pour utiliser le groupe de consommateurs par défaut.

   Acceptez les valeurs par défaut pour les autres champs.

   ![Capture d’écran montrant comment ajouter un flux d’entrée à la tâche Stream Analytics.](./media/event-hubs-tutorial-visualize-anomalies/stream-analytics-inputs.png)

5. Cliquez sur **Enregistrer**.

### <a name="add-an-output-to-the-stream-analytics-job"></a>Ajouter une sortie à la tâche Stream Analytics

1. Sous **Topologie de la tâche**, cliquez sur **Sorties**. Ce champ est le nom du flux de sortie, utilisé lors de la définition de la requête pour les données.

2. Dans le volet **Sorties**, cliquez sur **Ajouter**, puis sélectionnez **Power BI**. Dans l’écran qui apparaît, renseignez les champs suivants :

   **Alias de sortie** : utilisez **contosooutputs**. Ce champ est l’alias unique de la sortie. 

   **Nom du jeu de données** : utilisez **contosoehdataset**. Ce champ est le nom du jeu de données à utiliser dans Power BI. 

   **Nom de la table** : utilisez **contosoehtable**. Ce champ est le nom de la table à utiliser dans Power BI. 

   Acceptez les valeurs par défaut pour les autres champs.

   ![Capture d’écran montrant comment configurer la sortie pour une tâche Stream Analytics.](./media/event-hubs-tutorial-visualize-anomalies/stream-analytics-outputs.png)

3. Cliquez sur **Autoriser** et connectez-vous à votre compte Power BI.

4. Acceptez les valeurs par défaut pour les autres champs.

5. Cliquez sur **Enregistrer**.

### <a name="configure-the-query-of-the-stream-analytics-job"></a>Configurer la requête de la tâche Stream Analytics

Cette requête est utilisée pour récupérer les données qui sont envoyées au final à la visualisation Power BI. Elle utilise **contosoinputs** et **contosooutputs**, que vous avez définis précédemment lors de la configuration de la tâche. Cette requête récupère les transactions par carte de crédit estimées frauduleuses, qui sont des transactions où le même numéro de carte de crédit présente plusieurs transactions à différents endroits dans le même intervalle de cinq secondes.

1. Sous **Topologie de la tâche**, cliquez sur **Requête**.

2. Remplacez la requête par la suivante : 

   ```SQL
   /* criteria for fraud:
      credit card purchases with the same card
      in different locations within 5 seconds
   */
   SELECT System.Timestamp AS WindowEnd, 
     COUNT(*) as FraudulentUses      
   INTO contosooutputs
   FROM contosoinputs CS1 TIMESTAMP BY [Timestamp]
       JOIN contosoinputs CS2 TIMESTAMP BY [Timestamp]
       /* where the credit card # is the same */
       ON CS1.CreditCardId = CS2.CreditCardId
       /* and time between the two is between 0 and 5 seconds */
       AND DATEDIFF(second, CS1, CS2) BETWEEN 0 AND 5
       /* where the location is different */
   WHERE CS1.Location != CS2.Location
   GROUP BY TumblingWindow(Duration(second, 1))
   ```

4. Cliquez sur **Enregistrer**.

### <a name="test-the-query-for-the-stream-analytics-job"></a>Tester la requête pour la tâche Stream Analytics 

1. Exécutez l’application de détection d’anomalies pour envoyer des données au hub d’événements pendant que vous configurez et que vous exécutez le test. 

2. Dans le volet Requête, cliquez sur les points à côté de l’entrée **contosoinputs**, puis sélectionnez **Exemple de données de l’entrée**.

3. Spécifiez que vous voulez trois minutes de données, puis cliquez sur **OK**. Patientez jusqu’à ce que vous soyez informé de l’échantillonnage des données.

4. Cliquez sur **Test** et vérifiez que vous obtenez des résultats. Les résultats s’affichent dans la section **Résultats** du volet en bas à droite sous la requête.

5. Fermez le volet Requête.

### <a name="run-the-stream-analytics-job"></a>Exécuter la tâche Stream Analytics

Dans la tâche Stream Analytics, cliquez sur **Démarrer**, **Maintenant**, puis **Démarrer**. Une fois la tâche lancée, l’état correspondant passe de **Arrêté** à **Exécution**.

## <a name="set-up-the-power-bi-visualizations"></a>Configurer les visualisations Power BI

1. Exécutez l’application de détection d’anomalies pour envoyer des données au hub d’événements pendant que vous configurez la visualisation Power BI. Il peut être nécessaire de l’exécuter plusieurs fois, car elle génère seulement 1 000 transactions à chaque exécution.

2. Connectez-vous à votre compte [Power BI](https://powerbi.microsoft.com/).

3. Accédez à **Mon espace de travail**.

4. Cliquez sur **Jeux de données**.

   Vous devez voir le jeu de données que vous avez spécifié lors de la création de la sortie pour la tâche Stream Analytics (**contosoehdataset**). 5 à 10 minutes peuvent être nécessaires pour que le jeu de données apparaisse la première fois.

5. Cliquez sur **Tableaux de bord**, puis cliquez sur **Créer** et sélectionnez **Tableau de bord**.

   ![Capture d’écran des boutons Tableaux de bord et Créer.](./media/event-hubs-tutorial-visualize-anomalies/power-bi-add-dashboard.png)

6. Spécifiez le nom du tableau de bord, puis cliquez sur **Créer**. Utilisez **Anomalies de carte de crédit**.

   ![Capture d’écran de la spécification du nom du tableau de bord.](./media/event-hubs-tutorial-visualize-anomalies/power-bi-dashboard-name.png)

7. Dans la page Tableau de bord, cliquez sur **Ajouter une vignette**, sélectionnez **Données de streaming personnalisées** dans la section **DONNÉES EN TEMPS RÉEL**, puis cliquez sur **Suivant**.

   ![Capture d’écran de la spécification de la source pour la vignette.](./media/event-hubs-tutorial-visualize-anomalies/power-bi-add-card-real-time-data.png)

8. Sélectionnez votre jeu de données (**contosoehdataset**), puis cliquez sur **Suivant**.

   ![Capture d’écran de la spécification du jeu de données.](./media/event-hubs-tutorial-visualize-anomalies/power-bi-dashboard-select-dataset.png)

9. Sélectionnez **Carte** pour le type de visualisation. Sous **Champs**, cliquez sur **Ajouter une valeur**, puis sélectionnez `fraudulentuses`.

   ![Capture d’écran de la spécification du type de visualisation et des champs.](./media/event-hubs-tutorial-visualize-anomalies/power-bi-add-card-tile.png)

   Cliquez sur **Suivant**.

10. Spécifiez **Utilisations frauduleuses** comme titre et **Somme au cours des dernières minutes** comme sous-titre. Cliquez sur **Appliquer**. La vignette est enregistrée dans le tableau de bord.

    ![Capture d’écran de la spécification du titre et du sous-titre de la vignette du tableau de bord.](./media/event-hubs-tutorial-visualize-anomalies/power-bi-tile-details.png)

    > [!IMPORTANT]
    > Quand vous exécutez l’exemple d’application et que vous transmettez des données au hub d’événements, le nombre figurant sur cette vignette change rapidement (chaque seconde). En effet, la requête Stream Analytics met à jour la valeur **chaque seconde**. Mettez à jour la requête vers une fenêtre bascule de 3 minutes pour afficher la somme des dernières minutes. 
11. Ajoutez une autre visualisation. Répétez les premières étapes :

    * Cliquez sur **Ajouter une vignette**.
    * Sélectionnez **Données de streaming personnalisées**. 
    * Cliquez sur **Suivant**.
    * Sélectionnez votre jeu de données, puis cliquez **Suivant**. 

12. Sous **Type de visualisation**, sélectionnez **Graphique en courbes**.

13. Sous **Axe**, cliquez sur **Ajouter une valeur**, puis sélectionnez `windowend`. 

14. Sous **Valeurs**, cliquez sur **Ajouter une valeur**, puis sélectionnez `fraudulentuses`.

15. Sous **Fenêtre de temps à afficher**, sélectionnez les cinq dernières minutes. Cliquez sur **Suivant**.

16. Spécifiez **Montrer les utilisations frauduleuses au fil du temps** comme titre et laissez le sous-titre de la vignette vide, puis cliquez sur **Appliquer**. Vous revenez à votre tableau de bord.

17. Réexécutez l’application de détection d’anomalies pour envoyer des données au hub d’événements. Vous voyez que la vignette **Utilisations frauduleuses** change au fil de l’analyse les données et que le graphique en courbes affiche des données. 

    ![Capture d’écran montrant les résultats dans Power BI](./media/event-hubs-tutorial-visualize-anomalies/power-bi-results.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous voulez supprimer toutes les ressources que vous avez créées, supprimez les données de visualisation Power BI, puis supprimez le groupe de ressources. La suppression du groupe de ressources supprime toutes les ressources qu’il contient. Dans ce cas, il supprime le hub d’événements, l’espace de noms Event Hubs, la tâche Stream Analytics et le groupe de ressources lui-même. 

### <a name="clean-up-resources-in-the-power-bi-visualization"></a>Supprimer des ressources dans la visualisation Power BI

Connectez-vous à votre compte Power BI. Accédez à **Mon espace de travail**. Sur la ligne avec le nom de votre tableau de bord, cliquez sur l’icône Corbeille. Ensuite, accédez à **Jeux de données** et cliquez sur l’icône Corbeille pour supprimer le jeu de données (**contosoehdataset**).

### <a name="clean-up-resources-using-azure-cli"></a>Supprimer des ressources à l’aide d’Azure CLI

Pour supprimer le groupe de ressources, utilisez la commande [az group delete](/cli/azure/group?view=azure-cli-latest#az-group-delete).

```azurecli-interactive
az group delete --name $resourceGroup
```

### <a name="clean-up-resources-using-powershell"></a>Supprimer des ressources à l’aide de PowerShell

Pour supprimer le groupe de ressources, utilisez la commande [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```azurepowershell-interactive
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :
> [!div class="checklist"]
> * Créer un espace de noms Event Hubs
> * Créer un hub d’événements
> * Exécuter l’application qui simule les événements et les envoie au hub d’événements
> * Configurer une tâche Stream Analytics pour traiter les événements envoyés au hub
> * Configurer une visualisation Power BI pour afficher les résultats

Passez à l’article suivant pour découvrir plus d’informations sur Azure Event Hubs.

> [!div class="nextstepaction"]
> [Bien démarrer avec l’envoi de messages vers Azure Event Hubs dans .NET Standard](event-hubs-dotnet-standard-getstarted-send.md)

[créez un compte gratuit]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
