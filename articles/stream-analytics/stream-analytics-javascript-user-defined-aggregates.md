---
title: Agrégats JavaScript définis par l’utilisateur dans Azure Stream Analytics
description: Cet article décrit comment effectuer des mécanismes de requêtes avancées avec des agrégats JavaScript définis par l’utilisateur dans Azure Stream Analytics.
author: rodrigoaatmicrosoft
ms.author: rodrigoa
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 10/28/2017
ms.custom: devx-track-js
ms.openlocfilehash: 8891bb5cd3b8bd8ae9e02c871d6d0ffe42078c0b
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93124762"
---
# <a name="azure-stream-analytics-javascript-user-defined-aggregates"></a>Agrégats JavaScript définis par l’utilisateur Azure Stream Analytics
 
Azure Stream Analytics prend en charge les agrégats définis par l’utilisateur (UDA) écrits en JavaScript, ce qui vous permet d’implémenter une logique métier avec état complexe. Au sein de l’agrégat défini par l’utilisateur, vous avez un contrôle total sur la structure de données avec état, le cumul d’états, le non-cumul d’états et le calcul des résultats d’agrégation. Cet article présente les deux interfaces UDA JavaScript, les étapes pour créer un agrégat défini par l’utilisateur et l’emploi d’un agrégat avec des opérations basées sur une fenêtre dans une requête Stream Analytics.

## <a name="javascript-user-defined-aggregates"></a>Agrégats JavaScript définis par l’utilisateur

Un agrégat défini par l’utilisateur est employé en plus d’une spécification de fenêtre de temps pour agréger les événements de cette fenêtre et produire une valeur à résultat unique. Il existe deux types d’interfaces UDA actuellement pris en charge par Stream Analytics : AccumulateOnly et AccumulateDeaccumulate. Les deux types d’agrégats définis par l’utilisateur peuvent être utilisés par la fenêtre bascule, la fenêtre récurrente, la fenêtre glissante et la fenêtre de session. L’agrégat défini par l’utilisateur AccumulateDeaccumulate est plus performant que l’agrégat défini par l’utilisateur AccumulateOnly lorsqu’il est utilisé avec la fenêtre récurrente, la fenêtre glissante et la fenêtre de session. Vous choisissez un des deux types en fonction de l’algorithme que vous utilisez.

### <a name="accumulateonly-aggregates"></a>Agrégats AccumulateOnly

Les agrégats AccumulateOnly ne peuvent accumuler que les nouveaux événements à leur état ; l’algorithme n’autorise pas désaccumulation de valeurs. Choisissez ce type d’agrégation lorsque la désaccumulation des informations d’événement à partir de la valeur d’état est impossible à mettre en œuvre. Voici le modèle JavaScript pour les agrégats AccumulateOnly :

```JavaScript
// Sample UDA which state can only be accumulated.
function main() {
    this.init = function () {
        this.state = 0;
    }

    this.accumulate = function (value, timestamp) {
        this.state += value;
    }

    this.computeResult = function () {
        return this.state;
    }
}
```

### <a name="accumulatedeaccumulate-aggregates"></a>Agrégats AccumulateDeaccumulate

Les agrégats AccumulateDeaccumulate permettent la désaccumulation d’une valeur cumulée précédemment à partir de l’état (par exemple, supprimer une paire clé-valeur dans la liste des valeurs d’événement ou soustraire une valeur d’un état de somme d’agrégation). Voici le modèle JavaScript pour les agrégats AccumulateDeaccumulate :

```JavaScript
// Sample UDA which state can be accumulated and deaccumulated.
function main() {
    this.init = function () {
        this.state = 0;
    }

    this.accumulate = function (value, timestamp) {
        this.state += value;
    }

    this.deaccumulate = function (value, timestamp) {
        this.state -= value;
    }

    this.deaccumulateState = function (otherState){
        this.state -= otherState.state;
    }

    this.computeResult = function () {
        return this.state;
    }
}
```

## <a name="uda---javascript-function-declaration"></a>UDA - Déclaration de fonction JavaScript

Chaque UDA JavaScript est défini par une déclaration d’objet de fonction. Voici les principaux éléments d’une définition d’UDA.

### <a name="function-alias"></a>Alias de fonction

L’alias de fonction est l’identificateur de l’UDA. En cas d’appel dans une requête Stream Analytics, utilisez toujours des alias d’UDA avec le préfixe « uda.». .

### <a name="function-type"></a>Type de fonction

Pour un UDA, le type de fonction doit être **UDA Javascript**.

