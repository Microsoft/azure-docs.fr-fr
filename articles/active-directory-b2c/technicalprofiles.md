---
title: TechnicalProfiles
titleSuffix: Azure AD B2C
description: Spécifiez l’élément TechnicalProfiles d’une stratégie personnalisée dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 12/11/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 99ed7d3ad81202ab6fe67bf52888bbdbf0b28d2a
ms.sourcegitcommit: ea17e3a6219f0f01330cf7610e54f033a394b459
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97387086"
---
# <a name="technicalprofiles"></a>TechnicalProfiles

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Un profil technique fournit une infrastructure avec un mécanisme intégré pour communiquer avec différents types de parties au moyen d’une stratégie personnalisée dans Azure Active Directory B2C (Azure AD B2C). Les profils techniques sont utilisés pour communiquer avec votre locataire Azure AD B2C, créer un utilisateur ou lire un profil utilisateur. Un profil technique peut être déclaré automatiquement pour permettre l’interaction avec l’utilisateur. Par exemple, collecter les informations d’identification de l’utilisateur pour se connecter, puis afficher la page d’inscription ou la page de réinitialisation du mot de passe.

## <a name="type-of-technical-profiles"></a>Types de profils techniques

Un profil technique permet les types de scénarios suivants :

- [Application Insights](application-insights-technical-profile.md) - Envoi de données d’événement à [Application Insights](../azure-monitor/app/app-insights-overview.md).
- [Azure Active Directory](active-directory-technical-profile.md) : fournit une assistance pour la gestion des utilisateurs Azure Active Directory B2C.
- [Azure AD Multi-Factor Authentication](multi-factor-auth-technical-profile.md) : permet de vérifier un numéro de téléphone à l'aide d'Azure AD Multi-Factor Authentication (MFA). 
- [Transformation de revendications](claims-transformation-technical-profile.md) : appelle les transformations de revendications de sortie pour manipuler les valeurs de revendications, valider des revendications ou définir des valeurs par défaut pour un ensemble de revendications de sortie.
- [Indicateur de jeton d’ID](id-token-hint.md) : valide la signature de jeton JWT `id_token_hint`, le nom de l’émetteur et l’audience du jeton, puis extrait la revendication du jeton entrant.
- [Émetteur de jeton JWT](jwt-issuer-technical-profile.md) : émet un jeton JWT qui est retourné à l’application par partie de confiance.
- [OAuth1](oauth1-technical-profile.md) : fédération avec n’importe quel fournisseur d’identité du protocole OAuth 1.0.
- [OAuth2](oauth2-technical-profile.md) : fédération avec n’importe quel fournisseur d’identité du protocole OAuth 2.0.
- [Mot de passe à usage unique](one-time-password-technical-profile.md) : prend en charge la gestion de la génération et de la vérification d’un mot de passe à usage unique.
- [OpenID Connect](openid-connect-technical-profile.md) : fédération avec n'importe quel fournisseur d'identité du protocole OpenID Connect.
- [Facteur de téléphone](phone-factor-technical-profile.md) : prise en charge de l’inscription et de la vérification des numéros de téléphone.
- [Fournisseur RESTful](restful-technical-profile.md) : appel aux services d'API REST, comme la validation de l'entrée utilisateur, l'enrichissement des données utilisateur ou l'intégration avec les applications métier.
- [Fournisseur d’identité SAML](saml-identity-provider-technical-profile.md) : fédération avec n’importe quel fournisseur d’identité du protocole SAML.
- [Émetteur de jeton SAML](saml-issuer-technical-profile.md) : émet un jeton SAML qui est retourné à l’application par partie de confiance.
- [Autodéclaré](self-asserted-technical-profile.md) : interaction avec l’utilisateur. Par exemple, collecter les informations d’identification de l’utilisateur pour se connecter, afficher la page d’inscription ou la réinitialisation du mot de passe.
- [Gestion des sessions](custom-policy-reference-sso.md) : gère différents types de sessions.

## <a name="technical-profile-flow"></a>Flux du profil technique

Tous les types de profils techniques partagent le même concept. Vous pouvez envoyer des revendications d’entrée, exécuter la transformation des revendications et communiquer avec le tiers configuré, comme un fournisseur d’identité, une API REST ou les services d’annuaire Azure AD. Au terme du processus, le profil technique retourne les revendications de sortie et peut exécuter la transformation des revendications de sortie. Le schéma suivant explique comment sont traités les transformations et les mappages référencés dans le profil technique. Quel que soit le tiers avec qui le profil technique interagit, une fois la transformation des revendications exécutée, les revendications de sortie du profil technique sont immédiatement stockées dans le conteneur de revendications.

![Schéma illustrant le workflow du profil technique](./media/technical-profiles/technical-profile-flow.png)

