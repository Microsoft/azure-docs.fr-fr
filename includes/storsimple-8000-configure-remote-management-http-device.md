---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 473bc0a58fe49c7f454c81402b57ddce7fc745b2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177124"
---
#### <a name="to-configure-remote-management-on-cloud-appliance"></a>Pour configurer la gestion à distance sur l’appliance cloud

1. Dans le service StorSimple Device Manager, cliquez sur **Appareils**. Sélectionnez et cliquez sur votre appliance cloud dans la liste des appareils connectés au service.
    ![StorSimple - sélectionner une appliance cloud](./media/storsimple-8000-configure-remote-management-http-device/sca-remote-manage1.png)

2. Accédez à **Paramètres > sécurité** pour ouvrir le panneau **Paramètres de sécurité**.

     ![Paramètres de sécurité StorSimple](./media/storsimple-8000-configure-remote-management-http-device/sca-remote-manage2.png)

3. Accédez à la section **Gestion à distance**. Cochez la case **Gestion à distance**.
     ![Gestion à distance de StorSimple](./media/storsimple-8000-configure-remote-management-http-device/sca-remote-manage3.png)

4. Dans le panneau **Gestion à distance** :

    1. Vérifiez que la case **Activer l’administration à distance** est cochée.
    2. La valeur par défaut consiste à se connecter via HTTPS. Vous pouvez choisir de vous connecter à l’aide de HTTP. Une connexion via HTTP est acceptable uniquement sur des réseaux approuvés. Vérifiez que HTTP est activé.
    3. Dans la barre de commandes en haut du panneau, cliquez sur **... Plus**, puis cliquez sur **Télécharger le certificat** pour télécharger un certificat de gestion à distance. Vous pouvez spécifier un emplacement dans lequel enregistrer ce fichier. Ce certificat doit être installé sur la machine cliente ou hôte que vous utilisez pour vous connecter à l’appliance cloud.

        ![Panneau de gestion à distance](./media/storsimple-8000-configure-remote-management-http-device/sca-remote-manage4.png)
5. Cliquez sur **Enregistrer** et lorsque vous y êtes invité, confirmez les modifications.