---
title: Développer pour Azure Files avec Java | Microsoft Docs
description: Découvrez comment développer des services et applications Java qui utilisent Azure Files pour stocker les données de fichiers.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 11/18/2020
ms.custom: devx-track-java
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 25baa278961b93b04e60f2e997b98753cb6cf3ab
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024107"
---
# <a name="develop-for-azure-files-with-java"></a>Développer pour Azure Files avec Java

[!INCLUDE [storage-selector-file-include](../../../includes/storage-selector-file-include.md)]

Découvrez les bases du développement d’applications Java qui utilisent Azure Files pour stocker des données. Créez une application console et découvrez les actions de base à l’aide des API Azure Files :

- Créer et supprimer des partages de fichiers Azure
- Créer et supprimer des répertoires
- Énumérer des fichiers et répertoires dans un partage de fichiers Azure
- Charger, télécharger et supprimer un fichier

[!INCLUDE [storage-check-out-samples-java](../../../includes/storage-check-out-samples-java.md)]

## <a name="create-a-java-application"></a>Création d’une application Java

Pour générer les exemples, vous avez besoin du SDK Java et du [SDK Stockage Azure pour Java](https://github.com/azure/azure-sdk-for-java). Vous devez également avoir préalablement créé un compte de stockage Azure.

## <a name="set-up-your-application-to-use-azure-files"></a>Configurer votre application pour utiliser Azure Files

Pour utiliser les API Azure Files, ajoutez le code suivant au début du fichier Java depuis lequel vous voulez accéder à Azure Files.

# <a name="java-v12"></a>[Java v12](#tab/java)

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_ImportStatements":::

# <a name="java-v11"></a>[Java v11](#tab/java11)

```java
// Include the following imports to use Azure Files APIs v11
import com.microsoft.azure.storage.*;
import com.microsoft.azure.storage.file.*;
```

---

## <a name="set-up-an-azure-storage-connection-string"></a>Configuration d’une chaîne de connexion au stockage Azure

Pour utiliser Azure Files, vous devez vous connecter à votre compte de stockage Azure. Configurez une chaîne de connexion et utilisez-la pour vous connecter à votre compte de stockage. Définissez une variable statique pour contenir la chaîne de connexion.

# <a name="java-v12"></a>[Java v12](#tab/java)

Remplacez *\<storage_account_name\>* et *\<storage_account_key\>* par les valeurs réelles de votre compte de stockage.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_ConnectionString":::

# <a name="java-v11"></a>[Java v11](#tab/java11)

Remplacez *your_storage_account_name* (nom de votre compte de stockage) et *your_storage_account_key* (clé de votre compte de stockage) par les valeurs réelles de votre compte de stockage.

```java
// Configure the connection-string with your values
public static final String storageConnectionString =
    "DefaultEndpointsProtocol=http;" +
    "AccountName=your_storage_account_name;" +
    "AccountKey=your_storage_account_key";
```

---

## <a name="access-azure-files-storage"></a>Accéder au stockage Azure Files

# <a name="java-v12"></a>[Java v12](#tab/java)

Pour accéder à Azure Files, créez un objet [ShareClient](/java/api/com.azure.storage.file.share.shareclient). Utilisez la classe [ShareClientBuilder](/java/api/com.azure.storage.file.share.shareclientbuilder) pour générer un nouvel objet **ShareClient**.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_createClient":::

# <a name="java-v11"></a>[Java v11](#tab/java11)

Pour accéder à votre compte de stockage, utilisez l’objet **CloudStorageAccount**, en transmettant la chaîne de connexion à sa méthode **parse**.

```java
// Use the CloudStorageAccount object to connect to your storage account
try {
    CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);
} catch (InvalidKeyException invalidKey) {
    // Handle the exception
}
```

**CloudStorageAccount.parse** lève une exception InvalidKeyException. Vous devrez donc le placer dans un bloc try/catch.

---

## <a name="create-a-file-share"></a>Créer un partage de fichier

Tous les fichiers et répertoires dans Azure Files sont stockés dans un conteneur qu’on appelle un partage.

# <a name="java-v12"></a>[Java v12](#tab/java)

La méthode [ShareClient.create](/java/api/com.azure.storage.file.share.shareclient.create) lève une exception si le partage existe déjà. Placez l’appel à **create** dans un bloc `try/catch` et gérez l’exception.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_createFileShare":::

# <a name="java-v11"></a>[Java v11](#tab/java11)

Pour pouvoir accéder à un partage et à son contenu, créez un client Azure Files.

```java
// Create the Azure Files client.
CloudFileClient fileClient = storageAccount.createCloudFileClient();
```

À l’aide de ce dernier, vous pouvez ensuite obtenir une référence à un partage.

```java
// Get a reference to the file share
CloudFileShare share = fileClient.getShareReference("sampleshare");
```

Pour créer le partage, utilisez la méthode **createIfNotExists** de l’objet **CloudFileShare**.

```java
if (share.createIfNotExists()) {
    System.out.println("New share created");
}
```

À ce stade, le **partage** contient une référence à un partage nommé **sample share**.

---

## <a name="delete-a-file-share"></a>Supprimer un partage de fichiers

L’exemple de code suivant supprime un partage de fichiers.

# <a name="java-v12"></a>[Java v12](#tab/java)

Supprimez un partage en appelant la méthode [ShareClient.delete](/java/api/com.azure.storage.file.share.shareclient.delete).

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_deleteFileShare":::

# <a name="java-v11"></a>[Java v11](#tab/java11)

Supprimez un partage en appelant la méthode **deleteIfExists** sur un objet **CloudFileShare**.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

    // Create the file client.
   CloudFileClient fileClient = storageAccount.createCloudFileClient();

   // Get a reference to the file share
   CloudFileShare share = fileClient.getShareReference("sampleshare");

   if (share.deleteIfExists()) {
       System.out.println("sampleshare deleted");
   }
} catch (Exception e) {
    e.printStackTrace();
}
```

---

## <a name="create-a-directory"></a>Créer un répertoire

Organisez le stockage en plaçant des fichiers dans des sous-répertoires, plutôt que de tous les mettre dans le répertoire racine.

# <a name="java-v12"></a>[Java v12](#tab/java)

Le code suivant crée un répertoire en appelant [ShareDirectoryClient.create](/java/api/com.azure.storage.file.share.sharedirectoryclient.create). L’exemple de méthode retourne une valeur `Boolean` indiquant si le répertoire a bien été créé.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_createDirectory":::

# <a name="java-v11"></a>[Java v11](#tab/java11)

Le code ci-dessous crée un sous-répertoire nommé **sampledir** sous le répertoire racine.

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

//Get a reference to the sampledir directory
CloudFileDirectory sampleDir = rootDir.getDirectoryReference("sampledir");

if (sampleDir.createIfNotExists()) {
    System.out.println("sampledir created");
} else {
    System.out.println("sampledir already exists");
}
```

