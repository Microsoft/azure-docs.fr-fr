---
title: Copier des données depuis Amazon S3 vers le Stockage Azure avec AzCopy | Microsoft Docs
description: Utilisez AzCopy pour copier des données depuis Amazon S3 vers Stockage Azure. AzCopy est un utilitaire de ligne de commande que vous pouvez utiliser pour copier des blobs ou des fichiers vers ou depuis un compte de stockage.
services: storage
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 07/27/2020
ms.author: normesta
ms.subservice: common
ms.openlocfilehash: 07a8d2b394e8ca690925c677af676643064a9ba8
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96901828"
---
# <a name="copy-data-from-amazon-s3-to-azure-storage-by-using-azcopy"></a>Copier des données depuis Amazon S3 vers le Stockage Azure avec AzCopy

AzCopy est un utilitaire de ligne de commande que vous pouvez utiliser pour copier des blobs ou des fichiers vers ou depuis un compte de stockage. Cet article vous aide à copier des objets, des répertoires et des compartiments d’Amazon Web Services (AWS) S3 vers Stockage Blob Azure avec AzCopy.

## <a name="choose-how-youll-provide-authorization-credentials"></a>Choisissez comment vous allez fournir des informations d’identification d’autorisation.

* Pour autoriser avec Stockage Azure, utilisez Azure Active Directory (AD) ou un jeton SAS (signature d’accès partagé).

* Pour autoriser avec AWS S3, utilisez une clé d’accès AWS et une clé d’accès secrète.

### <a name="authorize-with-azure-storage"></a>Autoriser avec Stockage Azure

Consultez l’article [Prise en main d’AzCopy](storage-use-azcopy-v10.md) pour télécharger AzCopy, et choisissez comment vous allez fournir les informations d’identification d’autorisation au service de stockage.

> [!NOTE]
> Les exemples de cet article partent du principe que vous vous êtes authentifié à l’aide de la commande `AzCopy login`. AzCopy utilise ensuite votre compte Azure AD pour autoriser l’accès aux données dans le stockage d’objet blob.
>
> Si vous préférez utiliser un jeton SAS pour autoriser l’accès aux données d’objets blob, vous pouvez ajouter ce jeton à l’URL de ressource dans chaque commande AzCopy.
>
> Par exemple : `https://mystorageaccount.blob.core.windows.net/mycontainer?<SAS-token>`.

### <a name="authorize-with-aws-s3"></a>Autoriser avec AWS S3

Récupérez votre clé d’accès AWS et votre clé d’accès secrète, puis définissez ces variables d’environnement :

| Système d’exploitation | Commande  |
|--------|-----------|
| **Windows** | `set AWS_ACCESS_KEY_ID=<access-key>`<br>`set AWS_SECRET_ACCESS_KEY=<secret-access-key>` |
| **Linux** | `export AWS_ACCESS_KEY_ID=<access-key>`<br>`export AWS_SECRET_ACCESS_KEY=<secret-access-key>` |
| **macOS** | `export AWS_ACCESS_KEY_ID=<access-key>`<br>`export AWS_SECRET_ACCESS_KEY=<secret-access-key>`|

## <a name="copy-objects-directories-and-buckets"></a>Copier des objets, des répertoires et des compartiments

AzCopy utilise l’API [Placer un bloc à partir d’une URL](/rest/api/storageservices/put-block-from-url) pour copier les données directement d’un serveur AWS S3 à un serveur de stockage. Ces opérations de copie n’utilisent pas la bande passante réseau de votre ordinateur.

> [!TIP]
> Dans les exemples de cette section, les arguments de chemin d’accès sont entre guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

 Ces exemples fonctionnent également avec les comptes qui ont un espace de noms hiérarchique. [L’accès multiprotocole sur Data Lake Storage](../blobs/data-lake-storage-multi-protocol-access.md) vous permet d’utiliser la même syntaxe d’URL (`blob.core.windows.net`) sur ces comptes. 

