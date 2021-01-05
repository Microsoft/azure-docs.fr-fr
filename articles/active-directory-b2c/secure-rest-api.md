---
title: Sécuriser un service RESTful dans votre environnement Azure AD B2C
titleSuffix: Azure AD B2C
description: Sécurisez vos échanges de revendications d’API REST personnalisées dans votre environnement Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 10/15/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: f6907db7f6e53247a8f2fc0042e8c8e6b081dbd3
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516378"
---
# <a name="secure-your-restful-services"></a>Sécuriser vos services API RESTful 

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Lors de l’intégration d’une API REST dans un parcours utilisateur Azure AD B2C, vous devez protéger votre point de terminaison d’API REST en recourant à l’authentification. Cela garantit que seuls les services possédant des informations d’identification correctes, telles que Azure AD B2C, peuvent effectuer des appels à votre point de terminaison d’API REST.

Lisez les articles [Valider l’entrée utilisateur](custom-policy-rest-api-claims-validation.md) et [Ajouter des échanges de revendications d’API REST aux stratégies personnalisées](custom-policy-rest-api-claims-exchange.md) pour découvrir comment intégrer une API REST à votre parcours utilisateur Azure AD B2C.

Cet article montre comment sécuriser votre API REST avec l’authentification HTTP de base, l’authentification par certificat client ou l’authentification OAuth2. 

## <a name="prerequisites"></a>Prérequis

Effectuez les procédures de l’un des guides suivants :

- [Intégrer les échanges de revendications de l’API REST dans votre parcours utilisateur Azure AD B2C pour valider une entrée d’utilisateur](custom-policy-rest-api-claims-validation.md)
- [Ajouter des échanges de revendications de l’API REST](custom-policy-rest-api-claims-exchange.md)

## <a name="http-basic-authentication"></a>Authentification HTTP de base

