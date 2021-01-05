---
title: Ajouter des attributs utilisateur et personnaliser l’entrée utilisateur
titleSuffix: Azure AD B2C
description: Découvrez comment personnaliser l’entrée utilisateur et ajouter des attributs utilisateur au parcours d’inscription ou de connexion dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/10/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: eb7cba1de280793a1ca98687c71355c1ea702d4c
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97585222"
---
#  <a name="add-user-attributes-and-customize-user-input-in-azure-active-directory-b2c"></a>Ajouter des attributs utilisateur et personnaliser l’entrée utilisateur dans Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

Dans cet article, vous collectez un nouvel attribut lors de votre parcours d’inscription dans Azure Active Directory B2C (Azure AD B2C). Vous obtenez la ville des utilisateurs, vous la configurez en tant que liste déroulante et vous indiquez si elle doit être obligatoirement fournie.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

::: zone pivot="b2c-user-flow"

## <a name="add-user-attributes-your-user-flow"></a>Ajouter des attributs utilisateur à votre flux d’utilisateur

1. Dans votre locataire Azure AD B2C, sélectionnez **Flux d’utilisateur**.
1. Sélectionnez votre stratégie (par exemple, « B2C_1_SignupSignin ») pour l’ouvrir.
1. Sélectionnez **Attributs utilisateur**, puis sélectionnez l’attribut utilisateur (par exemple, « City »). 
1. Sélectionnez **Enregistrer**.

## <a name="provide-optional-claims-to-your-app"></a>Fournir des revendications facultatives à votre application

Les revendications d’application sont des valeurs qui sont retournées à l’application. Mettez à jour votre flux d’utilisateurs pour qu’il contienne les revendications souhaitées.

1. Sélectionnez votre stratégie (par exemple, « B2C_1_SignupSignin ») pour l’ouvrir.
1. Cliquez sur **Revendications de l’application**.
1. Sélectionnez les attributs que vous souhaitez renvoyer à votre application (par exemple, « Ville »).
1. Sélectionnez **Enregistrer**.
 
## <a name="configure-user-attributes-input-type"></a>Configurer le type d’entrée des attributs utilisateur

1. Sélectionnez votre stratégie (par exemple, « B2C_1_SignupSignin ») pour l’ouvrir.
1. Sélectionnez **Mises en page**.
1. Sélectionnez **Page d’inscription à un compte Local**.
1. Sous **Attributs utilisateur**, sélectionnez **Ville**.
    1. Dans la liste déroulante **Type d’entrée utilisateur**, sélectionnez **DropdownSingleSelect**.
    1. Dans la liste déroulante **Facultatif**, sélectionnez **Non**.
1. Sélectionnez **Enregistrer**. 

### <a name="provide-a-list-of-values-by-using-localized-collections"></a>Fournir une liste de valeurs à l’aide de collections localisées

Pour fournir une liste définie de valeurs pour l’attribut Ville : 

