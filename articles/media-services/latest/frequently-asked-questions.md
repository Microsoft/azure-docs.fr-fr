---
title: Azure Media Services v3 - Forum aux questions | Microsoft Docs
description: Cet article contient des réponses aux questions les plus fréquemment posées sur Azure Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 50a0fe0fa5dece41ac9e343d5a8939e8d9dc634e
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92426877"
---
# <a name="media-services-v3-frequently-asked-questions"></a>Forum Aux Questions (FAQ) Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Cet article contient des réponses aux questions les plus fréquemment posées sur Azure Media Services v3.

## <a name="general"></a>Général

### <a name="what-are-the-azure-portal-limitations-for-media-services-v3"></a>Quelles sont les limitations du portail Azure pour Media Services v3 ?

Vous pouvez utiliser le [portail Azure](https://portal.azure.com/) pour gérer les événements en direct v3, voir des actifs multimédias et des travaux v3, obtenir des informations sur l’accès aux API, et chiffrer du contenu. <br/>Pour toutes les autres tâches de gestion (par exemple la gestion des transformations et travaux ou l’analyse de contenu v3), utilisez l’[API REST](/rest/api/media/accountfilters), l’[interface CLI](/cli/azure/ams) ou l’un des [SDK](media-services-apis-overview.md#sdks) pris en charge.

Si votre vidéo a été chargée dans le compte Media Services à l’aide de l’API Media Services v3, ou si du contenu a été généré à partir d’une sortie en direct, vous ne verrez pas les boutons **Encoder** , **Analyser** ou **Chiffrer** dans le portail Azure. Utilisez les API Media Services v3 pour effectuer ces tâches.  

### <a name="what-azure-roles-can-perform-actions-on-azure-media-services-resources"></a>Quels sont les rôles Azure pouvant effectuer des actions sur les ressources Azure Media Services ? 

Voir [Contrôle d'accès en fonction du rôle Azure (Azure RBAC) pour les comptes Media Services](rbac-overview.md).

### <a name="how-do-i-stream-to-apple-ios-devices"></a>Comment diffuser en continu vers des appareils Apple iOS ?

Vérifiez que **(format=m3u8-aapl)** est présent à la fin de votre chemin (après la partie **/manifest** de l’URL) pour indiquer au serveur d’origine de la diffusion en continu qu’il doit retourner du contenu HTTP Live Streaming (HLS) utilisable sur des appareils Apple iOS natifs. Pour plus d’informations, consultez [Distribution de contenu](dynamic-packaging-overview.md).

### <a name="how-do-i-configure-media-reserved-units"></a>Comment configurer des unités réservées Multimédia ?

Pour les travaux d’analyse audio et vidéo déclenchés par Media Services v3 ou Video Indexer, nous vous recommandons d’approvisionner votre compte avec des unité réservées Multimédia (MRU) 10 S3. Si vous avez besoin de plus de 10 MRU S3, ouvrez un ticket de support sur le [portail Azure](https://portal.azure.com/).

Pour plus de détails, consultez [Mise à l’échelle du traitement multimédia](media-reserved-units-cli-how-to.md).

### <a name="what-is-the-recommended-method-to-process-videos"></a>Quelle est la méthode recommandée pour traiter les vidéos ?

Utilisez des [transformations](/rest/api/media/transforms) pour configurer des tâches courantes de codage ou d'analyse des vidéos. Chaque transformation décrit une recette, ou un flux de travail de tâches pour le traitement de vos fichiers vidéo ou audio. Un [travail](/rest/api/media/jobs) est la demande réelle envoyée à Media Services pour appliquer la transformation à un contenu vidéo ou audio d’entrée. Une fois la transformation créée, vous pouvez envoyer des travaux en utilisant des API Media Services ou l’un des kits de développement logiciel (SDK) publiés. Pour plus d’informations, consultez [Transformations et travaux](transforms-jobs-concept.md).

### <a name="i-uploaded-encoded-and-published-a-video-why-wont-the-video-play-when-i-try-to-stream-it"></a>J’ai chargé, encodé et publié une vidéo. Pourquoi la vidéo n’est-elle pas lue lorsque j’essaie de la diffuser ?

L’une des raisons les plus courantes est que le point de terminaison de streaming à partir duquel vous essayez de lire n’est pas à l’état En cours d’exécution.

### <a name="how-does-pagination-work"></a>Comment fonctionne la pagination ?

Lors de l’utilisation de la pagination, vous devez toujours utiliser le lien suivant pour énumérer la collection et ne pas dépendre d’une taille de page particulière. Pour voir des détails et des exemples, consultez [Filtrage, tri, pagination](entities-overview.md).

### <a name="what-features-are-not-yet-available-in-azure-media-services-v3"></a>Quelles fonctionnalités ne sont pas encore disponibles dans Azure Media Services v3 ?

Pour plus d’informations, consultez [Différences de fonctionnalités par rapport aux API v2](media-services-v2-vs-v3.md#feature-gaps-with-respect-to-v2-apis).

### <a name="what-is-the-process-of-moving-a-media-services-account-between-subscriptions"></a>Qu'est-ce que le déplacement d’un compte Media Services entre abonnements ?  

Pour en savoir plus, consultez [Déplacement d’un compte Media Services entre abonnements](media-services-account-concept.md).

## <a name="live-streaming"></a>Vidéo en flux continu 

### <a name="how-do-i-stop-the-live-stream-after-the-broadcast-is-done"></a>Comment arrêter le stream en direct une fois la diffusion terminée ?

Vous pouvez choisir une approche côté client ou côté serveur.

#### <a name="client-side"></a>Côté client

Votre application web doit demander à l’utilisateur s’il souhaite mettre fin à la diffusion quand il ferme le navigateur. Il s’agit d’un événement de navigateur que votre application web peut gérer.

#### <a name="server-side"></a>Côté serveur

Vous pouvez surveiller les événements en direct en vous abonnant à Azure Event Grid. Pour plus d’informations, consultez le [schéma d’événement Event Grid](media-services-event-schemas.md#live-event-types).

Vous pouvez :

* [vous abonner](reacting-to-media-services-events.md) aux événements [Microsoft.Media.LiveEventEncoderDisconnected](media-services-event-schemas.md#liveeventencoderdisconnected) au niveau du flux et surveiller qu’aucune reconnexion ne survient pendant un certain temps pour arrêter et supprimer votre événement en direct ;
* [vous abonner](reacting-to-media-services-events.md) aux [événements de pulsation](media-services-event-schemas.md#liveeventingestheartbeat) au niveau de la piste. Si toutes les pistes ont une vitesse de transmission entrante de 0 ou si le dernier timestamp n’augmente plus, vous pouvez arrêter en toute sécurité l’événement en direct. Les événements de pulsation sont générés toutes les 20 secondes pour chaque piste, ce qui peut être un peu trop détaillé.

###  <a name="how-do-i-insert-breaksvideos-and-image-slates-during-a-live-stream"></a>Comment insérer des pauses/vidéos et séquences d’images pendant un flux de données en direct ?

Le Live Encoding Media Services v3 ne prend pas encore en charge l’insertion de séquences de vidéos ou d’images pendant le flux de données en direct. 

Vous pouvez utiliser un [encodeur live local](recommended-on-premises-live-encoders.md) pour changer la vidéo source. De nombreuses applications offrent la possibilité de changer les sources, parmi lesquelles Telestream Wirecast, Switcher Studio (sur iOS) et OBS Studio (application gratuite).

## <a name="content-protection"></a>Protection du contenu

### <a name="should-i-use-aes-128-clear-key-encryption-or-a-drm-system"></a>Dois-je utiliser un chiffrement à clé en clair AES-128 ou un système DRM ?

Les clients se demandent souvent s’ils devraient utiliser un chiffrement AES ou un système DRM. La principale différence entre les deux systèmes est qu’avec le chiffrement AES, la clé de contenu est transmise au client via TLS. Elle est chiffrée en transit, mais sans autre chiffrement (« en clair »). Par conséquent, la clé utilisée pour déchiffrer le contenu peut être lue par le lecteur client et affichée dans la trace réseau du client, en texte en clair. Le chiffrement avec une clé en clair AES-128 convient à des cas d’utilisation dans lesquels l’observateur est fiable (par exemple, le chiffrement de vidéos d’entreprise distribuées dans une société et destinées aux employés).

Par rapport à la clé en clair AES-128, les systèmes DRM comme PlayReady, Widevine et FairPlay fournissent tous un niveau supplémentaire de chiffrement sur la clé utilisée pour déchiffrer le contenu. La clé de contenu est chiffrée à l’aide d’une clé protégée par le runtime DRM, en plus de tout chiffrement au niveau du transport fourni par TLS. De plus, le déchiffrement est pris en charge dans un environnement sécurisé au niveau du système d’exploitation, où il est plus difficile pour un utilisateur malintentionné de commettre une attaque. Nous recommandons l’utilisation de la technologie DRM dans des cas d’utilisation où l’observateur n’est pas nécessairement fiable et où un niveau de sécurité supérieur est requis.

### <a name="how-do-i-show-a-video-to-only-users-who-have-a-specific-permission-without-using-azure-ad"></a>Comment afficher une vidéo uniquement pour les utilisateurs disposant d’une autorisation spécifique sans utiliser Azure AD ?

Vous n’êtes pas obligé d’utiliser un fournisseur de jetons spécifique tel qu’Azure Active Directory (Azure AD). Vous pouvez créer votre propre fournisseur [JWT](https://jwt.io/) (appelé service d’émission de jeton de sécurité ou STS) en utilisant un chiffrement à clé asymétrique. Dans votre STS personnalisé, vous pouvez ajouter des revendications en fonction de votre logique métier.

Assurez-vous que l’émetteur, l’audience et les revendications correspondent exactement entre ce qui se trouve dans JWT et la valeur `ContentKeyPolicyRestriction` utilisée dans `ContentKeyPolicy`.

Pour plus d’informations, consultez [Protéger votre contenu à l’aide du chiffrement dynamique de Media Services](content-protection-overview.md).

### <a name="how-and-where-did-i-get-a-jwt-token-before-using-it-to-request-a-license-or-key"></a>Comment et où obtenir un jeton JWT avant de l’utiliser pour demander une licence ou une clé ?

Pour un environnement de production, vous devez disposer d’un service d’émission de jeton de sécurité (STS, Secure Token Service), c’est-à-dire un service web, qui émet un jeton JWT en réponse à une requête HTTPS. Dans le cas d’un environnement de test, vous pouvez utiliser le code figurant dans la méthode `GetTokenAsync` définie dans [Program.cs](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs).

Après l’authentification d’un utilisateur, le lecteur adresse une requête au STS afin d’obtenir ce jeton et l’attribuer comme valeur de jeton. Vous pouvez utiliser [l’API Lecteur multimédia Azure](https://amp.azure.net/libs/amp/latest/docs/).

Pour un exemple d’exécution de STS avec une clé symétrique ou asymétrique, consultez [Outil JWT](https://aka.ms/jwt). Pour un exemple de lecteur basé sur le Lecteur multimédia Azure utilisant un tel jeton JWT, consultez [Outil de test multimédia Azure](https://aka.ms/amtest) (développez le lien **player_settings** pour voir l’entrée du jeton).

### <a name="how-do-i-authorize-requests-to-stream-videos-with-aes-encryption"></a>Comment autoriser les requêtes de diffusion en continu de vidéos avec chiffrement AES ?

L’approche correcte consiste à utiliser STS. Dans STS, en fonction du profil utilisateur, ajoutez différentes revendications (par exemple, « Utilisateur Premium », « Utilisateur De base », « Utilisateur de la version d’évaluation gratuite »). La définition de différentes revendications dans un jeton JWT permet à l’utilisateur de visualiser des contenus distincts. Pour différents contenus ou ressources, `ContentKeyPolicyRestriction` aura la valeur `RequiredClaims` correspondante.

Utilisez l’API Azure Media Services pour la configuration de la livraison de licences/clés et pour le chiffrement de vos éléments multimédias (comme indiqué dans [cet exemple](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES/Program.cs)).

Pour plus d'informations, consultez les pages suivantes :

- [Présentation de la protection du contenu](content-protection-overview.md)
- [Concevoir un système de protection de contenu multi-DRM avec contrôle d’accès](design-multi-drm-system-with-access-control.md)

### <a name="should-i-use-http-or-https"></a>Dois-je utiliser le protocole HTTP ou HTTPs ?
L’application de lecteur MVC ASP.NET doit prendre en charge les éléments suivants :

* Authentification des utilisateurs via Azure AD, sous HTTPS.
* Échange de jeton JWT entre le client et Azure AD, sous HTTPS.
* Acquisition de licence DRM par le client, qui doit être sous HTTPS si la distribution de licences est assurée par Azure Media Services. La suite de produits PlayReady n’impose pas le format HTTPS pour la distribution de licences. Si votre serveur de licences PlayReady se trouve en dehors de Media Services, vous pouvez utiliser HTTP ou HTTPS.

Puisque l’application de lecteur ASP.NET utilise le protocole HTTPS en tant que meilleure pratique, Media Player se trouve sur une page sous HTTPS. Le protocole HTTP est cependant privilégié pour la diffusion en continu, ce qui signifie que vous devez tenir compte des problèmes de contenu mixte suivants :

* Le navigateur ne permet pas un contenu mixte. Mais vous pouvez utiliser des plug-ins tels que Silverlight et OSMF pour Smooth et DASH. Du contenu mixte pose un problème de sécurité en raison de la menace que constitue la possibilité d’injecter du code JavaScript malveillant mettant en danger les données du client. Les navigateurs bloquent cette fonctionnalité par défaut. Le seul moyen de contourner ce problème est d’autoriser tous les domaines (HTTPS ou HTTP) côté serveur (origine). Il ne s’agit probablement pas d’une idée judicieuse.
* Évitez le contenu mixte. L’application de lecteur et Media Player doivent utiliser HTTP ou HTTPS. Si vous lisez du contenu mixte, la technologie SilverlightSS exige la suppression d’un avertissement de contenu mixte. La technologie flashSS traite le contenu mixte sans émettre le moindre avertissement.
* Si votre point de terminaison de diffusion a été créé avant août 2014, il ne prend pas en charge HTTPS. Dans ce cas, créez et utilisez un nouveau point de terminaison pour le protocole HTTPS.

### <a name="what-about-live-streaming"></a>Qu’en est-il de la diffusion en continu ?

Vous pouvez utiliser exactement la même conception et la même implémentation pour protéger un service de diffusion en continu dans Media Services, en traitant l’élément multimédia associé à un programme comme un élément multimédia VOD. Pour pouvoir protéger du contenu en direct via la fonctionnalité multi-DRM, appliquez la même procédure d’installation/de traitement à l’élément multimédia que s’il s’agissait d’un élément VOD avant de l’associer à la sortie en direct.

### <a name="what-about-license-servers-outside-media-services"></a>Qu’en est-il des serveurs de licences hors Media Services ?

Souvent, les clients disposent d’une batterie de serveurs qu’ils hébergent dans leur propre centre de données ou dans un centre de données hébergé par un fournisseur de services DRM. Avec la protection de contenu Media Services, vous pouvez fonctionner en mode hybride. Le contenu peut être hébergé et protégé de façon dynamique dans Media Services, tandis que les licences DRM sont délivrées par des serveurs situés en dehors de Media Services. Dans ce cas, envisagez les modifications suivantes :

* Le service STS doit émettre des jetons acceptables et pouvant être vérifiés par la batterie de serveurs de licence. Par exemple, les serveurs de licences Widevine fournis par Axinom exigent un jeton JWT spécifique contenant un message d’octroi de droit. Vous devez disposer d’un STS pour émettre un tel jeton JWT. 
* Vous n’avez plus besoin de configurer le service de distribution de licences dans Media Services. Vous devez fournir les URL d’acquisition de la licence (pour PlayReady, Widevine et FairPlay) au moment où vous configurez `ContentKeyPolicy`.

> [!NOTE]
> Widevine est un service fourni par Google, soumis aux conditions de service et à la politique de confidentialité de Google.

## <a name="media-services-v2-vs-v3"></a>Media Services v2 par rapport à Media Services v3 

### <a name="can-i-use-the-azure-portal-to-manage-v3-resources"></a>Puis-je utiliser le portail Azure pour gérer des ressources v3 ?

Vous pouvez actuellement utiliser le portail [Azure](https://portal.azure.com/) pour :

* Gérer les [événements en direct](live-events-outputs-concept.md) dans Media Services v3. 
* Afficher (non gérer) des [éléments multimédias](assets-concept.md) v3. 
* [Obtenir des informations sur l’accès aux API](./access-api-howto.md). 

Pour toutes les autres tâches de gestion (par exemple, les [transformations et travaux](transforms-jobs-concept.md) et la [protection de contenu](content-protection-overview.md)), utilisez l’[API REST](/rest/api/media/), [Azure CLI](/cli/azure/ams) ou l’un des [kits de développement logiciel (SDK)](media-services-apis-overview.md#sdks) pris en charge.

### <a name="is-there-an-assetfile-concept-in-v3"></a>Existe-t-il un concept de fichier d’élément multimédia dans v3 ?

Le concept `AssetFile` a été supprimé de l’API Media Services afin de séparer Media Services de la dépendance du Kit de développement logiciel (SDK) Stockage. C’est désormais Stockage Azure, non plus Media Services, qui conserve les informations appartenant au Kit de développement logiciel (SDK) Stockage. 

Pour plus d’informations, consultez [Migrer vers Azure Media Services v3](media-services-v2-vs-v3.md).

### <a name="where-did-client-side-storage-encryption-go"></a>Où le chiffrement du stockage côté client se fait-il ?

Nous recommandons dorénavant d’utiliser le chiffrement de stockage côté serveur (qui est activé par défaut). Pour plus d’informations, consultez [Azure Storage Service Encryption pour les données au repos](../../storage/common/storage-service-encryption.md).

## <a name="offline-streaming"></a>Streaming hors connexion

### <a name="fairplay-streaming-for-ios"></a>Streaming FairPlay pour iOS

Les questions fréquemment posées suivantes vous aideront à résoudre les problèmes rencontrés avec le diffusion en continu FairPlay hors connexion pour iOS.

#### <a name="why-does-only-audio-play-but-not-video-during-offline-mode"></a>Pourquoi seul l’audio est-il lu en mode hors connexion en non la vidéo ?

Ce comportement semble être conçu tel quel dans l’application en exemple. Lorsqu’une autre piste audio est présente (ce qui est le cas pour HLS), en mode hors connexion, iOS 10 et iOS 11 passent par défaut à l’autre piste audio. Pour compenser ce comportement en mode hors connexion FPS, supprimez l’autre piste audio du flux. Pour effectuer cette opération sur Media Services, ajoutez le filtre de manifeste dynamique **audio-only=false**. En d’autres termes, une URL HLS se termine par **.ism/manifest(format=m3u8-aapl,audio-only=false)** . 

#### <a name="why-does-it-still-play-audio-only-without-video-during-offline-mode-after-i-add-audio-onlyfalse"></a>Pourquoi l’audio est toujours lu sans vidéo en mode hors connexion après l’ajout de audio-only=false ?

Selon la conception de la clé de cache pour le réseau de distribution de contenu, le contenu peut être mis en cache. Videz le cache.

#### <a name="is-fps-offline-mode-supported-on-ios-11-in-addition-to-ios-10"></a>Le mode hors connexion FPS est-il pris en charge sur iOS 11 en plus d’iOS 10 ?

Oui. Le mode hors connexion FPS est pris en charge sur iOS 10 et iOS 11.

#### <a name="why-cant-i-find-the-document-offline-playback-with-fairplay-streaming-and-http-live-streaming-in-the-fps-server-sdk"></a>Pourquoi ne puis-je pas trouver le document « Lecture hors connexion avec FairPlay Streaming et HTTP Live Streaming » dans le SDK du serveur FPS ?

Depuis le Kit SDK du serveur FPS version 4, ce document a été fusionné avec le « Guide de programmation FairPlay Streaming ».

#### <a name="what-is-the-downloadedoffline-file-structure-on-ios-devices"></a>Quelle est la structure du fichier téléchargé/hors connexion sur les appareils iOS ?

La structure du fichier téléchargé sur un appareil iOS ressemble à la capture d’écran suivante. Le dossier `_keys` stocke les licences FPS téléchargées, avec un fichier de magasin pour chaque hôte de service de licence. Le dossier `.movpkg` stocke le contenu audio et vidéo. 

Le premier dossier dont le nom se termine par un tiret suivi par un nombre comprend du contenu vidéo. La valeur numérique est la bande passante maximale des rendus de vidéo. Le deuxième dossier dont le nom se termine par un tiret suivi de 0 contient un contenu audio. Le troisième dossier nommé `Data` contient la sélection principale du contenu FPS. Enfin, boot.xml fournit une description complète du contenu du dossier `.movpkg`. 

![Structure de fichiers hors connexion pour l’exemple d’application iOS FairPlay](media/offline-fairplay-for-ios/offline-fairplay-file-structure.png)

Voici un exemple de fichier Boot. xml :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<HLSMoviePackage xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://apple.com/IMG/Schemas/HLSMoviePackage" xsi:schemaLocation="http://apple.com/IMG/Schemas/HLSMoviePackage /System/Library/Schemas/HLSMoviePackage.xsd">
  <Version>1.0</Version>
  <HLSMoviePackageType>PersistedStore</HLSMoviePackageType>
  <Streams>
    <Stream ID="1-4DTFY3A3VDRCNZ53YZ3RJ2NPG2AJHNBD-0" Path="1-4DTFY3A3VDRCNZ53YZ3RJ2NPG2AJHNBD-0" NetworkURL="https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/QualityLevels(127000)/Manifest(aac_eng_2_127,format=m3u8-aapl)">
      <Complete>YES</Complete>
    </Stream>
    <Stream ID="0-HC6H5GWC5IU62P4VHE7NWNGO2SZGPKUJ-310656" Path="0-HC6H5GWC5IU62P4VHE7NWNGO2SZGPKUJ-310656" NetworkURL="https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/QualityLevels(161000)/Manifest(video,format=m3u8-aapl)">
      <Complete>YES</Complete>
    </Stream>
  </Streams>
  <MasterPlaylist>
    <NetworkURL>https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/manifest(format=m3u8-aapl,audio-only=false)</NetworkURL>
  </MasterPlaylist>
  <DataItems Directory="Data">
    <DataItem>
      <ID>CB50F631-8227-477A-BCEC-365BBF12BCC0</ID>
      <Category>Playlist</Category>
      <Name>master.m3u8</Name>
      <DataPath>Playlist-master.m3u8-CB50F631-8227-477A-BCEC-365BBF12BCC0.data</DataPath>
      <Role>Master</Role>
    </DataItem>
  </DataItems>
</HLSMoviePackage>
```

### <a name="widevine-streaming-for-android"></a>Streaming Widevine pour Android

#### <a name="how-can-i-deliver-persistent-licenses-offline-enabled-for-some-clientsusers-and-non-persistent-licenses-offline-disabled-for-others-do-i-have-to-duplicate-the-content-and-use-separate-content-keys"></a>Comment puis-je remettre des licences persistantes (activées en mode hors connexion) à certains clients/utilisateurs et des licences non persistantes (désactivées en mode hors connexion) pour d’autres ? Dois-je dupliquer le contenu et utiliser une clé de contenu distincte ?

Étant donné que Media Services v3 permet qu’un élément multimédia ait plusieurs instances `StreamingLocator`, vous pouvez avoir :

* Une instance `ContentKeyPolicy` avec `license_type = "persistent"`, `ContentKeyPolicyRestriction` avec revendication sur `"persistent"` et son `StreamingLocator`.
* Une autre instance `ContentKeyPolicy` avec `license_type="nonpersistent"`, `ContentKeyPolicyRestriction` avec revendication sur `"nonpersistent` et son `StreamingLocator`.
* Deux instances de `StreamingLocator` dont les valeurs `ContentKey` diffèrent.

En fonction de la logique métier du service d’émission de jeton de sécurité personnalisé, différentes revendications peuvent être émises dans le jeton JWT. Avec le jeton, seule la licence correspondante peut être obtenue et seule l’URL correspondante peut être lue.

#### <a name="what-is-the-mapping-between-the-widevine-and-media-services-drm-security-levels"></a>Quelle est la correspondance entre le niveau de sécurité de Widevine et celui offert par la gestion des droits numériques (DRM) de Media Services ?

La « Vue d’ensemble de l’architecture DRM Widevine de Google » définit trois niveaux de sécurité. Toutefois, la [Documentation Azure Media Services sur le modèle de licence Widevine](widevine-license-template-overview.md) fait état de cinq niveaux de sécurité (exigences de robustesse du client pour la lecture). Cette section décrit la correspondance entre les niveaux de sécurité.

Les deux ensembles de niveaux de sécurité sont définis par Google Widevine. La différence réside dans le niveau d’utilisation : architecture ou API. Les cinq niveaux de sécurité sont utilisés dans l’API Widevine. L’objet `content_key_specs` qui contient `security_level` est désérialisé et transmis au service de remise globale Widevine par le service de licence Widevine Azure Media Services. Le tableau ci-après montre le mappage entre les deux ensembles de niveaux de sécurité.

| **Niveaux de sécurité définis dans l’architecture de Widevine** |**Niveaux de sécurité utilisés dans l’API Widevine**|
|---|---| 
| **Niveau de sécurité 1**  : tous les traitements, chiffrements et contrôles du contenu sont effectués au sein de l’environnement TEE (Trusted Execution Environment). En ce qui concerne certains modèles de mise en œuvre, le traitement de sécurité peut être effectué dans différentes processeurs.|**security_level=5**  : Le chiffrement, le décodage et le traitement du support (compressé et décompressé) doivent être gérés dans un environnement TEE doté de matériel.<br/><br/>**security_level=4**  : Le chiffrement et le décodage du contenu doivent être effectués dans un environnement TEE doté de matériel.|
**Niveau de sécurité 2**  : Le chiffrement (mais pas le traitement vidéo) est effectué dans l’environnement TEE. Les mémoires tampons déchiffrées sont retournées au domaine d’application et traitées par un matériel ou un logiciel vidéo distinct. Toutefois, au niveau 2, les informations de chiffrement sont toujours traitées uniquement dans l’environnement TEE.| **security_level=3**  : Les opérations clés de matériel et de chiffrement doivent être effectuées dans un environnement TEE doté de matériel. |
| **Niveau de sécurité 3**  : Il n’y a pas d’environnement TEE sur l’appareil. Des mesures appropriées peuvent être prises pour protéger les informations de chiffrement et le contenu déchiffré sur le système d’exploitation hôte. Une implémentation de niveau 3 peut également inclure un moteur de chiffrement matériel. Cependant, celui-ci n’améliore que les performances, pas la sécurité. | **security_level=2**  : le chiffrement logiciel et un décodeur masqué sont nécessaires.<br/><br/>**security_level=1**  : Le chiffrement whitebox basé sur le logiciel est requis.|

#### <a name="why-does-content-download-take-so-long"></a>Pourquoi le téléchargement de contenu prend-il tant de temps ?

Il existe deux façons d’améliorer la vitesse du téléchargement :

* Activer un réseau de distribution de contenu afin que les utilisateurs soient plus susceptibles d’atteindre celui-ci au lieu du point de terminaison de streaming d’origine pour le téléchargement du contenu. Si un utilisateur atteint un point de terminaison de streaming, chaque segment HLS ou fragment DASH est empaqueté et chiffré de façon dynamique. Bien que cette latence soit dans une échelle de millisecondes pour chaque segment/fragment, lorsque vous disposez d’une vidéo d’une durée d’une heure, la latence cumulée peut être volumineuse, entraînant un téléchargement plus long.
* Offrir aux utilisateurs finaux la possibilité de télécharger de manière sélective des couches de qualité vidéo et des pistes audio plutôt que tout le contenu. En ce qui concerne le mode hors connexion, il est inutile de télécharger toutes les couches de qualité. Il existe deux moyens de parvenir à cet objectif :

  * Contrôle par le client : L’application de lecteur sélectionne automatiquement ou l’utilisateur sélectionne la couche de qualité vidéo et les pistes audio à télécharger.
  * Contrôle par le service : Vous pouvez utiliser la fonctionnalité de manifeste dynamique dans Azure Media Services pour créer un filtre (global) qui limite la sélection HLS ou la MPD (Media Presentation Description) DASH à une couche de qualité vidéo et des pistes audio sélectionnées. Ensuite, l’URL de téléchargement présentée aux utilisateurs inclut ce filtre.

## <a name="next-steps"></a>Étapes suivantes

[Vue d’ensemble de Media Services v3](media-services-overview.md)