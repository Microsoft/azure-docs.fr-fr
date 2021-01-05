---
title: Déployer une instance de conteneur par action GitHub
description: Configurez une action GitHub qui automatise les étapes de création, de transmission et de déploiement d’une image conteneur pour Azure Container Instances.
ms.topic: article
ms.date: 08/20/2020
ms.custom: github-actions-azure, devx-track-azurecli
ms.openlocfilehash: 0068b9869e5ef7a3b6c0fd21c1b69cdae1d4ab4b
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96454819"
---
# <a name="configure-a-github-action-to-create-a-container-instance"></a>Configurer une action GitHub pour créer une instance de conteneur

[GitHub Actions](https://help.github.com/actions/getting-started-with-github-actions/about-github-actions) est une suite de fonctionnalités dans GitHub permettant d’automatiser vos workflows de développement logiciel dans le même emplacement que celui où vous stockez le code et collaborez sur les demandes de tirage (pull requests) et les problèmes.

Utilisez l’action GitHub [Déployer sur Azure Container Instances](https://github.com/azure/aci-deploy) pour automatiser le déploiement d’un seul conteneur sur Azure Container Instances. L’action vous permet de définir les propriétés d’une instance de conteneur similaire à celles de la commande [az container create][az-container-create].

Cet article explique comment configurer un workflow dans un référentiel GitHub qui effectue les actions suivantes :

* Générer une image à partir d’un fichier Dockerfile
* Envoyer l’image vers un registre de conteneurs Azure
* Déployer l’image conteneur dans une instance de conteneur Azure

Cet article présente deux méthodes de configuration du workflow :

* [Configurer le workflow GitHub](#configure-github-workflow) : créez un workflow dans un dépôt GitHub à l’aide de l’action Déployer sur Azure Container Instances et d’autres actions.  
* [Utiliser l’extension CLI](#use-deploy-to-azure-extension) : utilisez la commande `az container app up` de l’extension [Déployer sur Azure](https://github.com/Azure/deploy-to-azure-cli-extension) dans l’interface de ligne de commande Azure. Cette commande simplifie la création du workflow GitHub et des étapes de déploiement.

> [!IMPORTANT]
> L’action GitHub pour Azure Container Instances est actuellement en préversion. Les préversions sont à votre disposition, à condition que vous acceptiez les [conditions d’utilisation supplémentaires][terms-of-use]. Certains aspects de cette fonctionnalité sont susceptibles d’être modifiés avant la mise à disposition générale.

## <a name="prerequisites"></a>Prérequis

* **Action GitHub** : créez un compte sur https://github.com si vous n’en avez pas encore.
* **Azure CLI** : pour effectuer ces étapes, vous pouvez utiliser Azure Cloud Shell ou une installation locale d’Azure CLI. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli-install].
* **Registre de conteneurs Azure** : si vous n’en avez pas, créez un registre de conteneurs Azure de niveau de base à l’aide d’[Azure CLI](../container-registry/container-registry-get-started-azure-cli.md), du [Portail Azure](../container-registry/container-registry-get-started-portal.md) ou d’autres méthodes. Notez le groupe de ressources utilisé pour le déploiement, il sera utilisé pour le workflow GitHub.

## <a name="set-up-repo"></a>Configurer le référentiel

* Pour les exemples de cet article, utilisez GitHub pour dupliquer (fork) le référentiel suivant : https://github.com/Azure-Samples/acr-build-helloworld-node

  Ce référentiel contient un Dockerfile et des fichiers sources pour créer l’image conteneur d’une petite application web.

  ![Capture d’écran du bouton de duplication (mis en surbrillance) dans GitHub](../container-registry/media/container-registry-tutorial-quick-build/quick-build-01-fork.png)

* Assurez-vous que les actions sont activées pour votre référentiel. Accédez à votre référentiel dupliqué, puis sélectionnez **Paramètres** > **Actions**. Dans **Autorisations des actions**, assurez-vous que l’option **Activer les actions locales et tierces pour ce référentiel** est sélectionnée.

## <a name="configure-github-workflow"></a>Configurer le workflow GitHub

### <a name="create-service-principal-for-azure-authentication"></a>Créer un principal du service pour l’authentification Azure

Dans le workflow GitHub, vous devez fournir des informations d’identification Azure pour l’authentification auprès d’Azure CLI. L’exemple suivant crée un principal de service avec le rôle Contributeur étendu au groupe de ressources de votre registre de conteneurs.

Tout d’abord, récupérez l’ID de ressource de votre groupe de ressources. Substituez le nom de votre groupe dans la commande [az group show][az-group-show] suivante :

```azurecli
$groupId=$(az group show \
  --name <resource-group-name> \
  --query id --output tsv)
```

Utilisez [az ad sp create-for-rbac][az-ad-sp-create-for-rbac] pour créer le principal du service :

```azurecli
az ad sp create-for-rbac \
  --scope $groupId \
  --role Contributor \
  --sdk-auth
```

Le résultat se présente ainsi :

```json
{
  "clientId": "xxxx6ddc-xxxx-xxxx-xxx-ef78a99dxxxx",
  "clientSecret": "xxxx79dc-xxxx-xxxx-xxxx-aaaaaec5xxxx",
  "subscriptionId": "xxxx251c-xxxx-xxxx-xxxx-bf99a306xxxx",
  "tenantId": "xxxx88bf-xxxx-xxxx-xxxx-2d7cd011xxxx",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```

Enregistrez la sortie JSON, car elle est utilisée à une étape ultérieure. Notez également le `clientId`, dont vous avez besoin pour mettre à jour le principal du service dans la section suivante.

### <a name="update-service-principal-for-registry-authentication"></a>Mettre à jour le principal du service pour l’authentification du registre

Mettez à jour les informations d’identification du principal du service Azure pour autoriser les accès push et pull (envoi et tirage) à votre registre de conteneurs. Cette étape permet au workflow GitHub d’utiliser le principal du service pour [s’authentifier auprès de votre registre de conteneurs](../container-registry/container-registry-auth-service-principal.md) ainsi que d’envoyer et d’extraire une image Docker. 

Récupérez l’ID de ressource de votre registre de conteneurs. Substituez le nom de votre registre dans la commande [az acr show][az-acr-show] suivante :

```azurecli
registryId=$(az acr show \
  --name <registry-name> \
  --query id --output tsv)
```

Utilisez [az role assignment create][az-role-assignment-create] pour attribuer le rôle AcrPush, qui octroie un accès push et pull au registre. Substituez l’ID client de votre principal de service :

```azurecli
az role assignment create \
  --assignee <ClientId> \
  --scope $registryId \
  --role AcrPush
```

### <a name="save-credentials-to-github-repo"></a>Enregistrer les informations d’identification dans le référentiel GitHub

1. Dans l’interface utilisateur GitHub, accédez à votre référentiel dupliqué, puis sélectionnez **Paramètres** > **Secrets**. 

1. Sélectionnez **Ajouter un secret** pour ajouter les secrets suivants :

|Secret  |Valeur  |
|---------|---------|
|`AZURE_CREDENTIALS`     | Étape de création de la totalité de la sortie JSON à partir du principal de service |
|`REGISTRY_LOGIN_SERVER`   | Nom du serveur de connexion de votre registre (tout en minuscules). Exemple : *myregistry.azurecr.io*        |
|`REGISTRY_USERNAME`     |  `clientId` de la sortie JSON issue de la création du principal de service       |
|`REGISTRY_PASSWORD`     |  `clientSecret` de la sortie JSON issue de la création du principal de service |
| `RESOURCE_GROUP` | Nom du groupe de ressources que vous avez utilisé pour définir l’étendue du principal de service |

### <a name="create-workflow-file"></a>Créer un fichier de workflow

1. Dans l’interface utilisateur GitHub, sélectionnez **Actions** > **Nouveau workflow**.
1. Sélectionnez **Configurer vous-même un workflow**.
1. Dans **Modifier le nouveau fichier**, collez le contenu YAML suivant pour remplacer l’exemple de code. Acceptez le nom de fichier par défaut `main.yml` ou fournissez un nom de fichier de votre choix.
1. Sélectionnez **Démarrer la validation**, fournissez éventuellement des descriptions courtes et détaillées de votre validation, puis sélectionnez **Valider le nouveau fichier**.

```yml
on: [push]
name: Linux_Container_Workflow

jobs:
    build-and-deploy:
        runs-on: ubuntu-latest
        steps:
        # checkout the repo
        - name: 'Checkout GitHub Action'
          uses: actions/checkout@master
          
        - name: 'Login via Azure CLI'
          uses: azure/login@v1
          with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}
        
        - name: 'Build and push image'
          uses: azure/docker-login@v1
          with:
            login-server: ${{ secrets.REGISTRY_LOGIN_SERVER }}
            username: ${{ secrets.REGISTRY_USERNAME }}
            password: ${{ secrets.REGISTRY_PASSWORD }}
        - run: |
            docker build . -t ${{ secrets.REGISTRY_LOGIN_SERVER }}/sampleapp:${{ github.sha }}
            docker push ${{ secrets.REGISTRY_LOGIN_SERVER }}/sampleapp:${{ github.sha }}

        - name: 'Deploy to Azure Container Instances'
          uses: 'azure/aci-deploy@v1'
          with:
            resource-group: ${{ secrets.RESOURCE_GROUP }}
            dns-name-label: ${{ secrets.RESOURCE_GROUP }}${{ github.run_number }}
            image: ${{ secrets.REGISTRY_LOGIN_SERVER }}/sampleapp:${{ github.sha }}
            registry-login-server: ${{ secrets.REGISTRY_LOGIN_SERVER }}
            registry-username: ${{ secrets.REGISTRY_USERNAME }}
            registry-password: ${{ secrets.REGISTRY_PASSWORD }}
            name: aci-sampleapp
            location: 'west us'
```

### <a name="validate-workflow"></a>Valider le workflow

Une fois que vous avez validé le fichier du workflow, le workflow est déclenché. Pour passer en revue la progression du workflow, accédez à **Actions** > **Workflows**. 

![Afficher la progression du workflow](./media/container-instances-github-action/github-action-progress.png)

Pour plus d’informations sur l’affichage de l’état et des résultats de chaque étape de workflow, consultez [Gestion d’une exécution de workflow](https://help.github.com/actions/configuring-and-managing-workflows/managing-a-workflow-run). Si le workflow ne se termine pas, consultez [Consultation des journaux pour diagnostiquer les défaillances](https://docs.github.com/actions/configuring-and-managing-workflows/managing-a-workflow-run#viewing-logs-to-diagnose-failures).

Une fois le workflow terminé, récupérez des informations sur l’instance de conteneur nommée *aci-sampleapp* en exécutant la commande [az container show][az-container-show]. Substituez le nom de votre groupe de ressources : 

```azurecli
az container show \
  --resource-group <resource-group-name> \
  --name aci-sampleapp \
  --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" \
  --output table
```

Le résultat se présente ainsi :

```console
FQDN                                   ProvisioningState
---------------------------------      -------------------
aci-action01.westus.azurecontainer.io  Succeeded
```

Une fois l’instance approvisionnée, accédez au nom de domaine complet du conteneur dans votre navigateur pour afficher l’application web en cours d’exécution.

![Application web en cours d’exécution dans le navigateur](./media/container-instances-github-action/github-action-container.png)

## <a name="use-deploy-to-azure-extension"></a>Utiliser l’extension Déployer sur Azure

Vous pouvez également utiliser l’[extension Déployer sur Azure](https://github.com/Azure/deploy-to-azure-cli-extension) dans l’interface de ligne de commande Azure pour configurer le workflow. La commande `az container app up` de l’extension accepte des paramètres d’entrée vous permettant de configurer un workflow à déployer sur Azure Container Instances. 

Le workflow créé par Azure CLI est similaire au workflow que vous pouvez [créer manuellement à l’aide de GitHub](#configure-github-workflow).

### <a name="additional-prerequisite"></a>Autres conditions préalables

Outre les [conditions préalables](#prerequisites) et la [configuration du référentiel](#set-up-repo) pour ce scénario, vous devez installer l’**extension Déployer sur Azure** pour Azure CLI.

Exécutez la commande [az extension add][az-extension-add] pour installer l’extension :

```azurecli
az extension add \
  --name deploy-to-azure
```

Pour plus d’informations sur la recherche, l’installation et la gestion d’extensions, consultez [Utiliser des extensions avec Azure CLI](/cli/azure/azure-cli-extensions-overview).

### <a name="run-az-container-app-up"></a>Exécutez `az container app up`

Pour exécuter la commande [az container app up][az-container-app-up], indiquez au minimum :

* Le nom de votre registre de conteneurs Azure, par exemple : *myregistry*
* L’URL de votre référentiel GitHub, par exemple : `https://github.com/<your-GitHub-Id>/acr-build-helloworld-node`

Exemple de commande :

```azurecli
az container app up \
  --acr myregistry \
  --repository https://github.com/myID/acr-build-helloworld-node
```

### <a name="command-progress"></a>Progression de la commande

* Lorsque vous y êtes invité, fournissez vos informations d’identification GitHub ou un [jeton d’accès personnel (PAT) GitHub](https://help.github.com/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) doté des étendues *dépôt* et *utilisateur* pour vous authentifier auprès de votre compte GitHub. Si vous fournissez des informations d’identification GitHub, la commande crée un PAT pour vous. Suivez les invites supplémentaires pour configurer le workflow.

* La commande crée des secrets de référentiel pour le workflow :

  * Informations d’identification du principal de service pour Azure CLI
  * Informations d’identification pour accéder au registre de conteneurs Azure

* Une fois que la commande a validé le fichier de workflow sur votre référentiel, le workflow est déclenché. 

Le résultat se présente ainsi :

```console
[...]
Checking in file github/workflows/main.yml in the Github repository myid/acr-build-helloworld-node
Creating workflow...
GitHub Action Workflow has been created - https://github.com/myid/acr-build-helloworld-node/runs/515192398
GitHub workflow completed.
Workflow succeeded
Your app is deployed at:  http://acr-build-helloworld-node.eastus.azurecontainer.io:8080/
```

Pour afficher l’état du workflow et les résultats de chaque étape dans l’interface utilisateur GitHub, consultez [Gestion d’une exécution de workflow](https://help.github.com/actions/configuring-and-managing-workflows/managing-a-workflow-run).

### <a name="validate-workflow"></a>Valider le workflow

Le workflow déploie une instance de conteneur Azure avec le nom de base de votre référentiel GitHub, dans le cas présent : *acr-build-helloworld-node*. Une fois le workflow terminé, récupérez des informations sur l’instance de conteneur nommée *acr-build-helloworld-node* en exécutant la commande [az container show][az-container-show]. Substituez le nom de votre groupe de ressources : 

```azurecli
az container show \
  --resource-group <resource-group-name> \
  --name acr-build-helloworld-node \
  --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" \
  --output table
```

Le résultat se présente ainsi :

```console
FQDN                                                   ProvisioningState
---------------------------------                      -------------------
acr-build-helloworld-node.westus.azurecontainer.io     Succeeded
```

Une fois l’instance approvisionnée, accédez au nom de domaine complet du conteneur dans votre navigateur pour afficher l’application web en cours d’exécution.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Arrêtez l’instance de conteneur avec la commande [az container delete][az-container-delete] :

```azurecli
az container delete \
  --name <instance-name>
  --resource-group <resource-group-name>
```

Pour supprimer le groupe de ressources et toutes les ressources qu’il contient, exécutez la commande [az group delete][az-group-delete] :

```azurecli
az group delete \
  --name <resource-group-name>
```

## <a name="next-steps"></a>Étapes suivantes

Parcourez le [marketplace GitHub](https://github.com/marketplace?type=actions) afin d’obtenir plus d’actions pour automatiser votre workflow de développement.


<!-- LINKS - external -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - internal -->

[azure-cli-install]: /cli/azure/install-azure-cli
[az-group-show]: /cli/azure/group#az-group-show
[az-group-delete]: /cli/azure/group#az-group-delete
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-container-create]: /cli/azure/container#az-container-create
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-container-show]: /cli/azure/container#az-container-show
[az-container-delete]: /cli/azure/container#az-container-delete
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-container-app-up]: /cli/azure/ext/deploy-to-azure/container/app#ext-deploy-to-azure-az-container-app-up
