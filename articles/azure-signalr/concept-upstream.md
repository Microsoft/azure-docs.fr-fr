---
title: Paramètres en amont dans Azure SignalR Service
description: Découvrez les paramètres en amont et les protocoles des messages en amont.
author: chenyl
ms.service: signalr
ms.topic: conceptual
ms.date: 06/11/2020
ms.author: chenyl
ms.openlocfilehash: 1d51f5e8d2fac1e2b180a608c840d0a322e76271
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92143243"
---
# <a name="upstream-settings"></a>Paramètres en amont

En amont est une fonctionnalité d’évaluation qui permet à Azure SignalR Service d’envoyer des messages et des événements de connexion à un ensemble de points de terminaison en mode serverless. Vous pouvez utiliser l’amont pour appeler une méthode Hub à partir de clients en mode serverless et laisser les points de terminaison recevoir des notifications lorsque des connexions client sont établies ou interrompues.

> [!NOTE]
> Seul le mode serverless peut configurer des paramètres en amont.

## <a name="details-of-upstream-settings"></a>Détails des paramètres en amont

Les paramètres en amont sont constitués d’une liste d’éléments sensibles à l’ordre. Chaque élément se compose de ce qui suit :

* un modèle d’URL spécifiant où les messages sont envoyés ;
* un ensemble de règles ;
* des configurations d’authentification. 

Lorsque l’événement spécifié se produit, les règles d’un élément sont vérifiées une à une dans l’ordre. Des messages sont envoyés à la première URL amont de l’élément correspondant.

### <a name="url-template-settings"></a>Paramètres de modèle d’URL

Vous pouvez paramétrer l’URL pour prendre en charge différents modèles. Il existe trois paramètres prédéfinis :

