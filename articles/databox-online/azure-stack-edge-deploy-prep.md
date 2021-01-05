---
title: 'Tutoriel : Préparer le portail Azure et l’environnement du centre de données pour déployer Azure Stack Edge Pro | Microsoft Docs'
description: Le premier tutoriel sur le déploiement d’Azure Stack Edge Pro implique la préparation du portail Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 11/11/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to prepare the portal to deploy Azure Stack Edge Pro so I can use it to transfer data to Azure.
ms.openlocfilehash: 3943caba5249432b3a0a4b7c2e63b2b818e2b7a1
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575701"
---
# <a name="tutorial-prepare-to-deploy-azure-stack-edge-pro"></a>Tutoriel : Préparer le déploiement d’Azure Stack Edge Pro  

Ce tutoriel est le premier d’une série de tutoriels permettant de déployer entièrement Azure Stack Edge Pro. Il explique comment préparer le portail Azure pour déployer une ressource Azure Stack Edge.

Vous avez besoin de privilèges d’administrateur pour terminer le processus d’installation et de configuration. La préparation du portail prend moins de 10 minutes.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Créer une nouvelle ressource
> * Obtenir la clé d'activation

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="get-started"></a>Bien démarrer

Pour déployer Azure Stack Edge Pro, consultez les tutoriels suivants dans l’ordre indiqué.

| **#** | **Dans cette étape** | **Utilisez ces documents** |
| --- | --- | --- | 
| 1. |**[Préparer le portail Azure pour Azure Stack Edge Pro](azure-stack-edge-deploy-prep.md)** |Créez et configurez votre ressource Azure Stack Edge avant d’installer un appareil physique Azure Stack Edge. |
| 2. |**[Installer Azure Stack Edge Pro](azure-stack-edge-deploy-install.md)**|Déballez, montez en rack et branchez l'appareil physique Azure Stack Edge Pro.  |
| 3. |**[Connecter, configurer et activer Azure Stack Edge Pro](azure-stack-edge-deploy-connect-setup-activate.md)** |Connectez-vous à l’interface utilisateur web locale, terminez l’installation de l’appareil et activez-le. L’appareil est prêt à configurer des partages SMB ou NFS.  |
| 4. |**[Transférer des données avec Azure Stack Edge Pro](azure-stack-edge-deploy-add-shares.md)** |Ajoutez des partages et connectez-vous à ces derniers via SMB ou NFS. |
| 5. |**[Transformer des données avec Azure Stack Edge Pro](azure-stack-edge-deploy-configure-compute.md)** |Configurez des modules de calcul sur l’appareil pour transformer les données lors de leur déplacement vers Azure. |

Vous pouvez maintenant commencer à configurer le portail Azure.

## <a name="prerequisites"></a>Prérequis

Voici les prérequis relatifs à la configuration de votre ressource Azure Stack Edge, de votre appareil Azure Stack Edge Pro et du réseau du centre de données.

### <a name="for-the-azure-stack-edge-resource"></a>Pour la ressource Azure Stack Edge

Avant de commencer, assurez-vous que :

