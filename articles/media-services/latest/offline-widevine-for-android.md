---
title: Diffusez en continu Widevine Android hors connexion avec Azure Media Services V3
description: Cette rubrique montre comment configurer votre compte Azure Media Services v3 pour la diffusion en continu hors connexion du contenu protégé de Widevine.
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
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: willzhan
ms.custom: devx-track-csharp
ms.openlocfilehash: b8c4bed81a73957cc80318064f2aa2a58b3cfe11
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91597079"
---
# <a name="offline-widevine-streaming-for-android-with-media-services-v3"></a>Diffusion en continu Widevine hors connexion pour Android avec Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Outre la protection de contenu pour la diffusion en continu en ligne, les services d’abonnement et de location du contenu multimédia offrent un contenu téléchargeable qui fonctionne lorsque vous n’êtes pas connecté à internet. Vous devrez peut-être télécharger le contenu sur votre téléphone ou tablette pour une lecture en mode avion, lorsque vous êtes en vol et déconnecté du réseau. Scénarios supplémentaires, dans lesquels vous pouvez souhaiter télécharger du contenu :

- Certains fournisseurs de contenu peuvent interdire la remise de licence de gestion des droits numériques (DRM) au-delà de la frontière d’un pays/d’une région. Si un utilisateur souhaite regarder du contenu en voyage à l’étranger, le téléchargement hors connexion est nécessaire.
- Dans certains pays/certaines régions, la disponibilité Internet ou la bande passante sont limitées. Les utilisateurs peuvent choisir de télécharger du contenu afin de pouvoir le regarder en résolution suffisamment élevée pour une expérience d’affichage satisfaisante.

[!INCLUDE [Widevine is not available in the GovCloud region.](./includes/widevine-not-available-govcloud.md)]

Cet article explique comment mettre en œuvre la lecture en mode hors connexion pour le contenu DASH protégé par Widevine sur les appareils Android. La gestion des droits numériques (DRM) hors connexion vous permet de fournir des modèles d’abonnement, de location et d’achat pour votre contenu, permettant aux clients de vos services d’emporter facilement du contenu avec eux lorsqu’ils sont déconnectés d’internet.

Pour créer les applications du lecteur Android, nous soulignons trois options :

> [!div class="checklist"]
> * créer un lecteur à l’aide de l’API Java du kit de développement logiciel (SDK) ExoPlayer
> * créer un lecteur à l’aide de la liaison Xamarin du kit de développement logiciel (SDK) ExoPlayer
> * créer un lecteur à l’aide de Encrypted Media Extension (EME) et Media Source Extension (MSE) dans le navigateur mobile Chrome v62 ou version ultérieure

L’article répond également à certaines questions fréquentes sur la diffusion en continu du contenu protégé de Widevine.

> [!NOTE]
> DRM hors connexion est uniquement facturé pour une requête unique de licence lorsque vous téléchargez le contenu. Les erreurs ne sont pas facturées.

## <a name="prerequisites"></a>Prérequis 

Avant de mettre en œuvre la DRM hors connexion pour Widevine sur des appareils Android, vous devez, tout d’abord :

- Vous familiariser avec les concepts présentés pour la protection du contenu en ligne à l’aide de la DRM de Widevine. Vous trouverez toutes les informations nécessaires dans les documents/exemples suivants :
    - [Concevoir un système de protection de contenu multi-DRM avec contrôle d’accès](design-multi-drm-system-with-access-control.md)
    - [Utilisation du chiffrement dynamique DRM et du service de remise des licences](protect-with-drm.md)
- Clonez https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git.

    Pour ajouter des configurations Widevine, vous devrez modifier le code fourni dans [Encrypt with DRM using .NET](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/master/AMSV3Tutorials/EncryptWithDRM).  
