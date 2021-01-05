---
title: Exécuter le conteneur Détection de langue dans Kubernetes Service
titleSuffix: Text Analytics -  Azure Cognitive Services
description: Déployez le conteneur Détection de langue avec un exemple opérationnel sur Azure Kubernetes Service et testez-le dans un navigateur web.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 2a6e7ee125cac66b48e666a766c39813aa38fb1a
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968500"
---
# <a name="deploy-the-text-analytics-language-detection-container-to-azure-kubernetes-service"></a>Déployer le conteneur Détection de langue Analyse de texte sur Azure Kubernetes Service

Découvrez comment déployer le conteneur Détection de langue. Cette procédure vous montre comment créer les conteneurs Docker locaux, envoyer (push) les conteneurs à votre propre registre de conteneurs privés, exécuter le conteneur dans un cluster Kubernetes et le tester dans un navigateur web.

## <a name="prerequisites"></a>Prérequis

Cette procédure nécessite plusieurs outils qui doivent être installés et exécutés localement. N’utilisez pas Azure Cloud Shell.

* Utilisez un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services) avant de commencer.
* [Git](https://git-scm.com/downloads) pour votre système d’exploitation, de façon à pouvoir cloner l’[exemple](https://github.com/Azure-Samples/cognitive-services-containers-samples) utilisé dans cette procédure.
* [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest).
* [Moteur docker](https://www.docker.com/products/docker-engine) ; vérifiez que l’interface CLI Docker fonctionne dans une fenêtre de console.
* [kubectl](https://storage.googleapis.com/kubernetes-release/release/v1.13.1/bin/windows/amd64/kubectl.exe).
* Une ressource Azure avec le niveau de tarification approprié. Certains niveaux tarifaires ne fonctionnent pas avec ce conteneur :
  * La ressource **Analytique de texte** uniquement avec les niveaux tarifaires F0 ou Standard.
  * La ressource **Cognitive Services** avec un niveau tarifaire S0.

## <a name="running-the-sample"></a>Exécution de l’exemple

Cette procédure charge et exécute l’exemple de conteneur Cognitive Services pour la détection de langue. L’exemple a deux conteneurs : un pour l’application cliente et un pour le conteneur Cognitive Services. Nous enverrons (push) ces deux images à Azure Container Registry. Une fois qu’ils se trouvent sur votre propre registre, créez un service Azure Kubernetes pour accéder à ces images et exécuter les conteneurs. Quand les conteneurs sont en cours d’exécution, utilisez l’interface CLI **kubectl** pour observer les performances des conteneurs. Accédez à l’application cliente avec une requête HTTP et regardez les résultats.

![Idée conceptuelle de l’exécution d’exemples de conteneurs](../text-analytics/media/how-tos/container-instance-sample/containers.png)

## <a name="the-sample-containers"></a>Les exemples de conteneurs

L’exemple comporte deux images de conteneur, dont une pour le site web front-end. La deuxième image est le conteneur Détection de langue, qui retourne la langue (culture) détectée pour le texte. Les deux conteneurs sont accessibles à partir d’une adresse IP externe quand vous avez terminé.

### <a name="the-language-frontend-container"></a>Le conteneur front-end de détection de langue

Ce site web est équivalent à votre propre application côté client qui effectue des demandes du point de terminaison de détection de langue. Une fois la procédure terminée, vous obtenez la langue détectée d’une chaîne de caractères en accédant au conteneur du site web dans un navigateur avec `http://<external-IP>/<text-to-analyze>`. Voici un exemple de cette URL : `http://132.12.23.255/helloworld!`. Le résultat dans le navigateur est `English`.

### <a name="the-language-container"></a>Le conteneur de langue

Dans cette procédure spécifique, le conteneur Détection de langue est accessible à toute demande externe. Le conteneur n’a été changé en aucune façon : l’API de détection de langue spécifique au conteneur Cognitive Services standard est donc disponible.

Pour ce conteneur, cette API est une demande POST pour la détection de la langue. Comme pour tous les conteneurs Cognitive Services, vous pouvez en savoir plus sur le conteneur en consultant ses informations hébergées sur Swagger : `http://<external-IP>:5000/swagger/index.html`.

Le port 5000 est le port par défaut utilisé avec les conteneurs Cognitive Services.

## <a name="create-azure-container-registry-service"></a>Créer un service Azure Container Registry

Pour déployer le conteneur sur Azure Kubernetes Service, les images de conteneur doivent être accessibles. Créez votre propre service Azure Container Registry pour héberger les images.

1. Se connecter à Azure CLI

    ```azurecli-interactive
    az login
    ```

1. Créez un groupe de ressources nommé `cogserv-container-rg` pour contenir toutes les ressources créées dans cette procédure.

    ```azurecli-interactive
    az group create --name cogserv-container-rg --location westus
    ```

1. Créez votre propre registre Azure Container Registry, en utilisant votre nom suivi de `registry`, comme `pattyregistry`. N’utilisez pas de tirets ni de caractères de trait de soulignement dans le nom.

    ```azurecli-interactive
    az acr create --resource-group cogserv-container-rg --name pattyregistry --sku Basic
    ```

    Enregistrez les résultats pour obtenir la propriété **loginServer**. Elle fera partie de l’adresse du conteneur hébergé, utilisée ultérieurement dans le fichier `language.yml`.

    ```azurecli-interactive
    az acr create --resource-group cogserv-container-rg --name pattyregistry --sku Basic
    ```

    ```output
    {
        "adminUserEnabled": false,
        "creationDate": "2019-01-02T23:49:53.783549+00:00",
        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/cogserv-container-rg/providers/Microsoft.ContainerRegistry/registries/pattyregistry",
        "location": "westus",
        "loginServer": "pattyregistry.azurecr.io",
        "name": "pattyregistry",
        "provisioningState": "Succeeded",
        "resourceGroup": "cogserv-container-rg",
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

1. Connectez-vous à votre registre de conteneurs. Vous devez vous connecter pour pouvoir envoyer (push) des images à votre registre.

    ```azurecli-interactive
    az acr login --name pattyregistry
    ```

## <a name="get-website-docker-image"></a>Obtenir l’image Docker du site web

1. L’exemple de code utilisé dans cette procédure se trouve dans le dépôt d’exemples de conteneurs Cognitive Services. Clonez le dépôt pour obtenir une copie locale de l’exemple.

    ```console
    git clone https://github.com/Azure-Samples/cognitive-services-containers-samples
    ```

    Une fois le dépôt présent sur votre ordinateur local, recherchez le site web dans le répertoire [\dotnet\Language\FrontendService](https://github.com/Azure-Samples/cognitive-services-containers-samples/tree/master/dotnet/Language/FrontendService). Ce site web agit en tant qu’application cliente appelant l’API de détection de langue hébergée dans le conteneur Détection de langue.  

1. Générez l’image Docker pour ce site web. Vérifiez que la console se trouve dans le répertoire [\FrontendService](https://github.com/Azure-Samples/cognitive-services-containers-samples/tree/master/dotnet/Language/FrontendService) où le fichier Dockerfile se trouve quand vous exécutez la commande suivante :

    ```console
    docker build -t language-frontend -t pattiyregistry.azurecr.io/language-frontend:v1 .
    ```

    Pour effectuer le suivi de la version sur votre registre de conteneurs, ajoutez l’étiquette avec un format de version, comme `v1`.

1. Envoyez l’image vers votre registre de conteneurs. Cela peut prendre quelques minutes.

    ```console
    docker push pattyregistry.azurecr.io/language-frontend:v1
    ```

    Si vous obtenez une erreur `unauthorized: authentication required`, connectez-vous avec la commande `az acr login --name <your-container-registry-name>`. 

    Quand le processus est terminé, les résultats doivent être similaires à ceci :

    ```output
    The push refers to repository [pattyregistry.azurecr.io/language-frontend]
    82ff52ee6c73: Pushed
    07599c047227: Pushed
    816caf41a9a1: Pushed
    2924be3aed17: Pushed
    45b83a23806f: Pushed
    ef68f6734aa4: Pushed
    v1: digest: sha256:31930445deee181605c0cde53dab5a104528dc1ff57e5b3b34324f0d8a0eb286 size: 1580
    ```

## <a name="get-language-detection-docker-image"></a>Obtenir l’image Docker Détection de langue

1. Tirez (pull) la dernière version de l’image Docker vers l’ordinateur local. Cela peut prendre quelques minutes. S’il existe une version plus récente de ce conteneur, remplacez la valeur `1.1.006770001-amd64-preview` par la version plus récente.

    ```console
    docker pull mcr.microsoft.com/azure-cognitive-services/language:1.1.006770001-amd64-preview
    ```

1. Étiquetez l’image avec votre registre de conteneurs. Recherchez la dernière version et de remplacer la version `1.1.006770001-amd64-preview` si vous avez une version plus récente. 

    ```console
    docker tag mcr.microsoft.com/azure-cognitive-services/language pattiyregistry.azurecr.io/language:1.1.006770001-amd64-preview
    ```

1. Envoyez l’image vers votre registre de conteneurs. Cela peut prendre quelques minutes.

    ```console
    docker push pattyregistry.azurecr.io/language:1.1.006770001-amd64-preview
    ```

## <a name="get-container-registry-credentials"></a>Obtenir les informations d’identification du registre de conteneurs

Les étapes suivantes sont nécessaires pour obtenir les informations permettant de se connecter à votre registre de conteneurs avec le service Azure Kubernetes que vous créez plus loin dans cette procédure.

1. Créez le principal du service.

    ```azurecli-interactive
    az ad sp create-for-rbac --skip-assignment
    ```

    Enregistrez la valeur `appId` des résultats pour le paramètre correspondant à la personne responsable de l’étape 3, `<appId>`. Enregistrez le `password` pour le paramètre de secret du client `<client-secret>` de la section suivante.

    ```output
    {
      "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "displayName": "azure-cli-2018-12-31-18-39-32",
      "name": "http://azure-cli-2018-12-31-18-39-32",
      "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
    ```

1. Obtenez l’ID de votre registre de conteneurs.

    ```azurecli-interactive
    az acr show --resource-group cogserv-container-rg --name pattyregistry --query "id" --o table
    ```

    Enregistrez la sortie pour la valeur du paramètre d’étendue, `<acrId>`, de l’étape suivante. Voici comment il se présente :

    ```output
    /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/cogserv-container-rg/providers/Microsoft.ContainerRegistry/registries/pattyregistry
    ```

    Enregistrez la valeur complète pour l’étape 3 de cette section.

1. Pour accorder l’accès correct permettant au cluster AKS d’utiliser des images stockées dans votre registre de conteneurs, créez une attribution de rôle. Remplacez `<appId>` et `<acrId>` par les valeurs recueillies au cours des deux étapes précédentes.

    ```azurecli-interactive
    az role assignment create --assignee <appId> --scope <acrId> --role Reader
    ```

## <a name="create-azure-kubernetes-service"></a>Créer le service Azure Kubernetes

1. Créer un cluster Kubernetes. Toutes les valeurs des paramètres proviennent des sections précédentes, à l’exception du paramètre de nom. Choisissez un nom qui indique qui l’a créé et quel est son objectif, par exemple `patty-kube`.

    ```azurecli-interactive
    az aks create --resource-group cogserv-container-rg --name patty-kube --node-count 2  --service-principal <appId>  --client-secret <client-secret>  --generate-ssh-keys
    ```

    Cette étape peut prendre quelques minutes. Le résultat est le suivant :

    ```output
    {
      "aadProfile": null,
      "addonProfiles": null,
      "agentPoolProfiles": [
        {
          "count": 2,
          "dnsPrefix": null,
          "fqdn": null,
          "maxPods": 110,
          "name": "nodepool1",
          "osDiskSizeGb": 30,
          "osType": "Linux",
          "ports": null,
          "storageProfile": "ManagedDisks",
          "vmSize": "Standard_DS1_v2",
          "vnetSubnetId": null
        }
      ],
      "dnsPrefix": "patty-kube--65a101",
      "enableRbac": true,
      "fqdn": "patty-kube--65a101-341f1f54.hcp.westus.azmk8s.io",
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/cogserv-container-rg/providers/Microsoft.ContainerService/managedClusters/patty-kube",
      "kubernetesVersion": "1.9.11",
      "linuxProfile": {
        "adminUsername": "azureuser",
        "ssh": {
          "publicKeys": [
            {
              "keyData": "ssh-rsa AAAAB3NzaC...ohR2d81mFC
            }
          ]
        }
      },
      "location": "westus",
      "name": "patty-kube",
      "networkProfile": {
        "dnsServiceIp": "10.0.0.10",
        "dockerBridgeCidr": "172.17.0.1/16",
        "networkPlugin": "kubenet",
        "networkPolicy": null,
        "podCidr": "10.244.0.0/16",
        "serviceCidr": "10.0.0.0/16"
      },
      "nodeResourceGroup": "MC_patty_westus",
      "provisioningState": "Succeeded",
      "resourceGroup": "cogserv-container-rg",
      "servicePrincipalProfile": {
        "clientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "keyVaultSecretRef": null,
        "secret": null
      },
      "tags": null,
      "type": "Microsoft.ContainerService/ManagedClusters"
    }
    ```

    Le service est créé, mais il n’a pas encore le conteneur du site web ni le conteneur Détection de langue.  

1. Obtenez les informations d’identification du cluster Kubernetes.

    ```azurecli-interactive
    az aks get-credentials --resource-group cogserv-container-rg --name patty-kube
    ```

## <a name="load-the-orchestration-definition-into-your-kubernetes-service"></a>Charger la définition de l’orchestration dans votre service Kubernetes

Cette section utilise l’interface CLI **kubectl** pour dialoguer avec Azure Kubernetes Service.

1. Avant de charger la définition de l’orchestration, vérifiez que **kubectl** a accès aux nœuds.

    ```console
    kubectl get nodes
    ```

    La réponse se présente ainsi :

    ```output
    NAME                       STATUS    ROLES     AGE       VERSION
    aks-nodepool1-13756812-0   Ready     agent     6m        v1.9.11
    aks-nodepool1-13756812-1   Ready     agent     6m        v1.9.11
    ```

1. Copiez le fichier suivant et nommez-le `language.yml`. Le fichier a une section `service` et une section `deployment` pour chacun des deux types de conteneurs : le conteneur de site web `language-frontend` et le conteneur de détection `language`.

    [!code-yml[Kubernetes orchestration file for the Cognitive Services containers sample](~/samples-cogserv-containers/Kubernetes/language/language.yml "Kubernetes orchestration file for the Cognitive Services containers sample")]

1. Modifiez les lignes qui concernent le déploiement du front-end de langue dans `language.yml`, en vous basant sur le tableau suivant pour ajouter vos propres noms d’images du registre de conteneurs, votre secret de client et vos paramètres d’analytique de texte.

    Paramètres de déploiement du front-end de langue|Objectif|
    |--|--|
    |Ligne 32<br> Propriété `image`|Emplacement de l’image pour l’image de front-end dans votre registre de conteneurs<br>`<container-registry-name>.azurecr.io/language-frontend:v1`|
    |Ligne 44<br> Propriété `name`|Secret du registre de conteneurs pour l’image, dénommé `<client-secret>` dans une section précédente.|

1. Changez les lignes qui concernent le déploiement de langue dans `language.yml`, en vous basant sur le tableau suivant pour ajouter vos propres noms d’images du registre de conteneurs, votre secret de client et vos paramètres d’analytique de texte.

    |Paramètres de déploiement de langue|Objectif|
    |--|--|
    |Ligne 78<br> Propriété `image`|Emplacement de l’image pour l’image de langue dans votre registre de conteneurs<br>`<container-registry-name>.azurecr.io/language:1.1.006770001-amd64-preview`|
    |Ligne 95<br> Propriété `name`|Secret du registre de conteneurs pour l’image, dénommé `<client-secret>` dans une section précédente.|
    |Ligne 91<br> Propriété `apiKey`|Clé de votre ressource d’analytique de texte|
    |Ligne 92<br> Propriété `billing`|Le point de terminaison de facturation pour votre ressource d’analytique de texte.<br>`https://westus.api.cognitive.microsoft.com/text/analytics/v2.1`|

    Comme l’élément **apiKey** et **point de terminaison de facturation** sont définis dans le cadre de la définition de l’orchestration de Kubernetes, le conteneur de site web n’a pas besoin de connaître ces informations ni de les passer dans la demande. Le conteneur de site web référence le conteneur Détection de langue via son nom de conteneur `language` de l’orchestrateur.

1. Chargez le fichier de définition d’orchestration pour cet exemple à partir du dossier où vous avez créé et enregistré le fichier `language.yml`.

    ```console
    kubectl apply -f language.yml
    ```

    La réponse est la suivante :

    ```output
    service "language-frontend" created
    deployment.apps "language-frontend" created
    service "language" created
    deployment.apps "language" created
    ```

## <a name="get-external-ips-of-containers"></a>Obtenir les adresses IP externes des conteneurs

Pour les deux conteneurs, vérifiez que les services `language-frontend` et `language` sont en cours d’exécution et obtenez l’adresse IP externe.

```console
kubectl get all
```

```output
NAME                                     READY     STATUS    RESTARTS   AGE
pod/language-586849d8dc-7zvz5            1/1       Running   0          13h
pod/language-frontend-68b9969969-bz9bg   1/1       Running   1          13h

NAME                        TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)          AGE
service/kubernetes          ClusterIP      10.0.0.1      <none>          443/TCP          14h
service/language            LoadBalancer   10.0.39.169   104.42.172.68   5000:30161/TCP   13h
service/language-frontend   LoadBalancer   10.0.42.136   104.42.37.219   80:30943/TCP     13h

NAME                                      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/language            1         1         1            1           13h
deployment.extensions/language-frontend   1         1         1            1           13h

NAME                                                 DESIRED   CURRENT   READY     AGE
replicaset.extensions/language-586849d8dc            1         1         1         13h
replicaset.extensions/language-frontend-68b9969969   1         1         1         13h

NAME                                DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/language            1         1         1            1           13h
deployment.apps/language-frontend   1         1         1            1           13h

NAME                                           DESIRED   CURRENT   READY     AGE
replicaset.apps/language-586849d8dc            1         1         1         13h
replicaset.apps/language-frontend-68b9969969   1         1         1         13h
```

Si l’adresse `EXTERNAL-IP` pour le service est affichée comme étant en attente, réexécutez la commande jusqu’à ce que l’adresse IP soit affichée avant de passer à l’étape suivante.

## <a name="test-the-language-detection-container"></a>Tester le conteneur Détection de langue

Ouvrez un navigateur et accédez à l’adresse IP externe du conteneur `language` de la section précédente : `http://<external-ip>:5000/swagger/index.html`. Vous pouvez utiliser la fonctionnalité `Try it` de l’API pour tester le point de terminaison de détection de langue.

![Consulter la documentation Swagger du conteneur](../text-analytics/media/how-tos/container-instance-sample/language-detection-container-swagger-documentation.png)

## <a name="test-the-client-application-container"></a>Tester le conteneur de l’application cliente

Changez l’URL dans le navigateur pour l’adresse IP externe du conteneur `language-frontend` en utilisant le format suivant : `http://<external-ip>/helloworld`. Le texte de culture English (anglais) de `helloworld` est prédit comme étant `English`.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous avez terminé avec le cluster, supprimez le groupe de ressources Azure.

```azurecli-interactive
az group delete --name cogserv-container-rg
```

## <a name="related-information"></a>Informations connexes

* [kubectl pour les utilisateurs de Docker](https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Conteneurs Cognitive Services](../cognitive-services-container-support.md)