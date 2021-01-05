---
title: Limiter l’exfiltration de données vers le stockage Azure - Azure PowerShell
description: Dans cet article, vous allez apprendre à limiter et restreindre l’exfiltration de données de réseau virtuel aux ressources de stockage Azure avec des stratégies de point de terminaison de service de réseau virtuel en utilisant Azure PowerShell.
services: virtual-network
documentationcenter: virtual-network
author: RDhillon
manager: narayan
editor: ''
tags: azure-resource-manager
Customer intent: I want only resources in a virtual network subnet to access an Azure PaaS resource, such as an Azure Storage account.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/03/2020
ms.author: rdhillon
ms.custom: ''
ms.openlocfilehash: 1d4fcc280ba2e34d2fa81584846441ad6fe81431
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84708193"
---
# <a name="manage-data-exfiltration-to-azure-storage-accounts-with-virtual-network-service-endpoint-policies-using-azure-powershell"></a>Gérer l’exfiltration de données vers des comptes de stockage Azure avec des stratégies de point de terminaison de service de réseau virtuel à l’aide d’Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Les stratégies de points de terminaison de service de réseau virtuel vous permettent d’appliquer un contrôle d’accès sur des comptes Stockage Azure depuis un réseau virtuel sur des points de terminaison de service. Il s’agit d’une stratégie clé pour sécuriser vos charges de travail, gérer les comptes de stockage autorisés et gérer les destinations d’exfiltration de données autorisées.
Dans cet article, vous apprendrez comment :

* Créez un réseau virtuel.
* Ajouter un sous-réseau et activer un point de terminaison de service pour le stockage Azure
* Créer deux comptes de stockage Azure et autoriser l’accès réseau à ces derniers à partir du sous-réseau précédemment créé.
* Créer une stratégie de point de terminaison de service pour autoriser l’accès à l’un des comptes de stockage uniquement.
* Déployer une machine virtuelle sur le sous-réseau.
* Vérifier l’accès au compte de stockage autorisé à partir du sous-réseau.
* Vérifier que l’accès au compte de stockage non autorisé à partir du sous-réseau est refusé.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser PowerShell en local, vous devez exécuter le module Azure PowerShell version 1.0.0 ou ultérieure pour les besoins de cet article. Exécutez `Get-Module -ListAvailable Az` pour rechercher la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.

## <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Avant de créer un réseau virtuel, vous devez créer un groupe de ressources pour le réseau virtuel et toutes les autres ressources créées dans cet article. Créez un groupe de ressources avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). L’exemple suivant permet de créer un groupe de ressources nommé *myResourceGroup* : 

```azurepowershell-interactive
New-AzResourceGroup `
  -ResourceGroupName myResourceGroup `
  -Location EastUS
```

Créez un réseau virtuel avec [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). L’exemple suivant permet de créer un réseau virtuel nommé *myVirtualNetwork* avec le préfixe d’adresse *10.0.0.0/16*.

```azurepowershell-interactive
$virtualNetwork = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork `
  -AddressPrefix 10.0.0.0/16
```

## <a name="enable-a-service-endpoint"></a>Activer un point de terminaison de service

Créez un sous-réseau dans le réseau virtuel. Dans cet exemple, un sous-réseau nommé *Private* est créé avec un point de terminaison de service *Microsoft.Storage* : 

```azurepowershell-interactive
$subnetConfigPrivate = Add-AzVirtualNetworkSubnetConfig `
  -Name Private `
  -AddressPrefix 10.0.0.0/24 `
  -VirtualNetwork $virtualNetwork `
  -ServiceEndpoint Microsoft.Storage

$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="restrict-network-access-for-the-subnet"></a>Restreindre l’accès réseau pour le sous-réseau

Créez des règles de sécurité de groupe de sécurité réseau avec [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig). La règle suivante autorise un accès sortant vers les adresses IP publiques affectées au service Stockage Azure : 

```azurepowershell-interactive
$rule1 = New-AzNetworkSecurityRuleConfig `
  -Name Allow-Storage-All `
  -Access Allow `
  -DestinationAddressPrefix Storage `
  -DestinationPortRange * `
  -Direction Outbound `
  -Priority 100 -Protocol * `
  -SourceAddressPrefix VirtualNetwork `
  -SourcePortRange *
```

La règle suivante refuse l’accès à toutes les adresses IP publiques. La règle précédente remplace cette règle, du fait de sa priorité plus élevée, ce qui permet d’accéder aux adresses IP publiques du Stockage Azure.

```azurepowershell-interactive
$rule2 = New-AzNetworkSecurityRuleConfig `
  -Name Deny-Internet-All `
  -Access Deny `
  -DestinationAddressPrefix Internet `
  -DestinationPortRange * `
  -Direction Outbound `
  -Priority 110 -Protocol * `
  -SourceAddressPrefix VirtualNetwork `
  -SourcePortRange *
```

