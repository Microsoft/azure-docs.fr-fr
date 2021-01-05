---
title: Filtres et manifestes dynamiques | Microsoft Docs
description: Cette rubrique décrit comment créer des filtres pour que votre client puisse les utiliser pour diffuser des sections spécifiques d'un flux. Media Services crée des manifestes dynamiques pour l’archivage de cette diffusion sélective.
services: media-services
documentationcenter: ''
author: cenkdin
manager: femila
editor: ''
ms.assetid: ff102765-8cee-4c08-a6da-b603db9e2054
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 2d83b114487f882b7ee38d3d71c84b6abec04a2b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89266916"
---
# <a name="filters-and-dynamic-manifests"></a>Filtres et manifestes dynamiques

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector" title1="Sélectionnez la version de Media Services que vous utilisez :"]
> * [Version 2](media-services-dynamic-manifest-overview.md)
> * [Version 3](../latest/filters-dynamic-manifest-overview.md)

À partir de la version 2.17, Media Services vous permet de définir des filtres pour vos éléments multimédias. Ces filtres sont des règles côté serveur qui permettent à vos clients de choisir d’effectuer des opérations comme les suivantes : lecture d’une section d’une vidéo uniquement (au lieu de la vidéo entière), spécification d’un seul sous-ensemble de rendus audio et vidéo pouvant être gérés par l’appareil de votre client (au lieu de tous les rendus associés à l’élément multimédia). Ce filtrage de vos éléments multimédias est obtenu via des **manifestes dynamiques**créés à la demande de votre client pour diffuser une vidéo selon des filtres spécifiés.

Cette rubrique décrit les scénarios courants dans lesquels l’utilisation de filtres s’avère utile pour vos clients, elle fait également le lien avec d’autres rubriques qui montrent comment créer par programmation de tels filtres.

## <a name="overview"></a>Vue d’ensemble
Quand vous distribuez votre contenu aux clients (diffusion en continu d’événements en direct ou vidéo à la demande), votre objectif est de distribuer une vidéo de haute qualité à divers appareils dans des conditions de réseau différentes. Pour atteindre cet objectif, procédez comme suit :

* encodez votre flux dans un flux vidéo à débit binaire multiple ([débit binaire adaptatif](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming)) (les conditions de qualité et de réseau sont ainsi prises en charge) ; 
* utilisez l' [empaquetage dynamique](media-services-dynamic-packaging-overview.md) pour empaqueter de nouveau votre flux dans différents protocoles dynamiquement (la diffusion en continu sur différents appareils est ainsi prise en charge). Media Services prend en charge la distribution des technologies de streaming à débit adaptatif suivantes : HTTP Live Streaming (HLS), Smooth Streaming et MPEG DASH. 

### <a name="manifest-files"></a>Fichiers manifeste
Quand vous encodez un élément multimédia pour une diffusion à débit binaire adaptatif, un fichier **manifeste** (ou sélection) est créé (il s'agit d'un fichier texte ou XML). Le fichier **manifeste** inclut des métadonnées de diffusion en continu telles que les suivantes : type de piste (audio, vidéo ou texte), nom de piste, heure de début et de fin, débit binaire (qualités), langues de piste, fenêtre de présentation (fenêtre glissante de durée fixe), codec vidéo (FourCC). Il indique également au lecteur de récupérer le fragment suivant en fournissant des informations sur les fragments vidéo pouvant être lus suivants disponibles et leur emplacement. Les fragments (ou segments) correspondent aux « blocs » réels d'un contenu vidéo.

Voici un exemple de fichier manifeste : 

