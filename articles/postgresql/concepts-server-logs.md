---
title: Journaux - Azure Database pour PostgreSQL - Serveur unique
description: Décrit la configuration de la journalisation, le stockage et l’analyse dans Azure Database pour PostgreSQL - Serveur unique
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: conceptual
ms.date: 06/25/2020
ms.openlocfilehash: 2ae3c538c78be8af0fa4569592ac60547e7f5912
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92481293"
---
# <a name="logs-in-azure-database-for-postgresql---single-server"></a>Journaux dans Azure Database pour PostgreSQL - Serveur unique

Azure Database pour PostgreSQL vous permet de configurer et d’accéder aux journaux standard de Postgres. Les journaux d’activité peuvent servir à identifier, résoudre et réparer les erreurs de configuration et les problèmes de performances. Les informations de journalisation que vous pouvez configurer et auxquelles vous pouvez accéder incluent les erreurs, les informations de requête, les enregistrements de nettoyage automatique, les connexions et les points de contrôle. (L’accès aux journaux d’activité des transactions n’est pas disponible).

L’enregistrement d'audit est mis à disposition via une extension Postgres, pgaudit. Pour plus d’informations, consultez l’article [Concepts d’audit](concepts-audit.md).


## <a name="configure-logging"></a>Configuration de la journalisation 
Vous pouvez configurer la journalisation standard Postgres sur votre serveur avec les paramètres de journalisation. Sur chaque serveur Azure Database pour PostgreSQL, `log_checkpoints` et `log_connections` sont activés par défaut. Il existe d’autres paramètres que vous pouvez définir en fonction de vos besoins de journalisation : 

:::image type="content" source="./media/concepts-server-logs/log-parameters.png" alt-text="Azure Database pour PostgreSQL - Paramètres de journalisation":::

