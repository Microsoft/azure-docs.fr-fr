---
title: Sécuriser l’accès et les données
description: Sécuriser l’accès aux entrées, aux sorties, aux déclencheurs basés sur des requêtes, à l’historique des exécutions, aux tâches de gestion ainsi qu’à d’autres ressources dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: rarayudu, logicappspm
ms.topic: conceptual
ms.date: 11/05/2020
ms.openlocfilehash: 331c55a9f7a489aa58f9d3add7303dc18917215d
ms.sourcegitcommit: 46c5ffd69fa7bc71102737d1fab4338ca782b6f1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94331938"
---
# <a name="secure-access-and-data-in-azure-logic-apps"></a>Accès et données sécurisés dans Azure Logic Apps

Azure Logic Apps s’appuie sur le [Stockage Azure](../storage/index.yml) pour stocker et [chiffrer automatiquement les données au repos](../security/fundamentals/encryption-atrest.md). Ce chiffrement protège vos données et vous aide à répondre aux engagements de votre entreprise en matière de sécurité et de conformité. Par défaut, Stockage Azure utilise des clés managées par Microsoft pour chiffrer vos données. Pour plus d'informations, consultez [Fonctionnalité de chiffrement du service Stockage Azure pour les données au repos](../storage/common/storage-service-encryption.md).

Pour contrôler l’accès aux données sensibles et les protéger dans Azure Logic Apps, vous pouvez configurer des paramètres de sécurité supplémentaires dans les domaines suivants :

