---
title: Contrôle de maintenance des machines virtuelles Azure avec le portail Azure
description: Découvrez comment contrôler le moment où la maintenance est appliquée à vos machines virtuelles Azure à l’aide du contrôle de maintenance et du portail Azure.
author: cynthn
ms.service: virtual-machines
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 04/22/2020
ms.author: cynthn
ms.openlocfilehash: b174e2631131e6bf26d7b1cb62442c8a99102e2e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91397281"
---
# <a name="control-updates-with-maintenance-control-and-the-azure-portal"></a>Contrôler les mises à jour avec le contrôle de maintenance et le portail Azure

Le contrôle de maintenance vous permet de décider du moment où appliquer les mises à jour sur vos machines virtuelles isolées et vos hôtes dédiés Azure. Cette rubrique traite des options du portail Azure pour le contrôle de la maintenance. Pour en savoir plus sur les avantages de l’utilisation du contrôle de la maintenance, ses limitations et d’autres options de gestion, consultez [Gestion des mises à jour de plateforme avec le contrôle de maintenance](maintenance-control.md).

## <a name="create-a-maintenance-configuration"></a>Créer une configuration de maintenance

1. Connectez-vous au portail Azure.

1. Rechercher des **configurations de maintenance**.

   ![Capture d’écran montrant comment ouvrir les configurations de maintenance](media/virtual-machines-maintenance-control-portal/maintenance-configurations-search.png)

1. Cliquez sur **Add**.

   ![Capture d’écran montrant comment ajouter une configuration de maintenance](media/virtual-machines-maintenance-control-portal/maintenance-configurations-add.png)

1. Choisissez un abonnement et un groupe de ressources, attribuez un nom à la configuration, puis choisissez une région. Cliquez sur **Suivant**.

   ![Capture d’écran montrant les bases de la configuration de maintenance](media/virtual-machines-maintenance-control-portal/maintenance-configurations-basics.png)

1. Ajoutez des balises et des valeurs. Cliquez sur **Suivant**.

   ![Capture d’écran montrant comment ajouter des balises à la configuration de maintenance](media/virtual-machines-maintenance-control-portal/maintenance-configurations-tags.png)

1. Passez en revue le résumé. Cliquez sur **Créer**.

   ![Capture d’écran montrant comment créer une configuration de maintenance](media/virtual-machines-maintenance-control-portal/maintenance-configurations-create.png)

1. Une fois le déploiement terminé, cliquez sur **Accéder à la ressource**.

   ![Capture d’écran montrant le déploiement de la configuration de maintenance terminée](media/virtual-machines-maintenance-control-portal/maintenance-configurations-deployment-complete.png)

## <a name="assign-the-configuration"></a>Affecter la configuration

Sur la page des détails de la configuration de maintenance, cliquez sur Affectations, puis sur **Affecter la ressource**. 

![Capture d’écran montrant comment affecter une ressource](media/virtual-machines-maintenance-control-portal/maintenance-configurations-add-assignment.png)

Sélectionnez les ressources auxquelles vous souhaitez affecter la configuration de maintenance, puis cliquez sur **Ok**. La colonne **Type** indique si la ressource est une machine virtuelle isolée ou un hôte dédié Azure. La machine virtuelle doit être en cours d’exécution pour que l’on puisse affecter la configuration. Une erreur se produit si vous tentez d’affecter une configuration à une machine virtuelle arrêtée. 

<!---Shantanu to add details about the error case--->

![Capture d’écran montrant comment sélectionner une ressource](media/virtual-machines-maintenance-control-portal/maintenance-configurations-select-resource.png)

## <a name="check-configuration"></a>Vérifier la configuration

Vous pouvez vérifier que la configuration a été appliquée correctement ou si une configuration de maintenance est actuellement affectée à l’aide de la **configuration de maintenance**. La colonne **Type** indique si la configuration est affectée à une machine virtuelle isolée ou à un hôte dédié Azure. 

![Capture d’écran montrant comment vérifier une configuration de maintenance](media/virtual-machines-maintenance-control-portal/maintenance-configurations-host-type.png)

Vous pouvez également vérifier la configuration d’une machine virtuelle spécifique sur sa page de propriétés. Cliquez sur **Maintenance** pour voir la configuration affectée à cette machine virtuelle.

![Capture d’écran montrant comment vérifier la maintenance d’un hôte](media/virtual-machines-maintenance-control-portal/maintenance-configurations-check-config.png)

## <a name="check-for-pending-updates"></a>Rechercher les mises à jour en attente

Il y a deux façons de vérifier si des mises à jour sont en attente pour une configuration de maintenance. Dans **Configurations de maintenance**, sur les détails de la configuration, cliquez sur **Affectations** et vérifiez l’**état de la maintenance**.

![Capture d’écran montrant comment vérifier les mises à jour en attente](media/virtual-machines-maintenance-control-portal/maintenance-configurations-pending.png)

Vous pouvez également vérifier un hôte spécifique à l’aide de **Machines virtuelles** ou des propriétés de l’hôte dédié. 

![Capture d'écran montrant l'état de maintenance mis en surbrillance.](media/virtual-machines-maintenance-control-portal/maintenance-configurations-pending-vm.png)

## <a name="apply-updates"></a>Appliquer des mises à jour

Vous pouvez appliquer des mises à jour en attente à la demande à l’aide de **machines virtuelles**. Dans les détails de la machine virtuelle, cliquez sur **Maintenance**, puis sur **Appliquer la maintenance maintenant**.

![Capture d’écran montrant comment appliquer des mises à jour en attente](media/virtual-machines-maintenance-control-portal/maintenance-configurations-apply-updates-now.png)

## <a name="check-the-status-of-applying-updates"></a>Vérifier l’état de l’application des mises à jour 

Vous pouvez vérifier la progression des mises à jour d’une configuration dans **Configurations de maintenance** ou à l’aide de **Machines virtuelles**. Dans les détails de la machine virtuelle, cliquez sur **Maintenance**. Dans l’exemple suivant, l’**État de maintenance** indique qu’une mise à jour est **En attente**.

![Capture d’écran montrant comment vérifier l’état des mises à jour en attente](media/virtual-machines-maintenance-control-portal/maintenance-configurations-status.png)

## <a name="delete-a-maintenance-configuration"></a>Supprimer une configuration de maintenance

Pour supprimer une configuration, ouvrez les détails de la configuration, puis cliquez sur **Supprimer**.

![Capture d'écran montrant comment supprimer une configuration.](media/virtual-machines-maintenance-control-portal/maintenance-configurations-delete.png)


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Maintenance et mises à jour](maintenance-and-updates.md).
