---
title: Appeler, déclencher ou imbriquer des applications logiques à l’aide de déclencheurs de demande
description: Configurer des points de terminaison HTTPS pour appeler, déclencher ou imbriquer des flux de travail d’application logique dans Azure Logic Apps
services: logic-apps
ms.workload: integration
ms.reviewer: jonfan, logicappspm
ms.topic: article
ms.date: 11/19/2020
ms.openlocfilehash: b345168dad63b1846d46c12721587eaffb5f887e
ms.sourcegitcommit: f311f112c9ca711d88a096bed43040fcdad24433
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94981202"
---
# <a name="call-trigger-or-nest-logic-apps-by-using-https-endpoints-in-azure-logic-apps"></a>Appeler, déclencher ou imbriquer des applications logiques à l’aide de points de terminaison HTTPS dans Azure Logic Apps

Afin que votre application logique puisse être appelée via une URL, et recevoir des demandes entrantes d’autres services, vous pouvez exposer en mode natif un point de terminaison HTTPS synchrone utilisant un déclencheur basé sur une requête sur cette application logique. Avec cette fonctionnalité, vous pouvez appeler votre application logique à partir d’autres applications logiques, et créer un modèle de points de terminaison pouvant être appelés. Pour configurer un point de terminaison pouvant être appelé pour la gestion des appels entrants, vous pouvez utiliser l’un des types de déclencheurs suivants :

