---
title: Fichier Include
description: Fichier Include
services: storsimple
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 07/18/2018
ms.author: alkohli
ms.custom: include file
ms.openlocfilehash: 8ef8b9d3350d0599ab00dfdbb018cf8dd900e316
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95563954"
---
#### <a name="to-install-updates-via-the-azure-portal"></a>Pour installer des mises à jour par le biais du portail Azure

1. Accédez à votre gestionnaire de périphériques StorSimple et sélectionnez **Périphériques**. Dans la liste des périphériques connectés à votre service, sélectionnez et cliquez sur le périphérique que vous souhaitez mettre à jour.

2. Dans le panneau **Paramètres**, cliquez sur **Mises à jour de l’appareil**.  

3. Un message s’affiche si des mises à jour logicielles sont disponibles. Pour rechercher des mises à jour, vous pouvez également cliquer sur **Analyser**. Prenez note de la version du logiciel que vous exécutez. 

    ![Le volet Mises à jour de l’appareil indique que « De nouvelles mises à jour sont disponibles » (en surbrillance) et précise « Dernière analyse / 22/6/2018 14:46 / Version du logiciel / 10.0.10296.0 ». La version est mise en évidence.](../includes/media/storsimple-virtual-array-install-update-via-portal-11/azupdate3m1.png)

    Un message s’affiche au début et à la fin de l’analyse.
 
4. Une fois les mises à jour analysées, cliquez sur **Télécharger les mises à jour**. Dans le panneau **Nouvelles mises à jour**, passez en revue les notes de publication. Notez également qu’une fois que les mises à jour sont téléchargées, vous devez confirmer l’installation. Cliquez sur **OK**.

    ![Dans Mises à jour de l’appareil, le bouton Télécharger les mises à jour est ressorti. Dans Nouvelles mises à jour, un lien vers les notes de publication est affiché, ainsi qu’un message demandant de confirmer l’installation après le téléchargement des mises à jour. Un bouton OK est disponible.](../includes/media/storsimple-virtual-array-install-update-via-portal-11/azupdate6m.png)

    Un message s’affichera au début et à la fin du téléchargement.

5. Sous **Mises à jour de l’appareil**, cliquez sur **Installer**.

     ![Dans Mises à jour de l’appareil, un message indique de « Confirmer l’installation des mises à jour ». Le bouton Installer et la version du logiciel sont mis en évidence.](../includes/media/storsimple-virtual-array-install-update-via-portal-11/azupdate11m1.png)

6. Dans **Nouvelles mises à jour**, vous êtes averti que la mise à jour peut perturber le fonctionnement de l’appareil. Dans la mesure où Virtual Array est un appareil à nœud unique, il redémarre après la mise à jour. Cela perturbe les éventuelles E/S en cours. Cliquez sur **OK** pour installer les mises à jour.

    ![Dans Nouvelles mises à jour, un message précise que « Votre appareil subira un temps d’arrêt lors de l’installation de ces mises à jour ». Un lien vers les notes de publication et un bouton OK sont affichés.](../includes/media/storsimple-virtual-array-install-update-via-portal-11/azupdate12m.png)

    Vous êtes averti lorsque le travail d’installation démarre.

7.  Une fois l’installation terminée, cliquez sur le lien **Afficher la tâche**. Vous accédez alors au panneau **Installer les mises à jour**. Des informations détaillées sur le travail y sont indiquées. 

    ![Dans Mises à jour de l’appareil, un lien intitulé « Installation des mises à jour. Afficher le travail. » est présent.](../includes/media/storsimple-virtual-array-install-update-via-portal-11/azupdate16m1.png)

8. Si vous avez démarré avec un tableau virtuel qui exécutait la version du logiciel Update 1 (10.0.10296.0), vous exécutez à présent Update 1.1 et vous avez terminé. Vous pouvez donc ignorer les étapes restantes. 

    Si vous avez démarré avec un tableau virtuel qui exécutait une version du logiciel antérieure à Update 0.6 (10.0.10293.0), la mise à jour vers Update 1.0 est maintenant effectuée. Un autre message vous indique que des mises à jour sont disponibles. Répétez les étapes 4 à 7 pour installer Update 1.1.

    

