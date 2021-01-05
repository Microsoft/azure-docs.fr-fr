---
title: 'Tutoriel : Utiliser des fonctions Java avec Azure Cosmos DB et Event Hubs'
description: Ce didacticiel explique comment utiliser des événements d'Event Hubs pour effectuer des mises à jour dans Azure Cosmos DB à l'aide d'une fonction écrite en Java.
author: KarlErickson
ms.topic: tutorial
ms.date: 11/04/2019
ms.author: karler
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: 20792d58ab259f93d7725fbafda1507f9eddc740
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96862154"
---
# <a name="tutorial-create-a-function-in-java-with-an-event-hub-trigger-and-an-azure-cosmos-db-output-binding"></a>Tutoriel : Créer une fonction en Java avec un déclencheur Event Hub et une liaison de sortie Azure Cosmos DB

Ce didacticiel explique comment utiliser Azure Functions pour créer une fonction Java qui analyse un flux continu de données de température et de pression. Les événements Event Hubs qui représentent les relevés du capteur déclenchent la fonction. La fonction traite les données d'événement, puis ajoute des entrées d'état à une instance d'Azure Cosmos DB.

Dans ce didacticiel, vous allez :

> [!div class="checklist"]
> * Gérer et configurer des ressources Azure à l'aide de l'interface de ligne de commande Azure (Azure CLI)
> * Créer et tester des fonctions Java qui interagissent avec ces ressources
> * Déployer vos fonctions sur Azure et les surveiller à l'aide d'Application Insights

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous devez installer les ressources suivantes :

