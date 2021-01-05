---
title: Connexion et requête - Serveur unique MySQL
description: Liens vers des démarrages rapides montrant comment se connecter à votre serveur unique Azure SQL Database et exécuter des requêtes.
services: mysql
ms.service: mysql
ms.topic: how-to
author: mksuni
ms.author: sumuth
ms.date: 09/22/2020
ms.openlocfilehash: 0414aaad5a1cf6edb9c2152eed70f8753946cca1
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92546430"
---
# <a name="connect-and-query-overview-for-azure-database-for-mysql--single-server"></a>Vue d’ensemble de la connexion et des requêtes pour Azure Database pour MySQL - Serveur unique

Le document suivant comprend des liens vers des exemples montrant comment se connecter et exécuter des requêtes avec un serveur unique Azure Database pour MySQL. Ce guide comprend également des bibliothèques et des recommandations TLS que vous pouvez utiliser pour vous connecter au serveur dans les langages pris en charge ci-dessous.

## <a name="quickstarts"></a>Démarrages rapides

| Démarrage rapide | Description |
|---|---|
|[MySQL Workbench](connect-workbench.md)|Ce guide de démarrage rapide montre comment utiliser MySQL Workbench Client pour se connecter à une base de données. Vous pouvez alors utiliser des instructions MySQL pour interroger la base de données, la mettre à jour, y insérer des données ou en supprimer.|
|[Azure Cloud Shell](./quickstart-create-mysql-server-database-using-azure-cli.md#connect-to-azure-database-for-mysql-server-using-mysql-command-line-client)|Cet article montre comment exécuter **mysql.exe** dans [Azure Cloud Shell](../cloud-shell/overview.md) pour vous connecter à votre serveur, puis exécuter des instructions afin d’interroger la base de données, la mettre à jour, y insérer des données et en supprimer.|
|[MySQL avec Visual Studio](https://www.mysql.com/why-mysql/windows/visualstudio)|Vous pouvez utiliser MySQL pour Visual Studio pour vous connecter à votre serveur MySQL. MySQL pour Visual Studio s’intègre directement dans l’Explorateur de serveurs, ce qui facilite la configuration de nouvelles connexions et l’utilisation des objets de base de données.|
|[PHP](connect-php.md)|Ce guide de démarrage rapide explique comment utiliser PHP pour créer un programme en vue de se connecter à une base de données et l’interroger à l’aide d’instructions MySQL.|
|[Java](connect-java.md)|Ce guide de démarrage rapide explique comment utiliser Java pour se connecter à une base de données et comment l’interroger à l’aide d’instructions MySQL.|
|[Node.JS](connect-nodejs.md)|Ce guide de démarrage rapide explique comment utiliser Node.js pour créer un programme en vue de se connecter à une base de données et l’interroger à l’aide d’instructions MySQL.|
|[.NET(C#)](connect-csharp.md)|Ce guide de démarrage rapide explique comment utiliser .NET (#) pour créer un programme C# en vue de se connecter à une base de données et l’interroger à l’aide d’instructions MySQL.|
|[Go](connect-go.md)|Ce guide de démarrage rapide montre comment utiliser Go pour se connecter à une base de données. Les instructions Transact-SQL permettant d’interroger et de modifier des données sont également présentées.|
|[Python](connect-python.md)|Ce guide de démarrage rapide explique comment utiliser Python pour se connecter à une base de données et l’interroger à l’aide d’instructions MySQL. |
|[Ruby](connect-ruby.md)|Ce guide de démarrage rapide explique comment utiliser Ruby pour créer un programme en vue de se connecter à une base de données et l’interroger à l’aide d’instructions MySQL.|
|[C++](connect-cpp.md)|Ce guide de démarrage rapide explique comment utiliser C++ pour créer un programme en vue de se connecter à une base de données et l’interroger à l’aide d’instructions MySQL.|

## <a name="tls-considerations-for-database-connectivity"></a>Considérations relatives au protocole TLS pour la connectivité des bases de données

Le protocole TLS (Transport Layer Security) est utilisé par tous les pilotes fournis et pris en charge par Microsoft pour la connexion à des bases de données dans Azure Database pour MySQL. Aucune configuration spéciale n’est nécessaire, mais veillez à mettre en œuvre le protocole TLS 1.2 pour les serveurs récemment créés. Si vous utilisez les protocoles TLS 1.0 et 1.1, nous vous recommandons de mettre à jour ces versions sur vos serveurs. Voir [Comment configurer le protocole TLS](howto-tls-configurations.md)

## <a name="libraries"></a>Bibliothèques

En termes de base de données MySQL, Azure Database pour MySQL utilise la version community edition la plus populaire au monde. Ainsi, elle est compatible avec un large éventail de langages de programmation et de pilotes. L’objectif est de prendre en charge les trois versions les plus récentes de pilotes MySQL, et de poursuivre les efforts avec les auteurs de la Communauté open source, afin d’améliorer en permanence les fonctionnalités et la facilité d’utilisation des pilotes MySQL.

Découvrez les [pilotes](concepts-compatibility.md) compatibles avec un serveur unique Azure Database pour MySQL.

## <a name="next-steps"></a>Étapes suivantes

- [Migrer des données en utilisant l’image mémoire et la restauration](concepts-migrate-dump-restore.md)
- [Migrer des données par importation et exportation](concepts-migrate-import-export.md)