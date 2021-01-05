---
title: PowerShell pour la gestion des appareils StorSimple
description: Découvrez comment gérer votre appareil StorSimple à l’aide de Windows PowerShell for StorSimple.
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 01/09/2018
ms.author: alkohli
ms.openlocfilehash: 65e9657c3948d8ce5883cd33ca8720f501352105
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995423"
---
# <a name="use-windows-powershell-for-storsimple-to-administer-your-device"></a>Utiliser Windows PowerShell pour StorSimple pour appliquer votre appareil

## <a name="overview"></a>Vue d’ensemble

Windows PowerShell pour StorSimple fournit une interface de ligne de commande que vous pouvez utiliser pour gérer votre appareil Microsoft Azure StorSimple. Comme son nom l’indique, il s’agit d’une interface de ligne de commande basée sur Windows PowerShell, qui est créée dans une instance d’exécution avec restriction. Du point de vue de l’utilisateur au niveau de la ligne de commande, une instance d’exécution avec restriction apparaît comme une version limitée de Windows PowerShell. Tout en conservant certaines fonctionnalités de base de Windows PowerShell, cette interface a des applets de commande dédiées supplémentaires orientées vers la gestion de votre appareil Microsoft Azure StorSimple.

Cet article décrit les fonctionnalités de Windows PowerShell for StorSimple, y compris la façon dont vous pouvez vous connecter à cette interface, et comprend des liens vers des procédures pas à pas ou des flux de travail que vous pouvez effectuer à l’aide de cette interface. Les flux de travail sont notamment les suivants : comment inscrire votre appareil, configurer l’interface réseau sur votre appareil, installer les mises à jour nécessitant que l’appareil soit en mode maintenance, changer l’état de l’appareil et résoudre les problèmes que vous pouvez rencontrer.

Après avoir lu cet article, vous pourrez :

* Connectez-vous à votre appareil StorSimple à l’aide de Windows PowerShell for StorSimple.
* Administrez votre appareil StorSimple à l’aide de Windows PowerShell for StorSimple.
* Obtenez de l’aide dans Windows PowerShell for StorSimple.

