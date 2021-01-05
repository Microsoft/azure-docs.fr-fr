---
title: Définition de filtres dans Azure Media Services
description: Cette rubrique décrit comment créer des filtres pour que votre client puisse les utiliser pour diffuser des sections spécifiques d'un flux. Media Services crée des manifestes dynamiques pour obtenir cette diffusion sélective.
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
ms.openlocfilehash: bb5561ced93c3f5a899c6e48fdab0f14e52914bb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89291548"
---
# <a name="filters"></a>Filtres

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Lorsque vous transmettez du contenu à des clients (événements en streaming en direct ou vidéos à la demande), le fichier manifeste de l'élément multimédia par défaut ne permet pas toujours au client d'interagir avec le contenu comme il le voudrait. Azure Media Services propose des [manifestes dynamiques](filters-dynamic-manifest-overview.md) basés sur des filtres prédéfinis. 

Les filtres sont des règles côté serveur qui permettent à vos clients d’effectuer des opérations comme : 

- Lire une seule section d'une vidéo (au lieu de la vidéo entière). Par exemple :
  - Réduisez le manifeste pour afficher un sous-clip d'un événement en direct (« filtrage de sous-clip »), ou
  - Découpez le début d'une vidéo (« découpage d'une vidéo »).
- Distribuer uniquement les rendus spécifiés et/ou les pistes de langue spécifiées qui sont prises en charge par l’appareil utilisé pour lire le contenu (« filtrage de rendu »). 
- Ajustez la fenêtre de présentation (DVR) afin de fournir une longueur limitée de la fenêtre du DVR dans le lecteur (« ajustement de la fenêtre de présentation »).

Avec Media Services, vous pouvez créer des **filtres de compte** et des **filtres d’élément multimédia** à appliquer à votre contenu. En outre, vous pouvez associer vos filtres créés au préalable à un **localisateur de streaming**.

## <a name="defining-filters"></a>Définition des filtres

Il existe deux types de filtres : 

* [Filtres de compte](/rest/api/media/accountfilters) (globaux) : applicables à n'importe quel élément multimédia dans le compte Azure Media Services et dont la durée de vie correspond à celle du compte.
* [Filtres d’élément multimédia](/rest/api/media/assetfilters) (locaux) : applicables uniquement à un élément multimédia avec lequel le filtre a été associé au moment de la création et dont la durée de vie correspond à celle de l'élément multimédia. 

Les types **Filtres de compte** et **Filtres d’élément multimédia** utilisent exactement les mêmes propriétés pour définir/décrire le filtre. Sauf lorsque vous créez le **filtre d’élément multimédia**, vous devez spécifier le nom de la ressource à laquelle vous souhaitez associer le filtre.

Selon votre scénario, vous choisissez le type de filtre le plus adapté (filtre d’élément multimédia ou de compte). Les filtres de compte conviennent aux profils d'appareil (filtrage de rendu) alors que les filtres d’élément multimédia peuvent être utilisés pour découper un élément multimédia spécifique.

Utilisez les propriétés suivantes pour décrire les filtres. 

