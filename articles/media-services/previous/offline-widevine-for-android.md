---
title: Configurez votre compte pour une diffusion en continu hors connexion du contenu protégé de Widevine - Azure
description: Cette rubrique montre comment configurer votre compte Azure Media Services pour la diffusion en continu hors connexion du contenu protégé de Widevine.
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
ms.date: 04/16/2019
ms.author: willzhan
ms.reviewer: dwgeo
ms.openlocfilehash: 4b3b2b8c39b5b2552b5ce9f508bacd1ea86b2638
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006357"
---
# <a name="offline-widevine-streaming-for-android"></a>Widevine hors connexion pour Android

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector" title1="Sélectionnez la version de Media Services que vous utilisez :"]
> * [Version 3](../latest/offline-widevine-for-android.md)
> * [Version 2](offline-widevine-for-android.md)

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

Outre la protection de contenu pour la diffusion en continu en ligne, les services d’abonnement et de location du contenu multimédia offrent un contenu téléchargeable qui fonctionne lorsque vous n’êtes pas connecté à internet. Vous devrez peut-être télécharger le contenu sur votre téléphone ou tablette pour une lecture en mode avion, lorsque vous êtes en vol et déconnecté du réseau. Scénarios supplémentaires, dans lesquels vous pouvez souhaiter télécharger du contenu :

- Certains fournisseurs de contenu peuvent interdire la remise de licence de gestion des droits numériques (DRM) au-delà de la frontière d’un pays/d’une région. Si un utilisateur souhaite regarder du contenu en voyage à l’étranger, le téléchargement hors connexion est nécessaire.
- Dans certains pays/certaines régions, la disponibilité Internet ou la bande passante sont limitées. Les utilisateurs peuvent choisir de télécharger du contenu afin de pouvoir le regarder en résolution suffisamment élevée pour une expérience d’affichage satisfaisante.

Cet article explique comment mettre en œuvre la lecture en mode hors connexion pour le contenu DASH protégé par Widevine sur les appareils Android. La gestion des droits numériques (DRM) hors connexion vous permet de fournir des modèles d’abonnement, de location et d’achat pour votre contenu, permettant aux clients de vos services d’emporter facilement du contenu avec eux lorsqu’ils sont déconnectés d’internet.

Pour créer les applications du lecteur Android, nous soulignons trois options :

> [!div class="checklist"]
> * créer un lecteur à l’aide de l’API Java du kit de développement logiciel (SDK) ExoPlayer
> * créer un lecteur à l’aide de la liaison Xamarin du kit de développement logiciel (SDK) ExoPlayer
> * créer un lecteur à l’aide de Encrypted Media Extension (EME) et Media Source Extension (MSE) dans le navigateur mobile Chrome v62 ou version ultérieure

L’article répond également à certaines questions fréquentes sur la diffusion en continu du contenu protégé de Widevine.

## <a name="requirements"></a>Configuration requise 

Avant de mettre en œuvre la DRM hors connexion pour Widevine sur des appareils Android, vous devez, tout d’abord :

