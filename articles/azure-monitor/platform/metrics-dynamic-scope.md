---
title: Affichage de plusieurs ressources dans Metrics Explorer
description: Découvrez comment visualiser plusieurs ressources dans Azure Monitor Metrics Explorer
author: ritaroloff
services: azure-monitor
ms.topic: conceptual
ms.date: 12/14/2020
ms.author: riroloff
ms.subservice: metrics
ms.openlocfilehash: 724809dbce3ca1b5a36f4da0ba5c03d0f78897f5
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97577544"
---
# <a name="viewing-multiple-resources-in-metrics-explorer"></a>Affichage de plusieurs ressources dans Metrics Explorer

Le sélecteur d’étendue de ressource vous permet d’afficher les mesures sur plusieurs ressources qui se trouvent dans le même abonnement et la même région. Vous trouverez ci-dessous des instructions sur la façon d’afficher plusieurs ressources dans Azure Monitor Metrics Explorer. 

## <a name="selecting-a-resource"></a>Sélection d’une ressource 

Sélectionnez **Métriques** dans le menu **Azure Monitor** ou dans la section **Supervision** du menu d’une ressource. Cliquez sur le bouton « Sélectionner une étendue » pour ouvrir le sélecteur d’étendue de ressource, ce qui vous permettra de sélectionner la ou les ressources dont vous souhaitez afficher les métriques. Le sélecteur doit être déjà rempli si vous avez ouvert Metrics Explorer à partir du menu d’une ressource. 

![Capture d’écran du sélecteur d’étendue de ressource mis en évidence dans un cadre rouge](./media/metrics-charts/019.png)

## <a name="selecting-multiple-resources"></a>Sélection de plusieurs ressources 

Certains types de ressources permettent d’interroger des métriques sur plusieurs ressources, à condition que celles-ci se trouvent dans les mêmes abonnement et localisation. Ces types de ressources se trouvent en haut de la liste déroulante « Types de ressources ». 

![Capture d’écran montrant une liste déroulante de types de ressources qui sont compatibles avec plusieurs ressources ](./media/metrics-charts/020.png)

> [!WARNING] 
> Vous devez disposer de l’autorisation Lecteur d’analyse au niveau de l’abonnement pour visualiser les métriques sur plusieurs ressources, groupes de ressources ou un abonnement. Pour ce faire, suivez les instructions de [ce document](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal).

Pour visualiser les métriques sur plusieurs ressources, commencez par sélectionner plusieurs ressources dans le sélecteur d’étendue de ressource. 

![Capture d’écran montrant comment sélectionner plusieurs ressources](./media/metrics-charts/021.png)

> [!NOTE]
> Vous pouvez uniquement sélectionner plusieurs ressources dans les mêmes type de ressource, localisation et abonnement. Les ressources ne répondant pas à ce critère ne peuvent pas être sélectionnées. 

Une fois la sélection effectuée, cliquez sur le bouton « Appliquer » pour l’enregistrer. 

## <a name="selecting-a-resource-group-or-subscription"></a>Sélection d’un groupe de ressources ou d’un abonnement 

> [!WARNING]
> Vous devez disposer de l’autorisation Lecteur d’analyse au niveau de l’abonnement pour visualiser les métriques sur plusieurs ressources, groupes de ressources ou un abonnement. 

Pour les types compatibles avec plusieurs ressources, vous pouvez également interroger les métriques sur un abonnement ou plusieurs groupes de ressources. Commencez par sélectionner un abonnement ou un ou plusieurs groupes de ressources : 

![Capture d’écran montrant comment effectuer une interrogation portant sur plusieurs groupes de ressources ](./media/metrics-charts/022.png)

Vous devez ensuite sélectionner un type de ressource et une localisation avant de pouvoir appliquer votre nouvelle étendue. 

![Capture d’écran montrant les groupes de ressources sélectionnés ](./media/metrics-charts/023.png)

Vous pouvez également développer les étendues sélectionnées pour vérifier les ressources concernées.

![Capture d’écran montrant les ressources sélectionnées au sein des groupes ](./media/metrics-charts/024.png)

Une fois que vous avez fini de sélectionner vos étendues, cliquez sur « Appliquer » pour enregistrer vos sélections. 

## <a name="splitting-and-filtering-by-resource-group-or-resources"></a>Division et filtrage par groupe de ressources ou ressource

Après avoir effectué le traçage de vos ressources, vous pouvez utiliser l’outil de division et de filtrage pour obtenir plus d’insights sur vos données. 

La division vous permet de comparer les différents segments de la métrique. Par exemple, quand vous effectuez le traçage d’une métrique pour plusieurs ressources, vous pouvez utiliser l’outil « Appliquer la division » pour diviser par ID de ressource ou groupe de ressources. Vous pouvez ainsi comparer facilement une métrique unique sur plusieurs ressources ou groupes de ressources.  

Par exemple, voici un graphique du pourcentage d’utilisation du processeur sur 9 machines virtuelles. En effectuant une division par ID de ressource, vous pouvez voir facilement le pourcentage d’utilisation du processeur par machine virtuelle. 

![Capture d’écran montrant comment vous pouvez utiliser la division pour voir le pourcentage d’utilisation du processeur par machine virtuelle](./media/metrics-charts/026.png)

En plus de la division, vous pouvez utiliser la fonctionnalité de filtrage pour afficher uniquement les groupes de ressources qui vous intéressent.  Par exemple, si vous souhaitez afficher le pourcentage d’utilisation du processeur des machines virtuelles pour un groupe de ressources spécifique, vous pouvez utiliser l’outil « Ajouter un filtre » pour filtrer par groupe de ressources. Dans cet exemple, nous filtrons par TailspinToys, ce qui a pour effet de supprimer les métriques associées aux ressources dans TailspinToysDemo. 

![Capture d’écran montrant comment filtrer par groupe de ressources](./media/metrics-charts/027.png)

## <a name="pinning-your-multi-resource-charts"></a>Épinglage de vos graphiques multi-ressources 

> [!WARNING] 
> Vous devez disposer de l’autorisation Lecteur d’analyse au niveau de l’abonnement pour visualiser les métriques sur plusieurs ressources, groupes de ressources ou un abonnement. Pour ce faire, suivez les instructions de [ce document](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal). 

Pour épingler votre graphique multi-ressource, suivez les instructions [ici](https://docs.microsoft.com/azure/azure-monitor/platform/metrics-charts#create-alert-rules). 

## <a name="next-steps"></a>Étapes suivantes

* [Résolution des problèmes de Metrics Explorer](metrics-troubleshoot.md)
* [Consulter la liste des métriques disponibles pour les services Azure](metrics-supported.md)
* [Consulter des exemples de graphiques configurés](metric-chart-samples.md)

