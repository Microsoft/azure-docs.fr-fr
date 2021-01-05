---
title: Démarrage rapide pour la gestion des partages de fichiers Azure avec Azure PowerShell
description: Utilisez ce guide de démarrage rapide pour découvrir comment gérer les partages de fichiers Azure avec Azure PowerShell.
author: roygara
ms.service: storage
ms.topic: quickstart
ms.date: 10/26/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 8b4bd9ece5f010f1294356ad4673543834e5076a
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94626909"
---
# <a name="quickstart-create-and-manage-an-azure-file-share-with-azure-powershell"></a>Démarrage rapide : créer et gérer un partage de fichiers Azure avec Azure PowerShell 
Ce guide vous explique les bases de l’utilisation du [partage de fichiers](storage-files-introduction.md) Azure avec PowerShell. Le partage de fichiers Azure est similaire à d’autres partages de fichiers, mais est stocké dans le cloud et s’appuie sur la plateforme Azure. Les partages de fichiers Azure prennent en charge le protocole SMB (Server Message Block) standard, le protocole NFS (Network File System) (en préversion) et permettent de partager des fichiers sur plusieurs machines, applications et instances. 

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si vous souhaitez installer et utiliser PowerShell localement, vous devez exécuter le module Azure PowerShell Az version 0.7 ou ultérieure pour les besoins de ce guide. Pour connaître la version du module Azure PowerShell que vous exécutez, utilisez la commande `Get-Module -ListAvailable Az`. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Login-AzAccount` pour créer une connexion avec Azure.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources
Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Si vous n’avez pas encore de groupe de ressources Azure, vous pouvez en créer un avec l’applet de commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). 

L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* dans la région USA Ouest 2 :

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$region = "westus2"

New-AzResourceGroup `
    -Name $resourceGroupName `
    -Location $region | Out-Null
```

## <a name="create-a-storage-account"></a>Créez un compte de stockage.
Un compte de stockage est un pool partagé de stockage que vous pouvez utiliser pour déployer des partages de fichiers Azure. Un compte de stockage peut contenir un nombre illimité de partages, et un partage peut stocker un nombre illimité de fichiers, dans les limites de capacité du compte de stockage. Cet exemple crée une version 2 à usage général (compte de stockage GPv2), qui peut stocker des partages de fichiers Azure standard ou d’autres ressources de stockage, comme des objets blob ou des files d’attente, sur un média à rotation de lecteur de disque dur (HDD). Azure Files prend aussi en charge les disques SSD Premium ; des partages de fichiers Azure Premium peuvent être créés dans des comptes de stockage FileStorage.

Cet exemple crée un compte de stockage à l’aide de l’applet de commande [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount). Le compte de stockage est nommé *mystorageaccount\<random number>* et une référence à ce compte de stockage est stockée dans la variable **$storageAcct**. Les noms de compte de stockage doivent être uniques. Par conséquent, utilisez `Get-Random` pour ajouter un numéro au nom pour le rendre unique. 

```azurepowershell-interactive 
$storageAccountName = "mystorageacct$(Get-Random)"

$storageAcct = New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName `
    -Location $region `
    -Kind StorageV2 `
    -SkuName Standard_ZRS `
    -EnableLargeFileShare
```

> [!Note]  
> Les partages de plus de 5 Tio (maximum de 100 Tio par partage) sont uniquement disponibles dans les comptes de stockage localement redondant (LRS) et redondant interzone (ZRS). Pour créer un compte de stockage géo-redondant (GRS) ou géo-redondant interzone (GZRS), supprimez le paramètre `-EnableLargeFileShare`.

## <a name="create-an-azure-file-share"></a>Crée un partage de fichiers Azure
Vous pouvez désormais créer votre premier partage de fichiers Azure. Vous pouvez créer un partage de fichiers à l’aide de l’applet de commande [New-AzRmStorageShare](/powershell/module/az.storage/New-AzRmStorageShare). Cet exemple crée un partage nommé `myshare`.

```azurepowershell-interactive
$shareName = "myshare"

New-AzRmStorageShare `
    -StorageAccount $storageAcct `
    -Name $shareName `
    -EnabledProtocol SMB `
    -QuotaGiB 1024 | Out-Null
```

