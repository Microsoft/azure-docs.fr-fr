---
title: Qu’est-ce qu’un pool Azure SQL Managed Instance ?
titleSuffix: Azure SQL Managed Instance
description: Découvrez les pools Azure SQL Managed Instance (préversion), fonctionnalité qui offre un moyen pratique et économique de migrer de petites bases de données SQL Server vers le cloud à grande échelle et de gérer plusieurs instances managées.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: bonova
ms.author: bonova
ms.reviewer: sstein
ms.date: 09/05/2019
ms.openlocfilehash: bc345509db1c2a14afb0ae781eccad8f77395c18
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347062"
---
# <a name="what-is-an-azure-sql-managed-instance-pool-preview"></a>Qu’est-ce qu’un pool Azure SQL Managed Instance (préversion) ?
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Les pools d’instances dans Azure SQL Managed Instance offrent un moyen pratique et économique de migrer de petites instances SQL Server vers le cloud à grande échelle.

Les pools d’instances vous permettent de préconfigurer les ressources de calcul en fonction de vos besoins de migration totaux. Vous pouvez ensuite déployer plusieurs instances gérées individuelles jusqu’à votre niveau de calcul préconfiguré. Par exemple, si vous préconfigurez huit vCores, vous pouvez déployer deux instances à deux vCores et une instance à quatre vCores, puis migrer les bases de données vers ces instances. Avant la disponibilité des pools d’instances, les charges de travail nécessitant beaucoup plus de ressources de calcul doivent souvent être consolidées dans une plus grande instance gérée lors de la migration vers le cloud. La nécessité de migrer des groupes de bases de données vers une grande instance exige généralement une planification minutieuse des capacités et une gouvernance des ressources, des considérations supplémentaires sur la sécurité et des tâches de consolidation des données supplémentaires au niveau de l’instance.

En outre, les pools d’instances prennent en charge l’intégration au réseau virtuel de façon native, ce qui vous permet de déployer plusieurs pools d’instances et plusieurs instances uniques dans le même sous-réseau.

## <a name="key-capabilities"></a>Fonctionnalités clés

La fonctionnalité Pools d’instances permet de bénéficier des avantages suivants :

1. Possibilité d’héberger des instances à deux vCores. *\*Uniquement pour les instances dans les pools d’instances*.
2. Temps de déploiement d’instances prévisible et rapide (jusqu’à 5 minutes).
3. Allocation minimale d’adresses IP.

Le diagramme suivant illustre un pool d’instances avec plusieurs instances managées déployées sur un sous-réseau de réseau virtuel.

![Pool d’instances avec plusieurs instances](./media/instance-pools-overview/instance-pools1.png)

Les pools d’instances permettent le déploiement de plusieurs instances sur la même machine virtuelle, où la taille de calcul de la machine virtuelle est basée sur le nombre total de vCores alloués pour le pool. Cette architecture permet de *partitionner* la machine virtuelle en plusieurs instances, qui peuvent être de n’importe quelle taille prise en charge, y compris deux vCores (les instances à deux vCores sont uniquement disponibles pour les instances dans des pools).

