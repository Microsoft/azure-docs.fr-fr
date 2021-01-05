---
title: Didacticiel sur la configuration d’Azure Data Box Gateway dans Hyper-V | Microsoft Docs
description: Le deuxième didacticiel concernant le déploiement d’Azure Data Box Gateway implique la configuration d’un appareil virtuel dans Hyper-V.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: alkohli
ms.openlocfilehash: ffbfd3214242d8df5fe33faf465bc1da3eb9986d
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580252"
---
# <a name="tutorial-provision-azure-data-box-gateway-in-hyper-v"></a>Tutoriel : Provisionner Azure Data Box Gateway dans Hyper-V

## <a name="overview"></a>Vue d’ensemble

Ce didacticiel décrit comment configurer un Data Box Gateway sur un système hôte exécutant Hyper-V sur Windows Server 2016, Windows Server 2012 R2 ou Windows Server 2012.

Vous avez besoin de privilèges d’administrateur pour approvisionner et configurer un appareil virtuel. La configuration initiale peut prendre environ 10 minutes. 

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Vérifier que l’hôte répond à la configuration minimale requise de l’appareil
> * Configurer un appareil virtuel dans l’hyperviseur
> * Démarrer l’appareil virtuel et obtenir l’adresse IP

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Les conditions requises pour la configuration d’un appareil virtuel sur un système hôte exécutant Hyper-V sur Windows Server 2016 ou Windows Server 2012 R2 sont les suivantes.

### <a name="for-the-data-box-gateway-resource"></a>Pour la ressource Data Box Gateway

Avant de commencer, assurez-vous que :

* Vous avez terminé toutes les étapes de la rubrique [Préparer le portail pour Data Box Gateway](data-box-gateway-deploy-prep.md).
* Vous avez téléchargé l’image d’appareil virtuel pour Hyper-V à partir du portail Azure comme décrit dans [Préparer le portail pour Data Box Gateway](data-box-gateway-deploy-prep.md).

  > [!IMPORTANT]
  > Le logiciel en cours d’exécution sur Data Box Gateway peut seulement être utilisé avec la ressource Data Box Gateway.

### <a name="for-the-data-box-gateway-virtual-device"></a>Pour l’appareil virtuel Data Box Gateway

Avant de déployer un appareil, assurez-vous que :

* Vous avez accès à un système hôte exécutant Hyper-V sur Windows Server 2012 R2 ou une version ultérieure permettant de configurer un appareil.
* Le système hôte est en mesure de dédier les ressources suivantes pour configurer votre appareil virtuel :

  * Un minimum de 4 cœurs.
  * Au moins 8 Go de RAM.
  * Une interface réseau.
  * Un disque de système d’exploitation de 250 Go.
  * Un disque virtuel de 2 To pour les données.

### <a name="for-the-network-in-the-datacenter"></a>Pour le réseau dans le centre de données

Avant de commencer :

