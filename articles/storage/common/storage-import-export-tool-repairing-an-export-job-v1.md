---
title: Réparation d’un travail d’exportation Azure Import/Export - v1 | Microsoft Docs
description: Apprenez à réparer un travail d’exportation qui a été créé et exécuté à l’aide du service Azure Import/Export.
author: alkohli
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 01/23/2017
ms.author: alkohli
ms.subservice: common
ms.openlocfilehash: 35738db5b7bd4a1ac7aaf94e2dc5f1d26a075cdf
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92791832"
---
# <a name="repairing-an-export-job"></a>Réparation d’un travail d’exportation
Une fois qu’un travail d’exportation est terminé, vous pouvez exécuter l’outil Microsoft Azure Import/Export local pour :  
  
1.  Télécharger tous les fichiers que le service Azure Import/Export n’a pas pu exporter.  
  
2.  Vérifier que les fichiers sur le lecteur ont été correctement exportés.  
  
Vous devez disposer d’une connectivité au Stockage Azure pour utiliser cette fonctionnalité.  
  
La commande de réparation d’un travail d’exportation est **RepairExport** .

## <a name="repairexport-parameters"></a>Paramètres de RepairExport

Les paramètres suivants peuvent être spécifiés pour **RepairExport** :  
  
|Paramètre|Description|  
|---------------|-----------------|  
|**/r:&lt;RepairFile\>**|Obligatoire. Chemin d’accès au fichier de réparation, qui suit la progression de la réparation et vous permet de reprendre une réparation interrompue. Chaque disque ne doit avoir qu’un fichier de réparation. Quand vous lancez la réparation d’un lecteur donné, vous transmettez le chemin à un fichier de réparation qui n’existe pas encore. Pour reprendre une réparation interrompue, vous devez transmettre le nom d’un fichier de réparation existant. Spécifiez toujours le fichier de réparation correspondant au lecteur cible.|  
|**/logdir:&lt;LogDirectory\>**|facultatif. Répertoire du journal. Les fichiers journaux détaillés seront écrits dans ce répertoire. Si aucun répertoire de journaux n’est spécifié, le répertoire en cours est utilisé comme répertoire de journaux.|  
|**/d:&lt;TargetDirectory\>**|Obligatoire. Répertoire pour valider et réparer. Il s’agit généralement du répertoire racine du lecteur d’exportation, mais ce peut également être un partage de fichiers réseau contenant une copie des fichiers exportés.|  
|**/bk:&lt;BitLockerKey\>**|facultatif. Spécifiez la clé BitLocker si vous souhaitez que l’outil déverrouille un lecteur chiffré dans lequel les fichiers d’origine sont stockés.|  
|**/sn:&lt;StorageAccountName\>**|Obligatoire. Nom du compte de stockage du travail d’exportation.|  
|**/sk:&lt;StorageAccountKey\>**|**Obligatoire** si et seulement si une SAP de conteneur n’est pas spécifiée. Clé du compte de stockage du travail d’exportation.|  
|**/csas:&lt;ContainerSas\>**|**Obligatoire** si et seulement si la clé du compte de stockage n’est pas spécifiée. SAP de conteneur pour accéder aux objets blob associés au travail d’exportation.|  
|**/CopyLogFile:&lt;DriveCopyLogFile\>**|Obligatoire. Chemin du fichier journal de copie du disque. Le fichier est généré par le service Windows Azure Import/Export et peut être téléchargé à partir du stockage blob associé au travail. Le fichier journal de copie contient des informations sur les blob ou fichiers défectueux à réparer.|  
|**/ManifestFile:&lt;DriveManifestFile\>**|facultatif. Chemin du fichier de manifeste du lecteur d’exportation. Ce fichier est généré par le service Windows Azure Import/Export et stocké sur le lecteur d’exportation. Éventuellement, dans un blob dans le compte de stockage associé au travail.<br /><br /> Le contenu des fichiers sur le lecteur d’exportation sera vérifié avec les hachages MD5 contenus dans ce fichier. Les fichiers endommagés sont téléchargés et réécrits dans les répertoires cibles.|  
  