1. **Gestion de session d’authentification unique (SSO)**  : restaure l’état de session du profil technique à l’aide de la [gestion de session d’authentification unique](custom-policy-reference-sso.md).
1. **Transformation de revendications d’entrée** : avant le démarrage du profil technique, Azure AD B2C exécute [la transformation de revendications ] d’entrée (claimstransformations.md).
1. **Revendications d’entrée** : les revendications sont récupérées auprès du conteneur de revendications et sont utilisées pour le profil technique.
1. **Exécution du profil technique** : le profil technique échange les revendications avec le tiers configuré. Par exemple :
    - Redirigez l’utilisateur vers le fournisseur d’identité pour finaliser la connexion. Une fois connecté, l’utilisateur revient et l’exécution du profil technique se poursuit.
    - Appelez une API REST tout en envoyant les paramètres en tant que InputClaims et récupérant les informations en tant que OutputClaims.
    - Créez ou mettez à jour le compte d’utilisateur.
    - Envoie et vérifie le message texte MFA.
1. **Profils techniques de validation** : un [profil technique autodéclaré](self-asserted-technical-profile.md) peut appeler des [profils techniques de validation](validation-technical-profile.md).
1. **Revendications de sortie** : les revendications sont renvoyées au conteneur de revendications. Vous pouvez utiliser ces revendications dans la prochaine étape d’orchestration, ou dans les transformations de revendications de sortie.
1. **Transformations de revendications de sortie** : une fois le profil technique terminé, Azure AD B2C exécute la [transformation de revendications](claimstransformations.md) de sortie. 
1. **Gestion de session d’authentification unique (SSO)**  : conserve les données du profil technique dans la session à l’aide de la [gestion de session d’authentification unique](custom-policy-reference-sso.md).

Un élément **TechnicalProfiles** contient un ensemble de profils techniques pris en charge par le fournisseur de revendication. Chaque fournisseur de revendications doit avoir un ou plusieurs profils techniques déterminant les points de terminaison et les protocoles nécessaires pour communiquer avec lui. Un fournisseur de revendications peut avoir plusieurs profils techniques.

