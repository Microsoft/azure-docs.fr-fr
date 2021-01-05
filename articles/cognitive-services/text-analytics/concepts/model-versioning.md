---
title: Spécifier la version du modèle dans Analyse de texte v3
titleSuffix: Azure Cognitive Services
description: Découvrez comment spécifier le modèle d’API Analyse de texte utilisé sur vos données.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: article
ms.date: 07/16/2020
ms.author: aahi
ms.openlocfilehash: 7ab2e037cc7593431a668a216fcc152c0511c410
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965083"
---
# <a name="model-versioning-in-the-text-analytics-api"></a>Contrôle de version de modèle dans l’API Analyse de texte

La version 3 de l’API Analyse de texte vous permet de choisir la version de modèle utilisée sur vos données. Utilisez le paramètre facultatif `model-version` pour sélectionner la version du modèle dans vos demandes d’API. Par exemple : `<resource-url>/text/analytics/v3.0/sentiment?model-version=2020-04-01`. Si ce paramètre n’est pas spécifié, l’API a la dernière version stable par défaut. 

## <a name="available-versions"></a>Versions disponibles

Utilisez le tableau ci-dessous pour rechercher les versions de modèle prises en charge par chaque point de terminaison hébergé.


| Point de terminaison                        | Versions prises en charge                                     | version la plus récente |
|---------------------------------|--------------------------------------------------------|----------------|
| `/sentiment`                    | `2019-10-01`, `2020-04-01`                             | `2020-04-01`   |
| `/languages`                    | `2019-10-01`, `2020-07-01`, `2020-09-01`               | `2020-09-01`   |
| `/entities/linking`             | `2019-10-01`, `2020-02-01`                             | `2020-02-01`   |
| `/entities/recognition/general` | `2019-10-01`, `2020-02-01`, `2020-04-01`,              | `2020-04-01`   |
| `/entities/recognition/pii`     | `2019-10-01`, `2020-02-01`, `2020-04-01`,`2020-07-01`  | `2020-07-01`   |
| `/entities/health`              | `2020-09-03`                           | `2020-09-03`   |
| `/keyphrases`                   | `2019-10-01`, `2020-07-01`                             | `2020-07-01`   |


Pour plus d’informations sur les mises à jour de ces modèles, consultez la section [Nouveautés](../whats-new.md).

## <a name="text-analytics-for-health"></a>Analyse de texte pour la santé

Le conteneur [Analyse de texte pour l’intégrité](../how-tos/text-analytics-for-health.md) utilise une gestion de versions de modèle distinct des points de terminaison d’API ci-dessus.  Notez qu’une seule version de modèle est disponible par image conteneur.

| Point de terminaison                        | Étiquette de l’image conteneur                     | Version du modèle |
|---------------------------------|-----------------------------------------|---------------|
| `/entities/health`              | `1.1.013530001-amd64-preview` ou la plus récente          | `2020-09-03`  |
| `/entities/health`              | `1.1.013150001-amd64-preview`           | `2020-07-24`  |
| `/domains/health`               | `1.1.012640001-amd64-preview`           | `2020-05-08`  |
| `/domains/health`               | `1.1.012420001-amd64-preview`           | `2020-05-08`  |
| `/domains/health`               | `1.1.012070001-amd64-preview`           | `2020-04-16`  |


## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble d’Analyse de texte](../overview.md)
* [Analyse des sentiments](../how-tos/text-analytics-how-to-sentiment-analysis.md)
* [Reconnaissance d’entités](../how-tos/text-analytics-how-to-entity-linking.md)
