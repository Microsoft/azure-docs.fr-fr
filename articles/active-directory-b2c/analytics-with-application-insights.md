---
title: Suivre le comportement des utilisateurs avec Application Insights
titleSuffix: Azure AD B2C
description: Découvrez comment activer les journaux d’événements dans Application Insights à partir des parcours utilisateur Azure AD B2C à l’aide de stratégies personnalisées.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.date: 04/05/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 67ea7324419d86fa5b5c23a2f0aa5f8c057495d1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85385975"
---
# <a name="track-user-behavior-in-azure-active-directory-b2c-using-application-insights"></a>Suivre le comportement des utilisateurs dans Azure Active Directory B2C à l’aide d’Application Insights

[!INCLUDE [active-directory-b2c-public-preview](../../includes/active-directory-b2c-public-preview.md)]

Azure Active Directory B2C (Azure AD B2C) prend en charge l’envoi de données d’événement directement à [Application Insights](../azure-monitor/app/app-insights-overview.md) à l’aide de la clé d’instrumentation fournie à Azure AD B2C.  Avec un profil technique Application Insights, vous pouvez obtenir des journaux des événements détaillés et personnalisés pour vos parcours utilisateur :

* Obtenir des informations détaillées sur le comportement des utilisateurs
* Résoudre les problèmes de vos propres stratégies en développement ou en production
* Mesurer les performances
* Créer des notifications à partir d’Application Insights

## <a name="how-it-works"></a>Fonctionnement

Le profil technique [Application Insights](application-insights-technical-profile.md) définit un événement d’Azure AD B2C. Le profil spécifie le nom de l’événement, les revendications qui sont enregistrées et la clé d’instrumentation. Pour publier un événement, le profil technique est ensuite ajouté en tant qu’étape d’orchestration dans un [parcours utilisateur](userjourneys.md).

Application Insights peut unifier les événements en utilisant un ID de corrélation pour enregistrer une session utilisateur. Application Insights rend la session et les événements disponibles en quelques secondes et présente de nombreux outils d’analyse, d’exportation et de visualisation.

## <a name="prerequisites"></a>Prérequis

Suivez les étapes décrites dans [Bien démarrer avec les stratégies personnalisées dans Azure Active Directory B2C](custom-policy-get-started.md). Vous devez disposer d’une stratégie personnalisée fonctionnelle pour l’inscription et la connexion avec des comptes locaux.

## <a name="create-an-application-insights-resource"></a>Création d’une ressource Application Insights dans Azure

Quand vous utilisez Application Insights avec Azure AD B2C, vous devez uniquement créer une ressource et obtenir la clé d’instrumentation. Pour plus d’informations, consultez [Création d’une ressource Application Insights dans Azure](../azure-monitor/app/create-new-resource.md).

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Veillez à utiliser l’annuaire qui contient votre abonnement Azure en sélectionnant le filtre **Annuaire et abonnement** dans le menu supérieur et en choisissant l’annuaire qui contient votre abonnement. Ce locataire n’est pas votre locataire Azure AD B2C.
3. Choisissez **Créer une ressource** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Application Insights**.
4. Cliquez sur **Créer**.
5. Entrez un **Nom** pour la ressource.
6. Pour **Type d’application**, sélectionnez **Application web ASP.NET**.
7. Pour **Groupe de ressources**, sélectionnez un groupe existant ou entrez un nom pour un nouveau groupe.
8. Cliquez sur **Créer**.
4. Une fois la ressource Application Insights créée, ouvrez-la, développez **Éléments principaux**, puis copiez la clé d’instrumentation.

![Vue d’ensemble d’Application Insights et clé d’instrumentation](./media/analytics-with-application-insights/app-insights.png)

## <a name="define-claims"></a>Définition de revendications

Une revendication fournit un stockage temporaire de données lors d’une exécution de stratégie Azure AD B2C. Le [schéma de revendications](claimsschema.md) est l’endroit où vous déclarez vos revendications.

1. Ouvrez le fichier d’extensions de votre stratégie. Par exemple <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>.
1. Recherchez l’élément [BuildingBlocks](buildingblocks.md). Si l’élément n’existe pas, ajoutez-le.
1. Localisez l’élément [ClaimsSchema](claimsschema.md). Si l’élément n’existe pas, ajoutez-le.
1. Ajoutez les revendications suivantes à l’élément **ClaimsSchema**. 

