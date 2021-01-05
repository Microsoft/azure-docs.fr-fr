---
title: 'Tutoriel : Basculer des machines virtuelles Azure vers une région secondaire lors d’une reprise d’activité avec Azure Site Recovery'
description: Tutoriel expliquant comment basculer et reprotéger des machines virtuelles Azure répliquées sur une région Azure secondaire pour une reprise d’activité après sinistre avec le service Azure Site Recovery.
ms.topic: tutorial
ms.date: 11/05/2020
ms.custom: mvc
ms.openlocfilehash: 99263c83d25542073d63c1cba394a147bd5b2170
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93392735"
---
# <a name="tutorial-fail-over-azure-vms-to-a-secondary-region"></a>Tutoriel : Basculer des machines virtuelles Azure vers une région secondaire

Découvrez comment basculer les machines virtuelles Azure qui sont activées pour la reprise d’activité après sinistre vers une région Azure secondaire à l’aide d’[Azure Site Recovery](site-recovery-overview.md). Après le basculement, reprotégez les machines virtuelles dans la région cible afin qu’elles soient répliquées de nouveau dans la région primaire. Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Vérifier les conditions préalables
> * Vérifier les paramètres des machines virtuelles
> * Exécuter un basculement vers la région secondaire
> * Démarrez la réplication de la machine virtuelle sur la région primaire.


> [!NOTE]
> Ce tutoriel explique comment effectuer le basculement de machines virtuelles avec le moins d’étapes possible. Si vous souhaitez effectuer un basculement avec tous les paramètres, lisez d’abord les articles concernant les [réseaux](azure-to-azure-about-networking.md), l’[automatisation](azure-to-azure-powershell.md) et le [dépannage](azure-to-azure-troubleshoot-errors.md) des machines virtuelles Azure.



## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel, vous devez :

1. Configurer la réplication pour une ou plusieurs machines virtuelles Azure. Si vous ne l’avez pas encore fait, [suivez le premier tutoriel](azure-to-azure-tutorial-enable-replication.md) de cette série.
2. Nous vous recommandons de [tester une reprise d’activité après sinistre](azure-to-azure-tutorial-dr-drill.md) pour les machines virtuelles répliquées. L’exécution de ce test avant un basculement complet vous permet de vérifier que tout fonctionne comme prévu, sans impacter votre environnement de production. 


## <a name="verify-the-vm-settings"></a>Vérifier les paramètres de machine virtuelle

1. Dans le coffre > **Éléments répliqués**, sélectionnez la machine virtuelle.

    ![Option permettant d’ouvrir les propriétés de machine virtuelle dans la page Vue d’ensemble](./media/azure-to-azure-tutorial-failover-failback/vm-settings.png)

2. Dans la page **Vue d’ensemble**, vérifiez que la machine virtuelle est protégée et saine avant d’effectuer le basculement.
    ![Page où vérifier les propriétés et l’état de la machine virtuelle](./media/azure-to-azure-tutorial-failover-failback/vm-state.png)

