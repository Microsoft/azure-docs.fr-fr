---
title: Extension de machine virtuelle Agent Azure Network Watcher pour Windows
description: Déployez l’Agent Network Watcher sur une machine virtuelle Windows avec une extension de machine virtuelle.
services: virtual-machines-windows
documentationcenter: ''
author: mimckitt
manager: vashan
editor: ''
tags: azure-resource-manager
ms.assetid: 27e46af7-2150-45e8-b084-ba33de8c5e3f
ms.service: virtual-machines-windows
ms.subservice: extensions
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 02/14/2017
ms.author: mimckitt
ms.openlocfilehash: 813fc7db2a028590797bef1c54fff20b231c3d7d
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965899"
---
# <a name="network-watcher-agent-virtual-machine-extension-for-windows"></a>Extension de machine virtuelle Agent Network Watcher pour Windows

## <a name="overview"></a>Vue d’ensemble

[Azure Network Watcher](../../network-watcher/network-watcher-monitoring-overview.md) est un service d’analyse, de diagnostic et d’analytique des performances réseau permettant de surveiller les réseaux Azure. L’extension de machine virtuelle Agent Network Watcher est obligatoire pour capturer le trafic réseau à la demande et pour assurer la prise en charge d’autres fonctionnalités avancées sur les machines virtuelles Azure.


Ce document présente les plateformes et options de déploiement prises en charge pour l’extension de machine virtuelle Agent Network Watcher pour Windows. L’installation de l’agent n’est pas perturbante et ne nécessite pas de redémarrage de la machine virtuelle. Vous pouvez déployer l’extension dans les machines virtuelles que vous déployez. Si la machine virtuelle est déployée par un service Azure, consultez la documentation relative au service pour déterminer si elle autorise l’installation d’extensions dans la machine virtuelle.

## <a name="prerequisites"></a>Prérequis

### <a name="operating-system"></a>Système d’exploitation

L’extension Agent Network Watcher pour Windows peut être exécutée sur Windows Server 2008 R2, 2012, 2012 R2, 2016 et 2019. Nano Server n’est pas pris en charge pour le moment.

### <a name="internet-connectivity"></a>Connectivité Internet

Certaines fonctionnalités de l’Agent Network Watcher requièrent que la machine virtuelle cible soit connectée à Internet. Sans la capacité d’établir des connexions sortantes, l’Agent Network Watcher ne sera pas en mesure de charger des captures de paquets sur votre compte de stockage. Pour plus d’informations, consultez la [documentation de Network Watcher](../../network-watcher/network-watcher-monitoring-overview.md).

## <a name="extension-schema"></a>Schéma d’extensions

Le JSON suivant illustre le schéma de l’extension Agent Network Watcher. L’extension ne requiert ni ne prend en charge aucun paramètre fourni par l’utilisateur et s’appuie sur sa configuration par défaut.

```json
{
    "type": "extensions",
    "name": "Microsoft.Azure.NetworkWatcher",
    "apiVersion": "[variables('apiVersion')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
    ],
    "properties": {
        "publisher": "Microsoft.Azure.NetworkWatcher",
        "type": "NetworkWatcherAgentWindows",
        "typeHandlerVersion": "1.4",
        "autoUpgradeMinorVersion": true
    }
}
```

### <a name="property-values"></a>Valeurs de propriétés

| Nom | Valeur/Exemple |
| ---- | ---- |
| apiVersion | 2015-06-15 |
| publisher | Microsoft.Azure.NetworkWatcher |
| type | NetworkWatcherAgentWindows |
| typeHandlerVersion | 1.4 |


## <a name="template-deployment"></a>Déploiement de modèle

Vous pouvez déployer les extensions de machines virtuelles Azure avec des modèles Azure Resource Manager. Vous pouvez utiliser le schéma JSON détaillé dans la section précédente dans un modèle Azure Resource Manager pour exécuter l’extension Agent Network Watcher pendant un déploiement de modèle Azure Resource Manager.

## <a name="powershell-deployment"></a>Déploiement PowerShell

Utilisez la commande `Set-AzVMExtension` pour déployer l’extension de machine virtuelle Agent Network Watcher sur une machine virtuelle existante :

```powershell
Set-AzVMExtension `
  -ResourceGroupName "myResourceGroup1" `
  -Location "WestUS" `
  -VMName "myVM1" `
  -Name "networkWatcherAgent" `
  -Publisher "Microsoft.Azure.NetworkWatcher" `
  -Type "NetworkWatcherAgentWindows" `
  -TypeHandlerVersion "1.4"
```

## <a name="troubleshooting-and-support"></a>Résolution des problèmes et support

### <a name="troubleshooting"></a>Dépannage

Vous pouvez récupérer les données sur l’état des déploiements d’extension à partir du portail Azure et à l’aide de PowerShell. Pour afficher l’état du déploiement des extensions pour une machine virtuelle donnée, exécutez la commande suivante à l’aide du module Azure PowerShell :

```powershell
Get-AzVMExtension -ResourceGroupName myResourceGroup1 -VMName myVM1 -Name networkWatcherAgent
```

La sortie de l’exécution de l’extension est enregistrée dans les fichiers qui que se trouvent dans le répertoire suivant :

```cmd
C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.NetworkWatcher.NetworkWatcherAgentWindows\
```

### <a name="support"></a>Support

Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, vous pouvez consulter la documentation Guide d’utilisation de Network Watcher ou contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur Obtenir un support. Pour plus d’informations sur l’utilisation du support Azure, lisez le [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).
