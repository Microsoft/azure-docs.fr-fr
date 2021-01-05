---
title: Échanges de revendications de l’API REST - Azure Active Directory B2C
description: Ajoutez des échanges de revendications de l’API REST aux stratégies personnalisées dans Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 10/15/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 84053df34ffda0d4686ad80a9e5f3af00ac53d72
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94949493"
---
# <a name="walkthrough-add-rest-api-claims-exchanges-to-custom-policies-in-azure-active-directory-b2c"></a>Procédure pas à pas : Ajouter des échanges de revendications d’API REST aux stratégies personnalisées dans Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) permet aux développeurs d’identité d’intégrer une interaction avec une API RESTful dans un parcours utilisateur. À la fin de cette procédure pas à pas, vous pourrez créer un parcours utilisateur Azure AD B2C qui interagit avec des [services RESTful](custom-policy-rest-api-intro.md).

Dans ce scénario, nous enrichissons les données de jeton de l’utilisateur en les intégrant à un workflow métier. Lors de l’inscription ou de la connexion avec un compte local ou fédéré, Azure AD B2C appelle une API REST pour obtenir les données de profil étendues de l’utilisateur à partir d’une source de données distante. Dans cet exemple, Azure AD B2C envoie l’identificateur unique de l’utilisateur, objectId. L’API REST retourne ensuite le solde du compte de l’utilisateur (un nombre aléatoire). Utilisez cet exemple comme point de départ pour une intégration avec votre propre système CRM, votre base de données marketing ou tout workflow métier.

Vous pouvez aussi concevoir l’interaction comme un profil technique de validation. Cela convient lorsque l’API REST valide les données à l’écran et renvoie des revendications. Pour plus d’informations, consultez [Procédure pas à pas : Intégrer les échanges de revendications de l’API REST dans votre parcours utilisateur Azure AD B2C pour valider une entrée d’utilisateur](custom-policy-rest-api-claims-validation.md).

## <a name="prerequisites"></a>Prérequis

- Suivez les étapes décrites dans [Bien démarrer avec les stratégies personnalisées dans Azure Active Directory B2C](custom-policy-get-started.md). Vous devez disposer d’une stratégie personnalisée fonctionnelle pour l’inscription et la connexion avec des comptes locaux.
- Découvrez comment [intégrer des échanges de revendications de l’API REST dans votre stratégie personnalisée Azure AD B2C](custom-policy-rest-api-intro.md).

## <a name="prepare-a-rest-api-endpoint"></a>Préparer un point de terminaison d’API REST

Pour cette procédure pas à pas, vous devez disposer d’une API REST qui vérifie si l’objectId Azure AD B2C d’un utilisateur est inscrit dans votre système principal. S’il est inscrit, l’API REST renvoie le solde du compte d’utilisateur. Dans le cas contraire, l’API REST inscrit le nouveau compte dans le répertoire et retourne le solde initial de `50.00`.

Le code JSON suivant illustre les données qu’Azure AD B2C enverra à votre point de terminaison d’API REST. 

```json
{
    "objectId": "User objectId",
    "lang": "Current UI language"
}
```

Une fois que votre API REST valide les données, elle doit retourner un message HTTP 200 (OK), avec les données JSON suivantes :

```json
{
    "balance": "760.50"
}
```

