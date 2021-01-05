---
title: Créer une offre de machine virtuelle Azure à partir d’une base approuvée, Place de marché Azure
description: Découvrez comment créer une offre de machine virtuelle à partir d’une base approuvée.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: emuench
ms.author: krsh
ms.date: 10/20/2020
ms.openlocfilehash: c28595458fea61f66b6930ce72fa0702e00aecd9
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93129294"
---
# <a name="how-to-create-a-virtual-machine-using-an-approved-base"></a>Création d’une machine virtuelle à l’aide d’une base approuvée

Cet article explique comment utiliser Azure pour créer une machine virtuelle contenant un système d’exploitation préconfiguré et approuvé. Si cela n’est pas compatible avec votre solution, il est possible de [créer et configurer une machine virtuelle locale](azure-vm-create-using-own-image.md) à l’aide d’un système d’exploitation approuvé, puis de la configurer et de la préparer pour le téléchargement, comme décrit dans [Préparer un disque dur virtuel Windows ou un VHDX à télécharger sur Azure](../virtual-machines/windows/prepare-for-upload-vhd-image.md).

> [!NOTE]
> Avant de commencer cette procédure, passez en revue les [exigences techniques](marketplace-virtual-machines.md#technical-requirements) pour les offres de machine virtuelle Azure, y compris les exigences de disque dur virtuel (VHD).

## <a name="select-an-approved-base-image"></a>Sélectionner une image de base approuvée

Sélectionnez l’une des images Windows ou Linux suivantes comme base.

### <a name="windows"></a>Windows

- [Windows Server](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/microsoftwindowsserver.windowsserver?tab=Overview)
- Microsoft SQL Server [2019](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftsqlserver.sql2019-ws2019?tab=Overview), [2014](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftsqlserver.sql2014sp3-ws2012r2?tab=Overview), [2012](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftsqlserver.sql2012sp4-ws2012r2?tab=Overview)

### <a name="linux"></a>Linux

Azure offre toute une gamme de distributions de Linux approuvées. Pour obtenir la liste actuelle, consultez [Linux sur les distributions approuvées par Azure](../virtual-machines/linux/endorsed-distros.md).

## <a name="create-vm-on-the-azure-portal"></a>Créer une machine virtuelle sur le portail Azure

1. Connectez-vous au [portail Azure](https://ms.portal.azure.com/).
2. Sélectionnez **Machines virtuelles**.
3. Sélectionnez **+ ajouter**  pour ouvrir l’écran **Créer une machine virtuelle** .
4. Sélectionnez l’image dans la liste déroulante ou sélectionnez **Parcourir toutes les images publiques et privées** pour rechercher ou parcourir toutes les images de machines virtuelles disponibles.
5. Pour créer une machine virtuelle **Gen 2** , accédez à l’onglet **avancé** et sélectionnez l’option **Gen 2** .

    :::image type="content" source="media/create-vm/vm-gen-option.png" alt-text="Sélectionnez GEN 1 ou Gen 2.":::

6. Sélectionnez la taille de la machine virtuelle à déployer.

    :::image type="content" source="media/create-vm/create-virtual-machine-sizes.png" alt-text="Sélectionnez une taille de machine virtuelle recommandée pour l’image sélectionnée.":::

7. Fournissez les autres informations requises pour créer la machine virtuelle.
8. Sélectionnez **Vérifier + créer** pour passer en revue vos choix. Lorsque le message **Validation réussie** s’affiche, sélectionnez **Créer**.

Azure commence le provisionnement de la machine virtuelle que vous avez spécifiée. Suivez sa progression en sélectionnant l’onglet **Machines virtuelles** dans le menu de gauche. Une fois créé, l’état de la machine virtuelle passe à **En cours d’exécution**.


## <a name="configure-the-vm"></a>Configurer la machine virtuelle

Cette section décrit comment dimensionner, mettre à jour et généraliser une machine virtuelle Azure. Ces étapes sont nécessaires pour préparer le déploiement de votre machine virtuelle sur la Place de marché Microsoft Azure.

### <a name="connect-to-your-vm"></a>Connexion à votre machine virtuelle

Reportez-vous à la documentation suivante pour vous connecter à votre machine virtuelle [Windows](../virtual-machines/windows/connect-logon.md) ou [Linux](../virtual-machines/linux/ssh-from-windows.md#connect-to-your-vm) .

### <a name="install-the-most-current-updates"></a>Installer les dernières mises à jour

[!INCLUDE [Discussion of most current updates](includes/most-current-updates.md)]

### <a name="perform-additional-security-checks"></a>Effectuer des vérifications de sécurité supplémentaires

[!INCLUDE [Discussion of addition security checks](includes/additional-security-checks.md)]

### <a name="perform-custom-configuration-and-scheduled-tasks"></a>Exécuter des tâches planifiées de configuration personnalisée

[!INCLUDE [Discussion of custom configuration and scheduled tasks](includes/custom-config.md)]

[!INCLUDE [Discussion of addition security checks](includes/size-connect-generalize.md)]

## <a name="next-steps"></a>Étapes suivantes

- Étape suivante recommandée : [Testez votre image de machine virtuelle](azure-vm-image-test.md) pour vous assurer qu’elle répond aux exigences de publication de la Place de marché Azure. Cette étape est facultative.
- Si vous ne testez pas votre image de machine virtuelle, passez à [générer l’URI SAS](azure-vm-get-sas-uri.md).
- Si vous rencontrez des difficultés lors de la création de votre nouveau disque dur virtuel basé sur Azure, consultez [Forum aux questions sur la Place de marché Microsoft Azure](azure-vm-create-faq.md).