---
title: Authentifier les services Azure Batch avec Azure Active Directory
description: Batch prend en charge Azure AD pour l’authentification auprès du service Batch. Apprenez à vous authentifier de deux manières.
ms.topic: how-to
ms.date: 10/20/2020
ms.custom: has-adal-ref
ms.openlocfilehash: 784e92eaa2cd672d511177066befcfd7effc7ca4
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "95252638"
---
# <a name="authenticate-batch-service-solutions-with-active-directory"></a>Authentification de solutions de service Batch avec Active Directory

Azure Batch prend en charge l’authentification avec [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) (Azure AD). Azure AD est le service Microsoft de gestion des répertoires et des identités basé sur le cloud mutualisé. Azure lui-même utilise Azure AD pour authentifier ses clients, ses administrateurs de services fédérés et ses utilisateurs professionnels.

Lorsque vous utilisez l’authentification Azure AD avec Azure Batch, vous pouvez vous authentifier de deux manières :

- À l’aide de l’**authentification intégrée** pour authentifier un utilisateur qui interagit avec l’application. Une application utilisant l’authentification intégrée collecte les informations d’identification d’un utilisateur et les utilise pour authentifier l’accès aux ressources Batch.
- À l’aide d’un **principal de service** pour authentifier une application sans assistance. Un principal de service définit la stratégie et les autorisations pour une application afin de représenter l’application lors de l’accès aux ressources au moment de l’exécution.

Pour en savoir plus sur Azure AD, consultez la [documentation sur Azure Active Directory](../active-directory/index.yml).

## <a name="endpoints-for-authentication"></a>Points de terminaison pour l’authentification

Pour authentifier des applications Batch avec Azure AD, vous devez inclure des points de terminaison connus dans votre code.

### <a name="azure-ad-endpoint"></a>Point de terminaison Azure AD

Le point de terminaison d’autorité Azure AD est :

`https://login.microsoftonline.com/`

Pour s’authentifier avec Azure AD, vous utilisez ce point de terminaison avec l’ID client (ID de répertoire). L’ID client identifie le client Azure AD pour l’authentification. Pour récupérer l’ID client, suivez les étapes décrites dans [Obtenir l’ID client pour Azure Active Directory](#get-the-tenant-id-for-your-active-directory) :

`https://login.microsoftonline.com/<tenant-id>`

> [!NOTE]
> Le point de terminaison spécifique du client est requis lorsque vous vous authentifiez à l’aide d’un principal de service.
>
> Le point de terminaison spécifique du client est facultatif lorsque vous vous authentifiez à l’aide de l’authentification intégrée, mais est recommandé. Vous pouvez toutefois utiliser également le point de terminaison commun Azure AD. Le point de terminaison commun fournit une interface de collecte d’informations d’identification générique lorsqu’un client spécifique n’est pas spécifié. Le point de terminaison commun est `https://login.microsoftonline.com/common`.

Pour plus d’informations sur les points de terminaison de Azure AD, consultez [Authentification et autorisation]()../active-directory/develop/authentication-vs-authorization.md).

### <a name="batch-resource-endpoint"></a>Point de terminaison de ressource Batch

Utilisez le **point de terminaison de ressource Azure Batch** pour obtenir un jeton d’authentification des demandes au service Batch :

`https://batch.core.windows.net/`

## <a name="register-your-application-with-a-tenant"></a>Inscrire votre application avec un client

La première étape d’utilisation d’Azure AD pour l’authentification consiste à inscrire votre application dans un client Azure AD. L’inscription de votre application vous permet d’appeler la [Bibliothèque d’authentification Active Directory](../active-directory/azuread-dev/active-directory-authentication-libraries.md) (ADAL) Azure à partir de votre code. La bibliothèque ADAL fournit une API pour l’authentification avec Azure AD à partir de votre application. L’inscription de votre application est nécessaire si vous prévoyez d’utiliser l’authentification intégrée ou un principal de service.

