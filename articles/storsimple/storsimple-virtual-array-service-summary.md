---
title: Panneau de synthèse du service StorSimple Virtual Array | Microsoft Docs
description: Décrit le panneau de synthèse des services pour StorSimple Device Manager et explique comment l’utiliser pour surveiller l’intégrité de votre instance StorSimple Virtual Array.
services: storsimple
documentationcenter: ''
author: manuaery
manager: syadav
editor: ''
ms.assetid: 8a2b9a84-b0e6-48b9-b366-d16f004241a5
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2016
ms.author: manuaery
ms.openlocfilehash: 31220a8f8b012e08d46195b60a26be9cfc261be5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85514518"
---
# <a name="use-the-service-summary-blade-for-storsimple-device-manager-connected-to-storsimple-virtual-array"></a>Utiliser le panneau de synthèse des services pour StorSimple Device Manager connecté à StorSimple Virtual Array
## <a name="overview"></a>Vue d’ensemble
Le panneau de synthèse des services StorSimple Device Manager fournit un résumé des instances StorSimple Virtual Array (également appelés appareils virtuels ou appareils virtuels locaux StorSimple) qui sont connectés au service, mettant en surbrillance ceux qui requièrent l’attention particulière d’un administrateur système. Ce didacticiel présente le panneau de synthèse des services, explique le contenu et la fonctionnalité et décrit les tâches que vous pouvez effectuer à partir de ce panneau.

![Tableau de bord du service](./media/storsimple-virtual-array-service-summary/service-blade.png)

## <a name="management-commands-and-essentials"></a>Commandes de gestion et éléments principaux
Dans le panneau de synthèse StorSimple, vous avez accès aux options de gestion de votre service StorSimple Device Manager et aux tableaux virtuels enregistrés auprès de ce service. Les commandes de gestion se trouvent dans la partie supérieure et sur le côté gauche du panneau.

Utilisez ces options pour effectuer diverses opérations, telles qu’ajouter des partages ou des volumes, ou surveiller les différents travaux exécutés sur les tableaux virtuels.

La zone des éléments principaux comporte quelques-unes des propriétés importantes comme le groupe de ressources, l’emplacement et l’abonnement dans lequel a été créée votre instance StorSimple Device Manager.

## <a name="storsimple-device-manager-service-summary"></a>Synthèse des services StorSimple Device Manager
* La vignette **Alertes** fournit un instantané de toutes les alertes actives sur tous les appareils virtuels, groupées par gravité. Cliquez sur la vignette pour ouvrir le panneau **Alertes**, où vous pouvez cliquer sur une alerte individuelle pour afficher des détails supplémentaires, y compris les actions recommandées. Vous pouvez également effacer l'alerte si le problème a été résolu.
* La vignette **Capacité** affiche le stockage primaire qui est configuré et restant sur les appareils virtuels, par rapport au stockage total disponible sur l’ensemble des appareils virtuels. **Configuré** fait référence au volume de stockage préparé et alloué pour l’utilisation, **Restant** fait référence à la capacité restante pouvant être configurée sur l’ensemble des appareils virtuels. La capacité **Hiérarchisée restante** est la capacité disponible pouvant être configurée, notamment le cloud, tandis que la capacité **Locale restante** est la capacité restant sur les disques associés aux tableaux virtuels.
* Dans le tableau **Utilisation**, vous avez accès aux mesures appropriées pour vos appareils virtuels. Vous pouvez afficher le stockage principal utilisé sur tous les appareils virtuels, ainsi que le stockage de cloud utilisé par les appareils virtuels au cours des 7 derniers jours, la période par défaut. Utilisez l’option **Modifier** du coin supérieur droit du tableau pour définir une période différente.
* La vignette **Appareils** fournit une synthèse des tableaux virtuels dans votre instance StorSimple Device Manager, regroupés par état d’appareil. Cliquez sur cette vignette pour ouvrir le panneau de la liste des **Appareils**, puis cliquez sur un appareil spécifique afin d’accéder aux données le concernant. Vous pouvez également exécuter des actions spécifiques sur un appareil à partir d’un panneau de synthèse considéré. Pour plus d’informations sur le panneau de synthèse des appareils, accédez à [Panneau de synthèse des appareils](storsimple-virtual-array-device-summary.md).

## <a name="view-the-activity-logs"></a>Afficher les journaux d’activité
Pour afficher les opérations effectuées au sein de votre instance StorSimple Device Manager, cliquez sur le lien **Journaux d’activité** situé sur le côté gauche du panneau de synthèse des services StorSimple. Vous accédez au panneau **Journaux d’activité**, où vous avez accès à une synthèse des opérations récentes effectuées.

![Journaux d’activité](./media/storsimple-virtual-array-service-summary/activity-log.png)

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [utiliser l’interface utilisateur web locale pour gérer votre StorSimple Virtual Array](storsimple-ova-web-ui-admin.md).

