---
title: Générer une chaîne de connexion avec PowerShell – Azure Database pour PostgreSQL
description: Cet article fournit un exemple Azure PowerShell pour générer une chaîne de connexion à Azure Database pour PostgreSQL.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.custom: mvc, devx-track-azurepowershell
ms.topic: how-to
ms.date: 8/6/2020
ms.openlocfilehash: 29641bdc3696f0722ca430dde912768071fd9b16
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91708167"
---
# <a name="how-to-generate-an-azure-database-for-postgresql-connection-string-with-powershell"></a>Comment générer une chaîne de connexion Azure Database pour PostgreSQL avec PowerShell

Cet article montre comment générer une chaîne de connexion pour un serveur Azure Database pour PostgreSQL. Vous pouvez utiliser une chaîne de connexion pour vous connecter à une instance Azure Database pour PostgreSQL à partir de nombreuses applications différentes.

## <a name="requirements"></a>Spécifications

Cet article utilise comme point de départ les ressources créées dans le guide suivant :

* [Démarrage rapide : Créer un serveur Azure Database pour PostgreSQL à l’aide de PowerShell](quickstart-create-postgresql-server-database-using-azure-powershell.md)

## <a name="get-the-connection-string"></a>Obtenir la chaîne de connexion

La cmdlet `Get-AzPostgreSqlConnectionString` est utilisée pour générer une chaîne de connexion afin de connecter des applications à Azure Database pour PostgreSQL. L’exemple suivant retourne la chaîne de connexion pour un client PHP depuis **mydemoserver**.

```azurepowershell-interactive
Get-AzPostgreSqlConnectionString -Client PHP -Name mydemoserver -ResourceGroupName myresourcegroup
```

```Output
host=mydemoserver.postgres.database.azure.com port=5432 dbname={your_database} user=myadmin@mydemoserver password={your_password} sslmode=require
```

Les valeurs valides pour le paramètre `Client` sont :

* ADO&#46;NET
* JDBC
* Node.js
* PHP
* Python
* Ruby
* WebApp

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Personnaliser les paramètres de serveur Azure Database pour MySQL à l’aide de PowerShell](howto-configure-server-parameters-using-powershell.md)
