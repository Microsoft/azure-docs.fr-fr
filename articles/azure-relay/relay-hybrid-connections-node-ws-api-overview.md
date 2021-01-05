---
title: Vue d’ensemble des API Node Azure Relay | Microsoft Docs
description: Cet article fournit une vue d’ensemble de l’API node. js pour le service Azure Relay. Il montre également comment utiliser le package Hyco-WS node.
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-js
ms.openlocfilehash: 558f49c09203192ff4cbb1af392eaeef8d705c94
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91263485"
---
# <a name="relay-hybrid-connections-node-api-overview"></a>Vue d’ensemble des API Node pour les connexions hybrides Relay

## <a name="overview"></a>Vue d’ensemble

Le package Node [`hyco-ws`](https://www.npmjs.com/package/hyco-ws) pour les connexions hybrides Azure Relay repose sur le package NPM [« ws »](https://www.npmjs.com/package/ws) ainsi qu’il l’étend. Ce package réexporte toutes les exportations du package de base et ajoute de nouvelles exportations qui permettent l’intégration avec la fonctionnalité de connexions hybrides du service Azure Relay. 

Les applications existantes de type `require('ws')` peuvent également utiliser ce package avec `require('hyco-ws')`. Cela permet des scénarios hybrides dans lesquels une application écoute les connexions WebSocket en local depuis « l’intérieur du pare-feu » en même temps que via des connexions hybrides.
  
## <a name="documentation"></a>Documentation

Les API sont [documentées dans le package principal « ws »](https://github.com/websockets/ws/blob/master/doc/ws.md). Cet article décrit en quoi ce package diffère du package de base. 

Les principales différences entre le package de base et le package « hyco-ws » sont qu’il ajoute quelques méthodes d’assistance et une nouvelle classe de serveur laquelle est exportée via `require('hyco-ws').RelayedServer`.

### <a name="package-helper-methods"></a>Méthodes d’assistance du package

Plusieurs méthodes utilitaires sont disponibles dans l’exportation du package. Vous pouvez les référencer comme suit :

```JavaScript
const WebSocket = require('hyco-ws');

var listenUri = WebSocket.createRelayListenUri('namespace.servicebus.windows.net', 'path');
listenUri = WebSocket.appendRelayToken(listenUri, 'ruleName', '...key...')
...

```

Ces méthodes d’assistance doivent être utilisées avec ce package. Toutefois, elles peuvent également être utilisées par un serveur Node pour l’activation de clients web ou de périphérique et la création d’écouteurs ou d’expéditeurs. Le serveur utilise ces méthodes en leur transférant des URI qui intègrent des jetons de durée de vie limitée. Ces URI peuvent également être utilisés avec des piles WebSocket courantes qui ne prennent pas en charge la définition d’en-têtes HTTP pour le protocole de transfert WebSocket. L’intégration de jetons d’autorisation dans l’URI est principalement prise en charge pour les scénarios d’utilisation externes à la bibliothèque. 

#### <a name="createrelaylistenuri"></a>createRelayListenUri

```JavaScript
var uri = createRelayListenUri([namespaceName], [path], [[token]], [[id]])
```

Crée un URI d’écouteur de connexion hybride Azure Relay valide pour l’espace de noms et le chemin d’accès indiqués. Cet URI peut ensuite être utilisé avec la version Relay de la classe WebSocketServer.

- `namespaceName` (obligatoire) : nom de domaine qualifié de l’espace de noms Azure Relay à utiliser.
- `path` (obligatoire) : nom d’une connexion hybride Azure Relay existante dans cet espace de noms.
- `token` (facultatif) : jeton d’accès Azure Relay précédemment publié et intégré à l’URI d’écouteur (voir l’exemple suivant).
- `id` (facultatif) : identificateur de suivi qui permet un suivi des diagnostics de bout en bout des demandes.

La valeur `token` est facultative et doit être utilisée uniquement quand il est impossible d’envoyer des en-têtes HTTP avec le protocole de transfert WebSocket, comme c’est le cas avec la pile W3C WebSocket.                  


#### <a name="createrelaysenduri"></a>createRelaySendUri
 
```JavaScript
var uri = createRelaySendUri([namespaceName], [path], [[token]], [[id]])
```

Crée un URI d’envoi de connexion hybride Azure Relay valide pour l’espace de noms et le chemin d’accès indiqués. Cet URI peut être utilisé avec un client WebSocket quel qu’il soit.

- `namespaceName` (obligatoire) : nom de domaine qualifié de l’espace de noms Azure Relay à utiliser.
- `path` (obligatoire) : nom d’une connexion hybride Azure Relay existante dans cet espace de noms.
- `token` (facultatif) : jeton d’accès Azure Relay précédemment publié et intégré à l’URI d’envoi (voir l’exemple suivant).
- `id` (facultatif) : identificateur de suivi qui permet un suivi des diagnostics de bout en bout des demandes.

La valeur `token` est facultative et doit être utilisée uniquement quand il est impossible d’envoyer des en-têtes HTTP avec le protocole de transfert WebSocket, comme c’est le cas avec la pile W3C WebSocket.                   


#### <a name="createrelaytoken"></a>createRelayToken 

```JavaScript
var token = createRelayToken([uri], [ruleName], [key], [[expirationSeconds]])
```

Crée un jeton de signature d’accès partagé Azure Relay pour l’URI cible, la règle de signature d’accès partagé et la clé de règle de signature d’accès partagé indiqués. Ce jeton est valide pendant le nombre de secondes mentionné ou pendant une heure à partir de l’instant présent si l’argument d’expiration est omis.

- `uri` (obligatoire) : URI pour lequel le jeton doit être émis. L’URI est normalisé et utilise le schéma HTTP. Les informations de chaîne de requête sont supprimées.
- `ruleName` (obligatoire) : nom de règle SAS pour l’entité représentée par l’URI indiqué ou pour l’espace de noms représenté par la portion d’hôte URI.
- `key` (obligatoire) : clé valide pour la règle de signature d’accès partagé. 
- `expirationSeconds` (facultatif) : nombre de secondes avant expiration du jeton généré. Si aucune valeur n’est indiquée, la valeur par défaut est 1 heure (3600).

Le jeton émis confère les droits associés à la règle SAS indiquée pour la durée définie.

#### <a name="appendrelaytoken"></a>appendRelayToken

```JavaScript
var uri = appendRelayToken([uri], [ruleName], [key], [[expirationSeconds]])
```

Fonctionnellement parlant, cette méthode équivaut à la méthode `createRelayToken` décrite précédemment, à ceci près qu’elle renvoie le jeton après l’avoir ajouté à l’URI d’entrée.

### <a name="class-wsrelayedserver"></a>Class ws.RelayedServer

La classe `hycows.RelayedServer` constitue une alternative à la classe `ws.Server` qui n’écoute pas sur le réseau local, mais délègue l’écoute au service Azure Relay.

Ces deux classes sont en grande partie compatibles, en ce sens qu’une application existante qui utilise la classe `ws.Server` peut facilement être modifiée pour utiliser la version relayée. Les principales différences résident dans le constructeur et dans les options disponibles.

#### <a name="constructor"></a>Constructeur  

```JavaScript 
var ws = require('hyco-ws');
var server = ws.RelayedServer;

var wss = new server(
    {
        server: ws.createRelayListenUri(ns, path),
        token: function() { return ws.createRelayToken('http://' + ns, keyrule, key); }
    });
```

Le constructeur `RelayedServer` ne prend pas en charge les mêmes arguments que le constructeur `Server`, car il n’est ni un écouteur autonome ni intégrable dans une infrastructure d’écouteur HTTP existante. Il existe également moins d’options disponibles dans la mesure où la gestion WebSocket est en grande partie déléguée au service Relay.

Arguments du constructeur :

- `server` (obligatoire) : URI complet d’un nom de connexion hybride sur lequel écouter et habituellement construit avec la méthode d’assistance WebSocket.createRelayListenUri().
- `token` (obligatoire) : cet argument conserve une chaîne de jeton précédemment émise ou une fonction de rappel qui peut être appelée pour obtenir une telle chaîne de jeton. L’option de rappel est recommandée, car elle permet un renouvellement des jetons.

#### <a name="events"></a>Événements

Les instances `RelayedServer` émettent trois événements qui vous permettent de gérer les demandes entrantes, d’établir des connexions et de détecter les conditions d’erreur. Vous devez vous abonner à l’événement `connect` pour traiter les messages. 

##### <a name="headers"></a>headers

```JavaScript 
function(headers)
```

L’événement `headers` est déclenché juste avant l’acceptation d’une connexion entrante, ce qui permet de modifier les en-têtes à envoyer au client. 

##### <a name="connection"></a>connection

```JavaScript
function(socket)
```

Émis lorsqu’une nouvelle connexion WebSocket est acceptée. L’objet est de type `ws.WebSocket`, tout comme avec le package de base.


##### <a name="error"></a>error

```JavaScript
function(error)
```

Si le serveur sous-jacent émet une erreur, il est transféré ici.  

#### <a name="helpers"></a>Programmes d’assistance

Pour simplifier le démarrage d’un serveur relayé et l’abonnement aux connexions entrantes, le package expose une fonction d’assistance simple, qui est également utilisée dans les exemples suivants :

##### <a name="createrelayedlistener"></a>createRelayedListener

```JavaScript
var WebSocket = require('hyco-ws');

var wss = WebSocket.createRelayedServer(
    {
        server: WebSocket.createRelayListenUri(ns, path),
        token: function() { return WebSocket.createRelayToken('http://' + ns, keyrule, key); }
    }, 
    function (ws) {
        console.log('connection accepted');
        ws.onmessage = function (event) {
            console.log(JSON.parse(event.data));
        };
        ws.on('close', function () {
            console.log('connection closed');
        });       
});
``` 

##### <a name="createrelayedserver"></a>createRelayedServer

```javascript
var server = createRelayedServer([options], [connectCallback] )
```

Cette méthode appelle le constructeur pour créer une nouvelle instance RelayedServer, puis abonne le rappel fourni à l’événement de « connexion ».
 
##### <a name="relayedconnect"></a>relayedConnect

Par une mise en miroir du programme d’assistance `createRelayedServer` en fonction, `relayedConnect` crée une connexion client, puis s’abonne à l’événement d’« ouverture » sur le socket correspondant.

```JavaScript
var uri = WebSocket.createRelaySendUri(ns, path);
WebSocket.relayedConnect(
    uri,
    WebSocket.createRelayToken(uri, keyrule, key),
    function (socket) {
        ...
    }
);
```

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur Azure Relay, consultez les liens suivants :
* [Qu’est-ce qu’Azure Relay ?](relay-what-is-it.md)
* [API Relay disponibles](relay-api-overview.md)
