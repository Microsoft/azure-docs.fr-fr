---
title: Encodage vidéo et audio avec Media Services
titleSuffix: Azure Media Services
description: Cet article explique comment encoder des fichiers vidéo et audio avec Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18
ms.openlocfilehash: 8533c99011232385e31f4e698743bbb1c6c00bc5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89300202"
---
# <a name="encoding-video-and-audio-with-media-services"></a>Encodage vidéo et audio avec Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Le terme encodage dans Media Services s’applique au processus de conversion de fichiers contenant de la vidéo et/ou audio numérique d’un format standard vers un autre, dans le but de (a) réduire la taille des fichiers et/ou (b) produire un format compatible avec un large éventail d’appareils et d’applications. Ce processus est également appelé compression vidéo ou transcodage. Pour une discussion plus approfondie de ces concepts, consultez [Data compression](https://en.wikipedia.org/wiki/Data_compression) et [What Is Encoding and Transcoding?](https://www.streamingmedia.com/Articles/Editorial/What-Is-/What-Is-Encoding-and-Transcoding-75025.aspx).

Les vidéos sont généralement fournies aux appareils et aux applications par [téléchargement progressif](https://en.wikipedia.org/wiki/Progressive_download) ou par le biais de [streaming à débit adaptatif](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming).

> [!IMPORTANT]
> Media Services ne facture pas les travaux annulés ou erronés. Par exemple, un travail qui a atteint 50 % de progression et qui est annulé n’est pas facturé à hauteur de 50 % des minutes du travail. Vous n’êtes facturé que pour les travaux terminés.

* Pour le téléchargement progressif, vous pouvez utiliser Azure Media Services afin de convertir un fichier multimédia numérique (mezzanine) en fichier [MP4](https://en.wikipedia.org/wiki/MPEG-4_Part_14) qui contient la vidéo encodée avec le codec [H.264](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC) et le son encodé avec le codec [AAC](https://en.wikipedia.org/wiki/Advanced_Audio_Coding). Ce fichier MP4 est écrit dans un actif multimédia dans votre compte de stockage. Vous pouvez utiliser les SDK ou les API Stockage Azure (par exemple l’[API REST de stockage](../../storage/common/storage-rest-api-auth.md) ou le [SDK .NET](../../storage/blobs/storage-quickstart-blobs-dotnet.md)) pour télécharger le fichier directement. Si vous avez créé l’actif multimédia de sortie avec un nom de conteneur spécifique dans le stockage, utilisez cet emplacement. Sinon, vous pouvez utiliser Media Services pour [lister les URL de conteneurs d’actifs multimédias](/rest/api/media/assets/listcontainersas). 
* Pour préparer le contenu pour une diffusion en streaming à débit adaptatif, le fichier mezzanine doit être encodé à plusieurs débits (du plus élevé au plus faible). Pour garantir une transition appropriée de la qualité, le débit et la résolution de la vidéo sont réduits en parallèle. Il en résulte ce qu’on appelle une échelle d’encodage, c’est-à-dire un tableau de résolutions et de débits (voir [Échelle de débit adaptatif générée automatiquement](autogen-bitrate-ladder.md)). Vous pouvez utiliser Media Services pour encoder vos fichiers mezzanine à plusieurs débits binaires. Dans ce cas, vous obtiendrez un ensemble de fichiers MP4 et de fichiers de configuration de diffusion en continu associés écrits dans une ressource de votre compte de stockage. Vous pouvez ensuite utiliser la fonctionnalité d’[empaquetage dynamique](dynamic-packaging-overview.md) dans Media Services pour diffuser la vidéo par le biais de protocoles de streaming tels que [MPEG-DASH](https://en.wikipedia.org/wiki/Dynamic_Adaptive_Streaming_over_HTTP) et [HLS](https://en.wikipedia.org/wiki/HTTP_Live_Streaming). Vous devrez pour cela créer un [localisateur de streaming](streaming-locators-concept.md) et générer des URL de streaming correspondant aux protocoles pris en charge, qui peuvent ensuite être remises aux appareils/applications en fonction de leurs fonctionnalités.

Le diagramme suivant illustre le workflow d’encodage à la demande avec l’empaquetage dynamique.

![Flux de travail pour l’encodage à la demande avec l’empaquetage dynamique](./media/dynamic-packaging-overview/media-services-dynamic-packaging.svg)

Cette rubrique vous explique comment encoder votre contenu avec Media Services v3.

## <a name="transforms-and-jobs"></a>Transformations et travaux

Pour encoder avec Media Services v3, vous devez créer une [transformation](/rest/api/media/transforms) et un [travail](/rest/api/media/jobs). La transformation définit une recette à appliquer pour vos paramètres et sorties d’encodage ; le travail est une instance de la recette. Pour plus d’informations, consultez [Transformations et travaux](transforms-jobs-concept.md).

Lors de l’encodage avec Media Services, vous utilisez des préréglages pour indiquer comment traiter les fichiers multimédias en entrée. Dans Media Services v3, vous utilisez l’encodeur standard pour coder vos fichiers. Par exemple, vous pouvez spécifier la résolution vidéo et/ou le nombre de canaux audio souhaité dans le contenu encodé.

Vous pouvez commencer rapidement à utiliser un des préréglages intégrés recommandés et basés sur les bonnes pratiques du secteur, ou vous pouvez choisir de créer un préréglage personnalisé pour les besoins de votre scénario ou de votre appareil. Pour plus d’informations, consultez [Encoder avec une transformation personnalisée](customize-encoder-presets-how-to.md).

À partir de janvier 2019, en cas de codage avec l’encodeur standard pour produire des fichiers MP4, un nouveau fichier .mpi est généré et ajouté à la ressource de sortie. Ce fichier MPI est destiné à améliorer les performances pour les scénarios d’[empaquetage dynamique](dynamic-packaging-overview.md) et de diffusion en continu.

> [!NOTE]
> Vous ne devez ni modifier ni supprimer le fichier MPI, ni dépendre de l'existence (ou non) d'un tel fichier dans votre service.

### <a name="creating-job-input-from-an-https-url"></a>Créer une entrée de travail à partir d’une URL HTTPS

Quand vous soumettez des travaux pour traiter vos vidéos, vous devez indiquer à Media Services où trouver la vidéo d’entrée. L’une des options consiste à spécifier une URL HTTPS en tant qu’entrée de travail. Actuellement, Media Services v3 ne prend pas en charge l’encodage de transfert mémorisé en bloc sur les URL HTTPS.

#### <a name="examples"></a>Exemples

* [Encoder à partir d’une URL HTTPS avec .NET](stream-files-dotnet-quickstart.md)
* [Encoder à partir d’une URL HTTPS avec REST](stream-files-tutorial-with-rest.md)
* [Encoder à partir d’une URL HTTPS avec l’interface CLI](stream-files-cli-quickstart.md)
* [Encoder à partir d’une URL HTTPS avec Node.js](stream-files-nodejs-quickstart.md)

### <a name="creating-job-input-from-a-local-file"></a>Création d’une entrée de travail à partir d’un fichier local

La vidéo d’entrée peut être stockée sous la forme d’une ressource Media Service, dans laquelle vous créez une ressource d’entrée basée sur un fichier (stocké en local ou dans le stockage Blob Azure).

#### <a name="examples"></a>Exemples

[Encoder un fichier local à l’aide de préréglages intégrés](job-input-from-local-file-how-to.md)

### <a name="creating-job-input-with-subclipping"></a>Création d’une entrée de travail avec sous-clipage

Quand vous encodez une vidéo, vous pouvez spécifier qu’il faut également découper le fichier source et produire une sortie qui possède uniquement une partie souhaitée de la vidéo d’entrée. Cette fonctionnalité peut être utilisée avec n’importe quelle [transformation](/rest/api/media/transforms) qui est générée en utilisant les présélections [BuiltInStandardEncoderPreset](/rest/api/media/transforms/createorupdate#builtinstandardencoderpreset) ou [StandardEncoderPreset](/rest/api/media/transforms/createorupdate#standardencoderpreset).

Vous pouvez faire en sorte de créer un [travail](/rest/api/media/jobs/create) avec un seul clip d’une vidéo à la demande ou d’une archive en temps réel (un événement enregistré). L’entrée de travail peut être un actif multimédia ou une URL HTTPS.

> [!TIP]
> Si vous souhaitez diffuser un sous-clip de votre vidéo sans réencoder la vidéo, utilisez des [manifestes de préfiltrage avec empaquetage dynamique](filters-dynamic-manifest-overview.md).

#### <a name="examples"></a>Exemples

Consultez les exemples :

* [Sous-cliper une vidéo avec .NET](subclip-video-dotnet-howto.md)
* [Sous-cliper une vidéo avec REST](subclip-video-rest-howto.md)

## <a name="built-in-presets"></a>Préréglages intégrés

Media Services prend en charge les préréglages d’encodage intégrés suivants :  

### <a name="builtinstandardencoderpreset"></a>BuiltInStandardEncoderPreset

[BuiltInStandardEncoderPreset](/rest/api/media/transforms/createorupdate#builtinstandardencoderpreset) est utilisé pour définir un préréglage intégré pour l’encodage de la vidéo en entrée avec l’encodeur standard.

Les préréglages suivants sont actuellement pris en charge :

- **EncoderNamedPreset.AACGoodQualityAudio** : produit un fichier MP4 unique contenant seulement le contenu audio stéréo encodé à 192 kbits/s.
- **EncoderNamedPreset.AdaptiveStreaming** (recommandé) : Pour plus d’informations, consultez [Génération automatique d’une échelle de débit binaire](autogen-bitrate-ladder.md).
- **EncoderNamedPreset.ContentAwareEncoding** : expose un préréglage pour l’encodage sensible au contenu. Étant donné un contenu d’entrée, le service tente de déterminer automatiquement le nombre optimal de couches, le débit approprié et les paramètres de résolution pour la remise par streaming adaptatif. Les algorithmes sous-jacents continueront à évoluer au fil du temps. La sortie contiendra des fichiers MP4 avec vidéo et audio entrelacées. Pour plus d’informations, consultez [Encodage sensible au contenu](content-aware-encoding.md).

  > [!NOTE]
  > Veillez à utiliser **ContentAwareEncoding**, et non ContentAwareEncodingExperimental.
- **EncoderNamedPreset.H264MultipleBitrate1080** produit un ensemble de huit fichiers MP4 alignés sur GOP, de 6000 kbits/s à 400 kbits/s, et un contenu audio AAC stéréo. La résolution commence à 1 080p et descend à 360p.
- **EncoderNamedPreset.H264MultipleBitrate720p** produit un ensemble de six fichiers MP4 alignés sur GOP, de 3400 kbits/s à 400 kbits/s, et un contenu audio AAC stéréo. La résolution commence à 720p et descend à 360p.
- **EncoderNamedPreset.H264MultipleBitrateSD** produit un ensemble de cinq fichiers MP4 alignés sur GOP, de 1600 kbits/s à 400 kbits/s, et un contenu audio AAC stéréo. La résolution commence à 480p et descend à 360p.
- **EncoderNamedPreset.H264SingleBitrate1080p** : produit un fichier MP4 où la vidéo est encodée avec le codec H.264 à 6750 kbits/s et une hauteur d’image de 1080 pixels, et le contenu audio stéréo est encodé avec le codec AAC-LC à 64 kbits/s.
- **EncoderNamedPreset.H264SingleBitrate720p** : produit un fichier MP4 où la vidéo est encodée avec le codec H.264 à 4500 kbits/s et une hauteur d’image de 720 pixels, et le contenu audio stéréo est encodé avec le codec AAC-LC à 64 kbits/s.
- **EncoderNamedPreset.H264SingleBitrateSD** : produit un fichier MP4 où la vidéo est encodée avec le codec H.264 à 2200 kbits/s et une hauteur d’image de 480 pixels, et le contenu audio stéréo est encodé avec le codec AAC-LC à 64 kbits/s.

Pour afficher la liste des préréglages la plus récente, consultez [Préréglages intégrés à utiliser pour l’encodage de vidéos](/rest/api/media/transforms/createorupdate#encodernamedpreset).

Pour voir comment sont utilisés les préréglages, consultez [Charger, encoder et diffuser des vidéos en continu](stream-files-tutorial-with-api.md).

### <a name="standardencoderpreset"></a>StandardEncoderPreset

[StandardEncoderPreset](/rest/api/media/transforms/createorupdate#standardencoderpreset) décrit les paramètres utilisés lors de l’encodage de vidéo en entrée avec l’encodeur standard. Utilisez ce préréglage lors de la personnalisation des préréglages de transformation.

#### <a name="considerations"></a>Considérations

Lorsque vous créez des préréglages personnalisés, les considérations suivantes s’appliquent :

- Toutes les valeurs de hauteur et de largeur de contenu AVC doivent être un multiple de quatre.
- Dans Azure Media Services v3, toutes les vitesses d’encodage sont données en bits par seconde. Cela diffère des préréglages avec nos API v2, qui utilisaient des kilobits par seconde comme unité. Par exemple, si la vitesse de transmission dans v2 était de 128 (kilobits/seconde), elle sera définie sur 12 8000 (bits/seconde) dans v3.

### <a name="customizing-presets"></a>Personnalisation des préréglages

Media Services prend entièrement en charge la personnalisation de toutes les valeurs dans les présélections afin de répondre à vos exigences et vos besoins spécifiques de codage. Pour obtenir des exemples qui illustrent comment personnaliser des préréglages d’encodeur, consultez la liste ci-dessous :

#### <a name="examples"></a>Exemples

- [Personnaliser des préréglages avec .NET](customize-encoder-presets-how-to.md)
- [Personnaliser des préréglages avec l’interface CLI](custom-preset-cli-howto.md)
- [Personnaliser des préréglages avec REST](custom-preset-rest-howto.md)

## <a name="preset-schema"></a>Schéma de préréglage

Dans Media Services v3, les présélections sont des entités fortement typées dans l’API elle-même. Vous trouverez la définition « schema » (schéma) pour ces objets dans [Open API Specification (ou Swagger)](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/mediaservices/resource-manager/Microsoft.Media/stable/2018-07-01). Vous pouvez également consulter les définitions prédéfinies (comme **StandardEncoderPreset**) dans [l’API REST](/rest/api/media/transforms/createorupdate#standardencoderpreset), [le Kit de développement logiciel (SDK) .NET](/dotnet/api/microsoft.azure.management.media.models.standardencoderpreset?view=azure-dotnet) (ou d’autres documents de référence sur le Kit de développement logiciel (SDK) Media Services v3).

## <a name="scaling-encoding-in-v3"></a>Mise à l’échelle de l’encodage dans v3

Pour mettre à l’échelle le traitement multimédia, consultez [Mettre à l’échelle avec l’interface CLI](media-reserved-units-cli-how-to.md).

## <a name="billing"></a>Facturation

Media Services ne facture pas les travaux annulés ou erronés. Par exemple, un travail qui a atteint 50 % de progression et qui est annulé n’est pas facturé à hauteur de 50 % des minutes du travail. Vous n’êtes facturé que pour les travaux terminés.

Pour plus d’informations, voir la [tarification](https://azure.microsoft.com/pricing/details/media-services/).

## <a name="ask-questions-give-feedback-get-updates"></a>Poser des questions, envoyer des commentaires, obtenir des mises à jour

Découvrez l’article [Communauté Azure Media Services](media-services-community.md) pour découvrir les différentes façons dont vous pouvez poser des questions, faire des commentaires et obtenir des mises à jour sur Media Services.

## <a name="next-steps"></a>Étapes suivantes

* [Charger, encoder et diffuser en continu à l’aide de Media Services](stream-files-tutorial-with-api.md).
* [Encoder à partir d’une URL HTTPS à l’aide de préréglages intégrés](job-input-from-http-how-to.md).
* [Encoder un fichier local à l’aide de préréglages intégrés](job-input-from-local-file-how-to.md).
* [Créer un préréglage intégré pour les besoins de votre scénario ou votre appareil](customize-encoder-presets-how-to.md).
