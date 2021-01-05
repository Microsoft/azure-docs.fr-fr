---
title: Se connecter à Azure SQL Edge et l’interroger
description: Découvrez comment vous connecter à Azure SQL Edge et l’interroger.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 07/25/2020
ms.openlocfilehash: b56b65261950e9cf534a3755d214229ef7d5bb1e
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93395204"
---
# <a name="connect-and-query-azure-sql-edge"></a>Se connecter à Azure SQL Edge et l’interroger

Après avoir déployé un conteneur Azure SQL Edge, vous pouvez vous connecter au moteur de base de données SQL à partir d'un des emplacements suivants :

- À l'intérieur du conteneur
- À partir d'un autre conteneur Docker exécuté sur le même hôte
- À partir de l'ordinateur hôte
- À partir de n'importe quel autre ordinateur client du réseau

## <a name="tools-to-connect-to-azure-sql-edge"></a>Outils permettant de se connecter à Azure SQL Edge

Vous pouvez vous connecter à une instance Azure SQL Edge à partir de l’un de ces outils courants :

* [sqlcmd](/sql/linux/sql-server-linux-setup-tools) : les outils clients sqlcmd sont déjà inclus dans l'image de conteneur d’Azure SQL Edge. Si vous joignez un conteneur en cours d’exécution avec un interpréteur de commandes Bash interactif, vous pouvez exécuter les outils localement. Les outils clients SQL ne sont PAS disponibles sur la plateforme ARM64, car ils ne sont pas inclus dans la version ARM64 des conteneurs SQL Edge. 
* [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms)
* [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio)
* [Visual Studio Code](/sql/visual-studio-code/sql-server-develop-use-vscode)

Pour vous connecter à un moteur de base de données Azure SQL Edge à partir d'un ordinateur du réseau, vous devez disposer des éléments suivants :

- **Adresse IP ou nom de réseau de la machine hôte** : Il s’agit de la machine hôte sur laquelle le conteneur Azure SQL Edge est en cours d’exécution.
- **Mappage du port hôte du conteneur Azure SQL Edge**  : Il s’agit du mappage pour le port du conteneur Docker sur un port de l’hôte. Dans le conteneur, Azure SQL Edge est toujours mappé au port 1433. Vous pouvez modifier cette valeur si vous le souhaitez. Pour modifier le numéro de port, mettez à jour les **Options de création de conteneur** pour le module Azure SQL Edge dans Azure IoT Edge. Dans l’exemple suivant, le port 1433 sur le conteneur est mappé au port 1600 sur l’hôte.

    ```JSON
    {
        "PortBindings": {
          "1433/tcp": [
            {
              "HostPort": "1600"
            }
          ]
        }
    }
    ```

- **Mot de passe AS pour l’instance Azure SQL Edge** : Il s’agit de la valeur spécifiée pour la variable d’environnement `SA_PASSWORD` lors du déploiement d’Azure SQL Edge.

## <a name="connect-to-the-database-engine-from-within-the-container"></a>Connexion au moteur de base de données depuis le conteneur

Les [outils en ligne de commande SQL Server](/sql/linux/sql-server-linux-setup-tools) sont inclus dans l'image de conteneur d'Azure SQL Edge. Si vous joignez une invite de commandes interactive au conteneur, vous pouvez exécuter les outils localement. Les outils clients SQL ne sont PAS disponibles sur la plateforme ARM64, car ils ne sont pas inclus dans la version ARM64 des conteneurs SQL Edge. 

1. Utilisez la commande `docker exec -it` pour démarrer un interpréteur de commandes bash interactif dans votre conteneur en cours d’exécution. Dans l’exemple suivant, `e69e056c702d` est l’ID de conteneur.

    ```bash
    docker exec -it <Azure SQL Edge container ID or name> /bin/bash
    ```

    > [!TIP]
    > Vous n’avez pas toujours besoin de spécifier l’ID de conteneur entier. Vous ne devez spécifier que suffisamment de caractères pour l’identifier de manière unique. Ainsi, dans cet exemple, il peut suffire d'utiliser `e6` ou `e69` plutôt que l’ID complet.

