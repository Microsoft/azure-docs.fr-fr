---
title: Créer un contrôleur d’entrée avec une nouvelle instance Application Gateway
description: Cet article fournit des informations sur le déploiement d’un contrôleur d’entrée Application Gateway avec une nouvelle instance Application Gateway.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: 9f69f89f565b2d98e408b06e300ff781c13680ef
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97693664"
---
# <a name="how-to-install-an-application-gateway-ingress-controller-agic-using-a-new-application-gateway"></a>Comment installer un contrôleur d’entrée Application Gateway (AGIC) à l’aide d’une nouvelle instance Application Gateway

Les instructions ci-dessous supposent que le contrôleur d’entrée Application Gateway (AGIC) sera installé dans un environnement sans composants préexistants.

## <a name="required-command-line-tools"></a>Outils en ligne de commande requis

Nous vous recommandons d’utiliser [Azure Cloud Shell](https://shell.azure.com/) pour toutes les opérations de ligne de commande ci-dessous. Lancez votre interpréteur de commandes à partir de shell.azure.com ou en cliquant sur le lien suivant :

[![Lancement de l’incorporation](https://shell.azure.com/images/launchcloudshell.png "Lancement d’Azure Cloud Shell")](https://shell.azure.com)

Vous pouvez également lancer Cloud Shell à partir du Portail Azure à l’aide de l’icône suivante :

![Lancement du portail](./media/application-gateway-ingress-controller-install-new/portal-launch-icon.png)

Votre [Azure Cloud Shell](https://shell.azure.com/) contient déjà tous les outils nécessaires. Si vous choisissez d’utiliser un autre environnement, vérifiez que les outils en ligne de commande suivants sont installés :

* `az` - Azure CLI : [instructions d’installation](/cli/azure/install-azure-cli?view=azure-cli-latest)
* `kubectl` -Outil en ligne de commande Kubernetes : [instructions d’installation](https://kubernetes.io/docs/tasks/tools/install-kubectl)
* `helm` - Gestionnaire de package Kubernetes : [instructions d’installation](https://github.com/helm/helm/releases/latest)
* `jq` -Processeur JSON en ligne de commande : [instructions d’installation](https://stedolan.github.io/jq/download/)


## <a name="create-an-identity"></a>Créer une identité

Suivez les étapes ci-dessous pour créer un [objet principal du service](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) Azure Active Directory (AAD). Enregistrez les valeurs `appId`, `password` et `objectId`,-celles-ci seront utilisées dans les étapes suivantes.

1. Créer un principal de service AD ([En savoir plus sur Azure RBAC](../role-based-access-control/overview.md)) :
    ```azurecli
    az ad sp create-for-rbac --skip-assignment -o json > auth.json
    appId=$(jq -r ".appId" auth.json)
    password=$(jq -r ".password" auth.json)
    ```
    Les valeurs `appId` et `password` de la sortie JSON seront utilisées dans les étapes suivantes


1. Utilisez la `appId` de la sortie de la commande précédente pour récupérer le `objectId` du nouveau principal du service :
    ```azurecli
    objectId=$(az ad sp show --id $appId --query "objectId" -o tsv)
    ```
    La sortie de cette commande est `objectId`, qui sera utilisée dans le modèle Azure Resource Manager ci-dessous

1. Créez le fichier de paramètres qui sera utilisé ultérieurement dans le déploiement du modèle Azure Resource Manager.
    ```bash
    cat <<EOF > parameters.json
    {
      "aksServicePrincipalAppId": { "value": "$appId" },
      "aksServicePrincipalClientSecret": { "value": "$password" },
      "aksServicePrincipalObjectId": { "value": "$objectId" },
      "aksEnableRBAC": { "value": false }
    }
    EOF
    ```
    Pour déployer un cluster sur lequel **Kubernetes RBAC** est activé, définissez le champ `aksEnableRBAC` sur `true`.

## <a name="deploy-components"></a>Déployer des composants
Cette étape permet d’ajouter les composants suivants à votre abonnement :

- [Azure Kubernetes Service](../aks/intro-kubernetes.md)
- [Application Gateway](./overview.md) v2
- [Réseau virtuel](../virtual-network/virtual-networks-overview.md) avec 2 [sous-réseaux](../virtual-network/virtual-networks-overview.md)
- [Adresse IP publique](../virtual-network/virtual-network-public-ip-address.md)
- [Identité managée](../active-directory/managed-identities-azure-resources/overview.md) qui sera utilisée par [AAD Pod Identity](https://github.com/Azure/aad-pod-identity/blob/master/README.md)

1. Téléchargez le modèle Azure Resource Manager et modifiez le modèle en fonction des besoins.
    ```bash
    wget https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/deploy/azuredeploy.json -O template.json
    ```

1. Déployez le modèle Azure Resource Manager à l’aide de `az cli`. Cette opération peut durer jusqu’à 5 minutes.
    ```azurecli
    resourceGroupName="MyResourceGroup"
    location="westus2"
    deploymentName="ingress-appgw"

    # create a resource group
    az group create -n $resourceGroupName -l $location

    # modify the template as needed
    az deployment group create \
            -g $resourceGroupName \
            -n $deploymentName \
            --template-file template.json \
            --parameters parameters.json
    ```

1. Une fois le déploiement terminé, téléchargez la sortie du déploiement dans un fichier nommé `deployment-outputs.json`.
    ```azurecli
    az deployment group show -g $resourceGroupName -n $deploymentName --query "properties.outputs" -o json > deployment-outputs.json
    ```

## <a name="set-up-application-gateway-ingress-controller"></a>Configurer le contrôleur d’entrée Application Gateway

Avec les instructions de la section précédente, nous avons créé et configuré un nouveau cluster AKS ainsi qu’une passerelle d’application. Nous sommes maintenant prêts à déployer un exemple d’application et un contrôleur d’entrée dans notre nouvelle infrastructure Kubernetes.

### <a name="setup-kubernetes-credentials"></a>Configurer les informations d’identification Kubernetes
Pour les étapes suivantes, nous avons besoin de configurer la commande [kubectl](https://kubectl.docs.kubernetes.io/), que nous utiliserons pour nous connecter à notre nouveau cluster Kubernetes. [Cloud Shell](https://shell.azure.com/) a `kubectl` déjà installé. Nous utiliserons `az` CLI pour obtenir des informations d’identification pour Kubernetes.

Obtenir des informations d’identification pour votre AKS récemment déployé ([en savoir plus](../aks/kubernetes-walkthrough.md#connect-to-the-cluster)) :
```azurecli
# use the deployment-outputs.json created after deployment to get the cluster name and resource group name
aksClusterName=$(jq -r ".aksClusterName.value" deployment-outputs.json)
resourceGroupName=$(jq -r ".resourceGroupName.value" deployment-outputs.json)

az aks get-credentials --resource-group $resourceGroupName --name $aksClusterName
```

### <a name="install-aad-pod-identity"></a>Installer AAD Pod Identity
  Azure Active Directory Pod Identity offre un accès à base de jeton à [Azure Resource Manager (ARM)](../azure-resource-manager/management/overview.md).

  [AAD Pod Identity](https://github.com/Azure/aad-pod-identity) ajoute les composants suivants à votre cluster Kubernetes :
   * [Définitions CRD](https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/) Kubernetes : `AzureIdentity`, `AzureAssignedIdentity`, `AzureIdentityBinding`
   * Composant [MIC (Managed Identity Controller)](https://github.com/Azure/aad-pod-identity#managed-identity-controllermic)
   * Composant [NMI (Node Managed Identity)](https://github.com/Azure/aad-pod-identity#node-managed-identitynmi)


Pour installer AAD Pod Identity sur votre cluster :

   - Cluster AKS sur lequel *Kubernetes RBAC* est activé

     ```bash
     kubectl create -f https://raw.githubusercontent.com/Azure/aad-pod-identity/master/deploy/infra/deployment-rbac.yaml
     ```

   - Cluster AKS sur lequel *Kubernetes RBAC* est désactivé

     ```bash
     kubectl create -f https://raw.githubusercontent.com/Azure/aad-pod-identity/master/deploy/infra/deployment.yaml
     ```

### <a name="install-helm"></a>Installer Helm
[Helm](../aks/kubernetes-helm.md) est un gestionnaire de package pour Kubernetes. Nous allons l’utiliser pour installer le package `application-gateway-kubernetes-ingress` :

1. Installez [Helm](../aks/kubernetes-helm.md) et exécutez la commande suivante pour ajouter le package Helm `application-gateway-kubernetes-ingress` :

    - Cluster AKS sur lequel *Kubernetes RBAC* est activé

        ```bash
        kubectl create serviceaccount --namespace kube-system tiller-sa
        kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller-sa
        helm init --tiller-namespace kube-system --service-account tiller-sa
        ```

    - Cluster AKS sur lequel *Kubernetes RBAC* est désactivé

        ```bash
        helm init
        ```

1. Ajoutez le dépôt Helm AGIC :
    ```bash
    helm repo add application-gateway-kubernetes-ingress https://appgwingress.blob.core.windows.net/ingress-azure-helm-package/
    helm repo update
    ```

### <a name="install-ingress-controller-helm-chart"></a>Installer le chart Helm du contrôleur d’entrée

1. Utilisez le fichier `deployment-outputs.json` créé ci-dessus et créez les variables suivantes.
    ```bash
    applicationGatewayName=$(jq -r ".applicationGatewayName.value" deployment-outputs.json)
    resourceGroupName=$(jq -r ".resourceGroupName.value" deployment-outputs.json)
    subscriptionId=$(jq -r ".subscriptionId.value" deployment-outputs.json)
    identityClientId=$(jq -r ".identityClientId.value" deployment-outputs.json)
    identityResourceId=$(jq -r ".identityResourceId.value" deployment-outputs.json)
    ```
1. Téléchargez helm-config.yaml pour configurer AGIC :
    ```bash
    wget https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/sample-helm-config.yaml -O helm-config.yaml
    ```
    Ou copiez le fichier YAML ci-dessous : 
    
    ```yaml
    # This file contains the essential configs for the ingress controller helm chart

    # Verbosity level of the App Gateway Ingress Controller
    verbosityLevel: 3
    
    ################################################################################
    # Specify which application gateway the ingress controller will manage
    #
    appgw:
        subscriptionId: <subscriptionId>
        resourceGroup: <resourceGroupName>
        name: <applicationGatewayName>
    
        # Setting appgw.shared to "true" will create an AzureIngressProhibitedTarget CRD.
        # This prohibits AGIC from applying config for any host/path.
        # Use "kubectl get AzureIngressProhibitedTargets" to view and change this.
        shared: false
    
    ################################################################################
    # Specify which kubernetes namespace the ingress controller will watch
    # Default value is "default"
    # Leaving this variable out or setting it to blank or empty string would
    # result in Ingress Controller observing all acessible namespaces.
    #
    # kubernetes:
    #   watchNamespace: <namespace>
    
    ################################################################################
    # Specify the authentication with Azure Resource Manager
    #
    # Two authentication methods are available:
    # - Option 1: AAD-Pod-Identity (https://github.com/Azure/aad-pod-identity)
    armAuth:
        type: aadPodIdentity
        identityResourceID: <identityResourceId>
        identityClientID:  <identityClientId>
    
    ## Alternatively you can use Service Principal credentials
    # armAuth:
    #    type: servicePrincipal
    #    secretJSON: <<Generate this value with: "az ad sp create-for-rbac --subscription <subscription-uuid> --sdk-auth | base64 -w0" >>
    
    ################################################################################
    # Specify if the cluster is Kubernetes RBAC enabled or not
    rbac:
        enabled: false # true/false
    
    # Specify aks cluster related information. THIS IS BEING DEPRECATED.
    aksClusterConfiguration:
        apiServerAddress: <aks-api-server-address>
    ```

1. Modifiez le fichier helm-config.yaml qui vient d’être téléchargé et renseignez les sections `appgw` et `armAuth`.
    ```bash
    sed -i "s|<subscriptionId>|${subscriptionId}|g" helm-config.yaml
    sed -i "s|<resourceGroupName>|${resourceGroupName}|g" helm-config.yaml
    sed -i "s|<applicationGatewayName>|${applicationGatewayName}|g" helm-config.yaml
    sed -i "s|<identityResourceId>|${identityResourceId}|g" helm-config.yaml
    sed -i "s|<identityClientId>|${identityClientId}|g" helm-config.yaml

    # You can further modify the helm config to enable/disable features
    nano helm-config.yaml
    ```

   Valeurs :
     - `verbosityLevel`: définit le niveau de détail de l’infrastructure de journalisation AGIC. Pour connaître les valeurs possibles, consultez [Niveaux de journalisation](https://github.com/Azure/application-gateway-kubernetes-ingress/blob/463a87213bbc3106af6fce0f4023477216d2ad78/docs/troubleshooting.md#logging-levels).
     - `appgw.subscriptionId`: l’ID d’abonnement Azure dans lequel Application Gateway réside. Exemple : `a123b234-a3b4-557d-b2df-a0bc12de1234`
     - `appgw.resourceGroup`: nom du groupe de ressources Azure dans lequel la passerelle Application Gateway a été créée. Exemple : `app-gw-resource-group`
     - `appgw.name`: nom de la passerelle Application Gateway. Exemple : `applicationgatewayd0f0`
     - `appgw.shared`: cet indicateur booléen doit être défini par défaut sur `false`. Affectez-lui la valeur `true` si vous avez besoin d’une [passerelle Application Gateway partagée](https://github.com/Azure/application-gateway-kubernetes-ingress/blob/072626cb4e37f7b7a1b0c4578c38d1eadc3e8701/docs/setup/install-existing.md#multi-cluster--shared-app-gateway).
     - `kubernetes.watchNamespace`: spécifiez l’espace de noms que doit observer AGIC. Il peut être une valeur de chaîne unique ou une liste d’espaces de noms séparés par des virgules.
    - `armAuth.type`: peut être `aadPodIdentity` ou `servicePrincipal`
    - `armAuth.identityResourceID`: ID de ressource de l’identité managée Azure
    - `armAuth.identityClientId`: ID client de l’identité. Pour plus d’informations sur l’identité, voir plus bas
    - `armAuth.secretJSON`: nécessaire uniquement quand le type de secret du principal du service est choisi (quand `armAuth.type` a été défini sur `servicePrincipal`) 


   > [!NOTE]
   > `identityResourceID` et `identityClientID` sont les valeurs qui ont été créées aux étapes [Déployer des composants](ingress-controller-install-new.md#deploy-components) et peuvent être obtenues à nouveau à l’aide de la commande suivante :
   > ```azurecli
   > az identity show -g <resource-group> -n <identity-name>
   > ```
   > `<resource-group>` dans la commande ci-dessus est le groupe de ressources de votre passerelle Application Gateway. `<identity-name>` est le nom de l’identité créée. Toutes les identités d’un abonnement donné peuvent être listées à l’aide de : `az identity list`


1. Installez le package de contrôleur d’entrée Application Gateway :

    ```bash
    helm install -f helm-config.yaml application-gateway-kubernetes-ingress/ingress-azure
    ```

## <a name="install-a-sample-app"></a>Installer un exemple d’application
Maintenant que la passerelle App Gateway, AKS et AGIC sont installés, vous pouvez installer un exemple d’application via [Azure Cloud Shell](https://shell.azure.com/) :

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: aspnetapp
  labels:
    app: aspnetapp
spec:
  containers:
  - image: "mcr.microsoft.com/dotnet/core/samples:aspnetapp"
    name: aspnetapp-image
    ports:
    - containerPort: 80
      protocol: TCP

---

apiVersion: v1
kind: Service
metadata:
  name: aspnetapp
spec:
  selector:
    app: aspnetapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80

---

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: aspnetapp
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: aspnetapp
          servicePort: 80
EOF
```

Sinon, vous pouvez également :

* Téléchargez le fichier YAML ci-dessus :

```bash
curl https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/aspnetapp.yaml -o aspnetapp.yaml
```

* Appliquez le fichier YAML :

```bash
kubectl apply -f aspnetapp.yaml
```


## <a name="other-examples"></a>Autres exemples
Ce [guide pratique](ingress-controller-expose-service-over-http-https.md) contient des exemples supplémentaires sur la façon d’exposer un service AKS via HTTP ou HTTPS à Internet avec Application Gateway.