---
title: Charger un disque dur virtuel sur Azure ou copier un disque d'une région à une autre - Azure CLI
description: Apprenez à charger un disque dur virtuel sur un disque managé Azure et à copier un disque managé d'une région à une autre à l'aide d'Azure CLI, via un chargement direct.
services: virtual-machines,storage
author: roygara
ms.author: rogarana
ms.date: 06/15/2020
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: disks
ms.openlocfilehash: 473e87904742395eca6b7eeba0875cd93789104d
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91978983"
---
# <a name="upload-a-vhd-to-azure-or-copy-a-managed-disk-to-another-region---azure-cli"></a>Charger un disque dur virtuel sur Azure ou copier un disque dans une autre région - Azure CLI

[!INCLUDE [disks-upload-vhd-to-disk-intro](../../../includes/disks-upload-vhd-to-disk-intro.md)]

## <a name="prerequisites"></a>Prérequis

- Téléchargez la dernière [version d’AzCopy v10](../../storage/common/storage-use-azcopy-v10.md#download-and-install-azcopy).
- [Installer l’interface de ligne de commande Microsoft Azure](/cli/azure/install-azure-cli).
- Si vous envisagez de charger un VHD à partir d’un emplacement local : Un VHD de taille fixe qui a été [préparé pour Azure](../windows/prepare-for-upload-vhd-image.md) et stocké localement.
- Ou un disque managé dans Azure, si vous envisagez d’effectuer une action de copie.

## <a name="getting-started"></a>Prise en main

Si vous préférez charger des disques via une interface graphique utilisateur, vous pouvez utiliser l’Explorateur Stockage Azure. Pour plus d’informations, consultez : [Utiliser l’Explorateur Stockage Azure pour gérer des disques managés Azure](../disks-use-storage-explorer-managed-disks.md)

Pour charger votre disque dur virtuel sur Azure, vous devez créer un disque managé vide configuré pour ce processus de chargement. Avant de créer ce disque, vous devez prendre connaissance des informations ci-dessous.

Ce type de disque managé présente deux états uniques :

- ReadToUpload, qui signifie que le disque est prêt à recevoir un chargement, mais qu'aucune [signature d'accès partagé](../../storage/common/storage-sas-overview.md) (SAS) n'a été générée.
- ActiveUpload, qui signifie que le disque est prêt à recevoir un chargement et que la SAS a été générée.