```xml
<ClaimsProvider>
  <DisplayName>Display name</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Technical profile identifier">
      <DisplayName>Display name of technical profile</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        ...
      </Metadata>
      ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

L’élément **TechnicalProfile** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
|---------|---------|---------|
| Id | Oui | Identificateur unique du profil technique. Le profil technique peut être référencé à l’aide de cet identificateur à partir d’autres éléments dans le fichier de stratégie. Par exemple, **OrchestrationSteps** et **ValidationTechnicalProfile**. |

L’élément **TechnicalProfile** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| Domain | 0:1 | Nom de domaine pour le profil technique. Par exemple, si votre profil technique Spécifie le fournisseur d’identité Facebook, le nom de domaine est Facebook.com. |
| DisplayName | 1:1 | Nom d’affichage du profil technique. |
| Description | 0:1 | Description du profil technique. |
| Protocol | 1:1 | Protocole utilisé pour la communication avec l’autre partie. |
| Métadonnées | 0:1 | Collection de paires clé/valeur utilisées par le protocole pour communiquer avec le point de terminaison durant une transaction. |
| InputTokenFormat | 0:1 | Format du jeton d’entrée. Valeurs possibles : `JSON`, `JWT`, `SAML11` ou `SAML2`. La valeur `JWT` représente un JSON Web Token conforme à la spécification IETF. La valeur `SAML11` représente un jeton de sécurité SAML 1.1 conforme à la spécification OASIS.  La valeur `SAML2` représente un jeton de sécurité SAML 2.0 conforme à la spécification OASIS. |
| OutputTokenFormat | 0:1 | Format du jeton de sortie. Valeurs possibles : `JSON`, `JWT`, `SAML11` ou `SAML2`. |
| CryptographicKeys | 0:1 | Liste de clés de chiffrement utilisées dans le profil technique. |
| InputClaimsTransformations | 0:1 | Liste des références précédemment définies aux transformations de revendications qui doivent être exécutées avant l’envoi de toute revendication au fournisseur de revendications ou à la partie de confiance. |
| InputClaims | 0:1 | Liste des références précédemment définies aux types de revendications qui sont pris en tant qu’entrée dans le profil technique. |
| PersistedClaims | 0:1 | Liste des références précédemment définies aux types de revendications qui sont conservés par le fournisseur de revendications, qui a trait au profil technique. |
| DisplayClaims | 0:1 | Liste des références précédemment définies aux types de revendications qui sont présentés par le fournisseur de revendications, qui a trait au [profil technique autodéclaré](self-asserted-technical-profile.md). La fonctionnalité DisplayClaims est actuellement en **préversion**. |
| OutputClaims | 0:1 | Liste des références précédemment définies aux types de revendications qui sont pris en tant que sortie dans le profil technique. |
| OutputClaimsTransformations | 0:1 | Liste des références précédemment définies aux transformations de revendications qui doivent être exécutées après la réception des revendications provenant du fournisseur de revendications. |
| ValidationTechnicalProfiles | 0:n | Liste des références à d’autres profils techniques que le profil technique utilise à des fins de validation. Pour plus d’informations, voir [Profil technique de validation](validation-technical-profile.md)|
| SubjectNamingInfo | 0:1 | Contrôle la production du nom du sujet dans les jetons où le nom du sujet est spécifié séparément à partir de revendications. Par exemple, OAuth ou SAML.  |
| IncludeInSso | 0:1 |  Si l’utilisation de ce profil technique doit appliquer le comportement de l’authentification unique (SSO) pour la session, ou si une interaction explicite est exigée à la place. Cet élément est valide uniquement dans les profils SelfAsserted utilisés dans un profil technique de validation. Valeurs possibles : `true` (par défaut) ou `false`. |
| IncludeClaimsFromTechnicalProfile | 0:1 | Identificateur d’un profil technique à partir duquel vous souhaitez que toutes les revendications d’entrée et de sortie soient ajoutées à ce profil technique. Le profil technique référencé doit être défini dans le même fichier de stratégie. |
| IncludeTechnicalProfile |0:1 | Identificateur d’un profil technique à partir duquel vous souhaitez que toutes les sonnées soient ajoutées à ce profil technique. |
| UseTechnicalProfileForSessionManagement | 0:1 | Autre profil technique à utiliser pour la gestion de session. |
|EnabledForUserJourneys| 0:1 |Contrôle si le profil technique est exécuté dans un parcours utilisateur.  |

## <a name="protocol"></a>Protocol

Le **protocole** spécifie le protocole utilisé pour la communication avec l’autre partie. L’élément **Protocol** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Nom | Oui | Nom d’un protocole valide pris en charge par Azure AD B2C et utilisé dans le cadre du profil technique. Valeurs possibles : `OAuth1`, `OAuth2`, `SAML2`, `OpenIdConnect`, `Proprietary` ou `None`. |
| Handler | Non | Lorsque le nom de protocole est défini sur `Proprietary`, spécifiez le nom complet de l’assembly qu’Azure AD B2C utilise pour déterminer le gestionnaire de protocole. |

## <a name="metadata"></a>Métadonnées

L’élément **Metadata** contient les options de configuration appropriées pour un protocole spécifique. La liste des métadonnées prises en charge est documentée dans la spécification de [profil technique](#type-of-technical-profiles) correspondante. Un élément **Metadata** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| Élément | 0:n | Métadonnées relatives au profil technique. Chaque type de profil technique a un ensemble différent d’éléments de métadonnées. Pour plus d’informations, voir la section consacrée aux types de profils techniques. |

### <a name="item"></a>Élément

L’élément **Item** de l’élément **Metadata** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Clé | Oui | Clé de métadonnées. Consultez chaque [type de profil technique](#type-of-technical-profiles) pour obtenir la liste des éléments de métadonnées. |

L’exemple suivant illustre l’utilisation des métadonnées relatives au [profil technique OAuth2](oauth2-technical-profile.md#metadata).

```xml
<TechnicalProfile Id="Facebook-OAUTH">
  ...
  <Metadata>
    <Item Key="ProviderName">facebook</Item>
    <Item Key="authorization_endpoint">https://www.facebook.com/dialog/oauth</Item>
    <Item Key="AccessTokenEndpoint">https://graph.facebook.com/oauth/access_token</Item>
    <Item Key="HttpBinding">GET</Item>
    <Item Key="UsePolicyInRedirectUri">0</Item>
    ...
  </Metadata>
  ...
</TechnicalProfile>
```

L’exemple suivant illustre l’utilisation des métadonnées relatives au [profil technique d’API REST](restful-technical-profile.md#metadata).

```xml
<TechnicalProfile Id="REST-Validate-Email">
  ...
  <Metadata>
    <Item Key="ServiceUrl">https://api.sendgrid.com/v3/mail/send</Item>
    <Item Key="AuthenticationType">Bearer</Item>
    <Item Key="SendClaimsIn">Body</Item>
    ...
  </Metadata>
  ...
