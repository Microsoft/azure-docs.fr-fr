---
title: Préparation du portail pour StorSimple Virtual Array
description: Le premier didacticiel de déploiement de StorSimple Virtual Array implique la préparation du portail Azure
author: alkohli
ms.assetid: 68a4cfd3-94c9-46cb-805c-46217290ce02
ms.service: storsimple
ms.topic: conceptual
ms.date: 07/25/2019
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c1a3d0b05f04ad8c182d428843479160666b96ee
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87070324"
---
# <a name="deploy-storsimple-virtual-array---prepare-the-azure-portal"></a>Déploiement de StorSimple Virtual Array - Préparation du portail Azure

![Diagramme montrant les étapes nécessaires au déploiement d’un tableau virtuel. La première étape intitulée Prise en main est en surbrillance.](./media/storsimple-virtual-array-deploy1-portal-prep/getstarted4.png)


## <a name="overview"></a>Vue d’ensemble

[!INCLUDE [storsimple-virtual-array-eol-banner](../../includes/storsimple-virtual-array-eol-banner.md)]

Il s’agit du premier article de la série de didacticiels sur le déploiement nécessaire pour déployer complètement votre tableau virtuel en tant que serveur de fichiers ou serveur iSCSI à l’aide du modèle Resource Manager. Cet article décrit la préparation nécessaire pour créer et configurer votre service StorSimple Device Manager avant la configuration d'un tableau virtuel. Cet article contient également des liens vers une liste de vérification de la configuration du déploiement ainsi que la configuration requise.

Vous avez besoin de privilèges d’administrateur pour terminer le processus d’installation et de configuration. Nous vous recommandons de consulter la liste de contrôle de configuration du déploiement avant de commencer. La préparation du portail prend moins de 10 minutes.

Les informations publiées dans cet article s’appliquent au déploiement de StorSimple Virtual Array dans le portail Azure, ainsi que dans Microsoft Azure Government Cloud.

### <a name="get-started"></a>Bien démarrer
Le flux de travail du déploiement se compose de la préparation du portail, de la configuration d'un tableau virtuel dans votre environnement virtualisé et de la finalisation de la configuration. Pour commencer le déploiement de StorSimple Virtual Array en tant que serveur de fichiers ou serveur iSCSI, vous devrez consulter les ressources suivantes au format tableau.

#### <a name="deployment-articles"></a>Articles sur le déploiement

Consultez les articles suivants dans l'ordre indiqué pour déployer votre instance StorSimple Virtual Array.

| **#** | **Dans cette étape** | **Procédez comme suit …** | **Et utilisez ces documents.** |
| --- | --- | --- | --- |
| 1. |**Configuration du portail Azure** |Créez et configurez votre service StorSimple Device Manager avant de configurer une instance StorSimple Virtual Array. |[Préparation du portail](storsimple-virtual-array-deploy1-portal-prep.md) |
| 2. |**Configuration du tableau virtuel** |Pour Hyper-V, configurez et connectez-vous à une instance StorSimple Virtual Array sur un système hôte exécutant Hyper-V sur Windows Server 2012 R2, Windows Server 2012 ou Windows Server 2008 R2. <br></br> <br></br> Pour VMware, provisionnez et connectez-vous à une instance StorSimple Virtual Array sur un système hôte exécutant VMware ESXi 5.0, 5.5, 6.0 ou 6.5.<br></br> |[Configurer un tableau virtuel dans Hyper-V](storsimple-virtual-array-deploy2-provision-hyperv.md) <br></br> <br></br> [Configuration d’un tableau virtuel dans VMware](storsimple-virtual-array-deploy2-provision-vmware.md) |
| 3. |**Configuration du tableau virtuel** |Pour votre serveur de fichiers, effectuez la configuration initiale, inscrivez votre serveur de fichiers StorSimple, puis finalisez la configuration de l’appareil. Vous pouvez ensuite configurer les partages SMB. <br></br> <br></br> Pour votre serveur iSCSI, effectuez la configuration initiale, inscrivez votre serveur iSCSI StorSimple, puis finalisez la configuration de l’appareil. Vous pouvez ensuite configurer les volumes iSCSI. |[Configuration d’un tableau virtuel comme serveur de fichiers](storsimple-virtual-array-deploy3-fs-setup.md)<br></br> <br></br>[Configuration d’un virtuel tableau comme serveur iSCSI](storsimple-virtual-array-deploy3-iscsi-setup.md) |

Vous pouvez maintenant commencer à configurer le portail Azure.

## <a name="configuration-checklist"></a>Liste de contrôle de la configuration

