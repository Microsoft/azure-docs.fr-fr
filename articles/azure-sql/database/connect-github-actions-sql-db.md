---
title: 'Démarrage rapide : Se connecter à Azure SQL Database avec GitHub Actions'
description: Utiliser Azure SQL à partir d’un workflow GitHub Actions
author: juliakm
services: sql-database
ms.service: sql-database
ms.topic: quickstart
ms.author: jukullam
ms.date: 10/12/2020
ms.custom: github-actions-azure
ms.openlocfilehash: 216658b5f5443409e7bd44cbd29bff40cd56c75f
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97606978"
---
# <a name="use-github-actions-to-connect-to-azure-sql-database"></a>Utiliser GitHub Actions pour se connecter à Azure SQL Database

Familiarisez-vous avec [GitHub Actions](https://docs.github.com/en/free-pro-team@latest/actions) en utilisant un workflow pour déployer des mises à jour de base de données sur [Azure SQL Database](../azure-sql-iaas-vs-paas-what-is-overview.md). 

## <a name="prerequisites"></a>Prérequis

Éléments requis : 
- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un dépôt GitHub avec un package dacpac (`Database.dacpac`). Si vous n’avez pas de compte GitHub, [inscrivez-vous gratuitement](https://github.com/join).  
- Une base de données Azure SQL.
    - [Démarrage rapide : Créer une base de données Azure SQL](single-database-create-quickstart.md)
    - [Guide pratique pour créer un package dacpac à partir de votre base de données SQL Server existante](/sql/relational-databases/data-tier-applications/export-a-data-tier-application)

## <a name="workflow-file-overview"></a>Vue d’ensemble du fichier de workflow

Un workflow GitHub Actions est défini par un fichier YAML (.yml) dans le chemin `/.github/workflows/` de votre dépôt. Cette définition contient les étapes et les paramètres qui composent le workflow.

Le fichier comporte deux sections :

|Section  |Tâches  |
|---------|---------|
|**Authentification** | 1. Définissez un principal de service. <br /> 2. Créez un secret GitHub. |
|**Déployer** | 1. Déployez la base de données. |

## <a name="generate-deployment-credentials"></a>Générer les informations d’identification du déploiement

Vous pouvez créer un [principal de service](../../active-directory/develop/app-objects-and-service-principals.md) à l’aide de la commande [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac&preserve-view=true) dans [Azure CLI](/cli/azure/). Exécutez cette commande en utilisant [Azure Cloud Shell](https://shell.azure.com/) dans le portail Azure ou en sélectionnant le bouton **Essayer**.

Remplacez les espaces réservés `server-name` par le nom de votre serveur SQL hébergé sur Azure. Remplacez les valeurs `subscription-id` et `resource-group` par l’ID d’abonnement et le groupe de ressources connectés à votre serveur SQL.  

```azurecli-interactive
   az ad sp create-for-rbac --name {server-name} --role contributor \
                            --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group} \
                            --sdk-auth
```

La sortie est un objet JSON dont les informations d’identification d’attribution de rôle permettent d’accéder à votre base de données semblable à l’exemple suivant. Copiez votre objet JSON de sortie pour plus tard.

```output 
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```

> [!IMPORTANT]
> Il est toujours conseillé d’accorder un accès minimal. Dans l’exemple précédent, l’étendue est limitée au serveur spécifique, et non à l’ensemble du groupe de ressources.

## <a name="copy-the-sql-connection-string"></a>Copier la chaîne de connexion SQL 

Dans le portail Azure, accédez à votre base de données Azure SQL Database, puis ouvrez **Paramètres** > **Chaînes de connexion**. Copiez la chaîne de connexion **ADO.NET**. Remplacez les valeurs d’espace réservé spécifiées pour `your_database` et `your_password`. La chaîne de connexion ressemblera à cette sortie. 

```output
    Server=tcp:my-sql-server.database.windows.net,1433;Initial Catalog={your-database};Persist Security Info=False;User ID={admin-name};Password={your-password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```

Vous utiliserez la chaîne de connexion comme secret GitHub. 

## <a name="configure-the-github-secrets"></a>Configurer les secrets GitHub

1. Dans [GitHub](https://github.com/), accédez à votre référentiel.

1. Sélectionnez **Paramètres > Secrets > Nouveau secret**.

1. Collez l’intégralité de la sortie JSON de la commande Azure CLI dans le champ de valeur du secret. Nommez le secret `AZURE_CREDENTIALS`.

    Quand vous configurez le fichier de flux de travail ultérieurement, vous utilisez le secret pour l’entrée `creds` de l’action de connexion Azure. Par exemple :

    ```yaml
    - uses: azure/login@v1
    with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
   ```

1. Sélectionnez de nouveau **Nouveau secret**. 

1. Collez la valeur de la chaîne de connexion dans le champ de valeur du secret. Nommez le secret `AZURE_SQL_CONNECTION_STRING`.


## <a name="add-your-workflow"></a>Ajouter votre workflow

1. Accédez à **Actions** pour votre référentiel GitHub. 

2. Sélectionnez **Configurer vous-même un workflow**. 

2. Supprimez tous les éléments après la section `on:` de votre fichier de workflow. Par exemple, votre workflow restant peut ressembler à ce qui suit. 

    ```yaml
    name: CI

    on:
    push:
        branches: [ master ]
    pull_request:
        branches: [ master ]
    ```

1. Renommez votre workflow `SQL for GitHub Actions` et ajoutez les actions d’extraction et de connexion. Ces actions extraient votre code de site et vous authentifient auprès d’Azure à l’aide du secret GitHub `AZURE_CREDENTIALS` que vous avez créé précédemment. 

    ```yaml
    name: SQL for GitHub Actions

    on:
    push:
        branches: [ master ]
    pull_request:
        branches: [ master ]

    jobs:
    build:
        runs-on: windows-latest
        steps:
        - uses: actions/checkout@v1
        - uses: azure/login@v1
        with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}
    ```

1. Utilisez l’action Déployer d’Azure SQL pour vous connecter à votre instance SQL. Remplacez `SQL_SERVER_NAME` par le nom de votre serveur. Vous devez disposer d’un package dacpac (`Database.dacpac`) au niveau racine de votre dépôt. 

    ```yaml
    - uses: azure/sql-action@v1
      with:
        server-name: SQL_SERVER_NAME
        connection-string: ${{ secrets.AZURE_SQL_CONNECTION_STRING }}
        sql-file: './Database.dacpac'
    ``` 

1. Terminez votre workflow en ajoutant une action permettant de vous déconnecter d’Azure. Voici le workflow terminé. Le fichier apparaît dans le dossier `.github/workflows` de votre référentiel.

    ```yaml
   name: SQL for GitHub Actions

    on:
    push:
        branches: [ master ]
    pull_request:
        branches: [ master ]


    jobs:
    build:
        runs-on: windows-latest
        steps:
        - uses: actions/checkout@v1
        - uses: azure/login@v1
        with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}

    - uses: azure/sql-action@v1
      with:
        server-name: SQL_SERVER_NAME
        connection-string: ${{ secrets.AZURE_SQL_CONNECTION_STRING }}
        sql-file: './Database.dacpac'

        # Azure logout 
    - name: logout
      run: |
         az logout
    ```

## <a name="review-your-deployment"></a>Vérifier votre déploiement

1. Accédez à **Actions** pour votre référentiel GitHub. 

1. Ouvrez le premier résultat pour afficher les journaux détaillés de l’exécution de votre workflow. 
 
   :::image type="content" source="media/quickstart-sql-github-actions/github-actions-run-sql.png" alt-text="Journal de l’exécution GitHub Actions":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand votre dépôt et votre base de données Azure SQL ne sont plus nécessaires, nettoyez les ressources que vous avez déployées en supprimant le groupe de ressources et votre dépôt GitHub. 

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur l’intégration d’Azure et de GitHub](/azure/developer/github/)