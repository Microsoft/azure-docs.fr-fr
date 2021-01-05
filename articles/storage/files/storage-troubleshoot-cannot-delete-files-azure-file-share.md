---
title: 'Partage de fichiers Azure : échec de suppression de fichiers du partage de fichiers Azure'
description: Identifiez et résolvez l’échec de suppression de fichiers du partage de fichiers Azure.
author: v-miegge
ms.topic: troubleshooting
ms.author: kartup
manager: dcscontentpm
ms.date: 10/25/2019
ms.service: storage
ms.subservice: files
services: storage
tags: ''
ms.openlocfilehash: 3d4f10745d90ccd83e7251af40d3e92a230f2fcd
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94629680"
---
# <a name="azure-file-share--failed-to-delete-files-from-azure-file-share"></a>Partage de fichiers Azure : échec de suppression de fichiers du partage de fichiers Azure

L’échec de suppression de fichiers du partage de fichiers Azure peut avoir plusieurs symptômes :

**Symptôme 1 :**

Échec de suppression de fichier du partage de fichiers Azure en raison de l’un des deux problèmes suivants :

* Fichier marqué pour suppression
* La ressource spécifiée est peut-être utilisée par un client SMB

**Symptôme 2 :**

Quota insuffisant pour traiter cette commande

## <a name="cause"></a>Cause

L’erreur 1816 se produit lorsque vous atteignez la limite de descripteurs ouverts simultanément autorisés pour un fichier sur l’ordinateur où le partage de fichiers est en cours de montage. Pour plus d’informations, voir la [Liste de contrôle des performances et de l’extensibilité de Stockage Azure](../blobs/storage-performance-checklist.md).

## <a name="resolution"></a>Résolution

Réduisez le nombre de descripteurs ouverts simultanément en fermant certains d’entre eux.

## <a name="prerequisite"></a>Configuration requise

### <a name="install-the-latest-azure-powershell-module"></a>Installer le module Azure PowerShell

* [Installer le module Azure PowerShell](/powershell/azure/install-az-ps)

### <a name="connect-to-azure"></a>Connectez-vous à Azure :

```
# Connect-AzAccount
```

### <a name="select-the-subscription-of-the-target-storage-account"></a>Sélectionnez l’abonnement du compte de stockage cible :

```
# Select-AzSubscription -subscriptionid "SubscriptionID"
```

### <a name="create-context-for-the-target-storage-account"></a>Créez un contexte pour le compte de stockage cible :

```
$Context = New-AzStorageContext -StorageAccountName "StorageAccountName" -StorageAccountKey "StorageAccessKey"
```

### <a name="get-the-current-open-handles-of-the-file-share"></a>Obtenez les descripteurs ouverts actuels du partage de fichiers :

```
# Get-AzStorageFileHandle -Context $Context -ShareName "FileShareName" -Recursive
```

## <a name="example-result"></a>Exemple de résultat :

|HandleId|Path|ClientIp|ClientPort|OpenTime|LastReconnectTime|FileId|ParentId|SessionId|
|---|---|---|---|---|---|---|---|---|
|259101229083|---|10.222.10.123|62758|05-10-2019|12:16:50Z|0|0|9507758546259807489|
|259101229131|---|10.222.10.123|62758|05-10-2019|12:36:20Z|0|0|9507758546259807489|
|259101229137|---|10.222.10.123|62758|05-10-2019|12:36:53Z|0|0|9507758546259807489|
|259101229136|Nouveau dossier/test.zip|10.222.10.123|62758|05-10-2019|12:36:29Z|13835132822072852480|9223446803645464576|9507758546259807489|
|259101229135|test.zip|37.222.22.143|62758|05-10-2019|12:36:24Z|11529250230440558592|0|9507758546259807489|

### <a name="close-an-open-handle"></a>Fermer un descripteur ouvert :

Pour fermer un descripteur ouvert, utilisez la commande suivante :

```
# Close-AzStorageFileHandle -Context $Context -ShareName "FileShareName" -Path 'New folder/test.zip' -CloseAll
```

## <a name="next-steps"></a>Étapes suivantes

* [Résoudre les problèmes d’Azure Files dans Windows](storage-troubleshoot-windows-file-connection-problems.md)
* [Résoudre les problèmes d’Azure Files dans Linux](storage-troubleshoot-linux-file-connection-problems.md)
* [Résoudre les problèmes d’Azure File Sync](storage-sync-files-troubleshoot.md)