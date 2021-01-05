---
title: Forum Aux Questions (FAQ)
titleSuffix: Azure SQL Managed Instance
description: Forum aux questions sur Azure SQL Managed Instance
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein
ms.date: 09/21/2020
ms.openlocfilehash: 6b217e77310224779ea3ea840e613e28da6c86a3
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92779864"
---
# <a name="azure-sql-managed-instance-frequently-asked-questions-faq"></a>Forum aux questions sur Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Cet article contient les questions les plus fréquentes sur [Azure SQL Managed Instance](sql-managed-instance-paas-overview.md).

## <a name="supported-features"></a>Fonctionnalités prises en charge

**Où puis-je trouver une liste de fonctionnalités prises en charge sur SQL Managed Instance ?**

Pour obtenir la liste des fonctionnalités prises en charge dans SQL Managed Instance, consultez [Fonctionnalités d’Azure SQL Managed Instance](../database/features-comparison.md).

Pour connaître les différences de syntaxe et de comportement entre Azure SQL Managed Instance et SQL Server, consultez [Différences T-SQL par rapport à SQL Server](transact-sql-tsql-differences-sql-server.md).


## <a name="technical-specification-resource-limits-and-other-limitations"></a>Spécifications techniques, limites des ressources et autres limitations
 
**Où puis-je trouver des informations sur les caractéristiques techniques et les limites de ressources pour SQL Managed Instance ?**

