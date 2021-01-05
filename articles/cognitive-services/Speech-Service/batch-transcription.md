---
title: Guide pratique sur l’utilisation de la transcription par lots – Service Speech
titleSuffix: Azure Cognitive Services
description: La transcription Batch est idéale pour transcrire une grande quantité de données audio stockées dans des objets blob Azure, par exemple. Avec l’API REST dédiée, vous pouvez pointer vers des fichiers audio à l’aide d’un URI de signature d’accès partagé (SAS) et recevoir les transcriptions de manière asynchrone.
services: cognitive-services
author: wolfma61
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 11/03/2020
ms.author: wolfma
ms.custom: devx-track-csharp
ms.openlocfilehash: 426cf78f6b87acf1d8c7551b0b0a6172a30167b1
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96621093"
---
# <a name="how-to-use-batch-transcription"></a>Guide d’utilisation de la transcription par lots

La transcription par lots est un ensemble d’opérations d’API REST qui vous permet de transcrire une grande quantité de données audio dans un stockage. Vous pouvez pointer vers des fichiers audio à l’aide d’un URI type ou d’un URI de signature d’accès partagé (SAP) et recevoir les résultats de la transcription de manière asynchrone. L’API 3.0 vous permet de transcrire un ou plusieurs fichiers audio, ou de traiter un conteneur de stockage entier.

Vous pouvez utiliser des API REST de transcription par lots pour appeler les méthodes suivantes :

|    Opération de transcription par lots                                             |    Méthode    |    Appel d’API REST                                   |
|------------------------------------------------------------------------------|--------------|----------------------------------------------------|
|    Crée une transcription.                                              |    POST      |    speechtotext/v3.0/transcriptions            |
|    Récupère une liste de transcriptions pour l’abonnement authentifié.    |    GET       |    speechtotext/v3.0/transcriptions            |
|    Obtient la liste des paramètres régionaux pris en charge pour les transcriptions hors connexion.              |    GET       |    speechtotext/v3.0/transcriptions/locales    |
|    Met à jour les détails mutables de la transcription identifiée par son ID.    |    PATCH     |    speechtotext/v3.0/transcriptions/{id}       |
|    Supprime la tâche de transcription spécifiée.                                 |    Suppression    |    speechtotext/v3.0/transcriptions/{id}       |
|    Obtient la transcription identifiée par l’ID donné.                        |    GET       |    speechtotext/v3.0/transcriptions/{id}       |
|    Permet d'obtenir les fichiers de résultats de la transcription identifiés par l'ID donné.    |    GET       |    speechtotext/v3.0/transcriptions/{id}/files |

Vous pouvez examiner et tester l’API détaillée, disponible sous forme de [document Swagger](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0).

Les travaux de transcription par lots sont planifiés en faisant au mieux selon les circonstances.
Vous ne pouvez pas estimer le moment où un travail passe à l’état d’exécution, mais cela doit intervenir sous quelques minutes en présence d’une charge système normale. Une fois en cours d’exécution, la transcription est plus rapide que la vitesse de lecture du runtime.

## <a name="prerequisites"></a>Prérequis

