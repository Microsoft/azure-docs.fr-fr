---
title: Guide pratique pour gérer les affectations avec PowerShell
description: Découvrez comment gérer les affectations de blueprints avec le module PowerShell Azure Blueprint, Az.Blueprint.
ms.date: 08/27/2020
ms.topic: how-to
ms.openlocfilehash: 3bcb3731bd1270497945fa86406d08b2f9750c85
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89051404"
---
# <a name="how-to-manage-assignments-with-powershell"></a>Guide pratique pour gérer les affectations avec PowerShell

Une affectation de blueprint peut être gérée avec le module PowerShell **Az.Blueprint**. Le module prend en charge l’extraction, la création, la mise à jour et la suppression des affectations. Le module peut également extraire des détails sur les définitions de blueprints existants. Cet article explique comment installer le module et comment l’utiliser.

## <a name="add-the-azblueprint-module"></a>Ajouter le module Az.Blueprint

Pour permettre à Azure PowerShell de gérer les affectations de blueprints, vous devez ajouter le module. Vous pouvez utiliser ce module avec PowerShell installé localement, avec [Azure Cloud Shell](https://shell.azure.com) ou avec l’[image Docker Azure PowerShell](https://hub.docker.com/r/azuresdk/azure-powershell/).

### <a name="base-requirements"></a>Configuration de base requise

Le module Azure Blueprint nécessite les logiciels suivants :

- Azure PowerShell 1.5.0 ou ultérieur. S’il n’est pas installé, suivez [ces instructions](/powershell/azure/install-az-ps).
- PowerShellGet 2.0.1 ou une version ultérieure. S’il n’est pas installé ou mis à jour, suivez [ces instructions](/powershell/scripting/gallery/installing-psget).

### <a name="install-the-module"></a>Installer le module

Le module Azure Blueprints pour PowerShell est **Az.Blueprint**.

1. À partir d’une invite PowerShell **d’administration**, exécutez la commande suivante :

   ```azurepowershell-interactive
   # Install the Azure Blueprints module from PowerShell Gallery
   Install-Module -Name Az.Blueprint
   ```

   > [!NOTE]
   > Si **Az.Accounts** est déjà installé, il peut être nécessaire d’utiliser `-AllowClobber` pour forcer l’installation.

1. Vérifiez que le module a été importé et que sa version est correcte (0.2.6) :

   ```azurepowershell-interactive
   # Get a list of commands for the imported Az.Blueprint module
   Get-Command -Module 'Az.Blueprint' -CommandType 'Cmdlet'
   ```

## <a name="get-blueprint-definitions"></a>Gérer les définitions de blueprint

La première étape pour utiliser une affectation est souvent d’obtenir une référence à une définition de blueprint.
L’applet de commande `Get-AzBlueprint` obtient une ou plusieurs définitions de blueprint. L’applet de commande peut obtenir les définitions de blueprint auprès d’un groupe d’administration avec `-ManagementGroupId {mgId}` ou un abonnement avec `-SubscriptionId {subId}`. Le paramètre **Name** obtient une définition de blueprint, mais doit être utilisé avec **ManagementGroupId** ou **SubscriptionId**. Vous pouvez utiliser **Version** avec **Name** pour être plus explicite sur la définition de blueprint à retourner. Au lieu de **Version**, le commutateur `-LatestPublished` extrait la dernière version publiée.

L’exemple suivant utilise `Get-AzBlueprint` pour obtenir toutes les versions d’une définition de blueprint nommée « 101-blueprints-definition-subscription » auprès d’un abonnement spécifique représenté par `{subId}` :

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get all versions of the blueprint definition in the specified subscription
$blueprints = Get-AzBlueprint -SubscriptionId '{subId}' -Name '101-blueprints-definition-subscription'

# Display the blueprint definition object
$blueprints
```

L’exemple de sortie pour une définition de blueprint avec plusieurs versions se présente comme ceci :

```output
Name                 : 101-blueprints-definition-subscription
Id                   : /subscriptions/{subId}/providers/Microsoft.Blueprint/blueprints/101
                       -blueprints-definition-subscription
DefinitionLocationId : {subId}
Versions             : {1.0, 1.1}
TimeCreated          : 2019-02-25
TargetScope          : Subscription
Parameters           : {storageAccount_storageAccountType, storageAccount_location,
                       allowedlocations_listOfAllowedLocations, [Usergrouporapplicationname]:Reader_RoleAssignmentName}
ResourceGroups       : ResourceGroup
```

Les [paramètres de blueprint](../concepts/parameters.md#blueprint-parameters) sur la définition de blueprint peuvent être développés pour fournir plus d’informations.

```azurepowershell-interactive
$blueprints.Parameters
```

```output
Key                                                    Value
---                                                    -----
storageAccount_storageAccountType                      Microsoft.Azure.Commands.Blueprint.Models.PSParameterDefinition
storageAccount_location                                Microsoft.Azure.Commands.Blueprint.Models.PSParameterDefinition
allowedlocations_listOfAllowedLocations                Microsoft.Azure.Commands.Blueprint.Models.PSParameterDefinition
[Usergrouporapplicationname]:Reader_RoleAssignmentName Microsoft.Azure.Commands.Blueprint.Models.PSParameterDefinition
```

## <a name="get-blueprint-assignments"></a>Obtenir des affectations de blueprint

Si l’affectation de blueprint existe déjà, vous pouvez obtenir une référence à celle-ci avec l’applet de commande `Get-AzBlueprintAssignment`. L’applet de commande prend **SubscriptionId** et **Name** comme paramètres facultatifs. Si **SubscriptionId** n’est pas spécifié, le contexte de l’abonnement actuel est utilisé.

L’exemple suivant utilise `Get-AzBlueprintAssignment` pour obtenir une affectation de blueprint unique nommée « Assignment-lock-resource-groups » auprès d’un abonnement spécifique représenté par `{subId}` :

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get the blueprint assignment in the specified subscription
$blueprintAssignment = Get-AzBlueprintAssignment -SubscriptionId '{subId}' -Name 'Assignment-lock-resource-groups'

# Display the blueprint assignment object
$blueprintAssignment
```

L’exemple de sortie pour une affectation de blueprint se présente comme ceci :

```output
Name              : Assignment-lock-resource-groups
Id                : /subscriptions/{subId}/providers/Microsoft.Blueprint/blueprintAssignme
                    nts/Assignment-lock-resource-groups
Scope             : /subscriptions/{subId}
LastModified      : 2019-02-19
LockMode          : AllResourcesReadOnly
ProvisioningState : Succeeded
Parameters        :
ResourceGroups    : ResourceGroup
```

## <a name="create-blueprint-assignments"></a>Créer des affectations de blueprint

Si l’affectation de blueprint n’existe pas encore, vous pouvez la créer avec l’applet de commande `New-AzBlueprintAssignment`. Cette applet de commande utilise les paramètres suivants :

- **Name** [obligatoire]
  - Spécifie le nom de l’affectation de blueprint
  - Doit être unique et ne pas déjà exister dans **SubscriptionId**
- **Blueprint** [obligatoire]
  - Spécifie la définition de blueprint à affecter
  - Utilisez `Get-AzBlueprint` pour obtenir l’objet de référence
- **Location** [obligatoire]
  - Spécifie la région où créer l’identité managée et l’objet de déploiement d’abonnement affectés par le système.
- **Subscription** (facultatif)
  - Spécifie l’abonnement sur lequel l’affectation est déployée
  - S’il n’est pas spécifié, sa valeur par défaut est le contexte actuel de l’abonnement
- **Lock** (facultatif)
  - Définit le [verrouillage de ressources de blueprint](../concepts/resource-locking.md) à utiliser pour les ressources déployées
  - Options prises en charge : _None_, _AllResourcesReadOnly_, _AllResourcesDoNotDelete_
  - S’il n’est pas spécifié, sa valeur par défaut est _None_
- **SystemAssignedIdentity** (facultatif)
  - Sélectionnez cette option pour créer une identité managée affectée par le système pour l’affectation et pour déployer les ressources
  - Option par défaut pour l’ensemble de paramètres « identity »
  - Ne peut pas être utilisé avec **UserAssignedIdentity**
- **UserAssignedIdentity** (facultatif)
  - Spécifie l’identité managée affectée par l’utilisateur, à utiliser pour l’affectation et pour déployer les ressources
  - Fait partie de l’ensemble de paramètres « identity »
  - Ne peut pas être utilisé avec **SystemAssignedIdentity**
- **Parameter** (facultatif)
  - Une [table de hachage](/powershell/module/microsoft.powershell.core/about/about_hash_tables) de paires clé/valeur pour la définition de [paramètres dynamiques](../concepts/parameters.md#dynamic-parameters) sur l’affectation de blueprint
  - La valeur par défaut pour un paramètre dynamique est **defaultValue** dans la définition
  - Si un paramètre n’est pas spécifié et n’a pas de **defaultValue**, le paramètre n’est pas facultatif

    > [!NOTE]
    > **Parameter** ne prend pas en charge secureStrings.

- **ResourceGroupParameter** (facultatif)
  - Une [table de hachage](/powershell/module/microsoft.powershell.core/about/about_hash_tables) d’artefacts de groupe de ressources
  - Chaque espace réservé d’artefact de groupe de ressources dispose d’une paire clé/valeur pour définir dynamiquement **Name** et **Location** sur cet artefact de groupe de ressources
  - Si un paramètre de groupe de ressources n’est pas spécifié et n’a pas de **defaultValue**, le paramètre de groupe de ressources n’est pas facultatif
- **AssignmentFile** (facultatif)
  - Chemin de la représentation de fichier JSON d’une affectation de blueprint
  - Ce paramètre fait partie d’un jeu de paramètres PowerShell qui comprend uniquement **Name**, **Blueprint** et **SubscriptionId**, ainsi que les paramètres communs.

### <a name="example-1-provide-parameters"></a>Exemple 1 : Fournir des paramètres

L’exemple suivant crée une nouvelle affectation de la version « 1.1 » de la définition de blueprint « my-blueprint » extraite avec `Get-AzBlueprint`, définit l’identité managée et l’emplacement de l’objet d’affectation sur « westus2 », verrouille les ressources avec  _AllResourcesReadOnly_, et définit les tables de hachage pour **Parameter** et **ResourceGroupParameter** sur un abonnement spécifique représenté par `{subId}` :

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get version '1.1' of the blueprint definition in the specified subscription
$bpDefinition = Get-AzBlueprint -SubscriptionId '{subId}' -Name 'my-blueprint' -Version '1.1'

# Create the hash table for Parameters
$bpParameters = @{storageAccount_storageAccountType='Standard_GRS'}

# Create the hash table for ResourceGroupParameters
# ResourceGroup is the resource group artifact placeholder name
$bpRGParameters = @{ResourceGroup=@{name='storage_rg';location='westus2'}}

# Create the new blueprint assignment
$bpAssignment = New-AzBlueprintAssignment -Name 'my-blueprint-assignment' -Blueprint $bpDefinition `
    -SubscriptionId '{subId}' -Location 'westus2' -Lock AllResourcesReadOnly `
    -Parameter $bpParameters -ResourceGroupParameter $bpRGParameters
```

L’exemple de sortie pour la création d’une affectation de blueprint se présente comme ceci :

```output
Name              : my-blueprint-assignment
Id                : /subscriptions/{subId}/providers/Microsoft.Blueprint/blueprintAssi
                    gnments/my-blueprint-assignment
Scope             : /subscriptions/{subId}
LastModified      : 2019-03-13
LockMode          : AllResourcesReadOnly
ProvisioningState : Creating
Parameters        : {storageAccount_storageAccountType}
ResourceGroups    : ResourceGroup
```

### <a name="example-2-use-a-json-assignment-definition-file"></a>Exemple 2 : Utiliser un fichier de définition d’attribution JSON

L’exemple suivant crée presque la même attribution que l’[Exemple 1](#example-1-provide-parameters). Au lieu de passer des paramètres à l’applet de commande, l’exemple illustre l’utilisation d’un fichier de définition d’attribution JSON et le paramètre **AssignmentFile**. De plus, la propriété **excludedPrincipals** est configurée dans le cadre de **locks**. Il n’existe pas de paramètre PowerShell pour **excludedPrincipals** et la propriété ne peut être configurée qu’en le définissant par le biais du fichier de définition d’attribution JSON.

```json
{
  "identity": {
    "type": "SystemAssigned"
  },
  "location": "westus2",
  "properties": {
    "description": "Assignment of the 101-blueprint-definition-subscription",
    "blueprintId": "/subscriptions/{subId}/providers/Microsoft.Blueprint/blueprints/101-blueprints-definition-subscription",
    "locks": {
      "mode": "AllResourcesReadOnly",
      "excludedPrincipals": [
          "7be2f100-3af5-4c15-bcb7-27ee43784a1f",
          "38833b56-194d-420b-90ce-cff578296714"
      ]
    },
    "parameters": {
      "storageAccount_storageAccountType": {
        "value": "Standard_GRS"
      }
    },
    "resourceGroups": {
      "ResourceGroup": {
        "name": "storage_rg",
        "location": "westus2"
      }
    }
  }
}
```

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Create the new blueprint assignment
$bpAssignment = New-AzBlueprintAssignment -Name 'my-blueprint-assignment' -SubscriptionId '{subId}' `
    -AssignmentFile '.\assignment.json'
```

Pour obtenir un exemple de fichier de définition d’attribution JSON pour une identité managée affectée par l’utilisateur, consultez le corps de la demande dans [Exemple : Attribution avec identité managée affectée par l’utilisateur](/rest/api/blueprints/assignments/createorupdate#examples) pour l’API REST.

## <a name="update-blueprint-assignments"></a>Mettre à jour des affectations de blueprint

Il est parfois nécessaire de mettre à jour d’une affectation de blueprint qui a déjà été créée. L’applet de commande `Set-AzBlueprintAssignment` gère cette action. L’applet de commande prend en grande partie les mêmes paramètres que `New-AzBlueprintAssignment`, ce qui permet la mise à jour de tout ce qui a été défini sur l’affectation. Les exceptions sont les paramètres _Name_, _Blueprint_ et _SubscriptionId_. Seules les valeurs fournies sont mises à jour.

Pour comprendre ce qui se passe lors de la mise à jour d’une affectation de blueprint, consultez [Règles de mise à jour des affectations](./update-existing-assignments.md#rules-for-updating-assignments).

- **Name** [obligatoire]
  - Spécifie le nom de l’affectation de blueprint à mettre à jour
  - Utilisé pour localiser l’affectation à mettre à jour, pas pour modifier l’affectation
- **Blueprint** [obligatoire]
  - Spécifie la définition de blueprint de l’affectation de blueprint
  - Utilisez `Get-AzBlueprint` pour obtenir l’objet de référence
  - Utilisé pour localiser l’affectation à mettre à jour, pas pour modifier l’affectation
- **Location** (facultatif)
  - Spécifie la région où créer l’identité managée et l’objet de déploiement d’abonnement affectés par le système.
- **Subscription** (facultatif)
  - Spécifie l’abonnement sur lequel l’affectation est déployée
  - S’il n’est pas spécifié, sa valeur par défaut est le contexte actuel de l’abonnement
  - Utilisé pour localiser l’affectation à mettre à jour, pas pour modifier l’affectation
- **Lock** (facultatif)
  - Définit le [verrouillage de ressources de blueprint](../concepts/resource-locking.md) à utiliser pour les ressources déployées
  - Options prises en charge : _None_, _AllResourcesReadOnly_, _AllResourcesDoNotDelete_
- **SystemAssignedIdentity** (facultatif)
  - Sélectionnez cette option pour créer une identité managée affectée par le système pour l’affectation et pour déployer les ressources
  - Option par défaut pour l’ensemble de paramètres « identity »
  - Ne peut pas être utilisé avec **UserAssignedIdentity**
- **UserAssignedIdentity** (facultatif)
  - Spécifie l’identité managée affectée par l’utilisateur, à utiliser pour l’affectation et pour déployer les ressources
  - Fait partie de l’ensemble de paramètres « identity »
  - Ne peut pas être utilisé avec **SystemAssignedIdentity**
- **Parameter** (facultatif)
  - Une [table de hachage](/powershell/module/microsoft.powershell.core/about/about_hash_tables) de paires clé/valeur pour la définition de [paramètres dynamiques](../concepts/parameters.md#dynamic-parameters) sur l’affectation de blueprint
  - La valeur par défaut pour un paramètre dynamique est **defaultValue** dans la définition
  - Si un paramètre n’est pas spécifié et n’a pas de **defaultValue**, le paramètre n’est pas facultatif

    > [!NOTE]
    > **Parameter** ne prend pas en charge secureStrings.

- **ResourceGroupParameter** (facultatif)
  - Une [table de hachage](/powershell/module/microsoft.powershell.core/about/about_hash_tables) d’artefacts de groupe de ressources
  - Chaque espace réservé d’artefact de groupe de ressources dispose d’une paire clé/valeur pour définir dynamiquement **Name** et **Location** sur cet artefact de groupe de ressources
  - Si un paramètre de groupe de ressources n’est pas spécifié et n’a pas de **defaultValue**, le paramètre de groupe de ressources n’est pas facultatif

L’exemple suivant met à jour l’affectation de la version « 1.1 » de la définition de blueprint « my-blueprint » extraite avec `Get-AzBlueprint` en changeant le mode de verrouillage :

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get version '1.1' of the blueprint definition in the specified subscription
$bpDefinition = Get-AzBlueprint -SubscriptionId '{subId}' -Name 'my-blueprint' -Version '1.1'

# Update the existing blueprint assignment
$bpAssignment = Set-AzBlueprintAssignment -Name 'my-blueprint-assignment' -Blueprint $bpDefinition `
    -SubscriptionId '{subId}' -Lock AllResourcesDoNotDelete
```

L’exemple de sortie pour la création d’une affectation de blueprint se présente comme ceci :

```output
Name              : my-blueprint-assignment
Id                : /subscriptions/{subId}/providers/Microsoft.Blueprint/blueprintAssi
                    gnments/my-blueprint-assignment
Scope             : /subscriptions/{subId}
LastModified      : 2019-03-13
LockMode          : AllResourcesDoNotDelete
ProvisioningState : Updating
Parameters        : {storageAccount_storageAccountType}
ResourceGroups    : ResourceGroup
```

## <a name="remove-blueprint-assignments"></a>Supprimer des affectations de blueprint

Quand le moment est venu de supprimer une affectation de blueprint, l’applet de commande `Remove-AzBlueprintAssignment` gère cette action. L’applet de commande prend **Name** ou **InputObject** pour spécifier l’affectation de blueprint à supprimer. **SubscriptionId** est _obligatoire_ et doit être spécifié dans tous les cas.

L’exemple suivant extrait une affectation de blueprint existante avec `Get-AzBlueprintAssignment`, puis la supprime de l’abonnement spécifique représenté par `{subId}` :

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get the blueprint assignment in the specified subscription
$blueprintAssignment = Get-AzBlueprintAssignment -Name 'Assignment-lock-resource-groups'

# Remove the existing blueprint assignment
Remove-AzBlueprintAssignment -InputObject $blueprintAssignment -SubscriptionId '{subId}'
```

## <a name="code-example"></a>Exemple de code

Rassemblant toutes les étapes, l’exemple suivant obtient la définition de blueprint, puis crée, met à jour et supprime une affectation de blueprint dans l’abonnement spécifique représenté par `{subId}` :

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

#region GetBlueprint
# Get version '1.1' of the blueprint definition in the specified subscription
$bpDefinition = Get-AzBlueprint -SubscriptionId '{subId}' -Name 'my-blueprint' -Version '1.1'
#endregion

#region CreateAssignment
# Create the hash table for Parameters
$bpParameters = @{storageAccount_storageAccountType='Standard_GRS'}

# Create the hash table for ResourceGroupParameters
# ResourceGroup is the resource group artifact placeholder name
$bpRGParameters = @{ResourceGroup=@{name='storage_rg';location='westus2'}}

# Create the new blueprint assignment
$bpAssignment = New-AzBlueprintAssignment -Name 'my-blueprint-assignment' -Blueprint $bpDefinition `
    -SubscriptionId '{subId}' -Location 'westus2' -Lock AllResourcesReadOnly `
    -Parameter $bpParameters -ResourceGroupParameter $bpRGParameters
#endregion CreateAssignment

# Wait for the blueprint assignment to finish deployment prior to the next steps

#region UpdateAssignment
# Update the existing blueprint assignment
$bpAssignment = Set-AzBlueprintAssignment -Name 'my-blueprint-assignment' -Blueprint $bpDefinition `
    -SubscriptionId '{subId}' -Lock AllResourcesDoNotDelete
#endregion UpdateAssignment

# Wait for the blueprint assignment to finish deployment prior to the next steps

#region RemoveAssignment
# Remove the existing blueprint assignment
Remove-AzBlueprintAssignment -InputObject $bpAssignment -SubscriptionId '{subId}'
#endregion
```

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur le [cycle de vie des blueprints](../concepts/lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](../concepts/parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](../concepts/sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](../concepts/resource-locking.md).
- Résoudre les problèmes durant l’affectation d’un blueprint en suivant les étapes de [dépannage général](../troubleshoot/general.md).