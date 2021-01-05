---
title: Utiliser la bibliothèque de client Azure Batch pour Node.js
description: Découvrez les concepts de base d’Azure Batch et créez une solution simple à l’aide de Node.js.
ms.topic: how-to
ms.date: 10/08/2020
ms.openlocfilehash: 8d34d5bbb302e3781aabdd697de11d3d492b879a
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97106697"
---
# <a name="get-started-with-batch-sdk-for-nodejs"></a>Bien démarrer avec le Kit de développement logiciel (SDK) Batch pour Node.js

Découvrez les concepts de base de création d’un client Batch dans Node.js à l’aide du [Kit de développement logiciel (SDK) Node.js pour Azure Batch](/javascript/api/overview/azure/batch). Nous allons présenter pas à pas un scénario pour une application Batch, puis la configurer à l’aide d’un client Node.js.

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous avez acquis une connaissance pratique de Node.js et que vous êtes familiarisé avec Linux. Il suppose également que vous disposez d’un compte Azure configuré avec des droits d’accès pour créer des services Batch et Stockage.

Nous vous recommandons de lire [Présentation technique d’Azure Batch](batch-technical-overview.md) avant d’effectuer les étapes présentées dans cet article.

## <a name="understand-the-scenario"></a>Présentation du scénario

Nous avons écrit ici un script simple dans Python qui permet de télécharger tous les fichiers csv à partir d’un conteneur de stockage Blob Azure et de les convertir au format JSON. Pour traiter plusieurs conteneurs du compte de stockage en parallèle, nous pouvons déployer le script en tant que travail Azure Batch.

## <a name="azure-batch-architecture"></a>Architecture Azure Batch

Le diagramme suivant illustre la façon dont nous pouvons mettre à l’échelle le script Python à l’aide d’Azure Batch et d’un client Node.js.

![Diagramme montrant l’architecture du scénario.](./media/batch-nodejs-get-started/BatchScenario.png)

Le client Node.js déploie un traitement par lots avec une tâche de préparation (expliquée en détail plus loin) ainsi qu’un ensemble de tâches en fonction du nombre de conteneurs du compte de stockage. Vous pouvez télécharger les scripts à partir du référentiel GitHub.

