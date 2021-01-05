---
title: Baliser les ressources, les groupes de ressources et les abonnements pour l’organisation logique
description: Indique comment appliquer des étiquettes afin d'organiser des ressources Azure dédiées à la facturation et à la gestion.
ms.topic: conceptual
ms.date: 12/03/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: e47d3acf15ce5e4f5cb70444419b76beb21ae98b
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96558145"
---
# <a name="use-tags-to-organize-your-azure-resources-and-management-hierarchy"></a>Utiliser des étiquettes pour organiser vos ressources Azure et votre hiérarchie de gestion

Vous allez appliquer des étiquettes à vos ressources Azure, groupes de ressources et abonnements pour les organiser de façon logique dans une taxonomie. Chaque balise se compose d’une paire nom-valeur. Par exemple, vous pouvez appliquer le nom « Environnement » et la valeur « Production » à toutes les ressources en production.

Pour obtenir des recommandations sur la façon d’implémenter une stratégie d’étiquetage, consultez [Guides de décision concernant le nommage et l’étiquetage des ressources](/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json).

> [!IMPORTANT]
> Les noms des étiquettes ne respectent pas la casse pour les opérations. Une étiquette portant un nom, quelle que soit la casse, est mise à jour ou récupérée. Toutefois, il est possible que le fournisseur de ressources conserve la casse indiquée pour le nom de l’étiquette. Vous verrez cette casse dans les rapports sur les coûts.
> 
> Les valeurs des étiquettes respectent la casse.

[!INCLUDE [Handle personal data](../../../includes/gdpr-intro-sentence.md)]

## <a name="required-access"></a>Accès requis

Il existe deux façons d’obtenir l’accès requis aux ressources de balises.

