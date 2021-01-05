---
title: Configurer les paramètres de serveur - Portail Azure - Azure Database for MariaDB
description: Cet article explique comment configurer les paramètres de serveur MariaDB dans Azure Database for MariaDB à l’aide du portail Azure.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: how-to
ms.date: 10/1/2020
ms.openlocfilehash: 7081535bb709e6731a9a15436334e8742e7bdd08
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94539454"
---
# <a name="configure-server-parameters-in-azure-database-for-mariadb-using-the-azure-portal"></a>Configurer les paramètres de serveur dans Azure Database pour MySQL

Azure Database for MariaDB prend en charge la configuration de certains paramètres de serveur. Cet article décrit comment configurer ces paramètres à l’aide du portail Azure. Les paramètres du serveur ne sont pas tous modifiables.

>[!Note]
> Les paramètres serveur peuvent être mis à jour globalement au niveau du serveur à partir d’[Azure CLI](./howto-configure-server-parameters-cli.md), de [PowerShell](./howto-configure-server-parameters-using-powershell.md) ou du [portail Azure](./howto-server-parameters.md).

## <a name="configure-server-parameters"></a>Configurer les paramètres du serveur

1. Connectez-vous au portail Azure, puis localisez votre serveur Azure Database for MariaDB.
2. Dans la section **PARAMÈTRES**, cliquez sur **Paramètres du serveur** pour ouvrir la page de paramètres du serveur Azure Database for MariaDB.
![Page Paramètres du serveur du portail Azure](./media/howto-server-parameters/azure-portal-server-parameters.png)
3. Recherchez les paramètres que vous devez ajuster. Examinez la colonne **Description** pour comprendre la fonction et les valeurs autorisées.
![Bouton déroulant Énumérer](./media/howto-server-parameters/3-toggle_parameter.png)
4. Cliquez sur **Enregistrer** pour enregistrer vos modifications.
![Enregistrer ou annuler les modifications](./media/howto-server-parameters/4-save_parameters.png)
5. Si vous avez enregistré de nouvelles valeurs pour les paramètres, vous pouvez toujours rétablir toutes les valeurs par défaut en sélectionnant **Rétablir toutes les valeurs par défaut**.
![Rétablir toutes les valeurs par défaut](./media/howto-server-parameters/5-reset_parameters.png)

## <a name="setting-parameters-not-listed"></a>Configurer les paramètres non listés

Si le paramètre de serveur que vous souhaitez mettre à jour n’est pas listé dans le portail Azure, vous pouvez éventuellement définir le paramètre au niveau de la connexion avec `init_connect`. Cela définit les paramètres de serveur pour chaque client qui se connecte. 

1. Dans la section **PARAMÈTRES**, cliquez sur **Paramètres du serveur** pour ouvrir la page de paramètres du serveur Azure Database for MariaDB.
2. Rechercher `init_connect`
3. Ajoutez les paramètres du serveur au format suivant : `SET parameter_name=YOUR_DESIRED_VALUE` dans la colonne valeur.

    Par exemple, vous pouvez modifier le jeu de caractères de votre serveur en affectant à `init_connect` la valeur `SET character_set_client=utf8;SET character_set_database=utf8mb4;SET character_set_connection=latin1;SET character_set_results=latin1;`
4. Cliquez sur **Enregistrer** pour enregistrer vos modifications.

## <a name="working-with-the-time-zone-parameter"></a>Utilisation avec le paramètre de fuseau horaire

### <a name="populating-the-time-zone-tables"></a>Remplissage des tables de fuseaux horaires

Les tables de fuseaux horaires sur votre serveur peuvent être remplies en appelant la procédure stockée `mysql.az_load_timezone` à partir d’un outil tel que la ligne de commande MySQL ou MySQL Workbench.

> [!NOTE]
> Si vous exécutez la commande `mysql.az_load_timezone` à partir de MySQL Workbench, vous devrez peut-être d’abord désactiver le mode de mise à jour sécurisée à l’aide de `SET SQL_SAFE_UPDATES=0;`.

```sql
CALL mysql.az_load_timezone();
```

> [!IMPORTANT]
> Vous devez redémarrer le serveur pour vous assurer que les tables des fuseaux horaires sont correctement remplies. Pour redémarrer le serveur, utilisez le [Portail Azure](howto-restart-server-portal.md) ou [CLI](howto-restart-server-cli.md).
Pour afficher les valeurs de fuseau horaire disponibles, exécutez la commande suivante :

```sql
SELECT name FROM mysql.time_zone_name;
```

### <a name="setting-the-global-level-time-zone"></a>Définition du fuseau horaire global

Le fuseau horaire global peut être défini dans la page **Paramètres du serveur** sur le portail Azure. L’exemple ci-dessous définit le fuseau horaire global sur la valeur « US/Pacific ».

![Définir le paramètre de fuseau horaire](./media/howto-server-parameters/timezone.png)

### <a name="setting-the-session-level-time-zone"></a>Définition du fuseau horaire de la session

Le fuseau horaire de la session peut être défini en exécutant la commande `SET time_zone` à partir d’un outil tel que la ligne de commande MySQL ou MySQL Workbench. L’exemple ci-dessous définit le fuseau horaire **US/Pacific**.

```sql
SET time_zone = 'US/Pacific';
```

Reportez-vous à la documentation de MariaDB relative aux [fonction de date et heure](https://mariadb.com/kb/en/library/convert_tz/).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez-en plus sur les [paramètres de serveur](concepts-server-parameters.md)
