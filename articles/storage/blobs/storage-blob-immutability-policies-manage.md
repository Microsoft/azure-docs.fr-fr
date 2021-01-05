---
title: Définir et gérer des stratégies d’immuabilité pour le stockage Blob - Stockage Azure
description: Découvrez comment utiliser la prise en charge des disques optiques non réinscriptibles (WORM) pour le stockage d’objets blob, afin de stocker des données dans un état immuable et non effaçable sur une période donnée.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 11/26/2019
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: a09dbd7d778a4f7ea2a9aac9ca07b9e6d06bc1ae
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84463601"
---
# <a name="set-and-manage-immutability-policies-for-blob-storage"></a>Définir et gérer des stratégies d’immuabilité pour le stockage Blob

Le stockage immuable pour le Stockage Blob Azure permet aux utilisateurs de stocker des objets de données critiques pour l’entreprise dans un état WORM (disque optique non réinscriptible). Cet état les rend non effaçables et immuables pour une durée spécifiée par l’utilisateur. Au cours de cette période de conservation, il est possible de créer et de lire des blobs, mais pas de les modifier ou de les supprimer. Le stockage immuable est disponible pour les comptes de stockage Blob et comptes de stockage v2 universels dans toutes les régions Azure.

Cet article explique comment définir et gérer des stratégies d’immuabilité et des archivages juridiques pour les données dans le stockage Blob avec le portail Azure, PowerShell ou Azure CLI. Pour plus d’informations sur le stockage immuable, consultez [Stocker des données blob critiques pour l’entreprise avec un stockage immuable](storage-blob-immutable-storage.md).

## <a name="set-retention-policies-and-legal-holds"></a>Définir des stratégies de conservation et des archivages juridiques

### <a name="portal"></a>[Portail](#tab/azure-portal)

1. Créez un nouveau conteneur ou sélectionnez un conteneur existant pour stocker des objets Blob qui doivent demeurer dans l’état immuable. Le conteneur doit se trouver dans un compte de stockage Blob ou v2 universel.

2. Sélectionnez **Stratégie d’accès** dans les paramètres du conteneur. Ensuite, sélectionnez **Ajouter une stratégie** sous **Stockage blob non modifiable**.

    ![Paramètres du conteneur sur le portail](media/storage-blob-immutability-policies-manage/portal-image-1.png)

3. Pour activer la conservation limitée dans le temps, sélectionnez **Conservation limitée dans le temps** dans le menu déroulant.

    ![Sélection de « Conservation limitée dans le temps » sous « Type de stratégie »](media/storage-blob-immutability-policies-manage/portal-image-2.png)

4. Entrez la période de conservation en jours (les valeurs acceptées sont comprises entre 1 et 146000 jours).

    ![Zone « Mettre à jour la période de conservation »](media/storage-blob-immutability-policies-manage/portal-image-5-retention-interval.png)

    L’état initial de la stratégie est déverrouillé, ce qui vous permet de tester la fonctionnalité et d’apporter des modifications à la stratégie avant de la verrouiller. Le verrouillage de la stratégie est essentiel pour la conformité à la réglementation SEC 17a-4 notamment.

5. Verrouillez la stratégie. Cliquez avec le bouton droit sur les points de suspension ( **…** ) ; le menu suivant s’affiche avec des actions supplémentaires :

    ![« Verrouiller la stratégie » dans le menu](media/storage-blob-immutability-policies-manage/portal-image-4-lock-policy.png)

6. Sélectionnez **Verrouiller la stratégie** et confirmez le verrouillage. La stratégie est à présent verrouillée, sa suppression n’est plus possible ; seules les extensions de la période de conservation seront autorisées. Les suppressions et les remplacements d’objets blob ne sont pas autorisés. 

    ![Confirmer « Verrouiller la stratégie » dans le menu](media/storage-blob-immutability-policies-manage/portal-image-5-lock-policy.png)

7. Pour activer les archivages juridiques, sélectionnez **Ajouter une stratégie**. Sélectionnez **Archivage juridique** dans le menu déroulant.

    ![« Archivage juridique » dans le menu sous « Type de stratégie »](media/storage-blob-immutability-policies-manage/portal-image-legal-hold-selection-7.png)

8. Créez une stratégie d’archivage juridique avec une ou plusieurs balises.

    ![Zone « Nom de balise » sous le type de stratégie](media/storage-blob-immutability-policies-manage/portal-image-set-legal-hold-tags.png)

9. Pour effacer une conservation légale, supprimez la balise d’identificateur de conservation légale associée.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

