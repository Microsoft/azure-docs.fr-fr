---
title: Implémenter la confidentialité différentielle avec le package WhiteNoise (version préliminaire)
titleSuffix: Azure Machine Learning
description: Découvrez ce qu’est la confidentialité différentielle et comment le package WhiteNoise peut vous aider à implémenter des systèmes privés différentiels qui préservent la confidentialité des données.
author: luisquintanilla
ms.author: luquinta
ms.date: 07/09/2020
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.openlocfilehash: 9728bf2c86c0629b09e2325650ce288cf9b3cc7e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86199795"
---
# <a name="preserve-data-privacy-by-using-differential-privacy-and-the-whitenoise-package-preview"></a>Préserver la confidentialité des données en utilisant la confidentialité différentielle et le package WhiteNoise (version préliminaire)

Découvrez ce qu’est la confidentialité différentielle et comment le package WhiteNoise peut vous aider à implémenter des systèmes offrant une confidentialité différentielle.

Les préoccupations liées à la confidentialité et à la sécurité augmentent à mesure qu’augmente la quantité de données qu’une organisation collecte et utilise à des fins d’analyse. Les analyses requièrent des données. En règle générale, plus il y a de données utilisées pour effectuer l’apprentissage des modèles, plus ceux-ci sont précis. Lorsque des informations personnelles sont utilisées pour ces analyses, il est particulièrement important que les données restent confidentielles tout au long de leur utilisation.

> [!NOTE]
> Veuillez noter que nous allons renommer le kit de ressources et que nous présenterons le nouveau nom dans les semaines à venir. 

## <a name="how-differential-privacy-works"></a>Fonctionnement de la confidentialité différentielle

La confidentialité différentielle est un ensemble de systèmes et pratiques qui permettent de préserver la sécurité et la confidentialité des données des personnes.

> [!div class="mx-imgBorder"]
> ![Processus de confidentialité différentielle](./media/concept-differential-privacy/differential-privacy-process.jpg)

Dans les scénarios traditionnels, les données brutes sont stockées dans des fichiers et des bases de données. Lorsque des utilisateurs analysent des données, ils utilisent généralement des données brutes. C’est problématique, car cela peut entraîner une violation de données à caractère personnel. La confidentialité différentielle tente de résoudre ce problème en ajoutant du « bruit » ou une composante aléatoire aux données afin que les utilisateurs ne puissent pas identifier des points de données individuels. Un tel système offre au moins une possibilité de « démenti raisonnable ».

Dans les systèmes assortis d’une confidentialité différentielle, les données sont partagées via des demandes appelées **requêtes**. Quand un utilisateur soumet une requête de données, des opérations appelées **mécanismes de confidentialité** ajoutent du bruit aux données demandées. Ces mécanismes de confidentialité renvoient des *données approximatives* au lieu de données brutes. Le résultat de cette préservation de la confidentialité apparaît dans un **rapport**. Les rapports se composent de deux parties : les données réelles calculées et une description de la façon dont elles ont été créées.

## <a name="differential-privacy-metrics"></a>Métriques de confidentialité différentielle

La confidentialité différentielle vise à éviter qu’un utilisateur puisse produire un nombre indéfini de rapports pour révéler des données sensibles. Une valeur appelée **epsilon** mesure le niveau de bruit ou de confidentialité d’un rapport. La valeur epsilon est inversement proportionnelle au bruit ou à la confidentialité. Plus la valeur epsilon est basse, plus les données sont bruyantes (et confidentielles).

Les valeurs epsilon ne sont pas négatives. Les valeurs inférieures à 1 offrent une possibilité de « démenti raisonnable ». Toute valeur supérieure à 1 est associée à un risque plus élevé d’exposition des données réelles. Lorsque vous implémentez des systèmes assortis d’une confidentialité différentielle, vous souhaitez produire des rapports avec des valeurs epsilon comprises entre 0 et 1.

Une autre valeur directement corrélée à la valeur epsilon est le valeur **delta**. La valeur delta indique la probabilité qu’un rapport ne soit pas entièrement confidentiel. La valeur delta est directement proportionnelle à la valeur epsilon. Ces valeurs étant corrélées, la valeur epsilon est utilisée plus fréquemment.

## <a name="privacy-budget"></a>Budget de confidentialité

Pour garantir la confidentialité de systèmes où plusieurs requêtes sont autorisées, la confidentialité différentielle définit une limite de débit. Cette limite est appelée **budget de confidentialité**. Les budgets de confidentialité sont associés à un volume epsilon, généralement compris entre 1 et 3, pour limiter le risque de réidentification. Au fur et à mesure de la génération de rapports, les budgets de confidentialité assurent le suivi de la valeur epsilon des rapports individuels, ainsi que de l’agrégat de tous les rapports. Une fois le budget de confidentialité dépensé ou épuisé, les utilisateurs ne peuvent plus accéder aux données.  

