---
title: 'Démarrage rapide : Utiliser Node.js pour se connecter à Azure Database pour PostgreSQL (serveur unique)'
description: Ce démarrage rapide fournit un exemple de code Node.js que vous pouvez utiliser pour vous connecter et interroger des données à partir d’Azure Database pour PostgreSQL (serveur unique).
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.custom:
- mvc
- devcenter
- seo-javascript-september2019
- seo-javascript-october2019
- devx-track-js
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 5/6/2019
ms.openlocfilehash: 7569606429740de23b56767d490b9bb14283d468
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93331690"
---
# <a name="quickstart-use-nodejs-to-connect-and-query-data-in-azure-database-for-postgresql---single-server"></a>Démarrage rapide : Utiliser Node.js afin de se connecter à Azure Database pour PostgreSQL et d’interroger les données - Serveur unique

Dans ce guide de démarrage rapide, vous vous connectez à Azure Database pour PostgreSQL en utilisant une application Node.js. Il détaille l’utilisation d’instructions SQL pour interroger la base de données, la mettre à jour, y insérer des données ou en supprimer. Cet article suppose que vous connaissez les bases du développement via Node.js, et que vous ne savez pas utiliser Azure Database pour PostgreSQL.

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

- Exécution du [Démarrage rapide : Créer un serveur Azure Database pour PostgreSQL dans le portail Azure](quickstart-create-server-database-portal.md) ou [Démarrage rapide : Créer une instance de Azure Database pour PostgreSQL à l’aide de Azure CLI](quickstart-create-server-database-azure-cli.md).

