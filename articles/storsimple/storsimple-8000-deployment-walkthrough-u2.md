---
title: Déployer un appareil StorSimple 8000 Series dans le portail Azure
description: Décrit les procédures et meilleures pratiques de déploiement de l’appareil StorSimple 8000 exécutant les mises à jour Update 3 et ultérieures, et du service StorSimple Device Manager.
author: alkohli
ms.service: storsimple
ms.topic: conceptual
ms.date: 04/23/2018
ms.author: alkohli
ms.openlocfilehash: 2c9feb1131f6d2d0eb75ac71e27dc46c226c52c1
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94961054"
---
# <a name="deploy-your-on-premises-storsimple-device-update-3-and-later"></a>Déployer votre appareil StorSimple local (Update 3 et ultérieure)

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]

## <a name="overview"></a>Vue d’ensemble
Bienvenue dans cette série de didacticiels consacrée au déploiement d’appareils Microsoft Azure StorSimple. Ces tutoriels de déploiement s’appliquent à StorSimple 8000 Update 3 ou ultérieure. Ils proposent une liste de contrôle de la configuration, ainsi que la configuration requise et des étapes de configuration détaillées pour votre appareil StorSimple.

Les informations contenues dans ces didacticiels supposent que vous avez passé en revue les précautions de sécurité, et décompacté, monté en rack et câblé votre appareil StorSimple. Si vous devez encore effectuer ces tâches, commencez par lire les [précautions de sécurité](storsimple-8000-safety.md). Suivez les instructions spécifiques à l'appareil pour déballer, monter en rack et câbler votre appareil.

* [Décompacter, monter en rack et câbler votre appareil 8100](storsimple-8100-hardware-installation.md)
* [Décompacter, monter en rack et câbler votre appareil 8600](storsimple-8600-hardware-installation.md)

Vous avez besoin de privilèges d’administrateur pour terminer le processus d’installation et de configuration. Nous vous recommandons de consulter la liste de contrôle de configuration avant de commencer. Le processus de déploiement et de configuration peut prendre du temps.

