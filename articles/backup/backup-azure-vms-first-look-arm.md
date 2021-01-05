---
title: Sauvegarder une machine virtuelle Azure à partir des paramètres de la machine virtuelle
description: Dans cet article, découvrez comment sauvegarder une ou plusieurs machines virtuelles Azure avec le service Sauvegarde Azure.
ms.topic: conceptual
ms.date: 06/13/2019
ms.openlocfilehash: 55b71d2a2901cdde984df3ebfd68a2a643b78b74
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89667508"
---
# <a name="back-up-an-azure-vm-from-the-vm-settings"></a>Sauvegarder une machine virtuelle Azure à partir des paramètres de la machine virtuelle

Cet article explique comment sauvegarder des machines virtuelles Azure avec le service [Sauvegarde Azure](backup-overview.md). Pour sauvegarder des machines virtuelles Azure, vous avez le choix entre deux méthodes :

- Une seule machine virtuelle Azure : les instructions de cet article décrivent comment sauvegarder une machine virtuelle Azure directement à partir des paramètres de la machine virtuelle.
- Plusieurs machines virtuelles Azure : vous pouvez configurer un coffre Recovery Services et configurer la sauvegarde pour plusieurs machines virtuelles Azure. Pour ce scénario, suivez les instructions contenues dans [cet article](backup-azure-arm-vms-prepare.md).

## <a name="before-you-start"></a>Avant de commencer

1. [Découvrez](backup-architecture.md#how-does-azure-backup-work) le fonctionnement de la sauvegarde et [vérifiez](backup-support-matrix.md#azure-vm-backup-support) les conditions de prise en charge.
2. [Consultez une vue d’ensemble](backup-azure-vms-introduction.md) de la sauvegarde de machines virtuelles Azure.

### <a name="azure-vm-agent-installation"></a>Installation de l’agent de machine virtuelle Azure

Pour sauvegarder des machines virtuelles Azure, le service Sauvegarde Azure installe une extension sur l’agent de machine virtuelle en cours d’exécution sur la machine. Si votre machine virtuelle a été créée à partir d’une image de la Place de marché Azure, l’agent s’exécute. Dans certains cas, par exemple, si vous créez une machine virtuelle personnalisée, ou migrez une machine à partir d’un emplacement local, vous devrez peut-être installer l’agent manuellement.

- Si vous n’avez pas besoin d’installer l’agent de machine virtuelle manuellement, suivez les instructions fournies pour les machines virtuelles [Windows](../virtual-machines/extensions/agent-windows.md) ou [Linux](../virtual-machines/extensions/agent-linux.md).
- Une fois que l’agent est installé et que vous activez la sauvegarde, Sauvegarde Azure installe l’extension de sauvegarde sur l’agent. L’extension est mise à jour et corrigée sans aucune intervention de l’utilisateur.

## <a name="back-up-from-azure-vm-settings"></a>Sauvegarder à partir des paramètres de la machine virtuelle Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Sélectionnez **Tous les services**, puis, dans le filtre, tapez **Machines virtuelles**. Cliquez ensuite sur **Machines virtuelles**.
3. Dans la liste des machines virtuelles, sélectionnez celle à sauvegarder.
4. Dans le menu de la machine virtuelle, sélectionnez **Sauvegarde**.
5. Dans **Coffre Recovery Services**, effectuez les étapes suivantes :
   - Si vous disposez déjà d’un coffre, cliquez sur **Sélectionner**, puis choisissez un coffre.
   - Si vous n’avez pas de coffre, cliquez sur **Créer**. Nommez le coffre. Il est créé dans la même région et le même groupe de ressources que la machine virtuelle. Vous ne pouvez pas modifier ces paramètres quand vous activez la sauvegarde directement à partir des paramètres de la machine virtuelle.

        ![Assistant Activer la sauvegarde](./media/backup-azure-vms-first-look-arm/vm-menu-enable-backup-small.png)

6. Dans **Choisir une stratégie de sauvegarde**, procédez de l’une des façons suivantes :

   - Conservez la stratégie par défaut. La machine virtuelle est sauvegardée une fois par jour à l’heure spécifiée, et les sauvegardes sont conservées dans le coffre pendant 30 jours.
   - Sélectionnez une stratégie de sauvegarde existante si vous en avez une.
   - Créez une stratégie et définissez ses paramètres.  

       ![Sélectionner la stratégie de sauvegarde](./media/backup-azure-vms-first-look-arm/set-backup-policy.png)

7. Sélectionnez **Activer la Sauvegarde Azure**. La stratégie de sauvegarde est alors associée à la machine virtuelle.

    ![Bouton Activer la sauvegarde](./media/backup-azure-vms-first-look-arm/vm-management-menu-enable-backup-button.png)

8. Vous pouvez suivre la progression de la configuration dans les notifications du portail.
9. Une fois le travail terminé, dans le menu de la machine virtuelle, sélectionnez **Sauvegarde**. La page affiche l’état de la sauvegarde de la machine virtuelle, des informations sur les points de récupération, les travaux en cours d’exécution et les alertes émises.

   ![État de la sauvegarde](./media/backup-azure-vms-first-look-arm/backup-item-view-update.png)

10. Une fois la sauvegarde activée, une sauvegarde initiale s’exécute. Vous pouvez démarrer la sauvegarde initiale immédiatement ou attendre qu’elle démarre selon la planification de la sauvegarde.
    - Tant que la sauvegarde initiale n’est pas terminée, la zone **État de la dernière sauvegarde** indique **Avertissement (Sauvegarde initiale en attente)** .
    - Pour savoir quand la prochaine sauvegarde planifiée va s’exécuter, sélectionnez le nom de la stratégie de sauvegarde.

## <a name="run-a-backup-immediately"></a>Exécuter une sauvegarde immédiatement

1. Pour exécuter une sauvegarde immédiatement, dans le menu de la machine virtuelle, sélectionnez **Sauvegarde** > **Sauvegarder maintenant**.

    ![Exécuter la sauvegarde](./media/backup-azure-vms-first-look-arm/backup-now-update.png)

2. Dans **Sauvegarder maintenant**, utilisez le contrôle de calendrier pour sélectionner la durée de conservation du point de récupération, puis cliquez sur **OK**.

    ![Jour de fin de la conservation de la sauvegarde](./media/backup-azure-vms-first-look-arm/backup-now-blade-calendar.png)

3. Les notifications du portail vous informent que le travail de sauvegarde a été déclenché. Pour surveiller la progression de la sauvegarde, sélectionnez **Afficher tous les travaux**.

## <a name="back-up-from-the-recovery-services-vault"></a>Sauvegarder à partir du coffre Recovery Services

Suivez les instructions de [cet article](backup-azure-arm-vms-prepare.md) pour activer la sauvegarde des machines virtuelles Azure en configurant un coffre Recovery Services Sauvegarde Azure et en activant la sauvegarde dans le coffre.

## <a name="next-steps"></a>Étapes suivantes

- Si vous rencontrez des difficultés avec les procédures décrites dans cet article, consultez le [guide de résolution des problèmes](backup-azure-vms-troubleshoot.md).
- [Découvrez-en plus](backup-azure-manage-vms.md) sur la gestion de vos sauvegardes.
