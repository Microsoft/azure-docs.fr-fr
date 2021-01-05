---
title: 'Démarrage rapide : Créer un circuit ExpressRoute en utilisant un modèle Azure Resource Manager (modèle ARM)'
description: Ce guide de démarrage rapide vous montre comment créer un circuit ExpressRoute avec un modèle Azure Resource Manager (modèle ARM).
services: expressroute
author: duongau
mnager: kumud
ms.service: expressroute
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 10/12/2020
ms.author: duau
ms.openlocfilehash: 7521344a2bb6aae67724c8bfbb9131e2ff1e6b94
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92789724"
---
# <a name="quickstart-create-an-expressroute-circuit-with-private-peering-using-an-arm-template"></a>Démarrage rapide : Créer un circuit ExpressRoute avec un peering privé en utilisant un modèle ARM

Ce guide de démarrage rapide décrit comment utiliser un modèle Azure Resource Manager (modèle ARM) pour créer un circuit ExpressRoute avec un peering privé.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure** . Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-expressroute-private-peering-vnet%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-expressroute-private-peering-vnet).

Dans ce guide de démarrage rapide, vous créez un circuit ExpressRoute avec *Equinix* comme fournisseur de services. Le circuit utilise un *SKU Premium* , avec une bande passante de *50 Mbits/s* et la localisation de peering *Washington DC* . Le peering privé est activé avec les sous-réseaux principal et secondaire *192.168.10.16/30* et *192.168.10.20/30* , respectivement. Un réseau virtuel est également créé avec une *passerelle ExpressRoute HighPerformance* .

:::code language="json" source="~/quickstart-templates/101-expressroute-private-peering-vnet/azuredeploy.json":::

Plusieurs ressources Azure ont été définies dans le modèle :

* [**Microsoft.Network/expressRouteCircuits**](/azure/templates/microsoft.network/expressRouteCircuits)
* [**Microsoft.Network/expressRouteCircuits/peerings**](/azure/templates/microsoft.network/expressRouteCircuits/peerings) (utilisé pour activer le peering privé sur le circuit)
* [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networkSecurityGroups) (groupe de sécurité réseau appliqué aux sous-réseaux du réseau virtuel)
* [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualNetworks)
* [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicIPAddresses) (adresse IP publique utilisée par la passerelle ExpressRoute)
* [**Microsoft.Network/virtualNetworkGateways**](/azure/templates/microsoft.network/virtualNetworkGateways) (passerelle ExpressRoute utilisée pour connecter le réseau virtuel au circuit)

Pour rechercher d’autres modèles associés à ExpressRoute, consultez [Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Déployer le modèle

1. Sélectionnez **Essayer** à partir du bloc de code suivant pour ouvrir Azure Cloud Shell, puis suivez les instructions pour vous connecter à Azure.

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-expressroute-private-peering-vnet/azuredeploy.json"

    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

    Read-Host -Prompt "Press [ENTER] to continue ..."
    ```

    Patientez jusqu'à ce que vous voyiez l’invite de la console.

1. Sélectionnez **Copier** dans le bloc de code précédent pour copier le script PowerShell.

1. Cliquez sur le volet de la console shell, puis sélectionnez **Coller** .

1. Entrez les valeurs.

    Le nom du groupe de ressources est le nom du projet avec **rg** ajouté.

    Le déploiement du modèle prend environ 20 minutes. Une fois l’opération terminée, le résultat ressemble à ce qui suit :

    :::image type="content" source="./media/quickstart-create-expressroute-vnet/expressroute-powershell-output.png" alt-text="Sortie du déploiement PowerShell du modèle Resource Manager ExpressRoute":::

Azure PowerShell est utilisé pour déployer le modèle. Outre Azure PowerShell, vous pouvez également utiliser le portail Azure, l’interface Azure CLI et l’API REST. Pour découvrir d’autres méthodes de déploiement, consultez [Déployer des modèles](../azure-resource-manager/templates/deploy-portal.md).

## <a name="validate-the-deployment"></a>Valider le déploiement

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez **Groupes de ressources** dans le volet gauche.

1. Sélectionnez le groupe de ressources que vous avez créé dans la section précédente. Le nom du groupe de ressources par défaut est le nom du projet avec **rg** ajouté.

1. Le groupe de ressources doit contenir les ressources indiquées ci-dessous :

     :::image type="content" source="./media/quickstart-create-expressroute-vnet/expressroute-resource-group.png" alt-text="Sortie du déploiement PowerShell du modèle Resource Manager ExpressRoute":::

1. Sélectionnez le circuit ExpressRoute **er-ck01** pour vérifier que l’état du circuit est **Activé** , que l’état du fournisseur est **Non provisionné** et que l’état du peering privé est **Provisionné** .

    :::image type="content" source="./media/quickstart-create-expressroute-vnet/expressroute-circuit.png" alt-text="Sortie du déploiement PowerShell du modèle Resource Manager ExpressRoute":::

> [!NOTE]
> Vous devrez appeler le fournisseur pour terminer le processus de provisionnement avant de pouvoir connecter le réseau virtuel au circuit.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin des ressources que vous avez créées avec le circuit ExpressRoute, supprimez le groupe de ressources. Cette action supprime le circuit ExpressRoute et toutes les ressources qui y sont associées.

Pour supprimer le groupe de ressources, appelez l’applet de commande `Remove-AzResourceGroup` :

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Étapes suivantes

Dans le cadre de ce guide de démarrage rapide, vous avez créé les éléments suivants :

* Circuit ExpressRoute
* Réseau virtuel
* Passerelle VPN
* Adresse IP publique
* Groupes de sécurité réseau

Pour découvrir comment lier un réseau virtuel à un circuit, continuez avec les tutoriels ExpressRoute.

> [!div class="nextstepaction"]
> [Tutoriels ExpressRoute](expressroute-howto-linkvnet-portal-resource-manager.md)
