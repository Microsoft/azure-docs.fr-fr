---
title: Guide pratique pour utiliser le SDK Speech pour l’évaluation de la prononciation
titleSuffix: Azure Cognitive Services
description: Le SDK Speech prend en charge l’évaluation de la prononciation, qui évalue la qualité de la prononciation des entrées vocales, avec des indicateurs de justesse, d’aisance, d’exhaustivité, et ainsi de suite.
services: cognitive-services
author: yulin-li
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 09/29/2020
ms.author: yulili
ms.custom: references_regions
zone_pivot_groups: programming-languages-set-nineteen
ms.openlocfilehash: 1cc313daf0e76ddd14865959410b07d9bdc189d7
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94984919"
---
# <a name="pronunciation-assessment"></a>Évaluation de la prononciation

L’évaluation de la prononciation évalue la prononciation des entrées vocales et fournit des commentaires aux orateurs sur la justesse et l’aisance des paroles prononcées.
Grâce à l’évaluation de la prononciation, les élèves qui apprennent des langues peuvent pratiquer, obtenir des commentaires instantanés et améliorer leur prononciation pour pouvoir parler et se présenter en toute confiance.
Les enseignants peuvent utiliser la fonctionnalité pour évaluer la prononciation de plusieurs intervenants en temps réel.

Dans cet article, vous allez découvrir comment configurer `PronunciationAssessmentConfig` et récupérer le `PronunciationAssessmentResult` à l’aide du SDK Speech.

> [!NOTE]
> La fonctionnalité d’évaluation de la prononciation est actuellement disponible uniquement dans les régions `westus`, `eastasia` et `centralindia`, et ne prend en charge que la langue `en-US`.

## <a name="pronunciation-assessment-with-the-speech-sdk"></a>Évaluation de la prononciation avec le SDK Speech

Dans les exemples ci-dessous, vous allez créer un `PronunciationAssessmentConfig`, puis l’appliquer à un `SpeechRecognizer`.

Les extraits de code suivants montrent comment utiliser la détection automatique de la langue dans vos applications :

::: zone pivot="programming-language-csharp"

```csharp
var pronunciationAssessmentConfig = new PronunciationAssessmentConfig(
    "reference text", GradingSystem.HundredMark, Granularity.Phoneme);

using (var recognizer = new SpeechRecognizer(
    speechConfig,
    audioConfig))
{
    // apply the pronunciation assessment configuration to the speech recognizer
    pronunciationAssessmentConfig.ApplyTo(recognizer);
    var speechRecognitionResult = await recognizer.RecognizeOnceAsync();
    var pronunciationAssessmentResult =
        PronunciationAssessmentResult.FromResult(speechRecognitionResult);
    var pronunciationScore = pronunciationAssessmentResult.PronunciationScore;
}
```

::: zone-end

::: zone pivot="programming-language-cpp"

```cpp
auto pronunciationAssessmentConfig =
    PronunciationAssessmentConfig::Create("reference text",
        PronunciationAssessmentGradingSystem::HundredMark,
        PronunciationAssessmentGranularity::Phoneme);

auto recognizer = SpeechRecognizer::FromConfig(
    speechConfig,
    audioConfig);

// apply the pronunciation assessment configuration to the speech recognizer
pronunciationAssessmentConfig->ApplyTo(recognizer);
speechRecognitionResult = recognizer->RecognizeOnceAsync().get();
auto pronunciationAssessmentResult =
    PronunciationAssessmentResult::FromResult(speechRecognitionResult);
auto pronunciationScore = pronunciationAssessmentResult->PronunciationScore;
```

::: zone-end

::: zone pivot="programming-language-java"

```java
PronunciationAssessmentConfig pronunciationAssessmentConfig =
    new PronunciationAssessmentConfig("reference text", 
        PronunciationAssessmentGradingSystem.HundredMark,
        PronunciationAssessmentGranularity.Phoneme);

SpeechRecognizer recognizer = new SpeechRecognizer(
    speechConfig,
    audioConfig);

// apply the pronunciation assessment configuration to the speech recognizer
pronunciationAssessmentConfig.applyTo(recognizer);
Future<SpeechRecognitionResult> future = recognizer.recognizeOnceAsync();
SpeechRecognitionResult result = future.get(30, TimeUnit.SECONDS);
PronunciationAssessmentResult pronunciationAssessmentResult =
    PronunciationAssessmentResult.fromResult(result);
Double pronunciationScore = pronunciationAssessmentResult.getPronunciationScore();

recognizer.close();
speechConfig.close();
audioConfig.close();
pronunciationAssessmentConfig.close();
result.close();
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
pronunciation_assessment_config = \
        speechsdk.PronunciationAssessmentConfig(reference_text='reference text',
                grading_system=msspeech.PronunciationAssessmentGradingSystem.HundredMark,
                granularity=msspeech.PronunciationAssessmentGranularity.Phoneme)
speech_recognizer = speechsdk.SpeechRecognizer(
        speech_config=speech_config, \
        audio_config=audio_config)

# apply the pronunciation assessment configuration to the speech recognizer
pronunciation_assessment_config.apply_to(speech_recognizer)
result = speech_recognizer.recognize_once()
pronunciation_assessment_result = speechsdk.PronunciationAssessmentResult(result)
pronunciation_score = pronunciation_assessment_result.pronunciation_score
```

