---
title: Tutoriel - Mettre à l’échelle une application s’exécutant dans Azure Service Fabric Mesh
description: Dans ce didacticiel, vous allez découvrir comment mettre à l’échelle les services d’une application en cours d’exécution dans Service Fabric Mesh.
author: georgewallace
ms.topic: tutorial
ms.date: 01/11/2019
ms.author: gwallace
ms.custom: mvc, devcenter, devx-track-azurecli
ms.openlocfilehash: df28083a0522178b7327d9f6d24029d303e417a1
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92747870"
---
# <a name="tutorial-scale-an-application-running-in-service-fabric-mesh"></a>Tutoriel : Mettre à l’échelle une application en cours d’exécution dans Service Fabric Mesh

Ce tutoriel est le deuxième d’une série. Découvrez comment mettre à l’échelle manuellement le nombre d’instances de service d’une application qui a été [déployée sur Service Fabric Mesh](service-fabric-mesh-tutorial-template-deploy-app.md). Après avoir terminé, vous disposerez d’un service frontal exécutant trois instances et d’un service de données exécutant deux instances.

Dans ce deuxième volet, vous apprenez à :

> [!div class="checklist"]
> * Configurer le nombre souhaité d’instances de service
> * Exécuter une mise à niveau

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * [Déployer une application sur Service Fabric Mesh à l’aide d’un modèle](service-fabric-mesh-tutorial-template-deploy-app.md)
> * Mettre à l’échelle une application en cours d’exécution dans Service Fabric Mesh
> * [Mettre à niveau une application en cours d’exécution dans Service Fabric Mesh](service-fabric-mesh-tutorial-template-upgrade-app.md)
> * [Supprimer une application](service-fabric-mesh-tutorial-template-remove-app.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel :

* Si vous n’avez pas d’abonnement Azure, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

* [Installez localement l’interface de ligne de commande Azure et l’interface de ligne de commande Service Fabric Mesh](service-fabric-mesh-howto-setup-cli.md#install-the-azure-service-fabric-mesh-cli).

## <a name="manually-scale-your-services-in-or-out"></a>Augmenter ou diminuer manuellement la taille de vos services

Le déploiement d’applications vers Service Fabric Mesh permet notamment d’augmenter ou de diminuer la taille de vos services en toute simplicité. Vous pouvez ainsi gérer des charges variables sur vos services ou améliorer la disponibilité.

Ce tutoriel utilise l’exemple de liste de tâches, qui a été [déployé au préalable](service-fabric-mesh-tutorial-template-deploy-app.md) et doit être en cours d’exécution. L’application a deux services : WebFrontEnd et ToDoService. Chaque service a été initialement déployé avec 1 réplica.  Si vous souhaitez afficher le nombre de réplicas en cours d’exécution pour le service WebFrontEnd, exécutez la commande suivante :

```azurecli
az mesh service show --resource-group myResourceGroup --name WebFrontEnd --app-name todolistapp --query "replicaCount"
```

Si vous souhaitez afficher le nombre de réplicas en cours d’exécution pour le service ToDoService, exécutez la commande suivante :

```azurecli
az mesh service show --resource-group myResourceGroup --name ToDoService --app-name todolistapp --query "replicaCount"
```

Dans le modèle de déploiement de la ressource d’application, chaque service a une propriété *replicaCount* qui peut être utilisée pour définir le nombre de déploiements de ce service. Une application peut se composer de plusieurs services, chaque service disposant d’un nombre *replicaCount* unique, qui sont déployés et gérés ensemble. Pour mettre à l’échelle le nombre de réplicas de service, modifiez la valeur *replicaCount* pour chaque service que vous souhaitez mettre à l’échelle dans le modèle de déploiement ou les fichiers de paramètres.  Mettez ensuite à niveau l’application.

### <a name="modify-the-deployment-template-parameters"></a>Modifier les paramètres du modèle de déploiement

Si votre modèle contient des valeurs que vous prévoyez de changer une fois l’application déployée, ou que vous souhaitez pouvoir changer d’un déploiement à l’autre (si vous prévoyez de réutiliser ce modèle pour d’autres déploiements), la bonne pratique consiste à paramétrer les valeurs.

L’application a été déployée à l’aide des fichiers de [modèle de déploiement mesh_rp.windows.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json) et de [paramètres mesh_rp.windows.parameter.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.parameters.json).

Ouvrez le fichier de [paramètres mesh_rp.windows.parameter.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.parameters.json) localement et définissez la valeur de *frontEndReplicaCount* sur 3 et la valeur de *serviceReplicaCount* sur 2 :

```json
      "frontEndReplicaCount":{
        "value": "3"
      },
      "serviceReplicaCount":{
        "value": "2"
      }
```

Enregistrez les modifications apportées au fichier de paramètres.  Les paramètres *frontEndReplicaCount* et *serviceReplicaCount* sont déclarés dans la section *parameters* du [modèle de déploiement mesh_rp.windows.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json) :

```json
"frontEndReplicaCount":{
      "defaultValue": "1",
      "type": "string"
    },
    "serviceReplicaCount":{
      "defaultValue": "1",
      "type": "string"
    }
```

La propriété *replicaCount* du service WebFrontEnd fait référence au paramètre *frontEndReplicaCount* , tandis que la propriété *replicaCount* du service ToDoService fait référence au paramètre *serviceReplicaCount* :

```json
    "services": [
    {
    "name": "WebFrontEnd",
    "properties": {
        "description": "WebFrontEnd description.",
        "osType": "Windows",
        "codePackages": [
        {
            ...
        }
        ],
        "replicaCount": "[parameters('frontEndReplicaCount')]",
        "networkRefs": [
        {
            "name": "[resourceId('Microsoft.ServiceFabricMesh/networks', 'todolistappNetwork')]"
        }
        ]
    }
    },
    {
    "name": "ToDoService",
    "properties": {
        "description": "ToDoService description.",
        "osType": "Windows",
        "codePackages": [
        {
            ...
        }
        ],
        "replicaCount": "[parameters('serviceReplicaCount')]",
        "networkRefs": [
        {
            "name": "[resourceId('Microsoft.ServiceFabricMesh/networks', 'todolistappNetwork')]"
        }
        ]
    }
    }
],
```

Une fois que votre modèle a été modifié, mettez à niveau votre application.

### <a name="upgrade-your-application"></a>Mettre à niveau votre application

Pendant son exécution, vous pouvez mettre à niveau l’application en redéployant le modèle et le fichier de paramètres mis à jour :

```azurecli
az mesh deployment create --resource-group myResourceGroup --template-file c:\temp\mesh_rp.windows.json --parameters c:\temp\mesh_rp.windows.parameters.json
```

Cette commande démarre une mise à niveau propagée vers votre application, et les instances de service devraient augmenter en quelques minutes.  Si vous souhaitez afficher le nombre de réplicas en cours d’exécution pour le service WebFrontEnd, exécutez la commande suivante :

```azurecli
az mesh service show --resource-group myResourceGroup --name WebFrontEnd --app-name todolistapp --query "replicaCount"
```

Si vous souhaitez afficher le nombre de réplicas en cours d’exécution pour le service ToDoService, exécutez la commande suivante :

```azurecli
az mesh service show --resource-group myResourceGroup --name ToDoService --app-name todolistapp --query "replicaCount"
```

## <a name="next-steps"></a>Étapes suivantes

Dans cette partie du tutoriel, vous avez appris à :

> [!div class="checklist"]
> * Configurer le nombre souhaité d’instances de service
> * Exécuter une mise à niveau

Passez au tutoriel suivant :
> [!div class="nextstepaction"]
> [Mettre à niveau une application en cours d’exécution dans Service Fabric Mesh](service-fabric-mesh-tutorial-template-upgrade-app.md)
