---
title: Télécharger des blobs à partir de Stockage Blob Azure en utilisant AzCopy v10 | Microsoft Docs
description: Cet article contient une collection d’exemples de commandes AzCopy qui vous permettent de télécharger des blobs à partir de Stockage Blob Azure.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 12/11/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: 382adb36712fbf4bee83044c8b2d096223eb6269
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97630035"
---
# <a name="download-blobs-from-azure-blob-storage-by-using-azcopy-v10"></a>Télécharger des blobs à partir de Stockage Blob Azure en utilisant AzCopy v10

Vous pouvez télécharger des blobs et des répertoires à partir du stockage Blob à l’aide de l’utilitaire en ligne de commande AzCopy v10. 

Pour obtenir des exemples d’autres types de tâches, telles que le chargement de fichiers, la synchronisation avec le stockage Blob ou la copie de blobs entre des comptes, consultez les liens présentés dans la section [Étapes suivantes](#next-steps) de cet article.

## <a name="get-started"></a>Bien démarrer

Consultez l’article [Prise en main d’AzCopy](storage-use-azcopy-v10.md) pour télécharger AzCopy, et découvrez comment vous pouvez fournir des informations d’identification au service de stockage.

> [!NOTE] 
> Les exemples de cet article partent du principe que vous avez fourni des informations d’identification d’autorisation en utilisant Azure AD (Azure Active Directory).
>
> Si vous préférez utiliser un jeton SAS pour autoriser l’accès aux données d’objets blob, vous pouvez ajouter ce jeton à l’URL de ressource dans chaque commande AzCopy. Par exemple : `'https://<storage-account-name>.blob.core.windows.net/<container-name><SAS-token>'`.

## <a name="download-a-blob"></a>Télécharger un objet blob

Téléchargez un blob à l’aide de la commande [azcopy copy](storage-ref-azcopy-copy.md).

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-path>' '<local-file-path>'` |
| **Exemple** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt' 'C:\myDirectory\myTextFile.txt'` |
| **Exemple** (espace de noms hiérarchique) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myTextFile.txt' 'C:\myDirectory\myTextFile.txt'` |

> [!NOTE]
> Si la valeur de propriété `Content-md5` d’un objet blob contient un code de hachage, AzCopy calcule un code de hachage MD5 pour les données téléchargées et vérifie que le code de hachage MD5 stocké dans la propriété `Content-md5` de l’objet blob correspond au code de hachage calculé. Si ces valeurs ne correspondent pas, le téléchargement échoue, sauf si vous remplacez ce comportement en ajoutant `--check-md5=NoCheck` ou `--check-md5=LogOnly` à la commande de copie.

## <a name="download-a-directory"></a>Télécharger un annuaire

Téléchargez un répertoire à l’aide de la commande [azcopy copy](storage-ref-azcopy-copy.md).

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<directory-path>' '<local-directory-path>' --recursive` |
| **Exemple** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory' 'C:\myDirectory'  --recursive` |
| **Exemple** (espace de noms hiérarchique) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myBlobDirectory' 'C:\myDirectory'  --recursive` |

Cet exemple illustre la création d’un répertoire nommé `C:\myDirectory\myBlobDirectory` qui contient tous les blobs téléchargés.

## <a name="download-directory-contents"></a>Télécharger le contenu d’un répertoire

Vous pouvez télécharger le contenu d’un répertoire sans copier le répertoire proprement dit en utilisant le caractère générique (*).

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

> [!NOTE]
> Actuellement, ce scénario est pris en charge uniquement pour les comptes qui ne disposent pas d’un espace de noms hiérarchique.

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://<storage-account-name>.blob.core.windows.net/<container-name>/*' '<local-directory-path>/'` |
| **Exemple** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory/*' 'C:\myDirectory'` |

Ajoutez l’indicateur `--recursive` pour télécharger des fichiers dans tous les sous-répertoires.

## <a name="download-specific-blobs"></a>Télécharger des blobs spécifiques

Vous pouvez télécharger des blobs spécifiques en utilisant des noms de fichiers complets, des noms partiels avec des caractères génériques (*), ou des dates et des heures. 

> [!TIP]
> Ces exemples englobent les arguments de chemin avec des guillemets simples (''). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

#### <a name="specify-multiple-complete-blob-names"></a>Spécifier plusieurs noms de blob complets

Utilisez la commande [azcopy copy](storage-ref-azcopy-copy.md) avec l’option `--include-path`. Séparez les noms des blobs en utilisant un point-virgule (`;`).

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>' '<local-directory-path>'  --include-path <semicolon-separated-file-list>` |
| **Exemple** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-path 'photos;documents\myFile.txt' --recursive` |
| **Exemple** (espace de noms hiérarchique) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-path 'photos;documents\myFile.txt'--recursive` |

Dans cet exemple, AzCopy transfère le répertoire `https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/photos` et le fichier `https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/documents/myFile.txt`. Incluez l’option `--recursive` pour transférer tous les blobs dans le répertoire `https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/photos`.

Vous pouvez également exclure des blobs à l’aide de l’option `--exclude-path`. Pour plus d’informations, consultez la documentation de référence sur [azcopy copy](storage-ref-azcopy-copy.md).

#### <a name="use-wildcard-characters"></a>Utiliser des caractères génériques

Utilisez la commande [azcopy copy](storage-ref-azcopy-copy.md) avec l’option `--include-pattern`. Spécifiez des noms partiels qui incluent les caractères génériques. Séparez les noms à l’aide d’un point-virgule (`;`).

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>' '<local-directory-path>' --include-pattern <semicolon-separated-file-list-with-wildcard-characters>` |
| **Exemple** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-pattern 'myFile*.txt;*.pdf*'` |
| **Exemple** (espace de noms hiérarchique) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-pattern 'myFile*.txt;*.pdf*'` |

Vous pouvez également exclure des blobs à l’aide de l’option `--exclude-pattern`. Pour plus d’informations, consultez la documentation de référence sur [azcopy copy](storage-ref-azcopy-copy.md).

Les options `--include-pattern` et `--exclude-pattern` s’appliquent uniquement aux noms de blob, et non au chemin.  Si vous souhaitez copier tous les fichiers texte (objets blob) qui existent dans une arborescence de répertoires, utilisez l’option `–recursive` pour obtenir la totalité de l’arborescence de répertoires, puis utilisez `–include-pattern` et spécifiez `*.txt` pour obtenir tous les fichiers texte.

#### <a name="download-blobs-that-were-modified-before-or-after-a-date-and-time"></a>Télécharger les blobs qui ont été modifiés avant ou après une date et une heure 

Utilisez la commande [azcopy copy](storage-ref-azcopy-copy.md) avec l’option `--include-before` ou `--include-after`. Spécifiez une date et une heure au format ISO 8601 (par exemple : `2020-08-19T15:04:00Z`). 

Les exemples suivants téléchargent les fichiers qui ont été modifiés à la date spécifiée ou après celle-ci.

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>/*' '<local-directory-path>' --include-after <Date-Time-in-ISO-8601-format>` |
| **Exemple** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/*' 'C:\myDirectory'  --include-after '2020-08-19T15:04:00Z'` |
| **Exemple** (espace de noms hiérarchique) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory/*' 'C:\myDirectory'  --include-after '2020-08-19T15:04:00Z'` |

Pour obtenir des informations de référence détaillées, consultez les documents de référence [azcopy copy](storage-ref-azcopy-copy.md).

#### <a name="download-previous-versions-of-a-blob"></a>Télécharger les versions précédentes d’un objet blob

Si vous avez activé le [contrôle de version](../blobs/versioning-enable.md), vous pouvez télécharger une ou plusieurs versions précédentes d’un objet blob. 

Tout d’abord, créez un fichier texte qui contient une liste d’[ID de version](../blobs/versioning-overview.md). Chaque ID de version doit figurer sur une ligne distincte. Par exemple : 

```
2020-08-17T05:50:34.2199403Z
2020-08-17T05:50:34.5041365Z
2020-08-17T05:50:36.7607103Z
```

Utilisez ensuite la commande [azcopy copy](storage-ref-azcopy-copy.md) avec l’option `--list-of-versions`. Spécifiez l’emplacement du fichier texte qui contient la liste des versions (par exemple : `D:\\list-of-versions.txt`).  

#### <a name="download-a-blob-snapshot"></a>Télécharger un instantané d’objet blob

Vous pouvez télécharger un [instantané de blob](/azure/storage/blobs/snapshots-overview) en référençant la valeur **DateTime** d’un instantané de blob. 

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-path>?sharesnapshot=<DateTime-of-snapshot>' '<local-file-path>'` |
| **Exemple** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt?sharesnapshot=2020-09-23T08:21:07.0000000Z' 'C:\myDirectory\myTextFile.txt'` |
| **Exemple** (espace de noms hiérarchique) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myTextFile.txt?sharesnapshot=2020-09-23T08:21:07.0000000Z' 'C:\myDirectory\myTextFile.txt'` |

> [!NOTE]
> Si vous utilisez un jeton SAS pour autoriser l’accès aux données blob, ajoutez le **DateTime** de l’instantané après le jeton SAS. Par exemple : `'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D&sharesnapshot=2020-09-23T08:21:07.0000000Z'`.

## <a name="download-with-optional-flags"></a>Télécharger avec des indicateurs facultatifs

Vous pouvez ajuster votre opération de téléchargement à l’aide d’indicateurs facultatifs. Voici quelques exemples.

|Scénario|Indicateur|
|---|---|
|Décompresser automatiquement les fichiers|**DECOMPRESS**|
|Spécifier le niveau de détail des entrées de journal liées à la copie|**--log-level**=\[WARNING\|ERROR\|INFO\|NONE\]|
|Spécifier s’il faut remplacer les fichiers et objets blob en conflit dans la destination, et comment le faire|**--overwrite**=\[true\|false\|ifSourceNewer\|prompt\]|

Pour obtenir la liste complète, consultez [Options](storage-ref-azcopy-copy.md#options).

## <a name="next-steps"></a>Étapes suivantes

Vous trouverez plus d’exemples dans ces articles :

- [Exemples : Charger](storage-use-azcopy-blobs-upload.md)
- [Exemples : Copie entre comptes](storage-use-azcopy-blobs-copy.md)
- [Exemples : Synchroniser](storage-use-azcopy-blobs-synchronize.md)
- [Exemples : Compartiments Amazon S3](storage-use-azcopy-s3.md)
- [Exemples : Azure Files](storage-use-azcopy-files.md)
- [Tutoriel : Migrer des données locales vers un stockage cloud à l’aide d’AzCopy](storage-use-azcopy-migrate-on-premises-data.md)
- [Configurer, optimiser et dépanner AzCopy](storage-use-azcopy-configure.md)