- Vous pouvez disposer d’un accès en écriture au type de ressource **Microsoft.Resources/tags**. Cet accès vous permet d’étiqueter n’importe quelle ressource, même si vous n’avez pas accès à la ressource elle-même. Le rôle [Contributeur de balise](../../role-based-access-control/built-in-roles.md#tag-contributor) accorde cet accès. Actuellement, le rôle Contributeur d’étiquette ne peut pas appliquer d’étiquettes aux ressources ou groupes de ressources via le portail. Il peut appliquer des étiquettes aux abonnements via le portail. Il prend en charge toutes les opérations d’étiquettes via PowerShell et l’API REST.  

- Vous pouvez disposer d’un accès en écriture à la ressource elle-même. Le rôle [Contributeur](../../role-based-access-control/built-in-roles.md#contributor) accorde l’accès requis pour appliquer des étiquettes à n’importe quelle entité. Pour appliquer des étiquettes à un seul type de ressource, utilisez le rôle Contributeur correspondant à cette ressource. Par exemple, pour appliquer des étiquettes aux machines virtuelles, utilisez le rôle [Contributeur de machines virtuelles](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).

## <a name="powershell"></a>PowerShell

### <a name="apply-tags"></a>Appliquer des étiquettes

Azure PowerShell propose deux commandes pour appliquer des étiquettes : [New-AzTag](/powershell/module/az.resources/new-aztag) et [Update-AzTag](/powershell/module/az.resources/update-aztag). Vous devez disposer du module Az.Resources 1.12.0 ou version ultérieure. Vous pouvez vérifier votre version avec `Get-Module Az.Resources`. Vous pouvez installer ce module ou [installer Azure PowerShell](/powershell/azure/install-az-ps) 3.6.1 ou version ultérieure.

**New-AzTag** remplace toutes les étiquettes de la ressource, du groupe de ressources ou de l’abonnement. Lors de l’appel de la commande, transmettez l’ID de ressource de l’entité que vous souhaitez étiquetter.

L’exemple suivant applique un ensemble d’étiquettes à un compte de stockage :

```azurepowershell-interactive
$tags = @{"Dept"="Finance"; "Status"="Normal"}
$resource = Get-AzResource -Name demoStorage -ResourceGroup demoGroup
New-AzTag -ResourceId $resource.id -Tag $tags
```

Une fois la commande terminée, notez que la ressource a deux étiquettes.

```output
Properties :
        Name    Value
        ======  =======
        Dept    Finance
        Status  Normal
```

Si vous réexécutez la commande, mais cette fois avec des étiquettes différentes, vous remarquerez que les étiquettes précédentes sont supprimées.

```azurepowershell-interactive
$tags = @{"Team"="Compliance"; "Environment"="Production"}
New-AzTag -ResourceId $resource.id -Tag $tags
```

```output
Properties :
        Name         Value
        ===========  ==========
        Environment  Production
        Team         Compliance
```

Pour ajouter des étiquettes à une ressource qui a déjà des étiquettes, utilisez **Update-AzTag**. Définissez le paramètre **-Operation** sur **Fusion**.

```azurepowershell-interactive
$tags = @{"Dept"="Finance"; "Status"="Normal"}
Update-AzTag -ResourceId $resource.id -Tag $tags -Operation Merge
```

Notez que les deux nouvelles étiquettes ont été ajoutées aux deux étiquettes existantes.

```output
Properties :
        Name         Value
        ===========  ==========
        Status       Normal
        Dept         Finance
        Team         Compliance
        Environment  Production
```

Chaque nom d’étiquette ne peut avoir qu’une seule valeur. Si vous fournissez une nouvelle valeur pour une étiquette, l’ancienne valeur est remplacée même si vous utilisez l’opération de fusion. L’exemple suivant modifie l’étiquette d’état normal en vert.

```azurepowershell-interactive
$tags = @{"Status"="Green"}
Update-AzTag -ResourceId $resource.id -Tag $tags -Operation Merge
```

```output
Properties :
        Name         Value
        ===========  ==========
        Status       Green
        Dept         Finance
        Team         Compliance
        Environment  Production
```

Lorsque vous définissez le paramètre **-Operation** sur **Remplacer**, les étiquettes existantes sont remplacées par le nouvel ensemble d’étiquettes.

```azurepowershell-interactive
$tags = @{"Project"="ECommerce"; "CostCenter"="00123"; "Team"="Web"}
Update-AzTag -ResourceId $resource.id -Tag $tags -Operation Replace
```

Seules les nouvelles étiquettes restent sur la ressource.

```output
Properties :
        Name        Value
        ==========  =========
        CostCenter  00123
        Team        Web
        Project     ECommerce
```

Les mêmes commandes fonctionnent également avec les groupes de ressources ou les abonnements. Vous transmettez l’identificateur pour le groupe de ressources ou l’abonnement que vous souhaitez baliser.

Pour ajouter un nouvel ensemble d’étiquettes à un groupe de ressources, utilisez :

```azurepowershell-interactive
$tags = @{"Dept"="Finance"; "Status"="Normal"}
$resourceGroup = Get-AzResourceGroup -Name demoGroup
New-AzTag -ResourceId $resourceGroup.ResourceId -tag $tags
```

Pour mettre à jour les étiquettes d’un groupe de ressources, utilisez :

```azurepowershell-interactive
$tags = @{"CostCenter"="00123"; "Environment"="Production"}
$resourceGroup = Get-AzResourceGroup -Name demoGroup
Update-AzTag -ResourceId $resourceGroup.ResourceId -Tag $tags -Operation Merge
```

Pour ajouter un nouvel ensemble d’étiquettes à un abonnement, utilisez :

```azurepowershell-interactive
$tags = @{"CostCenter"="00123"; "Environment"="Dev"}
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
New-AzTag -ResourceId "/subscriptions/$subscription" -Tag $tags
```

Pour mettre à jour les étiquettes d’un abonnement, utilisez :

```azurepowershell-interactive
$tags = @{"Team"="Web Apps"}
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
Update-AzTag -ResourceId "/subscriptions/$subscription" -Tag $tags -Operation Merge
```

Vous pouvez avoir plusieurs ressources portant le même nom dans un groupe de ressources. Dans ce cas, vous pouvez définir chaque ressource à l’aide des commandes suivantes :

```azurepowershell-interactive
$resource = Get-AzResource -ResourceName sqlDatabase1 -ResourceGroupName examplegroup
$resource | ForEach-Object { Update-AzTag -Tag @{ "Dept"="IT"; "Environment"="Test" } -ResourceId $_.ResourceId -Operation Merge }
```

### <a name="list-tags"></a>Répertorier les balises

Pour obtenir les étiquettes d’une ressource, d’un groupe de ressources ou d’un abonnement, utilisez la commande [Get-AzTag](/powershell/module/az.resources/get-aztag) et transmettez l’ID de ressource de l’entité.

Pour afficher les étiquettes d’une ressource, utilisez :

```azurepowershell-interactive
$resource = Get-AzResource -Name demoStorage -ResourceGroup demoGroup
Get-AzTag -ResourceId $resource.id
```

Pour voir les étiquettes d’un groupe de ressources, utilisez :

```azurepowershell-interactive
$resourceGroup = Get-AzResourceGroup -Name demoGroup
Get-AzTag -ResourceId $resourceGroup.ResourceId
```

Pour afficher les étiquettes d’un abonnement, utilisez :

```azurepowershell-interactive
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
Get-AzTag -ResourceId "/subscriptions/$subscription"
```

### <a name="list-by-tag"></a>Liste par étiquette

Pour obtenir des ressources qui ont un nom et une valeur d’étiquette spécifiques, utilisez :

```azurepowershell-interactive
(Get-AzResource -Tag @{ "CostCenter"="00123"}).Name
```

Pour obtenir des ressources qui ont un nom et une valeur d’étiquette spécifiques avec une valeur d’étiquette, utilisez :

```azurepowershell-interactive
(Get-AzResource -TagName "Dept").Name
```

Pour obtenir des groupes de ressources qui ont un nom et une valeur d’étiquette spécifiques, utilisez :

```azurepowershell-interactive
(Get-AzResourceGroup -Tag @{ "CostCenter"="00123" }).ResourceGroupName
```

### <a name="remove-tags"></a>Supprimer des étiquettes

Pour supprimer des étiquette spécifiques, utilisez **Update-AzTag** et définissez **-Operation** sur **supprimer**. Transmettez les étiquettes que vous souhaitez supprimer.

```azurepowershell-interactive
$removeTags = @{"Project"="ECommerce"; "Team"="Web"}
Update-AzTag -ResourceId $resource.id -Tag $removeTags -Operation Delete
```

Les étiquettes spécifiées sont supprimées.

```output
Properties :
        Name        Value
        ==========  =====
        CostCenter  00123
```

Pour supprimer toutes les étiquettes, utilisez la commande [Remove-AzTag](/powershell/module/az.resources/remove-aztag).

```azurepowershell-interactive
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
Remove-AzTag -ResourceId "/subscriptions/$subscription"
```

## <a name="azure-cli"></a>Azure CLI

### <a name="apply-tags"></a>Appliquer des étiquettes

Azure CLI propose deux commandes pour appliquer des étiquettes : [az tag create](/cli/azure/tag#az_tag_create) et [az tag update](/cli/azure/tag#az_tag_update). Vous devez disposer d’Azure CLI 2.10.0 ou version ultérieure. Vous pouvez vérifier votre version avec `az version`. Pour la mise à jour ou l’installation, consultez [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli).

La commande **az tag create** remplace toutes les étiquettes de la ressource, du groupe de ressources ou de l’abonnement. Lors de l’appel de la commande, transmettez l’ID de ressource de l’entité que vous souhaitez étiquetter.

L’exemple suivant applique un ensemble d’étiquettes à un compte de stockage :

```azurecli-interactive
resource=$(az resource show -g demoGroup -n demoStorage --resource-type Microsoft.Storage/storageAccounts --query "id" --output tsv)
az tag create --resource-id $resource --tags Dept=Finance Status=Normal
```

Une fois la commande terminée, notez que la ressource a deux étiquettes.

```output
"properties": {
  "tags": {
    "Dept": "Finance",
    "Status": "Normal"
  }
},
```

Si vous réexécutez la commande, mais cette fois avec des étiquettes différentes, vous remarquerez que les étiquettes précédentes sont supprimées.

```azurecli-interactive
az tag create --resource-id $resource --tags Team=Compliance Environment=Production
```

```output
"properties": {
  "tags": {
    "Environment": "Production",
    "Team": "Compliance"
  }
},
```

Pour ajouter des étiquettes à une ressource qui a déjà des étiquettes, utilisez `az tag update`. Définissez le paramètre `--operation` sur `Merge`.

```azurecli-interactive
az tag update --resource-id $resource --operation Merge --tags Dept=Finance Status=Normal
```

Notez que les deux nouvelles étiquettes ont été ajoutées aux deux étiquettes existantes.

```output
"properties": {
  "tags": {
    "Dept": "Finance",
    "Environment": "Production",
    "Status": "Normal",
    "Team": "Compliance"
  }
},
```

Chaque nom d’étiquette ne peut avoir qu’une seule valeur. Si vous fournissez une nouvelle valeur pour une étiquette, l’ancienne valeur est remplacée même si vous utilisez l’opération de fusion. L’exemple suivant modifie l’étiquette d’état normal en vert.

```azurecli-interactive
az tag update --resource-id $resource --operation Merge --tags Status=Green
```

```output
"properties": {
  "tags": {
    "Dept": "Finance",
    "Environment": "Production",
    "Status": "Green",
    "Team": "Compliance"
  }
},
```

Lorsque vous définissez le paramètre `--operation` sur `Replace`, les étiquettes existantes sont remplacées par le nouvel ensemble d’étiquettes.

```azurecli-interactive
az tag update --resource-id $resource --operation Replace --tags Project=ECommerce CostCenter=00123 Team=Web
```

Seules les nouvelles étiquettes restent sur la ressource.

```output
"properties": {
  "tags": {
    "CostCenter": "00123",
    "Project": "ECommerce",
    "Team": "Web"
  }
},
```

Les mêmes commandes fonctionnent également avec les groupes de ressources ou les abonnements. Vous transmettez l’identificateur pour le groupe de ressources ou l’abonnement que vous souhaitez baliser.

Pour ajouter un nouvel ensemble d’étiquettes à un groupe de ressources, utilisez :

```azurecli-interactive
group=$(az group show -n demoGroup --query id --output tsv)
az tag create --resource-id $group --tags Dept=Finance Status=Normal
```

Pour mettre à jour les étiquettes d’un groupe de ressources, utilisez :

```azurecli-interactive
az tag update --resource-id $group --operation Merge --tags CostCenter=00123 Environment=Production
```

Pour ajouter un nouvel ensemble d’étiquettes à un abonnement, utilisez :

```azurecli-interactive
sub=$(az account show --subscription "Demo Subscription" --query id --output tsv)
az tag create --resource-id /subscriptions/$sub --tags CostCenter=00123 Environment=Dev
```

Pour mettre à jour les étiquettes d’un abonnement, utilisez :

```azurecli-interactive
az tag update --resource-id /subscriptions/$sub --operation Merge --tags Team="Web Apps"
```

### <a name="list-tags"></a>Répertorier les balises

Pour obtenir les étiquettes d’une ressource, d’un groupe de ressources ou d’un abonnement, utilisez la commande [az tag list](/cli/azure/tag#az_tag_list) et transmettez l’ID de ressource de l’entité.

Pour afficher les étiquettes d’une ressource, utilisez :

```azurecli-interactive
resource=$(az resource show -g demoGroup -n demoStorage --resource-type Microsoft.Storage/storageAccounts --query "id" --output tsv)
az tag list --resource-id $resource
```

Pour voir les étiquettes d’un groupe de ressources, utilisez :

```azurecli-interactive
group=$(az group show -n demoGroup --query id --output tsv)
az tag list --resource-id $group
```

Pour afficher les étiquettes d’un abonnement, utilisez :

```azurecli-interactive
sub=$(az account show --subscription "Demo Subscription" --query id --output tsv)
az tag list --resource-id /subscriptions/$sub
```

### <a name="list-by-tag"></a>Liste par étiquette

Pour obtenir des ressources qui ont un nom et une valeur d’étiquette spécifiques, utilisez :

```azurecli-interactive
az resource list --tag CostCenter=00123 --query [].name
```

Pour obtenir des ressources qui ont un nom et une valeur d’étiquette spécifiques avec une valeur d’étiquette, utilisez :

```azurecli-interactive
az resource list --tag Team --query [].name
```

Pour obtenir des groupes de ressources qui ont un nom et une valeur d’étiquette spécifiques, utilisez :

```azurecli-interactive
az group list --tag Dept=Finance
```

### <a name="remove-tags"></a>Supprimer des étiquettes

Pour supprimer des étiquette spécifiques, utilisez `az tag update` et définissez `--operation` sur `Delete`. Transmettez les étiquettes que vous souhaitez supprimer.

```azurecli-interactive
az tag update --resource-id $resource --operation Delete --tags Project=ECommerce Team=Web
```

Les étiquettes spécifiées sont supprimées.

```output
"properties": {
  "tags": {
    "CostCenter": "00123"
  }
},
```

Pour supprimer toutes les étiquettes, utilisez la commande [az tag delete](/cli/azure/tag#az_tag_delete).

```azurecli-interactive
az tag delete --resource-id $resource
```

### <a name="handling-spaces"></a>Gestion des espaces

Si les noms ou les valeurs de vos étiquettes comportent des espaces, mettez-les entre guillemets.

```azurecli-interactive
az tag update --resource-id $group --operation Merge --tags "Cost Center"=Finance-1222 Location="West US"
```

## <a name="templates"></a>Modèles

Vous pouvez baliser les ressources, les groupes de ressources et les abonnements pendant le déploiement avec un modèle Resource Manager.

### <a name="apply-values"></a>Appliquer les valeurs

L’exemple suivant déploie un compte de stockage avec trois étiquettes. Deux des étiquettes (`Dept` et `Environment`) sont définies sur des valeurs littérales. Une étiquette (`LastDeployed`) a pour valeur un paramètre dont la valeur par défaut est la date actuelle.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "utcShort": {
            "type": "string",
            "defaultValue": "[utcNow('d')]"
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[parameters('location')]",
            "tags": {
                "Dept": "Finance",
                "Environment": "Production",
                "LastDeployed": "[parameters('utcShort')]"
            },
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }
    ]
}
```

### <a name="apply-an-object"></a>Appliquer un objet

Vous pouvez définir un paramètre d’objet qui stocke plusieurs balises et appliquer cet objet à l’élément de balise. Cette approche offre plus de souplesse que l’exemple précédent, car l’objet peut avoir des propriétés différentes. Chaque propriété de l’objet devient une balise distincte pour la ressource. L’exemple suivant illustre un paramètre nommé `tagValues` appliqué à l’élément de balise.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        },
        "tagValues": {
            "type": "object",
            "defaultValue": {
                "Dept": "Finance",
                "Environment": "Production"
            }
        }
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[parameters('location')]",
            "tags": "[parameters('tagValues')]",
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }
    ]
}
```

### <a name="apply-a-json-string"></a>Analyser une chaîne JSON

Pour stocker plusieurs valeurs dans une seule balise, appliquez une chaîne JSON qui représente les valeurs. La chaîne JSON complète est stockée sous la forme d’une balise ne pouvant pas dépasser 256 caractères. L’exemple illustre une balise unique nommée `CostCenter` qui contient plusieurs valeurs d’une chaîne JSON :  

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[parameters('location')]",
            "tags": {
                "CostCenter": "{\"Dept\":\"Finance\",\"Environment\":\"Production\"}"
            },
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }
    ]
}
```

### <a name="apply-tags-from-resource-group"></a>Appliquer des balises à partir d’un groupe de ressources

Pour appliquer des balises d’un groupe de ressources à une ressource, utilisez la fonction [resourceGroup()](../templates/template-functions-resource.md#resourcegroup). Lors de l’obtention de la valeur de balise, utilisez la syntaxe `tags[tag-name]` au lieu de la syntaxe `tags.tag-name`, car certains caractères ne sont pas correctement analysés dans la notation par points.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[parameters('location')]",
            "tags": {
                "Dept": "[resourceGroup().tags['Dept']]",
                "Environment": "[resourceGroup().tags['Environment']]"
            },
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }
    ]
}
```

