---
title: Accéder aux journaux d’audit - Portail Azure - Azure Database for MariaDB
description: Cet article décrit comment configurer et consulter les journaux d’audit dans Azure Database for MariaDB à partir du Portail Azure.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: how-to
ms.date: 6/24/2020
ms.openlocfilehash: f1f3cda4ed9cdac6f6e03e88601b0a35d5c53faf
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021161"
---
# <a name="configure-and-access-audit-logs-in-the-azure-portal"></a>Configurer et consulter les journaux d’audit dans le portail Azure

Vous pouvez configurer les [journaux d’audit Azure Database for MariaDB](concepts-audit-logs.md) et les paramètres de diagnostic à partir du Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour parcourir ce guide pratique, vous avez besoin des éléments suivants :

- [Serveur Azure Database for MariaDB](quickstart-create-mariadb-server-database-using-azure-portal.md)

## <a name="configure-audit-logging"></a>Configurer l’enregistrement d’audit

>[!IMPORTANT]
> Il est recommandé de ne consigner que les types d’événements et les utilisateurs requis à des fins d’audit pour garantir que les performances de votre serveur ne sont pas fortement affectées.

Activez et configurez l’enregistrement d’audit.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Sélectionnez votre serveur Azure Database for MariaDB.

1. Dans la section **Paramètres** dans l’encadré, sélectionnez **Paramètres du serveur**.
    ![Paramètres du serveur](./media/howto-configure-audit-logs-portal/server-parameters.png)

1. Mettez à jour le paramètre **audit_log_enabled** sur ON.
    ![Activer les journaux d’audit](./media/howto-configure-audit-logs-portal/audit-log-enabled.png)

1. Sélectionnez les [types d’événements](concepts-audit-logs.md#configure-audit-logging) à enregistrer en mettant à jour le paramètre **audit_log_events**.
    ![Événements du journal d’audit](./media/howto-configure-audit-logs-portal/audit-log-events.png)

1. Ajoutez des utilisateurs MariaDB à exclure de la journalisation en mettant à jour le paramètre **audit_log_exclude_users**. Spécifiez les utilisateurs en fournissant leur nom d’utilisateur MariaDB.
    ![Utilisateurs à exclure du journal d’audit](./media/howto-configure-audit-logs-portal/audit-log-exclude-users.png)

1. Une fois que vous avez modifié les paramètres, vous pouvez cliquer sur **Enregistrer**. Vous pouvez également **Abandonner** vos modifications.
    ![Save](./media/howto-configure-audit-logs-portal/save-parameters.png)

## <a name="set-up-diagnostic-logs"></a>Configurer les journaux de diagnostic

1. Dans la section **Supervision** dans la barre latérale, sélectionnez **Paramètres de diagnostic**.

1. Cliquez sur « + Ajouter un paramètre de diagnostic » ![Ajouter un paramètre de diagnostic](./media/howto-configure-audit-logs-portal/add-diagnostic-setting.png)

1. Entrez un nom de paramètre de diagnostic.

1. Spécifiez les récepteurs de données auxquels envoyer les journaux d’audit (compte de stockage, hub d’événements et/ou espace de travail Log Analytics).

1. Sélectionnez « MySqlAuditLogs » comme type de journal.
![Configurer un paramètre de diagnostic](./media/howto-configure-audit-logs-portal/configure-diagnostic-setting.png)

1. Une fois que vous avez configuré les récepteurs de données auxquels envoyer les journaux d’audit, vous pouvez cliquer sur **Enregistrer**.
![Enregistrer un paramètre de diagnostic](./media/howto-configure-audit-logs-portal/save-diagnostic-setting.png)

1. Accédez aux journaux d’audit en les explorant dans les récepteurs de données que vous avez configurés. L’affichage des journaux peut prendre jusqu’à 10 minutes.

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur les [journaux d’audit](concepts-audit-logs.md) dans Azure Database for MariaDB
- Découvrir comment configurer les journaux d’audit dans [Azure CLI](howto-configure-audit-logs-cli.md)