</TechnicalProfile>
```

## <a name="cryptographic-keys"></a>Clés de chiffrement

Azure AD B2C stocke les secrets et les certificats sous forme de [clés de stratégie](policy-keys-overview.md) pour établir une relation de confiance avec les services auxquels il s’intègre. Au cours de l’exécution du profil technique, Azure AD B2C récupère les clés de chiffrement à partir des clés de stratégie Azure AD B2C, puis utilise les clés pour établir l’approbation, chiffrer ou signer un jeton. Ces approbations sont constituées des éléments suivants :

- Fédération avec les fournisseurs d’identité [OAuth1](oauth1-technical-profile.md#cryptographic-keys), [OAuth2](oauth2-technical-profile.md#cryptographic-keys)et [SAML](saml-identity-provider-technical-profile.md#cryptographic-keys)
- Sécuriser la connexion avec les [services d’API REST](secure-rest-api.md)
- Signature et chiffrement des jetons [JWT](jwt-issuer-technical-profile.md#cryptographic-keys) et [SAML](saml-issuer-technical-profile.md#cryptographic-keys)

L’élément **CryptographicKeys** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| Clé | 1:n | Clé de chiffrement utilisée dans ce profil technique. |

### <a name="key"></a>Clé

L’élément **Key** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Id | Non | Identificateur unique d’une paire de clés spécifique référencé à partir d’autres éléments dans le fichier de stratégie. |
| StorageReferenceId | Oui | Identificateur d’un conteneur de clé de stockage référencé à partir d’autres éléments dans le fichier de stratégie. |

## <a name="input-claims-transformations"></a>Transformations de revendications d’entrée

L’élément **InputClaimsTransformations** peut contenir une collection d’éléments de transformation de revendications d’entrée qui sont utilisés pour modifier les revendications d’entrée ou en générer de nouvelles. 

Les revendications de sortie d’une transformation de revendications précédente dans la collection de transformation de revendications peuvent être des revendications d’entrée d’une transformation de revendications d’entrée ultérieures, ce qui vous permet d’avoir une séquence de transformation de revendications dépendant l’une de l’autre.

L’élément **InputClaimsTransformations** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| InputClaimsTransformation | 1:n | Identificateur d’une transformation de revendications qui doit être exécutée avant l’envoi de toute revendication au fournisseur de revendications ou à la partie de confiance. Une transformation de revendications peut être utilisée pour modifier des revendications ClaimsSchema existantes ou en générer de nouvelles. |

### <a name="inputclaimstransformation"></a>InputClaimsTransformation

L’élément **InputClaimsTransformation** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ReferenceId | Oui | Identificateur d’une transformation de revendications déjà défini dans le fichier de stratégie ou un fichier de stratégie parent. |

Les profils techniques suivants font référence à la transformation de revendications **CreateOtherMailsFromEmail**. La transformation de revendications ajoute la valeur de la revendication `email` à la collection `otherMails` avant de rendre les données persistantes dans le répertoire.

```xml
<TechnicalProfile Id="AAD-UserWriteUsingAlternativeSecurityId">
  ...
  <InputClaimsTransformations>
    <InputClaimsTransformation ReferenceId="CreateOtherMailsFromEmail" />
  </InputClaimsTransformations>
  <PersistedClaims>
    <PersistedClaim ClaimTypeReferenceId="alternativeSecurityId" />
    <PersistedClaim ClaimTypeReferenceId="userPrincipalName" />
    <PersistedClaim ClaimTypeReferenceId="mailNickName" DefaultValue="unknown" />
    <PersistedClaim ClaimTypeReferenceId="displayName" DefaultValue="unknown" />
    <PersistedClaim ClaimTypeReferenceId="otherMails" />
    <PersistedClaim ClaimTypeReferenceId="givenName" />
    <PersistedClaim ClaimTypeReferenceId="surname" />
  </PersistedClaims>
  ...
