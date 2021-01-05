---
title: SSH dans les nœuds de cluster Azure Kubernetes Service (AKS)
description: Découvrez comment créer une connexion SSH avec des nœuds de cluster AKS (Azure Kubernetes Service) pour les tâches de maintenance et de dépannage.
services: container-service
ms.topic: article
ms.date: 07/31/2019
ms.openlocfilehash: 50a52584618e505aa2ae7bd9ed7e0a9f6bc330a9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87015610"
---
# <a name="connect-with-ssh-to-azure-kubernetes-service-aks-cluster-nodes-for-maintenance-or-troubleshooting"></a>Se connecter avec SSH à des nœuds de cluster AKS (Azure Kubernetes Service) pour effectuer des tâches de maintenance ou de dépannage

Tout au long du cycle de vie de votre cluster AKS (Azure Kubernetes Service ), vous pouvez être amené à accéder à un nœud AKS, que ce soit pour effectuer une tâche de maintenance, une collecte de journaux ou d’autres opérations de dépannage. Vous pouvez accéder aux nœuds AKS avec SSH, y compris aux nœuds Windows Server. Vous pouvez également [vous connecter aux nœuds Windows Server à l’aide de connexions RDP (Remote Desktop Protocol)][aks-windows-rdp]. Pour des raisons de sécurité, les nœuds AKS ne sont pas exposés à Internet. Pour vous connecter avec SSH aux nœuds AKS, vous utilisez l’adresse IP privée.

Cet article vous montre comment créer une connexion SSH avec un nœud AKS à l’aide de ses adresses IP privées.

## <a name="before-you-begin"></a>Avant de commencer

Cet article suppose que vous avez un cluster AKS existant. Si vous avez besoin d’un cluster AKS, consultez le guide de démarrage rapide d’AKS [avec Azure CLI][aks-quickstart-cli]ou avec le [Portail Azure][aks-quickstart-portal].

Par défaut, les clés SSH sont obtenues ou générées, puis ajoutées aux nœuds lorsque vous créez un cluster AKS. Cet article explique comment spécifier des clés SSH différentes de celles utilisées lors de la création de votre cluster AKS. L’article montre également comment déterminer l’adresse IP privée de votre nœud et vous y connecter à l’aide d’une clé SSH. Si vous n’avez pas besoin de spécifier une autre clé SSH, vous pouvez ignorer l’étape d’ajout de la clé publique SSH au nœud.

Cet article présuppose également que vous disposez d’une clé SSH. Vous pouvez créer une clé SSH à l’aide de [macOS, de Linux][ssh-nix] ou de [Windows][ssh-windows]. Si vous utilisez PuTTY Gen pour créer la paire de clés, enregistrez cette paire de clés dans un format OpenSSH plutôt que dans le format de clé privé PuTTy par défaut (fichier .ppk).

Le logiciel Azure CLI version 2.0.64 ou ultérieure doit également être installé et configuré. Exécutez  `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, consultez  [Installation d’Azure CLI][install-azure-cli].

## <a name="configure-virtual-machine-scale-set-based-aks-clusters-for-ssh-access"></a>Configurer des clusters AKS basés sur des groupes de machines virtuelles pour l’accès à l’aide d’une clé SSH

Pour configurer un cluster AKS basé sur un groupe de machines virtuelles pour l’accès à l’aide d’une clé SSH, recherchez le nom du groupe de machines virtuelles identiques de votre cluster et ajoutez votre clé publique SSH à ce groupe identique.

Utilisez la commande [az aks show][az-aks-show] pour obtenir le nom du groupe de ressources de votre cluster AKS, puis la commande [az vmss list][az-vmss-list] pour obtenir le nom de votre groupe identique.

```azurecli-interactive
CLUSTER_RESOURCE_GROUP=$(az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv)
SCALE_SET_NAME=$(az vmss list --resource-group $CLUSTER_RESOURCE_GROUP --query [0].name -o tsv)
```

L’exemple ci-dessus affecte le nom du groupe de ressources de cluster pour *myAKSCluster* dans *myResourceGroup* sur *CLUSTER_RESOURCE_GROUP*. L’exemple utilise ensuite *CLUSTER_RESOURCE_GROUP* pour afficher le nom du groupe identique et l’affecter à *SCALE_SET_NAME*.

> [!IMPORTANT]
> À ce stade, vous devez uniquement mettre à jour vos clés SSH pour les clusters AKS basés sur un groupe de machines virtuelles identiques à l’aide d’Azure CLI.
> 
> Pour les nœuds Linux, les clés SSH peuvent actuellement uniquement être ajoutées à l’aide d’Azure CLI. Si vous souhaitez vous connecter à un nœud Windows Server à l’aide de SSH, utilisez les clés SSH fournies lors de la création du cluster AKS et ignorez le jeu de commandes suivant pour l’ajout de votre clé publique SSH. Vous avez toujours besoin de l’adresse IP du nœud que vous souhaitez dépanner, comme indiqué dans la dernière commande de cette section. Vous pouvez également [vous connecter aux nœuds Windows Server à l’aide de connexions utilisant le protocole RDP (Remote Desktop Protocol)][aks-windows-rdp] plutôt que d’une clé SSH.

Pour ajouter vos clés SSH aux nœuds dans un groupe de machines virtuelles identiques, utilisez les commandes [az vmss extension set][az-vmss-extension-set] et [az vmss update-instances][az-vmss-update-instances].

```azurecli-interactive
az vmss extension set  \
    --resource-group $CLUSTER_RESOURCE_GROUP \
    --vmss-name $SCALE_SET_NAME \
    --name VMAccessForLinux \
    --publisher Microsoft.OSTCExtensions \
    --version 1.4 \
    --protected-settings "{\"username\":\"azureuser\", \"ssh_key\":\"$(cat ~/.ssh/id_rsa.pub)\"}"