La fonctionnalité est incluse dans les groupes de commandes suivants : `az storage container immutability-policy` et `az storage container legal-hold`. Exécutez `-h` sur ces groupes pour afficher les commandes.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Le module Az.Storage prend en charge le stockage immuable.  Pour activer cette fonctionnalité, suivez les étapes ci-dessous :

1. Vérifiez que la dernière version de PowerShellGet est installée : `Install-Module PowerShellGet –Repository PSGallery –Force`.
2. Supprimez les installations précédentes d’Azure PowerShell.
3. Installez Azure PowerShell : `Install-Module Az –Repository PSGallery –AllowClobber`.

L’exemple de script PowerShell suivant est fourni à titre de référence. Il crée un compte de stockage et un conteneur. Ensuite, il montre comment définir et effacer des stratégies d’archivage juridique, créer et verrouiller une stratégie de conservation limitée dans le temps (ou stratégie d’immuabilité) et étendre la période de conservation.

Créez d’abord un compte de stockage Azure :

```powershell
$resourceGroup = "<Enter your resource group>"
$storageAccount = "<Enter your storage account name>"
$container = "<Enter your container name>"
$location = "<Enter the storage account location>"

# Log in to Azure
Connect-AzAccount
Register-AzResourceProvider -ProviderNamespace "Microsoft.Storage"

# Create your Azure resource group
New-AzResourceGroup -Name $resourceGroup -Location $location

# Create your Azure storage account
$account = New-AzStorageAccount -ResourceGroupName $resourceGroup -StorageAccountName `
    $storageAccount -SkuName Standard_ZRS -Location $location -Kind StorageV2

# Create a new container using the context
$container = New-AzStorageContainer -Name $container -Context $account.Context

# List the containers in the account
Get-AzStorageContainer -Context $account.Context

# Remove a container
Remove-AzStorageContainer -Name $container -Context $account.Context
```

Définir et supprimer des archivages juridiques :

```powershell
# Set a legal hold
Add-AzRmStorageContainerLegalHold -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount -Name $container -Tag <tag1>,<tag2>,...

# Clear a legal hold
Remove-AzRmStorageContainerLegalHold -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount -Name $container -Tag <tag3>
```

Créez ou mettez à jour des stratégies d’immuabilité limitées dans le temps :

```powershell
# Create a time-based immutability policy
Set-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount -ContainerName $container -ImmutabilityPeriod 10
```

Récupérer des stratégies d’immuabilité :

```powershell
# Get an immutability policy
Get-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount -ContainerName $container
```

Verrouillez des stratégies d’immuabilité (ajoutez `-Force` pour ignorer l’invite) :

```powershell
# Lock immutability policies
$policy = Get-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $resourceGroup -StorageAccountName $storageAccount -ContainerName $container
Lock-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $resourceGroup -StorageAccountName $storageAccount -ContainerName $container `
    -Etag $policy.Etag
```

Étendre des stratégies d’immuabilité :

```powershell
# Extend immutability policies
$policy = Get-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $resourceGroup -StorageAccountName $storageAccount -ContainerName $container

Set-AzRmStorageContainerImmutabilityPolicy -ImmutabilityPolicy `
    $policy -ImmutabilityPeriod 11 -ExtendPolicy
```

Supprimez une stratégie d’immuabilité déverrouillée (ajoutez `-Force` pour ignorer l’invite) :

```powershell
# Remove an unlocked immutability policy
$policy = Get-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $resourceGroup -StorageAccountName $storageAccount -ContainerName $container

Remove-AzRmStorageContainerImmutabilityPolicy -ImmutabilityPolicy $policy
```

---

## <a name="enabling-allow-protected-append-blobs-writes"></a>Activation du paramètre Autoriser les écritures protégées de blobs d’ajout

### <a name="portal"></a>[Portail](#tab/azure-portal)

![Autoriser les écritures d’ajout supplémentaires](media/storage-blob-immutability-policies-manage/immutable-allow-additional-append-writes.png)

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

La fonctionnalité est incluse dans les groupes de commandes suivants : `az storage container immutability-policy` et `az storage container legal-hold`. Exécutez `-h` sur ces groupes pour afficher les commandes.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
# Create an immutability policy with appends allowed
Set-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount -ContainerName $container -ImmutabilityPeriod 10 -AllowProtectedAppendWrite $true
```

---

## <a name="next-steps"></a>Étapes suivantes

[Stocker des données blob critiques pour l’entreprise avec un stockage immuable](storage-blob-immutable-storage.md)
