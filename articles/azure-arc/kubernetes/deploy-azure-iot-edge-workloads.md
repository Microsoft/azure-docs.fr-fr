---
title: Déployer des charges de travail Azure IoT Edge (préversion)
services: azure-arc
ms.service: azure-arc
ms.date: 05/19/2020
ms.topic: article
author: mlearned
ms.author: mlearned
description: Déployer des charges de travail Azure IoT Edge
keywords: Kubernetes, Arc, Azure, K8s, conteneurs
ms.openlocfilehash: bfaa43a03ddd98616b22fc3fc7b4dccb4c38f44c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86103974"
---
# <a name="deploy-azure-iot-edge-workloads-preview"></a>Déployer des charges de travail Azure IoT Edge (préversion)

## <a name="overview"></a>Vue d’ensemble

Azure Arc et Azure IoT Edge se complètent à merveille. Azure Arc fournit des mécanismes pour les opérateurs de cluster permettant de configurer les principaux composants d’un cluster, ainsi que d’appliquer des stratégies de cluster. IoT Edge permet aux opérateurs d’application de déployer et de gérer les charges de travail, à distance et à grande échelle, grâce à une ingestion cloud pratique et à des primitives de communication bidirectionnelle. Le diagramme ci-dessous en offre une illustration :

![Configuration IoT Arc](./media/edge-arc.png)

## <a name="pre-requisites"></a>Conditions préalables

* [Inscrivez un appareil IoT Edge](../../iot-edge/quickstart-linux.md#register-an-iot-edge-device) et [déployez le module de capteur de température simulé](../../iot-edge/quickstart-linux.md#deploy-a-module). Veillez à noter la chaîne de connexion de l’appareil.

* Utilisez la [prise en charge IoT Edge pour Kubernetes](https://aka.ms/edgek8sdoc) afin de le déployer par le biais de l’opérateur Flux d’Azure Arc.

* Téléchargez le fichier [**values.yaml**](https://github.com/Azure/iotedge/blob/master/kubernetes/charts/edge-kubernetes/values.yaml) pour le chart Helm IoT Edge, et remplacez l’espace réservé **deviceConnectionString** situé à la fin du fichier par celui noté à l’étape 1. Vous pouvez définir toutes les autres options d’installation de graphique prises en charge, si nécessaire. Créez un espace de noms pour la charge de travail IoT Edge, puis créez-y un secret :

    ```
    $ kubectl create ns iotedge

    $ kubectl create secret generic dcs --from-file=fully-qualified-path-to-values.yaml --namespace iotedge
    ```

    Vous pouvez également procéder à cette configuration à distance en utilisant l’[exemple de configuration de cluster](./use-gitops-connected-cluster.md).

## <a name="connect-a-cluster"></a>Connecter un cluster

Utilisez l’extension `connectedk8s` de l’interface CLI `az` pour connecter un cluster Kubernetes à Azure Arc :

  ```
  az connectedk8s connect --name AzureArcIotEdge --resource-group AzureArcTest
  ```

## <a name="create-a-configuration-for-iot-edge"></a>Créer une configuration pour IoT Edge

Exemple de dépôt : https://github.com/veyalla/edgearc

Ce dépôt pointe vers le chart Helm IoT Edge et référence le secret créé dans la section concernant les prérequis.

1. Utilisez l’extension `k8sconfiguration` de l’interface CLI `az` pour créer une configuration en vue de lier le cluster connecté au dépôt git :

    ```
    az k8sconfiguration create --name iotedge --cluster-name AzureArcIotEdge --resource-group AzureArcTest --operator-instance-name iotedge --operator-namespace azure-arc-iot-edge --enable-helm-operator --helm-operator-chart-version 0.6.0 --helm-operator-chart-values "--set helm.versions=v3" --repository-url "git://github.com/veyalla/edgearc.git" --cluster-scoped
    ```

    Dans une minute ou deux, vous devriez voir les modules de charge de travail IoT Edge déployés dans l’espace de noms `iotedge` de votre cluster. Vous pouvez afficher les journaux du pod `SimulatedTemperatureSensor` dans cet espace de noms pour voir les valeurs qui sont générées. Vous pouvez également regarder les messages arriver sur votre hub IoT avec l’[extension Azure IoT Hub Toolkit pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit).

## <a name="cleanup"></a>Nettoyage

Vous pouvez supprimer la configuration avec ceci :

```
az k8sconfiguration delete -g AzureArcTest --cluster-name AzureArcIotEdge --name iotedge
```

## <a name="next-steps"></a>Étapes suivantes

[Utiliser Azure Policy pour gérer la configuration du cluster](./use-azure-policy.md)
