---
title: Réparer un compte Azure Automanage endommagé
description: Si vous avez récemment déplacé un abonnement qui contient un compte Automanage vers un nouveau locataire, vous devez le reconfigurer. Cet article vous explique comment procéder.
author: asinn826
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: conceptual
ms.date: 11/05/2020
ms.author: alsin
ms.openlocfilehash: 4694fa679c7bbff309a0452219ff39bacf2488c4
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183700"
---
# <a name="repair-an-automanage-account"></a>Réparer un compte Automanage
Votre [compte Azure Automanage](./automanage-virtual-machines.md#automanage-account) est l’identité ou le contexte de sécurité où les opérations automatisées se produisent. Si vous avez récemment déplacé un abonnement qui contient un compte Automanage vers un nouveau locataire, vous devez reconfigurer le compte. Pour le reconfigurer, vous devez réinitialiser le type d’identité et attribuer les rôles appropriés pour le compte.

## <a name="step-1-reset-the-automanage-account-identity-type"></a>Étape 1 : Réinitialiser le type d’identité du compte Automanage
Réinitialisez le type d’identité du compte Automanage en utilisant le modèle ARM (Azure Resource Manager) suivant. Enregistrez le fichier localement sous le nom armdeploy.json ou un nom similaire. Notez le nom et l’emplacement de votre compte Automanage, car il s’agit de paramètres requis dans le modèle ARM.

1. Créez un déploiement Resource Manager à l’aide du modèle suivant. Utiliser `identityType = None`.
    * Vous pouvez créer le déploiement dans Azure CLI à l’aide de `az deployment sub create`. Pour plus d’informations, consultez [az deployment sub](/cli/azure/deployment/sub).
    * Vous pouvez créer le déploiement dans PowerShell à l’aide du module `New-AzDeployment`. Pour plus d’informations, consultez [New-AzDeployment](/powershell/module/az.resources/new-azdeployment).

1. Exécutez à nouveau le même modèle ARM avec `identityType = SystemAssigned`.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "accountName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "identityType": {
            "type": "string",
            "allowedValues": [ "None", "SystemAssigned" ]
        }
    },
    "resources": [
        {
            "apiVersion": "2020-06-30-preview",
            "name": "[parameters('accountName')]",
            "location": "[parameters('location')]",
            "type": "Microsoft.Automanage/accounts",
            "identity": {
                "type": "[parameters('identityType')]"
            }
        }
    ]
}

```

## <a name="step-2-assign-appropriate-roles-for-the-automanage-account"></a>Étape 2 : Attribuer les rôles appropriés pour le compte Automanage
Le compte Automanage nécessite les rôles Contributeur et Contributeur de stratégie de ressource sur l’abonnement qui contient les machines virtuelles gérées par Automanage. Vous pouvez attribuer ces rôles à l’aide du portail Azure, de modèles ARM ou d’Azure CLI.

Si vous utilisez un modèle ARM ou Azure CLI, vous avez besoin de l’ID du principal (également appelé ID d’objet) de votre compte Automanage. (Vous n’avez pas besoin de l’ID si vous utilisez le portail Azure.) Vous pouvez trouver cet ID à l’aide des méthodes suivantes :

- [Azure CLI](/cli/azure/ad/sp) : Utilisez la commande `az ad sp list --display-name <name of your Automanage Account>`.

- Portail Azure : Accédez à **Azure Active Directory** et recherchez votre compte Automanage par son nom. Sous **Applications d’entreprise**, sélectionnez le nom du compte Automanage lorsqu’il apparaît.

### <a name="azure-portal"></a>Portail Azure
1. Sous **Abonnements**, accédez à l’abonnement qui contient vos machines virtuelles automanagées.
1. Accédez à **Contrôle d’accès (IAM)** .
1. Sélectionnez **Ajouter des attributions de rôle**.
1. Sélectionnez le rôle **Contributeur**, puis entrez le nom de votre compte Automanage.
1. Sélectionnez **Enregistrer**.
1. Répétez les étapes 3 à 5, cette fois avec le rôle **Contributeur de stratégie de ressource**.

### <a name="arm-template"></a>Modèle ARM
Exécutez le modèle ARM suivant. Vous aurez besoin de l’ID du principal de votre compte Automanage. La procédure à suivre est indiquée au début de cette section. Entrez l’ID lorsque vous y êtes invité.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "principalId": {
            "type": "string",
            "metadata": {
                "description": "The principal to assign the role to"
            }
        }
    },
    "variables": {
        "Contributor": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', 'b24988ac-6180-42a0-ab88-20f7382dd24c')]",
        "Resource Policy Contributor": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', '36243c78-bf99-498c-9df9-86d9f8d28608')]"
    },
    "resources": [
        {
            "type": "Microsoft.Authorization/roleAssignments",
            "apiVersion": "2018-09-01-preview",
            "name": "[guid(uniqueString(variables('Contributor')))]",
            "properties": {
                "roleDefinitionId": "[variables('Contributor')]",
                "principalId": "[parameters('principalId')]"
            }
        },
        {
            "type": "Microsoft.Authorization/roleAssignments",
            "apiVersion": "2018-09-01-preview",
            "name": "[guid(uniqueString(variables('Resource Policy Contributor')))]",
            "properties": {
                "roleDefinitionId": "[variables('Resource Policy Contributor')]",
                "principalId": "[parameters('principalId')]"
            }
        }
    ]
}
```

### <a name="azure-cli"></a>Azure CLI
Exécutez les commandes suivantes :

```azurecli
az role assignment create --assignee-object-id <your Automanage Account Object ID> --role "Contributor" --scope /subscriptions/<your subscription ID>

az role assignment create --assignee-object-id <your Automanage Account Object ID> --role "Resource Policy Contributor" --scope /subscriptions/<your subscription ID>
```

## <a name="next-steps"></a>Étapes suivantes
[En savoir plus sur Azure Automanage](./automanage-virtual-machines.md)