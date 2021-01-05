---
title: Journalisation d’audit - Azure Database pour PostgreSQL - Serveur unique
description: Concepts relatifs à la journalisation d’audit pgAudit dans Azure Database pour PostgreSQL - Serveur unique
author: niklarin
ms.author: nlarin
ms.service: postgresql
ms.topic: conceptual
ms.date: 01/28/2020
ms.openlocfilehash: 615297a4bf47d80c9313f011b90d343b7ae680e3
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92488042"
---
# <a name="audit-logging-in-azure-database-for-postgresql---single-server"></a>Journalisation d’audit dans Azure Database pour PostgreSQL - Serveur unique

La journalisation d’audit des activités de base de données dans Azure Database pour PostgreSQL - Serveur unique est disponible via l’extension d’audit PostgreSQL nommée [pgAudit](https://www.pgaudit.org/). pgAudit permet une journalisation d’audit détaillée des sessions et/ou des objets.

> [!NOTE]
> pgAudit est disponible en préversion sur Azure Database pour PostgreSQL.
> L’extension peut être activée uniquement sur les serveurs à usage général et les serveurs à mémoire optimisée.

Si vous souhaitez des journaux de niveau ressource Azure pour des opérations telles que la mise à l’échelle du stockage et du calcul, consultez le [journal d’activité Azure](../azure-monitor/platform/platform-logs-overview.md).

## <a name="usage-considerations"></a>Considérations sur l’utilisation
Par défaut, les instructions de journal pgAudit sont émises en même temps que vos instructions de journal habituelles à l’aide de la fonctionnalité de journalisation standard de Postgres. Dans Azure Database pour PostgreSQL, ces fichiers .log peuvent être téléchargés à partir du portail Azure ou de l’interface CLI. La taille de stockage maximale d’une collection de fichiers est de 1 Go, et chaque fichier est disponible pendant une période maximale de sept jours (la valeur par défaut est de trois jours). Ce service est une option de stockage à court terme.

Vous pouvez également configurer tous les journaux à envoyer au magasin de journaux Azure Monitor pour analyse ultérieure dans Log Analytics. Si vous activez la journalisation des ressources Azure Monitor, vos journaux sont automatiquement envoyés (au format JSON) au services Stockage Azure et Event Hubs, et/ou aux journaux Azure Monitor, conformément à votre choix.

L’activation de pgAudit génère un grand volume de journalisation sur un serveur, ce qui a un impact sur les performances et le stockage des journaux. Nous vous recommandons d’utiliser les journaux Azure Monitor, qui offrent des options de stockage à plus long terme, ainsi que des fonctionnalités d’analyse et de génération d’alertes. Nous vous recommandons de désactiver la journalisation standard afin de réduire l’impact d’une journalisation supplémentaire sur les performances :

   1. Définissez le paramètre `logging_collector` sur OFF. 
   2. Redémarrez le serveur pour appliquer les modifications.

Pour savoir comment configurer la journalisation dans les services Stockage Azure et Event Hubs ou les journaux Azure Monitor, consultez la section concernant les journaux de ressources dans l’[article sur les journaux de serveur](concepts-server-logs.md).

## <a name="installing-pgaudit"></a>Installation de pgAudit

Pour installer pgAudit, vous devez l’inclure dans les bibliothèques de préchargement partagées du serveur. Une modification apportée au paramètre `shared_preload_libraries` de Postgres nécessite un redémarrage du serveur pour prendre effet. Vous pouvez modifier les paramètres à l’aide du [portail Azure](howto-configure-server-parameters-using-portal.md), d’[Azure CLI](howto-configure-server-parameters-using-cli.md) ou de l’[API REST](/rest/api/postgresql/configurations/createorupdate).

À l’aide du [Portail Azure](https://portal.azure.com) :

   1. Sélectionnez votre serveur Azure Database pour PostgreSQL.
   2. Dans la barre latérale, sélectionnez **Paramètres du serveur** .
   3. Recherchez le paramètre `shared_preload_libraries`.
   4. Sélectionnez **pgaudit** .
   5. Redémarrez le serveur pour appliquer les modifications.

   6. Connectez-vous à votre serveur à l’aide d’un client (par exemple, psql), puis activez l’extension pgAudit.
      ```SQL
      CREATE EXTENSION pgaudit;
      ```

> [!TIP]
> Si une erreur s’affiche, vérifiez que vous avez redémarré le serveur après l’enregistrement de `shared_preload_libraries`.

## <a name="pgaudit-settings"></a>Paramètres pgAudit

pgAudit vous permet de configurer la journalisation d’audit des sessions et des objets. La [journalisation d’audit des sessions](https://github.com/pgaudit/pgaudit/blob/master/README.md#session-audit-logging) émet des journaux détaillés où se trouvent les instructions exécutées. La [journalisation d’audit des objets](https://github.com/pgaudit/pgaudit/blob/master/README.md#object-audit-logging) est limitée à certaines relations. Vous pouvez choisir de configurer un ou plusieurs types de journalisation. 

> [!NOTE]
> Les paramètres pgAudit sont spécifiés globalement et ne peuvent pas être spécifiés au niveau d’une base de données ou d’un rôle.

Une fois que vous avez [installé pgAudit](#installing-pgaudit), vous pouvez configurer ses paramètres pour démarrer la journalisation. La [documentation pgAudit](https://github.com/pgaudit/pgaudit/blob/master/README.md#settings) donne la définition de chaque paramètre. Testez d’abord les paramètres pour vérifier que vous obtenez le comportement attendu.

> [!NOTE]
> L’affectation de la valeur ON à `pgaudit.log_client` permet de rediriger les journaux vers un processus client (comme psql) au lieu de les écrire dans un fichier. Ce paramètre doit généralement rester désactivé. <br> <br>
> `pgaudit.log_level` est activé uniquement lorsque `pgaudit.log_client` est activé.

> [!NOTE]
> Dans Azure Database pour PostgreSQL, `pgaudit.log` ne peut pas être défini à l’aide d’un raccourci de signe `-` (moins) contrairement à ce qui est dit dans la documentation pgAudit. Toutes les classes d’instruction requises (READ, WRITE, etc.) doivent être spécifiées individuellement.

### <a name="audit-log-format"></a>Format du journal d’audit
Chaque entrée d’audit est indiquée par `AUDIT:` en début de ligne. Le format du reste de l’entrée est abordé en détail dans la [documentation pgAudit](https://github.com/pgaudit/pgaudit/blob/master/README.md#format).

Si vous avez besoin d’autres champs pour votre audit, utilisez le paramètre `log_line_prefix` Postgres. `log_line_prefix` est une chaîne qui est affichée au début de chaque ligne d’un journal Postgres. Par exemple, le paramètre `log_line_prefix` suivant fournit l’horodatage, le nom d’utilisateur, le nom de la base de données et l’ID de processus :

```
t=%m u=%u db=%d pid=[%p]:
```

Pour plus d’informations sur `log_line_prefix`, consultez la [documentation PostgreSQL](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-LINE-PREFIX).

### <a name="getting-started"></a>Prise en main
Pour commencer, définissez `pgaudit.log` sur `WRITE`, puis ouvrez vos journaux pour consulter la sortie. 

## <a name="viewing-audit-logs"></a>Affichage des journaux d’audit
Si vous utilisez des fichiers .log, vos journaux d’audit seront inclus dans le même fichier que vos journaux d’erreurs PostgreSQL. Vous pouvez télécharger des fichiers journaux à partir du [Portail](howto-configure-server-logs-in-portal.md) Azure ou de l’[interface CLI](howto-configure-server-logs-using-cli.md). 

Si vous utilisez la journalisation des ressources Azure, la façon dont vous accédez aux journaux dépend du point de terminaison que vous choisissez. Pour le stockage Azure, consultez l’article [Compte de stockage des journaux](../azure-monitor/platform/resource-logs.md#send-to-azure-storage). Pour Event Hubs, consultez l’article [Diffusion des journaux Azure](../azure-monitor/platform/resource-logs.md#send-to-azure-event-hubs).

Pour les journaux Azure Monitor, les journaux sont envoyés à l’espace de travail que vous avez sélectionné. Les journaux Postgres utilisent le mode de collecte **AzureDiagnostics** , pour qu’ils puissent être interrogés à partir de la table AzureDiagnostics. Les champs de la table sont décrits ci-dessous. En savoir plus sur l’interrogation et la génération d’alertes dans la vue d’ensemble [Interroger les journaux Azure Monitor](../azure-monitor/log-query/log-query-overview.md).

Vous pouvez utiliser cette requête pour commencer. Vous pouvez configurer des alertes basées sur les requêtes.

Rechercher tous les journaux Postgres pour un serveur particulier au cours du dernier jour
```
AzureDiagnostics
| where LogicalServerName_s == "myservername"
| where TimeGenerated > ago(1d) 
| where Message contains "AUDIT:"
```

## <a name="next-steps"></a>Étapes suivantes
- [En savoir plus sur la journalisation dans Azure Database pour PostgreSQL](concepts-server-logs.md)
- Découvrez comment définir les paramètres à l’aide du [portail Azure](howto-configure-server-parameters-using-portal.md), d’[Azure CLI](howto-configure-server-parameters-using-cli.md) ou de l’[API REST](/rest/api/postgresql/configurations/createorupdate).