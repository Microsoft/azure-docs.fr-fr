---
title: Créer une machine virtuelle Azure avec mise en réseau accélérée à l’aide d’Azure CLI
description: Découvrez comment créer une machine virtuelle Linux avec la mise en réseau accélérée activée.
services: virtual-network
documentationcenter: na
author: gsilva5
manager: gedegrac
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/10/2019
ms.author: gsilva
ms.custom: ''
ms.openlocfilehash: bccbfed96dd6cd87bdfe986baf4b52817a160ac0
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95533359"
---
# <a name="create-a-linux-virtual-machine-with-accelerated-networking-using-azure-cli"></a>Créer une machine virtuelle Linux avec mise en réseau accélérée à l’aide d’Azure CLI

Ce didacticiel explique comment créer une machine virtuelle Linux avec mise en réseau accélérée. Pour créer une machine virtuelle Windows avec accélération réseau, consultez [Créer une machine virtuelle Windows avec accélération réseau](create-vm-accelerated-networking-powershell.md). Une mise en réseau accélérée permet d’opérer une virtualisation d’E/S d’une racine unique (SR-IOV) sur une machine virtuelle, ce qui améliore considérablement les performances de mise en réseau. Cette voie hautement performante court-circuite l’hôte à partir du chemin d’accès aux données, réduisant ainsi la latence, l’instabilité et l’utilisation du processeur pour servir les charges de travail réseau les plus exigeantes sur les types de machines virtuelles pris en charge. L’illustration suivante montre la communication entre deux machines virtuelles avec ou sans mise en réseau accélérée :

![Comparaison](./media/create-vm-accelerated-networking/accelerated-networking.png)

