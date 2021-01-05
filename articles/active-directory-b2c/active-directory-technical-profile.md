---
title: Définir un profil technique Azure AD dans une stratégie personnalisée
titleSuffix: Azure AD B2C
description: Définissez un profil technique Azure Active Directory dans une stratégie personnalisée dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 12/11/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 3f92bfe98a45117264c957481a75493de652abc9
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97508099"
---
# <a name="define-an-azure-active-directory-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>Définir un profil technique Azure Active Directory dans une stratégie personnalisée Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) prend en charge la gestion des utilisateurs Azure Active Directory. Cet article décrit les caractéristiques d’un profil technique permettant d’interagir avec un fournisseur de revendications qui prend en charge ce protocole normalisé.

## <a name="protocol"></a>Protocol

L’attribut **Name** de l’élément **Protocol** doit être défini sur `Proprietary`. L’attribut **handler** doit contenir le nom complet de l’assembly du gestionnaire de protocole`Web.TPEngine.Providers.AzureActiveDirectoryProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null`.

Faisant suite au [pack de démarrage de stratégie personnalisée](custom-policy-get-started.md#custom-policy-starter-pack), les profils techniques Azure AD incluent le profil technique **AAD-Common**. Les profils techniques Azure AD ne spécifient pas le protocole parce que celui-ci est configuré dans le profil technique **AAD-Common** :
 
- **AAD-UserReadUsingAlternativeSecurityId** et **UserReadUsingAlternativeSecurityId-AAD-NoError** : rechercher un compte de réseau social dans le répertoire.
- **AAD-UserWriteUsingAlternativeSecurityId** : créer un compte de réseau social.
- **AAD-UserReadUsingEmailAddress** : rechercher un compte local dans le répertoire.
- **AAD-UserWriteUsingLogonEmail** : créer un compte local.
- **AAD-UserWritePasswordUsingObjectId** : mettre à jour un mot de passe de compte local.
- **AAD-UserWriteProfileUsingObjectId** : mettre à jour un profil utilisateur d’un compte local ou social.
- **AAD-UserReadUsingObjectId** : lire un profil utilisateur d’un compte local ou social.
- **AAD-UserWritePhoneNumberUsingObjectId** : écrire le numéro de téléphone d’authentification multifacteur d’un compte local ou social.

L’exemple suivant montre le profil technique **AAD-Common** :

```xml
<TechnicalProfile Id="AAD-Common">
  <DisplayName>Azure Active Directory</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.AzureActiveDirectoryProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />

  <CryptographicKeys>
    <Key Id="issuer_secret" StorageReferenceId="B2C_1A_TokenSigningKeyContainer" />
  </CryptographicKeys>

  <!-- We need this here to suppress the SelfAsserted provider from invoking SSO on validation profiles. -->
  <IncludeInSso>false</IncludeInSso>
  <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
</TechnicalProfile>
```

## <a name="inputclaims"></a>InputClaims

L’élément InputClaims contient une revendication qui est utilisée pour rechercher un compte dans l’annuaire, ou pour en créer un. Il doit y avoir exactement un élément InputClaim dans la collection de revendications d’entrée pour tous les profils techniques Azure AD. Il se peut que vous deviez mapper le nom de la revendication définie dans votre stratégie au nom défini dans Azure Active Directory.

Pour lire, mettre à jour ou supprimer un compte d’utilisateur existant, la revendication d’entrée est une clé qui identifie de façon unique le compte dans l’annuaire Azure AD. Par exemple, **objectId**, **userPrincipalName**, **signInNames.emailAddress**, **signInNames.userName** ou **alternativeSecurityId**. 

Pour créer un compte d’utilisateur, la revendication d’entrée est une clé qui identifie de façon unique un compte local ou fédéré. Par exemple, un compte local : **signInNames.emailAddress** ou **signInNames.userName**. Pour un compte fédéré : **alternativeSecurityId**.

L’élément [InputClaimsTransformations](technicalprofiles.md#input-claims-transformations) peut contenir une collection d’éléments de transformation de revendications d’entrée qui sont utilisés pour modifier la revendication d’entrée ou en générer une nouvelle.

## <a name="outputclaims"></a>OutputClaims

L’élément **OutputClaims** contient une liste de revendications retournée par le profil technique Azure AD. Il se peut que vous deviez mapper le nom de la revendication définie dans votre stratégie au nom défini dans Azure Active Directory. Vous pouvez également inclure des revendications qui ne sont pas retournées par Azure Active Directory, pour autant que vous définissiez l’attribut `DefaultValue`.

L’élément [OutputClaimsTransformations](technicalprofiles.md#output-claims-transformations) peut contenir une collection d’éléments **OutputClaimsTransformation** qui sont utilisés pour modifier les revendications de sortie ou en générer de nouvelles.

Par exemple, le profil technique **AAD-UserWriteUsingLogonEmail** crée un compte local et retourne les revendications suivantes :

- **objectId**, qui est l’identificateur du nouveau compte
- **newUser**, qui indique si l’utilisateur est nouveau
- **authenticationSource**, qui définit l’authentification sur `localAccountAuthentication`
- **userPrincipalName**, qui est le nom d’utilisateur principal du nouveau compte
- **signInNames.emailAddress**, qui est le nom de connexion du compte, similaire à la revendication d’entrée **email**

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="objectId" />
  <OutputClaim ClaimTypeReferenceId="newUser" PartnerClaimType="newClaimsPrincipalCreated" />
  <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="localAccountAuthentication" />
  <OutputClaim ClaimTypeReferenceId="userPrincipalName" />
  <OutputClaim ClaimTypeReferenceId="signInNames.emailAddress" />
</OutputClaims>
```

## <a name="persistedclaims"></a>PersistedClaims

L’élément **PersistedClaims** contient toutes les valeurs qui doivent être conservées par Azure AD, avec des informations de mappage possible entre un type de revendication déjà défini à la section [ClaimsSchema](claimsschema.md) dans la stratégie et le nom d’attribut Azure AD.

Le profil technique **AAD-UserWriteUsingLogonEmail**, qui crée un nouveau compte local, conserve les revendications suivantes :

```xml
  <PersistedClaims>
    <!-- Required claims -->
    <PersistedClaim ClaimTypeReferenceId="email" PartnerClaimType="signInNames.emailAddress" />
    <PersistedClaim ClaimTypeReferenceId="newPassword" PartnerClaimType="password"/>
    <PersistedClaim ClaimTypeReferenceId="displayName" DefaultValue="unknown" />
    <PersistedClaim ClaimTypeReferenceId="passwordPolicies" DefaultValue="DisablePasswordExpiration" />

    <!-- Optional claims. -->
    <PersistedClaim ClaimTypeReferenceId="givenName" />
    <PersistedClaim ClaimTypeReferenceId="surname" />
  </PersistedClaims>
```

Le nom de la revendication est le nom de l’attribut Azure AD, sauf si l’attribut **PartnerClaimType** est spécifié, qui contient le nom d’attribut Azure AD.

## <a name="requirements-of-an-operation"></a>Exigences d’une opération

- Il doit y avoir exactement un élément **InputClaim** dans le panier de revendications pour tous les profils techniques Azure AD.
- [L’article sur les attributs de profil utilisateur](user-profile-attributes.md) décrit les attributs de profil utilisateur Azure AD B2C pris en charge que vous pouvez utiliser dans les revendications d’entrée, les revendications de sortie et les revendications persistantes. 
- Si l’opération est `Write` ou `DeleteClaims`, il doit également apparaître dans un élément **PersistedClaims**.
- La valeur de la revendication **userPrincipalName** doit être au format `user@tenant.onmicrosoft.com`.
- La revendication **displayName** est obligatoire et ne peut pas être une chaîne vide.

## <a name="azure-ad-technical-provider-operations"></a>Opérations du fournisseur technique AD Azure

### <a name="read"></a>Lire

L’opération **Read** lit les données sur un seul compte d’utilisateur. Le profil technique suivant lit les données d’un compte d’utilisateur à l’aide de l’objectId de l’utilisateur :

```xml
<TechnicalProfile Id="AAD-UserReadUsingObjectId">
  <Metadata>
    <Item Key="Operation">Read</Item>
    <Item Key="RaiseErrorIfClaimsPrincipalDoesNotExist">true</Item>
  </Metadata>
  <IncludeInSso>false</IncludeInSso>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="objectId" Required="true" />
  </InputClaims>
  <OutputClaims>

    <!-- Required claims -->
    <OutputClaim ClaimTypeReferenceId="strongAuthenticationPhoneNumber" />

    <!-- Optional claims -->
    <OutputClaim ClaimTypeReferenceId="signInNames.emailAddress" />
    <OutputClaim ClaimTypeReferenceId="displayName" />
    <OutputClaim ClaimTypeReferenceId="otherMails" />
    <OutputClaim ClaimTypeReferenceId="givenName" />
    <OutputClaim ClaimTypeReferenceId="surname" />
  </OutputClaims>
  <IncludeTechnicalProfile ReferenceId="AAD-Common" />
</TechnicalProfile>
```

### <a name="write"></a>Write

L’opération **Write** crée ou met à jour un seul compte d’utilisateur. Le profil technique suivant crée un compte de réseau social :

```xml
<TechnicalProfile Id="AAD-UserWriteUsingAlternativeSecurityId">
  <Metadata>
    <Item Key="Operation">Write</Item>
    <Item Key="RaiseErrorIfClaimsPrincipalAlreadyExists">true</Item>
    <Item Key="UserMessageIfClaimsPrincipalAlreadyExists">You are already registered, please press the back button and sign in instead.</Item>
  </Metadata>
  <IncludeInSso>false</IncludeInSso>
  <InputClaimsTransformations>
    <InputClaimsTransformation ReferenceId="CreateOtherMailsFromEmail" />
  </InputClaimsTransformations>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="AlternativeSecurityId" PartnerClaimType="alternativeSecurityId" Required="true" />
  </InputClaims>
  <PersistedClaims>
    <!-- Required claims -->
    <PersistedClaim ClaimTypeReferenceId="alternativeSecurityId" />
    <PersistedClaim ClaimTypeReferenceId="userPrincipalName" />
    <PersistedClaim ClaimTypeReferenceId="mailNickName" DefaultValue="unknown" />
    <PersistedClaim ClaimTypeReferenceId="displayName" DefaultValue="unknown" />

    <!-- Optional claims -->
    <PersistedClaim ClaimTypeReferenceId="otherMails" />
    <PersistedClaim ClaimTypeReferenceId="givenName" />
    <PersistedClaim ClaimTypeReferenceId="surname" />
  </PersistedClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="objectId" />
    <OutputClaim ClaimTypeReferenceId="newUser" PartnerClaimType="newClaimsPrincipalCreated" />
    <OutputClaim ClaimTypeReferenceId="otherMails" />
  </OutputClaims>
  <IncludeTechnicalProfile ReferenceId="AAD-Common" />
  <UseTechnicalProfileForSessionManagement ReferenceId="SM-AAD" />
</TechnicalProfile>
```

### <a name="deleteclaims"></a>DeleteClaims

L’opération **DeleteClaims** efface les informations d’une liste fournie de revendications. Le profil technique suivant supprime les revendications :

```xml
<TechnicalProfile Id="AAD-DeleteClaimsUsingObjectId">
  <Metadata>
    <Item Key="Operation">DeleteClaims</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="objectId" Required="true" />
  </InputClaims>
  <PersistedClaims>
    <PersistedClaim ClaimTypeReferenceId="objectId" />
    <PersistedClaim ClaimTypeReferenceId="Verified.strongAuthenticationPhoneNumber" PartnerClaimType="strongAuthenticationPhoneNumber" />
  </PersistedClaims>
  <OutputClaims />
  <IncludeTechnicalProfile ReferenceId="AAD-Common" />
</TechnicalProfile>
```

### <a name="deleteclaimsprincipal"></a>DeleteClaimsPrincipal

L’opération **DeleteClaimsPrincipal** supprime un seul compte d’utilisateur de l’annuaire. Le profil technique suivant supprime un compte d’utilisateur du répertoire en utilisant le nom d’utilisateur principal :

```xml
<TechnicalProfile Id="AAD-DeleteUserUsingObjectId">
  <Metadata>
    <Item Key="Operation">DeleteClaimsPrincipal</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="objectId" Required="true" />
  </InputClaims>
  <OutputClaims/>
  <IncludeTechnicalProfile ReferenceId="AAD-Common" />
</TechnicalProfile>
```

Le profil technique suivant supprime un compte d’utilisateur social en utilisant **alternativeSecurityId** :

```xml
<TechnicalProfile Id="AAD-DeleteUserUsingAlternativeSecurityId">
  <Metadata>
    <Item Key="Operation">DeleteClaimsPrincipal</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="alternativeSecurityId" Required="true" />
  </InputClaims>
  <OutputClaims/>
  <IncludeTechnicalProfile ReferenceId="AAD-Common" />
</TechnicalProfile>
```
## <a name="metadata"></a>Métadonnées

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Opération | Oui | Opération à effectuer. Valeurs possibles : `Read`, `Write`, `DeleteClaims` ou `DeleteClaimsPrincipal`. |
| RaiseErrorIfClaimsPrincipalDoesNotExist | Non | Génère une erreur si l’objet utilisateur n’existe pas dans le répertoire. Valeurs possibles : `true` ou `false`. |
| RaiseErrorIfClaimsPrincipalAlreadyExists | Non | Génère une erreur si l’objet utilisateur existe déjà. Valeurs possibles : `true` ou `false`.|
| ApplicationObjectId | Non | Identificateur d’objet d’application pour les attributs d’extension. Valeur : ObjectId d'une application. Pour plus d’informations, consultez [Utiliser des attributs personnalisés](user-flow-custom-attributes.md?pivots=b2c-custom-policy). |
| ClientId | Non | Identificateur client pour accéder au locataire en tant que tiers. Pour plus d’informations, voir [Utiliser des attributs personnalisés dans une stratégie personnalisée de modification de profil](user-flow-custom-attributes.md?pivots=b2c-custom-policy). |
| IncludeClaimResolvingInClaimsHandling  | Non | Pour les revendications d’entrée et de sortie, spécifie si la [résolution des revendications](claim-resolver-overview.md) est incluse dans le profil technique. Valeurs possibles : `true` ou `false` (par défaut). Si vous souhaitez utiliser un programme de résolution des revendications dans le profil technique, définissez cette valeur sur `true`. |

### <a name="ui-elements"></a>Éléments d’interface utilisateur
 
Les paramètres suivants peuvent être utilisés pour configurer le message d’erreur affiché en cas d’échec. Les métadonnées doivent être configurées dans le profil technique [autodéclaré](self-asserted-technical-profile.md). Les messages d’erreur peuvent être [localisés](localization.md).

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| UserMessageIfClaimsPrincipalAlreadyExists | Non | Si une erreur doit être déclenchée (voir la description de l’attribut RaiseErrorIfClaimsPrincipalAlreadyExists), spécifiez le message à afficher à l’utilisateur si l’objet utilisateur existe déjà. |
| UserMessageIfClaimsPrincipalDoesNotExist | Non | Si une erreur doit être déclenchée (voir la description de l’attribut RaiseErrorIfClaimsPrincipalDoesNotExist), spécifiez le message à afficher à l’utilisateur si l’objet utilisateur n’existe pas. |


## <a name="next-steps"></a>Étapes suivantes

Consultez l’article suivant pour obtenir un exemple d’utilisation d’un profil technique Azure AD :

- [Ajouter des revendications et personnaliser l’entrée utilisateur avec des stratégies personnalisées dans Azure Active Directory B2C](configure-user-input.md)














