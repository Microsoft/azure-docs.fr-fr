---
title: 'Tutoriel : Concevoir un serveur - Azure PowerShell - Azure Database pour MySQL'
description: Ce tutoriel explique comment créer et gérer un serveur et une base de données Azure Database pour MySQL à l’aide de PowerShell.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurepowershell
ms.topic: tutorial
ms.date: 04/29/2020
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: fd8294d60ed0af4e8d1eeb8a3cd07c737b69aadd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011453"
---
# <a name="tutorial-design-an-azure-database-for-mysql-using-powershell"></a>Tutoriel : Concevoir une base de données Azure Database pour MySQL à l’aide de PowerShell

Azure Database pour MySQL est un service de base de données relationnelle dans le cloud de Microsoft basé sur le moteur de base de données MySQL Community Edition. Dans ce tutoriel, vous allez utiliser PowerShell et d’autres utilitaires pour apprendre à :

> [!div class="checklist"]
> - Créer une base de données Azure pour MySQL
> - Configurer le pare-feu du serveur
> - Utiliser [l’outil de ligne de commande mysql](https://dev.mysql.com/doc/refman/5.6/en/mysql.html) pour créer une base de données
> - Charger un exemple de données
> - Interroger des données
> - Mettre à jour des données
> - Restaurer des données

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

Si vous choisissez d’utiliser PowerShell localement, cet article vous demande d’installer le module Az PowerShell et de vous connecter à votre compte Azure avec l’applet de commande [Connect-AzAccount](/powershell/module/az.accounts/Connect-AzAccount). Pour en savoir plus sur l’installation du module Az PowerShell, consultez [Installer Azure PowerShell](/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Tant que le module Az.MySql PowerShell est en préversion, vous devez l’installer séparément du module Az PowerShell à l’aide de la commande suivante : `Install-Module -Name Az.MySql -AllowPrerelease`.
> Une fois que le module Az.MySql PowerShell est en disponibilité générale, il devient partie intégrante des versions futures du module Az PowerShell et disponible en mode natif dans Azure Cloud Shell.

Si c’est la première fois que vous utilisez le service Azure Database pour MySQL, vous devez inscrire le fournisseur de ressources **Microsoft.DBforMySQL**.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.DBforMySQL
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

## <a name="create-an-azure-database-for-mysql-server"></a>Création d’un serveur Azure Database pour MySQL

Créez un serveur Azure Database pour MySQL avec l’applet de commande `New-AzMySqlServer`. Un serveur peut gérer plusieurs bases de données. En règle générale, une base de données distincte est utilisée pour chaque projet ou pour chaque utilisateur.

L’exemple suivant crée un serveur MySQL dans la région **USA Ouest** nommé **mydemoserver** dans le groupe de ressources **myresourcegroup** avec une connexion d’administrateur de serveur **myadmin**. Il s’agit d’un serveur de génération 5 dans le niveau tarifaire à usage général avec 2 vCores et des sauvegardes géoredondantes activées. Documentez le mot de passe utilisé sur la première ligne de l’exemple, car il s’agit du mot de passe du compte d’administrateur du serveur MySQL.

> [!TIP]
> Un nom de serveur est mappé à un nom DNS et doit être globalement unique dans Azure.

```azurepowershell-interactive
$Password = Read-Host -Prompt 'Please enter your password' -AsSecureString
New-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup -Sku GP_Gen5_2 -GeoRedundantBackup Enabled -Location westus -AdministratorUsername myadmin -AdministratorLoginPassword $Password
```

La valeur du paramètre **Sku** suit la convention **niveau tarifaire\_génération de calcul\_vCores** comme illustré dans les exemples suivants.

- `-Sku B_Gen5_1` correspond à De base, Gen 5 et 1 vCore. Cette option correspond à la plus petite référence disponible.
- `-Sku GP_Gen5_32` correspond à Usage général, Gen 5 et 32 vCores.
- `-Sku MO_Gen5_2` correspond à Mémoire optimisée, Gen 5 et 2 vCores.

Pour plus d’informations sur les valeurs **Sku** valides par région et par niveau, consultez [Niveaux tarifaires Azure Database pour MySQL](./concepts-pricing-tiers.md).

Choisissez le niveau tarifaire De base si votre charge de travail n’a pas besoin d’une grande capacité de calcul et d’E/S élevées.

> [!IMPORTANT]
> Par la suite, les serveurs créés avec le niveau tarifaire De base ne pourront plus être mis à l’échelle vers le niveau à usage général ou mémoire optimisée, ni être géorépliqués.

## <a name="configure-a-firewall-rule"></a>Configurer une règle de pare-feu

Créez une règle de pare-feu au niveau du serveur Azure Database pour MySQL en utilisant l’applet de commande `New-AzMySqlFirewallRule`. Une règle de pare-feu au niveau du serveur permet à une application externe, comme l’outil de ligne de commande `mysql` ou MySQL Workbench, de se connecter à votre serveur via le pare-feu du service Azure Database pour MySQL.

L’exemple suivant crée une règle de pare-feu appelée **AllowMyIP** qui autorise les connexions depuis une adresse IP spécifique, 192.168.0.1. Remplacez une adresse IP ou une plage d’adresses IP qui correspond à l’emplacement à partir duquel vous vous connectez.

```azurepowershell-interactive
New-AzMySqlFirewallRule -Name AllowMyIP -ResourceGroupName myresourcegroup -ServerName mydemoserver -StartIPAddress 192.168.0.1 -EndIPAddress 192.168.0.1
```

> [!NOTE]
> Les connexions à la base de données Azure pour MySQL communiquent via le port 3306. Si vous essayez de vous connecter à partir d’un réseau d’entreprise, le trafic sortant sur le port 3306 peut être bloqué. Dans ce scénario, vous pouvez vous connecter à votre serveur seulement si le service informatique ouvre le port 3306.

## <a name="get-the-connection-information"></a>Obtenir les informations de connexion

Pour vous connecter à votre serveur, vous devez fournir des informations sur l’hôte et des informations d’identification pour l’accès. Utilisez l’exemple suivant pour déterminer les informations de connexion. Notez les valeurs des champs **FullyQualifiedDomainName** et **AdministratorLogin**.

```azurepowershell-interactive
Get-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  Select-Object -Property FullyQualifiedDomainName, AdministratorLogin
```

```Output
FullyQualifiedDomainName                    AdministratorLogin
------------------------                    ------------------
mydemoserver.mysql.database.azure.com       myadmin
```

## <a name="connect-to-the-server-using-the-mysql-command-line-tool"></a>Se connecter au serveur avec l’outil en ligne de commande mysql

Connectez-vous à votre serveur avec l’outil en ligne de commande `mysql`. Pour télécharger et installer l’outil en ligne de commande, consultez [MySQL Community Downloads](https://dev.mysql.com/downloads/shell/). Vous pouvez aussi accéder à une version préinstallée de l’outil en ligne de commande `mysql` dans Azure Cloud Shell en sélectionnant le bouton **Essayer** sur un exemple de code de cet article. D’autres moyens d’accéder à Azure Cloud Shell consistent à sélectionner le bouton **>_** dans la barre d’outils en haut à droite du portail Azure ou à accéder à [shell.azure.com](https://shell.azure.com/).

```azurepowershell-interactive
mysql -h mydemoserver.mysql.database.azure.com -u myadmin@mydemoserver -p
```

## <a name="create-a-database"></a>Création d'une base de données

Une fois que vous êtes connecté au serveur, créez une base de données vide.

```sql
mysql> CREATE DATABASE mysampledb;
```

À l’invite, exécutez la commande suivante pour basculer la connexion sur la base de données nouvellement créée :

```sql
mysql> USE mysampledb;
```

## <a name="create-tables-in-the-database"></a>Créer des tables dans la base de données

Maintenant que vous savez vous connecter à la base de données Azure Database pour MySQL, effectuez certaines tâches de base.

Tout d’abord, créez une table et chargez-y des données. Nous allons créer une table qui stocke des données d’inventaire.

```sql
CREATE TABLE inventory (
  id serial PRIMARY KEY,
  name VARCHAR(50),
  quantity INTEGER
);
```

## <a name="load-data-into-the-tables"></a>Charger des données dans les tables

Maintenant que vous disposez d’une table, insérez-y des données. Dans la fenêtre d’invite de commandes ouverte, exécutez la requête suivante pour insérer des lignes de données.

```sql
INSERT INTO inventory (id, name, quantity) VALUES (1, 'banana', 150);
INSERT INTO inventory (id, name, quantity) VALUES (2, 'orange', 154);
```

Vous avez maintenant chargé deux lignes de données dans la table que vous avez créée précédemment.

## <a name="query-and-update-the-data-in-the-tables"></a>Interroger et mettre à jour les données des tables

Exécutez la requête suivante pour récupérer des informations à partir de la table de base de données.

```sql
SELECT * FROM inventory;
```

Vous pouvez également mettre à jour les données des tables.

```sql
UPDATE inventory SET quantity = 200 WHERE name = 'banana';
```

La ligne est mise à jour en conséquence lorsque vous récupérez les données.

```sql
SELECT * FROM inventory;
```

## <a name="restore-a-database-to-a-previous-point-in-time"></a>Restaurer une version antérieure d’une base de données

Vous pouvez restaurer le serveur à un point dans le temps antérieur. Les données restaurées sont copiées sur un nouveau serveur, et le serveur existant est conservé tel quel. Par exemple, si une table est accidentellement supprimée, vous pouvez restaurer le serveur à l’état qu’il présentait au moment de la suppression. Vous pouvez ensuite récupérer la table et les données manquantes à partir de la copie restaurée du serveur.

Pour restaurer le serveur, utilisez l’applet de commande PowerShell `Restore-AzMySqlServer`.

### <a name="run-the-restore-command"></a>Exécuter la commande de restauration

Pour restaurer le serveur, exécutez l’exemple suivant à partir de PowerShell.

```azurepowershell-interactive
$restorePointInTime = (Get-Date).AddMinutes(-10)
Get-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  Restore-AzMySqlServer -Name mydemoserver-restored -ResourceGroupName myresourcegroup -RestorePointInTime $restorePointInTime -UsePointInTimeRestore
```

Quand vous restaurez un serveur à un point dans le temps antérieur, un nouveau serveur est créé. Le serveur d’origine et ses bases de données au point dans le temps spécifié sont copiés sur le nouveau serveur.

Les valeurs d’emplacement et de niveau tarifaire du serveur restauré restent les mêmes que celles du serveur d’origine.

Une fois la restauration terminée, recherchez le nouveau serveur et vérifiez que les données ont été restaurées correctement. Le nouveau serveur a les mêmes nom de connexion et mot de passe d’administrateur de serveur que ceux valides pour le serveur existant au moment du démarrage de la restauration. Le mot de passe peut être modifié sur la page **Vue d’ensemble** du nouveau serveur.

Le nouveau serveur créé lors d’une restauration ne dispose pas des points de terminaison de service de réseau virtuel qui se trouvaient sur le serveur d’origine. Ces règles doivent être configurées séparément pour le nouveau serveur. Les règles de pare-feu du serveur d’origine sont restaurées.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez pas besoin des ressources que vous avez créées dans ce tutoriel pour un autre guide de démarrage rapide ou tutoriel, vous pouvez les supprimer en exécutant l’exemple suivant.

> [!CAUTION]
> L’exemple suivant supprime le groupe de ressources spécifié et toutes les ressources qu’il contient.
> Si des ressources en dehors du cadre de ce tutoriel existent dans le groupe de ressources spécifié, elles seront également supprimées.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myresourcegroup
```

Pour supprimer uniquement le serveur créé dans ce tutoriel sans supprimer le groupe de ressources, utilisez l’applet de commande `Remove-AzMySqlServer`.

```azurepowershell-interactive
Remove-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Guide pratique pour sauvegarder et restaurer un serveur Azure Database pour MySQL à l’aide de PowerShell](howto-restore-server-powershell.md)