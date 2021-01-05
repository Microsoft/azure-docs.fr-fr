---
title: Utiliser les API REST afin d’effectuer des opérations de CI/CD pour Azure Stream Analytics sur IoT Edge
description: Découvrez comment implémenter un pipeline d’intégration et de déploiement continus pour Azure Stream Analytics à l’aide d’API REST.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 12/04/2018
ms.openlocfilehash: a7e56758a1a76933d6bb18883aa15ce33ce2e89e
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93130916"
---
# <a name="implement-cicd-for-stream-analytics-on-iot-edge-using-apis"></a>Implémenter CI/CD pour Stream Analytics sur IoT Edge à l’aide d’API

Vous pouvez activer l’intégration et le déploiement continus pour les tâches Azure Stream Analytics à l’aide d’API REST. Cet article fournit des exemples d’API à utiliser, et explique comment les utiliser. Les API REST ne sont pas prises en charge sur Azure Cloud Shell.

## <a name="call-apis-from-different-environments"></a>Appeler des API à partir de différents environnements

Vous pouvez appeler des API REST à partir de Linux et Windows. Les commandes suivantes illustrent la syntaxe correcte pour l’appel d’API. L’utilisation spécifique de chaque API est décrite dans les sections suivantes de cet article.

### <a name="linux"></a>Linux

Pour Linux, vous pouvez utiliser la commande `Curl` ou `Wget` :

```bash
curl -u { <username:password> }  -H "Content-Type: application/json" -X { <method> } -d "{ <request body> }" { <url> }   
```

```bash
wget -q -O- --{ <method> } -data="<request body>" --header=Content-Type:application/json --auth-no-challenge --http-user="<Admin>" --http-password="<password>" <url>
```
 
### <a name="windows"></a>Windows

Pour Windows, utilisez PowerShell : 

```powershell 
$user = "<username>" 
$pass = "<password>" 
$encodedCreds = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(("{0}:{1}" -f $user,$pass))) 
$basicAuthValue = "Basic $encodedCreds" 
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]" 
$headers.Add("Content-Type", 'application/json') 
$headers.Add("Authorization", $basicAuthValue) 
$content = "<request body>" 
$response = Invoke-RestMethod <url> -Method <method> -Body $content -Headers $Headers 
echo $response 
```
 
## <a name="create-an-asa-job-on-edge"></a>Créer une tâche ASA sur Edge 
 
Pour créer une tâche Stream Analytics, appelez la méthode PUT à l’aide de l’API Stream Analytics.

|Méthode|URL de la demande|
|------|-----------|
|PUT|`https://management.azure.com/subscriptions/{\**subscription-id**}/resourcegroups/{**resource-group-name**}/providers/Microsoft.StreamAnalytics/streamingjobs/{**job-name**}?api-version=2017-04-01-preview`|
 
Exemple d’utilisation de la commande **curl**  :

```curl
curl -u { <username:password> } -H "Content-Type: application/json" -X { <method> } -d "{ <request body> }" https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/Microsoft.StreamAnalytics/streamingjobs/{jobname}?api-version=2017-04-01-preview  
``` 
 
Exemple de corps de la requête au format JSON :

```json
{ 
  "location": "West US", 
  "tags": { "key": "value", "ms-suppressjobstatusmetrics": "true" }, 
  "sku": {  
      "name": "Standard" 
    }, 
  "properties": { 
    "sku": {  
      "name": "standard" 
    }, 
       "eventsLateArrivalMaxDelayInSeconds": 1, 
       "jobType": "edge", 
    "inputs": [ 
      { 
        "name": "{inputname}", 
        "properties": { 
         "type": "stream", 
          "serialization": { 
            "type": "JSON", 
            "properties": { 
              "fieldDelimiter": ",", 
              "encoding": "UTF8" 
            } 
          }, 
          "datasource": { 
            "type": "GatewayMessageBus", 
            "properties": { 
            } 
          } 
        } 
      } 
    ], 
    "transformation": { 
      "name": "{queryName}", 
      "properties": { 
        "query": "{query}" 
      } 
    }, 
    "package": { 
      "storageAccount" : { 
        "accountName": "{blobstorageaccountname}", 
        "accountKey": "{blobstorageaccountkey}" 
      }, 
      "container": "{blobcontaine}]" 
    }, 
    "outputs": [ 
      { 
        "name": "{outputname}", 
        "properties": { 
          "serialization": { 
            "type": "JSON", 
            "properties": { 
              "fieldDelimiter": ",", 
              "encoding": "UTF8" 
            } 
          }, 
          "datasource": { 
            "type": "GatewayMessageBus", 
            "properties": { 
            } 
          } 
        } 
      } 
    ] 
  } 
} 
```
 
Pour plus d’informations, consultez la [documentation de l’API](/rest/api/streamanalytics/).  
 
## <a name="publish-edge-package"></a>Publier le package de périphérie 
 
Pour publier une tâche Stream Analytics sur IoT Edge, appelez la méthode POST à l’aide de l’API de publication de package Edge.

|Méthode|URL de la demande|
|------|-----------|
|POST|`https://management.azure.com/subscriptions/{\**subscriptionid**}/resourceGroups/{**resourcegroupname**}/providers/Microsoft.StreamAnalytics/streamingjobs/{**jobname**}/publishedgepackage?api-version=2017-04-01-preview`|

