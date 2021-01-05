---
title: Informations de référence sur la syntaxe du modèle - LUIS
description: Créez des entités pour extraire les données clés des énoncés de l'utilisateur dans les applications LUIS (Language Understanding). Les données extraites sont utilisées par l’application cliente.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 04/14/2020
ms.openlocfilehash: 3caccd6766226ce68b371856b081b052c1033f71
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91542187"
---
# <a name="pattern-syntax"></a>Syntaxe du modèle

La syntaxe du modèle est un modèle d’énoncé. Le modèle doit contenir les mots et entités que vous souhaitez faire correspondre, ainsi que les mots et la [ponctuation](luis-reference-application-settings.md#punctuation-normalization) que vous souhaitez ignorer. Ce n’est **pas** une expression régulière.

> [!CAUTION]
> Les modèles incluent uniquement les parents d’entité de Machine Learning, pas les sous-entités.

Les entités des modèles sont placées entre accolades, `{}`. Les modèles peuvent inclure des entités et des entités avec rôles. [Pattern.any](luis-concept-entity-types.md#patternany-entity) est une entité qui n’est utilisée que dans les modèles.

Les modèles acceptent la syntaxe suivante :

|Fonction|Syntaxe|Niveau d’imbrication|Exemple|
|--|--|--|--|
|entité| {} – accolades|2|Où se trouve le formulaire {nom-entité} ?|
|facultatif|[] – crochets<BR><BR>Les niveaux d’imbrication de la syntaxe facultative et de la syntaxe de regroupement sont limités à trois, pour toutes les combinaisons possibles |2|Le point d’interrogation est facultatif [?]|
|regroupement|() – parenthèses|2|est (a \| b)|
|or| \| – barre verticale (pipe)<br><br>Les barres verticales (ou) sont limitées à deux par groupe |-|Où est le formulaire ({nom-court-formulaire} &#x7c; {nom-long-formulaire} &#x7c; {numéro-formulaire})|
|Début ou fin de l’énoncé|^ – accent circonflexe (caret)|-|^commence l’énoncé<br>l’énoncé est terminé^<br>^correspondance littérale stricte de tout l’énoncé avec l’entité {numéro}^|

## <a name="nesting-syntax-in-patterns"></a>Syntaxe d’imbrication dans les modèles

La syntaxe **facultatif**, entre crochets, peut être imbriquée sur deux niveaux. Par exemple : `[[this]is] a new form`. Cet exemple accepte les énoncés suivants :

|Exemple d’énoncé facultatif imbriqué|Explication|
|--|--|
|this is a new form|correspond à tous les mots du modèle|
|is a new form|correspond au mot facultatif externe et aux mots non facultatifs du modèle|
|a new form|correspond uniquement aux mots obligatoires|

La syntaxe **regroupement**, entre parenthèses, peut être imbriquée sur deux niveaux. Par exemple : `(({Entity1.RoleName1} | {Entity1.RoleName2} ) | {Entity2} )`. Cette fonctionnalité permet une correspondance avec chacune des trois entités.

Si Entity1 est un emplacement comportant des rôles comme l’origine (Seattle) et la destination (Cairo) et Entity2 un nom de bâtiment connu issu d’une entité de liste (RedWest-C), les énoncés suivants correspondent à ce modèle :

|Exemple d’énoncé de regroupement imbriqué|Explication|
|--|--|
|RedWest-C|correspond à l’entité de regroupement externe|
|Seattle|correspond à l’une des entités de regroupement internes|
|Le Caire|correspond à l’une des entités de regroupement internes|

## <a name="nesting-limits-for-groups-with-optional-syntax"></a>Limites d’imbrication des groupes à syntaxe facultative

Les combinaisons entre les syntaxes **regroupement** et **facultatif** sont limitées à trois niveaux d’imbrication.

|Autorisé|Exemple|
|--|--|
|Oui|( [ ( test1 &#x7c; test2 ) ] &#x7c; test3 )|
|Non|( [ ( [ test1 ] &#x7c; test2 ) ] &#x7c; test3 )|

## <a name="nesting-limits-for-groups-with-or-ing-syntax"></a>Limites d’imbrication des groupes à syntaxe de type ou

Les combinaisons entre les syntaxes **regroupement** et **ou** sont limitées à deux barres verticales.

|Autorisé|Exemple|
|--|--|
|Oui|( test1 &#x7c; test2 &#x7c; ( test3 &#x7c; test4 ) )|
|Non|( test1 &#x7c; test2 &#x7c; test3 &#x7c; ( test4 &#x7c; test5 ) ) |

## <a name="syntax-to-add-an-entity-to-a-pattern-template"></a>Syntaxe pour ajouter une entité à un modèle
Pour ajouter une entité au modèle, placez le nom de l’entité entre accolades, comme `Who does {Employee} manage?`.

|Modèle avec une entité|
|--|
|`Who does {Employee} manage?`|

## <a name="syntax-to-add-an-entity-and-role-to-a-pattern-template"></a>Syntaxe pour ajouter une entité et un rôle à un modèle
Le rôle d’une entité est indiqué par `{entity:role}` avec le nom de l’entité suivi d’un signe deux-points, puis du nom de rôle. Pour ajouter une entité et un rôle au modèle, placez le nom de l’entité et le nom du rôle entre accolades, comme `Book a ticket from {Location:Origin} to {Location:Destination}`.

|Modèle avec des rôles d’entités|
|--|
|`Book a ticket from {Location:Origin} to {Location:Destination}`|

## <a name="syntax-to-add-a-patternany-to-pattern-template"></a>Syntaxe pour ajouter un pattern.any à un modèle
L’entité Pattern.any vous permet d’ajouter une entité de longueur variable au modèle. Tant que le modèle est suivi, l’entité pattern.any peut être de n’importe quelle longueur.

Pour ajouter une entité **Pattern.any** au modèle, placez l’entité Pattern.any entre accolades, comme `How much does {Booktitle} cost and what format is it available in?`.

|Modèle avec une entité Pattern.any|
|--|
|`How much does {Booktitle} cost and what format is it available in?`|

|Titres de livres dans le modèle|
|--|
|Combien **Steal this book** coûte-t-il et dans quel format est-il disponible ?|
|Combien **Soufi, mon amour** coûte-t-il et dans quel format est-il disponible ?|
|Combien **Le Bizarre Incident du chien pendant la nuit** coûte-t-il et dans quel format est-il disponible ?|

Les mots du titre du livre ne gênent pas LUIS, car il sait où le titre du livre se termine grâce à l’entité Pattern.any.

## <a name="explicit-lists"></a>Listes explicites

Créez une [Liste explicite](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5ade550bd5b81c209ce2e5a8) via l’API de création pour autoriser l’exception si les deux conditions suivantes sont réunies :

* Votre modèle contient un [Pattern.any](luis-concept-entity-types.md#patternany-entity).
* La syntaxe de ce modèle autorise l’extraction d’une mauvaise entité selon l’énoncé.

Par exemple, supposons que vous disposez d’un modèle contenant la syntaxe facultative, `[]`, et la syntaxe d’entité, `{}`, combinées de façon à extraire les données de manière incorrecte.

Prenons l’exemple du modèle « [trouver] e-mail sur {objet} [de {personne}] ».

Dans les énoncés suivants, les entités **objet** et **personne** sont extraites correctement et incorrectement :

|Énoncé|Entité|Extraction correcte|
|--|--|:--:|
|e-mail sur les chiens de Chris|objet = les chiens<br>personne = Chris|✔|
|e-mail sur l’homme de La Manche|objet = l’homme<br>personne = La Manche|X|

Dans le tableau précédent, l’objet devrait être `the man from La Mancha` (titre de livre) ; cependant, comme il comporte le mot facultatif `from`, le titre est mal prédit.

Pour corriger cette exception dans le modèle, ajoutez `the man from la mancha` en tant que correspondance de liste explicite pour l’entité {objet} à l’aide de l’[API de création de liste explicite](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5ade550bd5b81c209ce2e5a8).

## <a name="syntax-to-mark-optional-text-in-a-template-utterance"></a>Syntaxe pour marquer le texte facultatif dans un modèle d’énoncé
Marquez un texte facultatif dans l’énoncé à l’aide de la syntaxe en crochet d’expression régulière, `[]`. Dans le texte facultatif, vous pouvez imbriquer jusqu'à deux crochets uniquement.

|Modèle avec un texte facultatif|Signification|
|--|--|
|`[find] email about {subject} [from {person}]`|`find` et `from {person}` sont facultatifs.|
|`Can you help me[?]|Le signe de ponctuation est facultatif|

Les signes de ponctuation (`?`, `!` et `.`) doivent être ignorés, avec la syntaxe entre crochets des modèles.

## <a name="next-steps"></a>Étapes suivantes

Découvrez-en plus sur les modèles :

* [Comment ajouter des modèles](luis-how-to-model-intent-pattern.md)
* [Comment ajouter une entité pattern.any](luis-how-to-add-entities.md#add-a-patternany-entity)
* [Concepts des modèles](luis-concept-patterns.md)

Découvrez comment les [sentiments](luis-reference-prebuilt-sentiment.md) sont retournés dans la réponse .json.