- [Node.JS](https://nodejs.org)

## <a name="install-pg-client"></a>Installer le client pg
Installez [pg](https://www.npmjs.com/package/pg), qui est un client PostgreSQL pour Node.js.

Pour ce faire, exécutez le gestionnaire de package de nœud (npm) pour JavaScript depuis la ligne de commande, afin d’installer le client pg.
```bash
npm install pg
```

Vérifiez l’installation en répertoriant les packages installés.
```bash
npm list
```

## <a name="get-connection-information"></a>Obtenir des informations de connexion
Obtenez les informations de connexion requises pour vous connecter à la base de données Azure pour PostgreSQL. Vous devez disposer du nom de serveur complet et des informations d’identification.

1. Dans le [portail Azure](https://portal.azure.com/), recherchez et sélectionnez le serveur que vous avez créé (par exemple, **mydemoserver** ).

1. Dans le panneau **Vue d’ensemble** du serveur, notez le **nom du serveur** et le **nom d’utilisateur de l’administrateur**. Si vous oubliez votre mot de passe, vous pouvez également le réinitialiser dans ce panneau.

   :::image type="content" source="./media/connect-nodejs/server-details-azure-database-postgresql.png" alt-text="Chaîne de connexion Azure Database pour PostgreSQL":::

## <a name="running-the-javascript-code-in-nodejs"></a>Exécution du code JavaScript dans Node.js
Vous pouvez lancer Node.js à partir de l’interpréteur de commandes Bash, du terminal ou de l’invite de commandes Windows, en saisissant `node`, puis exécuter l’exemple de code JavaScript de manière interactive, en le copiant et en le collant dans l’invite. Vous pouvez également enregistrer le code JavaScript dans un fichier texte, puis lancez `node filename.js` en indiquant le nom de fichier en tant que paramètre pour l’exécuter.

## <a name="connect-create-table-and-insert-data"></a>Se connecter, créer des tables et insérer des données
Utilisez le code suivant pour vous connecter et charger les données à l’aide d’instructions SQL **CREATE TABLE** et **INSERT INTO**.
L’objet [pg.Client](https://github.com/brianc/node-postgres/wiki/Client) est utilisé pour créer une interface avec le serveur PostgreSQL. La fonction [pg.Client.connect()](https://github.com/brianc/node-postgres/wiki/Client#method-connect) est utilisée pour établir la connexion avec le serveur. La fonction [pg.Client.query()](https://github.com/brianc/node-postgres/wiki/Query) est utilisée pour exécuter la requête SQL sur la base de données PostgreSQL. 

Remplacez les paramètres host, dbname, user et password par les valeurs spécifiées lors de la création du serveur et de la base de données.

```javascript
const pg = require('pg');

const config = {
    host: '<your-db-server-name>.postgres.database.azure.com',
    // Do not hard code your username and password.
    // Consider using Node environment variables.
    user: '<your-db-username>',     
    password: '<your-password>',
    database: '<name-of-database>',
    port: 5432,
    ssl: true
};

const client = new pg.Client(config);

client.connect(err => {
    if (err) throw err;
    else {
        queryDatabase();
    }
});

function queryDatabase() {
    const query = `
        DROP TABLE IF EXISTS inventory;
        CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);
        INSERT INTO inventory (name, quantity) VALUES ('banana', 150);
        INSERT INTO inventory (name, quantity) VALUES ('orange', 154);
        INSERT INTO inventory (name, quantity) VALUES ('apple', 100);
    `;

    client
        .query(query)
        .then(() => {
            console.log('Table created successfully!');
            client.end(console.log('Closed client connection'));
        })
        .catch(err => console.log(err))
        .then(() => {
            console.log('Finished execution, exiting now');
            process.exit();
        });
}
```

## <a name="read-data"></a>Lire les données
Utilisez le code suivant pour vous connecter et lire des données à l’aide d’une instruction SQL **SELECT**. L’objet [pg.Client](https://github.com/brianc/node-postgres/wiki/Client) est utilisé pour créer une interface avec le serveur PostgreSQL. La fonction [pg.Client.connect()](https://github.com/brianc/node-postgres/wiki/Client#method-connect) est utilisée pour établir la connexion avec le serveur. La fonction [pg.Client.query()](https://github.com/brianc/node-postgres/wiki/Query) est utilisée pour exécuter la requête SQL sur la base de données PostgreSQL. 

Remplacez les paramètres host, dbname, user et password par les valeurs spécifiées lors de la création du serveur et de la base de données. 

```javascript
const pg = require('pg');

const config = {
    host: '<your-db-server-name>.postgres.database.azure.com',
    // Do not hard code your username and password.
    // Consider using Node environment variables.
    user: '<your-db-username>',     
    password: '<your-password>',
    database: '<name-of-database>',
    port: 5432,
    ssl: true
};

const client = new pg.Client(config);

client.connect(err => {
    if (err) throw err;
    else { queryDatabase(); }
});

function queryDatabase() {
  
    console.log(`Running query to PostgreSQL server: ${config.host}`);

    const query = 'SELECT * FROM inventory;';

    client.query(query)
        .then(res => {
            const rows = res.rows;

            rows.map(row => {
                console.log(`Read: ${JSON.stringify(row)}`);
            });

            process.exit();
        })
        .catch(err => {
            console.log(err);
        });
}
```

## <a name="update-data"></a>Mettre à jour des données
Utilisez le code suivant pour vous connecter et lire des données à l’aide d’une instruction SQL **UPDATE**. L’objet [pg.Client](https://github.com/brianc/node-postgres/wiki/Client) est utilisé pour créer une interface avec le serveur PostgreSQL. La fonction [pg.Client.connect()](https://github.com/brianc/node-postgres/wiki/Client#method-connect) est utilisée pour établir la connexion avec le serveur. La fonction [pg.Client.query()](https://github.com/brianc/node-postgres/wiki/Query) est utilisée pour exécuter la requête SQL sur la base de données PostgreSQL. 

Remplacez les paramètres host, dbname, user et password par les valeurs spécifiées lors de la création du serveur et de la base de données. 

```javascript
const pg = require('pg');

const config = {
    host: '<your-db-server-name>.postgres.database.azure.com',
    // Do not hard code your username and password.
    // Consider using Node environment variables.
    user: '<your-db-username>',     
    password: '<your-password>',
    database: '<name-of-database>',
    port: 5432,
    ssl: true
};

const client = new pg.Client(config);

client.connect(err => {
    if (err) throw err;
    else {
        queryDatabase();
    }
});

function queryDatabase() {
    const query = `
        UPDATE inventory 
        SET quantity= 1000 WHERE name='banana';
    `;

    client
        .query(query)
        .then(result => {
            console.log('Update completed');
            console.log(`Rows affected: ${result.rowCount}`);
        })
        .catch(err => {
            console.log(err);
            throw err;
        });
}
```

## <a name="delete-data"></a>Suppression de données
Utilisez le code suivant pour vous connecter et lire des données à l’aide d’une instruction SQL **DELETE**. L’objet [pg.Client](https://github.com/brianc/node-postgres/wiki/Client) est utilisé pour créer une interface avec le serveur PostgreSQL. La fonction [pg.Client.connect()](https://github.com/brianc/node-postgres/wiki/Client#method-connect) est utilisée pour établir la connexion avec le serveur. La fonction [pg.Client.query()](https://github.com/brianc/node-postgres/wiki/Query) est utilisée pour exécuter la requête SQL sur la base de données PostgreSQL. 

Remplacez les paramètres host, dbname, user et password par les valeurs spécifiées lors de la création du serveur et de la base de données. 

```javascript
const pg = require('pg');

const config = {
    host: '<your-db-server-name>.postgres.database.azure.com',
    // Do not hard code your username and password.
    // Consider using Node environment variables.
    user: '<your-db-username>',     
    password: '<your-password>',
    database: '<name-of-database>',
    port: 5432,
    ssl: true
};

const client = new pg.Client(config);

client.connect(err => {
    if (err) {
        throw err;
    } else {
        queryDatabase();
    }
});

function queryDatabase() {
    const query = `
        DELETE FROM inventory 
        WHERE name = 'apple';
    `;

    client
        .query(query)
        .then(result => {
            console.log('Delete completed');
            console.log(`Rows affected: ${result.rowCount}`);
        })
        .catch(err => {
            console.log(err);
            throw err;
        });
}
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour nettoyer toutes les ressources utilisées dans le cadre de ce guide de démarrage rapide, supprimez le groupe de ressources à l’aide de la commande suivante :

```azurecli
az group delete \
    --name $AZ_RESOURCE_GROUP \
    --yes
```

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Migration de votre base de données PostgreSQL par exportation et importation](./howto-migrate-using-export-and-import.md)
