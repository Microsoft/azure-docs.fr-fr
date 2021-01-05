---
title: 'Démarrage rapide : Créer un équilibreur de charge interne à l’aide d’un modèle'
description: Ce guide de démarrage rapide montre comment créer un équilibreur de charge Azure interne avec un modèle Azure Resource Manager (modèle ARM).
services: load-balancer
author: asudbring
ms.service: load-balancer
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurecli
ms.author: allensu
ms.date: 09/14/2020
ms.openlocfilehash: 749b233b827c27d2c998cfd6be66cf79cf48089d
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94831648"
---
# <a name="quickstart-create-an-internal-load-balancer-to-load-balance-vms-by-using-an-arm-template"></a>Démarrage rapide : Créer un équilibreur de charge interne pour équilibrer la charge de machines virtuelles à l’aide d’un modèle Resource Manager

Ce guide de démarrage rapide décrit comment utiliser un modèle Azure Resource Manager pour créer un équilibreur de charge Azure interne.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-2-vms-internal-load-balancer%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce guide de démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/201-2-vms-internal-load-balancer).

:::code language="json" source="~/quickstart-templates/201-2-vms-internal-load-balancer/azuredeploy.json":::

Plusieurs ressources Azure ont été définies dans le modèle :

- [**Microsoft.Storage/storageAccounts**](/azure/templates/microsoft.storage/storageaccounts) : Comptes de stockage de machines virtuelles pour les diagnostics de démarrage.
- [**Microsoft.Compute/availabilitySets**](/azure/templates/microsoft.compute/availabilitySets) : Groupe à haute disponibilité pour des machines virtuelles.
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualNetworks) : Réseau virtuel pour l’équilibreur de charge et les machines virtuelles.
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkInterfaces) : Interfaces réseau pour les machines virtuelles.
- [**Microsoft.Network/loadBalancers**](/azure/templates/microsoft.network/loadBalancers) : Équilibreur de charge interne.
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualMachines) : Machines virtuelles

Pour rechercher d’autres modèles qui sont liés à l’équilibrage de charge Azure, consultez [Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Déployer le modèle

**Azure CLI**

```azurecli-interactive
read -p "Enter the location (i.e. westcentralus): " location
resourceGroupName="myResourceGroupLB"
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.json"

az group create \
--name $resourceGroupName \
--location $location

az deployment group create \
--resource-group $resourceGroupName \
--template-uri  $templateUri
```

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez **Groupes de ressources** dans le volet gauche.

1. Sélectionnez le groupe de ressources que vous avez créé dans la section précédente. Le nom du groupe de ressources par défaut est **myResourceGroupLB**.

1. Vérifiez que les ressources suivantes ont été créées dans le groupe de ressources :

:::image type="content" source="media/quickstart-load-balancer-standard-internal-template/verify-creation.png" alt-text="Utilisez le portail Azure pour vérifier la création de ressources." border="true":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, utilisez la commande [az group delete](/cli/azure/group#az-group-delete) pour supprimer le groupe de ressources et toutes les ressources qu’il contient.

```azurecli-interactive
  az group delete \
    --name myResourceGroupLB
```

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir un tutoriel pas à pas vous guidant tout au long du processus de création d’un modèle, consultez :

> [!div class="nextstepaction"]
> [Tutoriel : Créer et déployer votre premier modèle Resource Manager](../azure-resource-manager/templates/template-tutorial-create-first-template.md)