|Name|Description|
|---|---|
|firstQuality|Première qualité de vitesse de transmission du filtre.|
|presentationTimeRange|Plage horaire de présentation. Cette propriété sert à filtrer les points de début/fin du manifeste, la durée de la fenêtre de présentation, et la position de départ du flux en direct. <br/>Pour plus d’informations, consultez [PresentationTimeRange](#presentationtimerange).|
|tracks|Conditions de sélection des pistes. Pour plus d’informations, consultez [tracks](#tracks)|

### <a name="presentationtimerange"></a>presentationTimeRange

Utilisez cette propriété avec les **filtres d’élément multimédia**. Il n’est pas recommandé de définir la propriété avec des **filtres de compte**.

|Name|Description|
|---|---|
|**endTimestamp**|Applicable à la vidéo à la demande (VoD).<br/>Pour la présentation en streaming en direct, cette propriété est ignorée (mode silencieux) puis appliquée lorsque la présentation se termine et que le flux est transmis en VoD.<br/>Il s’agit d’une valeur longue qui représente un point de terminaison absolu de la présentation, arrondie au début GOP suivant le plus proche. L’unité étant l’échelle de temps, un endTimestamp de 1800000000 correspond à 3 minutes.<br/>Utilisez startTimestamp et endTimestamp pour découper les fragments qui seront présents dans la sélection (manifeste).<br/>Par exemple, startTimestamp = 40000000 et endTimestamp = 100000000 utilisant l'échelle de temps par défaut génèrent une playlist contenant des fragments entre 4 secondes et 10 secondes de la présentation VoD. Si un fragment approche la limite, le fragment entier sera inclus dans le manifeste.|
|**forceEndTimestamp**|S’applique au streaming en direct uniquement.<br/>Indique si la propriété endTimestamp doit être présente. Si la valeur est true, la propriété endTimestamp doit être spécifiée, à défaut de quoi un code de requête incorrecte est renvoyé.<br/>Valeurs autorisées : false, true.|
|**liveBackoffDuration**|S’applique au streaming en direct uniquement.<br/> Cette valeur définit la dernière position active qu'un client peut rechercher.<br/>Cette propriété vous permet de différer la position de lecture en direct et de créer une mémoire tampon côté serveur pour les lecteurs.<br/>L’unité de cette propriété est l’échelle de temps (voir ci-dessous).<br/>La durée d’interruption en direct maximale est de 300 secondes (3000000000).<br/>Par exemple, une valeur de 2000000000 signifie que le dernier contenu disponible est différé de 20 secondes par rapport à la session en direct.|
|**presentationWindowDuration**|S’applique au streaming en direct uniquement.<br/>Utilisez presentationWindowDuration pour appliquer une fenêtre glissante des fragments à ajouter à une sélection.<br/>L’unité de cette propriété est l’échelle de temps (voir ci-dessous).<br/>Par exemple, définissez presentationWindowDuration=1200000000 pour appliquer une fenêtre glissante de deux minutes. Les éléments multimédia situés à 2 minutes de la session en direct figureront dans la sélection. Si un fragment approche la limite, le fragment entier sera inclus dans la sélection. La durée minimale de la fenêtre de présentation est de 60 secondes.|
|**startTimestamp**|Applicable à la vidéo à la demande (VoD) ou au streaming en direct.<br/>Cette valeur longue représente un point de début absolu du flux. Elle est arrondie au début GOP suivant le plus proche. L’unité étant l’échelle de temps, un endTimestamp de 150000000 correspond à 15 minutes.<br/>Utilisez startTimestamp et endTimestampp pour découper les fragments qui seront présents dans la sélection (manifeste).<br/>Par exemple, startTimestamp = 40000000 et endTimestamp = 100000000 utilisant l'échelle de temps par défaut génèrent une playlist contenant des fragments entre 4 secondes et 10 secondes de la présentation VoD. Si un fragment approche la limite, le fragment entier sera inclus dans le manifeste.|
|**timescale**|S’applique à tous les timestamps et durées d'une plage horaire de présentation, spécifiés sous forme d'incréments par seconde.<br/>10000000 correspond à la valeur par défaut - dix millions d'incréments par seconde, où chaque incrément correspondrait à 100 nanosecondes.<br/>Par exemple, si vous souhaitez définir un startTimestamp sur 30 secondes, vous opterez pour une valeur de 300000000 par rapport à l'échelle de temps par défaut.|

### <a name="tracks"></a>Pistes

Vous spécifiez une liste de conditions de propriétés de suivi de filtre (FilterTrackPropertyConditions) selon laquelle les pistes de votre flux de données (streaming en direct ou vidéo à la demande) doivent être incluses dans le manifeste créé dynamiquement. Les filtres sont combinés à l’aide d’une opération logique **AND** et **OR**.

Les conditions de propriétés de suivi de filtre décrivent les types de suivi, les valeurs (spécifiées dans le tableau suivant) et les opérations (Equal, NotEqual). 

|Name|Description|
|---|---|
|**Bitrate**|Utilisez la vitesse de transmission de la piste pour le filtrage.<br/><br/>La valeur recommandée est une plage de vitesses de transmission, en bits par seconde. Par exemple, « 0-2427000 ».<br/><br/>Remarque : vous pouvez utiliser une valeur de vitesse de transmission spécifique, comme 250000 (bits par seconde), mais cette approche n'est pas recommandée car les vitesses de transmission exacts peuvent varier d’un élément multimédia à un autre.|
|**FourCC**|Utilisez la valeur FourCC de la piste pour le filtrage.<br/><br/>La valeur est le premier élément du format de codecs, tel que spécifié dans [RFC 6381](https://tools.ietf.org/html/rfc6381). À l’heure actuelle, les codecs suivants sont pris en charge : <br/>Pour la vidéo : « avc1 », « hev1 », « hvc1 »<br/>Pour l’audio : « mp4a », « ec-3 »<br/><br/>Pour déterminer les valeurs FourCC des pistes d’un élément multimédia, obtenez et examinez le fichier manifeste.|
|**Langage**|Utilisez la langue de la piste pour le filtrage.<br/><br/>La valeur est la balise d’une langue que vous souhaitez inclure, comme indiqué dans RFC 5646. Exemple : « en ».|
|**Nom**|Utilisez le nom de la piste pour le filtrage.|
|**Type**|Utilisez le type de la piste pour le filtrage.<br/><br/>Les valeurs suivantes sont autorisées : « video », « audio » ou « text ».|

### <a name="example"></a>Exemple

L’exemple suivant définit un filtre de streaming en direct : 

```json
{
  "properties": {
    "presentationTimeRange": {
      "startTimestamp": 0,
      "endTimestamp": 170000000,
      "presentationWindowDuration": 9223372036854776000,
      "liveBackoffDuration": 0,
      "timescale": 10000000,
      "forceEndTimestamp": false
    },
    "firstQuality": {
      "bitrate": 128000
    },
    "tracks": [
      {
        "trackSelections": [
          {
            "property": "Type",
            "operation": "Equal",
            "value": "Audio"
          },
          {
            "property": "Language",
            "operation": "NotEqual",
            "value": "en"
          },
          {
            "property": "FourCC",
            "operation": "NotEqual",
            "value": "EC-3"
          }
        ]
      },
      {
        "trackSelections": [
          {
            "property": "Type",
            "operation": "Equal",
            "value": "Video"
          },
          {
            "property": "Bitrate",
            "operation": "Equal",
            "value": "3000000-5000000"
          }
        ]
      }
    ]
  }
}
```

## <a name="associating-filters-with-streaming-locator"></a>Associer des filtres à un localisateur de streaming

Vous pouvez spécifier une liste de [filtres de compte ou d'élément multimédia](filters-concept.md) sur votre [localisateur de streaming](/rest/api/media/streaminglocators/create#request-body). Le [packager dynamique](dynamic-packaging-overview.md) applique cette liste de filtres avec ceux spécifiés par votre client dans l’URL. Cette combinaison génère un [manifeste dynamique](filters-dynamic-manifest-overview.md) qui est basé sur les filtres spécifiés dans l’URL ainsi que sur ceux que vous spécifiez dans le localisateur de streaming. 

Regardez les exemples suivants :

* [Associer des filtres à un localisateur de streaming - .NET](filters-dynamic-manifest-dotnet-howto.md#associate-filters-with-streaming-locator)
* [Associer des filtres à un localisateur de streaming - CLI](filters-dynamic-manifest-cli-howto.md#associate-filters-with-streaming-locator)

## <a name="updating-filters"></a>Mise à jour de filtres
 
Contrairement aux filtres, les **localisateurs de streaming** ne peuvent pas être mis à jour. 

Il est déconseillé de mettre à jour la définition des filtres associés à un **localisateur de streaming** activement publié, notamment lorsque CDN est activé. Les serveurs et CDN de streaming peuvent disposer de caches internes susceptibles de renvoyer des données en cache obsolètes. 

Si la définition du filtre doit être modifiée, envisagez de créer un nouveau filtre et de l'ajouter à l'URL du **localisateur de streaming** ou de publier un nouveau **localisateur de streaming** faisant directement référence au filtre.

## <a name="next-steps"></a>Étapes suivantes

Les articles suivants vous montrent comment créer des filtres par programmation.  

- [Créer des filtres avec les API REST](filters-dynamic-manifest-rest-howto.md)
- [Créer des filtres avec .NET](filters-dynamic-manifest-dotnet-howto.md)
- [Créer des filtres avec l’interface CLI](filters-dynamic-manifest-cli-howto.md)
