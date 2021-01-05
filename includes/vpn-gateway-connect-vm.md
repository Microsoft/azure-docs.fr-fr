---
title: Fichier include
description: Fichier include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/23/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 6986651d2b48d82382aa26a0a3947104c7e09981
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96028235"
---
Vous pouvez vous connecter à une machine virtuelle déployée sur votre réseau virtuel en créant une connexion Bureau à distance à votre machine virtuelle. La meilleure méthode pour vérifier initialement que vous pouvez vous connecter à votre machine virtuelle consiste à vous connecter à l’aide de son adresse IP privée, plutôt qu’avec le nom d’ordinateur. Vous testez ainsi si vous pouvez vous connecter, que la résolution de nom soit configurée correctement ou non.

1. Recherchez l’adresse IP privée. L’adresse IP privée d’une machine virtuelle peut être trouvée en étudiant les propriétés de la machine virtuelle dans le portail Azure ou à l’aide de PowerShell.

   * Portail Azure : recherchez votre machine virtuelle dans le portail Azure. Affichez les propriétés de la machine virtuelle. L’adresse IP privée est répertoriée.

   * PowerShell : utilisez l’exemple pour afficher la liste des machines virtuelles et adresses IP privées de vos groupes de ressources. Vous n’avez pas besoin de modifier cet exemple pour pouvoir l’utiliser.

     ```azurepowershell-interactive
     $VMs = Get-AzVM
     $Nics = Get-AzNetworkInterface | Where VirtualMachine -ne $null

     foreach($Nic in $Nics)
     {
      $VM = $VMs | Where-Object -Property Id -eq $Nic.VirtualMachine.Id
      $Prv = $Nic.IpConfigurations | Select-Object -ExpandProperty PrivateIpAddress
      $Alloc = $Nic.IpConfigurations | Select-Object -ExpandProperty PrivateIpAllocationMethod
      Write-Output "$($VM.Name): $Prv,$Alloc"
     }
     ```

1. Vérifiez que vous êtes connecté à votre réseau virtuel à l’aide de la connexion VPN point à site.
1. Ouvrez une **connexion Bureau à distance** en saisissant « RDP » ou « Connexion Bureau à distance » dans la zone de recherche de la barre des tâches, puis sélectionnez la connexion Bureau à distance. Vous pouvez également ouvrir une connexion Bureau à distance à l’aide de la commande « mstsc » dans PowerShell. 
1. Dans Connexion Bureau à distance, entrez l’adresse IP privée de la machine virtuelle. Vous pouvez cliquer sur « Afficher les Options » pour définir des paramètres supplémentaires, puis connectez-vous.

**Résoudre les problèmes d’une connexion**

Si vous rencontrez des problèmes de connexion à une machine virtuelle sur votre connexion VPN, vérifiez les points suivants :

* Vérifiez que votre connexion VPN aboutit.

* Vérifiez que vous vous connectez à l’adresse IP privée de la machine virtuelle.

* Si vous pouvez vous connecter à la machine virtuelle à l’aide de l’adresse IP privée, mais pas à l’aide du nom d’ordinateur, vérifiez que vous avez correctement configuré DNS. Pour plus d’informations sur le fonctionnement de la résolution de noms pour les machines virtuelles, consultez [Résolution de noms pour les machines virtuelles](../articles/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).

* Pour plus d’informations sur les connexions RDP, consultez [Résoudre des problèmes de connexion Bureau à distance à une machine virtuelle](../articles/virtual-machines/troubleshooting/troubleshoot-rdp-connection.md).