Pour en savoir plus sur les paramètres de journal Postgres, consultez les sections [Quand journaliser](https://www.postgresql.org/docs/current/runtime-config-logging.html#RUNTIME-CONFIG-LOGGING-WHEN) et [Que journaliser](https://www.postgresql.org/docs/current/runtime-config-logging.html#RUNTIME-CONFIG-LOGGING-WHAT) de la documentation Postgres. La plupart des paramètres de journalisation Postgres, mais pas tous, peuvent être configurés dans Azure Database pour PostgreSQL.

Pour savoir comment configurer les paramètres Azure Database pour PostgreSQL, consultez la [documentation du portail](howto-configure-server-parameters-using-portal.md) ou la [documentation de la CLI](howto-configure-server-parameters-using-cli.md). 

> [!NOTE]
> La configuration d’un volume élevé de journaux, par exemple la journalisation d’instructions, peut créer une surcharge significative sur les performances. 

## <a name="access-log-files"></a>Accéder aux fichiers .log
Le format de journal par défaut dans Azure Database pour PostgreSQL est .log. Voici un exemple de ligne de ce journal :

```
2019-10-14 17:00:03 UTC-5d773cc3.3c-LOG: connection received: host=101.0.0.6 port=34331 pid=16216
```

Azure Database pour PostgreSQL fournit un emplacement de stockage à court terme pour les fichiers .log. Un nouveau fichier commence 1 fois par heure ou tous les 100 Mo, selon ce qui se produit en premier. Les journaux sont ajoutés au fichier en cours lorsqu’ils sont émis à partir de Postgres.  

Vous pouvez définir la période de conservation pour ce stockage à court terme du journal à l’aide du paramètre `log_retention_period`. La valeur par défaut est de 3 jours et la valeur maximale est de 7 jours. L’emplacement de stockage à court terme peut contenir jusqu’à 1 Go de fichiers journaux. Après 1 Go, les fichiers les plus anciens, quelle que soit la période de conservation, seront supprimés pour faire de la place aux nouveaux journaux. 

Pour une conservation à long terme des journaux et l’analyse des journaux, vous pouvez télécharger les fichiers .log et les utiliser dans un service tiers. Vous pouvez télécharger les fichiers à l’aide du [portail Azure](howto-configure-server-logs-in-portal.md) et [d’Azure CLI](howto-configure-server-logs-using-cli.md). Vous pouvez également configurer les paramètres de diagnostic d’Azure Monitor pour émettre automatiquement vos journaux (au format JSON) à des emplacements à plus long terme. Pour plus d’informations sur cette option, consultez la section ci-dessous. 

Vous pouvez arrêter la génération de fichiers .log en définissant le paramètre `logging_collector` sur Désactivé. Désactiver la génération du fichier .log est recommandée si vous utilisez les paramètres de diagnostic d’Azure Monitor. Cette configuration permet de réduire l’impact sur les performances de la journalisation supplémentaire.

## <a name="resource-logs"></a>Journaux d’activité de ressources

Azure Database pour PostgreSQL est intégré aux journaux des paramètres de diagnostic d’Azure Monitor. Les paramètres de diagnostic vous permettent d’envoyer vos journaux Postgres au format JSON aux journaux Azure Monitor à des fins d’analyse et d’alerte, à Event Hubs pour la diffusion en continu et au stockage Azure pour l’archivage. 

> [!IMPORTANT]
> Cette fonctionnalité de diagnostic des journaux de serveur n’est disponible que dans les [niveaux tarifaires](concepts-pricing-tiers.md) Usage général et Mémoire optimisée.


### <a name="configure-diagnostic-settings"></a>Configurer les paramètres de diagnostic

Vous pouvez activer les paramètres de diagnostic pour votre serveur Postgres à l’aide du portail Azure, de l’interface CLI, de l’API REST et de PowerShell. La catégorie de journal à sélectionner est **PostgreSQLLogs** . (Il existe d’autres journaux que vous pouvez configurer si vous utilisez le [Magasin des requêtes](concepts-query-store.md).)

Pour activer les journaux de ressources à l’aide du portail Azure :

   1. Sur le portail, accédez à *Paramètres de diagnostic* dans le menu de navigation de votre serveur Postgres.
   2. Sélectionnez *Ajouter le paramètre de diagnostic* .
   3. Donnez un nom à ce paramètre. 
   4. Sélectionnez le point de terminaison de votre choix (compte de stockage, hub d’événements, analytique des journaux). 
   5. Sélectionnez le type de journal **PostgreSQLLogs** .
   7. Enregistrez votre paramètre.

Pour activer les journaux de ressources à l’aide de PowerShell, de l’interface de ligne de commande ou de l’API REST, consultez l’article [Paramètres de diagnostic](../azure-monitor/platform/diagnostic-settings.md).

### <a name="access-resource-logs"></a>Accéder aux journaux de ressources

La façon dont vous accédez aux journaux dépend du point de terminaison que vous choisissez. Pour le stockage Azure, consultez l’article [Compte de stockage des journaux](../azure-monitor/platform/resource-logs.md#send-to-azure-storage). Pour Event Hubs, consultez l’article [Diffusion des journaux Azure](../azure-monitor/platform/resource-logs.md#send-to-azure-event-hubs).

Pour les journaux Azure Monitor, les journaux sont envoyés à l’espace de travail que vous avez sélectionné. Les journaux Postgres utilisent le mode de collecte **AzureDiagnostics** , pour qu’ils puissent être interrogés à partir de la table AzureDiagnostics. Les champs de la table sont décrits ci-dessous. En savoir plus sur l’interrogation et la génération d’alertes dans la vue d’ensemble [Interroger les journaux Azure Monitor](../azure-monitor/log-query/log-query-overview.md).

Voici des requêtes que vous pouvez essayer pour commencer. Vous pouvez configurer des alertes basées sur les requêtes.

Rechercher tous les journaux Postgres pour un serveur particulier au cours du dernier jour
```
AzureDiagnostics
| where LogicalServerName_s == "myservername"
| where Category == "PostgreSQLLogs"
| where TimeGenerated > ago(1d) 
```

Rechercher toutes les tentatives de connexion autres que localhost
```
AzureDiagnostics
| where Message contains "connection received" and Message !contains "host=127.0.0.1"
| where Category == "PostgreSQLLogs" and TimeGenerated > ago(6h)
```
La requête ci-dessus affiche les résultats au cours des six dernières heures de toutes les journalisations de serveur PostgreSQL dans cet espace de travail.

### <a name="log-format"></a>Format de journal

Le tableau suivant décrit les champs du type **PostgreSQLLogs** . En fonction du point de terminaison de sortie choisi, les champs et l’ordre dans lequel ils apparaissent peuvent varier. 

|**Champ** | **Description** |
|---|---|
| TenantId | Votre ID d’abonné |
| SourceSystem | `Azure` |
| TimeGenerated [UTC] | Horodatage du moment où le journal a été enregistré en UTC |
| Type | Type de journal. Toujours `AzureDiagnostics` |
| SubscriptionId | GUID de l’abonnement auquel appartient le serveur |
| ResourceGroup | Nom du groupe de ressources auquel le serveur appartient |
| ResourceProvider | Nom du fournisseur de ressources. Toujours `MICROSOFT.DBFORPOSTGRESQL` |
| ResourceType | `Servers` |
| ResourceId | URI de ressource |
| Ressource | Nom du serveur |
| Category | `PostgreSQLLogs` |
| NomOpération | `LogEvent` |
| errorLevel | Niveau de journalisation, par exemple : LOG, ERROR, NOTICE |
| Message | Message de journal principal | 
| Domain | Version du serveur, par exemple : postgres-10 |
| Detail | Message du journal secondaire (le cas échéant) |
| ColumnName | Nom de la colonne (le cas échéant) |
| SchemaName | Nom du schéma (le cas échéant) |
| DatatypeName | Nom du type de données (le cas échéant) |
| LogicalServerName | Nom du serveur | 
| _ResourceId | URI de ressource |
| Préfixe | Préfixe de la ligne de journal |


## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur l’accès aux journaux d’activité à partir du [portail Azure](howto-configure-server-logs-in-portal.md) ou de l’[interface de ligne de commande Azure](howto-configure-server-logs-using-cli.md).
- En savoir plus sur la [tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).
- En savoir plus sur les [journaux d'audit](concepts-audit.md)