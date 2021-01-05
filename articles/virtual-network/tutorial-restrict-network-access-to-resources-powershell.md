---
title: Limiter l’accès réseau aux ressources PaaS - Azure PowerShell
description: Dans cet article, vous allez apprendre à limiter et restreindre l’accès réseau aux ressources Azure, telles que le stockage Azure et Azure SQL Database, avec les points de terminaison de service de réseau virtuel à l’aide d’Azure PowerShell.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
Customer intent: I want only resources in a virtual network subnet to access an Azure PaaS resource, such as an Azure Storage account.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/14/2018
ms.author: kumud
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 92a5ce539cdd61feb57722756ec8722916f346de
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89072381"
---
# <a name="restrict-network-access-to-paas-resources-with-virtual-network-service-endpoints-using-powershell"></a>Restreindre l’accès réseau aux ressources PaaS avec des points de terminaison de service de réseau virtuel en utilisant Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Les points de terminaison de service de réseau virtuel permettent de restreindre l’accès réseau à certaines ressources du service Azure en n’autorisant leur accès qu’à partir d’un sous-réseau du réseau virtuel. Vous pouvez également supprimer l’accès Internet aux ressources. Les points de terminaison de service fournissent une connexion directe entre votre réseau virtuel et les services Azure pris en charge, ce qui vous permet d’utiliser l’espace d’adressage privé de votre réseau virtuel pour accéder aux services Azure. Le trafic destiné aux ressources Azure via les points de terminaison de service reste toujours sur le serveur principal de Microsoft Azure. Dans cet article, vous apprendrez comment :

* Créer un réseau virtuel avec un sous-réseau
* Ajouter un sous-réseau et activer un point de terminaison de service
* Créer une ressource Azure et autoriser l’accès réseau à cette ressource uniquement à partir d’un sous-réseau
* Déployer une machine virtuelle sur chaque sous-réseau
* Vérifier l’accès à une ressource à partir d’un sous-réseau
* Vérifier que l’accès à une ressource est refusé à partir d’un sous-réseau et d’Internet

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser PowerShell en local, vous devez exécuter le module Azure PowerShell version 1.0.0 ou ultérieure pour les besoins de cet article. Exécutez `Get-Module -ListAvailable Az` pour rechercher la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.

## <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Avant de créer un réseau virtuel, vous devez créer un groupe de ressources pour le réseau virtuel et toutes les autres ressources créées dans cet article. Créez un groupe de ressources avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). L’exemple suivant permet de créer un groupe de ressources nommé *myResourceGroup* : 

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

Créez un réseau virtuel avec [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). L’exemple suivant permet de créer un réseau virtuel nommé *myVirtualNetwork* avec le préfixe d’adresse *10.0.0.0/16*.

```azurepowershell-interactive
$virtualNetwork = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork `
  -AddressPrefix 10.0.0.0/16
```

Créez une configuration de sous-réseau à l’aide de la commande [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig). L’exemple suivant crée une configuration de sous-réseau pour un sous-réseau nommé *Public* :

```azurepowershell-interactive
$subnetConfigPublic = Add-AzVirtualNetworkSubnetConfig `
  -Name Public `
  -AddressPrefix 10.0.0.0/24 `
  -VirtualNetwork $virtualNetwork
```

Créez le sous-réseau dans le réseau virtuel en écrivant la configuration du sous-réseau dans le réseau virtuel à l’aide de [Set-AzVirtualNetwork](/powershell/module/az.network/Set-azVirtualNetwork) :

