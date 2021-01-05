---
title: Sécuriser l’accès aux données d’application
titleSuffix: Azure Storage
description: Utilisez les jetons SAS, le chiffrement et HTTPS pour sécuriser les données de votre application dans le cloud.
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.topic: tutorial
ms.date: 06/10/2020
ms.author: tamram
ms.reviewer: ozgun
ms.custom: mvc, devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: f7c5dbaf30965fdd5f438f0351cfa2cd60e05b70
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92746562"
---
# <a name="secure-access-to-application-data"></a>Sécuriser l’accès aux données d’application

Ce tutoriel est le troisième de la série. Vous apprenez à sécuriser l’accès au compte de stockage. 

Dans ce troisième volet, vous apprenez à :

> [!div class="checklist"]
> * Utiliser des jetons SAS pour accéder à des images miniatures
> * Activer le chiffrement côté serveur
> * Activer le transport HTTPS uniquement

Le [stockage Blob Azure](../common/storage-introduction.md#blob-storage) fournit un service fiable pour stocker les fichiers des applications. Ce tutoriel est la suite de [la rubrique précédente][previous-tutorial] et décrit comment sécuriser l’accès à votre compte de stockage à partir d’une application web. À la fin de ce didacticiel, les images sont chiffrées et l’application web utilise des jetons SAS sécurisés pour accéder aux images miniatures.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vous devez avoir terminé le tutoriel précédent sur le stockage : [Automatiser le redimensionnement des images chargées à l’aide d’Event Grid][previous-tutorial].

## <a name="set-container-public-access"></a>Définir un conteneur d’accès public

Dans ce volet de la série de didacticiels, les jetons SAS sont utilisés pour accéder aux miniatures. Dans cette étape, vous définissez l’accès public du conteneur de *miniatures* sur `off`.

```bash
blobStorageAccount="<blob_storage_account>"

blobStorageAccountKey=$(az storage account keys list -g myResourceGroup \
    --account-name $blobStorageAccount --query [0].value --output tsv) 

az storage container set-permission \
    --account-name $blobStorageAccount \
    --account-key $blobStorageAccountKey \
    --name thumbnails \
    --public-access off
```

```powershell
$blobStorageAccount="<blob_storage_account>"

blobStorageAccountKey=$(az storage account keys list -g myResourceGroup `
    --account-name $blobStorageAccount --query [0].value --output tsv) 

az storage container set-permission `
    --account-name $blobStorageAccount `
    --account-key $blobStorageAccountKey `
    --name thumbnails `
    --public-access off
```

## <a name="configure-sas-tokens-for-thumbnails"></a>Configurer des jetons SAS pour les miniatures

Dans le premier volet de cette série de didacticiels, l’application web affichait des images à partir d’un conteneur public. Dans ce volet de la série, vous utilisez des jetons de signatures d’accès partagé (SAS) pour récupérer les images miniatures. Les jetons SAS vous permettent de fournir un accès limité à un conteneur ou un objet blob en fonction de l’adresse IP, du protocole, de l’intervalle de temps ou des droits autorisés. Pour plus d’informations sur les SAS, consultez [Accorder un accès limité aux ressources du Stockage Azure à l’aide des signatures d’accès partagé (SAS)](../common/storage-sas-overview.md).

Dans cet exemple, le dépôt de code source utilise la branche `sasTokens`, qui a un exemple de code mis à jour. Supprimez le déploiement GitHub existant avec la commande [az webapp deployment source delete](/cli/azure/webapp/deployment/source). Ensuite, configurez un déploiement GitHub local sur l’application web avec la commande [az webapp deployment source config](/cli/azure/webapp/deployment/source).

Dans la commande suivante, `<web-app>` est le nom de votre application web.

```bash
az webapp deployment source delete --name <web-app> --resource-group myResourceGroup

az webapp deployment source config --name <web_app> \
    --resource-group myResourceGroup --branch sasTokens --manual-integration \
    --repo-url https://github.com/Azure-Samples/storage-blob-upload-from-webapp
```

```powershell
az webapp deployment source delete --name <web-app> --resource-group myResourceGroup

az webapp deployment source config --name <web_app> `
    --resource-group myResourceGroup --branch sasTokens --manual-integration `
    --repo-url https://github.com/Azure-Samples/storage-blob-upload-from-webapp
