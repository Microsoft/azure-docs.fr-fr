---
title: Utiliser une adresse IP statique avec équilibrage de charge
titleSuffix: Azure Kubernetes Service
description: Découvrez comment créer et utiliser une adresse IP statique avec l’équilibreur de charge d’Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 11/14/2020
ms.openlocfilehash: 22fd099633556fa9ddce575c2ac238b4950667cb
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94651887"
---
# <a name="use-a-static-public-ip-address-and-dns-label-with-the-azure-kubernetes-service-aks-load-balancer"></a>Utiliser une adresse IP publique statique et une étiquette DNS avec l’équilibrage de charge d’Azure Kubernetes Service (AKS)

Par défaut, l’adresse IP publique affectée à une ressource d’équilibreur de charge créée par un cluster AKS est valide seulement pour la durée de vie de cette ressource. Si vous supprimez le service Kubernetes, l’adresse IP et l’équilibreur de charge associés sont également supprimés. Si vous voulez affecter une adresse IP spécifique ou conserver une adresse IP pour des services Kubernetes redéployés, vous pouvez créer et utiliser une adresse IP publique statique.

Cet article vous montre comment créer une adresse IP publique statique et l’affecter à votre service Kubernetes.

## <a name="before-you-begin"></a>Avant de commencer

Cet article suppose que vous avez un cluster AKS existant. Si vous avez besoin d’un cluster AKS, consultez le guide de démarrage rapide d’AKS [avec Azure CLI][aks-quickstart-cli]ou avec le [Portail Azure][aks-quickstart-portal].

Azure CLI 2.0.59 (ou une version ultérieure) doit également être installé et configuré. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

Cet article traite de l’utilisation d’une adresse IP SKU *standard* avec un équilibreur de charge SKU *standard*. Pour plus d’informations, consultez [Types d’adresses IP et méthodes d’allocation dans Azure][ip-sku].

## <a name="create-a-static-ip-address"></a>Créer une adresse IP statique

Créez une adresse IP publique statique avec la commande [az network public ip create][az-network-public-ip-create]. La commande suivante crée une ressource IP statique nommée *myAKSPublicIP* dans le groupe de ressources *myResourceGroup* :

```azurecli-interactive
az network public-ip create \
    --resource-group myResourceGroup \
    --name myAKSPublicIP \
    --sku Standard \
    --allocation-method static
```

> [!NOTE]
> Si vous utilisez un équilibrer la charge SKU *De base* dans votre cluster AKS, utilisez *De base*  pour le paramètre *sku* lors de la définition d’une adresse IP publique. Seules les adresses IP SKU *De base* fonctionnent avec l’équilibreur de charge SKU *De base* et seuls les adresses IP SKU *Standard* fonctionnent avec les équilibreurs de charge SKU *Standard*. 

L’adresse IP apparaît, comme illustré dans l’exemple de sortie condensée suivante :

```json
{
  "publicIp": {
    ...
    "ipAddress": "40.121.183.52",
    ...
  }
}
```

Vous pouvez obtenir ultérieurement l’adresse IP publique avec la commande [az network public-ip list][az-network-public-ip-list]. Spécifiez le nom du groupe de ressources du nœud et l’adresse IP publique que vos avez créés, puis recherchez *ipAddress* comme indiqué dans l’exemple suivant :

```azurecli-interactive
$ az network public-ip show --resource-group myResourceGroup --name myAKSPublicIP --query ipAddress --output tsv

40.121.183.52
```

## <a name="create-a-service-using-the-static-ip-address"></a>Créer un service utilisant l’adresse IP statique

Avant de créer un service, assurez-vous que Le principal de service utilisé par le cluster AKS a des autorisations déléguées sur l’autre groupe de ressources. Par exemple :

```azurecli-interactive
az role assignment create \
    --assignee <SP Client ID> \
    --role "Network Contributor" \
    --scope /subscriptions/<subscription id>/resourceGroups/<resource group name>
```

Vous pouvez également utiliser l’identité managée affectée par le système pour les autorisations au lieu du principal du service. Pour plus d’informations, consultez [Utiliser des identités managées](use-managed-identity.md).

> [!IMPORTANT]
> Si vous avez personnalisé votre adresse IP sortante, assurez-vous que l’identité de votre cluster dispose des autorisations d’accès tant à l’adresse IP publique sortante qu’à cette adresse IP publique entrante.

