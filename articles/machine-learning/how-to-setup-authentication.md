---
title: Configurer l’authentification
titleSuffix: Azure Machine Learning
description: Découvrez comment installer et configurer l’authentification pour divers workflows et ressources dans Azure Machine Learning.
services: machine-learning
author: cjgronlund
ms.author: cgronlun
ms.reviewer: larryfr
ms.service: machine-learning
ms.subservice: core
ms.date: 11/05/2020
ms.topic: conceptual
ms.custom: how-to, has-adal-ref, devx-track-js, devx-track-azurecli, contperf-fy21q2
ms.openlocfilehash: 27c8a0b80068124613af15565f387f15ac6b8e57
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97027252"
---
# <a name="set-up-authentication-for-azure-machine-learning-resources-and-workflows"></a>Configurer l’authentification pour des ressources et workflows Azure Machine Learning


Découvrez comment configurer l’authentification à votre espace de travail Azure Machine Learning. L’authentification auprès de votre espace de travail Azure Machine Learning est en grande partie basée sur __Azure Active Directory__ (Azure AD). En général, il existe trois workflows d’authentification que vous pouvez utiliser au moment de la connexion à l’espace de travail :

* __Interactive :__ vous utilisez votre compte dans Azure Active Directory pour l’authentification directe ou pour obtenir un jeton utilisé pour l’authentification. L’authentification interactive est utilisée pendant _l’expérimentation et le développement itératif_. L’authentification interactive vous permet de contrôler l’accès aux ressources (par exemple, un service web) pour chaque utilisateur.

* __Principal du service__ : Vous créez un compte de principal de service dans Azure Active Directory et l’utilisez pour authentifier ou obtenir un jeton. Un principal de service est utilisé lorsque vous avez besoin d’un _processus automatisé pour l’authentification_ auprès du service, sans intervention de l’utilisateur. Par exemple, un script d’intégration et de déploiement continus qui forme et teste un modèle chaque fois que le code d’apprentissage change.

* __Identité managée__ : Lorsque vous utilisez le kit de développement logiciel (SDK) Azure Machine Learning _sur une machine virtuelle Azure_, vous pouvez utiliser une identité managée pour Azure. Ce flux de travail permet à la machine virtuelle de se connecter à l’espace de travail à l’aide de l’identité gérée, sans stocker les informations d’identification dans le code Python ni inviter l’utilisateur à s’authentifier. Les clusters de calcul Azure Machine Learning peuvent également être configurés pour utiliser une identité managée afin d’accéder à l’espace de travail lors de la _formation de modèles_.

> [!IMPORTANT]
> Quel que soit le workflow d’authentification utilisé, le contrôle d’accès en fonction du rôle Azure (Azure RBAC) permet de définir l’étendue du niveau d’accès (autorisation) autorisé aux ressources. Par exemple, un administrateur ou un processus d’automatisation peut avoir accès à la création d’une instance de calcul, mais ne l’utilise pas, alors qu’un scientifique des données peut l’utiliser, mais pas la supprimer ou la créer. Pour plus d’informations, consultez [Gérer l’accès à un espace de travail Azure Machine Learning](how-to-assign-roles.md).

## <a name="prerequisites"></a>Prérequis