```xml
<?xml version="1.0" encoding="UTF-8"?>    
<SmoothStreamingMedia MajorVersion="2" MinorVersion="0" Duration="330187755" TimeScale="10000000">

<StreamIndex Chunks="17" Type="video" Url="QualityLevels({bitrate})/Fragments(video={start time})" QualityLevels="8">
<QualityLevel Index="0" Bitrate="5860941" FourCC="H264" MaxWidth="1920" MaxHeight="1080" CodecPrivateData="0000000167640028AC2CA501E0089F97015202020280000003008000001931300016E360000E4E1FF8C7076850A4580000000168E9093525" />
<QualityLevel Index="1" Bitrate="4602724" FourCC="H264" MaxWidth="1920" MaxHeight="1080" CodecPrivateData="0000000167640028AC2CA501E0089F97015202020280000003008000001931100011EDC00002CD29FF8C7076850A45800000000168E9093525" />
<QualityLevel Index="2" Bitrate="3319311" FourCC="H264" MaxWidth="1280" MaxHeight="720" CodecPrivateData="000000016764001FAC2CA5014016EC054808080A00000300020000030064C0800067C28000103667F8C7076850A4580000000168E9093525" />
<QualityLevel Index="3" Bitrate="2195119" FourCC="H264" MaxWidth="960" MaxHeight="540" CodecPrivateData="000000016764001FAC2CA503C045FBC054808080A000000300200000064C1000044AA0000ABA9FE31C1DA14291600000000168E9093525" />
<QualityLevel Index="4" Bitrate="1469881" FourCC="H264" MaxWidth="960" MaxHeight="540" CodecPrivateData="000000016764001FAC2CA503C045FBC054808080A000000300200000064C04000B71A0000E4E1FF8C7076850A4580000000168E9093525" />
<QualityLevel Index="5" Bitrate="978815" FourCC="H264" MaxWidth="640" MaxHeight="360" CodecPrivateData="000000016764001EAC2CA50280BFE5C0548303032000000300200000064C08001E8480004C4B7F8C7076850A45800000000168E9093525" />
<QualityLevel Index="6" Bitrate="638374" FourCC="H264" MaxWidth="640" MaxHeight="360" CodecPrivateData="000000016764001EAC2CA50280BFE5C0548303032000000300200000064C080013D60000C65DFE31C1DA1429160000000168E9093525" />
<QualityLevel Index="7" Bitrate="388851" FourCC="H264" MaxWidth="320" MaxHeight="180" CodecPrivateData="000000016764000DAC2CA505067E7C054830303200000300020000030064C040030D40003D093F8C7076850A45800000000168E9093525" />

<c t="0" d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="9600000"/>
</StreamIndex>


<StreamIndex Chunks="17" Type="audio" Url="QualityLevels({bitrate})/Fragments(AAC_und_ch2_128kbps={start time})" QualityLevels="1" Name="AAC_und_ch2_128kbps">
<QualityLevel AudioTag="255" Index="0" BitsPerSample="16" Bitrate="125658" FourCC="AACL" CodecPrivateData="1210" Channels="2" PacketSize="4" SamplingRate="44100" />

<c t="0" d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="6965987" /></StreamIndex>


<StreamIndex Chunks="17" Type="audio" Url="QualityLevels({bitrate})/Fragments(AAC_und_ch2_56kbps={start time})" QualityLevels="1" Name="AAC_und_ch2_56kbps">
<QualityLevel AudioTag="255" Index="0" BitsPerSample="16" Bitrate="53655" FourCC="AACL" CodecPrivateData="1210" Channels="2" PacketSize="4" SamplingRate="44100" />

<c t="0" d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="6965987" /></StreamIndex>

</SmoothStreamingMedia>
```

### <a name="dynamic-manifests"></a>Manifestes dynamiques
Il existe des [scénarios](media-services-dynamic-manifest-overview.md#scenarios) dans lesquels votre client a besoin de davantage de flexibilité que celle décrite dans le fichier manifeste de l'élément multimédia par défaut. Par exemple :

* Particularité de l’appareil : distribuez uniquement les rendus spécifiés et/ou les pistes de langue spécifiées qui sont prises en charge par l’appareil utilisé pour lire le contenu (« filtrage de rendu »). 
* Réduisez le manifeste pour afficher un sous-clip d'un événement en direct (« filtrage de sous-clip »).
* Découpez le début d'une vidéo (« découpage d'une vidéo »).
* Ajustez la fenêtre de présentation (DVR) afin de fournir une longueur limitée de la fenêtre du DVR dans le lecteur (« ajustement de la fenêtre de présentation »).

Pour obtenir cette flexibilité, Media Services propose des **manifestes dynamiques** basés sur des [filtres](media-services-dynamic-manifest-overview.md#filters)prédéfinis.  Une fois que vous avez défini les filtres, vos clients peuvent les utiliser pour diffuser un rendu spécifique ou des sous-clips de votre vidéo. Ils peuvent spécifier des filtres dans l'URL de diffusion en continu. Les filtres peuvent s’appliquer aux protocoles de streaming à débit adaptatif par [empaquetage dynamique](media-services-dynamic-packaging-overview.md) : HLS, MPEG-DASH et Smooth Streaming. Par exemple :

URL MPEG DASH avec filtre

`http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=mpd-time-csf,filter=MyLocalFilter)`

URL Smooth Streaming avec filtre

`http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(filter=MyLocalFilter)`


Pour plus d'informations sur la façon de distribuer votre contenu et générer des URL de diffusion en continu, consultez [Vue d'ensemble de la distribution de contenu](media-services-deliver-content-overview.md).