- [Java Developer Kit (JDK)](/azure/developer/java/fundamentals/java-jdk-long-term-support), version 8
- [Apache Maven](https://maven.apache.org), version 3.0 ou ultérieure
- [Azure Functions Core Tools](https://www.npmjs.com/package/azure-functions-core-tools), version 2.6.666 ou supérieure [!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

> [!IMPORTANT]
> Pour suivre ce didacticiel, vous devez définir la variable d'environnement `JAVA_HOME` sur l'emplacement d'installation du JDK.

Si vous préférez utiliser directement le code pour ce didacticiel, consultez le référentiel d'exemples [java-functions-eventhub-cosmosdb](https://github.com/Azure-Samples/java-functions-eventhub-cosmosdb).

## <a name="create-azure-resources"></a>Créer des ressources Azure

Dans ce didacticiel, vous aurez besoin des ressources suivantes :

* Un groupe de ressources pour contenir les autres ressources
* Un espace de noms Event Hubs, un hub d'événements et une règle d'autorisation
* Un compte, une base de données et une collection Cosmos DB
* Une application de fonction et un compte de stockage pour l'héberger

Les sections suivantes expliquent comment créer ces ressources à l'aide de l'interface de ligne de commande Azure (Azure CLI).

### <a name="set-environment-variables"></a>Définir des variables d’environnement

Créez ensuite des variables d'environnement pour les noms et l'emplacement des ressources que vous allez créer. Utilisez les commandes suivantes, en remplaçant les espaces réservés `<value>` par les valeurs de votre choix. Les valeurs doivent être conformes aux [règles de nommage et aux restrictions applicables aux ressources Azure](/azure/architecture/best-practices/resource-naming). Pour la variable `LOCATION`, utilisez l'une des valeurs produites par la commande `az functionapp list-consumption-locations`.

# <a name="bash"></a>[Bash](#tab/bash)

```Bash
RESOURCE_GROUP=<value>
EVENT_HUB_NAMESPACE=<value>
EVENT_HUB_NAME=<value>
EVENT_HUB_AUTHORIZATION_RULE=<value>
COSMOS_DB_ACCOUNT=<value>
STORAGE_ACCOUNT=<value>
FUNCTION_APP=<value>
LOCATION=<value>
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
set RESOURCE_GROUP=<value>
set EVENT_HUB_NAMESPACE=<value>
set EVENT_HUB_NAME=<value>
set EVENT_HUB_AUTHORIZATION_RULE=<value>
set COSMOS_DB_ACCOUNT=<value>
set STORAGE_ACCOUNT=<value>
set FUNCTION_APP=<value>
set LOCATION=<value>
```

---

Ces variables seront utilisées dans la suite de ce didacticiel. Sachez que ces variables ne persistent que pendant la durée de votre session Azure CLI ou Cloud Shell actuelle. Vous devrez réexécuter ces commandes si vous utilisez une autre fenêtre de terminal locale ou si votre session Cloud Shell expire.

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Azure utilise des groupes de ressources pour collecter toutes les ressources associées dans votre compte. Vous disposez ainsi d'une visibilité globale sur celles-ci et pouvez les supprimer à l'aide d'une seule commande lorsque vous n'en avez plus besoin.

Utilisez la commande suivante pour créer un groupe de ressources :

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli-interactive
az group create \
    --name $RESOURCE_GROUP \
    --location $LOCATION
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
az group create ^
    --name %RESOURCE_GROUP% ^
    --location %LOCATION%
```

---

### <a name="create-an-event-hub"></a>Créer un hub d’événements

Créez ensuite un espace de noms Azure Event Hubs, un hub d'événements et une règle d'autorisation à l'aide des commandes suivantes :

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli-interactive
az eventhubs namespace create \
    --resource-group $RESOURCE_GROUP \
    --name $EVENT_HUB_NAMESPACE
az eventhubs eventhub create \
    --resource-group $RESOURCE_GROUP \
    --name $EVENT_HUB_NAME \
    --namespace-name $EVENT_HUB_NAMESPACE \
    --message-retention 1
az eventhubs eventhub authorization-rule create \
    --resource-group $RESOURCE_GROUP \
    --name $EVENT_HUB_AUTHORIZATION_RULE \
    --eventhub-name $EVENT_HUB_NAME \
    --namespace-name $EVENT_HUB_NAMESPACE \
    --rights Listen Send
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
az eventhubs namespace create ^
    --resource-group %RESOURCE_GROUP% ^
    --name %EVENT_HUB_NAMESPACE%
az eventhubs eventhub create ^
    --resource-group %RESOURCE_GROUP% ^
    --name %EVENT_HUB_NAME% ^
    --namespace-name %EVENT_HUB_NAMESPACE% ^
    --message-retention 1
az eventhubs eventhub authorization-rule create ^
    --resource-group %RESOURCE_GROUP% ^
    --name %EVENT_HUB_AUTHORIZATION_RULE% ^
    --eventhub-name %EVENT_HUB_NAME% ^
    --namespace-name %EVENT_HUB_NAMESPACE% ^
    --rights Listen Send
```

---

L'espace de noms Event Hubs contient le hub d'événements réel et sa règle d'autorisation. La règle d'autorisation permet à vos fonctions d'envoyer des messages au hub et d'écouter les événements correspondants. Une fonction envoie des messages qui représentent des données de télémétrie. Une autre fonction écoute les événements, analyse leurs données et stocke les résultats dans Azure Cosmos DB.

### <a name="create-an-azure-cosmos-db"></a>Créer une base de données Azure Cosmos DB

Créez ensuite un compte, une base de données et une collection Azure Cosmos DB à l'aide des commandes suivantes :

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli-interactive
az cosmosdb create \
    --resource-group $RESOURCE_GROUP \
    --name $COSMOS_DB_ACCOUNT
az cosmosdb sql database create \
    --resource-group $RESOURCE_GROUP \
    --account-name $COSMOS_DB_ACCOUNT \
    --name TelemetryDb
az cosmosdb sql container create \
    --resource-group $RESOURCE_GROUP \
    --account-name $COSMOS_DB_ACCOUNT \
    --database-name TelemetryDb \
    --name TelemetryInfo \
    --partition-key-path '/temperatureStatus'
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
az cosmosdb create ^
    --resource-group %RESOURCE_GROUP% ^
    --name %COSMOS_DB_ACCOUNT%
az cosmosdb sql database create ^
    --resource-group %RESOURCE_GROUP% ^
    --account-name %COSMOS_DB_ACCOUNT% ^
    --name TelemetryDb
az cosmosdb sql container create ^
    --resource-group %RESOURCE_GROUP% ^
    --account-name %COSMOS_DB_ACCOUNT% ^
    --database-name TelemetryDb ^
    --name TelemetryInfo ^
    --partition-key-path "/temperatureStatus"
```

---

La valeur `partition-key-path` partitionne vos données en fonction de la valeur `temperatureStatus` de chaque élément. La clé de partition permet à Cosmos DB d'améliorer les performances en divisant vos données en sous-ensembles distincts accessibles de façon indépendante.

### <a name="create-a-storage-account-and-function-app"></a>Créer un compte de stockage et une application de fonction

Créez ensuite un compte Stockage Azure, requis par Azure Functions, puis créez l'application de fonction. Utilisez les commandes suivantes :

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli-interactive
az storage account create \
    --resource-group $RESOURCE_GROUP \
    --name $STORAGE_ACCOUNT \
    --sku Standard_LRS
az functionapp create \
    --resource-group $RESOURCE_GROUP \
    --name $FUNCTION_APP \
    --storage-account $STORAGE_ACCOUNT \
    --consumption-plan-location $LOCATION \
    --runtime java \
    --functions-version 2
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
az storage account create ^
    --resource-group %RESOURCE_GROUP% ^
    --name %STORAGE_ACCOUNT% ^
    --sku Standard_LRS
az functionapp create ^
    --resource-group %RESOURCE_GROUP% ^
    --name %FUNCTION_APP% ^
    --storage-account %STORAGE_ACCOUNT% ^
    --consumption-plan-location %LOCATION% ^
    --runtime java ^
    --functions-version 2
```

---

Lorsque la commande `az functionapp create` crée votre application de fonction, elle crée également une ressource Application Insights du même nom. L'application de fonction est automatiquement configurée avec un paramètre nommé `APPINSIGHTS_INSTRUMENTATIONKEY` qui la connecte à Application Insights. Vous pouvez afficher les données de télémétrie de l'application après avoir déployé vos fonctions sur Azure, comme décrit plus loin dans ce didacticiel.

## <a name="configure-your-function-app"></a>Configurer votre application de fonction

Votre application de fonction doit avoir accès aux autres ressources pour fonctionner correctement. Les sections suivantes vous expliquent comment configurer votre application de fonction afin qu'elle puisse s'exécuter sur votre ordinateur local.

### <a name="retrieve-resource-connection-strings"></a>Récupérer les chaînes de connexion aux ressources

Utilisez les commandes suivantes pour récupérer les chaînes de connexion de stockage, de hub d'événements et de base de données Cosmos DB, et les enregistrer dans les variables d'environnement :

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli-interactive
AZURE_WEB_JOBS_STORAGE=$( \
    az storage account show-connection-string \
        --name $STORAGE_ACCOUNT \
        --query connectionString \
        --output tsv)
echo $AZURE_WEB_JOBS_STORAGE
EVENT_HUB_CONNECTION_STRING=$( \
    az eventhubs eventhub authorization-rule keys list \
        --resource-group $RESOURCE_GROUP \
        --name $EVENT_HUB_AUTHORIZATION_RULE \
        --eventhub-name $EVENT_HUB_NAME \
        --namespace-name $EVENT_HUB_NAMESPACE \
        --query primaryConnectionString \
        --output tsv)
echo $EVENT_HUB_CONNECTION_STRING
COSMOS_DB_CONNECTION_STRING=$( \
    az cosmosdb keys list \
        --resource-group $RESOURCE_GROUP \
        --name $COSMOS_DB_ACCOUNT \
        --type connection-strings \
        --query 'connectionStrings[0].connectionString' \
        --output tsv)
echo $COSMOS_DB_CONNECTION_STRING
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
FOR /F "delims=" %X IN (' ^
    az storage account show-connection-string ^
        --name %STORAGE_ACCOUNT% ^
        --query connectionString ^
        --output tsv') DO SET AZURE_WEB_JOBS_STORAGE=%X
FOR /F "delims=" %X IN (' ^
    az eventhubs eventhub authorization-rule keys list ^
        --resource-group %RESOURCE_GROUP% ^
        --name %EVENT_HUB_AUTHORIZATION_RULE% ^
        --eventhub-name %EVENT_HUB_NAME% ^
        --namespace-name %EVENT_HUB_NAMESPACE% ^
        --query primaryConnectionString ^
        --output tsv') DO SET EVENT_HUB_CONNECTION_STRING=%X
FOR /F "delims=" %X IN (' ^
    az cosmosdb keys list ^
        --resource-group %RESOURCE_GROUP% ^
        --name %COSMOS_DB_ACCOUNT% ^
        --type connection-strings ^
        --query connectionStrings[0].connectionString ^
        --output tsv') DO SET COSMOS_DB_CONNECTION_STRING=%X
```

---

Ces variables sont définies sur des valeurs récupérées à partir de commandes Azure CLI. Chaque commande utilise une requête JMESPath pour extraire la chaîne de connexion à partir de la charge JSON renvoyée. Les chaînes de connexion sont également affichées à l'aide de `echo` pour vous permettre de vérifier qu'elles ont bien été récupérées.

### <a name="update-your-function-app-settings"></a>Mettre à jour les paramètres de votre application de fonction

Utilisez ensuite la commande suivante pour transmettre les valeurs de chaîne de connexion aux paramètres de l'application sur votre compte Azure Functions :

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli-interactive
az functionapp config appsettings set \
    --resource-group $RESOURCE_GROUP \
    --name $FUNCTION_APP \
    --settings \
        AzureWebJobsStorage=$AZURE_WEB_JOBS_STORAGE \
        EventHubConnectionString=$EVENT_HUB_CONNECTION_STRING \
        CosmosDBConnectionString=$COSMOS_DB_CONNECTION_STRING
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
az functionapp config appsettings set ^
    --resource-group %RESOURCE_GROUP% ^
    --name %FUNCTION_APP% ^
    --settings ^
        AzureWebJobsStorage=%AZURE_WEB_JOBS_STORAGE% ^
        EventHubConnectionString=%EVENT_HUB_CONNECTION_STRING% ^
        CosmosDBConnectionString=%COSMOS_DB_CONNECTION_STRING%
```

---

Vos ressources Azure ont été créées et configurées pour fonctionner correctement ensemble.

## <a name="create-and-test-your-functions"></a>Créer et tester vos fonctions

Vous allez ensuite créer un projet sur votre ordinateur local, ajouter du code Java et le tester. Vous allez utiliser des commandes qui fonctionnent avec le plug-in Azure Functions pour Maven et Azure Functions Core Tools. Vos fonctions seront exécutées localement, mais elles utiliseront les ressources cloud que vous avez créées. Lorsque les fonctions sont opérationnelles localement, vous pouvez utiliser Maven pour les déployer dans le cloud. Vous verrez alors vos données et analyses s'accumuler.

Si vous avez utilisé Cloud Shell pour créer vos ressources, vous ne serez pas connecté à Azure localement. Dans ce cas, utilisez la commande `az login` pour lancer le processus de connexion basé sur un navigateur. Puis, si nécessaire, définissez l'abonnement par défaut sui `az account set --subscription`, suivi de l'ID de l'abonnement. Enfin, exécutez les commandes suivantes pour recréer des variables d'environnement sur votre ordinateur local. Remplacez les espaces réservés `<value>` par les valeurs que vous avez utilisées précédemment.

# <a name="bash"></a>[Bash](#tab/bash)

```Bash
RESOURCE_GROUP=<value>
FUNCTION_APP=<value>
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
set RESOURCE_GROUP=<value>
set FUNCTION_APP=<value>
```

---

### <a name="create-a-local-functions-project"></a>Créer un projet de fonctions local

Utilisez la commande Maven suivante pour créer un projet Functions et ajouter les dépendances requises.

# <a name="bash"></a>[Bash](#tab/bash)

```Bash
mvn archetype:generate --batch-mode \
    -DarchetypeGroupId=com.microsoft.azure \
    -DarchetypeArtifactId=azure-functions-archetype \
    -DappName=$FUNCTION_APP \
    -DresourceGroup=$RESOURCE_GROUP \
    -DgroupId=com.example \
    -DartifactId=telemetry-functions
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
mvn archetype:generate --batch-mode ^
    -DarchetypeGroupId=com.microsoft.azure ^
    -DarchetypeArtifactId=azure-functions-archetype ^
    -DappName=%FUNCTION_APP% ^
    -DresourceGroup=%RESOURCE_GROUP% ^
    -DgroupId=com.example ^
    -DartifactId=telemetry-functions
```

---

Cette commande génère plusieurs fichiers dans un dossier `telemetry-functions` :

* Un fichier `pom.xml` à utiliser avec Maven
* Un fichier `local.settings.json` contenant les paramètres de l'application pour le test local
* Un fichier `host.json` qui active le groupe d'extensions Azure Functions, requis pour la liaison de sortie Cosmos DB dans votre fonction d'analyse de données
* Un fichier `Function.java` incluant une implémentation de fonction par défaut
* Quelques fichiers de test dont ce didacticiel n'a pas besoin

Pour éviter les erreurs de compilation, vous devez supprimer les fichiers de test. Exécutez les commandes suivantes pour accéder au dossier du nouveau projet et supprimer le dossier de test :

# <a name="bash"></a>[Bash](#tab/bash)

```Bash
cd telemetry-functions
rm -r src/test
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
cd telemetry-functions
rmdir /s /q src\test
```

---

### <a name="retrieve-your-function-app-settings-for-local-use"></a>Récupérer les paramètres de votre application de fonction en vue d'une utilisation locale

Pour les tests locaux, votre projet de fonction aura besoin des chaînes de connexion que vous avez précédemment ajoutées à votre application de fonction dans Azure. Utilisez la commande Azure Functions Core Tools suivante, qui permet de récupérer tous les paramètres de l'application de fonction stockés dans le cloud et de les ajouter à votre fichier `local.settings.json` :

# <a name="bash"></a>[Bash](#tab/bash)

```Bash
func azure functionapp fetch-app-settings $FUNCTION_APP
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
func azure functionapp fetch-app-settings %FUNCTION_APP%
```

---

### <a name="add-java-code"></a>Ajouter du code Java

Ouvrez ensuite le fichier `Function.java` et remplacez son contenu par le code suivant.

```java
package com.example;

import com.example.TelemetryItem.status;
import com.microsoft.azure.functions.annotation.Cardinality;
import com.microsoft.azure.functions.annotation.CosmosDBOutput;
import com.microsoft.azure.functions.annotation.EventHubOutput;
import com.microsoft.azure.functions.annotation.EventHubTrigger;
import com.microsoft.azure.functions.annotation.FunctionName;
import com.microsoft.azure.functions.annotation.TimerTrigger;
import com.microsoft.azure.functions.ExecutionContext;
import com.microsoft.azure.functions.OutputBinding;

public class Function {

    @FunctionName("generateSensorData")
    @EventHubOutput(
        name = "event",
        eventHubName = "", // blank because the value is included in the connection string
        connection = "EventHubConnectionString")
    public TelemetryItem generateSensorData(
        @TimerTrigger(
            name = "timerInfo",
            schedule = "*/10 * * * * *") // every 10 seconds
            String timerInfo,
        final ExecutionContext context) {

        context.getLogger().info("Java Timer trigger function executed at: "
            + java.time.LocalDateTime.now());
        double temperature = Math.random() * 100;
        double pressure = Math.random() * 50;
        return new TelemetryItem(temperature, pressure);
    }

    @FunctionName("processSensorData")
    public void processSensorData(
        @EventHubTrigger(
            name = "msg",
            eventHubName = "", // blank because the value is included in the connection string
            cardinality = Cardinality.ONE,
            connection = "EventHubConnectionString")
            TelemetryItem item,
        @CosmosDBOutput(
            name = "databaseOutput",
            databaseName = "TelemetryDb",
            collectionName = "TelemetryInfo",
            connectionStringSetting = "CosmosDBConnectionString")
            OutputBinding<TelemetryItem> document,
        final ExecutionContext context) {

        context.getLogger().info("Event hub message received: " + item.toString());

        if (item.getPressure() > 30) {
            item.setNormalPressure(false);
        } else {
            item.setNormalPressure(true);
        }

        if (item.getTemperature() < 40) {
            item.setTemperatureStatus(status.COOL);
        } else if (item.getTemperature() > 90) {
            item.setTemperatureStatus(status.HOT);
        } else {
            item.setTemperatureStatus(status.WARM);
        }

        document.setValue(item);
    }
}
```

Comme vous pouvez le constater, ce fichier contient deux fonctions : `generateSensorData` et `processSensorData`. La fonction `generateSensorData` simule un capteur qui envoie des relevés de température et de pression au hub d'événements. Un déclencheur exécute la fonction toutes les 10 secondes et une liaison de sortie du hub d'événements transmet la valeur renvoyée au hub d'événements.

Lorsque le hub d'événements reçoit le message, il génère un événement. La fonction `processSensorData` s'exécute lorsqu'il reçoit l'événement. Il traite ensuite les données d'événement et utilise une liaison de sortie Azure Cosmos DB pour envoyer les résultats à Azure Cosmos DB.

Les données utilisées par ces fonctions sont stockées à l'aide d'une classe appelée `TelemetryItem`, que vous devez implémenter. Créez un nouveau fichier appelé `TelemetryItem.java` au même emplacement que `Function.java`, et ajoutez le code suivant :

```java
package com.example;

public class TelemetryItem {

    private String id;
    private double temperature;
    private double pressure;
    private boolean isNormalPressure;
    private status temperatureStatus;
    static enum status {
        COOL,
        WARM,
        HOT
    }

    public TelemetryItem(double temperature, double pressure) {
        this.temperature = temperature;
        this.pressure = pressure;
    }

    public String getId() {
        return id;
    }

    public double getTemperature() {
        return temperature;
    }

    public double getPressure() {
        return pressure;
    }

    @Override
    public String toString() {
        return "TelemetryItem={id=" + id + ",temperature="
            + temperature + ",pressure=" + pressure + "}";
    }

    public boolean isNormalPressure() {
        return isNormalPressure;
    }

    public void setNormalPressure(boolean isNormal) {
        this.isNormalPressure = isNormal;
    }

    public status getTemperatureStatus() {
        return temperatureStatus;
    }

    public void setTemperatureStatus(status temperatureStatus) {
        this.temperatureStatus = temperatureStatus;
    }
}
```

### <a name="run-locally"></a>Exécution locale

Vous pouvez maintenant générer et exécuter les fonctions localement, et voir les données s'afficher dans votre instance d'Azure Cosmos DB.

Utilisez les commandes Maven suivantes pour générer et exécuter les fonctions :

# <a name="bash"></a>[Bash](#tab/bash)

```Bash
mvn clean package
mvn azure-functions:run
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
mvn clean package
mvn azure-functions:run
```

---

Après quelques messages relatifs à la génération et au démarrage, une sortie semblable à l'exemple suivant s'affiche à chaque exécution des fonctions :

```output
[10/22/19 4:01:30 AM] Executing 'Functions.generateSensorData' (Reason='Timer fired at 2019-10-21T21:01:30.0016769-07:00', Id=c1927c7f-4f70-4a78-83eb-bc077d838410)
[10/22/19 4:01:30 AM] Java Timer trigger function executed at: 2019-10-21T21:01:30.015
[10/22/19 4:01:30 AM] Function "generateSensorData" (Id: c1927c7f-4f70-4a78-83eb-bc077d838410) invoked by Java Worker
[10/22/19 4:01:30 AM] Executed 'Functions.generateSensorData' (Succeeded, Id=c1927c7f-4f70-4a78-83eb-bc077d838410)
[10/22/19 4:01:30 AM] Executing 'Functions.processSensorData' (Reason='', Id=f4c3b4d7-9576-45d0-9c6e-85646bb52122)
[10/22/19 4:01:30 AM] Event hub message received: TelemetryItem={id=null,temperature=32.728691307527015,pressure=10.122563042388165}
[10/22/19 4:01:30 AM] Function "processSensorData" (Id: f4c3b4d7-9576-45d0-9c6e-85646bb52122) invoked by Java Worker
[10/22/19 4:01:38 AM] Executed 'Functions.processSensorData' (Succeeded, Id=1cf0382b-0c98-4cc8-9240-ee2a2f71800d)
```

Vous pouvez ensuite rejoindre le [portail Azure](https://portal.azure.com) et accéder à votre compte Azure Cosmos DB. Sélectionnez **Explorateur de données**, développez **TelemetryInfo**, puis sélectionnez **Éléments** pour afficher vos données lors de leur arrivée.

![Explorateur de données Cosmos DB](media/functions-event-hub-cosmos-db/data-explorer.png)

## <a name="deploy-to-azure-and-view-app-telemetry"></a>Déployer sur Azure et consulter les données de télémétrie de l'application

Enfin, vous pouvez déployer votre application sur Azure et vérifier qu'elle continue à fonctionner de la même manière qu'en local.

Déployez votre projet sur Azure à l'aide de la commande suivante :

# <a name="bash"></a>[Bash](#tab/bash)

```Bash
mvn azure-functions:deploy
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
mvn azure-functions:deploy
```

---

Vos fonctions s'exécutent désormais dans Azure, et continuent à accumuler des données dans votre instance d'Azure Cosmos DB. Vous pouvez voir votre application de fonction déployée sur le portail Azure, et consulter les données de télémétrie de l'application via la ressource Application Insights connectée, comme illustré dans les captures d'écran suivantes :

**Flux de métriques temps réel :**

![Flux de métriques temps réel Application Insights](media/functions-event-hub-cosmos-db/application-insights-live-metrics-stream.png)

**Performances :**

![Panneau Performances Application Insights](media/functions-event-hub-cosmos-db/application-insights-performance.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque les ressources Azure que vous avez créées dans ce didacticiel ne vous sont plus d'aucune utilité, vous pouvez les supprimer à l'aide de la commande suivante :

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli-interactive
az group delete --name $RESOURCE_GROUP
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
az group delete --name %RESOURCE_GROUP%
```

---

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à créer une fonction Azure qui gère les événements Event Hub et met à jour une instance de Cosmos DB. Pour plus d'informations, consultez le [Guide des développeurs Java sur Azure Functions](./functions-reference-java.md). Pour plus d'informations sur les annotations utilisées, consultez la référence [com.microsoft.azure.functions.annotation](/java/api/com.microsoft.azure.functions.annotation).

Ce didacticiel a utilisé des variables d'environnement et des paramètres d'application pour stocker des secrets tels que des chaînes de connexion. Pour plus d'informations sur le stockage de ces secrets dans Azure Key Vault, consultez [Utiliser des références Key Vault pour App Service et Azure Functions](../app-service/app-service-key-vault-references.md).

Apprenez ensuite à utiliser l'approche CI/CD Azure Pipelines dans le cadre d'un déploiement automatisé :

> [!div class="nextstepaction"]
> [Créer et déployer Java sur Azure Functions](/azure/devops/pipelines/ecosystems/java-function)
