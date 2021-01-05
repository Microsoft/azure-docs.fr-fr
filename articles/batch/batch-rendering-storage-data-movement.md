---
title: Stockage et déplacement des données pour le rendu
description: En savoir plus sur les différentes options de déplacement de données et de stockage pour les charges de travail des fichiers d’éléments multimédias et de sorties destinés au rendu.
services: batch
ms.service: batch
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: how-to
ms.openlocfilehash: 55ec04df2a107dabfc72298bc8849c13f3a926e0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86147302"
---
# <a name="storage-and-data-movement-options-for-rendering-asset-and-output-files"></a>Options de stockage et de déplacement des données pour les fichiers d’éléments multimédias et de sortie destinés au rendu

Plusieurs options permettent de rendre les fichiers de scènes et d’éléments multimédias disponibles pour les applications de rendu sur les machines virtuelles du pool :

* [Stockage Blob Azure](../storage/blobs/storage-blobs-introduction.md) :
  * Les fichiers de scènes et d’éléments multimédias sont chargés vers le stockage d’objets blob à partir d’un système de fichiers local. Lorsque l’application est exécutée par une tâche, les fichiers requis sont copiés à partir du stockage d’objets blob sur la machine virtuelle. Ils sont ainsi accessibles à l’application de rendu. Les fichiers de sortie sont écrits par l’application de rendu sur le disque de la machine virtuelle, puis copiés dans le stockage d’objets blob.  Si nécessaire, les fichiers de sortie peuvent être téléchargés à partir du stockage d’objets blob sur un système de fichiers local.
  * Le Stockage Blob Azure offre une option simple et économique pour les projets de moindre envergure.  Tous les fichiers d’éléments multimédias sont requis sur chaque machine virtuelle du pool. Par conséquent, lorsque le nombre et la taille des fichiers d’éléments multimédias augmentent, il convient de s’assurer que les transferts de fichiers peuvent s’effectuer aussi efficacement que possible.  
* Stockage Azure en tant que système de fichiers en utilisant [blobfuse](../storage/blobs/storage-how-to-mount-container-linux.md) :
  * Pour les machines virtuelles Linux, un compte de stockage peut être exposé et utilisé en tant que système de fichiers lorsque le pilote de système de fichiers virtuel blobfuse est utilisé.
  * Cette option a pour avantage d’être particulièrement économique. En effet, aucune machine virtuelle n’est requise pour le système de fichiers. De plus, la mise en cache de blobfuse sur les machines virtuelles évite les téléchargements répétés des mêmes fichiers pour plusieurs travaux et tâches.  Le déplacement des données s’effectue également de façon simple. En effet, les fichiers sont simplement des blobs, et des API et outils standard comme azcopy peuvent être utilisés pour copier un fichier entre un système de fichiers local et le Stockage Azure.
