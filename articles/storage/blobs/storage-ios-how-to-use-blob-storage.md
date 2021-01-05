---
title: Utilisation du stockage d’objets (blob) à partir d’iOS - Azure | Microsoft Docs
description: Stockez des données non structurées dans le cloud avec Azure Blob Storage (stockage d’objets).
author: mhopkins-msft
ms.author: mhopkins
ms.date: 11/20/2018
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.openlocfilehash: 7fc1b2638c2ab17c4cd58ca8d4508d2e6d244cfa
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95996805"
---
# <a name="how-to-use-blob-storage-from-ios"></a>Utilisation du stockage d’objets blob à partir d’iOS

Cet article décrit le déroulement de scénarios courants dans le cadre de l’utilisation du service de stockage d’objets blob Microsoft Azure. Les exemples ont été écrits en Objective-C et utilisent la [bibliothèque du client de stockage Azure pour iOS](https://github.com/Azure/azure-storage-ios). Les scénarios traités incluent le chargement, l'énumération, le téléchargement et la suppression d'objets blob. Pour plus d’informations sur les objets blob, voir la section [Étapes suivantes](#next-steps) . Vous pouvez également télécharger l’ [exemple d’application](https://github.com/Azure/azure-storage-ios/tree/master/BlobSample) pour voir rapidement l’utilisation d’Azure Storage dans une application iOS.

Pour en savoir plus sur le Stockage Blob, consultez [Présentation du Stockage Blob Azure](storage-blobs-introduction.md).

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="import-the-azure-storage-ios-library-into-your-application"></a>Importer la bibliothèque iOS Azure Storage dans votre application

Vous pouvez importer la bibliothèque iOS Azure Storage dans votre application en utilisant le [CocoaPod Azure Storage](https://cocoapods.org/pods/AZSClient) ou en important le fichier **Framework** . CocoaPod est la solution recommandée, car elle simplifie l’intégration de la bibliothèque. Toutefois, l’importation à partir du fichier de framework est moins intrusive pour votre projet existant.

Pour utiliser cette bibliothèque, les éléments suivants sont nécessaires :

- iOS 8+
- Xcode 7+

## <a name="cocoapod"></a>CocoaPod

1. Si ce n’est déjà fait, [installez des CocoaPods](https://guides.cocoapods.org/using/getting-started.html#toc_3) sur votre ordinateur en ouvrant une fenêtre de terminal et en exécutant la commande suivante

    ```shell
    sudo gem install cocoapods
    ```

2. Puis, dans le répertoire du projet (le répertoire contenant votre fichier .xcodeproj), créez un fichier nommé _Podfile_ (sans extension de fichier). Ajoutez l’élément suivant à _Podfile_ puis enregistrez.

    ```ruby
    platform :ios, '8.0'

    target 'TargetName' do
      pod 'AZSClient'
    end
    ```

3. Dans la fenêtre de terminal, accédez au répertoire du projet et exécutez la commande suivante

    ```shell
    pod install
    ```

4. Si votre fichier .xcodeproj est ouvert dans Xcode, fermez-le. Dans le répertoire de votre projet, ouvrez le fichier de projet nouvellement créé, qui aura l’extension .xcworkspace. C’est le fichier que vous allez utiliser à partir de maintenant.

## <a name="framework"></a>Framework

Pour utiliser la bibliothèque, vous pouvez aussi créer l’infrastructure manuellement :

1. Tout d’abord, téléchargez ou clonez le [référentiel azure-storage-ios](https://github.com/azure/azure-storage-ios).
2. Accédez à *azure-storage-ios* -> *Bibliothèque* -> *Bibliothèque cliente Azure Storage*, puis ouvrez `AZSClient.xcodeproj` dans Xcode.
3. Dans le coin supérieur gauche de Xcode, changez le schéma actif de « Bibliothèque cliente Azure Storage » à « Infrastructure ».
4. Créez le projet (⌘+B). Cela créera un fichier `AZSClient.framework` sur votre bureau.

Vous pouvez ensuite importer le fichier d’infrastructure dans votre application en procédant comme suit :

1. Créez un projet ou ouvrez votre projet existant dans Xcode.
2. Faites glisser le `AZSClient.framework` dans votre navigateur de projets Xcode.
3. Sélectionnez *Copier les éléments si besoin*, puis cliquez sur *Terminer*.
4. Cliquez sur votre projet dans la barre de navigation située sur la gauche, puis cliquez sur l’onglet *Général* en haut de l’éditeur de projets.
5. Dans la section *Bibliothèques et infrastructures liées* , cliquez sur le bouton Ajouter (+).
6. Dans la liste des bibliothèques déjà fournie, recherchez la bibliothèque `libxml2.2.tbd` et ajoutez-la à votre projet.

## <a name="import-the-library"></a>Importer la bibliothèque

```objc
// Include the following import statement to use blob APIs.
#import <AZSClient/AZSClient.h>
```

Si vous utilisez Swift, vous devez créer un en-tête de pontage et importer \<AZSClient/AZSClient.h> ici :

1. Créez un fichier d’en-tête `Bridging-Header.h`, puis ajoutez l’instruction import ci-dessus.
2. Accédez à l’onglet *Paramètre de génération* et recherchez *En-tête de pontage Objective-C*.
3. Double-cliquez sur le champ *En-tête de pontage Objective-C* et ajoutez le chemin d’accès à votre fichier d’en-tête :`ProjectName/Bridging-Header.h`
4. Créez le projet (⌘ + B) pour vérifier que l’en-tête de pontage a été récupéré par Xcode.
5. Vous pouvez alors utiliser la bibliothèque directement dans n’importe quel fichier Swift sans instructions d’importation.

[!INCLUDE [storage-mobile-authentication-guidance](../../../includes/storage-mobile-authentication-guidance.md)]

## <a name="asynchronous-operations"></a>Opérations asynchrones

> [!NOTE]
> Toutes les méthodes qui effectuent une demande auprès du service sont des opérations asynchrones. Dans les exemples de code, vous pouvez voir que ces méthodes ont un gestionnaire de complétion. Le code dans le gestionnaire d’achèvement s’exécutera **après** que la demande sera terminée. Le code suivant le gestionnaire d’achèvement s’exécute **pendant** l’exécution de la demande.

## <a name="create-a-container"></a>Créez un conteneur.

Chaque objet blob dans Azure Storage doit résider dans un conteneur. L’exemple suivant montre comment créer un conteneur appelé *newcontainer* dans votre compte de stockage s’il n’existe pas déjà. Lorsque vous choisissez un nom pour votre conteneur, gardez à l’esprit les règles d’affectation de noms mentionnées ci-dessus.

```objc
-(void)createContainer{
    NSError *accountCreationError;

    // Create a storage account object from a connection string.
    AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

    if(accountCreationError){
        NSLog(@"Error in creating account.");
    }

    // Create a blob service client object.
    AZSCloudBlobClient *blobClient = [account getBlobClient];

    // Create a local container object.
    AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"newcontainer"];

    // Create container in your Storage account if the container doesn't already exist
    [blobContainer createContainerIfNotExistsWithCompletionHandler:^(NSError *error, BOOL exists) {
        if (error){
            NSLog(@"Error in creating container.");
        }
    }];
}
```

Vous pouvez vérifier que cela fonctionne en examinant [l’Explorateur de stockage Microsoft Azure](https://storageexplorer.com) et en vérifiant que *newcontainer* figure dans la liste des conteneurs pour votre compte de stockage.

## <a name="set-container-permissions"></a>Définir les autorisations du conteneur

Les autorisations d’un conteneur sont configurées pour l’accès **Privé** par défaut. Toutefois, les conteneurs fournissent d’autres options pour l’accès aux conteneurs :

- **Privé** : seul le propriétaire du compte peut lire les données de conteneur et d’objet blob.
- **Objet blob** : Les données d'objets blob à l'intérieur de ce conteneur peuvent être lues via une demande anonyme, mais les données du conteneur ne sont pas disponibles. Les clients ne peuvent pas énumérer les objets blob à l'intérieur du conteneur via une demande anonyme.
- **Conteneur** : les données de conteneur et d’objets blob sont lisibles au moyen d’une requête anonyme. Les clients peuvent énumérer les objets blob à l’intérieur du conteneur via une demande anonyme, mais ne peuvent pas énumérer les conteneurs dans le compte de stockage.

L’exemple suivant montre comment créer un conteneur avec des autorisations d’accès de type **Conteneur** , qui autorisent un accès public en lecture seule à tous les utilisateurs sur Internet :

```objc
-(void)createContainerWithPublicAccess{
    NSError *accountCreationError;

    // Create a storage account object from a connection string.
    AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

    if(accountCreationError){
        NSLog(@"Error in creating account.");
    }

    // Create a blob service client object.
    AZSCloudBlobClient *blobClient = [account getBlobClient];

    // Create a local container object.
    AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

    // Create container in your Storage account if the container doesn't already exist
    [blobContainer createContainerIfNotExistsWithAccessType:AZSContainerPublicAccessTypeContainer requestOptions:nil operationContext:nil completionHandler:^(NSError *error, BOOL exists){
        if (error){
            NSLog(@"Error in creating container.");
        }
    }];
}
```

## <a name="upload-a-blob-into-a-container"></a>Charger un objet blob dans un conteneur

Comme mentionné dans la section Concepts du service BLOB, Stockage Blob offre trois types d’objet blob : les objets blob de blocs, les objets blob d’ajouts et les objets blob de pages. La bibliothèque iOS de stockage Azure prend en charge les trois types d’objets blob. En règle générale, il est recommandé d’utiliser le type d’objet blob de blocs.

L’exemple suivant montre comment charger un objet blob de blocs à partir d’une chaîne NSString. Si un objet blob portant le même nom existe déjà dans ce conteneur, le contenu de cet objet blob est remplacé.

```objc
-(void)uploadBlobToContainer{
    NSError *accountCreationError;

    // Create a storage account object from a connection string.
    AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

    if(accountCreationError){
        NSLog(@"Error in creating account.");
    }

    // Create a blob service client object.
    AZSCloudBlobClient *blobClient = [account getBlobClient];

    // Create a local container object.
    AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

    [blobContainer createContainerIfNotExistsWithAccessType:AZSContainerPublicAccessTypeContainer requestOptions:nil operationContext:nil completionHandler:^(NSError *error, BOOL exists)
        {
            if (error){
                NSLog(@"Error in creating container.");
            }
            else{
                // Create a local blob object
                AZSCloudBlockBlob *blockBlob = [blobContainer blockBlobReferenceFromName:@"sampleblob"];

                // Upload blob to Storage
                [blockBlob uploadFromText:@"This text will be uploaded to Blob Storage." completionHandler:^(NSError *error) {
                    if (error){
                        NSLog(@"Error in creating blob.");
                    }
                }];
            }
        }];
}
```

Vous pouvez vérifier que cela fonctionne en examinant [l’Explorateur de stockage Microsoft Azure](https://storageexplorer.com) et en vérifiant que le conteneur, *containerpublic*, contient l’objet blob, *sampleblob*. Dans cet exemple, nous avons utilisé un conteneur public. Ainsi vous pouvez vérifier que l’application a fonctionné en accédant à l’URI des objets blob :

```http
https://nameofyourstorageaccount.blob.core.windows.net/containerpublic/sampleblob
```

Outre le chargement d’un objet blob de blocs à partir d’une chaîne NSString, des méthodes similaires existent pour NSData, NSInputStream ou un fichier local.

## <a name="list-the-blobs-in-a-container"></a>Créer la liste des objets blob d’un conteneur

L’exemple suivant montre comment répertorier tous les objets blob dans un conteneur. Lorsque vous effectuez cette opération, gardez à l’esprit les paramètres suivants :

- **continuationToken** : le jeton de continuation représente l’emplacement où l’opération de liste doit commencer. Si aucun jeton n’est fourni, il répertorie les objets blob depuis le début. N’importe quel nombre d’objets blob peut être répertorié, à partir de zéro jusqu’à un maximum spécifié. Même si cette méthode ne retourne aucun résultat, si la valeur `results.continuationToken` n’est pas nulle, il peut y avoir davantage d’objets blob sur le service qui n’ont pas été répertoriés.
- **prefix** : vous pouvez spécifier le préfixe à utiliser pour la liste d’objets blob. Seuls les objets blob qui commencent par ce préfixe sont répertoriés.
- **useFlatBlobListing** : comme indiqué dans la section [Désignation et référencement des conteneurs et des objets blob](/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata) , bien que le service BLOB soit un schéma de stockage plat, vous pouvez créer une hiérarchie virtuelle en nommant les objets blob avec des informations de chemin d’accès. Toutefois, les listes de stockage non plat ne sont actuellement pas prises en charge. Cette fonctionnalité sera bientôt disponible. Pour le moment, cette valeur doit être **YES**.
- **blobListingDetails** : vous pouvez spécifier les éléments à inclure lors de la création de la liste d’objets blob.
  - _AZSBlobListingDetailsNone_ : liste uniquement les objets blob validés et ne retourne pas de métadonnées d’objets blob.
  - _AZSBlobListingDetailsSnapshots_ : liste les objets blob validés et les instantanés d’objets blob.
  - _AZSBlobListingDetailsMetadata_ : récupère les métadonnées d’objets blob pour chaque objet blob retourné dans la liste.
  - _AZSBlobListingDetailsUncommittedBlobs_ : liste les objets blob validés et non validés.
  - _AZSBlobListingDetailsCopy_ : inclut des propriétés de copie dans la liste.
  - _AZSBlobListingDetailsAll_ : liste tous les objets blob validés, les objets blob non validés et les instantanés disponibles, et retourne l’état de toutes les métadonnées et de la copie pour ces objets blob.
- **maxResults** : nombre maximal de résultats à renvoyer pour cette opération. Utilisez -1 pour ne pas définir une limite.
- **completionHandler** : bloc de code à exécuter avec les résultats de l’opération de génération de liste.

Dans cet exemple, une méthode d’assistance est utilisée pour appeler de manière récursive la méthode de liste d’objets blob à chaque fois qu’un jeton de continuation est renvoyé.

```objc
-(void)listBlobsInContainer{
    NSError *accountCreationError;

    // Create a storage account object from a connection string.
    AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

    if(accountCreationError){
        NSLog(@"Error in creating account.");
    }

    // Create a blob service client object.
    AZSCloudBlobClient *blobClient = [account getBlobClient];

    // Create a local container object.
    AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

    //List all blobs in container
    [self listBlobsInContainerHelper:blobContainer continuationToken:nil prefix:nil blobListingDetails:AZSBlobListingDetailsAll maxResults:-1 completionHandler:^(NSError *error) {
        if (error != nil){
            NSLog(@"Error in creating container.");
        }
    }];
}

//List blobs helper method
-(void)listBlobsInContainerHelper:(AZSCloudBlobContainer *)container continuationToken:(AZSContinuationToken *)continuationToken prefix:(NSString *)prefix blobListingDetails:(AZSBlobListingDetails)blobListingDetails maxResults:(NSUInteger)maxResults completionHandler:(void (^)(NSError *))completionHandler
{
    [container listBlobsSegmentedWithContinuationToken:continuationToken prefix:prefix useFlatBlobListing:YES blobListingDetails:blobListingDetails maxResults:maxResults completionHandler:^(NSError *error, AZSBlobResultSegment *results) {
        if (error)
        {
            completionHandler(error);
        }
        else
        {
            for (int i = 0; i < results.blobs.count; i++) {
                NSLog(@"%@",[(AZSCloudBlockBlob *)results.blobs[i] blobName]);
            }
            if (results.continuationToken)
            {
                [self listBlobsInContainerHelper:container continuationToken:results.continuationToken prefix:prefix blobListingDetails:blobListingDetails maxResults:maxResults completionHandler:completionHandler];
            }
            else
            {
                completionHandler(nil);
            }
        }
    }];
}
```

## <a name="download-a-blob"></a>Télécharger un objet blob

L’exemple suivant montre comment télécharger un objet blob vers un objet NSString.

```objc
-(void)downloadBlobToString{
    NSError *accountCreationError;

    // Create a storage account object from a connection string.
    AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

    if(accountCreationError){
        NSLog(@"Error in creating account.");
    }

    // Create a blob service client object.
    AZSCloudBlobClient *blobClient = [account getBlobClient];

    // Create a local container object.
    AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

    // Create a local blob object
    AZSCloudBlockBlob *blockBlob = [blobContainer blockBlobReferenceFromName:@"sampleblob"];

    // Download blob
    [blockBlob downloadToTextWithCompletionHandler:^(NSError *error, NSString *text) {
        if (error) {
            NSLog(@"Error in downloading blob");
        }
        else{
            NSLog(@"%@",text);
        }
    }];
}
```

## <a name="delete-a-blob"></a>Supprimer un objet blob

L’exemple suivant indique comment supprimer un objet blob.

```objc
-(void)deleteBlob{
    NSError *accountCreationError;

    // Create a storage account object from a connection string.
    AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

    if(accountCreationError){
        NSLog(@"Error in creating account.");
    }

    // Create a blob service client object.
    AZSCloudBlobClient *blobClient = [account getBlobClient];

    // Create a local container object.
    AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

    // Create a local blob object
    AZSCloudBlockBlob *blockBlob = [blobContainer blockBlobReferenceFromName:@"sampleblob1"];

    // Delete blob
    [blockBlob deleteWithCompletionHandler:^(NSError *error) {
        if (error) {
            NSLog(@"Error in deleting blob.");
        }
    }];
}
```

## <a name="delete-a-blob-container"></a>Suppression d’un conteneur d’objets blob

L’exemple suivant indique comment supprimer un conteneur.

```objc
-(void)deleteContainer{
    NSError *accountCreationError;

    // Create a storage account object from a connection string.
    AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

    if(accountCreationError){
        NSLog(@"Error in creating account.");
    }

    // Create a blob service client object.
    AZSCloudBlobClient *blobClient = [account getBlobClient];

    // Create a local container object.
    AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

    // Delete container
    [blobContainer deleteContainerIfExistsWithCompletionHandler:^(NSError *error, BOOL success) {
        if(error){
            NSLog(@"Error in deleting container");
        }
    }];
}
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à utiliser le stockage d’objets blob d’iOS, suivez ces liens pour en savoir plus sur la bibliothèque iOS et le service de stockage.

- [Bibliothèque cliente d’Azure Storage pour iOS](https://github.com/azure/azure-storage-ios)
- [Documentation de référence d’Azure Storage pour iOS](https://azure.github.io/azure-storage-ios/)
- [API REST des services d’Azure Storage](/rest/api/storageservices/)
- [Blog de l'équipe Azure Storage](/archive/blogs/windowsazurestorage/)

Si vous avez des questions concernant cette bibliothèque, n’hésitez pas à les publier sur notre [Page de questions Microsoft Q&R](/answers/topics/azure-blob-storage.html) ou sur [Stack Overflow](https://stackoverflow.com/questions/tagged/windows-azure-storage+or+windows-azure-storage+or+azure-storage-blobs+or+azure-storage-tables+or+azure-table-storage+or+windows-azure-queues+or+azure-storage-queues+or+azure-storage-emulator+or+azure-storage-files).
Si vous avez des suggestions de fonctionnalités pour le stockage Azure, envoyez un message sur [Commentaires sur le stockage Azure](https://feedback.azure.com/forums/217298-storage/).