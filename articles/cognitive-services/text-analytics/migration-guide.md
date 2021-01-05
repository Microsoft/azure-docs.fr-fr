---
title: Guide de migration pour la version 2 de l’API Analyse de texte
titleSuffix: Azure Cognitive Services
description: Découvrez comment déplacer vos applications pour utiliser la version 3 de l’API Analyse de texte.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: article
ms.date: 12/17/2020
ms.author: aahi
ms.openlocfilehash: 6a71bcbfb8341098711e330cebf8545e1fd2751c
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97656952"
---
# <a name="migrate-to-version-3x-of-the-text-analytics-api"></a>Migrer vers la version 3.x de l’API Analyse de texte

Si vous utilisez la version 2.1 de l’API Analyse de texte, cet article vous aide à mettre à niveau votre application pour utiliser la version 3.x. La version 3.0 est en disponibilité générale et introduit de nouvelles fonctionnalités comme la [Reconnaissance d’entité nommée (NER)](how-tos/text-analytics-how-to-entity-linking.md#named-entity-recognition-features-and-versions) et le [contrôle de version de modèle](concepts/model-versioning.md). Une préversion de v3.1 (v3.1-preview.x) est également disponible, elle ajoute des fonctionnalités comme l’[exploration des opinions](how-tos/text-analytics-how-to-sentiment-analysis.md#sentiment-analysis-versions-and-features). Les modèles utilisés dans v2 ne reçoivent pas les mises à jour ultérieures. 

## <a name="sentiment-analysis"></a>[Analyse des sentiments](#tab/sentiment-analysis)

### <a name="feature-changes"></a>Évolution des fonctionnalités 

L’analyse des sentiments dans la version 2.1 retourne des scores de sentiment entre 0 et 1 pour chaque document envoyé à l’API, les scores plus proches de 1 indiquant un sentiment plus positif. La version 3 à la place retourne des étiquettes de sentiment (par exemple, « positif » ou « négatif ») pour les phrases et le document dans son ensemble, ainsi que les scores de confiance associés. 

### <a name="steps-to-migrate"></a>Étapes de migration

#### <a name="rest-api"></a>API REST

Si votre application utilise l’API REST, mettez à jour son point de terminaison de demande sur le point de terminaison v3 pour l’analyse des sentiments. Par exemple, `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/sentiment`. Vous devez également mettre à jour l’application pour utiliser les étiquettes de sentiment retournées dans la [réponse de l’API](how-tos/text-analytics-how-to-sentiment-analysis.md#view-the-results). 

Consultez la documentation de référence pour obtenir des exemples de réponse JSON.
* [Version 2.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v2-1/operations/56f30ceeeda5650db055a3c9)
* [Version 3.0](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/Sentiment) 
* [Version 3.1-preview](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/Sentiment)

#### <a name="client-libraries"></a>Bibliothèques clientes

[!INCLUDE [Client library migration information](includes/client-library-migration-section.md)]

## <a name="ner-and-entity-linking"></a>[NER et liaison d’entités](#tab/named-entity-recognition)

### <a name="feature-changes"></a>Évolution des fonctionnalités

> [!NOTE] 
> Actuellement, les [catégories d’entité v3](named-entity-types.md) sont retournées uniquement sur le texte en anglais et espagnol. L’API retourne les résultats de la version 2.1 pour les demandes dans d’autres langues, à condition qu’elles soient prises en charge dans la version 2.1.

Dans la version 2.1, l’API Analyse de texte utilise un seul point de terminaison pour la reconnaissance d’entité nommée (NER) et la liaison d’entités. La version 3 fournit une détection étendue des entités nommées et utilise des points de terminaison séparés pour les demandes NER et de liaison d’entités. À partir de v3.1-preview.1, NER peut également détecter les informations personnelles `pii` et médicales `phi`. 

### <a name="steps-to-migrate"></a>Étapes de migration

#### <a name="rest-api"></a>API REST

Si votre application utilise l’API REST, mettez à jour son point de terminaison de demande sur le point de terminaison v3 pour NER et/ou la liaison d’entités.

Liaison d’entités
* `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/entities/linking`

NER
* `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/entities/recognition/general`

Vous devez également mettre à jour votre application pour utiliser les [catégories d’entité](named-entity-types.md) retournées dans la [réponse de l’API](how-tos/text-analytics-how-to-entity-linking.md#view-results).

Consultez la documentation de référence pour obtenir des exemples de réponse JSON.
* [Version 2.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v2-1/operations/5ac4251d5b4ccd1554da7634)
* [Version 3.0](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/EntitiesRecognitionGeneral) 
* [Version 3.1-preview](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/EntitiesRecognitionGeneral)

#### <a name="client-libraries"></a>Bibliothèques clientes

[!INCLUDE [Client library migration information](includes/client-library-migration-section.md)]

## <a name="language-detection"></a>[Détection de la langue](#tab/language-detection)

### <a name="feature-changes"></a>Évolution des fonctionnalités 

La fonctionnalité de détection de langue n’a pas changé dans v3 en dehors de la version du point de terminaison, mais la réponse JSON contient `ConfidenceScore` au lieu de `score`. V3 retourne également seulement une langue dans la sortie. 

### <a name="steps-to-migrate"></a>Étapes de migration

#### <a name="rest-api"></a>API REST

Si votre application utilise l’API REST, mettez à jour son point de terminaison de demande sur le point de terminaison v3 pour la détection de langue. Par exemple, `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/languages`. Vous devez également mettre à jour l’application pour utiliser `ConfidenceScore` au lieu de `score` dans la [réponse de l’API](how-tos/text-analytics-how-to-language-detection.md#step-3-view-the-results). 

Consultez la documentation de référence pour obtenir des exemples de réponse JSON.
* [Version 2.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v2-1/operations/56f30ceeeda5650db055a3c7)
* [Version 3.0](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/Languages) 
* [Version 3.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/Languages)

#### <a name="client-libraries"></a>Bibliothèques clientes

[!INCLUDE [Client library migration information](includes/client-library-migration-section.md)]

## <a name="key-phrase-extraction"></a>[Extraction de phrases clés](#tab/key-phrase-extraction)

### <a name="feature-changes"></a>Évolution des fonctionnalités 

La fonctionnalité d’extraction d’expressions clés n’a pas changé dans v3 en dehors de la version du point de terminaison.

### <a name="steps-to-migrate"></a>Étapes de migration

#### <a name="rest-api"></a>API REST

Si votre application utilise l’API REST, mettez à jour son point de terminaison de demande sur le point de terminaison v3 pour l’extraction d’expressions clés. Par exemple : `https://<your-custom-subdomain>.api.cognitiveservices.azure.com/text/analytics/v3.0/keyPhrases`

Consultez la documentation de référence pour obtenir des exemples de réponse JSON.
* [Version 2.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v2-1/operations/56f30ceeeda5650db055a3c6)
* [Version 3.0](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/KeyPhrases) 
* [Version 3.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-1/operations/KeyPhrases)

#### <a name="client-libraries"></a>Bibliothèques clientes

[!INCLUDE [Client library migration information](includes/client-library-migration-section.md)]

---

## <a name="see-also"></a>Voir aussi

* [Qu’est-ce que l’API Analyse de texte ?](overview.md)
* [Prise en charge linguistique](language-support.md)
* [Contrôle de version de modèle](concepts/model-versioning.md)
