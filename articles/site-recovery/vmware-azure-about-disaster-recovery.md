---
title: Reprise d’activité après sinistre de VMware avec Azure Site Recovery
description: Cet article fournit une vue d’ensemble de la reprise d’activité de machines virtuelles VMware sur Azure à l’aide du service Azure Site Recovery.
author: rayne-wiselman
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/12/2019
ms.author: raynew
ms.openlocfilehash: dfbdff01064b483085233ece47d1d3b635b68743
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87021458"
---
# <a name="about-disaster-recovery-of-vmware-vms-to-azure"></a>À propos de la reprise d’activité de machines virtuelles VMware sur Azure

Cet article fournit une vue d’ensemble de la reprise d’activité de machines virtuelles VMware locales sur Azure à l’aide du service [Azure Site Recovery](site-recovery-overview.md).

## <a name="what-is-bcdr"></a>Qu’est-ce que la continuité d’activité et reprise d’activité (BCDR) ?

Une stratégie de continuité d’activité et reprise d’activité (BCDR) vous aide à maintenir votre entreprise opérationnelle. Pendant des temps d’arrêt planifiés et des interruptions inattendues, une stratégie BCDR préserve la sécurité et la disponibilité des données, et veille à ce que les applications continuent de fonctionner. En plus des fonctionnalités de continuité d’activité et reprise d’activité (BCDR) de plateforme telles que les homologations régionales et le stockage haute disponibilité, Azure fournit Recovery Services en tant que partie intégrante de votre solution BCDR. Microsoft Azure Recovery Services inclut les services suivants : 

- [Sauvegarde Azure](../backup/backup-overview.md) : sauvegarde vos données locales et de machines virtuelles Azure. Vous pouvez sauvegarder un fichier, des dossiers, des charges de travail spécifiques ou une machine virtuelle entière. 
- [Azure Site Recovery](site-recovery-overview.md) : assure la résilience et la reprise d’activité d’applications et charges de travail exécutées sur des machines locales ou des machines virtuelles Azure IaaS. Le service Site Recovery orchestre la réplication et gère le basculement vers Azure en cas d’interruption. Il gère également la récupération à partir d’Azure sur votre site principal. 

> [!NOTE]
> Site Recovery ne déplace pas et ne stocke pas les données client en dehors de la région cible dans laquelle la récupération d’urgence a été configurée pour les ordinateurs sources. S’ils le souhaitent, les clients peuvent sélectionner un coffre Recovery Services dans une autre région. Le coffre Recovery Services contient des métadonnées, mais pas de données client réelles.

## <a name="how-does-site-recovery-do-disaster-recovery"></a>Comment le service Site Recovery effectue-t-il la reprise d’activité ?

1. Après avoir préparé Azure et votre site local, vous configurez et activez la réplication pour vos machines locales.
2. Site Recovery orchestre la réplication initiale des machines, conformément aux paramètres de votre stratégie.
3. Après la réplication initiale, Site Recovery réplique les modifications d’ordre différentiel sur Azure. 
4. Lorsque la réplication fonctionne comme prévu, vous exécutez une simulation de reprise d’activité.
    - Cette simulation vous aide à vérifier que le basculement fonctionnera comme prévu en cas de réelle nécessité.
    - La simulation effectue un test de basculement n’affectant pas votre environnement de production.
5. En cas d’interruption, vous opérez un basculement complet vers Azure. Vous pouvez basculer une seule machine, ou créer un plan de récupération qui bascule plusieurs machines simultanément.
6. En cas de basculement, des machines virtuelles Azure sont créées à partir des données de machines virtuelles disponibles dans les disques managés et les comptes de stockage. Les utilisateurs peuvent continuer à accéder aux applications et aux charges de travail à partir de la machine virtuelle Azure.
7. Lorsque votre site local redevient disponible, vous pouvez opérer une restauration automatique à partir d’Azure.
8. Après une restauration automatique, lorsque vous retravaillez à partir de votre site principal, vous recommencez à répliquer les machines virtuelles locales sur Azure.


## <a name="how-do-i-know-if-my-environment-is-suitable-for-disaster-recovery-to-azure"></a>Comment savoir si mon environnement est approprié pour une reprise d’activité sur Azure ?

Le service Site Recovery peut répliquer toute charge de travail exécutée sur une machine virtuelle VMware ou un serveur physique pris en charge. Voici ce que vous devez vérifier dans votre environnement :

