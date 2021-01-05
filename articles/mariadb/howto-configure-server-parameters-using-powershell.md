---
title: Configurer les paramètres du serveur - Azure PowerShell - Azure Database for MariaDB
description: Cet article explique comment configurer les paramètres de service d'Azure Database for MariaDB à l'aide de PowerShell.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.devlang: azurepowershell
ms.topic: how-to
ms.date: 10/1/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8ace6306bec4c79cbce0a1572360db1acd2cea97
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94538238"
---
# <a name="configure-server-parameters-in-azure-database-for-mariadb-using-powershell"></a>Configurer les paramètres de serveur dans Azure Database pour MariaDB à l’aide de PowerShell

Vous pouvez lister, afficher et mettre à jour les paramètres de configuration d'un serveur Azure Database for MariaDB à l'aide de PowerShell. Un sous-ensemble de configurations de moteur est exposé au niveau du serveur et peut être modifié.

>[!Note]
> Les paramètres serveur peuvent être mis à jour globalement au niveau du serveur à partir d’[Azure CLI](./howto-configure-server-parameters-cli.md), de [PowerShell](./howto-configure-server-parameters-using-powershell.md) ou du [portail Azure](./howto-server-parameters.md).

## <a name="prerequisites"></a>Prérequis

Pour utiliser ce guide pratique, il vous faut :

- Le [module Az PowerShell](/powershell/azure/install-az-ps) installé localement ou [Azure Cloud Shell](https://shell.azure.com/) dans le navigateur
- Un [serveur Azure Database for MariaDB](quickstart-create-mariadb-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Tant que le module PowerShell Az.MariaDb est en préversion, vous devez l’installer séparément du module Az PowerShell à l’aide de la commande suivante : `Install-Module -Name Az.MariaDb -AllowPrerelease`.
> Une fois le module PowerShell Az.MariaDb généralement disponible, il devient partie intégrante des versions futures du module Az PowerShell et disponible en mode natif dans Azure Cloud Shell.

Si vous choisissez d’utiliser PowerShell en local, connectez-vous à votre compte Azure à l’aide de la cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="list-server-configuration-parameters-for-azure-database-for-mariadb-server"></a>Répertorier les paramètres de configuration de serveur pour Azure Database pour MariaDB

Pour lister tous les paramètres modifiables sur un serveur et leurs valeurs, exécutez l’applet de commande `Get-AzMariaDbConfiguration`.

L’exemple suivant liste les paramètres de configuration du serveur **mydemoserver** dans le groupe de ressources **myresourcegroup**.

```azurepowershell-interactive
Get-AzMariaDbConfiguration -ResourceGroupName myresourcegroup -ServerName mydemoserver
```

Pour obtenir la définition de chacun des paramètres répertoriés, consultez la section de référence MariaDB dans [Server System Variables](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html) (Variables système du serveur).

## <a name="show-server-configuration-parameter-details"></a>Affichage des détails des paramètres de configuration du serveur

Pour afficher les détails d’un paramètre de configuration particulier d’un serveur, exécutez l’applet de commande `Get-AzMariaDbConfiguration` et spécifiez le paramètre **Name**.

Cet exemple montre les détails du paramètre de configuration **slow\_query\_log** pour le serveur **mydemoserver** dans le groupe de ressources **myresourcegroup**.

```azurepowershell-interactive
Get-AzMariaDbConfiguration -Name slow_query_log -ResourceGroupName myresourcegroup -ServerName mydemoserver
```

## <a name="modify-a-server-configuration-parameter-value"></a>Modifier une valeur de paramètre de configuration de serveur

Vous pouvez également modifier la valeur d’un paramètre de configuration de serveur, ce qui a pour effet de mettre à jour la valeur de configuration sous-jacente du moteur du serveur MariaDB. Pour mettre à jour la configuration, utilisez l’applet de commande `Update-AzMariaDbConfiguration`.

Mettez à jour le paramètre de configuration **slow\_query\_log** du serveur **mydemoserver** dans le groupe de ressources **myresourcegroup**.

```azurepowershell-interactive
Update-AzMariaDbConfiguration -Name slow_query_log -ResourceGroupName myresourcegroup -ServerName mydemoserver -Value On
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Croissance automatique du stockage sur le serveur Azure Database for MariaDB à l'aide de PowerShell](howto-auto-grow-storage-powershell.md)