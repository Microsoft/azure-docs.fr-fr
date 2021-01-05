---
title: Résolution des problèmes liés aux insights basés sur des classeurs Azure Monitor
description: Fournit des conseils de résolution de problèmes liés aux insights basés sur des classeurs Azure Monitor pour les services que sont notamment Azure Key Vault, Azure CosmosDB, Stockage Azure et Azure Cache pour Redis.
services: azure-monitor
ms.author: mbullwin
author: mrbullwinkle
ms.topic: conceptual
ms.date: 06/17/2020
ms.openlocfilehash: 90743a213092aacfa10139c12f47bc11faf0d86d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86498710"
---
# <a name="troubleshooting-workbook-based-insights"></a>Résolution des problèmes liés aux insights basés sur des classeurs

Cet article vous aide à diagnostiquer et résoudre certains problèmes courants que vous êtes susceptible de rencontrer quand vous utilisez des insights basés sur des classeurs Azure Monitor.


## <a name="why-can-i-only-see-200-resources"></a>Pourquoi je ne vois pas plus de 200 ressources ?

Le nombre de ressources sélectionnées est limité à 200, quel que soit le nombre d’abonnements sélectionnés.

## <a name="what-happens-when-i-click-on-a-recently-pinned-tile-in-the-dashboard"></a>Que se passe-t-il quand je clique sur une vignette épinglée récemment dans le tableau de bord ?

* Si vous cliquez n’importe où sur la vignette, vous passez à l’onglet à partir duquel la vignette a été épinglée. Par exemple, si vous épinglez un graphique dans l’onglet « Vue d’ensemble », quand vous cliquez sur cette vignette dans le tableau de bord, il ouvre cette vue par défaut. Toutefois, si vous épinglez un graphique à partir de votre copie enregistrée, il ouvre la vue de votre copie enregistrée.
* L’icône de filtre dans le coin supérieur gauche du titre ouvre l’onglet « Configurer les paramètres de vignette ».
* L’icône représentant une ellipse en haut à droite vous donne les options « Personnaliser les données de titre », « Personnaliser », « Actualiser » et « Supprimer du tableau de bord ».

## <a name="what-happens-when-i-save-a-workbook"></a>Que se passe-t-il quand j’enregistre un classeur ?

* Quand vous enregistrez un classeur, vous pouvez en créer une copie avec vos modifications et en changer le titre. L’enregistrement ne remplace pas le classeur, le classeur actuel est toujours la vue par défaut.
* Un **classeur** non enregistré est simplement la vue par défaut.

## <a name="why-dont-i-see-all-my-subscriptions-in-the-portal"></a>Pourquoi je ne vois pas tous mes abonnements sur le portail ?

Le portail affichera uniquement les données des abonnements sélectionnés lors de son lancement. Pour modifier les abonnements sélectionnés, accédez au coin supérieur droit et cliquez sur le bloc-notes avec une icône de filtre. Cette option affiche l’onglet **Répertoire + abonnements**.

![Répertoire + abonnement](./media/storage-insights-overview/fqa3.png)

## <a name="what-is-time-range"></a>Qu’est-ce que l’intervalle de temps ?

L’intervalle de temps affiche des données pour un intervalle de temps donné. Par exemple, si l’intervalle de temps est de 24 heures, il indique les données des dernières 24 heures.

## <a name="what-is-time-granularity-time-grain"></a>Qu’est-ce que la granularité temporelle (fragment de temps) ?

La granularité temporelle est la différence de temps entre deux points de données. Par exemple, si le fragment de temps est défini sur 1 seconde, cela signifie que les métriques sont collectées chaque seconde.

## <a name="what-is-the-time-granularity-once-we-pin-any-part-of-the-workbooks-to-a-dashboard"></a>Quelle est la granularité temporelle après avoir épinglé une partie des classeurs à un tableau de bord ?

La granularité temporelle par défaut est définie sur Automatique, et ne peut pas être modifiée pour l’instant.

## <a name="how-do-i-change-the-timespan-time-range-of-the-workbook-step-on-my-dashboard"></a>Comment changer l’intervalle de temps de l’étape de classeur sur mon tableau de bord ?

Par défaut, l’intervalle de temps de la vignette de votre tableau de bord est défini sur 24 heures. Pour modifier cette valeur, cliquez sur l’ellipse en haut à droite, sélectionnez **Personnaliser les données de vignette**, cochez la case « Remplacer les paramètres de temps du tableau de bord au niveau du titre », puis choisissez un intervalle de temps à l’aide du menu déroulant.  

![Sélectionnez l’ellipse dans le coin droit de la vignette et choisissez Personnaliser ces données](./media/storage-insights-overview/fqa-data-settings.png)

![Dans Configurer les paramètres de vignette, sélectionnez la liste déroulante Intervalle de temps pour modifier la plage](./media/storage-insights-overview/fqa-timespan.png)

## <a name="how-do-i-change-the-title-of-the-workbook-or-a-workbook-step-i-pinned-to-a-dashboard"></a>Comment changer le titre pour une étape de classeur ou un classeur que j’ai épinglé à un tableau de bord ?

Le titre de l’étape de classeur ou du classeur qui est épinglé à un tableau de bord conserve le même nom que celui qu’il avait dans le classeur. Pour modifier le titre, vous devez enregistrer votre propre copie du classeur. Vous pourrez alors renommer le classeur avant d’appuyer sur Enregistrer.

![Sélectionnez Enregistrer en haut pour enregistrer une copie du classeur et modifier son nom](./media/storage-insights-overview/fqa-change-workbook-name.png)

Pour changer le nom d’une étape dans votre classeur enregistré, sélectionnez Modifier sous l’étape, puis sélectionnez l’engrenage en dessous des paramètres.

![Sélectionnez Modifier en bas de l’étape d’un classeur pour ouvrir les Paramètres](./media/storage-insights-overview/fqa-edit.png)
![Dans Paramètres, sélectionnez l’engrenage en bas pour pouvoir modifier le nom de l’étape](./media/storage-insights-overview/fqa-change-name.png)

## <a name="next-steps"></a>Étapes suivantes

Découvrez plus en détail les scénarios que les classeurs sont destinés à prendre en charge, comment créer et personnaliser des rapports existants et plus encore en consultant [Créer des rapports interactifs avec les classeurs Azure Monitor](../platform/workbooks-overview.md).
