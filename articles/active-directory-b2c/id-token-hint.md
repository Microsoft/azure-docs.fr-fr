---
title: Définir un profil technique d’indicateur de jeton d’ID dans une stratégie personnalisée
titleSuffix: Azure AD B2C
description: Définissez un profil technique de jeton id_token_hint dans une stratégie personnalisée d'Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 10/16/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 79a99d9f0ca117d8f47d56d76399210a72b91bb7
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94951653"
---
# <a name="define-an-id-token-hint-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>Définir un profil technique id_token_hint dans une stratégie personnalisée Azure Active Directory B2C

Azure AD B2C permet aux applications par partie de confiance d’envoyer un jeton JWT entrant dans le cadre de la demande d’autorisation OAuth2. Le jeton JWT peut être émis par une application de partie de confiance ou un fournisseur d’identité, et il peut transmettre une indication sur l’utilisateur ou la demande d’autorisation. Azure AD B2C valide la signature, le nom de l’émetteur et l’audience du jeton, puis extrait la revendication du jeton entrant.

## <a name="use-cases"></a>Cas d'utilisation

Vous pouvez utiliser cette solution pour envoyer des données à Azure AD B2C encapsulées dans un jeton JWT unique. La [solution S’abonner avec une invitation par courrier électronique](https://github.com/azure-ad-b2c/samples/blob/master/policies/invite/README.md), où votre administrateur système peut envoyer une invitation signée à des utilisateurs, est basée sur id_token_hint. Seuls les utilisateurs ayant accès à l’e-mail d’invitation peuvent créer le compte dans le répertoire.

## <a name="token-signing-approach"></a>Approche de signature de jetons

Avec id_token_hint, l’émetteur de jeton (une application de partie de confiance ou un fournisseur d’identité) compose le jeton, puis le signe à l’aide d’une clé de signature pour prouver que le jeton provient d’une source approuvée. La clé de signature peut être symétrique ou asymétrique. Le chiffrement symétrique, ou chiffrement à clé privée, utilise un secret partagé pour signer et valider la signature. Le chiffrement asymétrique, ou chiffrement à clé publique, est un système de chiffrement qui utilise à la fois une clé privée et une clé publique. La clé privée est connue uniquement de l’émetteur de jeton et est utilisée pour signer le jeton. La clé publique est partagée avec la stratégie Azure AD B2C pour valider la signature du jeton.

## <a name="token-format"></a>Format de jeton

id_token_hint doit être un jeton JWT valide. Le tableau suivant répertorie les revendications qui sont obligatoires. Les revendications supplémentaires sont facultatives.

| Name | Revendication | Valeur d'exemple | Description |
| ---- | ----- | ------------- | ----------- |
| Public visé | `aud` | `a489fc44-3cc0-4a78-92f6-e413cd853eae` | Identifie le destinataire du jeton. Il s’agit d’une chaîne arbitraire définie par l’émetteur du jeton. Azure AD B2C valide cette valeur et rejette le jeton si elle ne correspond pas.  |
| Émetteur | `iss` |`https://localhost` | Identifie le service de jeton de sécurité (émetteur de jeton). Il s’agit d’un URI arbitraire défini par l’émetteur du jeton. Azure AD B2C valide cette valeur et rejette le jeton si elle ne correspond pas.  |
| Heure d’expiration | `exp` | `1600087315` | Heure à laquelle le jeton n’est plus valide, représentée en heure epoch. Azure AD B2C ne valide pas cette revendication. |
| Pas avant | `nbf` | `1599482515` | Heure à laquelle le jeton est valide, représentée en heure epoch. Cette heure correspond généralement à l’heure à laquelle le jeton a été émis. Azure AD B2C ne valide pas cette revendication. |

 Le jeton suivant est un exemple de jeton d’ID valide :

```json
{
  "alg": "HS256",
  "typ": "JWT"
}.{
  "displayName": " John Smith",
  "userId": "john.s@contoso.com",
  "nbf": 1599482515,
  "exp": 1600087315,
  "iss": "https://localhost",
  "aud": "a489fc44-3cc0-4a78-92f6-e413cd853eae"
}
```

## <a name="protocol"></a>Protocol

L’attribut **Name** de l’élément **Protocol** doit être défini sur `None`. Par exemple, le protocole pour le profil technique **IdTokenHint_ExtractClaims** est `None` :

```xml
<TechnicalProfile Id="IdTokenHint_ExtractClaims">
  <DisplayName> My ID Token Hint TechnicalProfile</DisplayName>
  <Protocol Name="None" />
  ...
```

Le profil technique est appelé à partir d’une étape d’orchestration avec le type de `GetClaims`.

```xml
<OrchestrationStep Order="1" Type="GetClaims" CpimIssuerTechnicalProfileReferenceId="IdTokenHint_ExtractClaims" />
``` 

## <a name="output-claims"></a>Revendications de sortie

L’élément **OutputClaims** contient une liste de revendications à extraire du jeton JWT. Il se peut que vous deviez mapper le nom de la revendication définie dans votre stratégie au nom défini dans le jeton JWT. Vous pouvez également inclure des revendications qui ne sont pas retournées par le jeton JWT, pour autant que vous définissiez l’attribut `DefaultValue`.

## <a name="metadata"></a>Métadonnées

Les métadonnées suivantes sont pertinentes lors de l’utilisation d’une clé symétrique. 

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| émetteur | Oui | Identifie le service de jeton de sécurité (émetteur de jeton). Cette valeur doit être identique à la revendication `iss` dans la revendication de jeton JWT. | 
| IdTokenAudience | Oui | Identifie le destinataire du jeton. Doit être identique à la revendication `aud` dans la revendication de jeton JWT. | 

Les métadonnées suivantes sont pertinentes lors de l’utilisation d’une clé asymétrique. 

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| METADATA| Oui | URL qui pointe vers un document de configuration d’émetteur de jeton, également appelé point de terminaison de configuration OpenID connu.   |
| émetteur | Non | Identifie le service de jeton de sécurité (émetteur de jeton). Cette valeur peut être utilisée pour remplacer la valeur configurée dans les métadonnées et doit être identique à la revendication `iss` dans la revendication de jeton JWT. |  
| IdTokenAudience | Non | Identifie le destinataire du jeton. Doit être identique à la revendication `aud` dans la revendication de jeton JWT. |  

## <a name="cryptographic-keys"></a>Clés de chiffrement

Lors de l’utilisation d’une clé symétrique, l’élément **CryptographicKeys** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| client_secret | Oui | Clé de chiffrement utilisée pour valider la signature du jeton JWT.|


## <a name="how-to-guide"></a>Guide pratique

### <a name="issue-a-token-with-symmetric-keys"></a>Émettre un jeton avec des clés symétriques

#### <a name="step-1-create-a-shared-key"></a>Étape 1. Créer une clé partagée 

Créez une clé qui peut être utilisée pour signer le jeton. Par exemple, utilisez le code PowerShell suivant pour générer une clé.

```powershell
$bytes = New-Object Byte[] 32
$rand = [System.Security.Cryptography.RandomNumberGenerator]::Create()
$rand.GetBytes($bytes)
$rand.Dispose()
$newClientSecret = [System.Convert]::ToBase64String($bytes)
$newClientSecret
```

Ce code crée une chaîne secrète comme `VK62QTn0m1hMcn0DQ3RPYDAr6yIiSvYgdRwjZtU5QhI=`.

#### <a name="step-2-add-the-signing-key-to-azure-ad-b2c"></a>Étape 2. Ajouter la clé de signature à Azure AD B2C

La clé utilisée par l’émetteur de jeton doit être créée dans vos clés de stratégie Azure AD B2C.  

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire Azure AD B2C.
1. Dans la Portail Azure, recherchez et sélectionnez **Azure AD B2C**.
1. Dans la page de vue d’ensemble, sous **Stratégies**, sélectionnez **Identity Experience Framework**.
1. Sélectionner **Clés de stratégie** 
1. Sélectionnez **Manuel**.
1. Pour **Nom**, utilisez `IdTokenHintKey`.  
   Il est possible que le préfixe `B2C_1A_` soit ajouté automatiquement.
1. Dans la zone **Secret**, entrez la clé de connexion que vous avez générée précédemment.
1. Pour **Utilisation de la clé**, utilisez **Chiffrement**.
1. Sélectionnez **Create** (Créer).
1. Vérifiez que vous avez créé la clé `B2C_1A_IdTokenHintKey`.


#### <a name="step-3-add-the-id-token-hint-technical-profile"></a>Étape 3. Ajouter le profil technique de l’indicateur de jeton d’ID

Le profil technique suivant valide le jeton et extrait les revendications. 

```xml
<ClaimsProvider>
  <DisplayName>My ID Token Hint ClaimsProvider</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="IdTokenHint_ExtractClaims">
      <DisplayName> My ID Token Hint TechnicalProfile</DisplayName>
      <Protocol Name="None" />
      <Metadata>
        <Item Key="IdTokenAudience">a489fc44-3cc0-4a78-92f6-e413cd853eae</Item>
        <Item Key="issuer">https://localhost</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="client_secret" StorageReferenceId="B2C_1A_IdTokenHintKey" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="userId" />
      </OutputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

#### <a name="step-4-prepare-your-policy"></a>Étape 4. Préparer votre stratégie

Effectuez l’étape [Configurer votre stratégie](#configure-your-policy).

#### <a name="step-5-prepare-the-code"></a>Étape 5. Préparer le code  

L’exemple [GitHub](https://github.com/azure-ad-b2c/id_token_hint/tree/master/dotnet_core_symmetric_key) est une application web ASP.NET et une application console qui génère un jeton d’ID signé à l’aide d’une clé symétrique. 


### <a name="issue-a-token-with-asymmetric-keys"></a>Émettre un jeton avec des clés asymétriques

Avec une clé asymétrique, le jeton est signé à l’aide de certificats RSA. Cette application héberge un point de terminaison de métadonnées Open ID Connect et un point de terminaison de clés web JSON (JWK) qui est utilisé par Azure AD B2C pour valider la signature du jeton d’ID.

L’émetteur de jeton doit fournir les points de terminaison suivants :

* `/.well-known/openid-configuration` : un point de terminaison de configuration bien connu avec des informations pertinentes sur le jeton, comme le nom de l’émetteur du jeton et le lien vers le point de terminaison JWK. 
* `/.well-known/keys` : le point de terminaison de la clé web JSON (JWK) avec la clé publique utilisée pour signer la clé (avec la partie clé privée du certificat).

Consultez l’exemple de contrôleur MVC .Net [TokenMetadataController.cs](https://github.com/azure-ad-b2c/id-token-builder/blob/master/source-code/B2CIdTokenBuilder/Controllers/TokenMetadataController.cs).

#### <a name="step-1-prepare-a-self-signed-certificate"></a>Étape 1. Préparer un certificat auto-signé

Si vous n’avez pas encore de certificat, vous pouvez utiliser un certificat auto-signé pour ce guide pratique. Sur Windows, vous pouvez utiliser l’applet de commande [New-SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate) de PowerShell pour générer un certificat.

Exécutez cette commande PowerShell pour générer un certificat auto-signé. Modifiez l’argument `-Subject` comme il convient pour votre application et le nom de locataire Azure AD B2C. Vous pouvez également ajuster la date de `-NotAfter` pour spécifier un délai d’expiration différent pour le certificat.

```PowerShell
New-SelfSignedCertificate `
    -KeyExportPolicy Exportable `
    -Subject "CN=yourappname.yourtenant.onmicrosoft.com" `
    -KeyAlgorithm RSA `
    -KeyLength 2048 `
    -KeyUsage DigitalSignature `
    -NotAfter (Get-Date).AddMonths(12) `
    -CertStoreLocation "Cert:\CurrentUser\My"
```


#### <a name="step-2-add-the-id-token-hint-technical-profile"></a>Étape 2. Ajouter le profil technique de l’indicateur de jeton d’ID 

Le profil technique suivant valide le jeton et extrait les revendications. Remplacez l’URI des métadonnées par le point de terminaison de configuration bien connu de votre émetteur de jeton.  

```xml
<ClaimsProvider>
  <DisplayName>My ID Token Hint ClaimsProvider</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="IdTokenHint_ExtractClaims">
      <DisplayName> My ID Token Hint TechnicalProfile</DisplayName>
      <Protocol Name="None" />
      <Metadata>
        <!-- Replace with your endpoint location -->
        <Item Key="METADATA">https://your-app.azurewebsites.net/.well-known/openid-configuration</Item>
        <Item Key="IdTokenAudience">your_optional_audience</Item> -->
        <!-- <Item Key="issuer">your_optional_token_issuer_override</Item> -->
      </Metadata>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="userId" />
      </OutputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

#### <a name="step-3-prepare-your-policy"></a>Étape 3. Préparer votre stratégie

Effectuez l’étape [Configurer votre stratégie](#configure-your-policy).

#### <a name="step-4-prepare-the-code"></a>Étape 4. Préparer le code 

Cet [exemple GitHub](https://github.com/azure-ad-b2c/id-token-builder) d’application web ASP.NET génère des jetons d’ID et héberge les points de terminaison de métadonnées requis pour utiliser le paramètre « id_token_hint » dans Azure AD B2C.


### <a name="configure-your-policy"></a>Configurer votre stratégie

Pour les approches symétriques et asymétriques, le profil technique `id_token_hint` est appelé à partir d’une étape d’orchestration avec le type `GetClaims` et doit spécifier les revendications d’entrée de la stratégie de partie de confiance.

1. Ajoutez le profil technique IdTokenHint_ExtractClaims à votre stratégie d’extension.
1. Ajoutez l’étape d’orchestration suivante à votre parcours utilisateur en tant que premier élément.  

    ```xml
    <OrchestrationStep Order="1" Type="GetClaims" CpimIssuerTechnicalProfileReferenceId="IdTokenHint_ExtractClaims" />
    ``` 
1. Dans votre stratégie de partie de confiance, répétez les mêmes revendications d’entrée que celles que vous avez configurées dans le profil technique IdTokenHint_ExtractClaims. Par exemple :
    ```xml
   <RelyingParty>
     <DefaultUserJourney ReferenceId="SignUp" />
     <TechnicalProfile Id="PolicyProfile">
       <DisplayName>PolicyProfile</DisplayName>
       <Protocol Name="OpenIdConnect" />
       <InputClaims>
         <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="userId" />
        </InputClaims>
       <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
        <OutputClaim ClaimTypeReferenceId="identityProvider" />
       </OutputClaims>
       <SubjectNamingInfo ClaimType="sub" />
      </TechnicalProfile>
    </RelyingParty>
    ```

Selon les besoins de votre entreprise, vous devrez peut-être ajouter des validations de jeton, par exemple pour vérifier l’expiration du jeton, le format de l’adresse de messagerie et bien plus encore. Pour ce faire, ajoutez des étapes d’orchestration qui appellent un [profil technique de transformation des revendications](claims-transformation-technical-profile.md). Ajoutez également un [profil technique autodéclaré](self-asserted-technical-profile.md) pour présenter un message d’erreur. 

### <a name="create-and-sign-a-token"></a>Créer et signer un jeton

Les exemples GitHub illustrent comment créer un tel jeton JWT qui est envoyé par la suite en tant que paramètre de chaîne de requête `id_token_hint`. Voici un exemple de demande d’autorisation avec un paramètre id_token_hint
 
```html
https://tenant-name.b2clogin.com/tenant-name.onmicrosoft.com/B2C_1A_signup_signin/oauth2/v2.0/authorize?client_id=63ba0d17-c4ba-47fd-89e9-31b3c2734339&nonce=defaultNonce&redirect_uri=https%3A%2F%2Fjwt.ms&scope=openid&response_type=id_token&prompt=login&id_token_hint=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkaXNwbGF5TmFtZSI6IiBKb2huIFNtaXRoIiwidXNlcklkIjoiam9obi5zQGNvbnRvc28uY29tIiwibmJmIjoxNTk5NDgyNTE1LCJleHAiOjE2MDAwODczMTUsImlzcyI6Imh0dHBzOi8vbG9jYWxob3N0IiwiYXVkIjoiYTQ4OWZjNDQtM2NjMC00YTc4LTkyZjYtZTQxM2NkODUzZWFlIn0.nPmLXydI83PQCk5lRBYUZRu_aX58pL1khahHyQuupig
```

## <a name="next-steps"></a>Étapes suivantes

- Consultez la solution [Inscription par e-mail d’invitation](https://github.com/azure-ad-b2c/samples/blob/master/policies/invite/README.md) sur le référentiel GitHub de la communauté Azure AD B2C.