> [!NOTE]
> Notez que les manifestes dynamiques ne changent pas l'élément multimédia ni son manifeste par défaut. Votre client peut choisir de demander un flux avec ou sans filtres. 
> 
> 

### <a name="filters"></a><a id="filters"></a>Filtres
Il existe deux types de filtres d'éléments multimédias : 

* les filtres globaux (applicables à n'importe quel élément multimédia dans le compte Azure Media Services et dont la durée de vie correspond à celle du compte) ; 
* les filtres locaux (applicables uniquement à un élément multimédia avec lequel le filtre a été associé au moment de la création et dont la durée de vie correspond à celle de l'élément multimédia). 

Les filtres globaux et locaux ont exactement les mêmes propriétés. La principale différence entre les deux a trait aux scénarios auxquels ils sont les plus appropriés. Les filtres globaux conviennent généralement aux profils d'appareil (filtrage de rendu) alors que les filtres locaux peuvent être utilisés pour découper un élément multimédia spécifique.

## <a name="common-scenarios"></a><a id="scenarios"></a>Scénarios courants
Comme mentionné précédemment, quand vous distribuez votre contenu aux clients (streaming d’événements en direct ou de vidéo à la demande), votre objectif est de distribuer une vidéo de qualité supérieure à divers appareils, dans des conditions de réseau différentes. De plus, vous pouvez avoir d’autres impératifs impliquant le filtrage de vos éléments multimédias et l’utilisation de **manifestes dynamiques**. Les sections suivantes présentent brièvement différents scénarios de filtrage.

* Spécification d'un seul sous-ensemble de rendus audio et vidéos gérables par certains appareils (au lieu de tous les rendus associés à l'élément multimédia). 
* Lecture d'une seule section d'une vidéo (au lieu de la vidéo entière).
* Ajustement de la fenêtre de présentation DVR.

## <a name="rendition-filtering"></a>Filtrage de rendu
Vous pouvez choisir d'encoder votre élément multimédia pour plusieurs profils d'encodage (H.264 Baseline, H.264 High, AACL, AACH, Dolby Digital Plus) et plusieurs qualités de débit binaire. Toutefois, tous les appareils client ne prennent pas en charge tous les profils et débits binaires de tous vos éléments multimédias. Par exemple, les anciens appareils Android prennent uniquement en charge H.264 Baseline+AACL. L'envoi de débits binaires plus élevés à un appareil qui ne peut pas en tirer partie ne fait que gaspiller la bande passante et entraîner des calculs inutiles pour l'appareil. Ce dernier doit en effet décoder toutes les informations données, pour finalement les réduire pour pouvoir les afficher.

Avec un manifeste dynamique, vous pouvez créer des profils d'appareil mobile, de console, HD/SD, etc. et inclure les pistes et les qualités que vous voulez intégrer dans chaque profil.

![Exemple de filtrage de rendu][renditions2]

Dans l’exemple suivant, un encodeur a été utilisé pour encoder un élément multimédia mezzanine en sept rendus vidéo ISO MP4s (de 180 p à 1 080 p). L’élément multimédia encodé peut être empaqueté de manière dynamique dans l’un des protocoles de diffusion en continu suivants : HLS, Smooth et MPEG DASH.  En haut du diagramme, le manifeste HLS de l'élément multimédia sans aucun filtre apparaît (il contient les sept rendus).  Dans la partie inférieure gauche apparaît le manifeste HLS auquel un filtre nommé « ott » a été appliqué. Le filtre « ott » indique de supprimer tous les débits binaires inférieurs à 1 Mbit/s, ce qui entraîne la suppression des deux niveaux de qualité les plus bas dans la réponse. Dans la partie inférieure droite apparaît le manifeste HLS auquel un filtre nommé « mobile » a été appliqué. Le filtre « mobile » indique de supprimer les rendus pour lesquels la résolution est supérieure à 720 p, ce qui entraîne la suppression des deux rendus à 1 080 p.

![Filtrage de rendu][renditions1]

## <a name="removing-language-tracks"></a>Suppression des pistes de langue
Vos éléments multimédias peuvent inclure plusieurs langues audio telles que l'anglais, l'espagnol, le français, etc. Généralement, le Kit de développement logiciel (SDK) du lecteur gère la sélection de pistes audio par défaut et les pistes audio disponibles par sélection de l'utilisateur. Il est difficile de développer ces Kits de développement logiciel (SDK) de lecteur, car différentes implémentations sont requises entre les infrastructures de lecteur propres aux appareils. De plus, sur certaines plateformes, les API de lecteur sont limitées et n'incluent pas de fonctionnalité de sélection audio, ce qui ne permet pas aux utilisateurs de sélectionner ou de modifier la piste audio par défaut. Avec les filtres d'éléments multimédias, vous pouvez contrôler le comportement en créant des filtres qui incluent uniquement les langues audio souhaitées.

![Filtrage des pistes de langue][language_filter]

## <a name="trimming-start-of-an-asset"></a>Découpage du début d'un élément multimédia
Pour la plupart des événements de diffusion en continu en direct, les opérateurs exécutent des tests avant l'événement réel. Par exemple, ils peuvent inclure une ardoise comme celle-ci avant le début de l’événement : « Le programme va commencer dans un instant ». Si le programme est en cours d’archivage, le test et les données d’ardoise sont également archivés et donc inclus dans la présentation. Toutefois, ces informations ne doivent pas être montrées aux clients. Avec un manifeste dynamique, vous pouvez créer un filtre sur l'heure de début et supprimer les données indésirables du manifeste.

![Découpage du début][trim_filter]

## <a name="creating-subclips-views-from-a-live-archive"></a>Création de sous-clips (vues) à partir d’une archive en direct
De nombreux événements en direct ont une durée d'exécution longue et une archive en direct peut inclure plusieurs événements. Une fois l’événement en direct terminé, les diffuseurs peuvent décomposer cette archive en direct selon des séquences de démarrage et d’arrêt du programme logiques. Ensuite, il suffit de publier ces programmes virtuels séparément, sans post-traitement de l’archive en direct, et sans créer d’éléments multimédias distincts (qui ne bénéficient pas des fragments mis en cache existants dans les CDN). Par exemple, ces programmes virtuels peuvent correspondre aux mi-temps d’un match de football ou de basket, aux tours de batte au base-ball ou à des événements sportifs ponctuels.

Avec un manifeste dynamique, vous pouvez créer des filtres en utilisant des heures de début/fin et créer des vues virtuelles par-dessus votre archive en direct. 

![Filtre de sous-clip][subclip_filter]

Élément multimédia filtré :

![Ski][skiing]

## <a name="adjusting-presentation-window-dvr"></a>Ajustement de la fenêtre de présentation (DVR)
Actuellement, Azure Media Services propose une archive circulaire où la durée peut être configurée entre 5 minutes et 25 heures. Le filtrage de manifeste peut servir à créer une fenêtre DVR de substitution par-dessus l'archive, sans supprimer le média. Il existe de nombreux scénarios où les diffuseurs veulent fournir une fenêtre DVR limitée à déplacer avec la session en direct et qui garde en même temps une plus grande fenêtre d’archivage. Un diffuseur peut utiliser les données en dehors de la fenêtre DVR pour mettre en avant des clips ou il peut proposer des fenêtres DVR différentes pour différents appareils. Par exemple, la plupart des appareils mobiles ne gèrent pas les grandes fenêtres DVR (une fenêtre DVR peut durer deux minutes pour les appareils mobiles alors qu’elle dure une heure pour les ordinateurs de bureau).

![Fenêtre DVR][dvr_filter]

## <a name="adjusting-livebackoff-live-position"></a>Ajustement de LiveBackoff (position en direct)
Le filtrage de manifeste peut servir à supprimer plusieurs secondes à partir du bord en direct d'un programme en direct. Il permet aux diffuseurs de regarder la présentation sur le point de publication d’aperçu et de créer des points d’insertion d’annonce avant que les utilisateurs ne reçoivent le flux (reculé de 30 secondes). Les diffuseurs peuvent ensuite transmettre ces annonces à leurs infrastructures client à temps pour qu'ils reçoivent et traitent les informations avant l'opportunité d'annonce.

Outre la prise en charge d’annonces, le paramètre LiveBackoff peut servir à ajuster la position des utilisateurs, afin que lorsque les clients dérivent et atteignent la session en direct, ils puissent toujours obtenir des fragments du serveur au lieu d’obtenir une erreur HTTP 404 ou 412.

![livebackoff_filter][livebackoff_filter]

## <a name="combining-multiple-rules-in-a-single-filter"></a>Combinaison de plusieurs règles dans un seul filtre
Vous pouvez combiner plusieurs règles de filtrage dans un filtre unique. Par exemple, vous pouvez définir une « règle de plage » pour supprimer les ardoises d’une archive en direct tout en éliminant les débits binaires disponibles. Lors de l’application de plusieurs règles de filtrage, le résultat final est l’intersection de toutes les règles.

![plusieurs règles][multiple-rules]

## <a name="create-filters-programmatically"></a>Création de filtres par programme
L’article suivant aborde les entités Media Services liées aux filtres. L’article indique également comment créer par programmation des filtres.  

[Créez des filtres avec les API REST](media-services-rest-dynamic-manifest.md).

## <a name="combining-multiple-filters-filter-composition"></a>Combinaison de plusieurs filtres (composition de filtre)
Vous pouvez également combiner plusieurs filtres dans une URL unique. 

Le scénario suivant explique les avantages de la combinaison de filtres :

1. Vous devez filtrer vos qualités vidéos pour des appareils mobiles tels qu’Android ou iPAD (afin de limiter les qualités vidéos). Pour supprimer les qualités indésirables, vous pouvez créer un filtre global adapté aux profils de vos appareils. Comme indiqué plus haut dans cet article, les filtres globaux peuvent être utilisés pour tous vos éléments multimédias d’un même compte Media Services sans aucune autre association. 
2. Vous pouvez aussi avoir besoin d’ajuster l’heure de début et de fin d’un élément multimédia. Pour ce faire, il vous suffit de créer un filtre local et de définir l’heure de début et de fin. 
3. Vous pouvez combiner ces deux filtres (sans combinaison, vous devez ajouter le filtrage de la qualité au filtre de réglage, ce qui complique davantage l’utilisation de filtres).

Pour combiner des filtres, vous devez définir les noms de filtre dans l’URL du manifeste ou de la liste de lecture en les délimitant avec des points-virgules. Supposons que vous ayez un filtre nommé *MyMobileDevice* qui filtre les qualités et un autre nommé *MyStartTime* qui définit une heure de début spécifique. Vous pouvez les combiner comme suit :

`http://teststreaming.streaming.mediaservices.windows.net/3d56a4d-b71d-489b-854f-1d67c0596966/64ff1f89-b430-43f8-87dd-56c87b7bd9e2.ism/Manifest(filter=MyMobileDevice;MyStartTime)`

Vous pouvez combiner jusqu’à trois filtres. 

Pour plus d’informations, consultez [ce](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/) blog.

## <a name="know-issues-and-limitations"></a>Problèmes connus et limitations
* Le manifeste dynamique fonctionne dans les limites d'un groupe d'images (GOP) (images clés), par conséquent, le découpage est précis au niveau du GOP. 
* Vous pouvez utiliser le même nom de filtre pour les filtres locaux et globaux. Les filtres locaux ont une priorité plus élevée et remplacent les filtres globaux.
* Si vous mettez à jour un filtre, il peut falloir jusqu'à 2 minutes pour que le point de terminaison de diffusion en continu actualise les règles. Si le contenu a été servi à l'aide de filtres (puis mis en cache dans des proxys et des caches CDN), la mise à jour de ces filtres peut entraîner des défaillances du lecteur. Il est recommandé d’effacer le cache après la mise à jour du filtre. Si cette option n'est pas possible, envisagez d'utiliser un filtre différent.

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>Voir aussi
[Vue d’ensemble de la distribution de contenu aux clients](media-services-deliver-content-overview.md)

[renditions1]: ./media/media-services-dynamic-manifest-overview/media-services-rendition-filter.png
[renditions2]: ./media/media-services-dynamic-manifest-overview/media-services-rendition-filter2.png

[rendered_subclip]: ./media/media-services-dynamic-manifests/media-services-rendered-subclip.png
[timeline_trim_event]: ./media/media-services-dynamic-manifests/media-services-timeline-trim-event.png
[timeline_trim_subclip]: ./media/media-services-dynamic-manifests/media-services-timeline-trim-subclip.png

[multiple-rules]:./media/media-services-dynamic-manifest-overview/media-services-multiple-rules-filters.png

[subclip_filter]: ./media/media-services-dynamic-manifest-overview/media-services-subclips-filter.png
[trim_event]: ./media/media-services-dynamic-manifests/media-services-timeline-trim-event.png
[trim_subclip]: ./media/media-services-dynamic-manifests/media-services-timeline-trim-subclip.png
[trim_filter]: ./media/media-services-dynamic-manifest-overview/media-services-trim-filter.png
[redered_subclip]: ./media/media-services-dynamic-manifests/media-services-rendered-subclip.png
[livebackoff_filter]: ./media/media-services-dynamic-manifest-overview/media-services-livebackoff-filter.png
[language_filter]: ./media/media-services-dynamic-manifest-overview/media-services-language-filter.png
[dvr_filter]: ./media/media-services-dynamic-manifest-overview/media-services-dvr-filter.png
[skiing]: ./media/media-services-dynamic-manifest-overview/media-services-skiing.png