</TechnicalProfile>
```

## <a name="input-claims"></a>Revendications d’entrée

**InputClaims** récupère les revendications auprès du conteneur de revendications et est utilisé pour le profil technique. Par exemple, un [profil technique autodéclaré](self-asserted-technical-profile.md) utilise les revendications d’entrée pour préremplir les revendications de sortie fournies par l’utilisateur. Un profil technique d’API REST utilise les revendications d’entrée pour envoyer les paramètres d’entrée au point de terminaison de l’API REST. Azure Active Directory utilise la revendication d’entrée comme identificateur unique pour lire, mettre à jour ou supprimer un compte.

L’élément **InputClaims** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| InputClaim | 1:n | Type de revendication d’entrée attendu. |

### <a name="inputclaim"></a>InputClaim

L’élément **InputClaim** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | Oui | Identificateur d’un type de revendication déjà défini dans la section ClaimsSchema du fichier de stratégie ou d’un fichier de stratégie parent. |
| DefaultValue | Non | Valeur par défaut à utiliser pour créer une revendication si la revendication indiquée par l’identificateur ClaimTypeReferenceId n’existe pas, de sorte que le profil technique puisse utiliser la revendication obtenue comme InputClaim. |
| PartnerClaimType | Non | Identificateur du type de revendication du partenaire externe auquel le type de revendication de stratégie spécifié mappe. Si l’attribut PartnerClaimType n’est pas spécifié, le type de revendication de stratégie spécifié est mappé au type de revendication de partenaire du même nom. Utilisez cette propriété lorsque le nom de votre type de revendication diffère de celui de l’autre partie. Par exemple, le nom de la première revendication est « givenName », tandis que le partenaire utilise une revendication nommée « first_name ». |

## <a name="display-claims"></a>Revendications d’affichage

L’élément **DisplayClaims** contient une liste des revendications définie par le [profil technique autodéclaré](self-asserted-technical-profile.md) à présenter à l’écran pour collecter des données de l’utilisateur. Dans la collection de revendications d’affichage, vous pouvez inclure une référence à un [type de revendication](claimsschema.md) ou un [DisplayControl](display-controls.md) que vous avez créé. 

- Un type de revendication est une référence à une revendication à afficher à l’écran. 
  - Pour forcer l’utilisateur à fournir une valeur pour une revendication spécifique, affectez la valeur `true` à l’attribut **Required** de l’élément **DisplayClaims**.
  - Pour prérenseigner les valeurs des revendications d'affichage, utilisez les revendications d'entrée décrites précédemment. L’élément peut également contenir une valeur par défaut.
  - L’élément **ClaimType** dans la collection **DisplayClaims** doit définir l’élément **UserInputType** sur n’importe quel type d’entrée d’utilisateur pris en charge par Azure AD B2C. Par exemple, `TextBox` ou `DropdownSingleSelect`.
- Un contrôle d’affichage est un élément d’interface utilisateur qui présente une fonctionnalité particulière et interagit avec le service principal Azure AD B2C. Il permet à l’utilisateur d’exécuter des actions sur la page qui appellent un profil technique de validation dans le back end. Par exemple, vérifier une adresse e-mail, un numéro de téléphone ou un numéro de fidélité de client.

L’ordre des éléments dans **DisplayClaims** détermine l’ordre dans lequel Azure AD B2C affiche les revendications à l’écran. 

L’élément **DisplayClaims** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| DisplayClaim | 1:n | Type de revendication d’entrée attendu. |

### <a name="displayclaim"></a>DisplayClaim

L’élément **DisplayClaim** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | Non | Identificateur d’un type de revendication déjà défini dans la section ClaimsSchema du fichier de stratégie ou d’un fichier de stratégie parent. |
| DisplayControlReferenceId | Non | Identificateur d’un [contrôle d’affichage](display-controls.md) déjà défini dans la section ClaimsSchema du fichier de stratégie ou d’un fichier de stratégie parent. |
| Obligatoire | Non | Indique si la revendication d’affichage est requise. |

L’exemple suivant illustre l’utilisation des revendications d’affichage et des contrôles d’affichage dans un profil technique autodéclaré.

![Profil technique autodéclaré avec revendications d’affichage](./media/technical-profiles/display-claims.png)

Ajoutez le profil technique suivant :

- La première revendication d’affichage fait référence au contrôle d’affichage `emailVerificationControl` qui collecte et vérifie l’adresse e-mail.
- La cinquième revendication d’affichage fait référence au contrôle d’affichage `phoneVerificationControl` qui collecte et vérifie un numéro de téléphone.
- Les autres revendications d’affichage sont des ClaimTypes à recueillir auprès de l’utilisateur.

```xml
<TechnicalProfile Id="Id">
  <DisplayClaims>
    <DisplayClaim DisplayControlReferenceId="emailVerificationControl" />
    <DisplayClaim ClaimTypeReferenceId="displayName" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="givenName" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="surName" Required="true" />
    <DisplayClaim DisplayControlReferenceId="phoneVerificationControl" />
    <DisplayClaim ClaimTypeReferenceId="newPassword" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="reenterPassword" Required="true" />
  </DisplayClaims>
</TechnicalProfile>
```

### <a name="persisted-claims"></a>Revendications persistantes

L’élément **PersistedClaims** contient toutes les valeurs qui doivent être conservées par le [profil technique Azure AD](active-directory-technical-profile.md), avec des informations de mappage possible entre un type de revendication déjà défini dans la section [ClaimsSchema](claimsschema.md) dans la stratégie et le nom d’attribut Azure AD.

Le nom de la revendication est le nom de l’[attribut Azure AD](user-profile-attributes.md), sauf si l’attribut **PartnerClaimType** est spécifié, qui contient le nom d’attribut Azure AD.

L’élément **PersistedClaims** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| PersistedClaim | 1:n | Type de revendication à conserver. |

### <a name="persistedclaim"></a>PersistedClaim

L’élément **PersistedClaim** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | Oui | Identificateur d’un type de revendication déjà défini dans la section ClaimsSchema du fichier de stratégie ou d’un fichier de stratégie parent. |
| DefaultValue | Non | Valeur par défaut à utiliser pour créer une revendication si la revendication indiquée par l’identificateur ClaimTypeReferenceId n’existe pas, de sorte que le profil technique puisse utiliser la revendication obtenue comme InputClaim. |
| PartnerClaimType | Non | Identificateur du type de revendication du partenaire externe auquel le type de revendication de stratégie spécifié mappe. Si l’attribut PartnerClaimType n’est pas spécifié, le type de revendication de stratégie spécifié est mappé au type de revendication de partenaire du même nom. Utilisez cette propriété lorsque le nom de votre type de revendication diffère de celui de l’autre partie. Par exemple, le nom de la première revendication est « givenName », tandis que le partenaire utilise une revendication nommée « first_name ». |

Dans l’exemple suivant, le profil technique **AAD-UserWriteUsingLogonEmail** ou le [pack de démarrage](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/SocialAndLocalAccounts), qui crée un nouveau compte local, conserve les revendications suivantes :

```xml
<PersistedClaims>
  <PersistedClaim ClaimTypeReferenceId="email" PartnerClaimType="signInNames.emailAddress" />
  <PersistedClaim ClaimTypeReferenceId="newPassword" PartnerClaimType="password"/>
  <PersistedClaim ClaimTypeReferenceId="displayName" DefaultValue="unknown" />
  <PersistedClaim ClaimTypeReferenceId="passwordPolicies" DefaultValue="DisablePasswordExpiration" />
  <PersistedClaim ClaimTypeReferenceId="givenName" />
  <PersistedClaim ClaimTypeReferenceId="surname" />
