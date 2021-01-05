---
title: 'Démarrage rapide : Bibliothèque de stockage d’objets Blob Azure v12 – Xamarin'
description: Dans ce guide de démarrage rapide, vous apprenez à utiliser la bibliothèque cliente Stockage Blob Azure version 12 avec Xamarin afin de créer un conteneur et un objet blob dans le stockage (d’objets) blob. Vous apprenez ensuite à télécharger l’objet blob sur votre appareil mobile et à lister tous les objets blob dans un conteneur.
author: codemillmatt
ms.author: masoucou
ms.date: 10/09/2020
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: 641117414c855a044d3ed55004b6e6ea15b259ac
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96004736"
---
# <a name="quickstart-azure-blob-storage-client-library-v12-with-xamarin"></a>Démarrage rapide : Bibliothèque de client Stockage Blob Azure v12 avec Xamarin

Bien démarrer avec la bibliothèque de client Azure Storage Blob v12 avec Xamarin. Le stockage Blob Azure est la solution de stockage d’objet de Microsoft pour le cloud. Suivez les étapes pour installer le package et essayer l’exemple de code pour les tâches de base. Le stockage Blob est optimisé pour stocker de grandes quantités de données non structurées.

Utilisez la bibliothèque de client Azure Storage Blob v12 avec Xamarin pour :

* Créez un conteneur.
* Charger un blob dans le stockage Azure
* Lister tous les objets blob d’un conteneur
* Télécharger l’objet blob sur votre appareil
* Supprimer un conteneur

Liens de référence :

