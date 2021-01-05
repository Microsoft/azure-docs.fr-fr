---
title: Synchroniser avec Stockage Blob Azure à l’aide d’AzCopy v10 | Microsoft Docs
description: Cet article contient une collection d’exemples de commandes AzCopy qui vous aident à synchroniser avec Stockage Blob Azure.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 12/08/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: 6d1e9e8eeddaaa2ce8c891888935faad12d40295
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96907383"
---
# <a name="synchronize-with-azure-blob-storage-by-using-azcopy-v10"></a>Synchroniser avec Stockage Blob Azure à l’aide d’AzCopy v10

Vous pouvez synchroniser le stockage local avec Stockage Blob Azure à l’aide de l’utilitaire en ligne de commande AzCopy v10. 

Vous pouvez synchroniser le contenu d’un système de fichiers local avec un conteneur d’objets blob. Vous pouvez également synchroniser des conteneurs et des répertoires virtuels les uns avec les autres. La synchronisation est unidirectionnelle. En d’autres termes, vous choisissez lequel de ces deux points de terminaison est la source et lequel est la destination. La synchronisation utilise également un serveur pour les API de serveur. Les exemples présentés dans cette section fonctionnent également avec les comptes qui ont un espace de noms hiérarchique. 

> [!NOTE]
> La version actuelle de l’utilitaire AzCopy n’effectue pas de synchronisation entre d’autres sources et destinations (exemple : stockage de fichiers ou compartiments Amazon Web Services (AWS) S3).

Pour obtenir des exemples d’autres types de tâches, telles que le chargement de fichiers, le téléchargement d’objets blob ou la copie d’objets blob entre des comptes, consultez les liens présentés dans la section [Étapes suivantes](#next-steps) de cet article.

## <a name="get-started"></a>Bien démarrer

Consultez l’article [Prise en main d’AzCopy](storage-use-azcopy-v10.md) pour télécharger AzCopy, et découvrez comment vous pouvez fournir des informations d’identification au service de stockage.

> [!NOTE] 
> Les exemples de cet article partent du principe que vous avez fourni des informations d’identification d’autorisation en utilisant Azure AD (Azure Active Directory).
>
> Si vous préférez utiliser un jeton SAS pour autoriser l’accès aux données d’objets blob, vous pouvez ajouter ce jeton à l’URL de ressource dans chaque commande AzCopy. Par exemple : `'https://<storage-account-name>.blob.core.windows.net/<container-name><SAS-token>'`.ken>'`.

## <a name="guidelines"></a>Consignes

- La commande [sync](storage-ref-azcopy-sync.md) compare les noms de fichiers et les horodatages de la dernière modification. Définissez l’indicateur facultatif `--delete-destination` sur la valeur `true` ou `prompt` pour supprimer des fichiers dans le répertoire de destination si ces fichiers n’existent plus dans le répertoire source.

- Si vous affectez la valeur `true` à l’indicateur `--delete-destination`, AzCopy supprime les fichiers sans invite. Si vous souhaitez qu’une invite s’affiche avant que l’utilitaire AzCopy ne supprime un fichier, définissez l’indicateur `--delete-destination` sur `prompt`.

- Pour empêcher les suppressions accidentelles, veillez à activer la fonctionnalité de [suppression réversible](../blobs/soft-delete-blob-overview.md) avant d’utiliser l’indicateur `--delete-destination=prompt|true`.

## <a name="update-a-container-with-changes-to-a-local-file-system"></a>Mettre à jour un conteneur avec les modifications apportées à un système de fichiers local

Dans ce cas, le conteneur est la destination, et le système de fichiers local est la source. 

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy sync '<local-directory-path>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>' --recursive` |
| **Exemple** | `azcopy sync 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive` |

## <a name="update-a-local-file-system-with-changes-to-a-container"></a>Mettre à jour un système de fichiers local avec les modifications apportées à un conteneur

Dans ce cas, le système de fichiers local est la destination, et le conteneur est la source.

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy sync 'https://<storage-account-name>.blob.core.windows.net/<container-name>' 'C:\myDirectory' --recursive` |
| **Exemple** | `azcopy sync 'https://mystorageaccount.blob.core.windows.net/mycontainer' 'C:\myDirectory' --recursive` |

## <a name="update-a-container-with-changes-in-another-container"></a>Mettre à jour un conteneur avec les modifications d’un autre conteneur

Le premier conteneur qui s’affiche dans cette commande est la source. Le deuxième est la destination.

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy sync 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>' --recursive` |
| **Exemple** | `azcopy sync 'https://mysourceaccount.blob.core.windows.net/mycontainer' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive` |

## <a name="update-a-directory-with-changes-to-a-directory-in-another-file-share"></a>Mettre à jour un répertoire avec les modifications apportées à un répertoire dans un autre partage de fichiers

Le premier répertoire qui s’affiche dans cette commande est la source. Le deuxième est la destination.

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy sync 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' --recursive` |
| **Exemple** | `azcopy sync 'https://mysourceaccount.blob.core.windows.net/<container-name>/myDirectory' 'https://mydestinationaccount.blob.core.windows.net/mycontainer/myDirectory' --recursive` |

## <a name="synchronize-with-optional-flags"></a>Synchroniser avec des indicateurs facultatifs

Vous pouvez ajuster votre opération de synchronisation à l’aide d’indicateurs facultatifs. Voici quelques exemples.

|Scénario|Indicateur|
|---|---|
|Spécifier la manière dont les hachages MD5 doivent être validés lors du téléchargement|**--check-md5**=\[NoCheck\|LogOnly\|FailIfDifferent\|FailIfDifferentOrMissing\]|
|Exclure des fichiers en fonction d’un modèle|**--exclude-path**|
|Spécifier le niveau de détail des entrées de journal liées à la synchronisation|**--log-level**=\[WARNING\|ERROR\|INFO\|NONE\]|

Pour obtenir la liste complète, consultez [Options](storage-ref-azcopy-sync.md#options).

## <a name="next-steps"></a>Étapes suivantes

Vous trouverez plus d’exemples dans ces articles :

- [Exemples : Charger](storage-use-azcopy-blobs-upload.md)
- [Exemples : Téléchargement](storage-use-azcopy-blobs-download.md)
- [Exemples : Copier entre comptes](storage-use-azcopy-blobs-copy.md)
- [Exemples : Compartiments Amazon S3](storage-use-azcopy-s3.md)
- [Exemples : Azure Files](storage-use-azcopy-files.md)
- [Tutoriel : Migrer des données locales vers un stockage cloud à l’aide d’AzCopy](storage-use-azcopy-migrate-on-premises-data.md)
- [Configurer, optimiser et dépanner AzCopy](storage-use-azcopy-configure.md)