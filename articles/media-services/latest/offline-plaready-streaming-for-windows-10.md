---
title: Configurer la diffusion en continu PlayReady hors connexion avec Azure Media Services v3
description: Cet article vous montre comment configurer votre compte Azure Media Services v3 pour la diffusion en continu hors connexion de PlayReady pour Windows 10.
services: media-services
keywords: DASH, DRM, Widevine hors connexion, ExoPlayer, Android
documentationcenter: ''
author: willzhan
manager: steveng
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/31/2020
ms.author: willzhan
ms.custom: devx-track-csharp
ms.openlocfilehash: 19c930915de20d7f3a8938570e074458b32a8efb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90975676"
---
# <a name="offline-playready-streaming-for-windows-10-with-media-services-v3"></a>Diffusion en continu PlayReady hors connexion pour Windows 10 avec Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Azure Media Services prend en charge le téléchargement/la lecture hors connexion avec la protection DRM. Cet article décrit la prise en charge hors connexion d’Azure Media Services pour les clients Windows 10/PlayReady. Pour en savoir plus sur la prise en charge du mode hors connexion pour les appareils iOS/FairPlay et Android/Widevine, consultez les articles suivants :

- [Diffusion en continu de FairPlay en mode hors connexion pour iOS](offline-fairplay-for-ios.md)
- [Diffusion en continu de Widevine en mode hors connexion pour Android](offline-widevine-for-android.md)

> [!NOTE]
> DRM hors connexion est uniquement facturé pour une requête unique de licence lorsque vous téléchargez le contenu. Les erreurs ne sont pas facturées.

## <a name="overview"></a>Vue d’ensemble

Cette section donne des précisions sur la lecture en mode hors connexion, notamment sur les points suivants :

* Dans certains pays/certaines régions, la disponibilité Internet et/ou la bande passante sont encore limitées. Les utilisateurs peuvent choisir de télécharger d’abord afin de pouvoir regarder le contenu en résolution suffisamment élevée pour une expérience d’affichage satisfaisante. Dans ce cas, le problème ne concerne pas majoritairement la disponibilité du réseau, mais une bande passante réseau limitée. Les fournisseurs OTT/OVP doivent fournir une prise en charge du mode hors connexion.
* Comme l’a indiqué Reed Hastings, PDG de Netflix, lors de la réunion des actionnaires Netflix du troisième trimestre 2016, le téléchargement de contenu est une « fonctionnalité très demandée » à laquelle « nous sommes ouverts ».
* Certains fournisseurs de contenu peuvent interdire la remise de licence de gestion des droits numériques (DRM) au-delà de la frontière d’un pays/d’une région. Si un utilisateur en voyage à l’étranger souhaite regarder du contenu, le téléchargement hors connexion est nécessaire.
 
Le défi auquel nous sommes confrontés pour l’implémentation du mode hors connexion est le suivant :

* Le format MP4 est pris en charge par la plupart des lecteurs et outils d’encodeur, mais il n’y a aucune liaison entre le conteneur MP4 et DRM.
* À terme, il conviendra d’utiliser CFF avec CENC. Toutefois, à l’heure actuelle, l’écosystème de prise en charge des outils/lecteurs n’existe pas encore. Nous devons trouver une solution rapidement.
 
L’idée est la suivante : le format de fichier de diffusion en continu lisse ([PIFF](/iis/media/smooth-streaming/protected-interoperable-file-format)) avec H264/AAC possède une liaison avec PlayReady (AES-128 CTR). Un fichier .ismv de diffusion en continu lisse individuel (en admettant que l’audio soit multiplexé dans la vidéo) est en soi un fichier fMP4 et peut être utilisé pour la lecture. Si un contenu de diffusion en continu lisse subit un chiffrement PlayReady, chaque fichier .ismv devient un fichier MP4 fragmenté protégé PlayReady. Il est possible de choisir le fichier .ismv ayant la vitesse de transmission souhaitée et de le renommer en .mp4 pour le téléchargement.

Deux options sont disponibles pour héberger le fichier MP4 protégé PlayReady pour le téléchargement progressif :

* On peut placer ce fichier MP4 dans le même conteneur ou ressource de service multimédia et tirer parti du point de terminaison de streaming Azure Media Services pour le téléchargement progressif ;
* On peut utiliser le localisateur SAS pour le téléchargement progressif directement à partir de Stockage Azure, et ainsi contourner Azure Media Services.
 
Vous pouvez utiliser deux systèmes de distribution de licences PlayReady :

* Le service de distribution de licences PlayReady dans Azure Media Services ;
* Les serveurs de licences PlayReady où qu’ils soient hébergés.

