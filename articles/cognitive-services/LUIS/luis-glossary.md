---
title: Glossaire - LUIS
description: Le glossaire explique les termes que vous pourriez rencontrer en utilisant le service API LUIS.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 05/08/2020
ms.openlocfilehash: cc7dac3767ca5c9e2429f4691a458f88ad1af707
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92151977"
---
# <a name="language-understanding-glossary-of-common-vocabulary-and-concepts"></a>Glossaire Language Understanding des termes et concepts courants
Ce glossaire décrit les termes que vous pourriez rencontrer lors de l’utilisation de LUIS (Language Understanding Intelligent Service).

## <a name="active-version"></a>Version active

La version active est la [version](luis-how-to-manage-versions.md) de votre application qui est mise à jour lorsque vous apportez des modifications au modèle à l’aide du portail LUIS. Dans le portail LUIS, si vous souhaitez modifier une version qui n’est pas la version active, vous devrez tout d’abord la définir comme étant la version active.

## <a name="active-learning"></a>Apprentissage actif

L’apprentissage actif est une technique de Machine Learning dans laquelle le modèle d’apprentissage automatique est utilisé pour identifier les nouveaux exemples informatifs à étiqueter. Dans LUIS, l’apprentissage actif désigne le fait d’améliorer votre modèle en ajoutant des énoncés provenant du trafic de point de terminaison dont les prédictions actuelles ne sont pas claires. Cliquez sur « Réviser les énoncés de point de terminaison » pour afficher les énoncés à étiqueter.

Voir aussi :
* [Informations conceptuelles](luis-concept-review-endpoint-utterances.md)
* [Didacticiel sur la révision des énoncés de point de terminaison](luis-tutorial-review-endpoint-utterances.md)
* Comment améliorer l’application LUIS en [révisant les énoncés de point de terminaison](luis-how-to-review-endpoint-utterances.md)

## <a name="application-app"></a>Application (.app)

Dans LUIS, votre application est une collection de modèles issus du Machine Learning, basés sur le même jeu de données, qui fonctionnent ensemble pour prédire des intentions et des entités dans le cadre d’un scénario précis. Chaque application a son propre point de terminaison de prédiction.

Si vous créez un bot de RH, vous pouvez avoir un ensemble d’intentions, telles que « planifier les congés », « obtenir des informations sur les prestations » et « mettre à jour les informations personnelles », ainsi que les entités de chacune de ces intentions que vous regroupez dans une seule application.

## <a name="authoring"></a>Création

La création correspond à la possibilité de créer, de gérer et de déployer une application LUIS, soit avec le portail LUIS, soit avec les API de création.

### <a name="authoring-key"></a>Clé de création

La [clé de création](luis-how-to-azure-subscription.md) est utilisée pour créer l’application. et non pour les requêtes du point de terminaison au niveau de la production. Pour plus d'informations, voir [Limites des clés](luis-limits.md#key-limits).

### <a name="authoring-resource"></a>Ressource de création

