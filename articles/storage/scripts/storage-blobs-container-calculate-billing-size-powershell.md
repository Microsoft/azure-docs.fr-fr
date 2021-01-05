---
title: Exemple de script Azure PowerShell - Calculer la taille de facturation totale d’un conteneur d’objets blob | Microsoft Docs
description: Calculez la taille totale d’un conteneur dans le stockage d’objets blob Azure pour la facturation.
services: storage
author: fhryo-msft
ms.service: storage
ms.subservice: blobs
ms.devlang: powershell
ms.topic: sample
ms.date: 11/07/2017
ms.author: fryu
ms.openlocfilehash: 2d921a968f50f64788ccbd7637bc04c8492a3f90
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96010892"
---
# <a name="calculate-the-total-billing-size-of-a-blob-container"></a>Calculer la taille totale de facturation d’un conteneur d’objets blob

Ce script calcule la taille d’un conteneur dans le stockage d’objets blob Azure à des fins d’estimation des coûts de facturation. Le script additionne la taille des objets blob qui se trouvent dans le conteneur.

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

> [!NOTE]
> Ce script PowerShell calcule la taille d’un conteneur pour la facturation. Si vous calculez la taille du conteneur à d’autres fins, consultez [Calculer la taille totale d’un conteneur de stockage d’objets blob](../scripts/storage-blobs-container-calculate-size-powershell.md) pour un script simple qui fournit une estimation.

## <a name="determine-the-size-of-the-blob-container"></a>Déterminer la taille totale du conteneur d’objets blob

La taille totale du conteneur d’objets blob correspond à la taille du conteneur lui-même et de celle de tous les objets blob qu’il contient.

Les sections suivantes décrivent comment calculer la capacité de stockage pour les conteneurs d’objets blob et les objets blob.  Dans la section suivante, Len(X) désigne le nombre de caractères dans la chaîne.

### <a name="blob-containers"></a>Conteneurs d’objets blob

Le calcul suivant montre comment estimer la quantité de stockage utilisée par le conteneur d’objets blob :

```
48 bytes + Len(ContainerName) * 2 bytes +
For-Each Metadata[3 bytes + Len(MetadataName) + Len(Value)] +
For-Each Signed Identifier[512 bytes]
```

Voici la répartition :
* 48 octets de charge utile pour chaque conteneur comprenant l’heure de dernière modification, les autorisations, les paramètres publics et des métadonnées système.

* Le nom du conteneur est stocké au format Unicode. Vous devez donc prendre le nombre de caractères et le multiplier par deux.

* Pour chaque bloc de métadonnées du conteneur d’objets blob qui est stocké, nous stockons la longueur du nom (ASCII) et la longueur de la valeur de chaîne.

* Les 512 octets par identificateur de signature comprennent le nom de l’identificateur signé, l’heure de début, l’heure d’expiration et les autorisations.

### <a name="blobs"></a>Objets blob

Les calculs suivants montrent comment estimer la quantité de stockage utilisée par chaque objet blob.

* Objet blob de blocs (objet blob de base ou instantané) :

   ```
   124 bytes + Len(BlobName) * 2 bytes +
   For-Each Metadata[3 bytes + Len(MetadataName) + Len(Value)] +
   8 bytes + number of committed and uncommitted blocks * Block ID Size in bytes +
   SizeInBytes(data in unique committed data blocks stored) +
   SizeInBytes(data in uncommitted data blocks)
   ```

* Objet blob de pages (objet blob de base ou instantané) :

   ```
   124 bytes + Len(BlobName) * 2 bytes +
   For-Each Metadata[3 bytes + Len(MetadataName) + Len(Value)] +
   number of nonconsecutive page ranges with data * 12 bytes +
   SizeInBytes(data in unique pages stored)
   ```

Voici la répartition :

* 124 octets de surcharge pour l’objet blob, qui inclut les éléments suivants :
    - Heure de la dernière modification
    - Taille
    - Cache-Control
    - Content-Type
    - Content-Language
    - Content-Encoding
    - Content-MD5
    - Autorisations
    - Informations d’instantané
    - Lease
    - Certaines métadonnées système

* Le nom de l’objet blob est stocké au format Unicode. Vous devez donc prendre le nombre de caractères et le multiplier par deux.

* Pour chaque bloc de métadonnées stocké, ajoutez la longueur du nom (au format ASCII) et la longueur de la valeur de chaîne.

* Pour les objets blob de blocs :
  * Huit octets pour la liste de blocs.
  * Nombre de blocs multiplié par la taille de l’ID de bloc en octets.
  * Taille des données dans tous les blocs validés et non validés.

    >[!NOTE]
    >Quand des instantanés sont utilisés, cette taille inclut seulement les données uniques pour cet objet blob de base ou instantané. Si les blocs non validés ne sont pas utilisés après une semaine, ils sont supprimés de la mémoire. Après cela, ils ne sont plus pris en compte dans la facturation.

* Pour les objets blob de pages :
  * Nombre de plages de pages non consécutives comportant des données multiplié par 12 octets. Il s’agit du nombre de plages de pages uniques que vous voyez quand vous appelez l’API **GetPageRanges**.

  * Taille des données en octets de toutes les pages stockées.

    >[!NOTE]
    >Quand des instantanés sont utilisés, cette taille inclut seulement les pages uniques pour l’objet blob de base ou l’objet blob instantané qui est comptabilisé.

## <a name="sample-script"></a>Exemple de script

[!code-powershell[main](../../../powershell_scripts/storage/calculate-container-size/calculate-container-size-ex.ps1 "Calculate container size")]

## <a name="next-steps"></a>Étapes suivantes

- Consultez [Calculer la taille totale d’un conteneur de stockage d’objets blob](../scripts/storage-blobs-container-calculate-size-powershell.md) pour un script simple qui fournit une estimation de la taille du conteneur.

- Pour plus d’informations sur la facturation du Stockage Azure, consultez la page relative à la [facturation du Stockage Microsoft Azure](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/07/08/understanding-windows-azure-storage-billing-bandwidth-transactions-and-capacity/).

- Pour plus d’informations sur le module Azure PowerShell, consultez [Documentation Azure PowerShell](/powershell/azure/).

- Vous trouverez d’autres exemples de scripts PowerShell de stockage dans les [exemples PowerShell pour le Stockage Azure](../blobs/storage-samples-blobs-powershell.md).