---
title: Effectuer des opérations de Stockage File d’attente Azure dans PowerShell
description: Effectuez des opérations sur Stockage File d’attente Azure avec PowerShell. Grâce à Stockage File d’attente Azure, vous pouvez stocker un grand nombre de messages accessibles par HTTP/HTTPS.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 05/15/2019
ms.service: storage
ms.subservice: queues
ms.topic: how-to
ms.reviewer: dineshm
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: a2f1229ab8a292b06dfc43b95d9047ed8d233523
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93345701"
---
# <a name="perform-azure-queue-storage-operations-with-azure-powershell"></a>Effectuer des opérations sur Stockage File d’attente Azure avec Azure PowerShell

Les files d’attente de stockage Azure sont un service permettant de stocker un grand nombre de messages accessibles depuis n’importe où dans le monde via HTTP ou HTTPS. Pour plus d’informations, consultez [Présentation des files d’attente Azure](storage-queues-introduction.md). Cet article sur les procédures décrit les opérations courantes liées au Stockage File d’attente. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
>
> - Créer une file d’attente
> - Récupérer une file d’attente
> - Ajouter un message
> - Lire un message
> - Supprimer un message
> - Suppression d'une file d'attente

Ce guide pratique nécessite le module Az Azure PowerShell version 0.7 ou ultérieure. Exécutez `Get-Module -ListAvailable Az` pour trouver la version. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps).

Il n’existe aucune applet de commande PowerShell pour le plan de données des files d’attente. Pour effectuer des opérations de plan de données comme l’ajout, la lecture et la suppression d’un message, vous devez utiliser la bibliothèque cliente de stockage .NET exposée dans PowerShell. Vous créez un objet de message et vous pouvez alors utiliser des commandes comme AddMessage pour effectuer des opérations sur ce message. Cet article vous explique comment procéder.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous à votre abonnement Azure avec la commande `Connect-AzAccount` et suivez les instructions à l’écran.

```powershell
Connect-AzAccount
```

## <a name="retrieve-list-of-locations"></a>Récupérer la liste des régions

Si vous ne savez pas quelle localisation utiliser, affichez la liste des localisations disponibles. Dans la liste, trouvez celle que vous souhaitez utiliser. Cet exercice utilise **eastus**. Stockez cette région dans la variable **location** pour une utilisation ultérieure.

```powershell
Get-AzLocation | Select-Object Location
$location = "eastus"
```

## <a name="create-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Stockez le nom du groupe de ressources dans une variable pour une utilisation ultérieure. Dans cet exemple, un groupe de ressources nommé *howtoqueuesrg* est créé dans la région *eastus*.

```powershell
$resourceGroup = "howtoqueuesrg"
New-AzResourceGroup -ResourceGroupName $resourceGroup -Location $location
```

## <a name="create-storage-account"></a>Créer un compte de stockage

Créez un compte de stockage universel standard avec un stockage localement redondant (LRS) à l’aide de [New-AzStorageAccount](/powershell/module/az.storage/New-azStorageAccount). Obtenez le contexte du compte de stockage qui définit le compte de stockage à utiliser. Si un compte de stockage est utilisé, référencez le contexte au lieu d’entrer les informations d’identification à plusieurs reprises.

```powershell
$storageAccountName = "howtoqueuestorage"
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name $storageAccountName `
  -Location $location `
  -SkuName Standard_LRS

$ctx = $storageAccount.Context
```

## <a name="create-a-queue"></a>Créer une file d’attente

L'exemple suivant établit d'abord une connexion à Azure Storage à l'aide du contexte de compte de stockage, ce qui inclut le nom de compte de stockage et sa clé d'accès. Ensuite, il appelle la cmdlet [New-AzStorageQueue](/powershell/module/az.storage/New-AzStorageQueue) pour créer une file d’attente appelée « howtoqueue ».

```powershell
$queueName = "howtoqueue"
$queue = New-AzStorageQueue –Name $queueName -Context $ctx
```

Pour plus d’informations sur les conventions d’affectation de noms pour le service de File d’attente Azure, consultez la page [Affectation de noms pour les files d’attente et les métadonnées](/rest/api/storageservices/Naming-Queues-and-Metadata).

## <a name="retrieve-a-queue"></a>Récupérer une file d’attente

Vous pouvez interroger et récupérer une file d'attente spécifique ou une liste de toutes les files d'attente dans un compte de stockage. Les exemples suivants montrent comment récupérer toutes les files d’attente dans le compte de stockage et une file d’attente spécifique ; les deux commandes utilisent l’applet de commande [Get-AzStorageQueue](/powershell/module/az.storage/Get-AzStorageQueue).

```powershell
# Retrieve a specific queue
$queue = Get-AzStorageQueue –Name $queueName –Context $ctx
# Show the properties of the queue
$queue

# Retrieve all queues and show their names
Get-AzStorageQueue -Context $ctx | Select-Object Name
```

## <a name="add-a-message-to-a-queue"></a>Ajout d'un message à une file d'attente

Les opérations ayant un impact sur les messages réels dans la file d’attente utilisent la bibliothèque cliente de stockage .NET exposée dans PowerShell. Pour ajouter un message à une file d’attente, créez une instance de l’objet de message, la classe [Microsoft.Azure.Storage.Queue.CloudQueueMessage](/java/api/com.microsoft.azure.storage.queue.cloudqueuemessage). Appelez ensuite la méthode [AddMessage](/java/api/com.microsoft.azure.storage.queue.cloudqueue.addmessage) . Un CloudQueueMessage peut être créé à partir d'une chaîne (au format UTF-8) ou d'un tableau d'octets.

L’exemple suivant montre comment ajouter un message à votre file d’attente.

