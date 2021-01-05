---
title: Signaler des pistes audio descriptives avec Azure Media Services V3 | Microsoft Docs
description: Suivez les étapes décrites dans ce tutoriel pour charger un fichier, encoder la vidéo, ajouter des pistes audio descriptives, puis diffuser en continu votre contenu avec Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.custom: devx-track-csharp
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 0bd2002e8f6f233361dd85cc08cfbd97ca23ed60
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89291260"
---
# <a name="signal-descriptive-audio-tracks"></a>Signaler des pistes audio descriptives

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Vous pouvez ajouter une piste de narration à votre vidéo pour aider les clients malvoyants à suivre l’enregistrement vidéo en écoutant la narration. Dans Media Services V3, vous signalez des pistes audio descriptives en annotant la piste audio dans le fichier manifeste.

Cet article explique comment encoder une vidéo, charger un fichier MP4 audio uniquement (codec AAC) contenant des données audio descriptives dans la ressource de sortie, puis modifier le fichier .ism pour inclure l’audio descriptif.

## <a name="prerequisites"></a>Prérequis

- [Créer un compte Media Services](./create-account-howto.md).
- Suivez les étapes décrites dans [Accéder à l’API Azure Media Services avec Azure CLI](./access-api-howto.md) et enregistrez les informations d’identification. Vous en aurez besoin pour accéder à l’API.
- Passez en revue l’[empaquetage dynamique](dynamic-packaging-overview.md).
- Passez en revue le didacticiel [Charger, encoder et diffuser en continu des vidéos](stream-files-tutorial-with-api.md).

## <a name="create-an-input-asset-and-upload-a-local-file-into-it"></a>Créer une ressource d’entrée et charger un fichier local dans celle-ci 

La fonction **CreateInputAsset** crée une nouvelle [ressource](/rest/api/media/assets) d’entrée et charge le fichier vidéo local spécifié dans celle-ci. Cet **actif multimédia** est utilisé comme entrée de votre travail d’encodage. Dans Media Services v3, l’entrée d’un **travail** peut être un **actif multimédia** ou du contenu que vous mettez à la disposition de votre compte Media Services par le biais d’URL HTTPS. 

Si vous souhaitez savoir comment encoder à partir d’une URL HTTPS, consultez [cet article](job-input-from-http-how-to.md).  

Dans Media Services v3, vous utilisez des API Stockage Azure pour charger des fichiers. L’extrait de code .NET suivant vous explique comment faire.

La fonction suivante effectue les actions ci-après :

* Crée un **actif multimédia** 
* Obtient une [URL SAS](../../storage/common/storage-sas-overview.md) accessible en écriture vers le [conteneur de stockage](../../storage/blobs/storage-quickstart-blobs-dotnet.md#upload-blobs-to-a-container) de l’actif multimédia
* Elle charge le fichier dans le conteneur de stockage à l’aide de l’URL SAP

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateInputAsset)]

Si vous avez besoin de transmettre le nom de la ressource d’entrée créée à d’autres méthodes, veillez à utiliser la propriété `Name` sur l’objet de ressource retourné à partir de `CreateInputAssetAsync`, par exemple, inputAsset.Name. 

## <a name="create-an-output-asset-to-store-the-result-of-the-encoding-job"></a>Créer une ressource de sortie pour stocker le résultat du travail d’encodage

La [ressource](/rest/api/media/assets) de sortie stocke le résultat de votre travail d’encodage. La fonction suivante montre comment créer une ressource de sortie.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateOutputAsset)]

Si vous avez besoin de transmettre la ressource de sortie créée à d’autres méthodes, veillez à utiliser la propriété `Name` sur l’objet de ressource retourné à partir de `CreateIOutputAssetAsync`, par exemple, outputAsset.Name. 

Pour les besoins de cet article, transmettez la valeur `outputAsset.Name` aux fonctions `SubmitJobAsync` et `UploadAudioIntoOutputAsset`.

