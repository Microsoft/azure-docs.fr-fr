---
title: Didacticiel sur la préparation du portail Azure pour déployer Data Box Gateway | Microsoft Docs
description: Le premier didacticiel visant à déployer Azure Data Box Gateway implique la préparation du Portail Azure.
services: databox-edge-gateway
author: v-dalc
ms.service: databox
ms.subservice: gateway
ms.topic: tutorial
ms.date: 10/15/2020
ms.author: alkohli
ms.openlocfilehash: f1b00273314c845a36c7efdb9cf9f4de9d5cdd46
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580305"
---
# <a name="tutorial-prepare-to-deploy-azure-data-box-gateway"></a>Tutoriel : Préparer le déploiement d’Azure Data Box Gateway

Ce didacticiel est le premier d’une série de didacticiels permettant de déployer entièrement Azure Data Box Gateway. Il explique comment préparer le Portail Azure pour déployer la ressource Data Box Gateway.

Vous avez besoin de privilèges d’administrateur pour terminer le processus d’installation et de configuration. La préparation du portail prend moins de 10 minutes.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Créer une nouvelle ressource
> * Télécharger l'image de l’appareil virtuel
> * Obtenir la clé d'activation

## <a name="get-started"></a>Bien démarrer

Pour déployer Data Box Gateway, consultez les didacticiels suivants dans l’ordre indiqué.

| **#** | **Dans cette étape** | **Utilisez ces documents** |
| --- | --- | --- | 
| 1. |**[Préparer le Portail Azure pour Data Box Gateway](data-box-gateway-deploy-prep.md)** |Créez et configurez votre ressource Data Box Gateway avant l’approvisionnement d’un appareil virtuel Data Box Gateway. |
| 2. |**[Approvisionner Azure Data Box Gateway dans Hyper-V](data-box-gateway-deploy-provision-hyperv.md)** <br><br><br>**[Approvisionner Azure Data Box Gateway dans VMware](data-box-gateway-deploy-provision-vmware.md)**|Pour Hyper-V, approvisionnez un appareil virtuel Data Box Gateway et connectez-vous à celui-ci sur un système hôte exécutant Hyper-V sur Windows Server 2016, Windows Server 2012 ou R2. <br><br><br> Pour VMware, provisionnez un appareil virtuel Data Box Gateway et connectez-vous à celui-ci sur un système hôte exécutant VMware ESXi 6.0, 6.5 ou 6.7.<br></br> |
| 3. |**[Se connecter à Data Box Gateway, le configurer et l’activer](data-box-gateway-deploy-connect-setup-activate.md)** |Connectez-vous à l’interface utilisateur web locale, terminez l’installation de l’appareil et activez-le. Vous pouvez ensuite configurer les partages SMB.  |
| 4. |**[Transférer des données avec Data Box Gateway](data-box-gateway-deploy-add-shares.md)** |Ajoutez des partages et connectez-vous à des partages via SMB ou NFS. |

Vous pouvez maintenant commencer à configurer le portail Azure.

## <a name="prerequisites"></a>Prérequis

Dans les informations suivantes, vous trouverez la configuration requise pour votre ressource Data Box Gateway, votre appareil Data Box Gateway et le réseau du centre de données.

### <a name="for-the-data-box-gateway-resource"></a>Pour la ressource Data Box Gateway

Avant de commencer, assurez-vous que :

