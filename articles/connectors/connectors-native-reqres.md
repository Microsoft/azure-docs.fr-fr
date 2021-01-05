---
title: Recevoir des appels et y répondre à l’aide du protocole HTTPS
description: Gérer des requêtes HTTPS entrantes en provenance de services externes à l’aide d’Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewers: jonfan, logicappspm
ms.topic: conceptual
ms.date: 11/19/2020
tags: connectors
ms.openlocfilehash: 4997853fea97d14491bd9e9101f79f324807a6a1
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96920822"
---
# <a name="receive-and-respond-to-inbound-https-requests-in-azure-logic-apps"></a>Recevoir des requêtes HTTPS entrantes et y répondre dans Azure Logic Apps

Avec [Azure Logic Apps](../logic-apps/logic-apps-overview.md) et le déclencheur de requête et l’action de réponse intégrés, vous pouvez créer des tâches et des workflows automatisés capables de recevoir des requêtes entrantes sur HTTPS. Pour envoyer plutôt des requêtes sortantes, utilisez le [déclencheur HTTP ou l’action HTTP](../connectors/connectors-native-http.md) intégrés.

Par exemple, vous pouvez appliquer les actions suivantes à votre application logique :

* Recevoir et répondre à une requête HTTP pour des données situées dans une base de données locale.

* Déclencher un flux de travail lorsqu’un événement de webhook externe se produit.

* Recevoir et répondre à un appel HTTPS en provenance d’une autre application logique.

Cet article explique comment utiliser le déclencheur de requête et l’action de réponse pour que votre application logique puisse recevoir des appels entrants et y répondre.

Pour plus d’informations sur la sécurité, l’autorisation et le chiffrement des appels entrants dans votre application logique, par exemple, sur [TLS (Transport Layer Security)](https://en.wikipedia.org/wiki/Transport_Layer_Security), précédemment appelé SSL (Secure Sockets Layer), [Azure Active Directory Open Authentication (Azure AD OAuth)](../active-directory/develop/index.yml), l’exposition de votre application logique avec Gestion des API Azure, ou la restriction des adresses IP dont proviennent les appels entrants, consultez [Sécuriser l’accès et les données – Accès pour les appels entrants aux déclencheurs basés sur des requêtes](../logic-apps/logic-apps-securing-a-logic-app.md#secure-inbound-requests).

## <a name="prerequisites"></a>Prérequis

* Un compte et un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/).

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md). Si vous débutez avec les applications logiques, voir [Qu’est-ce qu’Azure Logic Apps](../logic-apps/logic-apps-overview.md) ?

<a name="add-request"></a>

## <a name="add-request-trigger"></a>Ajouter un déclencheur de requête

