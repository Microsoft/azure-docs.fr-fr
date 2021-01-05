---
title: Remplacer un contrôleur sur un appareil de la gamme StorSimple 8000 | Microsoft Docs
description: Explique comment retirer et remplacer un module de contrôleur, ou les deux, sur votre appareil de la gamme StorSimple 8000.
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
ms.date: 06/05/2017
ms.author: alkohli
ms.openlocfilehash: 9d8b75c48da2bb13d843258ead378d3e849da951
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85514074"
---
# <a name="replace-a-controller-module-on-your-storsimple-device"></a>Remplacement d’un module de contrôleur sur votre appareil StorSimple
## <a name="overview"></a>Vue d’ensemble
Ce didacticiel explique comment retirer et remplacer un module de contrôleur, ou les deux, dans un appareil StorSimple. Il aborde également la logique sous-jacente pour les scénarios de remplacement d’un seul et de deux contrôleurs.

> [!NOTE]
> Avant d’effectuer un remplacement de contrôleur, nous vous recommandons de toujours mettre à jour le microprogramme de votre contrôleur vers la dernière version.
> 
> Pour éviter d’endommager votre appareil StorSimple, n’éjectez pas le contrôleur tant que les voyants LED ne se présentent pas dans l’une des configurations suivantes :
> 
> * Tous les témoins lumineux sont éteints.
> * Le voyant LED 3, ![icône en forme de coche verte](./media/storsimple-controller-replacement/HCS_GreenCheckIcon.png), et ![icône en forme de croix rouge](./media/storsimple-controller-replacement/HCS_RedCrossIcon.png) clignotent, et les voyants LED 0 et 7 sont **allumés**.


Le tableau suivant indique les scénarios de remplacement de contrôleurs pris en charge.

