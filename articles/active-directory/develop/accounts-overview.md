---
title: Comptes de plateforme d’identités Microsoft et profils de locataires sur Android | Azure
description: Vue d'ensemble des comptes de la plate-forme d'identité Microsoft pour Android
services: active-directory
author: shoatman
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.devlang: java
ms.date: 09/14/2019
ms.author: shoatman
ms.custom: aaddev, devx-track-java
ms.reviewer: shoatman
ms.openlocfilehash: 404ffbc09a69b623a421bd0c01550d72e5c03158
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88115983"
---
# <a name="accounts--tenant-profiles-android"></a>Comptes et profils de locataire (Android)

Cet article donne un aperçu de ce que représente `account` dans la plate-forme d'identité Microsoft.

L'API Microsoft Authentication Library (MSAL) remplace le terme *user (utilisateur)* par le terme *account (compte)* . L'une des raisons est qu'un utilisateur (agent humain ou logiciel) peut avoir, ou utiliser, plusieurs comptes. Ces comptes peuvent appartenir à la propre organisation de l'utilisateur et/ou à d'autres organisations dont l'utilisateur est membre.

Dans la plate-forme d'identité Microsoft, un compte comprend les éléments suivants :

- Un identificateur unique.  
- Une ou plusieurs informations d'identification prouvant la propriété ou le contrôle du compte.
- Un ou plusieurs profils constitués d'attributs tels que :
  - Photo, prénom, nom de famille, poste, bureau
- Un compte comporte une source d'autorité ou un système d'enregistrement. Il s'agit du système où le compte est créé et où sont stockées les informations d'identification associées à ce compte. Dans les systèmes multilocataires comme la plate-forme d'identité Microsoft, le système d'enregistrement correspond au `tenant` où le compte a été créé. Ce locataire est également appelé `home tenant`.
- Dans la plate-forme d'identité Microsoft, les comptes comportent les systèmes d'enregistrement suivants :
  - Azure Active Directory, y compris Azure Active Directory B2C.
  - Compte Microsoft (Live).
- Les comptes de systèmes d'enregistrement en dehors de la plate-forme d'identité Microsoft sont représentés dans la plate-forme d'identité Microsoft, notamment :
  - les identités de répertoires locaux connectés (Windows Server Active Directory)
  - les identités externes de LinkedIn, GitHub, etc.
  Dans ces cas, un compte dispose à la fois d'un système d'enregistrement d'origine et d'un système d'enregistrement intégré à la plate-forme d'identité Microsoft.
- La plate-forme d'identité Microsoft permet d'utiliser un compte pour accéder aux ressources appartenant à plusieurs organisations (locataires Azure Active Directory).
  - Pour enregistrer le fait qu’un compte d'un système d'enregistrement (locataire AAD A) a accès à une ressource dans un autre système d'enregistrement (locataire AAD B), le compte doit être représenté dans le locataire où la ressource est définie. Pour cela, vous créez un enregistrement local du compte à partir du système A dans le système B.
  - Cet enregistrement local, c'est-à-dire la représentation du compte, est lié au compte original.
  - MSAL expose cet enregistrement local comme un `Tenant Profile`.
  - Le profil du locataire peut avoir différents attributs adaptés au contexte local, comme le poste, le bureau, les coordonnées du contact, etc.
- Étant donné qu’un compte peut être présent dans un ou plusieurs locataires, il peut avoir plusieurs profils.

> [!NOTE]
> MSAL traite le système de comptes Microsoft (Live, MSA) comme un autre locataire au sein de la plate-forme d'identité Microsoft. L'ID de locataire du locataire du compte Microsoft est : `9188040d-6c67-4c5b-b112-36a304b66dad`

## <a name="account-overview-diagram"></a>Diagramme de vue d'ensemble des comptes

