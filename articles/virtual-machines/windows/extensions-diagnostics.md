---
title: Extension Diagnostics Azure pour Windows
description: Surveiller les machines virtuelles Windows Azure à l’aide de l’extension Diagnostics Azure
author: johnkemnetz
manager: ashwink
ms.service: virtual-machines-windows
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 04/06/2018
ms.author: johnkem
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: c981214c69653b3a4a687a861cb348f3587def7c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89078756"
---
# <a name="azure-diagnostics-extension-for-windows-vms"></a>Extension Diagnostics Azure pour machines virtuelles Windows

## <a name="overview"></a>Vue d’ensemble

L’extension de machine virtuelle Diagnostics Azure vous permet de collecter des données de surveillance, comme des compteurs de performances et des journaux d’événements, à partir de votre machine virtuelle Windows. Vous pouvez définir de façon granulaire les données à collecter et leur emplacement de destination, par exemple un compte Stockage Azure ou un hub d’événements Azure. Vous pouvez également utiliser ces données pour générer des graphiques dans le portail Azure ou créer des alertes de métrique.

## <a name="prerequisites"></a>Conditions préalables requises

### <a name="operating-system"></a>Système d’exploitation

L’extension Diagnostics Azure peut être exécutée sur un client Windows 10, ainsi que sur Windows Server 2008 R2, 2012, 2012 R2 et 2016.

### <a name="internet-connectivity"></a>Connectivité Internet

L’extension Diagnostics Azure nécessite que la machine virtuelle cible soit connectée à Internet. 

## <a name="extension-schema"></a>Schéma d’extensions

[Le schéma et les valeurs de propriété de l’extension Diagnostics Azure sont décrits dans ce document.](../../azure-monitor/platform/diagnostics-extension-schema-windows.md)

## <a name="template-deployment"></a>Déploiement de modèle

Les extensions de machines virtuelles Azure peuvent être déployées avec des modèles Azure Resource Manager. Le schéma JSON détaillé dans la section précédente peut être utilisé dans un modèle Azure Resource Manager pour exécuter l’extension Diagnostics Azure pendant le déploiement d’un modèle Azure Resource Manager. Pour plus d’informations, consultez [Utiliser la surveillance et les diagnostics avec une machine virtuelle Windows et des modèles Azure Resource Manager](../extensions/diagnostics-template.md).

## <a name="azure-cli-deployment"></a>Déploiement de l’interface de ligne de commande Azure

Vous pouvez utiliser Azure CLI pour déployer l’extension Diagnostics Azure sur une machine virtuelle existante. Remplacez les paramètres protégés et leurs propriétés par un JSON valide issu du schéma d’extension ci-dessus. 

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name IaaSDiagnostics \
  --publisher Microsoft.Azure.Diagnostics \
  --version 1.9.0.0 --protected-settings protected-settings.json \
  --settings public-settings.json 
```

## <a name="powershell-deployment"></a>Déploiement PowerShell

Vous pouvez utiliser la commande `Set-AzVMDiagnosticsExtension` pour ajouter l’extension Diagnostics Azure à une machine virtuelle existante. Voir aussi [Utiliser PowerShell pour activer Diagnostics Azure sur une machine virtuelle exécutant Windows](../extensions/diagnostics-windows.md).

 


```powershell
$vm_resourcegroup = "myvmresourcegroup"
$vm_name = "myvm"
$diagnosticsconfig_path = "DiagnosticsPubConfig.xml"

Set-AzVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup `
  -VMName $vm_name `
  -DiagnosticsConfigurationPath $diagnosticsconfig_path
```

## <a name="troubleshoot-and-support"></a>Dépannage et support technique

### <a name="troubleshoot"></a>Dépanner

Vous pouvez récupérer les données sur l’état des déploiements d’extension à partir du portail Azure et à l’aide de l’interface de ligne de commande Azure. Pour afficher l’état du déploiement des extensions pour une machine virtuelle donnée, exécutez la commande suivante à l’aide de l’interface de ligne de commande Azure.

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

Pour plus d’informations sur la résolution des problèmes liés à l’extension Diagnostics Azure, [consultez cet article](../../azure-monitor/platform/diagnostics-extension-troubleshooting.md).

### <a name="support"></a>Support

Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, vous pouvez contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur Obtenir un support. Pour plus d’informations sur l’utilisation du support Azure, lisez le [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="next-steps"></a>Étapes suivantes
* [En savoir plus sur l’extension Diagnostics Azure](../../azure-monitor/platform/diagnostics-extension-overview.md)
* [Examinez le schéma et les versions de l’extension](../../azure-monitor/platform/diagnostics-extension-schema-windows.md)
