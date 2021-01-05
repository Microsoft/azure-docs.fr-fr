---
title: Créer un peering de réseaux virtuels Azure - Modèles de déploiement différents - Même abonnement | Microsoft Docs
description: Découvrez comment créer un peering de réseaux virtuels entre des réseaux virtuels créés via des modèles de déploiement Azure différents qui existent dans le même abonnement.
services: virtual-network
documentationcenter: ''
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/15/2018
ms.author: kumud
ms.reviewer: anavin
ms.openlocfilehash: 148d57da549e8364620c8417cbd61d975cea1498
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87046088"
---
# <a name="create-a-virtual-network-peering---different-deployment-models-same-subscription"></a>Créer un peering de réseaux virtuels Azure - Modèles de déploiement différents, même abonnement

Dans ce didacticiel, vous allez découvrir comment créer un peering de réseaux virtuels entre des réseaux virtuels créés par le biais de modèles de déploiement différents. Les deux réseaux virtuels doivent se trouver dans le même abonnement. Effectuer le peering de deux réseaux virtuels permet aux ressources de différents réseaux virtuels de communiquer entre elles avec la même bande passante et latence, comme si les ressources se trouvaient sur le même réseau virtuel. En savoir plus sur le [peering de réseaux virtuels](virtual-network-peering-overview.md).

Les étapes de création d’un peering de réseaux virtuels sont différentes, selon que les réseaux virtuels sont dans le même abonnement ou dans des abonnements différents, et selon le [modèle de déploiement Azure](../azure-resource-manager/management/deployment-models.md?toc=%2fazure%2fvirtual-network%2ftoc.json) utilisé pour les créer. Découvrez comment créer un peering de réseaux virtuels dans d’autres scénarios en cliquant sur le scénario souhaité dans le tableau suivant :

|Modèle de déploiement Azure  | Abonnement Azure  |
|--------- |---------|
|[Tous deux Resource Manager](tutorial-connect-virtual-networks-portal.md) |Identique|
|[Tous deux Resource Manager](create-peering-different-subscriptions.md) |Différent|
|[Un modèle Resource Manager, un modèle classique](create-peering-different-deployment-models-subscriptions.md) |Différent|

Vous ne pouvez pas créer de peering de réseaux virtuels entre deux réseaux virtuels déployés via le modèle de déploiement classique. Si vous avez besoin de connecter des réseaux virtuels tous deux créés par le biais du modèle de déploiement classique, vous pouvez utiliser une [passerelle VPN](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) Azure.

