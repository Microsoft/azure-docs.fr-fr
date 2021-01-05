---
title: Guide de référence des composants du Concepteur de vues dans Azure Monitor | Microsoft Docs
description: Grâce au Concepteur de vues d’Azure Monitor, vous pouvez créer des vues personnalisées affichées dans le portail Azure qui contiennent différentes visualisations sur les données dans l’espace de travail Log Analytics. Cet article est un guide de référence pour les paramètres des composants de visualisation disponibles dans vos vues personnalisées.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/12/2018
ms.openlocfilehash: 7b670cafa4d643e37fae068a4c0033fc97a96ccd
ms.sourcegitcommit: 419c8c8061c0ff6dc12c66ad6eda1b266d2f40bd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2020
ms.locfileid: "92166588"
---
# <a name="reference-guide-to-view-designer-visualization-parts-in-azure-monitor"></a>Guide de référence des composants de visualisation du Concepteur de vues dans Azure Monitor
Grâce au Concepteur de vues d’Azure Monitor, vous pouvez créer plusieurs vues personnalisées dans le portail Azure qui peuvent vous aider à visualiser les données dans votre espace de travail Log Analytics. Cet article est un guide de référence pour les paramètres des composants de visualisation disponibles dans vos vues personnalisées.

Pour plus d’informations sur le Concepteur de vues, consultez :

* [Concepteur de vues](view-designer.md) : fournit une présentation du Concepteur de vues et des procédures de création et de modification des vues personnalisées.
* [Référence de la vignette](view-designer-tiles.md) : fournit une référence pour les paramètres de chaque vignette disponible dans vos vues personnalisées.


Les types de vignettes du Concepteur de vues disponibles sont décrites dans le tableau suivant :

