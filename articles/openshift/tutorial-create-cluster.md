---
title: Tutoriel - Créer un cluster Azure Red Hat OpenShift 4
description: Découvrez comment créer un cluster Microsoft Azure Red Hat OpenShift à l’aide d’Azure CLI
author: sakthi-vetrivel
ms.author: suvetriv
ms.topic: tutorial
ms.service: container-service
ms.date: 10/26/2020
ms.openlocfilehash: 7b0aead6ada87ca259c838f3f56e68f1030302a2
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92675714"
---
# <a name="tutorial-create-an-azure-red-hat-openshift-4-cluster"></a>Tutoriel : Créer un cluster Azure Red Hat OpenShift 4

Dans ce tutoriel, première partie d’une série qui en compte trois, vous allez préparer votre environnement pour créer un cluster Azure Red Hat OpenShift exécutant OpenShift 4, puis créer un cluster. Vous découvrirez comment effectuer les actions suivantes :
> [!div class="checklist"]
> * Configurer les prérequis 
> * Créer le réseau virtuel et les sous-réseau requis
> * Déployer un cluster

## <a name="before-you-begin"></a>Avant de commencer

Si vous choisissez d’installer et d’utiliser l’interface de ligne de commande localement, ce tutoriel vous demande d’exécuter Azure CLI version 2.6.0 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest).

Azure Red Hat OpenShift requiert a minimum de 40 cœurs pour créer et exécuter un cluster OpenShift. Le quota de ressources Azure par défaut pour un nouvel abonnement Azure ne répond pas à cette exigence. Pour demander une augmentation de votre limite de ressources, consultez [Quota standard : augmenter les limites par série de machines virtuelles](../azure-portal/supportability/per-vm-quota-requests.md).

### <a name="verify-your-permissions"></a>Vérifier vos autorisations

Au cours de ce tutoriel, vous allez créer un groupe de ressources qui contiendra le réseau virtuel pour le cluster. Vous devez disposer d’autorisations d’Administrateur pour l’accès Utilisateur ou Contributeur, ou d’autorisations de Propriétaire, soit directement sur le réseau virtuel, soit sur le groupe de ressources ou l’abonnement qui le contient.

Vous aurez également besoin de suffisamment d’autorisations Azure Active Directory pour que les outils créent une application et un principal du service en votre nom pour le cluster.

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

