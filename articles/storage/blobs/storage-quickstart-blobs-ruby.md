---
title: 'Démarrage rapide : Bibliothèque de client de Stockage Blob Azure – Ruby'
description: Créez un compte de stockage et un conteneur dans Stockage Blob Azure. Utilisez la bibliothèque de client de stockage pour que Ruby crée un objet blob, télécharge un objet blob et liste les blobs dans un conteneur.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 12/04/2020
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.openlocfilehash: ec3fc490466f5fce36b67b2f3744e4ee5cc0ae79
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96781094"
---
# <a name="quickstart-azure-blob-storage-client-library-for-ruby"></a>Démarrage rapide : Bibliothèque de client de Stockage Blob Azure pour Ruby

Apprenez à utiliser Ruby pour créer, télécharger et lister des blobs dans un conteneur du Stockage Blob Azure de Microsoft.

## <a name="prerequisites"></a>Conditions préalables requises

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

Vérifiez que les prérequis suivants sont installés :

- [Ruby](https://www.ruby-lang.org/en/downloads/)
- [Bibliothèque de Stockage Azure pour Ruby](https://github.com/azure/azure-storage-ruby), en utilisant le [package RubyGem](https://rubygems.org/gems/azure-storage-blob) :

    ```console
    gem install azure-storage-blob
    ```

## <a name="download-the-sample-application"></a>Téléchargement de l'exemple d'application

L’[exemple d’application](https://github.com/Azure-Samples/storage-blobs-ruby-quickstart.git) utilisé dans ce démarrage rapide est une application Ruby de base.

Utilisez [Git](https://git-scm.com/) pour télécharger une copie de l’application dans votre environnement de développement. Cette commande clone le dépôt dans votre machine locale :

```console
git clone https://github.com/Azure-Samples/storage-blobs-ruby-quickstart.git
```

Accédez au dossier *storage-blobs-ruby-quickstart*, puis ouvrez le fichier *example.rb* dans votre éditeur de code.

[!INCLUDE [storage-copy-account-key-portal](../../../includes/storage-copy-account-key-portal.md)]

## <a name="configure-your-storage-connection-string"></a>Configurer votre chaîne de connexion de stockage

Fournissez le nom de votre compte de stockage et votre clé de compte pour créer une instance [BlobService](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob/BlobService) associée à votre application.

Le code suivant dans le fichier *example.rb* instancie un nouvel objet [BlobService](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob/BlobService). Remplacez les valeurs *accountname* et *accountkey* par votre nom de compte et votre clé de compte.

```ruby
# Create a BlobService object
account_name = "accountname"
account_key = "accountkey"

blob_client = Azure::Storage::Blob::BlobService.create(
    storage_account_name: account_name,
    storage_access_key: account_key
)
```

## <a name="run-the-sample"></a>Exécution de l'exemple

L’exemple crée un conteneur dans le Stockage Blob, il crée un blob dans le conteneur, liste les blobs dans le conteneur et télécharge le blob dans un fichier local.

Exécutez l’exemple. Voici un exemple de la sortie générée par l’exécution de l’application :

```console
C:\azure-samples\storage-blobs-ruby-quickstart> ruby example.rb

Creating a container: quickstartblobs18cd9ec0-f4ac-4688-a979-75c31a70503e

Creating blob: QuickStart_6f8f29a8-879a-41fb-9db2-0b8595180728.txt

List blobs in the container following continuation token
        Blob name: QuickStart_6f8f29a8-879a-41fb-9db2-0b8595180728.txt

Downloading blob to C:/Users/azureuser/Documents/QuickStart_6f8f29a8-879a-41fb-9db2-0b8595180728.txt

Paused, press the Enter key to delete resources created by the sample and exit the application
```

Quand vous appuyez sur Entrée pour continuer, l’exemple de programme supprime le conteneur de stockage et le fichier local. Avant de continuer, recherchez le fichier téléchargé dans votre dossier *Documents*.

Vous pouvez également utiliser l’[Explorateur Stockage Azure](https://storageexplorer.com) pour afficher les fichiers dans votre compte de stockage. L’Explorateur Stockage Azure est un outil multiplateforme gratuit qui vous permet d’accéder aux informations de votre compte de stockage.

Une fois les fichiers vérifiés, appuyez sur la touche Entrée pour supprimer les fichiers de test et mettre fin à la démonstration. Ouvrez le fichier *example.rb* pour examiner le code.

## <a name="understand-the-sample-code"></a>Comprendre l’exemple de code

Ensuite, nous allons parcourir l’exemple de code pas à pas pour vous montrer son fonctionnement.

### <a name="get-references-to-the-storage-objects"></a>Obtenir des références aux objets de stockage

La première chose à faire est de créer les instances des objets utilisés pour accéder et gérer le Stockage Blob. Ces objets reposent les uns sur les autres. Chacun est utilisé par le suivant dans la liste.

- Créez une instance de l’objet [BlobService](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob/BlobService) Stockage Azure pour configurer les informations d’identification de connexion.
- Créez l’objet [Container](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob/Container/Container), qui représente le conteneur auquel vous accédez. Les conteneurs sont utilisés pour organiser vos objets blob de la même façon que vous utilisez des dossiers pour organiser vos fichiers sur votre ordinateur.

Une fois que vous avez l’objet conteneur, vous pouvez créer un blob [Block](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob/Block) qui pointe vers un blob particulier qui vous intéresse. Utilisez l’objet [Block](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob/Block) pour créer, télécharger et copier des blobs.

> [!IMPORTANT]
> Les noms de conteneurs doivent être en minuscules. Pour plus d’informations sur les noms des conteneurs et des objets blob, consultez [Affectation de noms et références aux conteneurs, objets blob et métadonnées](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata).

L’exemple de code suivant :

- Crée un conteneur
- Définit des autorisations sur le conteneur pour que les blobs soient publics. Le conteneur est appelé *quickstartblobs*, nom auquel est ajouté un ID unique.

```ruby
# Create a container
container_name = "quickstartblobs" + SecureRandom.uuid
puts "\nCreating a container: " + container_name
container = blob_client.create_container(container_name)

# Set the permission so the blobs are public
blob_client.set_container_acl(container_name, "container")
```

### <a name="create-a-blob-in-the-container"></a>Créer un blob dans le conteneur

Le Stockage Blob prend en charge les objets blob de blocs, d’ajouts et de pages. Pour créer un blob, appelez la méthode [create_block_blob](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob#create_block_blob-instance_method) en passant les données du blob.

L’exemple suivant crée un blob appelé *QuickStart_* avec un ID unique et une extension de fichier *.txt* dans le conteneur créé précédemment.

```ruby
# Create a new block blob containing 'Hello, World!'
blob_name = "QuickStart_" + SecureRandom.uuid + ".txt"
blob_data = "Hello, World!"
puts "\nCreating blob: " + blob_name
blob_client.create_block_blob(container.name, blob_name, blob_data)
```

Les objets blob de blocs peuvent atteindre une taille maximale de 4,7 To, et représenter toutes sortes d’éléments allant des feuilles de calcul aux fichiers vidéo volumineux. Les objets blob de pages sont principalement utilisés pour les fichiers VHD qui viennent en soutien aux machines virtuelles IaaS. Les objets blob d’ajouts sont couramment utilisés pour la journalisation, par exemple quand vous voulez écrire dans un fichier et continuer à ajouter d’autres informations.

### <a name="list-the-blobs-in-a-container"></a>Lister les objets blob dans un conteneur

Vous pouvez obtenir la liste des fichiers du conteneur à l’aide de la méthode [list_blobs](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob/Container#list_blobs-instance_method). Le code suivant récupère la liste des blobs, puis affiche leur nom.

```ruby
# List the blobs in the container
puts "\nList blobs in the container following continuation token"
nextMarker = nil
loop do
    blobs = blob_client.list_blobs(container_name, { marker: nextMarker })
    blobs.each do |blob|
        puts "\tBlob name: #{blob.name}"
    end
    nextMarker = blobs.continuation_token
    break unless nextMarker && !nextMarker.empty?
end
```

### <a name="download-a-blob"></a>Télécharger un objet blob

Téléchargez un blob sur votre disque local à l’aide de la méthode [get_blob](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob#get_blob-instance_method). Le code suivant télécharge le blob créé dans une section précédente.

```ruby
# Download the blob

# Set the path to the local folder for downloading
if(is_windows)
    local_path = File.expand_path("~/Documents")
else 
    local_path = File.expand_path("~/")
end

# Create the full path to the downloaded file
full_path_to_file = File.join(local_path, blob_name)

puts "\nDownloading blob to " + full_path_to_file
blob, content = blob_client.get_blob(container_name, blob_name)
File.open(full_path_to_file,"wb") {|f| f.write(content)}
```

### <a name="clean-up-resources"></a>Nettoyer les ressources

Si un blob n’est plus nécessaire, utilisez [delete_blob](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob#delete_blob-instance_method) pour le supprimer. Supprimez un conteneur entier à l’aide de la méthode [delete_container](https://www.rubydoc.info/gems/azure-storage-blob/2.0.1/Azure/Storage/Blob/Container#delete_container-instance_method). La suppression d’un conteneur supprime également tous les blobs stockés dans le conteneur.

```ruby
# Clean up resources, including the container and the downloaded file
blob_client.delete_container(container_name)
File.delete(full_path_to_file)
```

## <a name="resources-for-developing-ruby-applications-with-blobs"></a>Ressources sur le développement d’applications Ruby avec des objets blob

Consultez ces ressources supplémentaires sur le développement Ruby :

- Affichez et téléchargez le [code source de la bibliothèque de client Ruby](https://github.com/Azure/azure-storage-ruby) pour Stockage Azure sur GitHub.
- Explorez les [exemples Azure](/samples/browse/?products=azure&languages=ruby) écrits avec la bibliothèque de client Ruby.
- [Exemple : Bien démarrer avec Stockage Azure en Ruby](https://github.com/Azure-Samples/storage-blob-ruby-getting-started)

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à transférer des fichiers entre le Stockage Blob Azure et un disque local avec Ruby. Pour en savoir plus sur l’utilisation de Stockage Blob, consultez la vue d’ensemble du compte de stockage.

> [!div class="nextstepaction"]
> [Vue d’ensemble du compte de stockage](../common/storage-account-overview.md)

Pour plus d’informations sur l’Explorateur Stockage et les blobs, consultez [Gérer les ressources de Stockage Blob Azure avec l’Explorateur Stockage](../../vs-azure-tools-storage-explorer-blobs.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