</PersistedClaims>
```

## <a name="output-claims"></a>Revendications de sortie

**OutputClaims** est la collection de revendications retournées au conteneur de revendications une fois le profil technique complété. Vous pouvez utiliser ces revendications dans la prochaine étape d’orchestration, ou dans les transformations de revendications de sortie. L’élément **OutputClaims** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| OutputClaim | 1:n | Type de revendication de sortie attendu. |

### <a name="outputclaim"></a>OutputClaim

L’élément **OutputClaim** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | Oui | Identificateur d’un type de revendication déjà défini dans la section ClaimsSchema du fichier de stratégie ou d’un fichier de stratégie parent. |
| DefaultValue | Non | Valeur par défaut à utiliser pour créer une revendication si la revendication indiquée par l’identificateur ClaimTypeReferenceId n’existe pas, de sorte que le profil technique puisse utiliser la revendication obtenue comme InputClaim. |
|AlwaysUseDefaultValue |Non |Forcer l’utilisation de la valeur par défaut.  |
| PartnerClaimType | Non | Identificateur du type de revendication du partenaire externe auquel le type de revendication de stratégie spécifié mappe. Si l’attribut PartnerClaimType n’est pas spécifié, le type de revendication de stratégie spécifié est mappé au type de revendication de partenaire du même nom. Utilisez cette propriété lorsque le nom de votre type de revendication diffère de celui de l’autre partie. Par exemple, le nom de la première revendication est « givenName », tandis que le partenaire utilise une revendication nommée « first_name ». |

## <a name="output-claims-transformations"></a>Transformations de revendications de sortie

L’élément **OutputClaimsTransformations** peut contenir une collection d’éléments **OutputClaimsTransformation** qui sont utilisés pour modifier les revendications de sortie ou en générer de nouvelles. Après l’exécution, les revendications de sortie sont replacées dans le conteneur de revendications. Vous pouvez utiliser ces revendications dans la prochaine étape d’orchestration.

Les revendications de sortie d’une transformation de revendications précédente dans la collection de transformation de revendications peuvent être des revendications d’entrée d’une transformation de revendications d’entrée ultérieures, ce qui vous permet d’avoir une séquence de transformation de revendications dépendant l’une de l’autre.

L’élément **OutputClaimsTransformations** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| OutputClaimsTransformation | 1:n | Identificateurs de transformations de revendications qui doivent être exécutées avant l’envoi de toute revendication au fournisseur de revendications ou à la partie de confiance. Une transformation de revendications peut être utilisée pour modifier des revendications ClaimsSchema existantes ou en générer de nouvelles. |

### <a name="outputclaimstransformation"></a>OutputClaimsTransformation

L’élément **OutputClaimsTransformation** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ReferenceId | Oui | Identificateur d’une transformation de revendications déjà défini dans le fichier de stratégie ou un fichier de stratégie parent. |

Le profil technique suivant fait référence à la transformation de revendications AssertAccountEnabledIsTrue pour déterminer si le compte est activé ou non après la lecture de la revendication `accountEnabled` à partir de l’annuaire.    

```xml
<TechnicalProfile Id="AAD-UserReadUsingEmailAddress">
  ...
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="objectId" />
    <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="localAccountAuthentication" />
    <OutputClaim ClaimTypeReferenceId="userPrincipalName" />
    <OutputClaim ClaimTypeReferenceId="displayName" />
    <OutputClaim ClaimTypeReferenceId="accountEnabled" />
    <OutputClaim ClaimTypeReferenceId="otherMails" />
    <OutputClaim ClaimTypeReferenceId="signInNames.emailAddress" />
  </OutputClaims>
  <OutputClaimsTransformations>
    <OutputClaimsTransformation ReferenceId="AssertAccountEnabledIsTrue" />
  </OutputClaimsTransformations>
  ...