Pour voir les caractéristiques des générations de matériel disponible, consultez [Différences techniques dans les générations de matériel](resource-limits.md#hardware-generation-characteristics).
Pour voir les niveaux de service disponibles et leurs caractéristiques, consultez [Différences techniques entre les niveaux de service](resource-limits.md#service-tier-characteristics).

**À quel niveau de service suis-je éligible ?**

Tout client est éligible à n’importe quel niveau de service. Cependant, si vous souhaitez échanger vos licences existantes et bénéficier de tarifs réduits sur Azure SQL Managed Instance grâce à [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/), gardez à l’esprit que les clients SQL Server Enterprise Edition avec Software Assurance sont éligibles aux niveaux de performance [Usage général](../database/service-tier-general-purpose.md) ou [Critique pour l’entreprise](../database/service-tier-business-critical.md) et que les clients SQL Server Standard Edition avec Software Assurance sont éligibles uniquement au niveau de performance Usage général. Pour plus d’informations, consultez [Droits spécifiques associés à AHB](../azure-hybrid-benefit.md?tabs=azure-powershell#what-are-the-specific-rights-of-the-azure-hybrid-benefit-for-sql-server).

**Quels types d’abonnement sont pris en charge pour SQL Managed Instance ?**

Pour voir la liste des types d’abonnements pris en charge, consultez [Types d’abonnements pris en charge](resource-limits.md#supported-subscription-types). 

**Quelles sont les régions Azure prises en charge ?**

Il est possible de créer des instances managées dans la plupart des régions Azure ; consultez [Régions prises en charge pour SQL Managed Instance](https://azure.microsoft.com/global-infrastructure/services/?products=sql-database&regions=all). Si vous avez besoin d’une instance managée dans une région qui n’est actuellement pas prise en charge, [envoyez une demande de support via le portail Azure](../database/quota-increase-request.md).

**Existe-t-il des limitations de quota pour les déploiements SQL Managed Instance ?**

Par défaut, une instance managée présente deux limites : une limite sur le nombre de sous-réseaux que vous pouvez utiliser et une limite sur le nombre de vCores que vous pouvez provisionner. Les limites varient en fonction du type d’abonnement et de la région. Pour voir la liste des limitations des ressources régionales par type d’abonnement, consultez le tableau de la section [Limitations des ressources régionales](resource-limits.md#regional-resource-limitations). Il s’agit de limites non strictes qui peuvent être relevées sur demande. Si vous avez besoin de provisionner davantage d’instances managées dans vos régions actuelles, envoyez une demande de support pour augmenter le quota à partir du portail Azure. Pour plus d’informations, consultez [Demander des augmentations de quota pour Azure SQL Database](../database/quota-increase-request.md).

**Puis-je demander à ce que le nombre maximal de bases de données (100) soit relevé pour mon instance managée ?**

Non et l’augmentation du nombre de bases de données pour SQL Managed Instance n’est pas envisagée pour l’instant. 

**Où puis-je effectuer une migration si j’ai plus de 8 TO de données ?**
Vous pouvez envisager de migrer vers d’autres versions d’Azure plus adaptées à votre charge de travail : [Azure SQL Database Hyperscale](../database/service-tier-hyperscale.md) ou [SQL Server sur les machines virtuelles Azure](../virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md).

**Où puis-je effectuer une migration si j’ai des exigences matérielles spécifiques, telles qu’un ratio RAM/vCore plus élevé ou davantage de processeurs ?**
Vous pouvez envisager de migrer vers [SQL Server sur les machines virtuelles Azure](../virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md) ou [Azure SQL Database](../database/sql-database-paas-overview.md) avec un ratio mémoire/processeur optimisé.

## <a name="known-issues-and-defects"></a>Problèmes et défauts connus

**Où puis-je trouver des informations sur les problèmes et les défauts connus ?**

Pour les défauts et les problèmes connus, consultez [Problèmes connus](../database/doc-changes-updates-release-notes.md#known-issues).

## <a name="new-features"></a>Nouvelles fonctionnalités

**Où puis-je trouver les fonctionnalités les plus récentes et les fonctionnalités disponibles en préversion publique ?**

Pour les fonctionnalités nouvelles et d’évaluation, consultez [Notes de publication](../database/doc-changes-updates-release-notes.md?tabs=managed-instance).

## <a name="create-update-delete-or-move-sql-managed-instance"></a>Créer, mettre à jour, supprimer ou déplacer une instance managée SQL

**Comment provisionner une instance managée SQL ?**

Vous pouvez provisionner une instance à partir du [Portail Azure](instance-create-quickstart.md), de [PowerShell](scripts/create-configure-managed-instance-powershell.md), d’[Azure CLI](https://techcommunity.microsoft.com/t5/azure-sql-database/create-azure-sql-managed-instance-using-azure-cli/ba-p/386281) et de [modèles ARM](/archive/blogs/sqlserverstorageengine/creating-azure-sql-managed-instance-using-arm-templates).

**Puis-je provisionner des instances managées dans un abonnement existant ?**

Oui, vous pouvez provisionner une instance managée dans un abonnement existant si cet abonnement appartient aux [types d’abonnements pris en charge](resource-limits.md#supported-subscription-types).

**Pourquoi ne puis-je pas provisionner une instance managée dans le sous-réseau dont le nom commence par un chiffre ?**

Il s’agit d’une limitation actuelle du composant sous-jacent qui vérifie le nom du sous-réseau par rapport à l’expression régulière ^[a-zA-Z_][^\\\/\:\*\?\"\<\>\|\`\'\^]*(?<![\.\s])$. Tous les noms qui réussissent le test de l’expression régulière et qui sont des noms de sous-réseau valides sont actuellement pris en charge.

**Comment faire pour mettre à l’échelle mon instance managée ?**

Vous pouvez mettre à l’échelle votre instance gérée à partir du [Portail Azure](../database/service-tiers-vcore.md?tabs=azure-portal#selecting-a-hardware-generation), de [PowerShell](/archive/blogs/sqlserverstorageengine/change-size-azure-sql-managed-instance-using-powershell), d’[Azure CLI](/cli/azure/sql/mi?view=azure-cli-latest#az-sql-mi-update&preserve-view=true) ou de [modèles ARM](/archive/blogs/sqlserverstorageengine/updating-azure-sql-managed-instance-properties-using-arm-templates).

**Puis-je déplacer mon instance managée d’une région vers une autre ?**

Oui, vous pouvez. Pour obtenir des instructions, consultez [Déplacer des ressources entre régions](../database/move-resources-across-regions.md).

**Comment faire pour supprimer mon instance managée ?**

Vous pouvez supprimer les instances gérées via le Portail Azure, [PowerShell](/powershell/module/az.sql/remove-azsqlinstance?preserve-view=true&view=azps-4.3.0), [Azure CLI](/cli/azure/sql/mi?view=azure-cli-latest#az-sql-mi-delete&preserve-view=true) ou les [API REST Resource Manager](/rest/api/sql/managedinstances/delete).

**Combien de temps faut-il pour créer ou mettre à jour une instance ou pour restaurer une base de données ?**

Le temps nécessaire à la création d’une instance managée ou au changement du niveau de service (vCores, stockage) dépend de plusieurs facteurs. Consultez [Opérations de gestion](sql-managed-instance-paas-overview.md#management-operations).
 
## <a name="naming-conventions"></a>Conventions d’affectation de noms

**Une instance managée peut-elle avoir le même nom qu’une instance SQL Server locale ?**

Il n’est pas possible de changer le nom d’une instance managée.

**Puis-je modifier le préfixe de zone DNS ?**

Oui, la zone DNS par défaut de l’instance managée *.database.windows.net* peut être modifiée. 

Pour utiliser une autre zone DNS au lieu de la valeur par défaut, par exemple, *.contoso.com*  : 
- Utilisez CliConfig pour définir un alias. L’outil étant simplement un wrapper de paramètres du registre, cela peut aussi se faire à l’aide d’une stratégie de groupe ou d’un script.
- Utilisez *CNAME* avec l’option *TrustServerCertificate=true* .

## <a name="migration-options"></a>Options de migration

**Comment puis-je migrer d’un pool unique ou élastique Azure SQL Database vers SQL Managed Instance ?**

L’instance gérée offre les mêmes niveaux de performance par calcul et taille de stockage que les autres options de déploiement d’Azure SQL Database. Si vous souhaitez consolider les données sur une seule instance ou si vous avez simplement besoin d’une fonctionnalité prise en charge exclusivement dans une instance gérée, vous pouvez migrer vos données en utilisant la fonctionnalité d’export/import (BACPAC). Voici d’autres façons de prendre en compte la migration SQL Database vers SQL Managed Instance : 
- Utilisation d'une [Source de données externe](https://techcommunity.microsoft.com/t5/azure-database-support-blog/lesson-learned-129-using-data-source-external-from-azure-sql/ba-p/1443210)
- Utilisation de [SQLPackage](https://techcommunity.microsoft.com/t5/azure-database-support-blog/how-to-migrate-azure-sql-database-to-azure-sql-managed-instance/ba-p/369182)
- Utilisation de la commande [BCP](https://medium.com/azure-sqldb-managed-instance/migrate-from-azure-sql-managed-instance-using-bcp-674c92efdca7)

**Comment puis-je migrer la base de données de mon instance vers une base de données Azure SQL Database unique ?**

Une option consiste à [exporter la base de données vers un fichier BACPAC](../database/database-export.md), puis à [importer le fichier BACPAC](../database/database-import.md). Cette approche est recommandée si votre base de données est inférieure à 100 Go.

La [réplication transactionnelle](replication-two-instances-and-sql-server-configure-tutorial.md?view=sql-server-2017&preserve-view=true) peut être utilisée si tous les tableaux dans la base de données ont des clés *primaires* et s’il n'y a pas d'objets OLTP en mémoire.

Les sauvegardes natives COPY_ONLY prises à partir d’une instance managée ne peuvent pas être restaurées sur SQL Server car l’instance managée a une version de base de données supérieure à celle de SQL Server. Pour plus d’informations, consultez [Sauvegarde en copie seule](/sql/relational-databases/backup-restore/copy-only-backups-sql-server?preserve-view=true&view=sql-server-ver15).

**Comment puis-je migrer mon instance SQL Server vers SQL Managed Instance ?**

Pour migrer votre instance de SQL Server, consultez [Migration d’instance SQL Server vers Azure SQL Managed Instance](migrate-to-instance-from-sql-server.md).

**Comment puis-je migrer des données d’autres plateformes vers SQL Managed Instance ?**

Pour obtenir des informations sur la migration à partir d’autres plateformes, consultez le [Guide de migration des bases de données Azure](https://datamigration.microsoft.com/).

## <a name="switch-hardware-generation"></a>Basculer la génération de matériel 

**Puis-je basculer la génération de matériel de mon instance managée entre Gen 4 et Gen 5 en ligne ?**

La commutation en ligne automatisée de la Gen4 à la Gen5 est possible si le matériel de Gen5 est disponible dans la même région où votre instance managée est approvisionnée. Dans ce cas, vous pouvez consulter la [page Présentation du modèle vCore](../database/service-tiers-vcore.md) qui explique comment basculer entre les générations de matériel.

Il s’agit d’une opération de longue durée car la nouvelle instance gérée sera approvisionnée en arrière-plan et les bases de données seront automatiquement transférées entre l’ancienne et la nouvelle instance, avec un rapide basculement au terme du processus.

Remarque : Le matériel Gen4 est en cours de retrait et n’est plus disponible pour les nouveaux déploiements. Toutes les nouvelles bases de données doivent être déployées sur le matériel Gen5. Le passage de Gen5 à Gen4 n’est pas non plus disponible.

## <a name="performance"></a>Performances 

**Comment puis-je comparer les performances de Managed Instance à celles de SQL Server ?**

Pour une comparaison des performances entre une instance gérée et SQL Server, commencez par consulter l'article [Meilleures pratiques pour comparer les performances entre une instance gérée Azure SQL et SQL Server](https://techcommunity.microsoft.com/t5/azure-sql-database/the-best-practices-for-performance-comparison-between-azure-sql/ba-p/683210).

**Quelles sont les causes des différences entre Managed Instance et SQL Server ?**

Consultez [Causes principales des différences entre SQL Managed Instance et SQL Server](https://azure.microsoft.com/blog/key-causes-of-performance-differences-between-sql-managed-instance-and-sql-server/). Pour plus d’informations sur l’impact de la taille du fichier journal sur les performances d’usage général de Managed Instance, consultez [Impact de la taille du fichier journal sur l’usage général](https://medium.com/azure-sqldb-managed-instance/impact-of-log-file-size-on-general-purpose-managed-instance-performance-21ad170c823e).

**Comment régler les performances de mon instance gérée ?**

Vous pouvez optimiser les performances de votre instance managée en procédant comme suit :
- Le [réglage automatique](../database/automatic-tuning-overview.md) fournit une optimisation des performances et une stabilisation des charges de travail grâce à un réglage continu des performances basé sur l’intelligence artificielle et le machine learning.
-   L’[OLTP en mémoire](../in-memory-oltp-overview.md) améliore le débit et la latence sur les charges de travail de traitement transactionnel et fournit des analyses plus rapides. 

Pour affiner encore davantage les performances, envisagez d’appliquer certaines des *bonnes pratiques* pour [optimiser les applications et des bases de données](../database/performance-guidance.md#tune-your-database).
Si votre charge de travail est constituée d’un grand nombre de petites transactions, envisagez de [passer le type de connexion du mode proxy au mode redirection](connection-types-overview.md#changing-connection-type) pour moins de latence et un débit plus élevé.

## <a name="monitoring-metrics-and-alerts"></a>Supervision, indicateurs de performances et alertes

**Quelles sont les options de surveillance et d’alerte pour mon instance managée ?**

Pour connaître toutes les options possibles pour surveiller et alerter sur la consommation et les performances de SQL Managed Instance, consultez la [publication sur les options d’analyse d’Azure SQL Managed Instance de notre blog](https://techcommunity.microsoft.com/t5/azure-sql-database/monitoring-options-available-for-azure-sql-managed-instance/ba-p/1065416). Pour la surveillance des performances en temps réel pour SQL MI, consultez [Surveillance des performances en temps réel pour Azure SQL DB Managed Instance](/archive/blogs/sqlcat/real-time-performance-monitoring-for-azure-sql-database-managed-instance).

**Puis-je utiliser le générateur de profils SQL pour le suivi des performances ?**

Oui, le générateur de profils SQL est pris en charge ou SQL Managed Instance. Pour plus d'informations, consultez [SQL Profiler](/sql/tools/sql-server-profiler/sql-server-profiler?preserve-view=true&view=sql-server-ver15).

**Database Advisor et Query Performance Insight sont-ils pris en charge pour les bases de données Managed Instance ?**

Ils ne sont pas prises en charge. Vous pouvez utiliser [DMV](../database/monitoring-with-dmvs.md) et le [Magasin de données des requêtes](/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store?preserve-view=true&view=sql-server-ver15) avec [SQL Profiler](/sql/tools/sql-server-profiler/sql-server-profiler?preserve-view=true&view=sql-server-ver15) et [XEvents](/sql/relational-databases/extended-events/extended-events?preserve-view=true&view=sql-server-ver15) pour surveiller vos bases de données.

**Puis-je créer des alertes basées sur les indicateurs de performances pour SQL Managed Instance ?**

Oui. Pour obtenir des instructions, consultez [Créer des alertes pour SQL Managed Instance](alerts-create.md).

**Puis-je créer des alertes pour ma base de données dans SQL Managed Instance ?**

Les alertes liées aux indicateurs de performances ne sont pas disponibles pour Managed Instance uniquement. Les métriques d’alerte pour les bases de données individuelles dans l’instance gérée ne sont pas disponibles.

## <a name="storage-size"></a>Taille de stockage

**Quelle est la taille de stockage maximale pour SQL Managed Instance ?**

La taille de stockage pour SQL Managed Instance dépend du niveau de service sélectionné (usage général ou critique pour l’entreprise). Pour connaître les limitations de stockage de ces niveaux de service, consultez [Caractéristiques de niveau de service](../database/service-tiers-general-purpose-business-critical.md).

**Quelle est la taille de stockage minimale disponible pour une instance managée ?**

La quantité minimale de stockage disponible dans une instance est de 32 Go. Le stockage peut être ajouté par incréments de 32 Go jusqu’à la taille de stockage maximale. Les 32 premiers Go sont gratuits.

**Puis-je augmenter l’espace de stockage affecté à une instance, indépendamment des ressources de calcul ?**

Oui, vous pouvez acheter le stockage de module complémentaire, indépendamment des ressources de calcul, dans une certaine mesure. Voir *Stockage maximal réservé aux instances* dans le [tableau](resource-limits.md#hardware-generation-characteristics).

**Comment puis-je optimiser les performances de mon stockage quant au niveau de service de l’usage général ?**

Pour optimiser les performances de stockage, consultez [Bonnes pratiques de stockage d’usage général](https://techcommunity.microsoft.com).

## <a name="backup-and-restore"></a>Sauvegarde et restauration

**Le stockage de sauvegarde est-il déduit de l’espace de stockage de mon instance managée ?**

Non, le stockage de sauvegarde n’est pas déduit de votre espace de stockage d’instance gérée. Le stockage de sauvegarde est indépendant de l’espace de stockage d’instance et n’est pas limité en taille. Le stockage de sauvegarde est limité par la durée de rétention de la sauvegarde de vos bases de données d’instance configurables jusqu’à 35 jours. Pour plus d’informations. consultez [Sauvegardes automatisées](../database/automated-backups-overview.md).

**Comment puis-je voir quand des sauvegardes automatisées sont effectuées sur mon instance managée ?**

Pour suivre les sauvegardes automatisées effectuées sur Managed Instance, consultez [Guide pratique pour suivre les sauvegardes automatisées pour Azure SQL Managed Instance](https://techcommunity.microsoft.com/t5/azure-database-support-blog/lesson-learned-128-how-to-track-the-automated-backup-for-an/ba-p/1442355).

**La sauvegarde à la demande est-elle prise en charge ?**

Oui, vous pouvez créer une sauvegarde complète en copie seule dans le stockage Blob Azure, mais elle ne peut être restaurée que dans Managed Instance. Pour plus d’informations, consultez [Sauvegarde en copie seule](/sql/relational-databases/backup-restore/copy-only-backups-sql-server?preserve-view=true&view=sql-server-ver15). Cependant, la sauvegarde en copie seule est impossible si la base de données est chiffrée par le TDE managé par le service, car le certificat utilisé pour le chiffrement est inaccessible. Dans ce cas-là, utilisez la fonctionnalité de restauration dans le temps pour déplacer la base de données vers un autre service SQL Managed Instance ou basculer vers une clé gérée par le client.

**La restauration native (des fichiers. bak) est-elle prise en charge par Managed Instance ?**

Oui, elle est prise en charge et disponible pour les versions SQL Server 2005 et ultérieures.  Pour utiliser la restauration native, chargez votre fichier. bak dans le stockage Blob Azure et exécutez des commandes T-SQL. Pour plus d’informations, consultez [Restauration native à partir d’une URL](./migrate-to-instance-from-sql-server.md#native-restore-from-url).

## <a name="business-continuity"></a>Continuité de l’activité

**Les bases de données système sont-elles répliquées vers l’instance secondaire dans un groupe de basculement ?**

Les bases de données système ne sont pas répliquées vers l’instance secondaire dans un groupe de basculement. Par conséquent, les scénarios qui dépendent des objets des bases de données système ne peuvent pas être appliqués sur l’instance secondaire, à moins que ces objets ne soient créés manuellement sur cette dernière. Pour contourner ce problème, consulter [Activer les scénarios dépendant des objets des bases de données système](../database/auto-failover-group-overview.md?tabs=azure-powershell#enable-scenarios-dependent-on-objects-from-the-system-databases).
 
## <a name="networking-requirements"></a>Configuration requise du réseau 

**Quelles sont les contraintes NSG entrantes/sortantes actuelles sur le sous-réseau Managed Instance ?**

Les règles NSG et UDR obligatoires sont documentées [ici](connectivity-architecture-overview.md#mandatory-inbound-security-rules-with-service-aided-subnet-configuration) et définies automatiquement par le service.
Ne perdez pas de vue que ces règles sont simplement celles dont nous avons besoin pour tenir à jour le service. Pour vous connecter à Managed Instance et utiliser des fonctionnalités différentes, vous devez définir des règles supplémentaires propres aux fonctionnalités, que vous devez tenir à jour.

**Comment puis-je définir des règles de groupe de sécurité réseau entrantes sur les ports de gestion ?**

La définition de règles au niveau des ports de gestion incombe à SQL Managed Instance. Pour cela, le service utilise une fonctionnalité nommée [Configuration de sous-réseau assistée par le service](connectivity-architecture-overview.md#service-aided-subnet-configuration).
Elle assure un flux ininterrompu de trafic de gestion pour satisfaire un contrat SLA.

**Puis-je obtenir les plages d’adresses IP sources utilisées pour le trafic de gestion entrant ?**

Oui. Vous pouvez analyser le trafic en provenance de votre groupe de sécurité réseau en [configurant des journaux de flux Network Watcher](../../network-watcher/network-watcher-monitoring-overview.md#analyze-traffic-to-or-from-a-network-security-group).

**Puis-je définir le groupe de sécurité réseau (NSG) de façon à contrôler l’accès au point de terminaison de données (port 1433) ?**

Oui. Dès lors qu’une instance managée est provisionnée, vous pouvez définir le groupe de sécurité réseau qui contrôle l’accès entrant au port 1433. Il est recommandé de limiter autant que possible sa plage d’adresses IP.

**Puis-je définir l’appliance virtuelle réseau ou le pare-feu local de façon à filtrer le trafic de gestion sortant en fonction des noms de domaine complets ?**

Non. Ce n’est pas pris en charge pour plusieurs raisons :
-   Le routage du trafic qui représente la réponse à la demande de gestion entrante serait asymétrique et ne pourrait pas fonctionner.
-   Le routage du trafic à destination du stockage serait affecté par les contraintes de débit et la latence, ce qui ne nous permettrait pas d’assurer la disponibilité et la qualité de service attendues.
-   L’expérience a montré que ces configurations étaient sujettes à erreurs et ne sont pas acceptables.

**Puis-je définir l’appliance virtuelle réseau ou le pare-feu pour le trafic sortant non liée à la gestion ?**

Oui. Le moyen le plus simple de procéder est d’ajouter la règle 0/0 à un itinéraire défini par l’utilisateur associé au sous-réseau de l’instance managée pour faire transiter le trafic par l’appliance virtuelle réseau.
 
**De combien d’adresses IP ai-je besoin pour une instance managée ?**

Le sous-réseau doit avoir un nombre suffisant d’[adresses IP](connectivity-architecture-overview.md#network-requirements) disponibles. Pour déterminer la taille de sous-réseau d’un réseau virtuel pour SQL Managed Instance, consultez [Déterminer la taille et la plage de sous-réseau requises pour Managed Instance](./vnet-subnet-determine-size.md). 

**Que se passe-t-il s’il n’y a pas suffisamment d’adresses IP pour effectuer une opération de mise à jour d’instance ?**

S’il n’y a pas suffisamment d’[adresses IP](connectivity-architecture-overview.md#network-requirements) dans le sous-réseau où votre instance managée est provisionnée, vous devez créer un nouveau sous-réseau et une nouvelle instance managée dans celui-ci. Nous vous suggérons aussi de créer le nouveau sous-réseau avec davantage d’adresses IP allouées pour éviter que des situations similaires se représentent lors des futures opérations de mise à jour. Une fois la nouvelle instance provisionnée, vous pouvez sauvegarder et restaurer manuellement les données entre l’ancienne instance et la nouvelle ou effectuer une [restauration dans le temps](point-in-time-restore.md?tabs=azure-powershell) entre les instances.

**Ai-je besoin d’un sous-réseau vide pour créer une instance managée ?**

Non. Vous pouvez utiliser un sous-réseau vide ou un sous-réseau qui contient déjà une ou plusieurs instances managées. 

**Puis-je modifier la plage d’adresses de sous-réseau ?**

Non si des instances managées s’y trouvent. Il s’agit d’une limitation de l’infrastructure réseau Azure. Vous êtes seulement autorisé à [ajouter un espace d’adressage supplémentaire à un sous-réseau vide](../../virtual-network/virtual-network-manage-subnet.md#change-subnet-settings). 

**Puis-je déplacer mon instance managée vers un autre sous-réseau ?**

Non. Il s’agit d’une limitation de conception actuelle de Managed Instance. En revanche, vous pouvez provisionner une nouvelle instance dans un autre sous-réseau pour ensuite sauvegarder et restaurer manuellement les données entre l’ancienne instance et la nouvelle ou effectuer une [restauration dans le temps](point-in-time-restore.md?tabs=azure-powershell) entre les instances.

**Ai-je besoin d’un réseau virtuel vide pour créer une instance managée ?**

Ce n’est pas obligatoire. Vous pouvez [créer un réseau virtuel pour Azure SQL Managed Instance](./virtual-network-subnet-create-arm-template.md) ou [configurer un réseau virtuel existant pour Azure SQL Managed Instance](./vnet-existing-add-subnet.md).

**Puis-je placer une instance managée dans un sous-réseau où figurent d’autres services ?**

Non. Pour l’heure, nous ne prenons pas en charge le placement d’une instance managée dans un sous-réseau qui contient déjà d’autres types de ressources.

## <a name="connectivity"></a>Connectivité 

**Puis-je me connecter à mon instance managée à l’aide d’une adresse IP ?**

Non, ce n’est pas pris en charge. Le nom d’hôte Managed Instance est mappé à un équilibreur de charge devant le cluster virtuel Managed Instance. Comme un cluster virtuel peut héberger plusieurs instances managées, la connexion ne peut pas être acheminée vers Managed Instance appropriée si son nom n’est pas spécifié.
Pour plus d’informations sur l’architecture de cluster virtuel SQL Managed Instance, consultez [Architecture de la connectivité du cluster virtuel](connectivity-architecture-overview.md#virtual-cluster-connectivity-architecture).

**Mon instance managée peut-elle avoir une adresse IP statique ?**

Non pris en charge actuellement.

Dans des situations rares mais nécessaires, nous pourrions avoir besoin de faire une migration en ligne d’une instance gérée vers un nouveau cluster virtuel. Si nécessaire, cette migration est due à des changements dans notre pile technologique visant à améliorer la sécurité et la fiabilité du service. La migration vers un nouveau cluster virtuel entraîne la modification de l’adresse IP associée au nom d’hôte de l’instance gérée. Le service d’instance gérée ne réclame pas le support des adresses IP statiques et se réserve le droit de les modifier sans préavis dans le cadre des cycles de maintenance réguliers.

Pour cette raison, nous déconseillons fortement de se fier à l’immuabilité de l’adresse IP car cela pourrait causer des temps d’arrêt inutiles.

**Managed Instance a-t-il un point de terminaison public ?**

Oui. Managed Instance dispose d’un point de terminaison public qui est utilisé par défaut uniquement pour le management des services, mais un client peut également l’activer pour l’accès aux données. Pour plus d’informations, consultez [Utiliser SQL Managed Instance avec des points de terminaison publics](./public-endpoint-overview.md). Pour configurer un point de terminaison public, accédez à [Configurer un point de terminaison public dans SQL Managed Instance](public-endpoint-configure.md).

**Comment Managed Instance contrôle l’accès au point de terminaison public ?**

Managed Instance contrôle l’accès au point de terminaison public au niveau du réseau et de l’application.

Les services de gestion et de déploiement se connectent à une instance gérée en utilisant un [point de terminaison de gestion](./connectivity-architecture-overview.md#management-endpoint) qui est mappé à un équilibreur de charge externe. Le trafic est routé vers les nœuds seulement s’il est reçu sur un ensemble de ports prédéfinis utilisés exclusivement par les composants de gestion de l’instance gérée. Un pare-feu intégré sur les nœuds est configuré de manière à autoriser le trafic provenant uniquement des plages d’adresses IP de Microsoft. Tous les certificats authentifient mutuellement les communications entre les composants de gestion et le plan de gestion. Pour en savoir plus, consultez [Architecture de connectivité pour SQL Managed Instance](./connectivity-architecture-overview.md#virtual-cluster-connectivity-architecture).

**Puis-je utiliser le point de terminaison public pour accéder aux données dans les bases de données Managed Instance ?**

Oui. Le client doit activer l’accès aux données du point de terminaison public à partir du [Portail Azure](public-endpoint-configure.md#enabling-public-endpoint-for-a-managed-instance-in-the-azure-portal) / [PowerShell](public-endpoint-configure.md#enabling-public-endpoint-for-a-managed-instance-using-powershell)/ARM et configurer NSG pour verrouiller l’accès au port de données (numéro de port 3342). Pour plus d’informations, consultez [Configurer un point de terminaison public dans SQL Managed Instance](public-endpoint-configure.md) et [Utiliser Azure SQL Managed Instance en toute sécurité avec un point de terminaison public](public-endpoint-overview.md). 

**Puis-je spécifier un port personnalisé pour un ou plusieurs terminaux de données SQL ?**

Non, cette option n'est pas disponible.  Pour le point de terminaison de données privé, Managed Instance utilise le numéro de port par défaut 1433 et pour le point de terminaison de données public, Managed Instance utilise le numéro de port par défaut 3342.

**Quelle est la méthode recommandée pour connecter des instances gérées de différentes régions ?**

Le peering des circuits de d’Express Route est le meilleur moyen pour y parvenir. L’appairage de réseaux virtuels mondiaux est pris en charge avec la limitation décrite dans la remarque ci-dessous.  

> [!IMPORTANT]
> [Le 22/09/2020, nous avons annoncé l’appairage de réseaux virtuels mondiaux pour les clusters virtuels nouvellement créés](https://azure.microsoft.com/en-us/updates/global-virtual-network-peering-support-for-azure-sql-managed-instance-now-available/). Cela signifie que l’appairage de réseaux virtuels mondiaux est pris en charge pour les instances gérées SQL créées dans des sous-réseaux vides après la date d’annonce, ainsi que pour toutes les instances gérées ultérieures créées dans ces sous-réseaux. Pour toutes les autres instances gérées SQL, la prise en charge de l’appairage est limitée aux réseaux de la même région en raison des [contraintes de l’appairage de réseaux virtuels mondiaux](../../virtual-network/virtual-network-manage-peering.md#requirements-and-constraints). Consultez également la section appropriée de l’article [Forum Aux Questions sur les réseaux virtuel Azure](../../virtual-network/virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers) pour plus d’informations. 

Si le Peering de circuits Express Route et l’appairage de réseaux virtuels mondiaux ne sont pas possibles, la seule autre option consiste à créer une connexion VPN site à site ([Portail Azure](../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md), [PowerShell](../../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md), [Azure CLI](../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-cli.md)).

## <a name="mitigate-data-exfiltration-risks"></a>Atténuer les risques liés à l’exfiltration de données  

**Comment atténuer les risques liés à l’exfiltration de données ?**

Afin d’atténuer les risques liés à l’exfiltration de données, il est recommandé aux clients d’appliquer un ensemble de paramètres et de contrôles de sécurité :

- Activez [Transparent Data Encryption (TDE)](../database/transparent-data-encryption-tde-overview.md) sur toutes les bases de données.
- Désactivez Common Language Runtime (CLR). Cela est recommandé localement également.
- Utilisez l’authentification Azure Active Directory (Azure AD) uniquement.
- Accédez à l’instance avec un compte DBA à faibles privilèges.
- Configurez l’accès au serveur de rebond (jumpbox) JIT pour le compte sysadmin.
- Activez l’[audit SQL](/sql/relational-databases/security/auditing/sql-server-audit-database-engine) et intégrez-le à des mécanismes d’alerte.
- Activez [Threat Detection](../database/threat-detection-configure.md) à partir de la suite [Azure Defender pour SQL](../database/azure-defender-for-sql.md).

## <a name="dns"></a>DNS

**Puis-je configurer un DNS personnalisé pour SQL Managed Instance ?**

Oui. Consultez le [guide pratique pour configurer un DNS personnalisé pour Azure SQL Managed Instance](./custom-dns-configure.md).

**Puis-je actualiser DNS ?**

Actuellement, nous ne fournissons pas de fonctionnalité pour actualiser la configuration du serveur DNS pour SQL Managed Instance.

La configuration DNS est actualisée par la suite :

- Lorsque le bail DHCP expire.
- À la mise à niveau de la plateforme.

En guise de solution de contournement, rétrogradez SQL Managed Instance à quatre vCore, puis remettez-le à niveau. Ceci a pour effet secondaire d’actualiser la configuration DNS.

## <a name="change-time-zone"></a>Changer le fuseau horaire

**Puis-je changer le fuseau horaire d’une instance managée existante ?**

La configuration de fuseau horaire peut être définie lorsqu’une instance gérée est configurée pour la première fois. Le fuseau horaire d’une instance managée existante ne peut pas être modifié. Pour plus d’informations, consultez [Limites liées aux fuseaux horaires](timezones-overview.md#limitations).

En guise de solutions de contournement, vous pouvez créer une nouvelle instance managée avec le fuseau horaire approprié, puis soit effectuer une sauvegarde et une restauration manuelles, soit (ce que nous recommandons) effectuer une [restauration à un moment donné entre les instances](/archive/blogs/sqlserverstorageengine/cross-instance-point-in-time-restore-in-azure-sql-database-managed-instance).


## <a name="security-and-database-encryption"></a>Sécurité et chiffrement de base de données

**Le rôle de serveur sysadmin est-il disponible pour SQL Managed Instance ?**

Oui, les clients peuvent créer des connexions membres du rôle sysadmin.  Les clients qui assument le privilège sysadmin peuvent aussi assumer la responsabilité d’exploiter l’instance, ce qui peut avoir un impact négatif sur l’engagement du contrat SLA. Pour ajouter une connexion au rôle de serveur sysadmin, consultez [Authentification Azure AD](./aad-security-configure-tutorial.md#azure-ad-authentication).

**La technologie Transparent Data Encryption est-elle prise en charge pour SQL Managed Instance ?**

Oui, Transparent Data Encryption est pris en charge pour SQL Managed Instance. Pour plus d’informations, consultez [Chiffrement transparent des données pour SQL Managed Instance](../database/transparent-data-encryption-tde-overview.md?tabs=azure-portal).

**Puis-je tirer parti du modèle BYOK (Bring Your Own Key) pour TDE ?**

Oui, le scénario Azure Key Vault pour BYOK est disponible pour Azure SQL Managed Instance. Pour plus d’informations, consultez [Chiffrement transparent des données avec une clé gérée par le client](../database/transparent-data-encryption-tde-overview.md?tabs=azure-portal#customer-managed-transparent-data-encryption---bring-your-own-key).

**Puis-je migrer une base de données SQL Server chiffrée ?**

Oui, vous pouvez. Pour migrer une base de données SQL Server chiffrée, vous devez exporter et importer vos certificats existants dans Managed Instance, puis effectuer une sauvegarde complète de la base de données et la restaurer dans Managed Instance. 

Vous pouvez aussi utiliser [Azure Database Migration Service](https://azure.microsoft.com/services/database-migration/) pour migrer les bases de données chiffrées avec TDE.

**Comment configurer la rotation du protecteur TDE pour SQL Managed Instance ?**

Vous pouvez faire tourner le protecteur TDE pour Managed Instance en utilisant Azure Cloud Shell. Pour obtenir des instructions, consultez [Transparent Data Encryption dans SQL Managed Instance à l’aide de votre propre clé Azure Key Vault](scripts/transparent-data-encryption-byok-powershell.md).

**Puis-je restaurer ma base de données chiffrée sur SQL Managed Instance ?**

Oui, vous n’avez pas besoin de déchiffrer votre base de données pour pouvoir la restaurer sur SQL Managed Instance. Vous devez toutefois fournir à SQL Managed Instance un certificat/une clé servant de protecteur de clé de chiffrement sur le système source pour pouvoir lire les données du fichier de sauvegarde chiffré. Il existe deux façons d'effectuer cette opération :

- *Chargez le protecteur de certificat dans SQL Managed Instance* . Cela est uniquement possible à l’aide de PowerShell. L’[exemple de script](./tde-certificate-migrate.md) décrit l’ensemble du processus.
- *Chargez le protecteur de clé asymétrique dans Azure Key Vault et faites pointer SQL Managed Instance vers celui-ci.* . Cette approche ressemble au cas d’usage TDE BYOK (Bring Your Own Key) qui utilise également l’intégration Key Vault pour stocker la clé de chiffrement. Si vous ne souhaitez pas utiliser la clé en tant que protecteur de clé de chiffrement, mais simplement la mettre à disposition de SQL Managed Instance pour restaurer les bases de données chiffrées, suivez les instructions pour [configurer BYOK TDE](../database/transparent-data-encryption-tde-overview.md#manage-transparent-data-encryption) et ne cochez pas la case **Définir la clé sélectionnée comme protecteur TDE par défaut** .

Une fois que vous mettez le protecteur de chiffrement à disposition de SQL Managed Instance, vous pouvez appliquer la procédure standard de restauration de la base de données.

## <a name="purchasing-models-and-benefits"></a>Modèles d’achat et avantages

**Quels sont les modèles d’achat disponibles pour SQL Managed Instance ?**

SQL Managed Instance propose le [modèle d’achat vCore](sql-managed-instance-paas-overview.md#vcore-based-purchasing-model).

**Quels sont les avantages proposés par SQL Managed Instance en matière de coûts ?**

Voici comment vous pouvez réduire les coûts grâce aux avantages Azure SQL :
-   Optimisez les investissements existants en licences locales et économisez jusqu’à 55 % avec [Azure Hybrid Benefit](../azure-hybrid-benefit.md?tabs=azure-powershell). 
-   Engagez-vous sur une réservation de ressources de calcul et économisez jusqu’à 33 % grâce à l’[avantage d’instance réservée](../database/reserved-capacity-overview.md). Combinez-le à Azure Hybrid Benefit pour bénéficier d’économies pouvant atteindre jusqu’à 82 %. 
-   Économisez jusqu’à 55 % par rapport aux prix catalogue grâce à l’[avantage Tarification Azure Dev/Test](https://azure.microsoft.com/pricing/dev-test/) qui offre des tarifs réduits pour vos charges de travail de développement et de test en cours.

**Qui est éligible à l’avantage d’instance réservée ?**

Pour bénéficier de l’avantage d’instance réservée, votre type d’abonnement doit être un contrat entreprise (numéros d’offre : MS-AZR-0017P ou MS-AZR-0148p) ou un accord individuel avec paiement à l’utilisation (numéros de l’offre : MS-AZR-0003P ou MS-AZR-0023P). Pour plus d’informations sur les réservations, consultez [Avantage d’instance réservée](../database/reserved-capacity-overview.md). 

**Est-il possible d’annuler, d’échanger ou de rembourser des réservations ?**

Vous pouvez annuler, échanger ou rembourser des réservations avec certaines limitations. Pour plus d’informations, consultez [Échanges et remboursements en libre-service pour les réservations Azure](../../cost-management-billing/reservations/exchange-and-refund-azure-reservations.md).

## <a name="billing-for-managed-instance-and-backup-storage"></a>Facturation pour Managed Instance et le stockage de sauvegarde

**Quelles sont les options tarifaires pour SQL Managed Instance ?**

Pour explorer les options tarifaires de Managed Instance, consultez la [page Tarification](https://azure.microsoft.com/pricing/details/azure-sql/sql-managed-instance/single/).

**Comment faire pour suivre la facturation de mon instance managée ?**

Vous pouvez le faire à l’aide de la [solution Azure Cost Management](../../cost-management-billing/index.yml). Accédez à **Abonnements** dans le [portail Azure](https://portal.azure.com) et sélectionnez **Analyse des coûts** . 

Utilisez l’option **Coûts cumulés** , puis filtrez en fonction du **type de ressource** comme `microsoft.sql/managedinstances`.

**Combien coûtent les sauvegardes automatisées ?**

Vous bénéficiez de la même quantité d’espace de stockage de sauvegarde gratuit que l’espace de stockage de données réservé acheté, quelle que soit la période de rétention de sauvegarde définie. Si votre consommation d’espace de stockage de sauvegarde se trouve dans les limites de l’espace de stockage de sauvegarde gratuit alloué, vous n’avez pas de coûts supplémentaires à supporter pour les sauvegardes automatisées d’une instance managée. Elles sont donc gratuites. Si l’utilisation de l’espace de stockage de sauvegarde dépasse la quantité d’espace gratuit, le coût par Go/mois dans les régions des États-Unis varie d’environ 0,20 à 0,24 USD. Vous trouverez le détail des tarifs pour votre région dans la page de tarification. Pour plus d’informations, consultez [Consommation du stockage de sauvegarde expliquée](https://techcommunity.microsoft.com/t5/azure-sql-database/backup-storage-consumption-on-managed-instance-explained/ba-p/1390923).

**Comment surveiller la facturation de ma consommation de stockage de sauvegarde ?**

Vous pouvez surveiller le coût du stockage de sauvegarde via le Portail Azure. Pour obtenir des instructions, consultez [Superviser le coût des sauvegardes automatisées](../database/automated-backups-overview.md?tabs=managed-instance#monitor-costs). 

**Comment optimiser les coûts de stockage des sauvegardes sur une instance managée ?**

Pour optimiser les coûts de stockage des sauvegardes, consultez [Fine backup tuning on SQL Managed Instance](https://techcommunity.microsoft.com/t5/azure-sql-database/fine-tuning-backup-storage-costs-on-managed-instance/ba-p/1390935).

## <a name="cost-saving-use-cases"></a>Cas d’usage de réduction des coûts

**Où puis-je trouver des cas d’usage et les économies de coûts qui en résultent avec SQL Managed Instance ?**

Études de cas SQL Managed Instance :

- [Komatsu](https://customers.microsoft.com/story/komatsu-australia-manufacturing-azure)
- [KMD](https://customers.microsoft.com/en-ca/story/kmd-professional-services-azure-sql-database)
- [PowerDETAILS](https://customers.microsoft.com/story/powerdetails-partner-professional-services-azure-sql-database-managed-instance)
- [Allscripts](https://customers.microsoft.com/story/allscripts-partner-professional-services-azure)

Pour mieux comprendre les avantages, les coûts et les risques associés au déploiement d’Azure SQL Managed Instance, une étude de Forrester est également disponible : [The Total Economic Impact of Microsoft Azure SQL Database Managed Instance](https://azure.microsoft.com/resources/forrester-tei-sql-database-managed-instance).

## <a name="password-policy"></a>Stratégie de mot de passe 

**Quelles sont les stratégies de mot de passe appliquées pour les connexions SQL Managed Instance ?**

La stratégie de mot de passe SQL Managed Instance pour les connexions SQL hérite des stratégies de plateforme Azure qui sont appliquées aux machines virtuelles formant un cluster virtuel contenant l’instance managée. À l’heure actuelle, il n’est pas possible de modifier ces paramètres, car ils sont définis par Azure et hérités par l’instance managée.

 > [!IMPORTANT]
 > La plateforme Azure peut modifier les exigences de stratégie sans en avertir les services qui reposent sur ces stratégies.

**Quelles sont les stratégies actuelles de la plateforme Azure ?**

Chaque connexion doit définir son mot de passe lors de la connexion, et le modifier une fois qu’il a atteint l’âge maximal.

| **Stratégie** | **Paramètre de sécurité** |
| --- | --- |
| Âge maximum du mot de passe | 42 jours |
| Âge minimum du mot de passe | 1 jour |
| Longueur minimale du mot de passe | 10 caractères |
| Le mot de passe doit respecter des exigences de complexité | activé |

**Est-il possible de désactiver la complexité et l’expiration des mots de passe dans SQL Managed Instance au niveau de la connexion ?**

Oui, vous pouvez contrôler les champs CHECK_POLICY et CHECK_EXPIRATION au niveau de la connexion. Vous pouvez vérifier les paramètres actuels en exécutant la commande T-SQL suivante :

```sql
SELECT *
FROM sys.sql_logins
```

Après cela, vous pouvez modifier les paramètres de connexion spécifiés en exécutant :

```sql
ALTER LOGIN <login_name> WITH CHECK_POLICY = OFF;
ALTER LOGIN <login_name> WITH CHECK_EXPIRATION = OFF;
```

(remplacez « test » par le nom de connexion souhaité et ajustez les valeurs de stratégie et d’expiration)


## <a name="service-updates"></a>Mises à jour de service

**Que représente le changement d’autorité de certification racine pour Azure SQL Database et SQL Managed Instance ?**

Voir [Rotation des certificats pour Azure SQL Database et SQL Managed Instance](../updates/ssl-root-certificate-expiring.md). 

**Qu’est-ce qu’un événement de maintenance planifiée pour SQL Managed Instance ?**

Consultez [Planifier les événements de maintenance Azure dans SQL Managed Instance](../database/planned-maintenance.md). 


## <a name="azure-feedback-and-support"></a>Commentaires et support Azure

**Où puis-je soumettre mes idées d’amélioration de SQL Managed Instance ?**

Vous pouvez voter pour une nouvelle fonctionnalité Managed Instance ou soumettre au vote une idée d’amélioration sur le [forum de commentaires SQL Managed Instance](https://feedback.azure.com/forums/915676-sql-managed-instance). De cette façon, vous pouvez contribuer au développement du produit et nous aider à hiérarchiser nos améliorations potentielles.

**Comment créer une demande de support Azure ?**

Pour savoir comment créer une demande de support Azure, consultez le [guide pratique pour créer une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md).