## <a name="create-a-transform-and-a-job-that-encodes-the-uploaded-file"></a>Créer une transformation et un travail qui encode le fichier chargé

Lors de l’encodage ou du traitement de contenus dans Media Services, il est courant de configurer les paramètres de codage en tant que formule. Vous envoyez ensuite un **travail** pour appliquer cette formule à une vidéo. En envoyant de nouveaux travaux pour chaque nouvelle vidéo, vous appliquez cette formule à toutes les vidéos de votre bibliothèque. Une formule dans Media Services est référencée comme une **transformation**. Pour plus d’informations, consultez [Transformations et travaux](./transforms-jobs-concept.md). L’exemple décrit dans ce didacticiel définit une formule qui encode la vidéo, afin de la diffuser en continu sur divers appareils iOS et Android. 

L’exemple suivant crée une transformation (le cas échéant).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#EnsureTransformExists)]

La fonction suivante envoie un travail.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#SubmitJob)]

## <a name="wait-for-the-job-to-complete"></a>Attendre la fin du travail

Le travail prend du temps à se terminer et vous voulez être prévenu lorsque c’est le cas. Nous vous recommandons d’utiliser Event Grid pour attendre la fin du travail.

Le travail passe généralement par les états suivants : **Planifié**, **En attente**, **Traitement en cours**, **Terminé** (l’état final). Si le travail a rencontré une erreur, vous obtenez l’état **Erreur**. Si le travail est en cours d’annulation, vous obtenez **Annulation en cours** et **Annulé** une fois l’opération terminée.

Pour plus d’informations, consultez [Traitement des événements Event Grid](reacting-to-media-services-events.md).

## <a name="upload-the-audio-only-mp4-file"></a>Charger le fichier MP4 audio uniquement

Chargez le fichier MP4 audio uniquement supplémentaire (codec AAC) contenant des données audio descriptives dans la ressource de sortie.  

```csharp
private static async Task UpoadAudioIntoOutputAsset(
    IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string outputAssetName,
    string fileToUpload)
{
    // Use the Assets.Get method to get the existing asset. 
    // In Media Services v3, the Get method on entities returns null 
    // if the entity doesn't exist (a case-insensitive check on the name).

    // Call Media Services API to create an Asset.
    // This method creates a container in storage for the Asset.
    // The files (blobs) associated with the asset will be stored in this container.
    Asset asset = await client.Assets.GetAsync(resourceGroupName, accountName, outputAssetName);
    
    if (asset != null)
    {
      // Use Media Services API to get back a response that contains
      // SAS URL for the Asset container into which to upload blobs.
      // That is where you would specify read-write permissions 
      // and the exparation time for the SAS URL.
      var response = await client.Assets.ListContainerSasAsync(
          resourceGroupName,
          accountName,
          outputAssetName,
          permissions: AssetContainerPermission.ReadWrite,
          expiryTime: DateTime.UtcNow.AddHours(4).ToUniversalTime());

      var sasUri = new Uri(response.AssetContainerSasUrls.First());

      // Use Storage API to get a reference to the Asset container
      // that was created by calling Asset's CreateOrUpdate method.  
      CloudBlobContainer container = new CloudBlobContainer(sasUri);
      var blob = container.GetBlockBlobReference(Path.GetFileName(fileToUpload));

      // Use Strorage API to upload the file into the container in storage.
      await blob.UploadFromFileAsync(fileToUpload);
    }
}
```

Voici un exemple d’appel à la fonction `UpoadAudioIntoOutputAsset` :

```csharp
await UpoadAudioIntoOutputAsset(client, config.ResourceGroup, config.AccountName, outputAsset.Name, "audio_description.m4a");
```

## <a name="edit-the-ism-file"></a>Modifier le fichier .ism

Lorsque votre travail d’encodage est terminé, la ressource de sortie contiendra les fichiers générés par le travail d’encodage. 