Les noms de partage ne doivent contenir que des minuscules, des nombres et des traits d’union uniques, mais ne peut commencer par un trait d’union. Pour plus d’informations sur la façon de nommer des partages de fichiers et des fichiers, consultez la rubrique [Affectation de noms et références aux partages, répertoires, fichiers et métadonnées](/rest/api/storageservices/Naming-and-Referencing-Shares--Directories--Files--and-Metadata).

## <a name="use-your-azure-file-share"></a>Utiliser votre partage de fichiers Azure
Azure Files offre deux méthodes pour utiliser les fichiers et dossiers au sein de votre partage de fichiers Azure : le [protocole Server Message Block (SMB)](/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) de norme industrielle et le [protocole REST de fichier](/rest/api/storageservices/file-service-rest-api). 

Pour monter un partage de fichiers avec SMB, consultez le document suivant en fonction de votre système d’exploitation :
- [Windows](storage-how-to-use-files-windows.md)
- [Linux](storage-how-to-use-files-linux.md)
- [macOS](storage-how-to-use-files-mac.md)

### <a name="using-an-azure-file-share-with-the-file-rest-protocol"></a>Utilisation d’un partage de fichiers Azure avec le protocole REST de fichier 
Vous pouvez utiliser le protocole REST de fichier directement (c’est-à-dire fabriquer vous-même les appels HTTP REST manuellement), mais la méthode d’utilisation du protocole REST de fichier la plus courante consiste à utiliser le module Azure PowerShell, l’interface [Azure CLI](storage-how-to-use-files-cli.md) ou un SDK Stockage Azure, tous faisant office de wrapper efficace autour du protocole REST de fichier dans le langage de script/programmation de votre choix.  

Dans la plupart des cas, vous allez utiliser votre partage de fichiers Azure sur le protocole SMB, car cela vous permet d’utiliser les applications et outils existants que l’on s’attend à utiliser. Toutefois, utiliser l’API REST de fichier plutôt que SMB est avantageux pour plusieurs raisons, par exemple :

- Vous naviguez sur votre partage de fichiers à partir du Cloud Shell PowerShell (qui ne peut pas monter les partages de fichiers sur SMB).
- Vous tirez profit de ressources serverless, comme [Azure Functions](../../azure-functions/functions-overview.md) par exemple. 
- Vous créez un service à valeur ajoutée qui interagit avec de nombreux partages de fichiers Azure, par exemple en effectuant des sauvegardes ou des analyses antivirus.

Les exemples suivants montrent comment utiliser le module Azure PowerShell pour manipuler votre partage de fichiers Azure avec le protocole REST de fichier. Le paramètre `-Context` sert à récupérer la clé de compte du stockage pour effectuer les actions indiquées sur le partage de fichiers. Pour récupérer la clé du compte de stockage, vous devez disposer du rôle Azure `Owner` sur le compte de stockage.

#### <a name="create-directory"></a>Créer un répertoire
Pour créer un répertoire nommé *myDirectory* à la racine de votre partage de fichiers Azure, utilisez l’applet de commande [New-AzStorageDirectory](/powershell/module/az.storage/New-AzStorageDirectory).

```azurepowershell-interactive
New-AzStorageDirectory `
   -Context $storageAcct.Context `
   -ShareName $shareName `
   -Path "myDirectory"
```

#### <a name="upload-a-file"></a>Charger un fichier
Pour montrer comment charger un fichier avec l’applet de commande [Set-AzStorageFileContent](/powershell/module/az.storage/Set-AzStorageFileContent), nous devons d’abord créer un fichier dans le disque de travail de PowerShell Cloud Shell à charger. 

Cet exemple place la date et l’heure actuelles dans un nouveau fichier sur votre disque de travail, puis charge le fichier vers le partage de fichiers.

```azurepowershell-interactive
# this expression will put the current date and time into a new file on your scratch drive
cd "~/CloudDrive/"
Get-Date | Out-File -FilePath "SampleUpload.txt" -Force

