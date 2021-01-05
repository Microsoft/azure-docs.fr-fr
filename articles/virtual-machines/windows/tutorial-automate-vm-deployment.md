---
title: 'Tutoriel : Installer des applications sur une machine virtuelle Windows dans Azure'
description: Avec ce didacticiel, vous allez apprendre à utiliser l’extension de script personnalisé pour exécuter des scripts et déployer des applications sur des machines virtuelles Windows dans Azure
author: cynthn
ms.service: virtual-machines-windows
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 11/29/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 14d0190a97c22a805065ceaf41dcd655b9e8182b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87065293"
---
# <a name="tutorial---deploy-applications-to-a-windows-virtual-machine-in-azure-with-the-custom-script-extension"></a>Didacticiel : déployer des applications sur une machine virtuelle Windows dans Azure avec l’extension de script personnalisé

Pour configurer des machines virtuelles de manière rapide et cohérente, vous pouvez utiliser l’[extension de script personnalisé pour Windows](../extensions/custom-script-windows.md). Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Utiliser une extension de script personnalisé pour installer IIS
> * Créer une machine virtuelle qui utilise l’extension de script personnalisé
> * Afficher un site IIS en cours d’exécution après l’application de l’extension

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. 

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.

## <a name="custom-script-extension-overview"></a>Vue d’ensemble de l’extension de script personnalisé
L’extension de script personnalisé télécharge et exécute des scripts sur des machines virtuelles Azure. Cette extension est utile pour la configuration post-déploiement, l’installation de logiciels ou toute autre tâche de configuration ou de gestion. Des scripts peuvent être téléchargés à partir de Stockage Azure ou de GitHub, ou fournis dans le portail Azure lors de l’exécution de l’extension.

L’extension de script personnalisé s’intègre aux modèles Azure Resource Manager et peut être exécutée à l’aide de l’interface de ligne de commande Azure, de PowerShell, du portail Azure ou de l’API REST de machine virtuelle Azure.

Vous pouvez utiliser l’extension de script personnalisé avec les machines virtuelles Linux et Windows.


## <a name="create-virtual-machine"></a>Créer une machine virtuelle
Définissez le nom d’utilisateur administrateur et un mot de passe pour la machine virtuelle avec [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1) :

```azurepowershell-interactive
$cred = Get-Credential
```

Vous pouvez maintenant créer la machine virtuelle avec [New-AzVM](/powershell/module/az.compute/new-azvm). L’exemple suivant permet de créer une machine virtuelle nommée *myVM* dans l’emplacement *EastUS*. S’ils n’existent pas, le groupe de ressources *myResourceGroupAutomate* et les ressources réseau prises en charge sont créés. Pour autoriser le trafic web, l’applet de commande ouvre également le port *80*.

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroupAutomate" `
    -Name "myVM" `
    -Location "East US" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -OpenPorts 80 `
    -Credential $cred
```

Quelques minutes sont nécessaires à la création des ressources et de la machine virtuelle.


## <a name="automate-iis-install"></a>Automatiser l’installation d’IIS
Utilisez [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension) pour installer l’extension de script personnalisé. L’extension exécute `powershell Add-WindowsFeature Web-Server` pour installer le serveur web IIS, puis met à jour la page *Default.htm* pour qu’elle affiche le nom d’hôte de la machine virtuelle :

```azurepowershell-interactive
Set-AzVMExtension -ResourceGroupName "myResourceGroupAutomate" `
    -ExtensionName "IIS" `
    -VMName "myVM" `
    -Location "EastUS" `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.8 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}'
```


## <a name="test-web-site"></a>Tester le site web
Obtenez l’adresse IP publique de votre équilibreur de charge avec [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress). L’exemple suivant obtient l’adresse IP pour l’adresse *myPublicIPAddress* créée précédemment :

```azurepowershell-interactive
Get-AzPublicIPAddress `
    -ResourceGroupName "myResourceGroupAutomate" `
    -Name "myPublicIPAddress" | select IpAddress
```

Vous pouvez alors entrer l’adresse IP publique dans un navigateur web. Le site web s’affiche, avec notamment le nom d’hôte de la machine virtuelle sur laquelle l’équilibreur de charge a distribué le trafic comme dans l’exemple suivant :

![Site web IIS en cours d’exécution](./media/tutorial-automate-vm-deployment/running-iis-website.png)


## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez automatisé l’installation d’IIS sur une machine virtuelle. Vous avez appris à :

> [!div class="checklist"]
> * Utiliser une extension de script personnalisé pour installer IIS
> * Créer une machine virtuelle qui utilise l’extension de script personnalisé
> * Afficher un site IIS en cours d’exécution après l’application de l’extension

Passez au didacticiel suivant pour découvrir comment créer des images de machine virtuelle personnalisées.

> [!div class="nextstepaction"]
> [Créer des images de machine virtuelle personnalisées](./tutorial-custom-images.md)
