---
title: Configurer la récupération d’urgence de SAP NetWeaver à l’aide d’Azure Site Recovery
description: Découvrez comment configurer la récupération d’urgence de SAP NetWeaver à l’aide d’Azure Site Recovery.
author: sideeksh
manager: rochakm
ms.topic: how-to
ms.date: 11/27/2018
ms.openlocfilehash: 69f0a20bdcba23d947e3d1b573c1a359da245161
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92369418"
---
# <a name="set-up-disaster-recovery-for-a-multi-tier-sap-netweaver-app-deployment"></a>Configurer la reprise d’activité pour un déploiement d’application SAP NetWeaver multiniveau

La plupart des déploiements SAP, de moyenne et grande taille, utilisent une forme de solution de reprise d’activité après sinistre. Disposer de solutions de reprise d’activité fiables et testables est de plus en plus important, d’autant qu’un nombre croissant de processus métier essentiels sont déplacés vers des applications telles que SAP. Azure Site Recovery a été testé et intégré aux applications SAP. Site Recovery dépasse les capacités de la plupart des solutions locales de récupération d’urgence, et à un coût total de possession moins élevé que les solutions concurrentes.

Avec Site Recovery, vous pouvez effectuer les actions décrites ici.
* Activer la protection des applications de production SAP NetWeaver et non-NetWeaver qui s’exécutent localement par la réplication des composants vers Azure.
* Activer la protection des applications de production SAP NetWeaver et non-NetWeaver qui s’exécutent sur Azure par la réplication des composants sur un autre centre de données Azure.
* Simplifier la migration vers le cloud en utilisant Site Recovery pour migrer votre déploiement SAP vers Azure.
* Simplifier les mises à niveau, les tests et la création de prototypes de projet SAP par la création d’un clone de production à la demande pour tester les applications SAP.

Vous pouvez protéger les déploiements d’applications SAP NetWeaver à l’aide d’[Azure Site Recovery](site-recovery-overview.md). Cet article traite des meilleures pratiques permettant de protéger un déploiement SAP NetWeaver à trois niveaux sur Azure lorsque vous répliquez vers un autre centre de données Azure à l’aide de Site Recovery. Il décrit les configurations et scénarios pris en charge ainsi que la façon de réaliser des tests de basculement (tests de récupération d’urgence) et des basculements réels.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que vous savez accomplir les tâches suivantes :

* [Répliquer une machine virtuelle vers Azure](./azure-to-azure-tutorial-enable-replication.md)
* [Constituer un réseau de récupération](./azure-to-azure-about-networking.md)
* [Effectuer un test de basculement vers Azure](./azure-to-azure-tutorial-dr-drill.md)
* [Procéder à un basculement vers Azure](site-recovery-failover.md)
* [Répliquer un contrôleur de domaine](site-recovery-active-directory.md)
* [Répliquer une instance SQL Server](site-recovery-sql.md)

## <a name="supported-scenarios"></a>Scénarios pris en charge

Vous pouvez utiliser Site Recovery pour implémenter une solution de reprise d’activité dans les scénarios décrits ici.
* Vous disposez de systèmes SAP qui s’exécutent dans un centre de données Azure et que vous répliquez dans un autre centre de données Azure (récupération d’urgence Azure vers Azure). 
   Pour plus d’informations, consultez [Architecture de réplication Azure vers Azure](./azure-to-azure-architecture.md).
* Les systèmes SAP s’exécutent sur des serveurs VMware (ou physiques) locaux. Vous répliquez également les systèmes SAP sur un site de récupération d’urgence dans un centre de centres Azure (récupération d’urgence VMware vers Azure). 
   Ce scénario nécessite quelques composants supplémentaires. Pour plus d’informations, consultez [Architecture de réplication VMware vers Azure](./vmware-azure-architecture.md).
