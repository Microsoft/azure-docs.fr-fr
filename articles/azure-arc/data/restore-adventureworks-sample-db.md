---
title: Restaurer l’exemple de base de données AdventureWorks dans SQL Managed Instance
description: Restaurer l’exemple de base de données AdventureWorks dans SQL Managed Instance
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: d238a889648e789087a0803f6b50288318462c7b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91629061"
---
# <a name="restore-the-adventureworks-sample-database-into-sql-managed-instance---azure-arc"></a>Restaurer l’exemple de base de données AdventureWorks dans SQL Managed Instance – Azure Arc

[AdventureWorks](/sql/samples/adventureworks-install-configure?view=sql-server-ver15&tabs=tsql&preserve-view=true) est un exemple de base de données contenant une base de données OLTP souvent utilisée dans des tutoriels et pour des exemples. Il est fourni et géré par Microsoft dans le [Dépôt GitHub d’échantillons SQL Server](https://github.com/microsoft/sql-server-samples/tree/master/samples/databases).

Ce document décrit un processus simple pour récupérer l’exemple de base de données AdventureWorks dans votre SQL Managed Instance – Azure Arc.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="download-the-adventureworks-backup-file"></a>Télécharger le fichier de sauvegarde AdventureWorks

Téléchargez le fichier de sauvegarde (.bak) AdventureWorks dans votre conteneur SQL Managed Instance. Dans cet exemple, utilisez la commande `kubectl exec` pour exécuter à distance une commande à l’intérieur du conteneur SQL Managed Instance afin de télécharger le fichier. bak dans le conteneur. Téléchargez ce fichier à partir de n’importe quel emplacement accessible par `wget` si vous avez d’autres fichiers de sauvegarde de base de données que vous souhaitez extraire dans le conteneur SQL Managed Instance. Une fois le fichier dans le conteneur SQL Managed Instance, il est facile de le restaurer à l’aide de la commande T-SQL `RESTORE DATABASE` standard.

Exécutez une commande comme celle-ci pour télécharger le fichier. bak en remplaçant la valeur du nom de pod et du nom d’espace de noms avant de l’exécuter.
> [!NOTE]
>  Pour pouvoir télécharger le fichier à partir de GitHub, votre conteneur doit disposer d’une connexion Internet sur le port 443

```console
kubectl exec <SQL pod name> -n <namespace name> -c arc-sqlmi -- wget https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2019.bak -O /var/opt/mssql/data/AdventureWorks2019.bak
```

Exemple

```console
kubectl exec sqltest1-0 -n arc -c arc-sqlmi -- wget https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2019.bak -O /var/opt/mssql/data/AdventureWorks2019.bak
```

## <a name="restore-the-adventureworks-database"></a>Restaurer la base de données AdventureWorks

De même, vous pouvez exécuter une commande exec `kubectl` pour utiliser l’outil d’interface de ligne de commande `sqlcmd` inclus dans le conteneur SQL Managed Instance afin d’exécuter la commande T-SQL RESTORE DATABASE.

Exécutez une commande comme celle-ci pour restaurer la base de données. Remplacez la valeur du nom de pod, le mot de passe et du nom d’espace de noms avant de l’exécuter.

```console
kubectl exec <SQL pod name> -n <namespace name> -c arc-sqlmi -- /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P <password> -Q "RESTORE DATABASE AdventureWorks2019 FROM  DISK = N'/var/opt/mssql/data/AdventureWorks2019.bak' WITH MOVE 'AdventureWorks2017' TO '/var/opt/mssql/data/AdventureWorks2019.mdf', MOVE 'AdventureWorks2017_Log' TO '/var/opt/mssql/data/AdventureWorks2019_Log.ldf'"
```
Exemple

```console
kubectl exec sqltest1-0 -n arc -- -c arc-sqlmi /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P MyPassword! -Q "RESTORE DATABASE AdventureWorks2019 FROM  DISK = N'/var/opt/mssql/data/AdventureWorks2019.bak' WITH MOVE 'AdventureWorks2017' TO '/var/opt/mssql/data/AdventureWorks2019.mdf', MOVE 'AdventureWorks2017_Log' TO '/var/opt/mssql/data/AdventureWorks2019_Log.ldf'"
```
