---
title: Panneau de synthèse de l’appareil StorSimple Virtual Array | Microsoft Docs
description: Décrit le panneau de synthèse des appareils pour StorSimple Device Manager et explique comment l’utiliser pour surveiller l’intégrité de votre instance StorSimple Virtual Array.
services: storsimple
documentationcenter: ''
author: manuaery
manager: syadav
editor: ''
ms.assetid: a13c1ea7-6428-4234-84a6-0ebf51670a85
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/29/2016
ms.author: manuaery
ms.openlocfilehash: 1ab72788c768568366f2627055015c74028330b5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005875"
---
# <a name="use-the-device-summary-blade-for-storsimple-device-manager-connected-to-storsimple-virtual-array"></a>Utiliser le panneau de synthèse des appareils pour StorSimple Device Manager connecté à StorSimple Virtual Array

## <a name="overview"></a>Vue d’ensemble

Le panneau de synthèse des appareils StorSimple Device Manager procure un résumé d’une instance StorSimple Virtual Array enregistrée auprès d’un service StorSimple Device Manager, en mettant en évidence les problèmes qui nécessitent l’intervention d’un administrateur système. Ce didacticiel présente le panneau de synthèse des appareils, explique le contenu et la fonctionnalité et décrit les travaux que vous pouvez effectuer à partir de ce panneau.

Le panneau de synthèse des appareils affiche les informations suivantes :

![Page du tableau de bord d’un appareil](./media/storsimple-virtual-array-device-summary/device-blade.png)



## <a name="management"></a>Gestion

Dans le panneau des appareils StorSimple, vous avez accès aux options de gestion de votre appareil StorSimple. Les commandes de gestion se trouvent dans la partie supérieure et sur le côté gauche du panneau. Utilisez ces options pour ajouter des partages ou des volumes, ou pour mettre à jour ou basculer votre tableau virtuel.

La zone des éléments principaux comporte quelques-unes des propriétés importantes, telles que le statut, le modèle, la version logicielle, ainsi qu’un lien vers l’**IU web** du tableau. Si vous vous trouvez sur un réseau interne, vous pouvez directement lancer [l’interface utilisateur web locale](storsimple-ova-web-ui-admin.md) pour administrer votre tableau virtuel.

![Éléments principaux des appareils](./media/storsimple-virtual-array-device-summary/device-essentials.png)

## <a name="storsimple-device-summary"></a>Synthèse des appareils StorSimple

* La vignette **Alertes** fournit un instantané de l’ensemble des alertes actives associées à votre tableau virtuel, et groupées par gravité. Cliquez sur la vignette pour ouvrir le panneau **Alertes**, puis cliquez sur une alerte pour afficher des détails supplémentaires sur cette alerte, y compris les actions recommandées. Vous pouvez également effacer l'alerte si le problème a été résolu.

* La vignette **Capacité** affiche le stockage primaire configuré et demeurant sur l’appareil virtuel, relativement au stockage total disponible pour cet élément. **Configuré** fait référence au volume de stockage préparé et alloué pour l’utilisation, **Restant** fait référence à la capacité restante pouvant être configurée sur cet appareil. La capacité **Hiérarchisée restante** est la capacité disponible pouvant être configurée, notamment le cloud, tandis que la capacité **Locale restante** est la capacité restant sur les disques associés à ce tableau virtuel.

* Dans le tableau **Utilisation**, vous pouvez consulter le stockage primaire utilisé au sein de votre tableau virtuel, ainsi que le stockage cloud consommé ces 7 derniers jours, la période par défaut. Utilisez l’option **Modifier** du coin supérieur droit du tableau pour définir une période différente.

* La vignette **Partages** ou **Volumes** fournit une synthèse des nombres de partages ou de volumes de votre appareil, groupés par état. Cliquez sur la vignette pour ouvrir le panneau de la liste **Partages** ou **Volumes**, puis cliquez sur un partage ou un volume individuel pour afficher ou modifier ses propriétés. Pour plus d’informations, découvrez comment [gérer des partages](storsimple-virtual-array-manage-shares.md) ou [gérer des volumes](storsimple-virtual-array-manage-volumes.md).

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment :
- [Gérer des partages sur une instance StorSimple Virtual Array](storsimple-virtual-array-manage-shares.md)
    
- [Gérer des volumes sur une instance StorSimple Virtual Array](storsimple-virtual-array-manage-volumes.md)