* Votre abonnement Microsoft Azure est activé pour une ressource Azure Stack Edge. Vérifiez que vous avez utilisé un abonnement pris en charge tel que [Contrat Entreprise (EA) Microsoft](https://azure.microsoft.com/overview/sales-number/), [Fournisseur de solutions cloud (CSP)](https://docs.microsoft.com/partner-center/azure-plan-lp) ou [Microsoft Azure Sponsorship](https://azure.microsoft.com/offers/ms-azr-0036p/).
* Vous disposez d’un accès Propriétaire ou Contributeur au niveau du groupe de ressources pour les ressources Azure Stack Edge / Data Box Gateway, IoT Hub et Stockage Azure.
    - Pour créer une ressource Azure Stack Edge / Data Box Gateway, vous devez disposer d’autorisations Contributeur (ou supérieures) au niveau du groupe de ressources. Vous devez également être sûr que le fournisseur `Microsoft.DataBoxEdge` est inscrit. Pour plus d’informations sur l’inscription, consultez [Inscrire un fournisseur de ressources](data-box-gateway-manage-access-power-connectivity-mode.md#register-resource-providers).
    - Pour créer une ressource de compte de stockage, là encore, vous avez besoin d’autorisations Contributeur (ou supérieures) au niveau du groupe de ressources. Par défaut, le stockage Azure est un fournisseur de ressources inscrit.
- Vous disposez d’un accès administrateur ou utilisateur à l’API Microsoft Graph. Pour plus d’informations, consultez la [documentation de référence sur les autorisations Microsoft Graph](https://docs.microsoft.com/graph/permissions-reference).
- Vous disposez d’un compte de stockage Microsoft Azure doté d’informations d’identification d’accès.

### <a name="for-the-data-box-gateway-device"></a>Pour l’appareil Data Box Gateway

Avant de déployer un appareil virtuel, assurez-vous que :

- Vous avez accès à un système hôte exécutant Hyper-V sur Windows Server 2012 R2 ou une version ultérieure ou sur VMware (ESXi 6.0, 6.5 ou 6.7) et permettant de provisionner un appareil.
- Le système hôte est en mesure de dédier les ressources suivantes pour approvisionner votre appareil virtuel Data Box :
  
  - Au moins 4 processeurs virtuels.
  - Au moins 8 Go de RAM.
  - Une interface réseau.
  - Un disque de système d’exploitation de 250 Go.
  - Un disque virtuel de 2 To pour les données système.

### <a name="for-the-datacenter-network"></a>Pour le réseau du centre de données

Avant de commencer, assurez-vous que :

- Le réseau de votre centre de données est configuré conformément à la configuration réseau requise pour votre appareil Data Box Gateway. Pour plus d’informations, consultez [Configuration requise pour Data Box Gateway](data-box-gateway-system-requirements.md).

- Pour utiliser votre appareil Data Box Gateway dans des conditions normales, vous devez avoir :

    - Une bande passante de chargement de minimum 10 Mbits/s pour s’assurer que l’appareil reste à jour.
    - Une bande passante de chargement et de téléchargement de minimum 20 Mbits/s dédiée pour transférer des fichiers.

## <a name="create-a-new-resource"></a>Créer une nouvelle ressource

Si vous disposez d’une ressource Data Box Gateway existante pour gérer vos appareils virtuels, ignorez cette étape et accédez à [Obtenir la clé d’activation](#get-the-activation-key).

Pour créer une ressource Data Box Gateway, effectuez les étapes suivantes dans le portail Azure.

1. Utilisez vos informations d’identification Microsoft Azure pour vous connecter à l’un de ces portails :

    - Au portail Azure à cette URL : [https://portal.azure.com](https://portal.azure.com)
    - Au portail Azure Government à cette URL : [https://portal.azure.us](https://portal.azure.us). Pour des détails, accédez à [Se connecter à Azure Government à l’aide du portail](https://docs.microsoft.com/azure/azure-government/documentation-government-get-started-connect-with-portal).
    
2. Sélectionnez **+ Créer une ressource**.

    ![Bouton Créer une ressource dans Azure Data Box Gateway](media/data-box-gateway-deploy-prep/data-box-gateway-create-a-resource.png)

3. Tapez **Data Box Gateway** dans la zone de recherche, puis appuyez sur Entrée.

    ![Rechercher le service Data Box Gateway](media/data-box-gateway-deploy-prep/data-box-gateway-search-box.png)

    Sélectionnez ensuite **Azure Data Box Gateway**.

    ![Sélectionner la référence SKU Data Box Gateway](media/data-box-gateway-deploy-prep/data-box-gateway-sku.png)

4. Sélectionnez **Create** (Créer).

    ![Cliquer sur Créer pour créer la ressource Data Box Gateway](media/data-box-gateway-deploy-prep/data-box-gateway-create.png)

5. Sous l’onglet **Général** :

    Entrez ou sélectionnez les **détails du projet** suivants.
    
    |Paramètre  |Valeur  |
    |---------|---------|
    |Abonnement    |Sélectionnez l’abonnement que vous souhaitez utiliser pour l’appareil Data Box Gateway. L’abonnement est lié à votre compte de facturation.|
    |Resource group  |Sélectionnez un groupe existant ou créez-en un.<br>Obtenez plus d’informations sur les [groupes de ressources Azure](../azure-resource-manager/management/overview.md).|

   Entrez ou sélectionnez les **détails de l’instance** suivants.

    |Paramètre  |Valeur  |
    |---------|---------|
    |Nom   |Entrez un nom reconnaissable pour identifier la ressource.<br>Le nom doit comporter entre 2 et 50 caractères, et être constitué uniquement de lettres, de chiffres et de traits d’union. <br> Il doit commencer et se terminer par une lettre ou un chiffre. |
    |Région  |Sélectionnez la région dans laquelle vous souhaitez déployer la ressource. Choisissez un endroit proche de la région géographique où vous souhaitez déployer votre appareil. <br> Pour obtenir la liste complète des régions où les ressources Data Base Gateway/Azure Stack Edge sont disponibles, consultez [Disponibilité des produits Azure par région](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all). <br> Pour Azure Government, toutes les régions administratives listées dans les [Régions Azure](https://azure.microsoft.com/global-infrastructure/regions/) sont disponibles.|

   Ensuite, sélectionnez **Vérifier + créer** pour passer en revue votre commande.

   ![Entrée des détails du projet et de l’instance pour une commande Data Box Gateway](media/data-box-gateway-deploy-prep/data-box-gateway-basics.png)

7. Sous l’onglet **Vérifier + créer**, passez en revue les **Détails de la tarification**, les **Conditions d’utilisation** et les détails de votre ressource. Sélectionnez **Create** (Créer).

    ![Révision des détails de la ressource Data Box Gateway](media/data-box-gateway-deploy-prep/data-box-gateway-resource-review-create.png)

La création de la ressource prend quelques minutes. Un message vous informe que la ressource a été créée et déployée. Sélectionnez **Accéder à la ressource**.

![Commande Data Box Gateway terminée](media/data-box-gateway-deploy-prep/data-box-gateway-completed-order.png)

## <a name="download-the-virtual-device-image"></a>Télécharger l'image de l’appareil virtuel

Une fois que la ressource Data Box Gateway est créée, téléchargez l'image de l’appareil virtuel appropriée pour approvisionner un appareil virtuel sur le système hôte. Les images d’appareil virtuel sont spécifiques à un système d’exploitation.

> [!IMPORTANT]
> Le logiciel en cours d’exécution sur Data Box Gateway peut seulement être utilisé avec la ressource Data Box Gateway.

Suivez ces étapes dans le [portail Azure](https://portal.azure.com/) pour télécharger une image d’appareil virtuel.

1. Dans la ressource que vous avez créée, sélectionnez **Vue d’ensemble**. Si vous disposez d’une ressource Azure Data Box Gateway existante, sélectionnez sur la ressource, puis accédez à **Vue d’ensemble**. Sélectionnez **Configuration de l’appareil**.

    ![Nouvelle ressource Data Box Gateway](media/data-box-gateway-deploy-prep/data-box-gateway-resource-created.png)

2. Sur la vignette **Télécharger l’image**, sélectionnez l’image d’appareil virtuel correspondant au système d’exploitation sur le serveur hôte utilisé pour provisionner la machine virtuelle. Les fichiers image ont une taille d’environ 5,6 Go.
   
   * [VHD pour Hyper-V sur Windows Server 2012 R2 et versions ultérieures](https://aka.ms/dbe-vhdx-2012)
   * [VMDK pour VMware ESXi 6.0, 6.5 ou 6.7](https://aka.ms/dbe-vmdk).

    ![Télécharger l’image d’appareil virtuel Data Box Gateway](media/data-box-gateway-deploy-prep/data-box-gateway-download-image.png)

5. Téléchargez et décompressez le fichier sur un lecteur local et notez l’emplacement du fichier décompressé.


## <a name="get-the-activation-key"></a>Obtenir la clé d'activation

Une fois que la ressource Data Box Gateway est active et en cours d’exécution, vous devez obtenir la clé d’activation. Cette clé sert à activer votre appareil Data Box Gateway et à le connecter à la ressource. Vous pouvez obtenir cette clé maintenant, lorsque vous vous trouvez dans le Portail Azure.

1. Sélectionnez la ressource que vous avez créée, puis **Vue d’ensemble**. Dans la **Configuration de l’appareil**, accédez à la vignette **Configurer et activer**.

    ![Vignette Configurer et activer](media/data-box-gateway-deploy-prep/data-box-gateway-configure-activate.png)

2. Sélectionnez **Générer une clé** pour créer une clé d’activation. Sélectionnez l’icône de copie pour copier la clé et l’enregistrer pour une utilisation ultérieure.

    ![Obtenir une clé d’activation](media/data-box-gateway-deploy-prep/get-activation-key.png)

> [!IMPORTANT]
> - La clé d’activation expire 3 jours après sa création.
> - Si la clé expire, générez une nouvelle clé. L’ancienne clé n'est plus valide.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez approfondi vos connaissances sur Data Box Gateway et avez notamment appris à :

> [!div class="checklist"]
> * Créer une nouvelle ressource
> * Télécharger l'image de l’appareil virtuel
> * Obtenir la clé d'activation

Passez au didacticiel suivant pour découvrir comment approvisionner une machine virtuelle pour Data Box Gateway. Selon votre système d'exploitation hôte, consultez les instructions détaillées dans :

> [!div class="nextstepaction"]
> [Approvisionner une instance Data Box Gateway dans Hyper-V](./data-box-gateway-deploy-provision-hyperv.md)

OR

> [!div class="nextstepaction"]
> [Approvisionner une instance Data Box Gateway dans VMware](./data-box-gateway-deploy-provision-vmware.md)


