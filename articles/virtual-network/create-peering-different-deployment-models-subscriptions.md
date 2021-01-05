---
title: Créer un peering de réseaux virtuels Azure - Modèles de déploiement différents - Abonnements différents
titlesuffix: Azure Virtual Network
description: Découvrez comment créer un peering de réseaux virtuels entre des réseaux virtuels créés via des modèles de déploiement Azure différents qui existent dans des abonnements différents.
services: virtual-network
documentationcenter: ''
author: KumudD
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/25/2020
ms.author: kumud
ms.reviewer: anavin
ms.openlocfilehash: 88e576231e0231a105cd9ec303f63307b5eaff89
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87051623"
---
# <a name="create-a-virtual-network-peering---different-deployment-models-and-subscriptions"></a>Créer un peering de réseaux virtuels Azure - Modèles de déploiement et abonnements différents

Dans ce didacticiel, vous allez découvrir comment créer un peering de réseaux virtuels entre des réseaux virtuels créés par le biais de modèles de déploiement différents. Les réseaux virtuels existent dans des abonnements différents. Effectuer le peering de deux réseaux virtuels permet aux ressources de différents réseaux virtuels de communiquer entre elles avec la même bande passante et latence, comme si les ressources se trouvaient sur le même réseau virtuel. En savoir plus sur le [peering de réseaux virtuels](virtual-network-peering-overview.md).

Les étapes de création d’un peering de réseaux virtuels sont différentes, selon que les réseaux virtuels sont dans le même abonnement ou dans des abonnements différents, et selon le [modèle de déploiement Azure](../azure-resource-manager/management/deployment-models.md?toc=%2fazure%2fvirtual-network%2ftoc.json) utilisé pour les créer. Découvrez comment créer un peering de réseaux virtuels dans d’autres scénarios en cliquant sur le scénario souhaité dans le tableau suivant :

|Modèle de déploiement Azure  | Abonnement Azure  |
|--------- |---------|
|[Tous deux Resource Manager](tutorial-connect-virtual-networks-portal.md) |Identique|
|[Tous deux Resource Manager](create-peering-different-subscriptions.md) |Différent|
|[Un modèle Resource Manager, un modèle classique](create-peering-different-deployment-models.md) |Identique|

