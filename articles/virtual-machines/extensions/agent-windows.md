---
title: Vue d’ensemble d’agent de machine virtuelle Azure
description: Vue d’ensemble d’agent de machine virtuelle Azure
services: virtual-machines-windows
ms.subservice: extensions
author: mimckitt
ms.service: virtual-machines-windows
ms.topic: article
ms.date: 07/20/2019
ms.author: mimckitt
ms.openlocfilehash: 3724b8a2afb89594c73f7dae782658ec8978963a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016469"
---
# <a name="azure-virtual-machine-agent-overview"></a>Vue d’ensemble d’agent de machine virtuelle Azure
L’agent de machine virtuelle Microsoft Azure est un processus léger et sécurisé qui gère l’interaction des machines virtuelles avec le contrôleur de structure Azure. L’agent de machine virtuelle a un rôle essentiel dans l’activation et l’exécution des extensions de machine virtuelle. Les extensions de machine virtuelle permettent la configuration post-déploiement de machines virtuelles, par exemple l’installation et la configuration de logiciels. Les extensions de machine virtuelle permettent également d’utiliser des fonctionnalités de récupération, telles que la réinitialisation du mot de passe d’administration d’une machine virtuelle. Sans l’agent de machine virtuelle Azure, vous ne pouvez pas exécuter d’extensions de machine virtuelle.

Cet article détaille l’installation et la détection de l’agent de machine virtuelle Azure.

## <a name="install-the-vm-agent"></a>Installer l'agent de machine virtuelle

### <a name="azure-marketplace-image"></a>Image Place de marché Azure

L’agent de machine virtuelle Azure est installé par défaut sur les machines virtuelles Windows qui sont déployées à partir d’une image Place de marché Azure. Lorsque vous déployez une image Place de marché Azure à partir du portail, de PowerShell, de l’interface de ligne de commande ou d’un modèle Azure Resource Manager, vous installez également l’agent de machine virtuelle Azure.

Le Package de l’agent invité de Windows est composé de deux parties :

- L’agent de provisionnement
- L’agent invité de Windows

Pour démarrer une machine virtuelle, vous devez installer l’agent de provisionnement sur la machine virtuelle, contrairement à l’agent invité qu’il n’est pas nécessaire d’installer. Au moment du déploiement de la machine virtuelle, vous pouvez choisir de ne pas installer l’agent invité de Windows. L’exemple suivant montre comment sélectionner l’option *provisionVmAgent* à l’aide d’un modèle Azure Resource Manager :

```json
"resources": [{
"name": "[parameters('virtualMachineName')]",
"type": "Microsoft.Compute/virtualMachines",
"apiVersion": "2016-04-30-preview",
"location": "[parameters('location')]",
"dependsOn": ["[concat('Microsoft.Network/networkInterfaces/', parameters('networkInterfaceName'))]"],
"properties": {
    "osProfile": {
    "computerName": "[parameters('virtualMachineName')]",
    "adminUsername": "[parameters('adminUsername')]",
    "adminPassword": "[parameters('adminPassword')]",
    "windowsConfiguration": {
        "provisionVmAgent": "false"
}
```

Si les agents ne sont pas installés, vous ne pouvez pas utiliser certains services Azure, tels que la Sauvegarde Azure ou Azure Security. Ces services nécessitent l’installation d’une extension. Si vous avez déployé une machine virtuelle sans l’agent invité de Windows, vous pouvez installer la dernière version de cet agent ultérieurement.

