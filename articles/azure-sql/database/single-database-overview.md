---
title: Qu'est-ce qu'une base de données unique ?
description: En savoir plus sur le type de ressource Base de données unique dans Azure SQL Database
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 04/08/2019
ms.openlocfilehash: 8f92fe8e4a4ebbc2d970bf28e415859249b9f67c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84343317"
---
# <a name="what-is-a-single-database-in-azure-sql-database"></a>Qu'est-ce qu'une base de données unique dans Azure SQL Database ?
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Le type de ressource Base de données unique crée dans Azure SQL Database une base de données dotée de ses propres ressources, et celle-ci est gérée par le biais d'un [serveur](logical-servers.md). Avec une seule base de données, chaque base de données est isolée et portable. Chacune a son propre niveau de service au sein du [modèle d’achat DTU](service-tiers-dtu.md) ou du [modèle d’achat vCore](service-tiers-vcore.md), ainsi qu’une taille de calcul garantie.

> [!IMPORTANT]
> Une base de données unique est des deux types de ressources d'Azure SQL Database. Le deuxième est [pools élastiques](elastic-pool-overview.md).

## <a name="dynamic-scalability"></a>Évolutivité dynamique

Vous pouvez créer votre première application sur une petite base de données unique à faible coût dans le niveau de calcul serverless ou une petite taille de calcul dans le niveau de calcul provisionné. Vous pouvez modifier le [niveau de calcul ou de service](single-database-scale.md) manuellement ou programmatiquement à tout moment pour répondre aux besoins de votre solution. Vous pouvez ajuster les performances sans perturber le fonctionnement de votre application, ni l’expérience de vos clients. L’évolutivité dynamique permet à votre base de données de répondre en toute transparence aux besoins en ressources qui évoluent sans cesse et de payer uniquement les ressources dont vous avez besoin, lorsque vous en avez besoin.

## <a name="single-databases-and-elastic-pools"></a>Bases de données uniques et pools élastiques

Une base de données unique peut être déplacée dans et hors d’un [pool élastique](elastic-pool-overview.md) en vue du partage de ressources. Pour de nombreuses entreprises et applications, la possibilité de créer des bases de données uniques et d’augmenter ou de ralentir les performances à la demande se révèle suffisante, surtout si les modèles d’utilisation sont relativement prévisibles. Mais si vous avez des modèles d'utilisation imprévisibles, il peut être difficile de gérer les coûts et votre modèle commercial. Les pools élastiques sont conçus pour résoudre ce problème. Le concept est simple. Vous allouez des ressources de performance à un pool plutôt qu’à une base de données individuelle et vous payez pour les ressources de performance d’un pool plutôt que pour celles d’une base de données unique.

## <a name="monitoring-and-alerting"></a>Surveillance et alerte

Vous utilisez les outils intégrés de [surveillance](performance-guidance.md) et [d’alerte](alerts-insights-configure-portal.md) de performances, combinées avec les évaluations de performance. Ces outils vous permettent d’évaluer rapidement l’impact des mises à l’échelle (montées ou descentes en charge) en fonction de vos besoins en performances actuels ou pour un projet. De plus, SQL Database peut [émettre des métriques et des journaux de ressources](metrics-diagnostic-telemetry-logging-streaming-export-configure.md) pour faciliter la supervision.

## <a name="availability-capabilities"></a>Fonctionnalités de disponibilité

Les bases de données uniques et les pools élastiques fournissent un grand nombre de caractéristiques de disponibilité. Pour plus d’informations, consultez [Caractéristiques de disponibilité](sql-database-paas-overview.md#availability-capabilities).

## <a name="transact-sql-differences"></a>Différences Transact-SQL

La plupart des fonctionnalités Transact-SQL utilisées par les applications sont entièrement prises en charge dans Microsoft SQL Server et Azure SQL Database. Par exemple, les principaux composants SQL tels que les types de données, les opérateurs ainsi que les fonctions de chaîne, arithmétiques, logiques et de curseur, fonctionnent de la même façon dans SQL Server et SQL Database. Il existe toutefois quelques différences au niveau des éléments du langage de définition de données (DDL) et des éléments du langage de manipulation de données (DML). Ceux-ci génèrent en effet des instructions et des requêtes T-SQL qui ne sont que partiellement prises en charge (nous y reviendrons plus loin dans cet article).

Par ailleurs, Azure SQL Database étant conçu pour isoler les fonctionnalités des dépendances sur la base de données MASTER et le système d'exploitation, certaines fonctionnalités et syntaxes ne sont pas prises en charge. Ainsi, de nombreuses activités de niveau serveur sont inappropriées pour SQL Database. Les instructions et les options T-SQL ne sont pas disponibles si elles configurent des options au niveau serveur ou des composants du système d’exploitation, ou spécifient la configuration du système de fichiers. Si de telles fonctionnalités sont nécessaires, vous trouverez la plupart du temps une alternative appropriée en procédant différemment à partir de SQL Database ou d’un autre service/d’une autre fonctionnalité Azure.

Pour plus d’informations, voir [Résolution des différences de Transact-SQL durant la migration vers SQL Database](transact-sql-tsql-differences-sql-server.md).

## <a name="security"></a>Sécurité

SQL Database fournit toute une gamme de [fonctionnalités intégrées de sécurité et conformité](security-overview.md) pour que votre application réponde aux différentes exigences de conformité et de sécurité.

> [!IMPORTANT]
> Azure SQL Database a été certifié par rapport à plusieurs normes de conformité. Pour plus d’informations, consultez le [Centre de confidentialité Microsoft Azure](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942), qui inclut la liste la plus à jour des certifications de conformité de SQL Database.

## <a name="next-steps"></a>Étapes suivantes

- Pour commencer à utiliser une base de données unique, consultez la documentation [Guide de démarrage rapide sur les bases de données uniques](quickstart-content-reference-guide.md).
- Pour en savoir plus sur la migration d’une base de données SQL Server, consultez la section [Migrer une base de données Azure SQL](migrate-to-database-from-sql-server.md).
- Pour plus d’informations sur les fonctionnalités prises en charge, consultez la page [Fonctionnalités](features-comparison.md).
