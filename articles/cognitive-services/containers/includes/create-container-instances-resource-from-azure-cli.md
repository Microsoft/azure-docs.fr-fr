---
title: Support pour les conteneurs
titleSuffix: Azure Cognitive Services
description: Découvrez comment créer une ressource d’instance de conteneur Azure depuis Azure CLI.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 903c36bc228faaac4cbf13fd00ca1b398e2f8add
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2020
ms.locfileid: "97704836"
---
## <a name="create-an-azure-container-instance-resource-from-the-azure-cli"></a>Créer une ressource d’instance de conteneur Azure depuis Azure CLI

Le YAML ci-dessous définit la ressource Azure Container Instance. Copiez et collez le contenu dans un nouveau fichier nommé `my-aci.yaml`, et remplacez les valeurs commentés par les vôtres. Reportez-vous au [format de modèle][template-format] pour obtenir du YAML valide. Reportez-vous à [Référentiels et images de conteneur][repositories-and-images] pour les noms d’image disponibles et leur référentiel correspondant. Pour plus d’informations sur la référence YAML pour les instances de conteneur, consultez [Référence YAML : Azure Container Instances][aci-yaml-ref].

```YAML
apiVersion: 2018-10-01
location: # < Valid location >
name: # < Container Group name >
properties:
  imageRegistryCredentials: # This is only required if you are pulling a non-public image that requires authentication to access. For example Text Analytics for health.
  - server: containerpreview.azurecr.io
    username: # < The username for the preview container registry >
    password: # < The password for the preview container registry >
  containers:
  - name: # < Container name >
    properties:
      image: # < Repository/Image name >
      environmentVariables: # These env vars are required
        - name: eula
          value: accept
        - name: billing
          value: # < Service specific Endpoint URL >
        - name: apikey
          value: # < Service specific API key >
      resources:
        requests:
          cpu: 4 # Always refer to recommended minimal resources
          memoryInGb: 8 # Always refer to recommended minimal resources
      ports:
        - port: 5000
  osType: Linux
  volumes: # This node, is only required for container instances that pull their model in at runtime, such as LUIS.
  - name: aci-file-share
    azureFile:
      shareName: # < File share name >
      storageAccountName: # < Storage account name>
      storageAccountKey: # < Storage account key >
  restartPolicy: OnFailure
  ipAddress:
    type: Public
    ports:
    - protocol: tcp
      port: 5000
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

> [!NOTE]
> Tous les emplacements n’ont pas les mêmes offres de processeur et de mémoire disponibles. Reportez-vous au tableau [Emplacement et ressources][location-to-resource] pour obtenir la liste des ressources disponibles pour les conteneurs par emplacement et par système d’exploitation.

Nous allons nous reposer sur le fichier YAML que nous avons créé pour la commande [`az container create`][azure-container-create]. À partir de l’interface CLI, exécutez la commande `az container create` en remplaçant le `<resource-group>` par le vôtre. En outre, pour la sécurisation des valeurs au sein d’un déploiement YAML, consultez [Valeurs sécurisées][secure-values].

```azurecli
az container create -g <resource-group> -f my-aci.yaml
```

La sortie de la commande est `Running...` si valide. Un peu après, la sortie devient une chaîne JSON représentant la ressource ACI nouvellement créée. L’image de conteneur ne sera probablement pas accessible pendant un certain temps, mais la ressource est maintenant déployée.

> [!TIP]
> Prêtez attention aux emplacements des offres d’Azure Cognitive Service en préversion, car le YAML devra être ajusté en conséquence pour correspondre à l’emplacement.

[azure-container-create]: /cli/azure/container#az-container-create
[template-format]: /azure/templates/Microsoft.ContainerInstance/2018-10-01/containerGroups#template-format
[aci-yaml-ref]: ../../../container-instances/container-instances-reference-yaml.md
[repositories-and-images]: ../../cognitive-services-container-support.md#container-repositories-and-images
[location-to-resource]: ../../../container-instances/container-instances-region-availability.md
[secure-values]: ../../../container-instances/container-instances-environment-variables.md#secure-values