::: zone-end

::: zone pivot="programming-language-objectivec"

```Objective-C
SPXPronunciationAssessmentConfiguration* pronunciationAssessmentConfig =
    [[SPXPronunciationAssessmentConfiguration alloc]init:@"reference text"
                                           gradingSystem:SPXPronunciationAssessmentGradingSystem_HundredMark
                                             granularity:SPXPronunciationAssessmentGranularity_Phoneme];

SPXSpeechRecognizer* speechRecognizer = \
        [[SPXSpeechRecognizer alloc] initWithSpeechConfiguration:speechConfig
                                              audioConfiguration:audioConfig];

// apply the pronunciation assessment configuration to the speech recognizer
[pronunciationAssessmentConfig applyToRecognizer:speechRecognizer];

SPXSpeechRecognitionResult *result = [speechRecognizer recognizeOnce];
SPXPronunciationAssessmentResult* pronunciationAssessmentResult = [[SPXPronunciationAssessmentResult alloc] init:result];
double pronunciationScore = pronunciationAssessmentResult.pronunciationScore;
```

::: zone-end

### <a name="pronunciation-assessment-configuration-parameters"></a>Paramètres de configuration de l’évaluation de la prononciation

Ce tableau liste les paramètres de configuration pour l’évaluation de la prononciation.

| Paramètre | Description | Requis ? |
|-----------|-------------|---------------------|
| ReferenceText | Texte duquel la prononciation est évaluée. | Obligatoire |
| GradingSystem | Système de points pour la calibration du score. Le système `FivePoint` donne un score à virgule flottante de 0 à 5 et `HundredMark` donne un score à virgule flottante de 0 à 100. Par défaut : `FivePoint`. | Facultatif |
| Granularité | Granularité de l’évaluation. Les valeurs acceptées sont `Phoneme`, qui affiche le score au niveau du texte intégral, du mot et du phonème, `Word`, qui affiche le score au niveau du texte intégral et du mot, `FullText`, qui affiche le score au niveau du texte intégral uniquement. Par défaut : `Phoneme`. | Facultatif |
| EnableMiscue | Active le calcul de faute de langue. Quand cette option est activée, les mots prononcés sont comparés au texte de référence et sont marqués comme omission/insertion en fonction de la comparaison. Les valeurs acceptées sont `False` et `True`. Par défaut : `False`. | Facultatif |
| ScenarioId | GUID indiquant un système de points personnalisé. | Facultatif |

### <a name="pronunciation-assessment-result-parameters"></a>Paramètres de résultat de l’évaluation de la prononciation

Ce tableau liste les paramètres de résultat de l’évaluation de la prononciation.

| Paramètre | Description |
|-----------|-------------|
| `AccuracyScore` | Précision de prononciation du discours. La précision indique dans quelle mesure les phonèmes correspondent à la prononciation d’un intervenant de langue maternelle. Le score de précision au niveau du mot et du texte intégral est agrégé à partir du score de précision du niveau du phonème. |
| `FluencyScore` | Fluidité du discours concerné. La fluidité indique dans quelle mesure le discours correspond à l’utilisation qu’un orateur de langue maternelle fait des pauses entre les mots. |
| `CompletenessScore` | Intégralité du discours, déterminée par le calcul du rapport entre les mots prononcés et l’entrée de texte de référence. |
| `PronunciationScore` | Score global indiquant la qualité de prononciation du discours concerné. Il est agrégé à partir de `AccuracyScore`, `FluencyScore` et `CompletenessScore` avec une pondération. |
| `ErrorType` | Cette valeur indique si un mot est omis, inséré ou mal prononcé par rapport à `ReferenceText`. Les valeurs possibles sont `None` (aucune erreur sur ce mot), `Omission`, `Insertion` et `Mispronunciation`. |

## <a name="next-steps"></a>Étapes suivantes

<!-- TODO: update the sample links after release -->

<!-- ::: zone pivot="programming-language-csharp"
* See the [sample code](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/csharp/sharedcontent/console/speech_recognition_samples.cs#L741) on GitHub for automatic language detection
::: zone-end

::: zone pivot="programming-language-cpp"
* See the [sample code](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/cpp/windows/console/samples/speech_recognition_samples.cpp#L507) on GitHub for automatic language detection
::: zone-end

::: zone pivot="programming-language-java"
* See the [sample code](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/java/jre/console/src/com/microsoft/cognitiveservices/speech/samples/console/SpeechRecognitionSamples.java#L521) on GitHub for automatic language detection
::: zone-end

::: zone pivot="programming-language-python"
* See the [sample code](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/python/console/speech_synthesis_sample.py#L434) on GitHub for automatic language detection
::: zone-end

::: zone pivot="programming-language-objectivec"
* See the [sample code](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/objective-c/ios/speech-samples/speech-samples/ViewController.m#L494) on GitHub for automatic language detection
::: zone-end -->

* [Documentation de référence du SDK Speech](speech-sdk.md)
