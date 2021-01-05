---
title: Guide du protocole de connexions hybrides Azure Relay | Microsoft Docs
description: Cet article décrit les interactions côté client avec le relais Connexions hybrides pour la connexion de clients ayant le rôle d’expéditeur ou d’écouteur.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 8a812aa401077b81934d89ada99cf1dc312d8dbc
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96862324"
---
# <a name="azure-relay-hybrid-connections-protocol"></a>Protocole de connexions hybrides Azure Relay

Azure Relay est l’une des fonctionnalités clés de la plateforme Azure Service Bus. La nouvelle fonctionnalité _Connexions hybrides_ de Relay est une évolution sécurisée, à protocole ouvert, sur HTTP et WebSocket. Elle remplace l’ancienne fonctionnalité, nommée _BizTalk Services_, conçue sur un protocole propriétaire. L’intégration des connexions hybrides dans Azure App Services continue de fonctionner telle quelle.

La fonctionnalité Connexions hybrides permet d’établir une communication de flux binaire bidirectionnel et un flux de datagrammes simple entre deux applications en réseau. L’une de ces applications ou les deux peuvent se trouver derrière un NAT ou un pare-feu.

Cet article décrit les interactions côté client avec le relais Connexions hybrides pour la connexion de clients ayant le rôle d’expéditeur ou d’écouteur. Cet article décrit également la façon dont les écouteurs acceptent de nouvelles connexions et requêtes.

## <a name="interaction-model"></a>Modèle d’interaction

Le relais Connexions hybrides connecte deux parties en fournissant un point de rencontre détectable par ces parties dans le cloud Azure, et auquel elles peuvent se connecter du point de vue de leur propre réseau. Ce point de rencontre est appelé « connexion hybride » dans ce document et dans d’autres articles, dans les API, ainsi que dans le Portail Azure. Le point de terminaison de service Connexions hybrides est appelé « service » dans la suite de cet article.

Le service autorise le relais de connexions WebSocket et de requêtes et réponses HTTP(S).

Le modèle d’interaction utilise la nomenclature établie par de nombreuses autres API de mise en réseau. Un écouteur indique tout d’abord qu’il est prêt à gérer des connexions entrantes, puis les accepte dès leur arrivée. De l’autre côté, un client se connecte à l’écouteur et attend l’acceptation de cette connexion pour établir un chemin de communication bidirectionnelle. « Se connecter », « écouter » et « accepter » sont les termes que vous trouverez dans la plupart des API de sockets.

Dans tout modèle de communication relayée, l’une des parties établit des connexions sortantes vers un point de terminaison de service, ce qui fait que « l’écouteur » est également un « client » dans le langage courant, entre autres surcharges terminologiques. Par conséquent, voici la terminologie précise que nous utilisons pour les connexions hybrides :

Les programmes des deux côtés d’une connexion sont appelés « clients », puisqu’il s’agit de clients du service. Le client qui attend et accepte les connexions est « l’écouteur » ; on peut également dire qu’il a le « rôle d’écouteur ». Le client qui démarre une nouvelle connexion vers un écouteur par le biais du service est appelé « expéditeur » ou considéré comme ayant le « rôle d’expéditeur ».

### <a name="listener-interactions"></a>Interactions de l’écouteur

L’écouteur a cinq interactions avec le service ; tous les détails sont décrits plus loin dans cet article, dans la section des références.

Les messages d’écoute, d’acceptation et de requête sont reçus en provenance du service. Les opérations de renouvellement et de test ping sont envoyées par l’écouteur.

#### <a name="listen-message"></a>Message d’écoute

Pour indiquer au service qu’il est prêt à accepter les connexions, l’écouteur crée une connexion WebSocket sortante. L’établissement de la connexion porte le nom d’une connexion hybride configurée dans l’espace de noms Relay, et un jeton de sécurité qui confère le droit « d’écoute » sur ce nom.

Quand le WebSocket est accepté par le service, l’inscription est terminée et le WebSocket établi reste actif en tant que « canal de contrôle » autorisant toutes les interactions suivantes. Le service autorise jusqu’à 25 écouteurs simultanés pour une même connexion hybride. Le quota pour les crochets d’application (AppHooks) doit être déterminé.

Dans le cas des connexions hybrides, s’il existe plusieurs écouteurs actifs, les connexions entrantes sont réparties entre tous ces écouteurs selon un ordre aléatoire, et de façon la plus équitable possible.

#### <a name="accept-message"></a>Message d’acceptation

Quand un expéditeur ouvre une nouvelle connexion sur le service, celui-ci choisit et informe l’un des écouteurs actifs sur la connexion hybride. Cette notification est envoyée à l’écouteur sur le canal de contrôle ouvert sous la forme d’un message JSON. Ce message contient l’URL du point de terminaison du WebSocket auquel doit se connecter l’écouteur pour accepter la connexion.

L’URL peut et doit être utilisée directement par l’écouteur sans aucun travail supplémentaire.
Les informations encodées sont valides uniquement pendant une courte durée, correspondant pour l’essentiel au temps que l’expéditeur est disposé à attendre que la connexion soit établie de bout en bout, mais sans dépasser 30 secondes. L’URL ne peut être utilisée que pour une seule tentative de connexion réussie. Dès que la connexion du WebSocket à l’URL de rencontre est établie, toutes les activités supplémentaires sur ce WebSocket sont relayées à partir et en direction de l’expéditeur. Cette opération s’effectue sans aucune intervention ni interprétation par le service.