Cette opération asynchrone retourne l’état 202 tant que la tâche n’est pas publiée. L’en-tête de réponse d’emplacement contient l’URI utilisé pour obtenir l’état du processus. Pendant que le processus est en cours d’exécution, un appel à l’URI dans l’en-tête d’emplacement retourne l’état 202. Une fois le processus terminé, l’URI dans l’en-tête d’emplacement retourne l’état 200. 

Exemple d’appel de publication de package Edge à l’aide de **curl**  : 

```bash
curl -d -X POST https://management.azure.com/subscriptions/{subscriptionid}/resourceGroups/{resourcegroupname}/providers/Microsoft.StreamAnalytics/streamingjobs/{jobname}/publishedgepackage?api-version=2017-04-01-preview
```
 
Après avoir effectué l’appel POST, attendez-vous à une réponse avec un corps vide. Recherchez l’URL située dans la tête de la réponse et notez-la pour plus tard.
 
Exemple d’URL de la tête de la réponse :

```
https://management.azure.com/subscriptions/{**subscriptionid**}/resourcegroups/{**resourcegroupname**}/providers/Microsoft.StreamAnalytics/StreamingJobs/{**resourcename**}/OperationResults/023a4d68-ffaf-4e16-8414-cb6f2e14fe23?api-version=2017-04-01-preview 
```
Patientez une ou deux minutes avant d’exécuter la commande suivante pour effectuer un appel d’API avec l’URL que vous avez trouvée dans la tête de la réponse. Réessayez d’exécuter la commande si vous n’obtenez pas une réponse HTTP 200.
 
Exemple d’appel d’API avec l’URL retournée avec **curl**  :

```bash
curl -d –X GET https://management.azure.com/subscriptions/{subscriptionid}/resourceGroups/{resourcegroupname}/providers/Microsoft.StreamAnalytics/streamingjobs/{resourcename}/publishedgepackage?api-version=2017-04-01-preview 
```

La réponse inclut les informations que vous devez ajouter au script de déploiement Edge. Les exemples ci-dessous illustrent les informations que vous devez recueillir et où les ajouter dans le manifeste du déploiement.
 
Exemple de corps de réponse une fois la publication réussie :

```json
{ 
  edgePackageUrl : null 
  error : null 
  manifest : "{"supportedPlatforms":[{"os":"linux","arch":"amd64","features":[]},{"os":"linux","arch":"arm","features":[]},{"os":"windows","arch":"amd64","features":[]}],"schemaVersion":"2","name":"{jobname}","version":"1.0.0.0","type":"docker","settings":{"image":"{imageurl}","createOptions":null},"endpoints":{"inputs":["\],"outputs":["{outputnames}"]},"twin":{"contentType":"assignments","content":{"properties.desired":{"ASAJobInfo":"{asajobsasurl}","ASAJobResourceId":"{asajobresourceid}","ASAJobEtag":"{etag}","PublishTimeStamp":"{publishtimestamp}"}}}}" 
  status : "Succeeded" 
} 
```

Exemple de manifeste de déploiement : 

```json
{ 
  "modulesContent": { 
    "$edgeAgent": { 
      "properties.desired": { 
        "schemaVersion": "1.0", 
        "runtime": { 
          "type": "docker", 
          "settings": { 
            "minDockerVersion": "v1.25", 
            "loggingOptions": "", 
            "registryCredentials": {} 
          } 
        }, 
        "systemModules": { 
          "edgeAgent": { 
            "type": "docker", 
            "settings": { 
              "image": "mcr.microsoft.com/azureiotedge-agent:1.0", 
              "createOptions": "{}" 
            } 
          }, 
          "edgeHub": { 
            "type": "docker", 
            "status": "running", 
            "restartPolicy": "always", 
            "settings": { 
              "image": "mcr.microsoft.com/azureiotedge-hub:1.0", 
              "createOptions": "{}" 
            } 
          } 
        }, 
        "modules": { 
          "<asajobname>": { 
            "version": "1.0", 
            "type": "docker", 
            "status": "running", 
            "restartPolicy": "always", 
            "settings": { 
              "image": "<settings.image>", 
              "createOptions": "<settings.createOptions>" 
            } 
            "version": "<version>", 
             "env": { 
              "PlanId": { 
               "value": "stream-analytics-on-iot-edge" 
          } 
        } 
      } 
    }, 
    "$edgeHub": { 
      "properties.desired": { 
        "schemaVersion": "1.0", 
        "routes": { 
            "route": "FROM /* INTO $upstream" 
        }, 
        "storeAndForwardConfiguration": { 
          "timeToLiveSecs": 7200 
        } 
      } 
    }, 
    "<asajobname>": { 
      "properties.desired": {<twin.content.properties.desired>} 
    } 
  } 
} 
```

Après la configuration du manifeste de déploiement, consultez [Déployer des modules Azure IoT Edge avec Azure CLI](../iot-edge/how-to-deploy-modules-cli.md) pour plus d’informations sur le déploiement.


## <a name="next-steps"></a>Étapes suivantes 
 
* [Azure Stream Analytics sur IoT Edge](stream-analytics-edge.md)
* [Didacticiel pour ASA sur IoT Edge](../iot-edge/tutorial-deploy-stream-analytics.md)
* [Développer des travaux Edge Stream Analytics avec les outils Visual Studio](stream-analytics-tools-for-visual-studio-edge-jobs.md)