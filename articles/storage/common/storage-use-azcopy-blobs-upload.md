---
title: Charger des fichiers vers Stockage Blob Azure avec AzCopy v10 | Microsoft Docs
description: Cet article contient une collection d’exemples de commandes AzCopy qui vous permettent de charger des fichiers vers Stockage Blob Azure.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 12/11/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: ec88a3c740ceda7ccf352f8f32f94e2cd52d0988
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358756"
---
# <a name="upload-files-to-azure-blob-storage-by-using-azcopy-v10"></a>Charger des fichiers vers Stockage Blob Azure avec AzCopy v10

Vous pouvez charger des fichiers et des répertoires vers le stockage d’objets blob à l’aide de l’utilitaire en ligne de commande AzCopy v10. 

Pour obtenir des exemples d’autres types de tâches, telles que le téléchargement d’objets blob, la synchronisation avec le stockage d’objets blob ou la copie d’objets blob entre des comptes, consultez les liens présentés dans la section [Étapes suivantes](#next-steps) de cet article.

## <a name="get-started"></a>Bien démarrer

Consultez l’article [Prise en main d’AzCopy](storage-use-azcopy-v10.md) pour télécharger AzCopy, et découvrez comment vous pouvez fournir des informations d’identification au service de stockage.

> [!NOTE] 
> Les exemples de cet article partent du principe que vous avez fourni des informations d’identification d’autorisation en utilisant Azure AD (Azure Active Directory).
>
> Si vous préférez utiliser un jeton SAS pour autoriser l’accès aux données d’objets blob, vous pouvez ajouter ce jeton à l’URL de ressource dans chaque commande AzCopy. Par exemple : `'https://<storage-account-name>.blob.core.windows.net/<container-name><SAS-token>'`.

## <a name="create-a-container"></a>Créez un conteneur.

Vous pouvez utiliser la commande [azcopy make](storage-ref-azcopy-make.md) pour créer un conteneur.

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy make 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>'` |
| **Exemple** | `azcopy make 'https://mystorageaccount.blob.core.windows.net/mycontainer'` |
| **Exemple** (espace de noms hiérarchique) | `azcopy make 'https://mystorageaccount.dfs.core.windows.net/mycontainer'` |

Pour obtenir des informations de référence détaillées, consultez [azcopy make](storage-ref-azcopy-make.md).

## <a name="upload-a-file"></a>Charger un fichier

Chargez un fichier à l’aide de la commande [azcopy copy](storage-ref-azcopy-copy.md).

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy '<local-file-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-name>'` |
| **Exemple** | `azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt'` |
| **Exemple** (espace de noms hiérarchique) | `azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myTextFile.txt'` |

Vous pouvez également charger un fichier en plaçant un caractère générique (*) n’importe où dans le chemin ou le nom de fichier. Par exemple : `'C:\myDirectory\*.txt'` ou `C:\my*\*.txt`.

## <a name="upload-a-directory"></a>Charger un annuaire

Chargez un répertoire à l’aide de la commande [azcopy copy](storage-ref-azcopy-copy.md). 

Cet exemple copie un répertoire (et tous les fichiers qu’il contient) dans un conteneur d’objets blob. Le conteneur contient alors un répertoire du même nom.

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy '<local-directory-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>' --recursive` |
| **Exemple** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive` |
| **Exemple** (espace de noms hiérarchique) | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.dfs.core.windows.net/mycontainer' --recursive` |

Pour copier des éléments dans un répertoire du conteneur, spécifiez simplement le nom de ce répertoire dans votre chaîne de commande.

|    |     |
|--------|-----------|
| **Exemple** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory' --recursive` |
| **Exemple** (espace de noms hiérarchique) | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myBlobDirectory' --recursive` |

Si vous spécifiez le nom d’un répertoire qui n’existe pas dans le conteneur, AzCopy crée un répertoire portant ce nom.

## <a name="upload-directory-contents"></a>Charger le contenu d’un répertoire

Chargez le contenu d’un répertoire à l’aide de la commande [azcopy copy](storage-ref-azcopy-copy.md). Utilisez le caractère générique (*) pour charger le contenu sans copier le répertoire contenant lui-même.

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy '<local-directory-path>\*' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<directory-path>'` |
| **Exemple** | `azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory'` |
| **Exemple** (espace de noms hiérarchique) | `azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myBlobDirectory'` |


Ajoutez l’indicateur `--recursive` pour charger des fichiers dans tous les sous-répertoires.

## <a name="upload-specific-files"></a>Charger des fichiers spécifiques

Vous pouvez charger des fichiers spécifiques en utilisant des noms de fichiers complets, des noms partiels avec des caractères génériques (*), ou des dates et des heures.

> [!TIP]
> Ces exemples englobent les arguments de chemin avec des guillemets simples (''). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

### <a name="specify-multiple-complete-file-names"></a>Spécifier plusieurs noms de fichiers complets

Utilisez la commande [azcopy copy](storage-ref-azcopy-copy.md) avec l’option `--include-path`. Séparez les noms de fichiers à l’aide d’un point-virgule (`;`).

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy '<local-directory-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>' --include-path <semicolon-separated-file-list>` |
| **Exemple** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --include-path 'photos;documents\myFile.txt' --recursive` |
| **Exemple** (espace de noms hiérarchique) | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.dfs.core.windows.net/mycontainer' --include-path 'photos;documents\myFile.txt' --recursive` |

Dans cet exemple, AzCopy transfère le répertoire `C:\myDirectory\photos` et le fichier `C:\myDirectory\documents\myFile.txt`. Incluez l’option `--recursive` pour transférer tous les fichiers dans le répertoire `C:\myDirectory\photos`.

Vous pouvez également exclure des fichiers à l’aide de l’option `--exclude-path`. Pour plus d’informations, consultez la documentation de référence sur [azcopy copy](storage-ref-azcopy-copy.md).

### <a name="use-wildcard-characters"></a>Utiliser des caractères génériques

Utilisez la commande [azcopy copy](storage-ref-azcopy-copy.md) avec l’option `--include-pattern`. Spécifiez des noms partiels qui incluent les caractères génériques. Séparez les noms à l’aide d’un point-virgule (`;`). 

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy '<local-directory-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>' --include-pattern <semicolon-separated-file-list-with-wildcard-characters>` |
| **Exemple** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --include-pattern 'myFile*.txt;*.pdf*'` |
| **Exemple** (espace de noms hiérarchique) | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.dfs.core.windows.net/mycontainer' --include-pattern 'myFile*.txt;*.pdf*'` |

Vous pouvez également exclure des fichiers à l’aide de l’option `--exclude-pattern`. Pour plus d’informations, consultez la documentation de référence sur [azcopy copy](storage-ref-azcopy-copy.md).

Les options `--include-pattern` et `--exclude-pattern` s’appliquent uniquement aux noms de fichiers, et non au chemin.  Si vous souhaitez copier tous les fichiers texte qui existent dans une arborescence de répertoires, utilisez l’option `–recursive` pour obtenir la totalité de l’arborescence de répertoires, puis utilisez `–include-pattern` et spécifiez `*.txt` pour obtenir tous les fichiers texte.

### <a name="upload-files-that-were-modified-before-or-after-a-date-and-time"></a>Charger les fichiers qui ont été modifiés avant ou après une date et une heure 

Utilisez la commande [azcopy copy](storage-ref-azcopy-copy.md) avec l’option `--include-before` ou `--include-after`. Spécifiez une date et une heure au format ISO 8601 (par exemple : `2020-08-19T15:04:00Z`). 

Les exemples suivants chargent les fichiers qui ont été modifiés à la date spécifiée ou après celle-ci.

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy '<local-directory-path>\*' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>'  --include-after <Date-Time-in-ISO-8601-format>` |
| **Exemple** | `azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory'  --include-after '2020-08-19T15:04:00Z'` |
| **Exemple** (espace de noms hiérarchique) | `azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory'   --include-after '2020-08-19T15:04:00Z'` |

Pour obtenir des informations de référence détaillées, consultez les documents de référence [azcopy copy](storage-ref-azcopy-copy.md).

## <a name="upload-with-index-tags"></a>Charger avec des balises d’index

Vous pouvez charger un fichier et ajouter des [balises d’index d’objet blob (préversion)](../blobs/storage-manage-find-blobs.md) à l’objet blob cible.  

Si vous utilisez l’autorisation Azure AD, votre principal de sécurité doit détenir le rôle [Propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner), ou l’autorisation d’accès à l’[opération du fournisseur de ressources Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` doit lui être accordée par le biais d’un rôle Azure personnalisé. Si vous utilisez un jeton de signature d’accès partagé (SAS), ce jeton doit fournir l’accès aux balises de l’objet blob par le biais de l’autorisation SAS `t`.

Pour ajouter des balises, utilisez l’option `--blob-tags` avec une paire clé-valeur encodée en URL. Par exemple, pour ajouter la clé `my tag` et une valeur `my tag value`, vous devez ajouter `--blob-tags='my%20tag=my%20tag%20value'` au paramètre de destination. 

Séparez plusieurs balises d’index à l’aide d’une esperluette (`&`).  Par exemple, si vous souhaitez ajouter une clé `my second tag` et une valeur `my second tag value`, la chaîne d’option complète est `--blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'`.

Les exemples suivants montrent comment utiliser l’option `--blob-tags`.

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

|    |     |
|--------|-----------|
| **Charger un fichier** | `azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt' --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'` |
| **Charger un annuaire** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'`|
| **Charger le contenu d’un répertoire** | `azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory' --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'` |

> [!NOTE]
> Si vous spécifiez un répertoire comme source, tous les objets blob copiés vers la destination auront les mêmes balises que celles que vous spécifiez dans la commande.

## <a name="upload-with-optional-flags"></a>Charger avec des indicateurs facultatifs

Vous pouvez ajuster votre opération de chargement à l’aide d’indicateurs facultatifs. Voici quelques exemples.

|Scénario|Indicateur|
|---|---|
|Charger des fichiers sous forme d’objets blob d’ajout ou de pages|**--blob-type**=\[BlockBlob\|PageBlob\|AppendBlob\]|
|Effectuer un chargement sur un niveau d’accès spécifique (tel que le niveau Archive)|**--block-blob-tier**=\[None\|Hot\|Cool\|Archive\]|

Pour obtenir la liste complète, consultez [Options](storage-ref-azcopy-copy.md#options).

## <a name="next-steps"></a>Étapes suivantes

Vous trouverez plus d’exemples dans ces articles :

- [Exemples : Téléchargement](storage-use-azcopy-blobs-download.md)
- [Exemples : Copie entre comptes](storage-use-azcopy-blobs-copy.md)
- [Exemples : Synchroniser](storage-use-azcopy-blobs-synchronize.md)
- [Exemples : Compartiments Amazon S3](storage-use-azcopy-s3.md)
- [Exemples : Azure Files](storage-use-azcopy-files.md)
- [Tutoriel : Migrer des données locales vers un stockage cloud à l’aide d’AzCopy](storage-use-azcopy-migrate-on-premises-data.md)
- [Configurer, optimiser et dépanner AzCopy](storage-use-azcopy-configure.md)