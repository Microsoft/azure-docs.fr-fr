---
title: Créer un coffre de clés Azure et une stratégie d’accès de coffre à l’aide d’un modèle Azure Resource Manager (ARM)
description: Cet article explique comment créer des coffres de clés Azure et des stratégies d’accès à l’aide d’un modèle Azure Resource Manager.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 10/5/2020
ms.author: mbaldwin
ms.openlocfilehash: d6f3d2051790ba02f5e4ea1687830ccbdf6dcf21
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93285136"
---
# <a name="how-to-create-an-azure-key-vault-and-vault-access-policy-by-using-a-resource-manager-template"></a>Comment créer un coffre de clés Azure et une stratégie d’accès à l’aide d’un modèle Resource Manager

[Azure Key Vault](../general/overview.md) est un service cloud qui fournit un magasin sécurisé pour des secrets tels que des clés, des mots de passe et des certificats. Ce guide décrit le processus de déploiement d’un modèle ARM (Azure Resource Manager) en vue de créer un coffre de clés.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

## <a name="prerequisites"></a>Prérequis

Pour accomplir les étapes décrites dans cet article :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.


## <a name="create-a-key-vault-resource-manager-template"></a>Créer un modèle Resource Manager de coffre de clés

Le modèle suivant montre comment créer un coffre de clés de façon très simple. Certaines valeurs sont spécifiées dans le modèle.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "keyVaultName": {
      "type": "string",
      "metadata": {
        "description": "Specifies the name of the key vault."
      }
    },
    "skuName": {
      "type": "string",
      "defaultValue": "Standard",
      "allowedValues": [
        "Standard",
        "Premium"
      ],
      "metadata": {
        "description": "Specifies whether the key vault is a standard vault or a premium vault."
      }
    }
   },
  "resources": [
    {
      "type": "Microsoft.KeyVault/vaults",
      "apiVersion": "2019-09-01",
      "name": "[parameters('keyVaultName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "enabledForDeployment": "false",
        "enabledForDiskEncryption": "false",
        "enabledForTemplateDeployment": "false",
        "tenantId": "[subscription().tenantId]",
        "accessPolicies": [],
        "sku": {
          "name": "[parameters('skuName')]",
          "family": "A"
        },
        "networkAcls": {
          "defaultAction": "Allow",
          "bypass": "AzureServices"
        }
      }
    }
  ]
}

```

Pour plus d’informations sur les paramètres du modèle de coffre de clés, consultez [Référence sur le modèle ARM de coffre de clés](/azure/templates/microsoft.keyvault/vaults).

> [!IMPORTANT]
> Si un modèle est redéployé, toutes les stratégies d’accès existantes dans le coffre de clés seront remplacées. Nous vous recommandons de renseigner la propriété `accessPolicies` avec les stratégies d’accès existantes afin d’éviter de perdre l’accès au coffre de clés. 

## <a name="add-an-access-policy-to-a-key-vault-resource-manager-template"></a>Ajouter une stratégie d’accès à un modèle Resource Manager de coffre de clés

Vous pouvez déployer des stratégies d’accès à un coffre de clés existant sans redéploiement de l’intégralité du modèle de coffre de clés. Le modèle suivant montre comment créer des stratégies d’accès :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "keyVaultName": {
      "type": "string",
      "metadata": {
        "description": "Specifies the name of the key vault."
      }
    },
    "objectId": {
      "type": "string",
      "metadata": {
        "description": "Specifies the object ID of a user, service principal or security group in the Azure Active Directory tenant for the vault. The object ID must be unique for the list of access policies. Get it by using Get-AzADUser or Get-AzADServicePrincipal cmdlets."
      }
    },
    "keysPermissions": {
      "type": "array",
      "defaultValue": [
        "list"
      ],
      "metadata": {
        "description": "Specifies the permissions to keys in the vault. Valid values are: all, encrypt, decrypt, wrapKey, unwrapKey, sign, verify, get, list, create, update, import, delete, backup, restore, recover, and purge."
      }
    },
    "secretsPermissions": {
      "type": "array",
      "defaultValue": [
        "list"
      ],
      "metadata": {
        "description": "Specifies the permissions to secrets in the vault. Valid values are: all, get, list, set, delete, backup, restore, recover, and purge."
      }
    },
    "certificatePermissions": {
      "type": "array",
      "defaultValue": [
        "list"
      ],
      "metadata": {
        "description": "Specifies the permissions to certificates in the vault. Valid values are: all,  create, delete, update, deleteissuers, get, getissuers, import, list, listissuers, managecontacts, manageissuers,  recover, backup, restore, setissuers, and purge."
      }
    },
  "resources": [
     {
      "type": "Microsoft.KeyVault/vaults/accessPolicies",
      "name": "[concat(parameters('keyVaultName'), '/add')]",
      "apiVersion": "2019-09-01",
      "properties": {
        "accessPolicies": [
          {
            "tenantId": "[subscription().tenantId]",
            "objectId": "[parameters('objectId')]",
            "permissions": {
              "keys": "[parameters('keysPermissions')]",
              "secrets": "[parameters('secretsPermissions')]",
              "certificates": [parameters('certificatesPermissions')]
            }
          }
        ]
      }
    }
  ]
}

```
Pour plus d’informations sur les paramètres du modèle de coffre de clés, consultez notre page de [référence sur le modèle ARM de coffre de clés](/azure/templates/microsoft.keyvault/vaults/accesspolicies).

## <a name="more-key-vault-resource-manager-templates"></a>Modèles Resource Manager de coffre de clés supplémentaires

Il existe d’autres modèles Resource Manager pour les objets Key Vault :

| Secrets | Keys | Certificats |
|--|--|--|
|<ul><li>[Démarrage rapide](../secrets/quick-create-template.md)<li>[Référence](/azure/templates/microsoft.keyvault/vaults/secrets)|N/A|N/A|

Des modèles de coffre de clés supplémentaires sont disponibles ici : [Référence Resource Manager relative aux coffres de clés](/azure/templates/microsoft.keyvault/allversions).

## <a name="deploy-the-templates"></a>Déployer les modèles

Vous pouvez utiliser le portail Azure pour déployer les modèles précédents à l’aide de l’option **Build your own template in editor** (Créer votre modèle dans l’éditeur), comme décrit ici : [Déployer des ressources à partir d’un modèle personnalisé](../../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template).

Vous pouvez également enregistrer les modèles précédents dans des fichiers et utiliser ces commandes :  [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) et [az group deployment create](/cli/azure/group/deployment#az-group-deployment-create) :

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateFile key-vault-template.json
```

```azurecli
az group deployment create --resource-group ExampleGroup --template-file key-vault-template.json
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous prévoyez d’utiliser d’autres démarrages rapides et tutoriels, il peut être utile de conserver ces ressources. Lorsque vous n’avez plus besoin des ressources, supprimez le groupe de ressources. Si vous supprimez le groupe, le coffre de clés et les ressources associées sont également supprimés. Pour supprimer le groupe de ressources avec Azure CLI ou Azure PowerShell, procédez comme suit :

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

## <a name="resources"></a>Ressources

- Consultez une [présentation d’Azure Key Vault](../general/overview.md).
- Découvrez [Azure Resource Manager](../../azure-resource-manager/management/overview.md).
- Examinez les [bonnes pratiques relatives à Azure Key Vault](../general/best-practices.md).

## <a name="next-steps"></a>Étapes suivantes

- [Sécuriser l’accès à un coffre de clés](secure-your-key-vault.md)
- [S’authentifier auprès d’un coffre de clés](authentication.md)
- [Guide du développeur de coffre de clés Azure](developers-guide.md)
