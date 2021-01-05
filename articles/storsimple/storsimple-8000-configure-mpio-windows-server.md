---
title: Configuration de MPIO pour votre appareil StorSimple | Microsoft Docs
description: Décrit comment configurer Multipath I/O (MPIO) pour votre appareil StorSimple connecté à un hôte exécutant Windows Server 2012 R2.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/26/2018
ms.author: alkohli
ms.openlocfilehash: 3d44fada1eddf2d3f80bec085d8a5bf751197eb1
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968806"
---
# <a name="configure-multipath-io-for-your-storsimple-device"></a>Configuration de Multipath I/O pour votre appareil StorSimple

Ce didacticiel décrit les étapes que vous devez suivre pour installer et utiliser la fonctionnalité MPIO (Multipath I/O) sur un hôte exécutant Windows Server 2012 R2 et connecté à un appareil physique StorSimple. Les instructions de cet article s’appliquent uniquement aux appareils physiques de la gamme StorSimple 8000. La fonctionnalité MPIO n’est actuellement pas prise en charge sur StorSimple Cloud Appliance.

Microsoft a créé la prise en charge de la fonctionnalité MPIO (Multipath I/O) dans Windows Server pour vous aider à créer des configurations réseau iSCSI hautement disponibles et tolérantes aux pannes. MPIO utilise les composants de chemin d’accès physique redondants (adaptateurs, câbles et commutateurs) pour créer des chemins d’accès logiques entre le serveur et l’appareil de stockage. En cas de défaillance de composant à l’origine de l’échec du chemin d’accès logique, la logique de gestion multivoie utilise un autre chemin pour l’E/S afin que les applications puissent toujours accéder à leurs données. De plus, selon votre configuration, MPIO peut également améliorer les performances en rééquilibrant la charge sur tous ces chemins d’accès. Pour plus d’informations, consultez [Vue d’ensemble de MPIO](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc725907(v=ws.11) "Vue d’ensemble et fonctionnalités de MPIO").

Pour la haute disponibilité de votre solution StorSimple, MPIO doit être configuré sur votre appareil StorSimple. Lorsque MPIO est installé sur vos serveurs hôtes exécutant Windows Server 2012 R2, les serveurs peuvent tolérer une défaillance de lien, de réseau ou d’interface.

## <a name="mpio-configuration-summary"></a>Résumé de la configuration MPIO

MPIO est une fonctionnalité facultative sur Windows Server et n’est pas installé par défaut. Il doit être installé en tant que fonctionnalité via le Gestionnaire de serveur.

Suivez ces étapes pour configurer MPIO sur votre appareil StorSimple :

* Étape 1 : installer MPIO sur l'hôte Windows Server
* Étape 2 : configurer MPIO pour les volumes StorSimple
* Étape 3 : monter les volumes StorSimple sur l'hôte
* Étape 4 : configurer MPIO pour la haute disponibilité et l'équilibrage de charge

Chacune des étapes précédentes est abordée dans les sections suivantes.

## <a name="step-1-install-mpio-on-the-windows-server-host"></a>Étape 1 : installer MPIO sur l'hôte Windows Server

Pour installer cette fonctionnalité sur votre hôte Windows Server, effectuez la procédure suivante.

#### <a name="to-install-mpio-on-the-host"></a>Installation de MPIO sur l’hôte

1. Ouvrez le Gestionnaire de serveur sur votre hôte Windows Server. Par défaut, le Gestionnaire de serveur démarre lorsqu’un membre du groupe Administrateurs ouvre une session sur un ordinateur qui exécute Windows Server 2012 R2 ou Windows Server 2012. Si le Gestionnaire de serveur n’est pas encore ouvert, cliquez sur **Démarrer > Gestionnaire de serveur**.
   
   ![Gestionnaire de serveur](./media/storsimple-configure-mpio-windows-server/IC740997.png)

