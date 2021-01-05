---
title: Configuration d’un serveur iSCSI Microsoft Azure StorSimple Virtual Array | Microsoft Docs
description: Explique comment effectuer la configuration initiale, inscrire votre serveur iSCSI StorSimple, puis finaliser la configuration de l’appareil.
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 4db116d1-978b-48e8-b572-a719a8425dbc
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 07/25/2019
ms.author: alkohli
ms.openlocfilehash: 6289f335234c9b7efec02a18e12b40a1140c1b3f
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94956549"
---
# <a name="deploy-storsimple-virtual-array--set-up-as-an-iscsi-server-via-azure-portal"></a>Déploiement de StorSimple Virtual Array - Configuration d’un serveur iSCSI via le portail Azure

![flux du processus d'installation iscsi](./media/storsimple-virtual-array-deploy3-iscsi-setup/iscsi4.png)

## <a name="overview"></a>Vue d’ensemble

[!INCLUDE [storsimple-virtual-array-eol-banner](../../includes/storsimple-virtual-array-eol-banner.md)]

Ce didacticiel de déploiement s’applique à l’instance Microsoft Azure StorSimple Virtual Array. Ce didacticiel explique comment effectuer la configuration initiale, inscrire votre serveur iSCSI StorSimple, finaliser l’installation de l’appareil, puis créer, monter, initialiser et formater des volumes sur votre instance StorSimple Virtual Array configurée en tant que serveur iSCSI. 

Les procédures décrites ici prennent de 30 minutes à 1 heure environ. Les informations publiées dans cet article s'appliquent uniquement aux tableaux virtuels StorSimple.

## <a name="setup-prerequisites"></a>Configuration requise

Avant de configurer votre solution StorSimple Virtual Array, assurez-vous que :

* Vous avez configuré un tableau virtuel et vous êtes connecté à celui-ci, comme décrit dans [Déploiement de StorSimple Virtual Array - Configuration d’un tableau virtuel dans Hyper-V](./storsimple-virtual-array-deploy2-provision-hyperv.md) ou [Déploiement de StorSimple Virtual Array - Configuration d’un tableau virtuel dans VMware](storsimple-virtual-array-deploy2-provision-vmware.md).
* Vous disposez de la clé d’inscription du service StorSimple Device Manager que vous avez créé pour gérer les instances StorSimple Virtual Array. Pour plus d'informations, consultez **Étape 2 : Obtention de la clé d'inscription** dans [Déploiement de StorSimple Virtual Array - Préparation du portail](storsimple-virtual-array-deploy1-portal-prep.md#step-2-get-the-service-registration-key).
* S’il s’agit de la deuxième baie virtuelle (ou d’une baie ultérieure) que vous inscrivez auprès d’un service StorSimple Device Manager existant, vous devez disposer de la clé de chiffrement des données du service. Cette clé a été générée lorsque le premier appareil a été correctement inscrit auprès de ce service. Si vous avez perdu cette clé, consultez la rubrique **Obtenir la clé de chiffrement des données de service** dans [Utiliser l’interface utilisateur web pour gérer votre StorSimple Virtual Array](storsimple-ova-web-ui-admin.md#get-the-service-data-encryption-key).

## <a name="step-by-step-setup"></a>Configuration étape par étape

Utilisez la procédure détaillée ci-dessous pour configurer votre solution StorSimple Virtual Array :

* [Étape 1 : Finalisation de la configuration de l'interface utilisateur web locale et inscription de votre appareil](#step-1-complete-the-local-web-ui-setup-and-register-your-device)
* Étape 2 : Finalisation de la configuration requise pour l’appareil
* [Étape 3 : Ajout d’un volume](#step-3-add-a-volume)
* [Étape 4 : Monter, initialiser et formater un volume](#step-4-mount-initialize-and-format-a-volume)

## <a name="step-1-complete-the-local-web-ui-setup-and-register-your-device"></a>Étape 1 : Finalisation de la configuration de l'interface utilisateur web locale et inscription de votre appareil

#### <a name="to-complete-the-setup-and-register-the-device"></a>Pour finaliser l'installation et l’inscription de l'appareil

1. Ouvrez une fenêtre de navigateur. Pour vous connecter au type d’UI web :
   
    `https://<ip-address of network interface>`
   
    Utilisez l'URL de connexion notée à l'étape précédente. Une erreur vous indique qu'il existe un problème avec le certificat de sécurité du site web. Cliquez sur **Poursuivre sur cette page web**.
   
    ![erreur de certificat de sécurité](./media/storsimple-virtual-array-deploy3-iscsi-setup/image3.png)
2. Connectez-vous à l'interface utilisateur web de votre appareil virtuel en tant que **StorSimpleAdmin**. Entrez le mot de passe de l'administrateur de l'appareil que vous avez modifié à l'Étape 3 : Démarrage de l’appareil virtuel dans [Déploiement de StorSimple Virtual Array - Configuration d’un appareil virtuel dans Hyper-V](storsimple-virtual-array-deploy2-provision-hyperv.md) ou [Déploiement de StorSimple Virtual Array - Configuration d’un appareil virtuel dans VMware](storsimple-virtual-array-deploy2-provision-vmware.md).
   
    ![page de connexion](./media/storsimple-virtual-array-deploy3-iscsi-setup/image4.png)
3. Vous accéderez à la page d’ **accueil** . Cette page décrit les différents paramètres requis pour configurer et inscrire l’appareil virtuel auprès du service StorSimple Manager. Notez que les **paramètres réseau**, **les paramètres du proxy web** et les **paramètres horaires** sont facultatifs. Les seuls paramètres obligatoires sont les **paramètres de l’appareil** et les **paramètres du cloud**.
   
    ![page d'accueil](./media/storsimple-virtual-array-deploy3-iscsi-setup/image5.png)
4. Dans la page **Paramètres réseau**, sous **Interfaces réseau**, DATA 0 sera automatiquement configuré pour vous. Chaque interface réseau est définie par défaut pour obtenir automatiquement une adresse IP (DHCP). Par conséquent, une adresse IP, un sous-réseau et une passerelle seront automatiquement attribués (pour IPv4 et IPv6).
   
    Si vous envisagez de déployer votre appareil en tant que serveur iSCSI (pour configurer un stockage en bloc), nous vous recommandons de désactiver l’option permettant d’ **obtenir automatiquement l’adresse IP** et de configurer des adresses IP statiques.
   
    ![page des paramètres réseau](./media/storsimple-virtual-array-deploy3-iscsi-setup/image6.png)
   
    Si vous avez ajouté plusieurs interfaces réseau lors de la configuration de l'appareil, vous pouvez les configurer ici. Notez que vous pouvez configurer votre interface réseau en IPv4 uniquement ou en IPv4 et IPv6. Les configurations en IPv6 uniquement ne sont pas prises en charge.
5. Des serveurs DNS sont requis car ils sont utilisés lorsque votre appareil tente de communiquer avec vos fournisseurs de services de stockage cloud ou pour résoudre votre appareil par nom s’il est configuré comme un serveur de fichiers. Dans la page **Paramètres réseau**, sous **Serveurs DNS** :
   
   1. Un serveur DNS principal et un serveur secondaire seront définis automatiquement. Si vous choisissez de configurer des adresses IP statiques, vous pouvez spécifier des serveurs DNS. Pour une haute disponibilité, nous vous recommandons de configurer un serveur DNS principal et un serveur secondaire.
   2. Cliquez sur **Appliquer**. Cette opération appliquera et validera les paramètres réseau.
6. Sur la page **Paramètres de l’appareil** :
   
   1. Attribuez un **nom** unique à votre appareil. Ce nom peut contenir 1 à 15 caractères ainsi que des lettres, des chiffres et des traits d'union.
   2. Cliquez sur l’icône **Serveur iSCSI**![icône Serveur iSCSI](./media/storsimple-virtual-array-deploy3-iscsi-setup/image7.png) pour le **type** d’appareil que vous créez. Un serveur iSCSI vous permettra de configurer un stockage de bloc.
   3. Indiquez si vous souhaitez associer cet appareil à un domaine. Si votre appareil est un serveur iSCSI, l’association à un domaine est facultative. Si vous décidez de ne pas associer votre serveur iSCSI à un domaine, cliquez sur **Appliquer**, attendez que les paramètres soient appliqués, puis passez à l'étape suivante.
      
       Si vous souhaitez associer l'appareil à un domaine. Entrez un **nom de domaine**, puis cliquez sur **Appliquer**.
      
      > [!NOTE]
      > Si vous joignez votre serveur iSCSI à un domaine, assurez-vous que votre tableau virtuel est dans sa propre unité organisationnelle (UO) pour Microsoft Azure Active Directory et qu’aucun objet de stratégie de groupe (GPO) ne lui est appliqué.
      > 
      > 
   4. Une boîte de dialogue s’affiche. Entrez vos informations d'identification de domaine au format spécifié. Cliquez sur l’icône de coche ![icône en forme de coche](./media/storsimple-virtual-array-deploy3-iscsi-setup/image15.png). Les informations d'identification de domaine seront vérifiées. Un message d'erreur apparaît si les informations d'identification sont incorrectes.
      
       ![credentials](./media/storsimple-virtual-array-deploy3-iscsi-setup/image8.png)
   5. Cliquez sur **Appliquer**. Cette opération appliquera et validera les paramètres de l’appareil.
7. (Facultatif) Configurez votre serveur proxy web. Bien que la configuration du proxy web soit facultative, si vous en utilisez un, vous pouvez uniquement le configurer ici.
   
    ![configuration du proxy web](./media/storsimple-virtual-array-deploy3-iscsi-setup/image9.png)
   
    Dans la page **Proxy web** :
   
   1. Indiquez l’**URL du proxy web** au format suivant : *http:\//adresse IP-hôte* ou *nom de domaine complet:numéro de port*. Notez que les URL HTTPS ne sont pas prises en charge.
   2. Définissez **Authentification** sur la valeur **De base** ou **Aucune**.
   3. Si vous utilisez une authentification, vous devez également fournir un **nom d’utilisateur** et un **mot de passe**.
   4. Cliquez sur **Appliquer**. Cette opération validera et appliquera les paramètres de proxy web configurés.
8. (Facultatif) Configurez les paramètres d'heure de votre appareil, notamment le fuseau horaire et les serveurs NTP principal et secondaire. Les serveurs NTP sont requis. En effet, votre appareil doit synchroniser les heures pour pouvoir s’authentifier auprès de vos fournisseurs de services cloud.
   
    ![paramètres horaires](./media/storsimple-virtual-array-deploy3-iscsi-setup/image10.png)
   
    Dans la page **Paramètres d’heure** :
   
   1. Dans la liste déroulante, définissez le **fuseau horaire** en fonction de l’emplacement géographique de l’appareil déployé. Le fuseau horaire par défaut de votre appareil est PST. Votre appareil utilise ce fuseau horaire pour toutes les opérations planifiées.
   2. Spécifiez un **serveur NTP principal** pour votre appareil ou acceptez la valeur par défaut time.windows.com. Vérifiez que votre réseau autorise le trafic NTP à passer de votre centre de données à Internet.
   3. (Facultatif) Spécifiez un **serveur NTP secondaire** pour votre appareil.
   4. Cliquez sur **Appliquer**. Cette opération validera et appliquera les paramètres d’heure configurés.
9. Configurez les paramètres de cloud de votre appareil. Dans le cadre de cette étape, vous allez effectuer la configuration de l’appareil local, puis inscrire l’appareil auprès de votre service StorSimple Device Manager.
   
   1. Entrez la **clé d'inscription au service** obtenue à l'**Étape 2 : Obtention de la clé d'inscription** dans [Déploiement de StorSimple Virtual Array - Préparation du portail](storsimple-virtual-array-deploy1-portal-prep.md#step-2-get-the-service-registration-key).
   2. S’il ne s’agit pas du premier appareil que vous inscrivez auprès de ce service, vous devrez fournir la **clé de chiffrement de données du service**. Cette clé et la clé d’inscription du service sont requises pour l’inscription d’appareils supplémentaires auprès du service StorSimple Device Manager. Pour plus d’informations, reportez-vous à la section permettant d’obtenir la [clé de chiffrement de données du service](storsimple-ova-web-ui-admin.md#get-the-service-data-encryption-key) sur votre interface utilisateur web locale.
   3. Cliquez sur **S'inscrire**. Cette opération redémarra l’appareil. Vous devrez peut-être attendre 2 à 3 minutes avant que l'appareil soit inscrit. Une fois que l’appareil a redémarré, vous êtes redirigé vers la page de connexion.
      
      ![enregistrer un appareil](./media/storsimple-virtual-array-deploy3-iscsi-setup/image11.png)
10. Revenez au Portail Azure.
11. Accédez au panneau **Appareils** de votre service. Si vous avez beaucoup de ressources, cliquez sur **Toutes les ressources**, cliquez sur votre nom de service (recherchez-le si nécessaire), puis cliquez sur **Appareils**.
12. Sur la page **Appareils** , vérifiez que l’appareil s’est bien connecté au service en vérifiant son état. L’état de l’appareil doit être **Prêt pour la configuration**.
    
    ![Déployer l’appareil](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis1m.png)

## <a name="step-2-configure-the-device-as-iscsi-server"></a>Étape 2 : Configurer l’appareil en tant que serveur iSCSI

Pour exécuter la configuration d’appareil requise, procédez comme suit dans le Portail Azure.

#### <a name="to-configure-the-device-as-iscsi-server"></a>Pour configurer l’appareil en tant que serveur iSCSI

1. Accédez à votre service StorSimple Device Manager, puis sélectionnez **Gestion > Appareils**. Dans le panneau **Appareils**, sélectionnez l’appareil que vous venez de créer. Cet appareil s’affiche avec l’état **Prêt pour la configuration**.
   
    ![Configurer l’appareil en tant que serveur iSCSI](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis1m.png) 
2. Cliquez sur l’appareil. Un message de bannière s’affiche pour vous indiquer que l’appareil est prêt pour la configuration.
   
    ![Configurer l’appareil en tant que serveur iSCSI 2](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis2m.png)  
3. Cliquez sur **Configurer** dans la barre de commandes de l’appareil. Le panneau **Configurer** s’affiche. Dans le panneau **Configurer**, procédez comme suit :
   
   * Le nom du serveur iSCSI est automatiquement renseigné.
   * Assurez-vous que le chiffrement du stockage cloud est défini sur **Activé**. Cela garantit que les données envoyées de l’appareil au cloud sont chiffrées.
   * Spécifiez une clé de chiffrement de 32 caractères et enregistrez-la dans une application de gestion des clés pour référence ultérieure.
   * Spécifiez un compte de stockage à utiliser avec votre appareil. Dans cet abonnement, vous pouvez sélectionnez un compte de stockage existant, ou cliquer sur **Ajouter** afin de choisir un compte d’un abonnement différent.
     
     ![Configurer l’appareil en tant que serveur iSCSI 3](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis4m.png)
4. Cliquez sur **Configurer** afin de terminer la configuration du serveur iSCSI.
   
    ![Configurer l’appareil en tant que serveur iSCSI 4](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis5m.png) 
5. Vous serez tenu informé de l’avancement de la création du serveur iSCSI. Une fois que le serveur iSCSI est créé, le panneau **Appareils** est mis à jour et l’état d’appareil correspondant est **En ligne**.
   
    ![Configurer l’appareil en tant que serveur iSCSI 5](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis9m.png)

## <a name="step-3-add-a-volume"></a>Étape 3 : Ajout d’un volume

1. Dans le panneau **Appareils**, sélectionnez l’appareil que vous venez de configurer en tant que serveur iSCSI. Cliquez sur **...** (sinon cliquez avec le bouton droit sur cette ligne), puis dans le menu contextuel, sélectionnez **Ajouter un volume**. Vous pouvez également cliquer sur **+ Ajouter un volume** dans la barre de commandes. Le panneau **Ajouter un volume** s’ouvre.
   
    ![Ajout d’un volume](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis10m.png)
2. Dans le panneau **Ajouter un volume**, procédez comme suit :
   
   * Dans le champ **Nom du volume**, saisissez un nom unique pour votre volume. Le nom doit être une chaîne contenant entre 3 et 127 caractères.
   * Dans la liste déroulante **Type**, spécifiez si vous souhaitez créer un volume **Hiérarchisé** ou **Attaché localement**. Pour les charges de travail qui nécessitent des garanties locales, une faible latence et les meilleures performances possibles, sélectionnez **Volume** **attaché localement**. Pour toutes les autres données, sélectionnez **Volume** **hiérarchisé**.
   * Dans le champ **Capacité**, spécifiez la taille du volume. Un volume hiérarchisé doit être compris entre 500 Go et 5 To, tandis qu’un volume attaché doit être compris entre 50 Go et 500 Go.
     
     La configuration d’un volume épinglé localement est complète, et garantit que les données principales sur le volume sont conservées sur l’appareil et ne débordent pas sur le cloud.
     
     La configuration d’un volume à plusieurs niveaux est légère. Lorsque vous créez un volume à plusieurs niveaux, environ 10 % de l'espace sont configurés au niveau local et 90 % dans le cloud. Par exemple, si vous avez configuré un volume de 1 To, 100 Go résident dans l'espace local et 900 Go sont utilisés dans le cloud lorsque les données sont stockées en niveaux. Cela implique que si vous n'avez plus d'espace local sur l’appareil, vous ne pouvez pas configurer un partage à plusieurs niveaux (car les 10 % ne seront pas disponibles).
     
     ![Ajouter un volume 2](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis12.png)
   * Cliquez sur **Hôtes connectés**, sélectionnez un enregistrement de contrôle d’accès correspondant à l’initiateur iSCSI que vous souhaitez connecter à ce volume, puis cliquez sur **Sélectionner**. <br><br> 
3. Pour ajouter un hôte connecté, cliquez sur **Ajouter nouveau**, saisissez un nom pour l’hôte et son nom complet iSCSI, puis cliquez sur **Ajouter**. Si vous ne possédez pas le nom qualifié, accédez à l'[Annexe A : Obtention du nom qualifié d’un hôte Windows Server](#appendix-a-get-the-iqn-of-a-windows-server-host).
   
      ![Ajouter un volume 3](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis15m.png)
4. Lorsque vous avez terminé de configurer votre volume, cliquez sur **OK**. Un volume est créé avec les paramètres spécifiés ; une notification s’affiche. Par défaut, la surveillance et la sauvegarde seront activées pour le volume.
   
     ![Ajouter un volume 4](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis18m.png)
5. Pour vérifier que le volume a été créé, accédez au panneau **Volumes**. Le volume devrait y apparaître.
   
   ![Ajouter un volume 5](./media/storsimple-virtual-array-deploy3-iscsi-setup/deployis20m.png)

## <a name="step-4-mount-initialize-and-format-a-volume"></a>Étape 4 : Monter, initialiser et formater un volume

Procédez comme suit pour monter, initialiser et formater vos volumes StorSimple sur un hôte Windows Server.

#### <a name="to-mount-initialize-and-format-a-volume"></a>Pour monter, initialiser et formater un volume

1. Ouvrez l’application de l’**initiateur iSCSI** sur le serveur approprié.
2. Dans la fenêtre **Propriétés de l’initiateur iSCSI**, sous l’onglet **Découverte**, cliquez sur **Découvrir un portail**.
   
    ![Découvrir un portail](./media/storsimple-virtual-array-deploy3-iscsi-setup/image22.png)
3. Dans la boîte de dialogue **Détecter un portail cible**, indiquez l’adresse IP de votre interface réseau compatible iSCSI, puis cliquez sur **OK**.
   
    ![Adresse IP](./media/storsimple-virtual-array-deploy3-iscsi-setup/image23.png)
4. Dans la fenêtre **Propriétés de l’initiateur iSCSI**, sous l’onglet **Cibles**, recherchez les **cibles découvertes**. (Chaque volume sera une cible découverte). L’appareil doit apparaître comme **inactif**.
   
    ![cibles découvertes](./media/storsimple-virtual-array-deploy3-iscsi-setup/image24.png)
5. Sélectionnez un appareil cible, puis cliquez sur **Connecter**. Une fois l’appareil connecté, son état doit indiquer **Connecté**. (Pour plus d’informations sur l’utilisation de l’initiateur Microsoft iSCSI, consultez [Installing and Configuring Microsoft iSCSI Initiator][1] (Installation et configuration de l’initiateur Microsoft iSCSI)).
   
    ![sélectionner un appareil cible](./media/storsimple-virtual-array-deploy3-iscsi-setup/image25.png)
6. Sur l’hôte Windows, appuyez sur la touche de logo Windows + X, puis cliquez sur **Exécuter**.
7. Dans la boîte de dialogue **Exécuter**, saisissez **Diskmgmt.msc**. Cliquez sur **OK**. La boîte de dialogue **Gestion des disques** s’affiche. Le volet de droite affiche les volumes de votre ordinateur hôte.
8. Dans la fenêtre **Gestion des disques** , les volumes montés sont affichés comme indiqué dans l’illustration suivante. Cliquez avec le bouton droit sur le volume détecté (cliquez sur le nom du disque), puis cliquez sur **En ligne**.
   
    ![Gestion des disques](./media/storsimple-virtual-array-deploy3-iscsi-setup/image26.png)
9. Cliquez avec le bouton droit et sélectionnez **Initialiser le disque**.
   
    ![initialiser le disque 1](./media/storsimple-virtual-array-deploy3-iscsi-setup/image27.png)
10. Dans la boîte de dialogue, sélectionnez le(s) disque(s) à initialiser, puis cliquez sur **OK**.
    
    ![initialiser le disque 2](./media/storsimple-virtual-array-deploy3-iscsi-setup/image28.png)
11. L'Assistant Nouveau volume simple se lance. Sélectionnez une taille de disque, puis cliquez sur **Suivant**.
    
    ![assistant nouveau volume 1](./media/storsimple-virtual-array-deploy3-iscsi-setup/image29.png)
12. Affectez une lettre de lecteur au volume, puis cliquez sur **Suivant**.
    
    ![assistant nouveau volume 2](./media/storsimple-virtual-array-deploy3-iscsi-setup/image30.png)
13. Entrez les paramètres pour formater le volume. **Sur Windows Server, seul le système NTFS est pris en charge.** Définissez la taille de l’unité d’allocation sur 64 Ko. Nommez votre volume. Il est recommandé de choisir un nom de volume identique à celui que vous avez fourni sur votre instance StorSimple Virtual Array. Cliquez sur **Suivant**.
    
    ![assistant nouveau volume 3](./media/storsimple-virtual-array-deploy3-iscsi-setup/image31.png)
14. Vérifiez les valeurs de votre volume, puis cliquez sur **Terminer**.
    
    ![assistant nouveau volume 4](./media/storsimple-virtual-array-deploy3-iscsi-setup/image32.png)
    
    Les volumes s’affichent en tant que **En ligne** on the **Gestion des disques** .
    
    ![volumes en ligne](./media/storsimple-virtual-array-deploy3-iscsi-setup/image33.png)

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [utiliser l’interface utilisateur web locale pour gérer votre StorSimple Virtual Array](storsimple-ova-web-ui-admin.md).

## <a name="appendix-a-get-the-iqn-of-a-windows-server-host"></a>Annexe A : Obtention du nom qualifié d’un hôte Windows Server

Procédez comme suit pour obtenir le nom qualifié iSCSI (IQN) d'un hôte Windows exécutant Windows Server 2012.

#### <a name="to-get-the-iqn-of-a-windows-host"></a>Pour obtenir le nom qualifié d’un hôte Windows

1. Démarrez l’initiateur Microsoft iSCSI sur l’hôte Windows.
2. Dans la fenêtre **Propriétés de l’initiateur iSCSI**, sous l’onglet **Configuration**, sélectionnez et copiez la chaîne affichée dans le champ **Nom de l’initiateur**.
   
    ![iSCSI Initiator Properties](./media/storsimple-virtual-array-deploy3-iscsi-setup/image34.png)
3. Enregistrez cette chaîne.

<!--Reference link-->
[1]: /previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/ee338480(v=ws.10)