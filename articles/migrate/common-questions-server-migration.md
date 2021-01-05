---
title: Questions courantes sur la migration de serveur Azure Migrate
description: Obtenez des réponses aux questions courantes sur l’utilisation d’Azure Migrate Server Migration pour migrer des machines.
author: anvar-ms
ms.author: anvar
ms.manager: bsiva
ms.topic: conceptual
ms.date: 08/28/2020
ms.openlocfilehash: 1f32e9788b2255be373a4db8c159ef2a6b6ccf04
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753839"
---
# <a name="azure-migrate-server-migration-common-questions"></a>Migration de serveur Azure Migrate : Questions courantes

Cet article répond à des questions courantes sur la migration de serveur Azure Migrate d’Azure Migrate. Si vous avez d’autres questions, consultez les ressources suivantes :

- [Questions générales](resources-faq.md) sur Azure Migrate.
- Questions sur l’[appliance Azure Migrate](common-questions-appliance.md)
- Questions sur la [découverte, l’évaluation et la visualisation des dépendances](common-questions-discovery-assessment.md)
- Obtenez des réponses à vos questions sur le [forum Azure Migrate](https://aka.ms/AzureMigrateForum)

## <a name="does-azure-migrate-convert-uefi-based-machines-to-bios-based-machines-and-migrate-them-to-azure-as-azure-generation-1-vms"></a>Azure Migrate convertit-il les machines UEFI en machines BIOS et les migre-t-il vers Azure en tant que machines virtuelles de 1ère génération Azure ?
Azure Migrate : L’outil Server Migration migre toutes les machines UEFI vers Azure en tant que machines virtuelles de 2e génération Azure. Nous ne prenons plus en charge la conversion de machines virtuelles UEFI en machines virtuelles BIOS. Notez que toutes les machines BIOS sont migrées vers Azure en tant que machines virtuelles de 1ère génération Azure uniquement.

## <a name="how-can-i-migrate-uefi-based-machines-to-azure-as-azure-generation-1-vms"></a>Comment migrer des machines UEFI vers Azure en tant que machines virtuelles de 1ère génération Azure ?
Azure Migrate : L’outil Server Migration migre les machines UEFI vers Azure en tant que machines virtuelles de 2e génération Azure. Si vous souhaitez les migrer vers des machines virtuelles de 1ère génération Azure, convertissez le type de démarrage en BIOS avant de commencer la réplication, puis utilisez l’outil Azure Migrate : Server Migration pour effectuer la migration vers Azure.
 
## <a name="which-operating-systems-are-supported-for-migration-of-uefi-based-machines-to-azure"></a>Quels sont les systèmes d’exploitation pris en charge pour la migration de machines UEFI vers Azure ?

| **Systèmes d’exploitation pris en charge pour les machines UEFI** | **VMware sans agent vers Azure**                                                                                                             | **Hyper-V sans agent vers Azure** | **VMware avec agent, supports physiques et autres clouds vers Azure** |
| ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------ | ---------------------------------------------------------- |
| Windows Server 2019, 2016, 2012 R2, 201                 | O                                                                                                                                         | Y                              | O                                                          |
| Windows 10 Professionnel, Windows 10 Entreprise                   | O                                                                                                                                         | Y                              | O                                                          |
| SUSE Linux Enterprise Server 15 SP1                     | O                                                                                                                                         | Y                              | O                                                          |
| SUSE Linux Enterprise Server 12 SP4                     | O                                                                                                                                         | Y                              | O                                                          |
| Ubuntu Server 16.04, 18.04, 19.04, 19.10                | O                                                                                                                                         | Y                              | O                                                          |
| RHEL 8.1, 8.0, 7.8, 7.7, 7.6, 7.5, 7.4, 7.0, 6.x        | O<br>                 _RHEL 8.x nécessite une [préparation manuelle](./prepare-for-migration.md#linux-machines)_   | O                              | O                                                          |
| CentOS 8.1, 8.0, 7.7, 7.6, 7.5, 7.4, 6.x               | O<br>_CentOS 8.x nécessite une [préparation manuelle](./prepare-for-migration.md#linux-machines)_ | O                              | O                                                          |
| Oracle Linux 7.7, 7.7-CI                                |  O                                                                                                                                        | Y                              | O                                                          |

## <a name="can-i-use-the-recovery-services-vault-created-by-azure-migrate-for-disaster-recovery-scenarios"></a>Est-ce que je peux utiliser le coffre Recovery Services créé par Azure Migrate pour les scénarios de reprise d’activité ?
Nous vous déconseillons d’utiliser le coffre Recovery Services créé par Azure Migrate pour les scénarios de reprise d’activité. Cela peut entraîner des échecs de démarrage de la réplication dans Azure Migrate. 

## <a name="where-should-i-install-the-replication-appliance-for-agent-based-migrations"></a>Où dois-je installer l’appliance de réplication pour les migrations basées sur des agents ?

L’appliance de réplication doit être installée sur un ordinateur dédié. L’appliance de réplication ne doit pas être installée sur une machine source que vous souhaitez répliquer, ou sur l’appliance de découverte et d’évaluation Azure Migrate que vous avez peut-être déjà installée. Suivez notre [tutoriel](./tutorial-migrate-physical-virtual-machines.md) pour plus d’informations.

## <a name="how-can-i-migrate-my-aws-ec2-instances-to-azure"></a>Comment puis-je migrer mes instances AWS EC2 vers Azure ?

Consultez cet [article](./tutorial-migrate-aws-virtual-machines.md) pour découvrir, évaluer et migrer vos instances AWS EC2 vers Azure.

## <a name="can-i-migrate-aws-vms-running-amazon-linux-operating-system"></a>Puis-je migrer des machines virtuelles AWS exécutant le système d’exploitation Amazon Linux ?

Il n’est pas possible de migrer les machines virtuelles qui exécutent Amazon Linux, car ce système d’exploitation n’est pris en charge que sur AWS.
Pour migrer des charges de travail s’exécutant sur Amazon Linux, vous pouvez créer une machine virtuelle CentOS/RHEL dans Azure et migrer la charge de travail s’exécutant sur la machine Linux AWS suivant une approche appropriée de migration des charges de travail. Pour certaines charges de travail, il peut exister des outils propres à la charge de travail qui facilitent la migration, par exemple pour les bases de données ou les outils de déploiement dans le cas de serveurs web.

## <a name="what-geographies-are-supported-for-migration-with-azure-migrate"></a>Quelles sont les zones géographiques prises en charge pour la migration avec Azure Migrate ?

Passez en revue les zones géographiques prises en charge pour les clouds [publics](migrate-support-matrix.md#supported-geographies-public-cloud) et du [secteur public](migrate-support-matrix.md#supported-geographies-azure-government).

## <a name="can-we-use-the-same-azure-migrate-project-to-migrate-to-multiple-regions"></a>Puis-je utiliser le même projet Azure Migrate pour migrer vers plusieurs régions ?

Bien que vous puissiez créer des évaluations pour plusieurs régions dans un projet Azure Migrate, un projet Azure Migrate peut uniquement être utilisé pour migrer des serveurs vers une seule région Azure. Vous pouvez créer des projets Azure Migrate supplémentaires pour chaque région vers laquelle vous devez migrer.

- Pour les migrations VMware sans agent, la région cible est verrouillée une fois que vous activez la première réplication.
- Pour les migrations basées sur des agents (VMware, serveurs physiques et serveurs d’autres clouds), la région cible est verrouillée lorsque l’utilisateur clique sur le bouton « Créer des ressources » sur le portail lors de la configuration de l’appliance de réplication.
- Pour les migrations Hyper-V sans agent, la région cible est verrouillée lorsque l’utilisateur clique sur le bouton « Créer des ressources » sur le portai lors de la configuration du fournisseur de réplication Hyper-V.

## <a name="can-we-use-the-same-azure-migrate-project-to-migrate-to-multiple-subscriptions"></a>Puis-je utiliser le même projet Azure Migrate pour migrer vers plusieurs abonnements ? 

Oui, vous pouvez migrer vers plusieurs abonnements dans la même région cible pour un seul projet Azure Migrate. Vous pouvez sélectionner l’abonnement cible lors de l’activation de la réplication pour une machine ou un ensemble de machines. La région cible est verrouillée après la première réplication pour les migrations VMware sans agent et pendant l’installation de l’appliance de réplication et du fournisseur Hyper-V pour les migrations basées sur des agents et les migrations Hyper-V sans agent, respectivement.

## <a name="what-are-the-migration-options-in-azure-migrate-server-migration"></a>Quelles sont les options de migration dans Azure Migrate : Migration de serveur ?

Azure Migrate : Migration de serveur est un outil qui fournit deux options permettant d’effectuer des migrations de vos serveurs/machines virtuelles sources vers Azure : la migration sans agent et la migration basée sur un agent.

Quelle que soit l’option de migration choisie, la première étape pour migrer un serveur à l’aide d’Azure Migrate : Migration de serveur consiste à activer la réplication pour le serveur. Cela effectue une réplication initiale des données de votre machine virtuelle/serveur vers Azure. Une fois la réplication initiale terminée, une réplication continue (sync-diff continue) est établie pour migrer des données incrémentielles vers Azure. Une fois que l’opération atteint le stade sync-diff, vous pouvez choisir de migrer vers Azure à tout moment.  

Voici quelques considérations à prendre en compte lorsque vous décidez de l’option de migration.

**Les migrations sans agent** ne nécessitent aucun déploiement de logiciels (agents) sur les machines virtuelles/serveurs à migrer. L’option sans agent orchestre la réplication en s’intégrant aux fonctionnalités fournies par le fournisseur de virtualisation.
Les options de réplication sans agent sont disponibles pour les [machines virtuelles VMware](./tutorial-migrate-vmware.md) et les [machines virtuelles Hyper-V](./tutorial-migrate-hyper-v.md).

**Les migrations basées sur des agents** requièrent l’installation du logiciel Azure Migrate (agents) sur les machines virtuelles/serveurs sources à migrer. L’option basée sur des agents ne repose pas sur la plateforme de virtualisation pour la fonctionnalité de réplication et peut donc être utilisée avec n’importe quel serveur exécutant une architecture x86/x64 et une version de système d’exploitation prise en charge par la méthode de réplication basée sur des agents.

L’option de migration basée sur des agents peut être utilisée pour [les machines virtuelles VMware](./tutorial-migrate-vmware-agent.md), [les machines virtuelles Hyper-V](./tutorial-migrate-physical-virtual-machines.md), [les serveurs physiques](./tutorial-migrate-physical-virtual-machines.md), [les machines virtuelles fonctionnant sous AWS](./tutorial-migrate-aws-virtual-machines.md), les machines virtuelles fonctionnant sous GCP ou les machines virtuelles fonctionnant sous un autre fournisseur de virtualisation. La migration basée sur des agents traite vos machines comme des serveurs physiques pour les besoins de la migration.

Bien que la migration sans agent offre plus de commodité et de simplicité sur les options de réplication basée sur des agents pour les scénarios pris en charge (VMware et Hyper-V), vous pouvez envisager d’utiliser le scénario basé sur un agent pour les cas d’utilisation suivants :

- Environnement contraint par l’IOPS : La réplication sans agent utilise des instantanés et consomme des IOPS/bande passante de stockage. Nous vous recommandons d’utiliser la méthode de migration basée sur un agent en présence de contraintes de stockage/IOPS dans votre environnement.
- Si vous n’avez pas de vCenter Server, vous pouvez traiter vos machines virtuelles VMware comme des serveurs physiques et utiliser le workflow de migration basé sur l’agent.

Pour plus d’informations, consultez cet [article](./server-migrate-overview.md) pour comparer les options de migration des migrations VMware.

## <a name="how-does-agentless-migration-work"></a>Comment fonctionne la migration sans agent ?

Azure Migrate : Migration de serveur fournit des options de réplication sans agent pour la migration des machines virtuelles VMware et des machines virtuelles Hyper-V fonctionnant sous Windows ou Linux. L’outil fournit également une option de réplication supplémentaire basée sur des agents pour les serveurs Windows et Linux qui peut être utilisée pour migrer des serveurs physiques, ainsi que des machines virtuelles x86/x64 sur VMware, Hyper-V, AWS, GCP, etc. L’option de réplication basée sur des agents nécessite l’installation d’un logiciel agent sur le serveur/la machine virtuelle en cours de migration, tandis que dans l’option sans agent, aucun logiciel n’a besoin d’être installé sur les machines virtuelles elles-mêmes, ce qui offre davantage de commodité et de simplicité par rapport à l’option de réplication basée sur des agents.

L’option de réplication sans agent fonctionne à l’aide de mécanismes fournis par le fournisseur de virtualisation (VMware, Hyper-V). Dans le cas des machines virtuelles VMware, le mécanisme de réplication sans agent utilise des instantanés VMware et la technologie Changed Block Tracking de VMware pour répliquer des données à partir de disques de machines virtuelles. Ce mécanisme est similaire à celui utilisé par de nombreux produits de sauvegarde. Dans le cas des machines virtuelles Hyper-V, le mécanisme de réplication sans agent utilise des instantanés de machine virtuelle et la capacité de suivi des modifications du réplica Hyper-V pour répliquer des données à partir de disques de machines virtuelles.

Lorsque la réplication est configurée pour une machine virtuelle, elle passe tout d’abord par une phase de réplication initiale. Pendant la réplication initiale, un instantané de machine virtuelle est pris et une copie complète des données des disques d’instantanés est répliquée sur les disques managés dans votre abonnement. Une fois la réplication initiale de la machine virtuelle terminée, le processus de réplication passe à une phase de réplication incrémentielle (réplication différentielle). Dans la phase de réplication incrémentielle, les modifications de données qui se sont produites depuis le dernier cycle de réplication sont périodiquement répliquées et appliquées aux disques managés du réplica, ce qui permet de synchroniser la réplication avec les modifications apportées à la machine virtuelle. Dans le cas des machines virtuelles VMware, la technologie Changed Block Tracking de VMware est utilisée pour effectuer le suivi des modifications entre les cycles de réplication. Au début du cycle de réplication, un instantané de machine virtuelle est pris et Changed Block Tracking est utilisée pour obtenir les modifications entre l’instantané actuel et le dernier instantané répliqué avec succès. Ainsi, seules les données qui ont été modifiées depuis le dernier cycle de réplication doivent être répliquées pour maintenir la réplication de la machine virtuelle synchronisée. À la fin de chaque cycle de réplication, l’instantané est publié, et la consolidation de l’instantané est effectuée pour la machine virtuelle. De même, dans le cas des machines virtuelles Hyper-V, le moteur de suivi des modifications du réplica Hyper-V est utilisé pour effectuer le suivi des modifications entre les cycles de réplication consécutifs.
Lorsque vous effectuez l’opération de migration sur une machine virtuelle de réplication, vous avez la possibilité d’arrêter la machine virtuelle locale et d’effectuer une réplication incrémentielle finale pour garantir l’absence de perte de données. Lors de l’exécution de l’option de migration, les disques managés de réplica correspondant à la machine virtuelle sont utilisés pour créer la machine virtuelle dans Azure.

Pour commencer, reportez-vous aux tutoriels [Migration VMware sans agent](./tutorial-migrate-vmware.md) et [Migration Hyper-V sans agent](./tutorial-migrate-hyper-v.md).

## <a name="how-does-agent-based-migration-work"></a>Comment fonctionne la migration sans agent ?

En plus des options de migration sans agent pour les machines virtuelles VMware et Hyper-V, l’outil Migration de serveur fournit une option de migration basée sur des agents pour migrer les serveurs Windows et Linux exécutés sur des serveurs physiques ou fonctionnant en tant que machines virtuelles x86/x64 sur VMware, Hyper-V, AWS, Google Cloud Platform, etc.

La méthode de migration basée sur des agents utilise le logiciel de l’agent installé sur le serveur à migrer pour répliquer les données du serveur sur Azure. Le processus de réplication utilise une architecture de déchargement dans laquelle l’agent relaie les données de réplication vers un serveur de réplication dédié appelé appliance de réplication ou serveur de configuration (ou vers un serveur de traitement en Scale-out). [En savoir plus](./agent-based-migration-architecture.md) sur le fonctionnement de l’option de migration basée sur des agents. 

Remarque : L’appliance de réplication est différente de l’appliance de détection Azure Migrate et doit être installée sur une machine distincte/dédiée.

## <a name="how-do-i-gauge-the-bandwidth-requirement-for-my-migrations"></a>Comment évaluer la bande passante requise pour mes migrations ?

La bande passante requise pour la réplication des données sur Azure dépend d’une série de facteurs et de la vitesse à laquelle l’appliance Azure Migrate locale peut lire et répliquer les données sur Azure. La réplication comporte deux phases : la réplication initiale et la réplication différentielle.

Lorsque la réplication commence pour une machine virtuelle, un premier cycle de réplication se produit, dans lequel des copies complètes des disques sont répliquées. Une fois la réplication initiale terminée, des cycles de réplication incrémentielle (cycles différentiels) sont planifiés régulièrement pour transférer les modifications qui se sont produites depuis le cycle de réplication précédent.

### <a name="agentless-vmware-vm-migration"></a>Migration de machines virtuelles VMware sans agent

Vous pouvez déterminer les besoins en bande passante en fonction du volume de données à déplacer au niveau de la vague et de la durée d’exécution de la réplication initiale que vous souhaitez : idéalement, la réplication initiale doit être terminée au moins 3 à 4 jours avant la fenêtre de migration réelle pour vous donner suffisamment de temps pour effectuer une migration test avant la fenêtre réelle et pour réduire au minimum les temps d’arrêt pendant la fenêtre.

Vous pouvez estimer la bande passante ou le temps nécessaire pour la migration de machines virtuelles VMware sans agent à l’aide de la formule suivante :

Durée d’exécution de la réplication initiale = {taille des disques (ou taille utilisée si elle est disponible) * 0,7 (en supposant une compression moyenne de 30 % – estimation prudente)}/bande passante disponible pour la réplication.

### <a name="agent-based-vmware-vm-migration"></a>Migration de machines virtuelles VMware basée sur des agents

Pour une méthode de réplication basée sur un agent, le planificateur de déploiement peut aider à profiler l’environnement en fonction de l’évolution des données et à prédire les besoins nécessaires en bande passante. Pour plus d’informations, consultez cet [article](./agent-based-migration-architecture.md#plan-vmware-deployment). 

## <a name="how-do-i-throttle-replication-in-using-azure-migrate-appliance-for-agentless-vmware-replication"></a>Comment faire pour limiter la réplication lors de l’utilisation de l’appliance Azure Migrate pour une réplication VMware sans agent ?  

Vous pouvez limiter l’utilisation de NetQosPolicy. Par exemple :

L’AppNamePrefix à utiliser dans NetQosPolicy est « GatewayWindowsService.exe ». Vous pouvez créer une stratégie sur l’appliance Azure Migrate pour limiter le trafic de réplication de l’appliance en créant une stratégie telle que celle-ci :

New-NetQosPolicy -Name "ThrottleReplication" -AppPathNameMatchCondition "GatewayWindowsService.exe" -ThrottleRateActionBitsPerSecond 1 MB

## <a name="how-is-the-data-transmitted-from-on-prem-environment-to-azure-is-it-encrypted-before-transmission"></a>Comment les données sont-elles transmises de l’environnement local à Azure ? Sont-elles chiffrées avant la transmission ?

Dans le cas de la réplication sans agent, l’appliance Azure Migrate compresse les données et les chiffre avant le chargement. Les données sont transmises via un canal de communication sécurisé sur HTTPS et utilisent TLS 1.2 ou une version ultérieure. De plus, le Stockage Azure chiffre automatiquement vos données quand elles sont conservées dans le cloud (chiffrement au repos).  

## <a name="how-does-churn-rate-affect-agentless-replication"></a>Dans quelle mesure le taux de variation affecte-t-il la réplication sans agent ?

La réplication sans agent opérant un repli des données, le *modèle de variation* est plus important que le *taux de variation*. Quand un fichier est écrit à plusieurs reprises, le taux n’a pas un impact important. Toutefois, un modèle dans lequel un secteur sur deux est écrit entraîne une variation élevée lors du cycle suivant. Comme nous réduisons au minimum la quantité de données que nous transférons, nous autorisons la réduction des données autant que possible avant de planifier le cycle suivant.

## <a name="how-frequently-is-a-replication-cycle-scheduled"></a>À quelle fréquence un cycle de réplication est-il planifié ?

La formule pour planifier le prochain cycle de réplication est (durée du cycle précédente / 2) ou une heure (la valeur la plus élevée s’applique).

Par exemple, si une machine virtuelle prend quatre heures pour un cycle delta, le cycle suivant est planifié en deux heures, et non dans l’heure suivante. Le processus diffère immédiatement après la réplication initiale, quand le premier cycle delta est planifié immédiatement.

## <a name="how-do-i-migrate-windows-server-2003-running-on-vmwarehyper-v-to-azure"></a>Comment migrer Windows Server 2003 s’exécutant sous VMware/Hyper-V vers Azure ?

[Le support étendu de Windows Server 2003](/troubleshoot/azure/virtual-machines/run-win-server-2003#microsoft-windows-server-2003-end-of-support) a pris fin le 14 juillet 2015.  L’équipe du support Azure continue de vous aider à résoudre les problèmes qui concernent l’exécution de Windows Server 2003 sur Azure. Toutefois, ce support est limité aux problèmes qui ne nécessitent pas de dépannage ou de patchs au niveau du système d’exploitation.
La migration de vos applications vers des instances Azure exécutant une version plus récente de Windows Server est l’approche recommandée pour vous assurer de tirer parti de la flexibilité et de la fiabilité du cloud Azure.

Toutefois, si vous choisissez de migrer votre instance Windows Server 2003 vers Azure, vous pouvez utiliser l’outil Azure Migrate : Migration de serveur si votre serveur Windows est une machine virtuelle fonctionnant sous VMware ou Hyper-V. Consultez cet article pour [préparer vos machines Windows Server 2003 pour la migration](./prepare-windows-server-2003-migration.md).

## <a name="what-is-the-difference-between-the-test-migration-and-migrate-operations"></a>Quelle est la différence entre les opérations de migration et de migration de test ?

La migration de test permet de tester et de valider des migrations avant la migration proprement dite. La migration de test fonctionne en vous permettant de créer des copies de test des machines virtuelles répliquées dans un environnement bac à sable dans Azure. L’environnement bac à sable est délimité par un réseau virtuel de test que vous spécifiez. L’opération de migration de test ne perturbe pas le bon fonctionnement des machines, car les applications continuent de s’exécuter à la source tout en vous permettant d’effectuer des tests sur une copie clonée dans un environnement bac à sable isolé. Vous pouvez effectuer autant de tests que nécessaire pour valider la migration, effectuer des tests d’applications et résoudre les éventuels problèmes avant la migration proprement dite.

## <a name="will-windows-server-2008-and-2008-r2-be-supported-in-azure-after-migration"></a>Windows Server 2008 et 2008 R2 seront-ils pris en charge dans Azure après la migration ?

Vous pouvez migrer vos serveurs Windows Server 2008 et 2008 R2 locaux vers des machines virtuelles Azure et obtenir des correctifs de sécurité étendus pendant trois ans après la fin du support, sans frais supplémentaires au-delà du coût d’exécution de la machine virtuelle. Vous pouvez utiliser l’outil Azure Migrate : Migration de serveur pour migrer vos charges de travail Windows Server 2008 et 2008 R2.

## <a name="is-there-a-rollback-option-for-azure-migrate"></a>Existe-t-il une option de restauration pour Azure Migrate ?

Vous pouvez utiliser l’option Migration de test pour valider les fonctionnalités et les performances de votre application dans Azure. Vous pouvez effectuer un nombre quelconque de migrations de test et exécuter la migration finale après avoir établi la confiance dans le cadre de l’opération de migration de test. Une migration de test n’a pas d’impact sur la machine locale, qui reste opérationnelle et continue la réplication jusqu’à ce que vous effectuiez la migration réelle. Si des erreurs se sont produites pendant l’UAT de migration de test, vous pouvez choisir de reporter la migration finale et de laisser votre machine virtuelle/serveur source fonctionner et se répliquer sur Azure. Vous pouvez retenter la migration finale une fois que vous avez résolu les erreurs.  
Remarque : Une fois que vous avez effectué une migration finale vers Azure et que la machine source locale a été arrêtée, vous ne pouvez pas effectuer de restauration d’Azure vers votre environnement local.

## <a name="can-i-select-the-virtual-network-and-subnet-to-use-for-test-migrations"></a>Puis-je sélectionner le réseau virtuel et le sous-réseau à utiliser pour les migrations de test ?

Vous pouvez sélectionner un réseau virtuel pour les migrations de test. Le sous-réseau est automatiquement sélectionné selon la logique suivante :

- Si un sous-réseau cible (autre que celui par défaut) a été spécifié en entrée lors de l’activation de la réplication, alors Azure Migrate donne la priorité à l’utilisation d’un sous-réseau portant le même nom dans le réseau virtuel sélectionné pour la migration de test.
- Si le sous-réseau portant le même nom est introuvable, Azure Migrate sélectionne le premier sous-réseau disponible dans l’ordre alphabétique qui n’est pas un sous-réseau de type passerelle/Application Gateway/Pare-feu/Bastion.

## <a name="why-is-the-test-migration-button-disabled-for-my-server"></a>Pourquoi le bouton Tester la migration est-il désactivé pour mon serveur ?

Le bouton Tester la migration peut être désactivé dans les scénarios suivants :

- Vous ne pouvez pas commencer une migration de test tant qu’une réplication initiale n’a pas été effectuée pour la machine virtuelle. Le bouton Tester la migration sera désactivé jusqu’à ce que le processus de réplication initiale soit terminé. Vous pouvez effectuer un test de migration une fois que votre machine virtuelle est dans une phase sync-diff.
- Le bouton peut être désactivé si une migration de test a déjà été effectuée, mais qu’un nettoyage de la migration de test n’a pas été effectué pour cette machine virtuelle. Effectuez un nettoyage de la migration de test et retentez l’opération.

## <a name="what-happens-if-i-dont-clean-up-my-test-migration"></a>Que se passe-t-il si je ne nettoie pas ma migration de test ?

La migration de test simule la migration réelle en créant une machine virtuelle Azure de test à l’aide de données répliquées. Le serveur sera déployé avec une copie ponctuelle des données répliquées dans le groupe de ressources cible (sélectionné pendant l’activation de la réplication) avec un suffixe « -test ». Les migrations de test sont destinées à valider les fonctionnalités du serveur afin de minimiser les problèmes post-migration. Si la migration de test n’est pas nettoyée après le test, la machine virtuelle de test continuera à fonctionner dans Azure et entraînera des frais. Pour nettoyer après une migration de test, accédez à la vue des machines répliquées dans l’outil Migration de serveur, puis utilisez l’action « Nettoyer la migration de test » sur la machine.

## <a name="can-i-migrate-active-directory-domain-controllers-using-azure-migrate"></a>Puis-je migrer des contrôleurs de domaine Active Directory à l’aide d’Azure Migrate ?

L’outil Migration de serveur est indépendant des applications et fonctionne pour la plupart d’entre elles. Lorsque vous migrez un serveur à l’aide de l’outil Migration de serveur, toutes les applications installées sur le serveur sont migrées avec lui. Toutefois, d’autres méthodes de migration, différente de la migration de serveur, peuvent être mieux adaptées à la migration de certaines applications.  Pour Active Directory, dans le cas d’environnements hybrides où le site local est connecté à votre environnement Azure, vous pouvez étendre votre répertoire dans Azure en ajoutant des contrôleurs de domaine supplémentaires dans Azure et en configurant la réplication d’Active Directory. Si vous effectuez une migration vers un environnement isolé dans Azure nécessitant ses propres contrôleurs de domaine (ou si vous testez des applications dans un environnement bac à sable), vous pouvez migrer des serveurs à l’aide de l’outil de migration de serveur.

## <a name="what-happens-if-i-dont-stop-replication-after-migration"></a>Que se passe-t-il si je n’arrête pas la réplication après la migration ?

Lorsque vous arrêtez la réplication, l’outil Azure Migrate : Migration de serveur nettoie les disques managés de l’abonnement qui ont été créés pour la réplication. Si vous n’arrêtez pas la réplication après une migration, vous continuez à payer des frais pour ces disques. L’arrêt de la réplication n’aura pas d’impact sur les disques attachés aux machines qui ont déjà été migrées.

## <a name="do-i-need-vmware-vcenter-to-migrate-vmware-vms"></a>Ai-je besoin de VMware vCenter pour migrer des machines virtuelles VMware ?

Pour [migrer des machines virtuelles VMware](server-migrate-overview.md) à l’aide d’une migration sans agent ou basée sur un agent VMware, les hôtes ESXi sur lesquels résident les machines virtuelles doivent être managés par vCenter Server. Si vous n’avez pas vCenter Server, vous pouvez migrer des machines virtuelles VMware en les migrant en tant que serveurs physiques. [Plus d’informations](migrate-support-matrix-physical-migration.md)

## <a name="can-i-upgrade-my-os-while-migrating"></a>Puis-je mettre à niveau mon système d’exploitation pendant la migration ?

Azure Migrate : Migration de serveur prend uniquement en charge les migrations comparables pour le moment. L’outil ne prend pas en charge la mise à niveau de la version du système d’exploitation pendant la migration. La machine migrée aura le même système d’exploitation que la machine source.

## <a name="i-deployed-two-or-more-appliances-to-discover-vms-in-my-vcenter-server-however-when-i-try-to-migrate-the-vms-i-only-see-vms-corresponding-to-one-of-the-appliances"></a>J’ai déployé deux appliances (ou plus) pour découvrir des machines virtuelles dans vCenter Server. Cependant, lorsque j’essaie de migrer les machines virtuelles, je ne vois que celles qui correspondent à l’une des appliances.

Si plusieurs appliances sont configurées, il est indispensable qu’il n’y ait pas de chevauchement entre les machines virtuelles sur les comptes vCenter fournis. Une découverte avec un tel chevauchement n’est pas un scénario pris en charge.

## <a name="how-do-i-know-if-my-vm-was-successfully-migrated"></a>Comment puis-je savoir si la migration de ma machine virtuelle s’est correctement effectuée ?

Une fois que vous avez correctement migré votre machine virtuelle/serveur, vous pouvez afficher et gérer la machine virtuelle à partir de la page Machines virtuelles. Connectez-vous à la machine virtuelle migrée pour le vérifier.
Vous pouvez également examiner l’état de la tâche pour que l’opération vérifie si la migration a abouti. Si vous constatez des erreurs, résolvez-les, puis recommencez l’opération de migration.

## <a name="can-i-consolidate-multiple-source-vms-into-one-vm-while-migrating"></a>Puis-je consolider plusieurs machines virtuelles sources en une seule machine virtuelle pendant la migration ?

Les capacités de migration de serveur d’Azure Migrate prennent actuellement en charge les migrations comparables. Nous ne prenons pas en charge la consolidation des serveurs ni la mise à niveau du système d’exploitation dans le cadre de la migration. 

## <a name="how-does-agentless-replication-affect-vmware-servers"></a>Dans quelle mesure la réplication sans agent affecte les serveurs VMware ?

La réplication sans agent a un impact sur les performances du serveur VMware vCenter et des hôtes VMware ESXi. La réplication sans agent utilisant des instantanés, elle consomme des IOPS en matière de stockage. Une bande passante de stockage IOPS est donc requise. Nous vous déconseillons d’utiliser la réplication sans agent si vous avez des contraintes de stockage ou d’IOPS dans votre environnement.


## <a name="next-steps"></a>Étapes suivantes

Consultez la [vue d’ensemble d’Azure Migrate](migrate-services-overview.md).