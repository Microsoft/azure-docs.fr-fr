---
title: Ajouter des entités - LUIS
description: Créez des entités pour extraire les données clés des énoncés de l'utilisateur dans les applications LUIS (Language Understanding). Les données d’entité extraites sont utilisées par l’application cliente pour satisfaire aux demandes des clients.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/17/2020
ms.openlocfilehash: c5c6836c2d68036bf2b9c5abe191943537349b8d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91540963"
---
# <a name="add-entities-to-extract-data"></a>Ajouter des entités pour extraire des données

Créez des entités pour extraire les données clés des énoncés de l'utilisateur dans les applications LUIS (Language Understanding). Les données d’entité extraites sont utilisées par votre application cliente pour satisfaire aux demandes des clients.

L’entité représente un mot ou une phrase dans l’énoncé que vous souhaitez extraire. Les entités décrivent des informations relatives à l’intention, et sont parfois essentiels pour que votre application effectue sa tâche. Vous pouvez créer des entités lorsque vous ajoutez un exemple d’énoncé à une intention (et avant ou après).

## <a name="plan-entities-then-create-and-label"></a>Planifier des entités, puis créer et étiqueter

Il est possible de créer des entités de machine-learning à partir des exemples d’énoncés ou à partir de la page **Entités**.

En règle générale, consacrer du temps à la planification des entités avant de créer une entité de machine-learning dans le portail se révèle une bonne pratique. Créez ensuite l’entité de machine-learning à partir de l’exemple d’énoncé avec tous les détails dans les sous-entités et fonctionnalités que vous connaissez à ce moment-là. Le [tutoriel sur les entités décomposables](tutorial-machine-learned-entity.md) montre comment utiliser cette méthode.

Dans le cadre de la planification des entités, vous savez peut-être déjà que vous avez besoin d’entités de correspondance à du texte (comme des entités prédéfinies, des entités d’expression régulière ou des entités de liste). Vous pouvez les créer à partir de la page **Entités** avant de les étiqueter dans les exemples d’énoncés.

Lors de l’étiquetage, vous pouvez étiqueter des entités individuelles, puis créer une entité de machine-learning parent. Vous pouvez aussi commencer avec une entité de machine-learning parent et la décomposer en entités enfant.

> [!TIP]
>Étiquetez tous les mots pouvant indiquer une entité, même si ces mots ne sont pas utilisés lors de l’extraction dans l’application cliente.

## <a name="when-to-create-an-entity"></a>Quand créer une entité

Après avoir planifié vos entités, vous devez créer vos entités et sous-entités de machine-learning. Il peut être nécessaire d’ajouter des entités prédéfinies ou des entités de texte correspondant à la machine pour fournir des fonctionnalités à vos entités de machine-learning. Ces opérations doivent toutes être effectuées avant l’étiquetage.

Une fois que vous avez commencé à étiqueter des exemples d’énoncés, vous pouvez créer des entités acquises sur ordinateur ou étendre les entités de liste.

Utilisez le tableau suivant pour comprendre où créer ou ajouter chaque type d’entité à l’application.

|Type d'entité|Où créer l’entité dans le portail LUIS|
|--|--|
|Entité de machine-learning|Détail de l’entité ou de l’intention|
|Entité de liste|Détail de l’entité ou de l’intention|
|Entité d’expression régulière|Entités|
|Entité Pattern.any|Entités|
|Entité prédéfinie|Entités|
|Détail du domaine prédéfini|Entités|

Vous pouvez créer toutes les entités à partir de la page **Entités**, ou créer deux entités dans le cadre de l’étiquetage de l’entité dans l’exemple d’énoncé de la page de **détail de l’intention**. Vous pouvez uniquement _étiqueter_ une entité dans un exemple d’énoncé à partir de la page de **détail de l’intention**.



## <a name="how-to-create-a-new-custom-entity"></a>Comment créer une entité personnalisée

Ce processus fonctionne pour les entités acquises sur ordinateur, les entités de liste et les entités d’expression régulière.

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Sélectionnez la page **Entités**.
1. Sélectionnez **+ Créer**, puis le type d’entité.
1. Poursuivez la configuration de l’entité, puis sélectionnez **Créer** une fois que vous avez terminé.

