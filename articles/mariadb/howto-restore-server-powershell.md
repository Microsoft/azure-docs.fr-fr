---
title: Sauvegarde et restauration – Azure PowerShell – Azure Database for MariaDB
description: Découvrez comment sauvegarder et restaurer un serveur dans Azure Database for MariaDB à l’aide d’Azure PowerShell.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.devlang: azurepowershell
ms.topic: how-to
ms.date: 05/26/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0207be2c983fd986d5852403e36462d2d7d2cdda
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94539615"
---
# <a name="how-to-back-up-and-restore-an-azure-database-for-mariadb-server-using-powershell"></a>Comment sauvegarder et restaurer un serveur Azure Database for MariaDB à l’aide de PowerShell

Les serveurs Azure Database for MariaDB sont sauvegardés régulièrement pour activer les fonctionnalités de restauration. À l’aide de cette fonctionnalité, vous pouvez restaurer le serveur et toutes ses bases de données à un point dans le temps antérieur, sur un nouveau serveur.

## <a name="prerequisites"></a>Prérequis

Pour utiliser ce guide pratique, il vous faut :

- Le [module Az PowerShell](/powershell/azure/install-az-ps) installé localement ou [Azure Cloud Shell](https://shell.azure.com/) dans le navigateur
- Un [serveur Azure Database for MariaDB](quickstart-create-mariadb-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Tant que le module PowerShell Az.MariaDb est en préversion, vous devez l'installer séparément du module Az PowerShell à l'aide de la commande suivante : `Install-Module -Name Az.MariaDb -AllowPrerelease`.
> Une fois le module PowerShell Az.MariaDb généralement disponible, il devient partie intégrante des versions futures du module Az PowerShell et disponible en mode natif dans Azure Cloud Shell.

Si vous choisissez d’utiliser PowerShell en local, connectez-vous à votre compte Azure à l’aide de la cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="set-backup-configuration"></a>Définir la configuration de sauvegarde

Lors de la création du serveur, vous choisissez entre la configuration de votre serveur pour des sauvegardes redondantes localement ou géographiquement redondantes.

> [!NOTE]
> Après la création d’un serveur, son type de redondance (géographique ou locale) ne peut pas être modifié.

Lors de la création d’un serveur via la commande `New-AzMariaDbServer`, le paramètre **GeoRedundantBackup** décide de votre option de redondance de sauvegarde. Si **Activé**, des sauvegardes géoredondantes sont effectuées. Si **Désactivé**, des sauvegardes redondantes localement sont effectuées.

La période de rétention des sauvegardes est définie par le paramètre **BackupRetentionDay**.

Pour plus d’informations sur la définition de ces valeurs lors de la création d’un serveur, consultez [Créer un serveur Azure Database for MariaDB à l’aide de PowerShell](quickstart-create-mariadb-server-database-using-azure-powershell.md).

La période de rétention des sauvegardes d’un serveur peut être modifiée comme suit :

```azurepowershell-interactive
Update-AzMariaDbServer -Name mydemoserver -ResourceGroupName myresourcegroup -BackupRetentionDay 10
```

L’exemple précédent modifie la période de période de rétention des sauvegardes de mydemoserver à 10 jours.

La période de rétention de sauvegarde détermine jusqu’à quelle date une restauration à un point dans le temps peut être récupérée, dans la mesure où elle est basée sur les sauvegardes disponibles. La limite de restauration dans le temps est décrite dans la section suivante.

## <a name="server-point-in-time-restore"></a>Limite de restauration dans le temps

Vous pouvez restaurer le serveur à un point antérieur dans le temps. Les données restaurées sont copiées sur un nouveau serveur, et le serveur existant est conservé tel quel. Par exemple, si une table est accidentellement supprimée, vous pouvez restaurer le serveur à l’état qu’il présentait au moment de la suppression. Vous pouvez ensuite récupérer la table et les données manquantes à partir de la copie restaurée du serveur.

Pour restaurer le serveur, utilisez l’applet de commande PowerShell `Restore-AzMariaDbServer`.

### <a name="run-the-restore-command"></a>Exécuter la commande de restauration

Pour restaurer le serveur, exécutez l’exemple suivant à partir de PowerShell.

```azurepowershell-interactive
$restorePointInTime = (Get-Date).AddMinutes(-10)
Get-AzMariaDbServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  Restore-AzMariaDbServer -Name mydemoserver-restored -ResourceGroupName myresourcegroup -RestorePointInTime $restorePointInTime -UsePointInTimeRestore
```

Le jeu de paramètres **PointInTimeRestore** de la cmdlet `Restore-AzMariaDbServer` requiert les paramètres suivants :

| Paramètre | Valeur suggérée | Description  |
| --- | --- | --- |
| ResourceGroupName |  myResourceGroup |  Groupe de ressources où se trouve le serveur source.  |
| Nom | mydemoserver-restored | Nom du serveur créé par la commande de restauration. |
| RestorePointInTime | 2020-03-13T13:59:00Z | Sélectionnez un point dans le temps à restaurer. Elles doivent être comprises dans la période de rétention de la sauvegarde du serveur source. Utilisez le format de date et d’heure ISO8601. Par exemple, vous pouvez utiliser votre fuseau horaire local, comme **2020-03-13T05:59:00-08:00**. Vous pouvez également utiliser le format UTC Zulu, par exemple, **2018-03-13T13:59:00Z**. |
| UsePointInTimeRestore | `<SwitchParameter>` | Utilisez le mode de point dans le temps pour effectuer la restauration. |

Lorsque vous restaurez un serveur à un point dans le temps antérieur, un nouveau serveur est créé. Le serveur d’origine et ses bases de données au point dans le temps spécifié sont copiés sur le nouveau serveur.

Les valeurs d’emplacement et de niveau tarifaire du serveur restauré restent les mêmes que celles du serveur d’origine.

Une fois la restauration terminée, recherchez le nouveau serveur et vérifiez que les données ont été restaurées correctement. Le nouveau serveur a les mêmes nom de connexion et mot de passe d’administrateur de serveur que ceux valides pour le serveur existant au moment du démarrage de la restauration. Le mot de passe peut être modifié sur la page **Vue d’ensemble** du nouveau serveur.

Le nouveau serveur créé lors d’une restauration ne dispose pas des points de terminaison de service de réseau virtuel qui se trouvaient sur le serveur d’origine. Ces règles doivent être configurées séparément pour le nouveau serveur. Les règles de pare-feu du serveur d’origine sont restaurées.

## <a name="geo-restore"></a>Restauration géographique

Si vous avez configuré votre serveur pour les sauvegardes redondantes géographiquement, un serveur peut être créé à partir de la sauvegarde du serveur existant. Ce nouveau serveur peut être créé dans toutes les régions dans lesquelles Azure Database for MariaDB est disponible.

Pour créer un serveur à l’aide d’une sauvegarde géoredondante, utilisez la commande `Restore-AzMariaDbServer` avec le paramètre **UseGeoRestore**.

> [!NOTE]
> Lorsqu’un serveur est créé, il est possible qu’il ne soit pas immédiatement disponible pour la restauration géographique. Le remplissage des métadonnées nécessaires peut prendre quelques heures.

Pour effectuer une restauration géographique du serveur, exécutez l’exemple suivant à partir de PowerShell :

```azurepowershell-interactive
Get-AzMariaDbServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  Restore-AzMariaDbServer -Name mydemoserver-georestored -ResourceGroupName myresourcegroup -Location eastus -Sku GP_Gen5_8 -UseGeoRestore
```

Cet exemple crée un nouveau serveur appelé **mydemoserver-georestored** dans la région USA Est appartenant à **myresourcegroup**. Il s’agit d’un serveur à usage général, de 5e génération avec 8 vCores. Le serveur est créé à partir de la sauvegarde géoredondante **mydemoserver**, qui est également dans le groupe de ressources **myresourcegroup**.

Pour créer le nouveau serveur dans un groupe de ressources différent du serveur existant, spécifiez le nouveau nom du groupe de ressources à l’aide du paramètre **ResourceGroupName** comme indiqué dans l’exemple suivant :

```azurepowershell-interactive
Get-AzMariaDbServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  Restore-AzMariaDbServer -Name mydemoserver-georestored -ResourceGroupName newresourcegroup -Location eastus -Sku GP_Gen5_8 -UseGeoRestore
```

Le jeu de paramètres **GeoRestore** de la cmdlet `Restore-AzMariaDbServer` requiert les paramètres suivants :

| Paramètre | Valeur suggérée | Description  |
| --- | --- | --- |
|ResourceGroupName | myResourceGroup | Le nom du groupe de ressources auquel appartient le nouveau serveur.|
|Nom | mydemoserver-georestored | Le nom du nouveau serveur. |
|Emplacement | eastus | L’emplacement du nouveau serveur. |
|UseGeoRestore | `<SwitchParameter>` | Utilisez le mode géo pour effectuer la restauration. |

Lorsque vous créez un nouveau serveur à l’aide de la restauration géographique, il hérite de la même taille de stockage et du même niveau tarifaire que le serveur source, sauf si le paramètre **Sku** est spécifié.

Une fois la restauration terminée, recherchez le nouveau serveur et vérifiez que les données ont été restaurées correctement. Le nouveau serveur a les mêmes nom de connexion et mot de passe d’administrateur de serveur que ceux valides pour le serveur existant au moment du démarrage de la restauration. Le mot de passe peut être modifié sur la page **Vue d’ensemble** du nouveau serveur.

Le nouveau serveur créé lors d’une restauration ne dispose pas des points de terminaison de service de réseau virtuel qui se trouvaient sur le serveur d’origine. Ces règles doivent être configurées séparément pour ce nouveau serveur. Les règles de pare-feu du serveur d’origine sont restaurées.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Comment générer une chaîne de connexion à Azure Database for MariaDB avec PowerShell](howto-connection-string-powershell.md)