---
title: 'Tutoriel : Restaurer automatiquement des machines virtuelles Azure vers une région primaire lors d’une reprise d’activité avec Azure Site Recovery.'
description: Tutoriel expliquant comment restaurer automatiquement des machines virtuelles Azure vers une région primaire à l’aide d’Azure Site Recovery.
ms.topic: tutorial
ms.date: 11/05/2020
ms.custom: mvc
ms.openlocfilehash: 5c127010a7988bf08c77340a4fc10bb32dc76f87
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93393887"
---
# <a name="tutorial-fail-back-azure-vm-to-the-primary-region"></a>Tutoriel : Restaurer automatiquement une machine virtuelle Azure vers la région primaire

Après avoir restauré une machine virtuelle Azure vers une région Azure secondaire, suivez ce tutoriel pour restaurer la machine virtuelle vers la région Azure primaire à l’aide d’[Azure Site Recovery](site-recovery-overview.md).  Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> 
> * Passez en revue les prérequis.
> * Restaurez automatiquement la machine virtuelle Azure dans la région secondaire.
> * Reprotégez les machines virtuelles primaires vers la région secondaire.
> 
> [!NOTE]
> Ce tutoriel explique comment effectuer une restauration automatique avec le moins d’étapes possible. Si vous souhaitez effectuer un basculement avec tous les paramètres, lisez d’abord les articles concernant les [réseaux](azure-to-azure-about-networking.md), l’[automatisation](azure-to-azure-powershell.md) et le [dépannage](azure-to-azure-troubleshoot-errors.md) des machines virtuelles Azure.



## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel, vous devez :

1. [Configurer la réplication](azure-to-azure-tutorial-enable-replication.md) pour au moins une machine virtuelle Azure, puis effectuer un [test de reprise d’activité après sinistre](azure-to-azure-tutorial-dr-drill.md) pour cette machine.
2. [Basculer la machine virtuelle](azure-to-azure-tutorial-failover-failback.md) de la région primaire vers une région secondaire, puis la reprotéger afin qu’elle soit répliquée de la région secondaire vers la région primaire. 
3. Vérifiez que la région primaire est disponible ; vous pouvez alors y créer des ressources et accéder à celles-ci.

## <a name="fail-back-to-the-primary-region"></a>Effectuer une restauration automatique vers la région primaire

Une fois les machines virtuelles reprotégées, vous pouvez effectuer une restauration automatique vers la région primaire en fonction de vos besoins.

1. Dans le coffre > **Éléments répliqués**, sélectionnez la machine virtuelle.

2. Avant d’effectuer un basculement, vérifiez que la machine virtuelle est saine et que la synchronisation est terminée dans la vue d’ensemble de la machine virtuelle. La machine virtuelle doit être à l’état *Protégé*.

    ![Page de vue d’ensemble de la machine virtuelle à l’état Protégé](./media/azure-to-azure-tutorial-failback/protected-state.png)

3. Dans la page de vue d’ensemble, sélectionnez **Basculement**. Étant donné que nous n’allons pas effectuer un test de basculement cette fois-ci, nous sommes invités à le vérifier.

    [Page indiquant que nous acceptons d’effectuer le basculement sans un test préalable](./media/azure-to-azure-tutorial-failback/no-test.png)

4. Dans **Basculement**, notez le sens de basculement qui va de la région secondaire à la région primaire, puis sélectionnez un point de récupération. La machine virtuelle Azure de la cible (région primaire) est créée à l’aide des données situées sur ce point.
   - **Dernier point traité** : Utilise le dernier point de récupération traité par Site Recovery. L’horodatage est affiché. Aucun temps n’est passé à traiter les données, l’objectif de délai de récupération (RTO) fourni est donc faible.
   -  **Les dernières** : Traite d’abord toutes les données qui ont été envoyées à Site Recovery, afin de créer un point de récupération pour chaque machine virtuelle avant de basculer vers celle-ci. Fournit l’objectif de point de récupération (RPO) le plus faible, car toutes les données sont répliquées vers Site Recovery au moment où le basculement est déclenché.
   - **Dernier point de cohérence des applications** : Cette option bascule les machines virtuelles vers le dernier point de récupération avec cohérence des applications. L’horodatage est affiché.
   - **Personnalisé** : Effectue un basculement vers un point de récupération particulier. La personnalisation est disponible uniquement quand vous effectuez le basculement d’une seule machine virtuelle, et non un plan de récupération.

    > [!NOTE]
    > Si vous basculez une machine virtuelle à laquelle vous avez ajouté un disque après avoir activé la réplication, les points de réplication afficheront les disques disponibles pour la récupération. Par exemple, un point de réplication qui a été créé avant l’ajout d’un deuxième disque s’affiche ainsi : « 1 sur 2 disques ».