* [Documentation de référence de l’API](/dotnet/api/azure.storage.blobs)
* [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Blobs)
* [Package (NuGet)](https://www.nuget.org/packages/Azure.Storage.Blobs)
* [Exemple](https://github.com/Azure-Samples/storage-blobs-xamarin-quickstart)

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/)
* Compte de stockage Azure : [créez un compte de stockage](../common/storage-account-create.md)
* Visual Studio avec [Développement mobile pour la charge de travail .NET](/xamarin/get-started/installation/?pivots=windows) installé ou [Visual Studio pour Mac](/visualstudio/mac/installation?view=vsmac-2019)

## <a name="setting-up"></a>Configuration
    
Cette section vous guide tout au long de la préparation d’un projet à utiliser avec la bibliothèque de client Azure Storage Blob v12 avec Xamarin.
    
### <a name="create-the-project"></a>Créer le projet

1. Ouvrez Visual Studio et créez une application Forms vide.
1. Nommez-la : BlobQuickstartV12

### <a name="install-the-package"></a>Installer le package

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur votre solution, puis sélectionnez **Gérer les packages NuGet pour la solution**.
1. Recherchez **Azure.Storage.Blobs** et installez la dernière version disponible sur tous les projets dans votre solution.

### <a name="set-up-the-app-framework"></a>Configurer le framework d’application

À partir du répertoire **BlobQuickstartV12** :

1. Ouvrez le fichier *MainPage.xaml* dans votre éditeur
1. Supprimez tout ce qui se trouve entre les éléments `<ContentPage></ContentPage>` et remplacez avec ce qui suit :

```xaml
<StackLayout HorizontalOptions="Center" VerticalOptions="Center">

    <Button x:Name="uploadButton" Text="Upload Blob" Clicked="Upload_Clicked"  IsEnabled="False"/>
    <Button x:Name="listButton" Text="List Blobs" Clicked="List_Clicked"  IsEnabled="False" />
    <Button x:Name="downloadButton" Text="Download Blob" Clicked="Download_Clicked"  IsEnabled="False" />
    <Button x:Name="deleteButton" Text="Delete Container" Clicked="Delete_Clicked" IsEnabled="False" />

    <Label Text="" x:Name="resultsLabel" HorizontalTextAlignment="Center" Margin="0,20,0,0" TextColor="Red" />
        
</StackLayout>
```

[!INCLUDE [storage-quickstart-credentials-xamarin-include](../../../includes/storage-quickstart-credentials-xamarin-include.md)]

## <a name="object-model"></a>Modèle objet

Le Stockage Blob Azure est optimisé pour stocker de grandes quantités de données non structurées. Les données non structurées sont des données qui n’obéissent pas à un modèle ou une définition de données en particulier, comme des données texte ou binaires. Le stockage Blob offre trois types de ressources :

* Le compte de stockage
* Un conteneur dans le compte de stockage.
* Un blob dans le conteneur

Le diagramme suivant montre la relation entre ces ressources.

![Diagramme de l’architecture du stockage Blob](./media/storage-blobs-introduction/blob1.png)

Utilisez les classes .NET suivantes pour interagir avec ces ressources :

* [BlobServiceClient](/dotnet/api/azure.storage.blobs.blobserviceclient): La classe `BlobServiceClient` vous permet de manipuler les ressources de stockage Azure et les conteneurs blob.
* [BlobContainerClient](/dotnet/api/azure.storage.blobs.blobcontainerclient) : La classe `BlobContainerClient` vous permet de manipuler des conteneurs de stockage Azure et leurs blobs.
* [BlobClient](/dotnet/api/azure.storage.blobs.blobclient) : La classe `BlobClient` vous permet de manipuler des blobs de stockage Azure.
* [BlobDownloadInfo](/dotnet/api/azure.storage.blobs.models.blobdownloadinfo) : La classe `BlobDownloadInfo` représente les propriétés et le contenu renvoyés par le téléchargement d’un objet Blob.

## <a name="code-examples"></a>Exemples de code

Ces exemples d’extraits de code vous montrent comment effectuer les tâches suivantes avec la bibliothèque cliente Stockage Blob Azure pour .NET dans une application Xamarin.Forms :

* [Créer des variables au niveau de la classe](#create-class-level-variables)
* [Créer un conteneur](#create-a-container)
* [Charger des objets blob sur un conteneur](#upload-blobs-to-a-container)
* [Lister les objets blob d’un conteneur](#list-the-blobs-in-a-container)
* [Télécharger des objets blob](#download-blobs)
* [Supprimer un conteneur](#delete-a-container)

### <a name="create-class-level-variables"></a>Créer des variables au niveau de la classe

Le code ci-dessous déclare plusieurs variables au niveau de la classe. Elles sont nécessaires pour communiquer avec le stockage d’objets BLOB Azure dans le reste de cet exemple.

Celles-ci s’ajoutent à la chaîne de connexion pour le compte de stockage défini dans la section [Configurer votre chaîne de connexion au stockage](#configure-your-storage-connection-string).

Ajoutez ce code en tant que variables au niveau de la classe dans le fichier *MainPage.xaml.cs* :

```csharp
string storageConnectionString = "{set in the Configure your storage connection string section}";
string fileName = $"{Guid.NewGuid()}-temp.txt";

BlobServiceClient client;
BlobContainerClient containerClient;
BlobClient blobClient;
```

### <a name="create-a-container"></a>Créez un conteneur.

Choisissez un nom pour le nouveau conteneur. Le code ci-dessous ajoute une valeur GUID au nom du conteneur pour s’assurer qu’il est unique.

> [!IMPORTANT]
> Les noms de conteneurs doivent être en minuscules. Pour plus d’informations sur l’affectation de noms aux conteneurs et objets blob, consultez [Affectation de noms et références aux conteneurs, objets blob et métadonnées](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata).

Création d’une instance de la classe [BlobServiceClient](/dotnet/api/azure.storage.blobs.blobserviceclient). Ensuite, appelez la méthode [CreateBlobContainerAsync](/dotnet/api/azure.storage.blobs.blobserviceclient.createblobcontainerasync) pour créer le conteneur dans votre compte de stockage.

Ajoutez ce code dans le fichier *MainPage.xaml.cs* :

```csharp
protected async override void OnAppearing()
{            
    string containerName = $"quickstartblobs{Guid.NewGuid()}";
    
    client = new BlobServiceClient(storageConnectionString);
    containerClient = await client.CreateBlobContainerAsync(containerName);

    resultsLabel.Text = "Container Created\n";

    blobClient = containerClient.GetBlobClient(fileName);

    uploadButton.IsEnabled = true;
}
```

### <a name="upload-blobs-to-a-container"></a>Charger des objets blob sur un conteneur

L’extrait de code suivant :

1. Crée un `MemoryStream` de texte.
1. Charge le texte dans un objet blob en appelant la fonction [UploadAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.uploadblobasync#Azure_Storage_Blobs_BlobContainerClient_UploadBlobAsync_System_String_System_IO_Stream_System_Threading_CancellationToken_) de la classe [BlobContainerClient](/dotnet/api/azure.storage.blobs.blobcontainerclient), en lui transmettant à la fois le nom de fichier et le `MemoryStream` de texte. Cette méthode crée l’objet blob s’il n’existe pas déjà, et le remplace s’il existe.

Ajoutez ce code au fichier *MainPage.xaml.cs* :

```csharp
async void Upload_Clicked(object sender, EventArgs e)
{                                    
    using MemoryStream memoryStream = new MemoryStream(Encoding.UTF8.GetBytes("Hello World!"));

    await containerClient.UploadBlobAsync(fileName, memoryStream);

    resultsLabel.Text += "Blob Uploaded\n";

    uploadButton.IsEnabled = false;
    listButton.IsEnabled = true;
}
```

### <a name="list-the-blobs-in-a-container"></a>Créer la liste des objets blob d’un conteneur

Répertoriez les objets Blob dans le conteneur en appelant la méthode [GetBlobsAsync](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsasync). Dans ce cas, un seul objet blob a été ajouté au conteneur. Il n’y a donc qu’un objet blob répertorié.

Ajoutez ce code au fichier *MainPage.xaml.cs* :

```csharp
async void List_Clicked(object sender, EventArgs e)
{            
    await foreach (BlobItem blobItem in containerClient.GetBlobsAsync())
    {
        resultsLabel.Text += blobItem.Name + "\n";                
    }

    listButton.IsEnabled = false;
    downloadButton.IsEnabled = true;
}
```

### <a name="download-blobs"></a>Télécharger des objets blob

Téléchargez l’objet Blob créé précédemment en appelant la méthode [DownloadAsync](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.downloadasync). L’exemple de code copie d’abord la représentation `Stream` de l’objet BLOB dans un `MemoryStream`, puis dans un `StreamReader` afin que le texte puisse être affiché.

Ajoutez ce code au fichier *MainPage.xaml.cs* :

```csharp
async void Download_Clicked(object sender, EventArgs e)
{
    BlobDownloadInfo downloadInfo = await blobClient.DownloadAsync();

    using MemoryStream memoryStream = new MemoryStream();
    
    await downloadInfo.Content.CopyToAsync(memoryStream);
    memoryStream.Position = 0;

    using StreamReader streamReader = new StreamReader(memoryStream);

    resultsLabel.Text += "Blob Contents: \n";
    resultsLabel.Text += await streamReader.ReadToEndAsync();
    resultsLabel.Text += "\n";

    downloadButton.IsEnabled = false;
    deleteButton.IsEnabled = true;
}
```

### <a name="delete-a-container"></a>Supprimer un conteneur

Le code suivant nettoie les ressources créées par l’application en supprimant l’ensemble du conteneur avec [DeleteAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.deleteasync).

L’application vous invite d’abord à confirmer avant de supprimer l’objet BLOB et le conteneur. C’est l’occasion de vérifier que les ressources ont été créées correctement, avant d’être supprimées.

Ajoutez ce code au fichier *MainPage.xaml.cs* :

```csharp
async void Delete_Clicked(object sender, EventArgs e)
{            
    var deleteContainer = await Application.Current.MainPage.DisplayAlert("Delete Container",
        "You are about to delete the container proceeed?", "OK", "Cancel");

    if (deleteContainer == false)
        return;

    await containerClient.DeleteAsync();

    resultsLabel.Text += "Container Deleted";

    deleteButton.IsEnabled = false;
}
```

## <a name="run-the-code"></a>Exécuter le code

Lorsque l’application démarre, elle crée d’abord le conteneur tel qu’il apparaît. Ensuite, vous devrez cliquer sur les boutons pour télécharger, répertorier, télécharger les objets BLOB et supprimer le conteneur.

Pour exécuter l’application sur Windows, appuyez sur F5. Pour exécuter l’application sur Mac, appuyez sur Cmd + Entrée.

L’application écrit à l’écran après chaque opération. La sortie de l’application ressemble à l’exemple ci-dessous :

```output
Container Created
Blob Uploaded
98d9a472-8e98-4978-ba4f-081d69d2e6f8-temp.txt
Blob Contents:
Hello World!
Container Deleted
```

Avant de commencer le processus de nettoyage, vérifiez que la sortie du contenu de l’objet blob à l’écran correspond à la valeur qui a été chargée.

Une fois que vous avez vérifié les valeurs, confirmez l’invite de suppression du conteneur et terminez la démonstration.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à télécharger et répertorier des objets BLOB à l’aide de la bibliothèque cliente de stockage BLOB Azure V12 avec Xamarin.

Pour afficher des exemples d’applications de stockage blob, passez à :

> [!div class="nextstepaction"]
> [Exemple Xamarin du kit de développement logiciel (SDK) du stockage d’objets Blob Azure v12](https://github.com/Azure-Samples/storage-blobs-xamarin-quickstart)

* Pour obtenir des didacticiels, des exemples, des démarrages rapides et d’autres documents, visitez [Azure pour les développeurs mobiles](/azure/mobile-apps).
* Pour en savoir plus sur Xamarin, consultez [Prise en main de Xamarin](/xamarin/get-started/).