1. [Activez la personnalisation de la langue sur le flux d’utilisateur](language-customization.md#support-requested-languages-for-ui_locales).
1. Sélectionnez votre stratégie (par exemple, « B2C_1_SignupSignin ») pour l’ouvrir.
1. Dans la page **Langues** du flux utilisateur, sélectionnez une langue que vous souhaitez personnaliser.
1. Sous **Fichiers de ressources de niveau page**, sélectionnez **Page d’inscription au compte local**.
1. Sélectionnez **Télécharger les valeurs par défaut** (ou **Télécharger les valeurs de remplacements** si vous avez déjà modifié cette langue).
1. Créez un attribut `LocalizedCollections`.

`LocalizedCollections` est un tableau de paires `Name` et `Value`. L’ordre des éléments sera l’ordre dans lequel ils apparaissent. 

* `ElementId` est l’attribut utilisateur auquel cet attribut `LocalizedCollections` apporte une réponse.
* `Name` est la valeur affichée à l’utilisateur.
* `Value` est ce qui est renvoyé dans la revendication lorsque cette option est sélectionnée.

```json
{
  "LocalizedStrings": [...],
  "LocalizedCollections": [
    {
      "ElementType": "ClaimType",
      "ElementId": "city",
      "TargetCollection": "Restriction",
      "Override": false,
      "Items": [
        {
          "Name": "Berlin",
          "Value": "Berlin"
        },
        {
          "Name": "London",
          "Value": "London"
        },
        {
          "Name": "Seattle",
          "Value": "Seattle"
        }
      ]
    }
  ]
}
```

#### <a name="upload-your-changes"></a>Charger vos modifications

1. Une fois que vous avez terminé de modifier votre fichier JSON, revenez à votre client B2C.
1. Sélectionnez **Flux d’utilisateurs** et votre stratégie (par exemple, « B2C_1_SignupSignin ») pour l’ouvrir.
1. Sélectionnez **Langues**.
1. Sélectionnez la langue dans laquelle vous voulez traduire.
1. Sélectionnez la **page d’inscription à un compte local**.
1. Sélectionnez l’icône de dossier et sélectionnez le fichier JSON à charger. Ces changements sont automatiquement enregistrés dans votre flux utilisateur.

## <a name="test-your-user-flow"></a>Tester votre flux d’utilisateur

1. Sélectionnez votre stratégie (par exemple, « B2C_1_SignupSignin ») pour l’ouvrir.
1. Pour tester votre stratégie, sélectionnez **Exécuter le flux d’utilisateur**.
1. Pour **Application**, sélectionnez l’application web *testapp1* que vous avez précédemment inscrite. L’**URL de réponse** doit être `https://jwt.ms`.
1. Cliquez sur **Exécuter le flux d’utilisateur**.

::: zone-end

::: zone pivot="b2c-custom-policy"

> [!NOTE]
> Cet exemple utilise la revendication intégrée « city ». Au lieu de cela, vous pouvez choisir l’un des [attributs intégrés Azure AD B2C](user-profile-attributes.md) pris en charge ou un attribut personnalisé. Pour utiliser un attribut personnalisé, [activez les attributs personnalisés](user-flow-custom-attributes.md). Pour utiliser un autre attribut intégré ou personnalisé, remplacez « city » par l’attribut de votre choix, par exemple l’attribut intégré *jobTitle* ou un attribut personnalisé comme *extension_loyaltyId*.  

Vous pouvez collecter les données initiales auprès des utilisateurs par le biais du parcours utilisateur d’inscription ou de connexion. Des revendications supplémentaires peuvent être collectées par la suite au moyen des parcours utilisateur de modification de profil. Chaque fois qu’Azure AD B2C rassemble de manière interactive des informations provenant directement de l’utilisateur, Identity Experience Framework utilise son [profil technique autodéclaré](self-asserted-technical-profile.md). Dans cet exemple, vous allez :

1. Définir une revendication « city ». 
1. Demander à l’utilisateur la ville où il vit.
1. conserver la ville dans le profil utilisateur de l’annuaire Azure AD B2C ;
1. lire la revendication de ville dans l’annuaire Azure AD B2C à chaque connexion ;
1. retourner la ville à votre application par partie de confiance après vous être connecté ou inscrit.  

## <a name="define-a-claim"></a>Définir une revendication

Une revendication fournit un stockage temporaire de données lors d’une exécution de stratégie Azure AD B2C. Le [schéma de revendications](claimsschema.md) est l’endroit où vous déclarez vos revendications. Les éléments suivants sont utilisés pour définir la revendication :

- **DisplayName** : chaîne qui définit l’étiquette orientée utilisateur.
- [DataType](claimsschema.md#datatype) : type de revendication.
- **UserHelpText** : texte permettant à l’utilisateur de comprendre ce qui est requis.
- [UserInputType](claimsschema.md#userinputtype) : type de contrôle d’entrée, tel qu’une zone de texte, une case d’option, une liste déroulante ou des sélections multiples.

Ouvrez le fichier d’extensions de votre stratégie. Par exemple <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>.

1. Recherchez l’élément [BuildingBlocks](buildingblocks.md). Si l’élément n’existe pas, ajoutez-le.
1. Localisez l’élément [ClaimsSchema](claimsschema.md). Si l’élément n’existe pas, ajoutez-le.
1. Ajoutez la revendication de ville à l’élément **ClaimsSchema**.  

```xml
<ClaimType Id="city">
  <DisplayName>City where you work</DisplayName>
  <DataType>string</DataType>
  <UserInputType>DropdownSingleSelect</UserInputType>
  <Restriction>
    <Enumeration Text="Bellevue" Value="bellevue" SelectByDefault="false" />
    <Enumeration Text="Redmond" Value="redmond" SelectByDefault="false" />
    <Enumeration Text="Kirkland" Value="kirkland" SelectByDefault="false" />
  </Restriction>
</ClaimType>
```

## <a name="add-a-claim-to-the-user-interface"></a>Ajouter une revendication à l’interface utilisateur

Les profils techniques suivants sont [autodéclarés](self-asserted-technical-profile.md), appelés lorsqu’un utilisateur est supposé entrer des informations :

- **LocalAccountSignUpWithLogonEmail** : flux d’inscription de compte local.
- **SelfAsserted-social** : compte fédéré avec première connexion de l’utilisateur.
- **SelfAsserted-ProfileUpdate** :flux de modification de profil.

Pour collecter la revendication de ville au cours de l’inscription, celle-ci doit être ajoutée en tant que revendication de sortie au profil technique `LocalAccountSignUpWithLogonEmail`. Remplacez ce profil technique dans le fichier d’extension. Spécifiez la liste complète des revendications de sortie pour contrôler l’ordre dans lequel les revendications sont présentées à l’écran. Recherchez l’élément **ClaimsProviders**. Ajoutez un nouveau ClaimsProvider comme suit :

```xml
<ClaimsProvider>
  <DisplayName>Local Account</DisplayName>
  <TechnicalProfiles>
    <!--Local account sign-up page-->
    <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
      <OutputClaims>
       <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="Verified.Email" Required="true" />
       <OutputClaim ClaimTypeReferenceId="newPassword" Required="true" />
       <OutputClaim ClaimTypeReferenceId="reenterPassword" Required="true" />
       <OutputClaim ClaimTypeReferenceId="displayName" />
       <OutputClaim ClaimTypeReferenceId="givenName" />
       <OutputClaim ClaimTypeReferenceId="surName" />
       <OutputClaim ClaimTypeReferenceId="city"/>
     </OutputClaims>
   </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

Pour collecter la revendication de ville après la connexion initiale avec un compte fédéré, celle-ci doit être ajoutée en tant que revendication de sortie au profil technique `SelfAsserted-Social`. Pour que les utilisateurs de comptes locaux et de comptes fédérés puissent modifier leurs données de profil ultérieurement, ajoutez la revendication de sortie au profil technique `SelfAsserted-ProfileUpdate`. Remplacez ces profils techniques dans le fichier d’extension. Spécifiez la liste complète des revendications de sortie pour contrôler l’ordre dans lequel les revendications sont présentées à l’écran. Recherchez l’élément **ClaimsProviders**. Ajoutez un nouveau ClaimsProvider comme suit :

```xml
<ClaimsProvider>
  <DisplayName>Self Asserted</DisplayName>
  <TechnicalProfiles>
    <!--Federated account first-time sign-in page-->
    <TechnicalProfile Id="SelfAsserted-Social">
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName"/>
        <OutputClaim ClaimTypeReferenceId="givenName"/>
        <OutputClaim ClaimTypeReferenceId="surname"/>
        <OutputClaim ClaimTypeReferenceId="city"/>
      </OutputClaims>
    </TechnicalProfile>
    <!--Edit profile page-->
    <TechnicalProfile Id="SelfAsserted-ProfileUpdate">
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName"/>
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="city"/>
      </OutputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="read-and-write-a-claim"></a>Lire et écrire une revendication

Les profils techniques suivants sont des [profils techniques Active Directory](active-directory-technical-profile.md) qui lisent et écrivent des données dans Azure Active Directory.  
Dans les profils techniques Active Directory respectifs, utilisez `PersistedClaims` pour écrire des données dans le profil utilisateur, et `OutputClaims` pour lire des données depuis le profil utilisateur.

Remplacez ces profils techniques dans le fichier d’extension. Recherchez l’élément **ClaimsProviders**.  Ajoutez un nouveau ClaimsProvider comme suit :

```xml
<ClaimsProvider>
  <DisplayName>Azure Active Directory</DisplayName>
  <TechnicalProfiles>
    <!-- Write data during a local account sign-up flow. -->
    <TechnicalProfile Id="AAD-UserWriteUsingLogonEmail">
      <PersistedClaims>
        <PersistedClaim ClaimTypeReferenceId="city"/>
      </PersistedClaims>
    </TechnicalProfile>
    <!-- Write data during a federated account first-time sign-in flow. -->
    <TechnicalProfile Id="AAD-UserWriteUsingAlternativeSecurityId">
      <PersistedClaims>
        <PersistedClaim ClaimTypeReferenceId="city"/>
      </PersistedClaims>
    </TechnicalProfile>
    <!-- Write data during edit profile flow. -->
    <TechnicalProfile Id="AAD-UserWriteProfileUsingObjectId">
      <PersistedClaims>
        <PersistedClaim ClaimTypeReferenceId="city"/>
      </PersistedClaims>
    </TechnicalProfile>
    <!-- Read data after user authenticates with a local account. -->
    <TechnicalProfile Id="AAD-UserReadUsingEmailAddress">
      <OutputClaims>  
        <OutputClaim ClaimTypeReferenceId="city" />
      </OutputClaims>
    </TechnicalProfile>
    <!-- Read data after user authenticates with a federated account. -->
    <TechnicalProfile Id="AAD-UserReadUsingObjectId">
      <OutputClaims>  
        <OutputClaim ClaimTypeReferenceId="city" />
      </OutputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="include-a-claim-in-the-token"></a>Inclure une revendication dans le jeton 

Pour retourner la revendication de ville à l’application par partie de confiance, ajoutez une revendication de sortie au fichier <em>`SocialAndLocalAccounts/`**`SignUpOrSignIn.xml`**</em>. Après un parcours utilisateur réussi, la revendication de sortie est ajoutée au jeton et envoyée à l’application. Modifiez l’élément de profil technique dans la section de partie de confiance pour ajouter la ville en tant que revendication de sortie.
 
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
      <OutputClaim ClaimTypeReferenceId="city" DefaultValue="" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```

## <a name="test-the-custom-policy"></a>Tester la stratégie personnalisée

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Veillez à utiliser l’annuaire qui contient votre locataire Azure AD en sélectionnant le filtre **Annuaire + abonnement** dans le menu du haut et en choisissant l’annuaire qui contient votre locataire Azure AD.
3. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Inscriptions d’applications**.
4. Sélectionnez **Infrastructure d’expérience d’identité**.
5. Sélectionnez **Charger une stratégie personnalisée**, puis chargez les deux fichiers de stratégie que vous avez modifiés.
2. Sélectionnez la stratégie d’inscription et de connexion que vous avez chargée, puis cliquez sur le bouton **Exécuter maintenant**.
3. Vous devriez pouvoir vous inscrire à l’aide d’une adresse de messagerie.

::: zone-end

L’écran d’inscription doit ressembler à la capture d’écran suivante :

![Capture d’écran de l’option d’inscription modifiée](./media/configure-user-input/signup-with-city-claim-drop-down-example.png)

Le jeton envoyé à votre application inclut la revendication `city`.

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "X5eXk4xyojNFum1kl2Ytv8dlNP4-c57dO6QGTVBwaNk"
}.{
  "exp": 1583500140,
  "nbf": 1583496540,
  "ver": "1.0",
  "iss": "https://contoso.b2clogin.com/f06c2fe8-709f-4030-85dc-38a4bfd9e82d/v2.0/",
  "aud": "e1d2612f-c2bc-4599-8e7b-d874eaca1ee1",
  "acr": "b2c_1a_signup_signin",
  "nonce": "defaultNonce",
  "iat": 1583496540,
  "auth_time": 1583496540,
  "name": "Emily Smith",
  "email": "joe@outlook.com",
  "given_name": "Emily",
  "family_name": "Smith",
  "city": "Bellevue"
  ...
}
```

::: zone pivot="b2c-custom-policy"

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur l’élément [ClaimsSchema](claimsschema.md) dans la référence IEF.
- Découvrez comment [utiliser des attributs personnalisés dans Azure AD B2C](user-flow-custom-attributes.md).

::: zone-end