```

La branche `sasTokens` du dépôt met à jour le fichier `StorageHelper.cs`. Elle remplace la tâche `GetThumbNailUrls` par l’exemple de code ci-dessous. La tâche mise à jour récupère les URL de miniature en utilisant un [BlobSasBuilder](/dotnet/api/azure.storage.sas.blobsasbuilder) pour spécifier l’heure de début, l’heure d’expiration et des autorisations pour le jeton SAS. Après le déploiement, l’application web récupère les miniatures avec une URL à l’aide d’un jeton SAS. La tâche mise à jour est illustrée dans l’exemple suivant :

```csharp
public static async Task<List<string>> GetThumbNailUrls(AzureStorageConfig _storageConfig)
{
    List<string> thumbnailUrls = new List<string>();

    // Create a URI to the storage account
    Uri accountUri = new Uri("https://" + _storageConfig.AccountName + ".blob.core.windows.net/");

    // Create BlobServiceClient from the account URI
    BlobServiceClient blobServiceClient = new BlobServiceClient(accountUri);

    // Get reference to the container
    BlobContainerClient container = blobServiceClient.GetBlobContainerClient(_storageConfig.ThumbnailContainer);

    if (container.Exists())
    {
        // Set the expiration time and permissions for the container.
        // In this case, the start time is specified as a few 
        // minutes in the past, to mitigate clock skew.
        // The shared access signature will be valid immediately.
        BlobSasBuilder sas = new BlobSasBuilder
        {
            Resource = "c",
            BlobContainerName = _storageConfig.ThumbnailContainer,
            StartsOn = DateTimeOffset.UtcNow.AddMinutes(-5),
            ExpiresOn = DateTimeOffset.UtcNow.AddHours(1)
        };

        sas.SetPermissions(BlobContainerSasPermissions.All);

        // Create StorageSharedKeyCredentials object by reading
        // the values from the configuration (appsettings.json)
        StorageSharedKeyCredential storageCredential =
            new StorageSharedKeyCredential(_storageConfig.AccountName, _storageConfig.AccountKey);

        // Create a SAS URI to the storage account
        UriBuilder sasUri = new UriBuilder(accountUri);
        sasUri.Query = sas.ToSasQueryParameters(storageCredential).ToString();

        foreach (BlobItem blob in container.GetBlobs())
        {
            // Create the URI using the SAS query token.
            string sasBlobUri = container.Uri + "/" +
                                blob.Name + sasUri.Query;

            //Return the URI string for the container, including the SAS token.
            thumbnailUrls.Add(sasBlobUri);
        }
    }
    return await Task.FromResult(thumbnailUrls);
}
```

Les classes, propriétés et méthodes suivantes sont utilisées dans la tâche précédente :

| Classe | Propriétés | Méthodes |
|-------|------------|---------|
|[StorageSharedKeyCredential](/dotnet/api/azure.storage.storagesharedkeycredential) |  |  |
|[BlobServiceClient](/dotnet/api/azure.storage.blobs.blobserviceclient) |  |[GetBlobContainerClient](/dotnet/api/azure.storage.blobs.blobserviceclient.getblobcontainerclient) |
|[BlobContainerClient](/dotnet/api/azure.storage.blobs.blobcontainerclient) | [Uri](/dotnet/api/azure.storage.blobs.blobcontainerclient.uri) |[Exists](/dotnet/api/azure.storage.blobs.blobcontainerclient.exists) <br> [GetBlobs](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobs) |
|[BlobSasBuilder](/dotnet/api/azure.storage.sas.blobsasbuilder) |  | [SetPermissions](/dotnet/api/azure.storage.sas.blobsasbuilder.setpermissions) <br> [ToSasQueryParameters](/dotnet/api/azure.storage.sas.blobsasbuilder.tosasqueryparameters) |
|[BlobItem](/dotnet/api/azure.storage.blobs.models.blobitem) | [Nom](/dotnet/api/azure.storage.blobs.models.blobitem.name) |  |
|[UriBuilder](/dotnet/api/system.uribuilder) | [Requête](/dotnet/api/system.uribuilder.query) |  |
|[Liste](/dotnet/api/system.collections.generic.list-1) | | [Ajouter](/dotnet/api/system.collections.generic.list-1.add) |

## <a name="azure-storage-encryption"></a>Chiffrement du stockage Azure

[Le chiffrement du stockage Azure](../common/storage-service-encryption.md) vous permet de protéger et de préserver vos données en chiffrant les données au repos et en gérant le chiffrement et le déchiffrement. Toutes les données sont chiffrées à l’aide du [chiffrement AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)256 bits, l’un des algorithmes de chiffrement par blocs les plus puissants disponibles.

Vous pouvez choisir de laisser la gestion des clés de chiffrement à Microsoft, ou vous pouvez apporter vos propres clés, c’est-à-dire les clés gérées par le client qui sont stockées dans Azure Key Vault ou dans Key Vault Managed Hardware Security Model (HSM) (préversion). Pour plus d’informations, consultez [Clés gérées par le client pour le chiffrement Azure Storage](../common/customer-managed-keys-overview.md).

Le chiffrement du stockage Azure chiffre automatiquement les données pour tous les niveaux de performance (Standard ou Premium), tous les modèles de déploiement (Azure Resource Manager et Classic) et tous les services de stockage Azure (blob, file d’attente, table et fichier).

## <a name="enable-https-only"></a>Activer HTTPS uniquement

Pour vérifier que les demandes de données vers et à partir d’un compte de stockage sont sécurisées, vous pouvez limiter les demandes à HTTPS uniquement. Mettez à jour le protocole demandé par le compte de stockage à l’aide de la commande [az storage account update](/cli/azure/storage/account).

```azurecli-interactive
az storage account update --resource-group myresourcegroup --name <storage-account-name> --https-only true
```

Testez la connexion à l’aide de `curl` avec le protocole `HTTP`.

```azurecli-interactive
curl http://<storage-account-name>.blob.core.windows.net/<container>/<blob-name> -I
```

Maintenant que le transfert sécurisé est obligatoire, vous recevez le message d’erreur suivant :

```
HTTP/1.1 400 The account being accessed does not support http.
```

## <a name="next-steps"></a>Étapes suivantes

Dans le troisième volet de la série, vous avez appris à sécuriser l’accès au compte de stockage, notamment :

> [!div class="checklist"]
> * Utiliser des jetons SAS pour accéder à des images miniatures
> * Activer le chiffrement côté serveur
> * Activer le transport HTTPS uniquement

Passer au quatrième volet de la série pour apprendre à surveiller et résoudre les problèmes d’une application de stockage cloud.

> [!div class="nextstepaction"]
> [Surveiller et résoudre les problèmes de stockage d’une application cloud](storage-monitor-troubleshoot-storage-application.md)

[previous-tutorial]: ../../event-grid/resize-images-on-storage-blob-upload-event.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json
