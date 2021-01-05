---
title: Échanges de revendications de l’API REST dans une stratégie personnalisée B2C
titleSuffix: Azure AD B2C
description: Introduction à la création d’un parcours utilisateur Azure AD B2C interagissant avec des services RESTful.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 05/18/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: bc2b72779460c2b7e3999204ace50ca57388b9a2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89594184"
---
# <a name="integrate-rest-api-claims-exchanges-in-your-azure-ad-b2c-custom-policy"></a>Intégrer des échanges de revendications de l’API REST dans votre stratégie personnalisée Azure AD B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

La plateforme Identity Experience Framework sur laquelle repose Azure Active Directory B2C (Azure AD B2C) peut s’intégrer avec des API RESTful à l’intérieur d’un parcours utilisateur. Cet article explique comment créer un parcours utilisateur interagissant avec un service RESTful à l’aide d’un [profil technique RESTful](restful-technical-profile.md).

Azure AD B2C vous permet d’ajouter votre propre logique métier à un parcours utilisateur en appelant votre propre service RESTful. La plateforme Identity Experience Framework peut envoyer et recevoir des données de votre service RESTful pour échanger des revendications. Vous pouvez par exemple :

- **Valider des données d’entrée utilisateur**. Par exemple, vous pouvez vérifier que l’adresse e-mail fournie par l’utilisateur existe dans la base de données de votre client et, si ce n’est pas le cas, présenter une erreur.
- **Traiter des revendications**. Si un utilisateur entre son prénom entièrement en minuscules ou majuscules, votre API REST peut modifier la mise en forme en utilisant une majuscule uniquement pour la première lettre avant de le renvoyer à Azure AD B2C.
- **Enrichir les données utilisateur en les intégrant davantage avec des applications métier d’entreprise**. votre service RESTful peut recevoir l’adresse e-mail de l’utilisateur, interroger la base de données de clients et retourner le numéro de fidélité de l’utilisateur à Azure AD B2C. Les revendications retournées peuvent alors être stockées dans le compte Azure AD de l’utilisateur, évaluées dans les étapes d’orchestration suivantes ou incluses dans le jeton d’accès.
- **Exécuter une logique métier personnalisée**. Vous pouvez envoyer des notifications Push, mettre à jour des bases de données d’entreprise, exécuter un processus de migration utilisateur, gérer les autorisations, auditer des bases de données et effectuer d’autres flux de travail.

![Diagramme d’un échange de revendications de service RESTful](media/custom-policy-rest-api-intro/restful-service-claims-exchange.png)

> [!NOTE]
> S’il n’y a pas de réponse ou que la réponse est lente depuis le service RESTful vers Azure AD B2C, le délai d’expiration est de 30 secondes et le nombre de tentatives est de 2 (ce qui signifie qu’il y a trois tentatives au total). Les paramètres de délai d’expiration et de nombre de tentatives ne sont pas configurables actuellement.

## <a name="calling-a-restful-service"></a>Appel d’un service RESTful

Cette interaction inclut un échange de revendications d’informations entre les revendications de l’API REST et Azure AD B2C. Vous pouvez concevoir l’intégration aux services RESTful comme suit :

- **Profil technique de validation**. L’appel au service RESTful se produit à l’intérieur d’un [profil technique de validation](validation-technical-profile.md) du [profil technique autodéclaré](self-asserted-technical-profile.md) spécifié, ou d’un [contrôle d’affichage de vérification](display-control-verification.md) d’un [contrôle d’affichage](display-controls.md). Le profil technique de validation valide les données fournies par l’utilisateur avant la suite du parcours utilisateur. Avec le profil technique de validation, vous pouvez :

  - Envoyer des revendications à votre API REST.
  - Valider les revendications et lever des messages d’erreur personnalisés qui s’affichent pour l’utilisateur.
  - Renvoyer les revendications de l’API REST aux étapes d’orchestration suivantes.

- **Échange de revendications**. Vous pouvez configurer un échange direct de revendications en appelant un profil technique d’API REST directement à partir d’une étape d’orchestration d’un [parcours utilisateur](userjourneys.md). Cette définition est limitée à :

  - Envoyer des revendications à votre API REST.
  - Valider les revendications et lever des messages d’erreur personnalisés qui sont retournés à l’application.
  - Renvoyer les revendications de l’API REST aux étapes d’orchestration suivantes.

