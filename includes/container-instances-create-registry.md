---
title: Fichier include
description: Fichier include
services: container-instances
author: dlepow
ms.service: container-instances
ms.topic: include
ms.date: 08/13/2020
ms.author: danlep
ms.custom: include file, devx-track-azurecli
ms.openlocfilehash: 173c9156f253e43111299b53287e97ab7b2c0aa5
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92746954"
---
## <a name="create-azure-container-registry"></a>Créer un registre de conteneurs Azure

Avant de créer votre registre de conteneurs, vous avez besoin d’un *groupe de ressources* vers lequel le déployer. Un groupe de ressources est une collection logique dans laquelle toutes les ressources Azure sont déployées et gérées.

Créez un groupe de ressources avec la commande [az group create][az-group-create]. Dans l’exemple suivant, un groupe de ressources nommé *myResourceGroup* est créé dans la région *eastus* :

```azurecli
az group create --name myResourceGroup --location eastus
```

Une fois que vous avez créé le groupe de ressources, créez un registre de conteneurs Azure avec la commande [az acr create][az-acr-create]. Le nom du registre de conteneurs doit être unique dans Azure et contenir de 5 à 50 caractères alphanumériques. Remplacez `<acrName>` par un nom unique pour votre registre :

```azurecli
az acr create --resource-group myResourceGroup --name <acrName> --sku Basic
```

Voici une sortie partielle pour un nouveau registre de conteneurs Azure nommé *mycontainerregistry082* :

```output
{
  "creationDate": "2020-07-16T21:54:47.297875+00:00",
  "id": "/subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/mycontainerregistry082",
  "location": "eastus",
  "loginServer": "mycontainerregistry082.azurecr.io",
  "name": "mycontainerregistry082",
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

Dans le reste de ce didacticiel, `<acrName>` désigne l’espace réservé du nom du registre des conteneurs que vous choisissez dans cette étape.

## <a name="log-in-to-container-registry"></a>Se connecter au registre de conteneurs

Vous devez vous connecter à votre instance Azure Container Registry avant de lui envoyer des images. Utilisez la commande [az acr login][az-acr-login] pour terminer l’opération. Vous devez fournir le nom unique que vous avez choisi pour le registre de conteneurs au moment de sa création.

```azurecli
az acr login --name <acrName>
```

Par exemple :

```azurecli
az acr login --name mycontainerregistry082
```

Une fois l’opération terminée, la commande renvoie `Login Succeeded` :

```output
Login Succeeded
```

<!-- LINKS - Internal -->
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-group-create]: /cli/azure/group#az-group-create
