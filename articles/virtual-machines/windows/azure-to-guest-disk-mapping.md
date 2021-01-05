---
title: Mapper des disques Azure à des disques invités de machine virtuelle Windows
description: Identifier les disques Azure qui sous-tendent les disques invités d’une machine virtuelle Windows.
author: timbasham
ms.service: virtual-machines-windows
ms.subservice: disks
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/17/2020
ms.author: tibasham
ms.openlocfilehash: f16e34f372016f284d4af79443e84d9d5cdea957
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96523288"
---
# <a name="how-to-map-azure-disks-to-windows-vm-guest-disks"></a>Mapper des disques Azure à des disques invités de machine virtuelle Windows

Vous devrez peut-être identifier les disques Azure qui sous-tendent les disques invités d’une machine virtuelle. Dans certains scénarios, vous pouvez comparer la taille ou le volume du disque à la taille des disques Azure attachés. Dans les scénarios où plusieurs disques Azure de la même taille sont attachés à la machine virtuelle, vous devez utiliser le numéro d’unité logique (LUN) des disques de données. 

## <a name="what-is-a-lun"></a>Qu’est qu’un LUN ?

Un numéro d’unité logique (LUN) est un numéro utilisé pour identifier un périphérique de stockage spécifique. Chaque périphérique de stockage reçoit un identificateur numérique unique, en commençant à zéro. Le chemin d’accès complet à un périphérique est représenté par le numéro de bus, le numéro d’ID cible et le numéro d’unité logique (LUN). 

Exemple : ***Bus Number 0, Target ID 0, LUN 3***

Pour notre exercice, le LUN suffit.

## <a name="finding-the-lun"></a>Recherche du LUN

Il existe deux méthodes pour rechercher le LUN, selon que vous utilisez des [espaces de stockage](https://docs.microsoft.com/windows-server/storage/storage-spaces/overview) ou non.

### <a name="disk-management"></a>Gestion des disques

Si vous n’utilisez pas de pools de stockage, vous pouvez utiliser [Gestion des disques](https://docs.microsoft.com/windows-server/storage/disk-management/overview-of-disk-management) pour rechercher le LUN.

1. Connectez-vous à la machine virtuelle et ouvrez Gestion des disques a. Cliquez avec le bouton droit sur le bouton Démarrer et choisissez « Gestion des disques » a. Vous pouvez également taper `diskmgmt.msc` dans la zone Rechercher
1. Dans le volet inférieur, cliquez avec le bouton droit sur l’un des disques, puis choisissez « Propriétés »
1. Le LUN apparaît dans la propriété « Emplacement » de l’onglet « Général »

### <a name="storage-pools"></a>Pools de stockage

1. Connectez-vous à la machine virtuelle et ouvrez Gestionnaire de serveur
1. Sélectionnez « Services de fichiers et de stockage », « Volumes », « Pools de stockage »
1. Une section « Disques physiques » apparaît dans le coin inférieur droit du Gestionnaire de serveur. Les disques qui composent le pool de stockage ainsi que le LUN de chaque disque sont répertoriés ici.

## <a name="finding-the-lun-for-the-azure-disks"></a>Recherche du LUN pour les disques Azure

Vous pouvez localiser le LUN d’un disque Azure à l’aide du portail Azure, de l’interface de ligne de commande Azure ou d’Azure PowerShell

### <a name="finding-an-azure-disks-lun-in-the-azure-portal"></a>Recherche du LUN d’un disque Azure dans le portail Azure

1. Dans le portail Azure, sélectionnez « Machines virtuelles » pour afficher la liste de vos machines virtuelles
1. Sélectionnez la machine virtuelle.
1. Sélectionnez « Disques »
1. Sélectionnez un disque de données dans la liste des disques attachés.
1. Le LUN du disque s’affiche dans le volet des détails du disque. Le LUN affiché ici correspond aux LUN recherchés dans l’invité à l’aide du Gestionnaire de périphériques ou du Gestionnaire de serveur.

### <a name="finding-an-azure-disks-lun-using-azure-cli-or-azure-powershell"></a>Recherche du LUN d’un disque Azure à l’aide de l’interface de ligne de commande Azure ou d’Azure PowerShell

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli-interactive
az vm show -g myResourceGroup -n myVM --query "storageProfile.dataDisks"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)
```azurepowershell-interactive
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
$vm.StorageProfile.DataDisks | ft
```
---
