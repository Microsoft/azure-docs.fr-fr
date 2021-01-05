---
title: Créer un cluster privé Azure Red Hat OpenShift 4
description: Découvrez comment créer un cluster privé Azure Red Hat OpenShift exécutant OpenShift 4
ms.service: container-service
ms.topic: article
ms.date: 03/12/2020
author: georgewallace
ms.author: gwallace
keywords: aro, openshift, az aro, red hat, cli
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 3864d48399f00d5cfbdfa0a94939be0d88a73322
ms.sourcegitcommit: dd45ae4fc54f8267cda2ddf4a92ccd123464d411
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92928057"
---
# <a name="create-an-azure-red-hat-openshift-4-private-cluster"></a>Créer un cluster privé Azure Red Hat OpenShift 4

Dans cet article, vous allez préparer votre environnement pour créer des clusters privés Azure Red Hat OpenShift exécutant OpenShift 4. Vous découvrirez comment effectuer les actions suivantes :

> [!div class="checklist"]
> * Préparer les éléments requis et créer le réseau virtuel et les sous-réseaux nécessaires
> * Déployer un cluster avec un point de terminaison de serveur d’API privé et un contrôleur d’entrée privé

Si vous choisissez d’installer et d’utiliser l’interface CLI localement, ce tutoriel demande au minimum la version 2.6.0 d’Azure CLI. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="before-you-begin"></a>Avant de commencer

### <a name="register-the-resource-providers"></a>Inscrire les fournisseurs de ressources

1. Si vous possédez plusieurs abonnements Azure, indiquez l’ID d’abonnement pertinent :

    ```azurecli-interactive
    az account set --subscription <SUBSCRIPTION ID>
    ```

1. Inscrivez le fournisseur de ressources `Microsoft.RedHatOpenShift` :

    ```azurecli-interactive
    az provider register -n Microsoft.RedHatOpenShift --wait
    ```

1. Inscrivez le fournisseur de ressources `Microsoft.Compute` :

    ```azurecli-interactive
    az provider register -n Microsoft.Compute --wait
    ```

1. Inscrivez le fournisseur de ressources `Microsoft.Storage` :

    ```azurecli-interactive
    az provider register -n Microsoft.Storage --wait
    ```

### <a name="get-a-red-hat-pull-secret-optional"></a>Récupération d’un secret d’extraction Red Hat (facultatif)

Un secret d’extraction Red Hat permet au cluster d’accéder à des registres de conteneurs Red Hat et à du contenu supplémentaire. Cette étape est facultative mais recommandée.