* Vous disposez de systèmes SAP s’exécutant sur Hyper-V localement. Vous répliquez également les systèmes SAP sur un site de récupération d’urgence dans un centre de centres Azure (récupération d’urgence Hyper-V vers Azure).
   Ce scénario nécessite quelques composants supplémentaires. Pour plus d’informations, consultez [Architecture de réplication Hyper-V vers Azure](./hyper-v-azure-architecture.md).

Dans cet article, nous utilisons une scénario de récupération d’urgence **Azure vers Azure** . Le scénario vous montre les fonctionnalités de récupération d’urgence SAP de Site Recovery. La réplication Site Recovery n’étant pas propre à l’application, le processus qui est décrit est censé s’appliquer également à d’autres scénarios.

### <a name="required-foundation-services"></a>Services de base nécessaires
Dans le scénario que cet article développe, les services de base suivants sont déployés :
* Azure ExpressRoute ou Passerelle VPN Azure
* Au moins un contrôleur de domaine Azure Active Directory et un serveur DNS, s’exécutant dans Azure

Nous vous recommandons de mettre en place cette infrastructure avant de déployer Site Recovery.

## <a name="reference-sap-application-deployment"></a>Déploiement des applications SAP de référence

Cette architecture de référence exécute SAP NetWeaver dans un environnement Windows sur Azure avec une haute disponibilité. Cette architecture est déployée avec des tailles de machine virtuelle spécifiques que vous pouvez modifier en fonction des besoins de votre organisation.

![Schéma d’un modèle de déploiement SAP classique](./media/site-recovery-sap/sap-netweaver_latest.png)

## <a name="disaster-recovery-considerations"></a>Considérations relatives à la récupération d’urgence

Pour la récupération d’urgence, vous devez être en mesure de basculer vers une région secondaire. Chaque couche utilise une stratégie différente pour protéger la récupération d’urgence.

#### <a name="vms-running-sap-web-dispatcher-pools"></a>Machines virtuelles exécutant des pools SAP Web Dispatcher

Le composant Web Dispatcher fonctionne comme un équilibreur de charge pour le trafic SAP entre les serveurs d’applications SAP. Afin d’assurer la haute disponibilité pour le composant Web Dispatcher, Azure Load Balancer implémente l’installation parallèle de Web Dispatcher. Web Dispatcher utilise une configuration de tourniquet (round robin) pour la distribution du trafic HTTP(S) entre les instances Web Dispatcher disponibles dans le pool d’équilibreurs.

#### <a name="vms-running-application-servers-pools"></a>Machines virtuelles exécutant des pools de serveurs d’applications
La transaction SMLG gère les groupes de connexion des serveurs d’applications ABAP. Elle s’appuie sur la fonction d’équilibrage de charge au sein du serveur de messages des services centraux pour répartir la charge de travail entre les pools de serveurs d’applications SAP pour les clients SAPGUI et le trafic RFC. Vous pouvez répliquer cette gestion à l’aide de Site Recovery.

#### <a name="vms-running-sap-central-services-clusters"></a>Machines virtuelles exécutant des clusters de services centraux SAP
Cette architecture de référence exécute les services centraux sur les machines virtuelles de la couche application. Les services centraux constituent un potentiel point de défaillance unique lorsqu’ils se trouvent dans une seule machine virtuelle. Un déploiement classique et la haute disponibilité ne sont pas des exigences.

Pour implémenter une solution à haute disponibilité, vous pouvez utiliser soit un cluster de disque partagé, soit un cluster de partage de fichiers. Pour configurer des machines virtuelles pour un cluster de disque partagé, utilisez le cluster de basculement Windows Server. Nous vous recommandons d’utiliser le témoin cloud comme témoin de quorum.

 > [!NOTE]
 > Étant donné que Site Recovery ne réplique pas le témoin cloud, nous vous recommandons de déployer le témoin cloud dans la région de récupération d’urgence.

