---
title: Utiliser le panneau de synthèse du service pour un appareil de la gamme StorSimple 8000
description: Décrit le panneau de synthèse du service StorSimple Manager, et explique comment l’utiliser pour surveiller l’intégrité de votre solution StorSimple.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/27/2017
ms.author: alkohli
ms.openlocfilehash: 7b4b697b3d27b57212fc59396e1f8111e297d6cf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85514950"
---
# <a name="use-the-service-summary-blade-for-storsimple-8000-series-device"></a>Utiliser le panneau de synthèse du service pour un appareil de la gamme StorSimple 8000

## <a name="overview"></a>Vue d’ensemble

Le panneau de synthèse du service de StorSimple Device Manager fournit un aperçu de tous les appareils connectés au service StorSimple Device Manager, en mettant en surbrillance ceux qui requièrent une attention particulière d’un administrateur système. Ce didacticiel présente le panneau de synthèse du service, explique le contenu et la fonction du tableau de bord, et décrit les tâches que vous pouvez effectuer à partir de cette page.

![Récapitulatif du service](./media/storsimple-8000-service-dashboard/service-summary1.png)


## <a name="management-commands"></a>Commandes de gestion

Le panneau de synthèse du service StorSimple présente les options de gestion de votre service StorSimple Device Manager, ainsi que les appareils de la gamme StorSimple 8000 inscrits auprès de ce service. Les commandes de gestion se trouvent dans la partie supérieure et sur le côté gauche du panneau.

![Barre de commandes](./media/storsimple-8000-service-dashboard/service-summary2.png)

Utilisez ces options pour effectuer diverses opérations, telles qu’ajouter des partages ou des volumes, ou surveiller les différents travaux en cours d’exécution sur les appareils StorSimple.


## <a name="essentials"></a>Essentials

La zone des éléments principaux comporte quelques-unes des propriétés importantes comme le groupe de ressources, l’emplacement et l’abonnement dans lequel a été créée votre instance StorSimple Device Manager.

![Essentials](./media/storsimple-8000-service-dashboard/service-summary3.png)

## <a name="storsimple-device-manager-service-summary"></a>Synthèse des services StorSimple Device Manager

* La vignette **Alertes** fournit un instantané de toutes les alertes actives sur tous les appareils, groupées par gravité.

    ![Vignette alertes](./media/storsimple-8000-service-dashboard/service-summary4.png)

    Cliquez sur la vignette pour ouvrir le panneau **Alertes**, où vous pouvez cliquer sur une alerte individuelle pour afficher des détails supplémentaires, y compris les actions recommandées. Vous pouvez également effacer l'alerte si le problème a été résolu.

    ![Cliquer sur la vignette Alertes](./media/storsimple-8000-service-dashboard/service-summary8.png)

* La vignette **Capacité** affiche le stockage primaire approvisionné et restant sur tous les appareils, par rapport au stockage total disponible sur ceux-ci. **Approvisionné** fait référence au volume de stockage préparé et alloué pour l’utilisation, **Restant** fait référence à la capacité restante pouvant être approvisionnée sur tous les appareils.

    ![Vignette Capacité](./media/storsimple-8000-service-dashboard/service-summary6.png)

    La capacité **Hiérarchisée restante** est la capacité disponible pouvant être approvisionnée, notamment le cloud, tandis que la capacité **Locale restante** est la capacité encore disponible sur les disques associés aux appareils de la gamme StorSimple 8000.


* Le graphique **Utilisation** présente les mesures appropriées pour vos appareils. Vous pouvez voir le stockage principal utilisé sur tous les appareils, ainsi que le stockage cloud utilisé par les appareils au cours des 7 derniers jours, soit la période par défaut. 

    ![Mosaïque Utilisation](./media/storsimple-8000-service-dashboard/service-summary7.png) 

    Pour choisir une autre échelle de temps, utilisez l’option **Modifier** accessible dans l’angle supérieur droit du graphique.

     ![Cliquer sur la Utilisation](./media/storsimple-8000-service-dashboard/service-summary10.png)

     ![Exporter les données du graphique](./media/storsimple-8000-service-dashboard/service-summary11.png)

* La vignette **Appareils** fournit une synthèse du nombre d’appareils de la gamme StorSimple 8000 présents dans votre StorSimple Device Manager, regroupés par état d’appareil. 

    ![Vignette Appareils](./media/storsimple-8000-service-dashboard/service-summary5.png)

    Cliquez sur cette vignette pour ouvrir le panneau de la liste des **Appareils**, puis cliquez sur un appareil spécifique afin d’accéder aux données le concernant. Vous pouvez également exécuter des actions spécifiques d’un appareil à partir d’un panneau de synthèse d’appareil donné. Pour plus d’informations sur le panneau de synthèse des appareils, accédez à [Panneau de synthèse des appareils](storsimple-8000-device-dashboard.md).

    ![Cliquer sur la vignette Appareils](./media/storsimple-8000-service-dashboard/service-summary9.png)

## <a name="view-the-activity-logs"></a>Afficher les journaux d’activité

Pour afficher les opérations effectuées au sein de votre instance StorSimple Device Manager, cliquez sur le lien **Journaux d’activité** situé sur le côté gauche du panneau de synthèse des services StorSimple. Vous accédez au panneau **Journaux d’activité**, où vous avez accès à une synthèse des opérations récentes effectuées.

![Journaux d’activité](./media/storsimple-8000-service-dashboard/activity-logs1.png)
## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [l’utilisation du service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).

