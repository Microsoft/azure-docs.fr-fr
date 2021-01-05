---
title: Qu’est-ce que le service Azure SQL Database ?
description: 'Obtenir une introduction à SQL Database : détails techniques et fonctionnalités du système de gestion de base de données relationnelle (SGBDR) Microsoft dans le cloud.'
keywords: introduction à sql, intro à sql, qu’est-ce qu’une base de données sql
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: sqldbrb=3
ms.devlang: ''
ms.topic: overview
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 09/21/2020
ms.openlocfilehash: 0bd6300f4b9dbcf76b5447a0fb58502b7aebf311
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96451415"
---
# <a name="what-is-azure-sql-database"></a>Qu’est-ce qu’Azure SQL Database ?
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Azure SQL Database est un moteur de base de données PaaS (Platform as a Service) complètement managé qui prend en charge la plupart des fonctions de gestion de base de données telles que la mise à niveau, la mise à jour corrective, les sauvegardes et la surveillance sans intervention de l’utilisateur. Azure SQL Database s’exécute toujours sur la dernière version stable du moteur de base de données SQL Server et un système d’exploitation corrigé offrant une disponibilité de 99,99 %. Les fonctionnalités PaaS intégrées dans Azure SQL Database vous permettent de vous concentrer sur les activités d’administration et d’optimisation de base de données spécifiques du domaine, qui sont critiques pour votre activité.

Avec Azure SQL Database, vous pouvez créer une couche de stockage de données hautement disponible et très performante pour les applications et les solutions dans Azure. SQL Database peut être le meilleur choix pour diverses applications cloud modernes car il vous permet de traiter à la fois des données relationnelles et des [structures non relationnelles](../multi-model-features.md), comme des graphiques, des données JSON, des données spatiales et des données XML.

Azure SQL Database repose sur la dernière version stable du [moteur de base de données Microsoft SQL Server](/sql/sql-server/sql-server-technical-documentation?toc=%2fazure%2fsql-database%2ftoc.json). Vous pouvez utiliser des fonctionnalités avancées de traitement des requêtes, comme les [technologies en mémoire hautes performances](../in-memory-oltp-overview.md) et le [traitement intelligent des requêtes](/sql/relational-databases/performance/intelligent-query-processing?toc=%2fazure%2fsql-database%2ftoc.json). En fait, les fonctionnalités les plus récentes de SQL Server sont publiées tout d’abord dans SQL Database, puis dans SQL Server proprement dit. Vous obtenez les fonctionnalités SQL Server les plus récentes sans frais d’application de correctifs ou de mise à niveau, testées sur des millions de bases de données. 

SQL Database vous permet de définir et de mettre à l'échelle facilement les performances dans deux modèles d’achat différents : un [modèle d’achat vCore](service-tiers-vcore.md) et un [modèle d’achat DTU](service-tiers-dtu.md). SQL Database est un service complètement managé qui intègre une haute disponibilité, des sauvegardes et d’autres opérations de maintenance courantes. Microsoft gère toutes les applications de correctifs et mises à jour du code SQL et du code du système d’exploitation. Vous n’avez pas à vous soucier de la gestion de l’infrastructure sous-jacente.

