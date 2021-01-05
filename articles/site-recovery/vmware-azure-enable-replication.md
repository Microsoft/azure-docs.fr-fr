---
title: Activer des machines virtuelles VMware pour la reprise d’activité après sinistre en utilisant Azure Site Recovery
description: Cet article explique comment activer la réplication des machines virtuelles VMware pour effectuer une reprise d’activité après sinistre à l’aide du service Azure Site Recovery.
author: Rajeswari-Mamilla
ms.service: site-recovery
ms.date: 04/01/2020
ms.topic: conceptual
ms.author: ramamill
ms.openlocfilehash: 74870d10348421bf726b9bdc58504a74cf4105a9
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96004209"
---
# <a name="enable-replication-to-azure-for-vmware-vms"></a>Activer la réplication des machines virtuelles VMware dans Azure

Cet article explique comment activer la réplication de machines virtuelles VMware locales sur Azure.

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que votre système remplit les critères suivants :

- [configuré votre environnement source local](vmware-azure-set-up-source.md) ;
- [configuré votre environnement cible dans Azure](vmware-azure-set-up-target.md).
- [Vérifiez la configuration requise et les conditions préalables](vmware-physical-azure-support-matrix.md) avant de commencer. Voici quelques points importants à prendre en considération :
  - [Systèmes d’exploitation pris en charge](vmware-physical-azure-support-matrix.md#replicated-machines) pour les machines répliquées.
  - Prise en charge [du stockage/des disques](vmware-physical-azure-support-matrix.md#storage).
  - [Exigences Azure](vmware-physical-azure-support-matrix.md#azure-vm-requirements) auxquelles les machines locales doivent satisfaire.

### <a name="resolve-common-issues"></a>Résoudre les problèmes courants

- Chaque disque doit être inférieur à 4 To.
- Le disque du système d’exploitation doit être un disque de base, non un disque dynamique.
- Pour les machines virtuelles de génération 2 compatibles UEFI, le système d’exploitation doit être de la famille Windows, et la taille du disque de démarrage doit être inférieure à 300 Go.

## <a name="before-you-start"></a>Avant de commencer

Lorsque vous répliquez des machines virtuelles VMware, gardez les informations ci-après à l’esprit :

- Votre compte d’utilisateur Azure doit disposer de certaines [autorisations](site-recovery-role-based-linked-access-control.md#permissions-required-to-enable-replication-for-new-virtual-machines) pour activer la réplication d’une machine virtuelle dans Azure.
- Les machines virtuelles VMware sont découvertes toutes les 15 minutes, et il peut s’écouler 15 minutes ou plus entre la découverte des machines virtuelles et leur affichage dans le portail Azure. Lorsque vous ajoutez un serveur vCenter ou un hôte vSphere, la découverte peut prendre jusqu’à 15 minutes ou plus.
- La répercussion des modifications apportées à l’environnement sur la machine virtuelle dans le portail peut prendre 15 minutes ou plus. C’est le cas, par exemple, de l’installation des outils VMware.
- Vous pouvez vérifier l’heure de dernière découverte des machines virtuelles VMware en consultant le champ **Dernier contact à** sur la page **Serveurs de configuration** du serveur vCenter ou de l’hôte vSphere.
- Pour ajouter des machines virtuelles à répliquer sans attendre la découverte planifiée, mettez en surbrillance le serveur de configuration (sans cliquer dessus), puis sélectionnez **Actualiser**.
- Lorsque vous activez la réplication, si la machine virtuelle est prête, le serveur de processus installe automatiquement sur celle-ci le service Mobilité Azure Site Recovery.

## <a name="enable-replication"></a>Activer la réplication

Avant de suivre la procédure décrite dans cette section, lisez les informations ci-après :

- Désormais, Azure Site Recovery effectue la réplication directement sur les disques managés pour toutes les nouvelles réplications. Le serveur de processus écrit les journaux de réplication sur un compte de stockage de cache dans la région cible. Ces journaux sont utilisés pour la création de points de récupération dans les disques managés de réplica qui suivent la convention d’affectation de noms `asrseeddisk`.
- La prise en charge de PowerShell pour la réplication sur des disques managés est disponible depuis la [ version Az.0RecoveryServices du module 2.0.0](https://www.powershellgallery.com/packages/Az.RecoveryServices/2.0.0-preview)
- Lors du basculement, le point de récupération que vous sélectionnez sert à créer le disque managé cible.
- Les machines virtuelles précédemment configurées pour la réplication sur les comptes de stockage cibles ne sont pas affectées.
- La réplication d’une nouvelle machine virtuelle sur des comptes de stockage n’est disponible que par le biais de l’API REST Representational State Transfer et de PowerShell. Utilisez l’API REST Azure version 2016-08-10 ou 2018-01-10 pour la réplication sur des comptes de stockage.

Pour activer la réplication, procédez comme suit :

1. Accédez à **l’étape 2 : Répliquer l’application** > **Source**. Après avoir activé la réplication pour la première fois, sélectionnez **+Répliquer** dans le coffre afin d’activer la réplication de machines virtuelles supplémentaires.
1. Dans la page **Source** > **Source**, sélectionnez le serveur de configuration.
1. Pour la zone **Type de machine**, sélectionnez **Machines virtuelles** ou **Machines physiques**.
1. Dans la zone **Hyperviseur vCenter/vSphere**, sélectionnez le serveur vCenter qui gère l’hôte vSphere, ou sélectionnez l’hôte. Ce paramètre n’est pas utile si vous répliquez des ordinateurs physiques.
1. Sélectionnez le serveur de processus. Si aucun autre serveur de processus n’est créé, le serveur de processus intégré du serveur de configuration sera disponible dans le menu déroulant. L’état d’intégrité de chaque serveur de processus est indiqué en fonction des limites et d’autres paramètres recommandés. Sélectionnez un serveur de processus sain. Vous ne pouvez pas choisir un serveur de processus [critique](vmware-physical-azure-monitor-process-server.md#process-server-alerts). Vous pouvez [détecter et résoudre](vmware-physical-azure-troubleshoot-process-server.md) les erreurs **ou** configurer un [serveur de processus de montée en puissance](vmware-azure-set-up-process-server-scale.md).

   :::image type="content" source="./media/vmware-azure-enable-replication/ps-selection.png" alt-text="Fenêtre d’activation de la source de réplication":::

   > [!NOTE]
   > Depuis la [version 9.24](site-recovery-whats-new.md), des alertes supplémentaires sont introduites pour améliorer les alertes d’intégrité du serveur de processus. Pour que toutes les alertes puissent être générées, mettez à niveau les composants Site Recovery vers la version 9.24 ou des versions ultérieures.

1. Pour la zone **Cible**, sélectionnez l’abonnement et le groupe de ressources dans lesquels vous voulez créer les machines virtuelles basculées. Choisissez le modèle de déploiement à utiliser dans Azure pour les machines virtuelles basculées.
1. Sélectionnez le réseau et le sous-réseau Azure auxquels les machines virtuelles Azure se connecteront après le basculement. Ce réseau doit se trouver dans la même région que le coffre du service Site Recovery.

   Sélectionnez **Effectuez maintenant la configuration pour les machines sélectionnées** pour appliquer le paramètre réseau à l’ensemble des machines virtuelles que vous sélectionnez à des fins de protection. Sélectionnez **Configurer ultérieurement** pour sélectionner le réseau Azure pour chaque machine virtuelle. Si vous n'avez pas de réseau, vous devez en créer un. Pour créer un réseau à l’aide d’Azure Resource Manager, sélectionnez **Créer**. Le cas échéant, sélectionnez un sous-réseau, puis sélectionnez **OK**.

   :::image type="content" source="./media/vmware-azure-enable-replication/enable-rep3.png" alt-text="Fenêtre cible pour l’activation de la réplication":::

1. Pour la zone **Machines virtuelles** > **Sélectionner les machines virtuelles**, sélectionnez chaque machine virtuelle à répliquer. Vous pouvez uniquement sélectionner les machines virtuelles pour lesquelles la réplication peut être activée. Sélectionnez ensuite **OK**. Si vous ne pouvez pas voir ou sélectionner une machine virtuelle spécifique, consultez l’article [Ordinateur source non répertorié sur le portail Azure](vmware-azure-troubleshoot-replication.md#step-3-troubleshoot-source-machines-that-arent-available-for-replication) pour résoudre le problème.

   :::image type="content" source="./media/vmware-azure-enable-replication/enable-replication5.png" alt-text="Fenêtre de sélection des machines virtuelles pour l’activation de la réplication":::

1. Pour la zone **Propriétés** > **Configurer les propriétés**, sélectionnez le compte utilisé par le serveur de processus pour installer automatiquement le service Mobilité Site Recovery sur la machine virtuelle. Choisissez également le type de disque managé a utiliser pour la réplication en fonction de vos modèles d’activité des données.
1. Par défaut, tous les disques d’une machine virtuelle source sont répliqués. Pour exclure des disques de la réplication, cochez la case **Inclure** des disques que vous ne souhaitez pas répliquer. Sélectionnez ensuite **OK**. Vous pouvez opter pour une définition ultérieure des propriétés. [Apprenez-en davantage](vmware-azure-exclude-disk.md) sur l’exclusion de disques.

   :::image type="content" source="./media/vmware-azure-enable-replication/enable-replication6.png" alt-text="Fenêtre de configuration des propriétés pour l’activation de la réplication":::

1. Dans la zone **Paramètres de réplication** > **Configurer les paramètres de réplication**, vérifiez que la stratégie de réplication sélectionnée est correcte. Vous pouvez modifier les paramètres de la stratégie de réplication dans **Paramètres** > **Stratégies de réplication** > _nom de la stratégie_ > **Modifier les paramètres**. Les modifications appliquées à une stratégie s’appliquent également à la réplication et aux nouvelles machines virtuelles.
1. Si vous souhaitez regrouper les machines virtuelles dans un groupe de réplication, activez l’option **Cohérence multimachine virtuelle**. Spécifiez un nom pour le groupe, puis sélectionnez **OK**.

   > [!NOTE]
   > - Toutes les machines virtuelles d’un groupe de réplication sont répliquées ensemble et partagent des points de récupération de cohérence en cas d’incident et de cohérence des applications lorsqu’elles basculent.
   > - Rassemblez les machines virtuelles et les serveurs physiques afin qu’ils reflètent vos charges de travail. L’activation de la cohérence multimachine virtuelle peut affecter les performances de la charge de travail. N’effectuez cette opération que si les machines virtuelles exécutent la même charge de travail, et que vous avez besoin de cohérence.

   :::image type="content" source="./media/vmware-azure-enable-replication/enable-replication7.png" alt-text="Fenêtre d’activation de la réplication":::

1. Sélectionnez **Activer la réplication**. Vous pouvez suivre la progression du travail **Activer la protection** en sélectionnant **Paramètres** > **Travaux** > **Travaux Site Recovery**. Lorsque la tâche **Finaliser la protection** s’exécute, la machine virtuelle est prête à être basculée.

## <a name="view-and-manage-vm-properties"></a>Afficher et gérer les propriétés des machines virtuelles

Ensuite, vérifiez les propriétés de la machine virtuelle source. N’oubliez pas que le nom de la machine virtuelle Azure doit respecter la [configuration requise pour les machines virtuelles Azure](vmware-physical-azure-support-matrix.md#replicated-machines).

1. Accédez à **Paramètres** > **Éléments répliqués**, puis sélectionnez la machine virtuelle. La page **Bases** affiche des informations sur les paramètres et l’état de la machine virtuelle.
1. Dans **Propriétés**, vous pouvez afficher les informations sur la réplication et le basculement de la machine virtuelle.
1. Dans **Calcul et réseau** > **Propriétés de calcul**, vous pouvez modifier plusieurs propriétés de machine virtuelle.

   :::image type="content" source="./media/vmware-azure-enable-replication/vmproperties.png" alt-text="Fenêtre de propriétés Calcul et réseau":::

   - **Nom de machine virtuelle Azure** : si besoin, modifiez ce nom afin de respecter les exigences d’Azure.
   - **Taille ou type de machine virtuelle cible** : la taille de machine virtuelle par défaut est choisie en fonction de paramètres, tels que le nombre de disques, le nombre de cartes réseau, le nombre de cœurs du processeur, la mémoire et les tailles de rôle de machine virtuelle disponibles dans la région Azure cible. Azure Site Recovery sélectionne la première taille de machine virtuelle disponible réunissant tous les critères. Si besoin, vous pouvez à tout moment sélectionner une autre taille de machine virtuelle avant le basculement. La taille du disque de la machine virtuelle dépend également de la taille du disque source et n’est modifiable qu’après le basculement. Pour plus d’informations sur les tailles de disque et les taux d’IOPS, consultez l’article [Cibles de scalabilité et de performance des disques de machine virtuelle sur Windows](../virtual-machines/windows/disk-scalability-targets.md).
   - **Groupe de ressources** : vous pouvez sélectionner un [groupe de ressources](../azure-resource-manager/management/overview.md#resource-groups) à partir duquel une machine virtuelle fait partie d’un post-basculement. Vous pouvez changer ce paramètre à tout moment avant le basculement. Après le basculement, si vous effectuez la migration de la machine virtuelle vers un autre groupe de ressources, les paramètres de protection de cette machine ne sont plus actifs.
   - **Groupe à haute disponibilité** : vous pouvez sélectionner un [groupe à haute disponibilité](../virtual-machines/windows/tutorial-availability-sets.md) si votre machine virtuelle doit faire partie d’un post-basculement. Lorsque vous sélectionnez un groupe à haute disponibilité, gardez à l’esprit les informations suivantes :
     - Seuls les groupes à haute disponibilité qui appartiennent au groupe de ressources spécifié sont répertoriés.
     - Les machines virtuelles situées sur différents réseaux virtuels ne peuvent pas faire partie du même groupe à haute disponibilité.
     - Seules les machines virtuelles de même taille peuvent faire partie d’un groupe à haute disponibilité.

1. Vous pouvez également ajouter des informations sur le réseau cible, le sous-réseau et l’adresse IP attribuée à la machine virtuelle Azure.
1. Les disques de données et du système d’exploitation de la machine virtuelle qui seront répliqués s’affichent dans **Disques** .

### <a name="configure-networks-and-ip-addresses"></a>Configurer les réseaux et les adresses IP

Vous pouvez définir l’adresse IP cible :

- Si vous ne fournissez pas d’adresse IP, la machine ayant basculé utilise le protocole DHCP.
- Si vous définissez une adresse qui n’est pas disponible au moment du basculement, ce dernier échoue.
- Si l’adresse est disponible sur le réseau de test de basculement, vous pouvez utiliser la même adresse IP cible pour le test de basculement.

Le nombre de cartes réseau est déterminé par la taille que vous spécifiez pour la machine virtuelle cible, comme suit :

- Si le nombre de cartes réseau sur la machine virtuelle source est inférieur ou égal au nombre de cartes autorisé pour la taille de la machine virtuelle cible, la cible présente le même nombre de cartes que la source.
- Si le nombre de cartes de la machine virtuelle source dépasse celui autorisé pour la taille de la machine virtuelle cible, le nombre maximal de cartes accepté par la taille cible est utilisé. Par exemple, si une machine virtuelle source présente deux cartes réseau et que la taille de la machine virtuelle cible en accepte quatre, la machine virtuelle cible comporte deux cartes. Si la machine virtuelle source présente deux cartes, mais que la taille cible n’en prend en charge qu’une seule, la machine virtuelle cible ne comporte qu’une seule carte.
- Si la machine virtuelle possède plusieurs cartes réseau, elles se connectent toutes au même réseau. En outre, la première carte qui s’affiche dans la liste devient la carte réseau par défaut dans la machine virtuelle Azure.

### <a name="azure-hybrid-benefit"></a>Azure Hybrid Benefit

Les clients Microsoft Software Assurance peuvent utiliser Azure Hybrid Benefit afin de réduire les coûts de licences pour les ordinateurs Windows Server qui font l’objet d’une migration vers Azure. Cet avantage s’applique également à la récupération d’urgence Azure. Si vous y avez droit, vous pouvez attribuer cet avantage à la machine virtuelle créée par Site Recovery en cas de basculement.

1. Accédez à la section des **propriétés de calcul et réseau** de la machine virtuelle répliquée.
1. Indiquez si vous disposez d’une licence Windows Server vous permettant de bénéficier d’Azure Hybrid Benefit.
1. Confirmez que vous possédez une licence Windows Server éligible avec Software Assurance vous permettant d’appliquer cet avantage à la machine virtuelle qui sera créée lors du basculement.
1. Enregistrez les paramètres de la machine virtuelle répliquée.

[En savoir plus](https://azure.microsoft.com/pricing/hybrid-benefit/) sur Azure Hybrid Benefit.

## <a name="next-steps"></a>Étapes suivantes

Une fois que la machine virtuelle se trouve à l’état protégé, essayez d’effectuer un [basculement](site-recovery-failover.md) pour vérifier si votre application apparaît dans Azure.

- [En savoir plus](site-recovery-manage-registration-and-protection.md) sur le nettoyage des paramètres d’inscription et de protection pour désactiver la réplication.
- [En savoir plus](vmware-azure-disaster-recovery-powershell.md) sur l’automatisation de la réplication pour vos machines virtuelles à l’aide de PowerShell.
