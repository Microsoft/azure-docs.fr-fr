---
title: Résoudre les problèmes de chargements de données à l’aide des journaux
titleSuffix: Azure Data Box Disk
description: Décrit comment utiliser les journaux et résoudre des problèmes rencontrés lors du chargement des données dans Azure Data Box Disk.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: troubleshooting
ms.date: 06/17/2019
ms.author: alkohli
ms.openlocfilehash: 4b53cf607bdf60c785c7324d9ede526a0983b7e6
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97605273"
---
# <a name="understand-logs-to-troubleshoot-data-upload-issues-in-azure-data-box-disk"></a>Comprendre les journaux pour résoudre les problèmes liés au chargement des données dans Azure Data Box Disk

Cet article s’applique à Microsoft Azure Data Box Disk et décrit les problèmes que vous rencontrez lors du chargement des données dans Azure.

## <a name="about-upload-logs"></a>À propos des journaux de chargement

Lorsque les données sont chargées sur un centre de données dans Azure, les fichiers `_error.xml` et `_verbose.xml` sont générés pour chaque compte de stockage. Ces journaux sont chargés sur le même compte de stockage utilisé pour charger des données. 

Les journaux sont au même format et contiennent des descriptions XML des événements qui se sont produits lors de la copie des données du disque vers le compte de stockage Azure.

Le journal détaillé contient des informations complètes sur l’état de l’opération de copie pour chaque objet blob ou fichier, alors que le journal d’erreurs contient uniquement les informations des objets blob ou fichiers qui ont rencontré des erreurs lors du chargement.

Le journal d’erreurs a la même structure que le journal détaillé mais il exclut les opérations réussies.

## <a name="download-logs"></a>Télécharger les journaux d’activité

Procédez comme suit pour localiser les journaux de chargement.

1. Si des erreurs se produisent lors du chargement des données dans Azure, le portail affiche un chemin d’accès au dossier dans lequel se trouvent les journaux de diagnostic.

    ![Lien vers les journaux dans le portail](./media/data-box-disk-troubleshoot-upload/data-box-disk-portal-logs.png)

