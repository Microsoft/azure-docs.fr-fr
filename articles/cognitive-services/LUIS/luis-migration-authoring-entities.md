---
title: Migrer vers une entité issue du Machine Learning v3
description: La création v3 offre un nouveau type d’entité, l’entité issue du Machine Learning, ainsi que la possibilité d’ajouter des relations à l’entité issue du Machine Learning et à d’autres entités ou fonctionnalités de l’application.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/08/2020
ms.openlocfilehash: 667226770d25ef1687420b1c13bc71863f987e33
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91324686"
---
# <a name="migrate-to-v3-authoring-entity"></a>Migrer vers l’entité de création v3

La création v3 offre un nouveau type d’entité, l’entité issue du Machine Learning, ainsi que la possibilité d’ajouter des relations à l’entité issue du Machine Learning et à d’autres entités ou fonctionnalités de l’application.

## <a name="entities-are-decomposable-in-v3"></a>Entités décomposables dans la v3

Les entités créées avec les API de création v3, soit à l’aide des [API](https://westeurope.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview) soit avec le portail, permettent de créer un modèle d’entité en couche avec un parent et des enfants. Le parent est connu sous le nom d’**entité issue du Machine Learning** et les enfants sont appelés **sous-entités** de l’entité issue du Machine Learning.

Chaque sous-entité est également une entité issue du Machine Learning mais avec les options de configuration supplémentaires des fonctionnalités.

* Les **fonctionnalités requises** sont des règles qui garantissent qu’une entité est extraite lorsqu’elle correspond à une fonctionnalité. La règle est définie par la fonctionnalité requise vers le modèle :
    * [Entité prédéfinie](luis-reference-prebuilt-entities.md)
    * [Entité d’expression régulière](reference-entity-regular-expression.md)
    * [Entité de liste](reference-entity-list.md)

## <a name="how-do-these-new-relationships-compare-to-v2-authoring"></a>Comparaison entre ces nouvelles relations et la création v2

La création v2 offrait des entités hiérarchiques et composites, ainsi que des rôles et des fonctionnalités pour accomplir cette même tâche. Étant donné que les entités, les fonctionnalités et les rôles n’étaient pas explicitement liés les uns aux autres, il était difficile de comprendre comment LUIS impliquait les relations lors de la prédiction.

Avec la v3, la relation est explicite et conçue par les auteurs de l’application, qui peuvent ainsi :

* voir visuellement comment LUIS prédit ces relations, dans les exemples d’énoncés ;
* tester ces relations avec le [volet de test interactif](luis-interactive-test.md) ou sur le point de terminaison ;
* utiliser ces relations dans l’application cliente, au moyen d’un [objet .json](reference-entity-machine-learned-entity.md) bien structuré, nommé et imbriqué.

## <a name="planning"></a>Planification

Prenez en compte les points suivants dans votre stratégie de migration :

* Sauvegardez votre application LUIS, puis effectuez la migration sur une application distincte. Le fait de disposer d’une application v2 et d’une application v3 disponibles en même temps permet de valider les modifications requises et l’impact sur les résultats de prédiction.
* Capturez les métriques de réussite des prédictions actuelles.
* Capturez les informations actuelles du tableau de bord sous la forme d’un instantané de l’état de l’application.
* Passez en revue les intentions, les entités, les listes d’expressions, les modèles et les tests par lot existants.
* Les éléments suivants peuvent être migrés **sans modification** :
    * Intentions
    * Entités
        * Entité d’expression régulière
        * Entité de liste
    * Fonctionnalités
        * Liste d’expressions
* Les éléments suivants doivent être migrés **avec modifications** :
    * Entités
        * Entité hiérarchique
        * Entité composite
    * Rôle (ils ne peuvent être appliqués qu’à une entité (parente) issue du Machine Learning, et non à des sous-entités)
    * Tests par lots et modèles qui utilisent les entités hiérarchiques et composites

Lorsque vous concevez votre plan de migration, laissez-vous du temps pour passer en revue les entités issues du Machine Learning finales, une fois que toutes les entités hiérarchiques et composites ont été migrées. Bien qu’une migration directe fonctionne, après modification et vérification de vos résultats de test par lot, ainsi que du JSON de prédiction, le JSON plus unifié est susceptible de vous amener à apporter des modifications afin que les informations finales fournies à l’application côté client soient organisées différemment. Le principe est similaire à la refactorisation du code et doit être traité suivant le processus de révision de votre organisation.

Si vous ne disposez pas de tests par lots pour votre modèle v2 et que vous migrez les tests par lots vers le modèle v3, vous ne pourrez pas valider l’impact de la migration sur les résultats de prédiction du point de terminaison.

## <a name="migrating-from-v2-entities"></a>Migration à partir d’entités v2

Lorsque vous commencez à migrer vers le modèle de création v3, réfléchissez à la façon dont vous passerez à l’entité issue du Machine Learning, à ses sous-entités et ses fonctionnalités.

Le tableau suivant présente les entités qui doivent migrer d’une conception d’entité v2 à v3.

|Type d’entité de création v2|Type d’entité de création v3|Exemple|
|--|--|--|
|Entité composite|Entité issue du Machine Learning|[En savoir plus](#migrate-v2-composite-entity)|
|Entité hiérarchique|Rôle de l’entité issue du Machine Learning|[En savoir plus](#migrate-v2-hierarchical-entity)|

## <a name="migrate-v2-composite-entity"></a>Migration d’une entité composite v2

Chacun des enfants de l’entité composite v2 doit être représenté par une sous-entité de l’entité issue du Machine Learning v3. Si l’enfant composite est une entité prédéfinie, une expression régulière ou une entité de liste, il doit être appliqué sous forme de fonctionnalité requise sur la sous-entité.

Considérations à prendre en compte pour planifier la migration d’une entité composite vers une entité issue du Machine Learning :
* Les entités enfants ne sont pas utilisables dans des modèles.
* Les entités enfants ne sont plus partagées.
* Les entités enfants doivent être étiquetées si elles n’étaient pas issues du Machine Learning à l’origine.

### <a name="existing-features"></a>Fonctionnalités existantes

Toute liste d’expressions utilisée pour amplifier les mots dans l’entité composite doit être appliquée comme fonctionnalité à l’entité issue du Machine Learning (parent), à l’entité de sous-entité (enfant) ou à l’intention (si la liste d’expressions ne s’applique qu’à une seule intention). Planifiez l’ajout de la fonctionnalité à l’entité qu’elle devrait améliorer le plus. N’ajoutez pas la fonctionnalité de façon générique à l’entité issue du Machine Learning (parent), si elle augmente considérablement la prédiction d’une sous-entité (enfant).

### <a name="new-features"></a>Nouvelles fonctionnalités

Dans la création v3, ajoutez une étape de planification pour évaluer les entités comme des fonctionnalités possibles de toutes les entités et intentions.

### <a name="example-entity"></a>Exemple d’entité

Il s’agit simplement d’un exemple d’entité. La migration de votre propre entité est susceptible d’impliquer d’autres considérations.

Imaginez une entité composite v2 pour la modification d’une commande `order` de pizza qui utilise :
* une entité prédéfinie datetimeV2 pour le délai de livraison ;
* une liste d’expressions pour améliorer certains mots, comme pizza, tarte, croûte et garniture ;
* une entité de liste pour détecter les garnitures, par exemple les champignons, les olives, les lardons.

Voici un exemple d’énoncé pour cette entité :

`Change the toppings on my pie to mushrooms and delivery it 30 minutes later`

Le tableau suivant illustre la migration :

|Modèles v2|Modèles v3|
|--|--|
|Parent : entité composant nommée `Order`|Parent : entité issue du Machine Learning nommée `Order`|
|Enfant : entité prédéfinie datetimeV2|* Migrer une entité prédéfinie sur une nouvelle application.<br>* Ajouter une fonctionnalité requise sur le parent de l’entité prédéfinie datetimeV2.|
|Enfant : entité de liste pour les garnitures|* Migrer une entité de liste sur une nouvelle application.<br>* Ajouter ensuite une fonctionnalité requise sur le parent de l’entité de liste.|


## <a name="migrate-v2-hierarchical-entity"></a>Migrer une entité hiérarchique v2

Dans la création v2, une entité hiérarchique était fournie avant les rôles existants dans LUIS. Les deux répondaient au même objectif d’extraction d’entités en fonction de l’utilisation du contexte. Si vous avez des entités hiérarchiques, vous pouvez les considérer comme des entités simples associées à des rôles.

Dans la création v3 :
* Un rôle peut être appliqué à l’entité issue du Machine Learning (parent).
* Un rôle ne peut pas être appliqué à des sous-entités.

Il s’agit simplement d’un exemple d’entité. La migration de votre propre entité est susceptible d’impliquer d’autres considérations.

Prenons l’exemple d’une entité hiérarchique v2 pour la modification d’une commande `order` de pizza :
* où chaque enfant détermine soit une garniture d’origine, soit la garniture finale

Voici un exemple d’énoncé pour cette entité :

`Change the topping from mushrooms to olives`

Le tableau suivant illustre la migration :

|Modèles v2|Modèles v3|
|--|--|
|Parent : entité composant nommée `Order`|Parent : entité issue du Machine Learning nommée `Order`|
|Enfant : entité hiérarchique avec garniture de pizza d’origine et finale|* Ajouter un rôle à `Order` pour chaque garniture.|

## <a name="api-change-constraint-replaced-with-required-feature"></a>Contrainte de modification d’API remplacée par une fonctionnalité requise

Cette modification a été effectuée en mai 2020 lors de la conférence //Build et s’applique uniquement aux API de création v3 dans lesquelles une application utilise une fonctionnalité contrainte. Si vous passez de la création v2 à la création v3, ou si vous n’avez pas utilisé de fonctionnalités contraintes v3, ignorez cette section.

**Fonctionnalité** : capacité à exiger une entité existante comme fonctionnalité d’un autre modèle et à extraire ce modèle si l’entité est détectée. La fonctionnalité n’a pas changé mais l’API et la terminologie ont changé.

|Ancienne terminologie|Nouvelle terminologie|
|--|--|
|`constrained feature`<br>`constraint`<br>`instanceOf`|`required feature`<br>`isRequired`|

#### <a name="automatic-migration"></a>Migration automatique

À partir du **19 juin 2020**, vous ne pourrez plus créer des contraintes par programme à l’aide de l’ancienne API de création qui exposait cette fonctionnalité.

Toutes les fonctionnalités contraintes existantes seront automatiquement migrées vers l’indicateur de fonctionnalité requise. Aucune modification par programme de votre API de prédiction n’est requise, ni aucune modification consécutive sur la qualité de la précision de la prédiction.

#### <a name="luis-portal-changes"></a>Modifications apportées au portail LUIS

Le portail LUIS en préversion référençait cette fonctionnalité comme une **contrainte**. Le portail LUIS actuel la désigne comme une **fonctionnalité requise**.

#### <a name="previous-authoring-api"></a>Ancienne API de création

Cette fonctionnalité était appliquée dans l’ **[API Création d’un enfant d’entité](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5d86cf3c6a25a45529767d77)** de création en préversion comme élément de la définition d’une entité, en utilisant la propriété `instanceOf` de l’enfant d’une entité :

```json
{
    "name" : "dayOfWeek",
    "instanceOf": "datetimeV2",
    "children": [
        {
           "name": "dayNumber",
           "instanceOf": "number",
           "children": []
        }
    ]
}
```

#### <a name="new-authoring-api"></a>Nouvelle API de création

Cette fonctionnalité est désormais appliquée avec l’ **[API Ajout d’une relation de fonctionnalité d’entité](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5d9dc1781e38aaec1c375f26)** en utilisant les propriétés `featureName` et `isRequired`. La valeur de la propriété `featureName` est le nom du modèle.

```json
{
    "featureName": "YOUR-MODEL-NAME-HERE",
    "isRequired" : true
}
```


## <a name="next-steps"></a>Étapes suivantes

* [Ressources pour les développeurs](developer-reference-resource.md)
