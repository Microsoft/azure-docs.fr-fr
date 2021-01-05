---
title: Récupération d’urgence pour une application SharePoint multiniveau avec Azure Site Recovery
description: Cet article décrit comment configurer la reprise d’activité pour une application SharePoint multiniveau à l’aide des fonctionnalités d’Azure Site Recovery.
author: sujayt
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 6/27/2019
ms.author: sutalasi
ms.openlocfilehash: 08e971e52f994ec5fa5663708fa9f173daf33d80
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013749"
---
# <a name="set-up-disaster-recovery-for-a-multi-tier-sharepoint-application-for-disaster-recovery-using-azure-site-recovery"></a>Configurer la reprise d’activité pour une application SharePoint multiniveau à l’aide d’Azure Site Recovery

Cet article explique en détail comment protéger une application SharePoint à l’aide d’[Azure Site Recovery](site-recovery-overview.md).


## <a name="overview"></a>Vue d’ensemble

Microsoft SharePoint est une application puissante qui peut aider un groupe ou service à s’organiser, à collaborer et à partager des informations. SharePoint permet les portails intranet, la gestion des documents et fichiers, la collaboration, les réseaux sociaux, les réseaux extranet, les sites web, la recherche de contenu d’entreprise et le décisionnel. Elle offre également des capacités d’intégration de système, d’intégration de processus et d’automatisation de flux de travail. En règle générale, les organisations le considèrent comme une application de niveau 1 sensible à la perte de données et au temps d’arrêt.

Aujourd'hui, Microsoft SharePoint ne fournit aucune capacité de récupération d’urgence prête à l’emploi. Quels que soient le type et l’ampleur d’un sinistre, la récupération implique d’utiliser un centre de données de secours grâce auquel vous pouvez récupérer la batterie de serveurs. Des centres de données de secours sont nécessaires lorsque les sauvegardes et systèmes redondants locaux ne peuvent pas être récupérés après une panne au centre de données principal.

Une bonne solution de récupération d’urgence doit permettre de modéliser des plans de récupération sur les architectures d’application complexes telles que SharePoint. Elle doit également offrir la possibilité d’ajouter des étapes personnalisées pour gérer des mappages d’application entre les différents niveaux et, par conséquent, fournir un basculement par simple clic avec un RTO inférieur en cas de sinistre.

Cet article explique en détail comment protéger une application SharePoint à l’aide d’[Azure Site Recovery](site-recovery-overview.md). Cet article décrit les recommandations à suivre pour répliquer une application SharePoint à trois niveaux dans Azure, et indique comment vous pouvez effectuer un exercice de récupération d’urgence et comment vous pouvez basculer l’application vers Azure.

Vous pouvez regarder la vidéo ci-dessous sur la récupération d’une application multiniveau dans Azure.