Lorsque vous inscrivez votre application, vous fournissez des informations sur votre application à Azure AD. Azure AD fournit ensuite un ID d’application (également appelé *ID client*) que vous utilisez pour associer votre application à Azure AD lors de l’exécution. Pour en savoir plus sur l’ID d’application, consultez [Objets application et principal du service dans Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md).

Pour inscrire votre application Batch, suivez les étapes décrites dans la section **Inscrire une application** dans [Démarrage rapide : Inscrire une application avec la plateforme des identités Microsoft](../active-directory/develop/quickstart-register-app.md). Si vous inscrivez votre application en tant qu’application native, vous pouvez spécifier n’importe quel URI valide pour l’**URI de redirection**. Aucun point de terminaison réel n’est nécessaire.

Une fois que vous avez inscrit votre application, vous verrez l’ID d’application :

![Capture d’écran de l’ID d’application affiché dans le portail Azure](./media/batch-aad-auth/app-registration-data-plane.png)

## <a name="get-the-tenant-id-for-your-active-directory"></a>Obtenir l’ID client pour votre Active Directory

L’ID client identifie le client Azure AD qui fournit des services d’authentification à votre application. Pour obtenir l’ID client, procédez comme suit :

1. Dans le portail Azure, sélectionnez votre Active Directory.
1. Sélectionner **Propriétés**.
1. Copiez la valeur GUID fournie pour l’**ID de répertoire**. Cette valeur est également appelée l’ID client.

![Capture d’écran de l’ID de répertoire dans le portail Azure.](./media/batch-aad-auth/aad-directory-id.png)

## <a name="use-integrated-authentication"></a>Utiliser l’authentification intégrée

Pour l’authentification avec l’authentification intégrée, vous devez autoriser votre application à se connecter à l’API de service Batch. Cette étape permet à votre application d’authentifier des appels de l’API de service Batch avec Azure AD.

Une fois que vous avez inscrit votre application, procédez comme suit dans le portail Azure pour lui accorder l’accès au service Batch :

1. Dans le portail Azure, sélectionnez **Tous les services**, puis **Inscriptions des applications**.
1. Recherchez le nom de votre application dans la liste des inscriptions d’application.
1. Sélectionnez l’application, puis sélectionnez **Autorisations de l’API**.
1. Dans la section **API autorisées**, sélectionnez **Ajouter une autorisation**.
1. Dans **Sélectionner une API**, recherchez l’API Batch. Recherchez chacune de ces chaînes, jusqu’à ce que vous trouviez l’API :
    1. **Microsoft Azure Batch**
    1. La chaîne **ddbf3205-c6bd-46ae-8127-60eb93363864** correspond à l’ID de l’API Batch.
1. Une fois que vous avez trouvé l’API Batch, sélectionnez-la, puis choisissez **Sélectionner**.
1. Dans **Sélectionner les autorisations**, cochez la case en regard de **Accéder au service Azure Batch**, puis sélectionnez **Ajouter des autorisations**.

La section **Autorisations des API** indique à présent que votre application Azure AD a accès à Microsoft Graph et à l’API de service Batch. Les autorisations sont accordées à Microsoft Graph automatiquement quand vous inscrivez votre application pour la première fois dans Azure AD.

## <a name="use-a-service-principal"></a>Utiliser un principal de service

Pour authentifier une application qui s’exécute sans assistance, vous utilisez un principal de service. Après avoir inscrit votre application, procédez comme suit dans le portail Azure pour configurer un principal de service :

1. Demandez un secret pour votre application.
1. Attribuer un contrôle d’accès en fonction du rôle Azure (Azure RBAC) à votre application

### <a name="request-a-secret-for-your-application"></a>Demander un secret pour votre application

Quand votre application s’authentifie auprès d’un principal de service, elle envoie l’ID d’application et un secret à Azure AD. Vous devez créer et copier la clé secrète à utiliser à partir de votre code.

Suivez les étapes ci-dessous dans le portail Azure :

