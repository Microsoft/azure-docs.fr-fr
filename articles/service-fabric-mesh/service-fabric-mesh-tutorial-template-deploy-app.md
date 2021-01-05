---
title: Tutoriel - Déployer une application sur Azure Service Fabric Mesh
description: Dans ce tutoriel, vous allez apprendre à déployer une application sur Service Fabric Mesh à l’aide d’un modèle.
author: georgewallace
ms.topic: tutorial
ms.date: 01/11/2019
ms.author: gwallace
ms.custom: mvc, devcenter, devx-track-azurecli
ms.openlocfilehash: 54ac7b27ada62a969dd40428fd9a753bb5a99530
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499830"
---
# <a name="tutorial-deploy-an-application-to-service-fabric-mesh-using-a-template"></a>Tutoriel : Déployer une application sur Service Fabric Mesh à l’aide d’un modèle.

Ce tutoriel est la première partie d’une série d’étapes. Vous allez apprendre à déployer une application Azure Service Fabric Mesh à l’aide d’un modèle.  L’application est composée d’un service frontal web ASP.NET et d’un service backend d’API web ASP.NET Core, qui se trouvent dans Docker Hub.  Vous tirez (pull) les deux images conteneur de Docker Hub et vous les envoyez (push) vers votre propre registre privé. Ensuite, vous créez un modèle Azure RM pour l’application, puis déployez celle-ci à partir de votre registre de conteneurs sur Service Fabric Mesh. Quand vous avez terminé, vous disposez d’une application simple de liste de tâches qui s’exécute dans Service Fabric Mesh.

Dans ce premier volet, vous apprenez à :

