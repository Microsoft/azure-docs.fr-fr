---
title: Écrire des createOptions pour des modules - Azure IoT Edge | Microsoft Docs
description: Explique comment utiliser createOptions dans le manifeste de déploiement pour configurer des modules au moment de l’exécution.
keywords: ''
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 04/01/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: c07e161042a497a232cbd5e3f11128893a095381
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80576744"
---
# <a name="how-to-configure-container-create-options-for-iot-edge-modules"></a>Guide pratique pour configurer les options de création de conteneur pour les modules IoT Edge

Le paramètre **createOptions** du manifeste de déploiement vous permet de configurer les conteneurs de module au moment de l’exécution. Ce paramètre étend votre contrôle sur les modules et permet d’effectuer des tâches telles que l’autorisation ou la restriction de l’accès du module aux ressources de l’appareil hôte, ou la configuration du réseau.

Les modules IoT Edge sont implémentés en tant que conteneurs compatibles avec Docker sur votre appareil IoT Edge. Docker offre de nombreuses options pour créer des conteneurs, et ces options s’appliquent également aux modules IoT Edge. Pour plus d’informations, consultez les [options de création de conteneur Docker](https://docs.docker.com/engine/api/v1.32/#operation/ContainerCreate).

## <a name="format-create-options"></a>Format des options de création

Le manifeste de déploiement IoT Edge accepte les options de création au format JSON. Par exemple, prenez les options de création qui sont automatiquement incluses pour chaque module edgeHub :

```json
"createOptions": {
  "HostConfig": {
    "PortBindings": {
      "5671/tcp": [
        {
          "HostPort": "5671"
        }
      ],
      "8883/tcp": [
        {
          "HostPort": "8883"
        }
      ],
      "443/tcp": [
        {
          "HostPort": "443"
        }
      ]
    }
  }
}
```

Cet exemple edgeHub utilise le paramètre **HostConfig.PortBindings** pour mapper les ports exposés sur le conteneur à un port sur l’appareil hôte.

Si vous utilisez les extensions Azure IoT Tools pour Visual Studio ou Visual Studio Code, vous pouvez écrire les options de création au format JSON dans le fichier **deployment.template.json**. Ensuite, quand vous utiliserez l’extension pour générer la solution IoT Edge ou le manifeste de déploiement, elle stringifiera le JSON pour vous dans le format attendu par le runtime IoT Edge. Par exemple :

```json
"createOptions": "{\"HostConfig\":{\"PortBindings\":{\"5671/tcp\":[{\"HostPort\":\"5671\"}],\"8883/tcp\":[{\"HostPort\":\"8883\"}],\"443/tcp\":[{\"HostPort\":\"443\"}]}}}"
```

L’une des astuces pour écrire des options de création consiste à utiliser la commande `docker inspect`. Dans le cadre de votre processus de développement, exécutez le module localement à l’aide de `docker run <container name>`. Une fois que le module fonctionne comme vous le souhaitez, exécutez `docker inspect <container name>`. Cette commande génère les détails du module au format JSON. Recherchez les paramètres que vous avez configurés et copiez le JSON. Par exemple :

[ ![Résultats de l’inspection docker edgeHub](./media/how-to-use-create-options/docker-inspect-edgehub-inline-and-expanded.png) ](./media/how-to-use-create-options/docker-inspect-edgehub-inline-and-expanded.png#lightbox)

## <a name="common-scenarios"></a>Scénarios courants

Les options de création de conteneur autorisent de nombreux scénarios ; en voici quelques-uns parmi les plus courants lors de la création de solutions IoT Edge :

* [Autoriser les modules à accéder au stockage hôte](how-to-access-host-storage-from-module.md)
* [Mapper un port hôte à un port de module](#map-host-port-to-module-port)
* [Restreindre l’utilisation de la mémoire et du processeur du module](#restrict-module-memory-and-cpu-usage)

### <a name="map-host-port-to-module-port"></a>Mapper un port hôte à un port de module

Si votre module doit communiquer avec un service en dehors de la solution IoT Edge et qu’il n’utilise pas pour cela le routage des messages, vous devez mapper un port hôte à un port de module.

>[!TIP]
>Ce mappage de ports n’est pas nécessaire pour la communication entre modules sur le même appareil. Si le module A doit interroger une API hébergée sur le module B, il peut le faire sans aucun mappage de ports. Le module B doit exposer un port dans son fichier dockerfile, par exemple : `EXPOSE 8080`. Ensuite, le module A peut interroger l’API à l’aide du nom du module B, par exemple : `http://ModuleB:8080/api`.

Tout d’abord, assurez-vous qu’un port à l’intérieur du module est exposé pour écouter les connexions. Vous pouvez pour cela utiliser une instruction [EXPOSE](https://docs.docker.com/engine/reference/builder/#expose) dans le fichier dockerfile. Par exemple : `EXPOSE 8080`. L’instruction EXPOSE utilise par défaut le protocole TCP s’il n’est pas spécifié, ou vous pouvez spécifier UDP.

Ensuite, utilisez le paramètre **PortBindings** dans le groupe **HostConfig** des [options de création de conteneur Docker](https://docs.docker.com/engine/api/v1.32/#operation/ContainerCreate) pour mapper le port exposé dans le module à un port sur l’appareil hôte. Par exemple, si vous avez exposé le port 8080 dans le module et que vous souhaitez le mapper au port 80 de l’appareil hôte, les options de création dans le fichier template.json se présenteront comme dans l’exemple suivant :

```json
"createOptions": {
  "HostConfig": {
    "PortBindings": {
      "8080/tcp": [
        {
          "HostPort": "80"
        }
      ]
    }
  }
}
```

Une fois stringifiée pour le manifeste de déploiement, la même configuration ressemblerait à l’exemple suivant :

```json
"createOptions": "{\"HostConfig\":{\"PortBindings\":{\"8080/tcp\":[{\"HostPort\":\"80\"}]}}}"
```

### <a name="restrict-module-memory-and-cpu-usage"></a>Restreindre l’utilisation de la mémoire et du processeur du module

Vous pouvez déclarer la quantité de ressources hôte qu’un module peut utiliser. Ce contrôle est utile pour s’assurer qu’un module ne peut pas consommer trop de mémoire ou de ressources de processeur, et pour empêcher d’autres processus de s’exécuter sur l’appareil. Vous pouvez gérer ces paramètres avec les [options de création de conteneur Docker](https://docs.docker.com/engine/api/v1.32/#operation/ContainerCreate) dans le groupe **HostConfig**, notamment :

* **Memory** : limite de mémoire en octets. Par exemple, 268435456 octets = 256 Mo.
* **MemorySwap** : limite totale de la mémoire (mémoire + échange). Par exemple, 536870912 octets = 512 Mo
* **CpuPeriod** : durée d’une période de processeur en microsecondes. La valeur par défaut est 100000. Par exemple, la valeur 25000 limite un conteneur à 25 % des ressources du processeur.

Dans le fichier template.json, ces valeurs ressembleraient à l’exemple suivant :

```json
"createOptions": {
  "HostConfig": {
    "Memory": 268435456,
    "MemorySwap": 536870912,
    "CpuPeriod": 25000
  }
}
```

Une fois stringifiées pour le manifeste de déploiement final, ces valeurs se présenteraient comme dans l’exemple suivant :

```json
"createOptions":"{\"HostConfig\":{\"Memory\":268435456,\"MemorySwap\":536870912,\"CpuPeriod\":25000}}"
```

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir plus d’exemples d’options de création en action, consultez les exemples IoT Edge suivants :

* [Custom Vision et Azure IoT Edge sur un Raspberry Pi 3](https://github.com/Azure-Samples/Custom-vision-service-iot-edge-raspberry-pi)
* [Exemple de stockage d’objets blob Azure IoT Edge](https://github.com/Azure-Samples/azure-iotedge-blobstorage-sample)
