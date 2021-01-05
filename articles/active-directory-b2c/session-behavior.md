---
title: Configurer le comportement de session – Azure Active Directory B2C | Microsoft Docs
description: Découvrez comment configurer le comportement de session dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/14/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: ad9bd8dec94660d94cf3a106d31dafdad06f47a8
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97584508"
---
# <a name="configure-session-behavior-in-azure-active-directory-b2c"></a>Configurer le comportement de session dans Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

Lorsque les utilisateurs s’authentifient auprès d’applications dans Azure Active Directory B2C (Azure AD B2C), l’authentification unique (SSO) constitue la méthode la plus sécurisée et la plus pratique. Cet article décrit les différentes méthodes d’authentification unique utilisées dans Azure AD B2C et vous aide à choisir la méthode SSO la plus appropriée lors de la configuration de votre stratégie.

Avec l’authentification unique, les utilisateurs se connectent une seule fois avec un seul compte et accèdent à plusieurs applications. L’application peut être une application web, mobile ou monopage, indépendamment du nom de la plateforme ou du domaine.

Lorsque l’utilisateur se connecte pour la première fois à une application, Azure AD B2C maintient une session basée sur un cookie. Lors des demandes d’authentification suivantes, Azure AD B2C lit et valide la session basée sur un cookie et émet un jeton d’accès sans inviter l’utilisateur à se reconnecter. Si la session basée sur un cookie expire ou devient non valide, l’utilisateur est invité à se connecter à nouveau.  

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="azure-ad-b2c-session-overview"></a>Vue d’ensemble de la session Azure AD B2C

L’intégration avec Azure AD B2C implique trois types de sessions SSO :

- **Azure AD B2C** : session gérée par Azure AD B2C
- **Fournisseur d’identité fédéré** : session gérée par le fournisseur d’identité, par exemple Facebook, Salesforce ou compte Microsoft
- **Application** : session gérée par l’application web, mobile ou monopage

![Session SSO](media/session-behavior/sso-session-types.png)

### <a name="azure-ad-b2c-session"></a>Session Azure AD B2C 

Quand un utilisateur réussit à s’authentifier avec un compte local ou social, Azure AD B2C stocke une session basée sur un cookie sur le navigateur de l’utilisateur. Le cookie est stocké sous le nom de domaine du locataire Azure AD B2C, par exemple, `https://contoso.b2clogin.com`.

Si un utilisateur se connecte initialement avec un compte fédéré, puis, pendant la fenêtre de temps de la session (durée de vie ou TTL), se connecte à la même application ou à une autre application, Azure AD B2C essaie d’acquérir un nouveau jeton d’accès auprès du fournisseur d’identité fédéré. Si la session du fournisseur d’identité fédéré a expiré ou n’est pas valide, le fournisseur d’identité fédéré invite l’utilisateur à entrer ses informations d’identification. Si la session est toujours active (ou si l’utilisateur s’est connecté avec un compte local au lieu d’un compte fédéré), Azure AD B2C autorise l’utilisateur et élimine les invites supplémentaires.

Vous pouvez configurer le comportement de la session, dont sa durée de vie et la façon dont Azure AD B2C partage la session entre les stratégies et les applications.

### <a name="federated-identity-provider-session"></a>Session du fournisseur d’identité fédéré

Un fournisseur d’identité de réseaux sociaux ou d’entreprise gère sa propre session. Le cookie est stocké sous le nom de domaine du fournisseur d’identité, par exemple `https://login.salesforce.com`. Azure AD B2C ne contrôle pas la session du fournisseur d’identité fédéré. Au lieu de cela, le comportement de session est déterminé par le fournisseur d’identité fédéré. 

Examinez le cas suivant :

1. Un utilisateur se connecte à Facebook pour vérifier son flux.
2. Plus tard, l’utilisateur ouvre votre application et démarre le processus de connexion. L’application redirige l’utilisateur vers Azure AD B2C pour terminer le processus de connexion.
3. Sur la page d’inscription ou de connexion d’Azure AD B2C, l’utilisateur choisit de se connecter avec son compte Facebook. L’utilisateur est redirigé vers Facebook. S’il existe une session active sur Facebook, l’utilisateur n’est pas invité à fournir ses informations d’identification et est immédiatement redirigé vers Azure AD B2C avec un jeton Facebook.

### <a name="application-session"></a>Session d’application