La règle suivante autorise le trafic du protocole RDP (Remote Desktop Protocol) entrant dans le sous-réseau à partir de n’importe quel endroit. Les connexions Bureau à distance sont autorisées dans le sous-réseau, afin que vous puissiez vérifier l’accès réseau à une ressource dans une étape ultérieure.

```azurepowershell-interactive
$rule3 = New-AzNetworkSecurityRuleConfig `
  -Name Allow-RDP-All `
  -Access Allow `
  -DestinationAddressPrefix VirtualNetwork `
  -DestinationPortRange 3389 `
  -Direction Inbound `
  -Priority 120 `
  -Protocol * `
  -SourceAddressPrefix * `
  -SourcePortRange *
```

Créez un groupe de sécurité réseau avec [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup). L’exemple suivant crée un groupe de sécurité réseau nommé *myNsgPrivate*.

```azurepowershell-interactive
$nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myNsgPrivate `
  -SecurityRules $rule1,$rule2,$rule3
```

Associez le groupe de sécurité réseau au sous-réseau *Private* avec [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig), puis écrivez la configuration du sous-réseau dans le réseau virtuel. L’exemple suivant associe le groupe de sécurité réseau *myNsgPrivate* au sous-réseau *Private* :

```azurepowershell-interactive
Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VirtualNetwork `
  -Name Private `
  -AddressPrefix 10.0.0.0/24 `
  -ServiceEndpoint Microsoft.Storage `
  -NetworkSecurityGroup $nsg

$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="restrict-network-access-to-azure-storage-accounts"></a>Restreindre l’accès réseau aux comptes Stockage Azure

Les étapes nécessaires pour restreindre l’accès réseau aux ressources créées par le biais des services Azure avec activation des points de terminaison varient d’un service à l’autre. Pour connaître les étapes à suivre, consultez la documentation relative à chacun des services. La suite de cet article comprend des étapes permettant de restreindre l’accès réseau pour un compte Stockage Azure.

### <a name="create-two-storage-accounts"></a>Créer deux comptes de stockage

Créez un compte de stockage Azure avec [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount).

```azurepowershell-interactive
$storageAcctName1 = 'allowedaccount'

New-AzStorageAccount `
  -Location EastUS `
  -Name $storageAcctName1 `
  -ResourceGroupName myResourceGroup `
  -SkuName Standard_LRS `
  -Kind StorageV2
```

Une fois le compte de stockage créé, récupérez la clé du compte de stockage dans une variable avec [Get-AzStorageAccountKey](/powershell/module/az.storage/get-azstorageaccountkey) :

```azurepowershell-interactive
$storageAcctKey1 = (Get-AzStorageAccountKey -ResourceGroupName myResourceGroup -AccountName $storageAcctName1).Value[0]
```

La clé sera utilisée pour créer un partage de fichiers lors d’une prochaine étape. Entrez `$storageAcctKey` et notez la valeur, car vous devez également l’entrer dans une étape ultérieure lorsque vous mapperez le partage de fichiers à un lecteur d’une machine virtuelle.

À présent, répétez les étapes ci-dessus pour créer un deuxième compte de stockage.

```azurepowershell-interactive
$storageAcctName2 = 'notallowedaccount'

New-AzStorageAccount `
  -Location EastUS `
  -Name $storageAcctName2 `
  -ResourceGroupName myResourceGroup `
  -SkuName Standard_LRS `
  -Kind StorageV2
```

Vous devez également récupérer la clé de compte de stockage à partir de ce compte. Vous l’utiliserez ultérieurement pour créer un partage de fichiers.

```azurepowershell-interactive
$storageAcctKey2 = (Get-AzStorageAccountKey -ResourceGroupName myResourceGroup -AccountName $storageAcctName2).Value[0]
```

### <a name="create-a-file-share-in-each-of-the-storage-account"></a>Créer un partage de fichiers dans chaque compte de stockage

Créez un contexte pour votre compte de stockage et votre clé avec [New-AzStorageContext](/powershell/module/az.storage/new-AzStoragecontext). Celui-ci encapsule le nom et la clé du compte de stockage :

```azurepowershell-interactive
$storageContext1 = New-AzStorageContext $storageAcctName1 $storageAcctKey1

$storageContext2 = New-AzStorageContext $storageAcctName2 $storageAcctKey2
```

Créez un partage de fichiers avec [New-AzStorageShare](/powershell/module/az.storage/new-azstorageshare) :

```azurepowershell-interactive
$share1 = New-AzStorageShare my-file-share -Context $storageContext1

