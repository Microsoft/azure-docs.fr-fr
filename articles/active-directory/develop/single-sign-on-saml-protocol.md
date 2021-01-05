---
title: Authentification unique Azure - protocole SAML
titleSuffix: Microsoft identity platform
description: Cet article décrit le protocole SAML d’authentification unique (SSO) dans Azure Active Directory
services: active-directory
documentationcenter: .net
author: kenwith
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 05/18/2020
ms.author: kenwith
ms.custom: aaddev
ms.reviewer: paulgarn
ms.openlocfilehash: 40bf202e0f14f18d817e4e918f8372ba3c0a4ad8
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91950667"
---
# <a name="single-sign-on-saml-protocol"></a>Protocole SAML d’authentification unique

Cet article traite des requêtes et réponses d’authentification SAML 2.0 prises en charge par Azure Active Directory (Azure AD) dans le cadre de l’authentification unique (SSO).

Le schéma de protocole ci-dessous décrit la séquence d’authentification unique. Le service cloud (le fournisseur de services) utilise une liaison de redirection HTTP pour transmettre un élément `AuthnRequest` (demande d’authentification) à Azure AD (le fournisseur d’identité). Azure AD utilise ensuite une liaison HTTP POST pour valider un élément `Response` auprès du service cloud.

![Flux de travail de l’authentification unique (SSO)](./media/single-sign-on-saml-protocol/active-directory-saml-single-sign-on-workflow.png)

