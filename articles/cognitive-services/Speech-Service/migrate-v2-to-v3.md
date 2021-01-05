---
title: Migrer de l’API REST v2 à v3 - Service Speech
titleSuffix: Azure Cognitive Services
description: Ce document aide les développeurs à migrer le code de la version v2 à la version V3 de l’API REST de reconnaissance vocale des services Speech.
services: cognitive-services
author: bexxx
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 02/12/2020
ms.author: rbeckers
ms.custom: devx-track-csharp
ms.openlocfilehash: c5bc00ecf5e4c8ae440ce6610e9be8c8f77ed666
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96862205"
---
# <a name="migrate-code-from-v20-to-v30-of-the-rest-api"></a>Migrer le code de la version v2.0 vers la version 3.0 de l’API REST

Par rapport à la version v2, la version v3 de l’API REST de reconnaissance vocale des services Speech est plus fiable, plus facile à utiliser et plus cohérente avec les API associées à des services similaires. La plupart des équipes peuvent effectuer une migration de la version v2 à la version V3 en un ou deux jours.

## <a name="forward-compatibility"></a>Compatibilité ascendante

Toutes les entités de la version v2 peuvent également se trouver dans l’API V3 sous la même identité. Lorsque le schéma d’un résultat a changé (par exemple, des transcriptions), le résultat d’une commande GET dans la version v3 de l’API utilise le schéma v3. Le résultat d’une commande GET dans la version v2 de l’API utilise le même schéma v2. Les entités nouvellement créées sur la version v3 ne sont **pas** disponibles dans les résultats des API v2.

## <a name="breaking-changes"></a>Changements cassants

La liste des changements cassants a été triée selon l’amplitude des modifications requises pour l’adaptation. Seuls quelques changements requièrent des modifications non triviales dans le code appelant. La plupart des changements nécessitent simplement une modification des noms d’élément.

### <a name="host-name-changes"></a>Modifications du nom d’hôte

Les noms d’hôte de point de terminaison `{region}.cris.ai` ont été renommés en `{region}.api.cognitive.microsoft.com`. Les chemins d’accès aux nouveaux points de terminaison ne contiennent plus `api/` qui fait partie du nom d’hôte. Le [document Swagger](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0) répertorie les régions et les chemins d’accès valides.

### <a name="identity-of-an-entity"></a>Identité d’une entité

La propriété `id` est maintenant `self`. Dans la version v2, un utilisateur d’API devait savoir de quelle manière nos chemins d’accès sur l’API sont créés. Cela était non extensible et nécessitait un travail inutile de la part de l’utilisateur. La propriété `id` (UUID) est remplacée par `self` (chaîne), qui est l’emplacement de l’entité (URL). La valeur est toujours unique entre toutes vos entités. Si `id` est stocké sous forme de chaîne dans votre code, un changement de nom suffit pour prendre en charge le nouveau schéma. Vous pouvez maintenant utiliser le contenu `self` comme URL pour les appels REST `GET`, `PATCH` et `DELETE` pour votre entité.

Si l’entité possède des fonctionnalités supplémentaires disponibles via d’autres chemins d’accès, elles sont répertoriées sous `links`. L’exemple suivant de transcription illustre une méthode distincte pour obtenir (`GET`) le contenu de la transcription :

**Transcription v2 :**

```json
{
    "id": "9891c965-bb32-4880-b14b-6d44efb158f3",
    "createdDateTime": "2019-01-07T11:34:12Z",
    "lastActionDateTime": "2019-01-07T11:36:07Z",
    "status": "Succeeded",
    "locale": "en-US",
    "name": "Transcription using locale en-US"
}
```

**Transcription v3 :**

```json
{
    "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions/9891c965-bb32-4880-b14b-6d44efb158f3",
    "createdDateTime": "2019-01-07T11:34:12Z",
    "lastActionDateTime": "2019-01-07T11:36:07Z",
    "status": "Succeeded",
    "locale": "en-US", 
    "displayName": "Transcription using locale en-US",
    "links": {
      "files": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions/9891c965-bb32-4880-b14b-6d44efb158f3/files"
    }
}
```

