---
title: Exercices de récupération d’urgence
description: Découvrez les conseils et bonnes pratiques pour l’utilisation d’Azure SQL Database pour effectuer des exercices de récupération d’urgence.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, sstein
ms.date: 12/18/2018
ms.openlocfilehash: f53a08a12c5afda8dbc3f25d9102f52b870ceea4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91321660"
---
# <a name="performing-disaster-recovery-drills"></a>Exécution d’exercices de récupération d’urgence
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Nous recommandons de valider régulièrement la préparation des applications à la récupération. La vérification du comportement de l'application et des implications en matière de pertes de données et/ou d'interruptions en cas basculement constitue une bonne pratique. Il s'agit également d'une exigence figurant dans la plupart des normes industrielles dans le cadre d'une certification de la continuité des activités.

L'exécution d'un exercice de récupération d'urgence comprend :

* la simulation d'une défaillance des couches de données
* la récupération
* la validation de l'intégrité des applications après la récupération

Le flux de travail à exécuter peut varier en fonction de la [conception de votre application pour la continuité des activités](business-continuity-high-availability-disaster-recover-hadr-overview.md). Cet article décrit les bonnes pratiques pour effectuer une reprise d’activité après sinistre dans le contexte d’une base de données Azure SQL.

## <a name="geo-restore"></a>La géorestauration

Afin d’éviter la perte potentielle de données lors d’un exercice de récupération d’urgence, effectuez l’exercice dans un environnement de test en créant une copie de l’environnement de production et en l’utilisant pour vérifier le flux de travail de basculement de l’application.

### <a name="outage-simulation"></a>Simulation d'une défaillance

Pour simuler la défaillance, vous pouvez renommer la base de données source. Ce changement de nom provoque des échecs de connexion d’application.

### <a name="recovery"></a>Récupération

* Effectuez la géorestauration de la base de données sur un autre serveur, comme décrit [ici](disaster-recovery-guidance.md).
* Modifiez la configuration de l’application pour établir une connexion aux bases de données récupérées, puis suivez le guide [Configurer une base de données après récupération](disaster-recovery-guidance.md) pour achever la récupération.

### <a name="validation"></a>Validation

Terminez l’exercice en vérifiant l’intégrité de l’application après la récupération (notamment les chaînes de connexion, les connexions, le test des fonctionnalités de base ou d’autres validations faisant partie des procédures d’approbation d’application standard).

## <a name="failover-groups"></a>Groupes de basculement

Pour une base de données protégée à l’aide de groupes de basculement, l’exercice implique un basculement planifié sur le serveur secondaire. Le basculement planifié garantit que les bases de données primaire et secondaire dans le groupe de basculement restent synchronisées quand les rôles sont permutés. À la différence du basculement non planifié, cette opération n’entraîne pas de perte de données. L’exercice peut donc être exécuté dans l’environnement de production.

### <a name="outage-simulation"></a>Simulation d'une défaillance

Pour simuler la défaillance, vous pouvez désactiver l’application web ou une machine virtuelle connectée à la base de données. Cette simulation de défaillance se traduit par des échecs de connectivité pour les clients web.

### <a name="recovery"></a>Récupération

* Vérifiez que la configuration de l’application dans la région de récupération d’urgence pointe vers l’ancienne base de données secondaire qui devient la nouvelle base de données primaire entièrement accessible.
* Lancez le [basculement planifié](scripts/setup-geodr-and-failover-database-powershell.md) du groupe de basculement sur le serveur secondaire.
* Suivez le guide [Configure a database after recovery](disaster-recovery-guidance.md) pour effectuer la restauration.

### <a name="validation"></a>Validation

Terminez l’exercice en vérifiant l’intégrité de l’application après la récupération (notamment la connectivité, le test des fonctionnalités de base ou d’autres validations nécessaire pour les autorisations de l’exercice).

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les scénarios de continuité d’activité, consultez [Scénarios de continuité](business-continuity-high-availability-disaster-recover-hadr-overview.md).
* Pour en savoir plus sur les sauvegardes automatisées Azure SQL Database, consultez [Sauvegardes automatisées SQL Database](automated-backups-overview.md)
* Pour en savoir plus sur l’utilisation des sauvegardes automatisées pour la récupération, consultez [Restaurer une base de données à partir des sauvegardes initiées par le service](recovery-using-backups.md).
* Pour découvrir des options de récupération plus rapides, consultez [Géoréplication active](active-geo-replication-overview.md) et [Groupes de basculement automatique](auto-failover-group-overview.md).
