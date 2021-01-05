---
title: Approvisionner StorSimple Virtual Array dans Hyper-V | Microsoft Docs
description: Ce deuxième didacticiel de déploiement de StorSimple Virtual Array implique la configuration d’un tableau virtuel dans Hyper-V.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 4354963c-e09d-41ac-9c8b-f21abeae9913
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 07/25/2019
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: d6dfc95820e911781ffa88e2207601703f165839
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87070591"
---
# <a name="deploy-storsimple-virtual-array---provision-in-hyper-v"></a>Déploiement de StorSimple Virtual Array - Configuration dans Hyper-V
![Diagramme montrant les étapes nécessaires au déploiement d’un tableau virtuel. La première partie de la deuxième étape est intitulée « Approvisionner sur Hyper-V » et est mise en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/hyperv4.png)

## <a name="overview"></a>Vue d’ensemble

[!INCLUDE [storsimple-virtual-array-eol-banner](../../includes/storsimple-virtual-array-eol-banner.md)]

Il décrit la configuration d’un appareil virtuel StorSimple (StorSimple Virtual Array) sur un système hôte exécutant Hyper-V sur Windows Server 2012 R2, Windows Server 2012 ou Windows Server 2008 R2. Cet article concerne le déploiement de StorSimple Virtual Array dans le portail Azure, ainsi que dans Microsoft Azure Government Cloud.

Vous aurez besoin de privilèges d’administrateur pour configurer un tableau virtuel. La configuration initiale peut prendre environ 10 minutes.

## <a name="provisioning-prerequisites"></a>Configuration des composants requis
Vous trouverez ici les conditions requises pour la configuration d’un tableauvirtuel sur un système hôte exécutant Hyper-V sur Windows Server 2012 R2, Windows Server 2012 ou Windows Server 2008 R2.

### <a name="for-the-storsimple-device-manager-service"></a>Pour le service StorSimple Device Manager
Avant de commencer, assurez-vous que :

* Vous avez terminé toutes les étapes de la rubrique [Préparation du portail pour StorSimple Virtual Array](storsimple-virtual-array-deploy1-portal-prep.md).
* Vous avez téléchargé l’image du tableau virtuel pour Hyper-V à partir du portail Azure. Pour plus d'informations, consultez **Étape 3 : Téléchargement de l’image du tableau virtuel** de [Guide de préparation du portail pour StorSimple Virtual Array](storsimple-virtual-array-deploy1-portal-prep.md).

  > [!IMPORTANT]
  > Le logiciel exécuté sur StorSimple Virtual Array peut uniquement servir en association avec le service StorSimple Device Manager.
  >
  >

### <a name="for-the-storsimple-virtual-array"></a>Pour StorSimple Virtual Array
Avant de déployer un tableau virtuel, assurez-vous que :

* Vous avez accès à un système hôte exécutant Hyper-V sur Windows Server 2008 R2 ou une version ultérieure permettant de configurer un appareil.
* Le système hôte est en mesure de dédier les ressources suivantes pour configurer votre tableau virtuel :

  * Un minimum de 4 cœurs.
  * Au moins 8 Go de RAM. Si vous envisagez de configurer le tableau virtuel en tant que serveur de fichiers, 8 Go prennent en charge moins de 2 millions de fichiers. Vous avez besoin de 16 Go de RAM pour prendre en charge 2 à 4 millions de fichiers.
  * Une interface réseau.
  * Un disque virtuel de 500 Go pour les données.

