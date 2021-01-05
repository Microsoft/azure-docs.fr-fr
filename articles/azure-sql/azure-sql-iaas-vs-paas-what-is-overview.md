---
title: Qu'est-ce que SQL Azure ?
description: 'Découvrez les différentes options de la famille de services Azure SQL : Azure SQL Database, Azure SQL Managed Instance et SQL Server sur machine virtuelle Azure.'
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: sqldbrb=4
ms.devlang: ''
ms.topic: overview
keywords: cloud SQL Server, SQL Server dans le cloud, base de données PaaS, SQL Server cloud, DBaaS, IaaS
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 07/27/2020
ms.openlocfilehash: 99703acc4965a38337a39fe23cc19cafddd1531d
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92791118"
---
# <a name="what-is-azure-sql"></a>Qu'est-ce que SQL Azure ? 
[!INCLUDE[appliesto-asf](includes/appliesto-asf.md)]

Azure SQL est une famille de produits gérés, sécurisés et intelligents qui utilisent le moteur de base de données SQL Server dans le Cloud Azure.

- **Azure SQL Database**  : Prenez en charge des applications cloud modernes sur un service de base de données géré, intelligent et doté des fonctionnalités de calcul serverless. 
- **Azure SQL Managed Instance**  : Modernisez vos applications SQL Server existantes à grande échelle avec une instance en tant que service, intelligente et pleinement gérée, avec presque 100 % de parité de fonctionnalités avec le moteur de base de données SQL Server. Idéales pour la plupart des migrations vers le cloud.
- **Machines virtuelles SQL Server sur Azure**  : Effectuez en toute simplicité le portage virtuel de vos charges de travail SQL Server et conservez 100 % de compatibilité avec SQL Server et un accès au niveau du système d’exploitation. 
 
Azure SQL repose sur le moteur SQL Server bien connu, ce qui vous permet de migrer facilement des applications et de continuer à utiliser les outils, les langages et les ressources qui vous sont familiers. Vos compétences et votre expérience sont transférées vers le cloud, ce qui vous permet d’en faire encore plus avec ce que vous possédez déjà. 

Découvrez comment chaque produit s’intègre sur la plateforme de données Azure SQL de Microsoft afin d’identifier la meilleure option pour vos besoins métier. Que vous privilégiiez les économies ou une administration minimale, cet article peut vous aider à déterminer l’approche adaptée à vos exigences.


