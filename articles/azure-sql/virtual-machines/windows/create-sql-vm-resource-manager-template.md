---
title: Créer une machine virtuelle SQL Server à l’aide d’un modèle ARM
description: Découvrez comment créer une machine virtuelle SQL Server sur Azure à l’aide d’un modèle Azure Resource Manager (modèle ARM).
author: MashaMSFT
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: mathoma
ms.date: 06/29/2020
ms.service: virtual-machines-sql
ms.subservice: deployment
ms.openlocfilehash: a20cb27ac91a0b01ed9cc3a5ac4c5c57f90ceda1
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359640"
---
# <a name="quickstart-create-sql-server-vm-using-an-arm-template"></a>Démarrage rapide : Créer une machine virtuelle SQL Server à l’aide d’un modèle ARM

Utilisez ce modèle Azure Resource Manager (modèle ARM) pour déployer une machine virtuelle SQL Server sur Azure. 

[!INCLUDE [About Azure Resource Manager](../../../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-sql-vm-new-storage%2fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Le modèle Resource Manager de machine virtuelle SQL Server nécessite les éléments suivants :

- La version la plus récente de l’[interface de ligne de commande Azure](/cli/azure/install-azure-cli) et/ou de [PowerShell](/powershell/scripting/install/installing-powershell). 
- Un [groupe de ressources](../../../azure-resource-manager/management/manage-resource-groups-portal.md#create-resource-groups) préconfiguré avec un [réseau virtuel](../../../virtual-network/quick-create-portal.md) et un [sous-réseau](../../../virtual-network/virtual-network-manage-subnet.md#add-a-subnet) préparés.
- Un abonnement Azure. Si vous n’en avez pas, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.


## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-sql-vm-new-storage/).

:::code language="json" source="~/quickstart-templates/101-sql-vm-new-storage/azuredeploy.json":::

Cinq ressources Azure sont définies dans le modèle : 

- [Microsoft.Network/publicIpAddresses](/azure/templates/microsoft.network/publicipaddresses) : Crée une adresse IP publique. 
- [Microsoft.Network/networkSecurityGroups](/azure/templates/microsoft.network/networksecuritygroups): Crée un groupe de sécurité réseau. 
- [Microsoft.Network/networkInterfaces](/azure/templates/microsoft.network/networkinterfaces) : configure l’interface réseau. 
- [Microsoft.Compute/virtualMachines](/azure/templates/microsoft.compute/virtualmachines) : crée une machine virtuelle dans Azure. 
- [Microsoft.SqlVirtualMachine/SqlVirtualMachines](/azure/templates/microsoft.sqlvirtualmachine/sqlvirtualmachines) : inscrit la machine virtuelle auprès de l’extension SQL IaaS Agent. 

Vous trouverez d’autres modèles de machine virtuelle SQL Server sur Azure dans la [galerie de modèles de démarrage rapide](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Sqlvirtualmachine&pageNumber=1&sort=Popular).


## <a name="deploy-the-template"></a>Déployer le modèle

1. Cliquez sur l’image ci-après pour vous connecter à Azure et ouvrir un modèle. Le modèle crée une machine virtuelle sur laquelle est installée la version de SQL Server prévue, et qui est inscrite auprès de l’extension SQL IaaS Agent. 

   [![Déployer sur Azure](../../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-sql-vm-new-storage%2fazuredeploy.json)

2. Sélectionnez ou entrez les valeurs suivantes.

    * **Abonnement**: Sélectionnez un abonnement Azure.
    * **Groupe de ressources** : groupe de ressources préparé pour votre machine virtuelle SQL Server. 
    * **Région** : Sélectionnez une région.  Par exemple, **USA Centre**.
    * **Nom de la machine virtuelle** : entrez un nom pour la machine virtuelle SQL Server. 
    * **Taille de machine virtuelle** : choisissez la taille appropriée pour votre machine virtuelle dans la liste déroulante.
    * **Nom de réseau virtuel existant** : entrez le nom du réseau virtuel préparé pour votre machine virtuelle SQL Server. 
    * **Groupe de ressources du réseau virtuel existant** : entrez le groupe de ressources dans lequel votre réseau virtuel a été préparé. 
    * **Nom du sous-réseau existant** : nom de votre sous-réseau préparé. 
    * **Offre de l’image** : choisissez l’image SQL Server et Windows Server qui correspond le mieux à vos besoins métier. 
    * **Référence SKU SQL** : choisissez l’édition de la référence SKU SQL Server qui correspond le mieux à vos besoins métier. 
    * **Nom d’utilisateur administrateur** : nom d’utilisateur de l’administrateur de la machine virtuelle. 
    * **Mot de passe administrateur** : mot de passe utilisé par le compte administrateur de la machine virtuelle. 
    * **Type de charge de travail de stockage** :  type de stockage pour la charge de travail qui correspond le mieux à votre entreprise. 
    * **Nombre de disques de données SQL** :  nombre de disques que SQL Server utilise pour les fichiers de données.  
    * **Chemin d’accès aux données** :  chemin des fichiers de données SQL Server. 
    * **Nombre de disques de journaux SQL** :  nombre de disques que SQL Server utilise pour les fichiers journaux. 
    * **Chemin d’accès au journal** :  chemin des fichiers journaux SQL Server. 
    * **Emplacement** :  emplacement de toutes les ressources ; cette valeur doit rester la valeur par défaut `[resourceGroup().location]`. 

3. Sélectionnez **Revoir + créer**. Une fois que la machine virtuelle SQL Server a été déployée avec succès, vous recevez une notification.

Le portail Azure est utilisé pour déployer le modèle. Outre le portail Azure, vous pouvez utiliser Azure PowerShell, l’interface de ligne de commande Azure et l’API REST. Pour découvrir d’autres méthodes de déploiement, consultez [Déployer des modèles](../../../azure-resource-manager/templates/deploy-powershell.md).

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
> [Tutoriel : Créer et déployer votre premier modèle Resource Manager](../../../azure-resource-manager/templates/template-tutorial-create-first-template.md)

Pour d’autres façons de déployer une machine virtuelle SQL Server, consultez : 
- [Azure portal](create-sql-vm-portal.md)
- [PowerShell](create-sql-vm-powershell.md)

Pour plus d’informations, consultez [une vue d’ensemble des machines virtuelles SQL Server sur Azure](sql-server-on-azure-vm-iaas-what-is-overview.md).