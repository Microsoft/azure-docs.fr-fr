---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 3129bbe171329ecf37f8712394cedf9b70188220
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "96005812"
---
#### <a name="to-install-updates-via-the-azure-portal"></a>Pour installer des mises à jour par le biais du portail Azure

1. Accédez à votre gestionnaire de périphériques StorSimple et sélectionnez **Périphériques**. Dans la liste des périphériques connectés à votre service, sélectionnez et cliquez sur le périphérique que vous souhaitez mettre à jour.

    ![Dans Récent, le Gestionnaire de périphériques MySSDevManager est mis en surbrillance et sélectionné, Appareils est mis en surbrillance et sélectionné, et l’appareil MYSSIS1103 est mis en surbrillance et sélectionné.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate1m.png) 

2. Dans le panneau **Paramètres**, cliquez sur **Mises à jour de l’appareil**.

    ![Les informations relatives à MYSSIS1103 sont affichées. Dans Paramètres, Mises à jour de l’appareil est mis en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal-04/azupdate2m.png)  

3. Un message s’affiche si des mises à jour logicielles sont disponibles. Pour rechercher des mises à jour, vous pouvez également cliquer sur **Analyser**. Prenez note de la version du logiciel que vous exécutez. 

    ![Le volet Mises à jour de l’appareil indique que « De nouvelles mises à jour sont disponibles » et précise « Dernière analyse / 01/11/2017 10:56 / Version du logiciel / 10.0.10289.0 ». La version est mise en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal-1/azupdate3m1.png)

    Un message s’affiche au début et à la fin de l’analyse.

    ![La notification indique « Analyse des mises à jour pour l’appareil MySVAFS110... 10:57 / L'opération est en cours. »](../includes/media/storsimple-virtual-array-install-update-via-portal-1/azupdate5m.png)

4. Une fois les mises à jour analysées, cliquez sur **Télécharger les mises à jour**.

    ![Dans le volet Mises à jour de l’appareil, un message indique que « De nouvelles mises à jour sont disponibles ». Le bouton Télécharger les mises à jour est mis en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal-1/azupdate6m.png)

5. Dans le panneau **Nouvelles mises à jour**, passez en revue les notes de publication. Notez également qu’une fois que les mises à jour sont téléchargées, vous devez confirmer l’installation. Cliquez sur **OK**.

    ![Le volet Nouvelles mises à jour indique « Une fois les mises à jour téléchargées, vous devez confirmer l'installation. » Le bouton OK est mis en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal-1/azupdate7m.png)

6. Un message s’affichera au début et à la fin du téléchargement.

     ![La notification indique « Téléchargement des mises à jour pour l’appareil MySVAFS110... 11:07 ».](../includes/media/storsimple-virtual-array-install-update-via-portal-1/azupdate8m.png)

5. Dans le panneau **Mises à jour de l’appareil**, cliquez sur **Installer**.

     ![Dans Mises à jour de l’appareil, un message indique de « Confirmer l’installation des mises à jour ». Le bouton Installer est mis en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal-1/azupdate11m1.png)

6. Dans le panneau **Nouvelles mises à jour**, vous êtes averti que la mise à jour peut perturber le fonctionnement de l’appareil. Dans la mesure où Virtual Array est un appareil à nœud unique, il redémarre après la mise à jour. Cela perturbe les éventuelles E/S en cours. Cliquez sur **OK** pour installer les mises à jour.

    ![Le message dans le volet indique « Votre appareil subira un temps d’arrêt lors de l’installation de ces mises à jour ». Le bouton OK est mis en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal-1/azupdate12m.png)

7. Vous êtes averti lorsque le travail d’installation démarre.

    ![La notification indique « Installation des mises à jour pour l’appareil MySVAFS110... 11:09 ».](../includes/media/storsimple-virtual-array-install-update-via-portal-1/azupdate13m.png)

8.  Une fois l’installation terminée, cliquez sur le lien **Afficher le travail** dans le panneau **Mises à jour de l’appareil** pour suivre l’installation. 

    ![Le volet Mises à jour de l’appareil contient un lien intitulé « Installation des mises à jour. Afficher le travail. » Le bouton Installer est mis en surbrillance.](../includes/media/storsimple-virtual-array-install-update-via-portal-1/azupdate15m1.png)

    Vous accédez alors au panneau **Installer les mises à jour**. Des informations détaillées sur le travail y sont indiquées.

    ![Le volet Installer les mises à jour contient des informations sur l’installation, notamment l’appareil, l’état, la durée, l’heure de début et l’heure d’arrêt.](../includes/media/storsimple-virtual-array-install-update-via-portal-1/azupdate16m1.png)

9. Si vous avez démarré avec un tableau virtuel qui exécutait la version du logiciel Update 0.6 (10.0.10293.0), vous exécutez à présent Update 1 et vous avez terminé. Vous pouvez donc ignorer les étapes restantes. Si vous avez démarré avec un tableau virtuel qui exécutait une version du logiciel antérieure à Update 0.6 (10.0.10293.0), la mise à jour vers Update 0.6 est maintenant effectuée. Un autre message vous indique que des mises à jour sont disponibles. Répétez les étapes 4 à 8 pour installer Update 1.

    ![Le volet Mises à jour de l’appareil indique que « De nouvelles mises à jour sont disponibles » et précise « Dernière analyse / 1/11/2017 10:56 / Version du logiciel / 10.0.10293.0 ».](../includes/media/storsimple-virtual-array-install-update-via-portal-1/azupdate17.png)