```azurepowershell-interactive
$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="enable-a-service-endpoint"></a>Activer un point de terminaison de service

Vous ne pouvez activer des points de terminaison de service que pour les services qui prennent en charge les points de terminaison de service. Affichez les services avec points de terminaison qui se trouvent à un emplacement Azure donné avec [Get-AzVirtualNetworkAvailableEndpointService](/powershell/module/az.network/get-azvirtualnetworkavailableendpointservice). L’exemple suivant retourne la liste des services de la région *eastus* pour lesquels les points de terminaison de service ont été activés. La liste des services retournés augmente avec chaque nouvelle activation des points de terminaison de service.

```azurepowershell-interactive
Get-AzVirtualNetworkAvailableEndpointService -Location eastus | Select Name
```

Créez un autre sous-réseau dans le réseau virtuel. Dans cet exemple, un sous-réseau nommé *Private* est créé avec un point de terminaison de service *Microsoft.Storage* : 

```azurepowershell-interactive
$subnetConfigPrivate = Add-AzVirtualNetworkSubnetConfig `
  -Name Private `
  -AddressPrefix 10.0.1.0/24 `
  -VirtualNetwork $virtualNetwork `
  -ServiceEndpoint Microsoft.Storage

$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="restrict-network-access-for-a-subnet"></a>Restreindre l’accès réseau d’un sous-réseau

Créez des règles de sécurité de groupe de sécurité réseau avec [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig). La règle suivante autorise un accès sortant vers les adresses IP publiques affectées au service Stockage Azure : 

```azurepowershell-interactive
$rule1 = New-AzNetworkSecurityRuleConfig `
  -Name Allow-Storage-All `
  -Access Allow `
  -DestinationAddressPrefix Storage `
  -DestinationPortRange * `
  -Direction Outbound `
  -Priority 100 `
  -Protocol * `
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
  -Priority 110 `
  -Protocol * `
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
  -AddressPrefix 10.0.1.0/24 `
  -ServiceEndpoint Microsoft.Storage `
  -NetworkSecurityGroup $nsg

$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="restrict-network-access-to-a-resource"></a>Restreindre l’accès réseau à une ressource

Les étapes nécessaires pour restreindre l’accès réseau aux ressources créées par le biais des services Azure avec activation des points de terminaison varient d’un service à l’autre. Pour connaître les étapes à suivre, consultez la documentation relative à chacun des services. La suite de cet article comprend des étapes permettant de restreindre l’accès réseau pour un compte Stockage Azure.

### <a name="create-a-storage-account"></a>Créez un compte de stockage.

Créez un compte de stockage Azure avec [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount). Remplacez `<replace-with-your-unique-storage-account-name>` par un nom qui n’existe dans aucun autre emplacement Azure. Le nom doit comprendre entre 3 et 24 caractères, correspondant à des chiffres et à des lettres en minuscules.

```azurepowershell-interactive
$storageAcctName = '<replace-with-your-unique-storage-account-name>'

New-AzStorageAccount `
  -Location EastUS `
  -Name $storageAcctName `
  -ResourceGroupName myResourceGroup `
  -SkuName Standard_LRS `
  -Kind StorageV2
```

Une fois le compte de stockage créé, récupérez la clé du compte de stockage dans une variable avec [Get-AzStorageAccountKey](/powershell/module/az.storage/get-azstorageaccountkey) :

```azurepowershell-interactive
$storageAcctKey = (Get-AzStorageAccountKey `
  -ResourceGroupName myResourceGroup `
  -AccountName $storageAcctName).Value[0]
```

La clé sera utilisée pour créer un partage de fichiers lors d’une prochaine étape. Entrez `$storageAcctKey` et notez la valeur, car vous devez également l’entrer dans une étape ultérieure lorsque vous mapperez le partage de fichiers à un lecteur d’une machine virtuelle.

### <a name="create-a-file-share-in-the-storage-account"></a>Créer un partage de fichiers dans le compte de stockage

Créez un contexte pour votre compte de stockage et votre clé avec [New-AzStorageContext](/powershell/module/az.storage/new-AzStoragecontext). Celui-ci encapsule le nom et la clé du compte de stockage :

```azurepowershell-interactive
$storageContext = New-AzStorageContext $storageAcctName $storageAcctKey
```

Créez un partage de fichiers avec [New-AzStorageShare](/powershell/module/az.storage/new-azstorageshare) :

$share = New-AzStorageShare my-file-share -Context $storageContext

### <a name="deny-all-network-access-to-a-storage-account"></a>Refuser tout accès réseau au compte de stockage

