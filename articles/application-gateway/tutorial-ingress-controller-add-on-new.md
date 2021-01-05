---
title: 'Tutoriel : Activer le module complémentaire Ingress Controller pour un cluster AKS existant avec une nouvelle instance Azure Application Gateway'
description: Dans ce tutoriel, vous allez apprendre à utiliser Azure CLI afin d’activer le module complémentaire Ingress Controller pour un cluster AKS existant avec une nouvelle instance Application Gateway.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: tutorial
ms.date: 09/24/2020
ms.author: caya
ms.openlocfilehash: 627d5b15a861c3d564cb4db33b366d3227092d37
ms.sourcegitcommit: 192f9233ba42e3cdda2794f4307e6620adba3ff2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96296248"
---
# <a name="tutorial-enable-the-ingress-controller-add-on-preview-for-a-new-aks-cluster-with-a-new-application-gateway-instance"></a>Tutoriel : Activer le module complémentaire Ingress Controller (préversion) pour un cluster AKS existant avec une nouvelle instance Application Gateway

Vous pouvez utiliser Azure CLI pour activer le module complémentaire [AGIC (Application Gateway Ingress Controller)](ingress-controller-overview.md) dans votre cluster [Azure Kubernetes Services (AKS)](https://azure.microsoft.com/services/kubernetes-service/). Le module complémentaire est actuellement en préversion.

Dans ce tutoriel, vous allez créer un cluster AKS avec le module complémentaire AGIC activé. La création du cluster crée automatiquement une instance Azure Application Gateway à utiliser. Vous déploierez ensuite un exemple d’application qui utilisera le module complémentaire pour exposer l’application par le biais de Application Gateway. 

Le module complémentaire offre un moyen beaucoup plus rapide de déployer AGIC pour votre cluster AKS que [précédemment via Helm](ingress-controller-overview.md#difference-between-helm-deployment-and-aks-add-on). Il offre également une expérience complètement managé.    

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créez un groupe de ressources. 
> * Créez un cluster AKS avec le module complémentaire AGIC activé. 
> * Déployez un exemple d’application à l’aide d’AGIC pour l’entrée sur le cluster AKS.
> * Vérifiez que l’application est accessible par le biais de la passerelle Application Gateway.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Ce tutoriel nécessite l’interface Azure CLI version 2.0.4 ou ultérieure. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée. Si vous utilisez Azure CLI, vous devez installer l’extension de préversion sur l’interface CLI à l’aide de la commande suivante, si ce n’est pas déjà fait :
    ```azurecli-interactive
    az extension add --name aks-preview
    ```

 - Enregistrez l’indicateur de fonctionnalité *AKS-IngressApplicationGatewayAddon* à l’aide de la commande [az feature register](/cli/azure/feature#az-feature-register), comme indiqué dans l’exemple suivant. Vous ne devez effectuer cette opération qu’une seule fois par abonnement, alors que le module complémentaire est toujours en préversion.
    ```azurecli-interactive
    az feature register --name AKS-IngressApplicationGatewayAddon --namespace Microsoft.ContainerService
    ```

   Quelques minutes peuvent être nécessaires avant que l’état apparaisse`Registered`. Vous pouvez vérifier l’état de l’inscription à l’aide de la commande [az feature list](/cli/azure/feature#az-feature-register) :
    ```azurecli-interactive
    az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/AKS-IngressApplicationGatewayAddon')].{Name:name,State:properties.state}"
    ```

 - Lorsque vous êtes prêt, actualisez l’inscription du fournisseur de ressources Microsoft.ContainerService à l’aide de la commande [az provider register](/cli/azure/provider#az-provider-register) :
    ```azurecli-interactive
    az provider register --namespace Microsoft.ContainerService
    ```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Dans Azure, vous allouez les ressources associées à un groupe de ressources. Créez un groupe de ressources à l’aide de la commande [az group create](/cli/azure/group#az-group-create). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement (région) *canadacentral* : 

```azurecli-interactive
az group create --name myResourceGroup --location canadacentral
```

## <a name="deploy-an-aks-cluster-with-the-add-on-enabled"></a>Déployez un cluster AKS avec le module complémentaire activé

Déployez ensuite un nouveau cluster AKS avec le module complémentaire AGIC activé. Si vous ne fournissez pas d’instance Application Gateway existante à utiliser dans ce processus, nous allons créer et configurer automatiquement une nouvelle instance Application Gateway pour servir le trafic vers le cluster AKS.  

> [!NOTE]
> Le module complémentaire Application Gateway Ingress Controller (AGIC) prend en charge *uniquement* les références SKU Application Gateway v2 (Standard et WAF), *pas* les références SKU Application Gateway v1. Lorsque vous déployez une nouvelle instance Application Gateway par le biais du module complémentaire AGIC, vous pouvez déployer uniquement une référence SKU Application Gateway Standard_v2. Si vous souhaitez activer le module complémentaire pour une référence SKU WAF_v2 Application Gateway, utilisez l’une des méthodes suivantes :
>
> - Activez le WAF sur Application Gateway via le portail. 
> - Créez d’abord l’instance WAF_v2 Application Gateway, puis suivez les instructions pour [activer le module complémentaire AGIC avec un cluster AKS existant et une instance Application Gateway existante](tutorial-ingress-controller-add-on-existing.md). 

Dans l’exemple suivant, vous allez déployer un nouveau cluster AKS nommé *myCluster* à l’aide de [Azure CNI](../aks/concepts-network.md#azure-cni-advanced-networking) et [des identités managées](../aks/use-managed-identity.md). Le module complémentaire AGIC sera activé dans le groupe de ressources que vous avez créé, *myResourceGroup*. 

Le déploiement d’un nouveau cluster AKS avec le module complémentaire AGIC activé sans spécifier une instance Application Gateway existante entraînera la création automatique d’une instance Application Gateway SKU Standard_v2. Ainsi, vous spécifierez également le nom et l’espace d’adressage du sous-réseau de l’instance Application Gateway. Le nom de l’instance Application Gateway sera *myApplicationGateway* et l’espace d’adressage de sous-réseau que nous utilisons est 10.2.0.0/16. Assurez-vous d’avoir ajouté ou mis à jour l’extension aks-preview au début de ce tutoriel. 

```azurecli-interactive
az aks create -n myCluster -g myResourceGroup --network-plugin azure --enable-managed-identity -a ingress-appgw --appgw-name myApplicationGateway --appgw-subnet-prefix "10.2.0.0/16" --generate-ssh-keys
```

Pour configurer des paramètres supplémentaires dans la commande `az aks create`, consultez ces [informations de référence disponibles](/cli/azure/aks?view=azure-cli-latest#az-aks-create). 

> [!NOTE]
> Le cluster AKS que vous avez créé apparaît dans le groupe de ressources que vous avez créé, *myResourceGroup*. Toutefois, l’instance Application Gateway créée automatiquement se trouve dans le groupe de ressources de nœud, là où se trouvent les pools d’agents. Le groupe de ressources de nœud est nommé *MC_resource-Group-name_cluster-name_location* par défaut, mais il peut être modifié. 

## <a name="deploy-a-sample-application-by-using-agic"></a>Déployer un exemple d’application en utilisant le complément AGIC

Vous allez maintenant déployer un exemple d’application sur le cluster AKS que vous avez créé. L’application utilisera le module complémentaire AGIC pour l’entrée et connectera l’instance Application Gateway au cluster AKS. 

Tout d’abord, exécutez la commande `az aks get-credentials` pour obtenir les informations de connexion au cluster AKS : 

```azurecli-interactive
az aks get-credentials -n myCluster -g myResourceGroup
```

Une fois que vous avez ces informations de connexion, exécutez la commande suivante afin de configurer un exemple d’application qui utilise AGIC pour l’entrée au cluster. AGIC mettra à jour Application Gateway que vous avez configurée précédemment avec les règles d’acheminement correspondantes vers le nouvel exemple d’application que vous avez déployé.  

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/aspnetapp.yaml 
```

## <a name="check-that-the-application-is-reachable"></a>Vérifier que l’application est accessible

Maintenant que la passerelle Application Gateway est configurée pour envoyer le trafic vers le cluster AKS, vous devez vérifier que votre application est accessible. Recherchez d’abord l’adresse IP de l’entrée : 

```azurecli-interactive
kubectl get ingress
```

Vérifiez que l’exemple d’application que vous avez créée est en cours d’exécution par l’une des deux opérations suivantes :

- En visitant l’adresse IP de l’instance Application Gateway que vous avez obtenue à partir de l’exécution de la commande précédente.
- Utilisation de `curl`. 

L’opération de mise à jour par Application Gateway peut prendre quelques minutes. Si Application Gateway est toujours en état de **Mise à jour** sur le portail, attendez que cela soit terminé avant d’essayer d’accéder à l’adresse IP. 

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, l’instance Application Gateway et toutes les ressources associées :

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur la désactivation du module complémentaire AGIC](./ingress-controller-disable-addon.md)