- Si vous répliquez des machines virtuelles VMware, exécutez-vous les versions appropriées des serveurs de virtualisation VMware ? [Vérifiez-le ici](vmware-physical-azure-support-matrix.md#on-premises-virtualization-servers).
- Les machines que vous souhaitez répliquer exécutent-elles un système d’exploitation pris en charge ? [Vérifiez-le ici](vmware-physical-azure-support-matrix.md#replicated-machines).
- Pour la reprise d’activité de Linux, les machines exécutent-elles un système de fichiers ou un stockage invité pris en charge ? [Vérifiez-le ici](vmware-physical-azure-support-matrix.md#linux-file-systemsguest-storage).
- Les machines à répliquer sont-elles conformes à la configuration requise pour Azure ? [Vérifiez-le ici](vmware-physical-azure-support-matrix.md#azure-vm-requirements).
- La configuration de votre réseau est-elle prise en charge ? [Vérifiez-le ici](vmware-physical-azure-support-matrix.md#network).
- La configuration de votre stockage est-elle prise en charge ? [Vérifiez-le ici](vmware-physical-azure-support-matrix.md#storage).


## <a name="what-do-i-need-to-set-up-in-azure-before-i-start"></a>Que dois-je configurer dans Azure avant de commencer ?

Dans Azure, vous devez effectuer les préparatifs suivants :

1. Vérifier que votre compte Azure dispose des autorisations nécessaires pour créer des machines virtuelles dans Azure.
2. Créer un réseau Azure que les machines virtuelles Azure rejoindront une fois créées à partir des comptes de stockage ou les disques managés après le basculement.
3. Créer un coffre Azure Recovery Services pour le service Site Recovery. Le coffre, qui réside dans le portail Azure, est utilisé pour déployer, configurer, orchestrer, surveiller et dépanner votre déploiement Azure Site Recovery.

*Besoin de plus d’aide ?*

Découvrez comment configurer Azure en [vérifiant votre compte](tutorial-prepare-azure.md#verify-account-permissions), en créant un [réseau](tutorial-prepare-azure.md#set-up-an-azure-network) et en un [configurant un coffre](tutorial-prepare-azure.md#create-a-recovery-services-vault).



## <a name="what-do-i-need-to-set-up-on-premises-before-i-start"></a>Que dois-je configurer localement avant de commencer ?

Voici ce que vous devez faire localement :

1. Vous devez configurer deux comptes :

    - Si vous répliquez des machines virtuelles VMware, un compte est nécessaire pour que Site Recovery puisse accéder à vCenter Server ou aux hôtes ESXi vSphere afin de découvrir automatiquement les machines virtuelles.
    - Un compte est nécessaire pour installer l’agent du service Mobilité Azure Site Recovery sur chaque machine physique ou virtuelle que vous souhaitez répliquer.

2. Vous devez vérifier la compatibilité de votre infrastructure VMware si ce n’est déjà fait.
3. Assurez-vous que vous pouvez vous connecter aux machines virtuelles Azure après un basculement. Vous configurez le protocole RDP sur des machines Windows locales, ou le protocole SSH sur des machines Linux.

*Besoin de plus d’aide ?*
- Préparez des comptes pour la [détection automatique](vmware-azure-tutorial-prepare-on-premises.md#prepare-an-account-for-automatic-discovery) et pour l’[installation du service Mobilité](vmware-azure-tutorial-prepare-on-premises.md#prepare-an-account-for-mobility-service-installation).
- [Vérifiez](vmware-azure-tutorial-prepare-on-premises.md#check-vmware-requirements) que vos paramètres VMware sont compatibles.
- [Préparez](vmware-azure-tutorial-prepare-on-premises.md#prepare-to-connect-to-azure-vms-after-failover) la connexion dans Azure après le basculement.
- Pour une aide plus détaillée sur la configuration de l’adressage IP pour les machines virtuelles Azure après basculement, [lisez cet article](concepts-on-premises-to-azure-networking.md).

## <a name="how-do-i-set-up-disaster-recovery"></a>Comment configurer la reprise d’activité ?

Une fois vos infrastructures Azure et locale en place, vous pouvez configurer la reprise d’activité.

1. Pour comprendre les composants que vous allez devoir déployer, examinez l’[Architecture VMware vers Azure](vmware-azure-architecture.md) et l’[Architecture de serveur physique vers Azure](physical-azure-architecture.md). Étant donné qu’il existe plusieurs composants, il est important de comprendre comment ils s’assemblent.
2. **Environnement source** : la première étape du déploiement consiste à configurer votre environnement source de réplication. Sélectionnez ce que vous voulez répliquer et l’emplacement de la réplication.
3. **Serveur de configuration** : vous devez configurer un serveur de configuration dans votre environnement source local :
    - Le serveur de configuration est une simple machine locale. Pour la reprise d’activité de VMware, nous vous recommandons de déployer une machine virtuelle VMware à partir d’un modèle OVF téléchargeable.
    - Le serveur de configuration coordonne les communications entre les machines locale et Azure.
    - Deux autres composants s’exécutent sur la machine serveur de configuration.
        - Le serveur de processus reçoit, optimise et envoie les données de réplication au compte de stockage de cache dans Azure. Il gère également l’installation automatique du service Mobilité sur les machines que vous souhaitez répliquer, et effectue la détection automatique des machines virtuelles sur les serveurs VMware.
        - Le serveur cible maître gère les données de réplication pendant la restauration automatique à partir d’Azure.
    - La configuration inclut l’inscription du serveur de configuration dans le coffre, le téléchargement du serveur MySQL et de VMware PowerCLI, ainsi que la spécification des comptes créés pour la détection automatique et l’installation du service Mobilité.
4. **Environnement cible** : vous configurez votre environnement Azure cible en spécifiant vos paramètres d’abonnement et de réseau Azure.
5. **Stratégie de réplication** : vous spécifiez la manière dont la réplication doit avoir lieu. Les paramètres définissent la fréquence à laquelle les points de récupération sont créés et stockés, et si des instantanés de cohérence d’application doivent être créés.
6. **Activer la réplication**. Vous activez la réplication pour les machines locales. Si vous avez créé un compte pour installer le service Mobilité, il sera installé lorsque vous activerez la réplication pour une machine. 

*Besoin de plus d’aide ?*

- Pour obtenir une présentant rapide de ces étapes, vous pouvez essayer notre [didacticiel VMware](vmware-azure-tutorial.md) et notre [procédure pas à pas de serveur physique](physical-azure-disaster-recovery.md).
- [Apprenez-en davantage](vmware-azure-set-up-source.md) sur la configuration de votre environnement source.
- [Découvrez](vmware-azure-deploy-configuration-server.md) la configuration requise du serveur de configuration et la façon de configurer celui-ci avec un modèle OVF pour la réplication de VMware. Si, pour une raison quelconque, vous ne pouvez pas utiliser de modèle, ou si vous répliquez des serveurs physiques, [suivez ces instructions](physical-azure-set-up-source.md#set-up-the-source-environment).
- [Apprenez-en davantage](vmware-azure-set-up-target.md) sur les paramètres cibles.
- [Obtenez des informations supplémentaires](vmware-azure-set-up-replication.md) sur la configuration d’une stratégie de réplication.
- [Découvrez](vmware-azure-enable-replication.md) comment activer la réplication et [exclure](vmware-azure-exclude-disk.md) des disques celle-ci.


## <a name="something-went-wrong-how-do-i-troubleshoot"></a>Quelque chose a mal tourné. Comment dépanner ?

- Dans un premier temps, essayez de [surveiller votre déploiement](site-recovery-monitor-and-troubleshoot.md) afin de vérifier l’état des éléments répliqués, les travaux et les problèmes d’infrastructure, ainsi qu’identifier des erreurs éventuelles.
- Si vous ne pouvez pas effectuer la réplication initiale ou si la réplication en cours ne fonctionne pas comme prévu, [lisez cet article](vmware-azure-troubleshoot-replication.md) décrivant des erreurs courantes et prodiguant des conseils de dépannage.
- Si vous rencontrez des problèmes avec l’installation automatique du service Mobilité sur des machines que vous souhaitez répliquer, passez en revue les erreurs courantes décrites dans [cet article](vmware-azure-troubleshoot-push-install.md).
- Si le basculement ne fonctionne pas comme prévu, passez en revue les erreurs courantes décrites dans [cet article](site-recovery-failover-to-azure-troubleshoot.md).
- Si la restauration automatique ne fonctionne pas, vérifiez si votre problème est décrit dans [cet article](vmware-azure-troubleshoot-failback-reprotect.md).



## <a name="next-steps"></a>Étapes suivantes

La réplication étant désormais en place, vous devriez [exécuter une simulation de reprise d’activité](tutorial-dr-drill-azure.md) pour vous assurer que le basculement fonctionne comme prévu. 
