---
title: Filtrer vos manifestes à l’aide de Dynamic Packager
titleSuffix: Azure Media Services
description: Découvrez comment créer des filtres à l’aide de Dynamic Packager pour filtrer et diffuser de manière sélective vos manifestes.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: acb30c1659c4c29e0af83da5594bdd9a7e3465d8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89299029"
---
# <a name="filter-your-manifests-using-dynamic-packager"></a>Filtrer vos manifestes à l’aide de Dynamic Packager

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Lorsque vous fournissez du contenu en streaming à débit adaptatif à des appareils, il est souvent nécessaire de publier plusieurs versions d'un manifeste pour cibler des fonctionnalités d'appareil spécifiques ou la bande passante réseau disponible. Le [Packager dynamique](dynamic-packaging-overview.md) vous permet de spécifier des filtres qui peuvent filtrer des codecs, des résolutions, des vitesses de transmission et des combinaisons de pistes audio spécifiques à la volée. Ce filtrage supprime la nécessité de créer plusieurs copies. Vous devez simplement publier une nouvelle URL avec un ensemble spécifique de filtres configurés pour vos appareils cibles (iOS, Android, SmartTV ou navigateurs) et les fonctionnalités réseau (scénarios de bande passante élevée, mobiles ou de bande passante faible). Dans ce cas, les clients peuvent manipuler le streaming de votre contenu par le biais de la chaîne de requête (en spécifiant les [filtres de ressources ou les filtres de comptes](filters-concept.md) disponibles) et utiliser des filtres pour diffuser des sections spécifiques d’un flux.

Certains scénarios de distribution exigent que vous vérifiiez qu’un client est dans l’impossibilité d’accéder à des pistes spécifiques. Par exemple, vous pouvez ne pas souhaiter publier un manifeste qui contient des pistes HD vers un niveau d’abonné spécifique. Ou vous pouvez souhaiter supprimer des pistes à débit adaptatif (ABR) spécifiques pour réduire les coûts de distribution à un appareil spécifique qui ne bénéficie pas des pistes supplémentaires. Dans ce cas, vous pouvez associer une liste de filtres précréés à votre [localisateur de streaming](streaming-locators-concept.md) lors de la création. Si c’est ce cas, les clients ne peuvent pas manipuler la façon dont le contenu est diffusé, car elle est définie par le **localisateur de streaming**.

