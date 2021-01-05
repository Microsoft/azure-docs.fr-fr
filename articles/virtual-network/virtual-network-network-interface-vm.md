---
title: Ajouter des interfaces de réseau ou supprimer des machines virtuelles Azure
description: Découvrez comment ajouter ou supprimer des interfaces réseau de machines virtuelles.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/13/2020
ms.author: kumud
ms.openlocfilehash: f7253be2844f40ca52df2f9b3bc9cbba552fea2b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85480131"
---
# <a name="add-network-interfaces-to-or-remove-network-interfaces-from-virtual-machines"></a>Ajouter ou supprimer des interfaces réseau pour des machines virtuelles

Découvrez comment ajouter une interface réseau existante quand vous créez une machine virtuelle Azure, et également comment ajouter ou supprimer des interfaces réseau d’une machine virtuelle existante à l’état arrêté (désalloué). Une interface réseau permet à une machine virtuelle Azure de communiquer avec des ressources sur Internet, sur Azure et locales. Une machine virtuelle a une ou plusieurs interfaces réseau. 

Si vous avez besoin d’ajouter, de modifier ou de supprimer des adresses IP pour une interface réseau, consultez la section sur la [gestion des adresses IP des interfaces réseau](virtual-network-network-interface-addresses.md). Pour créer, modifier ou supprimer des interfaces réseau, consultez [Gérer des interfaces réseau](virtual-network-network-interface.md).

## <a name="before-you-begin"></a>Avant de commencer

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Si vous n’en avez pas, configurez un compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). Effectuez une des tâches suivantes avant de commencer les étapes décrites dans la suite de cet article :

