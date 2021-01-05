---
title: Meilleures pratiques pour la création de votre application LUIS
description: Découvrez les meilleures pratiques pour obtenir les meilleurs résultats à partir du modèle de votre application LUIS.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 05/17/2020
ms.openlocfilehash: fc4475eb64744688e901d2b37e30632d044b0115
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95019311"
---
# <a name="best-practices-for-building-a-language-understanding-luis-app"></a>Meilleures pratiques pour la création d’une application LUIS (Language Understanding)
Suivez le processus de création d’applications pour générer votre application LUIS :

* Créer des modèles de langage (intentions et entités)
* Ajouter quelques exemples d’énoncés d’entraînement (15-30 par intention)
* Publier sur le point de terminaison
* Tester à partir du point de terminaison

Une fois votre application [publiée](luis-how-to-publish-app.md), utilisez le cycle de développement pour y ajouter des fonctionnalités, la publier et la tester à partir du point de terminaison. Ne commencez pas le cycle de création suivant en ajoutant des exemples d’énoncés supplémentaires, car LUIS ne pourrait pas apprendre votre modèle avec des énoncés d’utilisateurs réels.

Ne développez pas les énoncés tant que le jeu actuel d’exemples d’énoncés et d’énoncés du point de terminaison ne retourne pas des scores sûrs à haute valeur prédictive. Améliorez les scores à l’aide de l’[apprentissage actif](luis-concept-review-endpoint-utterances.md).




## <a name="do-and-dont"></a>Bonnes et mauvaises pratiques
La liste suivante indique les meilleures pratiques pour les applications LUIS :

