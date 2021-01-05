---
title: Configurer la récupération d’urgence pour SQL Server à l’aide d’Azure Site Recovery
description: Cet article explique comment configurer la récupération après sinistre pour SQL Server à l’aide de SQL Server et d’Azure Site Recovery.
services: site-recovery
author: sujayt
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 08/02/2019
ms.author: sutalasi
ms.openlocfilehash: 1b02b089fea7e883bdc6c58c7a2845af12b50a37
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011946"
---
# <a name="set-up-disaster-recovery-for-sql-server"></a>Configurer la récupération après sinistre pour SQL Server

Cet article explique comment protéger les SQL Server back end d’une application. Pour ce faire, vous utilisez une combinaison des technologies de continuité d’activité et récupération d’urgence de SQL Server et [Azure Site Recovery](site-recovery-overview.md).

Avant de commencer, assurez-vous que vous comprenez les fonctionnalités de récupération d’urgence de SQL Server. Ces fonctionnalités sont les suivantes :

* Clustering de basculement
* Groupes de disponibilité Always On
* Mise en miroir de bases de données
* Copie des journaux de transaction
* La géoréplication active
* Groupes de basculement automatique

## <a name="combining-bcdr-technologies-with-site-recovery"></a>Combinaison de technologies BCDR avec Site Recovery

Le choix d’une technologie BCDR pour la récupération des instances de SQL Server doit être basé sur votre objectif de temps de récupération (RTO) et sur l’objectif de point de récupération (RPO), comme décrit dans le tableau suivant. Combinez Site Recovery avec l’opération de basculement de la technologie choisie pour orchestrer la récupération de l’ensemble de votre application.