- Vous familiariser avec le kit de développement logiciel (SDK) Google ExoPlayer pour Android, un kit de développement logiciel (SDK) de lecteur vidéo open source capable de prendre en charge la lecture de la DRM de Widevine hors connexion. 
    - [Kit de développement logiciel SDK ExoPlayer](https://github.com/google/ExoPlayer)
    - [Guide du Développeur Exoplayer](https://google.github.io/ExoPlayer/guide.html)
    - [Blog du Développeur Exoplayer](https://medium.com/google-exoplayer)

## <a name="configure-content-protection-in-azure-media-services"></a>Configurer la protection du contenu dans Azure Media Services

Dans la méthode [GetOrCreateContentKeyPolicyAsync](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L189), les étapes nécessaires suivantes sont présentes :

1. Spécifiez comment la distribution des clés de contenu doit être autorisée dans le service de distribution des licences : 

    ```csharp
    ContentKeyPolicySymmetricTokenKey primaryKey = new ContentKeyPolicySymmetricTokenKey(tokenSigningKey);
    List<ContentKeyPolicyTokenClaim> requiredClaims = new List<ContentKeyPolicyTokenClaim>()
    {
        ContentKeyPolicyTokenClaim.ContentKeyIdentifierClaim
    };
    List<ContentKeyPolicyRestrictionTokenKey> alternateKeys = null;
    ContentKeyPolicyTokenRestriction restriction 
        = new ContentKeyPolicyTokenRestriction(Issuer, Audience, primaryKey, ContentKeyPolicyRestrictionTokenType.Jwt, alternateKeys, requiredClaims);
    ```
2. Configurez le modèle de licence Widevine :  

    ```csharp
    ContentKeyPolicyWidevineConfiguration widevineConfig = ConfigureWidevineLicenseTempate();
    ```

3. Créez ContentKeyPolicyOptions :

    ```csharp
    options.Add(
        new ContentKeyPolicyOption()
        {
            Configuration = widevineConfig,
            Restriction = restriction
        });
    ```

## <a name="enable-offline-mode"></a>Activer le mode hors connexion

Pour activer le mode **hors connexion** pour les licences Widevine, vous devez configurer le [modèle de licence Widevine](widevine-license-template-overview.md). Dans l’objet **policy_overrides**, définissez la propriété **can_persist** sur **true** (la valeur par défaut est false), comme indiqué dans [ConfigureWidevineLicenseTempate](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L563). 

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#ConfigureWidevineLicenseTempate)]

## <a name="configuring-the-android-player-for-offline-playback"></a>Configuration du lecteur Android pour la lecture hors connexion

Le moyen le plus simple de développer une application de lecteur natif pour les appareils Android est d’utiliser le [Kit de développement logiciel (SDK) Google ExoPlayer](https://github.com/google/ExoPlayer), un kit de développement logiciel du lecteur vidéo open source. ExoPlayer prend en charge des fonctionnalités qui ne sont actuellement pas prises en charge par l'API MediaPlayer native d'Android, notamment les protocoles de remise MPEG-DASH et Microsoft Smooth Streaming.

ExoPlayer version 2.6 et version supérieure inclut de nombreuses classes qui prennent en charge la lecture de la DRM de Widevine hors connexion. En particulier, la classe OfflineLicenseHelper fournit des fonctions utilitaires pour faciliter l’utilisation de la DefaultDrmSessionManager pour le téléchargement, le renouvellement et la libération des licences hors connexion. Les classes fournies dans le dossier du kit de développement logiciel "library/core/src/main/java/com/google/android/exoplayer2/offline/" prennent en charge le téléchargement du contenu vidéo hors connexion.

Les classes suivantes permettent le mode hors connexion dans le SDK ExoPlayer pour Android :

- library/core/src/main/java/com/google/android/exoplayer2/drm/OfflineLicenseHelper.java  
- library/core/src/main/java/com/google/android/exoplayer2/drm/DefaultDrmSession.java
- library/core/src/main/java/com/google/android/exoplayer2/drm/DefaultDrmSessionManager.java
- library/core/src/main/java/com/google/android/exoplayer2/drm/DrmSession.java
- library/core/src/main/java/com/google/android/exoplayer2/drm/ErrorStateDrmSession.java
- library/core/src/main/java/com/google/android/exoplayer2/drm/ExoMediaDrm.java
- library/core/src/main/java/com/google/android/exoplayer2/offline/SegmentDownloader.java
- library/core/src/main/java/com/google/android/exoplayer2/offline/DownloaderConstructorHelper.java 
- library/core/src/main/java/com/google/android/exoplayer2/offline/Downloader.java
- library/dash/src/main/java/com/google/android/exoplayer2/source/dash/offline/DashDownloader.java 

Les développeurs doivent référencer le [Guide du développeur ExoPlayer](https://google.github.io/ExoPlayer/guide.html) et [Blog des développeurs](https://medium.com/google-exoplayer) correspondant pendant le développement d’une application. Google n’a pas encore publié d’implémentation de référence entièrement documentée ni d’exemple de code pour l’application ExoPlayer qui prend en charge Widevine hors connexion. Les informations sont donc limitées au guide et au blog des développeurs. 

### <a name="working-with-older-android-devices"></a>Utilisation des anciens appareils Android

En ce qui concerne certains anciens appareils Android, vous devez définir des valeurs pour les propriétés **policy_overrides** suivantes (définies dans le [modèle de licence Widevine ](widevine-license-template-overview.md) : **rental_duration_seconds**, **playback_duration_seconds** et **license_duration_seconds**. Ou bien, vous pouvez les définir à zéro, ce qui signifie une durée illimitée/infinie.  

Les valeurs doivent être définies pour éviter un bogue de dépassement de capacité d’entier. Pour en savoir plus sur le problème, consultez https://github.com/google/ExoPlayer/issues/3150 et https://github.com/google/ExoPlayer/issues/3112. <br/>Si vous ne définissez pas explicitement les valeurs, des valeurs très élevées pour **PlaybackDurationRemaining** et **LicenseDurationRemaining** seront affectées, (par exemple, 9223372036854775807, qui est la valeur positive maximale pour un entier de 64 bits). Par conséquent, la licence Widevine s’affiche comme expirée, le déchiffrement n’a donc pas lieu. 

Ce problème ne se produit pas sur Android Lollipop 5.0 ou version ultérieure, Android 5.0 étant la première version Android conçue pour prendre entièrement en charge des plateformes ARMv8 ([Advanced RISC Machine](https://en.wikipedia.org/wiki/ARM_architecture)) et 64 bits, tandis qu’Android 4.4 KitKat a été conçu à l’origine pour prendre en charge des plateformes ARMv7 et 32 bits comme pour d’autres versions Android plus anciennes.

## <a name="using-xamarin-to-build-an-android-playback-app"></a>Utilisation de Xamarin pour générer une application de lecture Android

Vous pouvez rechercher des liaisons Xamarin pour ExoPlayer à l’aide des liens suivants :

- [Bibliothèque de liaisons Xamarin pour la bibliothèque de Google ExoPlayer](https://github.com/martijn00/ExoPlayerXamarin)
- [Liaisons Xamarin pour ExoPlayer NuGet](https://www.nuget.org/packages/Xam.Plugins.Android.ExoPlayer/)

Consultez également le thread suivant : [Liaison Xamarin](https://github.com/martijn00/ExoPlayerXamarin/pull/57). 

## <a name="chrome-player-apps-for-android"></a>Applications du lecteur Chrome pour Android

À partir de la version de [Chrome pour Android v. 62](https://developers.google.com/web/updates/2017/09/chrome-62-media-updates), la licence persistant dans EME est prise en charge. [Widevine L1](https://developers.google.com/web/updates/2017/09/chrome-62-media-updates#widevine_l1) est maintenant également pris en charge dans Chrome pour Android. Cela vous permet de créer des applications de lecture hors connexion dans Chrome si vos utilisateurs finaux disposent de cette version de Chrome (ou version ultérieure). 

En outre, Google a généré un exemple de Progressive Web App (PWA), procuré en open source : 

- [Code source](https://github.com/GoogleChromeLabs/sample-media-pwa)
- [Version Google hébergée](https://biograf-155113.appspot.com/ttt/episode-2/) (fonctionne uniquement dans Chrome v 62 et version ultérieure sur les appareils Android)

Si vous mettez à niveau votre navigateur Chrome mobile sur la v62 (ou version ultérieure) sur un téléphone Android et testez l’exemple d’application hébergée ci-dessus, vous constaterez que la lecture en continu en ligne et la lecture hors connexion fonctionnent.

L’application PWA open source ci-dessus a été créée dans Node.js. Si vous souhaitez héberger votre propre version sur un serveur Ubuntu, tenez compte des problèmes fréquemment rencontrés suivants qui peuvent empêcher la lecture :

1. Problème CORS : l’échantillon vidéo de l’exemple d’application est hébergé dans https://storage.googleapis.com/biograf-video-files/videos/. Google a défini CORS pour tous les exemples de test hébergés dans le compartiment de stockage Cloud de Google. Ils sont pris en charge avec les en-têtes CORS, en spécifiant explicitement l’entrée CORS : `https://biograf-155113.appspot.com` (domaine dans lequel Google héberge ses exemples) empêchant l’accès par d’autres sites. Si vous essayez, vous verrez l'erreur HTTP suivante : `Failed to load https://storage.googleapis.com/biograf-video-files/videos/poly-sizzle-2015/mp4/dash.mpd: No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'https:\//13.85.80.81:8080' is therefore not allowed access. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.`
2. Problème de certificat : À partir de Chrome v58, EME pour Widevine nécessite HTTPS. Par conséquent, vous devez héberger l’exemple d’application via le protocole HTTPS avec un certificat X509. Les certificats de test habituels ne fonctionnent pas en raison des exigences suivantes : Vous devez obtenir un certificat répondant aux exigences minimales suivantes :
    - Chrome et Firefox exige que le paramètre Nom alternatif de l’objet SAN existe dans le certificat
    - Le certificat doit avoir une certification approuvée et un certificat auto-signé de développement ne fonctionne pas
    - Le certificat doit avoir un CN correspondant au nom DNS de la passerelle ou du serveur web

## <a name="faqs"></a>FAQ

Pour plus d’informations, consultez la [FAQ de Widevine](frequently-asked-questions.md#widevine-streaming-for-android).

## <a name="additional-notes"></a>Remarques supplémentaires

Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="summary"></a>Résumé

Cet article explique comment mettre en œuvre la lecture en mode hors connexion pour le contenu DASH protégé par Widevine sur les appareils Android.  L’article répond également à certaines questions fréquentes sur la diffusion en continu du contenu protégé de Widevine.