- [Client Node.js](https://github.com/Azure/azure-batch-samples/blob/master/Node.js/GettingStarted/nodejs_batch_client_sample.js)
- [Scripts Shell de la tâche de préparation](https://github.com/Azure/azure-batch-samples/blob/master/Node.js/GettingStarted/startup_prereq.sh)
- [Processeur de fichiers csv Python au format JSON](https://github.com/Azure/azure-batch-samples/blob/master/Node.js/GettingStarted/processcsv.py)

> [!TIP]
> Le client Node.js dans le lien spécifié ne contient pas de code spécifique pour être déployé comme application de fonction Azure. Vous pouvez vous reporter aux liens suivants pour connaître les instructions à suivre pour en créer une.
> - [Créer une application de fonction](../azure-functions/functions-create-first-azure-function.md)
> - [Créer une fonction de déclencheur de minuteur](../azure-functions/functions-bindings-timer.md)

## <a name="build-the-application"></a>Créer l’application

À présent, laissez-nous suivre la procédure étape par étape de création du client Node.js :

### <a name="step-1-install-azure-batch-sdk"></a>Étape 1 : installer le Kit de développement logiciel (SDK) Azure Batch

Vous pouvez installer le Kit de développement logiciel (SDK) Azure Batch pour Node.js à l’aide de la commande npm install.

`npm install azure-batch`

Cette commande installe la dernière version du Kit de développement logiciel (SDK) Azure Batch pour Node.js.

>[!Tip]
> Dans une application Azure Function App, vous pouvez accéder à « Kudu Console » dans l’onglet Paramètres de la fonction Azure pour exécuter les commandes npm install, dans ce cas, pour installer le Kit de développement logiciel (SDK) Azure Batch pour Node.js.

### <a name="step-2-create-an-azure-batch-account"></a>Étape 2 : Créer un compte Azure Batch

Vous pouvez le créer à partir du [portail Azure](batch-account-create-portal.md) ou de la ligne de commande ([PowerShell](batch-powershell-cmdlets-get-started.md) /[Azure CLI](/cli/azure)).

Ci-après figurent les commandes permettant d’en créer un via Azure CLI.

Créez un groupe de ressources. Ignorez cette étape si vous en avez déjà un là où vous souhaitez créer le compte Batch :

`az group create -n "<resource-group-name>" -l "<location>"`

Créez ensuite un compte Azure Batch.

`az batch account create -l "<location>"  -g "<resource-group-name>" -n "<batch-account-name>"`

Chaque compte Batch possède ses clés d’accès correspondantes. Ces clés sont nécessaires pour créer d’autres ressources dans un compte Azure Batch. Pour un environnement de production, il est conseillé d’utiliser Azure Key Vault pour stocker ces clés. Vous pouvez alors créer un principal de service pour l’application. En utilisant ce principal de service, l’application peut créer un jeton OAuth pour les clés d’accès dans le coffre de clés.

`az batch account keys list -g "<resource-group-name>" -n "<batch-account-name>"`

Copiez et stockez la clé à utiliser dans les étapes suivantes.

### <a name="step-3-create-an-azure-batch-service-client"></a>Étape 3 : créer un client du service Azure Batch

L’extrait de code suivant importe tout d’abord le module Node.js pour Azure Batch, puis crée un client du service Batch. Vous devez d’abord créer un objet SharedKeyCredentials avec la clé du compte Batch copiée à l’étape précédente.

```nodejs
// Initializing Azure Batch variables

var batch = require('azure-batch');

var accountName = '<azure-batch-account-name>';

var accountKey = '<account-key-downloaded>';

var accountUrl = '<account-url>'

// Create Batch credentials object using account name and account key

var credentials = new batch.SharedKeyCredentials(accountName,accountKey);

// Create Batch service client

var batch_client = new batch.ServiceClient(credentials,accountUrl);

```

L’URI Azure Batch se trouve dans l’onglet Vue d’ensemble du portail Azure. Son format est le suivant :

`https://accountname.location.batch.azure.com`

Reportez-vous à la capture d’écran :

![URI Azure Batch](./media/batch-nodejs-get-started/azurebatchuri.png)

### <a name="step-4-create-an-azure-batch-pool"></a>Étape 4 : créer un pool Azure Batch

Un pool Azure Batch se compose de plusieurs machines virtuelles (également appelées nœuds Batch). Le service Azure Batch déploie les tâches sur ces nœuds et les gère. Vous pouvez définir les paramètres de configuration suivants pour le pool.

- Type d’image de machine virtuelle
- Taille des nœuds de machine virtuelle
- Nombre de nœuds de machine virtuelle

> [!TIP]
> La taille et le nombre de nœuds de machine virtuelle dépendent en grande partie du nombre de tâches à exécuter en parallèle et également de la tâche proprement dite. Nous vous recommandons d’effectuer des tests pour déterminer la taille et le nombre idéaux.

L’extrait de code suivant crée les objets de paramètre de configuration.

```nodejs
// Creating Image reference configuration for Ubuntu Linux VM
var imgRef = {publisher:"Canonical",offer:"UbuntuServer",sku:"14.04.2-LTS",version:"latest"}

// Creating the VM configuration object with the SKUID
var vmconfig = {imageReference:imgRef,nodeAgentSKUId:"batch.node.ubuntu 14.04"}

// Setting the VM size to Standard F4
var vmSize = "STANDARD_F4"

//Setting number of VMs in the pool to 4
var numVMs = 4
```

> [!TIP]
> Pour obtenir la liste des images de machine virtuelle Linux disponibles pour Azure Batch et de leurs ID de référence (SKU), consultez la section [Liste des images de machine virtuelle](batch-linux-nodes.md#list-of-virtual-machine-images).

Une fois la configuration du pool définie, vous pouvez créer le pool Azure Batch. La commande du pool Batch crée des nœuds de machine virtuelle Azure et les prépare à recevoir les tâches à exécuter. Chaque pool doit posséder un ID unique à titre de référence pour les étapes suivantes.

L’extrait de code suivant crée un pool Azure Batch.

```nodejs
// Create a unique Azure Batch pool ID
var poolid = "pool" + customerDetails.customerid;
var poolConfig = {id:poolid, displayName:poolid,vmSize:vmSize,virtualMachineConfiguration:vmconfig,targetDedicatedComputeNodes:numVms,enableAutoScale:false };
// Creating the Pool for the specific customer
var pool = batch_client.pool.add(poolConfig,function(error,result){
    if(error!=null){console.log(error.response)};
});
```

Vous pouvez vérifier l’état du pool créé et vous assurer que l’état est « actif » avant de poursuivre avec la soumission d’un travail à ce pool.

```nodejs
var cloudPool = batch_client.pool.get(poolid,function(error,result,request,response){
        if(error == null)
        {

            if(result.state == "active")
            {
                console.log("Pool is active");
            }
        }
        else
        {
            if(error.statusCode==404)
            {
                console.log("Pool not found yet returned 404...");

            }
            else
            {
                console.log("Error occurred while retrieving pool data");
            }
        }
        });
```

Ci-après figure un exemple d’objet de résultat retourné par la fonction pool.get.

```
{ id: 'processcsv_201721152',
  displayName: 'processcsv_201721152',
  url: 'https://<batch-account-name>.centralus.batch.azure.com/pools/processcsv_201721152',
  eTag: '<eTag>',
  lastModified: 2017-03-27T10:28:02.398Z,
  creationTime: 2017-03-27T10:28:02.398Z,
  state: 'active',
  stateTransitionTime: 2017-03-27T10:28:02.398Z,
  allocationState: 'resizing',
  allocationStateTransitionTime: 2017-03-27T10:28:02.398Z,
  vmSize: 'standard_a1',
  virtualMachineConfiguration:
   { imageReference:
      { publisher: 'Canonical',
        offer: 'UbuntuServer',
        sku: '14.04.2-LTS',
        version: 'latest' },
     nodeAgentSKUId: 'batch.node.ubuntu 14.04' },
  resizeTimeout:
   { [Number: 900000]
     _milliseconds: 900000,
     _days: 0,
     _months: 0,
     _data:
      { milliseconds: 0,
        seconds: 0,
        minutes: 15,
        hours: 0,
        days: 0,
        months: 0,
        years: 0 },
     _locale:
      Locale {
        _calendar: [Object],
        _longDateFormat: [Object],
        _invalidDate: 'Invalid date',
        ordinal: [Function: ordinal],
        _ordinalParse: /\d{1,2}(th|st|nd|rd)/,
        _relativeTime: [Object],
        _months: [Object],
        _monthsShort: [Object],
        _week: [Object],
        _weekdays: [Object],
        _weekdaysMin: [Object],
        _weekdaysShort: [Object],
        _meridiemParse: /[ap]\.?m?\.?/i,
        _abbr: 'en',
        _config: [Object],
        _ordinalParseLenient: /\d{1,2}(th|st|nd|rd)|\d{1,2}/ } },
  currentDedicated: 0,
  targetDedicated: 4,
  enableAutoScale: false,
  enableInterNodeCommunication: false,
  taskSlotsPerNode: 1,
  taskSchedulingPolicy: { nodeFillType: 'Spread' } }
```

### <a name="step-4-submit-an-azure-batch-job"></a>Étape 4 : soumettre un travail Azure Batch

Un travail Azure Batch est un groupe logique de tâches similaires. Dans notre scénario, il s’agit du travail « Process csv to JSON ». Ici, chaque tâche peut traiter les fichiers csv présents dans chaque conteneur Stockage Azure.

Ces tâches sont exécutées en parallèle et déployées sur plusieurs nœuds, et orchestrées par le service Azure Batch.

> [!TIP]
> Vous pouvez utiliser la propriété [taskSlotsPerNode](https://azure.github.io/azure-sdk-for-node/azure-batch/latest/Pool.html#add) pour spécifier le nombre maximal de tâches pouvant s’exécuter simultanément sur un seul nœud.

#### <a name="preparation-task"></a>Tâche de préparation

Les nœuds de machine virtuelle créés sont des nœuds Ubuntu vides. Il arrive souvent que vous deviez installer un ensemble de programmes comme composants requis.
En règle générale, pour les nœuds Linux, vous pouvez disposer d’un script Shell qui installe les composants requis avant l’exécution des tâches réelles. Cependant, il peut s’agir de n’importe quel exécutable programmable.

Le [script Shell](https://github.com/shwetams/azure-batchclient-sample-nodejs/blob/master/startup_prereq.sh) de cet exemple installe Python-pip et le Kit de développement logiciel (SDK) Stockage Azure pour Python.

Vous pouvez charger le script sur un compte de stockage Azure et générer un URI SAS pour accéder au script. Ce processus peut également être automatisé à l’aide du Kit de développement logiciel (SDK) Node.js pour Stockage Azure.

> [!TIP]
> Une tâche de préparation d’un travail s’exécute uniquement sur les nœuds de machine virtuelle où la tâche donnée doit s’exécuter. Si vous voulez installer les composants requis sur tous les nœuds, quelles que soient les tâches exécutées dessus, vous pouvez utiliser la propriété [startTask](https://azure.github.io/azure-sdk-for-node/azure-batch/latest/Pool.html#add) lorsque vous ajoutez un pool. Vous pouvez utiliser la définition de tâche de préparation suivante à titre de référence.

Une tâche de préparation est spécifiée lors de la soumission d’un travail Azure Batch. Ci-après figurent les paramètres de configuration de tâche de préparation :

- **ID** : identificateur unique de la tâche de préparation
- **commandLine** : ligne de commande pour exécuter la tâche exécutable
- **resourceFiles** : tableau des objets qui fournissent des détails des fichiers qui doivent être téléchargés pour l’exécution de cette tâche.  Ses options sont les suivantes :
  - blobSource : URI SAS du fichier
  - filePath : chemin d’accès local pour télécharger et enregistrer le fichier
  - fileMode : applicable uniquement pour les nœuds Linux, fileMode est au format octal avec la valeur 0770 par défaut
- **waitForSuccess** : si cette option est définie sur la valeur true, la tâche ne s’exécute pas en cas d’échec des tâches de préparation
- **runElevated** : définissez cette option sur la valeur True si des privilèges élevés sont requis pour l’exécution de la tâche.

L’extrait de code suivant illustre un exemple de configuration de script d’une tâche de préparation :

```nodejs
var job_prep_task_config = {id:"installprereq",commandLine:"sudo sh startup_prereq.sh > startup.log",resourceFiles:[{'blobSource':'Blob SAS URI','filePath':'startup_prereq.sh'}],waitForSuccess:true,runElevated:true}
```

Si aucun composant requis ne doit être installé pour l’exécution de vos tâches, vous pouvez ignorer les tâches de préparation. Le code suivant crée un travail dont le nom d’affichage est « process csv files ».

 ```nodejs
 // Setting up Batch pool configuration
 var pool_config = {poolId:poolid}
 // Setting up Job configuration along with preparation task
 var jobId = "processcsvjob"
 var job_config = {id:jobId,displayName:"process csv files",jobPreparationTask:job_prep_task_config,poolInfo:pool_config}
 // Adding Azure batch job to the pool
 var job = batch_client.job.add(job_config,function(error,result){
     if(error != null)
     {
         console.log("Error submitting job : " + error.response);
     }});
```

### <a name="step-5-submit-azure-batch-tasks-for-a-job"></a>Étape 5 : soumettre des tâches Azure Batch pour un travail

Maintenant que le travail de traitement des fichiers csv est créé, nous allons créer des tâches pour celui-ci. En supposant que nous possédons quatre conteneurs, nous devons créer quatre tâches, une par conteneur.

Le [script Python](https://github.com/shwetams/azure-batchclient-sample-nodejs/blob/master/processcsv.py) accepte deux paramètres :

- nom du conteneur : conteneur de stockage à partir duquel télécharger des fichiers
- modèle : paramètre facultatif d’un modèle de nom de fichier

En supposant que nous disposons de quatre conteneurs « con1 », « con2 », « con3 » et « con4 », le code suivant illustre la soumission des tâches pour le travail Azure Batch « process csv » que nous avons créé précédemment.

```nodejs
// storing container names in an array
var container_list = ["con1","con2","con3","con4"]
    container_list.forEach(function(val,index){

           var container_name = val;
           var taskID = container_name + "_process";
           var task_config = {id:taskID,displayName:'process csv in ' + container_name,commandLine:'python processcsv.py --container ' + container_name,resourceFiles:[{'blobSource':'<blob SAS URI>','filePath':'processcsv.py'}]}
           var task = batch_client.task.add(poolid,task_config,function(error,result){
                if(error != null)
                {
                    console.log(error.response);
                }
                else
                {
                    console.log("Task for container : " + container_name + "submitted successfully");
                }



           });

    });
```

Le code permet d’ajouter plusieurs tâches au pool. Chaque tâche est exécutée sur un nœud dans le pool de machines virtuelles créé. Si le nombre de tâches dépasse le nombre de machines virtuelles définies dans un pool ou la propriété taskSlotsPerNode, les tâches attendent qu’un nœud soit de nouveau disponible. Cette orchestration est gérée automatiquement par Azure Batch.

Le portail contient des vues détaillées sur les états des tâches et du travail. Vous pouvez également utiliser la liste et récupérer des fonctions dans le Kit de développement logiciel (SDK) Node Azure. Des détails sont indiqués dans le [lien](https://azure.github.io/azure-sdk-for-node/azure-batch/latest/Job.html) de la documentation.

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur le [workflow et les ressources principales du service Batch](batch-service-workflow-features.md) telles que les pools, les nœuds, les travaux et les tâches.
- Consultez la [référence Node.js Batch](/javascript/api/overview/azure/batch) pour explorer l’API Batch.