> [!NOTE]
> Les informations de déploiement StorSimple publiées sur le site web Microsoft Azure s’appliquent aux appareils StorSimple de la série 8000 uniquement. Pour obtenir des informations complètes sur les appareils de la série 7000, consultez la page : [http://onlinehelp.storsimple.com/](http://onlinehelp.storsimple.com). Pour plus d’informations sur le déploiement de la série 7000, consultez le [Guide de démarrage rapide du système StorSimple](http://onlinehelp.storsimple.com/111_Appliance/). 


## <a name="deployment-steps"></a>Étapes du déploiement
Suivez ces étapes requises pour configurer votre appareil StorSimple et le connecter à votre service StorSimple Device Manager. Outre les étapes requises, il existe des étapes et des procédures facultatives pendant le déploiement. Les instructions de déploiement étape par étape indiquent le moment où vous devez effectuer chacune de ces étapes facultatives.

| Étape | Description |
| --- | --- |
| **CONFIGURATION REQUISE** |Ces étapes doivent être effectuées en préparation du déploiement à venir. |
| [Liste de contrôle de la configuration du déploiement](#deployment-configuration-checklist) |Utilisez cette liste de contrôle pour rassembler et enregistrer des informations avant et pendant le déploiement. |
| [Conditions préalables au déploiement](#deployment-prerequisites) |Celles-ci valident que l'environnement est prêt pour le déploiement. |
|  | |
| **DÉPLOIEMENT ÉTAPE PAR ÉTAPE** |Ces étapes sont nécessaires pour déployer votre appareil StorSimple en production. |
| [Étape 1 : Création d’un nouveau service](#step-1-create-a-new-service) |Configurez le stockage et la gestion de cloud pour votre appareil StorSimple. *Ignorez cette étape si vous avez un service existant pour d'autres appareils StorSimple*. |
| [Étape 2 : Obtention de la clé d’inscription](#step-2-get-the-service-registration-key) |Utilisez cette clé pour inscrire et connecter votre appareil StorSimple auprès du service de gestion. |
| [Étape 3 : Configuration et inscription de l’appareil via Windows PowerShell pour StorSimple](#step-3-configure-and-register-the-device-through-windows-powershell-for-storsimple) |Pour terminer l’installation à l’aide du service de gestion, connectez l’appareil à votre réseau et enregistrez-le avec Azure. |
| [Étape 4 : Fin de l’installation minimale de l’appareil](#step-4-complete-minimum-device-setup)</br>[Meilleure pratique : mise à jour de votre appareil StorSimple](#scan-for-and-apply-updates) |Utilisez le service de gestion pour terminer la configuration de l'appareil et lui permettre d'assurer le stockage. |
| [Étape 5 : Création d’un conteneur de volumes](#step-5-create-a-volume-container) |Créez un conteneur pour mettre en service les volumes. Un conteneur de volumes dispose de paramètres de compte de stockage, de bande passante et de chiffrement pour tous les volumes qu’il contient. |
| [Étape 6 : Création d’un volume](#step-6-create-a-volume) |Mettez en service les volumes de stockage sur l’appareil StorSimple pour vos serveurs. |
| [Étape 7 : Montage, initialisation et formatage d’un volume](#step-7-mount-initialize-and-format-a-volume)</br>[Facultatif : Configuration de solution MPIO](storsimple-8000-configure-mpio-windows-server.md) |Connectez vos serveurs au stockage iSCSI fourni par l'appareil. Vous avez la possibilité de configurer la solution MPIO pour vous assurer que vos serveurs peuvent tolérer une panne de liaison, de réseau et d’interface. |
| [Étape 8 : Sauvegarde](#step-8-take-a-backup) |Configurez votre stratégie de sauvegarde pour protéger vos données. |
|  | |
| **AUTRES PROCÉDURES** |Vous devrez peut-être faire référence à ces procédures lors du déploiement de votre solution. |
| [Configuration d’un nouveau compte de stockage pour le service](#configure-a-new-storage-account-for-the-service) | |
| [Utilisation de PuTTY pour se connecter à la console série de l’appareil](#use-putty-to-connect-to-the-device-serial-console) | |
| [Obtention du nom qualifié d’un hôte Windows Server](#get-the-iqn-of-a-windows-server-host) | |
| [Création d’une sauvegarde manuelle](#create-a-manual-backup) | |


## <a name="deployment-configuration-checklist"></a>Liste de contrôle de la configuration du déploiement
Avant de déployer votre appareil, vous devez collecter des informations pour configurer le logiciel sur votre appareil StorSimple. L’étude préalable d’une partie de ces informations simplifie le déploiement de l’appareil StorSimple dans votre environnement. Téléchargez et utilisez cette liste de contrôle pour noter les détails de configuration lors du déploiement de votre appareil.

* [Télécharger la liste de contrôle de la configuration du déploiement StorSimple](https://www.microsoft.com/download/details.aspx?id=49159)

## <a name="deployment-prerequisites"></a>Conditions préalables au déploiement
Les sections suivantes décrivent les conditions préalables à la configuration de votre service StorSimple Device Manager et de votre appareil StorSimple.

### <a name="for-the-storsimple-device-manager-service"></a>Pour le service StorSimple Device Manager
Avant de commencer, assurez-vous que :

* Vous disposez d’un compte Microsoft doté d’informations d’identification d’accès.
* Vous disposez d’un compte de stockage Microsoft Azure doté d’informations d’identification d’accès.
* Le service StorSimple Device Manager est activé dans votre abonnement Microsoft Azure. Votre abonnement doit être acheté dans la page [Accord Entreprise](https://azure.microsoft.com/pricing/enterprise-agreement/).
* Vous avez accès à un logiciel d’émulation de terminal tel que PuTTY.

### <a name="for-the-device-in-the-datacenter"></a>Pour l’appareil dans le centre de données
Avant de configurer votre appareil, vérifiez qu'il est correctement décompacté, monté en rack et câblé à l'alimentation, au réseau et au port série comme cela est indiqué dans les articles suivants :

* [Décompacter, monter en rack et câbler votre appareil 8100](storsimple-8100-hardware-installation.md)
* [Décompacter, monter en rack et câbler votre appareil 8600](storsimple-8600-hardware-installation.md)

### <a name="for-the-network-in-the-datacenter"></a>Pour le réseau dans le centre de données
Avant de commencer, assurez-vous que :

* Les ports du pare-feu de votre centre de données sont ouverts pour autoriser le trafic iSCSI et du cloud, comme décrit dans la section [Configuration réseau requise pour un appareil StorSimple](storsimple-8000-system-requirements.md#networking-requirements-for-your-storsimple-device).

## <a name="step-by-step-deployment"></a>DÉPLOIEMENT ÉTAPE PAR ÉTAPE
Utilisez la procédure détaillée ci-dessous pour déployer votre appareil StorSimple dans le centre de données.

## <a name="step-1-create-a-new-service"></a>Étape 1 : Création d’un nouveau service
Un service StorSimple Device Manager peut gérer plusieurs appareils StorSimple. Procédez comme suit pour créer une instance du service StorSimple Device Manager.

[!INCLUDE [storsimple-create-new-service](../../includes/storsimple-8000-create-new-service.md)]

> [!IMPORTANT]
> Si vous n’avez pas activé la création automatique d’un compte de stockage avec votre service, vous devez créer au moins un compte de stockage après avoir créé un service. Ce compte de stockage est utilisé lorsque vous créez un conteneur de volumes.
>
> * Si vous n’avez pas créé de compte de stockage automatiquement, accédez à la page [Configuration d’un compte de stockage pour le service](#configure-a-new-storage-account-for-the-service) pour obtenir des instructions détaillées.
> * Si vous avez activé la création automatique d’un compte de stockage, passez à l’ [étape 2 : Obtention de la clé d’inscription](#step-2-get-the-service-registration-key).


## <a name="step-2-get-the-service-registration-key"></a>étape 2 : Obtention de la clé d’inscription
Une fois le service StorSimple Device Manager opérationnel, vous devez obtenir la clé d’inscription. Cette clé est utilisée pour inscrire et connecter votre appareil StorSimple auprès du service.

Procédez comme suit dans le portail Azure.

[!INCLUDE [storsimple-8000-get-service-registration-key](../../includes/storsimple-8000-get-service-registration-key.md)]

## <a name="step-3-configure-and-register-the-device-through-windows-powershell-for-storsimple"></a>Étape 3 : Configuration et inscription de l’appareil via Windows PowerShell pour StorSimple
Utilisez Windows PowerShell pour StorSimple pour terminer l’installation initiale de votre appareil StorSimple, comme expliqué dans la procédure suivante. Vous devez utiliser un logiciel d’émulation de terminal pour effectuer cette étape. Pour plus d’informations, consultez la rubrique [Utilisation de PuTTY pour se connecter à la console série de l’appareil](#use-putty-to-connect-to-the-device-serial-console).

[!INCLUDE [storsimple-8000-configure-and-register-device-u2](../../includes/storsimple-8000-configure-and-register-device-u2.md)]

## <a name="step-4-complete-minimum-device-setup"></a>Étape 4 : Fin de l'installation minimale de l'appareil
Pour pouvoir mener à bien la configuration minimale de votre appareil StorSimple, vous devez : 

* Entrez un nom convivial pour votre appareil.
* Définissez le fuseau horaire de l’appareil.
* affecter des adresses IP fixes aux deux contrôleurs.

Pour mener à bien la configuration minimale requise, procédez comme suit dans le portail Azure.

[!INCLUDE [storsimple-8000-complete-minimum-device-setup-u2](../../includes/storsimple-8000-complete-minimum-device-setup-u2.md)]

Après avoir terminé le programme d’installation minimale de l’appareil, il est recommandé de [rechercher et d’appliquer les dernières mises à jour](#scan-for-and-apply-updates).

## <a name="step-5-create-a-volume-container"></a>Étape 5 : Création d’un conteneur de volumes
Un conteneur de volumes dispose de paramètres de compte de stockage, de bande passante et de chiffrement pour tous les volumes qu’il contient. Vous devez créer un conteneur de volumes avant de commencer la configuration des volumes sur votre appareil StorSimple.

Pour créer un conteneur de volumes, procédez comme suit dans le portail Azure.

[!INCLUDE [storsimple-8000-create-volume-container](../../includes/storsimple-8000-create-volume-container.md)]

## <a name="step-6-create-a-volume"></a>Étape 6 : Création d’un volume
Après avoir créé un conteneur de volumes, vous pouvez configurer un volume de stockage sur l’appareil StorSimple pour vos serveurs. Pour créer un volume, procédez comme suit dans le portail Azure.

> [!IMPORTANT]
> StorSimple Device Manager peut créer des volumes alloués dynamiquement et de façon complète. Cependant, vous ne pouvez pas créer des volumes alloués partiellement.


[!INCLUDE [storsimple-8000-create-volume](../../includes/storsimple-8000-create-volume-u2.md)]

## <a name="step-7-mount-initialize-and-format-a-volume"></a>Étape 7 : Montage, initialisation et formatage d’un volume
Les étapes suivantes sont effectuées sur votre hôte Windows Server.

> [!IMPORTANT]
> * Pour une haute disponibilité de votre solution StorSimple, nous vous recommandons de configurer MPIO sur vos serveurs hôtes (facultatif) avant de configurer iSCSI. La configuration de MPIO sur les serveurs hôtes permet de s'assurer que les serveurs peuvent tolérer une panne de liaison, de réseau ou d'interface.
> * Pour obtenir les instructions d'installation et de configuration de MPIO et d'iSCSI sur l'hôte Windows Server, consultez [Configuration de MPIO pour votre appareil StorSimple](storsimple-8000-configure-mpio-windows-server.md). Ces instructions incluent également les étapes pour monter, initialiser et formater des volumes StorSimple.
> * Pour obtenir les instructions d'installation et de configuration de MPIO et d'iSCSI sur un hôte Linux, consultez [Configuration de MPIO pour votre hôte Linux StorSimple](storsimple-configure-mpio-on-linux.md)

Si vous décidez de ne pas configurer MPIO, procédez comme suit pour monter, initialiser et formater vos volumes StorSimple sur un hôte Windows Server.

[!INCLUDE [storsimple-8000-mount-initialize-format-volume](../../includes/storsimple-8000-mount-initialize-format-volume.md)]

## <a name="step-8-take-a-backup"></a>Étape 8 : Sauvegarde
Les sauvegardes fournissent une protection jusqu’à une date et une heure des volumes et optimisent la récupération tout en réduisant les délais de restauration. Vous pouvez effectuer deux types de sauvegarde sur votre appareil StorSimple : les instantanés locaux et les instantanés cloud. Chacun de ces types de sauvegarde peut être **Planifié** ou **Manuel**.

Pour créer une sauvegarde planifiée, procédez comme suit dans le portail Azure.

[!INCLUDE [storsimple-8000-take-backup](../../includes/storsimple-8000-take-backup.md)]

Vous pouvez à tout moment effectuer une sauvegarde manuelle. Pour connaître les procédures à suivre, consultez la rubrique [Création d’une sauvegarde manuelle](#create-a-manual-backup).

Vous avez terminé la configuration de l’appareil.

## <a name="configure-a-new-storage-account-for-the-service"></a>Configuration d’un compte de stockage pour le service
Il s’agit d’une étape facultative que vous devez exécuter uniquement si vous n’avez pas activé la création automatique d’un compte de stockage avec votre service. Un compte de stockage Microsoft Azure est requis pour créer un conteneur de volumes StorSimple.

Si vous devez créer un compte de stockage Azure dans une autre région, consultez la page [À propos des comptes de stockage Azure](../storage/common/storage-account-create.md) pour obtenir des instructions détaillées.

Procédez comme suit dans le portail Azure, sur la page **Service StorSimple Device Manager**.

[!INCLUDE [storsimple-8000-configure-new-storage-account-u2](../../includes/storsimple-8000-configure-new-storage-account-u2.md)]

## <a name="use-putty-to-connect-to-the-device-serial-console"></a>Utilisation de PuTTY pour se connecter à la console série de l’appareil
Pour vous connecter à Windows PowerShell pour StorSimple, vous devez utiliser un logiciel d’émulation de terminal tel que PuTTY. Vous pouvez utiliser PuTTY lorsque vous accédez à l’appareil directement via la console série ou en ouvrant une session telnet à partir d’un ordinateur distant.

[!INCLUDE [Use PuTTY to connect to the device serial console](../../includes/storsimple-use-putty.md)]

## <a name="scan-for-and-apply-updates"></a>Recherche et application des mises à jour
La mise à jour de votre appareil peut prendre plusieurs heures. Pour connaître les étapes détaillées d’installation de la mise à jour la plus récente, consultez la page relative à [l’installation d’Update 5](storsimple-8000-install-update-5.md).


## <a name="get-the-iqn-of-a-windows-server-host"></a>Obtention du nom qualifié d’un hôte Windows Server
Procédez comme suit pour obtenir le nom qualifié iSCSI d’un hôte Windows exécutant Windows Server® 2012.

[!INCLUDE [Create a manual backup](../../includes/storsimple-get-iqn.md)]

## <a name="create-a-manual-backup"></a>Création d’une sauvegarde manuelle
Pour créer une sauvegarde manuelle à la demande pour un seul volume sur votre appareil StorSimple, procédez comme suit dans le portail Azure.

[!INCLUDE [Create a manual backup](../../includes/storsimple-8000-create-manual-backup.md)]

## <a name="view-the-pinout-diagram-for-serial-cable-for-storsimple"></a>Afficher le diagramme de brochage pour le câblage série destiné à StorSimple
Le diagramme de brochage suivant peut être utilisé pour le câblage de la console série StorSimple.

Dans le cas présent, le connecteur femelle DB9 est P1 et le connecteur 3,5 mm est P2.

![Diagramme de brochage 1 pour le câblage de console série StorSimple](./media/storsimple-8000-deployment-walkthrough-u2/pinout-diagram1.png)

L’extrémité de la prise stéréo correspond à PIN 3 RX, le milieu correspond à PIN 2 TX et la base correspond à PIN 1 GND, comme indiqué dans le diagramme suivant.

![Diagramme de brochage 2 pour le câblage de console série StorSimple](./media/storsimple-8000-deployment-walkthrough-u2/pinout-diagram2.png)



## <a name="next-steps"></a>Étapes suivantes
* [Configurer StorSimple Cloud Appliance](storsimple-8000-cloud-appliance-u2.md).
* [Utilisez le service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).