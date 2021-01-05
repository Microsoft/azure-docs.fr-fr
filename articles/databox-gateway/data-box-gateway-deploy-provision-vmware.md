---
title: Didacticiel sur la configuration d’Azure Data Box Gateway dans VMware | Microsoft Docs
description: Le deuxième didacticiel concernant le déploiement d’Azure Data Box Gateway implique la configuration d’un appareil virtuel dans VMware.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: tutorial
ms.date: 10/16/2020
ms.author: alkohli
ms.openlocfilehash: dfbc63ccbdf6f0a8abe1e955b43240d799d6783d
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580300"
---
# <a name="tutorial-provision-azure-data-box-gateway-in-vmware"></a>Tutoriel : Provisionner Azure Data Box Gateway dans VMware

## <a name="overview"></a>Vue d’ensemble

Ce tutoriel explique comment provisionner une instance Data Box Gateway sur un système hôte exécutant VMware ESXi 6.0, 6.5 ou 6.7. 

Vous aurez besoin de privilèges d’administrateur pour configurer un appareil virtuel et vous y connecter. La configuration initiale peut prendre environ 10 minutes. 

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Vérifier que l’hôte répond à la configuration minimale requise de l’appareil
> * Approvisionner un appareil virtuel dans VMware
> * Démarrer l’appareil virtuel et obtenir l’adresse IP

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.


## <a name="prerequisites"></a>Prérequis

Vous trouverez ici les prérequis pour provisionner un appareil virtuel sur un système hôte exécutant VMware ESXi 6.0, 6.5 ou 6.7.

### <a name="for-the-data-box-gateway-resource"></a>Pour la ressource Data Box Gateway

Avant de commencer, assurez-vous que :

* Vous avez terminé toutes les étapes de la rubrique [Préparer le portail pour Data Box Gateway](data-box-gateway-deploy-prep.md).
* Vous avez téléchargé l’image d’appareil virtuel pour VMware à partir du portail Azure comme décrit dans [Préparation du portail pour Data Box Gateway](data-box-gateway-deploy-prep.md).

  > [!IMPORTANT]
  > Le logiciel en cours d’exécution sur Data Box Gateway peut seulement être utilisé avec la ressource Data Box Gateway.

### <a name="for-the-data-box-gateway-virtual-device"></a>Pour l’appareil virtuel Data Box Gateway

Avant de déployer un appareil virtuel, assurez-vous que :

* Vous avez accès à un système hôte exécutant VMware (ESXi 6.0, 6.5 ou 6.7) permettant de configurer un appareil.
* Le système hôte est en mesure de dédier les ressources suivantes pour configurer votre appareil virtuel :

  * Un minimum de 4 cœurs.
  * Au moins 8 Go de RAM.
  * Une interface réseau.
  * Un disque de système d’exploitation de 250 Go.
  * Un disque virtuel de 2 To pour les données système.

### <a name="for-the-network-in-datacenter"></a>Pour le réseau dans le centre de données

Avant de commencer :