---

## <a name="delete-a-directory"></a>Supprimer un répertoire

La suppression d’un répertoire est une tâche simple. Vous ne pouvez pas supprimer un répertoire qui contient toujours des fichiers ou des sous-répertoires.

# <a name="java-v12"></a>[Java v12](#tab/java)

La méthode [ShareDirectoryClient.delete](/java/api/com.azure.storage.file.share.sharedirectoryclient.delete) lève une exception si le répertoire n’existe pas ou s’il n’est pas vide. Placez l’appel à **delete** dans un bloc `try/catch` et gérez l’exception.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_deleteDirectory":::

# <a name="java-v11"></a>[Java v11](#tab/java11)

```java
// Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

// Get a reference to the directory you want to delete
CloudFileDirectory containerDir = rootDir.getDirectoryReference("sampledir");

// Delete the directory
if ( containerDir.deleteIfExists() ) {
    System.out.println("Directory deleted");
}
```

---

## <a name="enumerate-files-and-directories-in-an-azure-file-share"></a>Énumérer des fichiers et répertoires dans un partage de fichiers Azure

# <a name="java-v12"></a>[Java v12](#tab/java)

Obtenez la liste des fichiers et répertoires en appelant [ShareDirectoryClient.listFilesAndDirectories](/java/api/com.azure.storage.file.share.sharedirectoryclient.listfilesanddirectories). La méthode retourne une liste d’objets [ShareFileItem](/java/api/com.azure.storage.file.share.models.sharefileitem) sur laquelle vous pouvez effectuer une itération. Le code suivant répertorie les fichiers et les répertoires dans le répertoire spécifié par le paramètre *dirName*.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_enumerateFilesAndDirs":::

# <a name="java-v11"></a>[Java v11](#tab/java11)

Obtenez la liste des fichiers et répertoires en appelant **listFilesAndDirectories** sur une référence **CloudFileDirectory**. La méthode retourne une liste d’objets **ListFileItem** sur laquelle vous pouvez effectuer une itération. Le code suivant répertorie les fichiers et répertoires du répertoire racine.

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

