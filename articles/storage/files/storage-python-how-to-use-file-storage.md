---
title: Développement pour Azure Files avec Python | Microsoft Docs
description: Découvrez comment développer des services et applications Python qui utilisent Azure Files pour stocker les données de fichiers.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 10/08/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-python
ms.openlocfilehash: 8bef69037fad8bf8ee9537e90f26ca967560b9d2
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91876095"
---
# <a name="develop-for-azure-files-with-python"></a>Développement pour Azure Files avec Python

[!INCLUDE [storage-selector-file-include](../../../includes/storage-selector-file-include.md)]

Découvrez les notions de base pour développer en Python des applications ou des services qui stockent les données de fichiers avec Azure Files. Créez une application de console simple et apprenez à effectuer des actions de base avec Python et Azure Files :

- Créer des partages de fichiers Azure
- Créer des répertoires
- Énumérer des fichiers et répertoires dans un partage de fichiers Azure
- Charger, télécharger et supprimer un fichier
- Créer des sauvegardes de partages de fichiers à l’aide d’instantanés

> [!NOTE]
> Comme Azure Files est accessible sur SMB, vous pouvez écrire des applications simples qui accèdent au partage de fichiers Azure à l’aide des fonctions et classes d’E/S Python standard. Cet article explique comment écrire des applications qui utilisent le kit SDK Python du Stockage Fichier Azure, lequel s’appuie sur [l’API REST Azure Files](/rest/api/storageservices/file-service-rest-api) pour communiquer avec Azure Files.

## <a name="download-and-install-azure-storage-sdk-for-python"></a>Télécharger et installer le SDK Stockage Azure pour Python

> [!NOTE]
> Si vous mettez à niveau la version 0.36 ou antérieure du SDK Stockage Azure pour Python, désinstallez l’ancien SDK à l’aide de `pip uninstall azure-storage` avant d’installer le dernier package.

# <a name="python-v12"></a>[Python v12](#tab/python)

La version 2.7 ou 3.5 de Python est nécessaire pour la [bibliothèque de client de Stockage Fichier Azure v12.x pour Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-share).

# <a name="python-v2"></a>[Python v2](#tab/python2)

