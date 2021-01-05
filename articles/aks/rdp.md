---
title: Se connecter avec RDP à des nœuds Windows Server AKS
titleSuffix: Azure Kubernetes Service
description: Découvrez comment créer une connexion RDP avec des nœuds Windows Server de cluster AKS (Azure Kubernetes Service) pour les tâches de maintenance et de résolution des problèmes.
services: container-service
ms.topic: article
ms.date: 06/04/2019
ms.openlocfilehash: 4f221a98adb779d32fc5567b201699672c8b90f5
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183275"
---
# <a name="connect-with-rdp-to-azure-kubernetes-service-aks-cluster-windows-server-nodes-for-maintenance-or-troubleshooting"></a>Se connecter avec RDP à des nœuds Windows Server de cluster AKS (Azure Kubernetes Service) à des fins de maintenance ou de résolution des problèmes

Tout au long du cycle de vie de votre cluster AKS (Azure Kubernetes Service ), vous pouvez être amené à accéder à un nœud Windows Server AKS, que ce soit pour effectuer une tâche de maintenance, une collecte de journaux ou d’autres opérations de dépannage. Vous pouvez accéder aux nœuds Windows Server AKS à l’aide de RDP. Ou bien, si vous souhaitez utiliser SSH pour accéder aux nœuds Windows Server AKS et que vous avez accès à la paire de clés qui a été utilisée pendant la création du cluster, vous pouvez suivre les étapes décrites dans [SSH dans des nœuds de cluster AKS (Azure Kubernetes Service)][ssh-steps]. Pour des raisons de sécurité, les nœuds AKS ne sont pas exposés à Internet.

Cet article vous montre comment créer une connexion RDP avec des nœuds AKS en utilisant leurs adresses IP privées.

## <a name="before-you-begin"></a>Avant de commencer

Cet article part de l’hypothèse que vous disposez déjà d’un cluster AKS avec un nœud Windows Server. Si vous avez besoin d’un cluster AKS, consultez l’article traitant de la [création d’un cluster AKS avec un conteneur Windows à partir d’Azure CLI][aks-windows-cli]. Vous avez besoin du nom d’utilisateur et du mot de passe de l’administrateur Windows du nœud Windows Server dont vous souhaitez résoudre les problèmes. Si vous ne les connaissez pas, vous pouvez les réinitialiser en suivant la procédure [Réinitialiser les Services Bureau à distance ou le mot de passe administrateur associé dans une machine virtuelle Windows](../virtual-machines/troubleshooting/reset-rdp.md). Vous avez aussi besoin d’un client RDP comme [Bureau à distance Microsoft][rdp-mac].

Azure CLI version 2.0.61 ou ultérieure doit également être installé et configuré. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

## <a name="deploy-a-virtual-machine-to-the-same-subnet-as-your-cluster"></a>Déployer une machine virtuelle dans le même sous-réseau que votre cluster

Les nœuds Windows Server de votre cluster AKS n’ont pas d’adresses IP accessibles en externe. Pour établir une connexion RDP, vous pouvez déployer une machine virtuelle dotée d’une adresse IP accessible publiquement dans le même sous-réseau que vos nœuds Windows Server.

Dans l’exemple suivant, une machine virtuelle nommée *myVM* est créée dans le groupe de ressources *myResourceGroup*.

Dans un premier temps, obtenez le sous-réseau utilisé par votre pool de nœuds Windows Server. Pour obtenir l’ID du sous-réseau, vous avez besoin du nom du sous-réseau. Pour obtenir le nom du sous-réseau, vous avez besoin du nom du réseau virtuel. Obtenez le nom du réseau virtuel en demandant la liste des réseaux à votre cluster. Pour interroger le cluster, vous avez besoin de son nom. Vous pouvez obtenir tous ces éléments en exécutant la commande suivante dans Azure Cloud Shell :

```azurecli-interactive
CLUSTER_RG=$(az aks show -g myResourceGroup -n myAKSCluster --query nodeResourceGroup -o tsv)
VNET_NAME=$(az network vnet list -g $CLUSTER_RG --query [0].name -o tsv)
SUBNET_NAME=$(az network vnet subnet list -g $CLUSTER_RG --vnet-name $VNET_NAME --query [0].name -o tsv)
SUBNET_ID=$(az network vnet subnet show -g $CLUSTER_RG --vnet-name $VNET_NAME --name $SUBNET_NAME --query id -o tsv)
```

Maintenant que vous disposez du SUBNET_ID, exécutez la commande suivante dans la même fenêtre Azure Cloud Shell pour créer la machine virtuelle :

```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image win2019datacenter \
    --admin-username azureuser \
    --admin-password myP@ssw0rd12 \
    --subnet $SUBNET_ID \
    --query publicIpAddress -o tsv
```

L’exemple de sortie suivant montre que la machine virtuelle a bien été créée et présente l’adresse IP publique de la machine virtuelle.

```console
13.62.204.18
```

Enregistrez l’adresse IP publique de la machine virtuelle. Vous utiliserez cette adresse dans une étape ultérieure.

## <a name="allow-access-to-the-virtual-machine"></a>Autoriser l’accès à la machine virtuelle

Les sous-réseaux de pools de nœuds AKS sont protégés par défaut avec des groupes de sécurité réseau (NSG). Pour accéder à la machine virtuelle, vous devez activer l’accès au groupe de sécurité réseau.

