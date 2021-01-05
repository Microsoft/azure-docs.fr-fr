---
title: Notes de publication Azure Media Services v3 | Microsoft Docs
description: Pour vous informer des développements les plus récents, cet article détaille les toutes dernières mises à jour concernant Azure Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: na
ms.topic: article
ms.date: 10/21/2020
ms.author: inhenkel
ms.openlocfilehash: 4ddb02bbcf284566550517f8fa343a728e2275a9
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96483625"
---
# <a name="azure-media-services-v3-release-notes"></a>Notes de publication Azure Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

>Soyez notifié de la disponibilité des mises à jour sur cette page en faisant un copier-coller de cette URL : `https://docs.microsoft.com/api/search/rss?search=%22Azure+Media+Services+v3+release+notes%22&locale=en-us` dans votre lecteur de flux RSS.

Pour vous informer des développements les plus récents, cet article détaille les thèmes suivants :

* Versions les plus récentes
* Problèmes connus
* Résolution des bogues
* Fonctionnalités dépréciées

## <a name="known-issues"></a>Problèmes connus

> [!NOTE]
> Vous pouvez utiliser le [portail Azure](https://portal.azure.com/) pour gérer les [événements en direct](live-events-outputs-concept.md) v3, voir des [actifs multimédias](assets-concept.md) et des travaux v3, obtenir des informations sur l’accès aux API, et chiffrer du contenu. Pour toutes les autres tâches de gestion (par exemple gérer les transformations et les travaux), utilisez l’[API REST](/rest/api/media/accountfilters), l’[interface CLI](/cli/azure/ams) ou l’un des [kits SDK](media-services-apis-overview.md#sdks) pris en charge.
>
> Pour plus d’informations, consultez [les limitations du portail Azure pour Media Services v3](frequently-asked-questions.md#what-are-the-azure-portal-limitations-for-media-services-v3).

## <a name="october-2020"></a>Octobre 2020

### <a name="basic-audio-analysis"></a>Analyse audio De base
La présélection de l’analyse audio comprend désormais un niveau de tarification en mode De base. Le nouveau mode De base de l’analyseur audio offre une option plus économique pour extraire la transcription vocale et mettre en forme les légendes et les sous-titres de sortie. Ce mode effectue une transcription de la parole en texte et la génération d’un fichier de sous-titres/CC au format VTT. La sortie de ce mode comprend un fichier JSON Insights incluant uniquement les mots clés, la transcription et les informations relatives au minutage. La détection automatique de la langue et la diarisation de l’orateur ne sont pas incluses dans ce mode. Consultez la [liste des langues prises en charge](analyzing-video-audio-files-concept.md#built-in-presets).

Les clients utilisant Indexer v1 et Indexer v2 doivent migrer vers la présélection de l’analyse audio De base.

Pour plus d’informations sur le mode De base de l’analyseur audio, consultez [Analyse des fichiers vidéo et audio](analyzing-video-audio-files-concept.md).  Pour savoir comment utiliser le mode De base de l’analyseur audio avec l’API REST, consultez [Créer une transformation audio De base](how-to-create-basic-audio-transform.md).

## <a name="live-events"></a>Événements en direct

Les mises à jour de la plupart des propriétés sont désormais autorisées lorsque les événements en direct sont arrêtés. En outre, les utilisateurs sont autorisés à spécifier un préfixe pour le nom d’hôte statique des URL d’entrée et d’aperçu de l’événement en direct. VanityUrl est désormais appelé `useStaticHostName` pour mieux refléter l’objectif de la propriété.

Les événements en direct disposent désormais d’un état En attente.  Consultez [Événements en direct et sorties en direct dans Media Services](./live-events-outputs-concept.md).

Un événement en direct prend en charge la réception de différentes proportions d’entrée. Le mode Extension permet aux clients de spécifier le comportement d’extension de la sortie.

L’encodage en direct permet désormais de générer des fragments de fréquence d’images clés fixes entre 0,5 à 20 secondes.

## <a name="accounts"></a>Comptes

> [!WARNING]
> Si vous créez un compte Media Services avec la version d’API du 5 janvier 2020, il ne fonctionnera pas avec RESTv2. 

## <a name="august-2020"></a>Août 2020

### <a name="dynamic-encryption"></a>Chiffrement dynamique
La prise en charge du chiffrement PIFF 1.1 (PlayReady Protected Interoperable File Format) hérité est désormais disponible dans le Packager dynamique. Cela offre la prise en charge des jeux Smart TV hérités de Samsung et LG qui implémentaient les premières ébauches de la norme CENC (Common Encryption) publiées par Microsoft.  Le format PIFF 1.1 est également connu comme le format de chiffrement précédemment pris en charge par la bibliothèque de client Silverlight. Aujourd’hui, le seul scénario de cas d’usage pour ce format de chiffrement est le ciblage du marché Smart TV hérité où il reste un nombre non négligeable de Smart TV dans certaines régions qui prennent uniquement en charge Smooth Streaming avec le chiffrement PIFF 1.1. 

Pour utiliser la nouvelle prise en charge du chiffrement PIFF 1.1, remplacez la valeur de chiffrement par « piff » dans le chemin d’URL du localisateur de streaming. Pour plus d’informations, consultez la [vue d’ensemble de protection du contenu](content-protection-overview.md).
Par exemple : `https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(encryption=piff)`|

> [!NOTE]
> La prise en charge de PIFF 1.1 est fournie en tant que solution de compatibilité descendante pour les Smart TV (Samsung, LG) qui implémentaient la version « Silverlight » précoce de la norme CENC. Nous vous recommandons d’utiliser le format PIFF uniquement là où il est nécessaire pour la prise en charge des Smart TV Samsung ou LG livrés entre 2009 et 2015 qui prenaient en charge la version PIFF 1.1 du chiffrement PlayReady. 

## <a name="july-2020"></a>Juillet 2020

### <a name="live-transcriptions"></a>Transcriptions en direct

Les transcriptions en direct prennent désormais en charge 19 langues et 8 régions.

### <a name="protecting-your-content-with-media-services-and-azure-ad"></a>Protection de votre contenu avec Media Services et Azure AD

Nous avons publié un tutoriel intitulé [Protection de contenu de bout en bout avec Azure AD](./azure-ad-content-protection.md).

### <a name="high-availability"></a>Haute disponibilité

Nous avons publié une [vue d’ensemble](./media-services-high-availability-encoding.md) et un [exemple](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/master/HighAvailabilityEncodingStreaming) pour la Haute disponibilité avec Media Services et VOD (vidéo à la demande).

## <a name="june-2020"></a>Juin 2020

### <a name="live-video-analytics-on-iot-edge-preview-release"></a>Publication en préversion de Live Video Analytics sur IoT Edge

La préversion de Live Video Analytics sur IoT Edge est devenue publique. Pour plus d’informations, consultez les [notes de publication](../live-video-analytics-edge/release-notes.md).

Live Video Analytics sur IoT Edge est une extension de la famille Media Services. Elle vous permet d’analyser la vidéo en direct avec des modèles AI de votre choix sur vos propres appareils périphériques, et éventuellement de capturer et d’enregistrer cette vidéo. Vous pouvez désormais créer des applications avec une analytique vidéo en temps réel à la périphérie, sans vous soucier de la complexité liée à la création et à l’exploitation d’un pipeline vidéo en direct.

## <a name="may-2020"></a>Mai 2020

Azure Media Services est généralement disponible dans les régions suivantes : « Allemagne Nord », « Allemagne Centre-Ouest », « Suisse Nord » et « Suisse Ouest ». Les clients peuvent déployer Azure Media Services dans ces régions à l’aide du Portail Azure.

## <a name="april-2020"></a>Avril 2020

### <a name="improvements-in-documentation"></a>Améliorations de la documentation

La documentation relative au Lecteur multimédia Azure a été migrée vers la [documentation Azure](../azure-media-player/azure-media-player-overview.md).

## <a name="january-2020"></a>Janvier 2020

### <a name="improvements-in-media-processors"></a>Améliorations apportées aux processeurs multimédias

- Amélioration de la prise en charge des sources entrelacées dans l’analyse vidéo : ces contenus sont désormais désentrelacés correctement avant d’être envoyés aux moteurs d’inférence.
- Lors de la génération de miniatures avec le mode « optimal », l’encodeur effectue désormais une recherche au-delà de 30 secondes pour sélectionner une image qui n’est pas monochrome.

### <a name="azure-government-cloud-updates"></a>Mises à jour Azure Government Cloud

Media Services est proposé en disponibilité générale dans les régions Azure Government suivantes : *USGov Arizona* et *USGov Texas*.

## <a name="december-2019"></a>Décembre 2019

Ajout de la prise en charge du CDN pour les en-têtes *Origin-Assist Prefetch* pour le streaming à la demande et en direct ; disponible pour les clients disposant d’un contrat direct avec le CDN Akamai. La fonctionnalité Origin-Assist CDN-Prefetch implique les échanges d’en-têtes HTTP suivants entre le CDN Akamai et l’origine Azure Media Services :

|En-tête HTTP|Valeurs|Expéditeur|Destinataire|Objectif|
| ---- | ---- | ---- | ---- | ----- |
|CDN-Origin-Assist-Prefetch-Enabled | 1 (par défaut) ou 0 |CDN|Origine|Pour indiquer que la prérécupération est activée pour le CDN|
|CDN-Origin-Assist-Prefetch-Path| Exemple : <br/>Fragments(video=1400000000,format=mpd-time-cmaf)|Origine|CDN|Pour fournir le chemin de prérécupération du CDN|
|CDN-Origin-Assist-Prefetch-Request|1 (demande de prérécupération) ou 0 (demande normale)|CDN|Origine|Pour indiquer que la demande du CDN est une prérécupération|

Pour voir une partie de l’échange d’en-têtes en action, vous pouvez essayer les étapes suivantes :

1. Utilisez Postman ou curl pour émettre une requête destinée à l’origine Media Services pour un segment ou un fragment audio ou vidéo. Veillez à ajouter l’en-tête CDN-Origin-Assist-Prefetch-Enabled : 1 dans la requête.
2. Dans la réponse, vous devez voir l’en-tête CDN-Origin-Assist-Prefetch-Path avec un chemin relatif comme valeur.

## <a name="november-2019"></a>Novembre 2019

### <a name="live-transcription-preview"></a>Transcription en direct (préversion)

La transcription en direct est désormais en préversion publique et peut être utilisée dans la région USA Ouest 2.

La transcription en direct est conçue pour fonctionner conjointement avec les événements en direct comme une fonctionnalité complémentaire.  Elle est prise en charge à la fois pour les événements directs et les événements en direct avec un encodage Standard ou Premium.  Lorsque cette fonctionnalité est activée, le service utilise la fonctionnalité de [reconnaissance vocale](../../cognitive-services/speech-service/speech-to-text.md) de Cognitive Services pour transcrire les paroles du fichier audio entrant en texte. Ce texte, accompagné de fichiers vidéo et audio dans les protocoles MPEG-DASH et HLS, est ensuite mis à disposition pour diffusion. La facturation est basée sur un nouveau compteur complémentaire qui est un coût supplémentaire à l’événement en direct lorsqu’il est dans l’état « En cours d’exécution ».  Pour plus d’informations sur la transcription en direct et la facturation, consultez [Transcription en direct](live-transcription.md)

> [!NOTE]
> Actuellement, la transcription en direct est disponible uniquement en tant que fonctionnalité d’évaluation dans la région USA Ouest 2. Pour l’instant, elle prend seulement en charge la transcription en texte des Mots parlés en anglais (en-us).

### <a name="content-protection"></a>Protection du contenu

La fonctionnalité *Prévention de la relecture de jeton* publiée dans des régions limitées en septembre est désormais disponible dans toutes les régions.
Les clients de Media Services peuvent maintenant définir une limite au nombre de fois où le même jeton peut être utilisé pour demander une clé ou une licence. Pour plus d’informations, consultez [Prévention de la relecture de jeton](content-protection-overview.md#token-replay-prevention).

### <a name="new-recommended-live-encoder-partners"></a>Nouveaux encodeurs live partenaires recommandés

Ajout de la prise en charge des nouveaux encodeurs partenaires recommandés ci-dessous pour le streaming en direct RTMP :

- [Cambria Live 4.3](https://www.capellasystems.net/products/cambria-live/)
- [Caméras GoPro Hero7/8 et Max action](https://gopro.com/help/articles/block/getting-started-with-live-streaming)
- [Restream.io](https://restream.io/)

### <a name="file-encoding-enhancements"></a>Améliorations de l’encodage des fichiers

- Un nouvel encodage prédéfini sensible au contenu est maintenant disponible. Il produit un ensemble de fichiers MP4 alignés sur le groupe d’images à l’aide de l’encodage sensible au contenu. Lorsque du contenu est entré, le service effectue une analyse initiale légère du contenu d’entrée. Il utilise ces résultats pour déterminer le nombre optimal de couches, le débit approprié et les paramètres de résolution pour la remise par diffusion en continu adaptative. Ce préréglage est particulièrement efficace pour les vidéos de faible complexité et de complexité moyenne, où les fichiers de sortie sont à des vitesses de transmission inférieures, mais à une qualité qui offre toujours une bonne expérience aux utilisateurs. La sortie contiendra des fichiers MP4 avec vidéo et audio entrelacées. Pour plus d’informations, consultez les [spécifications d’API ouverte](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/mediaservices/resource-manager/Microsoft.Media/stable/2018-07-01/Encoding.json).
- Amélioration des performances et du multithreading pour le redimensionnement dans Standard Encoder. Dans des conditions spécifiques, le client doit voir une augmentation des performances du codage VOD de 5 à 40 %. Le contenu à faible complexité encodé en plusieurs vitesses de transmission dispose des meilleures améliorations de performances. 
- L’encodage standard gère maintenant une cadence GOP régulière pour les contenus à fréquence d’images variable (VFR) lors d’un encodage VOD en cas d’utilisation du paramètre GOP basé sur le temps.  Cela signifie qu’un client soumettant un contenu de fréquence d’images mixte variant entre 15 et 30 images par seconde, par exemple, doit maintenant voir les distances GOP normales calculées sur la sortie vers des fichiers MP4 de streaming à débit adaptatif. Cela permet d’améliorer la possibilité de basculer entre les pistes en cas de diffusion sur TLS ou DASH. 
-  Amélioration de la synchronisation AV pour un contenu source avec une fréquence d’images variable (VFR)

### <a name="video-indexer-video-analytics"></a>Video Indexer, analyse vidéo

- Les images clés extraites à l’aide de la présélection VideoAnalyzer sont désormais dans la résolution d’origine de la vidéo au lieu d’être redimensionnées. L’extraction d’images clés en haute résolution vous donne des images avec la qualité d’origine et vous permet d’utiliser les modèles d’intelligence artificielle basés sur une image fournis par les services Vision par ordinateur et Custom Vision de Microsoft pour obtenir encore plus d’informations à partir de votre vidéo.

## <a name="september-2019"></a>Septembre 2019

###  <a name="media-services-v3"></a>Media Services v3  

#### <a name="live-linear-encoding-of-live-events"></a>Encodage linéaire en direct des événements en direct

Media Services v3 annonce la préversion de 24 heures x 365 jours d’encodage linéaire en direct des événements en direct.

###  <a name="media-services-v2"></a>Media Services v2  

#### <a name="deprecation-of-media-processors"></a>Dépréciation des processeurs multimédias

Nous annonçons la dépréciation d’*Azure Media Indexer* et d’*Azure Media Indexer 2 Preview*. Pour connaître les dates de mise hors service, consultez l’article consacré aux [composants hérités](../previous/legacy-components.md). [Azure Media Services Video Indexer](../video-indexer/index.yml) remplace ces processeurs multimédias hérités.

Pour plus d’informations, consultez [Effectuer une migration depuis Azure Media Indexer et Azure Media Indexer 2 vers Azure Media Services Video Indexer](../previous/migrate-indexer-v1-v2.md).

## <a name="august-2019"></a>Août 2019

###  <a name="media-services-v3"></a>Media Services v3  

#### <a name="south-africa-regional-pair-is-open-for-media-services"></a>La paire régionale d’Afrique du Sud est ouverte pour Media Services 

Media Services est maintenant disponible dans les régions Afrique du Sud Nord et Afrique du Sud Ouest.

Pour plus d’informations, consultez [Clouds et régions dans lesquels Media Services v3 est présent](azure-clouds-regions.md).

###  <a name="media-services-v2"></a>Media Services v2  

#### <a name="deprecation-of-media-processors"></a>Dépréciation des processeurs multimédias

Nous annonçons la dépréciation des processeurs multimédias *Windows Azure Media Encoder* (WAME) et *Azure Media Encoder* (AME), qui sont en phase de mise hors service. Pour connaître les dates de mise hors service, consultez l’article consacré aux [composants hérités](../previous/legacy-components.md).

Pour plus d'informations, consultez [Migration de WAME vers Media Encoder Standard](../previous/migrate-windows-azure-media-encoder.md) et [Migration d'AME vers Media Encoder Standard](../previous/migrate-azure-media-encoder.md).
 
## <a name="july-2019"></a>Juillet 2019

### <a name="content-protection"></a>Protection du contenu

Lors du streaming de contenu protégé par une restriction de jeton, les utilisateurs finaux doivent obtenir un jeton envoyé dans le cadre de la requête de remise de clé. La fonctionnalité de *prévention de relecture de jeton* permet aux clients de Media Services de limiter le nombre de fois qu’un même jeton peut être utilisé pour demander une clé ou une licence. Pour plus d’informations, consultez [Prévention de la relecture de jeton](content-protection-overview.md#token-replay-prevention).

Depuis juillet, la fonctionnalité d’évaluation était uniquement disponible dans les régions USA Centre et USA Centre-Ouest.

## <a name="june-2019"></a>Juin 2019

### <a name="video-subclipping"></a>Sous-découpage de vidéos

Vous pouvez désormais découper ou sous-découper une vidéo lors de son encodage à l’aide d’un [travail](/rest/api/media/jobs). 

Cette fonctionnalité peut être utilisée avec n’importe quelle [transformation](/rest/api/media/transforms) qui est générée en utilisant les présélections [BuiltInStandardEncoderPreset](/rest/api/media/transforms/createorupdate#builtinstandardencoderpreset) ou [StandardEncoderPreset](/rest/api/media/transforms/createorupdate#standardencoderpreset). 

Consultez les exemples :

* [Sous-cliper une vidéo avec .NET](subclip-video-dotnet-howto.md)
* [Sous-cliper une vidéo avec REST](subclip-video-rest-howto.md)

## <a name="may-2019"></a>Mai 2019

### <a name="azure-monitor-support-for-media-services-diagnostic-logs-and-metrics"></a>Prise en charge Azure Monitor pour les journaux de diagnostic et métriques Media Services

Vous pouvez désormais utiliser Azure Monitor pour afficher les données de télémétrie émises par Media Services.

* Utilisez les journaux de diagnostic Azure Monitor pour surveiller les requêtes envoyées par le point de terminaison de remise des clés Media Services. 
* Surveillez les métriques émises par les [points de terminaison de streaming](streaming-endpoint-concept.md) Media Services.   

Pour plus d’informations, consultez [Superviser les métriques et les journaux de diagnostic Media Services](media-services-metrics-diagnostic-logs.md).

### <a name="multi-audio-tracks-support-in-dynamic-packaging"></a>Prise en charge de plusieurs pistes audio dans l’empaquetage dynamique 

L’[empaquetage dynamique](dynamic-packaging-overview.md) prend en charge plusieurs pistes audio pour la sortie HLS (version 4 ou ultérieure) pour le streaming des actifs multimédias qui ont plusieurs pistes audio avec plusieurs langues et codecs.

### <a name="korea-regional-pair-is-open-for-media-services"></a>La paire régionale de Corée est ouverte pour Media Services 

Media Services est maintenant disponible dans les régions Corée Centre et Corée Sud. 

Pour plus d’informations, consultez [Clouds et régions dans lesquels Media Services v3 est présent](azure-clouds-regions.md).

### <a name="performance-improvements"></a>Optimisation des performances

Ajout de mises à jour incluant l’amélioration des performances Media Services.

* La taille maximale du fichier pris en charge pour le traitement a été mise à jour. Consultez [Quotas et limites](limits-quotas-constraints.md).
* [Améliorations des vitesses d’encodage](concept-media-reserved-units.md).

## <a name="april-2019"></a>Avril 2019

### <a name="new-presets"></a>Nouvelles présélections

* [FaceDetectorPreset](/rest/api/media/transforms/createorupdate#facedetectorpreset) a été ajouté aux présélections d’analyseur intégrées.
* [ContentAwareEncodingExperimental](/rest/api/media/transforms/createorupdate#encodernamedpreset) a été ajouté aux présélections d’encodeur intégrées. Pour plus d’informations, consultez [Encodage sensible au contenu](content-aware-encoding.md). 

## <a name="march-2019"></a>Mars 2019

L’empaquetage dynamique prend désormais en charge Dolby Atmos. Pour plus d’informations, consultez [Codecs audio pris en charge par l’empaquetage dynamique](dynamic-packaging-overview.md#audio-codecs-supported-by-dynamic-packaging).

Vous pouvez désormais spécifier une liste de filtres de comptes ou d’actifs, qui s’applique à votre localisateur de streaming. Pour plus d’informations, consultez [Associer des filtres avec le localisateur de streaming](filters-concept.md#associating-filters-with-streaming-locator).

## <a name="february-2019"></a>Février 2019

Media Services v3 est désormais pris en charge dans les clouds nationaux Azure. Toutes les fonctionnalités ne sont pas encore disponibles dans tous les clouds. Pour en savoir plus, consultez [Clouds et régions dans lesquels Azure Media Services v3 existe](azure-clouds-regions.md).

L'événement [Microsoft.Media.JobOutputProgress](media-services-event-schemas.md#monitoring-job-output-progress) a été ajouté aux schémas Azure Event Grid de Media Services.

## <a name="january-2019"></a>Janvier 2019

### <a name="media-encoder-standard-and-mpi-files"></a>Fichiers Media Encoder Standard et MPI 

En cas d'encodage avec Media Encoder Standard pour produire des fichiers MP4, un nouveau fichier .mpi est généré et ajouté à la ressource de sortie. Ce fichier MPI est destiné à améliorer les performances pour les scénarios d’[empaquetage dynamique](dynamic-packaging-overview.md) et de diffusion en continu.

Vous ne devez ni modifier ni supprimer le fichier MPI, ni dépendre de l'existence (ou non) d'un tel fichier dans votre service.

## <a name="december-2018"></a>Décembre 2018

Les mises à jour de la version mise à la disposition générale de l'API V3 incluent :
       
* Les propriétés **PresentationTimeRange** ne sont plus « obligatoires » pour **AssetFilters** et **AccountFilters**. 
* Les options de requête $top et $skip de **Jobs** et **Transforms** ont été supprimées et $orderby a été ajouté. Lors de l'ajout de la nouvelle fonctionnalité de commande, il a été découvert que les options $top et $skip avaient déjà été accidentellement exposées, même si elles n'avaient pas été implémentées.
* L'extensibilité de l'énumération a été réactivée. Cette fonctionnalité a été activée dans les préversions du kit de développement logiciel (SDK), puis accidentellement désactivée dans la version mise à la disposition générale.
* Deux stratégies de diffusion en continu prédéfinies ont été renommées. **SecureStreaming** s'appelle désormais **MultiDrmCencStreaming**. **SecureStreamingWithFairPlay** s'appelle désormais **Predefined_MultiDrmStreaming**.

## <a name="november-2018"></a>Novembre 2018

Le module CLI 2.0 est désormais disponible pour [Azure Media Services v3 GA](/cli/azure/ams?view=azure-cli-latest&preserve-view=true) – v 2.0.50.

### <a name="new-commands"></a>Nouvelles commandes

- [az ams account](/cli/azure/ams/account?view=azure-cli-latest&preserve-view=true)
- [az ams account-filter](/cli/azure/ams/account-filter?view=azure-cli-latest&preserve-view=true)
- [az ams asset](/cli/azure/ams/asset?view=azure-cli-latest&preserve-view=true)
- [az ams asset-filter](/cli/azure/ams/asset-filter?view=azure-cli-latest&preserve-view=true)
- [az ams content-key-policy](/cli/azure/ams/content-key-policy?view=azure-cli-latest&preserve-view=true)
- [az ams job](/cli/azure/ams/job?view=azure-cli-latest&preserve-view=true)
- [az ams live-event](/cli/azure/ams/live-event?view=azure-cli-latest&preserve-view=true)
- [az ams live-output](/cli/azure/ams/live-output?view=azure-cli-latest&preserve-view=true)
- [az ams streaming-endpoint](/cli/azure/ams/streaming-endpoint?view=azure-cli-latest&preserve-view=true)
- [az ams streaming-locator](/cli/azure/ams/streaming-locator?view=azure-cli-latest&preserve-view=true)
- [az ams account mru](/cli/azure/ams/account/mru?view=azure-cli-latest&preserve-view=true) : permet de gérer les unités réservées Multimédia. Pour plus d’informations, consultez [Mise à l’échelle des unités réservées Multimédia](media-reserved-units-cli-how-to.md).

### <a name="new-features-and-breaking-changes"></a>Nouvelles fonctionnalités et dernières modifications

#### <a name="asset-commands"></a>Commandes des éléments multimédia

- Les arguments ```--storage-account``` et ```--container``` ont été ajoutés.
- Les valeurs par défaut pour le délai d’expiration (présent + 23h) et les autorisations (lecture) dans la commande ```az ams asset get-sas-url``` ont été ajoutées.

#### <a name="job-commands"></a>Commandes des tâches

- Les arguments ```--correlation-data``` et ```--label``` ont été ajoutés.
- ```--output-asset-names``` a été renommé en ```--output-assets```. Il accepte désormais une liste de ressources séparée par des espaces au format « assetName=label ». Une ressource sans étiquette peut être envoyée comme ceci : « assetName= ».

#### <a name="streaming-locator-commands"></a>Commandes du localisateur de diffusion en continu

- La commande de base ```az ams streaming locator``` a été remplacée par ```az ams streaming-locator```.
- Les arguments ```--streaming-locator-id``` et ```--alternative-media-id support``` ont été ajoutés.
- L'argument ```--content-keys argument``` a été mis à jour.
- ```--content-policy-name``` a été renommé en ```--content-key-policy-name```.

#### <a name="streaming-policy-commands"></a>Commandes des stratégies de diffusion en continu

- La commande de base ```az ams streaming policy``` a été remplacée par ```az ams streaming-policy```.
- La prise en charge des paramètres de chiffrement dans ```az ams streaming-policy create``` a été ajoutée.

#### <a name="transform-commands"></a>Commandes de transformation

- L'argument ```--preset-names``` a été remplacé par ```--preset```. À présent vous ne pouvez définir qu’une sortie/présélection à la fois (pour en ajouter d’autres, vous devez exécuter ```az ams transform output add```). En outre, vous pouvez définir StandardEncoderPreset de façon personnalisée en transmettant le chemin à votre JSON personnalisé.
- ```az ams transform output remove``` désormais possible en transmettant le chemin de l’index pour le supprimer.
- Les arguments ```--relative-priority, --on-error, --audio-language and --insights-to-extract``` ont été ajoutés dans les commandes ```az ams transform create``` et ```az ams transform output add```.

## <a name="october-2018---ga"></a>Octobre 2018 - Mise à la disposition générale

Cette section décrit les mises à jour d’octobre d’Azure Media Services (AMS).

### <a name="rest-v3-ga-release"></a>Version en disponibilité générale de REST v3

La [version en disponibilité générale de REST v3](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/mediaservices/resource-manager/Microsoft.Media/stable/2018-07-01) contient davantage d’API pour les filtres de manifeste de niveau En direct, Compte/Élément multimédia, ainsi qu’une prise en charge de DRM.

#### <a name="azure-resource-management"></a>Gestion des ressources Azure 

La prise en charge de la gestion des ressources Azure permet l’unification des API de gestion et des opérations (désormais tout se trouve à un même emplacement).

À partir de cette version, vous pouvez utiliser des modèles Resource Manager pour créer des événements en direct.

#### <a name="improvement-of-asset-operations"></a>Amélioration des opérations sur les éléments multimédias 

Voici les améliorations qui ont été apportées :

- Ingestion à partir d’URL HTTP(s) ou d’URL SAP de Stockage Blob Azure.
- Possibilité de spécifier des noms de conteneurs personnalisés pour les éléments multimédias. 
- Meilleure prise en charge de la sortie pour créer des flux de travail personnalisés avec Azure Functions.

#### <a name="new-transform-object"></a>Nouvel objet Transform

Le nouvel objet **Transform** simplifie le modèle d’encodage. Le nouvel objet facilite la création et le partage de modèles et de préréglages Resource Manager d’encodage. 

#### <a name="azure-active-directory-authentication-and-azure-rbac"></a>Authentification Azure Active Directory et Azure RBAC

Azure AD Authentication et le contrôle d’accès en fonction du rôle (Azure RBAC) permettent les transformations sécurisées, les événements en direct, les stratégies de clé de contenu ou les éléments multimédias par rôle ou utilisateurs dans Azure AD.

#### <a name="client-sdks"></a>Kits de développement logiciel (SDK) client  

Langages pris en charge dans Media Services v3 : .NET Core, Java, Node.js, Ruby, Typescript, Python, Go.

#### <a name="live-encoding-updates"></a>Mises à jour de l’encodage en direct

Voici les mises à jour de l’encodage en direct qui ont été apportées :

- Nouveau mode de faible latence pour le direct (10 secondes de bout en bout).
- Prise en charge améliorée de RTMP (stabilité accrue et meilleure prise en charge de l’encodeur source).
- Ingestion sécurisée RTMPS.

    Quand vous créez un événement en direct, vous obtenez maintenant 4 URL d’ingestion. Les 4 URL d’ingestion sont presque identiques, ont le même jeton de streaming (AppId) ; seule la partie du numéro de port est différente. Il existe deux URL principales et de secours pour RTMPS. 
- Prise en charge d’un transcodage de 24 heures. 
- Prise en charge améliorée de la signalisation des annonces dans RTMP via SCTE35.

#### <a name="improved-event-grid-support"></a>Prise en charge améliorée d’Event Grid

Voici les améliorations de la prise en charge d’Event Grid que vous pouvez constater :

- Intégration d’Azure EventGrid pour un développement facilité avec Logic Apps et Azure Functions. 
- Abonnement aux événements Encodage, Canaux live, etc.

### <a name="cmaf-support"></a>Prise en charge de CMAF

Prise en charge du chiffrement CMAF et « cbcs » pour les lecteurs Apple HLS (iOS 11+) et MPEG-DASH qui prennent en charge CMAF.

### <a name="video-indexer"></a>Video Indexer

La version en disponibilité générale de l’indexeur vidéo a été annoncée en août. Pour être au fait des dernières informations concernant les fonctionnalités actuellement prises en charge, consultez [Qu’est-ce que Video Indexer](../video-indexer/video-indexer-overview.md?bc=/azure/media-services/video-indexer/breadcrumb/toc.json&toc=/azure/media-services/video-indexer/toc.json). 

### <a name="plans-for-changes"></a>Modifications planifiées

#### <a name="azure-cli-20"></a>Azure CLI 2.0
 
Le module Azure CLI 2.0 qui inclut des opérations sur toutes les fonctionnalités (dont En direct, Stratégies de clé de contenu, Filtres de comptes/éléments multimédias, Stratégies de streaming) sera bientôt disponible. 

### <a name="known-issues"></a>Problèmes connus

Seuls les clients qui utilisaient la préversion de l’API pour les filtres d’éléments multimédias ou de comptes sont concernés par le problème suivant.

Si vous avez créé des filtres d’éléments multimédias ou de comptes entre le 28/09 et le 12/10 avec l’interface CLI ou les API Media Services v3, vous devez supprimer tous les filtres d’éléments multimédias et de comptes et les recréer en raison d’un conflit de version. 

## <a name="may-2018---preview"></a>Mai 2018 - Préversion

### <a name="net-sdk"></a>Kit de développement logiciel (SDK) .NET

Les fonctionnalités suivantes sont présentes dans le Kit de développement logiciel (SDK) .NET :

* **Transformations** et **Travaux** pour encoder ou analyser le contenu du média. Pour obtenir des exemples, consultez [Diffuser des fichiers](stream-files-tutorial-with-api.md) et [Analyser](analyze-videos-tutorial-with-api.md).
* **StreamingLocators** pour la publication et la diffusion en continu de contenu sur les appareils de l’utilisateur final
* **StreamingPolicies** et **ContentKeyPolicies** pour configurer la distribution de clés et la protection du contenu (DRM) lors de la remise de contenu.
* **LiveEvents** et **LiveOutputs** pour configurer l’ingestion et l’archivage de contenu en continu en direct.
* **Éléments multimédias** pour stocker et publier du contenu multimédia dans le stockage Azure. 
* **StreamingEndpoints** pour configurer et mettre à l’échelle l’empaquetage dynamique, le chiffrement et la diffusion en continu du contenu multimédia en direct et à la demande.

### <a name="known-issues"></a>Problèmes connus

* Quand vous soumettez un travail, vous pouvez spécifier l’ingestion de votre vidéo source à l’aide d’URL HTTPS, d’URL SAP ou de chemins de fichiers situés dans le stockage Blob Azure. Actuellement, Media Services v3 ne prend pas en charge l’encodage de transfert mémorisé en bloc sur les URL HTTPS.

## <a name="ask-questions-give-feedback-get-updates"></a>Poser des questions, envoyer des commentaires, obtenir des mises à jour

Découvrez l’article [Communauté Azure Media Services](media-services-community.md) pour découvrir les différentes façons dont vous pouvez poser des questions, faire des commentaires et obtenir des mises à jour sur Media Services.

## <a name="see-also"></a>Voir aussi

[Conseils de migration pour le passage de Media Services v2 à Media Services v3](migrate-from-v2-to-v3.md#known-issues)

## <a name="next-steps"></a>Étapes suivantes

- [Vue d'ensemble](media-services-overview.md)
- [Mises à jour de la documentation Media Services v3](docs-release-notes.md)
- [Notes de publication Media Services v2](../previous/media-services-release-notes.md)