Le [SDK Stockage Azure pour Python](https://github.com/azure/azure-storage-python) nécessite Python 2.7, 3.3, 3.4, 3.5 ou 3.6.

---

## <a name="install-via-pypi"></a>Effectuer l'installation via PyPi

Pour effectuer l’installation via PyPI (Python Package Index), tapez :

# <a name="python-v12"></a>[Python v12](#tab/python)

```console
pip install azure-storage-file-share
```

# <a name="python-v2"></a>[Python v2](#tab/python2)

```console
pip install azure-storage-file
```

### <a name="view-the-sample-application"></a>Voir l’exemple d’application

Pour voir et exécuter un exemple d’application qui montre comment utiliser Python avec Azure Files, consultez [Azure Storage: Getting Started with Azure Files in Python](https://github.com/Azure-Samples/storage-file-python-getting-started).

Pour exécuter l’exemple d’application, veillez à ce que les packages `azure-storage-file` et `azure-storage-common` soient installés.

---

## <a name="set-up-your-application-to-use-azure-files"></a>Configurer votre application pour utiliser Azure Files

Ajoutez le code suivant au début d’un fichier source Python pour pouvoir utiliser les extraits de code de cet article.

# <a name="python-v12"></a>[Python v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_Imports":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

```python
from azure.storage.file import FileService
```

---

## <a name="set-up-a-connection-to-azure-files"></a>Configurer une connexion à Azure Files

# <a name="python-v12"></a>[Python v12](#tab/python)

[ShareServiceClient](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.shareserviceclient) permet d’utiliser des partages, des répertoires et des fichiers. Le code suivant crée un objet `ShareServiceClient` à l’aide de la chaîne de connexion du compte de stockage.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_CreateShareServiceClient":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

L’objet [FileService](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true) permet d’utiliser des partages, des répertoires et des fichiers. Le code suivant crée un objet `FileService` à l’aide du nom et de la clé du compte de stockage. Remplacez `<myaccount>` et `<mykey>` par le nom et la clé de votre compte.

```python
file_service = FileService(account_name='myaccount', account_key='mykey')
```

---

## <a name="create-an-azure-file-share"></a>Crée un partage de fichiers Azure

# <a name="python-v12"></a>[Python v12](#tab/python)

L’exemple de code suivant utilise un objet [ShareClient](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.shareclient) pour créer le partage s’il n’existe pas.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_CreateFileShare":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

L’exemple de code suivant utilise un objet [FileService](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true) pour créer le partage s’il n’existe pas.

```python
file_service.create_share('myshare')
```

---

## <a name="create-a-directory"></a>Créer un répertoire

Vous pouvez organiser le stockage en plaçant des fichiers dans des sous-répertoires, plutôt que de tous les mettre dans le répertoire racine.

# <a name="python-v12"></a>[Python v12](#tab/python)

La méthode suivante permet de créer un répertoire à la racine du partage de fichiers spécifié à l’aide d’un objet [ShareDirectoryClient](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.sharedirectoryclient).

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_CreateDirectory":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Le code ci-dessous crée un sous-répertoire nommé *sampledir* sous le répertoire racine.

```python
file_service.create_directory('myshare', 'sampledir')
```

---

## <a name="upload-a-file"></a>Charger un fichier

Cette section explique comment charger un fichier du stockage local vers le Stockage Fichier Azure.

# <a name="python-v12"></a>[Python v12](#tab/python)

La méthode suivante permet de charger le contenu du fichier spécifié dans le répertoire indiqué au sein du partage de fichiers Azure prévu.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_UploadFile":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Un partage de fichiers Azure contient au minimum un répertoire racine dans lequel se trouvent les fichiers. Pour créer un fichier et charger des données, utilisez la méthode [create_file_from_path](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#create-file-from-path-share-name--directory-name--file-name--local-file-path--content-settings-none--metadata-none--validate-content-false--progress-callback-none--max-connections-2--file-permission-none--smb-properties--azure-storage-file-models-smbproperties-object---timeout-none-), la méthode [create_file_from_stream](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#create-file-from-stream-share-name--directory-name--file-name--stream--count--content-settings-none--metadata-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--file-permission-none--smb-properties--azure-storage-file-models-smbproperties-object--), [create_file_from_bytes](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#create-file-from-bytes-share-name--directory-name--file-name--file--index-0--count-none--content-settings-none--metadata-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--file-permission-none--smb-properties--azure-storage-file-models-smbproperties-object--) ou la méthode [create_file_from_text](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#create-file-from-text-share-name--directory-name--file-name--text--encoding--utf-8---content-settings-none--metadata-none--validate-content-false--timeout-none--file-permission-none--smb-properties--azure-storage-file-models-smbproperties-object--). Il s’agit de méthodes de haut niveau qui effectuent la segmentation nécessaire lorsque la taille des données est supérieure à 64 Mo.

`create_file_from_path` charge le contenu d’un fichier à partir du chemin spécifié, et `create_file_from_stream` charge le contenu à partir d’un flux/fichier déjà ouvert. `create_file_from_bytes` charge un tableau d’octets, et `create_file_from_text` charge la valeur texte spécifiée à l’aide de l’encodage spécifié (par défaut UTF-8).

L’exemple suivant charge le contenu du fichier *sunset.png* dans le fichier **myfile**.

```python
from azure.storage.file import ContentSettings
file_service.create_file_from_path(
    'myshare',
    None,  # We want to create this file in the root directory, so we specify None for the directory_name
    'myfile',
    'sunset.png',
    content_settings=ContentSettings(content_type='image/png'))
```

---

## <a name="enumerate-files-and-directories-in-an-azure-file-share"></a>Énumérer des fichiers et répertoires dans un partage de fichiers Azure

# <a name="python-v12"></a>[Python v12](#tab/python)

Pour lister les fichiers et les répertoires d’un sous-répertoire, utilisez la méthode [list_directories_and_files](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.shareclient#list-directories-and-files-directory-name-none--name-starts-with-none--marker-none----kwargs-). Cette méthode retourne un itérable à pagination automatique. Le code suivant donne en sortie dans la console le **nom** de chacun des fichiers et répertoires du répertoire spécifié.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_ListFilesAndDirs":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Pour lister les fichiers et répertoires d’un partage, utilisez la méthode [list_directories_and_files](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#list-directories-and-files-share-name--directory-name-none--num-results-none--marker-none--timeout-none--prefix-none--snapshot-none-). Cette méthode retourne un générateur. Le code suivant sort le **nom** de chaque fichier et répertoire d'un partage sur la console.

```python
generator = file_service.list_directories_and_files('myshare')
for file_or_dir in generator:
    print(file_or_dir.name)
```

---

## <a name="download-a-file"></a>Télécharger un fichier

# <a name="python-v12"></a>[Python v12](#tab/python)

Pour télécharger des données à partir d’un fichier, utilisez [download_file](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.sharefileclient#download-file-offset-none--length-none----kwargs-).

L’exemple suivant montre comment utiliser `download_file` pour récupérer le contenu du fichier spécifié et le stocker localement en ajoutant le préfixe **DOWNLOADED-** au nom de fichier.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_DownloadFile":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Pour télécharger des données à partir d’un fichier, utilisez [get_file_to_path](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#get-file-to-path-share-name--directory-name--file-name--file-path--open-mode--wb---start-range-none--end-range-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--snapshot-none-), [get_file_to_stream](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#get-file-to-stream-share-name--directory-name--file-name--stream--start-range-none--end-range-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--snapshot-none-), [get_file_to_bytes](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#get-file-to-bytes-share-name--directory-name--file-name--start-range-none--end-range-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--snapshot-none-) ou [get_file_to_text](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#get-file-to-text-share-name--directory-name--file-name--encoding--utf-8---start-range-none--end-range-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--snapshot-none-). Il s’agit de méthodes de haut niveau qui effectuent la segmentation nécessaire lorsque la taille des données est supérieure à 64 Mo.

L’exemple suivant illustre l’utilisation de `get_file_to_path` pour télécharger le contenu du fichier **myfile** et le stocker dans le fichier *out-sunset.png*.

```python
file_service.get_file_to_path('myshare', None, 'myfile', 'out-sunset.png')
```

---

## <a name="create-a-share-snapshot"></a>Créer un instantané de partage

Vous pouvez créer une copie à un point dans le temps de la totalité de votre partage de fichier.

# <a name="python-v12"></a>[Python v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_CreateSnapshot":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

```python
snapshot = file_service.snapshot_share(share_name)
snapshot_id = snapshot.snapshot
```

**Créer un instantané de partage avec des métadonnées**

```python
metadata = {"foo": "bar"}
snapshot = file_service.snapshot_share(share_name, metadata=metadata)
```

---

## <a name="list-shares-and-snapshots"></a>Répertorier les partages et les instantanés

Vous pouvez répertorier tous les instantanés pour un partage particulier.

# <a name="python-v12"></a>[Python v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_ListSharesAndSnapshots":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

```python
shares = list(file_service.list_shares(include_snapshots=True))
```

---

## <a name="browse-share-snapshot"></a>Parcourir un instantané de partage

Vous pouvez parcourir chaque instantané de partage pour récupérer les fichiers et les répertoires du point dans le temps correspondant.

# <a name="python-v12"></a>[Python v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_BrowseSnapshotDir":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

```python
directories_and_files = list(
    file_service.list_directories_and_files(share_name, snapshot=snapshot_id))
```

---

## <a name="get-file-from-share-snapshot"></a>Récupérer un fichier d’un instantané de partage

Vous pouvez télécharger un fichier à partir d’un instantané de partage, pour pouvoir restaurer une version précédente d’un fichier.

# <a name="python-v12"></a>[Python v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_DownloadSnapshotFile":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

```python
with open(FILE_PATH, 'wb') as stream:
    file = file_service.get_file_to_stream(
        share_name, directory_name, file_name, stream, snapshot=snapshot_id)
```

---

## <a name="delete-a-single-share-snapshot"></a>Supprimer un instantané de partage spécifique
Vous pouvez supprimer un instantané de partage spécifique.

# <a name="python-v12"></a>[Python v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_DeleteSnapshot":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

```python
file_service.delete_share(share_name, snapshot=snapshot_id)
```

---

## <a name="delete-a-file"></a>Supprimer un fichier

# <a name="python-v12"></a>[Python v12](#tab/python)

Pour supprimer un fichier, appelez [delete_file](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.sharefileclient#delete-file---kwargs-).

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_DeleteFile":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Pour supprimer un fichier, appelez [delete_file](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#delete-file-share-name--directory-name--file-name--timeout-none-).

```python
file_service.delete_file('myshare', None, 'myfile')
```

---

## <a name="delete-share-when-share-snapshots-exist"></a>Supprimer un partage quand existent des instantanés de partage

# <a name="python-v12"></a>[Python v12](#tab/python)

Pour supprimer un partage contenant des instantanés, appelez [delete_share](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.shareclient#delete-share-delete-snapshots-false----kwargs-) avec `delete_snapshots=True`.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_DeleteShare":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Pour supprimer un partage qui a des instantanés, vous devez d’abord supprimer la totalité de ces derniers.

```python
file_service.delete_share(share_name, delete_snapshots=DeleteSnapshot.Include)
```

---

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris comment manipuler Azure Files avec Python, suivez ces liens pour en savoir plus.

- [Centre de développement Python](/azure/developer/python/)
- [API REST des services d’Azure Storage](/rest/api/azure/)
- [Kit de développement logiciel (SDK) Microsoft Azure Storage pour Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage)
