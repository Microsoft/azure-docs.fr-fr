---
title: Mapper des disques Azure à des disques invités de machine virtuelle Linux
description: Identifier les disques Azure qui sous-tendent les disques invités d’une machine virtuelle Linux.
author: timbasham
ms.service: virtual-machines-linux
ms.subservice: disks
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/17/2020
ms.author: tibasham
ms.openlocfilehash: 4f0e48bf1c14728c54d4e89f30700017b0420d7d
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96523295"
---
# <a name="how-to-map-azure-disks-to-linux-vm-guest-disks"></a>Mapper des disques Azure à des disques invités de machine virtuelle Linux

Vous devrez peut-être identifier les disques Azure qui sous-tendent les disques invités d’une machine virtuelle. Dans certains scénarios, vous pouvez comparer la taille ou le volume du disque à la taille des disques Azure attachés. Dans les scénarios où plusieurs disques Azure de la même taille sont attachés à la machine virtuelle, vous devez utiliser le numéro d’unité logique (LUN) des disques de données. 

## <a name="what-is-a-lun"></a>Qu’est qu’un LUN ?

Un numéro d’unité logique (LUN) est un numéro utilisé pour identifier un périphérique de stockage spécifique. Chaque périphérique de stockage reçoit un identificateur numérique unique, en commençant à zéro. Le chemin d’accès complet à un périphérique est représenté par le numéro de bus, le numéro d’ID cible et le numéro d’unité logique (LUN). 

Par exemple : ***Bus Number 0, Target ID 0, LUN 3** _

Pour notre exercice, le LUN suffit.

## <a name="finding-the-lun"></a>Recherche du LUN

Vous trouverez ci-dessous deux méthodes pour rechercher le LUN d’un disque dans Linux.

### <a name="lsscsi"></a>lsscsi

1. Connexion à la machine virtuelle
1. `sudo lsscsi`

La première colonne indiquée affiche le LUN au format [Hôte:Canal:Cible:_*LUN**].

### <a name="listing-block-devices"></a>Liste des appareils de traitement par blocs

1. Connexion à la machine virtuelle
1. `sudo ls -l /sys/block/*/device`

La dernière colonne indiquée affiche le LUN au format [Hôte:Canal:Cible:**LUN**]

## <a name="finding-the-lun-for-the-azure-disks"></a>Recherche du LUN pour les disques Azure

Vous pouvez localiser le LUN d’un disque Azure à l’aide du portail Azure ou de l’interface de ligne de commande Azure.

### <a name="finding-an-azure-disks-lun-in-the-azure-portal"></a>Recherche du LUN d’un disque Azure dans le portail Azure

1. Dans le portail Azure, sélectionnez « Machines virtuelles » pour afficher la liste de vos machines virtuelles
1. Sélectionnez la machine virtuelle.
1. Sélectionnez « Disques »
1. Sélectionnez un disque de données dans la liste des disques attachés.
1. Le LUN du disque s’affiche dans le volet des détails du disque. Le LUN affiché ici correspond aux LUN que vous avez recherchés dans l’invité en utilisant lsscsi ou en répertoriant les appareils de traitement par blocs.

### <a name="finding-an-azure-disks-lun-using-azure-cli"></a>Recherche du LUN d’un disque Azure à l’aide de l’interface de ligne de commande Azure

```azurecli-interactive
az vm show -g myResourceGroup -n myVM --query "storageProfile.dataDisks"
```
