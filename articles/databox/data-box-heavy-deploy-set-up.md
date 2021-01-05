---
title: Tutoriel - Configurer Azure Data Box Heavy | Microsoft Docs
description: Dans ce tutoriel, découvrez comment raccorder, connecter et mettre sous tension Azure Data Box Heavy.
services: databox
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: tutorial
ms.date: 09/03/2019
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: c38ff7e642017afe5f220ae26d3a04c2c0b706ee
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/07/2020
ms.locfileid: "87920940"
---
::: zone target = "docs"

# <a name="tutorial-cable-and-connect-to-your-azure-data-box-heavy"></a>Tutoriel : Brancher votre Azure Data Box Heavy et s’y connecter

::: zone-end

::: zone target = "chromeless"

## <a name="get-started-with-azure-data-box-heavy"></a>Bien démarrer avec Azure Data Box Heavy

::: zone-end

::: zone target = "docs"

Ce tutoriel explique comment brancher votre Azure Data Box Heavy, vous y connecter et l’activer.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Brancher votre Data Box Heavy
> * Connexion à votre Data Box Heavy

::: zone-end

::: zone target = "chromeless"

Ce guide fournit des instructions sur la façon de passer en revue les prérequis, de raccorder et de connecter votre appareil, de copier des données, d’effectuer un chargement vers Azure, puis de vérifier les données chargées.

::: zone-end

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que :

