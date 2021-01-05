---
title: Exécuter un basculement pendant la récupération d’urgence avec Azure Site Recovery
description: Guide pratique pour basculer des machines virtuelles/serveurs physiques vers Azure avec Azure Site Recovery.
ms.service: site-recovery
ms.topic: article
ms.date: 12/10/2019
ms.openlocfilehash: 6737f64773f91ede1631d42cd7f28c7d961c0454
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92368619"
---
# <a name="run-a-failover-from-on-premises-to-azure"></a>Effectuer un basculement depuis le site local vers Azure

Cet article explique comment basculer des machines locales vers Azure dans [Azure Site Recovery](site-recovery-overview.md)

## <a name="before-you-start"></a>Avant de commencer

- [Découvrez](failover-failback-overview.md) le processus de basculement en cas de récupération d’urgence.
- Si vous souhaitez effectuer le basculement de plusieurs ordinateurs, [découvrez](recovery-plan-overview.md) comment rassembler des machines dans un plan de récupération.
- Avant de procéder à un basculement complet, effectuez un [test de récupération d’urgence](site-recovery-test-failover-to-azure.md) pour vérifier que tout fonctionne comme prévu.

## <a name="prepare-to-connect-after-failover"></a>Préparer la connexion après le basculement

Pour vous assurer que vous pouvez vous connecter aux machines virtuelles Azure créées après le basculement, voici un certain nombre de choses que vous devez effectuer localement avant le basculement.


### <a name="prepare-on-premises-to-connect-after-failover"></a>Préparer localement la connexion après le basculement

Si vous voulez vous connecter à des machines virtuelles Azure à l’aide de RDP/SSH après le basculement, voici un certain nombre de choses que vous devez effectuer localement avant le basculement.

