---
title: Informations de référence sur l’API de reconnaissance vocale (REST) - Service Speech
titleSuffix: Azure Cognitive Services
description: Découvrez comment utiliser l’API REST de reconnaissance vocale. Cet article vous présente les options d’autorisation, les options de requête, et vous explique comment structurer une demande et recevoir une réponse.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 12/10/2020
ms.author: trbye
ms.custom: devx-track-csharp
ms.openlocfilehash: c746666d58e21c2705a2ef1d6a17d0d1196f7590
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97504472"
---
# <a name="speech-to-text-rest-api"></a>API REST de reconnaissance vocale

La reconnaissance vocale utilise deux API REST distinctes. Chaque API remplit son objectif particulier et utilise différents ensembles de points de terminaison.

Les API REST de reconnaissance vocale sont les suivantes :
- L’[API REST de reconnaissance vocale v3.0](#speech-to-text-rest-api-v30) est utilisée pour la [Transcription par lot](batch-transcription.md) et [Custom Speech](custom-speech-overview.md). La version v3.0 [succède à la version v2.0](/azure/cognitive-services/speech-service/migrate-v2-to-v3).
- [L’API REST de reconnaissance vocale pour audio court](#speech-to-text-rest-api-for-short-audio) est utilisée pour la transcription en ligne. Elle constitue une alternative au [Kit de développement logiciel (SDK) Speech](speech-sdk.md). Les demandes qui utilisent cette API peuvent transmettre au maximum 60 secondes d’audio chacune. 

## <a name="speech-to-text-rest-api-v30"></a>API REST de reconnaissance vocale v3.0

L’API REST de reconnaissance vocale v3.0 est utilisée pour la [Transcription par lot](batch-transcription.md) et [Custom Speech](custom-speech-overview.md). Si vous avez besoin de communiquer avec une transcription en ligne via REST, utilisez l’[API REST de reconnaissance vocale pour audio court](#speech-to-text-rest-api-for-short-audio).

Utilisez l’API REST v3.0 pour effectuer les tâches suivantes :
- Copier des modèles vers d’autres abonnements si vous souhaitez que les collègues aient accès à un modèle que vous avez créé, ou si vous souhaitez déployer un modèle dans plusieurs régions.
- Transcrire les données d’un conteneur (transcription en bloc) et fournir plusieurs URL de fichiers audio.
- Charger des données à partir de comptes de stockage Azure à l’aide d’un Uri de SAP.
- Obtenir des journaux par point de terminaison si les journaux ont été demandés pour celui-ci.
- Demander le manifeste des modèles que vous créez, afin de configurer des conteneurs locaux.

L’API REST v3.0 comprend des fonctionnalités telles que les suivantes :
- **Notifications de Webhook** : tous les processus en cours d’exécution du service prennent désormais en charge les notifications de Webhook. L’API REST v3.0 fournit les appels pour vous permettre d’inscrire vos webhooks à l’endroit où les notifications sont envoyées.
- **Mise à jour de modèles derrière des points de terminaison** 
- **Adaptation de modèle avec plusieurs jeux de données** : adaptez un modèle à l’aide de plusieurs combinaisons de jeux de données acoustiques, linguistiques et de prononciation.
- **Apportez votre propre de stockage** : utilisez vos propres comptes de stockage pour les journaux, les fichiers de transcription et les autres données.

Pour obtenir des exemples d’utilisation de l’API REST v3.0 avec la transcription par lot, consultez [cet article](batch-transcription.md).

Si vous utilisez l’API REST de reconnaissance vocale v2.0, découvrez comment migrer vers la version v3.0 dans [ce guide](/azure/cognitive-services/speech-service/migrate-v2-to-v3).

Consultez les informations de référence complètes sur l’API REST de reconnaissance vocale v3.0 [ici](https://centralus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0).

## <a name="speech-to-text-rest-api-for-short-audio"></a>API REST de reconnaissance vocale pour audio court

En guise d’alternative au [Kit de développement logiciel (SDK)](speech-sdk.md), le service Speech vous permet de convertir la parole en texte à l’aide d’une API REST. Chaque point de terminaison accessible est associé à une région. Votre application nécessite une clé d’abonnement pour le point de terminaison que vous prévoyez d’utiliser. L’API REST pour audio court étant très limitée, elle ne doit être utilisée que dans les cas où le [Kit de développement logiciel (SDK) Speech](speech-sdk.md) ne peut pas l’être.

Avant d’utiliser l’API REST de reconnaissance vocale pour audio court, prenez en compte les aspects suivants :

* Les demandes qui utilisent l’API REST pour audio court et transmettent directement l’audio ne peuvent pas contenir plus de 60 secondes d’audio.
* L’API REST de reconnaissance vocale pour audio court retourne uniquement des résultats finaux. Les résultats partiels ne sont pas fournis.

Si vous devez envoyer un contenu audio plus long pour votre application, vous pouvez utiliser le [Kit de développement logiciel (SDK) Speech](speech-sdk.md) ou l’[API REST de reconnaissance vocale v3.0](#speech-to-text-rest-api-v30).

> [!TIP]
> Consultez la [documentation](../../azure-government/compare-azure-government-global-azure.md) Azure Government pour les points de terminaison Government Cloud (FairFax).

[!INCLUDE [](../../../includes/cognitive-services-speech-service-rest-auth.md)]

### <a name="regions-and-endpoints"></a>Régions et points de terminaison

Le point de terminaison de l’API REST pour audio court a le format suivant :

```
https://<REGION_IDENTIFIER>.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1
```

Remplacez `<REGION_IDENTIFIER>` par l'identificateur correspondant à la région de votre abonnement dans le tableau suivant :

[!INCLUDE [](../../../includes/cognitive-services-speech-service-region-identifier.md)]

> [!NOTE]
> Le paramètre de langue doit être ajouté à l'URL pour éviter de recevoir une erreur HTTP 4xx. Par exemple, la langue définie sur la valeur Anglais (États-Unis) à l’aide du point de terminaison USA Ouest est : `https://westus.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1?language=en-US`.

### <a name="query-parameters"></a>Paramètres de requête

Ces paramètres peuvent être inclus dans la chaîne de la requête REST.

| Paramètre | Description | Obligatoire/facultatif |
|-----------|-------------|---------------------|
| `language` | Identifie la langue parlée qui est reconnue. Voir [Langues prises en charge](language-support.md#speech-to-text). | Obligatoire |
| `format` | Spécifie le format du résultat. Les valeurs acceptées sont `simple` et `detailed`. Les résultats simples incluent `RecognitionStatus`, `DisplayText`, `Offset` et `Duration`. Les réponses détaillées incluent quatre représentations différentes du texte affiché. La valeur par défaut est `simple`. | Facultatif |
| `profanity` | Spécifie comment traiter la vulgarité dans les résultats de la reconnaissance. Les valeurs acceptées sont `masked`, qui remplace les vulgarités par des astérisques, `removed`, qui supprime les vulgarités du résultat, ou `raw`, qui inclut les vulgarités dans le résultat. La valeur par défaut est `masked`. | Facultatif |
| `cid` | Lorsque vous utilisez le [portail Custom Speech](./custom-speech-overview.md) pour créer des modèles personnalisés, vous pouvez utiliser des modèles personnalisés à l’aide de leur **ID de point de terminaison** figurant sur la page **Déploiement**. Utilisez l’**ID de point de terminaison** comme argument pour le paramètre de chaîne de requête `cid`. | Facultatif |

### <a name="request-headers"></a>En-têtes de requête

Ce tableau répertorie les en-têtes obligatoires et facultatifs pour les demandes de reconnaissance vocale.

|En-tête| Description | Obligatoire/facultatif |
|------|-------------|---------------------|
| `Ocp-Apim-Subscription-Key` | Votre clé d’abonnement du service Speech. | Cet en-tête ou `Authorization` est requis. |
| `Authorization` | Un jeton d’autorisation précédé du mot `Bearer`. Pour en savoir plus, consultez [Authentification](#authentication). | Cet en-tête ou `Ocp-Apim-Subscription-Key` est requis. |
| `Pronunciation-Assessment` | Spécifie les paramètres pour montrer les scores de prononciation dans les résultats de la reconnaissance, qui évaluent la qualité de la prononciation des entrées vocales, avec des indicateurs de justesse, d’aisance, d’exhaustivité, etc. Ce paramètre est un format JSON encodé en base64 contenant plusieurs paramètres détaillés. Pour plus d’informations sur la création de cet en-tête, consultez [Paramètres d’évaluation de la prononciation](#pronunciation-assessment-parameters). | Facultatif |
| `Content-type` | Décrit le format et le codec des données audio fournies. Les valeurs acceptées sont `audio/wav; codecs=audio/pcm; samplerate=16000` et `audio/ogg; codecs=opus`. | Obligatoire |
| `Transfer-Encoding` | Spécifie que les données audio sont envoyées en bloc plutôt que dans un seul fichier. Utilisez uniquement cet en-tête si vous envoyez les données audio en bloc. | Facultatif |
| `Expect` | Si vous utilisez le transfert en bloc, envoyez `Expect: 100-continue`. Le service Speech accuse réception de la requête initiale et attend des données supplémentaires.| Requis si vous envoyez les données audio en bloc. |
| `Accept` | Si cette valeur est fournie, elle doit être `application/json`. Le service Speech fournit les résultats au format JSON. Certains frameworks de demande fournissent une valeur par défaut incompatible. Il est recommandé de toujours inclure `Accept`. | Cette étape est facultative mais recommandée. |

### <a name="audio-formats"></a>Formats audio

L’audio est envoyé dans le corps de la requête HTTP `POST`. Il doit être dans l’un des formats de ce tableau :

| Format | Codec | Vitesse de transmission | Échantillonnage  |
|--------|-------|----------|--------------|
| WAV    | PCM   | 256 Kbits/s | 16 kHz, mono |
| OGG    | OPUS  | 256 Kbits/s | 16 kHz, mono |

>[!NOTE]
>Les formats ci-dessus sont pris en charge via l’API REST pour audio court et WebSocket dans le service Speech. Pour le moment, le [kit de développement logiciel (SDK) Speech](speech-sdk.md) prend en charge le format WAV avec codec PCM, ainsi que d'[autres formats](how-to-use-codec-compressed-audio-input-streams.md).

### <a name="pronunciation-assessment-parameters"></a>Paramètres d’évaluation de la prononciation

Ce tableau liste les paramètres obligatoires et facultatifs pour l’évaluation de la prononciation.

| Paramètre | Description | Requis ? |
|-----------|-------------|---------------------|
| ReferenceText | Texte duquel la prononciation est évaluée. | Obligatoire |
| GradingSystem | Système de points pour la calibration du score. Le système `FivePoint` donne un score à virgule flottante de 0 à 5 et `HundredMark` donne un score à virgule flottante de 0 à 100. Par défaut : `FivePoint`. | Facultatif |
| Granularité | Granularité de l’évaluation. Les valeurs acceptées sont `Phoneme`, qui affiche le score au niveau du texte intégral, du mot et du phonème, `Word`, qui affiche le score au niveau du texte intégral et du mot, `FullText`, qui affiche le score au niveau du texte intégral uniquement. La valeur par défaut est `Phoneme`. | Facultatif |
| Dimension | Définit les critères de sortie. Les valeurs acceptées sont `Basic`, qui affiche uniquement le score de justesse, `Comprehensive` affiche des scores sur davantage de dimensions (par exemple, le score d’aisance et le score d’exhaustivité au niveau du texte intégral, du type d’erreur et du mot). Consultez les [paramètres de réponse](#response-parameters) pour voir les définitions des différentes dimensions de score et des types d’erreur de mot. La valeur par défaut est `Basic`. | Facultatif |
| EnableMiscue | Active le calcul de faute de langue. Quand cette option est activée, les mots prononcés sont comparés au texte de référence et sont marqués comme omission/insertion en fonction de la comparaison. Les valeurs acceptées sont `False` et `True`. La valeur par défaut est `False`. | Facultatif |
| ScenarioId | GUID indiquant un système de points personnalisé. | Facultatif |

Voici un exemple de code JSON contenant les paramètres d’évaluation de la prononciation :

```json
{
  "ReferenceText": "Good morning.",
  "GradingSystem": "HundredMark",
  "Granularity": "FullText",
  "Dimension": "Comprehensive"
}
```

L’exemple de code suivant montre comment créer les paramètres d’évaluation de la prononciation dans l’en-tête `Pronunciation-Assessment` :

```csharp
var pronAssessmentParamsJson = $"{{\"ReferenceText\":\"Good morning.\",\"GradingSystem\":\"HundredMark\",\"Granularity\":\"FullText\",\"Dimension\":\"Comprehensive\"}}";
var pronAssessmentParamsBytes = Encoding.UTF8.GetBytes(pronAssessmentParamsJson);
var pronAssessmentHeader = Convert.ToBase64String(pronAssessmentParamsBytes);
```

Nous recommandons vivement un chargement par streaming (transfert en bloc) lors de la publication des données audio, ce qui peut réduire considérablement la latence. Consultez [exemple de code dans différents langages de programmation](https://github.com/Azure-Samples/Cognitive-Speech-TTS/tree/master/PronunciationAssessment) pour voir comment activer le streaming.

>[!NOTE]
>La fonctionnalité d’évaluation de la prononciation est actuellement disponible uniquement dans les régions `westus`, `eastasia` et `centralindia`. Par ailleurs, cette fonctionnalité n’est pour le moment disponible que dans la langue `en-US`.

### <a name="sample-request"></a>Exemple de requête

L’exemple ci-dessous inclut le nom d’hôte et les en-têtes requis. Il est important de noter que le service attend également des données audio, ce qui n’est pas inclus dans cet exemple. Comme mentionné précédemment, l’envoi en bloc est recommandé, mais pas nécessaire.

```HTTP
POST speech/recognition/conversation/cognitiveservices/v1?language=en-US&format=detailed HTTP/1.1
Accept: application/json;text/xml
Content-Type: audio/wav; codecs=audio/pcm; samplerate=16000
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
Host: westus.stt.speech.microsoft.com
Transfer-Encoding: chunked
Expect: 100-continue
```

Pour activer l’évaluation de la prononciation, vous pouvez ajouter l’en-tête ci-dessous. Pour plus d’informations sur la création de cet en-tête, consultez [Paramètres d’évaluation de la prononciation](#pronunciation-assessment-parameters).

```HTTP
Pronunciation-Assessment: eyJSZWZlcm...
```

### <a name="http-status-codes"></a>Codes d’état HTTP

Le code d’état HTTP de chaque réponse indique la réussite ou des erreurs courantes.

| Code d'état HTTP | Description | Raison possible |
|------------------|-------------|-----------------|
| `100` | Continue | La requête initiale a été acceptée. Passez à l’envoi du reste des données. (Utilisé avec le transfert en bloc) |
| `200` | OK | La requête a réussi ; le corps de réponse est un objet JSON. |
| `400` | Demande incorrecte | Le code de langue n’est pas fourni, la langue n’est pas prise en charge, le fichier audio est non valide, etc. |
| `401` | Non autorisé | La clé d’abonnement ou le jeton d’autorisation n’est pas valide dans la région spécifiée, ou le point de terminaison n’est pas valide. |
| `403` | Interdit | Clé d’abonnement ou jeton d’autorisation manquant. |

### <a name="chunked-transfer"></a>Transfert en bloc

Le transfert en bloc (`Transfer-Encoding: chunked`) peut aider à réduire la latence de la reconnaissance. Il permet au service Speech de commencer à traiter le fichier audio pendant sa transmission. L’API REST pour audio court ne fournit pas de résultats partiels ou intermédiaires.

Cet exemple de code montre comment envoyer l’audio en bloc. Seul le premier segment doit contenir l’en-tête du fichier audio. `request` est un objet `HttpWebRequest` connecté au point de terminaison REST approprié. `audioFile` est le chemin vers un fichier audio sur disque.

```csharp
var request = (HttpWebRequest)HttpWebRequest.Create(requestUri);
request.SendChunked = true;
request.Accept = @"application/json;text/xml";
request.Method = "POST";
request.ProtocolVersion = HttpVersion.Version11;
request.Host = host;
request.ContentType = @"audio/wav; codecs=audio/pcm; samplerate=16000";
request.Headers["Ocp-Apim-Subscription-Key"] = "YOUR_SUBSCRIPTION_KEY";
request.AllowWriteStreamBuffering = false;

using (var fs = new FileStream(audioFile, FileMode.Open, FileAccess.Read))
{
    // Open a request stream and write 1024 byte chunks in the stream one at a time.
    byte[] buffer = null;
    int bytesRead = 0;
    using (var requestStream = request.GetRequestStream())
    {
        // Read 1024 raw bytes from the input audio file.
        buffer = new Byte[checked((uint)Math.Min(1024, (int)fs.Length))];
        while ((bytesRead = fs.Read(buffer, 0, buffer.Length)) != 0)
        {
            requestStream.Write(buffer, 0, bytesRead);
        }

        requestStream.Flush();
    }
}
```

### <a name="response-parameters"></a>Paramètres de réponse

Les résultats sont fournis au format JSON. Le format `simple` inclut ces champs de niveau supérieur.

| Paramètre | Description  |
|-----------|--------------|
|`RecognitionStatus`|État, tel que `Success` pour une reconnaissance ayant réussi. Voir le tableau suivant.|
|`DisplayText`|Le texte reconnu après mise en majuscules, ponctuation, normalisation du texte inversée (conversion de texte lu en formes plus courtes, par exemple 200 pour « deux cents » ou « Dr. Smith » pour « docteur smith ») et masquage des grossièretés. Présent uniquement en cas de réussite.|
|`Offset`|Moment (en unités de 100 nanosecondes) à partir duquel la voix identifiée commence dans le flux audio.|
|`Duration`|Durée (en unités de 100 nanosecondes) de la voix reconnue dans le flux audio.|

Le champ `RecognitionStatus` peut contenir ces valeurs :

| Statut | Description |
|--------|-------------|
| `Success` | La reconnaissance a réussi et le champ `DisplayText` est présent. |
| `NoMatch` | Des paroles ont été détectées dans le flux audio, mais aucun mot de la langue cible n’a été mis en correspondance. Signifie généralement que la langue de reconnaissance est différente de celle parlée par l’orateur. |
| `InitialSilenceTimeout` | Le début du flux audio contenait uniquement un silence, et le service a dépassé son délai d’attente de paroles. |
| `BabbleTimeout` | Le début du flux audio contenait uniquement du bruit, et le service a dépassé son délai d’attente de paroles. |
| `Error` | Le service de reconnaissance a rencontré une erreur interne et n’a pas pu continuer. Réessayez si possible. |

> [!NOTE]
> Si l’audio est composé uniquement de grossièretés et que le paramètre de requête `profanity` a la valeur `remove`, le service ne retourne pas de résultat de reconnaissance vocale.

Le format `detailed` comprend des formes supplémentaires de résultats reconnus.
Lorsque vous utilisez le format `detailed`, `DisplayText` est fourni en tant que `Display` pour chaque résultat dans la liste `NBest`.

L’objet dans la liste `NBest` peut inclure :

| Paramètre | Description |
|-----------|-------------|
| `Confidence` | Le score de confiance de l’entrée, compris entre 0,0 (aucun niveau de confiance) et 1,0 (confiance totale) |
| `Lexical` | La forme lexicale du texte reconnu : les mots reconnus. |
| `ITN` | La forme « normalisation du texte inversée » (canonique) du texte reconnu, avec numéros de téléphone, chiffres, abréviations (« docteur smith » en « dr smith ») et autres transformations appliquées. |
| `MaskedITN` | La forme « normalisation du texte inversée » avec masquage des grossièretés appliqué, si nécessaire. |
| `Display` | La forme d’affichage du texte reconnu, avec signes de ponctuation et mise en majuscules ajoutés. Ce paramètre est identique à la valeur `DisplayText` fournie lorsque le format est défini sur `simple`. |
| `AccuracyScore` | Précision de prononciation du discours. La précision indique dans quelle mesure les phonèmes correspondent à la prononciation d’un intervenant de langue maternelle. Le score de précision au niveau du mot et du texte intégral est agrégé à partir du score de précision du niveau du phonème. |
| `FluencyScore` | Fluidité du discours concerné. La fluidité indique dans quelle mesure le discours correspond à l’utilisation qu’un orateur de langue maternelle fait des pauses entre les mots. |
| `CompletenessScore` | Intégralité du discours, déterminée par le calcul du rapport entre les mots prononcés et l’entrée de texte de référence. |
| `PronScore` | Score global indiquant la qualité de prononciation du discours concerné. Il est agrégé à partir de `AccuracyScore`, `FluencyScore` et `CompletenessScore` avec une pondération. |
| `ErrorType` | Cette valeur indique si un mot est omis, inséré ou mal prononcé par rapport à `ReferenceText`. Les valeurs possibles sont `None` (aucune erreur sur ce mot), `Omission`, `Insertion` et `Mispronunciation`. |

### <a name="sample-responses"></a>Exemples de réponses

Réponse classique pour la reconnaissance `simple` :

```json
{
  "RecognitionStatus": "Success",
  "DisplayText": "Remind me to buy 5 pencils.",
  "Offset": "1236645672289",
  "Duration": "1236645672289"
}
```

Réponse classique pour la reconnaissance `detailed` :

```json
{
  "RecognitionStatus": "Success",
  "Offset": "1236645672289",
  "Duration": "1236645672289",
  "NBest": [
      {
        "Confidence" : "0.87",
        "Lexical" : "remind me to buy five pencils",
        "ITN" : "remind me to buy 5 pencils",
        "MaskedITN" : "remind me to buy 5 pencils",
        "Display" : "Remind me to buy 5 pencils.",
      }
  ]
}
```

Réponse classique pour la reconnaissance avec évaluation de la prononciation :

```json
{
  "RecognitionStatus": "Success",
  "Offset": "400000",
  "Duration": "11000000",
  "NBest": [
      {
        "Confidence" : "0.87",
        "Lexical" : "good morning",
        "ITN" : "good morning",
        "MaskedITN" : "good morning",
        "Display" : "Good morning.",
        "PronScore" : 84.4,
        "AccuracyScore" : 100.0,
        "FluencyScore" : 74.0,
        "CompletenessScore" : 100.0,
        "Words": [
            {
              "Word" : "Good",
              "AccuracyScore" : 100.0,
              "ErrorType" : "None",
              "Offset" : 500000,
              "Duration" : 2700000
            },
            {
              "Word" : "morning",
              "AccuracyScore" : 100.0,
              "ErrorType" : "None",
              "Offset" : 5300000,
              "Duration" : 900000
            }
        ]
      }
  ]
}
```

## <a name="next-steps"></a>Étapes suivantes

- [Créez un compte Azure gratuit](https://azure.microsoft.com/free/cognitive-services/)
- [Personnaliser les modèles acoustiques](./how-to-custom-speech-train-model.md)
- [Personnaliser les modèles de langage](./how-to-custom-speech-train-model.md)
- [Familiarisez-vous avec la transcription par lot](batch-transcription.md)