## <a name="create-a-machine-learned-entity"></a>Créez une entité acquise sur ordinateur

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Dans la section **Build**, sélectionnez **Entités** dans le panneau gauche, puis **+ Créer**.
1. Dans la boîte de dialogue **Créer un type d’entité**, entrez le nom de l’entité et sélectionnez **Acquise sur ordinateur**. Pour ajouter des sous-entités, sélectionnez **Ajouter une structure**. Sélectionnez **Create** (Créer).

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de la création d’une entité acquise sur ordinateur.](media/add-entities/machine-learned-entity-with-structure.png)

1. Dans **ajouter des sous-entités**, ajoutez une sous-entité en sélectionnant le **+** sur la ligne de l’entité parent.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran d’ajout de sous-entités.](media/add-entities/machine-learned-entity-with-subentities.png)

1. Sélectionnez **Créer** pour terminer le processus de création.

## <a name="add-a-feature-to-a-machine-learned-entity"></a>Ajouter une fonctionnalité à une entité acquise sur ordinateur

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Dans la section **Build**, sélectionnez **Entités** dans le volet gauche, puis sélectionnez l’entité acquise par ordinateur.
1. Ajoutez une fonctionnalité en sélectionnant **+ Ajouter une fonctionnalité** sur la ligne de l’entité ou de la sous-entité.
1. Sélectionnez parmi les entités existantes et les listes d’expressions.
1. Si l’entité doit être extraite uniquement si la fonctionnalité est trouvée, sélectionnez l’astérisque, `*` pour cette fonctionnalité.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de l’ajout de la fonctionnalité à l’entité.](media/add-entities/machine-learned-entity-schema-with-features.png)

## <a name="create-a-regular-expression-entity"></a>Créer une entité d’expression régulière

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Dans la section **Build**, sélectionnez **Entités** dans le panneau gauche, puis **+ Créer**.

1. Dans la boîte de dialogue **Créer un type d’entité**, entrez le nom de l’entité et sélectionnez **RegEx**, entrez l’expression régulière dans le champ **Regex** et sélectionnez **créer**.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de la création d’une entité d’expression régulière.](media/add-entities/add-regular-expression-entity.png)


<a name="add-list-entities"></a>

## <a name="create-a-list-entity"></a>Créer une entité de liste

