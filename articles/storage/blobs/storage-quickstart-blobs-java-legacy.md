---
title: 'Démarrage rapide : Bibliothèque cliente Stockage Blob Azure v8 pour Java'
description: Créez un compte de stockage et un conteneur dans un stockage d’objets (blob). Ensuite, utilisez la bibliothèque de client Stockage Azure v8 pour Java, afin de charger un objet blob dans Stockage Azure, de télécharger un objet blob et de lister les objets blob dans un conteneur.
author: mhopkins-msft
ms.custom: devx-track-java
ms.author: mhopkins
ms.date: 07/24/2020
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.openlocfilehash: 39b5a812c5f9b8e148b5180569af110c72e2c75b
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95544596"
---
# <a name="quickstart-manage-blobs-with-java-v8-sdk"></a>Démarrage rapide : Gérer les objets blob avec le Kit de développement logiciel (SDK) Java V8

Ce guide de démarrage rapide explique comment gérer des objets blob à l’aide de Java. Les objets blob sont des objets pouvant contenir de grandes quantités de texte ou de données binaires, notamment des images, des documents, des éléments multimédias de diffusion en continu et des données d’archive. Vous allez charger, télécharger et répertorier des objets blob. Vous allez également créer et supprimer des conteneurs, ainsi que définir des autorisations sur ceux-ci.