```xml
<ClaimType Id="EventType">
  <DisplayName>Event type</DisplayName>
  <DataType>string</DataType>
</ClaimType>
<ClaimType Id="EventTimestamp">
  <DisplayName>Event timestamp</DisplayName>
  <DataType>string</DataType>
</ClaimType>
<ClaimType Id="PolicyId">
  <DisplayName>Policy Id</DisplayName>
  <DataType>string</DataType>
</ClaimType>
<ClaimType Id="Culture">
  <DisplayName>Culture ID</DisplayName>
  <DataType>string</DataType>
</ClaimType>
<ClaimType Id="CorrelationId">
  <DisplayName>Correlation Id</DisplayName>
  <DataType>string</DataType>
</ClaimType>
<ClaimType Id="federatedUser">
  <DisplayName>Federated user</DisplayName>
  <DataType>boolean</DataType>
</ClaimType>
<ClaimType Id="parsedDomain">
  <DisplayName>Domain name</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>The domain portion of the email address.</UserHelpText>
</ClaimType>
<ClaimType Id="userInLocalDirectory">
  <DisplayName>userInLocalDirectory</DisplayName>
  <DataType>boolean</DataType>
</ClaimType>
```

## <a name="add-new-technical-profiles"></a>Ajouter de nouveaux profils techniques

Les profils techniques peuvent être considérés comme des fonctions dans l’infrastructure d’expérience d’identité d’Azure AD B2C. Ce tableau définit les profils techniques qui servent à ouvrir une session et à publier des événements.

| Profil technique | Tâche |
| ----------------- | -----|
| AppInsights-Common | L’ensemble commun de paramètres à inclure dans tous les profils techniques Azure Insights. |
| AppInsights-SignInRequest | Enregistre un événement `SignInRequest` avec un ensemble de revendications quand une demande de connexion a été reçue. |
| AppInsights-UserSignUp | Enregistre un événement `UserSignUp` quand l’utilisateur déclenche l’option d’inscription dans un parcours d’inscription/connexion. |
| AppInsights-SignInComplete | Enregistre un événement `SignInComplete` lorsque l’authentification a réussi quand un jeton a été envoyé à l’application par partie de confiance. |

Ajoutez les profils au fichier *TrustFrameworkExtensions.xml* à partir du pack de démarrage. Ajoutez ces éléments à l’élément **ClaimsProviders** :

```xml
<ClaimsProvider>
  <DisplayName>Application Insights</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="AppInsights-Common">
      <DisplayName>Application Insights</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.Insights.AzureApplicationInsightsProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <!-- The ApplicationInsights instrumentation key which will be used for logging the events -->
        <Item Key="InstrumentationKey">xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx</Item>
        <Item Key="DeveloperMode">false</Item>
        <Item Key="DisableTelemetry ">false</Item>
      </Metadata>
      <InputClaims>
        <!-- Properties of an event are added through the syntax {property:NAME}, where NAME is property being added to the event. DefaultValue can be either a static value or a value that's resolved by one of the supported DefaultClaimResolvers. -->
        <InputClaim ClaimTypeReferenceId="EventTimestamp" PartnerClaimType="{property:EventTimestamp}" DefaultValue="{Context:DateTimeInUtc}" />
        <InputClaim ClaimTypeReferenceId="PolicyId" PartnerClaimType="{property:Policy}" DefaultValue="{Policy:PolicyId}" />
        <InputClaim ClaimTypeReferenceId="CorrelationId" PartnerClaimType="{property:CorrelationId}" DefaultValue="{Context:CorrelationId}" />
        <InputClaim ClaimTypeReferenceId="Culture" PartnerClaimType="{property:Culture}" DefaultValue="{Culture:RFC5646}" />
      </InputClaims>
    </TechnicalProfile>

    <TechnicalProfile Id="AppInsights-SignInRequest">
      <InputClaims>
        <!-- An input claim with a PartnerClaimType="eventName" is required. This is used by the AzureApplicationInsightsProvider to create an event with the specified value. -->
        <InputClaim ClaimTypeReferenceId="EventType" PartnerClaimType="eventName" DefaultValue="SignInRequest" />
      </InputClaims>
      <IncludeTechnicalProfile ReferenceId="AppInsights-Common" />
    </TechnicalProfile>

    <TechnicalProfile Id="AppInsights-UserSignUp">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="EventType" PartnerClaimType="eventName" DefaultValue="UserSignUp" />
      </InputClaims>
      <IncludeTechnicalProfile ReferenceId="AppInsights-Common" />
    </TechnicalProfile>
    
    <TechnicalProfile Id="AppInsights-SignInComplete">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="EventType" PartnerClaimType="eventName" DefaultValue="SignInComplete" />
        <InputClaim ClaimTypeReferenceId="federatedUser" PartnerClaimType="{property:FederatedUser}" DefaultValue="false" />
        <InputClaim ClaimTypeReferenceId="parsedDomain" PartnerClaimType="{property:FederationPartner}" DefaultValue="Not Applicable" />
      </InputClaims>
      <IncludeTechnicalProfile ReferenceId="AppInsights-Common" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

> [!IMPORTANT]
> Modifiez la clé d’instrumentation dans le profil technique `AppInsights-Common` et remplacez-le par le GUID fourni par votre ressource Application Insights.

## <a name="add-the-technical-profiles-as-orchestration-steps"></a>Ajouter les profils techniques en tant qu’étapes d’orchestration

Appelez `AppInsights-SignInRequest` en tant qu’étape d’orchestration 2 pour suivre une demande de connexion/d’inscription reçue :

```xml
<!-- Track that we have received a sign in request -->
<OrchestrationStep Order="2" Type="ClaimsExchange">
  <ClaimsExchanges>
    <ClaimsExchange Id="TrackSignInRequest" TechnicalProfileReferenceId="AppInsights-SignInRequest" />
  </ClaimsExchanges>