Vous pouvez combiner le filtrage en spécifiant des [filtres sur le localisateur de streaming](filters-concept.md#associating-filters-with-streaming-locator) et des filtres supplémentaires spécifiques à l’appareil que votre client spécifie dans l’URL. Cette combinaison peut être utile pour limiter les pistes supplémentaires comme les flux de métadonnées ou d'événements, les langues audio ou les pistes audio descriptives.

Cette possibilité de spécifier des filtres différents sur votre flux offre une puissante solution de manipulation de **manifeste dynamique** afin de cibler plusieurs scénarios de cas d’usage pour vos appareils cibles. Cette rubrique décrit les concepts liés aux **Manifestes dynamiques** et fournit des exemples de scénarios dans lesquels vous pouvez utiliser cette fonctionnalité.

> [!NOTE]
> Les manifestes dynamiques ne changent pas l'élément multimédia ni son manifeste par défaut.

## <a name="overview-of-manifests"></a>Vue d'ensemble des manifestes

Azure Media Services prend en charge les protocoles HLS, MPEG DASH et Smooth Streaming. Dans le cadre du l'[empaquetage dynamique](dynamic-packaging-overview.md), les manifestes du client de streaming (liste de lecture principale HLS, DASH MPD (Media Presentation Description) et Smooth Streaming) sont générés de façon dynamique en fonction du sélecteur de format de l'URL. Pour plus d’informations, consultez les protocoles de remise présentés dans [Workflow à la demande commun](dynamic-packaging-overview.md#to-prepare-your-source-files-for-delivery).

### <a name="get-and-examine-manifest-files"></a>Obtenir et examiner des fichiers manifeste

Vous spécifiez une liste de conditions de propriété de pistes de filtrage basée sur les pistes de votre flux (en direct ou en vidéo à la demande [VOD]) qui doivent être incluses dans un manifeste créé de façon dynamique. Pour obtenir et examiner les propriétés des pistes, vous devez d’abord charger le manifeste Smooth Streaming.

Le tutoriel [Charger, encoder et diffuser en continu des fichiers avec .NET](stream-files-tutorial-with-api.md#get-streaming-urls) explique comment créer des URL de diffusion en continu avec .NET. Si vous exécutez l’application, une des URL pointe vers le manifeste Smooth Streaming : `https://amsaccount-usw22.streaming.media.azure.net/00000000-0000-0000-0000-0000000000000/ignite.ism/manifest`.<br/> Copiez et collez l’URL dans la barre d’adresse d’un navigateur. Le fichier va être téléchargé. Vous pouvez l’ouvrir dans n’importe quel éditeur de texte.

Pour obtenir un exemple REST, consultez [Charger, encoder et diffuser en continu des fichiers avec REST](stream-files-tutorial-with-rest.md#list-paths-and-build-streaming-urls).

### <a name="monitor-the-bitrate-of-a-video-stream"></a>Surveiller la vitesse de transmission d’un flux vidéo

Vous pouvez utiliser la [page de démo Azure Media Player](https://aka.ms/azuremediaplayer) pour surveiller la vitesse de transmission d’un flux vidéo. La page de démo affiche des informations de diagnostic dans l'onglet **Diagnostics** :

![Diagnostics Azure Media Player][amp_diagnostics]

### <a name="examples-urls-with-filters-in-query-string"></a>Exemples : URL avec des filtres dans la chaîne de requête

Vous pouvez appliquer des filtres aux protocoles de diffusion en continu ABR : HLS, MPEG-DASH et Smooth Streaming. Le tableau suivant présente des exemples d’URL utilisant des filtres :

|Protocol|Exemple|
|---|---|
|HLS|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=m3u8-aapl,filter=myAccountFilter)`|
|MPEG DASH|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=mpd-time-csf,filter=myAssetFilter)`|
|Smooth Streaming|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(filter=myAssetFilter)`|

## <a name="rendition-filtering"></a>Filtrage de rendu

Vous pouvez choisir d'encoder votre élément multimédia pour plusieurs profils d'encodage (H.264 Baseline, H.264 High, AACL, AACH, Dolby Digital Plus) et plusieurs qualités de débit binaire. Toutefois, tous les appareils client ne prennent pas en charge tous les profils et débits binaires de tous vos éléments multimédias. Par exemple, les anciens appareils Android prennent uniquement en charge H.264 Baseline+AACL. L'envoi à des vitesses de transmission plus élevées à un appareil qui ne peut pas en tirer partie revient à gaspiller de la bande passante et entraîne des calculs inutiles pour l'appareil. Ce dernier doit en effet décoder toutes les informations données, pour finalement les mettre à l'échelle afin de les afficher.

Avec un manifeste dynamique, vous pouvez créer des profils d'appareil mobile, de console ou HD/SD, et inclure les pistes et les qualités à intégrer dans chaque profil. C'est ce qu'on appelle le filtrage de rendu. Le diagramme suivant en présente un exemple.

![Exemple de filtrage de rendu avec un manifeste dynamique][renditions2]

Dans l’exemple suivant, un encodeur a été utilisé pour encoder un élément multimédia mezzanine en sept rendus vidéo ISO MP4s (de 180 p à 1 080 p). La ressource encodée peut être [empaquetée de manière dynamique](dynamic-packaging-overview.md) dans l’un des protocoles de streaming suivants : HLS, MPEG DASH et Smooth.

Le haut du diagramme suivant illustre le manifeste HLS de l'élément multimédia sans filtre. (Il contient les sept rendus.)  Dans la partie inférieure gauche, le diagramme montre un manifeste HLS auquel un filtre nommé « ott » a été appliqué. Le filtre « ott » indique de supprimer toutes les vitesses de transmission inférieures à 1 Mbit/s, ce qui entraîne la suppression des deux niveaux de qualité les plus bas dans la réponse. Dans la partie inférieure droite, le diagramme montre le manifeste HLS auquel un filtre nommé « mobile » a été appliqué. Le filtre « mobile » indique de supprimer les rendus pour lesquels la résolution est supérieure à 720 p, ce qui entraîne la suppression des deux rendus à 1 080 p.

![Filtrage de rendu avec un manifeste dynamique][renditions1]

## <a name="removing-language-tracks"></a>Suppression des pistes de langue
Vos éléments multimédias peuvent inclure plusieurs langues audio telles que l’anglais, l’espagnol, le français, etc. Généralement, le kit SDK du lecteur gère la sélection de pistes audio par défaut et les pistes audio disponibles par sélection de l'utilisateur.

Le développement de ces kits SDK de lecteur est difficile car différentes implémentations sont requises entre les infrastructures de lecteur propres aux appareils. De plus, sur certaines plateformes, les API de lecteur sont limitées et n’incluent pas de fonctionnalité de sélection audio, ce qui ne permet pas aux utilisateurs de sélectionner ou de modifier la piste audio par défaut. Avec les filtres d'éléments multimédias, vous pouvez contrôler le comportement en créant des filtres qui incluent uniquement les langues audio souhaitées.

![Filtrage des pistes de langue avec un manifeste dynamique][language_filter]

## <a name="trimming-the-start-of-an-asset"></a>Découpage du début d'un élément multimédia

Pour la plupart des événements de diffusion en continu en direct, les opérateurs exécutent des tests avant l'événement réel. Par exemple, ils peuvent inclure une ardoise comme celle-ci avant le début de l’événement : « Le programme va commencer dans un instant ».

Si le programme est en cours d’archivage, le test et les données d’ardoise sont également archivés et donc inclus dans la présentation. Toutefois, ces informations ne doivent pas être montrées aux clients. Avec un manifeste dynamique, vous pouvez créer un filtre sur l'heure de début et supprimer les données indésirables du manifeste.

![Découpage du début d’un élément multimédia avec un manifeste dynamique][trim_filter]

## <a name="creating-subclips-views-from-a-live-archive"></a>Création de sous-clips (vues) à partir d’une archive en direct

De nombreux événements en direct ont une durée d'exécution longue et une archive en direct peut inclure plusieurs événements. Une fois l’événement en direct terminé, les diffuseurs peuvent décomposer cette archive en direct selon des séquences de démarrage et d’arrêt du programme logiques.

Vous pouvez publier ces programmes virtuels séparément, sans post-traitement de l'archive en direct, et sans créer d'éléments multimédias distincts (qui ne bénéficient pas des fragments mis en cache existants dans les CDN). Par exemple, ces programmes virtuels peuvent correspondre aux mi-temps d’un match de football ou de basket, aux tours de batte au base-ball ou à des événements sportifs ponctuels.

Avec un manifeste dynamique, vous pouvez créer des filtres en utilisant des heures de début/fin et créer des vues virtuelles par-dessus votre archive en direct.

![Filtre de sous-clip avec un manifeste dynamique][subclip_filter]

Voici l'élément multimédia filtré :

![Ressource filtrée avec un manifeste dynamique][skiing]

## <a name="adjusting-the-presentation-window-dvr"></a>Ajustement de la fenêtre de présentation (DVR)

Azure Media Services propose une archive circulaire où la durée peut être configurée entre 1 minute et 25 heures. Le filtrage de manifeste peut servir à créer une fenêtre DVR de substitution par-dessus l'archive, sans supprimer le média. Il existe de nombreux scénarios où les diffuseurs veulent fournir une fenêtre DVR limitée à déplacer avec la session en direct et qui garde en même temps une plus grande fenêtre d’archivage. Un diffuseur peut utiliser les données en dehors de la fenêtre DVR pour mettre en avant des clips ou il peut proposer des fenêtres DVR différentes pour différents appareils. Par exemple, la plupart des appareils mobiles ne gèrent pas les grandes fenêtres DVR (une fenêtre DVR peut durer 2 minutes pour les appareils mobiles alors qu’elle dure une heure pour les ordinateurs de bureau).

![Fenêtre DVR avec un manifeste dynamique][dvr_filter]

## <a name="adjusting-livebackoff-live-position"></a>Ajustement de LiveBackoff (position en direct)

Le filtrage de manifeste peut servir à supprimer plusieurs secondes à partir du bord en direct d'un programme en direct. Il permet aux diffuseurs de regarder la présentation sur le point de publication d'aperçu et de créer des points d'insertion d'annonce avant que les utilisateurs ne reçoivent le flux (reculé de 30 secondes). Les diffuseurs peuvent ensuite transmettre ces annonces à leurs infrastructures clientes à temps pour qu'ils reçoivent et traitent les informations avant l'opportunité d'annonce.

Outre la prise en charge des annonces, le paramètre LiveBackoff peut servir à ajuster la position des utilisateurs, afin que lorsque les clients dérivent et atteignent la session en direct, ils puissent toujours obtenir des fragments du serveur. De cette façon, les clients ne recevront pas d'erreur HTTP 404 ou 412.

![Filtrer les interruptions dynamiques avec un manifeste dynamique][livebackoff_filter]

## <a name="combining-multiple-rules-in-a-single-filter"></a>Combinaison de plusieurs règles dans un seul filtre

Vous pouvez combiner plusieurs règles de filtrage dans un filtre unique. Par exemple, vous pouvez définir une « règle de plage » pour supprimer les ardoises d’une archive en direct tout en éliminant les débits binaires disponibles. Lorsque vous appliquez plusieurs règles de filtrage, le résultat final est l'intersection de toutes les règles.

![Règles de filtrage multiples avec un manifeste dynamique][multiple-rules]

## <a name="combining-multiple-filters-filter-composition"></a>Combinaison de plusieurs filtres (composition de filtre)

Vous pouvez également combiner plusieurs filtres dans une URL unique. Le scénario suivant explique les avantages de la combinaison de filtres :

1. Vous devez filtrer vos qualités vidéo pour les appareils mobiles tels qu’Android ou iPad (afin de limiter les qualités vidéo). Pour supprimer les qualités indésirables, vous devez créer un filtre de compte adapté aux profils de vos appareils. Vous pouvez utiliser des filtres de compte pour tous les éléments multimédia liés au même compte Media Services sans aucune autre association.
1. Vous pouvez aussi avoir besoin d’ajuster l’heure de début et de fin d’un élément multimédia. Pour effectuer le découpage, vous devez créer un filtre d'élément multimédia et définir l'heure de début et de fin.
1. Vous pouvez combiner ces deux filtres. Sans combinaison, vous devrez ajouter le filtrage de la qualité au filtre de réglage, ce qui compliquera l'utilisation de filtres.

Pour combiner des filtres, définissez leurs noms dans l’URL du manifeste ou de la liste de lecture en les délimitant avec des points-virgules. Supposons que vous ayez un filtre nommé *MyMobileDevice* qui filtre les qualités et un autre nommé *MyStartTime* qui définit une heure de début spécifique. Vous pouvez combiner jusqu’à trois filtres.

Pour plus d’informations, consultez [ce billet de blog](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/)

## <a name="considerations-and-limitations"></a>Observations et limitations

- Les valeurs **forceEndTimestamp**, **presentationWindowDuration** et **liveBackoffDuration** ne doivent pas être définies pour un filtre de vidéo à la demande (VOD). Elles sont uniquement utilisées pour les scénarios de filtre en direct.
- Un manifeste dynamique fonctionne dans les limites d'un groupe d'images (GOP - images clés), par conséquent, le découpage est précis au niveau du GOP.
- Vous pouvez utiliser le même nom pour les filtres de compte et d'élément multimédia. Les filtres d'élément multimédia ont une priorité plus élevée et remplacent les filtres de compte.
- Si vous mettez à jour un filtre, 2 minutes peuvent être nécessaires pour que le point de terminaison de streaming actualise les règles. Si vous avez utilisé des filtres pour servir le contenu (avant de mettre le contenu en cache dans des proxys et des caches CDN), la mise à jour de ces filtres peut entraîner des défaillances du lecteur. Nous vous recommandons de vider le cache après la mise à jour du filtre. Si cela s'avère impossible, envisagez d'utiliser un autre filtre.
- Les clients doivent télécharger manuellement le manifeste et analyser l'heure de début et l'échelle de temps exacts.

    - Pour déterminer les propriétés des pistes d'un élément multimédia, [récupérez et examinez le fichier manifeste](#get-and-examine-manifest-files).
    - Pour définir les propriétés d'horodatage du filtre d'élément multimédia, la formule est la suivante : <br/>startTimestamp = &lt;heure de début dans le manifeste&gt; +  &lt;heure de début prévue du filtre en secondes&gt; * échelle de temps

## <a name="next-steps"></a>Étapes suivantes

Les articles suivants expliquent comment créer des filtres par programmation :  

- [Créer des filtres avec les API REST](filters-dynamic-manifest-rest-howto.md)
- [Créer des filtres avec .NET](filters-dynamic-manifest-dotnet-howto.md)
- [Créer des filtres avec l’interface CLI](filters-dynamic-manifest-cli-howto.md)

[renditions1]: ./media/filters-dynamic-manifest-overview/media-services-rendition-filter.png
[renditions2]: ./media/filters-dynamic-manifest-overview/media-services-rendition-filter2.png

[rendered_subclip]: ./media/filters-dynamic-manifests/media-services-rendered-subclip.png
[timeline_trim_event]: ./media/filters-dynamic-manifests/media-services-timeline-trim-event.png
[timeline_trim_subclip]: ./media/filters-dynamic-manifests/media-services-timeline-trim-subclip.png

[multiple-rules]:./media/filters-dynamic-manifest-overview/media-services-multiple-rules-filters.png

[subclip_filter]: ./media/filters-dynamic-manifest-overview/media-services-subclips-filter.png
[trim_event]: ./media/filters-dynamic-manifests/media-services-timeline-trim-event.png
[trim_subclip]: ./media/filters-dynamic-manifests/media-services-timeline-trim-subclip.png
[trim_filter]: ./media/filters-dynamic-manifest-overview/media-services-trim-filter.png
[redered_subclip]: ./media/filters-dynamic-manifests/media-services-rendered-subclip.png
[livebackoff_filter]: ./media/filters-dynamic-manifest-overview/media-services-livebackoff-filter.png
[language_filter]: ./media/filters-dynamic-manifest-overview/media-services-language-filter.png
[dvr_filter]: ./media/filters-dynamic-manifest-overview/media-services-dvr-filter.png
[skiing]: ./media/filters-dynamic-manifest-overview/media-services-skiing.png
[amp_diagnostics]: ./media/filters-dynamic-manifest-overview/amp_diagnostics.png
