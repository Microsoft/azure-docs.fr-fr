---
title: 'Tutoriel : Modèles - LUIS'
description: Utilisez des modèles pour accroître la prédiction d’intentions et d’entités tout en fournissant moins d’exemples d’énoncés dans ce tutoriel. Le modèle est fourni sous forme d’exemple d’énoncé de modèle, qui comprend la syntaxe pour identifier les entités et le texte pouvant être ignoré.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: tutorial
ms.date: 07/06/2020
ms.openlocfilehash: 9814304aed4d7a5f307fb2179491b0fa9635fd68
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91324652"
---
# <a name="tutorial-add-common-pattern-template-utterance-formats-to-improve-predictions"></a>Tutoriel : Ajouter des formats d’énoncé de modèle courants pour améliorer les prédictions

Dans ce tutoriel, vous allez utiliser des modèles pour accroître la prédiction d’intentions et d’entités, ce qui vous permet de fournir moins d’exemples d’énoncés. Le modèle est un énoncé de modèle affecté à une intention, qui contient la syntaxe pour identifier les entités et le texte pouvant être ignoré.

**Dans ce tutoriel, vous allez découvrir comment :**

> [!div class="checklist"]
> * Créer un modèle
> * Vérifier l’amélioration des prédictions de modèle
> * Marquer le texte comme pouvant être ignoré et imbriquer dans le modèle
> * Utiliser le panneau de test pour vérifier la réussite du modèle

