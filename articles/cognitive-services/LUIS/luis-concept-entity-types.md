---
title: Types d’entités - LUIS
description: Une entité extrait des données d’un énoncé de l’utilisateur au moment de l’exécution de la prédiction. Un objectif secondaire _facultatif_ est d’améliorer la prédiction de l’intention ou d’autres entités à l’aide de l’entité en tant que caractéristique.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 08/06/2020
ms.openlocfilehash: 398d18642052726af4d4920443bad515ec0b5bef
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91316560"
---
# <a name="extract-data-with-entities"></a>Extraire des données avec des entités

Une entité extrait des données d’un énoncé de l’utilisateur au moment de l’exécution de la prédiction. Un objectif secondaire _facultatif_ est d’améliorer la prédiction de l’intention ou d’autres entités à l’aide de l’entité en tant que caractéristique.

Il existe plusieurs types d’entités .

* [entité machine-learning](reference-entity-machine-learned-entity.md) : entité principale. Vous devez concevoir votre schéma avec ce type d’entité avant d’utiliser d’autres entités.
* Entité non-machine-learning utilisée en tant que [fonctionnalité](luis-concept-feature.md) requise : pour les correspondances de texte exactes, les correspondances au modèle ou la détection par des entités prédéfinies.
* [Pattern.any](#patternany-entity) : pour extraire du texte de forme libre tel que des titres de livres à partir d’un [modèle](reference-entity-pattern-any.md)

Les entités de machine-learning offrent la plus vaste éventail de choix pour l’extraction de données. Les entités non-machine-learning fonctionnent par correspondance de texte et sont utilisées en tant que [fonctionnalités requises](#design-entities-for-decomposition) pour une entité ou une intention de machine-learning.

## <a name="entities-represent-data"></a>Les entités représentent les données

Les entités sont des données que vous souhaitez extraire de l’énoncé, telles que des noms, des dates, des noms de produit ou des groupes de mots significatifs. Un énoncé peut inclure plusieurs entités ou aucune. Une application cliente _peut_ avoir besoin des données pour accomplir sa tâche.

Les entités doivent être étiquetées de manière cohérente sur tous les énoncés d’entraînement pour chaque intention dans un modèle.

 Vous pouvez définir vos propres entités ou utiliser des entités prédéfinies afin de gagner du temps pour les concepts courants comme [datetimeV2](luis-reference-prebuilt-datetimev2.md), [ordinal](luis-reference-prebuilt-ordinal.md), [e-mail](luis-reference-prebuilt-email.md) et [numéro de téléphone](luis-reference-prebuilt-phonenumber.md).

|Énoncé|Entité|Données|
|--|--|--|
|Acheter 3 tickets pour New York|Nombre prédéfini<br>Destination|3<br>New York|


### <a name="entities-are-optional-but-recommended"></a>Les entités sont facultatives mais recommandées

Les [intentions](luis-concept-intent.md) sont requises mais les entités sont facultatives. Vous n’avez pas besoin de créer d’entités pour chaque concept dans votre application, mais uniquement pour celles où l’application cliente a besoin des données, ou l’entité agit comme un conseil ou un signal auprès d’une autre entité ou intention.

À mesure que votre application se développe et qu’un nouveau besoin de données est identifié, vous pouvez ajouter des entités appropriées à votre modèle LUIS par la suite.

<a name="entity-compared-to-intent"></a>

## <a name="entity-represents-data-extraction"></a>L’entité représente l’extraction de données

L’entité représente un concept de données _à l’intérieur de l’énoncé_ que vous souhaitez extraire. Une intention classe _tout l’énoncé_.

Prenons les quatre énoncés suivants :

|Énoncé|Intention prédite|Entités extraites|Explication|
|--|--|--|--|
|Aide|help|-|Rien à extraire.|
|Envoyer quelque chose|sendSomething|-|Rien à extraire. Le modèle n’a pas de fonctionnalité requise pour extraire `something` dans ce contexte, et aucun destinataire n’est indiqué.|
|Envoyer un cadeau à Bob|sendSomething|`Bob`, `present`|Le modèle extrait `Bob` en ajoutant une fonctionnalité requise d’une entité `personName` prédéfinie. Une entité de machine-learning a été utilisée pour extraire `present`.|
|Envoyer une boîte de chocolats à Bob|sendSomething|`Bob`, `box of chocolates`|Les deux éléments de données importants, `Bob` et `box of chocolates`, ont été extraits par des entités de machine-learning.|

## <a name="label-entities-in-all-intents"></a>Étiqueter les entités dans toutes les intentions

Les entités extraient les données indépendamment de l’intention prédite. Veillez à étiqueter _tous_ les exemples d’énoncés dans toutes les intentions. L’absence d’étiquette d’entité de l’intention `None` provoque une confusion, même s’il existait beaucoup plus d’énoncés de formation pour les autres intentions.

## <a name="design-entities-for-decomposition"></a>Concevez des entités pour la décomposition

Les entités de machine-learning vous permettent de concevoir le schéma de votre application pour la décomposition, en divisant un concept complexe en sous-entités.

Concevoir pour la décomposition permet à LUIS de restituer un degré élevé de résolution des entités à votre application cliente. Cela permet à votre application cliente de se concentrer sur les règles métier et de confier la résolution des données à LUIS.

Une entité de machine-learning se déclenche en fonction du contexte appris à partir d’exemples d’énoncés.

Les [**entités de machine-learning**](tutorial-machine-learned-entity.md) constituent des extracteurs de niveau supérieur. Les sous-entités sont des entités enfants d’entités de machine-learning.

## <a name="effective-machine-learned-entities"></a>Entités effectives issues du machine learning

Pour créer efficacement les entités issues du machine learning :

* Votre étiquetage doit être cohérent au niveau des intentions. Cela inclut jusqu’aux énoncés que vous fournissez dans l’intention **Aucune**, qui incluent cette entité. Dans le cas contraire, le modèle n’est pas en mesure de déterminer les séquences efficacement.
* Si vous disposez d’une entité issue du machine learning avec des sous-entités, assurez-vous que les différentes commandes et variantes de l’entité et des sous-entités sont présentées dans les énoncés étiquetés. Les exemples d’énoncés étiquetés doivent inclure toutes les formes valides, ainsi que les entités qui apparaissent, sont absentes et sont réorganisées au sein de l’énoncé.
* Vous devez éviter de surajuster les entités à un jeu très fixe. Un **surajustement** se produit lorsque le modèle ne généralise pas bien et qu’il s’agit d’un problème courant dans les modèles Machine Learning. Cela implique que l’application ne fonctionnera pas correctement sur de nouvelles données. Vous devez faire varier les exemples d’énoncés étiquetés pour que l’application puisse généraliser au-delà des exemples limités que vous fournissez. Vous devez faire varier les différentes sous-entités avec suffisamment de modifications pour que le modèle considère plus le concept que les exemples indiqués.

## <a name="effective-prebuilt-entities"></a>Entités prédéfinies efficaces

Pour créer des entités efficaces qui extraient des données courantes, telles que celles fournies par les [entités prédéfinies](luis-reference-prebuilt-entities.md), nous vous recommandons de procéder comme suit.

Améliorez l’extraction des données en apportant vos propres données à une entité en tant que fonctionnalité. Ainsi, toutes les étiquettes supplémentaires provenant de vos données apprendront le contexte de l’emplacement des noms de personnes dans votre application.

<a name="composite-entity"></a>
<a name="list-entity"></a>
<a name="patternany-entity"></a>
<a name="prebuilt-entity"></a>
<a name="regular-expression-entity"></a>
<a name="simple-entity"></a>

## <a name="types-of-entities"></a>Types d’entités

Une sous-entité d’un parent doit être une entité de machine-learning. La sous-entité peut utiliser une entité non-machine-learning en tant que [fonctionnalité](luis-concept-feature.md).

Choisissez l’entité en fonction de la façon dont les données doivent être extraites et être représentées une fois extraites.

|Type d'entité|Objectif|
|--|--|
|[**Issue de l’apprentissage automatique**](tutorial-machine-learned-entity.md)|Extrayez les données imbriquées et complexes tirées des exemples étiquetés. |
|[**Liste**](reference-entity-list.md)|Liste d’éléments et de leurs synonymes extraits avec une **correspondance exacte du texte**.|
|[**Pattern.any**](#patternany-entity)|Entité dans laquelle la détermination de la fin de l’entité est difficile à déterminer parce que l’entité est de forme libre. Disponible uniquement dans les [modèles](luis-concept-patterns.md).|
|[**Prédéfinie**](luis-reference-prebuilt-entities.md)|Déjà entraînée à extraire un type spécifique de données comme une URL ou un e-mail. Certaines de ces entités prédéfinies dans le projet open source [Recognizers-Text](https://github.com/Microsoft/Recognizers-Text). Si votre culture ou entité spécifique n’est pas encore prise en charge, vous pouvez contribuer au projet.|
|[**Expression régulière**](reference-entity-regular-expression.md)|Utilise l’expression régulière pour une **correspondance exacte du texte**.|


## <a name="extraction-versus-resolution"></a>Extraction et résolution

Les entités extraient les données à mesure que celles-ci apparaissent dans l’énoncé. Les entités ne modifient pas et ne résolvent pas les données. L’entité ne résout pas le fait que le texte soit ou non une valeur valide pour l’entité.

Il existe différents moyens de résoudre l’extraction, mais vous devez savoir que cela limite la capacité de l’application à être immunisée contre les variations et les erreurs.

Des entités de liste et d’expression régulière (correspondance de texte) peuvent être utilisées en tant que [fonctionnalités requises](luis-concept-feature.md#required-features) pour une sous-entité, et cela agit comme un filtre pour l’extraction. Vous devez l’utiliser en ayant soin de ne pas entraver la capacité de prédiction de l’application.

## <a name="extracting-contextually-related-data"></a>Extraction de données relatives au contexte

Un énoncé peut contenir au moins deux occurrences d’une entité où la signification des données est basée sur le contexte dans l’énoncé. Par exemple, un énoncé pour la réservation d’un vol a deux localisations géographiques : l’origine et la destination.

`Book a flight from Seattle to Cairo`

Les deux localisations doivent être extraites de manière à ce que l’application cliente sache le type de chaque localisation, pour mener l’achat du billet.

Pour extraire l’origine et la destination, créez deux sous-entités faisant partir de l’entité de machine-learning dans la commande de ticket. Pour chacune des sous-entités, créez une fonctionnalité requise qui utilise geographyV2.

<a name="using-component-constraints-to-help-define-entity"></a>
<a name="using-subentity-constraints-to-help-define-entity"></a>

### <a name="using-required-features-to-constrain-entities"></a>Utilisation des fonctionnalités requises pour contraindre les entités

En savoir plus sur les [fonctionnalités requises](luis-concept-feature.md)

## <a name="patternany-entity"></a>Entité Pattern.any

Une entité Pattern.any est uniquement disponible dans un [modèle](luis-concept-patterns.md).

<a name="if-you-need-more-than-the-maximum-number-of-entities"></a>
## <a name="exceeding-app-limits-for-entities"></a>Dépassement des limites d’application pour les entités

Si vous devez dépasser la [limite](luis-limits.md#model-limits), contactez le support. Pour cela, rassemblez des informations détaillées sur votre système, accédez au site web [LUIS](luis-reference-regions.md#luis-website), puis sélectionnez **Support**. Si votre abonnement Azure comprend des services de support, contactez le [support technique Azure](https://azure.microsoft.com/support/options/).

## <a name="entity-prediction-status-and-errors"></a>État de prédiction d’entité et erreurs

Le portail LUIS indique quand l’entité a une prédiction d’entité différente de celle de l’entité que vous avez sélectionnée dans un exemple d’énoncé. Ce score différent est basé sur le modèle entraîné actuel. 

:::image type="content" source="./media/luis-concept-entities/portal-entity-prediction-error.png" alt-text="Le portail LUIS indique quand l’entité a une prédiction d’entité différente de celle de l’entité que vous avez sélectionnée dans un exemple d’énoncé.":::

Le texte d’erreur est mis en surbrillance dans l’exemple d’énoncé, et l’exemple de ligne d’énoncé comporte un indicateur d’erreur à droite, sous la forme d’un triangle rouge. 

Utilisez ces informations pour résoudre les erreurs d’entité à l’aide d’un ou plusieurs des éléments suivants :
* Le texte mis en surbrillance est mal étiqueté. Pour corriger, examiner, corriger et reformer. 
* Créez une [fonctionnalité](luis-concept-feature.md) pour l’entité afin d’identifier le concept de l’entité.
* Ajoutez des [exemples d’énoncés](luis-concept-utterance.md) et étiquetez-les avec l’entité.
* [Passez en revue les suggestions d’apprentissage actif](luis-concept-review-endpoint-utterances.md) pour les énoncés reçus au point de terminaison de prédiction qui peuvent aider à identifier le concept de l’entité.

## <a name="next-steps"></a>Étapes suivantes

Découvrez les concepts relatifs aux bons [énoncés](luis-concept-utterance.md).

Consulter [Ajouter des entités](luis-how-to-add-entities.md) pour découvrir comment ajouter des entités à votre application LUIS.

Voir le [tutoriel : Extraire des données structurées d’un énoncé d’utilisateur avec des entités de machine-learning dans LUIS](tutorial-machine-learned-entity.md) pour apprendre à extraire des données structurées d’un énoncé à l’aide de l’entité de machine-learning.