- Vous familiariser avec les concepts présentés pour la protection du contenu en ligne à l’aide de la DRM de Widevine. Vous trouverez toutes les informations nécessaires dans les documents/exemples suivants :
    - [Utiliser Azure Media Services pour fournir des licences DRM ou des clés AES](media-services-deliver-keys-and-licenses.md)
    - [CENC avec Multi-DRM et contrôle d’accès : une conception de référence et l’application sur Windows Azure et Azure Media Services](media-services-cenc-with-multidrm-access-control.md)
    - [Utilisation du chiffrement commun dynamique PlayReady et/ou Widevine avec .NET](https://azure.microsoft.com/resources/samples/media-services-dotnet-dynamic-encryption-with-drm/)
    - [Utiliser Azure Media Services pour fournir des licences PlayReady et/ou Widevine avec .NET](https://azure.microsoft.com/resources/samples/media-services-dotnet-deliver-playready-widevine-licenses/)
- Vous familiariser avec le kit de développement logiciel (SDK) Google ExoPlayer pour Android, un kit de développement logiciel (SDK) de lecteur vidéo open source capable de prendre en charge la lecture de la DRM de Widevine hors connexion. 
    - [Kit de développement logiciel SDK ExoPlayer](https://github.com/google/ExoPlayer)
    - [Guide du Développeur Exoplayer](https://google.github.io/ExoPlayer/guide.html)
    - [Blog du Développeur Exoplayer](https://medium.com/google-exoplayer)

## <a name="content-protection-configuration-in-azure-media-services"></a>Configuration de la protection du contenu dans Azure Media Services

Lorsque vous configurez la protection Widevine d’un élément dans Media Services, vous devez créer ContentKeyAuthorizationPolicyOption qui spécifie les trois objets suivants :

1. Système DRM (Widevine)
2. ContentKeyAuthorizationPolicyRestriction spécifiant comment la remise de la clé de contenu est autorisée dans le service de remise de licences (autorisation ouverte ou de type jeton)
3. Modèle de licence (Widevine) DRM

Pour activer le mode **hors connexion** pour les licences Widevine, vous devez configurer le [modèle de licence Widevine](media-services-widevine-license-template-overview.md). Dans l’objet **policy_overrides**, définissez la propriété **can_persist** sur **true** (la valeur par défaut est false). 

L’exemple de code suivant utilise .NET pour activer le mode **hors connexion** pour des licences Widevine. Le code est basé sur l’exemple [ Utilisation du chiffrement commun dynamique PlayReady et/ou Widevine avec .NET](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-drm). 

```
private static string ConfigureWidevineLicenseTemplateOffline(Uri keyDeliveryUrl)
{
    var template = new WidevineMessage
    {
        allowed_track_types = AllowedTrackTypes.SD_HD,
        content_key_specs = new[]
        {
            new ContentKeySpecs
            {
                required_output_protection = new RequiredOutputProtection { hdcp = Hdcp.HDCP_NONE},
                security_level = 1,
                track_type = "SD"
            }
        },
        policy_overrides = new
        {
            can_play = true,
            can_persist = true,
            //can_renew = true,                                //if you set can_renew = false, you do not need renewal_server_url
            //renewal_server_url = keyDeliveryUrl.ToString(),    //not mandatory, renewal_server_url is needed only if license_duration_seconds is set
            can_renew = false,
            //rental_duration_seconds = 1209600,
            //playback_duration_seconds = 1209600,
            //license_duration_seconds = 1209600
        }
        };

    string configuration = Newtonsoft.Json.JsonConvert.SerializeObject(template);
    return configuration;
}
```

## <a name="configuring-the-android-player-for-offline-playback"></a>Configuration du lecteur Android pour la lecture hors connexion

Le moyen le plus simple de développer une application de lecteur natif pour les appareils Android est d’utiliser le [Kit de développement logiciel (SDK) Google ExoPlayer](https://github.com/google/ExoPlayer), un kit de développement logiciel du lecteur vidéo open source. ExoPlayer prend en charge des fonctionnalités qui ne sont actuellement pas prises en charge par l'API MediaPlayer native d'Android, notamment les protocoles de remise MPEG-DASH et Microsoft Smooth Streaming.

ExoPlayer version 2.6 et version supérieure inclut de nombreuses classes qui prennent en charge la lecture de la DRM de Widevine hors connexion. En particulier, la classe OfflineLicenseHelper fournit des fonctions utilitaires pour faciliter l’utilisation de la DefaultDrmSessionManager pour le téléchargement, le renouvellement et la libération des licences hors connexion. Les classes fournies dans le dossier du kit de développement logiciel "library/core/src/main/java/com/google/android/exoplayer2/offline/" prennent en charge le téléchargement du contenu vidéo hors connexion.

La liste des classes suivante facilite le mode hors connexion dans le kit de développement logiciel ExoPlayer pour Android :

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

Les développeurs doivent référencer le [Guide du développeur ExoPlayer](https://google.github.io/ExoPlayer/guide.html) et [Blog des développeurs](https://medium.com/google-exoplayer) correspondant pendant le développement d’une application. Google n’a pas publié de mise en œuvre de référence entièrement documentée ou exemple de code pour l’application ExoPlayer prenant en charge Widevine hors connexion à ce stade, les informations sont donc limitées pour le guide et blog des développeurs. 

### <a name="working-with-older-android-devices"></a>Utilisation des anciens appareils Android

En ce qui concerne certains anciens appareils Android, vous devez définir des valeurs pour les propriétés **policy_overrides** suivantes (définies dans le [modèle de licence Widevine](media-services-widevine-license-template-overview.md) : **rental_duration_seconds**, **playback_duration_seconds** et **license_duration_seconds**. Ou bien, vous pouvez les définir à zéro, ce qui signifie une durée illimitée/infinie.  

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

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="question"></a>Question

Comment puis-je remettre des licences persistantes (activées en mode hors connexion) pour certains clients/utilisateurs et des licences non persistantes (désactivées en mode hors connexion) pour d’autres ? Dois-je dupliquer le contenu et utiliser une clé de contenu distincte ?

### <a name="answer"></a>Réponse
Vous n’avez pas besoin de dupliquer le contenu. Vous pouvez simplement utiliser une seule copie du contenu et une seule ContentKeyAuthorizationPolicy, mais deux ContentKeyAuthorizationPolicyOption distinctes :

1. IContentKeyAuthorizationPolicyOption 1 : utilise les licences persistantes et ContentKeyAuthorizationPolicyRestriction 1 qui contient une revendication comme license_type = "Persistent"
2. IContentKeyAuthorizationPolicyOption 2 : utilise une licence non persistante et ContentKeyAuthorizationPolicyRestriction 2 qui contient une revendication comme license_type = "Nonpersistent"

Ainsi, lorsqu’une requête de licence provient de l’application cliente, à partir de la demande de licence il n’existe aucune différence. Cependant, pour un appareil/utilisateur final différent, le STS doit avoir la logique d’entreprise pour émettre des jetons JWT différents contenant des revendications différentes (un des deux license_type ci-dessus). La valeur des revendications dans le jeton JWT sera utilisée pour permettre au service de licence de décider quel type de contrat de licence émettre : persistant ou non persistant.

Cela signifie que le Service STS (Secure Token Service) doit avoir les informations de logique métier et d’appareil/client pour ajouter la valeur de revendication correspondante dans un jeton.

### <a name="question"></a>Question

Pour les niveaux de sécurité Widevine, dans la [documentation Vue d’ensemble de l’architecture DRM Widevine](https://storage.googleapis.com/wvdocs/Widevine_DRM_Architecture_Overview.pdf) de Google, il définit trois niveaux différents de sécurité. Toutefois, dans [Documentation Azure Media Services sur le modèle de licence Widevine](./media-services-widevine-license-template-overview.md), cinq niveaux de sécurité différents sont soulignés. Quelle est la relation ou le mappage entre les deux différents ensembles de niveaux de sécurité ?

### <a name="answer"></a>Réponse

Dans [Vue d’ensemble de l’architecture DRM Widevine](https://storage.googleapis.com/wvdocs/Widevine_DRM_Architecture_Overview.pdf) de Google, il définit les trois niveaux de sécurité suivants :

1.  Niveau de sécurité 1 : tous les traitements, chiffrements et contrôles du contenu sont effectués au sein de l’environnement TEE (Trusted Execution Environment). En ce qui concerne certains modèles de mise en œuvre, le traitement de la sécurité peut être effectué dans différentes puces.
2.  Niveau de sécurité 2 : effectue le chiffrement (mais pas le traitement vidéo) dans l’environnement TEE : les mémoires tampons déchiffrées sont renvoyées au domaine d’application et traitées par le biais d’un matériel ou d’un logiciel vidéo distinct. Au niveau 2, toutefois, les informations de chiffrement sont toujours traitées uniquement dans l’environnement TEE.
3.  Le niveau de sécurité 3 n’a pas d’environnement TEE sur l’appareil. Les mesures appropriées peuvent être prises pour protéger les informations de chiffrement et le contenu déchiffré sur le système d’exploitation hôte. Une mise en œuvre de niveau 3 peut également inclure un moteur de chiffrement matériel, mais qui améliore seulement la performance, pas la sécurité.

Dans le même temps, dans [Documentation Azure Media Services sur le modèle de licence Widevine](./media-services-widevine-license-template-overview.md), la propriété security_level de content_key_specs peut avoir les cinq différentes valeurs suivantes (exigences de robustesse client pour la lecture) :

1.  Le chiffrement whitebox basé sur le logiciel est requis.
2.  Le chiffrement logiciel et un décodeur masqué sont requis.
3.  Les opérations de matériel clé et de chiffrement doivent être effectuées dans un environnement TEE soutenu par le matériel.
4.  Le chiffrement et le décodage du contenu doivent être effectués dans un environnement TEE soutenu par le matériel.
5.  Le chiffrement, le décodage et le traitement du support (compressé et décompressé) doivent être gérés dans un environnement TEE soutenu par le matériel.

Les deux niveaux de sécurité sont définis par Google Widevine. La différence réside dans son niveau d’utilisation : niveau d’architecture ou d’API. Les cinq niveaux de sécurité sont utilisés dans l’API Widevine. L’objet content_key_specs, qui contient security_level est désérialisé et transmis au service de remise globale Widevine par le service de licence Azure Media Services Widevine. Le tableau ci-dessous montre le mappage entre les deux ensembles de niveaux de sécurité.

| **Niveaux de sécurité définis dans l’architecture de Widevine** |**Niveaux de sécurité utilisés dans l’API de Widevine**|
|---|---| 
| **Niveau de sécurité 1** : tous les traitements, chiffrements et contrôles du contenu sont effectués au sein de l’environnement TEE (Trusted Execution Environment). En ce qui concerne certains modèles de mise en œuvre, le traitement de la sécurité peut être effectué dans différentes puces.|**security_level=5** : le chiffrement, le décodage et le traitement du support (compressé et décompressé) doivent être gérés dans un environnement TEE soutenu par le matériel.<br/><br/>**security_level=4** : le chiffrement et le décodage du contenu doivent être effectués dans un environnement TEE soutenu par le matériel.|
**Niveau de sécurité 2** : effectue le chiffrement (mais pas le traitement vidéo) dans l’environnement TEE : les mémoires tampons déchiffrées sont renvoyées au domaine d’application et traitées par le biais d’un matériel ou d’un logiciel vidéo distinct. Au niveau 2, toutefois, les informations de chiffrement sont toujours traitées uniquement dans l’environnement TEE.| **security_level=3** : les opérations de matériel clé et de chiffrement doivent être effectués dans un environnement TEE soutenu par le matériel. |
| **Niveau de sécurité 3** : n’a pas d’environnement TEE sur l’appareil. Les mesures appropriées peuvent être prises pour protéger les informations de chiffrement et le contenu déchiffré sur le système d’exploitation hôte. Une mise en œuvre de niveau 3 peut également inclure un moteur de chiffrement matériel, mais qui améliore seulement la performance, pas la sécurité. | **security_level=2** : le chiffrement logiciel et un décodeur masqué sont requis.<br/><br/>**security_level=1** : Le chiffrement whitebox basé sur le logiciel est requis.|

### <a name="question"></a>Question

Pourquoi le téléchargement de contenu prend-il tant de temps ?

### <a name="answer"></a>Réponse

Il existe deux façons d’améliorer la vitesse du téléchargement :

1.  Activer le CDN afin que les utilisateurs finaux soient plus susceptibles d’atteindre le CDN au lieu du point de terminaison de streaming pour le téléchargement du contenu. Si l’utilisateur atteint un point de terminaison de streaming, chaque segment HLS ou fragment DASH est dynamiquement empaqueté et chiffré. Bien que cette latence soit dans une échelle de millisecondes pour chaque segment/fragment, lorsque vous disposez d’une vidéo d’une durée d’une heure, la latence cumulée peut être volumineuse, entraînant un téléchargement plus long.
2.  Fournir aux utilisateurs finaux la possibilité de télécharger de manière sélective des couches de qualité vidéo et des pistes audio plutôt que tout le contenu. En ce qui concerne le mode hors connexion, il est inutile de télécharger toutes les couches de qualité. Il existe deux moyens de parvenir à cet objectif :
    1.  Contrôlé par le client : sélection automatique d’applications de lecteur ou sélection utilisateur de la couche de qualité vidéo et des pistes audio à télécharger ;
    2.  Contrôlé par le service : il est possible d’utiliser la fonctionnalité manifeste dynamique dans Azure Media Services pour créer un filtre (global), ce qui limite la sélection de HLS ou DASH MPD à une couche unique de qualité vidéo et des pistes audio sélectionnées. Ainsi, l’URL de téléchargement présentée aux utilisateurs finaux inclut ce filtre.

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="summary"></a>Résumé

Cet article explique comment mettre en œuvre la lecture en mode hors connexion pour le contenu DASH protégé par Widevine sur les appareils Android.  L’article répond également à certaines questions fréquentes sur la diffusion en continu du contenu protégé de Widevine.