1. [Accédez au portail de votre gestionnaire de cluster Red Hat OpenShift](https://cloud.redhat.com/openshift/install/azure/aro-provisioned) et connectez-vous.

   Vous devrez vous connecter à votre compte Red Hat existant, ou créer un compte Red Hat avec votre adresse e-mail professionnelle et accepter les conditions générales.

1. Cliquez sur **Download pull secret** et téléchargez un secret d’extraction à utiliser avec votre cluster ARO.

    Conservez le fichier `pull-secret.txt` enregistré dans un emplacement sûr. Le fichier sera utilisé dans chaque création de cluster si vous devez créer un cluster incluant des exemples ou des opérateurs pour Red Hat ou des partenaires certifiés.

    Lors de l’exécution de la commande `az aro create`, vous pouvez référencer votre secret d’extraction à l’aide du paramètre `--pull-secret @pull-secret.txt`. Exécutez `az aro create` à partir du répertoire où vous avez stocké votre fichier `pull-secret.txt`. Sinon, remplacez `@pull-secret.txt` par `@/path/to/my/pull-secret.txt`.

    Si vous copiez votre secret d’extraction, ou que vous le référencez dans d’autres scripts, il doit se présenter sous la forme d’une chaîne JSON valide.

### <a name="prepare-a-custom-domain-for-your-cluster-optional"></a>Préparer un domaine personnalisé pour votre cluster (facultatif)

Lorsque vous exécutez la commande `az aro create`, vous pouvez spécifier un domaine personnalisé pour votre cluster à l’aide du paramètre `--domain foo.example.com`.

Si vous fournissez un domaine personnalisé pour votre cluster, notez les points suivants :

* Après avoir créé votre cluster, vous devez créer deux enregistrements DNS A dans votre serveur DNS pour le `--domain` spécifié :
    * **api**  : pointant vers l’adresse IP du serveur d’API
    * **\*.apps**  : pointant vers l’adresse IP d’entrée
    * Récupérez ces valeurs en exécutant la commande suivante après la création du cluster : `az aro show -n -g --query '{api:apiserverProfile.ip, ingress:ingressProfiles[0].ip}'`.

* La console OpenShift sera disponible à une URL telle que `https://console-openshift-console.apps.example.com`, au lieu du domaine intégré `https://console-openshift-console.apps.<random>.<location>.aroapp.io`.

* Par défaut, OpenShift utilise des certificats auto-signés pour toutes les routes créées sur des domaines personnalisés `*.apps.example.com`.  Si vous choisissez d’utiliser un DNS personnalisé après vous être connecté au cluster, vous devez suivre la documentation OpenShift afin de [configurer une autorité de certification personnalisée pour votre contrôleur d’entrée](https://docs.openshift.com/aro/4/authentication/certificates/replacing-default-ingress-certificate.html) et configurer une [autorité de certification personnalisée pour votre serveur d’API](https://docs.openshift.com/aro/4/authentication/certificates/api-server.html).

### <a name="create-a-virtual-network-containing-two-empty-subnets"></a>Créer un réseau virtuel contenant deux sous-réseaux vides

Vous allez maintenant créer un réseau virtuel contenant deux sous-réseaux vides.

1. **Définissez les variables suivantes dans l’environnement de l’interpréteur de commandes où vous exécuterez les commandes `az`.**

   ```console
   LOCATION=eastus                 # the location of your cluster
   RESOURCEGROUP=aro-rg            # the name of the resource group where you want to create your cluster
   CLUSTER=cluster                 # the name of your cluster
   ```

2. **Créez un groupe de ressources**.

   Un groupe de ressources Azure est un groupe logique dans lequel des ressources Azure sont déployées et gérées. Lorsque vous créez un groupe de ressources, vous devez spécifier un emplacement. Il s’agit de l’emplacement de stockage des métadonnées de groupe de ressources. C’est également là que vos ressources s’exécutent dans Azure si vous ne spécifiez pas une autre région lors de la création de ressources. Créez un groupe de ressources avec la commande [az group create](/cli/azure/group?view=azure-cli-latest#az-group-create).
    
   > [!NOTE] 
   > Azure Red Hat OpenShift n’est pas disponible dans toutes les régions où un groupe de ressources Azure peut être créé. Pour savoir où Azure Red Hat OpenShift est pris en charge, consultez [Régions disponibles](https://azure.microsoft.com/en-gb/global-infrastructure/services/?products=openshift).

   ```azurecli-interactive
   az group create \
     --name $RESOURCEGROUP \
     --location $LOCATION
   ```

   L’exemple de sortie suivant montre que le groupe de ressources a été créé correctement :

   ```json
   {
     "id": "/subscriptions/<guid>/resourceGroups/aro-rg",
     "location": "eastus",
     "name": "aro-rg",
     "properties": {
       "provisioningState": "Succeeded"
     },
     "type": "Microsoft.Resources/resourceGroups"
   }
   ```

2. **Créez un réseau virtuel**.

   Les clusters Azure Red Hat OpenShift exécutant OpenShift 4 nécessitent un réseau virtuel avec deux sous-réseaux vides (pour les nœuds principaux et les nœuds worker).

   Créez un réseau virtuel dans le groupe de ressources que vous avez créé précédemment :

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
       "dhcpOptions": {
         "dnsServers": []
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

5. **[Désactivez les stratégies pour les points de terminaison privés](../private-link/disable-private-link-service-network-policy.md) dans le sous-réseau principal**. C’est nécessaire pour que le service puisse se connecter au cluster et le gérer.

   ```azurecli-interactive
   az network vnet subnet update \
     --name master-subnet \
     --resource-group $RESOURCEGROUP \
     --vnet-name aro-vnet \
     --disable-private-link-service-network-policies true
   ```

## <a name="create-the-cluster"></a>Créer le cluster

Exécutez la commande suivante pour créer un cluster. Si vous choisissez d’utiliser l’une des options suivantes, modifiez la commande en conséquence :
* Si vous le souhaitez, vous pouvez [transmettre votre secret d’extraction Red Hat](#get-a-red-hat-pull-secret-optional) pour permettre au cluster d’accéder à des registres de conteneurs Red Hat et à du contenu supplémentaire. Ajoutez l’argument `--pull-secret @pull-secret.txt` à votre commande.
* Si vous le souhaitez, vous pouvez [utiliser un domaine personnalisé](#prepare-a-custom-domain-for-your-cluster-optional). Ajoutez l’argument `--domain foo.example.com` à votre commande en remplaçant `foo.example.com` par votre propre domaine personnalisé.

> [!NOTE]
> Si vous ajoutez des arguments facultatifs à votre commande, veillez à fermer l’argument situé sur la ligne précédente de la commande à l’aide d’une barre oblique inverse de fin.

```azurecli-interactive
az aro create \
  --resource-group $RESOURCEGROUP \
  --name $CLUSTER \
  --vnet aro-vnet \
  --master-subnet master-subnet \
  --worker-subnet worker-subnet
```

Après l’exécution de la commande `az aro create`, la création d’un cluster prend normalement une trentaine de minutes.

## <a name="next-steps"></a>Étapes suivantes

Dans cette partie du tutoriel, vous avez appris à :
> [!div class="checklist"]
> * Préparer les prérequis, et créer le réseau virtuel et ses sous-réseaux nécessaires
> * Déployer un cluster

Passez au tutoriel suivant :
> [!div class="nextstepaction"]
> [Se connecter à un cluster Azure Red Hat OpenShift](tutorial-connect-cluster.md)