> [!NOTE]
> Ce guide de démarrage rapide utilise une version héritée de la bibliothèque de client Stockage Blob Azure. Pour démarrer avec la dernière version, consultez [Démarrage rapide : Gérer les blobs avec le Kit de développement logiciel (SDK) Java v12](storage-quickstart-blobs-java.md).

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- Un compte de stockage Azure. [Créer un compte de stockage](../common/storage-account-create.md).
- Un IDE avec intégration Maven. Ce guide utilise [Eclipse](https://www.eclipse.org/downloads/) avec la configuration « IDE Eclipse pour les développeurs Java ».

## <a name="download-the-sample-application"></a>Téléchargement de l'exemple d'application

L’[exemple d’application](https://github.com/Azure-Samples/storage-blobs-java-quickstart) est une application console de base.

Utilisez [git](https://git-scm.com/) pour télécharger une copie de l’application dans votre environnement de développement.

```bash
git clone https://github.com/Azure-Samples/storage-blobs-java-quickstart.git
```

Cette commande clone le dépôt dans votre dossier git local. Pour ouvrir le projet, lancez Eclipse et fermez l’écran d’accueil. Sélectionnez **Fichier**, puis **Open Projects from File System** (Ouvrir des projets à partir du système de fichiers). Vérifiez que l’option **Detect and configure project natures** (Détecter et configurer les natures de projet) est activée. Sélectionnez **Répertoire**, puis accédez à l’emplacement où vous avez stocké le référentiel cloné. Dans celui-ci, sélectionnez le dossier **blobAzureApp**. Vérifiez que le projet **blobAzureApp** s’affiche en tant que projet Eclipse, puis sélectionnez **Terminer**.

Une fois l’importation du projet accomplie, ouvrez **AzureApp.java** (situé dans **blobQuickstart.blobAzureApp** à l’intérieur de **src/main/java**) et remplacez `accountname` et `accountkey` dans la chaîne `storageConnectionString`. Puis exécutez l’application. Les instructions spécifiques pour effectuer ces tâches sont décrites dans les sections suivantes.

[!INCLUDE [storage-copy-connection-string-portal](../../../includes/storage-copy-connection-string-portal.md)]

## <a name="configure-your-storage-connection-string"></a>Configurer votre chaîne de connexion de stockage

Dans l’application, vous devez fournir la chaîne de connexion de votre compte de stockage. Ouvrez le fichier **AzureApp.Java**. Trouvez la variable `storageConnectionString` et collez la valeur de la chaîne de connexion copiée dans la section précédente. Votre variable `storageConnectionString` doit être semblable à l’exemple de code suivant :

```java
public static final String storageConnectionString =
"DefaultEndpointsProtocol=https;" +
"AccountName=<account-name>;" +
"AccountKey=<account-key>";
```

## <a name="run-the-sample"></a>Exécution de l'exemple

Cet exemple d’application crée un fichier de test dans votre répertoire par défaut (*C:\Users\<user>\AppData\Local\Temp*, for Windows users), le charge dans le stockage d’objets blob, répertorie les objets blob dans le conteneur, puis télécharge le fichier sous un nouveau nom pour vous permettre de comparer les fichiers ancien et nouveau.

Exécutez l’exemple à l’aide de Maven au niveau de la ligne de commande. Ouvrez un interpréteur de commandes et accédez à **blobAzureApp** à l’intérieur de votre répertoire cloné. Puis saisissez `mvn compile exec:java`.

L’exemple suivant illustre le résultat obtenu si vous deviez exécuter l’application sur Windows.

```
Azure Blob storage quick start sample
Creating container: quickstartcontainer
Creating a sample file at: C:\Users\<user>\AppData\Local\Temp\sampleFile514658495642546986.txt
Uploading the sample file
URI of blob is: https://myexamplesacct.blob.core.windows.net/quickstartcontainer/sampleFile514658495642546986.txt
The program has completed successfully.
Press the 'Enter' key while in the console to delete the sample files, example container, and exit the application.

Deleting the container
Deleting the source, and downloaded files
```

Avant de continuer, vérifiez votre répertoire par défaut (*C:\Users\<user>\AppData\Local\Temp*, pour les utilisateurs Windows) pour l’exemple de fichier. Copiez l’URL de l’objet blob en dehors de la fenêtre de console et collez-la dans un navigateur pour afficher le contenu du fichier dans Stockage Blob. Si vous comparez l’exemple de fichier dans votre répertoire avec le contenu stocké dans le stockage d’objets blob, vous verrez qu’ils sont identiques.

  >[!NOTE]
  >Vous pouvez également utiliser un outil comme l’[Explorateur Stockage Azure](https://storageexplorer.com/?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) pour afficher les fichiers dans Stockage Blob. L’Explorateur Stockage Azure est un outil multiplateforme gratuit qui vous permet d’accéder aux informations de votre compte de stockage.

Une fois que vous avez vérifié les fichiers, appuyez sur la touche **Entrée** pour accomplir la démonstration et supprimer les fichiers de test. Comme vous savez maintenant ce que fait l’exemple, ouvrez le fichier **AzureApp.java** pour examiner le code.

## <a name="understand-the-sample-code"></a>Comprendre l’exemple de code

Ensuite, nous allons parcourir l’exemple de code pas à pas pour vous montrer son fonctionnement.

### <a name="get-references-to-the-storage-objects"></a>Obtenir des références aux objets de stockage

La première chose à faire est de créer les références aux objets utilisés pour accéder et gérer Stockage Blob. Ces objets reposent l’un sur l’autre : chacun est utilisé par le suivant dans la liste.

* Créez une instance de l’objet [CloudStorageAccount](/java/api/com.microsoft.azure.management.storage.storageaccount) pointant vers le compte de stockage.

    L’objet **CloudStorageAccount** est une représentation de votre compte de stockage. Il vous permet de définir et lire les propriétés du compte de stockage par programmation. Vous pouvez utiliser l’objet **CloudStorageAccount** pour créer une instance de l’objet **CloudBlobClient**, nécessaire pour accéder au service BLOB.

* Créez une instance de l’objet **CloudBlobClient** pointant vers le [service Blob](/java/api/com.microsoft.azure.storage.blob.cloudblobclient) de votre compte de stockage.

    L’objet **CloudBlobClient** fournit un point d’accès au service BLOB, ce qui vous permet de définir et lire les propriétés de stockage d’objets blob par programmation. Vous pouvez utiliser l’objet **CloudBlobClient** pour créer une instance de l’objet **CloudBlobContainer**, nécessaire pour créer des conteneurs.

* Créez une instance de l’objet [CloudBlobContainer](/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer) qui représente le conteneur auquel vous accédez. Utilisez les conteneurs pour organiser vos objets blob de la même façon que vous utilisez des dossiers pour organiser vos fichiers sur votre ordinateur.

    Dès que vous disposez de **CloudBlobContainer**, vous pouvez créer une instance de l'objet [CloudBlockBlob](/java/api/com.microsoft.azure.storage.blob.cloudblockblob) qui pointe vers l'objet blob qui vous intéresse, et effectuer une opération de chargement, de téléchargement, de copie ou autre.

> [!IMPORTANT]
> Les noms de conteneurs doivent être en minuscules. Pour plus d’informations sur les conteneurs, consultez [Affectation de noms et références aux conteneurs, objets blob et métadonnées](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata).

### <a name="create-a-container"></a>Créez un conteneur.

Dans cette section, vous créez une instance des objets, un conteneur, puis définissez des autorisations sur le conteneur pour les objets blob soient publics et accessibles par une simple URL. Le conteneur est appelé **quickstartcontainer**.

Cet exemple utilise [CreateIfNotExists](/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer.createifnotexists), car nous voulons créer un conteneur chaque fois que l’exemple est exécuté. Dans un environnement de production où vous utilisez le même conteneur pour toute l’application, il est préférable d’appeler une seule fois **CreateIfNotExists**. Ou vous pouvez créer le conteneur à l’avance pour ne pas avoir à le créer dans le code.

```java
// Parse the connection string and create a blob client to interact with Blob storage
storageAccount = CloudStorageAccount.parse(storageConnectionString);
blobClient = storageAccount.createCloudBlobClient();
container = blobClient.getContainerReference("quickstartcontainer");

// Create the container if it does not exist with public access.
System.out.println("Creating container: " + container.getName());
container.createIfNotExists(BlobContainerPublicAccessType.CONTAINER, new BlobRequestOptions(), new OperationContext());
```

### <a name="upload-blobs-to-the-container"></a>Charger des objets blob dans le conteneur

Pour charger un fichier dans un objet blob de blocs, obtenez une référence à l’objet blob dans le conteneur cible. Une fois que vous avez la référence d’objet blob, vous pouvez y charger des données à l’aide de [CloudBlockBlob.Upload](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.upload). Cette opération crée l’objet blob s’il n’existe pas déjà, et le remplace s’il existe.

L’exemple de code crée un fichier local à utiliser pour le chargement et le téléchargement, en stockant le fichier à charger comme **source** et le nom de l’objet blob dans **blob**. L’exemple suivant charge le fichier sur votre conteneur nommé **quickstartcontainer**.

```java
//Creating a sample file
sourceFile = File.createTempFile("sampleFile", ".txt");
System.out.println("Creating a sample file at: " + sourceFile.toString());
Writer output = new BufferedWriter(new FileWriter(sourceFile));
output.write("Hello Azure!");
output.close();

//Getting a blob reference
CloudBlockBlob blob = container.getBlockBlobReference(sourceFile.getName());

//Creating blob and uploading file to it
System.out.println("Uploading the sample file ");
blob.uploadFromFile(sourceFile.getAbsolutePath());
```

Il existe plusieurs méthodes `upload`, y compris [upload](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.upload), [uploadBlock](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.uploadblock), [uploadFullBlob](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.uploadfullblob), [uploadStandardBlobTier](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.uploadstandardblobtier) et [uploadText](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.uploadtext) que vous pouvez utiliser avec le stockage d’objets blob. Par exemple, avec une chaîne, vous pouvez utiliser la méthode `UploadText` au lieu de la méthode `Upload`.

Les objets blob de blocs peuvent être n’importe quel type de fichier texte ou binaire. Les objets blob de pages sont principalement utilisés pour les fichiers VHD qui soutiennent les machines virtuelles IaaS. Utilisez des objets blob d’ajout pour la journalisation, par exemple, quand vous voulez écrire dans un fichier et continuer à ajouter d’autres informations. La plupart des objets stockés dans Stockage Blob sont des objets blob de blocs.

### <a name="list-the-blobs-in-a-container"></a>Créer la liste des objets blob d’un conteneur

Vous pouvez obtenir la liste des fichiers du conteneur à l’aide de [CloudBlobContainer.ListBlobs](/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer.listblobs). Le code suivant récupère la liste des objets blob, puis effectue une itération sur ces derniers pour montrer les URI des objets blob rencontrés. Vous pouvez copier l’URI à partir de la fenêtre Commande et la coller dans un navigateur pour afficher le fichier.

```java
//Listing contents of container
for (ListBlobItem blobItem : container.listBlobs()) {
    System.out.println("URI of blob is: " + blobItem.getUri());
}
```

### <a name="download-blobs"></a>Télécharger des objets blob

Téléchargez des objets blob sur votre disque local à l’aide de [CloudBlob.DownloadToFile](/java/api/com.microsoft.azure.storage.blob.cloudblob.downloadtofile).

Le code suivant télécharge l’objet blob chargé dans la section précédente et ajoute le suffixe « _DOWNLOADED » au nom de l’objet blob pour pouvoir voir les deux fichiers sur le disque local.

```java
// Download blob. In most cases, you would have to retrieve the reference
// to cloudBlockBlob here. However, we created that reference earlier, and
// haven't changed the blob we're interested in, so we can reuse it.
// Here we are creating a new file to download to. Alternatively you can also pass in the path as a string into downloadToFile method: blob.downloadToFile("/path/to/new/file").
downloadedFile = new File(sourceFile.getParentFile(), "downloadedFile.txt");
blob.downloadToFile(downloadedFile.getAbsolutePath());
```

### <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin des objets blob que vous avez chargés, vous pouvez supprimer le conteneur entier à l’aide de [CloudBlobContainer.DeleteIfExists](/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer.deleteifexists). Cette méthode permet de supprimer les fichiers du conteneur également.

```java
try {
if(container != null)
    container.deleteIfExists();
} catch (StorageException ex) {
System.out.println(String.format("Service error. Http code: %d and error code: %s", ex.getHttpStatusCode(), ex.getErrorCode()));
}

System.out.println("Deleting the source, and downloaded files");

if(downloadedFile != null)
downloadedFile.deleteOnExit();

if(sourceFile != null)
sourceFile.deleteOnExit();
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à transférer des fichiers entre un disque local et le Stockage Blob Azure avec Java. Pour en savoir plus sur l’utilisation de Java, accédez à notre dépôt de code source GitHub.

> [!div class="nextstepaction"]
> [Informations de référence sur les API Java](/java/api/overview/azure/storage?view=azure-java-legacy)
> [Exemples de code pour Java](../common/storage-samples-java.md)