|À faire|À ne pas faire|
|--|--|
|[Planifier votre schéma](#do-plan-your-schema)|[Créer et publier sans plan](#dont-publish-too-quickly)|
|[Définir des intentions distinctes](#do-define-distinct-intents)<br>[Ajouter des fonctionnalités aux intentions](#do-add-features-to-intents)<br>
[Utiliser des entités acquises sur ordinateur](#do-use-machine-learned-entities) |[Ajouter de nombreux exemples d’énoncés aux intentions](#dont-add-many-example-utterances-to-intents)<br>[Utiliser peu d’entités ou des entités simples](#dont-use-few-or-simple-entities) |
|[Trouver l’équilibre idéal, ni trop générique ni trop spécifique, pour chaque intention](#do-find-sweet-spot-for-intents)|[Utiliser LUIS comme plateforme d’apprentissage](#dont-use-luis-as-a-training-platform)|
|[Générer l’application de manière itérative avec des versions](#do-build-your-app-iteratively-with-versions)<br>[Générer des entités pour la décomposition du modèle](#do-build-for-model-decomposition)|[Ajouter de nombreux exemples d’énoncés du même format, en ignorant les autres formats](#dont-add-many-example-utterances-of-the-same-format-ignoring-other-formats)|
|[Ajouter des modèles dans les itérations ultérieures](#do-add-patterns-in-later-iterations)|[Mélanger la définition des intentions et des entités](#dont-mix-the-definition-of-intents-and-entities)|
|[Équilibrer vos énoncés entre toutes les intentions](#balance-your-utterances-across-all-intents), sauf l’intention None.<br>[Ajouter des exemples d’énoncés à l’intention None](#do-add-example-utterances-to-none-intent)|[Créer des listes d’expressions avec toutes les valeurs possibles](#dont-create-phrase-lists-with-all-the-possible-values)|
|[Tirer parti de la fonctionnalité de suggestion pour l’apprentissage actif](#do-leverage-the-suggest-feature-for-active-learning)|[Ajouter trop de modèles](#dont-add-many-patterns).|
|[Analyser les performances de l’application avec des tests par lots](#do-monitor-the-performance-of-your-app)|[Effectuer l’apprentissage et publier à chaque ajout d’exemple d’énoncé](#dont-train-and-publish-with-every-single-example-utterance)|

## <a name="do-plan-your-schema"></a>Planifier votre schéma

Avant de commencer à générer le schéma de votre application, vous devez identifier l’emplacement et l’endroit où vous envisagez d’utiliser cette application. Plus votre planification est détaillée et spécifique, plus votre application est performante.

* Rechercher des utilisateurs ciblés
* Définition de personnages de bout en bout pour représenter votre application : voix, avatar, gestion des problèmes (proactif, réactif)
* Identifiez les interactions de l’utilisateur (texte, discours) par le biais des canaux, en passant aux solutions existantes ou en créant une nouvelle solution pour cette application
* Parcours utilisateur de bout en bout
    * Que faut-il attendre de cette application ? * Quelles sont ses priorités ?
    * Quels sont les principaux cas d’usage ?
* Collecte des données : [en savoir plus](data-collection.md) sur la collecte et la préparation des données

## <a name="do-define-distinct-intents"></a>Définir des intentions distinctes
Le vocabulaire de chaque intention doit porter uniquement sur cette intention, sans chevauchement avec une autre. Par exemple, si vous souhaitez disposer d’une application qui gère les préparatifs de voyage, comme les vols et les hôtels, vous pouvez choisir de faire de ces domaines des intentions distinctes ou une même intention avec des entités pour les données précises à l’intérieur de l’énoncé.

Si le vocabulaire des deux intentions est identique, combinez l’intention et utilisez des entités.

Prenons les exemples d’énoncés suivants :

|Exemples d’énoncés|
|--|
|Book a flight|
|Book a hotel|

`Book a flight` et `Book a hotel` utilisent le même vocabulaire de `book a `. Le format étant le même, il doit s’agir de la même intention avec différentes entités extraites pour les mots `flight` et `hotel`.

## <a name="do-add-features-to-intents"></a>Ajouter des fonctionnalités aux intentions

Les fonctionnalités décrivent les concepts d’une intention. Une fonctionnalité peut être une liste d’expressions de mots significatifs pour cette intention ou une entité significative pour cette intention.

## <a name="do-find-sweet-spot-for-intents"></a>Trouver l’équilibre idéal pour les intentions
Utilisez les données de prédiction de LUIS pour déterminer si vos intentions se chevauchent, ce qui serait source de confusion pour LUIS. La meilleure intention serait alors trop proche d’une autre intention. Dans la mesure où LUIS n’utilise pas à chaque fois le même chemin d’accès à travers les données pour l’apprentissage, l’intention présentant un recoupement a des chances de finir première ou deuxième lors de l’apprentissage. Pour éviter cela, il faut que le score de l’énoncé de chaque intention soit plus éloigné. Lorsque les intentions sont bien distinctes les unes des autres, la meilleure intention devrait être à chaque fois conforme aux attentes.

## <a name="do-use-machine-learned-entities"></a>Utilisez des entités acquises sur ordinateur

Les entités acquises sur ordinateur sont adaptées à votre application et requièrent un étiquetage pour bien fonctionner. Si vous n’utilisez pas d’entité acquises sur ordinateur, vous utilisez peut-être le mauvais outil.

Les entités acquises sur ordinateur peuvent utiliser d’autres entités en tant que fonctionnalités. Ces entités peuvent être des entités personnalisées, telles que des entités d’expression régulière ou des entités de liste, ou des entités prédéfinies en tant que fonctionnalités.

En savoir plus sur les [entités acquises sur ordinateur efficaces](luis-concept-entity-types.md#effective-machine-learned-entities).

<a name="#do-build-the-app-iteratively"></a>

## <a name="do-build-your-app-iteratively-with-versions"></a>Générer l’application de manière itérative avec des versions

Chaque cycle de création doit figurer dans une nouvelle [version](./luis-concept-app-iteration.md), clonée à partir d’une version existante.

## <a name="do-build-for-model-decomposition"></a>Générer des entités pour la décomposition du modèle

La décomposition du modèle comprend un processus type :

* créer une **Intention** en fonction des intentions de l’utilisateur de l’application cliente
* ajouter 15-30 exemples d’énoncés basés sur une entrée utilisateur réelle
* étiqueter le concept de données de niveau supérieur dans l’exemple d’énoncé
* fractionner le concept de données en sous-entités
* ajouter des fonctionnalités aux sous-entités
* ajouter des fonctionnalités aux intentions

Une fois que vous avez créé l’intention et ajouté des exemples d’énoncés, l’exemple suivant décrit la décomposition d’entité.

Commencez par identifier les concepts de données complets que vous souhaitez extraire dans un énoncé. Il s’agit de votre entité de machine-learning. Décomposez ensuite l’expression en ses parties. Cela comprend l’identification des sous-entités et des fonctionnalités.

Par exemple, si vous souhaitez extraire une adresse, la première entité de machine-learning peut être appelée `Address`. Lors de la création de l’adresse, identifiez certaines de ses sous-entités, tels que l’adresse postale, la ville, l’État et le code postal.

Continuez à décomposer ces éléments en :
* Ajoutant une fonctionnalité requise du code postal en tant qu’entité d’expression régulière.
* Décomposition de l’adresse postale en parties :
    * Un **numéro de rue** avec une fonctionnalité requise d’une entité de nombre prédéfinie.
    * Un **nom de rue**.
    * Un **type de rue** avec une fonctionnalité requise d’une entité de liste incluant des mots tels que avenue, rond-point, route et ruelle.

L’API de création V3 permet la décomposition du modèle.

## <a name="do-add-patterns-in-later-iterations"></a>Ajouter des modèles dans les itérations ultérieures

Vous devez comprendre le comportement de l’application avant d’ajouter des [modèles](luis-concept-patterns.md) car les modèles sont pondérés plus fortement que les exemples d’énoncés et fausseront la confiance.

Une fois que vous comprenez le comportement de votre application, ajoutez des modèles tels qu’ils s’appliquent à votre application. Vous n’avez pas besoin de les ajouter avec chaque [itération](luis-concept-app-iteration.md).

Rien ne vous empêche de les ajouter au début de la conception de votre modèle, mais il est plus aisé de constater les changements produits par chaque modèle une fois le modèle testé avec des énoncés.

<a name="balance-your-utterances-across-all-intents"></a>

## <a name="do-balance-your-utterances-across-all-intents"></a>Équilibrer vos énoncés entre toutes les intentions

Pour que LUIS puisse faire des prédictions précises, la quantité d’exemples d’énoncés dans chaque intention (à l’exception de l’intention None) doit être plus ou moins égale.

Si vous avez une intention avec 100 exemples d’énoncés et une autre avec 20 exemples d’énoncés, celle avec 100 énoncés aura un taux de prédiction plus élevé.

## <a name="do-add-example-utterances-to-none-intent"></a>Ajouter des exemples d’énoncés à l’intention None

Cette intention est l’intention de secours, et indique tout ce qui ne concerne pas l’application. Ajoutez un exemple d’énoncé à l’intention None tous les 10 exemples d’énoncés dans le reste de votre application LUIS.

## <a name="do-leverage-the-suggest-feature-for-active-learning"></a>Tirer parti de la fonctionnalité de suggestion pour l’apprentissage actif

Utilisez régulièrement la fonctionnalité **Vérifier les énoncés du point de terminaison** de [l’apprentissage actif](luis-how-to-review-endpoint-utterances.md), au lieu d’ajouter d’autres exemples d’énoncés aux intentions. L’application reçoit constamment des énoncés du point de terminaison, ce qui allonge et fait évoluer cette liste.

## <a name="do-monitor-the-performance-of-your-app"></a>Analyser les performances de l’application

Supervisez la précision des prédictions à l’aide d’un jeu de [tests par lots](luis-concept-batch-test.md).

Conservez un jeu distinct d’énoncés ne servant pas d’[exemples d’énoncés](luis-concept-utterance.md) ou d’énoncés du point de terminaison. Continuez à améliorer l’application pour votre jeu de test. Adaptez celui-ci de façon à refléter les énoncés d’utilisateurs réels. Utilisez ce jeu de test pour évaluer chaque itération ou version de l’application.

## <a name="dont-publish-too-quickly"></a>Ne publiez pas trop rapidement

La publication trop rapide de votre application, sans [planification appropriée](#do-plan-your-schema), peut entraîner plusieurs problèmes, notamment :

* Votre application ne fonctionnera pas dans votre scénario réel à un niveau de performance acceptable.
* Le schéma (intentions et entités) ne conviendra pas et si vous avez développé une logique d’application cliente qui suit le schéma, vous devrez peut-être la réécrire depuis le début. Cela entraînerait des retards inattendus et un coût supplémentaire pour le projet sur lequel vous travaillez.
* Les énoncés que vous ajoutez au modèle peuvent entraîner un biais vers l’ensemble d’énoncés en exemple difficile à déboguer et à identifier. Cela complique également la suppression de l’ambiguïté une fois que vous avez validé un certain schéma.

## <a name="dont-add-many-example-utterances-to-intents"></a>Ne pas ajouter de nombreux exemples d’énoncés aux intentions

Une fois l’application publiée, ajoutez seulement des énoncés à partir de l’apprentissage actif au processus de cycle de développement. S’ils sont trop proches, ajoutez un modèle.

## <a name="dont-use-few-or-simple-entities"></a>Utiliser peu d’entités ou des entités simples

Des entités sont générées pour l’extraction et la prédiction de données. Il est important que chaque intention contienne des entités de machine-learning qui décrivent les données dans l’intention. Cela permet à LUIS de prédire l’intention, même si votre application cliente n’a pas besoin d’utiliser l’entité extraite.

## <a name="dont-use-luis-as-a-training-platform"></a>Ne pas utiliser LUIS comme plateforme d’apprentissage

LUIS est propre au domaine d’un modèle de langage. Il n’est pas conçu pour fonctionner comme une plateforme générale d’entraînement en langage naturel.

## <a name="dont-add-many-example-utterances-of-the-same-format-ignoring-other-formats"></a>Ne pas ajouter de nombreux exemples d’énoncés du même format, en ignorant les autres formats

LUIS attend des variations dans les énoncés d’une intention. Les énoncés peuvent différer (longueur, choix et place des mots notamment) tout en gardant la même signification générale.

|N’utilisez pas le même format|Utilisez des formats variables|
|--|--|
|Acheter un billet pour Seattle<br>Acheter un billet pour Paris<br>Acheter un billet pour Orlando|Acheter 1 billet pour Seattle<br>Réserver deux sièges pour le vol de nuit à destination de Paris lundi prochain<br>J’aimerais réserver 3 billets pour Orlando pour les vacances de printemps|

La deuxième colonne utilise différents verbes (acheter, réserver), différentes quantités (1, deux, 3) et différents ordres des mots, mais tous les énoncés ont la même intention : acheter des billets d’avion pour voyager.

## <a name="dont-mix-the-definition-of-intents-and-entities"></a>Ne pas mélanger la définition des intentions et des entités

Créez une intention pour chaque action du bot. Utilisez des entités comme des paramètres qui rendent cette action possible.

Pour un bot qui réserve des vols, créez une intention **BookFlight**. Ne créez pas une intention pour chaque compagnie aérienne ou chaque destination. Utilisez ces éléments de données comme [entités](luis-concept-entity-types.md) et marquez-les dans les exemples d’énoncés.

## <a name="dont-create-phrase-lists-with-all-the-possible-values"></a>Ne pas créer des listes d’expressions avec toutes les valeurs possibles

Donnez quelques exemples dans les [listes d’expressions](luis-concept-feature.md), mais pas tous les mots ou expressions. LUIS généralise et tient compte du contexte.

## <a name="dont-add-many-patterns"></a>Ne pas ajouter de nombreux modèles

Évitez d’ajouter trop de [modèles](luis-concept-patterns.md). LUIS est conçu pour apprendre rapidement avec peu d’exemples. Ne surchargez pas inutilement le système.

## <a name="dont-train-and-publish-with-every-single-example-utterance"></a>Ne pas effectuer l’apprentissage et publier à chaque ajout d’exemple d’énoncé

Ajoutez 10 ou 15 énoncés avant de passer à l’apprentissage et à la publication. Vous pourrez ainsi en voir l’impact sur la précision des prédictions. L’ajout d’un seul énoncé n’aura peut-être pas d’incidence visible sur le score.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [planifier votre application](luis-how-plan-your-app.md) dans votre application LUIS.