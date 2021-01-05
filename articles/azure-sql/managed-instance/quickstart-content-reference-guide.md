---
title: Références vers les contenus permettant de démarrer
titleSuffix: Azure SQL Managed Instance
description: 'Références vers les contenus vous permettant de démarrer avec Azure SQL Managed Instance '
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: quickstart
author: davidtrigano
ms.author: datrigan
ms.reviewer: vanto
ms.date: 07/11/2019
ms.openlocfilehash: 936e4f8f54e92ba90372fff1c9d8dfc1982bbd62
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96325113"
---
# <a name="getting-started-with-azure-sql-managed-instance"></a>Bien démarrer avec Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

[Azure SQL Managed Instance](sql-managed-instance-paas-overview.md) crée une base de données presque 100 % compatible avec le moteur de base de données SQL Server (Édition Entreprise) le plus récent et fournit une implémentation de [réseau virtuel (VNet)](../../virtual-network/virtual-networks-overview.md) native traitant les problèmes de sécurité courants ainsi qu’un [modèle d’entreprise](https://azure.microsoft.com/pricing/details/sql-database/) favorable aux clients SQL Server existants.

Dans cet article, vous trouverez des références vers les contenus vous expliquant comment créer et configurer rapidement une instance managée SQL et comment migrer vos bases de données.

## <a name="quickstart-overview"></a>Vue d’ensemble du guide de démarrage rapide

Les guides de démarrage rapide suivants vous permettent de créer rapidement une instance managée SQL, de configurer une machine virtuelle ou une connexion VPN point à site pour l’application cliente et de restaurer une base de données sur votre nouvelle instance managée SQL à l’aide d’un fichier `.bak`.

### <a name="configure-environment"></a>Configurer l’environnement

Dans un premier temps, vous devez créer votre première instance managée SQL avec l’environnement réseau dans lequel elle va être placée, puis activer la connexion à partir de l’ordinateur ou de la machine virtuelle où vous exécutez les requêtes vers l’instance. Vous pouvez utiliser les guides suivants :

- [Créer une instance managée SQL à l’aide du portail Azure](instance-create-quickstart.md). Dans le portail Azure, vous configurez les paramètres nécessaires (nom d’utilisateur/mot de passe, nombre de cœurs et quantité de stockage maximal) pour créer automatiquement l’environnement réseau Azure, sans vous occuper des détails du réseau ni des besoins d’infrastructure. Vous devez simplement vérifier que vous avez un [type d’abonnement](resource-limits.md#supported-subscription-types) actuellement autorisé à créer une instance managée SQL. Si vous disposez de votre propre réseau à utiliser ou que vous souhaitez personnaliser le réseau, consultez [Configurer un réseau virtuel existant pour Azure SQL Managed Instance](vnet-existing-add-subnet.md) ou [Créer un réseau virtuel pour Azure SQL Managed Instance](virtual-network-subnet-create-arm-template.md).
- Une instance managée SQL est créée dans son propre réseau virtuel sans aucun point de terminaison public. Pour l’accès aux applications clientes, vous pouvez **créer une machine virtuelle dans le même réseau virtuel (sous-réseau différent)** ou **créer une connexion VPN point à site au réseau virtuel à partir de votre ordinateur client**, à l’aide de l’un de ces guides de démarrage rapide :
  - Activez un [point de terminaison public](public-endpoint-configure.md) sur votre instance managée SQL pour accéder à vos données directement à partir de votre environnement.
  - Créez une [machine virtuelle Azure dans le réseau virtuel de l’instance managée SQL](connect-vm-instance-configure.md) pour la connectivité aux applications clientes, notamment SQL Server Management Studio.
  - Configurez une [connexion VPN de point à site à votre instance managée SQL](point-to-site-p2s-configure.md) à partir de votre ordinateur client où se trouvent SQL Server Management Studio et d’autres applications de connectivité clientes. Il s’agit là de l’une des deux options disponibles pour établir la connectivité à votre instance managée SQL et à son réseau virtuel.

  > [!NOTE]
  > - Vous pouvez également utiliser une connexion ExpressRoute ou de site à site à partir de votre réseau local, mais ces approches sortent du cadre de ces guides de démarrage rapide.
  > - Si vous remplacez la valeur 0 de la période de rétention (rétention illimitée) par une autre valeur, notez que la rétention s’appliquera uniquement aux journaux écrits après la modification de la valeur de rétention (les journaux écrits au cours de la période pendant laquelle la rétention était définie sur illimité sont conservés, même après activation de la rétention).

En guise d’alternative à la création manuelle d’une instance managée SQL, vous pouvez utiliser [PowerShell](scripts/create-configure-managed-instance-powershell.md), [PowerShell avec le modèle Resource Manager](scripts/create-powershell-azure-resource-manager-template.md) ou [Azure CLI](/cli/azure/sql/mi#az-sql-mi-create) pour générer des scripts et automatiser ce processus.

### <a name="migrate-your-databases"></a>Migrer vos bases de données

Après avoir créé une instance managée SQL et configuré l’accès, vous pouvez commencer à migrer vos bases de données SQL Server. La migration peut échouer si la base de données source à migrer contient des fonctionnalités non prises en charge. Pour éviter les échecs et vérifier la compatibilité, vous pouvez utiliser l’[Assistant Migration de données](https://www.microsoft.com/download/details.aspx?id=53595) pour analyser vos bases de données sur SQL Server et rechercher les problèmes susceptibles de bloquer la migration vers une instance managée SQL comme l’existence de [FileStream](/sql/relational-databases/blob/filestream-sql-server) ou de plusieurs fichiers journaux. Si vous résolvez ces problèmes, vous pouvez migrer vos bases de données vers une instance managée SQL. L’[Assistant Expérimentation de base de données](/sql/dea/database-experimentation-assistant-overview) est un autre outil pratique qui peut enregistrer votre charge de travail sur SQL Server et la réexécuter sur une instance managée SQL, ce qui vous permet de déterminer si vous allez rencontrer des problèmes de performances en cas de migration vers une instance managée SQL.

Quand vous êtes sûr de pouvoir migrer votre base de données vers une instance managée SQL, vous pouvez utiliser les fonctionnalités de restauration natives de SQL Server pour restaurer une base de données sur une instance managée SQL à partir d’un fichier `.bak`. Vous pouvez utiliser cette méthode pour migrer des bases de données à partir du moteur de base de données SQL Server installé en local ou de machines virtuelles Azure. Pour accéder à un guide de démarrage rapide, consultez [Restaurer une sauvegarde sur une instance managée SQL](restore-sample-database-quickstart.md). Dans ce guide de démarrage rapide, vous effectuez une restauration à partir d’un fichier `.bak` situé dans le stockage Blob Azure à l’aide de la commande Transact-SQL `RESTORE`.

> [!TIP]
> Pour utiliser la commande Transact-SQL `BACKUP` afin de créer une sauvegarde de votre base de données dans le stockage Blob Azure, consultez [Sauvegarde SQL Server vers une URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url).

Ces guides de démarrage rapide vous permettent de créer, de configurer et de restaurer une sauvegarde de base de données sur une instance managée SQL. Dans certains scénarios, vous devez personnaliser ou automatiser le déploiement de SQL Managed Instance et de l’environnement réseau nécessaire. Ces scénarios sont décrits ci-dessous.

## <a name="customize-network-environment"></a>Personnaliser l’environnement réseau

Même si le réseau virtuel/sous-réseau peut être configuré automatiquement quand l’instance est [créée avec le portail Azure](instance-create-quickstart.md), il peut être judicieux de le créer avant de commencer à créer des instances dans SQL Managed Instance, car vous pouvez ainsi configurer les paramètres du réseau virtuel et du sous-réseau. Le moyen le plus simple pour créer et configurer l’environnement réseau consiste à utiliser le modèle de [déploiement Azure Resource Manager](virtual-network-subnet-create-arm-template.md), qui crée et configure votre réseau et votre sous-réseau où l’instance sera placée. Vous devez simplement cliquer sur le bouton de déploiement Azure Resource Manager et remplir le formulaire avec les paramètres.

Vous pouvez également utiliser ce [script PowerShell](https://www.powershellmagazine.com/2018/07/23/configuring-azure-environment-to-set-up-azure-sql-database-managed-instance-preview/) pour automatiser la création du réseau.

Si vous avez déjà un réseau virtuel et un sous-réseau où vous souhaitez déployer votre instance managée SQL, vous devez vérifier que votre réseau virtuel et votre sous-réseau répondent à la [configuration réseau demandée](connectivity-architecture-overview.md#network-requirements). Utilisez ce [script PowerShell pour vérifier que votre sous-réseau est correctement configuré](vnet-existing-add-subnet.md). Ce script vérifie votre réseau et signale les éventuels problèmes. Il indique ce qui doit être changé et vous propose d’effectuer les modifications nécessaires dans votre réseau virtuel/sous-réseau. Exécutez ce script si vous ne souhaitez pas configurer votre réseau virtuel/sous-réseau manuellement. Vous pouvez également l’exécuter après toute reconfiguration majeure de votre infrastructure réseau. Si vous souhaitez créer et configurer votre propre réseau, lisez l’article consacré à l’[architecture de la connectivité](connectivity-architecture-overview.md) et ce [guide complet pour créer et configurer un environnement SQL Managed Instance](https://medium.com/azure-sqldb-managed-instance/the-ultimate-guide-for-creating-and-configuring-azure-sql-managed-instance-environment-91ff58c0be01).

## <a name="migrate-to-a-sql-managed-instance"></a>Migrer vers une instance managée SQL

Les guides de démarrage rapide mentionnés plus haut vous permettent de configurer une instance managée SQL et de déplacer vos bases de données rapidement à l’aide de la fonctionnalité `RESTORE` native. Il s’agit d’un bon point de départ si vous voulez effectuer une preuve de concept pour vérifier que votre solution peut fonctionner sur Managed Instance.

Toutefois, pour migrer votre base de données de production ou même des bases de données de Dev/Test à utiliser pour effectuer des tests de performances, vous devez envisager d’utiliser des techniques supplémentaires, telles que :

- Test des performances : vous devez mesurer les métriques de performance de base sur votre instance SQL Server source et les comparer aux métriques de performance de l’instance managée SQL de destination où vous avez migré la base de données. Apprenez-en davantage sur les [meilleures pratiques en matière de comparaison des performances](https://techcommunity.microsoft.com/t5/Azure-SQL-Database/The-best-practices-for-performance-comparison-between-Azure-SQL/ba-p/683210).
- Migration en ligne : avec la commande `RESTORE` native décrite dans cet article, vous devez attendre que les bases de données soient restaurées (et copiées vers Stockage Blob Azure si elles ne s’y trouvent pas déjà). Il en résulte des temps d’arrêt pour votre application, en particulier si les bases de données sont volumineuses. Pour déplacer votre base de données de production, utilisez [Database Migration Service (DMS)](../../dms/tutorial-sql-server-to-managed-instance.md?toc=%2fazure%2fsql-database%2ftoc.json) afin de migrer votre base de données avec un temps d’arrêt minimal. Pour y parvenir, DMS envoie (push) les changements effectués dans votre base de données source à la base de données SQL Managed Instance en cours de restauration. De cette façon, vous pouvez basculer rapidement votre application de la base de données source vers la base de données cible avec un temps d’arrêt minimal.

Apprenez-en davantage sur le [processus de migration recommandé](migrate-to-instance-from-sql-server.md).

## <a name="next-steps"></a>Étapes suivantes

- Vous trouverez une [liste générale des fonctionnalités prises en charge dans SQL Managed Instance ici](../database/features-comparison.md) et des [informations détaillées et problèmes connus ici](transact-sql-tsql-differences-sql-server.md).
- Apprenez-en davantage sur les [caractéristiques techniques de SQL Managed Instance](resource-limits.md#service-tier-characteristics).
- Consultez l’article sur l’[utilisation d’une instance managée SQL](how-to-content-reference-guide.md).
- [Identifiez la bonne référence SKU d’Azure SQL Managed Instance pour votre base de données locale](/sql/dma/dma-sku-recommend-sql-db/).