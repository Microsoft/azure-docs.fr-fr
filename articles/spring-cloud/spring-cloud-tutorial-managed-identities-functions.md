---
title: Tutoriel - Utilisation de l’identité managée pour appeler Azure Functions à partir d’une application Azure Spring Cloud
description: Utiliser une identité managée pour appeler Azure Functions d’une application Azure Spring Cloud
author: MarkGardner
ms.author: margard
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 07/10/2020
ms.openlocfilehash: 8ea8376307807abff8227d82bb6de7956fa3de99
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92088531"
---
# <a name="tutorial-use-a-managed-identity-to-invoke-azure-functions-from-an-azure-spring-cloud-app"></a>Tutoriel : Utiliser une identité managée pour appeler Azure Functions à partir d’une application Azure Spring Cloud

Cet article montre comment créer une identité managée pour une application Azure Spring Cloud et comment l’utiliser pour appeler des fonctions déclenchées via HTTP.

Azure Functions et App Services disposent d’une prise en charge intégrée de l’authentification Azure Active Directory (Azure AD). En tirant parti de l’authentification intégrée et des identités managées pour Azure Spring Cloud, nous pouvons appeler des services RESTful à l’aide de la sémantique OAuth moderne. Cette méthode ne demande pas le stockage de secrets dans le code et permet un contrôle plus précis de l’accès aux ressources externes. 


## <a name="prerequisites"></a>Prérequis

