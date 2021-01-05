---
title: Créer des groupes IP dans Pare-feu Azure
description: Les groupes IP vous permettent de regrouper et de gérer des adresses IP pour les règles de Pare-feu Azure.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 06/23/2020
ms.author: victorh
ms.openlocfilehash: c53b35351108717c7b597e052a66e9902be5ec6c
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93394524"
---
# <a name="create-ip-groups"></a>Créer des groupes IP

Les groupes IP vous permettent de regrouper et de gérer des adresses IP pour les règles de Pare-feu Azure. Ils peuvent avoir une seule adresse IP, plusieurs adresses IP ou une ou plusieurs plages d’adresses IP.

## <a name="create-an-ip-group---azure-portal"></a>Créer un groupe IP – Portail Azure

Pour créer un groupe IP à l’aide du portail Azure :

1. Sur la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
1. Dans la zone de recherche, entrez **Groupes IP** puis sélectionnez **Groupes IP**.
1. Sélectionnez **Create** (Créer).
1. Sélectionnez votre abonnement.
1. Sélectionnez un groupe de ressources ou créez-en un.
1. Entrez un nom unique pour votre groupe IP, puis sélectionnez une région.
1. Sélectionnez **Suivant : adresses IP**.
1. Saisissez une adresse IP, plusieurs adresses IP ou des plages d’adresses IP.

   Il existe deux façons d’entrer des adresses IP :
   - Vous pouvez les entrer manuellement
   - Vous pouvez les importer à partir d’un fichier

   Pour les importer à partir d’un fichier, sélectionnez **Import from a file** (Importer à partir d’un fichier). Vous pouvez faire glisser votre fichier dans la zone ou sélectionner **Browse for files** (Parcourir les fichiers). Si nécessaire, vous pouvez passer en revue et modifier vos adresses IP chargées.

   Lorsque vous saisissez une adresse IP, le portail la valide pour vérifier les problèmes de chevauchement, de doublons et de mise en forme.

1. Quand vous avez terminé, sélectionnez **Review + Create** (Vérifier + créer).
1. Sélectionnez **Create** (Créer).

## <a name="create-an-ip-group---azure-powershell"></a>Créer un groupe IP – Azure PowerShell

Cet exemple crée un groupe IP avec un préfixe d’adresse et une adresse IP à l’aide d’Azure PowerShell :

```azurepowershell
$ipGroup = @{
    Name              = 'ipGroup'
    ResourceGroupName = 'ipGroupRG'
    Location          = 'West US'
    IpAddress         = @('10.0.0.0/24', '192.168.1.10') 
}

New-AzIpGroup @ipGroup
```

## <a name="create-an-ip-group---azure-cli"></a>Créer un groupe IP – Azure CLI

Cet exemple crée un groupe IP avec un préfixe d’adresse et une adresse IP à l’aide d’Azure CLI :

```azurecli-interactive
az network ip-group create \
    --name ipGroup \ 
    --resource-group ipGroupRG \
    --location westus \
    --ip-addresses '10.0.0.0/24' '192.168.1.10'
```

## <a name="next-steps"></a>Étapes suivantes

- [En savoir plus sur les groupes IP](ip-groups.md)