* [Accès pour les appels entrants à des déclencheurs basés sur des requêtes](#secure-inbound-requests)
* [Accès aux opérations d’une application logique](#secure-operations)
* [Accès aux entrées et aux sorties de l’historique des exécutions](#secure-run-history)
* [Accès aux entrées de paramètres](#secure-action-parameters)
* [Accès pour les appels sortants à d’autres services et systèmes](#secure-outbound-requests)
* [Blocage de la création de connexions pour des connecteurs spécifiques](#block-connections)
* [Conseils d’isolation pour les applications logiques](#isolation-logic-apps)
* [Base de référence de sécurité Azure pour Azure Logic Apps](../logic-apps/security-baseline.md)

Pour plus d’informations sur la sécurité dans Azure, consultez les rubriques suivantes :

* [Vue d’ensemble du chiffrement Azure](../security/fundamentals/encryption-overview.md)
* [Azure Data Encryption-at-Rest](../security/fundamentals/encryption-atrest.md) (Chiffrement des données Azure au repos)
* [Benchmark de sécurité Azure](../security/benchmarks/overview.md)

<a name="secure-inbound-requests"></a>

## <a name="access-for-inbound-calls-to-request-based-triggers"></a>Accès pour les appels entrants à des déclencheurs basés sur des requêtes

Les appels entrants qu’une application logique reçoit via un déclencheur basé sur des requêtes, tel que le déclencheur de [demande](../connectors/connectors-native-reqres.md) ou le déclencheur [Webhook HTTP](../connectors/connectors-native-webhook.md), prennent en charge le chiffrement et sont sécurisés avec [au minimum Transport Layer Security (TLS) 1.2](https://en.wikipedia.org/wiki/Transport_Layer_Security), précédemment appelé Secure Sockets Layer (SSL). Logic Apps applique cette version lors de la réception d’un appel entrant au déclencheur de requête ou d’un rappel du déclencheur ou d’une action Webhook HTTP. Si vous constatez des erreurs de liaison TLS, assurez-vous d’utiliser le protocole TLS 1.2. Pour plus d'informations, consultez [Résolution du problème lié au protocole TLS 1.0](/security/solving-tls1-problem).

Les appels entrants prennent en charge les suites de chiffrement suivantes :

* TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
* TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
* TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
* TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
* TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
* TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
* TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
* TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256

Voici d’autres façons de limiter l’accès aux déclencheurs qui reçoivent des appels entrants à votre application logique afin que seuls des clients autorisés puissent appeler votre application logique :

* [Générer des signatures d’accès partagé (SAP)](#sas)
* [Activer Azure Active Directory Open Authentication (Azure AD OAuth)](#enable-oauth)
* [Exposer votre application logique avec Gestion des API Azure](#azure-api-management)
* [Limiter les adresses IP entrantes](#restrict-inbound-ip-addresses)

<a name="sas"></a>

### <a name="generate-shared-access-signatures-sas"></a>Générer des signatures d’accès partagé (SAP)

Dans une application logique, l’URL de chaque point de terminaison de requête comprend une [signature d’accès partagé](/rest/api/storageservices/constructing-a-service-sas) (SAP). L’URL est au format suivant :

`https://<request-endpoint-URI>sp=<permissions>sv=<SAS-version>sig=<signature>`

Chaque URL contient le paramètre de requête `sp`, `sv` et `sig`, comme vous pouvez le voir dans le tableau suivant :

| Paramètre de requête. | Description |
|-----------------|-------------|
| `sp` | Spécifie des autorisations pour les méthodes HTTP autorisées à utiliser. |
| `sv` | Spécifie la version SAP à utiliser pour générer la signature. |
| `sig` | Spécifie la signature à utiliser pour authentifier l’accès au déclencheur. La signature est générée à l’aide de l’algorithme SHA256 avec une clé d’accès secrète pour l’ensemble des chemins et des propriétés de l’URL. Cette clé n’est jamais exposée ni publiée. Elle est chiffrée et stockée avec l’application logique. Votre application logique autorise uniquement les déclencheurs contenant une signature valide créée avec la clé secrète. |
|||

Les appels entrants à un point de terminaison de demande ne peuvent utiliser qu’un seul schéma d’autorisation : SAS ou [Azure Active Directory Open Authentication](#enable-oauth). Bien que l’utilisation d’un schéma ne désactive pas l’autre, l’utilisation des deux schémas en même temps provoque une erreur, car le service ne sait quel schéma choisir.

Pour plus d’informations sur la sécurisation de l’accès avec la signature d’accès partagé, consultez les sections suivantes de cette rubrique :

* [Régénération des clés d’accès](#access-keys)
* [Création d’URL de rappel qui arrivent à expiration](#expiring-urls)
* [Création d’URL avec une clé primaire ou une clé secondaire](#primary-secondary-key)

<a name="access-keys"></a>

#### <a name="regenerate-access-keys"></a>Régénération de clés d'accès

Pour générer une nouvelle clé d’accès de sécurité à tout moment, utilisez l’API REST Azure ou le portail Azure. Toutes les URL précédemment générées qui utilisent l’ancienne clé sont invalidées et ne sont plus autorisées à déclencher l’application logique. Les URL que vous récupérez après la regénération sont signées avec la nouvelle clé d’accès.

1. Dans le [portail Azure](https://portal.azure.com), ouvrez l’application logique possédant la clé que vous souhaitez régénérer.

1. Dans le menu de l’application logique, sous **Paramètres**, sélectionnez **Clés d’accès**.

1. Sélectionnez la clé que vous souhaitez regénérer et terminez le processus.

<a name="expiring-urls"></a>

#### <a name="create-expiring-callback-urls"></a>Créer des URL de rappel qui expirent

Si vous décidez de partager l’URL d’un point de terminaison pour un déclencheur basé sur des requêtes, vous pouvez générer des URL de rappel qui utilisent des clés et pour lesquelles des dates d’expiration ont été configurées. De cette façon, vous pouvez facilement restaurer les clés ou restreindre l’accès au déclenchement de votre application logique selon une période donnée. Si vous souhaitez spécifier une date d’expiration pour une URL, utilisez l’[API REST Logic Apps](/rest/api/logic/workflowtriggers), par exemple :

```http
POST /subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.Logic/workflows/<workflow-name>/triggers/<trigger-name>/listCallbackUrl?api-version=2016-06-01
```

Dans le corps, ajoutez la propriété `NotAfter` en utilisant une chaîne de date JSON. Cette propriété retourne une URL de rappel qui est valide uniquement jusqu’à la date et l’heure `NotAfter`.

<a name="primary-secondary-key"></a>

#### <a name="create-urls-with-primary-or-secondary-secret-key"></a>Créer des URL avec une clé de secret principale ou secondaire

Lorsque vous générez ou listez des URL de rappel pour des déclencheurs basés sur une requête, vous pouvez également spécifier la clé à utiliser pour signer l’URL. Pour générer une URL signée par une clé spécifique, utilisez l’[API REST Logic Apps](/rest/api/logic/workflowtriggers), par exemple :

```http
POST /subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.Logic/workflows/<workflow-name>/triggers/<trigger-name>/listCallbackUrl?api-version=2016-06-01
```

Dans le corps, incluez la propriété `KeyType` en tant que `Primary` ou `Secondary`. Cette propriété retourne une URL signée par la clé de sécurité spécifiée.

<a name="enable-oauth"></a>

### <a name="enable-azure-active-directory-open-authentication-azure-ad-oauth"></a>Activer Azure Active Directory Open Authentication (Azure AD OAuth)

Pour les appels entrants à un point de terminaison qui est créé par un déclencheur basé sur une demande, vous pouvez activer [Azure Active Directory Open Authentication (Azure AD OAuth)](../active-directory/develop/index.yml) en définissant ou en ajoutant une stratégie d’autorisation pour votre application logique. De cette façon, les appels entrants utilisent des jetons d’accès [OAuth](../active-directory/develop/access-tokens.md) pour l’autorisation.

Quand votre application logique reçoit une demande entrante incluant un jeton d’accès OAuth, le service Azure Logic Apps compare les revendications du jeton à celles spécifiées par chaque stratégie d’autorisation. S’il existe une correspondance entre les revendications du jeton et toutes celles d’au moins une stratégie, l’autorisation est validée pour la requête entrante. Le jeton peut avoir plus de revendications que le nombre spécifié par la stratégie d’autorisation.

Avant d’activer Azure AD OAuth, passez en revue les considérations suivantes :

* Un appel entrant au point de terminaison de demande ne peut utiliser qu’un seul schéma d’autorisation : Azure AD OAuth ou [SAS (Shared Access Signature)](#sas). Bien que l’utilisation d’un schéma ne désactive pas l’autre, l’utilisation des deux schémas en même temps provoque une erreur, car le service Logic Apps ne sait quel schéma choisir.

* Seuls les schémas d’autorisation [de type porteur](../active-directory/develop/active-directory-v2-protocols.md#tokens) sont pris en charge pour les jetons d’accès OAuth Azure AD, ce qui signifie que l’en-tête `Authorization` pour le jeton d’accès doit spécifier le type `Bearer`.

* Votre application logique est limitée à un nombre maximal de stratégies d’autorisation. Chaque stratégie d’autorisation a également un nombre maximal de [revendications](../active-directory/develop/developer-glossary.md#claim). Pour plus d’informations, consultez [Limites et configuration pour Azure Logic Apps](../logic-apps/logic-apps-limits-and-config.md#authentication-limits).

* Une stratégie d’autorisation doit inclure au moins la revendication **Émetteur**, dont la valeur commence par `https://sts.windows.net/` ou `https://login.microsoftonline.com/` (OAuth v2) en tant qu’ID de l’émetteur Azure AD.

  Supposons, par exemple, que votre application logique dispose d’une stratégie d’autorisation qui nécessite deux types de revendication, **Public ciblé** et **Émetteur**. Cet exemple de [section de charge utile](../active-directory/develop/access-tokens.md#payload-claims) pour un jeton d’accès décodé comprend les deux types de revendication, où `aud` est la valeur de **Public ciblé** et `iss` la valeur d’**Émetteur** :

  ```json
  {
      "aud": "https://management.core.windows.net/",
      "iss": "https://sts.windows.net/<Azure-AD-issuer-ID>/",
      "iat": 1582056988,
      "nbf": 1582056988,
      "exp": 1582060888,
      "_claim_names": {
         "groups": "src1"
      },
      "_claim_sources": {
         "src1": {
            "endpoint": "https://graph.windows.net/7200000-86f1-41af-91ab-2d7cd011db47/users/00000-f433-403e-b3aa-7d8406464625d7/getMemberObjects"
         }
      },
      "acr": "1",
      "aio": "AVQAq/8OAAAA7k1O1C2fRfeG604U9e6EzYcy52wb65Cx2OkaHIqDOkuyyr0IBa/YuaImaydaf/twVaeW/etbzzlKFNI4Q=",
      "amr": [
         "rsa",
         "mfa"
      ],
      "appid": "c44b4083-3bb0-00001-b47d-97400853cbdf3c",
      "appidacr": "2",
      "deviceid": "bfk817a1-3d981-4dddf82-8ade-2bddd2f5f8172ab",
      "family_name": "Sophia Owen",
      "given_name": "Sophia Owen (Fabrikam)",
      "ipaddr": "167.220.2.46",
      "name": "sophiaowen",
      "oid": "3d5053d9-f433-00000e-b3aa-7d84041625d7",
      "onprem_sid": "S-1-5-21-2497521184-1604012920-1887927527-21913475",
      "puid": "1003000000098FE48CE",
      "scp": "user_impersonation",
      "sub": "KGlhIodTx3XCVIWjJarRfJbsLX9JcdYYWDPkufGVij7_7k",
      "tid": "72f988bf-86f1-41af-91ab-2d7cd011db47",
      "unique_name": "SophiaOwen@fabrikam.com",
      "upn": "SophiaOwen@fabrikam.com",
      "uti": "TPJ7nNNMMZkOSx6_uVczUAA",
      "ver": "1.0"
   }
   ```

<a name="define-authorization-policy-portal"></a>

#### <a name="define-authorization-policy-in-azure-portal"></a>Définir la stratégie d’autorisation dans le portail Azure

Afin d’activer Azure AD OAuth pour votre application logique dans le portail Azure, procédez comme suit pour ajouter une ou plusieurs stratégies d’autorisation à votre application logique :

1. Dans le [portail Azure](https://portal.microsoft.com), recherchez et ouvrez votre application logique dans le concepteur d’applications logiques.

1. Dans le menu de l’application logique, sous **Paramètres**, sélectionnez **Autorisation**. Une fois le volet Autorisation ouvert, sélectionnez **Ajouter une stratégie**.

   ![Sélectionner « Autorisation » > « Ajouter une stratégie »](./media/logic-apps-securing-a-logic-app/add-azure-active-directory-authorization-policies.png)

1. Fournissez des informations sur la stratégie d’autorisation en spécifiant les [types de revendication](../active-directory/develop/developer-glossary.md#claim) et les valeurs que votre application logique attend dans le jeton d’accès présenté par chaque appel entrant au déclencheur de demande :

   ![Fournir des informations pour la stratégie d’autorisation](./media/logic-apps-securing-a-logic-app/set-up-authorization-policy.png)

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Nom de la stratégie** | Oui | Nom que vous voulez utiliser pour la stratégie d’autorisation |
   | **Revendications** | Oui | Types de revendications et valeurs que votre application logique accepte des appels entrants. La valeur de la revendication est limitée à 80 caractères. Voici les types de revendications disponibles : <p><p>- **Émetteur** <br>- **Audience** <br>- **Subject** <br>- **ID JWT** (ID JSON Web Token) <p><p>Au minimum, la liste **Revendications** doit inclure la revendication **Émetteur**, dont la valeur commence par `https://sts.windows.net/` ou `https://login.microsoftonline.com/` en tant qu’ID d’émetteur Azure AD. Pour plus d’informations sur ces types de revendication, consultez [Revendications dans les jetons de sécurité Azure AD](../active-directory/azuread-dev/v1-authentication-scenarios.md#claims-in-azure-ad-security-tokens). Vous pouvez également spécifier vos propres type et valeur de revendication. |
   |||

1. Pour ajouter une autre revendication, sélectionnez l’une des options suivantes :

   * Pour ajouter un autre type de revendication, sélectionnez **Ajouter une revendication standard**, sélectionnez le type de revendication, puis spécifiez sa valeur.

   * Pour ajouter votre propre revendication, sélectionnez **Ajouter une revendication personnalisée** et spécifiez la valeur de revendication personnalisée.

1. Pour ajouter une autre stratégie d’autorisation, sélectionnez **Ajouter une stratégie**. Répétez les étapes précédentes pour configurer la stratégie.

1. Quand vous avez terminé, sélectionnez **Enregistrer**.

1. Pour inclure l’en-tête `Authorization` du jeton d’accès dans les sorties du déclencheur basé sur une demande, consultez [Inclure l’en-tête « Authorization » dans les sorties du déclencheur de demande](#include-auth-header).

<a name="define-authorization-policy-template"></a>

#### <a name="define-authorization-policy-in-azure-resource-manager-template"></a>Définir la stratégie d’autorisation dans un modèle Resource Manager

Pour activer Azure AD OAuth dans le modèle Resource Manager en vue du déploiement de votre application logique, suivez les étapes et la syntaxe ci-dessous :

1. Dans la section `properties` de la [définition de ressource de votre application logique](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md#logic-app-resource-definition), ajoutez un objet `accessControl` , s’il n’en existe aucun, qui contient un objet `triggers`.

   Pour plus d’informations sur l’objet `accessControl`, consultez [Restreindre les plages d’adresses IP entrantes dans un modèle Azure Resource Manager](#restrict-inbound-ip-template) et [Référence du modèle de workflow Microsoft.Logic](/azure/templates/microsoft.logic/2019-05-01/workflows).

1. Dans l’objet `triggers`, ajoutez un objet `openAuthenticationPolicies` qui contient l’objet `policies` dans lequel vous définissez une ou plusieurs stratégies d’autorisation.

1. Fournissez un nom pour la stratégie d’autorisation, définissez le type de stratégie sur `AAD` et incluez un tableau `claims` dans lequel vous spécifiez un ou plusieurs types de revendications.

   Au minimum, le tableau `claims` doit inclure le type de revendication de l’émetteur dans lequel vous définissez la propriété `name` de la revendication sur `iss` et le paramètre `value`, qui doit commencer par `https://sts.windows.net/` ou `https://login.microsoftonline.com/`, en guise d’ID d’émetteur Azure AD. Pour plus d’informations sur ces types de revendication, consultez [Revendications dans les jetons de sécurité Azure AD](../active-directory/azuread-dev/v1-authentication-scenarios.md#claims-in-azure-ad-security-tokens). Vous pouvez également spécifier vos propres type et valeur de revendication.

1. Pour inclure l’en-tête `Authorization` du jeton d’accès dans les sorties du déclencheur basé sur une demande, consultez [Inclure l’en-tête « Authorization » dans les sorties du déclencheur de demande](#include-auth-header).

Voici la syntaxe à suivre :

```json
"resources": [
   {
      // Start logic app resource definition
      "properties": {
         "state": "<Enabled-or-Disabled>",
         "definition": {<workflow-definition>},
         "parameters": {<workflow-definition-parameter-values>},
         "accessControl": {
            "triggers": {
               "openAuthenticationPolicies": {
                  "policies": {
                     "<policy-name>": {
                        "type": "AAD",
                        "claims": [
                           {
                              "name": "<claim-name>",
                              "value": "<claim-value>"
                           }
                        ]
                     }
                  }
               }
            },
         },
      },
      "name": "[parameters('LogicAppName')]",
      "type": "Microsoft.Logic/workflows",
      "location": "[parameters('LogicAppLocation')]",
      "apiVersion": "2016-06-01",
      "dependsOn": [
      ]
   }
   // End logic app resource definition
],
```

<a name="include-auth-header"></a>

#### <a name="include-authorization-header-in-request-trigger-outputs"></a>Inclure l’en-tête « Authorization » dans les sorties du déclencheur de demande

Pour les applications logiques qui [activent Azure Active Directory Open Authentication (Azure AD OAuth)](#enable-oauth) pour autoriser les appels entrants à accéder aux déclencheurs basés sur des demandes, vous pouvez autoriser les sorties du déclencheur de demande ou du déclencheur de Webhook HTTP à inclure l’en-tête `Authorization` du jeton d’accès OAuth. Dans la définition JSON sous-jacente du déclencheur, ajoutez et définissez la propriété `operationOptions` sur `IncludeAuthorizationHeadersInOutputs`. Voici un exemple de déclencheur de demande :

```json
"triggers": {
   "manual": {
      "inputs": {
         "schema": {}
      },
      "kind": "Http",
      "type": "Request",
      "operationOptions": "IncludeAuthorizationHeadersInOutputs"
   }
}
```

Pour plus d’informations, consultez les rubriques suivantes :

* [Informations de référence sur le schéma des types d’actions et de déclencheurs - Déclencheur de demande](../logic-apps/logic-apps-workflow-actions-triggers.md#request-trigger)
* [Informations de référence sur le schéma des types d’actions et de déclencheurs - Déclencheur de Webhook HTTP](../logic-apps/logic-apps-workflow-actions-triggers.md#http-webhook-trigger)
* [Informations de référence sur le schéma des types d’actions et de déclencheurs - Options d’opérations](../logic-apps/logic-apps-workflow-actions-triggers.md#operation-options)

<a name="azure-api-management"></a>

### <a name="expose-your-logic-app-with-azure-api-management"></a>Exposer votre application logique avec Gestion des API Azure

Pour ajouter des [protocoles d’authentification](../active-directory/develop/authentication-vs-authorization.md) supplémentaires à votre application logique, pensez à utiliser le service [Gestion des API Azure](../api-management/api-management-key-concepts.md). Ce service vous permet d’exposer votre application logique sous la forme d’une API. En outre, il fournit des fonctionnalités avancées de supervision, de sécurité et d’application des stratégies, ainsi qu’une documentation riche sur tous les points de terminaison. Le service Gestion des API peut exposer un point de terminaison public ou privé pour votre application logique. Pour autoriser l’accès à ce point de terminaison, vous pouvez utiliser Azure AD OAuth, un [certificat client](#client-certificate-authentication) ou d’autres normes de sécurité pour l’autorisation de l’accès à ce point de terminaison. Lorsque Gestion des API reçoit une requête, le service envoie celle-ci à votre application logique, en effectuant également toutes les transformations ou restrictions nécessaires tout au long du processus. Pour permettre uniquement au service Gestion des API d’appeler votre application logique, vous pouvez [limiter les adresses IP entrantes de votre application logique](#restrict-inbound-ip).

<a name="restrict-inbound-ip"></a>

### <a name="restrict-inbound-ip-addresses"></a>Limiter les adresses IP entrantes

En plus de la signature d’accès partagé (SAP), vous pouvez aussi restreindre spécifiquement les clients qui sont autorisés à appeler votre application logique. Par exemple, si vous gérez votre point de terminaison de demande avec la [Gestion des API Azure](../api-management/api-management-key-concepts.md), vous pouvez restreindre votre application logique de sorte qu’elle n’accepte que les requêtes qui proviennent de l’adresse IP de [l’instance de service Gestion des API créée par vos soins](../api-management/get-started-create-service-instance.md).

> [!NOTE]
> Quelles que soient les adresses IP que vous spécifiez, vous pouvez toujours exécuter une application logique comportant un déclencheur basé sur une requête en utilisant la requête [API REST Logic Apps : Déclencheurs de workflow – Exécuter](/rest/api/logic/workflowtriggers/run) ou la Gestion des API. Cependant, ce scénario nécessite encore une [authentification](../active-directory/develop/authentication-vs-authorization.md) auprès de l’API REST Azure. Tous les événements s’affichent dans le journal d’audit Azure. Veillez à définir les stratégies de contrôle d’accès en conséquence.

<a name="restrict-inbound-ip-portal"></a>

#### <a name="restrict-inbound-ip-ranges-in-azure-portal"></a>Restreindre les plages d’adresses IP entrantes dans le Portail Azure

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Dans le menu de votre application logique, sous **Paramètres**, sélectionnez **Paramètres de flux de travail**.

1. Dans la section **Configuration du contrôle d’accès**, sous **Adresses IP entrantes autorisées**, choisissez le chemin de votre scénario :

   * Pour que votre application logique puisse uniquement être appelée en tant qu’application logique imbriquée à l’aide de l’[action Azure Logic Apps intégrée](../logic-apps/logic-apps-http-endpoint.md), sélectionnez **Uniquement les autres Logic Apps**, ce qui fonctionne *uniquement* quand vous utilisez l’action **Azure Logic Apps** pour appeler l’application logique imbriquée.
   
     Cette option écrit un tableau vide dans votre ressource d’application logique et nécessite que seuls les appels d’autres applications logiques parentes qui utilisent l’action **Azure Logic Apps** intégrée puissent déclencher l’application logique imbriquée.

   * Pour que votre application logique puisse uniquement être appelée qu’en tant qu’application imbriquée à l’aide de l’action HTTP, sélectionnez **Plages d’adresses IP spécifiques**, et *pas* **Uniquement les autres Logic Apps**. Quand la fenêtre **Plages d’adresses IP pour les déclencheurs** s’affiche, entrez les [adresses IP sortantes](../logic-apps/logic-apps-limits-and-config.md#outbound) de l’application logique parente. Une plage d’adresses IP valide utilise ces formats : *x.x.x.x/x* ou *x.x.x.x-x.x.x.x*.
   
     > [!NOTE]
     > Si vous utilisez l’option **Uniquement les autres Logic Apps** et l’action HTTP pour appeler votre application logique imbriquée, l’appel est bloqué et vous recevez une erreur « 401 Non autorisé ».
        
   * Pour les scénarios où vous voulez limiter les appels entrants à partir d’autres adresses IP, quand la fenêtre **Plages d’adresses IP pour les déclencheurs** s’affiche, spécifiez les plages d’adresses IP acceptées par le déclencheur. Une plage d’adresses IP valide utilise ces formats : *x.x.x.x/x* ou *x.x.x.x-x.x.x.x*.

1. Éventuellement, sous **Limitez les appels visant à obtenir des messages d’entrée et de sortie à partir de l’historique d’exécution aux plages d’adresses IP fournies**, vous pouvez spécifier les plages d’adresses IP pour les appels entrants qui peuvent accéder aux messages d’entrée et de sortie dans l’historique des exécutions.

<a name="restrict-inbound-ip-template"></a>

#### <a name="restrict-inbound-ip-ranges-in-azure-resource-manager-template"></a>Restreindre les plages d’adresses IP entrantes dans un modèle Azure Resource Manager

Si vous [automatisez le déploiement des applications logiques à l’aide de modèles Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), vous pouvez spécifier les plages d’adresses IP entrantes autorisées dans la définition de ressource de votre application logique en utilisant la section `accessControl`. Dans cette section, utilisez les sections `triggers`, `actions` et éventuellement `contents` comme il convient en incluant la section `allowedCallerIpAddresses` avec la propriété `addressRange` et définissez la valeur de la propriété sur la plage d’adresses IP autorisées au format *x.x.x.x/x* ou *x.x.x.x-x.x.x.x*.

* Si votre application logique imbriquée utilise l’option **Uniquement les autres Logic Apps**, qui autorise les appels entrants uniquement à partir d’autres applications logiques qui utilisent l’action Azure Logic Apps, définissez la propriété `addressRange` sur un tableau vide ( **[]** ).

* Si votre application logique imbriquée utilise l’option **Plages d’adresses IP spécifiques** pour d’autres appels entrants, comme d’autres applications logiques qui utilisent l’action HTTP, définissez la propriété `addressRange` sur la plage d’adresses IP autorisées.

Cet exemple montre une définition de ressource pour une application logique imbriquée qui autorise les appels entrants uniquement à partir d’applications logiques qui utilisent l’action Azure Logic Apps intégrée :

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {},
   "variables": {},
   "resources": [
      {
         "name": "[parameters('LogicAppName')]",
         "type": "Microsoft.Logic/workflows",
         "location": "[parameters('LogicAppLocation')]",
         "tags": {
            "displayName": "LogicApp"
         },
         "apiVersion": "2016-06-01",
         "properties": {
            "definition": {
               <workflow-definition>
            },
            "parameters": {
            },
            "accessControl": {
               "triggers": {
                  "allowedCallerIpAddresses": [
                     {
                        "addressRange": []
                     }
                  ]
               },
               "actions": {
                  "allowedCallerIpAddresses": [
                     {
                        "addressRange": []
                     }
                  ]
               }
            },
            "endpointsConfiguration": {}
         }
      }
   ],
   "outputs": {}
}
```

Cet exemple montre une définition de ressource pour une application logique imbriquée qui autorise les appels entrants à partir d’applications logiques qui utilisent l’action HTTP :

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {},
   "variables": {},
   "resources": [
      {
         "name": "[parameters('LogicAppName')]",
         "type": "Microsoft.Logic/workflows",
         "location": "[parameters('LogicAppLocation')]",
         "tags": {
            "displayName": "LogicApp"
         },
         "apiVersion": "2016-06-01",
         "properties": {
            "definition": {
               <workflow-definition>
            },
            "parameters": {
            },
            "accessControl": {
               "triggers": {
                  "allowedCallerIpAddresses": [
                     {
                        "addressRange": "192.168.12.0/23"
                     }
                  ]
               },
               "actions": {
                  "allowedCallerIpAddresses": [
                     {
                        "addressRange": "192.168.12.0/23"
                     }
                  ]
               }
            },
            "endpointsConfiguration": {}
         }
      }
   ],
   "outputs": {}
}
```

<a name="secure-operations"></a>

## <a name="access-to-logic-app-operations"></a>Accès aux opérations d’une application logique

Vous pouvez autoriser seulement certains utilisateurs ou groupes à exécuter des tâches, telles que la gestion, la modification ou l’affichage des applications logiques. Pour contrôler leurs autorisations, utilisez le [Contrôle d’accès en fonction du rôle (Azure RBAC)](../role-based-access-control/role-assignments-portal.md) afin d’attribuer des rôles personnalisés ou intégrés aux membres de votre abonnement Azure :

* [Contributeur d’application logique](../role-based-access-control/built-in-roles.md#logic-app-contributor) : Permet de gérer des applications logiques, mais pas d’en modifier l’accès.

* [Opérateur d’application logique](../role-based-access-control/built-in-roles.md#logic-app-operator) : Permet de lire, d’activer et de désactiver des applications logiques, mais pas de les modifier ni de les mettre à jour.

Pour empêcher la modification ou la suppression de votre application logique, vous pouvez utiliser le [Verrouillage de la ressource Azure](../azure-resource-manager/management/lock-resources.md). Grâce à cette fonctionnalité, vous pouvez empêcher d’autres utilisateurs de modifier ou de supprimer des ressources de production.

<a name="secure-run-history"></a>

## <a name="access-to-run-history-data"></a>Accès à l’historique des exécutions

Pendant l’exécution d’une application logique, toutes les données sont [chiffrées pendant le transit](../security/fundamentals/encryption-overview.md#encryption-of-data-in-transit) à l’aide de Transport Layer Security (TLS) et [au repos](../security/fundamentals/encryption-atrest.md). À la fin de l’exécution de votre application logique, vous pouvez afficher l’historique de l’exécution. Celui-ci comprend notamment les étapes exécutées, ainsi que l’état, la durée, les entrées et les sorties associés à chaque action. Ces informations détaillées fournissent des insights sur la façon dont votre application logique s’est exécutée et vous permettent de résoudre les problèmes qui peuvent survenir.

Lorsque vous affichez l’historique des exécutions de votre application logique, Logic Apps authentifie votre accès, et fournit des liens vers les entrées et les sorties des requêtes et des réponses pour chaque exécution. Toutefois, pour les actions qui gèrent des mots de passe, des secrets, des clés ou d’autres informations sensibles, vous devez empêcher les autres utilisateurs d’accéder à ces données. Par exemple, si votre application logique obtient un secret dans [Azure Key Vault](../key-vault/general/overview.md) en vue de l’utiliser lors de l’authentification d’une action HTTP, vous devez empêcher l’affichage de ce secret.

Pour contrôler l’accès aux entrées et aux sorties dans l’historique des exécutions de votre application logique, vous disposez des options suivantes :

* [Restreindre l’accès en fonction de la plage d’adresses IP](#restrict-ip).

  Cette option vous aide à sécuriser l’accès à l’historique des exécutions en fonction des requêtes provenant d’une plage d’adresses IP spécifique.

* [Sécuriser les données dans l’historique des exécutions à l’aide d’une obfuscation](#obfuscate).

  Vous pouvez sécuriser les entrées et/ou sorties de nombreux déclencheurs et actions dans l’historique d’exécution d’une application logique.

<a name="restrict-ip"></a>

### <a name="restrict-access-by-ip-address-range"></a>Restreindre l’accès en fonction de la plage d’adresses IP

Vous pouvez restreindre l’accès aux entrées et aux sorties de l’historique des exécutions de votre application logique afin que seules les requêtes provenant de certaines plages d’adresses IP soient autorisées à y accéder. Par exemple, pour empêcher quiconque d’accéder aux entrées et aux sorties, spécifiez une plage d’adresses IP comme celle-ci : `0.0.0.0-0.0.0.0`. Seul un utilisateur disposant d’autorisations d’administrateur peut supprimer cette restriction, permettant ainsi d’obtenir un accès « juste-à-temps » aux données de votre application logique. Vous pouvez spécifier les plages d’adresses IP non autorisées à l’aide du portail Azure ou d’un modèle Azure Resource Manager que vous utilisez pour le déploiement des applications logiques.

#### <a name="restrict-ip-ranges-in-azure-portal"></a>Restreindre les plages d’adresses IP dans le portail Azure

1. Dans le portail Azure, ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Dans le menu de votre application logique, sous **Paramètres**, sélectionnez **Paramètres de flux de travail**.

1. Sous **Configuration du contrôle d’accès** > **Adresses IP entrantes autorisées**, sélectionnez **Plages d’adresses IP spécifiques**.

1. Sous **Plages d’adresses IP pour le contenu**, spécifiez les plages d’adresses IP qui peuvent accéder au contenu issu des entrées et sorties.

   Une plage d’adresses IP valide utilise ces formats : *x.x.x.x/x* ou *x.x.x.x-x.x.x.x*

#### <a name="restrict-ip-ranges-in-azure-resource-manager-template"></a>Restreindre les plages d’adresses IP dans un modèle Azure Resource Manager

Si vous [automatisez le déploiement des applications logiques à l’aide de modèles Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), vous pouvez spécifier les plages d’adresses IP à l’aide de la section `accessControl` avec la section `contents` de la définition de ressource de votre application logique. Par exemple :

``` json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {},
   "variables": {},
   "resources": [
      {
         "name": "[parameters('LogicAppName')]",
         "type": "Microsoft.Logic/workflows",
         "location": "[parameters('LogicAppLocation')]",
         "tags": {
            "displayName": "LogicApp"
         },
         "apiVersion": "2016-06-01",
         "properties": {
            "definition": {<workflow-definition>},
            "parameters": {},
            "accessControl": {
               "contents": {
                  "allowedCallerIpAddresses": [
                     {
                        "addressRange": "192.168.12.0/23"
                     },
                     {
                        "addressRange": "2001:0db8::/64"
                     }
                  ]
               }
            }
         }
      }
   ],
   "outputs": {}
}
```

<a name="obfuscate"></a>

### <a name="secure-data-in-run-history-by-using-obfuscation"></a>Sécuriser les données dans l’historique des exécutions à l’aide d’une obfuscation

De nombreux déclencheurs et actions disposent de paramètres permettant sécuriser les entrées et/ou sorties dans l’historique d’exécution d’une application logique. Avant d’utiliser ces paramètres pour sécuriser ces données, [prenez connaissance des considérations suivantes](#obfuscation-considerations).

#### <a name="secure-inputs-and-outputs-in-the-designer"></a>Sécuriser les entrées et sorties dans le concepteur

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques.

   ![Ouvrir une application logique dans le concepteur d’application logique](./media/logic-apps-securing-a-logic-app/open-sample-logic-app-in-designer.png)

1. Pour le déclencheur ou l’action dont vous souhaitez sécuriser les données sensibles, sélectionnez le bouton de sélection ( **…** ), puis sélectionnez **Paramètres**.

   ![Ouvrir les paramètres d’action ou de déclencheur](./media/logic-apps-securing-a-logic-app/open-action-trigger-settings.png)

1. Activez **Entrées sécurisées**, **Sorties sécurisées**, ou les deux. Quand vous avez terminé, cliquez sur **Terminé**.

   ![Activer « Entrées sécurisées » ou « Sorties sécurisées »](./media/logic-apps-securing-a-logic-app/turn-on-secure-inputs-outputs.png)

   L’action ou le déclencheur affiche désormais une icône de verrou dans la barre de titre.

   ![La barre de titre de l’action ou du déclencheur affiche l’icône de verrou](./media/logic-apps-securing-a-logic-app/lock-icon-action-trigger-title-bar.png)

   Les jetons qui représentent les sorties sécurisées d’actions précédentes affichent également un verrou. Par exemple, lorsque vous sélectionnez une sortie de ce type dans la liste de contenu dynamique en vue de l’utiliser dans une action, ce jeton affiche une icône de verrou.

   ![Sélectionner un jeton pour une sortie sécurisée](./media/logic-apps-securing-a-logic-app/select-secured-token.png)

1. Après l’exécution de l’application logique, vous pouvez voir l’historique de cette exécution.

   1. Dans le volet **Vue d’ensemble** de l’application logique, sélectionnez l’exécution que vous souhaitez voir.

   1. Dans le volet **Exécution d’application logique**, développez les actions que vous souhaitez examiner.

      Si vous avez choisi de rendre les entrées et les sorties secrètes, ces valeurs sont désormais masquées.

      ![Entrées et sorties masquées dans l’historique des exécutions](./media/logic-apps-securing-a-logic-app/hidden-data-run-history.png)

<a name="secure-data-code-view"></a>

#### <a name="secure-inputs-and-outputs-in-code-view"></a>Sécuriser les entrées et sorties en mode Code

Dans la définition de déclencheur ou d’action sous-jacente, ajoutez ou mettez à jour le tableau `runtimeConfiguration.secureData.properties` avec une des deux valeurs ou les deux :

* `"inputs"`: Sécurise les entrées dans l’historique des exécutions.
* `"outputs"`: Sécurise les sorties dans l’historique des exécutions.

Voici certains [aspects à prendre en considération](#obfuscation-considerations) quand vous utilisez ces paramètres pour sécuriser ces données.

```json
"<trigger-or-action-name>": {
   "type": "<trigger-or-action-type>",
   "inputs": {
      <trigger-or-action-inputs>
   },
   "runtimeConfiguration": {
      "secureData": {
         "properties": [
            "inputs",
            "outputs"
         ]
      }
   },
   <other-attributes>
}
```

<a name="obfuscation-considerations"></a>

#### <a name="considerations-when-securing-inputs-and-outputs"></a>Considérations relatives à la sécurisation des entrées et des sorties

* Quand vous rendez secrètes les entrées ou les sorties d’un déclencheur ou d’une action, Logic Apps n’envoie pas les données sécurisées à Azure Log Analytics. De plus, vous ne pouvez pas ajouter de [propriétés suivies](../logic-apps/monitor-logic-apps-log-analytics.md#extend-data) à ce déclencheur ou à cette action à des fins de supervision.

* L’[API Logic Apps qui permet de gérer l’historique des workflows](/rest/api/logic/) ne retourne pas de sorties sécurisées.

* Pour sécuriser les sorties d’une action qui rend les entrées secrètes ou qui rend explicitement les sorties secrètes, activez manuellement **Sorties sécurisées** dans cette action.

* Veillez à activer **Entrées sécurisées** ou **Sorties sécurisées** dans les actions en aval, où l’historique des exécutions doit rendre ces données secrètes.

  **Paramètre Sorties sécurisées**

  Lorsque vous activez manuellement les **sorties sécurisées** dans un déclencheur ou dans une action, Logic Apps masque ces sorties dans l’historique des exécutions. Si une action en aval utilise explicitement ces sorties sécurisées comme des entrées, Logic Apps masque les entrées de cette action dans l’historique des exécutions, mais *il n’active pas* le paramètre **Entrées sécurisées** de l’action.

  ![Sorties sécurisées en tant qu’entrées et impact en aval sur la plupart des actions](./media/logic-apps-securing-a-logic-app/secure-outputs-as-inputs-flow.png)

  Les actions Composer, Analyser JSON et Réponse comportent uniquement le paramètre **Entrées sécurisées**. Lorsqu’il est activé, ce paramètre masque également les sorties de ces actions. Si ces actions utilisent explicitement les sorties sécurisées en amont comme des entrées, Logic Apps masque les entrées et les sorties de ces actions, mais *il n’active pas* le paramètre **Entrées sécurisées** de ces actions. Si une action en aval utilise explicitement les sorties masquées des actions Composer, Analyser JSON ou Réponse comme des entrées, Logic Apps *ne masque pas les entrées ou les sorties de cette action en aval*.

  ![Sorties sécurisées en tant qu’entrées et impact en aval sur certaines actions](./media/logic-apps-securing-a-logic-app/secure-outputs-as-inputs-flow-special.png)

  **Paramètre Entrées sécurisées**

  Lorsque vous activez manuellement les **entrées sécurisées** dans un déclencheur ou dans une action, Logic Apps masque ces entrées dans l’historique des exécutions. Si une action en aval utilise explicitement les sorties visibles de ce déclencheur ou de cette action comme entrées, Logic Apps masque les entrées de cette action en aval dans l’historique des exécutions, mais *il n’active pas* les **entrées sécurisées** de cette action, et ne masque pas les sorties de cette action.

  ![Entrées sécurisées et impact en aval sur la plupart des actions](./media/logic-apps-securing-a-logic-app/secure-inputs-impact-on-downstream.png)

  Si les actions Composer, Analyser JSON et Réponse utilisent explicitement les sorties visibles du déclencheur ou de l’action qui a des entrées sécurisées, Logic Apps masque les entrées et les sorties de ces actions, mais *il n’active pas* le paramètre **Entrées sécurisées** de ces actions. Si une action en aval utilise explicitement les sorties masquées des actions Composer, Analyser JSON ou Réponse comme des entrées, Logic Apps *ne masque pas les entrées ou les sorties de cette action en aval*.

  ![Entrées sécurisées et impact en aval sur certaines actions](./media/logic-apps-securing-a-logic-app/secure-inputs-flow-special.png)

<a name="secure-action-parameters"></a>

## <a name="access-to-parameter-inputs"></a>Accès aux entrées de paramètres

Si vous effectuez des déploiements dans différents environnements, vous pouvez paramétrer les valeurs de votre définition de workflow qui varient selon l’environnement. De cette façon, vous pouvez éviter les données codées en dur en utilisant un [modèle Azure Resource Manager](../azure-resource-manager/templates/overview.md) pour déployer votre application logique, protéger les données sensibles en définissant des paramètres sécurisés et transmettre ces données sous forme d’entrées distinctes par le biais des [paramètres du modèle](../azure-resource-manager/templates/template-parameters.md) à l’aide d’un [fichier de paramètres](../azure-resource-manager/templates/parameter-files.md).

Par exemple, si vous authentifiez des actions HTTP avec [Azure Active Directory Open Authentication](#azure-active-directory-oauth-authentication) (Azure AD OAuth), vous pouvez définir et rendre secrets les paramètres qui acceptent l’ID client et la clé secrète client utilisés pour l’authentification. Si vous souhaitez configurer ces paramètres dans votre application logique, utilisez la section `parameters` située dans la définition de workflow de votre application logique et le modèle Resource Manager pour le déploiement. Pour sécuriser les valeurs de paramètre que vous ne voulez pas présenter lors de la modification de votre application logique ou de la consultation de l’historique des exécutions, configurez les paramètres avec le type `securestring` ou `secureobject` et utilisez un encodage si nécessaire. Les paramètres de ce type ne sont pas retournés avec la définition de la ressource, et ne sont pas accessibles lors de l’affichage de la ressource après le déploiement. Pour accéder à ces valeurs de paramètre pendant l’exécution, utilisez l’expression `@parameters('<parameter-name>')` située dans la définition de votre workflow. Cette expression est évaluée uniquement au moment de l’exécution et est décrite par le [langage de définition du workflow](../logic-apps/logic-apps-workflow-definition-language.md).

> [!NOTE]
> Si vous utilisez un paramètre dans l’en-tête ou le corps d’une requête, il peut être visible lorsque vous consultez l’historique des exécutions de l’application logique et la requête HTTP sortante. Veillez également à définir vos stratégies d’accès au contenu en conséquence.
> Vous pouvez également utiliser l’[obfuscation](#obfuscate) pour masquer les entrées et les sorties dans votre historique des exécutions. Les en-têtes d’autorisation ne sont jamais visibles par le biais d’entrées ou de sorties. Si un secret est utilisé ici, il n’est pas récupérable.

Pour en savoir plus, consultez les sections suivantes de la présente rubrique :

* [Sécuriser des paramètres dans les définitions de workflow](#secure-parameters-workflow)
* [Sécuriser les données dans l’historique des exécutions à l’aide d’une obfuscation](#obfuscate)

Si vous [automatisez le déploiement d’applications logiques à l’aide de modèles Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), vous pouvez définir des [paramètres de modèles](../azure-resource-manager/templates/template-parameters.md) sécurisés, qui sont évalués au moment du déploiement, à l’aide des types `securestring` et `secureobject`. Pour définir des paramètres de modèle, utilisez la section `parameters` de niveau supérieur de votre modèle, qui est différente de la section `parameters` de votre définition de workflow. Pour fournir les valeurs des paramètres de modèle, utilisez un autre [fichier de paramètres](../azure-resource-manager/templates/parameter-files.md).

Par exemple, si vous utilisez des secrets, vous pouvez définir et utiliser des paramètres de modèles sécurisés qui récupèrent ces secrets à partir d’[Azure Key Vault](../key-vault/general/overview.md) lors du déploiement. Vous pouvez ensuite référencer le coffre de clés et le secret dans votre fichier de paramètres. Pour plus d’informations, consultez les rubriques suivantes :

* [Transmettre des valeurs sensibles au déploiement à l’aide d’Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md)
* [Sécuriser des paramètres dans des modèles Azure Resource Manager](#secure-parameters-deployment-template) plus loin dans cette rubrique

<a name="secure-parameters-workflow"></a>

### <a name="secure-parameters-in-workflow-definitions"></a>Sécuriser des paramètres dans les définitions de workflow

Pour protéger des informations sensibles dans la définition de workflow de votre application logique, utilisez des paramètres sécurisés afin que ces informations ne soient pas visibles après l’enregistrement de votre application logique. Par exemple, supposons que vous avez une action HTTP qui nécessite une authentification de base, c’est-à-dire, un nom d’utilisateur et un mot de passe. Dans la définition de workflow, la section `parameters` définit les paramètres `basicAuthPasswordParam` et `basicAuthUsernameParam` à l’aide du type `securestring`. La définition de l’action référence ensuite ces paramètres dans la section `authentication`.

```json
"definition": {
   "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
   "actions": {
      "HTTP": {
         "type": "Http",
         "inputs": {
            "method": "GET",
            "uri": "https://www.microsoft.com",
            "authentication": {
               "type": "Basic",
               "username": "@parameters('basicAuthUsernameParam')",
               "password": "@parameters('basicAuthPasswordParam')"
            }
         },
         "runAfter": {}
      }
   },
   "parameters": {
      "basicAuthPasswordParam": {
         "type": "securestring"
      },
      "basicAuthUsernameParam": {
         "type": "securestring"
      }
   },
   "triggers": {
      "manual": {
         "type": "Request",
         "kind": "Http",
         "inputs": {
            "schema": {}
         }
      }
   },
   "contentVersion": "1.0.0.0",
   "outputs": {}
}
```

<a name="secure-parameters-deployment-template"></a>

### <a name="secure-parameters-in-azure-resource-manager-templates"></a>Sécuriser des paramètres dans des modèles Azure Resource Manager

Un [modèle Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md) d’une application logique comporte plusieurs sections `parameters`. Pour protéger les mots de passe, les clés, les secrets et d’autres informations sensibles, configurez des paramètres sécurisés au niveau du modèle et au niveau de la définition de workflow à l’aide du type `securestring` ou `secureobject`. Vous pouvez ensuite stocker ces valeurs dans [Azure Key Vault](../key-vault/general/overview.md) et utiliser le [fichier de paramètres](../azure-resource-manager/templates/parameter-files.md) pour référencer le coffre de clés et le secret. Votre modèle récupère ensuite ces informations au moment du déploiement. Pour plus d’informations, consultez [Transmettre des valeurs sensibles au déploiement à l’aide d’Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md).

Voici plus d’informations sur les sections `parameters` :

* En haut du modèle, une section `parameters` définit les paramètres à l’aide des valeurs utilisées par le modèle au moment du *déploiement*. Par exemple, ces valeurs peuvent inclure des chaînes de connexion pour un environnement de déploiement spécifique. Vous pouvez ensuite stocker ces valeurs dans un autre [fichier de paramètres](../azure-resource-manager/templates/parameter-files.md), ce qui facilite la modification de ces valeurs.

* Dans la définition de ressource de votre application logique, mais en dehors de la définition de workflow, une section `parameters` spécifie les valeurs des paramètres de votre définition de workflow. Dans cette section, vous pouvez affecter ces valeurs à l’aide d’expressions de modèle qui référencent les paramètres de votre modèle. Ces expressions sont évaluées au moment du déploiement.

* Dans votre définition de workflow, une section `parameters` définit les paramètres utilisés par votre application logique au moment de l’exécution. Vous pouvez ensuite référencer ces paramètres dans le workflow de votre application logique à l’aide d’expressions de définition de workflow, qui sont évaluées au moment de l’exécution.

Cet exemple de modèle avec plusieurs définitions de paramètres sécurisés qui utilisent le type `securestring` :

| Nom du paramètre | Description |
|----------------|-------------|
| `TemplatePasswordParam` | Paramètre de modèle acceptant un mot de passe qui est ensuite passé au paramètre `basicAuthPasswordParam` de la définition de workflow. |
| `TemplateUsernameParam` | Paramètre de modèle acceptant un nom d’utilisateur qui est ensuite passé au paramètre `basicAuthUserNameParam` de la définition de workflow. |
| `basicAuthPasswordParam` | Paramètre de définition de workflow qui accepte le mot de passe pour l’authentification de base dans une action HTTP |
| `basicAuthUserNameParam` | Paramètre de définition de workflow qui accepte le nom d’utilisateur pour l’authentification de base dans une action HTTP |
|||

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {
      "LogicAppName": {
         "type": "string",
         "minLength": 1,
         "maxLength": 80,
         "metadata": {
            "description": "Name of the Logic App."
         }
      },
      "TemplatePasswordParam": {
         "type": "securestring"
      },
      "TemplateUsernameParam": {
         "type": "securestring"
      },
      "LogicAppLocation": {
         "type": "string",
         "defaultValue": "[resourceGroup().location]",
         "allowedValues": [
            "[resourceGroup().location]",
            "eastasia",
            "southeastasia",
            "centralus",
            "eastus",
            "eastus2",
            "westus",
            "northcentralus",
            "southcentralus",
            "northeurope",
            "westeurope",
            "japanwest",
            "japaneast",
            "brazilsouth",
            "australiaeast",
            "australiasoutheast",
            "southindia",
            "centralindia",
            "westindia",
            "canadacentral",
            "canadaeast",
            "uksouth",
            "ukwest",
            "westcentralus",
            "westus2"
         ],
         "metadata": {
            "description": "Location of the Logic App."
         }
      }
   },
   "variables": {},
   "resources": [
      {
         "name": "[parameters('LogicAppName')]",
         "type": "Microsoft.Logic/workflows",
         "location": "[parameters('LogicAppLocation')]",
         "tags": {
            "displayName": "LogicApp"
         },
         "apiVersion": "2016-06-01",
         "properties": {
            "definition": {
               "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
               "actions": {
                  "HTTP": {
                     "type": "Http",
                     "inputs": {
                        "method": "GET",
                        "uri": "https://www.microsoft.com",
                        "authentication": {
                           "type": "Basic",
                           "username": "@parameters('basicAuthUsernameParam')",
                           "password": "@parameters('basicAuthPasswordParam')"
                        }
                     },
                  "runAfter": {}
                  }
               },
               "parameters": {
                  "basicAuthPasswordParam": {
                     "type": "securestring"
                  },
                  "basicAuthUsernameParam": {
                     "type": "securestring"
                  }
               },
               "triggers": {
                  "manual": {
                     "type": "Request",
                     "kind": "Http",
                     "inputs": {
                        "schema": {}
                     }
                  }
               },
               "contentVersion": "1.0.0.0",
               "outputs": {}
            },
            "parameters": {
               "basicAuthPasswordParam": {
                  "value": "[parameters('TemplatePasswordParam')]"
               },
               "basicAuthUsernameParam": {
                  "value": "[parameters('TemplateUsernameParam')]"
               }
            }
         }
      }
   ],
   "outputs": {}
}
```

<a name="secure-outbound-requests"></a>

## <a name="access-for-outbound-calls-to-other-services-and-systems"></a>Accès pour les appels sortants à d’autres services et systèmes

En fonction de la capacité du point de terminaison cible, les appels sortants envoyés par [le déclencheur ou l’action HTTP](../connectors/connectors-native-http.md) prennent en charge le chiffrement et sont sécurisés avec le [protocole TLS (Transport Layer Security) 1.0, 1.1 ou 1.2](https://en.wikipedia.org/wiki/Transport_Layer_Security), anciennement appelé Secure Sockets Layer (SSL). Logic Apps négocie avec le point de terminaison cible en utilisant la version la plus élevée possible prise en charge. Par exemple, si le point de terminaison cible prend en charge la version 1.2, le déclencheur ou l’action HTTP utilisent d’abord la version 1.2. Sinon, le connecteur utilise la version prise en charge juste inférieure.

Voici des informations sur les certificats auto-signés TLS/SSL :

* Pour les applications logiques situées dans l’environnement Azure multilocataire global, le connecteur HTTP n’autorise pas les certificats TLS/SSL auto-signés. Si votre application logique émet un appel HTTP vers un serveur et présente un certificat auto-signé TLS/SSL, l’appel HTTP échouera avec une erreur `TrustFailure`.

* Pour les applications logiques situées dans un [environnement ISE](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), le connecteur HTTP autorise les certificats auto-signés pour les négociations TLS/SSL. Toutefois, vous devez d’abord [activer la prise en charge des certificats auto-signés](../logic-apps/create-integration-service-environment-rest-api.md#request-body) pour un environnement ISE existant ou un nouvel ISE à l’aide de l’API REST Logic Apps, puis installer le certificat public à l’emplacement `TrustedRoot`.

Voici d’autres façons de sécuriser les points de terminaison qui gèrent les appels envoyés à partir de votre application logique :

* [Ajoutez l’authentification aux requêtes sortantes](#add-authentication-outbound).

  Lorsque vous utilisez le déclencheur ou l’action HTTP pour envoyer des appels sortants, vous pouvez ajouter l’authentification à la demande envoyée par votre application logique. Par exemple, vous pouvez sélectionner les types d’authentification suivants :

  * [Authentification de base](#basic-authentication)

  * [Authentification par certificat client](#client-certificate-authentication)

  * [Authentification OAuth Active Directory](#azure-active-directory-oauth-authentication)

  * [Authentification d’une identité managée](#managed-identity-authentication)

* Restreignez l’accès des adresses IP d’application logique.

  Tous les appels de points de terminaison émis par des applications logiques proviennent d’adresses IP désignées en fonction des régions de vos applications logiques. Vous pouvez ajouter le filtrage afin d’accepter les requêtes provenant uniquement de ces adresses IP. Pour obtenir ces adresses IP, consultez [Limites et configuration d’Azure Logic Apps](logic-apps-limits-and-config.md#configuration).

* Améliorez la sécurité des connexions aux systèmes locaux.

  Azure Logic Apps permet l’intégration avec ces services pour fournir une communication locale plus sécurisée et plus fiable.

  * Passerelle de données locale

    De nombreux connecteurs managés dans Azure Logic Apps facilitent les connexions sécurisées aux systèmes locaux, par exemple File System, SQL, SharePoint et DB2. La passerelle envoie les données des sources locales sur des canaux chiffrés via Azure Service Bus. L’ensemble du trafic est généré sous forme de trafic sortant sécurisé en provenance de l’agent de passerelle. Découvrez comment [la passerelle de données sur site fonctionne](logic-apps-gateway-install.md#gateway-cloud-service).

  * Connectez-vous via la Gestion des API Azure.

    La [Gestion des API Azure](../api-management/api-management-key-concepts.md) fournit des options de connexion locale, par exemple, un réseau privé virtuel de site à site et une intégration [ExpressRoute](../expressroute/expressroute-introduction.md) pour un proxy et une communication sécurisés vers les systèmes locaux. Si vous disposez d’une API qui permet d’accéder à votre système local et que vous l’avez exposée en créant une [instance de service Gestion des API](../api-management/get-started-create-service-instance.md), vous pouvez appeler cette API dans le flux de travail de votre application logique en sélectionnant l’action ou le déclencheur Gestion des API intégré dans le Concepteur d’applications logiques.

    > [!NOTE]
    > Le connecteur affiche uniquement les services Gestion des API pour lesquels vous disposez d’autorisations d’affichage et de connexion, et non les services Gestion des API basés sur la consommation.

    1. Dans le Concepteur d’applications logiques, entrez `api management` dans la zone de recherche. Choisissez l’étape appropriée, selon que vous ajoutez un déclencheur ou une action :<p>

       * Si vous ajoutez un déclencheur, ce qui est toujours le cas à la première étape d’un flux de travail, sélectionnez **Choisir un déclencheur Gestion des API Azure**.

       * Si vous ajoutez une action, sélectionnez **Choisir une action Gestion des API Azure**.

       Cet exemple ajoute un déclencheur :

       ![Ajout d’un déclencheur Gestion des API Azure](./media/logic-apps-securing-a-logic-app/select-api-management.png)

    1. Sélectionnez l’instance de service Gestion des API que vous avez créée.

       ![Sélection de l’instance de service Gestion des API](./media/logic-apps-securing-a-logic-app/select-api-management-service-instance.png)

    1. Sélectionnez l’appel d’API à utiliser.

       ![Sélection d’une API existante](./media/logic-apps-securing-a-logic-app/select-api.png)

<a name="add-authentication-outbound"></a>

### <a name="add-authentication-to-outbound-calls"></a>Ajouter l’authentification aux appels sortants

Les points de terminaison HTTP et HTTPS prennent en charge différents types d’authentification. Il est possible, sur certains déclencheurs et actions permettant d’envoyer des appels sortants ou des demandes à ces points de terminaison, de spécifier un type d’authentification. Dans le Concepteur d’applications logiques, les déclencheurs et les actions qui prennent en charge le choix du type d’authentification disposent d’une propriété **Authentification**. Toutefois, celle-ci n’apparaît pas toujours par défaut. Dans ce cas, ouvrez la liste **Ajouter un nouveau paramètre** dans le déclencheur ou l’action, puis sélectionnez **Authentification**.

> [!IMPORTANT]
> Pour protéger les informations sensibles gérées par votre application logique, utilisez des paramètres sécurisés et encodez les données si nécessaire.
> Pour plus d’informations sur l’utilisation et la sécurisation des paramètres, consultez [Accès aux entrées de paramètres](#secure-action-parameters).

Ce tableau identifie les types d’authentification disponibles sur les déclencheurs et les actions offrant la possibilité de sélectionner un type d’authentification :

| Type d'authentification | Déclencheurs et actions pris en charge |
|---------------------|--------------------------------|
| [De base](#basic-authentication) | Gestion des API Azure, Azure App Service, HTTP, HTTP + Swagger, Webhook HTTP |
| [Certificat client](#client-certificate-authentication) | Gestion des API Azure, Azure App Service, HTTP, HTTP + Swagger, Webhook HTTP |
| [OAuth Active Directory](#azure-active-directory-oauth-authentication) | Gestion des API Azure, Azure App Service, Azure Functions, HTTP, HTTP + Swagger, Webhook HTTP |
| [Brut](#raw-authentication) | Gestion des API Azure, Azure App Service, Azure Functions, HTTP, HTTP + Swagger, Webhook HTTP |
| [Identité gérée](#managed-identity-authentication) | Gestion des API Azure, Azure App Service, Azure Functions, HTTP, Webhook HTTP |
|||

<a name="basic-authentication"></a>

### <a name="basic-authentication"></a>Authentification de base

Si l’option [De base](../active-directory-b2c/secure-rest-api.md) est disponible, spécifiez les valeurs de propriété suivantes :

| Propriété (concepteur) | Propriété (JSON) | Obligatoire | Valeur | Description |
|---------------------|-----------------|----------|-------|-------------|
| **Authentification** | `type` | Oui | De base | Type d’authentification à utiliser |
| **Nom d’utilisateur** | `username` | Oui | <*user-name*>| Nom d’utilisateur permettant d’authentifier l’accès au point de terminaison de service cible |
| **Mot de passe** | `password` | Oui | <*password*> | Mot de passe permettant d’authentifier l’accès au point de terminaison de service cible |
||||||

Lorsque vous utilisez des [paramètres sécurisés](#secure-action-parameters) pour traiter et sécuriser des informations sensibles, par exemple dans un [modèle Azure Resource Manager pour l’automatisation du déploiement](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), vous pouvez utiliser des expressions pour accéder à ces valeurs de paramètre au moment de l’exécution. Cet exemple de définition d’action HTTP spécifie l’authentification `type` en tant que `Basic` et utilise la fonction [parameters()](../logic-apps/workflow-definition-language-functions-reference.md#parameters) pour récupérer les valeurs des paramètres :

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "@parameters('endpointUrlParam')",
      "authentication": {
         "type": "Basic",
         "username": "@parameters('userNameParam')",
         "password": "@parameters('passwordParam')"
      }
  },
  "runAfter": {}
}
```

<a name="client-certificate-authentication"></a>

### <a name="client-certificate-authentication"></a>Authentification par certificat client

Si l’option [Certificat client](../active-directory/authentication/active-directory-certificate-based-authentication-get-started.md) est disponible, spécifiez ces valeurs de propriété :

| Propriété (concepteur) | Propriété (JSON) | Obligatoire | Valeur | Description |
|---------------------|-----------------|----------|-------|-------------|
| **Authentification** | `type` | Oui | **Certificat client** <br>or <br>`ClientCertificate` | Type d’authentification à utiliser. Vous pouvez gérer les certificats avec la [Gestion des API Azure](../api-management/api-management-howto-mutual-certificates.md). <p></p>**Remarque** : Les connecteurs personnalisés ne prennent pas en charge l’authentification par certificat pour les appels entrants ni sortants. |
| **Pfx** | `pfx` | Oui | <*encoded-pfx-file-content*> | Contenu encodé en base64 à partir d’un fichier Personal Information Exchange (PFX) <p><p>Pour convertir le fichier PFX au format encodé en base64, vous pouvez utiliser PowerShell en procédant comme suit : <p>1. Enregistrez le contenu du certificat dans une variable : <p>   `$pfx_cert = get-content 'c:\certificate.pfx' -Encoding Byte` <p>2. Convertissez le contenu du certificat à l’aide de la fonction `ToBase64String()` et enregistrez ce contenu dans un fichier texte : <p>   `[System.Convert]::ToBase64String($pfx_cert) | Out-File 'pfx-encoded-bytes.txt'` |
| **Mot de passe** | `password`| Non | <*password-for-pfx-file*> | Mot de passe pour accéder au fichier PFX |
|||||

Lorsque vous utilisez des [paramètres sécurisés](#secure-action-parameters) pour traiter et sécuriser des informations sensibles, par exemple dans un [modèle Azure Resource Manager pour l’automatisation du déploiement](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), vous pouvez utiliser des expressions pour accéder à ces valeurs de paramètre au moment de l’exécution. Cet exemple de définition d’action HTTP spécifie l’authentification `type` en tant que `ClientCertificate` et utilise la fonction [parameters()](../logic-apps/workflow-definition-language-functions-reference.md#parameters) pour récupérer les valeurs des paramètres :

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "@parameters('endpointUrlParam')",
      "authentication": {
         "type": "ClientCertificate",
         "pfx": "@parameters('pfxParam')",
         "password": "@parameters('passwordParam')"
      }
   },
   "runAfter": {}
}
```

Pour plus d’informations sur la sécurisation des services à l’aide de l’authentification par certificat client, consultez les rubriques suivantes :

* [Améliorer la sécurité des API à l’aide de l’authentification par certificat client dans Gestion des API Azure](../api-management/api-management-howto-mutual-certificates-for-clients.md)
* [Améliorer la sécurité des services back-end à l’aide de l’authentification par certificat client dans Gestion des API Azure](../api-management/api-management-howto-mutual-certificates.md)
* [Améliorer la sécurité de votre service RESTful à l’aide de certificats clients](../active-directory-b2c/secure-rest-api.md)
* [Informations d’identification de certificat pour l’authentification d’application](../active-directory/develop/active-directory-certificate-credentials.md)
* [Utiliser un certificat TLS/SSL dans votre code dans Azure App Service](../app-service/configure-ssl-certificate-in-code.md)

<a name="azure-active-directory-oauth-authentication"></a>

### <a name="azure-active-directory-open-authentication"></a>Azure Active Directory Open Authentication

Sur les déclencheurs de requête, vous pouvez utiliser [Azure Active Directory Open Authentication](../active-directory/develop/index.yml) (Azure AD OAuth) pour authentifier les appels entrants après avoir [configuré des stratégies d’autorisation Azure AD](#enable-oauth) pour votre application logique. Pour tous les autres déclencheurs et actions qui proposent la sélection du type d’authentification **Active Directory OAuth**, spécifiez ces valeurs de propriété :

| Propriété (concepteur) | Propriété (JSON) | Obligatoire | Valeur | Description |
|---------------------|-----------------|----------|-------|-------------|
| **Authentification** | `type` | Oui | **OAuth Active Directory** <br>or <br>`ActiveDirectoryOAuth` | Type d’authentification à utiliser. Logic Apps suit actuellement le [protocole OAuth 2.0](../active-directory/develop/v2-overview.md). |
| **Authority** | `authority` | Non | <*URL de l’autorité émettrice du jeton*> | URL de l’autorité qui fournit le jeton d’accès. Par défaut, cette valeur est définie sur `https://login.windows.net`. |
| **Locataire** | `tenant` | Oui | <*ID de locataire*> | Identificateur du locataire Azure AD |
| **Public ciblé** | `audience` | Oui | <*ressource à autoriser*> | Ressource à utiliser pour l’autorisation, par exemple, `https://management.core.windows.net/` |
| **ID client** | `clientId` | Oui | <*ID client*> | ID client pour l’application demandant l’autorisation |
| **Type d’informations d’identification** | `credentialType` | Oui | Certificat <br>or <br>Secret | Type d’informations d’identification que le client utilise pour la demande d’autorisation. Ces propriété et valeur n’apparaissent pas dans la définition sous-jacente de votre application logique, mais elles déterminent les propriétés affichées pour le type d’informations d’identification sélectionné. |
| **Secret** | `secret` | Oui, uniquement pour le type d’informations d’identification « Secret » | <*client-secret*> | Clé secrète client permettant de demander une autorisation |
| **Pfx** | `pfx` | Oui, mais uniquement pour le type d’informations d’identification « Certificat » | <*encoded-pfx-file-content*> | Contenu encodé en base64 à partir d’un fichier Personal Information Exchange (PFX) |
| **Mot de passe** | `password` | Oui, mais uniquement pour le type d’informations d’identification « Certificat » | <*password-for-pfx-file*> | Mot de passe pour accéder au fichier PFX |
|||||

Lorsque vous utilisez des [paramètres sécurisés](#secure-action-parameters) pour traiter et sécuriser des informations sensibles, par exemple dans un [modèle Azure Resource Manager pour l’automatisation du déploiement](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), vous pouvez utiliser des expressions pour accéder à ces valeurs de paramètre au moment de l’exécution. Cet exemple de définition d’action HTTP spécifie l’authentification `type` en tant que `ActiveDirectoryOAuth`, le type d’informations d’identification en tant que `Secret` et utilise la [fonction Parameters()](../logic-apps/workflow-definition-language-functions-reference.md#parameters) pour récupérer les valeurs des paramètres :

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "@parameters('endpointUrlParam')",
      "authentication": {
         "type": "ActiveDirectoryOAuth",
         "tenant": "@parameters('tenantIdParam')",
         "audience": "https://management.core.windows.net/",
         "clientId": "@parameters('clientIdParam')",
         "credentialType": "Secret",
         "secret": "@parameters('secretParam')"
     }
   },
   "runAfter": {}
}
```

<a name="raw-authentication"></a>

### <a name="raw-authentication"></a>Authentification brute

Si l’option **Brut** est disponible, vous pouvez utiliser ce type d’authentification lorsque vous devez utiliser des [schémas d’authentification](https://iana.org/assignments/http-authschemes/http-authschemes.xhtml) qui ne suivent pas le [protocole OAuth 2.0](https://oauth.net/2/). Avec ce type, vous créez manuellement la valeur d’en-tête d’autorisation que vous envoyez avec la demande sortante, puis vous spécifiez cette valeur d’en-tête dans votre déclencheur ou action.

Par exemple, voici un exemple d’en-tête pour une requête HTTPS qui suit le protocole [OAuth 1.0](https://tools.ietf.org/html/rfc5849) :

```text
Authorization: OAuth realm="Photos",
   oauth_consumer_key="dpf43f3p2l4k3l03",
   oauth_signature_method="HMAC-SHA1",
   oauth_timestamp="137131200",
   oauth_nonce="wIjqoS",
   oauth_callback="http%3A%2F%2Fprinter.example.com%2Fready",
   oauth_signature="74KNZJeDHnMBp0EMJ9ZHt%2FXKycU%3D"
```

Dans le déclencheur ou l’action qui prend en charge l’authentification brute, spécifiez les valeurs de propriétés suivantes :

| Propriété (concepteur) | Propriété (JSON) | Obligatoire | Valeur | Description |
|---------------------|-----------------|----------|-------|-------------|
| **Authentification** | `type` | Oui | Brut | Type d’authentification à utiliser |
| **Valeur** | `value` | Oui | <*authorization-header-value*> | Valeur d’en-tête d’autorisation à utiliser pour l’authentification |
||||||

Lorsque vous utilisez des [paramètres sécurisés](#secure-action-parameters) pour traiter et sécuriser des informations sensibles, par exemple dans un [modèle Azure Resource Manager pour l’automatisation du déploiement](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), vous pouvez utiliser des expressions pour accéder à ces valeurs de paramètre au moment de l’exécution. Cet exemple de définition d’action HTTP spécifie l’authentification `type` en tant que `Raw` et utilise la fonction [parameters()](../logic-apps/workflow-definition-language-functions-reference.md#parameters) pour récupérer les valeurs des paramètres :

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "@parameters('endpointUrlParam')",
      "authentication": {
         "type": "Raw",
         "value": "@parameters('authHeaderParam')"
      }
   },
   "runAfter": {}
}
```

<a name="managed-identity-authentication"></a>

### <a name="managed-identity-authentication"></a>Authentification d’une identité managée

Si l’option [Identité managée](../active-directory/managed-identities-azure-resources/overview.md) est disponible sur un [déclencheur ou une action spécifiques](#add-authentication-outbound), votre application logique peut utiliser l’identité affectée par le système ou une *seule* identité affectée par l’utilisateur créée manuellement pour authentifier l’accès à d’autres ressources protégées par Azure Active Directory (Azure AD) sans avoir à se connecter. Azure gère cette identité pour vous et vous aide à sécuriser vos informations d’identification, car vous n’êtes pas obligé de fournir ni de faire pivoter des secrets. En savoir plus sur les [services Azure qui prennent en charge les identités managées pour l’authentification Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

1. Pour que votre application logique puisse utiliser une identité managée, suivez les étapes décrites dans [Authentifier l’accès aux ressources Azure à l’aide des identités managées dans Azure Logic Apps](../logic-apps/create-managed-service-identity.md). Ces étapes activent l’identité managée sur votre application logique et configurent l’accès de cette identité à la ressource Azure cible.

1. Pour qu’une fonction Azure puisse utiliser une identité managée, vous devez d’abord [activer l’authentification des fonctions Azure](../logic-apps/logic-apps-azure-functions.md#enable-authentication-for-azure-functions).

1. Dans le déclencheur ou l’action où vous souhaitez utiliser l’identité managée, spécifiez les valeurs de propriété suivantes :

   | Propriété (concepteur) | Propriété (JSON) | Obligatoire | Valeur | Description |
   |---------------------|-----------------|----------|-------|-------------|
   | **Authentification** | `type` | Oui | **Identité gérée** <br>or <br>`ManagedServiceIdentity` | Type d’authentification à utiliser |
   | **Identité gérée** | `identity` | Oui | * **Identité managée affectée par le système** <br>or <br>`SystemAssigned` <p><p>* <*nom-identité-affectée-par-utilisateur*> | Identité managée à utiliser |
   | **Public ciblé** | `audience` | Oui | <*target-resource-ID*> | ID de la ressource cible à laquelle vous souhaitez accéder. <p>Par exemple, `https://storage.azure.com/` rend les [jetons d’accès](../active-directory/develop/access-tokens.md) pour l’authentification valides pour tous les comptes de stockage. Toutefois, vous pouvez également spécifier une URL de service racine, par exemple `https://fabrikamstorageaccount.blob.core.windows.net` pour un compte de stockage spécifique. <p>**Remarque** : La propriété **Audience** peut être masquée dans certains déclencheurs ou certaines actions. Pour que la propriété apparaisse, dans le déclencheur ou l’action, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez **Audience**. <p><p>**Important !** Vérifiez que cet ID de ressource cible *correspond exactement* à ce qu’attend Azure AD, notamment les barres obliques de fin obligatoires. Ainsi, l’ID de ressource `https://storage.azure.com/` pour tous les comptes Stockage blob Azure requiert une barre oblique finale. Toutefois, l’ID de ressource pour un compte de stockage spécifique ne requiert pas de barre oblique finale. Pour rechercher ces ID de ressource, consultez les [services Azure qui prennent en charge Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication). |
   |||||

   Lorsque vous utilisez des [paramètres sécurisés](#secure-action-parameters) pour traiter et sécuriser des informations sensibles, par exemple dans un [modèle Azure Resource Manager pour l’automatisation du déploiement](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), vous pouvez utiliser des expressions pour accéder à ces valeurs de paramètre au moment de l’exécution. Cet exemple de définition d’action HTTP spécifie l’authentification `type` en tant que `ManagedServiceIdentity` et utilise la fonction [parameters()](../logic-apps/workflow-definition-language-functions-reference.md#parameters) pour récupérer les valeurs des paramètres :

   ```json
   "HTTP": {
      "type": "Http",
      "inputs": {
         "method": "GET",
         "uri": "@parameters('endpointUrlParam')",
         "authentication": {
            "type": "ManagedServiceIdentity",
            "identity": "SystemAssigned",
            "audience": "https://management.azure.com/"
         },
      },
      "runAfter": {}
   }
   ```

<a name="block-connections"></a>

## <a name="block-creating-connections"></a>Blocage de la création de connexions

Si votre organisation n’autorise pas la connexion à des ressources spécifiques à l’aide de leurs connecteurs dans Azure Logic Apps, vous pouvez [bloquer la possibilité de créer ces connexions](../logic-apps/block-connections-connectors.md) pour certains connecteurs dans les flux de travail d’application logique avec [Azure Policy](../governance/policy/overview.md). Pour plus d’informations, consultez [Blocage des connexions créées par des connecteurs spécifiques dans Azure Logic Apps](../logic-apps/block-connections-connectors.md).

<a name="isolation-logic-apps"></a>

## <a name="isolation-guidance-for-logic-apps"></a>Conseils d’isolation pour les applications logiques

Vous pouvez utiliser Azure Logic Apps dans [Azure Government](../azure-government/documentation-government-welcome.md), qui prend en charge tous les niveaux d’impact dans les régions décrites par les guides [Conseils d’isolation pour le niveau d’impact 5 Azure Government](../azure-government/documentation-government-impact-level-5.md#azure-logic-apps) et [Guide des exigences de sécurité (SRG) du cloud computing du département de la Défense des États-Unis](https://dl.dod.cyber.mil/wp-content/uploads/cloud/SRG/index.html). Pour répondre à ces exigences, Logic Apps gère la possibilité de créer et d’exécuter des flux de travail dans un environnement comportant des ressources dédiées, ce qui vous permet de réduire l’impact sur les performances d’autres locataires Azure sur vos applications logiques et d’éviter de partager des ressources de calcul avec d’autres locataires.

* Pour exécuter votre propre code ou effectuer une transformation XML, [créez et appelez une fonction Azure](../logic-apps/logic-apps-azure-functions.md), au lieu respectivement d’utiliser la [fonctionnalité de code inline](../logic-apps/logic-apps-add-run-inline-code.md) ou de fournir des [assemblys à utiliser comme mappages](../logic-apps/logic-apps-enterprise-integration-maps.md). En outre, configurez l’environnement d’hébergement de votre application de fonction de façon à respecter vos exigences d’isolation.

  Par exemple, pour répondre aux exigences du niveau d’impact 5, créez votre application de fonction avec le [plan App Service](../azure-functions/functions-scale.md#app-service-plan) suivant le [niveau tarifaire **isolé**](../app-service/overview-hosting-plans.md), ainsi qu’un [environnement ASE (App Service Environment)](../app-service/environment/intro.md) qui utilise également le niveau tarifaire **Isolé**. Dans cet environnement, les applications de fonction s’exécutent sur des machines virtuelles et des réseaux virtuels Azure dédiés, ce qui assure à vos applications l’isolement réseau en plus de l’isolation du calcul, ainsi que des capacités de Scale-out maximales. Pour plus d’informations, consultez [Conseils d’isolation pour le niveau d’impact 5 Azure Government – Azure Functions](../azure-government/documentation-government-impact-level-5.md#azure-functions).

  Pour plus d’informations, consultez les rubriques suivantes :<p>

  * [Plans Azure App Service](../app-service/overview-hosting-plans.md)
  * [Options de mise en réseau d’Azure Functions](../azure-functions/functions-networking-options.md)
  * [Azure Dedicated Host pour les machines virtuelles](../virtual-machines/dedicated-hosts.md)
  * [Isolation des machines virtuelles dans Azure](../virtual-machines/isolation.md)
  * [Déploiement de services Azure dédiés dans des réseaux virtuels](../virtual-network/virtual-network-for-azure-services.md)

* Pour créer des applications logiques qui s’exécutent sur des ressources dédiées et peuvent accéder à des ressources protégées par un réseau virtuel Azure, vous pouvez créer un [environnement de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md).

  * Certains réseaux virtuels Azure utilisent des points de terminaison privés ([Azure Private Link](../private-link/private-link-overview.md)) pour fournir un accès aux services PaaS Azure, tels que Stockage Azure, Azure Cosmos DB ou Azure SQL Database, aux services partenaires ou aux services clients hébergés sur Azure. Si vos applications logiques ont besoin d’accéder à des réseaux virtuels qui utilisent des points de terminaison privés, vous devez créer, déployer et exécuter ces applications logiques à l’intérieur d’un environnement ISE.

  * Pour mieux contrôler les clés de chiffrement utilisées par le Stockage Azure, vous pouvez configurer, utiliser et gérer votre propre clé avec [Azure Key Vault](../key-vault/general/overview.md). Cette fonctionnalité est également appelée « Bring Your Own Key » (BYOK) et votre clé est appelée « clé gérée par le client ». Pour plus d’informations, consultez [Configuration de clés gérées par le client afin de chiffrer les données au repos pour les environnements de service d’intégration (ISE) dans Azure Logic Apps](../logic-apps/customer-managed-keys-integration-service-environment.md).

Pour plus d’informations, consultez les rubriques suivantes :

* [Isolation dans le cloud public Azure](../security/fundamentals/isolation-choices.md)
* [Sécurité des applications IaaS hautement sensibles dans Azure](/azure/architecture/reference-architectures/n-tier/high-security-iaas)

## <a name="next-steps"></a>Étapes suivantes

* [Base de référence de sécurité Azure pour Azure Logic Apps](../logic-apps/security-baseline.md)
* [Automatiser le déploiement pour le service Azure Logic Apps](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md)
* [Surveiller les applications logiques](../logic-apps/monitor-logic-apps-log-analytics.md)
