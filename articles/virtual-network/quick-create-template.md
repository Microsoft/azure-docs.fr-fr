---
title: 'Démarrage rapide : Créer un réseau virtuel à l’aide d’un modèle Azure Resource Manager'
titleSuffix: Azure Virtual Network
description: Découvrez comment utiliser un modèle Resource Manager pour créer un réseau virtuel Azure
services: virtual-network
author: KumudD
ms.service: virtual-network
ms.topic: quickstart
ms.date: 06/23/2020
ms.author: kumud
ms.custom: ''
ms.openlocfilehash: c9e9ea2e59537a1c0c8e372a766fba3aa9a1b9a0
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88122153"
---
# <a name="quickstart-create-a-virtual-network---resource-manager-template"></a>Démarrage rapide : Créer un réseau virtuel – Modèle Resource Manager

Ce démarrage rapide montre comment créer un réseau virtuel avec deux sous-réseaux à l’aide du modèle Azure Resource Manager. Un réseau virtuel est le bloc de construction fondamental de votre réseau privé dans Azure. Il permet à des ressources Azure, comme des machines virtuelles, de communiquer de manière sécurisée entre elles et avec Internet.


[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Vous pouvez également suivre ce guide de démarrage rapide en utilisant le [portail Azure](quick-create-portal.md), [Azure PowerShell](quick-create-powershell.md) ou [Azure CLI](quick-create-cli.md).

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce guide de démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://github.com/Azure/azure-quickstart-templates/blob/master/101-vnet-two-subnets/azuredeploy.json).

:::code language="json" source="~/quickstart-templates/101-vnet-two-subnets/azuredeploy.json" range="001-96" highlight="56-92":::

Les ressources Azure suivantes ont été définies dans le modèle :
- [**Microsoft.Network/virtualNetworks**](https://docs.microsoft.com/azure/templates/microsoft.network/virtualnetworks) : permet de créer un réseau virtuel Azure.
-  [**Microsoft.Network/virtualNetworks/subnets**](https://docs.microsoft.com/azure/templates/microsoft.network/virtualnetworks/subnets) : permet de créer un sous-réseau.

## <a name="deploy-the-template"></a>Déployer le modèle

Déployez le modèle Resource Manager sur Azure :

1. Sélectionnez **Déployer sur Azure** pour vous connecter à Azure et ouvrir le modèle. Le modèle crée un réseau virtuel avec deux sous-réseaux.

   [![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-virtual-network-2vms-create%2Fazuredeploy.json)

2. Dans le portail, dans la page **Créer un réseau virtuel avec deux sous-réseaux**, tapez ou sélectionnez les valeurs suivantes :
   - **Groupe de ressources** : Sélectionnez **Créer nouveau**, tapez un nom pour le groupe de ressources et sélectionnez **OK**.
   - **Nom du réseau virtuel** : tapez le nom du nouveau réseau virtuel.
3. Sélectionnez **Examiner + créer**, puis sélectionnez **Créer**.

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

Explorez les ressources créées avec le réseau virtuel.

Pour découvrir la syntaxe et les propriétés JSON d’un réseau virtuel dans un modèle, consultez [Microsoft.Network/azureFirewalls](https://docs.microsoft.com/azure/templates/microsoft.network/virtualnetworks).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin des ressources que vous avez créées avec le pare-feu, supprimez le groupe de ressources. Cela a pour effet de supprimer le réseau virtuel et toutes les ressources associées.

Pour supprimer le groupe de ressources, appelez l’applet de commande `Remove-AzResourceGroup` :

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Étapes suivantes
Dans ce démarrage rapide, vous avez déployé un réseau virtuel Azure avec deux sous-réseaux. Pour en savoir plus sur les réseaux virtuels Azure, consultez le tutoriel sur les réseaux virtuels.

> [!div class="nextstepaction"]
> [Filtrer le trafic réseau](tutorial-filter-network-traffic.md)
