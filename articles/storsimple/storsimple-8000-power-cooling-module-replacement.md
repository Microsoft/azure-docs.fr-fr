---
title: Remplacer un PCM sur votre appareil StorSimple série 8000 | Microsoft Docs
description: Explique comment retirer et remplacer le PCM sur votre appareil StorSimple
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: troubleshooting
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/02/2017
ms.author: alkohli
ms.openlocfilehash: 045cec85174a88d1d608a4adc679461008852768
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85514588"
---
# <a name="replace-a-power-and-cooling-module-on-your-storsimple-device"></a>Remplacer un module d’alimentation et de refroidissement (PCM, Power and Cooling Module) sur votre appareil StorSimple
## <a name="overview"></a>Vue d’ensemble
Le PCM de votre appareil Microsoft Azure StorSimple se compose d’une alimentation et de ventilateurs de refroidissement qui sont contrôlés par le boîtier principal et le boîtier EBOD. Il n’existe qu’un seul modèle de PCM certifié pour chaque boîtier. Le boîtier principal est certifié pour un PCM de 764 W et le boîtier EBOD est certifié pour un PCM de 580 W. Bien que les PCM du boîtier principal et du boîtier EBOD soient différents, la procédure de remplacement est identique.

Ce didacticiel explique comment :

* Retirer un PCM
* Installer un PCM de remplacement

> [!IMPORTANT]
> Avant de retirer et de remplacer un PCM, passez en revue les informations de sécurité dans [Remplacement de composants matériels StorSimple](storsimple-8000-hardware-component-replacement.md).


## <a name="before-you-replace-a-pcm"></a>Avant de remplacer un PCM
Tenez compte des problèmes importants suivants avant de remplacer votre PCM :

* Si l’alimentation du PCM est défectueuse, laissez en place le module défectueux, mais retirez le cordon d’alimentation. Le ventilateur continue de recevoir du courant du boîtier et continue à fournir le refroidissement approprié. Si le ventilateur est défectueux, le PCM doit être remplacé immédiatement.
* Avant de retirer le PCM, déconnectez l’alimentation du PCM en éteignant le commutateur principal (s’il existe) ou en retirant physiquement le cordon d’alimentation. Ceci envoie un avertissement à votre système, indiquant qu’un arrêt de l’alimentation est imminent.
* Assurez-vous que l’autre PCM est fonctionnel pour que le système continue de fonctionner avant de remplacer le PCM défectueux. Un PCM défectueux doit être remplacé dès que possible par un PCM totalement opérationnel.
* Le remplacement du module PCM ne prend que quelques minutes, mais il doit être effectué dans un laps de temps de 10 minutes à partir du moment où vous le retirez, de façon à éviter une surchauffe.
* Notez que les modules PCM 764 W de remplacement en provenance de l’usine ne contiennent pas le module de batterie de secours. Vous devez retirer la batterie de votre module PCM défectueux, puis l’insérer dans le module de remplacement avant d’effectuer le remplacement. Pour plus d’informations, consultez la procédure à suivre pour [retirer et insérer un module de batterie de secours](storsimple-8000-battery-replacement.md).

## <a name="remove-a-pcm"></a>Retirer un PCM
Suivez ces instructions quand vous êtes prêt à retirer PCM de votre appareil Microsoft Azure StorSimple.

> [!NOTE]
> Avant de supprimer votre PCM, vérifiez que vous disposez d’un PCM de remplacement correct (764 W pour le boîtier principal ou 580 W pour le boîtier EBOD).

#### <a name="to-remove-a-pcm"></a>Pour retirer un PCM
1. Dans le portail Azure Classic, cliquez sur **Paramètres > Surveiller > Intégrité matérielle**. Vérifiez l’état des composants du PCM sous **Composants partagés** pour identifier le PCM défectueux :
   
   * Si une alimentation dans PCM 0 est défectueuse, l’état de **Alimentation dans PCM 0** sera en rouge.
   * Si une alimentation dans PCM 1 est défectueuse, l’état de **Alimentation dans PCM 1** sera en rouge.
   * Si le ventilateur dans PCM 1 est défectueux, l’état de **Refroidissement 0 pour PCM 0** ou **Refroidissement 1 pour PCM 0** sera en rouge.
2. Localisez le PCM défectueux à l’arrière du boîtier principal. Si vous exécutez un modèle 8600, identifiez le boîtier principal en examinant le numéro d’identification de l’unité système indiqué sur l’affichage LED du panneau avant. L’ID d’unité par défaut affiché sur le boîtier principal est **00**, alors que l’ID d’unité par défaut affiché sur le boîtier EBOD est **01**. Le diagramme et le tableau suivants expliquent le panneau avant de l’affichage LED.
   
    ![ID du système sur le panneau avant des opérations](./media/storsimple-power-cooling-module-replacement/IC740991.png)
   
     **Figure 1** : Panneau avant de l’appareil  
   
   | Étiquette | Description |
   |:--- |:--- |
   | 1 |Bouton Muet |
   | 2 |Alimentation du système |
   | 3 |Panne de module |
   | 4 |Erreur logique |
   | 5 |Affichage de l’ID de l’unité |
