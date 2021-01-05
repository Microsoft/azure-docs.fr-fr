---
title: Point de terminaison UserInfo | Microsoft Docs
description: Définissez un point de terminaison UserInfo dans une stratégie personnalisée dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 12/14/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 73964fa1840f3f220662000eb91d34c3eb37bbd8
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97584423"
---
# <a name="userinfo-endpoint"></a>Point de terminaison UserInfo

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

Le point de terminaison UserInfo fait partie de la spécification [OpenID Connect standard](https://openid.net/specs/openid-connect-core-1_0.html#UserInfo) (OIDC) et est conçu pour retourner des revendications sur l’utilisateur authentifié. Le point de terminaison UserInfo est défini dans la stratégie de partie de confiance à l’aide de l’élément [EndPoint](relyingparty.md#endpoints).  

::: zone pivot="b2c-user-flow"

[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [active-directory-b2c-customization-prerequisites-custom-policy](../../includes/active-directory-b2c-customization-prerequisites-custom-policy.md)]

## <a name="userinfo-endpoint-overview"></a>Vue d’ensemble du point de terminaison UserInfo

Les informations utilisateur UserJourney spécifient les éléments suivants :

- **Autorisation** : Le point de terminaison UserInfo est protégé par un jeton du porteur. Un jeton d’accès émis est présenté dans l’en-tête d’autorisation au point de terminaison UserInfo. La stratégie spécifie le profil technique qui valide le jeton entrant et extrait les revendications, telles que l’objectId de l’utilisateur. L’objectId de l’utilisateur est utilisé pour récupérer les revendications à retourner dans la réponse du parcours du point de terminaison UserInfo. 
- **Étape d’orchestration** : 
  - Une étape d’orchestration permet de recueillir des informations sur l’utilisateur. En fonction des revendications contenues dans le jeton d’accès entrant, le parcours utilisateur appelle un [profil technique Azure Active Directory](active-directory-technical-profile.md) pour récupérer les données relatives à l’utilisateur, par exemple la lecture de l’utilisateur par l’objectId. 
  - **Étapes d’orchestration facultatives** : Vous pouvez ajouter d’autres étapes d’orchestration, telles qu’un profil technique de l’API REST pour récupérer des informations supplémentaires sur l’utilisateur. 
  - **Émetteur UserInfo** : Spécifie la liste des revendications retournées par le point de terminaison UserInfo.

## <a name="create-a-userinfo-endpoint"></a>Créer un point de terminaison UserInfo

### <a name="1-add-the-token-issuer-technical-profile"></a>1. Ajouter le profil technique de l’émetteur de jeton

1. Ouvrez le fichier *TrustFrameworkExtensions.xml*.
1. S’il n’existe pas encore, ajoutez un élément ClaimsProvider et ses éléments enfants comme premier élément sous l’élément BuildingBlocks.
1. Ajoutez le fournisseur de revendications suivant :

    ```xml
    <ClaimsProvider>
      <DisplayName>Token Issuer</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="UserInfoIssuer">
          <DisplayName>JSON Issuer</DisplayName>
          <Protocol Name="None" />
          <OutputTokenFormat>JSON</OutputTokenFormat>
          <CryptographicKeys>
            <Key Id="issuer_secret" StorageReferenceId="B2C_1A_TokenSigningKeyContainer" />
          </CryptographicKeys>
          <!-- The Below claims are what will be returned on the UserInfo Endpoint if in the Claims Bag-->
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="objectId"/>
            <InputClaim ClaimTypeReferenceId="givenName"/>
            <InputClaim ClaimTypeReferenceId="surname"/>
            <InputClaim ClaimTypeReferenceId="displayName"/>
            <InputClaim ClaimTypeReferenceId="signInNames.emailAddress"/>
          </InputClaims>
          <OutputClaims />
        </TechnicalProfile>
        <TechnicalProfile Id="UserInfoAuthorization">
          <DisplayName>UserInfo authorization</DisplayName>
          <Protocol Name="None" />
          <InputTokenFormat>JWT</InputTokenFormat>
          <Metadata>
            <!-- Update the Issuer and Audience below -->
            <!-- Audience is optional, Issuer is required-->
            <Item Key="issuer">https://yourtenant.b2clogin.com/11111111-1111-1111-1111-111111111111/v2.0/</Item>
            <Item Key="audience">[ "22222222-2222-2222-2222-222222222222", "33333333-3333-3333-3333-333333333333" ]</Item>
            <Item Key="client_assertion_type">urn:ietf:params:oauth:client-assertion-type:jwt-bearer</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="issuer_secret" StorageReferenceId="B2C_1A_TokenSigningKeyContainer" />
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
            <!-- Optional claims to read from the access token  -->
            <!-- <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name"/>
                 <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name"/>
                 <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name"/> -->
          </OutputClaims>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ``` 

1. La section outputClaims du profil technique UserInfoIssuer spécifie les attributs que vous souhaitez retourner. Le profil technique UserInfoIssuer est appelé à la fin du parcours utilisateur. 
1. Le profil technique UserInfoAuthorization valide la signature, le nom de l’émetteur et l’audience du jeton, puis extrait la revendication du jeton entrant. Modifiez les métadonnées suivantes pour refléter votre environnement :
    1. **issuer** : Cette valeur doit être identique à la revendication `iss` dans la revendication du jeton d’accès. Les jetons émis par Azure AD B2C utilisent un émetteur au format `https://yourtenant.b2clogin.com/your-tenant-id/v2.0/`. En savoir plus sur la [personnalisation des jetons](configure-tokens.md).
    1. **IdTokenAudience** : Cette valeur doit être identique à la revendication `aud` dans la revendication du jeton d’accès. Dans Azure AD B2C, la revendication `aud` est l’ID de votre application par partie de confiance. Cette valeur est une collection et prend en charge plusieurs valeurs en utilisant une virgule de séparation.

Dans le jeton d’accès suivant, la valeur de la revendication `iss` est `https://contoso.b2clogin.com/11111111-1111-1111-1111-111111111111/v2.0/`. La valeur de la revendication `aud` est `22222222-2222-2222-2222-222222222222`.

```json
{
  "exp": 1605549468,
  "nbf": 1605545868,
  "ver": "1.0",
  "iss": "https://contoso.b2clogin.com/11111111-1111-1111-1111-111111111111/v2.0/",
  "sub": "44444444-4444-4444-4444-444444444444",
  "aud": "22222222-2222-2222-2222-222222222222",
  "acr": "b2c_1a_signup_signin",
  "nonce": "defaultNonce",
  "iat": 1605545868,
  "auth_time": 1605545868,
  "name": "John Smith",
  "given_name": "John",
  "family_name": "Smith",
  "tid": "11111111-1111-1111-1111-111111111111"
}
```

### <a name="2-add-the-userjourney-element"></a>2. Ajouter l’élément UserJourney 

L’élément [UserJourney](userjourneys.md) définit le chemin emprunté par l’utilisateur lors de l’interaction avec votre application. Ajoutez l’élément **UserJourneys** s’il n’existe pas avec l’élément **UserJourney** identifié comme `UserInfoJourney` :

```xml
<UserJourney Id="UserInfoJourney" DefaultCpimIssuerTechnicalProfileReferenceId="UserInfoIssuer">
  <Authorization>
    <AuthorizationTechnicalProfiles>
      <AuthorizationTechnicalProfile ReferenceId="UserInfoAuthorization" />
    </AuthorizationTechnicalProfiles>
  </Authorization>
  <OrchestrationSteps >
    <OrchestrationStep Order="1" Type="ClaimsExchange">
      <Preconditions>
        <Precondition Type="ClaimsExist" ExecuteActionsIf="false">
          <Value>objectId</Value>
          <Action>SkipThisOrchestrationStep</Action>
        </Precondition>
      </Preconditions>
      <ClaimsExchanges UserIdentity="false">
        <ClaimsExchange Id="AADUserReadWithObjectId" TechnicalProfileReferenceId="AAD-UserReadUsingObjectId" />
      </ClaimsExchanges>
    </OrchestrationStep>
    <OrchestrationStep Order="2" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="UserInfoIssuer" />
  </OrchestrationSteps>
</UserJourney>
```

### <a name="3-include-the-endpoint-to-the-relying-party-policy"></a>3. Inclure le point de terminaison à la stratégie de partie de confiance

Pour inclure le point de terminaison UserInfo dans l’application par partie de confiance, ajoutez un élément [Endpoint](relyingparty.md#endpoints) au fichier *SocialAndLocalAccounts/SignUpOrSignIn.xml*. 

```xml
<Endpoints>
  <Endpoint Id="UserInfo" UserJourneyReferenceId="UserInfoJourney" />
</Endpoints>
```

L’élément terminé de la partie de confiance est le suivant :

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<TrustFrameworkPolicy xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="http://www.w3.org/2001/XMLSchema"
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06" PolicySchemaVersion="0.3.0.0" TenantId="yourtenant.onmicrosoft.com" PolicyId="B2C_1A_signup_signin" PublicPolicyUri="http://yourtenant.onmicrosoft.com/B2C_1A_signup_signin">
  <BasePolicy>
    <TenantId>yourtenant.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkExtensions</PolicyId>
  </BasePolicy>
  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
    <Endpoints>
      <Endpoint Id="UserInfo" UserJourneyReferenceId="UserInfoJourney" />
    </Endpoints>
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
        <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
      </OutputClaims>
      <SubjectNamingInfo ClaimType="sub" />
    </TechnicalProfile>
  </RelyingParty>
</TrustFrameworkPolicy>
```

### <a name="4-upload-the-files"></a>4. Téléchargement des fichiers

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Veillez à utiliser l’annuaire qui contient votre locataire Azure AD B2C en sélectionnant le filtre **Annuaire + abonnement** dans le menu du haut et en choisissant l’annuaire qui contient votre locataire.
1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Infrastructure d’expérience d’identité**.
1. Dans la page **Stratégies personnalisées**, sélectionnez **Télécharger la stratégie personnalisée**.
1. Sélectionnez **Remplacer la stratégie personnalisée si elle existe déjà**, puis recherchez et sélectionnez le fichier *TrustframeworkExtensions.xml*.
1. Cliquez sur **Télécharger**.
1. Répétez les étapes 5 à 7 pour le fichier de la partie de confiance (par exemple, *SignUpOrSignIn.xml*).

## <a name="calling-the-userinfo-endpoint"></a>Appel du point de terminaison UserInfo

Le point de terminaison UserInfo utilise l’API de jeton du porteur OAuth2 standard, appelée à l’aide du jeton d’accès reçu lors de l’obtention d’un jeton pour votre application. Elle retourne une réponse JSON contenant les revendications relatives à l’utilisateur. Le point de terminaison UserInfo est hébergé sur Azure AD B2C à l’adresse :

```http
https://yourtenant.b2clogin.com/yourtenant.onmicrosoft.com/policy-name/openid/v2.0/userinfo
```

Le point de terminaison de configuration /.well-known (document de découverte OpenID Connect) répertorie le champ `userinfo_endpoint`. Vous pouvez découvrir par programmation le point de terminaison UserInfo à l’aide du point de terminaison de configuration /.well-known à l’adresse suivante : 

```http
https://yourtenant.b2clogin.com/yourtenant.onmicrosoft.com/policy-name/v2.0/.well-known/openid-configuration 
```

### <a name="test-the-policy"></a>Tester la stratégie

1. Sous **Stratégies personnalisées**, sélectionnez la stratégie avec laquelle vous avez intégré le point de terminaison UserInfo. Par exemple, *B2C_1A_SignUpOrSignIn*.
1. Sélectionnez **Exécuter maintenant**. 
1. Sous **Sélectionner une application**, sélectionnez l’application que vous avez précédemment inscrite. Pour **Sélectionner l’URL de réponse**, choisissez `https://jwt.ms`. Pour plus d’informations, voir [Inscrire une application web dans Azure Active Directory B2C](tutorial-register-applications.md).
1. Inscrivez-vous ou connectez-vous à l’aide d’une adresse e-mail ou d’un compte social.
1. Copiez l’élément id_token dans son format encodé à partir du site web [https://jwt.ms](https://jwt.ms). Vous pouvez l’utiliser pour envoyer une requête GET au point de terminaison UserInfo et récupérer les informations utilisateur.
1. Envoyez une requête GET au point de terminaison UserInfo et récupérez les informations utilisateur.

```http
GET /yourtenant.onmicrosoft.com/b2c_1a_signup_signin/openid/v2.0/userinfo
Host: b2cninja.b2clogin.com
Authorization: Bearer <your ID token>
```

Une réponse correcte se présente comme suit :

```json
{
    "objectId": "44444444-4444-4444-4444-444444444444",
    "givenName": "John",
    "surname": "Smith",
    "displayName": "John Smith",
    "signInNames.emailAddress": "john.s@contoso.com"
}
```

## <a name="next-steps"></a>Étapes suivantes

- Vous trouverez un exemple de stratégie de point de terminaison UserInfo sur [GitHub](https://github.com/azure-ad-b2c/samples/tree/master/policies/user-info-endpoint).

::: zone-end