### <a name="apply-tags-to-resource-groups-or-subscriptions"></a>Appliquer des étiquettes à des groupes de ressources ou à des abonnements

Vous pouvez ajouter des étiquettes à un groupe de ressources ou à un abonnement en déployant le type de ressource **Microsoft.Resources/tags**. Les étiquettes sont appliquées au groupe de ressources cible ou à l’abonnement pour le déploiement. Chaque fois que vous déployez le modèle, vous remplacez les étiquettes précédemment appliquées.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "tagName": {
            "type": "string",
            "defaultValue": "TeamName"
        },
        "tagValue": {
            "type": "string",
            "defaultValue": "AppTeam1"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Resources/tags",
            "name": "default",
            "apiVersion": "2019-10-01",
            "dependsOn": [],
            "properties": {
                "tags": {
                    "[parameters('tagName')]": "[parameters('tagValue')]"
                }
            }
        }
    ]
}
```

Pour appliquer les étiquettes à un groupe de ressources, utilisez PowerShell ou Azure CLI. Déployez sur le groupe de ressources que vous souhaitez baliser.

```azurepowershell-interactive
New-AzResourceGroupDeployment -ResourceGroupName exampleGroup -TemplateFile https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

```azurecli-interactive
az deployment group create --resource-group exampleGroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

Pour appliquer les étiquettes à un abonnement, utilisez PowerShell ou Azure CLI. Déployez sur l’abonnement que vous souhaitez baliser.

```azurepowershell-interactive
New-AzSubscriptionDeployment -name tagresourcegroup -Location westus2 -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

