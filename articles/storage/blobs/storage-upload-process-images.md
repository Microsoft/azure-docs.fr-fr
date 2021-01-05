---
title: Charger des données d’image dans le cloud avec le Stockage Azure | Microsoft Docs
description: Utilisez le Stockage Blob Azure avec des applications web pour stocker des données d’application dans un compte de stockage. Ce tutoriel montre comment créer une application web qui stocke et affiche des images à partir d’un stockage Azure.
author: mhopkins-msft
ms.service: storage
ms.subservice: blobs
ms.topic: tutorial
ms.date: 06/24/2020
ms.author: mhopkins
ms.reviewer: dineshm
ms.custom: devx-track-js, devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: c20c78cb3c946c666b1640ccac6f86c9b52387ea
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94843868"
---
# <a name="tutorial-upload-image-data-in-the-cloud-with-azure-storage"></a>Tutoriel : Charger des données d’image dans le cloud avec le Stockage Azure

Ce tutoriel est la première partie d’une série d’étapes. Dans ce tutoriel, vous allez apprendre à déployer une application web. L’application web utilise la bibliothèque de client Stockage Blob Azure dans le but de charger des images dans un compte de stockage. À la fin, vous disposerez d’une application web qui stocke et affiche des images à partir du Stockage Azure.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

![Application de redimensionnement d’image dans .NET](media/storage-upload-process-images/figure2.png)

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

![Application de redimensionnement d’image dans JavaScript](media/storage-upload-process-images/upload-app-nodejs-thumb.png)

---

Dans ce premier volet, vous apprenez à :

> [!div class="checklist"]

> * Créez un compte de stockage.
> * Créer un conteneur et définir des autorisations
> * Récupérer une clé d’accès
> * Déployer une application web dans Azure
> * Configuration des paramètres d’application
> * Interagir avec l’application web

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vous devez disposer d’un abonnement Azure. Avant de commencer, créez un [compte gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Pour installer et utiliser l’interface CLI en local, exécutez Azure CLI version 2.0.4 ou une version ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli). 

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.  

L’exemple suivant crée un groupe de ressources nommé `myResourceGroup`.

```bash
az group create --name myResourceGroup --location southeastasia
```

```powershell
az group create --name myResourceGroup --location southeastasia
```

## <a name="create-a-storage-account"></a>Créez un compte de stockage.

L’exemple charge des images sur un conteneur Blob dans un compte de stockage Azure. Le compte de stockage Azure fournit un espace de noms unique pour stocker les objets de données de Stockage Azure et y accéder. Créez un compte de stockage dans le groupe de ressources que vous avez créé à l’aide de la commande [az storage account create](/cli/azure/storage/account).