```powershell
# Create a new message using a constructor of the CloudQueueMessage class
$queueMessage = [Microsoft.Azure.Storage.Queue.CloudQueueMessage]::new("This is message 1")

# Add a new message to the queue
$queue.CloudQueue.AddMessageAsync($QueueMessage)

# Add two more messages to the queue
$queueMessage = [Microsoft.Azure.Storage.Queue.CloudQueueMessage]::new("This is message 2")
$queue.CloudQueue.AddMessageAsync($QueueMessage)

$queueMessage = [Microsoft.Azure.Storage.Queue.CloudQueueMessage]::new("This is message 3")
$queue.CloudQueue.AddMessageAsync($QueueMessage)
```

Si vous utilisez [l’Explorateur Stockage Azure](https://storageexplorer.com), vous pouvez vous connecter à votre compte Azure et afficher les files d’attente dans le compte de stockage, puis en explorer une au niveau du détail pour afficher les messages qu’elle contient.

## <a name="read-a-message-from-the-queue-then-delete-it"></a>Lire un message de la file d’attente, puis le supprimer

Les messages sont lus, dans la mesure du possible, dans l’ordre du premier entré, premier sorti. Nous n’offrons aucune garantie. Quand vous lisez le message de la file d’attente, il devient invisible à tous les autres processus ciblant la file d’attente. De cette façon, si votre code ne parvient pas à traiter un message à cause d’une défaillance matérielle ou logicielle, une autre instance de votre code peut obtenir le même message et réessayer.

Cette **période d’invisibilité** définit la durée d’invisibilité du message avant qu’il ne redevienne disponible pour traitement. La valeur par défaut est 30 secondes.

Votre code lit un message dans la file d’attente en deux étapes. Quand vous appelez la méthode [Microsoft.Azure.Storage.Queue.CloudQueue.GetMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.getmessage), vous obtenez le message suivant dans la file d’attente. Un message renvoyé par **GetMessage** devient invisible par les autres codes lisant les messages de cette file d'attente. Pour finaliser la suppression du message de la file d’attente, vous devez également appeler la méthode [Microsoft.Azure.Storage.Queue.CloudQueue.DeleteMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.deletemessage).

Dans l’exemple suivant, vous lisez les trois messages de la file d’attente, puis attendez 10 secondes (période d’invisibilité). Ensuite, quand vous relisez les trois messages, vous les supprimez un à un en appelant **DeleteMessage**. Si vous essayez de lire la file d’attente une fois les messages supprimés, $queueMessage retourne NULL.

```powershell
# Set the amount of time you want to entry to be invisible after read from the queue
# If it is not deleted by the end of this time, it will show up in the queue again
$invisibleTimeout = [System.TimeSpan]::FromSeconds(10)

# Read the message from the queue, then show the contents of the message. Read the other two messages, too.
$queueMessage = $queue.CloudQueue.GetMessageAsync($invisibleTimeout,$null,$null)
$queueMessage.Result
$queueMessage = $queue.CloudQueue.GetMessageAsync($invisibleTimeout,$null,$null)
$queueMessage.Result
$queueMessage = $queue.CloudQueue.GetMessageAsync($invisibleTimeout,$null,$null)
$queueMessage.Result

# After 10 seconds, these messages reappear on the queue.
# Read them again, but delete each one after reading it.
# Delete the message.
$queueMessage = $queue.CloudQueue.GetMessageAsync($invisibleTimeout,$null,$null)
$queueMessage.Result
$queue.CloudQueue.DeleteMessageAsync($queueMessage.Result.Id,$queueMessage.Result.popReceipt)
$queueMessage = $queue.CloudQueue.GetMessageAsync($invisibleTimeout,$null,$null)
$queueMessage.Result
$queue.CloudQueue.DeleteMessageAsync($queueMessage.Result.Id,$queueMessage.Result.popReceipt)
$queueMessage = $queue.CloudQueue.GetMessageAsync($invisibleTimeout,$null,$null)
$queueMessage.Result
$queue.CloudQueue.DeleteMessageAsync($queueMessage.Result.Id,$queueMessage.Result.popReceipt)
```

## <a name="delete-a-queue"></a>Suppression d'une file d'attente

Pour supprimer une file d’attente et tous les messages qu’elle contient, appelez l’applet de commande Remove-AzStorageQueue. L’exemple suivant montre comment supprimer la file d’attente utilisée dans cet exercice à l’aide de l’applet de commande Remove-AzStorageQueue.

```powershell
# Delete the queue
Remove-AzStorageQueue –Name $queueName –Context $ctx
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer toutes les ressources que vous avez créées dans cet exercice, supprimez le groupe de ressources. Cette opération supprime également toutes les ressources contenues dans le groupe. Dans le cas présent, le compte de stockage créé et le groupe de ressources sont supprimés.

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Cet article sur les procédures vous a présenté les bases de la gestion de Stockage File d’attente avec PowerShell. Vous avez notamment appris à effectuer les tâches suivantes :

> [!div class="checklist"]
>
> - Créer une file d’attente
> - Récupérer une file d’attente
> - Ajouter un message
> - Lire le message suivant
> - Supprimer un message
> - Suppression d'une file d'attente

### <a name="microsoft-azure-powershell-storage-cmdlets"></a>Applets de commande Microsoft Azure PowerShell - Stockage

- [Applets de commande PowerShell - Stockage](/powershell/module/az.storage)

### <a name="microsoft-azure-storage-explorer"></a>Explorateur Stockage Microsoft Azure

- [Microsoft Azure Storage Explorer](../../vs-azure-tools-storage-manage-with-storage-explorer.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json) est une application autonome et gratuite de Microsoft qui vous permet d’exploiter visuellement les données de Stockage Azure sur Windows, macOS et Linux.
