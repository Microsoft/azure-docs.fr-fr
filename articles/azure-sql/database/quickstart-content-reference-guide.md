---
title: Informations de référence des guides de démarrage rapide sur les bases de données uniques
description: Consultez les informations de référence de tous les guides de démarrage rapide qui vous aident à démarrer vite et bien avec les bases de données uniques dans Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: guide
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 07/29/2019
ms.openlocfilehash: 392f361f6fe487a15bdd63c2f3281ccc78b2f86d
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184414"
---
# <a name="getting-started-with-single-databases-in-azure-sql-database"></a>Bien démarrer avec des bases de données uniques dans Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

[Une base de données unique](../index.yml) est une base de données DbaaS (base de données en tant que service) PaaS (plateforme en tant que service) complètement managée, qui constitue le moteur de stockage idéal pour les applications cloud modernes. Dans cette section, vous allez apprendre à configurer et créer rapidement une base de données unique dans Azure SQL Database.

## <a name="quickstart-overview"></a>Vue d’ensemble du guide de démarrage rapide

Cette section contient une vue d’ensemble des articles disponibles qui peuvent vous aider à démarrer rapidement avec les bases de données uniques. Les guides de démarrage rapide suivants vous permettent de créer une base de données unique, de configurer une règle de pare-feu au niveau du serveur, puis d’importer une base de données dans la nouvelle base de données unique au moyen d’un fichier `.bacpac` :

- [Créez une base de données unique à l’aide du portail Azure](single-database-create-quickstart.md).
- Après avoir créé la base de données, vous devez [la sécuriser en configurant des règles de pare-feu](firewall-create-server-level-portal-quickstart.md).
- Si vous souhaitez migrer une base de données existante de SQL Server vers Azure SQL Database, vous devez installer l’[Assistant Migration de données](https://www.microsoft.com/download/details.aspx?id=53595), qui analyse vos bases de données sur SQL Server et détecte les problèmes susceptibles de bloquer la migration. Si vous ne trouvez pas de problème, vous pouvez exporter votre base de données en tant que fichier `.bacpac` et [l’importer en utilisant le portail Azure ou SqlPackage](database-import.md).


## <a name="automating-management-operations"></a>Automatisation des opérations de gestion

Vous pouvez utiliser PowerShell ou Azure CLI pour créer, configurer et mettre à l’échelle votre base de données.

- [Créer et configurer une base de données unique à l’aide de PowerShell](scripts/create-and-configure-database-powershell.md) ou d’[Azure CLI](scripts/create-and-configure-database-cli.md)
- [Mettre à jour votre base de données unique et mettre à l’échelle vos ressources avec PowerShell](scripts/monitor-and-scale-database-powershell.md) ou [Azure CLI](scripts/monitor-and-scale-database-cli.md)

## <a name="migrating-to-a-single-database-with-minimal-downtime"></a>Migration vers une base de données unique avec un temps d’arrêt minimal

Ces guides de démarrage rapide vous permettent de créer ou d’importer votre base de données dans Azure avec un fichier `.bacpac`. Cependant, les fichiers `.bacpac` et `.dacpac` sont conçus pour déplacer rapidement des bases de données entre des versions différentes de SQL Server et au sein d’Azure SQL, ou pour implémenter l’intégration continue dans votre pipeline DevOps. Toutefois, cette méthode n’est pas conçue pour la migration de vos bases de données de production avec un temps d’arrêt minimal dans la mesure où vous devez cesser d’ajouter de nouvelles données, attendre la fin de l’exportation de la base de données source vers un fichier `.bacpac`, puis attendre la fin de l’importation dans Azure SQL Database. Toute cette attente entraîne des temps d’arrêt pour votre application, en particulier si les bases de données sont volumineuses. Pour déplacer votre base de données de production, vous devez trouver une meilleure méthode de migration garantissant un temps d’arrêt minimal. Pour cela, utilisez [Data Migration Service (DMS)](../../dms/tutorial-sql-server-to-azure-sql.md?toc=%2fazure%2fsql-database%2ftoc.json) pour migrer votre base de données avec un temps d’arrêt minimal. Pour y parvenir, DMS envoie (push) les changements effectués dans votre base de données source à la base de données unique en cours de restauration. De cette façon, vous pouvez basculer rapidement votre application de la base de données source vers la base de données cible avec un temps d’arrêt minimal.

## <a name="hands-on-learning-modules"></a>Modules d’apprentissage pratique

Les modules Microsoft Learn suivants vous aident à vous familiariser gratuitement avec Azure SQL Database.

- [Provisionner une base de données dans SQL Database pour stocker les données d’application](/learn/modules/provision-azure-sql-db/)
- [Développer et configurer une application ASP.NET qui interroge une base de données dans Azure SQL Database](/learn/modules/develop-app-that-queries-azure-sql/)
- [Sécuriser votre base de données dans Azure SQL Database](/learn/modules/secure-your-azure-sql-database/)

## <a name="next-steps"></a>Étapes suivantes

- Découvrez une [liste générale des fonctionnalités prises en charge dans Azure SQL Database](features-comparison.md).
- Découvrez comment [renforcer la sécurité de votre base de données](secure-database-tutorial.md).
- Consultez l’article sur l’[utilisation d’une base de données unique dans Azure SQL Database](how-to-content-reference-guide.md) pour accéder à des guides pratiques plus avancés.
- Découvrez plus d’exemples de scripts écrits dans [PowerShell](powershell-script-content-guide.md) et dans [l’interface Azure CLI](az-cli-script-samples-content-guide.md).
- Découvrez plus d’informations sur l’[API de gestion](single-database-manage.md) que vous pouvez utiliser pour configurer vos bases de données.
- [Identifiez la bonne référence SKU Azure SQL Database ou Azure SQL Managed Instance pour votre base de données locale](/sql/dma/dma-sku-recommend-sql-db/).