1. Dans le portail Azure, choisissez **Tous les services**. Sélectionner **Inscriptions des applications**.
1. Sélectionnez votre application dans la liste des inscriptions d’applications.
1. Sélectionnez l’application, puis sélectionnez **Certificats & secrets**. Dans la section **Secrets client**, sélectionnez **Nouveau secret client**.
1. Pour créer un secret, entrez sa description. Ensuite, sélectionnez une expiration pour le secret : un an, deux ans ou aucune expiration.
1. Sélectionnez **Ajouter** pour créer et afficher le secret. Copiez la valeur du secret dans un endroit sûr, car vous n’y avez plus accès quand vous quittez la page.

### <a name="assign-azure-rbac-to-your-application"></a>Attribuer un contrôle Azure RBAC à votre application

Pour l’authentification auprès d’un principal de service, vous devez attribuer un contrôle Azure RBAC à votre application. Procédez comme suit :

1. Dans le portail Azure, accédez au compte Batch utilisé par votre application.
1. Dans la section **Paramètres** du compte Batch, sélectionnez **Contrôle d’accès (IAM)** .
1. Sélectionnez l’onglet **Attributions de rôles**.
1. Sélectionnez **Ajouter une attribution de rôle**.
1. Dans la liste déroulante **Rôle**, choisissez le rôle *Collaborateur* ou *Lecteur* pour votre application. Pour plus d’informations sur ces rôles, consultez [Prise en main du contrôle Azure d’accès en fonction du rôle dans le portail Azure](../role-based-access-control/overview.md).
1. Dans le champ **Sélectionner**, entrez le nom de votre application. Sélectionnez votre application dans la liste, puis sélectionnez **Enregistrer**.

Votre application doit maintenant apparaître dans vos paramètres de contrôle d’accès avec un rôle Azure qui lui est attribué.

![Attribuer un rôle Azure à votre application](./media/batch-aad-auth/app-rbac-role.png)

### <a name="assign-a-custom-role"></a>Attribuer un rôle personnalisé

Un rôle personnalisé accorde une autorisation précise à un utilisateur pour l’envoi de travaux, de tâches, etc. La possibilité est ainsi donnée d’empêcher les utilisateurs d’effectuer des opérations qui affectent le coût, telles que la création de pools ou la modification de nœuds.

Vous pouvez utiliser un rôle personnalisé pour accorder des autorisations à un groupe, un principal de service ou un utilisateur Azure AD sur les opérations Azure RBAC suivantes :

- Microsoft.Batch/batchAccounts/pools/write
- Microsoft.Batch/batchAccounts/pools/delete
- Microsoft.Batch/batchAccounts/pools/read
- Microsoft.Batch/batchAccounts/jobSchedules/write
- Microsoft.Batch/batchAccounts/jobSchedules/delete
- Microsoft.Batch/batchAccounts/jobSchedules/read
- Microsoft.Batch/batchAccounts/jobs/write
- Microsoft.Batch/batchAccounts/jobs/delete
- Microsoft.Batch/batchAccounts/jobs/read
- Microsoft.Batch/batchAccounts/certificates/write
- Microsoft.Batch/batchAccounts/certificates/delete
- Microsoft.Batch/batchAccounts/certificates/read
- Microsoft.Batch/batchAccounts/read (pour toute opération de lecture)
- Microsoft.Batch/batchAccounts/listKeys/action (pour toute opération)