### <a name="for-the-network-in-the-datacenter"></a>Pour le réseau dans le centre de données
Avant de commencer, passez en revue la configuration réseau requise pour déployer une instance StorSimple Virtual Array et configurez le réseau du centre de données en conséquence. Pour plus d’informations, consultez la [Configuration requise du réseau pour StorSimple Virtual Array](storsimple-ova-system-requirements.md#networking-requirements).

## <a name="step-by-step-provisioning"></a>Configuration étape par étape
Pour configurer et vous connecter à un tableau virtuel, vous devez effectuer les opérations suivantes :

1. Assurez-vous que le système hôte dispose de suffisamment de ressources pour répondre aux exigences minimales du tableau virtuel.
2. Configurez un tableau virtuel dans votre hyperviseur.
3. Démarrez le tableau virtuel et récupérez l’adresse IP.

Chacune de ces étapes est expliquée dans les sections suivantes.

## <a name="step-1-ensure-that-the-host-system-meets-minimum-virtual-array-requirements"></a>Étape 1 : Vérifier que le système hôte répond aux exigences minimales du tableau virtuel
Pour créer un tableau virtuel vous devez disposer des éléments suivants :

* Le rôle Hyper-V installé sur Windows Server 2012 R2, Windows Server 2012 ou Windows Server 2008 R2 SP1.
* Microsoft Hyper-V Manager sur un client Microsoft Windows connecté à l'hôte.

Assurez-vous que le matériel sous-jacent (système hôte) sur lequel vous créez le tableau virtuel est capable de dédier les ressources suivantes à votre tableau virtuel :

* Un minimum de 4 cœurs.
* Au moins 8 Go de RAM. Si vous envisagez de configurer le tableau virtuel en tant que serveur de fichiers, 8 Go prennent en charge moins de 2 millions de fichiers. Vous avez besoin de 16 Go de RAM pour prendre en charge 2 à 4 millions de fichiers.
* Une interface réseau.
* Un disque virtuel de 500 Go pour les données système.

## <a name="step-2-provision-a-virtual-array-in-hypervisor"></a>Étape 2 : Configurer un tableau virtuel dans l’hyperviseur
Procédez comme suit pour configurer un appareil dans votre hyperviseur.

#### <a name="to-provision-a-virtual-array"></a>Pour configurer un tableau virtuel
1. Sur votre hôte Windows Server, copiez l’image du tableau virtuel sur un disque local. Vous avez téléchargé cette image (VHD ou VHDX) via le portail Azure. Prenez note de l’emplacement où vous avez copié l’image car vous l’utiliserez plus loin dans la procédure.
2. Ouvrez **Gestionnaire de serveur**. Dans le coin supérieur droit, cliquez sur **Outils**, puis sélectionnez **Gestionnaire Hyper-V**.

   ![Capture d’écran de Gestionnaire de serveur montrant un menu Outils développé avec l’élément Gestionnaire Hyper-V mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image1.png)  

   Si vous utilisez Windows Server 2008 R2, ouvrez le Gestionnaire Hyper-V. Dans le Gestionnaire de serveur, cliquez sur **Rôles > Hyper-V > Gestionnaire Hyper-V**.
3. Dans le **Gestionnaire Hyper-V**, cliquez avec le bouton droit dans le volet d’étendue sur le nœud de votre système pour ouvrir le menu contextuel, puis cliquez sur **Nouveau** > **Machine virtuelle**.

   ![Capture d’écran de Gestionnaire Hyper-V montrant le menu contextuel d’un nœud système, avec les éléments Nouveau et Machine virtuelle mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image2.png)
4. Dans la page **Avant de commencer** de l’Assistant Nouvel ordinateur virtuel, cliquez sur **Suivant**.
5. Sur la page **Spécifier le nom et l’emplacement**, indiquez le **Nom** de votre tableau virtuel. Cliquez sur **Suivant**.

   ![Capture d’écran de la page Spécifier le nom et l’emplacement montrant la zone Nom renseignée et le bouton Suivant mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image4.png)
6. Dans la plage **Spécifier la génération**, choisissez le type d’image d’appareil, puis cliquez sur **Suivant**. Cette page ne s’affiche pas si vous utilisez Windows Server 2008 R2.

   * Choisissez **Génération 2** si vous avez téléchargé une image .vhdx pour Windows Server 2012 ou version ultérieure.
   * Choisissez **Génération 1** si vous avez téléchargé une image .vhdx pour Windows Server 2008 R2 ou version ultérieure.

   ![Capture d’écran de la page Spécifier la génération montrant l’option Génération 2 sélectionnée et le bouton Suivant mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image5.png)
7. Dans la page **Affecter la mémoire**, spécifiez une **Mémoire de démarrage** d’au moins **8 192 Mo** (n’activez pas la mémoire dynamique), puis cliquez sur **Suivant**.

   ![Capture d’écran de la page Affecter la mémoire montrant une valeur de 8192 dans la zone Mémoire de démarrage. Le bouton Suivant est mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image6.png)  
8. Dans la page **Configurer le réseau**, spécifiez le commutateur virtuel connecté à Internet, puis sur **Suivant**.

   ![Capture d’écran de la page Configurer le réseau montrant l’élément du commutateur virtuel Hyper-V sélectionné dans la zone Connexion et le bouton Suivant mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image7.png)
9. Dans la page **Connecter un disque dur virtuel** choisissez **Utiliser un disque dur virtuel existant**, spécifiez l’emplacement de l’image du tableau virtuel (.vhdx ou .vhd), puis cliquez sur **Suivant**.

   ![Capture d’écran de la page Connecter un disque dur virtuel avec l’option Utiliser un disque dur virtuel existant sélectionnée, un emplacement entré et le bouton Suivant mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image8m.png)
10. Passez en revue le **Sommaire**, puis cliquez sur **Terminer** pour créer la machine virtuelle.

    ![Capture d’écran de la page de l’Assistant Terminer la nouvelle machine virtuelle montrant la description d’une machine virtuelle. Le bouton Terminer est mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image9.png)
11. Pour répondre à la configuration minimale requise, vous avez besoin de 4 cœurs. Pour ajouter 4 processeurs virtuels, sélectionnez votre système hôte dans la fenêtre **Hyper-V Manager**. Dans le volet droit, sous la liste des **Machines virtuelles**, identifiez la machine virtuelle nouvellement créée. Sélectionnez et cliquez avec le bouton droit sur le nom de la machine, puis sélectionnez **Paramètres**.

    ![Capture d’écran de la fenêtre Gestionnaire Hyper-V. Une machine est sélectionnée dans la liste et l’élément Paramètres est mis en surbrillance dans le menu contextuel de la machine.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image10.png)
12. Sur la page **Paramètres**, cliquez sur **Processeur** dans le volet gauche. Dans le volet droit, définissez le **nombre de processeurs virtuels** sur 4 (ou plus). Cliquez sur **Appliquer**.

    ![Capture d’écran de la page Paramètres avec l’élément Processeur mis en surbrillance, le nombre de processeurs virtuels définis sur quatre et le bouton Appliquer mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image11.png)
13. Pour répondre à la configuration minimale requise, vous devez également ajouter un disque de données virtuel de 500 Go. Dans la page **Paramètres** :

    1. Dans le volet gauche, sélectionnez **Contrôleur SCSI**.
    2. Dans le volet droit, sélectionnez **Disque dur**, puis cliquez sur **Ajouter**.

    ![Capture d’écran de la page Paramètres avec l’élément Contrôleur SCSI sélectionné, l’élément Disque dur sélectionné et le bouton Ajouter mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image12.png)
14. Dans la page **Disque dur**, sélectionnez l’option **Disque dur virtuel**, puis cliquez sur **Nouveau**. L’**Assistant Nouveau disque dur virtuel** démarre.

    ![Capture d’écran de la page Paramètres avec l’élément Disque dur sélectionné, l’option Disque dur virtuel sélectionnée et le bouton Nouveau mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image13.png)
15. Dans la page **Avant de commencer** de l’Assistant Nouveau disque dur virtuel, cliquez sur **Suivant**.
16. Dans la page **Choisir le format de disque**, acceptez le format **VHDX** par défaut. Cliquez sur **Suivant**. Cet écran ne s’affiche pas si vous exécutez Windows Server 2008 R2.

    ![Capture d’écran de la page Choisir le format de disque avec l’option VHDX sélectionnée et le bouton Suivant mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image15.png)
17. Dans la page **Choisir le type de disque**, définissez le type de disque dur virtuel sur **Extension dynamique** (recommandé). Si vous choisissez un disque de **taille fixe**, il fonctionnera également, mais vous devrez peut-être patienter plus longtemps. Nous vous recommandons de ne pas utiliser l’option de **différenciation** . Cliquez sur **Suivant**. Dans Windows Server 2012 R2 et Windows Server 2012, **taille dynamique** est l’option par défaut, tandis que dans Windows Server 2008 R2, l’option par défaut est **Taille fixe**.

    ![Capture d’écran de la page Choisir le type de disque avec l’option Extension dynamique sélectionnée et le bouton Suivant mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image16.png)
18. Dans la page **Spécifier le nom et l’emplacement**, indiquez le **nom** et **l’emplacement** (vous pouvez naviguer vers cet emplacement) du disque de données. Cliquez sur **Suivant**.

    ![Capture d’écran de la page Spécifier le nom et l’emplacement montrant des valeurs dans les zones Nom et Emplacement. Le bouton Suivant est mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image17.png)
19. Dans la page **Configurer un disque**, sélectionnez l’option **Créer un disque dur virtuel vierge** et spécifiez une taille de **500 Go** (ou plus). Bien que 500 Go soit la configuration minimale requise, vous pouvez toujours configurer un disque plus volumineux. Notez que vous ne pouvez pas développer ou réduire le disque une fois mis en service. Pour plus d’informations sur la taille du disque à approvisionner, consultez la section sur le dimensionnement dans le document [Meilleures pratiques](storsimple-ova-best-practices.md). Cliquez sur **Suivant**.

    ![Capture d’écran de la page Configurer un disque avec l’option Créer un disque dur virtuel vierge sélectionnée, la taille définie sur 500 et le bouton Suivant mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image18.png)
20. Dans la page **Résumé**, passez en revue les détails de votre disque de données virtuel et, si vous êtes satisfait, cliquez sur **Terminer** pour créer le disque. L’Assistant se ferme et un disque dur virtuel est ajouté à votre machine.

    ![Capture d’écran de la page de l’Assistant Terminer le nouveau disque dur virtuel montrant la description d’un disque dur virtuel. Le bouton Terminer est mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image19.png)
21. Revenez à la page **Paramètres**. Cliquez sur **OK** pour fermer la page **Paramètres** et revenir à la fenêtre du Gestionnaire Hyper-V.

    ![Capture d’écran de la page Paramètres. Le bouton OK est mis en surbrillance.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image20.png)

## <a name="step-3-start-the-virtual-array-and-get-the-ip"></a>Étape 3 : Démarrer le tableau virtuel et récupérer l’adresse IP
Procédez comme suit pour démarrer votre tableau virtuel et vous y connecter.

#### <a name="to-start-the-virtual-array"></a>Pour démarrer le tableau virtuel
1. Démarrez le tableau virtuel.

   ![Capture d’écran de la fenêtre Gestionnaire Hyper-V. Le nouveau tableau est sélectionné dans la liste et l’élément Démarrer est mis en surbrillance dans le menu contextuel de ce tableau.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image21.png)
2. Une fois l'appareil en cours d'exécution, sélectionnez-le, cliquez avec le bouton droit et sélectionnez **Connexion**.

   ![Capture d’écran de la fenêtre Gestionnaire Hyper-V. Le nouveau tableau est sélectionné, son état est En cours d’exécution et l’élément Connexion est mis en surbrillance dans le menu contextuel.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image22.png)
3. Vous devrez peut-être patienter 5 à 10 minutes pour que l’appareil soit prêt. Un message d'état s'affiche sur la console pour indiquer la progression. Lorsque l'appareil est prêt, sélectionnez **Action**. Appuyez sur `Ctrl + Alt + Delete` pour vous connecter au tableau virtuel. L’utilisateur par défaut est *StorSimpleAdmin* et le mot de passe par défaut est *Password1*.

   ![Capture d’écran de l’onglet Console du client vSphere montrant des caractères indéterminés dans la zone de connexion.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image23.png)
4. Pour des raisons de sécurité, le mot de passe d’administrateur réseau expire à la première connexion. Vous êtes invité à modifier le mot de passe.

   ![Capture d’écran de l’onglet Console du client vSphere. Le texte sur la page indique que le mot de passe doit être modifié.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image24.png)

   Entrez un mot de passe contenant au moins 8 caractères. Le mot de passe doit contenir au moins 3 des 4 caractères suivants : majuscules, minuscules, caractères numériques et caractères spéciaux. Entrez de nouveau le mot de passe pour le confirmer. Vous êtes informé de la modification du mot de passe.

   ![Capture d’écran de l’onglet Console du client vSphere. Le texte sur la page indique que le mot de passe a été modifié.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image25.png)
5. Une fois le mot de passe modifié, le tableau virtuel peut redémarrer. Attendez le démarrage de l’appareil.

   ![Capture d’écran de la page StorSimpleAdmin principale. Le texte sur la page demande à l’utilisateur d’attendre le service de notification d’événements système.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image26.png)

    La console Windows PowerShell de l’appareil s’affiche avec une barre de progression.

   ![Capture d’écran montrant une fenêtre de console avec une barre de progression. Le texte dans la fenêtre indique que la configuration initiale est en cours et demande à l’utilisateur d’attendre.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image27.png)
6. Les étapes 6 à 8 s’appliquent uniquement lors de l’amorçage dans un environnement non DHCP. Si vous êtes dans un environnement DHCP, ignorez ces étapes et passez à l'étape 9. Si vous avez démarré votre appareil dans un environnement non DHCP, vous verrez l’écran suivant.

   ![Capture d’écran montrant une fenêtre de console avec le texte décrivant l’appareil. L’invite de commandes lit « Contrôleur » et apparaît comme étant prête pour l’entrée.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image28m.png)

    Ensuite, configurez le réseau.
7. Utilisez la commande `Get-HcsIpAddress` pour répertorier les interfaces réseau activées sur votre tableau virtuel. Si votre appareil possède une seule interface réseau activée, le nom par défaut affecté à cette interface est `Ethernet`.

   ![Capture d’écran montrant une fenêtre de console avec la sortie de la commande Get-HcsIpAddress. « Ethernet » est indiqué comme nom de l’appareil.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image29m.png)
8. Utilisez l’applet de commande `Set-HcsIpAddress` pour configurer le réseau. Voir l’exemple suivant :

    `Set-HcsIpAddress –Name Ethernet –IpAddress 10.161.22.90 –Netmask 255.255.255.0 –Gateway 10.161.22.1`

    ![Capture d’écran montrant une fenêtre de console avec la sortie de la commande Get-Help Set-HcsIpAddress et l’utilisation correcte de la commande Set-HcsIpAddress.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image30.png)
9. Une fois l'installation initiale terminée et l’appareil démarré, vous verrez le texte de la bannière de l’appareil. Notez l'adresse IP et l'URL affichées dans le texte de la bannière pour gérer l'appareil. Vous utiliserez cette adresse IP pour vous connecter à l’interface utilisateur web de votre tableau virtuel et pour finaliser la configuration et l’inscription locales.

   ![Capture d’écran montrant une fenêtre de console avec le texte de la bannière d’appareil. Ce texte comprend l’adresse IP et l’URL de l’appareil.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image31m.png)
10. (Facultatif) Effectuez cette étape uniquement si vous déployez votre appareil dans Government Cloud. Vous allez maintenant activer le mode FIPS (Federal Information Processing Standard) sur votre appareil. La norme FIPS 140 définit les algorithmes de chiffrement qui sont approuvés pour une utilisation sur les systèmes informatiques du gouvernement fédéral américain dans le but de protéger les données sensibles.

    1. Pour activer le mode FIPS, exécutez l’applet de commande suivante :

        `Enable-HcsFIPSMode`
    2. Après avoir activé le mode FIPS, redémarrez votre appareil pour que les validations de chiffrement prennent effet.

       > [!NOTE]
       > Vous pouvez activer ou désactiver le mode FIPS sur votre appareil. L’alternance entre le mode FIPS et le mode non FIPS n’est pas prise en charge.
       >
       >

Si votre appareil ne répond pas à la configuration minimale requise, une erreur apparaît dans le texte de bannière (voir ci-dessous). Modifier la configuration de l’appareil afin qu’il dispose des ressources nécessaires à la configuration minimale. Vous pouvez ensuite redémarrer et vous connecter à l'appareil. Reportez-vous à la configuration minimale requise dans Étape 1 : Vérifier que le système hôte répond aux exigences minimales du tableau virtuel.

![Capture d’écran montrant une fenêtre de console avec le texte de la bannière d’appareil. Ce texte inclut un message d’erreur qui fournit une URL pour résoudre le problème.](./media/storsimple-virtual-array-deploy2-provision-hyperv/image32.png)

SI vous rencontrez une autre erreur durant la configuration initiale effectuée avec l’interface utilisateur web locale, reportez-vous aux workflows suivants :

* Exécutez les tests de diagnostic pour [dépanner la configuration de l’interface utilisateur web](storsimple-ova-web-ui-admin.md#troubleshoot-web-ui-setup-errors).
* [Générez un package de journaux et affichez les fichiers journaux](storsimple-ova-web-ui-admin.md#generate-a-log-package).

## <a name="next-steps"></a>Étapes suivantes
* [Configurer StorSimple Virtual Array comme un serveur de fichiers](storsimple-virtual-array-deploy3-fs-setup.md)
* [Configurer StorSimple Virtual Array comme un serveur iSCSI](storsimple-virtual-array-deploy3-iscsi-setup.md)