3. Avant de procéder au basculement, vérifiez les éléments suivants :
    - La machine virtuelle exécute un système d’exploitation [Windows](azure-to-azure-support-matrix.md#windows) ou [Linux](azure-to-azure-support-matrix.md#replicated-machines---linux-file-systemguest-storage) pris en charge.
    - La machine virtuelle est conforme aux exigences relatives au [calcul](azure-to-azure-support-matrix.md#replicated-machines---compute-settings), au [stockage](azure-to-azure-support-matrix.md#replicated-machines---storage) et au [réseau](azure-to-azure-support-matrix.md#replicated-machines---networking).

## <a name="run-a-failover"></a>Exécuter un basculement


1. Dans la page **Vue d’ensemble** de la machine virtuelle, sélectionnez **Basculement**.

    ![Bouton de basculement pour l’élément répliqué](./media/azure-to-azure-tutorial-failover-failback/failover-button.png)

3. Dans **Basculement**, choisissez un point de récupération. La machine virtuelle Azure située dans la région cible est créée à l’aide des données provenant de ce point de récupération.
  
   - **Dernier point traité** : Utilise le dernier point de récupération traité par Site Recovery. L’horodatage est affiché. Aucun temps n’est passé à traiter les données, l’objectif de délai de récupération (RTO) fourni est donc faible.
   -  **Les dernières** : Traite d’abord toutes les données qui ont été envoyées à Site Recovery, afin de créer un point de récupération pour chaque machine virtuelle avant de basculer vers celle-ci. Fournit l’objectif de point de récupération (RPO) le plus faible, car toutes les données sont répliquées vers Site Recovery au moment où le basculement est déclenché.
   - **Dernier point de cohérence des applications** : Cette option bascule les machines virtuelles vers le dernier point de récupération avec cohérence des applications. L’horodatage est affiché.
   - **Personnalisé** : Effectue un basculement vers un point de récupération particulier. La personnalisation est disponible uniquement quand vous effectuez le basculement d’une seule machine virtuelle, et non un plan de récupération.

    > [!NOTE]
    > Si vous avez ajouté un disque à une machine virtuelle après avoir activé la réplication, les points de réplication indiqueront que le disque peut faire l’objet d’une récupération. Par exemple, un point de réplication qui a été créé avant l’ajout d’un deuxième disque s’affiche ainsi : « 1 sur 2 disques ».

4. Sélectionnez **Arrêter la machine avant de commencer le basculement** si vous souhaitez que Site Recovery tente d’arrêter les machines virtuelles sources avant de démarrer le basculement. L’arrêt permet d’éviter toute perte de données. Le basculement est effectué même en cas d’échec de l’arrêt. 

    ![Page des paramètres de basculement](./media/azure-to-azure-tutorial-failover-failback/failover-settings.png)    

3. Pour démarrer le basculement, sélectionnez **OK**.
4. Supervisez le basculement à l’aide des notifications.

    ![Notification de progression](./media/azure-to-azure-tutorial-failover-failback/notification-failover-start.png) ![Notification de réussite](./media/azure-to-azure-tutorial-failover-failback/notification-failover-finish.png)     

5. Une fois le basculement terminé, la machine virtuelle Azure créée dans la région cible s’affiche sous **Machines virtuelles**. Vérifiez que la machine virtuelle est en cours d’exécution et qu’elle est dimensionnée de manière appropriée. Si vous souhaitez utiliser un autre point de récupération pour la machine virtuelle, sélectionnez **Changer le point de récupération** dans la page **Essentials**.
6. Lorsque vous avez basculé la machine virtuelle souhaitée, sélectionnez **Valider** dans la page de vue d’ensemble pour finaliser le basculement.

    ![Bouton Valider](./media/azure-to-azure-tutorial-failover-failback/commit-button.png) 

7. Dans **Valider**, sélectionnez **OK** pour confirmer. La validation supprime tous les points de récupération qui sont disponibles pour la machine virtuelle dans Site Recovery. Vous ne pourrez donc plus changer le point de récupération.

8. Supervisez la progression de la validation dans les notifications.

    ![Notification de la progression de la validation](./media/azure-to-azure-tutorial-failover-failback/notification-commit-start.png) ![Notification de la réussite de la validation](./media/azure-to-azure-tutorial-failover-failback/notification-commit-finish.png)    

9. Site Recovery ne nettoie pas la machine virtuelle source après le basculement. Vous devez le faire manuellement.


## <a name="reprotect-the-vm"></a>Reprotéger la machine virtuelle

Après le basculement, reprotégez la machine virtuelle dans la région secondaire afin de la répliquer de nouveau dans la région primaire. 

1. Avant de commencer, vérifiez que l’**État** de la machine virtuelle est *Basculement validé*.
2. Vérifiez que la région primaire est disponible et que vous disposez des autorisations nécessaires pour créer des machines virtuelles dans cette région.
3. Dans la page **Vue d’ensemble** de la machine virtuelle, sélectionnez **Reprotéger**.

   ![Bouton permettant d’activer la reprotection d’une machine virtuelle.](./media/azure-to-azure-tutorial-failover-failback/reprotect-button.png)

4. Dans **Reprotéger**, vérifiez le sens de réplication (région secondaire vers région primaire), puis passez en revue les paramètres cibles de la région primaire. Les ressources marquées comme nouvelles sont créées par Site Recovery dans le cadre de l’opération de reprotection.

     ![Page des paramètres de reprotection](./media/azure-to-azure-tutorial-failover-failback/reprotect.png)

6. Sélectionnez **OK** pour démarrer le processus de reprotection. Le processus envoie des données initiales à l’emplacement cible, puis réplique les informations delta des machines virtuelles vers la cible.
7. Supervisez la progression de la reprotection dans les notifications. 

    ![Notification de la progression de la reprotection](./media/azure-to-azure-tutorial-failover-failback/notification-reprotect-start.png) ![Notification de la réussite de la reprotection](./media/azure-to-azure-tutorial-failover-failback/notification-reprotect-finish.png)
    

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez basculé de la région primaire vers la région secondaire, puis vous avez démarré la réplication des machines virtuelles vers la région primaire. Vous pouvez maintenant effectuer une restauration automatique, de la région secondaire vers la région primaire.

> [!div class="nextstepaction"]
> [Effectuer une restauration automatique vers la région primaire](azure-to-azure-tutorial-failback.md)
