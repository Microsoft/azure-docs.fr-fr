---
title: Examiner la sortie de Video Indexer générée par l’API v2 - Azure
titleSuffix: Azure Media Services
description: Cette rubrique examine la sortie de Video Indexer Azure Media Services générée par l’API v2.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 11/16/2020
ms.author: juliako
ms.openlocfilehash: bf48f873127a12c3cabb28da33d34cedcda2793b
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94831564"
---
# <a name="examine-the-video-indexer-output"></a>Examiner la sortie Video Indexer

Quand une vidéo est indexée, Video Indexer produit le contenu JSON qui contient les détails des insights vidéo spécifiés. Ces insights incluent : des transcriptions, des OCR, des visages, des rubriques, des blocs, etc. Chaque type d’insight comprend des instances d’intervalle de temps qui indiquent quand l’insight apparaît dans la vidéo. 

Vous pouvez examiner visuellement les insights résumés de la vidéo en appuyant sur le bouton **Lire** sur le site web de [Video Indexer](https://www.videoindexer.ai/). 

Vous pouvez également utiliser l’API en appelant l’API **Get Video Index** et, si l’état de la réponse est OK, vous obtenez une sortie JSON détaillée en tant que contenu de la réponse.

![Insights](./media/video-indexer-output-json/video-indexer-summarized-insights.png)

Cet article examine la sortie Video Indexer (contenu JSON). <br/>Pour plus d’informations sur les fonctionnalités et les insights à votre disposition, consultez [Insights de Video Indexer](video-indexer-overview.md#video-insights).

> [!NOTE]
> Tous les jetons d’accès dans Video Indexer arrivent à expiration au bout d’une heure.

## <a name="get-the-insights"></a>Obtenir des insights

### <a name="insightsoutput-produced-in-the-websiteportal"></a>Insights/sortie produits dans le site web ou le portail

1. Accédez au site web [Video Indexer](https://www.videoindexer.ai/) et connectez-vous.
1. Recherchez une vidéo dont vous souhaitez examiner la sortie.
1. Appuyez sur **Lecture**.
1. Sélectionnez l’onglet **Insights** (insights résumés) ou l’onglet **Chronologie** (permet de filtrer les insights pertinents).
1. Téléchargez les artefacts et ce qu’ils contiennent.

Pour plus d’informations, voir [View and edit video insights](video-indexer-view-edit.md) (Afficher et modifier les insights des vidéos).

## <a name="insightsoutput-produced-by-api"></a>Insights/sortie produits par l’API

1. Pour récupérer le fichier JSON, appelez [Get Video Index API](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Get-Video-Index?)
1. Si vous êtes également intéressé par des artefacts spécifiques, appelez [Get Video Artifact Download URL API](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Get-Video-Artifact-Download-Url?)

    Dans l’appel d’API, spécifiez le type d’artefact demandé (OCR, Faces, Key frames etc.)

## <a name="root-elements-of-the-insights"></a>Éléments racines des insights

|Nom|Description|
|---|---|
|accountId|ID du compte Video Indexer de la playlist.|
|id|ID de la playlist.|
|name|Nom de la playlist.|
|description|Description de la playlist.|
|userName|Nom du créateur de la playlist.|
|created|Heure de création de la playlist.|
|privacyMode|Mode de confidentialité de la playlist (Publique/Privée).|
|state|État de la playlist (chargée, en cours de traitement, traitée, en échec, sous contrôle).|
|isOwned|Indique si la playlist a été créée par l’utilisateur actuel.|
|isEditable|Indique si l’utilisateur actuel est autorisé à modifier la playlist.|
|isBase|Indique si la playlist est une playlist de base (une vidéo) ou si elle est constituée d’autres vidéos (dérivées).|
|durationInSeconds|Durée totale de la playlist.|
|summarizedInsights|Contient un attribut [summarizedInsights](#summarizedinsights).
|videos|Liste des [vidéos](#videos) constituant la playlist.<br/>Si cette playlist est constituée d’intervalles de temps d’autres vidéos (dérivées), les vidéos de cette liste ne contiennent que les données des intervalles de temps inclus.|

```json
{
  "accountId": "bca61527-1221-bca6-1527-a90000002000",
  "id": "abc3454321",
  "name": "My first video",
  "description": "I am trying VI",
  "userName": "Some name",
  "created": "2018/2/2 18:00:00.000",
  "privacyMode": "Private",
  "state": "Processed",
  "isOwned": true,
  "isEditable": false,
  "isBase": false,
  "durationInSeconds": 120, 
  "summarizedInsights" : { . . . }
  "videos": [{ . . . }]
}
```

## <a name="summarizedinsights"></a>summarizedInsights

Cette section présente le résumé des insights.

|Attribut | Description|
|---|---|
|name|Nom de la vidéo. Exemple : Azure Monitor.|
|id|ID de la vidéo. Exemple : 63c6d532ff.|
|privacyMode|La décomposition peut avoir l’un des modes suivants : **Privé**, **Public**. **Public** : tous les membres de votre compte et toutes les personnes disposant d’un lien vers la vidéo peuvent la visionner. **Privé** : tous les membres de votre compte peuvent la visionner.|
|duration|Contient la durée d’un insight. La durée est exprimée en secondes.|
|thumbnailVideoId|ID de la vidéo à partir de laquelle la vidéo miniature a été réalisée.
|thumbnailId|ID de la miniature de la vidéo. Pour obtenir la vidéo miniature réelle, appelez la méthode [Get-Thumbnail](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Video-Thumbnail) et transmettez-lui les attributs thumbnailVideoId et thumbnailId.|
|faces/animatedCharacters|Peut contenir zéro ou plusieurs visages. Pour plus d’informations, consultez la section [faces/animatedCharacters](#facesanimatedcharacters).|
|mots clés|Peut contenir zéro ou plusieurs mots clés. Pour plus d’informations, consultez la section [keywords](#keywords).|
|sentiments|Peut contenir zéro ou plusieurs sentiments. Pour plus d’informations, consultez la section [sentiments](#sentiments).|
|audioEffects| Peut contenir zéro ou plusieurs éléments audioEffect. Pour plus d’informations, consultez la section [audioEffects](#audioeffects).|
|étiquettes| Peut contenir zéro ou plusieurs étiquettes. Pour plus d’informations, consultez la section [labels](#labels).|
|brands| Peut contenir zéro ou plusieurs marques. Pour plus d’informations, consultez la section [brands](#brands).|
|statistiques | Pour plus d’informations, consultez la section [statistics](#statistics).|
|émotions| Peut contenir zéro ou plusieurs émotions. Pour plus d’informations, consultez la section [émotions](#emotions).|
|topics|Peut contenir zéro ou plusieurs rubriques. L’insight [rubriques](#topics).|

## <a name="videos"></a>videos

|Nom|Description|
|---|---|
|accountId|ID du compte Video Indexer de la vidéo.|
|id|ID de la vidéo.|
|name|Nom de la vidéo.
|state|État de la vidéo (chargée, en cours de traitement, traitée, en échec, sous contrôle).|
|processingProgress|Progression du traitement (par exemple, 20 %).|
|failureCode|Code en cas d’échec du traitement (par exemple, « UnsupportedFileType »).|
|failureMessage|Message en cas d’échec du traitement.|
|externalId|ID externe de la vidéo (s’il est spécifié par l’utilisateur).|
|externalUrl|URL externe de la vidéo (si elle est spécifiée par l’utilisateur).|
|metadata|Métadonnées externes de la vidéo (si elles sont spécifiées par l’utilisateur).|
|isAdult|Indique si la vidéo a été vérifiée manuellement et identifiée comme une vidéo pour adultes.|
|insights|Objet insights. Pour plus d’informations, consultez la section [insights](#insights).|
|thumbnailId|ID de la miniature de la vidéo. Pour obtenir la vidéo miniature réelle, appelez la méthode [Get-Thumbnail](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Video-Thumbnail) et transmettez-lui l’ID de la vidéo et l’attribut thumbnailId.|
|publishedUrl|URL de diffusion en continu de la vidéo.|
|publishedUrlProxy|URL à partir de laquelle diffuser la vidéo en continu (pour les appareils Apple).|
|viewToken|Jeton d’affichage de courte durée pour la diffusion en continu de la vidéo.|
|sourceLanguage|Langue source de la vidéo.|
|langage|Langue réelle de la vidéo (traduction).|
|indexingPreset|Présélection utilisée pour indexer la vidéo.|
|streamingPreset|Présélection utilisée pour publier la vidéo.|
|linguisticModelId|Modèle CRIS utilisé pour transcrire la vidéo.|
|statistiques | Pour plus d’informations, consultez la section [statistics](#statistics).|

```json
{
    "videos": [{
        "accountId": "2cbbed36-1972-4506-9bc7-55367912df2d",
        "id": "142a356aa6",
        "state": "Processed",
        "privacyMode": "Private",
        "processingProgress": "100%",
        "failureCode": "General",
        "failureMessage": "",
        "externalId": null,
        "externalUrl": null,
        "metadata": null,
        "insights": {. . . },
        "thumbnailId": "89d7192c-1dab-4377-9872-473eac723845",
        "publishedUrl": "https://videvmediaservices.streaming.mediaservices.windows.net:443/d88a652d-334b-4a66-a294-3826402100cd/Xamarine.ism/manifest",
        "publishedProxyUrl": null,
        "viewToken": "Bearer=<token>",
        "sourceLanguage": "En-US",
        "language": "En-US",
        "indexingPreset": "Default",
        "linguisticModelId": "00000000-0000-0000-0000-000000000000"
    }],
}
```
### <a name="insights"></a>insights

Chaque insight (par exemple, les lignes de transcription, les visages, les marques, etc.), contient une liste d’éléments uniques (par exemple, face1, face2, face3), et où chaque élément possède ses propres métadonnées et une liste de ses instances (qui sont des intervalles de temps avec des métadonnées facultatives supplémentaires).

Un visage peut être doté d’un ID, d’un nom, d’une miniature, d’autres métadonnées et d’une liste de ses instances temporelles (exemple : 00:00:05 – 00:00:10, 00:01:00 - 00:02:30 et 00:41:21 – 00:41:49.) Chaque instance temporelle peut compter des métadonnées supplémentaires. Exemple : les coordonnées du rectangle du visage (20, 230, 60, 60).

|Version|Version du code|
|---|---|
|sourceLanguage|Langue source de la vidéo (en supposant une langue principale). Sous la forme d’une chaîne [BCP-47](https://tools.ietf.org/html/bcp47).|
|langage|Langue des insights (traduite à partir de la langue source). Sous la forme d’une chaîne [BCP-47](https://tools.ietf.org/html/bcp47).|
|transcription|L’insight [transcription](#transcript).|
|ocr|L’insight [OCR](#ocr).|
|mots clés|L’insight [mots clés](#keywords).|
|blocks|Peut contenir un ou plusieurs [blocs](#blocks).|
|faces/animatedCharacters|L’insight [faces/animatedCharacters](#facesanimatedcharacters).|
|étiquettes|L’insight [étiquettes](#labels).|
|captures|L’insight [captures](#shots).|
|brands|L’insight [brands](#brands).|
|audioEffects|L’insight [audioEffects](#audioeffects).|
|sentiments|L’insight [sentiments](#sentiments).|
|visualContentModeration|L’insight [visualContentModeration](#visualcontentmoderation).|
|textualContentModeration|L’insight [textualContentModeration](#textualcontentmoderation).|
|émotions| L’insight [émotions](#emotions).|
|topics|L’insight [rubriques](#topics).|
|speakers|L’insight [speakers](#speakers) (intervenants).|

Exemple :

```json
{
  "version": "0.9.0.0",
  "sourceLanguage": "en-US",
  "language": "es-ES",
  "transcript": ...,
  "ocr": ...,
  "keywords": ...,
  "faces": ...,
  "labels": ...,
  "shots": ...,
  "brands": ...,
  "audioEffects": ...,
  "sentiments": ...,
  "visualContentModeration": ...,
  "textualContentModeration": ...
}
```

#### <a name="blocks"></a>blocks

Attribut | Description
---|---
id|ID du bloc.|
instances|Liste des intervalles de temps de ce bloc.|

#### <a name="transcript"></a>transcription

|Nom|Description|
|---|---|
|id|ID de la ligne.|
|text|La transcription proprement dite.|
|confidence|Fiabilité de la précision de la transcription.|
|speakerId|ID de l’intervenant.|
|langage|La langue de la transcription. Permet de prendre en charge la transcription lorsque chaque ligne peut avoir une langue différente.|
|instances|Liste des intervalles de temps pendant lesquels cette ligne est apparue. Si l’instance est un attribut transcript, il n’y a qu’une seule instance.|

Exemple :

```json
"transcript":[
{
  "id":1,
  "text":"Well, good morning everyone and welcome to",
  "confidence":0.8839,
  "speakerId":1,
  "language":"en-US",
  "instances":[
     {
    "adjustedStart":"0:00:10.21",
    "adjustedEnd":"0:00:12.81",
    "start":"0:00:10.21",
    "end":"0:00:12.81"
     }
  ]
},
{
  "id":2,
  "text":"ignite 2016. Your mission at Microsoft is to empower every",
  "confidence":0.8944,
  "speakerId":2,
  "language":"en-US",
  "instances":[
     {
    "adjustedStart":"0:00:12.81",
    "adjustedEnd":"0:00:17.03",
    "start":"0:00:12.81",
    "end":"0:00:17.03"
     }
  ]
},
```

#### <a name="ocr"></a>ocr

|Nom|Description|
|---|---|
|id|ID de la ligne ROC.|
|text|Texte de l’OCR.|
|confidence|Degré de confiance de la reconnaissance.|
|langage|Langue de l’OCR.|
|instances|Liste des intervalles de temps au cours desquels cette OCR est apparue (la même OCR peut apparaître plusieurs fois).|
|height|Hauteur du rectangle OCR|
|top|Emplacement supérieur en px|
|gauche| Emplacement gauche en px|
|width|Largeur du rectangle OCR|

```json
"ocr": [
    {
      "id": 0,
      "text": "LIVE FROM NEW YORK",
      "confidence": 675.971,
      "height": 35,
      "language": "en-US",
      "left": 31,
      "top": 97,
      "width": 400,      
      "instances": [
        {
          "start": "00:00:26",
          "end": "00:00:52"
        }
      ]
    }
  ],
```

#### <a name="keywords"></a>mots clés

|Nom|Description|
|---|---|
|id|ID du mot clé.|
|text|Texte du mot clé.|
|confidence|Degré de confiance de la reconnaissance du mot clé.|
|langage|Langue du mot clé (si traduction).|
|instances|Liste des intervalles de temps pendant lesquels ce mot clé est apparu (un mot clé peut apparaître plusieurs fois).|

```json
{
    id: 0,
    text: "technology",
    confidence: 1,
    language: "en-US",
    instances: [{
            adjustedStart: "0:05:15.782",
            adjustedEnd: "0:05:16.249",
            start: "0:05:15.782",
            end: "0:05:16.249"
    },
    {
            adjustedStart: "0:04:54.761",
            adjustedEnd: "0:04:55.228",
            start: "0:04:54.761",
            end: "0:04:55.228"
    }]
}
```

#### <a name="facesanimatedcharacters"></a>faces/animatedCharacters

L’élément `animatedCharacters` remplace `faces` dans le cas où la vidéo a été indexée avec un modèle de personnages animés. Cette opération s’effectue à l’aide d’un modèle personnalisé dans Custom Vision, et Video Indexer l’exécute sur des images clés.

Si des visages (et non des personnages animés) sont présents, Video Indexer utilise l’API Visage sur toutes les images de la vidéo pour détecter les visages et les célébrités.

|Nom|Description|
|---|---|
|id|ID du visage.|
|name|Nom du visage. Il peut s'agir de la valeur « Unknown #0 », d’une célébrité identifiée ou d'une personne formée par le client.|
|confidence|Degré de confiance de l’identification du visage.|
|description|Description de la célébrité. |
|thumbnailId|ID de la miniature de ce visage.|
|knownPersonId|S’il s’agit d’une personne connue, c’est son ID interne.|
|referenceId|Dans le cas d’une célébrité Bing, il s’agit de son ID Bing.|
|referenceType|Bing uniquement (pour le moment).|
|title|Dans le cas d’une célébrité, il s’agit de son poste (par exemple « PDG de Microsoft »).|
|imageUrl|Dans le cas d’une célébrité, il s’agit de l’URL de l’image associée.|
|instances|Instances où la visage est apparu dans l’intervalle de temps donné. Chaque instance possède également un thumbnailsId. |

```json
"faces": [{
    "id": 2002,
    "name": "Xam 007",
    "confidence": 0.93844,
    "description": null,
    "thumbnailId": "00000000-aee4-4be2-a4d5-d01817c07955",
    "knownPersonId": "8340004b-5cf5-4611-9cc4-3b13cca10634",
    "referenceId": null,
    "title": null,
    "imageUrl": null,
    "instances": [{
        "thumbnailsIds": ["00000000-9f68-4bb2-ab27-3b4d9f2d998e",
        "cef03f24-b0c7-4145-94d4-a84f81bb588c"],
        "adjustedStart": "00:00:07.2400000",
        "adjustedEnd": "00:00:45.6780000",
        "start": "00:00:07.2400000",
        "end": "00:00:45.6780000"
    },
    {
        "thumbnailsIds": ["00000000-51e5-4260-91a5-890fa05c68b0"],
        "adjustedStart": "00:10:23.9570000",
        "adjustedEnd": "00:10:39.2390000",
        "start": "00:10:23.9570000",
        "end": "00:10:39.2390000"
    }]
}]
```

#### <a name="labels"></a>étiquettes

|Nom|Description|
|---|---|
|id|ID de l’étiquette.|
|name|Nom de l’étiquette (par exemple, « ordinateur », « TV »).|
|langage|Langue du nom de l’étiquette (si traduction). BCP-47|
|instances|Liste des intervalles de temps au cours desquels cette étiquette est apparue (une étiquette peut apparaître plusieurs fois). Chaque instance possède un champ de confiance. |


```json
"labels": [
    {
      "id": 0,
      "name": "person",
      "language": "en-US",
      "instances": [
        {
          "confidence": 1.0,
          "start": "00: 00: 00.0000000",
          "end": "00: 00: 25.6000000"
        },
        {
          "confidence": 1.0,
          "start": "00: 01: 33.8670000",
          "end": "00: 01: 39.2000000"
        }
      ]
    },
    {
      "name": "indoor",
      "language": "en-US",
      "id": 1,
      "instances": [
        {
          "confidence": 1.0,
          "start": "00: 00: 06.4000000",
          "end": "00: 00: 07.4670000"
        },
        {
          "confidence": 1.0,
          "start": "00: 00: 09.6000000",
          "end": "00: 00: 10.6670000"
        },
        {
          "confidence": 1.0,
          "start": "00: 00: 11.7330000",
          "end": "00: 00: 20.2670000"
        },
        {
          "confidence": 1.0,
          "start": "00: 00: 21.3330000",
          "end": "00: 00: 25.6000000"
        }
      ]
    }
  ] 
```

#### <a name="scenes"></a>scenes

|Nom|Description|
|---|---|
|id|L’ID de la scène.|
|instances|Liste des intervalles de temps de cette scène (une scène ne peut avoir qu’une instance).|

```json
"scenes":[  
    {  
      "id":0,
      "instances":[  
          {  
            "start":"0:00:00",
            "end":"0:00:06.34",
            "duration":"0:00:06.34"
          }
      ]
    },
    {  
      "id":1,
      "instances":[  
          {  
            "start":"0:00:06.34",
            "end":"0:00:47.047",
            "duration":"0:00:40.707"
          }
      ]
    },

]
```

#### <a name="shots"></a>captures

|Nom|Description|
|---|---|
|id|ID de la capture.|
|keyFrames|Liste des images clés au sein de la capture (chacune possède un ID et une liste d’intervalles de temps d’instances). Chaque instance d’image clé dispose d’un champ thumbnailId, qui contient l’ID de la miniature de l’image clé.|
|instances|Liste des intervalles de temps de cette prise (une prise ne peut avoir qu’une instance).|

```json
"shots":[  
    {  
      "id":0,
      "keyFrames":[  
          {  
            "id":0,
            "instances":[  
                {  
                  "thumbnailId":"00000000-0000-0000-0000-000000000000",
                  "start":"0:00:00.209",
                  "end":"0:00:00.251",
                  "duration":"0:00:00.042"
                }
            ]
          },
          {  
            "id":1,
            "instances":[  
                {  
                  "thumbnailId":"00000000-0000-0000-0000-000000000000",
                  "start":"0:00:04.755",
                  "end":"0:00:04.797",
                  "duration":"0:00:00.042"
                }
            ]
          }
      ],
      "instances":[  
          {  
            "start":"0:00:00",
            "end":"0:00:06.34",
            "duration":"0:00:06.34"
          }
      ]
    },

]
```

#### <a name="brands"></a>brands

Noms des entreprises et des marques de produits détectés dans la reconnaissance vocale et/ou la reconnaissance optique de caractères des vidéos. Cela n’inclut pas la reconnaissance visuelle des marques ni la détection des logos.

|Nom|Description|
|---|---|
|id|ID de la marque.|
|name|Nom de la marque.|
|referenceId | Suffixe de l’URL Wikipédia de marque. Exemple : «Target_Corporation » est le suffixe de [https://en.wikipedia.org/wiki/Target_Corporation](https://en.wikipedia.org/wiki/Target_Corporation).
|referenceUrl | URL Wikipédia de la marque, le cas échéant. Par exemple : [https://en.wikipedia.org/wiki/Target_Corporation](https://en.wikipedia.org/wiki/Target_Corporation).
|description|Description de la marque.|
|tags|Liste des étiquettes prédéfinies qui ont été associées à cette marque.|
|confidence|Valeur de confiance du détecteur de marque de Video Indexer (0-1).|
|instances|Liste des intervalles de temps de cette marque. Chaque instance possède un élément brandType, qui indique si cette marque est apparue dans la transcription ou dans la reconnaissance optique de caractères.|

```json
"brands": [
{
    "id": 0,
    "name": "MicrosoftExcel",
    "referenceId": "Microsoft_Excel",
    "referenceUrl": "http: //en.wikipedia.org/wiki/Microsoft_Excel",
    "referenceType": "Wiki",
    "description": "Microsoft Excel is a sprea..",
    "tags": [],
    "confidence": 0.975,
    "instances": [
    {
        "brandType": "Transcript",
        "start": "00: 00: 31.3000000",
        "end": "00: 00: 39.0600000"
    }
    ]
},
{
    "id": 1,
    "name": "Microsoft",
    "referenceId": "Microsoft",
    "referenceUrl": "http: //en.wikipedia.org/wiki/Microsoft",
    "description": "Microsoft Corporation is...",
    "tags": [
    "competitors",
    "technology"
    ],
    "confidence": 1.0,
    "instances": [
    {
        "brandType": "Transcript",
        "start": "00: 01: 44",
        "end": "00: 01: 45.3670000"
    },
    {
        "brandType": "Ocr",
        "start": "00: 01: 54",
        "end": "00: 02: 45.3670000"
    }
    ]
}
]
```

#### <a name="statistics"></a>statistiques

|Nom|Description|
|---|---|
|CorrespondenceCount|Nombre de correspondances contenues dans la vidéo.|
|SpeakerWordCount|Nombre de mots par intervenant.|
|SpeakerNumberOfFragments|Quantité de fragments de l’intervenant dans une vidéo.|
|SpeakerLongestMonolog|Monologue le plus long de l’intervenant. Si le monologue de l’intervenant comporte des silences, ils sont inclus. Les silences du début et de la fin du monologue sont supprimés.| 
|SpeakerTalkToListenRatio|Le calcul est basé sur le temps passé sur le monologue de l’intervenant (sans les silences intermédiaires) divisé par la durée totale de la vidéo. L’heure est arrondie à la troisième décimale.|

#### <a name="audioeffects"></a>audioEffects

|Nom|Description|
|---|---|
|id|ID de l’effet audio.|
|type|Type d’effet audio (par exemple, applaudissements, discours, silence).|
|instances|Liste des intervalles de temps au cours desquels cet effet audio est apparu.|

```json
"audioEffects": [
{
    "id": 0,
    "type": "Clapping",
    "instances": [
    {
        "start": "00:00:00",
        "end": "00:00:03"
    },
    {
        "start": "00:01:13",
        "end": "00:01:21"
    }
    ]
}
]
```

#### <a name="sentiments"></a>sentiments

Les sentiments sont regroupés par leur champ sentimentType (neutre/positif/négatif). Par exemple, 0-0.1, 0.1-0.2.

|Nom|Description|
|---|---|
|id|ID du sentiment.|
|averageScore |Moyenne de tous les résultats obtenus pour toutes les instances de ce type de sentiment : neutre/positif/négatif|
|instances|Liste des intervalles de temps au cours desquels ce sentiment est apparu.|
|sentimentType |Le type peut être « Positive », « Neutral » ou «Negative ».|

```json
"sentiments": [
{
    "id": 0,
    "averageScore": 0.87,
    "sentimentType": "Positive",
    "instances": [
    {
        "start": "00:00:23",
        "end": "00:00:41"
    }
    ]
}, {
    "id": 1,
    "averageScore": 0.11,
    "sentimentType": "Positive",
    "instances": [
    {
        "start": "00:00:13",
        "end": "00:00:21"
    }
    ]
}
]
```

#### <a name="visualcontentmoderation"></a>visualContentModeration

Le bloc visualContentModeration contient des intervalles de temps qui sont susceptibles de contenir des éléments pour adultes selon Video Indexer. Si ce bloc est vide, aucun contenu pour adultes n’a donc été identifié.

Les vidéos trouvées qui contiennent des éléments pour adultes ou choquants peuvent être disponibles pour un affichage privé uniquement. Les utilisateurs peuvent soumettre une demande de révision manuelle du contenu, auquel cas l’attribut IsAdult contient le résultat de la révision manuelle.

|Nom|Description|
|---|---|
|id|ID de modération du contenu visuel.|
|adultScore|Degré du contenu pour adultes (d’après Content Moderator).|
|racyScore|Degré du contenu choquant (d’après Content Moderator).|
|instances|Liste des intervalles de temps où cette modération du contenu visuel est affichée.|

```json
"VisualContentModeration": [
{
    "id": 0,
    "adultScore": 0.00069,
    "racyScore": 0.91129,
    "instances": [
    {
        "start": "00:00:25.4840000",
        "end": "00:00:25.5260000"
    }
    ]
},
{
    "id": 1,
    "adultScore": 0.99231,
    "racyScore": 0.99912,
    "instances": [
    {
        "start": "00:00:35.5360000",
        "end": "00:00:35.5780000"
    }
    ]
}
] 
```

#### <a name="textualcontentmoderation"></a>textualContentModeration 

|Nom|Description|
|---|---|
|id|ID de modération du contenu textuel.|
|bannedWordsCount |Nombre de mots interdits.|
|bannedWordsRatio |Ratio par rapport au nombre total de mots.|

#### <a name="emotions"></a>émotions

Video Indexer identifie les émotions grâce à des signaux audio et vocaux. L’émotion identifiée peut être : le bonheur, la tristesse, la colère ou la peur.

|Nom|Description|
|---|---|
|id|ID de l’émotion.|
|type|Instant de l’émotion qui a été identifiée grâce à des signaux audio et vocaux. L’émotion peut être : le bonheur, la tristesse, la colère ou la peur.|
|instances|Liste des intervalles de temps pendant lesquels cette émotion est apparue.|

```json
"emotions": [{
    "id": 0,
    "type": "Fear",
    "instances": [{
      "adjustedStart": "0:00:39.47",
      "adjustedEnd": "0:00:45.56",
      "start": "0:00:39.47",
      "end": "0:00:45.56"
    },
    {
      "adjustedStart": "0:07:19.57",
      "adjustedEnd": "0:07:23.25",
      "start": "0:07:19.57",
      "end": "0:07:23.25"
    }]
  },
  {
    "id": 1,
    "type": "Anger",
    "instances": [{
      "adjustedStart": "0:03:55.99",
      "adjustedEnd": "0:04:05.06",
      "start": "0:03:55.99",
      "end": "0:04:05.06"
    },
    {
      "adjustedStart": "0:04:56.5",
      "adjustedEnd": "0:05:04.35",
      "start": "0:04:56.5",
      "end": "0:05:04.35"
    }]
  },
  {
    "id": 2,
    "type": "Joy",
    "instances": [{
      "adjustedStart": "0:12:23.68",
      "adjustedEnd": "0:12:34.76",
      "start": "0:12:23.68",
      "end": "0:12:34.76"
    },
    {
      "adjustedStart": "0:12:46.73",
      "adjustedEnd": "0:12:52.8",
      "start": "0:12:46.73",
      "end": "0:12:52.8"
    },
    {
      "adjustedStart": "0:30:11.29",
      "adjustedEnd": "0:30:16.43",
      "start": "0:30:11.29",
      "end": "0:30:16.43"
    },
    {
      "adjustedStart": "0:41:37.23",
      "adjustedEnd": "0:41:39.85",
      "start": "0:41:37.23",
      "end": "0:41:39.85"
    }]
  },
  {
    "id": 3,
    "type": "Sad",
    "instances": [{
      "adjustedStart": "0:13:38.67",
      "adjustedEnd": "0:13:41.3",
      "start": "0:13:38.67",
      "end": "0:13:41.3"
    },
    {
      "adjustedStart": "0:28:08.88",
      "adjustedEnd": "0:28:18.16",
      "start": "0:28:08.88",
      "end": "0:28:18.16"
    }]
  }
],
```

#### <a name="topics"></a>topics

Video Indexer fait des inférences des principales rubriques à partir de transcriptions. La taxonomie [IPTC](https://iptc.org/standards/media-topics/) de second niveau est incluse lorsque cela est possible. 

|Nom|Description|
|---|---|
|id|ID de la rubrique.|
|name|Nom de la rubrique, par exemple : « Produits pharmaceutiques ».|
|referenceId|Barres de navigation indiquant la hiérarchie des rubriques. Par exemple : « Santé et bien-être/Médecine et soins médicaux/Produits pharmaceutiques ».|
|confidence|Score de confiance dans la plage [0,1]. Un score plus élevé est d’une plus grande confiance.|
|langage|Langue utilisée dans la rubrique.|
|iptcName|Nom de code multimédia IPTC, si détecté.|
|instances |Video Indexer n’indexe actuellement aucune rubrique à intervalles de temps, la vidéo entière est donc utilisée en tant qu’intervalle.|

```json
"topics": [{
    "id": 0,
    "name": "INTERNATIONAL RELATIONS",
    "referenceId": "POLITICS AND GOVERNMENT/FOREIGN POLICY/INTERNATIONAL RELATIONS",
    "referenceType": "VideoIndexer",
    "confidence": 1,
    "language": "en-US",
    "instances": [{
        "adjustedStart": "0:00:00",
        "adjustedEnd": "0:03:36.25",
        "start": "0:00:00",
        "end": "0:03:36.25"
    }]
}, {
    "id": 1,
    "name": "Politics and Government",
    "referenceType": "VideoIndexer",
    "iptcName": "Politics",
    "confidence": 0.9041,
    "language": "en-US",
    "instances": [{
        "adjustedStart": "0:00:00",
        "adjustedEnd": "0:03:36.25",
        "start": "0:00:00",
        "end": "0:03:36.25"
    }]
}]
. . .
```

#### <a name="speakers"></a>speakers

|Nom|Description|
|---|---|
|id|ID de l’intervenant.|
|name|Nom de l’intervenant sous la forme « Speaker # *<number>*  », par exemple : « Speaker #1 ».|
|instances |Liste des intervalles de temps pendant lesquels cet intervenant est apparu.|

```json
"speakers":[
{
  "id":1,
  "name":"Speaker #1",
  "instances":[
     {
    "adjustedStart":"0:00:10.21",
    "adjustedEnd":"0:00:12.81",
    "start":"0:00:10.21",
    "end":"0:00:12.81"
     }
  ]
},
{
  "id":2,
  "name":"Speaker #2",
  "instances":[
     {
    "adjustedStart":"0:00:12.81",
    "adjustedEnd":"0:00:17.03",
    "start":"0:00:12.81",
    "end":"0:00:17.03"
     }
  ]
},
` ` `
```
## <a name="next-steps"></a>Étapes suivantes

[Portail des développeurs Video Indexer](https://api-portal.videoindexer.ai)

Pour plus d’informations sur l’incorporation de widgets dans votre application, voir [Embed Video Indexer widgets into your applications](video-indexer-embed-widgets.md) (Incorporer des widgets Video Indexer à vos applications). 

