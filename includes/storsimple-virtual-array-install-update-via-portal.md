---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: df572b88cf8a67163b28ec87154dcea01646b9a2
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95558898"
---
#### <a name="to-install-updates-via-the-azure-portal"></a>Pour installer des mises à jour par le biais du portail Azure

1. Accédez à votre gestionnaire de périphériques StorSimple et sélectionnez **Périphériques**. Dans la liste des périphériques connectés à votre service, sélectionnez et cliquez sur le périphérique que vous souhaitez mettre à jour. 

    ![Dans Récent, le Gestionnaire d’appareils MySSDevManager est mis en surbrillance et sélectionné, Appareils est mis en surbrillance et sélectionné, et l’appareil MYSSIS1103 est mis en surbrillance et sélectionné.](../includes/media/storsimple-virtual-array-install-update-via-portal/azupdate1m.png) 

2. Dans le panneau **Paramètres**, cliquez sur **Mises à jour de l’appareil**. 

    ![Les informations relatives à MYSSIS1103 s’affichent. Dans Paramètres, l’option Mises à jour de l’appareil est mise en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal/azupdate2m.png)  

3. Un message s’affiche si des mises à jour logicielles sont disponibles. Pour rechercher des mises à jour, vous pouvez également cliquer sur **Analyser**.

    ![Dans le volet Mises à jour de l’appareil, le bouton Analyse est mis en surbrillance. Il y a un message à quatre lignes : Dernière analyse/Non analysé/Version du logiciel/10.0.10280.0.](../includes/media/storsimple-virtual-array-install-update-via-portal/azupdate3m.png)

    Un message s’affichera au début et à la fin de l’analyse.

    ![La notification indique « Analyse des mises à jour pour l’appareil MySSIS1103.2:12 PM/Opération terminée avec succès ».](../includes/media/storsimple-virtual-array-install-update-via-portal/azupdate5m.png)

4. Une fois les mises à jour analysées, cliquez sur **Télécharger les mises à jour**. 

    ![Le volet Mises à jour de l’appareil indique que « De nouvelles mises à jour sont disponibles » et précise « Dernière analyse / 11/3/2016 2:12 PM / Version du logiciel / 10.0.10280.0 ». La version est mise en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal/azupdate6m.png)

5. Dans le panneau **Nouvelles mises à jour**, un message indique que lorsque les mises à jour seront téléchargées, vous devrez confirmer l’installation. Cliquez sur **OK**.

    ![Le volet Nouvelles mises à jour indique « Une fois les mises à jour téléchargées, vous devez confirmer l’installation. » Le bouton OK est mis en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal/azupdate7m.png)

6. Un message s’affichera au début et à la fin du téléchargement.

     ![La notification indique « Téléchargement des mises à jour pour l’appareil MySSIS… 2:13 PM ».](../includes/media/storsimple-virtual-array-install-update-via-portal/azupdate8m.png)

5. Dans le panneau **Mises à jour de l’appareil**, cliquez sur **Installer**.

     ![Le volet Mises à jour de l’appareil affiche le message « Confirmer l’installation des mises à jour ». Le bouton Installer est mis en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal/azupdate11m.png)   

6. Dans le panneau **Nouvelles mises à jour**, vous êtes averti que la mise à jour peut perturber le fonctionnement de l’appareil. Dans la mesure où Virtual Array est un appareil à nœud unique, il redémarre après la mise à jour. Cela perturbe les éventuelles E/S en cours. Cliquez sur **OK** pour installer les mises à jour. 

    ![Le message dans le volet Nouvelles mises à jour indique « Votre appareil subira un temps d’arrêt lors de l’installation de ces mises à jour ». Le bouton OK est mis en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal/azupdate12m.png) 

7. Vous êtes averti lorsque le travail d’installation démarre. 

    ![La notification indique « Installation des mises à jour pour l’appareil MYSSIS1103 ». 2:15 PM.](../includes/media/storsimple-virtual-array-install-update-via-portal/azupdate13m.png)

8.  Une fois l’installation terminée, cliquez sur le lien **Afficher le travail** dans le panneau **Mises à jour de l’appareil** pour suivre l’installation. 

    ![Dans le volet Mises à jour de l’appareil, le lien intitulé « Installation des mises à jour. Afficher le travail » est mis en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal/azupdate15m.png)

    Vous accéderez au panneau **Installer les mises à jour**. Des informations détaillées sur le travail y sont indiquées.

    ![Le volet Installer les mises à jour contient des informations sur l’installation, notamment l’appareil, l’état, la durée, l’heure de début et l’heure d’arrêt.](../includes/media/storsimple-virtual-array-install-update-via-portal/azupdate16m.png)

9. Une fois les mises à jour correctement installées, vous voyez un message vous l’indiquant dans le panneau Mises à jour de l’appareil. La version du logiciel devient également **10.0.10288.0**. 

    ![Le volet Mises à jour de l’appareil affiche « Votre appareil est à jour » et fournit la version du logiciel (10.0.10288.0).](../includes/media/storsimple-virtual-array-install-update-via-portal/azupdate17m.png)