Vous pouvez ajouter un appel d’API REST à n’importe quelle étape du parcours utilisateur défini par une stratégie personnalisée. Par exemple, vous pouvez appeler une API REST :

- lors de la connexion, juste qu’Azure AD B2C valide les informations d’identification ;
- immédiatement après la connexion ;
- avant qu’Azure AD B2C crée un compte dans l’annuaire ;
- après qu’Azure AD B2C a créé un compte dans l’annuaire ;
- avant qu’Azure AD B2C émette un jeton d’accès.

![Collecte de profil technique de validation](media/custom-policy-rest-api-intro/validation-technical-profile.png)

## <a name="sending-data"></a>Envoi de données

Dans le [profil technique RESTful](restful-technical-profile.md), l’élément `InputClaims` contient une liste de revendications à envoyer à votre service RESTful. Vous pouvez mapper le nom de votre revendication au nom défini dans le service RESTful, définir une valeur par défaut et utiliser des [programmes de résolution de revendications](claim-resolver-overview.md).

Vous pouvez configurer la façon dont les revendications d’entrée sont envoyées au fournisseur de revendications RESTful en utilisant l’attribut SendClaimsIn. Les valeurs possibles sont les suivantes :

- **Body**, envoyé dans le corps de la requête HTTP POST au format JSON.
- **Form**, envoyé dans le corps de la requête HTTP POST au format de valeur de clé séparée par des perluètes (« & »).
- **Header**, envoyé dans l’en-tête de requête HTTP GET.
- **QueryString**, envoyé dans la chaîne de requête HTTP GET.