> [!IMPORTANT]
> Dans la deuxième partie de ce tutoriel, vous utiliserez Azure Event Grid avec le stockage Blob. Veillez à créer votre compte de stockage dans une région Azure qui prend en charge Event Grid. Pour obtenir la liste des régions prises en charge, consultez [Produits Azure par région](https://azure.microsoft.com/global-infrastructure/services/?products=event-grid&regions=all).

Dans la commande suivante, indiquez le nom global unique de votre compte de stockage Blob dans l’espace réservé `<blob_storage_account>`.

```bash
blobStorageAccount="<blob_storage_account>"

az storage account create --name $blobStorageAccount --location southeastasia \
  --resource-group myResourceGroup --sku Standard_LRS --kind StorageV2 --access-tier hot
```

```powershell
$blobStorageAccount="<blob_storage_account>"

az storage account create --name $blobStorageAccount --location southeastasia `
  --resource-group myResourceGroup --sku Standard_LRS --kind StorageV2 --access-tier hot
```

## <a name="create-blob-storage-containers"></a>Créer des conteneurs de stockage Blob

L’application utilise deux conteneurs dans le compte de stockage Blob. Les conteneurs s’apparentent à des dossiers où sont stockés les objets blob. C’est dans le conteneur *images* que l’application charge les images pleine résolution. Dans l’une des parties suivantes de la série, une application de fonction Azure charge les miniatures d’images redimensionnées dans le conteneur *thumbnails*.

Récupérez la clé du compte de stockage avec la commande [az storage account keys list](/cli/azure/storage/account/keys). Ensuite, utilisez cette clé pour créer deux conteneurs avec la commande [az storage container create](/cli/azure/storage/container).

L’accès public du conteneur *Images* est défini sur `off`. L’accès public du conteneur *Miniatures* est défini sur `container`. Le paramètre d’accès public `container` permet aux utilisateurs qui visitent la page web de voir les miniatures.

```bash
blobStorageAccountKey=$(az storage account keys list -g myResourceGroup \
  -n $blobStorageAccount --query "[0].value" --output tsv)

az storage container create --name images \
  --account-name $blobStorageAccount \
  --account-key $blobStorageAccountKey

az storage container create --name thumbnails \
  --account-name $blobStorageAccount \
  --account-key $blobStorageAccountKey --public-access container
```

```powershell
$blobStorageAccountKey=$(az storage account keys list -g myResourceGroup `
  -n $blobStorageAccount --query "[0].value" --output tsv)

az storage container create --name images `
  --account-name $blobStorageAccount `
  --account-key $blobStorageAccountKey

az storage container create --name thumbnails `
  --account-name $blobStorageAccount `
  --account-key $blobStorageAccountKey --public-access container
```

Notez le nom et la clé de votre compte de stockage Blob. L’exemple d’application utilise ces paramètres pour se connecter au compte de stockage afin de charger les images. 

## <a name="create-an-app-service-plan"></a>Créer un plan App Service

Un [plan App Service](../../app-service/overview-hosting-plans.md) spécifie l’emplacement, la taille et les fonctionnalités de la batterie de serveurs web qui héberge votre application.

Créez un plan App Service avec la commande [az appservice plan create](/cli/azure/appservice/plan).

L’exemple suivant crée un plan App Service nommé `myAppServicePlan` dans le niveau tarifaire **Gratuit** :

```bash
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku Free
```

```powershell
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku Free
```

## <a name="create-a-web-app"></a>Créer une application web

L’application web offre un espace d’hébergement au code de l’exemple d’application qui est déployé à partir du dépôt d’exemples GitHub. Créez une [application web](../../app-service/overview.md) dans le plan App Service `myAppServicePlan` avec la commande [az webapp create](/cli/azure/webapp).  

Dans la commande suivante, remplacez `<web_app>` par un nom unique. Les caractères valides sont `a-z`, `0-9` et `-`. Si `<web_app>` n’est pas unique, le message d’erreur suivant s’affiche : *Un site web avec ce nom `<web_app>` existe déjà.* L’URL par défaut de l’application web est `https://<web_app>.azurewebsites.net`.  

```bash
webapp="<web_app>"

az webapp create --name $webapp --resource-group myResourceGroup --plan myAppServicePlan
```

```powershell
$webapp="<web_app>"

az webapp create --name $webapp --resource-group myResourceGroup --plan myAppServicePlan
```

## <a name="deploy-the-sample-app-from-the-github-repository"></a>Déployer l’exemple d’application à partir du référentiel GitHub

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

App Service prend en charge plusieurs façons de déployer du contenu vers une application web. Dans ce didacticiel, vous déployez l’application web depuis un [exemple de référentiel GitHub public](https://github.com/Azure-Samples/storage-blob-upload-from-webapp). Configurez le déploiement GitHub sur l’application web avec la commande [az webapp deployment source config](/cli/azure/webapp/deployment/source).

L’exemple de projet contient une application [ASP.NET MVC](https://www.asp.net/mvc). Cette application accepte une image, l’enregistre dans un compte de stockage et affiche des images à partir d’un conteneur de miniatures. L’application web utilise les espaces de noms [Azure.Storage](/dotnet/api/azure.storage), [Azure.Storage.Blobs](/dotnet/api/azure.storage.blobs) et [Azure.Storage.Blobs.Models](/dotnet/api/azure.storage.blobs.models) pour interagir avec le service de stockage Azure.

```bash
az webapp deployment source config --name $webapp --resource-group myResourceGroup \
  --branch master --manual-integration \
  --repo-url https://github.com/Azure-Samples/storage-blob-upload-from-webapp
```

```powershell
az webapp deployment source config --name $webapp --resource-group myResourceGroup `
  --branch master --manual-integration `
  --repo-url https://github.com/Azure-Samples/storage-blob-upload-from-webapp
```

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

App Service prend en charge plusieurs façons de déployer du contenu vers une application web. Dans ce didacticiel, vous déployez l’application web depuis un [exemple de référentiel GitHub public](https://github.com/Azure-Samples/azure-sdk-for-js-storage-blob-stream-nodejs). Configurez le déploiement GitHub sur l’application web avec la commande [az webapp deployment source config](/cli/azure/webapp/deployment/source).

```bash
az webapp deployment source config --name $webapp --resource-group myResourceGroup \
  --branch master --manual-integration \
  --repo-url https://github.com/Azure-Samples/azure-sdk-for-js-storage-blob-stream-nodejs
```

```powershell
az webapp deployment source config --name $webapp --resource-group myResourceGroup `
  --branch master --manual-integration `
  --repo-url https://github.com/Azure-Samples/azure-sdk-for-js-storage-blob-stream-nodejs
```

---

## <a name="configure-web-app-settings"></a>Configurer les paramètres de l’application web

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

L’exemple d’application web utilise les [API de stockage Azure pour .NET](/dotnet/api/overview/azure/storage) pour charger des images. Les informations d’identification du compte de stockage sont définies dans les paramètres de l’application web. Ajoutez des paramètres d’application à l’application déployée avec la commande [az webapp config appsettings set](/cli/azure/webapp/config/appsettings).

```bash
az webapp config appsettings set --name $webapp --resource-group myResourceGroup \
  --settings AzureStorageConfig__AccountName=$blobStorageAccount \
    AzureStorageConfig__ImageContainer=images \
    AzureStorageConfig__ThumbnailContainer=thumbnails \
    AzureStorageConfig__AccountKey=$blobStorageAccountKey
```

```powershell
az webapp config appsettings set --name $webapp --resource-group myResourceGroup `
  --settings AzureStorageConfig__AccountName=$blobStorageAccount `
    AzureStorageConfig__ImageContainer=images `
    AzureStorageConfig__ThumbnailContainer=thumbnails `
    AzureStorageConfig__AccountKey=$blobStorageAccountKey
```

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

L’exemple d’application web utilise la [bibliothèque de client de stockage Azure pour JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage) afin de charger des images. Les informations d’identification du compte de stockage sont définies dans les paramètres de l’application web. Ajoutez des paramètres d’application à l’application déployée avec la commande [az webapp config appsettings set](/cli/azure/webapp/config/appsettings).

```bash
az webapp config appsettings set --name $webapp --resource-group myResourceGroup \
  --settings AZURE_STORAGE_ACCOUNT_NAME=$blobStorageAccount \
    AZURE_STORAGE_ACCOUNT_ACCESS_KEY=$blobStorageAccountKey
```

```powershell
az webapp config appsettings set --name $webapp --resource-group myResourceGroup `
  --settings AZURE_STORAGE_ACCOUNT_NAME=$blobStorageAccount `
  AZURE_STORAGE_ACCOUNT_ACCESS_KEY=$blobStorageAccountKey
```

---

Une fois l’application web déployée et configurée, vous pouvez tester la fonctionnalité de chargement d’images dans l’application.

## <a name="upload-an-image"></a>Charger une image

Pour tester l’application web, accédez à l’URL de l’application publiée. L’URL par défaut de l’application web est `https://<web_app>.azurewebsites.net`.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

Sélectionnez la zone **Charger des photos** pour spécifier et charger un fichier, ou faites glisser un fichier dans la zone. L’image disparaît si le chargement a réussi. La section **Miniatures générées** reste vide jusqu’aux tests effectués plus loin dans ce tutoriel.

![Télécharger des photos dans .NET](media/storage-upload-process-images/figure1.png)

Dans l’exemple de code, la tâche `UploadFileToStorage` du fichier *Storagehelper.cs* sert à charger les images dans le conteneur *Images* au sein du compte de stockage suivant la méthode [UploadAsync](/dotnet/api/azure.storage.blobs.blobclient.uploadasync). L’exemple de code suivant contient la tâche `UploadFileToStorage`.

```csharp
public static async Task<bool> UploadFileToStorage(Stream fileStream, string fileName,
                                                    AzureStorageConfig _storageConfig)
{
    // Create a URI to the blob
    Uri blobUri = new Uri("https://" +
                          _storageConfig.AccountName +
                          ".blob.core.windows.net/" +
                          _storageConfig.ImageContainer +
                          "/" + fileName);

    // Create StorageSharedKeyCredentials object by reading
    // the values from the configuration (appsettings.json)
    StorageSharedKeyCredential storageCredentials =
        new StorageSharedKeyCredential(_storageConfig.AccountName, _storageConfig.AccountKey);

    // Create the blob client.
    BlobClient blobClient = new BlobClient(blobUri, storageCredentials);

    // Upload the file
    await blobClient.UploadAsync(fileStream);

    return await Task.FromResult(true);
}
```

Les classes et méthodes suivantes sont utilisées dans la tâche précédente :

| Classe | Méthode |
|-------|--------|
| [Uri](/dotnet/api/system.uri) | [Constructeur d’URI](/dotnet/api/system.uri.-ctor) |
| [StorageSharedKeyCredential](/dotnet/api/azure.storage.storagesharedkeycredential) | [Constructeur StorageSharedKeyCredential(chaîne, chaîne)](/dotnet/api/azure.storage.storagesharedkeycredential.-ctor) |
| [BlobClient](/dotnet/api/azure.storage.blobs.blobclient) | [UploadAsync](/dotnet/api/azure.storage.blobs.blobclient.uploadasync) |

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

Sélectionnez **Choisir un fichier** pour sélectionner un fichier, puis cliquez sur **Charger une image**. La section **Miniatures générées** reste vide jusqu’aux tests effectués plus loin dans ce tutoriel.

![Charger des photos dans Node.js](media/storage-upload-process-images/upload-app-nodejs.png)

Dans l’exemple de code, l’itinéraire `post` est responsable du chargement de l’image dans un conteneur d’objets blob. L’itinéraire utilise les modules pour faciliter le traitement du chargement :

- [multer](https://github.com/expressjs/multer) implémente la stratégie de chargement pour le gestionnaire de routage.
- [into-stream](https://github.com/sindresorhus/into-stream) convertit la mémoire tampon en flux, comme le nécessite [uploadStream](/javascript/api/%40azure/storage-blob/blockblobclient#uploadstream-readable--number--number--blockblobuploadstreamoptions-).

Étant donné que le fichier est envoyé à la route, le contenu du fichier reste en mémoire jusqu’à son chargement dans le conteneur d’objets blob.

> [!IMPORTANT]
> Le chargement en mémoire des fichiers volumineux peut nuire aux performances de votre application web. Si vous pensez que des utilisateurs pourraient publier des fichiers volumineux, vous devrez peut-être utiliser la mise en lots de fichiers sur le système de fichiers, puis planifier les chargements dans le stockage Blob. Une fois que les fichiers sont dans le stockage Blob, vous pouvez les supprimer du système de fichiers du serveur.

```javascript
if (process.env.NODE_ENV !== 'production') {
  require('dotenv').config();
}

const {
  BlobServiceClient,
  StorageSharedKeyCredential,
  newPipeline
} = require('@azure/storage-blob');

const express = require('express');
const router = express.Router();
const containerName1 = 'thumbnails';
const multer = require('multer');
const inMemoryStorage = multer.memoryStorage();
const uploadStrategy = multer({ storage: inMemoryStorage }).single('image');
const getStream = require('into-stream');
const containerName2 = 'images';
const ONE_MEGABYTE = 1024 * 1024;
const uploadOptions = { bufferSize: 4 * ONE_MEGABYTE, maxBuffers: 20 };

const sharedKeyCredential = new StorageSharedKeyCredential(
  process.env.AZURE_STORAGE_ACCOUNT_NAME,
  process.env.AZURE_STORAGE_ACCOUNT_ACCESS_KEY);
const pipeline = newPipeline(sharedKeyCredential);

const blobServiceClient = new BlobServiceClient(
  `https://${process.env.AZURE_STORAGE_ACCOUNT_NAME}.blob.core.windows.net`,
  pipeline
);

const getBlobName = originalName => {
  // Use a random number to generate a unique file name, 
  // removing "0." from the start of the string.
  const identifier = Math.random().toString().replace(/0\./, '');
  return `${identifier}-${originalName}`;
};

router.get('/', async (req, res, next) => {

  let viewData;

  try {
    const containerClient = blobServiceClient.getContainerClient(containerName1);
    const listBlobsResponse = await containerClient.listBlobFlatSegment();

    for await (const blob of listBlobsResponse.segment.blobItems) {
      console.log(`Blob: ${blob.name}`);
    }

    viewData = {
      title: 'Home',
      viewName: 'index',
      accountName: process.env.AZURE_STORAGE_ACCOUNT_NAME,
      containerName: containerName1
    };

    if (listBlobsResponse.segment.blobItems.length) {
      viewData.thumbnails = listBlobsResponse.segment.blobItems;
    }
  } catch (err) {
    viewData = {
      title: 'Error',
      viewName: 'error',
      message: 'There was an error contacting the blob storage container.',
      error: err
    };
    res.status(500);
  } finally {
    res.render(viewData.viewName, viewData);
  }
});

router.post('/', uploadStrategy, async (req, res) => {
  const blobName = getBlobName(req.file.originalname);
  const stream = getStream(req.file.buffer);
  const containerClient = blobServiceClient.getContainerClient(containerName2);;
  const blockBlobClient = containerClient.getBlockBlobClient(blobName);

  try {
    await blockBlobClient.uploadStream(stream,
      uploadOptions.bufferSize, uploadOptions.maxBuffers,
      { blobHTTPHeaders: { blobContentType: "image/jpeg" } });
    res.render('success', { message: 'File uploaded to Azure Blob storage.' });
  } catch (err) {
    res.render('error', { message: err.message });
  }
});

module.exports = router;
```

---

## <a name="verify-the-image-is-shown-in-the-storage-account"></a>Vérifier que l’image s’affiche dans le compte de stockage

Connectez-vous au [portail Azure](https://portal.azure.com). Dans le menu de gauche, sélectionnez **Comptes de stockage**, puis sélectionnez le nom de votre compte de stockage. Sélectionnez **Conteneurs**, puis sélectionnez le conteneur **images**.

Vérifiez que l’image s’affiche dans le conteneur.

![Conteneur de liste d’images du portail Azure](media/storage-upload-process-images/figure13.png)

## <a name="test-thumbnail-viewing"></a>Tester l’affichage des miniatures

Pour tester l’affichage des miniatures, chargez une image dans le conteneur **Miniatures** afin de vérifier que l’application peut lire les **miniatures** du conteneur.

Connectez-vous au [portail Azure](https://portal.azure.com). Dans le menu de gauche, sélectionnez **Comptes de stockage**, puis sélectionnez le nom de votre compte de stockage. Sélectionnez **Conteneurs**, puis sélectionnez le conteneur **thumbnails** (miniatures). Cliquez sur **Charger** pour ouvrir le volet **Charger des objets Blob**.

Choisissez un fichier à l’aide du sélecteur de fichiers, puis sélectionnez **Charger**.

Revenez à votre application pour vérifier que l’image chargée dans le conteneur **thumbnails** est visible.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

![Application de redimensionnement d’image .NET avec la nouvelle image affichée](media/storage-upload-process-images/figure2.png)

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

![Application de redimensionnement d’image Node.js avec la nouvelle image affichée](media/storage-upload-process-images/upload-app-nodejs-thumb.png)

---

Dans la deuxième partie de la série, vous allez automatiser la création d’images miniatures. Vous n’avez donc pas besoin de cette image. Dans le conteneur **thumbnails**, sélectionnez l’image que vous avez chargée et sélectionnez **Supprimer** pour la supprimer.

Vous pouvez activer Content Delivery Network (CDN) pour mettre en cache le contenu du compte de stockage Azure. Pour plus d’informations, consultez [Intégrer un compte de stockage Azure à Azure CDN](../../cdn/cdn-create-a-storage-account-with-cdn.md).

## <a name="next-steps"></a>Étapes suivantes

Dans la première partie de la série, vous avez vu comment configurer une application web pour interagir avec le stockage.

Passez à la deuxième partie de la série pour savoir comment utiliser Event Grid dans le but de déclencher une fonction Azure afin de redimensionner une image.

> [!div class="nextstepaction"]
> [Utiliser Event Grid pour déclencher une fonction Azure afin de redimensionner une image chargée](../../event-grid/resize-images-on-storage-blob-upload-event.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
