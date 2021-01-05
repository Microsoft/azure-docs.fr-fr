---
title: Guide de référence des vignettes du Concepteur de vues dans Azure Monitor | Microsoft Docs
description: Grâce au Concepteur de vues d’Azure Monitor, vous pouvez créer des vues personnalisées affichées dans le portail Azure qui contiennent différentes visualisations sur les données dans l’espace de travail Log Analytics. Cet article est un guide de référence pour les paramètres des vignettes disponibles dans vos vues personnalisées.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 01/17/2018
ms.openlocfilehash: b73fa9702a95897afbca53e2de70dcabc04a4665
ms.sourcegitcommit: 419c8c8061c0ff6dc12c66ad6eda1b266d2f40bd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2020
ms.locfileid: "92168504"
---
# <a name="reference-guide-to-view-designer-tiles-in-azure-monitor"></a>Guide de référence des vignettes du Concepteur de vues dans Azure Monitor
Grâce au Concepteur de vues d’Azure Monitor, vous pouvez créer plusieurs vues personnalisées dans le portail Azure qui peuvent vous aider à visualiser les données dans votre espace de travail Log Analytics. Cet article est un guide de référence pour les paramètres des vignettes disponibles dans vos vues personnalisées.

Pour plus d’informations sur le Concepteur de vues, consultez :

* [Concepteur de vues](view-designer.md) : fournit une présentation du Concepteur de vues et des procédures de création et de modification des vues personnalisées.
* [Référence des composants de visualisation](view-designer-parts.md) : fournit un guide de référence pour les paramètres des composants de visualisation disponibles dans vos vues personnalisées.


Les vignettes du Concepteur de vues disponibles sont décrites dans le tableau suivant :  

