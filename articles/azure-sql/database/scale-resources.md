---
title: Mettre les ressources à l’échelle
description: Cet article explique comment mettre à l'échelle votre base de données dans Azure SQL Database et SQL Managed Instance en ajoutant ou en supprimant des ressources allouées.
services: sql-database
ms.service: sql-db-mi
ms.subservice: performance
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: wiassaf, sstein
ms.date: 06/25/2019
ms.openlocfilehash: 453d7e118b946d60eb3d84c6a66abdbea7db2410
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499218"
---
# <a name="dynamically-scale-database-resources-with-minimal-downtime"></a>Mettre à l’échelle de façon dynamique les ressources de base de données moyennant un temps d’arrêt minimal
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Azure SQL Database et SQL Managed Instance vous permet d’ajouter de manière dynamique plus de ressources à votre base de données avec un [temps d’arrêt](https://azure.microsoft.com/support/legal/sla/sql-database) minimal ; toutefois, il existe un délai de basculement durant lequel la connectivité à la base de données est perdue pendant un court laps de temps, qui peut être atténué à l’aide d’une logique de nouvelle tentative.

## <a name="overview"></a>Vue d’ensemble

Lorsque la demande ciblant votre application s’accroît de quelques appareils et clients à plusieurs millions, Azure SQL Database et SQL Managed Instance se mettent à l’échelle immédiatement avec un temps d’arrêt minimal. La scalabilité est une des caractéristiques les plus importantes de PaaS (plateforme en tant que service) qui vous permet d’ajouter plus de ressources de façon dynamique à votre service, si besoin. Azure SQL Database vous permet de modifier en toute simplicité vos ressources (alimentation processeur, mémoire, débit E/S et stockage) allouées à vos bases de données.

Vous pouvez limiter les problèmes de performances dus à une utilisation croissante de votre application qui ne peuvent être résolus en utilisant des méthodes de réécriture de requête ou d’indexation. L’ajout de ressources vous permet de réagir rapidement lorsqu’une base de données atteint la limite de ressources actuelles et a besoin de plus de puissance pour gérer les charges entrantes. Azure SQL Database vous permet aussi de diminuer la taille des ressources lorsqu’elles ne sont pas nécessaires afin de réduire les coûts.

Vous n’avez pas à vous inquiéter de l’achat de matériel et du changement de l’infrastructure sous-jacente. La mise à l'échelle d’une base de données peut être effectuée en toute simplicité via le portail Azure à l’aide d’un curseur.

![Performances de SQL Database](./media/scale-resources/scale-performance.svg)

Azure SQL Database offre le modèle d’achat DTU et le [modèle d’achat DTU](service-tiers-dtu.md) et le [modèle d'achat vCore](service-tiers-vcore.md), tandis que Azure SQL Managed Instance offre uniquement le [modèle d’achat vCore](service-tiers-vcore.md). 

- Le [modèle d’achat DTU](service-tiers-dtu.md) offre une combinaison de ressources de calcul, de mémoire et d’E/S réparties sur trois niveaux de service pour prendre en charge les charges de travail de base de données, aussi bien légères qu’importantes : De base, Standard et Premium. Les niveaux de performance de chaque niveau fournissent une combinaison différente de ces ressources, à laquelle vous pouvez ajouter d’autres ressources de stockage.
- Le [modèle d’achat vCore](service-tiers-vcore.md) vous permet de choisir le nombre de vCores, la quantité de mémoire et de stockage, ainsi que la vitesse de stockage. Ce modèle d’achat propose trois niveaux de service : Usage général, Critique pour l’entreprise et Hyperscale.

Vous pouvez créer votre première application dans une petite base de données unique pour un coût mensuel modique avec le niveau de service De base, Standard ou Usage général, puis remplacer ce niveau de service manuellement ou par programme à tout moment par le niveau de service Premium ou Critique pour l’entreprise, afin de répondre aux besoins de votre solution. Vous pouvez ajuster les performances sans perturber le fonctionnement de votre application, ni l’expérience de vos clients. L’évolutivité dynamique permet à votre base de données de répondre en toute transparence aux besoins en ressources qui évoluent sans cesse et de payer uniquement les ressources dont vous avez besoin, lorsque vous en avez besoin.

> [!NOTE]
> La scalabilité dynamique est différente de la mise à l’échelle automatique. La mise à l’échelle automatique survient quand un service se met à l’échelle automatiquement en fonction de critères, tandis que l’extensibilité dynamique permet la mise à l’échelle manuelle avec un temps d’arrêt minimal.

Des bases de données uniques dans Azure SQL Database prennent en charge la scalabilité dynamique manuelle, mais pas la mise à l’échelle automatique. Pour plus expérience plus *automatique*, envisagez d’utiliser des pools élastiques, ce qui permet aux bases de données de partager des ressources dans un pool en fonction de leurs besoins individuels.
Toutefois, il existe des scripts qui peuvent aider à automatiser la scalabilité pour une base de données unique dans Azure SQL Database. Pour obtenir un exemple, consultez la rubrique [Utiliser PowerShell pour surveiller et mettre à l’échelle une base de données SQL](scripts/monitor-and-scale-database-powershell.md).

Vous pouvez changer les [niveaux de service DTU](service-tiers-dtu.md) ou les [caractéristiques vCore](resource-limits-vcore-single-databases.md) à tout moment, avec un temps d’arrêt minimal de votre application (généralement inférieur à quatre secondes). Pour de nombreuses entreprises et applications, la possibilité de créer des bases de données et d’augmenter ou ralentir les performances à la demande se révèle suffisante, surtout si les modèles d’utilisation sont relativement prévisibles. Mais si vous avez des modèles d'utilisation imprévisibles, il peut être difficile de gérer les coûts et votre modèle commercial. Pour ce scénario, vous utilisez un pool élastique avec un certain nombre d’eDTU qui sont partagées entre plusieurs bases de données dans le pool.

![Présentation de SQL Database : DTU de base de données unique par couche et niveau](./media/scale-resources/single_db_dtus.png)

Azure SQL Database offre la possibilité de mettre à l'échelle vos bases de données de façon dynamique :

- Avec une [base de données unique](single-database-scale.md), vous pouvez utiliser des modèles [DTU](resource-limits-dtu-single-databases.md) ou [vCore](resource-limits-vcore-single-databases.md) pour définir le volume maximal de ressources assignées à chaque base de données.
- Les [pools élastiques](elastic-pool-scale.md) vous permettent de définir la limite de ressources maximale par groupe de bases de données dans le pool.

Azure SQL Managed Instance vous permet également une mise à l’échelle : 

- [SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md) utilise le mode [vCores](../managed-instance/sql-managed-instance-paas-overview.md#vcore-based-purchasing-model) et vous permet de définir le nombre maximum de cœurs UC et la quantité maximale de stockage alloués à votre instance. Toutes les bases de données au sein de l’instance gérée partageront les ressources allouées à l’instance.

Quelle que soit la version, une action de mise à l’échelle (scale up ou scale down) aurait pour effet de redémarrer le processus du moteur de base de données et de le déplacer si nécessaire vers une autre machine virtuelle. Le déplacement du processus du moteur de base de données vers une nouvelle machine virtuelle est un **processus en ligne**, ce qui vous permet de continuer à utiliser votre service Azure SQL Database existant pendant sa progression. Une fois le moteur de base de données cible entièrement initialisé et prêt à traiter les requêtes, les connexions sont [basculées du moteur de base de données source vers le moteur de base de données cible](single-database-scale.md#impact).

> [!NOTE]
> Une courte interruption de la connexion risque de se produire à la fin du processus de mise à l’échelle. Si vous avez implémenté une [Logique de nouvelles tentatives pour les erreurs temporaires standard](troubleshoot-common-connectivity-issues.md#retry-logic-for-transient-errors), vous ne remarquerez pas le basculement.

## <a name="alternative-scale-methods"></a>Autres méthodes de mise à l’échelle

La mise à l'échelle des ressources reste la façon la plus facile et efficace pour améliorer les performances de votre base de données sans changer le code de la base de données ou de l’application. Dans certains cas, même les niveaux de service, les tailles de calcul et les optimisations de performances les plus élevés peuvent ne pas suffire à gérer votre charge de travail correctement et de façon économique. Dans ce cas, vous disposez d’autres options pour mettre à l’échelle votre base de données :

- [Lecture du Scale-out](read-scale-out.md) est une fonctionnalité disponible qui vous offre un réplica en lecture seule de vos données, sur lequel vous pouvez exécutez des requêtes en lecture seule exigeantes telles que les rapports. Un réplica en lecture seule gère votre charge de travail en lecture seule sans affecter l’utilisation des ressources sur votre base de données principale.
- Le [partitionnement de base de données](elastic-scale-introduction.md) est un ensemble de techniques qui vous permet de diviser vos données en plusieurs bases de données pour les mettre à l'échelle indépendamment.

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur l’amélioration des performances d’une base de données en modifiant son code, consultez [Trouver et appliquer des recommandations de performances](database-advisor-find-recommendations-portal.md).
- Pour en savoir sur l’optimisation de votre base de données par une intelligence intégrée, consultez [Ajustement automatique](automatic-tuning-overview.md).
- Pour en savoir plus sur la fonctionnalité Lecture du Scale-out dans Azure SQL Database, consultez comment [utiliser des réplicas en lecture seule pour équilibrer les charges de travail de requêtes en lecture seule](read-scale-out.md).
- Pour en savoir plus sur le partitionnement de base de données, consultez [Scale-out avec Azure SQL Database](elastic-scale-introduction.md).