</TechnicalProfile>
```

## <a name="validation-technical-profiles"></a>Profils techniques de validation

Un profil technique de validation sert à valider toutes ou certaines des revendications de sortie de référencement dans un [profil technique autodéclaré](self-asserted-technical-profile.md#validation-technical-profiles). Un profil technique de validation est un profil technique ordinaire issu de n’importe quel protocole, comme [Azure Active Directory](active-directory-technical-profile.md) ou une [API REST](restful-technical-profile.md). Le profil technique de validation retourne des revendications de sortie ou retourne un code d’erreur. Le message d’erreur s’affiche pour l’utilisateur, permettant à celui-ci de réessayer.

Le diagramme suivant illustre la façon dont Azure AD B2C utilise un profil technique de validation pour valider les informations d’identification de l’utilisateur.

![Flux du profil technique de validation du diagramme](./media/technical-profiles/validation-technical-profile.png) 

L’élément **ValidationTechnicalProfiles** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| ValidationTechnicalProfile | 1:n | Identificateur des profils techniques utilisés pour valider la totalité ou certaines des revendications de sortie du profil technique de référencement. Toutes les revendications d’entrée du profil technique référencé doivent apparaître dans les revendications de sortie du profil technique de référencement. |

### <a name="validationtechnicalprofile"></a>ValidationTechnicalProfile

L’élément **ValidationTechnicalProfile** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ReferenceId | Oui | Identificateur d’un profil technique déjà défini dans le fichier de stratégie ou dans un fichier de stratégie parent. |

## <a name="subjectnaminginfo"></a>SubjectNamingInfo

**SubjectNamingInfo** définit le nom du sujet utilisé dans les jetons d’une [stratégie de partie de confiance](relyingparty.md#subjectnaminginfo). L’élément **SubjectNamingInfo** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ClaimType | Oui | Identificateur d’un type de revendication déjà défini dans la section ClaimsSchema du fichier de stratégie. |

## <a name="include-technical-profile"></a>Inclure un profil technique

Un profil technique peut inclure un autre profil technique pour modifier des paramètres ou ajouter de nouvelles fonctionnalités. L’élément **IncludeTechnicalProfile** est une référence au profil technique courant dont un profil technique découle. Pour réduire la redondance et la complexité de vos éléments de stratégie, utilisez l’inclusion quand vous disposez de plusieurs profils techniques qui partagent les éléments de base. Utilisez un profil technique courant avec l’ensemble courant de configuration, ainsi que des profils techniques de tâches spécifiques qui incluent le profil technique courant. Supposons par exemple que vous ayez un [profil technique d’API REST](restful-technical-profile.md) avec un seul point de terminaison où vous devez envoyer différents jeux de revendications pour différents scénarios. Créez un profil technique courant avec les fonctionnalités partagées, telles que l’URI de point de terminaison de l’API REST, les métadonnées, le type d’authentification et les clés de chiffrement. Créez ensuite des profils techniques de tâche spécifiques qui incluent le profil technique courant, ajoutez les revendications d’entrée, les revendications de sortie ou remplacez l’URI de point de terminaison de l’API REST correspondant à ce profil technique.

L’élément **IncludeTechnicalProfile** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ReferenceId | Oui | Identificateur d’un profil technique déjà défini dans le fichier de stratégie ou dans un fichier de stratégie parent. |


L'exemple ci-dessous illustre l'utilisation de l'inclusion :

- *REST-API-Common* : profil technique courant avec la configuration de base.
- *REST-ValidateProfile* : comprend le profil technique *REST-API-Commom* et spécifie les revendications d’entrée et de sortie.
- *REST-UpdateProfile* : comprend le profil technique *REST-API-Commom*, spécifie les revendications d’entrée et écrase les métadonnées `ServiceUrl`.

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-API-Commom">
      <DisplayName>Base REST API configuration</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://your-app-name.azurewebsites.NET/api/identity</Item>
        <Item Key="AuthenticationType">Basic</Item>
        <Item Key="SendClaimsIn">Body</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="BasicAuthenticationUsername" StorageReferenceId="B2C_1A_B2cRestClientId" />
        <Key Id="BasicAuthenticationPassword" StorageReferenceId="B2C_1A_B2cRestClientSecret" />
      </CryptographicKeys>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
    </TechnicalProfile>

    <TechnicalProfile Id="REST-ValidateProfile">
      <DisplayName>Validate the account and return promo code</DisplayName>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="objectId" />
        <InputClaim ClaimTypeReferenceId="email" />
        <InputClaim ClaimTypeReferenceId="userLanguage" PartnerClaimType="lang" DefaultValue="{Culture:LCID}" AlwaysUseDefaultValue="true" />
      </InputClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="promoCode" />
      </OutputClaims>
      <IncludeTechnicalProfile ReferenceId="REST-API-Commom" />
    </TechnicalProfile>

    <TechnicalProfile Id="REST-UpdateProfile">
       <Metadata>
        <Item Key="ServiceUrl">https://your-app-name.azurewebsites.NET/api/identity/update</Item>
      </Metadata>
      <DisplayName>Update the user profile</DisplayName>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="objectId" />
        <InputClaim ClaimTypeReferenceId="email" />
      </InputClaims>
      <IncludeTechnicalProfile ReferenceId="REST-API-Commom" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

### <a name="multi-level-inclusion"></a>Inclusion de plusieurs niveaux 

Un profil technique peut inclure un profil technique unique. Le nombre de niveaux d’inclusion n’est pas limité. Par exemple, le profil technique **AAD-UserReadUsingAlternativeSecurityId-NoError** inclut le profil **AAD-UserReadUsingAlternativeSecurityId**. Ce profil technique définit l’élément de métadonnées `RaiseErrorIfClaimsPrincipalDoesNotExist` sur `true`, et génère une erreur si aucun compte de réseau social ne figure dans l’annuaire. **AAD-UserReadUsingAlternativeSecurityId-NoError** remplace ce comportement et désactive ce message d’erreur.

```xml
<TechnicalProfile Id="AAD-UserReadUsingAlternativeSecurityId-NoError">
  <Metadata>
    <Item Key="RaiseErrorIfClaimsPrincipalDoesNotExist">false</Item>
  </Metadata>
  <IncludeTechnicalProfile ReferenceId="AAD-UserReadUsingAlternativeSecurityId" />
