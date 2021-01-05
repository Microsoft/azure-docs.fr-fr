---
title: Test pour DevOps pour les applications LUIS
description: Comment tester votre application Language Understanding (LUIS) dans un environnement DevOps.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 06/3/2020
ms.openlocfilehash: f002bfdd5aeb784b5b10b549389e663216fa0361
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97561220"
---
# <a name="testing-for-luis-devops"></a>Tests pour LUIS dans DevOps

Les ingénieurs logiciels qui développent une application Language Understanding (LUIS) peuvent appliquer des pratiques DevOps sur le [contrôle de code source](luis-concept-devops-sourcecontrol.md), [des builds automatisés](luis-concept-devops-automation.md), de [test](luis-concept-devops-testing.md) et de [gestion des mises en production](luis-concept-devops-automation.md#release-management) en suivant ces consignes.

Dans les méthodologies Agile software development, le test joue un rôle essentiel dans la création de logiciels de qualité. Chaque modification importante apportée à une application LUIS doit être accompagnée de tests des nouvelles fonctionnalités que le développeur crée dans l’application. Ces tests sont vérifiés dans votre référentiel de code source, ainsi que la source de `.lu` de votre application LUIS. L’implémentation de la modification est terminée lorsque l’application valide les tests.

Les tests constituent une partie essentielle des [flux de travail CI/CD](luis-concept-devops-automation.md). Lorsque les modifications d’une application LUIS sont proposées dans une demande de tirage (PR) ou une fois que les modifications ont été fusionnées à votre branche principale, les flux de travail CI doivent exécuter les tests pour vérifier que les mises à jour n’ont pas entraîné de régression.

## <a name="how-to-do-unit-testing-and-batch-testing"></a>Comment effectuer des tests unitaires et des tests par lots

Il existe deux types de tests pour une application LUIS à exécuter dans les flux de travail d’intégration continue :

- Les **tests unitaires** qui sont relativement simples et vérifient la fonctionnalité clé de votre application LUIS. Un test unitaire est validé lorsque l’intention prévue et les entités attendues sont retournées pour un énoncé de test donné. Tous les tests unitaires doivent être validés pour que la série de tests se termine correctement.  
Ce type de test est similaire aux [tests interactifs](./luis-concept-test.md) que vous pouvez effectuer dans le [portail LUIS](https://www.luis.ai/).

- Les **tests par lot** sont plus complets et concernent votre modèle en cours d’apprentissage afin d’en mesurer les performances. Contrairement aux tests unitaires, les tests par lot ne sont pas validés ou bloqués. Les tests par lot ne retournent pas forcément l’intention et les entités attendues. Au lieu de cela, un test par lot vous aide à afficher la précision de chaque intention et entité dans votre application et vous aide à effectuer des comparaisons au fil du temps à mesure que vous apportez des améliorations.  
Ce type de test est le même que les [tests par lot](./luis-concept-batch-test.md) que vous pouvez effectuer de manière interactive dans le portail LUIS.

Vous pouvez utiliser des tests unitaires dès le début de votre projet. Les tests par lots ne sont vraiment utiles que lorsque vous avez développé le schéma de votre application LUIS et que vous travaillez à améliorer sa précision.

Pour les tests unitaires et les tests par lot, assurez-vous que vos énoncés de test sont conservés séparément de vos énoncés d’apprentissage. Si vous testez sur les mêmes données que celles sur lesquelles vous effectuez l’apprentissage, vous obtiendrez la fausse impression que votre application fonctionne correctement lorsqu’elle est simplement en train de s’ajuster aux données de test. Les tests doivent être invisibles pour le modèle afin de vérifier la qualité de sa généralisation.

### <a name="writing-tests"></a>Écriture des tests

Lorsque vous écrivez un ensemble de tests, vous devez définir pour chaque test :

* Énoncé de test
* Intention attendue
* Entités attendues.

Utilisez la [syntaxe du fichier de lot LUIS](./luis-concept-batch-test.md#batch-syntax-template-for-intents-with-entities) pour définir un groupe de tests dans un fichier au format JSON. Par exemple :

```JSON
[
  {
    "text": "example utterance goes here",
    "intent": "intent name goes here",
    "entities":
    [
        {
            "entity": "entity name 1 goes here",
            "startPos": 14,
            "endPos": 23
        },
        {
            "entity": "entity name 2 goes here",
            "startPos": 14,
            "endPos": 23
        }
    ]
  }
]
```

Certains outils de test, tels que [NLU. DevOps](https://github.com/microsoft/NLU.DevOps) peuvent également prendre en charge les fichiers de test au format LUDown.

#### <a name="designing-unit-tests"></a>Conception des tests unitaires

Les tests unitaires doivent être conçus pour tester les fonctionnalités principales de votre application LUIS. Dans chaque itération, ou sprint, du développement de votre application, vous devez écrire un nombre suffisant de tests pour vérifier que la fonctionnalité clé que vous implémentez dans cette itération fonctionne correctement.

Dans chaque test unitaire, pour un énoncé de test donné, vous pouvez :

* Vérifier que l’intention correcte est retournée.
* Tester que les entités « clé », c’est-à-dire celles qui sont critiques pour votre solution, sont retournées.
* Tester que le [score de prédiction](./luis-concept-prediction-score.md) pour l’intention et les entités dépasse le seuil que vous définissez. Par exemple, vous pouvez décider de considérer un test réussi si le score de prédiction pour l’intention et pour vos entités clés dépasse 0,75.

Pour les tests unitaires, il est judicieux de vérifier que vos entités clés ont été retournées dans la réponse de prédiction, mais d’ignorer les faux positifs. Les *faux positifs* sont des entités qui se trouvent dans la réponse de prédiction, mais qui ne sont pas définies dans les résultats attendus pour votre test. En ignorant les faux positifs, il est moins coûteux de créer des tests unitaires, tout en vous permettant de vous concentrer sur les tests que les données clés de votre solution sont renvoyées dans une réponse de prédiction.

> [!TIP]
> L’outil [NLU. DevOps](https://github.com/microsoft/NLU.DevOps) prend en charge tous vos besoins en matière de test LUIS. La commande `compare` lorsqu’elle est utilisée en [mode de test unitaire](https://github.com/microsoft/NLU.DevOps/blob/master/docs/Analyze.md#unit-test-mode) déclare que tous les tests réussissent et ignore les faux positifs pour les entités qui ne sont pas étiquetées dans les résultats attendus.

#### <a name="designing-batch-tests"></a>Conception de tests par lot

Les ensembles de tests par lot doivent contenir un grand nombre de cas de test, conçus pour effectuer des tests sur toutes les intentions et toutes les entités de votre application LUIS. Pour plus d’informations sur la définition d’un ensemble de tests par lot, consultez la section [Tests par lot dans le portail LUIS](./luis-concept-batch-test.md).

### <a name="running-tests"></a>Exécution des tests

Le portail LUIS offre des fonctionnalités pour faciliter les tests interactifs :

* Les [**tests interactifs**](./luis-concept-test.md) vous permettent de soumettre un exemple d’énoncé et d’obtenir une réponse à des intentions et des entités reconnues par LUIS. Vous pouvez vérifier si le test est validé par une simple inspection visuelle.

* Les [**tests par lot**](./luis-concept-batch-test.md) utilisent un fichier de test par lot comme source pour valider votre version entraînée active afin de mesurer la précision des prédictions. Un test par lot vous permet de visualiser la précision de chaque intention et entité dans votre version active, affichant ainsi les résultats avec un graphique.

#### <a name="running-tests-in-an-automated-build-workflow"></a>Exécution des tests dans un flux de travail de génération automatisé

Les fonctionnalités de test interactif dans le portail LUIS sont utiles, mais pour DevOps, les tests automatisés exécutés dans un flux de travail CI/CD présentent certaines exigences :

* Les outils de test doivent fonctionner dans une étape de flux de travail sur un serveur de génération de flux. Cela signifie que les outils doivent pouvoir s’exécuter en ligne de commande.
* Les outils de test doivent être en mesure d’exécuter un groupe de tests sur un point de terminaison et de vérifier automatiquement les résultats attendus par rapport aux résultats réels.
* Si les tests échouent, les outils de test doivent retourner un code d’état pour arrêter le flux de travail et « faire échouer la génération ».

LUIS n’offre pas d’outil en ligne de commande ou d’API de haut niveau qui offre ces fonctionnalités. Nous vous recommandons d’utiliser l’outil [NLU.DevOps](https://github.com/microsoft/NLU.DevOps) pour exécuter des tests et vérifier les résultats, à la fois en ligne de commande et pendant les tests automatisés au sein d’un flux de travail CI/CD.

Les capacités de test disponibles sur le portail LUIS ne nécessitent pas la publication d'un point de terminaison et font partie des capacités de création de LUIS. Lorsque vous implémentez des tests dans un flux de travail de génération automatisé, vous devez publier la version de l’application LUIS à tester sur un point de terminaison afin d’utiliser des outils de test tels que NLU.DevOps pour envoyer des demandes de prédiction dans le cadre du test.

> [!TIP]
> * Si vous implémentez votre propre solution de test et que vous écrivez du code pour envoyer des énoncés de test à un point de terminaison, n’oubliez pas que si vous utilisez la clé de création LUIS, le taux de transactions autorisé est limité à 5 TPS. Limitez la vitesse d’envoi ou utilisez une clé de prédiction à la place.
> * Lors de l’envoi de requêtes de test à un point de terminaison, n’oubliez pas d’utiliser `log=false` dans la chaîne de requête de votre demande de prédiction. Cela permet de s’assurer que vos énoncés de test ne sont pas enregistrés par LUIS et finissent dans la liste de vérification des énoncés de point de terminaison présentée par la fonctionnalité d’[apprentissage actif](./luis-concept-review-endpoint-utterances.md) de LUIS et, par conséquent, ajoutés accidentellement aux énoncés d’apprentissage de votre application.

#### <a name="running-unit-tests-at-the-command-line-and-in-cicd-workflows"></a>Exécution des tests unitaires en ligne de commande et dans les flux de travail CI/CD

Vous pouvez utiliser le package [NLU.DevOps](https://github.com/microsoft/NLU.DevOps) pour exécuter les tests en ligne de commande :

* Utilisez la [commande de test](https://github.com/microsoft/NLU.DevOps/blob/master/docs/Test.md) de NLU.DevOps pour envoyer des tests à partir d’un fichier de test à un point de terminaison et capturer les résultats de prédiction réels dans un fichier.
* Utilisez la [commande de comparaison](https://github.com/microsoft/NLU.DevOps/blob/master/docs/Analyze.md) de NLU.DevOps pour comparer les résultats réels avec les résultats attendus définis dans le fichier de test d’entrée. La commande `compare` génère une sortie de test NUnit et, lorsqu’elle est utilisée en [mode de test unitaire](https://github.com/microsoft/NLU.DevOps/blob/master/docs/Analyze.md#unit-test-mode) à l’aide de l’indicateur `--unit-test`, déclare que tous les tests réussissent.

### <a name="running-batch-tests-at-the-command-line-and-in-cicd-workflows"></a>Exécution des tests par lot en ligne de commande et dans les flux de travail CI/CD

Vous pouvez aussi utiliser le package NLU.DevOps pour exécuter les tests par lot en ligne de commande.

* Utilisez la [commande de test](https://github.com/microsoft/NLU.DevOps/blob/master/docs/Test.md) de NLU.DevOps pour envoyer des tests à partir d’un fichier de test à un point de terminaison et capturer les résultats de prédiction réels dans un fichier, de même que pour les tests unitaires.
* Utilisez la [commande de comparaison](https://github.com/microsoft/NLU.DevOps/blob/master/docs/Analyze.md) de NLU.DevOps en mode [test de performance](https://github.com/microsoft/NLU.DevOps/blob/master/docs/Analyze.md#performance-test-mode) pour mesurer les performances de votre application. Vous pouvez également comparer les performances de votre application à une référence de base, par exemple, les résultats du dernier envoi à la branche principale ou de la version actuelle. En mode test de performance, la commande `compare` génère des résultats de test NUnit et de [tests par lot](./luis-glossary.md#batch-test) au format JSON.

## <a name="luis-non-deterministic-training-and-the-effect-on-testing"></a>Apprentissage non déterministe LUIS et son effet sur les tests

Lorsque LUIS est en train d’effectuer l’apprentissage d’un modèle, comme un objectif, il a besoin de données positives : les énoncés d’apprentissage étiquetés que vous avez fournis pour les données de modèle et données négatives ne sont *pas* des exemples valides de l’utilisation de ce modèle. Au cours de l’apprentissage, LUIS crée les données négatives d’un modèle à partir de toutes les données positives que vous avez fournies pour les autres modèles, mais dans certains cas, cela peut entraîner un déséquilibre des données. Pour éviter ce déséquilibre, LUIS échantillonne un sous-ensemble des données négatives de manière non déterministe pour optimiser pour un meilleur ensemble d’apprentissage équilibré, améliorer les performances du modèle et accélérer l’apprentissage.

Le résultat de cette formation non déterministe est que vous pouvez obtenir une réponse de [prédiction légèrement différente entre les différentes sessions d’apprentissage](./luis-concept-prediction-score.md), généralement pour des intentions et/ou des entités où le [score de prédiction](./luis-concept-prediction-score.md) n'est pas élevé.

Si vous souhaitez désactiver l’apprentissage non déterministe pour les versions d’application LUIS que vous créez à des fins de test, utilisez l’[API des paramètres de version](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/versions-update-application-version-settings) avec le paramètre `UseAllTrainingData` défini sur `true`.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur l’[implémentation de flux de travail d’intégration en CI/CD](luis-concept-devops-automation.md)
* Comment [implémenter DevOps pour LUIS avec GitHub](luis-how-to-devops-with-github.md)