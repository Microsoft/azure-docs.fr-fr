---
title: Fichier Include
description: Fichier Include
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 03/05/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 4bdeef537556db94338ed50fcfa6e9d88431f25a
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96016253"
---
[!INCLUDE [virtual-machines-disks-incremental-snapshots-description](virtual-machines-disks-incremental-snapshots-description.md)]

## <a name="restrictions"></a>Restrictions

[!INCLUDE [virtual-machines-disks-incremental-snapshots-restrictions](virtual-machines-disks-incremental-snapshots-restrictions.md)]

## <a name="cli"></a>Interface de ligne de commande

Vous pouvez créer un instantané incrémentiel avec Azure CLI (dans ce cas, il vous faut la dernière version d’Azure CLI). 

Sur Windows, la commande suivante vous permet de l’installer ou de mettre à jour votre installation existante avec la dernière version :
```PowerShell
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'
```
Sur Linux, l’installation de l’interface de ligne de commande varie en fonction de la version du système d’exploitation.  Consultez [Installer l’interface de ligne de commande Microsoft Azure](/cli/azure/install-azure-cli) pour votre version Linux spécifique.

Pour créer un instantané incrémentiel, utilisez [az snapshot create](/cli/azure/snapshot?view=azure-cli-latest#az-snapshot-create) avec le paramètre `--incremental`.

L’exemple suivant crée un instantané incrémentiel. Remplacez `<yourDesiredSnapShotNameHere>`, `<yourResourceGroupNameHere>`, `<exampleDiskName>` et `<exampleLocation>` par vos propres valeurs, puis exécutez l’exemple :

```bash
sourceResourceId=$(az disk show -g <yourResourceGroupNameHere> -n <exampleDiskName> --query '[id]' -o tsv)

az snapshot create -g <yourResourceGroupNameHere> \
-n <yourDesiredSnapShotNameHere> \
-l <exampleLocation> \
--source "$sourceResourceId" \
--incremental
```

Vous pouvez identifier des instantanés incrémentiels à partir du même disque avec les propriétés `SourceResourceId` et `SourceUniqueId` des instantanés. `SourceResourceId` est l’ID de ressource Azure Resource Manager du disque parent. `SourceUniqueId` est la valeur héritée de la propriété `UniqueId` du disque. Si vous supprimez un disque et que vous créez ensuite un disque portant le même nom, la valeur de la propriété `UniqueId` change.

Vous pouvez utiliser `SourceResourceId` et `SourceUniqueId` pour créer une liste de tous les instantanés associés à un disque particulier. L’exemple suivant liste tous les instantanés incrémentiels associés à un disque particulier, mais il nécessite une certaine configuration.

Cet exemple utilise jq pour interroger les données. Pour exécuter l’exemple, vous devez [installer jq](https://stedolan.github.io/jq/download/).

Remplacez `<yourResourceGroupNameHere>` et `<exampleDiskName>` par vos valeurs, puis utilisez l’exemple suivant pour lister vos instantanés incrémentiels existants (à condition que jq soit aussi installé) :

```bash
sourceUniqueId=$(az disk show -g <yourResourceGroupNameHere> -n <exampleDiskName> --query '[uniqueId]' -o tsv)

 
sourceResourceId=$(az disk show -g <yourResourceGroupNameHere> -n <exampleDiskName> --query '[id]' -o tsv)

az snapshot list -g <yourResourceGroupNameHere> -o json \
| jq -cr --arg SUID "$sourceUniqueId" --arg SRID "$sourceResourceId" '.[] | select(.incremental==true and .creationData.sourceUniqueId==$SUID and .creationData.sourceResourceId==$SRID)'
```

## <a name="resource-manager-template"></a>Modèle Resource Manager

Vous pouvez également utiliser des modèles Azure Resource Manager pour créer un instantané incrémentiel. Vous devez vous assurer que le paramètre apiVersion est défini sur **2019-03-01** et que la propriété incremental est définie sur true. L'extrait de code suivant montre comment créer un instantané incrémentiel à l'aide de modèles Resource Manager :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "diskName": {
      "type": "string",
      "defaultValue": "contosodisk1"
    },
  "diskResourceId": {
    "defaultValue": "<your_managed_disk_resource_ID>",
    "type": "String"
  }
  }, 
  "resources": [
  {
    "type": "Microsoft.Compute/snapshots",
    "name": "[concat( parameters('diskName'),'_snapshot1')]",
    "location": "[resourceGroup().location]",
    "apiVersion": "2019-03-01",
    "properties": {
      "creationData": {
        "createOption": "Copy",
        "sourceResourceId": "[parameters('diskResourceId')]"
      },
      "incremental": true
    }
  }
  ]
}
```

## <a name="next-steps"></a>Étapes suivantes

Si vous souhaitez voir un exemple de code illustrant la capacité différentielle des instantanés incrémentiels à l’aide de .NET, consultez [Copy Azure Managed Disks backups to another region with differential capability of incremental snapshots](https://github.com/Azure-Samples/managed-disks-dotnet-backup-with-incremental-snapshots) (Copier des sauvegardes de disques managés Azure dans une autre région avec la fonctionnalité différentielle des instantanés incrémentiels).