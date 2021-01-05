---
title: Résolution des problèmes liés aux échecs d’extension de machine virtuelle Windows
description: En savoir plus sur la résolution des problèmes pour les échecs d’extension de machine virtuelle Windows dans Azure
services: virtual-machines-windows
documentationcenter: ''
author: kundanap
manager: gwallace
editor: ''
tags: top-support-issue,azure-resource-manager
ms.assetid: 878ab9b6-c3e6-40be-82d4-d77fecd5030f
ms.service: virtual-machines-windows
ms.subservice: extensions
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 03/29/2016
ms.author: kundanap
ms.openlocfilehash: bca826cda8dfe47c341886faaf4a0d66f09d37d2
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966341"
---
# <a name="troubleshooting-azure-windows-vm-extension-failures"></a>Dépannage des échecs d’extension de machine virtuelle Windows dans Azure
[!INCLUDE [virtual-machines-common-extensions-troubleshoot](../../../includes/virtual-machines-common-extensions-troubleshoot.md)]

## <a name="viewing-extension-status"></a>Affichage de l’état de l’extension
Les modèles Azure Resource Manager sont exécutables à partir d’Azure PowerShell. Une fois que le modèle est exécuté, l'état de l'extension peut être affiché à partir d'Azure Resource Explorer ou des outils de ligne de commande.

Voici un exemple :

Azure PowerShell :

```azurepowershell
Get-AzVM -ResourceGroupName $RGName -Name $vmName -Status
```

Voici l'exemple de sortie :

```output
Extensions:  {
  "ExtensionType": "Microsoft.Compute.CustomScriptExtension",
  "Name": "myCustomScriptExtension",
  "SubStatuses": [
    {
      "Code": "ComponentStatus/StdOut/succeeded",
      "DisplayStatus": "Provisioning succeeded",
      "Level": "Info",
      "Message": "    Directory: C:\\temp\\n\\n\\nMode                LastWriteTime     Length Name
          \\n----                -------------     ------ ----                              \\n-a---          9/1/2015   2:03 AM         11
          test.txt                          \\n\\n",
                  "Time": null
      },
    {
      "Code": "ComponentStatus/StdErr/succeeded",
      "DisplayStatus": "Provisioning succeeded",
      "Level": "Info",
      "Message": "",
      "Time": null
    }
  ]
}
```

## <a name="troubleshooting-extension-failures"></a>Dépannage des échecs d’extension

### <a name="rerun-the-extension-on-the-vm"></a>Réexécuter l’extension sur la machine virtuelle
Si vous exécutez des scripts sur la machine virtuelle à l’aide de l’extension de script personnalisé, cela peut générer une erreur indiquant que la machine virtuelle a été créée avec succès mais que le script a échoué. Dans ces conditions, la méthode recommandée pour corriger cette erreur consiste à supprimer l'extension et exécuter le modèle à nouveau.
Remarque : cette fonctionnalité sera ultérieurement améliorée de manière à ne plus devoir désinstaller l'extension.

#### <a name="remove-the-extension-from-azure-powershell"></a>Suppression de l’extension d’Azure PowerShell
```azurepowershell
Remove-AzVMExtension -ResourceGroupName $RGName -VMName $vmName -Name "myCustomScriptExtension"
```

Une fois que l'extension a été supprimée, le modèle peut être réexécuté pour exécuter les scripts sur la machine virtuelle.

### <a name="trigger-a-new-goalstate-to-the-vm"></a>Déclencher un nouveau GoalState sur la machine virtuelle
Vous pouvez remarquer qu’une extension n’a pas été exécutée ou ne peut pas s’exécuter en raison d’un « générateur de certificats CRP Windows Azure » manquant (ce certificat est utilisé pour sécuriser le transport des paramètres protégés de l’extension).
Ce certificat sera automatiquement régénéré en redémarrant l’agent invité Windows à partir de la machine virtuelle :
- Ouvrir le Gestionnaire des tâches
- Accéder à l’onglet Détails
- Rechercher le processus WindowsAzureGuestAgent.exe
- Cliquez avec le bouton droit, puis sélectionnez « Terminer la tâche ». Le processus sera redémarré automatiquement.


Vous pouvez également déclencher un nouveau GoalState sur la machine virtuelle, en exécutant une « mise à jour vide » :

Azure PowerShell :

```azurepowershell
$vm = Get-AzureRMVM -ResourceGroupName <RGName> -Name <VMName>  
Update-AzureRmVM -ResourceGroupName <RGName> -VM $vm  
```

Azure CLI :

```azurecli
az vm update -g <rgname> -n <vmname>
```

Si une « mise à jour vide » n’a pas fonctionné, vous pouvez ajouter un nouveau disque de données vide à la machine virtuelle à partir du portail de gestion Azure, puis le supprimer une fois le certificat rajouté.