$share2 = New-AzStorageShare my-file-share -Context $storageContext2
```

### <a name="deny-all-network-access-to-a-storage-accounts"></a>Refuser tout accès réseau aux comptes de stockage

Par défaut, les comptes de stockage acceptent les connexions réseau provenant des clients de n’importe quel réseau. Pour limiter l’accès aux réseaux sélectionnés, définissez l’action par défaut sur *Refuser* avec [Update-AzStorageAccountNetworkRuleSet](/powershell/module/az.storage/update-azstorageaccountnetworkruleset). Une fois l’accès réseau refusé, le compte de stockage n’est plus accessible par aucun des réseaux.

```azurepowershell-interactive
Update-AzStorageAccountNetworkRuleSet `
  -ResourceGroupName myresourcegroup `
  -Name $storageAcctName1 `
  -DefaultAction Deny

Update-AzStorageAccountNetworkRuleSet  `
  -ResourceGroupName myresourcegroup `
  -Name $storageAcctName2 `
  -DefaultAction Deny
```

### <a name="enable-network-access-only-from-the-vnet-subnet"></a>Autoriser l’accès réseau uniquement à partir du sous-réseau de réseau virtuel

Récupérez le réseau virtuel créé avec [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork), puis récupérez l’objet de sous-réseau privé dans une variable avec [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) :

```azurepowershell-interactive
$privateSubnet = Get-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Name myVirtualNetwork `
  | Get-AzVirtualNetworkSubnetConfig -Name Private
```

Autorisez l’accès réseau au compte de stockage à partir du sous-réseau *Private* avec [Add-AzStorageAccountNetworkRule](/powershell/module/az.network/add-aznetworksecurityruleconfig).

```azurepowershell-interactive
Add-AzStorageAccountNetworkRule `
  -ResourceGroupName myresourcegroup `
  -Name $storageAcctName1 `
  -VirtualNetworkResourceId $privateSubnet.Id

Add-AzStorageAccountNetworkRule `
  -ResourceGroupName myresourcegroup `
  -Name $storageAcctName2 `
  -VirtualNetworkResourceId $privateSubnet.Id
```

## <a name="apply-policy-to-allow-access-to-valid-storage-account"></a>Appliquer la stratégie pour autoriser l’accès au compte de stockage valide

Pour vous assurer que les utilisateurs du réseau virtuel peuvent accéder uniquement aux comptes de stockage Azure sécurisés et autorisés, vous pouvez créer une stratégie de point de terminaison de service avec la liste des comptes de stockage autorisés dans la définition. Cette stratégie est ensuite appliquée au sous-réseau de réseau virtuel connecté au stockage par le biais de points de terminaison de service.

### <a name="create-a-service-endpoint-policy"></a>Créer une stratégie de point de terminaison de service

Dans le cadre de cette section, vous allez créer la définition de stratégie avec la liste des ressources dont l’accès est autorisé sur le point de terminaison de service.

Récupérer l’ID de ressource pour le premier compte de stockage (autorisé) 

```azurepowershell-interactive
$resourceId = (Get-AzStorageAccount -ResourceGroupName myresourcegroup -Name $storageAcctName1).id
```

Créer la définition de stratégie pour autoriser la ressource ci-dessus

```azurepowershell-interactive
$policyDefinition = New-AzServiceEndpointPolicyDefinition -Name mypolicydefinition `
  -Description "Service Endpoint Policy Definition" `
  -Service "Microsoft.Storage" `
  -ServiceResource $resourceId
```

Créer la stratégie de point de terminaison de service à l’aide de la définition de stratégie créée ci-dessus

```azurepowershell-interactive
$sepolicy = New-AzServiceEndpointPolicy -ResourceGroupName myresourcegroup `
  -Name mysepolicy -Location EastUS
  -ServiceEndpointPolicyDefinition $policyDefinition
```

### <a name="associate-the-service-endpoint-policy-to-the-virtual-network-subnet"></a>Associer la stratégie de point de terminaison de service au sous-réseau de réseau virtuel

Après avoir créé la stratégie de point de terminaison de service, vous devez l’associer au sous-réseau cible avec la configuration de point de terminaison de service pour le stockage Azure.

```azurepowershell-interactive
Set-AzVirtualNetworkSubnetConfig -VirtualNetwork $VirtualNetwork `
  -Name Private `
  -AddressPrefix 10.0.0.0/24 `
  -NetworkSecurityGroup $nsg `
  -ServiceEndpoint Microsoft.Storage `
  -ServiceEndpointPolicy $sepolicy

