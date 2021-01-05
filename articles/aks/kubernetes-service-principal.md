---
title: Principaux de service pour Azure Kubernetes Services (AKS)
description: Créer et gérer un principal de service Azure Active Directory pour un cluster dans Azure Kubernetes Service (AKS)
services: container-service
ms.topic: conceptual
ms.date: 06/16/2020
ms.openlocfilehash: e95eae3ab8d992bc169e54700e7e31715e72102e
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96607821"
---
# <a name="service-principals-with-azure-kubernetes-service-aks"></a>Principaux de service avec Azure Kubernetes Service (AKS)

Pour interagir avec des API Azure, un cluster AKS nécessite un [principal de service Azure Active Directory (AD)][aad-service-principal] ou une [identité managée](use-managed-identity.md). Un principal de service ou une identité managée est nécessaire pour la création et la gestion dynamiques d’autres ressources Azure, comme un équilibreur de charge ou un registre de conteneurs Azure (ACR).

Cet article montre comment créer et utiliser un principal de service pour vos clusters AKS.

## <a name="before-you-begin"></a>Avant de commencer

Pour créer un principal de service Azure AD, vous devez disposer des autorisations suffisantes pour inscrire une application auprès de votre client Azure AD et l’affecter à un rôle dans votre abonnement. Si vous n’avez pas les privilèges nécessaires, vous devriez demander à votre administrateur Azure AD ou administrateur d’abonnement de vous attribuer les privilèges nécessaires, de ou créer au préalable un principal de service que vous pourrez utiliser avec le cluster AKS.

Si vous utilisez le principal du service d’un autre locataire Azure AD, il y a d’autres points à prendre en considération quant aux autorisations disponibles au moment de déployer le cluster. Vous n’avez peut-être pas les autorisations adéquates pour lire et écrire des informations d’annuaire. Pour plus d’informations, consultez [Quelles sont les autorisations utilisateur par défaut dans Azure Active Directory ?][azure-ad-permissions]

Azure CLI 2.0.59 (ou une version ultérieure) doit également être installé et configuré. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

## <a name="automatically-create-and-use-a-service-principal"></a>Créer et utiliser un principal de service automatiquement

Lorsque vous créez un cluster AKS dans le portail Azure ou à l’aide de la commande [az aks create][az-aks-create], Azure peut générer automatiquement un principal de service.

Dans l’exemple d’Azure CLI suivant, un principal de service n’est pas spécifié. Dans ce scénario, Azure CLI crée un principal de service pour le cluster AKS. Pour effectuer cette opération, votre compte Azure doit disposer des droits nécessaires pour la création d’un principal de service.

```azurecli
az aks create --name myAKSCluster --resource-group myResourceGroup
```

## <a name="manually-create-a-service-principal"></a>Créer manuellement un principal de service

Pour créer manuellement un principal de service avec Azure CLI, utilisez la commande [az ad sp create-for-rbac][az-ad-sp-create]. Dans l’exemple suivant, le paramètre `--skip-assignment` empêche toute affectation par défaut supplémentaire :

```azurecli-interactive
az ad sp create-for-rbac --skip-assignment --name myAKSClusterServicePrincipal
```

Le résultat ressemble à l’exemple qui suit. Prenez note de vos propres valeurs pour `appId` et `password`. Ces valeurs sont utilisées lorsque vous créez un cluster AKS dans la section suivante.

```json
{
  "appId": "559513bd-0c19-4c1a-87cd-851a26afd5fc",
  "displayName": "myAKSClusterServicePrincipal",
  "name": "http://myAKSClusterServicePrincipal",
  "password": "e763725a-5eee-40e8-a466-dc88d980f415",
  "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db48"
}
```

## <a name="specify-a-service-principal-for-an-aks-cluster"></a>Spécifier un principal de service pour un cluster AKS

Pour utiliser un principal de service existant lorsque vous créez un cluster AKS à l’aide la commande [az aks create][az-aks-create], utilisez les paramètres `--service-principal` et `--client-secret` pour préciser `appId` et `password` à partir de la sortie de la commande [az ad sp create-for-rbac][az-ad-sp-create].

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --service-principal <appId> \
    --client-secret <password>