# this expression will upload that newly created file to your Azure file share
Set-AzStorageFileContent `
   -Context $storageAcct.Context `
   -ShareName $shareName `
   -Source "SampleUpload.txt" `
   -Path "myDirectory\SampleUpload.txt"
```   

Si vous exécutez PowerShell en local, vous devez remplacer `~/CloudDrive/` par un chemin existant sur votre machine.

Après avoir chargé le fichier, vous pouvez exécuter l’applet de commande [Get-AzStorageFile](/powershell/module/Az.Storage/Get-AzStorageFile) pour vérifier que le fichier a bien été chargé dans votre partage de fichiers Azure. 

```azurepowershell-interactive
Get-AzStorageFile `
    -Context $storageAcct.Context `
    -ShareName $shareName `
    -Path "myDirectory\" 
```

#### <a name="download-a-file"></a>Téléchargement d’un fichier
Vous pouvez exécuter l’applet de commande [Get-AzStorageFileContent](/powershell/module/az.storage/Get-AzStorageFilecontent) pour télécharger une copie du fichier que vous venez de charger sur le disque de travail de Cloud Shell.

```azurepowershell-interactive
# Delete an existing file by the same name as SampleDownload.txt, if it exists because you've run this example before.
Remove-Item `
    -Path "SampleDownload.txt" `
    -Force `
    -ErrorAction SilentlyContinue

Get-AzStorageFileContent `
    -Context $storageAcct.Context `
    -ShareName $shareName `
    -Path "myDirectory\SampleUpload.txt" `
    -Destination "SampleDownload.txt"
```

Une fois le fichier téléchargé, vous pouvez utiliser `Get-ChildItem` pour voir si le fichier a été téléchargé sur votre disque de travail de PowerShell Cloud Shell.

```azurepowershell-interactive
Get-ChildItem | Where-Object { $_.Name -eq "SampleDownload.txt" }
``` 

#### <a name="copy-files"></a>Copie des fichiers
Une tâche courante consiste à copier les fichiers d’un partage de fichiers vers un autre. Pour illustrer cette fonctionnalité, vous pouvez créer un partage et copier le fichier que vous venez de charger dans ce nouveau partage avec l’applet de commande [Start-AzStorageFileCopy](/powershell/module/az.storage/Start-AzStorageFileCopy). 

```azurepowershell-interactive
$otherShareName = "myshare2"

New-AzRmStorageShare `
    -StorageAccount $storageAcct `
    -Name $otherShareName `
    -EnabledProtocol SMB `
    -QuotaGiB 1024 | Out-Null
  
New-AzStorageDirectory `
   -Context $storageAcct.Context `
   -ShareName $otherShareName `
   -Path "myDirectory2"

Start-AzStorageFileCopy `
    -Context $storageAcct.Context `
    -SrcShareName $shareName `
    -SrcFilePath "myDirectory\SampleUpload.txt" `
    -DestShareName $otherShareName `
    -DestFilePath "myDirectory2\SampleCopy.txt" `
    -DestContext $storageAcct.Context
```

Maintenant, si vous répertoriez les fichiers dans le nouveau partage, vous devez voir votre fichier copié.

```azurepowershell-interactive
Get-AzStorageFile `
    -Context $storageAcct.Context `
    -ShareName $otherShareName `
    -Path "myDirectory2" 