Une application web, mobile ou monopage peut être protégée par un accès OAuth, des jetons d’ID ou des jetons SAML. Quand un utilisateur tente d’accéder à une ressource protégée sur l’application, celle-ci vérifie s’il existe une session active côté application. S’il n’y a pas de session d’application ou si la session a expiré, l’application dirige l’utilisateur vers la page de connexion d’Azure AD B2C.

La session d’application peut être une session basée sur un cookie stockée sous le nom de domaine de l’application, par exemple `https://contoso.com`. Les applications mobiles peuvent stocker la session d’une façon différente, mais dont l’approche est similaire.

## <a name="configure-azure-ad-b2c-session-behavior"></a>Configurer le comportement de la session Azure AD B2C

Vous pouvez configurer le comportement de la session Azure AD B2C, notamment les éléments suivants :

- **Durée de vie de la session de l’application web (minutes)**  : durée pendant laquelle le cookie de session Azure AD B2C est stocké sur le navigateur de l’utilisateur après une authentification réussie. Vous pouvez définir la durée de vie de la session sur une valeur comprise entre 15 et 720 minutes.

- **Délai d’expiration de session d’application web** : indique comment une session est étendue par le paramètre Durée de vie de la session ou le paramètre Maintenir la connexion.
  - **Continue** : indique que la session est étendue chaque fois que l’utilisateur effectue une authentification basée sur un cookie (valeur par défaut).
  - **Absolue** : indique que l’utilisateur est obligé de se réauthentifier après la période spécifiée.

- **Configuration de l’authentification unique** : vous pouvez configurer la session Azure AD B2C avec les étendues suivantes :
  - **Client** : il s’agit du paramètre par défaut. L’utilisation de ce paramètre permet à plusieurs applications et flux d’utilisateur dans votre locataire B2C de partager la même session utilisateur. Par exemple, quand un utilisateur se connecte à une application, il peut également se connecter sans problème à une autre en y accédant.
  - **Application**: ce paramètre vous permet de maintenir une session utilisateur exclusivement pour une application, indépendamment des autres applications. Par exemple, vous pouvez utiliser ce paramètre si vous voulez que l’utilisateur se connecte à Contoso Pharmacy, que l’utilisateur y soit ou non déjà connecté.
  - **Stratégie** : ce paramètre vous permet de maintenir une session utilisateur exclusivement pour un flux d’utilisateur, indépendamment des applications qui l’utilisent. Par exemple, si l’utilisateur s’est déjà connecté et a effectué une étape d’authentification multifacteur, il peut obtenir l’accès à des parties plus sécurisées de plusieurs applications tant que la session liée au flux d’utilisateur n’expire pas.
  - **Désactivé** : ce paramètre oblige l’utilisateur à réexécuter tout le flux d’utilisateur à chaque exécution de la stratégie.
::: zone pivot="b2c-custom-policy"
- **Maintenir la connexion** : étend la durée de vie de la session à l’aide d’un cookie persistant. La session reste active lorsque l’utilisateur ferme et rouvre le navigateur. La session est révoquée uniquement quand un utilisateur se déconnecte. La fonctionnalité Maintenir la connexion s’applique uniquement à la connexion avec des comptes locaux. La fonctionnalité Maintenir la connexion prend le pas sur la durée de vie de la session. Si la fonctionnalité Maintenir la connexion est activée et que l’utilisateur la sélectionne, elle détermine la date d’expiration de la session. 
::: zone-end

::: zone pivot="b2c-user-flow"

Pour configurer le comportement de la session :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Veillez à utiliser l’annuaire qui contient votre locataire Azure AD B2C en sélectionnant le filtre **Annuaire et abonnement** dans le menu supérieur et en choisissant l’annuaire qui contient votre locataire Azure AD B2C.
1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Flux d’utilisateurs**.
1. Ouvrez le flux utilisateur que vous avez créé précédemment.
1. Sélectionner **Propriétés**.
1. Configurez **Durée de vie de la session de l’application web (minutes)** , **Délai d’expiration de la session de l’application web** , **Configuration de l’authentification unique** et **Exiger un jeton d’ID dans les demandes de déconnexion**, selon les besoins.
1. Cliquez sur **Enregistrer**.

::: zone-end

::: zone pivot="b2c-custom-policy"