Si vous ne connaissez pas Azure SQL Database, regardez la vidéo *Vue d’ensemble d’Azure SQL Database*, qui fait partie de notre [série de vidéos Azure SQL](https://channel9.msdn.com/Series/Azure-SQL-for-Beginners?WT.mc_id=azuresql4beg_azuresql-ch9-niner) :
> [!VIDEO https://channel9.msdn.com/Series/Azure-SQL-for-Beginners/Azure-SQL-Database-Overview-7-of-61/player]

## <a name="deployment-models"></a>Modèles de déploiement

Azure SQL Database fournit les options de déploiement suivantes pour une base de données :

- [Base de données unique](single-database-overview.md) représente une base de données isolée complètement managée. Vous pouvez utiliser cette option si vous avez des applications cloud modernes et des microservices nécessitant une source de données unique et fiable. Une base de données unique est similaire à une [base de données autonome](/sql/relational-databases/databases/contained-databases?toc=%2fazure%2fsql-database%2ftoc.json) dans le [moteur de base de données SQL Server](/sql/sql-server/sql-server-technical-documentation?toc=%2fazure%2fsql-database%2ftoc.json).
- [Pool élastique](elastic-pool-overview.md) représente une collection de [bases de données uniques](single-database-overview.md) avec un ensemble partagé de ressources telles que le processeur ou la mémoire. Les bases de données uniques peuvent être déplacées dans et hors d’un pool élastique.

> [!IMPORTANT]
> Pour comprendre les différences de fonctionnalités entre SQL Database et SQL Server, ainsi que les différences entre les diverses options d’Azure SQL Database, consultez [Fonctionnalités SQL Database](features-comparison.md).

SQL Database offre des performances prévisibles avec plusieurs types de ressources, plusieurs niveaux de service et plusieurs tailles de calcul. Il fournit une scalabilité dynamique sans temps d’arrêt, une optimisation intelligente intégrée, une scalabilité et une disponibilité à l’échelle mondiale, ainsi que des options de sécurité avancées. Ces fonctionnalités vous permettent de vous concentrer sur le développement rapide de vos applications et sur la façon de raccourcir le délai de commercialisation, plutôt que sur la gestion des machines virtuelles et de l’infrastructure. Le service SQL Database est actuellement présent dans 38 centres de données à travers le monde. Vous pouvez donc exécuter votre base de données dans un centre de données près de chez vous.

## <a name="scalable-performance-and-pools"></a>Pools et performances évolutives

Vous pouvez définir la quantité de ressources affectées. 
- Avec les bases de données uniques, chaque base de données est isolée des autres et est portable. Chacune possède sa propre quantité garantie de ressources de calcul, de mémoire et de stockage. La quantité de ressources assignées à la base de données est dédiée à cette base de données et n’est pas partagée avec d’autres bases de données dans Azure. Vous pouvez [effectuer un scale-up et un scale-down des ressources d’une base de données unique](single-database-scale.md) de manière dynamique. L’option de base de données unique fournit différentes ressources de calcul, de mémoire et de stockage pour différents besoins. Par exemple, vous pouvez obtenir de 1 à 80 vCores, ou de 32 Go à 4 To. Le [niveau de service hyperscale](service-tier-hyperscale.md) pour une base de données unique vous permet de mettre à l’échelle jusqu’à 100 To, avec des fonctionnalités de sauvegarde et de restauration rapides.
- Avec les pools élastiques, vous pouvez affecter des ressources qui sont partagées par toutes les bases de données du pool. Vous pouvez créer une base de données ou déplacer des bases de données uniques existantes vers un pool de ressources pour optimiser l’utilisation des ressources et réaliser des économies. Cette option vous permet également d’[effectuer un scale-up et un scale-down de ressources de pool élastique](elastic-pool-scale.md) de façon dynamique.

Vous pouvez créer votre première application sur une petite base de données unique pour un coût mensuel modique dans le niveau de service à usage général. Vous pouvez ensuite changer son niveau de service manuellement ou par programmation à tout moment pour définir le niveau de service Critique pour l’entreprise afin de répondre aux besoins de votre solution. Vous pouvez ajuster les performances sans perturber le fonctionnement de votre application, ni l’expérience de vos clients. L’évolutivité dynamique permet de répondre en toute transparence à l’évolution rapide des besoins en ressources de votre base de données. Vous payez uniquement pour les ressources dont vous avez besoin, quand vous en avez besoin.

La *scalabilité dynamique* est différente de la *mise à l’échelle automatique*. La mise à l’échelle survient lorsqu’un service se met à l’échelle automatiquement en fonction de critères, tandis que l’extensibilité dynamique permet la mise à l’échelle manuelle sans temps d’arrêt. L’option de base de données unique prend en charge la scalabilité dynamique manuelle, mais pas la mise à l’échelle automatique. Pour plus expérience plus automatique, envisagez d’utiliser des pools élastiques, ce qui permet aux bases de données de partager des ressources dans un pool en fonction de leurs besoins individuels. L’autre possibilité consiste à utiliser des scripts qui peuvent aider à automatiser la scalabilité pour une base de données unique. Pour obtenir un exemple, consultez [Utiliser PowerShell pour surveiller et mettre à l’échelle une base de données SQL](scripts/monitor-and-scale-database-powershell.md).

### <a name="purchasing-models"></a>Modèles d’achat

SQL Database propose les modèles d’achat suivants :
- Le [modèle d’achat vCore](service-tiers-vcore.md) vous permet de choisir le nombre de vCores, la quantité de mémoire et de stockage, ainsi que la vitesse de stockage. Le modèle d’achat vCore vous permet également d’utiliser [Azure Hybrid Benefit pour SQL Server](https://azure.microsoft.com/pricing/hybrid-benefit/) afin de réduire les coûts. Pour en savoir plus sur Azure Hybrid Benefit, consultez la section « Questions fréquentes (FAQ) », plus loin dans cet article.
- Le [modèle d’achat DTU](service-tiers-dtu.md) offre une combinaison de ressources de calcul, de mémoire et d’E/S réparties sur trois niveaux de service pour prendre en charge les charges de travail de base de données, aussi bien légères qu’importantes. Les tailles de calcul de chaque niveau fournissent une combinaison différente de ces ressources, auxquelles vous pouvez ajouter d’autres ressources de stockage.
- Le [modèle serverless](serverless-tier-overview.md) met automatiquement à l’échelle les ressources de calcul en fonction de la demande de charge de travail et facture la quantité de ressources de calcul utilisée par seconde. Le niveau de calcul serverless met aussi automatiquement en pause les bases de données pendant les périodes d’inactivité, quand seul le stockage est facturé, et reprend leur exécution automatiquement avec l’activité.

### <a name="service-tiers"></a>Niveaux de service

Azure SQL Database offre trois niveaux de services conçus pour différents types d'applications :
- [Usage général/Standard](service-tier-general-purpose.md) : adapté aux charges de travail courantes. Il offre des options de calcul et de stockage équilibrées et économiques.
- [Critique pour l’entreprise/Premium](service-tier-business-critical.md) : adapté aux applications OLTP avec des débits de transactions élevés et la latence des E/S la plus faible. Il offre la meilleure résilience aux défaillances grâce à l’utilisation de plusieurs réplicas isolés.
- [Hyperscale](service-tier-hyperscale.md) : adapté aux bases de données OLTP très volumineuse, avec une capacité de mise à l’échelle automatique du stockage et de mise à l’échelle des ressources de calcul de façon fluide.    

### <a name="elastic-pools-to-maximize-resource-utilization"></a>Pools élastiques pour optimiser l’utilisation des ressources

Pour de nombreuses entreprises et applications, la possibilité de créer des bases de données uniques et d’augmenter ou de ralentir les performances à la demande se révèle suffisante, surtout si les modèles d’utilisation sont relativement prévisibles. Des modèles d’utilisation imprévisibles peuvent compliquer la gestion des coûts et de votre modèle commercial. [Les pools élastiques](elastic-pool-overview.md) sont conçus pour résoudre ce problème. Vous allouez des ressources de performances à un pool plutôt qu’à une base de données individuelle. Vous payez pour les ressources de performances collectives du pool plutôt que pour les performances d’une base de données unique.

   ![Graphique illustrant les pools élastiques dans les éditions De base, Standard et Premium](./media/sql-database-paas-overview/sqldb_elastic_pools.png)

Les pools élastiques vous permettent de ne pas avoir à ajuster les performances de la base de données en fonction des besoins en ressources. Les bases de données mises en pool consomment les ressources de performance du pool élastique en fonction des besoins. Les bases de données mises en pool consomment mais ne dépassent pas les limites du pool. Vos coûts restent prévisibles, même si l’utilisation de la base de données individuelle ne l’est pas.

Vous pouvez [ajouter et supprimer des bases de données dans le pool](elastic-pool-overview.md), mettre à l’échelle votre application en passant de quelques bases de données à des milliers, le tout dans les limites d’un budget que vous contrôlez. Vous pouvez également contrôler le nombre minimal et maximal de ressources disponibles pour les bases de données dans le pool. Cela vous permet de garantir qu’aucune base de données mise en pool n’utilise toutes les ressources du pool et que chaque base de données mise en pool dispose d’un nombre minimal de ressources. Pour en savoir plus sur les modèles de conception pour les applications SaaS (Software as a Service) qui utilisent des pools élastiques, consultez [Modèles de conception pour les applications SaaS multi-locataires avec SQL Database](saas-tenancy-app-design-patterns.md).

Les scripts aident à la surveillance et à la mise à l’échelle des pools élastiques. Pour obtenir un exemple, consultez [Utiliser PowerShell pour superviser et mettre à l’échelle un pool élastique dans Azure SQL Database](scripts/monitor-and-scale-pool-powershell.md).


### <a name="blend-single-databases-with-pooled-databases"></a>Fusionner les bases de données uniques avec les bases de données mises en pool

Vous pouvez fusionner des bases de données uniques avec des pools élastiques, et changer les niveaux de service des bases de données uniques et des pools élastiques pour vous adapter à votre situation. Vous pouvez également combiner d’autres services Azure avec SQL Database afin de pouvoir répondre aux besoins uniques de vos applications en matière de conception, réduire les coûts, optimiser l’efficacité des ressources et créer de nouvelles opportunités professionnelles.

## <a name="extensive-monitoring-and-alerting-capabilities"></a>Fonctionnalités étendues de surveillance et d’alerte

Azure SQL Database offre des fonctionnalités avancées de supervision et de résolution des problèmes qui vous aident à obtenir des insights plus approfondis des caractéristiques de charge de travail. Ces outils et fonctionnalités sont notamment les suivants :
 - Les fonctionnalités de supervision intégrées fournies par la dernière version du moteur de base de données SQL Server. Elles vous permettent d’obtenir des insights sur les performances en temps réel. 
 - Les fonctionnalités de supervision PaaS fournies par Azure et qui vous permettent de superviser un grand nombre d’instances de base de données et de résoudre des problèmes les concernant.

Le [Magasin des requêtes](/sql/relational-databases/performance/best-practice-with-the-query-store), fonctionnalité de supervision SQL Server intégrée, consigne en temps réel les performances de vos requêtes et vous permet d’identifier les problèmes de performances potentiels et les principaux consommateurs de ressources. Le paramétrage automatique et les recommandations fournissent des conseils concernant les requêtes avec les performances régressées et les index manquants ou dupliqués. Dans SQL Database, le paramétrage automatique vous permet soit d’appliquer manuellement les scripts qui peuvent résoudre les problèmes, soit de laisser SQL Database appliquer le correctif. SQL Database peut également tester et vérifier si le correctif présente un avantage, et conserver ou annuler le changement en fonction du résultat. Outre les fonctionnalités du Magasin des requêtes et de paramétrage automatique, vous pouvez utiliser les composants [DMV et XEvent](monitoring-with-dmvs.md) standard pour superviser les performances des charges de travail.

Azure fournit des outils intégrés de [supervision](performance-guidance.md) et d’[alerte](alerts-insights-configure-portal.md) qui, combinés avec des évaluations des performances, vous permettent de superviser l’état de milliers de bases de données. Ces outils vous permettent d’évaluer rapidement l’impact d’un scale-up ou d’un scale-down en fonction de vos besoins actuels ou projetés en matière de performances. De plus, SQL Database peut [émettre des métriques et des journaux de ressources](metrics-diagnostic-telemetry-logging-streaming-export-configure.md) pour faciliter la supervision. Vous pouvez configurer SQL Database pour stocker l’utilisation des ressources, les employés et les sessions, ainsi que la connectivité dans une de ces ressources Azure :

- **Stockage Azure** : Pour archiver à petit prix de gros volumes de données de télémétrie.
- **Azure Event Hubs** : Pour intégrer des données de télémétrie SQL Database à votre solution de supervision personnalisée ou à vos pipelines chauds.
- **Journaux d’activité Azure Monitor** : Pour une solution de supervision intégrée offrant des fonctionnalités de génération de rapports, d’alerte et d’atténuation.

![Diagramme de l’architecture de supervision Azure](./media/sql-database-paas-overview/architecture.png)

## <a name="availability-capabilities"></a>Fonctionnalités de disponibilité

Azure SQL Database permet à votre entreprise de continuer à travailler pendant les interruptions. Dans un environnement SQL Server traditionnel, vous avez généralement au moins deux machines configurées localement. Ces machines disposent de copies exactes et gérées de façon synchrone des données afin de vous protéger contre la défaillance d’une machine ou d’un composant unique. Cet environnement assure une haute disponibilité, mais il ne protège pas contre une catastrophe naturelle détruisant votre centre de données.

La reprise d’activité après sinistre suppose qu’un événement catastrophique est suffisamment localisé géographiquement pour avoir une autre machine ou un autre ensemble de machines éloigné avec une copie de vos données. Dans SQL Server, vous pouvez utiliser des groupes de disponibilité Always On s’exécutant en mode asynchrone pour obtenir cette fonctionnalité. Souvent, les gens ne veulent pas attendre que la réplication se produise si loin avant de valider une transaction. Il existe donc un risque potentiel de perte de données quand vous effectuez des basculements non planifiés.

Les bases de données des niveaux de service Premium et Critique pour l’entreprise [effectuent déjà des actions proches](high-availability-sla.md#premium-and-business-critical-service-tier-locally-redundant-availability) de la synchronisation d’un groupe de disponibilité. Les bases de données des niveaux de service inférieurs assurent une redondance par le biais du stockage à l’aide d’un [mécanisme différent, mais équivalent](high-availability-sla.md#basic-standard-and-general-purpose-service-tier-locally-redundant-availability). Une logique intégrée permet de se protéger contre la défaillance d’une seule machine. La fonctionnalité de géoréplication active vous permet de protéger la machine contre les sinistres lorsqu’une région entière est détruite.

La fonctionnalité Zones de disponibilité Azure tente de protéger un centre de données unique contre les pannes au sein d’une même région. Elle assure la protection contre la perte d’alimentation ou de réseau dans un bâtiment. Dans SQL Database, vous placez les différents réplicas dans différentes zones de disponibilité (concrètement, dans différents bâtiments).

En fait, le contrat de niveau de service [(SLA)](https://azure.microsoft.com/support/legal/sla/) d’Azure, soutenu par un réseau mondial de centres de données gérés par Microsoft, permet d’exécuter votre application 24 heures sur 24, 7 jours sur 7. La plateforme Azure gère entièrement chaque base de données et garantit un pourcentage élevé de disponibilité des données, sans aucune perte. Azure gère automatiquement les tâches suivantes : mises à jour correctives, sauvegardes, réplication, détection des éventuelles défaillances matérielles, logicielles ou réseau sous-jacentes, déploiement de correctifs de bogues, basculements, mises à niveau de base de données et autres tâches de maintenance. Pour obtenir la disponibilité Standard, les couches de calcul et de stockage sont séparées. Pour obtenir la disponibilité Premium, les opérations de calcul et de stockage sont intégrées à un nœud unique afin d’améliorer les performances, puis une technologie similaire aux groupes de disponibilité Always On est implémentée. Pour accéder à une description complète des fonctionnalités haute disponibilité d’Azure SQL Database, consultez [Disponibilité de SQL Database](high-availability-sla.md). 

De plus, SQL Database fournit des fonctionnalités intégrées de [continuité d’activité et de scalabilité à l’échelle mondiale](business-continuity-high-availability-disaster-recover-hadr-overview.md). notamment :

- [Sauvegardes automatiques](automated-backups-overview.md) :

  SQL Database effectue automatiquement des sauvegardes de fichier journal complètes et différentielles des bases de données pour vous permettre de restaurer à n’importe quel point dans le temps. Pour les bases de données uniques et les bases de données mises en pool, vous pouvez configurer SQL Database de façon à stocker les sauvegardes de base de données complètes dans le Stockage Azure en vue d’une conservation des sauvegardes à long terme. Pour des instances gérées, vous pouvez également effectuer des sauvegardes de copie uniquement en vue d’une rétention à long terme.

- [Restaurations à un point dans le temps](recovery-using-backups.md) :

  Toutes les options de déploiement SQL Database prennent en charge la reprise d’activité à n’importe quel point dans le temps au cours de la période de conservation de sauvegarde automatique pour toute base de données.
- [Géoréplication active](active-geo-replication-overview.md) :

  Les options de base de données unique et de bases de données mises en pool vous permettent de configurer jusqu’à quatre bases de données secondaires accessibles en lecture, que ce soit dans le même centre de données ou dans des centres de données Azure distribués à l’échelle mondiale. Par exemple, si vous avez une application SaaS avec une base de données de catalogue qui a un volume élevé de transactions simultanées en lecture seule, utilisez la géo-réplication active pour permettre une mise à l’échelle en lecture globale. Cela supprime les goulots d’étranglement sur le principal qui sont dus à des charges de travail de lecture. Pour des instances gérées, utilisez des groupes de basculement automatique.
- [Groupes de basculement automatique](auto-failover-group-overview.md) :

  Toutes les options de déploiement SQL Database vous permettent d’utiliser des groupes de basculement pour activer la haute disponibilité et l’équilibrage de charge à l’échelle mondiale. Cela inclut la géo-réplication transparente ainsi que le basculement de grands ensembles de bases de données, de pools élastiques et d’instances managées. Les groupes de basculement permettent de créer des applications SaaS distribuées à l’échelle mondiale avec une surcharge administrative minimale. Cela laisse SQL Database se charger de toutes les tâches complexes de supervision, de routage et d’orchestration de basculement.
- [Bases de données redondantes interzone](high-availability-sla.md) :

  SQL Database vous permet d’approvisionner des bases de données ou des pools élastiques Premium ou critiques pour l’entreprise dans plusieurs zones de disponibilité. Les bases de données et les pools élastiques ont plusieurs réplicas redondants pour offrir une haute disponibilité. Placer ces réplicas dans plusieurs zones de disponibilité procure donc une plus haute résilience. Cela donne la possibilité d’une reprise d’activité automatique à partir des échecs de mise à l’échelle du centre de données sans perdre de données.

## <a name="built-in-intelligence"></a>Intelligence intégrée

Avec SQL Database, vous obtenez une intelligence intégrée qui permet de réduire considérablement les coûts d’exécution et de gestion des bases de données, et qui optimise les performances et la sécurité de votre application. Via l’exécution continue de millions de charges de travail client, SQL Database collecte et traite une quantité vertigineuse de données de télémétrie, tout en respectant également pleinement la confidentialité des clients. Différents algorithmes évaluent en permanence les données de télémétrie afin que le service puisse apprendre et s’adapter à votre application.

### <a name="automatic-performance-monitoring-and-tuning"></a>Surveillance et réglage des performances automatiques

SQL Database fournit des informations détaillées sur les requêtes que vous devez surveiller. SQL Database s’informe sur vos modèles de base de données et vous permet d’adapter votre schéma de base de données à votre charge de travail. SQL Database propose des [recommandations pour le réglage des performances](database-advisor-implement-performance-recommendations.md), où vous pouvez passer en revue les actions de réglage et les appliquer.

Toutefois, la surveillance permanente d’une base de données est une tâche difficile et fastidieuse, en particulier lorsque vous utilisez plusieurs bases de données. La fonctionnalité [Intelligent Insights](intelligent-insights-overview.md) effectue cette tâche pour vous en supervisant automatiquement les performances de SQL Database à grande échelle. Elle vous informe des problèmes de dégradation des performances, identifie la cause racine de chaque problème et fournit des recommandations pour l’amélioration des performances quand cela est possible.

La gestion d’un très grand nombre de bases de données peut s’avérer impossible même avec tous les rapports et outils disponibles avec SQL Database et Azure. Au lieu de superviser et de régler votre base de données manuellement, vous pouvez envisager de déléguer certaines actions de supervision et de réglage à SQL Database à l’aide du [paramétrage automatique](automatic-tuning-overview.md). SQL Database applique automatiquement les recommandations, puis teste et vérifie chacune de ses actions de réglage afin de garantir l’amélioration continue des performances. De cette manière, SQL Database s’adapte automatiquement à votre charge de travail de façon contrôlée et sécurisée. Le paramétrage automatique signifie que les performances de votre base de données sont soigneusement supervisées et comparées avant et après chaque action de paramétrage. Si les performances ne s’améliorent pas, l’action de paramétrage est annulée.

Un grand nombre de nos partenaires qui exécutent des [applications SaaS multi-locataires](saas-tenancy-app-design-patterns.md) sur SQL Database s’appuient sur le paramétrage automatique des performances pour garantir que leurs applications ont toujours des performances stables et prévisibles. Pour eux, cette fonctionnalité réduit considérablement le risque de voir survenir un incident lié aux performances au milieu de la nuit. En outre, comme une partie de leur clientèle utilise également SQL Server, ils utilisent les mêmes recommandations d’indexation que celles fournies par SQL Database pour aider leurs clients SQL Server.

Deux aspects de paramétrage automatique sont [disponibles dans SQL Database](automatic-tuning-overview.md) :

- **Gestion automatique des index** : identifie les index qui doivent être ajoutés à votre base de données et ceux qui doivent être supprimés.
- **Correction automatique du plan** : identifie les plans problématiques et résout les problèmes de performances du plan SQL.

### <a name="adaptive-query-processing"></a>Traitement de requêtes adaptatif

Vous pouvez utiliser le [traitement adaptatif des requêtes](/sql/relational-databases/performance/intelligent-query-processing),notamment l’exécution entrelacée pour les fonctions table à instructions multiples, les commentaires d’allocation de mémoire en mode batch et les jointures adaptatives en mode batch. Chacune de ces fonctionnalités de traitement adaptatif des requêtes applique des techniques similaires d’apprentissage et d’adaptation, ce qui contribue à résoudre les problèmes de performances liés aux soucis d’optimisation de requête historiquement intraitables.

## <a name="advanced-security-and-compliance"></a>Sécurité et conformité avancées

SQL Database fournit toute une gamme de [fonctionnalités intégrées de sécurité et conformité](../../active-directory/identity-protection/concept-identity-protection-security-overview.md) pour que votre application réponde aux différentes exigences de conformité et de sécurité.

> [!IMPORTANT]
> Microsoft a certifié Azure SQL Database (toutes les options de déploiement) selon plusieurs normes de conformité. Pour plus d’informations, consultez le [Centre de confidentialité Microsoft Azure](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942), qui inclut la liste la plus à jour des certifications de conformité de SQL Database.

### <a name="advance-threat-protection"></a>Protection avancée contre les menaces

Azure Defender pour SQL est un package unifié de fonctionnalités de sécurité SQL avancées. Il inclut des fonctions permettant de gérer les vulnérabilités de votre base de données et de détecter les activités anormales pouvant indiquer une menace ciblant votre base de données. Il vous permet d’activer et de gérer ces fonctionnalités à partir d’un seul et même emplacement.

- [Évaluation des vulnérabilités](sql-vulnerability-assessment.md) :

  Ce service découvre, suit et facilite la correction des vulnérabilités potentielles de base de données. Elle offre une visibilité sur votre état de sécurité et inclut des mesures pratiques pour résoudre les problèmes de sécurité et améliorer la protection de votre base de données.
- [Détection des menaces](threat-detection-configure.md) :

  cette fonctionnalité détecte les activités anormales qui indiquent des tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses de votre base de données. Elle surveille en permanence votre base de données pour détecter des activités suspectes et envoie immédiatement des alertes de sécurité en cas de vulnérabilités éventuelles, d’attaques par injection de code SQL et de modèles d’accès anormaux à la base de données. Les alertes générées par la fonctionnalité Détection des menaces fournissent des informations sur les activités suspectes et recommandent l’action à entreprendre pour analyser et atténuer la menace.

### <a name="auditing-for-compliance-and-security"></a>Audit de sécurité et de conformité

L’[audit](../../azure-sql/database/auditing-overview.md) suit les événements de base de données et les consigne dans un journal d’audit dans votre compte de stockage Azure. L’audit peut vous aider à vous conformer à la réglementation, à comprendre l’activité de la base de données, et à découvrir des discordances et des anomalies susceptibles d’indiquer des problèmes pour l’entreprise ou des violations de la sécurité.

### <a name="data-encryption"></a>Chiffrement des données

SQL Database contribue à sécuriser vos données par le biais du chiffrement. Pour les données en mouvement, il utilise le protocole [TLS (Transport Layer Security)](https://support.microsoft.com/kb/3135244). Pour les données au repos, il utilise [TDE, Transparent Data Encryption](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql). Pour les données en cours d’utilisation, il utilise la fonctionnalité [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine).

### <a name="data-discovery-and-classification"></a>Découverte et classification des données

[Découverte et classification des données](data-discovery-and-classification-overview.md) offre des fonctions avancées intégrées à Azure SQL Database pour la découverte, la classification, l’étiquetage et la protection des données sensibles dans vos bases de données. Elle offre une visibilité de l’état de classification de votre base de données et suit l’accès aux données sensibles dans la base de données et en dehors de celle-ci.

### <a name="azure-active-directory-integration-and-multi-factor-authentication"></a>Intégration d’Azure Active Directory et authentification multifacteur

SQL Database vous permet de gérer de manière centralisée les identités d’utilisateur de base de données et d’autres services Microsoft avec l’[intégration d’Azure Active Directory](authentication-aad-overview.md). Cette fonctionnalité simplifie la gestion des autorisations et améliore la sécurité. Azure Active Directory prend en charge l’[authentification multifacteur](authentication-mfa-ssms-overview.md) pour augmenter la sécurité des données et des applications, ainsi qu’un processus d’authentification unique.

## <a name="easy-to-use-tools"></a>Outils simples d’utilisation

SQL Database simplifie la création et la gestion des applications et vous fait gagner en productivité. SQL Database vous aide à vous concentrer sur ce que vous maîtrisez le mieux, à savoir le développement de formidables applications. Dans SQL Database, vous pouvez gérer et développer des applications en utilisant les outils et les compétences dont vous disposez déjà.

|Outil|Description|
|:---|:---|
|[Le portail Azure](https://portal.azure.com/)|application web dédiée à la gestion de tous les services Azure.|
|[Azure Data Studio](/sql/azure-data-studio/)|outil de base de données multiplateforme qui s’exécute sur Windows, macOS et Linux.|
|[SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms)|application client téléchargeable gratuitement, dédiée à la gestion des infrastructures SQL, depuis SQL Server jusqu’à SQL Database.|
|[SQL Server Data Tools dans Visual Studio](/sql/ssdt/download-sql-server-data-tools-ssdt)|application cliente téléchargeable gratuite, dédiée au développement de bases de données relationnelles SQL Server, de bases de données Azure SQL Database, de packages Integration Services, de modèles de données Analysis Services et de rapports Reporting Services.|
|[Visual Studio Code](https://code.visualstudio.com/docs)|éditeur de code open source téléchargeable gratuitement pour Windows, macOS et Linux. Il prend en charge les extensions, notamment l’[extension mssql](https://aka.ms/mssql-marketplace) pour l’exécution de requêtes dans Microsoft SQL Server, Azure SQL Database et Azure Synapse Analytics.|

SQL Database prend en charge la génération d’applications avec Python, Java, Node.js, PHP, Ruby et .NET sur macOS, Linux et Windows. SQL Database prend en charge les mêmes [bibliothèques de connexions](connect-query-content-reference-guide.md#libraries) que SQL Server.

[!INCLUDE [sql-database-create-manage-portal](../includes/sql-database-create-manage-portal.md)]

## <a name="sql-database-frequently-asked-questions"></a>Question fréquentes sur SQL Database

### <a name="can-i-control-when-patching-downtime-occurs"></a>Puis-je contrôler le moment où se produit le temps d’arrêt pour une mise à jour corrective ?

Non. L’impact d’une mise à jour corrective est généralement imperceptible si vous [utilisez une logique de nouvelle tentative](develop-overview.md#resiliency) dans votre application. Pour plus d’informations, consultez [Planification des événements de maintenance Azure dans Azure SQL Database](planned-maintenance.md).



## <a name="engage-with-the-sql-server-engineering-team"></a>Collaborer avec l’équipe d’ingénierie de SQL Server

- [DBA Stack Exchange](https://dba.stackexchange.com/questions/tagged/sql-server) : posez vos questions sur l’administration des bases de données.
- [Stack Overflow](https://stackoverflow.com/questions/tagged/sql-server) : posez vos questions sur le développement.
- [Page de questions Microsoft Q&A](/answers/topics/azure-synapse-analytics.html) : posez vos questions techniques.
- [Feedback](https://aka.ms/sqlfeedback) : signalez les bogues et demandez des fonctionnalités.
- [Reddit](https://www.reddit.com/r/SQLServer/) : échangez au sujet de SQL Server.

## <a name="next-steps"></a>Étapes suivantes

- Consultez la [page des tarifs](https://azure.microsoft.com/pricing/details/sql-database/) pour obtenir un comparatif des coûts et des calculateurs concernant les bases de données uniques et des pools élastiques.
- Consultez ces guides de démarrage rapide pour bien commencer :

  - [Créer une base de données dans le portail Azure](single-database-create-quickstart.md)  
  - [Créer une base de données avec Azure CLI](az-cli-script-samples-content-guide.md)
  - [Créer une base de données à l’aide de PowerShell](powershell-script-content-guide.md)

- Pour accéder à des exemples Azure CLI et PowerShell, consultez :
  - [Exemples d’interface de ligne de commande Azure pour SQL Database](az-cli-script-samples-content-guide.md)
  - [Exemples Azure PowerShell pour SQL Database](powershell-script-content-guide.md)

- Pour plus d’informations sur les nouvelles fonctionnalités annoncées, consultez [Feuille de route pour Azure SQL Database](https://azure.microsoft.com/roadmap/?category=databases).
- Consultez le [Blog Azure SQL Database](https://azure.microsoft.com/blog/topics/database), où les membres de l’équipe produit SQL Server publient des billets de blog sur les nouveautés et les fonctionnalités de SQL Database.