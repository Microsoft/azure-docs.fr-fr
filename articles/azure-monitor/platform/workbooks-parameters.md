---
title: Paramètres de création des classeurs Azure Monitor
description: Découvrez comment des paramètres permettent aux auteurs de classeurs de collecter des entrées auprès des consommateurs, et d’y faire référence dans d’autres parties du classeur.
services: azure-monitor
manager: carmonm
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 10/23/2019
ms.openlocfilehash: 5dccf8a11691ed02d50b88afc481ec0612fe15b9
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92143590"
---
# <a name="workbook-parameters"></a>Paramètres du classeur

Les paramètres permettent aux auteurs de classeurs de collecter des entrées auprès des consommateurs et d’y faire référence dans d’autres parties du classeur, généralement pour limiter l’étendue du jeu de résultats ou définir le bon visuel. Il s’agit d’une fonctionnalité clé pour créer des expériences et des rapports interactifs. 

Les classeurs permettent de contrôler la façon dont les contrôles de paramètres sont présentés aux consommateurs : zone de texte ou liste déroulante, sélection unique ou multiple, valeurs issues de texte, JSON, KQL, Azure Resource Graph, etc.  

Les types de paramètres pris en charge sont les suivants :
* [Temps](workbooks-time.md) : permet à l’utilisateur de sélectionner des intervalles de temps préremplis ou de sélectionner un intervalle personnalisé
* [Liste déroulante](workbooks-dropdowns.md) : permet à l’utilisateur de sélectionner une valeur ou un ensemble de valeurs
* [Texte](workbooks-text.md) : permet à l’utilisateur d’entrer du texte arbitraire
* [Ressource](workbooks-resources.md) : permet à l’utilisateur de sélectionner une ou plusieurs ressources Azure
* [Abonnement](workbooks-resources.md) : permet à l’utilisateur de sélectionner une ou plusieurs ressources d’abonnement Azure
* Type de ressource : permet à l’utilisateur de sélectionner une ou plusieurs valeurs de type de ressource Azure
* Emplacement : permet à l’utilisateur de sélectionner une ou plusieurs valeurs d’emplacement Azure

Il est possible de faire référence à ces valeurs de paramètre dans d’autres parties de classeurs avec des liaisons ou des expansions de valeur.

## <a name="creating-a-parameter"></a>Créer un paramètre
1. Démarrez avec un classeur vide en mode d’édition.
2. Choisissez _Ajouter des paramètres_ dans les liens du classeur.
3. Cliquez sur le bouton bleu _Ajouter un paramètre_.
4. Dans le volet Nouveau paramètre qui apparaît, entrez :
    1. Nom du paramètre : `TimeRange` *(notez que les __noms__ de paramètres **ne peuvent pas** comporter d’espaces ou de caractères spéciaux)*
    2. Nom d’affichage : `Time Range` *(cependant, les __noms d’affichage__ peuvent comporter des espaces, des caractères spéciaux, des emojis, etc.)*
    2. Type de paramètre : `Time range picker`
    3. Obligatoire : `checked`
    4. Intervalles de temps disponibles : Last hour (Dernière heure), Last 12 hours (12 dernières heures), Last 24 hours (Dernières 24 heures), Last 48 hours (48 dernières heures), Last 3 days (3 derniers jours), Last 7 days (7 derniers jours) et Allow custom time range selection (Autoriser la sélection d’un intervalle de temps personnalisé)
5. Choisissez « Enregistrer » dans la barre d’outils pour créer le paramètre.

   ![Image représentant la création d’un paramètre d’intervalle de temps](./media/workbooks-parameters/time-settings.png)

Voici à quoi ressemble le classeur en mode lecture, dans le style « Pills ».

   ![Image représentant un paramètre d’intervalle de temps en mode lecture](./media/workbooks-parameters/parameters-time.png)

## <a name="referencing-a-parameter"></a>Faire référence à un paramètre
### <a name="via-bindings"></a>Par liaisons
1. Ajoutez un contrôle de requête au classeur et sélectionnez une ressource Application Insights.
2. Ouvrez la liste déroulante _Intervalle de temps_ et sélectionnez l’option `Time Range` dans la section Paramètres en bas.
3. Cela permet de lier le paramètre d’intervalle de temps à l’intervalle de temps du graphique. La plage temporelle de l’exemple de requête est maintenant Last 24 hours (Dernières 24 heures).
4. Exécuter la requête pour afficher les résultats

    ![Image représentant une référence à un paramètre d’intervalle de temps par des liaisons](./media/workbooks-parameters/time-binding.png)

### <a name="in-kql"></a>En KQL
1. Ajoutez un contrôle de requête au classeur et sélectionnez une ressource Application Insights.
2. Dans KQL, entrez un filtre de plage temporelle avec le paramètre `| where timestamp {TimeRange}`.
3. Cela développe ainsi l’heure d’évaluation de la requête : `| where timestamp > ago(1d)`, à savoir la valeur de l’intervalle de temps du paramètre.
4. Exécuter la requête pour afficher les résultats

    ![Image représentant une référence à un intervalle de temps en KQL](./media/workbooks-parameters/time-in-code.png)

### <a name="in-text"></a>En texte 
1. Ajoutez un contrôle de texte au classeur.
2. Dans le Markdown, entrez `The chosen time range is {TimeRange:label}`.
3. Choisissez _Fin de l’édition_.
4. Le contrôle de texte affiche le texte : _L’intervalle de temps choisi est Last 24 hours (Dernières 24 heures)_ .

## <a name="parameter-options"></a>Options de paramètre
La section _En texte_ utilise le `label` du paramètre au lieu de sa valeur. Les paramètres présentent différentes options en fonction de leur type : par exemple, les sélecteurs d’intervalle de temps acceptent la valeur, l’étiquette, la requête, le début, la fin et le grain.

Utilisez la section `Previews` du volet _Modifier le paramètre_ pour afficher les options d’expansion de votre paramètre :

![Image représentant les options d’un paramètre d’intervalle de temps](./media/workbooks-parameters/time-previews.png)

## <a name="next-steps"></a>Étapes suivantes

* [Commencez](./workbooks-overview.md#visualizations) à en apprendre davantage sur les nombreuses options pour les visualisations enrichies des classeurs.
* [Contrôlez](workbooks-access-control.md) et partagez l’accès à vos ressources de classeur.