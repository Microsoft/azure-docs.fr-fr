---
title: Paramètres temporels des classeurs Azure Monitor
description: Découvrez comment définir des paramètres temporels pour permettre aux utilisateurs de définir le contexte temporel de l’analyse. Presque tous les rapports utilisent les paramètres temporels.
services: azure-monitor
manager: carmonm
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 10/23/2019
ms.openlocfilehash: 37f2219313a0d4c533d587a765dd22d527a5910f
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92143447"
---
# <a name="workbook-time-parameters"></a>Paramètres de temps des classeurs

Les paramètres de temps, utilisés par presque tous les rapports, permettent aux utilisateurs de définir le contexte temporel de l’analyse. Relativement simples à configurer et à utiliser, ils permettent aux auteurs de spécifier les intervalles de temps, y compris personnalisés, à afficher dans la liste déroulante. 

## <a name="creating-a-time-parameter"></a>Créer un paramètre de temps
1. Démarrez avec un classeur vide en mode d’édition.
2. Choisissez _Ajouter des paramètres_ dans les liens du classeur.
3. Cliquez sur le bouton bleu _Ajouter un paramètre_.
4. Dans le volet Nouveau paramètre qui apparaît, entrez :
    1. Nom du paramètre : `TimeRange`
    2. Type de paramètre : `Time range picker`
    3. Obligatoire : `checked`
    4. Intervalles de temps disponibles : Last hour (Dernière heure), Last 12 hours (12 dernières heures), Last 24 hours (Dernières 24 heures), Last 48 hours (48 dernières heures), Last 3 days (3 derniers jours), Last 7 days (7 derniers jours) et Allow custom time range selection (Autoriser la sélection d’un intervalle de temps personnalisé)
5. Choisissez « Enregistrer » dans la barre d’outils pour créer le paramètre.

    ![Image représentant la création d’un paramètre d’intervalle de temps](./media/workbooks-time/time-settings.png)

Voici à quoi ressemble le classeur en mode lecture.

![Image représentant un paramètre d’intervalle de temps en mode lecture](./media/workbooks-time/parameters-time.png)

## <a name="referencing-a-time-parameter"></a>Faire référence à un paramètre de temps
### <a name="via-bindings"></a>Par liaisons
1. Ajoutez un contrôle de requête au classeur et sélectionnez une ressource Application Insights.
2. La plupart des contrôles de classeur prennent en charge un sélecteur d’étendue _d’Intervalle de temps_. Ouvrez la liste déroulante _Intervalle de temps_ et sélectionnez `{TimeRange}` dans le groupe de paramètres d’intervalle de temps en bas.
3. Cela permet de lier le paramètre d’intervalle de temps à l’intervalle de temps du graphique. La plage temporelle de l’exemple de requête est maintenant Last 24 hours (Dernières 24 heures).
4. Exécuter la requête pour afficher les résultats

    ![Image représentant une référence à un paramètre d’intervalle de temps par des liaisons](./media/workbooks-time/time-binding.png)

### <a name="in-kql"></a>En KQL
1. Ajoutez un contrôle de requête au classeur et sélectionnez une ressource Application Insights.
2. Dans KQL, entrez un filtre de plage temporelle avec le paramètre `| where timestamp {TimeRange}`.
3. Cela développe ainsi l’heure d’évaluation de la requête : `| where timestamp > ago(1d)`, à savoir la valeur de l’intervalle de temps du paramètre.
4. Exécuter la requête pour afficher les résultats

    ![Image représentant une référence à un intervalle de temps en KQL](./media/workbooks-time/time-in-code.png)

### <a name="in-text"></a>En texte 
1. Ajoutez un contrôle de texte au classeur.
2. Dans le Markdown, entrez `The chosen time range is {TimeRange:label}`.
3. Choisissez _Fin de l’édition_.
4. Le contrôle de texte affiche le texte : _L’intervalle de temps choisi est Last 24 hours (Dernières 24 heures)_ .

## <a name="time-parameter-options"></a>Options de paramètre de temps
| Paramètre | Explication | Exemple |
| ------------- |:-------------|:-------------|
| `{TimeRange}` | Étiquette de l’intervalle de temps | 24 dernières heures |
| `{TimeRange:label}` | Étiquette de l’intervalle de temps | 24 dernières heures |
| `{TimeRange:value}` | Valeur de l’intervalle de temps | > ago(1d) |
| `{TimeRange:query}` | Requête d’intervalle de temps | > ago(1d) |
| `{TimeRange:start}` | Heure de début de l’intervalle de temps | 3/20/2019 4:18 PM |
| `{TimeRange:end}` | Heure de fin de l’intervalle de temps | 3/21/2019 4:18 PM |
| `{TimeRange:grain}` | Grain de l’intervalle de temps | 30 m |


### <a name="using-parameter-options-in-a-query"></a>Utiliser des options de paramètre dans une requête
```kusto
requests
| make-series Requests = count() default = 0 on timestamp from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain}
```

## <a name="next-steps"></a>Étapes suivantes

* [Commencez](./workbooks-overview.md#visualizations) à en apprendre davantage sur les nombreuses options pour les visualisations enrichies des classeurs.
* [Contrôlez](workbooks-access-control.md) et partagez l’accès à vos ressources de classeur.