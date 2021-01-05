---
title: Créer des images de machine virtuelle pour votre appareil Azure Stack Edge Pro avec GPU
description: Décrit comment créer des images de machine virtuelle Linux ou Windows à utiliser avec votre appareil Azure Stack Edge avec GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 12/08/2020
ms.author: alkohli
ms.openlocfilehash: fd55110bbfaed72051bfc388692421cd8209609d
ms.sourcegitcommit: 799f0f187f96b45ae561923d002abad40e1eebd6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/24/2020
ms.locfileid: "97763659"
---
# <a name="create-custom-vm-images-for-your-azure-stack-edge-pro-device"></a>Créer des images de machine virtuelle personnalisées pour votre appareil Azure Stack Edge Pro

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

Pour déployer des machines virtuelles sur votre appareil Azure Stack Edge Pro, vous devez être capable de créer des images de machine virtuelle personnalisées que vous pouvez utiliser pour créer des machines virtuelles. Cet article décrit les étapes nécessaires à la création d’images personnalisées de machines virtuelles Linux ou Windows, images que vous pouvez utiliser pour déployer des machines virtuelles sur votre appareil Azure Stack Edge Pro.

## <a name="vm-image-workflow"></a>Workflow d’image de machine virtuelle

Le workflow nécessite la création d’une machine virtuelle dans Azure, la personnalisation de la machine virtuelle, la généralisation, puis le téléchargement du disque dur virtuel correspondant à cette machine virtuelle. Ce disque dur virtuel généralisé est chargé sur Azure Stack Edge Pro, le disque managé est créé à partir de ce disque dur virtuel, l’image est créée à partir du disque managé et, pour finir, les machines virtuelles sont créées à partir de cette image.   

Pour plus d’informations, consultez [Déployer une machine virtuelle sur votre appareil Azure Stack Edge Pro à l’aide d’Azure PowerShell](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md).


## <a name="create-a-windows-custom-vm-image"></a>Créer une image de machine virtuelle personnalisée Windows

Effectuez les étapes suivantes pour créer une image de machine virtuelle Windows.

1. Créez une machine virtuelle Windows. Pour plus d'informations, consultez [Tutoriel : Créer et gérer des machines virtuelles Windows avec Azure PowerShell](../virtual-machines/windows/tutorial-manage-vm.md)

2. Téléchargez un disque de système d’exploitation existant.

    - Suivez les étapes décrites dans [Télécharger un disque dur virtuel](../virtual-machines/windows/download-vhd.md).

    - Utilisez la commande `sysprep` suivante à la place de ce qui est décrit dans la procédure précédente.
    
        `c:\windows\system32\sysprep\sysprep.exe /oobe /generalize /shutdown /mode:vm`
   
       Vous pouvez également consulter [Vue d’ensemble de Sysprep (préparation du système)](/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview).

Utilisez ce disque dur virtuel pour créer et déployer maintenant une machine virtuelle sur votre appareil Azure Stack Edge Pro.

## <a name="create-a-linux-custom-vm-image"></a>Créer une image de machine virtuelle personnalisée Linux

Effectuez les étapes suivantes pour créer une image de machine virtuelle Linux.

1. Créez une machine virtuelle Linux. Pour plus d'informations, consultez [Tutoriel : Créer et gérer des machines virtuelles Linux avec l’interface Azure CLI](../virtual-machines/linux/tutorial-manage-vm.md).

1. Annulez l’approvisionnement de la machine virtuelle. Utilisez l’agent de machine virtuelle Azure pour supprimer les fichiers et les données propres à la machine. Utilisez la commande `waagent` avec le paramètre `-deprovision+user` sur votre machine virtuelle Linux source. Pour plus d’informations, consultez [Comprendre et utiliser l’agent Linux Azure](../virtual-machines/extensions/agent-linux.md).

    1. Connectez-vous à votre machine virtuelle Linux avec un client SSH.
    2. Dans la fenêtre SSH, entrez la commande suivante :
       
        ```bash
        sudo waagent -deprovision+user
        ```
       > [!NOTE]
       > Exécutez uniquement cette commande sur une machine virtuelle que vous allez capturer en tant qu’image. Cette commande ne garantit pas que l’image est exempte de toute information sensible ou qu’elle convient pour la redistribution. Le paramètre `+user` supprime également le dernier compte d’utilisateur provisionné. Pour garder les informations d’identification du compte d’utilisateur dans la machine virtuelle, utilisez uniquement `-deprovision`.
     
    3. Tapez **Y** pour continuer. Vous pouvez ajouter le paramètre `-force` pour éviter cette étape de confirmation.
    4. Une fois la commande exécutée, entrez **exit** pour fermer le client SSH.  La machine virtuelle est toujours en cours d’exécution à ce stade.


1. [Téléchargez un disque de système d’exploitation existant](../virtual-machines/linux/download-vhd.md).

Utilisez ce disque dur virtuel pour créer et déployer maintenant une machine virtuelle sur votre appareil Azure Stack Edge Pro. Vous pouvez utiliser les deux images de la Place de marché Azure suivantes pour créer des images personnalisées Linux :

|Nom de l’élément  |Description  |Serveur de publication  |
|---------|---------|---------|
|[Serveur Ubuntu](https://azuremarketplace.microsoft.com/marketplace/apps/canonical.ubuntuserver) |Ubuntu Server est le Linux le plus populaire au monde pour les environnements cloud.|Canonical|
|[Debian 8 "Jessie"](https://azuremarketplace.microsoft.com/marketplace/apps/credativ.debian) |Debian GNU/Linux est l’une des distributions Linux les plus populaires.     |credativ|

Pour obtenir la liste complète des images de la Place de Marché Azure qui peuvent fonctionner (non testées pour le moment), accédez à [Éléments disponibles sur la Place de Marché Azure pour Azure Stack Hub](/azure-stack/operator/azure-stack-marketplace-azure-items?view=azs-1910).


## <a name="next-steps"></a>Étapes suivantes

[Déployer des machines virtuelles sur votre appareil Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md).