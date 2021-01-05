---
title: Comment suivre l’utilisation de la mémoire du Kit de développement logiciel (SDK) de reconnaissance vocale – Service de reconnaissance vocale
titleSuffix: Azure Cognitive Services
description: Le Kit de développement logiciel (SDK) du service de reconnaissance vocale prend en charge de nombreux langages de programmation pour la reconnaissance vocale, la synthèse vocale et la traduction vocale. Cet article décrit les outils de gestion de la mémoire intégrés dans le Kit de développement logiciel (SDK).
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 12/10/2019
ms.author: rhurey
ms.custom: devx-track-csharp
zone_pivot_groups: programming-languages-set-two
ms.openlocfilehash: c7e74ce8f4d0b7889d2e5bcd117eaa43e2b6991b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88934138"
---
# <a name="how-to-track-speech-sdk-memory-usage"></a>Comment suivre l’utilisation de la mémoire du Kit de développement logiciel (SDK) de reconnaissance vocale

Le Kit de développement logiciel (SDK) de reconnaissance vocale est basé sur un code natif qui est projeté dans plusieurs langages de programmation via une série de couches d’interopérabilité. Chaque projection spécifique d’une langue inclut des fonctionnalités correctes en langue familière pour gérer le cycle de vie de l’objet. En outre, le Kit de développement logiciel (SDK) de reconnaissance vocale inclut des outils de gestion de la mémoire pour suivre l’utilisation des ressources avec la journalisation et les limites des objets. 

## <a name="how-to-read-object-logs"></a>Comment lire les journaux d’objet

Si [la journalisation du Kit de développement logiciel (SDK) de reconnaissance vocale est activée](how-to-use-logging.md), des balises de suivi sont émises pour activer l’observation d’objets historiques. Ces balises sont les suivantes : 

* `TrackHandle` ou `StopTracking` 
* Type d’objet.
* Nombre actuel d’objets dont le type est suivi et nombre actuel d’objets suivis.

Voici un exemple de journal : 

```terminal
(284): 8604ms SPX_DBG_TRACE_VERBOSE:  handle_table.h:90 TrackHandle type=Microsoft::CognitiveServices::Speech::Impl::ISpxRecognitionResult handle=0x0x7f688401e1a0, ptr=0x0x7f688401e1a0, total=19
```

## <a name="set-a-warning-threshold"></a>Définir un seuil d’avertissement

Vous avez la possibilité de créer un seuil d’avertissement. En cas de dépassement de ce seuil (en supposant que la journalisation est activée), un message d’avertissement est consigné. Le message d’avertissement contient un vidage de tous les objets existants, ainsi que leur nombre. Ces informations permettent de mieux comprendre les problèmes. 

Pour activer un seuil d’avertissement, vous devez spécifier celui-ci sur un objet `SpeechConfig`. Cet objet est vérifié lors de la création d’un module de reconnaissance. Dans les exemples suivants, supposons que vous avez créé une instance de `SpeechConfig` appelée `config` :

::: zone pivot="programming-language-csharp"

```csharp
config.SetProperty("SPEECH-ObjectCountWarnThreshold", "10000");
```

::: zone-end

::: zone pivot="programming-language-cpp"

```C++
config->SetProperty("SPEECH-ObjectCountWarnThreshold", "10000");
```

::: zone-end

::: zone pivot="programming-language-java"

```java
config.setProperty("SPEECH-ObjectCountWarnThreshold", "10000");
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
speech_config.set_property_by_name("SPEECH-ObjectCountWarnThreshold", "10000")?
```

::: zone-end

::: zone pivot="programming-language-more"

```ObjectiveC
[config setPropertyTo:@"10000" byName:"SPEECH-ObjectCountWarnThreshold"];
```

::: zone-end

> [!TIP]
> La valeur par défaut de cette propriété est 10 000.

## <a name="set-an-error-threshold"></a>Définir un seuil d’erreur 

Le Kit de développement logiciel (SDK) de reconnaissance vocale vous permet de définir le nombre maximal d’objets autorisé à un moment donné. Si ce paramètre est activé, lorsque le nombre maximal est atteint, les nouvelles tentatives de création d’objets module de reconnaissance échouent. Les objets existants continuent à fonctionner.

Voici un exemple d’erreur :

```terminal
Runtime error: The maximum object count of 500 has been exceeded.
The threshold can be adjusted by setting the SPEECH-ObjectCountErrorThreshold property on the SpeechConfig object.
See http://https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-object-tracking-speech-sdk for more detailed information.
Handle table dump by ojbect type:
class Microsoft::CognitiveServices::Speech::Impl::ISpxRecognitionResult 0
class Microsoft::CognitiveServices::Speech::Impl::ISpxRecognizer 0
class Microsoft::CognitiveServices::Speech::Impl::ISpxAudioConfig 0
class Microsoft::CognitiveServices::Speech::Impl::ISpxSpeechConfig 0
```

Pour activer un seuil d’erreur, vous devez spécifier celui-ci sur un objet `SpeechConfig`. Cet objet est vérifié lors de la création d’un module de reconnaissance. Dans les exemples suivants, supposons que vous avez créé une instance de `SpeechConfig` appelée `config` :

::: zone pivot="programming-language-csharp"

```csharp
config.SetProperty("SPEECH-ObjectCountErrorThreshold", "10000");
```

::: zone-end

::: zone pivot="programming-language-cpp"

```C++
config->SetProperty("SPEECH-ObjectCountErrorThreshold", "10000");
```

::: zone-end

::: zone pivot="programming-language-java"

```java
config.setProperty("SPEECH-ObjectCountErrorThreshold", "10000");
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
speech_config.set_property_by_name("SPEECH-ObjectCountErrorThreshold", "10000")?
```

::: zone-end

::: zone pivot="programming-language-more"

```objc
[config setPropertyTo:@"10000" byName:"SPEECH-ObjectCountErrorThreshold"];
```

::: zone-end

> [!TIP]
> La valeur par défaut de cette propriété est la valeur maximale spécifique de la plateforme pour un type de données `size_t`. Une reconnaissance consomme généralement entre 7 et 10 objets internes.

## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur le SDK Speech](speech-sdk.md)