Pour changer le comportement des sessions et les configurations de SSO, vous devez ajouter un élément **UserJourneyBehaviors** à l’intérieur de l’élément [RelyingParty](relyingparty.md).  L’élément **UserJourneyBehaviors** doit suivre immédiatement l’élément **DefaultUserJourney**. Votre élément **UserJourneyBehaviors** devrait ressembler à l’exemple suivant :

```xml
<UserJourneyBehaviors>
   <SingleSignOn Scope="Application" />
   <SessionExpiryType>Absolute</SessionExpiryType>
   <SessionExpiryInSeconds>86400</SessionExpiryInSeconds>
</UserJourneyBehaviors>
```

## <a name="enable-keep-me-signed-in-kmsi"></a>Activer le maintien de la connexion (KMSI)

Vous pouvez activer la fonctionnalité Maintenir la connexion pour les utilisateurs de vos applications web et natives qui ont des comptes locaux dans votre annuaire Azure AD B2C (Azure Active Directory B2C). Cette fonctionnalité accorde l’accès aux utilisateurs qui retournent dans votre application sans avoir à entrer une nouvelle fois leur nom d’utilisateur et leur mot de passe. Cet accès est révoqué lorsque l’utilisateur se déconnecte.

![Exemple de page de connexion avec la case à cocher Maintenir la connexion](./media/session-behavior/keep-me-signed-in.png)

Vous ne devez pas activer cette option sur les ordinateurs publics.

### <a name="configure-the-page-identifier"></a>Configurer l’identificateur de page