2. Cliquez sur **Gestionnaire de serveur > Tableau de bord > Ajouter des rôles et fonctionnalités**. Cette action démarre l’Assistant **Ajouter des rôles et fonctionnalités** .
   
   ![Assistant Ajouter des rôles et fonctionnalités 1](./media/storsimple-configure-mpio-windows-server/IC740998.png)
3. Dans l’Assistant **Ajouter des rôles et fonctionnalités**, procédez comme suit :
   
   1. Sur la page **Avant de commencer**, cliquez sur **Suivant**.
   2. Sur la page **Sélectionner le type d’installation**, acceptez le paramètre par défaut de l’installation **Installation basée sur un rôle ou une fonctionnalité**. Cliquez sur **Suivant**.
   
       ![Assistant Ajouter des rôles et fonctionnalités 2](./media/storsimple-configure-mpio-windows-server/IC740999.png)
   3. Sur la page **Sélectionner le serveur de destination**, choisissez **Sélectionner un serveur du pool de serveurs**. Votre serveur hôte doit être détecté automatiquement. Cliquez sur **Suivant**.
   4. Sur la page **Sélectionner des rôles de serveurs**, cliquez sur **Suivant**.
   5. Sur la page **Sélectionner des fonctionnalités**, sélectionnez **MPIO (Multipath I/O)** , puis cliquez sur **Suivant**.
   
       ![Assistant Ajouter des rôles et fonctionnalités 5](./media/storsimple-configure-mpio-windows-server/IC741000.png)
   6. Sur la page **Confirmer les sélections pour l’installation**, confirmez la sélection, puis sélectionnez **Redémarrer automatiquement le serveur de destination, si nécessaire**, comme illustré ci-dessous. Cliquez sur **Installer**.
   
       ![Assistant Ajouter des rôles et fonctionnalités 8](./media/storsimple-configure-mpio-windows-server/IC741001.png)
   7. Vous recevez une notification une fois l’installation terminée. Cliquez sur **Fermer** pour fermer l’assistant.
   
       ![Assistant Ajouter des rôles et fonctionnalités 9](./media/storsimple-configure-mpio-windows-server/IC741002.png)

## <a name="step-2-configure-mpio-for-storsimple-volumes"></a>Étape 2 : configurer MPIO pour les volumes StorSimple

La fonctionnalité MPIO doit être configurée afin d’identifier les volumes StorSimple. Pour configurer MPIO pour reconnaître des volumes StorSimple, procédez comme suit.

#### <a name="to-configure-mpio-for-storsimple-volumes"></a>Configuration de MPIO pour les volumes StorSimple

1. Ouvrez **Configuration MPIO**. Cliquez sur **Gestionnaire de serveur > Tableau de bord > Outils > MPIO**.
2. Dans la boîte de dialogue **Propriétés MPIO**, sélectionnez l’onglet **Découvrir plusieurs chemins**.
3. Sélectionnez **Ajouter la prise en charge des périphériques iSCSI**, puis cliquez sur **Ajouter**.  
   ![Chemins d’accès multiples de détection de propriétés MPIO](./media/storsimple-configure-mpio-windows-server/IC741003.png)
4. Redémarrez le serveur lorsque vous y êtes invité.
5. Dans la boîte de dialogue **Propriétés MPIO**, cliquez sur l’onglet **Périphériques MPIO**. Cliquez sur **Ajouter**.
    </br>![Propriétés MPIO Périphériques MPIO](./media/storsimple-configure-mpio-windows-server/IC741004.png)
6. Dans la boîte de dialogue **Ajouter la prise en charge MPIO**, sous **ID du matériel de périphérique**, entrez le numéro de série de votre appareil. Pour obtenir le numéro de série de l’appareil, accédez à votre service StorSimple Device Manager. Accédez à **Appareils > Tableau de bord**. Le numéro de série de l’appareil s’affiche dans le volet **Aperçu rapide** de droite dans le tableau de bord de l’appareil.
    </br>
    ![Ajouter la prise en charge MPIO](./media/storsimple-configure-mpio-windows-server/IC741005.png)
7. Redémarrez le serveur lorsque vous y êtes invité.

