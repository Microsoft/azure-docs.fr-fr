---
title: Reprotéger les machines virtuelles VMware sur un site local à l’aide d’Azure Site Recovery
description: Découvrez comment reprotéger les machines virtuelles VMware après le basculement vers Azure avec Azure Site Recovery.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 12/17/2019
ms.author: mayg
ms.openlocfilehash: 6a11e3d0cb41383b44b76975ecbd1c2ae2825015
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89441491"
---
# <a name="reprotect-from-azure-to-on-premises"></a>Reprotection d’Azure vers votre site local

Après un [basculement](site-recovery-failover.md) de machines virtuelles VMware ou de serveurs physiques locaux vers Azure, la première étape de restauration automatique vers votre site local implique de reprotéger les machines virtuelles Azure qui ont été créées pendant le basculement. Cet article décrit ce processus de reprotection. 

## <a name="before-you-begin"></a>Avant de commencer

1. Suivez les étapes décrites dans [cet article](vmware-azure-prepare-failback.md) pour préparer la reprotection et la restauration automatique, notamment la configuration d’un serveur de processus dans Azure et d’un serveur cible maître local, ainsi que la configuration d’un VPN de site à site ou d’une homologation privée ExpressRoute pour la restauration automatique.
2. Vérifiez que le serveur de configuration local est en cours d’exécution et qu’il est connecté à Azure. Pendant le basculement vers Azure, le site local risque de ne pas être accessible et le serveur de configuration peut être indisponible ou à l’arrêt. Lors de la restauration automatique, la machine virtuelle doit exister dans la base de données du serveur de configuration. Sinon, la restauration automatique échoue.
3. Supprimez les captures instantanées sur le serveur cible maître local. La reprotection ne fonctionnera pas en présence de captures instantanées.  Les captures instantanées sur la machine virtuelle sont automatiquement fusionnées lors d’une tâche de reprotection.
4. Si vous reprotégez des machines virtuelles rassemblées dans un groupe de réplication pour assurer la cohérence de plusieurs machines virtuelles, vérifiez qu’elles ont toutes le même système d’exploitation (Windows ou Linux) et que le serveur cible maître que vous déployez a le même type de système d’exploitation. Toutes les machines virtuelles d’un groupe de réplication doivent utiliser le même serveur cible maître.
5. Ouvrez [les ports requis](vmware-azure-prepare-failback.md#ports-for-reprotectionfailback) pour la restauration automatique.
6. Assurez-vous que le serveur vCenter est connecté avant la restauration automatique. Sinon, la déconnexion des disques et leur attachement à la machine virtuelle échouent.
7. Si un serveur vCenter gère les machines virtuelles vers lesquelles vous allez effectuer une restauration automatique, vérifiez que vous disposez des autorisations nécessaires. Si vous effectuez une découverte utilisateur vCenter en mode lecture seule et protégez des machines virtuelles, la protection réussit et le basculement fonctionne. Cependant, pendant la reprotection, le basculement échoue parce que les magasins de données ne peuvent pas être détectés et ne sont pas listés. Pour résoudre ce problème, vous pouvez mettre à jour les informations d’identification vCenter avec [des autorisations ou un compte appropriés](vmware-azure-tutorial-prepare-on-premises.md#prepare-an-account-for-automatic-discovery), puis faire une nouvelle tentative. 
8. Si vous avez utilisé un modèle pour créer vos machines virtuelles, vérifiez que chaque machine virtuelle possède son propre UUID pour les disques. Si l’UUID de la machine virtuelle locale est en conflit avec l’UUID du serveur cible maître (car les deux ont été créés à partir du même modèle), la reprotection échoue. Déployez à partir d’un autre modèle.
9. Si vous effectuez une restauration automatique vers un autre serveur vCenter, vérifiez que le nouveau serveur vCenter et le serveur cible maître sont détectés. Généralement, si cela n’est pas le cas, les magasins de données ne sont pas accessibles ou ne sont pas visibles dans **Reprotéger**.
10. Vérifiez les scénarios suivants, qui ne permettent pas de restauration automatique :
    - Si vous utilisez les éditions gratuites d’ESXi 5.5 ou de vSphere Hypervisor 6. Effectuez une mise à niveau vers une autre version.
    - Si vous avez un serveur physique Windows Server 2008 R2 SP1.
    - Les machines virtuelles VMware ne peuvent pas effectuer de restauration automatique vers Hyper-V.
    - Machines virtuelles qui ont été migrées.
    - Une machine virtuelle qui a été déplacée vers un autre groupe de ressources.
    - Une machine virtuelle Azure de réplication qui a été supprimée.
    - Une machine virtuelle Azure de réplication qui n’est pas protégée (réplication sur le site local).
10. [Examinez en revue les types de restaurations automatiques](concepts-types-of-failback.md) que vous pouvez utiliser : récupération dans l’emplacement d’origine et récupération dans un autre emplacement.


## <a name="enable-reprotection"></a>Activer la reprotection

Activez la réplication. Vous pouvez reprotéger des machines virtuelles spécifiques ou un plan de récupération :

- Si vous reprotégez un plan de récupération, vous devez fournir les valeurs pour chaque machine protégée.
- Si les machines virtuelles appartiennent à un groupe de réplication pour assurer la cohérence de plusieurs machines virtuelles, elles peuvent uniquement être reprotégées à l’aide d’un plan de récupération. Les machines virtuelles d’un groupe de réplication doivent utiliser le même serveur cible maître

>[!NOTE]
>La quantité de données envoyées à partir d’Azure à la source ancienne lors de la reprotection peut être comprise entre 0 et la somme des tailles de disque de toutes les machines protégées. Elle ne peut pas être calculée.

### <a name="before-you-start"></a>Avant de commencer

- Après le démarrage d’une machine virtuelle dans Azure après le basculement, l’agent prend un certain temps (jusqu’à 15 minutes) pour se réinscrire auprès du serveur de configuration. Pendant ce temps, vous ne pouvez pas reprotéger, et un message d’erreur indique que l’agent n’est pas installé. Le cas échéant, patientez quelques minutes, puis relancez la reprotection.
- Si vous voulez restaurer automatiquement la machine virtuelle Azure sur une machine virtuelle locale existante, montez les magasins de données de la machine virtuelle locale avec un accès en lecture/écriture sur l’hôte ESXi du serveur cible maître.
- Si vous souhaitez effectuer la restauration automatique à un autre emplacement, par exemple si la machine virtuelle locale n’existe pas, sélectionnez le lecteur de rétention et le magasin de données configurés pour le serveur cible maître. Si vous effectuez la restauration automatique vers le site local, les machines virtuelles VMware du plan de protection de la restauration automatique utilisent la même banque de données que le serveur cible maître. Une machine virtuelle est alors créée dans vCenter.

Activez la reprotection comme suit :

1. Sélectionnez **Coffre** > **Éléments répliqués**. Cliquez avec le bouton droit sur la machine virtuelle qui a basculé et sélectionnez **Reprotéger**. Ou, à partir des boutons de commande, sélectionnez la machine et **Reprotéger**.
2. Vérifiez que le sens de protection **D’Azure à local** est activé.
3. Dans les champs **Serveur cible maître** et **Serveur de processus**, sélectionnez le serveur cible maître local et le serveur de processus.  
4. Pour **Banque de données**, sélectionnez la banque de données dans laquelle vous souhaitez récupérer les disques locaux. Cette option est utilisée lorsque la machine virtuelle locale est supprimée et que vous devez créer de nouveaux disques. Cette option est ignorée si les disques existent déjà. Vous devez quand même spécifier une valeur.
5. Sélectionnez le lecteur de rétention.
6. La stratégie de restauration automatique est sélectionnée automatiquement.
7. Sélectionnez **OK** pour commencer la reprotection.

    ![Reprotéger la boîte de dialogue](./media/vmware-azure-reprotect/reprotectinputs.png)
    
8. Un travail de réplication de la machine virtuelle Azure sur le site local commence. Vous pouvez en suivre la progression sous l’onglet **Tâches** .
    - Une fois la reprotection réussie, la machine virtuelle passe à l’état protégé.
    - La machine virtuelle locale est désactivée au cours de la reprotection. Cela permet de garantir la cohérence des données pendant la réplication.
    - N’activez pas la machine virtuelle locale une fois la reprotection terminée.
   

## <a name="next-steps"></a>Étapes suivantes

- Si vous rencontrez des problèmes, consultez l’[article sur la résolution des problèmes](vmware-azure-troubleshoot-failback-reprotect.md).
- Une fois les machines virtuelles Azure protégées, vous pouvez [exécuter une restauration automatique](vmware-azure-failback.md). La restauration automatique arrête la machine virtuelle Azure et démarre la machine virtuelle locale. Prévoyez un temps d’arrêt pour l’application et choisissez une heure de restauration en conséquence.


