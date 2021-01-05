---
title: Copier des objets blob entre des comptes Stockage Azure avec AzCopy v10 | Microsoft Docs
description: Cet article contient une collection d’exemples de commandes AzCopy qui vous permettent de copier des objets blob entre des comptes de stockage.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 12/08/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: f536e163e3d19d91c150506ab44fdd9cbc02c693
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96907382"
---
# <a name="copy-blobs-between-azure-storage-accounts-by-using-azcopy-v10"></a>Copier des objets blob entre des comptes Stockage Azure à l’aide d’AzCopy v10

Vous pouvez copier des objets blob, des répertoires et des conteneurs d’un compte de stockage à un autre à l’aide de l’utilitaire en ligne de commande AzCopy v10. 

Pour obtenir des exemples d’autres types de tâches, telles que le chargement de fichiers, le téléchargement d’objets blob et la synchronisation avec le stockage d’objets blob, consultez les liens présentés dans la section [Étapes suivantes](#next-steps) de cet article.

AzCopy utilise des [API](/rest/api/storageservices/put-page-from-url) [serveur à serveur](/rest/api/storageservices/put-block-from-url), de sorte que les données sont copiées directement entre les serveurs de stockage. Ces opérations de copie n’utilisent pas la bande passante réseau de votre ordinateur.

Pour télécharger AzCopy et découvrir comment fournir des informations d’identification d’autorisation au service de stockage, consultez [Bien démarrer avec AzCopy](storage-use-azcopy-v10.md). 

## <a name="guidelines"></a>Consignes

Respectez les consignes suivantes lors de l’exécution de commandes AzCopy. 

- Ajoutez un jeton SAS à chaque URL source. 

  Si vous fournissez des informations d’identification d’autorisation à l’aide d’Azure AD (Azure Active Directory), vous pouvez omettre le jeton SAS uniquement dans l’URL de destination. Assurez-vous que vous avez configuré les rôles appropriés dans votre compte de destination. Voir [Option 1 : Utiliser Azure Active Directory](storage-use-azcopy-v10.md?toc=/azure/storage/blobs/toc.json#option-1-use-azure-active-directory). 

  Les exemples de cet article partent du principe que vous avez authentifié votre identité à l’aide d’Azure AD ; ils omettent par conséquent les jetons SAS de l’URL de destination.

-  Si vous effectuez une copie vers un compte de stockage d’objets blob de blocs Premium, omettez le niveau d’accès d’un objet blob de l’opération de copie en affectant la valeur `false` à `s2s-preserve-access-tier` (par exemple : `--s2s-preserve-access-tier=false`). Les comptes de stockage d’objet blob de blocs Premium ne prennent pas en charge les niveaux d’accès. 

- Si vous copiez vers ou à partir d’un compte ayant un espace de noms hiérarchique, utilisez `blob.core.windows.net` au lieu de `dfs.core.windows.net` dans la syntaxe de l’URL. L’[accès multi-protocole pour Azure Data Lake Storage](../blobs/data-lake-storage-multi-protocol-access.md) vous permet d’utiliser `blob.core.windows.net`, et il s’agit de la seule syntaxe prise en charge pour les scénarios de copie de compte à compte. 

- Vous pouvez augmenter le débit des opérations de copie en définissant la valeur de la variable d’environnement `AZCOPY_CONCURRENCY_VALUE`. Pour en savoir plus, voir [Optimiser le débit](storage-use-azcopy-configure.md#optimize-throughput). 

- Si les objets blob sources contiennent des balises d’index et que vous souhaitez conserver ces balises, vous devez les réappliquer aux objets blob de destination. Pour plus d’informations sur la façon de définir des balises d’index, consultez la section [Copier des objets blob vers un autre compte de stockage avec des balises d’index](#copy-between-accounts-and-add-index-tags) de cet article.

## <a name="copy-a-blob"></a>Copier un objet blob

Copiez un objet blob vers un autre compte de stockage à l’aide de la commande [azcopy copy](storage-ref-azcopy-copy.md). 

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>/<blob-path><SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>/<blob-path>'` |
| **Exemple** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer/myTextFile.txt?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer/myTextFile.txt'` |

L’opération de copie étant synchrone, lorsque la commande retourne un résultat, cela indique que tous les fichiers ont été copiés. 

## <a name="copy-a-directory"></a>Copier un répertoire

Copiez un répertoire vers un autre compte de stockage à l’aide de la commande [azcopy copy](storage-ref-azcopy-copy.md). 

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>/<directory-path><SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>' --recursive` |
| **Exemple** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer/myBlobDirectory?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive` |

L’opération de copie étant synchrone, lorsque la commande retourne un résultat, cela indique que tous les fichiers ont été copiés.

## <a name="copy-a-container"></a>Copier un conteneur

Copiez un conteneur vers un autre compte de stockage à l’aide de la commande [azcopy copy](storage-ref-azcopy-copy.md).

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<container-name><SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>' --recursive` |
| **Exemple** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive` |

L’opération de copie étant synchrone, lorsque la commande retourne un résultat, cela indique que tous les fichiers ont été copiés.

## <a name="copy-containers-directories-and-blobs"></a>Copier des conteneurs, des répertoires et des objets blob

Copier tous les conteneurs, répertoires et objets blob vers un autre compte de stockage à l’aide de la commande [azcopy copy](storage-ref-azcopy-copy.md).

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/' --recursive` |
| **Exemple** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net' --recursive` |

L’opération de copie étant synchrone, lorsque la commande retourne un résultat, cela indique que tous les fichiers ont été copiés.

<a id="copy-between-accounts-and-add-index-tags"></a>

## <a name="copy-blobs-and-add-index-tags"></a>Copier des objets blob et ajouter des balises d’index

Copiez des objets blob vers un autre compte de stockage et ajouter des [balises d’index d’objet blob (préversion)](../blobs/storage-manage-find-blobs.md) à l’objet blob cible.

Si vous utilisez l’autorisation Azure AD, votre principal de sécurité doit détenir le rôle [Propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner), ou l’autorisation d’accès à l’[opération du fournisseur de ressources Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` doit lui être accordée par le biais d’un rôle Azure personnalisé. Si vous utilisez un jeton de signature d’accès partagé (SAS), ce jeton doit fournir l’accès aux balises de l’objet blob par le biais de l’autorisation SAS `t`.

Pour ajouter des balises, utilisez l’option `--blob-tags` avec une paire clé-valeur encodée en URL. 

Par exemple, pour ajouter la clé `my tag` et une valeur `my tag value`, vous devez ajouter `--blob-tags='my%20tag=my%20tag%20value'` au paramètre de destination. 

Séparez plusieurs balises d’index à l’aide d’une esperluette (`&`).  Par exemple, si vous souhaitez ajouter une clé `my second tag` et une valeur `my second tag value`, la chaîne d’option complète est `--blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'`.

Les exemples suivants montrent comment utiliser l’option `--blob-tags`.

> [!TIP]
> Ces exemples englobent les arguments de chemin avec des guillemets simples (''). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

|    |     |
|--------|-----------|
| **Objet blob** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer/myTextFile.txt?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer/myTextFile.txt' --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'` |
| **Directory** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer/myBlobDirectory?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'` |
| **Conteneur** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive --blob-tags="--blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'` |
| **Compte** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net' --recursive --blob-tags="--blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'` |

L’opération de copie étant synchrone, lorsque la commande retourne un résultat, cela indique que tous les fichiers ont été copiés.

> [!NOTE]
> Si vous spécifiez un répertoire, un conteneur ou un compte comme source, tous les objets blob copiés vers la destination auront les mêmes balises que celles que vous spécifiez dans la commande.

## <a name="copy-with-optional-flags"></a>Copier avec des indicateurs facultatifs

Vous pouvez ajuster votre opération de copie à l’aide d’indicateurs facultatifs. Voici quelques exemples.

|Scénario|Indicateur|
|---|---|
|Copier les objets blob en tant que blobs de bloc, de page ou d’ajout.|**--blob-type**=\[BlockBlob\|PageBlob\|AppendBlob\]|
|Effectuer une copie vers un niveau d’accès spécifique (tel que le niveau Archive)|**--block-blob-tier**=\[None\|Hot\|Cool\|Archive\]|
|Décompresser automatiquement les fichiers|**--decompress**=\[gzip\|deflate\]|

Pour obtenir la liste complète, consultez [Options](storage-ref-azcopy-copy.md#options). 

## <a name="next-steps"></a>Étapes suivantes

Vous trouverez plus d’exemples dans ces articles :

- [Exemples : Charger](storage-use-azcopy-blobs-upload.md)
- [Exemples : Téléchargement](storage-use-azcopy-blobs-download.md)
- [Exemples : Synchroniser](storage-use-azcopy-blobs-synchronize.md)
- [Exemples : Compartiments Amazon S3](storage-use-azcopy-s3.md)
- [Exemples : Azure Files](storage-use-azcopy-files.md)
- [Tutoriel : Migrer des données locales vers un stockage cloud à l’aide d’AzCopy](storage-use-azcopy-migrate-on-premises-data.md)
- [Configurer, optimiser et dépanner AzCopy](storage-use-azcopy-configure.md)