**Après le basculement** | **Lieu** | **Actions**
--- | --- | ---
**Machine virtuelle Azure exécutant Windows** | Machine locale avant le basculement | Pour accéder à la machine virtuelle Azure par Internet, activez la fonction RDP, vérifiez que les règles TCP et UDP sont ajoutées pour **Public** et que RDP est autorisé pour tous les profils dans **Pare-feu Windows** > **Applications autorisées** .<br/><br/> Pour accéder à la machine virtuelle Azure via une connexion de site à site, activez RDP sur la machine, en vérifiant que ce dernier est autorisé dans **Pare-feu Windows** -> **Applications et fonctionnalités autorisées** , pour les réseaux de types **Domaine et Privé** .<br/><br/> <br/><br/> Supprimez tous les itinéraires statiques persistants et le proxy WinHTTP. Vérifiez que la stratégie SAN du système d’exploitation est définie sur la valeur **OnlineAll** . [Plus d’informations](https://support.microsoft.com/kb/3031135)<br/><br/> Vérifiez qu’aucune mise à jour de Windows n’est en attente sur la machine virtuelle quand vous déclenchez un basculement. Une mise à jour de Windows peut démarrer quand vous procédez au basculement, et vous ne pouvez pas ouvrir de session sur la machine virtuelle tant que la mise à jour n’est pas terminée.
**Machine virtuelle Azure exécutant Linux** | Machine locale avant le basculement | Vérifiez que le service Secure Shell, sur la machine virtuelle, est défini pour démarrer automatiquement au démarrage du système.<br/><br/> Vérifiez que les règles de pare-feu autorisent une connexion SSH à ce dernier.


## <a name="run-a-failover"></a>Exécuter un basculement

Cette procédure explique comment exécuter un basculement dans le cadre d’un [plan de récupération](site-recovery-create-recovery-plans.md). Si vous souhaitez exécuter un basculement pour une seule machine virtuelle, suivez les instructions relatives à une [machine virtuelle VMware](vmware-azure-tutorial-failover-failback.md), à un [serveur physique](physical-to-azure-failover-failback.md) ou à une [machine virtuelle Hyper-V](hyper-v-azure-failover-failback-tutorial.md).


Exécutez le basculement du plan de récupération comme suit :

1. Dans le coffre Site Recovery, sélectionnez **Plans de récupération** > *recoveryplan_name* .
2. Cliquez sur **Basculement** .

    ![Capture d’écran d’Azure Site Recovery montrant le volet ADRP avec l’option Basculement sélectionnée dans le menu Plus.](./media/site-recovery-failover/Failover.png)

3. Dans **Basculement** > **Direction du basculement** , conservez la valeur par défaut si vous effectuez une réplication vers Azure.
4. Dans **Basculer** , sélectionnez un **point de récupération** vers lequel effectuer le basculement.

    - **Les dernières**  : Utilisez le point le plus récent. Cela traite toutes les données qui ont été envoyées au service Site Recovery et crée un point de récupération pour chaque ordinateur. Cette option fournit l’objectif de point de récupération (RPO) le plus faible, car la machine virtuelle créée après le basculement comporte toutes les données répliquées vers Site Recovery au moment où le basculement a été déclenché.
    Notez que lorsque la région source tombe en panne, il n’y a plus de traitement des journaux possible. Vous devez donc basculer vers le dernier point de récupération traité. Pour en savoir plus, consultez le point suivant.
   - **Dernier point traité**  : Utilisez cette option pour basculer les machines virtuelles vers le dernier point de récupération déjà traité par Site Recovery. Vous pouvez voir le dernier point de récupération traité dans les **derniers points de récupération** de la machine virtuelle. Cette option fournit un objectif de délai de récupération faible, car aucun temps n’est consacré à traiter les données non traitées.
   - **Dernier point de cohérence des applications**  : Utilisez cette option pour basculer les machines virtuelles vers le dernier point de récupération de cohérence des applications traité par Site Recovery.
   - **Dernier point multimachine virtuelle traité**  :  Avec cette option, les machines virtuelles appartenant à un groupe de réplication basculent vers le point de récupération multimachine virtuelle cohérent commun. Les autres machines virtuelles basculent vers leur dernier point de récupération traité. Cette option est disponible uniquement pour les plans de récupération qui incluent au moins une machine virtuelle avec la cohérence multimachine virtuelle activée.
   - **Dernière cohérence des applications multimachines virtuelles**  : Avec cette option, les machines virtuelles qui font partie d’un groupe de réplication basculent vers le dernier point de récupération de cohérence des applications multimachine virtuelle commun. Les autres machines virtuelles basculent vers leur dernier point de récupération d’application cohérent. Disponible uniquement pour les plans de récupération qui incluent au moins une machine virtuelle avec la cohérence multimachine virtuelle activée.
   - **Personnalisé**  : Non disponible pour les plans de récupération. Cette option concerne uniquement le basculement de machines virtuelles individuelles.

5. Sélectionnez **Arrêter la machine avant de commencer le basculement** si vous souhaitez que Site Recovery arrête les machines virtuelles sources avant de démarrer le basculement. Le basculement est effectué même en cas d’échec de l’arrêt.  

    > [!NOTE]
    > Si vous effectuez le basculement de machines virtuelles Hyper-V, l’arrêt tente de synchroniser et de répliquer les données locales qui n’ont pas encore été envoyées au service avant de déclencher le basculement. 

6. Suivez la progression du basculement sur la page **Tâches** . Même si des erreurs se produisent, le plan de récupération s’exécute jusqu’à la fin.
7. Après le basculement, connectez-vous à la machine virtuelle pour la valider. 
8. Utilisez **Changer le point de récupération** , si vous souhaitez vous servir d’un autre point de récupération pour le basculement.
9. Lorsque vous êtes prêt, vous pouvez valider le basculement. L’action **Valider** supprime tous les points de récupération disponibles avec le service. L’option **Changer le point de récupération** n’est plus disponible.

## <a name="run-a-planned-failover-hyper-v"></a>Exécuter un basculement planifié (Hyper-V)

Vous pouvez exécuter un basculement planifié pour des machines virtuelles Hyper-V.

- Un basculement planifié évite toute perte de données.
- Lorsqu’un basculement planifié se déclenche, les machines virtuelles sources sont arrêtées, les données les plus récentes sont synchronisées, puis un basculement est effectué.
- Vous exécutez un basculement planifié à l’aide de l’option **Basculement planifié** . Il s’exécute de façon similaire à un basculement normal.
 
## <a name="track-failovers"></a>Suivre les basculements

Un certain nombre de travaux sont associés au basculement.

![Capture d’écran de la page Travaux montrant une liste de travaux avec Groupe 1 : Démarrer(1) développé dans la colonne Nom. La ligne du travail SQLServer est mise en surbrillance.](./media/site-recovery-failover/FailoverJob.png)

- **Vérification des prérequis**  : Permet de garantir que toutes les conditions requises pour le basculement sont satisfaites.
- **Basculement** : Traite les données pour permettre la création d’une machine virtuelle Azure. Si vous avez choisi le **dernier** point de récupération, un point de récupération est créé à partir des données envoyées au service.
- **Démarrer** : Crée une machine virtuelle Azure en utilisant les données traitées à l’étape précédente.

> [!WARNING]
> **N’annulez pas un basculement en cours**  : Avant le démarrage d’un basculement, la réplication de la machine virtuelle est arrêtée. Si vous annulez un travail en cours, le basculement s’arrête, mais la réplication de la machine virtuelle ne démarre pas. Il n’est plus possible ensuite de démarrer à nouveau la réplication.


### <a name="extra-failover-time"></a>Temps de basculement supplémentaire

Dans certains cas, le basculement de machine virtuelle nécessite une étape intermédiaire qui prend généralement entre 8 et 10 minutes pour s’accomplir. Voici les machines qui sont affectées par cette étape/ce temps supplémentaire :

* Machines virtuelles VMware exécutant une version du service Mobility antérieure à la version 9.8.
* Serveurs physiques et machines virtuelles Hyper-V protégées en tant que serveurs physiques.
* Machines virtuelles VMware Linux
* Machines virtuelles VMware sur lesquelles ces pilotes ne sont pas présents en tant que pilotes de démarrage :
    * storvsc
    * vmbus
    * storflt
    * intelide
    * atapi
* Machines virtuelles VMware pour lesquelles DHCP n’est pas activé, qu’elles utilisent ou non des adresses DHCP ou IP statiques.


## <a name="automate-actions-during-failover"></a>Automatiser des actions pendant le basculement

Vous souhaiterez peut-être automatiser des actions pendant le basculement. Pour ce faire, vous pouvez utiliser des scripts ou des runbooks Azure Automation dans les plans de récupération.

- [Découvrez](site-recovery-create-recovery-plans.md) la création et la personnalisation des plans de récupération, notamment l’ajout de scripts.
- [Découvrez](site-recovery-runbook-automation.md) l’ajout de runbooks Azure Automation à des plans de récupération.


## <a name="configure-settings-after-failover"></a>Configurer des paramètres après un basculement

### <a name="retain-drive-letters-after-failover"></a>Conserver les lettres de lecteur après le basculement

Site Recovery gère la rétention des lettres de lecteur. Si vous excluez des disques pendant la réplication de machine virtuelle, [passez en revue un exemple](exclude-disks-replication.md#example-1-exclude-the-sql-server-tempdb-disk) de la façon dont cela fonctionne.

### <a name="prepare-in-azure-to-connect-after-failover"></a>Préparer dans Azure pour la connexion après le basculement

Si vous souhaitez vous connecter à des machines virtuelles Azure qui sont créées après le basculement à l’aide de RDP ou de SSH, respectez les exigences récapitulées dans le tableau.

**Type de basculement** | **Lieu** | **Actions**
--- | --- | ---
**Machine virtuelle Azure exécutant Windows** | Machine virtuelle Azure après le basculement |  [Ajoutez une adresse IP publique](/archive/blogs/srinathv/how-to-add-a-public-ip-address-to-azure-vm-for-vm-failed-over-using-asr) pour la machine virtuelle.<br/><br/> Les règles des groupes de sécurité réseau figurant sur la machine virtuelle basculée (et le sous-réseau Azure auquel elle est connectée) doivent autoriser les connexions entrantes avec le port RDP.<br/><br/> Cochez **Diagnostics de démarrage** pour examiner une capture d’écran de la machine virtuelle.<br/><br/> Si vous ne parvenez pas à vous connecter, vérifiez que la machine virtuelle est en cours d’exécution et consultez ces [conseils de résolution des problèmes](https://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx).
**Machine virtuelle Azure exécutant Linux** | Machine virtuelle Azure après le basculement | Les règles des groupes de sécurité réseau figurant sur la machine virtuelle basculée (et le sous-réseau Azure auquel elle est connectée) doivent autoriser les connexions entrantes avec le port SSH.<br/><br/> [Ajoutez une adresse IP publique](/archive/blogs/srinathv/how-to-add-a-public-ip-address-to-azure-vm-for-vm-failed-over-using-asr) pour la machine virtuelle.<br/><br/> Cochez **Diagnostics de démarrage** pour obtenir une capture d’écran de la machine virtuelle.<br/><br/>

Suivez les étapes décrites [ici](site-recovery-failover-to-azure-troubleshoot.md) pour résoudre les problèmes de connectivité après le basculement.

## <a name="set-up-ip-addressing"></a>Configurer l’adressage IP

- **Adresses IP internes** : Pour définir l’adresse IP interne d’une machine virtuelle Azure après le basculement, vous disposez de deux options :
    - Conserver la même adresse IP : vous pouvez utiliser la même adresse IP sur la machine virtuelle Azure que celle allouée à l’ordinateur local.
    - Utiliser une autre adresse IP : vous pouvez utiliser une adresse IP différente pour la machine virtuelle Azure.
    - [En savoir plus](concepts-on-premises-to-azure-networking.md#assign-an-internal-address) sur la configuration des adresses IP internes.
- **Adresses IP externes** : Vous pouvez conserver les adresses IP publiques lors du basculement. Les machines virtuelles Azure créées dans le cadre du processus de basculement doivent être affectées à une adresse IP publique Azure disponible dans la région Azure. Vous pouvez affecter une adresse IP publique soit manuellement, soit en automatisant le processus à l’aide d’un plan de récupération. [Plus d’informations](concepts-public-ip-address-with-site-recovery.md)


## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez effectué le basculement, vous devez réactiver la protection pour démarrer la réplication des machines virtuelles Azure sur le site local. Une fois que la réplication est en cours d’exécution, vous pouvez effectuer une restauration locale lorsque vous êtes prêt.

- [En savoir plus](failover-failback-overview.md#reprotectionfailback) sur la reprotection et la restauration automatique.
- [Faire les préparations ](vmware-azure-reprotect.md) de la reprotection et de la restauration VMware.
- [Restaurer automatiquement](hyper-v-azure-failback.md) des machines virtuelles Hyper-V.
- [En savoir plus](physical-to-azure-failover-failback.md) sur le processus de basculement et de restauration automatique pour les serveurs physiques.