---
title: À propos du basculement et de la restauration automatique dans Azure Site Recovery
description: En savoir plus sur le basculement et les défaillances dans Azure Site Recovery.
ms.topic: conceptual
ms.date: 12/24/2019
ms.openlocfilehash: 3617683200aa3ffba08061b70993613fd0cc7241
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92369877"
---
# <a name="about-on-premises-disaster-recovery-failoverfailback"></a>À propos du basculement/restauration de la récupération d’urgence au niveau local

Cet article fournit une vue d’ensemble du basculement et de la restauration automatique lors de la récupération d’urgence des machines locales vers Azure avec [Azure Site Recovery](site-recovery-overview.md).

## <a name="recovery-stages"></a>Étapes de la récupération

Le basculement et la restauration automatique dans Azure Site Recovery comportent quatre étapes :

- **Étape 1 : Basculement vers un site local** : Après la configuration de la réplication vers Azure pour les machines locales, lorsque votre site local tombe en panne, vous basculez ces machines vers Azure. Une fois le basculement effectué, les machines virtuelles Azure sont créées à partir des données répliquées.
- **Étape 2 : Reprotéger des machines virtuelles Azure :** Dans Azure, vous reprotégez les machines virtuelles Azure afin qu’elles soient répliquées sur le site local. La machine virtuelle locale (si elle est disponible) est désactivée au cours de la reprotection afin de garantir la cohérence des données.
- **Étape 3 : Basculement à partir d’Azure** : Lorsque votre site local s’exécute de nouveau normalement, vous exécutez un autre basculement, cette fois pour restaurer les machines virtuelles Azure sur votre site local. Vous pouvez effectuer une restauration automatique vers l’emplacement d’origine à partir duquel vous avez basculé ou vers un autre emplacement.
- **Étape 4 : Reprotéger les machines locales** : Après une restauration automatique, activez à nouveau la réplication des machines locales vers Azure.

## <a name="failover"></a>Basculement

Vous effectuez un basculement dans le cadre de votre stratégie de continuité d’activité et de reprise d’activité (BCDR).

- La première étape de votre stratégie BCDR consiste à répliquer vos machines locales sur Azure de manière continue. Les utilisateurs accèdent aux charges de travail et applications qui s’exécutent sur les machines sources locales.
- En cas de besoin, par exemple en cas de panne en local, les machines de réplication sont basculées vers Azure. Les machines virtuelles Azure sont créées à l’aide des données répliquées.
- Pour la continuité des activités, les utilisateurs peuvent continuer à accéder aux applications sur les machines virtuelles Azure.

Le basculement est une activité en deux phases :

- **Basculement** : Basculement qui crée et affiche une machine virtuelle Azure à l’aide du point de récupération sélectionné.
- **Valider**  : Après le basculement, vous vérifiez la machine virtuelle dans Azure :
    - Vous pouvez ensuite valider le basculement vers le point de récupération sélectionné ou sélectionner un autre point pour la validation.
    - Une fois le basculement validé, le point de récupération ne peut pas être modifié.


## <a name="connect-to-azure-after-failover"></a>Se connecter à Azure après le basculement

Pour vous connecter aux machines virtuelles Azure créées après le basculement à l’aide de RDP/SSH, il existe un certain nombre d’exigences.