### <a name="output-type"></a>Type de sortie

Type particulier pris en charge par le travail Stream Analytics, ou « Tout » si vous souhaitez gérer le type dans votre requête.

### <a name="function-name"></a>Nom de la fonction

Nom de cet objet de fonction. Le nom de la fonction doit correspondre à l’alias de l’agrégat défini par l’utilisateur.

### <a name="method---init"></a>Méthode - init()

La méthode init() initialise l’état de l’agrégat. Cette méthode est appelée au démarrage de la fenêtre.

### <a name="method--accumulate"></a>Méthode – accumulate()

La méthode accumulate() calcule l’état de l’UDA en fonction de l’état précédent et des valeurs d’événement en cours. Cette méthode est appelée lorsqu’un événement entre dans une fenêtre de temps (TUMBLINGWINDOW, HOPPINGWINDOW, SLIDINGWINDOW ou SESSIONWINDOW).

### <a name="method--deaccumulate"></a>Méthode – deaccumulate()

La méthode deaccumulate() recalcule l’état en fonction de l’état précédent et des valeurs d’événement en cours. Cette méthode est appelée lorsqu’un événement quitte une fenêtre SLIDINGWINDOW ou une fenêtre SESSIONWINDOW.

### <a name="method--deaccumulatestate"></a>Méthode – deaccumulateState()

La méthode deaccumulateState() recalcule l’état en fonction de l’état précédent et de l’état d’un tronçon. Cette méthode est appelée lorsqu’un ensemble d’événements quitte une fenêtre récurrente HOPPINGWINDOW.

### <a name="method--computeresult"></a>Méthode – computeResult()

La méthode computeResult() renvoie le résultat de l’agrégat en fonction de l’état actuel. Cette méthode est appelée à la fin d’une fenêtre de temps (TUMBLINGWINDOW, HOPPINGWINDOW, SLIDINGWINDOW ou SESSIONWINDOW).

## <a name="javascript-uda-supported-input-and-output-data-types"></a>Types de données d’entrée et de sortie pris en charge pour les UDA JavaScript
Pour les types de données des UDA JavaScript, reportez-vous à la section **Conversion de type Stream Analytics et JavaScript** de la rubrique [Intégration d’UDF JavaScript](stream-analytics-javascript-user-defined-functions.md).

## <a name="adding-a-javascript-uda-from-the-azure-portal"></a>Ajout d’un UDA JavaScript à partir du portail Azure

Vous trouverez ci-dessous la procédure de création d’un UDA à partir du portail. L’exemple utilisé ici concerne le calcul de moyennes pondérées de durée.

Maintenant, nous allons créer un UDA JavaScript sous un travail ASA existant en suivant les étapes.

1. Connectez-vous au portail Azure et recherchez le travail Stream Analytics existant.
1. Cliquez ensuite sur le lien de fonctions sous **Topologie de la tâche**.
1. Cliquez sur l’icône **Ajouter** pour ajouter une nouvelle fonction.
1. Dans la vue Nouvelle fonction, sélectionnez le type de fonction **UDA JavaScript**. Vous verrez alors un modèle d’UDA par défaut apparaître dans l’éditeur.
1. Indiquez l’alias d’UDA « TWA » et modifiez l’implémentation de la fonction de la façon suivante :

    ```JavaScript
    // Sample UDA which calculate Time-Weighted Average of incoming values.
    function main() {
        this.init = function () {
            this.totalValue = 0.0;
            this.totalWeight = 0.0;
        }

        this.accumulate = function (value, timestamp) {
            this.totalValue += value.level * value.weight;
            this.totalWeight += value.weight;

        }

        // Uncomment below for AccumulateDeaccumulate implementation
        /*
        this.deaccumulate = function (value, timestamp) {
            this.totalValue -= value.level * value.weight;
            this.totalWeight -= value.weight;
        }

        this.deaccumulateState = function (otherState){
            this.state -= otherState.state;
            this.totalValue -= otherState.totalValue;
            this.totalWeight -= otherState.totalWeight;
        }
        */

        this.computeResult = function () {
            if(this.totalValue == 0) {
                result = 0;
            }
            else {
                result = this.totalValue/this.totalWeight;
            }
            return result;
        }
    }
    ```

1. Une fois que vous avez cliqué sur le bouton « Enregistrer », votre UDA s’affiche dans la liste des fonctions.

1. Cliquez sur la nouvelle fonction « TWA » pour lire sa définition.

