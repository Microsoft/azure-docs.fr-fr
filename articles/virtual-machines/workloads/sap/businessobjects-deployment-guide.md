---
title: Déploiement de SAP BusinessObjects BI Platform sur Azure | Microsoft Docs
description: Planifier, déployer et configurer SAP BusinessObjects BI Platform sur Azure
services: virtual-machines-windows,virtual-network,storage,azure-netapp-files,azure-files
documentationcenter: saponazure
author: dennispadia
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-windows
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 10/05/2020
ms.author: depadia
ms.openlocfilehash: 0c2deb1ae1d41f8daaed8856f97c4b458930c616
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96484377"
---
# <a name="sap-businessobjects-bi-platform-planning-and-implementation-guide-on-azure"></a>Guide de planification et d’implémentation de la plateforme SAP BusinessObjects BI sur Azure

## <a name="overview"></a>Vue d’ensemble

Ce guide a été conçu pour vous fournir des instructions sur la planification, le déploiement et la configuration de SAP BusinessObjects BI Platform (SAP BOBI Platform) sur Azure. Il a vocation à couvrir les services et fonctionnalités Azure courants qui sont pertinents pour SAP BOBI Platform. Ce guide n'est pas une liste exhaustive de toutes les options de configuration possibles. Il couvre les solutions communes aux scénarios de déploiement classiques.

Ce guide n'a pas pour but de remplacer les guides d'installation et d'administration standard de SAP BOBI Platform, la documentation du système d'exploitation ou toute documentation relative aux bases de données.

## <a name="plan-and-implement-sap-businessobjects-bi-platform-on-azure"></a>Planifier et implémenter SAP BusinessObjects BI Platform sur Azure

Microsoft Azure offre un large éventail de services, comme le calcul, le stockage et la mise en réseau, pour permettre aux entreprises de créer leurs applications sans avoir à passer par de longs cycles d'approvisionnement. Les machines virtuelles Azure permettent aux entreprises de déployer des ressources informatiques à la demande et évolutives pour différentes applications SAP, comme les applications basées sur SAP NetWeaver, SAP Hybris et SAP BusinessObjects BI Platform, en fonction de leurs besoins. Azure prend également en charge la connectivité intersite, ce qui permet aux entreprises d'intégrer les machines virtuelles Azure dans leurs domaines locaux, leurs clouds privés et leur paysage SAP.

Ce document fournit des conseils sur la planification et l'implémentation de SAP BusinessObjects BI Platform sur Azure. Il vient compléter la documentation d'installation SAP et les Notes SAP, qui constituent les principales ressources à consulter pour les installations et les déploiements de SAP BOBI.

### <a name="architecture-overview"></a>Présentation de l'architecture