### <a name="copy-an-object"></a>Copier un objet

Utilisez la même syntaxe d’URL (`blob.core.windows.net`) pour les comptes qui ont un espace de noms hiérarchique.

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://s3.amazonaws.com/<bucket-name>/<object-name>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<blob-name>'` |
| **Exemple** | `azcopy copy 'https://s3.amazonaws.com/mybucket/myobject' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myblob'` |
| **Exemple** (espace de noms hiérarchique) | `azcopy copy 'https://s3.amazonaws.com/mybucket/myobject' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myblob'` |

> [!NOTE]
> Les exemples de cet article utilisent des URL de type chemin d’accès pour des compartiments AWS S3 (par exemple, `http://s3.amazonaws.com/<bucket-name>`). 
>
> Vous pouvez également utiliser des URL hébergées virtuelles (par exemple, `http://bucket.s3.amazonaws.com`). 
>
> Pour en savoir plus sur l’hébergement virtuel de compartiments, consultez [Hébergement virtuel de compartiments]](https://docs.aws.amazon.com/AmazonS3/latest/dev/VirtualHosting.html).

### <a name="copy-a-directory"></a>Copier un répertoire

Utilisez la même syntaxe d’URL (`blob.core.windows.net`) pour les comptes qui ont un espace de noms hiérarchique.

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://s3.amazonaws.com/<bucket-name>/<directory-name>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' --recursive=true` |
| **Exemple** | `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |
| **Exemple** (espace de noms hiérarchique)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

> [!NOTE]
> Cet exemple ajoute l’indicateur `--recursive` pour copier des fichiers dans tous les sous-répertoires.

### <a name="copy-the-contents-of-a-directory"></a>Copier le contenu d’un répertoire

Vous pouvez copier le contenu d’un répertoire sans copier le répertoire proprement dit en utilisant le caractère générique (*).

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://s3.amazonaws.com/<bucket-name>/<directory-name>/*' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' --recursive=true` |
| **Exemple** | `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory/*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |
| **Exemple** (espace de noms hiérarchique)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory/*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

### <a name="copy-a-bucket"></a>Copier un compartiment

Utilisez la même syntaxe d’URL (`blob.core.windows.net`) pour les comptes qui ont un espace de noms hiérarchique.

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://s3.amazonaws.com/<bucket-name>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>' --recursive=true` |
| **Exemple** | `azcopy copy 'https://s3.amazonaws.com/mybucket' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive=true` |
| **Exemple** (espace de noms hiérarchique)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

### <a name="copy-all-buckets-in-all-regions"></a>Copier tous les compartiments dans toutes les régions

Utilisez la même syntaxe d’URL (`blob.core.windows.net`) pour les comptes qui ont un espace de noms hiérarchique.

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://s3.amazonaws.com/' 'https://<storage-account-name>.blob.core.windows.net' --recursive=true` |
| **Exemple** | `azcopy copy 'https://s3.amazonaws.com' 'https://mystorageaccount.blob.core.windows.net' --recursive=true` |
| **Exemple** (espace de noms hiérarchique)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

### <a name="copy-all-buckets-in-a-specific-s3-region"></a>Copier tous les compartiments dans une région S3 spécifique

Utilisez la même syntaxe d’URL (`blob.core.windows.net`) pour les comptes qui ont un espace de noms hiérarchique.

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy copy 'https://s3-<region-name>.amazonaws.com/' 'https://<storage-account-name>.blob.core.windows.net' --recursive=true` |
| **Exemple** | `azcopy copy 'https://s3-rds.eu-north-1.amazonaws.com' 'https://mystorageaccount.blob.core.windows.net' --recursive=true` |
| **Exemple** (espace de noms hiérarchique)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

## <a name="handle-differences-in-object-naming-rules"></a>Gérer les différences dans les règles d’attributions de noms aux objets

AWS S3 possède un ensemble de conventions d’attribution de noms aux compartiments différent des conteneurs d’objets blob Azure. Vous pouvez les découvrir [ici](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules). Si vous choisissez de copier un groupe de compartiments dans un compte de stockage Azure, l’opération de copie peut échouer en raison des différences en matière d’attributions de noms.

AzCopy gère deux des problèmes les plus communs : les compartiments contenant des points et ceux contenant plusieurs traits d’union à la suite. Les noms de compartiment AWS S3 peuvent contenir des points et des traits d’union consécutifs, ce qui n’est pas le cas d’un conteneur dans Azure. AzCopy remplace les points par des traits d’union et les traits d’union consécutifs par un nombre qui représente le nombre de traits d’union consécutifs (par exemple, un compartiment nommé `my----bucket` devient `my-4-bucket`. 

En outre, comme AzCopy copie les fichiers, il vérifie les collisions de noms et tente de les résoudre. Par exemple, s’il y a des compartiments nommés `bucket-name` et `bucket.name`, AzCopy résout d’abord un compartiment nommé `bucket.name` en `bucket-name` puis en `bucket-name-2`.

## <a name="handle-differences-in-object-metadata"></a>Gérer les différences dans les métadonnées d’objets

AWS S3 et Azure autorisent différents jeux de caractères dans les noms des clés d’objet. Vous pouvez en apprendre plus sur les caractères qu’utilise AWS S3 [ici](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingMetadata.html#object-keys). Du côté d’Azure, les clés d’objets blob respectent les règles d’attribution de noms pour les [identificateurs C#](/dotnet/csharp/language-reference/).

Dans le cadre d’une commande `copy` AzCopy, vous pouvez fournir une valeur pour l’indicateur `s2s-handle-invalid-metadata` facultatif qui spécifie comment vous voulez gérer les métadonnées du fichier qui contient les noms de clés incompatibles. La table suivante décrit chaque valeur d’indicateur.

| Valeur d’indicateur | Description  |
|--------|-----------|
| **ExcludeIfInvalid** | (Option par défaut) Les métadonnées ne sont pas incluses dans l’objet transféré. AzCopy enregistre un avertissement. |
| **FailIfInvalid** | Les objets ne sont pas copiés. AzCopy enregistre une erreur et l’inclut dans le nombre d’échecs qui apparaît dans le résumé de transfert.  |
| **RenameIfInvalid**  | AzCopy résout la clé de métadonnées non valides et copie l’objet dans Azure à l’aide de la paire valeur/clé des métadonnées résolues. Pour connaître précisément les étapes suivies par AzCopy pour renommer les clés d’objet, consultez la section [Comment AzCopy renomme les clés d’objet](#rename-logic) ci-dessous. Si AzCopy n’est pas en mesure de renommer la clé, l’objet n’est pas copié. |

<a id="rename-logic"></a>

### <a name="how-azcopy-renames-object-keys"></a>Comment AzCopy renomme les clés d’objet

AzCopy suit cette procédure :

1. Remplace les caractères non valides par « _ ».

2. Ajoute la chaîne `rename_` au début d’une nouvelle clé valide.

   Cette clé doit être utilisée pour enregistrer la **valeur** d’origine des métadonnées.

3. Ajoute la chaîne `rename_key_` au début d’une nouvelle clé valide.
   Cette clé doit être utilisée pour enregistrer la **clé** invalide d’origine des métadonnées.
   Vous pouvez utiliser cette clé pour essayer de récupérer les métadonnées dans Azure, puisque la clé des métadonnées est conservée comme valeur dans le service de stockage d’objets blob.

## <a name="next-steps"></a>Étapes suivantes

Plus d’exemples dans ces articles :

- [Bien démarrer avec AzCopy](storage-use-azcopy-v10.md)

- [Transfert de données](storage-use-azcopy-v10.md#transfer-data)

- [Transférer des données avec AzCopy et le stockage de fichiers](storage-use-azcopy-files.md)

- [Configurer, optimiser et dépanner AzCopy](storage-use-azcopy-configure.md)
