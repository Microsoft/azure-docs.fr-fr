---
title: Fonctionnalités de Machine Learning avec LUIS
description: Ajoutez des fonctionnalités à un modèle de langage afin de fournir des conseils sur la façon de reconnaître les entrées que vous souhaitez étiqueter ou classer.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 08ab71375171d4bb4167c725bc7118bec2e1ebfa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91372001"
---
# <a name="machine-learning-features"></a>Caractéristiques issues de l’apprentissage automatique

En apprentissage automatique, une  *caractéristique* est un trait ou un attribut distinctif des données que votre système observe et dont il apprend.

Les caractéristiques de Machine Learning fournissent à LUIS des indications importantes pour rechercher les éléments qui distinguent un concept. Il ne s’agit pas de règles strictes mais de conseils que LUIS peut utiliser. LUIS utilise ces conseil conjointement avec les étiquettes pour rechercher les données.

Une fonctionnalité peut être décrite comme une fonction, par exemple f(x) = y. Dans l’exemple d’énoncé, la fonctionnalité indique où rechercher la caractéristique distinctive. Servez-vous de ces informations pour créer votre schéma.

## <a name="types-of-features"></a>Types de caractéristiques

Les caractéristiques sont essentielles à la conception de votre schéma. LUIS prend en charge les listes d’expressions et d’autres modèles en tant que caractéristiques :

* Caractéristiques de liste d’expressions
* Modèle (intention ou entité) en tant que caractéristique

## <a name="find-features-in-your-example-utterances"></a>Rechercher des caractéristiques dans vos exemples d’énoncés

LUIS étant une application basée sur la langue, les caractéristiques sont basées sur du texte. Choisissez le texte qui indique le trait que vous souhaitez distinguer. Pour LUIS, la plus petite unité est le *jeton*. Pour l’anglais, un jeton est une étendue continue de lettres et de chiffres sans espace ou signe de ponctuation.

Étant donné que les espaces et les signes de ponctuation ne sont pas des jetons, concentrez vous sur les indices textuels que vous pouvez utiliser comme caractéristiques. N’oubliez pas d’inclure les variantes de mots, par exemple :

* formes plurielles ;
* temps de verbes ;
* abréviations ;
* orthographes et fautes d’orthographe.

Déterminez si le texte, parce qu’il se distingue d’une caractéristique, doit :

* Correspondre à un mot ou une expression exacts. Envisagez d’ajouter une entité d’expression régulière ou une entité de liste comme caractéristique de l’entité ou de l’intention.
* Correspondre à un concept bien connu, comme des dates, des heures ou des noms de personnes. Utilisez une entité prédéfinie en tant que caractéristique de l’entité ou de l’intention.
* Apprendre de nouveaux exemples au fil du temps. Utilisez une liste d’expressions de quelques exemples du concept comme caractéristique de l’entité ou de l’intention.

## <a name="create-a-phrase-list-for-a-concept"></a>Créer une liste d’expressions pour un concept

Une liste d’expressions est une liste de mots ou d’expressions qui décrit un concept. Une liste d’expressions est appliquée en tant que correspondance ne respectant pas la casse au niveau du jeton.