* Système de fichiers ou partage de fichiers :
  * Plusieurs options sont disponibles selon le système d’exploitation des machines virtuelles et les exigences en matière de performances/mise à l’échelle : [Azure Files](../storage/files/storage-files-introduction.md), utilisation d’une machine virtuelle avec des disques attachés pour NFS, utilisation de plusieurs machines virtuelles avec des disques attachés pour un système de fichiers distribués comme GlusterFS ou utilisation d’une offre tierce.
  * [Avere Systems](https://www.averesystems.com/) fait désormais partie de Microsoft et proposera bientôt des solutions idéales pour le rendu hautes performances à grande échelle.  La solution Avere permettra de créer un cache SMB ou NFS Azure fonctionnant conjointement avec le stockage d’objets blob ou avec des périphériques de stockage NAS locaux.
  * Avec un système de fichiers, les fichiers peuvent être lus ou écrits directement sur le système de fichiers ou peuvent être copiés entre le système de fichiers et les machines virtuelles du pool.
  * Un système de fichiers partagé permet d’utiliser un grand nombre d’éléments multimédias partagés entre les projets et travaux avec des tâches de rendu accédant uniquement aux éléments nécessaires.

## <a name="using-azure-blob-storage"></a>Utilisation du Stockage Blob Azure

Il convient d’utiliser un compte de stockage d’objets blob ou un compte de stockage à usage général v2.  Ces deux types de compte de stockage peuvent être configurés avec des limites bien supérieures à celles d’un compte de stockage à usage général v1, comme expliqué en détail dans [ce billet de blog](https://azure.microsoft.com/blog/announcing-larger-higher-scale-storage-accounts/).  Une fois le compte configuré, ces limites supérieures améliorent considérablement les performances et l’extensibilité, en particulier lorsqu’un grand nombre de machines virtuelles du pool accèdent au compte de stockage.

### <a name="copying-files-between-client-and-blob-storage"></a>Copie de fichiers entre le client et le stockage d’objets blob

Plusieurs mécanismes permettent de copier des fichiers vers et à partir du Stockage Azure, notamment l’API de blob de stockage, la bibliothèque [Azure Storage Data Movement Library](https://github.com/Azure/azure-storage-net-data-movement), l’outil de ligne de commande azcopy pour [Windows](../storage/common/storage-use-azcopy-v10.md) ou [Linux](../storage/common/storage-use-azcopy-v10.md), [l’Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) et [Azure Batch Explorer](https://azure.github.io/BatchExplorer/).

Par exemple, azcopy permet de transférer tous les éléments multimédias d’un dossier comme suit :


`azcopy /source:. /dest:https://account.blob.core.windows.net/rendering/project /destsas:"?st=2018-03-30T16%3A26%3A00Z&se=2020-03-31T16%3A26%3A00Z&sp=rwdl&sv=2017-04-17&sr=c&sig=sig" /Y`

Pour copier uniquement les fichiers modifiés, vous pouvez utiliser le paramètre /XO :

`azcopy /source:. /dest:https://account.blob.core.windows.net/rendering/project /destsas:"?st=2018-03-30T16%3A26%3A00Z&se=2020-03-31T16%3A26%3A00Z&sp=rwdl&sv=2017-04-17&sr=c&sig=sig" /XO /Y`

### <a name="copying-input-asset-files-from-blob-storage-to-batch-pool-vms"></a>Copie de fichiers d’éléments multimédias d’entrée à partir d’un stockage d’objets blob vers les machines virtuelles d’un pool Batch

Les fichiers peuvent être copiés selon différentes approches. La meilleure approche dépend de la taille des éléments multimédias des travaux.
L’approche la plus simple consiste à copier tous les fichiers d’éléments multimédias sur les machines virtuelles du pool pour chaque travail :

* Lorsque des fichiers sont uniques à un travail mais sont requis pour toutes les tâches d’un travail, une [tâche de préparation du travail](/rest/api/batchservice/job/add#jobpreparationtask) peut être spécifiée pour copier tous les fichiers.  La tâche de préparation du travail est exécutée une fois lors de l’exécution de la première tâche de travail sur une machine virtuelle. Elle n’est pas exécutée de nouveau pour les tâches de travail suivantes.
* Une [tâche de fin du travail](/rest/api/batchservice/job/add#jobreleasetask) doit être spécifiée pour supprimer les fichiers de chaque travail une fois le travail terminé. Ceci permet d’éviter que les fichiers d’éléments multimédias des travaux ne remplissent le disque de la machine virtuelle.
* Lorsque plusieurs travaux utilisent les mêmes éléments multimédias et que seules des modifications incrémentielles sont apportées aux éléments multimédias de chaque travail, tous les fichiers d’éléments multimédias sont copiés, même si seul un sous-ensemble a été mis à jour.  Ce processus serait inefficace en présence de nombreux fichiers d’éléments multimédias volumineux.

Lorsque des fichiers d’éléments multimédias sont réutilisés entre des travaux et que seules des modifications incrémentielles sont apportées entre les travaux, une approche plus efficace mais légèrement plus complexe consiste à stocker les éléments multimédias dans le dossier partagé sur la machine virtuelle à synchroniser les fichiers modifiés.

* La tâche de préparation du travail effectue la copie à l’aide d’azcopy avec le paramètre /XO vers le dossier partagé de machine virtuelle spécifié par la variable d’environnement AZ_BATCH_NODE_SHARED_DIR.  Seuls les fichiers modifiés sont alors copiés sur chaque machine virtuelle.
* Il conviendra de réfléchir à la taille de l’ensemble des éléments multimédias pour s’assurer que le lecteur temporaire des machines virtuelles du pool pourra les stocker.

Azure Batch offre une prise en charge intégrée de la copie de fichiers entre un compte de stockage et les machines virtuelles d’un pool Batch.  Les [fichiers de ressources](/rest/api/batchservice/job/add#resourcefile) du travail copient les fichiers à partir du stockage vers les machines virtuelles du pool et peuvent être spécifiés pour la tâche de préparation du travail.  Malheureusement, en présence de plusieurs centaines de fichiers, une limite peut être atteinte et les travaux peuvent échouer.  En présence d’un grand nombre d’éléments multimédias, il est recommandé d’utiliser la ligne de commande azcopy dans la tâche de préparation du travail. Celle-ci accepte les caractères génériques et n’impose aucune limite.

### <a name="copying-output-files-to-blob-storage-from-batch-pool-vms"></a>Copie des fichiers de sortie vers le stockage d’objets blob à partir des machines virtuelles du pool Batch

Les [fichiers de sortie](/rest/api/batchservice/task/add#outputfile) peuvent être utilisés pour copier des fichiers à partir d’une machine virtuelle du pool vers le stockage.  Il est possible de copier un ou plusieurs fichiers à partir de la machine virtuelle vers un compte de stockage spécifié une fois la tâche terminée.  La sortie rendue doit être copiée, mais il peut également être judicieux de stocker les fichiers journaux.

## <a name="using-a-blobfuse-virtual-file-system-for-linux-vm-pools"></a>Utilisation d’un système de fichiers virtuel blobfuse pour les pools de machines virtuelles Linux

Blobfuse est un pilote de système de fichiers virtuel pour le Stockage Blob Azure, qui vous permet d’accéder aux fichiers stockés en tant que blobs dans un compte de stockage par le biais du système de fichiers Linux.

Les nœuds de pool peuvent monter le système de fichiers au démarrage. Le montage peut également s’effectuer dans le cadre d’une tâche de préparation du travail (une tâche exécutée uniquement lorsque la première tâche d’un travail s’exécute sur un nœud).  Blobfuse peut être configuré pour tirer parti d’un ramdisk et du disque SSD local des machines virtuelles pour la mise en cache des fichiers. Ceci améliore considérablement les performances lorsque plusieurs tâches sur un nœud accèdent aux mêmes fichiers.

Des [exemples de modèles sont disponibles](https://github.com/Azure/BatchExplorer-data/tree/master/ncj/vray/render-linux-with-blobfuse-mount), permettant d’exécuter des rendus V-Ray autonomes à l’aide d’un système de fichiers blobfuse. Ils peuvent être utilisés comme bases pour les modèles d’autres applications.

### <a name="accessing-files"></a>Accès aux fichiers

Les tâches de travail spécifient les chemins d’accès pour les fichiers d’entrée et les fichiers de sortie à l’aide du système de fichiers monté.

### <a name="copying-input-asset-files-from-blob-storage-to-batch-pool-vms"></a>Copie de fichiers d’éléments multimédias d’entrée à partir d’un stockage d’objets blob vers les machines virtuelles d’un pool Batch

Dans la mesure où les fichiers sont simplement des blobs dans le Stockage Azure, il est possible d’utiliser des API d’objet blob, outils et interfaces utilisateur standard pour copier des fichiers entre un système de fichiers local et un stockage d’objets blob, par exemple, azcopy, l’Explorateur Stockage, Batch Explorer, etc.

## <a name="using-azure-files-with-windows-vms"></a>Utilisation d’Azure Files avec des machines virtuelles Windows

[Azure Files](../storage/files/storage-files-introduction.md) offre des partages de fichiers entièrement gérés dans le cloud, accessibles à l’aide du protocole SMB.  Azure Files est basé sur le Stockage Blob Azure. Cette solution [économique](https://azure.microsoft.com/pricing/details/storage/files/) peut être configurée avec la réplication des données vers une autre région. Elle est donc redondante à l’échelle mondiale.  Vous devez revoir les [objectifs de mise à l’échelle](../storage/files/storage-files-scale-targets.md#azure-files-scale-targets) pour déterminer l’opportunité d’utiliser Azure Files selon la taille du pool et le nombre de fichiers d’éléments multimédias prévus.

Le montage d’un partage de fichiers Azure est expliqué dans cette [documentation](../storage/files/storage-how-to-use-files-windows.md).

### <a name="mounting-an-azure-files-share"></a>Montage d’un partage Azure Files

En vue d’une utilisation dans Batch, une opération de montage doit être effectuée à chaque exécution d’une tâche. En effet, il n’est pas possible de conserver la connexion entre les tâches.  Pour ce faire, la méthode la plus simple consiste à utiliser cmdkey pour conserver les informations d’identification à l’aide de la tâche de démarrage dans la configuration du pool, puis à monter le partage avant chaque tâche.

Exemple d’utilisation de cmdkey dans un modèle de pool (séquence d’échappement pour une utilisation dans le fichier JSON) : notez que lors de la séparation de l’appel cmdkey de l’appel net use, le contexte utilisateur pour la tâche de démarrage doit être le même que celui utilisé pour l’exécution des tâches :

```
"startTask": {
  "commandLine": "cmdkey /add:storageaccountname.file.core.windows.net
    /user:AZURE\\markscuscusbatch /pass:storage_account_key",
  "userIdentity":{
    "autoUser": {
      "elevationLevel": "nonadmin",
      "scope": "pool"
    }
}
```

Exemple de ligne de commande pour une tâche de travail :
```
"commandLine":"net use S:
  \\\\storageaccountname.file.core.windows.net\\rendering &
3dsmaxcmdio.exe -v:5 -rfw:0 -10 -end:10
  -bitmapPath:\"s:\\3dsMax\\Dragon\\Assets\"
  -outputName:\"s:\\3dsMax\\Dragon\\RenderOutput\\dragon.jpg\"
  -w:1280 -h:720
  \"s:\\3dsMax\\Dragon\\Assets\\Dragon_Character_Rig.max\""
```

### <a name="accessing-files"></a>Accès aux fichiers

Les tâches de travail spécifient les chemins d’accès pour les fichiers d’entrée et les fichiers de sortie à l’aide du système de fichiers monté avec un lecteur mappé ou un chemin d’accès UNC.

### <a name="copying-input-asset-files-from-blob-storage-to-batch-pool-vms"></a>Copie de fichiers d’éléments multimédias d’entrée à partir d’un stockage d’objets blob vers les machines virtuelles d’un pool Batch

Azure Files est pris en charge par tous les outils et API courants prenant en charge le Stockage Azure, notamment azcopy, Azure CLI, l’Explorateur Stockage, Azure PowerShell, Batch Explorer, etc.

[Azure File Sync](../storage/files/storage-sync-files-planning.md) permet de synchroniser automatiquement des fichiers entre un système de fichiers local et un partage de fichiers Azure.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les options de stockage, consultez la documentation détaillée :

* [Stockage Blob Azure](../storage/blobs/storage-blobs-introduction.md)
* [Blobfuse](../storage/blobs/storage-how-to-mount-container-linux.md)
* [Azure Files](../storage/files/storage-files-introduction.md)
