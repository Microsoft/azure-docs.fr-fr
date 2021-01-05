---
title: Utilisation du stockage d’objets (blob) à partir de C++ - Azure | Microsoft Docs
description: En savoir plus sur le stockage des données non structurées (blobs) dans le cloud via le stockage d’objet blob Azure avec le C++.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 07/16/2020
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.openlocfilehash: 64069292ea0059216d06bfc41316c2aed7484dd0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011096"
---
# <a name="how-to-use-blob-storage-from-c"></a>Utilisation du stockage d’objets blob à partir de C++

Ce guide explique le déroulement des scénarios courants dans le cadre de l’utilisation du Stockage Blob Azure. Les exemples illustrent comment charger, lister, télécharger et supprimer des objets blob. Les exemples ont été écrits en C++ et utilisent la [bibliothèque cliente Azure Storage pour C++](https://github.com/Azure/azure-storage-cpp/blob/master/README.md).
Pour en savoir plus sur le Stockage Blob, consultez [Présentation du Stockage Blob Azure](storage-blobs-introduction.md).

> [!NOTE]
> Ce guide cible la bibliothèque cliente Azure Storage pour C++ version 1.0.0 et les versions ultérieures. Microsoft recommande d’utiliser la dernière version de la bibliothèque cliente pour C++, disponible via [NuGet](https://www.nuget.org/packages/wastorage) ou [GitHub](https://github.com/Azure/azure-storage-cpp).

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-c-application"></a>Création d’une application C++
Dans ce guide, vous allez utiliser des fonctionnalités de stockage qui peuvent être exécutées dans une application C++.

Pour ce faire, vous devez installer la bibliothèque cliente Azure Storage pour C++ et créer un compte Azure Storage dans votre abonnement Azure.

Pour installer la bibliothèque cliente Azure Storage pour C++, vous pouvez procéder comme suit :

- **Linux :** Suivez les instructions disponibles dans la page [Bibliothèque de client du stockage Azure pour C++ README : Prise en main de la page Linux](https://github.com/Azure/azure-storage-cpp#getting-started-on-linux).
- **Windows :** Sous Windows, utilisez [vcpkg](https://github.com/microsoft/vcpkg) en tant que manager de dépendances. Suivez le [démarrage rapide](https://github.com/microsoft/vcpkg#quick-start) pour initialiser vcpkg. Puis, pour installer la bibliothèque, utilisez la commande suivante :

```powershell
.\vcpkg.exe install azure-storage-cpp
```

Un guide sur la façon de générer le code source et de l’exporter vers NuGet est disponible dans le fichier [LISEZ-MOI](https://github.com/Azure/azure-storage-cpp#download--install).

## <a name="configure-your-application-to-access-blob-storage"></a>Configurer votre application pour accéder au stockage d’objets blob
Ajoutez l'instruction include suivante au début du fichier C++ dans lequel vous voulez utiliser des API de stockage Azure pour accéder aux objets blob :

```cpp
#include <was/storage_account.h>
#include <was/blob.h>
#include <cpprest/filestream.h>  
#include <cpprest/containerstream.h> 
```

## <a name="setup-an-azure-storage-connection-string"></a>Configuration d’une chaîne de connexion de stockage Azure
Un client de stockage Azure utilise une chaîne de connexion de stockage pour stocker des points de terminaison et des informations d’identification permettant d’accéder aux services de gestion des données. Lors de l’exécution d’une application cliente, vous devez spécifier la chaîne de connexion au stockage au format suivant, en indiquant le nom de votre compte de stockage et sa clé d’accès de stockage, correspondant aux valeurs *AccountName* et *AccountKey*, sur le [portail Azure](https://portal.azure.com). Pour plus d'informations sur les comptes et les clés d'accès de stockage, consultez la page [À propos des comptes Azure Storage](../common/storage-account-create.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json). Cet exemple vous montre comment déclarer un champ statique pour qu’il contienne une chaîne de connexion :

```cpp
// Define the connection-string with your values.
const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=your_storage_account;AccountKey=your_storage_account_key"));
```

Pour tester votre application sur votre ordinateur Windows local, vous pouvez utiliser [l’émulateur de stockage Azurite](../common/storage-use-azurite.md). Azurite est un utilitaire qui simule sur votre ordinateur de développement local les services de blob et de file d’attente disponibles dans Azure. L’exemple suivant vous montre comment déclarer un champ statique pour qu’il contienne une chaîne de connexion vers votre émulateur de stockage local :

```cpp
// Define the connection-string with Azurite.
const utility::string_t storage_connection_string(U("UseDevelopmentStorage=true;"));
```

Pour démarrer Azurite, consultez [Utiliser l’émulateur Azurite à des fins de développement local pour Stockage Azure](../common/storage-use-azurite.md).

Les exemples ci-dessous partent du principe que vous avez utilisé l’une de ces deux méthodes pour obtenir la chaîne de connexion de stockage.

## <a name="retrieve-your-storage-account"></a>Récupérer votre compte de stockage
Vous pouvez utiliser la classe **cloud_storage_account** pour représenter les informations de votre compte de stockage. Pour extraire les informations de votre compte de stockage de la chaîne de connexion de stockage, vous pouvez utiliser la méthode **parse** .

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);
```

Ensuite, récupérez une référence pointant vers une classe **cloud_blob_client**, car elle permet de récupérer des objets représentant des conteneurs et des objets blob stockés dans le stockage d’objets blob. Le code suivant crée un objet **cloud_blob_client** en utilisant l’objet de compte de stockage récupéré ci-dessus :

```cpp
// Create the blob client.
azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();  
```

## <a name="how-to-create-a-container"></a>Procédure : Créez un conteneur.
[!INCLUDE [storage-container-naming-rules-include](../../../includes/storage-container-naming-rules-include.md)]

Cet exemple montre comment créer un conteneur, si celui-ci n’existe pas encore :

```cpp
try
{
    // Retrieve storage account from connection string.
    azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

    // Create the blob client.
    azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

    // Retrieve a reference to a container.
    azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

    // Create the container if it doesn't already exist.
    container.create_if_not_exists();
}
catch (const std::exception& e)
{
    std::wcout << U("Error: ") << e.what() << std::endl;
}  
```

Le nouveau conteneur est privé par défaut et vous devez indiquer votre clé d'accès de stockage pour télécharger des objets blob depuis ce conteneur. Si vous voulez que les fichiers (objets blob) du conteneur soient publics, vous pouvez configurer le conteneur en utilisant le code suivant :

```cpp
// Make the blob container publicly accessible.
azure::storage::blob_container_permissions permissions;
permissions.set_public_access(azure::storage::blob_container_public_access_type::blob);
container.upload_permissions(permissions);  
```

Tous les utilisateurs d’Internet peuvent afficher les objets blob d’un conteneur public, mais seuls ceux possédant la clé d’accès adéquate peuvent les modifier ou les supprimer.

## <a name="how-to-upload-a-blob-into-a-container"></a>Procédure : Charger un objet blob dans un conteneur
Le service de stockage d’objets blob Azure prend en charge les objets blob de blocs et de pages. Dans la plupart des cas, il est recommandé d’utiliser le type d’objet blob de blocs.

Pour télécharger un fichier vers un objet blob de blocs, obtenez une référence de conteneur et utilisez-la pour obtenir une référence d’objet blob de blocs. Lorsque vous disposez d’une référence d’objet blob, vous pouvez télécharger un flux de données vers cet objet en appelant la méthode **upload_from_stream**. Si l’objet blob n’existe pas, cette opération entraîne sa création. S’il existe, il est remplacé. L’exemple suivant illustre le chargement d’un objet blob dans un conteneur en partant du principe que le conteneur existe déjà.

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the blob client.
azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

// Retrieve a reference to a previously created container.
azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

// Retrieve reference to a blob named "my-blob-1".
azure::storage::cloud_block_blob blockBlob = container.get_block_blob_reference(U("my-blob-1"));

// Create or overwrite the "my-blob-1" blob with contents from a local file.
concurrency::streams::istream input_stream = concurrency::streams::file_stream<uint8_t>::open_istream(U("DataFile.txt")).get();
blockBlob.upload_from_stream(input_stream);
input_stream.close().wait();

// Create or overwrite the "my-blob-2" and "my-blob-3" blobs with contents from text.
// Retrieve a reference to a blob named "my-blob-2".
azure::storage::cloud_block_blob blob2 = container.get_block_blob_reference(U("my-blob-2"));
blob2.upload_text(U("more text"));

// Retrieve a reference to a blob named "my-blob-3".
azure::storage::cloud_block_blob blob3 = container.get_block_blob_reference(U("my-directory/my-sub-directory/my-blob-3"));
blob3.upload_text(U("other text"));  
```

Vous pouvez également utiliser la méthode **upload_from_file** pour télécharger un fichier vers un objet blob de blocs.

## <a name="how-to-list-the-blobs-in-a-container"></a>Procédure : Créer la liste des objets blob d’un conteneur
Pour créer une liste d’objets blob dans un conteneur, commencez par obtenir une référence pointant vers un conteneur. Vous pouvez ensuite utiliser la méthode **list_blobs** du conteneur pour récupérer les objets blob et/ou les répertoires qu’il contient. Pour accéder à l’ensemble complet des propriétés et méthodes d’un **list_blob_item** renvoyé, vous devez appeler la méthode **list_blob_item.as_blob** afin d’obtenir un objet **cloud_blob** ou la méthode **list_blob.as_directory** afin d’obtenir un objet cloud_blob_directory. Le code suivant illustre la récupération et la génération de l'URI de chaque élément du conteneur **my-sample-container** :

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the blob client.
azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

// Retrieve a reference to a previously created container.
azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

// Output URI of each item.
azure::storage::list_blob_item_iterator end_of_results;
for (auto it = container.list_blobs(); it != end_of_results; ++it)
{
    if (it->is_blob())
    {
        std::wcout << U("Blob: ") << it->as_blob().uri().primary_uri().to_string() << std::endl;
    }
    else
    {
        std::wcout << U("Directory: ") << it->as_directory().uri().primary_uri().to_string() << std::endl;
    }
}
```

Pour plus d’informations sur les opérations de listage, consultez [Listage des ressources Azure Storage en C++](../common/storage-c-plus-plus-enumeration.md).

## <a name="how-to-download-blobs"></a>Procédure : Télécharger des objets blob
Pour télécharger des objets blob, commencez par récupérer une référence d’objet blob, puis appelez la méthode **download_to_stream**. L’exemple suivant utilise la méthode **download_to_stream** pour transférer les contenus d’objets blob vers un objet de flux pouvant être rendu persistant dans un fichier local.

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the blob client.
azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

// Retrieve a reference to a previously created container.
azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

// Retrieve reference to a blob named "my-blob-1".
azure::storage::cloud_block_blob blockBlob = container.get_block_blob_reference(U("my-blob-1"));

// Save blob contents to a file.
concurrency::streams::container_buffer<std::vector<uint8_t>> buffer;
concurrency::streams::ostream output_stream(buffer);
blockBlob.download_to_stream(output_stream);

std::ofstream outfile("DownloadBlobFile.txt", std::ofstream::binary);
std::vector<unsigned char>& data = buffer.collection();

outfile.write((char *)&data[0], buffer.size());
outfile.close();  
```

Vous pouvez également utiliser la méthode **download_to_file** pour télécharger le contenu d’un objet blob dans un fichier.
De plus, vous pouvez aussi utiliser la méthode **download_text** pour télécharger le contenu d’un objet blob en tant que chaîne de texte.

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the blob client.
azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

// Retrieve a reference to a previously created container.
azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

// Retrieve reference to a blob named "my-blob-2".
azure::storage::cloud_block_blob text_blob = container.get_block_blob_reference(U("my-blob-2"));

// Download the contents of a blog as a text string.
utility::string_t text = text_blob.download_text();
```

## <a name="how-to-delete-blobs"></a>Procédure : Suppression d’objets blob
Pour supprimer un objet blob, commencez par obtenir une référence d’objet blob, puis appelez la méthode **delete_blob** associée.

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the blob client.
azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

// Retrieve a reference to a previously created container.
azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

// Retrieve reference to a blob named "my-blob-1".
azure::storage::cloud_block_blob blockBlob = container.get_block_blob_reference(U("my-blob-1"));

// Delete the blob.
blockBlob.delete_blob();
```

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous connaissez les bases du stockage d'objets blob, consultez les liens suivants pour en savoir plus sur Azure Storage.

- [Utilisation du service de stockage de files d'attente à partir de C++](../queues/storage-c-plus-plus-how-to-use-queues.md)
- [Utilisation du stockage de tables à partir de C++](../../cosmos-db/table-storage-how-to-use-c-plus.md)
- [Listage des ressources Azure Storage en C++](../common/storage-c-plus-plus-enumeration.md)
- [Référence de la bibliothèque cliente de stockage pour C++](https://azure.github.io/azure-storage-cpp)
- [Documentation d’Azure Storage](https://azure.microsoft.com/documentation/services/storage/)
- [Transfert de données avec l’utilitaire de ligne de commande AzCopy](../common/storage-use-azcopy-v10.md)