Les rôles personnalisés sont pour les utilisateurs authentifiés par Azure AD, pas les informations d’identification du compte Batch (clé partagée). Notez que les informations d’identification de compte Batch donnent les autorisations complètes sur le compte Batch. Notez également que les travaux utilisant le [pool automatique](nodes-and-pools.md#autopools) exigent des autorisations au niveau du pool.

> [!NOTE]
> Certaines attributions de rôle doivent être spécifiées dans le champ Action, tandis que d’autres doivent être spécifiées dans le champ DataAction. Pour plus d’informations, consultez [Opérations du fournisseur de ressources Azure](../role-based-access-control/resource-provider-operations.md#microsoftbatch).

Voici un exemple de définition de rôle personnalisé :

```json
{
 "properties":{
    "roleName":"Azure Batch Custom Job Submitter",
    "type":"CustomRole",
    "description":"Allows a user to submit jobs to Azure Batch but not manage pools",
    "assignableScopes":[
      "/subscriptions/88888888-8888-8888-8888-888888888888"
    ],
    "permissions":[
      {
        "actions":[
          "Microsoft.Batch/*/read",
          "Microsoft.Authorization/*/read",
          "Microsoft.Resources/subscriptions/resourceGroups/read",
          "Microsoft.Support/*",
          "Microsoft.Insights/alertRules/*"
        ],
        "notActions":[

        ],
        "dataActions":[
          "Microsoft.Batch/batchAccounts/jobs/*",
          "Microsoft.Batch/batchAccounts/jobSchedules/*"
        ],
        "notDataActions":[

        ]
      }
    ]
  }
}
```

Pour des informations concernant la création d’un rôle personnalisé, consultez [Rôles personnalisés Azure](../role-based-access-control/custom-roles.md).

### <a name="get-the-tenant-id-for-your-azure-active-directory"></a>Obtenir l’ID client pour votre Azure Active Directory

L’ID client identifie le client Azure AD qui fournit des services d’authentification à votre application. Pour obtenir l’ID client, procédez comme suit :

1. Dans le portail Azure, sélectionnez votre Active Directory.
1. Sélectionner **Propriétés**.
1. Copiez la valeur GUID fournie pour l’**ID de répertoire**. Cette valeur est également appelée l’ID client.

![Copier l’ID de répertoire](./media/batch-aad-auth/aad-directory-id.png)

## <a name="code-examples"></a>Exemples de code

Les exemples de code de cette section montrent comment s’authentifier avec Azure AD à l’aide de l’authentification intégrée et d’un principal de service. La plupart de ces exemples de code utilisent .NET, mais les concepts sont similaires pour d’autres langages.

> [!NOTE]
> Un jeton d’authentification Azure AD expire au bout d’une heure. Lorsque vous utilisez un objet **BatchClient** longue durée, nous vous recommandons de récupérer un jeton à partir d’ADAL à chaque demande pour garantir que vous disposez toujours d’un jeton valide.
>
> Pour mettre cela en place dans .NET, écrivez une méthode qui récupère le jeton à partir d’Azure AD et qui transmet cette méthode à un objet **BatchTokenCredentials** en tant que délégué. La méthode du délégué est appelée à chaque demande au service Batch pour garantir qu’un jeton valide est fourni. Par défaut, ADAL met en cache des jetons pour qu’un nouveau jeton soit récupéré à partir d’Azure AD uniquement lorsque cela est nécessaire. Pour plus d’informations sur les jetons dans Azure AD, consultez [Jetons de sécurité](../active-directory/develop/security-tokens.md).

### <a name="code-example-using-azure-ad-integrated-authentication-with-batch-net"></a>Exemple de code : utilisation de l’authentification intégrée Azure AD avec Batch .NET

Pour s’authentifier avec l’authentification intégrée à partir de Batch .NET, vous devez référencer les packages [Azure Batch .NET](https://www.nuget.org/packages/Microsoft.Azure.Batch/) et [ADAL](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/).

Ajoutez les instructions `using` suivantes dans votre code :

```csharp
using Microsoft.Azure.Batch;
using Microsoft.Azure.Batch.Auth;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
```

Référencez le point de terminaison Azure AD dans votre code, y compris l’ID client. Pour récupérer l’ID client, suivez les étapes décrites dans [Obtenir l’ID client pour Azure Active Directory](#get-the-tenant-id-for-your-active-directory) :

```csharp
private const string AuthorityUri = "https://login.microsoftonline.com/<tenant-id>";
```

Référencez le point de terminaison de ressource de service Batch :

```csharp
private const string BatchResourceUri = "https://batch.core.windows.net/";
```

Référencez votre compte Batch :

```csharp
private const string BatchAccountUrl = "https://myaccount.mylocation.batch.azure.com";
```

Spécifiez l’ID d’application (ID client) pour votre application. L’ID d’application est disponible dans votre inscription d’application dans le portail Azure :

```csharp
private const string ClientId = "<application-id>";
```

Copiez également l’URI de redirection que vous avez spécifiée, si vous avez enregistré votre application comme application native. L’URI de redirection spécifié dans votre code doit correspondre à l’URI de redirection que vous avez fourni lors de l’inscription de l’application :

```csharp
private const string RedirectUri = "http://mybatchdatasample";
```

Écrivez une méthode de rappel pour acquérir le jeton d’authentification à partir d’Azure AD. La méthode de rappel **GetAuthenticationTokenAsync** illustrée appelle ici la bibliothèque ADAL pour authentifier un utilisateur qui interagit avec l’application. La méthode **AcquireTokenAsync** fournie par la bibliothèque ADAL demande à l’utilisateur ses informations d’identification, et l’application continue une fois que l’utilisateur les a fournies (sauf si elle dispose déjà d’informations d’identification mises en cache) :

```csharp
public static async Task<string> GetAuthenticationTokenAsync()
{
    var authContext = new AuthenticationContext(AuthorityUri);

    // Acquire the authentication token from Azure AD.
    var authResult = await authContext.AcquireTokenAsync(BatchResourceUri,
                                                        ClientId,
                                                        new Uri(RedirectUri),
                                                        new PlatformParameters(PromptBehavior.Auto));

    return authResult.AccessToken;
}
```

Créez un objet **BatchTokenCredentials** qui prend le délégué comme paramètre. Utilisez ces informations d’identification pour ouvrir un objet **BatchClient**. Vous pouvez utiliser cet objet **BatchClient** pour les opérations suivantes sur le service Batch :

```csharp
public static void PerformBatchOperations()
{
    Func<Task<string>> tokenProvider = () => GetAuthenticationTokenAsync();

    using (var client = BatchClient.Open(new BatchTokenCredentials(BatchAccountUrl, tokenProvider)))
    {
        client.JobOperations.ListJobs();
    }
}
```

### <a name="code-example-using-an-azure-ad-service-principal-with-batch-net"></a>Exemple de code : utilisation d’un principal de service Azure AD avec Batch .NET

Pour s’authentifier avec un principal de service à partir de Batch .NET, vous devez référencer les packages [Azure Batch .NET](https://www.nuget.org/packages/Azure.Batch/) et [ADAL](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/).

Ajoutez les instructions `using` suivantes dans votre code :

```csharp
using Microsoft.Azure.Batch;
using Microsoft.Azure.Batch.Auth;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
```

Référencez le point de terminaison Azure AD dans votre code, y compris l’ID client. Lorsque vous utilisez un principal de service, vous devez indiquer un point de terminaison spécifique du client. Pour récupérer l’ID client, suivez les étapes décrites dans [Obtenir l’ID client pour Azure Active Directory](#get-the-tenant-id-for-your-active-directory) :

```csharp
private const string AuthorityUri = "https://login.microsoftonline.com/<tenant-id>";
```

Référencez le point de terminaison de ressource de service Batch :

```csharp
private const string BatchResourceUri = "https://batch.core.windows.net/";
```

Référencez votre compte Batch :

```csharp
private const string BatchAccountUrl = "https://myaccount.mylocation.batch.azure.com";
```

Spécifiez l’ID d’application (ID client) pour votre application. L’ID d’application est disponible dans votre inscription d’application dans le portail Azure :

```csharp
private const string ClientId = "<application-id>";
```

Spécifiez la clé secrète que vous avez copiée à partir du portail Azure :

```csharp
private const string ClientKey = "<secret-key>";
```

Écrivez une méthode de rappel pour acquérir le jeton d’authentification à partir d’Azure AD. La méthode de rappel **GetAuthenticationTokenAsync** illustrée ici appelle la bibliothèque ADAL pour l’authentification sans assistance :

```csharp
public static async Task<string> GetAuthenticationTokenAsync()
{
    AuthenticationContext authContext = new AuthenticationContext(AuthorityUri);
    AuthenticationResult authResult = await authContext.AcquireTokenAsync(BatchResourceUri, new ClientCredential(ClientId, ClientKey));

    return authResult.AccessToken;
}
```

Créez un objet **BatchTokenCredentials** qui prend le délégué comme paramètre. Utilisez ces informations d’identification pour ouvrir un objet **BatchClient**. Utilisez ensuite cet objet **BatchClient** pour les opérations suivantes sur le service Batch :

```csharp
public static void PerformBatchOperations()
{
    Func<Task<string>> tokenProvider = () => GetAuthenticationTokenAsync();

    using (var client = BatchClient.Open(new BatchTokenCredentials(BatchAccountUrl, tokenProvider)))
    {
        client.JobOperations.ListJobs();
    }
}
```

### <a name="code-example-using-an-azure-ad-service-principal-with-batch-python"></a>Exemple de code : utilisation d’un principal de service Azure AD avec Batch Python

Pour s’authentifier avec un principal de service à partir de Batch Python, installez et référencez les modules [azure-batch](https://pypi.org/project/azure-batch/) et [azure-common](https://pypi.org/project/azure-common/).

```python
from azure.batch import BatchServiceClient
from azure.common.credentials import ServicePrincipalCredentials
```

Lorsque vous utilisez un principal de service, vous devez indiquer un ID d’abonné. Pour récupérer l’ID client, suivez les étapes décrites dans [Obtenir l’ID client pour Azure Active Directory](#get-the-tenant-id-for-your-active-directory) :

```python
TENANT_ID = "<tenant-id>"
```

Référencez le point de terminaison de ressource de service Batch :

```python
RESOURCE = "https://batch.core.windows.net/"
```

Référencez votre compte Batch :

```python
BATCH_ACCOUNT_URL = "https://myaccount.mylocation.batch.azure.com"
```

Spécifiez l’ID d’application (ID client) pour votre application. L’ID d’application est disponible dans votre inscription d’application dans le portail Azure :

```python
CLIENT_ID = "<application-id>"
```

Spécifiez la clé secrète que vous avez copiée à partir du portail Azure :

```python
SECRET = "<secret-key>"
```

Créez un objet **ServicePrincipalCredentials** :

```python
credentials = ServicePrincipalCredentials(
    client_id=CLIENT_ID,
    secret=SECRET,
    tenant=TENANT_ID,
    resource=RESOURCE
)
```

Utilisez les informations d’identification du principal de service pour ouvrir un objet **BatchServiceClient**. Utilisez ensuite cet objet **BatchServiceClient** pour les opérations suivantes sur le service Batch.

```python
    batch_client = BatchServiceClient(
    credentials,
    batch_url=BATCH_ACCOUNT_URL
)
```

## <a name="next-steps"></a>Étapes suivantes

- Consultez la [documentation Azure Active Directory](../active-directory/index.yml). Des exemples détaillés illustrant l’utilisation d’ADAL sont disponibles dans la bibliothèque [Exemples de code Azure](https://azure.microsoft.com/resources/samples/?service=active-directory).
- En savoir plus sur les [Objets application et principal du service dans Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md) et [Comment créer une application et un principal Azure AD pouvant accéder aux ressources](../active-directory/develop/howto-create-service-principal-portal.md).
- En savoir plus sur [l’authentification de solutions de gestion Batch avec Active Directory](batch-aad-auth-management.md).
- Pour obtenir un exemple Python sur le mode de création d’un client Batch authentifié à l’aide d’un jeton Azure AD, consultez l’exemple [Déploiement d’une image Azure Batch personnalisée avec un script Python](https://github.com/azurebigcompute/recipes/blob/master/Azure%20Batch/CustomImages/CustomImagePython.md).