* Passez en revue les exigences de mise en réseau pour déployer Data Box Gateway et configurer le réseau du centre de données conformément à la configuration requise. Pour plus d’informations, consultez la [Configuration requise du réseau pour Data Box Gateway](data-box-gateway-system-requirements.md#networking-port-requirements).
* Assurez-vous que la bande passante Internet minimale est de 20 Mbits/s pour un fonctionnement optimal de l’appareil.

## <a name="check-the-host-system"></a>Vérifier le système hôte

Pour créer un appareil virtuel, vous avez besoin des éléments suivants :

* Le rôle Hyper-V installé sur Windows Server 2016, Windows Server 2012 R2 ou Windows Server 2012.
* Microsoft Hyper-V Manager sur un client Microsoft Windows connecté à l'hôte.
* Assurez-vous que le matériel sous-jacent (système hôte) sur lequel vous créez l’appareil virtuel est capable de dédier les ressources suivantes à votre appareil virtuel :

  * Au moins 4 processeurs virtuels.
  * Au moins 8 Go de RAM.
  * Une interface réseau connectée au réseau et capable d’acheminer le trafic vers Internet.
  * Un disque de système d’exploitation de 250 Go.
  * Un disque virtuel de 2 To pour les données système.

## <a name="bitlocker-considerations"></a>Considérations relatives à BitLocker

* Nous vous recommandons d’activer BitLocker sur votre machine virtuelle Data Box Gateway. Par défaut, BitLocker n’est pas activé. Pour plus d'informations, consultez les pages suivantes :
  * [Paramètres de prise en charge du chiffrement dans le Gestionnaire Hyper-V](hhttps://docs.microsoft.com/windows-server/virtualization/hyper-v/learn-more/generation-2-virtual-machine-security-settings-for-hyper-v#encryption-support-settings-in-hyper-v-manager)
  * [Prise en charge de BitLocker sur une machine virtuelle](https://kb.vmware.com/s/article/2036142)

## <a name="provision-a-virtual-device-in-hypervisor"></a>Configurer un appareil virtuel dans l’hyperviseur

Procédez comme suit pour configurer un appareil dans votre hyperviseur.

1. Sur votre hôte Windows Server, copiez l'image de l’appareil virtuel sur un disque local. Vous avez téléchargé cette image VHDX via le portail Azure. Prenez note de l’emplacement où vous avez copié l’image car vous l’utiliserez plus loin dans la procédure.
2. Ouvrez **Gestionnaire de serveur**. Dans le coin supérieur droit, cliquez sur **Outils**, puis sélectionnez **Gestionnaire Hyper-V**.

    ![Sélectionner le Gestionnaire Hyper-V dans le Gestionnaire de serveur](./media/data-box-gateway-deploy-provision-hyperv/image1.png)  
  
3. Dans le **Gestionnaire Hyper-V**, cliquez avec le bouton droit dans le volet d’étendue sur le nœud de votre système pour ouvrir le menu contextuel, puis cliquez sur **Nouveau** > **Machine virtuelle**.

   ![Créer une machine virtuelle dans le Gestionnaire Hyper-V](./media/data-box-gateway-deploy-provision-hyperv/image2.png)
4. Dans la page **Avant de commencer** de l’Assistant Nouvel ordinateur virtuel, cliquez sur **Suivant**.
5. Sur la page **Spécifier le nom et l’emplacement**, indiquez le **Nom** de votre appareil virtuel. Cliquez sur **Suivant**.

   ![Page Spécifier le nom et l’emplacement](./media/data-box-gateway-deploy-provision-hyperv/image3.png)
6. Dans la page **Spécifier la génération**, choisissez **Génération 2** pour le type d’image d’appareil .vhdx, puis cliquez sur **Suivant**.    

   ![Page Spécifier la génération](./media/data-box-gateway-deploy-provision-hyperv/image4.png)
7. Dans la page **Affecter la mémoire**, spécifiez une **Mémoire de démarrage** d’au moins **8 192 Mo** (n’activez pas la mémoire dynamique), puis cliquez sur **Suivant**.

   ![Page Affecter la mémoire](./media/data-box-gateway-deploy-provision-hyperv/image5.png) 
8. Dans la page **Configurer le réseau**, spécifiez le commutateur virtuel connecté à Internet, puis sur **Suivant**.

   ![Page Configurer la mise en réseau](./media/data-box-gateway-deploy-provision-hyperv/image6.png)
9. Dans la page **Connecter un disque dur virtuel**, choisissez **Utiliser un disque dur virtuel existant**, spécifiez l’emplacement de l’image de l’appareil virtuel, puis cliquez sur **Suivant**.

   ![Page Connecter un disque dur virtuel](./media/data-box-gateway-deploy-provision-hyperv/image7.png)
10. Passez en revue le **Sommaire**, puis cliquez sur **Terminer** pour créer la machine virtuelle.

    ![Page Fin de l’Assistant Nouvel ordinateur virtuel](./media/data-box-gateway-deploy-provision-hyperv/image8.png)
11. Pour répondre à la configuration minimale requise, vous avez besoin de 4 processeurs virtuels. Pour ajouter 4 processeurs virtuels, sélectionnez votre système hôte dans la fenêtre **Hyper-V Manager**. Dans le volet droit, sous la liste des **Machines virtuelles**, identifiez la machine virtuelle nouvellement créée. Sélectionnez et cliquez avec le bouton droit sur le nom de la machine, puis sélectionnez **Paramètres**.

    ![Paramètres de la machine virtuelle](./media/data-box-gateway-deploy-provision-hyperv/image9.png)
12. Sur la page **Paramètres**, cliquez sur **Processeur** dans le volet gauche. Dans le volet droit, définissez le **nombre de processeurs virtuels** sur 4 (ou plus). Cliquez sur **Appliquer**.

    ![Définir le nombre de processeurs virtuels dans la page Paramètres](./media/data-box-gateway-deploy-provision-hyperv/image10.png)
13. Pour répondre à la configuration minimale requise, vous devez également ajouter un disque de données virtuel de 2 To. Dans la page **Paramètres** :

    1. Dans le volet gauche, sélectionnez **Contrôleur SCSI**.
    2. Dans le volet droit, sélectionnez **Disque dur**, puis cliquez sur **Ajouter**.

    ![Ajouter un disque dur dans la page Paramètres](./media/data-box-gateway-deploy-provision-hyperv/image11.png)
14. Dans la page **Disque dur**, sélectionnez l’option **Disque dur virtuel**, puis cliquez sur **Nouveau**. L’**Assistant Nouveau disque dur virtuel** démarre.

    ![Assistant Nouveau disque dur virtuel](./media/data-box-gateway-deploy-provision-hyperv/image12.png)
15. Dans la page **Avant de commencer** de l’Assistant Nouveau disque dur virtuel, cliquez sur **Suivant**.
16. Dans la page **Choisir le format de disque**, acceptez le format **VHDX** par défaut. Cliquez sur **Suivant**.
17. Dans la page **Choisir le type de disque**, définissez le type de disque dur virtuel sur **Extension dynamique** (recommandé). Si vous choisissez un disque de **taille fixe**, il fonctionnera également, mais vous devrez peut-être patienter plus longtemps. Nous vous recommandons de ne pas utiliser l’option de **différenciation** . Cliquez sur **Suivant**.

    ![Page Choisir le type de disque](./media/data-box-gateway-deploy-provision-hyperv/image13.png)
18. Dans la page **Spécifier le nom et l’emplacement**, indiquez le **nom** et **l’emplacement** (vous pouvez naviguer vers cet emplacement) du disque de données. Cliquez sur **Suivant**.

    ![Page Spécifier le nom et l’emplacement](./media/data-box-gateway-deploy-provision-hyperv/image14.png)
19. Dans la page **Configurer un disque**, sélectionnez l’option **Créer un disque dur virtuel vierge** et spécifiez une taille de **2 To** (ou plus).

    Bien que 2 To soit la configuration minimale requise, vous pouvez toujours configurer un disque plus volumineux. Notez que vous ne pouvez pas réduire le disque une fois configuré. Toute tentative de réduction du disque entraîne la perte de toutes les données locales sur l’appareil. L’expansion du disque de données n’est pas prise en charge. Cliquez sur **Suivant**.

    ![Page Configurer un disque](./media/data-box-gateway-deploy-provision-hyperv/image15.png)
20. Dans la page **Résumé**, passez en revue les détails de votre disque de données virtuel et, si vous êtes satisfait, cliquez sur **Terminer** pour créer le disque. L’Assistant se ferme et un disque dur virtuel est ajouté à votre machine.

    ![Page Fin de l’Assistant Nouveau disque dur virtuel](./media/data-box-gateway-deploy-provision-hyperv/image16.png)
21. Revenez à la page **Paramètres**. Cliquez sur **OK** pour fermer la page **Paramètres** et revenir à la fenêtre du Gestionnaire Hyper-V.

    ![Page Paramètres](./media/data-box-gateway-deploy-provision-hyperv/image17.png)

## <a name="start-the-virtual-device-and-get-the-ip"></a>Démarrer l’appareil virtuel et obtenir l’adresse IP

Procédez comme suit pour démarrer votre appareil virtuel et vous y connecter.

#### <a name="to-start-the-virtual-device"></a>Pour démarrer l’appareil virtuel

1. Démarrez l’appareil virtuel.

   ![Démarrer l’appareil virtuel](./media/data-box-gateway-deploy-provision-hyperv/image18.png)
2. Une fois l'appareil en cours d'exécution, sélectionnez-le, cliquez avec le bouton droit et sélectionnez **Connexion**.

3. Vous devrez peut-être patienter 10 à 15 minutes pour que l’appareil soit prêt. Un message d'état s'affiche sur la console pour indiquer la progression. Lorsque l'appareil est prêt, sélectionnez **Action**. Appuyez sur `Ctrl + Alt + Delete` pour vous connecter à l’appareil virtuel. L’utilisateur par défaut est *EdgeUser* et le mot de passe par défaut est *Password1*.

   ![Se connecter à l’appareil virtuel](./media/data-box-gateway-deploy-provision-hyperv/image21.png)

4. Les étapes 5 à 7 s’appliquent uniquement lors de l’amorçage dans un environnement non DHCP. Si vous vous trouvez dans un environnement DHCP, ignorez ces étapes. Si vous avez démarré votre appareil dans un environnement non DHCP, vous verrez un message sur l’effet.

5. Pour configurer le réseau, utilisez la commande `Get-HcsIpAddress` pour répertorier les interfaces réseau activées sur votre appareil virtuel. Si votre appareil possède une seule interface réseau activée, le nom par défaut affecté à cette interface est `Ethernet`.

6. Utilisez l’applet de commande `Set-HcsIpAddress` pour configurer le réseau. Voir l’exemple suivant :

    `Set-HcsIpAddress –Name Ethernet –IpAddress 10.161.22.90 –Netmask 255.255.255.0 –Gateway 10.161.22.1`

7. Une fois l'installation initiale terminée et l’appareil démarré, vous verrez le texte de la bannière de l’appareil. Notez l'adresse IP et l'URL affichées dans le texte de la bannière pour gérer l'appareil. Utilisez cette adresse IP pour vous connecter à l’interface utilisateur web de votre appareil virtuel et pour finaliser la configuration et l’activation locales.

   ![Bannière de l’appareil virtuel avec l’adresse IP et l’URL de connexion](./media/data-box-gateway-deploy-provision-hyperv/image23.png)

Si votre appareil n’est pas conforme à la configuration minimale requise, une erreur apparaît dans le texte de bannière. Modifier la configuration de l’appareil afin qu’il dispose des ressources nécessaires à la configuration minimale. Vous pouvez ensuite redémarrer et vous connecter à l'appareil. Reportez-vous à la configuration minimale requise décrite à la section [Vérifier le système hôte](#check-the-host-system).

SI vous rencontrez une autre erreur durant la configuration initiale effectuée avec l’interface utilisateur web locale, reportez-vous aux workflows suivants :

* [Exécutez les tests de diagnostic pour dépanner la configuration de l’interface utilisateur web](data-box-gateway-troubleshoot.md#run-diagnostics).
* [Générez un package de journaux et affichez les fichiers journaux](data-box-gateway-troubleshoot.md#collect-support-package).

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez approfondi vos connaissances sur Data Box Gateway et avez notamment appris à :

> [!div class="checklist"]
> * Vérifier que l’hôte répond à la configuration minimale requise de l’appareil
> * Configurer un appareil virtuel dans l’hyperviseur
> * Démarrer l’appareil virtuel et obtenir l’adresse IP

Passez au tutoriel suivant pour savoir comment connecter, configurer et activer votre appareil virtuel.

> [!div class="nextstepaction"]
> [Connecter et configurer votre Data Box Gateway](./data-box-gateway-deploy-connect-setup-activate.md)
