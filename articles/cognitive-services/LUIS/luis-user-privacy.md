---
title: Exporter et supprimer des données - LUIS
titleSuffix: Azure Cognitive Services
description: Vous avez le contrôle total sur l’affichage, l’exportation et la suppression de leurs données. Supprimer les données des clients pour garantir la confidentialité et la conformité.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 12/08/2020
ms.openlocfilehash: cd9b353300b88c69efb0c458245a86963c579064
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97007736"
---
# <a name="export-and-delete-your-customer-data-in-language-understanding-luis-in-cognitive-services"></a>Exporter et supprimer vos données client dans Language Understanding (LUIS) dans Cognitive Services

Supprimer les données des clients pour garantir la confidentialité et la conformité.

## <a name="summary-of-customer-data-request-features"></a>Résumé des fonctionnalités de demande de données client
Language Understanding Intelligent Service (LUIS) préserve le contenu du client pour utiliser le service, mais l’utilisateur de LUIS contrôle totalement l’affichage, l’exportation et la suppression des ses données. Il peut le faire par le biais du [portail](luis-reference-regions.md) web LUIS ou des [API de création (également appelées programmatiques) de LUIS](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c2f).

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

Le contenu du client est chiffré dans le stockage Azure régional Microsoft et inclut ce qui suit :

- Contenu du compte utilisateur collecté lors de l’inscription
- Données de formation requises pour créer les modèles
- Demandes utilisateur enregistrées utilisées par l’ [apprentissage actif](luis-concept-review-endpoint-utterances.md) pour aider à améliorer le modèle
  - Les utilisateurs peuvent désactiver la journalisation des requêtes en ajoutant `&log=false` à la demande. Pour plus de détails, voir [ici](troubleshooting.md#how-can-i-disable-the-logging-of-utterances)

## <a name="deleting-customer-data"></a>Suppression des données client
Les utilisateurs de LUIS contrôlent totalement la suppression de tout contenu utilisateur, via le portail web de LUIS ou à l’aide des API de création (également appelées programmatiques) de LUIS. Le tableau suivant affiche les liens d’assistance pour les deux :

| | **Compte d’utilisateur** | **Application** | **Exemple(s) d’énoncé(s)** | **Requêtes utilisateur final** |
| --- | --- | --- | --- | --- |
| **Portail** | [Lien](luis-concept-data-storage.md#delete-an-account) | [Lien](luis-how-to-start-new-app.md#delete-app) | [Lien](luis-concept-data-storage.md#utterances-in-an-intent) | [Énoncés d’apprentissage actif](luis-how-to-review-endpoint-utterances.md#disable-active-learning)<br>[Énoncés enregistrés](luis-concept-data-storage.md#disable-logging-utterances) |
| **API** | [Lien](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c4c) | [Lien](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c39) | [Lien](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c0b) | [Lien](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/58b6f32139e2bb139ce823c9) |


## <a name="exporting-customer-data"></a>Exportation des données client
Les utilisateurs de LUIS contrôlent totalement l’affichage des données sur le portail. Celles-ci doivent cependant être exportées via les API de création (également appelées programmatiques) de LUIS. Le tableau suivant affiche des liens d’assistance pour l’exportation de données via les API de création (également appelées programmatiques) de LUIS :

| | **Compte d’utilisateur** | **Application** | **Énoncé(s)** | **Requêtes utilisateur final** |
| --- | --- | --- | --- | --- |
| **API** | [Lien](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c48) | [Lien](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c40) | [Lien](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c0a) | [Lien](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c36) |

## <a name="location-of-active-learning"></a>Emplacement de l’apprentissage actif

Pour activer l’[apprentissage actif](luis-how-to-review-endpoint-utterances.md#log-user-queries-to-enable-active-learning), les énoncés enregistrés des utilisateurs, reçus au niveau des points de terminaison LUIS publiés, sont stockés dans les zones géographiques Azure suivantes :

* [Europe](#europe)
* [Australie](#australia)
* [États-Unis](#united-states)

À l’exception des données d’apprentissage actif (décrites ci-dessous), LUIS suit les [pratiques de stockage de données pour les services régionaux](https://azuredatacentermap.azurewebsites.net/).

### <a name="europe"></a>Europe

Le portail [eu.luis.ai](https://eu.luis.ai) et Europe Authoring (également appelé API programmatiques) sont hébergés dans la zone géographique Europe d’Azure. Le portail eu.luis.ai et Europe Authoring (également appelé API programmatiques) prennent en charge le déploiement de points de terminaison dans les zones géographiques Azure suivantes :

* Europe
* France
* United Kingdom

Lors du déploiement dans ces zones géographiques Azure, les énoncés provenant des utilisateurs finaux de votre application et reçus par le point de terminaison sont stockés dans la zone géographique Europe d’Azure pour l’apprentissage actif. Vous pouvez désactiver l’apprentissage actif, consultez [Désactiver l’apprentissage actif](luis-how-to-review-endpoint-utterances.md#disable-active-learning). Pour gérer les énoncés stockés, consultez [Supprimer un énoncé](luis-how-to-review-endpoint-utterances.md#delete-utterance).

### <a name="australia"></a>Australie

Le portail [au.luis.ai](https://au.luis.ai) et Australia Authoring (également appelé API programmatiques) sont hébergés dans la zone géographique Australie d’Azure. Le portail au.luis.ai et Australia Authoring (également appelé API programmatiques) prennent en charge le déploiement de points de terminaison dans les zones géographiques Azure suivantes :

* Australie

Lors du déploiement dans ces zones géographiques Azure, les énoncés provenant des utilisateurs finaux de votre application et reçus par le point de terminaison sont stockés dans la zone géographique Australie d’Azure pour l’apprentissage actif. Vous pouvez désactiver l’apprentissage actif, consultez [Désactiver l’apprentissage actif](luis-how-to-review-endpoint-utterances.md#disable-active-learning). Pour gérer les énoncés stockés, consultez [Supprimer un énoncé](luis-how-to-review-endpoint-utterances.md#delete-utterance).

### <a name="united-states"></a>États-Unis

Le portail [luis.ai](https://www.luis.ai) et United States Authoring (également appelé API programmatiques) sont hébergés dans la zone géographique États-Unis d’Azure. Le portail luis.ai et United States Authoring (également appelé API programmatiques) prennent en charge le déploiement de points de terminaison dans les zones géographiques Azure suivantes :

* Zones géographiques Azure pas prises en charge par les régions de création d’Europe ou d’Australie

Lors du déploiement dans ces zones géographiques Azure, les énoncés provenant des utilisateurs finaux de votre application et reçus par le point de terminaison sont stockés dans la zone géographique États-Unis d’Azure pour l’apprentissage actif. Vous pouvez désactiver l’apprentissage actif, consultez [Désactiver l’apprentissage actif](luis-how-to-review-endpoint-utterances.md#disable-active-learning). Pour gérer les énoncés stockés, consultez [Supprimer un énoncé](luis-how-to-review-endpoint-utterances.md#delete-utterance).


## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Documentation de référence sur les régions LUIS](./luis-reference-regions.md)