</TechnicalProfile>
```

**AAD-UserReadUsingAlternativeSecurityId** inclut le profil technique `AAD-Common`.

```xml
<TechnicalProfile Id="AAD-UserReadUsingAlternativeSecurityId">
  <Metadata>
    <Item Key="Operation">Read</Item>
    <Item Key="RaiseErrorIfClaimsPrincipalDoesNotExist">true</Item>
    <Item Key="UserMessageIfClaimsPrincipalDoesNotExist">User does not exist. Please sign up before you can sign in.</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="AlternativeSecurityId" PartnerClaimType="alternativeSecurityId" Required="true" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="objectId" />
    <OutputClaim ClaimTypeReferenceId="userPrincipalName" />
    <OutputClaim ClaimTypeReferenceId="displayName" />
    <OutputClaim ClaimTypeReferenceId="otherMails" />
    <OutputClaim ClaimTypeReferenceId="givenName" />
    <OutputClaim ClaimTypeReferenceId="surname" />
  </OutputClaims>
  <IncludeTechnicalProfile ReferenceId="AAD-Common" />
</TechnicalProfile>
```

Ni **AAD-UserReadUsingAlternativeSecurityId-NoError** ni **AAD-UserReadUsingAlternativeSecurityId** ne spécifie l’élément **Protocole** requis, car celui-ci est spécifié dans le profil technique **AAD-Common**.

```xml
<TechnicalProfile Id="AAD-Common">
  <DisplayName>Azure Active Directory</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.AzureActiveDirectoryProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  ...
</TechnicalProfile>
```

## <a name="use-technical-profile-for-session-management"></a>Utiliser un profil technique pour la gestion des sessions

L’élément **UseTechnicalProfileForSessionManagement** fait référence à un [Profil technique de session d’authentification unique](custom-policy-reference-sso.md). L’élément **UseTechnicalProfileForSessionManagement** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ReferenceId | Oui | Identificateur d’un profil technique déjà défini dans le fichier de stratégie ou dans un fichier de stratégie parent. |

## <a name="enabled-for-user-journeys"></a>Compatible avec les parcours d'utilisateur

L’élément [ClaimsProviderSelections](userjourneys.md#claimsproviderselection) dans un parcours utilisateur définit la liste et l’ordre des options de sélection de fournisseur de revendications. Avec l’élément **EnabledForUserJourneys**, vous filtrez le fournisseur de revendications disponible pour l’utilisateur. L’élément **EnabledForUserJourneys** contient l’une des valeurs suivantes :

- **Always**, exécuter le profil technique.
- **Never**, ignorer le profil technique.
- **OnClaimsExistence**, exécuter seulement s’il existe une certaine revendication spécifiée dans le profil technique.
- **OnItemExistenceInStringCollectionClaim**, exécuter uniquement s’il existe un élément dans une revendication de collection de chaîne.
- **OnItemAbsenceInStringCollectionClaim**, exécuter uniquement s’il n’existe pas d’élément dans une revendication de collection de chaîne.

L’utilisation de **OnClaimsExistence**, **OnItemExistenceInStringCollectionClaim** ou **OnItemAbsenceInStringCollectionClaim**, requiert que vous fournissez les métadonnées suivantes : **ClaimTypeOnWhichToEnable** spécifie le type de la revendication à évaluer, **ClaimValueOnWhichToEnable** spécifie la valeur à comparer.

Le profil technique suivant est exécuté uniquement si la collection de chaînes **identityProviders** contient la valeur de `facebook.com` :

```xml
<TechnicalProfile Id="UnLink-Facebook-OAUTH">
  <DisplayName>Unlink Facebook</DisplayName>
...
    <Metadata>
      <Item Key="ClaimTypeOnWhichToEnable">identityProviders</Item>
      <Item Key="ClaimValueOnWhichToEnable">facebook.com</Item>
    </Metadata>
...
  <EnabledForUserJourneys>OnItemExistenceInStringCollectionClaim</EnabledForUserJourneys>
</TechnicalProfile>
```
