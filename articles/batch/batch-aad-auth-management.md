---
title: Utiliser Azure Active Directory pour authentifier des solutions Batch Management
description: Découvrez comment utiliser Azure Active Directory pour l’authentification à partir d’applications qui utilisent la bibliothèque Batch Management .NET.
ms.topic: how-to
ms.date: 04/27/2017
ms.custom: has-adal-ref, devx-track-csharp
ms.openlocfilehash: ff49d5e88df7c56ed4dee0685f09e45eb372aa5c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88930211"
---
# <a name="authenticate-batch-management-solutions-with-active-directory"></a>Authentification de solutions de gestion Batch avec Active Directory

Les applications qui appellent le service Azure Batch Management s’authentifient auprès d’[Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) (Azure AD). Azure AD est le service Microsoft de gestion des répertoires et des identités basé sur le cloud mutualisé. Azure lui-même utilise Azure AD pour l’authentification de ses clients, de ses administrateurs de services fédérés et de ses utilisateurs professionnels.

La bibliothèque .NET de gestion Batch expose des types pour l’utilisation des packages d’application, des applications, des clés de compte et des comptes Batch. La bibliothèque Batch Management .NET est un client de fournisseur de ressources Azure, utilisée avec [Azure Resource Manager](../azure-resource-manager/management/overview.md) pour gérer ces ressources programmatiquement. Azure AD est nécessaire pour authentifier les requêtes effectuées via les clients de fournisseur de ressources Azure, dont la bibliothèque Batch Management .NET, et via Azure Resource Manager.

Dans cet article, nous découvrons comment utiliser Azure AD pour l’authentification à partir des applications qui utilisent la bibliothèque .NET de gestion Batch. Nous montrons comment utiliser Azure AD pour authentifier un administrateur ou un coadministrateur d’abonnement avec l’authentification intégrée. Nous utilisons l’exemple de projet [AccountManagement](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/AccountManagement), disponible sur GitHub, pour parcourir la bibliothèque Batch Management .NET avec Azure AD.

Pour en savoir plus sur l’utilisation de la bibliothèque .NET de gestion Batch et l’exemple AccountManagement, consultez [Gérer les quotas et comptes Batch avec la bibliothèque cliente Batch Management pour .NET](batch-management-dotnet.md).

## <a name="register-your-application-with-azure-ad"></a>Inscrire votre application auprès d’Azure AD

La [bibliothèque d’authentification Azure Active Directory](../active-directory/azuread-dev/active-directory-authentication-libraries.md) (ADAL) fournit une interface de programmation à Azure AD, que vous pouvez utiliser dans vos applications. Pour appeler ADAL à partir de votre application, vous devez inscrire votre application dans un locataire Azure AD. Lorsque vous inscrivez votre application, vous fournissez des informations relatives à votre application à Azure AD, y compris un nom au sein du locataire Azure AD. Azure AD fournit ensuite un ID d’application que vous utilisez pour associer votre application à Azure AD lors de l’exécution. Pour en savoir plus sur l’ID d’application, consultez [Objets application et principal du service dans Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md).

Suivez les étapes de la section [Ajout d’une application](../active-directory/develop/quickstart-register-app.md) dans [Intégration d’applications dans Azure Active Directory](../active-directory/develop/quickstart-register-app.md) pour inscrire l’exemple d’application AccountManagement. Indiquez **Application cliente native** comme type d’application. L’URI conforme au standard OAuth 2.0 de l’**URI de redirection** est `urn:ietf:wg:oauth:2.0:oob`. Pour `http://myaccountmanagementsample`l’URI de redirection **, vous pouvez toutefois spécifier n’importe quel URI valide (tel que** ), puisqu’il n’est pas nécessaire que ce soit un point de terminaison réel.