* Créez un [espace de travail Azure Machine Learning](how-to-manage-workspace.md).
* [Configurez votre environnement de développement](how-to-configure-environment.md) pour installer le SDK Azure Machine Learning ou utilisez une [instance de calcul Azure Machine Learning](concept-azure-machine-learning-architecture.md#compute-instance) avec le SDK déjà installé.

## <a name="azure-active-directory"></a>Azure Active Directory

Tous les workflows d’authentification pour votre espace de travail reposent sur Azure Active Directory. Si vous souhaitez que les utilisateurs s’authentifient à l’aide de comptes individuels, ceux-ci doivent disposer de comptes dans votre Azure AD. Si vous souhaitez utiliser des principaux de service, ceux-ci doivent exister dans votre Azure AD. Les identités managées sont également une fonctionnalité d’Azure AD. 

Pour en savoir plus sur Azure AD, consultez [Qu’est-ce que l’authentification Azure Active Directory](..//active-directory/authentication/overview-authentication.md).

Après avoir créé les comptes Azure AD, consultez [Gérer l’accès à l’espace de travail Azure Machine Learning](how-to-assign-roles.md) pour plus d’informations sur l’octroi à ceux-ci de l’accès à l’espace de travail, ainsi que sur d’autres opérations dans Azure Machine Learning.

## <a name="configure-a-service-principal"></a>Configurer un principal du service

Pour utiliser un principal de service, vous devez d’abord créer le principal de service et lui accorder l’accès à votre espace de travail. Comme mentionné précédemment, le contrôle d’accès en fonction du rôle (Azure RBAC) permet de contrôler l’accès. Vous devez donc également décider de l’accès accordé au principal du service.

> [!IMPORTANT]
> Lorsque vous utilisez un principal de service, accordez-lui l’__accès minimal requis pour la tâche__ pour laquelle il est utilisé. Par exemple, vous ne pouvez pas accorder l’accès propriétaire ou contributeur au principal de service s’il est utilisé uniquement pour la lecture du jeton d’accès pour un déploiement web.
>
> La raison pour laquelle vous accordez l’accès le plus bas est qu’un principal de service utilise un mot de passe pour l’authentification et que le mot de passe peut être stocké dans le cadre d’un script d’automatisation. Si le mot de passe est divulgué, le fait d’avoir un accès minimal requis pour une tâche spécifique réduit au minimum l’utilisation malveillante du principal de service.

Le moyen le plus simple de créer un principal de service et de lui accorder l’accès à votre espace de travail consiste à utiliser [Azure CLI](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest). Pour créer un principal de service et lui accorder l’accès à votre espace de travail, procédez comme suit :

> [!NOTE]
> Vous devez être administrateur de l’abonnement pour pouvoir effectuer toutes les étapes ci-dessous.

1. Authentifiez-vous auprès de votre abonnement Azure :

    ```azurecli-interactive
    az login
    ```

    Si l’interface CLI peut ouvrir votre navigateur par défaut, elle le fera et chargera une page de connexion par la même occasion. Dans le cas contraire, vous devez ouvrir un navigateur et suivre les instructions de la ligne de commande. Les instructions impliquent de naviguer vers [https://aka.ms/devicelogin](https://aka.ms/devicelogin) et d’entrer un code d’autorisation.

    Si vous avez plusieurs abonnements Azure, vous pouvez utiliser la commande `az account set -s <subscription name or ID>` pour configurer l’abonnement. Pour plus d'informations, consultez [Utiliser plusieurs abonnements Azure](/cli/azure/manage-azure-subscriptions-azure-cli?view=azure-cli-latest).

    Pour les autres méthodes d’authentification, consultez [Se connecter avec Azure CLI](/cli/azure/authenticate-azure-cli?preserve-view=true&view=azure-cli-latest).

1. Installez l’extension Azure Machine Learning :

    ```azurecli-interactive
    az extension add -n azure-cli-ml
    ```

1. Créez le principal de service. Dans l’exemple suivant, un principal de service nommé **ml-auth** est créé :

    ```azurecli-interactive
    az ad sp create-for-rbac --sdk-auth --name ml-auth
    ```

    Vous devez obtenir une sortie JSON semblable à ce qui suit. Notez les valeurs des champs `clientId`, `clientSecret`et `tenantId`, car vous en aurez besoin pour certaines étapes ultérieures dans cet article.

    ```json
    {
        "clientId": "your-client-id",
        "clientSecret": "your-client-secret",
        "subscriptionId": "your-sub-id",
        "tenantId": "your-tenant-id",
        "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
        "resourceManagerEndpointUrl": "https://management.azure.com",
        "activeDirectoryGraphResourceId": "https://graph.windows.net",
        "sqlManagementEndpointUrl": "https://management.core.windows.net:5555",
        "galleryEndpointUrl": "https://gallery.azure.com/",
        "managementEndpointUrl": "https://management.core.windows.net"
    }
    ```

1. Récupérez les détails du principal de service à l’aide de la valeur `clientId` retournée à l’étape précédente :

    ```azurecli-interactive
    az ad sp show --id your-client-id
    ```

    Le code JSON ci-dessous est un exemple simplifié de sortie générée par la commande. Notez la valeur du champ `objectId`, car vous devrez l’indiquer à la prochaine étape.

    ```json
    {
        "accountEnabled": "True",
        "addIns": [],
        "appDisplayName": "ml-auth",
        ...
        ...
        ...
        "objectId": "your-sp-object-id",
        "objectType": "ServicePrincipal"
    }
    ```

1. Autorisez le principal de service à accéder à votre espace de travail Azure Machine Learning. Définissez les paramètres `-w` et `-g` respectivement avec le nom de votre espace de travail et le nom du groupe de ressources associé. Pour le paramètre `--user`, utilisez la valeur `objectId` notée à l’étape précédente. Le paramètre `--role` vous permet de définir le rôle d’accès du principal de service. Dans l’exemple suivant, le principal de service est attribué au rôle **propriétaire**. 

    > [!IMPORTANT]
    > L’accès propriétaire permet au principal de service d’effectuer quasiment n’importe quelle opération dans votre espace de travail. Il est utilisé dans ce document pour vous montrer comment accorder l’accès. Dans un environnement de production, Microsoft recommande d’accorder au principal de service l’accès minimal nécessaire pour qu’il puisse exécuter le rôle que vous lui donnez. Pour plus d’informations sur la création d’un rôle personnalisé avec l’accès nécessaire pour votre scénario, consultez [Gérer l’accès à l’espac de travail Azure Machine Learning](how-to-assign-roles.md).

    ```azurecli-interactive
    az ml workspace share -w your-workspace-name -g your-resource-group-name --user your-sp-object-id --role owner
    ```

    Cet appel ne produit pas de sortie en cas de réussite.

## <a name="configure-a-managed-identity"></a>Configurer une identité managée

> [!IMPORTANT]
> L’identité gérée n’est prise en charge que lors de l’utilisation du kit de développement logiciel (SDK) Azure Machine Learning à partir d’une machine virtuelle Azure ou d’un cluster de calcul Azure Machine Learning. L’utilisation d’une identité gérée avec un cluster de calcul est actuellement en version préliminaire.

### <a name="managed-identity-with-a-vm"></a>Identité managée avec une machine virtuelle

1. Activer une [Identité managée affectée par le système pour les ressources Azure sur la machine virtuelle](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#system-assigned-managed-identity).

1. Depuis le [portail Azure](https://portal.azure.com), sélectionnez votre espace de travail, puis sélectionnez __Access Control (IAM)__ , __Ajouter une attribution de rôle__ et sélectionnez __Machine virtuelle__ dans la liste déroulante __attribuer l’accès à__. Enfin, sélectionnez l’identité de votre machine virtuelle.

1. Sélectionnez le rôle à affecter à cette identité. Par exemple, le rôle de contributeur ou un rôle personnalisé. Pour plus d’informations, consultez [Contrôler l’accès aux ressources](how-to-assign-roles.md).

### <a name="managed-identity-with-compute-cluster"></a>Identité managée avec cluster de calcul

Pour plus d’informations, consultez [Configurer une identité managée pour un cluster de calcul](how-to-create-attach-compute-cluster.md#managed-identity).

<a id="interactive-authentication"></a>

## <a name="use-interactive-authentication"></a>Utiliser l’authentification interactive

> [!IMPORTANT]
> L’authentification interactive utilise votre navigateur et nécessite des cookies (notamment des cookies tiers). Si vous avez désactivé les cookies, vous risquez de recevoir un message d’erreur tel que « Nous n’avons pas pu vous connecter ». Cette erreur peut également se produire si vous avez activé [Azure AD Multi-Factor Authentication](../active-directory/authentication/concept-mfa-howitworks.md).

La plupart des exemples de la documentation et des exemples utilisent l’authentification interactive. Par exemple, lors de l’utilisation du SDK, deux appels de fonction vous invitent automatiquement à utiliser un flux d’authentification basée sur l’interface utilisateur :

* L’appel de la fonction `from_config()` génère l’invite.

    ```python
    from azureml.core import Workspace
    ws = Workspace.from_config()
    ```

    La fonction `from_config()` recherche un fichier JSON contenant les informations de connexion à votre espace de travail.

* Lorsque vous utilisez le constructeur `Workspace` pour fournir des informations sur l’abonnement, le groupe de ressources et l’espace de travail, vous êtes également invité à effectuer une authentification interactive.

    ```python
    ws = Workspace(subscription_id="your-sub-id",
                  resource_group="your-resource-group-id",
                  workspace_name="your-workspace-name"
                  )
    ```

> [!TIP]
> Si vous avez accès à plusieurs locataires, vous devrez peut-être importer la classe et définir explicitement le locataire ciblé. L’appel du constructeur pour `InteractiveLoginAuthentication` génère également une invite de connexion comme avec les appels montrés ci-dessus.
>
> ```python
> from azureml.core.authentication import InteractiveLoginAuthentication
> interactive_auth = InteractiveLoginAuthentication(tenant_id="your-tenant-id")
> ```

Lors de l’utilisation de l’interface de ligne de commande Azure, la commande `az login` est utilisée pour authentifier la session de l’interface de ligne de commande. Pour plus d’informations, consultez [Prise en main d’Azure CLI](/cli/azure/get-started-with-azure-cli).

> [!TIP]
> Si vous utilisez le kit de développement logiciel (SDK) à partir d’un environnement auquel vous vous êtes précédemment authentifié de façon interactive à l’aide de l’interface de ligne de commande Azure, vous pouvez utiliser la classe `AzureCliAuthentication` pour vous authentifier à l’espace de travail à l’aide des informations d’identification mises en cache par l’interface de ligne de commande :
>
> ```python
> from azureml.core.authentication import AzureCliAuthentication
> cli_auth = AzureCliAuthentication()
> ws = Workspace(subscription_id="your-sub-id",
>                resource_group="your-resource-group-id",
>                workspace_name="your-workspace-name",
>                auth=cli_auth
>                )
> ```

<a id="service-principal-authentication"></a>

## <a name="use-service-principal-authentication"></a>Utiliser une authentification de principal de service

Pour vous authentifier sur votre espace de travail à partir du SDK, à l’aide d’un principal de service, utilisez le constructeur de classe `ServicePrincipalAuthentication`. Utilisez les valeurs que vous avez obtenues lors de la création du fournisseur de services en tant que paramètres. Les paramètres `tenant_id`, `service_principal_id` et `service_principal_password` sont respectivement mappés aux valeurs `tenantId`, `clientId`et `clientSecret` notées plus haut.

```python
from azureml.core.authentication import ServicePrincipalAuthentication

sp = ServicePrincipalAuthentication(tenant_id="your-tenant-id", # tenantID
                                    service_principal_id="your-client-id", # clientId
                                    service_principal_password="your-client-secret") # clientSecret
```

La variable `sp` contient maintenant un objet d’authentification que vous utilisez directement dans le SDK. En règle générale, il est conseillé de stocker les ID/secrets utilisés ci-dessus dans des variables d’environnement, comme dans ce code. Le stockage dans des variables d’environnement empêche l’archivage accidentel des informations dans un dépôt GitHub.

```python
import os

sp = ServicePrincipalAuthentication(tenant_id=os.environ['AML_TENANT_ID'],
                                    service_principal_id=os.environ['AML_PRINCIPAL_ID'],
                                    service_principal_password=os.environ['AML_PRINCIPAL_PASS'])
```

Pour les workflows automatisés qui s’exécutent dans Python et utilisent principalement le SDK, vous pouvez utiliser cet objet tel quel dans la plupart des cas d’authentification. Le code suivant effectue l’authentification auprès de votre espace de travail à l’aide de l’objet d’authentification que vous avez créé.

```python
from azureml.core import Workspace

ws = Workspace.get(name="ml-example",
                   auth=sp,
                   subscription_id="your-sub-id")
ws.get_details()
```

### <a name="use-a-service-principal-from-the-azure-cli"></a>Utiliser un principal de service à partir d’Azure CLI

Vous pouvez utiliser un principal de service pour les commandes Azure CLI. Pour plus d’informations, consultez [Se connecter avec un principal de service](/cli/azure/create-an-azure-service-principal-azure-cli?preserve-view=true&view=azure-cli-latest#sign-in-using-a-service-principal).

### <a name="use-a-service-principal-with-the-rest-api-preview"></a>Utiliser un principal de service avec l’API REST (préversion)

Le principal de service permet également de s’authentifier auprès de l’[’API REST](/rest/api/azureml/) Azure Machine Learning (préversion). Vous utilisez le [flux d’octroi des informations d’identification du client](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md) Azure Active Directory, qui permet d’effectuer des appels de service à service pour l’authentification sans affichage dans les workflows automatisés. Les exemples sont implémentés avec la [bibliothèque ADAL](../active-directory/azuread-dev/active-directory-authentication-libraries.md) en Python et en Node.js, mais vous pouvez aussi utiliser n’importe quelle bibliothèque open source prenant en charge OpenID Connect 1.0.

> [!NOTE]
> MSAL.js est une bibliothèque plus récente qu’ADAL. Toutefois, MSAL.js ne prend pas en charge l’authentification de service à service avec les informations d’identification du client, car il s’agit principalement d’une bibliothèque côté client conçue pour l’authentification interactive/basée sur l’interface utilisateur qui est associée à un utilisateur spécifique. Nous vous recommandons d’utiliser ADAL comme indiqué ci-dessous pour créer des workflows automatisés avec l’API REST.

#### <a name="nodejs"></a>Node.js

Effectuez les étapes suivantes pour générer un jeton d’authentification avec Node.js. Dans votre environnement, exécutez `npm install adal-node`. Ensuite, utilisez les valeurs de `tenantId`, `clientId` et `clientSecret` du principal de service que vous avez créé précédemment comme valeurs des variables correspondantes dans ce script.

```javascript
const adal = require('adal-node').AuthenticationContext;

const authorityHostUrl = 'https://login.microsoftonline.com/';
const tenantId = 'your-tenant-id';
const authorityUrl = authorityHostUrl + tenantId;
const clientId = 'your-client-id';
const clientSecret = 'your-client-secret';
const resource = 'https://management.azure.com/';

const context = new adal(authorityUrl);

context.acquireTokenWithClientCredentials(
  resource,
  clientId,
  clientSecret,
  (err, tokenResponse) => {
    if (err) {
      console.log(`Token generation failed due to ${err}`);
    } else {
      console.dir(tokenResponse, { depth: null, colors: true });
    }
  }
);
```

La variable `tokenResponse` est un objet qui contient le jeton et des métadonnées associées, comme le délai d’expiration. Les jetons restent valides pendant 1 heure, après quoi vous devez les actualiser en réexécutant le même appel pour récupérer un nouveau jeton. L’extrait de code suivant est un exemple de réponse.

```javascript
{
    tokenType: 'Bearer',
    expiresIn: 3599,
    expiresOn: 2019-12-17T19:15:56.326Z,
    resource: 'https://management.azure.com/',
    accessToken: "random-oauth-token",
    isMRRT: true,
    _clientId: 'your-client-id',
    _authority: 'https://login.microsoftonline.com/your-tenant-id'
}
```

Utilisez la propriété `accessToken` pour récupérer (fetch) le jeton d’authentification. Consultez la [documentation de l’API REST](https://github.com/microsoft/MLOps/tree/master/examples/AzureML-REST-API) si vous souhaitez voir des exemples d’utilisation du jeton pour effectuer des appels d’API.

#### <a name="python"></a>Python

Effectuez les étapes suivantes pour générer un jeton d’authentification avec Python. Dans votre environnement, exécutez `pip install adal`. Ensuite, utilisez les valeurs de `tenantId`, `clientId` et `clientSecret` du principal de service que vous avez créé précédemment comme valeurs des variables correspondantes dans ce script.

```python
from adal import AuthenticationContext

client_id = "your-client-id"
client_secret = "your-client-secret"
resource_url = "https://login.microsoftonline.com"
tenant_id = "your-tenant-id"
authority = "{}/{}".format(resource_url, tenant_id)

auth_context = AuthenticationContext(authority)
token_response = auth_context.acquire_token_with_client_credentials("https://management.azure.com/", client_id, client_secret)
print(token_response)
```

La variable `token_response` est un dictionnaire qui contient le jeton et des métadonnées associées, comme le délai d’expiration. Les jetons restent valides pendant 1 heure, après quoi vous devez les actualiser en réexécutant le même appel pour récupérer un nouveau jeton. L’extrait de code suivant est un exemple de réponse.

```python
{
    'tokenType': 'Bearer',
    'expiresIn': 3599,
    'expiresOn': '2019-12-17 19:47:15.150205',
    'resource': 'https://management.azure.com/',
    'accessToken': 'random-oauth-token',
    'isMRRT': True,
    '_clientId': 'your-client-id',
    '_authority': 'https://login.microsoftonline.com/your-tenant-id'
}
```

Utilisez `token_response["accessToken"]` pour récupérer (fetch) le jeton d’authentification. Consultez la [documentation de l’API REST](https://github.com/microsoft/MLOps/tree/master/examples/AzureML-REST-API) si vous souhaitez voir des exemples d’utilisation du jeton pour effectuer des appels d’API.

#### <a name="java"></a>Java

Dans Java, récupérez le jeton du porteur à l’aide d’un appel REST standard :

```java
String tenantId = "your-tenant-id";
String clientId = "your-client-id";
String clientSecret = "your-client-secret";
String resourceManagerUrl = "https://management.azure.com";

HttpRequest tokenAuthenticationRequest = tokenAuthenticationRequest(tenantId, clientId, clientSecret, resourceManagerUrl);

HttpClient client = HttpClient.newBuilder().build();
Gson gson = new Gson();
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
if (response.statusCode == 200)
{
     body = gson.fromJson(body, AuthenticationBody.class);

    // ... etc ... 
}
// ... etc ...

static HttpRequest tokenAuthenticationRequest(String tenantId, String clientId, String clientSecret, String resourceManagerUrl){
    String authUrl = String.format("https://login.microsoftonline.com/%s/oauth2/token", tenantId);
    String clientIdParam = String.format("client_id=%s", clientId);
    String resourceParam = String.format("resource=%s", resourceManagerUrl);
    String clientSecretParam = String.format("client_secret=%s", clientSecret);

    String bodyString = String.format("grant_type=client_credentials&%s&%s&%s", clientIdParam, resourceParam, clientSecretParam);

    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(authUrl))
            .POST(HttpRequest.BodyPublishers.ofString(bodyString))
            .build();
    return request;
}

class AuthenticationBody {
    String access_token;
    String token_type;
    int expires_in;
    String scope;
    String refresh_token;
    String id_token;
    
    AuthenticationBody() {}
}
```

Le code ci-dessus devrait gérer les exceptions et les codes d’état autres que `200 OK`, mais présente le modèle : 

- Utiliser l’ID et le secret du client pour confirmer que votre programme doit bénéficier de l’accès
- Utiliser votre ID de locataire pour spécifier l’emplacement que doit consulter `login.microsoftonline.com`
- Utiliser Azure Resource Manager comme source du jeton d’autorisation

## <a name="use-managed-identity-authentication"></a>Utiliser l’authentification par identité managée

Pour vous authentifier à l’espace de travail à partir d’une machine virtuelle ou d’un cluster de calcul configuré avec une identité gérée, utilisez la classe `MsiAuthentication` . L’exemple suivant montre comment utiliser cette classe pour s’authentifier à un espace de travail :

```python
from azureml.core.authentication import MsiAuthentication

msi_auth = MsiAuthentication()

ws = Workspace(subscription_id="your-sub-id",
                resource_group="your-resource-group-id",
                workspace_name="your-workspace-name",
                auth=msi_auth
                )
```

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser des secrets dans les l’apprentissage](how-to-use-secrets-in-runs.md).
* [Configurer l’authentification pour des modèles déployés en tant que services web](how-to-authenticate-web-service.md).
* [Utilisez un modèle Azure Machine Learning déployé en tant que service web](how-to-consume-web-service.md).