Vous ne pouvez pas créer de peering de réseaux virtuels entre deux réseaux virtuels déployés via le modèle de déploiement classique. Ce didacticiel utilise des réseaux virtuels situés dans la même région. Ce didacticiel permet d’homologuer des réseaux virtuels situés dans la même région. Vous pouvez également homologuer des réseaux virtuels dans différentes [régions prise en charge](virtual-network-manage-peering.md#cross-region). Il est recommandé de vous familiariser avec les [exigences et contraintes du peering](virtual-network-manage-peering.md#requirements-and-constraints) avant d’effectuer le peering des réseaux virtuels.

Lors de la création d’un appairage de réseaux virtuels entre des réseaux virtuels dans différents abonnements, ceux-ci peuvent être associés au même locataire Azure Active Directory. Si vous n’avez pas encore de locataire Azure Active Directory, vous pouvez rapidement en [créer un](../active-directory/develop/quickstart-create-new-tenant.md?toc=%2fazure%2fvirtual-network%2ftoc.json#create-a-new-azure-ad-tenant).

Vous pouvez utiliser le [portail Azure](#portal), l’[interface de ligne de commande](#cli) ou Azure [PowerShell](#powershell) pour créer un peering de réseaux virtuels. Cliquez sur les liens des outils précédents pour accéder directement à la procédure permettant de créer un peering de réseaux virtuels à l’aide de l’outil de votre choix.

## <a name="create-peering---azure-portal"></a><a name="portal"></a>Créer un peering - portail Azure

Ce didacticiel utilise des comptes différents pour chaque abonnement. Si vous utilisez un compte qui a des autorisations pour les deux abonnements, vous pouvez utiliser le même compte pour toutes les étapes, ignorer les étapes de déconnexion du portail et ignorer les étapes d’affectation d’autorisations d’accès aux réseaux virtuels à un autre utilisateur.

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant que UserA. Le compte auquel vous vous connectez doit avoir les autorisations nécessaires pour créer un peering de réseaux virtuels. Pour obtenir une liste d’autorisations, consultez [Autorisations de peering de réseau virtuel](virtual-network-manage-peering.md#permissions).
2. Cliquez sur **+ Nouveau**, puis sur **Mise en réseau** et **Réseau virtuel**.
3. Dans le panneau **Créer un réseau virtuel**, entrez ou sélectionnez les valeurs des paramètres suivants, puis cliquez sur **Créer** :
    - **Nom** : *myVnetA*
    - **Espace d’adressage** : *10.0.0.0/16*
    - **Nom du sous-réseau** : *par défaut*
    - **Plage d’adresses de sous-réseau** : *10.0.0.0/24*
    - **Abonnement**: sélectionnez l’abonnement A.
    - **Groupe de ressources** : sélectionnez **Créer** et entrez *myResourceGroupA*.
    - **Emplacement** : *USA Est*
4. Dans le champ **Rechercher des ressources** située en haut du portail, tapez *myVnetA*. Quand la mention **myVnetA** apparaît dans les résultats de recherche, cliquez dessus. Un panneau apparaît pour le réseau virtuel **myVnetA**.
5. Dans le panneau **myVnetA** qui apparaît, cliquez sur **Contrôle d’accès (IAM)** dans la liste verticale des options sur le côté gauche du panneau.
6. Dans le panneau **myVnetA - Contrôle d’accès (IAM)** qui apparaît, cliquez sur **+ Ajouter une attribution de rôle**.
7. Dans le panneau **Ajouter une attribution de rôle** qui s’affiche, sélectionnez **Contributeur de réseaux** dans le champ **Rôle**.
8. Dans la zone **Sélectionner**, sélectionnez UserB ou tapez l’adresse e-mail de UserB pour le rechercher. La liste des utilisateurs affichée provient du même locataire Azure Active Directory que le réseau virtuel pour lequel vous configurez le peering. Cliquez sur UserB quand il apparaît dans la liste.
9. Cliquez sur **Enregistrer**.
10. Déconnectez-vous du portail en tant que UserA, puis connectez-vous en tant que UserB.
11. Cliquez sur **+ Nouveau**, tapez *réseau virtuel* dans la zone **Rechercher dans le marketplace**, puis cliquez sur **Réseau virtuel** dans les résultats de recherche.
12. Dans le panneau **Réseau virtuel** qui s’affiche, sélectionnez **Classique** dans la zone **Sélectionnez un modèle de déploiement**, puis cliquez sur **Créer**.
13. Dans la zone Créer un réseau virtuel (classique) qui s’affiche, entrez les valeurs suivantes :

    - **Nom** : *myVnetB*
    - **Espace d’adressage** : *10.1.0.0/16*
    - **Nom du sous-réseau** : *par défaut*
    - **Plage d’adresses de sous-réseau** : *10.1.0.0/24*
    - **Abonnement**: sélectionnez l’abonnement B.
    - **Groupe de ressources** : sélectionnez **Créer** et entrez *myResourceGroupB*.
    - **Emplacement** : *USA Est*

14. Dans le champ **Rechercher des ressources** située en haut du portail, tapez *myVnetB*. Quand la mention **myVnetB** apparaît dans les résultats de recherche, cliquez dessus. Un panneau apparaît pour le réseau virtuel **myVnetB**.
15. Dans le panneau **myVnetB** qui s’affiche, cliquez sur **Propriétés** dans la liste verticale des options sur le côté gauche du panneau. Copiez l’**ID de ressource**, qui vous servira lors d’une étape ultérieure. L’ID de la ressource ressemble à celui de l’exemple qui suit : `/subscriptions/<Subscription ID>/resourceGroups/myResourceGroupB/providers/Microsoft.ClassicNetwork/virtualNetworks/myVnetB`
16. Effectuez les étapes 5 à 9 pour myVnetB, en entrant **UserA** à l’étape 8.
17. Déconnectez-vous du portail en tant que UserB, puis connectez-vous en tant que UserA.
18. Dans le champ **Rechercher des ressources** située en haut du portail, tapez *myVnetA*. Quand la mention **myVnetA** apparaît dans les résultats de recherche, cliquez dessus. Un panneau apparaît pour le réseau virtuel **myVnet**.
19. Cliquez sur **myVnetA**.
20. Dans le panneau **myVnetA** qui s’affiche, cliquez sur **Peerings** dans la liste verticale des options sur le côté gauche du panneau.
21. Dans le panneau **myVnetA - Peerings** qui s’est affiché, cliquez sur **+ Ajouter**.
22. Dans le panneau **Ajouter le peering** qui s’affiche, entrez ou sélectionnez les options suivantes et cliquez sur **OK** :
     - **Nom** : *myVnetAToMyVnetB*
     - **Modèle de déploiement de réseau virtuel** :  sélectionnez **Classic**.
     - **Je connais mon ID de ressource** : cochez cette case.
     - **ID de ressource** : entrez l’ID de ressource de myVnetB noté à l’étape 15.
     - **Autoriser l’accès au réseau virtuel** : vérifiez que **Activé** est sélectionné.
    Il n’y aucun autre paramètre utilisé dans ce didacticiel. Pour en savoir plus sur tous les paramètres d’homologation, consultez [Create, change, or delete a virtual network peering](virtual-network-manage-peering.md#create-a-peering) (Créer, modifier ou supprimer une homologation de réseaux virtuels).
23. Après avoir cliqué sur **OK** à l’étape précédente, le panneau **Ajouter le peering** se ferme et vous pouvez à nouveau voir le panneau **myVnetA - Peerings**. Après quelques secondes, le peering que vous avez créé apparaît dans le panneau. **Connecté** est indiqué dans la colonne **ÉTAT DE PEERING** pour le peering **myVnetAToMyVnetB** que vous avez créé. Le peering est maintenant établi. Il n’est pas nécessaire d’homologuer le réseau virtuel (classique) au réseau virtuel (Resource Manager).

    Les ressources Azure que vous créez dans un réseau virtuel sont désormais en mesure de communiquer entre elles via leurs adresses IP. Si vous utilisez la résolution de noms Azure par défaut pour les réseaux virtuels, les ressources dans les réseaux virtuels ne sont pas en mesure de résoudre les noms dans les réseaux virtuels. Si vous souhaitez résoudre les noms dans les réseaux virtuels d’un peering, vous devez créer votre propre serveur DNS. Apprenez à configurer la [résolution de noms à l’aide de votre propre serveur DNS](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

24. **Facultatif** : bien que la création de machines virtuelles ne soit pas abordée dans ce tutoriel, vous pouvez créer une machine virtuelle sur chaque réseau virtuel et vous connecter d’une machine virtuelle à l’autre pour valider la connectivité.
25. **Facultatif** : pour supprimer les ressources créées dans ce tutoriel, effectuez les étapes décrites dans la section [Supprimer des ressources](#delete-portal) de cet article.

## <a name="create-peering---azure-cli"></a><a name="cli"></a>Créer un peering - interface de ligne de commande Azure

Ce didacticiel utilise des comptes différents pour chaque abonnement. Si vous utilisez un compte qui a des autorisations pour les deux abonnements, vous pouvez utiliser le même compte pour toutes les étapes, ignorer les étapes de déconnexion d’Azure et supprimer les lignes de script qui créent les affectations de rôle utilisateur. Remplacez UserA@azure.com et UserB@azure.com dans tous les scripts suivants par les noms d’utilisateurs que vous utilisez pour UserA et UserB. Effectuez les étapes suivantes à l’aide de l’interface Azure Classic CLI et de l’interface Azure CLI. Vous pouvez effectuer les étapes à partir d’Azure Cloud Shell, en sélectionnant simplement le bouton **Essayer** dans toutes les étapes suivantes, ou en installant l’interface [Azure Classic CLI](/cli/azure/install-classic-cli) et [Azure CLI](/cli/azure/install-azure-cli), puis en exécutant les commandes sur votre ordinateur local.

1. Si vous utilisez Cloud Shell, passez à l’étape 2, car Cloud Shell vous connecte automatiquement à Azure. Ouvrez une session de commande et connectez-vous à Azure à l’aide de la commande `azure login`.
2. Exécutez l’interface CLI classique en mode Gestion des services en entrant la commande `azure config mode asm`.
3. Entrez la commande CLI classique suivante pour créer le réseau virtuel (classique) :

    ```console
    azure network vnet create --vnet myVnetB --address-space 10.1.0.0 --cidr 16 --location "East US"
    ```

4. Les étapes restantes doivent être effectuées à l’aide d’un interpréteur de commandes Bash avec l’interface Azure CLI (pas la classique).
5. Copiez le script suivant dans un éditeur de texte sur votre PC. Remplacez `<SubscriptionB-Id>` par votre ID d’abonnement. Si vous ne connaissez pas votre ID d’abonnement, entrez la commande `az account show`. La valeur de **id** dans la sortie est votre ID d’abonnement. Copiez le script modifié, collez-le dans votre session CLI, puis appuyez sur `Enter`.

    ```azurecli-interactive
    az role assignment create \
      --assignee UserA@azure.com \
      --role "Classic Network Contributor" \
      --scope /subscriptions/<SubscriptionB-Id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/myVnetB
    ```

    Quand vous avez créé le réseau virtuel (classique) à l’étape 4, Azure a créé le réseau virtuel dans le groupe de ressources *Default-Networking*.
6. Déconnectez UserB d’Azure et connectez-vous en tant que UserA dans l’interface CLI.
7. Créez un groupe de ressources et un réseau virtuel (Resource Manager). Copiez le script suivant, collez-le dans votre session CLI, puis appuyez sur `Enter`.

    ```azurecli-interactive
    #!/bin/bash

    # Variables for common values used throughout the script.
    rgName="myResourceGroupA"
    location="eastus"

    # Create a resource group.
    az group create \
      --name $rgName \
      --location $location

    # Create virtual network A (Resource Manager).
    az network vnet create \
      --name myVnetA \
      --resource-group $rgName \
      --location $location \
      --address-prefix 10.0.0.0/16

    # Get the id for myVnetA.
    vNetAId=$(az network vnet show \
      --resource-group $rgName \
      --name myVnetA \
      --query id --out tsv)

    # Assign UserB permissions to myVnetA.
    az role assignment create \
      --assignee UserB@azure.com \
      --role "Network Contributor" \
      --scope $vNetAId
    ```

8. Créez un peering de réseaux virtuels entre les deux réseaux virtuels créés par le biais des modèles de déploiement différents. Copiez le script suivant dans un éditeur de texte sur votre PC. Remplacez `<SubscriptionB-id>` par votre ID d’abonnement. Si vous ne connaissez pas votre ID d’abonnement, entrez la commande `az account show`. La valeur de **id** dans la sortie est votre ID d’abonnement. Azure a créé le réseau virtuel (classique) que vous avez créé à l’étape 4 dans un groupe de ressources nommé *Default-Networking*. Collez le script modifié dans votre session CLI, puis appuyez sur `Enter`.

    ```azurecli-interactive
    # Peer VNet1 to VNet2.
    az network vnet peering create \
      --name myVnetAToMyVnetB \
      --resource-group $rgName \
      --vnet-name myVnetA \
      --remote-vnet-id  /subscriptions/<SubscriptionB-id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/myVnetB \
      --allow-vnet-access
    ```

9. Après l’exécution du script, passez en revue le peering pour le réseau virtuel (Resource Manager). Copiez le script suivant et collez-le dans votre session CLI :

    ```azurecli-interactive
    az network vnet peering list \
      --resource-group $rgName \
      --vnet-name myVnetA \
      --output table
    ```

    La sortie indique **Connecté** dans la colonne **État de peering**.

    Les ressources Azure que vous créez dans un réseau virtuel sont désormais en mesure de communiquer entre elles via leurs adresses IP. Si vous utilisez la résolution de noms Azure par défaut pour les réseaux virtuels, les ressources dans les réseaux virtuels ne sont pas en mesure de résoudre les noms dans les réseaux virtuels. Si vous souhaitez résoudre les noms dans les réseaux virtuels d’un peering, vous devez créer votre propre serveur DNS. Apprenez à configurer la [résolution de noms à l’aide de votre propre serveur DNS](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

10. **Facultatif** : bien que la création de machines virtuelles ne soit pas abordée dans ce tutoriel, vous pouvez créer une machine virtuelle sur chaque réseau virtuel et vous connecter d’une machine virtuelle à l’autre pour valider la connectivité.
11. **Facultatif** : pour supprimer les ressources créées dans ce tutoriel, effectuez les étapes décrites dans la section [Supprimer des ressources](#delete-cli) de cet article.

## <a name="create-peering---powershell"></a><a name="powershell"></a>Créer un peering - PowerShell

Ce didacticiel utilise des comptes différents pour chaque abonnement. Si vous utilisez un compte qui a des autorisations pour les deux abonnements, vous pouvez utiliser le même compte pour toutes les étapes, ignorer les étapes de déconnexion d’Azure et supprimer les lignes de script qui créent les affectations de rôle utilisateur. Remplacez UserA@azure.com et UserB@azure.com dans tous les scripts suivants par les noms d’utilisateurs que vous utilisez pour UserA et UserB. 

1. Installez la dernière version des modules PowerShell [Azure](https://www.powershellgallery.com/packages/Azure) et [Az](https://www.powershellgallery.com/packages/Az). Si vous débutez dans l’utilisation d’Azure PowerShell, voir [Vue d’ensemble d’Azure PowerShell](/powershell/azure/?toc=%2fazure%2fvirtual-network%2ftoc.json).
2. Démarrez une session PowerShell.
3. Dans PowerShell, connectez-vous à l’abonnement de UserB en tant que UserB en entrant la commande `Add-AzureAccount`. Le compte auquel vous vous connectez doit avoir les autorisations nécessaires pour créer un peering de réseaux virtuels. Pour obtenir une liste d’autorisations, consultez [Autorisations de peering de réseau virtuel](virtual-network-manage-peering.md#permissions).
4. Pour créer un réseau virtuel (classique) avec PowerShell, vous devez créer un fichier de configuration réseau ou en modifier un existant. Pour découvrir [comment exporter, mettre à jour et importer des fichiers de configuration réseau, consultez cet article](virtual-networks-using-network-configuration-file.md). Le fichier doit inclure l’élément **VirtualNetworkSite** suivant pour le réseau virtuel utilisé dans ce didacticiel :

    ```xml
    <VirtualNetworkSite name="myVnetB" Location="East US">
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

5. Connectez-vous à l’abonnement de UserB en tant que UserB pour utiliser les commandes de Resource Manager en entrant la commande `Connect-AzAccount`.
6. Affectez à UserA des autorisations sur le réseau virtuel B. Copiez le script suivant dans un éditeur de texte sur votre PC et remplacez `<SubscriptionB-id>` par l’ID de l’abonnement B. Si vous ne connaissez pas l’ID d’abonnement, entrez la commande `Get-AzSubscription` pour l’afficher. La valeur de **id** dans la sortie retournée est votre ID d’abonnement. Azure a créé le réseau virtuel (classique) que vous avez créé à l’étape 4 dans un groupe de ressources nommé *Default-Networking*. Pour exécuter le script, copiez le script modifié, collez-le dans PowerShell, puis appuyez sur `Enter`.

    ```powershell
    New-AzRoleAssignment `
      -SignInName UserA@azure.com `
      -RoleDefinitionName "Classic Network Contributor" `
      -Scope /subscriptions/<SubscriptionB-id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/myVnetB
    ```

7. Déconnectez-vous d’Azure en tant que UserB et connectez-vous à l’abonnement de UserA en tant que UserA en entrant la commande `Connect-AzAccount`. Le compte auquel vous vous connectez doit avoir les autorisations nécessaires pour créer un peering de réseaux virtuels. Pour obtenir une liste d’autorisations, consultez [Autorisations de peering de réseau virtuel](virtual-network-manage-peering.md#permissions).
8. Créez le réseau virtuel (Resource Manager) en copiant le script suivant, en le collant dans PowerShell, puis en appuyant sur `Enter` :

    ```powershell
    # Variables for common values
      $rgName='MyResourceGroupA'
      $location='eastus'

    # Create a resource group.
    New-AzResourceGroup `
      -Name $rgName `
      -Location $location

    # Create virtual network A.
    $vnetA = New-AzVirtualNetwork `
      -ResourceGroupName $rgName `
      -Name 'myVnetA' `
      -AddressPrefix '10.0.0.0/16' `
      -Location $location
    ```

9. Affectez à UserB des autorisations sur myVnetA. Copiez le script suivant dans un éditeur de texte sur votre PC et remplacez `<SubscriptionA-Id>` par l’ID de l’abonnement A. Si vous ne connaissez pas l’ID d’abonnement, entrez la commande `Get-AzSubscription` pour l’afficher. La valeur de **id** dans la sortie retournée est votre ID d’abonnement. Collez la version modifiée du script PowerShell, puis appuyez sur `Enter` pour l’exécuter.

    ```powershell
    New-AzRoleAssignment `
      -SignInName UserB@azure.com `
      -RoleDefinitionName "Network Contributor" `
      -Scope /subscriptions/<SubscriptionA-Id>/resourceGroups/myResourceGroupA/providers/Microsoft.Network/VirtualNetworks/myVnetA
    ```

10. Copiez le script suivant dans un éditeur de texte sur votre ordinateur, puis remplacez `<SubscriptionB-id>` par l’ID de l’abonnement B. Pour homologuer myVnetA à myVNetB, copiez le script modifié, collez-le dans PowerShell, puis appuyez sur `Enter`.

    ```powershell
    Add-AzVirtualNetworkPeering `
      -Name 'myVnetAToMyVnetB' `
      -VirtualNetwork $vnetA `
      -RemoteVirtualNetworkId /subscriptions/<SubscriptionB-id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/myVnetB
    ```

11. Affichez l’état de peering de myVnetA en copiant le script suivant, en le collant dans PowerShell et en appuyant sur `Enter`.

    ```powershell
    Get-AzVirtualNetworkPeering `
      -ResourceGroupName $rgName `
      -VirtualNetworkName myVnetA `
      | Format-Table VirtualNetworkName, PeeringState
    ```

    L’état est **Connecté**. Il devient **Connecté** une fois que vous avez configuré le peering à myVnetA à partir de myVnetB.

    Les ressources Azure que vous créez dans un réseau virtuel sont désormais en mesure de communiquer entre elles via leurs adresses IP. Si vous utilisez la résolution de noms Azure par défaut pour les réseaux virtuels, les ressources dans les réseaux virtuels ne sont pas en mesure de résoudre les noms dans les réseaux virtuels. Si vous souhaitez résoudre les noms dans les réseaux virtuels d’un peering, vous devez créer votre propre serveur DNS. Apprenez à configurer la [résolution de noms à l’aide de votre propre serveur DNS](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

12. **Facultatif** : bien que la création de machines virtuelles ne soit pas abordée dans ce tutoriel, vous pouvez créer une machine virtuelle sur chaque réseau virtuel et vous connecter d’une machine virtuelle à l’autre pour valider la connectivité.
13. **Facultatif** : pour supprimer les ressources créées dans ce tutoriel, effectuez les étapes décrites dans la section [Supprimer des ressources](#delete-powershell) de cet article.

## <a name="delete-resources"></a><a name="delete"></a>Supprimer des ressources
Lorsque vous aurez terminé ce didacticiel, vous souhaiterez peut-être supprimer les ressources que vous avez créées, afin que leur utilisation ne soit pas facturée. La suppression d’un groupe de ressources supprime également toutes les ressources qu’il contient.

### <a name="azure-portal"></a><a name="delete-portal"></a>Portail Azure

1. Dans le champ de recherche du portail, entrez **myResourceGroupA**. Dans les résultats de la recherche, cliquez sur **myResourceGroupA**.
2. Dans le panneau **myResourceGroupA**, cliquez sur l’icône **Supprimer**.
3. Pour confirmer la suppression, dans le champ **Tapez le nom du groupe de ressources**, entrez **myResourceGroupA**, puis cliquez sur **Supprimer**.
4. Dans le champ **Rechercher des ressources** située en haut du portail, tapez *myVnetB*. Quand la mention **myVnetB** apparaît dans les résultats de recherche, cliquez dessus. Un panneau apparaît pour le réseau virtuel **myVnetB**.
5. Dans le panneau **myVnetB**, cliquez sur **Supprimer**.
6. Pour confirmer la suppression, cliquez sur **Oui** dans la zone **Supprimer le réseau virtuel**.

### <a name="azure-cli"></a><a name="delete-cli"></a>Interface CLI Azure

1. Connectez-vous à Azure à l’aide de l’interface CLI pour supprimer le réseau virtuel (Resource Manager) avec la commande suivante :

   ```azurecli-interactive
   az group delete --name myResourceGroupA --yes
   ```

2. Connectez-vous à Azure à l’aide de l’interface Azure classique pour supprimer le réseau virtuel (classique) avec les commandes suivantes :

   ```console
   azure config mode asm

   azure network vnet delete --vnet myVnetB --quiet
   ```

### <a name="powershell"></a><a name="delete-powershell"></a>PowerShell

1. À l’invite de commandes PowerShell, entrez la commande suivante pour supprimer le réseau virtuel (Resource Manager) :

   ```powershell
   Remove-AzResourceGroup -Name myResourceGroupA -Force
   ```

2. Pour supprimer le réseau virtuel (classique) avec PowerShell, vous devez modifier un fichier de configuration réseau existant. Pour découvrir [comment exporter, mettre à jour et importer des fichiers de configuration réseau, consultez cet article](virtual-networks-using-network-configuration-file.md). Supprimez l’élément VirtualNetworkSite suivant pour le réseau virtuel utilisé dans ce didacticiel :

   ```xml
   <VirtualNetworkSite name="myVnetB" Location="East US">
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