Cet article ne traite pas de la configuration du point de terminaison d’API REST. Vous avez créé un exemple [Azure Functions](../azure-functions/functions-reference.md). Vous pouvez accéder au code complet de la fonction Azure sur le site de [GitHub](https://github.com/azure-ad-b2c/rest-api/tree/master/source-code/azure-function).

## <a name="define-claims"></a>Définir des revendications

Une revendication fournit un stockage temporaire de données lors d’une exécution de stratégie Azure AD B2C. Vous pouvez déclarer des revendications dans la section [Schéma de revendications](claimsschema.md). 

1. Ouvrez le fichier d’extensions de votre stratégie. Par exemple <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>.
1. Recherchez l’élément [BuildingBlocks](buildingblocks.md). Si l’élément n’existe pas, ajoutez-le.
1. Localisez l’élément [ClaimsSchema](claimsschema.md). Si l’élément n’existe pas, ajoutez-le.
1. Ajoutez les revendications suivantes à l’élément **ClaimsSchema**.  

```xml
<ClaimType Id="balance">
  <DisplayName>Your Balance</DisplayName>
  <DataType>string</DataType>
</ClaimType>
<ClaimType Id="userLanguage">
  <DisplayName>User UI language (used by REST API to return localized error messages)</DisplayName>
  <DataType>string</DataType>
</ClaimType>
```

## <a name="add-the-restful-api-technical-profile"></a>Ajouter le profil technique de l’API RESTful 

Un [profil technique RESTful](restful-technical-profile.md) prend en charge la création d’une interface avec votre propre service RESTful. Azure Active Directory B2C envoie des données au service RESTful dans une collection `InputClaims` et reçoit des données en retour dans une collection `OutputClaims`. Recherchez l’élément **ClaimsProviders** dans votre fichier <em> **`TrustFrameworkExtensions.xml`**</em> et ajoutez un fournisseur de revendications comme suit :

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-GetProfile">
      <DisplayName>Get user extended profile Azure Function web hook</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <!-- Set the ServiceUrl with your own REST API endpoint -->
        <Item Key="ServiceUrl">https://your-account.azurewebsites.net/api/GetProfile?code=your-code</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <!-- Set AuthenticationType to Basic or ClientCertificate in production environments -->
        <Item Key="AuthenticationType">None</Item>
        <!-- REMOVE the following line in production environments -->
        <Item Key="AllowInsecureAuthInProduction">true</Item>
      </Metadata>
      <InputClaims>
        <!-- Claims sent to your REST API -->
        <InputClaim ClaimTypeReferenceId="objectId" />
        <InputClaim ClaimTypeReferenceId="userLanguage" PartnerClaimType="lang" DefaultValue="{Culture:LCID}" AlwaysUseDefaultValue="true" />
      </InputClaims>
      <OutputClaims>
        <!-- Claims parsed from your REST API -->
        <OutputClaim ClaimTypeReferenceId="balance" />
      </OutputClaims>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
``` 

Dans cet exemple, le `userLanguage` sera envoyé au service REST en tant que `lang` au sein de la charge utile JSON. La valeur de la revendication `userLanguage` contient l’ID de langue de l’utilisateur actuel. Pour plus d’informations, consultez [Programmes de résolution de revendication](claim-resolver-overview.md).

### <a name="configure-the-restful-api-technical-profile"></a>Configurer le profil technique de l’API RESTful 

Après avoir déployé votre API REST, définissez les métadonnées du profil technique `REST-ValidateProfile` pour refléter votre propre API REST, notamment :

- **ServiceUrl**. Définissez l’URL du point de terminaison de l’API REST.
- **SendClaimsIn**. Spécifiez la façon dont les revendications d’entrée sont envoyées au fournisseur de revendications RESTful.
- **AuthenticationType**. Définissez le type de l’authentification effectuée par le fournisseur de revendications RESTful. 
- **AllowInsecureAuthInProduction**. Dans un environnement de production, veillez à définir ces métadonnées sur `true`
    
Pour plus d’informations sur les configurations, consultez [Métadonnées du profil technique RESTful](restful-technical-profile.md#metadata).

Les commentaires ci -dessus `AuthenticationType` et `AllowInsecureAuthInProduction` indiquent les modifications que vous devez effectuer lorsque vous passez à un environnement de production. Pour savoir comment sécuriser vos API RESTful pour la production, consultez [Sécuriser une API RESTful](secure-rest-api.md).

## <a name="add-an-orchestration-step"></a>Ajouter une étape d’orchestration

Les [parcours utilisateur](userjourneys.md) spécifient des chemins explicites par le biais desquels une stratégie autorise une application par partie de confiance à obtenir les revendications souhaitées pour un utilisateur. Un parcours utilisateur est représenté en tant que séquence d’orchestration qui doit être suivie pour que la transaction réussisse. Vous pouvez ajouter ou soustraire des étapes d’orchestration. Dans ce cas, vous allez ajouter une étape d’orchestration qui est utilisée pour compléter les informations fournies à l’application après l’inscription ou la connexion de l’utilisateur via l’appel de l’API REST.

1. Ouvrez le fichier de base de votre stratégie. Par exemple <em>`SocialAndLocalAccounts/`**`TrustFrameworkBase.xml`**</em>.
1. Recherchez l’élément `<UserJourneys>`. Copiez l’élément dans son intégralité, puis supprimez-le.
1. Ouvrez le fichier d’extensions de votre stratégie. Par exemple <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>.
1. Collez le `<UserJourneys>` dans le fichier des extensions après avoir fermé l’élément `<ClaimsProviders>`.
1. Localisez le `<UserJourney Id="SignUpOrSignIn">` et ajoutez l’étape d’orchestration suivante en avant-dernier.

    ```xml
    <OrchestrationStep Order="7" Type="ClaimsExchange">
      <ClaimsExchanges>
        <ClaimsExchange Id="RESTGetProfile" TechnicalProfileReferenceId="REST-GetProfile" />
      </ClaimsExchanges>
    </OrchestrationStep>
    ```

1. Refactorisez la dernière étape d’orchestration en changeant la valeur de `Order` en `8`. Les deux dernières étapes de l’orchestration doivent ressembler à ce qui suit :

    ```xml
    <OrchestrationStep Order="7" Type="ClaimsExchange">
      <ClaimsExchanges>
        <ClaimsExchange Id="RESTGetProfile" TechnicalProfileReferenceId="REST-GetProfile" />
      </ClaimsExchanges>
    </OrchestrationStep>

    <OrchestrationStep Order="8" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
    ```

1. Répétez les deux dernières étapes pour les parcours utilisateur **ProfileEdit** et **PasswordReset**.


## <a name="include-a-claim-in-the-token"></a>Inclure une revendication dans le jeton 

Pour retourner la revendication `balance` à l’application par partie de confiance, ajoutez une revendication de sortie au fichier <em>`SocialAndLocalAccounts/`**`SignUpOrSignIn.xml`**</em>. L’ajout d’une revendication de sortie émettra la revendication dans le jeton après un parcours utilisateur réussi et l’enverra à l’application. Modifiez l’élément de profil technique dans la section de partie de confiance pour ajouter `balance` en tant que revendication de sortie.
 
```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="OpenIdConnect" />
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="displayName" />
      <OutputClaim ClaimTypeReferenceId="givenName" />
      <OutputClaim ClaimTypeReferenceId="surname" />
      <OutputClaim ClaimTypeReferenceId="email" />
      <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
      <OutputClaim ClaimTypeReferenceId="identityProvider" />
      <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
      <OutputClaim ClaimTypeReferenceId="balance" DefaultValue="" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```

Répétez cette étape pour les parcours utilisateur **ProfileEdit.xml** et **PasswordReset.xml**.

Enregistrez les fichiers que vous avez modifiés : *TrustFrameworkBase.xml*, *TrustFrameworkExtensions.xml*, *SignUpOrSignin.xml*, *ProfileEdit.xml* et *PasswordReset.xml*. 

## <a name="test-the-custom-policy"></a>Tester la stratégie personnalisée

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Veillez à utiliser l’annuaire qui contient votre locataire Azure AD en sélectionnant le filtre **Annuaire + abonnement** dans le menu du haut et en choisissant l’annuaire qui contient votre locataire Azure AD.
1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Inscriptions d’applications**.
1. Sélectionnez **Infrastructure d’expérience d’identité**.
1. Sélectionnez **Charger une stratégie personnalisée**, puis chargez les fichiers de stratégie que vous avez modifiés : *TrustFrameworkBase.xml*, *TrustFrameworkExtensions.xml*, *SignUpOrSignin.xml*, *ProfileEdit.xml* et *PasswordReset.xml*. 
1. Sélectionnez la stratégie d’inscription et de connexion que vous avez chargée, puis cliquez sur le bouton **Exécuter maintenant**.
1. Vous devriez pouvoir vous inscrire au moyen d’une adresse e-mail ou d’un compte Facebook.
1. Le jeton envoyé à votre application inclut la revendication `balance`.

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "X5eXk4xyojNFum1kl2Ytv8dlNP4-c57dO6QGTVBwaNk"
}.{
  "exp": 1584961516,
  "nbf": 1584957916,
  "ver": "1.0",
  "iss": "https://contoso.b2clogin.com/f06c2fe8-709f-4030-85dc-38a4bfd9e82d/v2.0/",
  "aud": "e1d2612f-c2bc-4599-8e7b-d874eaca1ee1",
  "acr": "b2c_1a_signup_signin",
  "nonce": "defaultNonce",
  "iat": 1584957916,
  "auth_time": 1584957916,
  "name": "Emily Smith",
  "email": "emily@outlook.com",
  "given_name": "Emily",
  "family_name": "Smith",
  "balance": "202.75"
  ...
}
```

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la sécurisation de vos API, consultez les articles suivants :

- [Procédure pas à pas : Intégrer les échanges de revendications de l’API REST dans votre parcours utilisateur Azure AD B2C comme étape d’orchestration](custom-policy-rest-api-claims-exchange.md)
- [Sécuriser votre API RESTful](secure-rest-api.md)
- [Reference : Profil technique RESTful](restful-technical-profile.md)