Après le déploiement initial, les opérations de gestion sur les instances d’un pool sont beaucoup plus rapides. En effet, le déploiement ou l’extension d’un [cluster virtuel](connectivity-architecture-overview.md#high-level-connectivity-architecture) (ensemble dédié de machines virtuelles) ne fait pas partie du provisionnement de l’instance managée.

Étant donné que toutes les instances d’un pool partagent la même machine virtuelle, l’allocation d’adresses IP totale ne dépend pas du nombre d’instances déployées, ce qui est pratique pour le déploiement dans les sous-réseaux avec des plages d’adresses IP étroites.

Chaque pool a une allocation fixe d’adresses IP de neuf adresses IP seulement (sans compter les cinq adresses IP du sous-réseau qui sont réservées à ses propres besoins). Pour plus d’informations, consultez les [exigences de taille de sous-réseau pour les instances uniques](vnet-subnet-determine-size.md).

## <a name="application-scenarios"></a>Scénarios d’application

La liste suivante répertorie les principaux cas d’utilisation où les pools d’instances doivent être envisagés :

- La migration simultanée d’*un groupe d’instances SQL Server* dont la plupart sont de petite taille (par exemple deux ou quatre vCores).
- Les scénarios dans lesquels la *création ou la mise à l’échelle d’instances de façon prévisible et rapide* est importante. Par exemple, il peut s’agir du déploiement d’un nouveau locataire dans un environnement d’application SaaS mutualisée qui requiert des fonctionnalités au niveau de l’instance.
- Les scénarios où un *coût fixe* ou une *limite de dépense* sont importants. Par exemple, si vous exécutez des environnements de développement-test ou de démonstration partagés d’une taille fixe (ou rarement modifiée), où vous déployez régulièrement des instances gérées quand cela est nécessaire.
- Scénarios où *une allocation minimale d’adresses IP* dans un sous-réseau de réseau virtuel est importante. Toutes les instances d’un pool partagent une machine virtuelle ; ainsi, le nombre d’adresses IP allouées est inférieur dans le cas d’instances uniques.

## <a name="architecture"></a>Architecture

Les pools d’instances ont une architecture similaire aux instances managées ordinaires (*uniques*). Pour prendre en charge les [déploiements dans les réseaux virtuels Azure](../../virtual-network/virtual-network-for-azure-services.md) et assurer l’isolation et la sécurité des clients, les pools d’instances s’appuient sur des [clusters virtuels](connectivity-architecture-overview.md#high-level-connectivity-architecture). Les clusters virtuels représentent un ensemble dédié de machines virtuelles isolées déployées dans le sous-réseau du réseau virtuel du client.

La principale différence entre les deux modèles de déploiement est que les pools d’instances permettent plusieurs déploiements de processus SQL Server sur le même nœud de machine virtuelle, qui sont des ressources régies par des [objets de traitement Windows](/windows/desktop/ProcThread/job-objects), alors que les instances uniques sont toujours seules sur un nœud de machine virtuelle.

Le diagramme suivant montre un pool d’instances et deux instances individuelles déployées dans le même sous-réseau, et illustre les principaux détails de l’architecture pour les deux modèles de déploiement :

![Pool d’instances et deux instances individuelles](./media/instance-pools-overview/instance-pools2.png)

Chaque pool d’instances crée un cluster virtuel distinct sous-jacent. Les instances d’un pool et les instances uniques déployées sur le même sous-réseau ne partagent pas les ressources de calcul allouées aux processus SQL Server et aux composants de passerelle, ce qui garantit la prévisibilité des performances.

## <a name="resource-limitations"></a>Limitations des ressources

Il existe plusieurs limitations de ressources concernant les pools d’instances et les instances dans des pools :

- Les pools d’instances sont disponibles uniquement sur le matériel Gen5.
- Les instances managées d’un pool disposent d’un processeur et d’une RAM dédiés. Ainsi, le nombre agrégé de vCores sur toutes les instances doit être inférieur ou égal au nombre de vCores alloués au pool.
- Toutes les [limites au niveau de l’instance](resource-limits.md#service-tier-characteristics) s’appliquent aux instances créées au sein d’un pool.
- Outre les limites au niveau de l’instance, il existe également deux limites imposées *au niveau du pool d’instances* :
  - Taille de stockage totale par pool (8 To).
  - Nombre total de bases de données utilisateur par pool. Cette limite dépend de la valeur de vCores du pool :
    - un pool de 8 vCores prend en charge jusqu’à 200 bases de données,
    - un pool de 16 vCores prend en charge jusqu’à 400 bases de données,
    - un pool de 24 vCores et plus prend en charge jusqu’à 500 bases de données.
- Vous ne pouvez pas définir l’administrateur AAD pour les instances déployées à l’intérieur du pool d’instances. Vous ne pouvez donc pas utiliser l’authentification AAD.

L’allocation de stockage totale et le nombre de bases de données sur toutes les instances doivent être inférieurs ou égaux aux limites exposées par les pools d’instances.

- Les pools d’instances prennent en charge 8, 16, 24, 32, 40, 64 et 80 vCores.
- Les instances managées dans les pools prennent en charge 2, 4, 8, 16, 24, 32, 40, 64 et 80 vCores.
- Les instances managées dans les pools prennent en charge des tailles de stockage comprises entre 32 Go et 8 To, sauf dans les cas suivants :
  - les instances de 2 vCores prennent en charge des tailles comprises entre 32 Go et 640 Go,
  - les instances de 4 vCores prennent en charge des tailles comprises entre 32 Go et 2 To.
- Les instances managées à l’intérieur des pools sont limitées à 100 bases de données utilisateur chacune, à l’exception de 2 instances de vCore qui prennent en charge jusqu’à 50 bases de données utilisateur chacune.

La [propriété de niveau de service](resource-limits.md#service-tier-characteristics) étant associée à la ressource de pool d’instances, toutes les instances d’un pool doivent être du même niveau de service que le niveau de service du pool. À ce stade, seul le niveau de service Usage général est disponible (consultez la section suivante sur les limitations de la préversion actuelle).

### <a name="public-preview-limitations"></a>Limitations de la version préliminaire publique

La préversion publique présente les limitations suivantes :

- Actuellement, seul le niveau de service Usage général est disponible.
- Les pools d’instances ne peuvent pas être mis à l’échelle au cours de la préversion publique. Il est donc important de bien planifier la capacité avant le déploiement.
- La prise en charge du portail Azure pour la création et la configuration du pool d’instances n’est pas encore disponible. Toutes les opérations sur les pools d’instances sont uniquement prises en charge par le biais de PowerShell. Le déploiement d’instance initial dans un pool créé au préalable est également pris en charge uniquement via PowerShell. Une fois déployées dans un pool, les instances managées peuvent être mises à jour à l’aide du portail Azure.
- Les instances managées créées en dehors du pool ne peuvent pas être déplacées dans un pool existant, et les instances créées à l’intérieur d’un pool ne peuvent pas être déplacées en dehors en tant qu’instance unique ou vers un autre pool.
- Les tarifs des instances de la [capacité de réserve](../database/reserved-capacity-overview.md) ne sont pas disponibles.

## <a name="sql-features-supported"></a>Fonctionnalités SQL prises en charge

Les instances managées créées dans des pools prennent en charge les mêmes [niveaux de compatibilité et fonctionnalités prises en charge par les instances managées uniques](sql-managed-instance-paas-overview.md#sql-features-supported).

Chaque instance managée déployée dans un pool a une instance d’agent SQL distincte.

Les fonctionnalités facultatives qui nécessitent que vous choisissiez des valeurs spécifiques (telles que le classement au niveau de l’instance, le fuseau horaire, le point de terminaison public pour le trafic de données, les groupes de basculement) sont configurées au niveau de l’instance et peuvent être différentes pour chaque instance d’un pool.

## <a name="performance-considerations"></a>Considérations relatives aux performances

Bien que les instances managées au sein des pools disposent de vCores et de RAM dédiés, elles partagent le disque local (pour l’utilisation de tempdb) et les ressources réseau. Si cela est peu probable, il est possible d’observer un effet de *voisin bruyant* si plusieurs instances du pool ont une consommation élevée des ressources en même temps. Si vous observez ce comportement, envisagez de déployer ces instances dans un pool plus grand ou en tant qu’instances uniques.

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Étant donné que les instances déployées dans un pool partagent la même machine virtuelle, vous pouvez envisager de désactiver les fonctionnalités qui introduisent des risques de sécurité plus élevés ou contrôler fermement les autorisations d’accès à ces fonctionnalités. Par exemple, l’intégration du CLR, la sauvegarde et la restauration natives, la messagerie de base de données, etc.

## <a name="instance-pool-support-requests"></a>Demandes de support pour les pools d’instances

Créez et gérez les demandes de support pour les pools d’instances dans le [portail Azure](https://portal.azure.com).

Si vous rencontrez des problèmes liés au déploiement d’un pool d’instances (création ou suppression), assurez-vous de spécifier **Pools d’instances** dans le champ **Sous-type de problème**.

![Demande de support pour les pools d’instances](./media/instance-pools-overview/support-request.png)

Si vous rencontrez des problèmes liés à une instance managée unique ou à une base de données au sein d’un pool, vous devez créer un ticket de support standard pour Azure SQL Managed Instance.

Pour créer des déploiements SQL Managed Instance plus grands (avec ou sans pools d’instances), vous devrez peut-être obtenir un quota régional plus grand. Pour plus d’informations, consultez [Demander des augmentations de quota pour Azure SQL Database](../database/quota-increase-request.md). La logique de déploiement des pools d’instance compare la consommation totale de vCores *au niveau du pool* à votre quota pour déterminer si vous êtes autorisé à créer des ressources sans augmenter davantage votre quota.

## <a name="instance-pool-billing"></a>Facturation des pools d’instances

Les pools d’instances permettent de mettre à l’échelle le calcul et le stockage indépendamment. Les clients paient le calcul associé à la ressource de pool mesurée en vCores et le stockage associé à chaque instance, mesuré en gigaoctets (les 32 premiers Go sont gratuits pour chaque instance).

Le prix des vCores pour un pool est facturé quel que soit le nombre d’instances déployées dans ce pool.

Pour le prix du calcul (mesuré en vCores), deux options de tarifs sont disponibles :

  1. *Licence incluse* : Le prix des licences SQL Server est inclus. Cette option est destinée aux clients qui décident de ne pas appliquer de licences SQL Server existantes avec Software Assurance.
  2. *Azure Hybrid Benefit* : Un prix réduit incluant Azure Hybrid Benefit pour SQL Server. Les clients peuvent choisir cette tarification en utilisant leurs licences SQL Server existantes avec Software Assurance. Pour obtenir des informations sur l’éligibilité et d’autres informations, consultez [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/).

La définition d’options de tarification différentes n’est pas possible pour les instances individuelles d’un pool. Toutes les instances du pool parent doivent être au prix de la licence incluse ou au prix d’Azure Hybrid Benefit. Le modèle de licence du pool peut être modifié après la création du pool.

> [!IMPORTANT]
> Si vous spécifiez un modèle de licence pour l’instance qui est différent de celui du pool, le prix du pool est utilisé et la valeur au niveau de l’instance est ignorée.

Si vous créez des pools d’instances sur des [abonnements éligibles à l’avantage dev-test](https://azure.microsoft.com/pricing/dev-test/), vous bénéficiez automatiquement de tarifs réduits jusqu’à 55 % sur Azure SQL Managed Instance.

Pour plus d’informations sur les tarifs des pools d’instances, consultez la section *Pools d’instances* dans la [page des tarifs de SQL Managed Instance](https://azure.microsoft.com/pricing/details/sql-database/managed/).

## <a name="next-steps"></a>Étapes suivantes

- Pour vous familiariser avec les pools d’instances, consultez le [Guide de procédures des pools SQL Managed Instance](instance-pools-configure.md).
- Pour savoir comment créer votre première instance managée, consultez le [Guide de démarrage rapide](instance-create-quickstart.md).
- Pour consulter la liste des fonctionnalités et les comparer, consultez [Fonctionnalités Azure SQL communes](../database/features-comparison.md).
- Pour plus d’informations sur la configuration du réseau virtuel, consultez [Configuration de réseau virtuel SQL Managed Instance](connectivity-architecture-overview.md).
- Pour obtenir un guide de démarrage rapide qui crée une instance managée et restaure une base de données à partir d’un fichier de sauvegarde, consultez [Créer une instance managée](instance-create-quickstart.md).
- Pour accéder à un tutoriel expliquant comment utiliser Azure Database Migration Service pour la migration, consultez [Migration SQL Managed Instance à l’aide d’Azure Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md).
- Pour une supervision avancée des performances de base de données SQL Managed Instance avec des informations de dépannage intégrées, consultez [Superviser Azure SQL Managed Instance avec Azure SQL Analytics](../../azure-monitor/insights/azure-sql.md).
- Pour obtenir des informations sur les prix, consultez [Tarification SQL Managed Instance](https://azure.microsoft.com/pricing/details/sql-database/managed/).