| Cas | Scénario de remplacement | Procédure applicable |
|:--- |:--- |:--- |
| 1 |Un seul contrôleur est en panne, l’autre est intègre et actif. |[Remplacement d’un seul contrôleur](#replace-a-single-controller), qui décrit la [logique sous-jacente au remplacement d’un seul contrôleur](#single-controller-replacement-logic), ainsi que la [procédure de remplacement](#single-controller-replacement-steps). |
| 2 |Les deux contrôleurs sont en panne et doivent être remplacés. Le châssis, les disques et leur boîtier sont intègres. |[Remplacement des deux contrôleurs](#replace-both-controllers), qui décrit la [logique sous-jacente au remplacement des deux contrôleurs](#dual-controller-replacement-logic), ainsi que la [procédure de remplacement](#dual-controller-replacement-steps). |
| 3 |Des contrôleurs sont intervertis dans le même appareil ou dans différents appareils. Le châssis, les disques et leur boîtier sont intègres. |Un message d’alerte s’affiche pour signaler la mauvaise correspondance d’un emplacement. |
| 4 |Il manque un contrôleur et l’autre contrôleur est en panne. |[Remplacement des deux contrôleurs](#replace-both-controllers), qui décrit la [logique sous-jacente au remplacement des deux contrôleurs](#dual-controller-replacement-logic), ainsi que la [procédure de remplacement](#dual-controller-replacement-steps). |
| 5 |Un seul contrôleur ou les deux sont en panne. Vous ne pouvez pas accéder à l’appareil via la console série ni la communication à distance Windows PowerShell. |[contacter le support Microsoft](storsimple-8000-contact-microsoft-support.md) pour suivre une procédure de remplacement de contrôleur manuelle. |
| 6 |La version de build des contrôleurs est différente. Deux causes sont possible :<ul><li>Les versions de logiciel des contrôleurs sont différentes.</li><li>Les versions de microprogramme des contrôleurs sont différentes.</li></ul> |Si les versions de logiciel des contrôleurs sont différentes, la logique de remplacement le détecte et met à jour la version du logiciel sur le contrôleur de remplacement.<br><br>Si les versions de microprogramme des contrôleurs sont différentes et l’ancienne version ne peut **pas** être mise à niveau automatiquement, un message d’alerte s’affiche dans le portail Azure. Vous devez rechercher et installer les mises à jour du microprogramme.</br></br>Si les versions de microprogramme des contrôleurs sont différentes et que l’ancienne version peut être mise à niveau automatiquement, la logique de remplacement du contrôleur le détecte et, une fois le contrôleur démarré, le microprogramme est automatiquement mis à jour. |

Vous devez retirer un module de contrôleur s’il est tombé en panne. Un seul contrôleur ou les deux peuvent tomber en panne, ce qui entraîne le remplacement d’un seul ou de deux contrôleurs. Pour les procédures de remplacement et la logique sous-jacente, voir les rubriques suivantes :

* [Remplacer un seul contrôleur](#replace-a-single-controller)
* [Remplacer les deux contrôleurs](#replace-both-controllers)
* [Retirer un contrôleur](#remove-a-controller)
* [Insérer un contrôleur](#insert-a-controller)
* [Identifier le contrôleur actif sur votre appareil](#identify-the-active-controller-on-your-device)

> [!IMPORTANT]
> Avant le retrait et le remplacement d’un contrôleur, voir les informations de sécurité dans [Remplacement des composants matériels StorSimple](storsimple-8000-hardware-component-replacement.md).
> 
> 

## <a name="replace-a-single-controller"></a>Remplacer un seul contrôleur
Quand l’un des deux contrôleurs de l’appareil Microsoft Azure StorSimple tombe en panne, ne fonctionne pas correctement ou est absent, vous devez le remplacer.

### <a name="single-controller-replacement-logic"></a>Logique de remplacement d’un seul contrôleur
Pour remplacer un seul contrôleur, vous devez d’abord retirer le contrôleur défectueux. (Le contrôleur restant dans l’appareil devient le contrôleur actif.) Quand le contrôleur de remplacement est inséré, les actions suivantes se produisent :

1. Le contrôleur de remplacement commence immédiatement à communiquer avec l’appareil StorSimple.
2. Un instantané du disque dur virtuel (VHD) du contrôleur actif est copié sur le contrôleur de remplacement.
3. L’instantané est modifié pour que le contrôleur de remplacement soit reconnu en tant que contrôleur de secours quand il démarre à partir de ce disque dur virtuel (VHD).
4. Quand les modifications sont terminées, le contrôleur de remplacement démarre en tant que contrôleur de secours.
5. Quand les deux contrôleurs sont en cours d’exécution, le cluster est mis en ligne.

### <a name="single-controller-replacement-steps"></a>Procédure de remplacement d’un seul contrôleur
En cas d’échec de l’un des contrôleurs de votre appareil Microsoft Azure StorSimple, procédez comme suit. (L’autre contrôleur doit être actif et opérationnel. Si les deux contrôleurs sont en panne ou fonctionnent mal, voir [Procédure de remplacement des deux contrôleurs](#dual-controller-replacement-steps).)

> [!NOTE]
> Entre 30 et 45 minutes peuvent être nécessaires au redémarrage et à la récupération complète du contrôleur à l’issue de la procédure de son remplacement. La durée totale de la procédure (branchement des câbles compris) est approximativement de 2 heures.


#### <a name="to-remove-a-single-failed-controller-module"></a>Pour retirer un seul module de contrôleur défectueux
1. Dans le portail Azure, accédez au service StorSimple Device Manager, cliquez sur **Appareils**, puis sur le nom de l’appareil à surveiller.
2. Accédez à **Surveiller > Hardware health** (Intégrité matérielle). L’état de Contrôleur 0 ou Contrôleur 1 doit être rouge, ce qui indique une défaillance.
   
   > [!NOTE]
   > Le contrôleur défectueux est toujours un contrôleur de secours dans le cadre du remplacement d’un seul contrôleur.
   
3. Utilisez la figure 1 et le tableau ci-après pour repérer le module de contrôleur défectueux.
   
    ![Fond du panier des modules du boîtier principal de l’appareil](./media/storsimple-controller-replacement/IC740994.png)
   
    **Figure 1** Arrière de l’appareil StorSimple
   
   | Étiquette | Description |
   |:--- |:--- |
   | 1 |PCM 0 |
   | 2 |PCM 1 |
   | 3 |Contrôleur 0 |
   | 4 |Contrôleur 1 |
4. Retirez tous les câbles réseau connectés aux ports de données sur le contrôleur défectueux. Si vous utilisez un modèle 8600, débranchez également les câbles SAS reliant le contrôleur au contrôleur EBOD.
5. Suivez les étapes de la procédure [Retrait d’un contrôleur](#remove-a-controller) pour retirer le contrôleur défectueux.
6. Installez le contrôleur de remplacement dans le même emplacement que celui duquel vous avez retiré le contrôleur défectueux. La logique de remplacement d’un seul contrôleur est ainsi déclenchée. Pour plus d’informations, voir [Logique de remplacement d’un seul contrôleur](#single-controller-replacement-logic).
7. Pendant que la logique de remplacement d’un seul contrôleur se déroule en arrière-plan, rebranchez les câbles. Veillez à rebrancher tous les câbles exactement comme ils l’étaient avant le remplacement.
8. Une fois que le contrôleur a redémarré, vérifiez l’**état du contrôleur** et l’**état du cluster** dans le portail Azure pour vous assurer que le contrôleur est à nouveau intègre et en mode veille.

> [!NOTE]
> Si vous utilisez la console série pour surveiller l’appareil, il est possible que vous assistiez à plusieurs redémarrages pendant la récupération du contrôleur suite à la procédure de remplacement. Vous savez que la procédure de remplacement est terminée quand le menu de la console série s’affiche. Si ce menu ne s’affiche pas dans un délai de deux heures après le début du remplacement du contrôleur, veuillez [contacter le support Microsoft](storsimple-8000-contact-microsoft-support.md).
>
> À partir d’Update 4, vous pouvez également utiliser l’applet de commande `Get-HCSControllerReplacementStatus` dans l’interface Windows PowerShell de l’appareil pour surveiller l’état du processus de remplacement du contrôleur.
> 

## <a name="replace-both-controllers"></a>Remplacer les deux contrôleurs
Quand les deux contrôleurs de l’appareil Microsoft Azure StorSimple tombent en panne, ne fonctionnent pas correctement ou sont absents, vous devez les remplacez tous les deux. 

### <a name="dual-controller-replacement-logic"></a>Logique de remplacement de deux contrôleurs
Dans un remplacement de deux contrôleurs, vous devez d’abord retirer les deux contrôleurs en panne, puis insérer les contrôleurs de rechange. Quand les deux contrôleurs de remplacement sont insérés, les actions suivantes se produisent :

1. Le contrôleur de remplacement inséré dans l’emplacement 0 vérifie ce qui suit :
   
   1. Utilise-t-il les versions actuelles du microprogramme et du logiciel ?
   2. Fait-il partie du cluster ?
   3. Le contrôleur homologue est-il en cours d’exécution et en cluster ?
      
      Si aucune de ces conditions n’est remplie, le contrôleur recherche la dernière sauvegarde quotidienne (située dans le répertoire **nonDOMstorage** sur le lecteur S). Le contrôleur copie le dernier instantané du VHD à partir de la sauvegarde.
2. Le contrôleur inséré dans l’emplacement 0 utilise l’instantané pour créer une image de lui-même.
3. Pendant ce temps, le contrôleur inséré dans l’emplacement 1 attend que le contrôleur 0 termine la création de l’image et démarre.
4. Une fois que le contrôleur 0 démarre, le contrôleur 1 détecte le cluster créé par le contrôleur 0, ce qui déclenche la logique de remplacement d’un seul contrôleur. Pour plus d’informations, voir [Logique de remplacement d’un seul contrôleur](#single-controller-replacement-logic).
5. Ensuite, les deux contrôleurs s’exécutent et le cluster est mis en ligne.

> [!IMPORTANT]
> À l’issue d’un remplacement de deux contrôleurs, une fois l’appareil StorSimple configuré, il est primordial d’effectuer une sauvegarde manuelle de celui-ci. Les sauvegardes quotidiennes de la configuration de l’appareil se déclenchent uniquement après un délai de 24 heures. Pour effectuer une sauvegarde manuelle de votre appareil, collaborez avec le [support Microsoft](storsimple-8000-contact-microsoft-support.md) .


### <a name="dual-controller-replacement-steps"></a>Procédure de remplacement des deux contrôleurs
Ce flux de travail doit être suivi quand les deux contrôleurs de votre appareil Microsoft Azure StorSimple sont en panne. Cette situation peut se produire dans un centre de données dont le système de refroidissement cesse de fonctionner, entraînant rapidement une panne des deux contrôleurs. La procédure à suivre diffère selon que l’appareil StorSimple est sous ou hors tension, et selon le modèle utilisé (8600 ou 8100).

> [!IMPORTANT]
> Entre 45 minutes et 1 heure peuvent être nécessaires au redémarrage et à la récupération complète du contrôleur à l’issue de la procédure de remplacement des deux contrôleurs. La durée totale de la procédure (branchement des câbles compris) est approximativement de 2,5 heures.

#### <a name="to-replace-both-controller-modules"></a>Pour remplacer les deux modules de contrôleur
1. Si l’appareil est éteint, ignorez cette étape et passez à la suivante. Si l’appareil est allumé, éteignez-le.
   
   1. Si vous utilisez un modèle 8600, éteignez d’abord le boîtier principal, puis le boîtier EBOD.
   2. Patientez jusqu’à l’arrêt complet de l’appareil. Tous les voyants LED à l’arrière de l’appareil doivent être éteints.
2. Retirez tous les câbles réseau connectés aux ports de données. Si vous utilisez un modèle 8600, débranchez également les câbles SAS reliant le boîtier principal au boîtier EBOD.
3. Retirez les deux contrôleurs de l’appareil StorSimple. Pour plus d’informations, consultez [Retrait d’un contrôleur](#remove-a-controller).
4. Insérez d’abord le modèle de remplacement du contrôleur 0, puis celui du contrôleur 1. Pour plus d’informations, voir [Insertion d’un contrôleur](#insert-a-controller). La logique de remplacement des deux contrôleurs est ainsi déclenchée. Pour plus d’informations, voir [Logique de remplacement de deux contrôleurs](#dual-controller-replacement-logic).
5. Pendant que la logique de remplacement des deux contrôleurs se déroule en arrière-plan, rebranchez les câbles. Veillez à rebrancher tous les câbles exactement comme ils l’étaient avant le remplacement. Voir les instructions détaillées relatives à votre modèle dans la section sur le branchement des câbles de votre appareil de la rubrique [Installation de votre appareil StorSimple 8100](storsimple-8100-hardware-installation.md) ou [Installation de votre appareil StorSimple 8600](storsimple-8600-hardware-installation.md).
6. Mettez sous tension l’appareil StorSimple. Si vous utilisez un modèle 8600 :
   
   1. Veillez à activer le boîtier EBOD en premier.
   2. Attendez que le boîtier EBOD soit opérationnel.
   3. Mettez sous tension le boîtier principal.
   4. Une fois que le premier contrôleur redémarre et que son intégrité est vérifiée, le système redémarre.
      
      > [!NOTE]
      > Si vous utilisez la console série pour surveiller l’appareil, il est possible que vous assistiez à plusieurs redémarrages pendant la récupération du contrôleur suite à la procédure de remplacement. Vous savez que la procédure de remplacement est terminée quand le menu de la console série s’affiche. Si ce menu ne s’affiche pas dans un délai de 2,5 heures après le début du remplacement du contrôleur, veuillez [contacter le support Microsoft](storsimple-8000-contact-microsoft-support.md).
     
## <a name="remove-a-controller"></a>Retirer un contrôleur
Utilisez les procédures suivantes pour retirer un module de contrôleur défectueux de votre appareil StorSimple.

> [!NOTE]
> Les illustrations suivantes concernent le contrôleur 0. Pour le contrôleur 1, leur présentation doit être inversée.


#### <a name="to-remove-a-controller-module"></a>Pour retirer un module de contrôleur
1. Saisissez le verrou du module entre le pouce et l’index.
2. Pressez doucement le pouce et l’index pour libérer le verrou du contrôleur.
   
    ![Libération du verrou du contrôleur](./media/storsimple-controller-replacement/IC741047.png)
   
    **Figure 2** Libération du verrou du contrôleur
3. Utilisez le verrou comme une poignée pour faire glisser le contrôleur hors du châssis.
   
    ![Glissement du contrôleur hors du châssis](./media/storsimple-controller-replacement/IC741048.png)
   
    **Figure 3** Extraction du contrôleur hors du châssis

## <a name="insert-a-controller"></a>Insérer un contrôleur
Utilisez la procédure suivante pour installer un module de contrôleur d’usine après avoir retiré un module défectueux de votre appareil StorSimple.

#### <a name="to-install-a-controller-module"></a>Pour installer un module de contrôleur
1. Vérifiez que les connecteurs d’interface sont en bon état. N’installez pas le module si des broches sont endommagées ou tordues dans les connecteurs.
2. Faites glisser le module de contrôleur dans le châssis avec le verrou entièrement libéré.
   
    ![Insertion du contrôleur dans le châssis](./media/storsimple-controller-replacement/IC741053.png)
   
    **Figure 4** Insertion du contrôleur dans le châssis
3. Une fois le module de contrôleur inséré, commencez à fermer le verrou tout en continuant de pousser le module dans le châssis. L’engagement du verrou entraîne la mise en place du contrôleur.
   
    ![Fermeture du verrou du contrôleur](./media/storsimple-controller-replacement/IC741054.png)
   
    **Figure 5** Fermeture du verrou du contrôleur
4. Vous avez terminé quand le verrou s’enclenche. Le voyant LED **OK** doit maintenant être allumé.
   
   > [!NOTE]
   > L’activation du contrôleur et du voyant LED peut prendre jusqu’à 5 minutes.
  
5. Pour vérifier que le remplacement a réussi, dans le portail Azure, accédez à votre appareil, sélectionnez **Surveiller** > **Hardware health** (Intégrité matérielle), puis vérifiez que les contrôleurs 0 et 1 sont signalés comme étant intègres (état en vert).

## <a name="identify-the-active-controller-on-your-device"></a>Identifier le contrôleur actif sur votre appareil
Dans de nombreuses situations, notamment lors de la première inscription de l’appareil ou du premier remplacement du contrôleur, vous devez trouver le contrôleur actif sur un appareil StorSimple. Le contrôleur actif traite toutes les opérations de mise en réseau et de microprogramme du disque. Vous pouvez utiliser les méthodes suivantes pour identifier le contrôleur actif :

* [Utiliser le portail Azure pour identifier le contrôleur actif](#use-the-azure-portal-to-identify-the-active-controller)
* [Utiliser Windows PowerShell pour StorSimple pour identifier le contrôleur actif](#use-windows-powershell-for-storsimple-to-identify-the-active-controller)
* [Vérifier l’appareil physique pour identifier le contrôleur actif](#check-the-physical-device-to-identify-the-active-controller)

Chacune de ces procédures est décrite ci-dessous.

### <a name="use-the-azure-portal-to-identify-the-active-controller"></a>Utiliser le portail Azure pour identifier le contrôleur actif
Dans le portail Azure, accédez à votre appareil, sélectionnez **Surveiller** > **Hardware health** (Intégrité matérielle), puis faites défiler l’écran jusqu’à la section **Contrôleurs**. Dans cette section, vous pouvez vérifier quel contrôleur est actif.

![Identifier le contrôleur actif dans le portail Azure](./media/storsimple-controller-replacement/IC752072.png)

**Figure 6** Portail Azure montrant le contrôleur actif

### <a name="use-windows-powershell-for-storsimple-to-identify-the-active-controller"></a>Utiliser Windows PowerShell pour StorSimple pour identifier le contrôleur actif
Quand vous accédez à votre appareil via la console série, un message de bannière s’affiche. Ce message contient des informations de base sur l’appareil, telles que le modèle, le nom, la version logicielle installée et l’état du contrôleur auquel vous accédez. L’image suivante représente un exemple de message de bannière :

![Message de bannière série](./media/storsimple-controller-replacement/IC741098.png)

**Figure 7** Message de bannière indiquant que le contrôleur 0 est actif

Vous pouvez utiliser le message de bannière pour déterminer si le contrôleur auquel vous êtes connecté est actif ou passif.

### <a name="check-the-physical-device-to-identify-the-active-controller"></a>Vérifier l’appareil physique pour identifier le contrôleur actif
Pour identifier le contrôleur actif sur votre appareil, localisez le voyant LED bleu au-dessus du port DATA 5, à l’arrière du boîtier principal.

Si ce voyant clignote, le contrôleur est actif et l’autre contrôleur est en mode veille. Utilisez le schéma et le tableau suivants pour vous aider.

![Fond de panier du boîtier principal de l’appareil avec ports de données](./media/storsimple-controller-replacement/IC741055.png)

**Figure 8** Arrière du boîtier principal, avec les ports de données et les voyants LED de contrôle

| Étiquette | Description |
|:--- |:--- |
| 1 à 6 |Ports réseau DATA 0 – 5 |
| 7 |Voyant LED bleu |

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur le [remplacement des composants matériels StorSimple](storsimple-8000-hardware-component-replacement.md).