Par défaut, les comptes de stockage acceptent les connexions réseau provenant des clients de n’importe quel réseau. Pour limiter l’accès aux réseaux sélectionnés, définissez l’action par défaut sur *Refuser* avec [Update-AzStorageAccountNetworkRuleSet](/powershell/module/az.storage/update-azstorageaccountnetworkruleset). Une fois l’accès réseau refusé, le compte de stockage n’est plus accessible par aucun des réseaux.

```azurepowershell-interactive
Update-AzStorageAccountNetworkRuleSet  `
  -ResourceGroupName "myresourcegroup" `
  -Name $storageAcctName `
  -DefaultAction Deny
```

### <a name="enable-network-access-from-a-subnet"></a>Activer l’accès réseau à partir d’un sous-réseau

Récupérez le réseau virtuel créé avec [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork), puis récupérez l’objet de sous-réseau privé dans une variable avec [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) :

```azurepowershell-interactive
$privateSubnet = Get-AzVirtualNetwork `
  -ResourceGroupName "myResourceGroup" `
  -Name "myVirtualNetwork" `
  | Get-AzVirtualNetworkSubnetConfig `
  -Name "Private"
```

Autorisez l’accès réseau au compte de stockage à partir du sous-réseau *Private* avec [Add-AzStorageAccountNetworkRule](/powershell/module/az.network/add-aznetworksecurityruleconfig).

```azurepowershell-interactive
Add-AzStorageAccountNetworkRule `
  -ResourceGroupName "myresourcegroup" `
  -Name $storageAcctName `
  -VirtualNetworkResourceId $privateSubnet.Id
```

## <a name="create-virtual-machines"></a>Créer des machines virtuelles

Pour tester l’accès réseau à un compte de stockage, déployez une machine virtuelle sur chaque sous-réseau.

### <a name="create-the-first-virtual-machine"></a>Créer la première machine virtuelle

Créez une machine virtuelle dans le sous-réseau *Public* avec [New-AzVM](/powershell/module/az.compute/new-azvm). Lors de l’exécution de la commande qui suit, vous êtes invité à saisir vos informations d’identification. Les valeurs que vous saisissez sont configurées comme le nom d’utilisateur et le mot de passe pour la machine virtuelle. L’option `-AsJob` crée la machine virtuelle en arrière-plan. Vous pouvez donc passer à l’étape suivante.

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Location "East US" `
    -VirtualNetworkName "myVirtualNetwork" `
    -SubnetName "Public" `
    -Name "myVmPublic" `
    -AsJob
```

Une sortie similaire à la sortie suivante est renvoyée :

```powershell
Id     Name            PSJobTypeName   State         HasMoreData     Location             Command                  
--     ----            -------------   -----         -----------     --------             -------                  
1      Long Running... AzureLongRun... Running       True            localhost            New-AzVM     
```

### <a name="create-the-second-virtual-machine"></a>Créer la deuxième machine virtuelle

Créez une machine virtuelle dans le sous-réseau *Private* :

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Location "East US" `
    -VirtualNetworkName "myVirtualNetwork" `
    -SubnetName "Private" `
    -Name "myVmPrivate"
