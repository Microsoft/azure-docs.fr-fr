---
title: Créer une instance de DMS (modèle Azure Resource Manager)
description: Découvrez comment créer une instance de Database Migration Service à l’aide d’un modèle ARM (Azure Resource Manager).
author: MashaMSFT
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: mathoma
ms.date: 06/29/2020
ms.service: dms
ms.openlocfilehash: a9c68bca4d50af734a0a2cd8a91c7e46d9b56ff1
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94963213"
---
# <a name="quickstart-create-instance-of-azure-database-migration-service-using-arm-template"></a>Démarrage rapide : Créer une instance du service Azure Database Migration Service à l’aide du modèle ARM

Utilisez ce modèle ARM (Azure Resource Manager) pour déployer une instance d’Azure Database Migration Service. 

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-azure-database-migration-simple-deploy%2fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Le modèle Resource Manager Azure Database Migration Service nécessite les éléments suivants : 

- La version la plus récente de l’[interface de ligne de commande Azure](/cli/azure/install-azure-cli) et/ou de [PowerShell](/powershell/scripting/install/installing-powershell). 
- Un abonnement Azure. Si vous n’en avez pas, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-azure-database-migration-simple-deploy/).

:::code language="json" source="~/quickstart-templates/101-azure-database-migration-simple-deploy/azuredeploy.json":::

Trois ressources Azure sont définies dans le modèle : 

- [Microsoft.Network/virtualNetworks](/azure/templates/microsoft.network/virtualnetworks) : crée le réseau virtuel. 
- [Microsoft.Network/virtualNetworks/subnets](/azure/templates/microsoft.network/virtualnetworks/subnets) : crée le sous-réseau. 
- [Microsoft.DataMigration/services](/azure/templates/microsoft.datamigration/services) : Déploie une instance d’Azure Database Migration Service. 

Vous trouverez d’autres exemples de modèles Azure Database Migration Service dans la [galerie de modèles de démarrage rapide](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Datamigration&pageNumber=1&sort=Popular).


## <a name="deploy-the-template"></a>Déployer le modèle

1. Cliquez sur l’image ci-après pour vous connecter à Azure et ouvrir un modèle. Le modèle crée une instance d’Azure Database Migration Service. 

   [![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-azure-database-migration-simple-deploy%2fazuredeploy.json)

2. Sélectionnez ou entrez les valeurs suivantes.

    * **Abonnement**: Sélectionnez un abonnement Azure.
    * **Groupe de ressources** : sélectionnez un groupe de ressources existant dans la liste déroulante ou sélectionnez **Créer** pour créer un groupe de ressources. 
    * **Région** : emplacement de déploiement des ressources.
    * **Nom du service** : nom du nouveau service de migration.
    * **Emplacement** : emplacement du groupe de ressources ; laissez la valeur par défaut `[resourceGroup().location]`.
    * **Nom du réseau virtuel** : nom du nouveau réseau virtuel.
    * **Nom du sous-réseau** : nom du nouveau sous-réseau associé au réseau virtuel.



3. Sélectionnez **Revoir + créer**. Une fois que l’instance d’Azure Database Migration Service a été déployée avec succès, vous recevez une notification. 


Le portail Azure est utilisé pour déployer le modèle. Outre le portail Azure, vous pouvez également utiliser Azure PowerShell, l’interface Azure CLI et l’API REST. Pour découvrir d’autres méthodes de déploiement, consultez [Déployer des modèles](../azure-resource-manager/templates/deploy-powershell.md).

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

Vous pouvez utiliser Azure CLI pour vérifier les ressources déployées. 


```azurecli-interactive
echo "Enter the resource group where your SQL Server VM exists:" &&
read resourcegroupName &&
az resource list --resource-group $resourcegroupName 
```


## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, supprimez le groupe de ressources avec Azure CLI ou Azure PowerShell :

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

---

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir un tutoriel pas à pas vous guidant tout au long du processus de création d’un modèle, consultez :

> [!div class="nextstepaction"]
> [Tutoriel : Créer et déployer votre premier modèle Resource Manager](../azure-resource-manager/templates/template-tutorial-create-first-template.md)

Pour d’autres façons de déployer Azure Database Migration Service, consultez : 
- [Azure portal](quickstart-create-data-migration-service-portal.md)

Pour plus d’informations, consultez une [vue d’ensemble d’Azure Database Migration Service](dms-overview.md).