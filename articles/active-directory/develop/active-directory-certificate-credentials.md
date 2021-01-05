---
title: Informations d’identification de certificat de la Plateforme d’identités Microsoft
titleSuffix: Microsoft identity platform
description: Cet article traite de l’inscription et de l’utilisation des informations d’identification de certificat pour l’authentification d’application.
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 12/3/2020
ms.author: hirsin
ms.reviewer: nacanuma, jmprieur
ms.custom: aaddev
ms.openlocfilehash: f3222c790ccd0cee936b246253a16b5c434c61c8
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602204"
---
# <a name="microsoft-identity-platform-application-authentication-certificate-credentials"></a>Informations d’identification de certificat d’authentification d’application de la Plateforme d’identités Microsoft

La plateforme d’identités Microsoft permet à une application d’utiliser ses propres informations d’identification pour l’authentification partout où une clé secrète client pourrait être utilisée, par exemple, dans le [flux d’octroi d’informations d’identification du client](v2-oauth2-client-creds-grant-flow.md) OAuth 2.0 et le [flux On-Behalf-Of](v2-oauth2-on-behalf-of-flow.md) (OBO).

Parmi les types d’informations d’identification qu’une application peut utiliser pour l’authentification figure l’assertion [JSON Web Token](./security-tokens.md#json-web-tokens-jwts-and-claims) (JWT) signée avec un certificat dont est propriétaire l’application.

## <a name="assertion-format"></a>Format d’assertion

Pour calculer l’assertion, vous pouvez utiliser l’une des nombreuses bibliothèques JWT dans la langue de votre choix. Pour permettre cela, [MSAL utilise `.WithCertificate()`](msal-net-client-assertions.md). Les informations sont transmises par le jeton dans son [En-tête ](#header), ses [Revendications](#claims-payload) et sa [Signature](#signature).

### <a name="header"></a>En-tête

| Paramètre |  Remarque |
| --- | --- |
| `alg` | Doit être **RS256** |
| `typ` | Doit être **JWT** |
| `x5t` | Le code de hachage du certificat X.509 (également connu sous le nom d’*empreinte* de certificat SHA-1) est une représentation hexadécimale encodée en tant que valeur de chaîne Base64url. Par exemple, avec un hachage de certificat X.509 de `84E05C1D98BCE3A5421D225B140B36E86A3D5534` (hexadécimal), la revendication `x5t` serait `hOBcHZi846VCHSJbFAs26Go9VTQ=` (Base64url). |

### <a name="claims-payload"></a>Revendications (charge utile)

Type de revendication | Valeur | Description
---------- | ---------- | ----------
aud | `https://login.microsoftonline.com/{tenantId}/v2.0` | La revendication « AUD » (audience) identifie les destinataires auxquels le JWT est destiné (ici Azure AD) consultez [RFC 7519, section 4.1.3](https://tools.ietf.org/html/rfc7519#section-4.1.3).  Dans ce cas, ce destinataire est le serveur de connexion (login.microsoftonline.com).
exp | 1601519414 | La revendication « exp » (délai d'expiration) indique le délai d'expiration à partir duquel le JWT ne doit PAS être accepté pour être traité. Consultez [RFC 7519, section 4.1.4](https://tools.ietf.org/html/rfc7519#section-4.1.4).  Cela permet à l’instruction d’être utilisée jusqu’à ce moment-là. Réduisez-la donc au minimum : 5 à 10 minutes après `nbf` au maximum.  Azure AD ne détermine actuellement pas de restrictions sur l’heure de `exp`. 
iss | {ClientID} | La revendication « ISS » (émetteur) identifie le principal qui a émis le jeton JWT ; le cas échéant, votre application cliente.  Utilisez l’ID d’application GUID.
jti | (un guid) | La revendication « JTI » (ID JWT) fournit un identificateur unique pour le jeton JWT. La valeur de l’identificateur DOIT être assignée de manière à garantir qu’il y a une probabilité négligeable que la même valeur soit affectée par accident à un objet de données différent ; si l’application utilise plusieurs émetteurs, les collisions doivent également être évitées parmi les valeurs générées par différents émetteurs. La valeur « JTI » est une chaîne qui respecte la casse. [RFC 7519, section 4.1.7](https://tools.ietf.org/html/rfc7519#section-4.1.7)
nbf | 1601519114 | La revendication « nbf » (pas avant) indique le délai avant lequel le JWT ne doit PAS être accepté pour être traité. [RFC 7519, section 4.1.5](https://tools.ietf.org/html/rfc7519#section-4.1.5).  L’utilisation de l’heure actuelle est appropriée. 
sub | {ClientID} | La revendication « SUB » (objet) identifie l’objet du jeton JWT ; le cas échéant, votre application également. Utilisez la même valeur que `iss`. 

### <a name="signature"></a>Signature

La signature est calculée en appliquant le certificat, conformément à la [spécification RFC7519 sur les jetons Web JSON](https://tools.ietf.org/html/rfc7519).

## <a name="example-of-a-decoded-jwt-assertion"></a>Exemple d’une assertion JWT décodée

```JSON
{
  "alg": "RS256",
  "typ": "JWT",
  "x5t": "gx8tGysyjcRqKjFPnd7RFwvwZI0"
}
.
{
  "aud": "https: //login.microsoftonline.com/contoso.onmicrosoft.com/oauth2/token",
  "exp": 1484593341,
  "iss": "97e0a5b7-d745-40b6-94fe-5f77d35c6e05",
  "jti": "22b3bb26-e046-42df-9c96-65dbd72c1c81",
  "nbf": 1484592741,
  "sub": "97e0a5b7-d745-40b6-94fe-5f77d35c6e05"
}
.
"Gh95kHCOEGq5E_ArMBbDXhwKR577scxYaoJ1P{a lot of characters here}KKJDEg"
```

## <a name="example-of-an-encoded-jwt-assertion"></a>Exemple d’une assertion JWT encodée

La chaîne suivante est un exemple d’assertion encodée. Si vous regardez attentivement, vous remarquerez les trois sections séparées par des points (`.`) :

* La première section encode l’*en-tête*
* La deuxième section encode les *revendications* (charge utile)
* La dernière section est la *signature* calculée avec les certificats à partir du contenu des deux premières sections

```
"eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJhdWQiOiJodHRwczpcL1wvbG9naW4ubWljcm9zb2Z0b25saW5lLmNvbVwvam1wcmlldXJob3RtYWlsLm9ubWljcm9zb2Z0LmNvbVwvb2F1dGgyXC90b2tlbiIsImV4cCI6MTQ4NDU5MzM0MSwiaXNzIjoiOTdlMGE1YjctZDc0NS00MGI2LTk0ZmUtNWY3N2QzNWM2ZTA1IiwianRpIjoiMjJiM2JiMjYtZTA0Ni00MmRmLTljOTYtNjVkYmQ3MmMxYzgxIiwibmJmIjoxNDg0NTkyNzQxLCJzdWIiOiI5N2UwYTViNy1kNzQ1LTQwYjYtOTRmZS01Zjc3ZDM1YzZlMDUifQ.
Gh95kHCOEGq5E_ArMBbDXhwKR577scxYaoJ1P{a lot of characters here}KKJDEg"
```

## <a name="register-your-certificate-with-microsoft-identity-platform"></a>Inscrire votre certificat auprès de la Plateforme d’identités Microsoft

Vous pouvez associer les informations d’identification du certificat à l’application cliente dans la Plateforme d’identités Microsoft via le portail Azure en utilisant l’une des méthodes suivantes :

### <a name="uploading-the-certificate-file"></a>Chargement du fichier de certificat

Dans l’inscription d’application Azure pour l’application cliente :
1. Cliquez sur **Certificats et secrets**.
2. Cliquez sur **Charger un certificat** et sélectionnez le fichier de certificat à charger.
3. Cliquez sur **Add**.
  Une fois le certificat chargé, les valeurs d'empreinte numérique, de date de début et d’expiration s'affichent.

### <a name="updating-the-application-manifest"></a>Mise à jour du manifeste d’application

Sur la base de votre certificat, vous devez calculer :

- `$base64Thumbprint` -Valeur encodée en base64 du code de hachage du certificat
- `$base64Value` -Valeur encodée en base64 des données brutes du certificat

Vous devez également fournir un GUID pour identifier la clé dans le manifeste de l’application (`$keyId`).

Dans l’inscription d’application Azure pour l’application cliente :
1. Sélectionnez **Manifeste** pour ouvrir le manifeste d’application.
2. Remplacez la propriété *keyCredentials* par les nouvelles informations de votre certificat, en utilisant le schéma suivant.

   ```JSON
   "keyCredentials": [
       {
           "customKeyIdentifier": "$base64Thumbprint",
           "keyId": "$keyid",
           "type": "AsymmetricX509Cert",
           "usage": "Verify",
           "value":  "$base64Value"
       }
   ]
   ```
3. Enregistrez les modifications du manifeste de l’application, puis chargez-le dans la Plateforme d’identités Microsoft.

   La propriété `keyCredentials` peut avoir plusieurs valeurs. Vous pouvez donc charger plusieurs certificats pour une gestion plus élaborée des clés.
   
## <a name="using-a-client-assertion"></a>Utilisation d’une assertion de client

Les assertions de client peuvent être utilisées partout où une clé secrète client est utilisée.  Par exemple, dans le [flux du code d’autorisation](v2-oauth2-auth-code-flow.md), vous pouvez transmettre une `client_secret` pour prouver que la requête provient de votre application. Vous pouvez remplacer ceci par les paramètres `client_assertion` et `client_assertion_type`. 

| Paramètre | Valeur | Description|
|-----------|-------|------------|
|`client_assertion_type`|`urn:ietf:params:oauth:client-assertion-type:jwt-bearer`| Il s’agit d’une valeur fixe, indiquant que vous utilisez les informations d’identification d’un certificat. |
|`client_assertion`| JWT |Il s’agit du JWT créé ci-dessus. |

## <a name="next-steps"></a>Étapes suivantes

La [bibliothèque MSAL.NET gère ce scénario](msal-net-client-assertions.md) dans une seule ligne de code.

L’exemple de code [Application console de démon .NET Core utilisant la plateforme d’identité Microsoft](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2) sur GitHub montre comment une application utilise ses propres informations d’identification pour l’authentification. Il montre également comment vous pouvez [créer un certificat auto-signé](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/tree/master/1-Call-MSGraph#optional-use-the-automation-script) à l’aide de la cmdlet PowerShell `New-SelfSignedCertificate`. Vous pouvez également utiliser les [scripts de création d’application](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/blob/master/1-Call-MSGraph/AppCreationScripts-withCert/AppCreationScripts.md) dans le référentiel d’échantillons pour créer des certificats, calculer l’empreinte, etc.
