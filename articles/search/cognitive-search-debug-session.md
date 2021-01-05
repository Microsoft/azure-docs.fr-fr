---
title: Concepts des sessions de débogage (préversion)
titleSuffix: Azure Cognitive Search
description: Les sessions de débogage, accessibles via le portail Azure, fournissent un environnement de type IDE au sein duquel vous pouvez identifier et corriger les erreurs, valider les modifications et les transférer vers un ensemble de compétences dans le pipeline d’enrichissement par IA. Les sessions de débogage sont disponibles en préversion.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/05/2020
ms.openlocfilehash: f666cc2f9d11b1e05d11ba266897e6fca3cd287e
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92057618"
---
# <a name="debug-sessions-in-azure-cognitive-search"></a>Sessions de débogage dans Recherche cognitive Azure

Les sessions de débogage consistent en un éditeur visuel qui fonctionne avec un ensemble de compétences existant dans le portail Azure. Au sein d’une session de débogage, vous pouvez identifier et résoudre les erreurs, valider les modifications et les transférer vers un ensemble de compétences de production dans le pipeline d’enrichissement par IA.

> [!Important]
> Les sessions de débogage constituent une fonctionnalité d’évaluation qui est fournie sans contrat de niveau de service et qui n’est pas recommandée pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>

## <a name="using-debug-sessions"></a>Utilisation des sessions de débogage

Lorsque vous démarrez une session, le service crée une copie de l’ensemble de compétences et de l’indexeur, puis il indexe l’endroit où un document seul est utilisé pour tester l’ensemble de compétences. Les modifications apportées au sein de la session y sont enregistrées. Les modifications apportées dans la session de débogage n’affectent pas l’ensemble de compétences de production, sauf si elles sont validées. La validation des modifications remplacera l’ensemble de compétences de production.

Pendant une session de débogage, vous pouvez modifier un ensemble de compétences, inspecter, et valider chaque nœud de l’arborescence d’enrichissement dans le contexte d’un document spécifique. Une fois les problèmes de pipeline d’enrichissement résolus, les modifications peuvent être enregistrées dans la session et validées dans l’ensemble de compétences en production. 

Si le pipeline d’enrichissement ne contient pas d’erreur, une session de débogage peut être utilisée pour enrichir un document de façon incrémentielle, tester et valider chaque modification avant la validation finale.

## <a name="creating-a-debug-session"></a>Création d’une session de débogage

Pour démarrer une session de débogage, vous devez disposer d’un pipeline d’enrichissement par IA existant, incluant : une source de données, un ensemble de compétences, un indexeur et un index. Pour configurer une session de débogage, vous devez nommer la session et fournir un compte de stockage à usage général qui sera utilisé pour mettre en cache les exécutions de compétences lors de l’exécution de l’indexeur. Vous devrez également sélectionner l’indexeur qui sera exécuté. L’indexeur contient des références stockées dans la source de données, dans l’ensemble de compétences et dans l’index. La session de débogage utilise par défaut le premier document de la source de données. Sinon, vous pouvez spécifier le document à parcourir dans la source de données.

> :::image type="content" source="media/cognitive-search-debug/debug-session-new.png" alt-text="Création d’une session de débogage":::

## <a name="debug-session-features"></a>Fonctionnalités de la session de débogage

La session de débogage commence par exécuter l’ensemble de compétences sur le document sélectionné. La session de débogage enregistre les métadonnées supplémentaires associées à chaque opération au sein de l’ensemble de compétences. Les métadonnées créées par les exécutions de compétences du pipeline indiquent l’ensemble de fonctionnalités qui suit. Elles sont ensuite utilisées pour aider à identifier et résoudre les problèmes liés à l’ensemble de compétence.

## <a name="ai-enrichments"></a>Enrichissements par IA

À mesure que les compétences s’exécutent, un arbre d’enrichissement, représentant le document, se développe. L’utilisation d’une arborescence pour visualiser les sorties de compétences ou d’enrichissements offre une vue d’ensemble de tous les enrichissements effectués. Vous pouvez consulter l’ensemble du document et inspecter chaque nœud de l’arborescence d’enrichissement. Cette perspective facilite la modélisation d’objets. Ce format fournit également des signaux visuels pour le type, le chemin d’accès et le contenu de chaque nœud dans l’arborescence.

## <a name="skill-graph"></a>Graphique des compétences

L’affichage du **graphe des compétences** fournit une représentation visuelle hiérarchique de l’ensemble de compétences. Le graphique représente l’ordre dans lequel les compétences sont exécutées, de haut en bas. Les compétences qui dépendent de la sortie d’autres compétences sont indiquées plus bas dans le graphique. Les compétences sur le même niveau hiérarchique peuvent être exécutées en parallèle. 