</OrchestrationStep>
```

Immédiatement *avant* l’étape d’orchestration `SendClaims`, ajoutez une nouvelle étape qui appelle `AppInsights-UserSignup`. Celle-ci est déclenchée quand l’utilisateur sélectionne le bouton d’inscription dans un parcours d’inscription/connexion.

```xml
<!-- Handles the user clicking the sign up link in the local account sign in page -->
<OrchestrationStep Order="8" Type="ClaimsExchange">
  <Preconditions>
    <Precondition Type="ClaimsExist" ExecuteActionsIf="false">
      <Value>newUser</Value>
      <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
    <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
      <Value>newUser</Value>
      <Value>false</Value>
      <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
  </Preconditions>
  <ClaimsExchanges>
    <ClaimsExchange Id="TrackUserSignUp" TechnicalProfileReferenceId="AppInsights-UserSignup" />
  </ClaimsExchanges>
</OrchestrationStep>
```

Immédiatement après l’étape d’orchestration `SendClaims`, appelez `AppInsights-SignInComplete`. Cette étape montre la réussite d’un parcours.

```xml
<!-- Track that we have successfully sent a token -->
<OrchestrationStep Order="10" Type="ClaimsExchange">
  <ClaimsExchanges>
    <ClaimsExchange Id="TrackSignInComplete" TechnicalProfileReferenceId="AppInsights-SignInComplete" />
  </ClaimsExchanges>
</OrchestrationStep>
```

> [!IMPORTANT]
> Après avoir ajouté les nouvelles étapes d’orchestration, renumérotez les étapes séquentiellement sans sauter d’entiers de 1 à N.


## <a name="upload-your-file-run-the-policy-and-view-events"></a>Charger votre fichier, exécuter la stratégie et afficher les événements

Enregistrez et chargez le fichier *TrustFrameworkExtensions.xml*. Ensuite, appelez la stratégie de partie de confiance à partir de votre application ou utilisez l’option **Exécuter maintenant** dans le portail Azure. Quelques secondes plus tard, vos événements sont disponibles dans Application Insights.

1. Ouvrez la ressource **Application Insights** dans votre locataire Azure Active Directory.
2. Sélectionnez **Utilisation** > **Événements**.
3. Définissez **Pendant** sur **Dernière heure** et **Par** sur **3 minutes**.  Vous devrez peut-être sélectionner **Actualiser** pour afficher les résultats.

![Panneau d’événements - UTILISATION Application Insights](./media/analytics-with-application-insights/app-ins-graphic.png)

## <a name="optional-collect-more-data"></a>[Facultatif] Collecter plus de données

Ajoutez des types de revendication et des événements à votre parcours utilisateur en fonction de vos besoins. Vous pouvez utiliser des [résolveurs de revendication](claim-resolver-overview.md) ou n’importe quel type de revendication sous forme de chaîne, ou ajouter les revendications en ajoutant un élément **InputClaim** à l’événement Application Insights ou au profil technique AppInsights-Common.

- **ClaimTypeReferenceId** est la référence à un type de revendication.
- **PartnerClaimType** est le nom de la propriété qui s’affiche dans Azure Insights. Utilisez la syntaxe `{property:NAME}`, où `NAME` est la propriété qui est ajoutée à l’événement.
- **DefaultValue** est n’importe quelle valeur de chaîne ou le résolveur de revendication.

```xml
<InputClaim ClaimTypeReferenceId="app_session" PartnerClaimType="{property:app_session}" DefaultValue="{OAUTH-KV:app_session}" />
<InputClaim ClaimTypeReferenceId="loyalty_number" PartnerClaimType="{property:loyalty_number}" DefaultValue="{OAUTH-KV:loyalty_number}" />
<InputClaim ClaimTypeReferenceId="language" PartnerClaimType="{property:language}" DefaultValue="{Culture:RFC5646}" />
```

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur le profil technique [Application Insights](application-insights-technical-profile.md) dans la référence IEF. 
