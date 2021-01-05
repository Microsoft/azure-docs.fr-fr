---
title: Lire les journaux de flux de groupe de sécurité réseau | Documents Microsoft
description: Découvrez comment utiliser Azure PowerShell pour analyser les journaux de flux de groupe de sécurité réseau qui sont créés toutes les heures et mis à jour toutes les quelques minutes dans le service Network Watcher Azure.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/13/2017
ms.author: damendo
ms.openlocfilehash: 1f9b9e91cda93a986fdaaf0f53d8987544e783a2
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966460"
---
# <a name="read-nsg-flow-logs"></a>Lire des journaux de flux NSG

Découvrez comment lire les entrées de journaux de flux de groupe de sécurité réseau avec PowerShell.

Les journaux de flux de groupe de sécurité réseau sont stockés dans un compte de stockage dans de [objets blob de bloc](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs). Les objets blob de blocs sont composés de blocs plus petits. Chaque journal est un objet blob de blocs séparé généré toutes les heures. Les nouveaux journaux d’activité sont créés toutes les heures, les journaux d’activité sont mis à jour avec les nouvelles entrées toutes les quelques minutes avec les dernières données. Dans cet article vous allez apprendre à lire une partie des journaux de flux.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="scenario"></a>Scénario

Dans le scénario suivant, vous disposez d’un exemple de journal de flux stocké dans un compte de stockage. Vous découvrirez comment lire de manière sélective les derniers événements dans les journaux de flux de groupe de sécurité réseau. Vous utilisez PowerShell dans cet article, toutefois, les concepts abordés dans l’article ne sont pas limités au langage de programmation et sont applicables à tous les langages pris en charge par les API de stockage Azure.

## <a name="setup"></a>Programme d’installation

Avant de commencer, la journalisation des flux de groupe de sécurité réseau doit être activée sur un ou plusieurs groupes de sécurité réseau de votre compte. Pour obtenir des instructions sur l’activation des journaux de flux de sécurité réseau, consultez l’article suivant : [Introduction à la journalisation des flux pour les groupes de sécurité réseau](network-watcher-nsg-flow-logging-overview.md).

## <a name="retrieve-the-block-list"></a>Récupérer la liste de blocs

La commande PowerShell suivante définit les variables nécessaires pour interroger l’objet blob des journaux de flux de groupe de sécurité réseau et répertorier les blocs dans l’objet blob de blocs [CloudBlockBlob](/dotnet/api/microsoft.azure.storage.blob.cloudblockblob). Mettez à jour le script pour contenir des valeurs valides pour votre environnement.

```powershell
function Get-NSGFlowLogCloudBlockBlob {
    [CmdletBinding()]
    param (
        [string] [Parameter(Mandatory=$true)] $subscriptionId,
        [string] [Parameter(Mandatory=$true)] $NSGResourceGroupName,
        [string] [Parameter(Mandatory=$true)] $NSGName,
        [string] [Parameter(Mandatory=$true)] $storageAccountName,
        [string] [Parameter(Mandatory=$true)] $storageAccountResourceGroup,
        [string] [Parameter(Mandatory=$true)] $macAddress,
        [datetime] [Parameter(Mandatory=$true)] $logTime
    )

    process {
        # Retrieve the primary storage account key to access the NSG logs
        $StorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $storageAccountResourceGroup -Name $storageAccountName).Value[0]

        # Setup a new storage context to be used to query the logs
        $ctx = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

        # Container name used by NSG flow logs
        $ContainerName = "insights-logs-networksecuritygroupflowevent"

        # Name of the blob that contains the NSG flow log
        $BlobName = "resourceId=/SUBSCRIPTIONS/${subscriptionId}/RESOURCEGROUPS/${NSGResourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/${NSGName}/y=$($logTime.Year)/m=$(($logTime).ToString("MM"))/d=$(($logTime).ToString("dd"))/h=$(($logTime).ToString("HH"))/m=00/macAddress=$($macAddress)/PT1H.json"

        # Gets the storage blog
        $Blob = Get-AzStorageBlob -Context $ctx -Container $ContainerName -Blob $BlobName

        # Gets the block blog of type 'Microsoft.Azure.Storage.Blob.CloudBlob' from the storage blob
        $CloudBlockBlob = [Microsoft.Azure.Storage.Blob.CloudBlockBlob] $Blob.ICloudBlob

        #Return the Cloud Block Blob
        $CloudBlockBlob
    }
}

function Get-NSGFlowLogBlockList  {
    [CmdletBinding()]
    param (
        [Microsoft.Azure.Storage.Blob.CloudBlockBlob] [Parameter(Mandatory=$true)] $CloudBlockBlob
    )
    process {
        # Stores the block list in a variable from the block blob.
        $blockList = $CloudBlockBlob.DownloadBlockListAsync()

        # Return the Block List
        $blockList
    }
}


$CloudBlockBlob = Get-NSGFlowLogCloudBlockBlob -subscriptionId "yourSubscriptionId" -NSGResourceGroupName "FLOWLOGSVALIDATIONWESTCENTRALUS" -NSGName "V2VALIDATIONVM-NSG" -storageAccountName "yourStorageAccountName" -storageAccountResourceGroup "ml-rg" -macAddress "000D3AF87856" -logTime "11/11/2018 03:00" 

$blockList = Get-NSGFlowLogBlockList -CloudBlockBlob $CloudBlockBlob
```

