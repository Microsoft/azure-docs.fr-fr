---
title: Ajouter une identité managée à un type de nœud cluster géré par Service Fabric (préversion)
description: Cet article explique comment ajouter une identité managée à un type de nœud de cluster géré par Service Fabric.
ms.topic: how-to
ms.date: 11/24/2020
ms.custom: references_regions
ms.openlocfilehash: 00e679b07a44b799b6ac6677201bb59eeddcd6cf
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96839677"
---
# <a name="add-a-managed-identity-to-a-service-fabric-managed-cluster-node-type-preview"></a>Ajouter une identité managée à un type de nœud cluster géré par Service Fabric (préversion)

Chaque type de nœud d’un cluster managé Service Fabric est adossé à un groupe de machines virtuelles identiques. Pour permettre l’utilisation des identités managées avec un type de nœud de cluster géré, une propriété `vmManagedIdentity` a été ajoutée aux définitions de type de nœud contenant une liste d’identités qui peuvent être utilisées, `userAssignedIdentities`. La fonctionnalité reflète la manière dont les identités managées peuvent être utilisées dans les clusters non gérés, par exemple l’utilisation d’une identité managée avec l’[extension de groupe de machines virtuelles identiques Azure Key Vault](https://docs.microsoft.com/azure/virtual-machines/extensions/key-vault-windows).


Pour obtenir un exemple de déploiement de cluster géré par Service Fabric qui utilise l’identité managée sur un type de nœud, consultez [ce modèle](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/SF-Managed-Standard-SKU-1-NT-MI). Pour obtenir la liste des régions prises en charge, consultez [la FAQ relative aux clusters gérés](https://docs.microsoft.com/azure/service-fabric/faq-managed-cluster#what-regions-are-supported-in-the-preview).

> [!NOTE]
> Seules les identités affectées par l’utilisateur sont actuellement prises en charge pour cette fonctionnalité.

## <a name="prerequisites"></a>Prérequis

Avant de commencer :

* Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Si vous prévoyez d’utiliser PowerShell, [installez](https://docs.microsoft.com/cli/azure/install-azure-cli) l’interface Azure CLI pour exécuter les commandes de référence de l’interface de ligne de commande.

## <a name="create-a-user-assigned-managed-identity"></a>Créer une identité managée attribuée par l’utilisateur 

Une identité managée affectée par l’utilisateur peut être définie dans la section des ressources d’un modèle Azure Resource Manager (ARM) à des fins de création lors du déploiement :

```JSON
{ 
    "type": "Microsoft.ManagedIdentity/userAssignedIdentities", 
    "name": "[parameters('userAssignedIdentityName')]", 
    "apiVersion": "2018-11-30", 
    "location": "[resourceGroup().location]"  
},
```

ou créée via PowerShell :

```powershell
az group create --name <resourceGroupName> --location <location>
az identity create --name <userAssignedIdentityName> --resource-group <resourceGroupName>
```

## <a name="add-a-role-assignment-with-service-fabric-resource-provider"></a>Ajouter une attribution de rôle au fournisseur de ressources Service Fabric

Ajoutez une attribution de rôle à l’identité managée avec l’application de fournisseur de ressources Service Fabric. Cette attribution permet au fournisseur de ressources Service Fabric d’attribuer l’identité au groupe de machines virtuelles identiques du cluster géré. 

Les valeurs suivantes doivent être utilisées, le cas échéant :

|Name|Valeur du fournisseur de ressources Service Fabric correspondant|
|----|-------------------------------------|
|ID de l'application|74cb6831-0dbb-4be1-8206-fd4df301cdc2|
|ID de l'objet|fbc587f2-66f5-4459-a027-bcd908b9d278|


|Nom de la définition de rôle|ID de définition de rôle|
|----|-------------------------------------|
|Opérateur d’identités gérées|f1a07417-d97a-45cb-824c-7a7467783830
|



Cette attribution de rôle peut être définie dans la section Ressources à l’aide de l’ID d’objet et de l’ID de définition de rôle :

```JSON
{
    "type": "Microsoft.Authorization/roleAssignments", 
    "apiVersion": "2020-04-01-preview",
    "name": "[parameters('vmIdentityRoleNameGuid')]",
    "scope": "[concat('Microsoft.ManagedIdentity/userAssignedIdentities', '/', parameters('userAssignedIdentityName'))]",
    "dependsOn": [ 
        "[concat('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('userAssignedIdentityName'))]"
    ], 
    "properties": {
        "roleDefinitionId": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', 'f1a07417-d97a-45cb-824c-7a7467783830')]",
        "principalId": "fbc587f2-66f5-4459-a027-bcd908b9d278" 
    } 
}, 
```

ou créée via PowerShell soit à l’aide de l’ID d’application et de l’ID de définition de rôle :

```powershell
New-AzRoleAssignment -ApplicationId 74cb6831-0dbb-4be1-8206-fd4df301cdc2 -RoleDefinitionName "Managed Identity Operator" -Scope "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<userAssignedIdentityName>"
```

soit à l’aide de l’ID d’objet et de l’ID de définition de rôle :

```powershell
New-AzRoleAssignment -PrincipalId fbc587f2-66f5-4459-a027-bcd908b9d278 -RoleDefinitionName "Managed Identity Operator" -Scope "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<userAssignedIdentityName>"
```

## <a name="add-managed-identity-properties-to-node-type-definition"></a>Ajouter les propriétés d’identité managée à la définition de type de nœud

Enfin, ajoutez les propriétés `vmManagedIdentity` et `userAssignedIdentities` à la définition du type de nœud du cluster géré :

```json

"properties": {
    "isPrimary" : true,
    "vmInstanceCount": 5,
    "dataDiskSizeGB": 100,
    "vmSize": "Standard_D2",
    "vmImagePublisher" : "MicrosoftWindowsServer",
    "vmImageOffer" : "WindowsServer",
    "vmImageSku" : "2019-Datacenter",
    "vmImageVersion" : "latest",
    "vmManagedIdentity": {
        "userAssignedIdentities": [
            "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('userAssignedIdentityName'))]"
        ]
    }
}
```

Après le déploiement, l’identité managée créée a été ajoutée au groupe de machines virtuelles identiques du type de nœud désigné et peut être utilisée comme prévu, comme dans n’importe quel cluster non géré.

## <a name="troubleshooting"></a>Dépannage

Si vous n’ajoutez pas correctement une attribution de rôle, l’erreur suivante se produira lors du déploiement :

:::image type="content" source="media/how-to-managed-identity-managed-cluster-vmss/role-assignment-error.png" alt-text="Erreur de déploiement du portail Azure indiquant que le client possédant l’ID d’objet/application du SFRP n’est pas autorisé à effectuer une activité de gestion des identités":::

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Déployer une application sur un cluster Service Fabric managé](https://docs.microsoft.com/azure/service-fabric/tutorial-managed-cluster-deploy-app) 