4. Sélectionnez **Arrêter la machine avant de commencer le basculement** si vous souhaitez que Site Recovery tente d’arrêter les machines virtuelles sources avant de démarrer le basculement. L’arrêt permet d’éviter toute perte de données. Le basculement est effectué même en cas d’échec de l’arrêt. 

    ![Page des paramètres de basculement](./media/azure-to-azure-tutorial-failback/failover.png)    

3. Pour démarrer le basculement, sélectionnez **OK**.
4. Supervisez le basculement à l’aide des notifications.

    ![Notification concernant la progression du basculement](./media/azure-to-azure-tutorial-failback/notification-progress.png)  
    ![Notification concernant la réussite du basculement](./media/azure-to-azure-tutorial-failback/notification-success.png)   

## <a name="reprotect-vms"></a>Reprotéger des machines virtuelles

Après avoir restauré automatiquement les machines virtuelles vers la région primaire, vous devez les reprotéger afin qu’elles recommencent leur réplication dans la région secondaire.

1. Dans la page **Vue d’ensemble** de la machine virtuelle, sélectionnez **Reprotéger**.

    ![Bouton de reprotection à partir de la région primaire](./media/azure-to-azure-tutorial-failback/reprotect.png)  

2. Passez en revue les paramètres cibles de la région primaire. Les ressources marquées comme nouvelles sont créées par Site Recovery dans le cadre de l’opération de reprotection.
3. Sélectionnez **OK** pour démarrer le processus de reprotection. Le processus envoie des données initiales à l’emplacement cible, puis réplique les informations delta des machines virtuelles vers la cible.

     ![Page montrant les paramètres de réplication](./media/azure-to-azure-tutorial-failback/replication-settings.png) 

4. Supervisez la progression de la reprotection dans les notifications. 

    ![Notification de la progression de la reprotection](./media/azure-to-azure-tutorial-failback/notification-reprotect-start.png) [Notification de la progression de la reprotection](./media/azure-to-azure-tutorial-failback/notification-reprotect-finish.png)
    
  

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour les machines virtuelles comportant des disques managés, une fois la restauration automatique terminée et les machines virtuelles reconfigurées pour la réplication entre la région primaire et la région secondaire, Site Recovery nettoie automatiquement les machines dans la région secondaire de reprise d’activité après sinistre. Il n’est pas nécessaire de supprimer manuellement les machines virtuelles et les cartes d’interface réseau dans la région secondaire. Les machines virtuelles comportant des disques non managés ne sont pas nettoyées.

Si vous désactivez complètement la réplication après une restauration automatique, Site Recovery nettoie les machines protégées. Dans ce cas, il nettoie également les disques des machines virtuelles qui n’utilisent pas de disques managés. 
 
## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez restauré automatiquement des machines virtuelles entre la région secondaire et la région primaire. Il s’agit de la dernière étape du processus qui comprend l’activation de la réplication pour une machine virtuelle, le test d’une reprise d’activité après sinistre, le basculement de la région primaire vers la région secondaire, et enfin, la restauration automatique.

> [!div class="nextstepaction"]
> À présent, essayez d’effectuer une reprise d’activité après sinistre vers Azure pour une [machine virtuelle locale](vmware-azure-tutorial-prepare-on-premises.md).