```

La création de la machine virtuelle par Azure ne nécessite que quelques minutes. Ne passez pas à l’étape suivante tant qu’Azure n’a pas terminé la création de la machine virtuelle et retourné de sortie à PowerShell.

## <a name="confirm-access-to-storage-account"></a>Vérifier l’accès au compte de stockage

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

Sur la machine virtuelle *myVmPrivate*, mappez le partage de fichiers Azure au lecteur Z à l’aide de PowerShell. Avant d’exécuter les commandes qui suivent, remplacez `<storage-account-key>` et `<storage-account-name>` par les valeurs que vous avez fournies ou récupérées dans [Créer un compte de stockage](#create-a-storage-account).

```powershell
$acctKey = ConvertTo-SecureString -String "<storage-account-key>" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList "Azure\<storage-account-name>", $acctKey
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\<storage-account-name>.file.core.windows.net\my-file-share" -Credential $credential
```

PowerShell retourne un résultat semblable à l’exemple suivant :

```powershell
Name           Used (GB)     Free (GB) Provider      Root
----           ---------     --------- --------      ----
Z                                      FileSystem    \\vnt.file.core.windows.net\my-f...
```

Le partage de fichiers Azure est correctement mappé au lecteur Z.

Vérifiez que la machine virtuelle ne dispose d’aucune connexion sortante à d’autres adresses IP publiques :

```powershell
ping bing.com
```

Vous ne recevez aucune réponse, car le groupe de sécurité réseau associé au sous-réseau *Private* n’autorise pas l’accès sortant aux adresses IP publiques autres que les adresses affectées au service Stockage Azure.

Fermez les sessions Bureau à distance sur la machine virtuelle *myVmPrivate*.

## <a name="confirm-access-is-denied-to-storage-account"></a>Vérifier que l’accès au compte de stockage est refusé

Obtenir l’adresse IP publique de la machine virtuelle *myVmPublic* :

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVmPublic `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

Remplacez `<publicIpAddress>` dans la commande suivante par l’adresse IP publique adresse retournée à partir de la commande précédente, puis entrez la commande suivante : 

```powershell
mstsc /v:<publicIpAddress>
```

Sur la machine virtuelle *myVmPublic*, essayez de mapper le partage de fichiers Azure au lecteur Z. Avant d’exécuter les commandes qui suivent, remplacez `<storage-account-key>` et `<storage-account-name>` par les valeurs que vous avez fournies ou récupérées dans [Créer un compte de stockage](#create-a-storage-account).

```powershell
$acctKey = ConvertTo-SecureString -String "<storage-account-key>" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList "Azure\<storage-account-name>", $acctKey
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\<storage-account-name>.file.core.windows.net\my-file-share" -Credential $credential
```

L’accès au partage est refusé et vous recevez une erreur `New-PSDrive : Access is denied`. L’accès est refusé car la machine virtuelle *myVmPublic* est déployée sur le sous-réseau *Public*. Le sous-réseau *Public* ne dispose pas d’un point de terminaison de service activé pour Stockage Azure, et le compte de stockage autorise uniquement l’accès réseau à partir du sous-réseau *Private* et non à partir du sous-réseau *Public*.

Fermez les sessions Bureau à distance sur la machine virtuelle *myVmPublic*.

Sur votre ordinateur, essayez d’afficher les partages de fichier du compte de stockage avec la commande suivante :

```powershell-interactive
Get-AzStorageFile `
  -ShareName my-file-share `
  -Context $storageContext
```

L’accès est refusé, et vous recevez une erreur indiquant *Get-AzStorageFile : Le serveur distant a retourné une erreur : (403) Interdit. Code d’état HTTP : 403 - Message d’erreur HTTP : Cette requête n’est pas autorisée à effectuer cette opération*, car votre ordinateur ne se trouve pas dans le sous-réseau *Private* du réseau virtuel *MyVirtualNetwork*.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin d’un groupe de ressources, vous pouvez utiliser [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour le supprimer ainsi que toutes les ressources qu’il contient :

```azurepowershell-interactive 
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez activé un point de terminaison de service pour un sous-réseau de réseau virtuel. Vous avez vu que les points de terminaison de service peuvent être activés pour les ressources déployées à l’aide de différents services Azure. Vous avez créé un compte Stockage Azure et un accès réseau à ce compte de stockage, limité aux ressources du sous-réseau du réseau virtuel. Pour en savoir plus sur les points de terminaison de service, consultez [Points de terminaison de service de réseau virtuel](virtual-network-service-endpoints-overview.md) et [Ajouter, modifier ou supprimer un sous-réseau de réseau virtuel](virtual-network-manage-subnet.md).

Si votre compte comporte plusieurs réseaux virtuels, vous pouvez relier deux réseaux virtuels pour que les ressources de chaque réseau virtuel puissent communiquer entre elles. Pour connaître la marche à suivre, consultez [Connecter des réseaux virtuels](tutorial-connect-virtual-networks-powershell.md).