Lors de l’ajout d’une liste d’expressions, vous pouvez définir la caractéristique en tant que **[globale](#global-features)** . Une caractéristique globale s’applique à l’ensemble de l’application.

### <a name="when-to-use-a-phrase-list"></a>Quand utiliser une liste d’expressions

Utilisez une liste d’expressions, si votre application LUIS doit être en mesure de généraliser et d’identifier de nouveaux éléments. Les listes d’expressions sont similaires à un lexique spécifique d’un domaine. Elles améliorent la qualité de la compréhension des intentions et des entités.

### <a name="how-to-use-a-phrase-list"></a>Comment utiliser une liste d’expressions

Avec une liste d’expressions, LUIS examine le contexte et généralise pour identifier des éléments similaires, sans être une correspondance de texte exacte. Pour utiliser une liste d’expressions, procédez comme suit :

1. Commencez par une entité issue de l’apprentissage automatique :
    1. Ajoutez des exemples d’énoncés.
    1. Étiquetez avec une entité issue de l’apprentissage automatique.
1. Ajoutez une liste d’expressions :
    1. Ajoutez des mots de sens similaire. N’ajoutez pas tous les mots ou expressions possibles. Au lieu de cela, ajoutez quelques mots ou expressions à la fois. Entraînez à nouveau et publiez.
    1. Révisez le résultat et ajoutez des mots suggérés.

### <a name="a-typical-scenario-for-a-phrase-list"></a>Scénario classique pour une liste d’expressions

Un scénario classique pour une liste d’expressions consiste à amplifier les mots liés à une idée spécifique.

Les termes médicaux constituent un bon exemple de mots pouvant nécessiter une liste d’expressions pour améliorer leur sens. Ces termes peuvent avoir des sens physiques, chimiques, thérapeutiques ou abstraits. Sans liste d’expressions, LUIS ignore quels termes sont importants pour votre domaine de sujet.

Pour extraire les termes médicaux :

1. Créez des exemples d’énoncés et étiquetez-y les termes médicaux.
2. Créez une liste d’expressions contenant des exemples de termes du domaine concerné. Cette liste d’expressions doit inclure le terme que vous avez étiqueté et d’autres termes qui décrivent le même concept.
3. Ajoutez la liste d’expressions à l’entité ou à la sous-entité qui extrait le concept utilisé dans la liste d’expressions. Le scénario le plus courant est un composant (enfant) d’une entité de machine-learning. Si la liste d’expressions doit être appliquée à toutes les intentions ou entités, marquez-la en tant que liste d’expressions globale. L’indicateur **enabledForAllModels** contrôle la portée de ce modèle dans l’API.

### <a name="token-matches-for-a-phrase-list"></a>Correspondances de jeton pour une liste d’expressions

Une liste d’expressions s’applique toujours au niveau du jeton. Le tableau suivant montre comment une liste d’expressions contenant le mot **Anne** s’applique aux variations des mêmes caractères dans cet ordre.


| Variation du jeton **Anne** | Liste d’expressions correspondant à la détection du jeton |
|--------------------------|---------------------------------------|
| **ANNE**<br>**aNNE**<br>           | Oui : le jeton est **Anne**                  |
| **d’Anne**                    | Oui : le jeton est **Anne**                  |
| **Ann**                     | Non : le jeton est **Ann**                  |

<a name="how-to-use-phrase-lists"></a>
<a name="how-to-use-a-phrase-lists"></a>
<a name="phrase-lists-help-identify-simple-exchangeable-entities"></a>

## <a name="a-model-as-a-feature-helps-another-model"></a>Un modèle en tant que caractéristique renforce un autre modèle

Vous pouvez ajouter un modèle (intention ou entité) en tant que caractéristique à un autre modèle (intention ou entité). En ajoutant une intention ou une entité existantes en tant que caractéristique, vous ajoutez un concept bien défini avec des exemples étiquetés.

Lorsque vous ajoutez un modèle en tant que caractéristique, vous pouvez définir la caractéristique comme suit :
* **[Requise](#required-features)** . Une caractéristique requise doit être trouvée afin que le modèle soit retourné à partir du point de terminaison de prédiction.
* **[Globale](#global-features)** . Une caractéristique globale s’applique à l’ensemble de l’application.

### <a name="when-to-use-an-entity-as-a-feature-to-an-intent"></a>Quand utiliser une entité comme caractéristique d’une intention

Ajoutez une entité en tant que caractéristique d’une intention lorsque la détection de cette entité est significative pour l’intention.

Par exemple, si l’intention est de réserver un vol, comme **BookFlight**, et que l’entité désigne les informations du billet (telles que le nombre de sièges, l’origine et la destination), trouver l’entité d’informations du billet doit alors ajouter du poids à la prédiction de l’intention **BookFlight**.

### <a name="when-to-use-an-entity-as-a-feature-to-another-entity"></a>Quand utiliser une entité comme caractéristique d’une autre entité

Une entité (A) doit être ajoutée en tant que caractéristique à une autre entité (B) lorsque la détection de cette entité (A) est importante pour la prédiction de l’entité (B).

Par exemple, si une entité d’adresse de livraison est contenu dans une sous-entité d’adresse de rue, la recherche de la sous-entité d’adresse de rue ajoute un poids significatif à la prédiction pour l’entité d’adresse de livraison.

* Adresse de livraison (entité issue de l’apprentissage automatique) :

    * Numéro de rue (sous-entité)
    * Rue (sous-entité)
    * Ville (sous-entité)
    * Région ou département (sous-entité)
    * Pays/région (sous-entité)
    * Code postal (sous-entité)

## <a name="nested-subentities-with-features"></a>Sous-entités imbriquées avec des caractéristiques

Une sous-entité issue de l’apprentissage automatique indique qu’un concept est présent à l’entité parente. Celle-ci peut être une autre sous-entité ou l’entité supérieure. La valeur de la sous-entité fait office de caractéristique pour son entité parente.

Une sous-entité peut avoir pour caractéristique tant une liste d’expressions qu’un modèle (une autre entité).

Quand la sous-entité comprend une liste d’expressions, cela enrichit le vocabulaire du concept, mais n’ajoute pas d’informations à la réponse JSON de la prédiction.

Lorsque la sous-entité a une caractéristique d’une autre entité, la réponse JSON inclut les données extraites de celle-ci.


## <a name="required-features"></a>Caractéristiques requises

Une caractéristique requise doit être trouvée pour que le modèle soit retourné à partir du point de terminaison de prédiction. Utilisez une caractéristique requise lorsque vous savez que vos données entrantes doivent correspondre à la caractéristique.

Si le texte de l’énoncé ne correspond pas à la caractéristique requise, il n’est pas extrait.

Une caractéristique requise utilise une entité non issue de l’apprentissage automatique :

* Entité Expression régulière
* Entité de liste
* Entité prédéfinie

Si vous avec la conviction que votre modèle figure dans les données, définissez la caractéristique comme requise. Une caractéristique requise ne retourne rien si elle est introuvable.

Poursuivons avec l’exemple de l’adresse de livraison :

Adresse de livraison (entité issue du Machine Learning)

 * Numéro de rue (sous-entité)
 * Rue (sous-entité)
 * Numéro de rue (sous-entité)
 * Ville (sous-entité)
 * Région ou département (sous-entité)
 * Pays/région (sous-entité)
 * Code postal (sous-entité)

### <a name="required-feature-using-prebuilt-entities"></a>Caractéristique requise utilisant des entités prédéfinies

Les informations de localité/ville, département/province et pays/région sont généralement constitués d’un ensemble fermé de listes qui ne changent pas beaucoup au fil du temps. Ces entités peuvent avoir des caractéristiques recommandées pertinentes et ces caractéristiques peuvent être définies comme « requises ». Cela signifie que l’adresse de livraison entière n’est pas renvoyée si les entités qui ont des caractéristiques requises sont introuvables.

Que se passe-t-il si des informations de localité/ville, département/province et pays/région figurent dans l’énoncé, mais sont exprimées dans un dialecte ou un argot que LUIS ne comprend pas ? Si vous souhaitez utiliser un post-traitement pour résoudre l’entité (par exemple, si LUIS retourne un score de confiance faible), ne marquez pas la caractéristique comme « requise ».

Un autre exemple de caractéristique requise pour l’adresse de livraison consiste à définir le numéro de rue en tant que numéro [prédéfini](luis-reference-prebuilt-entities.md) requis. Cela permet à un utilisateur d’entrer « 1 rue Microsoft » ou « un rue Microsoft ». Les deux correspondent à la valeur numérique « 1 » pour la sous-entité de numéro de rue.

### <a name="required-feature-using-list-entities"></a>Caractéristique requise utilisant des entités de listes

Une [entité de liste](reference-entity-list.md) est utilisée en tant que liste de noms canoniques, ainsi que leurs synonymes. En tant que caractéristique requise, si l’énoncé n’inclut ni le nom canonique ni un synonyme, l’entité n’est pas retournée dans le cadre du point de terminaison de prédiction.

Supposons que votre entreprise n’expédie que vers un ensemble limité de pays/régions. Vous pouvez créer une entité de liste incluant plusieurs méthodes que votre client peut utiliser pour référencer le pays ou la région. Si LUIS ne trouve pas de correspondance exacte dans le texte de l’énoncé, l’entité (qui a la caractéristique requise de l’entité de liste) n’est pas renvoyée dans la prédiction.

|Nom canonique|Synonymes|
|--|--|
|États-Unis|États-Unis<br>É.-U. A.<br>US<br>États-Unis<br>0|

Une application cliente, telle qu’un bot conversationnel, peut poser une question de suivi pour vous aider. Cela permet au client de comprendre que la sélection de pays/régions est limitée et *requise*.

### <a name="required-feature-using-regular-expression-entities"></a>Caractéristique requise utilisant des entités d’expression régulière

Une [entité d’expression régulière](reference-entity-regular-expression.md) utilisée comme caractéristique requise offre des caractéristiques de correspondant de texte riches.

Dans notre exemple d’adresse de livraison, vous pouvez créer une expression régulière qui capture les règles de syntaxe des codes postaux de pays/régions.

## <a name="global-features"></a>Caractéristiques globales

Bien que l’utilisation la plus courante consiste à appliquer une caractéristique à un modèle spécifique, vous pouvez configurer la caractéristique en tant que **caractéristique globale** pour l’appliquer à l’ensemble de votre application.

L’utilisation la plus courante d’une caractéristique globale consiste à ajouter un vocabulaire à l’application. Par exemple, si vos clients utilisent une langue principale, mais s’attendent à pouvoir utiliser une autre langue dans le même énoncé, vous pouvez ajouter une caractéristique qui comprend des mots de la langue secondaire.

Étant donné que l’utilisateur s’attend à utiliser la langue secondaire pour exprimer toute intention ou entité, ajoutez des mots de la langue secondaire à la liste d’expressions. Configurez la liste d’expressions en tant que fonctionnalité globale.

## <a name="combine-features-for-added-benefit"></a>Combiner les caractéristiques pour bénéficier d'avantages supplémentaires

Vous pouvez utiliser plusieurs caractéristiques pour décrire un trait ou un concept. L'association suivante est couramment utilisée :

* Caractéristique de liste d'expressions : vous pouvez utiliser plusieurs listes d'expressions comme caractéristiques d'un même modèle.
* Modèle en tant que caractéristique : [entité prédéfinie](luis-reference-prebuilt-entities.md), [entité d'expression régulière](reference-entity-regular-expression.md), [entité de liste](reference-entity-list.md). 

### <a name="example-ticket-booking-entity-features-for-a-travel-app"></a>Exemple : caractéristiques d'une entité de réservation de billets pour une application de voyage  

Prenons l'exemple d'une application permettant de réserver un vol avec une _intention_ de réservation de vol et une _entité_ de réservation de billets. L'entité de réservation de billets capture les informations nécessaires à la réservation d'un billet d'avion dans un système de réservation. 

L'entité d'apprentissage automatique associée à la réservation de billets comporte deux sous-entités permettant de capturer l'origine et la destination. Les caractéristiques doivent être ajoutées à chaque sous-entité, et non à l'entité de niveau supérieur.

:::image type="content" source="media/luis-concept-features/ticket-booking-entity.png" alt-text="Schéma de l'entité de réservation de billets":::

L'entité de réservation de billets est une entité d'apprentissage automatique, avec des sous-entités _Origine_ et _Destination_. Ces sous-entités indiquent toutes deux un emplacement géographique. Pour faciliter l'extraction des emplacements, et faire la distinction entre _Origine_ et _Destination_, chaque sous-entité doit disposer de caractéristiques.

|Type|Sous-entité Origine |Sous-entité Destination|
|--|--|--|
|Modèle en tant que fonctionnalité|Entité prédéfinie [geographyV2](luis-reference-prebuilt-geographyv2.md?tabs=V3)|Entité prédéfinie [geographyV2](luis-reference-prebuilt-geographyv2.md?tabs=V3)|
|Liste d’expressions|**Expressions liées à l'origine** : `start at`, `begin from`, `leave`|**Expressions liées à la destination** : `to`, `arrive`, `land at`, `go`, `going`, `stay`, `heading`|
|Liste d’expressions|Codes d'aéroport - même liste pour l'origine et la destination|Codes d'aéroport - même liste pour l'origine et la destination|
|Liste d’expressions|Noms d'aéroport - même liste pour l'origine et la destination|Codes d'aéroport - même liste pour l'origine et la destination|

Si vous souhaitez que les gens utilisent les codes et les noms des aéroports, LUIS doit disposer de listes d'expressions qui utilisent les deux types d'expressions. Les codes d'aéroport peuvent être plus courants avec du texte saisi dans un bot conversationnel, tandis que les noms d'aéroport peuvent être plus courants avec une conversation orale telle qu'un bot conversationnel à reconnaissance vocale.

Les détails correspondants des caractéristiques sont uniquement renvoyés pour les modèles, et non pour les listes d'expressions, car seuls les modèles sont renvoyés dans le JSON de prédiction.

#### <a name="ticket-booking-labeling-in-the-intent"></a>Étiquetage des réservations de billets dans l'intention

Après avoir créé l'entité d'apprentissage automatique, vous devez ajouter des exemples d'énoncés à une intention, et étiqueter l'entité parente et toutes les sous-entités.

Pour l'exemple de réservation de billets, étiquetez les exemples d'énoncés de l'intention avec l'entité `TicketBooking` et les sous-entités du texte.

:::image type="content" source="media/luis-concept-features/intent-example-utterances-machine-learning-entity.png" alt-text="Schéma de l'entité de réservation de billets":::

### <a name="example-pizza-ordering-app"></a>Exemple : application de commande de pizzas

Prenons maintenant l'exemple d'une application de pizzeria qui reçoit des commandes de pizzas, avec les détails du type de pizza que commande le client. Chaque détail de la pizza doit être extrait, si possible, afin de traiter la commande.

L'entité d'apprentissage automatique de cet exemple est plus complexe, avec des sous-entités imbriquées, des listes d'expressions, des entités prédéfinies et des entités personnalisées.

:::image type="content" source="media/luis-concept-features/pizza-order-entity.png" alt-text="Schéma de l'entité de réservation de billets":::

Cet exemple utilise des caractéristiques aux niveaux de la sous-entité et de l'enfant de la sous-entité. La détermination du niveau qui reçoit tel ou tel type de liste d'expressions ou de modèle comme caractéristique constitue une partie importante de la conception de votre entité.

Tandis que les sous-entités peuvent disposer de nombreuses listes d'expressions comme caractéristiques pour la détection de l'entité, chaque sous-entité ne dispose que d'un seul modèle comme caractéristique. Dans cette [application de commandes de pizzas](https://github.com/Azure/pizza_luis_bot/blob/master/CognitiveModels/MicrosoftPizza.json), ces modèles sont principalement des listes.

:::image type="content" source="media/luis-concept-features/intent-example-utterances-machine-learning-entity-pizza.png" alt-text="Schéma de l'entité de réservation de billets":::

Les exemples d'énoncés correctement étiquetés illustrent la façon dont les entités sont imbriquées. 


## <a name="best-practices"></a>Meilleures pratiques

Découvrir les [meilleures pratiques](luis-concept-best-practices.md).

## <a name="next-steps"></a>Étapes suivantes

* [Étendez](schema-change-prediction-runtime.md) vos modèles d’application au niveau du runtime de prédiction.
* Consultez [Ajouter des fonctionnalités](luis-how-to-add-features.md) pour découvrir comment ajouter des fonctionnalités à votre application LUIS.