1. Dans le Portail Azure, accédez au compte de stockage associé à votre compte Media Services. 
1. Recherchez le conteneur portant le nom de votre ressource de sortie. 
1. Dans le conteneur, recherchez le fichier .ism, puis cliquez sur **Modifier l’objet blob** (dans la fenêtre de droite). 
1. Modifiez le fichier .ism en ajoutant les informations sur le fichier MP4 audio uniquement chargé (codec AAC) contenant l’audio descriptif et appuyez sur **Enregistrer** quand vous avez terminé.

    Pour signaler les pistes audio descriptives, vous devez ajouter les paramètres « accessibility » et « role » au fichier .ism. C’est à vous qu’il incombe de définir correctement ces paramètres pour signaler une piste audio en tant que description audio. Par exemple, ajoutez `<param name="accessibility" value="description" />` et `<param name="role" value="alternate" />` au fichier .ism pour une piste audio spécifique, comme illustré dans l’exemple suivant.
 
```xml
<?xml version="1.0" encoding="utf-8"?>
<smil xmlns="http://www.w3.org/2001/SMIL20/Language">
  <head>
    <meta name="clientManifestRelativePath" content="ignite.ismc" />
    <meta name="formats" content="mp4-v3" />
  </head>
  <body>
    <switch>
      <audio src="ignite_320x180_AACAudio_381.mp4" systemBitrate="128041" systemLanguage="eng">
        <param name="systemBitrate" value="128041" valuetype="data" />
        <param name="trackID" value="2" valuetype="data" />
        <param name="trackName" value="aac_eng_2_128041_2_1" valuetype="data" />
        <param name="systemLanguage" value="eng" valuetype="data" />
        <param name="trackIndex" value="ignite_320x180_AACAudio_381_2.mpi" valuetype="data" />
      </audio>
      <audio src="audio_description.m4a" systemBitrate="194000" systemLanguage="eng">
        <param name="trackName" value="aac_eng_audio_description" />
        <param name="accessibility" value="description" />
        <param name="role" value="alternate" />     
      </audio>          
      <video src="ignite_1280x720_AACAudio_3549.mp4" systemBitrate="3549855">
        <param name="systemBitrate" value="3549855" valuetype="data" />
        <param name="trackID" value="1" valuetype="data" />
        <param name="trackName" value="video" valuetype="data" />
        <param name="trackIndex" value="ignite_1280x720_AACAudio_3549_1.mpi" valuetype="data" />
      </video>
      <video src="ignite_960x540_AACAudio_2216.mp4" systemBitrate="2216764">
        <param name="systemBitrate" value="2216764" valuetype="data" />
        <param name="trackID" value="1" valuetype="data" />
        <param name="trackName" value="video" valuetype="data" />
        <param name="trackIndex" value="ignite_960x540_AACAudio_2216_1.mpi" valuetype="data" />
      </video>
      <video src="ignite_640x360_AACAudio_1154.mp4" systemBitrate="1154569">
        <param name="systemBitrate" value="1154569" valuetype="data" />
        <param name="trackID" value="1" valuetype="data" />
        <param name="trackName" value="video" valuetype="data" />
        <param name="trackIndex" value="ignite_640x360_AACAudio_1154_1.mpi" valuetype="data" />
      </video>
      <video src="ignite_480x270_AACAudio_721.mp4" systemBitrate="721893">
        <param name="systemBitrate" value="721893" valuetype="data" />
        <param name="trackID" value="1" valuetype="data" />
        <param name="trackName" value="video" valuetype="data" />
        <param name="trackIndex" value="ignite_480x270_AACAudio_721_1.mpi" valuetype="data" />
      </video>
      <video src="ignite_320x180_AACAudio_381.mp4" systemBitrate="381027">
        <param name="systemBitrate" value="381027" valuetype="data" />
        <param name="trackID" value="1" valuetype="data" />
        <param name="trackName" value="video" valuetype="data" />
        <param name="trackIndex" value="ignite_320x180_AACAudio_381_1.mpi" valuetype="data" />
      </video>
    </switch>
  </body>
</smil>
```

## <a name="get-a-streaming-locator"></a>Obtenir un localisateur de diffusion en continu (streaming)

