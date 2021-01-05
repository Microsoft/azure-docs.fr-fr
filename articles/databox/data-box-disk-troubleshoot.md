---
title: Résolution des problèmes liés à Azure Data Box Disk | Microsoft Docs
description: Découvrez comment utiliser les journaux pour résoudre les problèmes de validation susceptibles de survenir lorsque vous déployez Azure Data Box Disk.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: troubleshooting
ms.date: 06/14/2019
ms.author: alkohli
ms.openlocfilehash: fa8a6643f1b7bd60fbf6e5950234e0381666177e
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97605222"
---
# <a name="use-logs-to-troubleshoot-validation-issues-in-azure-data-box-disk"></a>Utiliser des journaux pour résoudre les problèmes de validation dans Azure Data Box Disk

Cet article s’applique à Microsoft Azure Data Box Disk. L’article décrit comment utiliser les journaux pour résoudre les problèmes de validation que vous pouvez rencontrer lorsque vous déployez cette solution.

## <a name="validation-tool-log-files"></a>Fichiers journaux de l’outil de validation

Lorsque vous validez les données sur les disques à l’aide de l’[outil de validation](data-box-disk-deploy-copy-data.md#validate-data), un fichier *error.xml* est généré afin d’enregistrer les erreurs. Le fichier journal se trouve dans le dossier `Drive:\DataBoxDiskImport\logs` de votre lecteur. Un lien vers le journal des erreurs est fourni lorsque vous exécutez la validation.

<!--![Validation tool with link to error log](media/data-box-disk-troubleshoot/validation-tool-link-error-log.png)-->

Si vous exécutez plusieurs sessions de validation, un seul journal des erreurs est généré par session.

- Voici un exemple de journal des erreurs généré lorsque les données chargées dans le dossier `PageBlob` ne sont pas des données de 512 octets alignés. Toutes les données chargées sur PageBlob doivent être de 512 octets alignés (par exemple, un VHD ou VHDX). Dans ce fichier, les erreurs sont dans `<Errors>` et les avertissements dans `<Warnings>`.

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
        <ErrorLog Version="2018-10-01">
            <SessionId>session#1</SessionId>
            <ItemType>PageBlob</ItemType>
            <SourceDirectory>D:\Dataset\TestDirectory</SourceDirectory>
            <Errors>
                <Error Code="Not512Aligned">
                    <Description>The file is not 512 bytes aligned.</Description>
                    <List>
                        <File Path="\Practice\myScript.ps1" />
                    </List>
                    <Count>1</Count>
                </Error>
            </Errors>
            <Warnings />
        </ErrorLog>
    ```

- Voici un exemple de journal des erreurs généré lorsque le nom du conteneur n’est pas valide. Le dossier que vous créez sous les dossiers `BlockBlob`, `PageBlob`, ou `AzureFile` du disque devient un conteneur dans votre compte de stockage Azure. Le nom du conteneur doit respecter les [conventions d’affectation de noms d’Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions).

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <ErrorLog Version="2018-10-01">
        <SessionId>bbsession</SessionId>
        <ItemType>BlockBlob</ItemType>
        <SourceDirectory>E:\BlockBlob</SourceDirectory>
        <Errors>
        <Error Code="InvalidShareContainerFormat">
            <List>
            <Container Name="Azu-reFile" />
            <Container Name="bbcont ainer1" />
            </List>
            <Count>2</Count>
        </Error>
        </Errors>
        <Warnings />
    </ErrorLog>
    ```

## <a name="validation-tool-errors"></a>Erreurs de l’outil de validation

Les erreurs contenues dans le fichier *error.xml* et les actions recommandées associées sont résumées dans le tableau suivant.

| Code d'erreur| Description                       | Actions recommandées               |
|------------|--------------------------|-----------------------------------|
| `None` | Les données ont été validées avec succès. | Aucune action n'est requise. |
| `InvalidXmlCharsInPath` |Impossible de créer un fichier manifeste car le chemin d’accès du fichier contient des caractères non valides. | Supprimez ces caractères pour continuer.  |
| `OpenFileForReadFailed`| Impossible de traiter le fichier. Cela peut être dû à un problème d’accès ou un système de fichiers endommagé.|Impossible de lire le fichier en raison d’une erreur. Les détails de l’erreur sont fournis dans l’exception. |
| `Not512Aligned` | Le format de ce fichier n’est pas un format valide pour le dossier PageBlob.| Chargez uniquement des données de 512 octets alignés sur le dossier `PageBlob`. Supprimer le fichier du dossier PageBlob ou déplacez-le vers le dossier BlockBlob. Réessayez la validation.|
| `InvalidBlobPath` | Le chemin d’accès du fichier n’est pas mappé à un chemin d’accès blob valide dans le cloud, conformément aux conventions d’affectation de noms d’Azure.|Renommez le chemin d’accès du fichier en suivant les instructions d’affectation de noms d’Azure. |
| `EnumerationError` | Impossible d’énumérer le fichier pour la validation. |Cette erreur peut avoir plusieurs causes. Un problème d’accès au fichier est la raison la plus probable. |
| `ShareSizeExceeded` | Après l’ajout de ce fichier, la taille du partage de fichiers Azure a dépassé la limite d’Azure de 5 To.|Réduisez la taille des données dans le partage de manière à respecter les [limites de taille d’objet d’Azure](data-box-disk-limits.md#azure-object-size-limits). Réessayez la validation. |
| `AzureFileSizeExceeded` | La taille du fichier dépasse les limites de taille de fichier d’Azure.| Réduisez la taille du fichier ou des données de manière à respecter les [limites de taille d’objet d’Azure](data-box-disk-limits.md#azure-object-size-limits). Réessayez la validation.|
| `BlockBlobSizeExceeded` | La taille du fichier dépasse les limites de taille d’objet blob de blocs d’Azure. | Réduisez la taille du fichier ou des données de manière à respecter les [limites de taille d’objet d’Azure](data-box-disk-limits.md#azure-object-size-limits). Réessayez la validation. |
| `ManagedDiskSizeExceeded` | La taille du fichier dépasse les limites de taille de disque managé d’Azure. | Réduisez la taille du fichier ou des données de manière à respecter les [limites de taille d’objet d’Azure](data-box-disk-limits.md#azure-object-size-limits). Réessayez la validation. |
| `PageBlobSizeExceeded` | La taille du fichier dépasse les limites de taille de disque managé d’Azure. | Réduisez la taille du fichier ou des données de manière à respecter les [limites de taille d’objet d’Azure](data-box-disk-limits.md#azure-object-size-limits). Réessayez la validation. |
| `InvalidShareContainerFormat`  |Les noms de répertoires ne respectent pas les conventions d’affectation de noms d’Azure pour les conteneurs ou les partages.         |Le premier dossier créé sous les dossiers préexistants du disque devient un conteneur dans votre compte de stockage. Le nom de ce partage ou ce conteneur ne respecte pas les conventions d’affectation de noms d’Azure. Renommez le fichier de manière à respecter les [conventions d’affectation de noms d’Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions). Réessayez la validation.   |
| `InvalidBlobNameFormat` | Le chemin d’accès du fichier n’est pas mappé à un chemin d’accès blob valide dans le cloud, conformément aux conventions d’affectation de noms d’Azure.|Renommez le fichier de manière à respecter les [conventions d’affectation de noms d’Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions). Réessayez la validation. |
| `InvalidFileNameFormat` | Le chemin d’accès du fichier n’est pas mappé à un chemin d’accès de fichier valide dans le cloud, conformément aux conventions d’affectation de noms de fichier d’Azure. |Renommez le fichier de manière à respecter les [conventions d’affectation de noms d’Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions). Réessayez la validation. |
| `InvalidDiskNameFormat` | Le chemin d’accès du fichier n’est pas mappé à un nom de disque valide dans le cloud, conformément aux conventions d’affectation de noms de disque managé d’Azure. |Renommez le fichier de manière à respecter les [conventions d’affectation de noms d’Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions). Réessayez la validation.       |
| `NotPartOfFileShare` | Le chemin de chargement des fichiers n’est pas valide. Chargez les fichiers dans un dossier dans Azure Files.   | Supprimez les fichiers générant des erreurs et chargez ces fichiers dans un dossier précréé. Réessayez la validation. |
| `NonVhdFileNotSupportedForManagedDisk` | Impossible de charger un fichier non VHD en tant que disque managé. |Supprimez les fichiers non VHD du dossier `ManagedDisk` car ils ne sont pas pris en charge ou déplacez-les vers un dossier `PageBlob`. Réessayez la validation. |


## <a name="next-steps"></a>Étapes suivantes

- Résoudre les problèmes liés aux [erreurs de chargement des données](data-box-disk-troubleshoot-upload.md).
