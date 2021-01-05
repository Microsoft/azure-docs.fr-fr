---
title: Déployer des machines virtuelles sur Azure Stack Edge Pro via le portail Azure
description: Découvrez comment déployer des machines virtuelles sur Azure Stack Edge Pro via le portail Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 11/02/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to configure compute on Azure Stack Edge Pro device so I can use it to transform the data before sending it to Azure.
ms.openlocfilehash: 04ba9777fe9e8470e6f02c83f3996d098023e05e
ms.sourcegitcommit: 799f0f187f96b45ae561923d002abad40e1eebd6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/24/2020
ms.locfileid: "97763453"
---
# <a name="deploy-vms-on-your-azure-stack-edge-pro-gpu-device-via-the-azure-portal"></a>Déployer des machines virtuelles sur Azure Stack Edge Pro avec GPU via le portail Azure

Vous pouvez créer et gérer des machines virtuelles sur un appareil Azure Stack Edge à l’aide du portail Azure, des modèles, des cmdlets Azure PowerShell et via des scripts Azure CLI/Python. Cet article explique comment créer et gérer une machine virtuelle sur votre appareil Azure Stack Edge à l’aide du portail Azure. 

Cet article s'applique aux appareils Azure Stack Edge Pro GPU, Azure Stack Edge Pro R et Azure Stack Edge Mini R. 

## <a name="vm-deployment-workflow"></a>Workflow du déploiement de machine virtuelle

Voici une synthèse globale du workflow de déploiement :

1. Activez une interface réseau pour le calcul sur votre appareil Azure Stack Edge. Cela crée un commutateur virtuel sur l’interface réseau spécifiée.
1. Activez la gestion cloud des machines virtuelles depuis le portail Azure.
1. Chargez un disque dur virtuel dans un compte Stockage Azure à l’aide de l’Explorateur de stockage. 
1. Utilisez le disque dur virtuel chargé pour télécharger le disque dur virtuel sur l’appareil et l’utiliser pour créer une image de machine virtuelle. 
1. Utilisez les ressources créées durant les étapes précédentes :
    1. Image de machine virtuelle que vous avez créée.
    1. VSwitch associé à l’interface réseau sur laquelle vous avez activé le calcul.
    1. Sous-réseau associé à VSwitch.

    Créez ou spécifiez les ressources suivantes en ligne :
    1. Nom de la machine virtuelle, choisissez une taille de machine virtuelle prise en charge, informations d’identification de connexion pour la machine virtuelle. 
    1. Créez des disques de données ou attachez des disques de données existants.
    1. Configurez une adresse IP statique ou dynamique pour la machine virtuelle. Si vous fournissez une adresse IP statique, choisissez à partir d’une adresse IP libre dans la plage de sous-réseau de l’interface réseau activée pour le calcul.

    Utilisez les ressources ci-dessus pour créer une machine virtuelle.


## <a name="prerequisites"></a>Prérequis

Avant de commencer à créer et à gérer des machines virtuelles sur votre appareil via le portail Azure, assurez-vous de ce qui suit :