**Type de basculement** | **Lieu** | **Actions**
--- | --- | ---
**Machine virtuelle Azure (Windows** | Sur la machine locale, avant le basculement | **Accès via Internet** : Activer le protocole RDP. Assurez-vous que les règles TCP et UDP sont ajoutées pour le profil **Public** et que RDP est autorisé pour tous les profils dans **Pare-feu Windows** > **Applications autorisées** .<br/><br/> **Accès via un VPN de site à site** : Activer le protocole RDP sur l’ordinateur. Vérifiez que RDP est autorisé dans **Pare-feu Windows** -> **Applications et fonctionnalités autorisées** , pour les réseaux **Domaine et Privé** .<br/><br/>  Vérifiez que la stratégie SAN du système d’exploitation est définie sur la valeur **OnlineAll** . [Plus d’informations](https://support.microsoft.com/kb/3031135)<br/><br/> Vérifiez qu’aucune mise à jour de Windows n’est en attente sur la machine virtuelle quand vous déclenchez un basculement. Windows Update peut démarrer lors du basculement, et vous ne pourrez pas vous connecter à la machine virtuelle tant que les mises à jour ne sont pas terminées.
**Machine virtuelle Azure exécutant Windows** | Sur la machine virtuelle Azure, après le basculement |  [Ajoutez une adresse IP publique](/archive/blogs/srinathv/how-to-add-a-public-ip-address-to-azure-vm-for-vm-failed-over-using-asr) pour la machine virtuelle.<br/><br/> Les règles des groupes de sécurité réseau sur la machine virtuelle basculée (et le sous-réseau Azure auquel elle est connectée) doivent autoriser les connexions entrantes avec le port RDP.<br/><br/> Cochez **Diagnostics de démarrage** pour examiner une capture d’écran de la machine virtuelle. Si vous ne parvenez pas à vous connecter, vérifiez que la machine virtuelle est en cours d’exécution et consultez ces [conseils de résolution des problèmes](https://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx).
**Machine virtuelle Azure exécutant Linux** | Sur la machine locale, avant le basculement | Vérifiez que le service Secure Shell, sur la machine virtuelle, est défini pour démarrer automatiquement au démarrage du système.<br/><br/> Vérifiez que les règles de pare-feu autorisent une connexion SSH à ce dernier.
**Machine virtuelle Azure exécutant Linux** | Sur la machine virtuelle Azure, après le basculement | Les règles des groupes de sécurité réseau figurant sur la machine virtuelle basculée (et le sous-réseau Azure auquel elle est connectée) doivent autoriser les connexions entrantes avec le port SSH.<br/><br/> [Ajoutez une adresse IP publique](/archive/blogs/srinathv/how-to-add-a-public-ip-address-to-azure-vm-for-vm-failed-over-using-asr) pour la machine virtuelle.<br/><br/> Cochez **Diagnostics de démarrage** pour obtenir une capture d’écran de la machine virtuelle.<br/><br/>

## <a name="types-of-failover"></a>Types de basculement

Site Recovery propose différentes options de basculement.

**Type de basculement** | **Détails** | **Récupération** | **Workflow**
--- | --- | --- | ---
**Test de basculement** | Utilisé pour exécuter une analyse qui valide votre stratégie BCDR, sans perte de données ni temps d’arrêt.| Crée une copie de la machine virtuelle dans Azure, sans impact sur la réplication en cours ou sur votre environnement de production. | 1. Exécutez un test de basculement sur une seule machine virtuelle ou sur plusieurs machines virtuelles dans un plan de récupération.<br/><br/> 2. Sélectionnez un point de récupération à utiliser pour le test de basculement.<br/><br/> 3. Sélectionnez un réseau Azure dans lequel la machine virtuelle Azure sera localisée lors de sa création après le basculement. Le réseau est utilisé uniquement pour le test de basculement.<br/><br/> 4. Vérifiez que le test a fonctionné comme prévu. Site Recovery nettoie automatiquement les machines virtuelles créées dans Azure pendant le test.
**Basculement planifié Hyper V**  | Généralement utilisé pour les temps d’arrêt planifiés.<br/><br/> Les machines virtuelles sources sont arrêtées. Les données les plus récentes sont synchronisées avant de lancer le basculement. | Aucune perte de données pour le flux de travail planifié. | 1. Planifiez une fenêtre de maintenance de temps d’arrêt et avertissez les utilisateurs.<br/><br/> 2. Mettez les applications orientées utilisateur hors connexion.<br/><br/> 3. Initiez un basculement planifié avec le dernier point de récupération. Le basculement ne s’exécute pas si la machine n’est pas arrêtée ou si des erreurs sont rencontrées.<br/><br/> 4. Après le basculement, vérifiez que la machine virtuelle Azure de réplica est active dans Azure.<br/><br/> 5. Validez le basculement pour terminer. L’action de validation annule tous les point de récupération.
**Basculement Hyper V** | S’exécute généralement en cas de panne non planifiée ou si le site principal n’est pas disponible.<br/><br/> Si vous le souhaitez, arrêtez la machine virtuelle et synchronisez les dernières modifications avant de lancer le basculement.  | Perte de données minimale pour les applications. | 1. Lancez votre plan BCDR. <br/><br/> 2. Lancez un basculement. Spécifiez si Site Recovery doit arrêter la machine virtuelle et synchroniser/répliquer les dernières modifications avant de déclencher le basculement.<br/><br/> 3. Vous pouvez basculer vers un certain nombre d'options de points de récupération, résumées dans le tableau ci-dessous.<br/><br/> Si vous n’activez pas l’option permettant d’arrêter la machine virtuelle, ou si Site Recovery ne pouvez pas l’arrêter, le point de récupération le plus récent est utilisé.<br/>Le basculement s’exécute même si la machine ne peut pas être arrêtée.<br/><br/> 4. Après le basculement, vous vérifiez que la machine virtuelle Azure de réplica est active dans Azure.<br/> Si nécessaire, vous pouvez sélectionner un autre point de récupération dans la fenêtre de rétention de 24 heures.<br/><br/> 5. Validez le basculement pour terminer. L’action de validation supprime tous les points de récupération disponibles.
**Basculement-VMware** | S’exécute généralement en cas de panne non planifiée ou si le site principal n’est pas disponible.<br/><br/> Spécifiez éventuellement que Site Recovery doit tenter de déclencher un arrêt de la machine virtuelle, et synchroniser et répliquer les modifications finales avant de lancer le basculement.  | Perte de données minimale pour les applications. | 1. Lancez votre plan BCDR. <br/><br/> 2. Initiez un basculement à partir de Site Recovery. Spécifiez si Site Recovery doit essayer de déclencher l’arrêt de la machine virtuelle et synchroniser avant d’exécuter le basculement.<br/> Le basculement s’exécute même si la machine ne peut pas être arrêtée.<br/><br/> 3. Après le basculement, vérifiez que la machine virtuelle Azure de réplica est active dans Azure. <br/>Si nécessaire, vous pouvez sélectionner un autre point de récupération dans la fenêtre de rétention de 72 heures.<br/><br/> 5. Validez le basculement pour terminer. L’action de validation annule tous les point de récupération.<br/> Pour les machines virtuelles Windows, Site Recovery désactive les outils VMware pendant le basculement.

## <a name="failover-processing"></a>Traitement du basculement

Dans certains scénarios, le basculement nécessite un traitement supplémentaire qui dure environ 8 à 10 minutes. Vous noterez éventuellement que les délais des tests de basculement sont plus longs pour les machines suivantes :

* Machines virtuelles VMware exécutant une version du service Mobilité antérieure à la version 9.8
* Serveurs physiques
* Machines virtuelles VMware Linux
* Machines virtuelles Hyper-V protégées en tant que serveurs physiques
* Machines virtuelles VMware sur lesquelles le service DHCP n’est pas activé
* Machines virtuelles VMware qui n’ont pas les pilotes de démarrage suivants : storvsc, vmbus, storflt, intelide, atapi.

## <a name="recovery-point-options"></a>Options de point de récupération

Pendant le basculement, vous pouvez sélectionner un certain nombre d’options de point de récupération.

**Option** | **Détails**
--- | ---
**Dernier (objectif de point de récupération le plus faible)** | Cette option fournit l’objectif de point de récupération (RPO) le plus bas. Il traite tout d’abord toutes les données qui ont été envoyées au service Site Recovery, afin de créer un point de récupération pour chaque machine virtuelle avant de basculer vers celle-ci. Toutes les données sont répliquées sur ce point de récupération sur Site Recovery lorsque le basculement a été déclenché.
**Dernier point traité**  | Cette option bascule les machines virtuelles vers le dernier point de récupération traité par Site Recovery. Pour voir le dernier point de récupération d’une machine virtuelle spécifique, cochez **Derniers points de récupération** dans les paramètres de la machine virtuelle. Cette option fournit un objectif de délai de récupération (RTO) faible, car aucun temps n’est consacré à traiter les données non traitées.
**Dernier point de cohérence des applications** |  Cette option bascule toutes les machines virtuelles du plan vers le dernier point de récupération de cohérence des applications traité par Site Recovery. Vérifiez le dernier point de récupération dans les paramètres de la machine virtuelle.
**Dernier point multimachine virtuelle traité** | cette option est disponible pour les plans de récupération qui comportent une ou plusieurs machines virtuelles dont la cohérence multimachine virtuelle est activée. Les machines virtuelles pour lesquelles ce paramètre est activé basculent vers le dernier point de récupération de cohérence multimachine virtuelle commun. Toutes les autres machines virtuelles du plan basculent vers le dernier point de récupération traité.
**Dernière cohérence des applications multimachines virtuelles**  : |  cette option est disponible pour les plans de récupération qui comportent une ou plusieurs machines virtuelles dont la cohérence multimachine virtuelle est activée. Les machines virtuelles qui font partie d’un groupe de réplication basculent vers le dernier point de récupération de cohérence des applications multimachine virtuelle commun. Les autres machines virtuelles basculent vers leur dernier point de récupération de cohérence des applications.
**Personnalisée** | Utilisez cette option pour basculer une machine virtuelle spécifique à un point de récupération particulier dans le temps. Cette option n’est pas disponible pour les plans de récupération.

> [!NOTE]
> Les points de récupération ne peuvent pas être migrés vers un autre coffre Recovery Services.

## <a name="reprotectionfailback"></a>Reprotection/restauration automatique

Après le basculement vers Azure, les machines virtuelles Azure répliquées sont dans un état non protégé.

- Pour effectuer une première étape de restauration vers votre site local, vous devez démarrer les machines virtuelles Azure répliquées en local. Le processus de reprotection dépend du type d’ordinateur sur lequel vous avez basculé.
- Après la réplication des machines d’Azure vers le site local, vous pouvez exécuter un basculement à partir d’Azure vers votre site local.
- Une fois les machines réexécutées en local, vous pouvez activer la réplication afin qu’elles soient répliquées vers Azure pour la récupération d’urgence.

La restauration automatique fonctionne comme suit :

- Pour effectuer la restauration automatique, une machine virtuelle a besoin d’au moins un point de récupération pour pouvoir être restaurée. Dans un plan de récupération, toutes les machines virtuelles du plan ont besoin d’au moins un point de récupération.
- Nous vous recommandons d’utiliser le **dernier** point de récupération pour effectuer une restauration automatique (il s’agit d’un point de cohérence en cas d’incident).
    - Il existe une option de point de récupération de cohérence des applications. Dans ce cas, une seule machine virtuelle est récupérée vers son dernier point de récupération cohérent d’application disponible. Pour un plan de récupération avec un groupe de réplication, chaque groupe de réplication est récupéré à son point de récupération disponible commun.
    - Les points de récupération de cohérence des applications peuvent être en retard, et il peut y avoir une perte de données.
- Lors du basculement d’Azure vers le site local, Site Recovery arrête les machines virtuelles Azure. Lorsque vous validez le basculement, Site Recovery supprime les machines virtuelles Azure restaurées automatiquement dans Azure.


## <a name="vmwarephysical-reprotectionfailback"></a>VMware/reprotection physique/restauration automatique

Pour reprotéger et restaurer automatiquement des machines VMware et des serveurs physiques à partir d’Azure vers le site local, vous avez besoin d’une infrastructure de restauration automatique et il existe un certain nombre d’exigences.

- **Serveur de traitement temporaire dans Azure**  : pour effectuer une restauration automatique à partir d’Azure, configurez une machine virtuelle Azure faisant office de serveur de traitement pour gérer la réplication à partir d’Azure. Vous pourrez supprimer cette machine virtuelle une fois la restauration terminée.
- **Connexion VPN**  : pour la restauration automatique, vous avez besoin d’une connexion VPN (ou ExpressRoute) du réseau Azure vers le site local.
- **Serveur cible maître distinct**  : par défaut, le serveur cible maître local qui a été installé avec le serveur de configuration, sur la machine virtuelle VMware locale, gère la restauration automatique. Si vous avez besoin de restaurer automatiquement de grands volumes de trafic, configurez un autre serveur cible maître local dédié.
- **Stratégie de restauration automatique**  : pour répliquer vers votre site local, vous avez besoin d’une stratégie de restauration automatique. Cette stratégie est automatiquement créée en même temps que la stratégie de réplication depuis le site local vers Azure.
    - Cette stratégie est automatiquement associée au serveur de configuration.
    - Vous ne pouvez pas modifier cette stratégie.
    - Valeurs de stratégie : Seuil de RPO - 15 minutes ; Rétention des points de récupération - 24 heures ; Fréquence des instantanés de cohérence des applications - 60 minutes.

En savoir plus sur VMware, la reprotection physique et la restauration automatique :
- [Passer en revue](vmware-azure-reprotect.md#before-you-begin) les exigences supplémentaires pour la reprotection et la restauration automatique.
- [Déployer](vmware-azure-prepare-failback.md#deploy-a-process-server-in-azure) un serveur de processus dans Azure.
- [Déployer](vmware-azure-prepare-failback.md#deploy-a-separate-master-target-server) un serveur cible maître distinct.

Lorsque vous reprotégez des machines virtuelles Azure en local, vous pouvez spécifier que vous souhaitez effectuer une restauration automatique vers l’emplacement d’origine ou vers un autre emplacement.

- **Récupération à l’emplacement d’origine** : Cette opération bascule à partir d’Azure vers la même machine locale source, si elle existe. Dans ce scénario, seules les modifications sont répliquées vers le serveur local.
- **Récupération à un autre emplacement** : Si la machine locale n’existe pas, vous pouvez effectuer une restauration automatique à partir d’Azure vers un autre emplacement. Lorsque vous reprotégez la machine virtuelle Azure en local, la machine locale est créée. La réplication complète des données s’effectue à partir d’Azure vers le site local. --[Passez en revue](concepts-types-of-failback.md) les exigences et limitations relatives à la restauration de l’emplacement.



## <a name="hyper-v-reprotectionfailback"></a>Reprotection/restauration automatique Hyper-V

Pour reprotéger et restaurer automatiquement des machines virtuelles Hyper-V à partir d’Azure vers le site local :

- Vous pouvez uniquement effectuer une restauration automatique des machines virtuelles Hyper-V répliquées à l’aide d’un compte de stockage. La restauration automatique des machines virtuelles Hyper-V répliquées à l’aide de disques managés n’est pas prise en charge.
- Les ordinateurs hôtes Hyper-V locaux (ou System Center VMM s’ils sont utilisés) doivent être connectés à Azure.
- Vous exécutez une restauration automatique planifiée d’Azure vers le site local.
- Aucun composant spécifique ne doit être configuré pour la restauration automatique des machines virtuelles Hyper-V.
- Pendant le basculement planifié, vous pouvez sélectionner des options pour synchroniser les données avant la restauration automatique :
    - **Synchroniser les données avant le basculement** : Cette option réduit le temps d’arrêt pour les machines virtuelles lors de la synchronisation des ordinateurs sans les arrêter.
        - Phase 1 : Elle prend une capture instantanée de la machine virtuelle Azure et la copie sur l’hôte Hyper-V local. La machine continue de s’exécuter dans Microsoft Azure.
        - Phase 2 : Elle arrête la machine virtuelle Azure : aucune nouvelle modification ne s’y produit donc. L’ensemble final des modifications d’ordre différentiel est transféré au serveur local et la machine virtuelle locale est démarrée.
    - **Synchroniser les données pendant le basculement uniquement** : Cette option est plus rapide, car nous pensons que la majeure partie du disque a changé et n’effectue donc pas de calculs de somme de contrôle. Elle effectue un téléchargement du disque. Nous vous recommandons d’utiliser cette option si la machine virtuelle a été exécutée dans Azure pendant un certain temps (un mois ou plus) ou si la machine virtuelle locale a été supprimée.

[En savoir plus](hyper-v-azure-failback.md) sur la reprotection Hyper-V et la restauration automatique.

Lorsque vous reprotégez des machines virtuelles Azure en local, vous pouvez spécifier que vous souhaitez effectuer une restauration automatique vers l’emplacement d’origine ou vers un autre emplacement.

- **Récupération à l’emplacement d’origine** : Cette opération bascule à partir d’Azure vers la même machine locale source, si elle existe. Dans ce scénario, vous sélectionnez l’une des options de synchronisation décrites dans la procédure précédente.
- **Récupération à un autre emplacement** : Si la machine locale n’existe pas, vous pouvez effectuer une restauration automatique à partir d’Azure vers un autre emplacement. Lorsque vous reprotégez la machine virtuelle Azure en local, la machine locale est créée. Avec cette option, nous vous recommandons de sélectionner l’option de synchronisation des données avant le basculement
- [Passez en revue](hyper-v-azure-failback.md) les exigences et limitations relatives à la restauration de l’emplacement.


Après une restauration automatique sur le site local, vous activez **Réplication inverse** pour démarrer la réplication de la machine virtuelle vers Azure et terminer le cycle.




## <a name="next-steps"></a>Étapes suivantes
- Basculer des [machines virtuelles VMware spécifiques](vmware-azure-tutorial-failover-failback.md)
- Basculer des [machines virtuelles Hyper-V spécifiques](hyper-v-azure-failover-failback-tutorial.md).
- [Créer](site-recovery-create-recovery-plans.md) un plan de récupération.
- Basculer des [machines virtuelles dans un plan de récupération](site-recovery-failover.md).
- [Faire les préparations ](vmware-azure-failback.md) de la reprotection et de la restauration de VMware.
- Restaurer automatiquement des [machines virtuelles Hyper-V](hyper-v-azure-failback.md).