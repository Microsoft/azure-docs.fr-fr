---
title: Reprotéger des machines virtuelles Azure dans un cloud privé Azure VMware Solution avec Azure Site Recovery
description: Découvrez comment reprotéger des machines virtuelles Azure VMware Solution après un basculement vers Azure avec Azure Site Recovery.
author: Harsha-CS
manager: rochakm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 09/30/2020
ms.author: harshacs
ms.custom: MVC
ms.openlocfilehash: 80ff2f3f3d5fdcf61770889dcdaaf075941b90ff
ms.sourcegitcommit: 5abc3919a6b99547f8077ce86a168524b2aca350
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91814202"
---
# <a name="reprotect-from-azure-to-azure-vmware-solution-private-cloud"></a>Reprotéger d’Azure vers un cloud privé Azure VMware Solution

Après un [basculement](avs-tutorial-failover.md) de machines virtuelles Azure VMware Solution vers Azure, la première étape de la restauration automatique sur votre cloud privé Azure VMware Solution consiste à reprotéger les machines virtuelles Azure qui ont été créées pendant le basculement. Cet article décrit ce processus de reprotection. 

## <a name="before-you-begin"></a>Avant de commencer

1. Suivez les étapes décrites dans [cet article](vmware-azure-prepare-failback.md) pour préparer la reprotection et la restauration automatique, notamment la configuration d’un serveur de processus dans Azure et d’un serveur cible maître de cloud privé Azure VMware Solution, ainsi que la configuration d’une connexion VPN de site à site ou d’un peering privé ExpressRoute pour la restauration automatique.
2. Vérifiez que le serveur de configuration du cloud privé Azure VMware Solution est en cours d’exécution et qu’il est connecté à Azure. Lors de la restauration automatique, la machine virtuelle doit exister dans la base de données du serveur de configuration. Sinon, la restauration automatique échoue.
3. Supprimez les captures instantanées sur le serveur cible maître du cloud privé Azure VMware Solution. La reprotection ne fonctionnera pas en présence de captures instantanées.  Les captures instantanées sur la machine virtuelle sont automatiquement fusionnées lors d’une tâche de reprotection.
4. Si vous reprotégez des machines virtuelles rassemblées dans un groupe de réplication pour assurer la cohérence de plusieurs machines virtuelles, vérifiez qu’elles ont toutes le même système d’exploitation (Windows ou Linux) et que le serveur cible maître que vous déployez a le même type de système d’exploitation. Toutes les machines virtuelles d’un groupe de réplication doivent utiliser le même serveur cible maître.
5. Ouvrez [les ports requis](vmware-azure-prepare-failback.md#ports-for-reprotectionfailback) pour la restauration automatique.
6. Assurez-vous que le serveur vCenter est connecté avant la restauration automatique. Sinon, la déconnexion des disques et leur attachement à la machine virtuelle échouent.
7. Si un serveur vCenter gère les machines virtuelles vers lesquelles vous allez effectuer une restauration automatique, vérifiez que vous disposez des autorisations nécessaires. Si vous effectuez une découverte utilisateur vCenter en mode lecture seule et protégez des machines virtuelles, la protection réussit et le basculement fonctionne. Cependant, pendant la reprotection, le basculement échoue parce que les magasins de données ne peuvent pas être détectés et ne sont pas listés. Pour résoudre ce problème, vous pouvez mettre à jour les informations d’identification vCenter avec [des autorisations ou un compte appropriés](avs-tutorial-prepare-avs.md#prepare-an-account-for-automatic-discovery), puis faire une nouvelle tentative. 
8. Si vous avez utilisé un modèle pour créer vos machines virtuelles, vérifiez que chaque machine virtuelle possède son propre UUID pour les disques. Si l’UUID d’une machine virtuelle Azure VMware Solution est en conflit avec l’UUID du serveur cible maître (car les deux ont été créés à partir du même modèle), la reprotection échoue. Déployez à partir d’un autre modèle.
9. Si vous effectuez une restauration automatique vers un autre serveur vCenter, vérifiez que le nouveau serveur vCenter et le serveur cible maître sont détectés. Généralement, si cela n’est pas le cas, les magasins de données ne sont pas accessibles ou ne sont pas visibles dans **Reprotéger**.
10. Vérifiez les scénarios suivants, qui ne permettent pas de restauration automatique :
    - Si vous utilisez les éditions gratuites d’ESXi 5.5 ou de vSphere Hypervisor 6. Effectuez une mise à niveau vers une autre version.
    - Si vous avez un serveur physique Windows Server 2008 R2 SP1.
    - Les machines virtuelles VMware ne peuvent pas effectuer de restauration automatique vers Hyper-V.
    - Machines virtuelles qui ont été migrées.
    - Une machine virtuelle qui a été déplacée vers un autre groupe de ressources.
    - Une machine virtuelle Azure de réplication qui a été supprimée.
    - Une machine virtuelle Azure de réplication qui n’est pas protégée.
10. [Examinez en revue les types de restaurations automatiques](concepts-types-of-failback.md) que vous pouvez utiliser : récupération dans l’emplacement d’origine et récupération dans un autre emplacement.


## <a name="enable-reprotection"></a>Activer la reprotection

Activez la réplication. Vous pouvez reprotéger des machines virtuelles spécifiques ou un plan de récupération :

- Si vous reprotégez un plan de récupération, vous devez fournir les valeurs pour chaque machine protégée.
- Si les machines virtuelles appartiennent à un groupe de réplication pour assurer la cohérence de plusieurs machines virtuelles, elles peuvent uniquement être reprotégées à l’aide d’un plan de récupération. Les machines virtuelles d’un groupe de réplication doivent utiliser le même serveur cible maître

>[!NOTE]
>La quantité de données envoyées à partir d’Azure à la source ancienne lors de la reprotection peut être comprise entre 0 et la somme des tailles de disque de toutes les machines protégées. Elle ne peut pas être calculée.

### <a name="before-you-start"></a>Avant de commencer

- Après le démarrage d’une machine virtuelle dans Azure après le basculement, l’agent prend un certain temps (jusqu’à 15 minutes) pour se réinscrire auprès du serveur de configuration. Pendant ce temps, vous ne pouvez pas reprotéger, et un message d’erreur indique que l’agent n’est pas installé. Le cas échéant, patientez quelques minutes, puis relancez la reprotection.
- Si vous voulez restaurer automatiquement la machine virtuelle Azure sur une machine virtuelle Azure VMware Solution existante, montez les magasins de données de machines virtuelles avec un accès en lecture/écriture sur l’hôte ESXi du serveur cible maître.
- Si vous souhaitez effectuer la restauration automatique à un autre emplacement, par exemple si la machine virtuelle Azure VMware Solution n’existe pas, sélectionnez le lecteur de conservation et le magasin de données qui sont configurés pour le serveur cible maître. Si vous effectuez la restauration automatique vers le cloud privé Azure VMware Solution, les machines virtuelles du plan de protection de la restauration automatique utilisent le même magasin de données que le serveur cible maître. Une machine virtuelle est alors créée dans vCenter.

Activez la reprotection comme suit :

1. Sélectionnez **Coffre** > **Éléments répliqués**. Cliquez avec le bouton droit sur la machine virtuelle qui a basculé et sélectionnez **Reprotéger**. Ou, à partir des boutons de commande, sélectionnez la machine et **Reprotéger**.
2. Vérifiez que le sens de protection **D’Azure à local** est activé.
3. Dans les champs **Serveur cible maître** et **Serveur de processus**, sélectionnez le serveur cible maître local et le serveur de processus.  
4. Pour **Magasin de données**, sélectionnez le magasin de données dans lequel vous souhaitez récupérer les disques d’Azure VMware Solution. Cette option est utilisée quand la machine virtuelle Azure VMware Solution est supprimée et que vous devez créer de nouveaux disques. Cette option est ignorée si les disques existent déjà. Vous devez quand même spécifier une valeur.
5. Sélectionnez le lecteur de rétention.
6. La stratégie de restauration automatique est sélectionnée automatiquement.
7. Sélectionnez **OK** pour commencer la reprotection.

    ![Reprotéger la boîte de dialogue](./media/vmware-azure-reprotect/reprotectinputs.png)
    
8. Une tâche de réplication de la machine virtuelle Azure sur le cloud privé Azure VMware Solution débute. Vous pouvez en suivre la progression sous l’onglet **Tâches** .
    - Une fois la reprotection réussie, la machine virtuelle passe à l’état protégé.
    - La machine virtuelle Azure VMware Solution est désactivée pendant la reprotection. Cela permet de garantir la cohérence des données pendant la réplication.
    - N’activez pas la machine virtuelle Azure VMware Solution à l’issue de la reprotection.
   

## <a name="next-steps"></a>Étapes suivantes

- Si vous rencontrez des problèmes, consultez l’[article sur la résolution des problèmes](vmware-azure-troubleshoot-failback-reprotect.md).
- Une fois les machines virtuelles Azure protégées, vous pouvez [exécuter une restauration automatique](avs-tutorial-failback.md). La restauration automatique arrête la machine virtuelle Azure et démarre la machine virtuelle Azure VMware Solution. Prévoyez un temps d’arrêt pour l’application et choisissez une heure de restauration en conséquence.


