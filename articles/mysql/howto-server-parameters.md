---
title: Configurer les paramètres de serveur - Portail Azure - Azure Database pour MySQL
description: Cet article décrit comment configurer les paramètres de serveur MySQL dans Azure Database pour MySQL à l’aide du portail Azure.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 10/1/2020
ms.openlocfilehash: 363be8b34f230b812bc24276e1f3925faf0cdc1c
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94540839"
---
# <a name="configure-server-parameters-in-azure-database-for-mysql-using-the-azure-portal"></a>Configurer les paramètres de serveur dans Azure Database pour MySQL

Azure Database pour MySQL prend en charge la configuration de certains paramètres de serveur. Cet article décrit comment configurer ces paramètres à l’aide du portail Azure. Les paramètres du serveur ne sont pas tous modifiables.

>[!Note]
> Les paramètres serveur peuvent être mis à jour globalement au niveau du serveur à partir d’[Azure CLI](./howto-configure-server-parameters-using-cli.md), de [PowerShell](./howto-configure-server-parameters-using-powershell.md) ou du [portail Azure](./howto-server-parameters.md).

## <a name="configure-server-parameters"></a>Configurer les paramètres du serveur

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis recherchez votre serveur Azure Database pour MySQL.
2. Dans la section **PARAMÈTRES**, cliquez sur **Paramètres du serveur** pour ouvrir la page de paramètres du serveur Azure Database pour MySQL.
:::image type="content" source="./media/howto-server-parameters/auzre-portal-server-parameters.png" alt-text="Page Paramètres du serveur du portail Azure":::
3. Recherchez les paramètres que vous devez ajuster. Examinez la colonne **Description** pour comprendre la fonction et les valeurs autorisées.
:::image type="content" source="./media/howto-server-parameters/3-toggle_parameter.png" alt-text="Bouton déroulant Énumérer":::
4. Cliquez sur **Enregistrer** pour enregistrer vos modifications.
:::image type="content" source="./media/howto-server-parameters/4-save_parameters.png" alt-text="Enregistrer ou annuler les modifications":::
5. Si vous avez enregistré de nouvelles valeurs pour les paramètres, vous pouvez toujours rétablir toutes les valeurs par défaut en sélectionnant **Rétablir toutes les valeurs par défaut**.
:::image type="content" source="./media/howto-server-parameters/5-reset_parameters.png" alt-text="Rétablir toutes les valeurs par défaut":::

## <a name="setting-parameters-not-listed"></a>Configurer les paramètres non listés

Si le paramètre de serveur que vous souhaitez mettre à jour n’est pas listé dans le portail Azure, vous pouvez éventuellement définir le paramètre au niveau de la connexion avec `init_connect`. Cela définit les paramètres de serveur pour chaque client qui se connecte. 

1. Dans la section **PARAMÈTRES**, cliquez sur **Paramètres du serveur** pour ouvrir la page de paramètres du serveur Azure Database pour MySQL.
2. Rechercher `init_connect`
3. Ajoutez les paramètres du serveur au format suivant : `SET parameter_name=YOUR_DESIRED_VALUE` dans la colonne valeur.

    Par exemple, vous pouvez modifier le jeu de caractères de votre serveur en affectant à `init_connect` la valeur `SET character_set_client=utf8;SET character_set_database=utf8mb4;SET character_set_connection=latin1;SET character_set_results=latin1;`
4. Cliquez sur **Enregistrer** pour enregistrer vos modifications.

>[!Note]
> `init_connect` peut être utilisé pour modifier des paramètres qui ne nécessitent pas de privilèges de super utilisateur (SUPER) au niveau de la session. Pour vérifier si vous pouvez définir le paramètre avec `init_connect`, exécutez la commande `set session parameter_name=YOUR_DESIRED_VALUE;` et, si vous obtenez une erreur **Accès refusé. Vous devez avoir des privilèges SUPER**, cela indique que vous n’êtes pas autorisé à définir le paramètre avec « init_connect ».

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

:::image type="content" source="./media/howto-server-parameters/timezone.png" alt-text="Définir le paramètre de fuseau horaire":::

### <a name="setting-the-session-level-time-zone"></a>Définition du fuseau horaire de la session

Le fuseau horaire de la session peut être défini en exécutant la commande `SET time_zone` à partir d’un outil tel que la ligne de commande MySQL ou MySQL Workbench. L’exemple ci-dessous définit le fuseau horaire **US/Pacific**.

```sql
SET time_zone = 'US/Pacific';
```

Reportez-vous à la documentation de MySQL sur les [fonction de date et heure](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_convert-tz).

## <a name="next-steps"></a>Étapes suivantes

- [Bibliothèques de connexions pour Azure Database pour MySQL](concepts-connection-libraries.md).