SAP BusinessObjects BI Platform est un système autonome qui peut se trouver sur une seule machine virtuelle Azure ou être mis à l'échelle dans un cluster de nombreuses machines virtuelles Azure exécutant différents composants. SAP BOBI Platform comprend six couches conceptuelles : couche Client, couche Web, couche Gestion, couche Stockage, couche Traitement et couche Données. (Pour plus d'informations sur chacune de ces couches, consultez le Guide de l'administrateur disponible sur le portail d'aide de [SAP BusinessObjects Business Intelligence Platform](https://help.sap.com/viewer/product/SAP_BUSINESSOBJECTS_BUSINESS_INTELLIGENCE_PLATFORM/4.3/en-US)). Vous trouverez ci-dessous une description détaillée de chaque couche :

- **Couche Client :** contient toutes les applications clientes de bureau qui interagissent avec BI Platform pour fournir différents types de fonctionnalités de création de rapports, d'analyse et d'administration.
- **Couche Web :** contient des applications web déployées sur des serveurs d'applications web JAVA. Les applications web fournissent la fonctionnalité BI Platform aux utilisateurs finaux via un navigateur web.
- **Couche Gestion :** coordonne et contrôle tous les composants de BI Platform. Comprend le serveur de gestion centralisée (CMS), ainsi que le serveur d'événements et les services associés.
- **Couche Stockage :** responsable du traitement des fichiers, tels que les documents et rapports. Gère également la mise en cache des rapports pour économiser des ressources système lorsque les utilisateurs accèdent aux rapports.
- **Couche Traitement :** analyse les données, et produit des rapports ainsi que d'autres types de sorties. Cette couche est la seule qui accède aux bases de données contenant les données des rapports.
- **Couche Données :** comprend les serveurs qui hébergent les bases de données du système CMS et le magasin de données d'audit.

SAP BI Platform est constituée d'un ensemble de serveurs exécutés sur un ou plusieurs hôtes. Il est essentiel de choisir la bonne stratégie de déploiement, en fonction du dimensionnement, des besoins de l'entreprise et du type d'environnement. Dans le cadre d'une installation de petite taille (développement ou test, par exemple), vous pouvez utiliser la même machine virtuelle Azure pour le serveur d'applications web, le serveur de bases de données et tous les serveurs de BI Platform. Si vous utilisez l'offre DBaaS (Base de données en tant que service) d'Azure, le serveur de bases de données s'exécutera séparément des autres composants. Dans le cadre d'une installation de taille intermédiaire ou de grande taille, les serveurs peuvent s'exécuter sur plusieurs machines virtuelles Azure.

La figure ci-dessous illustre l'architecture d'un déploiement à grande échelle de SAP BOBI Platform sur des machines virtuelles Azure. Chaque composant est distribué et placé dans des groupes à haute disponibilité qui peuvent prendre en charge le basculement en cas d'interruption de service.

![Architecture de SAP BusinessObjects BI Platform sur Azure](./media/businessobjects-deployment-guide/businessobjects-architecture-on-azure.png)

#### <a name="architecture-details"></a>Détails de l'architecture

- Équilibrage de charge

  Dans le cadre d'un déploiement multi-instance de SAP BOBI, les serveurs d'applications web (ou couche Web) sont exécutés sur au moins deux hôtes. Pour répartir uniformément la charge des utilisateurs sur les serveurs web, vous pouvez utiliser un équilibreur de charge entre les utilisateurs finaux et les serveurs web. Dans Azure, vous pouvez utiliser [Azure Load Balancer](../../../load-balancer/load-balancer-overview.md) ou [Azure Application Gateway](../../../application-gateway/overview.md) pour gérer le trafic vers vos serveurs web.

- Serveurs d'applications web

  Le serveur web héberge les applications web de SAP BOBI Platform, comme CMC et BI Launch Pad. Pour bénéficier de la haute disponibilité sur le serveur web, vous devez déployer au moins deux serveurs d'applications web afin de gérer la redondance et l'équilibrage de charge. Dans Azure, ces serveurs d'applications web peuvent être placés dans des groupes à haute disponibilité ou dans des zones de disponibilité pour profiter d'une meilleure disponibilité.

  Tomcat est l'application web par défaut de SAP BI Platform. Pour bénéficier de la haute disponibilité sur Tomcat, activez la réplication de session à l'aide de l'[intercepteur d'appartenance statique](https://tomcat.apache.org/tomcat-9.0-doc/config/cluster-membership.html#Static_Membership_Attributes) dans Azure. L'utilisateur aura ainsi toujours accès à l'application web SAP BI, même en cas d'interruption du service Tomcat.

  > [!Important]
  > Par défaut, Tomcat utilise l'adresse IP et le port de multidiffusion pour le clustering, ce qui n'est pas pris en charge sur Azure (Note SAP [2764907](https://launchpad.support.sap.com/#/notes/2764907)).

- Serveurs de BI Platform

  Les serveurs de BI Platform incluent tous les services qui font partie de l'application SAP BOBI (couche Gestion, couche Traitement et couche Stockage). Lorsqu'un serveur web reçoit une requête, il détecte tous les serveurs de BI Platform (plus précisément, tous les serveurs CMS d'un cluster) et équilibre automatiquement la charge des requêtes. En cas de défaillance d'un des hôtes de BI Platform, le serveur web envoie automatiquement les requêtes à un autre hôte.

  Pour bénéficier de la haute disponibilité ou de la redondance sur BI Platform, vous devez déployer l'application sur au moins deux machines virtuelles Azure. En fonction du dimensionnement, vous pouvez effectuer une mise à l'échelle de votre instance de BI Platform pour qu'elle s'exécute sur un plus grand nombre de machines virtuelles Azure.

- File Repository Server (FRS)

  File Repository Server contient tous les rapports et documents BI qui ont été créés. Dans le cadre d'un déploiement multi-instance, les serveurs de BI Platform sont exécutés sur plusieurs machines virtuelles, et chacune d'entre elles doit avoir accès à ces rapports et documents BI. Un système de fichiers doit donc être partagé entre tous les serveurs de BI Platform.

  Dans Azure, vous pouvez utiliser [Azure Premium Files](../../../storage/files/storage-files-introduction.md) ou [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) pour File Repository Server. Ces deux services Azure proposent une fonctionnalité de redondance intégrée.

  > [!Important]
  > Le protocole SMB pour Azure Files est en disponibilité générale tandis que la prise en charge du protocole NFS pour Azure Files est actuellement en préversion. Pour plus d'informations, consultez [La prise en charge de NFS 4.1 pour Azure Files est désormais disponible en préversion](https://azure.microsoft.com/en-us/blog/nfs-41-support-for-azure-files-is-now-in-preview/).

- Base de données CMS et Audit
  
  SAP BOBI Platform doit disposer d'une base de données pour stocker ses données système, appelées base de données CMS. Celle-ci permet de stocker les informations de BI Platform, comme les détails relatifs à l'utilisateur, au serveur, au dossier, au document, à la configuration et à l'authentification.

  Azure propose une offre DBaaS (Base de données en tant que service) [MySQL Database](https://azure.microsoft.com/en-us/services/mysql/) et [Azure SQL Database](https://azure.microsoft.com/en-us/services/sql-database/) qui peut être utilisée pour la base de données CMS et la base de données Audit. Comme il s'agit d'une offre PaaS, le client n'a pas à se soucier de l'exploitation, de la disponibilité et de la maintenance des bases de données. En fonction de ses besoins, le client peut également choisir sa propre base de données pour le référentiel CMS et Audit.

## <a name="support-matrix"></a>Matrice de prise en charge

Cette section décrit la prise en charge de différents composants SAP BOBI tels que la version de SAP BusinessObjects BI Platform, le système d'exploitation et les bases de données d'Azure.

### <a name="sap-businessobjects-bi-platform"></a>SAP BusinessObjects BI Platform

Le service IaaS (Infrastructure as a Service) Azure vous permet de déployer et de configurer SAP BusinessObjects BI Platform sur Azure Compute. Les versions suivantes de SAP BOBI Platform sont prises en charge :

- SAP BusinessObjects BI Platform 4.3
- SAP BusinessObjects BI Platform 4.2 SP04+
- SAP BusinessObjects BI Platform 4.1 SP05+

SAP BI Platform s'exécute sur différents systèmes d'exploitation et bases de données. La prise en charge de SAP BOBI Platform entre le système d'exploitation et la version de la base de données est présentée dans le [Tableau de disponibilité des produits](https://apps.support.sap.com/sap/support/pam) (SAP BOBI).

### <a name="operating-system"></a>Système d’exploitation

Azure prend en charge les systèmes d'exploitation suivants pour le déploiement de SAP BusinessObjects BI Platform.

- Microsoft Windows Server
- SUSE Linux Enterprise Server (SLES)
- Red Hat Enterprise Linux (RHEL)
- Oracle Linux (OL)

La version du système d'exploitation répertoriée dans le [Tableau de disponibilité des produits - SAP BusinessObjects BI Platform](https://support.sap.com/pam) est prise en charge tant qu'elle est compatible avec une exécution sur l'infrastructure Azure.

### <a name="databases"></a>Bases de données

BI Platform doit disposer d'une base de données pour le système de gestion de contenu (CMS) et le magasin de données d'audit ; celle-ci peut être installée sur toutes les bases de données prises en charge répertoriées dans le [Tableau de disponibilité des produits SAP](https://support.sap.com/pam), qui comprend ce qui suit :

- Microsoft SQL Server

- [Azure SQL Database](https://azure.microsoft.com/en-us/services/sql-database/) (base de données prise en charge pour SAP BOBI Platform sous Windows uniquement)

  Il s'agit d'un moteur de base de données SQL Server complètement managé, qui s'appuie sur la dernière édition Entreprise stable de SQL Server. Azure SQL Database prend en charge la plupart des fonctions de gestion de base de données, telles que la mise à niveau, la mise à jour corrective et la surveillance sans intervention de l'utilisateur. Avec Azure SQL Database, vous pouvez créer une couche de stockage de données hautement disponible et très performante pour les applications et les solutions dans Azure. Pour plus d'informations, consultez la documentation [Azure SQL Database](../../../azure-sql/azure-sql-iaas-vs-paas-what-is-overview.md).

- [Azure Database pour MySQL](https://azure.microsoft.com/en-us/services/mysql/) (Suivez les mêmes instructions de compatibilité que celles mentionnées pour MySQL AB dans le Tableau de disponibilité des produits SAP)

  Il s'agit d'un service de base de données relationnelle alimenté par MySQL Community Edition. En sa qualité d'offre DBaaS (Base de données en tant que service) complètement managée, ce service peut gérer des charges de travail critiques avec des performances prévisibles et une scalabilité dynamique. Il intègre la haute disponibilité, les sauvegardes automatiques, les mises à jour correctives des logiciels, la détection automatique des défaillances et la récupération jusqu'à 35 jours dans le passé, ce qui réduit considérablement les tâches opérationnelles. Pour plus d'informations, consultez la documentation d'[Azure Database pour MySQL](../../../mysql/overview.md).

- SAP HANA

- SAP ASE

- IBM DB2

- Oracle (pour plus d'informations sur les versions et restrictions, consultez la Note SAP [2039619](https://launchpad.support.sap.com/#/notes/2039619))

- MaxDB

Ce document illustre les instructions de déploiement de **SAP BOBI Platform sous Windows avec Azure SQL Database** et de **SAP BOBI Platform sous Linux avec Azure Database pour MySQL**. Il s'agit également de l'approche recommandée pour exécuter SAP BusinessObjects BI Platform sur Azure.

## <a name="sizing"></a>Dimensionnement

Le dimensionnement est un processus qui consiste à déterminer la configuration matérielle requise pour exécuter efficacement l'application. Pour SAP BOBI Platform, le dimensionnement doit être effectué à l'aide de l'outil de dimensionnement SAP [Quick Sizer](https://www.sap.com/about/benchmark/sizing.quick-sizer.html#quick-sizer). Cet outil fournit le nombre de SAPS en fonction de l'entrée, ce qui requiert ensuite un mappage avec les types de machines virtuelles Azure certifiés pour SAP. La Note SAP [1928533](https://launchpad.support.sap.com/#/notes/1928533) fournit la liste des produits SAP et des types de machines virtuelles Azure (avec les SAPS) pris en charge. Pour plus d'informations sur le dimensionnement, consultez le [Guide de dimensionnement SAP BI](https://wiki.scn.sap.com/wiki/display/BOBJ/Sizing+and+Deploying+SAP+BusinessObjects+BI+4.x+Platform+and+Add-Ons).

Pour satisfaire les besoins en stockage de SAP BOBI Platform, Azure propose différents types de [Disques managés](../../managed-disks-overview.md). Pour le répertoire d'installation de SAP BOBI, utilisez de préférence un disque managé premium, et pour la base de données exécutée sur les machines virtuelles, suivez les instructions fournies dans [Déploiement de SGBD pour une charge de travail SAP](dbms_guide_general.md).

Azure prend en charge deux offres DBaaS pour la couche Données de SAP BOBI Platform : [Azure SQL Database](https://azure.microsoft.com/en-us/services/sql-database) (application BI exécutée sous Windows) et [Azure Database pour MySQL](https://azure.microsoft.com/en-us/services/mysql) (application BI exécutée sous Linux et Windows). Ainsi, en fonction du résultat du dimensionnement, vous pouvez choisir le modèle d'achat qui correspond le mieux à vos besoins.

> [!Tip]
> Pour bénéficier d'une référence de dimensionnement rapide, sachez que 800 SAPS = 1 processeur virtuel lors du mappage du résultat en SAPS de la couche Base de données de SAP BOBI Platform avec Azure Database-as-a-Service (Azure SQL Database ou Azure Database pour MySQL).

### <a name="sizing-models-for-azure-sql-database"></a>Dimensionnement des modèles pour Azure SQL Database

Azure SQL Database propose les trois modèles d'achat suivants :

- Modèle vCore

  Ce modèle vous permet de choisir le nombre de vCores, la quantité de mémoire et de stockage, ainsi que la vitesse de stockage. Le modèle d’achat vCore vous permet également d’utiliser [Azure Hybrid Benefit pour SQL Server](https://azure.microsoft.com/pricing/hybrid-benefit/) afin de réduire les coûts. Ce modèle convient aux clients qui donnent la priorité à la flexibilité, au contrôle et à la transparence.

  Les trois [options de niveau de service](../../../azure-sql/database/service-tiers-vcore.md#service-tiers) proposées avec le modèle vCore sont les suivantes : Usage général, Critique pour l'entreprise et Hyperscale. Le niveau de service définit l'architecture de stockage, l'espace, les limites d'E/S et les options de continuité d'activité liées à la disponibilité et à la récupération d'urgence. Voici une description détaillée de chaque option de niveau de service :

  1. Le niveau de service **Usage général** est idéal pour les charges de travail d'entreprise. Il propose des options de calcul et de stockage équilibrées, évolutives et économiques. Pour plus d'informations, reportez-vous à [Options et limites des ressources](../../../azure-sql/database/resource-limits-vcore-single-databases.md#general-purpose---provisioned-compute---gen5).
  2. Le niveau de service **Critique pour l'entreprise** offre aux applications métier la meilleure résilience aux défaillances, en utilisant plusieurs réplicas isolés, ainsi que les meilleures performances d'E/S par réplica de base de données. Pour plus d'informations, reportez-vous à [Options et limites des ressources](../../../azure-sql/database/resource-limits-vcore-single-databases.md#business-critical---provisioned-compute---gen5).
  3. Le niveau de service **Hyperscale** convient aux charges de travail métier dont les besoins en termes de stockage et d'échelle lecture sont très évolutifs. Il offre une meilleure résilience aux défaillances en permettant la configuration de plusieurs réplicas de base de données isolés. Pour plus d'informations, reportez-vous à [Options et limites des ressources](../../../azure-sql/database/resource-limits-vcore-single-databases.md#hyperscale---provisioned-compute---gen5).

- Modèle DTU

  Le modèle d'achat DTU offre une combinaison de ressources de calcul, de mémoire et d'E/S réparties sur trois niveaux de service pour prendre en charge les charges de travail de base de données, aussi bien légères qu'importantes. Les tailles de calcul de chaque niveau fournissent une combinaison différente de ces ressources, auxquelles vous pouvez ajouter d’autres ressources de stockage. Ce modèle est idéal pour les clients qui souhaitent des options de ressources préconfigurées et simples.

  Les [niveaux de service](../../../azure-sql/database/service-tiers-dtu.md#compare-the-dtu-based-service-tiers) du modèle d'achat DTU se distinguent par une gamme de tailles de calcul, avec un volume fixe de stockage inclus, une période de conservation fixe des sauvegardes, ainsi qu'un prix fixe.

- Sans serveur

  Le modèle serverless met automatiquement à l’échelle les ressources de calcul en fonction de la demande de charge de travail et facture la quantité de ressources de calcul utilisée par seconde. Le niveau de calcul serverless met automatiquement en pause les bases de données pendant les périodes d'inactivité, lorsque seul le stockage est facturé, et relance automatiquement leur exécution lorsque l'activité reprend. Pour plus d'informations, reportez-vous à [Options et limites des ressources](../../../azure-sql/database/resource-limits-vcore-single-databases.md#general-purpose---serverless-compute---gen5).
  
  Ce modèle convient à une utilisation intermittente et imprévisible, avec une faible utilisation moyenne du calcul au fil du temps. Il peut donc être utilisé pour un déploiement hors production de SAP BOBI.

> [!Note]
> Pour SAP BOBI, le plus pratique consiste à utiliser le modèle basé sur vCore et de choisir le niveau de service Usage général ou Critique pour l'entreprise en fonction des besoins de l'entreprise.

### <a name="sizing-models-for-azure-database-for-mysql"></a>Modèles de dimensionnement pour Azure Database pour MySQL

Azure Database pour MySQL est fourni avec trois niveaux tarifaires différents. Ceux-ci diffèrent par la quantité de calcul dans les vCores, la mémoire par vCore et la technologie de stockage utilisée pour stocker les données. Vous trouverez ci-dessous une description détaillée des options. Pour en savoir plus sur les différents attributs, consultez la page [Niveau tarifaire](../../../mysql/concepts-pricing-tiers.md) d'Azure Database pour MySQL.

- De base

  Utilisé pour les charges de travail cibles qui nécessitent des performances légères en termes de calcul et d'E/S.

- Usage général

  Convient à la plupart des charges de travail d'entreprise qui nécessitent une capacité de calcul et de mémoire équilibrée avec un débit d'E/S évolutif.

- Mémoire optimisée

  Convient aux charges de travail de base de données haute performance qui nécessitent des performances en mémoire suffisantes pour un traitement plus rapide des transactions et une meilleure simultanéité.

> [!Note]
> Pour SAP BOBI, le plus pratique consiste à utiliser le niveau tarifaire Usage général ou À mémoire optimisée en fonction de la charge de travail de l'entreprise.

## <a name="azure-resources"></a>Ressources Azure

### <a name="choosing-regions"></a>Choix des régions

La région Azure est un centre de données (ou un ensemble de centres de données) qui contient l'infrastructure permettant d'exécuter et d'héberger différents services Azure. Cette infrastructure comprend un grand nombre de nœuds qui font office de nœuds de calcul ou de nœuds de stockage, ou qui exécutent des fonctionnalités réseau. Toutes les régions n'offrent pas les mêmes services.

SAP BI Platform contient différents composants qui peuvent nécessiter des types de machine virtuelle spécifiques, un stockage tel qu'Azure Files ou Azure NetApp Files, ou DBaaS (Base de données en tant que service) pour sa couche Données, mais ces composants peuvent ne pas être disponibles dans certaines régions. Vous trouverez des informations précises sur les types de machine virtuelle, les types de stockage Azure ou d'autres services Azure sur le site [Disponibilité des produits par région](https://azure.microsoft.com/en-us/global-infrastructure/services/). Si vous exécutez déjà vos systèmes SAP sur Azure, votre région est probablement identifiée. Dans ce cas, vous devez d'abord vérifier que les services nécessaires sont disponibles dans ces régions pour décider de l'architecture de SAP BI Platform.

### <a name="availability-zones"></a>Zones de disponibilité

Les zones de disponibilité sont des emplacements physiquement séparés au sein d’une région Azure. Chaque zone de disponibilité est constituée d'un ou de plusieurs centres de données équipés d'une alimentation, d'un refroidissement et d'un réseau indépendants.

Pour bénéficier de la haute disponibilité sur chacune des couches de SAP BI Platform, vous pouvez répartir les machines virtuelles de la zone de disponibilité en implémentant un framework de haute disponibilité, ce qui peut fournir le meilleur contrat SLA d'Azure. Pour plus d'informations sur le contrat SLA des machines virtuelles d'Azure, consultez la dernière version de [Contrats SLA des machines virtuelles](https://azure.microsoft.com/support/legal/sla/virtual-machines/).

Pour la couche Données, le service Azure Database as a Service (DBaaS) fournit un framework de haute disponibilité par défaut. Il vous suffit de sélectionner la région et les fonctionnalités de haute disponibilité, de redondance et de résilience inhérentes au service pour réduire les temps d'arrêt de la base de données dus aux interruptions planifiées et non planifiées, sans avoir à configurer de composants supplémentaires. Pour plus d'informations sur le contrat SLA de l'offre DBaaS prise en charge sur Azure, consultez [Haute disponibilité dans Azure Database pour MySQL](../../../mysql/concepts-high-availability.md) et [Haute disponibilité pour Azure SQL Database](../../../azure-sql/database/high-availability-sla.md).

### <a name="availability-sets"></a>Groupes à haute disponibilité

Un groupe à haute disponibilité est une fonctionnalité de regroupement logique qui permet d'isoler les ressources de machine virtuelle les unes des autres lors de leur déploiement. Azure veille à ce que les machines virtuelles que vous placez dans un groupe à haute disponibilité s'exécutent sur plusieurs serveurs physiques, racks de calcul, unités de stockage et commutateurs réseau. Si une défaillance matérielle ou logicielle se produit, seul un sous-ensemble de vos machines virtuelles est affecté et votre solution globale reste opérationnelle. Ainsi, lorsque des machines virtuelles sont placées dans des groupes à haute disponibilité, le contrôleur de structure Microsoft Azure répartit les machines virtuelles sur différents domaines d'[erreur](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/virtual-machines/workloads/sap/planning-guide.md#fault-domains) et de [mise à niveau](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/virtual-machines/workloads/sap/planning-guide.md#upgrade-domains) pour éviter que toutes les machines virtuelles ne deviennent inaccessibles en cas de maintenance de l'infrastructure ou de défaillance dans un domaine d'erreur.

SAP BI Platform contient de nombreux composants différents et, lors de la conception de l'architecture, vous devez vous assurer que chacun de ces composants résistera à d'éventuelles interruptions. Cela peut être réalisé en plaçant les machines virtuelles Azure de chaque composant dans des groupes à haute disponibilité. Gardez à l'esprit que lorsque vous combinez des machines virtuelles de différentes familles au sein d'un même groupe à haute disponibilité, vous pouvez rencontrer des problèmes qui vous empêchent d'inclure un certain type de machine virtuelle dans ce groupe. Concernant SAP BI Platform, il convient donc de définir un groupe à haute disponibilité distinct pour l'application web et l'application BI, comme indiqué dans Présentation de l'architecture.

De plus, le nombre de domaines de mise à jour et d'erreur qui peuvent être utilisés par un groupe à haute disponibilité Azure au sein d'une unité d'échelle Azure est limité. Ainsi, si vous continuez d'ajouter des machines virtuelles à un groupe à haute disponibilité, deux ou plusieurs machines virtuelles finiront par se retrouver dans le même domaine d'erreur ou de mise à jour. Pour plus d'informations, consultez la section [Groupes à haute disponibilité Azure](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/virtual-machines/workloads/sap/planning-guide.md#azure-availability-sets) du document Planification et implémentation de machines virtuelles Azure pour SAP.

Pour comprendre le concept des groupes à haute disponibilité Azure et leur relation avec les domaines d'erreur et de mise à niveau, consultez l'article [Gérer la disponibilité](../../manage-availability.md).

> [!Important]
> Les concepts de Zones de disponibilité Azure et de groupes à haute disponibilité Azure sont incompatibles. Cela signifie que vous pouvez déployer deux ou plusieurs machines virtuelles dans une zone de disponibilité ou dans un groupe à haute disponibilité Azure, mais pas dans les deux.

### <a name="virtual-machines"></a>Machines virtuelles

Machine virtuelle Azure est une offre de service qui vous permet de déployer des images personnalisées sur Azure en tant qu'instances IaaS (Infrastructure-as-a-Service). Elle simplifie la gestion et l’exploitation des applications en fournissant le calcul et le stockage à la demande pour héberger, mettre à l’échelle et gérer une application web et les applications connectées.

Azure propose un large éventail de machines virtuelles répondant à tous vos besoins en matière d'applications. Mais en termes de charge de travail SAP, Azure a restreint la sélection à différentes familles de machines virtuelles qui conviennent plus particulièrement à la charge de travail SAP et à la charge de travail SAP HANA. Pour plus d'informations, consultez [Logiciels SAP dont le déploiement est pris en charge sur Azure](sap-supported-product-on-azure.md).

En fonction du dimensionnement de SAP BI Platform, vous devez mapper vos besoins avec Machine virtuelle Azure, qui est prise en charge dans le produit Azure pour SAP. La Note SAP [1928533](https://launchpad.support.sap.com/#/notes/1928533) est un bon point de départ pour répertorier les types de machines virtuelles Azure pris en charge pour les produits SAP sous Windows et Linux. Gardez également à l'esprit qu'outre la sélection de types de machines virtuelles strictement pris en charge, vous devez également vérifier si ceux-ci sont disponibles dans une région donnée. Vous pouvez vérifier la disponibilité d'un type de machine virtuelle sur la page [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/). Pour choisir le modèle de tarification, vous pouvez consulter [Machines virtuelles Azure pour charge de travail SAP](planning-guide.md#azure-virtual-machines-for-sap-workload).

### <a name="storage"></a>Stockage

Stockage Azure est un service cloud géré par Azure qui fournit un stockage hautement disponible, sécurisé, durable, évolutif et redondant. L'usage de certains types de stockage est limité dans le cadre des scénarios SAP. Mais certains types de stockage Azure sont adaptés ou optimisés pour des scénarios de charge de travail spécifiques à SAP. Pour plus d'informations, consultez le guide [Types de stockage Azure pour charge de travail SAP](planning-guide-storage.md), car il met en évidence les différentes options de stockage adaptées à SAP.

Stockage Azure propose aux clients différents types de stockage, présentés en détails dans l'article [Quels sont les types de disque disponibles dans Azure ?](../../disks-types.md). SAP BOBI Platform utilise le stockage Azure suivant pour créer l'application :

- Disques managés Azure

  Il s'agit d'un volume de stockage de niveau bloc qui est géré par Azure. Vous pouvez utiliser les disques pour les serveurs d'applications et les bases de données de SAP BOBI Platform, lorsqu'ils sont installés sur des machines virtuelles Azure. Différents types de [Disques managés Azure](../../managed-disks-overview.md) sont disponibles, mais il est recommandé d'utiliser des [disques SSD Premium](../../disks-types.md#premium-ssd) pour l'application et la base de données SAP BOBI Platform.

  Dans l'exemple ci-dessous, des disques SSD Premium sont utilisés pour le répertoire d'installation de BOBI Platform. Concernant les bases de données installées sur une machine virtuelle, vous pouvez utiliser des disques managés pour les données et le volume du fichier journal, conformément aux instructions. Les bases de données CMS et Audit sont généralement de petite taille et ne présentent donc pas les mêmes exigences que les autres bases de données SAP OLTP/OLAP en termes de performances de stockage.

- Azure Premium Files ou Azure NetApp Files

  Dans SAP BOBI Platform, File Repository Server (FRS) fait référence aux répertoires de disque où sont stockés des contenus tels que les rapports, les univers et les connexions utilisés par tous les serveurs d'applications du système. Un stockage [Azure Premium Files](../../../storage/files/storage-files-introduction.md) ou [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) peut être utilisé comme système de fichiers partagé pour les applications SAP BOBI FRS. Cette offre de stockage n'étant pas disponible dans certaines régions, consultez le site [Disponibilité des produits par région](https://azure.microsoft.com/en-us/global-infrastructure/services/) pour bénéficier d'informations à jour.

  Si le service n'est pas disponible dans votre région, vous pouvez créer un serveur NFS à partir duquel vous pourrez partager le système de fichiers avec l'application SAP BOBI. Mais vous devrez également tenir compte de sa haute disponibilité.

![Disposition de stockage de SAP BusinessObjects BI Platform sur Azure](media/businessobjects-deployment-guide/businessobjects-storage-layout.png)

### <a name="networking"></a>Mise en réseau

SAP BOBI est une plateforme BI de création de rapports et d'analyse qui ne contient aucune donnée métier. Le système est donc connecté à d'autres serveurs de bases de données à partir desquels il récupère toutes les données et fournit des insights aux utilisateurs. Azure fournit une infrastructure réseau, qui permet le mappage de tous les scénarios pouvant être réalisés avec SAP BI Platform, comme la connexion au système local, aux systèmes de différents réseaux virtuels et autres. Pour plus d'informations, consultez [Mise en réseau Microsoft Azure pour charge de travail SAP](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/virtual-machines/workloads/sap/planning-guide.md#microsoft-azure-networking).

Pour l'offre DBaaS (Base de données en tant que service), toute base de données nouvellement créée (Azure SQL Database ou Azure Database pour MySQL) dispose d'un pare-feu qui bloque toutes les connexions externes. Pour autoriser l'accès au service DBaaS à partir des machines virtuelles de BI Platform, vous devez spécifier une ou plusieurs règles de pare-feu au niveau du serveur. Pour plus d'informations, consultez la section [Règles de pare-feu](../../../mysql/concepts-firewall-rules.md) pour Azure Database pour MySQL et la section [Contrôles d'accès réseau](../../../azure-sql/database/network-access-controls-overview.md) pour Azure SQL Database.

## <a name="next-steps"></a>Étapes suivantes

- [Déploiement de SAP BusinessObjects BI Platform sous Linux](businessobjects-deployment-guide-linux.md)
- [Planification et implémentation de machines virtuelles Azure pour SAP](planning-guide.md)
- [Déploiement de machines virtuelles Azure pour SAP](deployment-guide.md)
- [Déploiement SGBD de machines virtuelles Azure pour SAP](./dbms_guide_general.md)