La variable `$blockList` renvoie une liste des blocs dans l’objet blob. Chaque objet blob de blocs contient au moins deux blocs.  Le premier bloc a une longueur de `12` octets, ce bloc contient les crochets d’ouverture du journal json. L’autre bloc contient les crochets fermant avec une longueur de `2` octets.  Comme vous pouvez le constater, le journal d’exemple suivant comporte sept entrées, chacune étant une entrée individuelle. Toutes les nouvelles entrées dans le journal sont ajoutées à la fin, juste avant le dernier bloc.

```
Name                                         Length Committed
----                                         ------ ---------
ZDk5MTk5N2FkNGE0MmY5MTk5ZWViYjA0YmZhODRhYzY=     12      True
NzQxNDA5MTRhNDUzMGI2M2Y1MDMyOWZlN2QwNDZiYzQ=   2685      True
ODdjM2UyMWY3NzFhZTU3MmVlMmU5MDNlOWEwNWE3YWY=   2586      True
ZDU2MjA3OGQ2ZDU3MjczMWQ4MTRmYWNhYjAzOGJkMTg=   2688      True
ZmM3ZWJjMGQ0ZDA1ODJlOWMyODhlOWE3MDI1MGJhMTc=   2775      True
ZGVkYTc4MzQzNjEyMzlmZWE5MmRiNjc1OWE5OTc0OTQ=   2676      True
ZmY2MjUzYTIwYWIyOGU1OTA2ZDY1OWYzNmY2NmU4ZTY=   2777      True
Mzk1YzQwM2U0ZWY1ZDRhOWFlMTNhYjQ3OGVhYmUzNjk=   2675      True
ZjAyZTliYWE3OTI1YWZmYjFmMWI0MjJhNzMxZTI4MDM=      2      True
```

## <a name="read-the-block-blob"></a>Lire l’objet blob de blocs

Ensuite, vous avez besoin de lire la variable `$blocklist` pour récupérer les données. Dans cet exemple que nous itérons dans la liste de blocs, lisez les octets de chaque bloc et estimez-les dans un tableau. Utilisez la méthode [DownloadRangeToByteArray](/dotnet/api/microsoft.azure.storage.blob.cloudblob.downloadrangetobytearray) pour récupérer les données.

```powershell
function Get-NSGFlowLogReadBlock  {
    [CmdletBinding()]
    param (
        [System.Array] [Parameter(Mandatory=$true)] $blockList,
        [Microsoft.Azure.Storage.Blob.CloudBlockBlob] [Parameter(Mandatory=$true)] $CloudBlockBlob

    )
    # Set the size of the byte array to the largest block
    $maxvalue = ($blocklist | measure Length -Maximum).Maximum

    # Create an array to store values in
    $valuearray = @()

    # Define the starting index to track the current block being read
    $index = 0

    # Loop through each block in the block list
    for($i=0; $i -lt $blocklist.count; $i++)
    {
        # Create a byte array object to story the bytes from the block
        $downloadArray = New-Object -TypeName byte[] -ArgumentList $maxvalue

        # Download the data into the ByteArray, starting with the current index, for the number of bytes in the current block. Index is increased by 3 when reading to remove preceding comma.
        $CloudBlockBlob.DownloadRangeToByteArray($downloadArray,0,$index, $($blockList[$i].Length)) | Out-Null

        # Increment the index by adding the current block length to the previous index
        $index = $index + $blockList[$i].Length

        # Retrieve the string from the byte array

        $value = [System.Text.Encoding]::ASCII.GetString($downloadArray)

        # Add the log entry to the value array
        $valuearray += $value
    }
    #Return the Array
    $valuearray
}
$valuearray = Get-NSGFlowLogReadBlock -blockList $blockList -CloudBlockBlob $CloudBlockBlob
```

Maintenant le tableau `$valuearray` contient la valeur de chaîne de chaque bloc. Pour vérifier l’entrée, enregistrez le deuxième à la dernière valeur du tableau en exécutant `$valuearray[$valuearray.Length-2]`. Vous n’avez pas besoin de la dernière valeur car il s’agit du crochet fermant.

Les résultats de cette valeur sont affichés dans l’exemple suivant :

