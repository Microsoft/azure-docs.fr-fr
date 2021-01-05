---
title: Mise à niveau de vos visualisations de tableau de bord Log Analytics
description: Apprenez à mettre à niveau vos visualisations du tableau de bord Log Analytics avec des requêtes pouvant fournir de puissants insights.
ms.subservice: logs
ms.topic: article
author: rboucher
ms.author: robb
ms.date: 07/01/2020
ms.openlocfilehash: 96aea35a4796eae9d31062cedaf917a736f0fe82
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92547127"
---
# <a name="upgrading-your-log-analytics-dashboard-visualizations"></a>Mise à niveau de vos visualisations de tableau de bord Log Analytics

En février 2020, nous avons introduit une technologie de visualisation améliorée. Elle permet d’améliorer et de renforcer votre capacité à visualiser les résultats des requêtes et d’obtenir des informations précieuses, rapidement. 

Pour plus d’informations sur cette mise à niveau, consultez cette [mise à jour Azure](https://azure.microsoft.com/updates/azure-monitor-log-analytics-upgraded-results-visualization/). 

Cette nouvelle technologie de visualisation ouvre la voie à des expériences nouvelles et améliorées autour de votre jeu de résultats de requête. 

## <a name="dashboards-in-azure"></a>Tableaux de bord dans Azure

Les tableaux de bord Azure sont un moyen de visualiser l’état de toute votre surface d’exposition Azure. Ils sont conçus pour fournir un volet unique et transparent sur l’état de votre patrimoine Azure et vous permettent d’utiliser un large éventail de raccourcis vers des actions courantes. 

Pour plus d’informations, consultez [Tableaux de bord Azure](../../azure-portal/azure-portal-dashboards.md).


## <a name="upgrading-log-analytics-dashboard-parts"></a>Mise à niveau des mosaïques du tableau de bord Log Analytics

La nouvelle technologie de visualisation résout certains problèmes courants liés à l’ancienne implémentation et introduit de nouvelles capacités permettant d’épingler des mosaïques Log Analytics : 

- **Mêmes types disponibles**  : tous les types de visualisation disponibles dans Log Analytics sont disponibles sous forme de mosaïques épinglées sur les tableaux de bord.

- **Apparence cohérente**  : l’apparence visuelle des mosaïques épinglées est désormais presque identique à celle de Log Analytics. Les différences sont dues à des optimisations qui nécessitent des différences subtiles dans le contenu des données du contrôle. Pour plus d’informations sur ces différences, consultez la section Considérations de ce document.

- **Info-bulles et étiquettes**  : les nouvelles visualisations Log Analytics épinglées prennent en charge les info-bulles. Les graphiques à secteurs et en anneau prennent désormais en charge les étiquettes.

- **Légendes interactives**  : cliquer sur la légende de visualisation permet d’ajouter ou de supprimer des dimensions du contrôle épinglé comme dans Log Analytics.

## <a name="stage-1---opt-in-upgrade-message"></a>Étape 1 : Accepter le message de mise à niveau

Lorsqu’une mosaïque épinglée Log Analytics peut être mise à niveau, une nouvelle notification d’ *acceptation* s’affiche sur cette mosaïque dans les tableaux de bord, ce qui permet aux utilisateurs de mettre à niveau leur visualisation. Pour faire l’expérience des nouvelles visualisations, il est nécessaire de les mettre à niveau sur le tableau de bord.

 
![Barre latérale](media/dashboard-upgrade/update-message-1.png)
 
![Capture d’écran montrant comment mettre à jour la visualisation des vignettes.](media/dashboard-upgrade/update-message-2.png)

> [!WARNING]
> Une fois le tableau de bord publié, la mise à niveau est irréversible. Toutefois, les modifications sont abandonnées si vous quittez le tableau de bord sans le republier.  

Une fois que vous cliquez dessus, la visualisation est mise à jour vers la nouvelle technologie. Des différences subtiles dans la visualisation peuvent s’introduire afin de s’aligner sur leur apparence dans Log Analytics.

Une fois les visualisations mises à niveau, vous devez republier votre tableau de bord pour que la modification prenne effet.

![Capture d’écran montrant les visualisations mises à niveau.](media/dashboard-upgrade/update-message-3.png)

## <a name="stage-2---migration-of-all-dashboards"></a>Étape 2 : Migrer tous les tableaux de bord

Une fois la période initiale d’acceptation terminée, l’équipe Log Analytics met à niveau tous les tableaux de bord du système. L’alignement de tous les tableaux de bord Azure permet à l’équipe d’introduire plus de visualisations et d’apporter des améliorations à tous les niveaux.

## <a name="considerations"></a>Considérations

Les visualisations Log Analytics épinglées sur un tableau de bord ont un comportement spécifique conçu pour une expérience optimale. Examinez les considérations de conception suivantes lorsque vous épinglez une visualisation sur un tableau de bord.

### <a name="query-time-scope---30-day-limit"></a>Durée d’une requête : limitée à 30 jours

Comme les tableaux de bord peuvent contenir plusieurs visualisations provenant de plusieurs requêtes, la durée d’une requête épinglée est limitée à 30 jours. Une requête peut uniquement s’exécuter sur un intervalle de temps inférieur ou égal à 30 jours. Cette limitation vise à garantir un temps de chargement raisonnable du tableau de bord.

### <a name="query-data-values---25-values-and-other-grouping"></a>Valeurs de données des requêtes : 25 valeurs et autres regroupements

Les tableaux de bord peuvent être visuellement denses et complexes. Afin de réduire la charge cognitive lors de la consultation d’un tableau de bord, nous optimisons les visualisations en limitant l’affichage à 25 types de données différents. Lorsqu’il y en a plus de 25, Log Analytics optimise les données. Il affiche individuellement les 25 types en distinguant la plupart des données, puis regroupe les valeurs restantes dans une valeur « autre ». Le tableau suivant en est l’illustration.  

![Capture d’écran montrant un tableau de bord avec 25 types de données différents.](media/dashboard-upgrade/values-25-limit.png)

### <a name="dashboard-refresh-on-load"></a>Actualisation du tableau de bord lors du chargement

Les tableaux de bord sont actualisés lors du chargement. Toutes les requêtes relatives aux visualisations Log Analytics épinglées sur le tableau de bord sont exécutées, et le tableau de bord est actualisé une fois qu’il est chargé. Si la page du tableau de bord reste ouverte, les données du tableau de bord sont actualisées toutes les 60 minutes.

## <a name="next-steps"></a>Étapes suivantes

[Créer et partager des tableaux de bord dans Log Analytics](../learn/tutorial-logs-dashboards.md)
