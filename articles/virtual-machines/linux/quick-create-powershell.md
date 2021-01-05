---
title: 'Démarrage rapide : créer une machine virtuelle Linux avec Azure PowerShell'
description: Dans ce guide de démarrage rapide, vous allez apprendre à utiliser Azure PowerShell pour créer une machine virtuelle Linux
author: cynthn
ms.service: virtual-machines-linux
ms.topic: quickstart
ms.workload: infrastructure
ms.date: 07/31/2020
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: e3d400726bfb65b2548bc773ffb460fe1ad426a0
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "87513449"
---
# <a name="quickstart-create-a-linux-virtual-machine-in-azure-with-powershell"></a>Démarrage rapide : Créer une machine virtuelle Linux dans Azure avec PowerShell

Le module Azure PowerShell est utilisé pour créer et gérer des ressources Azure à partir de la ligne de commande PowerShell ou dans des scripts. Ce guide de démarrage rapide explique comment utiliser le module Azure PowerShell pour déployer une machine virtuelle Linux dans Azure. Ce guide de démarrage rapide utilise l’image de la Place de marché Ubuntu 18.04 LTS fournie par Canonical. Pour voir votre machine virtuelle en action, vous établirez également une connexion SSH à la machine virtuelle et installez le serveur web NGINX.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. 

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.

## <a name="create-ssh-key-pair"></a>Créer la paire de clés SSH

