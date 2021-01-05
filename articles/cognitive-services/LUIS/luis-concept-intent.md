---
title: Intentions et entités - LUIS
titleSuffix: Azure Cognitive Services
description: Une intention unique représente une tâche ou une action que l’utilisateur souhaite effectuer. Il s’agit d’une finalité ou d’un objectif exprimé dans l’énoncé d’un utilisateur. Définissez un ensemble d’intentions qui correspondent aux actions que les utilisateurs souhaitent effectuer dans votre application.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 10/10/2019
ms.openlocfilehash: bf5ab7361c84fb787366c7c361829e52362fe427
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025906"
---
# <a name="intents-in-your-luis-app"></a>Intentions dans votre application LUIS

Une intention représente une tâche ou une action que l’utilisateur souhaite effectuer. Il s’agit d’une finalité ou d’un objectif exprimé dans [l’énoncé](luis-concept-utterance.md) d’un utilisateur.

Définissez un ensemble d’intentions qui correspondent aux actions que les utilisateurs souhaitent effectuer dans votre application. Par exemple, une application de voyage définit plusieurs intentions :

Intentions de l’application de voyage   |   Exemples d’énoncés   |
------|------|
 RéserverVol     |   « Réserve-moi un vol pour Rio la semaine prochaine » <br/> « Emmène-moi à Rio le 24 » <br/> « J’ai besoin d’un billet d’avion pour Rio de Janeiro dimanche prochain »    |
 Salutations     |   « Salut » <br/>« Bonjour » <br/>« Bonsoir »  |
 BulletinMétéo | « Quel temps fait-il à Boston ? » <br/> « Montre-moi les prévisions pour ce week-end » |
 None         | « Trouve-moi une recette de cookies »<br>« Les Lakers ont-ils gagné ? » |