```azurecli-interactive
az deployment sub create --name tagresourcegroup --location westus2 --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

Pour plus d’informations sur les déploiements dans des abonnements, consultez [Créer des groupes de ressources et des ressources au niveau de l’abonnement](../templates/deploy-to-subscription.md).

Le modèle suivant ajoute les étiquettes depuis un objet vers un groupe de ressources ou un abonnement.

```json
"$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "tags": {
            "type": "object",
            "defaultValue": {
                "TeamName": "AppTeam1",
                "Dept": "Finance",
                "Environment": "Production"
            }
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Resources/tags",
            "name": "default",
            "apiVersion": "2019-10-01",
            "dependsOn": [],
            "properties": {
                "tags": "[parameters('tags')]"
            }
        }
    ]
}
```

## <a name="portal"></a>Portail

[!INCLUDE [resource-manager-tag-resource](../../../includes/resource-manager-tag-resources.md)]

## <a name="rest-api"></a>API REST

Pour travailler avec des étiquettes par le biais de l’API REST Azure, utilisez :

* [Étiquettes - Créer ou mettre à jour au niveau de l’étendue](/rest/api/resources/tags/createorupdateatscope) (opération PUT)
* [Balises - Mettre à jour au niveau de l’étendue](/rest/api/resources/tags/updateatscope) (opération PATCH)
* [Étiquettes - Obtenir au niveau de l’étendue](/rest/api/resources/tags/getatscope) (opération GET)
* [Étiquettes - Supprimer au niveau de l’étendue](/rest/api/resources/tags/deleteatscope) (opération DELETE)

## <a name="inherit-tags"></a>Hériter des étiquettes

Les balises appliquées au groupe de ressources ou à l’abonnement ne sont pas héritées par les ressources. Pour appliquer des balises d’un abonnement ou d’un groupe de ressources aux ressources, consultez [Stratégies Azure - Étiquettes](tag-policies.md).

## <a name="tags-and-billing"></a>Étiquettes et facturation

Vous pouvez utiliser des étiquettes pour regrouper vos données de facturation. Par exemple, si vous exécutez plusieurs machines virtuelles pour différentes organisations, vous pouvez recourir aux étiquettes afin de regrouper l'utilisation par centre de coûts. Vous pouvez également utiliser des étiquettes pour catégoriser les coûts par environnement d'exécution ; par exemple, l'utilisation de la facturation pour les machines virtuelles en cours d'exécution dans l'environnement de production.

Vous pouvez récupérer des informations sur les étiquettes par le biais des [API Resource Usage et RateCard](../../cost-management-billing/manage/usage-rate-card-overview.md) ou du fichier de valeurs séparées par des virgules (CSV). Vous téléchargez le fichier d’utilisation à partir du portail Azure. Pour plus d’informations, consultez [Télécharger et consulter votre facture Azure et vos données d’utilisation quotidienne](../../cost-management-billing/manage/download-azure-invoice-daily-usage-date.md). Lorsque vous téléchargez le fichier d’utilisation depuis le Centre des comptes Azure, sélectionnez **Version 2**. Pour les services qui prennent en charge les étiquettes avec la facturation, les étiquettes s'affichent dans la colonne **Étiquettes**.

Pour plus d’informations sur les opérations de l’API REST, consultez [Informations de référence sur l’API REST Azure Billing](/rest/api/billing/).

## <a name="limitations"></a>Limites

Les limites suivantes s’appliquent aux balises :

* Les types de ressources ne prennent pas tous en charge les étiquettes. Pour déterminer si vous pouvez appliquer une étiquette à un type de ressource, consultez [Prise en charge des étiquettes pour les ressources Azure](tag-support.md).
* Chaque ressource, groupe de ressources et abonnement peuvent inclure un maximum de 50 paires nom/valeur d’étiquette. Si vous devez appliquer plus de balises que le nombre maximal autorisé, utilisez une chaîne JSON comme valeur de balise. La chaîne JSON peut contenir plusieurs valeurs appliquées à un seul nom de balise. Un groupe de ressources ou un abonnement peut contenir de nombreuses ressources qui ont chacune 50 paires nom/valeur d’étiquette.
* Le nom de balise est limité à 512 caractères, et la valeur de balise à 256 caractères. Pour les comptes de stockage, le nom de balise est limité à 128 caractères, et la valeur de balise à 256 caractères.
* Les balises ne peuvent pas être appliquées à des ressources classiques comme les Services cloud.
* Les noms de balise ne peuvent pas contenir ces caractères : `<`, `>`, `%`, `&`, `\`, `?`, `/`

   > [!NOTE]
   > Actuellement, les zones Azure DNS et les services Traffic Manager n’autorisent pas non plus l’utilisation des espaces dans la balise.
   >
   > Azure Front Door ne prend pas en charge l’utilisation de `#` dans le nom de la balise.
   >
   > Azure Automation et Azure CDN ne gèrent que 15 balises sur les ressources.

## <a name="next-steps"></a>Étapes suivantes

* Les types de ressources ne prennent pas tous en charge les étiquettes. Pour déterminer si vous pouvez appliquer une étiquette à un type de ressource, consultez [Prise en charge des étiquettes pour les ressources Azure](tag-support.md).
* Pour obtenir des recommandations sur la façon d’implémenter une stratégie d’étiquetage, consultez [Guides de décision concernant le nommage et l’étiquetage des ressources](/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json).