> [!VIDEO https://channel9.msdn.com/Series/Azure-Site-Recovery/Disaster-Recovery-of-load-balanced-multi-tier-applications-using-Azure-Site-Recovery/player]


## <a name="prerequisites"></a>Prérequis

Avant de commencer, veillez à bien comprendre ce qui suit :

1. [Réplication d’une machine virtuelle dans Azure](./vmware-azure-tutorial.md)
2. [Conception d’un réseau de récupération](./concepts-on-premises-to-azure-networking.md)
3. [Réalisation d’un test de basculement vers Azure](site-recovery-test-failover-to-azure.md)
4. [Exécution d’un basculement vers Azure](site-recovery-failover.md)
5. [Réplication d’un contrôleur de domaine](site-recovery-active-directory.md)
6. [Réplication de SQL Server](site-recovery-sql.md)

## <a name="sharepoint-architecture"></a>Architecture SharePoint

SharePoint peut être déployée sur un ou plusieurs serveurs à l’aide de rôles de serveur et de topologies à plusieurs niveaux pour implémenter une conception de batterie de serveurs qui répond à des objectifs spécifiques. Une batterie de serveurs SharePoint de grande capacité et à forte demande qui prend en charge un grand nombre d’utilisateurs simultanés et un grand nombre d’éléments de contenu utilise le regroupement de service dans le cadre de sa stratégie d’évolutivité. Cette approche implique d’exécuter les services sur des serveurs dédiés, de regrouper ces services puis d’augmenter la taille des instances des serveurs en tant que groupe. La topologie suivante illustre le regroupement des services et serveurs pour une batterie de serveurs SharePoint à trois niveaux. Reportez-vous à la documentation de SharePoint et aux architectures de gamme de produits pour des instructions détaillées sur les différentes topologies de SharePoint. Vous trouverez plus d’informations sur le déploiement de SharePoint 2013 dans [ce document](/SharePoint/sharepoint-server).



![Modèle de déploiement 1](./media/site-recovery-sharepoint/sharepointarch.png)


## <a name="site-recovery-support"></a>Prise en charge de Site Recovery

Site Recovery est indépendant de toute application et doit fonctionner avec n'importe quelle version de SharePoint exécutée sur un ordinateur pris en charge. Pour la création de cet article, des machines virtuelles VMware avec Windows Server 2012 R2 Enterprise ont été utilisées. Les applications SharePoint 2013 Enterprise Edition et SQL Server 2014 Enterprise Edition ont été utilisées.

### <a name="source-and-target"></a>Source et cible

**Scénario** | **Vers un site secondaire** | **Vers Azure**
--- | --- | ---
**Hyper-V** | Oui | Oui
**VMware** | Oui | Oui
**Serveur physique** | Oui | Oui
**Microsoft Azure** | N/D | Oui


### <a name="things-to-keep-in-mind"></a>Points à prendre en compte

Si vous utilisez un cluster basé sur disque partagé comme n’importe quel niveau dans votre application, vous ne pourrez pas utiliser la réplication de Site Recovery pour répliquer ces machines virtuelles. Vous pouvez utiliser la réplication native fournie par l’application, puis utiliser un [plan de récupération](site-recovery-create-recovery-plans.md) pour basculer tous les niveaux.

## <a name="replicating-virtual-machines"></a>Réplication des machines virtuelles

Suivez [ce guide](./vmware-azure-tutorial.md) pour démarrer la réplication de la machine virtuelle dans Azure.

* Une fois la réplication terminée, assurez-vous que vous accédez à chaque machine virtuelle de chaque niveau et sélectionnez le même groupe à haute disponibilité dans « Élément répliqué > Paramètres > Propriétés > Calcul et réseau ». Par exemple, si votre niveau web comporte 3 machines virtuelles, vérifiez que les 3 machines virtuelles sont configurées pour faire partie du même groupe à haute disponibilité dans Azure.

    ![Set-Availability-Set](./media/site-recovery-sharepoint/select-av-set.png)

* Pour obtenir des conseils sur la protection d’Active Directory et de DNS, consultez le document [Protéger Active Directory et DNS](site-recovery-active-directory.md).

* Pour obtenir des conseils sur la protection d’un niveau de base de données en cours d’exécution sur SQL Server, consultez le document [Protéger SQL Server](site-recovery-sql.md).

## <a name="networking-configuration"></a>Configuration de la mise en réseau

### <a name="network-properties"></a>Propriétés réseau

* Pour les machines virtuelles de niveau web et de niveau application, configurez les paramètres réseau dans le portail Azure afin que les machines virtuelles soient associées au bon réseau de récupération d’urgence après le basculement.

    ![Sélectionner le réseau](./media/site-recovery-sharepoint/select-network.png)


* Si vous utilisez une adresse IP statique, spécifiez l’adresse IP que vous souhaitez attribuer à la machine virtuelle dans le champ **Adresse IP cible**

    ![Définir l’adresse IP statique](./media/site-recovery-sharepoint/set-static-ip.png)

### <a name="dns-and-traffic-routing"></a>Routage du trafic et DNS

Pour les sites accessibles sur Internet, [créez un profil Traffic Manager de type « Priorité »](../traffic-manager/quickstart-create-traffic-manager-profile.md) dans l’abonnement Azure. Puis, configurez votre profil DNS et Traffic Manager de la manière suivante.


| **Where** | **Source** | **Cible**|
| --- | --- | --- |
| DNS public | DNS public pour les sites SharePoint <br/><br/> Exemple : sharepoint.contoso.com | Traffic Manager <br/><br/> contososharepoint.trafficmanager.net |
| DNS local | sharepointonprem.contoso.com | Adresse IP publique sur la batterie de serveurs locale |


Dans le profil Traffic Manager, [créez les points de terminaison principaux et de récupération](../traffic-manager/traffic-manager-configure-priority-routing-method.md). Utilisez le point de terminaison externe pour le point de terminaison local et l’adresse IP publique pour le point de terminaison Azure. Vérifiez que la priorité définie est plus élevée que celle du point de terminaison local.

Hébergez une page de test sur un port spécifique (800 par exemple) du niveau web de SharePoint afin que Traffic Manager détecte automatiquement la disponibilité après basculement. Il s’agit d’une solution de contournement si vous ne pouvez pas activer l’authentification anonyme sur un de vos sites SharePoint.

[Configurez le profil Traffic Manager](../traffic-manager/traffic-manager-configure-priority-routing-method.md) avec les paramètres ci-dessous.

* Méthode de routage - « Priorité »
* Durée de vie du DNS - « 30 secondes »
* Paramètres d’analyse de point de terminaison - Si vous pouvez activer l’authentification anonyme, vous pouvez donner un point de terminaison de site web spécifique. Ou bien, vous pouvez utiliser une page de test sur un port spécifique (800 par exemple).

## <a name="creating-a-recovery-plan"></a>Création d’un plan de récupération

Un plan de récupération permet de séquencer le basculement de différents niveaux d’une application multiniveau, ce qui contribue au maintien de la cohérence de l’application. Suivez les étapes indiquées ci-dessous lorsque vous créez un plan de récupération pour une application web multiniveau. [En savoir plus sur la création d’un plan de récupération](site-recovery-runbook-automation.md#customize-the-recovery-plan).

### <a name="adding-virtual-machines-to-failover-groups"></a>Ajout de machines virtuelles à des groupes de basculement

1. Créez un plan de récupération en ajoutant des machines virtuelles de niveau web et application.
2. Cliquez sur « Personnaliser » pour regrouper les machines virtuelles. Par défaut, toutes les machines virtuelles font partie du « Groupe 1 ».

    ![Personnaliser un RP](./media/site-recovery-sharepoint/rp-groups.png)

3. Créez un autre groupe (Groupe 2) et déplacez les machines virtuelles de niveau web dans le nouveau groupe. Vos machines virtuelles de niveau application doivent faire partie du « Groupe 1 » tandis que les machines virtuelles de niveau web doivent faire partie du « Groupe 2 ». Cela permet de s’assurer que les machines virtuelles de niveau application démarrent avant les machines virtuelles de niveau web.


### <a name="adding-scripts-to-the-recovery-plan"></a>Ajout de scripts au plan de récupération

Vous pouvez déployer les scripts Azure Site Recovery les plus couramment utilisés dans votre compte Automation en cliquant sur le bouton « Déployer dans Azure » ci-dessous. Lorsque vous utilisez n’importe quel script publié, assurez-vous de suivre les instructions dans le script.

[![Déployer sur Azure](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/c4803408-340e-49e3-9a1f-0ed3f689813d.png)](https://aka.ms/asr-automationrunbooks-deploy)

1. Ajoutez un script d’action préalable au « Groupe 1 » pour basculer le groupe de disponibilité SQL. Utilisez le script « ASR-SQL-FailoverAG » publié dans les exemples de script. Assurez-vous de suivre les instructions dans le script et apportez les modifications requises dans le script en conséquence.

    ![Add-AG-Script-Step-1](./media/site-recovery-sharepoint/add-ag-script-step1.png)

    ![Add-AG-Script-Step-2](./media/site-recovery-sharepoint/add-ag-script-step2.png)

2. Ajoutez un script d’action postérieure pour attribuer un équilibrage de charge sur les machines virtuelles de niveau web basculées (Groupe 2). Utilisez le script « ASR-AddSingleLoadBalancer » publié dans les exemples de script. Assurez-vous de suivre les instructions dans le script et apportez les modifications requises dans le script en conséquence.

    ![Add-LB-Script-Step-1](./media/site-recovery-sharepoint/add-lb-script-step1.png)

    ![Add-LB-Script-Step-2](./media/site-recovery-sharepoint/add-lb-script-step2.png)

3. Ajoutez une étape manuelle pour mettre à jour les enregistrements DNS afin qu’ils indiquent la nouvelle batterie de serveurs dans Azure.

    * Pour les sites accessibles sur Internet, aucune mise à jour des DNS n’est obligatoire après le basculement. Suivez les étapes décrites dans la section « Guide de mise en réseau » pour configurer Traffic Manager. Si le profil Traffic Manager a été configuré comme décrit dans la section précédente, ajoutez un script pour ouvrir un port factice (800 dans l’exemple) sur la machine virtuelle Azure.

    * Pour les sites accessibles en interne, ajoutez une étape manuelle pour mettre à jour l’enregistrement DNS afin qu’il indique l’adresse IP de l’équilibrage de charge de la nouvelle machine virtuelle de niveau web.

4. Ajoutez une étape manuelle pour restaurer l’application de recherche à partir d’une sauvegarde ou pour démarrer un nouveau service de recherche.

5. Pour restaurer l’application de service de recherche à partir d’une sauvegarde, suivez les étapes ci-dessous.

    * Cette méthode suppose qu’une sauvegarde de l’application de service de recherche a été effectuée avant l’événement catastrophique et que cette sauvegarde est disponible sur le site de récupération d’urgence.
    * Pour y parvenir facilement, il suffit de planifier la sauvegarde (par exemple, une fois par jour) et d’utiliser une procédure de copie pour placer la sauvegarde sur le site de récupération d’urgence. Les procédures de copie peuvent inclure des programmes par script comme AzCopy (Azure Copy) ou la configuration de DFSR (Distributed File Services Replication).
    * Maintenant que la batterie de serveurs SharePoint est en cours d’exécution, parcourez le site Administration centrale pour accéder à « Sauvegarde et restauration » puis sélectionnez Restaurer. La restauration interroge l’emplacement de sauvegarde spécifié (vous devrez peut-être mettre à jour la valeur). Sélectionnez la sauvegarde de l’application de service de recherche que vous voulez restaurer.
    * La recherche est restaurée. N’oubliez pas que la restauration s’attend à trouver la même topologie (même nombre de serveurs) et les mêmes lettres de lecteur affectées à ces serveurs. Pour plus d’informations, consultez le document [« Restaurer une application de service de recherche dans SharePoint 2013 »](/SharePoint/administration/restore-a-search-service-application).


6. Pour démarrer avec une nouvelle application de service de recherche, suivez les étapes ci-dessous.

    * Cette méthode suppose qu’une sauvegarde de la base de données « Administration de la recherche » est disponible sur le site de récupération d’urgence.
    * Étant donné que les autres bases de données d’application de service de recherche ne sont pas répliquées, elles doivent être recréées. Pour ce faire, accédez à l’Administration centrale et supprimez l’application de service de recherche. Sur les serveurs qui hébergent l’index de recherche, supprimez les fichiers d’index.
    * Recréez l’application de service de recherche pour recréer les bases de données. Il est recommandé d’avoir un script préparé qui recrée cette application de service, car il n’est pas possible d’effectuer toutes les actions via l’interface utilisateur. Par exemple, il n’est possible de définir l’emplacement du lecteur d’index et de configurer la topologie de recherche qu’à l’aide des applets de commande PowerShell de SharePoint. Utilisez l’applet de commande Windows PowerShell Restore-SPEnterpriseSearchServiceApplication et spécifiez la base de données d’administration de recherche répliquée et pour laquelle les journaux ont été copiés, Search_Service__DB. Cette applet de commande permet la configuration de la recherche, le schéma, les propriétés gérées, les règles et les sources, et crée un ensemble par défaut des autres composants.
    * Une fois que l’application de service de recherche a été recréée, vous devez démarrer une analyse complète pour chaque source de contenu afin de restaurer le service de recherche. Vous perdrez des informations analytiques provenant de la batterie de serveurs locale, telles que des recommandations de recherche.

7. Une fois toutes les étapes effectuées, enregistrez le plan de récupération. Le plan de récupération final se présente alors comme suit.

    ![RP enregistré](./media/site-recovery-sharepoint/saved-rp.png)

## <a name="doing-a-test-failover"></a>Exécution d’un test de basculement
Suivez [ce guide](site-recovery-test-failover-to-azure.md) pour effectuer un test de basculement.

1.  Accédez au portail Azure et sélectionnez votre coffre Recovery Services.
2.  Cliquez sur le plan de récupération créé pour l’application SharePoint.
3.  Cliquez sur Test de basculement.
4.  Sélectionnez un point de récupération et un réseau virtuel Azure pour démarrer le test de basculement.
5.  Lorsque l’environnement secondaire est opérationnel, vous pouvez effectuer vos validations.
6.  Une fois les validations terminées, vous pouvez cliquer sur « Nettoyage du test de basculement » sur le plan de récupération. L’environnement de test de basculement est alors nettoyé.

Pour obtenir des conseils sur la réalisation du test de basculement pour Active Directory et DNS, consultez le document [Considérations relatives au test de basculement pour Active Directory et DNS](site-recovery-active-directory.md#test-failover-considerations).

Pour obtenir des conseils sur la réalisation du test de basculement pour les groupes de disponibilité SQL AlwaysOn, consultez le document [Performing Application DR with Azure Site Recovery and doing Test failover (Récupération d’urgence d’application avec Azure Site Recovery et basculement de test)](site-recovery-sql.md#disaster-recovery-of-an-application).

## <a name="doing-a-failover"></a>Exécution d’un basculement
Suivez [ce guide](site-recovery-failover.md) pour réaliser un basculement.

1.  Accédez au portail Azure et sélectionnez votre coffre Recovery Services.
2.  Cliquez sur le plan de récupération créé pour l’application SharePoint.
3.  Cliquez sur « Basculement ».
4.  Sélectionnez un point de récupération pour démarrer le processus de basculement.

## <a name="next-steps"></a>Étapes suivantes
Vous pouvez en savoir plus sur la [réplication d’autres applications](site-recovery-workload.md) à l’aide de Site Recovery.
