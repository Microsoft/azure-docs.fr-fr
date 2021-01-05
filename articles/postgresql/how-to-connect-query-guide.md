---
title: Se connecter et interroger - Serveur unique PostgreSQL
description: Liens vers des démarrages rapides montrant comment se connecter à votre serveur unique Azure Database pour PostgreSQL et exécuter des requêtes.
services: postgresql
ms.service: postgresql
ms.topic: how-to
author: mksuni
ms.author: sumuth
ms.date: 09/21/2020
ms.openlocfilehash: 413f5fc1f6579102b62042c1470816470c2e1449
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92546549"
---
# <a name="connect-and-query-overview-for-azure-database-for-postgresql--single-server"></a>Vue d’ensemble de la connexion et des requêtes pour Azure Database pour PostgreSQL - Serveur unique

Le document suivant comprend des liens vers des exemples montrant comment se connecter et exécuter des requêtes avec un serveur unique Azure Database pour PostgreSQL. Ce guide comprend également une extension et des recommandations TLS que vous pouvez utiliser pour vous connecter au serveur dans les langages pris en charge ci-dessous.

## <a name="quickstarts"></a>Démarrages rapides

| Démarrage rapide | Description |
|---|---|
|[Pgadmin](https://www.pgadmin.org/)|Vous pouvez utiliser pgadmin pour vous connecter au serveur et simplifier la création, la maintenance et l’utilisation des objets de base de données.|
|[psql dans Azure Cloud Shell](quickstart-create-server-database-azure-cli.md#connect-to-the-azure-database-for-postgresql-server-by-using-psql)|Cet article montre comment exécuter [**psql**](https://www.postgresql.org/docs/current/static/app-psql.html) dans [Azure Cloud Shell](../cloud-shell/overview.md) pour vous connecter à votre serveur, puis exécuter des instructions afin d’interroger la base de données, la mettre à jour, y insérer des données et en supprimer. Vous pouvez exécuter **psql** si elle est installée dans votre environnement de développement|
|[PostgreSQL avec VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)|L’extension Azure Databases pour VS Code (préversion) vous permet de parcourir et d’interroger votre serveur PostgreSQL localement et dans le cloud en utilisant des albums avec des fonctionnalités IntelliSense complètes. |
|[PHP](connect-php.md)|Ce guide de démarrage rapide explique comment utiliser PHP pour créer un programme en vue de se connecter à une base de données et utiliser des objets de base de données pour interroger des données.|
|[Java](connect-java.md)|Ce guide de démarrage rapide explique comment utiliser Java pour se connecter à une base de données, puis utiliser des objets de base de données pour interroger des données.|
|[Node.JS](connect-nodejs.md)|Ce guide de démarrage rapide explique comment utiliser Node.js pour créer un programme en vue de se connecter à une base de données et utiliser des objets de base de données pour interroger des données.|
|[.NET (C#)](connect-csharp.md)|Ce guide de démarrage rapide explique comment utiliser .NET (C#) pour créer un programme C# en vue de se connecter à une base de données et utiliser des objets de base de données pour interroger des données.|
|[Go](connect-go.md)|Ce guide de démarrage rapide montre comment utiliser Go pour se connecter à une base de données. Les instructions Transact-SQL permettant d’interroger et de modifier des données sont également présentées.|
|[Python](connect-python.md)|Ce guide de démarrage rapide explique comment utiliser Python pour se connecter à une base de données et utiliser des objets de base de données pour interroger des données. |
|[Ruby](connect-ruby.md)|Ce guide de démarrage rapide explique comment utiliser Ruby pour créer un programme en vue de se connecter à une base de données et utiliser des objets de base de données pour interroger des données.|

## <a name="tls-considerations-for-database-connectivity"></a>Considérations relatives au protocole TLS pour la connectivité des bases de données

Le protocole TLS (Transport Layer Security) est utilisé par tous les pilotes fournis ou pris en charge par Microsoft pour la connexion à des bases de données dans Azure Database pour PostgreSQL. Aucune configuration spéciale n’est nécessaire, mais veillez à mettre en œuvre le protocole TLS 1.2 pour les serveurs récemment créés. Si vous utilisez les protocoles TLS 1.0 et 1.1, nous vous recommandons de mettre à jour ces versions sur vos serveurs. Voir [Comment configurer le protocole TLS](howto-tls-configurations.md)

## <a name="postgresql-extensions"></a>Extensions PostgreSQL

PostgreSQL offre la possibilité d’étendre les fonctionnalités d’une base de données à l’aide des extensions. Les extensions regroupent plusieurs objets SQL associés au sein d’un package qui peut être chargé ou supprimé de votre base de données à l’aide d’une seule commande. Une fois chargées dans la base de données, les extensions fonctionnent comme des fonctionnalités intégrées.

- [Extensions Postgres 11](./concepts-extensions.md#postgres-11-extensions)
- [Extensions Postgres 10](./concepts-extensions.md#postgres-10-extensions)
- [Extensions Postgres 9.6](./concepts-extensions.md#postgres-96-extensions)
- [Extensions Postgres 9.5](./concepts-extensions.md#postgres-95-extensions)

Pour plus d’informations, consultez [Guide pratique pour utiliser des extensions PostgreSQL sur un serveur unique](concepts-extensions.md).

## <a name="next-steps"></a>Étapes suivantes

- [Migrer des données en utilisant l’image mémoire et la restauration](howto-migrate-using-dump-and-restore.md)
- [Migrer des données par importation et exportation](howto-migrate-using-export-and-import.md)