| Vignette | Description |
|:--- |:--- |
| [Nombre](#number-tile) |Nombre d’enregistrements retournés par une requête. |
| [Deux nombres](#two-numbers-tile) |Nombre d’enregistrements retournés par deux requêtes différentes. |
| [Anneau](#donut-tile) | Graphique basé sur une requête, avec une valeur de synthèse au centre. |
| Graphique en courbes et légende | Graphique en courbes basé sur une requête, et légende avec une valeur de synthèse. |
| [Graphique en courbes](#line-chart-tile) |Graphique en courbes basé sur une requête. |
| [Deux chronologies](#two-timelines-tile) | Histogramme avec deux séries basées chacune sur une requête distincte. |

Les sections suivantes décrivent les types de vignettes et leurs propriétés en détail.

> [!NOTE]
> Les vignettes dans les vues sont basées sur des [requêtes de journal](../log-query/log-query-overview.md) dans votre espace de travail Log Analytics. Elles ne prennent pas en charge les [requêtes inter-ressources](../log-query/cross-workspace-query.md) pour récupérer des données d’Application Insights.

## <a name="number-tile"></a>Vignette Nombre
La vignette **Nombre** affiche à la fois le nombre d’enregistrements retournés par une requête de journal et une étiquette.

![Vignette Nombre](media/view-designer-tiles/tile-number.png)

| Paramètre | Description |
|:--- |:--- |
| Name |Texte affiché en haut de la vignette. |
| Description |Texte affiché sous le nom de la vignette. |
| **Vignette** | |
| Légende |Texte affiché sous la valeur. |
| Requête |Requête exécutée. Le nombre d’enregistrements retournés par la requête est affiché. |
| **Avancée** |**&gt; Vérification du flux de données** |
| activé |Sélectionnez ce lien si la vérification du flux de données doit être activée pour la vignette. Cette approche fournit un autre message si les données ne sont pas disponibles. Normalement, vous adoptez cette approche pour fournir un message entre l’installation de la vue et le moment où les données deviennent disponibles. |
| Requête |Requête exécutée pour déterminer si les données sont disponibles pour la vue. Si la requête ne retourne aucun résultat, un message s’affiche à la place de la valeur de la requête principale. |
| Message |Message affiché si la requête de vérification du flux de données ne retourne aucune donnée. Si vous ne fournissez aucun message, un message d’état *Exécution de l’évaluation* s’affiche. |


## <a name="two-numbers-tile"></a>Vignette Deux nombres
Cette vignette affiche le nombre d’enregistrements retournés par deux requêtes de journal distinctes, et une étiquette pour chacune d’elles.

![Vignette Deux nombres](media/view-designer-tiles/tile-two-numbers.png)

| Paramètre | Description |
|:--- |:--- |
| Name |Texte affiché en haut de la vignette. |
| Description |Texte affiché sous le nom de la vignette. |
| **Première vignette** | |
| Légende |Texte affiché sous la valeur. |
| Requête |Requête exécutée. Le nombre d’enregistrements retournés par la requête est affiché. |
| **Deuxième vignette** | |
| Légende |Texte affiché sous la valeur. |
| Requête |Requête exécutée. Le nombre d’enregistrements retournés par la requête est affiché. |
| **Avancée** |**&gt; Vérification du flux de données** |
| activé |Sélectionnez ce lien si la vérification du flux de données doit être activée pour la vignette. Cette approche fournit un autre message si les données ne sont pas disponibles. Normalement, vous adoptez cette approche pour fournir un message entre l’installation de la vue et le moment où les données deviennent disponibles. |
| Requête |Requête exécutée pour déterminer si les données sont disponibles pour la vue. Si la requête ne retourne aucun résultat, un message s’affiche à la place de la valeur de la requête principale. |
| Message |Message affiché si la requête de vérification du flux de données ne retourne aucune donnée. Si vous ne fournissez aucun message, un message d’état *Exécution de l’évaluation* s’affiche. |


## <a name="donut-tile"></a>Vignette Anneau
La vignette **Anneau** affiche un nombre unique qui résume une colonne de valeur dans une requête de journal. L’anneau affiche sous forme graphique les résultats des trois premiers enregistrements.

![Vignette Anneau](media/view-designer-tiles/tile-donut.png)

| Paramètre | Description |
|:--- |:--- |
| Name |Texte affiché en haut de la vignette. |
| Description |Texte affiché sous le nom de la vignette. |
| **Anneau** | |
| Requête |Requête exécutée pour l’anneau. La première propriété est une valeur de texte et la seconde une valeur numérique. Cette requête utilise habituellement le mot clé *measure* pour synthétiser les résultats. |
| **Anneau** |**&gt; Centrer** |
| Texte |Texte affiché sous la valeur à l’intérieur de l’anneau. |
| Opération |Opération effectuée sur la valeur de propriété afin de la résumer en une valeur unique.<ul><li>Sum : additionner les valeurs de tous les enregistrements avec la valeur de propriété.</li><li>Percentage : pourcentage des valeurs additionnées des enregistrements avec la valeur de propriété, comparé aux valeurs additionnées de tous les enregistrements.</li></ul> |
| Valeurs de résultat utilisées dans l’opération relative au centre |Vous pouvez sélectionner le signe plus (+) pour ajouter une ou plusieurs valeurs. Les résultats de la requête sont alors limités aux enregistrements dont vous avez spécifié les valeurs de propriété. Si aucune valeur n’est ajoutée, tous les enregistrements sont inclus dans la requête. |
| **Anneau** |**&gt; Options supplémentaires** |
| Couleurs |Couleur affiché pour chacune des trois premières propriétés. Pour spécifier d’autres couleurs pour des valeurs de propriété spécifiques, utilisez l’option *Mappage avancé des couleurs*. |
| Mappage avancé des couleurs |Affiche une couleur qui représente des valeurs de propriété spécifiques. Si la valeur spécifiée figure parmi les trois premières, l’autre couleur s’affiche au lieu de la couleur standard. Si la propriété ne figure pas parmi les trois premières, la couleur ne s’affiche pas. |
| **Avancée** |**&gt; Vérification du flux de données** |
| activé |Sélectionnez ce lien si la vérification du flux de données doit être activée pour la vignette. Cette approche fournit un autre message si les données ne sont pas disponibles. Normalement, vous adoptez cette approche pour fournir un message entre l’installation de la vue et le moment où les données deviennent disponibles. |
| Requête |Requête exécutée pour déterminer si les données sont disponibles pour la vue. Si la requête ne retourne aucun résultat, un message s’affiche à la place de la valeur de la requête principale. |
| Message |Message affiché si la requête de vérification du flux de données ne retourne aucune donnée. Si vous ne fournissez aucun message, un message d’état *Exécution de l’évaluation* s’affiche. |


## <a name="line-chart-tile"></a>Vignette Graphique en courbes
Cette vignette est un graphique en courbes qui affiche plusieurs séries à partir d’une requête de journal dans le temps. 

![Capture d’écran d’une vignette de graphique en courbes dans le concepteur de vues Azure Monitor.](media/view-designer-tiles/tile-line-chart.png)

| Paramètre | Description |
|:--- |:--- |
| Name |Texte affiché en haut de la vignette. |
| Description |Texte affiché sous le nom de la vignette. |
| **Graphique en courbes** | |
| Requête |Requête exécutée pour le graphique en courbes. La première propriété est une valeur de texte et la seconde une valeur numérique. Cette requête utilise habituellement le mot clé *measure* pour synthétiser les résultats. Si la requête utilise le mot clé *interval*, l’axe des abscisses (X) utilise cet intervalle de temps. Si la requête n’utilise pas le mot clé *interval*, l’axe des abscisses utilise des intervalles d’une heure. |
| **Graphique en courbes** |**> Axe Y** |
| Utiliser l’échelle logarithmique |Sélectionnez ce lien pour utiliser une échelle logarithmique pour l’axe des ordonnées (Y). |
| Units |Spécifiez les unités à utiliser pour exprimer les valeurs retournées par la requête. Ces informations sont utilisées pour afficher sur le graphique des étiquettes indiquant les types de valeurs et, le cas échéant, pour convertir des valeurs. Le **Type d’unité** spécifie la catégorie de l’unité et définit les valeurs de **Type d’unité en cours** disponibles. Si vous sélectionnez une valeur dans **Convertir en**, les valeurs numériques sont converties du type **Unité actuelle** dans le type **Convertir en**. |
| Étiquette personnalisée |Texte affiché pour l’axe Y en regard de l’étiquette du type d’*Unité*. Si aucune étiquette n’est spécifiée, seul le type d’*Unité* est affiché. |
| **Avancée** |**&gt; Vérification du flux de données** |
| activé |Sélectionnez ce lien si la vérification du flux de données doit être activée pour la vignette. Cette approche fournit un autre message si les données ne sont pas disponibles. Normalement, vous adoptez cette approche pour fournir un message entre l’installation de la vue et le moment où les données deviennent disponibles. |
| Requête |Requête exécutée pour déterminer si les données sont disponibles pour la vue. Si la requête ne retourne aucun résultat, un message s’affiche à la place de la valeur de la requête principale. |
| Message |Message affiché si la requête de vérification du flux de données ne retourne aucune donnée. Si vous ne fournissez aucun message, un message d’état *Exécution de l’évaluation* s’affiche. |


## <a name="line-chart-and-callout-tile"></a>Vignette Graphique en courbes et légende
Cette vignette contient à la fois un graphique en courbes avec plusieurs séries à partir d’une requête de journal dans le temps, et une légende avec une valeur de synthèse. 

![Capture d’écran d’un graphique en courbes et d’une vignette de légende dans le concepteur de vues Azure Monitor. La légende décrit le graphique en courbes en présentant une valeur de synthèse.](media/view-designer-tiles/tile-line-chart-callout.png)

| Paramètre | Description |
|:--- |:--- |
| Name |Texte affiché en haut de la vignette. |
| Description |Texte affiché sous le nom de la vignette. |
| **Graphique en courbes** | |
| Requête |Requête exécutée pour le graphique en courbes. La première propriété est une valeur de texte et la seconde une valeur numérique. Cette requête utilise habituellement le mot clé *measure* pour synthétiser les résultats. Si la requête utilise le mot clé *interval*, l’axe des abscisses (X) utilise cet intervalle de temps. Si la requête n’utilise pas le mot clé *interval*, l’axe des abscisses utilise des intervalles d’une heure. |
| **Graphique en courbes** |**&gt; Légende** |
| Titre de la légende | Texte affiché au-dessus de la valeur de la légende. |
| Nom de la série |Valeur de propriété de série à utiliser comme valeur de la légende. Si aucune série n’est fournie, tous les enregistrements de la requête sont utilisés. |
| Opération |Opération effectuée sur la valeur de propriété afin de la résumer en une valeur unique pour la légende.<ul><li>Average : moyenne des valeurs de tous les enregistrements.</li><li>Count : nombre d’enregistrements retournés par la requête.</li><li>Last Sample : valeur du dernier intervalle inclus dans le graphique.</li><li>Max : valeur maximale des intervalles inclus dans le graphique.</li><li>Min : valeur minimale des intervalles inclus dans le graphique.</li><li>Sum : somme des valeurs de tous les enregistrements.</li></ul> |
| **Graphique en courbes** |**> Axe Y** |
| Utiliser l’échelle logarithmique |Sélectionnez ce lien pour utiliser une échelle logarithmique pour l’axe des ordonnées (Y). |
| Units |Spécifiez les unités à utiliser pour exprimer les valeurs retournées par la requête. Ces informations sont utilisées pour afficher sur le graphique des étiquettes indiquant les types de valeurs et, le cas échéant, pour convertir les valeurs. Le type d’*Unité* spécifie la catégorie de l’unité, et définit les valeurs de type *Unité actuelle* disponibles. Si vous sélectionnez une valeur pour l’option *Convertir en*, les valeurs numériques sont converties du type *Unité actuelle* au type *Convertir en*. |
| Étiquette personnalisée |Texte affiché pour l’axe Y en regard de l’étiquette du type d’*Unité*. Si aucune étiquette n’est spécifiée, seul le type d’*Unité* est affiché. |
| **Avancée** |**&gt; Vérification du flux de données** |
| activé |Sélectionnez ce lien si la vérification du flux de données doit être activée pour la vignette. Cette approche fournit un autre message si les données ne sont pas disponibles. Normalement, vous adoptez cette approche pour fournir un message entre l’installation de la vue et le moment où les données deviennent disponibles. |
| Requête |Requête exécutée pour déterminer si les données sont disponibles pour la vue. Si la requête ne retourne aucun résultat, un message s’affiche à la place de la valeur de la requête principale. |
| Message |Message affiché si la requête de vérification du flux de données ne retourne aucune donnée. Si vous ne fournissez aucun message, un message d’état *Exécution de l’évaluation* s’affiche. |


## <a name="two-timelines-tile"></a>Vignette Deux chronologies
La vignette **Deux chronologies** affiche les résultats de deux requêtes de journal dans le temps sous la forme de graphiques à barres. Une légende est affichée pour chaque série. 

![Vignette Deux chronologies](media/view-designer-tiles/tile-two-timelines.png)

| Paramètre | Description |
|:--- |:--- |
| Name |Texte affiché en haut de la vignette. |
| Description |Texte affiché sous le nom de la vignette. |
| Premier graphique | |
| Légende |Texte affiché sous la légende de la première série. |
| Couleur |Couleur utilisée pour les colonnes de la première série. |
| Requête de graphique |Requête exécutée pour la première série. Le nombre d’enregistrements sur chaque intervalle de temps est représenté par les colonnes de graphique. |
| Opération |Opération effectuée sur la valeur de propriété afin de la résumer en une valeur unique pour la légende.<ul><li>Average : moyenne des valeurs de tous les enregistrements.</li><li>Count : nombre d’enregistrements retournés par la requête.</li><li>Last Sample : valeur du dernier intervalle inclus dans le graphique.</li><li>Max : valeur maximale des intervalles inclus dans le graphique.</li></ul> |
| **Deuxième graphique** | |
| Légende |Texte affiché sous la légende de la deuxième série. |
| Couleur |Couleur utilisée pour les colonnes de la deuxième série. |
| Requête de graphique |Requête exécutée pour la deuxième série. Le nombre d’enregistrements sur chaque intervalle de temps est représenté par les colonnes de graphique. |
| Opération |Opération effectuée sur la valeur de propriété afin de la résumer en une valeur unique pour la légende.<ul><li>Average : moyenne des valeurs de tous les enregistrements.</li><li>Count : nombre d’enregistrements retournés par la requête.</li><li>Last Sample : valeur du dernier intervalle inclus dans le graphique.</li><li>Max : valeur maximale des intervalles inclus dans le graphique. |
| **Avancée** |**&gt; Vérification du flux de données** |
| activé |Sélectionnez ce lien si la vérification du flux de données doit être activée pour la vignette. Cette approche fournit un autre message si les données ne sont pas disponibles. Normalement, vous adoptez cette approche pour fournir un message entre l’installation de la vue et le moment où les données deviennent disponibles. |
| Requête |Requête exécutée pour déterminer si les données sont disponibles pour la vue. Si la requête ne retourne aucun résultat, un message s’affiche à la place de la valeur de la requête principale. |
| Message |Message affiché si la requête de vérification du flux de données ne retourne aucune donnée. Si vous ne fournissez aucun message, un message d’état *Exécution de l’évaluation* s’affiche. |


## <a name="next-steps"></a>Étapes suivantes
* Découvrez-en plus sur les [requêtes de journal](../log-query/log-query-overview.md) pour prendre en charge les requêtes dans des vignettes.
* Ajouter des [composants de visualisation](view-designer-parts.md) à votre vue personnalisée.