Voici deux jeux de ressources de test. Le premier utilise la distribution de licences PlayReady dans AMS et le deuxième utilise mon serveur de licences PlayReady hébergé sur une machine virtuelle Azure :

Ressource 1 :

* URL de téléchargement progressif : [https://willzhanmswest.streaming.mediaservices.windows.net/8d078cf8-d621-406c-84ca-88e6b9454acc/20150807-bridges-2500_H264_1644kbps_AAC_und_ch2_256kbps.mp4](https://willzhanmswest.streaming.mediaservices.windows.net/8d078cf8-d621-406c-84ca-88e6b9454acc/20150807-bridges-2500_H264_1644kbps_AAC_und_ch2_256kbps.mp4)
* PlayReady LA_URL (AMS) : `https://willzhanmswest.keydelivery.mediaservices.windows.net/PlayReady/`

Ressource 2 :

* URL de téléchargement progressif : [https://willzhanmswest.streaming.mediaservices.windows.net/7c085a59-ae9a-411e-842c-ef10f96c3f89/20150807-bridges-2500_H264_1644kbps_AAC_und_ch2_256kbps.mp4](https://willzhanmswest.streaming.mediaservices.windows.net/7c085a59-ae9a-411e-842c-ef10f96c3f89/20150807-bridges-2500_H264_1644kbps_AAC_und_ch2_256kbps.mp4)
* PlayReady LA_URL (local) : `https://willzhan12.cloudapp.net/playready/rightsmanager.asmx`

Pour tester la lecture, nous avons utilisé une application Windows universelle sur Windows 10. Dans [Exemples Windows 10 Universel](https://github.com/Microsoft/Windows-universal-samples) se trouve un exemple de lecteur de base appelé [Exemple de diffusion en continu adaptative](https://github.com/Microsoft/Windows-universal-samples/tree/master/Samples/AdaptiveStreaming). Il suffit d’ajouter le code pour sélectionner la vidéo téléchargée et l’utiliser comme source, à la place d’une source de diffusion en continu adaptative. Les modifications figurent dans le gestionnaire d’événements Click :

```csharp
private async void LoadUri_Click(object sender, RoutedEventArgs e)
{
    //Uri uri;
    //if (!Uri.TryCreate(UriBox.Text, UriKind.Absolute, out uri))
    //{
    // Log("Malformed Uri in Load text box.");
    // return;
    //}
    //LoadSourceFromUriTask = LoadSourceFromUriAsync(uri);
    //await LoadSourceFromUriTask;

    //willzhan change start
    // Create and open the file picker
    FileOpenPicker openPicker = new FileOpenPicker();
    openPicker.ViewMode = PickerViewMode.Thumbnail;
    openPicker.SuggestedStartLocation = PickerLocationId.ComputerFolder;
    openPicker.FileTypeFilter.Add(".mp4");
    openPicker.FileTypeFilter.Add(".ismv");
    //openPicker.FileTypeFilter.Add(".mkv");
    //openPicker.FileTypeFilter.Add(".avi");

    StorageFile file = await openPicker.PickSingleFileAsync();

    if (file != null)
    {
       //rootPage.NotifyUser("Picked video: " + file.Name, NotifyType.StatusMessage);
       this.mediaPlayerElement.MediaPlayer.Source = MediaSource.CreateFromStorageFile(file);
       this.mediaPlayerElement.MediaPlayer.Play();
       UriBox.Text = file.Path;
    }
    else
    {
       // rootPage.NotifyUser("Operation cancelled.", NotifyType.ErrorMessage);
    }

    // On small screens, hide the description text to make room for the video.
    DescriptionText.Visibility = (ActualHeight < 500) ? Visibility.Collapsed : Visibility.Visible;
}
```

![Lecture en mode hors connexion de fichiers fMP4 protégés PlayReady](./media/offline-playready-for-windows/offline-playready1.jpg)

La vidéo étant sous protection de PlayReady, elle ne pourra pas être incluse dans la capture d’écran.

En résumé, nous avons mis en place le mode hors connexion dans Azure Media Services :

* Le transcodage de contenu et le chiffrement PlayReady peuvent être effectués dans Azure Media Services ou d’autres outils ;
* Le contenu peut être hébergé dans Azure Media Services ou Stockage Azure pour le téléchargement progressif ;
* La distribution de licences PlayReady peut se faire avec Azure Media Services ou un autre service ;
* Le contenu de diffusion en continu lisse préparé peut toujours être utilisé pour la diffusion en continu en ligne via DASH ou lisse avec PlayReady comme DRM.

## <a name="next-steps"></a>Étapes suivantes

[Concevoir un système de protection de contenu multi-DRM avec contrôle d’accès](design-multi-drm-system-with-access-control.md)
