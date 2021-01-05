---
title: Créer un contrôleur d’entrée avec une instance Application Gateway existante
description: Cet article fournit des informations sur le déploiement d’un contrôleur d’entrée Application Gateway avec une instance Application Gateway existante.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: 2d64766c754c0ea104ae83fde799a514e9da6d68
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97693726"
---
# <a name="install-an-application-gateway-ingress-controller-agic-using-an-existing-application-gateway"></a>Installer un contrôleur d’entrée Application Gateway (AGIC) à l’aide d’une instance Application Gateway existante

Le contrôleur d’entrée Application Gateway (AGIC) est un pod au sein de votre cluster Kubernetes.
AGIC surveille les ressources [d’entrée](https://kubernetes.io/docs/concepts/services-networking/ingress/) Kubernetes, et crée et applique la configuration Application Gateway en fonction de l’état du cluster Kubernetes.

## <a name="outline"></a>Plan :
- [Composants requis](#prerequisites)
- [Authentification Azure Resource Manager (ARM)](#azure-resource-manager-authentication)
    - Option 1 : [Configurer aad-pod-identity](#set-up-aad-pod-identity) et créer une identité Azure sur ARM
    - Option n°2 : [Utiliser un principal du service](#using-a-service-principal)
- [Installer le contrôleur d’entrée avec Helm](#install-ingress-controller-as-a-helm-chart)
- [Instance Application Gateway partagée/sur plusieurs clusters](#multi-cluster--shared-application-gateway) : Installez AGIC dans un environnement où l’instance Application Gateway est partagée entre un ou plusieurs clusters AKS et/ou d’autres composants Azure.

## <a name="prerequisites"></a>Prérequis
Ce document suppose que vous avez déjà installé l’infrastructure et les outils suivants :
- [AKS](https://azure.microsoft.com/services/kubernetes-service/) avec [Mise en réseau avancée](../aks/configure-azure-cni.md) activée
- [Application Gateway v2](./tutorial-autoscale-ps.md) dans le même réseau virtuel qu’AKS
- [AAD Pod Identity](https://github.com/Azure/aad-pod-identity) installé sur votre cluster AKS
- [Cloud Shell](https://shell.azure.com/) est l’environnement d’interpréteur de commandes Azure, qui a `az` CLI, `kubectl` et `helm` installés. Ces outils sont requis pour les commandes ci-dessous.

Veuillez __sauvegarder la configuration de votre instance Application Gateway__ avant d’installer AGIC :
  1. à l’aide du [Portail Azure](https://portal.azure.com/), accédez à votre instance `Application Gateway`
  2. dans `Export template`, cliquez sur `Download`

Le fichier zip que vous avez téléchargé aura des modèles JSON, des scripts bash et PowerShell que vous pouvez utiliser pour restaurer l’instance Application Gateway si nécessaire

## <a name="install-helm"></a>Installer Helm
[Helm](../aks/kubernetes-helm.md) est un gestionnaire de package pour Kubernetes. Nous allons l’utiliser pour installer le package `application-gateway-kubernetes-ingress`.
Utilisez [Cloud Shell](https://shell.azure.com/) pour installer Helm :

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

## <a name="azure-resource-manager-authentication"></a>Authentification Azure Resource Manager

AGIC communique avec le serveur d’API Kubernetes et Azure Resource Manager. Une identité est nécessaire pour accéder à ces API.

## <a name="set-up-aad-pod-identity"></a>Configurer AAD Pod Identity

[AAD Pod Identity](https://github.com/Azure/aad-pod-identity) est un contrôleur, similaire à AGIC, qui s’exécute également sur votre instance AKS. Il lie des identités Azure Active Directory à vos pods Kubernetes. Une identité est nécessaire pour qu’une application dans un pod Kubernetes puisse communiquer avec d’autres composants Azure. Dans le cas présent, nous avons besoin d’une autorisation pour que le pod AGIC envoie des requêtes HTTP à [ARM](../azure-resource-manager/management/overview.md).

Suivez les [instructions d’installation d’AAD Pod Identity](https://github.com/Azure/aad-pod-identity#deploy-the-azure-aad-identity-infra) pour ajouter ce composant à votre instance AKS.

Ensuite, nous devons créer une identité Azure et lui accorder des autorisations ARM.
Utilisez [Cloud Shell](https://shell.azure.com/) pour exécuter toutes les commandes suivantes et créer une identité :

1. Créez une identité Azure **dans le même groupe de ressources que les nœuds AKS**. Il est important de choisir le groupe de ressources approprié. Le groupe de ressources requis dans la commande ci-dessous n’est *pas* celui référencé dans le volet du portail AKS. Il s’agit du groupe de ressources des machines virtuelles `aks-agentpool`. En général, le groupe de ressources commence par `MC_` et contient le nom de votre instance AKS. Par exemple : `MC_resourceGroup_aksABCD_westus`

    ```azurecli
    az identity create -g <agent-pool-resource-group> -n <identity-name>
    ```

1. Pour les commandes d’attribution de rôle ci-dessous, nous devons obtenir `principalId` pour l’identité nouvellement créée :

    ```azurecli
    az identity show -g <resourcegroup> -n <identity-name>
    ```

1. Donnez à l’identité un accès `Contributor` à votre instance Application Gateway. Pour cela, vous avez besoin de l’ID de l’instance Application Gateway, qui ressemble à ce qui suit : `/subscriptions/A/resourceGroups/B/providers/Microsoft.Network/applicationGateways/C`

    Récupérez la liste des ID de l’instance Application Gateway dans votre abonnement avec : `az network application-gateway list --query '[].id'`

    ```azurecli
    az role assignment create \
        --role Contributor \
        --assignee <principalId> \
        --scope <App-Gateway-ID>
    ```

1. Donnez à l’identité un accès `Reader` au groupe de ressources Application Gateway. L’ID du groupe de ressources ressemble à ce qui suit : `/subscriptions/A/resourceGroups/B`. Vous pouvez récupérer tous les groupes de ressources avec : `az group list --query '[].id'`

    ```azurecli
    az role assignment create \
        --role Reader \
        --assignee <principalId> \
        --scope <App-Gateway-Resource-Group-ID>
    ```

## <a name="using-a-service-principal"></a>Utiliser un principal du service
Il est également possible de fournir un accès AGIC à ARM via un secret Kubernetes.

1. Créez un principal du service Azure Active Directory et encodez-le en base64. L’encodage base64 est requis pour enregistrer le blob JSON dans Kubernetes.

```azurecli
az ad sp create-for-rbac --sdk-auth | base64 -w0
```

2. Ajoutez le blob JSON encodé en base64 au fichier `helm-config.yaml`. Pour en savoir plus sur `helm-config.yaml`, consultez la section suivante.
```yaml
armAuth:
    type: servicePrincipal
    secretJSON: <Base64-Encoded-Credentials>
```

## <a name="install-ingress-controller-as-a-helm-chart"></a>Installer le contrôleur d’entrée en tant que chart Helm
Les premières étapes consistent à installer Tiller de Helm sur votre cluster Kubernetes. Utilisez [Cloud Shell](https://shell.azure.com/) pour installer le package AGIC Helm :

1. Ajoutez le référentiel Helm `application-gateway-kubernetes-ingress` et mettez à jour à Helm

    ```bash
    helm repo add application-gateway-kubernetes-ingress https://appgwingress.blob.core.windows.net/ingress-azure-helm-package/
    helm repo update
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
    #    secretJSON: <<Generate this value with: "az ad sp create-for-rbac --sdk-auth | base64 -w0" >>
    
    ################################################################################
    # Specify if the cluster is Kubernetes RBAC enabled or not
    rbac:
        enabled: false # true/false
    
    # Specify aks cluster related information. THIS IS BEING DEPRECATED.
    aksClusterConfiguration:
        apiServerAddress: <aks-api-server-address>
    ```

1. Modifiez helm-config.yam et renseignez les valeurs pour `appgw` et `armAuth`.
    ```bash
    nano helm-config.yaml
    ```

    > [!NOTE] 
    > `<identity-resource-id>` et `<identity-client-id>` sont les propriétés de l’identité Azure AD que vous avez configurée dans la section précédente. Vous pouvez récupérer ces informations en exécutant la commande suivante : `az identity show -g <resourcegroup> -n <identity-name>`, où `<resourcegroup>` est le groupe de ressources dans lequel l’objet cluster AKS de niveau supérieur, l’instance Application Gateway et l’identité managée sont déployés.

1. Installez le chart Helm `application-gateway-kubernetes-ingress` avec la configuration `helm-config.yaml` de l’étape précédente

    ```bash
    helm install -f <helm-config.yaml> application-gateway-kubernetes-ingress/ingress-azure
    ```

    Vous pouvez également combiner `helm-config.yaml` et la commande Helm en une seule étape :
    ```bash
    helm install ./helm/ingress-azure \
         --name ingress-azure \
         --namespace default \
         --debug \
         --set appgw.name=applicationgatewayABCD \
         --set appgw.resourceGroup=your-resource-group \
         --set appgw.subscriptionId=subscription-uuid \
         --set appgw.shared=false \
         --set armAuth.type=servicePrincipal \
         --set armAuth.secretJSON=$(az ad sp create-for-rbac --sdk-auth | base64 -w0) \
         --set rbac.enabled=true \
         --set verbosityLevel=3 \
         --set kubernetes.watchNamespace=default \
         --set aksClusterConfiguration.apiServerAddress=aks-abcdefg.hcp.westus2.azmk8s.io
    ```

1. Consultez le journal du pod nouvellement créé pour vérifier s’il a démarré correctement

Reportez-vous à [ce guide pratique](ingress-controller-expose-service-over-http-https.md) pour comprendre comment vous pouvez exposer un service AKS via HTTP ou HTTPS sur Internet, à l’aide d’une instance Azure Application Gateway.



## <a name="multi-cluster--shared-application-gateway"></a>Instance Application Gateway partagée/sur plusieurs clusters
Par défaut, AGIC est totalement propriétaire de l’instance Application Gateway à laquelle il est lié. AGIC version 0.8.0 et ultérieure peut partager une seule instance Application Gateway unique avec d’autres composants Azure. Par exemple, nous pouvons utiliser la même instance Application Gateway pour une application hébergée sur un groupe de machines virtuelles identiques et un cluster AKS.

Veuillez __sauvegarder la configuration de votre instance Application Gateway__ avant d’activer ce paramètre :
  1. à l’aide du [Portail Azure](https://portal.azure.com/), accédez à votre instance `Application Gateway`
  2. dans `Export template`, cliquez sur `Download`

Le fichier zip que vous avez téléchargé aura des modèles JSON, des scripts bash et PowerShell que vous pouvez utiliser pour restaurer l’instance Application Gateway

### <a name="example-scenario"></a>Exemple de scénario
Examinons une instance Application Gateway imaginaire, qui gère le trafic de deux sites web :
  - `dev.contoso.com` - hébergé sur une nouvelle instance AKS à l’aide d’une instance Application Gateway et d’AGIC
  - `prod.contoso.com` - hébergé sur un [groupe de machines virtuelles identiques Azure](https://azure.microsoft.com/services/virtual-machine-scale-sets/)

Avec les paramètres par défaut, AGIC est à 100 % propriétaire de l’instance Application Gateway à laquelle il est lié. AGIC remplace toute la configuration de l’instance Application Gateway. Si nous devions créer manuellement un écouteur pour `prod.contoso.com` (sur l’instance Application Gateway), sans le définir dans l’entrée Kubernetes, AGIC supprimerait la configuration `prod.contoso.com` en quelques secondes.

Pour installer AGIC et traiter `prod.contoso.com` à partir de notre groupe de machines virtuelles identiques, nous devons contraindre AGIC à configurer `dev.contoso.com` uniquement. Cela est facilité par l’instanciation du [CRD](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) suivant :

```bash
cat <<EOF | kubectl apply -f -
apiVersion: "appgw.ingress.k8s.io/v1"
kind: AzureIngressProhibitedTarget
metadata:
  name: prod-contoso-com
spec:
  hostname: prod.contoso.com
EOF
```

La commande ci-dessus crée un objet `AzureIngressProhibitedTarget`. Cela permet à AGIC (version 0.8.0 et ultérieure) de prendre conscience de l’existence de la configuration de l’instance Application Gateway pour `prod.contoso.com` et de lui demander explicitement d’éviter de modifier la configuration liée à ce nom d’hôte.


### <a name="enable-with-new-agic-installation"></a>Activer avec une nouvelle installation AGIC
Pour limiter AGIC (version 0.8.0 et ultérieure) à un sous-ensemble de la configuration de l’instance Application Gateway, modifiez le modèle `helm-config.yaml`.
Dans la section `appgw:`, ajoutez la clé `shared` et affectez-lui la valeur `true`.

```yaml
appgw:
    subscriptionId: <subscriptionId>    # existing field
    resourceGroup: <resourceGroupName>  # existing field
    name: <applicationGatewayName>      # existing field
    shared: true                        # <<<<< Add this field to enable shared Application Gateway >>>>>
```

Appliquez les modifications Helm :
  1. Assurez-vous que le CRD `AzureIngressProhibitedTarget` est installé avec :
      ```bash
      kubectl apply -f https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/ae695ef9bd05c8b708cedf6ff545595d0b7022dc/crds/AzureIngressProhibitedTarget.yaml
      ```
  2. Mettez à jour Helm :
      ```bash
      helm upgrade \
          --recreate-pods \
          -f helm-config.yaml \
          ingress-azure application-gateway-kubernetes-ingress/ingress-azure
      ```

Par conséquent, votre instance AKS aura une nouvelle instance de `AzureIngressProhibitedTarget` appelée `prohibit-all-targets` :
```bash
kubectl get AzureIngressProhibitedTargets prohibit-all-targets -o yaml
```

L’objet `prohibit-all-targets`, comme son nom l’indique, interdit à AGIC de modifier la configuration de *tout* hôte et chemin d’accès.
L’installation de Helm avec `appgw.shared=true` déploie AGIC, mais n’apporte aucune modification à l’instance Application Gateway.


### <a name="broaden-permissions"></a>Élargir les autorisations
Étant donné que Helm avec `appgw.shared=true` et l’élément `prohibit-all-targets` par défaut empêchent AGIC d’appliquer des configurations,

élargissez les autorisations AGIC avec :
1. Créez un élément `AzureIngressProhibitedTarget` avec votre configuration spécifique :
    ```bash
    cat <<EOF | kubectl apply -f -
    apiVersion: "appgw.ingress.k8s.io/v1"
    kind: AzureIngressProhibitedTarget
    metadata:
      name: your-custom-prohibitions
    spec:
      hostname: your.own-hostname.com
    EOF
    ```

2. Une fois que vous avez créé votre interdiction personnalisée, vous pouvez supprimer la valeur par défaut, qui est trop large :

    ```bash
    kubectl delete AzureIngressProhibitedTarget prohibit-all-targets
    ```

### <a name="enable-for-an-existing-agic-installation"></a>Activer pour une installation AGIC existante
Supposons que nous disposons déjà d’une instance AKS opérationnelle, d’une instance Application Gateway et d’un AGIC configurés dans notre cluster. Nous avons une entrée pour `prod.contoso.com` et traitons correctement le trafic de celui-ci à partir de l’instance AKS. Nous souhaitons ajouter `staging.contoso.com` à notre instance Application Gateway existante, mais devons l’héberger sur une [machine virtuelle](https://azure.microsoft.com/services/virtual-machines/). Nous allons réutiliser l’instance Application Gateway existante et configurer manuellement un écouteur et des pools principaux pour `staging.contoso.com`. Cependant, la modification manuelle de la configuration de l’instance Application Gateway (via le [portail](https://portal.azure.com), les [API ARM](/rest/api/resources/) ou [Terraform](https://www.terraform.io/)) poserait problème car AGIC possède en totalité l’instance Application Gateway. Quelques instants après l’application de modifications, AGIC les remplacerait ou les supprimerait.

Nous pouvons empêcher AGIC d’apporter des modifications à un sous-ensemble de la configuration.

1. Créez un objet `AzureIngressProhibitedTarget` :
    ```bash
    cat <<EOF | kubectl apply -f -
    apiVersion: "appgw.ingress.k8s.io/v1"
    kind: AzureIngressProhibitedTarget
    metadata:
      name: manually-configured-staging-environment
    spec:
      hostname: staging.contoso.com
    EOF
    ```

2. Affichez l’objet nouvellement créé :
    ```bash
    kubectl get AzureIngressProhibitedTargets
    ```

3. Modifiez la configuration de l’instance Application Gateway via le portail : ajoutez des écouteurs, des règles d’acheminement, des serveurs principaux, etc. Le nouvel objet que nous avons créé (`manually-configured-staging-environment`) empêchera AGIC de remplacer la configuration de l’instance Application Gateway liée à `staging.contoso.com`.
