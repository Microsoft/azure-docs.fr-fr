---
title: Tutoriel - Créer un graphique de métriques dans Azure Monitor
description: Découvrez comment créer un graphique de métrique avec Azure Metrics Explorer.
author: bwren
ms.author: bwren
ms.subservice: metrics
ms.topic: tutorial
ms.date: 03/09/2020
ms.openlocfilehash: 6ce9183fad50c43ea7070f5fa09e0d445caf89df
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96451247"
---
# <a name="tutorial-create-a-metrics-chart-in-azure-monitor"></a>Tutoriel : Créer un graphique de métriques dans Azure Monitor
Metrics Explorer est une fonctionnalité d’Azure Monitor dans le portail Azure qui vous permet de créer des graphiques à partir de valeurs de métriques, de corréler visuellement des tendances et d’examiner des pics et des creux dans des valeurs de métriques. Utilisez Metrics Explorer pour examiner l’intégrité et l’utilisation de vos ressources Azure ou pour tracer des graphiques à partir de métriques personnalisées. 

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Sélectionner une métrique pour laquelle vous souhaitez tracer un graphique.
> * Effectuer différentes agrégations de valeurs de métriques.
> * Modifier l’intervalle de temps et la granularité du graphique.

Voici une vidéo qui présente un scénario plus détaillé que la procédure décrite dans cet article. Si vous débutez avec les métriques, nous vous suggérons de d’abord lire cet article, puis de regarder la vidéo pour plus de détails. 

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4qO59]

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vous avez besoin d’une ressource Azure à superviser. Vous pouvez utiliser n’importe quelle ressource de votre abonnement Azure qui prend en charge les métriques. Pour déterminer si une ressource prend en charge les métriques, accédez à son menu dans le portail Azure et vérifiez qu’une option **Métriques** figure dans la section **Supervision** du menu.


## <a name="log-in-to-azure"></a>Connexion à Azure
Connectez-vous au portail Azure à l’adresse [https://portal.azure.com](https://portal.azure.com).

## <a name="open-metrics-explorer-and-select-a-scope"></a>Ouvrir Metrics Explorer et sélectionner une étendue
Vous pouvez ouvrir Metrics Explorer à partir du menu Azure Monitor ou du menu d’une ressource dans le portail Azure. Les métriques de toutes les ressources sont disponibles quelle que soit l’option que vous utilisez. 

1. Sélectionnez **Métriques** dans le menu **Azure Monitor** ou dans la section **Supervision** du menu d’une ressource.

1. Sélectionnez l’**Étendue**, autrement dit la ressource pour laquelle vous souhaitez afficher les métriques. L’étendue est déjà remplie si vous avez ouvert Metrics Explorer à partir du menu d’une ressource.

    ![Sélectionner une étendue](media/tutorial-metrics-explorer/scope-picker.png)

2. Sélectionnez un **Espace de noms** si l’étendue en a plusieurs. L’espace de noms est simplement une méthode d’organisation des métriques pour que vous puissiez les retrouver facilement. Par exemple, les comptes de stockage ont des espaces de noms distincts pour le stockage des métriques de fichiers, tables, objets Blob et files d’attente. Plusieurs types de ressources n’ont qu’un seul espace de noms.

3. Sélectionnez une métrique dans la liste des métriques disponibles pour l’étendue et l’espace de noms sélectionnés.

    ![Sélectionner une métrique](media/tutorial-metrics-explorer/metric-picker.png)

4. Si vous le souhaitez, modifiez l’**Agrégation** des métriques. Cela définit la manière dont les valeurs des métriques sont agrégées en fonction de la granularité temporelle du graphique. Par exemple, si la granularité temporelle est définie sur 15 minutes et que l’agrégation est définie sur sum, chaque point du graphique correspond à la somme de toutes les valeurs collectées durant chaque segment de 15 minutes.

    ![La capture d’écran montre un graphique intitulé Sum Ingress for contosoretailweb (Somme de l’entrée pour contosoretailweb).](media/tutorial-metrics-explorer/chart.png)

5. Utilisez le bouton **Ajouter une métrique** et répétez ces étapes si vous souhaitez afficher plusieurs métriques tracées dans le même graphique. Pour afficher plusieurs graphiques dans une même vue, sélectionnez le bouton **Nouveau graphique**.

## <a name="select-a-time-range-and-granularity"></a>Sélectionner un intervalle de temps et une granularité

Par défaut, le graphique affiche les dernières 24 heures de données de métriques. Utilisez le sélecteur d’heure pour changer l’**Intervalle de temps** du graphique ou la **Granularité temporelle** qui définit l’intervalle de temps pour chaque point de données. Le graphique utilise l’agrégation spécifiée pour calculer toutes les valeurs échantillonnées durant la granularité temporelle spécifiée.

![Modifier le panneau d’intervalle de temps](media/tutorial-metrics-explorer/time-picker.png)


Utilisez le curseur de **balayage temporel** pour étudier une zone intéressante du graphique, telle qu’un pic ou une baisse. Placez le pointeur de la souris au début de la zone, cliquez et maintenez le bouton gauche de la souris enfoncé, faites glisser de l’autre côté de la zone, puis relâchez le bouton. Cet intervalle de temps sera agrandi dans le graphique. 

![Curseur de balayage temporel](media/tutorial-metrics-explorer/time-brush.png)

## <a name="apply-dimension-filters-and-splitting"></a>Appliquer des filtres de dimension et un fractionnement
Pour découvrir des fonctionnalités avancées qui vous permettent d’effectuer des analyses supplémentaires de vos métriques et d’identifier les valeurs hors norme potentielles dans vos données, consultez les références suivantes.

- Le [filtrage](../platform/metrics-charts.md#apply-filters-to-charts) vous permet de choisir les valeurs de dimension à inclure dans le graphique. Par exemple, vous souhaiterez peut-être afficher uniquement les requêtes ayant réussi dans la représentation graphique d’une métrique de *temps de réponse du serveur*. 

- Le [fractionnement](../platform/metrics-charts.md#apply-splitting-to-a-chart) contrôle si le graphique affiche des lignes distinctes pour chaque valeur d’une dimension ou agrège les valeurs dans une seule ligne. Par exemple, vous souhaiterez peut-être voir une ligne pour le temps de réponse moyen sur toutes les instances de serveur, ou voir des lignes distinctes pour chaque serveur. 

Consultez les [exemples de graphiques](../platform/metric-chart-samples.md) auxquels un filtrage et un fractionnement sont appliqués.

## <a name="advanced-chart-settings"></a>Paramètres de graphique avancés

Vous pouvez personnaliser le style et le titre du graphique, et modifier les paramètres de graphique avancés. Lorsque vous avez terminé la personnalisation, épinglez-le à un tableau de bord pour enregistrer votre travail. Vous pouvez également configurer des alertes de métriques. Pour en savoir plus sur ces fonctionnalités et d’autres fonctionnalités avancées d’Azure Monitor Metrics Explorer, consultez [Fonctionnalités avancées d’Azure Metrics Explorer](../platform/metrics-charts.md#lock-boundaries-of-chart-y-axis).


## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez appris à utiliser les métriques dans Azure Monitor, découvrez comment les utiliser pour envoyer des alertes proactives.

> [!div class="nextstepaction"]
> [Créer, afficher et gérer des alertes de métrique à l'aide d'Azure Monitor](../platform/metrics-charts.md#create-alert-rules)