2. Accédez à **waies**.

    ![journaux d'erreurs et journaux détaillés](./media/data-box-disk-troubleshoot-upload/data-box-disk-portal-logs-1.png)

Dans chaque cas, vous voyez les journaux d’activité d’erreurs et les journaux d’activité détaillés. Sélectionnez chaque journal et téléchargez une copie locale.

## <a name="sample-upload-logs"></a>Exemples de journaux de chargement

Voici un exemple de `_verbose.xml` obtenue. Dans ce cas, l'ordre s'est terminé sans erreur.

```xml

<?xml version="1.0" encoding="utf-8"?>
<DriveLog Version="2018-10-01">
  <DriveId>184020D95632</DriveId>
  <Blob Status="Completed">
    <BlobPath>$root/botetapageblob.vhd</BlobPath>
    <FilePath>\PageBlob\botetapageblob.vhd</FilePath>
    <Length>1073742336</Length>
    <ImportDisposition Status="Created">rename</ImportDisposition>
    <PageRangeList>
      <PageRange Offset="0" Length="4194304" Status="Completed" />
      <PageRange Offset="4194304" Length="4194304" Status="Completed" />
      <PageRange Offset="8388608" Length="4194304" Status="Completed" />
      --------CUT-------------------------------------------------------
      <PageRange Offset="1061158912" Length="4194304" Status="Completed" />
      <PageRange Offset="1065353216" Length="4194304" Status="Completed" />
      <PageRange Offset="1069547520" Length="4194304" Status="Completed" />
      <PageRange Offset="1073741824" Length="512" Status="Completed" />
    </PageRangeList>
  </Blob>
  <Blob Status="Completed">
    <BlobPath>$root/botetablockblob.txt</BlobPath>
    <FilePath>\BlockBlob\botetablockblob.txt</FilePath>
    <Length>19</Length>
    <ImportDisposition Status="Created">rename</ImportDisposition>
    <BlockList>
      <Block Offset="0" Length="19" Status="Completed" />
    </BlockList>
  </Blob>
  <File Status="Completed">
    <FileStoragePath>botetaazurefilesfolder/botetaazurefiles.txt</FileStoragePath>
    <FilePath>\AzureFile\botetaazurefilesfolder\botetaazurefiles.txt</FilePath>
    <Length>20</Length>
    <ImportDisposition Status="Created">rename</ImportDisposition>
    <FileRangeList>
      <FileRange Offset="0" Length="20" Status="Completed" />
    </FileRangeList>
  </File>
  <Status>Completed</Status>
</DriveLog>
```

Pour le même ordre, voici un exemple de `_error.xml` obtenu.

```xml

<?xml version="1.0" encoding="utf-8"?>
<DriveLog Version="2018-10-01">
  <DriveId>184020D95632</DriveId>
  <Summary>
    <ValidationErrors>
      <None Count="3" />
    </ValidationErrors>
    <CopyErrors>
      <None Count="3" Description="No errors encountered" />
    </CopyErrors>
  </Summary>
  <Status>Completed</Status>
</DriveLog>
```

Voici un exemple de `_error.xml` obtenu où l'ordre s'est terminé avec des erreurs. 

Dans ce cas, le fichier d’erreur contient une section `Summary` et une autre section contenant toutes les erreurs au niveau du fichier. 

Le `Summary` contient les `ValidationErrors` et les `CopyErrors`. Dans ce cas, 8 fichiers ou dossiers ont été chargés dans Azure et aucune erreur de validation n'a été enregistrée. Lorsque les données ont été copiées dans le compte de stockage Azure, 5 fichiers ou dossiers ont été correctement chargés. Les 3 fichiers ou dossiers restants ont été renommés conformément aux conventions d’affectation de noms des conteneurs Azure, puis correctement chargés dans Azure.

L'état au niveau des fichiers indique `BlobStatus`, ce qui décrit les mesures prises pour charger les objets blob. Dans ce cas, trois conteneurs ont été renommés, car les dossiers dans lesquels les données ont été copiées n'étaient pas conformes aux conventions d’affectation de noms des conteneurs Azure. Pour les objets blob téléchargés dans ces conteneurs, le nouveau nom de conteneur, le chemin d’accès de l’objet blob dans Azure, le chemin d’accès au fichier d'origine non valide et la taille de l’objet blob sont inclus.
  
```xml
 <?xml version="1.0" encoding="utf-8"?>
  <DriveLog Version="2018-10-01">
    <DriveId>18041C582D7E</DriveId>
    <Summary>
     <!--Summary for validation and data copy to Azure -->
      <ValidationErrors>
        <None Count="8" />
      </ValidationErrors>
      <CopyErrors>
        <Completed Count="5" Description="No errors encountered" />
        <ContainerRenamed Count="3" Description="Renamed the container as the original container name does not follow Azure conventions." />
      </CopyErrors>
    </Summary>
    <!--List of renamed containers with the new names, new file path in Azure, original invalid file path, and size -->
    <Blob Status="ContainerRenamed">
      <BlobPath>databox-c2073fd1cc379d83e03d6b7acce23a6cf29d1eef/private.vhd</BlobPath>
      <OriginalFilePath>\PageBlob\pageblob test\private.vhd</OriginalFilePath>
      <SizeInBytes>10490880</SizeInBytes>
    </Blob>
    <Blob Status="ContainerRenamed">
      <BlobPath>databox-c2073fd1cc379d83e03d6b7acce23a6cf29d1eef/resource.vhd</BlobPath>
      <OriginalFilePath>\PageBlob\pageblob test\resource.vhd</OriginalFilePath>
      <SizeInBytes>71528448</SizeInBytes>
    </Blob>
    <Blob Status="ContainerRenamed">
      <BlobPath>databox-c2073fd1cc379d83e03d6b7acce23a6cf29d1eef/role.vhd</BlobPath>
      <OriginalFilePath>\PageBlob\pageblob test\role.vhd</OriginalFilePath>
      <SizeInBytes>10490880</SizeInBytes>
    </Blob>
    <Status>CompletedWithErrors</Status>
  </DriveLog>
```

## <a name="data-upload-errors"></a>Erreurs de chargement des données

Les erreurs générées lors du chargement des données dans Azure sont résumées dans le tableau suivant.

| Code d'erreur | Description                   |
|-------------|------------------------------|
|`None` |  Opération réussie.           |
|`Renamed` | Objet blob renommé avec succès.   |
|`CompletedWithErrors` | Chargement terminé avec des erreurs. Les détails des fichiers de l’erreur sont inclus dans le fichier journal.  |
|`Corrupted`|Le CRC calculé au cours de l’ingestion de données ne correspond pas au CRC calculé lors du chargement.  |  
|`StorageRequestFailed` | Échec de la demande de stockage Azure.   |     
|`LeasePresent` | Cet élément est loué et est utilisé par un autre utilisateur. |
|`StorageRequestForbidden` |Échec du chargement en raison de problèmes d’authentification. |
|`ManagedDiskCreationTerminalFailure` | Échec du chargement en tant que disques managés. Les fichiers sont disponibles dans le compte de stockage intermédiaire en tant qu’objets blob de pages. Vous pouvez convertir manuellement des objets blob de pages en disques managés.  |
|`DiskConversionNotStartedTierInfoMissing` | Dans la mesure où le fichier de disque dur virtuel a été copié hors des dossiers de niveau précréés, un disque managé n’a pas été créé. Le fichier est téléchargé en tant qu’objet blob de pages pour le compte de stockage intermédiaire tel que spécifié lors de la création de l’ordre. Vous pouvez le convertir manuellement en un disque managé.|
|`InvalidWorkitem` | Impossible de charger les données car elles ne sont pas conformes aux normes de nomination et de limites Azure.|
|`InvalidPageBlobUploadAsBlockBlob` | Téléchargés comme objets blob de blocs dans un conteneur avec le préfixe `databoxdisk-invalid-pb-`.|
|`InvalidAzureFileUploadAsBlockBlob` | Téléchargés comme objets blob de blocs dans un conteneur avec le préfixe `databoxdisk-invalid-af`-.|
|`InvalidManagedDiskUploadAsBlockBlob` | Téléchargés comme objets blob de blocs dans un conteneur avec le préfixe `databoxdisk-invalid-md`-.|
|`InvalidManagedDiskUploadAsPageBlob` |Téléchargés comme objets blob de pages dans un conteneur avec le préfixe `databoxdisk-invalid-md-`. |
|`MovedToOverflowShare` |Téléchargement des fichiers dans un nouveau partage car le partage d’origine a dépassé la limite de taille maximale d’Azure. Le nouveau nom du partage de fichier comprend le nom original avec le suffixe `-2`.   |
|`MovedToDefaultAzureShare` |Téléchargement des fichiers qui ne faisaient partie d’aucun dossier dans un partage par défaut. Le nom du partage commence par `databox-`. |
|`ContainerRenamed` |Le conteneur de ces fichiers n’était pas conforme aux conventions d’affectation de noms d’Azure et a été renommé. Le nouveau nom commence par `databox-` et est suivi du suffixe SHA1 du nom d’origine |
|`ShareRenamed` |Le partage de ces fichiers n’était pas conforme aux conventions d’affectation de noms d’Azure et a été renommé. Le nouveau nom commence par `databox-` et est suivi du suffixe SHA1 du nom d’origine. |
|`BlobRenamed` |Ces fichiers n’étaient pas conformes aux conventions d’affectation de noms d’Azure et ont été renommés. Vérifiez le nouveau nom dans le champ `BlobPath`. |
|`FileRenamed` |Ces fichiers n’étaient pas conformes aux conventions d’affectation de noms d’Azure et ont été renommés. Vérifiez le nouveau nom dans le champ `FileStoragePath`. |
|`DiskRenamed` |Ces fichiers n’étaient pas conformes aux conventions d’affectation de noms d’Azure et ont été renommés. Vérifiez le nouveau nom dans le champ `BlobPath`. |


## <a name="next-steps"></a>Étapes suivantes

- [Ouvrir un ticket de support concernant les problèmes Data Box Disk](data-box-disk-contact-microsoft-support.md).
