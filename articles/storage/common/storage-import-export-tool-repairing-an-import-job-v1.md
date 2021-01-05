---
title: Réparation d’un travail d’importation Azure Import/Export - v1 | Microsoft Docs
description: Apprenez à réparer un travail d’importation qui a été créé et exécuté à l’aide du service Azure Import/Export.
author: alkohli
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 01/23/2017
ms.author: alkohli
ms.subservice: common
ms.openlocfilehash: 0454e7bcc81c71cdffcddcd859bb6d335cc8aef2
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92791815"
---
# <a name="repairing-an-import-job"></a>Réparation d’un travail d’importation
Le service Microsoft Azure Import/Export peut ne pas copier certains de vos fichiers ou des parties d’un fichier dans le service BLOB Windows Azure. Voici quelques exemples de raisons de ces échecs :  
  
-   Fichiers endommagés  
  
-   Lecteurs endommagés  
  
-   La clé du compte de stockage a été modifiée pendant le transfert du fichier.  
  
Vous pouvez exécuter l’outil Microsoft Azure Import/Export avec les fichiers journaux de copie du travail d’importation. L’outil charge les fichiers manquants ou des parties d’un fichier dans votre compte de stockage Windows Azure pour terminer le travail d’importation.  
  
## <a name="repairimport-parameters"></a>Paramètres de RepairImport

Les paramètres suivants peuvent être spécifiés pour **RepairImport** : 
  
|||  
|-|-|  
|**/r:** &lt;RepairFile\>|**Obligatoire.** Chemin d’accès au fichier de réparation, qui suit la progression de la réparation et vous permet de reprendre une réparation interrompue. Chaque disque ne doit avoir qu’un fichier de réparation. Quand vous lancez la réparation d’un lecteur donné, transmettez le chemin d’un fichier de réparation, qui n’existe pas encore. Pour reprendre une réparation interrompue, vous devez transmettre le nom d’un fichier de réparation existant. Spécifiez toujours le fichier de réparation correspondant au lecteur cible.|  
|**/logdir:** &lt;LogDirectory\>|**Facultatif.** Répertoire du journal. Les fichiers journaux détaillés sont écrits dans ce répertoire. Si aucun répertoire de journaux n’est spécifié, le répertoire actif est utilisé comme répertoire de journaux.|  
|**/d:** &lt;TargetDirectories\>|**Obligatoire.** Un ou plusieurs répertoires séparés par des points-virgules contenant les fichiers d’origine qui ont été importés. Le disque d’importation peut également être utilisé, mais il n’est pas obligatoire si d’autres emplacements de fichiers d’origine sont disponibles.|  
|**/bk:** &lt;BitLockerKey\>|**Facultatif.** Spécifiez la clé BitLocker si vous souhaitez que l’outil déverrouille un disque chiffré dans lequel les fichiers d’origine sont disponibles.|  
|**/sn:** &lt;StorageAccountName\>|**Obligatoire.** Nom du compte de stockage du travail d’importation.|  
|**/sk:** &lt;StorageAccountKey\>|**Obligatoire** si et seulement si une SAP de conteneur n’est pas spécifiée. Clé du compte de stockage du travail d’importation.|  
|**/csas:** &lt;ContainerSas\>|**Required** if and only if the storage account key isn't specified. SAP de conteneur pour accéder aux objets blob associés au travail d’importation.|  
|**/CopyLogFile:** &lt;DriveCopyLogFile\>|**Obligatoire.** Chemin d’accès au fichier journal de copie du disque (journal détaillé ou d’erreurs). Le fichier est généré par le service Windows Azure Import/Export et peut être téléchargé à partir du stockage blob associé au travail. Le fichier journal de copie contient des informations sur les objets blob ou les fichiers en échec qui doivent être réparés.|  
|**/PathMapFile:** &lt;DrivePathMapFile\>|**Facultatif.** Chemin d’accès à un fichier texte utilisé pour résoudre les ambiguïtés si vous importez dans le même travail plusieurs fichiers portant le même nom. La première fois que l’outil est exécuté, il peut remplir ce fichier avec tous les noms ambigus. Les exécutions suivantes de l’outil utilisent ce fichier pour résoudre les ambiguïtés.|  
  
## <a name="using-the-repairimport-command"></a>Utilisation de la commande RepairImport  
Pour réparer des données d’importation en diffusant en continu les données sur le réseau, vous devez spécifier les répertoires qui contiennent les fichiers d’origine que vous importez à l’aide du paramètre `/d`. Spécifiez également le fichier journal de copie que vous avez téléchargé à partir de votre compte de stockage. Une ligne de commande classique pour réparer un travail d’importation avec des échecs partiels est semblable à :  
  
```  
WAImportExport.exe RepairImport /r:C:\WAImportExport\9WM35C2V.rep /d:C:\Users\bob\Pictures;X:\BobBackup\photos /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\WAImportExport\9WM35C2V.log  
```  
  
