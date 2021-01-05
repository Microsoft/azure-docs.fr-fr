---
title: Limites - LUIS
description: Cet article liste les limites connues d’Azure Cognitive Services Language Understanding (LUIS). LUIS a plusieurs zones de limites. La limite de modèle contrôle les intentions, les entités et les caractéristiques dans LUIS. La limite de quota dépend du type de clé. La combinaison de touches contrôle le site web de LUIS.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 06/04/2020
ms.openlocfilehash: c855be6d31a1ee46434ecadbeae7a36dd6a3ff95
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95018801"
---
# <a name="limits-for-your-luis-model-and-keys"></a>Limites de vos clés et de votre modèle LUIS
LUIS a plusieurs zones de limites. La première est la [limite de modèle](#model-limits), qui contrôle les intentions, les entités et les caractéristiques dans LUIS. Le deuxième domaine est la [limite de quota](#key-limits), qui est fonction du type de clé. La troisième zone de limites est la [séquence de touches](#keyboard-controls) pour contrôler le site web LUIS. Un quatrième domaine est le [mappage de régions du monde](luis-reference-regions.md) entre le site web de création de LUIS et les API du [point de terminaison](luis-glossary.md#endpoint) de LUIS.

<a name="model-boundaries"></a>

## <a name="model-limits"></a>Limites de modèle

Si votre application dépasse les limites de modèle LUIS, utilisez une application de [distribution LUIS](luis-concept-enterprise.md#dispatch-tool-and-model) ou un [conteneur LUIS](luis-container-howto.md).

|Domaine|Limite|
|--|:--|
| [Nom de l’application][luis-get-started-create-app] | \* Nombre maximum de caractère par défaut |
| Applications| 500 applications par ressource de création Azure |
| [Test par lot][batch-testing]| jeux de 10 données, 1 000 énoncés par jeu de données|
| Liste explicite | 50 par application|
| Entités externes | sans limite |
| [Intentions][intents]|500 par application : 499 intentions personnalisées et l’intention _Aucune_ obligatoire.<br>L’application [dispatch](https://aka.ms/dispatch-tool) a 500 sources de dispatch correspondantes.|
| [Répertorier des entités](./luis-concept-entity-types.md) | Parent : 50, Enfant : 20 000 éléments. Le nom canonique est *nombre maximum de caractère par défaut. Les valeurs synonymes n’ont aucune restriction de longueur. |
| [Entités de machine-learning + rôles](./luis-concept-entity-types.md) :<br> composite,<br>simple,<br>rôles d’entité|Limite de 100 entités parentes ou de 330 entités, selon la limite que l’utilisateur atteint en premier. Un rôle est comptabilisé comme une entité pour les besoins de cette limite. Un exemple est un composite avec une entité simple qui possède 2 rôles : 1 composite + 1 simple + 2 rôles = 4 des 330 entités.<br>Les sous-entités peuvent avoir un maximum de 5 niveaux d’imbrication.|
|Modèle en tant que fonctionnalité| Nombre maximum de modèles pouvant être utilisés en tant que fonctionnalité sur un modèle spécifique pour avoir 10 modèles. Nombre maximum de listes d’expressions utilisées en tant que fonctionnalité sur un modèle spécifique pour avoir 10 listes d’expressions.|
| [Préversion - Entités de liste dynamique](./luis-migration-api-v3.md)|2 listes de 1 000 environ par demande pour interroger le point de terminaison de prédiction|
| [Modèles](luis-concept-patterns.md)|500 modèles par application.<br>La longueur maximale du modèle est de 400 caractères.<br>3 entités Pattern.any par modèle<br>2 textes facultatifs maximum imbriqués dans le modèle|
| [Pattern.any](./luis-concept-entity-types.md)|100 par application, 3 entités pattern.any par modèle |
| [Liste d’expressions][phrase-list]|500 listes d’expressions. 10 listes d’expressions globales liées au modèle en tant que limite de fonctionnalité. La liste d’expressions non interchangeables comprend un maximum de 5 000 expressions. La liste d’expressions interchangeables comprend un maximum de 50 000 expressions. Nombre maximum d’expressions au total par application de 500 000 expressions.|
| [Entités prédéfinies](./howto-add-prebuilt-models.md) | aucune limite|
| [Entité d’expression régulière](./luis-concept-entity-types.md)|20 entités<br>500 caractères maximum par modèle d’entité d’expression régulière|
| [Rôles](./luis-concept-entity-types.md)|300 rôles par application. 10 rôles par entité|
| [Énoncé][utterances] | 500 caractères<br><br>Si vous avez un texte d’une longueur supérieure à cette limite de caractères, vous devez segmenter l’énoncé avant de l’entrer dans LUIS afin de recevoir les réponses d’intention individuelles propres à chaque segment. Vous pouvez utiliser des arrêts évidents, tels que les signes de ponctuation et les pauses marquées dans le discours.|
| [Exemples d’énoncés][utterances] | 15 000 par application, il n’existe aucune limite du nombre d’énoncés par intention<br><br>Si vous devez entraîner l’application avec plus d’exemples, utilisez une approche de modèle [dispatch](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch). Vous entraînez les applications LUIS individuelles (appelées « applications enfants » par rapport à l’application de distribution parente) avec une ou plusieurs intentions, puis vous entraînez une application de distribution qui échantillonne les énoncés de chaque application LUIS enfant pour diriger la demande de prédiction vers l’application enfant appropriée. |
| [Versions](./luis-concept-app-iteration.md)| 100 versions par application |
| [Nom de version][luis-how-to-manage-versions] | 128 caractères |

\* 50 caractères par défaut maximum.

<a name="intent-and-entity-naming"></a>

## <a name="name-uniqueness"></a>Unicité du nom

Les noms d’objet doivent être uniques par rapport aux autres objets du même niveau.

|Objets|Restrictions|
|--|--|
|Intention, entité|Les noms des intentions et des entités doivent être uniques dans la version d’une application.|
|Composants de l’entité ML|Tous les composants de l’entité de machine-learning (entités enfants) doivent être uniques au sein de cette entité pour les composants du même niveau.|
|Fonctionnalités | Toutes les fonctionnalités nommées, telles que les listes d’expressions, doivent être uniques au sein d’une version d’application.|
|Rôles d’entité|Tous les rôles d’une entité ou d’un composant d’entité doivent être uniques lorsqu’ils se trouvent au même niveau d’entité (parent, enfant, petit-enfant, etc.).|

## <a name="object-naming"></a>Attribution de noms aux objets

N’utilisez pas les caractères suivants dans les noms suivants.

|Object|Exclure des caractères|
|--|--|
|Noms d’intention, d’entité et de rôle|`:`<br>`$` <br> `&`|
|Nom de version|`\`<br> `/`<br> `:`<br> `?`<br> `&`<br> `=`<br> `*`<br> `+`<br> `(`<br> `)`<br> `%`<br> `@`<br> `$`<br> `~`<br> `!`<br> `#`|

## <a name="resource-usage-and-limits"></a>Utilisation et limites des ressources

Language Understanding possède des ressources distinctes, un type pour la création et un type pour l'interrogation du point de terminaison de prédiction. Pour en savoir plus sur les différences entre les types de clés, voir [Clés de point de terminaison de création et de prédiction de requête dans LUIS](luis-how-to-azure-subscription.md).

<a name="key-limits"></a>

### <a name="authoring-resource-limits"></a>Limites des ressources de création

Utilisez le _genre_, `LUIS.Authoring`, lors du filtrage des ressources dans le portail Azure. LUIS limite à 500 applications par ressource de création Azure.

|Ressource de création|TPS de création|
|--|--|
|Starter|1 000 000/mois, 5/seconde|
|F0 - Niveau gratuit |1 000 000/mois, 5/seconde|

* TPS = Transactions par seconde

[Apprenez-en davantage sur la tarification.][pricing]

### <a name="query-prediction-resource-limits"></a>Interroger les limites des ressources de prédiction

Utilisez le _genre_, `LUIS`, lors du filtrage des ressources dans le portail Azure. La ressource de point de terminaison de prédiction de requête LUIS, utilisée dans le runtime, est uniquement valide pour les requêtes de point de terminaison.

|Interroger la ressource de prédiction|TPS des requêtes|
|--|--|
|F0 - Niveau gratuit |10 000 par mois, 5/seconde|
|S0 - Niveau Standard|50/seconde|

### <a name="sentiment-analysis"></a>analyse de sentiments

L'[intégration de l'analyse des sentiments](luis-how-to-publish-app.md#enable-sentiment-analysis), qui fournit des informations sur les sentiments, est fournie sans nécessiter une autre ressource Azure.

### <a name="speech-integration"></a>Intégration du Speech

L'[intégration du Speech](../speech-service/how-to-recognize-intents-from-speech-csharp.md) fournit 1 000 requêtes de point de terminaison par coût unitaire.

[Apprenez-en davantage sur la tarification.][pricing]

## <a name="keyboard-controls"></a>Commandes du clavier

|Entrée de clavier | Description |
|--|--|
|Contrôle + E|bascule entre les jetons et les entités dans la liste d’énoncés|

## <a name="website-sign-in-time-period"></a>Période de connexion au site web

Votre accès de connexion est valable **60 minutes**. Une fois ce délai expiré, le message d’erreur suivant s’affichera. Vous devez vous reconnecter.

[luis-get-started-create-app]: ./luis-get-started-create-app.md
[batch-testing]: ./luis-concept-test.md#batch-testing
[intents]: ./luis-concept-intent.md
[phrase-list]: ./luis-concept-feature.md
[utterances]: ./luis-concept-utterance.md
[luis-how-to-manage-versions]: ./luis-how-to-manage-versions.md
[pricing]: https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/
<!-- TBD: fix this link -->
[speech-to-intent-pricing]: https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/