> [!NOTE]
> Cet article traite de l’utilisation de SAML pour l’authentification unique. Pour plus d’informations sur d’autres façons de gérer l’authentification unique (par exemple, à l’aide d’OpenID Connect ou de l’Authentification Windows intégrée), consultez [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="authnrequest"></a>AuthnRequest

Pour demander une authentification utilisateur, les services cloud envoient un élément `AuthnRequest` à Azure AD. Exemple de `AuthnRequest` SAML 2.0 :

```
<samlp:AuthnRequest
xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
ID="id6c1c178c166d486687be4aaf5e482730"
Version="2.0" IssueInstant="2013-03-18T03:28:54.1839884Z"
xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
<Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://www.contoso.com</Issuer>
</samlp:AuthnRequest>
```

| Paramètre | Type | Description |
| --- | --- | --- |
| id | Obligatoire | Azure AD utilise cet attribut pour compléter l’attribut `InResponseTo` de la réponse retournée. L’ID ne doit pas commencer par un nombre ; vous pouvez donc suivre la stratégie courante qui consiste à ajouter une chaîne de type « id » devant la représentation sous forme de chaîne d’un GUID. Par exemple, `id6c1c178c166d486687be4aaf5e482730` est un ID valide. |
| Version | Obligatoire | Ce paramètre doit être défini sur **2.0**. |
| IssueInstant | Obligatoire | Chaîne DateTime associée à une valeur UTC et comportant le [format aller-retour (« o »)](/dotnet/standard/base-types/standard-date-and-time-format-strings). Azure AD attend une valeur DateHeure de ce type, sans pour autant l’évaluer ni l’utiliser. |
| AssertionConsumerServiceURL | Facultatif | Si ce paramètre est fourni, il doit correspondre à l’élément `RedirectUri` du service cloud dans Azure AD. |
| ForceAuthn | Facultatif | Il s’agit d’une valeur booléenne. Une valeur true signifie que l’utilisateur devra se réauthentifier, même s’il a ouvert une session valide avec Azure AD. |
| IsPassive | Facultatif | Il s’agit d’une valeur booléenne qui spécifie si Azure AD doit authentifier l’utilisateur en mode silencieux, sans intervention de l’utilisateur, en utilisant le cookie de la session s’il en existe un. Si la valeur est true, Azure AD tente d’authentifier l’utilisateur en utilisant le cookie de la session. |

Tous les autres attributs `AuthnRequest`, comme Consent, Destination, AssertionConsumerServiceIndex, AttributeConsumerServiceIndex et ProviderName, sont **ignorés**.

Azure AD ignore également l’élément `Conditions` dans `AuthnRequest`.

### <a name="issuer"></a>Émetteur

L’élément `Issuer` dans `AuthnRequest` doit correspondre exactement à l’un des **ServicePrincipalNames** du service cloud dans Azure AD. En règle générale, il est défini sur **l’URI ID d’application** spécifié au moment de l’inscription de l’application.

Exemple d’extrait SAML contenant l’élément `Issuer` :

```
<Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://www.contoso.com</Issuer>
```

### <a name="nameidpolicy"></a>NameIDPolicy

Cet élément demande un format d’ID de nom particulier dans la réponse et est facultatif dans les éléments `AuthnRequest` envoyés à Azure AD.

Voici un exemple d’élément `NameIdPolicy` :

```
<NameIDPolicy Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent"/>
```

Si `NameIDPolicy` est fourni, vous pouvez inclure son attribut `Format` facultatif. L’attribut `Format` ne peut comporter qu’une des valeurs suivantes ; toute autre valeur produira une erreur.

* `urn:oasis:names:tc:SAML:2.0:nameid-format:persistent`: Azure Active Directory émet la revendication NameID sous la forme d’un identificateur par paire.
* `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`: Azure Active Directory émet la revendication NameID sous la forme d’une adresse e-mail.
* `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified`: cette valeur permet à Azure Active Directory de sélectionner le format de revendication. Azure Active Directory émet la revendication NameID sous la forme d’un identificateur par paire.
* `urn:oasis:names:tc:SAML:2.0:nameid-format:transient`: Azure Active Directory émet la revendication NameID sous la forme d’une valeur générée de manière aléatoire et propre à l’opération d’authentification unique en cours. Cela signifie que la valeur est temporaire et ne peut pas être utilisée pour identifier l’utilisateur à l’origine de l’authentification.

Si `SPNameQualifier` est spécifié, Azure AD inclura le même `SPNameQualifier` dans la réponse.

Azure AD ignore l’attribut `AllowCreate` .

### <a name="requestauthncontext"></a>RequestAuthnContext
L’élément `RequestedAuthnContext` spécifie les méthodes d’authentification souhaitées. Il est facultatif dans les éléments `AuthnRequest` envoyés à Azure AD. Azure AD prend en charge des valeurs `AuthnContextClassRef`, telles que `urn:oasis:names:tc:SAML:2.0:ac:classes:Password`.

### <a name="scoping"></a>Scoping
L’élément `Scoping`, qui comprend une liste de fournisseurs d’identité, est facultatif dans les éléments `AuthnRequest` envoyés à Azure AD.

S’il est fourni, n’incluez ni l’attribut `ProxyCount` ni l’élément `IDPListOption` ou `RequesterID`, car ils ne sont pas pris en charge.

### <a name="signature"></a>Signature
Un élément `Signature` dans des éléments `AuthnRequest` est facultatif. Azure AD ne valide pas les demandes d’authentification signées si une signature est présente. La vérification du demandeur est fournie en répondant uniquement aux URL Assertion Consumer Service inscrites.

### <a name="subject"></a>Objet
N’incluez pas d’élément `Subject`. Azure AD ne prend pas en charge la spécification d’un objet pour une requête et renverra une erreur si elle est fournie.

## <a name="response"></a>response
Lorsqu’une demande d’authentification aboutit, Azure AD publie une réponse au service cloud. Exemple de réponse à une tentative d’ouverture de session réussie :

```
<samlp:Response ID="_a4958bfd-e107-4e67-b06d-0d85ade2e76a" Version="2.0" IssueInstant="2013-03-18T07:38:15.144Z" Destination="https://contoso.com/identity/inboundsso.aspx" InResponseTo="id758d0ef385634593a77bdf7e632984b6" xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
  <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion"> https://login.microsoftonline.com/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
  <ds:Signature xmlns:ds="https://www.w3.org/2000/09/xmldsig#">
    ...
  </ds:Signature>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success" />
  </samlp:Status>
  <Assertion ID="_bf9c623d-cc20-407a-9a59-c2d0aee84d12" IssueInstant="2013-03-18T07:38:15.144Z" Version="2.0" xmlns="urn:oasis:names:tc:SAML:2.0:assertion">
    <Issuer>https://login.microsoftonline.com/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
    <ds:Signature xmlns:ds="https://www.w3.org/2000/09/xmldsig#">
      ...
    </ds:Signature>
    <Subject>
      <NameID>Uz2Pqz1X7pxe4XLWxV9KJQ+n59d573SepSAkuYKSde8=</NameID>
      <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
        <SubjectConfirmationData InResponseTo="id758d0ef385634593a77bdf7e632984b6" NotOnOrAfter="2013-03-18T07:43:15.144Z" Recipient="https://contoso.com/identity/inboundsso.aspx" />
      </SubjectConfirmation>
    </Subject>
    <Conditions NotBefore="2013-03-18T07:38:15.128Z" NotOnOrAfter="2013-03-18T08:48:15.128Z">
      <AudienceRestriction>
        <Audience>https://www.contoso.com</Audience>
      </AudienceRestriction>
    </Conditions>
    <AttributeStatement>
      <Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name">
        <AttributeValue>testuser@contoso.com</AttributeValue>
      </Attribute>
      <Attribute Name="http://schemas.microsoft.com/identity/claims/objectidentifier">
        <AttributeValue>3F2504E0-4F89-11D3-9A0C-0305E82C3301</AttributeValue>
      </Attribute>
      ...
    </AttributeStatement>
    <AuthnStatement AuthnInstant="2013-03-18T07:33:56.000Z" SessionIndex="_bf9c623d-cc20-407a-9a59-c2d0aee84d12">
      <AuthnContext>
        <AuthnContextClassRef> urn:oasis:names:tc:SAML:2.0:ac:classes:Password</AuthnContextClassRef>
      </AuthnContext>
    </AuthnStatement>
  </Assertion>
</samlp:Response>
```

### <a name="response"></a>response

L’élément `Response` inclut le résultat de la demande d’autorisation. Azure AD définit les valeurs `ID`, `Version` et `IssueInstant` dans l’élément `Response`. Il définit également les attributs suivants :

* `Destination`: lorsque l’authentification aboutit, il est défini sur l’élément `RedirectUri` du fournisseur de services (service cloud).
* `InResponseTo`: cet attribut est défini sur l’attribut `ID` de l’élément `AuthnRequest` qui a émis la réponse.

### <a name="issuer"></a>Émetteur

Azure AD définit l’élément `Issuer` sur `https://sts.windows.net/<TenantIDGUID>/`, où \<TenantIDGUID> correspond à l’ID client du client Azure AD.

Exemple de réponse comportant l’élément Issuer :

```
<Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion"> https://sts.windows.net/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
```

### <a name="status"></a>Statut

L’élément `Status` indique si l’authentification a abouti ou échoué. Il comporte l’élément `StatusCode`, qui contient un code ou un ensemble de codes imbriqués représentant l’état de la demande. Il inclut également l’élément `StatusMessage` , qui contient des messages d’erreur personnalisés générés pendant le processus d’authentification.

<!-- TODO: Add an authentication protocol error reference -->

Exemple de réponse SAML à une tentative d’ouverture de session infructueuse :

```
<samlp:Response ID="_f0961a83-d071-4be5-a18c-9ae7b22987a4" Version="2.0" IssueInstant="2013-03-18T08:49:24.405Z" InResponseTo="iddce91f96e56747b5ace6d2e2aa9d4f8c" xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
  <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://sts.windows.net/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Requester">
      <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:RequestUnsupported" />
    </samlp:StatusCode>
    <samlp:StatusMessage>AADSTS75006: An error occurred while processing a SAML2 Authentication request. AADSTS90011: The SAML authentication request property 'NameIdentifierPolicy/SPNameQualifier' is not supported.
Trace ID: 66febed4-e737-49ff-ac23-464ba090d57c
Timestamp: 2013-03-18 08:49:24Z</samlp:StatusMessage>
  </samlp:Status>
```

### <a name="assertion"></a>Assertion

Outre les éléments `ID`, `IssueInstant` et `Version`, Azure Active Directory définit les éléments suivants dans l’élément `Assertion` de la réponse.

#### <a name="issuer"></a>Émetteur

Cet élément est défini sur `https://sts.windows.net/<TenantIDGUID>/`, où \<TenantIDGUID> correspond à l’ID client du client Azure AD.

```
<Issuer>https://sts.windows.net/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
```

#### <a name="signature"></a>Signature

Azure AD signe l’assertion en réponse à une ouverture de session réussie. L’élément `Signature` contient une signature numérique que le service cloud peut utiliser pour authentifier la source afin de vérifier l’intégrité de l’assertion.

Pour générer cette signature numérique, Azure AD utilise la clé de signature spécifiée dans l’élément `IDPSSODescriptor` de son document de métadonnées.

```
<ds:Signature xmlns:ds="https://www.w3.org/2000/09/xmldsig#">
      digital_signature_here
    </ds:Signature>
```

#### <a name="subject"></a>Objet

Spécifie le principal qui fait l’objet des instructions contenues dans l’assertion. Il contient un élément `NameID` qui représente l’utilisateur authentifié. La valeur `NameID` est un identificateur ciblé qui est dirigé uniquement vers le fournisseur de services visé pour le jeton. Elle est persistante : elle peut être révoquée, mais n’est jamais réaffectée. Elle est également opaque, car elle ne révèle rien sur l’utilisateur et ne peut pas être utilisée comme identificateur pour les requêtes d’attribut.

L’attribut `Method` de l’élément `SubjectConfirmation` est toujours défini sur `urn:oasis:names:tc:SAML:2.0:cm:bearer`.

```
<Subject>
      <NameID>Uz2Pqz1X7pxe4XLWxV9KJQ+n59d573SepSAkuYKSde8=</NameID>
      <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
        <SubjectConfirmationData InResponseTo="id758d0ef385634593a77bdf7e632984b6" NotOnOrAfter="2013-03-18T07:43:15.144Z" Recipient="https://contoso.com/identity/inboundsso.aspx" />
      </SubjectConfirmation>
</Subject>
```

#### <a name="conditions"></a>Conditions

Cet élément spécifie les conditions qui définissent l’usage acceptable des assertions SAML.

```
<Conditions NotBefore="2013-03-18T07:38:15.128Z" NotOnOrAfter="2013-03-18T08:48:15.128Z">
      <AudienceRestriction>
        <Audience>https://www.contoso.com</Audience>
      </AudienceRestriction>
</Conditions>
```

Les attributs `NotBefore` et `NotOnOrAfter` spécifient l’intervalle pendant lequel l’assertion est valide.

* La valeur de l’attribut `NotBefore` est égale à la valeur de l’attribut `IssueInstant` de l’élément `Assertion`, ou légèrement supérieure (moins d’une seconde). Azure AD ne tient pas compte des différences de temps entre sa propre horloge et celle du service cloud (fournisseur de services) et n’ajoute pas de mémoire tampon à cette durée.
* La valeur de l’attribut `NotOnOrAfter` est de 70 minutes de plus que la valeur de l’attribut `NotBefore`.

#### <a name="audience"></a>Public visé

Contient un URI qui identifie une audience visée. Azure AD définit la valeur de cet élément sur la valeur de l’élément `Issuer` de l’attribut `AuthnRequest` qui a lancé la session. Pour évaluer la valeur `Audience`, utilisez la valeur de `App ID URI` spécifiée lors de l’inscription de l’application.

```
<AudienceRestriction>
        <Audience>https://www.contoso.com</Audience>
</AudienceRestriction>
```

Comme la valeur `Issuer`, la valeur `Audience` doit correspondre exactement à un des noms de principaux du service qui représentent le service cloud dans Azure AD. Toutefois, si la valeur de l’élément `Issuer` n’est pas une valeur d’URI, la valeur `Audience` contenue dans la réponse correspond à la valeur `Issuer` précédée de `spn:`.

#### <a name="attributestatement"></a>AttributeStatement

Contient les revendications sur l’objet ou l’utilisateur. L’extrait suivant contient un exemple d’élément `AttributeStatement` . Les points de suspension indiquent que l’élément peut contenir plusieurs attributs et valeurs d’attribut.

```
<AttributeStatement>
      <Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name">
        <AttributeValue>testuser@contoso.com</AttributeValue>
      </Attribute>
      <Attribute Name="http://schemas.microsoft.com/identity/claims/objectidentifier">
        <AttributeValue>3F2504E0-4F89-11D3-9A0C-0305E82C3301</AttributeValue>
      </Attribute>
      ...
</AttributeStatement>
```        

* **Revendication Name** : la valeur de l’attribut `Name` (`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`) est le nom d’utilisateur principal de l’utilisateur authentifié, par exemple, `testuser@managedtenant.com`.
* **Réclamation ObjectIdentifier** : la valeur de l’attribut `ObjectIdentifier` (`http://schemas.microsoft.com/identity/claims/objectidentifier`) est l’élément `ObjectId` de l’objet annuaire représentant l’utilisateur authentifié dans Azure AD. `ObjectId` est un identificateur global unique sûr, immuable et réutilisable de l’utilisateur authentifié.

#### <a name="authnstatement"></a>AuthnStatement

Cet élément déclare que le sujet de l’assertion a été authentifié par un moyen précis à un moment donné.

* L’attribut `AuthnInstant` spécifie l’heure à laquelle l’utilisateur s’est authentifié auprès d’Azure AD.
* L’élément `AuthnContext` spécifie le contexte d’authentification utilisé pour authentifier l’utilisateur.

```
<AuthnStatement AuthnInstant="2013-03-18T07:33:56.000Z" SessionIndex="_bf9c623d-cc20-407a-9a59-c2d0aee84d12">
      <AuthnContext>
        <AuthnContextClassRef> urn:oasis:names:tc:SAML:2.0:ac:classes:Password</AuthnContextClassRef>
      </AuthnContext>
</AuthnStatement>
```