Les entités de liste représentent un ensemble fixe et fermé de mots liés les uns aux autres. Alors que vous pouvez, en tant qu’auteur, modifier la liste, LUIS ne peut l’augmenter ni la réduire. Vous pouvez également effectuer une importation vers une entité de liste existante à l’aide d’un [format d’entité de liste .json](reference-entity-list.md#example-json-to-import-into-list-entity).

La liste suivante montre le nom canonique et les synonymes.

|Couleur : nom d’élément de liste|Couleur : synonymes|
|--|--|
|Rouge|carmin, sang, cerise, camion de pompier|
|Bleu|ciel, cobalt|
|Vert|émeraude, olive|

Utilisez la procédure pour créer une entité de liste. Une fois l’entité de liste créée, vous n’avez pas besoin d’étiqueter les exemples d’énoncés dans une intention. Les éléments de liste et les synonymes sont mis en correspondance à l’aide du texte exact.
1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Dans la section **Build**, sélectionnez **Entités** dans le panneau gauche, puis **+ Créer**.

1. Dans la boîte de dialogue **Créer un type d’entité**, entrez le nom de l’entité, comme `Colors`, puis sélectionnez **Liste**.
1. Dans la boîte de dialogue **Créer une entité de liste**, dans **Ajouter une nouvelle sous-liste**, entrez le nom de l’élément de liste, comme `Green`, puis ajoutez des synonymes.

    > [!div class="mx-imgBorder"]
    > ![Créez une liste de couleurs en tant qu’entité de liste dans la page de détail de l’entité.](media/how-to-add-entities/create-list-entity-of-colors.png)

1. Quand vous avez terminé d’ajouter des éléments de liste et des synonymes, sélectionnez **Créer**.

    Quand vous avez terminé d’apporter un groupe de modifications à l’application, n’oubliez pas d’**entraîner** l’application. N’entraînez pas l’application après une seule modification.

    > [!NOTE]
    > Cette procédure montre comment créer et étiqueter une entité de liste à partir d’un exemple d’énoncé dans la page de **détail de l’intention**. Vous pouvez également créer la même entité à partir de la page **Entités**.

## <a name="add-a-role-for-an-entity"></a>Ajouter un rôle pour une entité

Un rôle est un sous-type nommé d’une entité, en fonction du contexte.

### <a name="add-a-role-to-distinguish-different-contexts"></a>Ajouter un rôle pour distinguer les différents contextes

Dans l’énoncé suivant, il existe deux emplacements, chacun étant spécifié sémantiquement par les mots qui l’entourent, comme `to` et `from` :

`Pick up the package from Seattle and deliver to New York City.`

Dans cette procédure, ajoutez des rôles `origin` et `destination` à une entité geographyV2 prédéfinie.
1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Dans la section **Build**, sélectionnez **Entities** dans le volet gauche.

1. Sélectionnez **+ Ajouter une entité prédéfinie**. Sélectionnez **geographyV2**, puis **Terminé**. Une entité prédéfinie est alors ajoutée à l’application.

    Si vous constatez que votre modèle, quand il comporte une entité Pattern.any, extrait mal les entités, utilisez une [liste explicite](reference-pattern-syntax.md#explicit-lists) pour corriger ce problème.

1. Sélectionnez l’entité geographyV2 prédéfinie qui vient d’être ajoutée dans la liste des entités de la page **Entités**.
1. Pour ajouter un nouveau rôle, sélectionnez **+** en regard de **Aucun rôle ajouté**.
1. Dans la zone de texte **Rôle de type**, entrez le nom du rôle `Origin`, puis appuyez sur Entrée. Ajoutez un deuxième nom de rôle `Destination`, puis appuyez sur Entrée.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran d’ajout du rôle Origin à l’entité Location](media/how-to-add-entities//add-role-to-prebuilt-geographyv2-entity.png)

    Le rôle est ajouté à l’entité prédéfinie, mais il n’est pas ajouté aux énoncés qui utilise cette entité.

### <a name="label-text-with-a-role-in-an-example-utterance"></a>Étiqueter le texte avec un rôle dans un exemple d’énoncé

> [!TIP]
> Les rôles peuvent être remplacés par des étiquettes avec des sous-entités d’une entité de machine-learning.

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Accédez à la page de détail de l’intention, qui contient des exemples d’énoncés qui utilisent le rôle.
1. Pour étiqueter avec le rôle, sélectionnez l’étiquette d’entité (ligne continue sous le texte) dans l’exemple d’énoncé, puis sélectionnez **Afficher dans le volet d’entités** dans la liste déroulante.

    > [!div class="mx-imgBorder"]
    > ![Capture d'écran sur laquelle l'élément de menu Afficher dans le volet d'entités est sélectionné.](media/add-entities/view-in-entity-pane.png)

    La palette d’entités s’ouvre à droite.

1. Sélectionnez l’entité, puis accédez au bas de la palette et sélectionnez le rôle.

    > [!div class="mx-imgBorder"]
    > ![Capture d'écran montrant où sélectionner le rôle.](media/add-entities/select-role-in-entity-palette.png)

<a name="add-pattern-any-entities"></a>
<a name="add-a-patternany-entity"></a>
<a name="create-a-pattern-from-an-utterance"></a>

## <a name="create-a-patternany-entity"></a>Créer une entité pattern.any

L’entité **Pattern.any** n’est disponible qu’avec [Modèles](luis-how-to-model-intent-pattern.md).


## <a name="do-not-change-entity-type"></a>Ne modifiez pas le type d’entité

LUIS n’autorise pas la modification du type de l’entité, car il ne sait pas quoi ajouter ou supprimer pour construire cette entité. Pour modifier le type, il est préférable de créer une nouvelle entité du bon type avec un nom légèrement différent. Ensuite, supprimez l’ancien nom d’entité étiqueté et ajoutez le nouveau nom d’entité dans chaque énoncé. Une fois tous les énoncés réétiquetés, supprimez l’ancienne entité.


## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Utiliser des modèles prédéfinis](howto-add-prebuilt-models.md)

Pour en savoir plus :
* Guide pratique pour [effectuer l’apprentissage](luis-how-to-train.md)
* Guide pratique pour [tester](luis-interactive-test.md)
* Guide pratique pour [publier](luis-how-to-publish-app.md)
* Modèles :
    * [Concepts](luis-concept-patterns.md)
    * [Syntaxe](reference-pattern-syntax.md)
* [Référentiel GitHub des entités prédéfinies](https://github.com/Microsoft/Recognizers-Text)
* [Concepts de l’extraction des données](luis-concept-data-extraction.md)