### <a name="request-message"></a>Message de requête

Outre les connexions WebSocket, l’écouteur peut recevoir des trames de requêtes HTTP de la part d’un expéditeur, si cette fonctionnalité est explicitement activée sur la connexion hybride.

Les écouteurs qui se joignent aux connexions hybrides prenant en charge HTTP DOIVENT gérer l’opération `request`. Un écouteur ne prenant pas en charge l’opération `request` et occasionnant par conséquent des erreurs répétées de dépassement de délai de connexion RISQUE par la suite d’être bloqué par le service.

Les métadonnées d’en-tête de trame HTTP sont converties au format JSON pour en faciliter le traitement par l’infrastructure d’écouteur, et également parce que les bibliothèques d’analyse d’en-tête HTTP sont plus rares que les analyseurs JSON. Les métadonnées HTTP qui s’appliquent uniquement à la relation entre l’expéditeur et la passerelle HTTP de relais, y compris les informations d’autorisation, ne sont pas transférées. Le corps des requêtes HTTP est transféré de manière transparente sous la forme d’une trame WebSocket binaire.

L’écouteur peut répondre aux requêtes HTTP à l’aide d’une opération de réponse équivalente.

Le flux de requêtes/réponses utilise le canal de contrôle par défaut, mais peut être « mis à niveau » vers un WebSocket de rencontre distinct chaque fois que nécessaire. L’utilisation de connexions WebSocket distinctes améliore le débit de chaque conversation des clients, mais elle surcharge l’écouteur en augmentant le nombre de connexions que ce dernier doit traiter, ce qui n’est pas souhaitable dans le cas des clients légers.