## <a name="using-repairexport-mode-to-correct-failed-exports"></a>Utilisation du mode RepairExport pour corriger les échecs d’exportation  
Vous pouvez utiliser l’outil Azure Import/Export pour télécharger les fichiers dont l’exportation a échoué. Le fichier journal de copie contient une liste des fichiers qui n’ont pas pu être exportés.  
  
Les causes d’échec d’exportation incluent les possibilités suivantes :  
  
-   Lecteurs endommagés  
  
-   Clé de compte de stockage modifiée pendant le processus de transfert  
  
Pour exécuter l’outil en mode **RepairExport** , vous devez d’abord connecter le lecteur contenant les fichiers exportés à votre ordinateur. Ensuite, exécutez l’outil Azure Import/Export en spécifiant le chemin d’accès à ce lecteur avec le paramètre `/d`. Vous devez également spécifier le chemin d’accès au fichier journal de copie du lecteur que vous avez téléchargé. L’exemple de ligne de commande ci-dessous exécute l’outil pour réparer tous les fichiers dont l’exportation a échoué :  
  
```  
WAImportExport.exe RepairExport /r:C:\WAImportExport\9WM35C3U.rep /d:G:\ /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\WAImportExport\9WM35C3U.log  
```  
  
L’exemple suivant est un fichier journal de copie qui montre que l’exportation d’un bloc dans le blob a échoué :  
  
```xml
<?xml version="1.0" encoding="utf-8"?>  
<DriveLog>  
  <DriveId>9WM35C2V</DriveId>  
  <Blob Status="CompletedWithErrors">  
    <BlobPath>pictures/wild/desert.jpg</BlobPath>  
    <FilePath>\pictures\wild\desert.jpg</FilePath>  
    <LastModified>2012-09-18T23:47:08Z</LastModified>  
    <Length>163840</Length>  
    <BlockList>  
      <Block Offset="65536" Length="65536" Id="AQAAAA==" Status="Failed" />  
    </BlockList>  
  </Blob>  
  <Status>CompletedWithErrors</Status>  
</DriveLog>  
```  
  
Le fichier journal de copie indique qu’une défaillance s’est produite pendant que le service Windows Azure Import/Export téléchargeait un des blocs de l’objet blob dans le fichier sur le lecteur d’exportation. Les autres composants du fichier ont été téléchargés avec succès et la longueur du fichier a été correctement définie. Dans ce cas, l’outil ouvre le fichier sur le lecteur, télécharge le bloc à partir du compte de stockage et l’écrit dans la plage des fichiers à partir du décalage (offset) 65536 avec la longueur (length) 65536.  
  
## <a name="using-repairexport-to-validate-drive-contents"></a>Utilisation de RepairExport pour valider le contenu du lecteur  
Vous pouvez également utiliser Azure Import/Export avec l’option **RepairExport** pour vérifier que le contenu sur le lecteur est correct. Le fichier manifeste sur chaque lecteur d’exportation contient des MD5 pour le contenu du lecteur.  
  
Le service Azure Import/Export peut également enregistrer les fichiers manifeste sur un compte de stockage pendant le processus d’exportation. L’emplacement des fichiers manifeste est disponible via l’opération [Get Job](/rest/api/storageimportexport/jobs) une fois le travail terminé. Pour plus d’informations sur le format d’un fichier manifeste de lecteur, consultez [Format de fichier manifeste du service d’importation/exportation](/previous-versions/azure/storage/common/storage-import-export-file-format-metadata-and-properties).  
  
L’exemple suivant montre comment exécuter l’outil Azure Import/Export avec les paramètres **/ManifestFile** et **/CopyLogFile** :  
  
```  
WAImportExport.exe RepairExport /r:C:\WAImportExport\9WM35C3U.rep /d:G:\ /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\WAImportExport\9WM35C3U.log /ManifestFile:G:\9WM35C3U.manifest  
```  
  