Pour activer la fonctionnalité « Maintenir la connexion », définissez l’élément `DataUri` de définition de contenu sur l’[identificateur de page](contentdefinitions.md#datauri) `unifiedssp` et la [version de page](page-layout.md) *1.1.0* ou ultérieure.

1. Ouvrez le fichier d’extension de votre stratégie. Par exemple <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>. Ce fichier d’extension est un des fichiers de stratégie inclus dans le pack de démarrage des stratégies personnalisées, que vous avez dû obtenir en suivant la rubrique prérequise [Bien démarrer avec les stratégies personnalisées](custom-policy-get-started.md).
1. Recherchez l’élément **BuildingBlocks**. Si l’élément n’existe pas, ajoutez-le.
1. Ajoutez l’élément **ContentDefinitions** à l’élément **BuildingBlocks** de la stratégie.

    Votre stratégie personnalisée doit se présenter comme l’extrait de code suivant :

    ```xml
    <BuildingBlocks>
      <ContentDefinitions>
        <ContentDefinition Id="api.signuporsignin">
          <DataUri>urn:com:microsoft:aad:b2c:elements:unifiedssp:1.1.0</DataUri>
        </ContentDefinition>
      </ContentDefinitions>
    </BuildingBlocks>
    ```

### <a name="add-the-metadata-to-the-self-asserted-technical-profile"></a>Ajouter les métadonnées au profil technique autodéclaré

Pour ajouter la case à cocher Maintenir la connexion dans les pages d’inscription et de connexion, définissez les métadonnées `setting.enableRememberMe` sur true. Remplacez les profils techniques SelfAsserted-LocalAccountSignin-Email dans le fichier d’extension.

1. Recherchez l’élément ClaimsProviders. Si l’élément n’existe pas, ajoutez-le.
1. Ajoutez le fournisseur de revendications suivant à l’élément ClaimsProviders :

```xml
<ClaimsProvider>
  <DisplayName>Local Account</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="SelfAsserted-LocalAccountSignin-Email">
      <Metadata>
        <Item Key="setting.enableRememberMe">True</Item>
      </Metadata>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

1. Enregistrez le fichier d’extensions.

### <a name="configure-a-relying-party-file"></a>Configurer un fichier de partie de confiance

Mettez à jour le fichier de partie de confiance qui lance le parcours utilisateur que vous avez créé.

1. Ouvrez votre fichier de stratégie personnalisée. Par exemple *SignUpOrSignin.xml*.
1. S’il n’existe pas déjà, ajoutez un nœud enfant `<UserJourneyBehaviors>` au nœud `<RelyingParty>`. Il doit être placé immédiatement après `<DefaultUserJourney ReferenceId="User journey Id" />`, par exemple : `<DefaultUserJourney ReferenceId="SignUpOrSignIn" />`.
1. Ajoutez le nœud suivant en tant qu’enfant de l’élément `<UserJourneyBehaviors>`.

    ```xml
    <UserJourneyBehaviors>
      <SingleSignOn Scope="Tenant" KeepAliveInDays="30" />
      <SessionExpiryType>Absolute</SessionExpiryType>
      <SessionExpiryInSeconds>1200</SessionExpiryInSeconds>
    </UserJourneyBehaviors>
    ```

Nous vous recommandons de définir la valeur de SessionExpiryInSeconds sur une courte période (1 200 secondes), tandis que vous pouvez définir la valeur de KeepAliveInDays sur une période relativement longue (30 jours), comme l’illustre l’exemple suivant :

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <UserJourneyBehaviors>
    <SingleSignOn Scope="Tenant" KeepAliveInDays="30" />
    <SessionExpiryType>Absolute</SessionExpiryType>
    <SessionExpiryInSeconds>1200</SessionExpiryInSeconds>
  </UserJourneyBehaviors>
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
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```

::: zone-end

## <a name="sign-out"></a>Se déconnecter

Quand vous souhaitez déconnecter l’utilisateur de l’application, il ne suffit pas de supprimer les cookies de l’application ou d’arrêter la session de l’utilisateur. Vous devez rediriger l’utilisateur vers Azure AD B2C pour le déconnecter. Autrement, l’utilisateur peut être en mesure de se réauthentifier auprès de votre application sans entrer à nouveau ses informations d’identification.

Lors d’une demande de déconnexion, Azure AD B2C effectue les opérations suivantes :

1. Invalide la session Azure AD B2C basée sur un cookie.
::: zone pivot="b2c-user-flow"
2. Tente de se déconnecter des fournisseurs d’identité fédérés
::: zone-end
::: zone pivot="b2c-custom-policy"
3. Tente de se déconnecter des fournisseurs d’identité fédérés :
   - OpenId Connect : si le point de terminaison de configuration connu du fournisseur d’identité spécifie un emplacement `end_session_endpoint`.
   - OAuth2 : si les [métadonnées du fournisseur d’identité](oauth2-technical-profile.md#metadata) contiennent l’emplacement `end_session_endpoint`.
   - SAML : si les [métadonnées du fournisseur d’identité](saml-identity-provider-technical-profile.md#metadata) contiennent l’emplacement `SingleLogoutService`.
4. Si vous le souhaitez, se déconnecte d’autres applications. Pour plus d’informations, consultez la section [Déconnexion unique](#single-sign-out).

> [!NOTE]
> Vous pouvez désactiver la déconnexion des fournisseurs d’identité fédérés en définissant les métadonnées du profil technique du fournisseur d’identité `SingleLogoutEnabled` sur `false`.
::: zone-end

La déconnexion efface l’état d’authentification unique de l’utilisateur auprès d’Azure AD B2C, mais il est possible qu’elle ne déconnecte pas l’utilisateur de sa session de fournisseur d’identité sociale. Si l’utilisateur sélectionne le même fournisseur d’identité lors d’une connexion ultérieure, il pourrait se réauthentifier sans avoir à entrer ses informations d’identification. Si un utilisateur veut se déconnecter de l’application, cela ne signifie pas nécessairement qu’il souhaite se déconnecter de son compte Facebook. Toutefois, si les comptes locaux sont utilisés, la session de l’utilisateur se termine correctement.

::: zone pivot="b2c-custom-policy"

### <a name="single-sign-out"></a>Déconnexion unique 

Lorsque vous redirigez l’utilisateur vers le point de terminaison de déconnexion Azure AD B2C (pour les protocoles OAuth2 et SAML), Azure AD B2C efface la session de l’utilisateur du navigateur. Toutefois, l’utilisateur pourrait rester connecté à d’autres applications utilisant Azure AD B2C pour l’authentification. Pour permettre à ces applications de déconnecter l’utilisateur simultanément, Azure AD B2C envoie une requête HTTP GET à la `LogoutUrl` inscrite de toutes les applications auxquelles l’utilisateur est actuellement connecté.

Les applications doivent répondre à cette requête en effaçant toute session qui identifie l’utilisateur et en renvoyant une réponse `200`. Si vous souhaitez prendre en charge la déconnexion unique dans votre application, vous devez implémenter une `LogoutUrl` dans le code de votre application. 

Pour prendre en charge la déconnexion unique, les profils techniques de l’émetteur de jetons pour JWT et SAML doivent spécifier les éléments suivants :

- Le nom du protocole, tel que `<Protocol Name="OpenIdConnect" />`
- La référence au profil technique de session, tel que `UseTechnicalProfileForSessionManagement ReferenceId="SM-OAuth-issuer" />`.

L’exemple suivant illustre les émetteurs de jetons JWT et SAML avec la fonctionnalité de déconnexion unique :

```xml
<ClaimsProvider>
  <DisplayName>Local Account SignIn</DisplayName>
  <TechnicalProfiles>
    <!-- JWT Token Issuer -->
    <TechnicalProfile Id="JwtIssuer">
      <DisplayName>JWT token Issuer</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <OutputTokenFormat>JWT</OutputTokenFormat>
      ...    
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-OAuth-issuer" />
    </TechnicalProfile>

    <!-- Session management technical profile for OIDC based tokens -->
    <TechnicalProfile Id="SM-OAuth-issuer">
      <DisplayName>Session Management Provider</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.OAuthSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
    </TechnicalProfile>

    <!--SAML token issuer-->
    <TechnicalProfile Id="Saml2AssertionIssuer">
      <DisplayName>SAML token issuer</DisplayName>
      <Protocol Name="SAML2" />
      <OutputTokenFormat>SAML2</OutputTokenFormat>
      ...
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-Saml-issuer" />
    </TechnicalProfile>

    <!-- Session management technical profile for SAML based tokens -->
    <TechnicalProfile Id="SM-Saml-issuer">
      <DisplayName>Session Management Provider</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.SamlSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

::: zone-end

### <a name="secure-your-logout-redirect"></a>Sécuriser la redirection de déconnexion

Après la déconnexion, l’utilisateur est redirigé vers l’URI spécifié dans le paramètre `post_logout_redirect_uri`, quelles que soient les URL de réponse qui ont été spécifiées pour l’application. Cependant, si un `id_token_hint` valide est transmis et que l’option **Exiger un jeton d’ID dans les demandes de déconnexion** est activée, Azure AD B2C vérifie que la valeur de `post_logout_redirect_uri` correspond à l’un des URI de redirection configurés de l’application avant d’effectuer la redirection. Si aucune URL de réponse correspondante n’a été configurée pour l’application, un message d’erreur s’affiche et l’utilisateur n’est pas redirigé. 

::: zone pivot="b2c-user-flow"

Pour exiger un jeton d’ID dans les demandes de déconnexion :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Veillez à utiliser l’annuaire qui contient votre locataire Azure AD B2C en sélectionnant le filtre **Annuaire et abonnement** dans le menu supérieur et en choisissant l’annuaire qui contient votre locataire Azure AD B2C.
1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Flux d’utilisateurs**.
1. Ouvrez le flux utilisateur que vous avez créé précédemment.
1. Sélectionner **Propriétés**.
1. Activez l’option **Exiger un jeton d’ID dans les demandes de déconnexion**.
1. Revenez à **Azure AD B2C**.
1. Sélectionnez **Inscriptions d’applications**, puis sélectionnez votre application.
1. Sélectionnez **Authentification**.
1. Dans la zone de texte **URL de déconnexion**, saisissez votre URI de redirection après déconnexion, puis sélectionnez **Enregistrer**.

::: zone-end

::: zone pivot="b2c-custom-policy"

Pour exiger un jeton d’ID dans les demandes de déconnexion, ajoutez un élément **UserJourneyBehaviors** à l’intérieur de l’élément [RelyingParty](relyingparty.md). Définissez ensuite **EnforceIdTokenHintOnLogout** dans l’élément **SingleSignOn** sur `true`. Votre élément **UserJourneyBehaviors** devrait ressembler à l’exemple suivant :

```xml
<UserJourneyBehaviors>
  <SingleSignOn Scope="Tenant" EnforceIdTokenHintOnLogout="true"/>
</UserJourneyBehaviors>
```

::: zone-end

Pour configurer l’URL de déconnexion de votre application :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Veillez à utiliser l’annuaire qui contient votre locataire Azure AD B2C en sélectionnant le filtre **Annuaire et abonnement** dans le menu supérieur et en choisissant l’annuaire qui contient votre locataire Azure AD B2C.
1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Inscriptions d’applications**, puis sélectionnez votre application.
1. Sélectionnez **Authentification**.
1. Dans la zone de texte **URL de déconnexion**, saisissez votre URI de redirection après déconnexion, puis sélectionnez **Enregistrer**.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [configurer les jetons dans Azure AD B2C](configure-tokens.md).