2. Une fois dans le conteneur, connectez-vous localement avec sqlcmd. Sqlcmd n’est pas dans le chemin par défaut, vous devez spécifier le chemin complet.

    ```bash
    /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P '<YourPassword>'
    ```

3. Lorsque vous avez terminé avec sqlcmd, tapez `exit`.

4. Lorsque vous avez terminé avec l’invite de commandes interactive, saisissez `exit`. Le conteneur continue de s’exécuter une fois que vous avez quitté l’interpréteur de commandes bash interactif.

## <a name="connect-to-azure-sql-edge-from-another-container-on-the-same-host"></a>Se connecter à Azure SQL Edge à partir d'un autre conteneur du même hôte

Comme deux conteneurs exécutés sur le même hôte se trouvent sur le même réseau Docker, ceux-ci sont facilement accessibles à l'aide du nom du conteneur et de l'adresse de port du service. Par exemple, si vous vous connectez à l'instance d’Azure SQL Edge à partir d'un autre module python (conteneur) du même hôte, vous pouvez utiliser une chaîne de connexion semblable à la suivante. (Cet exemple suppose qu’Azure SQL Edge est configuré pour écouter sur le port par défaut.)

```python

import pyodbc
server = 'MySQLEdgeContainer' # Replace this with the actual name of your SQL Edge Docker container
username = 'sa' # SQL Server username
password = 'MyStrongestP@ssword' # Replace this with the actual SA password from your deployment
database = 'MyEdgeDatabase' # Replace this with the actual database name from your deployment. If you do not have a database created, you can use Master database.
db_connection_string = "Driver={ODBC Driver 17 for SQL Server};Server=" + server + ";Database=" + database + ";UID=" + username + ";PWD=" + password + ";"
conn = pyodbc.connect(db_connection_string, autocommit=True)

```

## <a name="connect-to-azure-sql-edge-from-another-network-machine"></a>Se connecter à Azure SQL Edge depuis un autre ordinateur du réseau

Vous souhaiterez peut-être vous connecter à l’instance d’Azure SQL Edge à partir d’une autre machine sur le réseau. Pour ce faire, utilisez l’adresse IP de l’hôte Docker et le port hôte auquel le conteneur Azure SQL Edge est mappé. Par exemple, si l'adresse IP de l'hôte Docker est *xxx.xxx.xxx.xxx* et que le conteneur Azure SQL Edge est mappé au port *1600* de l'hôte, l'adresse serveur de l'instance de Azure SQL Edge sera *xxx.xxx.xxx.xxx,1600*. Le script Python mis à jour sera :

```python

import pyodbc
server = 'xxx.xxx.xxx.xxx,1600' # Replace this with the actual name of your SQL Edge Docker container
username = 'sa' # SQL Server username
password = 'MyStrongestP@ssword' # Replace this with the actual SA password from your deployment
database = 'MyEdgeDatabase' # Replace this with the actual database name from your deployment. If you do not have a database created, you can use Master database.
db_connection_string = "Driver={ODBC Driver 17 for SQL Server};Server=" + server + ";Database=" + database + ";UID=" + username + ";PWD=" + password + ";"
conn = pyodbc.connect(db_connection_string, autocommit=True)

```

Pour vous connecter à une instance d’Azure SQL Edge à l'aide de SQL Server Management Studio sur un ordinateur Windows, consultez [SQL Server Management Studio](/sql/linux/sql-server-linux-manage-ssms).

Pour vous connecter à une instance d’Azure SQL Edge à l'aide de Visual Studio Code sur un ordinateur Windows, Mac ou Linux, consultez [Visual Studio Code](/sql/visual-studio-code/sql-server-develop-use-vscode).

Pour vous connecter à une instance d’Azure SQL Edge à l'aide d'Azure Data Studio sur un ordinateur Windows, Mac ou Linux, consultez [Azure Data Studio](/sql/azure-data-studio/quickstart-sql-server).

## <a name="next-steps"></a>Étapes suivantes

[Se connecter et interroger](/sql/linux/sql-server-linux-configure-docker#connect-and-query)

[Installer les outils SQL Server sur Linux](/sql/linux/sql-server-linux-setup-tools)