* [Souscrire à un abonnement Azure](https://azure.microsoft.com/free/)
* [Installez Azure CLI 2.0.67 ou version ultérieure](/cli/azure/install-azure-cli)
* [Installez Maven 3.0 ou ultérieur](https://maven.apache.org/download.cgi)
* [Installer Azure Functions Core Tools version 3.0.2009 ou ultérieure](../azure-functions/functions-run-local.md#install-the-azure-functions-core-tools).


## <a name="create-a-resource-group"></a>Créer un groupe de ressources
Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Créez un groupe de ressources devant contenir à la fois l’application de fonction et Spring Cloud, à l’aide de la commande [az group create](/cli/azure/group#az-group-create) :

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```


## <a name="create-a-function-app"></a>Créer une application de fonction
Pour créer une application de fonction, vous devez d’abord créer un compte de stockage de secours, à l’aide de la commande [az storage account create](/cli/azure/storage/account#az-storage-account-create) :

> [!Important]
> Chaque application de fonction et chaque compte de stockage doit avoir un nom unique. Dans les exemples suivants, remplacez <your-functionapp-name> par le nom de votre application de fonction, et <your-storageaccount-name> par le nom de votre compte de stockage.

```azurecli-interactive
az storage account create --name <your-storageaccount-name> --resource-group myResourceGroup --location eastus --sku Standard_LRS
```

Une fois le compte de stockage créé, vous pouvez créer l’application de fonction.

```azurecli-interactive
az functionapp create --name <your-functionapp-name> --resource-group myResourceGroup --consumption-plan-location eastus --os-type windows --runtime node --storage-account <your-storageaccount-name> --functions-version 3
```

Notez la valeur **hostNames** qui sera retournée au format « https://<nom-de-votre-application-de-fonction>.azurewebsites.net ». Vous en aurez besoin dans une prochaine étape.


## <a name="enable-azure-active-directory-authentication"></a>Activer l’authentification Azure Active Directory

Accédez à l’application de fonction que vous venez de créer à partir du [portail Azure](https://portal.azure.com), puis sélectionnez « Authentification/Autorisation » dans le menu Paramètres. Activez l’authentification App Service, puis définissez « Action à exécuter quand une demande n’est pas authentifiée » sur « Se connecter avec Azure Active Directory ». Ce paramètre garantit que toutes les demandes non authentifiées seront rejetées (réponse 401).

![Paramètres d’authentification où Azure Active Directory est le fournisseur par défaut](media/spring-cloud-tutorial-managed-identities-functions/function-auth-config-1.jpg)

Sous Fournisseurs d’authentification, sélectionnez Azure Active Directory pour configurer l’inscription de l’application. Si vous sélectionnez le mode de gestion Express, une inscription d’application sera automatiquement créée dans votre locataire Azure AD avec la configuration adaptée.

![Fournisseur Azure Active Directory avec le mode de gestion Express](media/spring-cloud-tutorial-managed-identities-functions/function-auth-config-2.jpg)

Une fois les paramètres enregistrés, l’application de fonction redémarre et toutes les demandes suivantes sont invitées à se connecter via Azure AD. Vous pouvez vérifier si les demandes non authentifiées sont désormais rejetées en accédant à l’URL racine des applications de fonction (retournée dans la sortie **hostNames** de l’étape ci-dessus). Vous devez être redirigé vers l’écran de connexion Azure AD de votre organisation.


## <a name="create-an-http-triggered-function"></a>Créer une fonction déclenchée via HTTP

Dans un répertoire local vide, créez une application de fonction, puis ajoutez une fonction déclenchée via HTTP.

```console
func init --worker-runtime node
func new --template HttpTrigger --name HttpTrigger
```

Par défaut, les fonctions utilisent l’authentification basée sur les clés pour sécuriser les points de terminaison HTTP. Étant donné que nous allons activer l’authentification Azure AD pour sécuriser l’accès aux fonctions, nous voulons [définir le niveau d’authentification de la fonction sur Anonyme](../azure-functions/functions-bindings-http-webhook-trigger.md#secure-an-http-endpoint-in-production).

```json function.json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      ...
    }
  ]
}
```

L’application peut maintenant être publiée dans l’instance d’[application de fonction](#create-a-function-app) créée à l’étape précédente.

```console
func azure functionapp publish <your-functionapp-name>
```

La sortie de la commande de publication doit comprendre l’URL de la fonction que vous venez de créer.

```output
Deployment completed successfully.
Syncing triggers...
Functions in <your-functionapp-name>:
    HttpTrigger - [httpTrigger]
        Invoke url: https://<your-functionapp-name>.azurewebsites.net/api/httptrigger
```


## <a name="create-azure-spring-cloud-service-and-app"></a>Créer un service et une application Azure Spring Cloud
Après l’installation de l’extension spring-cloud, créez une instance Azure Spring Cloud avec la commande Azure CLI `az spring-cloud create`. 

```azurecli-interactive
az extension add --name spring-cloud
az spring-cloud create --name mymsispringcloud --resource-group myResourceGroup --location eastus
```

L’exemple suivant crée une application nommée `msiapp` avec une identité managée affectée par le système, comme le demande le paramètre `--assign-identity`.

```azurecli
az spring-cloud app create --name "msiapp" --service "mymsispringcloud" --resource-group "myResourceGroup" --is-public true --assign-identity
```

## <a name="build-sample-spring-boot-app-to-invoke-the-function"></a>Générer un exemple d’application Spring Boot pour appeler la fonction

Cet exemple appellera la fonction déclenchée via HTTP en demandant d’abord un jeton d’accès au [point de terminaison MSI](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md#get-a-token-using-http), puis en utilisant ce jeton pour authentifier la requête HTTP de la fonction.

1. Clonez l’exemple de projet. 

    ```console
    git clone https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples.git
    ```

2. Spécifiez l’URI de votre fonction et le nom du déclencheur dans les propriétés de votre application. 

    ```bash
    cd Azure-Spring-Cloud-Samples/managed-identity-function
    vim src/main/resources/application.properties
    ```

    Pour utiliser une identité managée pour les applications Azure Spring Cloud, ajoutez des propriétés avec le contenu suivant à *src/main/resources/application.properties*.

    ```
    azure.function.uri=https://<your-functionapp-name>.azurewebsites.net
    azure.function.triggerPath=httptrigger
    ```

3. Empaquetez votre exemple d’application. 

    ```console
    mvn clean package
    ```

4. Déployez à présent l’application sur Azure avec la commande Azure CLI `az spring-cloud app deploy`. 

    ```azurecli
    az spring-cloud app deploy  --name "msiapp" --service "mymsispringcloud" --resource-group "myResourceGroup" --jar-path target/sc-managed-identity-function-sample-0.1.0.jar
    ```

5. Pour tester votre application, accédez au point de terminaison public ou au point de terminaison de test. 

    ```console
    curl https://mymsispringcloud-msiapp.azuremicroservices.io/func/springcloud
    ```

    Le message suivant sera retourné dans le corps de la réponse.
    ```output
    Function Response: Hello, springcloud. This HTTP triggered function executed successfully.
    ```
    
    Vous pouvez essayer de passer différentes valeurs à la fonction en modifiant le paramètre Path.

## <a name="next-steps"></a>Étapes suivantes

* [Comment activer une identité managée affectée par le système pour une application Azure Spring Cloud](./spring-cloud-howto-enable-system-assigned-managed-identity.md)
* [En savoir plus sur les identités managées pour les ressources Azure](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/active-directory/managed-identities-azure-resources/overview.md)
* [Configurer une application cliente de démon pour des appels de service à service](../app-service/configure-authentication-provider-aad.md#configure-a-daemon-client-application-for-service-to-service-calls)