---
title: Tutoriel - Mettre à niveau une application s’exécutant dans Azure Service Fabric Mesh
description: Dans ce tutoriel, vous allez apprendre à mettre à niveau une application Service Fabric s’exécutant dans Service Fabric Mesh.
author: georgewallace
ms.topic: tutorial
ms.date: 01/11/2019
ms.author: gwallace
ms.custom: mvc, devcenter, devx-track-azurecli
ms.openlocfilehash: 331830c50206d14f7894aa837b483656de4222f2
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92747855"
---
# <a name="tutorial-upgrade-a-service-fabric-application-running-in-service-fabric-mesh"></a>Tutoriel : Mettre à niveau une application Service Fabric s’exécutant dans Service Fabric Mesh

Ce tutoriel est le troisième de la série. Vous allez apprendre à mettre à niveau une application Service Fabric [déployée sur Service Fabric Mesh](service-fabric-mesh-tutorial-template-deploy-app.md) en augmentant les ressources d’UC allouées.  Une fois l’opération effectuée, vous disposez d’un service web front-end qui s’exécute avec des ressources d’UC plus importantes.

Dans ce troisième volet, vous apprenez à :

> [!div class="checklist"]
> * Changer les configurations d’application
> * Mettre à niveau une application s’exécutant dans Service Fabric Mesh

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * [Déployer une application sur Service Fabric Mesh à l’aide d’un modèle](service-fabric-mesh-tutorial-template-deploy-app.md)
> * [Mettre à l’échelle une application s’exécutant dans Service Fabric Mesh](service-fabric-mesh-tutorial-template-scale-services.md)
> * Mettre à niveau une application s’exécutant dans Service Fabric Mesh
> * [Supprimer une application](service-fabric-mesh-tutorial-template-remove-app.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer ce tutoriel :

* Si vous n’avez pas d’abonnement Azure, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

* Ouvrez [Azure Cloud Shell](service-fabric-mesh-howto-setup-cli.md) ou [installez localement Azure CLI et l’interface CLI Service Fabric Mesh](service-fabric-mesh-howto-setup-cli.md#install-the-azure-service-fabric-mesh-cli).

## <a name="upgrade-application-configurations"></a>Mettre à niveau les configurations d’application

L’un des principaux avantages du déploiement d’applications sur Service Fabric Mesh est la facilité avec laquelle vous pouvez mettre à jour la configuration de votre application.  C’est le cas, par exemple, pour les ressources d’UC ou de mémoire de vos services.

Ce tutoriel utilise l’exemple de liste de tâches, qui a été [déployé au préalable](service-fabric-mesh-tutorial-template-deploy-app.md) et doit être en cours d’exécution. L’application a deux services : WebFrontEnd et ToDoService. Chaque service a été initialement déployé avec des ressources d’UC de l’ordre de 0,5.  Pour afficher les ressources d’UC du service WebFrontEnd, exécutez ce qui suit :

```azurecli
az mesh service show --resource-group myResourceGroup --name WebFrontEnd --app-name todolistapp
```

Dans le modèle de déploiement de la ressource d’application, chaque service a une propriété *cpu* qui permet de définir les ressources d’UC demandées. Une application peut comprendre plusieurs services (chacun disposant d’un seul paramètre *cpu* ) déployés et gérés ensemble. Pour augmenter les ressources d’UC du service web front-end, modifiez la valeur *cpue* dans le fichier de modèle de déploiement ou le fichier de paramètres.  Mettez ensuite à niveau l’application.

### <a name="modify-the-deployment-template-parameters"></a>Modifier les paramètres du modèle de déploiement

Si votre modèle contient des valeurs que vous prévoyez de changer une fois l’application déployée, ou que vous souhaitez pouvoir changer d’un déploiement à l’autre (si vous prévoyez de réutiliser ce modèle pour d’autres déploiements), la bonne pratique consiste à paramétrer les valeurs.

L’application a été déployée à l’aide des fichiers de [modèle de déploiement mesh_rp.windows.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json) et de [paramètres mesh_rp.windows.parameter.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.parameters.json).

Ouvrez le fichier de paramètres [mesh_rp.windows.parameter.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.parameters.json) localement, puis affectez la valeur 1 à *frontEndCpu*  :

```json
      "frontEndCpu":{
        "value": "1"
      }
```

Enregistrez les modifications apportées au fichier de paramètres.  

Le paramètre *frontEndCpu* est déclaré dans la section *parameters* du modèle de déploiement [mesh_rp.windows.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json) :

```json
"frontEndCpu": {
    "defaultValue": "0.5",
    "type": "string",
    "metadata": {
        "description": "The CPU resources for the front end web service."
    }
}
```

La propriété *codePackages->ressources->requêtes->uc* du service WebFrontEnd référence le paramètre *frontEndCpu*  :

```json
    "services": [
          {
            "name": "WebFrontEnd",
            "properties": {
              "description": "WebFrontEnd description.",
              "osType": "Windows",
              "codePackages": [
                {
                  "name": "WebFrontEnd",
                  "image": "[parameters('frontEndImage')]",
                  ...
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
              ...
```

### <a name="upgrade-your-application"></a>Mettre à niveau votre application

Pendant son exécution, vous pouvez mettre à niveau l’application en redéployant le modèle et le fichier de paramètres mis à jour :

```azurecli
az mesh deployment create --resource-group myResourceGroup --template-file c:\temp\mesh_rp.windows.json --parameters c:\temp\mesh_rp.windows.parameters.json
```

Cela entraîne une mise à niveau propagée de votre application. Ainsi, vous devez voir les ressources d’UC augmenter en quelques minutes.  Pour afficher les ressources d’UC du service WebFrontEnd, exécutez ce qui suit :

```azurecli
az mesh service show --resource-group myResourceGroup --name WebFrontEnd --app-name todolistapp
```

## <a name="next-steps"></a>Étapes suivantes

Dans cette partie du tutoriel, vous avez appris à :

> [!div class="checklist"]
> * Changer les configurations d’application
> * Mettre à niveau une application s’exécutant dans Service Fabric Mesh

Passez au tutoriel suivant :
> [!div class="nextstepaction"]
> [Supprimer une application Service Fabric Mesh](service-fabric-mesh-tutorial-template-remove-app.md)