Selon l’implémentation de votre code, il est possible qu’il ne soit pas suffisant de renommer la propriété. Nous vous recommandons d’utiliser les valeurs `self` et `links` retournées comme URL cibles de vos appels REST, plutôt que de générer des chemins d’accès dans votre client. En utilisant les URL retournées, vous avez la garantie que les modifications ultérieures apportées aux chemins d’accès n’interrompent pas votre code client.

### <a name="working-with-collections-of-entities"></a>Utilisation de collections d’entités

Précédemment, l’API v2 retournait toutes les entités disponibles dans un résultat. Pour permettre un contrôle plus précis de la taille de réponse attendue dans la version v3, tous les résultats de la collection sont paginés. Vous contrôlez le nombre d’entités retournées et le décalage de départ de la page. Ce comportement permet de prédire facilement le runtime du processeur de réponse.

La forme de base de la réponse est la même pour toutes les collections :

```json
{
    "values": [
        {     
        }
    ],
    "@nextLink": "https://{region}.api.cognitive.microsoft.com/speechtotext/v3.0/{collection}?skip=100&top=100"
}
```

La propriété `values` contient un sous-ensemble des entités de collection disponibles. Le nombre et le décalage peuvent être contrôlés à l’aide des paramètres de requête `skip` et `top`. Lorsque `@nextLink` n’est pas `null`, il y a plus de données disponibles et le lot de données suivant peut être récupéré en exécutant une commande GET sur `$.@nextLink`.

Cette modification nécessite l’appel de la commande `GET` pour la collection dans une boucle jusqu’à ce que tous les éléments aient été retournés.

### <a name="creating-transcriptions"></a>Création de transcriptions

Vous trouverez une description détaillée de la façon de créer des lots de transcriptions dans [Guide d’utilisation de la transcription par lots](./batch-transcription.md).