1. Vous avez terminé le [Tutoriel : Commander Azure Data Box Heavy](data-box-heavy-deploy-ordered.md).
2. Vous avez reçu votre Data Box Heavy et l’état de la commande dans le portail est **Livré**.
3. Vous avez consulté les [consignes de sécurité Data Box Heavy](data-box-safety.md).
4. Vous devez avoir accès à une surface plane dans le centre de données à proximité d’une connexion réseau disponible et pouvant loger un appareil ayant l’encombrement mentionné. Cet appareil ne peut pas être monté sur un rack.
5. Vous avez reçu quatre cordons d’alimentation avec mise à la terre à utiliser avec votre dispositif de stockage.
6. Vous disposez d’un ordinateur hôte connecté au réseau du centre de données. Votre Data Box Heavy va copier les données de cet ordinateur. Votre ordinateur hôte doit exécuter un [système d’exploitation pris en charge](data-box-heavy-system-requirements.md).
7. Votre centre de données doit avoir un réseau haut débit. Nous vous recommandons vivement d’utiliser au minimum une connexion 10 GbE. 
8. Vous devez disposer d’un ordinateur portable équipé d’un câble RJ-45 pour vous connecter à l’interface utilisateur locale et configurer l’appareil. Utilisez l’ordinateur portable pour configurer une fois chaque nœud de l’appareil.
9. Vous devez disposer d’un câble 40 Gbits/s ou d’un câble 10 Gbits/s par nœud d’appareil.
    - Choisissez des câbles compatibles avec l’interface réseau [Mellanox MCX314A-BCCT](https://store.mellanox.com/products/mellanox-mcx314a-bcct-connectx-3-pro-en-network-interface-card-40-56gbe-dual-port-qsfp-pcie3-0-x8-8gt-s-rohs-r6.html).
    - Concernant le câble 40 Gbits/s, l’extrémité du câble côté appareil doit être QSFP+.
    - Concernant le câble 10 Gbits/s, vous devez disposer d’un câble SFP+ dont une extrémité est branchée à un commutateur 10 Gbits/s, avec une carte QSFP+ à SFP+ (ou une carte QSA) pour l’extrémité qui est branchée sur l’appareil.

::: zone target = "docs"

## <a name="cable-your-device-for-power"></a>Raccorder votre appareil à l’alimentation électrique

Effectuez les étapes suivantes pour brancher les câbles de votre appareil.

1. Vérifiez que l’appareil n’a pas été endommagé. Si l’appareil est altéré ou gravement endommagé, ne continuez pas. [Contactez immédiatement le Support Microsoft](data-box-disk-contact-microsoft-support.md) pour savoir si l’appareil est en état de marche ou si Microsoft doit vous envoyer un appareil de remplacement.
2. Déplacez l’appareil sur l’emplacement d’installation.

    ![Emplacement d’installation de l’appareil Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-install-site.png)

3. Verrouillez les roulettes arrière situées sur l’appareil, comme indiqué ci-dessous.

    ![Roulettes verrouillées de l’appareil Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-casters-locked.png)

4. Localisez les boutons qui déverrouillent les portes avant et arrière de l’appareil. Déverrouillez et déplacez la porte avant jusqu’à ce qu’elle se trouve au niveau du côté de l’appareil. Répétez cette procédure avec la porte arrière également.
    Les deux portes doivent rester ouvertes quand l’appareil est opérationnel pour permettre une circulation optimale du flux d’air de l’avant vers l’arrière de l’appareil.

    ![Portes de Data Box Heavy ouvertes](media/data-box-heavy-deploy-set-up/data-box-heavy-doors-open.png)

5. Le plateau situé à l’arrière de l’appareil doit avoir quatre câbles d’alimentation. Retirez tous les câbles du plateau et mettez-les de côté.

    ![Cordons d’alimentation sur le plateau du Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-power-cords-tray.png)

6. L’étape suivante consiste à identifier les différents ports à l’arrière de l’appareil. Il existe deux nœuds d’appareil : **NODE1** et **NODE2**. Chaque nœud comprend quatre interfaces réseau : **MGMT**, **DATA1**, **DATA2**, **DATA3**. **MGMT** est utilisée pour configurer la gestion lors de la configuration initiale de l’appareil. **DATA1**-**DATA3** sont des ports de données. Les ports **MGMT** et **DATA3** sont à 1 Gbit/s, tandis que **DATA1** et **DATA2** peuvent fonctionner en tant que ports 40 Gbit/s ou que ports 10 Gbits/s. En bas des deux nœuds d’appareil se trouvent quatre unités d’alimentation qui sont partagées entre eux. Quand vous vous trouvez face à cet appareil, les **unités d’alimentation** sont, de gauche à droite, **PSU1**, **PSU2**, **PSU3** et **PSU4**.

    ![Ports Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-ports.png)

7. Connectez les quatre câbles aux blocs d’alimentation de l’appareil. Les LED vertes s’allument et clignotent.
8. Utilisez les boutons d’alimentation situés sur le plan avant pour activer les nœuds de l’appareil. Gardez le bouton d’alimentation enfoncé pendant quelques secondes jusqu’à ce que les voyants bleus s’allument. Toutes les LED vertes des blocs d’alimentation situés à l’arrière de l’appareil doivent maintenant être fixes. Le panneau de commande avant de l’appareil comporte également des LED de panne. Quand une LED de panne est allumée, cela indique un ventilateur ou une unité d’alimentation défectueux, ou un problème avec les lecteurs de disque.  

    ![Panneau de commande avant du Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-front-ops-panel.png)

## <a name="cable-first-node-for-network"></a>Brancher le premier nœud au réseau

Sur l’un des nœuds de l’appareil, effectuez les étapes suivantes pour effectuer un branchement au réseau.

1. Utilisez un câble réseau RJ-45 CAT 6 (câble raccordé au connecteur MGMT en haut à droite sur l’image) pour connecter l’ordinateur hôte au port de gestion 1 Gbit/s.
2. Utilisez un câble QSFP + (fibre ou cuivre) pour connecter au moins une interface réseau 40 Gbits/s (plutôt que 1 Gbit/s) pour les données. En cas d’utilisation d’un commutateur 10 Gbits/s, utilisez un câble SFP+ avec une carte QSFP+ à SFP+ (la carte QSA) pour connecter l’interface réseau 40 Gbits/s pour les données.

    ![Ports Data Box Heavy câblés](media/data-box-heavy-deploy-set-up/data-box-heavy-ports-cabled.png)

    > [!IMPORTANT]
    > DATA1 et DATA2 sont permutés et ne correspondent pas à ce qui est affiché dans l’interface utilisateur web locale.
    > La carte du câble 40 Gbits/s se connecte quand elle est insérée comme indiqué ci-dessous.

    ![Carte du câble 40 Gbits/s Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-cable-adaptor.png)

## <a name="configure-first-node"></a>Configurer le premier nœud

Effectuez les étapes suivantes pour configurer votre appareil à l’aide de la configuration locale et du portail Azure.

1. Téléchargez les informations d’identification de l’appareil à partir du portail. Accédez à **Général > Détails de l’appareil**. Copiez le **mot de passe de l'appareil**. Ces mots de passe sont liés à une commande spécifique dans le portail. Vous verrez les deux numéros de série d’appareils correspondant aux deux nœuds dans Data Box Heavy. Le mot de passe administrateur est identique pour les deux nœuds.

    ![Informations d’identification de l’appareil Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-device-credentials.png)

2. Connectez votre station de travail cliente à l’appareil par le biais d’un câble réseau CAT 6 RJ-45.
3. Configurez la carte Ethernet sur l’ordinateur que vous utilisez pour vous connecter à l’appareil avec l’adresse IP statique `192.168.100.5` et le sous-réseau `255.255.255.0`.

    ![Data Box Heavy se connecte à l’interface utilisateur web locale](media/data-box-heavy-deploy-set-up/data-box-heavy-connect-local-web-ui.png)

4. Connectez-vous à l’interface utilisateur web locale de l’appareil à l’adresse URL suivante : `http://192.168.100.10`. Cliquez sur **Avancé**, puis sur **Continuer vers 192.168.100.10 (non sécurisé)** .
5. Vous voyez une page de **connexion** pour l’interface utilisateur web locale.
    
    - L’un des numéros de série de nœud affichés dans cette page correspond entre l’interface utilisateur du portail et l’interface utilisateur web locale. Notez le numéro du nœud dans le mappage des numéros de série. Il existe deux nœuds et deux numéros de série d’appareils dans le portail. Ce mappage vous permet de comprendre quel nœud correspond à quel numéro de série.
    - L’appareil est verrouillé à ce stade.
    - Fournissez le mot de passe administrateur de l’appareil que vous avez obtenu à l’étape précédente pour vous connecter à l’appareil. Cliquez sur **Se connecter**.

    ![Se connecter à l’interface utilisateur web locale Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-unlock-device.png)

5. Sur le tableau de bord, vérifiez que les interfaces réseau sont configurées. Il existe quatre interfaces réseau sur votre nœud d’appareil : deux à 1 Gbit/s et deux à 40 Gbits/s. L’une des interfaces 1 Gbit/s est une interface de gestion, donc non configurable par l’utilisateur. Les trois interfaces réseau restantes sont dédiées aux données et peuvent être configurées par l’utilisateur.

- Si le protocole DHCP est activé dans votre environnement, les interfaces réseau sont configurées automatiquement.
- Si DHCP n’est pas activé, accédez à Définir des interfaces réseau, puis attribuez des adresses IP statiques si nécessaire.

    ![Nœud 1 du tableau de bord Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-dashboard-1.png)

## <a name="configure-second-node"></a>Configurer le deuxième nœud

Effectuez les étapes décrites dans la section [Configurer le premier nœud](#configure-first-node) pour le deuxième nœud de l’appareil.

![Nœud 2 du tableau de bord Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-dashboard-2.png)

Une fois que l’appareil est configuré, vous pouvez vous connecter aux partages de l’appareil et copier les données de votre ordinateur vers l’appareil.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez approfondi vos connaissances sur Azure Data Box Heavy, notamment concernant les points suivants :

> [!div class="checklist"]
> * Brancher votre Data Box Heavy
> * Connexion à votre Data Box Heavy

Passez au tutoriel suivant pour découvrir comment copier des données sur votre Data Box Heavy.

> [!div class="nextstepaction"]
> [Copier des données sur une Azure Data Box](./data-box-heavy-deploy-copy-data.md)

::: zone-end
