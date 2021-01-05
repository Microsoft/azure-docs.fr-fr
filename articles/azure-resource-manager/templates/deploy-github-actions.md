---
title: Déployer des modèles Resource Manager à l’aide de GitHub Actions
description: Explique comment déployer des modèles Azure Resource Manager (modèles ARM) à l’aide de GitHub Actions.
ms.topic: conceptual
ms.date: 10/13/2020
ms.custom: github-actions-azure, devx-track-azurecli
ms.openlocfilehash: 4cda8307d417880469e6043b84c3ac55ed30071c
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905840"
---
# <a name="deploy-arm-templates-by-using-github-actions"></a>Déployer des modèles ARM à l’aide de GitHub Actions

[GitHub Actions](https://help.github.com/actions/getting-started-with-github-actions/about-github-actions) est une suite de fonctionnalités dans GitHub permettant d’automatiser vos workflows de développement logiciel dans le même emplacement que celui où vous stockez le code et collaborez sur les demandes de tirage (pull requests) et les problèmes.

Utilisez l’action [Déployer un modèle Azure Resource Manager](https://github.com/marketplace/actions/deploy-azure-resource-manager-arm-template) pour automatiser le déploiement d’un modèle Azure Resource Manager (modèle ARM) sur Azure.

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un compte GitHub. Si vous n’en avez pas, inscrivez-vous [gratuitement](https://github.com/join).
    - Un référentiel GitHub pour stocker vos modèles Resource Manager et vos fichiers de workflow. Pour en créer un, consultez [Création d’un référentiel](https://help.github.com/en/enterprise/2.14/user/articles/creating-a-new-repository).


## <a name="workflow-file-overview"></a>Vue d’ensemble du fichier de workflow

Un workflow est défini par un fichier YAML (.yml) situé dans le chemin `/.github/workflows/` de votre dépôt. Cette définition contient les étapes et les paramètres qui composent le workflow.

Le fichier comporte deux sections :

|Section  |Tâches  |
|---------|---------|
|**Authentification** | 1. Définissez un principal de service. <br /> 2. Créez un secret GitHub. |
|**Déployer** | 1. Déployez le modèle Resource Manager. |

## <a name="generate-deployment-credentials"></a>Générer les informations d’identification du déploiement


Vous pouvez créer un [principal de service](../../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) à l’aide de la commande [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac&preserve-view=true) dans [Azure CLI](/cli/azure/). Exécutez cette commande en utilisant [Azure Cloud Shell](https://shell.azure.com/) dans le portail Azure ou en sélectionnant le bouton **Essayer**.

Créez un groupe de ressources si vous n’en avez pas déjà un.

```azurecli-interactive
    az group create -n {MyResourceGroup}
```

Remplacez l’espace réservé `myApp` par le nom de votre application.

```azurecli-interactive
   az ad sp create-for-rbac --name {myApp} --role contributor --scopes /subscriptions/{subscription-id}/resourceGroups/{MyResourceGroup} --sdk-auth
```

Dans l’exemple ci-dessus, remplacez les espaces réservés par votre ID d’abonnement et le nom de votre groupe de ressources. La sortie correspond à un objet JSON avec les informations d’identification de l’attribution de rôle qui fournit l’accès à votre application App Service, similaire à ce qui suit. Copiez cet objet JSON pour une version ultérieure. Vous n’aurez besoin que des sections avec les valeurs `clientId`, `clientSecret`, `subscriptionId` et `tenantId`.

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
> Il est toujours conseillé d’accorder un accès minimal. Dans l’exemple précédent, l’étendue se limite au groupe de ressources.



## <a name="configure-the-github-secrets"></a>Configurer les secrets GitHub

Vous devez créer des secrets pour vos informations d’identification, votre groupe de ressources et vos abonnements Azure.

1. Dans [GitHub](https://github.com/), accédez à votre référentiel.

1. Sélectionnez **Paramètres > Secrets > Nouveau secret**.

1. Collez l’intégralité de la sortie JSON de la commande Azure CLI dans le champ de valeur du secret. Nommez le secret `AZURE_CREDENTIALS`.

1. Créez un autre secret nommé `AZURE_RG`. Ajoutez le nom de votre groupe de ressources au champ de valeur du secret (exemple : `myResourceGroup`).

1. Créez un secret supplémentaire nommé `AZURE_SUBSCRIPTION`. Ajoutez votre ID d’abonnement au champ de valeur du secret (exemple : `90fd3f9d-4c61-432d-99ba-1273f236afa2`).

## <a name="add-resource-manager-template"></a>Ajouter un modèle Resource Manager

Ajoutez un modèle Resource Manager à votre référentiel GitHub. Ce modèle crée un compte de stockage.

```url
https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
```

Vous pouvez placer le fichier n’importe où dans le référentiel. L’exemple de workflow dans la section suivante suppose que le fichier de modèle est nommé **azuredeploy.json** et qu’il est stocké à la racine de votre référentiel.

## <a name="create-workflow"></a>Créer un workflow

Le fichier de workflow doit être stocké dans le dossier **.github/workflows** à la racine de votre référentiel. L’extension du fichier de workflow peut être **.yml** ou **.yaml**.

1. À partir de votre référentiel GitHub, sélectionnez **Actions** dans le menu supérieur.
1. Sélectionnez **Nouveau workflow**.
1. Sélectionnez **Configurer vous-même un workflow**.
1. Renommez le fichier de workflow si vous préférez utiliser un autre nom que **main.yml**. Par exemple : **deployStorageAccount.yml**.
1. Remplacez le contenu du fichier yml par ce qui suit :

    ```yml
    on: [push]
    name: Azure ARM
    jobs:
      build-and-deploy:
        runs-on: ubuntu-latest
        steps:

          # Checkout code
        - uses: actions/checkout@main

          # Log into Azure
        - uses: azure/login@v1
          with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}

          # Deploy ARM template
        - name: Run ARM deploy
          uses: azure/arm-deploy@v1
          with:
            subscriptionId: ${{ secrets.AZURE_SUBSCRIPTION }}
            resourceGroupName: ${{ secrets.AZURE_RG }}
            template: ./azuredeploy.json
            parameters: storageAccountType=Standard_LRS

          # output containerName variable from template
        - run: echo ${{ steps.deploy.outputs.containerName }}
    ```
    > [!NOTE]
    > À la place, vous pouvez spécifier un fichier de paramètres au format JSON dans l’action de déploiement ARM (exemple : `.azuredeploy.parameters.json`).

    La première section du fichier de workflow comprend les éléments suivants :

    - **nom** : Nom du workflow.
    - **on** : nom des événements GitHub qui déclenchent le workflow. Le workflow est déclenché lorsqu’il y a un événement push sur la branche primaire, qui modifie au moins l’un des deux fichiers spécifiés. Les deux fichiers sont le fichier de workflow et le modèle de fichier.

1. Sélectionnez **Démarrer la validation**.
1. Sélectionnez **Valider directement sur la branche primaire**.
1. Sélectionnez **Valider un nouveau fichier** (ou **Valider les modifications**).

Étant donné que le workflow est configuré pour être déclenché par le fichier de workflow ou le modèle de fichier mis à jour, le workflow démarre juste après la validation des modifications.

## <a name="check-workflow-status"></a>Vérifier l’état du workflow

1. Sélectionnez l’onglet **Actions**. Un workflow **Créer deployStorageAccount.yml** s’affiche. L’exécution du workflow prend 1 à 2 minutes.
1. Sélectionnez le workflow pour l’ouvrir.
1. Sélectionnez **Exécuter le déploiement ARM** dans le menu pour vérifier le déploiement.

## <a name="clean-up-resources"></a>Nettoyer les ressources
Lorsque votre groupe de ressource et référentiel ne sont plus nécessaires, nettoyez les ressources que vous avez déployées en supprimant le groupe de ressources et votre référentiel GitHub.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer votre premier modèle ARM](./template-tutorial-create-first-template.md)