Si vous ne connaissez pas Azure SQL, regardez la vidéo *Qu'est-ce qu’Azure SQL ?* , qui fait partie de notre [série de vidéos Azure SQL](https://channel9.msdn.com/Series/Azure-SQL-for-Beginners?WT.mc_id=azuresql4beg_azuresql-ch9-niner) :
> [!VIDEO https://channel9.msdn.com/Series/Azure-SQL-for-Beginners/What-is-Azure-SQL-3-of-61/player]



## <a name="overview"></a>Vue d’ensemble

Dans le monde des données actuel, diriger la transformation numérique dépend de plus en plus de notre capacité à gérer des quantités massives de données et à exploiter leur potentiel. Mais aujourd’hui, les patrimoines de données sont de plus en plus complexes, avec des données hébergées localement, dans le cloud ou à la périphérie du réseau. Les développeurs qui créent des applications intelligentes et immersives peuvent se trouver contraints par des limitations susceptibles d’impacter au final leur expérience. Les limitations résultant de plateformes incompatibles, une sécurité inadéquate des données, des ressources insuffisantes et des barrières en matière de rapport prix/performances créent une complexité qui peut entraver la modernisation et le développement des applications. 

L'une des premières choses à comprendre dans toute discussion comparant les bases de données Azure et SQL Server locales est que vous pouvez tout utiliser. La plateforme de données de Microsoft utilise la technologie SQL Server et la rend disponible sur les machines physiques locales, les environnements de cloud privé, les environnements de cloud privé hébergé par un tiers et le cloud public. 


### <a name="fully-managed-and-always-up-to-date"></a>Entièrement géré et toujours à jour 

Consacrez plus de temps à l’innovation et passez moins de temps à installer des correctifs, à mettre à jour et à sauvegarder vos bases de données. Azure est le seul cloud doté de fonctionnalités SQL persistantes, automatiquement mises à jour et corrigées afin que vos bases de données soient toujours à jour, éliminant ainsi les tracas de fin de support. Même des tâches complexes telles que le réglage des performances, la haute disponibilité, la reprise d’activité après sinistre et les sauvegardes sont automatisées, ce qui vous permet de vous concentrer sur les applications.  

### <a name="protect-your-data-with-built-in-intelligent-security"></a>Données protégées grâce à la sécurité intelligente intégrée 

Azure surveille constamment vos données pour détecter les menaces. Avec Azure SQL, vous pouvez :

- Résoudre des menaces potentielles en temps réel avec une [détection avancée des menaces](../security/fundamentals/threat-detection.md#advanced-threat-detection-features-other-azure-services) intelligente et des alertes proactives d’évaluation des vulnérabilités. 
- Bénéficier d’une protection multicouche leader sur le marché grâce à des [contrôles de sécurité intégrés](https://azure.microsoft.com/overview/security/), notamment T-SQL, l’authentification, la mise en réseau et la gestion des clés. 
- Tirer parti de la couverture de [conformité](https://azure.microsoft.com/overview/trusted-cloud/compliance/) la plus complète de n’importe quel service de base de données cloud. 


### <a name="business-motivations"></a>Facteurs de motivation

Plusieurs facteurs peuvent influencer votre choix parmi les différentes offres de données :

- [Coût](#cost) : les options PaaS et IaaS incluent toutes les deux un prix de base couvrant l’infrastructure sous-jacente et les licences. Toutefois, avec l’option IaaS, vous devez consacrer plus de temps et de ressources à la gestion de votre base de données, alors qu’avec PaaS, ces fonctionnalités d’administration sont incluses dans le prix. L’option IaaS vous permet d’arrêter vos ressources quand vous ne les utilisez pas afin de réduire le coût, alors que la version PaaS s’exécute toujours à moins que vous supprimiez et recréiez vos ressources quand vous en avez besoin.
- [Administration](#administration) : les options PaaS réduisent le temps que vous devez consacrer à l’administration de la base de données. Toutefois, elles limitent également l’éventail des tâches et scripts administratifs personnalisés que vous pouvez effectuer ou exécuter. Par exemple, le CLR n’est pas pris en charge avec SQL Database, mais il l’est pour une instance de SQL Managed Instance. De plus, aucune option de déploiement dans PaaS ne prend en charge l’utilisation des indicateurs de trace.
- [Contrat de niveau de service](#service-level-agreement-sla) : IaaS et PaaS fournissent un contrat SLA standard de niveau élevé. L’option PaaS garantit un contrat SLA de 99,99 %, alors que IaaS garantit un contrat SLA de 99,95 % pour l’infrastructure, ce qui signifie que vous devez implémenter des mécanismes supplémentaires pour garantir la disponibilité de vos bases de données. Vous pouvez atteindre un contrat de niveau de service de 99,99 % en créant une machine virtuelle SQL supplémentaire et en implémentant la solution de haute disponibilité du groupe de disponibilité SQL Server Always On. 
- [Il est temps de migrer vers Azure](#market) : SQL Server sur une machine virtuelle Azure correspond exactement à votre environnement, de sorte que la migration du système local vers la machine virtuelle Azure ne diffère pas du déplacement des bases de données d’un serveur local vers un autre. SQL Managed Instance facilite également la migration. Toutefois, vous devrez peut-être appliquer certains changements avant la migration. 


## <a name="service-comparison"></a>Comparaison des services

   ![Options cloud de SQL Server : SQL Server sur IaaS ou SQL Database SaaS dans le cloud.](./media/azure-sql-iaas-vs-paas-what-is-overview/SQLIAAS_SQL_Server_Cloud_Continuum.png)

Comme l’indique le schéma, chaque offre de service peut être caractérisée par le niveau d’administration dont vous disposez sur l’infrastructure, ainsi que par le degré de rentabilité.

Dans Azure, vos charges de travail SQL Server peuvent s’exécuter en tant que service hébergé ([PaaS](https://azure.microsoft.com/overview/what-is-paas/)) ou infrastructure hébergée ([IaaS](https://azure.microsoft.com/overview/what-is-iaas/)). Dans PaaS, vous avez plusieurs options de produit et plusieurs niveaux de service dans chaque option. La question clé que vous devez poser quand vous choisissez entre PaaS ou IaaS est si vous voulez gérer votre base de données, appliquer des correctifs et effectuer des sauvegardes, ou plutôt déléguer ces opérations à Azure ?

### <a name="azure-sql-database"></a>Azure SQL Database

[Azure SQL Database](database/sql-database-paas-overview.md) est une base de données en tant que service (DBaaS) relationnelle hébergée dans Azure, qui appartient à la catégorie *PaaS (Plateforme en tant que service)* . 
- Idéales pour les applications cloud modernes qui veulent utiliser les dernières fonctionnalités stables de SQL Server et ont des contraintes de délais de développement et de commercialisation. 
- Un moteur de base de données SQL Server complètement managé, qui s’appuie sur la dernière édition Entreprise stable de SQL Server. Azure SQL Database offre deux options de déploiement reposant sur du matériel et des logiciels standardisés, qui sont détenus, hébergés et gérés par Microsoft. 

Avec SQL Server, vous pouvez utiliser les fonctionnalités intégrées ainsi que celles qui nécessitent une configuration étendue (localement ou sur une machine virtuelle Azure). Lorsque vous utilisez SQL Database, vous payez à l’utilisation, avec des possibilités de mise à l’échelle pour obtenir plus de puissance sans interruption du service. SQL Database offre des fonctionnalités supplémentaires qui ne sont pas disponibles dans SQL Server, comme la haute disponibilité, l’intelligence et la gestion intégrées.


Azure SQL Database propose les options de déploiement suivantes  :
  - En tant que [*base de données unique*](database/single-database-overview.md) avec son propre ensemble de ressources gérées via un [serveur SQL logique](database/logical-servers.md). Une base de données unique est similaire à une [base de données autonome](/sql/relational-databases/databases/contained-databases) dans SQL Server. Cette option est optimisée pour le développement moderne de nouvelles applications issues du cloud. Les options [Hyperscale](database/service-tier-hyperscale.md) et [serverless](database/serverless-tier-overview.md) sont disponibles.
  - Un [*pool élastique*](database/elastic-pool-overview.md), qui est une collection de bases de données avec un ensemble partagé de ressources gérées via un [serveur SQL logique](database/logical-servers.md). Les bases de données uniques peuvent être déplacées dans et hors d’un pool élastique. Cette option est optimisée pour le développement moderne de nouvelles applications issues du cloud au moyen du modèle d’application SaaS mutualisée. Les pools élastiques offrent une solution rentable pour gérer les performances de plusieurs bases de données ayant des modèles d’utilisation variables.

### <a name="azure-sql-managed-instance"></a>Azure SQL Managed Instance

[Azure SQL Managed Instance](managed-instance/sql-managed-instance-paas-overview.md) s’inscrit dans la catégorie *PaaS (Plateforme en tant que service)* et est parfaitement adapté à la plupart des migrations vers le cloud. SQL Managed Instance est une collection de bases de données système et utilisateur avec un ensemble partagé de ressources prêtes pour la technologie de portage virtuel (lift-and-shift).  
- Idéal pour de nouvelles applications ou des applications locales existantes qui utilisent les dernières fonctionnalités stables de SQL Server et sont migrées vers le cloud avec des modifications minimales. Une instance d’Azure SQL Managed Instance est similaire à une instance du [moteur de base de données Microsoft SQL Server](/sql/database-engine/sql-server-database-engine-overview) offrant des ressources partagées pour les bases de données, ainsi que des fonctionnalités supplémentaires incluses dans l’instance. 
- Une instance SQL Managed Instance prend en charge la migration de base de données depuis un emplacement local avec un minimum de changements, voire sans aucun changement, de la base de données. Cette option fournit non seulement tous les avantages PaaS d’Azure SQL Database, mais ajoute également des fonctionnalités qui n’étaient auparavant disponibles que sur les machines virtuelles SQL Server. Parmi elles, un réseau virtuel natif et une compatibilité proche de 100 % avec le SQL Server local. Les instances d’Azure SQL Managed Instance offrent un accès complet à SQL Server et la compatibilité complète des fonctionnalités pour la migration des serveurs SQL Server vers Azure.

### <a name="sql-server-on-azure-vm"></a>SQL Server sur une machine virtuelle Azure

[SQL Server sur une machine virtuelle Azure](virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md) appartient à la catégorie *IaaS (Infrastructure en tant que service)* et vous permet d’exécuter SQL Server sur une machine virtuelle complètement managée dans Azure. 
- Idéales pour les migrations et applications nécessitant un accès au niveau du système d’exploitation. Les machines virtuelles SQL dans Azure sont prêtes pour le portage virtuel (lift-and-shift) pour des applications existantes qui requièrent une migration rapide vers le cloud avec un minimum de changements, voire sans aucun changement. Les machines virtuelles SQL offrent un contrôle administratif complet sur l’instance SQL Server et le système d’exploitation sous-jacent pour la migration vers Azure. 
- Scénarios de développement et de test rapides lorsque vous ne souhaitez pas acheter du matériel SQL Server local non destiné à la production. Les machines virtuelles SQL s’exécutent également sur du matériel standardisé détenu, hébergé et entretenu par Microsoft. Quand vous utilisez des machines virtuelles SQL, vous pouvez payer à l’utilisation pour une licence SQL Server déjà incluse dans une image SQL Server ou utiliser simplement une licence existante. Vous pouvez également arrêter ou redémarrer la machine virtuelle en fonction des besoins. 
- SQL Server installé et hébergé dans le cloud s’exécute sur des machines virtuelles Windows Server ou Linux opérant sur Azure, également appelées infrastructure en tant que service (IaaS). Les machines virtuelles SQL constituent une bonne option pour la migration d’applications et de bases de données SQL Server locales sans aucun changement de base de données. Toutes les dernières versions et éditions de SQL Server sont disponibles pour l’installation dans une machine virtuelle IaaS. 

    La différence la plus significative par rapport à SQL Database et aux instances SQL Managed Instance est que SQL Server sur des machines virtuelles Azure permet un contrôle total du moteur de base de données. Vous pouvez choisir quand la maintenance/mise à jour corrective démarrera, de passer d’un mode de récupération simple à un mode utilisant les journaux de transactions, d’arrêter ou de démarrer le service quand c’est nécessaire et vous pouvez personnaliser entièrement le moteur de base de données SQL Server. Ce contrôle supplémentaire est assorti d’une la responsabilité de gestion accrue de la machine virtuelle.
- Optimisé pour la migration d’applications existantes vers Azure ou l’extension d’applications locales existantes au cloud dans des déploiements hybrides. En outre, vous pouvez utiliser SQL Server sur une machine virtuelle pour développer et tester des applications traditionnelles SQL Server. Avec les machines virtuelles SQL, vous disposez de droits d’administration complets sur une instance SQL Server dédiée et sur une machine virtuelle dans le cloud. C'est le choix idéal lorsqu'une organisation possède déjà les ressources informatiques disponibles pour maintenir les machines virtuelles. Ces fonctionnalités vous permettent de créer un système hautement personnalisé afin de répondre aux exigences de performances de disponibilité de votre application.

Le tableau ci-dessous répertorie d’autres différences. Cependant, *SQL Database et SQL Managed Instance sont optimisés pour réduire au minimum les coûts de gestion globaux liés au provisionnement et à la gestion de nombreuses bases de données.* Les coûts d’administration en cours sont réduits car vous n’avez pas à gérer les machines virtuelles, le système d’exploitation ou le logiciel de base de données. Vous n’avez pas à gérer les mises à niveau, la haute disponibilité, ni les [sauvegardes](database/automated-backups-overview.md). 

En règle générale, SQL Database et SQL Managed Instance peuvent augmenter considérablement le nombre de bases de données gérées par un informaticien ou un développeur. Les [pools élastiques](database/elastic-pool-overview.md) prennent également en charge les architectures d’applications SaaS multi-locataires grâce à des fonctionnalités incluant l’isolation des locataires et la possibilité de mettre à l’échelle pour réduire les coûts en partageant des ressources entre les bases de données. [SQL Managed Instance](managed-instance/sql-managed-instance-paas-overview.md) prend en charge les fonctionnalités incluses dans l’instance permettant de migrer facilement des applications existantes, ainsi que de partager des ressources entre bases de données.

### <a name="comparison-table"></a>Tableau de comparaison

| Azure SQL Database | Azure SQL Managed Instance | SQL Server sur une machine virtuelle Azure |
| :--- | :--- | :--- |
|Prend en charge la plupart des fonctionnalités locales au niveau de la base de données. Les fonctionnalités SQL Server les plus couramment utilisées sont disponibles.<br/>99,995 % de disponibilité garantie.<br/>Sauvegardes, mise à jour corrective, récupération intégrées.<br/>Dernière version stable du moteur de base de données.<br/>Possibilité d’attribuer les ressources nécessaires (processeur/stockage) à des bases de données individuelles.<br/>Intelligence et sécurité avancées intégrées.<br/>Changement en ligne des ressources (processeur/stockage).| Prend en charge presque toutes les fonctionnalités locales au niveau de l’instance et de la base de données. Haute compatibilité avec SQL Server.<br/>99,99 % de disponibilité garantie.<br/>Sauvegardes, mise à jour corrective, récupération intégrées.<br/>Dernière version stable du moteur de base de données.<br/>Migration facile à partir de SQL Server.<br/>Adresse IP privée dans un réseau virtuel Azure.<br/>Intelligence et sécurité avancées intégrées.<br/>Changement en ligne des ressources (processeur/stockage).| Vous avez le contrôle total du moteur SQL Server. Prend en charge toutes les fonctionnalités locales.<br/>Disponibilité jusqu’à 99,99 %.<br/>Parité complète avec la version locale correspondante de SQL Server.<br/>Version du moteur de base de données fixe et connue.<br/>Migration facile à partir de SQL Server.<br/>Adresse IP privée dans un réseau virtuel Azure.<br/>Vous avez la possibilité de déployer des applications ou des services sur l’hôte où réside SQL Server.|
|La migration depuis SQL Server peut être difficile.<br/>Certaines fonctionnalités SQL Server ne sont pas disponibles.<br/>Temps de maintenance exact non garanti (mais presque transparent).<br/>Vous pouvez obtenir la compatibilité avec la version de SQL Server uniquement à l’aide des niveaux de compatibilité de base de données.<br/>Prise en charge des adresses IP privées avec [Azure Private Link](database/private-endpoint-overview.md).|Un nombre minime de fonctionnalités SQL Server ne sont toujours pas disponibles.<br/>Temps de maintenance exact non garanti (mais presque transparent).<br/>Vous pouvez obtenir la compatibilité avec la version de SQL Server uniquement à l’aide des niveaux de compatibilité de base de données.|Vous devez gérer vos sauvegardes et correctifs.<br>Vous devez implémenter votre propre solution de haute disponibilité.<br/>Vous avez un temps d’arrêt pendant le changement des ressources (processeur/storage)|
| Bases de données comprenant jusqu’à 100 To. | Jusqu’à 8 To. | Instances SQL Server avec 256 To de stockage maximum. L’instance peut prendre en charge autant de bases de données que nécessaire. |
| L’application locale peut accéder aux données dans Azure SQL Database. | [Implémentation d’un réseau virtuel natif](managed-instance/vnet-existing-add-subnet.md) et sa connexion à votre environnement local à l’aide d’Azure Express Route ou d’une passerelle VPN. | Avec des machines virtuelles SQL, vous pouvez avoir des applications qui s’exécutent en partie dans le cloud et en partie localement. Par exemple, vous pouvez étendre votre réseau local et votre domaine Active Directory au cloud par le biais d’ [Azure Virtual Network](../virtual-network/virtual-networks-overview.md). Pour plus d’informations sur les solutions cloud hybrides, consultez [Extension des solutions de données locales vers le cloud](/azure/architecture/data-guide/scenarios/hybrid-on-premises-and-cloud). |


## <a name="cost"></a>Coût

Que vous soyez une start-up à court de liquidités ou une équipe dans une société établie qui subit de fortes contraintes budgétaires, le manque de capitaux est souvent le principal moteur pour décider du mode d’hébergement des bases de données. Dans cette section, vous allez découvrir les principes de base de facturation et de licence dans Azure associés à la famille de services Azure SQL.  Vous apprendrez également à calculer le coût total de l’application.

### <a name="billing-and-licensing-basics"></a>Notions de base sur la facturation et les licences

Actuellement, **SQL Database** et **SQL Managed Instance** sont vendus en tant que service et sont disponibles avec diverses options, ainsi qu’avec divers niveaux de service à des prix différents selon les ressources. Tous sont facturés à un tarif horaire fixe en fonction du niveau de service et de la taille de calcul choisis. Pour obtenir les dernières informations sur les niveaux de service, les tailles de calcul et les quantités de stockage actuellement pris en charge, consultez [Modèle d’achat DTU pour SQL Database](database/service-tiers-dtu.md) et [Modèle d’achat vCore pour SQL Database et SQL Managed Instance](database/service-tiers-vcore.md).

- Avec SQL Database, vous pouvez choisir un niveau de service qui répond à vos besoins à partir de 5 USD/mois pour le niveau de base et vous pouvez créer des [pools élastiques](database/elastic-pool-overview.md) pour partager des ressources entre les bases de données afin de réduire les coûts et d’absorber les pics d’utilisation.
- Avec SQL Managed Instance, vous pouvez également utiliser votre propre licence. Pour plus d’informations sur l’utilisation de votre propre licence, consultez [License Mobility via Software Assurance sur Azure](https://azure.microsoft.com/pricing/license-mobility/) ou utilisez la [calculatrice Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/#sql-database) pour voir comment **économiser jusqu’à 40 %** .

Par ailleurs, vous êtes facturé pour le trafic internet sortant aux [tarifs de transfert de données](https://azure.microsoft.com/pricing/details/data-transfers/)standard. Vous pouvez ajuster les niveaux de service et les tailles de calcul de manière dynamique pour répondre aux besoins de débit variés de votre application.

Avec **SQL Database** et **SQL Managed Instance** , Azure configure, corrige et met à niveau automatiquement le logiciel de base de données, ce qui réduit vos coûts d’administration. En outre, ses fonctionnalités de [sauvegarde intégrée](database/automated-backups-overview.md) vous permettent de réaliser d’importantes économies, notamment si vous avez un grand nombre de bases de données.

Avec **SQL sur des machines virtuelles Azure** , vous pouvez utiliser une image SQL Server fournie par la plateforme (qui inclut une licence) ou votre propre licence SQL Server. Toutes les versions SQL Server (2008R2, 2012, 2014, 2016, 2017, 2019) et les éditions (Developer, Express, Web, Standard, Enterprise) prises en charge sont disponibles. Les versions BYOL (apportez votre propre licence) des images sont également disponibles. Lorsque vous utilisez les images fournies par Azure, les coûts opérationnels dépendent de la taille de la machine virtuelle et de l’édition SQL Server choisie. Quelle que soit la taille de la machine virtuelle ou l’édition de SQL Server, vous payez à la minute l’utilisation de la licence de SQL Server et du serveur Windows ou Linux, ainsi que l’utilisation du stockage Azure pour les disques de machine virtuelle. L’option de facturation à la minute vous permet d’utiliser SQL Server aussi longtemps que vous en avez besoin, sans acheter de licences SQL Server supplémentaires. Si vous utilisez votre propre licence SQL Server sur Azure, seuls les coûts du serveur et du stockage vous sont facturés. Pour plus d’informations sur l’utilisation de votre propre licence, consultez [License Mobility via Software Assurance sur Azure](https://azure.microsoft.com/pricing/license-mobility/). Par ailleurs, vous êtes facturé pour le trafic internet sortant aux [tarifs de transfert de données](https://azure.microsoft.com/pricing/details/data-transfers/)standard.

#### <a name="calculating-the-total-application-cost"></a>Calcul du coût total de l'application

Lorsque vous utilisez une plateforme cloud, le coût d’exécution de votre application inclut le coût du nouveau développement et les frais d’administration courante, ainsi que les coûts de service pour la plateforme de cloud public.

Pour plus d'informations sur la tarification, consultez les ressources suivantes :

- [Tarification SQL Database et SQL Managed Instance](https://azure.microsoft.com/pricing/details/sql-database/)
- [Tarification des machines virtuelles](https://azure.microsoft.com/pricing/details/virtual-machines/) pour [SQL](https://azure.microsoft.com/pricing/details/virtual-machines/#sql) et [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/#windows)
- [Calcul des coûts Azure](https://azure.microsoft.com/pricing/calculator/)

## <a name="administration"></a>Administration

Pour de nombreuses entreprises, la décision de migrer vers un service cloud vise essentiellement à simplifier la complexité d’administration, qui a un coût. Avec IaaS et Paas, Azure administre l’infrastructure sous-jacente et réplique automatiquement toutes les données pour assurer la récupération d'urgence, configure et met à niveau le logiciel de base de données, gère l’équilibrage de charge et procède au basculement transparent en cas de défaillance du serveur au sein d’un centre de données.

- Avec **SQL Database** et **SQL Managed Instance** , vous pouvez continuer d’administrer votre base de données, mais vous n’avez plus à gérer le moteur de base de données, le système d’exploitation ni le matériel. Parmi les éléments que vous pouvez continuer d’administrer, il y a les bases de données et les connexions, l’optimisation des index et des requêtes, ainsi que l’audit et la sécurité. Par ailleurs, la configuration de la haute disponibilité vers un autre centre de données ne requiert qu’une configuration et une administration minimales.
- Avec **SQL sur des machines virtuelles Azure** , vous avez un contrôle total sur le système d’exploitation et la configuration des instances SQL Server. Avec une machine virtuelle, c’est à vous de décider quand mettre à jour ou à niveau le système d’exploitation et le logiciel de base de données, et quand installer d’autres logiciels tels que l’antivirus. Certaines fonctionnalités automatisées simplifient considérablement la gestion des correctifs, la sauvegarde et la haute disponibilité. En outre, vous pouvez contrôler la taille de la machine virtuelle, le nombre de disques et leurs configurations de stockage. Azure vous permet de modifier la taille d’une machine virtuelle en fonction des besoins. Pour plus d’informations, consultez [Tailles de machines virtuelles et de services cloud pour Azure](../virtual-machines/sizes.md).

## <a name="service-level-agreement-sla"></a>Contrat de niveau de service (SLA)

Pour bon nombre de services informatiques, répondre aux obligations de temps d’exécution d’un contrat de niveau de service (SLA) est la priorité absolue. Dans cette section, nous allons détailler les implications du contrat SLA pour chaque option d’hébergement de base de données.

Pour **Azure SQL Database** et **Azure SQL Managed Instance** , Microsoft fournit un contrat SLA de disponibilité de 99,99 %. Pour obtenir les dernières informations, consultez [Contrat de niveau de service](https://azure.microsoft.com/support/legal/sla/sql-database/).

Pour **SQL sur des machines virtuelles Azure** , Microsoft fournit un contrat SLA de disponibilité de 99,95 % couvrant uniquement la machine virtuelle. Ce contrat SLA ne couvre pas les processus (comme SQL Server) exécutés sur la machine virtuelle et nécessite l’hébergement d’au moins deux instances de machine virtuelle dans un groupe à haute disponibilité. Pour plus d’informations, consultez le [contrat SLA de machine virtuelle](https://azure.microsoft.com/support/legal/sla/virtual-machines/). Pour obtenir une haute disponibilité de la base de données sur des machines virtuelles, vous devez configurer une des options de haute disponibilité prises en charge dans SQL Server, telle que les [groupes de disponibilité Always On](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server). L’utilisation d’une option de haute disponibilité prise en charge ne fournit pas de SLA supplémentaire, mais vous pouvez atteindre une disponibilité de base de données > 99,99 %.

## <a name="time-to-move-to-azure"></a><a name="market"></a>Il est temps de migrer vers Azure

**Azure SQL Database** est la solution idéale pour des applications cloud quand la productivité des développeurs et la rapidité de mise sur le marché de nouvelles solutions sont essentielles. Avec les fonctionnalités de programmation comme le DBA, il est parfait pour les architectes et les développeurs du cloud, car il tempère la nécessité de gérer le système d'exploitation et la base de données sous-jacents.

**Azure SQL Managed Instance** simplifie considérablement la migration d’applications existantes vers Azure, ce qui vous permet de commercialiser rapidement des applications de base de données migrées dans Azure.

**SQL sur un machine virtuelle Azure** est la solution idéale si vos applications nouvelles ou existantes exigent des bases de données volumineuses ou un accès à toutes les fonctionnalités dans SQL Server ou Windows/Linux, et si vous voulez éviter la perte de temps et les coûts occasionnés par l’acquisition de nouveau matériel local. Cette solution convient également si vous souhaitez migrer des applications et des bases de données locales en l’état vers Azure, dans les cas où SQL Database et SQL Managed Instance ne sont pas adaptés. Étant donné que vous n’avez pas besoin de changer la présentation, l’application et les couches de données, vous économisez en temps et en budget sur le remaniement de votre solution existante. Ainsi, vous pouvez vous concentrer sur la migration de toutes vos solutions vers Azure et sur l’optimisation des performances requises par la plateforme Azure. Pour plus d’informations, consultez [Meilleures pratiques relatives aux performances de SQL Server dans les machines virtuelles Azure](virtual-machines/windows/performance-guidelines-best-practices.md).

[!INCLUDE [sql-database-create-manage-portal](includes/sql-database-create-manage-portal.md)]

## <a name="next-steps"></a>Étapes suivantes

- Pour prendre en main SQL Database, consultez la page [Votre première base de données Azure SQL Database](database/single-database-create-quickstart.md).
- Consultez [votre première instance Azure SQL Managed Instance](managed-instance/instance-create-quickstart.md) pour bien démarrer avec SQL Managed Instance. 
- Voir [Tarification de Base de données SQL](https://azure.microsoft.com/pricing/details/sql-database/).
- Pour plus d’informations sur la prise en main de SQL Server sur Azure Virtual Machines, voir [Approvisionnement d’une machine virtuelle SQL Server dans Azure](virtual-machines/windows/create-sql-vm-portal.md) .
- [Identifier la bonne référence SKU SQL Database or SQL Managed Instance pour votre base de données locale](/sql/dma/dma-sku-recommend-sql-db/).