|Paramètre prédéfini|Description|
|---------|---------|
|{hub}| Un Hub est un concept d’Azure SignalR Service. Un Hub est une unité d’isolation. L’étendue des utilisateurs et de la remise des messages est limitée à un Hub.|
|{category}| Une catégorie peut avoir l’une des valeurs suivantes : <ul><li>**connections** : événements de durée de vie de connexion. La valeur est déclenchée quand une connexion client est connectée ou déconnectée. Elle inclut les événements connectés et déconnectés.</li><li>**messages** : la valeur est déclenchée quand les clients appellent une méthode Hub. Elle inclut tous les autres événements, hormis ceux de la catégorie **connections**.</li></ul>|
|{event}| Pour la catégorie **messages**, un événement est la cible d’un [message d’appel](https://github.com/dotnet/aspnetcore/blob/master/src/SignalR/docs/specs/HubProtocol.md#invocation-message-encoding) que le client envoie. Pour la catégorie **connections**, seules les valeurs *connecté* et *déconnecté* sont utilisées.|

Ces paramètres prédéfinis peuvent être utilisés dans le modèle d’URL. Les paramètres seront remplacés par une valeur spécifiée lorsque vous évaluerez l’URL en amont. Par exemple : 
```
http://host.com/{hub}/api/{category}/{event}
```
quand une connexion client dans le Hub « conversation » est établie, un message est envoyé à cette URL :
```
http://host.com/chat/api/connections/connected
```
Quand un client dans le Hub « conversation » appelle la méthode Hub `broadcast`, un message est envoyé à cette URL :
```
http://host.com/chat/api/messages/broadcast
```

### <a name="rule-settings"></a>Paramètres de la règle

Vous pouvez définir des règles pour les *règles de Hub*, les *règles de catégorie* et les *règles d’événement* séparément. La règle de correspondance prend en charge trois formats. Prenons l’exemple des règles d’événement :
- Utilisez un astérisque (*) pour faire correspondre tous les événements.
- Utilisez une virgule (,) pour joindre plusieurs événements. Par exemple, `connected, disconnected` correspond aux événements connectés et déconnectés.
- Utilisez le nom complet de l’événement pour correspondre à l’événement. Par exemple, `connected` correspond à l’événement connecté.

> [!NOTE]
> Si vous utilisez Azure Functions et le [déclencheur Signalr](../azure-functions/functions-bindings-signalr-service-trigger.md), celui-ci expose un seul point de terminaison au format suivant : `https://<APP_NAME>.azurewebsites.net/runtime/webhooks/signalr?code=<API_KEY>`.
> Vous pouvez simplement configurer le modèle d’url sur cette url.

### <a name="authentication-settings"></a>Authentication settings

Vous pouvez configurer l’authentification pour chaque paramètre en amont séparément. Quand vous configurez l’authentification, un jeton est défini dans l’en-tête `Authentication` du message en amont. Actuellement, Azure SignalR Service prend en charge les types d’authentifications suivants :
- `None`
- `ManagedIdentity`

Lorsque vous sélectionnez `ManagedIdentity`, vous devez activer une identité managée dans Azure SignalR Service à l’avance et éventuellement spécifier une ressource. Pour plus de détails, consultez [Identités managées pour Azure SignalR Service](howto-use-managed-identity.md).

## <a name="create-upstream-settings-via-the-azure-portal"></a>Créer des paramètres en amont via le portail Azure

1. Accédez à Azure SignalR Service.
2. Sélectionnez **Paramètres** et définissez l’option **Mode de service** sur **serverless**. Les paramètres en amont s’affichent :

    :::image type="content" source="media/concept-upstream/upstream-portal.png" alt-text="Paramètres en amont":::

3. Ajoutez des URL sous **Modèle d’URL en amont**. Ensuite, des paramètres tels que **Règles de Hub** affichent la valeur par défaut.
4. Pour définir des paramètres pour les **Règles de Hub**, les **Règles d’événement**, les **Règles de catégorie** et l’**Authentification en amont**, sélectionnez la valeur des **Règles de Hub**. Une page permettant de modifier les paramètres s’affiche :

    :::image type="content" source="media/concept-upstream/upstream-detail-portal.png" alt-text="Paramètres en amont":::

5. Pour définir l’**Authentification en amont**, vérifiez que vous avez d’abord activé une identité managée. Sélectionnez ensuite **Utiliser l’identité managée**. Selon vos besoins, vous pouvez choisir des options sous **ID de ressource d’authentification**. Pour plus de détails, consultez [Identités managées pour Azure SignalR Service](howto-use-managed-identity.md).

## <a name="create-upstream-settings-via-resource-manager-template"></a>Créer des paramètres en amont via un modèle Resource Manager

Pour créer des paramètres en amont à l’aide d’un [modèle de Azure Resource Manager](../azure-resource-manager/templates/overview.md), définissez la propriété `upstream` dans la propriété `properties`. L’extrait de code suivant montre comment définir la propriété `upstream` pour la création et la mise à jour de paramètres en amont.

```JSON
{
  "properties": {
    "upstream": {
      "templates": [
        {
          "UrlTemplate": "http://host.com/{hub}/api/{category}/{event}",
          "EventPattern": "*",
          "HubPattern": "*",
          "CategoryPattern": "*",
          "Auth": {
            "Type": "ManagedIdentity",
            "ManagedIdentity": {
              "Resource": "<resource>"
            }
          }
        }
      ]
    }
  }
}
```

## <a name="serverless-protocols"></a>Protocoles serverless

Azure SignalR Service envoie des messages aux points de terminaison qui suivent les protocoles suivants.

### <a name="method"></a>Méthode

POST

### <a name="request-header"></a>En-tête de requête

|Nom |Description|
|---------|---------|
|X-ASRS-Connection-Id |ID de connexion pour la connexion client.|
|X-ASRS-Hub |Hub auquel appartient la connexion client.|
|X-ASRS-Category |Catégorie à laquelle appartient le message.|
|X-ASRS-Event |Événement auquel appartient le message.|
|X-ASRS-Signature |Code HMAC (Hash-based Message Authentication Code) utilisé pour la validation. Pour plus d’informations, consultez [Signature](#signature).|
|X-ASRS-User-Claims |Groupe de revendications de la connexion client.|
|X-ASRS-User-Id |Identité d’utilisateur du client qui envoie le message.|
|X-ASRS-Client-Query |Requête de la demande quand des clients se connectent au service.|
|Authentification |Jeton facultatif lorsque vous utilisez `ManagedIdentity`. |

### <a name="request-body"></a>Corps de la demande

#### <a name="connected"></a>Connecté

Content-Type: application/json

#### <a name="disconnected"></a>Déconnecté

Type de contenu : `application/json`

|Nom  |Type  |Description  |
|---------|---------|---------|
|Error |string |Message d’erreur d’une connexion fermée. Vide quand la connexions se ferme sans erreur.|

#### <a name="invocation-message"></a>Message d’appel

Type de contenu : `application/json` ou `application/x-msgpack`

|Nom  |Type  |Description  |
|---------|---------|---------|
|InvocationId |string | Chaîne facultative représentant un message d’appel. Pour plus d’informations, consultez [Appels](https://github.com/dotnet/aspnetcore/blob/master/src/SignalR/docs/specs/HubProtocol.md#invocations).|
|Cible |string | Identique à l’événement et à la cible dans un [message d’appel](https://github.com/dotnet/aspnetcore/blob/master/src/SignalR/docs/specs/HubProtocol.md#invocation-message-encoding). |
|Arguments |Tableau d’objets |Tableau contenant des arguments à appliquer à la méthode référencée dans `Target`. |

### <a name="signature"></a>Signature

Le service calcule le code SHA256 pour la valeur `X-ASRS-Connection-Id` à l’aide de la clé d’accès principale et de la clé d’accès secondaire en tant que clé `HMAC`. Le service définit la signature dans l’en-tête `X-ASRS-Signature` lors du placement de requêtes HTTP en amont :
```
Hex_encoded(HMAC_SHA256(accessKey, connection-id))
```

## <a name="next-steps"></a>Étapes suivantes

- [Identités managées pour Azure SignalR Service](howto-use-managed-identity.md)
- [Développement et configuration Azure Functions avec Azure SignalR Service](signalr-concept-serverless-development-config.md)