| Type de vue | Description |
|:--- |:--- |
| [Liste de requêtes](#list-of-queries-part) |Affiche une liste de requêtes de journal. Vous pouvez sélectionner chaque requête pour afficher ses résultats. |
| [Nombre et liste](#number-and-list-part) |L’en-tête affiche une valeur qui indique le nombre d’enregistrements obtenus à partir d’une requête de journal. La liste affiche les dix premiers résultats d’une requête, avec un graphique qui indique la valeur relative d’une colonne numérique ou ses changements avec le temps. |
| [Deux nombres et liste](#two-numbers-and-list-part) |L’en-tête affiche deux valeurs qui indiquent les nombres d’enregistrements obtenus à partir de requêtes de journal distinctes. La liste affiche les dix premiers résultats d’une requête, avec un graphique qui indique la valeur relative d’une colonne numérique ou ses changements avec le temps. |
| [Anneau et liste](#donut-and-list-part) |L’en-tête affiche un nombre unique qui résume une colonne de valeur dans une requête de journal. L’anneau affiche sous forme graphique les résultats des trois premiers enregistrements. |
| [Deux chronologies et liste](#two-timelines-and-list-part) |L’en-tête affiche les résultats de deux requêtes de journal dans le temps, sous forme d’histogrammes avec une légende affichant un nombre qui résume une colonne de valeur dans une requête de journal. La liste affiche les dix premiers résultats d’une requête, avec un graphique qui indique la valeur relative d’une colonne numérique ou ses changements avec le temps. |
| [Informations](#information-part) |L’en-tête affiche un texte statique et un lien facultatif. La liste affiche un ou plusieurs éléments avec un titre et un texte statiques. |
| [Graphique en courbes, légende et liste](#line-chart-callout-and-list-part) |L’en-tête affiche un graphique en courbes avec plusieurs séries à partir d’une requête de journal dans le temps, et une légende avec une valeur de synthèse. La liste affiche les dix premiers résultats d’une requête, avec un graphique qui indique la valeur relative d’une colonne numérique ou ses changements avec le temps. |
| [Graphique en courbes et liste](#line-chart-and-list-part) |L’en-tête affiche un graphique en courbes avec plusieurs séries à partir d’une requête de journal dans le temps. La liste affiche les dix premiers résultats d’une requête, avec un graphique qui indique la valeur relative d’une colonne numérique ou ses changements avec le temps. |
| [Partie de pile de graphiques de courbes](#stack-of-line-charts-part) |Affiche trois graphiques en courbes distincts avec plusieurs séries à partir d’une requête de journal dans le temps. |

Les sections suivantes décrivent les types de vignettes et leurs propriétés en détail.

> [!NOTE]
> Les composants des vues sont basés sur des [requêtes de journal](../log-query/log-query-overview.md) dans votre espace de travail Log Analytics. Elles ne prennent pas en charge les [requêtes inter-ressources](../log-query/cross-workspace-query.md) pour récupérer des données d’Application Insights.

## <a name="list-of-queries-part"></a>Liste de parties de requêtes
La liste des parties de requêtes affiche une liste de requêtes de journal. Vous pouvez sélectionner chaque requête pour afficher ses résultats. La vue inclut une requête par défaut, et vous pouvez sélectionner **+ Requête** pour ajouter des requêtes supplémentaires.

![Capture d’écran du composant de visualisation Liste des requêtes dans le concepteur de vues Azure Monitor.](media/view-designer-parts/view-list-queries.png)

| Paramètre | Description |
|:--- |:--- |
| **Généralités** | |
| Intitulé |Texte affiché en haut de la vue. |
| Nouveau groupe |Sélectionnez ce lien pour créer un groupe dans la vue, en commençant avec la vue actuelle. |
| Filtres présélectionnés |Liste de propriétés séparées par des virgules à inclure dans le volet de filtre de gauche quand vous sélectionnez une requête. |
| Mode d’affichage |Vue initiale affichée quand la requête est sélectionnée. Vous pouvez sélectionner toutes les vues disponibles une fois qu’elles ont ouvert la requête. |
| **Requêtes** | |
| Requête de recherche |Requête à exécuter. |
| Nom convivial | Nom descriptif affiché. |

## <a name="number-and-list-part"></a>Partie Nombre et liste
L’en-tête affiche une valeur qui indique le nombre d’enregistrements obtenus à partir d’une requête de journal. La liste affiche les dix premiers résultats d’une requête, avec un graphique qui indique la valeur relative d’une colonne numérique ou ses changements avec le temps.

![Capture d’écran des composants de visualisation Nombre et Liste dans le concepteur de vues Azure Monitor.](media/view-designer-parts/view-number-list.png)

| Paramètre | Description |
|:--- |:--- |
| **Généralités** | |
| Titre du groupe |Texte affiché en haut de la vue. |
| Nouveau groupe |Sélectionnez ce lien pour créer un groupe dans la vue, en commençant avec la vue actuelle. |
| Icône |Fichier image affiché à côté du résultat dans l’en-tête. |
| Icône Utiliser |Sélectionnez ce lien pour afficher l’icône. |
| **Titre** | |
| Légende |Texte affiché en haut de l’en-tête. |
| Requête |Requête à exécuter pour l’en-tête. Le nombre d’enregistrements retournés par la requête est affiché. |
| Navigation par clic | Action effectuée lorsque vous cliquez sur l’en-tête.  Pour plus d’informations, consultez [Paramètres courants](#click-through-navigation). |
| **Liste** | |
| Requête |Requête à exécuter pour obtenir la liste. Les deux premières propriétés des dix premiers enregistrements dans les résultats sont affichées. La première propriété est une valeur de texte et la seconde une valeur numérique. Les barres sont créées automatiquement en fonction de la valeur relative de la colonne numérique.<br><br>Utilisez la commande `Sort` dans la requête pour trier les enregistrements de la liste. Pour exécuter la requête et retourner tous les enregistrements, vous pouvez sélectionner **Afficher tout** . |
| Masquer le graphique |Sélectionnez ce lien pour désactiver le graphique à droite de la colonne numérique. |
| Activation des sparklines |Sélectionnez ce lien pour afficher un graphique Sparkline au lieu d’une barre horizontale. Pour plus d’informations, consultez [Paramètres courants](#sparklines). |
| Couleur |Couleur des barres ou graphiques Sparkline. |
| Séparateur de noms et de valeurs |Délimiteur de caractère unique à utiliser pour analyser la propriété de texte en plusieurs valeurs. Pour plus d’informations, consultez [Paramètres courants](#sparklines). |
| Navigation par clic | Action effectuée lorsque vous cliquez sur un élément dans la liste.  Pour plus d’informations, consultez [Paramètres courants](#click-through-navigation). |
| **Liste** |**> Titres des colonnes** |
| Name |Texte affiché en haut de la première colonne. |
| Valeur |Texte affiché en haut de la deuxième colonne. |
| **Liste** |**&gt; Seuils** |
| Activer les seuils |Sélectionnez ce lien pour activer les seuils. Pour plus d’informations, consultez [Paramètres courants](#thresholds). |

## <a name="two-numbers-and-list-part"></a>Partie Deux nombres et liste
L’en-tête affiche deux valeurs indiquant le nombre d’enregistrements obtenus à partir de requêtes de journal. La liste affiche les dix premiers résultats d’une requête, avec un graphique qui indique la valeur relative d’une colonne numérique ou ses changements avec le temps.

![Deux nombres et affichage de liste](media/view-designer-parts/view-two-numbers-list.png)

| Paramètre | Description |
|:--- |:--- |
| **Généralités** | |
| Titre du groupe |Texte affiché en haut de la vue. |
| Nouveau groupe |Sélectionnez ce lien pour créer un groupe dans la vue, en commençant avec la vue actuelle. |
| Icône |Fichier image affiché à côté du résultat dans l’en-tête. |
| Icône Utiliser |Sélectionnez ce lien pour afficher l’icône. |
| **Navigation par titre** | |
| Navigation par clic | Action effectuée lorsque vous cliquez sur l’en-tête.  Pour plus d’informations, consultez [Paramètres courants](#click-through-navigation). |
| **Titre** | |
| Légende |Texte affiché en haut de l’en-tête. |
| Requête |Requête à exécuter pour l’en-tête. Le nombre d’enregistrements retournés par la requête est affiché. |
| **Liste** | |
| Requête |Requête à exécuter pour obtenir la liste. Les deux premières propriétés des dix premiers enregistrements dans les résultats sont affichées. La première propriété est une valeur de texte et la seconde une valeur numérique. Les barres sont créées automatiquement en fonction de la valeur relative de la colonne numérique.<br><br>Utilisez la commande `Sort` dans la requête pour trier les enregistrements de la liste. Pour exécuter la requête et retourner tous les enregistrements, vous pouvez sélectionner **Afficher tout** . |
| Masquer le graphique |Sélectionnez ce lien pour désactiver le graphique à droite de la colonne numérique. |
| Activation des sparklines |Sélectionnez ce lien pour afficher un graphique Sparkline au lieu d’une barre horizontale. Pour plus d’informations, consultez [Paramètres courants](#sparklines). |
| Couleur |Couleur des barres ou graphiques Sparkline. |
| Opération |Opération à effectuer pour le graphique Sparkline. Pour plus d’informations, consultez [Paramètres courants](#sparklines). |
| Séparateur de noms et de valeurs |Délimiteur de caractère unique à utiliser pour analyser la propriété de texte en plusieurs valeurs. Pour plus d’informations, consultez [Paramètres courants](#sparklines). |
| Navigation par clic | Action effectuée lorsque vous cliquez sur un élément dans la liste.  Pour plus d’informations, consultez [Paramètres courants](#click-through-navigation). |
| **Liste** |**> Titres des colonnes** |
| Name |Texte affiché en haut de la première colonne. |
| Valeur |Texte affiché en haut de la deuxième colonne. |
| **Liste** |**&gt; Seuils** |
| Activer les seuils |Sélectionnez ce lien pour activer les seuils. Pour plus d’informations, consultez [Paramètres courants](#thresholds). |

## <a name="donut-and-list-part"></a>Partie Anneau et liste
L’en-tête affiche un nombre unique qui résume une colonne de valeur dans une requête de journal. L’anneau affiche sous forme graphique les résultats des trois premiers enregistrements.

![Vue Anneau et liste](media/view-designer-parts/view-donut-list.png)

| Paramètre | Description |
|:--- |:--- |
| **Généralités** | |
| Titre du groupe |Texte affiché en haut de la vignette. |
| Nouveau groupe |Sélectionnez ce lien pour créer un groupe dans la vue, en commençant avec la vue actuelle. |
| Icône |Fichier image affiché à côté du résultat dans l’en-tête. |
| Icône Utiliser |Sélectionnez ce lien pour afficher l’icône. |
| **En-tête** | |
| Intitulé |Texte affiché en haut de l’en-tête. |
| Sous-titre |Texte affiché sous le titre en haut de l’en-tête. |
| **Anneau** | |
| Requête |Requête à exécuter pour obtenir l’anneau. La première propriété est une valeur de texte et la seconde une valeur numérique. |
| Navigation par clic | Action effectuée lorsque vous cliquez sur l’en-tête.  Pour plus d’informations, consultez [Paramètres courants](#click-through-navigation). |
| **Anneau** |**&gt; Centrer** |
| Texte |Texte affiché sous la valeur à l’intérieur de l’anneau. |
| Opération |Opération à effectuer sur la valeur de propriété afin de la résumer en une valeur unique.<ul><li>Sum : additionne les valeurs de tous les enregistrements.</li><li>Percentage : pourcentage des enregistrements retournés par les valeurs figurant dans **Valeurs de résultat utilisées dans l’opération relative au centre** par rapport au nombre total d’enregistrements dans la requête.</li></ul> |
| Valeurs de résultat utilisées dans l’opération relative au centre |Vous pouvez sélectionner le signe plus (+) pour ajouter une ou plusieurs valeurs. Les résultats de la requête sont alors limités aux enregistrements dont vous avez spécifié les valeurs de propriété. Si aucune valeur n’est ajoutée, tous les enregistrements sont inclus dans la requête. |
| **Options supplémentaires** |**&gt; Couleurs** |
| Couleur 1<br>Couleur 2<br>Couleur 3 |Sélectionnez la couleur pour chacune des valeurs affichées dans l’anneau. |
| **Options supplémentaires** |**> Mappage avancé des couleurs** |
| Valeur du champ |Saisissez le nom d’un champ pour l’afficher dans une couleur différente s’il est inclus dans l’anneau. |
| Couleur |Sélectionnez la couleur pour le champ unique. |
| **Liste** | |
| Requête |Requête à exécuter pour obtenir la liste. Le nombre d’enregistrements retournés par la requête est affiché. |
| Masquer le graphique |Sélectionnez ce lien pour désactiver le graphique à droite de la colonne numérique. |
| Activation des sparklines |Sélectionnez ce lien pour afficher un graphique Sparkline au lieu d’une barre horizontale. Pour plus d’informations, consultez [Paramètres courants](#sparklines). |
| Couleur |Couleur des barres ou graphiques Sparkline. |
| Opération |Opération à effectuer pour le graphique Sparkline. Pour plus d’informations, consultez [Paramètres courants](#sparklines). |
| Séparateur de noms et de valeurs |Délimiteur de caractère unique à utiliser pour analyser la propriété de texte en plusieurs valeurs. Pour plus d’informations, consultez [Paramètres courants](#sparklines). |
| Navigation par clic | Action effectuée lorsque vous cliquez sur un élément dans la liste.  Pour plus d’informations, consultez [Paramètres courants](#click-through-navigation). |
| **Liste** |**> Titres des colonnes** |
| Name |Texte affiché en haut de la première colonne. |
| Valeur |Texte affiché en haut de la deuxième colonne. |
| **Liste** |**&gt; Seuils** |
| Activer les seuils |Sélectionnez ce lien pour activer les seuils. Pour plus d’informations, consultez [Paramètres courants](#thresholds). |

## <a name="two-timelines-and-list-part"></a>Partie Deux chronologies et liste
L’en-tête affiche les résultats de deux requêtes de journal dans le temps, sous forme d’histogrammes avec une légende affichant un nombre qui résume une colonne de valeur dans une requête de journal. La liste affiche les dix premiers résultats d’une requête, avec un graphique qui indique la valeur relative d’une colonne numérique ou ses changements avec le temps.

![Vue Deux chronologies et liste](media/view-designer-parts/view-two-timelines-list.png)

| Paramètre | Description |
|:--- |:--- |
| **Généralités** | |
| Titre du groupe |Texte affiché en haut de la vignette. |
| Nouveau groupe |Sélectionnez ce lien pour créer un groupe dans la vue, en commençant avec la vue actuelle. |
| Icône |Fichier image affiché à côté du résultat dans l’en-tête. |
| Icône Utiliser |Sélectionnez ce lien pour afficher l’icône. |
| **Navigation par titre** | |
| Navigation par clic | Action effectuée lorsque vous cliquez sur l’en-tête.  Pour plus d’informations, consultez [Paramètres courants](#click-through-navigation). |
| **Premier graphique<br>Deuxième graphique** | |
| Légende |Texte affiché sous la légende de la première série. |
| Couleur |Couleur à utiliser pour les colonnes de la série. |
| Requête |Requête à exécuter pour la première série. Le nombre d’enregistrements sur chaque intervalle de temps est représenté par les colonnes de graphique. |
| Opération |Opération à effectuer sur la valeur de propriété afin de la résumer en une valeur unique pour la légende.<ul><li>Sum : somme des valeurs de tous les enregistrements.</li><li>Average : moyenne des valeurs de tous les enregistrements.</li><li>Last Sample : valeur du dernier intervalle inclus dans le graphique.</li><li>First Sample : valeur du premier intervalle inclus dans le graphique.</li><li>Count : nombre d’enregistrements retournés par la requête.</li></ul> |
| **Liste** | |
| Requête |Requête à exécuter pour obtenir la liste. Le nombre d’enregistrements retournés par la requête est affiché. |
| Masquer le graphique |Sélectionnez ce lien pour désactiver le graphique à droite de la colonne numérique. |
| Activation des sparklines |Sélectionnez ce lien pour afficher un graphique Sparkline au lieu d’une barre horizontale. Pour plus d’informations, consultez [Paramètres courants](#sparklines). |
| Couleur |Couleur des barres ou graphiques Sparkline. |
| Opération |Opération à effectuer pour le graphique Sparkline. Pour plus d’informations, consultez [Paramètres courants](#sparklines). |
| Navigation par clic | Action effectuée lorsque vous cliquez sur un élément dans la liste.  Pour plus d’informations, consultez [Paramètres courants](#click-through-navigation). |
| **Liste** |**> Titres des colonnes** |
| Name |Texte affiché en haut de la première colonne. |
| Valeur |Texte affiché en haut de la deuxième colonne. |
| **Liste** |**&gt; Seuils** |
| Activer les seuils |Sélectionnez ce lien pour activer les seuils. Pour plus d’informations, consultez [Paramètres courants](#thresholds). |

## <a name="information-part"></a>Partie des informations
L’en-tête affiche un texte statique et un lien facultatif. La liste affiche un ou plusieurs éléments avec un titre et un texte statiques.

![Vue Informations](media/view-designer-parts/view-information.png)

| Paramètre | Description |
|:--- |:--- |
| **Généralités** | |
| Titre du groupe |Texte affiché en haut de la vignette. |
| Nouveau groupe |Sélectionnez ce lien pour créer un groupe dans la vue, en commençant avec la vue actuelle. |
| Couleur |Couleur d’arrière-plan de l’en-tête. |
| **En-tête** | |
| Image |Fichier image affiché dans l’en-tête. |
| Étiquette |Texte affiché dans l’en-tête. |
| **En-tête** |**&gt; Lien** |
| Étiquette |Texte du lien. |
| Url |URL du lien. |
| **Éléments d’information** | |
| Intitulé |Texte affiché pour le titre de chaque élément. |
| Contenu |Texte affiché pour chaque élément. |

## <a name="line-chart-callout-and-list-part"></a>Partie Graphique en courbes, légende et liste
L’en-tête affiche un graphique en courbes avec plusieurs séries à partir d’une requête de journal dans le temps, et une légende avec une valeur de synthèse. La liste affiche les dix premiers résultats d’une requête, avec un graphique qui indique la valeur relative d’une colonne numérique ou ses changements avec le temps.

![Vue Graphique en courbes, légende et liste](media/view-designer-parts/view-line-chart-callout-list.png)

| Paramètre | Description |
|:--- |:--- |
| **Généralités** | |
| Titre du groupe |Texte affiché en haut de la vignette. |
| Nouveau groupe |Sélectionnez ce lien pour créer un groupe dans la vue, en commençant avec la vue actuelle. |
| Icône |Fichier image affiché à côté du résultat dans l’en-tête. |
| Icône Utiliser |Sélectionnez ce lien pour afficher l’icône. |
| **En-tête** | |
| Intitulé |Texte affiché en haut de l’en-tête. |
| Sous-titre |Texte affiché sous le titre en haut de l’en-tête. |
| **Graphique en courbes** | |
| Requête |Requête à exécuter pour obtenir le graphique en courbes. La première propriété est une valeur de texte et la seconde une valeur numérique. Cette requête utilise habituellement le mot clé *measure* pour synthétiser les résultats. Si la requête utilise le mot clé *interval* , l’axe des abscisses (X) du graphique utilise cet intervalle de temps. Si la requête ne contient pas le mot clé *interval* , l’axe des abscisses utilise des intervalles d’une heure. |
| Navigation par clic | Action effectuée lorsque vous cliquez sur l’en-tête.  Pour plus d’informations, consultez [Paramètres courants](#click-through-navigation). |
| **Graphique en courbes** |**&gt; Légende** |
| Titre de la légende |Texte affiché au-dessus de la valeur de la légende. |
| Nom de la série |Valeur de propriété pour la série à utiliser pour la valeur de la légende. Si aucune série n’est fournie, tous les enregistrements de la requête sont utilisés. |
| Opération |Opération à effectuer sur la valeur de propriété afin de la résumer en une valeur unique pour la légende.<ul><li>Average : moyenne des valeurs de tous les enregistrements.</li><li>Count : nombre d’enregistrements retournés par la requête.</li><li>Last Sample : valeur du dernier intervalle inclus dans le graphique.</li><li>Max : valeur maximale des intervalles inclus dans le graphique.</li><li>Min : valeur minimale des intervalles inclus dans le graphique.</li><li>Sum : somme des valeurs de tous les enregistrements.</li></ul> |
| **Graphique en courbes** |**> Axe Y** |
| Utiliser l’échelle logarithmique |Sélectionnez ce lien pour utiliser une échelle logarithmique pour l’axe des ordonnées (Y). |
| Units |Spécifiez les unités à utiliser pour exprimer les valeurs retournées par la requête. Ces informations sont utilisées pour afficher sur le graphique des étiquettes indiquant les types de valeurs et, le cas échéant, pour convertir les valeurs. Le type d’ *Unité* spécifie la catégorie de l’unité, et définit les valeurs de type *Unité actuelle* disponibles. Si vous sélectionnez une valeur pour l’option *Convertir en* , les valeurs numériques sont converties du type *Unité actuelle* au type *Convertir en* . |
| Étiquette personnalisée |Texte affiché pour l’axe Y en regard de l’étiquette du type d’ *Unité* . Si aucune étiquette n’est spécifiée, seul le type d’ *Unité* est affiché. |
| **Liste** | |
| Requête |Requête à exécuter pour obtenir la liste. Le nombre d’enregistrements retournés par la requête est affiché. |
| Masquer le graphique |Sélectionnez ce lien pour désactiver le graphique à droite de la colonne numérique. |
| Activation des sparklines |Sélectionnez ce lien pour afficher un graphique Sparkline au lieu d’une barre horizontale. Pour plus d’informations, consultez [Paramètres courants](#sparklines). |
| Couleur |Couleur des barres ou graphiques Sparkline. |
| Opération |Opération à effectuer pour le graphique Sparkline. Pour plus d’informations, consultez [Paramètres courants](#sparklines). |
| Séparateur de noms et de valeurs |Délimiteur de caractère unique à utiliser pour analyser la propriété de texte en plusieurs valeurs. Pour plus d’informations, consultez [Paramètres courants](#sparklines). |
| Navigation par clic | Action effectuée lorsque vous cliquez sur un élément dans la liste.  Pour plus d’informations, consultez [Paramètres courants](#click-through-navigation). |
| **Liste** |**> Titres des colonnes** |
| Name |Texte affiché en haut de la première colonne. |
| Valeur |Texte affiché en haut de la deuxième colonne. |
| **Liste** |**&gt; Seuils** |
| Activer les seuils |Sélectionnez ce lien pour activer les seuils. Pour plus d’informations, consultez [Paramètres courants](#thresholds). |

## <a name="line-chart-and-list-part"></a>Partie Graphique en courbes et liste
L’en-tête affiche un graphique en courbes avec plusieurs séries à partir d’une requête de journal dans le temps. La liste affiche les dix premiers résultats d’une requête, avec un graphique qui indique la valeur relative d’une colonne numérique ou ses changements avec le temps.

![Vue Graphique en courbes et liste](media/view-designer-parts/view-line-chart-callout-list.png)

| Paramètre | Description |
|:--- |:--- |
| **Généralités** | |
| Titre du groupe |Texte affiché en haut de la vignette. |
| Nouveau groupe |Sélectionnez ce lien pour créer un groupe dans la vue, en commençant avec la vue actuelle. |
| Icône |Fichier image affiché à côté du résultat dans l’en-tête. |
| Icône Utiliser |Sélectionnez ce lien pour afficher l’icône. |
| **En-tête** | |
| Intitulé |Texte affiché en haut de l’en-tête. |
| Sous-titre |Texte affiché sous le titre en haut de l’en-tête. |
| **Graphique en courbes** | |
| Requête |Requête à exécuter pour obtenir le graphique en courbes. La première propriété est une valeur de texte et la seconde une valeur numérique. Cette requête utilise habituellement le mot clé *measure* pour synthétiser les résultats. Si la requête utilise le mot clé *interval* , l’axe des abscisses (X) du graphique utilise cet intervalle de temps. Si la requête ne contient pas le mot clé *interval* , l’axe des abscisses utilise des intervalles d’une heure. |
| Navigation par clic | Action effectuée lorsque vous cliquez sur l’en-tête.  Pour plus d’informations, consultez [Paramètres courants](#click-through-navigation). |
| **Graphique en courbes** |**> Axe Y** |
| Utiliser l’échelle logarithmique |Sélectionnez ce lien pour utiliser une échelle logarithmique pour l’axe des ordonnées (Y). |
| Units |Spécifiez les unités à utiliser pour exprimer les valeurs retournées par la requête. Ces informations sont utilisées pour afficher sur le graphique des étiquettes indiquant les types de valeurs et, le cas échéant, pour convertir les valeurs. Le type d’ *Unité* spécifie la catégorie de l’unité, et définit les valeurs de type *Unité actuelle* disponibles. Si vous sélectionnez une valeur pour l’option *Convertir en* , les valeurs numériques sont converties du type *Unité actuelle* au type *Convertir en* . |
| Étiquette personnalisée |Texte affiché pour l’axe Y en regard de l’étiquette du type d’ *Unité* . Si aucune étiquette n’est spécifiée, seul le type d’ *Unité* est affiché. |
| **Liste** | |
| Requête |Requête à exécuter pour obtenir la liste. Le nombre d’enregistrements retournés par la requête est affiché. |
| Masquer le graphique |Sélectionnez ce lien pour désactiver le graphique à droite de la colonne numérique. |
| Activation des sparklines |Sélectionnez ce lien pour afficher un graphique Sparkline au lieu d’une barre horizontale. Pour plus d’informations, consultez [Paramètres courants](#sparklines). |
| Couleur |Couleur des barres ou graphiques Sparkline. |
| Opération |Opération à effectuer pour le graphique Sparkline. Pour plus d’informations, consultez [Paramètres courants](#sparklines). |
| Séparateur de noms et de valeurs |Délimiteur de caractère unique à utiliser pour analyser la propriété de texte en plusieurs valeurs. Pour plus d’informations, consultez [Paramètres courants](#sparklines). |
| Navigation par clic | Action effectuée lorsque vous cliquez sur un élément dans la liste.  Pour plus d’informations, consultez [Paramètres courants](#click-through-navigation). |
| **Liste** |**> Titres des colonnes** |
| Name |Texte affiché en haut de la première colonne. |
| Valeur |Texte affiché en haut de la deuxième colonne. |
| **Liste** |**&gt; Seuils** |
| Activer les seuils |Sélectionnez ce lien pour activer les seuils. Pour plus d’informations, consultez [Paramètres courants](#thresholds). |

## <a name="stack-of-line-charts-part"></a>Partie de pile de graphiques de courbes
La pile de graphique en courbes affiche trois graphiques en courbes distincts avec plusieurs séries à partir d’une requête de journal dans le temps, comme illustré ici :

![Pile de graphiques en courbes](media/view-designer-parts/view-stack-line-charts.png)

| Paramètre | Description |
|:--- |:--- |
| **Généralités** | |
| Titre du groupe |Texte affiché en haut de la vignette. |
| Nouveau groupe |Sélectionnez ce lien pour créer un groupe dans la vue, en commençant avec la vue actuelle. |
| Icône |Fichier image affiché à côté du résultat dans l’en-tête. |
| **Graphique 1<br>Graphique 2<br>Graphique 3** |**&gt; En-tête** |
| Intitulé |Texte affiché en haut du graphique. |
| Sous-titre |Texte affiché sous le titre en haut du graphique. |
| **Graphique 1<br>Graphique 2<br>Graphique 3** |**Graphique en courbes** |
| Requête |Requête à exécuter pour obtenir le graphique en courbes. La première propriété est une valeur de texte et la seconde une valeur numérique. Cette requête utilise habituellement le mot clé *measure* pour synthétiser les résultats. Si la requête utilise le mot clé *interval* , l’axe des abscisses (X) du graphique utilise cet intervalle de temps. Si la requête ne contient pas le mot clé *interval* , l’axe des abscisses utilise des intervalles d’une heure. |
| Navigation par clic | Action effectuée lorsque vous cliquez sur l’en-tête.  Pour plus d’informations, consultez [Paramètres courants](#click-through-navigation). |
| **Graphique** |**> Axe Y** |
| Utiliser l’échelle logarithmique |Sélectionnez ce lien pour utiliser une échelle logarithmique pour l’axe des ordonnées (Y). |
| Units |Spécifiez les unités à utiliser pour exprimer les valeurs retournées par la requête. Ces informations sont utilisées pour afficher sur le graphique des étiquettes indiquant les types de valeurs et, le cas échéant, pour convertir les valeurs. Le type d’ *Unité* spécifie la catégorie de l’unité, et définit les valeurs de type *Unité actuelle* disponibles. Si vous sélectionnez une valeur pour l’option *Convertir en* , les valeurs numériques sont converties du type *Unité actuelle* au type *Convertir en* . |
| Étiquette personnalisée |Texte affiché pour l’axe Y en regard de l’étiquette du type d’ *Unité* . Si aucune étiquette n’est spécifiée, seul le type d’ *Unité* est affiché. |

## <a name="common-settings"></a>Paramètres courants
Les sections suivantes décrivent les paramètres communs à plusieurs parties de visualisation.

### <a name="name-and-value-separator"></a><a name="name-value-separator"></a>Séparateur de noms et de valeurs
Le séparateur de noms et de valeurs est le délimiteur à caractère unique à utiliser pour analyser la propriété de texte d’une requête de liste en plusieurs valeurs. Si vous spécifiez un délimiteur, vous pouvez fournir des noms pour chaque champ, en les séparant par le même délimiteur que dans le champ **Nom** .

Par exemple, imaginez une propriété nommée *Location* incluant des valeurs telles que *Redmond-Building 41* et *Bellevue-Building 12* . Vous pouvez spécifier un tiret (-) comme séparateur de noms et de valeurs, et *City-Building* comme nom. Chaque valeur est alors analysée en deux propriétés respectivement nommées *City* et *Building* .

### <a name="click-through-navigation"></a><a name="click-through-navigation"></a>Navigation par clic
La navigation par clic définit l’action à effectuer quand vous cliquez sur un en-tête ou élément de liste dans une vue.  Soit une requête s’ouvre dans le [Log Analytics](../log-query/log-query-overview.md), soit une autre vue apparaît.

Le tableau suivant décrit les paramètres de la navigation par clic.

| Paramètre           | Description |
|:--|:--|
| Recherche dans les journaux (automatique) | Requête de journal à exécuter quand vous sélectionnez un élément d’en-tête.  Il s’agit de la même requête de journal que celle sur laquelle se base l’élément.
| Recherche dans les journaux        | Requête de journal à exécuter quand vous sélectionnez un élément dans une liste.  Tapez la requête dans la zone **Requête de navigation** .   Utilisez *{selected item}* pour inclure la syntaxe de l’élément sélectionné par l’utilisateur.  Par exemple, si la requête comprend une colonne nommée *Computer* et que la requête de navigation est *{selected item}* , une requête telle que *Computer="MyComputer"* est exécutée quand vous sélectionnez un ordinateur. Si la requête de navigation est *Type=Event {selected item}* , la requête *Type=Event Computer="MyComputer"* est exécutée. |
| Affichage              | Vue à ouvrir lorsque vous sélectionnez un élément d’en-tête ou un élément dans une liste.  Sélectionnez le nom d’une vue dans votre espace de travail dans la zone **Nom de la vue** . |



### <a name="sparklines"></a><a name="sparklines"></a>Graphiques Sparkline
Une sparkline est un petit graphique en courbes qui illustre la valeur d’une entrée de liste au fil du temps. Pour les parties de visualisation avec une liste, vous pouvez sélectionner si vous souhaitez afficher une barre horizontale, qui indique la valeur relative d’une colonne numérique, ou un graphique Sparkline, qui indique sa valeur au fil du temps.

Le tableau suivant décrit les paramètres pour les graphiques Sparkline :

| Paramètre | Description |
|:--- |:--- |
| Activation des sparklines |Sélectionnez ce lien pour afficher un graphique Sparkline au lieu d’une barre horizontale. |
| Opération |Si les sparklines sont activées, il s’agit de l’opération à effectuer sur chaque propriété dans la liste pour calculer les valeurs du graphique sparkline.<ul><li>Last Sample : dernière valeur de la série sur l’intervalle de temps.</li><li>Max : valeur maximale de la série sur l’intervalle de temps.</li><li>Min : valeur minimale de la série sur l’intervalle de temps.</li><li>Sum : somme des valeurs de la série sur l’intervalle de temps.</li><li>Summary : utilise la même commande `measure` que la requête dans l’en-tête.</li></ul> |

### <a name="thresholds"></a><a name="thresholds"></a>Seuils
Les seuils vous permettent d’afficher une icône de couleur en regard de chaque élément dans une liste. Ils fournissent un indicateur visuel rapide des éléments qui dépassent une valeur particulière ou sont compris dans une plage particulière. Par exemple, vous pouvez afficher une icône verte pour les éléments avec une valeur acceptable, jaune si la valeur est dans une plage qui indique un avertissement, et rouge si elle dépasse une valeur d’erreur.

Lorsque vous activez des seuils pour une partie, vous devez spécifier un ou plusieurs seuils. Si la valeur d’un élément est supérieure à une valeur de seuil et inférieure à la valeur de seuil suivante, la couleur de cette valeur est utilisée. Si l’élément est supérieur à la valeur de seuil la plus élevée, une autre couleur est utilisée. 

Chaque ensemble de seuils a un seuil avec la valeur **par défaut** . Il s’agit de la couleur définie si aucune autre valeur n’est dépassée. Vous pouvez ajouter ou supprimer des seuils en sélectionnant le bouton **Ajouter** (+) ou **Supprimer** (x).

Le tableau suivant décrit les paramètres pour les seuils :

| Paramètre | Description |
|:--- |:--- |
| Activer les seuils |Sélectionnez ce lien pour afficher une icône de couleur à gauche de chaque valeur. L’icône indique l’intégrité de la valeur par rapport aux seuils spécifiés. |
| Name |Nom de la valeur de seuil. |
| Seuil |Valeur du seuil. La couleur d’intégrité de chaque élément de liste est définie sur la couleur de la valeur du seuil le plus élevé dépassée par la valeur de l’élément. Si aucune valeur de seuil n’est dépassée, une couleur par défaut est utilisée. |
| Couleur |Couleur qui indique la valeur de seuil. |

## <a name="next-steps"></a>Étapes suivantes
* Découvrez-en plus sur les [requêtes de journal](../log-query/log-query-overview.md) pour prendre en charge les requêtes dans des composants de visualisation.
