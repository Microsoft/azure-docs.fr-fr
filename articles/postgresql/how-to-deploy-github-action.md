---
title: 'Démarrage rapide : Se connecter à Azure PostgreSQL avec GitHub Actions'
description: Utiliser Azure PostgreSQL à partir d’un workflow GitHub Actions
author: mksuni
ms.service: postgresql
ms.topic: quickstart
ms.author: sumuth
ms.date: 10/12/2020
ms.custom: github-actions-azure
ms.openlocfilehash: 2e546801f95d9d884bdfb3f09a18b3fa6e2d78a1
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97364980"
---
# <a name="quickstart-use-github-actions-to-connect-to-azure-postgresql"></a>Démarrage rapide : Utiliser GitHub Actions pour se connecter à Azure PostgreSQL

<Token>**S’APPLIQUE À :** :::image type="icon" source="./media/applies-to/yes.png" border="false":::Azure Database pour PostgreSQL - Serveur unique :::image type="icon" source="./media/applies-to/yes.png" border="false":::Azure Database pour PostgreSQL - Serveur flexible</Token>

Démarrez avec [GitHub Actions](https://docs.github.com/en/actions) en utilisant un workflow pour déployer des mises à jour de base de données sur [Azure Database pour PostgreSQL](https://azure.microsoft.com/services/postgresql/).

## <a name="prerequisites"></a>Prérequis

Éléments requis :
- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un dépôt GitHub avec des exemples de données (`data.sql`). Si vous n’avez pas de compte GitHub, [inscrivez-vous gratuitement](https://github.com/join).
- Un serveur Azure Database pour PostgreSQL.
    - [Démarrage rapide : Créer un serveur Azure Database pour PostgreSQL dans le portail Azure](quickstart-create-server-database-portal.md)

## <a name="workflow-file-overview"></a>Vue d’ensemble du fichier de workflow

Un workflow GitHub Actions est défini par un fichier YAML (.yml) dans le chemin `/.github/workflows/` de votre dépôt. Cette définition contient les étapes et les paramètres qui composent le workflow.

Le fichier comporte deux sections :

|Section  |Tâches  |
|---------|---------|
|**Authentification** | 1. Définissez un principal de service. <br /> 2. Créez un secret GitHub. |
|**Déployer** | 1. Déployez la base de données. |

## <a name="generate-deployment-credentials"></a>Générer les informations d’identification du déploiement

Vous pouvez créer un [principal de service](../active-directory/develop/app-objects-and-service-principals.md) à l’aide de la commande [az ad sp create-for-rbac](/cli/azure/ad/sp#az-ad-sp-create-for-rbac&preserve-view=true) dans [Azure CLI](/cli/azure/). Exécutez cette commande en utilisant [Azure Cloud Shell](https://shell.azure.com/) dans le portail Azure ou en sélectionnant le bouton **Essayer**.

Remplacez les espaces réservés `server-name` par le nom de votre serveur PostgreSQL hébergé sur Azure. Remplacez les valeurs `subscription-id` et `resource-group` par l’ID d’abonnement et le groupe de ressources connecté à votre serveur PostgreSQL.

```azurecli-interactive
   az ad sp create-for-rbac --name {server-name} --role contributor \
                            --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group} \
                            --sdk-auth
```

La sortie est un objet JSON dont les informations d’identification d’attribution de rôle qui permettent d’accéder à votre base de données sont semblables à celles fournies ci-dessous. Copiez cet objet JSON de sortie pour plus tard.

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

## <a name="copy-the-postgresql-connection-string"></a>Copier la chaîne de connexion PostgreSQL

Dans le portail Azure, accédez à votre serveur Azure Database pour PostgreSQL, puis ouvrez **Paramètres** > **Chaînes de connexion**. Copiez la chaîne de connexion **ADO.NET**. Remplacez les valeurs d’espace réservé spécifiées pour `your_database` et `your_password`. La chaîne de connexion ressemblera à ceci :

> [!IMPORTANT]
> - Pour le serveur unique, utilisez ```user=adminusername@servername```. Notez que ```@servername``` est obligatoire.
> - Pour le serveur flexible, utilisez ```user= adminusername``` sans ```@servername```.

```output
psql host={servername.postgres.database.azure.com} port=5432 dbname={your_database} user={adminusername} password={your_database_password} sslmode=require
```

Vous utiliserez cette chaîne de connexion comme secret GitHub.

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

1. Collez la valeur de la chaîne de connexion dans le champ de valeur du secret. Nommez le secret `AZURE_POSTGRESQL_CONNECTION_STRING`.


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

1. Renommez votre workflow `PostgreSQL for GitHub Actions` et ajoutez les actions d’extraction et de connexion. Ces actions extraient votre code de site et vous authentifient auprès d’Azure à l’aide du secret GitHub `AZURE_CREDENTIALS` que vous avez créé précédemment.

    ```yaml
    name: PostgreSQL for GitHub Actions

    on:
    push:
        branches: [ master ]
    pull_request:
        branches: [ master ]

    jobs:
    build:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v1
        - uses: azure/login@v1
        with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}
    ```

2. Utilisez l’action Déployer Azure PostgreSQL pour vous connecter à votre instance PostgreSQL. Remplacez `POSTGRESQL_SERVER_NAME` par le nom de votre serveur. Vous devez disposer d’un fichier de données PostgreSQL nommé `data.sql` au niveau racine de votre dépôt.

    ```yaml
     - uses: azure/postgresql@v1
       with:
        connection-string: ${{ secrets.AZURE_POSTGRESQL_CONNECTION_STRING }}
        server-name: POSTGRESQL_SERVER_NAME
        sql-file: './data.sql'
    ```

3. Terminez votre workflow en ajoutant une action permettant de vous déconnecter d’Azure. Voici le workflow terminé. Le fichier apparaît dans le dossier `.github/workflows` de votre référentiel.

    ```yaml
   name: PostgreSQL for GitHub Actions

    on:
    push:
        branches: [ master ]
    pull_request:
        branches: [ master ]


    jobs:
    build:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v1
        - uses: azure/login@v1
        with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}

    - uses: azure/postgresql@v1
      with:
        server-name: POSTGRESQL_SERVER_NAME
        connection-string: ${{ secrets.AZURE_POSTGRESQL_CONNECTION_STRING }}
        sql-file: './data.sql'

        # Azure logout
    - name: logout
      run: |
         az logout
    ```

## <a name="review-your-deployment"></a>Vérifier votre déploiement

1. Accédez à **Actions** pour votre référentiel GitHub.

1. Ouvrez le premier résultat pour afficher les journaux détaillés de l’exécution de votre workflow.

    :::image type="content" source="media/how-to-deploy-github-action/gitbub-action-postgres-success.png" alt-text="Journal de l’exécution GitHub Actions":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand votre dépôt et votre base de données Azure PostgreSQL ne sont plus nécessaires, nettoyez les ressources que vous avez déployées en supprimant le groupe de ressources et votre dépôt GitHub.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur l’intégration d’Azure et de GitHub](/azure/developer/github/)
<br/>
> [!div class="nextstepaction"]
> [Savoir comment se connecter au serveur](how-to-connect-query-guide.md)
