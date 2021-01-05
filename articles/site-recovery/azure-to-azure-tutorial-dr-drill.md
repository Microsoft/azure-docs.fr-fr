---
title: 'Tutoriel : Tester une reprise d’activité après sinistre pour des machines virtuelles Azure à l’aide d’Azure Site Recovery'
description: Dans ce tutoriel, vous allez voir comment tester une reprise d’activité après sinistre pour des machines virtuelles Azure en les répliquant dans une autre région à l’aide de Site Recovery.
services: site-recovery
ms.topic: tutorial
ms.date: 11/05/2020
ms.custom: mvc
ms.openlocfilehash: c7cd1898f27f3b7255009efb40f6bcc8938dbf9e
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93395598"
---
# <a name="tutorial-run-a-disaster-recovery-drill-for-azure-vms"></a>Tutoriel : Tester une reprise d’activité après sinistre pour des machines virtuelles Azure

Découvrez comment tester une reprise d’activité après sinistre pour des machines virtuelles Azure en les répliquant dans une autre région Azure à l’aide d’[Azure Site Recovery](site-recovery-overview.md). Dans cet article, vous découvrirez comment :

> [!div class="checklist"]
> * Vérifier les conditions préalables
> * Vérifier les paramètres de machine virtuelle avant d’effectuer le test
> * Exécuter un test de basculement
> * Effectuer le nettoyage après le test


> [!NOTE]
> Ce tutoriel présente les étapes principales qui sont nécessaires au test de reprise d’activité après sinistre. Si vous souhaitez effectuer un test de reprise d’activité en testant l’intégralité de l’infrastructure, lisez d’abord les articles concernant les [réseaux](azure-to-azure-about-networking.md), l’[automatisation](azure-to-azure-powershell.md) et le [dépannage](azure-to-azure-troubleshoot-errors.md) des machines virtuelles Azure.

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel, vous devez activer la reprise d’activité après sinistre pour une ou plusieurs machines virtuelles Azure. Pour ce faire, [suivez le premier tutoriel](azure-to-azure-tutorial-enable-replication.md) de cette série.

## <a name="verify-vm-settings"></a>Vérifier les paramètres des machines virtuelles

1. Dans le coffre > **Éléments répliqués**, sélectionnez la machine virtuelle.

    ![Option permettant d’ouvrir la page de reprise d’activité après sinistre dans les propriétés de machine virtuelle](./media/azure-to-azure-tutorial-dr-drill/vm-settings.png)

2. Dans la page **Vue d’ensemble**, vérifiez que la machine virtuelle est protégée et saine.
3. Lorsque vous exécutez un test de basculement, vous sélectionnez un réseau virtuel Azure situé dans la région cible. La machine virtuelle Azure créée après le basculement est placée sur ce réseau. 

    - Dans ce tutoriel, nous allons sélectionner un réseau existant pour le test de basculement.
    - Pour ce test, nous vous recommandons de choisir un réseau autre qu’un réseau de production, afin que les adresses IP et les composants réseau restent disponibles pour les réseaux de production.
   - Vous pouvez également préconfigurer les paramètres réseau à utiliser pour le test de basculement. Les paramètres que vous pouvez attribuer précisément à chaque carte réseau correspondent au sous-réseau, à l’adresse IP privée, à l’adresse IP publique, à l’équilibreur de charge et au groupe de sécurité réseau. Nous n’allons pas utiliser cette méthode, mais vous pouvez [lire cet article](azure-to-azure-customize-networking.md#customize-failover-and-test-failover-networking-configurations) pour en savoir plus.


## <a name="run-a-test-failover"></a>Exécuter un test de basculement


1. Dans la page **Vue d’ensemble**, sélectionnez **Tester le basculement**.

    
    ![Bouton Tester le basculement pour l’élément répliqué](./media/azure-to-azure-tutorial-dr-drill/test-failover-button.png)

2. Dans **Tester le basculement**, choisissez un point de récupération. La machine virtuelle Azure située dans la région cible est créée à l’aide des données provenant de ce point de récupération.
  
   - **Dernier point traité** : Utilise le dernier point de récupération traité par Site Recovery. L’horodatage est affiché. Aucun temps n’est passé à traiter les données, l’objectif de délai de récupération (RTO) fourni est donc faible.
   -  **Les dernières** : Traite d’abord toutes les données qui ont été envoyées à Site Recovery, afin de créer un point de récupération pour chaque machine virtuelle avant de basculer vers celle-ci. Fournit l’objectif de point de récupération (RPO) le plus faible, car toutes les données sont répliquées vers Site Recovery au moment où le basculement est déclenché.
   - **Dernier point de cohérence des applications** : Cette option bascule les machines virtuelles vers le dernier point de récupération avec cohérence des applications. L’horodatage est affiché.
   - **Personnalisé** : Effectue un basculement vers un point de récupération particulier. La personnalisation est disponible uniquement quand vous effectuez le basculement d’une seule machine virtuelle, et non un plan de récupération.

3. Dans **Réseau virtuel Azure**, sélectionnez le réseau cible sur lequel placer les machines virtuelles Azure créées après le basculement. Si possible, sélectionnez un réseau autre qu’un réseau de production, et n’utilisez pas le réseau qui a été créé lorsque vous avez activé la réplication.

    ![Page des paramètres du test de basculement](./media/azure-to-azure-tutorial-dr-drill/test-failover-settings.png)    

4. Pour démarrer le basculement, sélectionnez **OK**.
5. Supervisez le test de basculement à l’aide des notifications.

    ![Notification de progression](./media/azure-to-azure-tutorial-dr-drill/notification-start-test-failover.png) ![Notification de réussite](./media/azure-to-azure-tutorial-dr-drill/notification-finish-test-failover.png)     


5. Une fois le basculement terminé, la machine virtuelle Azure créée dans la région cible apparaît dans le portail Azure sous **Machines virtuelles**. Vérifiez que la machine virtuelle est en cours d’exécution, qu’elle est correctement dimensionnée et qu’elle est connectée au réseau que vous avez sélectionné.

## <a name="clean-up-resources"></a>Nettoyer les ressources

1. Dans la page **Essentials**, sélectionnez **Nettoyer le test de basculement**.

    ![Bouton permettant de démarrer le processus de nettoyage](./media/azure-to-azure-tutorial-dr-drill/select-cleanup.png)

2. Dans **Nettoyage du test de basculement** > **Notes**, enregistrez toutes les observations faites à propos du test de basculement. 
3. Sélectionnez **Le test est terminé** pour supprimer les machines virtuelles qui ont été créées pendant le test de basculement.

    ![Page comprenant les options de nettoyage](./media/azure-to-azure-tutorial-dr-drill/cleanup-failover.png)

4. Supervisez la progression du nettoyage dans les notifications.

    ![Notification de la progression du nettoyage](./media/azure-to-azure-tutorial-dr-drill/notification-start-cleanup.png) ![Notification de réussite du nettoyage](./media/azure-to-azure-tutorial-dr-drill/notification-finish-cleanup.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez effectué un test de reprise d’activité après sinistre afin de vérifier que le basculement fonctionne comme prévu. Vous pouvez maintenant tenter un basculement complet.

> [!div class="nextstepaction"]
> [Exécuter un basculement de production](azure-to-azure-tutorial-failover-failback.md)
