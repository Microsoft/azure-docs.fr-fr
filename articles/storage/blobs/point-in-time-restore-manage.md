---
title: Effectuer une restauration jusqu’à une date et heure sur les données d’objet blob de blocs
titleSuffix: Azure Storage
description: Découvrez comment utiliser la restauration jusqu’à une date et heure pour restaurer un ensemble d’objets blob de blocs à leur état précédent à un moment donné.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 09/23/2020
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: 2350177373bc99907c437d814d8f01193f18f3fd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95895721"
---
# <a name="perform-a-point-in-time-restore-on-block-blob-data"></a>Effectuer une restauration jusqu’à une date et heure sur les données d’objet blob de blocs

Vous pouvez utiliser la récupération jusqu`à une date et heure pour restaurer un ou plusieurs jeux d’objets blob de blocs à un état antérieur. Cet article explique comment activer la limite de restauration dans le temps pour un compte de stockage et comment effectuer une opération de restauration.

Pour en savoir plus sur la restauration jusqu’à une date et heure, consultez [Restauration dans le temps pour les objets blob de blocs](point-in-time-restore-overview.md).

> [!CAUTION]
> La limite de restauration dans le temps prend en charge la restauration des opérations sur les objets blob de blocs uniquement. Les opérations sur les conteneurs ne peuvent pas être restaurées. Si vous supprimez un conteneur du compte de stockage en appelant l’opération [Supprimer le conteneur](/rest/api/storageservices/delete-container), ce conteneur ne peut pas être restauré à l’aide d’une opération de restauration. Au lieu de supprimer un conteneur, supprimez chacun des objets blob si vous souhaitez les restaurer.

## <a name="enable-and-configure-point-in-time-restore"></a>Activer et configurer la restauration dans le temps

Avant d’activer et de configurer la limite de restauration dans le temps, activez ses prérequis pour le compte de stockage : suppression réversible, flux de modification et gestion des versions d’objets blob. Pour plus d’informations sur l’activation de chacune de ces fonctionnalités, consultez les articles suivants :

- [Activer la suppression réversible pour les objets blob](./soft-delete-blob-enable.md)
- [Activer et désactiver le flux de modification](storage-blob-change-feed.md#enable-and-disable-the-change-feed)
- [Activer et gérer le contrôle de version des objets blob](versioning-enable.md)

> [!IMPORTANT]
> L’activation de la suppression réversible, du flux de modification et du contrôle de version des objets blob peut entraîner des frais supplémentaires. Pour plus d’informations, consultez [Suppression réversible pour les objets blob](soft-delete-blob-overview.md), [Prise en charge du flux de modification dans le stockage d’objets blob Azure](storage-blob-change-feed.md) et [Contrôle de version des objets blob](versioning-overview.md).

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour configurer une limite de restauration dans le temps avec le Portail Azure, procédez comme suit :

1. Accédez à votre compte de stockage dans le portail Azure.
1. Sous **Paramètres**, choisissez **Protection des données**.
1. Sélectionnez la restauration **Activer la restauration dans le temps**. Lorsque vous sélectionnez cette option, la suppression réversible pour les objets blob, le contrôle de version et le flux de modification est également activée.
1. Définissez le point de restauration maximal pour la restauration jusqu’à une date et heure, en jours. Ce nombre doit être inférieur ou égal à la période de rétention spécifiée pour la suppression réversible d’objet blob.
1. Enregistrez vos modifications.

L’illustration suivante montre un compte de stockage configuré pour la récupération jusqu’à une date et heure avec un point de restauration de sept jours plus tôt, et une période de rétention pour la suppression réversible des objets blob de 14 jours.

:::image type="content" source="media/point-in-time-restore-manage/configure-point-in-time-restore-portal.png" alt-text="Capture d’écran montrant comment configurer la restauration jusqu’à une date et heure dans le Portail Azure":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour configurer la limite de restauration dans le temps avec PowerShell, commencez par installer le module [Az.Storage](https://www.powershellgallery.com/packages/Az.Storage) version 2.6.0 ou une version ultérieure. Appelez ensuite la commande Enable-AzStorageBlobRestorePolicy pour activer la restauration jusqu’à une date et heure pour le compte de stockage.

L’exemple suivant active la suppression réversible et définit la période de rétention de suppression réversible, active le flux de modification et le contrôle de version, puis active la restauration à un instant dans le passé.    Lorsque vous exécutez l’exemple, n’oubliez pas de remplacer les valeurs entre crochets par vos propres valeurs :

```powershell
# Sign in to your Azure account.
Connect-AzAccount