### <a name="manual-installation"></a>Installation manuelle
L’agent de machine virtuelle Windows peut être installé manuellement avec un package Windows Installer. L’installation manuelle peut s’avérer nécessaire lorsque vous créez une image de machine virtuelle personnalisée qui est déployée sur Azure. Pour installer manuellement l’agent de machine virtuelle Windows, [téléchargez le programme d’installation de l’agent de machine virtuelle](https://go.microsoft.com/fwlink/?LinkID=394789). L’agent de machine virtuelle est pris en charge sur Windows Server 2008 (64 bits) et ultérieur.

> [!NOTE]
> Il est important de mettre à jour l’option AllowExtensionOperations après l’installation manuelle de VMAgent sur une machine virtuelle déployée à partir d’une image sans que ProvisionVMAgent soit activé.

```powershell
$vm.OSProfile.AllowExtensionOperations = $true
$vm | Update-AzVM
```

### <a name="prerequisites"></a>Prérequis

- L’agent de machine virtuelle Windows a besoin d’au moins Windows Server 2008 SP2 (64 bits) pour fonctionner avec .NET Framework 4.0. Voir [Prise en charge de version minimale pour les agents de machine virtuelle dans Azure](https://support.microsoft.com/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support).

- Vérifiez que votre machine virtuelle a accès à l’adresse IP 168.63.129.16. Pour plus d’informations, consultez [Qu’est-ce que l’adresse IP 168.63.129.16 ?](../../virtual-network/what-is-ip-address-168-63-129-16.md)

- Vérifiez que le protocole DHCP est activé à l’intérieur de la machine virtuelle invitée. Il est nécessaire pour obtenir l’adresse de l’hôte ou de l’infrastructure à partir de DHCP pour que l’agent de machine virtuelle IaaS et les extensions fonctionnent. Si vous avez besoin d’une adresse IP privée statique, vous devez la configurer via le portail Azure ou PowerShell et vérifier que l’option DHCP dans la machine virtuelle est activée. [Apprenez-en davantage](../../virtual-network/virtual-networks-static-private-ip-arm-ps.md#change-the-allocation-method-for-a-private-ip-address-assigned-to-a-network-interface) sur la configuration d’une adresse IP statique avec PowerShell.


## <a name="detect-the-vm-agent"></a>Détecter l’agent de machine virtuelle

### <a name="powershell"></a>PowerShell

Avec le module PowerShell pour Azure Resource Manager, vous pouvez récupérer des informations sur les machines virtuelles Azure. Pour afficher des informations concernant une machine virtuelle, par exemple l’état de provisionnement de l’agent de machine virtuelle Azure, utilisez [Get-AzVM](/powershell/module/az.compute/get-azvm) :

```powershell
Get-AzVM
```

L’exemple de sortie condensée suivant montre la propriété *ProvisionVMAgent* imbriquée dans `OSProfile`. Cette propriété peut être utilisée pour déterminer si l’agent de machine virtuelle a été déployé sur la machine virtuelle :

```powershell
OSProfile                  :
  ComputerName             : myVM
  AdminUsername            : myUserName
  WindowsConfiguration     :
    ProvisionVMAgent       : True
    EnableAutomaticUpdates : True
```

Vous pouvez utiliser le script suivant pour retourner une liste concise des noms de machines virtuelles et l’état de l’agent de machine virtuelle :

```powershell
$vms = Get-AzVM

foreach ($vm in $vms) {
    $agent = $vm | Select -ExpandProperty OSProfile | Select -ExpandProperty Windowsconfiguration | Select ProvisionVMAgent
    Write-Host $vm.Name $agent.ProvisionVMAgent
}
```

### <a name="manual-detection"></a>Détection manuelle

Lorsque vous êtes connecté à une machine virtuelle Windows, utilisez le Gestionnaire des tâches pour examiner les processus en cours d’exécution. Pour rechercher l’agent de machine virtuelle Azure, ouvrez le Gestionnaire des tâches, cliquez sur l’onglet *Détails* et recherchez le nom de processus **WindowsAzureGuestAgent.exe**. La présence de ce processus indique que l’agent de machine virtuelle est installé.


## <a name="upgrade-the-vm-agent"></a>Mettre à niveau l’agent de machine virtuelle
L’agent de machine virtuelle Azure pour Windows est automatiquement mis à niveau sur les images déployées à partir de la Place de marché Azure. Lorsque de nouvelles machines virtuelles sont déployées sur Azure, elles reçoivent l’agent de machine virtuelle le plus récent au moment de leur provisionnement. Si vous avez installé l’agent manuellement ou si vous déployez des images de machine virtuelle personnalisées, vous devez effectuer une mise à jour manuelle pour inclure le nouvel agent de machine virtuelle au moment de la création de l’image.

## <a name="windows-guest-agent-automatic-logs-collection"></a>Collecte automatique des journaux de l’agent invité Windows
L’agent invité Windows dispose d’une fonctionnalité qui permet de collecter automatiquement certains journaux. Cette fonctionnalité est contrôlée par le processus CollectGuestLogs. exe. Elle existe à la fois pour les services cloud PaaS et pour les machines virtuelles IaaS. Sa fonction est de collecter de façon rapide et automatique certains journaux de diagnostic à partir d’une machine virtuelle en vue d’une analyse hors connexion. Parmi les journaux collectés figurent les journaux des événements, les journaux du système d’exploitation, les journaux Azure et certaines clés de Registre. Elle génère un fichier ZIP qui est transféré à l’hôte de la machine virtuelle. Ce fichier ZIP peut ensuite être examiné par les équipes d’ingénieurs et les professionnels du support technique pour étudier les problèmes à la demande du client propriétaire de la machine virtuelle.

## <a name="guest-agent-and-osprofile-certificates"></a>Agent invité et certificats OSProfile
L’agent de machine virtuelle Azure est chargé d’installer les certificats référencés dans l’`OSProfile` d’une machine virtuelle ou d’un groupe de machines virtuelles identiques. Si vous supprimez manuellement ces certificats de la console MMC Certificats à l’intérieur de la machine virtuelle invitée, vous pouvez vous attendre à ce que l’agent invité les rajoute.
Pour supprimer définitivement un certificat, vous devez le supprimer de l’`OSProfile`, puis le supprimer à partir du système d’exploitation invité.

Pour une machine virtuelle, utilisez [Remove-AzVMSecret]() pour supprimer des certificats de l’`OSProfile`.

Pour plus d’informations sur les certificats de groupe de machines virtuelles identiques, consultez [Groupes de machines virtuelles identiques - Comment supprimer des certificats obsolètes ?](../../virtual-machine-scale-sets/virtual-machine-scale-sets-faq.md#how-do-i-remove-deprecated-certificates)


## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les extensions de machine virtuelle, consultez [Vue d’ensemble des extensions et des fonctionnalités des machines virtuelles Azure](overview.md).