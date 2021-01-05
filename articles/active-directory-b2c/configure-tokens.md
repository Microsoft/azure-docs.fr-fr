---
title: Configurer les jetons – Azure Active Directory B2C | Microsoft Docs
description: Découvrez comment configurer les paramètres de compatibilité des jetons et leur durée de vie dans Azure Active Directory B2C.
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
ms.openlocfilehash: a4299eff296d9795f8d256ff1236a8e8b4ad3e42
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97585205"
---
# <a name="configure-tokens-in-azure-active-directory-b2c"></a>Configurer les jetons dans Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

Dans cet article, vous allez apprendre à configurer la [durée de vie et la compatibilité d’un jeton](tokens-overview.md) dans Azure Active Directory B2C (Azure AD B2C).

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="token-lifetime-behavior"></a>Comportement de durée de vie de jeton

Vous pouvez configurer la durée de vie d’un jeton, à savoir :

- **Durées de vie des jetons d’accès et d’ID (minutes)**  : durée de vie du jeton du porteur OAuth 2.0 et des jetons d’ID. La valeur par défaut est de 60 minutes (1 heure). La valeur minimale (inclusive) est de 5 minutes. La valeur maximale (inclusive) est de 1 440 minutes (24 heures).
- **Durée de vie du jeton d’actualisation (jours)**  : période maximale avant laquelle un jeton d’actualisation peut être utilisé pour acquérir un nouveau jeton d’accès si l’étendue `offline_access` a été accordée à votre application. La valeur par défaut est de 14 jours. La valeur minimale (inclusive) est de 1 jour. La valeur maximale (inclusive) est de 90 jours.
- **Durée de vie de la fenêtre glissante du jeton d’actualisation** : type de fenêtre glissante du jeton d’actualisation. `Bounded` indique que le jeton d’actualisation peut être étendu comme spécifié dans la **Durée de vie (jours)** . `No expiry` indique que la durée de vie de la fenêtre glissante du jeton d’actualisation n’expire jamais.
- **Longueur de la durée de vie (jours)**  : une fois cette période écoulée, l’utilisateur est obligé de s’authentifier de nouveau, quelle que soit la période de validité du dernier jeton d’actualisation acquis par l’application. La valeur doit être supérieure ou égale à la valeur de **Durée de vie du jeton d’actualisation**.

Le diagramme suivant montre le comportement de durée de vie de la fenêtre glissante du jeton d’actualisation.

![Durée de vie du jeton d’actualisation](./media/configure-tokens/refresh-token-lifetime.png)

