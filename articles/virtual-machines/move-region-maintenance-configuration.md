---
title: Déplacer une configuration de maintenance vers une autre région Azure
description: Découvrez comment déplacer une configuration de maintenance de machine virtuelle vers une autre région Azure.
author: shants123
ms.service: virtual-machines
ms.topic: how-to
ms.tgt_pltfrm: vm
ms.date: 03/04/2020
ms.author: shants
ms.openlocfilehash: 4cff7eb4a69005f2e74747b6e58447f100c69b60
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86501600"
---
# <a name="move-a-maintenance-control-configuration-to-another-region"></a>Déplacer une configuration de contrôle de maintenance vers une autre région

Suivez cet article pour déplacer une configuration de contrôle de maintenance vers une autre région Azure. Vous pouvez envisager de déplacer une configuration pour plusieurs raisons. Par exemple, pour tirer parti d’une nouvelle région, pour déployer des fonctionnalités ou des services disponibles dans une région spécifique, pour répondre à des exigences de stratégie et de gouvernance internes ou pour respecter une planification de capacité.

Le contrôle de maintenance, avec des configurations de maintenance personnalisées, vous permet de contrôler la façon dont les mises à jour de plateforme sont appliquées aux machines virtuelles [Windows](./maintenance-control-cli.md?toc=/azure/virtual-machines/windows/toc.json&bc=/azure/virtual-machines/windows/breadcrumb/toc.json) et [Linux](./maintenance-control-cli.md?toc=%2Fazure%2Fvirtual-machines%2Flinux%2Ftoc.json&bc=%2Fazure%2Fvirtual-machines%2Flinux%2Fbreadcrumb%2Ftoc.json&view=azure-java-stable) et aux hôtes dédiés Azure. Différents scénarios peuvent impliquer le déplacement du contrôle de maintenance entre régions :

- Pour déplacer votre configuration de contrôle de maintenance, mais pas les ressources associées à la configuration, suivez les instructions de cet article.
- Pour déplacer les ressources associées à une configuration de maintenance, mais pas la configuration elle-même, suivez [ces instructions](move-region-maintenance-configuration-resources.md).
- Pour déplacer la configuration de maintenance et les ressources associées, suivez d’abord les instructions de cet article. Puis, suivez [ces instructions](move-region-maintenance-configuration-resources.md).

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer à déplacer une configuration de contrôle de maintenance :

- Les configurations de maintenance sont associées aux machines virtuelles Azure ou aux hôtes dédiés Azure. Assurez-vous d’abord que les machines virtuelles/ressources hôtes existent dans la nouvelle région.
- Identifiez : 
    - Les configurations de contrôle de maintenance existantes.
    - Les groupes de ressources dans lesquels résident actuellement les configurations existantes. 
    - Les groupes de ressources auxquels les configurations seront ajoutées après le déplacement vers la nouvelle région. 
    - Les ressources associées à la configuration de maintenance que vous souhaitez déplacer.
    - Vérifiez que les ressources de la nouvelle région sont les mêmes que celles associées aux configurations de maintenance actuelles. Les configurations peuvent avoir, dans la nouvelle région, les noms qu’elles portaient dans l’ancienne, mais ce n’est pas obligatoire.

## <a name="prepare-and-move"></a>Préparer et déplacer 

1. Récupérez toutes les configurations de maintenance de chaque abonnement. Pour cela, exécutez la commande CLI [az maintenance configuration list](/cli/azure/ext/maintenance/maintenance/configuration?view=azure-cli-latest#ext-maintenance-az-maintenance-configuration-list) en remplaçant $subId par votre ID d’abonnement.

    ```
    az maintenance configuration list --subscription $subId --query "[*].{Name:name, Location:location, ResGroup:resourceGroup}" --output table
    ```
2. Passez en revue la liste en tableau retournée, qui présente les enregistrements de configuration de l’abonnement. Voici un exemple. Votre liste contiendra les valeurs correspondant à votre environnement spécifique.

    **Nom** | **Lieu** | **Groupe de ressources**
    --- | --- | ---
    Skip Maintenance | eastus2 | configuration-resource-group
    IgniteDemoConfig | eastus2 | configuration-resource-group
    defaultMaintenanceConfiguration-eastus | eastus | test-configuration
    

3. Enregistrez votre liste pour référence. Elle vous permet de vérifier que toutes les configurations sont déplacées, au fur et à mesure.
4. À titre de référence, mappez chaque configuration/groupe de ressources au nouveau groupe de ressources dans la nouvelle région.
5. Créez des configurations de maintenance dans la nouvelle région à l’aide de [PowerShell](../virtual-machines/maintenance-control-powershell.md#create-a-maintenance-configuration) ou de l’[interface CLI](../virtual-machines/maintenance-control-cli.md#create-a-maintenance-configuration).
6. Associez les configurations aux ressources dans la nouvelle région à l’aide de [PowerShell](../virtual-machines/maintenance-control-powershell.md#assign-the-configuration) ou de l’[interface CLI](../virtual-machines/maintenance-control-cli.md#assign-the-configuration).


## <a name="verify-the-move"></a>Vérifier le déplacement

Après avoir déplacé les configurations, comparez les configurations et les ressources de la nouvelle région à la liste en tableau que vous avez préparée.


## <a name="clean-up-source-resources"></a>Nettoyer les ressources sources

Après le déplacement, pensez à supprimer les configurations de maintenance déplacées de la région source avec [PowerShell](../virtual-machines/maintenance-control-powershell.md#remove-a-maintenance-configuration) ou l’[interface CLI](../virtual-machines/maintenance-control-cli.md#delete-a-maintenance-configuration).


## <a name="next-steps"></a>Étapes suivantes

Suivez [ces instructions](move-region-maintenance-configuration-resources.md) si vous devez déplacer les ressources associées aux configurations de maintenance. 