Votre [ressource de création](luis-how-to-azure-subscription.md#azure-resources-for-luis) LUIS est un élément gérable disponible via Azure. La ressource est votre accès aux capacités de création, de formation et de publication associées du service Azure. La ressource comprend les informations d’authentification, d’autorisation et de sécurité dont vous avez besoin pour accéder au service Azure associé.

La ressource de création correspond au « genre » Azure `LUIS-Authoring`.

## <a name="batch-test"></a>Tests par lot

Les tests par lot permettent de valider un modèle actuel d’application LUIS avec un ensemble cohérent et connu d’énoncés utilisateurs. Le test par lot est défini dans un [fichier au format JSON](luis-concept-batch-test.md#batch-file-format).

Voir aussi :
* [Concepts](luis-concept-batch-test.md)
* [Procédure](luis-how-to-batch-test.md) : exécuter un test par lot
* [Didacticiel](luis-tutorial-batch-testing.md) : créer et exécuter des tests par lot

### <a name="f-measure"></a>F-mesure

Il s’agit d’une mesure de précision d’un test par lots.

### <a name="false-negative-fn"></a>Faux négatif (FN)

Dans les tests par lots, les points de données représentent les énoncés dans lesquels votre application a mal prédit l’absence de l’intention/entité cible.

### <a name="false-positive-fp"></a>Faux positif (FP)

Dans les tests par lots, les points de données représentent les énoncés dans lesquels votre application a mal prédit l’existence de l’intention/entité cible.

### <a name="precision"></a>Precision
Dans les tests par lots, la précision (également appelée coefficient de prévision d'un test positif) est la part d’énoncés pertinents parmi les énoncés récupérés.

Si l’on prend un exemple dans le domaine de l’élevage, un test par lots correspond au nombre de moutons prédits divisé par le nombre total d’animaux (moutons et non-mouton).

### <a name="recall"></a>Rappel

Dans les tests par lots, le rappel (également appelé sensibilité) correspond à la capacité de généralisation de LUIS.

Si l’on prend un exemple dans le domaine de l’élevage, un test par lots correspond au nombre de moutons prédits divisé par le nombre total de moutons disponibles.

### <a name="true-negative-tn"></a>Vrai négatif (VN)

Un vrai négatif se produit lorsque votre application prédit correctement une absence de correspondance. Dans les tests par lot, un vrai négatif se produit lorsque votre application prédit une intention ou une entité pour un exemple qui n’a pas été étiqueté avec cette intention ou entité.

### <a name="true-positive-tp"></a>Vrai positif (VP)

Vrai positif (VP) se produit lorsque votre application prédit correctement une correspondance. Dans les tests par lot, un vrai positif se produit lorsque votre application prédit une intention ou une entité pour un exemple qui a été étiqueté avec cette intention ou entité.

## <a name="classifier"></a>Classifieur

Un classifieur est un modèle d’apprentissage automatique qui prédit la catégorie ou la classe dans laquelle une entrée s’adapte.

Une [intention](#intent) est un exemple de classifieur.

## <a name="collaborator"></a>Collaborateur

Le concept d’un collaborateur est le même que celui d’un [contributeur](#contributor). Un collaborateur reçoit l’accès lorsqu’un propriétaire ajoute l’adresse email du collaborateur à une application qui n’est pas contrôlée par le contrôle d’accès en fonction du rôle Azure (ou Azure RBAC, pour « Role-Based Access Control »). Si vous utilisez toujours des collaborateurs, vous devez migrer votre compte LUIS et utiliser les ressources de création de LUIS pour gérer les contributeurs à l’aide d’Azure RBAC.

## <a name="contributor"></a>Contributeur

Un contributeur n’est pas le [propriétaire](#owner) de l’application, mais il dispose des mêmes autorisations pour ajouter, modifier et supprimer les intentions, les entités et les énoncés. Un contributeur fournit un contrôle d’accès en fonction du rôle Azure (Azure RBAC) à une application LUIS.

Voir aussi :
* [Découvrez comment](luis-how-to-collaborate.md#add-contributor-to-azure-authoring-resource) ajouter des contributeurs

## <a name="descriptor"></a>Descripteur

Un descripteur est le terme utilisé précédemment pour une [caractéristique](#features) Machine Learning.

## <a name="domain"></a>Domain

Dans le contexte de LUIS, un domaine désigne un domaine de connaissances Votre domaine est propre à votre scénario. Chaque domaine a sa langue et sa terminologie, qui ont une signification dans son contexte. Par exemple, si vous créez une application pour écouter de la musique, votre application utilisera des termes et une langue propres au monde de la musique : « chanson, piste, album, paroles, face b, artiste ». Pour obtenir des exemples de domaines, consultez l’article sur les [domaines prédéfinis](#prebuilt-domain).

## <a name="endpoint"></a>Point de terminaison

### <a name="authoring-endpoint"></a>Point de terminaison de création

L’URL du point de terminaison de création LUIS correspond à l’endroit où vous créez, formez et publiez votre application. Elle contient la région ou le sous-domaine personnalisé de l’application publiée, ainsi que l’ID de l’application.

Pour en savoir plus sur la création de votre application par programmation, consultez nos [références pour les développeurs](developer-reference-resource.md#rest-endpoints)

### <a name="prediction-endpoint"></a>Point de terminaison de prédiction

L’URL du point de terminaison de prédiction de LUIS est celle à laquelle les requêtes LUIS sont envoyées après création et publication de l’[application LUIS](#application-app). Elle contient la région ou le sous-domaine personnalisé de l’application publiée, ainsi que l’ID de l’application. Vous trouverez le point de terminaison dans la page **[Azure resources](luis-how-to-azure-subscription.md)** (Ressources Azure) de votre application, ou vous pouvez obtenir l’URL de point de terminaison à partir de l’API [Get application info](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c37) (Obtenir des informations sur l’application).

Votre accès au point de terminaison de prédiction est autorisé avec la clé de prédiction de LUIS.

## <a name="entity"></a>Entité

Les [entités](luis-concept-entity-types.md) sont des mots dans des énoncés qui décrivent les informations utilisées pour traiter ou identifier une intention. Si votre entité est complexe et que vous souhaitez que votre modèle identifie des parties spécifiques, vous pouvez diviser votre modèle en sous-entités. Par exemple, vous pouvez définir votre modèle pour qu’il puisse prédire une adresse, mais aussi les sous-entités rue, ville, région et code postal. Les entités peuvent également être utilisées en tant que caractéristiques pour les modèles. Votre réponse à partir de l’application LUIS inclura à la fois les intentions prédites et toutes les entités.

### <a name="entity-extractor"></a>Extracteur d’entité

Un extracteur d’entité, ou simplement « extracteur », est le type de modèle issu du Machine Learning utilisé par LUIS pour prédire les entités.

### <a name="entity-schema"></a>Schéma d’entité

Le schéma d’entité est la structure que vous définissez pour les entités issues du Machine Learning avec des sous-entités. Le point de terminaison de prédiction retourne toutes les entités et sous-entités extraites définies dans le schéma.

### <a name="entitys-subentity"></a>Sous-entité d’entité

Une sous-entité est une entité enfant d’une entité de machine-learning.

### <a name="non-machine-learning-entity"></a>Entité non-machine-learning

Entité qui utilise la correspondance de texte pour extraire les données :
* Entité de liste
* Entité d’expression régulière

### <a name="list-entity"></a>Entité de liste

Une [entité de liste](reference-entity-list.md) représente un ensemble fixe, fermé de mots associés, ainsi que leurs synonymes. Les entités de liste sont des correspondances exactes, contrairement aux entités issues du Machine Learning.

L’entité sera prédite si un mot de l’entité de liste est inclus dans la liste. Par exemple, si vous avez une entité de liste appelée « taille » et que vous avez les mots « petit, moyen, grand » dans la liste, l’entité Taille est prédite pour tous les énoncés où les mots « petit », « moyen» ou « grand » sont utilisés, quel que soit le contexte.

### <a name="regular-expression"></a>Expression régulière

Une [entité d’expression régulière](reference-entity-regular-expression.md) représente une expression régulière. Les entités d’expressions régulières sont des correspondances exactes, contrairement aux entités issues du Machine Learning.
### <a name="prebuilt-entity"></a>Entité prédéfinie

Consultez l’entrée Modèle prédéfini pour [entité prédéfinie](#prebuilt-entity)

## <a name="features"></a>Fonctionnalités

Dans le Machine Learning, une caractéristique est un trait typique qui aide le modèle à reconnaître un concept particulier. Il s’agit d’une indication que LUIS peut utiliser, mais ce n’est pas une règle absolue.

On parle également de **[fonctionnalité de machine learning](luis-concept-feature.md)** .

Ces indicateurs sont utilisés conjointement avec les étiquettes pour apprendre à prédire de nouvelles données. LUIS prend en charge les listes d’expressions et l’utilisation d’autres modèles en tant que caractéristiques.

### <a name="required-feature"></a>Caractéristique requise

Une caractéristique requise est un moyen de contraindre la sortie d’un modèle LUIS. Quand une caractéristique pour une entité est marquée comme étant obligatoire, la caractéristique doit être présente dans l’exemple pour que l’entité soit prédite, indépendamment de ce que prédit le modèle issu du Machine Learning.

Prenons un exemple dans lequel vous disposez d’une caractéristique prédéfinie que vous avez marquée comme étant requise sur l’entité « quantité » pour un bot de commande de menu. Lorsque votre bot voit `I want a bajillion large pizzas?`, « bajillion » n’est pas prédit comme une quantité, quel que soit le contexte dans lequel il apparaît. « Bajillion » n’est pas un nombre valide et ne sera pas prédit par l’entité de nombre prédéfinie.

## <a name="intent"></a>Intentionnel

Une [intention](luis-concept-intent.md) représente une tâche ou une action que l’utilisateur souhaite effectuer. Il s’agit d’un but ou d’un objectif exprimé dans l’énoncé d’un utilisateur, par exemple réserver un vol ou régler une facture. Dans LUIS, l’ensemble d’un énoncé est considéré comme une intention, mais les parties de l’énoncé sont extraites en tant qu’entités

## <a name="labeling-examples"></a>Exemples d’étiquetage

L’étiquetage, ou marquage, est le processus qui consiste à associer un exemple positif ou négatif à un modèle.

### <a name="labeling-for-intents"></a>Étiquetage des intentions
Dans LUIS, les intentions dans une application s’excluent mutuellement. Cela signifie que lorsque vous ajoutez un énoncé à une intention, il est considéré comme un exemple _positif_ à cet intention et un exemple _négatif_ pour toutes les autres intentions. Les exemples négatifs ne doivent pas être confondus avec l’intention « None », qui représente les énoncés qui se trouvent en dehors de l’étendue de l’application.

### <a name="labeling-for-entities"></a>Étiquetage des entités
Dans LUIS, vous [étiquetez](label-entity-example-utterance.md) un mot ou une expression dans l’énoncé d’un exemple d’intention avec une entité en tant qu’exemple _positif_ . L’étiquetage indique l’intention qu’il doit prédire pour cet énoncé. Les énoncés étiquetés sont utilisés pour former l’intention.

## <a name="luis-app"></a>Application LUIS

Consultez la définition d’[application](#application-app).

## <a name="model"></a>Modèle

Un modèle (issu du Machine Learning) est une fonction qui effectue une prédiction sur les données d’entrée. Dans LUIS, nous faisons référence aux classifieurs d’intention et aux extracteurs d’entités de façon générique comme des « modèles », et nous faisons référence à une collection de modèles qui sont formés, publiés et interrogés ensemble comme une « application ».

## <a name="normalized-value"></a>Valeur normalisée

Vous ajoutez des valeurs à vos entités de [liste](#list-entity). Chacune de ces valeurs peut avoir une liste d’un ou plusieurs synonymes. Seule la valeur normalisée est retournée dans la réponse.

## <a name="overfitting"></a>Surajustement

Un surajustement se produit quand le modèle est fixé sur les exemples spécifiques et qu’il n’est pas possible de le généraliser correctement.

## <a name="owner"></a>Propriétaire

Chaque application possède un propriétaire, à savoir la personne qui a créé l’application. Le propriétaire gère les autorisations de l’application dans le Portail Azure.

## <a name="phrase-list"></a>Liste d’expressions

Une [liste d’expressions](luis-concept-feature.md) est un type de caractéristique Machine Learning qui inclut un groupe de valeurs (mots ou expressions) qui appartiennent à la même classe et doivent être traitées de la même façon (par exemple, des noms de villes ou de produits).

## <a name="prebuilt-model"></a>Modèle prédéfini

Un [modèle prédéfini](luis-concept-prebuilt-model.md) est une intention, une entité ou une collection des deux, ainsi que des exemples étiquetés. Ces modèles prédéfinis courants peuvent être ajoutés à votre application pour réduire le travail de développement de modèle requis pour votre application.

### <a name="prebuilt-domain"></a>Domaine prédéfini

Un domaine prédéfini est une application LUIS configurée pour un domaine particulier, par exemple, la domotique (HomeAutomation) ou les réservations de restaurants (RestaurantReservation). Les intentions, les énoncés et les entités sont configurés pour ce domaine.

### <a name="prebuilt-entity"></a>Entité prédéfinie

Une entité prédéfinie est une entité fournie par LUIS pour les types d’informations courants, comme le nombre, l’URL et l’adresse électronique. Celles-ci sont créées en fonction des données publiques. Vous pouvez choisir d’ajouter une entité prédéfinie en tant qu’entité autonome ou en tant que caractéristique d’une entité

### <a name="prebuilt-intent"></a>Intention prédéfinie

Une intention prédéfinie est un objectif que LUIS fournit pour des types d’information courants et qui possède ses propres exemples d’énoncés étiquetés.

## <a name="prediction"></a>Prédiction

Une prédiction est une requête REST au service de prédiction Azure LUIS qui accepte de nouvelles données (énoncé utilisateur) et alimente l’application formée et publiée avec ces données pour déterminer les intentions et les entités trouvées.

### <a name="prediction-key"></a>Clé de prédiction

La [clé de prédiction](luis-how-to-azure-subscription.md) (précédemment appelée clé d’abonnement) est la clé associée au service LUIS que vous avez créé dans Azure, qui autorise votre utilisation du point de terminaison de prédiction.

Il ne s’agit pas de la clé de création. Si vous avez une clé de point de terminaison de prédiction, vous devrez l’utiliser pour toutes les demandes du point de terminaison au lieu de la clé de création. Vous pouvez voir votre clé de prédiction actuelle au sein de l’URL du point de terminaison en bas de la page des ressources Azure sur le site web LUIS. Il s’agit de la valeur de la paire nom/valeur subscription-key.

### <a name="prediction-resource"></a>Ressource de prédiction

Votre ressource de prédiction LUIS est un élément gérable qui est disponible via Azure. La ressource est votre accès à la prédiction associée du service Azure. La ressource comprend des prédictions.

La ressource de prédiction correspond au « genre » Azure `LUIS`.

### <a name="prediction-score"></a>Scores de prédictions

Le [score](luis-concept-prediction-score.md) est un nombre compris entre 0 et 1. Il mesure la confiance du système par rapport au fait qu’un énoncé d’entrée puisse correspondre à une intention. Plus le score est proche de 1, plus le système est confiant en ce qui concerne sa sortie. Plus le score est proche de 0, plus le système est certain que l’entrée ne correspond pas à une sortie attendue. Les scores entre ses deux valeurs signifient que le système ne sait pas vraiment quelle décision prendre.

Prenons l’exemple d’un modèle utilisé pour identifier si un texte provenant d’un client correspond à une commande alimentaire. Il peut attribuer un score de 1 pour « j’aimerais commander un café » (le système est certain qu’il s’agit d’une commande) et un score de 0 pour « mon équipe a remporté le match hier soir » (le système est certain qu’il ne s’agit pas d’une commande). Et il peut avoir un score de 0,5 pour « prenons le thé » (il ne sait pas s’il s’agit d’une commande).

## <a name="programmatic-key"></a>Clé programmatique

Renommée [clé de création](#authoring-key).

## <a name="publish"></a>Publish

La [publication](luis-how-to-publish-app.md) consiste à rendre disponible une version active de LUIS sur le [point de terminaison](#endpoint) de mise en lots ou de production.

## <a name="quota"></a>Quota

Le quota LUIS correspond à la limitation du niveau d’abonnement Azure. Il peut être exprimé en demandes par seconde (état HTTP 429), en nombre total de demandes par mois (état HTTP 403) ou les deux.

## <a name="schema"></a>schéma

Votre schéma inclut vos intentions et entités, ainsi que les sous-entités. Le schéma est initialement planifié, puis itéré dans le temps. Le schéma n’inclut pas les paramètres d’application, les caractéristiques ou les exemples de énoncés.

## <a name="sentiment-analysis"></a>Analyse des sentiments
L’analyse des sentiments attribue des valeurs positives ou négatives aux énoncés fournis par [l’Analyse de texte](../text-analytics/overview.md).

## <a name="speech-priming"></a>Préparation vocale

La préparation vocale améliore la reconnaissance des mots prononcés et des expressions couramment utilisés dans votre scénario avec [Speech Services](../speech-service/overview.md). Pour les applications avec fonctions de préparation vocale, tous les exemples de LUIS étiquetés sont utilisés pour améliorer la précision de la reconnaissance vocale en créant un modèle vocal personnalisé pour cette application spécifique. Par exemple, pour une partie d’échec, il faut être sûr que lorsque l’utilisateur dit « Déplacer la reine », la phrase ne sera pas interprétée comme « Déplacer l’arène ». L’application LUIS doit inclure des exemples dans lesquels le mot « reine » est étiqueté en tant qu’entité.

## <a name="starter-key"></a>Clé de démarrage

Une clé libre à utiliser lors du premier démarrage à l’aide de LUIS.

## <a name="synonyms"></a>Synonymes

Vous pouvez attribuer une valeur normalisée aux [entités de listes](reference-entity-list.md) de LUIS pour les doter d’une liste de synonymes. Par exemple, vous pouvez créer une entité « taille » qui a comme valeurs normalisées « petite », « moyenne », « grande » et « très grande ». Vous pouvez ensuite créer des synonymes pour chaque valeur comme suit :

|Valeur normalisée| Synonymes|
|--|--|
|Petite| la petite, S|
|Moyenne| normale, M|
|grand| large, L|
|Très grande| la plus grande, XL|

Le modèle retourne la valeur normalisée de l’entité lorsque l’un des synonymes est visible dans l’entrée.

## <a name="test"></a>Test

[Tester](luis-concept-test.md) une application LUIS signifie afficher des prédictions de modèle.

## <a name="timezone-offset"></a>Décalage de fuseaux horaires

Le point de terminaison comporte le paramètre [timezoneOffset](luis-concept-data-alteration.md#change-time-zone-of-prebuilt-datetimev2-entity). qui correspond au nombre de minutes à ajouter ou supprimer de l’entité prédéfinie datetimeV2. Par exemple, si l’énoncé est « Quelle heure est-il maintenant ? », la valeur datetimeV2 retournée est l’heure actuelle de la demande du client. Si la demande de client provient d’un bot ou de toute autre application différente de l’utilisateur de votre bot, vous devrez indiquer le décalage entre le bot et l’utilisateur.

Voir [Changer le fuseau horaire de l’entité datetimeV2 prédéfinie](luis-concept-data-alteration.md?#change-time-zone-of-prebuilt-datetimev2-entity).

## <a name="token"></a>par jeton
Un [jeton](luis-language-support.md#tokenization) est la plus petite unité de texte que LUIS peut reconnaître. Il diffère légèrement d’une langue à l’autre.

Pour l’ **anglais** , un jeton est une étendue continue (sans espaces ni ponctuation) de lettres et de chiffres. Un espace N’EST PAS un jeton.

|Expression|Nombre de jetons|Explication|
|--|--|--|
|`Dog`|1|Mot unique sans ponctuation ni espace.|
|`RMT33W`|1|Numéro de localisateur d’enregistrement. Elle peut comporter des chiffres et des lettres, mais elle n’a pas de ponctuation.|
|`425-555-5555`|5|Un numéro de téléphone Chaque signe de ponctuation est un jeton unique, donc `425-555-5555` correspond à 5 jetons :<br>`425`<br>`-`<br>`555`<br>`-`<br>`5555` |
|`https://luis.ai`|7|`https`<br>`:`<br>`/`<br>`/`<br>`luis`<br>`.`<br>`ai`<br>|

## <a name="train"></a>Former

La [formation](luis-how-to-train.md) est le processus consistant à enseigner à LUIS toutes les modifications apportées à la version active depuis la dernière formation.

### <a name="training-data"></a>Données de formation

Les données de formation correspondent à l’ensemble d’informations nécessaires pour former un modèle. Elles comprennent le schéma, les énoncés étiquetés, les caractéristiques et les paramètres d’application.

### <a name="training-errors"></a>Erreurs de formation

Les erreurs de formation sont des prédictions sur vos données d’apprentissage qui ne correspondent pas à leurs étiquettes.

## <a name="utterance"></a>Énoncé

Un [énoncé](luis-concept-utterance.md) est une entrée d’utilisateur qui représente un texte bref représentatif d’une phrase dans une conversation. Un énoncé est une expression en langage naturel comme « Réserve 2 billets pour le match OM-PSG de mardi prochain ». Des exemples d’énoncés sont ajoutés pour former le modèle et le modèle établit sa prédiction en fonction d’un nouvel énoncé au moment de l’exécution

## <a name="version"></a>Version

Une [version](luis-how-to-manage-versions.md) de LUIS est une instance spécifique d’une application LUIS associée à un ID d’application LUIS et au point de terminaison publié. Chaque application LUIS a au moins une version.
