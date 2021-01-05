---
title: Utiliser le panneau de synthèse de l’appareil dans le service StorSimple Device Manager
description: Décrit le panneau de synthèse de l’appareil offert par le service StorSimple Device Manager et explique comment l’utiliser pour afficher les métriques de stockage et les initiateurs connectés, et rechercher le numéro de série et l’IQN pour un appareil.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 07/03/2017
ms.author: alkohli
ms.openlocfilehash: 34bbf4d723e46663efe77560245db74d9dea1fc4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017149"
---
# <a name="use-the-device-summary-in-storsimple-device-manager-service"></a>Utiliser le panneau de synthèse de l’appareil dans le service StorSimple Device Manager

## <a name="overview"></a>Vue d’ensemble
Le panneau de synthèse de l’appareil StorSimple offre une vue d’ensemble des informations relatives à un appareil StorSimple spécifique, à la différence du panneau de synthèse du service, qui regroupe les informations concernant tous les appareils inclus dans votre solution Microsoft Azure StorSimple.

Le panneau de synthèse de l’appareil fournit une vue récapitulative d’un appareil de la gamme StorSimple 8000 inscrit auprès d’une instance StorSimple Device Manager donnée, en mettant en évidence les problèmes qui nécessitent l’intervention d’un administrateur système. Ce didacticiel présente le panneau de synthèse des appareils, explique le contenu et la fonctionnalité et décrit les travaux que vous pouvez effectuer à partir de ce panneau.

Le panneau de synthèse des appareils affiche les informations suivantes :

![Panneau de synthèse de l’appareil](./media/storsimple-8000-device-dashboard/device-summary1.png)

## <a name="management-command-bar"></a>Barre de commandes de gestion

Dans le panneau des appareils StorSimple, vous avez accès aux options de gestion de votre appareil StorSimple. Les commandes de gestion se trouvent dans la partie supérieure et sur le côté gauche du panneau. Utilisez ces options pour ajouter des partages ou des volumes, ou pour mettre à jour ou basculer votre appareil.

![Barre de commandes de gestion](./media/storsimple-8000-device-dashboard/device-summary2.png)

## <a name="essentials"></a>Essentials

La zone des éléments principaux comporte quelques-unes des propriétés importantes, telles que le statut, le modèle, l’IQN cible et la version logicielle. 

![Éléments principaux des appareils](./media/storsimple-8000-device-dashboard/device-summary3.png)

## <a name="monitoring"></a>Surveillance

* La vignette **Alertes** fournit un instantané de l’ensemble des alertes actives associées à votre appareil, regroupées par niveau de gravité.

    ![Vignette Alertes](./media/storsimple-8000-device-dashboard/device-summary4.png)

    Cliquez sur la vignette pour ouvrir le panneau **Alertes**, puis cliquez sur une alerte pour afficher des détails supplémentaires sur cette alerte, y compris les actions recommandées. Vous pouvez également effacer l'alerte si le problème a été résolu.

    ![Cliquer sur la vignette Alertes](./media/storsimple-8000-device-dashboard/device-summary10.png)

* La vignette **État et intégrité** fournit des informations sur l’intégrité des composants matériels d’un appareil, y compris l’état de l’appareil. L’appareil peut présenter l’état Hors connexion, En ligne, Désactivé ou Prêt pour la configuration.

    ![Vignette État et intégrité](./media/storsimple-8000-device-dashboard/device-summary5.png)

* La vignette **Volumes** fournit une synthèse du nombre de volumes de votre appareil, ces derniers étant regroupés par état.

    ![Vignette Volumes](./media/storsimple-8000-device-dashboard/device-summary6.png)

    Cliquez sur la vignette pour ouvrir le panneau de la liste **Volumes**, puis cliquez sur un volume individuel pour afficher ou modifier ses propriétés.
    
    ![Cliquer sur la vignette Volumes](./media/storsimple-8000-device-dashboard/device-summary9.png)
    
    Pour plus d’informations, consultez [Gérer les volumes](storsimple-8000-manage-volumes-u2.md).

* Dans le graphique **Utilisation**, vous pouvez consulter le stockage principal utilisé pour votre appareil, ainsi que le stockage cloud consommé ces 7 derniers jours, la période par défaut.

     ![Mosaïque Utilisation](./media/storsimple-8000-device-dashboard/device-summary7.png)
    
     Pour choisir une autre échelle de temps, utilisez l’option **Modifier** accessible dans l’angle supérieur droit du graphique.

     ![Modifier le graphique Utilisation](./media/storsimple-8000-device-dashboard/device-summary12.png)

     Ce graphique vous permet d’examiner les métriques relatives au stockage principal total (quantité de données écrites par les hôtes sur votre appareil), ainsi que le stockage cloud total consommé par votre appareil au cours d’une période donnée.
  
     Dans ce contexte, le *stockage principal* fait référence à la quantité totale de données écrites par l’hôte, et peut être détaillé par type de volume : le *stockage hiérarchisé principal* inclut à la fois les données stockées localement et les données stockées dans le cloud. Le *stockage principal attaché localement* inclut uniquement les données stockées localement. Le *stockage cloud*, quant à lui, est une mesure de la quantité totale de données stockées dans le cloud. Ce stockage inclut les données hiérarchisées et les sauvegardes. Les données stockées dans le cloud sont dédupliquées et compressées, tandis que le stockage principal indique la quantité de stockage utilisée avant que les données ne soient dédupliquées et compressées. Vous pouvez comparer ces deux nombres pour avoir une idée du taux de compression. Qu’il s’agisse du stockage principal ou du stockage cloud, les quantités indiquées dépendent de la fréquence de suivi que vous configurez. Par exemple, si vous optez pour une fréquence hebdomadaire, le graphique affiche des données pour chaque jour de la semaine précédente.

     Pour afficher la quantité de stockage cloud consommée au fil du temps, sélectionnez l’option **STOCKAGE CLOUD UTILISÉ**. Pour afficher le stockage total des données écrites par l’hôte, sélectionnez les options **STOCKAGE HIÉRARCHISÉ PRINCIPAL UTILISÉ** et **STOCKAGE PRINCIPAL ÉPINGLÉ LOCALEMENT UTILISÉ**. 
     Pour plus d’informations, consultez [Utilisation du service StorSimple Device Manager pour surveiller votre appareil StorSimple](./storsimple-8000-monitor-device.md).


* La vignette **Capacité** affiche le stockage primaire configuré et restant sur l’appareil, relativement au stockage total disponible pour ce dernier. **Configuré** fait référence au volume de stockage préparé et alloué pour l’utilisation, **Restant** fait référence à la capacité restante pouvant être configurée sur cet appareil. 

    ![Mosaïque Utilisation 2](./media/storsimple-8000-device-dashboard/device-summary8.png)

    Cliquez sur cette vignette pour voir comment la capacité est approvisionnée sur les volumes attachés localement et hiérarchisés. La **capacité hiérarchisée restante** est la capacité disponible, notamment dans le cloud, pouvant être approvisionnée, tandis que la **capacité locale restante** correspond à la capacité restant sur les disques attachés à cet appareil.

    ![Cliquer sur le graphique Utilisation](./media/storsimple-8000-device-dashboard/device-summary13.png)


## <a name="next-steps"></a>Étapes suivantes
* Accédez à des informations supplémentaires sur le [panneau de synthèse du service StorSimple](storsimple-8000-service-dashboard.md).
* En savoir plus sur l’[utilisation du service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).