## <a name="calling-javascript-uda-in-asa-query"></a>Appel d’un UDA JavaScript dans une requête ASA

Dans le portail Azure, ouvrez votre projet, modifiez la requête, puis appelez la fonction TWA() avec un préfixe de mandat « uda. ». Par exemple :

```SQL
WITH value AS
(
    SELECT
    NoiseLevelDB as level,
    DurationSecond as weight
FROM
    [YourInputAlias] TIMESTAMP BY EntryTime
)
SELECT
    System.Timestamp as ts,
    uda.TWA(value) as NoseDoseTWA
FROM value
GROUP BY TumblingWindow(minute, 5)
```

## <a name="testing-query-with-uda"></a>Test de requête avec UDA

Créez un fichier JSON local avec le contenu ci-dessous, téléchargez le fichier dans le travail Stream Analytics et testez la requête ci-dessus.

```JSON
[
  {"EntryTime": "2017-06-10T05:01:00-07:00", "NoiseLevelDB": 80, "DurationSecond": 22.0},
  {"EntryTime": "2017-06-10T05:02:00-07:00", "NoiseLevelDB": 81, "DurationSecond": 37.8},
  {"EntryTime": "2017-06-10T05:02:00-07:00", "NoiseLevelDB": 85, "DurationSecond": 26.3},
  {"EntryTime": "2017-06-10T05:03:00-07:00", "NoiseLevelDB": 95, "DurationSecond": 13.7},
  {"EntryTime": "2017-06-10T05:03:00-07:00", "NoiseLevelDB": 88, "DurationSecond": 10.3},
  {"EntryTime": "2017-06-10T05:05:00-07:00", "NoiseLevelDB": 103, "DurationSecond": 5.5},
  {"EntryTime": "2017-06-10T05:06:00-07:00", "NoiseLevelDB": 99, "DurationSecond": 23.0},
  {"EntryTime": "2017-06-10T05:07:00-07:00", "NoiseLevelDB": 108, "DurationSecond": 1.76},
  {"EntryTime": "2017-06-10T05:07:00-07:00", "NoiseLevelDB": 79, "DurationSecond": 17.9},
  {"EntryTime": "2017-06-10T05:08:00-07:00", "NoiseLevelDB": 83, "DurationSecond": 27.1},
  {"EntryTime": "2017-06-10T05:09:00-07:00", "NoiseLevelDB": 91, "DurationSecond": 17.1},
  {"EntryTime": "2017-06-10T05:09:00-07:00", "NoiseLevelDB": 115, "DurationSecond": 7.9},
  {"EntryTime": "2017-06-10T05:09:00-07:00", "NoiseLevelDB": 80, "DurationSecond": 28.3},
  {"EntryTime": "2017-06-10T05:10:00-07:00", "NoiseLevelDB": 55, "DurationSecond": 18.2},
  {"EntryTime": "2017-06-10T05:10:00-07:00", "NoiseLevelDB": 93, "DurationSecond": 25.8},
  {"EntryTime": "2017-06-10T05:11:00-07:00", "NoiseLevelDB": 83, "DurationSecond": 11.4},
  {"EntryTime": "2017-06-10T05:12:00-07:00", "NoiseLevelDB": 89, "DurationSecond": 7.9},
  {"EntryTime": "2017-06-10T05:15:00-07:00", "NoiseLevelDB": 112, "DurationSecond": 3.7},
  {"EntryTime": "2017-06-10T05:15:00-07:00", "NoiseLevelDB": 93, "DurationSecond": 9.7},
  {"EntryTime": "2017-06-10T05:18:00-07:00", "NoiseLevelDB": 96, "DurationSecond": 3.7},
  {"EntryTime": "2017-06-10T05:20:00-07:00", "NoiseLevelDB": 108, "DurationSecond": 0.99},
  {"EntryTime": "2017-06-10T05:20:00-07:00", "NoiseLevelDB": 113, "DurationSecond": 25.1},
  {"EntryTime": "2017-06-10T05:22:00-07:00", "NoiseLevelDB": 110, "DurationSecond": 5.3}
]
```

## <a name="get-help"></a>Obtenir de l’aide

Pour obtenir de l’aide, consultez notre [page de questions Microsoft Q&A sur Azure Stream Analytics](/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Étapes suivantes

* [Présentation d’Azure Stream Analytics](stream-analytics-introduction.md)
* [Prise en main d'Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Mise à l’échelle des travaux Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Références sur le langage des requêtes Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Références sur l’API REST de gestion d’Azure Stream Analytics](/rest/api/streamanalytics/)