- Passez en revue les exigences de mise en réseau pour déployer Data Box Gateway et configurer le réseau du centre de données conformément à la configuration requise. Pour plus d’informations, consultez la [Configuration requise du réseau pour Data Box Gateway](data-box-gateway-system-requirements.md#networking-port-requirements).
- Assurez-vous que la bande passante Internet minimale est de 20 Mbits/s pour un fonctionnement optimal de l’appareil.

## <a name="check-the-host-system"></a>Vérifier le système hôte

Pour créer un appareil virtuel, vous avez besoin des éléments suivants :

* Un accès à un système hôte exécutant VMware ESXi Server 6.0, 6.5 ou 6.7. Le système hôte est en mesure de dédier les ressources suivantes pour votre appareil virtuel :
 
  * Au moins 4 processeurs virtuels.
  * Au moins 8 Go de RAM. 
  * Une interface réseau connectée au réseau et capable d’acheminer le trafic vers Internet.
  * Un disque de système d’exploitation de 250 Go.
  * Un disque virtuel de 2 To pour les données.
* Un client VMware vSphere sur votre système pour gérer l'hôte ESXi.


## <a name="provision-a-virtual-device-in-hypervisor"></a>Configurer un appareil virtuel dans l’hyperviseur

Procédez comme suit pour configurer un appareil virtuel dans votre hyperviseur.

1. Copiez l'image de l’appareil virtuel sur votre système. Vous avez téléchargé cette image virtuelle (deux fichiers) via le portail Azure. Prenez note de l’emplacement où vous avez copié l’image car vous l’utiliserez plus loin dans la procédure.

2. Connectez-vous au serveur ESXi via un navigateur à l’URL suivante : `https://<IP address of the ESXi server>`. Vous devez disposer de privilèges d’administrateur pour créer une machine virtuelle.

   ![Page de connexion](./media/data-box-gateway-deploy-provision-vmware/image1.png)
  
3. Téléchargez le VMDK sur le serveur ESXi. Dans le volet de navigation, sélectionnez **Stockage**.

   ![Capture d’écran d’une page sur le site du serveur ESXi, montrant le volet Navigator avec l’option Storage sélectionnée.](./media/data-box-gateway-deploy-provision-vmware/image2.png)

4. Dans le panneau droit, sous **Magasins de données**, sélectionnez le magasin de données dans lequel vous voulez charger le fichier VMDK. 

    - Le magasin de données doit être de type VMFS5. 
    - Le magasin de données doit avoir suffisamment d'espace libre pour les disques du système d'exploitation et des données.
   
5. Cliquez avec le bouton droit et sélectionnez **Parcourir les magasins de données**.

   ![Parcourir les magasins de données](./media/data-box-gateway-deploy-provision-vmware/image3.png)

6. Une fenêtre **Navigateur du magasin de données** s’ouvre.

   ![Navigateur du magasin de données](./media/data-box-gateway-deploy-provision-vmware/image4.png)

7. Dans la barre d’outils, cliquez sur l’icône **Créer un répertoire** pour créer un dossier. Spécifiez et notez le nom du dossier. Vous utiliserez ce nom de dossier plus tard lors de la création d'une machine virtuelle (recommandée). Cliquez sur **Créer un répertoire**.

   ![Créer un répertoire](./media/data-box-gateway-deploy-provision-vmware/image5.png)

8. Le nouveau dossier s’affiche dans le panneau gauche du **Navigateur de magasins de données**. Cliquez sur l’icône de **chargement** et sélectionnez **Charger un fichier**.

    ![Charger le fichier](./media/data-box-gateway-deploy-provision-vmware/image6.png)

9. Recherchez et sélectionnez les fichiers VMDK que vous avez téléchargés. Il existe deux fichiers. Sélectionnez un fichier à télécharger.

    ![Sélectionner un fichier à charger](./media/data-box-gateway-deploy-provision-vmware/image7.png)

10. Cliquez sur **Ouvrir**. Ceci démarrera le téléchargement du fichier VMDK vers le magasin de données spécifié. Le téléchargement du fichier peut prendre plusieurs minutes.
11. Une fois le téléchargement terminé, le fichier apparaît dans le magasin de données, dans le dossier que vous avez créé. Vous devez maintenant télécharger le deuxième fichier VMDK dans le même magasin de données. Une fois les deux fichiers chargés, ils sont fusionnés en un seul fichier. Vous voyez ensuite un seul fichier dans le répertoire.

    ![Deux fichiers VMDK sont fusionnés en un seul fichier](./media/data-box-gateway-deploy-provision-vmware/image8.png)

12. Revenez à la fenêtre du client vSphere. Dans le volet de navigation, sélectionnez **Machines virtuelles**. Dans le volet droit, cliquez sur **Créer/Inscrire une machine virtuelle**.

    ![Créer ou inscrire une machine virtuelle](./media/data-box-gateway-deploy-provision-vmware/image9.png)

13. Une **nouvelle machine virtuelle** apparaît. Sous Sélectionner un type de création, choisissez **Créer une machine virtuelle** et cliquez sur **Suivant**.
    ![Page Sélectionner un type de création](./media/data-box-gateway-deploy-provision-vmware/image10.png)

14. Sur la page **Sélectionner un nom, un nom de système d’exploitation et un emplacement**, spécifiez le **nom** de votre machine virtuelle. Ce nom doit correspondre au nom du dossier (meilleure pratique recommandée) spécifié à l'étape 7. Choisissez Windows comme **Famille de système d’exploitation invité** et Microsoft Windows Server 2016 (64 bits) comme **Version du système d’exploitation invité**. Cliquez sur **Suivant**.

    ![Page de sélection d’un nom, d’un nom de système d’exploitation et d’un emplacement](./media/data-box-gateway-deploy-provision-vmware/image11.png)

15. Sur la page **Sélectionner un stockage**, sélectionnez le magasin de données que vous souhaitez utiliser pour configurer votre machine virtuelle. Cliquez sur **Suivant**.

    ![Page Sélectionner un stockage](./media/data-box-gateway-deploy-provision-vmware/image12.png)
16. Sur la page **Personnaliser les paramètres**, définissez le **processeur** sur 4, **Mémoire** sur 8192 Mo (ou plus), **Disque dur 1** sur 2 To (ou plus). Choisissez le **disque dur SCSI** à ajouter. Dans ce cas, il s’agit de LSI Logic SAS. **Les disques durs IDE statiques ne sont pas pris en charge.** Le **disque dur 1** est le disque de données virtuel. Notez que vous ne pouvez pas réduire le disque une fois configuré. Toute tentative de réduction du disque entraîne la perte de toutes les données locales sur l’appareil. 

    ![Page Personnaliser les paramètres](./media/data-box-gateway-deploy-provision-vmware/image13.png)

    Dans la même page, cliquez sur **Ajouter le disque dur**, puis sélectionnez **Disque dur existant**. Sélectionnez le fichier VMDK dans le magasin de données. Cette opération ajoute un disque de système d’exploitation. 

     ![Personnaliser les paramètres - Ajouter un disque dur](./media/data-box-gateway-deploy-provision-vmware/customize-settings-add-hard-disk.png)

    Défilez vers le bas jusqu'à voir le **Nouveau disque dur** et développez-le pour afficher les paramètres. Définissez le **Nœud d’appareil virtuel** sur **contrôleur IDE 0**.

     ![Personnaliser les paramètres - Configurer un nouveau disque dur](./media/data-box-gateway-deploy-provision-vmware/customize-settings-new-disk.png)

17. (Facultatif) *Effectuez cette étape uniquement si vous exécutez VMware ESXi Server 6.7*. Dans la page **Customize settings** (Personnaliser les paramètres), cliquez sur **VM options** (Options de machine virtuelle). Accédez à **Boot options > Firmware**  (Options de démarrage > Microprogramme), puis remplacez la valeur existante par **BIOS**. Par défaut, la valeur est EFI. Cliquez sur **Suivant**.

    ![Page Personnaliser des paramètres si exécution de VMware ESXi Server 6.7](./media/data-box-gateway-deploy-provision-vmware/customize-settings-new-disk-esxi.png)

18. Sur la page **Prêt à finaliser** , passez en revue tous les paramètres associés à la nouvelle machine virtuelle. Vérifiez que le processeur est configuré sur 4, la mémoire sur 8192 Mo, l’interface réseau sur 1 et que le disque dur 2 possède un contrôleur IDE 0. Cliquez sur **Terminer**.
   
    ![Page Prêt pour l’exécution](./media/data-box-gateway-deploy-provision-vmware/image16.png)
    ![Page Prêt pour l’exécution 2](./media/data-box-gateway-deploy-provision-vmware/image17.png)

Votre machine virtuelle est désormais configurée. Vous voyez une notification et la machine virtuelle est ajoutée à la liste des machines virtuelles.

![Nouvelle machine virtuelle ajoutée à la liste des machines virtuelles](./media/data-box-gateway-deploy-provision-vmware/image17.png)

L’étape suivante consiste à mettre cette machine virtuelle sous tension et à obtenir l’adresse IP.

> [!NOTE]
> Nous vous recommandons de ne pas installer les outils VMware sur votre appareil virtuel (configuré plus haut). Cette configuration avec les outils VMware installés n’est pas prise en charge.

## <a name="start-the-virtual-device-and-get-the-ip"></a>Démarrer l’appareil virtuel et obtenir l’adresse IP

Procédez comme suit pour démarrer votre appareil virtuel et vous y connecter.

#### <a name="to-start-the-virtual-device"></a>Pour démarrer l’appareil virtuel
1. Démarrez l’appareil virtuel. Dans le volet droit, sélectionnez votre appareil dans la liste des machines virtuelles et faites un clic droit pour afficher le menu contextuel. Sélectionnez **Alimentation** puis **Mise sous tension**. Cette opération met votre machine virtuelle sous tension. Vous pouvez afficher l'état de l’opération dans le volet inférieur du client web.

    ![Mettre sous tension de l’appareil virtuel](./media/data-box-gateway-deploy-provision-vmware/image19.png)

2. Sélectionnez de nouveau votre machine virtuelle. Faites un clic droit et sélectionnez **Console**, puis **Ouvrir dans une nouvelle fenêtre**.

    ![Ouvrir la console de l’appareil virtuel](./media/data-box-gateway-deploy-provision-vmware/image20.png)

3. La console de machine virtuelle s’ouvre dans une nouvelle fenêtre. 

    ![Console de l’appareil virtuel](./media/data-box-gateway-deploy-provision-vmware/image21.png)

4. Une fois que l’appareil est en marche, cliquez sur l’onglet dans la partie centrale supérieure de la fenêtre de console. Sélectionnez **Système d’exploitation invité > Envoyer des clés > Ctrl + Alt + Suppr**. La machine virtuelle est alors déverrouillée.

   ![Déverrouiller l’appareil virtuel](./media/data-box-gateway-deploy-provision-vmware/image22.png)

5. Fournissez le mot de passe pour se connecter à la machine. Le mot de passe par défaut est *Password1*.

   ![Entrer le mot de passe de l’appareil virtuel](./media/data-box-gateway-deploy-provision-vmware/image23.png)

6. Les étapes 6 à 8 s’appliquent uniquement lors de l’amorçage dans un environnement non DHCP. Si vous êtes dans un environnement DHCP, ignorez ces étapes et passez à l'étape 9. Si vous avez démarré votre appareil dans un environnement non DHCP, un message s’affiche : **Utilisez l’applet de commande Set-HcsIPAddress pour configurer le réseau**. 
   
7. Pour configurer le réseau, dans l’invite de commandes, utilisez la commande `Get-HcsIpAddress` pour répertorier les interfaces réseau activées sur votre appareil virtuel. Si votre appareil possède une seule interface réseau activée, le nom par défaut affecté à cette interface est `Ethernet`.

8. Utilisez l’applet de commande `Set-HcsIpAddress` pour configurer le réseau. Voici un exemple :

    `Set-HcsIpAddress –Name Ethernet0 –IpAddress 10.161.22.90 –Netmask 255.255.255.0 –Gateway 10.161.22.1`

9. Une fois l'installation initiale terminée et l’appareil démarré, vous verrez le texte de la bannière de l’appareil. Notez l'adresse IP et l'URL affichées dans le texte de la bannière pour gérer l'appareil. Vous utiliserez cette adresse IP pour vous connecter à l'interface utilisateur web de votre appareil virtuel et pour finaliser la configuration et l’activation locales.

   ![Texte de la bannière et URL de connexion pour l’appareil virtuel](./media/data-box-gateway-deploy-provision-vmware/image24.png)

Si votre périphérique ne répond pas à la configuration minimale requise, une erreur apparaît dans le texte de bannière (voir ci-dessous). Vous devez modifier la configuration de l'appareil afin qu'il dispose des ressources nécessaires à la configuration minimale. Vous pouvez ensuite redémarrer et vous connecter à l'appareil. Reportez-vous à la configuration minimale requise dans [Vérifier le système hôte](#check-the-host-system).

SI vous rencontrez une autre erreur durant la configuration initiale effectuée avec l’interface utilisateur web locale, reportez-vous aux workflows suivants :

- [Exécutez les tests de diagnostic pour dépanner la configuration de l’interface utilisateur web](data-box-gateway-troubleshoot.md#run-diagnostics).
- [Générez un package de journaux et affichez les fichiers journaux](data-box-gateway-troubleshoot.md#collect-support-package).

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez approfondi vos connaissances sur Data Box Gateway et avez notamment appris à :

> [!div class="checklist"]
> * Vérifier que l’hôte répond à la configuration minimale requise de l’appareil
> * Approvisionner un appareil virtuel dans VMware
> * Démarrer l’appareil virtuel et obtenir l’adresse IP

Passez au tutoriel suivant pour savoir comment connecter, configurer et activer votre appareil virtuel.

* [Configurer et se connecter aux partages sur votre Data Box Gateway](data-box-gateway-deploy-connect-setup-activate.md)