![Diagramme de vue d'ensemble des comptes](./media/accounts-overview/accounts-overview.svg)

Dans le diagramme ci-dessus :

- Le compte `bob@contoso.com` est créé dans le système Windows Server Active Directory local (système d'enregistrement local d'origine).
- Le compte `tom@live.com` est créé dans le locataire du compte Microsoft.
- `bob@contoso.com` a accès à au moins une ressource des locataires Azure Active Directory suivants :
  - contoso.com (système d'enregistrement dans le cloud - lié au système d'enregistrement local)
  - fabrikam.com
  - woodgrovebank.com
  - Un profil de locataire pour `bob@contoso.com` existe dans chacun de ces locataires.
- `tom@live.com` a accès aux ressources des locataires Microsoft suivants :
  - contoso.com
  - fabrikam.com
  - Un profil de locataire pour `tom@live.com` existe dans chacun de ces locataires.
- Les informations sur Tom et Bob dans d'autres locataires peuvent différer de celles figurant dans le système d'enregistrement. Elles peuvent différer au niveau de certains attributs tels que le poste, le bureau, etc. Ces personnes peuvent être membres de groupes et/ou de rôles au sein de chaque organisation (locataire Azure Active Directory). Ces informations sont référencées sous le terme de profil de locataire bob@contoso.com.

Dans le diagramme, bob@contoso.com et tom@live.com ont accès aux ressources de différents locataires Azure Active Directory. Pour plus d’informations, consultez [Ajouter des utilisateurs Azure Active Directory B2B Collaboration dans le portail Azure](../external-identities/add-users-administrator.md).

## <a name="accounts-and-single-sign-on-sso"></a>Comptes et authentification unique (SSO)

Le cache de jeton MSAL stocke un seul *jeton d'actualisation* par compte. Ce jeton d’actualisation peut être utilisé pour demander silencieusement des jetons d'accès à plusieurs locataires de la plate-forme d'identité Microsoft. Lorsqu'un répartiteur est installé sur un appareil, le compte est géré par ce répartiteur, et l'authentification unique au niveau de l'appareil devient possible.

> [!IMPORTANT]
> Le comportement d’un compte Business to Consumer (B2C) et d’un jeton d'actualisation diffère du reste de la plate-forme d'identité Microsoft. Pour plus d’informations, voir [Stratégies et comptes B2C](#b2c-policies--accounts).

## <a name="account-identifiers"></a>Identificateurs de compte

L'ID de compte MSAL n'est pas un ID d'objet de compte. Il n'est pas destiné à être analysé et/ou utilisé pour transmettre autre chose que l'unicité au sein de la plate-forme d'identité Microsoft.

Pour garantir la compatibilité avec la bibliothèque Azure AD Authentication Library (ADAL) et faciliter la migration d'ADAL vers MSAL, MSAL peut rechercher des comptes en utilisant n'importe quel identifiant valide pour le compte disponible dans le cache MSAL.  Par exemple, la méthode suivante récupérera toujours le même objet de compte pour tom@live.com car chacun des identifiants est valide :

```java
// The following would always retrieve the same account object for tom@live.com because each identifier is valid

IAccount account = app.getAccount("<tome@live.com msal account id>");
IAccount account = app.getAccount("<tom@live.com contoso user object id>");
IAccount account = app.getAccount("<tom@live.com woodgrovebank user object id>");
```

## <a name="accessing-claims-about-an-account"></a>Accès à des revendications concernant un compte

En plus de demander un code d'accès, MSAL demande toujours un code d'identification à chaque locataire. Pour cela, il demande toujours les étendues suivantes :

- openid
- profile

Le jeton d'ID contient une liste de revendications. `Claims` représentent des paires nom/valeur sur le compte, et sont utilisées pour faire la demande.

Comme nous l'avons mentionné précédemment, chaque locataire qui possède un compte peut stocker différentes informations sur le compte, y compris, mais sans s'y limiter, des attributs tels que le poste, le bureau, etc.

Bien qu'un compte puisse être membre ou invité de plusieurs organisations, MSAL n'interroge pas un service pour obtenir une liste des locataires dont le compte est membre. Au lieu de cela, MSAL dresse une liste des locataires dans lesquels le compte est présent, en réponse aux demandes de jeton qui ont été faites.

Les revendications exposées sur l'objet du compte sont toujours les revendications de 'home tenant'/{authority} pour un compte. Si ce compte n'a pas été utilisé afin de demander un jeton pour son locataire de base, MSAL ne peut pas fournir de réclamations via l'objet du compte.  Par exemple :

```java
// Psuedo Code
IAccount account = getAccount("accountid");

String username = account.getClaims().get("preferred_username");
String tenantId = account.getClaims().get("tid"); // tenant id
String objectId = account.getClaims().get("oid"); // object id
String issuer = account.getClaims().get("iss"); // The tenant specific authority that issued the id_token
```

> [!TIP]
> Pour afficher une liste des réclamations disponibles à partir de l'objet du compte, référez-vous à la section [Revendications dans un jeton id_token](./id-tokens.md#claims-in-an-id_token)

> [!TIP]
> Pour inclure d’autres revendications dans votre id_token, reportez-vous à la documentation sur les revendications facultatives dans [Procédure  : Fournir des revendications facultatives à votre application Azure AD](./active-directory-optional-claims.md)

### <a name="access-tenant-profile-claims"></a>Accéder aux revendications d’un profil de locataire

Pour accéder aux réclamations concernant un compte, telles qu'elles apparaissent dans d'autres locataires, vous devez d'abord définir l'objet de votre compte sur `IMultiTenantAccount`. Tous les comptes peuvent être multilocataires, mais le nombre de profils de locataires disponibles via MSAL est basé sur les locataires pour lesquels les jetons utiliseront le compte actuel.  Par exemple :

```java
// Psuedo Code
IAccount account = getAccount("accountid");
IMultiTenantAccount multiTenantAccount = (IMultiTenantAccount)account;

multiTenantAccount.getTenantProfiles().get("tenantid for fabrikam").getClaims().get("family_name");
multiTenantAccount.getTenantProfiles().get("tenantid for contoso").getClaims().get("family_name");
```

## <a name="b2c-policies--accounts"></a>Stratégies et comptes B2C

Les jetons d’actualisation pour un compte ne sont pas partagés entre les stratégies B2C. Par conséquent, l'authentification unique à l'aide de jetons n'est pas possible. Cela ne signifie pas pour autant qu'une authentification unique n'est pas possible. Cela signifie que l'authentification unique doit utiliser une expérience interactive dans laquelle un cookie est disponible pour permettre l'authentification unique.

Cela signifie également que dans le cas de MSAL, si vous acquérez des jetons en utilisant différentes polices B2C, ceux-ci sont traités comme des comptes séparés, chacun avec son propre identifiant. Si vous voulez utiliser un compte pour demander un jeton en utilisant `acquireTokenSilent`, vous devrez sélectionner le compte dans la liste des comptes correspondant à la stratégie que vous utilisez avec la demande de jeton. Par exemple :

```java
// Get Account For Policy

String policyId = "SignIn";
IAccount signInPolicyAccount = getAccountForPolicyId(app, policyId);

private IAccount getAccountForPolicy(IPublicClientApplication app, String policyId)
{
    List<IAccount> accounts = app.getAccounts();

    foreach(IAccount account : accounts)
   {
        if (account.getClaims().get("tfp").equals(policyId))
        {
            return account;
        }
    }

    return null;
}
```