---
title: 'Démarrage rapide : Créer un serveur - Azure PowerShell - Azure Database for MariaDB'
description: Ce guide de démarrage rapide explique comment utiliser PowerShell pour créer un serveur Azure Database for MariaDB dans un groupe de ressources Azure.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.devlang: azurepowershell
ms.topic: quickstart
ms.date: 05/26/2020
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 05082ffa891b72b472ed5433282198c61080f073
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999482"
---
# <a name="quickstart-create-an-azure-database-for-mariadb-server-using-powershell"></a>Démarrage rapide : Créer un serveur Azure Database for MariaDB à l’aide de PowerShell

Ce guide de démarrage rapide explique comment utiliser PowerShell pour créer un serveur Azure Database for MariaDB dans un groupe de ressources Azure. Vous pouvez utiliser PowerShell pour créer et gérer des ressources Azure de façon interactive ou dans des scripts.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

Si vous choisissez d’utiliser PowerShell localement, cet article vous demande d’installer le module Az PowerShell et de vous connecter à votre compte Azure avec l’applet de commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Pour en savoir plus sur l’installation du module Az PowerShell, consultez [Installer Azure PowerShell](/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Tant que le module PowerShell Az.MariaDb est en préversion, vous devez l’installer séparément du module Az PowerShell à l’aide de la commande suivante : `Install-Module -Name Az.MariaDb -AllowPrerelease`.
> Une fois le module PowerShell Az.MariaDb en disponibilité générale, il devient partie intégrante des versions futures du module Az PowerShell et disponible en mode natif dans Azure Cloud Shell.

Si vous utilisez le service Azure Database for MariaDB pour la première fois, vous devez inscrire le fournisseur de ressources **Microsoft.DBforMariaDB**.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.DBforMariaDB
```

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

Si vous avez plusieurs abonnements Azure, sélectionnez l’abonnement approprié dans lequel les ressources doivent être facturées. Sélectionnez un ID d’abonnement spécifique avec l’applet de commande [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un [groupe de ressources Azure](../azure-resource-manager/management/overview.md) avec l’applet de commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées en tant que groupe.

L’exemple suivant crée un groupe de ressources nommé **myresourcegroup** dans la région **USA Ouest**.

```azurepowershell-interactive
New-AzResourceGroup -Name myresourcegroup -Location westus
```

## <a name="create-an-azure-database-for-mariadb-server"></a>Créer un serveur Azure Database for MariaDB

Créez un serveur Azure Database for MariaDB au moyen de la cmdlet `New-AzMariaDbServer`. Un serveur peut gérer plusieurs bases de données. En règle générale, une base de données distincte est utilisée pour chaque projet ou pour chaque utilisateur.

Le tableau suivant contient la liste des paramètres couramment utilisés et des exemples de valeurs pour l’applet de commande `New-AzMariaDbServer`.

|        **Paramètre**         | **Exemple de valeur** |                                                                                                                                                             **Description**                                                                                                                                                              |
| -------------------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Nom                       | mydemoserver     | Choisissez un nom globalement unique dans Azure qui identifie votre serveur Azure Database for MariaDB. Le nom du serveur peut uniquement contenir des lettres, des chiffres et le caractère de trait d’union (-). Les caractères en majuscules spécifiés sont convertis automatiquement en minuscules pendant le processus de création. Il doit inclure entre 3 et 63 caractères. |
| ResourceGroupName          | myResourceGroup  | Indiquez le nom du groupe de ressources Azure.                                                                                                                                                                                                                                                                                            |
| Sku                        | GP_Gen5_2        | Nom de la référence SKU. Suit la convention **niveau tarifaire\_génération de calcul\_vCores** dans le raccourci. Pour plus d’informations sur le paramètre Sku, consultez les informations après ce tableau.                                                                                                                                           |
| BackupRetentionDay         | 7                | Durée pendant laquelle la sauvegarde doit être conservée. Exprimée en jours. La plage s’étend de 7 à 35.                                                                                                                                                                                                                                                                       |
| GeoRedundantBackup         | activé          | Indique si les sauvegardes géoredondantes doivent être activées ou non pour ce serveur. Cette valeur ne peut pas être activée pour les serveurs dans le niveau tarifaire de base et ne peut pas être modifiée après la création du serveur. Valeurs autorisées : Activé, Désactivé.                                                                                                      |
| Emplacement                   | westus           | Région Azure pour le serveur.                                                                                                                                                                                                                                                                                                         |
| SslEnforcement             | activé          | Indique si le protocole SSL doit être activé ou non pour ce serveur. Valeurs autorisées : Activé, Désactivé.                                                                                                                                                                                                                                                 |
| StorageInMb                | 51200            | Capacité de stockage du serveur (en mégaoctets). Un StorageInMb valide est un minimum de 5 120 Mo et augmente par incrément de 1 024 Mo. Pour plus d’informations sur les limites de taille du stockage, consultez [Niveaux tarifaires pour Azure Database for MariaDB](./concepts-pricing-tiers.md).                                                                               |
| Version                    | 5.7              | Version majeure de MariaDB                                                                                                                                                                                                                                                                                                                 |
| AdministratorUserName      | myadmin          | Nom d’utilisateur du compte administrateur. Il ne peut pas être **azure_superuser** (superutilisateur), **admin**, **administrator** (administrateur), **root** (racine), **guest** (invité) ou **public**.                                                                                                                                                                                            |
| AdministratorLoginPassword | `<securestring>` | Mot de passe de l’utilisateur administrateur sous la forme d’une chaîne sécurisée. Il doit contenir entre 8 et 128 caractères. Votre mot de passe doit contenir des caractères de trois des catégories suivantes : Lettres majuscules, lettres minuscules, chiffres et caractères non alphanumériques.                                       |

La valeur du paramètre **Sku** suit la convention **niveau tarifaire\_génération de calcul\_vCores** comme dans les exemples suivants.

- `-Sku B_Gen5_1` correspond à De base, Gen 5 et 1 vCore. Cette option correspond à la plus petite référence disponible.
- `-Sku GP_Gen5_32` correspond à Usage général, Gen 5 et 32 vCores.
- `-Sku MO_Gen5_2` correspond à Mémoire optimisée, Gen 5 et 2 vCores.

Pour plus d’informations sur les valeurs **Sku** valides par région et par niveau, consultez [Niveaux tarifaires pour Azure Database for MariaDB](./concepts-pricing-tiers.md).

L’exemple suivant crée un serveur MariaDB dans la région **USA Ouest** nommé **mydemoserver** dans le groupe de ressources **myresourcegroup** avec une connexion d’administrateur de serveur **myadmin**. Il s’agit d’un serveur de génération 5 dans le niveau tarifaire à usage général avec 2 vCores et des sauvegardes géoredondantes activées. Documentez le mot de passe utilisé sur la première ligne de l’exemple, car il s’agit du mot de passe du compte d’administrateur du serveur MariaDB.

> [!TIP]
> Un nom de serveur est mappé à un nom DNS et doit être globalement unique dans Azure.

```azurepowershell-interactive
$Password = Read-Host -Prompt 'Please enter your password' -AsSecureString
New-AzMariaDbServer -Name mydemoserver -ResourceGroupName myresourcegroup -Sku GP_Gen5_2 -GeoRedundantBackup Enabled -Location westus -AdministratorUsername myadmin -AdministratorLoginPassword $Password
```

Choisissez le niveau tarifaire De base si votre charge de travail n’a pas besoin d’une grande capacité de calcul et d’E/S.

> [!IMPORTANT]
> Par la suite, les serveurs créés avec le niveau tarifaire De base ne pourront plus être mis à l’échelle vers le niveau à usage général ou mémoire optimisée, ni être géorépliqués.

## <a name="configure-a-firewall-rule"></a>Configurer une règle de pare-feu

Créez une règle de pare-feu au niveau du serveur Azure Database for MariaDB en utilisant la cmdlet `New-AzMariaDbFirewallRule`. Une règle de pare-feu au niveau du serveur permet à une application externe (comme l’outil de ligne de commande `mysql` ou MariaDB Workbench) de se connecter à votre serveur par le biais du pare-feu du service Azure Database for MariaDB.

L’exemple suivant crée une règle de pare-feu appelée **AllowMyIP** qui autorise les connexions depuis une adresse IP spécifique, 192.168.0.1. Remplacez une adresse IP ou une plage d’adresses IP qui correspond à l’emplacement à partir duquel vous vous connectez.

```azurepowershell-interactive
New-AzMariaDbFirewallRule -Name AllowMyIP -ResourceGroupName myresourcegroup -ServerName mydemoserver -StartIPAddress 192.168.0.1 -EndIPAddress 192.168.0.1
```

> [!NOTE]
> Les connexions à Azure Database for MariaDB communiquent sur le port 3306. Si vous essayez de vous connecter à partir d’un réseau d’entreprise, le trafic sortant sur le port 3306 peut être bloqué. Dans ce scénario, vous pouvez vous connecter à votre serveur seulement si le service informatique ouvre le port 3306.

## <a name="configure-ssl-settings"></a>Configurer les paramètres SSL

Par défaut, des connexions SSL entre votre serveur et les applications clientes sont appliquées. Ce paramètre par défaut garantit la sécurité des données _en mouvement_ en chiffrant le flux de données sur Internet. Pour ce démarrage rapide, désactivez les connexions SSL de votre serveur. Pour plus d’informations, voir [Configuration de la connectivité SSL dans votre application pour se connecter de manière sécurisée à Azure Database for MariaDB](./howto-configure-ssl.md).

> [!WARNING]
> La désactivation des connexions SSL n’est pas recommandée pour les serveurs de production.

L’exemple suivant désactive SSL sur votre serveur Azure Database for MariaDB.

```azurepowershell-interactive
Update-AzMariaDbServer -Name mydemoserver -ResourceGroupName myresourcegroup -SslEnforcement Disabled
```

## <a name="get-the-connection-information"></a>Obtenir les informations de connexion

Pour vous connecter à votre serveur, vous devez fournir des informations sur l’hôte et des informations d’identification pour l’accès. Utilisez l’exemple suivant pour déterminer les informations de connexion. Notez les valeurs des champs **FullyQualifiedDomainName** et **AdministratorLogin**.

```azurepowershell-interactive
Get-AzMariaDbServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  Select-Object -Property FullyQualifiedDomainName, AdministratorLogin
```

```Output
FullyQualifiedDomainName                    AdministratorLogin
------------------------                    ------------------
mydemoserver.mariadb.database.azure.com       myadmin
```

## <a name="connect-to-the-server-using-the-mysql-command-line-tool"></a>Se connecter au serveur avec l’outil en ligne de commande mysql

Connectez-vous à votre serveur avec l’outil en ligne de commande `mysql`. Pour télécharger et installer l’outil en ligne de commande, consultez [MySQL Community Downloads](https://dev.mysql.com/downloads/shell/). Vous pouvez aussi accéder à une version préinstallée de l’outil en ligne de commande `mysql` dans Azure Cloud Shell en sélectionnant le bouton **Essayer** sur un exemple de code de cet article. D’autres moyens d’accéder à Azure Cloud Shell consistent à sélectionner le bouton **>_** dans la barre d’outils en haut à droite du portail Azure ou en visitant [shell.azure.com](https://shell.azure.com/).

1. Connectez-vous au serveur avec l’outil en ligne de commande `mysql`.

   ```azurepowershell-interactive
   mysql -h <servername>.mariadb.database.azure.com -u myadmin@<servername> -p
   ```

1. Consultez l’état du serveur.

   ```sql
   mysql> status
   ```

   ```Output
   C:\Users\>mysql -h mydemoserver.mariadb.database.azure.com -u myadmin@mydemoserver -p
   Enter password: *************
   Welcome to the MySQL monitor.  Commands end with ; or \g.
   Your MySQL connection id is 64793
   Server version: 5.6.42.0 MariaDB Server

   Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

   Oracle is a registered trademark of Oracle Corporation and/or its
   affiliates. Other names may be trademarks of their respective
   owners.

   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

   mysql> status
   --------------
   /usr/bin/mysql  Ver 14.14 Distrib 5.7.29, for Linux (x86_64) using  EditLine wrapper

   Connection id:          64793
   Current database:
   Current user:           myadmin@myipaddress
   SSL:                    Cipher in use is ECDHE-RSA-AES256-GCM-SHA384
   Current pager:          stdout
   Using outfile:          ''
   Using delimiter:        ;
   Server version:         5.6.42.0 MariaDB Server
   Protocol version:       10
   Connection:             mydemoserver.mariadb.database.azure.com via TCP/IP
   Server characterset:    latin1
   Db     characterset:    latin1
   Client characterset:    utf8
   Conn.  characterset:    utf8
   TCP port:               3306
   Uptime:                 17 min 4 sec

   Threads: 19  Questions: 482  Slow queries: 0  Opens: 50  Flush tables: 3  Open tables: 12  Queries per second avg: 0.470
   --------------

   mysql>
   ```

Pour les autres commandes, consultez le [Manuel de référence de MySQL 5.7 - Chapitre 4.5.1](https://dev.mysql.com/doc/refman/5.7/en/mysql.html).

## <a name="connect-to-the-server-using-mariadb-workbench"></a>Se connecter au serveur à l’aide de MariaDB Workbench

1. Lancez l’application MySQL Workbench sur votre ordinateur client. Pour télécharger et installer MySQL Workbench, consultez [Download MySQL Workbench](https://dev.mysql.com/downloads/workbench/).

1. Dans la boîte de dialogue **Configurer une nouvelle connexion**, entrez les informations suivantes dans l’onglet **Paramètres** :

   ![configurer une nouvelle connexion](./media/quickstart-create-mariadb-server-database-using-azure-powershell/setup-new-connection.png)

    |    **Paramètre**    |            **Valeur suggérée**            |                      **Description**                       |
    | ----------------- | ----------------------------------------- | ---------------------------------------------------------- |
    | Nom de connexion   | Ma connexion                             | Spécifier une étiquette pour cette connexion                        |
    | Méthode de connexion | Standard (TCP/IP)                         | Utilisez le protocole TCP/IP pour vous connecter à Azure Database for MariaDB |
    | HostName          | `mydemoserver.mariadb.database.azure.com` | Nom du serveur que vous avez noté précédemment                           |
    | Port              | 3306                                      | Port par défaut pour MariaDB                                 |
    | Nom d’utilisateur          | myadmin@mydemoserver                      | Connexion d’administrateur serveur que vous avez notée précédemment                |
    | Mot de passe          | *************                             | Utiliser le mot de passe du compte Administrateur que vous avez configuré précédemment      |

1. Pour tester si les paramètres sont correctement configurés, cliquez sur le bouton **Tester la connexion**.

1. Sélectionnez la connexion pour vous connecter au serveur.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez pas besoin des ressources que vous avez créées dans ce démarrage rapide pour un autre démarrage rapide ou un tutoriel, vous pouvez les supprimer en exécutant l’exemple suivant.

> [!CAUTION]
> L’exemple suivant supprime le groupe de ressources spécifié et toutes les ressources qu’il contient.
> Si des ressources en dehors du cadre de ce démarrage rapide existent dans le groupe de ressources spécifié, elles seront également supprimées.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myresourcegroup
```

Pour supprimer uniquement le serveur créé dans ce démarrage rapide sans supprimer le groupe de ressources, utilisez l’applet de commande `Remove-AzMariaDbServer`.

```azurepowershell-interactive
Remove-AzMariaDbServer -Name mydemoserver -ResourceGroupName myresourcegroup
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Concevoir une base de données Azure Database for MariaDB à l’aide de PowerShell](tutorial-design-database-using-powershell.md)