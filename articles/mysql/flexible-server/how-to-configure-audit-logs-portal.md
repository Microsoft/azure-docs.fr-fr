---
title: Configurer les journaux d’audit - Portail Azure - Azure Database pour MySQL - Serveur flexible
description: Cet article décrit comment configurer et consulter les journaux d’audit sur un serveur flexible Azure Database pour MySQL à partir du portail Azure.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 9/29/2020
ms.openlocfilehash: ebb980aa257fc09c3d6a407febbf60f2d1a26a4e
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94536470"
---
# <a name="configure-and-access-audit-logs-for-azure-database-for-mysql---flexible-server-using-the-azure-portal"></a>Configurer et consulter les journaux d’audit sur un serveur flexible Azure Database pour MySQL à l’aide du portail Azure

> [!IMPORTANT]
> Azure Database pour MySQL - Serveur flexible est actuellement en préversion publique.

Vous pouvez configurer les [journaux d’audit](concepts-audit-logs.md) et les paramètres de diagnostic d’un serveur flexible Azure Database pour MySQL à partir du portail Azure.

## <a name="prerequisites"></a>Prérequis
Les étapes décrites dans cet article nécessitent un [serveur flexible](quickstart-create-server-portal.md).

## <a name="configure-audit-logging"></a>Configurer l’enregistrement d’audit

>[!IMPORTANT]
> Il est recommandé de ne consigner que les types d’événements et les utilisateurs requis à des fins d’audit pour garantir que les performances de votre serveur ne sont pas fortement affectées.

Activez et configurez l’enregistrement d’audit.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Sélectionnez votre serveur flexible.

1. Dans la section **Paramètres** dans l’encadré, sélectionnez **Paramètres du serveur**.
    :::image type="content" source="./media/how-to-configure-audit-logs-portal/server-parameters.png" alt-text="Paramètres du serveur":::

1. Mettez à jour le paramètre **audit_log_enabled** sur ON.
    :::image type="content" source="./media/how-to-configure-audit-logs-portal/audit-log-enabled.png" alt-text="Activer les journaux d’audit":::

1. Sélectionnez les [types d’événements](concepts-audit-logs.md#configure-audit-logging) à enregistrer en mettant à jour le paramètre **audit_log_events**.
    :::image type="content" source="./media/how-to-configure-audit-logs-portal/audit-log-events.png" alt-text="Événements du journal d’audit":::

1. Ajoutez des utilisateurs MySQL à inclure dans la journalisation ou à exclure de celle-ci en mettant à jour les paramètres **audit_log_exclude_users** et **audit_log_include_users**. Spécifiez les utilisateurs en fournissant leur nom d’utilisateur MySQL.
    :::image type="content" source="./media/how-to-configure-audit-logs-portal/audit-log-exclude-users.png" alt-text="Utilisateurs à exclure du journal d’audit":::

1. Une fois que vous avez modifié les paramètres, vous pouvez cliquer sur **Enregistrer**. Vous pouvez également **Abandonner** vos modifications.
    :::image type="content" source="./media/how-to-configure-audit-logs-portal/save-parameters.png" alt-text="Save":::

## <a name="set-up-diagnostics"></a>Configurer les diagnostics

Les journaux d’audit sont intégrés aux paramètres de diagnostic Azure Monitor pour vous permettre de canaliser vos journaux vers les journaux Azure Monitor, Event Hubs ou Stockage Azure.

1. Dans la section **Supervision** dans la barre latérale, sélectionnez **Paramètres de diagnostic**.

1. Cliquez sur « + Ajouter un paramètre de diagnostic » :::image type="content" source="./media/how-to-configure-audit-logs-portal/add-diagnostic-setting.png" alt-text="Ajouter un paramètre de diagnostic":::

1. Entrez un nom de paramètre de diagnostic.

1. Spécifiez les destinations auxquelles envoyer les journaux d’audit (compte de stockage, hub d’événements et/ou espace de travail Log Analytics).

1. Sélectionnez **MySqlAuditLogs** comme type de journal.
    :::image type="content" source="./media/how-to-configure-audit-logs-portal/configure-diagnostic-setting.png" alt-text="Configurer un paramètre de diagnostic":::

1. Une fois que vous avez configuré les récepteurs de données auxquels envoyer les journaux d’audit, vous pouvez cliquer sur **Enregistrer**.
    :::image type="content" source="./media/how-to-configure-audit-logs-portal/save-diagnostic-setting.png" alt-text="Enregistrer un paramètre de diagnostic":::

1. Accédez aux journaux d’audit en les explorant dans les récepteurs de données que vous avez configurés. L’affichage des journaux peut prendre jusqu’à 10 minutes.

Si vous avez canalisé vos journaux d’audit vers les journaux Azure Monitor (Log Analytics), reportez-vous aux [exemples de requêtes](concepts-audit-logs.md#analyze-logs-in-azure-monitor-logs) que vous pouvez utiliser à des fins d’analyse.  

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [journaux d'audit](concepts-audit-logs.md)
- En savoir plus sur les [journaux des requêtes lentes](concepts-slow-query-logs.md)
<!-- - Learn how to configure audit logs in the [Azure CLI](howto-configure-audit-logs-cli.md)-->