* [Requête](../connectors/connectors-native-reqres.md)
* [Déclencheur HTTPWebhook](../connectors/connectors-native-webhook.md)
* Déclencheurs de connecteur managé du [type ApiConnectionWebhook](../logic-apps/logic-apps-workflow-actions-triggers.md#apiconnectionwebhook-trigger) et pouvant recevoir des requêtes HTTPS entrantes

Cet article montre comment créer un point de terminaison pouvant être appelé sur votre application logique en utilisant le déclencheur de requête, et appeler ce point de terminaison à partir d’une autre application logique. Tous les principes s’appliquent de la même façon aux autres types de déclencheurs que vous pouvez utiliser pour recevoir des demandes entrantes.


Pour plus d’informations sur la sécurité, l’autorisation et le chiffrement des appels entrants dans votre application logique, par exemple, sur [TLS (Transport Layer Security)](https://en.wikipedia.org/wiki/Transport_Layer_Security), précédemment appelé SSL (Secure Sockets Layer), [Azure Active Directory Open Authentication (Azure AD OAuth)](../active-directory/develop/index.yml), l’exposition de votre application logique avec Gestion des API Azure, ou la restriction des adresses IP dont proviennent les appels entrants, consultez [Sécuriser l’accès et les données - Accès pour les appels entrants aux déclencheurs basés sur des requêtes](../logic-apps/logic-apps-securing-a-logic-app.md#secure-inbound-requests).

## <a name="prerequisites"></a>Prérequis

* Un compte et un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/).

* L’application logique dans laquelle vous souhaitez utiliser le déclencheur pour créer le point de terminaison pouvant être appelé. Vous pouvez commencer avec une application logique vide ou une application logique existante dans de laquelle vous pouvez remplacer le déclencheur actuel. Pour cet exemple, vous avez besoin d’une application logique vide. Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps](../logic-apps/logic-apps-overview.md) et [Démarrage rapide : Créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="create-a-callable-endpoint"></a>Créer un point de terminaison pouvant être appelé

1. Connectez-vous au [portail Azure](https://portal.azure.com). Créez et ouvrez une application logique vide dans le concepteur d’applications logiques.

1. Dans la zone de recherche, sélectionnez **Intégré**. Dans la zone de recherche, entrez `request` en guise de filtre. Dans la liste Déclencheurs, sélectionnez **Lors de la réception d’une requête HTTP**.

   ![Localisez et sélectionnez le déclencheur Requête](./media/logic-apps-http-endpoint/find-and-select-request-trigger.png)

1. Si vous le souhaitez, dans la boîte du **schéma JSON du corps de la requête**, vous pouvez entrer un schéma JSON qui décrit la charge utile ou les données que le déclencheur doit recevoir.

   Le concepteur utilise ce schéma pour générer des jetons qui représentent des sorties de déclencheur. Vous pouvez ensuite facilement référencer ces sorties dans le flux de travail de votre application logique. Cliquez sur le lien renvoyant à la section [Jetons générés à partir de schémas JSON pour votre application logique](#generated-tokens) pour en apprendre davantage.

   Pour cet exemple, entrez ce schéma :

   ```json
      {
      "type": "object",
      "properties": {
         "address": {
            "type": "object",
            "properties": {
               "streetNumber": {
                  "type": "string"
               },
               "streetName": {
                  "type": "string"
               },
               "town": {
                  "type": "string"
               },
               "postalCode": {
                  "type": "string"
               }
            }
         }
      }
   }
    ```

   ![Fournir le schéma JSON pour l’action de requête](./media/logic-apps-http-endpoint/manual-request-trigger-schema.png)

   Ou vous pouvez générer un schéma JSON en fournissant un exemple de charge utile :

   1. Dans le déclencheur de **requête**, sélectionnez **Utiliser l’exemple de charge utile pour générer le schéma**.

   1. Dans la boîte **Entrer ou coller un exemple de charge utile JSON**, entrez votre exemple de charge utile, par exemple :

      ```json
      {
         "address": {
            "streetNumber": "00000",
            "streetName": "AnyStreet",
            "town": "AnyTown",
            "postalCode": "11111-1111"
        }
      }
      ```

   1. Quand vous êtes prêt, sélectionnez **Terminé**.

      La boîte de **Schéma JSON du corps de la requête** affiche maintenant le schéma généré.

1. Enregistrez votre application logique.

   La zone **URL POST HTTP** affiche désormais l’URL de rappel générée que les autres services peuvent utiliser pour appeler et déclencher votre application logique. Cette URL contient des paramètres de requête qui spécifient une clé de signature d’accès partagé (SAP), qui est utilisée pour l’authentification.

   ![URL de rappel générée pour le point de terminaison](./media/logic-apps-http-endpoint/generated-endpoint-url.png)

1. Pour copier l’URL de rappel, vous disposez des options suivantes :

   * À droite de la zone **HTTP POST URL** (URL POST HTTP), sélectionnez **Copy Url** (Copier l’URL) (icône de copie de fichier).

   * Effectuez cet appel à l’aide de la méthode attendue par le déclencheur de requête. Cet exemple utilise la méthode `POST` :

     `POST https://management.azure.com/{logic-app-resource-ID}/triggers/{endpoint-trigger-name}/listCallbackURL?api-version=2016-06-01`

   * Copiez l’URL de rappel à partir du volet **Overview** (Vue d’ensemble) de l’application logique.

     1. Dans le menu de votre application logique, sélectionnez **Vue d’ensemble**.

     1. Dans la section **Résumé**, sélectionnez **Afficher l’historique du déclencheur**.

        ![Obtenir l’URL de point de terminaison à partir du portail Azure](./media/logic-apps-http-endpoint/find-manual-trigger-url.png)

     1. Sous **URL de rappel [POST]** , copiez l’URL :

        ![Copier l’URL de point de terminaison à partir du portail Azure](./media/logic-apps-http-endpoint/copy-manual-trigger-callback-url-post.png)

<a name="select-method"></a>

## <a name="select-expected-request-method"></a>Sélectionner la méthode de requête attendue

Par défaut, le déclencheur de requête attend une requête `POST`. Cependant, vous pouvez spécifier une autre méthode à utiliser par l’appelant, mais il doit s’agir d’une méthode unique.

1. Dans le déclencheur de requête, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez **Méthode** qui ajoute cette propriété au déclencheur.

   ![Ajouter la propriété « Méthode » au déclencheur](./media/logic-apps-http-endpoint/select-add-new-parameter-for-method.png)

1. Dans la liste **Méthode**, sélectionnez la méthode attendue par le déclencheur. Ou vous pouvez spécifier une méthode personnalisée.

   Par exemple, sélectionnez la méthode **GET** pour pouvoir tester ultérieurement l’URL de votre point de terminaison.

   ![Sélectionner la méthode de requête attendue par le déclencheur](./media/logic-apps-http-endpoint/select-method-request-trigger.png)

<a name="endpoint-url-parameters"></a>

## <a name="pass-parameters-through-endpoint-url"></a>Passer des paramètres via l’URL de point de terminaison

Lorsque vous souhaitez accepter des valeurs de paramètre par le biais de l’URL du point de terminaison, vous disposez des options suivantes :

* [Accepter les valeurs par le biais des paramètres d’extraction](#get-parameters) ou des paramètres d’URL.

  Ces valeurs sont passées en tant que paires nom-valeur dans l’URL du point de terminaison. Pour cette option, vous devez utiliser la méthode GET dans votre déclencheur de requête. Plus tard, vous pouvez obtenir les valeurs de paramètre en tant que sorties de déclencheur à l’aide de la fonction `triggerOutputs()` dans une expression.

* [Acceptez les valeurs via un chemin d’accès relatif](#relative-path) pour les paramètres de votre déclencheur de requête.

  Ces valeurs sont transmises via un chemin d’accès relatif dans l’URL du point de terminaison. Vous devez également [explicitement sélectionner la méthode](#select-method) que le déclencheur attend. Plus tard, vous pouvez obtenir les valeurs des paramètre en tant que sorties de déclencheur en référençant directement ces sorties.

<a name="get-parameters"></a>

### <a name="accept-values-through-get-parameters"></a>Accepter les valeurs par le biais des paramètres GET

1. Dans le déclencheur Request, ouvrez la liste **Add new parameter** (Ajouter un nouveau paramètre), ajoutez la propriété **Method** (Méthode) au déclencheur, puis sélectionnez la méthode **GET**.

   Pour plus d’informations, consultez [Sélectionner la méthode de requête attendue](#select-method).

1. Sous le déclencheur Request, ajoutez l’action dans laquelle vous souhaitez utiliser la valeur de paramètre. Pour cet exemple, ajoutez l’action **Response**.

   1. Sous le déclencheur de requête, sélectionnez **Nouvelle étape** > **Ajouter une action**.
   
   1. Sous **Choisir une action**, dans la zone de recherche, entrez `response` en guise de filtre. Dans la liste des actions, sélectionnez l’action **Réponse**.

1. Pour générer l’expression `triggerOutputs()` qui récupère la valeur du paramètre, procédez comme suit :

   1. Cliquez à l’intérieur de la propriété **Body** de l’action Response pour que la liste de contenu dynamique s’affiche, puis sélectionnez **Expression**.

   1. Dans la zone **Expression**, entrez cette expression, en remplaçant `parameter-name` par le nom de votre paramètre, puis sélectionnez **OK**.

      `triggerOutputs()['queries']['parameter-name']`

      ![Ajoute de l’expression "triggerOutputs()" au déclencheur](./media/logic-apps-http-endpoint/trigger-outputs-expression.png)

      Dans la propriété **Body**, l’expression est résolue en jeton `triggerOutputs()`.

      ![Expression "triggerOutputs()" résolue](./media/logic-apps-http-endpoint/trigger-outputs-expression-token.png)

      Si vous enregistrez l’application logique, quittez le concepteur, puis revenez-y. Le jeton affiche le nom du paramètre que vous avez spécifié, par exemple :

      ![Expression résolue pour le nom de paramètre](./media/logic-apps-http-endpoint/resolved-expression-parameter-token.png)

      En mode Code, la propriété **Body** apparaît dans la définition de l’action Response comme suit :

      `"body": "@{triggerOutputs()['queries']['parameter-name']}",`

      Supposons, par exemple, que vous souhaitiez passer une valeur pour un paramètre nommé `postalCode`. La propriété **Body** spécifie la chaîne, `Postal Code: ` avec un espace de fin, suivi par l’expression correspondante :

      ![Ajout d’expression d’exemple "triggerOutputs()" au déclencheur](./media/logic-apps-http-endpoint/trigger-outputs-expression-postal-code.png)

1. Pour tester votre point de terminaison pouvant être appelé, copiez l’URL de rappel du déclencheur Request, puis collez l’URL dans une autre fenêtre de navigateur. Dans l’URL, ajoutez le nom du paramètre et la valeur suivant le point d’interrogation (`?`) à l’URL au format suivant, puis appuyez sur Entrée.

   `...?{parameter-name=parameter-value}&api-version=2016-10-01...`

   `https://prod-07.westus.logic.azure.com:433/workflows/{logic-app-resource-ID}/triggers/manual/paths/invoke?{parameter-name=parameter-value}&api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig={shared-access-signature}`

   Le navigateur renvoie une réponse avec le texte suivant : `Postal Code: 123456`

   ![Réponse à l’envoi d’une requête vers l’URL de rappel](./media/logic-apps-http-endpoint/callback-url-returned-response.png)

1. Pour placer le nom et la valeur du paramètre à un autre endroit de l’URL, veillez à utiliser l’esperluette (`&`) comme préfixe, par exemple :

   `...?api-version=2016-10-01&{parameter-name=parameter-value}&...`

   Cet exemple montre l’URL de rappel avec l’exemple de nom de paramètre et la valeur `postalCode=123456` à des positions différentes dans l’URL :

   * 1re position : `https://prod-07.westus.logic.azure.com:433/workflows/{logic-app-resource-ID}/triggers/manual/paths/invoke?postalCode=123456&api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig={shared-access-signature}`

   * 2e position : `https://prod-07.westus.logic.azure.com:433/workflows/{logic-app-resource-ID}/triggers/manual/paths/invoke?api-version=2016-10-01&postalCode=123456&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig={shared-access-signature}`

> [!NOTE]
> Si vous souhaitez inclure le code de hachage ou le symbole dièse ( **#** ) dans l’URI, utilisez plutôt cette version encodée : `%25%23`.

<a name="relative-path"></a>

### <a name="accept-values-through-a-relative-path"></a>Accepter les valeurs via un chemin d’accès relatif

1. Dans le déclencheur de requête, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez **Chemin d’accès relatif** qui ajoute cette propriété au déclencheur.

   ![Ajouter la propriété « Chemin d’accès relatif » au déclencheur](./media/logic-apps-http-endpoint/select-add-new-parameter-for-relative-path.png)

1. Dans la propriété **Chemin d’accès relatif**, spécifiez le chemin d’accès relatif que vous souhaitez que votre URL accepte pour le paramètre dans votre schéma JSON, par exemple `/address/{postalCode}`.

   ![Spécifier le chemin d’accès relatif pour le paramètre](./media/logic-apps-http-endpoint/relative-path-url-value.png)

1. Sous le déclencheur Request, ajoutez l’action dans laquelle vous souhaitez utiliser la valeur du paramètre. Pour cet exemple, ajoutez l’action **Response**.

   1. Sous le déclencheur de requête, sélectionnez **Nouvelle étape** > **Ajouter une action**.

   1. Sous **Choisir une action**, dans la zone de recherche, entrez `response` en guise de filtre. Dans la liste des actions, sélectionnez l’action **Réponse**.

1. Dans la propriété de **corps** de l’action de réponse, incluez le jeton qui représente le paramètre que vous avez spécifié dans le chemin d’accès relatif de votre déclencheur.

   Par exemple, supposons que vous souhaitiez que l’action de réponse retourne `Postal Code: {postalCode}`.

   1. Dans la propriété de **corps**, entrez `Postal Code: ` avec un espace de fin. Maintenez votre curseur à l’intérieur de la zone d’édition pour que la liste de contenu dynamique reste ouverte.

   1. Dans la liste de contenu dynamique, dans la section **When a HTTP request is received** (Lors de la réception d’une requête HTTP), sélectionnez le jeton **postalCode**.

      ![Ajouter le paramètre spécifié au corps de la réponse](./media/logic-apps-http-endpoint/relative-url-with-parameter-token.png)

      La propriété de **corps** contient désormais le paramètre sélectionné :

      ![Exemple de corps de la réponse avec le paramètre](./media/logic-apps-http-endpoint/relative-url-with-parameter.png)

1. Enregistrez votre application logique.

   L’URL de rappel est mise à jour dans le déclencheur Request. Elle comprend maintenant le chemin d’accès relatif, par exemple :

   `https://prod-07.westus.logic.azure.com/workflows/{logic-app-resource-ID}/triggers/manual/paths/invoke/address/{postalCode}?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig={shared-access-signature}`

1. Pour tester votre point de terminaison pouvant être appelé, copiez l’URL de rappel mise à jour à partir du déclencheur Request, collez-la dans une autre fenêtre de navigateur, remplacez `{postalCode}` dans l’URL par `123456`, puis appuyez sur entrée.

   Le navigateur renvoie une réponse avec le texte suivant : `Postal Code: 123456`

   ![Réponse à l’envoi d’une requête vers l’URL de rappel](./media/logic-apps-http-endpoint/callback-url-returned-response.png)

> [!NOTE]
> Si vous souhaitez inclure le code de hachage ou le symbole dièse ( **#** ) dans l’URI, utilisez plutôt cette version encodée : `%25%23`.

## <a name="call-logic-app-through-endpoint-url"></a>Appeler une application logique via une URL de point de terminaison

Après avoir créé le point de terminaison, vous pouvez déclencher l’application logique en envoyant une requête HTTPS à l’URL complète du point de terminaison. Les applications logiques ont une prise en charge intégrée pour les points de terminaison à accès direct.

<a name="generated-tokens"></a>

## <a name="tokens-generated-from-schema"></a>Jetons générés à partir du schéma

Lorsque vous fournissez un schéma JSON dans le déclencheur de requête, le concepteur d’application logique génère des jetons pour les propriétés de ce schéma. Vous pouvez ensuite utiliser ces jetons pour transmettre des données au workflow de votre application logique.

Par exemple, si vous ajoutez d’autres propriétés, telles que `"suite"`, à votre schéma JSON, vous pouvez utiliser des jetons pour ces propriétés dans les étapes ultérieures de votre application logique. Voici le schéma JSON complet :

```json
   {
   "type": "object",
   "properties": {
      "address": {
         "type": "object",
         "properties": {
            "streetNumber": {
               "type": "string"
            },
            "streetName": {
               "type": "string"
            },
            "suite": {
               "type": "string"
            },
            "town": {
               "type": "string"
            },
            "postalCode": {
               "type": "string"
            }
         }
      }
   }
}
```

## <a name="create-nested-logic-apps"></a>Créer des applications logiques imbriquées

Vous pouvez imbriquer des workflows dans votre application logique en ajoutant d’autres applications logiques qui peuvent recevoir des requêtes. Pour inclure ces applications logiques, procédez comme suit :

1. Sous l’étape lors de laquelle vous souhaitez appeler une autre application logique, sélectionnez **Nouvelle étape** > **Ajouter une action**.

1. Sous **Choisir une action**, sélectionnez **Intégré**. Dans la zone de recherche, entrez `logic apps` en guise de filtre. Dans la liste actions, sélectionnez **Choisir un workflow Logic Apps**.

   ![Imbriquer une application logique à l’intérieur de l’application logique actuelle](./media/logic-apps-http-endpoint/choose-logic-apps-workflow.png)

   Le concepteur affiche les applications logiques éligibles parmi lesquelles vous pouvez choisir.

1. Sélectionnez l’application logique à appeler à partir de votre application logique actuelle.

   ![Sélectionner une application logique à appeler à partir de l’application logique actuelle](./media/logic-apps-http-endpoint/select-logic-app-to-nest.png)

## <a name="reference-content-from-an-incoming-request"></a>Référencer le contenu à partir d’une requête entrante

Si le type de contenu de la requête entrante est `application/json`, vous pouvez référencer les propriétés dans la requête entrante. Sinon, ce contenu est traité comme une seule unité binaire que vous pouvez transmettre à d’autres API. Pour référencer ce contenu dans le flux de travail de votre application logique, vous devez d’abord convertir ce contenu.

Par exemple, si vous transmettez du contenu dont le type de `application/xml` est, vous pouvez utiliser l’expression [`@xpath()`](../logic-apps/workflow-definition-language-functions-reference.md#xpath) pour effectuer une extraction XPath ou utiliser l’expression [`@json()`](../logic-apps/workflow-definition-language-functions-reference.md#json) pour la conversion de XML en JSON. En savoir plus sur l’utilisation des [types de contenus](../logic-apps/logic-apps-content-type.md) pris en charge.

Pour obtenir la sortie à partir d’une requête entrante, vous pouvez utiliser l’expression [`@triggerOutputs`](../logic-apps/workflow-definition-language-functions-reference.md#triggerOutputs). Par exemple, supposons que vous ayez une sortie ressemblant à l’exemple suivant :

```json
{
   "headers": {
      "content-type" : "application/json"
   },
   "body": {
      "myProperty" : "property value"
   }
}
```

Pour accéder spécifiquement à la propriété `body`, vous pouvez utiliser l’expression [`@triggerBody()`](../logic-apps/workflow-definition-language-functions-reference.md#triggerBody) comme raccourci.

## <a name="respond-to-requests"></a>Répondre aux requêtes

Parfois, vous souhaitez répondre à certaines demandes qui déclenchent votre application logique en renvoyant le contenu à l’appelant. Pour créer le code d’état, l’en-tête et le corps de votre réponse, utilisez l’action Réponse. Cette action peut apparaître n’importe où dans votre application logique, et pas seulement à la fin de votre workflow. Si votre application logique n’inclut pas d’action Réponse, le point de terminaison répond *immédiatement* avec un état **202 - Accepté**.

Pour que l’appelant d’origine obtienne la réponse, toutes les étapes nécessaires pour la réponse doivent être terminées avant la [limite du délai d’expiration de la requête](./logic-apps-limits-and-config.md), sauf si l’application logique déclenchée est appelée en tant qu’application logique imbriquée. Si aucune réponse n’est retournée avant cette limite, la requête entrante expire et reçoit la réponse **408 - Dépassement du délai d’expiration par le client**.

Pour les applications logiques imbriquées, l’application logique parente continue à attendre une réponse jusqu’à ce que toutes les étapes aient été effectuées, quelle que soit la durée de l’opération.

### <a name="construct-the-response"></a>Construire la réponse

Vous pouvez inclure plusieurs en-têtes et n’importe quel type de contenu dans le corps de la réponse. Par exemple, l’en-tête de cette réponse spécifie que le type de contenu de la réponse est `application/json` et que le corps contient des valeurs pour les propriétés `town` et `postalCode`, en fonction du schéma JSON décrit précédemment dans cette rubrique pour le déclencheur de requête.

![Fournir le contenu de la réponse pour l’action de réponse HTTPS](./media/logic-apps-http-endpoint/content-for-response-action.png)

Les réponses ont ces propriétés :

| Propriété (affichage) | Propriété (JSON) | Description |
|--------------------|-----------------|-------------|
| **Code d’état** | `statusCode` | Le code d’état HTTPS à utiliser dans la réponse pour la requête entrante. Ce code peut être tout code d’état valide commençant par 2xx, 4xx ou 5xx. Cependant, les codes d’état 3xx ne sont pas autorisés. |
| **En-têtes** | `headers` | Un ou plusieurs en-têtes à inclure dans la réponse |
| **Corps** | `body` | Un objet corps peut être une chaîne, un objet JSON ou même du contenu binaire référencé à partir d’une étape précédente |
||||

Pour afficher la définition JSON de l’action de réponse et de la définition JSON complète de votre application logique, sélectionnez **Mode Code** dans la barre d’outils du concepteur d’applications logiques.

``` json
"Response": {
   "type": "Response",
   "kind": "http",
   "inputs": {
      "body": {
         "postalCode": "@triggerBody()?['address']?['postalCode']",
         "town": "@triggerBody()?['address']?['town']"
      },
      "headers": {
         "content-type": "application/json"
      },
      "statusCode": 200
   },
   "runAfter": {}
}
```

## <a name="q--a"></a>Questions et réponses

#### <a name="q-what-about-url-security"></a>Q : Qu’en est-il de la sécurité de l’URL ?

**R** : Les URL de rappel de l’application logique sont générées de façon sécurisée par Azure à l’aide d’une [signature d’accès partagé (SAP)](/rest/api/storageservices/delegate-access-with-shared-access-signature). Cette signature est transmise directement comme paramètre de requête et doit être validée avant que votre application logique puisse être exécutée. Azure génère cette signature via la combinaison unique d’une clé secrète par application logique, du nom du déclencheur et de l’opération qui est effectuée. Ainsi, à moins que quelqu’un ait accès à la clé secrète de l’application logique, personne ne peut générer de signature valide.

> [!IMPORTANT]
> Pour les systèmes de production et les systèmes ayant un niveau de sécurité supérieur, nous vous déconseillons fortement d’appeler votre application logique directement à partir du navigateur pour les raisons suivantes :
>
> * la clé d’accès partagé s’affiche dans l’URL ;
> * Vous ne pouvez pas gérer les stratégies de contenu de sécurité en raison du partage de domaines entre les clients Azure Logic Apps.

Pour plus d’informations sur la sécurité, l’autorisation et le chiffrement des appels entrants dans votre application logique, par exemple, sur [TLS (Transport Layer Security)](https://en.wikipedia.org/wiki/Transport_Layer_Security), précédemment appelé SSL (Secure Sockets Layer), [Azure Active Directory Open Authentication (Azure AD OAuth)](../active-directory/develop/index.yml), l’exposition de votre application logique avec Gestion des API Azure, ou la restriction des adresses IP dont proviennent les appels entrants, consultez [Sécuriser l’accès et les données - Accès pour les appels entrants aux déclencheurs basés sur des requêtes](../logic-apps/logic-apps-securing-a-logic-app.md#secure-inbound-requests).

#### <a name="q-can-i-configure-callable-endpoints-further"></a>Q : Puis-je configurer des points de terminaison que je peux appeler de façon plus approfondie ?

**R** : Oui, les points de terminaison HTTPS prennent en charge une configuration plus avancée par le biais de la [Gestion des API Azure](../api-management/api-management-key-concepts.md). Ce service vous offre également la possibilité de gérer toutes vos API de façon systématique, y compris les applications logiques, de configurer les noms de domaines personnalisés, d’utiliser plus de méthodes d’authentification et bien plus encore, comme par exemple :

* [Modification de la méthode de la requête](../api-management/api-management-advanced-policies.md#SetRequestMethod)
* [Modification des segments d’URL de la requête](../api-management/api-management-transformation-policies.md#RewriteURL)
* Configuration de vos domaines Gestion des API dans le [portail Azure](https://portal.azure.com/)
* Configuration d’une stratégie pour vérifier l’authentification de base

## <a name="next-steps"></a>Étapes suivantes

* [Recevoir et répondre aux appels HTTPS entrants à l’aide d’Azure Logic Apps](../connectors/connectors-native-reqres.md)
* [Sécuriser l’accès et les données dans Azure Logic Apps - Accès pour les appels entrants aux déclencheurs basés sur des requêtes](../logic-apps/logic-apps-securing-a-logic-app.md#secure-inbound-requests)