Ce didacticiel permet d’homologuer des réseaux virtuels situés dans la même région. Vous pouvez également homologuer des réseaux virtuels dans différentes [régions prise en charge](virtual-network-manage-peering.md#cross-region). Il est recommandé de vous familiariser avec les [exigences et contraintes du peering](virtual-network-manage-peering.md#requirements-and-constraints) avant d’effectuer le peering des réseaux virtuels.

Pour créer un peering de réseaux virtuels, vous pouvez utiliser le portail Azure, l'[interface de ligne de commande](#cli) (CLI) Azure, Azure [PowerShell](#powershell) ou un modèle Azure Resource Manager. Cliquez sur les liens des outils précédents pour accéder directement à la procédure permettant de créer un peering de réseaux virtuels à l’aide de l’outil de votre choix.

## <a name="create-peering---azure-portal"></a>Créer un peering - portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com). Le compte auquel vous vous connectez doit avoir les autorisations nécessaires pour créer un peering de réseaux virtuels. Pour obtenir une liste d’autorisations, consultez [Autorisations de peering de réseau virtuel](virtual-network-manage-peering.md#requirements-and-constraints).
2. Cliquez sur **+ Nouveau**, puis sur **Mise en réseau** et **Réseau virtuel**.
3. Dans le panneau **Créer un réseau virtuel**, entrez ou sélectionnez les valeurs des paramètres suivants, puis cliquez sur **Créer** :
    - **Nom** : *myVnet1*
    - **Espace d’adressage** : *10.0.0.0/16*
    - **Nom du sous-réseau** : *par défaut*
    - **Plage d’adresses de sous-réseau** : *10.0.0.0/24*
    - **Abonnement**: Sélectionnez votre abonnement
    - **Groupe de ressources** : Sélectionnez **Créer** et entrez *myResourceGroup*.
    - **Emplacement** : *USA Est*
4. Cliquez sur **+ Nouveau**. Dans le champ **Rechercher dans le marketplace**, tapez *Réseau virtuel*. Quand la mention **Réseau virtuel** apparaît dans les résultats de la recherche, cliquez dessus.
5. Dans le panneau **Réseau virtuel**, sélectionnez **Classique** dans la zone **Sélectionnez un modèle de déploiement**, puis cliquez sur **Créer**.
6. Dans le panneau **Créer un réseau virtuel**, entrez ou sélectionnez les valeurs des paramètres suivants, puis cliquez sur **Créer** :
    - **Nom** : *myVnet2*
    - **Espace d’adressage** : *10.1.0.0/16*
    - **Nom du sous-réseau** : *par défaut*
    - **Plage d’adresses de sous-réseau** : *10.1.0.0/24*
    - **Abonnement**: Sélectionnez votre abonnement
    - **Groupe de ressources** : Sélectionnez **Utiliser l'existant**, puis *myResourceGroup*
    - **Emplacement** : *USA Est*
7. Dans le champ **Rechercher des ressources** située en haut du portail, tapez *myResourceGroup*. Cliquez sur **myResourceGroup** quand il apparaît dans les résultats de la recherche. Un panneau s’affiche pour le groupe de ressources **myresourcegroup**. Le groupe de ressources garde les deux réseaux virtuels créés dans les étapes précédentes.
8. Cliquez sur **myVNet1**.
9. Dans le panneau **myVnet1** qui s’affiche, cliquez sur **Peerings** dans la liste verticale des options sur le côté gauche du panneau.
10. Dans le panneau **myVnet1 - Peerings** qui s’est affiché, cliquez sur **+ Ajouter**.
11. Dans le panneau **Ajouter le peering** qui s’affiche, entrez ou sélectionnez les options suivantes et cliquez sur **OK** :
     - **Nom** : *myVnet1ToMyVnet2*
     - **Modèle de déploiement de réseau virtuel** :  sélectionnez **Classic**.
     - **Abonnement**: Sélectionnez votre abonnement
     - **Réseau virtuel** :  cliquez sur **Choisir un réseau virtuel**, puis sur **myVnet2**.
     - **Autoriser l’accès au réseau virtuel** : vérifiez que **Activé** est sélectionné.
    Il n’y aucun autre paramètre utilisé dans ce didacticiel. Pour en savoir plus sur tous les paramètres d’homologation, consultez [Create, change, or delete a virtual network peering](virtual-network-manage-peering.md#create-a-peering) (Créer, modifier ou supprimer une homologation de réseaux virtuels).
12. Après avoir cliqué sur **OK** à l’étape précédente, le panneau **Ajouter le peering** se ferme et vous pouvez à nouveau voir le panneau **myVnet1 - Peerings**. Après quelques secondes, le peering que vous avez créé apparaît dans le panneau. **Connecté** est indiqué dans la colonne **ÉTAT DE PEERING** pour le peering **myVnet1ToMyVnet2** que vous avez créé.

    Le peering est maintenant établi. Les ressources Azure que vous créez dans un réseau virtuel sont désormais en mesure de communiquer entre elles via leurs adresses IP. Si vous utilisez la résolution de noms Azure par défaut pour les réseaux virtuels, les ressources des réseaux virtuels ne peuvent pas résoudre les noms dans les réseaux virtuels. Si vous souhaitez résoudre les noms dans les réseaux virtuels d’un peering, vous devez créer votre propre serveur DNS. Apprenez à configurer la [résolution de noms à l’aide de votre propre serveur DNS](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).
13. **Facultatif** : bien que la création de machines virtuelles ne soit pas abordée dans ce tutoriel, vous pouvez créer une machine virtuelle sur chaque réseau virtuel et vous connecter d'une machine virtuelle à l'autre pour valider la connectivité.
14. **Facultatif** : pour supprimer les ressources créées dans ce tutoriel, effectuez les étapes décrites dans la section [Supprimer des ressources](#delete-portal) de cet article.

## <a name="create-peering---azure-cli"></a><a name="cli"></a>Créer un peering - interface de ligne de commande Azure

Effectuez les étapes suivantes à l’aide de l’interface Azure Classic CLI et de l’interface Azure CLI. Vous pouvez effectuer les étapes à partir d’Azure Cloud Shell, en sélectionnant simplement le bouton **Essayer** dans toutes les étapes suivantes, ou en installant l’interface [Azure Classic CLI](/cli/azure/install-cli-version-1.0?toc=%2fazure%2fvirtual-network%2ftoc.json) et [Azure CLI](/cli/azure/install-azure-cli?toc=%2fazure%2fvirtual-network%2ftoc.json), puis en exécutant les commandes sur votre ordinateur local.

1. Si vous utilisez Cloud Shell, passez à l’étape 2, car Cloud Shell vous connecte automatiquement à Azure. Ouvrez une session de commande et connectez-vous à Azure à l’aide de la commande `azure login`.
2. Exécutez l’interface CLI en mode Gestion des services en entrant la commande `azure config mode asm`.
3. Entrez la commande suivante pour créer le réseau virtuel (classique) :

   ```azurecli-interactive
   azure network vnet create --vnet myVnet2 --address-space 10.1.0.0 --cidr 16 --location "East US"
   ```

4. Exécutez le script CLI bash suivant à l’aide d’Azure CLI, et non d’Azure Classic CLI. Pour connaître les options d’exécution de scripts CLI bash sur un ordinateur Windows, consultez [Installer Azure CLI sur Windows](/cli/azure/install-azure-cli-windows).

   ```azurecli-interactive
   #!/bin/bash

   # Create a resource group.
   az group create \
     --name myResourceGroup \
     --location eastus

   # Create the virtual network (Resource Manager).
   az network vnet create \
     --name myVnet1 \
     --resource-group myResourceGroup \
     --location eastus \
     --address-prefix 10.0.0.0/16
   ```

5. Créez un peering entre les deux réseaux virtuels créés par le biais des différents modèles de déploiement à l’aide de l’interface CLI. Copiez le script suivant dans un éditeur de texte sur votre PC. Remplacez `<subscription id>` par votre ID d’abonnement. Si vous ne connaissez pas votre ID d’abonnement, entrez la commande `az account show`. La valeur de **id** dans la sortie est votre ID d’abonnement. Collez le script modifié dans votre session CLI, puis appuyez sur `Enter`.

   ```azurecli-interactive
   # Get the ID for VNet1.
   vnet1Id=$(az network vnet show \
     --resource-group myResourceGroup \
     --name myVnet1 \
     --query id --out tsv)

   # Peer VNet1 to VNet2.
   az network vnet peering create \
     --name myVnet1ToMyVnet2 \
     --resource-group myResourceGroup \
     --vnet-name myVnet1 \
     --remote-vnet-id /subscriptions/<subscription id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/myVnet2 \
     --allow-vnet-access
   ```

6. Après l’exécution du script, passez en revue le peering pour le réseau virtuel (Resource Manager). Copiez la commande suivante, collez-la dans votre session CLI, puis appuyez sur `Enter` :

   ```azurecli-interactive
   az network vnet peering list \
     --resource-group myResourceGroup \
     --vnet-name myVnet1 \
     --output table
   ```

   La sortie indique **Connecté** dans la colonne **État de peering**.

   Les ressources Azure que vous créez dans un réseau virtuel sont désormais en mesure de communiquer entre elles via leurs adresses IP. Si vous utilisez la résolution de noms Azure par défaut pour les réseaux virtuels, les ressources des réseaux virtuels ne peuvent pas résoudre les noms dans les réseaux virtuels. Si vous souhaitez résoudre les noms dans les réseaux virtuels d’un peering, vous devez créer votre propre serveur DNS. Apprenez à configurer la [résolution de noms à l’aide de votre propre serveur DNS](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).
7. **Facultatif** : bien que la création de machines virtuelles ne soit pas abordée dans ce tutoriel, vous pouvez créer une machine virtuelle sur chaque réseau virtuel et vous connecter d'une machine virtuelle à l'autre pour valider la connectivité.
8. **Facultatif** : pour supprimer les ressources créées dans ce tutoriel, effectuez les étapes décrites dans la section [Supprimer des ressources](#delete-cli) de cet article.

## <a name="create-peering---powershell"></a><a name="powershell"></a>Créer un peering - PowerShell

1. Installez la dernière version des modules PowerShell [Azure](https://www.powershellgallery.com/packages/Azure) et [Az](https://www.powershellgallery.com/packages/Az/). Si vous débutez dans l’utilisation d’Azure PowerShell, voir [Vue d’ensemble d’Azure PowerShell](/powershell/azure/?toc=%2fazure%2fvirtual-network%2ftoc.json).
2. Démarrez une session PowerShell.
3. Dans PowerShell, connectez-vous à Azure en entrant la commande `Add-AzureAccount`. Le compte auquel vous vous connectez doit avoir les autorisations nécessaires pour créer un peering de réseaux virtuels. Pour obtenir une liste d’autorisations, consultez [Autorisations de peering de réseau virtuel](virtual-network-manage-peering.md#requirements-and-constraints).
4. Pour créer un réseau virtuel (classique) avec PowerShell, vous devez créer un fichier de configuration réseau ou en modifier un existant. Pour découvrir [comment exporter, mettre à jour et importer des fichiers de configuration réseau, consultez cet article](virtual-networks-using-network-configuration-file.md). Le fichier doit inclure l’élément **VirtualNetworkSite** suivant pour le réseau virtuel utilisé dans ce didacticiel :

    ```xml
    <VirtualNetworkSite name="myVnet2" Location="East US">
      <AddressSpace>
        <AddressPrefix>10.1.0.0/16</AddressPrefix>
      </AddressSpace>
      <Subnets>
        <Subnet name="default">
          <AddressPrefix>10.1.0.0/24</AddressPrefix>
        </Subnet>
      </Subnets>
    </VirtualNetworkSite>
    ```

    > [!WARNING]
    > L’importation d’un fichier de configuration réseau modifié peut entraîner des modifications des réseaux virtuels (classiques) existants dans votre abonnement. Assurez-vous de n’ajouter que le réseau virtuel précédent et de ne modifier ou supprimer aucun réseau virtuel existant de votre abonnement.
5. Connectez-vous à Azure pour créer le réseau virtuel (Resource Manager) en entrant la commande `Connect-AzAccount`. Le compte auquel vous vous connectez doit avoir les autorisations nécessaires pour créer un peering de réseaux virtuels. Pour obtenir une liste d’autorisations, consultez [Autorisations de peering de réseau virtuel](virtual-network-manage-peering.md#requirements-and-constraints).
6. Créez un groupe de ressources et un réseau virtuel (Resource Manager). Copiez le script, collez-le dans PowerShell, puis appuyez sur `Enter`.

    ```powershell
    # Create a resource group.
      New-AzResourceGroup -Name myResourceGroup -Location eastus

    # Create the virtual network (Resource Manager).
      $vnet1 = New-AzVirtualNetwork `
      -ResourceGroupName myResourceGroup `
      -Name 'myVnet1' `
      -AddressPrefix '10.0.0.0/16' `
      -Location eastus
    ```

7. Créez un peering de réseaux virtuels entre les deux réseaux virtuels créés par le biais des modèles de déploiement différents. Copiez le script suivant dans un éditeur de texte sur votre PC. Remplacez `<subscription id>` par votre ID d’abonnement. Si vous ne connaissez pas votre ID d’abonnement, entrez la commande `Get-AzSubscription` pour l’afficher. La valeur de **id** dans la sortie retournée est votre ID d’abonnement. Pour exécuter le script, copiez le script modifié à partir de votre éditeur de texte, cliquez avec le bouton droit dans votre session PowerShell, puis appuyez sur `Enter`.

    ```powershell
    # Peer VNet1 to VNet2.
    Add-AzVirtualNetworkPeering `
      -Name myVnet1ToMyVnet2 `
      -VirtualNetwork $vnet1 `
      -RemoteVirtualNetworkId /subscriptions/<subscription Id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/myVnet2
    ```

8. Après l’exécution du script, passez en revue le peering pour le réseau virtuel (Resource Manager). Copiez la commande suivante, collez-la dans votre session PowerShell, puis appuyez sur `Enter` :

    ```powershell
    Get-AzVirtualNetworkPeering `
      -ResourceGroupName myResourceGroup `
      -VirtualNetworkName myVnet1 `
      | Format-Table VirtualNetworkName, PeeringState
    ```

    La sortie indique **Connecté** dans la colonne **État de peering**.

    Les ressources Azure que vous créez dans un réseau virtuel sont désormais en mesure de communiquer entre elles via leurs adresses IP. Si vous utilisez la résolution de noms Azure par défaut pour les réseaux virtuels, les ressources des réseaux virtuels ne peuvent pas résoudre les noms dans les réseaux virtuels. Si vous souhaitez résoudre les noms dans les réseaux virtuels d’un peering, vous devez créer votre propre serveur DNS. Apprenez à configurer la [résolution de noms à l’aide de votre propre serveur DNS](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

9. **Facultatif** : bien que la création de machines virtuelles ne soit pas abordée dans ce tutoriel, vous pouvez créer une machine virtuelle sur chaque réseau virtuel et vous connecter d'une machine virtuelle à l'autre pour valider la connectivité.
10. **Facultatif** : pour supprimer les ressources créées dans ce tutoriel, effectuez les étapes décrites dans la section [Supprimer des ressources](#delete-powershell) de cet article.

## <a name="delete-resources"></a><a name="delete"></a>Supprimer des ressources

Lorsque vous aurez terminé ce didacticiel, vous souhaiterez peut-être supprimer les ressources que vous avez créées, afin que leur utilisation ne soit pas facturée. La suppression d’un groupe de ressources supprime également toutes les ressources qu’il contient.

### <a name="azure-portal"></a><a name="delete-portal"></a>Portail Azure

1. Dans le champ de recherche du portail, entrez **myResourceGroup**. Dans les résultats de la recherche, cliquez sur **myResourceGroup**.
2. Dans le panneau **myResourceGroup**, cliquez sur l’icône **Supprimer**.
3. Pour confirmer la suppression, dans le champ **TAPEZ LE NOM DU GROUPE DE RESSOURCES :** , entrez **myResourceGroup**, puis cliquez sur **Supprimer**.

### <a name="azure-cli"></a><a name="delete-cli"></a>Interface CLI Azure

1. Utilisez Azure CLI pour supprimer le réseau virtuel (Resource Manager) avec la commande suivante :

    ```azurecli-interactive
    az group delete --name myResourceGroup --yes
    ```

2. Utilisez l’interface CLI classique pour supprimer le réseau virtuel (classique) avec les commandes suivantes :

    ```azurecli-interactive
    azure config mode asm

    azure network vnet delete --vnet myVnet2 --quiet
    ```

### <a name="powershell"></a><a name="delete-powershell"></a>PowerShell

1. Entrez la commande suivante pour supprimer le réseau virtuel (Resource Manager) :

    ```powershell
    Remove-AzResourceGroup -Name myResourceGroup -Force
    ```

2. Pour supprimer le réseau virtuel (classique) avec PowerShell, vous devez modifier un fichier de configuration réseau existant. Pour découvrir [comment exporter, mettre à jour et importer des fichiers de configuration réseau, consultez cet article](virtual-networks-using-network-configuration-file.md). Supprimez l’élément VirtualNetworkSite suivant pour le réseau virtuel utilisé dans ce didacticiel :

    ```xml
    <VirtualNetworkSite name="myVnet2" Location="East US">
      <AddressSpace>
        <AddressPrefix>10.1.0.0/16</AddressPrefix>
      </AddressSpace>
      <Subnets>
        <Subnet name="default">
          <AddressPrefix>10.1.0.0/24</AddressPrefix>
        </Subnet>
      </Subnets>
    </VirtualNetworkSite>
    ```

    > [!WARNING]
    > L’importation d’un fichier de configuration réseau modifié peut entraîner des modifications des réseaux virtuels (classiques) existants dans votre abonnement. Assurez-vous de ne supprimer que le réseau virtuel précédent et de ne modifier ou supprimer aucun réseau virtuel existant de votre abonnement.

## <a name="next-steps"></a>Étapes suivantes

- Familiarisez-vous bien avec les [comportements et contraintes importants du peering de réseaux virtuels](virtual-network-manage-peering.md#requirements-and-constraints) avant d’en créer un pour une utilisation en production.
- Apprenez-en davantage sur tous les [paramètres de peering de réseaux virtuels](virtual-network-manage-peering.md#create-a-peering).
- Découvrez comment [créer une topologie de réseau de type hub et spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke#virtual-network-peering) avec le peering de réseaux virtuels.