## <a name="reliability-of-data"></a>Fiabilité des données

Même si la préservation de la confidentialité doit être l’objectif, il existe un compromis sur le plan de l’exploitabilité et de la fiabilité des données. En analytique données, l’exactitude peut être considérée comme une mesure de l’incertitude introduite par des erreurs d’échantillonnage. Cette incertitude tend s’inscrire dans certaines limites. L’**exactitude** du point de vue de la confidentialité différentielle, mesure plutôt la fiabilité des données, qui est affectée par l’incertitude qu’introduisent les mécanismes de confidentialité. En deux mots, un niveau élevé de bruit ou de confidentialité se traduit par des données dont la valeur epsilon, l’exactitude et la fiabilité sont moindres. Bien que les données soient plus confidentielles, en raison de leur manque de fiabilité, leur utilisation est moins probable.

## <a name="implementing-differentially-private-systems"></a>Implémentation de systèmes de confidentialité différentielle

L’implémentation de systèmes de confidentialité différentielle est difficile. WhiteNoise est un projet open source contenant différents composants pour la création de systèmes globaux assortis d’une confidentialité différentielle. WhiteNoise est constitué des composants de niveau supérieur suivants :

- Core
- Système

### <a name="core"></a>Core

La bibliothèque principale comprend les mécanismes de confidentialité suivants pour l’implémentation d’un système assorti d’une confidentialité différentielle :

|Composant  |Description  |
|---------|---------|
|Analyse     | Description graphique des calculs arbitraires. |
|Validateur     | Bibliothèque Rust contenant un ensemble d’outils permettant de vérifier et dériver les conditions nécessaires pour qu’une analyse bénéficie d’une confidentialité différentielle.          |
|Runtime     | Moyen d’exécuter l’analyse. Le runtime de référence est écrit en Rust, mais vous pouvez écrire des runtimes à l’aide de n’importe quelle infrastructure de calcul telle que SQL ou Spark, en fonction de vos besoins en matière de données.        |
|Liaisons     | Liaisons de langage et bibliothèques d’assistance pour générer des analyses. Actuellement, WhiteNoise fournit des liaisons Python. |

### <a name="system"></a>Système

La bibliothèque système fournit les outils et services suivants pour travailler avec des données tabulaires et relationnelles :

|Composant  |Description  |
|---------|---------|
|Accès aux données     | Bibliothèque qui intercepte et traite des requêtes SQL, et génère des rapports. Cette bibliothèque est implémentée en Python et prend en charge les sources de données ODBC et DBAPI suivantes :<ul><li>PostgreSQL</li><li>SQL Server</li><li>Spark</li><li>Preston</li><li>Pandas</li></ul>|
|Service     | Service d’exécution fournissant un point de terminaison REST pour traiter des demandes ou requêtes sur des sources de données partagées. Le service est conçu pour permettre la composition de modules de confidentialité différentielle qui opèrent sur des demandes contenant différentes valeurs delta et epsilon, également appelées demandes hétérogènes. Cette implémentation de référence explique l’impact supplémentaire des requêtes sur les données corrélées. |
|Évaluateur     | Évaluateur stochastique qui vérifie les violations de confidentialité, l’exactitude et le biais. L’évaluateur prend en charge les tests suivants : <ul><li>Test de confidentialité : détermine si un rapport respecte les conditions de la confidentialité différentielle.</li><li>Test d’exactitude : mesure si la fiabilité des rapports s’inscrit dans les limites supérieure et inférieure, compte tenu d’un niveau de confiance de 95 %.</li><li>Test d’utilitaire : détermine si les limites de confiance d’un rapport sont suffisamment proches des données tout en optimisant la confidentialité.</li><li>Test de biais : mesure la distribution des rapports pour des requêtes répétées afin de s’assurer qu’elles ne sont pas déséquilibrées.</li></ul> |

## <a name="next-steps"></a>Étapes suivantes

[Préserver la confidentialité des données](how-to-differential-privacy.md) dans Azure Machine Learning.

Pour en savoir plus sur les composants de WhiteNoise, consultez les référentiels GitHub pour [Package principal WhiteNoise](https://github.com/opendifferentialprivacy/whitenoise-core), [Package système WhiteNoise](https://github.com/opendifferentialprivacy/whitenoise-system) et [Exemples WhiteNoise](https://github.com/opendifferentialprivacy/whitenoise-samples) (disponibles en anglais uniquement).