L’API de transcription v3 vous permet de définir explicitement des options de transcription spécifiques. Toutes les propriétés de configuration (facultatives) peuvent maintenant être définies dans la propriété `properties`.
La version v3 prend également en charge plusieurs fichiers d’entrée. Par conséquent, elle nécessite une liste d’URL plutôt qu’une URL unique comme c’était le cas avec la version v2. Le nom de la propriété v2 `recordingsUrl` est à présent `contentUrls` dans la version v3. La fonctionnalité d’analyse des sentiments dans les transcriptions a été supprimée dans la version v3. Consultez l’API [Analyse de texte](https://azure.microsoft.com/en-us/services/cognitive-services/text-analytics/) Microsoft Cognitive Services pour obtenir les options d’analyse de sentiments.

La nouvelle propriété `timeToLive` sous `properties` peut aider à nettoyer les entités terminées existantes. La propriété `timeToLive` spécifie une durée après laquelle une entité terminée est automatiquement supprimée. Affectez-lui une valeur élevée (par exemple `PT12H`) lorsque les entités sont continuellement suivies, consommées et supprimées et, par conséquent, généralement traitées longtemps avant que 12 heures se soient écoulées.

**Corps de la demande POST de la transcription v2 :**

```json
{
  "locale": "en-US",
  "name": "Transcription using locale en-US",
  "recordingsUrl": "https://contoso.com/mystoragelocation",
  "properties": {
    "AddDiarization": "False",
    "AddWordLevelTimestamps": "False",
    "PunctuationMode": "DictatedAndAutomatic",
    "ProfanityFilterMode": "Masked"
  }
}
```

**Corps de la demande POST de la transcription v3 :**

```json
{
  "locale": "en-US",
  "displayName": "Transcription using locale en-US",
  "contentUrls": [
    "https://contoso.com/mystoragelocation",
    "https://contoso.com/myotherstoragelocation"
  ],
  "properties": {
    "diarizationEnabled": false,
    "wordLevelTimestampsEnabled": false,
    "punctuationMode": "DictatedAndAutomatic",
    "profanityFilterMode": "Masked"
  }
}
```

### <a name="format-of-v3-transcription-results"></a>Format des résultats de transcription v3

Le schéma des résultats de transcription a changé légèrement pour s’aligner sur les transcriptions créées par les points de terminaison en temps réel. Vous trouverez une description détaillée du nouveau format dans la page [Guide d’utilisation de la transcription par lots](./batch-transcription.md). Le schéma du résultat est publié dans notre exemple de dépôt [GitHub](https://aka.ms/csspeech/samples) sous `samples/batch/transcriptionresult_v3.schema.json`.

Les noms de propriétés sont maintenant en casse mixte et les valeurs de `channel` et `speaker` utilisent maintenant des types entiers. Le format des durées utilise à présent la structure décrite dans ISO 8601, qui correspond à la mise en forme des durées utilisée dans les autres API Azure.

Exemple d’un résultat de transcription v3. Les différences sont décrites dans les commentaires.

```json
{
  "source": "...",                      // (new in v3) was AudioFileName / AudioFileUrl
  "timestamp": "2020-06-16T09:30:21Z",  // (new in v3)
  "durationInTicks": 41200000,          // (new in v3) was AudioLengthInSeconds
  "duration": "PT4.12S",                // (new in v3)
  "combinedRecognizedPhrases": [        // (new in v3) was CombinedResults
    {
      "channel": 0,                     // (new in v3) was ChannelNumber
      "lexical": "hello world",
      "itn": "hello world",
      "maskedITN": "hello world",
      "display": "Hello world."
    }
  ],
  "recognizedPhrases": [                // (new in v3) was SegmentResults
    {
      "recognitionStatus": "Success",   // 
      "channel": 0,                     // (new in v3) was ChannelNumber
      "offset": "PT0.07S",              // (new in v3) new format, was OffsetInSeconds
      "duration": "PT1.59S",            // (new in v3) new format, was DurationInSeconds
      "offsetInTicks": 700000.0,        // (new in v3) was Offset
      "durationInTicks": 15900000.0,    // (new in v3) was Duration

      // possible transcriptions of the current phrase with confidences
      "nBest": [
        {
          "confidence": 0.898652852,phrase
          "speaker": 1,
          "lexical": "hello world",
          "itn": "hello world",
          "maskedITN": "hello world",
          "display": "Hello world.",

          "words": [
            {
              "word": "hello",
              "offset": "PT0.09S",
              "duration": "PT0.48S",
              "offsetInTicks": 900000.0,
              "durationInTicks": 4800000.0,
              "confidence": 0.987572
            },
            {
              "word": "world",
              "offset": "PT0.59S",
              "duration": "PT0.16S",
              "offsetInTicks": 5900000.0,
              "durationInTicks": 1600000.0,
              "confidence": 0.906032
            }
          ]
        }
      ]
    }
  ]
}
```

### <a name="getting-the-content-of-entities-and-the-results"></a>Obtention du contenu des entités et résultats

Dans la version v2, les liens vers les fichiers d’entrée ou de résultats ont été alignés (inlined) avec le reste des métadonnées d’entité. Une des améliorations apportées dans la version v3 est qu’il existe une séparation claire entre les métadonnées d’entité (retournées par une commande GET sur `$.self`) et les détails et les informations d’identification pour accéder aux fichiers de résultats. Cette séparation permet de protéger les données client et permet un contrôle précis de la durée de validité des informations d’identification.

Dans la version v3, les liens (`links`) incluent une sous-propriété appelée `files` au cas où l’entité expose des données (jeux de données, transcriptions, points de terminaison ou évaluations). Une commande GET sur `$.links.files` retourne une liste de fichiers et une URL SAS pour accéder au contenu de chaque fichier. Pour contrôler la durée de validité des URL SAS, le paramètre de requête `sasValidityInSeconds` peut être utilisé pour spécifier la durée de vie.

**Transcription v2 :**

```json
{
  "id": "9891c965-bb32-4880-b14b-6d44efb158f3",
  "status": "Succeeded",
  "reportFileUrl": "https://contoso.com/report.txt?st=2018-02-09T18%3A07%3A00Z&se=2018-02-10T18%3A07%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=6c044930-3926-4be4-be76-f728327c53b5",
  "resultsUrls": {
    "channel_0": "https://contoso.com/audiofile1.wav?st=2018-02-09T18%3A07%3A00Z&se=2018-02-10T18%3A07%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=6c044930-3926-4be4-be76-f72832e6600c",
    "channel_1": "https://contoso.com/audiofile2.wav?st=2018-02-09T18%3A07%3A00Z&se=2018-02-10T18%3A07%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=3e0163f1-0029-4d4a-988d-3fba7d7c53b5"
  }
}
```

**Transcription v3 :**

```json
{
    "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions/9891c965-bb32-4880-b14b-6d44efb158f3",
    "links": {
      "files": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions/9891c965-bb32-4880-b14b-6d44efb158f3/files"
    } 
}
```

**Une commande GET sur `$.links.files` donne le résultat suivant :**

```json
{
  "values": [
    {
      "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions/9891c965-bb32-4880-b14b-6d44efb158f3/files/f23e54f5-ed74-4c31-9730-2f1a3ef83ce8",
      "name": "Name",
      "kind": "Transcription",
      "properties": {
        "size": 200
      },
      "createdDateTime": "2020-01-13T08:00:00Z",
      "links": {
        "contentUrl": "https://customspeech-usw.blob.core.windows.net/artifacts/mywavefile1.wav.json?st=2018-02-09T18%3A07%3A00Z&se=2018-02-10T18%3A07%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=e05d8d56-9675-448b-820c-4318ae64c8d5"
      }
    },
    {
      "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions/9891c965-bb32-4880-b14b-6d44efb158f3/files/28bc946b-c251-4a86-84f6-ea0f0a2373ef",
      "name": "Name",
      "kind": "TranscriptionReport",
      "properties": {
        "size": 200
      },
      "createdDateTime": "2020-01-13T08:00:00Z",
      "links": {
        "contentUrl": "https://customspeech-usw.blob.core.windows.net/artifacts/report.json?st=2018-02-09T18%3A07%3A00Z&se=2018-02-10T18%3A07%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=e05d8d56-9675-448b-820c-4318ae64c8d5"
      }
    }
  ],
  "@nextLink": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions/9891c965-bb32-4880-b14b-6d44efb158f3/files?skip=2&top=2"
}
```

La propriété `kind` indique le format du contenu du fichier. Pour les transcriptions, les fichiers de type `TranscriptionReport` sont le résumé du travail et les fichiers du type `Transcription` sont le résultat du travail lui-même.

### <a name="customizing-models"></a>Personnalisation des modèles

Avant la version v3, il y avait une distinction entre un _modèle acoustique_ et un _modèle de langage_ lors de l’entraînement d’un modèle. Cette distinction entraînait la nécessité de spécifier plusieurs modèles lors de la création de points de terminaison ou de transcriptions. Pour simplifier ce processus pour un appelant, nous avons supprimé les différences et tout a été rendu dépendant du contenu des jeux de données utilisés pour l’entraînement du modèle. Avec cette modification, la création de modèle prend à présent en charge des jeux de données mixtes (données de langue et données acoustiques). Les points de terminaison et les transcriptions requièrent à présent un seul modèle.

Avec cette modification, la nécessité d’un élément `kind` dans l’opération `POST` a été supprimée et le tableau `datasets[]` peut maintenant contenir plusieurs jeux de données de même type ou de types mixtes.

Pour améliorer les résultats d’un modèle entraîné, les données acoustiques sont automatiquement utilisées en interne pendant l’apprentissage de la langue. En général, les modèles créés via l’API v3 fournissent des résultats plus précis que les modèles créés avec l’API v2.

### <a name="retrieving-base-and-custom-models"></a>Récupération des modèles de base et personnalisés

Pour simplifier l’obtention des modèles disponibles, la version v3 a séparé les collections de « modèles de base » des « modèles personnalisés » détenus par le client. Les deux routes sont à présent `GET /speechtotext/v3.0/models/base` et `GET /speechtotext/v3.0/models/`.

Dans la version v2, tous les modèles étaient retournés ensemble dans une réponse unique.

### <a name="name-of-an-entity"></a>Nom d’une entité

La propriété `name` est maintenant `displayName`. Cela est cohérent avec d’autres API Azure de ne pas indiquer les propriétés d’identité. La valeur de cette propriété ne doit pas être unique et peut être modifiée après la création de l’entité à l’aide d’une opération `PATCH`.

**Transcription v2 :**

```json
{
    "name": "Transcription using locale en-US"
}
```

**Transcription v3 :**

```json
{
    "displayName": "Transcription using locale en-US"
}
```

### <a name="accessing-referenced-entities"></a>Accès aux entités référencées

Dans la version v2, les entités référencées étaient toujours inlined, par exemple les modèles utilisés d’un point de terminaison. L’imbrication des entités entraînait des réponses importantes et les consommateurs consommaient rarement le contenu imbriqué. Pour réduire la taille de la réponse et améliorer les performances, les entités référencées ne sont plus incluses dans la réponse. Au lieu de cela, une référence à l’autre entité apparaît et peut être utilisée directement pour une autre commande `GET` (il s’agit également d’une URL), suivant le même modèle que le lien `self`.

**Transcription v2 :**

```json
{
  "id": "9891c965-bb32-4880-b14b-6d44efb158f3",
  "models": [
    {
      "id": "827712a5-f942-4997-91c3-7c6cde35600b",
      "modelKind": "Language",
      "lastActionDateTime": "2019-01-07T11:36:07Z",
      "status": "Running",
      "createdDateTime": "2019-01-07T11:34:12Z",
      "locale": "en-US",
      "name": "Acoustic model",
      "description": "Example for an acoustic model",
      "datasets": [
        {
          "id": "702d913a-8ba6-4f66-ad5c-897400b081fb",
          "dataImportKind": "Language",
          "lastActionDateTime": "2019-01-07T11:36:07Z",
          "status": "Succeeded",
          "createdDateTime": "2019-01-07T11:34:12Z",
          "locale": "en-US",
          "name": "Language dataset",
        }
      ]
    },
  ]
}
```

**Transcription v3 :**

```json
{
  "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions/9891c965-bb32-4880-b14b-6d44efb158f3",
  "model": {
    "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/models/021a72d0-54c4-43d3-8254-27336ead9037"
  }
}
```

Si vous avez besoin de consommer les détails d’un modèle référencé comme indiqué dans l’exemple ci-dessus, émettez simplement une commande GET sur `$.model.self`.

### <a name="retrieving-endpoint-logs"></a>Récupération des journaux des points de terminaison

La version v2 du service prenait en charge la journalisation des résultats du point de terminaison. Pour récupérer les résultats d’un point de terminaison avec la version v2, vous deviez créer une « exportation de données », qui représente un instantané des résultats définis par un intervalle de temps. Le processus d’exportation des lots de données était rigide. L’API V3 donne accès à chaque fichier individuel et permet d’effectuer des itérations.

**Un point de terminaison v3 a été exécuté avec succès :**

```json
{
  "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/endpoints/afa0669c-a01e-4693-ae3a-93baf40f26d6",
  "links": {
    "logs": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/endpoints/afa0669c-a01e-4693-ae3a-93baf40f26d6/files/logs" 
  }
}
```

**Réponse de la commande GET `$.links.logs` :**

```json
{
  "values": [
    {
      "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/endpoints/6d72ad7e-f286-4a6f-b81b-a0532ca6bcaa/files/logs/2019-09-20_080000_3b5f4628-e225-439d-bd27-8804f9eed13f.wav",
      "name": "2019-09-20_080000_3b5f4628-e225-439d-bd27-8804f9eed13f.wav",
      "kind": "Audio",
      "properties": {
        "size": 12345
      },
      "createdDateTime": "2020-01-13T08:00:00Z",
      "links": {
        "contentUrl": "https://customspeech-usw.blob.core.windows.net/artifacts/2019-09-20_080000_3b5f4628-e225-439d-bd27-8804f9eed13f.wav?st=2018-02-09T18%3A07%3A00Z&se=2018-02-10T18%3A07%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=e05d8d56-9675-448b-820c-4318ae64c8d5"
      }
    }    
  ],
  "@nextLink": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/endpoints/afa0669c-a01e-4693-ae3a-93baf40f26d6/files/logs?top=2&SkipToken=2!188!MDAwMDk1ITZhMjhiMDllLTg0MDYtNDViMi1hMGRkLWFlNzRlOGRhZWJkNi8yMDIwLTA0LTAxLzEyNDY0M182MzI5NGRkMi1mZGYzLTRhZmEtOTA0NC1mODU5ZTcxOWJiYzYud2F2ITAwMDAyOCE5OTk5LTEyLTMxVDIzOjU5OjU5Ljk5OTk5OTlaIQ--"
}
```

La pagination des journaux de point de terminaison fonctionne de façon similaire à toutes les autres collections, à ceci près qu’aucun décalage ne peut être spécifié. En raison de la grande quantité de données disponibles, la pagination est déterminée par le serveur.

Dans la version v3, chaque journal de point de terminaison peut être supprimé individuellement en émettant une opération `DELETE` sur l’élément `self` d’un fichier, ou à l’aide de `DELETE` sur `$.links.logs`. Pour spécifier une date de fin, le paramètre de requête `endDate` peut être ajouté à la demande.

### <a name="using-custom-properties"></a>Utilisation de propriétés personnalisées

Pour séparer les propriétés personnalisées des propriétés de configuration facultatives, toutes les propriétés explicitement nommées se trouvent maintenant dans la propriété `properties` et toutes les propriétés définies par les appelants se trouvent maintenant dans la propriété `customProperties`.

**Entité de transcription v2 :**

```json
{
  "properties": {
    "customerDefinedKey": "value",
    "diarizationEnabled": "False",
    "wordLevelTimestampsEnabled": "False"
  }
}
```

**Entité de transcription v3 :**

```json
{
  "properties": {
    "diarizationEnabled": false,
    "wordLevelTimestampsEnabled": false 
  },
  "customProperties": {
    "customerDefinedKey": "value"
  }
}
```

Cette modification vous permet également d’utiliser des types corrects sur toutes les propriétés explicitement nommées sous `properties` (par exemple valeur booléenne au lieu de chaîne).

### <a name="response-headers"></a>En-têtes de réponse

La version v3 ne retourne plus l’en-tête `Operation-Location` en plus de l’en-tête `Location` sur les demandes `POST`. La valeur des deux en-têtes dans la version v2 était la même. À présent, seul `Location` est retourné.

Étant donné que la nouvelle version de l’API est maintenant gérée par la gestion des API Azure (APIM), les en-têtes de limitation `X-RateLimit-Limit`, `X-RateLimit-Remaining` et `X-RateLimit-Reset` ne sont pas contenus dans les en-têtes de réponse.

### <a name="accuracy-tests"></a>Tests de précision

Les tests de précision ont été renommés en évaluations, car le nouveau nom décrit mieux ce qu’ils représentent. Voici les nouveaux chemins d’accès : `https://{region}.api.cognitive.microsoft.com/speechtotext/v3.0/evaluations`.

## <a name="next-steps"></a>Étapes suivantes

Examinez toutes les fonctionnalités de ces API REST couramment utilisées fournies par les services Speech :

* [API REST de reconnaissance vocale](rest-speech-to-text.md)
* [Document Swagger](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0) pour la version v3 de l’API REST
* Pour obtenir un exemple de code afin d’effectuer des transcriptions par lots, consultez l’exemple de dépôt [GitHub](https://aka.ms/csspeech/samples) dans le sous-répertoire `samples/batch`.
