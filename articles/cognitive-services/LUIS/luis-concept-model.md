---
title: Concevoir avec des modèles – LUIS
description: Language Understanding fournit plusieurs types de modèles. Certains modèles peuvent être utilisés de plusieurs manières.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 04/30/2020
ms.openlocfilehash: 576ba945018d13db9cd24888f3c41a2215857694
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91316509"
---
# <a name="design-with-intent-and-entity-models"></a>Conception avec modèles d’intention et d’entité

Language Understanding fournit deux types de modèles qui vous permettent de définir le schéma de votre application. Le schéma de votre application détermine les informations que vous recevez de la prédiction d’un nouvel énoncé utilisateur.

Le schéma d’application est construit à partir des modèles que vous créez à l’aide du [Machine Teaching](#authoring-uses-machine-teaching) :
* Les [intentions](#intents-classify-utterances) classifient les énoncés utilisateur
* Les [entités](#entities-extract-data) extraient des données à partir de l’énoncé

## <a name="authoring-uses-machine-teaching"></a>Création utilise le Machine Teaching

La méthodologie de Machine Teaching de LUIS vous permet d’enseigner facilement des concepts à un ordinateur. Il n’est pas nécessaire d’utiliser LUIS pour comprendre le _Machine Learning_. Au lieu de cela, en tant qu’intervenant, vous communiquez des concepts à LUIS en fournissant des exemples du concept et en expliquant comment un concept doit être modélisé à l’aide d’autres concepts connexes. En tant qu’intervenant, vous pouvez également améliorer le modèle de LUIS de manière interactive en identifiant et en corrigeant les erreurs de prédiction.

<a name="v3-authoring-model-decomposition"></a>

## <a name="intents-classify-utterances"></a>Les intentions classifient les énoncés

Une intention classifie des exemples d’énoncés à enseigner à LUIS à propos de l’intention. Les exemples d’énoncés dans une intention sont utilisés comme exemples positifs d’énoncé. Ces mêmes énoncés sont utilisés comme exemples négatifs dans toutes les autres intentions.

Considérons une application qui doit déterminer l’intention d’un utilisateur de commander un livre et une application qui a besoin de l’adresse de livraison du client. Cette application a deux intentions : `OrderBook` et `ShippingLocation`.

L’énoncé suivant est un **exemple positif** pour l’intention `OrderBook` et un **exemple négatif** pour les intentions `ShippingLocation` et `None` :

`Buy the top-rated book on bot architecture.`

## <a name="entities-extract-data"></a>Les entités extraient les données

Une entité représente une unité de données que vous souhaitez extraire de l’énoncé. Une entité de machine-learning est une entité de niveau supérieur contenant des sous-entités qui sont également des entités de machine-learning.

Un exemple d’entité de machine-learning est une commande de billet d’avion. Conceptuellement, il s’agit d’une transaction unique comportant de nombreuses unités de données plus petites, telles que la date, l’heure, le nombre de places, le type de siège, le lieu d’origine, le lieu de destination et le choix du repas.

## <a name="intents-versus-entities"></a>Intentions et entités

Une intention est le résultat souhaité de l’énoncé _entier_, tandis que des entités sont des éléments de données extraits de l’énoncé. En général, les intentions sont liées aux actions que l’application cliente doit réaliser. Les entités sont des informations nécessaires à l’exécution de cette action. Du point de vue de la programmation, une intention déclenche un appel de méthode et les entités sont utilisées comme paramètres de cet appel de méthode.

Cet énoncé _doit_ avoir une intention et _peut_ avoir des entités :

`Buy an airline ticket from Seattle to Cairo`

Cet énoncé n’a qu’une seule intention :

* Achat d’un billet d’avion

Cet énoncé _peut_ avoir plusieurs entités :

* Emplacements de Seattle (origine) et Caire (destination)
* Quantité d’un billet nique

## <a name="entity-model-decomposition"></a>Décomposition du modèle d’entité

LUIS prend en charge la _décomposition de modèle_ avec des API de création, décomposant un concept en éléments plus petits. Cela vous permet de construire vos modèles en toute confiance dans la manière dont les différents éléments sont construits et prédits.

La décomposition du modèle comprend les éléments suivants :

* [intentions](#intents-classify-utterances)
    * [fonctionnalités](#features)
* [Entité de machine-learning](reference-entity-machine-learned-entity.md)
    * sous-entités (également des entités de machine-learning)
        * [fonctionnalités](#features)
            * [liste d’expressions](luis-concept-feature.md)
            * [entités non machine-learning](luis-concept-feature.md) telles que des [expressions régulières](reference-entity-regular-expression.md), des [listes](reference-entity-list.md) et des [entités prédéfinies](luis-reference-prebuilt-entities.md)

<a name="entities-extract-data"></a>
<a name="machine-learned-entities"></a>

## <a name="features"></a>Fonctionnalités

Une [caractéristique](luis-concept-feature.md) est un trait ou un attribut distinctif des données que votre système observe. Les caractéristiques de Machine Learning fournissent à LUIS des indications importantes pour rechercher les éléments qui distinguent un concept. Il ne s’agit pas de règles strictes mais de conseils que LUIS peut utiliser. Ces conseils sont utilisés conjointement avec les étiquettes pour rechercher les données.

## <a name="patterns"></a>Modèles

Les [modèles](luis-concept-patterns.md) sont conçus pour améliorer la précision lorsque plusieurs énoncés sont très similaires. Un modèle vous permet d’obtenir plus de précision pour une intention sans fournir de nombreux énoncés en plus.

## <a name="extending-the-app-at-runtime"></a>Extension de l’application au moment de l’exécution

Le schéma de l’application (modèles et caractéristiques) est formé et publié sur le point de terminaison de prédiction. Vous pouvez [transmettre de nouvelles informations](schema-change-prediction-runtime.md), en même temps que l’énoncé de l’utilisateur, au point de terminaison de prédiction pour améliorer la prédiction.

## <a name="next-steps"></a>Étapes suivantes

* Comprendre les [intentions](luis-concept-intent.md) et les [entités](luis-concept-entity-types.md).
* En savoir plus sur les [caractéristiques](luis-concept-feature.md)