1. Vous avez appliqué les paramètres réseau sur votre appareil Azure Stack Edge Pro comme décrit dans [Étape 1 : Configurer l’appareil Azure Stack Edge Pro](azure-stack-edge-j-series-connect-resource-manager.md#step-1-configure-azure-stack-edge-pro-device).

    1. Vous avez activé une interface réseau pour le calcul. Cette adresse IP d’interface réseau sert à créer un commutateur virtuel pour le déploiement de la machine virtuelle. Dans l’interface utilisateur locale de votre appareil, accédez à **Calcul**. Sélectionnez l’interface réseau que vous utiliserez pour créer un commutateur virtuel.

        > [!IMPORTANT] 
        > Vous ne pouvez configurer qu’un seul port pour le calcul.

    1. Activez le calcul sur l’interface réseau. Azure Stack Edge Pro crée et gère un commutateur virtuel correspondant à cette interface réseau.

1. Vous avez accès à un disque dur virtuel Windows ou Linux que vous utiliserez pour créer l’image de la machine virtuelle que vous souhaitez créer.

## <a name="deploy-a-vm"></a>Déployer une machine virtuelle

Pour créer une machine virtuelle sur votre appareil Azure Stack Edge, procédez comme suit.

### <a name="add-a-vm-image"></a>Ajouter une image de machine virtuelle

1. Chargez de disque dur virtuel sur un compte de stockage Azure. Suivez la procédure décrite dans [Charger un disque dur virtuel à l’aide de l’Explorateur Stockage Azure](../devtest-labs/devtest-lab-upload-vhd-using-storage-explorer.md).

1. Dans le portail Azure, accédez à la ressource Azure Stack Edge de votre appareil Azure Stack Edge. Accédez à **Computing en périphérie > Machines virtuelles**.

    ![Ajouter une image de machine virtuelle 1](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-1.png)

1. Sélectionnez **Machines virtuelles** pour accéder à la page **Vue d’ensemble**. **Activez** la gestion cloud des machines virtuelles.
    ![Ajouter une image de machine virtuelle 2](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-2.png)

1. La première étape consiste à ajouter une image de machine virtuelle. Au cours de l’étape précédente, vous chargé un disque dur virtuel dans le compte de stockage. Vous allez utiliser ce disque dur virtuel pour créer une image de machine virtuelle.

    Sélectionnez **Ajouter une image** pour télécharger le disque dur virtuel à partir du compte de stockage et l’ajouter à l’appareil. Le processus de téléchargement prend plusieurs minutes selon la taille du disque dur virtuel et de la bande passante Internet disponible pour le téléchargement. 

    ![Ajouter une image de machine virtuelle 3](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-3.png)

1. Dans le panneau **Ajouter une image**, entrez les paramètres suivants. Sélectionnez **Ajouter**.


    |Paramètre  |Description  |
    |---------|---------|
    |Télécharger à partir d’un objet blob de stockage    |Accédez à l’emplacement de l’objet blob de stockage dans le compte de stockage où vous avez chargé le disque dur virtuel.         |
    |Télécharger vers    | Défini automatiquement sur l’appareil actuel sur lequel vous déployez la machine virtuelle.        |
    |Enregistrer l’image sous      | Nom de l’image de machine virtuelle que vous créez à partir du disque dur virtuel chargé dans le compte de stockage.        |
    |Type de système d’exploitation     |Sélectionnez Windows ou Linux en tant que système d’exploitation du disque dur virtuel que utiliserez pour créer l’image de machine virtuelle.         |
   

    ![Ajouter une image de machine virtuelle 4](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-6.png)

1. Le disque dur virtuel est téléchargé et l’image de machine virtuelle est créée. Cette opération prend plusieurs minutes. Une notification s’affiche une fois l’image de machine virtuelle créée.

    ![Ajouter une image de machine virtuelle 5](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-8.png)


1. Une fois créée, l’image de machine virtuelle est ajoutée à la liste des images dans le panneau **Images**.
    ![Ajouter une image de machine virtuelle 6](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-9.png)

    Le panneau **Déploiements** se met à jour pour indiquer l’état du déploiement.

    ![Ajouter une image de machine virtuelle 7](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-10.png)

    L’image nouvellement ajoutée s’affiche également dans la page **Vue d’ensemble**.
    ![Ajouter une image de machine virtuelle 8](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-11.png)


### <a name="add-a-vm"></a>Ajouter une machine virtuelle

Suivez la procédure ci-dessous pour créer une machine virtuelle après avoir créé une image de machine virtuelle.

1. Dans la page **Vue d’ensemble**, sélectionnez **Ajouter une machine virtuelle**.

    ![Ajouter une machine virtuelle 1](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-1.png)

1. Sous l’onglet **Informations de base**, entrez les paramètres suivants.


    |Paramètre |Description  |
    |---------|---------|
    |Nom de la machine virtuelle     |         |
    |Image     | Effectuez une sélection parmi les images de machine virtuelle disponibles sur l’appareil.        |
    |Taille     | Effectuez une sélection parmi les [Tailles de machine virtuelle prises en charge](azure-stack-edge-gpu-virtual-machine-sizes.md).        |
    |Nom d’utilisateur     | Utilisez le nom d’utilisateur par défaut *azureuser*.        |
    |Type d'authentification    | Sélectionnez une clé publique SSH ou un mot de passe défini par l’utilisateur.       |
    |Mot de passe     | Entrez un mot de passe pour vous connecter à la machine virtuelle. Le mot de passe doit contenir au moins 12 caractères et satisfaire aux [exigences de complexité ](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm)définies.        |
    |Confirmer le mot de passe    | Entrez de nouveau le mot de passe.        |


    ![Ajouter une machine virtuelle 2](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-basics-1.png)

    Sélectionnez **Suivant : Disques**.

1. Sous l’onglet **Disques**, vous allez attacher des disques à votre machine virtuelle. 
    
    1. Vous pouvez choisir entre **Créer et attacher un disque** ou **Attacher un disque existant**.

        ![Ajouter une machine virtuelle 3](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-disks-1.png)

    1. Sélectionnez **Créer et attacher un disque**. Dans le panneau **Créer un disque**, indiquez le nom du disque et sa taille en Gio.

        ![Ajouter une machine virtuelle 4](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-disks-2.png)

    1.  Répétez les étapes ci-dessus pour ajouter d’autres disques. Une fois les disques créés, ils s’affichent sous l’onglet **Disques**.

        ![Ajouter une machine virtuelle 5](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-disks-3.png)

        Sélectionnez **Suivant : Réseau**.

1. Sous l’onglet **Mise en réseau**, vous allez configurer la connectivité réseau de votre machine virtuelle.

    
    |Paramètre  |Description |
    |---------|---------|
    |Réseau virtuel    | Dans la liste déroulante, sélectionnez le commutateur virtuel créé sur votre appareil Azure Stack Edge lorsque vous avez activé le calcul sur l’interface réseau.    |
    |Subnet     | Ce champ est automatiquement renseigné avec le sous-réseau associé à l’interface réseau sur laquelle vous avez activé le calcul.         |
    |Adresse IP     | Indiquez une adresse IP statique ou dynamique pour votre machine virtuelle. L’adresse IP statique doit correspondre à une adresse IP disponible dans la plage de sous-réseau spécifiée.        |

    ![Ajouter une machine virtuelle 6](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-networking-1.png)

    Sélectionnez **Suivant : Vérifier + créer**.

1. Sur l’onglet **Vérifier + Créer**, passez en revue les spécifications de la machine virtuelle, puis sélectionnez **Créer**.

    ![Ajouter une machine virtuelle 7](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-review-create-1.png)

1. La création de la machine virtuelle démarre et peut prendre jusqu’à 20 minutes. Vous pouvez accéder à **Déploiements** pour suivre la création de la machine virtuelle.

    ![Ajouter une machine virtuelle 8](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-deployments-page-1.png)

    
1. Une fois la machine virtuelle créée, la page **Vue d’ensemble** se met à jour pour afficher la nouvelle machine virtuelle.

    ![Ajouter une machine virtuelle 9](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-overview-page-1.png)

1. Sélectionnez la machine virtuelle que vous venez de créer pour accéder à **Machines virtuelles**.

    ![Ajouter une machine virtuelle 10](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-page-1.png)

    Sélectionnez la machine virtuelle pour afficher les détails correspondants. 

    ![Ajouter une machine virtuelle 11](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-details-1.png)

## <a name="connect-to-a-vm"></a>Se connecter à une machine virtuelle

Selon que vous avez créé une machine virtuelle Windows ou Linux, les étapes de connexion peuvent être différentes. Vous ne pouvez pas vous connecter aux machines virtuelles déployées sur votre appareil via le portail Azure. Vous devez effectuer suivre la procédure ci-dessous pour vous connecter à votre machine virtuelle Linux ou Windows.

### <a name="connect-to-linux-vm"></a>Se connecter à une machine virtuelle Linux

Suivez ces étapes pour vous connecter à une machine virtuelle Linux.

[!INCLUDE [azure-stack-edge-gateway-connect-vm](../../includes/azure-stack-edge-gateway-connect-virtual-machine-linux.md)]

### <a name="connect-to-windows-vm"></a>Se connecter à une machine virtuelle Windows

Suivez ces étapes pour vous connecter à une machine virtuelle Windows.

[!INCLUDE [azure-stack-edge-gateway-connect-vm](../../includes/azure-stack-edge-gateway-connect-virtual-machine-windows.md)]

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment administrer votre appareil Azure Stack Edge Pro, consultez [Administrer un appareil Azure Stack Edge Pro avec l’interface utilisateur web locale](azure-stack-edge-manage-access-power-connectivity-mode.md).