L’exemple suivant montre un fichier manifeste :  
  
```xml
<?xml version="1.0" encoding="utf-8"?>  
<DriveManifest Version="2011-10-01">  
  <Drive>  
    <DriveId>9WM35C3U</DriveId>  
    <ClientCreator>Windows Azure Import/Export service</ClientCreator>  
    <BlobList>
      <Blob>  
        <BlobPath>pictures/city/redmond.jpg</BlobPath>  
        <FilePath>\pictures\city\redmond.jpg</FilePath>  
        <Length>15360</Length>  
        <PageRangeList>  
          <PageRange Offset="0" Length="3584" Hash="72FC55ED9AFDD40A0C8D5C4193208416" />  
          <PageRange Offset="3584" Length="3584" Hash="68B28A561B73D1DA769D4C24AA427DB8" />  
          <PageRange Offset="7168" Length="512" Hash="F521DF2F50C46BC5F9EA9FB787A23EED" />  
        </PageRangeList>  
        <PropertiesPath Hash="E72A22EA959566066AD89E3B49020C0A">\pictures\city\redmond.jpg.properties</PropertiesPath>  
      </Blob>  
      <Blob>  
        <BlobPath>pictures/wild/canyon.jpg</BlobPath>  
        <FilePath>\pictures\wild\canyon.jpg</FilePath>  
        <Length>10884</Length>  
        <BlockList>  
          <Block Offset="0" Length="2721" Id="AAAAAA==" Hash="263DC9C4B99C2177769C5EBE04787037" />  
          <Block Offset="2721" Length="2721" Id="AQAAAA==" Hash="0C52BAE2CC20EFEC15CC1E3045517AA6" />  
          <Block Offset="5442" Length="2721" Id="AgAAAA==" Hash="73D1CB62CB426230C34C9F57B7148F10" />  
          <Block Offset="8163" Length="2721" Id="AwAAAA==" Hash="11210E665C5F8E7E4F136D053B243E6A" />  
        </BlockList>  
        <PropertiesPath Hash="81D7F81B2C29F10D6E123D386C3A4D5A">\pictures\wild\canyon.jpg.properties</PropertiesPath>  
      </Blob> 
    </BlobList>  
 </Drive>  
</DriveManifest>  
``` 
  
Après avoir terminé le processus de réparation, l’outil parcourt chaque fichier référencé dans le fichier manifeste et vérifie l’intégrité du fichier avec les hachages MD5. Pour le manifeste ci-dessus, il parcourt les composants suivants.  

```  
G:\pictures\city\redmond.jpg, offset 0, length 3584  
  
G:\pictures\city\redmond.jpg, offset 3584, length 3584  
  
G:\pictures\city\redmond.jpg, offset 7168, length 3584  
  
G:\pictures\city\redmond.jpg.properties  
  
G:\pictures\wild\canyon.jpg, offset 0, length 2721  
  
G:\pictures\wild\canyon.jpg, offset 2721, length 2721  
  
G:\pictures\wild\canyon.jpg, offset 5442, length 2721  
  
G:\pictures\wild\canyon.jpg, offset 8163, length 2721  
  
G:\pictures\wild\canyon.jpg.properties  
```

Tout composant dont la vérification échoue sera téléchargé par l’outil et réécrit dans le même fichier sur le lecteur.  
  
## <a name="next-steps"></a>Étapes suivantes
 
* [Configuration de l’outil Azure Import/Export](storage-import-export-tool-setup-v1.md)   
* [Préparation des disques durs pour un travail d’importation](/previous-versions/azure/storage/common/storage-import-export-tool-preparing-hard-drives-import-v1)   
* [Consultation de l’état du travail avec les fichiers journaux de copie](storage-import-export-tool-reviewing-job-status-v1.md)   
* [Réparation d’un travail d’importation](storage-import-export-tool-repairing-an-import-job-v1.md)