* Votre abonnement Microsoft Azure est activé pour une ressource Azure Stack Edge. Vérifiez que vous avez utilisé un abonnement pris en charge tel que [Contrat Entreprise (EA) Microsoft](https://azure.microsoft.com/overview/sales-number/), [Fournisseur de solutions cloud (CSP)](/partner-center/azure-plan-lp) ou [Microsoft Azure Sponsorship](https://azure.microsoft.com/offers/ms-azr-0036p/). Les abonnements de type Paiement à l’utilisation ne sont pas pris en charge.

* Vous disposez d’un accès Propriétaire ou Contributeur au niveau du groupe de ressources pour les ressources Azure Stack Edge / Data Box Gateway, IoT Hub et Stockage Azure.

  * Vous devez être **propriétaire** au niveau de l’abonnement pour accorder l’accès contributeur. Pour accorder un accès contributeur à quelqu’un d’autre, dans le portail Azure, accédez à **Tous les services** > **Abonnements** > **Contrôle d’accès (IAM)**  >  **+Ajouter** > **Ajouter une attribution de rôle**. Pour plus d’informations, consultez [Didacticiel : Accorder un accès utilisateur aux ressources Azure à l’aide du portail Azure](../role-based-access-control/quickstart-assign-role-user-portal.md).

  * Pour créer une ressource Azure Stack Edge / Data Box Gateway, vous devez disposer d’autorisations Contributeur (ou supérieures) au niveau du groupe de ressources. Vous devez également être sûr que le fournisseur de ressources `Microsoft.DataBoxEdge` est inscrit. Pour plus d’informations sur l’inscription d’un fournisseur de ressources, consultez [Inscrire un fournisseur de ressources](azure-stack-edge-manage-access-power-connectivity-mode.md#register-resource-providers).
  * Pour créer une ressource IoT Hub, vérifiez que le fournisseur Microsoft.Devices est bien inscrit. Pour plus d’informations sur l’inscription, consultez [Inscrire un fournisseur de ressources](azure-stack-edge-manage-access-power-connectivity-mode.md#register-resource-providers).
  * Pour créer une ressource de compte de stockage, là encore, vous avez besoin d’autorisations Contributeur (ou supérieures) au niveau du groupe de ressources. Par défaut, le stockage Azure est un fournisseur de ressources inscrit.
* Vous avez un accès Administrateur ou Utilisateur à l’API Graph Azure Active Directory. Pour plus d’informations, consultez [API Graph Azure Active Directory](/previous-versions/azure/ad/graph/howto/azure-ad-graph-api-permission-scopes#default-access-for-administrators-users-and-guest-users-).
* Vous disposez d’un compte de stockage Microsoft Azure doté d’informations d’identification d’accès.
* Vous n’êtes pas bloqué par une stratégie Azure configurée par votre administrateur système. Pour plus d’informations sur les stratégies, consultez [Démarrage rapide : Création d’une affectation de stratégie pour identifier les ressources non conformes](../governance/policy/assign-policy-portal.md).

### <a name="for-the-azure-stack-edge-pro-device"></a>Pour l'appareil Azure Stack Edge Pro

Avant de déployer un appareil physique, assurez-vous que :

* Vous avez consulté les informations relatives à la sécurité qui étaient incluses dans le colis d’expédition.
* Dans votre centre de données, un emplacement 1U est disponible dans un rack standard 19 pouces pour le montage en rack de l’appareil.
* Vous avez accès à une surface de travail plane, stable et de niveau sur laquelle poser l’appareil en sécurité.
* Le site où vous envisagez de configurer l’appareil dispose d’une alimentation secteur standard provenant d’une source indépendante ou d’une unité de distribution de l’alimentation (PDU) en rack avec un onduleur (UPS).
* Vous avez accès à un appareil physique.

### <a name="for-the-datacenter-network"></a>Pour le réseau du centre de données

Avant de commencer, assurez-vous que :

* Le réseau de votre centre de données est configuré conformément à la configuration réseau nécessaire pour votre appareil Azure Stack Edge Pro. Pour plus d'informations, consultez [Configuration système nécessaire pour Azure Stack Edge Pro](azure-stack-edge-system-requirements.md).

* Pour utiliser votre appareil Azure Stack Edge Pro dans des conditions normales, vous avez :

  * Une bande passante de chargement de minimum 10 Mbits/s pour s’assurer que l’appareil reste à jour.
  * Une bande passante de chargement et de téléchargement de minimum 20 Mbits/s dédiée pour transférer des fichiers.

## <a name="create-a-new-resource"></a>Créer une nouvelle ressource

Si vous disposez d’une ressource Azure Stack Edge existante pour gérer votre appareil physique, ignorez cette étape et passez à la section [Obtenir la clé d’activation](#get-the-activation-key).

Pour créer une ressource Azure Stack Edge, suivez ces étapes dans le portail Azure.

1. Utilisez vos informations d’identification Microsoft Azure pour vous connecter : 

    - Au portail Azure à cette URL : [https://portal.azure.com](https://portal.azure.com)
    - Ou bien, au portail Azure Government à cette URL : [https://portal.azure.us](https://portal.azure.us) Pour plus d’informations, accédez à [Se connecter à Azure Government à l’aide du portail](../azure-government/documentation-government-get-started-connect-with-portal.md).

2. Dans le volet de gauche, sélectionnez **+ Créer une ressource**. Recherchez et sélectionnez **Azure Stack Edge / Data Box Gateway**. Sélectionnez **Create** (Créer).
3. Sélectionnez l'abonnement que vous souhaitez utiliser pour l'appareil Azure Stack Edge Pro. Sélectionnez la région dans laquelle vous souhaitez déployer la ressource Azure Stack Edge. Pour obtenir la liste complète des régions où la ressource Azure Stack Edge est disponible, consultez [Disponibilité des produits Azure par région](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all).

    Choisissez l’emplacement le plus proche de la région géographique dans laquelle vous souhaitez déployer votre appareil. La région stocke les métadonnées uniquement pour la gestion des appareils. Les données réelles peuvent être stockées dans n’importe quel compte de stockage.
    
    Dans l’option **Azure Stack Edge Pro**, sélectionnez **Créer**.

    ![Rechercher le service Azure Stack Edge](media/azure-stack-edge-deploy-prep/data-box-edge-sku.png)

3. Sous l’onglet **Bases**, entrez ou sélectionnez les **détails du projet** suivants.
    
    |Paramètre  |Value  |
    |---------|---------|
    |Abonnement    |Ce champ est automatiquement renseigné en fonction de la sélection antérieure. L’abonnement est lié à votre compte de facturation. |
    |Resource group  |Sélectionnez un groupe existant ou créez-en un.<br>Obtenez plus d’informations sur les [groupes de ressources Azure](../azure-resource-manager/management/overview.md).     |

4. Entrez ou sélectionnez les **détails de l’instance** suivants.

    |Paramètre  |Valeur  |
    |---------|---------|
    |Nom   | Entrez un nom reconnaissable pour identifier la ressource.<br>Le nom doit être compris entre 2 et 50 caractères, et se composer uniquement de lettres, de chiffres et de traits d’union.<br> Le nom doit commencer et se terminer par une lettre ou un chiffre.        |
    |Région     |Pour obtenir la liste complète des régions où la ressource Azure Stack Edge est disponible, consultez [Disponibilité des produits Azure par région](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all). Si vous utilisez Azure Government, toutes les régions administratives sont disponibles, comme indiqué dans [Régions Azure](https://azure.microsoft.com/global-infrastructure/regions/).<br> Choisissez l’emplacement le plus proche de la région géographique dans laquelle vous souhaitez déployer votre appareil.|

    ![Détails du projet et de l’instance](media/azure-stack-edge-deploy-prep/data-box-edge-resource.png)

5. Sélectionnez **Suivant : Adresse de livraison**.

    - Si vous disposez déjà d'un appareil, cochez la case **J'ai un appareil Azure Stack Edge Pro**.
    - Si vous commandez un nouvel appareil, entrez le nom du contact, la société, l’adresse de livraison de l’appareil et les informations de contact.

    ![Adresse de livraison du nouvel appareil](media/azure-stack-edge-deploy-prep/data-box-edge-resource1.png)

6. Sélectionnez **Suivant : Vérifier + créer**.

7. Sous l’onglet **Vérifier + créer**, passez en revue les **Détails de la tarification**, les **Conditions d’utilisation** et les détails de votre ressource. Cochez la case **J’ai pris connaissance des conditions de confidentialité**.

    ![Passer en revue les détails de la ressource Azure Stack Edge et les conditions de confidentialité](media/azure-stack-edge-deploy-prep/data-box-edge-resource2.png)

8. Sélectionnez **Create** (Créer).

La création de la ressource prend quelques minutes. Un message vous informe que la ressource a été créée et déployée. Sélectionnez **Accéder à la ressource**.

![Accéder à la ressource Azure Stack Edge](media/azure-stack-edge-deploy-prep/data-box-edge-resource3.png)

Une fois la commande passée, Microsoft l’examine et vous communique (par e-mail) les détails de l’expédition.

![Notification relative à l’examen de la commande Azure Stack Edge Pro](media/azure-stack-edge-deploy-prep/data-box-edge-resource4.png)

## <a name="get-the-activation-key"></a>Obtenir la clé d'activation

Une fois que la ressource Azure Stack Edge est active et en cours d’exécution, vous devez obtenir la clé d’activation. Cette clé sert à activer votre appareil Azure Stack Edge Pro et à le connecter à la ressource. Vous pouvez obtenir cette clé maintenant, lorsque vous vous trouvez dans le Portail Azure.

1. Sélectionnez la ressource que vous avez créée. Sélectionnez **Vue d’ensemble**, puis **Configuration de l’appareil**.

    ![Sélectionner Configuration de l’appareil](media/azure-stack-edge-deploy-prep/data-box-edge-select-devicesetup.png)

2. Sur la vignette **Activer**, sélectionnez **Générer une clé** pour créer une clé d’activation. Sélectionnez l’icône de copie pour copier la clé et l’enregistrer pour une utilisation ultérieure.

    ![Obtenir une clé d’activation](media/azure-stack-edge-deploy-prep/get-activation-key.png)

> [!IMPORTANT]
>
> * La clé d’activation expire 3 jours après sa création.
> * Si la clé expire, générez une nouvelle clé. L’ancienne clé n'est plus valide.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez approfondi vos connaissances sur Azure Stack Edge Pro et avez notamment appris à :

> [!div class="checklist"]
>
> * Créer une nouvelle ressource
> * Obtenir la clé d'activation

Passez au tutoriel suivant pour apprendre à installer Azure Stack Edge Pro.

> [!div class="nextstepaction"]
> [Installer Azure Stack Edge Pro](./azure-stack-edge-deploy-install.md)