Sélectionner une compétence dans le graphique mettra en surbrillance les compétences qui y sont connectées, les nœuds qui créent ses entrées et ceux qui acceptent ses sorties. Le type, les erreurs ou avertissements ainsi que le nombre d’exécutions de chaque nœud de compétence apparaissent. Le **graphique des compétence** vous permet de sélectionner les compétences à déboguer ou améliorer. Lorsque vous sélectionnez une compétence, les informations la concernant sont affichées dans le volet d’information de la compétence à droite du graphique.

> :::image type="content" source="media/cognitive-search-debug/skills-graph.png" alt-text="Création d’une session de débogage":::

## <a name="skill-details"></a>Détails de la compétence

Le volet d’information sur la compétence affiche un ensemble de zones pour l’utilisation d’une compétence spécifique, lorsque celle-ci est en surbrillance dans le **graphique des compétence**. Vous pouvez examiner et modifier les informations des paramètres de la compétence. La définition JSON de la compétence est fournie. Les informations de l’exécution de la compétence, ainsi que les erreurs et avertissements, sont également affichés. L’onglet **Paramètres des compétences** et **l’éditeur de compétences JSON** permettent de modifier directement la compétence. [`</>`](#expression-evaluator) ouvre une fenêtre permettant d’afficher et de modifier les expressions des entrées et des sorties de compétences.

Les contrôles d’entrée imbriqués dans la fenêtre « paramètres des compétences » peuvent être utilisés pour créer des formes complexes pour les projections, des mappages de champs de sortie pour un champ complexe ou une entrée de compétence. Lorsqu’elles sont utilisées avec **l’évaluateur d’expression**, les entrées imbriquées constituent un test facile pour valider le générateur d’expressions.

## <a name="skill-execution-history"></a>Historique d’exécution des compétences

Une compétence peut s’exécuter plusieurs fois dans un ensemble de compétences sur un document seul. Par exemple, la compétence de reconnaissance optique de caractères (OCR) s’exécutera une fois pour chaque image extraite de chacun des documents. Dans le volet d’information sur la compétence, l’onglet **Exécutions** affiche l’historique d’exécution de la compétence, ce qui permet un examen plus approfondi de chaque appel de la compétence. 

L’historique d’exécution permet de remonter jusqu’à la compétence qui a généré un enrichissement spécifique. Cliquer sur une entrée de compétence pour accéder à la compétence qui a généré cette entrée. Cela permet d’identifier la cause racine d’un problème qui peut se manifester dans une compétence en aval. 

Lorsqu’un problème potentiel est identifié, l’historique d’exécution affiche des liens vers les compétences qui ont généré les entrées spécifiques, et offre une fonctionnalité de traçage de piles similaires. Cliquer sur la compétence associée à l’entrée, accéder à la compétence pour corriger les bogues ou continuer à remonter vers le problème spécifique.

Lors de la création d’une compétence personnalisée ou du débogage d’une erreur avec une compétence personnalisée, il est possible de générer une requête d’appel de compétence dans l’historique d’exécution.

## <a name="enriched-data-structure"></a>Structure de données enrichie

Le volet **Structure de données enrichie** montre les enrichissements du document via l’ensemble de compétences, en détaillant le contexte et la compétence d’origine pour chaque enrichissement. **L’évaluateur d’expression** peut également servir à afficher le contenu de chaque enrichissement.

> :::image type="content" source="media/cognitive-search-debug/enriched-data-structure-display.png" alt-text="Création d’une session de débogage":::

## <a name="expression-evaluator"></a>Évaluateur d’expression

**L’évaluateur d’expression** offre un aperçu de la valeur de tout chemin d’accès. Il permet de modifier le chemin d’accès et de tester les résultats avant de mettre à jour les entrées ou le contexte d’une compétence ou d’une projection.

## <a name="errorswarnings"></a>Erreurs/avertissements

Cette fenêtre affiche toutes les erreurs et les avertissements produits par l’ensemble de compétences lorsqu’il est exécuté sur le document dans la session de débogage.

## <a name="limitations"></a>Limites

Les sessions de débogage fonctionnent avec toutes les sources de données généralement disponibles et la plupart des sources de données en préversion. Les API MongoDB et Cassandra de Cosmos DB (toutes deux en préversion) ne sont actuellement pas prises en charge.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez compris les composantes des sessions de débogage, essayez le tutoriel pour mettre la théorie en pratique.

> [!div class="nextstepaction"]
> [Tutoriel : explorer les fonctionnalités des sessions de débogage](./cognitive-search-tutorial-debug-sessions.md)