```

Bien que l’applet de commande `Start-AzStorageFileCopy` soit pratique pour les déplacements de fichiers ad hoc entre les partages de fichiers Azure, pour les migrations et les déplacements de données plus importants, nous préconisons `robocopy` sur Windows et `rsync` sur macOS et Linux. `robocopy` et `rsync` utilisent SMB pour effectuer les déplacement de données et non l’API FileREST.

## <a name="create-and-manage-share-snapshots"></a>Créer et gérer des instantanés de partage
Avec un partage de fichiers Azure, vous pouvez aussi créer des instantanés de partage. Un instantané conserve un point dans le temps pour un partage de fichiers Azure. Les instantanés de partage sont similaires aux technologies de systèmes d’exploitation que vous connaissez peut-être déjà comme :

- Le [service VSS](/windows/desktop/VSS/volume-shadow-copy-service-portal) pour les systèmes de fichiers Windows comme NTFS et ReFS.
- Les instantanés du [gestionnaire de volumes logiques (LVM)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)#Basic_functionality) pour les systèmes Linux.
- Les instantanés du [système de fichiers Apple (APFS)](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/APFS_Guide/Features/Features.html) pour macOS. 

Vous pouvez créer un instantané de partage pour un partage à l’aide de la méthode `Snapshot` sur un objet PowerShell pour un partage de fichiers, qui est récupéré avec l’applet de commande [Get-AzStorageShare](/powershell/module/az.storage/Get-AzStorageShare). 

```azurepowershell-interactive
$share = Get-AzStorageShare -Context $storageAcct.Context -Name $shareName
$snapshot = $share.CloudFileShare.Snapshot()
```

### <a name="browse-share-snapshots"></a>Parcourir les instantanés de partage
Vous pouvez parcourir le contenu de l’instantané de partage en transmettant la référence d’instantané (`$snapshot`) au paramètre `-Share` de la cmdlet `Get-AzStorageFile`.

```azurepowershell-interactive
Get-AzStorageFile -Share $snapshot
```

### <a name="list-share-snapshots"></a>Répertorier les instantanés de partage
Vous pouvez afficher la liste d’instantanés pris pour votre partage avec la commande suivante.

```azurepowershell-interactive
Get-AzStorageShare `
        -Context $storageAcct.Context | `
    Where-Object { $_.Name -eq $shareName -and $_.IsSnapshot -eq $true }
```

### <a name="restore-from-a-share-snapshot"></a>Restaurer à partir d’un instantané de partage
Vous pouvez restaurer un fichier avec la commande `Start-AzStorageFileCopy` que nous avons utilisée plus tôt. Dans le cadre de ce démarrage rapide, nous allons tout d’abord supprimer notre fichier `SampleUpload.txt` que nous avons téléchargé précédemment afin de pouvoir le restaurer à partir de l’instantané.

```azurepowershell-interactive
# Delete SampleUpload.txt
Remove-AzStorageFile `
    -Context $storageAcct.Context `
    -ShareName $shareName `
    -Path "myDirectory\SampleUpload.txt"

# Restore SampleUpload.txt from the share snapshot
Start-AzStorageFileCopy `
    -SrcShare $snapshot `
    -SrcFilePath "myDirectory\SampleUpload.txt" `
    -DestContext $storageAcct.Context `
    -DestShareName $shareName `
    -DestFilePath "myDirectory\SampleUpload.txt"
```

### <a name="delete-a-share-snapshot"></a>Supprimer un instantané de partage
Pour supprimer un instantané de partage, exécutez l’applet de commande [Remove-AzStorageShare](/powershell/module/az.storage/Remove-AzStorageShare), avec la variable contenant la référence `$snapshot` au paramètre `-Share`.

```azurepowershell-interactive
Remove-AzStorageShare `
    -Share $snapshot `
    -Confirm:$false `
    -Force
```

## <a name="clean-up-resources"></a>Nettoyer les ressources
Une fois que vous avez terminé, vous pouvez utiliser l’applet de commande [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour supprimer le groupe de ressources et toutes les ressources associées. 

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

Vous pouvez aussi supprimer les ressources une par une :

- Pour supprimer les partages de fichiers Azure que nous avons créés pour ce guide de démarrage rapide.

    ```azurepowershell-interactive
    Get-AzRmStorageShare -StorageAccount $storageAcct | Remove-AzRmStorageShare -Force
    ```

    > [!Note]  
    > Vous devez supprimer tous les instantanés de partage pour les partages de fichiers Azure que vous avez créés avant de supprimer ces derniers.

- Pour supprimer le compte de stockage (ce qui supprimera par la même occasion les partages de fichiers créés ainsi que toutes les autres ressources de stockage créées telles qu’un conteneur de stockage d’objets blob Azure).

    ```azurepowershell-interactive
    Remove-AzStorageAccount `
        -ResourceGroupName $storageAcct.ResourceGroupName `
        -Name $storageAcct.StorageAccountName
    ```

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Qu’est-ce qu’Azure Files ?](storage-files-introduction.md)