Pour créer un service *LoadBalancer* avec l’adresse IP publique statique, ajoutez la propriété `loadBalancerIP` et la valeur de l’adresse IP publique statique au manifeste YAML. Créez un fichier nommé `load-balancer-service.yaml` et copiez-y le YAML suivant. Spécifiez votre propre adresse IP publique créée à l’étape précédente. L’exemple suivant définit aussi l’annotation pour le groupe de ressources nommé *myResourceGroup*. Indiquez le nom de votre propre groupe de ressources.

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-resource-group: myResourceGroup
  name: azure-load-balancer
spec:
  loadBalancerIP: 40.121.183.52
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-load-balancer
```

Créez le service et le déploiement avec la commande `kubectl apply`.

```console
kubectl apply -f load-balancer-service.yaml
```

## <a name="apply-a-dns-label-to-the-service"></a>Appliquer une étiquette DNS au service

Si votre service utilise une adresse IP publique dynamique ou statique, vous pouvez utiliser l'annotation de service `service.beta.kubernetes.io/azure-dns-label-name` pour définir une étiquette DNS destinée au public. Il publie un nom de domaine entièrement qualifié pour votre service en utilisant les serveurs DNS publics et le domaine de niveau supérieur Azure. La valeur de l’annotation devant être unique dans l’emplacement Azure, il est recommandé d’utiliser une étiquette suffisamment qualifiée.   

Azure ajoutera alors automatiquement un sous-réseau par défaut, par exemple `<location>.cloudapp.azure.com` (où emplacement est l’emplacement que vous avez sélectionné) au nom que vous indiquez pour créer le nom DNS complet. Par exemple :

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/azure-dns-label-name: myserviceuniquelabel
  name: azure-load-balancer
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-load-balancer
```

> [!NOTE] 
> Pour publier le service sur votre propre domaine, voir [Azure DNS][azure-dns-zone] et le projet [external-dns][external-dns].

## <a name="troubleshoot"></a>Dépanner

Si l’adresse IP statique définie dans la propriété *loadBalancerIP* du manifeste du service Kubernetes n’existe pas ou n’a pas été créée dans le groupe de ressources du nœud et qu’aucune autre délégation n’est configurée, la création du service d’équilibreur de charge échoue. Pour résoudre les problèmes, examinez les événements de création du service avec la commande [kubectl describe][kubectl-describe]. Spécifiez le nom du service tel qu’il est spécifié dans le manifeste YAML, comme indiqué dans l’exemple suivant :

```console
kubectl describe service azure-load-balancer
```

Les informations sur la ressource de service Kubernetes sont affichées. Les *événements* à la fin de l’exemple de sortie suivant indiquent que *l’adresse IP fournie par l’utilisateur est introuvable*. Dans ces scénarios, vérifiez que vous avez créé l’adresse IP publique statique dans le groupe de ressources du nœud et que l’adresse IP spécifiée dans le manifeste du service Kubernetes est correct.

```
Name:                     azure-load-balancer
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=azure-load-balancer
Type:                     LoadBalancer
IP:                       10.0.18.125
IP:                       40.121.183.52
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32582/TCP
Endpoints:                <none>
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type     Reason                      Age               From                Message
  ----     ------                      ----              ----                -------
  Normal   CreatingLoadBalancer        7s (x2 over 22s)  service-controller  Creating load balancer
  Warning  CreatingLoadBalancerFailed  6s (x2 over 12s)  service-controller  Error creating load balancer (will retry): Failed to create load balancer for service default/azure-load-balancer: user supplied IP Address 40.121.183.52 was not found
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus de contrôle du trafic réseau vers vos applications, vous pouvez plutôt [créer un contrôleur d’entrée][aks-ingress-basic]. Vous pouvez aussi [créer un contrôleur d’entrée avec une adresse IP publique statique][aks-static-ingress].

<!-- LINKS - External -->
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[azure-dns-zone]: https://azure.microsoft.com/services/dns/
[external-dns]: https://github.com/kubernetes-sigs/external-dns

<!-- LINKS - Internal -->
[aks-faq-resource-group]: faq.md#why-are-two-resource-groups-created-with-aks
[az-network-public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[az-network-public-ip-list]: /cli/azure/network/public-ip#az-network-public-ip-list
[az-aks-show]: /cli/azure/aks#az-aks-show
[aks-ingress-basic]: ingress-basic.md
[aks-static-ingress]: ingress-static-ip.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[ip-sku]: ../virtual-network/public-ip-addresses.md#sku