Sans mise en réseau accélérée, tout le trafic réseau en direction et en provenance de la machine virtuelle doit transiter par l’hôte et le commutateur virtuel. Le commutateur virtuel fournit au trafic réseau toutes les stratégies, telles que les groupes de sécurité réseau, les listes de contrôle d’accès, l’isolation et d’autres services de réseau virtualisé. Pour plus d’informations sur les commutateurs virtuels, voir [Vue d’ensemble de la virtualisation de réseau Hyper-V](https://technet.microsoft.com/library/jj945275.aspx).

Dans le cas d'une mise en réseau accélérée, le trafic réseau parvient à la carte réseau de la machine virtuelle avant d'être transféré vers celle-ci. Toutes les stratégies réseau que le commutateur virtuel applique sont déchargées et appliquées dans le matériel. L’application de la stratégie au niveau du matériel permet à la carte réseau de transférer le trafic directement à la machine virtuelle, en ignorant l’hôte et le commutateur virtuel, tout en conservant toutes les stratégies qu’il a appliquées dans l’hôte.

Les avantages d’une mise en réseau accélérée s’appliquent uniquement à la machine virtuelle activée. Pour des résultats optimaux, il convient d'activer cette fonctionnalité sur au moins deux machines virtuelles connectées au même réseau virtuel Azure. Lors de la communication entre réseaux virtuels ou d’une connexion locale, cette fonctionnalité a une incidence minimale sur la latence globale.

## <a name="benefits"></a>Avantages
* **Latence plus faible/Nombre supérieur de paquets par seconde (pps) :** la suppression du commutateur virtuel du chemin de données évite que les paquets liés au traitement des stratégies séjournent sur l'hôte, ce qui augmente le nombre de paquets pouvant être traités dans la machine virtuelle.
* **Instabilité réduite :** le traitement du commutateur virtuel dépend de la stratégie à appliquer et de la charge de travail du processeur qui effectue le traitement. Le déchargement des stratégies sur le matériel supprime cette variabilité en fournissant des paquets directement à la machine virtuelle, en supprimant l’hôte dans la communication de la machine virtuelle et toutes les interruptions de logiciel et les changements de contexte.
* **Utilisation réduite du processeur :** en contournant le commutateur virtuel de l'hôte, le processeur utilise moins de ressources pour traiter le trafic réseau.

## <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge
Les distributions suivantes sont prises en charge sans configuration supplémentaire à partir de la galerie Azure : 
* **Ubuntu 14.04 avec le noyau linux-azure**
* **Ubuntu 16.04 ou version ultérieure** 
* **SLES12 SP3 ou version ultérieure** 
* **RHEL 7.4 ou version ultérieure**
* **CentOS 7.4 ou version ultérieure**
* **CoreOS Linux**
* **Debian « Stretch » avec noyau backports, Debian « Buster » ou versions ultérieures**
* **Oracle Linux 7.4 et versions ultérieures avec Red Hat Compatible Kernel (RHCK)**
* **Oracle Linux 7.5 et versions ultérieures avec UEK version 5**
* **FreeBSD 10.4, 11.1 & 12.0 ou versions ultérieures**

## <a name="limitations-and-constraints"></a>Limitations et restrictions

### <a name="supported-vm-instances"></a>Instances de machines virtuelles prises en charge
La mise en réseau accélérée est prise en charge dans la plupart des instances d’usage général et optimisées pour le calcul (2 processeurs virtuels ou plus).  Séries prises en charge : D/DSv2 et F/Fs

Dans des instances qui acceptent l’hyperthreading, la mise en réseau accélérée est prise en charge dans des instances de machine virtuelle comptant au minimum 4 processeurs virtuels. Séries prises en charge : D/Dsv3, D/Dsv4, Dd/Ddv4, Da/Dasv4, E/Esv3, E/Esv4, Ed/Edsv4, Ea/Easv4, Fsv2, Lsv2, Ms/Mms et Ms/Mmsv2.

Pour plus d’informations sur les instances de machine virtuelle, consultez la section [Tailles des machines virtuelles Linux](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

### <a name="custom-images"></a>Images personnalisées
Si vous utilisez une image personnalisée et si votre image prend en charge l’accélération réseau, veillez à disposer des pilotes nécessaires pour utiliser les cartes réseau Mellanox ConnectX-3 et ConnectX-4 Lx dans Azure.

### <a name="regions"></a>Régions
Disponible dans toutes les régions Azure publiques ainsi que dans les clouds Azure Government.

<!-- ### Network interface creation 
Accelerated networking can only be enabled for a new NIC. It cannot be enabled for an existing NIC.
removed per issue https://github.com/MicrosoftDocs/azure-docs/issues/9772 -->
### <a name="enabling-accelerated-networking-on-a-running-vm"></a>Activation de la mise en réseau accélérée sur une machine virtuelle en cours d’exécution
Cette fonctionnalité peut être activée sur une machine virtuelle sans mise en réseau accélérée uniquement si la machine virtuelle a une taille prise en charge et si elle est arrêtée/libérée.  
### <a name="deployment-through-azure-resource-manager"></a>Déploiement par le biais d’Azure Resource Manager
Le déploiement de machines virtuelles (classiques) avec mise en réseau accélérée n’est pas possible.

## <a name="create-a-linux-vm-with-azure-accelerated-networking"></a>Créer une machine virtuelle Linux avec mise en réseau accélérée Azure
## <a name="portal-creation"></a>Création de portail
Bien que cet article fournit des étapes pour créer une machine virtuelle avec mise en réseau accélérée à l’aide de l’interface CLI d’Azure, vous pouvez également [Créer une machine virtuelle avec mise en réseau accélérée via le portail Azure](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Lorsque vous créez une machine virtuelle dans le portail, dans le panneau **Créer une machine virtuelle**, choisissez l’onglet **Mise en réseau**.  Dans cet onglet, il existe une option de **Mise en réseau accélérée**.  Si vous avez choisi un [système d’exploitation pris en charge](#supported-operating-systems) et une [taille de machine virtuelle](#supported-vm-instances), cette option est automatiquement définie sur « Activé ».  Dans le cas contraire, l’option est définie sur « Désactivé » pour la Mise en réseau accélérée et l’utilisateur est informé de la raison pour laquelle elle ne peut pas être activée.   

* *Remarque :* Seuls les systèmes d’exploitation pris en charge peuvent être activés via le portail.  Si vous utilisez une image personnalisée, et que votre image prend en charge la Mise en réseau accélérée, créez votre machine virtuelle à l’aide de CLI ou PowerShell. 

Une fois la machine virtuelle créée, vous pouvez confirmer l’activation de la Mise en réseau accélérée en suivant les instructions de la section [Confirmer l’activation de la Mise en réseau accélérée](#confirm-that-accelerated-networking-is-enabled).

## <a name="cli-creation"></a>Création de CLI
### <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Installez la dernière version [d’Azure CLI](/cli/azure/install-azure-cli) et connectez-vous à un compte Azure avec [az login](/cli/azure/reference-index). Dans les exemples suivants, remplacez les exemples de noms de paramètre par vos propres valeurs. Les noms de paramètre sont par exemple *myResourceGroup*, *myNic* et *myVm*.

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *centralus* :

```azurecli
az group create --name myResourceGroup --location centralus
```

Sélectionnez une région Linux prise en charge et répertoriée dans [Mise en réseau accélérée Linux](https://azure.microsoft.com/updates/accelerated-networking-in-expanded-preview).

Créez un réseau virtuel avec la commande [az network vnet create](/cli/azure/network/vnet). L’exemple suivant crée un réseau virtuel nommé *myVnet* avec un sous-réseau :

```azurecli
az network vnet create \
    --resource-group myResourceGroup \
    --name myVnet \
    --address-prefix 192.168.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefix 192.168.1.0/24
```

### <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau
Créez un groupe de sécurité réseau avec la commande [az network nsg create](/cli/azure/network/nsg). L’exemple suivant crée un groupe de sécurité réseau nommé *myNetworkSecurityGroup* :

```azurecli
az network nsg create \
    --resource-group myResourceGroup \
    --name myNetworkSecurityGroup
```

Le groupe de sécurité réseau contient plusieurs règles par défaut, dont l’une désactive tous les accès entrants à partir d’Internet. Ouvrez un port pour autoriser l’accès SSH à la machine virtuelle avec [az network nsg rule create](/cli/azure/network/nsg/rule) :

```azurecli
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNetworkSecurityGroup \
  --name Allow-SSH-Internet \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 100 \
  --source-address-prefix Internet \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range 22
```

### <a name="create-a-network-interface-with-accelerated-networking"></a>Créer une interface réseau avec mise en réseau accélérée

Créez une adresse IP publique avec la commande [az network public-ip create](/cli/azure/network/public-ip). Une adresse IP publique n’est pas nécessaire si vous ne prévoyez pas d’accéder à l’ordinateur virtuel depuis Internet. Par contre, elle l’est pour procéder selon les étapes mentionnées dans cet article.

```azurecli
az network public-ip create \
    --name myPublicIp \
    --resource-group myResourceGroup
```

Créez une interface réseau avec [az network nic create](/cli/azure/network/nic), puis activez la mise en réseau accélérée. L’exemple suivant permet de créer une interface réseau nommée *myNic* dans le sous-réseau *mySubnet* du réseau virtuel *myVnet* et de lui associer le groupe de sécurité réseau *myNetworkSecurityGroup* :

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic \
    --vnet-name myVnet \
    --subnet mySubnet \
    --accelerated-networking true \
    --public-ip-address myPublicIp \
    --network-security-group myNetworkSecurityGroup
```

### <a name="create-a-vm-and-attach-the-nic"></a>Créer une machine virtuelle et attacher la carte réseau
Lorsque vous créez la machine virtuelle, spécifiez la carte réseau que vous avez générée avec `--nics`. Sélectionnez une taille et une distribution répertoriées dans [Mise en réseau accélérée Linux](https://azure.microsoft.com/updates/accelerated-networking-in-expanded-preview). 

Créez une machine virtuelle avec la commande [az vm create](/cli/azure/vm). L’exemple suivant crée une machine virtuelle nommée *myVM* avec l’image UbuntuLTS et une taille qui prend en charge la mise en réseau accélérée (*Standard_DS4_v2*) :

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --size Standard_DS4_v2 \
    --admin-username azureuser \
    --generate-ssh-keys \
    --nics myNic
```

Pour obtenir la liste de toutes les tailles de machine virtuelle et leurs caractéristiques, consultez [Tailles de machines virtuelles Linux](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Une fois que la machine virtuelle est créée, une sortie similaire à la sortie suivante est renvoyée. Veuillez noter **publicIpAddress**. Cette adresse sera utilisée pour accéder à la machine virtuelle dans les étapes suivantes.

```output
{
  "fqdns": "",
  "id": "/subscriptions/<ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "centralus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "192.168.0.4",
  "publicIpAddress": "40.68.254.142",
  "resourceGroup": "myResourceGroup"
}
```

### <a name="confirm-that-accelerated-networking-is-enabled"></a>Confirmer l’activation de la mise en réseau accélérée

Utilisez la commande suivante pour créer une session SSH avec la machine virtuelle. Remplacez `<your-public-ip-address>` par l’adresse IP publique affectée à la machine virtuelle que vous avez créée, puis remplacez *azureuser* si vous avez utilisé une valeur différente pour `--admin-username` au moment de la création de la machine virtuelle.

```bash
ssh azureuser@<your-public-ip-address>
```

À partir de l’interpréteur de commandes Bash, entrez `uname -r`, puis confirmez que la version du noyau est bien l’une des versions suivantes ou une version supérieure :

* **Ubuntu 16.04** : 4.11.0-1013
* **SLES SP3** : 4.4.92-6.18
* **RHEL** : 7.4.2017120423
* **CentOS** : 7.4.20171206


Utilisez la commande `lspci` pour confirmer que l’appareil Mellanox VF est exposé à la machine virtuelle. Le résultat renvoyé ressemble à la sortie suivante :

```output
0000:00:00.0 Host bridge: Intel Corporation 440BX/ZX/DX - 82443BX/ZX/DX Host bridge (AGP disabled) (rev 03)
0000:00:07.0 ISA bridge: Intel Corporation 82371AB/EB/MB PIIX4 ISA (rev 01)
0000:00:07.1 IDE interface: Intel Corporation 82371AB/EB/MB PIIX4 IDE (rev 01)
0000:00:07.3 Bridge: Intel Corporation 82371AB/EB/MB PIIX4 ACPI (rev 02)
0000:00:08.0 VGA compatible controller: Microsoft Corporation Hyper-V virtual VGA
0001:00:02.0 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
```

Utilisez la commande `ethtool -S eth0 | grep vf_` pour rechercher l’activité sur VF (fonction virtuelle). Si vous recevez une sortie semblable à l’exemple suivant, la mise en réseau accélérée est donc activée et elle fonctionne.

```output
vf_rx_packets: 992956
vf_rx_bytes: 2749784180
vf_tx_packets: 2656684
vf_tx_bytes: 1099443970
vf_tx_dropped: 0
```
La mise en réseau accélérée est maintenant activée pour votre machine virtuelle.

## <a name="handle-dynamic-binding-and-revocation-of-virtual-function"></a>Gérer la liaison dynamique et la révocation de fonction virtuelle 
Les applications doivent s’exécuter via la carte réseau synthétique qui est exposée dans la machine virtuelle. Si l’application s’exécute directement via la carte réseau de fonction virtuelle, elle ne reçoit pas **tous** les paquets destinés à la machine virtuelle, car certains paquets s’affichent via l’interface synthétique.
Si vous exécutez une application via la carte réseau synthétique, ceci garantit que l’application reçoit **tous** les paquets qui lui sont destinés. Il est par ailleurs certain que l’application continuera à s’exécuter, même si la fonction virtuelle est révoquée lorsque l’hôte est en cours de maintenance. La liaison des applications à la carte réseau synthétique est une exigence **obligatoire** pour toutes les applications tirant parti de la **mise en réseau accélérée**.

## <a name="enable-accelerated-networking-on-existing-vms"></a>Activer la mise en réseau accélérée sur des machines virtuelles existantes
Si vous avez créé une machine virtuelle sans mise en réseau accélérée, vous pouvez activer cette fonctionnalité sur une machine virtuelle existante.  La machine virtuelle doit prendre en charge la mise en réseau accélérée et remplir les prérequis suivants (ces prérequis ont déjà été décrits plus haut) :

* La machine virtuelle doit avoir une taille prise en charge pour la mise en réseau accélérée
* La machine virtuelle doit être une image de la galerie Azure prise en charge (et une version de noyau prise en charge pour Linux)
* Toutes les machines virtuelles membres d’un groupe à haute disponibilité ou d’un groupe de machines virtuelles identiques doivent être arrêtées ou libérées avant l’activation de la mise en réseau accélérée sur une carte réseau

### <a name="individual-vms--vms-in-an-availability-set"></a>Machines virtuelles individuelles et machines virtuelles d’un groupe à haute disponibilité
Tout d’abord, arrêtez/libérez la machine virtuelle individuelle ou, dans le cas d’un groupe à haute disponibilité, toutes les machines virtuelles du groupe :

```azurecli
az vm deallocate \
    --resource-group myResourceGroup \
    --name myVM
```

Important : Si la machine virtuelle a été créée individuellement, en dehors d’un groupe à haute disponibilité, arrêtez/libérez simplement la machine virtuelle individuelle pour activer la mise en réseau accélérée.  Si la machine virtuelle a été créée dans un groupe à haute disponibilité, vous devez arrêter/libérer toutes les machines virtuelles membres de ce groupe avant d’activer la mise en réseau accélérée sur les cartes réseau. 

Après avoir arrêté la machine virtuelle, activez la mise en réseau accélérée sur la carte réseau de la machine virtuelle :

```azurecli
az network nic update \
    --name myNic \
    --resource-group myResourceGroup \
    --accelerated-networking true
```

Redémarrez la machine virtuelle ou, dans le cas d’un groupe à haute disponibilité, toutes les machines virtuelles du groupe, puis vérifiez que la mise en réseau accélérée est bien activée : 

```azurecli
az vm start --resource-group myResourceGroup \
    --name myVM
```

### <a name="vmss"></a>Groupe de machines virtuelles identiques (VMSS)
Un groupe de machines virtuelles identiques fonctionne un peu différemment, mais suit le même workflow.  Tout d’abord, arrêtez les machines virtuelles :

```azurecli
az vmss deallocate \
    --name myvmss \
    --resource-group myrg
```

Une fois que les machines virtuelles sont arrêtées, mettez à jour la propriété Accelerated Networking sur l’interface réseau :

```azurecli
az vmss update --name myvmss \
    --resource-group myrg \
    --set virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].enableAcceleratedNetworking=true
```

Notez que, dans un groupe de machines virtuelles identiques, les machines virtuelles sont mises à niveau selon trois méthodes de mise à jour différentes : manuelle, automatique et propagée.  Dans ces instructions, la stratégie est définie sur automatique afin que le groupe de machines virtuelles identiques récupère les mises à jour aussitôt après le redémarrage.  Pour définir la stratégie sur automatique afin que les mises à jour soient immédiatement récupérées : 

```azurecli
az vmss update \
    --name myvmss \
    --resource-group myrg \
    --set upgradePolicy.mode="automatic"
```

Enfin, redémarrez le groupe de machines virtuelles identiques :

```azurecli
az vmss start \
    --name myvmss \
    --resource-group myrg
```

Après le redémarrage, attendez que les mises à niveau se terminent. Vous voyez alors la carte VF apparaître sur la machine virtuelle.  (Vérifiez que vous utilisez un système d’exploitation et des tailles de machine virtuelle pris en charge.)

### <a name="resizing-existing-vms-with-accelerated-networking"></a>Redimensionnement des machines virtuelles existantes avec mise en réseau accélérée

Les machines virtuelles avec mise en réseau accélérée peuvent être redimensionnées uniquement en machines virtuelles qui prennent en charge la mise en réseau accélérée.  

Une machine virtuelle avec mise en réseau accélérée ne peut pas être redimensionnée en une instance de machine virtuelle qui ne prend pas en charge la mise en réseau accélérée.  Pour redimensionner l’une de ces machines virtuelles, vous devez effectuer ces opérations : 

* Arrêtez/libérez la machine virtuelle ou, dans le cas d’un groupe à haute disponibilité ou d’un groupe de machines virtuelles identiques, arrêtez/libérez toutes les machines virtuelles du groupe.
* Désactivez la mise en réseau accélérée sur la carte réseau de la machine virtuelle ou, dans le cas d’un groupe à haute disponibilité ou d’un groupe de machines virtuelles identiques, de toutes les machines virtuelles du groupe.
* Une fois que la mise en réseau accélérée est désactivée, redimensionnez la machine virtuelle, ou toutes les machines virtuelles du groupe à haute disponibilité ou du groupe de machines virtuelles identiques, à une nouvelle taille qui ne prend pas en charge la mise en réseau accélérée, et redémarrez la ou les machines.  