```json
        {
             "time": "2017-06-16T20:59:43.7340000Z",
             "systemId": "5f4d02d3-a7d0-4ed4-9ce8-c0ae9377951c",
             "category": "NetworkSecurityGroupFlowEvent",
             "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/CONTOSORG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/CONTOSONSG",
             "operationName": "NetworkSecurityGroupFlowEvents",
             "properties": {"Version":1,"flows":[{"rule":"DefaultRule_AllowInternetOutBound","flows":[{"mac":"000D3A18077E","flowTuples":["1497646722,10.0.0.4,168.62.32.14,44904,443,T,O,A","1497646722,10.0.0.4,52.240.48.24,45218,443,T,O,A","1497646725,10.
0.0.4,168.62.32.14,44910,443,T,O,A","1497646725,10.0.0.4,52.240.48.24,45224,443,T,O,A","1497646728,10.0.0.4,168.62.32.14,44916,443,T,O,A","1497646728,10.0.0.4,52.240.48.24,45230,443,T,O,A","1497646732,10.0.0.4,168.62.32.14,44922,443,T,O,A","14976
46732,10.0.0.4,52.240.48.24,45236,443,T,O,A","1497646735,10.0.0.4,168.62.32.14,44928,443,T,O,A","1497646735,10.0.0.4,52.240.48.24,45242,443,T,O,A","1497646738,10.0.0.4,168.62.32.14,44934,443,T,O,A","1497646738,10.0.0.4,52.240.48.24,45248,443,T,O,
A","1497646742,10.0.0.4,168.62.32.14,44942,443,T,O,A","1497646742,10.0.0.4,52.240.48.24,45256,443,T,O,A","1497646745,10.0.0.4,168.62.32.14,44948,443,T,O,A","1497646745,10.0.0.4,52.240.48.24,45262,443,T,O,A","1497646749,10.0.0.4,168.62.32.14,44954
,443,T,O,A","1497646749,10.0.0.4,52.240.48.24,45268,443,T,O,A","1497646753,10.0.0.4,168.62.32.14,44960,443,T,O,A","1497646753,10.0.0.4,52.240.48.24,45274,443,T,O,A","1497646756,10.0.0.4,168.62.32.14,44966,443,T,O,A","1497646756,10.0.0.4,52.240.48
.24,45280,443,T,O,A","1497646759,10.0.0.4,168.62.32.14,44972,443,T,O,A","1497646759,10.0.0.4,52.240.48.24,45286,443,T,O,A","1497646763,10.0.0.4,168.62.32.14,44978,443,T,O,A","1497646763,10.0.0.4,52.240.48.24,45292,443,T,O,A","1497646766,10.0.0.4,
168.62.32.14,44984,443,T,O,A","1497646766,10.0.0.4,52.240.48.24,45298,443,T,O,A","1497646769,10.0.0.4,168.62.32.14,44990,443,T,O,A","1497646769,10.0.0.4,52.240.48.24,45304,443,T,O,A","1497646773,10.0.0.4,168.62.32.14,44996,443,T,O,A","1497646773,
10.0.0.4,52.240.48.24,45310,443,T,O,A","1497646776,10.0.0.4,168.62.32.14,45002,443,T,O,A","1497646776,10.0.0.4,52.240.48.24,45316,443,T,O,A","1497646779,10.0.0.4,168.62.32.14,45008,443,T,O,A","1497646779,10.0.0.4,52.240.48.24,45322,443,T,O,A"]}]}
,{"rule":"DefaultRule_DenyAllInBound","flows":[]},{"rule":"UserRule_ssh-rule","flows":[]},{"rule":"UserRule_web-rule","flows":[{"mac":"000D3A18077E","flowTuples":["1497646738,13.82.225.93,10.0.0.4,1180,80,T,I,A","1497646750,13.82.225.93,10.0.0.4,
1184,80,T,I,A","1497646768,13.82.225.93,10.0.0.4,1181,80,T,I,A","1497646780,13.82.225.93,10.0.0.4,1336,80,T,I,A"]}]}]}
        }
```

Ce scénario est un exemple montrant comment lire les entrées dans les journaux de flux de groupe de sécurité réseau sans avoir à analyser l’ensemble du journal. Vous pouvez lire les nouvelles entrées dans le journal comme elles sont écrites à l’aide de l’ID de bloc ou en effectuant le suivi de la longueur des blocs stockés dans l’objet blob de blocs. Cela vous permet de lire uniquement les nouvelles entrées.

## <a name="next-steps"></a>Étapes suivantes


Visitez [Utiliser Elasic Stack](network-watcher-visualize-nsg-flow-logs-open-source-tools.md), [Utiliser Grafana](network-watcher-nsg-grafana.md) et [Utiliser Graylog](network-watcher-analyze-nsg-flow-logs-graylog.md) pour en savoir plus sur les façons de visualiser les journaux de flux NSG. Une approche Azure Function Open permettant de consommer directement les blobs et d’émettre vers les différents consommateurs de l’analytique des journaux d’activité est décrite ici : [Connecteur des journaux de flux de groupe de sécurité réseau Azure Network Watcher](https://github.com/Microsoft/AzureNetworkWatcherNSGFlowLogsConnector).

Vous pouvez utiliser [Azure Traffic Analytics](./traffic-analytics.md) pour obtenir des insights sur vos flux de trafic. Traffic Analytics utilise [Log Analytics](../azure-monitor/log-query/log-analytics-tutorial.md) pour rendre votre flux de trafic interrogeable.

Pour en savoir plus sur les objets blob de stockage, visitez : [Liaisons de stockage Blob Azure Functions](../azure-functions/functions-bindings-storage-blob.md)