Type de déploiement | Technologie BCDR | RTO attendu pour SQL Server | RPO attendu pour SQL Server |
--- | --- | --- | ---
SQL Server sur une machine virtuelle (VM) infrastructure as a service (IaaS) Azure ou localement.| [Groupe de disponibilité AlwaysOn](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server?view=sql-server-2017) | Le temps nécessaire pour faire du réplica secondaire le réplica principal. | La réplication étant asynchrone vers le réplica secondaire, il y a une perte de données.
SQL Server sur une machine virtuelle Azure IaaS ou localement.| [Clustering de basculement (instance de cluster de basculement AlwaysOn)](/sql/sql-server/failover-clusters/windows/windows-server-failover-clustering-wsfc-with-sql-server?view=sql-server-2017) | Temps nécessaire pour basculer entre les nœuds. | Étant donné que Always On FCI utilise un stockage partagé, la même vue de l’instance de stockage est disponible lors du basculement.
SQL Server sur une machine virtuelle Azure IaaS ou localement.| [Mise en miroir de bases de données (mode hautes performances)](/sql/database-engine/database-mirroring/database-mirroring-sql-server?view=sql-server-2017) | Temps nécessaire pour forcer le service, qui utilise le serveur miroir en tant que serveur de secours actif. | La réplication est asynchrone. La base de données miroir peut être un peu en retard par rapport à la base de données principale. Le décalage est généralement faible. Mais il peut devenir important si le système du serveur principal ou du serveur miroir subit une charge importante.<br/><br/>La copie des journaux de transaction peut être un complément à la mise en miroir de bases de données. Il s’agit d’une alternative favorable à la mise en miroir asynchrone de bases de données.
SQL As Platform as a service (PaaS) sur Azure.<br/><br/>Ce type de déploiement comprend des bases de données uniques et des pools élastiques. | La géoréplication active | 30 secondes après le déclenchement du basculement.<br/><br/>Lorsque le basculement est activé pour l’une des bases de données secondaires, toutes les autres bases de données secondaires sont automatiquement liées à la nouvelle base de données primaire. | RPO de cinq secondes.<br/><br/>La géo-réplication active utilise la technologie Always On de SQL Server. Elle réplique de manière asynchrone les transactions validées entre une base de données primaire vers une base de données secondaire à l’aide de l’isolation d’instantané.<br/><br/>Il est garanti que les données secondaires n’auront jamais de transactions partielles.
SQL en tant que PaaS configuré avec la géoréplication active sur Azure.<br/><br/>Ce type de déploiement comprend des instances gérées, des pools élastiques et des bases de données uniques. | Groupes de basculement automatique | RTO d’une heure. | RPO de cinq secondes.<br/><br/>Les groupes de basculement automatique fournissent la sémantique de groupe en plus de la géo-réplication Active. Toutefois, le même mécanisme de réplication asynchrone est utilisé.
SQL Server sur une machine virtuelle Azure IaaS ou localement.| Réplication avec Azure Site Recovery | Le RTO est généralement inférieur à 15 minutes. Pour plus d’informations, consultez le [SLA RTO fourni par Site Recovery](https://azure.microsoft.com/support/legal/sla/site-recovery/v1_2/). | Une heure pour la cohérence des applications et cinq minutes pour la cohérence des incidents. Si vous souhaitez réduire le RPO, utilisez d’autres technologies BCDR.

> [!NOTE]
> Voici quelques considérations importantes à prendre en compte lorsque vous contribuez à protéger les charges de travail SQL avec Site Recovery :
> * Site Recovery est indépendant des applications. Site Recovery peut aider à protéger n’importe quelle version de SQL Server déployée sur un système d’exploitation pris en charge. Pour en savoir plus, consultez la [matrice d’assistance](vmware-physical-azure-support-matrix.md#replicated-machines) pour la récupération des machines répliquées.
> * Vous pouvez choisir d’utiliser Site Recovery pour n’importe quel déploiement d’une infrastructure Azure, Hyper-V, VMware ou physique. Suivez les instructions fournies à la fin de cet article sur la [façon de protéger un cluster SQL Server](#how-to-help-protect-a-sql-server-cluster) avec Site Recovery.
> * Assurez-vous que le taux de modification des données observé sur l'ordinateur se situe dans les [limites de Site Recovery](vmware-physical-azure-support-matrix.md#churn-limits). Le taux de modification est mesuré en octets écrits par seconde. Pour les machines exécutant Windows, vous pouvez afficher ce taux de modification en sélectionnant l’onglet **Performance** dans le gestionnaire des tâches. Observez la vitesse d’écriture pour chaque disque.
> * Site Recovery prend en charge la réplication des instances de cluster de basculement sur Storage Spaces Direct. Pour plus d’informations, consultez [Comment activer la réplication de Storage Spaces Direct](azure-to-azure-how-to-enable-replication-s2d-vms.md).
> 
> Lorsque vous migrez votre charge de travail SQL vers Azure, il est recommandé d’appliquer les [Recommandations de performances pour SQL Server sur les machines virtuelles Azure](../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md).

## <a name="disaster-recovery-of-an-application"></a>Récupération d’urgence d’une application

Site Recovery orchestre le test de basculement et le basculement de toute votre application à l’aide de plans de récupération.

Il existe certaines conditions préalables pour garantir que votre plan de récupération est entièrement personnalisé en fonction de vos besoins. Tout déploiement de SQL Server a généralement besoin d’un déploiement Active Directory. Il a également besoin d’une connectivité pour votre couche application.

### <a name="step-1-set-up-active-directory"></a>Étape 1 : Configurer Active Directory

Configurez Active Directory sur le site de récupération secondaire afin que SQL Server fonctionne correctement.

* **Petite entreprise** : Vous disposez d’un petit nombre d’applications et d’un seul contrôleur de domaine pour le site local. Si vous souhaitez basculer l’ensemble du site, utilisez la réplication Site Recovery. Ce service réplique le contrôleur de domaine dans le centre de données secondaire ou sur Azure.
* **Moyenne à grande entreprise** : Vous devrez peut-être configurer des contrôleurs de domaine supplémentaires.
  - Si vous avez un grand nombre d’applications, que vous disposez d’une forêt Active Directory et que vous souhaitez effectuer un basculement par application ou charge de travail, configurez un autre contrôleur de domaine dans le centre de données secondaire ou dans Azure.
  -  Si vous utilisez groupes de disponibilité Always On pour effectuer une récupération sur un site distant, configurez un autre contrôleur de domaine sur le site secondaire ou dans Azure. Ce contrôleur de domaine est utilisé pour l’instance de SQL Server récupérée.

Les instructions fournies dans cet article supposent qu’un contrôleur de domaine est disponible sur le site secondaire. Pour en savoir plus, consultez les procédures pour [aider à protéger Active Directory avec Site Recovery](site-recovery-active-directory.md).

### <a name="step-2-ensure-connectivity-with-other-tiers"></a>Étape 2 : Garantir la connectivité avec d’autres niveaux

Une fois que la couche base de données est en cours d’exécution dans la région Azure cible, assurez-vous que vous disposez d’une connectivité avec les niveaux application et Web. Prenez les mesures nécessaires à l’avance pour valider la connectivité avec le test de basculement.

Pour comprendre comment vous pouvez concevoir des applications pour les considérations de connectivité, consultez les exemples suivants :

* [Concevoir une application pour la reprise d’activité dans le cloud](../azure-sql/database/designing-cloud-solutions-for-disaster-recovery.md)
* [Stratégies de reprise d’activité pour les pools élastiques](../azure-sql/database/disaster-recovery-strategies-for-applications-with-elastic-pool.md)

### <a name="step-3-interoperate-with-always-on-active-geo-replication-and-auto-failover-groups"></a>Étape 3 : Interagir avec Always On, la géoréplication Active et les groupes de basculement automatique

Les technologies BCDR Always On, géoréplication active et groupes de basculement automatique ont des réplicas secondaires de SQL Server en cours d’exécution dans la région Azure cible. La première étape pour le basculement de votre application consiste à spécifier ce réplica comme principal. Cette étape suppose que vous disposiez déjà d’un contrôleur de domaine dans le réplica secondaire. Cette étape ne sera peut-être pas nécessaire si vous choisissez d’effectuer un basculement automatique. Basculez vos couches application et Web uniquement après la fin du basculement de la base de données.

> [!NOTE]
> Si vous avez aidé à protéger les machines SQL avec Site Recovery, il vous suffit de créer un groupe de récupération de ces machines et d’ajouter leur basculement dans le plan de récupération.

[Créez un plan de récupération](site-recovery-create-recovery-plans.md) avec les machines virtuelles des couches application et web. Les étapes suivantes montrent comment ajouter le basculement de la couche base de données:

1. Importez les scripts pour basculer le groupe de disponibilité SQL dans une [machine virtuelle Resource Manager](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/asr-automation-recovery/scripts/ASR-SQL-FailoverAG.ps1) et une [machine virtuelle classique](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/asr-automation-recovery/scripts/ASR-SQL-FailoverAGClassic.ps1). Importez les scripts sur votre compte Azure Automation.

    [![Image d’un logo « Déployer sur Azure »](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/c4803408-340e-49e3-9a1f-0ed3f689813d.png)](https://aka.ms/asr-automationrunbooks-deploy)

1. Ajoutez le script ASR-SQL-FailoverAG comme une action préalable du premier groupe dans le plan de récupération.

1. Suivez les instructions disponibles dans le script pour créer une variable Automation. Cette variable fournit le nom des groupes de disponibilité.

### <a name="step-4-conduct-a-test-failover"></a>Étape 4 : Effectuer un test de basculement

Certaines technologies BCDR, telles que SQL Always On, ne prennent pas en charge les tests de basculement de manière native. Nous vous recommandons l’approche suivante *uniquement lors de l'utilisation de ces technologies*.

1. Configurer la [Sauvegarde Azure](../backup/backup-azure-vms-first-look-arm.md) sur la machine virtuelle qui héberge le réplica du groupe de disponibilité dans Azure.

1. Avant de déclencher le test de basculement du plan de récupération, récupérez la machine virtuelle à partir de la sauvegarde effectuée à l’étape précédente.

    ![Capture d’écran montrant la fenêtre de restauration d’une configuration à partir d’Azure Backup](./media/site-recovery-sql/restore-from-backup.png)

1. [Forcez un quorum](/sql/sql-server/failover-clusters/windows/force-a-wsfc-cluster-to-start-without-a-quorum#PowerShellProcedure) dans la machine virtuelle restaurée à partir d’une sauvegarde.

1. Mettez à jour l’adresse IP de l’écouteur pour qu’elle soit une adresse disponible dans le réseau de test de basculement.

    ![Capture d’écran de la fenêtre règles et de la boîte de dialogue Propriétés de l’adresse IP](./media/site-recovery-sql/update-listener-ip.png)

1. Mettez l'écouteur en ligne.

    ![Capture d’écran de la fenêtre intitulée Content_AG montrant les noms et les états des serveurs](./media/site-recovery-sql/bring-listener-online.png)

1. Vérifiez que l’équilibreur de charge dans le réseau de basculement possède une adresse IP provenant du pool d’adresses IP front-end qui correspond à chaque écouteur de groupe de disponibilité et créée avec la machine virtuelle SQL Server du pool back-end.

     ![Capture d’écran de la fenêtre intitulée « SQL-AlwaysOn-LB – Frontend IP Pool »](./media/site-recovery-sql/create-load-balancer1.png)

    ![Capture d’écran de la fenêtre intitulée « SQL-AlwaysOn-LB – Backend IP Pool »](./media/site-recovery-sql/create-load-balancer2.png)

1. Dans les groupes de récupération ultérieurs, ajoutez le basculement de votre couche application, suivi de votre couche Web pour ce plan de récupération.

1. Effectuez un test de basculement du plan de récupération afin de tester le basculement de bout en bout de votre application.

## <a name="steps-to-do-a-failover"></a>Procédure de basculement

Après avoir ajouté le script à l’étape 3 et l’avoir validé à l’étape 4, vous pouvez effectuer un basculement du plan de récupération créé à l’étape 3.

Notez que les étapes de basculement pour les couches application et web doivent être les mêmes dans les plans de récupération de basculement et de test de basculement.

## <a name="how-to-help-protect-a-sql-server-cluster"></a>Comment protéger un cluster SQL Server

Pour un cluster exécutant SQL Server Standard Edition ou SQL Server 2008 R2, nous vous recommandons d’utiliser la réplication Site Recovery pour protéger SQL Server.

### <a name="azure-to-azure-and-on-premises-to-azure"></a>Azure vers Azure et local vers Azure

Site Recovery ne prend pas en charge le cluster invité lors de la réplication vers une région Azure. De même, l’édition standard de SQL Server n’inclut aucune solution de récupération d'urgence à faible coût. Dans ce scénario, nous vous recommandons de protéger le cluster SQL Server sur un serveur SQL Server autonome à l’emplacement principal, et de le récupérer à l’emplacement secondaire.

1. Configurez une instance SQL Server autonome supplémentaire dans la région Azure principale ou sur le site local.

1. Configurez l’instance comme une copie miroir des bases de données que vous souhaitez protéger. Configurez la mise en miroir en mode haute sécurité.

1. Configurez Site Recovery sur le site principal pour [Azure](azure-to-azure-tutorial-enable-replication.md), [Hyper-V](./hyper-v-azure-tutorial.md) ou [serveurs physiques/machines virtuelles VMware](./vmware-azure-tutorial.md).

1. Utilisez la réplication Site Recovery pour répliquer la nouvelle instance SQL Server sur le site secondaire. Comme il s’agit d’une copie miroir de haute sécurité, elle est synchronisée avec le cluster principal, mais répliquée à l’aide de la réplication Site Recovery.

   ![Image d’un cluster standard montrant la relation et le flux entre un site principal, Site Recovery et Azure](./media/site-recovery-sql/standalone-cluster-local.png)

### <a name="failback-considerations"></a>Considérations en matière de restauration automatique

Pour les clusters SQL Server Standard, la restauration automatique après un basculement non planifié nécessite une sauvegarde et une restauration SQL Server. Cette opération est effectuée à partir de l’instance miroir vers le cluster d’origine, avec le rétablissement de la mise en miroir.

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="how-does-sql-server-get-licensed-when-used-with-site-recovery"></a>Comment les SQL Server sont-ils concédés sous licence lorsqu’ils sont utilisés avec Site Recovery ?

La réplication Site Recovery pour les SQL Server est couverte par l’avantage de la récupération d’urgence de Software Assurance. Cette couverture s’applique à tous les scénarios de Site Recovery : récupération d’urgence d’un site local vers Azure et récupération d’urgence Azure IaaS inter-régions. Pour en savoir plus, consultez [la tarification Azure Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery/).

### <a name="will-site-recovery-support-my-sql-server-version"></a>Site Recovery prendra-t-il en charge ma version de SQL ?

Site Recovery est indépendant des applications. Site Recovery peut aider à protéger n’importe quelle version de SQL Server déployée sur un système d’exploitation pris en charge. Pour en savoir plus, consultez la [matrice d’assistance pour la récupération](vmware-physical-azure-support-matrix.md#replicated-machines) des machines répliquées.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur l’[architecture de Site Recovery](./azure-to-azure-architecture.md).
* Pour les serveurs SQL dans Azure, apprenez-en davantage sur [solutions de haute disponibilité](../azure-sql/virtual-machines/windows/business-continuity-high-availability-disaster-recovery-hadr-overview.md#azure-only-high-availability-solutions) pour la récupération dans la région Azure secondaire.
* Pour SQL Database, apprenez-en davantage sur les options de [continuité d’activité](../azure-sql/database/business-continuity-high-availability-disaster-recover-hadr-overview.md) et de [haute disponibilité](../azure-sql/database/high-availability-sla.md) pour la récupération dans la région Azure secondaire.
* Pour les ordinateurs SQL server locaux, [apprenez-en davantage](../azure-sql/virtual-machines/windows/business-continuity-high-availability-disaster-recovery-hadr-overview.md#hybrid-it-disaster-recovery-solutions) sur les options de haute disponibilité pour la récupération dans Machines virtuelles Azure.