> [!NOTE]
> Les applications monopages utilisant le flux de code d’autorisation avec PKCE ont toujours une durée de vie de jeton d’actualisation de 24 heures. [En savoir plus sur les implications pour la sécurité des jetons d’actualisation dans le navigateur](../active-directory/develop/reference-third-party-cookies-spas.md#security-implications-of-refresh-tokens-in-the-browser).

## <a name="configure-token-lifetime"></a>Configurer la durée de vie des jetons

::: zone pivot="b2c-user-flow"

Pour configurer la durée de vie du jeton de votre flux utilisateur :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Veillez à bien utiliser l’annuaire qui contient votre locataire Azure AD B2C. Sélectionnez le filtre **Annuaire + abonnement** dans le menu du haut, puis choisissez l’annuaire qui contient votre locataire Azure AD B2C.
1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Flux utilisateur (stratégies)** .
1. Ouvrez le flux utilisateur que vous avez créé précédemment.
1. Sélectionner **Propriétés**.
1. Sous **Durée de vie du jeton**, ajustez les propriétés en fonction des besoins de votre application.
1. Cliquez sur **Enregistrer**.

::: zone-end

::: zone pivot="b2c-custom-policy"

Pour modifier les paramètres de compatibilité de votre jeton, définissez les métadonnées de profil technique de l’[émetteur de jeton ](jwt-issuer-technical-profile.md) dans l’extension, ou le fichier de partie de confiance de la stratégie qui doit être affectée. Le profil technique d’émetteur de jeton ressemble à l’exemple suivant :

```xml
<ClaimsProviders>
  <ClaimsProvider>
    <DisplayName>Token Issuer</DisplayName>
    <TechnicalProfiles>
      <TechnicalProfile Id="JwtIssuer">
        <Metadata>
          <Item Key="token_lifetime_secs">3600</Item>
          <Item Key="id_token_lifetime_secs">3600</Item>
          <Item Key="refresh_token_lifetime_secs">1209600</Item>
          <Item Key="rolling_refresh_token_lifetime_secs">7776000</Item>
          <!--<Item Key="allow_infinite_rolling_refresh_token">true</Item>-->
          <Item Key="IssuanceClaimPattern">AuthorityAndTenantGuid</Item>
          <Item Key="AuthenticationContextReferenceClaimPattern">None</Item>
        </Metadata>
      </TechnicalProfile>
    </TechnicalProfiles>
  </ClaimsProvider>
</ClaimsProviders>
```

Les valeurs suivantes sont définies dans l’exemple précédent :

- **token_lifetime_secs** : durées de vie de jeton d’accès (secondes). La valeur par défaut est 3 600 (1 heure). La valeur minimale est 300 (5 minutes). La valeur maximale est 86 400 (24 heures). 
- **id_token_lifetime_secs** : durées de vie de jeton d’ID (secondes). La valeur par défaut est 3 600 (1 heure). La valeur minimale est 300 (5 minutes). La valeur maximale est 86 400 (24 heures). 
- **refresh_token_lifetime_secs** : durée de vie de jeton d’actualisation (secondes). La valeur par défaut est 1 209 600 (14 jours). La valeur minimale est 86 400 (24 heures). La valeur maximale est 7 776 000 jours (90 ans). 
- **rolling_refresh_token_lifetime_secs** : durée de vie de la fenêtre glissante du jeton d’actualisation (secondes). La valeur par défaut est 7 776 000 (90 jours). La valeur minimale est 86 400 (24 heures). Le maximum est 31 536 000 (365 jours). Si vous ne souhaitez pas appliquer une durée de vie de fenêtre glissante, définissez la valeur de `allow_infinite_rolling_refresh_token` sur `true`. 
- **allow_infinite_rolling_refresh_token** : la durée de vie de la fenêtre glissante du jeton d’actualisation n’expire jamais. 

::: zone-end


## <a name="token-compatibility-settings"></a>Paramètres de conformité de jeton

Vous pouvez configurer la compatibilité du jeton, à savoir :

- **Revendication de l’émetteur (iss)**  : format de l’émetteur du jeton d’accès et d’ID.
- **Revendication d’objet (obj)**  : objet principal sur lequel portent les assertions d’informations du jeton, comme l’utilisateur d’une application. Cette valeur est immuable et ne peut pas être réattribuée ou réutilisée. Vous pouvez l’utiliser pour effectuer des vérifications d’autorisation en toute sécurité, comme lorsque le jeton est utilisé pour accéder à une ressource. Par défaut, la revendication de l’objet est remplie avec l’ID d’objet de l’utilisateur dans le répertoire.
- **Revendication représentant le workflow utilisateur** : cette revendication identifie le flux utilisateur exécuté. Valeurs possibles : `tfp` (par défaut) ou `acr`.

::: zone pivot="b2c-user-flow"

Pour configurer les paramètres de compatibilité de votre flux utilisateur :

1. Sélectionnez **Flux utilisateur (stratégies)** .
1. Ouvrez le flux utilisateur que vous avez créé précédemment.
1. Sélectionner **Propriétés**.
1. Sous **Paramètres de compatibilité de jeton**, ajustez les propriétés en fonction des besoins de votre application.
1. Cliquez sur **Enregistrer**.

::: zone-end

::: zone pivot="b2c-custom-policy"

Pour modifier les paramètres de compatibilité de votre jeton, définissez les métadonnées de profil technique de l’[émetteur de jeton ](jwt-issuer-technical-profile.md) dans l’extension, ou le fichier de partie de confiance de la stratégie qui doit être affectée. Le profil technique d’émetteur de jeton ressemble à l’exemple suivant :

```xml
<ClaimsProviders>
  <ClaimsProvider>
    <DisplayName>Token Issuer</DisplayName>
    <TechnicalProfiles>
      <TechnicalProfile Id="JwtIssuer">
        <Metadata>
          ...
          <Item Key="IssuanceClaimPattern">AuthorityAndTenantGuid</Item>
          <Item Key="AuthenticationContextReferenceClaimPattern">None</Item>
        </Metadata>
      </TechnicalProfile>
    </TechnicalProfiles>
  </ClaimsProvider>
</ClaimsProviders>
```

- **Revendication de l’émetteur (iss)** : la revendication de l’émetteur (iss) est définie avec l’élément de métadonnées **IssuanceClaimPattern**. Les valeurs possibles sont `AuthorityAndTenantGuid` et `AuthorityWithTfp`.
- **Paramétrage de l’ID de stratégie représentant la revendication** : les options permettant de définir cette valeur sont `TFP` (Trust Framework Policy) et `ACR` (Authentication Context Reference). `TFP` est la valeur recommandée. Définissez **AuthenticationContextReferenceClaimPattern** avec la valeur de `None`.

    Dans l'élément **ClaimsSchema**, ajoutez l'élément suivant :

    ```xml
    <ClaimType Id="trustFrameworkPolicy">
      <DisplayName>Trust framework policy name</DisplayName>
      <DataType>string</DataType>
    </ClaimType>
    ```

    Dans votre élément **OutputClaims**, ajoutez l'élément suivant :

    ```xml
    <OutputClaim ClaimTypeReferenceId="trustFrameworkPolicy" Required="true" DefaultValue="{policy}" />
    ```

    Pour l’option ACR, supprimez l’élément **AuthenticationContextReferenceClaimPattern**.

- **Revendication de sujet (sub)** : cette option est définie par défaut sur ObjectID. Si vous souhaitez que cette option soit définie sur `Not Supported`, remplacez cette ligne :

    ```xml
    <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub" />
    ```

    par cette ligne :

    ```xml
    <OutputClaim ClaimTypeReferenceId="sub" />
    ```

::: zone-end

## <a name="provide-optional-claims-to-your-app"></a>Fournir des revendications facultatives à votre application

Les revendications d’application sont des valeurs qui sont retournées à l’application. Mettez à jour votre flux d’utilisateurs pour qu’il contienne les revendications souhaitées.

::: zone pivot="b2c-user-flow"

1. Sélectionnez **Flux utilisateur (stratégies)** .
1. Ouvrez le flux utilisateur que vous avez créé précédemment.
1. Cliquez sur **Revendications de l’application**.
1. Choisissez les revendications et les attributs que vous souhaitez renvoyer à votre application.
1. Cliquez sur **Enregistrer**.

::: zone-end

::: zone pivot="b2c-custom-policy"

Les revendications de sortie du [profil technique de la stratégie de partie de confiance](relyingparty.md#technicalprofile) sont des valeurs qui sont retournées à une application. L’ajout de revendications de sortie émettra les revendications dans le jeton après un parcours utilisateur réussi et les enverra à l’application. Modifiez l’élément de profil technique dans la section de partie de confiance pour ajouter les revendications souhaitées en tant que revendication de sortie.

1. Ouvrez votre fichier de stratégie personnalisée. Par exemple SignUpOrSignin.xml.
1. Recherchez l’élément OutputClaims. Ajoutez l’élément OutputClaim que vous souhaitez inclure dans le jeton. 
1. Définissez les attributs de la revendication de sortie. 

L’exemple suivant ajoute la revendication `accountBalance`. La revendication accountBalance est envoyée à l’application sous forme de solde. 

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
      <!--Add the optional claims here-->
      <OutputClaim ClaimTypeReferenceId="accountBalance" DefaultValue="" PartnerClaimType="balance" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```

L’élément OutputClaim contient les attributs suivants :

- **ClaimTypeReferenceId** : Identificateur d’un type de revendication déjà défini dans la section [ClaimsSchema](claimsschema.md) du fichier de stratégie ou d’un fichier de stratégie parent.
- **PartnerClaimType** : Vous permet de modifier le nom de la revendication dans le jeton. 
- **DefaultValue** : Valeur par défaut. Vous pouvez également définir la valeur par défaut sur un [programme de résolution de revendications](claim-resolver-overview.md), par exemple l’ID de locataire.
- **AlwaysUseDefaultValue** : force l’utilisation de la valeur par défaut.

::: zone-end

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [demander des jetons d’accès](access-tokens.md).