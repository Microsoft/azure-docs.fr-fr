---
title: Configurer StorSimple Virtual Array comme un serveur de fichiers | Microsoft Docs
description: Ce troisième didacticiel du déploiement StorSimple Virtual Array vous explique comment configurer un appareil virtuel en tant que serveur de fichiers.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: f609f6ff-0927-48bb-a68a-6d8985d2fe34
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 07/25/2019
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 244fdbf7cb723fe85e0987d176a13242f0bff064
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005926"
---
# <a name="deploy-storsimple-virtual-array---set-up-as-file-server-via-azure-portal"></a>Déploiement de StorSimple Virtual Array - Configuration comme un serveur de fichiers via le portail Azure
![Diagramme montrant les étapes nécessaires au déploiement d’un tableau virtuel. La première partie de la troisième étape est intitulée « Configurer en tant que serveur de fichiers » et est mise en évidence.](./media/storsimple-virtual-array-deploy3-fs-setup/fileserver4.png)

## <a name="introduction"></a>Introduction

[!INCLUDE [storsimple-virtual-array-eol-banner](../../includes/storsimple-virtual-array-eol-banner.md)]

Cet article décrit comment effectuer la configuration initiale, inscrire votre serveur de fichiers StorSimple, finaliser la configuration de l’appareil, créer des partages SMB et s’y connecter. Il s’agit du dernier article de la série de didacticiels sur le déploiement nécessaires pour déployer complètement votre tableau virtuel en tant que serveur de fichiers ou serveur iSCSI.

Le processus d’installation et de configuration peut prendre jusqu’à 10 minutes. Les informations fournies dans cet article s’appliquent uniquement au déploiement de la solution StorSimple Virtual Array. Pour le déploiement d'appareils StorSimple 8000, accédez à : [Déployer un appareil StorSimple 8000 exécutant Update 2](./storsimple-8000-deployment-walkthrough-u2.md).

## <a name="setup-prerequisites"></a>Configuration requise
Avant de configurer votre solution StorSimple Virtual Array, assurez-vous que :

