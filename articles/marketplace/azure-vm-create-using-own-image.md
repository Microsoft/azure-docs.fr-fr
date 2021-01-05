---
title: Créer une offre de machine virtuelle Azure sur la Place de marché Azure à l’aide de votre propre image
description: Découvrez comment publier une offre de machine virtuelle sur la Place de marché Azure à l’aide de votre propre image.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: emuench
ms.author: krsh
ms.date: 10/20/2020
ms.openlocfilehash: 42022d1204c3b524ee2e9ef2770f616fba89dc8c
ms.sourcegitcommit: b6f3ccaadf2f7eba4254a402e954adf430a90003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92283072"
---
# <a name="how-to-create-a-virtual-machine-using-your-own-image"></a>Création d’une machine virtuelle à l’aide de votre propre image

Cet article explique comment créer et déployer une image de machine virtuelle fournie par l’utilisateur.

> [!NOTE]
> Avant de commencer cette procédure, passez en revue les [exigences techniques](marketplace-virtual-machines.md#technical-requirements) pour les offres de machine virtuelle Azure, y compris les exigences de disque dur virtuel (VHD).

Pour utiliser une image de base approuvée à la place, suivez les instructions fournies dans [Créer une image de machine virtuelle à partir d’une base approuvée](azure-vm-create-using-approved-base.md).

## <a name="configure-the-vm"></a>Configurer la machine virtuelle

Cette section décrit comment dimensionner, mettre à jour et généraliser une machine virtuelle Azure. Ces étapes sont nécessaires pour préparer le déploiement de votre machine virtuelle sur la Place de marché Microsoft Azure.

### <a name="size-the-vhds"></a>Taille des disques durs virtuels

[!INCLUDE [Discussion of VHD sizing](includes/vhd-size.md)]

### <a name="install-the-most-current-updates"></a>Installer les dernières mises à jour

[!INCLUDE [Discussion of most current updates](includes/most-current-updates.md)]

### <a name="perform-additional-security-checks"></a>Effectuer des vérifications de sécurité supplémentaires

[!INCLUDE [Discussion of addition security checks](includes/additional-security-checks.md)]

### <a name="perform-custom-configuration-and-scheduled-tasks"></a>Exécuter des tâches planifiées de configuration personnalisée

[!INCLUDE [Discussion of custom configuration and scheduled tasks](includes/custom-config.md)]

## <a name="upload-the-vhd-to-azure"></a>Charger le disque dur virtuel dans Azure

Configurez et préparez la machine virtuelle à télécharger comme décrit dans [Préparer un disque dur virtuel Windows ou un VHDX à charger sur Azure](../virtual-machines/windows/prepare-for-upload-vhd-image.md) ou [Créer et Charger un VHD Linux](../virtual-machines/linux/create-upload-generic.md).

## <a name="extract-the-vhd-from-image-if-using-image-building-services"></a>Extraire le disque dur virtuel de l’image (si vous utilisez les services de création d’images)

Si vous utilisez un service de génération d’images, tel que [Packer](https://www.packer.io/), vous devrez peut-être extraire le disque dur virtuel de l’image. Il n’existe pas de méthode directe pour effectuer cette opération. Vous devez créer une machine virtuelle et extraire le disque dur virtuel à partir du disque de machine virtuelle.

### <a name="create-the-vm-on-the-azure-portal"></a>Créez la machine virtuelle sur le Portail Azure

Effectuez les étapes suivantes pour créer l’image de machine virtuelle de base sur le [Portail Azure](https://ms.portal.azure.com/).

1. Connectez-vous au [portail Azure](https://ms.portal.azure.com/).
2. Sélectionnez **Machines virtuelles** .
3. Sélectionnez **+ ajouter des** pour ouvrir l’écran **Créer une machine virtuelle** .
4. Sélectionnez l’image dans la liste déroulante ou sélectionnez **Parcourir toutes les images publiques et privées** pour rechercher ou parcourir toutes les images de machines virtuelles disponibles.
5. Pour créer une machine virtuelle **Gen 2** , accédez à l’onglet **avancé** et sélectionnez l’option **Gen 2** .

    :::image type="content" source="media/create-vm/vm-gen-option.png" alt-text="Sélectionnez GEN 1 ou Gen 2.":::

6. Sélectionnez la taille de la machine virtuelle à déployer.

    :::image type="content" source="media/create-vm/create-virtual-machine-sizes.png" alt-text="Sélectionnez GEN 1 ou Gen 2.":::

7. Fournissez les autres informations requises pour créer la machine virtuelle.
8. Sélectionnez **Vérifier + créer** pour passer en revue vos choix. Lorsque le message **Validation réussie** s’affiche, sélectionnez **Créer** .

Azure commence le provisionnement de la machine virtuelle que vous avez spécifiée. Suivez sa progression en sélectionnant l’onglet **Machines virtuelles** dans le menu de gauche. Une fois créé, l’état des modifications de la machine virtuelle est modifié pour **en cours d’exécution** .

### <a name="connect-to-your-vm"></a>Connexion à votre machine virtuelle

Reportez-vous à la documentation suivante pour vous connecter à votre machine virtuelle [Windows](../virtual-machines/windows/connect-logon.md) ou [Linux](../virtual-machines/linux/ssh-from-windows.md#connect-to-your-vm) .

[!INCLUDE [Discussion of addition security checks](includes/size-connect-generalize.md)]

## <a name="next-steps"></a>Étapes suivantes

- Étape suivante recommandée : [Testez votre image de machine virtuelle](azure-vm-image-test.md) pour vous assurer qu’elle répond aux exigences de publication de la Place de marché Azure. Cette étape est facultative.
- Si vous ne Testez pas votre image de machine virtuelle, passez à [générer l’URI SAS](azure-vm-get-sas-uri.md).
- Si vous rencontrez des difficultés lors de la création de votre nouveau disque dur virtuel basé sur Azure, consultez [Forum aux questions sur la Place de marché Microsoft Azure](azure-vm-create-faq.md).