![Ajout d'une application](./media/batch-aad-auth-management/app-registration-management-plane.png)

Lorsque vous avez terminé le processus d’inscription, l’ID d’application et l’ID d’objet (principal de service) sont répertoriés pour votre application.

![Processus d’inscription terminé](./media/batch-aad-auth-management/app-registration-client-id.png)

## <a name="grant-the-azure-resource-manager-api-access-to-your-application"></a>Accorder l’accès à l’API Azure Resource Manager à votre application

Ensuite, vous devez déléguer l’accès à votre application à l’API Azure Resource Manager. L’identificateur Azure AD pour l’API Resource Manager est **API Gestion des services Windows Azure**.

Suivez les étapes ci-dessous dans le portail Azure :

1. Dans le volet de navigation de gauche du portail Azure, choisissez **Tous les services**, cliquez sur **Inscriptions des applications**, puis cliquez sur **Ajouter**.
2. Recherchez le nom de votre application dans la liste des inscriptions d’application :

    ![Rechercher le nom de votre application](./media/batch-aad-auth-management/search-app-registration.png)

3. Affichez le panneau **Paramètres**. Dans la section **Accès API**, sélectionnez **Autorisations requises**.
4. Cliquez sur **Ajouter** pour ajouter une autorisation requise.
5. À l’étape 1, entrez **API Gestion des services Windows Azure**, sélectionnez cette API dans la liste des résultats, puis cliquez sur le bouton **Sélectionner**.
6. À l’étape 2, activez la case à cocher en regard de **Access Azure classic deployment model as organization users** (Accéder au modèle de déploiement Azure Classic en tant qu’utilisateurs de l’organisation), puis cliquez sur le bouton **Sélectionner**.
7. Cliquez sur le bouton **Terminé**.

Le panneau **Autorisations requises** indique à présent que les autorisations pour votre application sont accordées aux API Resource Manager et à ADAL. Les autorisations sont accordées par défaut à ADAL lorsque vous commencez par inscrire votre application auprès d’Azure AD.

![Déléguer des autorisations à l’API Azure Resource Manager](./media/batch-aad-auth-management/required-permissions-management-plane.png)

## <a name="azure-ad-endpoints"></a>Points de terminaison Azure AD

Pour authentifier vos solutions de gestion Batch avec Azure AD, vous devez inclure 2 points de terminaison connus.

- Le **point de terminaison commun Azure AD** fournit une interface de collecte d’informations d’identification générique lorsqu’un client spécifique n’est pas spécifié, comme dans le cas de l’authentification intégrée :

    `https://login.microsoftonline.com/common`

- Le **point de terminaison Azure Resource Manager** permet d’obtenir un jeton d’authentification des demandes transmises au service de gestion Batch :

    `https://management.core.windows.net/`

L’exemple d’application AccountManagement définit des constantes pour ces points de terminaison. Laissez ces constantes telles quelles :

```csharp
// Azure Active Directory "common" endpoint.
private const string AuthorityUri = "https://login.microsoftonline.com/common";
// Azure Resource Manager endpoint
private const string ResourceUri = "https://management.core.windows.net/";
```

## <a name="reference-your-application-id"></a>Référencer votre ID d’application

Votre application cliente utilise l’ID d’application (également appelé l’ID client) pour accéder à Azure AD lors de l’exécution. Une fois que vous avez inscrit votre application dans le portail Azure, mettez à jour votre code pour utiliser l’ID d’application fourni par Azure AD pour votre application inscrite. Dans l’exemple d’application AccountManagement, copiez votre ID d’application à partir du portail Azure vers la constante appropriée :

```csharp
// Specify the unique identifier (the "Client ID") for your application. This is required so that your
// native client application (i.e. this sample) can access the Microsoft Graph API. For information
// about registering an application in Azure Active Directory, please see "Register an application with the Microsoft identity platform" here:
// https://docs.microsoft.com/azure/active-directory/develop/quickstart-register-app
private const string ClientId = "<application-id>";
```
Copiez également l’URI de redirection que vous avez spécifiée pendant l’inscription. L’URI de redirection spécifié dans votre code doit correspondre à l’URI de redirection que vous avez fourni lors de l’inscription de l’application.

```csharp
// The URI to which Azure AD will redirect in response to an OAuth 2.0 request. This value is
// specified by you when you register an application with AAD (see ClientId comment). It does not
// need to be a real endpoint, but must be a valid URI (e.g. https://accountmgmtsampleapp).
private const string RedirectUri = "http://myaccountmanagementsample";
```

## <a name="acquire-an-azure-ad-authentication-token"></a>Acquérir un jeton d’authentification Azure AD

Après avoir inscrit l’exemple AccountManagement auprès du locataire Azure AD et mis à jour le code source de l’exemple avec vos valeurs, l’exemple est prêt pour une authentification avec Azure AD. Lorsque vous exécutez l’exemple, ADAL tente d’acquérir un jeton d’authentification. À ce stade, vous êtes invité à renseigner vos informations d’identification Microsoft :

```csharp
// Obtain an access token using the "common" AAD resource. This allows the application
// to query AAD for information that lies outside the application's tenant (such as for
// querying subscription information in your Azure account).
AuthenticationContext authContext = new AuthenticationContext(AuthorityUri);
AuthenticationResult authResult = authContext.AcquireToken(ResourceUri,
                                                        ClientId,
                                                        new Uri(RedirectUri),
                                                        PromptBehavior.Auto);
```

Après avoir fourni vos informations d’identification, l’exemple d’application peut continuer à émettre des demandes authentifiées au service de gestion Batch.

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur l’exécution de l’[exemple d’application AccountManagement](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/AccountManagement), consultez [Gérer les quotas et comptes Batch avec la bibliothèque cliente Batch Management pour .NET](batch-management-dotnet.md).
- Pour en savoir plus sur Azure AD, consultez la [documentation sur Azure Active Directory](../active-directory/index.yml).
- Des exemples détaillés illustrant l’utilisation d’ADAL sont disponibles dans la bibliothèque [Exemples de code Azure](https://azure.microsoft.com/resources/samples/?service=active-directory).
- Pour authentifier des applications de service Batch à l’aide d’Azure AD, consultez [Authentifier des solutions de service Batch auprès d’Active Directory](batch-aad-auth.md).