az vmss update-instances --instance-ids '*' \
    --resource-group $CLUSTER_RESOURCE_GROUP \
    --name $SCALE_SET_NAME
```

L’exemple ci-dessus utilise les variables *CLUSTER_RESOURCE_GROUP* et *SCALE_SET_NAME* des commandes précédentes. L’exemple ci-dessus utilise également *~/.ssh/id_rsa.pub* comme emplacement pour votre clé publique SSH.

> [!NOTE]
> Par défaut, le nom d’utilisateur pour les nœuds AKS est *azureuser*.

Une fois que vous avez ajouté votre clé publique SSH au groupe identique, vous pouvez utiliser la clé SSH pour accéder à une machine virtuelle de nœud dans ce groupe identique en utilisant son adresse IP. Affichez les adresses IP privées des nœuds de cluster AKS à l’aide de la commande [kubectl get][kubectl-get].

```console
kubectl get nodes -o wide
```

L’exemple de sortie suivant affiche les adresses IP internes de tous les nœuds du cluster, y compris un nœud Windows Server.

```console
$ kubectl get nodes -o wide

NAME                                STATUS   ROLES   AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                    KERNEL-VERSION      CONTAINER-RUNTIME
aks-nodepool1-42485177-vmss000000   Ready    agent   18h   v1.12.7   10.240.0.4    <none>        Ubuntu 16.04.6 LTS          4.15.0-1040-azure   docker://3.0.4
aksnpwin000000                      Ready    agent   13h   v1.12.7   10.240.0.67   <none>        Windows Server Datacenter   10.0.17763.437
```

Enregistrez l’adresse IP interne du nœud que vous souhaitez dépanner.

Pour accéder à votre nœud à l’aide d’une clé SSH, suivez les étapes décrites dans [Créer la connexion SSH](#create-the-ssh-connection).

## <a name="configure-virtual-machine-availability-set-based-aks-clusters-for-ssh-access"></a>Configurer des clusters AKS basés sur un groupe à haute disponibilité de machines virtuelles pour l’accès à l’aide d’une clé SSH

Pour configurer votre cluster AKS basé sur un groupe à haute disponibilité de machines virtuelles pour l’accès à l’aide d’une clé SSH, recherchez le nom du nœud Linux de votre cluster et ajoutez votre clé publique SSH à ce nœud.

Utilisez la commande [az aks show][az-aks-show] pour obtenir le nom du groupe de ressources de votre cluster AKS, puis la commande [az vmss list][az-vm-list] pour afficher le nom de machine virtuelle du nœud Linux de votre cluster.

```azurecli-interactive
CLUSTER_RESOURCE_GROUP=$(az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv)
az vm list --resource-group $CLUSTER_RESOURCE_GROUP -o table
```

L’exemple ci-dessus affecte le nom du groupe de ressources de cluster pour *myAKSCluster* dans *myResourceGroup* sur *CLUSTER_RESOURCE_GROUP*. L’exemple utilise ensuite la variable *CLUSTER_RESOURCE_GROUP* pour afficher le nom de la machine virtuelle. L’exemple de sortie affiche le nom de la machine virtuelle :

```
Name                      ResourceGroup                                  Location
------------------------  ---------------------------------------------  ----------
aks-nodepool1-79590246-0  MC_myResourceGroupAKS_myAKSClusterRBAC_eastus  eastus
```

Pour ajouter vos clés SSH au nœud, utilisez la commande [az vm user update][az-vm-user-update].

```azurecli-interactive
az vm user update \
    --resource-group $CLUSTER_RESOURCE_GROUP \
    --name aks-nodepool1-79590246-0 \
    --username azureuser \
    --ssh-key-value ~/.ssh/id_rsa.pub