> [!div class="checklist"]
> * Créer une instance Azure Container Registry privée
> * Envoyer une image de conteneur vers le Registre
> * Créer des fichiers de paramètres et de modèle RM
> * Déployer une application sur Service Fabric Mesh

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * Déployer une application sur Service Fabric Mesh à l’aide d’un modèle
> * [Mettre à l’échelle des services à l’intérieur d’une application en cours d’exécution dans Service Fabric Mesh](service-fabric-mesh-tutorial-template-scale-services.md)
> * [Mettre à niveau une application en cours d’exécution dans Service Fabric Mesh](service-fabric-mesh-tutorial-template-upgrade-app.md)
> * [Supprimer une application](service-fabric-mesh-tutorial-template-remove-app.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer ce tutoriel :

* Si vous n’avez pas d’abonnement Azure, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

* [Installation de Docker](service-fabric-mesh-howto-setup-developer-environment-sdk.md#install-docker)

* [Installez localement l’interface de ligne de commande Azure et l’interface de ligne de commande Service Fabric Mesh](service-fabric-mesh-howto-setup-cli.md#install-the-azure-service-fabric-mesh-cli).

## <a name="create-a-container-registry"></a>Créer un registre de conteneur

Les images conteneur associées aux services dans votre application Service Fabric Mesh doivent être stockées dans un registre de conteneurs.  Ce tutoriel utilise une instance Azure Container Registry (ACR) privée. 

Effectuez les étapes suivantes pour créer une instance ACR.  Si vous avez déjà configuré une instance ACR, vous pouvez ignorer cette procédure.

### <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous à Azure et définissez l’abonnement actif.

```azurecli
az login
az account set --subscription "<subscriptionName>"
```

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Utilisez la commande suivante pour créer un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus*.

```azurecli
az group create --name myResourceGroup --location eastus
```

### <a name="create-the-container-registry"></a>Créer le registre de conteneurs

Créez une instance ACR à l’aide de la commande `az acr create`. Le nom du registre doit être unique dans Azure et contenir entre 5 et 50 caractères alphanumériques. Dans l’exemple suivant, nous utilisons le nom *myContainerRegistry*. Si vous obtenez une erreur selon laquelle le nom de registre est déjà utilisé, choisissez un autre nom.

```azurecli
az acr create --resource-group myResourceGroup --name myContainerRegistry --sku Basic
```

Une fois le registre créé, vous voyez une sortie semblable à ce qui suit :

```json
{
  "adminUserEnabled": false,
  "creationDate": "2018-09-13T19:43:57.388203+00:00",
  "id": "/subscriptions/<subscription>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/myContainerRegistry",
  "location": "eastus",
  "loginServer": "mycontainerregistry.azurecr.io",
  "name": "myContainerRegistry",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

## <a name="push-the-images-to-azure-container-registry"></a>Envoyer les images vers Azure Container Registry

Ce tutoriel utilise l’exemple d’application de liste de tâches.  Les images conteneur pour les services [WebFrontEnd](https://hub.docker.com/r/seabreeze/azure-mesh-todo-webfrontend/) et [ToDoService](https://hub.docker.com/r/seabreeze/azure-mesh-todo-service/) se trouvent sur Docker Hub. Pour plus d’informations sur la génération de l’application dans Visual Studio, consultez [Générer une application web Service Fabric Mesh](service-fabric-mesh-tutorial-create-dotnetcore.md). Service Fabric Mesh peut exécuter des conteneurs Docker Linux ou Windows.  Si vous utilisez des conteneurs Linux, sélectionnez **Switch to Linux containers** (Basculer vers les conteneurs Linux) dans Docker.  Si vous utilisez des conteneurs Windows, sélectionnez **Switch to Windows containers** (Basculer vers les conteneurs Windows) dans Docker.

Pour envoyer une image vers une instance ACR, vous devez tout d’abord disposer d’une image conteneur. Si vous n’avez pas encore d’images conteneur locales, utilisez la commande [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) pour tirer les images [WebFrontEnd](https://hub.docker.com/r/seabreeze/azure-mesh-todo-webfrontend/) et [ToDoService](https://hub.docker.com/r/seabreeze/azure-mesh-todo-service/) de Docker Hub.

>[!NOTE]
> Depuis le 2 novembre 2020, des [limites de taux de téléchargement s’appliquent](https://docs.docker.com/docker-hub/download-rate-limit/) aux requêtes anonymes et authentifiées qui sont envoyées à Docker Hub à partir de comptes de plan Docker Gratuit. Ces limites sont appliquées par adresse IP. 
> 
> Ces commandes utilisent des images publiques issues de Docker Hub. Notez que votre taux de téléchargement peut être limité. Pour plus d’informations, consultez [S’authentifier auprès de Docker Hub](../container-registry/buffer-gate-public-content.md#authenticate-with-docker-hub).

Tirer les images Windows :

```bash
docker pull seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709
docker pull seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709
```

Avant de pousser (push) une image vers le registre, vous devez la marquer avec le nom complet de votre serveur de connexion ACR.

Exécutez la commande suivante pour obtenir le nom complet du serveur de connexion de votre instance ACR.

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table

AcrLoginServer
---------------------------------
mycontainerregistry.azurecr.io
```

À présent, étiquetez l’image en utilisant la commande [docker tag](https://docs.docker.com/engine/reference/commandline/tag/). L’exemple suivant étiquette les images seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709 et seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709. Si vous utilisez des images différentes, substituez les noms de ces images dans la commande suivante.

```bash
docker tag seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709 mycontainerregistry.azurecr.io/seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709
docker tag seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709 mycontainerregistry.azurecr.io/seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709
```

Connectez-vous à Azure Container Registry.

```azurecli
az acr login -n myContainerRegistry
```

Envoyez l’image vers l’instance ACR à l’aide de la commande [docker push](https://docs.docker.com/engine/reference/commandline/push/).

```bash
docker push mycontainerregistry.azurecr.io/seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709
docker push mycontainerregistry.azurecr.io/seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709
```

### <a name="list-container-images"></a>Répertorier les images conteneur

Pour afficher les référentiels dans votre instance ACR, exécutez la commande suivante :

```azurecli
az acr repository list --name myContainerRegistry --output table

Result
-------------------------------
seabreeze/azure-mesh-todo-webfrontend
seabreeze/azure-mesh-todo-service
```

L’exemple suivant répertorie les étiquettes sur le référentiel **azure-mesh-todo-service**.

```azurecli
az acr repository show-tags --name myContainerRegistry --repository seabreeze/azure-mesh-todo-service --output table

Result
--------
1.0-nanoserver-1709
```

La sortie précédente confirme la présence de `azure-mesh-todo-service:1.0-nanoserver-1709` dans le registre de conteneurs privé.  Vérifiez également la présence d’`azure-mesh-todo-webfrontend:1.0-nanoserver-1709`.

## <a name="retrieve-credentials-for-the-registry"></a>Récupérer les informations d’identification pour le registre

> [!IMPORTANT]
> L’activation de l’utilisateur administrateur sur une instance ACR n’est pas recommandée pour les scénarios de production. Elle est effectuée ici pour des raisons pratiques. Pour les scénarios de production, utilisez un [principal de service](../container-registry/container-registry-auth-service-principal.md) pour l’authentification utilisateur et système.

Pour déployer une instance de conteneur à partir du registre créé à l’aide d’un modèle, vous devez fournir les informations d’identification du registre pendant le déploiement. Tout d’abord, activez l’utilisateur administrateur sur votre registre avec la commande suivante :

```azurecli
az acr update --name myContainerRegistry --admin-enabled true
```

Ensuite, obtenez le nom de serveur de connexion, le nom d’utilisateur et le mot de passe du registre en utilisant les commandes suivantes :

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
az acr credential show --name myContainerRegistry --query username
az acr credential show --name myContainerRegistry --query "passwords[0].value"
```

Les valeurs retournées pour le nom de serveur de connexion ACR, le nom d’utilisateur et le mot de passe vous serviront à créer les fichiers de paramètres et de modèle RM dans la section suivante.

## <a name="download-and-explore-the-template-and-parameters-files"></a>Télécharger et explorer les fichiers de modèle et de paramètres

Une application Service Fabric Mesh est une ressource Azure que vous pouvez déployer et gérer à l’aide de modèles Azure Resource Manager (RM). Si vous n’avez pas une bonne connaissance des concepts de déploiement et de gestion des solutions Azure, consultez [Vue d’ensemble d’Azure Resource Manager](../azure-resource-manager/management/overview.md) et [Comprendre la structure et la syntaxe des modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md).

Ce tutoriel utilise l’exemple de liste de tâches.  Au lieu de créer des fichiers de modèle et de paramètres, téléchargez les fichiers de [modèle de déploiement mesh_rp.windows.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json) et de [paramètres mesh_rp.windows.parameter.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.parameters.json).

### <a name="parameters"></a>Paramètres
Si votre modèle contient des valeurs que vous prévoyez de changer une fois l’application déployée, ou que vous souhaitez pouvoir changer d’un déploiement à l’autre (si vous prévoyez de réutiliser ce modèle pour d’autres déploiements), la bonne pratique consiste à paramétrer les valeurs. La bonne façon de procéder consiste à créer une section « parameters » en haut de votre modèle de déploiement, où vous spécifiez les noms et propriétés des paramètres, qui sont ensuite référencés dans le modèle de déploiement. Chaque définition de paramètre inclut les propriétés *type* et *defaultValue*, ainsi qu’une section *metadata* facultative comportant une *description*.

La section parameters est définie au début de votre modèle de déploiement, juste avant la section *ressources* :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
      ...
    },
    "resources": [
```

Dans le [modèle de déploiement mesh_rp.windows.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json), les paramètres suivants sont déclarés : location, registryPassword, registryUserName, registryServer, frontEndImage, serviceImage, frontEndCpu, serviceCpu, frontEndMemory, serviceMemory, frontEndReplicaCount, serviceReplicaCount.  Vous trouverez les descriptions de chaque paramètre dans le fichier de modèle de déploiement.

Ces paramètres sont utilisés dans le fichier de [paramètres mesh_rp.windows.parameter.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.parameters.json).  L’utilisation d’un fichier de paramètres distinct vous permet de changer les valeurs des paramètres d’un déploiement à l’autre sans mettre à jour le modèle de déploiement proprement dit.

### <a name="overview-of-the-application-and-services"></a>Vue d’ensemble de l’application et des services

Les services sont spécifiés dans le modèle en tant que propriétés de la ressource d’application.  Les applications sont déployées sur un réseau privé, qui est déclaré en tant que ressource dans le modèle.  Les services peuvent utiliser des volumes pour conserver les données, qui sont déclarés en tant que ressources dans le modèle.  Pour chaque service, le type de système d’exploitation, le(s) package(s) de code, le nombre de réplicas et le réseau sont spécifiés en tant que propriétés du service.  Pour chaque package de code, spécifiez l’image conteneur, les points de terminaison, les ressources en mémoire et UC, ainsi que les informations d’identification du référentiel d’images. L’aspect général du modèle pour une application Service Fabric Mesh comportant plusieurs services est le suivant :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    ...
  },
  "resources": [
    {
      "apiVersion": "2018-09-01-preview",
      "name": "MyMeshApplication",
      "type": "Microsoft.ServiceFabricMesh/applications",
      "location": "[parameters('location')]",
      "dependsOn": [
        "Microsoft.ServiceFabricMesh/networks/MeshAppNetwork",
        "Microsoft.ServiceFabricMesh/volumes/ServiceAVolume"
      ],
      "properties": {
        "services": [
          {
            "name": "ServiceA",
            "properties": {
              "description": "ServiceA description.",
              "osType": "Linux",
              "codePackages": [
                {
                  "name": "ServiceA",
                  "image": "[parameters('frontEndImage')]",
                  "volumeRefs": [
                    {
                      "name": "[resourceId('Microsoft.ServiceFabricMesh/volumes', 'ServiceAVolume')]",
                      "destinationPath": "/app/data"
                    }
                  ],
                  "endpoints": [
                    {
                      "name": "ServiceAListener",
                      "port": 80
                    }
                  ],
                  "resources": {
                    "requests": {
                      "cpu": "[parameters('frontEndCpu')]",
                      "memoryInGB": "[parameters('frontEndMemory')]"
                    }
                  },
                  "imageRegistryCredential": {
                    "server": "[parameters('registryServer')]",
                    "username": "[parameters('registryUserName')]",
                    "password": "[parameters('registryPassword')]"
                  }
                }
              ],
              "replicaCount": "[parameters('frontEndReplicaCount')]",
              "networkRefs": [
                {
                  "name": "[resourceId('Microsoft.ServiceFabricMesh/networks', 'MeshAppNetwork')]"
                }
              ]
            }
          },
          {
            "name": "ServiceB",
            ...
          }
        ],
        "description": "Application description."
      }
    },
    {
      "apiVersion": "2018-07-01-preview",
      "name": "MeshAppNetwork",
      "type": "Microsoft.ServiceFabricMesh/networks",
      "location": "[parameters('location')]",
      "dependsOn": [],
      "properties": {
        "description": "MeshAppNetwork description.",
        "addressPrefix": "10.0.0.4/22",
        "ingressConfig": {
          "layer4": [
            {
              "name": "ServiceAIngress",
              "publicPort": "20001",
              "applicationName": "MyMeshApplication",
              "serviceName": "ServiceA",
              "endpointName": "ServiceAListener"
            }
          ]
        }
      }
    },
    {
      "apiVersion": "2018-09-01-preview",
      "name": "ServiceAVolume",
      "type": "Microsoft.ServiceFabricMesh/volumes",
      "location": "[parameters('location')]",
      "dependsOn": [],
      "properties": {
        "description": "Azure Files storage volume for counter App.",
        "provider": "SFAzureFile",
        "azureFileParameters": {
          "shareName": "[parameters('fileShareName')]",
          "accountName": "[parameters('storageAccountName')]",
          "accountKey": "[parameters('storageAccountKey')]"
        }
      }
    }
  ]
}
```

Reportez-vous au fichier du [modèle de déploiement mesh_rp.windows.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json) pour connaître les caractéristiques de l’application de liste de tâche.

## <a name="deploy-the-application-to-service-fabric-mesh"></a>Déployer l’application sur Service Fabric Mesh
Créez l’application et les ressources associées à l’aide de la commande suivante, et fournissez les informations d’identification de l’étape précédente [Récupérer les informations d’identification pour le registre](#retrieve-credentials-for-the-registry).

Dans le fichier de paramètres, mettez à jour les valeurs de paramètre suivantes :

|Paramètre|Valeur|
|---|---|
|location|Région dans laquelle déployer l’application.  Par exemple, « eastus ».|
|registryPassword|Mot de passe que vous avez obtenu à l’étape [Récupérer les informations d’identification pour le registre](#retrieve-credentials-for-the-registry). Ce paramètre dans le modèle est une chaîne sécurisée ; il ne s’affiche pas dans l’état du déploiement ou les commandes `az mesh service show`.|
|registryUserName|Nom d’utilisateur que vous avez obtenu à l’étape [Récupérer les informations d’identification pour le registre](#retrieve-credentials-for-the-registry).|
|registryServer|Nom de serveur de registre que vous avez obtenu à l’étape [Récupérer les informations d’identification pour le registre](#retrieve-credentials-for-the-registry).|
|frontEndImage|Image conteneur pour le service frontal.  Par exemple : `<myregistry>.azurecr.io/seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709`.|
|serviceImage|Image conteneur pour le service backend.  Par exemple : `<myregistry>.azurecr.io/seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709`.|

Pour déployer l’application, exécutez la commande suivante :

```azurecli
az mesh deployment create --resource-group myResourceGroup --template-file c:\temp\mesh_rp.windows.json --parameters c:\temp\mesh_rp.windows.parameters.json
```

Cette commande génère l’extrait de code JSON ci-dessous. Sous la section ```outputs``` de la sortie JSON, copiez la propriété ```publicIPAddress```.

```json
"outputs": {
    "publicIPAddress": {
    "type": "String",
    "value": "40.83.78.216"
    }
}
```

Ces informations proviennent de la section ```outputs``` du modèle ARM. Comme illustré ci-dessous, cette section fait référence à la ressource de passerelle qui récupère l’adresse IP publique. 

```json
  "outputs": {
    "publicIPAddress": {
      "value": "[reference('todolistappGateway').ipAddress]",
      "type": "string"
    }
  }
```

## <a name="open-the-application"></a>Ouvrir l’application

À l’issue du déploiement de l’application, obtenez l’adresse IP publique du point de terminaison de service. La commande de déploiement renvoie l’adresse IP publique du point de terminaison de service. Si vous le souhaitez, vous pouvez également interroger la ressource réseau pour trouver l’adresse IP publique du point de terminaison de service. Le nom de la ressource réseau pour cette application est `todolistappNetwork`. Extrayez les informations la concernant à l’aide de la commande suivante. 

```azurecli
az mesh gateway show --resource-group myResourceGroup --name todolistappGateway
```

Accédez à l’adresse IP dans un navigateur web.

## <a name="check-application-status"></a>Vérifier l’état de l’application

Vous pouvez vérifier l’état de l’application à l’aide de la commande app show. Le nom d’application pour l’application déployée est « todolistapp ». Donc, extrayez ses détails.

```azurecli
az mesh app show --resource-group myResourceGroup --name todolistapp
```

Examinez les journaux d’activité de l’application déployée en utilisant la commande `az mesh code-package-log get` :
```azurecli
az mesh code-package-log get --resource-group myResourceGroup --application-name todolistapp --service-name WebFrontEnd --replica-name 0 --code-package-name WebFrontEnd
```

## <a name="next-steps"></a>Étapes suivantes

Dans cette partie du tutoriel, vous avez appris à :

> [!div class="checklist"]
> * Créer un registre de conteneurs privé
> * Envoyer une image de conteneur vers le Registre
> * Créer un fichier de modèle et de paramètres
> * Déployer une application sur Service Fabric Mesh

Passez au tutoriel suivant :
> [!div class="nextstepaction"]
> [Mettre à l’échelle une application en cours d’exécution dans Service Fabric Mesh](service-fabric-mesh-tutorial-template-scale-services.md)