> [!NOTE]
> * Les applets de commande de Windows PowerShell pour StorSimple vous permettent de gérer votre appareil StorSimple depuis une console série ou à distance via l’accès distant Windows PowerShell. Pour plus d’informations sur chacune des applets de commande individuelles qui peuvent être utilisés dans cette interface, consultez les [informations de référence sur les applets de commande pour Windows PowerShell pour StorSimple](/powershell/module/hcs/?viewFallbackFrom=winserverr2-ps).
> * Les applets de commande d’Azure PowerShell StorSimple sont une autre collection d’applets de commande qui vous permettent d’automatiser des tâches de niveau service et de migration StorSimple à partir de la ligne de commande. Pour plus d’informations sur les applets de commande d’Azure PowerShell pour StorSimple, consultez [Informations de référence sur les applets de commande d’Azure StorSimple](/powershell/module/servicemanagement/azure.service/?view=azuresmps-4.0.0&viewFallbackFrom=azuresmps-3.7.0#azure).


Vous pouvez accéder à Windows PowerShell pour StorSimple selon une des méthodes suivantes :

* [Se connecter à la console série de l’appareil StorSimple](#connect-to-windows-powershell-for-storsimple-via-the-device-serial-console)
* [Se connecter à distance à StorSimple en utilisant Windows PowerShell](#connect-remotely-to-storsimple-using-windows-powershell-for-storsimple)

## <a name="connect-to-windows-powershell-for-storsimple-via-the-device-serial-console"></a>Se connecter à Windows PowerShell pour StorSimple via la console série de l’appareil

Vous pouvez [télécharger PuTTY](https://www.putty.org/) ou un logiciel d’émulation de terminal similaire pour vous connecter à Windows PowerShell pour StorSimple. Vous devez configurer PuTTY de façon spécifique pour accéder à l’appareil Microsoft Azure StorSimple. Les rubriques suivantes décrivent en détail comment configurer PuTTy et se connecter à l’appareil. Différentes options de menu de la console série sont également expliquées.

### <a name="putty-settings"></a>Paramètres puTTY

Assurez-vous d’utiliser les paramètres PuTTY suivants pour vous connecter à l’interface Windows PowerShell à partir de la console série.

#### <a name="to-configure-putty"></a>Pour configurer PuTTY

1. Dans la boîte de dialogue **Reconfiguration**, dans le volet **Catégorie**, sélectionnez **Clavier**.
2. Assurez-vous que les options suivantes sont sélectionnées (ce sont les paramètres par défaut quand vous démarrez une nouvelle session).
   
   | Élément du clavier | Sélectionnez |
   | --- | --- |
   | Touche Retour arrière |Ctrl+? (127) |
   | Touches Origine et Fin |standard |
   | Touches de fonction et pavé numérique |ÉCHAP[n~ |
   | État initial des touches de direction |Normal |
   | État initial du pavé numérique |Normal |
   | Activer des fonctionnalités supplémentaires du clavier |Ctrl+Alt est différent de AltGr |
   
    ![Paramètres de PuTTY pris en charge](./media/storsimple-windows-powershell-administration/IC740877.png)
3. Cliquez sur **Appliquer**.
4. Dans le volet **Catégorie**, sélectionnez **Traduction**.
5. Dans la zone de liste **Jeu de caractères distant**, sélectionnez **UTF-8**.
6. Sous **Gestion des caractères de dessin de ligne**, sélectionnez **Utiliser des points de code de dessin de ligne Unicode**. La capture d’écran suivante montre les sélections correctes de PuTTY.
   
    ![Paramètres puTTY pour UTF](./media/storsimple-windows-powershell-administration/IC740878.png)
7. Cliquez sur **Appliquer**.

Vous pouvez maintenant utiliser PuTTY pour vous connecter à la console série de l’appareil en procédant comme suit :

[!INCLUDE [storsimple-use-putty](../../includes/storsimple-use-putty.md)]

### <a name="about-the-serial-console"></a>À propos de la console série

Quand vous accédez à l’interface Windows PowerShell de votre appareil StorSimple via la console série, un message de bannière est présenté, suivi d’options de menu.

Le message de bannière contient des informations de base de l’appareil StorSimple, comme le modèle, le nom, la version du logiciel installé et l’état du contrôleur auquel vous accédez. L’image suivante montre un exemple de message de bannière.

![Message de bannière série](./media/storsimple-windows-powershell-administration/IC741098.png)

> [!IMPORTANT]
> Vous pouvez utiliser le message de bannière pour déterminer si le contrôleur auquel vous êtes connecté est _actif_ ou _passif_.

L’image suivante montre les différentes options d’instance d’exécution qui sont disponibles dans le menu de la console série.

![Inscrire votre appareil 2](./media/storsimple-windows-powershell-administration/IC740906.png)

Vous pouvez choisir parmi les paramètres suivants :

1. **Se connecter avec accès total** : cette option vous permet de vous connecter (avec les informations d’identification correctes) à l’instance d’exécution **SSAdminConsole** sur le contrôleur local. (Le contrôleur local est le contrôleur auquel vous accédez actuellement via la console série de votre appareil StorSimple.) Cette option peut également être utilisée pour permettre au support technique de Microsoft d’accéder à une instance d’exécution sans restriction (une session de support) pour résoudre tous les problèmes possibles de l’appareil. Après avoir utilisé l’option 1 pour vous connecter, vous pouvez autoriser l’ingénieur du support technique de Microsoft à accéder à une instance d’exécution sans restriction en exécutant une applet de commande spécifique. Pour plus d’informations, reportez-vous à [Démarrer une session de support](storsimple-8000-contact-microsoft-support.md#start-a-support-session-in-windows-powershell-for-storsimple).
   
2. **Se connecter au contrôleur homologue avec accès total** : cette option est identique à l’option 1, sauf que vous pouvez vous connecter (avec les informations d’identification correctes) à l’instance d’exécution **SSAdminConsole** sur le contrôleur homologue. Étant donné que l’appareil StorSimple est un appareil à haute disponibilité avec deux contrôleurs dans une configuration active-passive, « homologue » fait référence à l’autre contrôleur dans l’appareil auquel vous accédez via la console série.
   Similaire à l’option 1, cette option peut également être utilisée pour permettre au support technique de Microsoft d’accéder à une instance d’exécution sans restriction sur un contrôleur homologue.

3. **Se connecter avec un accès limité** : cette option est utilisée pour accéder à l’interface Windows PowerShell en mode limité. Des informations d’identification d’accès ne vous sont pas demandées. Cette option se connecte à une instance d’exécution plus restreinte par rapport aux options 1 et 2.  Voici certaines des tâches qui sont disponibles via l’option 1 et qui **ne peuvent pas* être effectuées dans cette instance d’exécution :
   
   * Réinitialiser les paramètres d’usine
   * Changer le mot de passe
   * Activer ou désactiver l’accès du support
   * Appliquer des mises à jour
   * Installer des correctifs

     > [!NOTE]
     > Il s’agit de l’option préférée si vous avez oublié le mot de passe administrateur et que vous ne pouvez pas vous connecter via l’option 1 ou 2.

4. **Changer la langue** : cette option vous permet de changer la langue d’affichage de l’interface Windows PowerShell. Les langues prises en charge sont : anglais, japonais, russe, français, coréen du sud, espagnol, italien, allemand, chinois et portugais.

## <a name="connect-remotely-to-storsimple-using-windows-powershell-for-storsimple"></a>Se connecter à distance à StorSimple en utilisant Windows PowerShell pour StorSimple

Vous pouvez utiliser l’accès distant Windows PowerShell pour vous connecter à votre appareil StorSimple. Quand vous vous connectez de cette façon, vous ne voyez pas de menu. (Vous voyez un menu seulement si vous utilisez la console série sur l’appareil pour vous connecter. La connexion à distance permet d’accéder directement à l’équivalent de « option 1 : accès complet » de la console série.) Avec l’accès distant Windows PowerShell, vous vous connectez à une instance d’exécution spécifique. Vous pouvez également spécifier la langue d’affichage.

La langue d’affichage est indépendante de la langue que vous définissez à l’aide de l’option **Changer la langue** dans le menu de la console série. PowerShell à distance récupère automatiquement les paramètres régionaux de l’appareil à partir duquel vous vous connectez si ces paramètres ne sont pas spécifiés.

> [!NOTE]
> Si vous travaillez avec des hôtes virtuels Microsoft Azure et des appliances cloud StorSimple, vous pouvez utiliser l’accès distant Windows PowerShell et l’hôte virtuel pour vous connecter à l’appliance cloud. Si vous avez configuré un emplacement de partage sur l’hôte où enregistrer les informations de la session Windows PowerShell, notez bien que le _principal Tout le monde_ inclut seulement des utilisateurs authentifiés. Par conséquent, si vous avez configuré le partage pour permettre l’accès par _Tout le monde_ et que vous vous connectez sans spécifier d’informations d’identification, le principal Anonyme non authentifié est utilisé et vous recevez une erreur. Pour résoudre ce problème, sur le partage hôte, vous devez activer le compte Invité, puis donner à ce compte un accès total au partage, ou bien vous devez spécifier des informations d’identification valides avec l’applet de commande Windows PowerShell.


Vous pouvez utiliser HTTP ou HTTPS pour vous connecter via l’accès distant Windows PowerShell. Utilisez les instructions des didacticiels suivants :

* [Se connecter à distance en utilisant HTTP](storsimple-8000-remote-connect.md#connect-through-http)
* [Se connecter à distance en utilisant HTTPS](storsimple-8000-remote-connect.md#connect-through-https)

## <a name="connection-security-considerations"></a>Considérations sur la sécurité des connexions

Quand vous décidez comment vous connecter à Windows PowerShell pour StorSimple, prenez en compte les points suivants :

* Une connexion directe à la console série de l’appareil est sécurisée, mais une connexion à la console via des commutateurs réseau ne l’est pas. Méfiez-vous des risques de sécurité lors de la connexion à la console série d’un appareil via des commutateurs réseau.
* Une connexion via une session HTTP peut offrir davantage de sécurité que la connexion via la console série sur un réseau. Bien que cela ne soit pas la méthode la plus sécurisée, elle est acceptable sur des réseaux approuvés.
* La connexion via une session HTTPS est l’option la plus sécurisée et c’est celle qui est recommandée.

## <a name="administer-your-storsimple-device-using-windows-powershell-for-storsimple"></a>Administrer votre appareil StorSimple à l’aide de Windows PowerShell pour StorSimple

Le tableau suivant récapitule toutes les tâches de gestion courantes et les flux de travail complexes pouvant être effectués dans l’interface utilisateur Windows PowerShell de votre appareil StorSimple Manager. Pour plus d’informations sur chaque flux de travail, cliquez sur l’entrée correspondante dans le tableau.

#### <a name="windows-powershell-for-storsimple-workflows"></a>Flux de travail Windows PowerShell pour StorSimple

| Pour ce faire... | Suivez cette procédure. |
| --- | --- |
| Inscrire votre appareil |[Configurer et inscrire l’appareil en utilisant Windows PowerShell pour StorSimple](storsimple-8000-deployment-walkthrough-u2.md#step-3-configure-and-register-the-device-through-windows-powershell-for-storsimple) |
| Configurer le proxy web</br>Afficher les paramètres de proxy web |[Configurer le proxy web pour votre appareil StorSimple](storsimple-8000-configure-web-proxy.md) |
| Modifier les paramètres d’interface réseau DATA 0 sur votre appareil |[Modifier l’interface réseau DATA 0 pour votre appareil StorSimple](storsimple-8000-modify-data-0.md) |
| Arrêter un contrôleur </br> Redémarrage ou arrêt d’un contrôleur </br> Arrêt d'un appareil</br>Rétablissement des paramètres d’usine de l’appareil |[Gérer les contrôleurs de l’appareil](storsimple-8000-manage-device-controller.md) |
| Installer des mises à jour et des correctifs en mode maintenance |[Mise à jour de votre appareil](storsimple-update-device.md) |
| Entrer en mode maintenance </br>Quitter le mode maintenance |[Modes de l’appareil StorSimple](storsimple-8000-device-modes.md) |
| Créer un package de support</br>Déchiffrer et modifier un package de support |[Création et gestion d’un package de prise en charge](storsimple-8000-create-manage-support-package.md) |
| Démarrer une session de support</br> |[Démarrage d’une session de support dans Windows PowerShell pour StorSimple](storsimple-8000-create-manage-support-package.md#create-a-support-package) |

## <a name="get-help-in-windows-powershell-for-storsimple"></a>Obtenir de l’aide dans Windows PowerShell pour StorSimple

Dans Windows PowerShell pour StorSimple, une aide sur les applets de commande est disponible. Une version en ligne et à jour de cette aide est également disponible, que vous pouvez utiliser pour mettre à jour l’aide sur votre système.

Obtenir de l’aide dans cette interface est similaire à ce qu’il faut faire dans Windows PowerShell, et la plupart des applets de commande relatives à l’aide fonctionnent. Vous pouvez trouver de l’aide en ligne pour Windows PowerShell : [Microsoft.PowerShell.Core](/powershell/module/Microsoft.PowerShell.Core/).

Voici une brève description des types d’aide pour cette interface Windows PowerShell, y compris la mise à jour de l’aide.

### <a name="to-get-help-for-a-cmdlet"></a>Pour obtenir de l’aide pour une applet de commande

* Pour obtenir de l’aide pour une applet de commande ou une fonction, utilisez la commande suivante : `Get-Help <cmdlet-name>`
* Pour obtenir de l’aide en ligne pour une applet de commande, utilisez l’applet de commande précédente avec le paramètre `-Online` : `Get-Help <cmdlet-name> -Online`
* Pour obtenir une aide complète, vous pouvez utiliser le paramètre `–Full`. Pour obtenir des exemples, utilisez le paramètre `–Examples`.

### <a name="to-update-help"></a>Pour mettre à jour l’aide

Vous pouvez facilement mettre à jour l’aide de l’interface Windows PowerShell. Procédez comme suit pour mettre à jour l’aide sur votre système.

#### <a name="to-update-cmdlet-help"></a>Pour mettre à jour l’aide des applets de commande
1. Démarrez Windows PowerShell avec l’option **Exécuter en tant qu’administrateur** .
2. À l’invite de commandes, tapez : `Update-Help`
3. Les fichiers d’aide mis à jour sont installés.
4. Une fois les fichiers d’aide sont installés, tapez : `Get-Help Get-Command`. Ceci affiche une liste des applets de commande pour lesquelles de l’aide est disponible.

> [!NOTE]
> Pour obtenir une liste de toutes les applets de commande disponibles dans une instance d’exécution, connectez-vous à l’option de menu correspondante et exécutez l’applet de commande `Get-Command`.


## <a name="next-steps"></a>Étapes suivantes

Si vous rencontrez des problèmes avec votre appareil StorSimple lors de l’exécution d’un des flux de travail ci-dessus, consultez la rubrique [Outils de résolution des problèmes de déploiement de StorSimple](storsimple-8000-troubleshoot-deployment.md#tools-for-troubleshooting-storsimple-deployments).