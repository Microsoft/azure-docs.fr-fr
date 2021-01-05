---
title: Supprimer les blobs de stockage pour les journaux de flux des groupes de sécurité réseau dans Azure Network Watcher | Microsoft Docs
description: Cet article explique comment supprimer les blobs de stockage pour les journaux de flux des groupes de sécurité réseau qui sont en dehors de la période de la stratégie de conservation dans Azure Network Watcher.
services: network-watcher
documentationcenter: na
author: damendo
manager: ''
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/16/2019
ms.author: damendo
ms.openlocfilehash: 84f34c9441805352422215fcbac300bd900024d0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84737952"
---
# <a name="delete-network-security-group-flow-log-storage-blobs-in-network-watcher"></a>Supprimer les blobs de stockage pour les journaux de flux des groupes de sécurité réseau dans Network Watcher

Si vous avez besoin de supprimer manuellement les journaux de flux de votre compte de stockage, vous pouvez utiliser le script PowerShell ci-dessous.
Ce script supprime uniquement les objets blob de stockage antérieurs à la stratégie de conservation existante spécifiée par l’utilisateur.

## <a name="run-powershell-script-to-delete-nsg-flow-logs"></a>Exécuter un script PowerShell pour supprimer les journaux de flux des groupes de sécurité réseau
 
Copiez le script suivant et enregistrez-le à un emplacement tel que votre répertoire de travail actuel. 

```powershell
# This powershell script deletes all NSG flow log blobs that should not be retained anymore as per configured retention policy.
# While configuring NSG flow logs on Azure portal, the user configures the retention period of NSG flow log blobs in
# their storage account (in days).
# This script reads all blobs and deletes blobs that are not to be retained (outside retention window)
# if the retention days are zero; all blobs are retained forever and hence no blobs are deleted.
#
#

param (
        [string] [Parameter(Mandatory=$true)]  $SubscriptionId,
        [string] [Parameter(Mandatory=$true)]  $Location,
        [switch] [Parameter(Mandatory=$false)] $Confirm
    )

Login-AzAccount

$SubId = Get-AzSubscription| Where-Object {$_.Id.contains($SubscriptionId.ToLower())}

if ($SubId.Count -eq 0)
{
    Write-Error 'The SubscriptionId does not exist' -ErrorAction Stop
}

Set-AzContext -SubscriptionId $SubscriptionId

$NsgList = Get-AzNetworkSecurityGroup | Where-Object {$_.Location -eq $Location}
$NW = Get-AzNetworkWatcher | Where-Object {$_.Location -eq $Location}

$FlowLogsList = @()
foreach ($Nsg in $NsgList)
{
    # Query Flow Log Status which are enabled
    $NsgFlowLog = Get-AzNetworkWatcherFlowLogStatus -NetworkWatcher $NW -TargetResourceId $Nsg.Id | Where-Object {$_.Enabled -eq "True"}
    if ($NsgFlowLog.Count -gt 0)
    {
        $FlowLogsList +=  $NsgFlowLog
        Write-Output ('Enabled NSG found: ' +  $NsgFlowLog.TargetResourceId)
    }
}

foreach ($Psflowlog in $FlowLogsList)
{
    $RetentionDays = $Psflowlog.RetentionPolicy.Days
    if ($RetentionDays -le 0)
    {
        continue
    }

    $Strings = $Psflowlog.StorageId -split '/'
    $RGName = $Strings[4]
    $StorageAccountName = $Strings[-1]

    $Key = (Get-AzStorageAccountKey -ResourceGroupName $RGName -Name $StorageAccountName).Value[1]
    $StorageAccount = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $Key

    $ContainerName = 'insights-logs-networksecuritygroupflowevent'  
    $BLobsList = Get-AzStorageBlob -Container $ContainerName -Context $StorageAccount.Context

    $TargetBLobsList = $BLobsList | Where-Object {$_.Name.contains($Psflowlog.TargetResourceId.ToUpper())}

    $RetentionDate = Get-Date
    $RetentionDate = $RetentionDate.AddDays(-1*$RetentionDays)
    $RetentionDateInUTC = $RetentionDate.ToUniversalTime()

    foreach ($Blob in $TargetBLobsList)
    {
        $BlobLastModifietedDTinUTC = [datetime]$Blob.LastModified.UtcDateTime

        if ($BlobLastModifietedDTinUTC -ge  $RetentionDateInUTC)
        {
            Write-Output ($Blob.Name + '===>' + $BlobLastModifietedDTinUTC  + ' ===> RETAINED')
            continue
        }

        if ($Confirm)
        {
            Write-Output (Blob to be deleted: $Blob.Name)
            $Confirmation = Read-Host "Are you sure you want to remove this blob (Y/N)?"
        }

        if ((-not $Confirm) -or ($Confirmation -eq 'Y'))
        {
            Write-Output ($Blob.Name + '===>' + $BlobLastModifietedDTinUTC  + ' ===> DELETED')
            Remove-AzStorageBlob -Container $ContainerName -Context $StorageAccount.Context -Blob $Blob.Name
        }
        else
        {
            Write-Output ($Blob.Name + '===>' + $BlobLastModifietedDTinUTC  + ' ===> RETAINED')
        }
    }
}

Write-Output ('Retention policy for all NSGs evaluated and completed successfully')
```

1. Entrez les paramètres appropriés suivants dans le script :
   - **SubscriptionId** [Obligatoire] : ID de l’abonnement duquel vous souhaitez supprimer les blobs des journaux de flux des groupes de sécurité réseau.
   - **Location** [Obligatoire] : _chaîne d’emplacement_ de la région des groupes de sécurité réseau pour lesquels vous souhaitez supprimer les blobs de flux. Cette information est disponible sur le portail Azure ou sur [GitHub](https://github.com/Azure/azure-extensions-cli/blob/beb3d3fe984cfa9c7798cb11a274c5337968cbc5/regions.go#L23).
   - **Confirm** [Obligatoire] : passez l’indicateur Confirm si vous souhaitez confirmer manuellement la suppression de chaque blob de stockage.

1. Exécutez le script enregistré comme dans l’exemple suivant où le fichier de script a été enregistré sous le nom **Delete-NsgFlowLogsBlobs.ps1** :
   ```
   .\Delete-NsgFlowLogsBlobs.ps1 -SubscriptionId <subscriptionId> -Location  <location> -Confirm
   ```
    
## <a name="next-steps"></a>Étapes suivantes
- Les clients peuvent automatiser l’exécution du script avec [Azure Logic Apps](../logic-apps/logic-apps-overview.md) ou avec [Azure Automation](https://azure.microsoft.com/services/automation/)
- Pour en savoir plus sur la journalisation du Groupe de sécurité réseau, consultez [Journaux Azure Monitor des Groupes de sécurité réseau (NSG)](../virtual-network/virtual-network-nsg-manage-log.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json).

