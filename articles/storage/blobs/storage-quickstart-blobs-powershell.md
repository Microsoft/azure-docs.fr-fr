---
title: 'Démarrage rapide : Créer un blob avec PowerShell'
titleSuffix: Azure Storage
description: Dans ce guide de démarrage rapide, vous utilisez Azure PowerShell dans un stockage d’objets (blob). Ensuite, vous utilisez PowerShell pour charger un objet blob dans Stockage Azure, télécharger un objet blob et répertorier les objets blob dans un conteneur.
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.date: 03/31/2020
ms.author: tamram
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: b051a4ffe4d24e1ef0e69ab7c18a8ed3388b57e5
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "89078280"
---
# <a name="quickstart-upload-download-and-list-blobs-with-powershell"></a>Démarrage rapide : Charger, télécharger et lister des blobs avec PowerShell

Utilisez le module Azure PowerShell pour créer et gérer les ressources Azure. La création ou la gestion des ressources Azure peut se faire depuis une ligne de commande PowerShell ou dans des scripts. Ce guide décrit l’utilisation de PowerShell pour transférer des fichiers entre un disque local et le Stockage Blob Azure.

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="prerequisites"></a>Prérequis

Pour accéder à Stockage Azure, vous avez besoin d’un abonnement Azure. Si vous n’avez pas d’abonnement, vous pouvez [créer un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Ce démarrage rapide requiert le module Azure PowerShell Az version 0.7 ou ultérieure. Exécutez `Get-InstalledModule -Name Az -AllVersions | select Name,Version` pour trouver la version. Si vous devez installer ou mettre à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps).

[!INCLUDE [storage-quickstart-tutorial-intro-include-powershell](../../../includes/storage-quickstart-tutorial-intro-include-powershell.md)]

## <a name="create-a-container"></a>Créez un conteneur.

Les objets blob sont toujours chargés dans un conteneur. Vous pouvez organiser des groupes d’objets blob de la même façon que vous organisez vos fichiers dans les dossiers de l’ordinateur.

Définissez le nom du conteneur, puis créez le conteneur à l’aide de la commande [New-AzStorageContainer](/powershell/module/az.storage/new-azstoragecontainer). Définissez les autorisations dans `blob` pour autoriser l’accès aux fichiers. Le nom du conteneur dans cet exemple est *quickstartblobs*.

```powershell
$containerName = "quickstartblobs"
New-AzStorageContainer -Name $containerName -Context $ctx -Permission blob
```

## <a name="upload-blobs-to-the-container"></a>Charger des objets blob dans le conteneur

Stockage Blob prend en charge les objets blob de blocs, d’ajout et de pages. Les fichiers de disque dur virtuel utilisés pour appuyer les machines virtuelles IaaS sont des objets blob de pages. Utilisez des objets blob d’ajout pour la journalisation, par exemple, quand vous voulez écrire dans un fichier et continuer à ajouter d’autres informations. La plupart des fichiers stockés dans Stockage Blob sont des objets blob de blocs. 

Pour charger un fichier sur un objet blob de blocs, obtenez une référence de conteneur, puis obtenez une référence à l’objet blob de blocs dans ce conteneur. Une fois que vous avez la référence à l’objet blob, vous pouvez y charger des données à l’aide de [Set-AzStorageBlobContent](/powershell/module/az.storage/set-azstorageblobcontent). Si l’objet blob n’existe pas, cette opération le crée. S’il existe, il est remplacé.

Les exemples suivants chargent *Image001.jpg* et *Image002.png* à partir du dossier *D:\\_TestImages* sur le disque local, dans le conteneur que vous avez créé.

```powershell
# upload a file to the default account (inferred) access tier
Set-AzStorageBlobContent -File "D:\_TestImages\Image000.jpg" `
  -Container $containerName `
  -Blob "Image001.jpg" `
  -Context $ctx 

