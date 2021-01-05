---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 48ce594fa5f4b736e69b5b4ef7a10011fe8031fa
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96026341"
---
#### <a name="to-install-updates-via-the-azure-portal"></a>Pour installer des mises à jour par le biais du portail Azure

1. Accédez à votre gestionnaire de périphériques StorSimple et sélectionnez **Périphériques**. Dans la liste des périphériques connectés à votre service, sélectionnez et cliquez sur le périphérique que vous souhaitez mettre à jour. 

    ![Dans Récent, le Gestionnaire d’appareils MySSDevManager est mis en surbrillance et sélectionné, Appareils est mis en surbrillance et sélectionné, et l’appareil MYSSIS1103 est mis en surbrillance et sélectionné.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate1m.png) 

2. Dans le panneau **Paramètres**, cliquez sur **Mises à jour de l’appareil**. 

    ![Les informations relatives à MYSSIS1103 sont affichées. Dans Paramètres, Mises à jour de l’appareil est mis en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate2m.png)  

3. Un message s’affiche si des mises à jour logicielles sont disponibles. Pour rechercher des mises à jour, vous pouvez également cliquer sur **Analyser**.

    ![Le volet Mises à jour de l’appareil indique que « De nouvelles mises à jour sont disponibles » et précise « Dernière analyse / 18/01/2017 2:42 PM / Version du logiciel / 10.0.10288.0 ». La version est mise en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate3m1.png)

    Un message s’affichera au début et à la fin de l’analyse.

    ![La notification indique « Analyse des mises à jour pour l’appareil MySSIS1103. 2:12 PM / L’opération s’est terminée correctement. »](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate5m.png)

4. Une fois les mises à jour analysées, cliquez sur **Télécharger les mises à jour**. 

    ![Dans le volet Mises à jour de l’appareil, un message indique que « De nouvelles mises à jour sont disponibles ». Le bouton Télécharger les mises à jour est mis en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate6m.png)

5. Dans le panneau **Nouvelles mises à jour**, un message indique que lorsque les mises à jour seront téléchargées, vous devrez confirmer l’installation. Cliquez sur **OK**.

    ![Le volet Nouvelles mises à jour indique « Une fois les mises à jour téléchargées, vous devez confirmer l’installation. » Le bouton OK est mis en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate7m.png)

6. Un message s’affichera au début et à la fin du téléchargement.

     ![La notification indique « Téléchargement des mises à jour pour l’appareil MySSIS1… 2:13 PM ».](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate8m.png)

5. Dans le panneau **Mises à jour de l’appareil**, cliquez sur **Installer**.

     ![Dans Mises à jour de l’appareil, un message indique de « Confirmer l’installation des mises à jour ». Un bouton Installer est disponible.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate11m1.png)   

6. Dans le panneau **Nouvelles mises à jour**, vous êtes averti que la mise à jour peut perturber le fonctionnement de l’appareil. Dans la mesure où Virtual Array est un appareil à nœud unique, il redémarre après la mise à jour. Cela perturbe les éventuelles E/S en cours. Cliquez sur **OK** pour installer les mises à jour. 

    ![Le message dans le volet Nouvelles mises à jour indique « Votre appareil subira un temps d’arrêt lors de l’installation de ces mises à jour ». Le bouton OK est mis en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate12m.png) 

7. Vous êtes averti lorsque le travail d’installation démarre. 

    ![La notification indique « Installation des mises à jour pour l’appareil MySSIS1103. 2:15 PM ».](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate13m.png)

8.  Une fois l’installation terminée, cliquez sur le lien **Afficher le travail** dans le panneau **Mises à jour de l’appareil** pour suivre l’installation. 

    ![Le volet Mises à jour de l’appareil contient un lien intitulé « Installation des mises à jour. Afficher le travail. »](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate15m1.png)

    Vous accéderez au panneau **Installer les mises à jour**. Des informations détaillées sur le travail y sont indiquées.

    ![Le volet Installer les mises à jour contient des informations sur l’installation, notamment l’appareil, l’état, la durée, l’heure de début et l’heure d’arrêt.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate16m1.png)

9. Une fois les mises à jour correctement installées, vous recevez un message vous l’indiquant dans le panneau **Mises à jour de l’appareil**. 