$virtualNetwork | Set-AzVirtualNetwork
```
## <a name="validate-access-restriction-to-azure-storage-accounts"></a>Vérifier la restriction d’accès aux comptes de stockage Azure

### <a name="deploy-the-virtual-machine"></a>Déployer la machine virtuelle

Pour tester l’accès réseau à un compte de stockage, déployez une machine virtuelle sur le sous-réseau.

Créez une machine virtuelle dans le sous-réseau *Privé* avec [New-AzVM](/powershell/module/az.compute/new-azvm). Lors de l’exécution de la commande qui suit, vous êtes invité à saisir vos informations d’identification. Les valeurs que vous saisissez sont configurées comme le nom d’utilisateur et le mot de passe pour la machine virtuelle. L’option `-AsJob` crée la machine virtuelle en arrière-plan. Vous pouvez donc passer à l’étape suivante.

```azurepowershell-interactive
New-AzVm -ResourceGroupName myresourcegroup `
  -Location "East US" `
  -VirtualNetworkName myVirtualNetwork `
  -SubnetName Private `
  -Name "myVMPrivate" -AsJob
```

Une sortie similaire à la sortie suivante est renvoyée :

```powershell
Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Long Running... AzureLongRun... Running       True            localhost            New-AzVM
```

### <a name="confirm-access-to-the-allowed-storage-account"></a>Vérifier l’accès au compte de stockage *autorisé*

Utilisez [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) pour retourner l’adresse IP publique d’une machine virtuelle. L’exemple suivant retourne l’adresse IP publique de la machine virtuelle *myVmPrivate* :

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVmPrivate `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

Remplacez `<publicIpAddress>` dans la commande suivante par l’adresse IP publique adresse retournée à partir de la commande précédente, puis entrez la commande suivante :

```powershell
mstsc /v:<publicIpAddress>
```

Un fichier .rdp (Remote Desktop Protocol) est créé et téléchargé sur votre ordinateur. Ouvrez le fichier .rdp téléchargé. Si vous y êtes invité, sélectionnez **Connexion**. Entrez le nom d’utilisateur et le mot de passe spécifiés lors de la création de la machine virtuelle. Vous devrez peut-être sélectionner **Plus de choix**, puis **Utiliser un autre compte**, pour spécifier les informations d’identification que vous avez entrées lorsque vous avez créé la machine virtuelle. Sélectionnez **OK**. Un avertissement de certificat peut s’afficher pendant le processus de connexion. Si vous recevez l’avertissement, sélectionnez **Oui** ou **Continuer** pour poursuivre le processus de connexion.

Sur la machine virtuelle *myVmPrivate*, mappez le partage de fichiers Azure du compte de stockage autorisé au lecteur Z à l’aide de PowerShell. 

```powershell
$acctKey = ConvertTo-SecureString -String $storageAcctKey1 -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList ("Azure\allowedaccount"), $acctKey
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\allowedaccount.file.core.windows.net\my-file-share" -Credential $credential
```

PowerShell retourne un résultat semblable à l’exemple suivant :

```powershell
Name           Used (GB)     Free (GB) Provider      Root
----           ---------     --------- --------      ----
Z                                      FileSystem    \\allowedaccount.file.core.windows.net\my-f...
```

Le partage de fichiers Azure est correctement mappé au lecteur Z.

Fermez les sessions Bureau à distance sur la machine virtuelle *myVmPrivate*.

### <a name="confirm-access-is-denied-to-non-allowed-storage-account"></a>Vérifier que l’accès au compte de stockage *non autorisé* est refusé

Toujours sur la machine virtuelle *myVmPrivate*, essayez de mapper le partage de fichiers Azure au lecteur X. 

```powershell
$acctKey = ConvertTo-SecureString -String $storageAcctKey1 -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList "Azure\notallowedaccount", $acctKey
New-PSDrive -Name X -PSProvider FileSystem -Root "\\notallowedaccount.file.core.windows.net\my-file-share" -Credential $credential
```

L’accès au partage est refusé et vous recevez une erreur `New-PSDrive : Access is denied`. L’accès est refusé, car le compte de stockage *notallowedaccount* ne figure pas dans la liste des ressources autorisées de la stratégie de point de terminaison de service. 

Fermez les sessions Bureau à distance sur la machine virtuelle *myVmPublic*.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin d’un groupe de ressources, vous pouvez utiliser [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour le supprimer ainsi que toutes les ressources qu’il contient :

```azurepowershell-interactive 
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Étapes suivantes

Dans le cadre de cet article, vous avez appliqué une stratégie de point de terminaison de service sur un point de terminaison de service de réseau virtuel Azure au stockage Azure. Vous avez créé des comptes de stockage Azure et vous avez limité l’accès réseau à certains comptes de stockage (et refusé l’accès à d’autres) à partir d’un sous-réseau de réseau virtuel. Pour en savoir plus sur les stratégies de point de terminaison de service, consultez [Vue d’ensemble des stratégies de points de terminaison de service](virtual-network-service-endpoint-policies-overview.md).