1. **[Accédez à votre portail de gestion des clusters Red Hat OpenShift](https://cloud.redhat.com/openshift/install/azure/aro-provisioned) et connectez-vous.**

   Vous devrez vous connecter à votre compte Red Hat existant, ou créer un compte Red Hat avec votre adresse e-mail professionnelle et accepter les conditions générales.

2. **Cliquez sur Download pull secret (Télécharger le secret d’extraction)** .

Conservez le fichier `pull-secret.txt` enregistré à un emplacement sûr, car vous en aurez besoin lors de la création de chaque cluster.

Lors de l’exécution de la commande `az aro create`, vous pouvez référencer votre secret d’extraction à l’aide du paramètre `--pull-secret @pull-secret.txt`. Exécutez `az aro create` à partir du répertoire où vous avez stocké votre fichier `pull-secret.txt`. Sinon, remplacez `@pull-secret.txt` par `@<path-to-my-pull-secret-file`.

Si vous copiez votre secret d’extraction, ou que vous le référencez dans d’autres scripts, il doit se présenter sous la forme d’une chaîne JSON valide.

### <a name="create-a-virtual-network-containing-two-empty-subnets"></a>Créer un réseau virtuel contenant deux sous-réseaux vides

Vous allez maintenant créer un réseau virtuel contenant deux sous-réseaux vides.

1. **Définissez les variables suivantes**.

   ```console
   LOCATION=eastus                 # the location of your cluster
   RESOURCEGROUP="v4-$LOCATION"    # the name of the resource group where you want to create your cluster
   CLUSTER=aro-cluster             # the name of your cluster
   ```

1. **Créer un groupe de ressources**

    Un groupe de ressources Azure est un groupe logique dans lequel des ressources Azure sont déployées et gérées. Lorsque vous créez un groupe de ressources, vous devez spécifier un emplacement. Il s’agit de l’emplacement de stockage des métadonnées de groupe de ressources. C’est également là que vos ressources s’exécutent dans Azure si vous ne spécifiez pas une autre région lors de la création de ressources. Créez un groupe de ressources à l’aide de la commande [az group create][az-group-create].

    ```azurecli-interactive
    az group create --name $RESOURCEGROUP --location $LOCATION
    ```

    L’exemple de sortie suivant montre que le groupe de ressources a été créé correctement :

    ```json
    {
    "id": "/subscriptions/<guid>/resourceGroups/aro-rg",
    "location": "eastus",
    "managedBy": null,
    "name": "aro-rg",
    "properties": {
        "provisioningState": "Succeeded"
    },
    "tags": null
    }
    ```

2. **Créez un réseau virtuel**.

    Les clusters Azure Red Hat OpenShift exécutant OpenShift 4 nécessitent un réseau virtuel avec deux sous-réseaux vides (pour les nœuds principaux et les nœuds worker).

    Créez un réseau virtuel dans le même groupe de ressources que vous avez créé précédemment.

    ```azurecli-interactive
    az network vnet create \
    --resource-group $RESOURCEGROUP \
    --name aro-vnet \
    --address-prefixes 10.0.0.0/22
    ```

    L’exemple de sortie suivant montre le réseau virtuel qui a été créé :

    ```json
    {
    "newVNet": {
        "addressSpace": {
        "addressPrefixes": [
            "10.0.0.0/22"
        ]
        },
        "id": "/subscriptions/<guid>/resourceGroups/aro-rg/providers/Microsoft.Network/virtualNetworks/aro-vnet",
        "location": "eastus",
        "name": "aro-vnet",
        "provisioningState": "Succeeded",
        "resourceGroup": "aro-rg",
        "type": "Microsoft.Network/virtualNetworks"
    }
    }
    ```

3. **Ajoutez un sous-réseau vide pour les nœuds principaux**.

    ```azurecli-interactive
    az network vnet subnet create \
    --resource-group $RESOURCEGROUP \
    --vnet-name aro-vnet \
    --name master-subnet \
    --address-prefixes 10.0.0.0/23 \
    --service-endpoints Microsoft.ContainerRegistry
    ```

4. **Ajoutez un sous-réseau vide pour les nœuds worker**.

    ```azurecli-interactive
    az network vnet subnet create \
    --resource-group $RESOURCEGROUP \
    --vnet-name aro-vnet \
    --name worker-subnet \
    --address-prefixes 10.0.2.0/23 \
    --service-endpoints Microsoft.ContainerRegistry
    ```

5. **[Désactivez les stratégies pour les points de terminaison privés](../private-link/disable-private-link-service-network-policy.md) dans le sous-réseau principal**. Cela est obligatoire pour pouvoir se connecter au cluster et le gérer.

    ```azurecli-interactive
    az network vnet subnet update \
    --name master-subnet \
    --resource-group $RESOURCEGROUP \
    --vnet-name aro-vnet \
    --disable-private-link-service-network-policies true
    ```

## <a name="create-the-cluster"></a>Créer le cluster

Exécutez la commande suivante pour créer un cluster. Si vous le souhaitez, vous pouvez [transmettre votre secret d’extraction Red Hat](#get-a-red-hat-pull-secret-optional) pour permettre au cluster d’accéder à des registres de conteneurs Red Hat et à du contenu supplémentaire.

>[!NOTE]
> Si vous copiez/collez des commandes et utilisez l’un des paramètres facultatifs, veillez à supprimer les mots-dièse initiaux ainsi que le texte de commentaire de fin. De même, fermez l’argument sur la ligne précédente de la commande avec une barre oblique inverse de fin.

```azurecli-interactive
az aro create \
  --resource-group $RESOURCEGROUP \
  --name $CLUSTER \
  --vnet aro-vnet \
  --master-subnet master-subnet \
  --worker-subnet worker-subnet \
  --apiserver-visibility Private \
  --ingress-visibility Private
  # --domain foo.example.com # [OPTIONAL] custom domain
  # --pull-secret @pull-secret.txt # [OPTIONAL]
```

Après l’exécution de la commande `az aro create`, la création d’un cluster prend normalement une trentaine de minutes.

>[!IMPORTANT]
> Si vous choisissez de spécifier un domaine personnalisé, par exemple **foo.example.com** , la console OpenShift sera disponible sur une URL telle que `https://console-openshift-console.apps.foo.example.com`, au lieu de l’URL de domaine intégrée `https://console-openshift-console.apps.<random>.<location>.aroapp.io`.
>
> Par défaut, OpenShift utilise des certificats auto-signés pour toutes les routes créées sur `*.apps.<random>.<location>.aroapp.io`.  Si vous choisissez DNS personnalisé après vous être connecté au cluster, vous devez suivre la documentation OpenShift afin de [configurer une autorité de certification personnalisée pour votre contrôleur d’entrée](https://docs.openshift.com/container-platform/4.3/authentication/certificates/replacing-default-ingress-certificate.html) et configurer une [autorité de certification personnalisée pour votre serveur d’API](https://docs.openshift.com/container-platform/4.3/authentication/certificates/api-server.html).

## <a name="connect-to-the-private-cluster"></a>Établir une connexion au cluster privé

Vous pouvez vous connecter au cluster à l’aide de l’utilisateur `kubeadmin`.  Exécutez la commande suivante pour rechercher le mot de passe de l’utilisateur `kubeadmin`.

```azurecli-interactive
az aro list-credentials \
  --name $CLUSTER \
  --resource-group $RESOURCEGROUP
```

L’exemple de sortie suivant montre que le mot de passe se trouve dans `kubeadminPassword`.

```json
{
  "kubeadminPassword": "<generated password>",
  "kubeadminUsername": "kubeadmin"
}
```

Vous trouverez l’URL de la console de cluster en exécutant la commande suivante, qui se présente comme suit : `https://console-openshift-console.apps.<random>.<region>.aroapp.io/`

```azurecli-interactive
 az aro show \
    --name $CLUSTER \
    --resource-group $RESOURCEGROUP \
    --query "consoleProfile.url" -o tsv
```

>[!IMPORTANT]
> Pour vous connecter à un cluster Azure Red Hat OpenShift privé, vous devez effectuer l’étape suivante à partir d’un hôte qui se trouve sur le réseau virtuel que vous avez créé ou sur un réseau virtuel [appairé](../virtual-network/virtual-network-peering-overview.md) avec le réseau virtuel sur lequel le cluster a été déployé.

Lancez l’URL de la console dans un navigateur et connectez-vous à l’aide des informations d’identification `kubeadmin`.

![Capture d’écran montrant l’écran de connexion Azure Red Hat OpenShift.](media/aro4-login.png)

## <a name="install-the-openshift-cli"></a>Installer l’interface CLI OpenShift

Une fois que vous êtes connecté à la console web OpenShift, cliquez sur **?** en haut à droite, puis sur **Command Line Tools** (Outils en ligne de commande). Téléchargez la version qui convient pour votre ordinateur.

![Écran de connexion Azure Red Hat OpenShift](media/aro4-download-cli.png)

Vous pouvez aussi télécharger la dernière version de l’interface CLI qui convient pour votre ordinateur à partir de <https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/>.

## <a name="connect-using-the-openshift-cli"></a>Se connecter à partir de l’interface CLI OpenShift

Récupérez l’adresse du serveur d’API.

```azurecli-interactive
apiServer=$(az aro show -g $RESOURCEGROUP -n $CLUSTER --query apiserverProfile.url -o tsv)
```

>[!IMPORTANT]
> Pour vous connecter à un cluster Azure Red Hat OpenShift privé, vous devez effectuer l’étape suivante à partir d’un hôte qui se trouve sur le réseau virtuel que vous avez créé ou sur un réseau virtuel [appairé](../virtual-network/virtual-network-peering-overview.md) avec le réseau virtuel sur lequel le cluster a été déployé.

Connectez-vous au serveur d’API du cluster OpenShift à l’aide de la commande suivante. Remplacez **\<kubeadmin password>** par le mot de passe que vous venez de récupérer.

```azurecli-interactive
oc login $apiServer -u kubeadmin -p <kubeadmin password>
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, un cluster Azure Red Hat OpenShift exécutant OpenShift 4 a été déployé. Vous avez appris à :

> [!div class="checklist"]
> * Préparer les éléments requis et créer le réseau virtuel et ses sous-réseaux nécessaires
> * Déployer un cluster
> * Vous connecter au cluster à l’aide de l’utilisateur `kubeadmin`

Passez à l’article suivant afin d’apprendre à configurer le cluster pour l’authentification à l’aide d’Azure Active Directory.


* [Configurer l’authentification avec Azure Active Directory à l’aide de la ligne de commande](configure-azure-ad-cli.md)


* [Configurer l’authentification avec Azure Active Directory à l’aide du portail Azure et de la console web OpenShift](configure-azure-ad-cli.md)