Pour prendre en charge l’environnement de cluster de basculement, [SIOS DataKeeper Cluster Edition](https://azuremarketplace.microsoft.com/marketplace/apps/sios_datakeeper.sios-datakeeper-8) assure la fonction de volume partagé du cluster. Dans la fonction, SIOS DataKeeper Cluster réplique les disques indépendants détenus par les nœuds de cluster. Dans la mesure où Azure ne prend pas en charge de manière native les disques partagés, la plateforme requiert des solutions fournies par SIOS.

Vous pouvez également gérer le clustering en implémentant un cluster de partage de fichiers. [SAP](https://blogs.sap.com/2018/03/19/migration-from-a-shared-disk-cluster-to-a-file-share-cluster) a récemment modifié le modèle de déploiement des services centraux pour accéder aux répertoires globaux /sapmnt via un chemin d’accès UNC. Nous vous recommandons toutefois de vous assurer que le partage UNC /sapmnt est hautement disponible. Vous pouvez vérifier votre instance des services centraux. Utilisez le cluster de basculement Windows Server avec le serveur de fichiers Scale Out (SOFS) et la fonctionnalité d’espaces de stockage direct (S2D) de Windows Server 2016.

 > [!NOTE]
 > Site Recovery prend actuellement en charge uniquement la réplication de points cohérents en cas d’incident de machines virtuelles qui utilisent des espaces de stockage direct et le nœud passif de SIOS Datakeeper.


## <a name="more-disaster-recovery-considerations"></a>Autres considérations relatives à la récupération d’urgence

Vous pouvez utiliser Site Recovery pour orchestrer le basculement d’un déploiement SAP complet entre des régions Azure.
Voici les étapes de configuration de la récupération d’urgence :

1. Répliquer des machines virtuelles
1. Constituer un réseau de récupération
1. Répliquer un contrôleur de domaine
1. Répliquer le niveau de base de données
1. Exécuter un test de basculement
1. Effectuer un basculement

Voici la recommandation pour la récupération d’urgence de chaque niveau utilisé dans cet exemple.

 **Niveaux SAP** | **Recommandation**
 --- | ---
**Pool SAP Web Dispatcher** |  Répliquer à l’aide de Site Recovery 
**Pool de serveurs d’applications SAP** |  Répliquer à l’aide de Site Recovery 
**Cluster des services centraux SAP** |  Répliquer à l’aide de Site Recovery 
**Machines virtuelles Active directory** |  Utiliser la réplication Active Directory 
**Serveurs SQL Database** |  Utiliser la réplication SQL Server Always On

## <a name="replicate-virtual-machines"></a>Répliquer des machines virtuelles

Pour commencer la réplication de toutes les machines virtuelles d’application SAP vers le centre de données de reprise d’activité Azure, suivez les instructions dans [Répliquer une machine virtuelle vers Azure](./azure-to-azure-tutorial-enable-replication.md).

* Pour obtenir des conseils sur la protection d’Active Directory et de DNS, découvrez [comment protéger Active Directory et DNS](site-recovery-active-directory.md).

* Pour obtenir des conseils sur la protection d’une couche de base de données s’exécutant sur SQL Server, découvrez [comment protéger SQL Server](site-recovery-sql.md).

## <a name="networking-configuration"></a>Configuration de la mise en réseau

Si vous utilisez une adresse IP statique, vous pouvez spécifier l’adresse IP que vous souhaitez attribuer à la machine virtuelle. Pour définir l’adresse IP, accédez à **Paramètres Calcul et réseau** > **Carte d’interface réseau** .

![Capture d’écran qui illustre la définition d’une adresse IP privée dans le volet de carte d’interface réseau de Site Recovery](./media/site-recovery-sap/sap-static-ip.png)


## <a name="create-a-recovery-plan"></a>Créer un plan de récupération

Lors d’un basculement, un plan de récupération prend en charge le séquencement des différents niveaux d’une application multiniveau. La mise en séquence permet d’assurer la cohérence de l’application. Lorsque vous créez un plan de récupération pour une application web multiniveau, suivez les étapes décrites dans [Créer un plan de récupération à l’aide de Site Recovery](site-recovery-create-recovery-plans.md).

### <a name="add-virtual-machines-to-failover-groups"></a>Ajouter des machines virtuelles à des groupes de basculement

1. Créez un plan de récupération en ajoutant le serveur d’applications, Web Dispatcher et des machines virtuelles de services centraux SAP.
1. Sélectionnez **Personnaliser** pour regrouper les machines virtuelles. Par défaut, toutes les machines virtuelles font partie du groupe 1.

### <a name="add-scripts-to-the-recovery-plan"></a>Ajouter des scripts au plan de récupération
Afin de vous assurer du bon fonctionnement de vos applications, vous pouvez être amené à effectuer certaines opérations sur les machines virtuelles Azure. Effectuez ces opérations après le basculement ou pendant un test de basculement. Vous pouvez également automatiser certaines opérations après le basculement. Par exemple, mettez à jour l’entrée DNS, puis modifiez les liaisons et les connexions en ajoutant au plan de récupération les scripts correspondants.

Vous pouvez déployer les scripts Site Recovery les plus utilisés dans votre compte Azure Automation en sélectionnant **Déployer dans Azure** . Lorsque vous utilisez un script publié, suivez les instructions du script.

[![Déployer sur Azure](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/c4803408-340e-49e3-9a1f-0ed3f689813d.png)](https://aka.ms/asr-automationrunbooks-deploy)

1. Ajoutez un script d’action préalable au groupe 1 pour basculer le groupe de disponibilité SQL Server. Utilisez le script ASR-SQL-FailoverAG publié dans les exemples de script. Suivez les instructions du script et apportez les modifications requises dans le script en conséquence.
1. Ajoutez un script d’action postérieure pour attribuer un équilibreur de charge aux machines virtuelles basculées de niveau web (Groupe 1). Utilisez le script ASR-AddSingleLoadBalancer publié dans les exemples de script. Suivez les instructions du script et apportez les modifications requises dans le script le cas échéant.

![Plan de récupération SAP](./media/site-recovery-sap/sap_recovery_plan.png)


## <a name="run-a-test-failover"></a>Exécuter un test de basculement

1. Dans le portail Azure, sélectionnez votre coffre Recovery Services.
1. Sélectionnez le plan de récupération que vous avez créé pour les applications SAP.
1. Sélectionnez **Test de basculement** .
1. Pour démarrer le test de basculement, sélectionnez le point de récupération et le réseau virtuel Azure.
1. Lorsque l’environnement secondaire est opérationnel, procédez aux validations.
1. Lorsque les validations sont terminées, nettoyez l’environnement de basculement en sélectionnant **Nettoyer le test de basculement** .

Pour plus d’informations, consultez [Tester le basculement vers Azure dans Site Recovery](site-recovery-test-failover-to-azure.md).

## <a name="run-a-failover"></a>Exécuter un basculement

1. Dans le portail Azure, sélectionnez votre coffre Recovery Services.
1. Sélectionnez le plan de récupération que vous avez créé pour les applications SAP.
1. Sélectionnez **Basculement** .
1. Pour démarrer le processus de basculement, sélectionnez le point de récupération.

Pour plus d’informations, consultez [Basculement dans Site Recovery](site-recovery-failover.md).

## <a name="next-steps"></a>Étapes suivantes
* Découvrez en détail la création d’une solution de récupération d’urgence pour les déploiements SAP NetWeaver à l’aide d’Azure Site Recovery. Consultez le livre blanc téléchargeable [SAP NetWeaver : Création d’une solution de reprise d’activité après sinistre avec Site Recovery](https://aka.ms/asr_sap). Ce livre blanc traite de recommandations pour diverses architectures SAP. Vous pouvez voir les applications et les types de machines virtuelles pris en charge pour SAP sur Azure. Il existe également des options de plan pour le test de votre solution de récupération d’urgence.
* Approfondissez vos connaissances sur la [réplication d’autres charges de travail](site-recovery-workload.md) à l’aide de Site Recovery.