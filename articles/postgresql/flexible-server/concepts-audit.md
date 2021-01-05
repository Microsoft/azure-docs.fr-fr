---
title: Journalisation d’audit - Azure Database pour PostgreSQL - Serveur flexible
description: Concepts relatifs à la journalisation d’audit pgAudit dans Azure Database pour PostgreSQL - Serveur flexible.
author: niklarin
ms.author: nlarin
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: cf749f3aef10a0c67814722577f79906f447ffdb
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92532779"
---
# <a name="audit-logging-in-azure-database-for-postgresql---flexible-server"></a>Journalisation d’audit dans Azure Database pour PostgreSQL - Serveur flexible

La journalisation d’audit des activités de base de données dans Azure Database pour PostgreSQL - Serveur flexible est disponible via l’extension d’audit PostgreSQL nommée [pgAudit](https://www.pgaudit.org/). pgAudit permet une journalisation d’audit détaillée des sessions et/ou des objets.

> [!IMPORTANT]
> Azure Database pour PostgreSQL - Serveur flexible est en préversion

Si vous souhaitez des journaux de niveau ressource Azure pour des opérations telles que la mise à l’échelle du stockage et du calcul, consultez le [journal d’activité Azure](../../azure-monitor/platform/platform-logs-overview.md).

## <a name="usage-considerations"></a>Considérations sur l’utilisation
Par défaut, les instructions de journal pgAudit sont émises en même temps que vos instructions de journal habituelles à l’aide de la fonctionnalité de journalisation standard de Postgres. Dans Azure Database pour PostgreSQL - Serveur flexible, vous pouvez également configurer tous les journaux à envoyer au magasin de journaux Azure Monitor pour analyse ultérieure dans Log Analytics. Si vous activez la journalisation des ressources Azure Monitor, vos journaux sont automatiquement envoyés (au format JSON) au services Stockage Azure et Event Hubs, et/ou aux journaux Azure Monitor, conformément à votre choix.

Pour savoir comment configurer la journalisation dans les services Stockage Azure et Event Hubs ou les journaux Azure Monitor, consultez la section concernant les journaux de ressources dans l’[article sur les journaux de serveur](concepts-logging.md).

## <a name="enabling-pgaudit"></a>Activation de pgAudit

Pour activer pgAudit, vous devez vous connecter à votre serveur à l’aide d’un client (comme psql) et activer l’extension pgAudit en exécutant la commande suivante :
```SQL
CREATE EXTENSION pgaudit;
```

## <a name="pgaudit-settings"></a>Paramètres pgAudit

pgAudit vous permet de configurer la journalisation d’audit des sessions et des objets. La [journalisation d’audit des sessions](https://github.com/pgaudit/pgaudit/blob/master/README.md#session-audit-logging) émet des journaux détaillés où se trouvent les instructions exécutées. La [journalisation d’audit des objets](https://github.com/pgaudit/pgaudit/blob/master/README.md#object-audit-logging) est limitée à certaines relations. Vous pouvez choisir de configurer un ou plusieurs types de journalisation. 

> [!NOTE]
> Les paramètres pgAudit sont spécifiés globalement et ne peuvent pas être spécifiés au niveau d’une base de données ou d’un rôle.

Une fois que vous avez [activé pgAudit](#enabling-pgaudit), vous pouvez configurer ses paramètres pour démarrer la journalisation. La [documentation pgAudit](https://github.com/pgaudit/pgaudit/blob/master/README.md#settings) donne la définition de chaque paramètre. Testez d’abord les paramètres pour vérifier que vous obtenez le comportement attendu.

> [!NOTE]
> L’affectation de la valeur ON à `pgaudit.log_client` permet de rediriger les journaux vers un processus client (comme psql) au lieu de les écrire dans un fichier. Ce paramètre doit généralement rester désactivé. <br> <br>
> `pgaudit.log_level` est activé uniquement lorsque `pgaudit.log_client` est activé.

> [!NOTE]
> Dans Azure Database pour PostgreSQL - Serveur flexible, `pgaudit.log` ne peut pas être défini à l’aide d’un raccourci de signe `-` (moins) contrairement à ce qui est dit dans la documentation pgAudit. Toutes les classes d’instruction requises (READ, WRITE, etc.) doivent être spécifiées individuellement.

## <a name="audit-log-format"></a>Format du journal d’audit
Chaque entrée d’audit est indiquée par `AUDIT:` en début de ligne. Le format du reste de l’entrée est abordé en détail dans la [documentation pgAudit](https://github.com/pgaudit/pgaudit/blob/master/README.md#format).

## <a name="getting-started"></a>Prise en main
Pour commencer, définissez `pgaudit.log` sur `WRITE`, puis ouvrez vos journaux de serveur pour consulter la sortie. 

## <a name="viewing-audit-logs"></a>Affichage des journaux d’audit
La façon dont vous accédez aux journaux dépend du point de terminaison que vous choisissez. Pour le stockage Azure, consultez l’article [Compte de stockage des journaux](../../azure-monitor/platform/resource-logs.md#send-to-azure-storage). Pour Event Hubs, consultez l’article [Diffusion des journaux Azure](../../azure-monitor/platform/resource-logs.md#send-to-azure-event-hubs).

Pour les journaux Azure Monitor, les journaux sont envoyés à l’espace de travail que vous avez sélectionné. Les journaux Postgres utilisent le mode de collecte **AzureDiagnostics** , pour qu’ils puissent être interrogés à partir de la table AzureDiagnostics. Les champs de la table sont décrits ci-dessous. En savoir plus sur l’interrogation et la génération d’alertes dans la vue d’ensemble [Interroger les journaux Azure Monitor](../../azure-monitor/log-query/log-query-overview.md).

Vous pouvez utiliser cette requête pour commencer. Vous pouvez configurer des alertes basées sur les requêtes.

Rechercher toutes les entrées pgAudit dans les journaux Postgres pour un serveur particulier au cours du dernier jour
```kusto
AzureDiagnostics
| where LogicalServerName_s == "myservername"
| where TimeGenerated > ago(1d) 
| where Message contains "AUDIT:"
```

## <a name="next-steps"></a>Étapes suivantes
- [En savoir plus sur la journalisation dans Azure Database pour PostgreSQL - Serveur flexible](concepts-logging.md)
- [Découvrez comment configurer la journalisation dans Azure Database pour PostgreSQL - Serveur flexible et comment accéder aux journaux](howto-configure-and-access-logs.md)