3. Les voyants LED de surveillance à l’arrière du boîtier principal peuvent également être utilisés pour identifier le PCM défectueux. Consultez le diagramme et le tableau suivants pour comprendre comment utiliser les LED pour localiser le PCM défectueux. Par exemple, si la LED correspondant à **Ventilateur défectueux** est allumée, le ventilateur est défectueux. De même, si la LED correspondant à **Alimentation défectueuse** est allumée, l’alimentation est défectueuse. 
   
    ![Fond de panier des voyants LED de surveillance du PCM de l’appareil](./media/storsimple-power-cooling-module-replacement/IC740992.png)
   
     **Figure 2** : Arrière du PCM avec les voyants LED
   
   | Étiquette | Description |
   |:--- |:--- |
   | 1 |Panne d’alimentation secteur |
   | 2 |Panne de ventilateur |
   | 3 |Panne de batterie |
   | 4 |PCM OK |
   | 5 |Panne d’alimentation secteur |
   | 6 |Batterie saine |
4. Reportez-vous au diagramme suivant de l’arrière de l’appareil StorSimple pour localiser le module PCM défectueux. PCM 0 se trouve à gauche et PCM 1 à droite. Le tableau suivant explique les modules.
   
     ![Fond du panier des modules du boîtier principal de l’appareil](./media/storsimple-power-cooling-module-replacement/IC740994.png)
   
     **Figure 3** : Arrière de l’appareil avec des modules enfichables 
   
   | Étiquette | Description |
   |:--- |:--- |
   | 1 |PCM 0 |
   | 2 |PCM 1 |
   | 3 |Contrôleur 0 |
   | 4 |Contrôleur 1 |
5. Éteignez le PCM défectueux et déconnectez le cordon d’alimentation. Vous pouvez maintenant retirer le PCM.
6. Saisissez le verrou et le côté de la poignée du PCM entre le pouce et l’index, et serrez-les l’un contre l’autre pour ouvrir la poignée.
   
    ![Ouverture de la poignée du PCM](./media/storsimple-power-cooling-module-replacement/IC740995.png)
   
    **Figure 4** : Ouverture de la poignée du PCM
7. Saisissez la poignée et retirez le PCM.
   
    ![Retrait du PCM de l’appareil](./media/storsimple-power-cooling-module-replacement/IC740996.png)
   
    **Figure 5** : Retrait du PCM

## <a name="install-a-replacement-pcm"></a>Installer un PCM de remplacement
Suivez ces instructions pour installer un PCM dans votre appareil StorSimple. Assurez-vous que vous avez inséré le module de batterie de secours avant d’installer le module PCM de remplacement (s’applique aux modules PCM 764 W uniquement). Pour plus d’informations, consultez la procédure à suivre pour [retirer et insérer un module de batterie de secours](storsimple-8000-battery-replacement.md).

#### <a name="to-install-a-pcm"></a>Pour installer un PCM
1. Vérifiez que vous disposez du PCM de remplacement correct pour ce boîtier. Le boîtier principal nécessite un PCM de 764 W et le boîtier EBOD un PCM de 580 W. Vous ne devez pas essayer d’utiliser le PCM de 580 W dans le boîtier principal, ni le PCM de 764 W dans le boîtier EBOD. L’illustration suivante montre où identifier ces informations sur l’étiquette apposée sur le PCM.
   
    ![Étiquette du PCM de l’appareil](./media/storsimple-power-cooling-module-replacement/IC740973.png)
   
    **Figure 6** : Étiquette du PCM
2. Vérifiez si le boîtier n’est pas endommagé, en faisant particulièrement attention aux connecteurs. 
   
   > [!NOTE]
   > **N’installez pas le module si une ou plusieurs broches des connecteurs sont tordues.**
   > 
   > 
3. Avec la poignée du PCM en position ouverte, faites glisser le module dans le boîtier.
   
    ![Installation du PCM de l’appareil](./media/storsimple-power-cooling-module-replacement/IC740975.png)
   
    **Figure 7** : Installation du PCM
4. Fermez manuellement la poignée du PCM. Vous devez entendre un clic quand le verrou de la poignée s’engage.
   
   > [!NOTE]
   > Pour vous assurer que les broches des connecteurs sont bien engagées, vous pouvez tirer doucement sur la poignée sans libérer le verrou. Si le PCM ressort, cela signifie que le verrou a été fermé avant que les connecteurs soient engagés.
   
5. Connectez les câbles d’alimentation à la source de courant électrique et au PCM.
6. Resserrez les colliers réducteurs de tension.
7. Allumez le PCM.
8. Vérifiez que le remplacement a réussi : dans le portail Azure de votre service StorSimple Device Manager, accédez à votre appareil, puis à **Paramètres > Surveiller > Intégrité matérielle**. Sous **Composants partagés**, l’état du PCM doit être en vert.
   
   > [!NOTE]
   > La réinitialisation complète du PCM de remplacement peut prendre quelques minutes.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur le [remplacement des composants matériels StorSimple](storsimple-8000-hardware-component-replacement.md).