# Set resource group and account variables.
$rgName = "<resource-group>"
$accountName = "<storage-account>"

# Enable soft delete with a retention of 14 days.
Enable-AzStorageBlobDeleteRetentionPolicy -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -RetentionDays 14

# Enable change feed and versioning.
Update-AzStorageBlobServiceProperty -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -EnableChangeFeed $true `
    -IsVersioningEnabled $true

# Enable point-in-time restore with a retention period of 7 days.
# The retention period for point-in-time restore must be at least
# one day less than that set for soft delete.
Enable-AzStorageBlobRestorePolicy -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -RestoreDays 7

# View the service settings.
Get-AzStorageBlobServiceProperty -ResourceGroupName $rgName `
    -StorageAccountName $accountName
```

---

## <a name="perform-a-restore-operation"></a>Effectuer une opération de restauration

Lorsque vous effectuez une opération de restauration, vous devez spécifier le point de restauration en tant que valeur **DateTime** UTC. Les conteneurs et les objets blob sont restaurés à leur état à cette date et heure. L’opération de restauration peut prendre plusieurs minutes.

Vous pouvez restaurer tous les conteneurs dans le compte de stockage, ou vous pouvez restaurer une plage d’objets blob dans un ou plusieurs conteneurs. Une plage d’objets blob est définie de façon lexicographique, ce qui signifie dans l’ordre du dictionnaire. Dix plages lexicographiques sont prises en charge par opération de restauration. Le début de la plage est inclusif et la fin de la plage est exclusive.

Le modèle de conteneur spécifié pour la plage de début et la plage de fin doit comprendre au moins trois caractères. La barre oblique (/) utilisée pour séparer le nom de conteneur du nom d’objet blob n’est pas prise en compte dans cette valeur minimale.

Les caractères génériques ne sont pas pris en charge dans une plage lexicographique. Les caractères génériques sont traités comme des caractères standard.

Vous pouvez restaurer des objets blob dans les conteneurs `$root` et `$web` en les spécifiant explicitement dans une plage transmise à une opération de restauration. Les conteneurs `$root` et `$web` sont restaurés uniquement s’ils sont spécifiés explicitement. Les autres conteneurs système ne peuvent pas être restaurés.

Seuls les objets blob de blocs sont restaurés. Les objets blob de pages et les objets blob d’ajout ne sont pas inclus dans une opération de restauration. Pour plus d’informations sur les limitations liées à l’ajout d’objets blob, consultez [Restauration dans le temps pour les objets blob de blocs](point-in-time-restore-overview.md).

> [!IMPORTANT]
> Lorsque vous effectuez une opération de restauration, Stockage Azure bloque les opérations de données sur les objets blob de la plage en cours de restauration pendant toute la durée de l'opération. Les opérations de lecture, d’écriture et de suppression sont bloquées dans l’emplacement principal. C’est la raison pour laquelle les opérations telles que l’énumération des conteneurs sur le portail Azure peuvent ne pas se dérouler comme prévu pendant que l’opération de restauration est en cours.
>
> Les opérations de lecture à partir de l’emplacement secondaire peuvent se poursuivre pendant l’opération de restauration si le compte de stockage est géorépliqué.

### <a name="restore-all-containers-in-the-account"></a>Restaurer tous les conteneurs du compte

Vous pouvez restaurer tous les conteneurs dans le compte de stockage pour les ramener à leur état précédent à un moment donné.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour restaurer tous les conteneurs et objets blob dans le compte de stockage à l’aide du Portail Azure, procédez comme suit :

1. Accédez à la liste des conteneurs pour votre compte de stockage.
1. Dans la barre d’outils, choisissez **Restaurer les conteneurs**, puis **Tout restaurer**.
1. Dans le volet **Restaurer tous les conteneurs**, spécifiez le point de restauration en fournissant une date et une heure.
1. Confirmez que vous souhaitez continuer en cochant la case.
1. Sélectionnez **Restaurer** pour commencer l’opération de restauration.

    :::image type="content" source="media/point-in-time-restore-manage/restore-all-containers-portal.png" alt-text="Capture d’écran montrant comment restaurer tous les conteneurs sur un point de restauration spécifié":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour restaurer tous les conteneurs et objets blob du compte de stockage avec PowerShell, appelez la commande **Restore-AzStorageBlobRange**. Par défaut, la commande **Restore-AzStorageBlobRange** s’exécute de façon asynchrone et retourne un objet de type **PSBlobRestoreStatus** que vous pouvez utiliser pour vérifier l’état de l’opération de restauration.

L’exemple suivant restaure de manière asynchrone les conteneurs du compte de stockage dans leur état 12 heures avant l’heure actuelle, et vérifie certaines des propriétés de l’opération de restauration :

```powershell
# Specify -TimeToRestore as a UTC value
$restoreOperation = Restore-AzStorageBlobRange -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -TimeToRestore (Get-Date).AddHours(-12)

