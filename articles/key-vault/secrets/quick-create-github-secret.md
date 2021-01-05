---
title: Démarrage rapide - Utiliser des secrets Azure Key Vault dans des workflows GitHub Actions
description: Utiliser des secrets Key Vault dans GitHub Actions pour automatiser vos workflows de développement logiciel
author: juliakm
ms.custom: github-actions-azure
ms.author: jukullam
ms.date: 11/24/2020
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.openlocfilehash: 9509f84b14a42180189a529282b5db348deab279
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95920230"
---
# <a name="use-key-vault-secrets-in-github-actions-workflows"></a>Utiliser des secrets Key Vault dans des workflows GitHub Actions

Utilisez des secrets Key Vault dans [GitHub Actions](https://help.github.com/en/articles/about-github-actions) et stockez de manière sécurisée des mots de passe et autres secrets dans un coffre de clés Azure. Apprenez-en davantage sur [Key Vault](../general/overview.md). 

Avec Key Vault et GitHub Actions, vous bénéficiez des avantages d’un outil de gestion des secrets centralisé et de tous les avantages offerts par GitHub Actions. GitHub Actions est une suite de fonctionnalités GitHub permettant d’automatiser vos workflows de développement logiciel. Vous pouvez déployer des workflows à l’emplacement où vous stockez le code et collaborez sur des demandes et des problèmes de tirage. 


## <a name="prerequisites"></a>Prérequis 
- Un compte GitHub. Si vous n’en avez pas, inscrivez-vous [gratuitement](https://github.com/join).  
- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Une application Azure connectée à un dépôt GitHub. Cet exemple utilise [Déployer des conteneurs sur Azure App Service](https://docs.microsoft.com/azure/developer/javascript/tutorial-vscode-docker-node-01). 
- Un coffre de clés Azure.  Vous pouvez créer un coffre de clés Azure à l’aide du portail Azure, d’Azure CLI ou d’Azure PowerShell.

## <a name="workflow-file-overview"></a>Vue d’ensemble du fichier de workflow

Un workflow est défini par un fichier YAML (.yml) situé dans le chemin `/.github/workflows/` de votre dépôt. Cette définition contient les étapes et les paramètres qui composent le workflow.

Le fichier comporte deux sections pour l’authentification auprès de GitHub Actions :

|Section  |Tâches  |
|---------|---------|
|**Authentification** | 1. Définissez un principal de service. <br /> 2. Créez un secret GitHub. <br /> 3. Ajouter une attribution de rôle. |
|**Key Vault** | 1. Ajouter l’action du coffre de clés. <br /> 2. Référencer un secret de coffre de clés. |

## <a name="authentication"></a>Authentification

Vous pouvez créer un [principal de service](../../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) à l’aide de la commande [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac&preserve-view=true) dans [Azure CLI](/cli/azure/). Exécutez cette commande en utilisant [Azure Cloud Shell](https://shell.azure.com/) dans le portail Azure ou en sélectionnant le bouton **Essayer**.

```azurecli-interactive
   az ad sp create-for-rbac --name {myApp} --role contributor --scopes /subscriptions/{subscription-id}/resourceGroups/{MyResourceGroup} --sdk-auth
```

Dans l’exemple ci-dessus, remplacez les espaces réservés par votre ID d’abonnement et le nom de votre groupe de ressources. Remplacez l’espace réservé `myApp` par le nom de votre application. La sortie correspond à un objet JSON avec les informations d’identification de l’attribution de rôle qui fournit l’accès à votre application App Service, similaire à ce qui suit. Copiez cet objet JSON pour une version ultérieure. Vous n’aurez besoin que des sections avec les valeurs `clientId`, `clientSecret`, `subscriptionId` et `tenantId`. 

```output 
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```

### <a name="configure-the-github-secret"></a>Configurer le secret GitHub

Créez des secrets pour vos informations d’identification, votre groupe de ressources et vos abonnements Azure. 

1. Dans [GitHub](https://github.com/), accédez à votre référentiel.

1. Sélectionnez **Paramètres > Secrets > Nouveau secret**.

1. Collez l’intégralité de la sortie JSON de la commande Azure CLI dans le champ de valeur du secret. Nommez le secret `AZURE_CREDENTIALS`.

1. Copiez la valeur de `clientId` pour l’utiliser ultérieurement. 

### <a name="add-a-role-assignment"></a>Ajouter une attribution de rôle 
 
Vous devez accorder l’accès au principal de service Azure afin de pouvoir accéder à votre coffre de clés pour les opérations `get` et `list`. Si vous ne le faites pas, vous ne pourrez pas utiliser le principal de service. 

Remplacez `keyVaultName` par le nom de votre coffre de clés et `clientIdGUID` par la valeur de votre `clientId`. 

```azurecli-interactive
    az keyvault set-policy -n {keyVaultName} --secret-permissions get list --spn {clientIdGUID}
```

## <a name="use-the-azure-key-vault-action"></a>Utiliser l’action de coffre de clés Azure

Avec l’[action de coffre de clés Azure](https://github.com/marketplace/actions/azure-key-vault-get-secrets), vous pouvez récupérer un ou plusieurs secrets à partir d’une instance de coffre de clés Azure et les consommer dans vos workflows GitHub Actions.

Les secrets récupérés sont définis en tant que sorties et également en tant que variables d’environnement. Les variables sont masquées automatiquement lorsqu’elles sont imprimées sur la console ou dans des journaux.

```yaml
    - uses: Azure/get-keyvault-secrets@v1
      with:
        keyvault: "my Vault" # name of key vault in Azure portal
        secrets: 'mySecret'  # comma separated list of secret keys to fetch from key vault 
      id: myGetSecretAction # ID for secrets that you will reference
```

Pour utiliser un coffre de clés dans votre workflow, vous avez besoin de l’action du coffre de clés et vous devez référencer cette action. 

Dans cet exemple, le coffre de clés se nomme `containervault`. Deux secrets de coffre de clés sont ajoutés à l’environnement avec l’action de coffre de clés : `containerPassword` et `containerUsername`. 

Les valeurs de coffre de clés sont ensuite référencées dans la tâche de connexion docker avec le préfixe `steps.myGetSecretAction.outputs`. 

```yaml
name: Example key vault flow

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    # checkout the repo
    - uses: actions/checkout@v2
    - uses: Azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    - uses: Azure/get-keyvault-secrets@v1
      with: 
        keyvault: "containervault"
        secrets: 'containerPassword, containerUsername'
      id: myGetSecretAction
    - uses: azure/docker-login@v1
      with:
        login-server: myregistry.azurecr.io
        username: ${{ steps.myGetSecretAction.outputs.containerUsername }}
        password: ${{ steps.myGetSecretAction.outputs.containerPassword }}
    - run: |
        docker build . -t myregistry.azurecr.io/myapp:${{ github.sha }}
        docker push myregistry.azurecr.io/myapp:${{ github.sha }}     
    - uses: azure/webapps-deploy@v2
      with:
        app-name: 'myapp'
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        images: 'myregistry.azurecr.io/myapp:${{ github.sha }}'
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque votre application Azure, votre dépôt GitHub et votre coffre de clés ne sont plus nécessaires, nettoyez les ressources que vous avez déployées en supprimant le groupe de ressources de l’application, le dépôt GitHub et le coffre de clés.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur Azure Key Vault](../general/overview.md)