> [!NOTE]
> Dans l'un ou l'autre de ces états, le disque managé est facturé au [tarif HDD Standard](https://azure.microsoft.com/pricing/details/managed-disks/), quel que soit le type de disque. Par exemple, un P10 est facturé comme un S10. Ceci s’applique jusqu’à ce que `revoke-access` soit appelé sur le disque managé, ce qui est nécessaire pour joindre le disque à une machine virtuelle.

## <a name="create-an-empty-managed-disk"></a>Créer un disque managé vierge

Avant de pouvoir créer un disque HDD standard vierge pour le chargement, vous devez connaître la taille de fichier (en octets) du disque dur virtuel que vous souhaitez charger. Pour l’obtenir, vous pouvez utiliser `wc -c <yourFileName>.vhd` ou `ls -al <yourFileName>.vhd`. Cette valeur est utilisée lors de la spécification du paramètre **--upload-size-bytes**.

Créez un disque dur standard vierge pour le chargement en spécifiant les paramètres **-–for-upload** et **--upload-size-bytes** dans une applet de commande [disk create](/cli/azure/disk#az-disk-create) :

Remplacez `<yourdiskname>`, `<yourresourcegroupname>`, `<yourregion>` par les valeurs de votre choix. Le paramètre `--upload-size-bytes` contient un exemple de valeur : `34359738880`. Remplacez cet exemple par la valeur appropriée.

> [!TIP]
> Si vous créez un disque de système d’exploitation, ajoutez --hyper-v-generation <yourGeneration> à `az disk create`.

```azurecli
az disk create -n <yourdiskname> -g <yourresourcegroupname> -l <yourregion> --for-upload --upload-size-bytes 34359738880 --sku standard_lrs
```

Si vous souhaitez charger un disque SSD Premium ou Standard, remplacez **standard_LRS** par **premium_LRS** ou **standardssd_lrs**. Les disques Ultra ne sont pas pris en charge pour le moment.

Maintenant que vous avez créé un disque managé vierge configuré pour le processus de chargement, vous pouvez y charger un disque dur virtuel. Pour charger un disque dur virtuel sur le disque, il vous faut une signature d'accès partagé (SAP) accessible en écriture afin de pouvoir la référencer en tant que destination pour votre chargement.

Pour générer une SAP accessible en écriture de votre disque managé vierge, remplacez `<yourdiskname>` et `<yourresourcegroupname>`, puis utilisez la commande suivante :

```azurecli
az disk grant-access -n <yourdiskname> -g <yourresourcegroupname> --access-level Write --duration-in-seconds 86400
```

Exemple de valeur renvoyée :

```output
{
  "accessSas": "https://md-impexp-t0rdsfgsdfg4.blob.core.windows.net/w2c3mj0ksfgl/abcd?sv=2017-04-17&sr=b&si=600a9281-d39e-4cc3-91d2-923c4a696537&sig=xXaT6mFgf139ycT87CADyFxb%2BnPXBElYirYRlbnJZbs%3D"
}
```

## <a name="upload-a-vhd"></a>Télécharger un disque dur virtuel

Maintenant que vous disposez d'une SAS pour votre disque managé vierge, vous pouvez l'utiliser pour définir votre disque managé en tant que destination de votre commande de chargement.

Utilisez AzCopy v10 pour charger votre fichier de disque dur virtuel local sur un disque managé en spécifiant l'URI de la SAS que vous avez générée.

Ce chargement présente le même débit que le disque [HDD Standard](../disks-types.md#standard-hdd) correspondant. Par exemple, pour une taille correspondant à S4, vous aurez un débit allant jusqu'à 60 Mio/s. Mais pour une taille correspondant à S70, le débit ira jusqu’à 500 Mio/s.

```bash
AzCopy.exe copy "c:\somewhere\mydisk.vhd" "sas-URI" --blob-type PageBlob
```

Lorsque vous avez terminé le chargement et que vous n’avez plus rien à écrire sur le disque, révoquez la SAS. La révocation de la SAS changera l'état du disque managé et vous permettra de le joindre à une machine virtuelle.

Remplacez `<yourdiskname>`et `<yourresourcegroupname>`, puis utilisez la commande suivante pour rendre le disque utilisable :

```azurecli
az disk revoke-access -n <yourdiskname> -g <yourresourcegroupname>
```

## <a name="copy-a-managed-disk"></a>Copier un disque managé

Le chargement direct simplifie également le processus de copie d’un disque managé. Vous pouvez effectuer une copie au sein d’une même région ou entre des régions différentes (vers une autre région).

Le script suivant effectuera cette opération pour vous. Le processus est similaire aux étapes décrites précédemment, à quelques différences près puisque vous utilisez un disque existant.

> [!IMPORTANT]
> Vous devez ajouter un décalage de 512 quand vous fournissez la taille en octets d’un disque managé d’Azure. En effet, Azure omet le pied de page lors du retour de la taille du disque. Si vous ne le faites pas, la copie échouera. Le script suivant s’en charge pour vous.

Remplacez `<sourceResourceGroupHere>`, `<sourceDiskNameHere>`, `<targetDiskNameHere>`, `<targetResourceGroupHere>` et `<yourTargetLocationHere>` (la valeur d’emplacement pourrait par exemple être uswest2) par vos valeurs, puis exécutez le script suivant afin de copier un disque managé.

> [!TIP]
> Si vous créez un disque de système d’exploitation, ajoutez --hyper-v-generation <yourGeneration> à `az disk create`.

```azurecli
sourceDiskName=<sourceDiskNameHere>
sourceRG=<sourceResourceGroupHere>
targetDiskName=<targetDiskNameHere>
targetRG=<targetResourceGroupHere>
targetLocation=<yourTargetLocationHere>

sourceDiskSizeBytes=$(az disk show -g $sourceRG -n $sourceDiskName --query '[diskSizeBytes]' -o tsv)

az disk create -g $targetRG -n $targetDiskName -l $targetLocation --for-upload --upload-size-bytes $(($sourceDiskSizeBytes+512)) --sku standard_lrs

targetSASURI=$(az disk grant-access -n $targetDiskName -g $targetRG  --access-level Write --duration-in-seconds 86400 -o tsv)

sourceSASURI=$(az disk grant-access -n $sourceDiskName -g $sourceRG --duration-in-seconds 86400 --query [accessSas] -o tsv)

.\azcopy copy $sourceSASURI $targetSASURI --blob-type PageBlob

az disk revoke-access -n $sourceDiskName -g $sourceRG

az disk revoke-access -n $targetDiskName -g $targetRG
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez réussi à charger un disque dur virtuel sur un disque managé, vous pouvez joindre le disque en tant que [disque de données à une machine virtuelle existante](add-disk.md) ou [joindre le disque à une machine virtuelle en tant que disque de système d'exploitation](upload-vhd.md#create-the-vm) pour créer une nouvelle machine virtuelle.