> [!NOTE]
> Les groupes de sécurité réseau sont contrôlés par le service AKS. Toute modification apportée au groupe de sécurité réseau sera remplacée à tout moment par le plan de contrôle.
>

Tout d’abord, récupérez le groupe de ressources et le nom groupe de sécurité réseau auquel ajouter la règle :

```azurecli-interactive
CLUSTER_RG=$(az aks show -g myResourceGroup -n myAKSCluster --query nodeResourceGroup -o tsv)
NSG_NAME=$(az network nsg list -g $CLUSTER_RG --query [].name -o tsv)
```

Ensuite, créez la règle du groupe de sécurité réseau :

```azurecli-interactive
az network nsg rule create --name tempRDPAccess --resource-group $CLUSTER_RG --nsg-name $NSG_NAME --priority 100 --destination-port-range 3389 --protocol Tcp --description "Temporary RDP access to Windows nodes"
```

## <a name="get-the-node-address"></a>Obtenir l’adresse des nœuds

Pour gérer un cluster Kubernetes, vous utilisez [kubectl][kubectl], le client de ligne de commande Kubernetes. Si vous utilisez Azure Cloud Shell, `kubectl` est déjà installé. Pour installer `kubectl` en local, utilisez la commande [az aks install-cli][az-aks-install-cli] :
    
```azurecli-interactive
az aks install-cli
```

Pour configurer `kubectl` afin de vous connecter à votre cluster Kubernetes, exécutez la commande [az aks get-credentials][az-aks-get-credentials]. Cette commande télécharge les informations d’identification et configure l’interface CLI Kubernetes pour les utiliser.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Listez les adresses IP internes des nœuds Windows Server en utilisant la commande [kubectl get][kubectl-get] :

```console
kubectl get nodes -o wide
```

L’exemple de sortie suivant présente les adresses IP internes de tous les nœuds du cluster, notamment des nœuds Windows Server.

```console
$ kubectl get nodes -o wide
NAME                                STATUS   ROLES   AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                    KERNEL-VERSION      CONTAINER-RUNTIME
aks-nodepool1-42485177-vmss000000   Ready    agent   18h   v1.12.7   10.240.0.4    <none>        Ubuntu 16.04.6 LTS          4.15.0-1040-azure   docker://3.0.4
aksnpwin000000                      Ready    agent   13h   v1.12.7   10.240.0.67   <none>        Windows Server Datacenter   10.0.17763.437
```

Enregistrez l’adresse IP interne du nœud Windows Server dont vous voulez résoudre les problèmes. Vous utiliserez cette adresse dans une étape ultérieure.

## <a name="connect-to-the-virtual-machine-and-node"></a>Se connecter à la machine virtuelle et au nœud

Connectez-vous à l’adresse IP publique de la machine virtuelle que vous avez créée précédemment à l’aide d’un client RDP comme le [Bureau à distance Microsoft][rdp-mac].

![Image illustrant la connexion à la machine virtuelle à l’aide d’un client RDP](media/rdp/vm-rdp.png)

Après vous être connecté à la machine virtuelle, connectez-vous à l’*adresse IP interne* du nœud Windows Server dont vous voulez résoudre les problèmes à l’aide d’un client RDP depuis votre machine virtuelle.

![Image illustrant la connexion au nœud Windows Server à l’aide d’un client RDP](media/rdp/node-rdp.png)

Vous êtes maintenant connecté à votre nœud Windows Server.

![Image de la fenêtre cmd dans le nœud Windows Server](media/rdp/node-session.png)

Vous pouvez maintenant exécuter des commandes de dépannage dans la fenêtre *cmd*. Comme les nœuds Windows Server utilisent Windows Server Core, vous ne disposez pas d’interface GUI complète ou d’autres outils GUI au moment où vous vous connectez à un nœud Windows Server via RDP.

## <a name="remove-rdp-access"></a>Retirer l’accès RDP

Lorsque vous avez terminé, quittez la connexion RDP au nœud Windows Server, puis quittez la session RDP sur la machine virtuelle. Après avoir quitté les deux sessions RDP, supprimez la machine virtuelle avec la commande [az vm delete][az-vm-delete] :

```azurecli-interactive
az vm delete --resource-group myResourceGroup --name myVM
```

Et la règle du groupe de sécurité réseau :

```azurecli-interactive
CLUSTER_RG=$(az aks show -g myResourceGroup -n myAKSCluster --query nodeResourceGroup -o tsv)
NSG_NAME=$(az network nsg list -g $CLUSTER_RG --query [].name -o tsv)
```

```azurecli-interactive
az network nsg rule delete --resource-group $CLUSTER_RG --nsg-name $NSG_NAME --name tempRDPAccess
```

## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin de données de dépannage supplémentaires, vous pouvez [consulter les journaux de nœud principal Kubernetes][view-master-logs] ou [Azure Monitor][azure-monitor-containers].

<!-- EXTERNAL LINKS -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[rdp-mac]: https://aka.ms/rdmac

<!-- INTERNAL LINKS -->
[aks-windows-cli]: windows-container-cli.md
[az-aks-install-cli]: /cli/azure/aks?view=azure-cli-latest#az-aks-install-cli
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[az-vm-delete]: /cli/azure/vm#az-vm-delete
[azure-monitor-containers]: ../azure-monitor/insights/container-insights-overview.md
[install-azure-cli]: /cli/azure/install-azure-cli
[ssh-steps]: ssh.md
[view-master-logs]: view-master-logs.md