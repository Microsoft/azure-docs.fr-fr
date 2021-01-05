---
title: Créer, tester et déployer des conteneurs sur Azure Kubernetes Service à l’aide de GitHub Actions
description: Découvrez comment utiliser GitHub Actions pour déployer votre conteneur sur Kubernetes
services: container-service
author: azooinmyluggage
ms.topic: article
ms.date: 11/06/2020
ms.author: atulmal
ms.custom: github-actions-azure
ms.openlocfilehash: d03acab340e593a925f042ca41f9e8967b468858
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97605443"
---
# <a name="github-actions-for-deploying-to-kubernetes-service"></a>GitHub Actions pour un déploiement sur le service Kubernetes

[GitHub Actions](https://docs.github.com/en/free-pro-team@latest/actions) vous donne la possibilité de créer un flux de travail de cycle de vie de développement logiciel automatisé. Vous pouvez utiliser plusieurs actions Kubernetes pour déployer dans des conteneurs à partir d’Azure Container Registry vers Azure Kubernetes Service avec GitHub Actions. 

## <a name="prerequisites"></a>Prérequis 

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un compte GitHub. Si vous n’en avez pas, inscrivez-vous [gratuitement](https://github.com/join).  
- Cluster Kubernetes opérationnel
    - [Tutoriel : Préparer une application pour Azure Kubernetes Service](tutorial-kubernetes-prepare-app.md)

## <a name="workflow-file-overview"></a>Vue d’ensemble du fichier de workflow

Un workflow est défini par un fichier YAML (.yml) situé dans le chemin `/.github/workflows/` de votre dépôt. Cette définition contient les étapes et les paramètres qui composent le workflow.

Pour un workflow ciblant AKS, le fichier comporte trois sections :

|Section  |Tâches  |
|---------|---------|
|**Authentification** | Connexion à un registre de conteneurs privé (ACR) |
|**Créer** | Création et envoi de l’image conteneur via une transmission de type push  |
|**Déployer** | 1. Définition du cluster AKS cible |
| |2. Création d’un secret générique/du registre Docker dans le cluster Kubernetes  |
||3. Déploiement sur le cluster Kubernetes|

## <a name="create-a-service-principal"></a>Créer un principal du service

Vous pouvez créer un [principal de service](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) avec la commande [az ad sp create-for-rbac](/cli/azure/ad/sp#az-ad-sp-create-for-rbac) dans [Azure CLI](/cli/azure/). Vous pouvez exécuter cette commande en utilisant [Azure Cloud Shell](https://shell.azure.com/) dans le portail Azure ou en sélectionnant le bouton **Essayer**.

```azurecli-interactive
az ad sp create-for-rbac --name "myApp" --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP> --sdk-auth
```

Dans la commande ci-dessus, remplacez les espaces réservés par votre ID d’abonnement et votre groupe de ressources. La sortie correspond aux informations d’identification de l’attribution de rôle qui permettent d’accéder à votre ressource. La commande doit générer un objet JSON similaire à celui-ci.

```json
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```
Copiez cet objet JSON, que vous pouvez utiliser pour vous authentifier à partir de GitHub.

## <a name="configure-the-github-secrets"></a>Configurer les secrets GitHub

Suivez les étapes de configuration des secrets :

1. Dans [GitHub](https://github.com/), accédez à votre référentiel, sélectionnez **Paramètres > Secrets > Ajouter un nouveau secret**.

    ![La capture d’écran montre le lien Ajouter un nouveau secret pour un référentiel.](media/kubernetes-action/secrets.png)

2. Collez le contenu de la commande `az cli` ci-dessus en tant que valeur de la variable secrète. Par exemple : `AZURE_CREDENTIALS`.

3. De même, définissez les secrets supplémentaires suivants associés aux informations d’identification du registre de conteneurs et configurez-les dans l’action de connexion Docker. 

    - REGISTRY_USERNAME
    - REGISTRY_PASSWORD

4. Vous verrez les secrets comme indiqué ci-dessous une fois qu’il seront définis.

    ![Capture d’écran montrant les secrets existants pour un référentiel.](media/kubernetes-action/kubernetes-secrets.png)

##  <a name="build-a-container-image-and-deploy-to-azure-kubernetes-service-cluster"></a>Créer une image conteneur et la déployer dans un cluster Azure Kubernetes Service

La génération et l’envoi des images conteneur via une transmission de type push s’effectuent à l’aide de l’action `Azure/docker-login@v1`. 


```yml
env:
  REGISTRY_NAME: {registry-name}
  CLUSTER_NAME: {cluster-name}
  CLUSTER_RESOURCE_GROUP: {resource-group-name}
  NAMESPACE: {namespace-name}
  APP_NAME: {app-name}
  
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@main
    
    # Connect to Azure Container registry (ACR)
    - uses: azure/docker-login@v1
      with:
        login-server: ${{ env.REGISTRY_NAME }}.azurecr.io
        username: ${{ secrets.REGISTRY_USERNAME }} 
        password: ${{ secrets.REGISTRY_PASSWORD }}
    
    # Container build and push to a Azure Container registry (ACR)
    - run: |
        docker build . -t ${{ env.REGISTRY_NAME }}.azurecr.io/${{ env.APP_NAME }}:${{ github.sha }}
        docker push ${{ env.REGISTRY_NAME }}.azurecr.io/${{ env.APP_NAME }}:${{ github.sha }}

```

### <a name="deploy-to-azure-kubernetes-service-cluster"></a>Déployer des images dans un cluster Azure Kubernetes Service

Pour déployer une image conteneur sur AKS, vous devez utiliser l’action `Azure/k8s-deploy@v1`. Cette action a cinq paramètres :

| **Paramètre**  | **Explication**  |
|---------|---------|
| **namespace** | (Facultatif) Choisissez l’espace de noms Kubernetes cible. Si l’espace de noms n’est pas fourni, les commandes s’exécutent dans l’espace de noms par défaut. | 
| **manifests** |  (Obligatoire) Chemin d’accès aux fichiers manifestes qui seront utilisés pour le déploiement. |
| **images** | (Facultatif) URL de ressource complète de la ou des images à utiliser pour les substitutions dans les fichiers manifestes. |
| **imagepullsecrets** | (Facultatif) Nom d’un secret du registre Docker déjà configuré dans le cluster. Chaque nom de secret est ajouté sous le champ imagePullSecrets pour les charges de travail trouvées dans les fichiers manifestes d’entrée. |
| **kubectl-version** | (Facultatif) Installe une version spécifique du fichier binaire kubectl. |


Avant de pouvoir déployer sur AKS, vous devez définir l’espace de noms Kubernetes cible et créer un secret d’extraction (pull) d’image. Pour en savoir plus sur le fonctionnement de l’extraction (pull) d’images, consultez [Extraire des images depuis un registre de conteneurs Azure vers un cluster Kubernetes](../container-registry/container-registry-auth-kubernetes.md). 

```yaml
  # Create namespace if doesn't exist
  - run: |
      kubectl create namespace ${{ env.NAMESPACE }} --dry-run -o json | kubectl apply -f -
  
  # Create image pull secret for ACR
  - uses: azure/k8s-create-secret@v1
    with:
      container-registry-url: ${{ env.REGISTRY_NAME }}.azurecr.io
      container-registry-username: ${{ secrets.REGISTRY_USERNAME }}
      container-registry-password: ${{ secrets.REGISTRY_PASSWORD }}
      secret-name: ${{ env.SECRET }}
      namespace: ${{ env.NAMESPACE }}
      force: true
```


Terminez votre déploiement avec l’action `k8s-deploy`. Remplacez les variables d’environnement par les valeurs adaptées à votre application. 

```yaml

on: [push]

# Environment variables available to all jobs and steps in this workflow
env:
  REGISTRY_NAME: {registry-name}
  CLUSTER_NAME: {cluster-name}
  CLUSTER_RESOURCE_GROUP: {resource-group-name}
  NAMESPACE: {namespace-name}
  SECRET: {secret-name}
  APP_NAME: {app-name}
  
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@main
    
    # Connect to Azure Container registry (ACR)
    - uses: azure/docker-login@v1
      with:
        login-server: ${{ env.REGISTRY_NAME }}.azurecr.io
        username: ${{ secrets.REGISTRY_USERNAME }} 
        password: ${{ secrets.REGISTRY_PASSWORD }}
    
    # Container build and push to a Azure Container registry (ACR)
    - run: |
        docker build . -t ${{ env.REGISTRY_NAME }}.azurecr.io/${{ env.APP_NAME }}:${{ github.sha }}
        docker push ${{ env.REGISTRY_NAME }}.azurecr.io/${{ env.APP_NAME }}:${{ github.sha }}
    
    # Set the target Azure Kubernetes Service (AKS) cluster. 
    - uses: azure/aks-set-context@v1
      with:
        creds: '${{ secrets.AZURE_CREDENTIALS }}'
        cluster-name: ${{ env.CLUSTER_NAME }}
        resource-group: ${{ env.CLUSTER_RESOURCE_GROUP }}
    
    # Create namespace if doesn't exist
    - run: |
        kubectl create namespace ${{ env.NAMESPACE }} --dry-run -o json | kubectl apply -f -
    
    # Create image pull secret for ACR
    - uses: azure/k8s-create-secret@v1
      with:
        container-registry-url: ${{ env.REGISTRY_NAME }}.azurecr.io
        container-registry-username: ${{ secrets.REGISTRY_USERNAME }}
        container-registry-password: ${{ secrets.REGISTRY_PASSWORD }}
        secret-name: ${{ env.SECRET }}
        namespace: ${{ env.NAMESPACE }}
        force: true
    
    # Deploy app to AKS
    - uses: azure/k8s-deploy@v1
      with:
        manifests: |
          manifests/deployment.yml
          manifests/service.yml
        images: |
          ${{ env.REGISTRY_NAME }}.azurecr.io/${{ env.APP_NAME }}:${{ github.sha }}
        imagepullsecrets: |
          ${{ env.SECRET }}
        namespace: ${{ env.NAMESPACE }}
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand votre registre de conteneurs, référentiel et cluster Kubernetes ne sont plus nécessaires, nettoyez les ressources que vous avez déployées en supprimant le groupe de ressources et votre référentiel GitHub. 

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur Azure Kubernetes Service](/azure/architecture/reference-architectures/containers/aks-start-here)

### <a name="more-kubernetes-github-actions"></a>Autres Kubernetes GitHub Actions

* [Programme d’installation de l’outil](https://github.com/Azure/setup-kubectl) (`azure/setup-kubectl`) : Installe une version spécifique de kubectl sur l’exécuteur.
* [Kubernetes - Définir le contexte](https://github.com/Azure/k8s-set-context) (`azure/k8s-set-context`) : Définissez le contexte de cluster Kubernetes cible qui sera utilisé par d’autres actions ou pour exécuter des commandes kubectl.
* [AKS - Définir le contexte](https://github.com/Azure/aks-set-context) (`azure/aks-set-context`) : Définissez le contexte de cluster Azure Kubernetes Service cible.
* [Kubernetes - Créer un secret](https://github.com/Azure/k8s-create-secret) (`azure/k8s-create-secret`) : Créez un secret générique ou un secret docker-registre dans le cluster Kubernetes.
* [Kubernetes - Déployer](https://github.com/Azure/k8s-deploy) (`azure/k8s-deploy`) : Effectuez un bake et déployez des manifestes sur des clusters Kubernetes.
* [Configurer Helm](https://github.com/Azure/setup-helm) (`azure/setup-helm`) : Installez une version spécifique du fichier binaire Helm sur l’exécuteur.
* [Kubernetes - Effectuer un bake](https://github.com/Azure/k8s-bake) (`azure/k8s-bake`) : Effectuez un bake du fichier manifeste à utiliser pour les déploiements à l’aide de helm2, kustomize ou kompose.
* [Kubernetes - Lint](https://github.com/azure/k8s-lint) (`azure/k8s-lint`) : Validez/lint vos fichiers manifestes.