[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="utterances-in-intent-and-pattern"></a>Énoncés dans l’intention et le modèle

Les énoncés stockés dans l’application LUIS peuvent être de deux types :

* Exemples d’énoncés dans l’intention
* Énoncés de modèle dans le modèle

Le fait d’ajouter des énoncés de modèle en tant que modèle vous permet globalement de fournir moins d’exemples d’énoncés à une intention.

Un modèle appliqué est une correspondance de texte couplée au machine learning.  L’énoncé de modèle contenu dans le modèle, conjugué aux exemples d’énoncés de l’intention, permet à LUIS de mieux identifier les énoncés qui correspondent à l’intention.

## <a name="import-example-app-and-clone-to-new-version"></a>Importer un exemple d’application et un clone dans une nouvelle version

Utiliser les étapes suivantes :

1.  Téléchargez et enregistrez le [fichier JSON de l’application](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/luis/apps/tutorial-fix-unsure-predictions.json?raw=true).

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Importez le fichier JSON dans une nouvelle application dans le [portail LUIS](https://www.luis.ai). Dans la page **Mes applications**, sélectionnez **+ Nouvelle application de conversation**, puis choisissez **Importer en tant que JSON**. Sélectionnez le fichier que vous avez téléchargé à l’étape précédente, puis nommez l’application `Patterns tutorial`

## <a name="create-new-intents-and-their-utterances"></a>Créer de nouvelles intentions et leurs énoncés

Les deux intentions localisent les subordonnés directs du ou des managers, en fonction du texte de l’énoncé. La difficulté est que les deux intentions _signifient_ des choses différentes, mais la plupart des mots sont identiques. Seul l’ordre des mots est différent. Pour une bonne prédiction de l’intention, il faudrait un grand nombre d’exemples.

1. Dans la barre de navigation, sélectionnez **Build** (Générer).

1. Dans la page **Intents** (Intentions), sélectionnez **+ Create** (Créer) pour créer une intention.

1. Entrez `OrgChart-Manager` dans la boîte de dialogue contextuelle, puis sélectionnez **Terminé**.

    ![Créer une fenêtre contextuelle de message](media/luis-tutorial-pattern/hr-create-new-intent-popup.png)

1. Ajoutez des exemples d’énoncés à l’intention. Ces énoncés ne sont pas _exactement_ identiques, mais ils ont un modèle qui peut être extrait.

    |Exemples d’énoncés|
    |--|
    |`Who is John W. Smith the subordinate of?`|
    |`Who does John W. Smith report to?`|
    |`Who is John W. Smith's manager?`|
    |`Who does Jill Jones directly report to?`|
    |`Who is Jill Jones supervisor?`|

1. Sélectionnez **Intentions** dans le volet de navigation gauche.

1. Sélectionner **+ Create** (Créer) pour créer une intention. Entrez `OrgChart-Reports` dans la boîte de dialogue contextuelle, puis sélectionnez **Terminé**.

1. Ajoutez des exemples d’énoncés à l’intention.

    |Exemples d’énoncés|
    |--|
    |`Who are John W. Smith's subordinates?`|
    |`Who reports to John W. Smith?`|
    |`Who does John W. Smith manage?`|
    |`Who are Jill Jones direct reports?`|
    |`Who does Jill Jones supervise?`|

### <a name="caution-about-example-utterance-quantity"></a>Attention à la quantité des exemples d’énoncés

[!INCLUDE [Too few examples](../../../includes/cognitive-services-luis-too-few-example-utterances.md)]

### <a name="train-the-app-before-testing-or-publishing"></a>Entraîner l’application avant un test ou la publication

[!INCLUDE [LUIS How to Train steps](includes/howto-train.md)]

### <a name="publish-the-app-to-query-from-the-endpoint"></a>Publier l’application pour faire des requêtes à partir du point de terminaison

[!INCLUDE [LUIS How to Publish steps](includes/howto-publish.md)]

### <a name="get-intent-and-entities-from-endpoint"></a>Obtenir l’intention et les entités à partir du point de terminaison

1. [!INCLUDE [LUIS How to get endpoint first step](includes/howto-get-endpoint.md)]

1. Allez à la fin de l’URL dans la barre d’adresses, puis remplacez _YOUR_QUERY_HERE_ par : `Who is the boss of Jill Jones?`.

    ```json
    {
        "query": "Who is the boss of Jill Jones?",
        "prediction": {
            "topIntent": "OrgChart-Manager",
            "intents": {
                "OrgChart-Manager": {
                    "score": 0.326605469
                },
                "OrgChart-Reports": {
                    "score": 0.127583548
                },
                "EmployeeFeedback": {
                    "score": 0.0299124215
                },
                "MoveEmployee": {
                    "score": 0.01159851
                },
                "GetJobInformation": {
                    "score": 0.0104600191
                },
                "ApplyForJob": {
                    "score": 0.007508645
                },
                "Utilities.StartOver": {
                    "score": 0.00359402061
                },
                "Utilities.Stop": {
                    "score": 0.00336530479
                },
                "FindForm": {
                    "score": 0.002653719
                },
                "Utilities.Cancel": {
                    "score": 0.00263288687
                },
                "None": {
                    "score": 0.00238638581
                },
                "Utilities.Help": {
                    "score": 0.00226386427
                },
                "Utilities.Confirm": {
                    "score": 0.00211663754
                }
            },
            "entities": {
                "keyPhrase": [
                    "boss of Jill Jones"
                ],
                "EmployeeListEntity": [
                    [
                        "Employee-45612"
                    ]
                ],
                "$instance": {
                    "keyPhrase": [
                        {
                            "type": "builtin.keyPhrase",
                            "text": "boss of Jill Jones",
                            "startIndex": 11,
                            "length": 18,
                            "modelTypeId": 2,
                            "modelType": "Prebuilt Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ],
                    "EmployeeListEntity": [
                        {
                            "type": "EmployeeListEntity",
                            "text": "Jill Jones",
                            "startIndex": 19,
                            "length": 10,
                            "modelTypeId": 5,
                            "modelType": "List Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ]
                }
            }
        }
    }
    ```

La bonne intention principale a été prédite (`OrgChart-Manager`). Toutefois, le score n’est pas supérieur à 70 % et n’est pas suffisamment supérieur à l’intention la plus haute suivante. Utiliser des modèles pour augmenter significativement le score de l’intention en pourcentage et l’éloigner du score le plus élevé suivant.

Laissez cette deuxième fenêtre de navigation s’ouvrir. Vous utiliserez cette valeur plus loin dans le tutoriel.

## <a name="template-utterances"></a>Modèles d’énoncés
Du fait de la nature du domaine des ressources humaines, il existe plusieurs façons courantes de poser des questions sur les relations des employés dans les organisations. Par exemple :

|Énoncés|
|--|
|`Who does Jill Jones report to?`|
|`Who reports to Jill Jones?`|

Ces énoncés sont trop proches pour permettre de déterminer l’unicité contextuelle de chacun sans fournir _trop_ d’exemples d’énoncés. Si l’on ajoute un modèle pour une intention, LUIS apprend les modèles d’énoncés courants de cette intention sans qu’il soit nécessaire de fournir de nombreux exemples d’énoncés.

Quelques exemples de modèles d’énoncés pour cette intention :

|Exemples de modèles d’énoncés|signification de la syntaxe|
|--|--|
|`Who does {EmployeeListEntity} report to[?]`|interchangeable `{EmployeeListEntity}`<br>ignorer `[?]`|
|`Who reports to {EmployeeListEntity}[?]`|interchangeable `{EmployeeListEntity}`<br>ignorer `[?]`|

La syntaxe `{EmployeeListEntity}` marque le type et l’emplacement de l’entité dans l’énoncé de modèle. La syntaxe facultative, `[?]`, marque les mots ou la [ponctuation](luis-reference-application-settings.md#punctuation-normalization) facultative. LUIS établit une correspondance avec l’énoncé et ignore le texte facultatif à l’intérieur des crochets.

Bien que la syntaxe ressemble à une expression régulière, ce n’en est pas une. Seule la syntaxe d’accolade, `{}`, et de crochet, `[]`, est prise en charge. Ils peuvent être imbriqués jusqu’à deux niveaux.

Pour qu’un modèle corresponde à un énoncé, les entités au sein de l’énoncé doivent _d’abord_ correspondre aux entités du modèle d’énoncé. Cela signifie que les entités doivent avoir suffisamment d’exemples dans les exemples d’énoncés avec un fort degré de prédiction avant que les modèles avec entités réussissent. Cependant, le modèle contribue uniquement à la prédicition d’intentions, mais pas d’entités.

**Même si les modèles vous permettent de fournir moins d’exemples d’énoncés, si les entités ne sont pas détectées, le modèle ne correspond pas.**

### <a name="add-the-patterns-for-the-orgchart-manager-intent"></a>Ajouter les modèles pour l’intention OrgChart-Manager

1. Sélectionnez **Build** dans le menu supérieur.

1. Dans le volet de navigation gauche, sous **Améliorer les performances de l’application**, sélectionnez **Modèles**.

1. Sélectionnez l’intention **OrgChart-Manager**, puis entrez les modèles d’énoncés suivants :

    |Modèles d’énoncés|
    |:--|
    |`Who is {EmployeeListEntity} the subordinate of[?]`|
    |`Who does {EmployeeListEntity} report to[?]`|
    |`Who is {EmployeeListEntity}['s] manager[?]`|
    |`Who does {EmployeeListEntity} directly report to[?]`|
    |`Who is {EmployeeListEntity}['s] supervisor[?]`|
    |`Who is the boss of {EmployeeListEntity}[?]`|

    Ces énoncés de modèle comprennent l’entité **EmployeeListEntity** qui est placée entre accolades.

1. Toujours sur la page Modèles, sélectionnez l’intention **OrgChart-Reports**, puis entrez les modèles d’énoncés suivants :

    |Modèles d’énoncés|
    |:--|
    |`Who are {EmployeeListEntity}['s] subordinates[?]`|
    |`Who reports to {EmployeeListEntity}[?]`|
    |`Who does {EmployeeListEntity} manage[?]`|
    |`Who are {EmployeeListEntity} direct reports[?]`|
    |`Who does {EmployeeListEntity} supervise[?]`|
    |`Who does {EmployeeListEntity} boss[?]`|

### <a name="query-endpoint-when-patterns-are-used"></a>Point de terminaison de requête lorsque des modèles sont utilisés

Maintenant que les modèles sont ajoutés à l’application, entraînez, publiez puis interrogez l’application sur le point de terminaison du runtime de prédiction.

1. Sélectionnez **Train** (Entraîner). Une fois l’entraînement terminé, sélectionnez **Publish** (Publier), sélectionnez l’emplacement **Production**, puis **Done** (Terminé).

1. Après avoir effectué la publication, rebasculez vers l’onglet du navigateur où figure l’URL du point de terminaison.

1. Accédez à la fin de l’URL dans la barre d’adresses pour vérifier que votre requête est toujours `Who is the boss of Jill Jones?`, puis envoyez l’URL pour une nouvelle prédiction.

    ```json
    {
        "query": "Who is the boss of Jill Jones?",
        "prediction": {
            "topIntent": "OrgChart-Manager",
            "intents": {
                "OrgChart-Manager": {
                    "score": 0.999999046
                },
                "OrgChart-Reports": {
                    "score": 3.237443E-05
                },
                "EmployeeFeedback": {
                    "score": 4.364242E-06
                },
                "GetJobInformation": {
                    "score": 1.616159E-06
                },
                "MoveEmployee": {
                    "score": 7.575752E-07
                },
                "ApplyForJob": {
                    "score": 5.234157E-07
                },
                "None": {
                    "score": 3.3E-09
                },
                "Utilities.StartOver": {
                    "score": 1.26E-09
                },
                "FindForm": {
                    "score": 1.13636367E-09
                },
                "Utilities.Cancel": {
                    "score": 1.13636367E-09
                },
                "Utilities.Confirm": {
                    "score": 1.13636367E-09
                },
                "Utilities.Help": {
                    "score": 1.13636367E-09
                },
                "Utilities.Stop": {
                    "score": 1.13636367E-09
                }
            },
            "entities": {
                "keyPhrase": [
                    "boss of Jill Jones"
                ],
                "EmployeeListEntity": [
                    [
                        "Employee-45612"
                    ]
                ],
                "$instance": {
                    "keyPhrase": [
                        {
                            "type": "builtin.keyPhrase",
                            "text": "boss of Jill Jones",
                            "startIndex": 11,
                            "length": 18,
                            "modelTypeId": 2,
                            "modelType": "Prebuilt Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ],
                    "EmployeeListEntity": [
                        {
                            "type": "EmployeeListEntity",
                            "text": "Jill Jones",
                            "startIndex": 19,
                            "length": 10,
                            "modelTypeId": 5,
                            "modelType": "List Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ]
                }
            }
        }
    }
    ```

La prédiction d’intention est désormais beaucoup plus fiable et le score de la prochaine intention la plus haute est bien inférieur. Ces deux intentions ne seront pas inversées pendant l’entraînement.

### <a name="working-with-optional-text-and-prebuilt-entities"></a>Utilisation d’un texte facultatif et d’entités prédéfinies

Les énoncés de modèle précédents dans ce didacticiel possédaient quelques exemples de texte facultatifs tels que l’utilisation possessive de la lettre s, `'s`, et l’utilisation du point d’interrogation, `?`. Supposons que vous devez tenir compte des dates actuelles et futures dans l’énoncé.

Les exemples d’énoncés sont :

|Intentionnel|Exemples d’énoncés avec un texte facultatif et des entités prédéfinies|
|:--|:--|
|OrgChart-Manager|`Who was Jill Jones manager on March 3?`|
|OrgChart-Manager|`Who is Jill Jones manager now?`|
|OrgChart-Manager|`Who will be Jill Jones manager in a month?`|
|OrgChart-Manager|`Who will be Jill Jones manager on March 3?`|

Chacun de ces exemples utilise une conjugaison du verbe, `was`, `is`, `will be`, ainsi qu’une date, `March 3`, `now`, et `in a month`, que requiert LUIS pour effectuer correctement ses prédictions. Notez que les deux derniers exemples du tableau utilisent presque le même texte à l’exception de `in` et `on`.

Exemples d’énoncés de modèle pour ces informations facultatives :

|Intentionnel|Exemples d’énoncés avec un texte facultatif et des entités prédéfinies|
|:--|:--|
|OrgChart-Manager|`who was {EmployeeListEntity}['s] manager [[on]{datetimeV2}?]`|
|OrgChart-Manager|`who is {EmployeeListEntity}['s] manager [[on]{datetimeV2}?]`|


L’utilisation de la syntaxe facultative des crochets, `[]`, facilite l’ajout d’un énoncé de modèle à ce texte facultatif, ce dernier peut se voir imbriqué à un deuxième niveau, `[[]]`, et peut inclure des entités ou du texte.


**Question : Pourquoi toutes les lettres `w`, la première lettre de chaque énoncé de modèle, sont en minuscules ? Le fait qu’elles soient en majuscules ou en minuscules n’est-il pas facultatif ?** L’énoncé soumis au point de terminaison de requête par l’application cliente, est converti en minuscules. L’énoncé de modèle peut être en majuscules ou minuscules, tout comme l’énoncé du point de terminaison. La comparaison est toujours effectuée après la conversion en minuscules.

**Question : Pourquoi le nombre prédéfini ne fait pas partie de l’énoncé de modèle alors que le 3 mars est prédit à la fois comme un nombre `3` et une date `March 3` ?** L’énoncé de modèle utilise une date en fonction du contexte, soit littéralement comme dans `March 3`, ou de façon abstraite comme dans `in a month`. Une date peut contenir un nombre mais un nombre ne peut pas nécessairement être considéré comme une date. Utilisez toujours l’entité qui représente au mieux le type à retourner dans les résultats JSON de prédiction.

**Question : Qu’en est-il des énoncés incompréhensibles comme `Who will {EmployeeListEntity}['s] manager be on March 3?`.** Les conjugaisons grammaticalement différentes, comme dans le cas où le `will` et le `be` sont séparés, doivent constituer un nouvel énoncé de modèle. L’énoncé de modèle existant ne correspond pas. Bien que l’intention de l’énoncé n’ait pas changé, le placement du mot dans l’énoncé a changé. Cette modification affecte la prédiction dans LUIS. Vous pouvez [regrouper et/ou](#use-the-or-operator-and-groups) les temps des verbes pour combiner ces énoncés.

> [!CAUTION]
> **N’oubliez pas : les entités sont les premières trouvées, le modèle étant ensuite mis en correspondance.**

### <a name="add-new-pattern-template-utterances"></a>Ajouter de nouveaux énoncés de modèle

1. Alors que vous êtes toujours dans la section **Modèles** de **Générer**, ajoutez plusieurs nouveaux énoncés de modèle. Sélectionnez **OrgChart-Manager** dans le menu déroulant d’intentions et entrez chacun des énoncés de modèle suivants :

    |Intentionnel|Exemples d’énoncés avec un texte facultatif et des entités prédéfinies|
    |--|--|
    |OrgChart-Manager|`who was {EmployeeListEntity}['s] manager [[on]{datetimeV2}?]`|
    |OrgChart-Manager|`who will be {EmployeeListEntity}['s] manager [[in]{datetimeV2}?]`|
    |OrgChart-Manager|`who will be {EmployeeListEntity}['s] manager [[on]{datetimeV2}?]`|

2. Sélectionnez **Train** (Entraîner) dans la barre de navigation pour entraîner l’application.

3. À la fin de l’entraînement, sélectionnez **Test** (Tester) en haut du panneau pour ouvrir le panneau de test.

4. Entrez plusieurs énoncés de test pour vérifier que le modèle est mis en correspondance et que le score de l’intention est très élevé.

    Après avoir entré le premier énoncé, sélectionnez **Inspecter** sous le résultat, afin que vous puissiez voir tous les résultats de prédiction. Chaque énoncé doit avoir l’intention **OrgChart-Manager** et doit extraire les valeurs des entités `EmployeeListEntity` et `datetimeV2`.

    |Énoncé|
    |--|
    |`Who will be Jill Jones manager`|
    |`who will be jill jones's manager`|
    |`Who will be Jill Jones's manager?`|
    |`who will be Jill jones manager on March 3`|
    |`Who will be Jill Jones manager next Month`|
    |`Who will be Jill Jones manager in a month?`|

Tous ces énoncés ont trouvé les entités à l’intérieur, par conséquent, ils correspondent au même modèle et ont un score de prédiction élevé. Vous avez ajouté plusieurs modèles qui correspondront à de nombreuses variantes d’énoncés. Vous n’avez pas eu besoin d’ajouter des exemples d’énoncés dans l’intention pour faire fonctionner l’énoncé de modèle dans le modèle.

Cette utilisation de modèles a fourni :
* Des scores de prédiction plus élevés
* Avec les mêmes exemples d’énoncés dans l’intention
* Avec seulement quelques énoncés de modèle bien construits dans le modèle

### <a name="use-the-or-operator-and-groups"></a>Utiliser l’opérateur OR et les groupes

Plusieurs des énoncés de modèle précédents sont très proches. Utilisez la syntaxe **group** `()` et **OR** `|` pour réduire les énoncés de modèle.

Les deux modèles suivants peuvent être combinés dans un modèle unique à l’aide de la syntaxe de groupe `()` et OR `|`.

|Intentionnel|Exemples d’énoncés avec un texte facultatif et des entités prédéfinies|
|--|--|
|OrgChart-Manager|`who will be {EmployeeListEntity}['s] manager [[in]{datetimeV2}?]`|
|OrgChart-Manager|`who will be {EmployeeListEntity}['s] manager [[on]{datetimeV2}?]`|

Le nouvel énoncé de modèle sera :

`who ( was | is | will be ) {EmployeeListEntity}['s] manager [([in]|[on]){datetimeV2}?]`.

Cet exemple utilise un **groupe** autour de la conjugaison du verbe requis `in` et `on` facultatifs avec un canal **ou** entre les deux.

> [!NOTE]
> Lorsque vous utilisez le symbole _OR_ ou `|` (barre verticale) dans l’exemple de modèle, veillez à ajouter un espace avant et après la barre verticale.

1. Sur la page **Modèles**, sélectionnez le filtre **OrgChart-Manager**. Limitez la liste en recherchant `manager`.

1. Conservez une version de l’énoncé de modèle (à modifier dans l’étape suivante) et supprimez les autres variations.

1. Modifiez l’énoncé de modèle en :

    `who ( was | is | will be ) {EmployeeListEntity}['s] manager [([in]|[on]){datetimeV2}?]`

2. Sélectionnez **Train** (Entraîner) dans la barre de navigation pour entraîner l’application.

3. À la fin de l’entraînement, sélectionnez **Test** (Tester) en haut du panneau pour ouvrir le panneau de test.

    Utilisez le volet de test pour tester les versions de l’énoncé :

    |Énoncés à entrer dans le volet de test|
    |--|
    |`Who is Jill Jones manager this month`|
    |`Who is Jill Jones manager on July 5th`|
    |`Who was Jill Jones manager last month`|
    |`Who was Jill Jones manager on July 5th`|
    |`Who will be Jill Jones manager in a month`|
    |`Who will be Jill Jones manager on July 5th`|

En utilisant une syntaxe de modèle plus riche, vous réduisez le nombre d’énoncés de modèle à maintenir dans votre application, tout en ayant toujours un score de prédiction élevé.

### <a name="use-the-utterance-beginning-and-ending-anchors"></a>Utilisez les ancres de début et de fin de l’énoncé

La syntaxe du modèle fournit la syntaxe d’ancre de début et de fin d’énoncé, à savoir le signe insertion, `^`. Les ancres de début et de fin d’énoncé peuvent être utilisées ensemble pour cibler des énoncé très spécifiques et éventuellement littéraux ou utilisées séparément pour cibler des intentions.

## <a name="using-patternany-entity"></a>Utilisation de l’entité pattern.any

[!INCLUDE [Pattern.any entity - concepts](./includes/pattern-any-entity.md)]

### <a name="add-example-utterances-with-patternany"></a>Ajouter des exemples d’énoncés avec Pattern.any

1. Sélectionnez **Build** dans le volet de navigation supérieur, puis **Intentions** dans le volet de navigation gauche.

1. Sélectionnez **FindForm** dans la liste des intentions.

1. Ajoutez quelques exemples d’énoncés. Le texte qui doit être prédit en tant que Pattern.any apparaît **en gras**. Il est difficile de déterminer le nom du formulaire à partir des autres mots situés autour de l’énoncé. À cet effet, Pattern.any aide en marquant les limites de l’entité.

    |Exemple d’énoncé|Nom du formulaire|
    |--|--|
    |Où se trouve le formulaire **What to do when a fire breaks out in the Lab** (Que faire en cas d’incendie dans le labo) ? Et qui doit le signer une fois que je l’ai lu ?|What to do when a fire breaks out in the Lab
    |Où se trouve le formulaire **Request relocation from employee new to the company** (demande de réaffectation d'un nouvel employé) sur le serveur ?|Request relocation from employee new to the company|
    |Qui a écrit le formulaire **Health and wellness requests on the main campus** (demandes concernant la santé et le bien-être sur le campus principal) et quelle est la version la plus récente ?|Health and wellness requests on the main campus|
    |Je recherche le formulaire intitulé **Office move request including physical assets** (demande de déménagement de bureau, y compris les biens matériels). |Office move request including physical assets|

    Sans entité Pattern.any, il serait difficile pour LUIS de comprendre où se termine le titre du formulaire en raison des nombreuses variations des noms de formulaire.

### <a name="create-a-patternany-entity"></a>Créer une entité Pattern.any
L’entité Pattern.any extrait des entités de longueur variable. Elle fonctionne uniquement dans un modèle, car le modèle marque le début et la fin de l’entité avec la syntaxe.

1. Sélectionnez **Entités** dans le volet de navigation gauche.

1. Sélectionnez **+ Create** (Créer), entrez le nom `FormName`, puis sélectionnez le type **Pattern.any**. Sélectionnez **Create** (Créer).

### <a name="add-a-pattern-that-uses-the-patternany"></a>Ajouter un modèle qui utilise l’entité Pattern.any

1. Sélectionnez **Modèles** dans le volet de navigation gauche.

1. Sélectionnez l’intention **FindForm**.

1. Entrez les modèles d’énoncés suivants, qui utilisent la nouvelle entité :

    |Modèles d’énoncés|
    |--|
    |`Where is the form ["]{FormName}["] and who needs to sign it after I read it[?]`|
    |`Where is ["]{FormName}["] on the server[?]`|
    |`Who authored ["]{FormName}["] and what is the most current version[?]`|
    |`I'm looking for the form named ["]{FormName}["][.]`|

1. Effectuez l’apprentissage de l’application.

### <a name="test-the-new-pattern-for-free-form-data-extraction"></a>Tester l’extraction de données à structure libre dans le nouveau modèle
1. Sélectionnez **Tester** dans la barre supérieure pour ouvrir le panneau de test.

1. Entrez l’énoncé suivant :

    `Where is the form Understand your responsibilities as a member of the community and who needs to sign it after I read it?`

1. Sélectionnez **Inspecter** sous le résultat afin de voir les résultats des tests pour l’entité et l’intention.

    L’entité `FormName` est trouvée en premier, suivie du modèle, indiquant l’intention. Si, dans votre résultat de test, les entités ne sont pas détectées et, par conséquent, le modèle est introuvable, vous devrez ajouter des exemples d’énoncés sur l’intention (et non le modèle).

1. Fermez le panneau de test en sélectionnant le bouton **Tester** dans le volet de navigation supérieur.

### <a name="using-an-explicit-list"></a>Avec une liste explicite

Si vous constatez que votre modèle, quand il comporte une entité Pattern.any, extrait mal les entités, utilisez une [liste explicite](reference-pattern-syntax.md#explicit-lists) pour corriger ce problème.

## <a name="what-did-this-tutorial-accomplish"></a>Récapitulatif de ce tutoriel

Dans ce didacticiel, vous avez ajouté des modèles pour permettre à LUIS de prédire l’intention avec un score beaucoup plus élevé sans avoir à ajouter davantage d’exemples d’énoncés. Le marquage des entités et du texte pouvant être ignoré a permis à LUIS d’appliquer le modèle à une plus grande variété d’énoncés.

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>Étapes suivantes


> [!div class="nextstepaction"]
> [En savoir plus sur l’utilisation de rôles avec un modèle](luis-tutorial-pattern.md)
