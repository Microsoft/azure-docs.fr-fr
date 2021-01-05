---
title: Utiliser REST pour gérer des ressources ML
titleSuffix: Azure Machine Learning
description: Comment utiliser des API REST pour créer, exécuter et supprimer des ressources Azure Machine Learning, telles qu’un espace de travail, ou enregistrer des modèles.
author: lobrien
ms.author: laobri
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.date: 01/31/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 09a0580adbe6d51e4de811a57ee17203d65a2435
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93316901"
---
# <a name="create-run-and-delete-azure-ml-resources-using-rest"></a>Créer, exécuter et supprimer des ressources Azure ML à l’aide de REST



Il existe plusieurs façons de gérer vos ressources Azure ML. Vous pouvez utiliser le [portail](https://portal.azure.com/), l’[interface de ligne de commande](/cli/azure/?preserve-view=true&view=azure-cli-latest) ou le [Kit de développement logiciel (SDK) Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py). Ou vous pouvez choisir l’API REST. L’API REST utilise des verbes HTTP de manière standard pour créer, récupérer, mettre à jour et supprimer des ressources. L’API REST fonctionne avec tout langage de programmation ou outil pouvant effectuer des requêtes HTTP. La structure simple de REST en fait souvent un bon choix dans les environnements de script et pour l’automatisation MLOps. 

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Récupérer un jeton d’autorisation
> * Créer une requête REST correctement formatée à l’aide de l’authentification du principal de service
> * Utiliser des requêtes GET pour récupérer des informations sur les ressources hiérarchiques d’Azure ML
> * Utiliser des requêtes PUT et POST pour créer et modifier des ressources
> * Utiliser des requêtes DELETE pour nettoyer les ressources 
> * Utiliser l’autorisation basée sur une clé pour attribuer un score aux modèles déployés

## <a name="prerequisites"></a>Conditions préalables requises

- Un **abonnement Azure** pour lequel vous disposez de droits d’administration. Si vous n’avez pas d’abonnement de ce type, essayez l’[abonnement personnel gratuit ou payant](https://aka.ms/AMLFree)
- Un [espace de travail Azure Machine Learning](./how-to-manage-workspace.md)
- Les requêtes REST administratives utilisent l’authentification du principal de service. Suivez les étapes décrites dans [Configurer l’authentification pour des ressources et workflows Azure Machine Learning](./how-to-setup-authentication.md#service-principal-authentication) pour créer un principal de service dans votre espace de travail.
- L’utilitaire **curl**. Le programme **curl** est disponible dans le [Sous-système Windows pour Linux](/windows/wsl/install-win10) ou dans toute distribution UNIX. Dans PowerShell, **curl** est un alias pour **Invoke-WebRequest** et `curl -d "key=val" -X POST uri` devient `Invoke-WebRequest -Body "key=val" -Method POST -Uri uri`. 

## <a name="retrieve-a-service-principal-authentication-token"></a>Récupérer un jeton d’authentification du principal de service

Les requêtes REST administratives sont authentifiées à l’aide d’un flux implicite OAuth2. Ce flux d’authentification utilise un jeton fourni par le principal de service de votre abonnement. Pour récupérer ce jeton, vous aurez besoin des éléments suivants :

- Votre ID de locataire (identifie l’organisation à laquelle votre abonnement appartient)
- Votre ID client (qui sera associé au jeton créé)
- Votre clé secrète client (que vous devez protéger)

Vous devez avoir obtenu ces valeurs dans la réponse à la création de votre principal de service. L’obtention de ces valeurs est documentée dans [Configurer l’authentification pour des ressources et workflows Azure Machine Learning](./how-to-setup-authentication.md#service-principal-authentication). Si vous utilisez l’abonnement de votre entreprise, vous n’êtes peut-être pas autorisé à créer un principal de service. Dans ce cas, vous devez utiliser une [abonnement personnel gratuit ou payant](https://aka.ms/AMLFree).

Pour récupérer un jeton :

1. Ouvrez une fenêtre de terminal.
1. Entrez le code suivant sur la ligne de commande
1. Remplacez `{your-tenant-id}`, `{your-client-id}` et `{your-client-secret}` par vos propres valeurs. Tout au long de cet article, les chaînes placées entre accolades sont des variables que vous devrez remplacer par vos propres valeurs appropriées.
1. Exécutez la commande.

```bash
curl -X POST https://login.microsoftonline.com/{your-tenant-id}/oauth2/token \
-d "grant_type=client_credentials&resource=https%3A%2F%2Fmanagement.azure.com%2F&client_id={your-client-id}&client_secret={your-client-secret}" \
```

La réponse doit fournir un jeton d’accès valable pendant une heure :

```json
{
    "token_type": "Bearer",
    "expires_in": "3599",
    "ext_expires_in": "3599",
    "expires_on": "1578523094",
    "not_before": "1578519194",
    "resource": "https://management.azure.com/",
    "access_token": "your-access-token"
}
```

Notez le jeton, car vous l’utiliserez pour authentifier toutes les requêtes administratives ultérieures. Pour ce faire, vous devez définir un en-tête d’autorisation dans toutes les requêtes :

```bash
curl -h "Authentication: Bearer {your-access-token}" ...more args...
```

Notez que la valeur commence par la chaîne « Bearer », comprenant une espace unique avant que vous n’ajoutiez le jeton.

## <a name="get-a-list-of-resource-groups-associated-with-your-subscription"></a>Obtenir la liste des groupes de ressources associés à votre abonnement

Pour récupérer la liste des groupes de ressources associés à votre abonnement, exécutez :

```bash
curl https://management.azure.com/subscriptions/{your-subscription-id}/resourceGroups?api-version=2019-11-01 -H "Authorization:Bearer {your-access-token}"
```

Dans Azure, de nombreuses API REST sont publiées. Chaque fournisseur de services met à jour son API à son propre rythme, mais le fait sans arrêter les programmes existants. Le fournisseur de services utilise l’argument `api-version` pour garantir la compatibilité. L’argument `api-version` varie d’un service à l’autre. Pour le service Machine Learning, par exemple, la version actuelle de l’API est `2019-11-01`. Pour les comptes de stockage, il s’agit de `2019-06-01`. Pour les coffres de clés, il s’agit de `2019-09-01`. Tous les appels REST doivent définir l’argument `api-version` sur la valeur attendue. Vous pouvez vous appuyer sur la syntaxe et la sémantique de la version spécifiée, même si l’API continue d’évoluer. Si vous envoyez une requête à un fournisseur sans l’argument `api-version`, la réponse contiendra une liste lisible par un humain des valeurs prises en charge. 

L’appel ci-dessus génère une réponse JSON compactée au format suivant : 

```json
{
    "value": [
        {
            "id": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourceGroups/RG1",
            "name": "RG1",
            "type": "Microsoft.Resources/resourceGroups",
            "location": "westus2",
            "properties": {
                "provisioningState": "Succeeded"
            }
        },
        {
            "id": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourceGroups/RG2",
            "name": "RG2",
            "type": "Microsoft.Resources/resourceGroups",
            "location": "eastus",
            "properties": {
                "provisioningState": "Succeeded"
            }
        }
    ]
}
```


## <a name="drill-down-into-workspaces-and-their-resources"></a>Explorer au niveau du détail des espaces de travail et leurs ressources

Pour récupérer l’ensemble d’espaces de travail se trouvant dans un groupe de ressources, exécutez la commande suivante, en remplaçant `{your-subscription-id}`, `{your-resource-group}` et `{your-access-token}` : 

```
curl https://management.azure.com/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/providers/Microsoft.MachineLearningServices/workspaces/?api-version=2019-11-01 \
-H "Authorization:Bearer {your-access-token}"
```

Là encore, vous recevrez une liste JSON, contenant cette fois-ci une liste dont chaque élément détaille un espace de travail :

```json
{
    "id": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourceGroups/DeepLearningResourceGroup/providers/Microsoft.MachineLearningServices/workspaces/my-workspace",
    "name": "my-workspace",
    "type": "Microsoft.MachineLearningServices/workspaces",
    "location": "centralus",
    "tags": {},
    "etag": null,
    "properties": {
        "friendlyName": "",
        "description": "",
        "creationTime": "2020-01-03T19:56:09.7588299+00:00",
        "storageAccount": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourcegroups/DeepLearningResourceGroup/providers/microsoft.storage/storageaccounts/myworkspace0275623111",
        "containerRegistry": null,
        "keyVault": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourcegroups/DeepLearningResourceGroup/providers/microsoft.keyvault/vaults/myworkspace2525649324",
        "applicationInsights": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourcegroups/DeepLearningResourceGroup/providers/microsoft.insights/components/myworkspace2053523719",
        "hbiWorkspace": false,
        "workspaceId": "cba12345-abab-abab-abab-ababab123456",
        "subscriptionState": null,
        "subscriptionStatusChangeTimeStampUtc": null,
        "discoveryUrl": "https://centralus.experiments.azureml.net/discovery"
    },
    "identity": {
        "type": "SystemAssigned",
        "principalId": "abcdef1-abab-1234-1234-abababab123456",
        "tenantId": "1fedcba-abab-1234-1234-abababab123456"
    },
    "sku": {
        "name": "Basic",
        "tier": "Basic"
    }
}
```

Pour travailler avec des ressources au sein d’un espace de travail, vous basculerez du serveur général **management.azure.com** vers un serveur d’API REST spécifique à l’emplacement de l’espace de travail. Notez la valeur de la clé de `discoveryUrl` dans la réponse JSON ci-dessus. Si vous envoyez une requête GET à cette URL, vous recevrez une réponse semblable à la suivante :

```json
{
  "api": "https://centralus.api.azureml.ms",
  "catalog": "https://catalog.cortanaanalytics.com",
  "experimentation": "https://centralus.experiments.azureml.net",
  "gallery": "https://gallery.cortanaintelligence.com/project",
  "history": "https://centralus.experiments.azureml.net",
  "hyperdrive": "https://centralus.experiments.azureml.net",
  "labeling": "https://centralus.experiments.azureml.net",
  "modelmanagement": "https://centralus.modelmanagement.azureml.net",
  "pipelines": "https://centralus.aether.ms",
  "studiocoreservices": "https://centralus.studioservice.azureml.com"
}
```

La valeur de la réponse `api` est l’URL du serveur que vous utiliserez pour des requêtes supplémentaires. Par exemple, pour répertorier les expériences, envoyez la commande suivante. Remplacez `regional-api-server` par la valeur de la réponse `api` (par exemple : `centralus.api.azureml.ms`). Remplacez également `your-subscription-id`, `your-resource-group`, `your-workspace-name` et `your-access-token` comme d’habitude :

```bash
curl https://{regional-api-server}/history/v1.0/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/experiments?api-version=2019-11-01 \
-H "Authorization:Bearer {your-access-token}"
```

De même, pour récupérer les modèles inscrits dans votre espace de travail, envoyez :

```bash
curl https://{regional-api-server}/modelmanagement/v1.0/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/models?api-version=2019-11-01 \
-H "Authorization:Bearer {your-access-token}"
```

Notez que pour répertorier les expériences, le chemin d’accès commence par `history/v1.0`, tandis que pour répertorier les modèles, le chemin d’accès commence par `modelmanagement/v1.0`. L’API REST est divisée en plusieurs groupes opérationnels, chacun doté d’un chemin d’accès distinct. 

|Domaine|Path|
|-|-|
|Artefacts|/rest/api/azureml|
|Magasins de données|/azure/machine-learning/how-to-access-data|
|Optimisation des hyperparamètres|hyperdrive/v1.0/|
|Modèles|modelmanagement/v1.0/|
|Historique d’exécution|execution/v1.0/ et history/v1.0/|

Vous pouvez explorer l’API REST à l’aide du modèle général de :

|Composant URL|Exemple|
|-|-|
| https://| |
| regional-api-server/ | centralus.api.azureml.ms/ |
| operations-path/ | history/v1.0/ |
| subscriptions/{your-subscription-id}/ | subscriptions/abcde123-abab-abab-1234-0123456789abc/ |
| resourceGroups/{your-resource-group}/ | resourceGroups/MyResourceGroup/ |
| providers/operation-provider/ | providers/Microsoft.MachineLearningServices/ |
| provider-resource-path/ | workspaces/MLWorkspace/MyWorkspace/FirstExperiment/runs/1/ |
| operations-endpoint/ | artifacts/metadata/ |


## <a name="create-and-modify-resources-using-put-and-post-requests"></a>Créer et modifier des ressources à l’aide de requêtes PUT et POST

En plus de la récupération des ressources à l’aide du verbe GET, l’API REST prend en charge la création de toutes les ressources nécessaires pour l’apprentissage, le déploiement et la surveillance des solutions ML. 

La formation et l’exécution de modèles ML nécessitent des ressources de calcul. Vous pouvez répertorier les ressources de calcul d’un espace de travail avec : 

```bash
curl https://management.azure.com/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/computes?api-version=2019-11-01 \
-H "Authorization:Bearer {your-access-token}"
```

Pour créer ou remplacer une ressource de calcul nommée, vous utiliserez une requête PUT. Dans ce qui suit, outre les substitutions familières des éléments `your-subscription-id`, `your-resource-group`, `your-workspace-name` et `your-access-token`, remplacez `your-compute-name` ainsi que les valeurs pour `location`, `vmSize`, `vmPriority`, `scaleSettings`, `adminUserName` et `adminUserPassword`. Comme indiqué dans la référence à [Capacité de calcul Machine Learning : créer ou mettre à jour la référence du Kit de développement logiciel (SDK)](/rest/api/azureml/workspacesandcomputes/machinelearningcompute/createorupdate), la commande suivante crée une ressource Standard_D1 dédiée à nœud unique (ressource de calcul de base de l’UC) qui descendra en puissance après 30 minutes :

```bash
curl -X PUT \
  'https://management.azure.com/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/computes/{your-compute-name}?api-version=2019-11-01' \
  -H 'Authorization:Bearer {your-access-token}' \
  -H 'Content-Type: application/json' \
  -d '{
    "location": "{your-azure-location}",
    "properties": {
        "computeType": "AmlCompute",
        "properties": {
            "vmSize": "Standard_D1",
            "vmPriority": "Dedicated",
            "scaleSettings": {
                "maxNodeCount": 1,
                "minNodeCount": 0,
                "nodeIdleTimeBeforeScaleDown": "PT30M"
            }
        }
    },
    "userAccountCredentials": {
        "adminUserName": "{adminUserName}",
        "adminUserPassword": "{adminUserPassword}"
    }
}'
```

> [!Note]
> Dans les terminaux Windows, vous devrez peut-être faire précéder les guillemets doubles d’un caractère d’échappement lors de l’envoi de données JSON. Autrement dit, un texte comme `"location"` devient `\"location\"`. 

Une requête réussie recevra une réponse `201 Created`, mais notez que cette réponse signifie simplement que le processus d’approvisionnement a commencé. Vous devrez procéder à un sondage (ou utiliser le portail) pour confirmer son bon déroulement.

### <a name="create-an-experimental-run"></a>Créer une exécution expérimentale

Pour démarrer une exécution dans le cadre d’une expérience, vous avez besoin d’un dossier zip contenant votre script de formation et les fichiers associés, ainsi que d’un fichier JSON de définition d’exécution. Le dossier zip doit contenir le fichier d’entrée Python dans son répertoire racine. Par exemple, compressez au format zip un programme Python simple tel que le programme suivant dans un dossier appelé **train.zip**.

```python
# hello.py
# Entry file for run
print("Hello, REST!")
```

Enregistrez l’extrait de code suivant sous **definition.json**. Confirmez que la valeur « Script » correspond au nom du fichier Python que vous venez de compresser. Confirmez que la valeur « Cible » correspond au nom d’une ressource de calcul disponible. 

```json
{
    "Configuration":{  
       "Script":"hello.py",
       "Arguments":[  
          "234"
       ],
       "SourceDirectoryDataStore":null,
       "Framework":"Python",
       "Communicator":"None",
       "Target":"cpu-compute",
       "MaxRunDurationSeconds":1200,
       "NodeCount":1,
       "Environment":{  
          "Python":{  
             "InterpreterPath":"python",
             "UserManagedDependencies":false,
             "CondaDependencies":{  
                "name":"project_environment",
                "dependencies":[  
                   "python=3.6.2",
                   {  
                      "pip":[  
                         "azureml-defaults"
                      ]
                   }
                ]
             }
          },
          "Docker":{  
             "BaseImage":"mcr.microsoft.com/azureml/base:intelmpi2018.3-ubuntu16.04"
          }
      },
       "History":{  
          "OutputCollection":true
       }
    }
}
```

Publiez ces fichiers sur le serveur à l’aide du contenu `multipart/form-data` :

```bash
curl https://{regional-api-server}/execution/v1.0/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/experiments/{your-experiment-name}/startrun?api-version=2019-11-01 \
  -X POST \
  -H "Content-Type: multipart/form-data" \
  -H "Authorization:Bearer {your-access-token}" \
  -F projectZipFile=@train.zip \
  -F runDefinitionFile=@runDefinition.json
```

Une requête POST réussie génère un état `200 OK`, avec un corps de réponse contenant l’identificateur de l’exécution créée :

```json
{
  "runId": "my-first-experiment_1579642222877"
}
```

Vous pouvez surveiller une exécution à l’aide du modèle REST-ful qui devrait maintenant vous être familier :

```bash
curl 'https://{regional-api-server}/history/v1.0/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/experiments/{your-experiment-names}/runs/{your-run-id}?api-version=2019-11-01' \
  -H 'Authorization:Bearer {your-access-token}'
```

### <a name="delete-resources-you-no-longer-need"></a>Supprimer les ressources dont vous n’avez plus besoin

Certaines ressources, mais pas toutes, prennent en charge le verbe DELETE. Vérifiez les [informations de référence sur l’API](/rest/api/azureml/) avant de valider l’API REST pour les cas d’utilisation en suppression. Par exemple, pour supprimer un modèle, vous pouvez utiliser :

```bash
curl
  -X DELETE \
'https://{regional-api-server}/modelmanagement/v1.0/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/models/{your-model-id}?api-version=2019-11-01' \
  -H 'Authorization:Bearer {your-access-token}' 
```

## <a name="use-rest-to-score-a-deployed-model"></a>Utiliser REST pour attribuer un score à un modèle déployé

Bien qu’il soit possible de déployer un modèle de sorte qu’il s’authentifie auprès d’un principal de service, la plupart des déploiements exposés aux clients utilisent l’authentification basée sur une clé. Vous pouvez trouver la clé appropriée dans la page de votre déploiement, dans l’onglet **Points de terminaison** de Studio. Le même emplacement affiche l’URI de scoring de votre point de terminaison. Les entrées de votre modèle doivent être modélisées en tant que tableau JSON nommé `data` :

```bash
curl 'https://{scoring-uri}' \
 -H 'Authorization:Bearer {your-key}' \
 -H 'Content-Type: application/json' \
  -d '{ "data" : [ {model-specific-data-structure} ] }
```

## <a name="create-a-workspace-using-rest"></a>Créer un espace de travail à l’aide de REST 

Chaque espace de travail Azure ML est doté d’une dépendance sur quatre autres ressources Azure : un registre de conteneurs dont l’administration est activée, un coffre de clés, une ressource Application Insights et un compte de stockage. Vous ne pouvez pas créer un espace de travail tant que ces ressources n’existent pas. Pour plus d’informations sur la création de ces ressources, consultez les informations de référence sur l’API REST.

Pour créer un espace de travail, faites une requête PUT auprès de `management.azure.com` selon un appel similaire à ce qui suit. Bien que cet appel vous oblige à définir un grand nombre de variables, il est structurellement identique aux autres appels dont il a été question dans cet article. 

```bash
curl -X PUT \
  'https://management.azure.com/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}\
/providers/Microsoft.MachineLearningServices/workspaces/{your-new-workspace-name}?api-version=2019-11-01' \
  -H 'Authorization: Bearer {your-access-token}' \
  -H 'Content-Type: application/json' \
  -d '{
    "location": "{desired-region}",
    "properties": {
        "friendlyName" : "{your-workspace-friendly-name}",
        "description" : "{your-workspace-description}",
        "containerRegistry" : "/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.ContainerRegistry/registries/{your-registry-name}",
        keyVault" : "/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}\
/providers/Microsoft.Keyvault/vaults/{your-keyvault-name}",
        "applicationInsights" : "subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.insights/components/{your-application-insights-name}",
        "storageAccount" : "/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.Storage/storageAccounts/{your-storage-account-name}"
    },
    "identity" : {
        "type" : "systemAssigned"
    }
}'
```

Vous devez recevoir une réponse `202 Accepted` et, dans les en-têtes retournés, un URI `Location`. Vous pouvez faire une requête GET auprès de cet URI pour plus d’informations sur le déploiement, y compris des informations utiles pour le débogage en cas de problème avec l’une de vos ressources dépendantes (par exemple, si vous avez oublié d’activer l’accès administrateur sur votre registre de conteneurs). 

## <a name="troubleshooting"></a>Dépannage

### <a name="resource-provider-errors"></a>Erreurs du fournisseur de ressources

[!INCLUDE [machine-learning-resource-provider](../../includes/machine-learning-resource-provider.md)]

### <a name="moving-the-workspace"></a>Déplacement de l’espace de travail

> [!WARNING]
> Le déplacement de votre espace de travail Azure Machine Learning vers un autre abonnement, ou le déplacement de l’abonnement propriétaire vers un nouveau locataire, n’est pas pris en charge. En effet, cela peut provoquer des erreurs.

### <a name="deleting-the-azure-container-registry"></a>Suppression d'Azure Container Registry

L'espace de travail Azure Machine Learning utilise Azure Container Registry (ACR) pour certaines opérations. Il crée automatiquement une instance ACR dès qu'il en a besoin.

[!INCLUDE [machine-learning-delete-acr](../../includes/machine-learning-delete-acr.md)]

## <a name="next-steps"></a>Étapes suivantes

- Explorez la [documentation complète de l’API REST AzureML](/rest/api/azureml/).
- Découvrez comment utiliser le concepteur pour [prédire le prix de voitures avec le concepteur](./tutorial-designer-automobile-price-train-score.md).
- Explorez [Azure Machine Learning avec des notebooks Jupyter](..//machine-learning/samples-notebooks.md).