Quand l’option **Body** est configurée, le profil technique de l’API REST vous permet d’envoyer une charge utile JSON complexe à un point de terminaison. Pour plus d’informations, consultez [Envoyer une charge utile JSON](restful-technical-profile.md#send-a-json-payload).

## <a name="receiving-data"></a>Réception de données

L’élément `OutputClaims` du [profil technique RESTful](restful-technical-profile.md) contient une liste de revendications retournées par l’API REST. Il se peut que vous deviez mapper le nom de la revendication définie dans votre stratégie au nom défini dans l’API REST. Vous pouvez également inclure des revendications qui ne sont pas retournées par le fournisseur d’identité de l’API REST, pour autant que vous définissiez l’attribut DefaultValue.

Les revendications de sortie analysées par le fournisseur de revendications RESTful s’attendent toujours à analyser une réponse de corps JSON plate, par exemple :

```json
{
  "name": "Emily Smith",
  "email": "emily@outlook.com",
  "loyaltyNumber":  1234
}
```

Les revendications de sortie doivent ressembler à ceci :

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
  <OutputClaim ClaimTypeReferenceId="email" />
  <OutputClaim ClaimTypeReferenceId="loyaltyNumber" />
</OutputClaims>
```

Pour analyser une réponse de corps JSON imbriquée, définissez les métadonnées ResolveJsonPathsInJsonTokens sur true. Dans la revendication de sortie, définissez PartnerClaimType sur l’élément de chemin d’accès JSON que vous souhaitez générer.

```json
"contacts": [
  {
    "id": "MAINCONTACT_1",
    "person": {
      "name": "Emily Smith",
      "loyaltyNumber":  1234,
      "emails": [
        {
          "id": "EMAIL_1",
          "type": "WORK",
          "email": "email@domain.com"
        }
      ]
    }
  }
],
```


Les revendications de sortie doivent ressembler à ceci :

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="contacts[0].person.name" />
  <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="contacts[0].person.emails[0].email" />
  <OutputClaim ClaimTypeReferenceId="loyaltyNumber" PartnerClaimType="contacts[0].person.loyaltyNumber" />
</OutputClaims>
```

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Vous devez protéger le point de terminaison de votre API REST afin que seuls des clients authentifiés puissent communiquer avec celui-ci. L’API REST doit utiliser un point de terminaison HTTPS. Définissez les métadonnées AuthenticationType sur l’une des méthodes d’authentification suivantes :

- La méthode **Certificat client** restreint l’accès à l’aide de l’authentification du certificat client. Seuls des services disposant des certificats appropriés peuvent accéder à votre API. Vous stockez le certificat client dans une clé de stratégie de Azure AD B2C. Apprenez-en davantage sur la manière de [Sécuriser votre service RESTful à l’aide de certificats clients](secure-rest-api.md#https-client-certificate-authentication).
- La méthode **De base** sécurise l’API REST avec une authentification HTTP de base. Seuls des utilisateurs vérifiés, notamment Azure AD B2C, peuvent accéder à votre API. Le nom d’utilisateur et le mot de passe sont stockés dans des clés de stratégie Azure AD B2C. Découvrez comment [Sécuriser vos services RESTful à l’aide d’une authentification HTTP de base](secure-rest-api.md#http-basic-authentication).
- La méthode **Bearer** restreint l’accès à l’aide d’un jeton d’accès OAuth2 client. Le jeton d’accès est stocké dans une clé de stratégie Azure AD B2C. Apprenez-en davantage sur la manière de [sécuriser votre service RESTful à l’aide d’un jeton Bearer](secure-rest-api.md#oauth2-bearer-authentication).

## <a name="rest-api-platform"></a>Plateforme de l’API REST
Votre API REST peut être basée sur n’importe quelle plateforme et écrite dans n’importe quel langage de programmation pour autant qu’elle soit sécurisée et puisse envoyer et recevoir des revendications comme indiqué dans le [profil technique RESTful](restful-technical-profile.md).

## <a name="localize-the-rest-api"></a>Localisez l’API REST
Dans un profil technique RESTful, vous pouvez envoyer la langue/les paramètres régionaux de la session active et, si nécessaire, déclencher un message d’erreur localisé. Le [programme de résolution des revendications](claim-resolver-overview.md) vous permet d’envoyer une revendication contextuelle, telle que la langue de l’utilisateur. L’exemple suivant présente un profil technique RESTful illustrant ce scénario.

```xml
<TechnicalProfile Id="REST-ValidateUserData">
  <DisplayName>Validate user input data</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="ServiceUrl">https://your-app.azurewebsites.net/api/identity</Item>
    <Item Key="AuthenticationType">None</Item>
    <Item Key="SendClaimsIn">Body</Item>
    <Item Key="IncludeClaimResolvingInClaimsHandling">true</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="userLanguage" DefaultValue="{Culture:LCID}" AlwaysUseDefaultValue="true" />
    <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="emailAddress" />
  </InputClaims>
  <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
</TechnicalProfile>
```

## <a name="handling-error-messages"></a>Gestion des messages d’erreur

Il se peut que votre API REST doive retourner un message d’erreur tel que « Utilisateur introuvable dans le système CRM ». Quand une erreur se produit, l’API REST doit retourner un message d’erreur HTTP 409 (code d’état de réponse Conflit). Pour plus d’informations, consultez le [Profil technique RESTful ](restful-technical-profile.md#returning-validation-error-message).

Pour ce faire, il suffit d’appeler un profil technique d’API REST à partir d’un profil technique de validation. Cela permet à l’utilisateur de corriger les données sur la page et de réexécuter la validation lors de l’envoi de la page.

Une réponse HTTP 409 est nécessaire pour empêcher le traitement des profils techniques de validation suivants dans le cadre de cette étape d’orchestration.

Si vous référencez un profil technique d’API REST directement à partir d’un parcours utilisateur, l’utilisateur est redirigé vers l’application basée sur les revendications avec le message d’erreur approprié.

## <a name="publishing-your-rest-api"></a>Publication de votre API REST

La demande adressée à votre service d’API REST provient de serveurs Azure AD B2C. Le service de l’API REST doit être publié sur un point de terminaison HTTPS accessible publiquement. Les appels d’API REST proviennent d’une adresse IP de centre de données Azure.

Concevez votre service d’API REST et ses composants sous-jacents (tels que la base de données et le système de fichiers) pour qu’ils soient hautement disponibles.

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour obtenir des exemples d’utilisation d’un profil technique RESTful :

- [Procédure pas à pas : Intégrer les échanges de revendications de l’API REST dans votre parcours utilisateur Azure AD B2C comme validation d’une entrée de l’utilisateur](custom-policy-rest-api-claims-validation.md)
- [Procédure pas à pas : Ajouter des échanges de revendications de l’API REST aux stratégies personnalisées dans Azure Active Directory B2C](custom-policy-rest-api-claims-validation.md)
- [Sécuriser vos services d’API REST](secure-rest-api.md)
- [Reference : Profil technique RESTful](restful-technical-profile.md)