Utilisez [ssh-keygen](https://www.ssh.com/ssh/keygen/) pour créer une paire de clés SSH. Si vous disposez déjà d’une paire de clés SSH, vous pouvez ignorer cette étape.


```azurepowershell-interactive
ssh-keygen -m PEM -t rsa -b 4096
```

Vous êtes invité à fournir un nom de fichier pour la paire de clés, ou vous pouvez appuyer sur **Entrée** pour utiliser l’emplacement par défaut de `/home/<username>/.ssh/id_rsa`. Si vous le souhaitez, vous pouvez aussi créer un mot de passe pour les clés.

Pour plus d’informations sur la création de paires de clés SSH, consultez [Guide pratique pour utiliser des clés SSH avec Windows](ssh-from-windows.md).

Si vous créez votre paire de clés SSH en utilisant Cloud Shell, elle sera stockée dans un [compte de stockage créé automatiquement par Cloud Shell](../../cloud-shell/persisting-shell-storage.md). Ne supprimez pas ce compte de stockage ou le partage de fichiers qu’il contient tant que vous n’avez pas récupéré vos clés, faute de quoi vous perdrez votre accès à la machine virtuelle. 

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources Azure avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées :

```azurepowershell-interactive
New-AzResourceGroup -Name "myResourceGroup" -Location "EastUS"
```

## <a name="create-virtual-network-resources"></a>Créer des ressources de réseau virtuel

Créez un réseau virtuel, un sous-réseau et une adresse IP publique. Ces ressources sont utilisées pour fournir une connectivité réseau à la machine virtuelle et la connecter à Internet :

```azurepowershell-interactive
# Create a subnet configuration
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name "mySubnet" `
  -AddressPrefix 192.168.1.0/24

# Create a virtual network
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -Name "myVNET" `
  -AddressPrefix 192.168.0.0/16 `
  -Subnet $subnetConfig

# Create a public IP address and specify a DNS name
$pip = New-AzPublicIpAddress `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -AllocationMethod Static `
  -IdleTimeoutInMinutes 4 `
  -Name "mypublicdns$(Get-Random)"
```

Créez un groupe de sécurité réseau Azure et une règle de trafic. Le groupe de sécurité réseau sécurise la machine virtuelle avec des règles entrantes et sortantes. Dans l’exemple suivant, une règle de trafic entrant est créée pour le port TCP 22 qui autorise les connexions SSH. Pour autoriser le trafic web entrant, une règle de trafic entrant pour le port TCP 80 est également créée.

```azurepowershell-interactive
# Create an inbound network security group rule for port 22
$nsgRuleSSH = New-AzNetworkSecurityRuleConfig `
  -Name "myNetworkSecurityGroupRuleSSH"  `
  -Protocol "Tcp" `
  -Direction "Inbound" `
  -Priority 1000 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 22 `
  -Access "Allow"

# Create an inbound network security group rule for port 80
$nsgRuleWeb = New-AzNetworkSecurityRuleConfig `
  -Name "myNetworkSecurityGroupRuleWWW"  `
  -Protocol "Tcp" `
  -Direction "Inbound" `
  -Priority 1001 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80 `
  -Access "Allow"

# Create a network security group
$nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -Name "myNetworkSecurityGroup" `
  -SecurityRules $nsgRuleSSH,$nsgRuleWeb
```

Créez une carte d’interface réseau virtuel avec [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface). La carte réseau virtuelle connecte la machine virtuelle à un sous-réseau, un groupe de sécurité réseau et une adresse IP publique.

```azurepowershell-interactive
# Create a virtual network card and associate with public IP address and NSG
$nic = New-AzNetworkInterface `
  -Name "myNic" `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -SubnetId $vnet.Subnets[0].Id `
  -PublicIpAddressId $pip.Id `
  -NetworkSecurityGroupId $nsg.Id
```

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Pour créer une machine virtuelle dans PowerShell, vous créez une configuration qui a des paramètres tels que l’image à utiliser, la taille et les options d’authentification. La configuration est ensuite utilisée pour générer la machine virtuelle.

Définissez les informations d’identification SSH, les informations du système d’exploitation et la taille de machine virtuelle. Dans cet exemple, la clé SSH est stockée dans `~/.ssh/id_rsa.pub`. 

```azurepowershell-interactive
# Define a credential object
$securePassword = ConvertTo-SecureString ' ' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ("azureuser", $securePassword)

# Create a virtual machine configuration
$vmConfig = New-AzVMConfig `
  -VMName "myVM" `
  -VMSize "Standard_D1" | `
Set-AzVMOperatingSystem `
  -Linux `
  -ComputerName "myVM" `
  -Credential $cred `
  -DisablePasswordAuthentication | `
Set-AzVMSourceImage `
  -PublisherName "Canonical" `
  -Offer "UbuntuServer" `
  -Skus "18.04-LTS" `
  -Version "latest" | `
Add-AzVMNetworkInterface `
  -Id $nic.Id

# Configure the SSH key
$sshPublicKey = cat ~/.ssh/id_rsa.pub
Add-AzVMSshPublicKey `
  -VM $vmconfig `
  -KeyData $sshPublicKey `
  -Path "/home/azureuser/.ssh/authorized_keys"
```

Combinez maintenant les définitions de configuration précédentes à créer avec [New-AzVM](/powershell/module/az.compute/new-azvm) :

```azurepowershell-interactive
New-AzVM `
  -ResourceGroupName "myResourceGroup" `
  -Location eastus -VM $vmConfig
```

Quelques minutes sont nécessaires pour le déploiement de votre machine virtuelle. Lorsque le déploiement est terminé, passez à la section suivante.

## <a name="connect-to-the-vm"></a>Connexion à la machine virtuelle

Créer une connexion SSH à la machine virtuelle à l’aide de l’adresse IP publique. Pour voir l’adresse IP publique de la machine virtuelle, utilisez l’applet de commande [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) :

```azurepowershell-interactive
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" | Select "IpAddress"
```

Avec le même shell que celui que vous avez utilisé pour créer votre paire de clés SSH, collez la commande suivante dans le shell pour créer une session SSH. Remplacez *10.111.12.123* par l’adresse IP de votre machine virtuelle.

```bash
ssh azureuser@10.111.12.123
```

Lorsque vous y êtes invité, le nom d’utilisateur à saisir est *azureuser*. Si une phrase secrète est utilisée avec vos clés SSH, vous devez la saisir que lorsque vous y êtes invité.


## <a name="install-nginx"></a>Installer NGINX

Pour voir votre machine virtuelle en action, installez le serveur web NGINX. Dans votre session SSH, mettez à jour vos sources de package, puis installez le dernier package NGINX.

```bash
sudo apt-get -y update
sudo apt-get -y install nginx
```

Lorsque vous avez terminé, tapez `exit` pour quitter la session SSH.


## <a name="view-the-web-server-in-action"></a>Voir le serveur web en action

Utilisez le navigateur web de votre choix pour visualiser la page d’accueil NGINX par défaut. Entrez l’adresse IP publique de la machine virtuelle comme adresse web. Vous trouverez l’adresse IP publique sur la page de vue d’ensemble de la machine virtuelle ou en tant que partie de la chaîne de connexion SSH que vous avez utilisée précédemment.

![Page d’accueil par défaut de NGINX](./media/quick-create-cli/nginix-welcome-page.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, vous pouvez utiliser la cmdlet [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées :

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="next-steps"></a>Étapes suivantes

Avec ce guide de démarrage rapide, vous avez déployé une machine virtuelle simple, créé un groupe de sécurité réseau et une règle, et installé un serveur web de base. Pour en savoir plus sur les machines virtuelles Azure, suivez le didacticiel pour les machines virtuelles Linux.

> [!div class="nextstepaction"]
> [Didacticiels sur les machines virtuelles Azure Linux](./tutorial-manage-vm.md)