```

L’exemple ci-dessus utilise la variable *CLUSTER_RESOURCE_GROUP* et le nom de la machine virtuelle de nœud obtenus à l’aide des commandes précédentes. L’exemple ci-dessus utilise également *~/.ssh/id_rsa.pub* comme emplacement pour votre clé publique SSH. Vous pouvez également utiliser le contenu de votre clé publique SSH au lieu de spécifier un chemin.

> [!NOTE]
> Par défaut, le nom d’utilisateur pour les nœuds AKS est *azureuser*.

Une fois que vous avez ajouté votre clé publique SSH à la machine virtuelle de nœud, vous pouvez utiliser la clé SSH pour accéder à une machine virtuelle en utilisant son adresse IP. Affichez l’adresse IP privée d’un nœud de cluster AKS à l’aide de la commande [az vm list-ip-addresses][az-vm-list-ip-addresses].

```azurecli-interactive
az vm list-ip-addresses --resource-group $CLUSTER_RESOURCE_GROUP -o table
```

L’exemple ci-dessus utilise la variable *CLUSTER_RESOURCE_GROUP* définit dans les commandes précédentes. L’exemple de sortie suivant montre les adresses IP privées des nœuds AKS :

```
VirtualMachine            PrivateIPAddresses
------------------------  --------------------
aks-nodepool1-79590246-0  10.240.0.4
```

## <a name="create-the-ssh-connection"></a>Créer la connexion SSH

Pour créer une connexion SSH à un nœud AKS, vous exécutez un pod d’assistance dans votre cluster AKS. Ce pod d’assistance vous fournit l’accès SSH au cluster, puis l’accès au nœud SSH. Pour créer et utiliser ce pod d’assistance, effectuez les étapes suivantes :

1. Exécutez une image conteneur `debian` et attachez-y une session de terminal. Vous pouvez ensuite utiliser ce conteneur pour créer une session SSH avec n’importe quel nœud du cluster AKS :

    ```console
    kubectl run -it --rm aks-ssh --image=debian
    ```

    > [!TIP]
    > Si vous utilisez des nœuds Windows Server, ajoutez un sélecteur de nœud à la commande pour planifier le conteneur Debian sur un nœud Linux :
    >
    > ```console
    > kubectl run -it --rm aks-ssh --image=debian --overrides='{"apiVersion":"v1","spec":{"nodeSelector":{"beta.kubernetes.io/os":"linux"}}}'
    > ```

1. Une fois la session du terminal connectée au conteneur, installez un client SSH en utilisant `apt-get` :

    ```console
    apt-get update && apt-get install openssh-client -y
    ```

1. Ouvrez une nouvelle fenêtre de terminal, qui n’est pas connectée à votre conteneur, puis copiez votre clé SSH privée dans le pod d’assistance. Cette clé privée permet de créer la clé SSH permettant d’accéder au nœud AKS. 

   Si nécessaire, remplacez *~/.ssh/id_rsa* par l’emplacement de votre clé SSH privée :

    ```console
    kubectl cp ~/.ssh/id_rsa $(kubectl get pod -l run=aks-ssh -o jsonpath='{.items[0].metadata.name}'):/id_rsa
    ```

1. Revenez à la session de terminal de votre conteneur, mettez à jour les autorisations sur la clé SSH privée `id_rsa` copiée afin qu’elle soit accessible à l’utilisateur en lecture seule :

    ```console
    chmod 0600 id_rsa
    ```

1. Créez une connexion SSH à votre nœud AKS. Là encore, le nom d’utilisateur par défaut pour les nœuds AKS est *azureuser*. Acceptez l’invite pour poursuivre la connexion puisque la clé SSH est approuvée en premier. Ensuite, l’invite bash de votre nœud AKS apparaît :

    ```console
    $ ssh -i id_rsa azureuser@10.240.0.4

    ECDSA key fingerprint is SHA256:A6rnRkfpG21TaZ8XmQCCgdi9G/MYIMc+gFAuY9RUY70.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '10.240.0.4' (ECDSA) to the list of known hosts.

    Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.15.0-1018-azure x86_64)

     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage

      Get cloud support with Ubuntu Advantage Cloud Guest:
        https://www.ubuntu.com/business/services/cloud

    [...]

    azureuser@aks-nodepool1-79590246-0:~$
    ```

## <a name="remove-ssh-access"></a>Supprimer l’accès SSH

Quand vous avez terminé, quittez (`exit`) la session SSH, puis quittez (`exit`) la session de conteneur interactive. Quand cette session de conteneur se ferme, le pod utilisé pour l’accès SSH à partir du cluster AKS est supprimé.

## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin de données de dépannage supplémentaires, vous pouvez consulter [les journaux d’activité kubelet][view-kubelet-logs] ou [les journaux d’activité de nœud principal Kubernetes][view-master-logs].

<!-- EXTERNAL LINKS -->
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

<!-- INTERNAL LINKS -->
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-vm-list]: /cli/azure/vm#az-vm-list
[az-vm-user-update]: /cli/azure/vm/user#az-vm-user-update
[az-vm-list-ip-addresses]: /cli/azure/vm#az-vm-list-ip-addresses
[view-kubelet-logs]: kubelet-logs.md
[view-master-logs]: view-master-logs.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[aks-windows-rdp]: rdp.md
[ssh-nix]: ../virtual-machines/linux/mac-create-ssh-keys.md
[ssh-windows]: ../virtual-machines/linux/ssh-from-windows.md
[az-vmss-list]: /cli/azure/vmss#az-vmss-list
[az-vmss-extension-set]: /cli/azure/vmss/extension#az-vmss-extension-set
[az-vmss-update-instances]: /cli/azure/vmss#az-vmss-update-instances