# upload a file to the Hot access tier
Set-AzStorageBlobContent -File "D:\_TestImages\Image001.jpg" `
  -Container $containerName `
  -Blob "Image001.jpg" `
  -Context $ctx 
  -StandardBlobTier Hot

# upload another file to the Cool access tier
Set-AzStorageBlobContent -File "D:\_TestImages\Image002.png" `
  -Container $containerName `
  -Blob "Image002.png" `
  -Context $ctx
  -StandardBlobTier Cool

# upload a file to a folder to the Archive access tier
Set-AzStorageBlobContent -File "D:\_TestImages\foldername\Image003.jpg" `
  -Container $containerName `
  -Blob "Foldername/Image003.jpg" `
  -Context $ctx 
  -StandardBlobTier Archive
```

Chargez autant de fichiers que vous le souhaitez avant de continuer.

## <a name="list-the-blobs-in-a-container"></a>Créer la liste des objets blob d’un conteneur

Obtenez la liste des objets blob dans le conteneur à l’aide de la commande [Get-AzStorageBlob](/powershell/module/az.storage/get-azstorageblob). Cet exemple montre uniquement les noms des objets blob chargés.

```powershell
Get-AzStorageBlob -Container $ContainerName -Context $ctx | select Name
```

## <a name="download-blobs"></a>Télécharger des objets blob

Téléchargez les objets blob sur votre disque local. Pour chaque objet blob à télécharger, définissez le nom et appelez la commande [Get-AzStorageBlobContent](/powershell/module/az.storage/get-azstorageblobcontent) pour télécharger l’objet blob.

Cet exemple télécharge les objets blob dans *D:\\_TestImages\Downloads* sur le disque local. 

```powershell
# download first blob
Get-AzStorageBlobContent -Blob "Image001.jpg" `
  -Container $containerName `
  -Destination "D:\_TestImages\Downloads\" `
  -Context $ctx 

# download another blob
Get-AzStorageBlobContent -Blob "Image002.png" `
  -Container $containerName `
  -Destination "D:\_TestImages\Downloads\" `
  -Context $ctx
```

## <a name="data-transfer-with-azcopy"></a>Transfert de données avec AzCopy

L’utilitaire en ligne de commande AzCopy assure un transfert de données hautement performant et scriptable pour Stockage Azure. Vous pouvez utiliser AzCopy pour transférer des données depuis et vers le stockage Blob et Azure Files. Pour plus d’informations sur AzCopy v10, la dernière version d’AzCopy, consultez [Bien démarrer avec AzCopy](../common/storage-use-azcopy-v10.md). Pour plus d’informations sur l’utilisation d’AzCopy v10 avec le stockage Blob, consultez [Transférer des données avec AzCopy et le stockage Blob](../common/storage-use-azcopy-blobs.md).

L’exemple suivant utilise AzCopy pour charger un fichier local dans un objet blob. Pensez à remplacer les valeurs de l’exemple par vos propres valeurs :

```powershell
azcopy login
azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt'
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Supprimez toutes les ressources que vous avez créées. Pour ce faire, le plus simple consiste à supprimer le groupe de ressources. La suppression du groupe de ressources supprime aussi toutes les ressources qu’il contient. Dans l’exemple suivant, la suppression du groupe de ressources supprime aussi le compte de stockage et le groupe de ressources.

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez transféré des fichiers entre un système de fichiers local et le Stockage Blob Azure. Pour en savoir plus sur l’utilisation du Stockage Blob à l’aide de PowerShell, explorez les exemples PowerShell afin de découvrir le Stockage Blob.

> [!div class="nextstepaction"]
> [Exemples Azure PowerShell pour le Stockage Blob Azure](storage-samples-blobs-powershell.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)

### <a name="microsoft-azure-powershell-storage-cmdlets-reference"></a>Référence des applets de commande de stockage Microsoft Azure PowerShell

* [Applets de commande PowerShell - Stockage](/powershell/module/az.storage)

### <a name="microsoft-azure-storage-explorer"></a>Explorateur Stockage Microsoft Azure

* [Microsoft Azure Storage Explorer](../../vs-azure-tools-storage-manage-with-storage-explorer.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) est une application autonome et gratuite de Microsoft qui vous permet d’exploiter visuellement les données de Stockage Azure sur Windows, macOS et Linux.