* Vous avez approvisionné une baie virtuelle et vous vous y êtes connecté comme indiqué dans l’article [Configurer StorSimple Virtual Array dans Hyper-V](storsimple-virtual-array-deploy2-provision-hyperv.md) ou [Configurer StorSimple Virtual Array dans VMware](storsimple-virtual-array-deploy2-provision-vmware.md).
* Vous disposez de la clé d’inscription du service StorSimple Device Manager que vous avez créé pour gérer les baies StorSimple Virtual Array. Pour plus d'informations, consultez [Étape 2 : Obtention de la clé d'inscription](storsimple-virtual-array-deploy1-portal-prep.md#step-2-get-the-service-registration-key) pour StorSimple Virtual Array.
* S’il s’agit de la deuxième baie virtuelle (ou d’une baie ultérieure) que vous inscrivez auprès d’un service StorSimple Device Manager existant, vous devez disposer de la clé de chiffrement des données du service. Cette clé a été générée lorsque le premier appareil a été correctement inscrit auprès de ce service. Si vous avez perdu cette clé, consultez la rubrique [Obtenir la clé de chiffrement des données de service](storsimple-ova-web-ui-admin.md#get-the-service-data-encryption-key) pour votre StorSimple Virtual Array.

## <a name="step-by-step-setup"></a>Configuration étape par étape
Utilisez la procédure détaillée ci-dessous pour configurer votre solution StorSimple Virtual Array.

## <a name="step-1-complete-the-local-web-ui-setup-and-register-your-device"></a>Étape 1 : Finalisation de la configuration de l'interface utilisateur web locale et inscription de votre appareil
#### <a name="to-complete-the-setup-and-register-the-device"></a>Pour finaliser l'installation et l’inscription de l'appareil
1. Ouvrez une fenêtre de navigateur et connectez-vous à l’interface utilisateur web locale. Tapez :
   
   `https://<ip-address of network interface>`
   
   Utilisez l'URL de connexion notée à l'étape précédente. Une erreur vous indique qu’il existe un problème avec le certificat de sécurité du site web. Cliquez sur **Poursuivre sur cette page web**.
   
   ![Capture d’écran d’une fenêtre de navigateur signalant un problème de certificat de sécurité. Deux liens sont visibles : l’un pour le site web et l’autre pour la page d’accueil de l’utilisateur.](./media/storsimple-virtual-array-deploy3-fs-setup/image2.png)
2. Connectez-vous à l’interface utilisateur web de votre baie virtuelle sous le nom **StorSimpleAdmin**. Entrez le mot de passe de l’administrateur de l’appareil que vous avez modifié à l’étape 3 relative au démarrage de la baie virtuelle dans l’article [Configurer StorSimple Virtual Array dans Hyper-V](storsimple-virtual-array-deploy2-provision-hyperv.md) ou [Configurer StorSimple Virtual Array dans VMware](storsimple-virtual-array-deploy2-provision-vmware.md).
   
   ![Capture d’écran de la page de connexion StorSimple Le nom d’utilisateur StorSimpleAdmin est visible, et une zone de mot de passe contient des caractères indéterminés.](./media/storsimple-virtual-array-deploy3-fs-setup/image3.png)
3. Vous accédez alors à la page **Accueil**. Cette page décrit les différents paramètres requis pour configurer et inscrire la baie virtuelle auprès du service StorSimple Device Manager. Les **paramètres réseau**, les **paramètres de proxy web** et les **paramètres de l’heure** sont facultatifs. Les seuls paramètres obligatoires sont les **paramètres de l’appareil** et les **paramètres du cloud**.
   
   ![Capture d’écran de la page d’accueil. Texte indiquant que l’appareil n’est pas configuré. Des liens vers différents types de paramètres sont visibles.](./media/storsimple-virtual-array-deploy3-fs-setup/image4.png)
4. Dans la page **Paramètres réseau**, sous **Interfaces réseau**, DATA 0 sera automatiquement configuré pour vous. Chaque interface réseau est définie par défaut pour obtenir automatiquement l’adresse IP (DHCP). Par conséquent, une adresse IP, un sous-réseau et une passerelle sont automatiquement attribués (pour IPv4 et IPv6).
   
   ![Capture d’écran de la page Paramètres réseau montrant les adresses IP qui sont configurées pour différentes versions du protocole Internet.](./media/storsimple-virtual-array-deploy3-fs-setup/image5.png)
   
   Si vous avez ajouté plusieurs interfaces réseau lors de la configuration de l'appareil, vous pouvez les configurer ici. Notez que vous pouvez configurer votre interface réseau en IPv4 uniquement ou en IPv4 et IPv6. Les configurations en IPv6 uniquement ne sont pas prises en charge.
5. Des serveurs DNS sont requis car ils sont utilisés lorsque votre appareil tente de communiquer avec vos fournisseurs de services de stockage cloud ou pour résoudre votre appareil par nom lorsqu’il configuré comme un serveur de fichiers. Dans la page **Paramètres réseau**, sous **Serveurs DNS** :
   
   1. Un serveur DNS principal et un serveur secondaire sont automatiquement configurés. Si vous choisissez de configurer des adresses IP statiques, vous pouvez spécifier des serveurs DNS. Pour une haute disponibilité, nous vous recommandons de configurer un serveur DNS principal et un serveur secondaire.
   2. Cliquez sur **Appliquer** pour appliquer et valider les paramètres réseau.
6. Sur la page **Paramètres de l’appareil** :
   
   1. Attribuez un **nom** unique à votre appareil. Ce nom peut contenir 1 à 15 caractères ainsi que des lettres, des chiffres et des traits d'union.
   2. Cliquez sur l’icône **Serveur de fichiers**:::image type="icon" source="./media/storsimple-virtual-array-deploy3-fs-setup/image6.png"::: pour le **type** d’appareil que vous créez. Un serveur de fichiers vous permettra de créer des dossiers partagés.
   3. Votre appareil étant un serveur de fichiers, vous devez l’associer à un domaine. Entrez un **nom de domaine**.
   4. Cliquez sur **Appliquer**.
7. Une boîte de dialogue s’affiche. Entrez vos informations d'identification de domaine au format spécifié. Cliquez sur l’icône en forme de coche. Les informations d’identification de domaine sont vérifiées. Si les informations d’identification sont incorrectes, un message d’erreur apparaît.
   
   ![Capture d’écran montrant une boîte de dialogue avec le nom d’utilisateur et le mot de passe remplis.](./media/storsimple-virtual-array-deploy3-fs-setup/image7.png)
8. Cliquez sur **Appliquer**. Cette opération appliquera et validera les paramètres de l’appareil.
   
   ![Capture d’écran de la page Paramètres de l’appareil. Le nom de l’appareil et le nom de domaine sont renseignés.](./media/storsimple-virtual-array-deploy3-fs-setup/image8.png)
   
   > [!NOTE]
   > Assurez-vous que votre tableau virtuel est dans sa propre unité organisationnelle (UO) pour Active Directory et qu'aucun objet de stratégie de groupe (GPO) ne lui est appliqué ou qu’il n’en a pas hérité. La stratégie de groupe peut éventuellement installer des applications telles qu’un logiciel antivirus sur le StorSimple Virtual Array. L’installation de logiciels supplémentaires n’est pas prise en charge et peut entraîner une altération des données. 
   > 
   > 
9. (Facultatif) Configurez votre serveur proxy web. Bien que la configuration du proxy web soit facultative, si vous en utilisez un, vous pouvez uniquement le configurer ici.
   
   ![Capture d’écran de la page Paramètres du proxy web. L’option Activer le proxy web est désactivée et l’authentification est définie sur Aucune. Aucune autre valeur n’est définie.](./media/storsimple-virtual-array-deploy3-fs-setup/image9.png)
   
   Sur la page **Proxy Web** :
   
   1. Indiquez l'**URL du proxy web** au format suivant : *http://&lt;adresse IP ou nom de domaine complet de l'hôte&gt;:numéro de port*. Notez que les URL HTTPS ne sont pas prises en charge.
   2. Définissez **Authentification** sur la valeur **De base** ou **Aucune**.
   3. Si vous utilisez une authentification, vous devrez également fournir un **nom d’utilisateur** et un **mot de passe**.
   4. Cliquez sur **Appliquer**. Cette opération validera et appliquera les paramètres de proxy web configurés.
10. (Facultatif) Configurez les paramètres d'heure de votre appareil, notamment le fuseau horaire et les serveurs NTP principal et secondaire. Les serveurs NTP sont requis. En effet, votre appareil doit synchroniser les heures pour pouvoir s’authentifier auprès de vos fournisseurs de services cloud.
    
    ![Capture d’écran de la page Paramètres d’heure. Le fuseau horaire et le serveur NTP principal sont renseignés. Le serveur NTP secondaire est vide.](./media/storsimple-virtual-array-deploy3-fs-setup/image10.png)
    
    Sur la page **Paramètres d’heure** :
    
    1. Dans la liste déroulante, définissez le **fuseau horaire** en fonction de l’emplacement géographique de l’appareil déployé. Le fuseau horaire par défaut de votre appareil est PST. Votre appareil utilise ce fuseau horaire pour toutes les opérations planifiées.
    2. Spécifiez un **serveur NTP principal** pour votre appareil ou acceptez la valeur par défaut time.windows.com. Vérifiez que votre réseau autorise le trafic NTP à passer de votre centre de données à Internet.
    3. (Facultatif) Spécifiez un **serveur NTP secondaire** pour votre appareil.
    4. Cliquez sur **Appliquer**. Cette opération validera et appliquera les paramètres d’heure configurés.
11. Configurez les paramètres de cloud de votre appareil. Dans le cadre de cette étape, vous allez effectuer la configuration de l’appareil local, puis inscrire l’appareil auprès de votre service StorSimple Device Manager.
    
    1. Entrez la **clé d'inscription au service** obtenue à l'[Étape 2 : Obtention de la clé d'inscription](storsimple-virtual-array-deploy1-portal-prep.md#step-2-get-the-service-registration-key) pour StorSimple Virtual Array.
    2. S’il s’agit du premier appareil que vous inscrivez auprès de ce service, la **clé de chiffrement des données du service** vous est présentée. Copiez-la et enregistrez-la en lieu sûr. Cette clé et la clé d’inscription du service sont requises pour l’inscription d’appareils supplémentaires auprès du service StorSimple Device Manager. 
       
       S’il ne s’agit pas du premier appareil que vous inscrivez auprès de ce service, vous devrez fournir la clé de chiffrement des données du service. Pour plus d’informations, reportez-vous à la section permettant d’obtenir la [clé de chiffrement de données du service](storsimple-ova-web-ui-admin.md#get-the-service-data-encryption-key) sur votre interface utilisateur web locale.
    3. Cliquez sur **S'inscrire**. Cette opération redémarra l’appareil. Vous devrez peut-être attendre 2 à 3 minutes avant que l'appareil soit inscrit. Une fois que l’appareil a redémarré, vous êtes redirigé vers la page de connexion.
       
       ![Capture d’écran de la page Paramètres du cloud. Les zones Clé d’inscription et Clé de chiffrement sont renseignées, mais les valeurs sont censurées.](./media/storsimple-virtual-array-deploy3-fs-setup/image13.png)
12. Revenez au Portail Azure. Sélectionnez **Toutes les ressources**, puis recherchez votre service StorSimple Device Manager.
    
    ![Capture d’écran de la page Toutes les ressources du portail Azure. Un service Gestionnaire de périphériques est mis en évidence.](./media/storsimple-virtual-array-deploy3-fs-setup/searchdevicemanagerservice1.png) 
13. Dans la liste filtrée, sélectionnez votre service StorSimple Device Manager, puis accédez à **Gestion > Appareils**. Dans le panneau **Appareils**, vérifiez que l’appareil s’est correctement connecté au service et qu’il présente l’état **Prêt pour la configuration**.
    
    ![Déployer](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs2m.png)

## <a name="step-2-configure-the-device-as-file-server"></a>Étape 2 : Configurer l'appareil en tant que serveur de fichiers
Pour exécuter la configuration d’appareil requise, procédez comme suit dans le [Portail Azure](https://portal.azure.com/) .

#### <a name="to-configure-the-device-as-file-server"></a>Pour configurer l’appareil en tant que serveur de fichiers
1. Accédez à votre service StorSimple Device Manager, puis sélectionnez **Gestion > Appareils**. Dans le panneau **Appareils**, sélectionnez l’appareil que vous venez de créer. Cet appareil s’affiche avec l’état **Prêt pour la configuration**.
   
   ![Configurer un serveur de fichiers](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs2m.png) 
2. Cliquez sur l’appareil. Un message de bannière s’affiche pour vous indiquer que l’appareil est prêt pour la configuration.
   
    ![Configurer un serveur de fichiers 2](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs3m.png)
3. Cliquez sur **Configurer** dans la barre de commandes. Le panneau **Configurer** s’affiche. Dans le panneau **Configurer**, procédez comme suit :
   
   1. Le nom du serveur de fichiers est automatiquement renseigné.
    
   2. Assurez-vous que le chiffrement du stockage cloud est défini sur **Activé**. Cette fonctionnalité chiffre toutes les données qui sont envoyées au cloud. 
    
   3. Une clé AES de 256 bits est utilisée avec la clé définie par l’utilisateur pour le chiffrement. Spécifiez une clé de 32 caractères, puis entrez de nouveau la clé pour la confirmer. Enregistrez la clé dans une application de gestion de clés à des fins de référence ultérieure.
    
   4. Cliquez sur **Configurer les paramètres requis** pour spécifier les informations d’identification du compte de stockage à utiliser avec votre appareil. Si aucune information d’identification du compte de stockage n’est configurée, cliquez sur **Ajouter nouveau**. **Vérifiez que le compte de stockage que vous utilisez prend en charge les objets blob de blocs. Les objets blob de pages ne sont pas pris en charge.** En savoir plus sur [les objets blob de blocs et de pages](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs).
   
      ![Configurer un serveur de fichiers 3](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs6m.png) 
4. Dans le panneau **Ajouter les informations d’identification d’un compte de stockage**, procédez comme suit : 

    1. Si le compte de stockage figure dans le même abonnement que le service, choisissez l’abonnement actuel. Si le compte de stockage ne se trouve pas dans le même abonnement, spécifiez un autre abonnement. 
    
    2. Dans la liste déroulante, choisissez un compte de stockage existant. 
    
    3. L’emplacement est automatiquement renseigné en fonction du compte de stockage spécifié. 
    
    4. Activez le protocole TLS pour garantir un canal de communication réseau sécurisé entre l’appareil et le cloud.
    
    5. Cliquez sur **Ajouter** pour ajouter ces informations d’identification du compte de stockage. 
   
        ![Configurer un serveur de fichiers 4](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs8m.png)

5. Une fois les informations d’identification du compte de stockage correctement créées, le panneau **Configurer** est actualisé afin d’afficher les informations d’identification spécifiées. Cliquez sur **Configurer**.
   
   ![Configurer un serveur de fichiers 5](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs11m.png)
   
   Vous pouvez constater qu’un serveur de fichiers est en cours de création. Une fois le serveur créé, vous en êtes informé par un message.
   
   ![Configurer un serveur de fichiers 5b](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs13m.png)
   
   L’appareil passe à l’état **En ligne**.
   
   ![Configurer un serveur de fichiers 5c](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs14m.png)
   
   Vous pouvez poursuivre la procédure par l’ajout d’un partage.

## <a name="step-3-add-a-share"></a>Étape 3 : Ajouter un partage
Pour créer un partage, procédez comme suit dans le [Portail Azure](https://portal.azure.com/).

#### <a name="to-create-a-share"></a>Pour créer un partage
1. Sélectionnez l’appareil que vous avez configuré en tant que serveur de fichiers à l’étape précédente, puis cliquez sur **...** (ou cliquez avec le bouton droit). Dans le menu contextuel, sélectionnez **Ajouter un partage**. Une autre possibilité consiste à cliquer sur **+ Ajouter un partage** dans la barre de commandes de l’appareil.
   
   ![Ajouter un partage](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs15m.png)
2. Spécifiez les paramètres de partage suivants :

   1. Nom unique pour votre partage. Le nom doit être une chaîne contenant entre 3 et 127 caractères.
    
   2. Une **Description** facultative pour le partage. La description permet d'identifier les propriétaires du partage.
    
   3. **Type** du partage. Le partage peut présenter le type **Hiérarchisé** (valeur par défaut) ou **Attaché localement**. Pour les charges de travail qui nécessitent des garanties locales, une faible latence et les meilleures performances possibles, sélectionnez un partage **épinglé localement** . Pour toutes les autres données, sélectionnez un partage **à plusieurs niveaux** .
      La configuration d’un partage épinglé localement est complète, et garantit que les données principales sur le partage sont conservées en local sur l’appareil et ne débordent pas sur le cloud. D’autre part, la configuration d’un partage à plusieurs niveaux est légère. Lorsque vous créez un partage à plusieurs niveaux, 10 % de l'espace sont configurés au niveau local et 90 % dans le cloud. Par exemple, si vous avez configuré un volume de 1 To, 100 Go résident dans l'espace local et 900 Go sont utilisés dans le cloud lorsque les données sont stockées en niveaux. Cela implique que si vous n'avez plus d'espace local sur l’appareil, vous ne pouvez pas configurer un partage à plusieurs niveaux.
   
   4. Dans le champ **Affecter une autorisation d’accès total par défaut**, attribuez les autorisations à l’utilisateur ou au groupe qui doivent accéder à ce partage. Spécifiez le nom de l’utilisateur ou du groupe d’utilisateurs sous la forme *john\@contoso.com*. Nous vous recommandons d'utiliser un groupe d'utilisateurs (plutôt qu'un seul utilisateur) lorsque vous accordez des privilèges d'administrateur pour accéder à ces partages. Après avoir attribué les autorisations à cet emplacement, vous pouvez utiliser l’Explorateur de fichiers par la suite pour modifier ces autorisations.
   
   5. Cliquez sur **Ajouter** pour créer le partage. 
    
       ![Ajouter un partage 1](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs18m.png)
   
       Vous êtes averti que le partage est en cours de création.
   
       ![Ajouter un partage 2](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs19m.png)
   
      Une fois le partage créé avec les paramètres spécifiés, le panneau **Partages** est actualisé pour refléter le nouveau partage. Par défaut, la surveillance et la sauvegarde sont activées pour le partage.
   
      ![Ajouter un partage 3](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs22m.png)

## <a name="step-4-connect-to-the-share"></a>Étape 4 : Se connecter au partage
Vous devez à présent vous connecter aux partages que vous avez créés à l’étape précédente. Exécutez cette procédure sur votre hôte Windows Server connecté à votre baie StorSimple Virtual Array.

#### <a name="to-connect-to-the-share"></a>Pour vous connecter au partage
1. Appuyez sur :::image type="icon" source="./media/storsimple-virtual-array-deploy3-fs-setup/image22.png"::: + R. Dans la fenêtre Exécuter, spécifiez *&#92;&#92;&lt;nom du serveur de fichiers&gt;* comme chemin d’accès en remplaçant *nom du serveur de fichiers* par le nom de l’appareil que vous avez attribué à votre serveur de fichiers. Cliquez sur **OK**.
   
   ![Capture d’écran de la boîte de dialogue Exécuter. La zone Ouvrir est renseignée par un chemin d’accès menant au serveur de fichiers.](./media/storsimple-virtual-array-deploy3-fs-setup/image23.png)
2. L’Explorateur de fichiers s’affiche. Vous devriez maintenant voir les partages que vous avez créés sous forme de dossiers. Sélectionnez et double-cliquez sur un partage (dossier) pour afficher son contenu.
   
   ![Capture d’écran de l’Explorateur de fichiers. Plusieurs dossiers sont visibles et représentent les partages nouvellement créés.](./media/storsimple-virtual-array-deploy3-fs-setup/image24.png)
3. Vous pouvez maintenant ajouter des fichiers à ces partages et effectuer une sauvegarde.

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [utiliser l’interface utilisateur web locale pour gérer votre StorSimple Virtual Array](storsimple-ova-web-ui-admin.md).