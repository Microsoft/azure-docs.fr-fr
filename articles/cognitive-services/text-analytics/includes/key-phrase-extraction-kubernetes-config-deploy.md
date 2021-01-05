---
title: Étapes de configuration et de déploiement Kubernetes du conteneur Extraction de phrases clés
titleSuffix: Azure Cognitive Services
description: Étapes de configuration et de déploiement Kubernetes du conteneur Extraction de phrases clés
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 27c78566877f27e80ae5ae27c5250f228c7ae676
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017853"
---
### <a name="deploy-the-key-phrase-extraction-container-to-an-aks-cluster"></a>Déployer le conteneur Extraction de phrases clés vers un cluster AKS

1. Ouvrir Azure CLI et se connecter à Azure.

    ```azurecli
    az login
    ```

1. Connectez-vous au cluster AKS. Remplacez `your-cluster-name` et `your-resource-group` par les valeurs appropriées.

    ```azurecli
    az aks get-credentials -n your-cluster-name -g -your-resource-group
    ```

    Une fois cette commande exécutée, elle signale un message similaire au message suivant :

    ```output
    Merged "your-cluster-name" as current context in /home/username/.kube/config
    ```

    > [!WARNING]
    > Si plusieurs abonnements sont disponibles sur votre compte Azure et que la commande `az aks get-credentials` renvoie une erreur, le problème peut être que vous n’utilisez pas l’abonnement approprié. Définissez le contexte de votre session Azure CLI pour utiliser le même abonnement avec lequel vous avez créé les ressources, puis réessayez.
    > ```azurecli
    >  az account set -s subscription-id
    > ```

1. Ouvrez l’éditeur de texte de votre choix. L’exemple suivant utilise Visual Studio Code.

    ```console
    code .
    ```

1. Dans l’éditeur de texte, créez un nouveau fichier nommé *keyphrase.yaml* et collez-y le fichier YAML suivant. Assurez-vous de remplacer les valeurs `billing/value` et `apikey/value` par vos propres informations.

    ```yaml
    apiVersion: apps/v1beta1
    kind: Deployment
    metadata:
      name: keyphrase
    spec:
      template:
        metadata:
          labels:
            app: keyphrase-app
        spec:
          containers:
          - name: keyphrase
            image: mcr.microsoft.com/azure-cognitive-services/keyphrase
            ports:
            - containerPort: 5000
            resources:
              requests:
                memory: 2Gi
                cpu: 1
              limits:
                memory: 4Gi
                cpu: 1
            env:
            - name: EULA
              value: "accept"
            - name: billing
              value: # {ENDPOINT_URI}
            - name: apikey
              value: # {API_KEY}
     
    --- 
    apiVersion: v1
    kind: Service
    metadata:
      name: keyphrase
    spec:
      type: LoadBalancer
      ports:
      - port: 5000
      selector:
        app: keyphrase-app
    ```

1. Enregistrez le fichier et fermez l’éditeur de texte.
1. Exécutez la commande Kubernetes `apply` avec le fichier *keyphrase.yaml* comme cible :

    ```console
    kubectl apply -f keyphrase.yaml
    ```

    Une fois la configuration de déploiement correctement appliquée par la commande, un message similaire à la sortie suivante s’affiche :

    ```output
    deployment.apps "keyphrase" created
    service "keyphrase" created
    ```
1. Vérifiez que le pod a été déployé :

    ```console
    kubectl get pods
    ```

    Vous obtiendrez ainsi l’état d’exécution du pod :

    ```output
    NAME                         READY     STATUS    RESTARTS   AGE
    keyphrase-5c9ccdf575-mf6k5   1/1       Running   0          1m
    ```

1. Vérifiez que le service est disponible et obtenez l’adresse IP.

    ```console
    kubectl get services
    ```

    Sortie de l’état d’exécution du service *KeyPhrase* dans le pod :

    ```output
    NAME         TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)          AGE
    kubernetes   ClusterIP      10.0.0.1      <none>           443/TCP          2m
    keyphrase    LoadBalancer   10.0.100.64   168.61.156.180   5000:31234/TCP   2m
    ```
