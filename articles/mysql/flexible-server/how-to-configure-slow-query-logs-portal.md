---
title: Configurer les journaux des requêtes lentes – Portail Azure – Azure Database pour MySQL – Serveur flexible
description: Cet article décrit comment configurer et consulter les journaux des requêtes lentes dans un serveur flexible Azure Database pour MySQL à partir du portail Azure.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 9/21/2020
ms.openlocfilehash: a61c8e3451d661dae2e5ad56a0d4a947252ec873
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94540061"
---
# <a name="configure-and-access-slow-query-logs-for-azure-database-for-mysql---flexible-server-using-the-azure-portal"></a>Configurer et consulter des journaux des requêtes lentes pour Azure Database pour MySQL – Serveur flexible à l’aide du portail Azure

> [!IMPORTANT]
> Azure Database pour MySQL – Serveur flexible est actuellement en préversion publique.

Vous pouvez configurer, répertorier et télécharger les [journaux des requêtes lentes](concepts-slow-query-logs.md) de serveur flexible Azure Database pour MySQL à partir du portail Azure.

## <a name="prerequisites"></a>Prérequis
Les étapes décrites dans cet article nécessitent un [serveur flexible](quickstart-create-server-portal.md).

## <a name="configure-logging"></a>Configuration de la journalisation
Configurer l’accès au journal des requêtes lentes de MySQL. 

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Sélectionnez votre serveur flexible.

1. Dans la section **Paramètres** dans l’encadré, sélectionnez **Paramètres du serveur**.
   :::image type="content" source="./media/how-to-configure-slow-query-logs-portal/server-parameters.png" alt-text="Page Paramètres du serveur.":::

1. Mettez à jour le paramètre **slow_query_log** en le définissant sur **ACTIVÉ**.
   :::image type="content" source="./media/how-to-configure-slow-query-logs-portal/slow-query-log-enable.png" alt-text="Activation des journaux des requêtes lentes.":::

1. Changez tous les autres paramètres nécessaires (par exemple, `long_query_time`, `log_slow_admin_statements`). Pour plus de paramètres, consultez la documentation sur les [journaux des requêtes lentes](./concepts-slow-query-logs.md#configure-slow-query-logging).  
   :::image type="content" source="./media/how-to-configure-slow-query-logs-portal/long-query-time.png" alt-text="Mise à jour des paramètres relatifs au journal des requêtes lentes.":::

1. Sélectionnez **Enregistrer**. 
   :::image type="content" source="./media/how-to-configure-slow-query-logs-portal/save-parameters.png" alt-text="Enregistrement des paramètres du journal des requêtes lentes.":::

À partir de la page **Paramètres du serveur**, vous pouvez revenir à la liste des journaux en fermant la page.

## <a name="set-up-diagnostics"></a>Configurer les diagnostics

Les journaux de requêtes lentes sont intégrés avec les paramètres de diagnostic Azure Monitor pour vous permettre de canaliser vos journaux vers les journaux Azure Monitor, Event Hubs ou Stockage Azure.

1. En dessous de la section **Supervision** dans la barre latérale, sélectionnez **Paramètres de diagnostic** > **Ajouter des paramètres de diagnostic**.

   :::image type="content" source="./media/how-to-configure-slow-query-logs-portal/add-diagnostic-setting.png" alt-text="Capture d’écran des options Paramètres de diagnostic":::

1. Entrez un nom de paramètre de diagnostic.

1. Spécifiez les destinations auxquelles envoyer les journaux des requêtes lentes (compte de stockage, hub d’événements ou espace de travail Log Analytics).

1. Sélectionnez **MySqlSlowLogs** comme type de journal.
    :::image type="content" source="./media/how-to-configure-slow-query-logs-portal/configure-diagnostic-setting.png" alt-text="Capture d’écran des options de configuration des paramètres de diagnostic":::

1. Après avoir configuré les récepteurs de données auxquels envoyer les journaux des requêtes lentes, sélectionnez **Enregistrer**.
    :::image type="content" source="./media/how-to-configure-slow-query-logs-portal/save-diagnostic-setting.png" alt-text="Capture d’écran des options de configuration des paramètres de diagnostic, avec l’option Enregistrer mise en évidence":::

1. Accédez aux journaux des requêtes lentes en les explorant dans les récepteurs de données que vous avez configurés. L’affichage des journaux peut prendre jusqu’à 10 minutes.

Si vous avez canalisé vos journaux vers les journaux Azure Monitor (Log Analytics), consultez les [exemples de requêtes](concepts-slow-query-logs.md#analyze-logs-in-azure-monitor-logs) que vous pouvez utiliser à des fins d’analyse. 

## <a name="next-steps"></a>Étapes suivantes
<!-- - See [Access slow query Logs in CLI](howto-configure-server-logs-in-cli.md) to learn how to download slow query logs programmatically.-->
- En savoir plus sur les [journaux des requêtes lentes](concepts-slow-query-logs.md)
- Pour plus d’informations sur les définitions de paramètres et la journalisation MySQL, consultez la documentation MySQL sur les [journaux](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html).
- Découvrir les [journaux d’audit](concepts-audit-logs.md)
