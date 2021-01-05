---
title: Mettre à l’échelle la récupération d'urgence de machines virtuelles physiques et VMware avec Azure Site Recovery
description: Découvrez comment configurer la récupération d’urgence sur Azure de grands nombres de machines virtuelles VMware ou de serveurs physiques locaux avec Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/14/2019
ms.author: raynew
ms.openlocfilehash: 101e42263e46c5a21f26b0fa9cdeed798525fee9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89047076"
---
# <a name="set-up-disaster-recovery-at-scale-for-vmware-vmsphysical-servers"></a>Configurer la récupération d’urgence à grande échelle de machines virtuelles VMware et de serveurs physiques

Cet article explique comment configurer la récupération d’urgence sur Azure de grands nombres (> 1 000) de machines virtuelles VMware locales ou de serveurs physiques locaux dans votre environnement de production à l’aide du service [Azure Site Recovery](site-recovery-overview.md).


## <a name="define-your-bcdr-strategy"></a>Définir votre stratégie de continuité d’activité et reprise d’activité

Dans le cadre de votre stratégie de continuité d’activité et reprise d’activité (BCDR), vous définissez des objectifs de point de récupération (RPO) et des objectifs de délai de récupération (RTO) pour vos applications métier et charges de travail. Un objectif de délai de récupération mesure la période et le niveau de service dans lesquels une application métier ou un processus doivent être restaurés et disponibles afin d’éviter les problèmes de continuité d’activité.
- Site Recovery assure la réplication continue des machines virtuelles VMware et des serveurs physiques et implémente un [Contrat de niveau de service](https://azure.microsoft.com/support/legal/sla/site-recovery/v1_2/) pour l’objectif de délai de récupération.
- Lorsque vous planifiez une récupération d’urgence à grande échelle pour des machines virtuelles VMware et déterminez les ressources Azure dont vous avez besoin, vous pouvez spécifier une valeur RTO qui sera utilisée pour les calculs de capacité.


## <a name="best-practices"></a>Meilleures pratiques

Meilleures pratiques générales pour la récupération d’urgence à grande échelle. Ces meilleures pratiques sont décrites plus en détail dans les sections suivantes du document.

- **Identifier la configuration requise de la cible** : estimez les besoins en capacité et ressources d’Azure avant de configurer la récupération d’urgence.
- **Planifier les composants Site Recovery** : déterminez les composants de Site Recovery (serveur de configuration, serveurs de processus) dont vous avez besoin pour atteindre la capacité estimée.
- **Configurer un ou plusieurs serveurs de processus scale-out** : n’utilisez pas le serveur de processus qui s’exécute par défaut sur le serveur de configuration. 
- **Obtenir les dernières mises à jour** : l’équipe Site Recovery publie régulièrement de nouvelles versions des composants Site Recovery, et vous devez vous assurer que vous exécutez des versions les plus récentes. À cette fin, suivez les [Nouveautés](site-recovery-whats-new.md) afin d’être informé des mises à jour, et [activez et installez les mises à jour](service-updates-how-to.md) au fur et à mesure de leur publication.
- **Surveiller de manière proactive** : une fois la récupération d’urgence opérationnelle, vous devez surveiller de manière proactive l’état et l’intégrité des machines répliquées et des ressources d’infrastructure.
- **Exercices de récupération d’urgence** : vous devez exécuter régulièrement des exercices de récupération d’urgence. Ceux-ci n’ont aucune incidence sur votre environnement de production, mais vous aident à vous assurer que le basculement vers Azure fonctionnera comme prévu le cas échéant.



## <a name="gather-capacity-planning-information"></a>Collecter les informations pour la planification de la capacité

Collectez des informations sur votre environnement local afin d’évaluer les besoins en capacité de la cible (Azure).
- Pour les machines virtuelles VMware, exécutez le Planificateur de déploiement.
- Pour les serveurs physiques, collectez les informations manuellement.

### <a name="run-the-deployment-planner-for-vmware-vms"></a>Exécuter les Planificateur de déploiement pour les serveurs physiques

Le Planificateur de déploiement vous aide à recueillir des informations sur votre environnement local VMware.

- Exécutez le Planificateur de déploiement pendant une période correspondant à un cycle typique pour vos machines virtuelles. Cela permet de générer des estimations et recommandations plus précises.
- Nous vous recommandons d’exécuter le Planificateur de déploiement sur le serveur de configuration afin de calculer le débit de celui-ci. [Apprenez-en davantage](site-recovery-vmware-deployment-planner-run.md#get-throughput) sur la mesure du débit.
- Si vous n’avez pas encore de serveur de configuration configuré :
    - [Consultez la présentation](vmware-physical-azure-config-process-server-overview.md) des composants Site Recovery.
    - [Configurez un serveur de configuration ](vmware-azure-deploy-configuration-server.md) pour pouvoir exécuter le Planificateur de déploiement sur celui-ci.

Ensuite, exécutez le Planificateur comme suit :

1. [Découvrez](site-recovery-deployment-planner.md) le Planificateur de déploiement. Vous pouvez télécharger la dernière version à partir du portail ou [directement ici](https://aka.ms/asr-deployment-planner).
2. Examinez les [conditions préalables](site-recovery-deployment-planner.md#prerequisites) et les [dernières mises à jour](site-recovery-deployment-planner-history.md) pour le Planificateur de déploiement, puis [ téléchargez et extrayez ](site-recovery-deployment-planner.md#download-and-extract-the-deployment-planner-tool) l’outil.
3. [Exécutez le Planificateur de déploiement](site-recovery-vmware-deployment-planner-run.md) sur le serveur de configuration.
4. [Générez un rapport](site-recovery-vmware-deployment-planner-run.md#generate-report) pour synthétiser des estimations et recommandations.
5. Analysez les [recommandations du rapport](site-recovery-vmware-deployment-planner-analyze-report.md) et les [estimations de coûts](site-recovery-vmware-deployment-planner-cost-estimation.md).

>[!NOTE]
> Par défaut, l’outil est configuré pour profiler jusqu’à 1 000 machines virtuelles et générer un rapport sur celles-ci. Vous pouvez modifier cette limite en augmentant la valeur de la clé MaxVMsSupported dans le fichier ASRDeploymentPlanner.exe.config.

## <a name="plan-target-azure-requirements-and-capacity"></a>Planifier la configuration requise et la capacité de la cible (Azure)

À l’aide des estimations et recommandations rassemblées, vous pouvez planifier les ressources et la capacité de la cible. Si vous avez exécuté le Planificateur de déploiement pour des machines virtuelles VMware, vous pouvez vous aider d’un certain nombre de [recommandations de rapport ](site-recovery-vmware-deployment-planner-analyze-report.md#recommendations).

- **Machines virtuelles compatibles** : ce nombre permet de déterminer combien de machines virtuelles sont prêtes pour la récupération d’urgence sur Azure. Les recommandations en matière de bande passante réseau et de cœurs Azure sont basées sur ce nombre.
- **Bande passante réseau requise** : notez la bande passante dont vous avez besoin pour la réplication delta de machines virtuelles compatibles. 
    - Lorsque vous exécutez le planificateur, vous spécifiez l’objectif de point de récupération souhaité exprimé en minutes. Les recommandations vous indiquent la bande passante nécessaire pour respecter cet objectif de point de récupération de 100 % et 90 % du temps. 
    - Les recommandations en matière de bande passante réseau prennent en compte la bande passante nécessaire pour le nombre total de serveurs de configuration et de serveurs de processus recommandé par le Planificateur.
- **Cœurs Azure nécessaires** : notez le nombre de cœurs dont vous avez besoin dans la région Azure cible, en fonction du nombre de machines virtuelles compatibles. Si vous ne disposez pas de suffisamment de cœurs, lors du basculement, Site Recovery ne pourra pas créer les machines virtuelles Azure requises.
- **Taille de lot de machines virtuelles recommandée** : la taille de lot recommandée est basée sur la capacité à terminer la réplication initiale pour le lot dans les 72 heures par défaut, tout en conservant un objectif de point de récupération de 100 %. La valeur d’heures peut être modifiée.

Vous pouvez utiliser ces recommandations pour planifier les ressources Azure, la bande passante réseau et les lots de machines virtuelles.

## <a name="plan-azure-subscriptions-and-quotas"></a>Planifier les abonnements et quotas Azure

Nous tenons à nous assurer que les quotas disponibles dans l’abonnement cible sont suffisants pour gérer le basculement.

**Tâche** | **Détails** | **Action**
--- | --- | ---
**Vérifier les cœurs** | Si les cœurs que compte le quota disponible ne sont pas en nombre égal ou supérieur au nombre total cible au moment du basculement, les basculements échoueront. | Pour les machines virtuelles VMware, vérifiez que l’abonnement cible contient suffisamment de cœurs pour respecter la recommandation du Planificateur de déploiement en matière de cœurs.<br/><br/> Pour les serveurs physiques, vérifiez que les cœurs Azure répondent à vos estimations manuelles.<br/><br/> Pour vérifier les quotas, dans le portail Azure > **Abonnement**, cliquez sur **Utilisation + quotas**.<br/><br/> [Apprenez-en davantage](../azure-portal/supportability/resource-manager-core-quotas-request.md) sur l’augmentation des quotas.
**Vérifier les limites de nombre de basculements** | Le nombre de basculements ne doit pas dépasser les limites de nombre de basculements de Site Recovery. |  Si le nombre de basculements dépasse les limites, vous pouvez ajouter des abonnements pour basculer vers plusieurs abonnements ou augmenter le quota pour un abonnement. 


### <a name="failover-limits"></a>Limites de nombre de basculements

Les limites indiquent le nombre de basculements que Site Recovery prend en charge en une heure, en supposant que chaque machine dispose de trois disques.

Que signifie « conforme » ? Pour démarrer une machine virtuelle Azure, Azure requiert que certains pilotes soient à l’état de démarrage, et que des services tels que DHCP soient configurés pour démarrer automatiquement.
- Sur les machines « conformes », ces paramètres sont déjà configurés.
- Pour les ordinateurs exécutant Windows, vous pouvez en vérifier la conformité de façon proactive et les rendre conformes si nécessaire. [Plus d’informations](site-recovery-failover-to-azure-troubleshoot.md#failover-failed-with-error-id-170010)
- Les machines Linux ne sont mises en conformité qu’au moment du basculement.

**La machine est-elle conforme pour Azure ?** | **Limites de machine virtuelle Azure (basculement de disque managé)**
--- | --- 
Oui | 2000
Non | 1 000

- Les limites supposent qu’un minimum d’autres travaux sont en cours dans la région cible pour l’abonnement.
- Certaines régions Azure de petite taille peuvent avoir des limites légèrement inférieures.

## <a name="plan-infrastructure-and-vm-connectivity"></a>Planifier l’infrastructure et la connectivité de machine virtuelle

Après le basculement vers Azure, vous avez besoin que vos charges de travail fonctionnent de la même manière que localement, et que vos utilisateurs puissent accéder aux charges de travail exécutées sur les machines virtuelles Azure.

- [Apprenez-en davantage](site-recovery-active-directory.md#test-failover-considerations) sur le basculement de votre infrastructure locale Active Directory ou DNS vers Azure.
- [Apprenez-en davantage](site-recovery-test-failover-to-azure.md#prepare-to-connect-to-azure-vms-after-failover) sur la préparation de la connexion aux machines virtuelles Azure après le basculement.



## <a name="plan-for-source-capacity-and-requirements"></a>Planifier la capacité et la configuration requise de la source

Il est important que vous disposiez d’un nombre suffisant de serveurs de configuration et de serveurs de processus scale-out pour répondre aux besoins en capacité. Lorsque vous décidez d’opérer votre déploiement à grande échelle, commencez avec un seul serveur de configuration et un seul serveur de processus scale-out. À mesure que vous atteignez les limites prescrites, ajoutez des serveurs.

>[!NOTE]
> Pour les machines virtuelles VMware, le Planificateur de déploiement formule des recommandations sur la configuration et les serveurs de processus dont vous avez besoin. Nous vous recommandons de vous reporter aux tables incluses dans les procédures suivantes au lieu de suivre les recommandations du Planificateur de déploiement. 


## <a name="set-up-a-configuration-server"></a>Configurer un serveur de configuration
 
La capacité du serveur de configuration dépend du nombre de machines à répliquer, non du taux de variation des données. Pour déterminer si vous avez besoin de serveurs de configuration supplémentaires, utilisez ces limites de machine virtuelle définies.

**UC** | **Mémoire** | **Disque cache** | **Limite de machine répliquée**
 --- | --- | --- | ---
8 processeurs virtuels<br> 2 sockets * 4 cœurs à 2,5 GHz | 16 Go | 600 Go | Jusqu’à 550 machines<br> En supposant que chaque machine source dispose de trois disques de 100 Go chacun.

- Ces limites sont basées sur un serveur de configuration configuré à l’aide d’un modèle OVF.
- Les limites sont basées sur l’hypothèse que n’utilisez pas le serveur de processus qui s’exécute par défaut sur le serveur de configuration.

Si vous avez besoin d’ajouter un serveur de configuration, procédez comme suit :

- [Configurez un serveur de configuration ](vmware-azure-deploy-configuration-server.md) pour la récupération d’urgence de machines virtuelles VMware en utilisant un modèle OVF.
- [Configurez un serveur de configuration](physical-azure-set-up-source.md) manuellement pour les serveurs physiques ou les déploiements VMware qui ne peuvent pas utiliser de modèle OVF.

Lorsque vous configurez un serveur de configuration, tenez compte de ce qui suit :

- Lorsque vous configurez un serveur de configuration, il est important de prendre en compte l’abonnement et le coffre dans lesquels il réside, car ceux-ci ne doivent pas être modifiés après la configuration. Si vous n’avez pas besoin de modifier le coffre, vous devez en dissocier le serveur de configuration, puis le réinscrire. Cela a pour effet d’arrêter la réplication des machines virtuelles dans le coffre.
- Si vous souhaitez configurer un serveur de configuration avec plusieurs cartes réseau, vous devez le faire pendant la configuration. Vous ne pouvez pas le faire après l’inscription du serveur de configuration dans le coffre.

## <a name="set-up-a-process-server"></a>Configurer un serveur de processus

La capacité du serveur de processus dépend des taux de variation des données, non du nombre de machines activées pour la réplication.

- Pour les déploiements de grande envergure, vous devez toujours avoir au moins un serveur de processus scale-out.
- Pour déterminer si vous avez besoin de serveurs supplémentaires, reportez-vous au tableau suivant.
- Nous vous recommandons d’ajouter un serveur répondant à la spécification la plus élevée. 


**UC** | **Mémoire** | **Disque cache** | **Taux de variation**
 --- | --- | --- | --- 
12 processeurs virtuels<br> 2 sockets * 6 cœurs à 2,5 GHz | 24 Go | 1 Go | Jusqu’à 2 To par jour

Configurez le serveur de processus comme suit :

1. Examinez les [conditions préalables](vmware-azure-set-up-process-server-scale.md#prerequisites).
2. Installez le serveur via le [portail](vmware-azure-set-up-process-server-scale.md#install-from-the-ui) ou à partir de la [ligne de commande](vmware-azure-set-up-process-server-scale.md#install-from-the-command-line).
3. Configurez des machines répliquées pour utiliser le nouveau serveur. Si vous avez déjà des machines qui répliquent :
    - Vous pouvez [déplacer](vmware-azure-manage-process-server.md#switch-an-entire-workload-to-another-process-server) toute la charge de travail d’un serveur de processus vers le nouveau serveur de processus.
    - Vous pouvez également [déplacer](vmware-azure-manage-process-server.md#move-vms-to-balance-the-process-server-load) des machines virtuelles spécifiques vers le nouveau serveur de processus.



## <a name="enable-large-scale-replication"></a>Activer la réplication à grande échelle

Après avoir planifié la capacité et déployé les composants et l’infrastructure requis, activez la réplication pour de grands nombres de machines virtuelles.

1. Triez les machines en lots. Vous activez la réplication pour les machines virtuelles d’un lot, puis passez au lot suivant.

    - Pour les machines virtuelles VMware, vous pouvez utiliser la [taille de lot de machines virtuelles recommandée](site-recovery-vmware-deployment-planner-analyze-report.md#recommended-vm-batch-size-for-initial-replication) dans le rapport du Planificateur de déploiement.
    - Pour les machines physiques, nous vous recommandons de constituer des lots basés sur l’homogénéité de la taille et de la quantité de données des machines, ainsi que sur le débit réseau disponible. L’objectif est d’obtenir des lots de machines susceptibles d’accomplir leur réplication initiale à peu près dans le même temps.
    
2. Si la variation de disque pour une machine est élevée ou dépasse les limites préconisées par le Planificateur de déploiement, vous pouvez écarter de la machine des fichiers non critiques que vous n’avez pas besoin de répliquer (tels que des vidages de journaux ou des fichiers temporaires). Pour les machines virtuelles VMware, vous pouvez déplacer ces fichiers vers un disque séparé, puis [exclure ce disque](vmware-azure-exclude-disk.md) de la réplication.
3. Avant d’activer la réplication, vérifiez que les machines répondent aux [exigences de réplication](vmware-physical-azure-support-matrix.md#replicated-machines).
4. Configurez une stratégie de réplication pour des [machines virtuelles VMware](vmware-azure-set-up-replication.md#create-a-policy) ou des [serveurs physiques](physical-azure-disaster-recovery.md#create-a-replication-policy).
5. Activez la réplication pour des [machines virtuelles VMware](vmware-azure-enable-replication.md) ou des [serveurs physiques](physical-azure-disaster-recovery.md#enable-replication). Cela a pour effet de lancer la réplication initiale pour les machines sélectionnées.

## <a name="monitor-your-deployment"></a>Surveiller votre déploiement

Une fois la réplication lancée pour le premier lot de machines virtuelles, démarrez la surveillance de votre déploiement comme suit :  

1. Affectez un administrateur de récupération d’urgence pour surveiller l’état d’intégrité des machines répliquées.
2. [Surveillez les événements](site-recovery-monitor-and-troubleshoot.md) pour les éléments et l’infrastructure répliqués.
3. [Surveillez l’intégrité](vmware-physical-azure-monitor-process-server.md) de vos serveurs de processus scale-out.
4. Inscrivez-vous pour recevoir des [notifications par e-mail](./site-recovery-monitor-and-troubleshoot.md#subscribe-to-email-notifications) concernant les événements, afin de faciliter la surveillance.
5. Exécutez des [exercices de récupération d’urgence](site-recovery-test-failover-to-azure.md) pour vous assurer que tout fonctionne comme prévu.


## <a name="plan-for-large-scale-failovers"></a>Planifier des basculements à grande échelle

En cas d’urgence, il se peut que vous deviez basculer un grand nombre de machines/charges de travail vers Azure. Préparez-vous à ce type d’événement comme suit.

Vous pouvez vous préparer au basculement comme suit :

- [Préparez votre infrastructure et vos machines virtuelles](#plan-infrastructure-and-vm-connectivity) de telle sorte que vos charges de travail soient disponibles après le basculement et que les utilisateurs puissent accéder aux machines virtuelles Azure.
- Vérifiez les [limites de nombre de basculements](#failover-limits) plus haut dans ce document. Assurez-vous que vos basculements s’inscrivent dans ces limites.
- Effectuez régulièrement des [exercices de récupération d’urgence](site-recovery-test-failover-to-azure.md). Les exercices sont utiles pour :
    - Trouver les lacunes dans votre déploiement avant le basculement.
    - Estimer l’objectif de délai de récupération de bout en bout pour vos applications.
    - Estimer l’objectif de point de récupération de bout en bout pour vos applications.
    - Identifier des conflits de plage d’adresses IP.
    - Nous vous recommandons de ne pas utiliser de réseaux de production pour les exercices et de nettoyer les tests de basculements après chaque exercice.

Pour exécuter un basculement à grande échelle, nous recommandons de procéder comme suit :

1. Créez des plans de récupération pour le basculement de charge de travail.
    - Chaque plan de récupération peut déclencher le basculement de jusqu’à 100 machines.
    - [Découvrez d’autres informations](recovery-plan-overview.md) sur les plans de récupération.
2. Ajoutez des scripts de runbook Azure Automation aux plans de récupération pour automatiser toutes les tâches manuelles sur Azure. Les tâches typiques incluent la configuration d’équilibreurs de charge, la mise à jour de DNS, etc. [En savoir plus](site-recovery-runbook-automation.md)
2. Avant le basculement, préparez les machines Windows afin qu’elles soient conformes à l’environnement Azure. Les [limites de nombre de basculements](#plan-azure-subscriptions-and-quotas) sont plus élevées pour les machines conformes. [Apprenez-en davantage](site-recovery-failover-to-azure-troubleshoot.md#failover-failed-with-error-id-170010) sur les runbooks.
4.  Déclenchez le basculement avec la cmdlet PowerShell [Start-AzRecoveryServicesAsrPlannedFailoverJob](/powershell/module/az.recoveryservices/start-azrecoveryservicesasrplannedfailoverjob?view=azps-2.0.0&viewFallbackFrom=azps-1.1.0), ainsi qu’un plan de récupération.



## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Surveiller Site Recovery](site-recovery-monitor-and-troubleshoot.md)