for ( ListFileItem fileItem : rootDir.listFilesAndDirectories() ) {
    System.out.println(fileItem.getUri());
}
```

---

## <a name="upload-a-file"></a>Charger un fichier

Découvrez comment charger un fichier à partir du stockage local.

# <a name="java-v12"></a>[Java v12](#tab/java)

Le code suivant charge un fichier local dans le stockage Azure Files en appelant la méthode [ShareFileClient.uploadFromFile](/java/api/com.azure.storage.file.share.sharefileclient.uploadfromfile). L’exemple de méthode suivant retourne une valeur `Boolean` indiquant si le fichier spécifié a bien été chargé.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_uploadFile":::

# <a name="java-v11"></a>[Java v11](#tab/java11)

Obtenez une référence au répertoire dans lequel le fichier sera chargé en appelant la méthode **getRootDirectoryReference** sur l’objet share.

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();
```

Maintenant que vous avez une référence au répertoire racine du partage, vous pouvez télécharger un fichier vers ce répertoire à l’aide du code ci-après.

```java
// Define the path to a local file.
final String filePath = "C:\\temp\\Readme.txt";

CloudFile cloudFile = rootDir.getFileReference("Readme.txt");
cloudFile.uploadFromFile(filePath);
```

---

## <a name="download-a-file"></a>Téléchargement d’un fichier

Le téléchargement de fichiers à partir du stockage Azure Files est l’une des opérations les plus fréquentes.

# <a name="java-v12"></a>[Java v12](#tab/java)

L’exemple suivant télécharge le fichier spécifié dans le répertoire local spécifié dans le paramètre *destDir*. L’exemple de méthode rend le nom de fichier téléchargé unique en ajoutant la date et l’heure au début.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_downloadFile":::

# <a name="java-v11"></a>[Java v11](#tab/java11)

L’exemple de code suivant télécharge SampleFile.txt et affiche son contenu.

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

//Get a reference to the directory that contains the file
CloudFileDirectory sampleDir = rootDir.getDirectoryReference("sampledir");

//Get a reference to the file you want to download
CloudFile file = sampleDir.getFileReference("SampleFile.txt");

//Write the contents of the file to the console.
System.out.println(file.downloadText());
```

---

## <a name="delete-a-file"></a>Supprimer un fichier

La suppression de fichiers est également une opération courante dans Azure Files.

# <a name="java-v12"></a>[Java v12](#tab/java)

Le code suivant supprime le fichier spécifié. Tout d’abord, l’exemple crée un [ShareDirectoryClient](/java/api/com.azure.storage.file.share.sharedirectoryclient) en fonction du paramètre *dirName*. Ensuite, le code obtient un [ShareFileClient](/java/api/com.azure.storage.file.share.sharefileclient) à partir du client de répertoire, en fonction du paramètre *fileName*. Enfin, l’exemple de méthode appelle [ShareFileClient.delete](/java/api/com.azure.storage.file.share.sharefileclient.delete) pour supprimer le fichier.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_deleteFile":::

# <a name="java-v11"></a>[Java v11](#tab/java11)

Le code suivant supprime un fichier nommé SampleFile.txt et stocké dans un répertoire nommé **sampledir**.

```java
// Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

// Get a reference to the directory where the file to be deleted is in
CloudFileDirectory containerDir = rootDir.getDirectoryReference("sampledir");

String filename = "SampleFile.txt"
CloudFile file;

file = containerDir.getFileReference(filename)
if ( file.deleteIfExists() ) {
    System.out.println(filename + " was deleted");
}
```

---

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les autres API de stockage Azure, suivez ces liens.

- [Azure for Java developers](/azure/developer/java) (Azure pour les développeurs Java)
- [Kit de développement logiciel (SDK) Azure pour Java](https://github.com/azure/azure-sdk-for-java)
- [Kit de développement logiciel (SDK) Azure pour Android](https://github.com/azure/azure-sdk-for-android)
- [Bibliothèque de client de partage de fichiers Azure pour Java - Référence du Kit de développement logiciel (SDK)](/java/api/overview/azure/storage-file-share-readme)
- [API REST des services d’Azure Storage](/rest/api/storageservices/)
- [Blog de l'équipe Azure Storage](https://azure.microsoft.com/blog/topics/storage-backup-and-recovery/)
- [Transfert de données avec l’utilitaire de ligne de commande AzCopy](../common/storage-use-azcopy-v10.md)
- [Résolution des problèmes liés à Azure Files - Windows](storage-troubleshoot-windows-file-connection-problems.md)