# Get the status of the restore operation.
$restoreOperation.Status
# Get the ID for the restore operation.
$restoreOperation.RestoreId
# Get the restore point in UTC time.
$restoreOperation.Parameters.TimeToRestore
```

Pour exécuter l’opération de restauration de façon synchrone, incluez le paramètre **-WaitForComplete** sur la commande. Lorsque le paramètre **-WaitForComplete** est présent, PowerShell affiche un message qui comprend l’ID de restauration de l’opération, puis il bloque l’exécution jusqu’à ce que l’opération de restauration soit terminée. Gardez à l’esprit que la durée nécessaire à une opération de restauration dépend de la quantité de données à restaurer et qu’une opération de restauration peut prendre jusqu’à une heure.

```powershell
Restore-AzStorageBlobRange -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -TimeToRestore (Get-Date).AddHours(-12) -WaitForComplete
```

---

### <a name="restore-ranges-of-block-blobs"></a>Restaurer des plages d’objets blob de blocs

Vous pouvez restaurer une ou plusieurs plages d’objets blob dans un seul conteneur ou dans plusieurs conteneurs pour ramener ces objets blob à leur état antérieur à un moment donné.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour restaurer une plage d’objets blob dans un ou plusieurs conteneurs avec le Portail Azure, procédez comme suit :

1. Accédez à la liste des conteneurs pour votre compte de stockage.
1. Sélectionnez le ou les conteneurs à restaurer.
1. Dans la barre d’outils, choisissez **Restaurer les conteneurs**, puis **Restaurer la sélection**.
1. Dans le volet **Restaurer les conteneurs sélectionnés**, spécifiez le point de restauration en fournissant une date et une heure.
1. Spécifiez les plages à restaurer. Utilisez une barre oblique (/) pour détourer le nom du conteneur du préfixe de l’objet blob.
1. Par défaut, le volet **Restaurer les conteneurs sélectionnés** spécifie une plage qui comprend tous les objets blob du conteneur. Supprimez cette plage si vous ne souhaitez pas restaurer l’intégralité du conteneur. La plage par défaut est indiquée dans l’image suivante.

    :::image type="content" source="media/point-in-time-restore-manage/delete-default-blob-range.png" alt-text="Capture d’écran montrant la plage d’objets blob par défaut à supprimer avant de spécifier la plage personnalisée":::

1. Confirmez que vous souhaitez continuer en cochant la case.
1. Sélectionnez **Restaurer** pour commencer l’opération de restauration.

L’illustration suivante montre une opération de restauration sur un ensemble de plages.

:::image type="content" source="media/point-in-time-restore-manage/restore-multiple-container-ranges-portal.png" alt-text="Capture d’écran montrant comment restaurer des plages d’objets blob dans un ou plusieurs conteneurs":::

L’opération de restauration indiquée dans l’image effectue les actions suivantes :

- Restaure le contenu complet de *container1*.
- Restaure les objets blob dans la plage lexicographique de *blob1* à *blob5* dans *container2*. Cette plage restaure les objets blob avec des noms tels que *blob1*, *blob11*, *blob100*, *blob2*, et ainsi de suite. Étant donné que la fin de la plage est exclusive, elle restaure les objets blob dont le nom commence par *blob4*, mais pas les objets blob dont le nom commence par *blob5*.
- Restaure tous les objets blob dans *container3* et *container4*. Étant donné que la fin de la plage est exclusive, cette plage ne restaure pas *container5*.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour restaurer une unique plage d’objets blob, appelez la commande **Restore-AzStorageBlobRange** et spécifiez une plage lexicographique de noms de conteneurs et d’objets blob pour le paramètre `-BlobRestoreRange`. Par exemple, pour restaurer les objets blob dans un seul conteneur appelé *container1*, vous pouvez spécifier une plage qui commence par *container1* et se termine par *container2*. Il n’est pas nécessaire que les conteneurs nommés dans les plages de début et de fin existent. Étant donné que la fin de la plage est exclusive, même si le compte de stockage comprend un conteneur nommé *container2*, seul le conteneur nommé *container1* sera restauré :

```powershell
$range = New-AzStorageBlobRangeToRestore -StartRange container1 `
    -EndRange container2
```