Toutes les applications s’accompagnent de l’intention prédéfinie « [None](#none-intent) », qui correspond à l’intention de secours.

## <a name="prebuilt-domains-provide-intents"></a>Intentions fournies par les domaines prédéfinis
En plus des intentions que vous définissez, vous pouvez utiliser les intentions prédéfinies d’un des [domaines prédéfinis](./howto-add-prebuilt-models.md).

## <a name="return-all-intents-scores"></a>Retourner le score de toutes les intentions
Vous attribuez une seule intention à un énoncé. Lorsque LUIS reçoit un énoncé sur le point de terminaison, il retourne par défaut la meilleure intention pour cet énoncé.

Si vous souhaitez connaître les scores de toutes les intentions de l’énoncé, vous pouvez ajouter un indicateur sur la chaîne de requête de l’API de prédiction.

|Version de l’API de prédiction|Indicateur|
|--|--|
|V2|`verbose=true`|
|V3|`show-all-intents=true`|

## <a name="intent-compared-to-entity"></a>Différence entre intention et entité
L’intention correspond à l’action que l’application doit effectuer pour l’utilisateur ; elle s’appuie sur la totalité de l’énoncé. Un énoncé ne peut avoir qu’une seule meilleure intention, mais il peut comporter de nombreuses entités.

<a name="how-do-intents-relate-to-entities"></a>

Créez une intention lorsque le _but_ de l’utilisateur doit déclencher une action dans votre application cliente, comme un appel à la fonction checkweather(). Ensuite, créez des entités afin de représenter les paramètres requis pour exécuter l’action.

|Intentionnel   | Entité | Exemple d’énoncé   |
|------------------|------------------------------|------------------------------|
| BulletinMétéo | { "type": "location", "entity": "seattle" }<br>{ "type": "builtin.datetimeV2.date","entity": "demain","resolution":"2018-05-23" } | « Quel temps fait-il à `Seattle` `tomorrow` ? » |
| BulletinMétéo | { "type": "date_range", "entity": "ce weekend" } | Montre-moi les prévisions pour `this weekend` |
||||

## <a name="prebuilt-domain-intents"></a>Intentions de domaine prédéfini

Les [domaines prédéfinis](./howto-add-prebuilt-models.md) fournissent des intentions avec des énoncés.

## <a name="none-intent"></a>Intention None

L’intention **None** est créée mais laissée vide intentionnellement. L’intention **None** est requise ; elle n’est ni supprimable ni renommable. Remplissez-la avec des énoncés extérieurs à votre domaine.

L’intention **None** est l’intention de secours, importante dans chaque application, et doit avoir 10 % des énoncés totaux. Elle sert à montrer à LUIS des expressions qui ne sont pas importantes dans le domaine d’application (sujet). Si vous n’ajoutez pas d’énoncés à l’intention **None**, LUIS force un énoncé extérieur au domaine dans l’une des intentions du domaine. Cela fausse les scores de prédiction en indiquant à LUIS la mauvaise intention pour l’énoncé.

Lorsqu’un énoncé est annoncé comme ayant l’intention None, l’application cliente peut poser des questions supplémentaires ou proposer un menu afin d’orienter l’utilisateur vers des choix valides.

## <a name="negative-intentions"></a>Intentions négatives
Si vous souhaitez identifier les intentions négatives et positives, par exemple, « je **veux** une voiture » et « je **ne veux pas** de voiture », vous pouvez créer deux intentions (une positive et une négative) et ajouter les énoncés correspondant à chacune. Vous pouvez également créer une intention unique et marquer les deux termes, positif et négatif, comme une entité.

## <a name="intents-and-patterns"></a>Intentions et les modèles

Si vous avez des exemples d’énoncé, qui peuvent être définis en partie ou intégralement sous la forme d’une expression régulière, envisagez d’utiliser [l’entité d’expression régulière](luis-concept-entity-types.md#regular-expression-entity) appariée à un [modèle](luis-concept-patterns.md).

L’utilisation d’une entité de l’expression régulière garantit l’extraction de données, de sorte que le modèle est mis en correspondance. Les critères spéciaux garantissent qu'une intention exacte est renvoyée.

## <a name="intent-balance"></a>Équilibre des intentions
Les intentions du domaine d’application doivent avoir un équilibre entre les énoncés sur chaque objectif. Évitez d’avoir une intention avec 10 énoncés et une autre avec 500 énoncés. Ce n’est pas équilibré. Si vous vous trouvez dans cette situation, vérifiez l’intention comportant 500 énoncés pour voir si la majeure partie d’entre eux peut être réorganisée en un [modèle](luis-concept-patterns.md).

L’intention **None** n’est pas prise en compte dans la recherche d’équilibre. Cette intention doit contenir 10 % du nombre total d’énoncés de l’application.

## <a name="intent-limits"></a>Limites des intentions
Consultez les [limites](luis-limits.md#model-boundaries) pour savoir combien d’intentions vous pouvez ajouter à un modèle.

### <a name="if-you-need-more-than-the-maximum-number-of-intents"></a>Dépasser le nombre maximal d’intentions
Demandez-vous d’abord si votre système utilise trop d’intentions.

### <a name="can-multiple-intents-be-combined-into-single-intent-with-entities"></a>Combiner plusieurs intentions en une seule avec des entités
Des intentions trop proches peuvent être plus difficiles à distinguer pour LUIS. Elles doivent être suffisamment variées pour capturer les principales tâches demandées par l’utilisateur, mais il n’est pas nécessaire de saisir la totalité des chemins de votre code. Par exemple, RéserverVol et ServiceClientVol peuvent être des intentions distinctes dans une application de voyage, mais RéserverVolInternational et RéserverVolIntérieur sont trop proches. Si votre système a besoin de les distinguer, utilisez les entités ou une logique autre que des intentions.

### <a name="dispatcher-model"></a>Modèle du répartiteur
Découvrez comment combiner des applications LUIS et QnA Maker avec le [modèle de répartition](luis-concept-enterprise.md#when-you-need-to-combine-several-luis-and-qna-maker-apps).

### <a name="request-help-for-apps-with-significant-number-of-intents"></a>Aide pour les applications comportant de nombreuses intentions
S’il ne vous est pas possible de réduire le nombre d’intentions ou de les diviser en plusieurs applications, contactez le support. Si votre abonnement Azure comprend des services de support, contactez le [support technique Azure](https://azure.microsoft.com/support/options/).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [entités](luis-concept-entity-types.md) (mots importants pertinents pour les intentions).
* Découvrir comment [ajouter et gérer les intentions](luis-how-to-add-intents.md) dans une application LUIS.
* Voir les [meilleures pratiques](luis-concept-best-practices.md) en matière d’intention.