## <a name="step-3-mount-storsimple-volumes-on-the-host"></a>Étape 3 : monter les volumes StorSimple sur l'hôte

Une fois MPIO configuré sur Windows Server, le ou les volumes créés sur l’appareil StorSimple peuvent être montés et peuvent alors tirer parti de MPIO pour la redondance. Pour monter un volume, effectuez les étapes suivantes.

#### <a name="to-mount-volumes-on-the-host"></a>Montage de volumes sur l’hôte

1. Ouvrez la fenêtre **Propriétés de l’initiateur iSCSI** sur l’hôte Windows Server. Cliquez sur **Gestionnaire de serveur > Tableau de bord > Outils > Initiateur iSCSI**.
2. Dans la boîte de dialogue **Propriétés de l’initiateur iSCSI**, cliquez sur l’onglet Détection, puis cliquez sur **Détecter un portail cible**.
3. Dans la boîte de dialogue **Détecter un portail cible**, procédez comme suit :
   
   1. Entrez l’adresse IP du port DATA de votre appareil StorSimple (par exemple, entrez DATA 0).
   2. Cliquez sur **OK** pour revenir à la boîte de dialogue **Propriétés de l’initiateur iSCSI**.
     
      > [!IMPORTANT]
      > **Si vous utilisez un réseau privé pour les connexions iSCSI, entrez l’adresse IP du port DATA connecté au réseau privé.**
    
