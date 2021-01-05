---
title: Journaux - Hyperscale (Citus) - Azure Database pour PostgreSQL
description: Accéder aux journaux de base de données Azure Database pour PostgreSQL - Hyperscale (Citus)
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 7/13/2020
ms.openlocfilehash: f8840d5115cb552ed203705d37f8c692b3418947
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492333"
---
# <a name="logs-in-azure-database-for-postgresql---hyperscale-citus"></a>Journaux dans Azure Database pour PostgreSQL - Hyperscale (Citus)

Des journaux PostgreSQL sont disponibles sur chaque nœud d’un groupe de serveurs Hyperscale (Citus). Vous pouvez envoyer des journaux à un serveur de stockage ou à un service d’analytique. Les journaux d’activité peuvent servir à identifier, résoudre et réparer les erreurs de configuration et les problèmes de performances.

## <a name="accessing-logs"></a>Accès aux journaux d’activité

Pour accéder aux journaux PostgreSQL d’un coordinateur Hyperscale (Citus) ou d’un nœud Worker, ouvrez le nœud dans le portail Azure :

:::image type="content" source="media/howto-hyperscale-logging/choose-node.png" alt-text="Liste des nœuds":::

Pour le nœud sélectionné, ouvrez **Paramètres de diagnostic**, puis cliquez sur **+Ajouter un paramètre de diagnostic**.

:::image type="content" source="media/howto-hyperscale-logging/diagnostic-settings.png" alt-text="Ajouter des paramètres de diagnostic - Bouton":::

Choisissez un nom pour les nouveaux paramètres de diagnostic, puis cochez la case **PostgreSQLLogs**.  Choisissez la ou les destinations qui doivent recevoir les journaux.

:::image type="content" source="media/howto-hyperscale-logging/diagnostic-create-setting.png" alt-text="Choisir des journaux PostgreSQL":::

## <a name="next-steps"></a>Étapes suivantes

- [Bien démarrer avec les requêtes Log Analytics](../azure-monitor/log-query/log-analytics-tutorial.md)
- En savoir plus sur les [hubs d’événements Azure](../event-hubs/event-hubs-about.md)