Pour spécifier un sous-ensemble d’objets blob dans un conteneur à restaurer, utilisez une barre oblique (/) pour séparer le nom du conteneur du modèle de préfixe d’objet blob. Par exemple, la plage suivante sélectionne des objets blob dans un conteneur unique dont les noms commencent par les lettres *d* à *f* :

```powershell
$range = New-AzStorageBlobRangeToRestore -StartRange container1/d `
    -EndRange container1/g
```

Ensuite, fournissez la plage à la commande **Restore-AzStorageBlobRange**. Spécifiez le point de restauration en fournissant une valeur UTC **DateTime** pour le paramètre `-TimeToRestore`. L’exemple suivant restaure les objets blob de la plage spécifiée dans leur état 3 jours avant le moment présent :

```powershell
# Specify -TimeToRestore as a UTC value
Restore-AzStorageBlobRange -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -BlobRestoreRange $range `
    -TimeToRestore (Get-Date).AddDays(-3)
```

Par défaut, la commande **Restore-AzStorageblobRange** s’exécute de façon asynchrone. Lorsque vous lancez une opération de restauration de manière asynchrone, PowerShell affiche immédiatement une table de propriétés pour l’opération :  

```powershell
Status     RestoreId                            FailureReason Parameters.TimeToRestore     Parameters.BlobRanges
------     ---------                            ------------- ------------------------     ---------------------
InProgress 459c2305-d14a-4394-b02c-48300b368c63               2020-09-15T23:23:07.1490859Z ["container1/d" -> "container1/g"]
```

Pour restaurer plusieurs plages d’objets blob de blocs, spécifiez un groupe de plages pour le paramètre `-BlobRestoreRange`. L’exemple suivant spécifie deux plages pour restaurer le contenu complet de *container1* et *container4* à leur état il y a 24 heures, et enregistre le résultat dans une variable :

```powershell
# Specify a range that includes the complete contents of container1.
$range1 = New-AzStorageBlobRangeToRestore -StartRange container1 `
    -EndRange container2
# Specify a range that includes the complete contents of container4.
$range2 = New-AzStorageBlobRangeToRestore -StartRange container4 `
    -EndRange container5

$restoreOperation = Restore-AzStorageBlobRange -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -TimeToRestore (Get-Date).AddHours(-24) `
    -BlobRestoreRange @($range1, $range2)

# Get the status of the restore operation.
$restoreOperation.Status
# Get the ID for the restore operation.
$restoreOperation.RestoreId
# Get the blob ranges specified for the operation.
$restoreOperation.Parameters.BlobRanges
```

Pour exécuter l’opération de restauration de façon synchrone et bloquer l’exécution jusqu’à ce qu’elle soit terminée, incluez le paramètre **-WaitForComplete** sur la commande.

---

## <a name="next-steps"></a>Étapes suivantes

- [Restauration dans le temps pour les objets blob de blocs](point-in-time-restore-overview.md)
- [Suppression réversible](./soft-delete-blob-overview.md)
- [Flux de modification](storage-blob-change-feed.md)
- [Contrôle de version des blobs](versioning-overview.md)