Comme pour toutes les fonctionnalités du service Speech, créez une clé d’abonnement à partir du [Portail Azure](https://portal.azure.com) en suivant les instructions du [guide de démarrage rapide](overview.md#try-the-speech-service-for-free).

>[!NOTE]
> Pour pouvoir utiliser la transcription Batch, vous avez besoin d’un abonnement standard (S0) pour le service Speech. Les clés d’abonnement gratuit (F0) ne fonctionnent pas. Pour plus d’informations, voir [Tarification et limites](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

Si vous envisagez de personnaliser des modèles, suivez les étapes décrites dans les sections [Personnalisation acoustique](./how-to-custom-speech-train-model.md) et [Personnalisation de la langue](./how-to-custom-speech-train-model.md). Pour utiliser les modèles créés dans la transcription de lot, vous avez besoin de l’emplacement du modèle. L’emplacement d’un modèle figure dans les détails associés (propriété `self`). Il n’est *pas nécessaire* de déployer un point de terminaison personnalisé pour le service de transcription par lot.

>[!NOTE]
> Dans le cadre de l’API REST, la transcription Batch comporte un ensemble de [quotas et limites](speech-services-quotas-and-limits.md#batch-transcription), que nous vous encourageons à examiner. Pour tirer pleinement parti de la fonctionnalité de transcription Batch afin de transcrire efficacement un grand nombre de fichiers audio, nous vous recommandons d’envoyer toujours plusieurs fichiers par requête ou de pointer vers un conteneur Stockage Blob avec les fichiers audio à transcrire. Le service va transcrire les fichiers en réduisant par la même occasion le temps de bouclage. L’utilisation de plusieurs fichiers dans une requête unique est très simple et directe : consultez la section [Configuration](#configuration). 

## <a name="batch-transcription-api"></a>API de transcription Batch

L’API de transcription Batch prend en charge les formats suivants :

| Format | Codec | Bits par échantillon | Échantillonnage             |
|--------|-------|---------|---------------------------------|
| WAV    | PCM   | 16 bits  | 8 kHz ou 16 kHz, mono ou stéréo |
| MP3    | PCM   | 16 bits  | 8 kHz ou 16 kHz, mono ou stéréo |
| OGG    | OPUS  | 16 bits  | 8 kHz ou 16 kHz, mono ou stéréo |

Dans le cas des flux audio stéréo, les canaux gauche et droit sont fractionnés lors de la transcription. Un fichier de résultat JSON est créé pour chaque canal.
Pour créer une transcription finale ordonnée chronologiquement, utilisez les timestamps générés par énoncé.

### <a name="configuration"></a>Configuration

Les paramètres de configuration sont fournis au format JSON.

**Transcription d’un ou plusieurs fichiers individuels.** Si vous avez plusieurs fichiers à transcrire, nous vous recommandons de les envoyer dans une seule requête. L’exemple ci-dessous utilise trois fichiers :

```json
{
  "contentUrls": [
    "<URL to an audio file 1 to transcribe>",
    "<URL to an audio file 2 to transcribe>",
    "<URL to an audio file 3 to transcribe>"
  ],
  "properties": {
    "wordLevelTimestampsEnabled": true
  },
  "locale": "en-US",
  "displayName": "Transcription of file using default model for en-US"
}
```

**Traitement d’un conteneur de stockage entier :**

```json
{
  "contentContainerUrl": "<SAS URL to the Azure blob container to transcribe>",
  "properties": {
    "wordLevelTimestampsEnabled": true
  },
  "locale": "en-US",
  "displayName": "Transcription of container using default model for en-US"
}
```

**Utilisez un modèle entraîné personnalisé dans une transcription Batch.** L’exemple utilise trois fichiers :

```json
{
  "contentUrls": [
    "<URL to an audio file 1 to transcribe>",
    "<URL to an audio file 2 to transcribe>",
    "<URL to an audio file 3 to transcribe>"
  ],
  "properties": {
    "wordLevelTimestampsEnabled": true
  },
  "locale": "en-US",
  "model": {
    "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/models/{id}"
  },
  "displayName": "Transcription of file using default model for en-US"
}
```


### <a name="configuration-properties"></a>Propriétés de configuration

Utilisez les propriétés facultatives suivantes pour configurer la transcription :

:::row:::
   :::column span="1":::
      **Paramètre**
   :::column-end:::
   :::column span="2":::
      **Description**
:::row-end:::
:::row:::
   :::column span="1":::
      `profanityFilterMode`
   :::column-end:::
   :::column span="2":::
      (Facultatif) La valeur par défaut est `Masked`. Spécifie comment traiter la vulgarité dans les résultats de la reconnaissance. Les valeurs acceptées sont `None` pour désactiver le filtrage des termes vulgaires, `Masked` pour remplacer les termes vulgaires par des astérisques, `Removed` pour supprimer tous les termes vulgaires du résultat ou `Tags` pour ajouter des balises « termes vulgaires ».
:::row-end:::
:::row:::
   :::column span="1":::
      `punctuationMode`
   :::column-end:::
   :::column span="2":::
      (Facultatif) La valeur par défaut est `DictatedAndAutomatic`. Spécifie comment traiter la ponctuation dans les résultats de la reconnaissance. Les valeurs acceptées sont `None` pour désactiver la ponctuation, `Dictated` pour indiquer une ponctuation explicite (parlée), `Automatic` pour permettre au décodeur de traiter la ponctuation ou `DictatedAndAutomatic` pour utiliser la ponctuation dictée et automatique.
:::row-end:::
:::row:::
   :::column span="1":::
      `wordLevelTimestampsEnabled`
   :::column-end:::
   :::column span="2":::
      (Facultatif) `false` par défaut. Spécifie si les timestamps au niveau des mots doivent être ajoutés à la sortie.
:::row-end:::
:::row:::
   :::column span="1":::
      `diarizationEnabled`
   :::column-end:::
   :::column span="2":::
      (Facultatif) `false` par défaut. Spécifie que l’analyse de diarisation doit être effectuée sur l’entrée qui est censée être un canal unique contenant deux voix. Remarque : Requiert la définition de `wordLevelTimestampsEnabled` sur `true`.
:::row-end:::
:::row:::
   :::column span="1":::
      `channels`
   :::column-end:::
   :::column span="2":::
      (Facultatif) `0` et `1` sont transcrits par défaut. Tableau de numéros de canaux à traiter. Ici, vous pouvez spécifier un sous-ensemble des canaux disponibles dans le fichier audio (par exemple, `0` uniquement).
:::row-end:::
:::row:::
   :::column span="1":::
      `timeToLive`
   :::column-end:::
   :::column span="2":::
      (Facultatif) Pas de suppression par défaut. Durée pour supprimer automatiquement les transcriptions terminées. Le paramètre `timeToLive` est utile pour le traitement en masse des transcriptions afin de s’assurer qu’elles seront effectivement supprimées (par exemple, `PT12H` pour 12 heures).
:::row-end:::
:::row:::
   :::column span="1":::
      `destinationContainerUrl`
   :::column-end:::
   :::column span="2":::
      URL facultative avec [SAP ad hoc de service](../../storage/common/storage-sas-overview.md) vers un conteneur accessible en écriture dans Azure. Le résultat est stocké dans ce conteneur. Les SAP avec stratégie d’accès stockée ne sont **pas** prises en charge. Si aucune URL n’est spécifiée, Microsoft stocke les résultats dans un conteneur de stockage géré par Microsoft. Lorsque la transcription est supprimée en appelant [Supprimer la transcription](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/DeleteTranscription), les données de résultats sont également supprimées.
:::row-end:::

### <a name="storage"></a>Stockage

La transcription Batch peut lire l’audio à partir d’un URI Internet visible par le public, et peut lire ou écrire des transcriptions à l’aide d’un URI SAS avec [Stockage Blob Azure](../../storage/blobs/storage-blobs-overview.md).

## <a name="batch-transcription-result"></a>Résultat de la transcription Batch

Pour chaque entrée audio, un fichier de résultat de transcription est créé.
L’opération [Obtenir les fichiers de transcription](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/GetTranscriptionFiles) renvoie une liste de fichiers de résultats pour cette transcription. Pour rechercher le fichier de transcription d’un fichier d’entrée spécifique, filtrez tous les fichiers retournés avec `kind` == `Transcription` et `name` == `{originalInputName.suffix}.json`.

Le format de chaque fichier de résultats de transcription est le suivant :

```json
{
  "source": "...",                      // sas url of a given contentUrl or the path relative to the root of a given container
  "timestamp": "2020-06-16T09:30:21Z",  // creation time of the transcription, ISO 8601 encoded timestamp, combined date and time
  "durationInTicks": 41200000,          // total audio duration in ticks (1 tick is 100 nanoseconds)
  "duration": "PT4.12S",                // total audio duration, ISO 8601 encoded duration
  "combinedRecognizedPhrases": [        // concatenated results for simple access in single string for each channel
    {
      "channel": 0,                     // channel number of the concatenated results
      "lexical": "hello world",
      "itn": "hello world",
      "maskedITN": "hello world",
      "display": "Hello world."
    }
  ],
  "recognizedPhrases": [                // results for each phrase and each channel individually
    {
      "recognitionStatus": "Success",   // recognition state, e.g. "Success", "Failure"
      "channel": 0,                     // channel number of the result
      "offset": "PT0.07S",              // offset in audio of this phrase, ISO 8601 encoded duration 
      "duration": "PT1.59S",            // audio duration of this phrase, ISO 8601 encoded duration
      "offsetInTicks": 700000.0,        // offset in audio of this phrase in ticks (1 tick is 100 nanoseconds)
      "durationInTicks": 15900000.0,    // audio duration of this phrase in ticks (1 tick is 100 nanoseconds)

      // possible transcriptions of the current phrase with confidences
      "nBest": [
        {
          "confidence": 0.898652852,    // confidence value for the recognition of the whole phrase
          "speaker": 1,                 // if `diarizationEnabled` is `true`, this is the identified speaker (1 or 2), otherwise this property is not present
          "lexical": "hello world",
          "itn": "hello world",
          "maskedITN": "hello world",
          "display": "Hello world.",

          // if wordLevelTimestampsEnabled is `true`, there will be a result for each word of the phrase, otherwise this property is not present
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

Le résultat contient les champs suivants :

:::row:::
   :::column span="1":::
      **Champ**
   :::column-end:::
   :::column span="2":::
      **Contenu**
:::row-end:::
:::row:::
   :::column span="1":::
      `lexical`
   :::column-end:::
   :::column span="2":::
      Les mots réels sont reconnus.
:::row-end:::
:::row:::
   :::column span="1":::
      `itn`
   :::column-end:::
   :::column span="2":::
      Forme « normalisation du texte inversée » du texte reconnu. Abréviations (« Docteur Smith » en « Dr Smith »), numéros de téléphone, et d’autres transformations sont appliquées.
:::row-end:::
:::row:::
   :::column span="1":::
      `maskedITN`
   :::column-end:::
   :::column span="2":::
      Forme « normalisation du texte inversée » avec masquage des termes vulgaires appliqué.
:::row-end:::
:::row:::
   :::column span="1":::
      `display`
   :::column-end:::
   :::column span="2":::
      Forme d’affichage du texte reconnu. Des signes de ponctuation et des majuscules ajoutés sont inclus.
:::row-end:::

## <a name="speaker-separation-diarization"></a>Séparation des orateurs (diarisation)

La diarisation est le processus de séparation des orateurs dans une partie de l’audio. Le pipeline Batch prend en charge la diarisation et est capable de reconnaître deux orateurs dans les enregistrements sur canal mono. La fonctionnalité n’est pas disponible pour les enregistrements stéréo.

La sortie d’une transcription avec diarisation activée contient une entrée `Speaker` pour chaque expression transcrite. Si aucune diarisation n’est utilisée, la propriété `Speaker` n’est pas présente dans la sortie JSON. Nous prenons en charge deux voix pour la diarisation. Les orateurs sont identifiés en tant que `1` ou `2`.

Pour demander la diarisation, définissez la propriété `diarizationEnabled` sur `true` comme le montre la requête HTTP ci-dessous.

 ```json
{
  "contentUrls": [
    "<URL to an audio file to transcribe>",
  ],
  "properties": {
    "diarizationEnabled": true,
    "wordLevelTimestampsEnabled": true,
    "punctuationMode": "DictatedAndAutomatic",
    "profanityFilterMode": "Masked"
  },
  "locale": "en-US",
  "displayName": "Transcription of file using default model for en-US"
}
```

Les timestamps au niveau du mot doivent être activés, comme l’indiquent les paramètres dans la requête ci-dessus.

## <a name="best-practices"></a>Meilleures pratiques

Le service de transcription Batch peut traiter un grand nombre de transcriptions envoyées. Vous pouvez interroger l’état de vos transcriptions avec [Obtenir les transcriptions](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/GetTranscriptions).
Appelez régulièrement [Supprimer la transcription](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/DeleteTranscription) à partir du service une fois que vous avez récupéré les résultats. Vous pouvez définir la propriété `timeToLive` pour garantir la suppression définitive des résultats.

## <a name="sample-code"></a>Exemple de code

Des exemples complets sont disponibles dans le [dépôt d’exemples GitHub](https://aka.ms/csspeech/samples) à l’intérieur du sous-répertoire `samples/batch`.

Mettez à jour l’exemple de code avec vos informations d’abonnement, la région du service, le pointage URI vers le fichier audio à transcrire et l’emplacement du modèle si vous utilisez un modèle personnalisé.

[!code-csharp[Configuration variables for batch transcription](~/samples-cognitive-services-speech-sdk/samples/batch/csharp/program.cs#transcriptiondefinition)]

L’exemple de code configure le client et envoie la demande de transcription. Il demande ensuite des informations d’état et imprime les détails de la progression de la transcription.

```csharp
// get the status of our transcriptions periodically and log results
int completed = 0, running = 0, notStarted = 0;
while (completed < 1)
{
    completed = 0; running = 0; notStarted = 0;

    // get all transcriptions for the user
    paginatedTranscriptions = null;
    do
    {
        // <transcriptionstatus>
        if (paginatedTranscriptions == null)
        {
            paginatedTranscriptions = await client.GetTranscriptionsAsync().ConfigureAwait(false);
        }
        else
        {
            paginatedTranscriptions = await client.GetTranscriptionsAsync(paginatedTranscriptions.NextLink).ConfigureAwait(false);
        }

        // delete all pre-existing completed transcriptions. If transcriptions are still running or not started, they will not be deleted
        foreach (var transcription in paginatedTranscriptions.Values)
        {
            switch (transcription.Status)
            {
                case "Failed":
                case "Succeeded":
                    // we check to see if it was one of the transcriptions we created from this client.
                    if (!createdTranscriptions.Contains(transcription.Self))
                    {
                        // not created form here, continue
                        continue;
                    }

                    completed++;

                    // if the transcription was successful, check the results
                    if (transcription.Status == "Succeeded")
                    {
                        var paginatedfiles = await client.GetTranscriptionFilesAsync(transcription.Links.Files).ConfigureAwait(false);

                        var resultFile = paginatedfiles.Values.FirstOrDefault(f => f.Kind == ArtifactKind.Transcription);
                        var result = await client.GetTranscriptionResultAsync(new Uri(resultFile.Links.ContentUrl)).ConfigureAwait(false);
                        Console.WriteLine("Transcription succeeded. Results: ");
                        Console.WriteLine(JsonConvert.SerializeObject(result, SpeechJsonContractResolver.WriterSettings));
                    }
                    else
                    {
                        Console.WriteLine("Transcription failed. Status: {0}", transcription.Properties.Error.Message);
                    }

                    break;

                case "Running":
                    running++;
                    break;

                case "NotStarted":
                    notStarted++;
                    break;
            }
        }

        // for each transcription in the list we check the status
        Console.WriteLine(string.Format("Transcriptions status: {0} completed, {1} running, {2} not started yet", completed, running, notStarted));
    }
    while (paginatedTranscriptions.NextLink != null);

    // </transcriptionstatus>
    // check again after 1 minute
    await Task.Delay(TimeSpan.FromMinutes(1)).ConfigureAwait(false);
}
```

Pour plus d’informations sur les appels précédents, consultez notre [document Swagger](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0). L’exemple complet présenté ici est disponible sur [GitHub](https://aka.ms/csspeech/samples) dans le sous-répertoire `samples/batch`.

Cet exemple utilise une configuration asynchrone pour publier des données audio et recevoir l’état de la transcription.
La méthode `PostTranscriptions` envoie les détails du fichier audio et la méthode `GetTranscriptions` reçoit les états.
`PostTranscriptions` renvoie un descripteur et `GetTranscriptions` l’utilise pour créer un descripteur permettant d’obtenir l’état de la transcription.

Cet exemple de code ne spécifie pas de modèle personnalisé. Le service utilise le modèle de base pour la transcription du ou des fichiers. Pour spécifier le modèle, vous pouvez appliquer la même méthode de référence de modèle pour le modèle personnalisé.

> [!NOTE]
> Dans le cas des transcriptions de base, vous n’avez pas besoin de déclarer l’ID du modèle de base.

## <a name="next-steps"></a>Étapes suivantes

- [Référence de l’API de reconnaissance vocale v3](https://centralus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/CopyModelToSubscription)