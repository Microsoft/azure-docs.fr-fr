---
title: Applications clientes publiques monocomptes et multicomptes | Azure
description: Vue d’ensemble des applications clientes publiques monocomptes et multicomptes.
services: active-directory
author: shoatman
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/26/2019
ms.author: shoatman
ms.custom: aaddev
ms.reviewer: shoatman
ms.openlocfilehash: 89a383aabf3487a0938604bc28ddb06c0541d13e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80881330"
---
# <a name="single-and-multiple-account-public-client-apps"></a>Applications clientes publiques monocomptes et multicomptes

Cet article va vous aider à comprendre les types utilisés dans les applications clientes publiques monocomptes et multicomptes, puis va s’intéresser plus particulièrement aux applications clientes publiques monocomptes. 

La bibliothèque d’authentification Active Directory (ADAL) Azure modélise le serveur.  En revanche, Microsoft Authentication Library (MSAL) modélise plutôt votre application cliente.  La majorité des applications Android sont considérées comme des clients publics. Un client public est une application qui ne peut pas conserver un secret de façon sécurisée.  

MSAL spécialise la surface d’API de `PublicClientApplication` pour simplifier et clarifier l’expérience de développement des applications qui autorisent l’utilisation d’un seul compte à la fois. `PublicClientApplication` est sous-classé par `SingleAccountPublicClientApplication` et `MultipleAccountPublicClientApplication`.  Le diagramme suivant montre la relation entre ces classes.

![Diagramme des classes UML SingleAccountPublicClientApplication](./media/single-multi-account/single-and-multiple-account.png)

## <a name="single-account-public-client-application"></a>Application cliente publique monocompte

La classe `SingleAccountPublicClientApplication` vous permet de créer une application MSAL qui autorise la connexion d’un seul compte à la fois. `SingleAccountPublicClientApplication` diffère de `PublicClientApplication` des manières suivantes :

- MSAL suit le compte actuellement connecté.
  - Si votre application utilise un répartiteur (par défaut lors de l’inscription d’application sur le portail Azure) et qu’elle est installée sur un appareil sur lequel un répartiteur est présent, MSAL vérifie que le compte est toujours disponible sur l’appareil.
- `signIn` vous permet de connecter un compte explicitement et indépendamment de la demande d’étendues.
- `acquireTokenSilent` n’exige pas de paramètre de compte.  Si vous fournissez un compte et si le compte que vous fournissez ne correspond pas au compte actuellement suivi par MSAL, une exception `MsalClientException` est levée.
- `acquireToken` ne permet pas à l’utilisateur de changer de compte. Si l’utilisateur essaie de passer à un autre compte, une exception est levée.
- `getCurrentAccount` retourne un objet de résultat qui fournit ce qui suit :
  - Une valeur booléenne indiquant si le compte a changé. Vous pouvez changer de compte suite à sa suppression de l’appareil, par exemple.
  - Le compte précédent. Cette information s’avère utile si vous avez besoin d’effectuer un nettoyage des données locales quand le compte est supprimé de l’appareil ou quand un nouveau compte est connecté.
  - Le compte actuel.
- `signOut` supprime de l’appareil tous les jetons associés à votre client.  

Quand un répartiteur d’authentification Android, comme Microsoft Authenticator ou Portail d’entreprise Intune est installé sur l’appareil et que votre application est configurée pour utiliser le répartiteur, `signOut` ne supprime pas le compte de l’appareil.

## <a name="single-account-scenario"></a>Scénario monocompte

Le pseudo-code suivant illustre l’utilisation de `SingleAccountPublicClientApplication`.

```java
// Construct Single Account Public Client Application
ISingleAccountPublicClientApplication app = PublicClientApplication.createSingleAccountPublicClientApplication(getApplicationContext(), R.raw.msal_config);

String[] scopes = {"User.Read"};
IAccount mAccount = null;

// Acquire a token interactively
// The user will get a UI prompt before getting the token.
app.signIn(getActivity(), scopes, new AuthenticationCallback()
{

        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) 
        {
            mAccount = authenticationResult.getAccount();
        }

        @Override
        public void onError(MsalException exception)
        {
        }

        @Override
        public void onCancel()
        {
        }
    }
);

// Load Account Specific Data
getDataForAccount(account);

// Get Current Account
ICurrentAccountResult currentAccountResult = app.getCurrentAccount();
if (currentAccountResult.didAccountChange())
{
    // Account Changed Clear existing account data
    clearDataForAccount(currentAccountResult.getPriorAccount());
    mAccount = currentAccountResult.getCurrentAccount();
    if (account != null)
    {
        //load data for new account
        getDataForAccount(account);
    }
}

// Sign out
if (app.signOut())
{
    clearDataForAccount(mAccount);
    mAccount = null;
}
```

## <a name="multiple-account-public-client-application"></a>Application cliente publique multicompte

La classe `MultipleAccountPublicClientApplication` est utilisée pour créer des applications MSAL qui autorisent plusieurs comptes à se connecter en même temps. Elle vous permet d’obtenir, d’ajouter et de supprimer des comptes comme suit :

### <a name="add-an-account"></a>Ajouter un compte

Utilisez un ou plusieurs comptes dans votre application en appelant `acquireToken` une ou plusieurs fois.  

### <a name="get-accounts"></a>Obtenir des comptes

- Appelez `getAccount` pour obtenir un compte spécifique.
- Appelez `getAccounts` pour obtenir la liste des comptes actuellement connus de l’application.

Votre application ne sera pas en mesure d’énumérer tous les comptes de plateforme d’identités Microsoft présents sur l’appareil et connus de l’application de répartiteur. Elle peut uniquement énumérer les comptes qui ont été utilisés par votre application.  Les comptes qui ont été supprimés de l’appareil ne sont pas retournés par ces fonctions.

### <a name="remove-an-account"></a>Supprimer un compte

Pour supprimer un compte, appelez `removeAccount` avec un identificateur de compte.

Si votre application est configurée pour utiliser un répartiteur et qu’un répartiteur est installé sur l’appareil, le compte n’est pas supprimé du répartiteur quand vous appelez `removeAccount`.  Seuls les jetons associés à votre client sont supprimés.

## <a name="multiple-account-scenario"></a>Scénario multicompte

Le pseudo-code suivant montre comment créer une application multicompte, lister les comptes présents sur l’appareil et acquérir des jetons.

```java
// Construct Multiple Account Public Client Application
IMultipleAccountPublicClientApplication app = PublicClientApplication.createMultipleAccountPublicClientApplication(getApplicationContext(), R.raw.msal_config);

String[] scopes = {"User.Read"};
IAccount mAccount = null;

// Acquire a token interactively
// The user will be required to interact with a UI to obtain a token
app.acquireToken(getActivity(), scopes, new AuthenticationCallback()
 {

        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) 
        {
            mAccount = authenticationResult.getAccount();
        }

        @Override
        public void onError(MsalException exception)
        {
        }

        @Override
        public void onCancel()
        {
        }
 });

...

// Get the default authority
String authority = app.getConfiguration().getDefaultAuthority().getAuthorityURL().toString();

// Get a list of accounts on the device
List<IAccount> accounts = app.getAccounts();

// Pick an account to obtain a token from without prompting the user to sign in
IAccount selectedAccount = accounts.get(0);

// Get a token without prompting the user
app.acquireTokenSilentAsync(scopes, selectedAccount, authority, new SilentAuthenticationCallback()
{

        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) 
        {
            mAccount = authenticationResult.getAccount();
        }

        @Override
        public void onError(MsalException exception)
        {
        }
});
```