4. Répétez les étapes 2 et 3 pour une deuxième interface réseau (par exemple, DATA 1) sur votre périphérique. N’oubliez pas que ces interfaces doivent être activées pour iSCSI. Pour plus d’informations, consultez [Modification des interfaces réseau](storsimple-8000-modify-device-config.md#modify-network-interfaces).
5. Sélectionnez l’onglet **Cibles** dans la boîte de dialogue **Propriétés de l’initiateur iSCSI**. Vous devez voir le nom qualifié cible de l’appareil StorSimple sous **Cibles découvertes**.

   ![Onglet Cibles des propriétés de l’initiateur iSCSI](./media/storsimple-configure-mpio-windows-server/IC741007.png)
   
6. Cliquez sur **Connexion** pour établir une session iSCSI avec votre appareil StorSimple. Une boîte de dialogue **Se connecter à la cible** s’affiche.
7. Dans la boîte de dialogue **Se connecter à la cible**, sélectionnez la case à cocher **Activer la prise en charge de plusieurs chemins d’accès**. Cliquez sur **Avancé**.
8. Dans la boîte de dialogue **Paramètres avancés**, procédez comme suit :
   
   1. Dans la liste déroulante **Adaptateur local**, sélectionnez **Initiateur Microsoft iSCSI**.
   2. Dans la liste déroulante **IP de l’initiateur** , sélectionnez l’adresse IP de l’hôte.
   3. Dans la liste déroulante **IP du portail cible** , sélectionnez l’adresse IP de l’interface de l’appareil.
   4. Cliquez sur **OK** pour revenir à la boîte de dialogue **Propriétés de l’initiateur iSCSI**.
9. Cliquez sur **Propriétés**. Dans la boîte de dialogue **Propriétés**, cliquez sur **Ajouter une session**.
10. Dans la boîte de dialogue **Se connecter à la cible**, sélectionnez la case à cocher **Activer la prise en charge de plusieurs chemins d’accès**. Cliquez sur **Avancé**.
11. Dans la boîte de dialogue **Paramètres avancés** :

    1. Dans la liste déroulante **Adaptateur local** , sélectionnez Initiateur Microsoft iSCSI.
    2. Dans la liste déroulante **IP de l’initiateur** , sélectionnez l’adresse IP correspondant à l’hôte. Dans ce cas, vous connectez deux interfaces réseau sur l’appareil à une seule interface réseau sur l’hôte. Par conséquent, cette interface est identique à celle fournie pour la première session.
    3. Dans la liste déroulante **IP du portail cible** , sélectionnez l’adresse IP de la deuxième interface de données activée sur l’appareil.
    4. Cliquez sur **OK** pour revenir à la boîte de dialogue Propriétés de l’initiateur iSCSI. Vous avez ajouté une deuxième session à la cible.
12. Ouvrez **Gestion de l’ordinateur** en accédant à **Gestionnaire de serveur > Tableau de bord > Gestion de l’ordinateur**. Dans le volet gauche, cliquez sur **Stockage > Gestion des disques**. Le volume créé sur l’appareil StorSimple visible pour cet hôte s’affiche sous **Gestion des disques** en tant que nouveau disque.
13. Initialisez le disque et créez un nouveau volume. Pendant le processus de formatage, sélectionnez une taille de bloc de 64 Ko.
    
    ![Gestion des disques](./media/storsimple-configure-mpio-windows-server/IC741008.png)
14. Sous **Gestion des disques**, cliquez avec le bouton droit sur le **Disque** et sélectionnez **Propriétés**.
15. Dans la boîte de dialogue du modèle StorSimple #### **Propriétés du périphérique de disque à chemins multiples**, cliquez sur l’onglet **MPIO**.
    
    ![DeviceProp disque à chemins d’accès multiples StorSimple 8100.](./media/storsimple-configure-mpio-windows-server/IC741009.png)
16. Dans la section **Nom DSM**, cliquez sur **Détails** et vérifiez que les paramètres par défaut sont définis. Les paramètres par défaut sont les suivants :
    
    * Période de vérification du chemin d’accès = 30
    * Nombre de tentatives = 3
    * Période de suppression d’objets de périphériques physiques = 20
    * Intervalle avant nouvelle tentative = 1
    * Vérification du chemin activée = désactivé.

> [!NOTE]
> **Ne modifiez pas les paramètres par défaut.**


## <a name="step-4-configure-mpio-for-high-availability-and-load-balancing"></a>Étape 4 : configurer MPIO pour la haute disponibilité et l'équilibrage de charge

Pour les chemins d’accès multiples basés sur la haute disponibilité et l’équilibrage de charge, vous devez ajouter plusieurs sessions manuellement pour déclarer les différents chemins d’accès disponibles. Par exemple, si l’hôte possède deux interfaces connectées au réseau iSCSI et si l’appareil possède deux interfaces connectées au réseau iSCSI, vous avez besoin de quatre sessions configurées avec les permutations de chemin d’accès appropriées (seules deux sessions seront requises si chaque interface DATA et interface d’hôte se trouve sur un sous-réseau d’adresse IP différent et n’est pas routable).

**Nous vous recommandons d’avoir au moins 8 sessions parallèles actives entre l’appareil et votre hôte d’application.** Cela est possible en activant 4 interfaces réseau sur votre système Windows Server. Utilisez des interfaces réseau physiques ou des interfaces virtuelles via des technologies de virtualisation de réseau au niveau du matériel ou du système d’exploitation sur votre hôte Windows Server. Avec deux interfaces réseau sur l’appareil, cette configuration permet 8 sessions actives. Cette configuration permet d’optimiser le débit de l’appareil et du cloud.

> [!IMPORTANT]
> **Nous vous recommandons de ne pas mélanger les interfaces réseau 1 Gigabit Ethernet et 10 Gigabit Ethernet. Si vous utilisez deux interfaces réseau, elles doivent être d’un même type.**

La procédure suivante décrit comment ajouter des sessions lorsqu’un appareil StorSimple possédant deux interfaces réseau est connecté à un hôte avec deux interfaces réseau. Cela vous donne seulement 4 sessions. Utilisez la même procédure avec un appareil StorSimple possédant deux interfaces réseau et connecté à un hôte avec quatre interfaces réseau. Vous devez configurer 8 sessions au lieu des 4 décrites ici.

### <a name="to-configure-mpio-for-high-availability-and-load-balancing"></a>Configuration de MPIO pour la haute disponibilité et l’équilibrage de charge

1. Effectuez une détection de la cible : dans la boîte de dialogue **Propriétés de l’initiateur iSCSI**, sur l’onglet **Détection**, cliquez sur **Découvrir un portail**.
2. Dans la boîte de dialogue **Se connecter à la cible** , entrez l’adresse IP de l’une des interfaces réseau de l’appareil.
3. Cliquez sur **OK** pour revenir à la boîte de dialogue **Propriétés de l’initiateur iSCSI**.
4. Dans la boîte de dialogue **Propriétés de l’initiateur iSCSI**, sélectionnez l’onglet **Cibles**, mettez en surbrillance la cible détectée, puis cliquez sur **Connexion**. La boîte de dialogue **Se connecter à la cible** s’affiche.
5. Dans la boîte de dialogue **Se connecter à la cible** :
   
   1. Laissez le paramètre cible sélectionné par défaut pour **Ajouter cette connexion** à la liste des cibles favorites. L’appareil tente automatiquement de redémarrer la connexion à chaque redémarrage de cet ordinateur.
   2. Sélectionnez la case à cocher **Activer la prise en charge de plusieurs chemins d’accès** .
   3. Cliquez sur **Avancé**.
6. Dans la boîte de dialogue **Paramètres avancés** :
   
   1. Dans la liste déroulante **Adaptateur local**, sélectionnez **Initiateur Microsoft iSCSI**.
   2. Dans la liste déroulante **IP de l’initiateur**, sélectionnez l’adresse IP correspondant à la première interface sur l’hôte (interface iSCSI).
   3. Dans la liste déroulante **IP du portail cible**, sélectionnez l’adresse IP de la première interface de données activée sur l’appareil.
   4. Cliquez sur **OK** pour revenir à la boîte de dialogue Propriétés de l’initiateur iSCSI.
7. Cliquez sur **Propriétés**, puis, dans la boîte de dialogue **Propriétés**, cliquez sur **Ajouter une session**.
8. Dans la boîte de dialogue **Se connecter à la cible**, sélectionnez la case à cocher **Activer la prise en charge de plusieurs chemins d’accès**, puis cliquez sur **Avancé**.
9. Dans la boîte de dialogue **Paramètres avancés** :
   
   1. Dans la liste déroulante **Adaptateur local**, sélectionnez **Initiateur Microsoft iSCSI**.
   2. Dans la liste déroulante **IP de l’initiateur**, sélectionnez l’adresse IP correspondant à la deuxième interface iSCSI sur l’hôte.
   3. Dans la liste déroulante **IP du portail cible** , sélectionnez l’adresse IP de la deuxième interface de données activée sur l’appareil.
   4. Cliquez sur **OK** pour revenir à la boîte de dialogue **Propriétés de l’initiateur iSCSI**. Vous avez à présent ajouté une deuxième session à la cible.
10. Répétez les étapes 8 à 10 pour ajouter des sessions supplémentaires (chemins d’accès) à la cible. Avec deux interfaces sur l’hôte et deux sur l’appareil, vous pouvez ajouter un total de quatre sessions.
11. Après avoir ajouté les sessions souhaitées (chemins d’accès), dans la boîte de dialogue **Propriétés de l’initiateur iSCSI**, sélectionnez la cible et cliquez sur **Propriétés**. Sous l’onglet Sessions de la boîte de dialogue **Propriétés** , notez les quatre identificateurs de session qui correspondent aux permutations de chemin d’accès possibles. Pour annuler une session, activez la case à cocher en regard d’un identificateur de session, puis cliquez sur **Déconnexion**.
12. Pour afficher les appareils présentés dans les sessions, sélectionnez l’onglet **Périphériques** . Pour configurer la stratégie MPIO pour un appareil sélectionné, cliquez sur **MPIO**. La boîte de dialogue **Informations sur l’appareil** s’affiche. Sous l’onglet **MPIO**, vous pouvez sélectionner les paramètres **Stratégie d’équilibrage de charge** appropriés. Vous pouvez également afficher le type de chemin d’accès **Actif** ou **Veille**.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur [l’utilisation du service StorSimple Device Manager pour modifier la configuration de votre appareil StorSimple](storsimple-8000-modify-device-config.md).