---
title: Guide de démarrage rapide Azure - Créer un objet blob dans un stockage d’objets à l’aide de Go | Microsoft Docs
description: Dans le cadre de ce guide de démarrage rapide, vous allez créer un compte de stockage et un conteneur dans un stockage d’objets (blob). Ensuite, vous utilisez la bibliothèque de client de stockage pour Go, afin de charger un objet blob dans Stockage Azure, de télécharger un objet blob et de répertorier les objets blob dans un conteneur.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 11/14/2018
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.openlocfilehash: bf0e868e9ee746da1dfe1b03403d21f7edb3bd5e
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95544647"
---
# <a name="quickstart-upload-download-and-list-blobs-using-go"></a>Démarrage rapide : Charger, télécharger et répertorier des objets blob à l’aide de Go

Dans ce guide de démarrage rapide, vous apprenez à utiliser le langage de programmation Go pour charger, télécharger et lister des objets blob de blocs dans un conteneur de stockage blob Azure. 

## <a name="prerequisites"></a>Conditions préalables requises

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

Vérifiez que les prérequis suivants sont installés :
 
* [Go 1.8 ou ultérieur](https://golang.org/dl/)
* [SDK Stockage Blob Azure pour Go](https://github.com/azure/azure-storage-blob-go/), en utilisant la commande suivante :

    ```
    go get -u github.com/Azure/azure-storage-blob-go/azblob
    ``` 

    > [!NOTE]
    > Vérifiez que `Azure` figure bien en majuscules dans l’URL afin d’éviter les problèmes d’importation liés à la casse quand vous utilisez le SDK. Mettez aussi `Azure` en majuscules dans vos instructions d’importation.
    
## <a name="download-the-sample-application"></a>Téléchargement de l'exemple d'application
L’[exemple d’application](https://github.com/Azure-Samples/storage-blobs-go-quickstart.git) utilisé dans ce démarrage rapide est une application Go de base.  

Utilisez [git](https://git-scm.com/) pour télécharger une copie de l’application dans votre environnement de développement. 

```bash
git clone https://github.com/Azure-Samples/storage-blobs-go-quickstart 
```

Cette commande clone le dépôt dans votre dossier git local. Pour ouvrir l’exemple Go pour le stockage d’objets Blob, recherchez le fichier storage-quickstart.go.  

[!INCLUDE [storage-copy-account-key-portal](../../../includes/storage-copy-account-key-portal.md)]

## <a name="configure-your-storage-connection-string"></a>Configurer votre chaîne de connexion de stockage
Cette solution demande que le nom et la clé de votre compte de stockage soient stockés en toute sécurité dans les variables d’environnement locales de la machine exécutant l’exemple. Suivez l’un des exemples ci-dessous, en fonction de votre système d’exploitation, pour créer la variable d’environnement.

# <a name="linux"></a>[Linux](#tab/linux)

```
export AZURE_STORAGE_ACCOUNT="<youraccountname>"
export AZURE_STORAGE_ACCESS_KEY="<youraccountkey>"
```

# <a name="windows"></a>[Windows](#tab/windows)

```
setx AZURE_STORAGE_ACCOUNT "<youraccountname>"
setx AZURE_STORAGE_ACCESS_KEY "<youraccountkey>"
```

---

## <a name="run-the-sample"></a>Exécution de l'exemple
Cet exemple crée un fichier de test dans le dossier actuel, le charge dans Stockage Blob, liste les objets blob du conteneur, puis charge le fichier dans la mémoire tampon. 

Pour exécutez l’exemple, utilisez la commande suivante : 

```go run storage-quickstart.go```

La sortie suivante est un exemple de sortie retournée durant l’exécution de l’application :
  
```
Azure Blob storage quick start sample
Creating a container named quickstart-5568059279520899415
Creating a dummy file to test the upload and download
Uploading the file with blob name: 630910657703031215
Blob name: 630910657703031215
Downloaded the blob: hello world
this is a blob
Press the enter key to delete the sample files, example container, and exit the application.
```
Quand vous appuyez sur la touche pour continuer, l’exemple de programme supprime le conteneur de stockage et les fichiers. 

> [!TIP]
> Vous pouvez également utiliser un outil comme l’[Explorateur Stockage Azure](https://storageexplorer.com) pour afficher les fichiers dans Stockage Blob. L’Explorateur Stockage Azure est un outil multiplateforme gratuit qui vous permet d’accéder aux informations de votre compte de stockage. 
>

## <a name="understand-the-sample-code"></a>Comprendre l’exemple de code

Ensuite, nous allons parcourir l’exemple de code pas à pas pour vous montrer son fonctionnement.

### <a name="create-containerurl-and-bloburl-objects"></a>Créer des objets ContainerURL et BlobURL
La première chose à faire est de créer les références aux objets ContainerURL et BlobURL utilisés pour accéder et gérer Stockage Blob. Ces objets offrent des API de bas niveau telles que Create, Upload et Download pour publier des API REST.

* Utilisez la structure [**SharedKeyCredential**](https://godoc.org/github.com/Azure/azure-storage-blob-go/azblob#SharedKeyCredential) pour stocker vos informations d’identification. 

* Créez un [**pipeline**](https://godoc.org/github.com/Azure/azure-storage-blob-go/azblob#NewPipeline) utilisant les options et les informations d’identification. Le pipeline spécifie des éléments tels que les stratégies de nouvelle tentative, la journalisation, la désérialisation des charges utiles de réponse HTTP et bien plus encore.  

* Instanciez un nouveau [**ContainerURL**](https://godoc.org/github.com/Azure/azure-storage-blob-go/azblob#ContainerURL) et un nouvel objet [**BlobURL**](https://godoc.org/github.com/Azure/azure-storage-blob-go/azblob#BlobURL) pour exécuter des opérations sur le conteneur (Create) et les objets blob (Upload et Download).


Une fois que vous avez le ContainerURL, vous pouvez instancier l’objet **BlobURL** qui pointe vers un objet blob et effectuer des opérations telles que le chargement, le téléchargement et la copie.

> [!IMPORTANT]
> Les noms de conteneurs doivent être en minuscules. Pour plus d’informations sur les noms des conteneurs et des objets blob, consultez [Affectation de noms et références aux conteneurs, objets blob et métadonnées](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata).

Dans cette section, vous créez un nouveau conteneur. Le conteneur est appelé **quickstartblobs-[chaîne aléatoire]** . 

```go 
// From the Azure portal, get your storage account name and key and set environment variables.
accountName, accountKey := os.Getenv("AZURE_STORAGE_ACCOUNT"), os.Getenv("AZURE_STORAGE_ACCESS_KEY")
if len(accountName) == 0 || len(accountKey) == 0 {
    log.Fatal("Either the AZURE_STORAGE_ACCOUNT or AZURE_STORAGE_ACCESS_KEY environment variable is not set")
}

// Create a default request pipeline using your storage account name and account key.
credential, err := azblob.NewSharedKeyCredential(accountName, accountKey)
if err != nil {
    log.Fatal("Invalid credentials with error: " + err.Error())
}
p := azblob.NewPipeline(credential, azblob.PipelineOptions{})

// Create a random string for the quick start container
containerName := fmt.Sprintf("quickstart-%s", randomString())

// From the Azure portal, get your storage account blob service URL endpoint.
URL, _ := url.Parse(
    fmt.Sprintf("https://%s.blob.core.windows.net/%s", accountName, containerName))

// Create a ContainerURL object that wraps the container URL and a request
// pipeline to make requests.
containerURL := azblob.NewContainerURL(*URL, p)

// Create the container
fmt.Printf("Creating a container named %s\n", containerName)
ctx := context.Background() // This example uses a never-expiring context
_, err = containerURL.Create(ctx, azblob.Metadata{}, azblob.PublicAccessNone)
handleErrors(err)
```
### <a name="upload-blobs-to-the-container"></a>Charger des objets blob dans le conteneur

Stockage Blob prend en charge les objets blob de blocs, d’ajout et de pages. Comme les objets blob de blocs sont les plus couramment utilisés, nous les utilisons dans ce démarrage rapide.  

Pour télécharger un fichier vers un objet blob, ouvrez le fichier à l’aide de **os.Open**. Vous pouvez ensuite télécharger le fichier vers le chemin d’accès spécifié à l’aide de l’une des API REST : Upload (PutBlob), StageBlock/CommitBlockList (PutBlock/PutBlockList). 

Sinon, le kit de développement logiciel offre des [API de niveau supérieur](https://github.com/Azure/azure-storage-blob-go/blob/master/azblob/highlevel.go) qui s’appuient sur les API REST de bas niveau. Par exemple, la fonction **_UploadFileToBlockBlob_* _ utilise des opérations StageBlock (PutBlock) pour charger simultanément un fichier en plusieurs blocs afin d’optimiser le débit. Si le fichier fait moins de 256 Mo, il utilise Upload (PutBlob) à la place pour effectuer le transfert en une seule transaction.

L’exemple suivant charge le fichier dans votre conteneur nommé _*quickstartblobs-[randomstring]**.

```go
// Create a file to test the upload and download.
fmt.Printf("Creating a dummy file to test the upload and download\n")
data := []byte("hello world this is a blob\n")
fileName := randomString()
err = ioutil.WriteFile(fileName, data, 0700)
handleErrors(err)

// Here's how to upload a blob.
blobURL := containerURL.NewBlockBlobURL(fileName)
file, err := os.Open(fileName)
handleErrors(err)

// You can use the low-level Upload (PutBlob) API to upload files. Low-level APIs are simple wrappers for the Azure Storage REST APIs.
// Note that Upload can upload up to 256MB data in one shot. Details: https://docs.microsoft.com/rest/api/storageservices/put-blob
// To upload more than 256MB, use StageBlock (PutBlock) and CommitBlockList (PutBlockList) functions. 
// Following is commented out intentionally because we will instead use UploadFileToBlockBlob API to upload the blob
// _, err = blobURL.Upload(ctx, file, azblob.BlobHTTPHeaders{ContentType: "text/plain"}, azblob.Metadata{}, azblob.BlobAccessConditions{})
// handleErrors(err)

// The high-level API UploadFileToBlockBlob function uploads blocks in parallel for optimal performance, and can handle large files as well.
// This function calls StageBlock/CommitBlockList for files larger 256 MBs, and calls Upload for any file smaller
fmt.Printf("Uploading the file with blob name: %s\n", fileName)
_, err = azblob.UploadFileToBlockBlob(ctx, file, blobURL, azblob.UploadToBlockBlobOptions{
    BlockSize:   4 * 1024 * 1024,
    Parallelism: 16})
handleErrors(err)
```

### <a name="list-the-blobs-in-a-container"></a>Créer la liste des objets blob d’un conteneur

Obtenez la liste des fichiers du conteneur à l’aide de la méthode **listBlobs** sur un **ContainerURL**. ListBlobs retourne un seul segment d’objets blob (jusqu’à 5 000) à partir du **marqueur** spécifié. Utilisez un marqueur vide pour commencer l’énumération depuis le début. Les noms des objets blob retournés dans l’ordre lexicographique. Après avoir obtenu un segment, traitez-le, puis appelez de nouveau ListBlobs en passant le marqueur retourné précédemment.  

```go
// List the container that we have created above
fmt.Println("Listing the blobs in the container:")
for marker := (azblob.Marker{}); marker.NotDone(); {
    // Get a result segment starting with the blob indicated by the current Marker.
    listBlob, err := containerURL.ListBlobsFlatSegment(ctx, marker, azblob.ListBlobsSegmentOptions{})
    handleErrors(err)

    // ListBlobs returns the start of the next segment; you MUST use this to get
    // the next segment (after processing the current result segment).
    marker = listBlob.NextMarker

    // Process the blobs returned in this result segment (if the segment is empty, the loop body won't execute)
    for _, blobInfo := range listBlob.Segment.BlobItems {
        fmt.Print(" Blob name: " + blobInfo.Name + "\n")
    }
}
```

### <a name="download-the-blob"></a>Télécharger l’objet blob

Téléchargez des objets BLOB à l’aide de la fonction de bas niveau **Download** sur un BlobURL. Ceci renverra un struct **DownloadResponse**. Exécutez la fonction **Body** au niveau du struct pour obtenir un flux de lecture de données **RetryReader**. Si une connexion échoue lors de la lecture, des requêtes supplémentaires sont envoyées pour rétablir une connexion et poursuivre la lecture. En cas de spécification d’une option RetryReaderOption avec MaxRetryRequests défini sur 0 (valeur par défaut), le corps de réponse d’origine est renvoyé et aucune nouvelle tentative ne sera effectuée. Vous pouvez également utiliser l’API de haut niveau **DownloadBlobToBuffer** ou **DownloadBlobToFile** pour simplifier votre code.

Le code suivant télécharge l’objet blob à l’aide de la fonction **Download**. Le contenu de l’objet blob est écrit dans une mémoire tampon et affiché sur la console.

```go
// Here's how to download the blob
downloadResponse, err := blobURL.Download(ctx, 0, azblob.CountToEnd, azblob.BlobAccessConditions{}, false)

// NOTE: automatically retries are performed if the connection fails
bodyStream := downloadResponse.Body(azblob.RetryReaderOptions{MaxRetryRequests: 20})

// read the body into a buffer
downloadedData := bytes.Buffer{}
_, err = downloadedData.ReadFrom(bodyStream)
handleErrors(err)
```

### <a name="clean-up-resources"></a>Nettoyer les ressources
Si vous n’avez plus besoin des objets blob chargés dans ce guide de démarrage rapide, vous pouvez supprimer l’intégralité du conteneur à l’aide de la méthode **Delete**. 

```go
// Cleaning up the quick start by deleting the container and the file created locally
fmt.Printf("Press enter key to delete the sample files, example container, and exit the application.\n")
bufio.NewReader(os.Stdin).ReadBytes('\n')
fmt.Printf("Cleaning up.\n")
containerURL.Delete(ctx, azblob.ContainerAccessConditions{})
file.Close()
os.Remove(fileName)
```

## <a name="resources-for-developing-go-applications-with-blobs"></a>Ressources sur le développement d’applications Go avec des objets blob

Consultez ces ressources supplémentaires sur le développement Go avec le stockage Blob :

- Affichez et installez le [code source de la bibliothèque de client Go](https://github.com/Azure/azure-storage-blob-go) pour Stockage Azure sur GitHub.
- Explorez les [exemples de Stockage Blob](https://godoc.org/github.com/Azure/azure-storage-blob-go/azblob#pkg-examples) écrits avec la bibliothèque de client Go.

## <a name="next-steps"></a>Étapes suivantes
 
Dans ce guide de démarrage rapide, vous avez appris à transférer des fichiers entre un disque local et Stockage Blob Azure avec Go. Pour plus d’informations sur le kit de développement logiciel Azure Storage Blob, affichez le [Code Source](https://github.com/Azure/azure-storage-blob-go/) et la [référence de l’API](https://godoc.org/github.com/Azure/azure-storage-blob-go/azblob).