```

> [!NOTE]
> Si vous utilisez un principal de service existant avec un secret personnalisé, assurez-vous que le secret ne dépasse pas 190 octets.

Si vous déployez un cluster AKS à l’aide du portail Azure, sur la page *Authentification* de la boîte de dialogue **Créer un cluster Kubernetes**, choisissez **Configurer un principal de service**. Sélectionnez **Utiliser existant** et spécifiez les valeurs suivantes :

- **ID client du principal de service** correspond à votre *appId*
- **Secret client du principal de service** correspond à la valeur *password*

![Image de la navigation vers Azure Vote](media/kubernetes-service-principal/portal-configure-service-principal.png)

## <a name="delegate-access-to-other-azure-resources"></a>Déléguer l’accès à d’autres ressources Azure

Vous pouvez utiliser le principal du service du cluster AKS pour accéder à d’autres ressources. Par exemple, si vous souhaitez déployer votre cluster AKS dans un sous-réseau de réseau virtuel Azure existant ou vous connecter à ACR (Azure Container Registry), vous devez déléguer l’accès à ces ressources au principal du service.

Pour déléguer des autorisations, créez une attribution de rôle à l’aide de la commande [az role assignment create][az-role-assignment-create]. Affectez `appId` à une étendue spécifique, comme un groupe de ressources ou une ressource de réseau virtuel. Un rôle définit ensuite les autorisations dont le principal du service dispose sur la ressource, comme indiqué dans l’exemple suivant :

```azurecli
az role assignment create --assignee <appId> --scope <resourceScope> --role Contributor
```

L’élément `--scope` d’une ressource doit être un ID de ressource complet, tel que */subscriptions/\<guid\>/resourceGroups/myResourceGroup* ou  */subscriptions/\<guid\>/resourceGroups/myResourceGroupVnet/providers/Microsoft.Network/virtualNetworks/myVnet*

> [!NOTE]
> Si vous avez supprimé l’attribution de rôle contributeur du groupe de ressources de nœud, les opérations ci-dessous peuvent échouer.  

Les sections suivantes détaillent les délégations courantes que vous serez peut-être amené à effectuer.

### <a name="azure-container-registry"></a>Azure Container Registry

Si vous utilisez Azure Container Registry (ACR) comme magasin d’images conteneur, vous devez accorder des autorisations au principal de service pour que votre cluster AKS puisse lire et extraire des images. Actuellement, la configuration recommandée est d’utiliser la commande [az aks create ][az-aks-create] ou la commande [az aks update][az-aks-update] pour l’intégration à un registre et l’attribution du rôle approprié au principal de service. Pour connaître les étapes détaillées, consultez [S’authentifier avec Azure Container Registry à partir d’Azure Kubernetes Service][aks-to-acr].

### <a name="networking"></a>Mise en réseau

Vous pouvez utiliser un réseau avancé dans lequel le réseau virtuel et le sous-réseau, ou les adresses IP publiques, se trouvent dans un autre groupe de ressources. Assignez un des ensembles d’autorisations de rôle suivants :

- Créez un [rôle personnalisé][rbac-custom-role] et définissez les autorisations de rôle suivantes :
  - *Microsoft.Network/virtualNetworks/subnets/join/action*
  - *Microsoft.Network/virtualNetworks/subnets/read*
  - *Microsoft.Network/publicIPAddresses/join/action*
  - *Microsoft.Network/publicIPAddresses/read*
  - *Microsoft.Network/publicIPAddresses/write*
  - Si vous utilisez [des tables de routage personnalisées sur des clusters Kubenet](configure-kubenet.md#bring-your-own-subnet-and-route-table-with-kubenet) ajoutez ces autorisations supplémentaires :
    - *Microsoft.Network/routeTables/write*
    - *Microsoft.Network/routeTables/read*
- Ou attribuez le rôle intégré [Contributeur de réseaux][rbac-network-contributor] sur le sous-réseau dans le réseau virtuel

### <a name="storage"></a>Stockage

Vous pouvez avoir besoin d’accéder à des ressources disque existantes dans un autre groupe de ressources. Assignez un des ensembles d’autorisations de rôle suivants :

- Créez un [rôle personnalisé][rbac-custom-role] et définissez les autorisations de rôle suivantes :
  - *Microsoft.Compute/disks/read*
  - *Microsoft.Compute/disks/write*
- Ou attribuez le rôle intégré [Contributeur de comptes de stockage][rbac-storage-contributor] sur le groupe de ressources

### <a name="azure-container-instances"></a>Azure Container Instances

Si vous utilisez Virtual Kubelet pour intégrer à AKS et choisissez d’exécuter Azure Container Instances (ACI) dans un groupe de ressources distinct sur le cluster AKS, le principal du service AKS doit disposer de l’autorisation *Contributeur* sur le groupe de ressources ACI.

## <a name="additional-considerations"></a>Considérations supplémentaires

Lorsque vous travaillez avec des principaux de service AKS et Azure AD, gardez les points suivants à l’esprit.

- Le principal de service pour Kubernetes fait partie de la configuration du cluster. Toutefois, n’utilisez pas l’identité pour déployer le cluster.
- Par défaut, les informations d’identification du principal du service sont valides pendant un an. Vous pouvez à tout moment [mettre à jour ou faire pivoter les informations d’identification du principal du service][update-credentials].
- Chaque principal de service est associé à une application Azure AD. Le principal de service pour un cluster Kubernetes peut être associé à tout nom d’application Azure AD valide (par exemple : *https://www.contoso.org/example* ). L’URL de l’application ne doit pas être un point de terminaison réel.
- Lorsque vous spécifiez **l’ID client** du principal de service, utilisez la valeur de `appId`.
- Sur les machines virtuelles du nœud de l’agent du cluster Kubernetes, les informations d’identification du principal du service sont stockées dans le fichier `/etc/kubernetes/azure.json`.
- Si vous utilisez la commande [az aks create][az-aks-create] pour générer automatiquement le principal de service, les informations d’identification du principal de service sont écrites dans le fichier `~/.azure/aksServicePrincipal.json` sur la machine utilisée pour exécuter la commande.
- Si vous ne transmettez pas spécifiquement un principal de service dans des commandes CLI AKS supplémentaires, le principal de service par défaut situé dans `~/.azure/aksServicePrincipal.json` est utilisé.  
- Vous pouvez également supprimer le fichier aksServicePrincipal.json, auquel cas AKS créera un principal de service.
- Si vous supprimez un cluster AKS qui a été créé par [az aks create][az-aks-create], le principal de service qui a été créé automatiquement ne sera pas supprimé.
    - Pour supprimer le principal du service, recherchez votre *servicePrincipalProfile.clientId* de cluster et procédez à la suppression en utilisant [az ad app delete][az-ad-app-delete]. Remplacez les noms de cluster et de groupe de ressources suivants par vos propres valeurs :

        ```azurecli
        az ad sp delete --id $(az aks show -g myResourceGroup -n myAKSCluster --query servicePrincipalProfile.clientId -o tsv)
        ```

## <a name="troubleshoot"></a>Dépanner

Les informations d’identification du principal du service d’un cluster AKS sont mises en cache par Azure CLI. Si ces informations d’identification ont expiré, vous rencontrez des erreurs de déploiement des clusters AKS. Lors de l’exécution de [az aks create][az-aks-create], le message d’erreur suivant peut indiquer un problème lié aux informations d’identification du principal du service mises en cache :

```console
Operation failed with status: 'Bad Request'.
Details: The credentials in ServicePrincipalProfile were invalid. Please see https://aka.ms/aks-sp-help for more details.
(Details: adal: Refresh request failed. Status Code = '401'.
```

Pour vérifier l’âge du fichier d’informations d’identification, utilisez la commande suivante :

```console
ls -la $HOME/.azure/aksServicePrincipal.json
```

Le délai d’expiration par défaut des informations d’identification du principal du service est d’une année. Si votre fichier *aksServicePrincipal.json* a plus d’un an, supprimez-le et essayez de redéployer un cluster AKS.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les principaux de service Azure Active Directory, consultez [Objets application et principal de service][service-principal].

Pour plus d’informations sur la mise à jour des informations d’identification, consultez [Mettre à jour ou faire pivoter les informations d’identification d’un principal de service dans AKS][update-credentials].

<!-- LINKS - internal -->
[aad-service-principal]:../active-directory/develop/app-objects-and-service-principals.md
[acr-intro]: ../container-registry/container-registry-intro.md
[az-ad-sp-create]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[azure-load-balancer-overview]: ../load-balancer/load-balancer-overview.md
[install-azure-cli]: /cli/azure/install-azure-cli
[service-principal]:../active-directory/develop/app-objects-and-service-principals.md
[user-defined-routes]: ../load-balancer/load-balancer-overview.md
[az-ad-app-list]: /cli/azure/ad/app#az-ad-app-list
[az-ad-app-delete]: /cli/azure/ad/app#az-ad-app-delete
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-update]: /cli/azure/aks#az-aks-update
[rbac-network-contributor]: ../role-based-access-control/built-in-roles.md#network-contributor
[rbac-custom-role]: ../role-based-access-control/custom-roles.md
[rbac-storage-contributor]: ../role-based-access-control/built-in-roles.md#storage-account-contributor
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[aks-to-acr]: cluster-container-registry-integration.md
[update-credentials]: update-credentials.md
[azure-ad-permissions]: ../active-directory/fundamentals/users-default-permissions.md