- **Utilisateurs du portail** : Connectez-vous au [portail Azure](https://portal.azure.com) avec votre compte Azure.

- **Utilisateurs de PowerShell** : exécutez les commandes dans [Azure Cloud Shell](https://shell.azure.com/powershell) ou exécutez PowerShell à partir de votre ordinateur. Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. Sous l’onglet de navigateur Azure Cloud Shell, ouvrez la liste déroulante **Sélectionner un environnement**, puis choisissez **PowerShell** si ce n’est pas déjà fait.

    Si vous exécutez PowerShell localement, utilisez le module Azure PowerShell version 1.0.0 ou ultérieure. Exécutez `Get-Module -ListAvailable Az.Network` pour rechercher la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Exécutez `Connect-AzAccount` pour créer une connexion avec Azure.

- **Utilisateurs de l’interface de ligne de commande Azure (CLI)**  : exécutez les commandes dans [Azure Cloud Shell](https://shell.azure.com/bash) ou exécutez l’interface CLI à partir de votre ordinateur. Si vous exécutez l’interface Azure CLI localement, utilisez Azure CLI version 2.0.26 ou ultérieure. Exécutez `az --version` pour rechercher la version installée. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli). Exécutez `az login` pour créer une connexion avec Azure.

## <a name="add-existing-network-interfaces-to-a-new-vm"></a>Ajouter des interfaces réseau existantes à une nouvelle machine virtuelle

Quand vous créez une machine virtuelle par le biais du portail, celui-ci crée une interface réseau avec des paramètres par défaut et l’attache à la machine virtuelle pour vous. Vous ne pouvez pas utiliser le portail pour ajouter des interfaces réseau existantes à une nouvelle machine virtuelle, ni pour créer une machine virtuelle avec plusieurs interfaces réseau. Vous pouvez en revanche effectuer ces deux opérations à l’aide de l’interface de ligne de commande ou de PowerShell. Veillez à vous familiariser avec les [contraintes](#constraints). Si vous créez une machine virtuelle avec plusieurs interfaces réseau, vous devez également configurer le système d’exploitation pour les utiliser correctement une fois la machine virtuelle créée. Découvrez comment configurer [Linux](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) ou [Windows](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) pour plusieurs interfaces réseau.

### <a name="commands"></a>Commandes

Avant de créer la machine virtuelle, [créez une interface réseau](virtual-network-network-interface.md#create-a-network-interface).

|Outil|Commande|
|---|---|
|Interface de ligne de commande|[az network nic create](/cli/azure/network/nic?toc=%2fazure%2fvirtual-network%2ftoc.json#az-network-nic-create)|
|PowerShell|[New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface?toc=%2fazure%2fvirtual-network%2ftoc.json)|

## <a name="add-a-network-interface-to-an-existing-vm"></a>Ajouter une interface réseau à une machine virtuelle existante

Pour ajouter une interface réseau à votre machine virtuelle

1. Accédez au [portail Azure](https://portal.azure.com) pour trouver une machine virtuelle existante. Recherchez et sélectionnez **Machines virtuelles**.

2. Sélectionnez le nom de votre machine virtuelle. La machine virtuelle doit prendre en charge le nombre d’interfaces réseau que vous souhaitez ajouter. Pour connaître le nombre d’interfaces réseau prises en charge par chaque taille de machine virtuelle, consultez les tailles dans Azure pour les [machines virtuelles Linux](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) ou les [machines virtuelles Windows](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

3. Dans la barre de commandes de la machine virtuelle, sélectionnez **Arrêter**, puis **OK** dans la boîte de dialogue de confirmation. Ensuite, attendez que la zone **État** de la machine virtuelle passe à **Arrêté (désalloué)** .

4. Dans la barre de menus de la machine virtuelle, choisissez **Réseau** > **Attacher l’interface réseau**. Ensuite, dans **Attacher l’interface réseau existante**, sélectionnez l’interface réseau que vous souhaitez attacher, puis sélectionnez **OK**.

    >[!NOTE]
    >L’interface réseau que vous sélectionnez doit remplir certaines conditions : l’accélération réseau ne doit pas être activée, aucune adresse IPv6 ne doit lui être affectée et elle doit se trouver sur le même réseau virtuel que l’interface réseau actuellement attachée à la machine virtuelle.

    Si vous ne disposez pas d’une interface réseau existante, vous devez d’abord en créer une. Pour ce faire, sélectionnez **Créer une interface réseau**. Pour plus d’informations sur la création d’une interface réseau, consultez [Créer une interface réseau](virtual-network-network-interface.md#create-a-network-interface). Pour plus d’informations sur les contraintes supplémentaires qui s’appliquent à l’ajout d’interfaces réseau à des machines virtuelles, consultez [Contraintes](#constraints).

5. Dans la barre de menus de la machine virtuelle, choisissez **Vue d’ensemble** > **Démarrer** pour redémarrer la machine virtuelle.

Vous pouvez maintenant configurer le système d’exploitation de la machine virtuelle de façon à utiliser correctement plusieurs interfaces réseau. Découvrez comment configurer [Linux](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) ou [Windows](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) pour plusieurs interfaces réseau.

### <a name="commands"></a>Commandes

|Outil|Commande|
|---|---|
|Interface de ligne de commande|[az vm nic add](/cli/azure/vm/nic?toc=%2fazure%2fvirtual-network%2ftoc.json#az-vm-nic-add) (référence) ; [procédure détaillée](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#add-a-nic-to-a-vm)|
|PowerShell|[Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface?toc=%2fazure%2fvirtual-network%2ftoc.json) (référence) ; [procédure détaillée](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#add-a-nic-to-an-existing-vm)|

## <a name="view-network-interfaces-for-a-vm"></a>Afficher les interfaces réseau d’une machine virtuelle

Vous pouvez afficher les interfaces réseau actuellement attachées à une machine virtuelle pour découvrir la configuration de chaque interface réseau et l’adresse IP qui lui est attribuée. 

1. Accédez au [portail Azure](https://portal.azure.com) pour trouver une machine virtuelle existante. Recherchez et sélectionnez **Machines virtuelles**.

    >[!NOTE]
    >Connectez-vous à l’aide d’un compte disposant des autorisations associées au rôle Propriétaire, Collaborateur ou Collaborateur de réseau pour votre abonnement. Pour en savoir plus sur l’attribution de rôles à des comptes, consultez [Rôles intégrés pour le contrôle d’accès en fonction du rôle Azure](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor).

2. Sélectionnez le nom de la machine virtuelle pour laquelle vous voulez afficher les interfaces réseau attachées.

3. Dans la barre de menus de la machine virtuelle, sélectionnez **Réseau**.

Pour plus d’informations sur les paramètres d’interface réseau et leur modification, consultez [Gérer les interfaces réseau](virtual-network-network-interface.md). Pour savoir comment ajouter, modifier ou supprimer des adresses IP pour une interface réseau, consultez la section sur la [gestion des adresses IP des interfaces réseau](virtual-network-network-interface-addresses.md).

### <a name="commands"></a>Commandes

|Outil|Commande|
|---|---|
|Interface de ligne de commande|[az vm nic list](/cli/azure/vm/nic?toc=%2fazure%2fvirtual-network%2ftoc.json#az-vm-nic-list)|
|PowerShell|[Get-AzVM](/powershell/module/az.compute/get-azvm?toc=%2fazure%2fvirtual-network%2ftoc.json)|

## <a name="remove-a-network-interface-from-a-vm"></a>Supprimer une interface réseau d’une machine virtuelle

1. Accédez au [portail Azure](https://portal.azure.com) pour trouver une machine virtuelle existante. Recherchez et sélectionnez **Machines virtuelles**.

2. Sélectionnez le nom de la machine virtuelle pour laquelle vous voulez afficher les interfaces réseau attachées.

3. Dans la barre d’outils de la machine virtuelle, choisissez **Arrêter**.

4. Attendez que la zone **État** de la machine virtuelle passe à **Arrêté (désalloué)** .

5. Dans la barre de menus de la machine virtuelle, choisissez **Réseau** > **Détacher l’interface réseau**.

6. Dans la boîte de dialogue **Détacher l’interface réseau**, sélectionnez l’interface réseau que vous souhaitez détacher. Sélectionnez ensuite **OK**.

    >[!NOTE]
    >S’il n’y a qu’une seule interface réseau, vous ne pouvez pas la détacher, car une machine virtuelle doit toujours avoir au moins une interface réseau attachée.

### <a name="commands"></a>Commandes

|Outil|Commande|
|---|---|
|Interface de ligne de commande|[az vm nic remove](/cli/azure/vm/nic?toc=%2fazure%2fvirtual-network%2ftoc.json#az-vm-nic-remove) (référence) ; [procédure détaillée](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#remove-a-nic-from-a-vm)|
|PowerShell|[Remove-AzVMNetworkInterface](/powershell/module/az.compute/remove-azvmnetworkinterface?toc=%2fazure%2fvirtual-network%2ftoc.json) (référence) ; [procédure détaillée](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#remove-a-nic-from-an-existing-vm)|

## <a name="constraints"></a>Contraintes

- Une machine virtuelle doit avoir au moins une interface réseau attachée.

- Le nombre d’interfaces réseau d’une machine virtuelle est limité par ce que la taille de machine virtuelle prend en charge. Pour en savoir plus sur le nombre d’interfaces réseau prises en charge par chaque taille de machine virtuelle, consultez les tailles dans Azure pour les [machines virtuelles Linux](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) ou les [machines virtuelles Windows](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Toutes les tailles prennent en charge au moins deux interfaces réseau.

- Actuellement, les interfaces réseau que vous ajoutez à une machine virtuelle ne peuvent pas être attachées à une autre machine virtuelle. Pour plus d’informations sur la création d’une interface réseau, consultez [Créer une interface réseau](virtual-network-network-interface.md#create-a-network-interface).

- Auparavant, vous pouviez ajouter des interfaces réseau uniquement aux machines virtuelles prenant en charge plusieurs interfaces réseau et créées avec au moins deux interfaces réseau. Vous ne pouviez pas ajouter une interface réseau à une machine virtuelle créée avec une seule interface réseau, même si la taille de machine virtuelle prenait en charge plusieurs interfaces réseau. À l’inverse, vous pouviez uniquement supprimer des interfaces réseau à partir d’une machine virtuelle comportant au moins trois interfaces réseau, car les machines virtuelles créées avec au moins deux interfaces réseau devaient toujours avoir au moins deux interfaces réseau. Ces contraintes ne s’appliquent plus. Vous pouvez désormais créer des machines virtuelles avec un nombre quelconque d’interfaces réseau (dans la limite du nombre pris en charge par la taille de machine virtuelle).

- Par défaut, la première interface réseau attachée à une machine virtuelle est l’interface réseau *principale*. Toutes les autres interfaces réseau de la machine virtuelle sont des interfaces réseau *secondaires*.

- Vous pouvez contrôler l’interface réseau vers laquelle vous envoyez le trafic sortant. Toutefois, par défaut une machine virtuelle envoie tout le trafic sortant à l’adresse IP affectée à la configuration IP principale de l’interface réseau principale.

- Auparavant, toutes les machines virtuelles du même groupe à haute disponibilité étaient requises pour une seule ou plusieurs interfaces réseau. Des machines virtuelles comportant un nombre quelconque d’interfaces réseau peuvent désormais exister dans le même groupe à haute disponibilité, pour autant que ce nombre soit pris en charge par la taille de la machine virtuelle. Vous ne pouvez ajouter une machine virtuelle à un groupe à haute disponibilité qu’au moment de la création de celui-ci. Pour en savoir plus sur les groupes à haute disponibilité, consultez [Gérer la disponibilité des machines virtuelles dans Azure](../virtual-machines/windows/manage-availability.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy).

- Vous pouvez connecter des interfaces réseau dans la même machine virtuelle à différents sous-réseaux au sein d’un réseau virtuel. Toutefois, les interfaces réseau doivent toutes être connectées au même réseau virtuel.

- Vous pouvez ajouter n’importe quelle adresse IP pour n’importe quelle configuration IP d’une interface réseau principale ou secondaire à un pool principal Azure Load Balancer. Auparavant, seule l’adresse IP principale de l’interface réseau principale pouvait être ajoutée à un pool principal. Pour en savoir plus sur les configurations et les adresses IP, consultez [Ajouter, modifier ou supprimer des adresses IP](virtual-network-network-interface-addresses.md).

- La suppression d’une machine virtuelle n’a pas pour effet de supprimer les interfaces réseau qui y sont attachées. Lorsque vous supprimez une machine virtuelle, les interfaces réseau sont détachées de la machine virtuelle. Vous pouvez attacher ces interfaces réseau à différentes machines virtuelles, ou les supprimer.

- Pour obtenir des performances optimales, vous devez disposer d’une mise en réseau accélérée. Dans certains cas, vous devez activer explicitement la mise en réseau accélérée pour les machines virtuelles [Windows](create-vm-accelerated-networking-powershell.md) ou [Linux](create-vm-accelerated-networking-cli.md).

## <a name="next-steps"></a>Étapes suivantes

Pour créer une machine virtuelle avec plusieurs interfaces réseau ou adresses IP, consultez :

|Tâche|Outil|
|---|---|
|Créer une machine virtuelle avec plusieurs cartes d’interface réseau|[CLI](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json), [PowerShell](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json)|
|Créer une machine virtuelle à carte réseau unique avec plusieurs adresses IPv4|[CLI](virtual-network-multiple-ip-addresses-cli.md), [PowerShell](virtual-network-multiple-ip-addresses-powershell.md)|
|Créer une machine virtuelle à carte réseau unique avec une adresse IPv6 privée (derrière Azure Load Balancer)|[CLI](../load-balancer/load-balancer-ipv6-internet-cli.md?toc=%2fazure%2fvirtual-network%2ftoc.json), [PowerShell](../load-balancer/load-balancer-ipv6-internet-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json), [modèle Azure Resource Manager](../load-balancer/load-balancer-ipv6-internet-template.md?toc=%2fazure%2fvirtual-network%2ftoc.json)|