Dans l’exemple suivant d’un fichier journal de copie, une partie de 64 Ko d’un fichier a été endommagée sur le disque qui a été envoyé pour le travail d’importation. Dans la mesure où il s’agit de la seule erreur indiquée, les autres objets blob du travail ont bien été importés.  
  
```xml
<?xml version="1.0" encoding="utf-8"?>  
<DriveLog>  
 <DriveId>9WM35C2V</DriveId>  
 <Blob Status="CompletedWithErrors">  
 <BlobPath>pictures/animals/koala.jpg</BlobPath>  
 <FilePath>\animals\koala.jpg</FilePath>  
 <Length>163840</Length>  
 <ImportDisposition Status="Overwritten">overwrite</ImportDisposition>  
 <PageRangeList>  
  <PageRange Offset="65536" Length="65536" Hash="AA2585F6F6FD01C4AD4256E018240CD4" Status="Corrupted" />  
 </PageRangeList>  
 </Blob>  
 <Status>CompletedWithErrors</Status>  
</DriveLog>  
```
  
Quand ce journal de copie est transmis à l’outil Azure Import/Export, celui-ci essaie de terminer l’importation de ce fichier en copiant le contenu manquant via le réseau. Dans l’exemple ci-dessus, l’outil recherche le fichier d’origine `\animals\koala.jpg` dans les deux répertoires (`C:\Users\bob\Pictures` et `X:\BobBackup\photos`). Si le fichier `C:\Users\bob\Pictures\animals\koala.jpg` existe, l’outil Azure Import/Export copie la plage de données manquante dans l’objet blob correspondant `http://bobmediaaccount.blob.core.windows.net/pictures/animals/koala.jpg`.  
  
## <a name="resolving-conflicts-when-using-repairimport"></a>Résolution des conflits lors de l’utilisation de RepairImport  
Dans certains cas, l’outil peut ne pas être en mesure de trouver ou d’ouvrir le fichier nécessaire pour l’une des raisons suivantes : le fichier est introuvable, le fichier n’est pas accessible, le nom de fichier est ambigu ou le contenu du fichier n’est plus approprié.  
  
Une erreur ambiguë peut se produire si l’outil essaie de localiser `\animals\koala.jpg` et s’il existe un fichier portant ce nom dans `C:\Users\bob\pictures` et `X:\BobBackup\photos`. Autrement dit, `C:\Users\bob\pictures\animals\koala.jpg` et `X:\BobBackup\photos\animals\koala.jpg` existent sur les disques de travail d’importation.  
  
L’option `/PathMapFile` vous permet de résoudre ces erreurs. Vous pouvez spécifier le nom du fichier qui contient la liste des fichiers que l’outil n’a pas pu identifier correctement. L’exemple de ligne de commande suivant remplit `9WM35C2V_pathmap.txt` :  
  
```
WAImportExport.exe RepairImport /r:C:\WAImportExport\9WM35C2V.rep /d:C:\Users\bob\Pictures;X:\BobBackup\photos /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\WAImportExport\9WM35C2V.log /PathMapFile:C:\WAImportExport\9WM35C2V_pathmap.txt  
```
  
L’outil écrit ensuite les chemins d’accès de fichier problématique dans `9WM35C2V_pathmap.txt`, à raison d’un par ligne. Par exemple, le fichier peut contenir les entrées suivantes après avoir exécuté la commande :  
 
```
\animals\koala.jpg  
\animals\kangaroo.jpg  
```
  
 Pour chaque fichier de la liste, vous devez essayer de localiser et d’ouvrir le fichier pour vous assurer qu’il est disponible pour l’outil. Si vous souhaitez indiquer explicitement à l’outil où rechercher un fichier, modifiez le fichier de mappage de chemin d’accès et ajoutez le chemin d’accès à chaque fichier sur la même ligne, séparé par un caractère de tabulation :  
  
```
\animals\koala.jpg           C:\Users\bob\Pictures\animals\koala.jpg  
\animals\kangaroo.jpg        X:\BobBackup\photos\animals\kangaroo.jpg  
```
  
Après avoir mis les fichiers nécessaires à disposition de l’outil ou après avoir mis à jour le fichier de mappage de chemin d’accès, vous pouvez réexécuter l’outil pour terminer le processus d’importation.  
  
## <a name="next-steps"></a>Étapes suivantes
 
* [Configuration de l’outil Azure Import/Export](storage-import-export-tool-setup-v1.md)   
* [Préparation des disques durs pour un travail d’importation](/previous-versions/azure/storage/common/storage-import-export-tool-preparing-hard-drives-import-v1)   
* [Consultation de l’état du travail avec les fichiers journaux de copie](storage-import-export-tool-reviewing-job-status-v1.md)   
* [Réparation d’un travail d’exportation](./storage-import-export-tool-repairing-an-export-job-v1.md)