Sur le canal de contrôle, la taille du corps des requêtes et des réponses ne doit pas dépasser 64 Ko. Quant aux métadonnées d’en-tête HTTP, elles sont limitées à une taille totale de 32 Ko. Si la requête ou la réponse dépassent ce seuil, l’écouteur DOIT effectuer une mise à niveau vers un WebSocket de rencontre à l’aide d’une opération équivalente au traitement du message [d’acceptation](#accept-message).

Pour les requêtes, le service décide d’acheminer ou non les requêtes sur le canal de contrôle. Ceci inclut notamment les cas dans lesquels la taille d’une requête dépasse sensiblement 64 Ko (en-têtes + corps), ou lorsque la requête est envoyée avec un [encodage de transfert en bloc](https://tools.ietf.org/html/rfc7230#section-4.1) et que le service a des raisons de s’attendre à ce que la requête dépasse 64 Ko ou à ce que la lecture de la requête ne soit pas instantanée. Si le service choisit de distribuer la requête par le biais du point de rencontre, il transmet uniquement l’adresse de ce dernier à l’écouteur.
L’écouteur DOIT alors établir le WebSocket de rencontre, après quoi le service distribue rapidement la requête complète, y compris le corps, sur ce WebSocket. La réponse DOIT également utiliser le WebSocket de rencontre.

Dans le cas des requêtes qui arrivent sur le canal de contrôle, l’écouteur décide de répondre ou non sur le canal de contrôle ou par le biais du socket de rencontre. Le service DOIT inclure une adresse de rencontre avec chaque requête acheminée sur le canal de contrôle. Cette adresse est uniquement valide pour la mise à niveau à partir de la requête actuelle.

Si l’écouteur choisit d’effectuer une mise à niveau, il se connecte et distribue rapidement la réponse sur le socket de rencontre.

Une fois le WebSocket de rencontre établi, l’écouteur DOIT le maintenir en place pour le traitement des requêtes et des réponses ultérieures à partir du même client. Le service préserve le WebSocket aussi longtemps que la connexion socket HTTPS avec l’expéditeur persiste, et il achemine toutes les requêtes ultérieures émanant de cet expéditeur par l’intermédiaire du WebSocket conservé. Si l’écouteur choisit de supprimer le WebSocket de rencontre de son côté, le service supprime également la connexion vers l’expéditeur, qu’une requête ultérieure soit ou non déjà en cours.

#### <a name="renew-operation"></a>Opération de renouvellement

Le jeton de sécurité qui doit être utilisé pour enregistrer l’écouteur et maintenir le canal de contrôle peut expirer alors que l’écouteur est actif. L’expiration du jeton n’affecte pas les connexions en cours, mais elle entraîne l’annulation du canal de contrôle par le service au moment de l’expiration ou juste après. L’opération « renouveler » est un message JSON que peut envoyer l’écouteur pour remplacer le jeton associé au canal de contrôle, afin de conserver le canal de contrôle plus longtemps.

#### <a name="ping-operation"></a>Opération de test ping

Si le canal de contrôle reste inactif pendant une longue période, les intermédiaires présents sur l’itinéraire, comme les équilibreurs de charge ou les NAT, peuvent supprimer la connexion TCP. L’opération « ping » permet d’éviter cela en envoyant une petite quantité de données sur le canal, qui rappelle à tous les éléments présents sur l’itinéraire du réseau que la connexion est destinée à rester active ; elle sert également de test d’activité de l’écouteur. Si la commande ping échoue, le canal de contrôle doit être considéré comme inutilisable et l’écouteur doit se reconnecter.

### <a name="sender-interaction"></a>Interaction de l’expéditeur

L’expéditeur a deux interactions avec le service : il connecte un WebSocket ou il envoie des requêtes par le biais de HTTPS. Il est impossible d’envoyer des requêtes sur un WebSocket à partir du rôle d’expéditeur.

#### <a name="connect-operation"></a>Opération de connexion

L’opération « se connecter » ouvre un WebSocket sur le service, fournissant le nom de la connexion hybride et un jeton de sécurité (facultatif, mais obligatoire par défaut) conférant l’autorisation « Envoyer » dans la chaîne de requête. Le service interagit ensuite avec l’écouteur de la façon décrite précédemment, et l’écouteur crée une connexion conjointe avec ce WebSocket. Une fois le WebSocket accepté, toutes les autres interactions sur ce WebSocket se font avec un écouteur connecté.

#### <a name="request-operation"></a>Opération de requête

Dans le cas des connexions hybrides pour lesquelles cette fonctionnalité a été activée, l’expéditeur peut envoyer aux écouteurs des requêtes HTTP quasiment illimitées.

À l’exception d’un jeton d’accès Relay qui est incorporé dans la chaîne de requête ou dans un en-tête HTTP de la requête, le relais est totalement transparent pour toutes les opérations HTTP sur l’adresse de relais et tous les suffixes du chemin d’adresse de relais, laissant ainsi à l’écouteur le contrôle total de l’autorisation de bout en bout, et même des fonctionnalités d’extension HTTP telles que [CORS](https://www.w3.org/TR/cors/).

L’autorisation d’expéditeur avec le point de terminaison de relais est activée par défaut, mais est FACULTATIVE. Le propriétaire de la connexion hybride peut choisir d’autoriser les expéditeurs anonymes. Le service intercepte, inspecte et supprime les informations d’autorisation comme suit :

1. Si la chaîne de requête contient une expression `sb-hc-token`, cette expression est TOUJOURS supprimée de la chaîne de requête. Elle est évaluée si l’autorisation de relais est activée.
2. Si les en-têtes de requête contiennent un en-tête `ServiceBusAuthorization`, l’expression d’en-tête est TOUJOURS supprimée de la collection d’en-têtes.
   Elle est évaluée si l’autorisation de relais est activée.
3. L’en-tête n’est évalué et supprimé que si l’autorisation de relais est activée, que les en-têtes de requête contiennent un en-tête `Authorization`, et qu’aucune des expressions précédentes n’est présente. Dans le cas contraire, l’en-tête `Authorization` est toujours transmis tel quel.

Si aucun écouteur n’est actif, le service renvoie un code d’erreur 502 « Passerelle incorrecte ». Si le service ne semble pas traiter la requête, le service renvoie un message 504 « Dépassement du délai de la passerelle » au bout de 60 secondes.

### <a name="interaction-summary"></a>Résumé de l’interaction

Le résultat de ce modèle d’interaction est que le client expéditeur ressort de la liaison avec un WebSocket « propre », qui est connecté à un écouteur et n’a besoin d’aucun préambule et d’aucune préparation supplémentaire. Avec ce modèle, presque toutes les implémentations client existantes de WebSocket peuvent facilement tirer parti du service Connexions hybrides en fournissant une URL correctement construite dans la couche client de leur WebSocket.

Le WebSocket de connexion jointe qu’obtient l’écouteur par le biais de l’interaction d’acceptation est également propre et peut être transmis à une implémentation serveur de WebSocket existante avec une abstraction supplémentaire minimale qui fait la distinction entre les opérations « accepter » sur les écouteurs du réseau local de leur infrastructure et les opérations « accepter » à distance des Connexions hybrides.

Le modèle de requête/réponse HTTP offre à l’expéditeur une surface d’exposition de protocole HTTP quasiment illimitée avec une couche d’autorisation FACULTATIVE. L’écouteur obtient une section d’en-tête de requête HTTP préalablement analysée qui peut être reconvertie en requête HTTP en aval ou traitée telle quelle, avec des trames binaires véhiculant les corps HTTP. Les réponses utilisent le même format. Les interactions avec des corps de requête et de réponse inférieurs à 64 Ko peuvent être traitées par le biais d’un WebSocket unique partagé par tous les expéditeurs. Les requêtes et réponses plus volumineuses peuvent être traitées à l’aide du modèle de point de rencontre.

## <a name="protocol-reference"></a>Référence sur le protocole

Cette section fournit les détails des interactions de protocole décrites plus haut.

Toutes les connexions de WebSocket sont effectuées sur le port 443 mis à niveau à partir de HTTPS 1.1, qui est couramment abstrait par les API et les infrastructures de WebSocket. Cette description ne tient pas compte de l’implémentation et ne suggère aucune infrastructure en particulier.

### <a name="listener-protocol"></a>Protocole de l’écouteur

Le protocole de l’écouteur se compose de deux mouvements de connexion et de trois opérations messages.

#### <a name="listener-control-channel-connection"></a>Connexion du canal de contrôle de l’écouteur

Le canal de contrôle est ouvert lors de la création d’une connexion de WebSocket à :

`wss://{namespace-address}/$hc/{path}?sb-hc-action=...[&sb-hc-id=...]&sb-hc-token=...`

`namespace-address` est le nom de domaine complet de l’espace de noms Azure Relay qui héberge la connexion hybride, généralement de la forme `{myname}.servicebus.windows.net`.

Voici les options de paramètres des chaînes de requête.

| Paramètre        | Obligatoire | Description
| ---------------- | -------- | -------------------------------------------
| `sb-hc-action`   | Oui      | Pour le rôle d’écouteur, le paramètre doit être **sb-hc-action=listen**
| `{path}`         | Oui      | Chemin de l’espace de noms encodé au format URL de la connexion hybride préconfigurée sur laquelle cet écouteur doit être inscrit. Cette expression est ajoutée à la partie fixe `$hc/` du chemin.
| `sb-hc-token`    | Oui\*    | L’écouteur doit fournir un jeton d’accès partagé Service Bus valide et encodé au format URL pour l’espace de noms ou la connexion hybride qui confère le droit **d’écoute**.
| `sb-hc-id`       | Non       | Cet ID facultatif fourni par le client permet un suivi de diagnostic de bout en bout.

Si la connexion du WebSocket échoue car le chemin de la connexion hybride n’est pas inscrit, en raison d’un jeton non valide ou manquant ou à cause d’une erreur d’un autre type, les commentaires sur l’erreur sont fournis d’après le modèle classique de commentaires d’état HTTP 1.1. La description d’état contient un ID de suivi d’erreur qui peut être communiqué au personnel de support Azure :

| Code | Error          | Description
| ---- | -------------- | -------------------------------------------------------------------
| 404  | Introuvable      | Le chemin de la connexion hybride n’est pas valide ou l’URL de base est incorrecte.
| 401  | Non autorisé   | Le jeton de sécurité est manquant, incorrect ou non valide.
| 403  | Interdit      | Le jeton de sécurité n’est pas valide pour ce chemin d’accès et pour cette action.
| 500  | Erreur interne | Un problème est survenu dans le service.

Si la connexion du WebSocket est intentionnellement arrêtée par le service après sa configuration initiale, la raison de cette opération est communiquée par le biais d’un code d’erreur approprié du protocole WebSocket avec un message d’erreur descriptif qui inclut également un ID de suivi. Le service ne peut pas arrêter le canal de contrôle sans rencontrer un état d’erreur. Les arrêts normaux sont contrôlés par le client.

| État du SW | Description
| --------- | -------------------------------------------------------------------------------
| 1001      | Le chemin d’accès de la connexion hybride a été supprimé ou désactivé.
| 1008      | Le jeton de sécurité a expiré et, par conséquent, la stratégie d’autorisation n’est pas respectée.
| 1010      | Un problème est survenu dans le service.

#### <a name="accept-handshake"></a>Négociation d’acceptation

La notification d’acceptation est envoyée par le service à l’écouteur sur le canal de contrôle établi précédemment sous forme de message JSON dans un bloc de texte de WebSocket. Il n’y a pas de réponse à ce message.

Le message contient un objet JSON nommé « accept », qui définit les propriétés suivantes à ce jour :

* **address** : chaîne d’URL à utiliser pour que le WebSocket du service accepte une connexion entrante.
* **ID** : identificateur unique de cette connexion. Si l’ID a été fourni par le client expéditeur, il s’agit de la valeur fournie par l’expéditeur, sinon d’une valeur générée par le système.
* **connectHeaders** : tous les en-têtes HTTP qui ont été fournis au point de terminaison Relay par l’expéditeur, dont également les en-têtes Sec-WebSocket-Protocol et Sec-WebSocket-Extensions.

```json
{
    "accept" : {
        "address" : "wss://dc-node.servicebus.windows.net:443/$hc/{path}?..."
        "id" : "4cb542c3-047a-4d40-a19f-bdc66441e736",
        "connectHeaders" : {
            "Host" : "...",
            "Sec-WebSocket-Protocol" : "...",
            "Sec-WebSocket-Extensions" : "..."
        }
     }
}
```

L’adresse URL fournie dans le message JSON est utilisée par l’écouteur pour que le WebSocket accepte ou rejette le socket de l’expéditeur.

##### <a name="accepting-the-socket"></a>Accepter le socket

Pour accepter, l’écouteur établit une connexion WebSocket à l’adresse fournie.

Si le message d’acceptation comporte un en-tête `Sec-WebSocket-Protocol`, il est attendu que l’écouteur accepte uniquement le WebSocket s’il prend en charge ce protocole. De plus, il définit l’en-tête au moment où le WebSocket est établi.

Il en va de même pour l’en-tête `Sec-WebSocket-Extensions`. Si le framework prend en charge une extension, il doit définir l’en-tête comme la réponse côté serveur de la liaison `Sec-WebSocket-Extensions` requise pour l’extension.

L’URL doit être utilisée telle quelle pour établir le socket d’acceptation, mais elle contient les paramètres suivants :

| Paramètre      | Obligatoire | Description
| -------------- | -------- | -------------------------------------------------------------------
| `sb-hc-action` | Oui      | Pour accepter un socket, le paramètre doit être `sb-hc-action=accept`
| `{path}`       | Oui      | (voir le paragraphe suivant)
| `sb-hc-id`     | Non       | Voir la description précédente de **id**.

`{path}` est le chemin de l’espace de noms encodé au format URL de la connexion hybride préconfigurée sur laquelle cet écouteur doit être inscrit. Cette expression est ajoutée à la partie fixe `$hc/` du chemin.

L’expression `path` peut être étendue par un suffixe et une expression de chaîne de requête qui suit le nom inscrit après une barre oblique de séparation.
Cela permet au client expéditeur de transmettre des arguments à l’écouteur quand il n’est pas possible d’inclure des en-têtes HTTP. L’objectif est que le framework de l’écouteur analyse la partie fixe du chemin et le nom inscrit dans le chemin, et rende le reste (éventuellement sans argument de chaîne de requête préfixé par `sb-`), accessible à l’application pour que celle-ci puisse décider d’accepter ou non la connexion.

Pour plus d’informations, consultez la section suivante, « Protocole de l’expéditeur ».

En cas d’erreur, le service peut répondre ceci :

| Code | Error          | Description
| ---- | -------------- | -----------------------------------
| 403  | Interdit      | L’URL n’est pas valide.
| 500  | Erreur interne | Un problème est survenu dans le service.

 Une fois la connexion établie, le serveur arrête le WebSocket quand le WebSocket expéditeur s’arrête, ou avec l’état suivant :

| État du SW | Description                                                                     |
| --------- | ------------------------------------------------------------------------------- |
| 1001      | Le client expéditeur arrête la connexion.                                    |
| 1001      | Le chemin d’accès de la connexion hybride a été supprimé ou désactivé.                        |
| 1008      | Le jeton de sécurité a expiré et, par conséquent, la stratégie d’autorisation n’est pas respectée. |
| 1010      | Un problème est survenu dans le service.                                            |

##### <a name="rejecting-the-socket"></a>Rejeter le socket

 Le rejet du socket après l’inspection du message `accept` nécessite l’établissement d’une liaison similaire afin que le code et la description d’état communiquant le motif du rejet puissent être transmis à l’expéditeur.

 Le choix de conception du protocole consiste ici à utiliser une liaison WebSocket (qui est conçue pour se terminer avec un état d’erreur défini) afin que les implémentations du client écouteur puissent continuer à s’appuyer sur un client WebSocket sans devoir utiliser un client HTTP supplémentaire sans système d’exploitation.

 Pour rejeter le socket, le client prend l’URI d’adresse du message `accept` et y ajoute deux paramètres de chaîne de requête, comme suit :

| Paramètre                   | Obligatoire | Description                              |
| ----------------------- | -------- | ---------------------------------------- |
| sb-hc-statusCode        | Oui      | Code d’état HTTP numérique.                |
| sb-hc-statusDescription | Oui      | Motif du rejet lisible. |

L’URI obtenu est ensuite utilisé pour établir une connexion WebSocket.

En cas de réussite, cette liaison échoue intentionnellement avec un code d’erreur HTTP 410, car aucun WebSocket n’a été établi. Si une erreur survient, les codes suivants la décrivent :

| Code | Error          | Description                          |
| ---- | -------------- | ------------------------------------ |
| 403  | Interdit      | L’URL n’est pas valide.                |
| 500  | Erreur interne | Un problème est survenu dans le service. |

#### <a name="request-message"></a>Message de requête

Le message `request` est envoyé par le service à l’écouteur sur le canal de contrôle. Le même message est également envoyé par le biais du WebSocket de rencontre, une fois ce dernier établi.

Le message `request` comporte deux parties : un en-tête et une ou plusieurs trames de corps binaires.
S’il n’existe aucun corps, les trames de corps sont omises. L’indicateur qui détermine si un corps est ou non présent correspond à la propriété `body` booléenne dans le message de requête.

Dans le cas d’une requête avec un corps, la structure peut ressembler à ceci :

``` text
----- Web Socket text frame ----
{
    "request" :
    {
        "body" : true,
        ...
    }
}
----- Web Socket binary frame ----
FEFEFEFEFEFEFEFEFEFEF...
----- Web Socket binary frame ----
FEFEFEFEFEFEFEFEFEFEF...
----- Web Socket binary frame -FIN
FEFEFEFEFEFEFEFEFEFEF...
----------------------------------
```

L’écouteur DOIT traiter la réception du corps de la requête fractionné en plusieurs trames binaires (consultez le document relatif aux [fragments WebSocket](https://tools.ietf.org/html/rfc6455#section-5.4)).
La requête s’achève lorsqu’une trame binaire avec l’indicateur FIN a été reçue.

Dans le cas d’une requête sans corps, il n’existe qu’une seule trame de texte.

``` text
----- Web Socket text frame ----
{
    "request" :
    {
        "body" : false,
        ...
    }
}
----------------------------------
```

Le contenu JSON pour `request` est le suivant :

* **address** :chaîne d’URI. Il s’agit de l’adresse de rencontre à utiliser pour cette requête. Si la requête entrante est supérieure à 64 Ko, le reste de ce message est vide, et le client DOIT initialiser un établissement de liaison de rencontre équivalent à l’opération `accept` décrite ci-après. Le service place ensuite l’objet `request` complet sur le WebSocket établi. Si la réponse est censée dépasser 64 Ko, l’écouteur DOIT également initier un établissement de liaison de rencontre, puis transférer la réponse sur le WebSocket établi.
* **id** : chaîne. Identificateur unique de cette requête.
* **requestHeaders** : cet objet contient tous les en-têtes HTTP qui ont été fournis au point de terminaison par l’expéditeur, à l’exception des informations d’autorisation comme expliqué [ci-dessus](#request-operation), ainsi que les en-têtes qui se rapportent strictement à la connexion avec la passerelle. Plus précisément, TOUS les en-têtes définis ou réservés dans [RFC7230](https://tools.ietf.org/html/rfc7230), à l’exception de `Via`, sont supprimés et ne sont pas transférés :

  * `Connection` (RFC7230, section 6.1)
  * `Content-Length` (RFC7230, section 3.3.2)
  * `Host` (RFC7230, section 5.4)
  * `TE` (RFC7230, section 4.3)
  * `Trailer` (RFC7230, section 4.4)
  * `Transfer-Encoding` (RFC7230, section 3.3.1)
  * `Upgrade` (RFC7230, section 6.7)
  * `Close` (RFC7230, section 8.1)

* **requestTarget** : chaîne. Cette propriété contient la [« cible de requête » (RFC7230, section 5.3)](https://tools.ietf.org/html/rfc7230#section-5.3) de la requête. Ceci inclut la partie chaîne de la requête, qui est supprimée de TOUS les paramètres présentant le préfixe `sb-hc-`.
* **method** : chaîne. Il s’agit de la méthode de la requête, conformément à [RFC7231, section 4](https://tools.ietf.org/html/rfc7231#section-4). La méthode `CONNECT` NE DOIT PAS être utilisée.
* **body** : valeur booléenne. Indique si une ou plusieurs trames de corps binaires suivent ou non.

``` JSON
{
    "request" : {
        "address" : "wss://dc-node.servicebus.windows.net:443/$hc/{path}?...",
        "id" : "42c34cb5-7a04-4d40-a19f-bdc66441e736",
        "requestTarget" : "/abc/def?myarg=value&otherarg=...",
        "method" : "GET",
        "requestHeaders" : {
            "Host" : "...",
            "Content-Type" : "...",
            "User-Agent" : "..."
        },
        "body" : true
     }
}
```

##### <a name="responding-to-requests"></a>Réponse aux requêtes

Le destinataire DOIT répondre. Des échecs répétés des réponses aux requêtes pendant que la connexion est maintenue peuvent entraîner le blocage de l’écouteur.

Les réponses peuvent être envoyées dans n’importe quel ordre, mais chaque requête doit obtenir une réponse dans les 60 secondes, sans quoi la remise sera considérée comme ayant échoué. Le délai de 60 secondes est pris en compte jusqu’à ce que la trame `response` ait été reçue par le service. Une réponse en cours avec plusieurs trames binaires ne peut pas rester inactive pendant plus de 60 secondes ; dans le cas contraire, elle est interrompue.

Si la requête est reçue sur le canal de contrôle, la réponse DOIT être envoyée soit sur le canal de contrôle à partir duquel la requête a été reçue, soit par le biais d’un canal de rencontre.

La réponse est un objet JSON nommé « response ». Les règles de traitement du contenu du corps sont exactement identiques à celles du message `request` et reposent sur la propriété `body`.

* **requestId** : chaîne. OBLIGATOIRE. Valeur de la propriété `id` du message `request` auquel la réponse s’adresse.
* **statusCode** : valeur numérique. OBLIGATOIRE. Code d’état HTTP numérique indiquant le résultat de la notification. Tous les codes d’état de [RFC7231, section 6](https://tools.ietf.org/html/rfc7231#section-6) sont autorisés, à l’exception des codes [502 « Passerelle incorrecte »](https://tools.ietf.org/html/rfc7231#section-6.6.3) et [504 « Dépassement du délai de la passerelle »](https://tools.ietf.org/html/rfc7231#section-6.6.5).
* **statusDescription** : chaîne. FACULTATIF. Motif de code d’état HTTP conformément à [RFC7230, section 3.1.2](https://tools.ietf.org/html/rfc7230#section-3.1.2)
* **responseHeaders** : en-têtes HTTP à définir dans une réponse HTTP externe.
  Comme dans le cas de `request`, les en-têtes définis dans RFC7230 NE DOIVENT PAS être utilisés.
* **body** : valeur booléenne. Indique si une ou plusieurs trames de corps binaires suivent ou non.

``` text
----- Web Socket text frame ----
{
    "response" : {
        "requestId" : "42c34cb5-7a04-4d40-a19f-bdc66441e736",
        "statusCode" : "200",
        "responseHeaders" : {
            "Content-Type" : "application/json",
            "Content-Encoding" : "gzip"
        }
         "body" : true
     }
}
----- Web Socket binary frame -FIN
{ "hey" : "mydata" }
----------------------------------
```

##### <a name="responding-via-rendezvous"></a>Réponse par le biais d’un socket de rencontre

Dans le cas des réponses qui dépassent 64 Ko, la réponse DOIT être remise par l’intermédiaire d’un socket de rencontre. En outre, si la requête dépasse 64 Ko et que l’objet `request` contient uniquement le champ d’adresse, il est nécessaire d’établir un socket de rencontre pour obtenir l’objet `request`. Une fois qu’un socket de rencontre a été établi, les réponses destinées au client concerné et les requêtes ultérieures à partir de ce client DOIVENT être distribuées par le biais du socket de rencontre pendant que ce dernier est maintenu.

L’URL `address` de l’objet `request` doit être utilisée telle quelle pour l’établissement du socket de rencontre, mais elle contient les paramètres suivants :

| Paramètre      | Obligatoire | Description
| -------------- | -------- | -------------------------------------------------------------------
| `sb-hc-action` | Oui      | Pour accepter un socket, le paramètre doit être `sb-hc-action=request`

En cas d’erreur, le service peut répondre ceci :

| Code | Error           | Description
| ---- | --------------- | -----------------------------------
| 400  | Demande non valide | Action non reconnue ou URL non valide.
| 403  | Interdit       | L’URL a expiré.
| 500  | Erreur interne  | Un problème est survenu dans le service.

 Une fois la connexion établie, le serveur arrête le WebSocket lorsque le socket HTTP du client s’arrête, ou avec l’état suivant :

| État du SW | Description                                                                     |
| --------- | ------------------------------------------------------------------------------- |
| 1001      | Le client expéditeur arrête la connexion.                                    |
| 1001      | Le chemin d’accès de la connexion hybride a été supprimé ou désactivé.                        |
| 1008      | Le jeton de sécurité a expiré et, par conséquent, la stratégie d’autorisation n’est pas respectée. |
| 1010      | Un problème est survenu dans le service.                                            |


#### <a name="listener-token-renewal"></a>Renouvellement du jeton de l’écouteur

Lorsque le jeton de l’écouteur est sur le point d’expirer, celui-ci peut le remplacer en envoyant un message de bloc de texte au service via le canal de contrôle établi. Le message contient un objet JSON nommé `renewToken`, qui définit la propriété suivante à ce moment donné :

* **token** : jeton d’accès partagé Service Bus valide et encodé au format URL pour l’espace de noms ou la connexion hybride qui confère le droit d’**écoute**.

```json
{
  "renewToken": {
    "token":
      "SharedAccessSignature sr=http%3a%2f%2fcontoso.servicebus.windows.net%2fhyco%2f&amp;sig=XXXXXXXXXX%3d&amp;se=1471633754&amp;skn=SasKeyName"
  }
}
```

Si la validation du jeton échoue, l’accès est refusé et le service cloud ferme le WebSocket du canal de contrôle avec une erreur. Sinon, il n’y a aucune réponse.

| État du SW | Description                                                                     |
| --------- | ------------------------------------------------------------------------------- |
| 1008      | Le jeton de sécurité a expiré et, par conséquent, la stratégie d’autorisation n’est pas respectée. |

### <a name="web-socket-connect-protocol"></a>Protocole de connexion de WebSocket

Le protocole de l’expéditeur est identique au mode d’établissement d’un écouteur.
L’objectif est une transparence maximale pour le WebSocket de bout en bout. L’adresse de connexion est la même que pour l’écouteur, mais « l’action » diffère et le jeton a besoin d’une autorisation différente :

```
wss://{namespace-address}/$hc/{path}?sb-hc-action=...&sb-hc-id=...&sb-hc-token=...
```

_namespace-address_ est le nom de domaine complet de l’espace de noms Azure Relay qui héberge la connexion hybride, généralement de la forme `{myname}.servicebus.windows.net`.

La demande peut contenir des en-têtes HTTP arbitraires supplémentaires, notamment ceux définis par l’application. Tous les en-têtes fournis sont transmis à l’écouteur et se trouvent sur l’objet `connectHeader` du message de contrôle **accept**.

Voici les options de paramètres des chaînes de requête :

| Paramètre          | Requis ? | Description
| -------------- | --------- | -------------------------- |
| `sb-hc-action` | Oui       | Pour le rôle de l’expéditeur, le paramètre doit être `sb-hc-action=connect`.
| `{path}`       | Oui       | (voir le paragraphe suivant)
| `sb-hc-token`  | Oui\*     | L’écouteur doit fournir un jeton d’accès partagé Service Bus valide et encodé au format URL pour l’espace de noms ou la connexion hybride qui confère le droit **d’envoi**.
| `sb-hc-id`     | Non        | ID facultatif qui permet le suivi de diagnostic de bout en bout et est accessible à l’écouteur pendant la liaison d’acceptation.

 Le `{path}` est le chemin d’accès de l’espace de noms encodé au format URL de la connexion hybride préconfigurée sur laquelle cet écouteur doit être inscrit. L’expression `path` peut être étendue par un suffixe ou une expression de chaîne de requête pour prolonger la communication. Si la connexion hybride est enregistrée sous le chemin `hyco`, l’expression `path` peut être `hyco/suffix?param=value&...`, suivi des paramètres de chaîne de requête définis ici. Ainsi, l’expression complète peut être celle-ci :

```
wss://{namespace-address}/$hc/hyco/suffix?param=value&sb-hc-action=...[&sb-hc-id=...&]sb-hc-token=...
```

L’expression `path` est transmise à l’écouteur dans l’URI d’adresse contenu dans le message de contrôle « accept ».

Si la connexion du WebSocket échoue car le chemin de la connexion hybride n’est pas inscrit, en raison d’un jeton non valide ou manquant ou à cause d’une erreur d’un autre type, les commentaires sur l’erreur sont fournis d’après le modèle classique de commentaires d’état HTTP 1.1. La description d’état contient un ID de suivi d’erreur qui peut être communiqué au personnel de support Azure :

| Code | Error          | Description
| ---- | -------------- | -------------------------------------------------------------------
| 404  | Introuvable      | Le chemin de la connexion hybride n’est pas valide ou l’URL de base est incorrecte.
| 401  | Non autorisé   | Le jeton de sécurité est manquant, incorrect ou non valide.
| 403  | Interdit      | Le jeton de sécurité n’est pas valide pour ce chemin et pour cette action.
| 500  | Erreur interne | Un problème est survenu dans le service.

Si la connexion du WebSocket est intentionnellement arrêtée par le service après sa configuration initiale, la raison de cette opération est communiquée par le biais d’un code d’erreur approprié du protocole WebSocket avec un message d’erreur descriptif qui inclut également un ID de suivi.

| État du SW | Description
| --------- | ------------------------------------------------------------------------------- 
| 1 000      | L’écouteur a fermé le socket.
| 1001      | Le chemin d’accès de la connexion hybride a été supprimé ou désactivé.
| 1008      | Le jeton de sécurité a expiré et, par conséquent, la stratégie d’autorisation n’est pas respectée.
| 1010      | Un problème est survenu dans le service.

### <a name="http-request-protocol"></a>Protocole de requête HTTP

Le protocole de requête HTTP autorise les requêtes HTTP arbitraires, à l’exception des mises à niveau du protocole.
Les requêtes HTTP pointent vers l’adresse d’exécution régulière de l’entité, sans l’infixe $hc qui est utilisé pour les clients WebSocket des connexions hybrides.

```
https://{namespace-address}/{path}?sb-hc-token=...
```

_namespace-address_ est le nom de domaine complet de l’espace de noms Azure Relay qui héberge la connexion hybride, généralement de la forme `{myname}.servicebus.windows.net`.

La demande peut contenir des en-têtes HTTP arbitraires supplémentaires, notamment ceux définis par l’application. Tous les en-têtes fournis, à l’exception de ceux directement définis dans RFC7230 (voir la section relative au [message de requête](#request-message)) sont transmis à l’écouteur et se trouvent sur l’objet `requestHeader` du message de **requête**.

Voici les options de paramètres des chaînes de requête :

| Paramètre          | Requis ? | Description
| -------------- | --------- | ---------------- |
| `sb-hc-token`  | Oui\*     | L’écouteur doit fournir un jeton d’accès partagé Service Bus valide et encodé au format URL pour l’espace de noms ou la connexion hybride qui confère le droit **d’envoi**.

Le jeton peut également être véhiculé dans l’en-tête HTTP `ServiceBusAuthorization` ou `Authorization`. Le jeton peut être omis si la connexion hybride est configurée de façon à autoriser les requêtes anonymes.

Étant donné que le service agit en réalité à la façon d’un proxy, il ajoute un en-tête `Via` ou annote l’en-tête `Via` existant conforme à [RFC7230, section 5.7.1](https://tools.ietf.org/html/rfc7230#section-5.7.1).
Le service ajoute le nom d’hôte d’espace de noms de relais à l’objet `Via`.

| Code | Message  | Description                    |
| ---- | -------- | ------------------------------ |
| 200  | OK       | La requête a été traitée par un écouteur au moins.  |
| 202  | Acceptée | La requête a été acceptée par un écouteur au moins. |

En cas d’erreur, le service peut répondre comme suit. Il est possible de déterminer si la réponse provient du service ou de l’écouteur grâce à la présence ou à l’absence de l’en-tête `Via`. Si cet en-tête est présent, la réponse émane de l’écouteur.

| Code | Error           | Description
| ---- | --------------- |--------- |
| 404  | Introuvable       | Le chemin de la connexion hybride n’est pas valide ou l’URL de base est incorrecte.
| 401  | Non autorisé    | Le jeton de sécurité est manquant, incorrect ou non valide.
| 403  | Interdit       | Le jeton de sécurité n’est pas valide pour ce chemin et pour cette action.
| 500  | Erreur interne  | Un problème est survenu dans le service.
| 503  | Passerelle incorrecte     | La requête n’a pas pu être acheminée vers un écouteur.
| 504  | Dépassement du délai de la passerelle | La requête a été acheminée vers un écouteur, mais ce dernier n’a pas accusé réception dans le délai requis.

## <a name="next-steps"></a>Étapes suivantes

* [FAQ Relay](relay-faq.md)
* [Créer un espace de noms](relay-create-namespace-portal.md)
* [Prise en main de .NET](relay-hybrid-connections-dotnet-get-started.md)
* [Prise en main de Node](relay-hybrid-connections-node-get-started.md)