Ce déclencheur intégré crée un point de terminaison manuellement appelable qui peut gérer *uniquement* des requêtes entrantes sur HTTPS. Quand un appelant envoie une requête à ce point de terminaison, le [déclencheur de requête](../logic-apps/logic-apps-workflow-actions-triggers.md#request-trigger) déclenche et exécute l’application logique. Pour plus d’informations sur la manière d’appeler ce déclencheur, consultez [Appeler, déclencher ou imbriquer des workflows avec des points de terminaison HTTPS dans Azure Logic Apps](../logic-apps/logic-apps-http-endpoint.md).

Votre application logique garde une requête entrante ouverte seulement pendant une [durée limitée](../logic-apps/logic-apps-limits-and-config.md#http-limits). En supposant que votre application logique inclue une [action de réponse](#add-response), si elle ne renvoie pas de réponse à l’appelant une fois ce délai écoulé, elle lui renvoie un état `504 GATEWAY TIMEOUT`. Si votre application logique n’inclut pas d’action Réponse, votre application logique renvoie immédiatement un état `202 ACCEPTED` à l’appelant.

1. Connectez-vous au [portail Azure](https://portal.azure.com). Créez une application logique vide.

1. Une fois que le concepteur d’applications logiques s’ouvre, entrez `http request` comme filtre dans la zone de recherche. Dans la liste des déclencheurs, sélectionnez **Lors de la réception d’une requête HTTP**.

   ![Sélectionner le déclencheur de requête](./media/connectors-native-reqres/select-request-trigger.png)

   Le déclencheur de requête affiche les propriétés suivantes :

   ![Déclencheur de requête](./media/connectors-native-reqres/request-trigger.png)

   | Nom de la propriété | Nom de la propriété JSON | Obligatoire | Description |
   |---------------|--------------------|----------|-------------|
   | **URL HTTP POST** | {aucune} | Oui | L’URL de point de terminaison générée après l’enregistrement de l’application logique et utilisée pour appeler votre application logique |
   | **Schéma JSON du corps de la demande** | `schema` | Non | Schéma JSON qui décrit les propriétés et les valeurs dans le corps de la demande entrante |
   |||||

1. Dans la zone **Schéma JSON du corps de la demande**, entrez éventuellement un schéma JSON qui décrit le corps de la demande entrante, par exemple :

   ![Exemple de schéma JSON](./media/connectors-native-reqres/provide-json-schema.png)

   Le concepteur utilise ce schéma pour générer des jetons pour les propriétés dans la requête. De cette façon, votre application logique peut analyser, consommer et transmettre les données de la requête via le déclencheur dans votre flux de travail.

   Voici l'exemple de schéma :

   ```json
   {
      "type": "object",
      "properties": {
         "account": {
            "type": "object",
            "properties": {
               "name": {
                  "type": "string"
               },
               "ID": {
                  "type": "string"
               },
               "address": {
                  "type": "object",
                  "properties": {
                     "number": {
                        "type": "string"
                     },
                     "street": {
                        "type": "string"
                     },
                     "city": {
                        "type": "string"
                     },
                     "state": {
                        "type": "string"
                     },
                     "country": {
                        "type": "string"
                     },
                     "postalCode": {
                        "type": "string"
                     }
                  }
               }
            }
         }
      }
   }
   ```

   Lorsque vous entrez un schéma JSON, le concepteur affiche un rappel pour inclure l'en-tête `Content-Type` dans votre demande et définit cette valeur d’en-tête sur `application/json`. Pour plus d’informations, consultez l’article [Gérer les types de contenu](../logic-apps/logic-apps-content-type.md).

   ![Rappel pour inclure l’en-tête « Content-type »](./media/connectors-native-reqres/include-content-type.png)

   Voici à quoi ressemble cet en-tête au format JSON :

   ```json
   {
      "Content-Type": "application/json"
   }
   ```

   Pour générer un schéma JSON basé sur la charge utile attendue (données), vous pouvez utiliser un outil tel que [JSONSchema.NET](https://jsonschema.net) ou suivre ces étapes :

   1. Dans le déclencheur de requête, sélectionnez **Utiliser l’exemple de charge utile pour générer le schéma**.

      ![Capture d’écran montrant l’option « Utiliser l’exemple de charge utile pour générer le schéma » sélectionnée](./media/connectors-native-reqres/generate-from-sample-payload.png)

   1. Entrez l’exemple de charge utile, puis sélectionnez **Terminé**.

      ![Entrer l’exemple de charge utile pour générer le schéma](./media/connectors-native-reqres/enter-payload.png)

      Voici l'exemple de charge utile :

      ```json
      {
         "account": {
            "name": "Contoso",
            "ID": "12345",
            "address": {
               "number": "1234",
               "street": "Anywhere Street",
               "city": "AnyTown",
               "state": "AnyState",
               "country": "USA",
               "postalCode": "11111"
            }
         }
      }
      ```

1. Pour vérifier que le corps de la demande de l’appel entrant correspond au schéma spécifié, procédez comme suit :

   1. Dans la barre de titre du déclencheur de requête, sélectionnez le bouton représentant des points de suspension ( **...** ).

   1. Dans les paramètres du déclencheur, activez la **validation du schéma**, puis sélectionnez **terminé**.

      Si le corps de la demande de l’appel entrant ne correspond pas à votre schéma, le déclencheur renvoie une `HTTP 400 Bad Request` erreur.

1. Pour spécifier des propriétés supplémentaires, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez les paramètres que vous souhaitez ajouter.

   | Nom de la propriété | Nom de la propriété JSON | Obligatoire | Description |
   |---------------|--------------------|----------|-------------|
   | **Méthode** | `method` | Non | Méthode que les requêtes entrantes doivent utiliser pour appeler l’application logique |
   | **Chemin relatif** | `relativePath` | Non | Chemin relatif pour le paramètre que l’URL de point de terminaison de l’application logique peut accepter |
   |||||

   L’exemple suivant ajoute la propriété **Méthode** :

   ![Ajouter un paramètre de méthode](./media/connectors-native-reqres/add-parameters.png)

   La propriété de **Méthode** apparaît dans le déclencheur afin que vous puissiez sélectionner une méthode dans la liste.

   ![Sélectionner la méthode](./media/connectors-native-reqres/select-method.png)

1. À présent, ajoutez une autre action comme étape suivante dans votre flux de travail. Sous le déclencheur, sélectionnez **Étape suivante** afin de trouver l’action que vous souhaitez ajouter.

   Par exemple, vous pouvez répondre à la demande en [ajoutant une action Réponse](#add-response), que vous pouvez utiliser pour renvoyer une réponse personnalisée et qui est décrite plus loin dans cette rubrique.

   Votre application logique garde la requête entrante ouverte seulement pendant une [durée limitée](../logic-apps/logic-apps-limits-and-config.md#http-limits). En supposant que votre flux de travail d’application logique inclue une action Réponse, si l’application logique ne renvoie pas de réponse une fois ce délai écoulé, votre application logique renvoie un `504 GATEWAY TIMEOUT` à l’appelant. Dans le cas contraire, si votre application logique n’inclut pas d’action Réponse, votre application logique renvoie immédiatement une réponse `202 ACCEPTED` à l’appelant.

1. Lorsque vous avez terminé, enregistrez votre application logique. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

   Cette étape génère l’URL à utiliser pour envoyer la requête qui déclenche l’application logique. Pour copier cette URL, sélectionnez l’icône de copie en regard de l’URL.

   ![URL à utiliser pour déclencher votre application logique](./media/connectors-native-reqres/generated-url.png)

   > [!NOTE]
   > Si vous souhaitez inclure le code de hachage ou le symbole dièse ( **#** ) dans l’URI lors d’un appel au déclencheur de requête, utilisez plutôt cette version encodée : `%25%23`

1. Pour tester votre application logique, envoyez une requête HTTP à l’URL générée.

   Par exemple, vous pouvez utiliser un outil tel que [Postman](https://www.getpostman.com/) pour envoyer la requête HTTP. Pour plus d’informations sur la définition JSON sous-jacente du déclencheur et sur la façon d’appeler ce dernier, consultez ces rubriques : [Type de déclencheur de requête](../logic-apps/logic-apps-workflow-actions-triggers.md#request-trigger) et [Appeler, déclencher ou imbriquer des workflows avec des points de terminaison HTTP dans Azure Logic Apps](../logic-apps/logic-apps-http-endpoint.md).

Pour plus d’informations sur la sécurité, l’autorisation et le chiffrement des appels entrants dans votre application logique, par exemple, sur [TLS (Transport Layer Security)](https://en.wikipedia.org/wiki/Transport_Layer_Security), précédemment appelé SSL (Secure Sockets Layer), [Azure Active Directory Open Authentication (Azure AD OAuth)](../active-directory/develop/index.yml), l’exposition de votre application logique avec Gestion des API Azure, ou la restriction des adresses IP dont proviennent les appels entrants, consultez [Sécuriser l’accès et les données – Accès pour les appels entrants aux déclencheurs basés sur des requêtes](../logic-apps/logic-apps-securing-a-logic-app.md#secure-inbound-requests).

## <a name="trigger-outputs"></a>Sorties du déclencheur

Voici plus d’informations sur les sorties du déclencheur de requête :

| Nom de la propriété JSON | Type de données | Description |
|--------------------|-----------|-------------|
| `headers` | Object | Objet JSON qui décrit les en-têtes de la requête |
| `body` | Object | Objet JSON qui décrit le contenu du corps de la requête |
||||

<a name="add-response"></a>

## <a name="add-a-response-action"></a>Ajouter une action Réponse

Quand vous utilisez le déclencheur de requête pour gérer les requêtes entrantes, vous pouvez modéliser la réponse et renvoyer les résultats de la charge utile à l’appelant à l’aide de l’[action de réponse](../logic-apps/logic-apps-workflow-actions-triggers.md#response-action) intégrée. Vous pouvez utiliser l’action de réponse *uniquement* avec le déclencheur de requête. Cette combinaison avec le déclencheur de requête et l’action de réponse crée le [modèle requête-réponse](https://en.wikipedia.org/wiki/Request%E2%80%93response). Vous pouvez ajouter l’action de réponse n’importe où dans votre workflow, sauf à l’intérieur des boucles Foreach et Until, et des branches parallèles.

> [!IMPORTANT]
> Si une action Réponse contient les en-têtes ci-dessous, Logic Apps les supprime du message de réponse généré sans afficher d’avertissement ou d’erreur :
>
> * `Allow`
> * `Content-*` à ces exceptions près : `Content-Disposition`, `Content-Encoding` et `Content-Type`
> * `Cookie`
> * `Expires`
> * `Last-Modified`
> * `Set-Cookie`
> * `Transfer-Encoding`
>
> Si Logic Apps ne vous empêche pas d’enregistrer des applications logiques ayant une action Réponse contenant ces en-têtes, le service les ignore.

1. Pour le Concepteur d'application logique, sous l’étape où vous souhaitez ajouter l'action Réponse, sélectionnez **Nouvelle étape**.

   Par exemple, avec le déclencheur de requête vu plus tôt :

   ![Ajouter une nouvelle étape](./media/connectors-native-reqres/add-response.png)

   Pour ajouter une action entre des étapes, placez votre pointeur au-dessus de la flèche qui les sépare. Cliquez sur le signe ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action**.

1. Sous **Choisir une action**, dans la zone de recherche, entrez `response` comme filtre, puis sélectionnez l’action **Réponse**.

   ![Sélectionner l’action Réponse](./media/connectors-native-reqres/select-response-action.png)

   Le déclencheur de requête est réduit dans cet exemple pour des raisons de simplicité.

1. Ajoutez toutes les valeurs requises pour le message de réponse.

   Dans certains champs, cliquer dans leurs zones ouvre la liste de contenu dynamique. Vous pouvez ensuite sélectionner des jetons qui représentent les sorties disponibles à partir des étapes précédentes du workflow. Les propriétés du schéma spécifié dans l’exemple précédent apparaissent désormais dans la liste de contenu dynamique.

   Par exemple, pour la zone **En-têtes**, incluez `Content-Type` comme nom de clé et définissez la valeur de clé sur `application/json` comme indiqué précédemment dans cette rubrique. Pour la zone **Corps**, vous pouvez sélectionner la sortie du corps du déclencheur dans la liste de contenu dynamique.

   ![Détails de l’action Réponse](./media/connectors-native-reqres/response-details.png)

   Pour afficher les en-têtes au format JSON, sélectionnez **Basculer vers la vue texte**.

   ![En-têtes - Basculer vers la vue texte](./media/connectors-native-reqres/switch-to-text-view.png)

   Voici plus d’informations sur les propriétés que vous pouvez définir dans l’action Réponse.

   | Nom de la propriété | Nom de la propriété JSON | Obligatoire | Description |
   |---------------|--------------------|----------|-------------|
   | **Code d’état** | `statusCode` | Oui | Code d’état à retourner dans la réponse |
   | **En-têtes** | `headers` | Non | Objet JSON qui décrit un ou plusieurs en-têtes à inclure dans la réponse |
   | **Corps** | `body` | Non | Le corps de texte de la réponse |
   |||||

1. Pour spécifier des propriétés supplémentaires, comme un schéma JSON pour le corps de la réponse, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez les paramètres que vous souhaitez ajouter.

1. Lorsque vous avez terminé, enregistrez votre application logique. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

## <a name="next-steps"></a>Étapes suivantes

* [Sécuriser l’accès et les données - Accès pour les appels entrants aux déclencheurs basés sur des requêtes](../logic-apps/logic-apps-securing-a-logic-app.md#secure-inbound-requests)
* [Connecteurs pour Logic Apps](../connectors/apis-list.md)