L’authentification de base HTTP est définie dans le document [RFC 2617](https://tools.ietf.org/html/rfc2617). L’authentification de base fonctionne comme suit : Azure AD B2C envoie une requête HTTP avec les informations d’identification du client dans l’en-tête Authorization. Les informations d’identification sont mises en forme en tant que chaîne « nom : mot de passe » codée en base64.  

### <a name="add-rest-api-username-and-password-policy-keys"></a>Ajouter les clés de stratégie de mot de passe et nom d’utilisateur de l’API REST

Pour configurer un profil technique API REST avec l’authentification HTTP de base, créez les clés de chiffrement suivantes pour stocker le nom d’utilisateur et le mot de passe :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Veillez à bien utiliser l’annuaire qui contient votre locataire Azure AD B2C. Sélectionnez le filtre **Annuaire + abonnement** dans le menu du haut, puis choisissez votre annuaire Azure AD B2C.
1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
1. Dans la page de vue d’ensemble, sélectionnez **Infrastructure d’expérience d’identité**.
1. Sélectionnez **Clés de stratégie**, puis **Ajouter**.
1. Pour **Options**, sélectionnez **Manuel**.
1. Dans **Nom**, indiquez **RestApiUsername**.
    Il est possible que le préfixe *B2C_1A_* soit ajouté automatiquement.
1. Dans la zone **Secret**, entrez le nom d’utilisateur de l’API REST.
1. Pour **Utilisation de la clé**, sélectionnez **Chiffrement**.
1. Sélectionnez **Create** (Créer).
1. Sélectionnez de nouveau **Clés de stratégie**.
1. Sélectionnez **Ajouter**.
1. Pour **Options**, sélectionnez **Manuel**.
1. Dans **Nom**, indiquez **RestApiPassword**.
    Il est possible que le préfixe *B2C_1A_* soit ajouté automatiquement.
1. Dans la zone **Secret**, entrez le mot de passe de l’API REST.
1. Pour **Utilisation de la clé**, sélectionnez **Chiffrement**.
1. Sélectionnez **Create** (Créer).

### <a name="configure-your-rest-api-technical-profile-to-use-http-basic-authentication"></a>Configurer votre profil technique d’API REST afin d’utiliser l’authentification de base HTTP

Après avoir créé la clé nécessaire, configurez vos métadonnées du profil technique de l’API REST pour référencer les informations d’identification.

1. Dans votre répertoire de travail, ouvrez le fichier de stratégie d’extension (TrustFrameworkExtensions.xml).
1. Recherchez le profil technique API REST. Par exemple, `REST-ValidateProfile` ou `REST-GetProfile`.
1. Recherchez l’élément `<Metadata>`.
1. Définissez *AuthenticationType* sur `Basic`.
1. Définissez *AllowInsecureAuthInProduction* sur `false`.
1. Immédiatement après l’élément `</Metadata>` fermant, ajoutez l’extrait de code XML suivant :
    ```xml
    <CryptographicKeys>
        <Key Id="BasicAuthenticationUsername" StorageReferenceId="B2C_1A_RestApiUsername" />
        <Key Id="BasicAuthenticationPassword" StorageReferenceId="B2C_1A_RestApiPassword" />
    </CryptographicKeys>
    ```

Voici un exemple de profil technique RESTful configuré avec l’authentification de base HTTP :

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-GetProfile">
      <DisplayName>Get user extended profile Azure Function web hook</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://your-account.azurewebsites.net/api/GetProfile?code=your-code</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <Item Key="AuthenticationType">Basic</Item>
        <Item Key="AllowInsecureAuthInProduction">false</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="BasicAuthenticationUsername" StorageReferenceId="B2C_1A_RestApiUsername" />
        <Key Id="BasicAuthenticationPassword" StorageReferenceId="B2C_1A_RestApiPassword" />
      </CryptographicKeys>
      ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="https-client-certificate-authentication"></a>Authentification par certificat client HTTPS

L’authentification par certificat client est une authentification mutuelle basée sur un certificat dans laquelle le client, Azure AD B2C, fournit son certificat client au serveur pour prouver son identité. Cela se produit dans le cadre de la négociation SSL. Seuls les services qui ont des certificats appropriés, tels qu’Azure AD B2C, peuvent accéder à votre service API REST. Le certificat client est un certificat numérique X.509. Dans les environnements de production, il doit être signé par une autorité de certification.

### <a name="prepare-a-self-signed-certificate-optional"></a>Préparer un certificat auto-signé (facultatif)

Pour les environnements hors production, si vous n’avez pas encore de certificat, vous pouvez utiliser un certificat auto-signé. Sur Windows, vous pouvez utiliser l’applet de commande [New-SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate) de PowerShell pour générer un certificat.

1. Exécutez cette commande PowerShell pour générer un certificat auto-signé. Modifiez l’argument `-Subject` comme il convient pour votre application et le nom de locataire Azure AD B2C. Vous pouvez également ajuster la date de `-NotAfter` pour spécifier un délai d’expiration différent pour le certificat.
    ```powershell
    New-SelfSignedCertificate `
        -KeyExportPolicy Exportable `
        -Subject "CN=yourappname.yourtenant.onmicrosoft.com" `
        -KeyAlgorithm RSA `
        -KeyLength 2048 `
        -KeyUsage DigitalSignature `
        -NotAfter (Get-Date).AddMonths(12) `
        -CertStoreLocation "Cert:\CurrentUser\My"
    ```    
1. Ouvrez **Gérer les certificats utilisateur** > **Utilisateur actuel** > **Personnel** > **Certificats** > *yourappname.yourtenant.onmicrosoft.com*.
1. Sélectionnez le certificat > **Action** > **Toutes les tâches** > **Exporter**.
1. Sélectionnez **Oui** > **Suivant** > **Oui, exporter la clé privée** > **Suivant**.
1. Acceptez les valeurs par défaut pour **Format de fichier d’exportation**.
1. Fournissez un mot de passe pour le certificat.

### <a name="add-a-client-certificate-policy-key"></a>Ajoutez une clé de stratégie de certificat client

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Veillez à bien utiliser l’annuaire qui contient votre locataire Azure AD B2C. Sélectionnez le filtre **Annuaire + abonnement** dans le menu du haut, puis choisissez votre annuaire Azure AD B2C.
1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
1. Dans la page de vue d’ensemble, sélectionnez **Infrastructure d’expérience d’identité**.
1. Sélectionnez **Clés de stratégie**, puis **Ajouter**.
1. Dans la zone **Options**, sélectionnez **Charger**.
1. Dans la zone **Nom**, tapez **RestApiClientCertificate**.
    Le préfixe *B2C_1A_* est ajouté automatiquement.
1. Dans la zone **Chargement de fichier**, sélectionnez le fichier .pfx de votre certificat avec une clé privée.
1. Dans la zone **Mot de passe**, entrez le mot de passe du certificat.
1. Sélectionnez **Create** (Créer).

### <a name="configure-your-rest-api-technical-profile-to-use-client-certificate-authentication"></a>Configurer votre profil technique d’API REST afin d’utiliser l’authentification par certificat client

Après avoir créé la clé nécessaire, configurez vos métadonnées du profil technique de l’API REST pour référencer le certificat client.

1. Dans votre répertoire de travail, ouvrez le fichier de stratégie d’extension (TrustFrameworkExtensions.xml).
1. Recherchez le profil technique API REST. Par exemple, `REST-ValidateProfile` ou `REST-GetProfile`.
1. Recherchez l’élément `<Metadata>`.
1. Définissez *AuthenticationType* sur `ClientCertificate`.
1. Définissez *AllowInsecureAuthInProduction* sur `false`.
1. Immédiatement après l’élément `</Metadata>` fermant, ajoutez l’extrait de code XML suivant :
    ```xml
    <CryptographicKeys>
       <Key Id="ClientCertificate" StorageReferenceId="B2C_1A_RestApiClientCertificate" />
    </CryptographicKeys>
    ```

Voici un exemple de profil technique RESTful configuré avec un certificat client HTTP :

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-GetProfile">
      <DisplayName>Get user extended profile Azure Function web hook</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://your-account.azurewebsites.net/api/GetProfile?code=your-code</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <Item Key="AuthenticationType">ClientCertificate</Item>
        <Item Key="AllowInsecureAuthInProduction">false</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="ClientCertificate" StorageReferenceId="B2C_1A_RestApiClientCertificate" />
      </CryptographicKeys>
      ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="oauth2-bearer-authentication"></a>Authentification du porteur OAuth2 

[!INCLUDE [b2c-public-preview-feature](../../includes/active-directory-b2c-public-preview.md)]

L’authentification du jeton du porteur est définie dans le document [OAuth2.0 Authorization Framework: Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt). Dans l’authentification par jeton du porteur, Azure AD B2C envoie une requête HTTP avec un jeton dans l’en-tête d’autorisation.

```http
Authorization: Bearer <token>
```

Un jeton de porteur est une chaîne opaque. Il peut s’agir d’un jeton d’accès JWT ou d’une chaîne que l’API REST attend que le client Azure AD B2C envoie dans l’en-tête Authorization. Azure AD B2C prend en charge les types suivants :

- **Jeton du porteur**. Pour pouvoir envoyer le jeton du porteur dans le profil technique RESTful, votre stratégie doit d’abord acquérir le jeton du porteur, puis l’utiliser dans le profil technique RESTful.  
- **Jeton du porteur statique**. Utilisez cette approche lorsque votre API REST émet un jeton d’accès à long terme. Pour utiliser un jeton de porteur statique, créez une clé de stratégie et faites une référence du profil technique RESTful à votre clé de stratégie. 


## <a name="using-oauth2-bearer"></a>Utilisation du porteur OAuth2  

Les étapes suivantes montrent comment utiliser les informations d’identification du client pour obtenir un jeton du porteur et le transmettre à l’en-tête d’autorisation des appels de l’API REST.  

### <a name="define-a-claim-to-store-the-bearer-token"></a>Définir une revendication pour stocker le jeton du porteur

Une revendication fournit un stockage temporaire de données lors d’une exécution de stratégie Azure AD B2C. Le [schéma de revendications](claimsschema.md) est l’endroit où vous déclarez vos revendications. Le jeton d’accès doit être stocké dans une revendication à utiliser ultérieurement. 

1. Ouvrez le fichier d’extensions de votre stratégie. Par exemple <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>.
1. Recherchez l’élément [BuildingBlocks](buildingblocks.md). Si l’élément n’existe pas, ajoutez-le.
1. Localisez l’élément [ClaimsSchema](claimsschema.md). Si l’élément n’existe pas, ajoutez-le.
1. Ajoutez les revendications suivantes à l’élément **ClaimsSchema**.  

```xml
<ClaimType Id="bearerToken">
  <DisplayName>Bearer token</DisplayName>
  <DataType>string</DataType>
</ClaimType>
<ClaimType Id="grant_type">
  <DisplayName>Grant type</DisplayName>
  <DataType>string</DataType>
</ClaimType>
<ClaimType Id="scope">
  <DisplayName>scope</DisplayName>
  <DataType>string</DataType>
</ClaimType>
```

### <a name="acquiring-an-access-token"></a>Acquisition d’un jeton d’accès 

Vous pouvez obtenir un jeton d’accès de l’une des manières suivantes : en l’obtenant [auprès d’un fournisseur d’identité fédéré](idp-pass-through-user-flow.md), en appelant une API REST qui retourne un jeton d’accès, à l’aide d’un [flux ROPC](../active-directory/develop/v2-oauth-ropc.md) ou à l’aide du [flux d’informations d’identification du client](../active-directory/develop/v2-oauth2-client-creds-grant-flow.md).  

L’exemple suivant utilise un profil technique de l’API REST pour soumettre une requête au point de terminaison du jeton Azure AD à l’aide des informations d’identification du client passées en tant qu’authentification de base HTTP. Pour effectuer la configuration dans Azure AD, consultez [Plateforme d’identités Microsoft et flux d’informations d’identification du client OAuth 2.0](../active-directory/develop/v2-oauth2-client-creds-grant-flow.md). Vous devrez peut-être modifier cette valeur pour l’interface avec votre fournisseur d’identité. 

Au niveau de ServiceUrl, remplacez your-tenant-name par le nom de votre locataire Azure AD. Consultez la référence [Profil technique RESTful](restful-technical-profile.md) pour connaître toutes les options disponibles.

```xml
<TechnicalProfile Id="SecureREST-AccessToken">
  <DisplayName></DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="ServiceUrl">https://login.microsoftonline.com/your-tenant-name.onmicrosoft.com/oauth2/v2.0/token</Item>
    <Item Key="AuthenticationType">Basic</Item>
     <Item Key="SendClaimsIn">Form</Item>
  </Metadata>
  <CryptographicKeys>
    <Key Id="BasicAuthenticationUsername" StorageReferenceId="B2C_1A_SecureRESTClientId" />
    <Key Id="BasicAuthenticationPassword" StorageReferenceId="B2C_1A_SecureRESTClientSecret" />
  </CryptographicKeys>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="grant_type" DefaultValue="client_credentials" />
    <InputClaim ClaimTypeReferenceId="scope" DefaultValue="https://secureb2cfunction.azurewebsites.net/.default" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="bearerToken" PartnerClaimType="access_token" />
  </OutputClaims>
  <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
</TechnicalProfile>
```

### <a name="change-the-rest-technical-profile-to-use-bearer-token-authentication"></a>Modifier le profil technique REST pour utiliser l’authentification par jeton du porteur

Pour prendre en charge l’authentification par jeton du porteur dans votre stratégie personnalisée, modifiez le profil technique de l’API REST avec les éléments suivants :

1. Dans votre répertoire de travail, ouvrez le fichier de stratégie d’extension *TrustFrameworkExtensions.xml*.
1. Recherchez le nœud `<TechnicalProfile>` qui inclut `Id="REST-API-SignUp"`.
1. Recherchez l’élément `<Metadata>`.
1. Définissez *AuthenticationType* sur *Bearer*, comme suit :
    ```xml
    <Item Key="AuthenticationType">Bearer</Item>
    ```
1. Modifiez ou ajoutez *UseClaimAsBearerToken* en le définissant sur *bearerToken*, comme suit. *bearerToken* est le nom de la revendication à partir de laquelle le jeton du porteur sera récupéré (revendication de sortie de `SecureREST-AccessToken`).

    ```xml
    <Item Key="UseClaimAsBearerToken">bearerToken</Item>
    ```
    
1. Veillez à ajouter la revendication utilisée ci-dessus en tant que revendication d’entrée :

    ```xml
    <InputClaim ClaimTypeReferenceId="bearerToken"/>
    ```    

Une fois les extraits de code ci-dessus ajoutés, votre profil technique doit se présenter comme le code XML suivant :

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-GetProfile">
      <DisplayName>Get user extended profile Azure Function web hook</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://your-account.azurewebsites.net/api/GetProfile?code=your-code</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <Item Key="AuthenticationType">Bearer</Item>
        <Item Key="UseClaimAsBearerToken">bearerToken</Item>
        <Item Key="AllowInsecureAuthInProduction">false</Item>
      </Metadata>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="bearerToken"/>
      </InputClaims>
      ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="using-a-static-oauth2-bearer"></a>Utilisation d’un porteur OAuth2 statique 

### <a name="add-the-oauth2-bearer-token-policy-key"></a>Ajouter la clé de stratégie de jeton du porteur OAuth2

Pour configurer un profil technique API REST avec un jeton du porteur OAuth2, obtenez un jeton d’accès à partir du propriétaire de l’API REST. Créez ensuite la clé de chiffrement suivante pour stocker le jeton du porteur.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Veillez à bien utiliser l’annuaire qui contient votre locataire Azure AD B2C. Sélectionnez le filtre **Annuaire + abonnement** dans le menu du haut, puis choisissez votre annuaire Azure AD B2C.
1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
1. Dans la page de vue d’ensemble, sélectionnez **Infrastructure d’expérience d’identité**.
1. Sélectionnez **Clés de stratégie**, puis **Ajouter**.
1. Pour **Options**, choisissez `Manual`.
1. Entrez un **nom** pour la clé de stratégie. Par exemple : `RestApiBearerToken`. Le préfixe `B2C_1A_` est ajouté automatiquement au nom de votre clé.
1. Dans **Secret**, entrez la clé secrète client que vous avez enregistrée.
1. Pour **Utilisation de la clé**, sélectionnez `Encryption`.
1. Sélectionnez **Create** (Créer).

### <a name="configure-your-rest-api-technical-profile-to-use-the-bearer-token-policy-key"></a>Configurer votre profil technique de l’API REST pour utiliser la clé de stratégie de jeton du porteur

Après avoir créé la clé nécessaire, configurez vos métadonnées du profil technique de l’API REST pour référencer le jeton du porteur.

1. Dans votre répertoire de travail, ouvrez le fichier de stratégie d’extension (TrustFrameworkExtensions.xml).
1. Recherchez le profil technique API REST. Par exemple, `REST-ValidateProfile` ou `REST-GetProfile`.
1. Recherchez l’élément `<Metadata>`.
1. Définissez *AuthenticationType* sur `Bearer`.
1. Définissez *AllowInsecureAuthInProduction* sur `false`.
1. Immédiatement après l’élément `</Metadata>` fermant, ajoutez l’extrait de code XML suivant :
    ```xml
    <CryptographicKeys>
       <Key Id="BearerAuthenticationToken" StorageReferenceId="B2C_1A_RestApiBearerToken" />
    </CryptographicKeys>
    ```

Voici un exemple de profil technique RESTful configuré avec l’authentification du jeton du porteur :

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-GetProfile">
      <DisplayName>Get user extended profile Azure Function web hook</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://your-account.azurewebsites.net/api/GetProfile?code=your-code</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <Item Key="AuthenticationType">Bearer</Item>
        <Item Key="AllowInsecureAuthInProduction">false</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="BearerAuthenticationToken" StorageReferenceId="B2C_1A_RestApiBearerToken" />
      </CryptographicKeys>
      ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="api-key-authentication"></a>Authentification par clé API

La clé API est un identificateur unique utilisé pour authentifier un utilisateur afin d’accéder à un point de terminaison d’API REST. La clé est envoyée dans un en-tête HTTP personnalisé. Par exemple, le [déclencheur HTTP Azure Functions](../azure-functions/functions-bindings-http-webhook-trigger.md#authorization-keys) utilise l’en-tête HTTP `x-functions-key` pour identifier le demandeur.  

### <a name="add-api-key-policy-keys"></a>Ajouter des clés de stratégie de clé API

Pour configurer un profil technique d’API REST avec l’authentification par clé API, créez la clé de chiffrement suivante pour stocker la clé API :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Veillez à bien utiliser l’annuaire qui contient votre locataire Azure AD B2C. Sélectionnez le filtre **Annuaire + abonnement** dans le menu du haut, puis choisissez votre annuaire Azure AD B2C.
1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
1. Dans la page de vue d’ensemble, sélectionnez **Infrastructure d’expérience d’identité**.
1. Sélectionnez **Clés de stratégie**, puis **Ajouter**.
1. Pour **Options**, sélectionnez **Manuel**.
1. Dans **Nom**, indiquez **RestApiKey**.
    Il est possible que le préfixe *B2C_1A_* soit ajouté automatiquement.
1. Dans la zone **Secret**, entrez la clé de l’API REST.
1. Pour **Utilisation de la clé**, sélectionnez **Chiffrement**.
1. Sélectionnez **Create** (Créer).


### <a name="configure-your-rest-api-technical-profile-to-use-api-key-authentication"></a>Configurer votre profil technique d’API REST afin d’utiliser l’authentification par clé API

Après avoir créé la clé nécessaire, configurez les métadonnées du profil technique de votre API REST pour référencer les informations d’identification.

1. Dans votre répertoire de travail, ouvrez le fichier de stratégie d’extension (TrustFrameworkExtensions.xml).
1. Recherchez le profil technique API REST. Par exemple, `REST-ValidateProfile` ou `REST-GetProfile`.
1. Recherchez l’élément `<Metadata>`.
1. Définissez *AuthenticationType* sur `ApiKeyHeader`.
1. Définissez *AllowInsecureAuthInProduction* sur `false`.
1. Immédiatement après l’élément `</Metadata>` fermant, ajoutez l’extrait de code XML suivant :
    ```xml
    <CryptographicKeys>
        <Key Id="x-functions-key" StorageReferenceId="B2C_1A_RestApiKey" />
    </CryptographicKeys>
    ```

L’**ID** de la clé de chiffrement définit l’en-tête HTTP. Dans cet exemple, la clé API est envoyée sous la forme **x-functions-key**.

Voici un exemple de profil technique RESTful configuré pour appeler une application de fonction avec une authentification par clé API :

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-GetProfile">
      <DisplayName>Get user extended profile Azure Function web hook</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://your-account.azurewebsites.net/api/GetProfile?code=your-code</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <Item Key="AuthenticationType">ApiKeyHeader</Item>
        <Item Key="AllowInsecureAuthInProduction">false</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="x-functions-key" StorageReferenceId="B2C_1A_RestApiKey" />
      </CryptographicKeys>
      ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur le [profil technique RESTful](restful-technical-profile.md) dans le guide de référence IEF.