Une fois l’encodage terminé, l’étape suivante consiste à mettre à la disposition des clients la vidéo dans la ressource de sortie pour qu’ils puissent la lire. Vous pouvez le faire en deux étapes : d’abord, créez un [localisateur de streaming](/rest/api/media/streaminglocators), puis générez les URL de streaming que les clients peuvent utiliser. 

Le processus de création d’un **localisateur de streaming** est appelée « publication ». Par défaut, le **localisateur de streaming** est valide immédiatement après avoir effectué les appels d’API et dure jusqu’à ce qu’il soit supprimé, sauf si vous configurez les durées de début et de fin optionnelles. 

Lors de la création d’un élément [StreamingLocator](/rest/api/media/streaminglocators), vous devez spécifier le **StreamingPolicyName** souhaité. Dans cet exemple, vous allez diffuser en continu du contenu en clair ou non chiffré ; la stratégie de diffusion en continu en clair prédéfinie (**PredefinedStreamingPolicy.ClearStreamingOnly**) est donc utilisée.

> [!IMPORTANT]
> Quand vous utilisez une [stratégie de streaming](/rest/api/media/streamingpolicies) personnalisée, vous devez concevoir un ensemble limité de ces stratégies pour votre compte Media Services et les réutiliser pour vos éléments StreamingLocators chaque fois que les mêmes protocoles et options de chiffrement sont nécessaires. Votre compte Media Services a un quota en matière de nombre d’entrées de stratégie de streaming. Vous ne devez pas créer une stratégie de diffusion en continu pour chaque localisateur de diffusion en continu.

Le code suivant suppose que vous appelez la fonction avec un locatorName unique.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateStreamingLocator)]

Tandis que l’exemple de cette rubrique traite de streaming, vous pouvez utiliser le même appel pour créer un localisateur de streaming pour la diffusion de contenu vidéo par le biais d’un téléchargement progressif.

### <a name="get-streaming-urls"></a>Obtenir des URL de diffusion en continu

Maintenant que le [localisateur de streaming](/rest/api/media/streaminglocators) a été créé, vous pouvez obtenir les URL de streaming, comme indiqué dans **GetStreamingURLs**. Pour générer une URL, vous devez concaténer le nom d’hôte du [point de terminaison de streaming](/rest/api/media/streamingendpoints) et le chemin du **localisateur de streaming**. Dans cet exemple, le **point de terminaison de streaming** *par défaut* est utilisé. Quand vous créez pour la première fois un compte Media Services, ce **point de terminaison de streaming** *par défaut* est dans l’état Arrêté. Vous devez donc appeler **Start**.

> [!NOTE]
> Dans cette méthode, vous avez besoin de la valeur locatorName qui a été utilisé lors de la création du **localisateur de streaming** pour l’actif multimédia de sortie.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#GetStreamingURLs)]

## <a name="test-with-azure-media-player"></a>Test avec Azure Media Player

Pour tester la diffusion en continu, cet article utilise le lecteur multimédia Azure. 

> [!NOTE]
> Si un lecteur est hébergé sur un site https, veillez à mettre à jour l’URL vers « https ».

1. Ouvrez un navigateur web et accédez à [https://aka.ms/azuremediaplayer/](https://aka.ms/azuremediaplayer/).
2. Dans le champ **URL :** , collez l’une des valeurs d’URL de diffusion en continu que vous avez obtenues de votre application. 
 
     Vous pouvez coller l’URL dans le format HLS, Dash ou Smooth : le lecteur multimédia Azure passe automatiquement à un protocole de streaming approprié pour la lecture sur votre appareil.
3. Appuyez sur **Mise à jour du Lecteur Windows Media**.

Le lecteur multimédia Azure peut être utilisé pour effectuer des tests, mais ne doit pas être utilisé dans un environnement de production. 

## <a name="next-steps"></a>Étapes suivantes

[Analyser des vidéos](analyze-videos-tutorial-with-api.md)
