---
title: Déplacer des ressources entre différentes régions en utilisant PowerShell dans Azure Resource Mover
description: Découvrez comment déplacer des ressources entre différentes régions en utilisant PowerShell dans Azure Resource Mover.
manager: evansma
author: rayne-wiselman
ms.service: resource-move
ms.topic: how-to
ms.date: 11/30/2020
ms.author: raynew
ms.openlocfilehash: 0aca0e49d72025686cf44d434fa7a43ae0c86e0b
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461100"
---
# <a name="move-resources-across-regions-in-powershell"></a>Déplacer des ressources entre des régions dans PowerShell

Découvrez comment déplacer des ressources dans une région différente en utilisant PowerShell dans Azure Resource Mover. 

> [!NOTE]
> Azure Resource Mover est actuellement en préversion.



## <a name="before-you-start"></a>Avant de commencer

- Votre abonnement Azure doit avoir accès à Resource Mover, et vous devez disposer d’autorisations [Propriétaire](../role-based-access-control/built-in-roles.md#owner) ou [Administrateur de l’accès utilisateur](../role-based-access-control/built-in-roles.md#user-access-administrator) pour l’abonnement.
- Sachant que Resource Mover n’assure pas le suivi des modifications et des mises à niveau, apportez les modifications nécessaires aux ressources avant d’entamer leur déplacement.
- Pour l’heure, quand vous déplacez des ressources avec PowerShell, vous ne pouvez pas modifier les paramètres de la région cible. Modifiez ces paramètres directement sur le portail.
- Quand vous ajoutez des ressources à une collection de déplacements, pour préparer leur déplacement vers une autre région, les métadonnées relatives au déplacement sont stockées dans un groupe de ressources créé à cet effet. Pour l’heure, ce groupe de ressources peut résider dans les régions USA Est 2 ou Europe Nord. Les ressources Azure peuvent être déplacées entre n’importe quelles régions publiques en utilisant les métadonnées présentes dans l’une ou l’autre de ces régions.

## <a name="sample-values"></a>Exemples de valeurs

Voici les valeurs que nous utilisons dans nos exemples de script :

**Paramètre** | **Valeur** 
--- | ---
Identifiant d’abonnement | subscription-id
Localisation du service Resource Mover | USA Est 2
Groupe de ressources des métadonnées | RegionMoveRG-centralus-westcentralus
Localisation du groupe de ressources des métadonnées | USA Est 2
Nom de la collection de déplacement | MoveCollection-centralus-westcentralus
Région source |  centralus
Groupe de ressources source | PSDemoRM
Région cible | westcentralus
Groupe de ressources cible | PSDemoRMtgt
Machine virtuelle à déplacer | PSDemoVM

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous à votre abonnement Azure.

```azurepowershell-interactive
# Sign in to an Azure subscription
Connect-AzAccount – Subscription "<subscription-id>"
```

## <a name="create-the-resource-group-for-metadata"></a>Créer le groupe de ressources pour les métadonnées

Créez le groupe de ressources destiné à conserver les métadonnées de la collection de déplacement ainsi que les informations de configuration.

```azurepowershell-interactive
# Create the resource group for metadata
New-AzResourceGroup -Name MoveCollection-centralus-westcentralus -Location "East US 2"
```

**Sortie attendue** :

![Texte de sortie après la création du groupe de ressources](./media/move-region-powershell/output-create-resource-group.png)

## <a name="register-the-resource-provider"></a>Inscrire le fournisseur de ressources

Inscrivez le fournisseur de ressources Microsoft.Migrate de façon à pouvoir créer la ressource MoveCollection.

```azurepowershell-interactive
# Register the provider
Register-AzResourceProvider -ProviderNamespace Microsoft.Migrate 

# Wait for registration
While(((Get-AzResourceProvider -ProviderNamespace Microsoft.Migrate)| where {$_.RegistrationState -eq "Registered" -and $_.ResourceTypes.ResourceTypeName -eq "moveCollections"}|measure).Count -eq 0)
  {
      Start-Sleep -Seconds 5
      Write-Output "Waiting for registration to complete."
  }
```

## <a name="create-the-move-collection"></a>Créer la collection de déplacement

L’objet MoveCollection stocke les métadonnées et les informations de configuration relatives aux ressources à déplacer.

- Pour que l’objet MoveCollection accède à l’abonnement dans lequel se trouve le service Azure Resource Mover, il a besoin d’une [identité managée attribuée par le système](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) (anciennement Managed Service Identity (MSI)) et approuvée par l’abonnement.
- L’identité est attribuée au rôle Contributeur ou Administrateur de l’accès utilisateur pour l’abonnement source.
- Pour attribuer un rôle à l’identité, vous devez disposer d’un rôle dans l’abonnement (par exemple, Propriétaire ou Administrateur de l’accès utilisateur), avec les autorisations suivantes :
    -  Microsoft.Authorization/roleAssignments/write
    - Microsoft.Authorization/roleAssignments/delete


```azurepowershell-interactive
# Create a MoveCollection object
New-AzResourceMoverMoveCollection -Name "MoveCollection-centralus-westcentralus" -ResourceGroupName "RegionMoveRG-centralus-westcentralus" -SubscriptionId "<subscription-id>" -SourceRegion "centralus" -TargetRegion "westcentralus" -Location  "East US 2" -IdentityType SystemAssigned
```
**Sortie attendue** :

![Texte de sortie après la création de la collection de déplacement](./media/move-region-powershell/output-move-collection.png)


## <a name="assign-a-managed-identity"></a>Attribuer une identité managée 

Définissez l’ID d’abonnement, récupérez le principal d’identité de l’objet MoveCollection et attribuez-lui les rôles Azure.


```azurepowershell-interactive
# Set the subscriptionId
$subscriptionId = "<subscription-id>"

# Retrieve the principal managed identity of the MoveCollection
$moveCollection = Get-AzResourceMoverMoveCollection -SubscriptionId $subscriptionId -ResourceGroupName "RegionMoveRG-centralus-westcentralus" -Name "MoveCollection-centralus-westcentralus"

# Set the IdentityPrincipalID
$identityPrincipalId = $moveCollection.IdentityPrincipalId

# Assign the role to the IdentityPrincipalID
New-AzRoleAssignment -ObjectId $identityPrincipalId -RoleDefinitionName Contributor -Scope "/subscriptions/$subscriptionId"

New-AzRoleAssignment -ObjectId $identityPrincipalId -RoleDefinitionName "User Access Administrator" -Scope "/subscriptions/$subscriptionId"
```

## <a name="add-resources-to-the-move-collection"></a>Ajouter des ressources à la collection de déplacement

Récupérez l’ID de la ressource source à déplacer. Ajoutez-la ensuite à la collection de déplacement.

```azurepowershell-interactive
# Retrieve the resource ID
Get-AzResource -Name PSDemoVM -ResourceGroupName PSDemoRM
```

**Sortie attendue**

![Texte de sortie après avoir récupéré l’ID de ressource](./media/move-region-powershell/output-retrieve-resource.png)


```azurepowershell-interactive
# Add the resource to the move collection
New-AzResourceMoverMoveResource -SubscriptionId “<subscription-id>” -ResourceGroupName “RegionMoveRG-centralus-westcentralus” -MoveCollectionName “MoveCollection-centralus-westcentralus” -SourceId “/subscriptions/e80eb9fa-c996-4435-aa32-5af6f3d3077c/resourceGroups/PSDemoRM/providers/Microsoft.Compute/virtualMachines/PSDemoVM” -Name “PSDemoVM” -ResourceSettingResourceType “Microsoft.Compute/virtualMachines” -ResourceSettingTargetResourceName “PSDemoVM”
```

**Sortie attendue**
![Texte de sortie après l’ajout de la ressource](./media/move-region-powershell/output-add-resource.png)

## <a name="resolve-dependencies"></a>Résoudre les erreurs de dépendance

Validez les ressources que vous avez ajoutées et résolvez les dépendances vis-à-vis d’autres ressources.

```azurepowershell-interactive
# Resolve dependencies
Resolve-AzResourceMoverMoveCollectionDependency -SubscriptionId “<subscription-id>” -ResourceGroupName “RegionMoveRG-centralus-westcentralus” -MoveCollectionName “MoveCollection-centralus-westcentralus”
```
**Sortie en cas d’existence de dépendances**

Si la ressource dans la collection de déplacement a des dépendances vis-à-vis d’autres ressources, un message d’échec est émis.

![Texte de sortie après vérification des dépendances](./media/move-region-powershell/dependency-error.png)

### <a name="retrieve-dependencies"></a>Récupérer les dépendances

Si la ressource à déplacer présente des dépendances vis-à-vis d’autres ressources, vous pouvez récupérer des informations sur les dépendances manquantes.

```azurepowershell-interactive
# Identify dependencies
Get-AzResourceMoverUnresolvedDependency -MoveCollectionName “MoveCollection-centralus-westcentralus” -ResourceGroupName “RegionMoveRG-centralus-westcentralus” -SubscriptionId  “<subscription-id>”
```

**Sortie attendue**

![Texte de sortie après avoir récupéré les dépendances](./media/move-region-powershell/missing-dependencies.png)

Dans cet exemple, deux dépendances sont identifiées comme étant manquantes :

- Groupe de ressources source : PSDemoRM
- Carte réseau sur la machine virtuelle : PSDemoVM62

## <a name="add-dependencies-to-collection"></a>Ajouter des dépendances à la collection


À partir des ID de ressource que vous avez récupérés, identifiez les noms des ressources manquantes et ajoutez-les à la collection de déplacement.

### <a name="add-the-nic"></a>Ajouter la carte réseau

Récupérez le nom et le type de la carte réseau, puis ajoutez-la à la collection.

```azurepowershell-interactive
# Get the NIC name and resource type
Get-AzResource -ResourceId “/subscriptions/<subscription-id>/resourcegroups/psdemorm/providers/microsoft.network/networkinterfaces/psdemovm62”
```

**Sortie attendue**

![Texte de sortie après avoir récupéré la carte réseau](./media/move-region-powershell/output-retrieve-nic.png)

Ajoutez à présent la carte réseau à la collection de déplacement. Dans cet exemple, le même nom est attribué à la carte réseau cible. Veillez à la cohérence des paramètres et de la casse.

```azurepowershell-interactive
# Add the NIC to the collection. 
New-AzResourceMoverMoveResource -SubscriptionId “<subscription-id>” -ResourceGroupName “RegionMoveRG-centralus-westcentralus” -MoveCollectionName “MoveCollection-centralus-westcentralus” -SourceId “/subscriptions/<subscription-id>/resourceGroups/PSDemoRM/providers/Microsoft.Network/networkInterfaces/psdemovm62” -Name “psdemovm62” -ResourceSettingResourceType “Microsoft.Network/networkInterfaces” -ResourceSettingTargetResourceName “psdemovm62”
```

**Sortie attendue**

![Texte de sortie après avoir ajouté la carte réseau à la collection](./media/move-region-powershell/add-nic.png)

## <a name="add-the-source-resource-group"></a>Ajouter le groupe de ressources source

```azurepowershell-interactive
# Get the resource group name and resource type
Get-AzResource -ResourceId “/subscriptions/<subscription-id>/resourcegroups/psdemorm”

# Add the resource group to the collection. 
New-AzResourceMoverMoveResource -SubscriptionId “<subscription-id>” -ResourceGroupName “RegionMoveRG-centralus-westcentralus” -MoveCollectionName “MoveCollection-centralus-westcentralus” -SourceId “/subscriptions/<subscription-id>/resourcegroups/psdemorm” -Name “psdemorm” -ResourceSettingResourceType “resourcegroups” -ResourceSettingTargetResourceName “psdemorm”
```

**Sortie attendue**

![Texte de sortie après avoir ajouté le groupe de ressources à la collection](./media/move-region-powershell/add-source-resource-group.png)

## <a name="recheck-dependencies"></a>Revérifier les dépendances

Vérifiez à nouveau s’il manque des dépendances.

```azurepowershell-interactive
Get-AzResourceMoverUnresolvedDependency -MoveCollectionName “MoveCollection-centralus-westcentralus” -ResourceGroupName “RegionMoveRG-centralus-westcentralus” -SubscriptionId  “<subscription-id>”
```

**Sortie**

Il manque d’autres dépendances.

![Texte de sortie après la nouvelle vérification des dépendances](./media/move-region-powershell/recheck-dependencies.png)

## <a name="retrieve-additional-dependencies"></a>Récupérer d’autres dépendances

```azurepowershell-interactive
# Identify dependencies
Get-AzResourceMoverUnresolvedDependency -MoveCollectionName “MoveCollection-centralus-westcentralus” -ResourceGroupName “RegionMoveRG-centralus-westcentralus” -SubscriptionId  “<subscription-id>”
```

**Sortie attendue**


![Texte de sortie après avoir récupéré les autres dépendances](./media/move-region-powershell/additional-missing-dependencies.png)

Dans cet exemple, trois dépendances supplémentaires sont identifiées comme étant manquantes :

- Adresse IP publique : psdemovm-ip
- Réseau virtuel Azure : psdemorm-vnet
- Groupe de sécurité réseau (NSG) : psdemovm-nsg

## <a name="add-additional-dependencies"></a>Ajouter des dépendances supplémentaires

1. [Suivez les instructions](#add-dependencies-to-collection) pour ajouter ces ressources à la collection de déplacement.
2. Après avoir ajouté les ressources, validez à nouveau jusqu’à ce que toutes les dépendances aient été ajoutées.


## <a name="prepare-and-move-the-source-resource-group"></a>Préparer et déplacer le groupe de ressources source

Dans la région source, préparez les ressources avant de les déplacer. Le processus de préparation varie en fonction des ressources que vous déplacez. Par exemple, pour les ressources sans état, il peut s’avérer nécessaire de préparer et exporter un modèle Azure Resource Manager (ARM). Pour les ressources avec état, la réplication peut démarrer. 

Avant de préparer les ressources sources, vous devez préparer et déplacer le groupe de ressources source.

### <a name="prepare"></a>Préparation

```azurepowershell-interactive
# Prepare the source resource group
Invoke-AzResourceMoverPrepare -SubscriptionId “<subscription-id>” -ResourceGroupName “RegionMoveRG-centralus-westcentralus” -MoveCollectionName “MoveCollection-centralus-westcentralus” -MoveResource “PSDemoRM”
```

**Sortie attendue**

![Texte de sortie après la préparation du groupe de ressources source](./media/move-region-powershell/prepare-source-resource-group.png)

### <a name="initiate-move"></a>Lancer le déplacement

```azurepowershell-interactive
# Initiate move
Invoke-AzResourceMoverInitiateMove -SubscriptionId “<subscription-id>” -ResourceGroupName “RegionMoveRG-centralus-westcentralus” -MoveCollectionName “MoveCollection-centralus-westcentralus” -MoveResource “PSDemoRM”
```

**Sortie attendue**

![Texte de sortie après le lancement du déplacement du groupe de ressources source](./media/move-region-powershell/initiate-move-resource-group.png)


```azurepowershell-interactive
# Commit move
Invoke-AzResourceMoverCommit -SubscriptionId “<subscription-id” -ResourceGroupName “RegionMoveRG-centralus-westcentralus” -MoveCollectionName “PS-centralus-
westcentralus-demoRM” -MoveResource “PSDemoRM”
```

**Sortie attendue**

![Texte de sortie après la validation du groupe de ressources source](./media/move-region-powershell/commit-move-resource-group.png)


## <a name="prepare-resources"></a>Préparer les ressources

Une fois le groupe de ressources source déplacé dans la région cible, récupérez la liste des ressources de la collection de déplacement, puis préparez-les au déplacement.

```azurepowershell-interactive
# Retrieve resources in the collection
Get-AzResourceMoverMoveResource  -SubscriptionId “<subscription-id>“ -ResourceGroupName “RegionMoveRG-centralus-westcentralus” -MoveCollectionName “MoveCollection-centralus-westcentralus”   | Where-Object {  $_.MoveStatusMoveState -eq “PreparePending” } | Select Name
```
**Sortie attendue**

![Texte de sortie après avoir récupéré les ressources de la collection](./media/move-region-powershell/collection-resources.png)

À présent, préparez les ressources.

```azurepowershell-interactive
# Prepare the resources, using the resource name
Invoke-AzResourceMoverPrepare -SubscriptionId  <subscription-id> -ResourceGroupName RegionMoveRG-centralus-westcentralus  -MoveCollectionName MoveCollection-centralus-westcentralus   -MoveResource $('psdemovm62', 'PSDemoVM-ip', 'PSDemoRM-vnet','PSDemoVM-nsg', ‘PSDemoVM’)
```
Vous pouvez aussi préparer et déplacer les ressources en utilisant l’ID de ressource plutôt que le nom :

```azurepowershell-interactive
# Prepare the resources using the resource ID
Invoke-AzResourceMoverPrepare -SubscriptionId  <subscription-id> -ResourceGroupName RegionMoveRG-centralus-westcentralus  -MoveCollectionName MoveCollection-centralus-westcentralus  -MoveResourceInputType MoveResourceSourceId  -MoveResource $('/subscriptions/<subscription-id>/resourceGroups/PSDemoRM/providers/Microsoft.Network/networkSecurityGroups/PSDemoVM-nsg')
```

**Sortie attendue**

![Texte de sortie après la préparation des ressources de la collection](./media/move-region-powershell/collection-prepare.png)

## <a name="initiate-resource-move"></a>Lancer le déplacement des ressources

Une fois les ressources préparées, lancez le déplacement.

```azurepowershell-interactive
# Initiate the move 
Invoke-AzResourceMoverInitiateMove -SubscriptionId <subscription-id> -ResourceGroupName RegionMoveRG-centralus-westcentralus  -MoveCollectionName MoveCollection-centralus-westcentralus   -MoveResource $('psdemovm62', 'PSDemoVM-ip', 'PSDemoRM-vnet','PSDemoVM-nsg', ‘PSDemoVM’)
```
**Sortie attendue**

![Texte de sortie après le lancement du déplacement des ressources](./media/move-region-powershell/initiate-resource-move.png)

## <a name="discard-or-commit-the-move"></a>Abandonner ou valider le déplacement

Après le déplacement initial, vous pouvez décider de valider le déplacement ou de l’abandonner. 

- **Abandonner** : vous pouvez abandonner un déplacement si vous effectuez un test et que vous ne souhaitez pas déplacer réellement la ressource source. Si vous abandonnez le déplacement, la ressource reviendra à l’état *Lancement du déplacement en attente*. Vous pouvez alors lancer à nouveau le déplacement, si nécessaire.
- **Valider** : La validation termine le déplacement vers la région cible. Après sa validation, une ressource source se présente avec l’état de *Suppression de la source en attente*, et vous pouvez décider si vous voulez la supprimer.

### <a name="discard"></a>Abandonner

Pour abandonner le déplacement :

```azurepowershell-interactive
# Discard the move 
Invoke-AzResourceMoverDiscard -SubscriptionId  <subscription-id> `-ResourceGroupName RegionMoveRG-centralus-westcentralus  -MoveCollectionName MoveCollection-centralus-westcentralus   -MoveResource $('psdemovm62', 'PSDemoVM-ip', 'PSDemoRM-vnet','PSDemoVM-nsg', ‘PSDemoVM’)
```
**Sortie attendue**

![Texte de sortie après l’abandon du déplacement](./media/move-region-powershell/discard-move.png)


### <a name="commit"></a>Valider

Pour valider le déplacement :

```azurepowershell-interactive
# Commit the move 
Invoke-AzResourceMoverCommit -SubscriptionId  <subscription-id> -ResourceGroupName RegionMoveRG-centralus-westcentralus  -MoveCollectionName MoveCollection-centralus-westcentralus   -MoveResource $('psdemovm62', 'PSDemoVM-ip', 'PSDemoRM-vnet','PSDemoVM-nsg', ‘PSDemoVM’)
```
**Sortie attendue**

![Texte de sortie après la validation du déplacement](./media/move-region-powershell/commit-move.png)

## <a name="delete-source-resources"></a>Supprimer les ressources sources

Après avoir validé le déplacement et vérifié que les ressources fonctionnent comme prévu dans la région cible, vous pouvez supprimer les ressources sources à partir du [portail Azure](../azure-resource-manager/management/manage-resources-portal.md#delete-resources), de [PowerShell ](../azure-resource-manager/management/manage-resources-powershell.md#delete-resources) ou d’[Azure CLI](../azure-resource-manager/management/manage-resources-cli.md#delete-resources).

## <a name="next-steps"></a>Étapes suivantes

[Découvrez comment ](remove-move-resources.md)supprimer des ressources d’une collection de déplacement.