La liste de contrôle de la configuration présente les informations dont vous avez besoin avant de configurer le logiciel sur votre instance StorSimple Virtual Array. L’étude préalable de ces informations simplifie le déploiement de l’appareil StorSimple dans votre environnement. Selon que votre instance StorSimple Virtual Array sera déployée comme serveur de fichiers ou serveur iSCSI, suivez l’une des listes de contrôle suivantes.

* Téléchargez la [liste de vérification de la configuration du serveur de fichiers StorSimple Virtual Array](https://download.microsoft.com/download/E/E/6/EE690BB0-B442-4B84-8165-4731EE727ACF/MicrosoftAzureStorSimpleVirtualArrayFileServerConfigurationChecklist.pdf).
* Téléchargez la [liste de vérification de la configuration du serveur iSCSI StorSimple Virtual Array](https://download.microsoft.com/download/E/E/6/EE690BB0-B442-4B84-8165-4731EE727ACF/MicrosoftAzureStorSimpleVirtualArrayiSCSIServerConfigurationChecklist.pdf).

## <a name="prerequisites"></a>Prérequis

Vous trouverez ici les conditions préalables à la configuration de votre service StorSimple Device Manager, de votre instance StorSimple Virtual Array et du réseau de votre centre de données.

### <a name="for-the-storsimple-device-manager-service"></a>Pour le service StorSimple Device Manager

Avant de commencer, assurez-vous que :

* Vous disposez d’un compte Microsoft doté d’informations d’identification d’accès.
* Vous disposez d’un compte de stockage Microsoft Azure doté d’informations d’identification d’accès.
* Votre abonnement Microsoft Azure devrait être activé pour le service StorSimple Device Manager.

### <a name="for-the-storsimple-virtual-array"></a>Pour StorSimple Virtual Array

Avant de déployer un tableau virtuel, assurez-vous que :

* Vous avez accès à un système hôte exécutant Hyper-V sous Windows Server 2008 R2 ou version ultérieure ou sous VMware (ESXi 5.0, 5.5, 6.0 ou 6.5) et permettant de configurer un appareil.
* Le système hôte est en mesure de dédier les ressources suivantes pour configurer votre tableau virtuel :
  
  * Un minimum de 4 cœurs.
  * Au moins 8 Go de RAM. Si vous envisagez de configurer le tableau virtuel en tant que serveur de fichiers, 8 Go prennent en charge 2 millions de fichiers. Vous avez besoin de 16 Go de RAM pour prendre en charge 2 à 4 millions de fichiers.
  * Une interface réseau.
  * Un disque virtuel de 500 Go pour les données système.

### <a name="for-the-datacenter-network"></a>Pour le réseau du centre de données

Avant de commencer, assurez-vous que :

* Le réseau de votre centre de données est configuré conformément à la configuration réseau requise pour votre appareil StorSimple. Pour plus d'informations, consultez la [Configuration système requise pour StorSimple Virtual Array](storsimple-ova-system-requirements.md).
* Votre instance StorSimple Virtual Array a une bande passante Internet dédiée de 5 Mbits/s (ou plus) disponible à tout moment. La bande passante ne doit pas être partagée avec d’autres applications.

## <a name="step-by-step-preparation"></a>Préparation étape par étape

Utilisez la procédure pas à pas suivante afin de préparer votre portail pour le service StorSimple Device Manager.

## <a name="step-1-create-a-new-service"></a>Étape 1 : Créer un service

Une seule instance du service StorSimple Device Manager peut gérer plusieurs instances StorSimple Virtual Array. Procédez comme suit pour créer une instance du service StorSimple Device Manager. Si vous disposez d'un service StorSimple Device Manager existant pour gérer vos tableaux virtuels, ignorez cette étape et passez à l’[Étape 2 : Obtention de la clé d’inscription](#step-2-get-the-service-registration-key).

[!INCLUDE [storsimple-virtual-array-create-new-service](../../includes/storsimple-virtual-array-create-new-service.md)]

> [!IMPORTANT]
> Si vous n’avez pas activé la création automatique d’un compte de stockage avec votre service, vous devez créer au moins un compte de stockage après avoir créé un service.
> 
> * Si vous n’avez pas créé de compte de stockage automatiquement, accédez à la page [Configuration d’un compte de stockage pour le service](#optional-step-configure-a-new-storage-account-for-the-service) pour obtenir des instructions détaillées.
> * Si vous avez activé la création automatique d'un compte de stockage, passez à l'[Étape 2 : Obtenir la clé d’inscription du service](#step-2-get-the-service-registration-key).
> 
> 

## <a name="step-2-get-the-service-registration-key"></a>Étape 2 : Obtenir la clé d’inscription du service

Une fois le service StorSimple Device Manager opérationnel, vous devez obtenir la clé d’inscription. Cette clé est utilisée pour inscrire et connecter votre appareil StorSimple auprès du service.

Procédez comme suit dans le [portail Azure](https://portal.azure.com/).

[!INCLUDE [storsimple-virtual-array-get-service-registration-key](../../includes/storsimple-virtual-array-get-service-registration-key.md)]

> [!NOTE]
> La clé d’inscription au service sert à inscrire tous les appareils StorSimple Device Manager que vous devez inscrire auprès du service StorSimple Device Manager.
> 
> 

## <a name="step-3-download-the-virtual-array-image"></a>Étape 3 : Télécharger l'image du tableau virtuel

Après avoir obtenu la clé d'inscription du service, vous devrez télécharger l'image du tableau virtuel approprié pour configurer un tableau virtuel sur votre système hôte. Les images du tableau virtuel sont spécifiques au système d'exploitation et peuvent être téléchargées depuis la page Démarrage rapide du portail Azure.

> [!IMPORTANT]
> Le logiciel exécuté sur StorSimple Virtual Array peut uniquement servir en association avec le service StorSimple Device Manager.
> 
> 

Procédez comme suit dans le [portail Azure](https://portal.azure.com/).

#### <a name="to-get-the-virtual-array-image"></a>Pour obtenir l'image du tableau virtuel

1. Connectez-vous au [portail Azure](https://portal.azure.com/). 
2. Dans le portail Azure, cliquez sur **Parcourir > Instances de StorSimple Device Manager**.
3. Sélectionnez un service StorSimple Device Manager existant. Dans le panneau **Instance de StorSimple Device Manager**, cliquez sur **Démarrage rapide**. 
4. Cliquez sur le lien correspondant à l’image que vous souhaitez télécharger à partir du Centre de téléchargement Microsoft. Les fichiers image ont une taille d’environ 4,8 Go.
   
   * VHDX pour Hyper-V sur Windows Server 2012 et versions ultérieures
   * VHD pour Hyper-V sur Windows Server 2008 R2 et versions ultérieures
   * VMDK pour VMWare ESXi 5.0, 5.5, 6.0 ou 6.5
5. Téléchargez et décompressez le fichier sur un lecteur local et notez l’emplacement du fichier décompressé.

## <a name="optional-step-configure-a-new-storage-account-for-the-service"></a>Étape facultative : Configuration d’un compte de stockage pour le service

Il s’agit d’une étape facultative que vous devez exécuter uniquement si vous n’avez pas activé la création automatique d’un compte de stockage avec votre service.

Si vous devez créer un compte de stockage Azure dans une autre région, consultez la rubrique [Créez un compte de stockage](../storage/common/storage-account-create.md) pour obtenir des instructions détaillées.

Procédez comme suit dans le [portail Azure](https://ms.portal.azure.com/) sur la page du service StorSimple Device Manager pour ajouter un compte de stockage Microsoft Azure existant.

#### <a name="to-add-a-storage-account-credential-that-has-the-same-azure-subscription-as-the-device-manager-service"></a>Pour ajouter une information d’identification de compte de stockage bénéficiant de la même inscription Azure que le service Device Manager

1. Accédez au service Device Manager, sélectionnez-le et double-cliquez. Cette opération provoque l’ouverture du panneau **Vue d’ensemble**.
2. Sélectionnez **Informations d’identification du compte de stockage** dans la section **Configuration**.
3. Cliquez sur **Add**.
4. Dans le panneau **Ajouter un compte de stockage**, procédez comme suit :
   
   1. Pour **Abonnement**, sélectionnez **Actuel**.
   
   2. Fournissez le nom de votre compte de stockage Azure.
   
   3. Sélectionnez **Activer** afin de créer un canal sécurisé pour la communication réseau entre votre appareil StorSimple et le cloud. Sélectionnez **Désactiver** uniquement si vous évoluez au sein d’un cloud privé.
   
   4. Cliquez sur **Add**. Un message s’affiche pour vous informer une fois le compte de stockage correctement créé.<br></br>
   
      ![Ajouter une information d’identification de compte de stockage existant](./media/storsimple-virtual-array-manage-storage-accounts/ova-add-storageacct.png)

## <a name="next-step"></a>Étape suivante

L'étape suivante consiste à configurer une machine virtuelle pour votre instance StorSimple Virtual Array. Selon votre système d'exploitation hôte, consultez les instructions détaillées dans :

* [Configurer StorSimple Virtual Array dans Hyper-V](storsimple-virtual-array-deploy2-provision-hyperv.md)
* [Configurer StorSimple Virtual Array dans VMware](storsimple-virtual-array-deploy2-provision-vmware.md)

